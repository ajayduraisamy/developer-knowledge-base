# GIL - Global Interpreter Lock, impact, workarounds (Python 3.13+ free-threading)
## Introduction
The Global Interpreter Lock (GIL) is a mutex in CPython that protects access to Python objects, preventing multiple threads from executing Python bytecode simultaneously. This file explains what the GIL is, how it affects performance, workarounds for CPU-bound code, and Python 3.13+'s free-threading mode that makes the GIL optional.

## Global Interpreter Lock
### What It Is
The GIL is a mutex that allows only one thread to execute Python bytecode at a time in a single process. It is part of CPython (the reference Python implementation), not a language requirement. Jython and IronPython do not have a GIL.

### Why It Is Important
The GIL simplifies CPython internals by making memory management (reference counting) thread-safe without needing fine-grained locks on every object. This keeps single-threaded performance high and simplifies C extension development. However, it limits multi-threaded CPU-bound parallelism.

### How It Works Internally
The GIL is released and re-acquired periodically (every ~5ms in Python 3.2+, via `PyEval_EvalFrameEx`). The mechanism:
1. A thread holding the GIL executes bytecode for one "tick"
2. After the tick, it releases the GIL via a condition variable
3. All waiting threads compete to re-acquire the GIL
4. I/O operations explicitly release the GIL before blocking and re-acquire after

The GIL switching uses `PyThread_release_lock` and `PyThread_acquire_lock` around the condition variable. Python 3.2+ uses a more efficient mechanism that reduces switching overhead.

### Syntax
```python
import sys
sys._enable_gil  # (Python 3.13+) check if GIL is disabled

# The GIL is invisible in normal Python code
# You never directly interact with it
```

### Beginner Examples
```python
import threading
import time

def count_down(n):
    while n > 0:
        n -= 1

# CPU-bound function -- GIL prevents parallel speedup
n = 10000000
start = time.time()
count_down(n)
single_time = time.time() - start
print(f"Single thread: {single_time:.3f}s")

# Two threads -- no speedup (GIL limits parallelism)
def thread_worker():
    count_down(n // 2)

t1 = threading.Thread(target=thread_worker)
t2 = threading.Thread(target=thread_worker)
start = time.time()
t1.start()
t2.start()
t1.join()
t2.join()
parallel_time = time.time() - start
print(f"Two threads: {parallel_time:.3f}s")
print(f"Ratio: {single_time / parallel_time:.2f}")  # ~1.0, not 2.0!
```

### Intermediate Examples
```python
import threading
import time
import math

# GIL impact visualization
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(math.sqrt(n)) + 1):
        if n % i == 0:
            return False
    return True

def count_primes(start, end):
    count = 0
    for n in range(start, end):
        if is_prime(n):
            count += 1
    return count

# Sequential
start = time.time()
r1 = count_primes(1, 50000)
r2 = count_primes(50001, 100000)
seq_time = time.time() - start
print(f"Sequential: {seq_time:.3f}s, primes: {r1 + r2}")

# Threaded (GIL-bound -- similar time)
results = [0, 0]
def worker1():
    results[0] = count_primes(1, 50000)
def worker2():
    results[1] = count_primes(50001, 100000)

t1 = threading.Thread(target=worker1)
t2 = threading.Thread(target=worker2)
start = time.time()
t1.start(); t2.start()
t1.join(); t2.join()
thread_time = time.time() - start
print(f"Threaded: {thread_time:.3f}s, primes: {results[0] + results[1]}")
print(f"Speedup: {seq_time / thread_time:.2f}x")

# I/O-bound tasks DO benefit from threading despite GIL
def io_task(delay):
    time.sleep(delay)
    return delay

start = time.time()
io_task(1)
io_task(1)
seq_io = time.time() - start

start = time.time()
t1 = threading.Thread(target=io_task, args=(1,))
t2 = threading.Thread(target=io_task, args=(1,))
t1.start(); t2.start()
t1.join(); t2.join()
thread_io = time.time() - start
print(f"I/O sequential: {seq_io:.3f}s, threaded: {thread_io:.3f}s")
print(f"I/O speedup: {seq_io / thread_io:.2f}x")
```

