# Multithreading - threading.Thread, Lock, Queue, concurrent.futures
## Introduction
Multithreading allows a program to run multiple threads of execution concurrently within the same process. Python's `threading` module provides a high-level interface for thread management, synchronization, and communication. While the GIL limits CPU-bound parallelism, threading excels at I/O-bound and concurrent operations.

## threading.Thread
### What It Is
`threading.Thread` represents a single thread of execution. It wraps a callable (function or method) and runs it in a separate OS thread. Each thread shares the same process memory, global interpreter lock, and file descriptors.

### Why It Is Important
Threads enable concurrent execution — handling multiple network requests, processing I/O streams, or maintaining responsiveness in GUI applications. They share memory space, making data exchange easier than with multiprocessing.

### How It Works Internally
When a `Thread` is started, the CPython interpreter calls the OS thread creation API (`CreateThread` on Windows, `pthread_create` on POSIX). The new thread acquires the GIL, executes the target callable, releases the GIL, and terminates. Thread objects are managed by the OS scheduler. Python threads are real OS threads (not green threads).

### Syntax
```python
import threading

# Function-based
t = threading.Thread(target=func, args=(arg1,))
t.start()
t.join(timeout=5)

# Class-based
class MyThread(threading.Thread):
    def run(self):
        pass

# Properties
t.name        # thread name
t.daemon      # daemon flag (exits with main thread)
t.ident       # thread ID
t.is_alive()  # check if running
```

### Beginner Examples
```python
import threading
import time

def worker(name, delay):
    for i in range(5):
        print(f"Worker {name}: iteration {i}")
        time.sleep(delay)

# Create and start threads
t1 = threading.Thread(target=worker, args=("A", 0.5))
t2 = threading.Thread(target=worker, args=("B", 0.3))

t1.start()
t2.start()

# Wait for completion
t1.join()
t2.join()
print("All threads done")
```

### Intermediate Examples
```python
import threading
import time
import random

class DownloadThread(threading.Thread):
    def __init__(self, url, output_dir):
        super().__init__()
        self.url = url
        self.output_dir = output_dir
        self.result = None
    
    def run(self):
        # Simulate download
        delay = random.uniform(0.5, 2.0)
        time.sleep(delay)
        self.result = f"Downloaded {self.url} in {delay:.2f}s"
        print(self.result)

# Download multiple files concurrently
urls = ["http://example.com/file1", "http://example.com/file2", "http://example.com/file3"]
threads = [DownloadThread(url, "./downloads") for url in urls]

for t in threads:
    t.start()

for t in threads:
    t.join()
    print(f"Result: {t.result}")

# Thread naming and identification
def show_thread_info():
    t = threading.current_thread()
    print(f"Thread: {t.name}, ID: {t.ident}, Alive: {t.is_alive()}")

t = threading.Thread(target=show_thread_info, name="InfoThread")
t.start()
t.join()

# Daemon threads (terminate when main thread exits)
daemon = threading.Thread(target=lambda: time.sleep(100), daemon=True)
daemon.start()
```

