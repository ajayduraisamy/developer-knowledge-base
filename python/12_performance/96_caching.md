# Caching - lru_cache, redis, memoization, cache invalidation strategies

## Introduction

Caching is a technique that stores computed results so that future requests for the same data can be served faster. Python provides built-in caching mechanisms like `functools.lru_cache` and `functools.cache`, as well as third-party libraries like `cachetools` and `redis` for distributed caching.

## Why It Is Important

Caching significantly improves application performance by avoiding redundant computations, reducing database load, and minimizing network requests. It is essential for scaling web applications, optimizing expensive function calls, and improving user experience through faster response times.

## Syntax

```python
# LRU Cache (Least Recently Used)
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_function(n):
    return n ** 2

# Simple cache (Python 3.9+)
from functools import cache

@cache
def cached_function(n):
    return n ** 2

# Cachetools library
from cachetools import cached, LRUCache, TTLCache

@cached(cache=LRUCache(maxsize=100))
def cached_func(n):
    return n ** 2

@cached(cache=TTLCache(maxsize=100, ttl=300))
def ttl_cached_func(n):
    return n ** 2

# Manual dict-based cache
cache = {}
def get_or_compute(key, compute_func):
    if key not in cache:
        cache[key] = compute_func(key)
    return cache[key]
```

## Examples

```python
import time
from functools import lru_cache, cache


def fibonacci_without_cache(n: int) -> int:
    if n < 2:
        return n
    return fibonacci_without_cache(n - 1) + fibonacci_without_cache(n - 2)


@lru_cache(maxsize=None)
def fibonacci_with_cache(n: int) -> int:
    if n < 2:
        return n
    return fibonacci_with_cache(n - 1) + fibonacci_with_cache(n - 2)


start = time.perf_counter()
fib_without = fibonacci_without_cache(35)
t1 = time.perf_counter() - start
print(f"Without cache: {t1:.4f}s")

start = time.perf_counter()
fib_with = fibonacci_with_cache(35)
t2 = time.perf_counter() - start
print(f"With cache: {t2:.4f}s")
print(f"Speedup: {t1 / t2:.0f}x")
```

## Beginner Examples

```python
import time
from functools import lru_cache


def lru_cache_basics():
    print("LRU Cache Basics:")
    print("1. @lru_cache(maxsize=128) caches recent calls")
    print("2. maxsize=None for unlimited cache")
    print("3. Cache is thread-safe")
    print("4. Arguments must be hashable")


lru_cache_basics()


@lru_cache(maxsize=32)
def compute_square(n: int) -> int:
    print(f"Computing square of {n}")
    time.sleep(0.1)
    return n * n


def demonstrate_caching():
    print("First calls (will compute):")
    print(f"  square(5) = {compute_square(5)}")
    print(f"  square(10) = {compute_square(10)}")
    print(f"  square(5) = {compute_square(5)}")
    print("  square(5) retrieved from cache!")
    print(f"Cache info: {compute_square.cache_info()}")
    compute_square.cache_clear()
    print("Cache cleared")
    print(f"  square(5) = {compute_square(5)}")


demonstrate_caching()


@lru_cache(maxsize=128)
def expensive_string_operation(text: str) -> str:
    print(f"Processing: {text}")
    time.sleep(0.2)
    return text.upper()[::-1]


def demonstrate_string_caching():
    print("\nString caching:")
    print(expensive_string_operation("hello"))
    print(expensive_string_operation("world"))
    print(expensive_string_operation("hello"))
    print(f"Cache info: {expensive_string_operation.cache_info()}")


demonstrate_string_caching()
```

## Intermediate Examples

