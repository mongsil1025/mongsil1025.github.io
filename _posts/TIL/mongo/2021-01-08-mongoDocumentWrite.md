---
title:  "[Mongo] Dto class를 Document에 업데이트 하는 방법"
date: 2021-01-08
excerpt: ""
tags: [til, mongo]
categories: [til/mongo]
---

POJO인 Dto 객체를 Mongo Document로 update 혹은 insert 하기 위해서 Object class 를 Document class로 convert 해줘야 한다.

*Item.class*

``` java
public class Item {
	private String itemId;
  private String itemNm;
  // 무수히 많은 속성들...
}
```

위의 POJO 클래스를 mongodb에 업데이트 하려면 Update query 를 만들어서, 속성 하나하나 마다 `set`을 해줘야 한다.

- Mongo query [$set (aggregation)](https://docs.mongodb.com/manual/reference/operator/aggregation/set/#pipe._S_set)
	```
	{ $set: { <newField>: <expression>, ... } }
	```
- Spring java [Update (Spring Data MongoDB 3.1.2 API)](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/query/Update.html)
	```
	Update upd = new Update();
	upd.set(<fieldName>, <fieldValue>);
	```

That is so annoying.. Fortunately, we can convert Object class to Document one time by using `write` method in Mongo Converter.

## `Object`를 `Document`로 변환하는 코드

``` java
private final MongoTemplate mongoTemplate;

mongoTemplate.getConverter().write(itemDto, doc); // (1) itemDto의 모든 field를 document로 변환한다.
update = Update.fromDocument(doc); // (2) document를 update 한다.
```

1. `mongoTemplate.getConverter().write()`: class 객체를 bson 으로 변환
  - java Reflection 사용해서 Field의 값을 get한 뒤, **값이 null 이 아니면 bson에 추가**

2. `Update.fromDocument(doc)`: document를 update하는 update쿼리를 반환
  - `<field>:<value>` 쌍을 update문의 파라미터로 넘긴다.
	- ★ `<field>:<value>`를 넘겼으니, document 가 **교체된다**. 만약 특정 필드만 update하고자 할때는 이 메소드를 쓰면 안된다.

``` java
mongoTemplate.upsert(query, update, "item_base");
```

반환받은 update 문을 `mongoTemplate.upsert`의 두번째 파라미터로 수행하면 된다.

## 특정 필드만 몽고에 업데이트 (Update specific field to mongo document)

모든필드가 아닌 특정 필드들만 업데이트 해야된다면, `Update.fromDocument`를 사용할 수 없고 따로 메소드를 구현해야 한다.

프로세스를 정확히 알고 생각해보니, 굳이 위와 같이 스프링에서 제공하는 메소드를 쓰지 않아도 (내가 알기 쉽게) 코드를 다시 짤 수 있었다.
{: .notice--info}

``` java
for(String modCol : payload.getModCols()) {
  Field field = ReflectionUtils.findField(apiDomain.getClass(), modCol);
  field.setAccessible(true);
  Object value = field.get(apiDomain);
  if(value != null) {
    update.set(modCol, value);
  } else{
		update.unset(modCol); // 값이 null 이면 지워야 하니 unset을 한다.
	}
}
```

- `payload.getModCols()`에 업데이트 하려는 필드명을 담아두고, 이를 루핑한다.
- `modCols`를 루핑하면서, `ReflectionUtils`를 사용해서 값을 가져오고, 해당 값이 null 이 아닐 경우에만 `Update`에 set, null이면 지워야 하니 unset을 해준다.

mongo에서 제공해주는 메소드를 쓰다가, 위와 같은 코드로 바꿨을 때 성능상 이슈가 없을까 걱정했는데, mongo 메소드를 쓰던, 위와 같이 쓰던 성능 차이는 없을 것 같다.

중요한 것은, Update문에 어떤 파라미터를 넘기는지에 따라 Document를 replace 혹은 update 할지 결정되기 때문에 주의해야 한다.
