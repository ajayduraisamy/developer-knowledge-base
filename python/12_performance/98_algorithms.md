# Algorithms - Sorting, searching, recursion, dynamic programming

## Introduction

Algorithms are step-by-step procedures for solving computational problems. Python provides powerful built-in algorithms like sorted (Timsort), bisect (binary search), and heapq (priority queue), as well as supporting various algorithmic paradigms including divide and conquer, dynamic programming, and greedy algorithms.

## Why It Is Important

Understanding algorithms is fundamental to writing efficient code. The right algorithm can reduce time complexity from exponential to polynomial, or from O(n^2) to O(n log n). Algorithmic knowledge is essential for technical interviews and building scalable systems.

## Syntax

`python
# Sorting
sorted_list = sorted([3, 1, 4, 1, 5])  # Returns new list
[3, 1, 4, 1, 5].sort()  # In-place sort
sorted(list, key=lambda x: x[1], reverse=True)  # Custom sort

# Binary search
import bisect
pos = bisect.bisect_left(sorted_list, value)
pos = bisect.bisect_right(sorted_list, value)
bisect.insort(sorted_list, value)  # Insert maintaining sort

# Priority queue
import heapq
heap = [3, 1, 4]
heapq.heapify(heap)
smallest = heapq.heappop(heap)
heapq.heappush(heap, 2)

# Recursion
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)
`

## Examples

`python
import time
import random
import bisect
import heapq
from typing import List, Dict, Any, Optional


def timsort_demo():
    print("Timsort (Python's sorting algorithm):")
    data = [random.randint(0, 1000) for _ in range(10000)]
    start = time.perf_counter()
    sorted_data = sorted(data)
    print(f"Sorted 10000 elements in {time.perf_counter() - start:.6f}s")
    print(f"Timsort is O(n log n) in worst case")
    print(f"Timsort is O(n) for nearly sorted data")


timsort_demo()


def binary_search_demo():
    print("\nBinary search with bisect:")
    sorted_list = sorted([random.randint(0, 100) for _ in range(100)])
    target = sorted_list[len(sorted_list) // 2]
    pos = bisect.bisect_left(sorted_list, target)
    print(f"List: {sorted_list[:10]}...{sorted_list[-10:]}")
    print(f"Target {target} at index {pos}")
    print(f"bisect_left: O(log n) search")


binary_search_demo()
`

## Beginner Examples

`python
import time
import random
import bisect
from typing import List


def sorting_basics():
    print("Sorting in Python:")

    data = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]

    sorted_copy = sorted(data)
    print(f"sorted(): {sorted_copy}")

    data.sort()
    print(f"list.sort(): {data}")

    words = ["apple", "Banana", "cherry", "Date"]
    words_sorted = sorted(words, key=str.lower)
    print(f"Case-insensitive sort: {words_sorted}")

    students = [("Alice", 85), ("Bob", 75), ("Charlie", 95)]
    by_grade = sorted(students, key=lambda s: s[1], reverse=True)
    print(f"By grade descending: {by_grade}")


sorting_basics()


def searching_basics():
    print("\nSearching in Python:")

    def linear_search(arr, target):
        for i, val in enumerate(arr):
            if val == target:
                return i
        return -1

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

    data = sorted([1, 3, 5, 7, 9, 11, 13, 15])
    target = 7
    print(f"Binary search for {target} in {data}: index {binary_search(data, target)}")
    print(f"bisect equivalent: {bisect.bisect_left(data, target)}")


searching_basics()


def recursion_vs_iteration():
    print("\nRecursion vs Iteration:")

    def factorial_recursive(n):
        if n <= 1:
            return 1
        return n * factorial_recursive(n - 1)

    def factorial_iterative(n):
        result = 1
        for i in range(2, n + 1):
            result *= i
        return result

    n = 20
    start = time.perf_counter()
    fact_r = factorial_recursive(n)
    t1 = time.perf_counter() - start

    start = time.perf_counter()
    fact_i = factorial_iterative(n)
    t2 = time.perf_counter() - start

    print(f"Recursive: {fact_r}, Time: {t1:.6f}s")
    print(f"Iterative: {fact_i}, Time: {t2:.6f}s")


recursion_vs_iteration()
`

## Intermediate Examples