```python
import time
from functools import lru_cache, cache
from cachetools import cached, LRUCache, TTLCache
from typing import Any, Callable, Dict, Tuple, List
import threading


class CacheDecorators:
    def __init__(self):
        pass

    def functools_cache(self):
        print("functools.cache (Python 3.9+):")
        print("""
    from functools import cache

    @cache
    def compute(data):
        return data ** 2

    # Same as @lru_cache(maxsize=None)
    # Simpler, unlimited cache
        """)

    def cachetools_lru(self):
        print("cachetools LRUCache:")
        print("""
    from cachetools import cached, LRUCache

    @cached(cache=LRUCache(maxsize=100))
    def expensive_func(n):
        return n ** n
        """)

    def cachetools_ttl(self):
        print("cachetools TTLCache (time-to-live):")
        print("""
    from cachetools import cached, TTLCache

    @cached(cache=TTLCache(maxsize=100, ttl=300))
    def get_data_from_db(id):
        # Expires after 5 minutes
        return database.query(id)
        """)


decorators = CacheDecorators()
decorators.functools_cache()
decorators.cachetools_lru()
decorators.cachetools_ttl()


class FibonacciCache:
    def __init__(self):
        self._cache: Dict[int, int] = {0: 0, 1: 1}
        self._lock = threading.Lock()

    def get(self, n: int) -> int:
        with self._lock:
            if n not in self._cache:
                self._cache[n] = self.get(n - 1) + self.get(n - 2)
            return self._cache[n]

    def clear(self):
        with self._lock:
            self._cache.clear()
            self._cache[0] = 0
            self._cache[1] = 1


fib_cache = FibonacciCache()
start = time.perf_counter()
result = fib_cache.get(100)
print(f"Fibonacci(100) = {result}")
print(f"Time: {time.perf_counter() - start:.6f}s")
print(f"Cache size: {len(fib_cache._cache)}")


def lru_cache_with_args():
    print("\nLRU cache with keyword arguments:")

    @lru_cache(maxsize=128)
    def complex_func(a: int, b: int, c: int = 0) -> int:
        return a * b + c

    print(complex_func(2, 3))
    print(complex_func(2, 3, 1))
    print(complex_func(2, 3))
    print(f"Cache info: {complex_func.cache_info()}")


lru_cache_with_args()
```

## Advanced Examples

```python
import time
import json
import pickle
import hashlib
from functools import lru_cache, wraps
from typing import Any, Callable, Dict, Tuple, Optional, Union
import threading


class AdvancedCaching:
    def __init__(self):
        pass

    def redis_caching(self):
        print("Redis Caching Pattern:")
        print("""
    import redis
    import json

    r = redis.Redis(host='localhost', port=6379, db=0)

    def get_user(user_id):
        cache_key = f'user:{user_id}'
        cached = r.get(cache_key)
        if cached:
            return json.loads(cached)
        user = database.query(f"SELECT * FROM users WHERE id = {user_id}")
        r.setex(cache_key, 300, json.dumps(user))
        return user
        """)

    def multi_level_cache(self):
        print("Multi-level caching strategy:")
        print("""
    Level 1: Local memory cache (fastest, limited)
    Level 2: Redis cache (fast, distributed)
    Level 3: Database (slow, authoritative)
        """)

    def cache_invalidation(self):
        print("Cache invalidation strategies:")
        print("1. TTL (Time To Live) - expire after time")
        print("2. Write-through - update cache on write")
        print("3. Write-behind - async cache update")
        print("4. Event-driven - invalidate on changes")
        print("5. Manual - explicit cache clearing")


adv = AdvancedCaching()
adv.redis_caching()
adv.multi_level_cache()
adv.cache_invalidation()


class GenericCache:
    def __init__(self, maxsize: int = 128, ttl: Optional[float] = None):
        self._cache: Dict[str, Tuple[Any, float]] = {}
        self._maxsize = maxsize
        self._ttl = ttl
        self._lock = threading.Lock()

    def _make_key(self, args, kwargs) -> str:
        key = str(args) + str(sorted(kwargs.items()))
        return hashlib.md5(key.encode()).hexdigest()

    def get(self, key: str) -> Optional[Any]:
        with self._lock:
            if key in self._cache:
                value, timestamp = self._cache[key]
                if self._ttl is None or time.time() - timestamp < self._ttl:
                    return value
                del self._cache[key]
        return None

    def set(self, key: str, value: Any):
        with self._lock:
            if len(self._cache) >= self._maxsize:
                oldest = min(self._cache.keys(), key=lambda k: self._cache[k][1])
                del self._cache[oldest]
            self._cache[key] = (value, time.time())

    def clear(self):
        with self._lock:
            self._cache.clear()


def cached_with_ttl(ttl: Optional[float] = None, maxsize: int = 128):
    def decorator(func: Callable):
        cache = GenericCache(maxsize=maxsize, ttl=ttl)

        @wraps(func)
        def wrapper(*args, **kwargs):
            key = cache._make_key(args, kwargs)
            result = cache.get(key)
            if result is not None:
                return result
            result = func(*args, **kwargs)
            cache.set(key, result)
            return result
        return wrapper
    return decorator


@cached_with_ttl(ttl=10, maxsize=100)
def expensive_data_fetch(query: str) -> dict:
    print(f"Fetching: {query}")
    time.sleep(0.5)
    return {'query': query, 'result': 'data', 'timestamp': time.time()}


print(expensive_data_fetch("SELECT * FROM users"))
print(expensive_data_fetch("SELECT * FROM users"))
time.sleep(11)
print(expensive_data_fetch("SELECT * FROM users"))


class CacheStatistics:
    def __init__(self):
        self.hits = 0
        self.misses = 0

    @property
    def hit_rate(self) -> float:
        total = self.hits + self.misses
        return self.hits / total if total > 0 else 0.0

    def record_hit(self):
        self.hits += 1

    def record_miss(self):
        self.misses += 1

    def report(self) -> dict:
        return {'hits': self.hits, 'misses': self.misses, 'hit_rate': self.hit_rate}
```

