# Algorithms - Sorting, searching, recursion, dynamic programming

## Introduction
Algorithms are step-by-step procedures for solving computational problems. In Python, you can implement algorithms directly or use built-in functions (`sorted`, `bisect`, `min`, `max`) that are optimised in C. Understanding the core algorithm families — sorting, searching, recursion, and dynamic programming — enables you to write efficient code that scales to large datasets.

## Sorting Algorithms

### What It Is
Sorting arranges data in a specific order (typically ascending or descending). Python's built-in `list.sort()` and `sorted()` use Timsort, a hybrid stable sort with O(n log n) worst-case and O(n) best-case performance on partially sorted data.

### Why It Is Important
Sorting is a prerequisite for many efficient algorithms: binary search (requires sorted data), merge joins in databases, deduplication, and top-k selection. Python's Timsort is highly optimised, but understanding its behaviour helps you avoid anti-patterns (e.g., sorting with a custom key function that is slow).

### How It Works Internally
Timsort is a hybrid of merge sort and insertion sort:
1. **Run detection**: scans the array for natural runs (already sorted subsequences).
2. **Insertion sort**: builds small runs (default min-run = 32–64).
3. **Merge**: merges runs using a stack; ensures that merged runs are approximately equal in size using a galloping mode.
4. **Galloping**: when one run's elements consistently win, it switches to binary search to locate the insertion point in bulk.

### Syntax
```python
lst = [5, 2, 8, 1, 9]
lst.sort()                     # in-place, O(n log n)
sorted_lst = sorted(lst)       # returns new list

lst.sort(reverse=True)
lst.sort(key=lambda x: x % 3)  # sort by custom key
lst.sort(key=len)              # sort strings by length

# Stability: equal elements preserve original order
pairs = [(1, 'a'), (2, 'b'), (1, 'c')]
pairs.sort(key=lambda x: x[0])  # (1, 'a') before (1, 'c')
```

### Beginner Examples
```python
# Built-in sort
numbers = [3, 1, 4, 1, 5, 9, 2, 6]
numbers.sort()
print(numbers)

# Sorting with key
words = ['banana', 'apple', 'cherry', 'date']
words.sort(key=len)
print(words)  # date, apple, banana, cherry

# Custom sorting with lambda
students = [
    {'name': 'Alice', 'grade': 90},
    {'name': 'Bob', 'grade': 85},
    {'name': 'Charlie', 'grade': 95},
]
students.sort(key=lambda s: s['grade'], reverse=True)
```

### Intermediate Examples
```python
# Manual quicksort (for learning)
def quicksort(arr):
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    return quicksort(left) + middle + quicksort(right)

# Mergesort implementation
def mergesort(arr):
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left = mergesort(arr[:mid])
    right = mergesort(arr[mid:])
    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result

# Sorting with operator.attrgetter
from operator import attrgetter

class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    def __repr__(self):
        return f'{self.name}({self.age})'

people = [Person('Alice', 30), Person('Bob', 25), Person('Charlie', 35)]
people.sort(key=attrgetter('age'))
print(people)

# Partial sort with heapq.nsmallest
import heapq
data = [random.randint(0, 1000) for _ in range(10000)]
top_5 = heapq.nsmallest(5, data)
```

