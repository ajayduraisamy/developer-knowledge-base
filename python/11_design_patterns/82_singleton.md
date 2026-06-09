# Singleton - __new__ override, metaclass singleton, Borg pattern
## Introduction
The Singleton pattern ensures a class has only one instance and provides a global point of access to it. In Python, Singleton can be implemented in multiple ways: overriding `__new__`, using metaclasses, or using module-level singletons (which are inherently singleton in Python due to module caching). The Borg pattern (also called Monostate) offers an alternative where instances share state rather than enforcing identity.

## __new__ override singleton
### What It Is
The `__new__` method in Python is responsible for creating a new instance of a class. By overriding `__new__`, we can control instance creation and ensure only one instance exists.

### Why It Is Important
This is the simplest and most common Singleton implementation in Python. It leverages the language's object creation mechanism directly without additional constructs.

### How It Works Internally
`__new__` is called before `__init__`. By checking if an instance exists in a class variable and creating one only if it doesn't, we ensure a single instance. The lock prevents race conditions in multithreaded environments.

```python
class Singleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        self.value = 0

# Usage
s1 = Singleton()
s2 = Singleton()
assert s1 is s2  # Same instance
s1.value = 42
assert s2.value == 42  # Shared state
```

### Thread-Safe Singleton
```python
import threading

class ThreadSafeSingleton:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance
```

### Lazy Initialization Singleton
```python
class DatabaseConnection:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            print("Creating database connection...")
            cls._instance = super().__new__(cls)
            cls._instance._initialized = False
        return cls._instance

    def connect(self, host, port):
        if not self._initialized:
            self.host = host
            self.port = port
            self._initialized = True
            print(f"Connected to {host}:{port}")
        return self

    def query(self, sql):
        if self._initialized:
            return f"Result of: {sql}"
        raise RuntimeError("Not connected")

# Usage
db1 = DatabaseConnection().connect("localhost", 5432)
db2 = DatabaseConnection().connect("localhost", 5432)
assert db1 is db2
assert db1.query("SELECT 1") == db2.query("SELECT 1")
print(f"Same connection: {db1 is db2}")  # True
```

## Metaclass singleton
### What It Is
A metaclass Singleton uses the metaclass mechanism to control class instantiation. The metaclass overrides `__call__` to control what happens when the class is called (i.e., instantiated).

### Why It Is Important
Metaclass-based singletons are cleaner because they encapsulate the singleton logic in the metaclass, leaving the actual class free of singleton code. This separation of concerns is more Pythonic and reusable.

### How It Works Internally
When a class with a metaclass is instantiated with `ClassName()`, Python calls the metaclass's `__call__` method. This method creates the instance (via `__new__`) and initializes it (via `__init__`). By caching the instance, we return the same object each time.

```python
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]

class Logger(metaclass=SingletonMeta):
    def __init__(self):
        self.logs = []

    def log(self, message):
        self.logs.append(message)
        print(f"Log: {message}")

# Usage
logger1 = Logger()
logger2 = Logger()
assert logger1 is logger2
logger1.log("First message")
assert len(logger2.logs) == 1  # Shared state
```

### Multiple Singleton Classes
```python
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]

class Config(metaclass=SingletonMeta):
    def __init__(self):
        self.settings = {}

class Cache(metaclass=SingletonMeta):
    def __init__(self):
        self.data = {}

# Each class has its own singleton instance
config1 = Config()
config2 = Config()
cache1 = Cache()
cache2 = Cache()

assert config1 is config2
assert cache1 is cache2
assert config1 is not cache1  # Different classes, different instances
```

## Borg pattern
### What It Is
The Borg pattern (also called Monostate) allows multiple instances of a class to share the same state. Unlike traditional Singleton where there's one instance, Borg instances are separate objects that share `__dict__`.

### Why It Is Important
Borg provides more flexibility than Singleton. Subclassing works more naturally, and the pattern is more transparent to clients. It also allows different instances to temporarily diverge if needed.

### How It Works Internally
Borg works by making all instances share the same `__dict__` (instance attribute dictionary). When any instance modifies an attribute, all instances see the change. This is achieved by overriding `__new__` to set the instance's `__dict__` to a shared class-level dictionary.