### Advanced Examples
```python
import threading
import time
import sys

# GIL release in I/O and C extensions
# The GIL is released automatically for blocking I/O operations
# C extensions can manually release the GIL using Py_BEGIN_ALLOW_THREADS

# Example: numpy releases the GIL (C extension)
import numpy as np

def numpy_work():
    a = np.random.rand(1000, 1000)
    b = np.random.rand(1000, 1000)
    for _ in range(100):
        np.dot(a, b)

# Two threads -- numpy releases GIL, so this actually parallelizes
start = time.time()
t1 = threading.Thread(target=numpy_work)
t2 = threading.Thread(target=numpy_work)
t1.start(); t2.start()
t1.join(); t2.join()
numpy_time = time.time() - start
print(f"Numpy threaded: {numpy_time:.3f}s")

# Sequential for comparison
start = time.time()
numpy_work()
numpy_work()
seq_numpy = time.time() - start
print(f"Numpy sequential: {seq_numpy:.3f}s")
print(f"Numpy speedup: {seq_numpy / numpy_time:.2f}x")

# GIL checking utility
def check_gil_status():
    if hasattr(sys, "_enable_gil"):
        # Python 3.13+
        print(f"GIL enabled: {sys._enable_gil}")
    else:
        print(f"Python {sys.version}")
        print("GIL is always enabled in this Python version")

# Measuring GIL contention
def gil_contention():
    lock = threading.Lock()
    data = []

    def cpu_work():
        for i in range(1000000):
            # CPU work that doesn't call any C extension
            x = i * i + i
        with lock:
            data.append(1)

    start = time.time()
    threads = [threading.Thread(target=cpu_work) for _ in range(4)]
    for t in threads: t.start()
    for t in threads: t.join()
    total = time.time() - start
    print(f"4 threads (GIL contention): {total:.3f}s")
    # About the same as single-threaded

gil_contention()
```

### Real-World Use Cases
- Understanding why threading doesn't speed up CPU-bound Python
- Choosing between threading (I/O) and multiprocessing (CPU)
- Optimizing C extension development (releasing GIL)
- Migration planning for Python 3.13+ free-threading
- Debugging performance issues in concurrent applications

### Common Mistakes
- Expecting thread-level parallelism for CPU-bound pure Python code
- Blaming the GIL for all performance issues (often I/O bound)
- Assuming GIL removal is a magic bullet (requires locks everywhere)
- Not using multiprocessing for CPU-bound parallel work

### Best Practices
- Use threading for I/O-bound tasks (network, disk, database)
- Use multiprocessing for CPU-bound tasks
- Use C extensions (numpy, pandas, numba) that release the GIL
- Profile to confirm the GIL is actually your bottleneck
- Consider asyncio for high-concurrency I/O workloads
- For Python 3.13+, evaluate free-threading mode for CPU-bound multi-threaded code

### Performance Considerations
- Pure Python CPU-bound code gets no benefit from threading
- I/O-bound code works well with threading (GIL released during I/O)
- C extensions can release the GIL, achieving parallelism in native code
- GIL contention adds ~10-20% overhead even for single-threaded programs (negligible)
- Context switching between threads adds small overhead
- The GIL's tick interval (~5ms) is a heuristic -- too short wastes CPU, too long hurts responsiveness

### Interview Questions
1. What is the GIL and why does Python have it?
2. How does the GIL affect threading performance?
3. When does threading still benefit despite the GIL?
4. Does the GIL affect asyncio or multiprocessing?
5. How do C extensions like numpy achieve parallelism?
6. What changed about the GIL in Python 3.13?

### Coding Challenges
- Write a benchmark that demonstrates GIL impact on CPU-bound vs I/O-bound tasks
- Create a script that checks if the Python runtime has the GIL enabled
- Compare threading vs multiprocessing for a CPU-intensive task

### Related Topics
- Threading, multiprocessing, free-threading, asyncio, C extensions

## Performance impact
### What It Is
The GIL's performance impact depends on the workload type. CPU-bound pure Python code sees no parallel speedup with threads. I/O-bound code sees full parallel benefit. Mixed workloads fall in between.

