# Multithreading - threading.Thread, Lock, Queue, concurrent.futures

## Introduction

Multithreading in Python allows multiple threads of execution to run within a single process, sharing the same memory space. The `threading` module provides a high-level interface for working with threads, including synchronization primitives like locks, semaphores, and events.

## Why It Is Important

Threads are ideal for I/O-bound tasks where the program spends time waiting for external resources (network requests, file operations, database queries). Using threads can significantly improve responsiveness and throughput by overlapping I/O waits with computation. Threads also share memory, making data sharing simpler than with processes.

## Syntax

```python
import threading

# Create a thread
t = threading.Thread(target=function, args=(arg1, arg2))
t.start()
t.join()

# Lock for synchronization
lock = threading.Lock()
lock.acquire()
try:
    # critical section
finally:
    lock.release()

# With context manager
with lock:
    # critical section

# Daemon thread
t = threading.Thread(target=function, daemon=True)
```

## Examples

### Basic Thread Creation and Joining

```python
import threading
import time
import random

def worker(name, delay):
    for i in range(3):
        time.sleep(delay)
        print(f"Worker {name}: iteration {i}")

threads = []
for i in range(3):
    t = threading.Thread(target=worker, args=(f"T-{i}", random.uniform(0.1, 0.5)))
    threads.append(t)
    t.start()
    print(f"Started thread {i}")

for t in threads:
    t.join()

print("All threads completed")
```

### Thread Class Subclassing

```python
import threading
import time

class CounterThread(threading.Thread):
    def __init__(self, name, count_to, delay=0.1):
        super().__init__(name=name)
        self.count_to = count_to
        self.delay = delay
        self.result = None

    def run(self):
        total = 0
        for i in range(self.count_to):
            total += i
            time.sleep(self.delay)
        self.result = total
        print(f"{self.name}: sum 0..{self.count_to - 1} = {total}")

threads = [
    CounterThread("Counter-A", 10, 0.05),
    CounterThread("Counter-B", 20, 0.03),
    CounterThread("Counter-C", 30, 0.02),
]

for t in threads:
    t.start()

for t in threads:
    t.join()
    print(f"Result from {t.name}: {t.result}")
```

### Daemon Threads

```python
import threading
import time

def background_monitor(name):
    count = 0
    while True:
        count += 1
        print(f"{name}: monitoring tick {count}")
        time.sleep(0.5)

def main_work():
    for i in range(3):
        print(f"Main work: step {i}")
        time.sleep(1)
    print("Main work done")

monitor = threading.Thread(target=background_monitor, args=("Monitor",), daemon=True)
print("Daemon thread started (will exit when main thread exits)")
monitor.start()

main_work()
print("Main thread exiting - daemon thread will be killed automatically")
```

### Threading.Lock — Mutual Exclusion

```python
import threading
import time

counter = 0
lock = threading.Lock()
increments_per_thread = 10000

def increment_counter():
    global counter
    for _ in range(increments_per_thread):
        with lock:
            counter += 1

threads = [
    threading.Thread(target=increment_counter)
    for _ in range(5)
]

for t in threads:
    t.start()

for t in threads:
    t.join()

expected = increments_per_thread * len(threads)
print(f"Counter: {counter}, Expected: {expected}, Correct: {counter == expected}")
```

### RLock — Reentrant Lock

```python
import threading
import time

rlock = threading.RLock()

def recursive_lock_example(level):
    with rlock:
        print(f"Level {level}: acquired lock")
        if level > 0:
            recursive_lock_example(level - 1)
        print(f"Level {level}: releasing lock")

t = threading.Thread(target=recursive_lock_example, args=(3,))
t.start()
t.join()
print("RLock example completed")

def deadlock_with_lock():
    lock = threading.Lock()
    with lock:
        print("Acquired lock")
        with lock:  # This would deadlock with a normal Lock
            print("This won't print with Lock")

def safe_with_rlock():
    lock = threading.RLock()
    with lock:
        print("RLock: acquired first time")
        with lock:
            print("RLock: acquired second time (same thread)")

try:
    deadlock_with_lock()
except RuntimeError as e:
    print(f"Lock deadlock: {e}")

safe_with_rlock()
```

