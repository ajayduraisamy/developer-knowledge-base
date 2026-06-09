# Asyncio Advanced - async generators, context managers, queues
## Introduction
Python's asyncio goes beyond basic coroutines with advanced constructs for managing asynchronous resources and data flow. Async generators enable streaming data with `async for`, async context managers handle setup/teardown with `async with`, and `asyncio.Queue` coordinates producer-consumer workflows in concurrent async code.

## Async generators
### What It Is
An async generator is a coroutine that yields values using `yield` inside an `async def` function. It combines the iteration protocol with asynchronous operations, enabling lazy production and consumption of data with I/O between yields.

### Why It Is Important
Async generators bridge the gap between infinite/streaming data sources and async consumers. They allow you to paginate through API results, stream data from databases, read files line-by-line, or generate real-time data — all without loading everything into memory.

### How It Works Internally
When `async for value in async_gen():` is executed, the event loop iterates the async generator protocol:
1. `__aiter__()` returns the async iterator
2. `__anext__()` returns an awaitable that, when awaited, runs the generator until the next `yield`
3. On `StopAsyncIteration`, the loop exits

The implementation uses the same suspension/resumption mechanism as coroutines, but with yield points that send values to the consumer.

### Syntax
```python
# Async generator definition
async def async_gen():
    for i in range(10):
        await asyncio.sleep(0.1)
        yield i

# Consumption
async for value in async_gen():
    print(value)

# Generator expression (Python 3.6+)
gen = (x async for x in async_stream())
```

### Beginner Examples
```python
import asyncio

async def countdown(start, delay=1):
    for i in range(start, 0, -1):
        await asyncio.sleep(delay)
        yield i

async def main():
    async for count in countdown(5, 0.5):
        print(f"Count: {count}")
    print("Blast off!")

asyncio.run(main())
```

### Intermediate Examples
```python
import asyncio

# Simulated async pagination
async def paginate(url_template, max_pages=3):
    for page in range(1, max_pages + 1):
        await asyncio.sleep(0.5)  # Simulate network request
        items = [f"{url_template}/item/{i}" for i in range((page-1)*10, page*10)]
        print(f"Fetched page {page}, {len(items)} items")
        yield items

async def main():
    all_items = []
    async for page_items in paginate("https://api.example.com/data"):
        all_items.extend(page_items)
    print(f"Total items: {len(all_items)}")

asyncio.run(main())

# Async generator with aiter and anext
async def numbers():
    for i in range(5):
        await asyncio.sleep(0.1)
        yield i

async def manual_iteration():
    gen = numbers().__aiter__()
    try:
        while True:
            value = await gen.__anext__()
            print(f"Manual next: {value}")
    except StopAsyncIteration:
        print("Generator exhausted")

asyncio.run(manual_iteration())
```

### Advanced Examples
```python
import asyncio
from typing import AsyncGenerator

# Async generator with error handling
async def resilient_stream() -> AsyncGenerator[int, None]:
    for i in range(10):
        try:
            await asyncio.sleep(0.1)
            if i == 5:
                raise ValueError("Error at 5")
            yield i
        except ValueError as e:
            print(f"Caught: {e}")
            yield -1  # Fallback value
            continue

async def consume_resilient():
    async for val in resilient_stream():
        print(f"Got: {val}")

asyncio.run(consume_resilient())

# Async generator for real-time data simulation
async def sensor_data(sensor_id: str, interval: float = 0.5):
    import random
    while True:
        await asyncio.sleep(interval)
        yield {
            "sensor": sensor_id,
            "value": random.uniform(20, 30),
            "timestamp": asyncio.get_running_loop().time(),
        }

async def monitor_sensors():
    # Create multiple sensor streams
    sensors = [sensor_data(f"S{i}", 0.3 + i * 0.1) for i in range(3)]
    
    # Merge them with asyncio.gather-like approach
    async def read_sensor(agen):
        async for data in agen:
            print(f"Sensor {data['sensor']}: {data['value']:.2f}")
            await asyncio.sleep(0)  # Yield to event loop
    
    # Run all sensor readers concurrently (but stop after 3 seconds)
    tasks = [asyncio.create_task(read_sensor(s)) for s in sensors]
    await asyncio.sleep(3)
    for t in tasks:
        t.cancel()
    await asyncio.gather(*tasks, return_exceptions=True)
    print("Monitoring stopped")

asyncio.run(monitor_sensors())

# Async generator with cleanup
async def managed_stream():
    try:
        for i in range(5):
            await asyncio.sleep(0.2)
            yield i
    finally:
        print("Cleanup: releasing resources")
        await asyncio.sleep(0.1)
        print("Cleanup done")

async def use_managed():
    gen = managed_stream()
    async for val in gen:
        print(f"Value: {val}")
        if val == 2:
            break  # Trigger cleanup via finally
    # Generator will be garbage collected, triggering cleanup

asyncio.run(use_managed())
```