`python
import time
import random
from typing import List, Dict, Any, Tuple


def divide_and_conquer():
    print("Divide and Conquer: Merge Sort")

    def merge_sort(arr):
        if len(arr) <= 1:
            return arr
        mid = len(arr) // 2
        left = merge_sort(arr[:mid])
        right = merge_sort(arr[mid:])
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

    data = [random.randint(0, 1000) for _ in range(1000)]
    start = time.perf_counter()
    sorted_data = merge_sort(data)
    t = time.perf_counter() - start
    print(f"Merge sort 1000 items: {t:.6f}s")
    print(f"First 5: {sorted_data[:5]}")
    print(f"Last 5: {sorted_data[-5:]}")


divide_and_conquer()


def dynamic_programming():
    print("\nDynamic Programming: Fibonacci")

    def fib_dp(n):
        if n < 2:
            return n
        dp = [0] * (n + 1)
        dp[1] = 1
        for i in range(2, n + 1):
            dp[i] = dp[i - 1] + dp[i - 2]
        return dp[n]

    def fib_optimized(n):
        if n < 2:
            return n
        a, b = 0, 1
        for _ in range(n - 1):
            a, b = b, a + b
        return b

    for n in [10, 30, 50]:
        print(f"Fibonacci({n}) = {fib_optimized(n)}")


dynamic_programming()


def greedy_algorithm():
    print("\nGreedy: Coin Change")

    def min_coins_greedy(amount, coins):
        coins.sort(reverse=True)
        count = 0
        for coin in coins:
            while amount >= coin:
                amount -= coin
                count += 1
        return count

    amount = 67
    coins = [1, 5, 10, 25]
    result = min_coins_greedy(amount, coins)
    print(f"Minimum coins for {amount} cents: {result}")
    print(f"(Quarters, dimes, nickels, pennies)")


greedy_algorithm()
`

## Advanced Examples

`python
import time
import random
from collections import deque
from typing import List, Dict, Any, Optional, Set, Tuple


class GraphAlgorithms:
    def __init__(self):
        pass

    def bfs(self, graph: Dict[int, List[int]], start: int) -> List[int]:
        visited = set()
        queue = deque([start])
        visited.add(start)
        result = []
        while queue:
            vertex = queue.popleft()
            result.append(vertex)
            for neighbor in graph.get(vertex, []):
                if neighbor not in visited:
                    visited.add(neighbor)
                    queue.append(neighbor)
        return result

    def dfs(self, graph: Dict[int, List[int]], start: int) -> List[int]:
        visited = set()
        result = []

        def dfs_recursive(vertex):
            visited.add(vertex)
            result.append(vertex)
            for neighbor in graph.get(vertex, []):
                if neighbor not in visited:
                    dfs_recursive(neighbor)

        dfs_recursive(start)
        return result

    def topological_sort(self, graph: Dict[int, List[int]]) -> List[int]:
        visited = set()
        result = []

        def dfs(vertex):
            visited.add(vertex)
            for neighbor in graph.get(vertex, []):
                if neighbor not in visited:
                    dfs(neighbor)
            result.insert(0, vertex)

        for vertex in graph:
            if vertex not in visited:
                dfs(vertex)
        return result

    def dijkstra(self, graph: Dict[int, List[Tuple[int, int]]], start: int) -> Dict[int, float]:
        distances = {vertex: float('inf') for vertex in graph}
        distances[start] = 0
        pq = [(0, start)]
        while pq:
            current_dist, current = heapq.heappop(pq)
            if current_dist > distances[current]:
                continue
            for neighbor, weight in graph.get(current, []):
                distance = current_dist + weight
                if distance < distances[neighbor]:
                    distances[neighbor] = distance
                    heapq.heappush(pq, (distance, neighbor))
        return distances


import heapq
graph = GraphAlgorithms()
adj_list = {
    0: [1, 2],
    1: [2, 3],
    2: [3],
    3: [4],
    4: []
}
print(f"BFS from 0: {graph.bfs(adj_list, 0)}")
print(f"DFS from 0: {graph.dfs(adj_list, 0)}")

weighted_graph = {
    0: [(1, 4), (2, 1)],
    1: [(3, 1)],
    2: [(1, 2), (3, 5)],
    3: []
}
print(f"Dijkstra from 0: {graph.dijkstra(weighted_graph, 0)}")


def knapsack_01():
    print("\n0/1 Knapsack (Dynamic Programming):")

    def knapsack(values, weights, capacity):
        n = len(values)
        dp = [[0] * (capacity + 1) for _ in range(n + 1)]
        for i in range(1, n + 1):
            for w in range(1, capacity + 1):
                if weights[i - 1] <= w:
                    dp[i][w] = max(
                        values[i - 1] + dp[i - 1][w - weights[i - 1]],
                        dp[i - 1][w]
                    )
                else:
                    dp[i][w] = dp[i - 1][w]
        return dp[n][capacity]

    values = [60, 100, 120]
    weights = [10, 20, 30]
    capacity = 50
    result = knapsack(values, weights, capacity)
    print(f"Maximum value: {result}")


knapsack_01()
`