### Advanced Examples
```python
import threading
import time
import queue

# Thread pool implementation from scratch
class SimpleThreadPool:
    def __init__(self, num_workers):
        self.tasks = queue.Queue()
        self.workers = []
        for _ in range(num_workers):
            t = threading.Thread(target=self._worker_loop, daemon=True)
            t.start()
            self.workers.append(t)
    
    def _worker_loop(self):
        while True:
            func, args, kwargs, result_queue = self.tasks.get()
            try:
                result = func(*args, **kwargs)
                if result_queue:
                    result_queue.put(result)
            except Exception as e:
                if result_queue:
                    result_queue.put(e)
            finally:
                self.tasks.task_done()
    
    def submit(self, func, *args, **kwargs):
        result_queue = queue.Queue()
        self.tasks.put((func, args, kwargs, result_queue))
        return result_queue
    
    def wait_completion(self):
        self.tasks.join()

# Use custom pool
pool = SimpleThreadPool(4)
results = []
for i in range(10):
    q = pool.submit(lambda x: x * x, i)
    results.append(q)

pool.wait_completion()
for i, q in enumerate(results):
    print(f"Result {i}: {q.get()}")

# Thread-local data
thread_local = threading.local()

def set_and_print_value():
    thread_local.value = threading.current_thread().name
    time.sleep(0.1)
    print(f"{threading.current_thread().name}: {thread_local.value}")

threads = [threading.Thread(target=set_and_print_value, name=f"T-{i}") for i in range(5)]
for t in threads: t.start()
for t in threads: t.join()

# Graceful shutdown with event
stop_event = threading.Event()

def worker_with_stop():
    while not stop_event.is_set():
        print("Working...")
        stop_event.wait(0.5)

t = threading.Thread(target=worker_with_stop, daemon=True)
t.start()
time.sleep(2)
stop_event.set()
t.join()
```

### Real-World Use Cases
- **Web scraping** — downloading multiple pages concurrently
- **Network servers** — handling multiple client connections
- **GUI applications** — keeping UI responsive during background work
- **Database operations** — parallel queries to different shards
- **File processing** — reading/writing multiple files simultaneously

### Common Mistakes
- Not joining threads (main thread exits before workers complete)
- Modifying shared data without locks (race conditions)
- Creating too many threads (OS limit, resource exhaustion)
- Using threads for CPU-bound tasks (GIL limitation)
- Catching and ignoring exceptions in threads (errors are silent)

### Best Practices
- Use `concurrent.futures.ThreadPoolExecutor` for most thread pool needs
- Set `daemon=True` for background threads that should not block shutdown
- Always `join()` threads to ensure clean shutdown
- Use `threading.local()` for thread-specific data
- Use `threading.Event` for signaling between threads
- Avoid shared state; prefer message passing with `queue.Queue`
- Name threads meaningfully for debugging

### Performance Considerations
- Thread creation has overhead — use a pool for many tasks
- Context switching between threads has CPU cost
- GIL contention can degrade performance for CPU-bound code
- I/O-bound tasks benefit most from threading
- Too many threads increases memory usage (each thread has its own stack)

### Interview Questions
1. What is the difference between `start()` and `run()` in Thread?
2. What is a daemon thread and when is it useful?
3. How do you share data between threads?
4. What is the GIL and how does it affect threading?
5. How do you handle exceptions raised in a thread?

### Coding Challenges
- Implement a simple web crawler that fetches pages in parallel using threads
- Build a producer-consumer pattern using threading and Queue
- Create a thread pool from scratch with a fixed number of workers

### Related Topics
- Lock, Queue, concurrent.futures, multiprocessing, async IO, GIL

## Lock
### What It Is
`threading.Lock` is a synchronization primitive that ensures only one thread can access a critical section of code at a time. It prevents race conditions when multiple threads read/write shared data.

### Why It Is Important
Without locks, concurrent access to shared variables can corrupt data, cause crashes, or produce incorrect results. Locks provide mutual exclusion — the simplest and most essential synchronization mechanism.

### How It Works Internally
A Lock has two states: locked and unlocked. When a thread calls `.acquire()`:
1. If unlocked, the lock becomes locked and the thread proceeds
2. If locked, the thread blocks until the lock is released
The implementation uses a semaphore (via `threading._allocate_lock()`) backed by the OS mutex (futex on Linux, CRITICAL_SECTION on Windows, or `pthread_mutex_t` on POSIX).

### Syntax
```python
import threading

lock = threading.Lock()

# Blocking acquire
lock.acquire()
try:
    # critical section
finally:
    lock.release()

# Context manager (preferred)
with lock:
    # critical section

# Non-blocking acquire
if lock.acquire(blocking=False):
    try:
        pass
    finally:
        lock.release()
```

