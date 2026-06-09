# Caching - lru_cache, redis, memoization, cache invalidation strategies

## Introduction
Caching is the practice of storing the results of expensive computations so that future requests for the same input can be served in constant time. Python provides built-in caching via `functools.lru_cache` and `functools.cache` (Python 3.9+), while external systems like Redis provide distributed, persistent, and TTL-based caches. Choosing the right caching strategy — memoization, LRU, TTL, or write-through — directly impacts application latency, throughput, and consistency.

## lru_cache

### What It Is
`functools.lru_cache` is a decorator that wraps a function with a least-recently-used (LRU) cache. It stores the results of function calls keyed by the arguments. When the cache reaches `maxsize`, the least recently used entries are evicted. In Python 3.9+, `functools.cache` is an alias for `lru_cache(maxsize=None)` — an unbounded cache.

### Why It Is Important
LRU caching eliminates redundant computation for pure functions (functions whose return value depends only on their arguments). It is especially effective for recursive algorithms (Fibonacci, dynamic programming), database query results (same query, same result), and expensive mathematical computations.

### How It Works Internally
lru_cache uses a Python dict for O(1) lookups combined with a doubly-linked list for LRU ordering:
1. **Dict**: maps `key` (derived from positional and keyword arguments) to `(result, link)`.
2. **Linked list**: each entry is a `_Link` node with `prev` and `next` pointers. The list head is the most recently used; the tail is the least recently used.
3. **On hit**: the link is moved to the head of the list.
4. **On miss and full**: the tail link is popped, its dict entry deleted, and the new entry is inserted at the head.
5. **Hashing**: arguments are hashed via `_make_key(args, kwds, typed)` — `typed=True` distinguishes `3` from `3.0`.

### Syntax
```python
from functools import lru_cache, cache

@lru_cache(maxsize=128)
def expensive_function(x):
    ...

@lru_cache(maxsize=None)
def deterministic(x):
    ...

@cache
def cached_func(x):
    ...

cached_func.cache_info()
cached_func.cache_clear()
```

### Beginner Examples
```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(100))
print(fibonacci.cache_info())
fibonacci.cache_clear()
```

### Intermediate Examples
```python
from functools import lru_cache
import time

@lru_cache(maxsize=256)
def fetch_user_data(user_id):
    time.sleep(0.1)
    return {'id': user_id, 'name': f'User_{user_id}', 'score': user_id * 10}

t0 = time.perf_counter()
data1 = fetch_user_data(42)
t1 = time.perf_counter()
data2 = fetch_user_data(42)
t2 = time.perf_counter()
print(f'First call: {t1-t0:.3f}s, Cached: {t2-t1:.3f}s')

@lru_cache(maxsize=128, typed=True)
def multiply(a, b):
    return a * b

multiply(3, 4); multiply(3.0, 4.0)
print(multiply.cache_info())
```

### Advanced Examples
```python
from functools import lru_cache, wraps
import time
import pickle

def ttl_cache(seconds=60, maxsize=128):
    def decorator(func):
        cache = {}
        @wraps(func)
        def wrapper(*args, **kwargs):
            key = pickle.dumps((args, tuple(sorted(kwargs.items()))))
            now = time.monotonic()
            if key in cache:
                result, timestamp = cache[key]
                if now - timestamp < seconds:
                    return result
            result = func(*args, **kwargs)
            cache[key] = (result, now)
            if len(cache) > maxsize:
                oldest = min(cache.keys(), key=lambda k: cache[k][1])
                del cache[oldest]
            return result
        wrapper.cache_clear = lambda: cache.clear()
        return wrapper
    return decorator

@ttl_cache(seconds=30)
def get_weather(city):
    return {'city': city, 'temp': 22.5}

class Calculator:
    def __init__(self, name):
        self.name = name
    @lru_cache(maxsize=32)
    def compute(self, x, y):
        return (x ** y + y ** x) ** 0.5
```

