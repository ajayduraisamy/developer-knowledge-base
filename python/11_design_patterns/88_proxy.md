# Proxy - Virtual, protection, remote, caching proxy patterns

## Introduction

The Proxy pattern provides a surrogate or placeholder for another object to control access to it. It creates a representative object that acts as an intermediary between a client and the real subject. There are several types including virtual proxy (lazy loading), protection proxy (access control), remote proxy (network communication), and caching proxy (result caching).

## Why It Is Important

Proxies enable lazy initialization of expensive objects, controlled access to sensitive resources, transparent remote communication, and performance optimization through caching. They follow the Open/Closed Principle by adding functionality without modifying the real subject. Proxies are essential in ORMs (lazy loading), RPC frameworks, authentication layers, and caching systems.

## Syntax

A proxy implements the same interface as the real subject and holds a reference to it. The proxy intercepts client calls and performs additional operations (lazy initialization, access control, logging, caching) before delegating to the real subject.

## Examples

```python
# Virtual Proxy — lazy loading
from abc import ABC, abstractmethod
import time


class Image(ABC):
    @abstractmethod
    def display(self) -> str:
        pass


class HighResolutionImage(Image):
    def __init__(self, filename: str):
        self._filename = filename
        self._load_image()

    def _load_image(self):
        print(f"Loading high-resolution image: {self._filename}")
        time.sleep(1)

    def display(self) -> str:
        return f"Displaying {self._filename}"


class ImageProxy(Image):
    def __init__(self, filename: str):
        self._filename = filename
        self._real_image = None

    def display(self) -> str:
        if self._real_image is None:
            self._real_image = HighResolutionImage(self._filename)
        return self._real_image.display()


print("Creating proxy (no image loaded yet)")
proxy = ImageProxy("photo.jpg")
print("Calling display (image loads now)")
print(proxy.display())
print("Calling display again (image already loaded)")
print(proxy.display())
```

```python
# Protection Proxy — access control
class Document(ABC):
    @abstractmethod
    def read(self) -> str:
        pass

    @abstractmethod
    def write(self, content: str) -> None:
        pass


class SecureDocument(Document):
    def __init__(self, content: str = ""):
        self._content = content

    def read(self) -> str:
        return self._content

    def write(self, content: str) -> None:
        self._content = content
        print("Document updated")


class DocumentProxy(Document):
    def __init__(self, document: Document, user_role: str):
        self._document = document
        self._user_role = user_role
        self._permissions = {
            "admin": {"read", "write"},
            "editor": {"read", "write"},
            "viewer": {"read"},
        }

    def _check_access(self, operation: str) -> bool:
        allowed = self._permissions.get(self._user_role, set())
        if operation not in allowed:
            raise PermissionError(f"{self._user_role} cannot {operation}")
        return True

    def read(self) -> str:
        self._check_access("read")
        print(f"[AUDIT] {self._user_role} read document")
        return self._document.read()

    def write(self, content: str) -> None:
        self._check_access("write")
        print(f"[AUDIT] {self._user_role} wrote to document")
        self._document.write(content)


doc = SecureDocument("Secret plans")
admin_proxy = DocumentProxy(doc, "admin")
viewer_proxy = DocumentProxy(doc, "viewer")

print(admin_proxy.read())
admin_proxy.write("Updated plans")
print(viewer_proxy.read())
try:
    viewer_proxy.write("Hacker attempt")
except PermissionError as e:
    print(f"Blocked: {e}")
```

```python
# Caching Proxy
class DataFetcher(ABC):
    @abstractmethod
    def fetch(self, url: str) -> str:
        pass


class HTTPDataFetcher(DataFetcher):
    def fetch(self, url: str) -> str:
        print(f"Fetching {url} from network...")
        import time
        time.sleep(0.5)
        return f"Response from {url}"


class CachingProxy(DataFetcher):
    def __init__(self, fetcher: DataFetcher):
        self._fetcher = fetcher
        self._cache = {}

    def fetch(self, url: str) -> str:
        if url in self._cache:
            print(f"Cache HIT for {url}")
            return self._cache[url]
        print(f"Cache MISS for {url}")
        response = self._fetcher.fetch(url)
        self._cache[url] = response
        return response


proxy = CachingProxy(HTTPDataFetcher())
print(proxy.fetch("https://api.example.com/data"))
print(proxy.fetch("https://api.example.com/data"))
print(proxy.fetch("https://api.example.com/other"))
```

