# Redis - redis.Redis(), string/set/hash ops, pub/sub, caching

## Introduction

Redis (REmote DIctionary Server) is an open-source, in-memory data structure store used as a database, cache, message broker, and streaming engine. Python's `redis-py` library provides a comprehensive client interface for interacting with Redis servers, supporting strings, hashes, lists, sets, sorted sets, streams, and more.

## Why It Is Important

Redis is essential for high-performance applications requiring sub-millisecond response times. It excels at caching database queries, managing session state, implementing rate limiting, real-time analytics, pub/sub messaging, leaderboards, job queues, and distributed locking. Its rich data structure support makes it far more versatile than simple key-value stores like Memcached.

## Syntax

```python
import redis

r = redis.Redis(host='localhost', port=6379, db=0)

# String operations
r.set('key', 'value')
value = r.get('key')

# Hash operations
r.hset('hash', 'field', 'value')
value = r.hget('hash', 'field')

# List operations
r.lpush('list', 'item')
items = r.lrange('list', 0, -1)

# Set operations
r.sadd('set', 'member')
members = r.smembers('set')

# Pipeline
pipe = r.pipeline()
pipe.set('a', 1)
pipe.set('b', 2)
pipe.execute()
```

## Examples

### Connection and Basic Operations

```python
import redis
import json
import time
import threading
from datetime import datetime, timedelta
from typing import Optional, List, Dict, Any, Set
from redis.exceptions import RedisError, ConnectionError, TimeoutError

REDIS_CONFIG = {
    "host": "localhost",
    "port": 6379,
    "db": 0,
    "decode_responses": True,
    "socket_timeout": 5,
    "socket_connect_timeout": 3,
    "retry_on_timeout": True,
    "health_check_interval": 30
}

redis_client = redis.Redis(**REDIS_CONFIG)


def check_connection() -> bool:
    try:
        return redis_client.ping()
    except RedisError:
        return False


def string_operations():
    redis_client.set("greeting", "Hello, Redis!")
    value = redis_client.get("greeting")
    print(f"GET greeting: {value}")

    redis_client.setex("temporary", 60, "This will expire in 60 seconds")
    ttl = redis_client.ttl("temporary")
    print(f"TTL of temporary: {ttl} seconds")

    redis_client.set("counter", 0)
    redis_client.incr("counter")
    redis_client.incrby("counter", 5)
    redis_client.decr("counter")
    counter = redis_client.get("counter")
    print(f"Counter: {counter}")

    redis_client.append("greeting", " How are you?")
    full = redis_client.get("greeting")
    print(f"After append: {full}")

    length = redis_client.strlen("greeting")
    print(f"String length: {length}")
    substring = redis_client.getrange("greeting", 0, 4)
    print(f"First 5 chars: {substring}")

    exists = redis_client.exists("greeting")
    print(f"greeting exists: {bool(exists)}")

    redis_client.delete("temporary")
    print("Deleted temporary key")


def hash_operations():
    redis_client.hset("user:1000", mapping={
        "username": "alice",
        "email": "alice@example.com",
        "age": 30,
        "city": "New York"
    })

    username = redis_client.hget("user:1000", "username")
    print(f"Username: {username}")

    all_fields = redis_client.hgetall("user:1000")
    print(f"All fields: {all_fields}")

    age = redis_client.hincrby("user:1000", "age", 1)
    print(f"Age after increment: {age}")

    keys = redis_client.hkeys("user:1000")
    values = redis_client.hvals("user:1000")
    print(f"Keys: {keys}")
    print(f"Values: {values}")

    field_count = redis_client.hlen("user:1000")
    print(f"Field count: {field_count}")

    has_email = redis_client.hexists("user:1000", "email")
    print(f"Has email: {has_email}")

    redis_client.hdel("user:1000", "city")
    print("Deleted city field")

    multiple = redis_client.hmget("user:1000", ["username", "email"])
    print(f"Multiple fields: {multiple}")


def list_operations():
    redis_client.delete("queue:tasks")

    redis_client.rpush("queue:tasks", "task1", "task2", "task3")
    redis_client.lpush("queue:tasks", "urgent_task")

    length = redis_client.llen("queue:tasks")
    print(f"Queue length: {length}")

    all_tasks = redis_client.lrange("queue:tasks", 0, -1)
    print(f"All tasks: {all_tasks}")

    first = redis_client.lindex("queue:tasks", 0)
    print(f"First task: {first}")

    popped = redis_client.lpop("queue:tasks")
    print(f"Popped: {popped}")

    redis_client.ltrim("queue:tasks", 0, 1)
    trimmed = redis_client.lrange("queue:tasks", 0, -1)
    print(f"After trim: {trimmed}")

    redis_client.rpush("queue:tasks", "task4", "task5")
    redis_client.lset("queue:tasks", 0, "modified_task")
    print(f"After set: {redis_client.lrange('queue:tasks', 0, -1)}")

    position = redis_client.lpos("queue:tasks", "task4")
    print(f"Position of task4: {position}")


def set_operations():
    redis_client.sadd("skills:python", "alice", "bob", "charlie", "dave")
    redis_client.sadd("skills:java", "bob", "eve", "frank")
    redis_client.sadd("skills:javascript", "alice", "charlie", "eve")

    python_count = redis_client.scard("skills:python")
    print(f"Python developers: {python_count}")

    is_alice_python = redis_client.sismember("skills:python", "alice")
    print(f"Is Alice a Python dev? {is_alice_python}")

    intersection = redis_client.sinter("skills:python", "skills:java")
    print(f"Both Python and Java: {intersection}")

    union = redis_client.sunion("skills:python", "skills:javascript")
    print(f"Python or JavaScript: {union}")

    diff = redis_client.sdiff("skills:python", "skills:java")
    print(f"Python but not Java: {diff}")

    all_python = redis_client.smembers("skills:python")
    print(f"All Python devs: {all_python}")

    random_member = redis_client.srandmember("skills:python", 2)
    print(f"Random Python devs: {random_member}")

    popped = redis_client.spop("skills:python")
    print(f"Popped from Python set: {popped}")

    redis_client.smove("skills:python", "skills:java", "dave")
    print("Moved Dave from Python to Java")


def sorted_set_operations():
    redis_client.zadd("leaderboard", {
        "alice": 1500,
        "bob": 2300,
        "charlie": 1800,
        "dave": 2100,
        "eve": 1950
    })

    count = redis_client.zcard("leaderboard")
    print(f"Players: {count}")

    alice_rank = redis_client.zrank("leaderboard", "alice")
    print(f"Alice rank: {alice_rank}")

    alice_score = redis_client.zscore("leaderboard", "alice")
    print(f"Alice score: {alice_score}")

    top3 = redis_client.zrevrange("leaderboard", 0, 2, withscores=True)
    print(f"Top 3: {top3}")

    top3_scores = redis_client.zrevrangebyscore("leaderboard", "+inf", 2000, withscores=True)
    print(f"Scores above 2000: {top3_scores}")

    redis_client.zincrby("leaderboard", 100, "charlie")
    charlie_score = redis_client.zscore("leaderboard", "charlie")
    print(f"Charlie new score: {charlie_score}")

    players_in_range = redis_client.zcount("leaderboard", 1700, 2200)
    print(f"Players between 1700-2200: {players_in_range}")

    redis_client.zrem("leaderboard", "eve")
    print("Removed Eve")


def key_expiry_and_ttl():
    redis_client.setex("session:token:abc123", 3600, "user_data")
    ttl = redis_client.ttl("session:token:abc123")
    print(f"TTL: {ttl}s")

    redis_client.expire("greeting", 300)
    new_ttl = redis_client.ttl("greeting")
    print(f"New TTL: {new_ttl}s")

    no_ttl = redis_client.ttl("counter")
    print(f"No TTL set: {no_ttl}")

    persistent = redis_client.persist("greeting")
    print(f"Removed expiry from greeting: {persistent}")

    redis_client.expireat("session:token:abc123", int(time.time()) + 86400)
    print("Set expiry at specific timestamp")


def main():
    if not check_connection():
        print("Redis is not running. Please start Redis.")
        return

    redis_client.flushdb()
    print("Flushed current database\n")

    print("=== String Operations ===")
    string_operations()
    print()

    print("=== Hash Operations ===")
    hash_operations()
    print()

    print("=== List Operations ===")
    list_operations()
    print()

    print("=== Set Operations ===")
    set_operations()
    print()

    print("=== Sorted Set Operations ===")
    sorted_set_operations()
    print()

    print("=== Key Expiry ===")
    key_expiry_and_ttl()

    redis_client.flushdb()


if __name__ == "__main__":
    main()
```

