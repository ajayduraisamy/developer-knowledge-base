# Profiling - cProfile, timeit, line_profiler, memory_profiler

## Introduction

Profiling is the process of measuring the time and memory complexity of a program to identify performance bottlenecks. Python provides several built-in and third-party tools like cProfile, profile, pstats, and snakeviz to help developers analyze and optimize their code.

## Why It Is Important

Profiling helps developers understand which parts of their code are slow or memory-intensive. Without profiling, optimizations are guesswork. Profiling ensures that effort is focused on the actual bottlenecks, leading to efficient code with minimal wasted development time.

## Syntax

```python
# Using cProfile from the command line
# python -m cProfile script.py

# Using cProfile in code
import cProfile
profiler = cProfile.Profile()
profiler.enable()
# code to profile
profiler.disable()
profiler.print_stats(sort='time')

# Using pstats to analyze results
import pstats
from pstats import SortKey
p = pstats.Stats('output.prof')
p.sort_stats(SortKey.CUMULATIVE).print_stats(20)

# Using timeit
import timeit
timeit.timeit('"-".join(str(n) for n in range(100))', number=10000)
timeit.repeat('"-".join(str(n) for n in range(100))', number=10000, repeat=5)
```

## Examples

```python
import cProfile
import pstats
import io
import time


def slow_function():
    total = 0
    for i in range(1000000):
        total += i ** 2
    return total


def fast_function():
    return sum(i ** 2 for i in range(1000000))


def main():
    slow_function()
    fast_function()


if __name__ == '__main__':
    profiler = cProfile.Profile()
    profiler.enable()
    main()
    profiler.disable()
    stream = io.StringIO()
    stats = pstats.Stats(profiler, stream=stream)
    stats.sort_stats('cumulative')
    stats.print_stats(10)
    print(stream.getvalue())
```

## Beginner Examples

```python
import timeit


def square_numbers():
    result = []
    for i in range(1000):
        result.append(i ** 2)
    return result


def square_numbers_comprehension():
    return [i ** 2 for i in range(1000)]


time_for_loop = timeit.timeit(square_numbers, number=10000)
time_comp = timeit.timeit(square_numbers_comprehension, number=10000)

print(f"For loop: {time_for_loop:.4f} seconds")
print(f"Comprehension: {time_comp:.4f} seconds")


def measure_small_snippets():
    snippet1 = "sum([i**2 for i in range(100)])"
    snippet2 = "sum(i**2 for i in range(100))"
    t1 = timeit.timeit(snippet1, number=100000)
    t2 = timeit.timeit(snippet2, number=100000)
    print(f"List comprehension: {t1:.4f}")
    print(f"Generator expression: {t2:.4f}")


measure_small_snippets()
```

## Intermediate Examples

```python
import cProfile
import pstats
import io
import functools


def profile_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        profiler = cProfile.Profile()
        profiler.enable()
        result = func(*args, **kwargs)
        profiler.disable()
        stream = io.StringIO()
        stats = pstats.Stats(profiler, stream=stream)
        stats.sort_stats('cumulative')
        stats.print_stats(15)
        print(f"Profile for {func.__name__}:")
        print(stream.getvalue())
        return result
    return wrapper


@profile_decorator
def data_processing_pipeline():
    data = [i for i in range(100000)]
    filtered = [x for x in data if x % 2 == 0]
    mapped = [x ** 2 for x in filtered]
    reduced = sum(mapped)
    return reduced


@profile_decorator
def alternative_pipeline():
    return sum(x ** 2 for x in range(100000) if x % 2 == 0)


result1 = data_processing_pipeline()
result2 = alternative_pipeline()
print(f"Results match: {result1 == result2}")


def profile_and_save(filename='profile_output.prof'):
    profiler = cProfile.Profile()
    profiler.enable()
    data = [i ** 2 for i in range(500000)]
    sorted_data = sorted(data, reverse=True)
    total = sum(sorted_data[:1000])
    profiler.disable()
    profiler.dump_stats(filename)
    print(f"Profile saved to {filename}")


profile_and_save()


def analyze_profile(filename='profile_output.prof'):
    p = pstats.Stats(filename)
    print("=" * 60)
    print("Top 10 by cumulative time:")
    p.sort_stats('cumulative').print_stats(10)
    print("\nTop 10 by number of calls:")
    p.sort_stats('calls').print_stats(10)
    print("\nTop 10 by time per call:")
    p.sort_stats('time').print_stats(10)


analyze_profile()
```

