---
title:  "[Terraform] Terraform을 처음 직접 굴려보며 알게 된 것들"
date: 2026-08-06
excerpt: "로컬 OpenSearch를 대상으로 init/plan/apply를 돌려보며 배운 것. tfstate와 lock 파일의 차이, provider가 별도 프로세스인 이유, 그리고 replace가 데이터를 조용히 날리는 함정."
tags: [til, terraform]
toc: true
toc_sticky: true
categories: [til/terraform]
---

Terraform을 직접 실행해본 적이 없었다. 인프라팀이 짜놓은 코드를 읽어본 게 전부였다.

클라우드 계정 없이 손으로 익혀보려고, 로컬 Docker에 OpenSearch를 띄우고 그 안의 스키마를
Terraform으로 관리하는 실습을 처음부터 끝까지 돌려봤다.
이 글은 그중 **Terraform 자체에 대해 알게 된 것**만 정리한 것이다.

OpenSearch 쪽 이야기(인덱스 템플릿, 매핑, alias 재색인 패턴)는
[[OpenSearch] 인덱스 템플릿, 불변 매핑, 그리고 alias 재색인](/til/opensearch/opensearch-mapping-alias-reindex/)에 따로 적었다.

가장 크게 얻은 건 문법이 아니라 이 한 문장이었다.

> **Terraform이 관리하는 것과 실제로 존재하는 것은 다르다.**

## 최소 구성

```hcl
terraform {
  required_providers {
    opensearch = {
      source  = "opensearch-project/opensearch"
      version = "~> 2.3"
    }
  }
}

provider "opensearch" {
  url         = "http://localhost:9200"
  healthcheck = true   # provider 초기화 시점에 대상이 살아있는지 확인
  sniff       = false
}
```

`terraform` 블록은 "이 프로젝트가 어떤 도구를 쓸지"의 선언이고,
`provider` 블록은 "그 도구를 어디에 붙일지"의 설정이다. 둘 다 리소스가 아니라서
이 상태로 apply해도 아무것도 만들어지지 않는다.

## Terraform이 만드는 세 가지 파일

`init`과 `apply`를 돌리면 파일 세 개가 생긴다. 이름이 비슷해서 헷갈렸는데 역할이 완전히 다르다.

| | 무엇을 기록 | git |
|---|---|---|
| `.terraform/` | 다운로드한 provider 바이너리 (캐시) | ❌ |
| `.terraform.lock.hcl` | **어떤 도구 버전**을 쓰는지 + 무결성 해시 | ✅ |
| `terraform.tfstate` | **실제 인프라가 어떤 상태**인지 | ❌ (원격 백엔드로) |

### lock 파일은 인프라의 해시가 아니다

처음에 나는 lock 파일이 "현재 인프라 형상의 해시"인 줄 알았다. 아니다.

```hcl
provider "registry.terraform.io/opensearch-project/opensearch" {
  version     = "2.4.0"    # 실제로 해석된 버전
  constraints = "~> 2.3"   # 내가 코드에 쓴 제약
  hashes = [ "h1:...", "zh:..." ]   # provider 바이너리(zip)의 체크섬
}
```

내가 쓴 건 `~> 2.3`인데 실제로 설치된 건 **2.4.0**이다. 제약은 범위일 뿐이고,
`init`이 그 범위 안에서 최신을 골라 여기에 못 박는다. 이 파일이 없으면 다음 달에 2.5.0이
나왔을 때 팀원은 나와 다른 버전을 받는다. `package-lock.json`과 정확히 같은 역할이다.

`zh:` 해시가 12개나 되는 게 처음엔 이상했는데, **플랫폼별 zip의 해시를 전부 적어두기 때문**이었다.
내 맥은 그중 하나만 받았지만, 팀원이 Linux CI에서 `init` 해도 검증이 되어야 하니까.

인프라의 현재 상태를 기록하는 건 lock 파일이 아니라 `tfstate`다. 둘은 역할이 전혀 겹치지 않는다.

### tfstate의 `resources: []`가 알려준 것

리소스를 하나도 선언하지 않은 상태로 apply를 돌리면 tfstate가 이렇게 생긴다.

```json
{
  "version": 4,
  "serial": 1,
  "lineage": "5b7a848c-...",
  "resources": []
}
```

이때 `terraform plan`은 `No changes.`를 낸다. 나는 이걸 "클러스터 상태와 코드가 일치해서"라고
이해했는데, 틀렸다. **관리 대상이 0개라서** 나오는 결과다.

plan이 하는 일은 이거다.

```
코드에 선언된 리소스 목록  ↔  tfstate에 기록된 리소스 목록
        (0개)                        (0개)
```

