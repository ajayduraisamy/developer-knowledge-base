# Async IO - async/await, asyncio, coroutines, event loop (Python 3.5+)
## Introduction
Async IO (asynchronous I/O) is a programming paradigm that enables concurrent execution of I/O-bound tasks using cooperative multitasking. Python's `asyncio` module, introduced in Python 3.4 and stabilized with `async`/`await` syntax in Python 3.5, provides the framework for writing concurrent code using coroutines, event loops, and futures.

## async/await
### What It Is
`async` and `await` are keywords introduced in Python 3.5 (PEP 492) for defining and invoking coroutines. `async def` declares a coroutine function, and `await` suspends execution until the awaited awaitable completes, yielding control back to the event loop.

### Why It Is Important
`async`/`await` makes asynchronous code read like synchronous code, eliminating callback hell and complex state machines. It provides a structured, readable way to write concurrent I/O code without threads or processes.

### How It Works Internally
When a coroutine hits `await`, it suspends execution, yielding a `coroutine` object that wraps the awaitable. The event loop receives this object, registers a callback for the underlying I/O operation, and switches to another ready coroutine. When the I/O completes, the event loop resumes the coroutine from the suspension point. This is implemented via Python's generator-based coroutine protocol (PEP 342/380) under the hood.

### Syntax
```python
async def my_coroutine():
    result = await some_awaitable()
    return result

# Awaitable types:
# - Another coroutine: await another_coro()
# - asyncio.Task: await asyncio.create_task(coro())
# - asyncio.Future: await future
# - asyncio.sleep(): await asyncio.sleep(1)
```

### Beginner Examples
```python
import asyncio

async def greet(name, delay):
    await asyncio.sleep(delay)
    print(f"Hello, {name}!")

async def main():
    # Run concurrently
    await asyncio.gather(
        greet("Alice", 2),
        greet("Bob", 1),
        greet("Charlie", 0.5),
    )
    print("All greetings done!")

asyncio.run(main())
```

### Intermediate Examples
```python
import asyncio
import time

async def fetch_data(url, delay):
    print(f"Fetching {url}...")
    await asyncio.sleep(delay)  # Simulate network I/O
    data = f"Data from {url}"
    print(f"Finished {url}")
    return data

async def main():
    # Sequential execution
    start = time.time()
    sequential = await asyncio.gather(
        fetch_data("url1", 2),
        fetch_data("url2", 1),
        fetch_data("url3", 0.5),
    )
    seq_time = time.time() - start
    print(f"Sequential time: {seq_time:.2f}s")
    print(f"Results: {sequential}")

    # Compare with await one by one (sequential)
    start = time.time()
    r1 = await fetch_data("url4", 2)
    r2 = await fetch_data("url5", 1)
    r3 = await fetch_data("url6", 0.5)
    seq_time2 = time.time() - start
    print(f"Await one-by-one time: {seq_time2:.2f}s (total of delays)")

asyncio.run(main())
```

### Advanced Examples
```python
import asyncio
import functools

# Creating and awaiting tasks explicitly
async def background_task(name):
    for i in range(3):
        await asyncio.sleep(1)
        print(f"Background {name}: tick {i}")
    return f"{name} done"

async def main():
    # Create tasks for concurrent execution
    task1 = asyncio.create_task(background_task("T1"))
    task2 = asyncio.create_task(background_task("T2"))
    
    print("Main: doing other work...")
    await asyncio.sleep(1.5)
    
    # Await tasks later
    result1 = await task1
    result2 = await task2
    print(f"Results: {result1}, {result2}")

asyncio.run(main())

# Creating multiple tasks with list comprehension
async def fetch_all(urls):
    async def fetch_one(url):
        await asyncio.sleep(1)
        return f"Data:{url}"
    
    tasks = [asyncio.create_task(fetch_one(url)) for url in urls]
    return await asyncio.gather(*tasks)

# Task cancellation
async def cancellable_work():
    try:
        for i in range(10):
            await asyncio.sleep(0.5)
            print(f"Working... {i}")
    except asyncio.CancelledError:
        print("Work was cancelled!")
        raise  # Must re-raise or handle cleanly

async def cancel_example():
    task = asyncio.create_task(cancellable_work())
    await asyncio.sleep(1.2)
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("Cancellation acknowledged")

asyncio.run(cancel_example())
```