### Real-World Use Cases
- **API pagination** — lazily fetching and yielding pages from REST APIs
- **File streaming** — reading large files line-by-line with async I/O
- **Database cursors** — yielding rows from async database queries
- **Real-time data feeds** — streaming sensor data, stock prices, or logs
- **WebSocket streams** — yielding messages as they arrive over WebSocket

### Common Mistakes
- Using `return` with a value in an async generator (use `return` alone to stop)
- Forgetting that `async for` requires `__aiter__` and `__anext__`
- Not handling cleanup in `finally` blocks (generator can be garbage collected)
- Mixing sync and async generators incorrectly

### Best Practices
- Use `typing.AsyncGenerator` for type annotations
- Clean up resources in `finally` blocks
- Use `async for` for consuming async generators
- Use `async` list comprehensions: `[x async for x in async_gen()]`
- Limit generator state to avoid memory leaks in infinite generators
- Handle `StopAsyncIteration` properly in manual iteration

### Performance Considerations
- Each `yield` in an async generator involves a coroutine suspension/resumption
- Overhead is slightly higher than a sync generator but much lower than threading
- Use `asyncio.sleep(0)` at yield points to allow other tasks to run
- Batching yields can reduce overhead (yield chunks instead of individual items)

### Interview Questions
1. What is the difference between a generator and an async generator?
2. How does `StopAsyncIteration` work in async generators?
3. Can you use `return` with a value in an async generator?
4. How do you type hint an async generator?

### Coding Challenges
- Write an async generator that fetches paginated API results
- Build a real-time log tailer using async generators
- Create a merge function that combines multiple async generators into one

### Related Topics
- asyncio, async iterators, coroutines, yield, Python generators

## Async context managers
### What It Is
An async context manager uses `__aenter__` and `__aexit__` methods (instead of `__enter__`/`__exit__`) for setup and teardown that involve asynchronous operations. The `async with` statement manages the lifecycle of such objects.

### Why It Is Important
Many async resources — network connections, file handles, database sessions, locks — require async setup and teardown. Async context managers provide the same RAII (Resource Acquisition Is Initialization) pattern as sync context managers, but for async operations.

### How It Works Internally
When `async with obj as x:` is executed:
1. `await obj.__aenter__()` is called (returns the resource)
2. The `as x` target is bound to the result
3. The body executes
4. `await obj.__aexit__(exc_type, exc_val, exc_tb)` is called on exit (even on exception)

The `contextlib` module provides `@asynccontextmanager` for creating async context managers from async generators.

### Syntax
```python
# Class-based
class AsyncResource:
    async def __aenter__(self):
        await self.connect()
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.close()

async with AsyncResource() as res:
    await res.work()

# Generator-based (contextlib)
from contextlib import asynccontextmanager

@asynccontextmanager
async def my_context():
    resource = await acquire()
    try:
        yield resource
    finally:
        await release(resource)
```

### Beginner Examples
```python
import asyncio
from contextlib import asynccontextmanager

# Generator-based async context manager
@asynccontextmanager
async def timed_operation(name):
    print(f"Starting {name}")
    start = asyncio.get_running_loop().time()
    try:
        yield
    finally:
        elapsed = asyncio.get_running_loop().time() - start
        print(f"{name} took {elapsed:.2f}s")

async def main():
    async with timed_operation("sleep operation"):
        await asyncio.sleep(1)
    
    # Nested contexts
    async with timed_operation("outer"):
        await asyncio.sleep(0.5)
        async with timed_operation("inner"):
            await asyncio.sleep(0.5)

asyncio.run(main())
```