## Real-World Use Cases

```python
import time
import json
from functools import lru_cache
from typing import Dict, List, Optional


def web_api_caching():
    print("Real-world: Web API Response Caching")

    @lru_cache(maxsize=256)
    def get_user_profile(user_id: int) -> dict:
        print(f"Fetching profile for user {user_id}")
        time.sleep(0.2)
        return {'id': user_id, 'name': f'User_{user_id}', 'email': f'user{user_id}@example.com'}

    for uid in [1, 2, 3, 1, 2, 3]:
        profile = get_user_profile(uid)
        print(f"  User {uid}: {profile['name']}")
    print(f"Cache info: {get_user_profile.cache_info()}")


def database_query_caching():
    print("Real-world: Database Query Caching")

    class QueryCache:
        def __init__(self, maxsize: int = 100):
            self._cache: Dict[str, List[dict]] = {}
            self._maxsize = maxsize

        def get(self, sql: str) -> Optional[List[dict]]:
            return self._cache.get(sql)

        def set(self, sql: str, results: List[dict]):
            if len(self._cache) >= self._maxsize:
                self._cache.pop(next(iter(self._cache)))
            self._cache[sql] = results

        def invalidate(self, table: str):
            keys_to_delete = [k for k in self._cache if table in k]
            for k in keys_to_delete:
                del self._cache[k]

    cache = QueryCache()
    sql1 = "SELECT * FROM users WHERE id = 1"
    sql2 = "SELECT * FROM users WHERE id = 2"
    cache.set(sql1, [{'id': 1, 'name': 'Alice'}])
    cache.set(sql2, [{'id': 2, 'name': 'Bob'}])
    print(f"Cache hit: {cache.get(sql1)}")
    cache.invalidate('users')
    print(f"After invalidation: {cache.get(sql1)}")


def http_caching():
    print("Real-world: HTTP Response Caching")
    print("""
    import requests
    from cachetools import cached, TTLCache

    session = requests.Session()
    api_cache = TTLCache(maxsize=100, ttl=60)

    @cached(api_cache)
    def fetch_api(endpoint):
        return session.get(f'https://api.example.com/{endpoint}').json()
    """)


def memoization_pattern():
    print("Real-world: Memoization for Dynamic Programming")

    @lru_cache(maxsize=None)
    def edit_distance(s1: str, s2: str) -> int:
        if not s1:
            return len(s2)
        if not s2:
            return len(s1)
        if s1[0] == s2[0]:
            return edit_distance(s1[1:], s2[1:])
        return 1 + min(
            edit_distance(s1[1:], s2),
            edit_distance(s1, s2[1:]),
            edit_distance(s1[1:], s2[1:])
        )

    print(f"Edit distance 'kitten' vs 'sitting': {edit_distance('kitten', 'sitting')}")
    print(f"Cache info: {edit_distance.cache_info()}")


web_api_caching()
database_query_caching()
http_caching()
memoization_pattern()
```

## Common Mistakes

