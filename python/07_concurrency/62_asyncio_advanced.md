# Asyncio Advanced - async generators, context managers, queues

## Introduction

Advanced asyncio patterns extend beyond basic coroutines to include async context managers, async iterators, async generators, synchronization primitives, subprocess management, and debugging techniques. These patterns enable building robust, production-grade asynchronous applications.

## Why It Is Important

Mastering advanced asyncio patterns allows developers to write clean, efficient, and maintainable asynchronous code. Async context managers ensure proper resource cleanup, async iterators enable streaming data processing, and advanced patterns like semaphores and queues control concurrency. Debugging techniques are essential for diagnosing issues in complex async systems.

## Syntax

```python
# Async context manager
class AsyncContextManager:
    async def __aenter__(self): ...
    async def __aexit__(self, exc_type, exc_val, exc_tb): ...

# Async iterator
class AsyncIterator:
    def __aiter__(self): return self
    async def __anext__(self): ...

# Async generator
async def async_generator():
    for i in range(10):
        await asyncio.sleep(0.1)
        yield i

# Async comprehension
result = [x async for x in async_generator()]
result = {x: x**2 async for x in async_generator()}

# Async subprocess
proc = await asyncio.create_subprocess_exec('cmd', *args)
stdout, stderr = await proc.communicate()
```

## Examples

### Async Context Managers

```python
import asyncio
import time

class AsyncFile:
    def __init__(self, filename, mode='r'):
        self.filename = filename
        self.mode = mode
        self.file = None

    async def __aenter__(self):
        print(f"Opening {self.filename}")
        await asyncio.sleep(0.1)
        self.file = open(self.filename, self.mode)
        return self.file

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print(f"Closing {self.filename}")
        await asyncio.sleep(0.1)
        if self.file:
            self.file.close()
        if exc_type:
            print(f"Exception was handled: {exc_val}")
        return True

async def main():
    import tempfile
    import os
    with tempfile.NamedTemporaryFile(mode='w', delete=False) as f:
        f.write("Hello, async world!")
        tmpname = f.name

    async with AsyncFile(tmpname, 'r') as f:
        content = f.read()
        print(f"Content: {content}")

    os.unlink(tmpname)

    async with AsyncFile("nonexistent.txt", 'r') as f:
        content = f.read()
    print("This still runs because __aexit__ returned True")

asyncio.run(main())
```

### Async Iterators

```python
import asyncio

class AsyncCounter:
    def __init__(self, start, end, delay=0.1):
        self.current = start
        self.end = end
        self.delay = delay

    def __aiter__(self):
        return self

    async def __anext__(self):
        if self.current >= self.end:
            raise StopAsyncIteration
        await asyncio.sleep(self.delay)
        value = self.current
        self.current += 1
        return value

async def main():
    print("AsyncCounter (0 to 5):")
    async for value in AsyncCounter(0, 5, 0.2):
        print(f"  {value}", end=" ")
    print()

    print("\nAsyncCounter with filtering:")
    async for value in AsyncCounter(0, 8, 0.1):
        if value % 2 == 0:
            print(f"  {value}", end=" ")
    print()

asyncio.run(main())
```

### Async Generators

```python
import asyncio

async def async_range(start, end, delay=0.1):
    for i in range(start, end):
        await asyncio.sleep(delay)
        yield i

async def fibonacci_async(n):
    a, b = 0, 1
    for _ in range(n):
        await asyncio.sleep(0.1)
        yield a
        a, b = b, a + b

async def main():
    print("Async range:")
    async for value in async_range(0, 5, 0.1):
        print(f"  {value}", end=" ")
    print()

    print("\nFibonacci sequence:")
    async for num in fibonacci_async(8):
        print(f"  {num}", end=" ")
    print()

    squares = [x async for x in async_range(0, 5, 0.05)]
    print(f"\nAsync comprehension (squares): {squares}")

    even_squares = {
        x: x**2
        async for x in async_range(0, 8, 0.05)
        if x % 2 == 0
    }
    print(f"Async dict comprehension: {even_squares}")

    gen = async_range(0, 3, 0.1)
    first = await gen.__anext__()
    print(f"\nManual next on generator: {first}")

asyncio.run(main())
```

### asyncio.Queue Advanced Usage