### Pipeline and Transactions

```python
import redis
from redis.exceptions import WatchError

r = redis.Redis(decode_responses=True)


def pipeline_example():
    pipe = r.pipeline()

    pipe.set("key1", "value1")
    pipe.set("key2", "value2")
    pipe.get("key1")
    pipe.get("key2")
    pipe.hset("hash1", "field1", "val1")
    pipe.hgetall("hash1")
    pipe.zadd("zset1", {"a": 1, "b": 2})
    pipe.zcard("zset1")

    results = pipe.execute()
    print("Pipeline results:", results)


def atomic_transaction_example():
    with r.pipeline(transaction=True) as pipe:
        pipe.set("account:alice", 100)
        pipe.set("account:bob", 50)
        pipe.execute()

    with r.pipeline(transaction=True) as pipe:
        pipe.decrby("account:alice", 30)
        pipe.incrby("account:bob", 30)
        results = pipe.execute()
        print("Transfer results:", results)
        print(f"Alice: {r.get('account:alice')}, Bob: {r.get('account:bob')}")


def optimistic_locking_example():
    r.set("inventory:item1", 10)

    while True:
        r.watch("inventory:item1")
        current = int(r.get("inventory:item1"))
        if current < 1:
            print("Out of stock")
            r.unwatch()
            break
        pipe = r.pipeline(transaction=True)
        pipe.decr("inventory:item1")
        try:
            pipe.execute()
            print(f"Purchase successful. Remaining: {current - 1}")
            break
        except WatchError:
            print("Retrying due to concurrent modification")
            continue


def bulk_insert_example():
    pipe = r.pipeline()
    for i in range(1000):
        pipe.set(f"bulk:key:{i}", f"value_{i}")
    pipe.execute()
    print("Inserted 1000 keys")


def conditional_update_example():
    r.set("counter", 5)
    pipe = r.pipeline()
    pipe.watch("counter")
    current = int(pipe.get("counter"))
    if current > 0:
        pipe.multi()
        pipe.decr("counter")
        pipe.set("last_decrement", str(time.time()))
        pipe.execute()
        print(f"Decremented counter from {current} to {current - 1}")
    else:
        pipe.unwatch()
        print("Counter is already zero")


pipeline_example()
atomic_transaction_example()
optimistic_locking_example()
bulk_insert_example()
conditional_update_example()
```