### Why It Is Important
Understanding when the GIL is a bottleneck prevents misdesign -- using threads when multiprocessing or asyncio is needed, or expecting parallelism where the GIL prevents it.

### How It Works Internally
When multiple threads compete for the GIL:
- Each thread runs for ~5ms (the "tick")
- Then releases the GIL and competes with others to re-acquire
- The OS scheduler decides which thread runs next
- Waiting threads spin briefly, then sleep on a condition variable
- The overhead of GIL contention can be up to ~20% per thread

### Beginner Examples
```python
import threading
import time

def heavy_computation():
    total = 0
    for i in range(5000000):
        total += i * i
    return total

# Single thread
start = time.time()
heavy_computation()
heavy_computation()
single = time.time() - start

# Two threads
start = time.time()
t1 = threading.Thread(target=heavy_computation)
t2 = threading.Thread(target=heavy_computation)
t1.start(); t2.start()
t1.join(); t2.join()
parallel = time.time() - start

print(f"Single: {single:.3f}s")
print(f"Two threads: {parallel:.3f}s")
# parallel is approximately equal to single (no speedup)
```

### Intermediate Examples
```python
import threading
import time

# GIL impact on scalability
def measure_scalability(worker_fn, num_workers_range):
    for n in num_workers_range:
        start = time.time()
        threads = [threading.Thread(target=worker_fn) for _ in range(n)]
        for t in threads: t.start()
        for t in threads: t.join()
        elapsed = time.time() - start
        print(f"{n} workers: {elapsed:.3f}s")

# CPU-bound
def cpu_work():
    x = 0
    for _ in range(2000000):
        x += 1

print("CPU-bound (GIL limits):")
measure_scalability(cpu_work, [1, 2, 4, 8])

# I/O-bound
def io_work():
    time.sleep(1)

print("I/O-bound (GIL not an issue):")
measure_scalability(io_work, [1, 2, 4, 8])
```

### Advanced Examples
```python
import threading
import time
import sys

# Measuring GIL contention overhead
def gil_overhead_measurement():
    lock = threading.Lock()
    counter = [0]

    def contended_worker():
        for _ in range(100000):
            with lock:
                counter[0] += 1

    # Measure with varying thread counts
    for n_threads in [1, 2, 4, 8]:
        start = time.time()
        threads = [threading.Thread(target=contended_worker) for _ in range(n_threads)]
        for t in threads: t.start()
        for t in threads: t.join()
        elapsed = time.time() - start
        ops_per_sec = 100000 * n_threads / elapsed
        print(f"{n_threads} threads: {elapsed:.3f}s, {ops_per_sec:.0f} ops/s")

    print("\nGIL contention overhead increases with thread count")

# Mixed CPU and I/O workload
def mixed_workload(cpu_fraction=0.5):
    start = time.time()
    def worker():
        for _ in range(10):
            # CPU phase
            x = 0
            for i in range(100000):
                x += i
            # I/O phase (sleep releases GIL)
            time.sleep(0.01)

    threads = [threading.Thread(target=worker) for _ in range(4)]
    for t in threads: t.start()
    for t in threads: t.join()
    return time.time() - start

print(f"\nMixed workload: {mixed_workload():.3f}s")
```

### Real-World Use Cases
- Web servers -- I/O bound, threading works well
- Data processing -- if CPU-bound and pure Python, need multiprocessing
- ML inference -- model forward pass in C++/CUDA (releases GIL)
- Desktop apps -- threading works for UI responsiveness

### Common Mistakes
- Assuming threading always speeds things up
- Using threads for CPU-heavy data science code
- Blaming GIL without profiling first
- Not checking if a C extension releases the GIL

### Best Practices
- Profile before optimizing for the GIL
- Use multiprocessing for CPU-bound pure Python
- Use C extensions that release the GIL for numerical work
- For mixed workloads, use a combination of threads and processes
- Consider asyncio for very high-concurrency I/O