### Real-World Use Cases
- **Web servers** — handling thousands of concurrent connections (FastAPI, aiohttp)
- **Web scraping** — fetching multiple pages concurrently
- **Database access** — async ORM queries (SQLAlchemy async, asyncpg)
- **Real-time services** — chat servers, WebSocket handlers
- **Microservices** — coordinating multiple external API calls

### Common Mistakes
- Blocking the event loop with synchronous calls (`time.sleep()` instead of `asyncio.sleep()`)
- Forgetting to `await` a coroutine (creates a warning, coroutine is garbage collected)
- Mixing sync and async code incorrectly (calling async from sync without `run()`)
- Creating too many tasks (resource exhaustion on connections/file handles)
- Not handling `CancelledError` properly

### Best Practices
- Use `asyncio.run()` as the main entry point (Python 3.7+)
- Use `asyncio.create_task()` for fire-and-forget or concurrent execution
- Use `asyncio.gather()` for collecting results from multiple coroutines
- Avoid blocking calls in async functions (use async libraries)
- Use `asyncio.timeout()` (Python 3.11+) or `asyncio.wait_for()` for timeouts
- Handle `asyncio.CancelledError` for clean shutdown
- Use `try/finally` or `AsyncExitStack` for resource cleanup

### Performance Considerations
- Async I/O excels at I/O-bound tasks (network, disk, sleep)
- Not suitable for CPU-bound tasks (use multiprocessing)
- Context switching between coroutines is very fast (microseconds)
- The event loop adds minimal overhead when tasks are I/O waiting
- Large numbers of concurrent tasks (10k+) are feasible

### Interview Questions
1. What is the difference between a coroutine and a task?
2. What does `await` actually do at the execution level?
3. How does `asyncio.gather` differ from individual awaits?
4. What is `CancelledError` and how should you handle it?
5. Can you call an async function from a sync context?

### Coding Challenges
- Write a coroutine that fetches multiple URLs concurrently and returns their lengths
- Implement a simple rate limiter using async/await
- Build a concurrent web scraper that processes pages as they arrive

### Related Topics
- asyncio, coroutines, event loop, futures, async context managers

## asyncio module
### What It Is
The `asyncio` module is the standard library's asynchronous I/O framework. It provides the event loop, coroutine runners, synchronization primitives (Lock, Semaphore, Event), streams, subprocess support, and queue implementations.

### Why It Is Important
`asyncio` is the foundation for all async Python code. It provides the infrastructure for writing concurrent, non-blocking I/O programs and is the basis for popular libraries like aiohttp, FastAPI, asyncpg, and aioredis.

### How It Works Internally
The core of `asyncio` is the event loop (`SelectorEventLoop` on Unix, `ProactorEventLoop` on Windows). It uses `selectors` module (wrapping `epoll`/`kqueue`/`IOCP`) to monitor file descriptors for I/O readiness. When a coroutine awaits I/O, the event loop registers the file descriptor with the selector and suspends the coroutine. When the selector signals readiness, the event loop resumes the corresponding coroutine.

### Syntax
```python
import asyncio

# Running
asyncio.run(main())                    # Python 3.7+
loop = asyncio.new_event_loop()        # lower-level
asyncio.set_event_loop(loop)

# Sleeping
await asyncio.sleep(0.1)

# Waiting
await asyncio.gather(*coros)           # all complete
await asyncio.wait(tasks, timeout=5)   # done, pending sets
await asyncio.wait_for(coro, timeout=5)  # with timeout

# Shield (prevent cancellation)
await asyncio.shield(coro)

# Timeout (Python 3.11+)
async with asyncio.timeout(5):
    await coro()
```