### Pub/Sub

```python
import redis
import threading
import time
import json

r = redis.Redis(decode_responses=True)


def publisher():
    time.sleep(0.5)
    channels = ["news:sports", "news:tech", "alerts"]

    for i in range(5):
        msg = json.dumps({"id": i, "text": f"Sports news #{i}", "timestamp": time.time()})
        r.publish("news:sports", msg)
        time.sleep(0.3)

    for i in range(3):
        msg = json.dumps({"id": i, "text": f"Tech news #{i}", "timestamp": time.time()})
        r.publish("news:tech", msg)
        time.sleep(0.3)

    msg = json.dumps({"severity": "critical", "message": "Server load high!"})
    r.publish("alerts", msg)


def subscriber(channel: str, name: str):
    pubsub = r.pubsub()
    pubsub.subscribe(channel)
    print(f"{name}: subscribed to {channel}")

    for message in pubsub.listen():
        if message["type"] == "message":
            data = json.loads(message["data"])
            print(f"{name}: received on {channel}: {data}")
        if message["type"] == "unsubscribe":
            break

    pubsub.close()


def pattern_subscriber(pattern: str, name: str):
    pubsub = r.pubsub()
    pubsub.psubscribe(pattern)
    print(f"{name}: subscribed to pattern {pattern}")

    count = 0
    for message in pubsub.listen():
        if message["type"] == "pmessage":
            data = json.loads(message["data"])
            print(f"{name}: pattern matched {message['channel']}: {data}")
            count += 1
            if count >= 3:
                pubsub.punsubscribe(pattern)
                break

    pubsub.close()


pub_thread = threading.Thread(target=publisher)
sub_thread1 = threading.Thread(target=subscriber, args=("news:sports", "Sub1"))
sub_thread2 = threading.Thread(target=subscriber, args=("news:tech", "Sub2"))
sub_thread3 = threading.Thread(target=pattern_subscriber, args=("news:*", "PatternSub"))

sub_thread1.start()
sub_thread2.start()
sub_thread3.start()
pub_thread.start()
pub_thread.join()
time.sleep(1)
pub_thread = threading.Thread(target=publisher)
pub_thread.start()
pub_thread.join()
time.sleep(1)
```