### Performance Considerations
- GIL contention adds ~5-20% overhead per additional thread
- Thread count beyond CPU cores increases contention
- Short tasks suffer more from GIL overhead than long tasks
- Python 3.2+ GIL improvements significantly reduced switching overhead

### Interview Questions
1. Does the GIL affect single-threaded performance?
2. What workloads benefit from threads despite the GIL?
3. How does a C extension indicate it releases the GIL?

### Coding Challenges
- Write a script that finds the optimal thread count for an I/O-bound workload
- Benchmark threading vs multiprocessing for a CPU-intensive task

### Related Topics
- Threading, multiprocessing, profiling, C extensions, numba

## Workarounds
### What It Is
Workarounds to overcome GIL limitations include using multiprocessing, C extensions that release the GIL, asyncio for I/O concurrency, and just-in-time compilation (numba).

### Why It Is Important
Knowing GIL workarounds is essential for writing performant concurrent Python. The right workaround depends on the workload type (CPU, I/O, mixed) and constraints (memory, latency, library availability).

### How It Works Internally
- Multiprocessing: each process has its own GIL and interpreter
- C extensions: use Py_BEGIN_ALLOW_THREADS / Py_END_ALLOW_THREADS macros to release GIL during native computation
- Numba: compiles Python to machine code, releasing the GIL via nogil=True
- asyncio: single-threaded cooperative multitasking (no GIL contention)

### Beginner Examples
```python
import time
import math

# Workaround 1: multiprocessing (true parallelism)
from multiprocessing import Pool

def cpu_intensive(x):
    total = 0
    for i in range(5000000):
        total += i * i
    return total

start = time.time()
with Pool(processes=2) as pool:
    results = pool.map(cpu_intensive, [1, 1])
multiprocess_time = time.time() - start

# Compare with sequential
start = time.time()
cpu_intensive(1)
cpu_intensive(1)
sequential_time = time.time() - start

print(f"Sequential: {sequential_time:.3f}s")
print(f"Multiprocessing: {multiprocess_time:.3f}s")
print(f"Speedup: {sequential_time / multiprocess_time:.2f}x")
```

### Intermediate Examples
```python
import threading
import time
import math
from multiprocessing import Process, Queue

# Workaround 2: releasing GIL with C extension (numpy)
import numpy as np

def numpy_parallel():
    a = np.random.rand(500, 500)
    b = np.random.rand(500, 500)
    for _ in range(50):
        np.dot(a, b)

start = time.time()
t1 = threading.Thread(target=numpy_parallel)
t2 = threading.Thread(target=numpy_parallel)
t1.start(); t2.start()
t1.join(); t2.join()
numpy_time = time.time() - start

start = time.time()
numpy_parallel()
numpy_parallel()
single_time = time.time() - start
print(f"Numpy single: {single_time:.3f}s, threaded: {numpy_time:.3f}s")
print(f"Speedup: {single_time / numpy_time:.2f}x (C ext releases GIL)")

# Workaround 3: asyncio for I/O concurrency
import asyncio

async def async_io_task():
    await asyncio.sleep(1)

async def async_concurrent():
    await asyncio.gather(*[async_io_task() for _ in range(10)])

start = time.time()
asyncio.run(async_concurrent())
async_time = time.time() - start
print(f"Async 10 tasks: {async_time:.3f}s (single-threaded, non-blocking)")

# Workaround 4: numba JIT with nogil
try:
    from numba import njit

    @njit(nogil=True)
    def numba_work(n):
        total = 0
        for i in range(n):
            total += i * i
        return total

    def threaded_numba():
        start = time.time()
        t1 = threading.Thread(target=numba_work, args=(10000000,))
        t2 = threading.Thread(target=numba_work, args=(10000000,))
        t1.start(); t2.start()
        t1.join(); t2.join()
        print(f"Numba threaded: {time.time() - start:.3f}s")
except ImportError:
    print("numba not available")
```