## Beginner Examples

```python
# Simple logging proxy
class Calculator(ABC):
    @abstractmethod
    def add(self, a: int, b: int) -> int:
        pass

    @abstractmethod
    def multiply(self, a: int, b: int) -> int:
        pass


class RealCalculator(Calculator):
    def add(self, a: int, b: int) -> int:
        return a + b

    def multiply(self, a: int, b: int) -> int:
        return a * b


class LoggingProxy(Calculator):
    def __init__(self, calculator: Calculator):
        self._calculator = calculator

    def add(self, a: int, b: int) -> int:
        result = self._calculator.add(a, b)
        print(f"[LOG] add({a}, {b}) = {result}")
        return result

    def multiply(self, a: int, b: int) -> int:
        result = self._calculator.multiply(a, b)
        print(f"[LOG] multiply({a}, {b}) = {result}")
        return result


calc = LoggingProxy(RealCalculator())
calc.add(3, 4)
calc.multiply(5, 6)
```

```python
# Virtual proxy for database queries
class UserRepository:
    def get_user(self, user_id: int) -> dict:
        print(f"Database query: SELECT * FROM users WHERE id = {user_id}")
        return {"id": user_id, "name": "Alice", "email": "alice@example.com"}


class LazyUserProxy:
    def __init__(self, user_id: int):
        self._user_id = user_id
        self._user = None

    def get(self) -> dict:
        if self._user is None:
            self._user = UserRepository().get_user(self._user_id)
        return self._user


proxy = LazyUserProxy(42)
print("Proxy created, no query yet")
user = proxy.get()
print(user["name"])
user = proxy.get()  # Cached
```

## Intermediate Examples

```python
# Remote Proxy (RPC simulation)
import json
from typing import Any


class RemoteService:
    def process_data(self, data: dict) -> dict:
        return {"result": f"Processed: {data}", "status": "ok"}

    def compute(self, x: int, y: int) -> int:
        return x * y + x - y


class RemoteProxy:
    def __init__(self, host: str = "localhost", port: int = 8080):
        self._host = host
        self._port = port
        self._connection_id = None

    def _connect(self):
        if self._connection_id is None:
            self._connection_id = f"conn-{self._host}-{self._port}"
            print(f"Connecting to {self._host}:{self._port}...")
        return self._connection_id

    def _call(self, method: str, *args, **kwargs):
        self._connect()
        print(f"Remote call: {method}(args={args}, kwargs={kwargs})")
        service = RemoteService()
        result = getattr(service, method)(*args, **kwargs)
        return result

    def process_data(self, data: dict) -> dict:
        return self._call("process_data", data)

    def compute(self, x: int, y: int) -> int:
        return self._call("compute", x, y)


proxy = RemoteProxy()
result = proxy.process_data({"message": "hello"})
print(result)
print(proxy.compute(5, 3))
```

```python
# Virtual proxy with resource pooling
from typing import List, Optional


class DatabaseConnection:
    def __init__(self, conn_id: str):
        self._id = conn_id
        print(f"Creating database connection {conn_id}")

    def query(self, sql: str) -> str:
        return f"Result from {self._id}: {sql}"

    def close(self):
        print(f"Closing connection {self._id}")


class ConnectionPoolProxy:
    def __init__(self, pool_size: int = 3):
        self._pool: List[DatabaseConnection] = []
        self._in_use: set = set()
        self._pool_size = pool_size

    def _acquire(self) -> DatabaseConnection:
        for conn in self._pool:
            if id(conn) not in self._in_use:
                self._in_use.add(id(conn))
                return conn
        if len(self._pool) < self._pool_size:
            conn = DatabaseConnection(f"conn-{len(self._pool) + 1}")
            self._pool.append(conn)
            self._in_use.add(id(conn))
            return conn
        raise Exception("No available connections")

    def _release(self, conn: DatabaseConnection):
        self._in_use.discard(id(conn))

    def query(self, sql: str) -> str:
        conn = self._acquire()
        try:
            return conn.query(sql)
        finally:
            self._release(conn)


pool = ConnectionPoolProxy(2)
print(pool.query("SELECT 1"))
print(pool.query("SELECT 2"))
print(pool.query("SELECT 3"))
```

