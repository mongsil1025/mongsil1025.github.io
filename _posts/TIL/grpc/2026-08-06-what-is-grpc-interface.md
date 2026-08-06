---
title:  "[gRPC] gRPC 인터페이스란?"
date: 2026-08-06
excerpt: "Terraform이 provider와 gRPC로 통신한다는 걸 알게 되면서 정리한 내용. 인터페이스를 먼저 정의한다는 게 무슨 뜻인지, REST와 뭐가 다른지."
tags: [til, grpc]
toc: true
toc_sticky: true
categories: [til/grpc]
---

Terraform 실습을 하다가
"[core와 provider가 gRPC로 통신한다](/til/terraform/terraform-opensearch-schema/)"는 걸 알게 됐다.
gRPC라는 이름은 알고 있었지만 "인터페이스가 정해져 있다"는 표현이 정확히 무슨 뜻인지
설명할 수 없어서 정리해봤다.

## 한 줄 요약

> **gRPC는 다른 프로세스의 함수를 내 프로세스의 함수처럼 호출하게 해주는 것이고,
> 인터페이스는 "어떤 함수가 있고 인자와 반환값이 뭔지"를 미리 못 박아둔 계약이다.**

## RPC부터

gRPC의 RPC는 **Remote Procedure Call**, 원격 프로시저 호출이다.

원래 함수 호출은 같은 프로세스 안에서 일어난다.

```java
User user = userService.getUser(123);
```

RPC의 아이디어는 이 코드를 그대로 두고 `userService`만 **다른 서버에 있는 것**으로
바꾸는 거다. 호출하는 쪽 코드는 로컬 호출과 똑같이 생겼는데, 실제로는 네트워크를 타고
저쪽 프로세스의 함수가 실행되고 결과가 돌아온다.

중간에 필요한 일(인자 직렬화, 네트워크 전송, 응답 역직렬화, 에러 전달)은
전부 생성된 코드가 대신 해준다. 그래서 개발자 눈에는 그냥 함수 호출로 보인다.

**gRPC는 구글이 만든 RPC 구현체**다. 앞의 g는 공식적으로는 그때그때 다른 뜻으로 붙인다고 한다.

## "인터페이스가 정해져 있다"는 것

여기가 핵심이다. gRPC에서는 **코드를 짜기 전에 인터페이스를 먼저 파일로 정의한다.**
`.proto` 확장자를 쓰고, 이 언어를 **Protocol Buffers(protobuf)** 라고 부른다.

```protobuf
syntax = "proto3";

// 서비스 = 인터페이스. 여기 적힌 것만 호출할 수 있다
service UserService {
  rpc GetUser (GetUserRequest) returns (User);
  rpc ListUsers (ListUsersRequest) returns (stream User);
}

// 메시지 = 주고받는 데이터의 구조
message GetUserRequest {
  int64 id = 1;
}

message User {
  int64  id    = 1;
  string name  = 2;
  string email = 3;
}
```

읽는 법은 이렇다.

- `service UserService` — 인터페이스 하나
- `rpc GetUser (GetUserRequest) returns (User)` — 메서드 하나. **인자 타입과 반환 타입이 고정**
- `message` — 주고받는 구조체
- `= 1`, `= 2` — 필드 번호. 이름이 아니라 **이 번호로 직렬화된다**

### 필드 번호가 이름보다 중요하다

처음에 `= 1`이 기본값인 줄 알았는데 아니었다. **필드의 고유 번호**다.

JSON은 필드 이름을 문자열로 같이 보낸다. `{"name": "sunmin"}`처럼.
protobuf는 이름을 안 보내고 번호만 보낸다. 그래서 훨씬 작다.

그리고 이게 **하위 호환성의 핵심**이다.

```protobuf
message User {
  int64  id    = 1;
  string name  = 2;
  string email = 3;
  string phone = 4;   // 새로 추가 — 옛날 클라이언트는 4번을 모르니 그냥 무시한다
}
```

필드를 **추가**하는 건 안전하다. 옛 클라이언트는 모르는 번호를 무시하고,
새 서버는 옛 클라이언트가 안 보낸 필드를 기본값으로 채운다.

반대로 **번호를 재사용하거나 타입을 바꾸는 건 위험하다.**
2번이 `string name`이었는데 나중에 `int64 age`로 바꾸면,
옛 클라이언트가 보낸 이름 문자열을 새 서버가 나이로 읽는다.
그래서 필드를 지울 때는 번호를 `reserved`로 막아둔다.

```protobuf
message User {
  reserved 2;              // 이 번호는 다시 쓰지 않는다
  reserved "name";
}
```

### 코드는 생성된다

`.proto` 파일을 컴파일하면 언어별 코드가 나온다.

```
user.proto ──protoc──┬──→ Java   (UserServiceGrpc.java, User.java)
                     ├──→ Go     (user.pb.go)
                     ├──→ Python (user_pb2.py)
                     └──→ ...
```

서버는 생성된 인터페이스를 **구현**하고, 클라이언트는 생성된 **스텁(stub)** 을 호출한다.

```java
// 클라이언트 — 로컬 함수처럼 보인다
User user = stub.getUser(GetUserRequest.newBuilder().setId(123).build());
```