### Advanced Examples
```python
import time
from multiprocessing import Process, shared_memory
import numpy as np

# Workaround 5: hybrid threading + multiprocessing
def hybrid_approach(data_size=1000000):
    """Use multiprocessing for CPU work, threading for I/O coordination."""
    from multiprocessing import Pool as MPool
    from concurrent.futures import ThreadPoolExecutor

    # Heavy computation in processes
    def compute_chunk(chunk):
        return np.sum(np.square(chunk))

    data = np.random.rand(data_size)
    chunks = np.array_split(data, 4)

    with MPool(processes=4) as pool:
        partial_sums = pool.map(compute_chunk, chunks)

    # I/O coordination in threads
    results = []
    def save_result(r):
        results.append(r)

    with ThreadPoolExecutor(max_workers=4) as ex:
        futures = [ex.submit(save_result, s) for s in partial_sums]

    return sum(results)

start = time.time()
total = hybrid_approach()
print(f"Hybrid result: {total:.2f}, time: {time.time() - start:.3f}s")

# Workaround 6: selective GIL release via subinterpreters (PEP 554)
# Python 3.12+ has subinterpreters, but they are experimental
try:
    import _xxsubinterpreters as interpreters

    def subinterpreter_demo():
        interp = interpreters.create()
        interpreters.run_string(interp, "result = sum(range(1000000))")
        # Each interpreter has its own GIL
        print(f"Subinterpreter created")
except ImportError:
    print("Subinterpreters not available")

# Workaround 7: Cython with nogil
# # cython: language_level=3, boundscheck=False, wraparound=False
# def cython_work(double[:] arr) nogil:
#     cdef double total = 0
#     cdef int i
#     for i in range(arr.shape[0]):
#         total += arr[i] * arr[i]
#     return total

# Performance comparison of all approaches
def benchmark_workarounds():
    import math

    def cpu_task(n=5000000):
        total = 0
        for i in range(n):
            total += i * i
        return total

    results = {}

    # Sequential
    start = time.time()
    cpu_task()
    cpu_task()
    results["sequential"] = time.time() - start

    # Threading
    start = time.time()
    t1 = threading.Thread(target=cpu_task)
    t2 = threading.Thread(target=cpu_task)
    t1.start(); t2.start()
    t1.join(); t2.join()
    results["threading"] = time.time() - start

    # Multiprocessing
    from multiprocessing import Process
    q = Queue()
    def task_with_queue():
        q.put(cpu_task())
    start = time.time()
    p1 = Process(target=task_with_queue)
    p2 = Process(target=task_with_queue)
    p1.start(); p2.start()
    p1.join(); p2.join()
    results["multiprocessing"] = time.time() - start

    for approach, elapsed in results.items():
        print(f"{approach}: {elapsed:.3f}s")
```

### Real-World Use Cases
- Data science: use numpy/pandas (C extensions release GIL)
- Web scraping: threading for network I/O, multiprocessing for parsing
- ML training: multiprocessing for data loading, GPU for computation (GIL irrelevant)
- Real-time systems: asyncio for single-threaded high concurrency
- Desktop apps: main thread for GUI, worker threads for I/O, processes for computation

### Common Mistakes
- Using multiprocessing for I/O-bound tasks (unnecessary overhead)
- Using multiprocessing for tasks that don't benefit from parallelism
- Forgetting that multiprocessing requires pickling data
- Not checking if a library already handles parallelism internally

### Best Practices
- Match the workaround to the workload type
- Use multiprocessing for CPU-bound pure Python
- Use threading for I/O-bound tasks
- Use asyncio for very high-concurrency I/O
- Use C extensions (numpy, numba) that release the GIL
- Consider hybrid approaches for mixed workloads
- Measure before and after to verify improvement

### Performance Considerations
- Multiprocessing overhead: process creation + data serialization
- C extension GIL release: zero overhead during native computation
- Asyncio: lowest overhead for I/O concurrency (single thread)
- Numba JIT: compilation time upfront, then fast execution

### Interview Questions
1. What are the main workarounds for GIL limitations?
2. When would you use threading vs multiprocessing?
3. How do C extensions achieve parallelism despite the GIL?
4. Does asyncio bypass the GIL?

### Coding Challenges
- Write a benchmark comparing threading, multiprocessing, and asyncio for a mixed workload
- Convert a CPU-bound function to use multiprocessing with a Pool