### Semaphore — Limit Concurrent Access

```python
import threading
import time
import random

semaphore = threading.Semaphore(3)
active = []
active_lock = threading.Lock()

def limited_worker(name):
    with semaphore:
        with active_lock:
            active.append(name)
            print(f"Active: {active}")

        time.sleep(random.uniform(0.5, 1.5))

        with active_lock:
            active.remove(name)

threads = [
    threading.Thread(target=limited_worker, args=(f"W-{i}",))
    for i in range(10)
]

for t in threads:
    t.start()

for t in threads:
    t.join()

print("All limited workers completed")
```

### Event — Thread Signaling

```python
import threading
import time

event = threading.Event()

def waiter(name):
    print(f"{name}: waiting for event...")
    event.wait()
    print(f"{name}: event received! proceeding")

def setter():
    time.sleep(2)
    print("Setter: setting event in 3 seconds...")
    time.sleep(3)
    event.set()
    print("Setter: event set!")

waiters = [
    threading.Thread(target=waiter, args=(f"Waiter-{i}",))
    for i in range(3)
]

setter_thread = threading.Thread(target=setter)

for w in waiters:
    w.start()

setter_thread.start()

for w in waiters:
    w.join()

setter_thread.join()
print("Event example completed")

event.clear()
print("Event cleared for reuse")
```

### Condition — Producer-Consumer

```python
import threading
import time
import random

condition = threading.Condition()
buffer = []
BUFFER_MAX = 5

def producer(name):
    for i in range(10):
        with condition:
            while len(buffer) >= BUFFER_MAX:
                print(f"{name}: buffer full, waiting...")
                condition.wait()

            item = f"{name}-item-{i}"
            buffer.append(item)
            print(f"{name}: produced {item} (buffer: {len(buffer)})")
            condition.notify_all()

        time.sleep(random.uniform(0.1, 0.3))

def consumer(name):
    consumed = 0
    while consumed < 15:
        with condition:
            while not buffer:
                print(f"{name}: buffer empty, waiting...")
                condition.wait()

            item = buffer.pop(0)
            consumed += 1
            print(f"{name}: consumed {item} (buffer: {len(buffer)})")
            condition.notify_all()

        time.sleep(random.uniform(0.2, 0.4))

producers = [
    threading.Thread(target=producer, args=(f"P-{i}",))
    for i in range(2)
]

consumers = [
    threading.Thread(target=consumer, args=(f"C-{i}",))
    for i in range(3)
]

for t in producers + consumers:
    t.start()

for t in producers + consumers:
    t.join()

print("Producer-consumer completed")
```

### Queue — Thread-Safe Queue

```python
import threading
import time
import random
from queue import Queue

def producer(q, name):
    for i in range(5):
        item = f"{name}-item-{i}"
        time.sleep(random.uniform(0.1, 0.3))
        q.put(item)
        print(f"{name}: queued {item}")
    q.put(None)

def consumer(q, name):
    while True:
        item = q.get()
        if item is None:
            q.task_done()
            break
        print(f"{name}: processing {item}")
        time.sleep(random.uniform(0.2, 0.5))
        q.task_done()

q = Queue(maxsize=3)

producers = [
    threading.Thread(target=producer, args=(q, f"P-{i}"))
    for i in range(2)
]

consumers = [
    threading.Thread(target=consumer, args=(q, f"C-{i}"))
    for i in range(3)
]

for t in consumers:
    t.start()

for t in producers:
    t.start()

for t in producers:
    t.join()

q.join()
print("Queue example completed")
```

### Timer — Delayed Execution

```python
import threading
import time

def delayed_action(name):
    print(f"{name}: executed at {time.time():.1f}")

print(f"Starting at {time.time():.1f}")

timer1 = threading.Timer(1.0, delayed_action, args=("Timer-1",))
timer2 = threading.Timer(2.0, delayed_action, args=("Timer-2",))
timer3 = threading.Timer(3.0, delayed_action, args=("Timer-3",))

timer1.start()
timer2.start()
timer3.start()

timer1.cancel()
print("Timer-1 cancelled")

time.sleep(4)
print("Timer example completed")
```

