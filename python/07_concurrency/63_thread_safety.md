# Thread Safety - race conditions, Lock, RLock, deadlock prevention

## Introduction

Thread safety refers to code that functions correctly when multiple threads access shared data concurrently. In Python, thread safety is achieved through synchronization primitives like locks, reentrant locks, semaphores, and thread-local storage to prevent race conditions and ensure data consistency.

## Why It Is Important

Without thread safety, concurrent access to shared data can lead to race conditions, corrupted state, deadlocks, and subtle bugs that are difficult to reproduce and debug. Understanding thread safety is crucial for writing reliable multi-threaded applications, especially in server environments, GUI applications, and data processing pipelines.

## Syntax

```python
import threading
from threading import Lock, RLock, Semaphore, local
from concurrent.futures import ThreadPoolExecutor

# Lock
lock = Lock()
lock.acquire()
try:
    # critical section
finally:
    lock.release()

# with statement (preferred)
with lock:
    # critical section

# RLock for reentrant locking
rlock = RLock()

# Thread-local storage
local_data = threading.local()
local_data.value = 42

# concurrent.futures
with ThreadPoolExecutor(max_workers=4) as executor:
    future = executor.submit(func, arg)
    result = future.result()
```

## Examples

### Race Condition Demonstration

```python
import threading
import time

counter = 0

def increment_without_lock(amount):
    global counter
    for _ in range(amount):
        temp = counter
        time.sleep(0.000001)
        counter = temp + 1

threads = [
    threading.Thread(target=increment_without_lock, args=(10000,))
    for _ in range(5)
]

for t in threads:
    t.start()
for t in threads:
    t.join()

print(f"Without lock: counter = {counter} (expected: {50000})")

counter = 0
lock = threading.Lock()

def increment_with_lock(amount):
    global counter
    for _ in range(amount):
        with lock:
            counter += 1

threads = [
    threading.Thread(target=increment_with_lock, args=(10000,))
    for _ in range(5)
]

for t in threads:
    t.start()
for t in threads:
    t.join()

print(f"With lock: counter = {counter} (expected: {50000})")
```

### Lock vs RLock

```python
import threading
import time

lock = threading.Lock()

def recursive_function_with_lock(n):
    with lock:
        print(f"Level {n} with Lock")
        if n > 0:
            recursive_function_with_lock(n - 1)

# This will deadlock
try:
    t = threading.Thread(target=recursive_function_with_lock, args=(3,))
    t.start()
    t.join(timeout=2)
    if t.is_alive():
        print("DEADLOCK with Lock!")
except RuntimeError as e:
    print(f"Error: {e}")

rlock = threading.RLock()

def recursive_function_with_rlock(n):
    with rlock:
        print(f"Level {n} with RLock")
        if n > 0:
            recursive_function_with_rlock(n - 1)

t = threading.Thread(target=recursive_function_with_rlock, args=(3,))
t.start()
t.join()
print("RLock works fine with recursive calls")

# RLock ownership tracking
print(f"\nRLock owned by thread: {rlock._is_owned()}")
```

### Deadlock Example and Prevention

```python
import threading
import time

def deadlock_example():
    lock_a = threading.Lock()
    lock_b = threading.Lock()

    def worker_a():
        with lock_a:
            print("Worker A: acquired lock A")
            time.sleep(0.01)
            print("Worker A: waiting for lock B...")
            with lock_b:  # Deadlock if B holds B
                print("Worker A: acquired lock B")

    def worker_b():
        with lock_b:
            print("Worker B: acquired lock B")
            time.sleep(0.01)
            print("Worker B: waiting for lock A...")
            with lock_a:  # Deadlock if A holds A
                print("Worker B: acquired lock A")

    t1 = threading.Thread(target=worker_a)
    t2 = threading.Thread(target=worker_b)

    t1.start()
    t2.start()

    t1.join(timeout=2)
    t2.join(timeout=2)

    if t1.is_alive() and t2.is_alive():
        print("DEADLOCK detected!")

def deadlock_prevention():
    lock_a = threading.Lock()
    lock_b = threading.Lock()

    def worker(lock_first, lock_second, name):
        with lock_first:
            print(f"{name}: acquired first lock")
            time.sleep(0.01)
            with lock_second:
                print(f"{name}: acquired second lock")

    t1 = threading.Thread(target=worker, args=(lock_a, lock_b, "Worker-1"))
    t2 = threading.Thread(target=worker, args=(lock_a, lock_b, "Worker-2"))

    t1.start()
    t2.start()
    t1.join()
    t2.join()
    print("No deadlock with consistent lock ordering")

deadlock_example()
time.sleep(0.5)
print("\n--- Prevention ---")
deadlock_prevention()
```

