# Redis - redis.Redis(), string/set/hash ops, pub/sub, caching

## Introduction
Redis (Remote Dictionary Server) is an in-memory data structure store that functions as a database, cache, message broker, and streaming engine. It supports a rich set of data structures including strings, hashes, lists, sets, sorted sets, bitmaps, hyperloglogs, and geospatial indexes. Python developers interact with Redis primarily through the `redis-py` library (`redis.Redis()`), which provides a clean, Pythonic interface to Redis commands. Redis's sub-millisecond response times, built-in replication, persistence options, and pub/sub messaging make it indispensable for high-performance applications requiring real-time data access, session storage, job queues, and caching layers.

## redis.Redis()

### What It Is
`redis.Redis()` is the main client class in the `redis-py` library. It provides methods that map directly to Redis commands (e.g., `set()`, `get()`, `hset()`, `lpush()`). The client manages connections to a Redis server, handling connection pooling, authentication, and protocol encoding transparently.

### Why It Is Important
The Redis client is the primary interface for all Redis operations. It abstracts socket-level communication, supports connection pooling for high concurrency, provides automatic type conversion between Python types and Redis wire protocol, and handles reconnection logic. Without it, every Redis interaction would require manual socket management and protocol encoding.

### How It Works Internally
When `redis.Redis()` is instantiated, it creates a `ConnectionPool` that manages a set of TCP connections to the Redis server. Commands are serialized using the Redis Serialization Protocol (RESP) and sent over the connection. Responses are deserialized back into Python types (bytes, str, int, list, dict). The client supports pipelining (sending multiple commands without waiting for responses) and transactions (via `MULTI/EXEC` blocks).

### Syntax
```python
import redis

# Basic connection
r = redis.Redis(host="localhost", port=6379, db=0)

# With password
r = redis.Redis(host="localhost", port=6379, password="secret", db=0)

# With connection pool (recommended for production)
pool = redis.ConnectionPool(
    host="localhost", port=6379, db=0,
    max_connections=50, decode_responses=True
)
r = redis.Redis(connection_pool=pool)

# From URL
r = redis.from_url("redis://user:pass@localhost:6379/0")

# For SSL/TLS
r = redis.Redis(host="localhost", port=6380, ssl=True, ssl_cert_reqs="required")
```

### Beginner Examples
```python
import redis

r = redis.Redis(host="localhost", port=6379, db=0, decode_responses=True)

# Basic string operations
r.set("greeting", "Hello, Redis!")
value = r.get("greeting")
print(value)  # "Hello, Redis!"

# Check if key exists
print(r.exists("greeting"))  # 1

# Delete a key
r.delete("greeting")

# Set with expiration (seconds)
r.setex("session_token", 3600, "abc123")
```

### Intermediate Examples
```python
import redis

r = redis.Redis(decode_responses=True)

# Atomic counter
r.set("visitor_count", 0)
r.incr("visitor_count")
r.incrby("visitor_count", 5)
print(r.get("visitor_count"))  # "6"

# Batch operations with pipeline
pipe = r.pipeline(transaction=True)
pipe.set("key1", "value1")
pipe.set("key2", "value2")
pipe.incr("counter")
pipe.execute()

# Scan keys (avoid KEYS in production)
cursor = 0
while True:
    cursor, keys = r.scan(cursor=cursor, match="user:*", count=100)
    for key in keys:
        print(key)
    if cursor == 0:
        break

# Key expiration and TTL
r.setex("temp_data", 60, "will expire")
print(r.ttl("temp_data"))  # ~57
r.expire("temp_data", 120)  # Extend TTL
r.persist("temp_data")  # Remove TTL
```

## String operations

### What It Is
Redis strings are the most fundamental data type. A string value can be text, binary data (up to 512MB), numbers, or serialized objects. String operations include setting, getting, appending, incrementing, and bit-level manipulation.