### Intermediate Examples
```python
import asyncio
from contextlib import asynccontextmanager

# Async database connection pool simulation
class AsyncConnection:
    def __init__(self, name):
        self.name = name
        self.connected = False
    
    async def connect(self):
        await asyncio.sleep(0.5)
        self.connected = True
        print(f"Connected to {self.name}")
    
    async def close(self):
        await asyncio.sleep(0.2)
        self.connected = False
        print(f"Closed {self.name}")
    
    async def query(self, sql):
        if not self.connected:
            raise RuntimeError("Not connected")
        await asyncio.sleep(0.3)
        return f"Results for: {sql}"

class AsyncConnectionPool:
    def __init__(self, name):
        self.conn = AsyncConnection(name)
    
    async def __aenter__(self):
        await self.conn.connect()
        return self.conn
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.conn.close()

async def main():
    async with AsyncConnectionPool("mydb") as conn:
        result = await conn.query("SELECT * FROM users")
        print(result)

asyncio.run(main())

# Multiple resources with async context managers
@asynccontextmanager
async def connection(name):
    print(f"Acquiring {name}")
    await asyncio.sleep(0.3)
    try:
        yield f"conn-{name}"
    finally:
        await asyncio.sleep(0.2)
        print(f"Releasing {name}")

async def multi_resource():
    # Using asynccontextmanager with multiple contexts
    async with connection("db1") as c1, connection("db2") as c2:
        print(f"Working with {c1} and {c2}")

asyncio.run(multi_resource())
```

### Advanced Examples
```python
import asyncio
from contextlib import asynccontextmanager
from typing import AsyncIterator

# Async context manager with error handling
@asynccontextmanager
async def managed_transaction(dsn: str) -> AsyncIterator[str]:
    txn_id = f"txn-{id(dsn)}"
    print(f"BEGIN transaction {txn_id}")
    await asyncio.sleep(0.2)
    
    try:
        yield txn_id
    except Exception as e:
        print(f"ROLLBACK transaction {txn_id}: {e}")
        await asyncio.sleep(0.1)
        raise  # Re-raise after rollback
    else:
        print(f"COMMIT transaction {txn_id}")
        await asyncio.sleep(0.1)

async def main():
    # Successful transaction
    async with managed_transaction("db://local") as txn:
        print(f"Working in {txn}")
        await asyncio.sleep(0.3)
    
    # Failing transaction (rolls back)
    try:
        async with managed_transaction("db://local") as txn:
            print(f"Working in {txn}")
            raise ValueError("Constraint violation")
    except ValueError:
        print("Caught transaction error")

asyncio.run(main())

# Reusable async context manager (class-based with state)
class AsyncFileReader:
    def __init__(self, path):
        self.path = path
        self.content = None
    
    async def __aenter__(self):
        await asyncio.sleep(0.1)  # Simulate open
        with open(self.path) as f:
            self.content = f.read()
        return self.content
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        self.content = None
        await asyncio.sleep(0.1)  # Simulate close

# asyncio.run(AsyncFileReader("test.txt").__aenter__())

# AsyncExitStack — dynamic context management
from contextlib import AsyncExitStack

async def dynamic_resources():
    async with AsyncExitStack() as stack:
        # Push contexts dynamically
        c1 = await stack.enter_async_context(connection("db1"))
        c2 = await stack.enter_async_context(connection("db2"))
        
        # Push callbacks for cleanup
        stack.push_async_callback(lambda: asyncio.sleep(0.1))
        
        print(f"Using {c1} and {c2}")
    # All contexts cleaned up on exit from stack

asyncio.run(dynamic_resources())
```

### Real-World Use Cases
- **Database sessions** — async session begin/commit/rollback (SQLAlchemy async)
- **HTTP clients** — client session setup/takedown (aiohttp.ClientSession)
- **File I/O** — async file open/close (aiofiles)
- **Lock management** — async lock acquire/release
- **Network connections** — WebSocket connect/disconnect

### Common Mistakes
- Using `@contextmanager` instead of `@asynccontextmanager` for async contexts
- Forgetting to `await` the `__aenter__` or `__aexit__` (implicit in `async with`)
- Not handling exceptions in `__aexit__` (re-raising is handled automatically)
- Mixing sync and async cleanup code in the same context manager
- Yielding multiple times in an asynccontextmanager (yield at most once)

### Best Practices
- Use `@asynccontextmanager` for simple cases, class-based for complex state
- Always clean up resources in `finally` blocks (both generator and class-based)
- Use `AsyncExitStack` for dynamically adding contexts
- Handle exceptions in `__aexit__` and return True to suppress them
- Use `contextlib.suppress` for known and expected errors
- Document the async nature of the context manager in docstrings

### Performance Considerations
- Async context managers add two coroutine suspension points (enter/exit)
- The overhead is minimal compared to the actual async operations
- `@asynccontextmanager` has slightly more overhead than class-based
- Nested `async with` blocks don't add significant overhead

