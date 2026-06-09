# Async IO - async/await, asyncio, coroutines, event loop (Python 3.5+)

## Introduction

Asyncio is Python's library for writing concurrent code using the `async`/`await` syntax. It provides an event loop that manages coroutines, allowing single-threaded concurrent I/O operations without blocking. It is ideal for I/O-bound and high-level structured network code.

## Why It Is Important

Asyncio enables handling thousands of concurrent connections without the overhead of threads or processes. It provides fine-grained control over concurrency with coroutines, tasks, and futures, making it perfect for web servers, network clients, APIs, and any application that spends time waiting for I/O.

## Syntax

```python
import asyncio

# Define a coroutine
async def my_coroutine():
    await asyncio.sleep(1)
    return "done"

# Run the coroutine
result = asyncio.run(my_coroutine())

# Create a task
task = asyncio.create_task(my_coroutine())
await task

# Wait for multiple coroutines
results = await asyncio.gather(coro1(), coro2(), coro3())

# Run in executor for blocking code
result = await loop.run_in_executor(None, blocking_func, arg)
```

## Examples

### Basic Coroutine and Event Loop

```python
import asyncio
import time

async def say_hello():
    print("Hello...")
    await asyncio.sleep(1)
    print("...World!")
    return "Done"

async def main():
    print(f"Start at {time.strftime('%X')}")
    result = await say_hello()
    print(f"Result: {result}")
    print(f"End at {time.strftime('%X')}")

asyncio.run(main())
```

### Multiple Coroutines with gather

```python
import asyncio
import time

async def fetch_data(name, delay):
    print(f"Fetching {name}...")
    await asyncio.sleep(delay)
    print(f"Fetched {name}!")
    return f"Data from {name}"

async def main():
    start = time.time()

    results = await asyncio.gather(
        fetch_data("api-1", 2),
        fetch_data("api-2", 1),
        fetch_data("api-3", 3),
        fetch_data("api-4", 0.5),
    )

    elapsed = time.time() - start
    print(f"\nResults: {results}")
    print(f"Total time: {elapsed:.2f}s (vs ~3s sequentially)")

asyncio.run(main())
```

### Creating and Awaiting Tasks

```python
import asyncio
import time

async def background_work(name, duration):
    for i in range(3):
        await asyncio.sleep(duration)
        print(f"Task {name}: iteration {i}")
    return f"{name} completed"

async def main():
    task1 = asyncio.create_task(background_work("A", 0.5))
    task2 = asyncio.create_task(background_work("B", 0.3))

    print("Tasks created, doing other work...")
    await asyncio.sleep(0.2)
    print("Still working while tasks run...")
    await asyncio.sleep(0.5)

    result1 = await task1
    result2 = await task2
    print(f"Results: {result1}, {result2}")

asyncio.run(main())
```

### Using asyncio.wait()

```python
import asyncio

async def worker(name, delay):
    await asyncio.sleep(delay)
    return f"{name} finished after {delay}s"

async def main():
    tasks = {
        asyncio.create_task(worker("A", 1), name="A"),
        asyncio.create_task(worker("B", 2), name="B"),
        asyncio.create_task(worker("C", 3), name="C"),
    }

    done, pending = await asyncio.wait(tasks, timeout=2.5)
    print(f"Completed: {len(done)}, Pending: {len(pending)}")

    for task in done:
        print(f"  {task.get_name()}: {task.result()}")

    for task in pending:
        print(f"  {task.get_name()}: still pending, cancelling...")
        task.cancel()

    if pending:
        await asyncio.wait(pending)

    print("All done")

asyncio.run(main())
```

### asyncio.run_coroutine_threadsafe

```python
import asyncio
import threading
import time

async def async_task(name):
    await asyncio.sleep(1)
    return f"Result from {name}"

def thread_worker(loop):
    asyncio.set_event_loop(loop)
    loop.run_forever()

async def main():
    loop = asyncio.new_event_loop()
    t = threading.Thread(target=thread_worker, args=(loop,), daemon=True)
    t.start()

    future = asyncio.run_coroutine_threadsafe(async_task("Thread-safe"), loop)
    result = future.result(timeout=5)
    print(f"Got result via thread-safe call: {result}")

    loop.call_soon_threadsafe(loop.stop)
    t.join()
    loop.close()

asyncio.run(main())
```

### Handling Exceptions in Coroutines