## Advanced Examples

```python
import cProfile
import pstats
import io
import functools
import time
import random
from typing import List, Dict, Any


class ProfilingContextManager:
    def __init__(self, sort_by='cumulative', lines=20, output_file=None):
        self.sort_by = sort_by
        self.lines = lines
        self.output_file = output_file
        self.profiler = cProfile.Profile()
        self.stream = io.StringIO()

    def __enter__(self):
        self.profiler.enable()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.profiler.disable()
        stats = pstats.Stats(self.profiler, stream=self.stream)
        stats.sort_stats(self.sort_by)
        stats.print_stats(self.lines)
        if self.output_file:
            self.profiler.dump_stats(self.output_file)
            print(f"Profile saved to {self.output_file}")
        print(self.stream.getvalue())


def fibonacci_recursive(n: int) -> int:
    if n < 2:
        return n
    return fibonacci_recursive(n - 1) + fibonacci_recursive(n - 2)


def fibonacci_iterative(n: int) -> int:
    if n < 2:
        return n
    a, b = 0, 1
    for _ in range(n - 1):
        a, b = b, a + b
    return b


def fibonacci_memoized(n: int, memo: Dict[int, int] = None) -> int:
    if memo is None:
        memo = {}
    if n in memo:
        return memo[n]
    if n < 2:
        return n
    memo[n] = fibonacci_memoized(n - 1, memo) + fibonacci_memoized(n - 2, memo)
    return memo[n]


def sort_large_list(data: List[int]) -> List[int]:
    return sorted(data)


def process_data(data: List[int]) -> Dict[str, Any]:
    result = {}
    result['sum'] = sum(data)
    result['mean'] = sum(data) / len(data)
    result['sorted'] = sort_large_list(data)
    result['median'] = result['sorted'][len(data) // 2]
    return result


def complex_workflow():
    data = [random.randint(0, 10000) for _ in range(10000)]
    processed = process_data(data)
    fib_result = fibonacci_memoized(30)
    print(f"Fibonacci(30) = {fib_result}")
    print(f"Processed data median: {processed['median']}")
    return processed


with ProfilingContextManager(sort_by='cumulative', lines=20):
    complex_workflow()


def line_profiler_example():
    import line_profiler
    import sys
    print("Line profiler usage:")
    print("  pip install line_profiler")
    print("  kernprof -l -v script.py")
    print("  @profile decorator on functions to profile")


line_profiler_example()


def memory_profiler_example():
    import memory_profiler
    print("Memory profiler usage:")
    print("  pip install memory_profiler")
    print("  python -m memory_profiler script.py")
    print("  @profile decorator to track memory usage")


memory_profiler_example()
```

## Real-World Use Cases

