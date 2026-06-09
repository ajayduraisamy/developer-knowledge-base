# Proxy - Virtual, protection, remote, caching proxy patterns
## Introduction
The Proxy pattern provides a surrogate or placeholder for another object to control access to it. Proxies are used for lazy initialization (virtual proxy), access control (protection proxy), local representation of remote objects (remote proxy), and caching (caching proxy). The proxy implements the same interface as the real subject, making it transparent to the client.

## Virtual proxy
### What It Is
A Virtual Proxy delays the creation and initialization of an expensive object until it's actually needed. It acts as a stand-in that creates the real object on first access, providing lazy initialization.

### Why It Is Important
Virtual proxies optimize resource usage by deferring expensive operations (database connections, file loading, network requests) until the result is actually required. This improves startup time and reduces memory footprint.

### How It Works Internally
The virtual proxy holds a reference to the real subject (initially None). When a method is called, the proxy checks if the real subject exists; if not, it creates it. Subsequent calls use the cached instance.

```python
from abc import ABC, abstractmethod

class Image(ABC):
    @abstractmethod
    def display(self) -> str:
        pass

class HighResolutionImage(Image):
    def __init__(self, filename: str):
        self._filename = filename
        self._load_from_disk()

    def _load_from_disk(self):
        print(f"Loading high-resolution image from {self._filename}...")
        # Simulate expensive loading
        import time
        time.sleep(0.5)
        print("Image loaded.")

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

# Usage
print("Creating proxy (no loading yet)")
proxy = ImageProxy("photo.jpg")

print("Calling display (loading happens now)")
print(proxy.display())

print("Calling display again (no loading)")
print(proxy.display())
```

### Lazy Database Connection
```python
class DatabaseConnection:
    def __init__(self, host, port, database, user, password):
        self.host = host
        self.port = port
        self.database = database
        self.user = user
        self.password = password
        self._connection = None

    def connect(self):
        if self._connection is None:
            print(f"Connecting to {self.host}:{self.port}/{self.database}")
            self._connection = f"Connection({self.database})"
        return self._connection

    def query(self, sql):
        conn = self.connect()
        return f"Executed '{sql}' on {conn}"

    def close(self):
        if self._connection:
            print("Closing connection")
            self._connection = None

class DatabaseProxy:
    def __init__(self, host, port, database, user, password):
        self._config = dict(host=host, port=port, database=database,
                           user=user, password=password)
        self._real_db = None

    def query(self, sql):
        if self._real_db is None:
            self._real_db = DatabaseConnection(**self._config)
        return self._real_db.query(sql)

    def close(self):
        if self._real_db:
            self._real_db.close()

# Usage
db = DatabaseProxy("localhost", 5432, "mydb", "user", "pass")
print("Database proxy created (no connection yet)")
result = db.query("SELECT 1")  # Connection established here
print(result)
```

## Protection proxy
### What It Is
A Protection Proxy controls access to an object based on access rights. It checks permissions before delegating method calls to the real subject, preventing unauthorized access.

### Why It Is Important
Protection proxies implement access control without modifying the real subject. They enforce security policies, rate limiting, or role-based access transparently.

### How It Works Internally
The protection proxy wraps the real subject and checks credentials/permissions before each method call. If the check fails, it raises an exception or returns a fallback value.

```python
from abc import ABC, abstractmethod
from enum import Enum

class UserRole(Enum):
    GUEST = "guest"
    USER = "user"
    ADMIN = "admin"
    SUPER_ADMIN = "super_admin"

class Document(ABC):
    @abstractmethod
    def read(self) -> str:
        pass

    @abstractmethod
    def write(self, content: str) -> None:
        pass

    @abstractmethod
    def delete(self) -> None:
        pass

class RealDocument(Document):
    def __init__(self, title: str, content: str = ""):
        self._title = title
        self._content = content

    def read(self) -> str:
        return f"Content of '{self._title}': {self._content}"

    def write(self, content: str) -> None:
        self._content = content
        print(f"Document '{self._title}' updated")

    def delete(self) -> None:
        print(f"Document '{self._title}' deleted")

class DocumentProxy(Document):
    _PERMISSIONS = {
        "read": {UserRole.GUEST, UserRole.USER, UserRole.ADMIN, UserRole.SUPER_ADMIN},
        "write": {UserRole.USER, UserRole.ADMIN, UserRole.SUPER_ADMIN},
        "delete": {UserRole.ADMIN, UserRole.SUPER_ADMIN},
    }

    def __init__(self, document: Document, user_role: UserRole):
        self._document = document
        self._user_role = user_role

    def _check_permission(self, operation: str) -> None:
        if self._user_role not in self._PERMISSIONS[operation]:
            raise PermissionError(
                f"{self._user_role.value} does not have {operation} permission"
            )

    def read(self) -> str:
        self._check_permission("read")
        return self._document.read()

    def write(self, content: str) -> None:
        self._check_permission("write")
        self._document.write(content)

    def delete(self) -> None:
        self._check_permission("delete")
        self._document.delete()

# Usage
doc = RealDocument("Secret Plans", "Launch on Monday")

guest_proxy = DocumentProxy(doc, UserRole.GUEST)
print(guest_proxy.read())  # OK
try:
    guest_proxy.write("Change plans")  # Error
except PermissionError as e:
    print(f"Access denied: {e}")

admin_proxy = DocumentProxy(doc, UserRole.ADMIN)
admin_proxy.write("Revised plans")  # OK
admin_proxy.delete()  # OK
```