### Livelock Example

```python
import threading
import time

def livelock_example():
    lock_a = threading.Lock()
    lock_b = threading.Lock()

    def worker_a():
        while True:
            if lock_a.acquire(blocking=False):
                time.sleep(0.01)
                if lock_b.acquire(blocking=False):
                    print("Worker A: acquired both locks")
                    lock_b.release()
                    lock_a.release()
                    break
                else:
                    print("Worker A: backing off...")
                    lock_a.release()
                    time.sleep(0.01)

    def worker_b():
        while True:
            if lock_b.acquire(blocking=False):
                time.sleep(0.01)
                if lock_a.acquire(blocking=False):
                    print("Worker B: acquired both locks")
                    lock_a.release()
                    lock_b.release()
                    break
                else:
                    print("Worker B: backing off...")
                    lock_b.release()
                    time.sleep(0.01)

    t1 = threading.Thread(target=worker_a, daemon=True)
    t2 = threading.Thread(target=worker_b, daemon=True)
    t1.start()
    t2.start()

    time.sleep(2)
    print("Livelock: threads are still running, making no progress")

livelock_example()
```

### threading.local() — Thread-Local Storage

```python
import threading
import time
import random

thread_data = threading.local()

class RequestContext:
    def __init__(self, user_id, request_id):
        self.user_id = user_id
        self.request_id = request_id

def process_request(user_id):
    thread_data.context = RequestContext(user_id, random.randint(1000, 9999))
    thread_data.user_id = user_id

    time.sleep(random.uniform(0.1, 0.3))
    ctx = thread_data.context
    print(f"Thread {threading.current_thread().name}: "
          f"processing request {ctx.request_id} for user {ctx.user_id}")

def main():
    threads = [
        threading.Thread(target=process_request, args=(i,), name=f"Worker-{i}")
        for i in range(5)
    ]

    for t in threads:
        t.start()

    try:
        ctx = thread_data.context
    except AttributeError:
        print("Main thread: no thread-local context (expected)")

    for t in threads:
        t.join()

    print("Thread-local storage ensures data isolation")

main()
```

### Atomic Operations

```python
import threading
import time
import dis

def show_non_atomic():
    counter = 0

    def increment():
        nonlocal counter
        counter += 1

    print("Bytecode for 'counter += 1':")
    for instr in dis.get_instructions(increment):
        if 'counter' in str(instr):
            print(f"  {instr.opname}")

    print("\nThe operation requires multiple bytecode instructions")
    print("This is why increment is NOT atomic without a lock")

show_non_atomic()

def demonstrate_race_condition():
    counter = 0
    iterations = 50000

    def increment():
        nonlocal counter
        for _ in range(iterations):
            counter += 1

    t1 = threading.Thread(target=increment)
    t2 = threading.Thread(target=increment)

    t1.start()
    t2.start()
    t1.join()
    t2.join()

    expected = iterations * 2
    print(f"\nNon-atomic increment: counter = {counter}, expected = {expected}")
    print(f"Difference: {expected - counter} (race condition)")

demonstrate_race_condition()
```

### concurrent.futures.ThreadPoolExecutor