```python
from functools import lru_cache
from typing import List


def mistake_1_mutable_arguments():
    print("Mistake 1: Using unhashable types as arguments")
    print("Lists, dicts, sets can't be cached")

    @lru_cache(maxsize=128)
    def process_list(items: tuple) -> int:
        return sum(items)

    print(f"Process tuple: {process_list((1, 2, 3))}")
    print("Fix: Convert to tuple before caching")


def mistake_2_unlimited_cache_growth():
    print("Mistake 2: Using maxsize=None without limits")
    print("Can lead to memory exhaustion")
    print("Always set a reasonable maxsize")


def mistake_3_caching_side_effects():
    print("Mistake 3: Caching functions with side effects")
    print("Cache assumes pure functions (same input -> same output)")
    print("Don't cache functions that modify global state")


def mistake_4_caching_io_operations():
    print("Mistake 4: Caching stale I/O results")
    print("File contents, DB queries, API responses change")
    print("Use TTL-based caching for dynamic data")


def mistake_5_over_caching():
    print("Mistake 5: Caching too aggressively")
    print("Cache overhead can exceed computation cost")
    print("Profile to verify caching is beneficial")


def mistake_6_not_clearing_cache():
    print("Mistake 6: Never clearing the cache")
    print("Implement cache invalidation strategy")
    print("Use .cache_clear() when data changes")


mistake_1_mutable_arguments()
mistake_2_unlimited_cache_growth()
mistake_3_caching_side_effects()
mistake_4_caching_io_operations()
mistake_5_over_caching()
mistake_6_not_clearing_cache()
```

## Best Practices

```python
from functools import lru_cache, wraps
from typing import Any, Callable, TypeVar

F = TypeVar('F', bound=Callable[..., Any])


def best_practice_1_measure_benefit():
    print("Best Practice 1: Measure cache effectiveness")
    print("Track hit/miss ratios")


def best_practice_2_set_reasonable_maxsize():
    print("Best Practice 2: Set appropriate maxsize")
    print("Too small: low hit rate")
    print("Too large: memory waste")


def best_practice_3_use_ttl_for_dynamic_data():
    print("Best Practice 3: Use TTL for time-sensitive data")
    print("API responses, DB queries, file contents")


def best_practice_4_cache_pure_functions():
    print("Best Practice 4: Only cache pure functions")
    print("No side effects, deterministic output")


def best_practice_5_implement_invalidation():
    print("Best Practice 5: Implement cache invalidation")
    print("Write-through, write-behind, or event-driven")


def best_practice_6_use_cachetools_for_complex():
    print("Best Practice 6: Use cachetools for advanced needs")
    print("TTLCache, LRUCache, LFUCache, FIFOCache")


def best_practice_7_serialize_for_redis():
    print("Best Practice 7: Serialize for distributed caches")
    print("Use pickle or JSON for Redis caching")


best_practice_1_measure_benefit()
best_practice_2_set_reasonable_maxsize()
best_practice_3_use_ttl_for_dynamic_data()
best_practice_4_cache_pure_functions()
best_practice_5_implement_invalidation()
best_practice_6_use_cachetools_for_complex()
best_practice_7_serialize_for_redis()
```

## Interview Questions

```python
def interview_q1():
    print("Q: What is the difference between LRU and LFU?")
    print("A: LRU removes least recently used items.")
    print("   LFU removes least frequently used items.")


def interview_q2():
    print("Q: How does functools.lru_cache work?")
    print("A: Uses a dictionary to store results.")
    print("   Evicts oldest items when maxsize exceeded.")


def interview_q3():
    print("Q: What is cache invalidation?")
    print("A: The process of removing stale data from cache.")
    print("   Hardest problem in computer science.")


def interview_q4():
    print("Q: What is a write-through cache?")
    print("A: Data written to cache and DB simultaneously.")
    print("   Ensures consistency, higher write latency.")


def interview_q5():
    print("Q: How do you implement TTL-based caching?")
    print("A: Store timestamp with each cached value.")
    print("   Check expiration on retrieval.")


def interview_q6():
    print("Q: What are cache stampede issues?")
    print("A: Multiple requests for expired cache simultaneously.")
    print("   Use dog-pile prevention or early recomputation.")


def interview_q7():
    print("Q: What is memoization?")
    print("A: Caching function results based on arguments.")
    print("   Common in dynamic programming.")


interview_q1()
interview_q2()
interview_q3()
interview_q4()
interview_q5()
interview_q6()
interview_q7()
```