### Beginner Examples
```python
import asyncio

async def say_after(delay, msg):
    await asyncio.sleep(delay)
    print(msg)

async def main():
    print("Start")
    
    # Sequential
    await say_after(1, "First")
    await say_after(2, "Second")
    
    # Concurrent via gather
    await asyncio.gather(
        say_after(1, "Concurrent 1"),
        say_after(2, "Concurrent 2"),
        say_after(3, "Concurrent 3"),
    )
    
    print("Done")

asyncio.run(main())
```

### Intermediate Examples
```python
import asyncio

# Using asyncio.wait with different return conditions
async def worker(name, delay):
    await asyncio.sleep(delay)
    return f"{name}: done after {delay}s"

async def wait_examples():
    tasks = [
        asyncio.create_task(worker("A", 1)),
        asyncio.create_task(worker("B", 2)),
        asyncio.create_task(worker("C", 3)),
    ]
    
    # Wait for FIRST_COMPLETED
    done, pending = await asyncio.wait(tasks, return_when=asyncio.FIRST_COMPLETED)
    for t in done:
        print(f"First done: {t.result()}")
    
    # Cancel remaining
    for t in pending:
        t.cancel()

asyncio.run(wait_examples())

# Using asyncio.wait_for for timeouts
async def slow_operation():
    await asyncio.sleep(10)
    return "Done"

async def with_timeout():
    try:
        result = await asyncio.wait_for(slow_operation(), timeout=2)
        print(result)
    except asyncio.TimeoutError:
        print("Operation timed out!")

asyncio.run(with_timeout())

# asyncio.as_completed
async def fetch_urls():
    urls = ["A", "B", "C"]
    
    async def fetch(url):
        await asyncio.sleep(hash(url) % 3 + 0.5)
        return url
    
    tasks = [asyncio.create_task(fetch(u)) for u in urls]
    for coro in asyncio.as_completed(tasks):
        result = await coro
        print(f"Got: {result}")  # Results in completion order

asyncio.run(fetch_urls())

# Running blocking code in executor
import time

async def main_with_executor():
    loop = asyncio.get_running_loop()
    
    # Run blocking code in a thread pool
    result = await loop.run_in_executor(
        None,  # default executor (ThreadPoolExecutor)
        lambda: time.sleep(2) or "Blocking result"
    )
    print(result)

asyncio.run(main_with_executor())
```

### Advanced Examples
```python
import asyncio
import signal

# Graceful shutdown example
async def server_handler(name):
    try:
        while True:
            await asyncio.sleep(1)
            print(f"Server {name}: running")
    except asyncio.CancelledError:
        print(f"Server {name}: shutting down cleanly...")
        await asyncio.sleep(0.5)  # Cleanup
        print(f"Server {name}: shutdown complete")

async def main():
    servers = [asyncio.create_task(server_handler(f"SRV-{i}")) for i in range(3)]
    
    # Simulate running for 3 seconds
    await asyncio.sleep(3)
    
    print("Initiating shutdown...")
    for s in servers:
        s.cancel()
    
    await asyncio.gather(*servers, return_exceptions=True)
    print("All servers shut down")

asyncio.run(main())

# asyncio subprocess (Python 3.8+)
async def run_command(cmd):
    proc = await asyncio.create_subprocess_shell(
        cmd,
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE,
    )
    stdout, stderr = await proc.communicate()
    return proc.returncode, stdout.decode(), stderr.decode()

async def run_cmds():
    results = await asyncio.gather(
        run_command("echo hello"),
        run_command("echo world"),
    )
    for rc, out, err in results:
        print(f"rc={rc}, out={out.strip()}, err={err.strip()}")

asyncio.run(run_cmds())

# Synchronization primitives
async def sync_example():
    lock = asyncio.Lock()
    sem = asyncio.Semaphore(3)
    event = asyncio.Event()
    
    async def limited_work(i):
        async with sem:
            await asyncio.sleep(1)
            return i
    
    results = await asyncio.gather(*[limited_work(i) for i in range(10)])
    print(results)

asyncio.run(sync_example())

# asyncio.gather with return_exceptions
async def might_fail(x):
    if x == 5:
        raise ValueError("Bad value")
    await asyncio.sleep(0.1)
    return x * 2

async def gather_with_errors():
    results = await asyncio.gather(
        *[might_fail(i) for i in range(10)],
        return_exceptions=True
    )
    for i, r in enumerate(results):
        if isinstance(r, Exception):
            print(f"Task {i} failed: {r}")
        else:
            print(f"Task {i}: {r}")

asyncio.run(gather_with_errors())
```