```python
import asyncio

async def might_fail(name, fail=False):
    await asyncio.sleep(0.5)
    if fail:
        raise ValueError(f"{name} failed!")
    return f"{name} succeeded"

async def main():
    tasks = [
        asyncio.create_task(might_fail("A", fail=False)),
        asyncio.create_task(might_fail("B", fail=True)),
        asyncio.create_task(might_fail("C", fail=False)),
    ]

    results = await asyncio.gather(*tasks, return_exceptions=True)
    for i, r in enumerate(results):
        if isinstance(r, Exception):
            print(f"Task {i}: Exception - {r}")
        else:
            print(f"Task {i}: {r}")

    task = asyncio.create_task(might_fail("D", fail=True))
    try:
        await task
    except ValueError as e:
        print(f"Caught exception: {e}")

asyncio.run(main())
```

### Timeout with asyncio.wait_for

```python
import asyncio

async def slow_operation():
    await asyncio.sleep(10)
    return "Slow result"

async def quick_operation():
    await asyncio.sleep(1)
    return "Quick result"

async def main():
    try:
        result = await asyncio.wait_for(slow_operation(), timeout=2)
    except asyncio.TimeoutError:
        print("Slow operation timed out!")

    try:
        result = await asyncio.wait_for(quick_operation(), timeout=5)
        print(f"Quick operation: {result}")
    except asyncio.TimeoutError:
        print("Quick operation timed out!")

asyncio.run(main())
```

### Using asyncio.Queue

```python
import asyncio
import random

async def producer(queue, name, count):
    for i in range(count):
        item = f"{name}-item-{i}"
        await asyncio.sleep(random.uniform(0.1, 0.3))
        await queue.put(item)
        print(f"Produced: {item}")
    await queue.put(None)

async def consumer(queue, name):
    while True:
        item = await queue.get()
        if item is None:
            queue.task_done()
            break
        await asyncio.sleep(random.uniform(0.2, 0.4))
        print(f"Consumer {name}: processed {item}")
        queue.task_done()

async def main():
    queue = asyncio.Queue(maxsize=5)

    producers = [asyncio.create_task(producer(queue, f"P{i}", 4)) for i in range(2)]
    consumers = [asyncio.create_task(consumer(queue, i)) for i in range(3)]

    await asyncio.gather(*producers)
    await queue.join()

    for c in consumers:
        c.cancel()

    print("Queue processing completed")

asyncio.run(main())
```

### Running Blocking Code in Executor

```python
import asyncio
import time
import concurrent.futures

def blocking_io(name, duration):
    print(f"Blocking {name} started")
    time.sleep(duration)
    print(f"Blocking {name} completed")
    return f"Result from {name}"

async def async_task(name):
    await asyncio.sleep(0.5)
    return f"Async {name} done"

async def main():
    loop = asyncio.get_running_loop()

    with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
        blocking_tasks = [
            loop.run_in_executor(executor, blocking_io, "IO-1", 2),
            loop.run_in_executor(executor, blocking_io, "IO-2", 3),
            loop.run_in_executor(executor, blocking_io, "IO-3", 1),
        ]

        async_tasks = [
            async_task("A"),
            async_task("B"),
        ]

        all_tasks = blocking_tasks + async_tasks
        results = await asyncio.gather(*all_tasks)
        print(f"All results: {results}")

asyncio.run(main())
```

### Semaphore for Rate Limiting

```python
import asyncio
import time

semaphore = asyncio.Semaphore(3)

async def limited_request(url, request_id):
    async with semaphore:
        print(f"Request {request_id} to {url} started")
        await asyncio.sleep(1)
        print(f"Request {request_id} completed")
        return f"Response for {request_id}"

async def main():
    start = time.time()
    tasks = [
        limited_request("https://api.example.com", i)
        for i in range(10)
    ]

    results = await asyncio.gather(*tasks)
    elapsed = time.time() - start
    print(f"\nAll {len(results)} requests completed in {elapsed:.2f}s")
    print(f"(Would be ~10s without semaphore limiting to 3 concurrent)")

asyncio.run(main())
```

## Beginner Examples