### Beginner Examples
```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100000):
        with lock:
            counter += 1

threads = [threading.Thread(target=increment) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()

print(f"Counter: {counter}")  # 1000000 (correct with lock)

# Without lock (race condition demonstration)
counter2 = 0

def increment_unprotected():
    global counter2
    for _ in range(100000):
        counter2 += 1  # Race condition!

threads = [threading.Thread(target=increment_unprotected) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()

print(f"Unprotected counter: {counter2}")  # Likely less than 1000000
```

### Intermediate Examples
```python
import threading
import time

# Bank account example
class BankAccount:
    def __init__(self, balance=0):
        self.balance = balance
        self._lock = threading.Lock()
    
    def deposit(self, amount):
        with self._lock:
            new_balance = self.balance + amount
            time.sleep(0.001)  # Simulate processing
            self.balance = new_balance
            return self.balance
    
    def withdraw(self, amount):
        with self._lock:
            if self.balance >= amount:
                new_balance = self.balance - amount
                time.sleep(0.001)
                self.balance = new_balance
                return self.balance
            return False
    
    def get_balance(self):
        with self._lock:
            return self.balance

account = BankAccount(1000)

def do_transactions():
    for _ in range(100):
        account.deposit(100)
        account.withdraw(50)

threads = [threading.Thread(target=do_transactions) for _ in range(5)]
for t in threads: t.start()
for t in threads: t.join()

print(f"Final balance: {account.get_balance()}")

# Non-blocking acquire (try-lock)
lock = threading.Lock()
lock.acquire()  # Main thread acquires

def try_acquire():
    if lock.acquire(blocking=False):
        try:
            print("Secondary thread got the lock")
        finally:
            lock.release()
    else:
        print("Lock not available, doing other work")

t = threading.Thread(target=try_acquire)
t.start()
t.join()

lock.release()  # Main thread releases
```

### Advanced Examples
```python
import threading
import time

# Fine-grained locking (multiple locks, lock ordering)
class Account:
    def __init__(self, name, balance):
        self.name = name
        self.balance = balance
        self.lock = threading.Lock()
    
    def transfer(self, target, amount):
        # Lock ordering to prevent deadlock
        first, second = (self, target) if id(self) < id(target) else (target, self)
        
        with first.lock:
            with second.lock:
                if self.balance >= amount:
                    self.balance -= amount
                    target.balance += amount
                    return True
                return False

a1 = Account("A", 1000)
a2 = Account("B", 1000)

def transfer_money():
    for _ in range(100):
        a1.transfer(a2, 10)
        a2.transfer(a1, 10)

threads = [threading.Thread(target=transfer_money) for _ in range(5)]
for t in threads: t.start()
for t in threads: t.join()
print(f"A: {a1.balance}, B: {a2.balance}")

# Reentrant lock (RLock) — same thread can acquire multiple times
rlock = threading.RLock()

def recursive_function(n):
    with rlock:
        if n > 0:
            recursive_function(n - 1)  # Same thread re-acquires

recursive_function(5)
print("Reentrant lock works")

# Condition variable with Lock
condition = threading.Condition()
items = []

def producer():
    with condition:
        items.append("item")
        condition.notify()

def consumer():
    with condition:
        while not items:
            condition.wait()
        item = items.pop()
        return item
```

### Real-World Use Cases
- **Counter/accumulator protection** — ensuring atomic increments
- **Banking transactions** — preventing race conditions on balances
- **Cache updates** — protecting shared cache dictionaries
- **Resource pools** — coordinating access to limited resources
- **File rotation** — ensuring log file rotation is thread-safe

### Common Mistakes
- Forgetting to release the lock after acquire (use `with` statement!)
- Acquiring locks in inconsistent order (causes deadlocks)
- Holding locks during I/O or long operations (reduces concurrency)
- Using Lock when RLock is needed (re-entrant calls)
- Locking at too coarse a granularity (serializes all threads)