### Real-World Use Cases
- **Async web servers** — FastAPI, Sanic, aiohttp
- **Database clients** — asyncpg, aiomysql, redis-py async
- **Message queues** — aio-pika, aiokafka
- **Network clients** — aiohttp client, httpx (async mode)
- **Async file I/O** — aiofiles (wraps sync file operations in executor)

### Common Mistakes
- Calling `loop.run_until_complete()` on a stopped loop
- Not using `asyncio.run()` correctly (only call once per Python process)
- Mixing different event loop policies (need to set before asyncio.run())
- Using `asyncio.wait` with `FIRST_EXCEPTION` (not a valid constant)
- Forgetting `return_exceptions=True` in gather (one exception cancels others)

### Best Practices
- Use `asyncio.run()` as the single entry point
- Use `create_task()` for fire-and-forget background tasks
- Use `gather()` for collecting results from concurrent operations
- Use `wait_for()` or `timeout()` (Python 3.11+) to prevent hangs
- Use `run_in_executor()` for blocking code that can't be made async
- Use `asyncio.shield()` to protect critical sections from cancellation
- Use `return_exceptions=True` in gather when some tasks may fail
- Use `asyncio.Queue` for producer-consumer patterns

### Performance Considerations
- The event loop can handle 10k+ concurrent connections
- Each coroutine yields at `await` points — minimal overhead
- `asyncio.run_in_executor` adds thread pool overhead for blocking code
- `asyncio.gather` is more efficient than multiple `await` statements
- Subprocess in asyncio uses threads under the hood on some platforms

### Interview Questions
1. How does `asyncio.run()` differ from manually managing the event loop?
2. What is the difference between `asyncio.gather` and `asyncio.wait`?
3. How do you run blocking code in an async application?
4. What is `asyncio.shield` and when would you use it?
5. How does asyncio handle file I/O (which is blocking on most systems)?

### Coding Challenges
- Write an async rate limiter using asyncio.Semaphore
- Build a simple async echo server using asyncio.start_server
- Create a task scheduler that runs tasks with different priorities

### Related Topics
- event loop, coroutines, async/await, async streams, asyncio subprocess

## Coroutines
### What It Is
A coroutine is a function declared with `async def` that can be suspended and resumed. Unlike regular functions, coroutines can yield control at `await` points, allowing other coroutines to run while waiting for I/O.

### Why It Is Important
Coroutines are the building blocks of async Python. They provide structured concurrency — each coroutine represents a sequential flow that cooperatively yields to others, without the complexity of threads or callbacks.

### How It Works Internally
Coroutines are implemented using Python's generators internally (before Python 3.5, coroutines were indeed generator-based with `@asyncio.coroutine` and `yield from`). An `async def` function returns a coroutine object when called. The coroutine object wraps the function's code and state. When awaited, it resumes execution until the next await, at which point it yields a `Future` or another coroutine to the event loop.

### Syntax
```python
# Coroutine definition
async def my_coro():
    return 42

# Coroutine object creation
coro_obj = my_coro()  # Does NOT execute — returns a coroutine object

# Execution
result = await coro_obj  # Executes and returns 42

# Or schedule via event loop
asyncio.run(my_coro())

# Coroutine introspection
coro_obj.cr_code      # code object
coro_obj.cr_frame     # current frame (if suspended)
coro_obj.cr_running   # is it running?
coro_obj.cr_await     # what it's awaiting (or None)
```