```python
import asyncio
import random
from dataclasses import dataclass
from enum import Enum

class Priority(Enum):
    LOW = 0
    MEDIUM = 1
    HIGH = 2

@dataclass(order=True)
class PrioritizedItem:
    priority: int
    timestamp: float
    data: str = ""

class AsyncPriorityQueue:
    def __init__(self, maxsize=0):
        self._queue = asyncio.PriorityQueue(maxsize=maxsize)

    async def put(self, item, priority=Priority.MEDIUM):
        wrapped = PrioritizedItem(
            priority=priority.value,
            timestamp=asyncio.get_event_loop().time(),
            data=item
        )
        await self._queue.put(wrapped)

    async def get(self):
        wrapped = await self._queue.get()
        return wrapped.data

    def qsize(self):
        return self._queue.qsize()

async def producer(queue, name, count):
    for i in range(count):
        priority = random.choice(list(Priority))
        item = f"{name}-item-{i}"
        await queue.put(item, priority)
        print(f"Produced: {item} (priority: {priority.name})")
        await asyncio.sleep(random.uniform(0.1, 0.3))
    await queue.put(None, Priority.HIGH)
    await queue.put(None, Priority.HIGH)

async def consumer(queue, name):
    while True:
        item = await queue.get()
        if item is None:
            break
        await asyncio.sleep(random.uniform(0.2, 0.4))
        print(f"Consumer {name}: processed {item}")

async def main():
    queue = AsyncPriorityQueue(maxsize=10)

    producers = [
        asyncio.create_task(producer(queue, f"P{i}", 3))
        for i in range(2)
    ]

    consumers = [
        asyncio.create_task(consumer(queue, i))
        for i in range(2)
    ]

    await asyncio.gather(*producers)
    for c in consumers:
        c.cancel()

    print("Priority queue example completed")

asyncio.run(main())
```

### Semaphores and Rate Limiting

```python
import asyncio
import time
from collections import deque

class AsyncRateLimiter:
    def __init__(self, max_calls, period=1.0):
        self.max_calls = max_calls
        self.period = period
        self.calls = deque()

    async def acquire(self):
        now = time.monotonic()
        while self.calls and self.calls[0] <= now - self.period:
            self.calls.popleft()

        if len(self.calls) >= self.max_calls:
            wait_time = self.calls[0] + self.period - now
            if wait_time > 0:
                await asyncio.sleep(wait_time)
            self.calls.popleft()

        self.calls.append(time.monotonic())

    async def __aenter__(self):
        await self.acquire()
        return self

    async def __aexit__(self, *args):
        pass

async def limited_task(limiter, task_id):
    async with limiter:
        print(f"Task {task_id} executing at {time.strftime('%X')}")
        await asyncio.sleep(0.2)
        return task_id

async def main():
    limiter = AsyncRateLimiter(max_calls=3, period=1.0)
    tasks = [limited_task(limiter, i) for i in range(10)]
    results = await asyncio.gather(*tasks)
    print(f"Completed {len(results)} tasks with rate limiting")

    sem = asyncio.Semaphore(3)

    async def sem_task(id):
        async with sem:
            await asyncio.sleep(0.5)
            return f"Sem task {id}"

    results = await asyncio.gather(*[sem_task(i) for i in range(10)])
    print(f"Completed {len(results)} tasks with semaphore")

asyncio.run(main())
```

### Async Subprocess

```python
import asyncio
import sys

async def run_command(cmd, *args):
    print(f"Running: {cmd} {' '.join(args)}")
    proc = await asyncio.create_subprocess_exec(
        cmd, *args,
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE
    )

    stdout, stderr = await proc.communicate()
    return proc.returncode, stdout.decode(), stderr.decode()

async def stream_command(cmd, *args):
    print(f"Streaming: {cmd} {' '.join(args)}")
    proc = await asyncio.create_subprocess_exec(
        cmd, *args,
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE
    )

    async for line in proc.stdout:
        print(f"  [stdout] {line.decode().strip()}")

    await proc.wait()
    return proc.returncode

async def main():
    code, stdout, stderr = await run_command(
        sys.executable, "-c", "print('Hello from subprocess'); import sys; sys.exit(0)"
    )
    print(f"Exit code: {code}, Output: {stdout.strip()}")

    try:
        code = await asyncio.wait_for(
            run_command(sys.executable, "-c", "import time; time.sleep(10)"),
            timeout=2
        )
    except asyncio.TimeoutError:
        print("Subprocess timed out")

    print("\nShell command:")
    proc = await asyncio.create_subprocess_shell(
        f'echo "Hello from shell"',
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE
    )
    stdout, _ = await proc.communicate()
    print(f"Shell output: {stdout.decode().strip()}")

    code = await stream_command(sys.executable, "-c", """
for i in range(5):
    print(f"Line {i}")
""")
    print(f"Stream exit code: {code}")

asyncio.run(main())
```

