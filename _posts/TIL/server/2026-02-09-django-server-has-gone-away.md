---
title:  "Django async worker 에서 MySQL Server has gone away 에러 해결"
date: 2026-02-09
excerpt: "sync_to_async의 thread_sensitive 옵션이 Django DB connection 관리에 미치는 영향"
tags: [til, django, mysql, async]
classes: wide
categories: [til/server]
---

Django 기반의 비동기 worker 에서 retry 로직 수행 중 `OperationalError: (2006, 'Server has gone away')` 에러가 반복 발생하는 이슈를 디버깅한 내용을 정리한다.

## 증상

비동기 worker 에서 특정 작업이 실패 후 재시도할 때, 첫 번째 시도에서 DB 연결이 끊기면 이후 **모든 retry 에서도 동일한 에러가 반복**되는 현상이 발생했다.

```
MySQLdb.OperationalError: (2006, 'Server has gone away')
```

왜 retry 를 하는데도 connection 이 복구되지 않는 것일까?

## Background: Django의 DB Connection 관리

Django 는 **thread-local storage** 에 DB connection 을 저장한다. 즉, 각 스레드마다 고유한 DB connection 을 가진다.

```python
# django/db/utils.py
class ConnectionHandler:
    def __init__(self, databases=None):
        self._databases = databases
        self._settings = databases
        self.thread_critical = True  # thread-local 사용
```

그리고 connection 의 수명은 `CONN_MAX_AGE` 설정으로 관리되는데, 핵심은 이 cleanup 이 **request/response boundary** 에서만 동작한다는 것이다.

```python
# django/db/__init__.py
# request_started signal에 연결됨
def reset_queries(**kwargs):
    for conn in connections.all():
        conn.queries_log.clear()

signals.request_started.connect(reset_queries)
signals.request_started.connect(close_old_connections)  # 여기서 stale connection 정리
signals.request_finished.connect(close_old_connections)
```

이 signal 은 Django 의 WSGI/ASGI handler 에서 HTTP request 가 들어올 때 발생한다. **Celery worker 나 비동기 task runner 같은 non-request-based worker 에서는 이 signal 이 발생하지 않기 때문에, `CONN_MAX_AGE` 에 의한 connection cleanup 이 동작하지 않는다.**

## 핵심: sync_to_async 와 thread_sensitive

이 문제의 핵심 원인은 `sync_to_async` 의 `thread_sensitive` 옵션이다.

```python
from asgiref.sync import sync_to_async

# thread_sensitive=True 가 기본값
result = await sync_to_async(some_django_orm_call)()
```

`thread_sensitive=True` (기본값) 일 때, 모든 호출이 **하나의 공유된 메인 스레드**에서 실행된다. Django ORM 은 thread-safe 하지 않기 때문에 이것이 기본값인 것이다.

이것이 의미하는 바는:

```
[async event loop]
    │
    ├─ attempt 1: sync_to_async(db_call)  ──→  [Main Thread] ──→ DB Connection A (thread-local)
    │                                                                    ↓
    │                                                              connection 끊김!
    │
    ├─ attempt 2: sync_to_async(db_call)  ──→  [Main Thread] ──→ DB Connection A (여전히 같은 broken connection)
    │                                                                    ↓
    │                                                              Server has gone away!
    │
    └─ attempt 3: sync_to_async(db_call)  ──→  [Main Thread] ──→ DB Connection A (또 같은 broken connection)
                                                                     ↓
                                                               Server has gone away!
```

**같은 스레드 → 같은 thread-local connection → 같은 broken connection** 이 계속 재사용되는 것이다.

만약 `thread_sensitive=False` 라면, 각 호출이 thread pool 의 랜덤한 스레드에서 실행되기 때문에 다른 스레드의 fresh connection 을 쓸 수도 있다. 하지만 이 방식은 Django ORM 의 thread-safety 문제가 있으므로 권장되지 않는다.

## ProxySQL 을 쓰면 해결되지 않을까?

