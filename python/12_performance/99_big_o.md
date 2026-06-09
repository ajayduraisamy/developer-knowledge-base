# Big O Notation - Time complexity, space complexity, algorithm analysis

## Introduction

Big O notation describes the upper bound of an algorithm's time or space complexity as the input size grows. It provides a standardized way to analyze and compare algorithm efficiency, independent of hardware or implementation details.

## Why It Is Important

Big O notation helps developers predict how algorithms scale with input size, choose the most efficient approach, and communicate about performance with other developers. Understanding Big O is essential for technical interviews and building scalable systems.

## Syntax

`python
# O(1) - Constant time
def get_first(arr):
    return arr[0] if arr else None

# O(log n) - Logarithmic time
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1

# O(n) - Linear time
def linear_search(arr, target):
    for i, val in enumerate(arr):
        if val == target:
            return i
    return -1

# O(n log n) - Log-linear time
def sort_and_search(arr):
    arr.sort()  # O(n log n)
    return arr

# O(n^2) - Quadratic time
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        for j in range(n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]

# O(2^n) - Exponential time
def fibonacci_recursive(n):
    if n < 2:
        return n
    return fibonacci_recursive(n - 1) + fibonacci_recursive(n - 2)
`

## Examples

`python
import time
import random
from typing import List


def demonstrate_constant_time():
    print("O(1) - Constant Time:")
    data = [1, 2, 3, 4, 5]
    print(f"  Get first: {data[0]}")
    print(f"  Get last: {data[-1]}")
    print(f"  Length: {len(data)}")
    print("  These operations take the same time regardless of size")


demonstrate_constant_time()


def demonstrate_linear_time():
    print("\nO(n) - Linear Time:")
    for n in [1000, 10000, 100000]:
        data = list(range(n))
        start = time.perf_counter()
        total = sum(data)
        t = time.perf_counter() - start
        print(f"  n={n:6d}: {t:.6f}s (expect 10x increase)")


demonstrate_linear_time()


def demonstrate_quadratic_time():
    print("\nO(n^2) - Quadratic Time:")

    def bubble_sort(arr):
        n = len(arr)
        for i in range(n):
            for j in range(n - i - 1):
                if arr[j] > arr[j + 1]:
                    arr[j], arr[j + 1] = arr[j + 1], arr[j]

    for n in [100, 200, 400]:
        data = [random.randint(0, 1000) for _ in range(n)]
        start = time.perf_counter()
        bubble_sort(data)
        t = time.perf_counter() - start
        print(f"  n={n:4d}: {t:.6f}s (expect 4x increase for 2x n)")


demonstrate_quadratic_time()
`

## Beginner Examples

`python
import time
from typing import List


def big_o_basics():
    print("Big O Notation Basics:")
    print("  O(1): Constant - array index, dict lookup")
    print("  O(log n): Logarithmic - binary search")
    print("  O(n): Linear - simple loop, linear search")
    print("  O(n log n): Log-linear - sorting (Timsort)")
    print("  O(n^2): Quadratic - nested loops")
    print("  O(2^n): Exponential - recursive Fibonacci")
    print("  O(n!): Factorial - traveling salesman (brute force)")


big_o_basics()


def analyze_loop_complexity():
    print("\nAnalyzing Loop Complexity:")

    def example1(n):
        for i in range(n):
            print(i, end=' ')
        print("  -> O(n)")

    def example2(n):
        for i in range(n):
            for j in range(n):
                print(i, j, end=' ')
        print("  -> O(n^2)")

    def example3(n):
        i = 1
        while i < n:
            print(i, end=' ')
            i *= 2
        print("  -> O(log n)")


analyze_loop_complexity()


def common_misconceptions():
    print("\nCommon Misconceptions:")
    print("  1. O(n) doesn't mean 'slower than O(1)' for small n")
    print("  2. Constants are dropped: O(2n) = O(n)")
    print("  3. Only the fastest-growing term matters")
    print("  4. Best case is rarely as important as worst case")
    print("  5. Space complexity matters too")


common_misconceptions()
`

## Intermediate Examples