### Rate Limiting Proxy
```python
import time
from functools import wraps

class RateLimitingProxy:
    def __init__(self, target, max_calls=10, period=60):
        self._target = target
        self._max_calls = max_calls
        self._period = period
        self._call_times = []

    def _check_rate_limit(self):
        now = time.time()
        self._call_times = [t for t in self._call_times if now - t < self._period]
        if len(self._call_times) >= self._max_calls:
            raise RuntimeError(f"Rate limit exceeded: {self._max_calls} calls per {self._period}s")
        self._call_times.append(now)

    def __getattr__(self, name):
        attr = getattr(self._target, name)
        if callable(attr):
            @wraps(attr)
            def rate_limited(*args, **kwargs):
                self._check_rate_limit()
                return attr(*args, **kwargs)
            return rate_limited
        return attr

class APIService:
    def fetch_data(self, endpoint):
        return f"Data from {endpoint}"

    def post_data(self, endpoint, data):
        return f"Posted to {endpoint}: {data}"

api = RateLimitingProxy(APIService(), max_calls=3, period=10)
print(api.fetch_data("/users"))   # OK
print(api.fetch_data("/posts"))   # OK
print(api.post_data("/users", {}))  # OK
try:
    print(api.fetch_data("/error"))  # Rate limited
except RuntimeError as e:
    print(f"Rate limited: {e}")
```

## Remote proxy
### What It Is
A Remote Proxy represents an object that exists in a different address space (different process, server, or network). It handles the communication details, making remote objects feel local to the client.

### Why It Is Important
Remote proxies hide network complexity (serialization, transport, error handling) from the client. They enable distributed systems where objects communicate across network boundaries transparently.

### How It Works Internally
The remote proxy serializes method calls and arguments, sends them over the network to the remote service, waits for the response, deserializes it, and returns the result to the client.

```python
import json
import urllib.request
from abc import ABC, abstractmethod

class WeatherService(ABC):
    @abstractmethod
    def get_temperature(self, city: str) -> float:
        pass

    @abstractmethod
    def get_forecast(self, city: str, days: int) -> list:
        pass

class RemoteWeatherProxy(WeatherService):
    def __init__(self, base_url: str, api_key: str = None):
        self._base_url = base_url.rstrip('/')
        self._api_key = api_key

    def _request(self, endpoint: str, params: dict = None) -> dict:
        url = f"{self._base_url}{endpoint}"
        if params:
            query = urllib.parse.urlencode(params)
            url = f"{url}?{query}"
        if self._api_key:
            url = f"{url}&apikey={self._api_key}"

        print(f"Remote call: GET {url}")
        # Simulated network call
        response = self._simulate_http_get(url)
        return json.loads(response)

    def _simulate_http_get(self, url: str) -> str:
        return '{"temperature": 22.5, "forecast": ["Sunny", "Cloudy"]}'

    def get_temperature(self, city: str) -> float:
        data = self._request("/weather", {"city": city})
        return data["temperature"]

    def get_forecast(self, city: str, days: int) -> list:
        data = self._request("/forecast", {"city": city, "days": days})
        return data["forecast"]

# XML-RPC Remote Proxy
from xmlrpc.client import ServerProxy

class XMLRPCProxy:
    def __init__(self, uri: str):
        self._proxy = ServerProxy(uri)

    def __getattr__(self, name):
        return getattr(self._proxy, name)

# Usage
weather = RemoteWeatherProxy("https://api.weather.com", "test_key")
print(f"Temperature: {weather.get_temperature('London')}°C")
```

## Caching proxy
### What It Is
A Caching Proxy stores results of expensive operations and returns cached results for repeated requests with the same parameters. It improves performance by avoiding redundant computations or network calls.