### Related Topics
- Multiprocessing, asyncio, numba, C extensions, subinterpreters

## Free-threading (Python 3.13+)
### What It Is
Python 3.13 introduces experimental support for disabling the GIL (PEP 703), enabling true multi-threaded parallelism for CPU-bound Python code. This is called "free-threading" or "no-GIL" mode.

### Why It Is Important
Free-threading finally removes the biggest limitation of CPython threading. It allows Python to scale to multiple cores with threads, simplifying concurrent programming and potentially matching performance of languages without a GIL.

### How It Works Internally
Without the GIL, CPython must make all object operations thread-safe through:
- Biased reference counting (per-thread refcount with occasional merging)
- Per-object locking for mutable objects
- Immortal objects (refcount never changes) for frequently accessed singletons
- Modified garbage collector (thread-safe cycle detection)

This adds ~10-30% overhead to single-threaded code but enables true multi-threaded speedup on multi-core systems.

### Syntax
```python
# Build Python 3.13+ with --disable-gil
# ./configure --disable-gil && make

# Check if GIL is enabled
import sys
if hasattr(sys, "_enable_gil"):
    print(f"GIL enabled: {sys.get_ints() and sys._enable_gil}")
else:
    print("Not a free-threaded build")

# Run with GIL disabled:
# python -X gil=0 script.py

# Run with GIL enabled (on free-threaded build):
# python -X gil=1 script.py
```

### Beginner Examples
```python
import sys
import threading
import time

# Check free-threading support
print(f"Python version: {sys.version}")
if hasattr(sys, "_enable_gil"):
    print("Free-threaded build available")
    print(f"GIL currently enabled: {sys._enable_gil}")
else:
    print("Standard CPython build (GIL always enabled)")

# CPU-bound task benefits from free-threading
def compute():
    total = 0
    for i in range(5000000):
        total += i * i
    return total

def run_in_threads(n):
    start = time.time()
    threads = [threading.Thread(target=compute) for _ in range(n)]
    for t in threads: t.start()
    for t in threads: t.join()
    return time.time() - start

# On free-threaded Python, this shows near-linear speedup
# On standard Python, no speedup
print(f"1 thread: {run_in_threads(1):.3f}s")
print(f"4 threads: {run_in_threads(4):.3f}s")
```

### Intermediate Examples
```python
import sys
import threading
import time
from concurrent.futures import ThreadPoolExecutor

# Free-threading: ThreadPoolExecutor now works for CPU-bound tasks
def factorize(n):
    factors = []
    d = 2
    while d * d <= n:
        while n % d == 0:
            factors.append(d)
            n //= d
        d += 1
    if n > 1:
        factors.append(n)
    return factors

def benchmark_executor():
    numbers = [112272535095293, 112582705942171, 115280095190773,
               115797848077099, 1099726899285419] * 2

    # Sequential
    start = time.time()
    seq_results = [factorize(n) for n in numbers]
    seq_time = time.time() - start

    # ThreadPoolExecutor
    start = time.time()
    with ThreadPoolExecutor(max_workers=4) as ex:
        par_results = list(ex.map(factorize, numbers))
    par_time = time.time() - start

    if hasattr(sys, "_enable_gil") and not sys._enable_gil:
        print(f"FREE-THREADING: Speedup = {seq_time / par_time:.2f}x")
    else:
        print(f"GIL ENABLED: Speedup = {seq_time / par_time:.2f}x")

# Thread safety without GIL
# In free-threaded Python, mutable objects need synchronization

lock = threading.Lock()
shared_counter = 0
iterations = 100000

def safe_increment():
    global shared_counter
    for _ in range(iterations):
        # Still need lock! Free-threading doesn't make Python atomic
        with lock:
            shared_counter += 1
```

