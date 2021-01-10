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

mongoTemplate.getConverter().write(itemDto, doc);
update = Update.fromDocument(doc);
```

**코드개요**

- class 객체를 bson 으로 변환
  - java Reflection 사용해서 Field의 값을 get한 뒤, 값이 null 이 아니면 bson에 추가
  - field 개수만큼 루핑
- bson에 있는 data 값들을 루핑
  - 루프 돌면서 `update.keysToUpdate`에 `add`

**내부로직**

- `mongoTemplate.getConverter().write` 메소드 안을 보면, source class를 bson type인 document로 만들어주는 로직을 볼 수 있다.
	- 값이 `null` 이라면 pass 한다. (해당 값은 update/insert 하지 않는다.)

- `Update.fromDocument(doc)` : bson의 데이터들을 Update.set에 모두 추가해준다.

``` java
public static Update fromDocument(Document object, String... exclude) {
    Update update = new Update();
    List<String> excludeList = Arrays.asList(exclude);
    Iterator var4 = object.keySet().iterator();

    while(true) {
        while(true) {
            String key;
            do {
                if (!var4.hasNext()) {
                    return update;
                }

                key = (String)var4.next();
            } while(excludeList.contains(key));

            Object value = object.get(key);
            update.modifierOps.put(key, value);
            if (isKeyword(key) && value instanceof Document) {
                update.keysToUpdate.addAll(((Document)value).keySet()); // add field to update keys
            } else {
                update.keysToUpdate.add(key);
            }
        }
    }
}
```

그리고 해당 `Update` 문을 mongodb에 update 메소드의 파라미터로 넘겨주면 된다.

``` java
mongoTemplate.upsert(query, update, "item_base");
```

## 특정 필드만 몽고에 업데이트 (Update specific field to mongo document)

``` java
private final MongoTemplate mongoTemplate;

mongoTemplate.getConverter().write(itemDto, doc);
update = Update.fromDocument(doc);
```

`Update.fromDocument` 는 위에서 살펴봤듯이 내부적으로 **모든 필드를 Update에 추가한다.**


모든필드가 아닌 특정 필드들만 업데이트 하고자할 경우, `Update.fromDocument`를 사용할 수 없고 따로 메소드를 구현해야 한다.


프로세스를 정확히 알고 생각해보니, 굳이 위와 같이 스프링에서 제공하는 메소드를 쓰지 않아도 (내가 알기 쉽게) 코드를 다시 짤 수 있었다.
{: .notice--info}

``` java
for(String modCol : payload.getModCols()) {
  Field field = ReflectionUtils.findField(apiDomain.getClass(), modCol);
  field.setAccessible(true);
  Object value = field.get(apiDomain);
  if(value != null) {
    update.set(modCol, value);
  }
}
```

- `payload.getModCols()`에 업데이트 하려는 필드를 담아두고, 이를 루핑한다.
- `modCols`를 루핑하면서, `ReflectionUtils`를 사용해서 값을 가져오고, 해당 값이 null 이 아닐 경우에만 `Update`에 add

mongo에서 제공해주는 메소드를 쓰다가, 위와 같은 코드로 바꿨을 때 성능상 이슈가 없을까 걱정했는데, mongo 메소드를 쓰던, 위와 같이 쓰던 성능 차이는 없을 것 같다.
