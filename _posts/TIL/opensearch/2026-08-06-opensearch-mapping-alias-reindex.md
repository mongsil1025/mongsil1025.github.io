---
title:  "[OpenSearch] 인덱스 템플릿, 불변 매핑, 그리고 alias 재색인"
date: 2026-08-06
excerpt: "로컬에 OpenSearch를 띄우고 스키마를 만져보며 정리한 것. 템플릿과 인덱스의 차이, 매핑 타입을 왜 바꿀 수 없는지, 그리고 alias 기반 블루/그린 재색인 패턴."
tags: [til, opensearch]
toc: true
toc_sticky: true
categories: [til/opensearch]
---

OpenSearch를 임베딩 벡터 저장용으로 썼던 정도였다. 스키마를 직접 설계해본 적은 없었다.

로컬에 띄워놓고 인덱스 템플릿부터 alias 재색인까지 손으로 돌려보면서 정리한 내용이다.
이걸 Terraform으로 관리하면서 알게 된 것들은
[[Terraform] Terraform을 처음 직접 굴려보며 알게 된 것들](/til/terraform/terraform-opensearch-schema/)에 따로 적었다.

## 로컬에 띄우기

```yaml
# docker-compose.yml (핵심만)
services:
  opensearch:
    image: opensearchproject/opensearch:2.17.1
    environment:
      - discovery.type=single-node
      - OPENSEARCH_JAVA_OPTS=-Xms1g -Xmx1g
      - bootstrap.memory_lock=true
      - DISABLE_SECURITY_PLUGIN=true
    ulimits:
      memlock: { soft: -1, hard: -1 }
```

로컬에서 OpenSearch가 자꾸 죽는다면 십중팔구 **`OPENSEARCH_JAVA_OPTS`** 다.
지정하지 않으면 JVM이 호스트 메모리의 절반을 잡으려 들어서 컨테이너가 조용히 OOM으로 종료된다.
`Xms`와 `Xmx`를 같은 값으로 고정하는 건 힙 리사이징 중 GC가 몰리는 걸 막는 JVM 관례다.

`bootstrap.memory_lock`과 `ulimits.memlock`은 한 세트다. 힙이 스왑으로 밀려나는 걸 막는데,
하나만 켜면 부팅 시 경고가 뜬다.

보안 플러그인은 실습이라 껐다. 2.12 버전부터는 켜면 `OPENSEARCH_INITIAL_ADMIN_PASSWORD`가
필수이고 https + 자체서명 인증서 처리까지 따라붙는다. 배우려는 주제와 무관한 삽질이 늘어난다.

## RDB와의 대응

| OpenSearch | RDB |
|---|---|
| index | 테이블 |
| document | 행(row), JSON |
| mapping | 스키마 (컬럼 타입 정의) |
| shard | 파티션 (index를 물리적으로 쪼갠 조각) |
| alias | 뷰 / 심볼릭 링크 |

문서를 넣고 빼는 것도 REST로 대응된다.

```
POST /logs-2026.08.06/_doc          → INSERT (id 자동 생성)
PUT  /logs-2026.08.06/_doc/abc123   → INSERT or UPDATE (id 지정)
GET  /logs-2026.08.06/_doc/abc123   → SELECT by id
DELETE /logs-2026.08.06/_doc/abc123 → DELETE
```

**중요한 차이가 하나 있다. 인덱스는 문서를 POST하면 자동으로 생긴다.**
테이블처럼 미리 만들어둘 필요가 없다. 편해 보이지만, 그래서
"생길 때 어떤 스키마로 생길지"를 미리 정해두는 장치가 따로 필요해진다. 그게 템플릿이다.

두 가지를 더 알아뒀다.

- **색인 직후에는 검색되지 않는다.** 기본 1초 뒤(refresh interval)에 검색 가능해진다.
  실습에서 넣자마자 안 나오면 이것 때문이다. `?refresh=true`를 붙이면 즉시 반영된다
- **대량 삽입은 `_bulk`.** 문서 하나당 요청 하나는 너무 느리다.
  형식이 NDJSON이라 줄마다 JSON 하나씩 넣고, Content-Type도 `application/x-ndjson`이며,
  **마지막에 개행이 반드시 있어야 한다**

## 템플릿은 인덱스가 아니다

로그용 인덱스 템플릿을 만들었다.

```json
PUT /_index_template/logs
{
  "index_patterns": ["logs-*"],
  "priority": 100,
  "template": {
    "settings": { "number_of_shards": 1, "number_of_replicas": 0 },
    "mappings": {
      "dynamic": "strict",
      "properties": {
        "@timestamp": { "type": "date" },
        "level":      { "type": "keyword" },
        "message":    { "type": "text" },
        "service":    { "type": "keyword" }
      }
    }
  }
}
```

**이걸 등록해도 인덱스는 0개다.** 템플릿은 "앞으로 `logs-*` 패턴으로 인덱스가 생기면
이 설정을 적용해라"는 규칙일 뿐이다.
RDB의 `CREATE TABLE`을 미리 등록해두고 이름이 패턴에 맞으면 자동 실행되는 것에 가깝다.