### asyncio.Lock, Event, Condition

```python
import asyncio
import random

async def worker_lock(name, lock, shared_list):
    async with lock:
        print(f"{name}: acquired lock")
        await asyncio.sleep(random.uniform(0.1, 0.3))
        shared_list.append(name)
        print(f"{name}: released lock")

async def worker_event(name, event):
    print(f"{name}: waiting for event")
    await event.wait()
    print(f"{name}: event received!")

async def producer_condition(condition, queue):
    for i in range(5):
        async with condition:
            await asyncio.sleep(0.2)
            queue.append(f"item-{i}")
            print(f"Produced: item-{i}")
            condition.notify_all()

async def consumer_condition(name, condition, queue):
    while True:
        async with condition:
            while not queue:
                await condition.wait()
            item = queue.pop(0)
        if item is None:
            break
        print(f"Consumer {name}: got {item}")
        await asyncio.sleep(random.uniform(0.1, 0.3))

async def main():
    shared_list = []
    lock = asyncio.Lock()
    await asyncio.gather(*[
        worker_lock(f"W-{i}", lock, shared_list)
        for i in range(5)
    ])
    print(f"Shared list: {shared_list}")

    event = asyncio.Event()
    await asyncio.sleep(0.3)
    event.set()
    await asyncio.gather(*[
        worker_event(f"E-{i}", event)
        for i in range(3)
    ])

    condition = asyncio.Condition()
    queue = []
    await asyncio.gather(
        producer_condition(condition, queue),
        consumer_condition("C1", condition, queue),
        consumer_condition("C2", condition, queue),
    )

asyncio.run(main())
```

## Beginner Examples

```python
import asyncio

# Async file reader using async generator
async def read_chunks(filename, chunk_size=1024):
    loop = asyncio.get_event_loop()
    import tempfile, os

    with tempfile.NamedTemporaryFile(mode='wb', delete=False) as f:
        f.write(b"x" * 5000)
        tmpname = f.name

    try:
        def sync_read():
            with open(tmpname, 'rb') as f:
                while True:
                    chunk = f.read(chunk_size)
                    if not chunk:
                        break
                    yield chunk

        for chunk in sync_read():
            yield chunk
            await asyncio.sleep(0.01)
    finally:
        os.unlink(tmpname)

async def main():
    count = 0
    async for chunk in read_chunks("dummy.txt", 1024):
        count += len(chunk)
    print(f"Read {count} bytes via async generator")

asyncio.run(main())

# Async timeout decorator
def async_timeout(seconds):
    def decorator(coro_func):
        async def wrapper(*args, **kwargs):
            try:
                return await asyncio.wait_for(coro_func(*args, **kwargs), timeout=seconds)
            except asyncio.TimeoutError:
                return None
        return wrapper
    return decorator

@async_timeout(2)
async def slow_function(delay):
    await asyncio.sleep(delay)
    return "Completed"

async def main2():
    result1 = await slow_function(1)
    result2 = await slow_function(3)
    print(f"Fast call (1s delay, 2s timeout): {result1}")
    print(f"Slow call (3s delay, 2s timeout): {result2}")

asyncio.run(main2())
```

## Intermediate Examples

