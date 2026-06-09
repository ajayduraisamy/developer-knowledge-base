# Multiprocessing - multiprocessing.Process, Pool, shared memory

## Introduction

The `multiprocessing` module allows Python programs to spawn multiple processes, bypassing the Global Interpreter Lock (GIL) to achieve true parallelism. Each process runs its own Python interpreter with its own memory space, making it suitable for CPU-bound tasks.

## Why It Is Important

Multiprocessing enables Python to fully utilize multi-core CPUs for computationally intensive tasks. Unlike threading, it sidesteps the GIL limitation, providing true parallel execution. It also offers process isolation, making it less susceptible to memory corruption and allowing separate GILs for each worker.

## Syntax

```python
import multiprocessing as mp

# Process
p = mp.Process(target=function, args=(arg,))
p.start()
p.join()

# Pool
with mp.Pool(processes=4) as pool:
    pool.map(func, iterable)
    pool.apply(func, args=(arg,))
    result = pool.apply_async(func, args=(arg,))
    results = pool.starmap(func, iterable_of_tuples)

# Queue
q = mp.Queue()
q.put(item)
item = q.get()

# Pipe
conn1, conn2 = mp.Pipe()
conn1.send(data)
data = conn2.recv()

# Shared memory
shared_value = mp.Value('i', 0)
shared_array = mp.Array('d', [0.0] * 10)

# Manager
manager = mp.Manager()
shared_dict = manager.dict()
shared_list = manager.list()
```

## Examples

### Basic Process Creation

```python
import multiprocessing as mp
import os
import time

def worker(name):
    pid = os.getpid()
    print(f"Worker {name} (PID: {pid}) starting")
    time.sleep(1)
    print(f"Worker {name} finishing")

if __name__ == "__main__":
    processes = []
    for i in range(4):
        p = mp.Process(target=worker, args=(f"W-{i}",))
        processes.append(p)
        p.start()
        print(f"Started process {i}")

    for p in processes:
        p.join()

    print("All processes completed")
```

### Process with Return Values (Queue)

```python
import multiprocessing as mp
import time
import random

def compute_square(numbers, queue):
    results = []
    for n in numbers:
        time.sleep(random.uniform(0.1, 0.3))
        results.append(n * n)
    queue.put(results)

if __name__ == "__main__":
    data_chunks = [
        [1, 2, 3],
        [4, 5, 6],
        [7, 8, 9],
    ]

    queue = mp.Queue()
    processes = []
    for i, chunk in enumerate(data_chunks):
        p = mp.Process(target=compute_square, args=(chunk, queue))
        processes.append(p)
        p.start()

    all_results = []
    for _ in processes:
        all_results.extend(queue.get())

    for p in processes:
        p.join()

    print(f"Results: {sorted(all_results)}")
```

### Pool.map and Pool.starmap

```python
import multiprocessing as mp
import time

def square(n):
    time.sleep(0.5)
    return n * n

def multiply(a, b):
    time.sleep(0.3)
    return a * b

if __name__ == "__main__":
    with mp.Pool(processes=4) as pool:
        numbers = list(range(10))
        squares = pool.map(square, numbers)
        print(f"Map squares: {squares}")

    with mp.Pool(processes=4) as pool:
        pairs = [(i, j) for i in range(1, 4) for j in range(1, 4)]
        products = pool.starmap(multiply, pairs)
        print(f"Starmap products: {products}")

    with mp.Pool(processes=2) as pool:
        results = []
        for number in [5, 10, 15, 20]:
            result = pool.apply_async(square, (number,))
            results.append(result)

        squares = [r.get() for r in results]
        print(f"Apply async squares: {squares}")
```

### Pool.imap and imap_unordered

```python
import multiprocessing as mp
import time
import random

def slow_square(n):
    time.sleep(random.uniform(0.1, 0.5))
    return n * n

if __name__ == "__main__":
    numbers = list(range(10))

    with mp.Pool(4) as pool:
        print("imap (ordered):")
        for result in pool.imap(slow_square, numbers):
            print(f"  {result}", end=" ")
        print()

    with mp.Pool(4) as pool:
        print("\nimap_unordered (completion order):")
        for result in pool.imap_unordered(slow_square, numbers):
            print(f"  {result}", end=" ")
        print()
```

### Queue for Inter-Process Communication

