# Data Structures - list vs dict vs set performance, Big O analysis

## Introduction

Python provides a rich set of built-in data structures including lists, dicts, sets, and tuples. Each has different time and space complexity characteristics. Understanding these helps developers choose the right data structure for their specific use case.

## Why It Is Important

Choosing the wrong data structure can lead to O(n^2) or worse performance. The right data structure can reduce time complexity from O(n) to O(1) for critical operations. This knowledge is essential for writing scalable and efficient Python code.

## Syntax

`python
# List (dynamic array)
lst = [1, 2, 3]
lst.append(4)
lst.pop()

# Dict (hash table)
d = {'a': 1, 'b': 2}
d['c'] = 3
d.get('a')

# Set (hash table)
s = {1, 2, 3}
s.add(4)
s.remove(1)
1 in s  # O(1)

# Deque (double-ended queue)
from collections import deque
dq = deque([1, 2, 3])
dq.append(4)
dq.appendleft(0)
dq.pop()
dq.popleft()

# Heap (priority queue)
import heapq
heap = [3, 1, 4, 1, 5]
heapq.heapify(heap)
heapq.heappush(heap, 0)
smallest = heapq.heappop(heap)

# Bisect for sorted lists
import bisect
sorted_lst = [1, 3, 5, 7]
pos = bisect.bisect_left(sorted_lst, 4)
bisect.insort(sorted_lst, 4)
`

## Examples

`python
import time
import random
from collections import deque, defaultdict, Counter
import heapq
import bisect
from typing import List, Dict, Any


def list_vs_deque():
    print("List vs Deque performance:")
    n = 100000
    lst = list(range(n))
    dq = deque(range(n))
    start = time.perf_counter()
    for _ in range(1000):
        lst.pop(0)
    print(f"List pop(0): {time.perf_counter() - start:.4f}s")
    start = time.perf_counter()
    for _ in range(1000):
        dq.popleft()
    print(f"Deque popleft(): {time.perf_counter() - start:.4f}s")


list_vs_deque()


def dict_vs_list_search():
    print("\nDict vs List search:")
    n = 100000
    data_list = list(range(n))
    data_dict = {i: i for i in range(n)}
    search_items = random.sample(range(n), 1000)
    start = time.perf_counter()
    for item in search_items:
        item in data_list
    print(f"List search: {time.perf_counter() - start:.4f}s")
    start = time.perf_counter()
    for item in search_items:
        item in data_dict
    print(f"Dict search: {time.perf_counter() - start:.4f}s")


dict_vs_list_search()
`

## Beginner Examples

`python
import time
import random
from collections import deque
from typing import List


def list_basics():
    print("List (dynamic array) time complexities:")
    print("  Index: O(1)")
    print("  Append: O(1) amortized")
    print("  Insert/Delete at beginning: O(n)")
    print("  Insert/Delete at end: O(1)")
    print("  Search: O(n)")
    print("  Sort: O(n log n)")


list_basics()


def dict_basics():
    print("\nDict (hash table) time complexities:")
    print("  Get/Set: O(1) average, O(n) worst")
    print("  Delete: O(1) average")
    print("  Membership test (in): O(1)")
    print("  Iteration: O(n)")
    print("  Memory: more overhead than list")


dict_basics()


def set_basics():
    print("\nSet (hash table) time complexities:")
    print("  Add: O(1) average")
    print("  Remove: O(1) average")
    print("  Membership: O(1) average")
    print("  Union/Intersection: O(min(m, n))")
    print("  Difference: O(n)")


set_basics()


def tuple_basics():
    print("\nTuple (immutable array) time complexities:")
    print("  Index: O(1)")
    print("  Iteration: O(n)")
    print("  Memory: slightly less than list")
    print("  Hashable: can be used as dict key")


tuple_basics()
`

## Intermediate Examples