```python
import asyncio
import time

# Async task supervisor with health checks
class TaskSupervisor:
    def __init__(self, check_interval=5):
        self.tasks = {}
        self.check_interval = check_interval
        self.running = True

    async def add_task(self, name, coro_factory, restart=True):
        task = asyncio.create_task(self._run_with_supervision(name, coro_factory, restart))
        self.tasks[name] = task
        return task

    async def _run_with_supervision(self, name, coro_factory, restart):
        while self.running:
            try:
                print(f"Supervisor: starting {name}")
                await coro_factory()
            except asyncio.CancelledError:
                print(f"Supervisor: {name} cancelled")
                break
            except Exception as e:
                print(f"Supervisor: {name} failed with {e}")
                if restart:
                    print(f"Supervisor: restarting {name} in 1s...")
                    await asyncio.sleep(1)
                else:
                    break

    async def health_check(self):
        while self.running:
            await asyncio.sleep(self.check_interval)
            alive = sum(1 for t in self.tasks.values() if not t.done())
            print(f"Supervisor health: {alive}/{len(self.tasks)} tasks alive")

    async def shutdown(self):
        self.running = False
        for name, task in self.tasks.items():
            task.cancel()
        await asyncio.gather(*self.tasks.values(), return_exceptions=True)
        print("Supervisor: all tasks stopped")

async def worker_task(name):
    for i in range(3):
        print(f"Worker {name}: iteration {i}")
        await asyncio.sleep(0.5)
    if name == "faulty":
        raise RuntimeError("Worker failed!")

async def main():
    supervisor = TaskSupervisor(check_interval=2)
    supervisor_task = asyncio.create_task(supervisor.health_check())

    await supervisor.add_task("worker-1", lambda: worker_task("worker-1"))
    await supervisor.add_task("faulty", lambda: worker_task("faulty"))

    await asyncio.sleep(5)
    await supervisor.shutdown()
    supervisor_task.cancel()

    print("Supervisor example completed")

asyncio.run(main())

# Async event bus with pattern matching
import re

class AsyncEventBus:
    def __init__(self):
        self._subscribers = []
        self._lock = asyncio.Lock()

    def subscribe(self, pattern, callback):
        self._subscribers.append((re.compile(pattern), callback))

    async def publish(self, event_type, data=None):
        async with self._lock:
            for pattern, callback in self._subscribers:
                if pattern.match(event_type):
                    await callback(event_type, data)

    async def publish_many(self, events):
        for event_type, data in events:
            await self.publish(event_type, data)

async def on_user_event(event_type, data):
    print(f"User event: {event_type} - {data}")
    await asyncio.sleep(0.1)

async def on_system_event(event_type, data):
    print(f"System event: {event_type} - {data}")

async def on_any_event(event_type, data):
    if data:
        pass

async def main2():
    bus = AsyncEventBus()
    bus.subscribe(r"user\.\w+", on_user_event)
    bus.subscribe(r"system\.\w+", on_system_event)
    bus.subscribe(r".*", on_any_event)

    events = [
        ("user.login", {"user_id": 1}),
        ("user.logout", {"user_id": 1}),
        ("system.error", {"code": 500}),
        ("system.health", {"status": "ok"}),
        ("data.update", {"table": "users"}),
    ]

    await bus.publish_many(events)
    print("Event bus example completed")

asyncio.run(main2())
```

## Advanced Examples