이 시점에 Terraform은 **대상 시스템을 들여다보지도 않았다.** 서버에 뭐가 있든 상관없다.
선언된 리소스가 있어야 비로소 그 리소스에 한해 실제 상태를 조회(refresh)한다.

`lineage`도 재미있었다. state의 고유 ID인데, 원격 백엔드에서 두 사람이 각자 다른 state를
만들었을 때 "이건 같은 state의 다른 버전이 아니라 아예 남남이다"를 감지해서 거부하는 용도다.
`serial`(apply할 때마다 +1)과 조합해서 남의 인프라를 덮어쓰는 사고를 막는다.

`tfstate`를 git에 올리지 않는 이유도 명확했다. 리소스의 모든 속성이 평문으로 들어가서
DB 비밀번호나 API 키가 그대로 노출되고, 두 사람이 각자 커밋하면 나는 충돌은 손으로 풀 수 없다.

## core와 provider는 별도 프로세스다

에러 메시지가 이상하게 두 층으로 나와서 찾아보다 알게 된 건데,
`terraform apply` 중에는 프로세스가 **두 개** 돈다.

```
[프로세스 1] terraform core
    - 코드 파싱, 의존성 그래프, diff 계산, state 읽기/쓰기
    - 대상 시스템이 뭔지 모른다. HTTP 요청을 하나도 안 보낸다.
         ↕ gRPC
[프로세스 2] terraform-provider-opensearch
    - .terraform/ 안의 그 바이너리를 core가 띄운다
    - 실제로 localhost:9200에 HTTP를 치는 건 얘다
```

정해진 gRPC 인터페이스(`GetSchema`, `PlanResourceChange`, `ApplyResourceChange`)로 대화한다.
"인터페이스가 정해져 있다"가 무슨 뜻인지는
[[gRPC] gRPC 인터페이스란?](/til/grpc/what-is-grpc-interface/)에 따로 정리했다.

```
core → provider : PlanResourceChange(현재=state 값, 원하는=코드 값)
provider → core : "결과는 이렇게 될 거다"        ← plan에 찍히는 diff
core → provider : ApplyResourceChange(...)
                  ↓ 여기서 provider가 실제 API를 호출한다
provider → core : "만들었다. id와 최종 상태는 이렇다"
core            : 받은 걸 tfstate에 기록
```

그래서 에러가 이렇게 섞여 나온다.

```
Error: elastic: Error 400 (Bad Request): ...     ← provider가 받은 서버 응답
  with opensearch_index.docs_v1,                 ← core가 붙인 위치 정보
  on main.tf line 30
```

앞줄은 provider가 전달한 서버 응답, 뒷줄은 core가 붙인 코드 위치다.
두 프로세스의 말이 한 화면에 섞여 있는 거였다.

이 구조라서 얻는 게 있다. provider를 본체와 독립적으로 배포할 수 있고(그래서 provider가
수천 개나 되어도 각자 자기 속도로 릴리스한다), provider가 패닉으로 죽어도 core는 살아서
에러를 정리해준다. lock 파일에 바이너리 해시를 박을 수 있는 것도 이 구조라서 가능하다.
라이브러리로 링크되어 있었다면 어느 것도 불가능했다.

## plan을 읽는 법

### `jsonencode()`를 쓰면 diff가 읽힌다

provider가 JSON 문자열을 받는 필드가 있었다. heredoc(`<<EOF`)으로 써도 되지만,
`jsonencode()`로 감싸면 Terraform이 JSON 구조를 이해해서 **필드 단위 diff**를 보여준다.

```
~ body = jsonencode(
    ~ {
        ~ properties = {
            + trace_id = { + type = "keyword" }
              # (4 unchanged attributes hidden)
```

heredoc이었다면 긴 문자열 두 개가 통째로 찍혔을 거다.

### 서버가 정규화해서 돌려주면 diff가 안 사라진다

plan을 돌리다가 이런 게 계속 붙어 나왔다.

```
~ settings = {
    - index              = { - number_of_replicas = "0" ... }
    + number_of_replicas = 0
```

내가 코드에 `settings.number_of_shards`로 썼는데, 서버는 `settings.index.number_of_shards`
형태로 정규화해서 돌려준다. **값은 같고 표현만 다른데 Terraform은 차이로 인식한다.**
그래서 아무것도 안 바꿔도 매번 diff가 뜬다.

provider가 응답을 정규화해주지 않아서 생기는 현상인데, 실무에서 "plan이 계속 깨끗해지지 않는"
이유 중 하나라고 한다. plan 출력이 항상 신뢰할 수 있는 신호는 아니라는 걸 처음 체감했다.

## 가장 크게 배운 것 — replace는 데이터 손실이다