### Beginner Examples
```python
import asyncio

# Basic coroutine
async def hello():
    print("Hello")
    await asyncio.sleep(1)
    print("World")
    return "Done"

# Coroutines are NOT executed when called — they return coroutine objects
coro = hello()
print(f"Coroutine object: {coro}")
print(f"Type: {type(coro)}")

# Execution requires await or asyncio.run
result = asyncio.run(hello())
print(f"Result: {result}")
```

### Intermediate Examples
```python
import asyncio

# Coroutine chaining
async def step1():
    await asyncio.sleep(0.5)
    return "Step 1"

async def step2():
    await asyncio.sleep(0.3)
    return "Step 2"

async def coordinator():
    # Run steps concurrently
    results = await asyncio.gather(step1(), step2())
    return results

async def main():
    result = await coordinator()
    print(f"Coordinator result: {result}")

asyncio.run(main())

# Coroutine with error handling
async def safe_coro():
    try:
        await asyncio.sleep(1)
        raise ValueError("Oops")
    except ValueError as e:
        return f"Handled: {e}"

# Generator-based coroutine (legacy, pre-3.5)
@asyncio.coroutine
def legacy_coro():
    yield from asyncio.sleep(1)
    return "Legacy"

# Modern equivalent:
async def modern_coro():
    await asyncio.sleep(1)
    return "Modern"

# Running both
async def compare():
    l = await legacy_coro()
    m = await modern_coro()
    print(l, m)

asyncio.run(compare())
```

### Advanced Examples
```python
import asyncio

# Asynchronous generator (async for)
async def async_range(n):
    for i in range(n):
        await asyncio.sleep(0.1)
        yield i

async def use_async_gen():
    async for value in async_range(5):
        print(f"Got: {value}")

asyncio.run(use_async_gen())

# Coroutine local storage (like threading.local)
import contextvars

request_id = contextvars.ContextVar("request_id")

async def handler():
    request_id.set("req-123")
    await processor()

async def processor():
    rid = request_id.get()
    print(f"Processing request: {rid}")
    await asyncio.sleep(0.1)
    print(f"Still processing: {request_id.get()}")
    # Each concurrent invocation has its own context

async def main():
    await asyncio.gather(handler(), handler())

asyncio.run(main())

# Coroutine lifecycle
import inspect

async def lifecycle_demo():
    print("Coroutine started")
    await asyncio.sleep(0.5)
    print("Coroutine resumed")
    return 42

async def inspect_coroutine():
    coro = lifecycle_demo()
    
    print(f"Inspect results:")
    print(f"  type: {type(coro)}")
    print(f"  cr_running: {coro.cr_running}")
    print(f"  cr_await: {coro.cr_await}")
    print(f"  cr_code: {coro.cr_code}")
    print(f"  awaitable: {inspect.isawaitable(coro)}")
    
    result = await coro
    print(f"  executed, result: {result}")

asyncio.run(inspect_coroutine())
```

### Real-World Use Cases
- **Request handlers** — FastAPI path operations are coroutines
- **Data pipelines** — async generators for streaming data
- **Middleware** — coroutine-based middleware for request processing
- **Background tasks** — periodic cleanup, health checks
- **Protocol handlers** — implementing custom async protocols