### Real-World Use Cases
- **Web framework view caching**: Django/Flask views decorated with `lru_cache` to cache rendered templates.
- **Recursive dynamic programming**: memoising Fibonacci, edit distance, knapsack problems.
- **API response normalisation**: cache parsed and validated API responses keyed by raw payload hash.

### Common Mistakes
- Caching functions with unhashable arguments (lists, dicts).
- Using `lru_cache` on non-pure functions that depend on global state or I/O.
- Forgetting that `lru_cache` key includes `self` for methods — every instance has a separate cache.

### Best Practices
- Use `lru_cache` only for pure functions — same inputs always produce same outputs.
- Set `maxsize` to a power of 2 (128, 256, 1024) based on expected working set.
- Monitor `cache_info()` in production to tune `maxsize`.
- Use `typed=True` when `3` and `3.0` should be cached separately.

### Performance Considerations
- Cache lookup is O(1) hash + dict lookup; overhead is ~50–100 ns.
- Each cached entry stores the return value — be careful with large result objects.
- `maxsize=None` can cause unbounded memory growth in long-running processes.

### Interview Questions
- **Q**: How does `lru_cache` evict entries when the cache is full?  
  **A**: It maintains a doubly-linked list; the tail (LRU) link is removed, its dict entry deleted, and the new entry inserted at the head.
- **Q**: What happens if you apply `lru_cache` to a method with many instances?  
  **A**: Each instance gets its own cache because `self` is part of the key, potentially causing memory leaks.

### Coding Challenges
- Implement your own LRU cache from scratch using `OrderedDict`.
- Write a decorator combining LRU eviction with TTL expiry.