`python
import time
import random
from typing import List, Dict, Any


def analyze_nlogn_vs_n_squared():
    print("O(n log n) vs O(n^2) Comparison:")

    for n in [100, 1000, 5000]:
        data = [random.randint(0, 10000) for _ in range(n)]

        start = time.perf_counter()
        sorted_data = sorted(data)
        t_nlogn = time.perf_counter() - start

        def bubble_sort(arr):
            n = len(arr)
            for i in range(n):
                for j in range(n - i - 1):
                    if arr[j] > arr[j + 1]:
                        arr[j], arr[j + 1] = arr[j + 1], arr[j]

        data_copy = data[:]
        start = time.perf_counter()
        bubble_sort(data_copy)
        t_nsq = time.perf_counter() - start

        print(f"  n={n:5d}: O(n log n)={t_nlogn:.6f}s, O(n^2)={t_nsq:.6f}s, ratio={t_nsq/t_nlogn:.1f}x")


analyze_nlogn_vs_n_squared()


def analyze_time_complexity():
    print("\nTime Complexity Analysis:")

    def find_duplicates_naive(arr):
        for i in range(len(arr)):
            for j in range(i + 1, len(arr)):
                if arr[i] == arr[j]:
                    return arr[i]
        return None

    def find_duplicates_fast(arr):
        seen = set()
        for item in arr:
            if item in seen:
                return item
            seen.add(item)
        return None

    for n in [100, 1000, 5000]:
        data = [random.randint(0, n // 2) for _ in range(n)]

        start = time.perf_counter()
        find_duplicates_naive(data)
        t1 = time.perf_counter() - start

        start = time.perf_counter()
        find_duplicates_fast(data)
        t2 = time.perf_counter() - start

        print(f"  n={n:5d}: O(n^2)={t1:.6f}s, O(n)={t2:.6f}s, speedup={t1/t2:.0f}x")


analyze_time_complexity()


def ammortized_analysis():
    print("\nAmortized Analysis:")
    print("  list.append() is O(1) amortized")
    print("  dict.insert() is O(1) amortized")
    print("  Python list overallocates to make appends O(1)")
    print("  Occasional O(n) resize is 'paid for' by O(1) appends")


ammortized_analysis()
`

## Advanced Examples

`python
import time
import random
import sys
from typing import List, Dict, Any


class ComplexityAnalyzer:
    def __init__(self):
        self.results: List[Dict[str, Any]] = []

    def measure(self, name: str, func, sizes: List[int], *args, **kwargs):
        for n in sizes:
            start = time.perf_counter()
            func(n, *args, **kwargs)
            elapsed = time.perf_counter() - start
            self.results.append({'name': name, 'n': n, 'time': elapsed})
            print(f"  {name}: n={n:6d}, time={elapsed:.6f}s")

    def compare(self, name1: str, name2: str):
        r1 = [r for r in self.results if r['name'] == name1]
        r2 = [r for r in self.results if r['name'] == name2]
        if r1 and r2:
            ratio = r2[-1]['time'] / r1[-1]['time']
            print(f"\n  Ratio ({name2}/{name1}) for largest n: {ratio:.1f}x")


analyzer = ComplexityAnalyzer()


def o_n_example(n):
    total = 0
    for i in range(n):
        total += i
    return total


def o_n2_example(n):
    total = 0
    for i in range(n):
        for j in range(n):
            total += i * j
    return total


def o_nlogn_example(n):
    data = [random.randint(0, 1000) for _ in range(n)]
    return sorted(data)


print("Complexity comparison:")
analyzer.measure("O(n)", o_n_example, [1000, 10000, 100000])
analyzer.measure("O(n log n)", o_nlogn_example, [1000, 10000, 100000])
analyzer.measure("O(n^2)", o_n2_example, [100, 500, 1000])


def space_complexity():
    print("\nSpace Complexity Analysis:")

    def create_list(n):
        return [0] * n  # O(n) space

    def create_matrix(n):
        return [[0] * n for _ in range(n)]  # O(n^2) space

    for n in [100, 1000, 10000]:
        lst = create_list(n)
        print(f"  List of {n}: {sys.getsizeof(lst) + n * 28} bytes (approx)")

    print("\n  Space complexity examples:")
    print("    O(1): In-place operations")
    print("    O(n): Copying an array")
    print("    O(n^2): 2D matrix")


space_complexity()
`

## Real-World Use Cases