### Advanced Examples
```python
import sys
import threading
import time
import gc

# Biased reference counting demonstration
# In free-threaded Python, objects use biased refcounting
# Most refcount operations are on the owning thread (fast)
# Cross-thread access requires merging (slow)

# Thread-local vs shared objects
def thread_local_vs_shared():
    n = 500000
    results = {}

    # Thread-local object (no contention)
    def local_work():
        lst = []
        for i in range(n):
            lst.append(i)
        return len(lst)

    start = time.time()
    threads = [threading.Thread(target=local_work) for _ in range(4)]
    for t in threads: t.start()
    for t in threads: t.join()
    results["local"] = time.time() - start

    # Shared object (contention on refcount)
    shared = {}
    lock = threading.Lock()

    def shared_work():
        for i in range(n // 4):
            with lock:
                shared[i] = i

    start = time.time()
    threads = [threading.Thread(target=shared_work) for _ in range(4)]
    for t in threads: t.start()
    for t in threads: t.join()
    results["shared"] = time.time() - start

    print(f"Thread-local: {results['local']:.3f}s")
    print(f"Shared (locked): {results['shared']:.3f}s")

# Immortal objects (small integers, None, True, False)
# In free-threading, these are "immortal" -- refcount never changes
# This avoids refcount contention on built-in singletons

# GC in free-threading mode
def gc_comparison():
    # Free-threading requires a modified GC for thread safety
    print(f"GC enabled: {gc.isenabled()}")
    print(f"GC thresholds: {gc.get_threshold()}")

# Migration considerations
def migration_notes():
    print("""
    Migrating to free-threading:
    - Existing threading code using Lock is still correct
    - No need to add locks where there were none (still unsafe!)
    - Atomic operations are NOT guaranteed (use Lock)
    - C extensions must be updated for thread safety
    - Single-threaded code may be 10-30% slower
    - Benefits CPU-bound multi-threaded code significantly
    """)
```

### Real-World Use Cases
- **Data science** — parallel DataFrame operations on multiple columns
- **Web servers** — handling CPU-heavy requests in thread pool (not just I/O)
- **Scientific computing** — parallel algorithm execution in pure Python
- **Game engines** — CPU-bound game logic in threads
- **CI/CD** — running tests in threads with true parallelism

### Current Limitations (Python 3.13 experimental)
- 10-30% single-threaded performance regression
- Not all C extensions are thread-safe yet
- Requires recompilation with --disable-gil
- Some standard library modules may have thread-safety issues
- Not suitable for production use (experimental)
- Garbage collector has additional overhead
- Debugging free-threaded code is harder (races become visible)

### Common Mistakes
- Assuming free-threading eliminates need for locks
- Expecting linear speedup (Amdahl's law still applies)
- Running free-threaded Python for single-threaded code (wastes ~20% perf)
- Not testing with actual multi-threaded workloads
- Expecting all C extensions to work without changes

### Best Practices
- Test with both gil=0 and gil=1 to compare performance
- Keep existing locks in thread-safe code (still needed)
- Profile to measure actual speedup vs overhead
- Consider numpy/numba for numerical work (already parallel)
- Use standard Python for single-threaded apps (no benefit from free-threading)
- For new projects targeting free-threading, design with thread safety from start
- Contribute to C extension compatibility efforts
- Wait for Python 3.14+ for production-ready free-threading

### Performance Considerations
- Single-threaded regression: ~10-30% for pure Python, less if using C extensions
- Multi-threaded speedup: up to Nx on N cores for embarassingly parallel workloads
- Biased refcounting: most ops are fast (same thread), cross-thread ops slower
- Immortal objects: no refcount contention on singletons
- C extensions: need explicit thread safety (Py_GIL_DISABLED support)
- Memory usage: slightly higher due to biased refcount fields

### Interview Questions
1. What is PEP 703 and what does it propose?
2. How does free-threading affect single-threaded performance?
3. What is biased reference counting?
4. Do you still need locks in free-threaded Python?
5. What are immortal objects?
6. Is free-threading ready for production in Python 3.13?

### Coding Challenges
- Write a script that runs the same computation with threads and checks speedup
- Compare performance of Python built with and without GIL
- Implement a test that verifies thread safety under free-threading
- Benchmark the overhead of free-threading mode for a single-threaded workload

### Related Topics
- PEP 703, GIL, threading, multiprocessing, C extension thread safety, CPython internals