```python
import concurrent.futures
import threading
import time
import random

shared_results = []
results_lock = threading.Lock()

def worker_task(task_id):
    time.sleep(random.uniform(0.1, 0.3))
    result = task_id * task_id

    with results_lock:
        shared_results.append(result)

    return result

with concurrent.futures.ThreadPoolExecutor(max_workers=4) as executor:
    futures = [executor.submit(worker_task, i) for i in range(10)]

    for future in concurrent.futures.as_completed(futures):
        try:
            result = future.result(timeout=5)
            print(f"Completed: {result}")
        except Exception as e:
            print(f"Failed: {e}")

print(f"\nShared results: {sorted(shared_results)}")

with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
    numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    results = list(executor.map(lambda x: x * 2, numbers))

print(f"Map results: {results}")

def future_callbacks():
    executor = concurrent.futures.ThreadPoolExecutor(max_workers=2)

    def on_complete(future):
        try:
            result = future.result()
            print(f"Callback: result = {result}")
        except Exception as e:
            print(f"Callback: error = {e}")

    future = executor.submit(threading.current_thread, None)
    future.add_done_callback(on_complete)
    time.sleep(0.1)
    executor.shutdown()

future_callbacks()
```

## Beginner Examples

```python
# Safe counter class
import threading

class ThreadSafeCounter:
    def __init__(self):
        self._value = 0
        self._lock = threading.Lock()

    def increment(self):
        with self._lock:
            self._value += 1

    def decrement(self):
        with self._lock:
            self._value -= 1

    def get_value(self):
        with self._lock:
            return self._value

counter = ThreadSafeCounter()

def worker(increment_count):
    for _ in range(increment_count):
        counter.increment()

threads = [threading.Thread(target=worker, args=(10000,)) for _ in range(5)]
for t in threads: t.start()
for t in threads: t.join()

print(f"Thread-safe counter: {counter.get_value()} (expected: {50000})")

# Safe singleton pattern
class ThreadSafeSingleton:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance.initialized = False
        return cls._instance

    def initialize(self):
        if not self.initialized:
            with self._lock:
                if not self.initialized:
                    self.initialized = True
                    self.value = 42

def test_singleton():
    s1 = ThreadSafeSingleton()
    s2 = ThreadSafeSingleton()
    print(f"Singleton test: s1 is s2 = {s1 is s2}")

t1 = threading.Thread(target=test_singleton)
t2 = threading.Thread(target=test_singleton)
t1.start(); t2.start()
t1.join(); t2.join()
```

## Intermediate Examples