```python
import multiprocessing as mp
import time
import random

def producer(queue, id, num_items):
    for i in range(num_items):
        item = f"P{id}-item-{i}"
        time.sleep(random.uniform(0.1, 0.3))
        queue.put(item)
        print(f"Producer {id}: produced {item}")
    queue.put(None)

def consumer(queue, id):
    while True:
        item = queue.get()
        if item is None:
            queue.put(None)
            break
        print(f"Consumer {id}: consumed {item}")
        time.sleep(random.uniform(0.2, 0.4))

if __name__ == "__main__":
    queue = mp.Queue(maxsize=5)

    producers = [
        mp.Process(target=producer, args=(queue, i, 5))
        for i in range(2)
    ]

    consumers = [
        mp.Process(target=consumer, args=(queue, i))
        for i in range(3)
    ]

    for c in consumers:
        c.start()

    for p in producers:
        p.start()

    for p in producers:
        p.join()

    for c in consumers:
        c.join()

    print("Queue IPC example completed")
```

### Pipe for Two-Way Communication

```python
import multiprocessing as mp
import time

def worker(conn, name):
    print(f"{name}: waiting for message...")
    msg = conn.recv()
    print(f"{name}: received '{msg}'")
    time.sleep(0.5)
    conn.send(f"Response from {name}")
    conn.close()

if __name__ == "__main__":
    parent_conn, child_conn = mp.Pipe()

    p = mp.Process(target=worker, args=(child_conn, "Worker"))
    p.start()

    parent_conn.send("Hello from main process!")
    response = parent_conn.recv()
    print(f"Main: received '{response}'")

    p.join()
    parent_conn.close()
    print("Pipe example completed")
```

### Shared Memory (Value and Array)

```python
import multiprocessing as mp
import time

def increment_counter(counter, lock, increments):
    for _ in range(increments):
        with lock:
            counter.value += 1

def process_array(shared_arr, lock, start, end):
    for i in range(start, end):
        with lock:
            shared_arr[i] = shared_arr[i] * 2

if __name__ == "__main__":
    counter = mp.Value("i", 0)
    lock = mp.Lock()

    processes = [
        mp.Process(target=increment_counter, args=(counter, lock, 10000))
        for _ in range(4)
    ]

    for p in processes:
        p.start()

    for p in processes:
        p.join()

    print(f"Counter value: {counter.value} (expected: {40000})")

    shared_arr = mp.Array("d", range(10))
    processes = [
        mp.Process(target=process_array, args=(shared_arr, lock, 0, 5)),
        mp.Process(target=process_array, args=(shared_arr, lock, 5, 10)),
    ]

    for p in processes:
        p.start()

    for p in processes:
        p.join()

    print(f"Doubled array: {list(shared_arr)}")
```

### Manager for Shared Data Structures

```python
import multiprocessing as mp
import time

def worker(shared_dict, shared_list, name):
    shared_dict[name] = mp.current_process().pid
    shared_list.append(name)
    time.sleep(0.5)

if __name__ == "__main__":
    with mp.Manager() as manager:
        shared_dict = manager.dict()
        shared_list = manager.list()

        processes = [
            mp.Process(target=worker, args=(shared_dict, shared_list, f"W-{i}"))
            for i in range(4)
        ]

        for p in processes:
            p.start()

        for p in processes:
            p.join()

        print(f"Shared dict: {dict(shared_dict)}")
        print(f"Shared list: {list(shared_list)}")
```

### Pool with Context Manager

```python
import multiprocessing as mp
import time

def cpu_intensive(n):
    total = 0
    for i in range(n):
        total += i ** 2
    return total

if __name__ == "__main__":
    tasks = [10_000_000, 20_000_000, 30_000_000, 15_000_000]

    start = time.time()
    with mp.Pool(processes=4) as pool:
        results = pool.map(cpu_intensive, tasks)

    elapsed = time.time() - start
    print(f"Results: {results}")
    print(f"Parallel execution time: {elapsed:.3f}s")

    start = time.time()
    serial_results = [cpu_intensive(t) for t in tasks]
    serial_elapsed = time.time() - start
    print(f"Serial execution time: {serial_elapsed:.3f}s")
    print(f"Speedup: {serial_elapsed / elapsed:.2f}x")
```

## Beginner Examples