`python
import time
from typing import List, Dict


def database_indexing():
    print("Real-world: Database Indexing (O(log n))")
    print("  B-tree indexes enable O(log n) lookups")
    print("  vs. full table scan O(n)")
    print("  Critical for large tables")


def search_algorithms():
    print("Real-world: Search Engines")
    print("  Inverted index: O(1) term lookup")
    print("  Ranking: O(n log n) sorting of results")
    print("  Web graph: PageRank O(V + E)")


def caching_strategies():
    print("Real-world: Caching Systems")
    print("  LRU cache: O(1) get/put with hash table + linked list")
    print("  Redis: O(log n) for sorted sets")


def compression_tradeoffs():
    print("Real-world: Compression Trade-offs")
    print("  gzip: O(n) compression, good ratio")
    print("  lz4: O(n) very fast, lower ratio")
    print("  Trade-off between compression time and ratio")


def machine_learning():
    print("Real-world: Machine Learning Training")
    print("  Linear regression: O(n^3) matrix inversion")
    print("  SVM: O(n^2) to O(n^3)")
    print("  k-NN: O(n) prediction (no training)")


database_indexing()
search_algorithms()
caching_strategies()
compression_tradeoffs()
machine_learning()
`

## Common Mistakes

`python
from typing import List


def mistake_1_dropping_wrong_terms():
    print("Mistake 1: Dropping the wrong terms")
    print("  O(n + n^2) -> O(n^2) (correct)")
    print("  O(n + m) -> O(n + m) (can't drop if n != m)")


def mistake_2_confusing_best_worst():
    print("Mistake 2: Confusing best, average, worst case")
    print("  Quick sort: O(n log n) average, O(n^2) worst")
    print("  Usually we care about worst case")


def mistake_3_ignoring_constants():
    print("Mistake 3: Ignoring constants for small n")
    print("  O(n) with C=1000 may be slower than O(n^2) with C=1")
    print("  For small n, constants matter!")


def mistake_4_space_complexity_ignored():
    print("Mistake 4: Ignoring space complexity")
    print("  An O(1) space solution may be better than O(n)")
    print("  Even if time complexity is the same")


def mistake_5_not_considering_input():
    print("Mistake 5: Not considering input characteristics")
    print("  Nearly sorted data: Timsort is O(n)")
    print("  Random data: Timsort is O(n log n)")


def mistake_6_amortized_analysis():
    print("Mistake 6: Not understanding amortized vs worst-case")
    print("  list.append() is O(1) amortized, O(n) worst-case")
    print("  But the worst case is rare and predictable")


mistake_1_dropping_wrong_terms()
mistake_2_confusing_best_worst()
mistake_3_ignoring_constants()
mistake_4_space_complexity_ignored()
mistake_5_not_considering_input()
mistake_6_amortized_analysis()
`

## Best Practices

`python
from typing import List, Any


def best_practice_1_profile_to_verify():
    print("Best Practice 1: Profile to verify complexity")
    print("  Theoretical complexity != actual performance")
    print("  Profile with different input sizes")


def best_practice_2_consider_input_size():
    print("Best Practice 2: Choose algorithm by input size")
    print("  n < 100: Simple O(n^2) may be fastest")
    print("  n > 10000: Need O(n log n) or better")


def best_practice_3_analyze_bottlenecks():
    print("Best Practice 3: Analyze the dominant term")
    print("  Find the part with highest complexity")
    print("  That's where optimization matters most")


def best_practice_4_trade_off_time_for_space():
    print("Best Practice 4: Time-space trade-off")
    print("  Use hash tables (more memory) for O(1) lookup")
    print("  Use sorting (more time) for O(1) space")


def best_practice_5_use_builtins():
    print("Best Practice 5: Use optimized built-in algorithms")
    print("  sorted() is C-optimized Timsort")
    print("  dict/set lookups are O(1) implemented in C")


def best_practice_6_think_in_big_o():
    print("Best Practice 6: Always think in Big O")
    print("  Before coding, estimate complexity")
    print("  After coding, verify with timing")


best_practice_1_profile_to_verify()
best_practice_2_consider_input_size()
best_practice_3_analyze_bottlenecks()
best_practice_4_trade_off_time_for_space()
best_practice_5_use_builtins()
best_practice_6_think_in_big_o()
`

## Interview Questions

