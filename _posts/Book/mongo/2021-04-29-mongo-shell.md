---
title:  "[MongoDB in Action] MongoDB 셸"
date: 2021-04-29
excerpt: ""
tags: [book, mongoDB-in-action]
classes: wide
categories: [book/mongoDB-in-action]
---

이 내용은 **몽고디비 인 액션 2nd Edition** 책을 읽고 개인적으로 정리한 내용입니다.
{: .notice-info}


## 코어 서버 mongod

코어 데이터베이스 서버는 `mongod`라는 실행 프로그램을 통해 구동된다. mongod 프로세스의 데이터 파일은 모두 같은 서버에 저장되고, 저장되는 위치는 `/data/db`에 기본 설정되어 있다.

`mongod`는 stand-alone 모드와 replica-set의 모드로 실행될 수 있다.

## mongo shell

MongoDB 명령어 shell은 자바스크립트에 기반한 툴이다. `mongo` 실행 파일은 shell을 로드하고 지정된 mongod 프로세스에 연결한다. 여기서 query를 날릴 수도 있으며 서버의 상태를 체크할 수도 있다.

몽고가 설치 되어 있다면 여러 커맨드로 다양한 작업을 할 수 있다.

- mongodump, mongorestore : mongodump로 백업하고, mongorestore로 복구한다.
- mongoexport, mongoimport : JSON, CSV, TSV 타입의 데이터를 export, import 할 수 있다. (대용량 데이터를 초기에 임포트하기 좋다)
- mongostat : MongoDB 시스템을 지속적으로 poll 해서 모니터링에 유용한 통계 데이터를 제공한다.
![화면 캡처 2021-04-29 222508](/assets/화면%20캡처%202021-04-29%20222508.png)
- mongotop : 각 컬렉션의 데이터를 읽고 쓰는데 걸린 시간의 양을 보여준다.
- mongoperf : MongoDB 인스턴스가 구동되고 있을 때 발생하는 디스크 오퍼레이션을 이해하는데 도움을 준다.
- mongooplog : MongoDB oplog에서 어떤일이 발생하고 있는지를 보여준다.
- bsondump : BSON 파일을 JSON을 비롯해 사람이 읽을 수 있는 형태로 변환해준다.

## 몽고 DB를 사용하는 이유

MongoDB는 관계 데이터베이스와 Key-Value 저장 시스템의 장점만을 모아서 설계되었다.

Key-Value 데이터베이스 : 시스템의 단순성으로 인해 속도가 매우 빠르고 확장도 상대적으로 용이하다
  - 대표적으로 Memcached가 있다. Memcached는 메모리에만 데이터를 저장함으로써 데이터의 지속성을 포기한 대신에 속도가 매우 빠르다. (저장 시스템으로는 적합하지 않고, 캐싱 계층이나 잡 큐와 같은 서비스에 쓰인다.)
관계 데이터베이스 : 다양한 데이터 모델과 강력한 쿼리 언어를 가지고 있다.



## 사용 예와 한계

## 최근의 MongoDB 변화
