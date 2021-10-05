---
title:  "[MongoDB in Action] MongoDB를 이용한 프로그래밍"
date: 2021-10-05
excerpt: ""
tags: [book, mongoDB-in-action]
classes: narrow
toc: true
toc_sticky: true
categories: [book/mongoDB-in-action]
---

이 내용은 **몽고디비 인 액션 2nd Edition** 책을 읽고 개인적으로 정리한 내용입니다.
{: .notice-info}

3장에서는 루비 드라이버를 통한 CRUD를 알아본다. 이러한 드라이버가 어떻게 MongoDB와 연결되는지 설명한다. 그리고 간단한 샘플 애플리케이션을 작성해본다.

## 루비를 통해 보는 몽고DB

루비가 기본적으로 설치되어 있어야 한다.

```
gem install mongo
```

위의 명령어를 실행하면 루비 드라이버를 설치할 수 있다.

몽고 드라이버를 load 하고, 데이터베이스 연결하는 스크립트를 작성하면, 스크립트로 간편하게 몽고DB를 연결할 수 있다.

실행은 `ruby connect.rb` 명령어로 실행할 수 있다.

### 루비에 대한 간단한 설명

자바스크립트에서 JSON을 사용하는 것 처럼 루비에서는 해시 데이터를 사용한다. JSON은 ":" 으로 key-value 를 구분하지만 루비는 로켓 (=>) 으로 구분한다.

루비 대화형 셸은 `irb` 이다. `irb`는 REPL(Read, Evaluable, Print, Loop) 콘솔로, 동적으로 실행되는 루비 코드를 작성하고 테스트하기에 최적의 상태로 만들 수 있게 해준다.

아까 실행했던 스크립트를 `irb -r` 로 수행하면, 커넥션, 데이터베이스, 컬렉션 객체에 직접 액세스할 수 있다.

### 삽입

아래와 같이 몽고DB에 데이터를 삽입할 수 있다.
![화면 캡처 2021-10-05 010610](/assets/화면%20캡처%202021-10-05%20010610.png)

루비 쉘을 이용해서, 자유롭게 전역변수를 설정하고 mongo 쉘의 명령어를 쓸 수 있다. (오타가 나지 않게 조심하자 😂)
```
irb(main):006:0> sunmin = {"last_name" => "jeong", "age" => 28}
=> {"last_name"=>"jeong", "age"=>28}
irb(main):010:0> puts sunmin
{"last_name"=>"jeong", "age"=>28}
=> nil
irb(main):011:0> sunmin_id = $users.insert_one(sunmin)
```

루비의 `p` 메서드를 앞에 붙여서 스크린에 프린트할 수 있다.

```
p $users.find( :age => {"$gt" => 20}).to_a
```

### 쿼리와 커서

위에서 age > 20 인 유저를 2개 추가했다. "age > 20" 조건인 쿼리를 날려보면, Mongo::Collection::View 객체에 결과값이 반환된다. 이게 커서이다.

```
irb(main):025:0> $users.find({"age" => {"gt" => 20}})
=> #<Mongo::Collection::View:0x54170960 namespace='tutorial.users' @filter={"age"=>{"gt"=>20}} @options={}>
```

루비에서는 위와 같은 커서를 통해 `each` 문으로 루핑을 하며 데이터를 출력할 수 있다. MongoDB 셸 또한 이와 같은 방법으로 `find()`를 실행하는데, 자동으로 커서를 20회 반복하여 결과를 반환한다는 점이 다르다.


### 업데이트와 삭제

```
irb(main):032:0> $users.find({"last_name" => "jeong"}).update_one({"$set" => {"home" => "SweetHome"}})
=> #<Mongo::Operation::Update::Result:0x55110020 documents=[{"n"=>1, "nModified"=>1, "ok"=>1.0}]>
```

- `update_one` : 조건에 부합하는 다큐먼트 한개만 업데이트한다.
- `update_many` : 조건에 부합하는 여러 개의 도큐먼트를 업데이트 한다.
- `delete_one` : 파라미터 없이 수행하며, 1개의 도큐먼트를 삭제한다.
- `delete_many` : 조건에 맞는 모든 도큐먼트를 삭제한다.
- `drop` : 모든 도큐먼트를 삭제한다.