### Best Practices
- Always use `with lock:` context manager, never raw `acquire()`/`release()`
- Keep critical sections as short as possible
- Establish a consistent lock ordering to prevent deadlocks
- Use `RLock` if the same thread needs to re-acquire the same lock
- Prefer higher-level concurrency primitives when possible
- Avoid holding locks during I/O, network calls, or sleeping

### Performance Considerations
- Lock contention reduces parallelism (threads waiting for lock)
- Fine-grained locking improves concurrency but increases complexity
- Lock acquisition/release has overhead (but is very fast in practice)
- Reading shared data also needs locking (or use atomic operations)
- Over-locking can make threaded code slower than single-threaded

### Interview Questions
1. What is a race condition and how does Lock prevent it?
2. What is the difference between Lock and RLock?
3. What is a deadlock and how do you prevent it?
4. What does `acquire(blocking=False)` do?

### Coding Challenges
- Implement a thread-safe counter using Lock
- Build a thread-safe queue from scratch using Lock and Condition
- Create a bank transfer system that prevents deadlocks

### Related Topics
- RLock, Semaphore, Condition, Event, Queue, deadlock

## Queue
### What It Is
`queue.Queue` is a thread-safe FIFO (first-in-first-out) queue for exchanging data between threads. It provides blocking `put()` and `get()` operations, optional size limits, and built-in synchronization.

### Why It Is Important
Queue is the foundation of the producer-consumer pattern — the most common multi-threaded programming model. It eliminates the need for manual Lock management when passing data between threads.

### How It Works Internally
Queue uses `threading.Lock` for mutex protection and `threading.Condition` (with `notify`/`wait`) for blocking operations. Internally it wraps a `collections.deque` for the underlying storage. When the queue is full, `put()` blocks on a condition until space is available. When empty, `get()` blocks until an item is available.

### Syntax
```python
from queue import Queue, LifoQueue, PriorityQueue

q = Queue(maxsize=10)
q.put(item)           # blocks if full
q.put(item, block=False)  # raises Full
q.put(item, timeout=5)    # blocks up to 5 seconds
item = q.get()        # blocks if empty
item = q.get_nowait() # raises Empty
q.task_done()         # signal task completion
q.join()              # wait until all tasks processed
q.qsize()             # approximate size (not reliable)
q.empty()             # is empty?
q.full()              # is full?
```

### Beginner Examples
```python
import threading
import queue
import time

# Simple producer-consumer
q = queue.Queue(maxsize=5)

def producer():
    for i in range(10):
        item = f"item-{i}"
        q.put(item)
        print(f"Produced: {item}")
        time.sleep(0.2)

def consumer():
    while True:
        item = q.get()
        if item is None:  # Poison pill
            q.task_done()
            break
        print(f"Consumed: {item}")
        time.sleep(0.5)
        q.task_done()

prod = threading.Thread(target=producer)
cons = threading.Thread(target=consumer)
prod.start()
cons.start()

prod.join()
q.put(None)  # Signal consumer to stop
cons.join()
```

### Intermediate Examples
```python
import threading
import queue
import random
import time

# Multiple producers and consumers
q = queue.Queue(maxsize=20)
stop_event = threading.Event()

def producer(name):
    while not stop_event.is_set():
        item = f"{name}-{random.randint(1, 100)}"
        try:
            q.put(item, timeout=1)
            print(f"[Producer {name}] put {item}")
        except queue.Full:
            print(f"[Producer {name}] queue full")
        time.sleep(random.uniform(0.1, 0.3))

def consumer(name):
    while not stop_event.is_set():
        try:
            item = q.get(timeout=1)
            print(f"[Consumer {name}] got {item}")
            time.sleep(random.uniform(0.2, 0.5))
            q.task_done()
        except queue.Empty:
            pass

producers = [threading.Thread(target=producer, args=(f"P{i}",), daemon=True) for i in range(3)]
consumers = [threading.Thread(target=consumer, args=(f"C{i}",), daemon=True) for i in range(5)]

for t in producers + consumers: t.start()
time.sleep(5)
stop_event.set()
time.sleep(1)
print("Stopped")

# Priority queue
pq = queue.PriorityQueue()
pq.put((2, "medium priority"))
pq.put((1, "high priority"))
pq.put((3, "low priority"))

while not pq.empty():
    priority, task = pq.get()
    print(f"Processing: {task} (priority {priority})")
```