### Advanced Examples
```python
# Custom comparison with functools.cmp_to_key
from functools import cmp_to_key

def compare(a, b):
    if a['grade'] != b['grade']:
        return b['grade'] - a['grade']
    return -1 if a['name'] < b['name'] else 1

students.sort(key=cmp_to_key(compare))

# Stable sort with multiple keys
data = [(3, 'c'), (1, 'a'), (3, 'b'), (2, 'a')]
# Python's sort is stable — chain sorts
data.sort(key=lambda x: x[1])  # sort by letter first
data.sort(key=lambda x: x[0])  # then by number (stable preserves letter order)

# Counting sort (O(n+k) for integers)
def counting_sort(arr, max_val):
    counts = [0] * (max_val + 1)
    for x in arr:
        counts[x] += 1
    result = []
    for val, count in enumerate(counts):
        result.extend([val] * count)
    return result

# External sorting (for data too large for memory)
def external_sort(file_path, chunk_size=10000):
    import tempfile
    import os

    chunks = []
    with open(file_path) as f:
        while True:
            chunk = []
            for _ in range(chunk_size):
                line = f.readline()
                if not line:
                    break
                chunk.append(int(line.strip()))
            if not chunk:
                break
            chunk.sort()
            tmp = tempfile.NamedTemporaryFile(delete=False, mode='w')
            for val in chunk:
                tmp.write(f'{val}\n')
            tmp.close()
            chunks.append(tmp.name)

    # Merge chunks
    import heapq
    handles = [open(f) for f in chunks]
    merged = heapq.merge(*(int(h.readline().strip()) for h in handles))
    with open('sorted_output.txt', 'w') as out:
        for val in merged:
            out.write(f'{val}\n')
    for h in handles:
        h.close()
    for f in chunks:
        os.unlink(f)
```

### Real-World Use Cases
- **Database query results**: `ORDER BY` clauses use sorting; Python's Timsort is used for in-memory sorting in SQLite.
- **Data visualisation**: sorting data before plotting ensures meaningful axis ordering.
- **Recommendation engines**: sort items by relevance score before returning top-N results.

### Common Mistakes
- Using `list.sort()` in a generator expression or comprehension — it returns `None`, not the sorted list.
- Sorting with a `key` function that is O(n) itself (e.g., `key=expensive_function`) — the key is called O(n log n) times.
- Expecting `sorted()` to work on iterators — it needs a sequence; convert to list first.

### Best Practices
- Use `list.sort()` instead of `sorted()` when you don't need the original list — it avoids a copy.
- Precompute key values and use `key` argument rather than `cmp` (removed in Python 3).
- Use `heapq.nsmallest` or `heapq.nlargest` for top-k instead of full sort.
- Use `bisect.insort` to maintain a sorted list incrementally.

### Performance Considerations
- Timsort: O(n log n) worst, O(n) best (already sorted), O(1) extra space.
- Quicksort: O(n log n) average, O(n^2) worst (pivot choice), O(log n) space.
- Mergesort: O(n log n) all cases, O(n) extra space.
- Counting/radix sort: O(n + k) for integers with bounded range, O(k) space.

### Interview Questions
- **Q**: Why is Timsort stable and why does that matter?  
  **A**: Stability means equal elements retain their original order. This allows multi-key sorting by chaining sorts (e.g., sort by name, then by grade).
- **Q**: When would you use `heapq.nsmallest` instead of `sorted(lst)[:k]`?  
  **A**: When k is much smaller than n. `nsmallest` is O(n log k) vs `sorted` O(n log n).

### Coding Challenges
- Implement a custom sort that groups elements by parity (even numbers first, then odd numbers), preserving relative order within groups.
- Write a function that sorts a list of strings by the number of vowels, breaking ties alphabetically.