`python
import time
import random
import heapq
import bisect
from collections import deque, Counter, defaultdict
from typing import List, Dict, Tuple, Optional


def deque_applications():
    print("Deque applications:")
    dq = deque(maxlen=5)
    for i in range(10):
        dq.append(i)
    print(f"Sliding window: {list(dq)}")
    dq = deque()
    dq.extend([1, 2, 3])
    dq.extendleft([0, -1, -2])
    print(f"Extended: {list(dq)}")
    dq.rotate(2)
    print(f"Rotated right 2: {list(dq)}")
    dq.rotate(-2)
    print(f"Rotated left 2: {list(dq)}")


deque_applications()


def heap_applications():
    print("\nHeap (priority queue) applications:")

    def merge_k_sorted_lists(lists: List[List[int]]) -> List[int]:
        heap = []
        result = []
        for i, lst in enumerate(lists):
            if lst:
                heapq.heappush(heap, (lst[0], i, 0))
        while heap:
            val, list_idx, elem_idx = heapq.heappop(heap)
            result.append(val)
            if elem_idx + 1 < len(lists[list_idx]):
                next_val = lists[list_idx][elem_idx + 1]
                heapq.heappush(heap, (next_val, list_idx, elem_idx + 1))
        return result

    lists = [[1, 4, 7], [2, 5, 8], [3, 6, 9]]
    merged = merge_k_sorted_lists(lists)
    print(f"Merged sorted lists: {merged}")
    data = [random.randint(0, 100) for _ in range(10000)]
    heapq.heapify(data)
    largest = heapq.nlargest(5, data)
    smallest = heapq.nsmallest(5, data)
    print(f"5 largest: {largest}")
    print(f"5 smallest: {smallest}")


heap_applications()


def bisect_applications():
    print("\nBisect (binary search on sorted lists) applications:")
    sorted_list = [1, 3, 5, 7, 9, 11, 13]

    def find_closest(arr, target):
        pos = bisect.bisect_left(arr, target)
        if pos == 0:
            return arr[0]
        if pos >= len(arr):
            return arr[-1]
        before = arr[pos - 1]
        after = arr[pos]
        if after - target < target - before:
            return after
        return before

    for target in [0, 4, 8, 14]:
        closest = find_closest(sorted_list, target)
        print(f"Closest to {target}: {closest}")
    grades = [60, 70, 80, 90]

    def letter_grade(score):
        idx = bisect.bisect_right(grades, score)
        return ['F', 'D', 'C', 'B', 'A'][idx]

    for score in [55, 65, 75, 85, 95]:
        print(f"Score {score} -> Grade {letter_grade(score)}")


bisect_applications()
`

## Advanced Examples

`python
import time
import random
import hashlib
from typing import List, Dict, Any, Optional, Set, Tuple
from collections import defaultdict


class BloomFilter:
    def __init__(self, size: int = 1000, num_hashes: int = 3):
        self.size = size
        self.num_hashes = num_hashes
        self.bit_array = [False] * size

    def _hashes(self, item: str) -> List[int]:
        result = []
        for i in range(self.num_hashes):
            hash_input = f"{i}:{item}".encode()
            hash_val = int(hashlib.md5(hash_input).hexdigest(), 16)
            result.append(hash_val % self.size)
        return result

    def add(self, item: str):
        for pos in self._hashes(item):
            self.bit_array[pos] = True

    def __contains__(self, item: str) -> bool:
        return all(self.bit_array[pos] for pos in self._hashes(item))


bf = BloomFilter(size=1000, num_hashes=5)
bf.add("apple")
bf.add("banana")
bf.add("cherry")
print(f"'apple' in bloom filter: {'apple' in bf}")
print(f"'durian' in bloom filter: {'durian' in bf}")
print("(False positives are possible with bloom filters)")


class TrieNode:
    def __init__(self):
        self.children: Dict[str, 'TrieNode'] = {}
        self.is_end = False


class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str):
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end = True

    def search(self, word: str) -> bool:
        node = self.root
        for char in word:
            if char not in node.children:
                return False
            node = node.children[char]
        return node.is_end

    def starts_with(self, prefix: str) -> bool:
        node = self.root
        for char in prefix:
            if char not in node.children:
                return False
            node = node.children[char]
        return True

    def autocomplete(self, prefix: str) -> List[str]:
        node = self.root
        for char in prefix:
            if char not in node.children:
                return []
            node = node.children[char]
        results = []
        self._dfs(node, prefix, results)
        return results

    def _dfs(self, node: TrieNode, path: str, results: List[str]):
        if node.is_end:
            results.append(path)
        for char, child in sorted(node.children.items()):
            self._dfs(child, path + char, results)


trie = Trie()
words = ["cat", "car", "card", "care", "caret", "carrot"]
for w in words:
    trie.insert(w)
print(f"Search 'car': {trie.search('car')}")
print(f"Search 'can': {trie.search('can')}")
print(f"Autocomplete 'car': {trie.autocomplete('car')}")


class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache: Dict[int, int] = {}
        self.order: List[int] = []

    def get(self, key: int) -> int:
        if key in self.cache:
            self.order.remove(key)
            self.order.append(key)
            return self.cache[key]
        return -1

    def put(self, key: int, value: int):
        if key in self.cache:
            self.order.remove(key)
        elif len(self.cache) >= self.capacity:
            oldest = self.order.pop(0)
            del self.cache[oldest]
        self.cache[key] = value
        self.order.append(key)


lru = LRUCache(3)
lru.put(1, 1)
lru.put(2, 2)
lru.put(3, 3)
lru.get(1)
lru.put(4, 4)
print(f"LRU cache: {lru.cache}")
`