```python
import cProfile
import pstats
import io
import time
import functools
from typing import List


def profile_api_endpoint(endpoint_func):
    @functools.wraps(endpoint_func)
    def wrapper(*args, **kwargs):
        profiler = cProfile.Profile()
        profiler.enable()
        start = time.perf_counter()
        result = endpoint_func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        profiler.disable()
        stream = io.StringIO()
        stats = pstats.Stats(profiler, stream=stream)
        stats.sort_stats('time')
        stats.print_stats(10)
        print(f"Endpoint {endpoint_func.__name__} took {elapsed:.4f}s")
        print(stream.getvalue())
        return result
    return wrapper


@profile_api_endpoint
def process_user_data(user_ids: List[int]) -> List[dict]:
    users = []
    for uid in user_ids:
        user = {'id': uid, 'name': f'User_{uid}', 'scores': [random.randint(0, 100) for _ in range(1000)]}
        user['average'] = sum(user['scores']) / len(user['scores'])
        users.append(user)
    users.sort(key=lambda u: u['average'], reverse=True)
    return users


import random
top_users = process_user_data(list(range(1, 101)))
print(f"Top user: {top_users[0]['name']} with avg {top_users[0]['average']:.2f}")


def database_query_profiling():
    print("Example: Profiling database queries")
    print("1. Profile the ORM query execution")
    print("2. Identify N+1 query problems")
    print("3. Measure query response times")
    print("4. Compare raw SQL vs ORM performance")


database_query_profiling()


def image_processing_profile():
    print("Example: Profiling image processing pipeline")
    print("1. Profile image loading time")
    print("2. Profile filtering operations")
    print("3. Profile resize and transform operations")
    print("4. Profile save/export operations")


image_processing_profile()
```

## Common Mistakes

```python
import timeit
import cProfile


def mistake_1_optimizing_without_profiling():
    print("Mistake 1: Optimizing code without profiling first")
    print("Always profile before optimizing to find real bottlenecks")


def mistake_2_too_few_iterations():
    print("Mistake 2: Using too few iterations in timeit")
    result = timeit.timeit('x = 1 + 2', number=10)
    print(f"10 iterations: {result:.10f}s (unreliable)")
    result = timeit.timeit('x = 1 + 2', number=1000000)
    print(f"1M iterations: {result:.6f}s (reliable)")


def mistake_3_profile_overhead():
    print("Mistake 3: Ignoring profiler overhead")
    print("cProfile adds overhead - use it for relative comparisons")


def mistake_4_not_saving_profiles():
    print("Mistake 4: Not saving profiles for comparison")
    print("Save .prof files to compare before/after optimization")


def mistake_5_wrong_sort_key():
    print("Mistake 5: Using wrong sort key in pstats")
    print("Use 'cumulative' for total time, 'time' for per-call time")


mistake_1_optimizing_without_profiling()
mistake_2_too_few_iterations()
mistake_3_profile_overhead()
mistake_4_not_saving_profiles()
mistake_5_wrong_sort_key()
```

## Best Practices

```python
import cProfile
import pstats
import io


def profile_small_functions():
    print("Best Practice 1: Profile small, focused functions")
    print("Break down large functions for better profiling")


def use_calibrate():
    print("Best Practice 2: Calibrate the profiler for accuracy")
    profiler = cProfile.Profile()
    profiler.calibrate(100000)


def compare_alternatives():
    print("Best Practice 3: Compare alternative implementations")
    print("Profile both versions and compare the results")


def profile_memory_too():
    print("Best Practice 4: Profile memory along with time")
    print("Use memory_profiler to track memory usage")


def automate_profiling():
    print("Best Practice 5: Automate profiling in CI/CD")
    print("Use pytest-benchmark or custom profiling in test suite")


def use_snakeviz():
    print("Best Practice 6: Use snakeviz for visualization")
    print("  pip install snakeviz")
    print("  snakeviz profile_output.prof")


def profile_in_stages():
    print("Best Practice 7: Profile in stages")
    print("1. High-level profiling to find slow modules")
    print("2. Medium-level profiling to find slow functions")
    print("3. Low-level profiling to find slow lines")


profile_small_functions()
use_calibrate()
compare_alternatives()
profile_memory_too()
automate_profiling()
use_snakeviz()
profile_in_stages()
```

## Interview Questions

