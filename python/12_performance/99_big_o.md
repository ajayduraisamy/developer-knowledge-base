# Big O Notation - Time complexity, space complexity, algorithm analysis

## Introduction
Big O notation is the language we use to describe the efficiency of algorithms. It expresses how the runtime or memory requirements of an algorithm grow as the input size increases, ignoring constants and lower-order terms. Mastery of Big O is essential for choosing the right data structures, designing scalable systems, and performing well in technical interviews.

## Time Complexity

### What It Is
Time complexity measures the amount of time an algorithm takes as a function of the input size `n`. It is expressed as an asymptotic upper bound — the worst-case scenario. Common time complexities, from fastest to slowest:

- **O(1)**: constant — array index, dict/set lookup
- **O(log n)**: logarithmic — binary search, balanced tree operations
- **O(n)**: linear — single loop, linear search
- **O(n log n)**: linearithmic — sorting (Timsort), divide-and-conquer
- **O(n^2)**: quadratic — nested loops, bubble sort
- **O(2^n)**: exponential — naive recursion (Fibonacci)
- **O(n!)**: factorial — generating all permutations

### Why It Is Important
Time complexity predicts how an algorithm will scale. An O(n^2) algorithm that runs in 1 ms for 100 items will take ~100 seconds for 10,000 items. An O(n log n) algorithm for the same data takes ~0.1 seconds. Understanding complexity prevents performance disasters in production.