### 데이터베이스 명령어

`listDatabases` 명령어를 통해 드라이버에서 어떻게 데이터베이스 명령어를 실행할 수 있는지 살펴보겠다. 이 명령어는 admin 데이터베이스에 대해 실행해야 하는 명령어 가운데 하나이다. admin 데이터베이스는 인증이 사용될 때 특별하게 취급된다. (10장에서 자세히 다룬다.)

```
irb(main):034:0> $admin_db.command({"listDatabases" => 1})
=> #<Mongo::Operation::Result:0x55079920 documents=[{"databases"=>[{"name"=>"admin", "sizeOnDisk"=>69632.0, "empty"=>false}, {"name"=>"config", "sizeOnDisk"=>110592.0, "empty"=>false}, {"name"=>"local", "sizeOnDisk"=>86016.0, "empty"=>false}, {"name"=>"tutorial", "sizeOnDisk"=>73728.0, "empty"=>false}], "totalSize"=>339968.0, "ok"=>1.0}]>
```

`listDatabases` : 현재 존재하는 모든 데이터베이스와 디스크상의 용량에 대한 정보를 루비의 해시로 보여준다.

## 드라이버 작동 원리

MongoDB 드라이버는 세 가지 주요한 기능을 수행한다.
- 모든 도큐먼트의 `_id` 필드에 디폴트로 저장되는 값인 객체 ID를 생성한다.
- 특정 언어로 표현된 도큐먼트와 MongoDB의 이진 데이터 포맷인 BSON 사이의 변환을 수행한다. (직렬화 & 역직렬화)
- MongoDB의 와이어 프로토콜을 사용해서 네트워크 소켓을 통해 데이터베이스와 통신한다.


### 객체 ID 생성

몽고DB에서 `_id` 필드는 PK이다. 개발자가 임의의 값을 넣어주는것이 가능하지만, 주어지지 않을 경우에는 MongoDB 객체 ID가 사용된다.

```
615b22e5/22c7e25178/ae6807
```

몽고DB ObjectID 는 다음과 같은 규약을 따른다. [Object Id](https://docs.mongodb.com/manual/reference/method/ObjectId/)

- 4-byte timestamp value, representing the ObjectId's creation, measured in seconds since the Unix epoch
- 5-byte random value
- 3-byte incrementing counter, initialized to a random value

도큐먼트에서는 5byte가 랜덤값이라고 명시되어 있지만, 책에서는 3byte 는 서버의 id, 나머지 2byte는 프로세스의 id라고 얘기한다.

PK 중복이 이루어지지 않게 하기 위해서 유니크한 식별자를 생성해야 하는데, 그 수단으로 타임스템프, 서버ID, 그리고 프로세스ID를 활용한 것이다.

`_id`에 타임스탬프 정보가 있기 때문에, 도큐먼트의 생성시간을 알 수 있다.

```
irb(main):035:0> id = BSON::ObjectId.from_string('615b22e522c7e25178ae6807')
=> BSON::ObjectId('615b22e522c7e25178ae6807')
irb(main):036:0> id.generation_time
=> 2021-10-04 15:51:01 UTC
```

객체의 생성시간에 대한 범위 질의를 할 때도 ObjectID를 활용할 수 있다.

```
jun_id = BSON::ObjectId.from_time(Time.utc(2021, 10, 5))
```

## 트위터 모니터링 애플리케이션 구축

책에 있는 실습코드 예제를 수행해본다.

여기서는 `gem` 을 통해 필요한 모듈을 받아온다.

```
source 'https://rubygems.org'

gem 'mongo', '1.9.2'
gem 'bson_ext', '1.9.2'
gem 'twitter', '5.8.0'
gem 'sinatra', '1.4.4'
```

sinatra 는 가벼운 웹서버를 실행시킬 수 있는 루비로 개발된 프레임워크다. sinatra를 이용하면, 간단하게 html 페이지를 만들 수 있는게 장점이다.


```
get '/' do
  if params['tag']
    selector = {:tags => params['tag']}
  else
    selector = {}
  end

  @tweets = TWEETS.find(selector).sort(["id", -1])
  erb :tweets
end

```
