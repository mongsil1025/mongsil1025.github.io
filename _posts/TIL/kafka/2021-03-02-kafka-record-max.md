---
title:  "[카프카] RecordTooLargeException 예외 해결법"
date: 2021-03-02
excerpt: ""
tags: [til, kafka]
classes: wide
categories: [til/kafka]
---

## 오류전문

`org.apache.kafka.common.errors.RecordTooLargeException: The message is 1382477 bytes when serialized which is larger than 1048576`


<div>
2021-02-25 16:37:32.004 ERROR 1 --- [tainer#10-0-C-1] c.s.a.c.c.c.BasicConsumerErrorHandler    : [Consumer Error] ----- Listener method 'public void com.ssg.api.dataSync.consumer.ItemOptnApiDataSyncConsumer.syncTableItemListener(java.lang.String) throws java.lang.Exception' threw exception; nested exception is org.springframework.kafka.KafkaException: Send failed; nested exception is org.apache.kafka.common.errors.RecordTooLargeException: The message is 1382477 bytes when serialized which is larger than 1048576, which is the value of the max.request.size configuration.; nested exception is org.springframework.kafka.KafkaException: Send failed; nested exception is org.apache.kafka.common.errors.RecordTooLargeException: The message is 1382477 bytes when serialized which is larger than 1048576, which is the value of the max.request.size configuration.
</div>


kafka로 보내는 메시지의 크기가 default 크기인 1048576 byte 보다 크면 발생하는 오류이다.

해당 오류를 접할 때 아래 세 가지를 체크한다.

- 브로커 : `message.max.bytes` - this is the largest size of the message that can be recieved by the **broker from a producer**
- 프로듀서 : `max.request.size` - max size of the message that **producer can send**
  아래와 같이 사이즈를 적당히 늘려준다.
  ``` java
  configProps.put(ProducerConfig.MAX_REQUEST_SIZE_CONFIG, 31_457_280); // DF는 10Mb , MAX는 50MB .. 30MB로 일단 해본다.
  ```
- 컨슈머 : `max.partition.fetch.bytes` - max size of the message that **consumer can receive**
  받을 컨슈머도 받는 max 크기를 적당히 조절해준다.
  ``` java
  props.put(ConsumerConfig.FETCH_MAX_BYTES_CONFIG, 52_428_800);
  ```


## 참고 문서
https://stackoverflow.com/questions/21020347/how-can-i-send-large-messages-with-kafka-over-15mb