```python
# Thread-safe queue implementation
import threading
import time
from collections import deque

class ThreadSafeBoundedQueue:
    def __init__(self, maxsize=0):
        self._queue = deque()
        self._maxsize = maxsize
        self._mutex = threading.Lock()
        self._not_empty = threading.Condition(self._mutex)
        self._not_full = threading.Condition(self._mutex)

    def put(self, item, timeout=None):
        with self._not_full:
            if self._maxsize > 0:
                deadline = time.monotonic() + timeout if timeout else None
                while len(self._queue) >= self._maxsize:
                    remaining = deadline - time.monotonic() if deadline else None
                    if remaining is not None and remaining <= 0:
                        raise TimeoutError("Queue full")
                    self._not_full.wait(timeout=remaining)
            self._queue.append(item)
            self._not_empty.notify()

    def get(self, timeout=None):
        with self._not_empty:
            deadline = time.monotonic() + timeout if timeout else None
            while not self._queue:
                remaining = deadline - time.monotonic() if deadline else None
                if remaining is not None and remaining <= 0:
                    raise TimeoutError("Queue empty")
                self._not_empty.wait(timeout=remaining)
            item = self._queue.popleft()
            self._not_full.notify()
            return item

    def qsize(self):
        with self._mutex:
            return len(self._queue)

bq = ThreadSafeBoundedQueue(maxsize=3)

def producer():
    for i in range(6):
        bq.put(f"item-{i}")
        print(f"Produced: item-{i}")
        time.sleep(0.1)

def consumer():
    for _ in range(6):
        item = bq.get()
        print(f"Consumed: {item}")
        time.sleep(0.3)

t1 = threading.Thread(target=producer)
t2 = threading.Thread(target=consumer)
t1.start(); t2.start()
t1.join(); t2.join()
print("Bounded queue example completed")

# Thread-safe read-write lock
class ReadWriteLock:
    def __init__(self):
        self._read_ready = threading.Condition(threading.Lock())
        self._readers = 0

    def acquire_read(self):
        with self._read_ready:
            self._readers += 1

    def release_read(self):
        with self._read_ready:
            self._readers -= 1
            if self._readers == 0:
                self._read_ready.notify_all()

    def acquire_write(self):
        self._read_ready.acquire()
        while self._readers > 0:
            self._read_ready.wait()

    def release_write(self):
        self._read_ready.release()

    def reader_lock(self):
        return _ReadLock(self)

    def writer_lock(self):
        return _WriteLock(self)

class _ReadLock:
    def __init__(self, rwlock): self.rwlock = rwlock
    def __enter__(self): self.rwlock.acquire_read()
    def __exit__(self, *args): self.rwlock.release_read()

class _WriteLock:
    def __init__(self, rwlock): self.rwlock = rwlock
    def __enter__(self): self.rwlock.acquire_write()
    def __exit__(self, *args): self.rwlock.release_write()

rwlock = ReadWriteLock()
shared_resource = {}
read_count = 0
write_count = 0

def reader(id):
    global read_count
    for _ in range(3):
        with rwlock.reader_lock():
            read_count += 1
            value = shared_resource.get("key", "N/A")
            print(f"Reader {id}: read '{value}' (reads: {read_count})")
        time.sleep(0.1)

def writer(id):
    global write_count
    for i in range(2):
        with rwlock.writer_lock():
            write_count += 1
            shared_resource["key"] = f"value-{id}-{i}"
            print(f"Writer {id}: wrote 'value-{id}-{i}' (writes: {write_count})")
        time.sleep(0.3)

threads = []
for i in range(3): threads.append(threading.Thread(target=reader, args=(i,)))
for i in range(2): threads.append(threading.Thread(target=writer, args=(i,)))
for t in threads: t.start()
for t in threads: t.join()
print(f"Read-write lock: {read_count} reads, {write_count} writes")
```

## Advanced Examples