## Real-World Use Cases

`python
import time
import heapq
import bisect
from collections import deque, Counter, defaultdict
from typing import List, Dict, Any, Tuple


def web_server_rate_limiter():
    print("Real-world: Rate Limiter with Deque")

    class RateLimiter:
        def __init__(self, max_requests: int, window: float):
            self.max_requests = max_requests
            self.window = window
            self.requests: Dict[str, deque] = defaultdict(deque)

        def allow_request(self, client_id: str) -> bool:
            now = time.time()
            client_requests = self.requests[client_id]
            while client_requests and client_requests[0] < now - self.window:
                client_requests.popleft()
            if len(client_requests) < self.max_requests:
                client_requests.append(now)
                return True
            return False

    limiter = RateLimiter(3, 10.0)
    for _ in range(5):
        allowed = limiter.allow_request("client_1")
        print(f"  Request allowed: {allowed}")
    print(f"  Requests in window: {len(limiter.requests['client_1'])}")


def task_scheduler():
    print("\nReal-world: Task Scheduler with Heap")

    class TaskScheduler:
        def __init__(self):
            self.tasks: List[Tuple[int, int, str]] = []

        def add_task(self, priority: int, task_id: int, name: str):
            heapq.heappush(self.tasks, (priority, task_id, name))

        def get_next(self) -> str:
            if self.tasks:
                priority, task_id, name = heapq.heappop(self.tasks)
                return f"Task: {name} (priority: {priority})"
            return "No tasks"

    scheduler = TaskScheduler()
    scheduler.add_task(3, 1, "Low priority")
    scheduler.add_task(1, 2, "High priority")
    scheduler.add_task(2, 3, "Medium priority")
    print(f"Next: {scheduler.get_next()}")
    print(f"Next: {scheduler.get_next()}")
    print(f"Next: {scheduler.get_next()}")


def autocomplete_system():
    print("\nReal-world: Autocomplete with Trie")
    print("Used in search engines, text editors, etc.")
    print("Trie enables O(k) prefix search (k = prefix length)")


web_server_rate_limiter()
task_scheduler()
autocomplete_system()
`

## Common Mistakes

`python
import time
from typing import List


def mistake_1_list_pop_front():
    print("Mistake 1: Using list.pop(0) instead of deque.popleft()")
    print("list.pop(0) is O(n), deque.popleft() is O(1)")


def mistake_2_list_membership():
    print("Mistake 2: Checking membership with list instead of set")
    print("item in list: O(n), item in set: O(1)")


def mistake_3_dict_key_types():
    print("Mistake 3: Using mutable types as dict keys")
    print("Only hashable types can be keys (not list, dict, set)")


def mistake_4_forgetting_heapify():
    print("Mistake 4: Not calling heapify() on list before heappop")
    print("heapq works on any list, but must be heapified first")


def mistake_5_unsorted_bisect():
    print("Mistake 5: Using bisect on unsorted lists")
    print("bisect assumes the list is sorted, wrong results otherwise")


def mistake_6_list_instead_of_heap():
    print("Mistake 6: Using list for priority queue")
    print("list min/max: O(n), heap push/pop: O(log n)")


mistake_1_list_pop_front()
mistake_2_list_membership()
mistake_3_dict_key_types()
mistake_4_forgetting_heapify()
mistake_5_unsorted_bisect()
mistake_6_list_instead_of_heap()
`

## Best Practices

`python
from collections import deque, defaultdict, Counter
from typing import List, Dict, Any


def best_practice_1_choose_right_structure():
    print("Best Practice 1: Choose based on operation patterns")
    print("  - Frequent inserts/deletes at ends: deque")
    print("  - Frequent membership checks: set")
    print("  - Key-value lookups: dict")
    print("  - Priority queue: heapq")
    print("  - Sorted inserts/searches: bisect")


def best_practice_2_use_counter():
    print("Best Practice 2: Use Counter for counting")
    data = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
    counter = Counter(data)
    print(f"  Counter: {counter}")
    print(f"  Most common: {counter.most_common(2)}")


def best_practice_3_use_defaultdict():
    print("Best Practice 3: Use defaultdict to avoid key errors")

    groups = defaultdict(list)
    for i in range(10):
        groups[i % 3].append(i)
    print(f"  Groups: {dict(groups)}")


def best_practice_4_preallocate_list():
    print("Best Practice 4: Pre-allocate list size when known")
    print("  [0] * n is faster than appending n times")


def best_practice_5_use_sets_for_uniqueness():
    print("Best Practice 5: Use set for uniqueness checks")
    items = [1, 2, 2, 3, 3, 3]
    unique = list(set(items))
    print(f"  Unique: {unique}")


def best_practice_6_ordered_dict():
    print("Best Practice 6: dict preserves insertion order (3.7+)")
    print("  No need for OrderedDict in most cases")


best_practice_1_choose_right_structure()
best_practice_2_use_counter()
best_practice_3_use_defaultdict()
best_practice_4_preallocate_list()
best_practice_5_use_sets_for_uniqueness()
best_practice_6_ordered_dict()
`