```python
# Synchronization proxy for thread safety
import threading
from typing import Any


class Counter:
    def __init__(self):
        self._count = 0

    def increment(self) -> int:
        self._count += 1
        return self._count

    def decrement(self) -> int:
        self._count -= 1
        return self._count

    def value(self) -> int:
        return self._count


class ThreadSafeProxy:
    def __init__(self, target: Counter):
        self._target = target
        self._lock = threading.Lock()

    def increment(self) -> int:
        with self._lock:
            return self._target.increment()

    def decrement(self) -> int:
        with self._lock:
            return self._target.decrement()

    def value(self) -> int:
        with self._lock:
            return self._target.value()


counter = ThreadSafeProxy(Counter())

def worker():
    for _ in range(1000):
        counter.increment()
        counter.decrement()

threads = [threading.Thread(target=worker) for _ in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()
print(f"Final count: {counter.value()}")
```

## Advanced Examples

```python
# Caching proxy with TTL and LRU eviction
import time
from typing import Any, Optional
from collections import OrderedDict


class TTLCacheProxy:
    def __init__(self, target, ttl: int = 60, max_size: int = 100):
        self._target = target
        self._ttl = ttl
        self._max_size = max_size
        self._cache = OrderedDict()
        self._timestamps = {}

    def _is_expired(self, key: str) -> bool:
        if key not in self._timestamps:
            return True
        return time.time() - self._timestamps[key] > self._ttl

    def _evict_if_needed(self):
        while len(self._cache) >= self._max_size:
            oldest_key, _ = self._cache.popitem(last=False)
            del self._timestamps[oldest_key]

    def _get_cached(self, key: str) -> Optional[Any]:
        if key in self._cache and not self._is_expired(key):
            self._cache.move_to_end(key)
            return self._cache[key]
        if key in self._cache:
            del self._cache[key]
            del self._timestamps[key]
        return None

    def _set_cached(self, key: str, value: Any):
        self._evict_if_needed()
        self._cache[key] = value
        self._timestamps[key] = time.time()

    def fetch_data(self, query: str) -> str:
        cached = self._get_cached(query)
        if cached is not None:
            print(f"[CACHE] Hit for '{query}'")
            return cached
        print(f"[CACHE] Miss for '{query}'")
        result = self._target.fetch_data(query)
        self._set_cached(query, result)
        return result


class ExpensiveDataSource:
    def fetch_data(self, query: str) -> str:
        time.sleep(0.5)
        return f"Result for: {query}"


proxy = TTLCacheProxy(ExpensiveDataSource(), ttl=10, max_size=3)
print(proxy.fetch_data("SELECT * FROM users"))
print(proxy.fetch_data("SELECT * FROM users"))
print(proxy.fetch_data("SELECT * FROM orders"))
print(proxy.fetch_data("SELECT * FROM products"))
print(proxy.fetch_data("SELECT * FROM users"))
```

```python
# Protection proxy with rate limiting
import time
from typing import Dict, List
from collections import defaultdict


class RateLimitingProxy:
    def __init__(self, target, max_requests: int = 10, window_seconds: int = 60):
        self._target = target
        self._max_requests = max_requests
        self._window = window_seconds
        self._requests: Dict[str, List[float]] = defaultdict(list)

    def _check_rate_limit(self, client_id: str) -> bool:
        now = time.time()
        self._requests[client_id] = [
            t for t in self._requests[client_id] if now - t < self._window
        ]
        return len(self._requests[client_id]) < self._max_requests

    def _record_request(self, client_id: str):
        self._requests[client_id].append(time.time())

    def process(self, client_id: str, data: Any) -> Any:
        if not self._check_rate_limit(client_id):
            raise Exception(f"Rate limit exceeded for {client_id}")
        self._record_request(client_id)
        return self._target.process(data)


class APIHandler:
    def process(self, data: Any) -> str:
        return f"Processed: {data}"


api = RateLimitingProxy(APIHandler(), max_requests=3, window_seconds=10)
for i in range(5):
    try:
        result = api.process("client-1", f"request-{i}")
        print(result)
    except Exception as e:
        print(f"Error: {e}")
```