확인하려고 매핑을 주지 않고 인덱스만 만들어봤다.

```bash
curl -X PUT 'http://localhost:9200/logs-2026.08.06'
curl 'http://localhost:9200/logs-2026.08.06/_mapping?pretty'
# → 주지도 않은 level, message, service가 들어있다. 템플릿이 먹은 것
```

### 템플릿 API가 두 개인 이유

| API | 동작 |
|---|---|
| `_index_template` (현행, composable) | 여러 개가 매칭되면 **priority 최고 하나만** 이긴다 |
| `_template` (deprecated) | 여러 개가 `order`대로 **병합**된다 |

동작이 달라서 API를 아예 분리해놨다. 새로 짜면 `_index_template`을 쓰면 된다.

`_component_template`이라는 것도 있는데, 템플릿의 **부품**이다. 혼자서는 아무 일도 안 한다.

```
component_template "timestamp-mapping"   ← @timestamp 정의만
component_template "single-shard"        ← shards=1, replicas=0만
        ↓ composed_of 로 참조
index_template "logs"     composed_of = ["timestamp-mapping", "single-shard"]
index_template "metrics"  composed_of = ["timestamp-mapping", "single-shard"]
```

"composable"이라는 이름이 여기서 온다. 조각을 조립할 수 있어서.
템플릿이 두세 개뿐이면 굳이 안 써도 된다.

### `dynamic: "strict"`

매핑에 없는 필드가 들어오면 색인을 거부한다.

```bash
# 매핑에 없는 trace_id를 넣어본다
curl -X POST 'http://localhost:9200/logs-2026.08.06/_doc' -H 'Content-Type: application/json' -d '{
  "@timestamp":"2026-08-06T10:00:00Z","level":"ERROR","message":"boom","trace_id":"abc"
}'
# → 400 strict_dynamic_mapping_exception
```

기본값(`true`)이면 OpenSearch가 알아서 타입을 추측해 필드를 추가해준다.
편해 보이지만 **오타 필드 하나가 매핑에 영구히 박히고**(매핑은 삭제가 안 된다) 필드 수가 폭발한다.
스키마를 코드로 관리하겠다면 `strict`로 막는 게 맞다.

### `keyword` vs `text`

| | 저장 방식 | 용도 |
|---|---|---|
| `keyword` | 분석하지 않고 문자열 통째로 | 정확히 일치 필터, 집계, 정렬 |
| `text` | 분석기로 토큰화 | 전문 검색(`match`) |

로그의 `level`은 `"ERROR"`로 정확히 필터링하니 keyword, `message`는 검색하니 text다.
`text`는 집계나 정렬이 안 된다는 게 포인트다.

### 템플릿 변경은 소급 적용되지 않는다

`trace_id` 필드를 템플릿에 추가하고 확인한 결과다.

| | trace_id |
|---|---|
| 템플릿 | ✅ |
| `logs-2026.08.06` (기존 인덱스) | ❌ |
| `logs-2026.08.07` (신규 인덱스) | ✅ |

로그를 날짜별로 쪼개는 이유가 하나 더 생긴다. **내일 인덱스부터 새 스키마가 자연히 적용된다.**

거꾸로 말하면, **인덱스가 하나뿐인 데이터에는 이 탈출구가 없다.** 여기서 진짜 문제가 시작된다.

## 매핑은 바꿀 수 없다

문서 검색용 인덱스를 하나 만들면서, 조회수를 실수로 `keyword`로 잡은 상황을 가정했다.

```json
"properties": {
  "title":    { "type": "text" },
  "category": { "type": "keyword" },
  "views":    { "type": "keyword" }
}
```

평균을 내보면 거부당한다.

```json
{ "type": "illegal_argument_exception",
  "reason": "Field [views] of type [keyword] is not supported for aggregation [avg]" }
```

`"120"`, `"85"`, `"310"`이 눈에는 숫자로 보여도 OpenSearch에는 문자열 셋이다.
정렬하면 사전순으로 `"120" < "310" < "85"`가 된다.

당연히 `integer`로 고치고 싶어지는데, **고칠 수 없다.**

### 왜 못 바꾸나

Lucene이 타입별로 디스크에 다른 자료구조를 쓰기 때문이다.

```
keyword → 역색인 + doc_values(정렬된 바이트 문자열)
integer → BKD 트리 + doc_values(8바이트 정수)
```

이미 문자열로 써놓은 세그먼트가 타입 선언을 바꾼다고 숫자가 되지는 않는다.
전부 다시 읽어서 다시 쓰는 수밖에 없고, 그게 **재색인(reindex)** 이다.

| 매핑 변경 | 가능 |
|---|---|
| 필드 **추가** | ✅ |
| 필드 **타입 변경** | ❌ |
| 필드 **삭제** | ❌ (매핑에서 지워도 세그먼트에는 남는다) |

벡터 검색에 쓰는 `knn_vector`의 `dimension`도 정확히 같은 이유로 불변이다.
384차원으로 저장해둔 벡터를 768로 늘릴 방법은 없다.
임베딩 모델을 바꾸면 인덱스를 새로 만들어야 한다는 뜻이다.

