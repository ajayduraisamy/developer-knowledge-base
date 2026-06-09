# Multiprocessing - multiprocessing.Process, Pool, shared memory
## Introduction
The `multiprocessing` module allows running multiple processes, each with its own Python interpreter and memory space. It sidesteps the GIL limitation, making it suitable for CPU-bound parallel computation. The module provides Process, Pool, Queue, Pipe, and shared memory abstractions.

## multiprocessing.Process
### What It Is
`multiprocessing.Process` represents a separate OS process with its own Python interpreter, GIL, and memory space. It follows an API similar to `threading.Thread` but forks (or spawns) a completely new process.

### Why It Is Important
Unlike threads, processes run in parallel across CPU cores without GIL contention. They provide true parallelism for CPU-intensive tasks, fault isolation (one process crash doesn't affect others), and memory isolation (no accidental shared state).

### How It Works Internally
On POSIX systems, `Process` uses `os.fork()` by default (start method `fork`), creating a copy of the parent process. On Windows (and with `spawn` method), it starts a new Python interpreter by executing `__main__` or a specified module. The child process gets a fresh GIL, its own memory space, and a copy of the parent's data (though shared memory can be explicitly created).

### Syntax
```python
import multiprocessing

p = multiprocessing.Process(target=func, args=(arg,))
p.start()
p.join(timeout=5)
p.terminate()     # force kill
p.is_alive()
p.exitcode        # exit code (0 = success)
p.pid             # process ID
p.daemon          # daemon flag

# Start methods
multiprocessing.set_start_method("fork")   # POSIX only
multiprocessing.set_start_method("spawn")  # Windows default
multiprocessing.set_start_method("forkserver")  # POSIX
```

### Beginner Examples
```python
import multiprocessing
import os

def worker(name):
    print(f"Worker {name} running in process {os.getpid()}")

if __name__ == "__main__":
    processes = []
    for i in range(4):
        p = multiprocessing.Process(target=worker, args=(i,))
        processes.append(p)
        p.start()
    
    for p in processes:
        p.join()
    
    print(f"Main process: {os.getpid()}")
```

### Intermediate Examples
```python
import multiprocessing
import os
import time

class WorkerProcess(multiprocessing.Process):
    def __init__(self, task_id, duration):
        super().__init__()
        self.task_id = task_id
        self.duration = duration
    
    def run(self):
        print(f"Process {self.task_id} (PID: {os.getpid()}) working for {self.duration}s")
        time.sleep(self.duration)
        print(f"Process {self.task_id} done")
        return self.task_id * 2

if __name__ == "__main__":
    # Start all processes
    procs = [WorkerProcess(i, 2 - i * 0.3) for i in range(4)]
    for p in procs:
        p.start()
    
    # Monitor
    for p in procs:
        p.join(timeout=3)
        if p.is_alive():
            print(f"Terminating {p.pid}")
            p.terminate()
        
        print(f"Process {p.pid} exit code: {p.exitcode}")

# Process naming and identification
def show_info():
    current = multiprocessing.current_process()
    print(f"Name: {current.name}, PID: {current.pid}, Daemon: {current.daemon}")

if __name__ == "__main__":
    p = multiprocessing.Process(target=show_info, name="MyWorker")
    p.start()
    p.join()
```

### Advanced Examples
```python
import multiprocessing
import os
import signal
import time

# Graceful shutdown with Event
stop_event = multiprocessing.Event()

def worker_with_shutdown():
    while not stop_event.is_set():
        print(f"Working (PID: {os.getpid()})")
        stop_event.wait(1)
    print(f"Shutting down {os.getpid()}")

if __name__ == "__main__":
    procs = [multiprocessing.Process(target=worker_with_shutdown) for _ in range(3)]
    for p in procs: p.start()
    time.sleep(3)
    stop_event.set()
    for p in procs: p.join(timeout=2)
    
# Custom start method
def check_and_set_start():
    import multiprocessing.context
    ctx = multiprocessing.get_context("spawn")
    p = ctx.Process(target=lambda: print("Spawned!"))
    p.start()
    p.join()

if __name__ == "__main__":
    print(f"Start method: {multiprocessing.get_start_method()}")
    check_and_set_start()

# Process identity and inheritance
import multiprocessing.shared_memory

class ComputationProcess(multiprocessing.Process):
    def __init__(self, data_slice, result_queue):
        super().__init__()
        self.data_slice = data_slice
        self.result_queue = result_queue
    
    def run(self):
        # Heavy computation
        result = sum(x * x for x in self.data_slice)
        self.result_queue.put(result)

if __name__ == "__main__":
    large_data = list(range(1000000))
    chunk_size = len(large_data) // 4
    q = multiprocessing.Queue()
    
    procs = [
        ComputationProcess(large_data[i:i+chunk_size], q)
        for i in range(0, len(large_data), chunk_size)
    ]
    
    for p in procs: p.start()
    for p in procs: p.join()
    
    total = sum(q.get() for _ in procs)
    print(f"Total: {total}")
```

### Real-World Use Cases
- **Parallel data processing** — split large datasets across CPU cores
- **Web scraping at scale** — each process handles independent URL batches
- **Machine learning** — hyperparameter tuning, model training in parallel
- **Image/video processing** — frame-by-frame parallel processing
- **Scientific computing** — Monte Carlo simulations, numerical integration

### Common Mistakes
- Forgetting the `if __name__ == "__main__":` guard (causes infinite spawning on Windows)
- Passing unpicklable arguments to Process (all args must be picklable)
- Starting too many processes (OS resource limit, thrashing)
- Not handling zombie processes (failing to `join()`)
- Assuming shared state is automatically synchronized (each process has its own copy)

### Best Practices
- Always use `if __name__ == "__main__":` guard for cross-platform compatibility
- Use `Process` for independent tasks; use `Pool` for data parallelism
- Terminate processes that hang or take too long
- Use `multiprocessing.Queue` or `Pipe` for inter-process communication
- Prefer `join()` with timeout to prevent indefinite blocking
- Limit the number of concurrent processes to `os.cpu_count()`

### Performance Considerations
- Process creation is expensive (spawn method copies interpreter state)
- `fork` start method is faster but unsafe in threaded programs
- IPC (inter-process communication) adds overhead for data transfer
- Context switching between processes is more expensive than between threads
- Memory usage scales with number of processes

### Interview Questions
1. What is the difference between Process and Thread?
2. What are the three start methods in multiprocessing?
3. Why must `if __name__ == "__main__":` be used?
4. How do processes communicate with each other?
5. What is a zombie process and how do you avoid it?

### Coding Challenges
- Write a program that computes the sum of squares for a large array using 4 processes
- Build a parallel file compressor using multiprocessing.Process
- Create a process pool from scratch that manages worker processes

### Related Topics
- multiprocessing.Pool, shared memory, Queue, threading, concurrent.futures

## Pool
### What It Is
`multiprocessing.Pool` provides a convenient way to parallelize function execution across multiple input values, distributing the work across a pool of worker processes. It supports `map`, `apply`, `starmap`, and `imap` operations.

### Why It Is Important
Pool abstracts away the complexity of manually managing process creation, task distribution, and result collection. It is the go-to tool for embarrassingly parallel problems — applying the same function to many independent data items.

### How It Works Internally
Pool maintains a fixed number of worker processes. When `map()` is called, the input iterable is chunked and distributed to worker processes via an internal Queue. Each worker picks up chunks, processes items, and sends results back through a result queue. The `Pool` uses `multiprocessing.Queue` internally for task distribution and result collection.

### Syntax
```python
from multiprocessing import Pool

with Pool(processes=4) as pool:
    results = pool.map(func, iterable)
    results = pool.starmap(func, iterable_of_tuples)
    results = pool.map_async(func, iterable).get()
    result = pool.apply(func, args)
    result = pool.apply_async(func, args).get()
    
    # Lazy iteration
    for res in pool.imap(func, iterable, chunksize=10):
        pass
```

### Beginner Examples
```python
import multiprocessing as mp
import time

def square(n):
    return n * n

if __name__ == "__main__":
    numbers = list(range(20))
    
    # Create a pool with 4 workers
    with mp.Pool(processes=4) as pool:
        results = pool.map(square, numbers)
    
    print(f"Squares: {results}")

# Parallel sleep (demonstrates parallelism)
def slow_square(n):
    time.sleep(1)
    return n * n

if __name__ == "__main__":
    start = time.time()
    with mp.Pool(processes=4) as pool:
        results = pool.map(slow_square, range(8))
    elapsed = time.time() - start
    print(f"8 tasks with 4 workers: {elapsed:.2f}s (would be ~8s sequentially)")
```

### Intermediate Examples
```python
import multiprocessing as mp

# Starmap for multiple arguments
def power(base, exp):
    return base ** exp

if __name__ == "__main__":
    args = [(2, 3), (3, 4), (4, 5), (5, 6)]
    with mp.Pool(processes=2) as pool:
        results = pool.starmap(power, args)
    print(f"Powers: {results}")

# Apply with single tasks
def process_data(data, config=None):
    return {"data": data, "config": config, "result": data * 2}

if __name__ == "__main__":
    with mp.Pool(processes=2) as pool:
        result1 = pool.apply(process_data, args=(10,), kwds={"config": "fast"})
        result2 = pool.apply_async(process_data, args=(20,), kwds={"config": "slow"})
        print(f"Direct: {result1}")
        print(f"Async: {result2.get()}")

# Map with chunksize control
import math

def is_prime(n):
    if n < 2: return False
    for i in range(2, int(math.sqrt(n)) + 1):
        if n % i == 0: return False
    return True

if __name__ == "__main__":
    numbers = range(10**7, 10**7 + 1000)
    with mp.Pool(processes=4) as pool:
        # Larger chunksize reduces IPC overhead
        result = any(pool.imap(is_prime, numbers, chunksize=100))
    print(f"Found prime: {result}")

# imap (lazy) vs map (eager)
if __name__ == "__main__":
    with mp.Pool(processes=2) as pool:
        # imap yields results as they're ready
        for result in pool.imap_unordered(slow_square, range(10)):
            print(f"Got: {result}")  # Results in arbitrary order
```

### Advanced Examples
```python
import multiprocessing as mp
import time
import random
from functools import partial

# Partial function application with Pool
def process_row(delimiter, row):
    fields = row.strip().split(delimiter)
    return [len(f) for f in fields]

if __name__ == "__main__":
    data_rows = ["a,b,c", "hello,world", "python,multiprocessing,pool"]
    
    process_csv = partial(process_row, ",")
    
    with mp.Pool(processes=2) as pool:
        results = pool.map(process_csv, data_rows)
    print(f"CSV stats: {results}")

# Progress bar with imap
def simulate_work(n):
    time.sleep(random.uniform(0.1, 0.5))
    return n * n

if __name__ == "__main__":
    total = 50
    with mp.Pool(processes=4) as pool:
        for i, result in enumerate(pool.imap_unordered(simulate_work, range(total))):
            # Simple progress indicator
            if (i + 1) % 10 == 0:
                print(f"Progress: {i + 1}/{total}")

# Nested pools (one pool per task)
def worker_group(group_id, items):
    with mp.Pool(processes=2) as inner_pool:
        return inner_pool.map(simulate_work, items)

if __name__ == "__main__":
    groups = [(i, range(i * 10, (i + 1) * 10)) for i in range(4)]
    with mp.Pool(processes=4) as outer_pool:
        results = outer_pool.starmap(worker_group, groups)
    print(f"Group results: {results}")

# Worker initialization and state
def init_worker(data_ref):
    global shared_data
    shared_data = data_ref

def worker_func(x):
    return x * shared_data["multiplier"]

if __name__ == "__main__":
    config = {"multiplier": 10}
    # initializer is called once per worker at startup
    with mp.Pool(processes=2, initializer=init_worker, initargs=(config,)) as pool:
        results = pool.map(worker_func, [1, 2, 3, 4])
    print(f"With initializer: {results}")
```

### Real-World Use Cases
- **Data processing pipelines** — CSV parsing, image processing, text analysis
- **Web scraping** — parallel page fetching and parsing
- **Scientific computing** — parameter sweeps, Monte Carlo simulations
- **ETL jobs** — transform and load large datasets in parallel
- **Machine learning** — cross-validation folds, grid search hyperparameters

### Common Mistakes
- Using `map()` with a generator (it's consumed eagerly but needs a sequence)
- Not calling `.get()` on `apply_async()` results (errors are silently swallowed)
- Using `Pool` inside a class method without proper pickling
- Creating too many workers (oversubscription)
- Passing large data through `map()` repeatedly (expensive serialization)

### Best Practices
- Use `with Pool() as pool:` context manager for automatic cleanup
- Use `chunksize=...` to control granularity — larger for fast functions
- Use `imap_unordered` for the best performance when order doesn't matter
- Use `apply_async` for heterogeneous tasks
- Call `.get()` on async results with timeout to prevent hangs
- Use `initializer` to set up per-worker state
- Match `processes` to `os.cpu_count()` for CPU-bound tasks

### Performance Considerations
- `map()` chunks the iterable and distributes chunks — large chunks reduce IPC
- `imap` and `imap_unordered` are more memory-efficient than `map`
- Serialization/pickling of arguments and results adds overhead
- Pool workers inherit the parent process's memory (with `fork`)
- Creating and destroying pools is expensive — reuse if possible

### Interview Questions
1. What is the difference between `map()` and `imap()`?
2. How does `chunksize` affect performance?
3. What is the advantage of `imap_unordered` over `imap`?
4. How do you pass initial state to Pool workers?
5. What happens when you call `apply_async().get()`?

### Coding Challenges
- Write a parallel word count using Pool.map on a large text file
- Build a parallel image thumbnail generator using starmap
- Implement a simple map-reduce framework using Pool

### Related Topics
- concurrent.futures.ProcessPoolExecutor, multiprocessing.Process, dask, ray

## Shared memory
### What It Is
The `multiprocessing.shared_memory` module (Python 3.8+) allows multiple processes to access the same region of memory, avoiding the overhead of serialization and IPC for large data. It includes `SharedMemory` for raw memory access and `ShareableList` for typed arrays.

### Why It Is Important
Sharing memory directly eliminates the serialization bottleneck when passing large numpy arrays, images, or dataframes between processes. It enables true zero-copy parallelism for data-intensive workloads.

### How It Works Internally
`SharedMemory` allocates a POSIX shared memory object (`/dev/shm` on Linux) or a Windows named shared memory section. Each process maps this memory into its address space via `mmap` (POSIX) or `MapViewOfFile` (Windows). The memory is reference-counted — when all handles are closed, the memory is deallocated.

### Syntax
```python
from multiprocessing import shared_memory

# Create
shm = shared_memory.SharedMemory(name="myshm", create=True, size=1024)

# Attach to existing
shm = shared_memory.SharedMemory(name="myshm")

# Read/write via buffer protocol
shm.buf[0:4] = bytes([0, 1, 2, 3])
data = bytes(shm.buf[:10])

# ShareableList
slist = shared_memory.ShareableList(["hello", 42, 3.14])

# Cleanup
shm.close()
shm.unlink()  # remove the shared memory
```

### Beginner Examples
```python
from multiprocessing import shared_memory, Process

def modify_buffer(name):
    shm = shared_memory.SharedMemory(name=name)
    shm.buf[0:5] = b"WORLD"
    print(f"Child read: {bytes(shm.buf[:11])}")
    shm.close()

if __name__ == "__main__":
    # Create 11 bytes of shared memory (for "HELLO WORLD")
    shm = shared_memory.SharedMemory(create=True, size=11)
    shm.buf[:5] = b"HELLO"
    shm.buf[6:11] = b"WORLD"
    
    print(f"Before: {bytes(shm.buf[:11])}")
    
    p = Process(target=modify_buffer, args=(shm.name,))
    p.start()
    p.join()
    
    print(f"After: {bytes(shm.buf[:11])}")
    shm.close()
    shm.unlink()
```

### Intermediate Examples
```python
from multiprocessing import shared_memory, Process
import array

# Share numeric arrays via shared memory
def worker(shm_name, size):
    shm = shared_memory.SharedMemory(name=shm_name)
    arr = array.array("i", [0]) * size
    arr.buffer_info()  # Use buffer protocol
    # Read from shared memory into array
    for i in range(size):
        # Read 4 bytes per int
        val = int.from_bytes(shm.buf[i*4:(i+1)*4], "little")
        shm.buf[i*4:(i+1)*4] = (val * 2).to_bytes(4, "little")
    shm.close()

if __name__ == "__main__":
    size = 1000
    shm = shared_memory.SharedMemory(create=True, size=size * 4)
    
    # Initialize with values 0..999
    for i in range(size):
        shm.buf[i*4:(i+1)*4] = i.to_bytes(4, "little")
    
    p = Process(target=worker, args=(shm.name, size))
    p.start()
    p.join()
    
    # Read results
    results = [int.from_bytes(shm.buf[i*4:(i+1)*4], "little") for i in range(size)]
    print(f"First 10 values: {results[:10]}")  # Each value doubled
    
    shm.close()
    shm.unlink()
```

### Advanced Examples
```python
from multiprocessing import shared_memory, Process
import numpy as np

# Zero-copy numpy array sharing
def process_numpy(shm_name, shape, dtype):
    shm = shared_memory.SharedMemory(name=shm_name)
    arr = np.ndarray(shape, dtype=dtype, buffer=shm.buf)
    
    # Modify in-place (zero-copy)
    arr[:] = arr * 2
    
    shm.close()

if __name__ == "__main__":
    # Create a large numpy array
    shape = (1000, 1000)
    original = np.ones(shape, dtype=np.float64)
    nbytes = original.nbytes
    
    # Create shared memory and map numpy array to it
    shm = shared_memory.SharedMemory(create=True, size=nbytes)
    shared_arr = np.ndarray(shape, dtype=np.float64, buffer=shm.buf)
    shared_arr[:] = original  # Copy data into shared memory
    
    # Process modifies in-place
    p = Process(target=process_numpy, args=(shm.name, shape, np.float64))
    p.start()
    p.join()
    
    print(f"First element: {shared_arr[0, 0]}")  # 2.0 (doubled)
    
    shm.close()
    shm.unlink()

# ShareableList example
def modify_shareable(shm_name):
    slist = shared_memory.ShareableList(name=shm_name)
    slist[0] = slist[0].upper()
    slist[1] = slist[1] * 2
    slist.shm.close()

if __name__ == "__main__":
    slist = shared_memory.ShareableList(
        ["hello", 42, 3.14, "world"],
        name="mylist"
    )
    
    p = Process(target=modify_shareable, args=(slist.shm.name,))
    p.start()
    p.join()
    
    print(f"After: list(slist) = {list(slist)}")
    
    slist.shm.close()
    slist.shm.unlink()

# Race condition demonstration (need locking with shared memory)
from multiprocessing import Lock

def unsafe_increment(shm_name, lock):
    shm = shared_memory.SharedMemory(name=shm_name)
    for _ in range(1000):
        with lock:
            val = int.from_bytes(shm.buf[0:4], "little")
            val += 1
            shm.buf[0:4] = val.to_bytes(4, "little")
    shm.close()

if __name__ == "__main__":
    shm = shared_memory.SharedMemory(create=True, size=4)
    shm.buf[0:4] = (0).to_bytes(4, "little")
    lock = Lock()
    
    procs = [Process(target=unsafe_increment, args=(shm.name, lock)) for _ in range(4)]
    for p in procs: p.start()
    for p in procs: p.join()
    
    result = int.from_bytes(shm.buf[0:4], "little")
    print(f"Counter: {result} (expected 4000)")
    
    shm.close()
    shm.unlink()
```

### Real-World Use Cases
- **Data science** — sharing large DataFrames/numpy arrays between processes
- **Video processing** — raw frame buffer shared across encoding processes
- **Real-time systems** — sensor data shared between acquisition and processing
- **Web servers** — shared cache among worker processes
- **Game engines** — shared world state across process boundaries

### Common Mistakes
- Forgetting to `unlink()` shared memory (causes resource leak)
- Not using synchronization (locks) with shared mutable data
- Using `SharedMemory` with incompatible Python versions (< 3.8)
- Assuming `ShareableList` supports arbitrary types (limited to basic types)
- Creating shared memory without checking if it already exists

### Best Practices
- Always pair `create=True` with an eventual `unlink()` call
- Use `try/finally` to ensure cleanup on exceptions
- Use Lock or other synchronization for mutable shared data
- Prefer `ShareableList` for simple shared data structures
- Use numpy's buffer interface for zero-copy array sharing
- Name shared memory segments meaningfully for debugging
- Close shared memory in all processes after use

### Performance Considerations
- Shared memory is significantly faster than Queue/Pipe for large data (microseconds vs milliseconds)
- No serialization overhead — zero-copy access
- Memory is allocated once and mapped into each process
- Synchronization (Lock) adds overhead when coordinating writes
- `ShareableList` uses more memory than raw `SharedMemory` (metadata overhead)

### Interview Questions
1. How does `multiprocessing.shared_memory` differ from using `Queue`?
2. What happens if you don't call `unlink()` on a `SharedMemory` object?
3. How can numpy arrays be shared between processes with shared memory?
4. What types does `ShareableList` support?
5. What synchronization issues arise with shared memory?

### Coding Challenges
- Build a shared counter that multiple processes increment safely
- Implement a producer-consumer with shared memory ring buffer
- Create a parallel image processing pipeline sharing image buffers via shared memory
- Write a benchmark comparing Queue vs shared memory for large data transfer

### Related Topics
- multiprocessing.Queue, multiprocessing.Pipe, mmap, numpy, multiprocessing.Manager