인덱스 하나를 만들어두고, 필드 타입을 잘못 잡은 걸 고치려고 코드에서 한 글자 바꿨다.
(왜 애초에 못 바꾸는지는 [OpenSearch 쪽 글](/til/opensearch/opensearch-mapping-alias-reindex/)에 적었다)

나는 apply가 실패할 거라고 예상했다. 서버가 400을 뱉을 테니까. 실제 plan은 이랬다.

```
-/+ destroy and then create replacement

  # opensearch_index.docs_v1 must be replaced
      ~ mappings = jsonencode(
          ~ { ~ views = { ~ type = "keyword" -> "integer" } }
        ) # forces replacement

Plan: 1 to add, 0 to change, 1 to destroy.
```

**apply하면 성공한다. 그리고 데이터가 전부 사라진다.**

provider가 이 속성을 **ForceNew**로 선언해뒀다. "이건 수정할 수 없는 속성"이라는 걸
provider 작성자가 이미 알고 있어서, 값이 바뀌면 리소스를 `DELETE` 후 `PUT`하는 계획을 세운다.
게다가 실습 편의로 넣어둔 `force_destroy = true` 때문에 삭제가 막히지도 않는다.

```
1. DELETE /docs-v1   ← 문서 전부 삭제
2. PUT /docs-v1      ← 빈 인덱스로 재생성
   결과: Apply complete!  초록불. 데이터는 없음.
```

에러가 났다면 오히려 안전했을 거다. apply가 실패하고 데이터는 그대로 남았을 테니까.
**성공하면서 데이터만 조용히 사라지는 쪽이 훨씬 위험하다.**

`-/+`와 `1 to destroy`를 못 보고 `yes`를 치면 프로덕션 데이터가 날아가는 시나리오다.
이건 특정 provider의 문제가 아니다. `aws_db_instance`처럼 상태를 가진 리소스는
어디서나 **replace = 데이터 손실**이다.

plan 출력에서 `destroy`와 `replace`는 반드시 눈으로 확인해야 하는 이유를 몸으로 배웠다.
`Plan: N to add, N to change, N to destroy` 한 줄만이라도 매번 읽어야 한다.

## 선언적 도구가 잘 못 하는 것

데이터를 살리면서 타입을 바꾸려면 새 리소스를 만들고 데이터를 옮긴 다음 전환해야 한다.
그 과정에서 Terraform의 경계가 두 번 보였다.

**첫째, 데이터 이동은 Terraform이 표현할 대상이 아니다.**
"최종 상태가 이래야 한다"로 쓸 수 없는, 일회성 작업이기 때문이다. 서버 API를 직접 호출했다.

**둘째, 전환이 원자적이지 않다.**
alias를 A에서 떼고 B에 붙이는 걸 Terraform은 **API 호출 두 번**으로 처리한다.
그 사이 짧은 순간 아무 데도 안 붙은 상태가 생긴다.
서버가 제공하는 원자적 액션을 쓰면 한 요청으로 끝나지만, 그건 Terraform 밖의 일이다.

> **Terraform은 "최종 상태가 이래야 한다"를 잘 표현하지만,
> "어떤 순서로, 얼마나 원자적으로 바꿀 것인가"는 잘 표현하지 못한다.**

그래서 정의는 Terraform으로 관리하고 전환하는 순간만 스크립트나 파이프라인으로 빼는 구성이
흔하다고 한다. 예전에 읽었던 인프라 코드에 왜 Terraform 옆에 스크립트가 같이 있었는지
이제 이해가 된다.

## 정리

- **`.terraform.lock.hcl`은 도구 버전의 해시, `tfstate`는 인프라 상태.** 완전히 다른 파일이다
- **`No changes`는 "일치한다"가 아니라 "관리 대상이 없다"일 수 있다**
- **core와 provider는 gRPC로 대화하는 별도 프로세스다.** 에러 메시지가 두 층인 이유
- **`jsonencode()`를 쓰면 diff가 필드 단위로 읽힌다**
- **서버가 정규화해서 돌려주면 diff가 영원히 안 사라진다.** plan이 항상 깨끗해지진 않는다
- **replace는 stateful 리소스에서 데이터 손실이다.** 실패보다 성공이 위험할 수 있다
- **순서와 원자성은 선언적 도구의 약점이다.** 그 부분은 여전히 명령형으로 남는다

가장 인상 깊었던 건 실습 마지막에 확인한 사실이다.
`curl`로 직접 만든 인덱스들은 `terraform state list`에 없다.
그래서 `terraform destroy`를 해도 **지워지지 않는다.**

Terraform이 관리하는 세계와 실제로 존재하는 세계는 다르다.
plan이 깨끗하다고 해서 실제 시스템이 내가 생각한 모습인 건 아니다.