### Interview Questions
1. What is the difference between `__aenter__`/`__aexit__` and `__enter__`/`__exit__`?
2. How does `@asynccontextmanager` work internally?
3. What is `AsyncExitStack` and when would you use it?
4. How do you suppress exceptions in an async context manager?

### Coding Challenges
- Write an async context manager that measures execution time of the enclosed block
- Build an async connection pool with context manager interface
- Create an async file reader that reports progress as it reads

### Related Topics
- contextlib, asynccontextmanager, AsyncExitStack, async with, RAII

## asyncio.Queue
### What It Is
`asyncio.Queue` is a coroutine-based FIFO queue for coordinating async producers and consumers. It provides `await put()` and `await get()` methods that suspend when the queue is full or empty, respectively.

### Why It Is Important
`asyncio.Queue` is the standard way to implement producer-consumer patterns in async code. Unlike `queue.Queue`, it doesn't use threading primitives — it integrates with the event loop for efficient, non-blocking coordination.

### How It Works Internally
`asyncio.Queue` uses `collections.deque` for storage and manages waiting producers/consumers via internal lists. When `put()` encounters a full queue, it creates a Future and suspends until a consumer calls `get()`. When `get()` finds an empty queue, it suspends until an item is available. No OS threads or locks are involved — the event loop handles the suspension.

### Syntax
```python
from asyncio import Queue, PriorityQueue, LifoQueue

q = Queue(maxsize=100)

await q.put(item)          # suspend if full
item = await q.get()       # suspend if empty
q.put_nowait(item)         # raises QueueFull
q.get_nowait()             # raises QueueEmpty
q.task_done()              # mark task processed
await q.join()             # wait until all items processed
q.qsize()                  # approximate size
q.empty() / q.full()
```

### Beginner Examples
```python
import asyncio
import random

async def producer(q: asyncio.Queue, name: str):
    for i in range(10):
        item = f"{name}-{i}"
        await q.put(item)
        print(f"Produced: {item}")
        await asyncio.sleep(random.uniform(0.1, 0.3))

async def consumer(q: asyncio.Queue, name: str):
    while True:
        item = await q.get()
        print(f"Consumer {name} got: {item}")
        await asyncio.sleep(random.uniform(0.2, 0.5))
        q.task_done()

async def main():
    q = asyncio.Queue(maxsize=5)
    
    producers = [asyncio.create_task(producer(q, f"P{i}")) for i in range(2)]
    consumers = [asyncio.create_task(consumer(q, f"C{i}")) for i in range(3)]
    
    await asyncio.gather(*producers)
    await q.join()  # Wait for all items to be processed
    
    # Cancel consumers
    for c in consumers:
        c.cancel()
    
    print("All done")

asyncio.run(main())
```

### Intermediate Examples
```python
import asyncio

# Queue with poison pills
async def worker(name, q):
    while True:
        item = await q.get()
        if item is None:  # Poison pill
            q.task_done()
            break
        await asyncio.sleep(0.3)
        print(f"Worker {name} processed: {item}")
        q.task_done()

async def main():
    q = asyncio.Queue()
    
    # Add work items
    for i in range(20):
        await q.put(f"task-{i}")
    
    # Add poison pills (one per worker)
    workers = [asyncio.create_task(worker(f"W{i}", q)) for i in range(4)]
    for _ in workers:
        await q.put(None)
    
    await asyncio.gather(*workers)
    print("All workers stopped")

asyncio.run(main())

# PriorityQueue example
async def priority_example():
    pq = asyncio.PriorityQueue()
    
    # Put items with priority (lower number = higher priority)
    await pq.put((3, "low priority"))
    await pq.put((1, "high priority"))
    await pq.put((2, "medium priority"))
    
    while not pq.empty():
        priority, task = await pq.get()
        print(f"Processing: {task} (priority {priority})")
        pq.task_done()

asyncio.run(priority_example())

# Queue with timeout
async def get_with_timeout(q, timeout):
    try:
        item = await asyncio.wait_for(q.get(), timeout=timeout)
        q.task_done()
        return item
    except asyncio.TimeoutError:
        print("Get timed out")
        return None
```