```python
# Parallel file hashing
import multiprocessing as mp
import hashlib
from pathlib import Path

def hash_file(filepath):
    path = Path(filepath)
    if not path.is_file():
        return (filepath, None, "NOT FOUND")
    content = path.read_bytes()
    md5 = hashlib.md5(content).hexdigest()
    sha256 = hashlib.sha256(content).hexdigest()
    return (filepath, md5, sha256)

if __name__ == "__main__":
    import sys
    files = [str(p) for p in Path(".").glob("*.py")[:8]]

    if not files:
        print("No Python files found for hashing demo")
    else:
        with mp.Pool(processes=4) as pool:
            results = pool.map(hash_file, files)

        for filepath, md5, sha256 in results:
            if md5:
                print(f"{filepath}: MD5={md5[:16]}... SHA256={sha256[:16]}...")
            else:
                print(f"{filepath}: {sha256}")

# Simple worker process pool from scratch
import multiprocessing as mp
import time
from queue import Empty

class SimpleProcessPool:
    def __init__(self, num_workers):
        self.tasks = mp.Queue()
        self.results = mp.Queue()
        self.workers = []

        for i in range(num_workers):
            p = mp.Process(target=self._worker_loop, args=(self.tasks, self.results))
            p.start()
            self.workers.append(p)

    @staticmethod
    def _worker_loop(tasks, results):
        while True:
            try:
                task = tasks.get(timeout=1)
                if task is None:
                    break
                func, args, kwargs = task
                try:
                    result = func(*args, **kwargs)
                    results.put((True, result))
                except Exception as e:
                    results.put((False, str(e)))
            except Exception:
                break

    def submit(self, func, *args, **kwargs):
        self.tasks.put((func, args, kwargs))

    def get_results(self, timeout=None):
        collected = []
        start = time.time()
        while len(collected) < len(self.workers):
            try:
                success, value = self.results.get(timeout=0.1)
                collected.append(value if success else None)
            except Exception:
                if timeout and time.time() - start > timeout:
                    break
        return collected

    def shutdown(self):
        for _ in self.workers:
            self.tasks.put(None)
        for w in self.workers:
            w.join()
```

## Intermediate Examples

```python
# Parallel data processing pipeline
import multiprocessing as mp
import time
import random

def stage1_reader(data_chunk):
    """Simulate reading and parsing raw data."""
    time.sleep(random.uniform(0.1, 0.2))
    return [x * 10 for x in data_chunk]

def stage2_processor(data):
    """Simulate CPU-intensive transformation."""
    time.sleep(random.uniform(0.2, 0.4))
    return [x ** 2 for x in data]

def stage3_writer(data):
    """Simulate writing results."""
    time.sleep(random.uniform(0.1, 0.2))
    return sum(data)

if __name__ == "__main__":
    data = [list(range(i * 10, (i + 1) * 10)) for i in range(8)]

    start = time.time()

    with mp.Pool(4) as pool1, mp.Pool(4) as pool2, mp.Pool(2) as pool3:
        stage1_results = pool1.map(stage1_reader, data)
        stage2_results = pool2.map(stage2_processor, stage1_results)
        stage3_results = pool3.map(stage3_writer, stage2_results)

    elapsed = time.time() - start
    print(f"Pipeline results: {stage3_results}")
    print(f"Total: {sum(stage3_results)}")
    print(f"Pipeline completed in {elapsed:.3f}s")

# Parallel matrix multiplication
def multiply_row(args):
    matrix_a, matrix_b_row, row_idx = args
    result_row = []
    n = len(matrix_b_row[0]) if matrix_b_row else 0
    for j in range(n):
        total = 0
        for k in range(len(matrix_a[row_idx])):
            total += matrix_a[row_idx][k] * matrix_b_row[k][j]
        result_row.append(total)
    return result_row

def parallel_matrix_multiply(A, B, num_workers=None):
    if not A or not B:
        return []
    B_transposed = list(zip(*B))
    with mp.Pool(num_workers) as pool:
        args = [(A, B_transposed, i) for i in range(len(A))]
        result = pool.map(multiply_row, args)
    return result

if __name__ == "__main__":
    size = 200
    A = [[1 if i == j else 0 for j in range(size)] for i in range(size)]
    B = [[i * size + j for j in range(size)] for i in range(size)]

    start = time.time()
    C = parallel_matrix_multiply(A, B)
    elapsed = time.time() - start
    print(f"Matrix multiplication ({size}x{size}) completed in {elapsed:.3f}s")
```

## Advanced Examples