### Syntax
```python
r.set("key", "value")           # Set key to value
r.get("key")                    # Get value
r.getset("key", "new_value")    # Set and return old value
r.append("key", " appended")    # Append to existing value
r.strlen("key")                 # Get length
r.incr("counter")               # Increment by 1
r.incrby("counter", 10)         # Increment by 10
r.decr("counter")               # Decrement by 1
r.mset({"a": 1, "b": 2})       # Set multiple keys
r.mget(["a", "b"])             # Get multiple keys
```

### Advanced Examples
```python
import json

r = redis.Redis(decode_responses=True)

# Storing complex objects as JSON strings
user = {"id": 1, "name": "Alice", "email": "alice@example.com"}
r.set(f"user:{user['id']}", json.dumps(user))

# Retrieve and deserialize
data = json.loads(r.get("user:1"))
print(data["name"])

# Rate limiting with string counters
def check_rate_limit(user_id, max_requests=100, window=60):
    key = f"ratelimit:{user_id}:{int(time.time() / window)}"
    count = r.incr(key)
    if count == 1:
        r.expire(key, window)
    return count <= max_requests

# Bit operations
r.setbit("bitmap", 7, 1)  # Set bit at offset 7 to 1
print(r.getbit("bitmap", 7))  # 1
print(r.bitcount("bitmap"))  # Count of set bits
```

## Set operations

### What It Is
Redis sets are unordered collections of unique strings. They support efficient membership testing, set algebra (union, intersection, difference), and are ideal for tagging, deduplication, and social graph features.

### Syntax
```python
r.sadd("tags:python", "flask", "django", "fastapi")
r.smembers("tags:python")     # All members
r.sismember("tags:python", "flask")  # Check membership
r.scard("tags:python")        # Cardinality (count)
r.srem("tags:python", "flask")  # Remove member
r.spop("tags:python")         # Remove and return random member

# Set operations
r.sadd("set1", "a", "b", "c")
r.sadd("set2", "b", "c", "d")
r.sunion("set1", "set2")        # {"a", "b", "c", "d"}
r.sinter("set1", "set2")        # {"b", "c"}
r.sdiff("set1", "set2")         # {"a"}
```

### Advanced Examples
```python
# Tag-based article system
def tag_article(article_id, tags):
    r.sadd(f"article:{article_id}:tags", *tags)
    for tag in tags:
        r.sadd(f"tag:{tag}:articles", article_id)

def get_articles_by_tags(include_tags, exclude_tags=None):
    if include_tags:
        keys = [f"tag:{t}:articles" for t in include_tags]
        article_ids = r.sinter(*keys)
    else:
        article_ids = set()
    if exclude_tags:
        exclude_keys = [f"tag:{t}:articles" for t in exclude_tags]
        article_ids -= r.sunion(*exclude_keys)
    return article_ids

# Tracking unique visitors
def record_visit(date, user_id):
    r.sadd(f"visitors:{date}", user_id)

def unique_visitors(date):
    return r.scard(f"visitors:{date}")

# Social graph: mutual followers
r.sadd("user:1:following", "2", "3", "4")
r.sadd("user:2:following", "1", "3", "5")
mutual = r.sinter("user:1:following", "user:2:following")
```

## Hash operations

### What It Is
Redis hashes are maps between string fields and string values, best thought of as a row in a relational database or a Python dict. They are ideal for representing objects with multiple fields, as they allow individual field access, update, and deletion without serializing/deserializing the entire object.

### Syntax
```python
r.hset("user:100", "name", "Alice")
r.hset("user:100", mapping={"email": "alice@example.com", "age": "30"})
r.hget("user:100", "name")          # "Alice"
r.hgetall("user:100")               # {"name": "Alice", "email": "...", "age": "30"}
r.hkeys("user:100")                 # ["name", "email", "age"]
r.hvals("user:100")                 # ["Alice", "...", "30"]
r.hexists("user:100", "email")      # True
r.hdel("user:100", "age")           # Delete field
r.hlen("user:100")                  # Number of fields
r.hincrby("user:100", "login_count", 1)
```