### Why It Is Important
Caching proxies dramatically improve response times and reduce load on backend systems. They are essential for applications with repeated identical requests and slow or expensive data sources.

### How It Works Internally
The caching proxy maintains a cache dictionary keyed by method arguments. Before delegating to the real subject, it checks if a cached result exists. If so, it returns the cached value; otherwise, it calls the real subject, stores the result, and returns it.

```python
from abc import ABC, abstractmethod
import time
import hashlib
import json

class DataProvider(ABC):
    @abstractmethod
    def get_data(self, query: str) -> dict:
        pass

class ExpensiveDataProvider(DataProvider):
    def get_data(self, query: str) -> dict:
        print(f"Executing expensive query: {query}")
        time.sleep(1)  # Simulate slow operation
        return {"query": query, "result": f"Result for '{query}'", "timestamp": time.time()}

class CachingProxy(DataProvider):
    def __init__(self, provider: DataProvider, ttl: int = 300):
        self._provider = provider
        self._cache = {}
        self._ttl = ttl

    def _make_key(self, query: str) -> str:
        return hashlib.md5(query.encode()).hexdigest()

    def _is_expired(self, cached_time: float) -> bool:
        return time.time() - cached_time > self._ttl

    def get_data(self, query: str) -> dict:
        key = self._make_key(query)

        if key in self._cache:
            cached = self._cache[key]
            if not self._is_expired(cached["cached_at"]):
                print(f"Cache hit for: {query}")
                return cached["data"]

        print(f"Cache miss for: {query}")
        data = self._provider.get_data(query)
        self._cache[key] = {
            "data": data,
            "cached_at": time.time()
        }
        return data

    def clear_cache(self):
        self._cache.clear()
        print("Cache cleared")

    def invalidate(self, query: str):
        key = self._make_key(query)
        self._cache.pop(key, None)

# Usage
provider = ExpensiveDataProvider()
cached_provider = CachingProxy(provider, ttl=10)

print(cached_provider.get_data("SELECT * FROM users"))  # Cache miss
print(cached_provider.get_data("SELECT * FROM users"))  # Cache hit
print(cached_provider.get_data("SELECT * FROM orders"))  # Different query, cache miss
```

### LRU Caching Proxy
```python
from collections import OrderedDict

class LRUCachingProxy:
    def __init__(self, provider, max_size=100):
        self._provider = provider
        self._cache = OrderedDict()
        self._max_size = max_size

    def get_data(self, query):
        if query in self._cache:
            self._cache.move_to_end(query)
            print(f"LRU cache hit: {query}")
            return self._cache[query]

        result = self._provider.get_data(query)
        self._cache[query] = result

        if len(self._cache) > self._max_size:
            oldest = next(iter(self._cache))
            self._cache.pop(oldest)
            print(f"Evicted: {oldest}")

        return result
```

### Real-World Use Cases
- ORM lazy loading (SQLAlchemy, Django ORM)
- Virtual proxies for large media files
- Authentication/authorization proxies
- API rate limiting proxies
- Redis/Memcached caching proxies
- gRPC/stub proxies for microservices
- Image thumbnailing proxies

### Common Mistakes
- Not implementing the full interface (missing methods)
- Thread safety issues in caching proxies
- Memory leaks from unbounded caches
- Stale cache data (not handling TTL properly)
- Over-proxying (unnecessary abstraction)

### Best Practices
- Implement the exact same interface as the real subject
- Keep proxies lightweight (don't add business logic)
- Use weak references for cache to prevent memory leaks
- Implement cache invalidation strategies
- Consider thread safety for shared proxies
- Log proxy operations for debugging

### Performance Considerations
- Proxy overhead is minimal (single method dispatch)
- Caching proxy trades memory for speed
- Virtual proxy can significantly improve startup time
- Protection proxy adds permission check overhead
- Cache size management is crucial for memory usage

### Interview Questions
1. What are the four main types of proxy patterns?
2. How does a virtual proxy differ from lazy initialization?
3. How would you implement a thread-safe caching proxy?
4. When would you use a proxy vs a decorator?
5. Explain the difference between Proxy and Adapter patterns.

### Coding Challenges
1. Implement a virtual proxy for a large configuration file loader
2. Create a protection proxy that implements role-based access control
3. Build an LRU caching proxy for a slow API service
4. Implement a logging proxy that records all method calls and timing

### Related Topics
- Decorator pattern
- Adapter pattern
- Facade pattern
- Lazy loading
- Caching strategies (LRU, TTL, LFU)
- Distributed systems
- AOP (Aspect-Oriented Programming)