### Advanced Examples
```python
import threading
import queue
import time
from dataclasses import dataclass, field
from enum import Enum

class TaskStatus(Enum):
    PENDING = "pending"
    PROCESSING = "processing"
    DONE = "done"

@dataclass(order=True)
class Task:
    priority: int
    created_at: float = field(compare=False)
    data: dict = field(compare=False)
    status: TaskStatus = field(compare=False, default=TaskStatus.PENDING)

# Worker pool with queue
class WorkerPool:
    def __init__(self, num_workers, task_queue):
        self.task_queue = task_queue
        self.workers = []
        self.results = queue.Queue()
        self.stop_event = threading.Event()
        
        for i in range(num_workers):
            t = threading.Thread(target=self._worker_loop, args=(i,), daemon=True)
            t.start()
            self.workers.append(t)
    
    def _worker_loop(self, worker_id):
        while not self.stop_event.is_set():
            try:
                task = self.task_queue.get(timeout=0.5)
                task.status = TaskStatus.PROCESSING
                # Simulate work
                result = task.data.get("value", 0) * 2
                time.sleep(0.1)
                task.status = TaskStatus.DONE
                self.results.put({
                    "worker": worker_id,
                    "task": task,
                    "result": result,
                })
                self.task_queue.task_done()
            except queue.Empty:
                continue
    
    def shutdown(self):
        self.stop_event.set()
        for t in self.workers:
            t.join(timeout=1)

# Usage
task_queue = queue.PriorityQueue()
pool = WorkerPool(4, task_queue)

for i in range(10):
    task = Task(priority=i, created_at=time.time(), data={"value": i})
    task_queue.put(task)

task_queue.join()
pool.shutdown()

while not pool.results.empty():
    r = pool.results.get_nowait()
    print(f"Worker {r['worker']}: {r['result']}")
```

### Real-World Use Cases
- **Web scraping pipelines** — crawl URLs, parse HTML, save results
- **Log processing** — read log lines, parse, write to database
- **Image processing** — load images, transform, save thumbnails
- **Task schedulers** — distribute work across worker threads
- **Data streaming** — buffer between data source and sink

### Common Mistakes
- Not calling `task_done()` after processing (prevents `join()` from working)
- Using `qsize()` for synchronization decisions (not precise)
- Blocking forever on `get()` with no poison pill mechanism
- Putting mutable objects without copying (shared state issues)
- Not handling `queue.Empty` and `queue.Full` exceptions

### Best Practices
- Use poison pills (sentinel values) to signal workers to stop
- Use `join()` with `task_done()` for reliable completion tracking
- Use `timeout` parameter to handle shutdown gracefully
- Prefer `get_nowait()` or `get(timeout=...)` over bare `get()`
- Use `PriorityQueue` for ordered task processing
- Use `LifoQueue` for LIFO processing patterns (stack)
- Put immutable data or copies in the queue

### Performance Considerations
- Queue operations have lock overhead but are highly optimized
- `maxsize` prevents unbounded memory growth
- `task_done()`/`join()` pair adds synchronization overhead
- Using a single queue as bottleneck between producers and consumers is efficient
- Consider `queue.SimpleQueue` (Python 3.7+) for lightweight single-producer scenarios

### Interview Questions
1. How does Queue achieve thread safety?
2. What is the producer-consumer pattern and how does Queue implement it?
3. What is a poison pill and how is it used?
4. What is the difference between Queue, LifoQueue, and PriorityQueue?
5. How do `task_done()` and `join()` work together?