### How It Works Internally
Big O analysis counts the number of primitive operations (comparisons, arithmetic, assignments) as a function of `n`. The dominant term determines the complexity:
- `3n^2 + 5n + 2` → O(n^2) (drops constants and lower-order terms)
- `1000n + 50000` → O(n) (constant factor doesn't matter asymptotically)
- `2^n + n^5` → O(2^n) (exponential dominates polynomial)

### Syntax
```python
# O(1) — constant time
def get_first(lst):
    return lst[0]

# O(n) — linear time
def linear_sum(lst):
    total = 0
    for x in lst:       # n iterations
        total += x
    return total

# O(n^2) — quadratic time
def pairwise_sum(lst):
    result = []
    for i in range(len(lst)):      # n
        for j in range(len(lst)):  # n
            result.append(lst[i] + lst[j])
    return result

# O(log n) — logarithmic time
def binary_search(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo <= hi:                # log2(n) iterations
        mid = (lo + hi) // 2
        if arr[mid] == target:
            return mid
        if arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```

### Beginner Examples
```python
import time

# Demonstrating O(n) vs O(n^2)
def linear(n):
    total = 0
    for i in range(n):
        total += i
    return total

def quadratic(n):
    total = 0
    for i in range(n):
        for j in range(n):
            total += i * j
    return total

for n in [1000, 2000, 4000]:
    t0 = time.perf_counter(); linear(n); t1 = time.perf_counter()
    t2 = time.perf_counter(); quadratic(n); t3 = time.perf_counter()
    print(f'n={n:5d}: O(n)={t1-t0:.5f}s, O(n^2)={t3-t2:.5f}s')

# Constant time operations
import sys
d = {i: i**2 for i in range(1000)}
t0 = time.perf_counter()
_ = d[500]
t1 = time.perf_counter()
print(f'Dict lookup (O(1)): {t1-t0:.8f}s')
```

### Intermediate Examples
```python
# Analysing nested loops
def example1(n):
    for i in range(n):          # O(n)
        print(i)

def example2(n):
    for i in range(n):          # O(n)
        for j in range(n):      # O(n)
            print(i, j)         # O(n^2)

def example3(n):
    for i in range(n):          # O(n)
        for j in range(i, n):   # O(n) — n + (n-1) + ... + 1 = n(n+1)/2
            print(i, j)         # O(n^2)

def example4(n):
    i = 1
    while i < n:                # O(log n)
        i *= 2
        print(i)

def example5(n):
    for i in range(n):          # O(n log n)
        j = 1
        while j < n:
            j *= 2
            print(i, j)

# Amortised analysis: list.append
def amortised_demo():
    lst = []
    for i in range(1000000):
        lst.append(i)           # O(1) amortised

# Best, average, worst case
def linear_search(arr, target):
    for i, x in enumerate(arr):
        if x == target:         # Best: O(1), Worst: O(n), Avg: O(n)
            return i
    return -1
```

### Advanced Examples
```python
# Master Theorem for divide-and-conquer
# T(n) = aT(n/b) + f(n)
# Examples:
# Merge Sort: T(n) = 2T(n/2) + O(n) -> O(n log n)
# Binary Search: T(n) = T(n/2) + O(1) -> O(log n)

# Empirical complexity measurement
import time
import numpy as np

def measure_complexity(func, inputs):
    times = []
    for n in inputs:
        data = list(range(n))
        t0 = time.perf_counter()
        func(data)
        t1 = time.perf_counter()
        times.append(t1 - t0)

    # Fit to n, n^2, n log n
    n = np.array(inputs)
    t = np.array(times)

    # Estimate exponent
    log_n = np.log(n)
    log_t = np.log(t)
    exponent = np.polyfit(log_n, log_t, 1)[0]
    return f'O(n^{exponent:.2f})'

def sort_test(arr):
    return sorted(arr)

def linear_test(arr):
    return sum(arr)

print(measure_complexity(sort_test, [1000, 2000, 4000, 8000, 16000]))
print(measure_complexity(linear_test, [100000, 200000, 400000, 800000]))

# Complexity of Python operations
op_complexity = {
    'list[i]': 'O(1)',
    'list.append': 'O(1) amortised',
    'list.pop()': 'O(1)',
    'list.pop(0)': 'O(n)',
    'list.insert(i, x)': 'O(n)',
    'list.__contains__ (in)': 'O(n)',
    'dict[key]': 'O(1) avg',
    'dict.__contains__': 'O(1) avg',
    'set.__contains__': 'O(1) avg',
    'list.sort': 'O(n log n)',
    'sorted': 'O(n log n)',
    'len()': 'O(1)',
}

# Tight bound analysis
def analyze_function(func, n_values=[10, 50, 100, 500, 1000, 5000]):
    for n in n_values:
        t0 = time.perf_counter()
        func(n)
        t1 = time.perf_counter()
        ratio = t1 - t0
        print(f'n={n:5d}: {ratio:.6f}s')

# Constant factor matters
def constant_factor_demo():
    n = 10_000_000
    data = list(range(n))

    # Both O(n), but different constants
    t0 = time.perf_counter()
    result = [x * 2 for x in data]     # comprehension
    t1 = time.perf_counter()

    result2 = []
    for x in data:                      # manual loop
        result2.append(x * 2)
    t2 = time.perf_counter()

    print(f'Comprehension: {t1-t0:.3f}s, Loop: {t2-t1:.3f}s')
```

### Real-World Use Cases
- **System design interviews**: justify choosing a hash map (O(1) lookup) over a list (O(n)).
- **Database query planning**: the query optimiser estimates the complexity of different join strategies and picks the cheapest.
- **API rate limiting**: ensure the rate limiter itself is O(1) so it doesn't become the bottleneck.

### Common Mistakes
- Ignoring the worst case: an algorithm may be O(n) on average but O(n^2) worst-case (e.g., naive quicksort with bad pivot).
- Assuming O(1) operations are always fast — a dict lookup with a slow `__hash__` can be expensive.
- Confusing `time` measurements with complexity — a fast computer can make an O(n^2) algorithm look fast for small n.

### Best Practices
- Always consider the worst-case input when stating complexity.
- Profile with realistic input sizes — Big O is an asymptotic guide, not an exact runtime predictor.
- Document the complexity of your functions in docstrings so callers understand the cost.
- Favour O(n log n) over O(n^2) for n > 1000.

### Performance Considerations
- CPU cache effects: an O(n log n) algorithm with good locality can outperform O(n) with random memory access.
- Python overhead: built-in functions (`sorted`, `bisect`) are implemented in C and have much lower constants than equivalent Python loops.
- Memory bandwidth: for large n, moving data from RAM to CPU is often the bottleneck, not the number of operations.

### Interview Questions
- **Q**: What is the time complexity of `list.sort()` in Python?  
  **A**: O(n log n) average and worst case. Python uses Timsort, which is O(n) for already-sorted data.
- **Q**: How do you compute the complexity of a recursive function?  
  **A**: Write a recurrence relation T(n) and apply the Master Theorem: T(n) = aT(n/b) + f(n).

### Coding Challenges
- Write a function that estimates the empirical time complexity of a given function by running it with increasing input sizes and computing the ratio of runtimes.
- Given two functions with complexities O(n^2) and O(n log n), determine the crossover point (the value of n where the O(n^2) becomes slower).

### Related Topics
- [Space complexity](#space-complexity)
- [Algorithm analysis](#algorithm-analysis)
- [Data Structures (Big O)](#data-structures---list-vs-dict-vs-set-performance-big-o-analysis)

---

## Space Complexity

### What It Is
Space complexity measures the amount of memory an algorithm uses as a function of input size `n`. It includes both the input storage and the auxiliary space (temporary variables, recursion stack, dynamic allocations). Like time complexity, it is expressed in Big O notation.

### Why It Is Important
Space complexity determines whether an algorithm can run within available memory. An algorithm with O(n^2) space may run out of memory for large n, even if its time complexity is acceptable. In data-intensive applications (big data, embedded systems), space efficiency is often more critical than time.

### How It Works Internals
Space analysis accounts for:
1. **Input space**: the storage for the input data (usually not counted, as it is required by the problem).
2. **Auxiliary space**: extra memory allocated during computation (the focus of analysis).
3. **Stack space**: memory for the call stack in recursive algorithms.

### Syntax
```python
# O(1) auxiliary space — only a few variables
def constant_space(lst):
    total = 0
    for x in lst:
        total += x
    return total

# O(n) auxiliary space — creates a new list
def linear_space(n):
    return [i ** 2 for i in range(n)]

# O(n^2) auxiliary space — creates a matrix
def quadratic_space(n):
    return [[i * j for j in range(n)] for i in range(n)]

# O(log n) stack space — recursive binary search
def binary_search_recursive(arr, target, lo=0, hi=None):
    if hi is None:
        hi = len(arr) - 1
    if lo > hi:
        return -1
    mid = (lo + hi) // 2
    if arr[mid] == target:
        return mid
    if arr[mid] > target:
        return binary_search_recursive(arr, target, lo, mid - 1)
    return binary_search_recursive(arr, target, mid + 1, hi)
```

### Beginner Examples
```python
import sys

# Measuring memory usage
def measure_space(n):
    # O(1) space
    a = 1
    print(f'O(1): {sys.getsizeof(a)} bytes')

    # O(n) space
    lst = list(range(n))
    print(f'O(n) list of {n}: ~{sys.getsizeof(lst) + n * 8} bytes')

    # O(n) space with dict
    d = {i: i for i in range(n)}
    dict_size = sys.getsizeof(d) + sum(sys.getsizeof(k) + sys.getsizeof(v) for k, v in d.items())
    print(f'O(n) dict of {n}: ~{dict_size} bytes')

measure_space(10000)

# In-place vs out-of-place
def in_place_double(lst):
    for i in range(len(lst)):
        lst[i] *= 2          # O(1) auxiliary space

def out_of_place_double(lst):
    return [x * 2 for x in lst]  # O(n) auxiliary space
```

### Intermediate Examples
```python
# Space-time tradeoff: precomputation
# Without precomputation: O(1) space, O(n) time per query
def is_prime_slow(n):
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True

# With precomputation: O(n) space, O(1) time per query
def sieve_of_eratosthenes(n):
    is_prime = [True] * (n + 1)
    is_prime[0] = is_prime[1] = False
    for i in range(2, int(n ** 0.5) + 1):
        if is_prime[i]:
            for j in range(i * i, n + 1, i):
                is_prime[j] = False
    return is_prime

# Recursive space (stack depth)
def factorial_recursive(n):
    if n <= 1:
        return 1
    return n * factorial_recursive(n - 1)  # O(n) stack space

def factorial_iterative(n):
    result = 1
    for i in range(2, n + 1):
        result *= i          # O(1) auxiliary space
    return result

# In-place algorithm: reversing a list
def reverse_in_place(lst):
    i, j = 0, len(lst) - 1
    while i < j:
        lst[i], lst[j] = lst[j], lst[i]
        i += 1
        j -= 1                # O(1) auxiliary space

def reverse_out_of_place(lst):
    return lst[::-1]          # O(n) auxiliary space
```

### Advanced Examples
```python
import sys

# Measuring object sizes recursively
def total_size(obj, seen=None):
    if seen is None:
        seen = set()
    obj_id = id(obj)
    if obj_id in seen:
        return 0
    seen.add(obj_id)
    size = sys.getsizeof(obj)
    if isinstance(obj, (list, tuple)):
        size += sum(total_size(item, seen) for item in obj)
    elif isinstance(obj, dict):
        size += sum(total_size(k, seen) + total_size(v, seen) for k, v in obj.items())
    elif isinstance(obj, set):
        size += sum(total_size(item, seen) for item in obj)
    return size

# Space-efficient generator
def read_large_file(file_path):
    with open(file_path) as f:
        for line in f:          # O(1) space — yields one line at a time
            yield line.strip()

# vs list-based approach
def read_all_lines(file_path):
    with open(file_path) as f:
        return f.readlines()    # O(n) space — loads entire file

# In-place matrix transposition
def transpose_in_place(matrix):
    n = len(matrix)
    for i in range(n):
        for j in range(i + 1, n):
            matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]

# Space-efficient DP (state compression)
def fib_compressed(n):
    if n <= 1:
        return n
    a, b = 0, 1                     # O(1) space instead of O(n)
    for _ in range(2, n + 1):
        a, b = b, a + b
    return b

# Tracking space usage with tracemalloc
import tracemalloc

def trace_allocation():
    tracemalloc.start()
    data = [bytearray(10000) for _ in range(1000)]
    snapshot = tracemalloc.take_snapshot()
    stats = snapshot.statistics('lineno')
    for stat in stats[:5]:
        print(stat)
    tracemalloc.stop()

# Generator as space-efficient alternative to list
def fibonacci_generator(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b            # O(1) space

def fibonacci_list(n):
    result = [0, 1]
    for _ in range(2, n):
        result.append(result[-1] + result[-2])
    return result                   # O(n) space
```

### Real-World Use Cases
- **Mobile/embedded development**: memory is constrained; algorithms with O(1) auxiliary space are preferred.
- **Big data processing**: when processing terabytes of data, O(n) space is infeasible; use streaming algorithms with O(1) or O(log n) space.
- **Database indexing**: B-tree indices use O(n) space for the index but reduce query time from O(n) to O(log n) — a classic space-time tradeoff.

### Common Mistakes
- Counting input storage as auxiliary space (it should be excluded).
- Ignoring recursion stack depth — an O(1) auxiliary algorithm can still overflow the stack.
- Using generators when you need random access — generators are O(1) space but O(1) forward-only iteration.
- Creating unnecessary copies (slicing, copying list comprehensions) that turn O(1) auxiliary into O(n).

### Best Practices
- Prefer in-place algorithms when the original data can be modified.
- Use generators for large data streams instead of building intermediate lists.
- Measure memory with `tracemalloc` or `memory_profiler` for critical paths.
- Document space complexity in function docstrings alongside time complexity.

### Performance Considerations
- Each Python object has overhead: `sys.getsizeof(int)` = 28 bytes, `sys.getsizeof(list)` = 56 bytes + 8 bytes per pointer.
- Space complexity interacts with cache performance: O(n) algorithms with good locality (arrays) are faster than O(n) with random access (linked lists, hash tables).
- Garbage collection can mask space issues — unreferenced objects linger until GC runs.

### Interview Questions
- **Q**: What is the space complexity of a recursive Fibonacci implementation?  
  **A**: O(n) for the call stack (depth of recursion), which is often the limiting factor.
- **Q**: How would you reverse a linked list in O(1) space?  
  **A**: Iterative reversal with three pointers (prev, curr, next) — no additional allocations.

### Coding Challenges
- Implement a function that merges two sorted arrays in-place (O(1) auxiliary space, O(n+m) time).
- Write a space-efficient version of Pascal's triangle that uses O(n) space instead of O(n^2) by computing rows iteratively.

### Related Topics
- [Time complexity](#time-complexity)
- [Algorithm analysis](#algorithm-analysis)
- [Memory Management](#memory-management---garbage-collection-reference-counting-weakref)

---

## Algorithm Analysis

### What It Is
Algorithm analysis is the systematic evaluation of an algorithm's efficiency using time and space complexity. It involves deriving a recurrence relation, classifying the complexity using Big O, and verifying empirically. Analysis guides algorithm selection and identifies optimisation opportunities.

### Why It Is Important
Rigorous analysis separates intuition from evidence. A developer may *feel* that a nested loop is "fast enough" until they analyse it and realise it will be O(n^2) with n=1,000,000. Analysis provides the mathematical foundation for comparing algorithms independent of hardware.

### How It Works Internally
The analysis process:
1. **Identify the input size** `n`.
2. **Count operations**: assignments, comparisons, arithmetic, function calls.
3. **Derive the cost function** `T(n)` in terms of operations.
4. **Simplify to Big O**: drop constants and lower-order terms.
5. **Consider best, average, and worst cases**.
6. **Verify empirically** with benchmarks across multiple input sizes.

### Syntax
```python
# Step-by-step analysis

def analyze_me(n):
    total = 0                    # 1 assignment
    for i in range(n):           # n iterations
        for j in range(n):       # n * n iterations
            total += i * j       # n^2 arithmetic ops
    return total                 # 1 return

# T(n) = 1 + n^2 + 1 = O(n^2)

# Counting comparisons
def count_comparisons(arr, target):
    comparisons = 0
    for x in arr:
        comparisons += 1
        if x == target:
            return True, comparisons
    return False, comparisons

# Amortised analysis: dynamic array resizing
class DynamicArray:
    def __init__(self):
        self._data = []
        self._copies = 0

    def append(self, value):
        old_cap = len(self._data)
        self._data.append(value)
        if len(self._data) > old_cap:  # reallocation happened
            self._copies += old_cap or 1

    def total_copies(self, n):
        for i in range(n):
            self.append(i)
        return self._copies
        # Total copies: 1 + 2 + 4 + ... + n/2 ≈ n-1 = O(n) amortised
```

### Beginner Examples
```python
# Analysis of three sum implementations

def three_sum_brute(arr, target):
    """O(n^3) time, O(1) space."""
    n = len(arr)
    for i in range(n):
        for j in range(i + 1, n):
            for k in range(j + 1, n):
                if arr[i] + arr[j] + arr[k] == target:
                    return (i, j, k)
    return None

def three_sum_better(arr, target):
    """O(n^2) time, O(n) space."""
    seen = {}
    for i, x in enumerate(arr):
        for j, y in enumerate(arr[i+1:], i+1):
            needed = target - x - y
            if needed in seen:
                return (seen[needed], i, j)
            seen[y] = j
    return None

def three_sum_best(arr, target):
    """O(n^2) time, O(1) space (sorted)."""
    arr.sort()
    n = len(arr)
    for i in range(n - 2):
        lo, hi = i + 1, n - 1
        while lo < hi:
            current = arr[i] + arr[lo] + arr[hi]
            if current == target:
                return (i, lo, hi)
            if current < target:
                lo += 1
            else:
                hi -= 1
    return None
```

### Intermediate Examples
```python
# Master Theorem application
# T(n) = aT(n/b) + f(n)
# Case 1: f(n) = O(n^c) where c < log_b(a) -> T(n) = Theta(n^{log_b(a)})
# Case 2: f(n) = Theta(n^c) where c = log_b(a) -> T(n) = Theta(n^c log n)
# Case 3: f(n) = Omega(n^c) where c > log_b(a) -> T(n) = Theta(f(n))

# Example: Merge sort T(n) = 2T(n/2) + O(n)
# a=2, b=2, log_2(2)=1, f(n)=O(n^1) => Case 2 => O(n log n)

# Example: Binary search T(n) = T(n/2) + O(1)
# a=1, b=2, log_2(1)=0, f(n)=O(1) => Case 2 => O(log n)

# Worst-case analysis
def worst_case_linear(arr, target):
    """Worst case: target not found -> O(n)."""
    for x in arr:
        if x == target:
            return True
    return False

# Average-case analysis
def average_case_linear(arr, target):
    """Average case: target at position n/2 -> O(n)."""
    for i, x in enumerate(arr):
        if x == target:
            return i
    return -1

# Comparing growth rates empirically
def compare_growth():
    import time
    for n in [10, 100, 1000]:
        # O(n log n)
        data = list(range(n))
        t0 = time.perf_counter()
        sorted(data)
        t1 = time.perf_counter()

        # O(n^2)
        t2 = time.perf_counter()
        for i in range(n):
            for j in range(n):
                _ = data[i] + data[j]
        t3 = time.perf_counter()

        print(f'n={n:5d}: O(n log n)={t1-t0:.6f}s, O(n^2)={t3-t2:.6f}s, ratio={(t3-t2)/(t1-t0):.1f}')
```

### Advanced Examples
```python
import time
import random
import math

# Empirical complexity verification
def verify_complexity(func, input_generator, expected_exponent, trials=5):
    """
    Verify that func's runtime grows as O(n^expected_exponent).
    """
    sizes = [100, 200, 400, 800, 1600, 3200]
    times = []

    for n in sizes:
        data = input_generator(n)
        t0 = time.perf_counter()
        for _ in range(trials):
            func(data)
        t1 = time.perf_counter()
        times.append((t1 - t0) / trials)

    # Check doubling ratios
    ratios = [times[i+1] / times[i] for i in range(len(times) - 1)]
    expected_ratio = 2 ** expected_exponent
    avg_ratio = sum(ratios) / len(ratios)
    return f'Expected ratio: {expected_ratio:.2f}, Actual: {avg_ratio:.2f}'

# Amortised analysis verification
def verify_amortised():
    n = 1000000
    lst = []
    import sys

    reallocations = 0
    prev_cap = 0

    for i in range(n):
        lst.append(i)
        cap = sys.getsizeof(lst)
        if cap != prev_cap:
            reallocations += 1
            prev_cap = cap

    print(f'n={n}: {reallocations} reallocations')
    # Expected: O(log n) reallocations, total copies O(n)

# Lower bound analysis (Omega)
def lower_bound_demo():
    """Any comparison-based sort requires Omega(n log n) comparisons."""
    # Proof: there are n! permutations; each comparison splits the space in half.
    # Minimum comparisons = ceil(log2(n!)) ~ n log2 n

# Tight bound analysis (Theta)
def tight_bound_demo():
    """When upper bound (O) equals lower bound (Omega), we have Theta."""
    # Example: Merge sort is Theta(n log n) — it's both O(n log n) and Omega(n log n)

# Little-o and little-omega
def little_notation():
    """
    f(n) = o(g(n)): f grows strictly slower than g (e.g., n = o(n^2))
    f(n) = omega(g(n)): f grows strictly faster than g (e.g., n^2 = omega(n))
    f(n) = Theta(g(n)): f grows at the same rate as g
    """

# P vs NP discussion
def complexity_classes():
    """
    P: problems solvable in polynomial time (sorting, searching, shortest path).
    NP: problems verifiable in polynomial time (subset sum, SAT, TSP).
    NP-complete: hardest problems in NP (if any NP-complete problem is in P, then P=NP).
    """
    pass
```

### Real-World Use Cases
- **Code review**: reject PRs with O(n^2) algorithms in hot paths without justification.
- **Library selection**: choose between libraries based on their documented complexity guarantees.
- **Capacity planning**: estimate server requirements based on algorithmic complexity and expected load.
- **Interview preparation**: algorithmic analysis is the core of technical interviews at top tech companies.

### Common Mistakes
- Confusing complexity with actual runtime: an O(n^2) algorithm with a small constant can be faster than O(n log n) for small n.
- Ignoring hidden complexity: `list.copy()`, string concatenation, and implicit loops add hidden O(n) costs.
- Forgetting that Python's `in` operator is O(n) for lists but O(1) for sets/dicts — the single most common analysis error.

### Best Practices
- Analyse complexity before writing code, especially for algorithms that will process large datasets.
- Use empirical measurement to verify your analysis — real-world performance includes constants and caching effects.
- Document the complexity of public functions in docstrings (e.g., `"""O(n log n) time, O(n) space."""`).
- When comparing two algorithms, consider both time and space complexity — they often trade off.

### Performance Considerations
- Complexity analysis assumes uniform cost model; in practice, memory hierarchy (L1/L2 cache, RAM, disk) affects performance.
- Python's interpreter adds a constant factor of ~50x over C, so an O(n log n) algorithm in Python may be slower than an O(n^2) algorithm in C for moderate n.
- Parallelism can change effective complexity: an O(n^2) algorithm running on n processors has O(n) span.

### Interview Questions
- **Q**: What is the difference between Big O, Big Omega, and Big Theta?  
  **A**: Big O is an upper bound (worst case). Big Omega is a lower bound (best case). Big Theta is a tight bound (both upper and lower).
- **Q**: How would you analyse the complexity of a recursive function with two recursive calls?  
  **A**: Write the recurrence (e.g., T(n) = 2T(n/2) + O(n)) and apply the Master Theorem.

### Coding Challenges
- Write a function that takes a list of algorithms and their measured runtimes across different input sizes and determines the empirical Big O of each.
- Implement a profiler decorator that records the number of operations (comparisons, swaps) performed by a sorting function and prints the complexity class.

### Related Topics
- [Time complexity](#time-complexity)
- [Space complexity](#space-complexity)
- [Data Structures (Big O)](#data-structures---list-vs-dict-vs-set-performance-big-o-analysis)
- [Algorithms](#algorithms---sorting-searching-recursion-dynamic-programming)