## Coding Challenges

```python
import time
from functools import lru_cache
from typing import List, Tuple


def challenge_1_implement_lru():
    print("Challenge 1: Implement your own LRU Cache")

    class LRUCache:
        def __init__(self, capacity: int):
            self.capacity = capacity
            self._cache = {}
            self._order = []

        def get(self, key: int) -> int:
            if key in self._cache:
                self._order.remove(key)
                self._order.append(key)
                return self._cache[key]
            return -1

        def put(self, key: int, value: int):
            if key in self._cache:
                self._order.remove(key)
            elif len(self._cache) >= self.capacity:
                oldest = self._order.pop(0)
                del self._cache[oldest]
            self._cache[key] = value
            self._order.append(key)

    cache = LRUCache(2)
    cache.put(1, 1)
    cache.put(2, 2)
    print(f"Get 1: {cache.get(1)}")
    cache.put(3, 3)
    print(f"Get 2 (should be -1): {cache.get(2)}")
    cache.put(4, 4)
    print(f"Get 1 (should be -1): {cache.get(1)}")
    print(f"Get 3: {cache.get(3)}")
    print(f"Get 4: {cache.get(4)}")


def challenge_2_memoize_fibonacci():
    print("Challenge 2: Memoize Fibonacci and compare")

    def fib_recursive(n):
        if n < 2:
            return n
        return fib_recursive(n - 1) + fib_recursive(n - 2)

    @lru_cache(maxsize=None)
    def fib_memoized(n):
        if n < 2:
            return n
        return fib_memoized(n - 1) + fib_memoized(n - 2)

    import time
    start = time.perf_counter()
    fib_recursive(35)
    t1 = time.perf_counter() - start

    start = time.perf_counter()
    fib_memoized(35)
    t2 = time.perf_counter() - start

    print(f"Recursive: {t1:.4f}s")
    print(f"Memoized: {t2:.6f}s")
    print(f"Speedup: {t1 / t2:.0f}x")


def challenge_3_ttl_cache_implementation():
    print("Challenge 3: Implement TTL Cache")

    class TTLCache:
        def __init__(self, ttl: float = 60.0):
            self._cache = {}
            self._ttl = ttl

        def get(self, key: str):
            if key in self._cache:
                value, timestamp = self._cache[key]
                if time.time() - timestamp < self._ttl:
                    return value
                del self._cache[key]
            return None

        def set(self, key: str, value):
            self._cache[key] = (value, time.time())

    cache = TTLCache(ttl=2.0)
    cache.set('key1', 'value1')
    print(f"Immediate: {cache.get('key1')}")
    time.sleep(3)
    print(f"After TTL: {cache.get('key1')}")


def challenge_4_cache_decorator():
    print("Challenge 4: Write a generic cache decorator")

    def memoize(maxsize: int = 128):
        def decorator(func):
            cache = {}
            order = []

            def wrapper(*args):
                if args in cache:
                    order.remove(args)
                    order.append(args)
                    return cache[args]
                result = func(*args)
                if len(cache) >= maxsize:
                    oldest = order.pop(0)
                    del cache[oldest]
                cache[args] = result
                order.append(args)
                return result

            wrapper.cache_info = lambda: {'size': len(cache), 'maxsize': maxsize}
            wrapper.cache_clear = lambda: (cache.clear(), order.clear())
            return wrapper
        return decorator

    @memoize(maxsize=10)
    def slow_square(n):
        time.sleep(0.1)
        return n * n

    print(slow_square(5))
    print(slow_square(5))
    print(f"Cache info: {slow_square.cache_info()}")


challenge_1_implement_lru()
challenge_2_memoize_fibonacci()
challenge_3_ttl_cache_implementation()
challenge_4_cache_decorator()
```

## Summary

Caching is crucial for application performance. Python provides functools.lru_cache and functools.cache for simple memoization, cachetools for advanced strategies (LRU, TTL, LFU), and Redis for distributed caching. Key considerations include cache invalidation, TTL management, and avoiding common pitfalls like caching impure functions.

## Related Topics

- Data Structures (97_data_structures.md)
- Memory Management (92_memory_management.md)
- Profiling (91_profiling.md)
- Algorithms (98_algorithms.md)