ProxySQL 을 sidecar 로 배포하면 backend connection pooling 을 관리해주기 때문에, DB connection 관련 문제가 해결될 것으로 기대할 수 있다.

```
Django ──(unix socket)──→ ProxySQL Sidecar ──(TCP)──→ MySQL
              ↑                    ↑
         client-side          backend pooling
         connection           (ProxySQL 관리)
         (Django 관리)
```

하지만 ProxySQL 은 **backend** (ProxySQL ↔ MySQL) 연결만 관리한다. **frontend** (Django ↔ ProxySQL) 연결은 Django의 `MySQLdb` client library 가 관리하며, ProxySQL 이 Django 에게 "connection 을 다시 열어라" 라고 지시할 방법이 없다.

즉, Django 의 `MySQLdb` connection 객체가 broken 상태가 되면, ProxySQL 이 아무리 건강한 backend connection pool 을 가지고 있어도 Django 쪽에서 먼저 stale connection 을 닫고 새로 열어야 한다.

## 해결

retry exception handler 에서 `close_old_connections()` 를 명시적으로 호출한다.

```python
from django.db import close_old_connections
from asgiref.sync import sync_to_async

async def handle(self):
    for attempt in range(max_retries):
        try:
            result = await sync_to_async(self.repository.get_something)(id)
            # ... 비즈니스 로직
            break
        except Exception:
            await sync_to_async(close_old_connections)()  # stale connection 정리
            logger.exception("failed, attempt %s/%s", attempt + 1, max_retries)
```

`close_old_connections()` 는 `CONN_MAX_AGE` 를 초과한 connection 을 닫는 함수인데, `CONN_MAX_AGE=0` 일 경우 **모든 connection 을 닫는다**. Worker 환경에서는 보통 `CONN_MAX_AGE=0` 이므로, 이 호출로 broken connection 이 정리되고 다음 ORM 호출 시 새로운 connection 이 생성된다.

포인트는 `sync_to_async` 로 감싸서 호출해야 한다는 것이다. `close_old_connections()` 는 **현재 스레드**의 thread-local connection 을 닫는 함수이므로, `thread_sensitive=True` (기본값) 로 호출해야 broken connection 이 있는 그 메인 스레드에서 실행된다.

```
[async event loop]
    │
    ├─ attempt 1: sync_to_async(db_call)  ──→  [Main Thread] ──→ connection 끊김
    │
    ├─ sync_to_async(close_old_connections)  ──→  [Main Thread] ──→ broken connection 닫음 ✅
    │
    └─ attempt 2: sync_to_async(db_call)  ──→  [Main Thread] ──→ 새로운 connection 생성 ✅
```

## 정리

| 구분 | 설명 |
|------|------|
| `sync_to_async(thread_sensitive=True)` | 모든 호출이 하나의 공유 스레드에서 실행됨 (기본값) |
| Django DB connection | thread-local storage 에 저장됨 |
| `CONN_MAX_AGE` cleanup | request/response boundary 에서만 동작 → worker 에서는 무효 |
| ProxySQL sidecar | backend pooling 만 관리, client-side connection state 는 Django 책임 |
| 해결 | retry 시 `close_old_connections()` 를 명시적으로 호출하여 stale connection 정리 |

Worker 환경에서 Django ORM 을 사용할 때는, request/response lifecycle 이 없다는 것을 항상 염두에 두어야 한다. `CONN_MAX_AGE` 에 의존하지 말고, 필요한 시점에 명시적으로 connection 을 관리해야 한다.

### Reference

[Django Database - Connection management](https://docs.djangoproject.com/en/5.0/ref/databases/#connection-management)
[asgiref - sync_to_async](https://github.com/django/asgiref#sync_to_async)
[Django Ticket #21597 - close_old_connections for Celery](https://code.djangoproject.com/ticket/21597)
[ProxySQL Documentation - Connection Pooling](https://proxysql.com/documentation/main-runtime/#mysql-connection-pool)