```python
# Advanced process manager with dynamic scaling, health checks, and result aggregation
import multiprocessing as mp
import time
import random
import os
import signal
from typing import Callable, List, Any, Optional

class ProcessWorker(mp.Process):
    def __init__(self, task_queue, result_queue, worker_id):
        super().__init__()
        self.task_queue = task_queue
        self.result_queue = result_queue
        self.worker_id = worker_id
        self.daemon = True

    def run(self):
        print(f"Worker {self.worker_id} (PID: {os.getpid()}) started")
        while True:
            try:
                task = self.task_queue.get(timeout=1)
                if task is None:
                    break
                task_id, func, args, kwargs = task
                try:
                    result = func(*args, **kwargs)
                    self.result_queue.put((task_id, True, result))
                except Exception as e:
                    self.result_queue.put((task_id, False, str(e)))
            except Exception:
                break
        print(f"Worker {self.worker_id} shutting down")

class DynamicProcessPool:
    def __init__(self, min_workers=2, max_workers=8, idle_timeout=10):
        self.min_workers = min_workers
        self.max_workers = max_workers
        self.idle_timeout = idle_timeout
        self.task_queue = mp.Queue()
        self.result_queue = mp.Queue()
        self.workers: List[ProcessWorker] = []
        self.next_task_id = 0
        self.pending_tasks = {}
        self._stop = mp.Event()

        self._start_workers(min_workers)

    def _start_workers(self, count):
        for _ in range(count):
            w = ProcessWorker(self.task_queue, self.result_queue, len(self.workers))
            w.start()
            self.workers.append(w)

    def submit(self, func: Callable, *args, **kwargs) -> int:
        task_id = self.next_task_id
        self.next_task_id += 1
        self.pending_tasks[task_id] = (func, args, kwargs)
        self.task_queue.put((task_id, func, args, kwargs))

        if len(self.pending_tasks) > len(self.workers) * 2 and len(self.workers) < self.max_workers:
            self._start_workers(1)
            print(f"Scaled up to {len(self.workers)} workers")

        return task_id

    def get_results(self, timeout: Optional[float] = None) -> List[Any]:
        results = []
        collected = set()
        deadline = time.time() + timeout if timeout else None

        while len(collected) < len(self.pending_tasks):
            try:
                remaining = (deadline - time.time()) if deadline else 1
                task_id, success, value = self.result_queue.get(timeout=min(remaining, 1))
                if task_id not in collected:
                    collected.add(task_id)
                    if success:
                        results.append(value)
                    else:
                        print(f"Task {task_id} failed: {value}")
            except Exception:
                if deadline and time.time() > deadline:
                    print(f"Timeout waiting for {len(self.pending_tasks) - len(collected)} tasks")
                    break

        return results

    def shutdown(self):
        for _ in self.workers:
            self.task_queue.put(None)
        for w in self.workers:
            w.join(timeout=5)
        print(f"Pool shut down. {len(self.workers)} workers terminated.")

def heavy_compute(x):
    time.sleep(random.uniform(0.5, 1.5))
    return x * x * x

pool = DynamicProcessPool(min_workers=2, max_workers=6)

task_ids = []
for i in range(20):
    tid = pool.submit(heavy_compute, i)
    task_ids.append(tid)

results = pool.get_results(timeout=30)
print(f"Collected {len(results)} results: {sorted(results)[:5]}...{sorted(results)[-5:]}")

pool.shutdown()

# Parallel genetic algorithm skeleton
import multiprocessing as mp
import random

def evaluate_fitness(individual):
    """Evaluate fitness function for a single individual."""
    return sum(gene ** 2 for gene in individual)

def mutate(individual, mutation_rate=0.1):
    return [
        gene + random.gauss(0, 1) if random.random() < mutation_rate else gene
        for gene in individual
    ]

def crossover(parent1, parent2):
    point = random.randint(1, len(parent1) - 1)
    child = parent1[:point] + parent2[point:]
    return child

def parallel_ga_step(args):
    population, fitness_fn, mutation_rate, elite_count = args
    fitnesses = [fitness_fn(ind) for ind in population]
    sorted_indices = sorted(range(len(fitnesses)), key=lambda i: fitnesses[i])
    population = [population[i] for i in sorted_indices]

    elite = population[:elite_count]
    offspring = elite[:]

    while len(offspring) < len(population):
        p1 = random.choice(population[:len(population)//2])
        p2 = random.choice(population[:len(population)//2])
        child = crossover(p1, p2)
        child = mutate(child, mutation_rate)
        offspring.append(child)

    return offspring, fitnesses[sorted_indices[0]]

class ParallelGeneticAlgorithm:
    def __init__(self, population_size=100, genome_length=10, num_islands=4):
        self.population_size = population_size
        self.genome_length = genome_length
        self.num_islands = num_islands

    def run(self, generations=50):
        islands = [
            [[random.gauss(0, 1) for _ in range(self.genome_length)]
             for _ in range(self.population_size // self.num_islands)]
            for _ in range(self.num_islands)
        ]

        with mp.Pool(self.num_islands) as pool:
            for gen in range(generations):
                args = [(island, evaluate_fitness, 0.1, 2) for island in islands]
                results = pool.map(parallel_ga_step, args)

                best_fitness = min(r[1] for r in results)
                islands = [r[0] for r in results]

                if gen % 10 == 0:
                    print(f"Generation {gen}: best fitness = {best_fitness:.4f}")

        return islands

if __name__ == "__main__":
    ga = ParallelGeneticAlgorithm(population_size=40, genome_length=5, num_islands=4)
    final_populations = ga.run(generations=20)
    print(f"Final population size: {sum(len(p) for p in final_populations)}")
```