> 참고로 이 상황에서 Terraform은 인덱스를 지우고 다시 만드는 계획을 세운다.
> 성공하면서 데이터가 사라진다. 그 얘기는
> [Terraform 쪽 글](/til/terraform/terraform-opensearch-schema/)에 적었다.

## alias 기반 블루/그린 재색인

그래서 실무에서 쓰는 패턴이 이거다.

```
[초기]     docs ──→ docs-v1 (views: keyword)      서비스 중
[① 생성]   docs ──→ docs-v1                        서비스 계속
                    docs-v2 (views: integer)       빈 인덱스, alias 없음
[② 재색인] docs ──→ docs-v1                        서비스 계속
                    docs-v2 ← 데이터 복사 완료
[③ 전환]   docs ──────────→ docs-v2                무중단 전환
           docs-v1 (데이터 보존)                    롤백용
```

### alias를 한 겹 끼우는 이유

```
애플리케이션:  POST /docs/_search     ← 이 이름만 안다
alias "docs" ──→ docs-v1  ⟹  docs-v2   코드 수정 0줄, 배포 0회
```

애플리케이션이 `docs-v1`이라는 실제 이름을 알고 있으면 재색인할 때마다 코드를 고치고
배포해야 한다. alias가 있으면 **인덱스 교체가 인프라 작업으로 끝난다.**

alias가 요청 시점에 실제 인덱스로 해석된다는 건 에러 메시지에서도 보인다.
`docs`로 요청했는데 에러 응답의 `"index"`에는 `docs-v1`이 찍힌다.

**순서가 중요하다.** 빈 인덱스에 alias를 먼저 붙이면 서비스가 빈 결과를 보게 된다.
인덱스 생성 → 데이터 복사 → 그 다음에 전환이다.

### ② 재색인

```bash
curl -X POST 'http://localhost:9200/_reindex?refresh=true' \
  -H 'Content-Type: application/json' -d '{
  "source": { "index": "docs-v1" },
  "dest":   { "index": "docs-v2" }
}'
```

여기서 `"120"` 문자열이 `integer` 매핑으로 들어가면서 자동 변환(coercion)된다.
변환할 수 없는 값이 섞여 있었다면 응답의 `failures`에 잡힌다.
**재색인이 데이터 검증 지점 역할도 하는 셈이다.**

이 시점까지 서비스는 아무것도 모른다. alias가 아직 v1을 가리키니 트래픽은 그대로 옛 인덱스로 간다.
블루/그린의 핵심이 이거다. **준비가 다 끝날 때까지 아무것도 바뀌지 않는다.**

### ③ 전환은 원자적으로

```bash
curl -X POST 'http://localhost:9200/_aliases' -H 'Content-Type: application/json' -d '{
  "actions": [
    { "remove": { "index": "docs-v1", "alias": "docs" } },
    { "add":    { "index": "docs-v2", "alias": "docs" } }
  ]
}'
```

`_aliases`의 `actions`는 **한 요청 안에서 원자적으로** 처리된다.
remove와 add를 따로 호출하면 그 사이 짧은 순간 alias가 아무 데도 안 붙어서
`index_not_found_exception`이 날 수 있다. 이게 이 API가 존재하는 이유다.

전환 후 아까 실패했던 그 쿼리를 **똑같은 URL로** 다시 던지면 이번엔 성공한다.

```json
{ "aggregations": { "avg_views": { "value": 171.66666666666666 } } }
```

같은 URL, 같은 쿼리, 애플리케이션 코드 변경 0줄.
v1은 데이터를 그대로 갖고 살아있으니 문제가 생기면 alias만 되돌리면 된다.
안심되면 그때 v1을 삭제한다.

## 정리

- **인덱스는 문서를 넣으면 자동 생성된다.** 그래서 템플릿으로 미리 틀을 잡아둔다
- **템플릿 ≠ 인덱스.** 등록해도 인덱스는 안 생기고, 기존 인덱스에 소급되지도 않는다
- **`dynamic: strict`로 막아두자.** 자동 필드 추가는 편하지만 되돌릴 수 없다
- **`keyword`는 필터·집계·정렬, `text`는 전문 검색.** text는 집계가 안 된다
- **매핑 타입은 불변이다.** Lucene이 타입별로 다른 자료구조를 쓰기 때문
- **alias는 인덱스 교체를 무중단으로 만든다.** 재색인의 전제 조건
- **전환은 `_aliases`의 actions로 원자적으로.** 따로 호출하면 틈이 생긴다

처음부터 스키마를 완벽하게 잡는 건 불가능하다.
그래서 **바꿀 수 없다는 걸 전제로 설계하는 것**이 결론이었다.
alias를 처음부터 끼워두고, 인덱스 이름에 버전을 붙여두면
나중에 매핑을 바꿔야 할 때 재색인 한 번으로 끝난다.
그게 없으면 애플리케이션 배포까지 엮인 큰 작업이 된다.

다음에는 `knn_vector`를 넣은 벡터 인덱스와
ISM 정책(rollover, 오래된 인덱스 자동 삭제)까지 이어서 해볼 생각이다.