```python
# Virtual proxy with progress tracking
import time
from typing import Callable


class ProgressTrackingProxy:
    def __init__(self, target, progress_callback: Callable):
        self._target = target
        self._callback = progress_callback

    def process_large_file(self, filename: str) -> str:
        print(f"Starting to process {filename}")
        total_steps = 5
        for step in range(total_steps):
            time.sleep(0.2)
            progress = (step + 1) / total_steps * 100
            self._callback(filename, progress)
        return self._target.process_large_file(filename)


class FileProcessor:
    def process_large_file(self, filename: str) -> str:
        return f"Processed {filename}"


def show_progress(filename: str, percent: float):
    bar = "#" * int(percent / 10) + "-" * (10 - int(percent / 10))
    print(f"[{bar}] {percent:.0f}%")


processor = ProgressTrackingProxy(FileProcessor(), show_progress)
result = processor.process_large_file("data.csv")
print(result)
```

```python
# Composite proxy — combining virtual + protection + caching
class SmartProxy:
    def __init__(self, real_subject, user_role: str = "viewer"):
        self._real = real_subject
        self._role = user_role
        self._cache = {}
        self._initialized = False

    def _lazy_init(self):
        if not self._initialized:
            print("[PROXY] Initializing real subject...")
            self._initialized = True

    def _check_access(self, operation: str) -> bool:
        permissions = {
            "admin": {"read", "write", "delete"},
            "editor": {"read", "write"},
            "viewer": {"read"},
        }
        allowed = permissions.get(self._role, set())
        if operation not in allowed:
            raise PermissionError(f"{self._role} cannot {operation}")
        return True

    def get_data(self, key: str) -> str:
        self._lazy_init()
        self._check_access("read")
        if key in self._cache:
            print(f"[PROXY] Cache hit for {key}")
            return self._cache[key]
        print(f"[PROXY] Cache miss for {key}")
        result = self._real.get_data(key)
        self._cache[key] = result
        return result

    def set_data(self, key: str, value: str) -> None:
        self._lazy_init()
        self._check_access("write")
        self._real.set_data(key, value)
        self._cache[key] = value
        print(f"[PROXY] Set {key} = {value}")

    def delete_data(self, key: str) -> None:
        self._lazy_init()
        self._check_access("delete")
        self._real.delete_data(key)
        self._cache.pop(key, None)
        print(f"[PROXY] Deleted {key}")


class DataStore:
    def __init__(self):
        self._store = {}

    def get_data(self, key: str) -> str:
        return self._store.get(key, "not found")

    def set_data(self, key: str, value: str) -> None:
        self._store[key] = value

    def delete_data(self, key: str) -> None:
        self._store.pop(key, None)


store = SmartProxy(DataStore(), user_role="editor")
store.set_data("name", "Alice")
print(store.get_data("name"))
print(store.get_data("name"))
try:
    store.delete_data("name")
except PermissionError as e:
    print(f"Blocked: {e}")
```

## Real-World Use Cases

```python
# ORM-style lazy loading proxy
class UserModel:
    def __init__(self, user_id: int):
        self._id = user_id
        self._data = None

    def _load(self):
        print(f"[DB] Loading user {self._id}...")
        self._data = {
            "id": self._id,
            "name": "Alice",
            "email": "alice@example.com",
            "orders": OrderCollectionProxy(self._id),
        }

    def __getattr__(self, name):
        if self._data is None:
            self._load()
        return self._data.get(name)


class OrderCollectionProxy:
    def __init__(self, user_id: int):
        self._user_id = user_id
        self._orders = None

    def __iter__(self):
        if self._orders is None:
            print(f"[DB] Loading orders for user {self._user_id}...")
            self._orders = [
                {"id": 1, "total": 49.99},
                {"id": 2, "total": 99.99},
            ]
        return iter(self._orders)

    def __len__(self):
        if self._orders is None:
            return 0
        return len(self._orders)

    def total_spent(self) -> float:
        return sum(o["total"] for o in self)


user = UserModel(1)
print("User created, not loaded yet")
print(f"Name: {user.name}")
print("Orders:")
for order in user.orders:
    print(f"  Order {order['id']}: ${order['total']}")
```