### Advanced Examples
```python
# Session storage
def create_session(session_id, user_data, ttl=3600):
    r.hset(f"session:{session_id}", mapping=user_data)
    r.expire(f"session:{session_id}", ttl)

def get_session(session_id):
    data = r.hgetall(f"session:{session_id}")
    if data:
        r.expire(f"session:{session_id}", 3600)  # Slide expiration
    return data

# Partial updates (useful for large objects)
r.hset("product:42", "stock", "100")
r.hincrby("product:42", "stock", -1)  # Atomic decrement

# Cache with hash field expiration simulation
def set_cached_field(key, field, value, ttl=60):
    r.hset(key, field, value)
    r.expire(key, ttl)  # Whole key expires
```

## Pub/sub

### What It Is
Redis Pub/Sub (Publish/Subscribe) is a messaging pattern where senders (publishers) send messages to channels without knowing who the receivers (subscribers) are. Subscribers express interest in one or more channels and receive messages asynchronously. This enables decoupled, real-time communication between components.

### Why It Is Important
Pub/sub enables event-driven architectures, real-time notifications, chat systems, and broadcasting. Unlike message queues (Redis lists or dedicated systems), pub/sub is fire-and-forget: if no subscriber is listening, the message is lost. This makes it suitable for ephemeral, real-time updates where delivery guarantees are not required.

### Syntax
```python
import redis
import threading

r = redis.Redis(decode_responses=True)
pubsub = r.pubsub()

# Subscribe to channels
pubsub.subscribe("news", "sports")

# Listen for messages
def listener():
    for message in pubsub.listen():
        if message["type"] == "message":
            print(f"Channel {message['channel']}: {message['data']}")

thread = threading.Thread(target=listener, daemon=True)
thread.start()

# Publish messages
r.publish("news", "Breaking news!")
r.publish("sports", "Game score update")
```

### Advanced Examples
```python
import redis
import json
import threading
from collections import defaultdict

r = redis.Redis(decode_responses=True)

# Pattern-based subscriptions
pubsub = r.pubsub()
pubsub.psubscribe("events:*")

def event_handler():
    for message in pubsub.listen():
        if message["type"] == "pmessage":
            pattern = message["pattern"]
            channel = message["channel"]
            data = json.loads(message["data"])
            print(f"Pattern: {pattern}, Channel: {channel}, Data: {data}")

threading.Thread(target=event_handler, daemon=True).start()

# Publish with different channels
r.publish("events:user_signup", json.dumps({"user_id": 1, "timestamp": "2024-01-01"}))
r.publish("events:order_placed", json.dumps({"order_id": 42, "total": 99.99}))

# Multi-threaded subscriber manager
class PubSubManager:
    def __init__(self, redis_client):
        self.r = redis_client
        self.handlers = defaultdict(list)

    def subscribe(self, channel, handler):
        self.handlers[channel].append(handler)
        pubsub = self.r.pubsub()
        pubsub.subscribe(**{channel: self._make_callback(channel)})
        threading.Thread(target=pubsub.run_in_thread, daemon=True).start()

    def _make_callback(self, channel):
        def callback(message):
            for handler in self.handlers[channel]:
                handler(message["data"])
        return callback
```

## Caching

### What It Is
Redis excels as a cache due to its in-memory performance, built-in expiration, and versatile data structures. Caching with Redis involves storing frequently accessed data so that subsequent requests can be served from memory rather than recomputing or querying a slower backend.

### Why It Is Important
Caching reduces database load, decreases response latency, and improves application throughput. Redis's `EXPIRE` command provides automatic cache invalidation, and its `LRU` eviction policies ensure efficient memory usage under capacity constraints.