### Redis as Cache

```python
import redis
import json
import time
import hashlib
from functools import wraps
from typing import Callable, Any

r = redis.Redis(decode_responses=True)

CACHE_PREFIX = "cache:v1:"
DEFAULT_TTL = 300


def make_cache_key(*args, **kwargs) -> str:
    key_parts = [str(a) for a in args]
    key_parts.extend(f"{k}:{v}" for k, v in sorted(kwargs.items()))
    raw = ":".join(key_parts)
    return CACHE_PREFIX + hashlib.md5(raw.encode()).hexdigest()


def cache(ttl: int = DEFAULT_TTL):
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs):
            cache_key = make_cache_key(func.__name__, *args, **kwargs)
            cached = r.get(cache_key)
            if cached is not None:
                print(f"CACHE HIT: {cache_key}")
                return json.loads(cached)

            print(f"CACHE MISS: {cache_key}")
            result = func(*args, **kwargs)
            r.setex(cache_key, ttl, json.dumps(result))
            return result
        return wrapper
    return decorator


class CacheManager:
    def __init__(self, client: redis.Redis, prefix: str = "app:"):
        self.client = client
        self.prefix = prefix

    def get(self, key: str) -> Any:
        data = self.client.get(f"{self.prefix}{key}")
        return json.loads(data) if data else None

    def set(self, key: str, value: Any, ttl: int = DEFAULT_TTL):
        self.client.setex(f"{self.prefix}{key}", ttl, json.dumps(value))

    def delete(self, key: str):
        self.client.delete(f"{self.prefix}{key}")

    def clear_pattern(self, pattern: str):
        cursor = 0
        while True:
            cursor, keys = self.client.scan(cursor=cursor, match=f"{self.prefix}{pattern}", count=100)
            if keys:
                self.client.delete(*keys)
            if cursor == 0:
                break

    def get_or_set(self, key: str, func: Callable, ttl: int = DEFAULT_TTL) -> Any:
        cached = self.get(key)
        if cached is not None:
            return cached
        value = func()
        self.set(key, value, ttl)
        return value

    def increment(self, key: str, amount: int = 1) -> int:
        return self.client.incrby(f"{self.prefix}{key}", amount)

    def add_to_set(self, key: str, *members: str):
        return self.client.sadd(f"{self.prefix}{key}", *members)

    def get_set_members(self, key: str) -> set:
        return self.client.smembers(f"{self.prefix}{key}")

    def exists(self, key: str) -> bool:
        return self.client.exists(f"{self.prefix}{key}") > 0

    def ttl(self, key: str) -> int:
        return self.client.ttl(f"{self.prefix}{key}")

    def clear_all(self):
        self.client.flushdb()


cache_mgr = CacheManager(redis_client, prefix="myapp:")


@cache(ttl=60)
def get_user_data(user_id: int) -> dict:
    time.sleep(2)
    return {
        "id": user_id,
        "name": f"User {user_id}",
        "email": f"user{user_id}@example.com",
        "role": "member",
        "last_login": time.time()
    }


def cache_aside_demo():
    user1 = get_user_data(1)
    print(f"First call (miss): {user1['name']}")

    user1_cached = get_user_data(1)
    print(f"Second call (hit): {user1_cached['name']}")

    cache_mgr.set("config:theme", {"primary": "#007bff", "secondary": "#6c757d"})
    config = cache_mgr.get("config:theme")
    print(f"Cached config: {config}")

    hot_data = cache_mgr.get_or_set(
        "analytics:daily",
        lambda: {"views": 1500, "visitors": 450, "date": str(time.time())},
        ttl=120
    )
    print(f"Hot data: {hot_data}")


def rate_limiter_demo():
    user_ip = "192.168.1.1"
    key = f"ratelimit:{user_ip}"
    max_requests = 5
    window = 60

    current = r.get(key)
    if current and int(current) >= max_requests:
        ttl = r.ttl(key)
        print(f"Rate limited. Try again in {ttl} seconds")
    else:
        count = r.incr(key)
        if count == 1:
            r.expire(key, window)
        print(f"Request {count}/{max_requests} allowed")

    for _ in range(6):
        current = r.get(key)
        if current and int(current) >= max_requests:
            ttl = r.ttl(key)
            print(f"Rate limited after {_} requests")
            break
        r.incr(key)


def session_store_demo():
    session_id = "sess_" + hashlib.md5(str(time.time()).encode()).hexdigest()
    session_data = {
        "user_id": 42,
        "username": "alice",
        "roles": ["admin", "editor"],
        "logged_in_at": time.time()
    }

    r.setex(f"session:{session_id}", 3600, json.dumps(session_data))
    print(f"Session stored: {session_id}")

    data = json.loads(r.get(f"session:{session_id}"))
    print(f"Session retrieved: {data['username']}")

    r.expire(f"session:{session_id}", 30)
    print(f"Session TTL reset to 30s")


cache_aside_demo()
rate_limiter_demo()
session_store_demo()
```