### Barrier — Synchronize N Threads

```python
import threading
import time
import random

barrier = threading.Barrier(5, action=lambda: print("\n=== Barrier reached! All threads proceed ===\n"))

def barrier_worker(name):
    for i in range(3):
        time.sleep(random.uniform(0.5, 1.5))
        print(f"{name}: reached barrier {i}")
        barrier.wait()
    print(f"{name}: completed all phases")

threads = [
    threading.Thread(target=barrier_worker, args=(f"W-{i}",))
    for i in range(5)
]

for t in threads:
    t.start()

for t in threads:
    t.join()

print("Barrier example completed")
```

## Beginner Examples

```python
# Simple thread pool from scratch
import threading
import time
from queue import Queue

class SimpleThreadPool:
    def __init__(self, num_workers):
        self.tasks = Queue()
        self.workers = []
        self._stop = threading.Event()

        for _ in range(num_workers):
            t = threading.Thread(target=self._worker_loop, daemon=True)
            t.start()
            self.workers.append(t)

    def _worker_loop(self):
        while not self._stop.is_set():
            try:
                func, args, kwargs = self.tasks.get(timeout=0.1)
                try:
                    func(*args, **kwargs)
                except Exception as e:
                    print(f"Task error: {e}")
                finally:
                    self.tasks.task_done()
            except Exception:
                pass

    def submit(self, func, *args, **kwargs):
        self.tasks.put((func, args, kwargs))

    def wait_completion(self):
        self.tasks.join()

    def shutdown(self):
        self._stop.set()

def sample_task(name, duration):
    time.sleep(duration)
    print(f"Task {name} completed (took {duration}s)")

pool = SimpleThreadPool(3)
for i in range(6):
    pool.submit(sample_task, f"T-{i}", 0.5)

pool.wait_completion()
print("All tasks completed")
pool.shutdown()

# Thread-local data
thread_local = threading.local()

def set_and_print_value(name):
    thread_local.value = name
    time.sleep(0.1)
    print(f"{name}: thread-local value = {thread_local.value}")

threads = [
    threading.Thread(target=set_and_print_value, args=(f"W-{i}",))
    for i in range(5)
]

for t in threads:
    t.start()

for t in threads:
    t.join()

print("Thread-local data example completed")
```

## Intermediate Examples

```python
# Rate limiter using threading primitives
import threading
import time
from collections import deque

class RateLimiter:
    def __init__(self, max_calls, period=1.0):
        self.max_calls = max_calls
        self.period = period
        self.calls = deque()
        self.lock = threading.Lock()

    def acquire(self):
        with self.lock:
            now = time.time()
            while self.calls and self.calls[0] <= now - self.period:
                self.calls.popleft()

            if len(self.calls) >= self.max_calls:
                wait_time = self.calls[0] + self.period - now
                if wait_time > 0:
                    print(f"Rate limit reached, waiting {wait_time:.2f}s")
                    self.lock.release()
                    time.sleep(wait_time)
                    self.lock.acquire()
                self.calls.popleft()

            self.calls.append(time.time())

    def __enter__(self):
        self.acquire()
        return self

    def __exit__(self, *args):
        pass

limiter = RateLimiter(max_calls=3, period=1.0)

def rate_limited_task(name):
    with limiter:
        print(f"{name}: executing at {time.time():.2f}")
        time.sleep(0.2)

threads = [
    threading.Thread(target=rate_limited_task, args=(f"T-{i}",))
    for i in range(10)
]

for t in threads:
    t.start()

for t in threads:
    t.join()

print("Rate limiter example completed")

# Read-Write Lock implementation
import threading

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

    @property
    def reader_lock(self):
        return _ReaderLock(self)

    @property
    def writer_lock(self):
        return _WriterLock(self)

class _ReaderLock:
    def __init__(self, rwlock):
        self.rwlock = rwlock
    def __enter__(self):
        self.rwlock.acquire_read()
    def __exit__(self, *args):
        self.rwlock.release_read()

class _WriterLock:
    def __init__(self, rwlock):
        self.rwlock = rwlock
    def __enter__(self):
        self.rwlock.acquire_write()
    def __exit__(self, *args):
        self.rwlock.release_write()

rwlock = ReadWriteLock()
shared_data = {}
```