```python
class Borg:
    _shared_state = {}

    def __new__(cls, *args, **kwargs):
        instance = super().__new__(cls, *args, **kwargs)
        instance.__dict__ = cls._shared_state
        return instance

class ConfigManager(Borg):
    def __init__(self):
        if not hasattr(self, 'initialized'):
            self.config = {}
            self.initialized = True

    def set(self, key, value):
        self.config[key] = value

    def get(self, key, default=None):
        return self.config.get(key, default)

# Usage
c1 = ConfigManager()
c2 = ConfigManager()
c1.set("database_url", "postgres://localhost/db")
assert c2.get("database_url") == "postgres://localhost/db"
assert c1 is not c2  # Different instances
assert c1.__dict__ is c2.__dict__  # But shared state
```

### Borg with Subclassing
```python
class BaseBorg:
    _shared_state = {}

    def __new__(cls, *args, **kwargs):
        instance = super().__new__(cls, *args, **kwargs)
        instance.__dict__ = cls._shared_state
        return instance

class DatabaseConfig(BaseBorg):
    def __init__(self):
        if not hasattr(self, 'host'):
            self.host = "localhost"
            self.port = 5432

class APIConfig(BaseBorg):
    def __init__(self):
        if not hasattr(self, 'api_key'):
            self.api_key = "default_key"

# Note: All Borg subclasses share the SAME state
# This can be a feature or a bug depending on intent
db_config = DatabaseConfig()
api_config = APIConfig()

print(db_config.host)      # "localhost"
print(api_config.host)     # "localhost" - shared!
print(db_config is api_config)  # False (different instances)
print(db_config.__dict__ is api_config.__dict__)  # True (shared state)
```

### Per-Child-Class Borg
```python
class PerClassBorg:
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        cls._shared_state = {}

    def __new__(cls, *args, **kwargs):
        instance = super().__new__(cls, *args, **kwargs)
        instance.__dict__ = cls._shared_state
        return instance

class DBConfig(PerClassBorg):
    def __init__(self):
        if not hasattr(self, 'db_host'):
            self.db_host = "localhost"

class CacheConfig(PerClassBorg):
    def __init__(self):
        if not hasattr(self, 'cache_host'):
            self.cache_host = "localhost"

db1 = DBConfig()
cache1 = CacheConfig()
db2 = DBConfig()

assert db1 is not db2  # Different instances
assert db1.__dict__ is db2.__dict__  # Same class, shared state
assert db1.__dict__ is not cache1.__dict__  # Different classes, different state
```

### Module-Level Singleton
```python
# singletons.py
class Database:
    def __init__(self):
        self.connected = False

    def connect(self):
        print("Connecting...")
        self.connected = True

# Module-level instance (imported once, cached by Python)
db = Database()

# In other files:
# from singletons import db
# db.connect()  # Always the same instance
```

### Real-World Use Cases
- Database connection pools
- Configuration managers
- Logging services
- Cache managers
- Thread pools
- Window managers in GUI applications
- Print spoolers

### Common Mistakes
- Not handling thread safety in `__new__` Singleton
- Forgetting that Borg shares state across all instances, including subclasses
- Overusing Singleton (often unnecessary complexity)
- Making Singletons that are hard to test (global state)
- Not considering module-level singletons as simpler alternative

### Best Practices
- Use module-level singletons for simple cases (most Pythonic)
- Prefer Borg pattern when you need inheritance
- Use metaclass Singleton for framework-level patterns
- Consider dependency injection as an alternative
- Make Singletons testable by allowing instance reset

### Performance Considerations
- Singleton overhead is negligible (one extra pointer dereference)
- Thread-safe Singleton with double-checked locking has minimal contention
- Borg pattern has slightly more overhead due to `__dict__` indirection
- Singleton can become a bottleneck if heavily contended

### Interview Questions
1. What are the different ways to implement Singleton in Python?
2. How does Borg pattern differ from Singleton?
3. When would you choose Borg over Singleton?
4. How do you make a Singleton thread-safe?
5. What are the criticisms of Singleton pattern?

### Coding Challenges
1. Implement a thread-safe configuration manager as Singleton
2. Create a database connection pool using Borg pattern
3. Implement a cache that uses Singleton with TTL expiration
4. Write a Logger class that works as a metaclass Singleton

### Related Topics
- Factory pattern
- Dependency injection
- Global state management
- Metaclasses in Python
- Module caching system
- Thread safety patterns