## Beginner Examples

```python
import redis

r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

r.set('name', 'Alice')
print(r.get('name'))

r.hset('person', 'name', 'Bob')
r.hset('person', 'age', '25')
print(r.hgetall('person'))

r.rpush('fruits', 'apple', 'banana', 'cherry')
print(r.lrange('fruits', 0, -1))

r.sadd('colors', 'red', 'green', 'blue')
print(r.smembers('colors'))

r.zadd('scores', {'Alice': 95, 'Bob': 87, 'Charlie': 92})
print(r.zrevrange('scores', 0, -1, withscores=True))

r.setex('temp_key', 10, 'I will expire')
print(r.get('temp_key'))
time.sleep(11)
print(r.get('temp_key'))
```

## Intermediate Examples

```python
import redis
import json
import time

r = redis.Redis(decode_responses=True)


def implement_cache_with_patterns():
    prefix = "products:"
    products = [
        {"id": 1, "name": "Laptop", "price": 999.99},
        {"id": 2, "name": "Phone", "price": 499.99},
    ]

    for p in products:
        r.set(f"{prefix}{p['id']}", json.dumps(p))

    r.set(f"{prefix}all", json.dumps(products))

    keys = r.keys("products:*")
    print(f"Product keys: {keys}")

    cursor = 0
    all_keys = []
    while True:
        cursor, scan_keys = r.scan(cursor=cursor, match="products:*", count=10)
        all_keys.extend(scan_keys)
        if cursor == 0:
            break
    print(f"Scanned keys: {all_keys}")


def implement_leaderboard():
    game_scores = {"alice": 2500, "bob": 3100, "charlie": 2800, "dave": 2200}

    for player, score in game_scores.items():
        r.zadd("game:leaderboard", {player: score})

    r.zincrby("game:leaderboard", 500, "alice")

    top3 = r.zrevrange("game:leaderboard", 0, 2, withscores=True)
    print(f"Leaderboard top 3: {top3}")

    rank = r.zrevrank("game:leaderboard", "alice")
    print(f"Alice rank: {rank + 1}")

    around_player = r.zrevrangebyscore(
        "game:leaderboard", "+inf", "-inf",
        offset=0, count=5, withscores=True
    )
    print(f"Top 5: {around_player}")


def implement_message_queue():
    r.delete("job:queue")
    jobs = [f"job_{i}" for i in range(10)]

    for job in jobs:
        r.rpush("job:queue", job)

    worker_id = 1
    while r.llen("job:queue") > 0:
        job = r.blpop("job:queue", timeout=5)
        if job:
            print(f"Worker {worker_id}: processing {job[1]}")
            time.sleep(0.1)

    r.rpush("job:queue", "priority_job")
    r.lpush("job:queue", "urgent_job")

    next_job = r.brpoplpush("job:queue", "job:processing", timeout=5)
    print(f"Next job: {next_job}")


implement_cache_with_patterns()
implement_leaderboard()
implement_message_queue()
```