### Coding Challenges
- Implement a thread-safe priority-based job scheduler using Queue
- Build a web crawler that uses Queue to manage URLs to visit
- Create a data processing pipeline with multiple stages connected by Queues

### Related Topics
- threading, multiprocessing.Queue, asyncio.Queue, concurrent.futures

## concurrent.futures
### What It Is
`concurrent.futures` provides a high-level interface for asynchronously executing callables using threads (`ThreadPoolExecutor`) or processes (`ProcessPoolExecutor`). It abstracts thread/process creation, task submission, and result collection behind a consistent API.

### Why It Is Important
`concurrent.futures` simplifies concurrent programming by providing a clean, Pythonic API. It replaces manual thread/process management with executors, futures, and result callbacks, reducing boilerplate and error-prone code.

### How It Works Internally
`ThreadPoolExecutor` maintains a fixed-size pool of worker threads. When `submit()` is called, the callable is placed in an internal Queue. Idle workers pick up tasks from the queue, execute them, and store results. A `Future` object is returned immediately — it acts as a proxy for the result, which is populated when the worker completes. The `Future` uses threading.Event to allow blocking `result()` calls.

### Syntax
```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

# Submit single tasks
with ThreadPoolExecutor(max_workers=4) as ex:
    future = ex.submit(func, arg1, arg2)
    result = future.result(timeout=5)

# Map
results = ex.map(func, iterable)

# As completion
from concurrent.futures import as_completed, wait
for future in as_completed(futures):
    result = future.result()
```

### Beginner Examples
```python
from concurrent.futures import ThreadPoolExecutor
import time

def square(n):
    time.sleep(0.5)
    return n * n

numbers = [1, 2, 3, 4, 5]

# Using submit
with ThreadPoolExecutor(max_workers=3) as executor:
    futures = {executor.submit(square, n): n for n in numbers}
    for future in futures:
        result = future.result()
        print(f"Result: {result}")

# Using map (simpler)
with ThreadPoolExecutor(max_workers=3) as executor:
    results = list(executor.map(square, numbers))
    print(f"Map results: {results}")
```

### Intermediate Examples
```python
from concurrent.futures import ThreadPoolExecutor, as_completed, wait, ALL_COMPLETED
import time
import random

def fetch_url(url):
    delay = random.uniform(0.5, 2.0)
    time.sleep(delay)
    return f"Data from {url} (took {delay:.2f}s)"

urls = [f"http://example.com/page/{i}" for i in range(10)]

with ThreadPoolExecutor(max_workers=5) as executor:
    # Submit all tasks
    fut_to_url = {executor.submit(fetch_url, url): url for url in urls}
    
    # Process results as they complete
    for future in as_completed(fut_to_url):
        url = fut_to_url[future]
        try:
            data = future.result()
            print(f"Completed: {url} -> {data}")
        except Exception as e:
            print(f"Failed: {url} -> {e}")
    
    # Wait with timeout
    all_futures = list(fut_to_url.keys())
    done, not_done = wait(all_futures, timeout=1.0, return_when=ALL_COMPLETED)
    print(f"Done: {len(done)}, Still running: {len(not_done)}")

# Exception handling
def might_fail(x):
    if x == 5:
        raise ValueError("Bad value")
    return x * 2

with ThreadPoolExecutor(max_workers=2) as ex:
    futures = [ex.submit(might_fail, i) for i in range(10)]
    for f in as_completed(futures):
        try:
            print(f.result())
        except ValueError as e:
            print(f"Error: {e}")
```

