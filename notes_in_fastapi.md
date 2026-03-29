[Back](./README.md)

---

## 📑 Table of Contents

| Section | Topic |
|---------|-------|
| [1](#-core-principles) | Core Principles |
| [2](#-production-checklist-for-high-concurrency) | Production Checklist |

---

# 🚀 FastAPI Performance & Concurrency Guide

A comprehensive guide to ensuring FastAPI actually supports **high concurrency** in production. 🎯

---

## ⚡ Core Principles

### 1️⃣ Avoid Blocking I/O

#### ❌ Don't use blocking libraries

- **`requests`** → blocks the thread for the whole HTTP call
- **`sqlite3` (default driver)** → blocks unless using async driver
- **Regular file operations** like `open()` for big files → block

#### ✅ Use async-friendly alternatives

| Task | ❌ Blocking Library | ✅ Async Alternative |
|------|---------------------|---------------------|
| HTTP requests | `requests` | `httpx` (with await) or `aiohttp` |
| Databases | synchronous drivers (e.g., `psycopg2`) | async drivers (e.g., `asyncpg`, `databases`, SQLAlchemy 2.0 async) |
| File I/O | built-in `open()` for large reads/writes | `aiofiles` |
| Redis | `redis-py` | `aioredis` |
| Kafka | `confluent-kafka` | `aiokafka` |

---

### 2️⃣ Watch Out for CPU-bound Work

Even if it's async code, **heavy CPU work** (e.g., image processing, big JSON parsing) will block the loop. 🔥

#### 💡 Solution:

Offload to background tasks via:

- `asyncio.to_thread()` for threads
- `ProcessPoolExecutor` for heavy CPU jobs
- External workers (Celery, RQ, Dramatiq)

#### 📝 Example:

```python
import asyncio

def cpu_heavy():
    # big image processing here
    pass

async def endpoint():
    result = await asyncio.to_thread(cpu_heavy)
    return result
```

---

### 3️⃣ Always Await Asynchronous Calls

If you forget `await`, the coroutine **never runs** as intended and might block later. ⚠️

#### 🔴 Bad:

```python
httpx.get("https://slow-api.com")  # Blocking
```

#### 🟢 Good:

```python
await httpx.get("https://slow-api.com")  # Non-blocking
```

---

### 4️⃣ Avoid Mixing Sync and Async Code Without Care

If you must use a sync library, **run it in a thread pool**: 🧵

```python
from fastapi import FastAPI
import requests
import asyncio

app = FastAPI()

def blocking_call():
    return requests.get("https://slow-api.com").text

@app.get("/")
async def get_data():
    data = await asyncio.to_thread(blocking_call)
    return {"data": data}
```

This prevents blocking the event loop. ✅

---

### 5️⃣ Monitor and Test for Blocking

- Use tools like `asyncio-run-in-process` or `uvicorn --reload --loop asyncio --http httptools --debug` to see if things stall
- Load-test with **Locust** or **k6** to see concurrency behavior

---

## 🎯 Production Checklist for High Concurrency

If your main goal is to make sure FastAPI actually supports high concurrency in production, you need to think beyond just `async`/`await`. Here's a comprehensive checklist so you don't accidentally kill your concurrency benefits. 📋

---

### 1️⃣ Always Use Async-Friendly Libraries

FastAPI's concurrency advantage **only works** if you never block the event loop. 🔄

**Recommended async libraries:**

- **HTTP** → `httpx` (async mode) or `aiohttp` instead of `requests`
- **DB** → `asyncpg`, `databases`, SQLAlchemy 2.0 async
- **Redis** → `aioredis`
- **File I/O** → `aiofiles`

**If you must use a sync lib** → wrap it with:

```python
await asyncio.to_thread(sync_function)
```

---

### 2️⃣ Use an Async Server

Run FastAPI with:

```bash
uvicorn app.main:app --workers 1 --loop uvloop --http httptools
```

- **`uvloop`** → faster event loop (Cython-based) 🚄
- **`httptools`** → faster HTTP parsing ⚡

⚠️ **`--workers 1`** for maximum concurrency per worker — scale workers if CPU-bound tasks are common.

---

### 3️⃣ Offload CPU-Bound Work

If you do **image processing**, **data crunching**, or **ML inference**: 🧠

- Use `asyncio.to_thread()` or `ProcessPoolExecutor`
- Or move it to a task queue (Celery, RQ, Dramatiq)
- Keep API thread **free** for handling I/O

---

### 4️⃣ Avoid Global Locks and Heavy Middleware

- ❌ No `threading.Lock` unless it's absolutely necessary
- ❌ Avoid middlewares that do blocking I/O
- ❌ Avoid per-request logging that writes to disk synchronously

---

### 5️⃣ Handle Timeouts and Circuit Breaking

If you call slow external APIs: 🌐

- Set timeouts in `httpx.AsyncClient(timeout=...)`
- Use libraries like `aiobreaker` to avoid flooding a failing service
- This prevents **worker pileups** 🚧

---

### 6️⃣ Don't Mix Sync Endpoints by Accident

This is subtle — if you write:

#### 🔴 Bad:

```python
@app.get("/")
def sync_handler():
    ...
```

That blocks your worker like Flask. 😱

#### 🟢 Good:

```python
@app.get("/")
async def async_handler():
    ...
```

---

### 7️⃣ Tune Your Event Loop & Workers

- Start with **1 worker per CPU core** for CPU-bound tasks
- For **I/O-bound**, fewer workers can still handle thousands of requests 🌊
- Monitor with `uvicorn --access-log` + **Prometheus metrics** 📊

---

### 8️⃣ Test Under Load

Use **locust**, **k6**, or **wrk** to simulate 100s/1000s of concurrent requests. 🔬

**Check:**

- ⏱️ Average latency
- 🔄 Event loop lag (`aiomonitor`, `aiomisc`)
- 💪 Worker utilization

---

### 9️⃣ Keep Responses Small if Possible

- Streaming large files? → use `StreamingResponse` instead of loading entire file into memory 📦
- For JSON, avoid massive payloads unless necessary

---

### 🔟 Be Careful with BackgroundTasks

FastAPI's `BackgroundTasks` still run in the **same process** — if they're CPU-heavy, they block. ⚠️

**Use them only** for light async work (e.g., sending an email, logging to DB).

---

## 💡 Key Mindsets

### ✅ The Waiting Mindset

> If your endpoint does something that could make a human wait… make it `await` in code.
> 
> If it eats CPU for more than a split second… push it off the main thread.

---

### ✅ The Golden Rule

> **If it's I/O** — make sure it's async. 🌐
> 
> **If it's CPU** — push it off the main thread/process. 🧵

---

## 🎯 General Rule for Any Library

If it reads files, does network calls, or does heavy computation, and it **doesn't provide async methods** → assume it's **blocking**. 🚫

### In FastAPI async endpoints:

- **For I/O-bound blocking calls** → replace with async library if available
- **For CPU-bound blocking calls** → push to thread/process pool

---

## 📚 Quick Reference Summary

| Scenario | ❌ Avoid | ✅ Use Instead |
|----------|---------|---------------|
| HTTP calls | `requests` | `httpx`, `aiohttp` |
| Database | sync drivers | `asyncpg`, `databases`, SQLAlchemy async |
| File operations | `open()` | `aiofiles` |
| Redis | `redis-py` | `aioredis` |
| Kafka | `confluent-kafka` | `aiokafka` |
| CPU-heavy work | Running in main thread | `asyncio.to_thread()`, Celery |
| Sync functions | Calling directly | `await asyncio.to_thread()` |
| Endpoint definition | `def` | `async def` |

---

## 🎓 Remember

**FastAPI's async capabilities are powerful, but only if you don't accidentally block the event loop!** 🚀

Keep your I/O async, push CPU work off the main thread, and test under load to ensure you're getting the concurrency benefits you expect. 💪

---

[Back](./README.md)