## Advanced Examples

```python
import redis
import json
import time
import asyncio
import threading
from typing import Optional, Callable, Any
from collections import defaultdict


class DistributedLock:
    def __init__(self, client: redis.Redis, lock_name: str, ttl: int = 10):
        self.client = client
        self.lock_name = f"lock:{lock_name}"
        self.ttl = ttl
        self.lock_value = None

    def acquire(self, blocking: bool = True, timeout: float = 10) -> bool:
        import uuid
        self.lock_value = str(uuid.uuid4())
        deadline = time.time() + timeout if blocking else None

        while True:
            acquired = self.client.set(
                self.lock_name, self.lock_value,
                nx=True, ex=self.ttl
            )
            if acquired:
                return True
            if not blocking:
                return False
            if deadline and time.time() >= deadline:
                return False
            time.sleep(0.1)

    def release(self) -> bool:
        if not self.lock_value:
            return False
        lua_script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """
        result = self.client.eval(lua_script, 1, self.lock_name, self.lock_value)
        return bool(result)

    def __enter__(self):
        self.acquire()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.release()


class BloomFilter:
    def __init__(self, client: redis.Redis, key: str, capacity: int = 10000, error_rate: float = 0.01):
        self.client = client
        self.key = key
        self.capacity = capacity
        self.error_rate = error_rate
        self._init_filter()

    def _init_filter(self):
        try:
            self.client.execute_command("BF.RESERVE", self.key, self.error_rate, self.capacity)
        except redis.ResponseError:
            pass

    def add(self, item: str) -> bool:
        return self.client.execute_command("BF.ADD", self.key, item)

    def exists(self, item: str) -> bool:
        return self.client.execute_command("BF.EXISTS", self.key, item)

    def add_batch(self, *items: str) -> list:
        pipe = self.client.pipeline()
        for item in items:
            pipe.execute_command("BF.ADD", self.key, item)
        return pipe.execute()


class RateLimiter:
    def __init__(self, client: redis.Redis):
        self.client = client

    def sliding_window(self, key: str, max_requests: int, window_seconds: int) -> tuple:
        now = time.time()
        window_start = now - window_seconds
        pipe = self.client.pipeline()
        pipe.zremrangebyscore(key, 0, window_start)
        pipe.zcard(key)
        pipe.zadd(key, {str(now): now})
        pipe.expire(key, window_seconds)
        _, current_count, _, _ = pipe.execute()

        allowed = current_count <= max_requests
        return allowed, current_count

    def token_bucket(self, key: str, capacity: int, refill_rate: float) -> tuple:
        now = time.time()
        lua_script = """
        local key = KEYS[1]
        local now = tonumber(ARGV[1])
        local capacity = tonumber(ARGV[2])
        local refill_rate = tonumber(ARGV[3])

        local tokens = redis.call("hget", key, "tokens")
        local last_refill = redis.call("hget", key, "last_refill")

        if not tokens then
            tokens = capacity
            last_refill = now
        else
            tokens = tonumber(tokens)
            last_refill = tonumber(last_refill)
            local elapsed = now - last_refill
            local new_tokens = elapsed * refill_rate
            tokens = math.min(capacity, tokens + new_tokens)
        end

        local allowed = 0
        if tokens >= 1 then
            tokens = tokens - 1
            allowed = 1
        end

        redis.call("hmset", key, "tokens", tokens, "last_refill", now)
        redis.call("expire", key, math.ceil(capacity / refill_rate) + 1)
        return {allowed, tokens}
        """
        result = self.client.eval(lua_script, 1, key, now, capacity, refill_rate)
        return bool(result[0]), float(result[1])


class RedisStreamProducer:
    def __init__(self, client: redis.Redis, stream: str):
        self.client = client
        self.stream = stream

    def add(self, fields: dict, maxlen: int = 1000) -> str:
        return self.client.xadd(self.stream, fields, maxlen=maxlen)

    def add_batch(self, messages: list, maxlen: int = 1000) -> list:
        pipe = self.client.pipeline()
        for msg in messages:
            pipe.xadd(self.stream, msg, maxlen=maxlen)
        return pipe.execute()


class RedisStreamConsumer:
    def __init__(self, client: redis.Redis, stream: str, group: str, consumer: str):
        self.client = client
        self.stream = stream
        self.group = group
        self.consumer = consumer

    def create_group(self, id: str = "0"):
        try:
            self.client.xgroup_create(self.stream, self.group, id=id, mkstream=True)
        except redis.ResponseError as e:
            if "BUSYGROUP" not in str(e):
                raise

    def read(self, count: int = 10, block: int = 5000) -> list:
        return self.client.xreadgroup(
            self.group, self.consumer,
            {self.stream: ">"},
            count=count, block=block
        )

    def acknowledge(self, *message_ids: str):
        self.client.xack(self.stream, self.group, *message_ids)

    def pending(self, count: int = 10) -> list:
        return self.client.xpending_range(
            self.stream, self.group,
            min="-", max="+",
            count=count
        )

    def claim(self, consumer: str, min_idle_time: int, *message_ids: str):
        return self.client.xclaim(
            self.stream, self.group, consumer,
            min_idle_time, message_ids
        )


class HyperLogLog:
    def __init__(self, client: redis.Redis, key: str):
        self.client = client
        self.key = key

    def add(self, *elements: str):
        self.client.pfadd(self.key, *elements)

    def count(self) -> int:
        return self.client.pfcount(self.key)

    def merge(self, *source_keys: str):
        self.client.pfmerge(self.key, *source_keys)


def advanced_examples():
    r = redis.Redis(decode_responses=True)
    r.flushdb()

    with DistributedLock(r, "resource:critical") as lock:
        print(f"Acquired lock: {lock.lock_value}")
        counter = r.get("critical:counter") or 0
        r.set("critical:counter", int(counter) + 1)
    print("Lock released")

    bf = BloomFilter(r, "bloom:usernames")
    bf.add("alice")
    bf.add("bob")
    print(f"alice exists: {bf.exists('alice')}")
    print(f"charlie exists: {bf.exists('charlie')}")
    bf.add_batch("dave", "eve", "frank")

    limiter = RateLimiter(r)
    for i in range(6):
        allowed, tokens = limiter.token_bucket("apikey:abc", 5, 1)
        print(f"Request {i+1}: allowed={allowed}, tokens={tokens:.2f}")

    producer = RedisStreamProducer(r, "mystream")
    for i in range(10):
        msg_id = producer.add({"event": f"event_{i}", "data": f"payload_{i}"})
        print(f"Published: {msg_id}")

    consumer = RedisStreamConsumer(r, "mystream", "mygroup", "consumer1")
    consumer.create_group()
    messages = consumer.read(count=5, block=2000)
    print(f"Consumed: {len(messages) if messages else 0} messages")

    hll = HyperLogLog(r, "hll:visitors")
    for i in range(10000):
        hll.add(f"visitor_{i}")
    print(f"Unique visitors (approx): {hll.count()}")

    r.flushdb()


advanced_examples()
```

