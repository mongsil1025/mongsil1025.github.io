---
title:  "[Mongo]"
date: 2021-01-08
excerpt: ""
tags: [til, mongo]
categories: [til/mongo]
---

## POJO class를 Mongo Document로 변환하는 방법

Item.class

``` java
public class Item {
	private String itemId;
  private String itemNm;
  // 무수히 많은 속성들...
}
```

위의 POJO 클래스를 mongodb에 업데이트 하려면 Document로 변환해야한다.

``` java
private final MongoTemplate mongoTemplate;

mongoTemplate.getConverter().write(itemDto, doc);
update = Update.fromDocument(doc);
```

`mongoTemplate.getConverter().write` 메소드 안을 보면, source class를 bson type인 document로 만들어주는 로직을 볼 수 있다.

`Update.fromDocument(doc)` : bson의 데이터들을 Update.set을 모두 추가해준다.

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
                update.keysToUpdate.addAll(((Document)value).keySet());
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

내부 로직을 확인해보니까, class 객체를 mongodb에 update 하는 로직은 크게 다음과 같이 표현할 수 있다.
- class 객체를 bson 으로 변환
  - java Reflection 사용해서 Field의 값을 get한 뒤, 값이 null 이 아니면 bson에 추가
  - field 개수만큼 루핑
- bson에 있는 data 값들을 루핑
  - 루프 돌면서 `update.keysToUpdate`에 `add`

내부 로직을 정확히 알고보니까, 내가 알기쉽게 코드를 다시 짤 수 있었다.

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

`modCols`를 루핑하면서, `ReflectionUtils`를 사용해서 값을 가져오고, 해당 값이 null 이 아닐 경우에만 `Update`에 add 하는 것이다. mongo에서 제공해주는 메소드를 쓰다가, 위와 같은 코드로 바꿨을때 성능상 이슈가 없을까 걱정했는데, mongo 메소드를 쓰던, 위와 같이 쓰던 성능 차이는 없을 것 같다.
