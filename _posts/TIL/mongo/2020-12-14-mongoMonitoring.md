---
title:  "[Mongo] 몽고 db 모니터링을 위한 exporter 설치"
date: 2021-01-05
excerpt: ""
tags: [til, mongo]
categories: [til/mongo]
---

Mongo DB를 모니터링하기 위해 여러 방법이 있다.

## [Free Cloud monitoring](https://docs.mongodb.com/manual/administration/free-monitoring/)

By default, you can enable/disable free monitoring during runtime using `db.enableFreeMonitoring()` and `db.disableFreeMonitoring()`.

> 주의
Mongo 4.0 이상에서만 가능하다.


Monitored Data
  - Operation Execution Times
  - Memory Usage
  - CPU Usage
  - Operation Counts

mogod에서 아래 커맨드를 입력하면 모니터링 URL이 리턴된다.
``` console
db.getFreeMonitoringStatus() // 이게 OK이면 모니터링 되고 있다는 뜻
db.enableFreeMonitoring() // 실행하면 unique url 이 나온다.
```

실시간이 아닌, 24시간 이내 데이터가 추출되고 24시간 후에는 해당 데이터가 expire 된다.

## Mongo DB에서 기본 제공하는 CLI 명령어

1. [mongotop](https://docs.mongodb.com/database-tools/mongotop/#bin.mongotop)을 이용한 각 collection별 read write 속도 모니터링

```console
./mongotop --host localhost --port 27017 -u user -p 'password' --authenticationDatabase admin
```

2. [mongostat](https://docs.mongodb.com/database-tools/mongostat/#bin.mongostat)을 이용한 query 실행 보니터링

```console
./mongostat --host localhost --port 27017 -u user -p 'password' --authenticationDatabase admin
```

[참고문서]
https://docs.ncloud.com/ko/database/database-10-6.html


## mongod Command

mongod command를 이용한 데이터는 위의 데이터를 가공한 데이터다.

``` console
> db.currentOp
> db.serverStatus()
> db.dbStats()
```

[참고문서]
https://docs.mongodb.com/manual/administration/monitoring/

---
## Mongodb exporter로 데이터 metrics 추출

위에서 언급한 data를 모드 metrics로 추출한다. `db.serverStatus()` 로 얻을 수있는 정보는 모두 포함된다.

> Much of the output of serverStatus is also displayed dynamically by mongostat. See the mongostat command for more information.

#### [David Cuadrado가 만든 exporter](https://github.com/dcu/mongodb_exporter) 의 정보

##### Collect below metrics
- MongoDB Server Status metrics (cursors, operations, indexes, storage, etc)
- MongoDB Replica Set metrics (members, ping, replication lag, etc)
- MongoDB Replication Oplog metrics (size, length in time, etc)
- MongoDB Sharding metrics (shards, chunks, db/collections, balancer operations) --> dcu로는 안되고 percona exporter로는 된다고 한다. [percona exporter](https://github.com/percona/mongodb_exporter)
- MongoDB RocksDB storage-engine metrics (levels, compactions, cache usage, i/o rates, etc)
- MongoDB WiredTiger storage-engine metrics (cache, blockmanger, tickets, etc)
- MongoDB Top Metrics per collection (writeLock, readLock, query, etc*)

##### Available groups of data

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

#### exporter 설치방법
아래 포스트해서 자세히 설명되어 있다.
https://devconnected.com/mongodb-monitoring-with-grafana-prometheus/

- docker로 exporter image import
- docker image 실행
``` console
docker run --name mongodb_exporter --rm -d -p 9001:9001 mongodb_exporter --mongodb.uri=mongodb://mongodb_exporter:mongodb_exporter@10.203.7.214:27017
```
- metric 수집되는지 확인
``` console
curl localhost:9001/metrics
```
- prometheus 에 해당 metrics url 등록
- 그라파나 dashboad import

---
- Percona exporter 사용법
https://devconnected.com/mongodb-monitoring-with-grafana-prometheus/
- dcu exporter 사용법
https://github.com/dcu/mongodb_exporter
- dcu exporter 그라파나
https://grafana.com/grafana/dashboards/2583