## Real-World Use Cases

Redis is used for session management in web applications, caching database queries, real-time leaderboards, rate limiting APIs, message queues with pub/sub or streams, distributed locking, feature flags, real-time analytics, chat systems, job queues (RQ, Celery), and geospatial queries.

## Common Mistakes

- Not setting `decode_responses=True` when working with string data (results in bytes).
- Using `keys()` in production (blocks Redis; use `scan()` instead).
- Forgetting to handle connection errors and reconnections.
- Not setting TTL on cache keys, causing memory exhaustion.
- Storing large values in Redis (designed for small, fast data).
- Using Redis as a primary database for complex relational data.
- Ignoring pipeline transactions for atomicity when needed.

## Best Practices

- Always set TTL/expiry for cache keys to prevent memory leaks.
- Use pipelines to batch multiple commands and reduce round trips.
- Use `scan()` instead of `keys()` for key pattern matching.
- Set `decode_responses=True` for string data.
- Implement connection pooling with `redis.ConnectionPool`.
- Use appropriate data structures for the task (e.g., hashes for objects, sorted sets for leaderboards).
- Monitor memory usage and set `maxmemory` with an eviction policy.
- Use Redis Streams instead of pub/sub for reliable message delivery.

## Interview Questions

1. What data structures does Redis support? — Strings, hashes, lists, sets, sorted sets, bitmaps, hyperloglogs, geospatial indexes, streams, and Bloom filters (via modules).
2. How does Redis handle persistence? — RDB snapshots (point-in-time) and AOF logs (append-only file); can use both or either.
3. Explain Redis pub/sub vs streams. — Pub/sub is fire-and-forget with no message persistence; streams provide persistent, consumer-group-based messaging with acknowledgment.
4. What is a Redis pipeline and when would you use it? — Batches multiple commands to reduce network round trips; use when executing many independent commands.
5. How does Redis handle expiration? — Passive (key accessed while expired) and active (periodic sampling) expiration mechanisms.

## Coding Challenges

1. **URL Shortener** — Build a URL shortener using Redis hashes, with TTL-based expiry and click tracking with sorted sets.
2. **Real-Time Chat** — Implement a chat system using Redis pub/sub for message distribution and streams for message history.
3. **Distributed Rate Limiter** — Implement sliding window and token bucket rate limiters with Redis for API protection.
4. **Shopping Cart** — Build a shopping cart using Redis hashes with TTL-based expiry and session management.
5. **Real-Time Leaderboard** — Implement a gaming leaderboard with sorted sets, including rank updates, top-N queries, and player rank lookups.

## Summary

Redis is an in-memory data structure store that provides sub-millisecond performance for caching, messaging, and real-time applications. The `redis-py` library supports all Redis data types, pipelines for batch operations, pub/sub for messaging, streams for reliable message queues, and features like TTL, transactions, and Lua scripting for advanced use cases.

## Related Topics

- Redis Stack (Redis with modules: JSON, Search, TimeSeries, Bloom)
- Celery / RQ (task queues backed by Redis)
- Redis Sentinel / Cluster (high availability and scaling)
- Memcached (alternative caching solution)
- Asyncio Redis (async Redis clients like aioredis)
- Django/Flask/FastAPI session backends (Redis-backed sessions)