```python
# Lock-free data structure (Treiber stack) demonstration
import threading
import time
import random

class LockFreeStack:
    def __init__(self):
        self._top = None
        self._lock = threading.Lock()

    def push(self, value):
        with self._lock:
            self._top = (value, self._top)

    def pop(self):
        with self._lock:
            if self._top is None:
                return None
            value, self._top = self._top
            return value

    def is_empty(self):
        with self._lock:
            return self._top is None

lfs = LockFreeStack()

def worker_push(stack, count):
    for i in range(count):
        stack.push(f"item-{threading.current_thread().name}-{i}")
        time.sleep(random.uniform(0.001, 0.01))

def worker_pop(stack, count):
    popped = 0
    while popped < count:
        item = stack.pop()
        if item is not None:
            popped += 1

threads = []
for i in range(4):
    t = threading.Thread(target=worker_push, args=(lfs, 100), name=f"P-{i}")
    threads.append(t)

for i in range(2):
    t = threading.Thread(target=worker_pop, args=(lfs, 200), name=f"C-{i}")
    threads.append(t)

for t in threads:
    t.start()
for t in threads:
    t.join()

print(f"Stack empty: {lfs.is_empty()}")
print(f"Remaining items: {sum(1 for _ in iter(lambda: lfs.pop(), None))}")

# Thread-safe event bus
class ThreadSafeEventBus:
    def __init__(self):
        self._handlers = {}
        self._lock = threading.Lock()

    def subscribe(self, event_type, handler):
        with self._lock:
            if event_type not in self._handlers:
                self._handlers[event_type] = []
            self._handlers[event_type].append(handler)

    def unsubscribe(self, event_type, handler):
        with self._lock:
            if event_type in self._handlers:
                self._handlers[event_type].remove(handler)

    def publish(self, event_type, *args, **kwargs):
        handlers = []
        with self._lock:
            if event_type in self._handlers:
                handlers = list(self._handlers[event_type])

        for handler in handlers:
            try:
                handler(event_type, *args, **kwargs)
            except Exception as e:
                print(f"Handler error: {e}")

    def publish_safe(self, event_type, *args, **kwargs):
        handlers = []
        with self._lock:
            if event_type in self._handlers:
                handlers = list(self._handlers[event_type])

        results = []
        for handler in handlers:
            try:
                result = handler(event_type, *args, **kwargs)
                results.append(result)
            except Exception as e:
                results.append(e)
        return results

bus = ThreadSafeEventBus()

def on_data(event, data):
    print(f"Data handler: {data}")
    return f"processed-{data}"

def on_error(event, code, message):
    print(f"Error handler: {code} - {message}")

bus.subscribe("data", on_data)
bus.subscribe("error", on_error)

def publisher_thread():
    for i in range(5):
        bus.publish("data", f"value-{i}")
        time.sleep(0.1)
    bus.publish("error", 500, "Internal Server Error")

pub = threading.Thread(target=publisher_thread)
pub.start()
pub.join()

print("Event bus example completed")

# Concurrent data structure: Thread-safe LRU Cache
import time
from collections import OrderedDict

class ThreadSafeLRUCache:
    def __init__(self, capacity=100):
        self.capacity = capacity
        self._cache = OrderedDict()
        self._lock = threading.Lock()
        self._hits = 0
        self._misses = 0

    def get(self, key):
        with self._lock:
            if key in self._cache:
                self._cache.move_to_end(key)
                self._hits += 1
                return self._cache[key]
            self._misses += 1
            return None

    def put(self, key, value):
        with self._lock:
            if key in self._cache:
                self._cache.move_to_end(key)
            self._cache[key] = value
            if len(self._cache) > self.capacity:
                self._cache.popitem(last=False)

    def stats(self):
        with self._lock:
            total = self._hits + self._misses
            hit_rate = (self._hits / total * 100) if total > 0 else 0
            return {
                "hits": self._hits,
                "misses": self._misses,
                "hit_rate": f"{hit_rate:.1f}%",
                "size": len(self._cache),
                "capacity": self.capacity
            }

cache = ThreadSafeLRUCache(capacity=5)

def cache_worker(name):
    for i in range(10):
        key = f"key-{i % 7}"
        value = cache.get(key)
        if value is None:
            cache.put(key, f"value-{i}")
            print(f"{name}: cache miss for {key}")
        else:
            print(f"{name}: cache hit for {key}: {value}")
        time.sleep(random.uniform(0.05, 0.15))

workers = [threading.Thread(target=cache_worker, args=(f"W-{i}",)) for i in range(4)]
for w in workers: w.start()
for w in workers: w.join()

print(f"\nCache stats: {cache.stats()}")

# Non-blocking vs blocking operations benchmark
def benchmark_lock_approaches():
    import time
    from queue import Queue

    iterations = 100000

    results = {}

    lock = threading.Lock()
    value = [0]

    def lock_increment():
        for _ in range(iterations):
            with lock:
                value[0] += 1

    start = time.time()
    t1 = threading.Thread(target=lock_increment)
    t2 = threading.Thread(target=lock_increment)
    t1.start(); t2.start()
    t1.join(); t2.join()
    lock_time = time.time() - start
    results["lock"] = (lock_time, value[0])

    value[0] = 0

    rlock = threading.RLock()
    def rlock_increment():
        for _ in range(iterations):
            with rlock:
                value[0] += 1

    start = time.time()
    t1 = threading.Thread(target=rlock_increment)
    t2 = threading.Thread(target=rlock_increment)
    t1.start(); t2.start()
    t1.join(); t2.join()
    rlock_time = time.time() - start
    results["rlock"] = (rlock_time, value[0])

    for name, (elapsed, final) in results.items():
        print(f"{name}: {elapsed:.3f}s, final value: {final}, expected: {iterations * 2}")

benchmark_lock_approaches()
```

