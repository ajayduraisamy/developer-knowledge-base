# Profiling - cProfile, timeit, line_profiler, memory_profiler

## Introduction
Profiling is the systematic measurement of a program's execution characteristics — time spent per function, memory allocated, and call frequency. Python provides both built-in tools (`cProfile`, `timeit`) and third-party packages (`line_profiler`, `memory_profiler`) that help developers identify bottlenecks, reduce latency, and optimise resource usage. Without profiling, performance tuning is guesswork; with it, every optimisation is data-driven.

## cProfile

### What It Is
`cProfile` is Python's built-in deterministic profiler written in C. It hooks into the interpreter's call mechanism and records every function entry and exit, capturing cumulative and per-call timing statistics. It is the recommended profiler for most use cases because it imposes moderate overhead (typically 10–100x slowdown) and works without modification to the target code.

### Why It Is Important
cProfile reveals exactly which functions consume the most total time (cumulative) and which contribute the most per-call overhead. This data lets you focus optimisation effort on the hotspots that actually matter, rather than on code that runs trivially fast.

### How It Works Internally
cProfile registers a callback with `PyEval_SetProfile` inside CPython. On each function call and return, the callback records the current `PyObject_Call` location, increments a call count, and adds the wall-clock delta to the function's cumulative timer. At the end, the collected `Stat` objects are accumulated into a dictionary keyed by function identity (filename, line, function name) and sorted by various metrics.

### Syntax
```python
import cProfile
import pstats

# Profile a single expression
cProfile.run('my_function()', sort='cumtime')

# Profile and save to file
cProfile.run('my_function()', 'output.prof')

# Analyse saved stats
p = pstats.Stats('output.prof')
p.sort_stats('cumtime').print_stats(20)

# Profile from the command line
# python -m cProfile -s cumtime my_script.py
```

### Beginner Examples
```python
import cProfile

def slow_sum(n):
    total = 0
    for i in range(n):
        total += i ** 2
    return total

cProfile.run('slow_sum(100000)', sort='cumtime')
```

### Intermediate Examples
```python
import cProfile
import pstats
import io

def process_data(items):
    return [expensive(x) for x in items]

def expensive(x):
    total = 0
    for i in range(1000):
        total += (x * i) ** 0.5
    return total

profiler = cProfile.Profile()
profiler.enable()
result = process_data(range(200))
profiler.disable()

s = io.StringIO()
ps = pstats.Stats(profiler, stream=s).sort_stats('cumtime')
ps.print_stats()
print(s.getvalue())
```

### Advanced Examples
```python
import cProfile
import pstats
import functools

def profile_decorator(output_file=None, sort_by='cumtime'):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            profiler = cProfile.Profile()
            profiler.enable()
            try:
                return func(*args, **kwargs)
            finally:
                profiler.disable()
                s = io.StringIO()
                ps = pstats.Stats(profiler, stream=s).sort_stats(sort_by)
                ps.print_stats(30)
                with open(output_file or f'{func.__name__}.prof', 'w') as f:
                    f.write(s.getvalue())
        return wrapper
    return decorator

@profile_decorator(output_file='compute.prof')
def compute(matrix_size):
    return [[i * j for j in range(matrix_size)] for i in range(matrix_size)]

# Filtering specific functions
p = pstats.Stats('compute.prof')
p.sort_stats('time')
p.print_callers('compute')
p.print_callees('numpy')
```

### Real-World Use Cases
- **API endpoint profiling**: Profile a Flask/Django view to find which database query or serialisation step dominates response time.
- **Batch job optimisation**: Identify functions that scale quadratically with input size and target them for algorithmic improvement.
- **CI benchmarking**: Run cProfile as part of a test suite and fail a build if cumulative time regresses beyond a threshold.

### Common Mistakes
- Profiling with optimisation flags (`-O`) that alter bytecode and can mask overhead.
- Profiling in an IDE debugger that itself adds profiling hooks — results are contaminated.
- Relying on wall-clock time alone without separating I/O wait from CPU time.
- Forgetting to sort or filter stats when dealing with thousands of calls — leads to analysis paralysis.