## Real-World Use Cases

`python
import time
import heapq
import bisect
from typing import List, Dict, Any, Tuple


def search_engine_ranking():
    print("Real-world: Search Engine Ranking with Heap")
    print("Top K results from millions of documents")
    print("Use heapq.nlargest() for O(n log k) complexity")


def ride_sharing_matching():
    print("Real-world: Ride Sharing Matching (BFS/Dijkstra)")
    print("Find nearest drivers using BFS on grid")
    print("Dijkstra for shortest path considering traffic")


def ecommerce_recommendations():
    print("Real-world: E-commerce Recommendations")
    print("Collaborative filtering with matrix operations")
    print("Sorting products by relevance score")


def network_routing():
    print("Real-world: Network Routing (Dijkstra/Bellman-Ford)")
    print("OSPF routing protocol uses Dijkstra")
    print("BGP uses path vector protocol")


def compression_algorithms():
    print("Real-world: Data Compression")
    print("Huffman coding (greedy) for text compression")
    print("LZ77/LZ78 for dictionary-based compression")


search_engine_ranking()
ride_sharing_matching()
ecommerce_recommendations()
network_routing()
compression_algorithms()
`

## Common Mistakes

`python
from typing import List


def mistake_1_wrong_sort_stability():
    print("Mistake 1: Not understanding sort stability")
    print("Python's sort is stable - equal elements keep order")
    print("Use this for multi-key sorting")


def mistake_2_recursion_depth():
    print("Mistake 2: Exceeding recursion depth")
    print("Default recursion limit is 1000")
    print("Use iteration or sys.setrecursionlimit()")


def mistake_3_mutable_default_args():
    print("Mistake 3: Mutable default arguments in algorithms")
    print("Default mutable arguments persist across calls")


def mistake_4_not_using_heapq():
    print("Mistake 4: Using list.sort() repeatedly for priority")
    print("Use heapq for O(log n) push/pop instead of O(n log n)")


def mistake_5_forgetting_base_case():
    print("Mistake 5: Forgetting base case in recursion")
    print("Leads to infinite recursion and stack overflow")


def mistake_6_off_by_one():
    print("Mistake 6: Off-by-one errors in binary search")
    print("Be careful with left <= right vs left < right")


mistake_1_wrong_sort_stability()
mistake_2_recursion_depth()
mistake_3_mutable_default_args()
mistake_4_not_using_heapq()
mistake_5_forgetting_base_case()
mistake_6_off_by_one()
`

## Best Practices

`python
import time
import random
from typing import List


def best_practice_1_use_builtins():
    print("Best Practice 1: Use built-in algorithms")
    print("sorted(), min(), max(), sum() are C-optimized")
    print("Faster than manual implementations")


def best_practice_2_choose_right_algorithm():
    print("Best Practice 2: Choose algorithm by data size")
    print("Small n: simple algorithms may be faster")
    print("Large n: use O(n log n) or better")


def best_practice_3_consider_memory():
    print("Best Practice 3: Consider space complexity")
    print("In-place algorithms save memory")
    print("Sometimes memory is more important than speed")


def best_practice_4_profile_before_optimizing():
    print("Best Practice 4: Profile before optimizing")
    print("The bottleneck may not be where you think")


def best_practice_5_use_generators():
    print("Best Practice 5: Use generators for large sequences")
    print("They process one item at a time")
    print("Avoid creating huge intermediate lists")


def best_practice_6_use_functools():
    print("Best Practice 6: Use functools for memoization")
    print("@lru_cache for dynamic programming")
    print("Automatically caches recursive results")


best_practice_1_use_builtins()
best_practice_2_choose_right_algorithm()
best_practice_3_consider_memory()
best_practice_4_profile_before_optimizing()
best_practice_5_use_generators()
best_practice_6_use_functools()
`

## Interview Questions