`python
def interview_q1():
    print("Q: What is Big O notation?")
    print("A: Describes scaling behavior of algorithms.")
    print("   Upper bound of time/space as input grows.")


def interview_q2():
    print("Q: What is O(log n) and give an example?")
    print("A: Logarithmic growth. Binary search divides input.")
    print("   Doubling input adds only 1 more operation.")


def interview_q3():
    print("Q: What is amortized analysis?")
    print("A: Average cost per operation over a sequence.")
    print("   list.append() is O(1) amortized.")


def interview_q4():
    print("Q: How do you compare O(n) and O(n log n)?")
    print("A: O(n) is faster, but both are 'efficient'.")
    print("   At n=1M, O(n) = 1M ops, O(n log n) = 20M ops.")


def interview_q5():
    print("Q: What is the complexity of dict.get()?")
    print("A: O(1) average case, O(n) worst case (hash collisions).")


def interview_q6():
    print("Q: How do you find the complexity of recursive functions?")
    print("A: Master Theorem or recurrence relation analysis.")
    print("   Fibonacci: T(n) = T(n-1) + T(n-2) + 1 -> O(2^n)")


def interview_q7():
    print("Q: What is space complexity?")
    print("A: Additional memory an algorithm needs.")
    print("   In-place sort: O(1) space. Merge sort: O(n).")


interview_q1()
interview_q2()
interview_q3()
interview_q4()
interview_q5()
interview_q6()
interview_q7()
`

## Coding Challenges

`python
import time
import random
from typing import List


def challenge_1_identify_complexity():
    print("Challenge 1: Identify the Big O of these functions")

    def func1(n):
        for i in range(n):
            print(i)
    print("  func1: O(n)")

    def func2(n):
        for i in range(n):
            for j in range(n):
                print(i, j)
    print("  func2: O(n^2)")

    def func3(n):
        i = 1
        while i < n:
            print(i)
            i *= 2
    print("  func3: O(log n)")

    def func4(n):
        for i in range(n):
            j = 1
            while j < n:
                print(i, j)
                j *= 2
    print("  func4: O(n log n)")


def challenge_2_empirical_analysis():
    print("Challenge 2: Empirically measure complexity")

    def unknown_algorithm(n):
        total = 0
        for i in range(n):
            for j in range(i):
                total += i * j
        return total

    for n in [100, 200, 400, 800]:
        start = time.perf_counter()
        unknown_algorithm(n)
        t = time.perf_counter() - start
        print(f"  n={n:4d}: {t:.6f}s")

    print("  When n doubles, time quadruples -> O(n^2)")


def challenge_3_optimize_to_better_complexity():
    print("Challenge 3: Optimize from O(n^2) to O(n)")

    def has_duplicate_naive(arr):
        for i in range(len(arr)):
            for j in range(i + 1, len(arr)):
                if arr[i] == arr[j]:
                    return True
        return False

    def has_duplicate_fast(arr):
        seen = set()
        for x in arr:
            if x in seen:
                return True
            seen.add(x)
        return False

    data = [random.randint(0, 10000) for _ in range(5000)]
    start = time.perf_counter()
    has_duplicate_naive(data)
    t1 = time.perf_counter() - start

    start = time.perf_counter()
    has_duplicate_fast(data)
    t2 = time.perf_counter() - start
    print(f"  O(n^2): {t1:.4f}s, O(n): {t2:.4f}s, speedup: {t1/t2:.0f}x")


def challenge_4_space_time_tradeoff():
    print("Challenge 4: Space-time trade-off analysis")

    def two_sum_naive(nums, target):
        for i in range(len(nums)):
            for j in range(i + 1, len(nums)):
                if nums[i] + nums[j] == target:
                    return [i, j]
        return [-1, -1]

    def two_sum_fast(nums, target):
        seen = {}
        for i, num in enumerate(nums):
            complement = target - num
            if complement in seen:
                return [seen[complement], i]
            seen[num] = i
        return [-1, -1]

    nums = [random.randint(0, 10000) for _ in range(1000)]
    target = 9999

    start = time.perf_counter()
    result1 = two_sum_naive(nums, target)
    t1 = time.perf_counter() - start

    start = time.perf_counter()
    result2 = two_sum_fast(nums, target)
    t2 = time.perf_counter() - start

    print(f"  O(n^2) no extra space: {t1:.6f}s -> {result1}")
    print(f"  O(n) with O(n) space: {t2:.6f}s -> {result2}")
    print(f"  Speedup: {t1/t2:.0f}x, Extra space: O(n)")


challenge_1_identify_complexity()
challenge_2_empirical_analysis()
challenge_3_optimize_to_better_complexity()
challenge_4_space_time_tradeoff()
`

## Summary

Big O notation is essential for analyzing and comparing algorithm efficiency. Key complexities range from O(1) (constant) to O(n!) (factorial). Understanding time and space complexity trade-offs helps developers choose appropriate algorithms, optimize bottlenecks, and build scalable systems.

## Related Topics

- Algorithms (98_algorithms.md)
- Data Structures (97_data_structures.md)
- Profiling (91_profiling.md)
- CPython Internals (93_cpython_internals.md)