## Advanced Examples

```python
# Thread pool with result collection and error handling
import threading
import time
import random
from queue import Queue
from typing import Callable, Any, List, Tuple

class Future:
    def __init__(self):
        self._event = threading.Event()
        self._result = None
        self._exception = None

    def set_result(self, value):
        self._result = value
        self._event.set()

    def set_exception(self, exc):
        self._exception = exc
        self._event.set()

    def result(self, timeout=None):
        self._event.wait(timeout)
        if self._exception:
            raise self._exception
        return self._result

class AdvancedThreadPool:
    def __init__(self, num_workers: int, name: str = "Pool"):
        self.name = name
        self._work_queue: Queue = Queue()
        self._workers: List[threading.Thread] = []
        self._stop = threading.Event()

        for i in range(num_workers):
            t = threading.Thread(
                target=self._worker_loop,
                name=f"{name}-Worker-{i}",
                daemon=True
            )
            t.start()
            self._workers.append(t)

    def _worker_loop(self):
        while not self._stop.is_set():
            try:
                work = self._work_queue.get(timeout=0.1)
                if work is None:
                    self._work_queue.task_done()
                    break
                func, args, kwargs, future = work
                try:
                    result = func(*args, **kwargs)
                    future.set_result(result)
                except Exception as e:
                    future.set_exception(e)
                finally:
                    self._work_queue.task_done()
            except Exception:
                pass

    def submit(self, func: Callable, *args: Any, **kwargs: Any) -> Future:
        future = Future()
        self._work_queue.put((func, args, kwargs, future))
        return future

    def map(self, func: Callable, iterable: List[Any]) -> List[Any]:
        futures = [self.submit(func, item) for item in iterable]
        return [f.result() for f in futures]

    def shutdown(self, wait: bool = True):
        self._stop.set()
        if wait:
            for _ in self._workers:
                self._work_queue.put(None)
            for w in self._workers:
                w.join()

pool = AdvancedThreadPool(4, "AdvancedPool")

def compute_square(n):
    time.sleep(random.uniform(0.1, 0.3))
    if n < 0:
        raise ValueError(f"Negative input: {n}")
    return n * n

futures = []
for i in range(-1, 10):
    future = pool.submit(compute_square, i)
    futures.append(future)

for i, future in enumerate(futures):
    try:
        result = future.result(timeout=2.0)
        print(f"Result {i}: {result}")
    except ValueError as e:
        print(f"Result {i}: Error - {e}")
    except Exception as e:
        print(f"Result {i}: Timeout or error - {e}")

results = pool.map(compute_square, [1, 2, 3, 4, 5])
print(f"\nMap results: {results}")

pool.shutdown()
print("Advanced thread pool shutdown")

# Non-blocking event-driven timer system
class TimerWheel:
    def __init__(self, num_buckets=10, resolution=0.1):
        self.resolution = resolution
        self.wheel = [[] for _ in range(num_buckets)]
        self.current = 0
        self.lock = threading.Lock()
        self.running = threading.Event()
        self.thread = None

    def start(self):
        self.running.set()
        self.thread = threading.Thread(target=self._tick_loop, daemon=True)
        self.thread.start()

    def stop(self):
        self.running.clear()
        if self.thread:
            self.thread.join()

    def add_timer(self, delay, callback, *args):
        ticks = int(delay / self.resolution)
        bucket = (self.current + ticks) % len(self.wheel)
        with self.lock:
            self.wheel[bucket].append((callback, args))

    def _tick_loop(self):
        while self.running.is_set():
            time.sleep(self.resolution)
            with self.lock:
                expired = self.wheel[self.current]
                self.wheel[self.current] = []
            self.current = (self.current + 1) % len(self.wheel)
            for callback, args in expired:
                try:
                    callback(*args)
                except Exception as e:
                    print(f"Timer callback error: {e}")

wheel = TimerWheel(num_buckets=10, resolution=0.1)
wheel.start()

def on_timer(name):
    print(f"Timer {name} fired at {time.time():.2f}")

wheel.add_timer(0.3, on_timer, "A")
wheel.add_timer(0.7, on_timer, "B")
wheel.add_timer(1.5, on_timer, "C")

time.sleep(2.0)
wheel.stop()
print("Timer wheel example completed")
```