### Advanced Examples
```python
import redis
import hashlib
import json
from functools import wraps

r = redis.Redis(decode_responses=True)

# Decorator-based caching
def cache(ttl=60):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            cache_key = f"cache:{func.__name__}:{hashlib.md5(str((args, kwargs)).encode()).hexdigest()}"
            cached = r.get(cache_key)
            if cached is not None:
                return json.loads(cached)
            result = func(*args, **kwargs)
            r.setex(cache_key, ttl, json.dumps(result, default=str))
            return result
        return wrapper
    return decorator

@cache(ttl=300)
def get_expensive_data(user_id, filters):
    # Simulate expensive database query
    return {"user_id": user_id, "data": "..." * 1000}

# Cache-aside pattern (lazy loading)
def get_user(user_id):
    cache_key = f"user:{user_id}"
    user_data = r.get(cache_key)
    if user_data is not None:
        return json.loads(user_data)

    # Cache miss - load from DB
    user_data = query_database(user_id)
    r.setex(cache_key, 3600, json.dumps(user_data))
    return user_data

# Cache invalidation on update
def update_user(user_id, updates):
    update_database(user_id, updates)
    r.delete(f"user:{user_id}")  # Invalidate cache
    # Optionally, write-through: set new value immediately
    # r.setex(f"user:{user_id}", 3600, json.dumps(updated_user))

# Distributed locking for cache stampede prevention
def compute_if_absent(key, compute_func, ttl=60):
    value = r.get(key)
    if value is not None:
        return json.loads(value)

    lock_key = f"lock:{key}"
    if r.setnx(lock_key, "1"):
        r.expire(lock_key, 10)  # Lock timeout
        try:
            value = compute_func()
            r.setex(key, ttl, json.dumps(value))
            return value
        finally:
            r.delete(lock_key)
    else:
        # Wait for other process to populate cache
        import time
        time.sleep(0.1)
        return compute_if_absent(key, compute_func, ttl)
```

### Real-World Use Cases
- **Session stores**: Storing web session data with automatic expiration.
- **Rate limiting**: Per-IP or per-user request counters with sliding windows.
- **Leaderboards**: Sorted sets for gaming or competitive scoring.
- **Job queues**: Using lists (`LPUSH`/`BRPOP`) for task queues (e.g., RQ, Celery).
- **Real-time analytics**: Counting page views, unique visitors, events.
- **Full-page cache**: Caching rendered HTML fragments or API responses.

### Common Mistakes
- Not setting `decode_responses=True` and working with bytes everywhere.
- Using `KEYS *` in production (blocks Redis for large key spaces).
- Forgetting to handle connection failures with retry logic.
- Storing overly large values (above 10MB) impacting performance.
- Not configuring `maxmemory` and eviction policy properly.
- Using pub/sub for reliable message delivery (use Redis streams or a proper queue).

### Best Practices
- Always use connection pools in production.
- Set `decode_responses=True` for string data.
- Use `SCAN` instead of `KEYS` for key pattern matching.
- Implement exponential backoff for reconnection.
- Configure `maxmemory` and an eviction policy (`allkeys-lru` for caches).
- Use pipelining for batch operations to reduce round trips.
- Monitor with `INFO` command and Redis metrics tools.

### Performance Considerations
- Redis is single-threaded for command execution; avoid expensive commands (`KEYS`, `SMEMBERS` on large sets).
- Pipeline reduces latency for batch operations by 5-10x.
- Smaller key names reduce memory usage (use short but readable names).
- Use `UNLINK` instead of `DEL` for large key deletion (non-blocking).
- Consider Redis Cluster for horizontal scaling beyond a single node.
- For caching, use appropriate data types: strings for simple values, hashes for objects.

### Interview Questions
1. Explain the different Redis data types and when to use each.
2. How does Redis handle persistence (RDB vs AOF)?
3. What is the difference between `SETEX` and `SET` + `EXPIRE`?
4. How do you implement distributed locking with Redis?
5. What is Redis pub/sub and how is it different from message queues?
6. Explain Redis eviction policies (LRU, LFU, TTL).
7. How would you design a rate limiter using Redis?

### Coding Challenges
1. **URL Shortener**: Implement a URL shortener service using Redis hashes and string operations.
2. **Real-time Leaderboard**: Build a gaming leaderboard using Redis sorted sets with rank queries.
3. **In-memory Cache**: Implement a decorator-based cache with TTL, max size, and LRU eviction using Redis.

### Related Topics
- Redis Streams (persistent, consumer-group-based messaging)
- Redis Cluster (sharding and high availability)
- Celery (distributed task queue using Redis as broker)
- SQLAlchemy caching with Redis
- Session management in Flask/Django with Redis