```python
# Simple async HTTP client simulation
import asyncio

async def fetch_url(url):
    print(f"Fetching {url}...")
    await asyncio.sleep(1)
    return f"<html>Content of {url}</html>"

async def process_urls():
    urls = [
        "https://example.com",
        "https://python.org",
        "https://asyncio.org",
        "https://docs.python.org",
    ]

    tasks = [asyncio.create_task(fetch_url(url)) for url in urls]
    pages = await asyncio.gather(*tasks)

    for url, page in zip(urls, pages):
        print(f"{url}: {len(page)} bytes")

asyncio.run(process_urls())

# Async context manager
import asyncio

class AsyncResource:
    async def __aenter__(self):
        print("Acquiring resource...")
        await asyncio.sleep(0.5)
        print("Resource acquired")
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Releasing resource...")
        await asyncio.sleep(0.3)
        print("Resource released")

    async def use(self):
        await asyncio.sleep(0.2)
        return "Using resource"

async def main():
    async with AsyncResource() as resource:
        result = await resource.use()
        print(result)

asyncio.run(main())
```

## Intermediate Examples

```python
# Async retry with exponential backoff
import asyncio
import random

async def unstable_operation(name, fail_probability=0.6):
    await asyncio.sleep(random.uniform(0.1, 0.3))
    if random.random() < fail_probability:
        raise ConnectionError(f"{name} failed transiently")
    return f"{name} succeeded"

async def retry_with_backoff(coro_factory, max_retries=3, base_delay=0.1):
    for attempt in range(max_retries):
        try:
            return await coro_factory()
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            delay = base_delay * (2 ** attempt) + random.uniform(0, 0.1)
            print(f"  Attempt {attempt + 1} failed: {e}. Retrying in {delay:.2f}s")
            await asyncio.sleep(delay)
    return None

async def main():
    for i in range(5):
        try:
            result = await retry_with_backoff(
                lambda: unstable_operation(f"Op-{i}", fail_probability=0.5),
                max_retries=3
            )
            print(f"Success: {result}")
        except Exception as e:
            print(f"All retries exhausted for Op-{i}: {e}")

asyncio.run(main())

# Async producer-consumer with graceful shutdown
import asyncio
import signal

class GracefulShutdown:
    def __init__(self):
        self.shutdown_event = asyncio.Event()

    async def wait_shutdown(self):
        await self.shutdown_event.wait()

    def shutdown(self):
        self.shutdown_event.set()

async def producer(queue, shutdown):
    for i in range(20):
        if shutdown.shutdown_event.is_set():
            break
        await queue.put(f"item-{i}")
        await asyncio.sleep(0.2)
    for _ in range(5):
        await queue.put(None)

async def consumer(queue, id, shutdown):
    while not shutdown.shutdown_event.is_set():
        try:
            item = await asyncio.wait_for(queue.get(), timeout=1)
            if item is None:
                queue.task_done()
                break
            await asyncio.sleep(0.3)
            print(f"Consumer {id}: processed {item}")
            queue.task_done()
        except asyncio.TimeoutError:
            continue

async def main():
    shutdown = GracefulShutdown()
    queue = asyncio.Queue()

    tasks = [
        asyncio.create_task(producer(queue, shutdown)),
        asyncio.create_task(consumer(queue, 1, shutdown)),
        asyncio.create_task(consumer(queue, 2, shutdown)),
    ]

    await asyncio.sleep(2)
    shutdown.shutdown()

    await asyncio.gather(*tasks, return_exceptions=True)
    print("Graceful shutdown completed")
```

## Advanced Examples