### Related Topics
- [Redis caching](#redis-caching)
- [Memoization](#memoization)
- [Cache invalidation](#cache-invalidation-strategies)

---

## Redis Caching

### What It Is
Redis is an in-memory key-value store that serves as a distributed, persistent, high-throughput cache. Python interacts with Redis via `redis-py`. Redis caches survive process restarts, are shareable across application instances, and support TTL, pub/sub, and atomic operations.

### Why It Is Important
Unlike `lru_cache`, Redis is external to the Python process: cached data survives restarts, is accessible by multiple workers, and supports rich data structures (strings, hashes, lists, sets, sorted sets) beyond simple key-value pairs. Redis is the standard for production caching in web applications.

### How It Works Internally
Redis stores all data in memory with sub-millisecond latency. It uses a single-threaded event loop for commands (no race conditions per key) with optional RDB snapshots and AOF logs for persistence. The `redis-py` client communicates via the Redis Serialization Protocol (RESP) over TCP with connection pooling.

### Syntax
```python
import redis
import json

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

r.set('key', 'value')
r.set('key', 'value', ex=60)
r.set('key', 'value', nx=True)

value = r.get('key')
r.delete('key')
r.exists('key')

r.set('user:42', json.dumps({'name': 'Alice', 'score': 100}))
user = json.loads(r.get('user:42'))

r.incr('counter')
r.incrby('counter', 10)
```

### Beginner Examples
```python
import redis
import json
import time

r = redis.Redis(decode_responses=True)

def get_user(user_id):
    cache_key = f'user:{user_id}'
    cached = r.get(cache_key)
    if cached is not None:
        return json.loads(cached)
    time.sleep(0.1)
    user = {'id': user_id, 'name': f'User_{user_id}'}
    r.setex(cache_key, 3600, json.dumps(user))
    return user
```

### Intermediate Examples
```python
import redis
import json

r = redis.Redis(decode_responses=True)

# Cache-aside pattern
def get_product(product_id):
    key = f'product:{product_id}'
    data = r.get(key)
    if data is not None:
        return json.loads(data)
    data = query_database(product_id)
    r.setex(key, 300, json.dumps(data))
    return data

# Session store with hash
def store_session(session_id, user_data):
    r.hset(f'session:{session_id}', mapping=user_data)
    r.expire(f'session:{session_id}', 1800)

def get_session(session_id):
    return r.hgetall(f'session:{session_id}')

# Rate limiting
def check_rate_limit(user_id, max_requests=100, window=60):
    key = f'ratelimit:{user_id}:{int(time.time() / window)}'
    count = r.incr(key)
    if count == 1:
        r.expire(key, window + 1)
    return count <= max_requests

# Distributed lock
def acquire_lock(lock_name, timeout=10):
    return r.set(f'lock:{lock_name}', '1', nx=True, ex=timeout)

def release_lock(lock_name):
    r.delete(f'lock:{lock_name}')
```

### Advanced Examples
```python
import redis
import json
import hashlib

r = redis.Redis(decode_responses=True)

# Cache stampede protection with locking
def get_expensive_data(key, recompute_func, ttl=300):
    data = r.get(key)
    if data is not None:
        return json.loads(data)

    lock_key = f'lock:{key}'
    if r.set(lock_key, '1', nx=True, ex=10):
        try:
            data = recompute_func()
            r.setex(key, ttl, json.dumps(data))
            return data
        finally:
            r.delete(lock_key)
    else:
        import time
        time.sleep(0.05)
        return get_expensive_data(key, recompute_func, ttl)

# Redis pipeline for batching
def bulk_lookup(keys):
    pipe = r.pipeline()
    for k in keys:
        pipe.get(k)
    results = pipe.execute()
    return {k: json.loads(v) if v else None for k, v in zip(keys, results)}

# Sorted set for leaderboard
def update_score(user_id, score):
    r.zadd('leaderboard', {user_id: score})

def get_top(n=10):
    return r.zrevrange('leaderboard', 0, n-1, withscores=True)

# Pub/sub for cache invalidation
def publish_invalidation(pattern):
    r.publish('cache:invalidate', pattern)

def subscribe_invalidation():
    pubsub = r.pubsub()
    pubsub.subscribe('cache:invalidate')
    for message in pubsub.listen():
        if message['type'] == 'message':
            pattern = message['data']
            for key in r.scan_iter(match=pattern):
                r.delete(key)
```

### Real-World Use Cases
- **Web application session storage**: storing user sessions across multiple web server instances.
- **API response caching**: cache entire API responses in Redis and serve them in <1ms.
- **Job queue**: use Redis lists as a lightweight queue with `r.lpush` / `r.brpop`.

### Common Mistakes
- Not setting TTL on cache entries — causes stale data and unbounded memory growth.
- Using Redis for every cache operation without considering serialisation overhead.
- Forgetting connection pooling — creating a new connection per request is slow.

### Best Practices
- Always set `decode_responses=True` for string-based data.
- Use `pipeline()` for batch operations to reduce round trips.
- Monitor memory with `INFO memory` and set `maxmemory` / `maxmemory-policy`.
- Use connection pooling via `redis.ConnectionPool`.

### Performance Considerations
- Redis operations take ~0.1–1 ms on a local network.
- Serialisation (JSON/pickle) can dominate for large objects; use binary protocols (MessagePack, protobuf) when needed.
- Pipeline batching reduces latency from O(N * RTT) to O(RTT + N * process_time).

### Interview Questions
- **Q**: How does Redis evict keys when memory is full?  
  **A**: Based on `maxmemory-policy` — common policies: `allkeys-lru`, `volatile-lru`, `allkeys-lfu`, `noeviction`.
- **Q**: What is the cache-aside pattern and how is it implemented with Redis?  
  **A**: On read, check cache; on miss, load from DB, store in cache. On write, update DB and delete/update cache entry.

### Coding Challenges
- Implement a distributed rate limiter using Redis sorted sets (sliding window algorithm).
- Build a Redis-backed job queue with retry and dead-letter queue support.

### Related Topics
- [lru_cache](#lru-cache)
- [Memoization](#memoization)
- [Cache invalidation](#cache-invalidation-strategies)

---

## Memoization

### What It Is
Memoization is a specific caching technique where a function's return value is cached based on its input arguments. The first call with a given set of arguments computes the result and stores it; subsequent calls with the same arguments return the cached result. Python's `functools.lru_cache` and `functools.cache` are implementations of memoization.

### Why It Is Important
Memoization transforms exponential-time recursive algorithms into polynomial or linear time (e.g., Fibonacci from O(2^n) to O(n)). It is one of the simplest and most impactful performance optimisations available.

### How It Works Internally
A memoized function maintains a dictionary (or similar mapping) from argument tuples to results. The dictionary is checked before any computation; if a key exists, the stored value is returned without executing the function body. This is conceptually identical to `lru_cache(maxsize=None)`.

### Syntax
```python
from functools import lru_cache, cache

@cache
def f(x): ...

# Manual memoization
memo = {}
def f(x):
    if x not in memo:
        memo[x] = expensive_compute(x)
    return memo[x]
```

### Beginner Examples
```python
from functools import lru_cache

@lru_cache(maxsize=None)
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

# Without memoization: factorial recursively recomputes
# With memoization: each n computed exactly once
print(factorial(500))
```

### Intermediate Examples
```python
from functools import lru_cache

@lru_cache(maxsize=2048)
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

# Manual memoization decorator
def memoize(func):
    cache = {}
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    wrapper.cache = cache
    return wrapper

@memoize
def knapsack(capacity, weights, values, n):
    if n == 0 or capacity == 0:
        return 0
    if weights[n-1] > capacity:
        return knapsack(capacity, weights, values, n-1)
    return max(
        values[n-1] + knapsack(capacity - weights[n-1], weights, values, n-1),
        knapsack(capacity, weights, values, n-1)
    )
```

### Advanced Examples
```python
from functools import lru_cache
import functools

# Memoization with cache eviction based on external events
def memoized_with_invalidation(func):
    cache = {}
    version = [0]

    def invalidate():
        version[0] += 1
        cache.clear()

    @functools.wraps(func)
    def wrapper(*args):
        key = (version[0], args)
        if key not in cache:
            cache[key] = func(*args)
        return cache[key]

    wrapper.invalidate = invalidate
    wrapper.cache_clear = cache.clear
    return wrapper

# Memoization for class methods with per-instance cache
class DataProcessor:
    def __init__(self, name):
        self.name = name
        self._cache = {}

    def process(self, key):
        if key not in self._cache:
            self._cache[key] = self._expensive_transform(key)
        return self._cache[key]

    def _expensive_transform(self, key):
        return sum(ord(c) * i for i, c in enumerate(str(key) * 1000))
```

### Real-World Use Cases
- **Compiler optimisation**: memoise the results of constant folding and common subexpression elimination.
- **Parsing**: memoise parse results for ambiguous grammars (e.g., with `pyparsing` or `lark`).
- **Image processing**: memoise tile rendering results in map/reduce pipelines.

### Common Mistakes
- Memoizing functions with side effects — cached results may not reflect updated global state.
- Using mutable objects as keys — they are unhashable and cause `TypeError`.
- Memoizing functions that depend on I/O (file reads, network requests) without considering staleness.

### Best Practices
- Use `@functools.cache` for pure functions with no side effects.
- For methods, be aware that `self` is included in the cache key — each instance has its own cache.
- Clear caches explicitly when underlying data changes.

### Performance Considerations
- Memoization trades memory for speed — a highly memoized function can consume significant memory.
- The cost of dictionary lookup (~50 ns) is negligible compared to most expensive computations.
- For functions with many unique arguments, memoization is ineffective — the cache fills with one-hit entries.

### Interview Questions
- **Q**: What is the space-time tradeoff in memoization?  
  **A**: Memoization uses additional memory (stores all computed results) in exchange for O(1) retrieval of previously computed values, reducing time from exponential to polynomial or linear.
- **Q**: How would you memoize a function with keyword arguments?  
  **A**: Convert `kwargs` to a sorted tuple of `(key, value)` pairs and combine with positional args as a single hashable key.

### Coding Challenges
- Implement a memoization decorator that supports a `maxsize` parameter and uses LRU eviction with `OrderedDict`.
- Write a memoized version of the Ackermann function and measure its performance.

### Related Topics
- [lru_cache](#lru-cache)
- [Cache invalidation](#cache-invalidation-strategies)
- [Algorithms (dynamic programming)](#algorithms---sorting-searching-recursion-dynamic-programming)

---

## Cache Invalidation Strategies

### What It Is
Cache invalidation is the process of removing or updating cached data when the underlying source data changes. It is famously one of the two hard things in computer science. Common strategies include TTL (time-to-live), write-through, write-behind, and event-driven invalidation.

### Why It Is Important
Without invalidation, caches serve stale data — users see outdated information, calculations are based on obsolete inputs, and system correctness is compromised. Proper invalidation ensures cache consistency without sacrificing performance.

### How It Works Internally
Each invalidation strategy uses different mechanisms:
- **TTL**: a timestamp or expiry duration is stored with each cache entry; on access, expired entries are treated as misses.
- **Write-through**: the application updates the cache synchronously whenever it writes to the database.
- **Write-behind**: updates are queued and applied to the cache asynchronously after a delay.
- **Event-driven**: the data source publishes invalidation events (via Redis pub/sub, RabbitMQ, Kafka) that cache consumers listen to and react to.

### Syntax
```python
# TTL-based invalidation (setex)
r.setex('key', 3600, value)

# Write-through
def update_user(user_id, data):
    db_update(user_id, data)
    r.set(f'user:{user_id}', json.dumps(data))

# Manual invalidation
def invalidate_user(user_id):
    r.delete(f'user:{user_id}')

# Pattern-based invalidation
def invalidate_pattern(pattern):
    for key in r.scan_iter(match=pattern):
        r.delete(key)
```

### Beginner Examples
```python
import time
from functools import lru_cache

# TTL-based cache without external dependencies
class TTLCache:
    def __init__(self, ttl_seconds=60):
        self.ttl = ttl_seconds
        self.cache = {}

    def get(self, key):
        if key in self.cache:
            value, timestamp = self.cache[key]
            if time.monotonic() - timestamp < self.ttl:
                return value
            del self.cache[key]
        return None

    def set(self, key, value):
        self.cache[key] = (value, time.monotonic())

cache = TTLCache(30)
cache.set('weather', 'sunny')
print(cache.get('weather'))
```

### Intermediate Examples
```python
# Cache invalidation with version keys
class VersionedCache:
    def __init__(self, redis_client):
        self.r = redis_client

    def get(self, key):
        version = self.r.get(f'version:{key}')
        data = self.r.get(f'data:{key}')
        if data and version:
            stored_version = self.r.hget(data, '_version')
            if stored_version == version:
                return data
        return None

    def set(self, key, value, ttl=300):
        import time
        version = str(time.time())
        self.r.set(f'version:{key}', version)
        value['_version'] = version
        self.r.setex(f'data:{key}', ttl, json.dumps(value))

    def invalidate(self, key):
        self.r.incr(f'version:{key}')

# Bulk invalidation with tags
class TaggedCache:
    def __init__(self, redis_client):
        self.r = redis_client

    def set(self, key, value, tags=None, ttl=300):
        self.r.setex(f'cache:{key}', ttl, json.dumps(value))
        if tags:
            for tag in tags:
                self.r.sadd(f'tag:{tag}', key)
                self.r.expire(f'tag:{tag}', ttl + 60)

    def invalidate_tag(self, tag):
        keys = self.r.smembers(f'tag:{tag}')
        if keys:
            self.r.delete(*[f'cache:{k}' for k in keys])
            self.r.delete(f'tag:{tag}')
```

### Advanced Examples
```python
# Stale-while-revalidate pattern
class StaleWhileRevalidate:
    def __init__(self, redis_client, stale_ttl=60, revalidate_func=None):
        self.r = redis_client
        self.stale_ttl = stale_ttl
        self.revalidate = revalidate_func

    def get(self, key):
        data = self.r.get(key)
        if data is None:
            return None

        entry = json.loads(data)
        if entry['fresh_until'] > time.time():
            return entry['value']

        if entry.get('stale_until', 0) > time.time() and self.revalidate:
            import threading
            threading.Thread(target=self._revalidate, args=(key,), daemon=True).start()
            return entry['value']

        return None

    def set(self, key, value, fresh_ttl=60):
        entry = {
            'value': value,
            'fresh_until': time.time() + fresh_ttl,
            'stale_until': time.time() + fresh_ttl + self.stale_ttl,
        }
        self.r.setex(key, fresh_ttl + self.stale_ttl, json.dumps(entry))

    def _revalidate(self, key):
        if self.revalidate:
            new_value = self.revalidate(key)
            if new_value is not None:
                self.set(key, new_value)

# Cache invalidation with database triggers
def setup_invalidation_triggers(db_connection, redis_client):
    def on_row_update(table, row_id):
        redis_client.delete(f'cache:{table}:{row_id}')
        redis_client.publish('cache:invalidate', f'{table}:{row_id}')
    return on_row_update

# Read-through cache with write-invalidate
class ReadThroughCache:
    def __init__(self, redis_client, loader, ttl=300):
        self.r = redis_client
        self.loader = loader
        self.ttl = ttl

    def get(self, key):
        data = self.r.get(key)
        if data is not None:
            return json.loads(data)
        value = self.loader(key)
        self.r.setex(key, self.ttl, json.dumps(value))
        return value

    def invalidate(self, key):
        self.r.delete(key)
```

### Real-World Use Cases
- **E-commerce product catalogue**: invalidate product cache when price or inventory changes.
- **Content management systems**: purge CDN/page cache when a page is published or edited.
- **Social media feeds**: invalidate user feed cache when new posts or follows occur.

### Common Mistakes
- Using TTL too long — users see stale data until the TTL expires.
- Using TTL too short — cache hit ratio drops, defeating the purpose of caching.
- Forgetting to invalidate related caches when data changes — e.g., invalidating both `user:42` and `feed:42` when a user updates their profile.
- Implementing write-behind without handling failures — the cache may permanently diverge from the source of truth.

### Best Practices
- Prefer TTL-based invalidation for data that changes infrequently and where slight staleness is acceptable.
- Use event-driven invalidation (Redis pub/sub, database triggers) for data that changes frequently or where consistency is critical.
- Use write-through for critical data where stale reads are unacceptable.
- Always measure cache hit ratio — a low hit ratio suggests the invalidation strategy is too aggressive or TTL is too short.

### Performance Considerations
- TTL checks add no overhead — Redis lazily evicts on access or when the key is next scanned.
- Event-driven invalidation has propagation delay (milliseconds via pub/sub, potentially seconds via polling).
- Write-through adds latency to write operations; for write-heavy workloads, consider write-behind or TTL.

### Interview Questions
- **Q**: What are the two hard things in computer science according to the famous quote?  
  **A**: Cache invalidation, naming things, and off-by-one errors.
- **Q**: Compare TTL invalidation vs event-driven invalidation.  
  **A**: TTL is simple and decoupled but allows stale data for up to the TTL period. Event-driven is immediate but requires infrastructure (pub/sub, message bus) and adds system complexity.

### Coding Challenges
- Implement a write-through cache layer in front of a simulated database, ensuring that every write updates both the DB and the cache atomically.
- Design and implement a two-level cache (L1 in-memory LRU, L2 Redis) with write-invalidate propagation from L2 to L1 using pub/sub.

### Related Topics
- [lru_cache](#lru-cache)
- [Redis caching](#redis-caching)
- [Memoization](#memoization)