## Interview Questions

`python
def interview_q1():
    print("Q: What is the time complexity of list.append()?")
    print("A: O(1) amortized due to overallocation strategy.")


def interview_q2():
    print("Q: How is dict implemented in Python?")
    print("A: Hash table with open addressing and quadratic probing.")


def interview_q3():
    print("Q: What is the difference between list and deque?")
    print("A: list: O(1) append/pop at end. deque: O(1) at both ends.")


def interview_q4():
    print("Q: When would you use heapq vs bisect?")
    print("A: heapq for priority queue (dynamic). bisect for static sorted lists.")


def interview_q5():
    print("Q: What are the downsides of dict?")
    print("A: Higher memory overhead than list, O(n) worst-case lookup.")


def interview_q6():
    print("Q: How does set intersection work?")
    print("A: Iterates over smaller set, checks membership in larger set.")


def interview_q7():
    print("Q: What is a bloom filter good for?")
    print("A: Space-efficient membership test with possible false positives.")


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
from collections import deque, Counter
import heapq
import bisect
from typing import List


def challenge_1_implement_queue():
    print("Challenge 1: Implement a queue using two stacks")

    class QueueWithStacks:
        def __init__(self):
            self.stack_in = []
            self.stack_out = []

        def enqueue(self, x: int):
            self.stack_in.append(x)

        def dequeue(self) -> int:
            if not self.stack_out:
                while self.stack_in:
                    self.stack_out.append(self.stack_in.pop())
            return self.stack_out.pop()

    q = QueueWithStacks()
    for i in range(5):
        q.enqueue(i)
    print(f"Dequeue: {q.dequeue()}")
    print(f"Dequeue: {q.dequeue()}")


def challenge_2_top_k_frequent():
    print("Challenge 2: Find top K frequent elements")

    def top_k_frequent(nums: List[int], k: int) -> List[int]:
        count = Counter(nums)
        return [item for item, _ in count.most_common(k)]

    nums = [1, 1, 1, 2, 2, 3, 3, 3, 3]
    print(f"Top 2: {top_k_frequent(nums, 2)}")


def challenge_3_merge_intervals():
    print("Challenge 3: Merge overlapping intervals")

    def merge_intervals(intervals: List[List[int]]) -> List[List[int]]:
        if not intervals:
            return []
        intervals.sort()
        merged = [intervals[0]]
        for current in intervals[1:]:
            if current[0] <= merged[-1][1]:
                merged[-1][1] = max(merged[-1][1], current[1])
            else:
                merged.append(current)
        return merged

    intervals = [[1, 3], [2, 6], [8, 10], [15, 18]]
    print(f"Merged: {merge_intervals(intervals)}")


def challenge_4_lfu_cache():
    print("Challenge 4: Implement LFU cache concept")

    class LFUCache:
        def __init__(self, capacity: int):
            self.capacity = capacity
            self.cache = {}
            self.freq = defaultdict(int)

        def get(self, key: int) -> int:
            if key in self.cache:
                self.freq[key] += 1
                return self.cache[key]
            return -1

        def put(self, key: int, value: int):
            if self.capacity <= 0:
                return
            if key in self.cache:
                self.cache[key] = value
                self.freq[key] += 1
            else:
                if len(self.cache) >= self.capacity:
                    lfu_key = min(self.freq, key=lambda k: self.freq[k])
                    del self.cache[lfu_key]
                    del self.freq[lfu_key]
                self.cache[key] = value
                self.freq[key] = 1

    cache = LFUCache(2)
    cache.put(1, 1)
    cache.put(2, 2)
    print(f"Get 1: {cache.get(1)}")
    cache.put(3, 3)
    print(f"Get 2 (evicted): {cache.get(2)}")


challenge_1_implement_queue()
challenge_2_top_k_frequent()
challenge_3_merge_intervals()
challenge_4_lfu_cache()
`

## Summary

Choosing the right data structure is crucial for performance. Python offers list (dynamic array), dict (hash table), set (hash table), deque (double-ended queue), heapq (priority queue), and bisect (binary search). Understanding their time complexities helps write efficient, scalable code.

## Related Topics

- Big O Notation (99_big_o.md)
- Algorithms (98_algorithms.md)
- Caching (96_caching.md)
- Memory Management (92_memory_management.md)