`python
def interview_q1():
    print("Q: What sorting algorithm does Python use?")
    print("A: Timsort, a hybrid of merge sort and insertion sort.")


def interview_q2():
    print("Q: What is the time complexity of sorted()?")
    print("A: O(n log n) worst case, O(n) for nearly sorted.")


def interview_q3():
    print("Q: When would you use BFS vs DFS?")
    print("A: BFS for shortest path in unweighted graph.")
    print("   DFS for topological sort, cycle detection.")


def interview_q4():
    print("Q: What is dynamic programming?")
    print("A: Solving problems by breaking into overlapping subproblems.")
    print("   Uses memoization or tabulation.")


def interview_q5():
    print("Q: What is the greedy algorithm property?")
    print("A: Makes locally optimal choice at each step.")
    print("   Works when local optimum leads to global optimum.")


def interview_q6():
    print("Q: How does Dijkstra's algorithm work?")
    print("A: Uses priority queue, processes nodes by shortest distance.")
    print("   O((V + E) log V) with binary heap.")


def interview_q7():
    print("Q: What is the difference between merge sort and quicksort?")
    print("A: Merge sort: O(n log n) always, O(n) space.")
    print("   Quicksort: O(n log n) average, O(1) space.")


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
from typing import List, Dict, Tuple, Optional
from collections import deque
import heapq


def challenge_1_two_sum():
    print("Challenge 1: Two Sum (find two numbers that add to target)")

    def two_sum(nums: List[int], target: int) -> Tuple[int, int]:
        seen = {}
        for i, num in enumerate(nums):
            complement = target - num
            if complement in seen:
                return (seen[complement], i)
            seen[num] = i
        return (-1, -1)

    nums = [2, 7, 11, 15]
    target = 9
    print(f"Two sum for {target} in {nums}: {two_sum(nums, target)}")


def challenge_2_max_subarray():
    print("Challenge 2: Maximum Subarray (Kadane's Algorithm)")

    def max_subarray(nums: List[int]) -> int:
        max_ending_here = max_so_far = nums[0]
        for num in nums[1:]:
            max_ending_here = max(num, max_ending_here + num)
            max_so_far = max(max_so_far, max_ending_here)
        return max_so_far

    nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
    print(f"Maximum subarray sum: {max_subarray(nums)}")


def challenge_3_lca_bst():
    print("Challenge 3: Lowest Common Ancestor in BST")

    class TreeNode:
        def __init__(self, val=0, left=None, right=None):
            self.val = val
            self.left = left
            self.right = right

    def lowest_common_ancestor(root: TreeNode, p: TreeNode, q: TreeNode) -> TreeNode:
        while root:
            if p.val < root.val and q.val < root.val:
                root = root.left
            elif p.val > root.val and q.val > root.val:
                root = root.right
            else:
                return root
        return root

    root = TreeNode(6)
    root.left = TreeNode(2)
    root.right = TreeNode(8)
    root.left.left = TreeNode(0)
    root.left.right = TreeNode(4)
    result = lowest_common_ancestor(root, root.left, root.left.right)
    print(f"LCA of 2 and 4: {result.val}")


def challenge_4_merge_k_lists():
    print("Challenge 4: Merge K Sorted Lists")

    def merge_k_lists(lists: List[List[int]]) -> List[int]:
        heap = []
        for i, lst in enumerate(lists):
            if lst:
                heapq.heappush(heap, (lst[0], i, 0))
        result = []
        while heap:
            val, list_idx, elem_idx = heapq.heappop(heap)
            result.append(val)
            if elem_idx + 1 < len(lists[list_idx]):
                next_val = lists[list_idx][elem_idx + 1]
                heapq.heappush(heap, (next_val, list_idx, elem_idx + 1))
        return result

    lists = [[1, 4, 7], [2, 5, 8], [3, 6, 9]]
    merged = merge_k_lists(lists)
    print(f"Merged: {merged}")


challenge_1_two_sum()
challenge_2_max_subarray()
challenge_3_lca_bst()
challenge_4_merge_k_lists()
`

## Summary

Algorithms are essential for efficient problem-solving. Python's built-in algorithms (Timsort, bisect, heapq) are highly optimized. Key paradigms include divide and conquer, dynamic programming, greedy algorithms, and graph algorithms (BFS, DFS, Dijkstra). Understanding time complexity helps choose the right approach.

## Related Topics

- Big O Notation (99_big_o.md)
- Data Structures (97_data_structures.md)
- Profiling (91_profiling.md)
- Caching (96_caching.md)
