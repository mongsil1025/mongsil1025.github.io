---
title:  "[카프카] sending LeaveGroup request to coordinator XXX due to consumer poll timeout has expired 해결법"
date: 2021-06-01
excerpt: ""
tags: [til, kafka]
classes: wide
categories: [til/kafka]
---

운영중인 컨슈머가 shutdown 되서 컨슈머 그룹의 rebalancing이 빈번하게 발생했고, 그에 따라 1번 수행되어야 할 로직이 N번 수행되는 이슈가 있었다.

간략히 로그를 보면 아래와 같다.

``` console

-- 1.  컨슈머가 너무 오래걸려서 (default로 설정된 5분을 넘겨서) 죽어버림
2021-06-01 11:03:58.893   Member consumer-XXXX sending LeaveGroup request to coordinator due to consumer poll timeout has expired. This means the time between subsequent calls to poll() was longer than the configured max.poll.interval.ms, which typically implies that the poll loop is spending too much time processing messages. You can address this either by increasing max.poll.interval.ms or by reducing the maximum size of batches returned in poll() with max.poll.records.

-- 2. 컨슈머가 죽어서 커밋이 안됨
2021-06-01 11:05:14.796   Failing OffsetCommit request since the consumer is not part of an active group

-- 3. 파티션 해제
2021-06-01 11:05:14.797   Giving away all assigned partitions as lost since generation has been reset,indicating that consumer is no longer part of the group

-- 4-1. 다시 Rejoin
2021-06-01 11:05:14.797   (Re-)joining group
2021-06-01 11:05:14.798   Join group failed with org.apache.kafka.common.errors.MemberIdRequiredException: The group member needs to have a valid member id before actually entering a consumer group  -> member id 가 없음

-- 4-2. Rejoin 성공
2021-06-01 11:08:58.965   Successfully joined group with generation 189
2021-06-01 11:08:58.965   Adding newly assigned partitions:
2021-06-01 11:08:58.966   Setting offset for partition to the committed offset

-- 5. 또 오래걸려서 다시 rebalancing
2021-06-01 11:45:44.718   Attempt to heartbeat failed since group is rebalancing
```

[Apache Kafka의 rebalancing protocol](https://medium.com/streamthoughts/apache-kafka-rebalance-protocol-or-the-magic-behind-your-streams-applications-e94baf68e4f2)
[Apache Kafka 의 설정 변수들](https://kafka.apache.org/documentation/#consumerconfigs)
[@Value를 static 변수에 담고 싶을때](https://jkpark.me/springboot/java/2020/06/04/Spring-Boot-static-%EB%B3%80%EC%88%98%EC%97%90%EC%84%9C-@value-annontation-%EC%82%AC%EC%9A%A9.html)
[빈생성 순서](https://jeong-pro.tistory.com/167)