```python
# Custom async event loop implementation basics
import asyncio
import time
from collections import deque
from typing import Coroutine, Any

class SimpleEventLoop:
    def __init__(self):
        self._ready = deque()
        self._stopping = False

    def call_soon(self, callback, *args):
        self._ready.append((callback, args))

    def stop(self):
        self._stopping = True

    def run_forever(self):
        while not self._stopping:
            if self._ready:
                callback, args = self._ready.popleft()
                callback(*args)
            else:
                time.sleep(0.001)

    def run_until_complete(self, coro):
        task = asyncio.ensure_future(coro, loop=self)
        self.run_forever()
        return task.result()

async def demo_coroutine():
    print("Coroutine started")
    await asyncio.sleep(0.5)
    print("Coroutine resumed after sleep")
    return 42

async def advanced_main():
    print("=== Advanced Asyncio Patterns ===\n")

    # Custom task group
    class TaskGroup:
        def __init__(self):
            self._tasks = set()

        def create_task(self, coro):
            task = asyncio.create_task(coro)
            self._tasks.add(task)
            task.add_done_callback(self._tasks.discard)
            return task

        async def __aenter__(self):
            return self

        async def __aexit__(self, *args):
            if self._tasks:
                await asyncio.gather(*self._tasks, return_exceptions=True)

    async with TaskGroup() as tg:
        tg.create_task(asyncio.sleep(0.5, result="Task 1"))
        tg.create_task(asyncio.sleep(0.3, result="Task 2"))
        print("Tasks created in group")

    print("Task group completed")

    # Async generator
    async def async_range(start, end, delay=0.1):
        for i in range(start, end):
            await asyncio.sleep(delay)
            yield i

    print("\nAsync range:")
    async for value in async_range(0, 5, 0.1):
        print(f"  {value}", end=" ")
    print()

    # Async context manager with state
    class AsyncConnection:
        def __init__(self, host):
            self.host = host
            self.connected = False

        async def connect(self):
            await asyncio.sleep(0.2)
            self.connected = True
            return self

        async def disconnect(self):
            await asyncio.sleep(0.1)
            self.connected = False

        async def query(self, sql):
            if not self.connected:
                raise RuntimeError("Not connected")
            await asyncio.sleep(0.1)
            return f"Results for: {sql}"

        async def __aenter__(self):
            return await self.connect()

        async def __aexit__(self, *args):
            await self.disconnect()

    async with AsyncConnection("localhost") as conn:
        result = await conn.query("SELECT * FROM users")
        print(f"\nDatabase result: {result}")

    print(f"Connection closed: {not conn.connected}")

    # Async memoization
    class AsyncMemoize:
        def __init__(self, func):
            self.func = func
            self.cache = {}
            self.lock = asyncio.Lock()

        async def __call__(self, *args, **kwargs):
            key = (args, tuple(sorted(kwargs.items())))
            async with self.lock:
                if key not in self.cache:
                    self.cache[key] = await self.func(*args, **kwargs)
                return self.cache[key]

    @AsyncMemoize
    async def expensive_computation(n):
        await asyncio.sleep(1)
        return n * n

    start = time.time()
    results = await asyncio.gather(
        expensive_computation(5),
        expensive_computation(5),
        expensive_computation(10),
        expensive_computation(10),
    )
    elapsed = time.time() - start
    print(f"\nMemoized results: {results}")
    print(f"Time: {elapsed:.2f}s (would be ~4s without memoization)")

    # Async pipeline with error recovery
    async def pipeline_stage(name, input_queue, output_queue, process_func):
        while True:
            try:
                item = await asyncio.wait_for(input_queue.get(), timeout=1)
            except asyncio.TimeoutError:
                break

            try:
                result = await process_func(item)
                await output_queue.put(result)
            except Exception as e:
                print(f"Stage {name}: error processing {item}: {e}")
                continue

    async def stage1_process(item):
        await asyncio.sleep(0.1)
        return item * 2

    async def stage2_process(item):
        await asyncio.sleep(0.15)
        if item % 10 == 0:
            raise ValueError(f"Bad value: {item}")
        return f"Processed-{item}"

    q1, q2 = asyncio.Queue(), asyncio.Queue()

    for i in range(8):
        await q1.put(i)

    stages = [
        asyncio.create_task(pipeline_stage("S1", q1, q2, stage1_process)),
        asyncio.create_task(pipeline_stage("S2", q2, None, stage2_process)),
    ]

    await asyncio.sleep(2)
    for s in stages:
        s.cancel()

    print("\nPipeline completed with error recovery")

    # Async rate limiter with token bucket
    class TokenBucket:
        def __init__(self, rate, capacity):
            self.rate = rate
            self.capacity = capacity
            self.tokens = capacity
            self.last_refill = time.monotonic()
            self.lock = asyncio.Lock()

        async def acquire(self):
            async with self.lock:
                now = time.monotonic()
                elapsed = now - self.last_refill
                self.tokens = min(self.capacity, self.tokens + elapsed * self.rate)
                self.last_refill = now

                if self.tokens >= 1:
                    self.tokens -= 1
                    return True

                wait_time = (1 - self.tokens) / self.rate
                await asyncio.sleep(wait_time)
                self.tokens = 0
                self.last_refill = time.monotonic()
                return True

            async def __aenter__(self):
                await self.acquire()
                return self

            async def __aexit__(self, *args):
                pass

    print("\nRate limiter demo: 5 requests/s")
    bucket = TokenBucket(rate=5, capacity=5)

    async def rate_limited_request(i):
        await bucket.acquire()
        print(f"  Request {i} at {time.strftime('%X')}")
        return i

    start = time.time()
    results = await asyncio.gather(*[rate_limited_request(i) for i in range(12)])
    elapsed = time.time() - start
    print(f"12 requests completed in {elapsed:.1f}s (expected ~2.4s at 5 req/s)")

asyncio.run(advanced_main())
```