```python
import asyncio
import time
import random
from typing import Any, Callable, Dict, List, Optional
from dataclasses import dataclass, field

@dataclass
class CircuitBreakerState:
    failures: int = 0
    last_failure_time: float = 0
    state: str = "CLOSED"

class AsyncCircuitBreaker:
    def __init__(
        self,
        failure_threshold: int = 5,
        recovery_timeout: float = 30.0,
        half_open_max_requests: int = 3
    ):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.half_open_max_requests = half_open_max_requests
        self._state = CircuitBreakerState()
        self._half_open_count = 0
        self._lock = asyncio.Lock()

    async def call(self, coro_factory: Callable, *args, **kwargs) -> Any:
        async with self._lock:
            if self._state.state == "OPEN":
                if time.monotonic() - self._state.last_failure_time >= self.recovery_timeout:
                    self._state.state = "HALF_OPEN"
                    self._half_open_count = 0
                    print("Circuit: OPEN -> HALF_OPEN")
                else:
                    raise CircuitBreakerOpenError("Circuit breaker is OPEN")

            if self._state.state == "HALF_OPEN" and self._half_open_count >= self.half_open_max_requests:
                raise CircuitBreakerOpenError("Half-open max requests reached")

        try:
            result = await coro_factory(*args, **kwargs)
            async with self._lock:
                if self._state.state == "HALF_OPEN":
                    self._state.state = "CLOSED"
                    self._state.failures = 0
                    print("Circuit: HALF_OPEN -> CLOSED (success)")
                elif self._state.state == "CLOSED":
                    self._state.failures = 0
            return result
        except Exception as e:
            async with self._lock:
                self._state.failures += 1
                self._state.last_failure_time = time.monotonic()
                if self._state.failures >= self.failure_threshold:
                    self._state.state = "OPEN"
                    self._half_open_count = 0
                    print(f"Circuit: CLOSED -> OPEN ({self._state.failures} failures)")
            raise

class CircuitBreakerOpenError(Exception):
    pass

async def unstable_service(name, fail_probability=0.4):
    await asyncio.sleep(random.uniform(0.1, 0.3))
    if random.random() < fail_probability:
        raise ConnectionError(f"{name}: transient error")
    return f"{name}: success"

async def circuit_breaker_demo():
    cb = AsyncCircuitBreaker(failure_threshold=3, recovery_timeout=2, half_open_max_requests=2)

    for i in range(20):
        try:
            result = await cb.call(unstable_service, f"request-{i}", 0.5)
            print(f"  Request {i}: {result}")
        except (CircuitBreakerOpenError, ConnectionError) as e:
            print(f"  Request {i}: {type(e).__name__}: {e}")
        await asyncio.sleep(0.1)

async def main():
    print("=== Async Circuit Breaker ===")
    await circuit_breaker_demo()

    # Async retry with exponential backoff and jitter
    print("\n=== Async Retry with Jitter ===")
    async def retry_with_jitter(coro_factory, max_retries=5, base_delay=0.1):
        for attempt in range(max_retries):
            try:
                return await coro_factory()
            except Exception as e:
                if attempt == max_retries - 1:
                    raise
                delay = base_delay * (2 ** attempt)
                jitter = random.uniform(0, delay * 0.5)
                total_delay = delay + jitter
                print(f"  Attempt {attempt + 1} failed. Retrying in {total_delay:.2f}s")
                await asyncio.sleep(total_delay)
        return None

    async def flaky_service():
        await asyncio.sleep(0.1)
        if random.random() < 0.7:
            raise TimeoutError("Service timeout")
        return "Service response"

    try:
        result = await retry_with_jitter(flaky_service, max_retries=4)
        print(f"Retry result: {result}")
    except Exception as e:
        print(f"All retries exhausted: {e}")

    # Async pipeline with backpressure
    print("\n=== Async Pipeline with Backpressure ===")
    MAX_SIZE = 3

    async def stage1(input_queue, output_queue):
        while True:
            try:
                item = await asyncio.wait_for(input_queue.get(), timeout=1)
            except asyncio.TimeoutError:
                break
            await asyncio.sleep(0.2)
            transformed = item * 2
            await output_queue.put(transformed)
            print(f"Stage1: {item} -> {transformed} (queue: ~{output_queue.qsize()})")

    async def stage2(input_queue, output_queue):
        while True:
            try:
                item = await asyncio.wait_for(input_queue.get(), timeout=1)
            except asyncio.TimeoutError:
                break
            await asyncio.sleep(0.3)
            transformed = f"P{item}"
            await output_queue.put(transformed)
            print(f"Stage2: {item} -> {transformed}")

    async def stage3(input_queue):
        results = []
        while True:
            try:
                item = await asyncio.wait_for(input_queue.get(), timeout=1)
            except asyncio.TimeoutError:
                break
            results.append(item)
            print(f"Stage3: collected {item}")
        return results

    q1, q2, q3 = asyncio.Queue(MAX_SIZE), asyncio.Queue(MAX_SIZE), asyncio.Queue()

    for i in range(8):
        await q1.put(i)

    s1 = asyncio.create_task(stage1(q1, q2))
    s2 = asyncio.create_task(stage2(q2, q3))
    s3 = asyncio.create_task(stage3(q3))

    await asyncio.gather(s1, s2, s3)
    result = s3.result()
    print(f"Pipeline final results: {result}")

    # Async context manager for pooled connections
    print("\n=== Async Connection Pool ===")

    class AsyncConnection:
        def __init__(self, id):
            self.id = id
            self.in_use = False

        async def query(self, sql):
            await asyncio.sleep(0.1)
            return f"Conn-{self.id}: result for '{sql}'"

        async def close(self):
            await asyncio.sleep(0.05)

    class AsyncConnectionPool:
        def __init__(self, size=3):
            self._pool = asyncio.Queue(maxsize=size)
            for i in range(size):
                self._pool.put_nowait(AsyncConnection(i))
            self._sem = asyncio.Semaphore(size)

        async def acquire(self):
            await self._sem.acquire()
            conn = await self._pool.get()
            conn.in_use = True
            return conn

        async def release(self, conn):
            conn.in_use = False
            await self._pool.put(conn)
            self._sem.release()

        async def __aenter__(self):
            return await self.acquire()

        async def __aexit__(self, *args):
            await self.release(self._conn)
            self._conn = None

    pool = AsyncConnectionPool(3)

    async def use_pool(pool, id):
        async with pool as conn:
            result = await conn.query(f"SELECT {id}")
            print(f"  {result}")
            await asyncio.sleep(random.uniform(0.2, 0.5))

    await asyncio.gather(*[use_pool(pool, i) for i in range(8)])
    print("Connection pool example completed")

asyncio.run(main())
```