### Related Topics
- [Searching algorithms](#searching-algorithms)
- [Recursion](#recursion)
- [Dynamic programming](#dynamic-programming)
- [Big O Notation](#big-o-notation)

---

## Searching Algorithms

### What It Is
Searching algorithms find the position of a target value within a collection. The two fundamental approaches are linear search (O(n) on unsorted data) and binary search (O(log n) on sorted data). Python provides `bisect` module for binary search and `list.index` for linear search.

### Why It Is Important
Searching is the most common operation in programming. Choosing the right search algorithm can reduce time from seconds to microseconds. Binary search on sorted data is exponentially faster than linear search for large collections.

### How It Works Internally
**Binary search** (bisect): starts in the middle of a sorted array, compares the target to the middle element, and eliminates the half that cannot contain the target. It repeats until the target is found or the interval is empty.

**Linear search** (`list.index`): iterates through the list from the first element, comparing each to the target. Returns the index of the first match.

### Syntax
```python
import bisect

# Binary search on sorted lists
index = bisect.bisect_left(sorted_lst, target)   # first >= target
index = bisect.bisect_right(sorted_lst, target)  # first > target
bisect.insort(sorted_lst, value)                  # insert preserving order

# Linear search
lst.index(target)          # raises ValueError if not found
target in lst              # returns bool

# Custom search
min(lst), max(lst)         # O(n)
```

### Beginner Examples
```python
import bisect

# Binary search
sorted_numbers = [1, 3, 5, 7, 9, 11, 13]
pos = bisect.bisect_left(sorted_numbers, 7)
print(f'7 found at index {pos}')

# Linear search
fruits = ['apple', 'banana', 'cherry']
print(fruits.index('banana'))  # 1

# Using bisect for nearest value
def find_closest(sorted_lst, target):
    pos = bisect.bisect_left(sorted_lst, target)
    if pos == 0:
        return sorted_lst[0]
    if pos == len(sorted_lst):
        return sorted_lst[-1]
    before = sorted_lst[pos - 1]
    after = sorted_lst[pos]
    return before if (target - before) < (after - target) else after
```

### Intermediate Examples
```python
import bisect
import time

# Performance comparison
n = 10000000
data = list(range(n))
target = n // 2

# Linear search
t0 = time.perf_counter()
_ = target in data
t1 = time.perf_counter()

# Binary search
import bisect
t2 = time.perf_counter()
_ = bisect.bisect_left(data, target)
t3 = time.perf_counter()

print(f'Linear: {t1-t0:.4f}s, Binary: {t3-t2:.6f}s')

# Exponential search (for unbounded/infinite lists)
def exponential_search(arr, target):
    if arr[0] == target:
        return 0
    i = 1
    while i < len(arr) and arr[i] <= target:
        i *= 2
    return bisect.bisect_left(arr, target, i // 2, min(i, len(arr)))

# Interpolation search (for uniformly distributed data)
def interpolation_search(arr, target):
    low, high = 0, len(arr) - 1
    while low <= high and arr[low] <= target <= arr[high]:
        if arr[high] == arr[low]:
            return low if arr[low] == target else -1
        pos = low + ((target - arr[low]) * (high - low)) // (arr[high] - arr[low])
        if arr[pos] == target:
            return pos
        if arr[pos] < target:
            low = pos + 1
        else:
            high = pos - 1
    return -1
```

### Advanced Examples
```python
import bisect
import numpy as np

# Ternary search (for unimodal functions)
def ternary_search(f, left, right, precision=1e-6):
    while right - left > precision:
        m1 = left + (right - left) / 3
        m2 = right - (right - left) / 3
        if f(m1) < f(m2):
            left = m1
        else:
            right = m2
    return (left + right) / 2

# Search in rotated sorted array
def search_rotated(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if nums[mid] == target:
            return mid
        if nums[lo] <= nums[mid]:
            if nums[lo] <= target < nums[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        else:
            if nums[mid] < target <= nums[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    return -1

# Fuzzy search with tolerance
def fuzzy_search(sorted_arr, target, tolerance=5):
    pos = bisect.bisect_left(sorted_arr, target)
    candidates = []
    for i in range(max(0, pos - tolerance), min(len(sorted_arr), pos + tolerance)):
        if abs(sorted_arr[i] - target) <= tolerance:
            candidates.append(i)
    return candidates

# Searching with custom comparator (bisect doesn't support key directly)
# Workaround: search on a list of transformed values
class KeyedList:
    def __init__(self, lst, key_func):
        self._keys = [key_func(x) for x in lst]
        self._lst = lst

    def find(self, target_key):
        pos = bisect.bisect_left(self._keys, target_key)
        if pos < len(self._keys) and self._keys[pos] == target_key:
            return pos
        return -1

    def __getitem__(self, idx):
        return self._lst[idx]
```

### Real-World Use Cases
- **Autocomplete**: binary search on a sorted dictionary of words.
- **Database indexing**: B-tree search is O(log n) — the generalisation of binary search.
- **Machine learning**: nearest-neighbour search (often accelerated with KD-trees or ball trees).

### Common Mistakes
- Using binary search on unsorted data — produces undefined results (usually -1 or wrong index).
- Forgetting that `bisect_left` returns the insertion point even if the target is not found.
- Using `list.index` in a hot loop — O(n) each call; consider building a dict for O(1) lookups.

### Best Practices
- Always keep data sorted if you need repeated searches.
- Use `bisect` for binary search on sorted lists; it's implemented in C and faster than manual loops.
- For membership queries, use `set` or `dict` (O(1) average) instead of searching a list.
- For very large data that doesn't fit in memory, use an external search index (e.g., SQLite B-tree).

### Performance Considerations
- Linear search: O(n) — acceptable for n < 1000.
- Binary search: O(log n) — optimal for sorted static data.
- Exponential search: O(log i) where i is the target's position — useful for unbounded arrays.
- Interpolation search: O(log log n) average for uniform data, O(n) worst.

### Interview Questions
- **Q**: When would you use exponential search instead of binary search?  
  **A**: When the array is unbounded (e.g., a data stream) or when the target is likely near the beginning.
- **Q**: How does `bisect_left` differ from `bisect_right` when the target exists?  
  **A**: `bisect_left` returns the index of the first occurrence; `bisect_right` returns the index after the last occurrence.

### Coding Challenges
- Implement a search function that finds the peak in a bitonic array (first increases, then decreases) in O(log n).
- Write a function that searches a sorted matrix (rows and columns sorted) for a target value in O(m + n).

### Related Topics
- [Sorting algorithms](#sorting-algorithms)
- [Recursion](#recursion)
- [Big O Notation](#big-o-notation)

---

## Recursion

### What It Is
Recursion is a technique where a function calls itself to solve a smaller instance of the same problem. Every recursive function must have a base case (termination condition) and a recursive case that reduces the problem size. Python's recursion limit defaults to 1000.

### Why It Is Important
Recursion provides elegant solutions for problems with a naturally recursive structure: tree traversal, divide-and-conquer algorithms, backtracking, and mathematical sequences (factorial, Fibonacci). Many sorting and searching algorithms (quicksort, mergesort, binary search) are naturally expressed recursively.

### How It Works Internally
Each recursive call pushes a new frame onto the call stack. The frame contains local variables, the return address, and the function arguments. When the base case is reached, frames are popped from the stack in reverse order (last-in, first-out). Deep recursion can overflow the stack (`RecursionError`).

### Syntax
```python
def recursive_func(n):
    if base_case(n):        # base case
        return base_value
    return recursive_func(smaller_n)  # recursive case

# Python recursion limit
import sys
sys.getrecursionlimit()  # default 1000
sys.setrecursionlimit(5000)  # increase (with caution)
```

### Beginner Examples
```python
# Factorial
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

# Fibonacci (naive — exponential)
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)

# Sum of list
def sum_list(lst):
    if not lst:
        return 0
    return lst[0] + sum_list(lst[1:])

# Power function
def power(base, exp):
    if exp == 0:
        return 1
    return base * power(base, exp - 1)
```

### Intermediate Examples
```python
# Binary search (recursive)
def binary_search(arr, target, lo=0, hi=None):
    if hi is None:
        hi = len(arr) - 1
    if lo > hi:
        return -1
    mid = (lo + hi) // 2
    if arr[mid] == target:
        return mid
    if arr[mid] > target:
        return binary_search(arr, target, lo, mid - 1)
    return binary_search(arr, target, mid + 1, hi)

# Tree traversal
class TreeNode:
    def __init__(self, val, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def inorder(node):
    if node is None:
        return []
    return inorder(node.left) + [node.val] + inorder(node.right)

# Permutations (backtracking)
def permutations(nums):
    result = []
    def backtrack(path, remaining):
        if not remaining:
            result.append(path[:])
            return
        for i, val in enumerate(remaining):
            path.append(val)
            backtrack(path, remaining[:i] + remaining[i+1:])
            path.pop()
    backtrack([], nums)
    return result

# Tower of Hanoi
def hanoi(n, source, target, auxiliary):
    if n == 1:
        print(f'Move disk 1 from {source} to {target}')
        return
    hanoi(n - 1, source, auxiliary, target)
    print(f'Move disk {n} from {source} to {target}')
    hanoi(n - 1, auxiliary, target, source)
```

### Advanced Examples
```python
import sys
from functools import lru_cache

# Tail recursion optimisation via trampoline
def trampoline(f):
    def trampolined(*args):
        result = f(*args)
        while callable(result):
            result = result()
        return result
    return trampolined

@trampoline
def factorial_tail(n, acc=1):
    if n <= 1:
        return acc
    return lambda: factorial_tail(n - 1, n * acc)

# Mutual recursion (even/odd)
def is_even(n):
    if n == 0:
        return True
    return is_odd(n - 1)

def is_odd(n):
    if n == 0:
        return False
    return is_even(n - 1)

# Convert recursion to iteration with explicit stack
def inorder_iterative(root):
    result = []
    stack = []
    curr = root
    while stack or curr:
        while curr:
            stack.append(curr)
            curr = curr.left
        curr = stack.pop()
        result.append(curr.val)
        curr = curr.right
    return result

# Memoised recursion
@lru_cache(maxsize=None)
def edit_distance(s1, s2):
    if not s1:
        return len(s2)
    if not s2:
        return len(s1)
    if s1[0] == s2[0]:
        return edit_distance(s1[1:], s2[1:])
    return 1 + min(
        edit_distance(s1[1:], s2),      # delete
        edit_distance(s1, s2[1:]),      # insert
        edit_distance(s1[1:], s2[1:]),  # replace
    )

# Ackermann function (deep recursion)
def ackermann(m, n):
    if m == 0:
        return n + 1
    if n == 0:
        return ackermann(m - 1, 1)
    return ackermann(m - 1, ackermann(m, n - 1))
```

### Real-World Use Cases
- **Filesystem traversal**: recursively list all files in a directory tree (`os.walk` is iterative but recursive mental model).
- **JSON processing**: recursively parse/transform nested JSON structures.
- **Web crawling**: recursively follow links to a configurable depth.
- **Compiler design**: recursive descent parsers for expression grammars.

### Common Mistakes
- Forgetting the base case — infinite recursion until `RecursionError`.
- Using recursion when an iterative solution is simpler and safer (no stack limit).
- Not increasing `sys.setrecursionlimit` for deep recursion — default 1000 is easily exceeded.
- Creating a new list at each recursive step (`lst[1:]`) — O(n^2) memory and time.

### Best Practices
- Use recursion when the problem is naturally recursive (trees, divide-and-conquer).
- Use memoization (`@lru_cache`) to avoid exponential blowup in recursive computations.
- Convert deep recursion to iteration when the depth could exceed 1000.
- Prefer iterative solutions for linear problems (sum, factorial) — they are faster and safer.

### Performance Considerations
- Each recursive call has overhead: frame allocation, argument copying, stack management.
- Python does not have tail-call optimisation (TCO) — tail-recursive functions still consume stack frames.
- Recursive functions with `@lru_cache` can be much faster than iterative solutions for certain problems (Fibonacci: O(n) vs O(2^n)).
- Iterative conversion using an explicit stack (list or `collections.deque`) is often faster than recursion.

### Interview Questions
- **Q**: Why doesn't Python have tail-call optimisation?  
  **A**: Guido van Rossum argued that TCO would complicate stack traces and encourage a programming style that is less readable. Python prioritises debuggability over tail-call optimisation.
- **Q**: How would you convert a recursive function to an iterative one?  
  **A**: Replace the call stack with an explicit stack data structure (list). Push frames instead of calling recursively, and pop them in a while loop.

### Coding Challenges
- Implement a recursive solution for the N-Queens puzzle, then convert it to iterative with an explicit stack.
- Write a function that computes the depth of a nested dictionary structure using recursion.

### Related Topics
- [Dynamic programming](#dynamic-programming)
- [Sorting algorithms](#sorting-algorithms)
- [Searching algorithms](#searching-algorithms)

---

## Dynamic Programming

### What It Is
Dynamic programming (DP) solves problems by breaking them into overlapping subproblems, solving each subproblem once, and storing the results. It is essentially recursion + memoisation. DP is used for optimisation problems where a brute-force search would be exponential.

### Why It Is Important
DP transforms exponential-time problems into polynomial time. Classic examples: Fibonacci (O(2^n) -> O(n)), knapsack (O(2^n) -> O(n*W)), longest common subsequence (O(2^n) -> O(m*n)), and edit distance. Many coding interview problems are DP-based.

### How It Works Internally
DP has two main approaches:
1. **Top-down (memoisation)**: recursive with a cache dictionary. Compute subproblems on demand.
2. **Bottom-up (tabulation)**: iterative, filling a table from the smallest subproblems upward.

Both store intermediate results in an array (or dict) keyed by the problem state. The state typically consists of indices, remaining capacity, or other parameters that describe the subproblem.

### Syntax
```python
from functools import lru_cache

# Top-down DP
@lru_cache(maxsize=None)
def dp(state):
    if base_case(state):
        return base_value
    return combine(dp(next_state1), dp(next_state2))

# Bottom-up DP
def dp_bottom_up(n):
    dp_table = [0] * (n + 1)
    dp_table[0] = base_value
    for i in range(1, n + 1):
        dp_table[i] = combine(dp_table[i-1], dp_table[i-2])
    return dp_table[n]
```

### Beginner Examples
```python
from functools import lru_cache

# Fibonacci (top-down)
@lru_cache(maxsize=None)
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)

# Fibonacci (bottom-up)
def fib_bu(n):
    if n <= 1:
        return n
    dp = [0] * (n + 1)
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    return dp[n]

# Climbing stairs
@lru_cache(maxsize=None)
def climb_stairs(n):
    if n <= 2:
        return n
    return climb_stairs(n - 1) + climb_stairs(n - 2)

# Minimum path sum (grid)
def min_path_sum(grid):
    m, n = len(grid), len(grid[0])
    dp = [[0] * n for _ in range(m)]
    dp[0][0] = grid[0][0]
    for i in range(1, m):
        dp[i][0] = dp[i-1][0] + grid[i][0]
    for j in range(1, n):
        dp[0][j] = dp[0][j-1] + grid[0][j]
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + grid[i][j]
    return dp[m-1][n-1]
```

### Intermediate Examples
```python
from functools import lru_cache

# 0/1 Knapsack (top-down)
@lru_cache(maxsize=None)
def knapsack(capacity, i):
    if i == 0 or capacity == 0:
        return 0
    if weights[i-1] > capacity:
        return knapsack(capacity, i-1)
    return max(
        values[i-1] + knapsack(capacity - weights[i-1], i-1),
        knapsack(capacity, i-1)
    )

# 0/1 Knapsack (bottom-up)
def knapsack_bu(capacity, weights, values):
    n = len(weights)
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]
    for i in range(1, n + 1):
        for w in range(1, capacity + 1):
            if weights[i-1] <= w:
                dp[i][w] = max(
                    values[i-1] + dp[i-1][w - weights[i-1]],
                    dp[i-1][w]
                )
            else:
                dp[i][w] = dp[i-1][w]
    return dp[n][capacity]

# Longest Common Subsequence
@lru_cache(maxsize=None)
def lcs(i, j):
    if i == 0 or j == 0:
        return 0
    if text1[i-1] == text2[j-1]:
        return 1 + lcs(i-1, j-1)
    return max(lcs(i-1, j), lcs(i, j-1))

# Coin change (minimum coins)
@lru_cache(maxsize=None)
def coin_change(amount):
    if amount == 0:
        return 0
    if amount < 0:
        return float('inf')
    return 1 + min(coin_change(amount - coin) for coin in coins)
```

### Advanced Examples
```python
from functools import lru_cache

# Edit Distance (Levenshtein)
@lru_cache(maxsize=None)
def edit_distance(i, j):
    if i == 0:
        return j
    if j == 0:
        return i
    if word1[i-1] == word2[j-1]:
        return edit_distance(i-1, j-1)
    return 1 + min(
        edit_distance(i-1, j),    # delete
        edit_distance(i, j-1),    # insert
        edit_distance(i-1, j-1),  # replace
    )

# Longest Increasing Subsequence
def lis(nums):
    import bisect
    tails = []
    for x in nums:
        i = bisect.bisect_left(tails, x)
        if i == len(tails):
            tails.append(x)
        else:
            tails[i] = x
    return len(tails)

# DP with state compression (space optimisation)
def fib_compressed(n):
    if n <= 1:
        return n
    a, b = 0, 1
    for _ in range(2, n + 1):
        a, b = b, a + b
    return b

# Palindrome partitioning
@lru_cache(maxsize=None)
def min_cut(s):
    if s == s[::-1]:
        return 0
    cuts = float('inf')
    for i in range(1, len(s)):
        if s[:i] == s[:i][::-1]:
            cuts = min(cuts, 1 + min_cut(s[i:]))
    return cuts

# DP with bitmask (Travelling Salesman)
def tsp(dist):
    n = len(dist)
    @lru_cache(maxsize=None)
    def dp(mask, pos):
        if mask == (1 << n) - 1:
            return dist[pos][0] or float('inf')
        best = float('inf')
        for nxt in range(n):
            if not mask & (1 << nxt):
                cost = dist[pos][nxt] + dp(mask | (1 << nxt), nxt)
                best = min(best, cost)
        return best
    return dp(1, 0)
```

### Real-World Use Cases
- **Bioinformatics**: sequence alignment (edit distance, Smith-Waterman).
- **Navigation**: shortest path algorithms (Dijkstra, Bellman-Ford) are DP-based.
- **Resource allocation**: knapsack for budget-constrained project selection.
- **Natural language processing**: Viterbi algorithm for part-of-speech tagging (DP on HMMs).

### Common Mistakes
- Using top-down DP without memoization — degenerates to brute-force.
- Using bottom-up DP with the wrong iteration order (e.g., forgetting that some problems need reversed loops for 1D space optimisation).
- Defining the state too narrowly or too broadly — state explosion (too many parameters) or missing necessary information.
- Forgetting to handle base cases (index 0, empty string, etc.).

### Best Practices
- Start with the recursive brute-force solution, then add memoization (top-down).
- Convert to bottom-up when you understand the state transition clearly.
- Optimise space by keeping only the necessary rows/columns (state compression).
- Draw the DP table on paper for small inputs to verify the recurrence.

### Performance Considerations
- Top-down: O(number of states * transition cost). Cache hit is O(1).
- Bottom-up: same time complexity, often slightly faster due to iterative overhead vs function call overhead.
- Space: O(number of states) typically; can be reduced to O(min(states)) with compression.
- Python's `@lru_cache` adds ~100 ns per call — negligible for DP with moderate state counts.

### Interview Questions
- **Q**: What is the difference between top-down and bottom-up DP?  
  **A**: Top-down is recursive with memoisation — it computes only needed subproblems. Bottom-up iteratively fills a table — it computes all subproblems but avoids recursion overhead.
- **Q**: When would you use state compression in DP?  
  **A**: When the DP recurrence for position `i` depends only on `i-1` (or a fixed number of previous states), you can keep a rolling array instead of the full table.

### Coding Challenges
- Solve the "Longest Palindromic Substring" problem using DP (O(n^2) time, O(n^2) space), then optimise to O(n) with Manacher's algorithm.
- Implement the "Wildcard Matching" DP (`?` matches any char, `*` matches any sequence) for pattern matching.

### Related Topics
- [Recursion](#recursion)
- [Sorting algorithms](#sorting-algorithms)
- [Big O Notation](#big-o-notation)
- [Caching](#caching---lru-cache-redis-memoization-cache-invalidation-strategies)