### Advanced Examples
```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor
import time
import math

# CPU-bound: compare ThreadPool vs ProcessPool
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(math.sqrt(n)) + 1):
        if n % i == 0:
            return False
    return True

numbers = [112272535095293, 112582705942171, 115280095190773, 115797848077099,
           1099726899285419] * 2  # Big primes to test

# ThreadPool (slower due to GIL)
start = time.time()
with ThreadPoolExecutor(max_workers=4) as ex:
    list(ex.map(is_prime, numbers))
thread_time = time.time() - start
print(f"ThreadPool time: {thread_time:.3f}s")

# ProcessPool (faster for CPU-bound)
start = time.time()
with ProcessPoolExecutor(max_workers=4) as ex:
    list(ex.map(is_prime, numbers))
process_time = time.time() - start
print(f"ProcessPool time: {process_time:.3f}s")

# Callback style via add_done_callback
def on_complete(future):
    print(f"Future done: {future.result()}")

with ThreadPoolExecutor(max_workers=2) as ex:
    future = ex.submit(lambda: "Hello from callback")
    future.add_done_callback(on_complete)
    time.sleep(0.1)

# Chaining futures
def step1(x):
    return x + 1

def step2(x):
    return x * 2

def step3(x):
    return x ** 2

with ThreadPoolExecutor(max_workers=3) as ex:
    f1 = ex.submit(step1, 5)
    f2 = ex.submit(step2, f1.result())
    f3 = ex.submit(step3, f2.result())
    print(f"Chained result: {f3.result()}")  # ((5+1)*2)^2 = 144

# Cancelling futures
def long_running():
    time.sleep(10)
    return "done"

with ThreadPoolExecutor(max_workers=1) as ex:
    future = ex.submit(long_running)
    time.sleep(0.1)
    cancelled = future.cancel()
    print(f"Cancelled: {cancelled}")
```

### Real-World Use Cases
- **Web API requests** — fetching multiple endpoints simultaneously
- **Database batch operations** — parallel queries across shards
- **File processing** — compressing/transforming multiple files
- **Microservice orchestration** — calling multiple services concurrently
- **Test runners** — running tests in parallel
- **Image/video processing** — encoding multiple files simultaneously

### Common Mistakes
- Using `ThreadPoolExecutor` for CPU-bound tasks when `ProcessPoolExecutor` is needed
- Not setting `max_workers` appropriately (defaults to CPU count * 5)
- Forgetting to call `.result()` on futures (silently ignores errors)
- Mixing thread and process executors incorrectly
- Using mutable shared state without synchronization in thread pools

### Best Practices
- Use `ThreadPoolExecutor` for I/O-bound, `ProcessPoolExecutor` for CPU-bound
- Use the context manager (`with` statement) for automatic cleanup
- Use `as_completed()` for processing results as they arrive
- Use `map()` for simple batch operations with homogeneous results
- Handle exceptions via `future.result()` in try/except
- Set `max_workers` based on workload: I/O tasks = higher count, CPU tasks = CPU count
- Use `wait()` with `FIRST_COMPLETED` for early-exit patterns

### Performance Considerations
- `ThreadPoolExecutor` is lightweight (threads share memory)
- `ProcessPoolExecutor` has higher overhead (pickling, IPC, process creation)
- `max_workers` beyond a point causes diminishing returns (context switching)
- `map()` is slightly more efficient than individual `submit()` calls
- Passing large data to subprocesses is expensive (must be pickled)
- `as_completed()` adds minimal overhead over waiting for all futures

### Interview Questions
1. What is the difference between ThreadPoolExecutor and ProcessPoolExecutor?
2. What is a Future and how does it work?
3. How do `submit()`, `map()`, and `as_completed()` differ?
4. When would you use `add_done_callback` instead of `result()`?
5. How do you handle exceptions from tasks submitted to an executor?

### Coding Challenges
- Write a parallel web scraper using ThreadPoolExecutor that respects rate limits
- Implement a parallel prime number finder using ProcessPoolExecutor
- Build a simple task orchestration system that chains dependent futures
- Create a benchmark comparing ThreadPool vs ProcessPool for different workloads

### Related Topics
- threading, multiprocessing, asyncio, futures (standard library)