### Best Practices
- Profile realistic workloads, not tiny unit tests — the profile must reflect actual usage patterns.
- Always run the profiled code at least three times and take median values to reduce noise.
- Use `pstats` filtering (`print_stats(20)`) to focus on the top offenders.
- Combine cProfile with `time.Timer` for coarse-grained phase-level measurement.

### Performance Considerations
- cProfile adds 10–100x overhead; for very short-running functions, consider `timeit` instead.
- Writing large profile files can consume significant disk space — use filters or summarise in memory.
- Multithreaded code is profiled per-thread; the GIL limits concurrency, so profile results may not reflect parallel execution.

### Interview Questions
- **Q**: What is the difference between `cProfile` and `profile`?  
  **A**: `cProfile` is written in C and has much lower overhead; `profile` is pure Python and primarily exists as a fallback.
- **Q**: How do you identify which function a caller is spending the most time in?  
  **A**: Use `pstats.print_callers()` to see which callers invoke which callees and how much time they consume.

### Coding Challenges
- Write a context manager that profiles a block of code and prints the top 5 functions by cumulative time.
- Implement a lightweight line-count profiler that tracks total time per source line without using cProfile.

### Related Topics
- [timeit module](#timeit-module)
- [line_profiler](#line_profiler)
- [memory_profiler](#memory_profiler)

---

## timeit module

### What It Is
The `timeit` module provides a precise way to measure the execution time of small code snippets. It avoids common pitfalls such as garbage collection interference and global variable lookup overhead by creating a clean execution environment and repeating the measurement many times.

### Why It Is Important
Micro-benchmarks help compare two implementation approaches at the granularity of a single expression or function call. `timeit` ensures reproducibility by disabling the garbage collector, using `time.perf_counter` internally, and automatically determining the number of repetitions needed for statistical stability.

### How It Works Internally
`timeit.Timer` constructs a miniature execution environment using `timeit`'s internal namespace. It compiles the statement into bytecode once, then calls `PyEval_EvalCode` in a loop. Before timing, it calls `gc.enable()` / `gc.disable()` based on the `globals` parameter. The actual timing is done with `time.perf_counter` on POSIX or `QueryPerformanceCounter` on Windows, which offers sub-microsecond resolution.

### Syntax
```python
import timeit

# Time a statement
timeit.timeit('"-".join(str(n) for n in range(100))', number=10000)

# Time a callable
timeit.timeit(lambda: sum(range(100)), number=10000)

# With setup code
timeit.timeit('func(10)', setup='from __main__ import func', number=1000)

# Command line
# python -m timeit -n 1000 -r 5 "sum(range(100))"

# Repeat for statistics
timeit.repeat('sqrt(2)', setup='from math import sqrt', repeat=5, number=100000)
```

### Beginner Examples
```python
import timeit

# Compare list comprehension vs explicit loop
list_comp_time = timeit.timeit(
    '[i**2 for i in range(1000)]',
    number=1000
)

loop_time = timeit.timeit(
    '''
result = []
for i in range(1000):
    result.append(i**2)
''',
    number=1000
)

print(f'List comp: {list_comp_time:.4f}s, Loop: {loop_time:.4f}s')
```

### Intermediate Examples
```python
import timeit
from functools import partial

# Timing a function with arguments
def multiply(x, y):
    return x * y

t = timeit.timeit(
    partial(multiply, 10, 20),
    number=100000
)

# Using globals to access module functions
t2 = timeit.timeit(
    'multiply(10, 20)',
    globals=globals(),
    number=100000
)

# Repeat to get min/max/stdev
results = timeit.repeat(
    '[i**0.5 for i in range(100)]',
    repeat=5,
    number=10000
)
print(f'Min: {min(results):.6f}, Max: {max(results):.6f}')
```

### Advanced Examples
```python
import timeit
import numpy as np

# Benchmark different data structure lookups
setup_dict = '''
d = {i: i for i in range(10000)}
keys = list(range(10000))
'''
setup_list = '''
lst = list(range(10000))
keys = list(range(10000))
'''

dict_lookup = timeit.timeit(
    'for k in keys: _ = d[k]',
    setup=setup_dict,
    number=1000
)

list_index = timeit.timeit(
    'for k in keys: _ = lst[k]',
    setup=setup_list,
    number=1000
)

# Custom Timer subclass with warmup
class WarmupTimer(timeit.Timer):
    def timeit(self, number=1000000):
        self.warmup()
        return super().timeit(number)

    def warmup(self, iterations=100):
        _ = self.timeit(number=iterations)

# Benchmarking with noise reduction
import statistics
raw = timeit.repeat('sorted(lst)', setup='import random; lst = [random.random() for _ in range(1000)]', repeat=15, number=100)
clean = [t for t in raw if abs(t - statistics.median(raw)) < statistics.stdev(raw) * 2]
print(f'Mean: {statistics.mean(clean):.6f}s')
```

### Real-World Use Cases
- **Library comparison**: compare `json.loads` vs `orjson.loads` on identical payloads.
- **Algorithm micro-benchmark**: determine whether `bisect.insort` outperforms `list.append` + `sorted` for incremental inserts.
- **CI regression detection**: run a suite of timeit benchmarks as part of pre-merge checks and fail if any degrades beyond 5%.

### Common Mistakes
- Timing I/O-bound code — `timeit` measures wall clock and disk/network variance will dominate.
- Using `number` too small, causing timer resolution errors.
- Forgetting to include `setup` — code that imports modules or builds data structures should run once in setup, not in the timed statement.

### Best Practices
- Always use `number` sufficiently large (aim for at least 0.2 seconds per measurement).
- Use `timeit.repeat` and report min (best-case) rather than mean to reduce noise from system interrupts.
- Keep timed statements minimal — anything outside the expression of interest distorts results.
- Disable the GC before timing critical micro-benchmarks when allocation patterns are relevant.

### Performance Considerations
- The overhead of the `timeit` loop itself is negligible for statements above ~1 microsecond.
- For nanosecond-level precision, use `perf_counter_ns` and a tight C loop — timeit cannot go below its overhead floor.
- Results are deterministic only on the same machine under the same load; never compare absolute values across environments.

### Interview Questions
- **Q**: How does `timeit` avoid garbage collection interference?  
  **A**: By default, `timeit` temporarily disables the garbage collector with `gc.disable()` before running the timed loop.
- **Q**: What does `timeit.repeat` return and how do you interpret it?  
  **A**: It returns a list of `number * repeat` total runs, each entry being the best of one `repeat` call. The minimum of the list is the most reliable measure.

### Coding Challenges
- Implement a `@benchmark` decorator that uses `timeit` to automatically measure and log function execution time.
- Write a function that compares two callables and returns a confidence interval indicating whether one is statistically faster.

### Related Topics
- [cProfile](#cprofile)
- [line_profiler](#line_profiler)

---

## line_profiler

### What It Is
`line_profiler` is a third-party package (installable via `pip install line_profiler`) that measures time spent per line of Python code. It uses a C extension to inject profiling hooks at the bytecode-line level, producing output that shows exactly which line inside a function is the bottleneck.

### Why It Is Important
While cProfile tells you *which function* is slow, line_profiler tells you *which line* within that function is slow. This granularity is critical for optimising complex functions where a single expression (e.g., a list comprehension or a deep attribute access) accounts for most of the runtime.

### How It Works Internally
LineProfiler registers a `trace` function via `sys.settrace`. For each line executed, it records a timestamp using `PyTime_GetSystemTime`. The C extension maintains a per-function array of line timings stored as `long long` microseconds. During decoration, the profiler replaces the original function with a wrapper that enables tracing only while the function runs. At `print_stats()`, it loads the source file and correlates line numbers with accumulated timings.

### Syntax
```python
# Decorate functions you want to profile
from line_profiler import LineProfiler

profiler = LineProfiler()

@profiler
def my_function():
    ...

# Or add functions later
profiler.add_function(other_func)

# Run and dump
profiler.enable()
my_function()
profiler.disable()
profiler.print_stats()

# Command line
# kernprof -l -v my_script.py
```

### Beginner Examples
```python
from line_profiler import LineProfiler

def process(items):
    result = []
    for item in items:
        transformed = item * 2 + 1
        result.append(transformed ** 0.5)
    return result

lp = LineProfiler()
lp_wrapper = lp(process)
lp_wrapper(range(10000))
lp.print_stats()
```

### Intermediate Examples
```python
from line_profiler import LineProfiler
import numpy as np

def complex_pipeline(data):
    cleaned = [x for x in data if x > 0]          # line 1
    normalized = np.log1p(cleaned)                 # line 2
    fft = np.fft.fft(normalized)                   # line 3
    magnitudes = np.abs(fft)                       # line 4
    threshold = np.percentile(magnitudes, 95)      # line 5
    peaks = magnitudes[magnitudes > threshold]     # line 6
    return peaks

data = np.random.randn(100000)
lp = LineProfiler()
lp.add_function(complex_pipeline)
lp.enable()
complex_pipeline(data)
lp.disable()
lp.print_stats()
```

### Advanced Examples
```python
from line_profiler import LineProfiler
import functools

# Context manager for selective line profiling
class LineProfileContext:
    def __init__(self, *funcs):
        self.profiler = LineProfiler(*funcs)

    def __enter__(self):
        self.profiler.enable()
        return self.profiler

    def __exit__(self, *args):
        self.profiler.disable()
        self.profiler.print_stats()

# Usage
with LineProfileContext(complex_pipeline) as lp:
    result = complex_pipeline(data)

# Profiling class methods
class DataProcessor:
    def transform(self, x):
        return x ** 2

    def run(self, items):
        return [self.transform(i) for i in items]

lp = LineProfiler(DataProcessor.transform, DataProcessor.run)
lp.enable()
dp = DataProcessor()
dp.run(range(10000))
lp.disable()
lp.print_stats()
```

### Real-World Use Cases
- **Data pipeline debugging**: identify which pandas operation in a 20-line transformation chain is taking 80% of the time.
- **Game loop optimisation**: pinpoint the exact line in the physics tick that exceeds the frame budget.
- **Scientific computation**: locate the array expression that triggers an unexpected copy or misaligned memory access.

### Common Mistakes
- Leaving `@profile` decorators in production code — they produce an error if `line_profiler` is not installed.
- Profiling functions that call C extensions — the profiler cannot instrument C-level lines, so time accumulates on the Python call line.
- Using `sys.settrace` in the same process — line_profiler interferes with other tracing tools.

### Best Practices
- Decorate only the top 2–3 functions suspected of being slow; line_profiler overhead is significant (10–100x on each profiled line).
- Run with realistic data sizes — profiling a 100-item list won't show the O(n^2) problem that manifests at 100k items.
- Use `kernprof -l -v` for scripts — it handles enabling/disabling and writes a `.lprof` file for later review.

### Performance Considerations
- Line_profiler slows execution by 10–100x because it calls the trace function for every line.
- Memory overhead grows with the number of unique lines traced; for very large modules, memory can become an issue.
- Cannot profile C extension internals — only the Python call site that enters the extension.

### Interview Questions
- **Q**: How does line_profiler differ from cProfile?  
  **A**: cProfile profiles at function granularity with lower overhead; line_profiler profiles at per-line granularity with higher overhead.
- **Q**: What is the role of `sys.settrace` in line_profiler?  
  **A**: It registers a callback that fires on every line executed, enabling per-line timing accumulation.

### Coding Challenges
- Implement a minimal line profiler that records cumulative time per line using `sys.settrace` and `time.perf_counter`.
- Write a script that runs `kernprof` on a given Python file and parses the `.lprof` output to find the slowest 5 lines.

### Related Topics
- [cProfile](#cprofile)
- [memory_profiler](#memory_profiler)

---

## memory_profiler

### What It Is
`memory_profiler` is a third-party package (`pip install memory_profiler[psutil]`) that tracks memory usage line-by-line, function-by-function, or over time via a separate monitoring process. It uses `psutil` to read process memory from `/proc/self/status` on Linux or the Win32 API on Windows.

### Why It Is Important
Time performance is only half the story — memory leaks, excessive allocation, and high peak memory can crash production services. memory_profiler helps track down which function or line allocates the most objects, enabling targeted reduction of memory pressure.

### How It Works Internally
memory_profiler supports two modes:
1. **Line-by-line**: Similar to line_profiler, it uses `sys.settrace` and records memory before and after each line via `psutil.Process().memory_info().rss`.
2. **Sampling**: spawns a background thread that reads RSS at regular intervals while the target function runs, producing a time-series memory usage chart.

### Syntax
```python
from memory_profiler import profile

@profile
def my_function():
    ...

# Line-by-line profiling
# python -m memory_profiler my_script.py

# Time-based memory usage
from memory_profiler import memory_usage
mem = memory_usage((func, args, kwargs), interval=0.1, timeout=30)
```

### Beginner Examples
```python
from memory_profiler import profile

@profile
def load_large_file():
    with open('data.txt', 'r') as f:
        content = f.read()
    data = content.split('\n')
    return [len(line) for line in data]

load_large_file()
```

### Intermediate Examples
```python
from memory_profiler import profile
import numpy as np

@profile
def memory_hungry(size=10000):
    a = np.random.randn(size, size)       # 800 MB
    b = a ** 2
    c = np.sqrt(b)
    d = c.sum(axis=0)
    return d

# Tracking memory over time
from memory_profiler import memory_usage
import matplotlib.pyplot as plt

def leaky():
    container = []
    for i in range(1000):
        container.append('x' * 10_000_000)
    return container

mem_ts = memory_usage((leaky, (), {}), interval=0.1)
plt.plot(mem_ts)
plt.ylabel('Memory (MB)')
```

### Advanced Examples
```python
from memory_profiler import profile
import gc

# Selective profiling with filtering
@profile(precision=4, stream=open('memory.log', 'w'))
def process_chunks(chunks):
    results = []
    for chunk in chunks:
        buf = bytearray(chunk)           # large allocation
        transformed = buf.hex()
        results.append(transformed)
        del buf
        gc.collect()
    return results

# Object-level memory tracking
import tracemalloc
tracemalloc.start()

snapshot1 = tracemalloc.take_snapshot()
data = [bytearray(10_000) for _ in range(500)]
snapshot2 = tracemalloc.take_snapshot()

stats = snapshot2.compare_to(snapshot1, 'lineno')
for stat in stats[:10]:
    print(stat)

# Memory usage of a generator vs list
@profile
def use_list(n):
    return [i for i in range(n)]

@profile
def use_generator(n):
    return (i for i in range(n))

use_list(10_000_000)
use_generator(10_000_000)
```

### Real-World Use Cases
- **Docker memory limit debugging**: find which request handler spikes memory above the container limit.
- **Data pipeline memory planning**: measure peak RSS of an ETL job to choose instance size.
- **Leak detection in long-running services**: run memory_usage on a loop and alert if RSS grows monotonically.

### Common Mistakes
- Using `@profile` without the decorator being defined — wrap in a try/except ImportError for production safety.
- Profiling with the Garbage Collector enabled — GC may collect at unpredictable times, distorting line-level deltas.
- Expecting deterministic results — RSS includes OS page cache and shared libraries, so readings vary between runs.

### Best Practices
- Use `tracemalloc` for object-level allocation tracing and `memory_profiler` for high-level RSS tracking.
- Profile memory with a realistic but reduced dataset first — full-scale profiling can be extremely slow.
- Combine memory profiling with `gc.get_objects()` and `objgraph` to identify reference cycles.

### Performance Considerations
- Line-by-line memory profiling slows execution by 20–100x due to the trace hook.
- Sampling mode (memory_usage) has negligible overhead (< 1%) and is suitable for production monitoring.
- RSS measurements have ~1 MB resolution on most systems — small allocations won't register.

### Interview Questions
- **Q**: What is the difference between RSS and the sum of `sys.getsizeof` for live objects?  
  **A**: RSS includes interpreter overhead, shared libraries, and memory fragmentation; `getsizeof` only measures individual object sizes.
- **Q**: How does `tracemalloc` track allocations without a trace hook?  
  **A**: Tracemalloc hooks into CPython's memory allocator (PyMem_Malloc) via C API callbacks, recording stack traces for each allocation.

### Coding Challenges
- Write a decorator that logs peak memory usage and peak allocation time for any function.
- Use `tracemalloc` to find the top 10 allocation sources in a running application and print their stack traces.

### Related Topics
- [cProfile](#cprofile)
- [line_profiler](#line_profiler)
- [Memory Management](#memory-management---garbage-collection-reference-counting-weakref)