```python
def interview_q1():
    print("Q: What is the difference between cProfile and profile?")
    print("A: cProfile is written in C for lower overhead.")
    print("   profile is written in pure Python (more overhead).")


def interview_q2():
    print("Q: How do you identify bottlenecks using cProfile?")
    print("A: Sort by cumulative time, look for functions with")
    print("   high total time but low per-call time (many calls).")


def interview_q3():
    print("Q: What is snakeviz and how does it help?")
    print("A: A browser-based visualization tool for cProfile output.")
    print("   It shows icicle and flame graphs of call stacks.")


def interview_q4():
    print("Q: How does timeit work internally?")
    print("A: It disables garbage collection, runs the code")
    print("   multiple times, and uses the best result to")
    print("   minimize system interference.")


def interview_q5():
    print("Q: What is line_profiler?")
    print("A: A tool that profiles your code line by line.")
    print("   Use @profile decorator and kernprof command.")


def interview_q6():
    print("Q: How do you interpret pstats output?")
    print("A: ncalls (call count), tottime (total time in function),")
    print("   percall (tottime/ncalls), cumtime (cumulative time),")
    print("   filename:lineno(function)")


interview_q1()
interview_q2()
interview_q3()
interview_q4()
interview_q5()
interview_q6()
```

## Coding Challenges

```python
import cProfile
import pstats
import io
import functools


def challenge_1_find_slowest_function():
    print("Challenge 1: Profile this code and find the slowest function")

    def func_a():
        return sum(i ** 2 for i in range(100000))

    def func_b():
        return sum(i ** 3 for i in range(100000))

    def func_c():
        return sum(i ** 0.5 for i in range(100000))

    profiler = cProfile.Profile()
    profiler.enable()
    for _ in range(100):
        func_a()
        func_b()
        func_c()
    profiler.disable()
    stream = io.StringIO()
    stats = pstats.Stats(profiler, stream=stream)
    stats.sort_stats('cumulative')
    stats.print_stats(5)
    print(stream.getvalue())


def challenge_2_optimize_with_profiling():
    print("Challenge 2: Profile and optimize this function")

    def slow_duplicate_removal(data):
        result = []
        for item in data:
            if item not in result:
                result.append(item)
        return result

    def fast_duplicate_removal(data):
        return list(dict.fromkeys(data))

    data = [i % 1000 for i in range(10000)]
    profiler = cProfile.Profile()
    profiler.enable()
    slow_duplicate_removal(data)
    fast_duplicate_removal(data)
    profiler.disable()
    stream = io.StringIO()
    stats = pstats.Stats(profiler, stream=stream)
    stats.sort_stats('time')
    stats.print_stats(5)
    print(stream.getvalue())


def challenge_3_profile_recursion():
    print("Challenge 3: Profile recursive vs iterative Fibonacci")

    def fib_recursive(n):
        if n < 2:
            return n
        return fib_recursive(n - 1) + fib_recursive(n - 2)

    def fib_iterative(n):
        if n < 2:
            return n
        a, b = 0, 1
        for _ in range(n - 1):
            a, b = b, a + b
        return b

    profiler = cProfile.Profile()
    profiler.enable()
    fib_recursive(20)
    fib_iterative(20)
    profiler.disable()
    stream = io.StringIO()
    stats = pstats.Stats(profiler, stream=stream)
    stats.sort_stats('cumulative')
    stats.print_stats(5)
    print(stream.getvalue())


def challenge_4_profile_memory():
    print("Challenge 4: Use memory_profiler to track memory")
    print("Run: python -m memory_profiler this_script.py")
    print("Add @profile decorator to track memory")


challenge_1_find_slowest_function()
challenge_2_optimize_with_profiling()
challenge_3_profile_recursion()
challenge_4_profile_memory()
```

## Summary

Profiling is essential for writing performant Python code. Tools like cProfile, pstats, snakeviz, timeit, line_profiler, and memory_profiler help identify bottlenecks in time and memory. Always profile before optimizing, use appropriate sort keys, visualize with snakeviz, and automate profiling where possible.

## Related Topics

- Memory Management (92_memory_management.md)
- CPython Internals (93_cpython_internals.md)
- Big O Notation (99_big_o.md)
- Algorithms (98_algorithms.md)
- Data Structures (97_data_structures.md)