## Real-World Use Cases

- **Database connection pools**: Managing async database connections with automatic reconnection
- **Message brokers**: Async consumers with backpressure and dead-letter queues
- **Microservice orchestration**: Coordinating multiple async service calls with circuit breakers
- **Stream processing**: Processing data streams with async generators and backpressure
- **Real-time dashboards**: WebSocket connections broadcasting updates to thousands of clients
- **Task queues**: Distributed async task processing with priority queues and retries

## Common Mistakes

- Not using `async for` with async iterators (returns `__anext__` coroutine, not value)
- Forgetting that async generators don't support `yield from` (use `async for ...: yield value`)
- Using `asyncio.Queue.get()` without `task_done()` and `queue.join()`
- Blocking the event loop inside an async generator with `time.sleep()`
- Not handling `asyncio.CancelledError` in async context managers' `__aexit__`
- Overlooking the need for locks when sharing mutable state between coroutines
- Using `asyncio.subprocess` without `communicate()` leads to resource leaks
- Forgetting that `asyncio.create_subprocess_shell` has security implications

## Best Practices

- Use async context managers for any resource that needs cleanup (files, connections, locks)
- Use async generators for streaming data processing pipelines
- Use `asyncio.Queue` with `maxsize` for backpressure in producer-consumer patterns
- Implement circuit breakers and retry with jitter for resilient external service calls
- Use `asyncio.gather(return_exceptions=True)` to prevent one failure from killing all tasks
- Keep async generators pure (no side effects) for testability
- Use `asyncio.create_task()` with proper supervision rather than fire-and-forget

## Interview Questions

1. **Q**: How does an async context manager differ from a regular context manager?
   **A**: Async context managers use `__aenter__` and `__aexit__` coroutines instead of `__enter__`/`__exit__`. They are used with `async with` and allow await expressions during setup/teardown.

2. **Q**: What is the difference between an async generator and an async iterator?
   **A**: An async generator is defined with `async def` and `yield`, automatically implementing `__aiter__` and `__anext__`. An async iterator requires manually implementing `__aiter__` and `__anext__` as coroutines.

3. **Q**: How do you implement backpressure in an async pipeline?
   **A**: Use bounded `asyncio.Queue(maxsize=N)`. When the queue is full, producers block on `put()`, naturally limiting the production rate to match consumption.

4. **Q**: What is the purpose of `asyncio.subprocess` vs `asyncio.create_subprocess_exec`?
   **A**: Both are the same; `create_subprocess_exec` is the coroutine. The module also provides `create_subprocess_shell` for shell-based commands (with security caveats).

5. **Q**: How do you debug asyncio programs effectively?
   **A**: Enable debug mode with `PYTHONASYNCIODEBUG=1` or `asyncio.run(main(), debug=True)`. Use `loop.slow_callback_duration` (default 0.1s) to detect blocking callbacks. Enable `ResourceWarning` for detecting unclosed resources.

## Coding Challenges

1. **Async Task Scheduler**: Build a scheduler that runs tasks at specified times/cron expressions, with retry logic and notification callbacks.

2. **Streaming Log Aggregator**: Implement an async log pipeline that reads from multiple log streams, transforms lines, and writes to an output with backpressure handling.

3. **Async Web Crawler with Robots.txt**: Build a concurrent crawler that respects robots.txt caching, implements politeness delays, and uses a priority queue for URLs.

4. **Resilient Microservice Client**: Write an async HTTP client with circuit breaker, retry with exponential backoff, request deduplication, and timeout handling.

5. **Real-time Data Pipeline**: Create a multi-stage async data processing pipeline with backpressure, monitoring, and graceful shutdown.

## Summary

Advanced asyncio patterns provide the building blocks for production-grade asynchronous applications. Async context managers, iterators, and generators enable clean resource management and streaming. Synchronization primitives (locks, semaphores, events, conditions), queues with priorities, subprocess management, and circuit breakers handle real-world complexity. Debugging tools help diagnose performance issues and resource leaks.

## Related Topics

Async IO (61.x), Multithreading (59.x), Multiprocessing (60.x), Thread Safety (63.x)
