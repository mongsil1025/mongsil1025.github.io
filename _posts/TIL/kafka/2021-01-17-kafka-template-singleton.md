---
title:  "[카프카] KafkaTemplate<K,V>를 Generic 파라미터로 만들고 싱글턴으로 사용함에 대한 고민"
date: 2021-01-17
excerpt: ""
tags: [til, kafka]
classes: wide
categories: [til/kafka]
---

Effective-java 책을 읽으면서 문득 **Generic 파라미터를 받는 싱글턴은 과연 thread-safe 하나?** 라는 고민이 들었다.

왜냐하면 Kafka를 사용하는 어플리케이션에서 Generic 파라미터를 받는 싱글턴을 사용하고 있기 때문이다.

``` java
@Bean
@Primary
public KafkaTemplate<String, T> kafkaTemplate() {
    return new KafkaTemplate<>(producerFactory());
}
```

KafkaTemplate<K,V>, Key는 String, Value는 제너릭으로 받는 kafkaTemplate()을 빈으로 만들어, 어플리케이션에서 싱글턴으로 공유해서 사용하고 있다.


``` java
// 클라이언트 코드
private final IKafkaProducer<ItemBaseApiMsg> itemKafkaProducer;
private final IKafkaProducer<DisplayApiMsg> displayKafkaProducer;
private final IKafkaProducer<ItemAppeApiMsg> itemAppeKafkaProducer;
```

테스트 해보니 위 인스턴스 모두 같은 객체를 가르키고 있었다.

아차, 이렇게 되면 멀티스레드 환경에서 각 Producer가 가지는 Value 형이 다른데, 문제가 없을까? 여태까지 왜 문제가 없었지? 에러가 생기는데도 내가 놓치고 있었나? 하는 생각이 들었다.


부랴부랴 여러가지를 찾아보니 Kafka Producer Instance는 Generic 타입을 허용하고, Thread-Safe하다는 결론이 나왔다.

## Kafka Template은 One Instance로 사용하기를 권장한다.

[Kafka Template](https://docs.spring.io/spring-kafka/api/org/springframework/kafka/core/KafkaTemplate.html) 문서를 보면 KafkaTemplate은 Producer 인스턴스를 wrap하고 있으며 Producer 인스턴스는 thread-safe 를 보장한다고 나와있다.

> A template for executing high-level operations. When used with a DefaultKafkaProducerFactory, the template is thread-safe. The producer factory and KafkaProducer ensure this;

이어서 [DefaultKafkaProducerFactory](https:/docs.spring.io/spring-kafka/api/org/springframework/kafka/core/DefaultKafkaProducerFactory.html) 문서를 보니 KafkaTemplate이 사용하는 ProducerFactory는 싱글턴이며 하나의 Producer Instance를 share 한다고 나와있다.

> The ProducerFactory implementation for a singleton shared Producer instance.

즉, Producer Instance는 share 한다. 이 Producer Instance의 특징은 Key, Value Serializer를 가지며, 그 외에 카프카에 메세지를 발송하는 역할을 한다.

## Generic 파라미터를 가지는 class를 싱글턴으로 만들어도 문제가 생기지 않는다.

Kafka Producer 가 공유하는 것은 메시지를 발송, 어떤 Key, Value Serializer를 쓸지 등등을 공유하는 것이라서 싱글턴이어도 문제가 생기지 않는 것 같다.

내 생각에는, Generic Type을 변수로, 상태값을 들고 있는 것이 아니라면 싱글턴이 Generic Type을 파라미터로 가지고 있는 것이 문제가 되지 않는 것 같다.

Generic Type을 파라미터로 받아서, 해당 값의 상태를 저장하고 사용하는 경우에는 문제가 생기지 않을까? (그런 행위는 하지 말자)