## Real-World Use Cases

- **Web frameworks**: FastAPI, aiohttp, Sanic use asyncio for high-performance HTTP servers
- **Web scraping/crawling**: Async HTTP clients crawl hundreds of pages concurrently
- **Real-time applications**: Chat servers, live dashboards, WebSocket connections
- **API gateways**: Aggregating responses from multiple microservices concurrently
- **Database access**: asyncpg, aiomysql for non-blocking database queries
- **Message queues**: Async consumers for RabbitMQ, Kafka, Redis streams

## Common Mistakes

- Blocking the event loop with synchronous I/O or CPU-heavy code
- Forgetting to `await` a coroutine (returns a coroutine object, not the result)
- Mixing asyncio and threading without proper synchronization
- Not handling cancellation properly (tasks may raise `CancelledError`)
- Creating too many tasks at once without rate limiting (overwhelming resources)
- Using `asyncio.run()` inside a running event loop (use `await` instead)
- Sharing mutable objects between coroutines without locks
- Ignoring exception handling in `asyncio.gather()` (use `return_exceptions=True`)

## Best Practices

- Use `asyncio.run()` as the main entry point for async programs
- Prefer `asyncio.gather()` over manually awaiting multiple coroutines sequentially
- Use `asyncio.create_task()` to schedule coroutines concurrently
- Handle `asyncio.CancelledError` properly in cleanup code
- Use `asyncio.timeout()` (3.11+) or `asyncio.wait_for()` for timeouts
- Keep coroutines focused on I/O; offload CPU work to `run_in_executor()`
- Use `asyncio.Queue` for producer-consumer patterns
- Always use `async with` for proper resource cleanup

## Interview Questions

1. **Q**: How does asyncio achieve concurrency with a single thread?
   **A**: Asyncio uses an event loop that schedules and runs coroutines cooperatively. Coroutines yield control at `await` points, allowing the event loop to switch to other ready coroutines. This is cooperative multitasking, not preemptive.

2. **Q**: What is the difference between a coroutine and a task?
   **A**: A coroutine is an async function that can be awaited. A task wraps a coroutine and schedules it on the event loop, providing control methods like `cancel()` and allowing inspection of its state.

3. **Q**: How does `asyncio.gather()` differ from `asyncio.wait()`?
   **A**: `gather()` returns results in order when all tasks complete (or raises on first exception). `wait()` returns two sets of tasks (done/pending) and supports timeouts and more flexible control over when to return.

4. **Q**: What is the problem with blocking the event loop?
   **A**: Blocking the event loop with `time.sleep()` or CPU-intensive code prevents the loop from switching to other tasks. All coroutines stall, defeating the purpose of async concurrency.

5. **Q**: How do you run synchronous code from an async context?
   **A**: Use `loop.run_in_executor(None, sync_func, *args)` which runs the blocking function in a thread pool and returns an awaitable.

## Coding Challenges

1. **Async Web Crawler**: Build a concurrent web crawler that fetches pages concurrently, respects `robots.txt`, and implements politeness delays.

2. **Real-time Chat Server**: Implement a WebSocket chat server using asyncio that handles multiple clients, rooms, and message broadcasting.

3. **Async Task Scheduler**: Create a scheduler that runs async tasks at specified intervals with error handling, retries, and status reporting.

4. **Rate-Limited API Client**: Build an async HTTP client that respects rate limits using a token bucket or sliding window algorithm.

5. **Distributed Event Bus**: Implement a publish-subscribe event bus using asyncio queues with pattern-based subscription matching.

## Summary

Asyncio enables single-threaded concurrent I/O through coroutines, tasks, and an event loop. It excels at I/O-bound workloads requiring high concurrency, such as web servers, network clients, and real-time applications. Key patterns include `gather()`, `create_task()`, `Queue`, semaphores, and offloading blocking code to executors.

## Related Topics

Asyncio Advanced (62.x), Multithreading (59.x), Multiprocessing (60.x), GIL (64.x)
