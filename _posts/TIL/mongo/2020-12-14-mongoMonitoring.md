---
title:  "[Mongo] 몽고 db 모니터링 방법 (서버 모니터링 부터 그라파나 연동까지)"
date: 2021-01-05
excerpt: ""
tags: [til, mongo]
categories: [til/mongo]
---

회사에서 Mongo DB를 모니터링하기 위해서 여러 Tool을 찾아보았고 그 정보를 정리했습니다.
{: .notice--info}

## 1. mongod 내부 메서드 실행

- `mongo.exe`를 실행
- `db.enableFreeMonitoring()`를 실행
- mongodb 모니터링 URL 확인

![화면 캡처 2021-01-13 225544](https://i.imgur.com/La2MyYU.png)

FreeMonitoring 관련해서는 [Free Cloud monitoring](https://docs.mongodb.com/manual/administration/free-monitoring/) 에서 확인 가능하다.

> By default, you can enable/disable free monitoring during runtime using `db.enableFreeMonitoring()` and `db.disableFreeMonitoring()`.


**Warn:** Mongo 4.0 이상에서만 가능하다.
{: .notice--warning}

return된 url을 브라우저에서 실행해보면 간단한 모니터링 대쉬보드가 나온다.

![FireShot Capture 006 - MongoDB Free Monitoring - cloud.mongodb.com](https://i.imgur.com/wpMgDQA.png)

주의할 것은 **실시간이 아닌, 24시간 이내 데이터가 추출되고 24시간 후에는 해당 데이터가 expire 된다**.


아래와 같은 정보를 얻을 수 있다.
  - Operation Execution Times
  - Memory Usage
  - CPU Usage
  - Operation Counts

위처럼 대쉬보드말고 서버에서 아래 명령어를 통해 서버 정보, db 상태 등을 직접 볼 수 있다.
``` console
> db.currentOp
> db.serverStatus()
> db.dbStats()
```

[참고문서]
https://docs.mongodb.com/manual/administration/monitoring/


## 2. Mongo DB에서 기본 제공하는 CLI 명령어

- [mongotop](https://docs.mongodb.com/database-tools/mongotop/#bin.mongotop)을 이용한 각 collection별 read write 속도 모니터링

```console
./mongotop --host localhost --port 27017 -u user -p 'password' --authenticationDatabase admin
```

- [mongostat](https://docs.mongodb.com/database-tools/mongostat/#bin.mongostat)을 이용한 query 실행 모니터링

```console
./mongostat --host localhost --port 27017 -u user -p 'password' --authenticationDatabase admin
```

[참고문서]
https://docs.ncloud.com/ko/database/database-10-6.html


## 3. Mongodb exporter로 데이터 metrics 추출 (프로메테우스에 등록할 metrics)

위에서 언급한 data를 모두 metrics로 추출한다. `db.serverStatus()` 로 얻을 수있는 정보는 모두 포함된다.

> Much of the output of serverStatus is also displayed dynamically by mongostat. See the mongostat command for more information.

Mongodb exporter 는 두 가지가 있다.
- David Cuadrado가 만든 exporter [David Cuadrado가 만든 exporter](https://github.com/dcu/mongodb_exporter)
- Percona가 만든 exporter

### 3-1. David Cuadrado가 만든 exporter

dcu 보다 percona 버전으로 사용하자. dcu는 더이상 업데이트 되지 않아서, 높은 버전의 몽고일 경우 percona exporter로 깔아야지 정상작동한다.
{: .notice--danger}

#### exporter 설치방법
[해당 링크](https://devconnected.com/mongodb-monitoring-with-grafana-prometheus/0)에서 자세히 설명되어 있다.

1. 둘 중 하나로, mongodb-exporter 다운로드
- go get -u github.com/percona/mongodb_exporter
- $ wget https://github.com/percona/mongodb_exporter/releases/download/v0.7.1/mongodb_exporter-0.7.1.linux-amd64.tar.gz

2. 다운받은 path 로 가서
make build
make docker

3. docker image 실행
``` console
docker run --name mongodb_exporter --rm -d -p 9001:9001 mongodb_exporter --mongodb.uri=mongodb://mongodb_exporter:mongodb_exporter@10.203.7.214:27017
```
4. metric 수집되는지 확인
``` console
curl localhost:9001/metrics
```
5. prometheus 에 해당 metrics url 등록
6. 그라파나 dashboad import

**Collect below metrics**
- MongoDB Server Status metrics (cursors, operations, indexes, storage, etc)
- MongoDB Replica Set metrics (members, ping, replication lag, etc)
- MongoDB Replication Oplog metrics (size, length in time, etc)
- MongoDB Sharding metrics (shards, chunks, db/collections, balancer operations) --> dcu로는 안되고 percona exporter로는 된다고 한다. [percona exporter](https://github.com/percona/mongodb_exporter)
- MongoDB RocksDB storage-engine metrics (levels, compactions, cache usage, i/o rates, etc)
- MongoDB WiredTiger storage-engine metrics (cache, blockmanger, tickets, etc)
- MongoDB Top Metrics per collection (writeLock, readLock, query, etc*)

**Available groups of data**

Name     | Description
---------|------------
asserts | The asserts group reports the number of asserts on the database. While assert errors are typically uncommon, if there are non-zero values for the asserts, you should check the log file for the mongod process for more information. In many cases these errors are trivial, but are worth investigating.
durability | The durability group contains data regarding the mongod's journaling-related operations and performance. mongod must be running with journaling for these data to appear in the output of "serverStatus".
background_flushing | mongod periodically flushes writes to disk. In the default configuration, this happens every 60 seconds. The background_flushing group contains data regarding these operations. Consider these values if you have concerns about write performance and journaling.
connections | The connections groups contains data regarding the current status of incoming connections and availability of the database server. Use these values to assess the current load and capacity requirements of the server.
extra_info | The extra_info group holds data collected by the mongod instance about the underlying system. Your system may only report a subset of these fields.
global_lock | The global_lock group contains information regarding the database’s current lock state, historical lock status, current operation queue, and the number of active clients.
index_counters | The index_counters groupp reports information regarding the state and use of indexes in MongoDB.
network | The network group contains data regarding MongoDB’s network use.
op_counters | The op_counters group provides an overview of database operations by type and makes it possible to analyze the load on the database in more granular manner. These numbers will grow over time and in response to database use. Analyze these values over time to track database utilization.
op_counters_repl | The op_counters_repl group, similar to the op_counters data structure, provides an overview of database replication operations by type and makes it possible to analyze the load on the replica in more granular manner. These values only appear when the current host has replication enabled. These values will differ from the opcounters values because of how MongoDB serializes operations during replication. These numbers will grow over time in response to database use. Analyze these values over time to track database utilization.
memory | The memory group holds information regarding the target system architecture of mongod and current memory use
locks | The locks group containsdata that provides a granular report on MongoDB database-level lock use
metrics | The metrics group holds a number of statistics that reflect the current use and state of a running mongod instance.
cursors | The cursors group contains data regarding cursor state and use. This group is disabled by default because it is deprecated in mongodb >= 2.6.
top | The top group provides an overview of database operations by type for each database collections and makes it possible to analyze the load on the database in more granular manner. These numbers will grow over time and in response to database use. Analyze these values over time to track database utilization. For more information see [the official documentation.](http://docs.mongodb.com/manual/reference/command/top/index.html)

---

#### `db.serverStatus`로 얻을 수 있는 정보
  - 주요 항목
    - `uptime` : The number of seconds that the current MongoDB process has been active
    - `asserts` : MongoDB 가 started 된 후부터 체크하는 항목
    - `connections` : MongoDB 서버에 연결된 커넥션에 대한 정보
    - `flowControl` : MongoDB의 primary DB의 commit rate를 제어할 때 유용하다.
    - `globallock` : DB의 lock state를 report한다.
    - `opLatencies` : operation 지연을 리포팅한다.
    - `wiredTiger` : sotrage engince 에 대한 통계
      - db의 transaction, session 등 database 관련한 정보

  - 자세한 내용은 여기서.. [https://docs.mongodb.com/manual/reference/command/serverStatus/](https://docs.mongodb.com/manual/reference/command/serverStatus/)

#### `replSetGetStatus`로 얻을 수 있는 정보

  `admin` 권한을 가지는 계정으로만 해당 정보를 얻을 수 있다. replica set에 대한 통계를 얻을 수 있음

  - `members` : replica set으로 구성된 여러 서버들에 대한 정보
  - 자세한 내용은 여기서.. [https://docs.mongodb.com/manual/reference/command/replSetGetStatus/](https://docs.mongodb.com/manual/reference/command/replSetGetStatus/)


### 3-2. Percona exporter

위의 exporter에서 sharding 모니터링까지 추가된 exporter 이다.
[Percona Exporter](https://devconnected.com/mongodb-monitoring-with-grafana-prometheus/)

그라파나 대쉬보드는 3-1. exporter에 주요한 정보가 잘 보여서 해당 exporter를 채택 했다.

---
[참고문서]
- dcu exporter 사용법
https://github.com/dcu/mongodb_exporter
- dcu exporter 그라파나
https://grafana.com/grafana/dashboards/2583