### Common Mistakes
- Calling `async def` functions without `await` (creates coroutine object, doesn't execute)
- Trying to `await` a regular function (TypeError)
- Forgetting that coroutines must be awaited to execute
- Creating coroutine objects and not using them (garbage collector warns)
- Using `yield from` instead of `await` (Python 3.5+)

### Best Practices
- Always `await` coroutines (or create tasks from them)
- Use `async def` for I/O-bound functions
- Use `asyncio.create_task()` to run coroutines concurrently
- Use `gather()` to await multiple coroutines
- Use `contextvars` for request-scoped data in async contexts
- Prefer modern `async`/`await` over legacy `@asyncio.coroutine`

### Performance Considerations
- Creating a coroutine object is very cheap (no execution)
- Await overhead is minimal (tens of nanoseconds)
- Resuming a suspended coroutine is fast (function call overhead)
- Too many nested coroutines increases call stack depth for debugging

### Interview Questions
1. What is the difference between a coroutine and a regular function?
2. What happens when you call an async function without await?
3. How are coroutines different from generators?
4. What is `inspect.isawaitable()` used for?

### Coding Challenges
- Write a coroutine that takes another coroutine as input and adds logging
- Build a simple coroutine-based middleware chain
- Create a coroutine that times out and cancels if execution exceeds a limit

### Related Topics
- async/await, generators, asyncio, contextvars, yield from

## Event loop
### What It Is
The event loop is the core of asyncio — it continuously monitors I/O events and resumes suspended coroutines when their awaited operations complete. It manages the scheduling and execution of all coroutines, callbacks, and tasks.

### Why It Is Important
The event loop is the runtime that makes async programming work. Understanding the event loop is essential for advanced use cases: custom loop integration, signal handling, embedding async in sync code, and debugging async applications.

### How It Works Internally
The event loop runs in a single thread. Its cycle:
1. Run all ready callbacks and coroutine steps
2. Call `select()` (or `epoll`/`kqueue`/`IOCP`) to wait for I/O events with a timeout
3. Process triggered I/O events, resuming waiting coroutines
4. Repeat until stop is called

Implementation classes:
- `SelectorEventLoop` (Unix) — uses `selectors` module wrapping `epoll`/`kqueue`/`select`
- `ProactorEventLoop` (Windows) — uses I/O Completion Ports (IOCP). Windows default in Python 3.8+

### Syntax
```python
import asyncio

# Getting the event loop
loop = asyncio.get_running_loop()  # Inside async function (Python 3.7+)
loop = asyncio.get_event_loop()    # May create new loop (deprecated)
loop = asyncio.new_event_loop()    # Create new explicitly

# Loop operations
loop.run_until_complete(coro)
loop.run_forever()
loop.stop()
loop.close()

# Running in executor
await loop.run_in_executor(None, sync_func)

# Calling soon
loop.call_soon(callback, arg)
loop.call_later(delay, callback, arg)
loop.call_at(when, callback, arg)
```

### Beginner Examples
```python
import asyncio

# The modern way — asyncio.run() handles loop creation/destruction
async def main():
    loop = asyncio.get_running_loop()
    print(f"Event loop: {loop}")
    print(f"Default executor: {loop._default_executor}")

asyncio.run(main())

# Lower-level loop management (not typically needed)
async def low_level_example():
    loop = asyncio.get_running_loop()
    
    # Schedule a callback immediately
    loop.call_soon(lambda: print("Immediate callback"))
    
    # Schedule after 1 second
    loop.call_later(1, lambda: print("Delayed callback"))
    
    await asyncio.sleep(1.5)
    print("Async done")

asyncio.run(low_level_example())
```

### Intermediate Examples
```python
import asyncio
import time

# Custom event loop policies
class DebugEventLoopPolicy(asyncio.DefaultEventLoopPolicy):
    def get_event_loop(self):
        loop = super().get_event_loop()
        loop.set_debug(True)
        return loop

# Note: Must set policy before creating any loops
# asyncio.set_event_loop_policy(DebugEventLoopPolicy())

# Running multiple loops (rare, advanced)
async def task_in_loop(name):
    for i in range(3):
        await asyncio.sleep(0.5)
        print(f"Task {name}: tick {i}")

async def main_with_loop():
    loop = asyncio.get_running_loop()
    
    # Check loop properties
    print(f"Loop debug mode: {loop.is_running()}")
    print(f"Loop closed: {loop.is_closed()}")
    
    # Create tasks
    tasks = [asyncio.create_task(task_in_loop(i)) for i in range(3)]
    await asyncio.gather(*tasks)

asyncio.run(main_with_loop())

# Slow callback detection (debug mode)
async def debug_demo():
    loop = asyncio.get_running_loop()
    loop.slow_callback_duration = 0.1  # Log callbacks slower than 100ms
    
    def slow_callback():
        time.sleep(0.2)  # This will be logged
    
    loop.call_soon(slow_callback)
    await asyncio.sleep(0.5)

# asyncio.run(main_with_loop())
```

### Advanced Examples
```python
import asyncio
import signal
import sys

# Signal handling with the event loop
async def signal_handler_demo():
    loop = asyncio.get_running_loop()
    
    stop_event = asyncio.Event()
    
    def handle_signal():
        print("Signal received, stopping...")
        stop_event.set()
    
    # Register signal handlers
    if sys.platform != "win32":
        loop.add_signal_handler(signal.SIGINT, handle_signal)
        loop.add_signal_handler(signal.SIGTERM, handle_signal)
    
    await stop_event.wait()
    print("Clean shutdown complete")

# asyncio.run(signal_handler_demo())

# Custom event loop implementation sketch
class MinimalEventLoop:
    def __init__(self):
        self._ready = []
        self._stopping = False
    
    def call_soon(self, callback, *args):
        self._ready.append((callback, args))
    
    def stop(self):
        self._stopping = True
    
    def run_forever(self):
        while not self._stopping:
            while self._ready:
                cb, args = self._ready.pop(0)
                cb(*args)
            # In real implementation: select() for I/O
            if not self._ready:
                self._stopping = True  # Exit if no work
    
    def run_until_complete(self, coro):
        task = asyncio.ensure_future(coro, loop=self)
        task.add_done_callback(lambda t: self.stop())
        self.run_forever()
        return task.result()

# Server with asyncio.start_server
async def handle_client(reader, writer):
    data = await reader.read(100)
    message = data.decode()
    addr = writer.get_extra_info("peername")
    print(f"Received {message!r} from {addr}")
    
    writer.write(data)
    await writer.drain()
    writer.close()

async def run_server():
    server = await asyncio.start_server(handle_client, "127.0.0.1", 8888)
    async with server:
        await server.serve_forever()

# asyncio.run(run_server())
```

### Real-World Use Cases
- **Web frameworks** — FastAPI, Sanic, aiohttp run a server on an event loop
- **Custom protocols** — implementing TCP/UDP servers with event-driven I/O
- **Signal handling** — graceful shutdown on SIGINT/SIGTERM
- **Integration testing** — controlling loop time for deterministic tests
- **Embedded systems** — embedding async event loops into larger applications

### Common Mistakes
- Trying to create a new event loop inside an async context (use `get_running_loop()`)
- Calling `loop.close()` while tasks are still running
- Mixing `run_forever()` with `run_until_complete()` incorrectly
- Not setting a new event loop policy before the first `asyncio.run()` call
- Using `get_event_loop()` inside async functions (use `get_running_loop()`)

### Best Practices
- Use `asyncio.run()` in simple scripts — it manages the loop lifecycle
- Use `asyncio.get_running_loop()` inside async functions
- Use `loop.run_in_executor()` for blocking code
- Use `loop.add_signal_handler()` for signal handling (Unix)
- Set `loop.set_debug(True)` during development
- Enable `PYTHONASYNCIODEBUG=1` environment variable for full debugging
- Avoid mixing multiple event loops unless absolutely necessary

### Performance Considerations
- Event loop overhead per iteration is tiny (microseconds)
- `call_soon` is very fast (appends to a deque)
- `call_later` uses a heap for scheduling — O(log n) for scheduling
- `select()`/`epoll` overhead depends on number of monitored FDs
- The `ProactorEventLoop` on Windows has different performance characteristics (IOCP)

### Interview Questions
1. What is the event loop and what is its main responsibility?
2. What happens if you call `time.sleep()` instead of `asyncio.sleep()` in a coroutine?
3. How does the event loop handle I/O multiplexing?
4. What is the difference between `run_until_complete()` and `run_forever()`?
5. How do you properly shut down an event loop?

### Coding Challenges
- Implement a minimal event loop that can schedule and run coroutines
- Write a function that measures async overhead by timing event loop cycles
- Build a simple timer that fires callbacks at specified intervals using the event loop

### Related Topics
- asyncio, selector module, ProactorEventLoop, I/O multiplexing, cooperative multitasking
