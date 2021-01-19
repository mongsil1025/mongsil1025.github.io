---
title:  "[Mongo] 몽고 Update 하는 법"
date: 2021-01-19
excerpt: ""
tags: [til, mongo]
categories: [til/mongo]
---

## db.collection.update()

`dp.collection.update(query, update, options)`

 collection에 존재하고 있는 document를 업데이트 한다. update parameter를 통해 특정 필드를 수정할 수 있고, 또한 통째로 document를 교체할수도 있다.

```
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ],
     hint:  <document|string>        // Available starting in MongoDB 4.2
   }
)
```

두 번째 파라미터인 "update" 영역에 아래와 같은 파라미터들을 넣을 수 있는데, 각각 기능하는 방식이 다르다.

|파라미터|업데이트 방식|
|------|---|
|contains only update operator expressions|Update document|
|Contains only <field1>: <value1> pairs.|Replacement document|
|Contains only the following aggregation stages|Aggregation pipeline|

## document를 통째로 update 할 경우

"update" 영역에 `<field>:<value>` 쌍을 넣으면 document 가 해당 값들로 **교체된다**

```
db.getCollection('itemShpp').update({"_id":"0000000023887"}, {"frgShppPsblYn": "Y"})
```

## document의 특정 field 만 update 할 경우

"update" 영역에 [update operator expressions](https://docs.mongodb.com/manual/reference/operator/update/#id1)를 지정해줘야 한다.

여기서는 특정 필드만 업데이트 하기위해 `$set`을 사용했다.

```
db.getCollection('itemShpp').update({"_id":"0000000023887"}, { $set : {"deliPsblYn": "N"}})
```

이 외에 여러 Update operator 가 있으니 자세한건 공식문서를 참고하자. 또한 aggregation (그룹핑 등 집계함수 이용해서 Update)하는 방법도 추가로 있다.

---
참고문서

[https://docs.mongodb.com/manual/reference/method/db.collection.update/](https://docs.mongodb.com/manual/reference/method/db.collection.update/)
