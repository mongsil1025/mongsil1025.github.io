---
title:  "KafkaTemplate<K,V>를 Generic 파라미터로 만들고 싱글턴으로 사용함에 대한 고민"
date: 2021-01-17
excerpt: ""
tags: [book, effective-java]
classes: wide
categories: [book/effective-java]
---

Effective-java 책을 읽으면서 문득 Generic 파라미터를 받는 싱글턴은 과연 thread-safe 하나? 라는 고민이 들었다.

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
