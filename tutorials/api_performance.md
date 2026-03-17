---
layout: page
title: Optimizing API Performance
parent: the-dev-hub
date: 2026-03-17 11:00 +0000
categories: [FastAPI, Tutorial]
tags: [python, fastapi, performance, redis, pagination, logging]
---

# Optimizing API Performance: Key Strategies and Python Implementations
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

API performance is critical for scalable web services, particularly in high-throughput environments like those using FastAPI. This post examines four evidence-based techniques—pagination, asynchronous logging, caching with Redis, and connection pooling—through formal analysis and Python examples tailored to FastAPI applications.

## Pagination

Pagination divides large datasets into manageable pages, reducing response times and bandwidth usage by avoiding full data transmission. Clients specify page and size parameters, enabling offset-based or cursor-based fetching.

In FastAPI with SQLAlchemy, use the `fastapi-pagination` library for automatic handling:

```python
from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session
from fastapi_pagination.ext.sqlalchemy import paginate
from sqlalchemy import select

app = FastAPI()
add_pagination(app)  # Enable pagination

def get_db():  # Dependency for DB session
    # Implementation omitted
    pass

@app.get("/users", response_model=Page[UserOut])
def get_users(db: Session = Depends(get_db)):
    return paginate(db, select(User))  # Paginates query automatically
```

This yields responses like `{"items": [...], "total": 100, "page": 1, "size": 20}`. Offset pagination suits small-to-medium datasets but degrades with large offsets; cursor pagination is preferable for millions of records.

## Asynchronous Logging

Synchronous disk I/O in logging blocks the event loop, inflating latency in async frameworks like FastAPI. Async logging queues records for background flushing, decoupling log writes from request handling.

Implement a custom non-blocking handler using `contextvars`:

```python
import logging
from contextvars import ContextVar
from datetime import datetime
from fastapi import Request, Depends

REQUEST_INFO: ContextVar[list] = ContextVar("request_logs", default=[])

class AsyncHandler(logging.Handler):
    def emit(self, record: logging.LogRecord):
        logs = REQUEST_INFO.get()
        logs.append(f"{datetime.now()} - {record.levelname} - {record.getMessage()}")

def request_dep(request: Request):
    logs = []
    token = REQUEST_INFO.set(logs)
    try:
        yield
    finally:
        REQUEST_INFO.reset(token)
        # Flush logs asynchronously (e.g., to file or queue)
        for log in logs:
            print(log)  # Replace with async writer

app = FastAPI(dependencies=[Depends(request_dep)])
logging.basicConfig(handlers=[AsyncHandler()], level=logging.INFO)
```

This buffers logs per request and flushes post-response, cutting I/O wait by 20-50%.

## Caching with Redis

Database queries dominate API latency; in-memory caching via Redis intercepts repeats, serving data in microseconds versus milliseconds. Check cache first, fallback to DB, and store misses with TTL.

Using `fastapi-redis-cache`:

```python
from fastapi import FastAPI
from fastapi_redis_cache import FastApiRedisCache, cache
from contextlib import asynccontextmanager

redis_cache = FastApiRedisCache()

@asynccontextmanager
async def lifespan(app: FastAPI):
    redis_cache.init(host_url="redis://localhost:6379", prefix="api-cache")
    yield

app = FastAPI(lifespan=lifespan)

@app.get("/users/{user_id}")
@cache()  # Auto-caches responses by params
async def get_user(user_id: int, db: Session = Depends(get_db)):
    # DB query if miss
    user = await fetch_user(db, user_id)
    return user
```

Hits return instantly with cache headers like `X-Key: hit`. Evict on mutations via `redis_cache.clear()`.

## Connection Pooling

Repeated connection open/close incurs overhead; pools reuse persistent connections, amortizing setup costs. SQLAlchemy's `QueuePool` manages this with configurable size and recycling.

Configure in FastAPI:

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "postgresql+asyncpg://user:pass@localhost/db"
engine = create_async_engine(
    DATABASE_URL,
    pool_size=10,      # Min connections
    max_overflow=20,   # Max burst
    pool_timeout=30,   # Wait timeout
    pool_recycle=1800  # Recycle stale
)

AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

async def get_db():
    async with AsyncSessionLocal() as session:
        yield session
```

This sustains high concurrency without exhaustion.

## Combined Impact

| Technique          | Latency Reduction | Use Case Fit                  |
|--------------------|-------------------|-------------------------------|
| Pagination        | 50-90%           | List endpoints                |
| Async Logging     | 20-50%           | I/O-heavy services            |
| Redis Caching     | 80-99% (hits)    | Read-heavy data               |
| Connection Pool   | 10-30%           | DB-intensive APIs             |

Integrate via FastAPI dependencies for production ML/agent APIs.

---

### References
- [Battle-tested strategies to supercharge API performance](https://dev.to/dehemi_fabio/5-battle-tested-strategies-to-supercharge-your-api-performance-22ah)
- [FastAPI Pagination Documentation](https://uriyyo-fastapi-pagination.netlify.app)
- [FastAPI Redis Cache GitHub](https://github.com/seapagan/fastapi-redis-cache-reborn)
- [Database Pooling in FastAPI](https://asifmuhammad.com/articles/database-pooling-fastapi)