## Real-World Use Cases

- **Web scraping**: Concurrent HTTP requests to multiple URLs
- **GUI applications**: Keeping the UI responsive while doing background work
- **Database operations**: Concurrent queries to improve throughput
- **File I/O**: Reading/writing multiple files simultaneously
- **Network servers**: Handling multiple client connections
- **Real-time data processing**: Processing streaming data with multiple consumers

## Common Mistakes

- Forgetting that Python threads are not truly parallel for CPU-bound tasks due to the GIL
- Not using locks when accessing shared data, leading to race conditions
- Using `Lock` when `RLock` is needed (e.g., recursive calls within the same thread)
- Creating too many threads, causing context switching overhead
- Not setting threads as daemon, causing the program to hang on exit
- Starting a thread multiple times or joining a thread that hasn't started
- Holding locks across I/O operations (reduces concurrency)
- Ignoring `threading.local()` when thread-local storage is needed

## Best Practices

- Use `threading.Thread` with `target=` for simple tasks; subclass for complex thread logic
- Always use `with lock:` context manager instead of manual `acquire()/release()`
- Prefer `queue.Queue` for thread-safe communication over shared variables with locks
- Use `RLock` instead of `Lock` when a thread might need to re-acquire the same lock
- Keep critical sections as short as possible
- Use daemon threads for background tasks that should not block program exit
- Limit thread count using `Semaphore` or `ThreadPoolExecutor`
- Use `threading.local()` for thread-local data to avoid shared state

## Interview Questions

1. **Q**: What is the difference between a thread and a process?
   **A**: Threads share the same memory space within a process and are lighter weight. Processes have isolated memory and require IPC for communication. Threads are affected by the GIL; processes are not.

2. **Q**: How does the GIL affect threading in Python?
   **A**: The GIL prevents multiple threads from executing Python bytecode simultaneously, making CPython threads unsuitable for CPU-bound parallel computation. For I/O-bound tasks, the GIL is released during blocking operations, allowing concurrency.

3. **Q**: What is a daemon thread?
   **A**: A daemon thread runs in the background and is automatically killed when all non-daemon threads exit. Daemon threads are useful for background monitoring tasks that should not prevent program termination.

4. **Q**: Explain the difference between Lock and RLock.
   **A**: A Lock can only be acquired once and will deadlock if the same thread tries to acquire it again. An RLock (Reentrant Lock) can be acquired multiple times by the same thread without deadlocking, as long as releases match acquires.

5. **Q**: What is the purpose of `threading.local()`?
   **A**: It provides thread-local storage, allowing each thread to have its own independent copy of data. This avoids the need for locks when each thread only accesses its own data.

## Coding Challenges

1. **Thread-safe Counter**: Implement a counter that can be safely incremented by multiple threads, using both Lock and atomic operations. Benchmark both approaches.

2. **Bounded Blocking Queue**: Implement a thread-safe bounded queue without using `queue.Queue`. Use Condition variables for blocking put/get.

3. **Parallel Web Scraper**: Write a multi-threaded web scraper that downloads URLs concurrently, respects robots.txt, and implements rate limiting.

4. **Request Throttler**: Implement a sliding window rate limiter using threading primitives that limits requests per second across all threads.

5. **Multi-Threaded Merge Sort**: Implement a parallel merge sort that spawns threads for recursive subdivisions and merges results.

## Summary

The `threading` module provides a comprehensive set of tools for concurrent programming in Python: threads, locks (Lock, RLock), Semaphore, Event, Condition, Barrier, and Timer. While the GIL limits CPU-bound parallelism, threading excels at I/O-bound tasks. Proper use of synchronization primitives is essential for writing correct, race-condition-free concurrent code.

## Related Topics

Multiprocessing (60.x), Thread Safety (63.x), GIL (64.x), Async IO (61.x)