```python
# Virtual proxy for heavy ML models
import time


class MLModel:
    def predict(self, features: list) -> float:
        raise NotImplementedError


class HeavyModel(MLModel):
    def __init__(self):
        print("Loading heavy model (5 seconds)...")
        time.sleep(5)
        self._loaded = True

    def predict(self, features: list) -> float:
        return sum(features) * 0.5 + 0.1


class LazyModelProxy(MLModel):
    def __init__(self):
        self._model = None

    def predict(self, features: list) -> float:
        if self._model is None:
            self._model = HeavyModel()
        return self._model.predict(features)


proxy = LazyModelProxy()
print("Proxy created, model not loaded")
result = proxy.predict([1.0, 2.0, 3.0])
print(f"Prediction: {result}")
result = proxy.predict([4.0, 5.0, 6.0])
print(f"Prediction: {result}")
```

## Common Mistakes

```python
# MISTAKE 1: Proxy not implementing the same interface
class BadProxy:
    def fetch(self, url):  # Should match Subject interface
        pass
```

```python
# MISTAKE 2: Proxy vs Decorator confusion
# Decorator: adds NEW behavior
# Proxy: CONTROLS access to existing behavior
```

```python
# MISTAKE 3: Breaking transparency — clients aware of proxy
# Clients should treat proxy and real subject identically
```

```python
# MISTAKE 4: Not handling real subject failures
class FragileProxy:
    def request(self):
        return self._real.request()  # No error handling!
```

## Best Practices

```python
# 1. Always implement the same interface as the real subject
# 2. Keep proxy logic focused on one concern
# 3. Make proxy transparent to clients
# 4. Consider using __getattr__ for generic forwarding
# 5. Document which proxy type you're implementing
```

```python
# Generic forwarding proxy using __getattr__
class GenericProxy:
    def __init__(self, real):
        self._real = real

    def __getattr__(self, name):
        return getattr(self._real, name)
```

## Interview Questions

```python
# Q1: Proxy vs Decorator pattern?
# Proxy: controls access, may create/lazy-load subject
# Decorator: adds new behavior, wraps existing object
```

```python
# Q2: Types of proxy?
# Virtual (lazy loading), Protection (access control),
# Remote (network), Caching, Smart (logging/reference counting)
```

```python
# Q3: When would you use a virtual proxy?
# Expensive object creation, large data loading,
# network resources, ORM lazy loading
```

## Coding Challenges

```python
# Challenge 1: Implement a proxy that logs all method
# calls with arguments and return values
```

```python
# Challenge 2: Build a retry proxy that retries failed
# operations with exponential backoff
```

```python
# Challenge 3: Create a circuit breaker proxy that
# stops calling a failing service after N failures
```

```python
# Challenge 4: Implement a proxy that provides
# transparent serialization/deserialization
```

```python
# Challenge 5: Build a proxy that monitors performance
# metrics (call count, latency, error rate)
```

## Summary

The Proxy pattern provides a surrogate that controls access to another object. Virtual proxies enable lazy loading, protection proxies enforce access control, remote proxies handle network communication, and caching proxies optimize performance. The key distinction from the Decorator pattern is that proxies control access while decorators add behavior. Proxies maintain the same interface as the real subject, ensuring transparency for clients.

## Related Topics

- Decorator Pattern: Similar structure, different intent (add vs control)
- Adapter Pattern: Changes interface; proxy preserves it
- Lazy Loading: Virtual proxy implements this
- Caching: Caching proxy implements this
- Access Control: Protection proxy implements this
- RPC/Remote Objects: Remote proxy implements this
