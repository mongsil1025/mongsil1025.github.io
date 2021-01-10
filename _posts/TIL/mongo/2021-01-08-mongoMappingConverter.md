---
title:  "[Mongo] 스프링 mongodb Bean 작동 방법"
date: 2021-01-08
excerpt: ""
tags: [til, mongo]
categories: [til/mongo]
---

## 스프링 mongodb Bean 자동 생성

![20210108_mongoMappingConverterBean](https://i.imgur.com/Cd6whko.png)

확실하진 않지만, maven dependency로 추가한 spring mongo에는, 위 설정으로 자동으로 `org.springframework.data.mongodb.core.convert.MappingMongoConverter` 이 Bean으로 생성되는 것 같다.

`MappingMongoConverter` 는 자바의 여러 객체들을 Mongo의 Document로 전환하기 위한 Converter이다.

> MongoConverter that uses a MappingContext to do sophisticated mapping of domain objects to Document.
> by [MappingMongoConverter](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/convert/MappingMongoConverter.html)

## MappingMongoConverter `write` 메소드
``` java
private void writeProperties(Bson bson, MongoPersistentEntity<?> entity, PersistentPropertyAccessor<?> accessor, DocumentAccessor dbObjectAccessor, @Nullable MongoPersistentProperty idProperty) {
    Iterator var6 = entity.iterator();

    while(var6.hasNext()) {
        MongoPersistentProperty prop = (MongoPersistentProperty)var6.next();
        if (!prop.equals(idProperty) && prop.isWritable()) {
            if (prop.isAssociation()) {
                this.writeAssociation(prop.getRequiredAssociation(), accessor, dbObjectAccessor);
            } else {
                Object value = accessor.getProperty(prop);
                if (value != null) {
                    if (!this.conversions.isSimpleType(value.getClass())) {
                        this.writePropertyInternal(value, dbObjectAccessor, prop);
                    } else {
                        this.writeSimpleInternal(value, bson, prop);
                    }
                }
            }
        }
    }

}
```

`Object`를 Bson 객체로 만드는 메소드이다. 값이 null이 아닌경우 에만 `this.writePropertyInternal` 를 통해 Object 객체 안에 있는 값들을 bson으로 만들어준다.

Dto 안에 값이 null인 필드는 bson으로 전환되지 않는다
{: .notice--danger}

참고문서
- https://stackoverflow.com/questions/30071464/spring-data-mongo-update-null-value-in-document-with-mappingmongoconverter

## MongoConverter 커스터마이징

이 MongoConverter를 Customizing 할 수 있다. 그 방법은, 똑같은 이름으로 `@Bean` 을 설정하는 것이다.


기본 MongoMappingConverter는 Object의 class 형을 Document에 포함한다.

``` java
public MappingMongoConverter(DbRefResolver dbRefResolver, MappingContext<? extends MongoPersistentEntity<?>, MongoPersistentProperty> mappingContext) {
    // ... 생략
    this.typeMapper = new DefaultMongoTypeMapper("_class", mappingContext, this::getWriteTarget);
}
```

`_class`를 제외하고 싶다면 `DefaultMongoTypeMapper(null)` 을 `TypeMapper`로 지정한 Bean을 하나 만들면 된다.

``` java
@Configuration
@EnableMongoAuditing
public class MongoConfig {

	@Bean
    public MappingMongoConverter mappingMongoConverter(MongoDatabaseFactory mongoDbFactory,
                                                       MongoMappingContext mongoMappingContext) {
        DbRefResolver dbRefResolver = new DefaultDbRefResolver(mongoDbFactory);
        MappingMongoConverter converter = new MappingMongoConverter(dbRefResolver, mongoMappingContext);
        converter.setTypeMapper(new DefaultMongoTypeMapper(null));
        return converter;
    }

}
```

이와 같이, 스프링 몽고에서 기본적으로 띄우는 `Conveter` 나 설정 같은 것은, 커스터마이징을 할 수 있으며, **그 방법은 같은 이름으로 된 Bean을 올려서 주입하면 된다.**