## Real-World Use Cases

- **Web servers**: Handling concurrent requests with thread-safe request/response handling
- **Database connection pools**: Thread-safe pool management with borrow/return semantics
- **Caching systems**: Thread-safe LRU caches in multi-threaded applications
- **Event systems**: Thread-safe publish-subscribe in GUI frameworks
- **Logging**: Thread-safe loggers that don't interleave log messages
- **State management**: Thread-safe configuration and state updates in server applications

## Common Mistakes

- Assuming simple operations like `x += 1` are atomic (they require multiple bytecode instructions)
- Using Lock when RLock is needed (causes deadlocks with recursive calls)
- Inconsistent lock ordering leading to deadlocks
- Holding locks during I/O operations or slow calls (reduces concurrency)
- Forgetting to release locks in exception paths (use `with lock:` to avoid this)
- Not using `try/except` or `timeout` in lock acquisition can cause indefinite blocking
- Using `threading.local()` for data that needs to be shared between threads
- Over-locking (using locks when thread-local storage would suffice)

## Best Practices

- Always use `with lock:` context manager instead of manual `acquire()`/`release()`
- Use `RLock` over `Lock` when the same thread might need to re-acquire
- Use `queue.Queue` for thread-safe communication instead of shared variables with locks
- Use `threading.local()` for data that is inherently thread-specific
- Keep critical sections as small as possible to maximize concurrency
- Use consistent lock ordering to prevent deadlocks
- Prefer `concurrent.futures.ThreadPoolExecutor` for thread management
- Add timeout to lock acquisition to detect potential deadlocks

## Interview Questions

1. **Q**: What is a race condition and how do you prevent it?
   **A**: A race condition occurs when two or more threads access shared data concurrently and the final result depends on the timing of their execution. Prevention uses locks, atomic operations, or thread-local storage.

2. **Q**: What is the difference between lock contention and deadlock?
   **A**: Lock contention is when threads compete for the same lock, causing delays but eventual progress. Deadlock is when threads hold locks while waiting for each other's locks, making no progress at all.

3. **Q**: How does `threading.local()` work internally?
   **A**: Each thread has a dictionary-like storage. When you access `threading.local()` attributes, it stores/retrieves values from the current thread's private storage, providing automatic isolation.

4. **Q**: What is the difference between a Lock and a Semaphore?
   **A**: A Lock allows only one thread to access a resource at a time (binary semaphore). A Semaphore allows a configurable number of concurrent accesses, useful for limiting connection pool usage.

5. **Q**: What is the difference between `concurrent.futures` and raw threading?
   **A**: `concurrent.futures` provides a higher-level API with thread pools, future objects for async results, callbacks, and unified interface for both threads and processes.

## Coding Challenges

1. **Thread-safe Cache**: Implement a thread-safe LRU cache with configurable capacity, supporting `get`/`put` operations with proper lock granularity.

2. **Concurrent Hash Map**: Build a hash map that supports concurrent reads and synchronized writes using read-write locks.

3. **Deadlock Detector**: Write a tool that monitors lock acquisitions and detects potential deadlocks using lock ordering graphs.

4. **Thread Pool with Priority**: Implement a thread pool that executes tasks based on priority, with proper synchronization.

5. **Lock-Free Ring Buffer**: Implement a lock-free single-producer single-consumer ring buffer using atomic operations.

## Summary

Thread safety is essential for correct concurrent programming. Python provides Lock, RLock, Semaphore, threading.local(), and Condition variables for synchronization. The `concurrent.futures` module simplifies thread pool management. Key practices include using context managers for locks, consistent lock ordering to prevent deadlocks, and preferring thread-safe queues over shared state.

## Related Topics

Multithreading (59.x), Multiprocessing (60.x), GIL (64.x), Async IO (61.x)