**여기서 "인터페이스가 정해져 있다"의 의미가 나온다.**
`.proto`에 없는 메서드는 호출할 수 없다. 오타를 내면 **런타임이 아니라 컴파일 타임에 잡힌다.**
REST에서 URL 경로를 잘못 쳐서 404를 받는 것과 대비된다.

## REST와 뭐가 다른가

| | REST (JSON/HTTP) | gRPC |
|---|---|---|
| 계약 | 문서나 OpenAPI 스펙 (선택) | `.proto` 파일 **필수** |
| 데이터 형식 | JSON (텍스트) | protobuf (바이너리) |
| 전송 | 주로 HTTP/1.1 | **HTTP/2** |
| 코드 생성 | 선택 | 기본 |
| 타입 체크 | 런타임 | **컴파일 타임** |
| 브라우저 | 그냥 됨 | 프록시 필요 (grpc-web) |
| 사람이 읽기 | curl로 바로 | 도구 필요 (grpcurl) |

### HTTP/2를 쓴다는 것

gRPC가 HTTP/2 위에서 도는 건 스트리밍 때문이다.
HTTP/1.1은 요청 하나에 응답 하나인데, HTTP/2는 하나의 연결에서
여러 메시지를 양방향으로 주고받을 수 있다(멀티플렉싱).

그래서 gRPC는 네 가지 호출 방식을 지원한다.

```protobuf
rpc GetUser (Request) returns (Response);                 // 1. 단항 — REST와 비슷
rpc ListUsers (Request) returns (stream Response);        // 2. 서버 스트리밍
rpc Upload (stream Request) returns (Response);           // 3. 클라이언트 스트리밍
rpc Chat (stream Request) returns (stream Response);      // 4. 양방향 스트리밍
```

2번은 결과가 많을 때 유용하다. 10만 건을 한 번에 말아서 주는 대신 흘려보낼 수 있다.
4번은 채팅이나 실시간 동기화에 쓴다.

### 언제 쓰나

**gRPC가 맞는 곳**은 내부 서비스 간 통신이다.
호출량이 많고, 양쪽 다 내가 관리하고, 스키마를 강제하고 싶은 경우.
바이너리라 payload가 작고 파싱이 빠르다.

**REST가 맞는 곳**은 외부 공개 API다.
브라우저에서 바로 호출해야 하고, 아무나 curl로 찔러볼 수 있어야 하고,
클라이언트가 누군지 모르는 경우.

디버깅 난이도가 실제로 다르다. REST는 curl 한 줄이면 되는데
gRPC는 `grpcurl` 같은 도구가 필요하고, 응답이 바이너리라 tcpdump로 봐도 안 읽힌다.

## 다시 Terraform 얘기

이걸 알고 나니 Terraform 구조가 이해됐다.

```
[프로세스 1] terraform core
    - 코드 파싱, 그래프 계산, state 관리
         ↕ gRPC
[프로세스 2] terraform-provider-opensearch (별도 바이너리)
    - 실제 API 호출
```

Terraform은 **plugin 프로토콜을 `.proto`로 정의**해뒀다.
`GetSchema`, `PlanResourceChange`, `ApplyResourceChange` 같은 메서드가 거기 적혀 있다.

```
core → provider : PlanResourceChange(현재 상태, 원하는 상태)
provider → core : "결과는 이렇게 될 거다"
```

여기서 "인터페이스가 정해져 있다"가 왜 중요한지가 드러난다.

- **언어 무관** — provider는 이 인터페이스만 구현하면 된다.
  HashiCorp가 provider 하나하나를 알 필요가 없다
- **독립 배포** — provider 수천 개가 본체와 무관하게 각자 릴리스한다.
  프로토콜 버전만 맞으면 된다
- **격리** — 별도 프로세스라 provider가 패닉으로 죽어도 core는 살아남는다
- **버전 고정** — 별도 바이너리니까 lock 파일에 해시를 박을 수 있다

라이브러리로 링크되어 있었다면 어느 것도 불가능했다.
**프로세스를 나누고 그 사이에 명시적 계약을 두는 것**이 이 설계의 전부였다.

## 정리

- **RPC는 원격 함수 호출을 로컬 함수처럼 보이게 하는 것**
- **인터페이스는 `.proto`에 미리 정의한 계약이다.** 메서드 이름, 인자 타입, 반환 타입
- **필드 번호가 이름보다 중요하다.** 추가는 안전, 번호 재사용은 위험
- **코드는 생성된다.** 그래서 오타가 컴파일 타임에 잡힌다
- **HTTP/2 덕분에 스트리밍이 된다.** 단항 외에 세 가지 방식이 더 있다
- **내부 서비스 간 통신에 강하고, 외부 공개 API에는 REST가 낫다**

플러그인 구조를 설계할 때 gRPC를 쓰는 이유가 성능만은 아니었다.
**계약을 파일로 못 박아두면 양쪽을 완전히 분리해서 각자 배포할 수 있다**는 게 더 컸다.
Terraform이 provider 수천 개를 관리하지 않고도 생태계를 굴리는 방식이 이거였다.