## Real-World Use Cases

- **Image/Video processing**: Applying filters, transformations, or encoding to multiple frames in parallel
- **Data science preprocessing**: Parallel feature extraction, normalization, and data cleaning on large datasets
- **Scientific simulations**: Monte Carlo simulations, parameter sweeps, and ensemble computations
- **Web scraping**: Crawling multiple pages simultaneously without being limited by I/O
- **Batch file processing**: Converting, compressing, or analyzing thousands of files
- **Machine learning**: Parallel grid search for hyperparameter tuning, ensemble training

## Common Mistakes

- Forgetting to wrap process-spawning code in `if __name__ == "__main__":` on Windows
- Passing unpicklable objects (lambdas, bound methods) as targets or arguments
- Creating more processes than CPU cores, causing excessive context switching
- Not handling process termination properly, leaving zombie processes
- Using shared state without proper synchronization (locks on shared Value/Array)
- Overusing Manager for data sharing (slower than Value/Array/Pipe)
- Ignoring that `fork` (default on Linux) can cause deadlocks with threads
- Not closing Queues and Pipes, leading to resource leaks

## Best Practices

- Always guard process-spawning code with `if __name__ == "__main__":`
- Use `Pool` for most parallel tasks instead of managing individual processes
- Prefer `Pool.map()` and `Pool.starmap()` for data-parallel operations
- Use `mp.Queue` or `mp.Pipe` for communication, avoiding shared state when possible
- Use `mp.Manager` for complex shared data structures, `mp.Value`/`mp.Array` for simple types
- Set reasonable chunk sizes in `map` to balance load distribution
- Use `pool.imap_unordered()` when result order doesn't matter for better performance
- Always terminate pools with `pool.close()` followed by `pool.join()`

## Interview Questions

1. **Q**: How does multiprocessing bypass the GIL?
   **A**: Each process has its own Python interpreter with its own GIL, so multiple processes can execute Python bytecode truly in parallel across CPU cores.

2. **Q**: What is the difference between `Pool.map()` and `Pool.apply_async()`?
   **A**: `Pool.map()` applies a function to every item in an iterable, blocking until all results are ready. `Pool.apply_async()` submits a single task and returns a result object immediately, allowing asynchronous result retrieval.

3. **Q**: When would you use `mp.Queue` vs `mp.Pipe`?
   **A**: Use Queue for multiple producers/consumers (thread-safe, supports many-to-many). Use Pipe for two-way communication between two processes (simpler and faster for point-to-point).

4. **Q**: What is the difference between `fork`, `spawn`, and `forkserver` start methods?
   **A**: `fork` (default on Unix) clones the parent process including all threads, which can cause deadlocks. `spawn` (default on Windows) starts a fresh Python process. `forkserver` (Unix) uses a server process that forks for each new process, avoiding thread issues.

5. **Q**: How do you share state between processes?
   **A**: Options include: (1) `mp.Value`/`mp.Array` for simple shared memory, (2) `mp.Manager` for high-level shared objects, (3) `mp.Queue`/`mp.Pipe` for message passing, (4) Redis or other external stores for distributed state.

## Coding Challenges

1. **Parallel Prime Sieve**: Implement a parallel version of the Sieve of Eratosthenes that splits the number range across multiple processes and merges results.

2. **Distributed MapReduce**: Build a simple MapReduce framework using multiprocessing. Implement word count as a test case with mapper and reducer processes.

3. **Parallel Image Processor**: Write a script that applies a filter (e.g., Gaussian blur) to multiple images in parallel using a process pool.

4. **Monte Carlo Pi Estimator**: Estimate pi using Monte Carlo simulation with parallel workers, each running a fraction of the total samples.

5. **Process Monitoring Dashboard**: Create a tool that spawns worker processes, monitors their CPU/memory usage, and provides a live dashboard of their status.

## Summary

The `multiprocessing` module provides true parallelism in Python by spawning separate processes with independent GILs. Key components include Process, Pool, Queue, Pipe, Value/Array for shared memory, and Manager for complex shared data. Multiprocessing is the go-to solution for CPU-bound tasks that need to utilize multiple cores.

## Related Topics

Multithreading (59.x), Thread Safety (63.x), GIL (64.x), Async IO (61.x)