### Advanced Examples
```python
import asyncio
import random

# Work distribution with Queue
async def crawler_worker(worker_id, url_queue, result_queue):
    while True:
        url = await url_queue.get()
        if url is None:
            url_queue.task_done()
            break
        
        # Simulate crawling
        await asyncio.sleep(random.uniform(0.2, 0.8))
        result = {
            "url": url,
            "worker": worker_id,
            "title": f"Title for {url}",
            "links": [f"{url}/link1", f"{url}/link2"],
        }
        await result_queue.put(result)
        url_queue.task_done()

async def result_collector(result_queue, max_results=20):
    results = []
    while len(results) < max_results:
        result = await result_queue.get()
        results.append(result)
        print(f"Collected: {result['url']} by worker {result['worker']}")
        result_queue.task_done()
    return results

async def main():
    url_queue = asyncio.Queue()
    result_queue = asyncio.Queue()
    
    # Seed URLs
    urls = [f"https://example.com/page/{i}" for i in range(50)]
    for url in urls:
        await url_queue.put(url)
    
    # Start workers
    workers = [asyncio.create_task(crawler_worker(i, url_queue, result_queue)) for i in range(5)]
    collector = asyncio.create_task(result_collector(result_queue, 20))
    
    # Add poison pills
    for _ in workers:
        await url_queue.put(None)
    
    await asyncio.gather(*workers)
    results = await collector
    
    print(f"Total results: {len(results)}")

asyncio.run(main())

# Throttled Queue with rate limiter
class ThrottledQueue(asyncio.Queue):
    def __init__(self, maxsize=0, rate_per_second=10):
        super().__init__(maxsize)
        self.rate_per_second = rate_per_second
        self._timestamps = []
    
    async def put(self, item):
        now = asyncio.get_running_loop().time()
        # Clean old timestamps
        self._timestamps = [t for t in self._timestamps if now - t < 1.0]
        
        if len(self._timestamps) >= self.rate_per_second:
            wait = 1.0 - (now - self._timestamps[0])
            if wait > 0:
                await asyncio.sleep(wait)
        
        self._timestamps.append(asyncio.get_running_loop().time())
        await super().put(item)

async def main():
    q = ThrottledQueue(maxsize=10, rate_per_second=5)
    
    async def producer():
        for i in range(15):
            await q.put(f"item-{i}")
            print(f"Produced item-{i}")
    
    async def consumer():
        async def consume():
            while True:
                item = await q.get()
                print(f"  Consumed: {item}")
                q.task_done()
                await asyncio.sleep(0.3)
        task = asyncio.create_task(consume())
        await asyncio.sleep(5)
        task.cancel()
    
    await asyncio.gather(producer(), consumer())

asyncio.run(main())
```

### Real-World Use Cases
- **Web crawlers** — coordinating URL fetching across multiple workers
- **Data processing pipelines** — buffering between stages with backpressure
- **Rate-limited API clients** — queuing requests to respect rate limits
- **Log aggregation** — collecting log entries from multiple producers
- **Task schedulers** — distributing jobs to worker coroutines

### Common Mistakes
- Using `Queue` without `task_done()`/`join()` (loses completion tracking)
- Forgetting that `get()` blocks forever if queue is empty and no more items arrive
- Not using poison pills to gracefully stop consumer tasks
- Using `qsize()` for synchronization decisions (not reliable)
- Confusing `asyncio.Queue` with `queue.Queue` (different blocking semantics)

### Best Practices
- Always call `task_done()` after processing each `get()` item
- Use `join()` to wait for all items to be processed
- Use poison pills (sentinel values) to signal workers to stop
- Use `maxsize` to prevent unbounded memory growth
- Use `PriorityQueue` when task ordering matters
- Use `LifoQueue` for LIFO patterns (stack)
- Use with `asyncio.gather()` or `asyncio.create_task()` for worker management
- Handle `CancelledError` in workers for clean shutdown

### Performance Considerations
- Queue operations are very fast (mostly Python-level operations)
- Suspension on full/empty uses Futures, not polling (zero CPU while waiting)
- Large `maxsize` uses more memory but reduces producer contention
- `task_done()`/`join()` pair adds synchronization overhead
- `PriorityQueue` uses a heap — O(log n) for put/get

### Interview Questions
1. How does `asyncio.Queue` differ from `queue.Queue` in threading?
2. What is the purpose of `task_done()` and `join()`?
3. How do you implement a bounded queue with backpressure?
4. What is the poison pill pattern and how do you implement it?
5. How do you add timeouts to queue operations?

### Coding Challenges
- Build a web crawler with asyncio.Queue that limits concurrent requests
- Implement a rate-limited API client using a throttled queue
- Create a data processing pipeline with multiple stages connected by queues
- Write a priority-based task scheduler using asyncio.PriorityQueue

### Related Topics
- asyncio, producer-consumer, queue module, async streams, backpressure
