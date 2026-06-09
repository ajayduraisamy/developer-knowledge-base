# Singleton - __new__ override, metaclass singleton, Borg pattern

## Introduction

The Singleton pattern ensures a class has only one instance and provides a global point of access to it. It is one of the most well-known creational design patterns from the Gang of Four. In Python, singletons can be implemented in multiple ways due to the language's dynamic nature and flexible object model.

## Why It Is Important

The Singleton pattern is critical when exactly one object must coordinate actions across a system. Common use cases include configuration managers, logging services, thread pools, database connection pools, and caching layers. Without a singleton, multiple instances could lead to inconsistent state, wasted resources, or conflicting operations. However, singletons are often overused and can introduce hidden dependencies and testability problems.

## Syntax

There is no dedicated syntax for singletons in Python. Instead, developers use language features like the `__new__` method, class decorators, metaclasses, or module-level imports to enforce single-instance behavior.

## Examples

```python
# Singleton using __new__ override
class SingletonNew:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self, value=None):
        if not hasattr(self, '_initialized'):
            self.value = value
            self._initialized = True


s1 = SingletonNew("first")
s2 = SingletonNew("second")
print(s1 is s2)       # True
print(s1.value)       # first — __init__ only runs once
print(s2.value)       # first
```

```python
# Borg / Monostate pattern — share state, not identity
class Borg:
    _shared_state = {}

    def __new__(cls, *args, **kwargs):
        obj = super().__new__(cls, *args, **kwargs)
        obj.__dict__ = cls._shared_state
        return obj


class ConfigManager(Borg):
    def __init__(self):
        if not hasattr(self, 'initialized'):
            self.settings = {}
            self.initialized = True

    def set(self, key, value):
        self.settings[key] = value

    def get(self, key, default=None):
        return self.settings.get(key, default)


c1 = ConfigManager()
c2 = ConfigManager()
c1.set("theme", "dark")
print(c2.get("theme"))  # dark — shared state
print(c1 is c2)         # False — different objects
```

```python
# Module-level singleton (most Pythonic)
# singleton_module.py
class _DatabaseConnectionPool:
    def __init__(self):
        self.connections = []
        self.max_size = 10

    def acquire(self):
        if self.connections:
            return self.connections.pop()
        return f"Connection-{id(self)}-new"

    def release(self, conn):
        if len(self.connections) < self.max_size:
            self.connections.append(conn)


# At module level — one instance per import
db_pool = _DatabaseConnectionPool()

# In any other file:
# from singleton_module import db_pool
```

```python
# Metaclass singleton
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]


class Logger(metaclass=SingletonMeta):
    def __init__(self):
        self.log_file = "app.log"

    def log(self, level, message):
        print(f"[{level}] {message}")


class AppConfig(metaclass=SingletonMeta):
    def __init__(self):
        self.config = {}


log1 = Logger()
log2 = Logger()
print(log1 is log2)  # True
```

```python
# Thread-safe singleton with locking
import threading
import time


class ThreadSafeSingleton:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls, *args, **kwargs)
        return cls._instance

    def __init__(self):
        if not hasattr(self, '_initialized'):
            self.counter = 0
            self._initialized = True


results = []


def worker():
    obj = ThreadSafeSingleton()
    obj.counter += 1
    results.append(id(obj))


threads = [threading.Thread(target=worker) for _ in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(all(r == results[0] for r in results))  # True — all same instance
```

```python
# Singleton decorator
def singleton(cls):
    instances = {}

    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]

    return get_instance


@singleton
class Database:
    def __init__(self):
        self.connected = False

    def connect(self):
        self.connected = True
        return "Connected"


db1 = Database()
db2 = Database()
print(db1 is db2)  # True
```

```python
# Lazy singleton with descriptor
class LazySingleton:
    _instance = None

    def __getattr__(self, name):
        return getattr(self._instance, name)

    @classmethod
    def instance(cls):
        if cls._instance is None:
            cls._instance = cls.__new__(cls)
            cls._instance.__init__()
        return cls._instance
```

## Beginner Examples

```python
# Simplest singleton — module-level object

# config.py
class Configuration:
    def __init__(self):
        self.data = {
            "app_name": "MyApp",
            "version": "1.0.0",
            "debug": True,
            "database_url": "sqlite:///app.db",
        }

    def get(self, key):
        return self.data.get(key)

    def set(self, key, value):
        self.data[key] = value


config = Configuration()


# main.py
# from config import config

def start_app():
    print(f"Starting {config.get('app_name')} v{config.get('version')}")
    if config.get("debug"):
        print("Debug mode enabled")


start_app()
# Starting MyApp v1.0.0
# Debug mode enabled
```

```python
# Simple __new__ singleton
class Printer:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.jobs = []
        return cls._instance

    def add_job(self, document):
        self.jobs.append(document)
        print(f"Added: {document}")

    def print_all(self):
        for job in self.jobs:
            print(f"Printing: {job}")
        self.jobs.clear()


p1 = Printer()
p2 = Printer()
p1.add_job("Report.pdf")
p2.add_job("Invoice.pdf")
print(len(p1.jobs))  # 2
```

## Intermediate Examples

```python
# Thread-safe database connection pool as singleton
import threading
import time
import random
from typing import Optional


class ConnectionPool:
    _instance = None
    _lock = threading.Lock()
    _pool_lock = threading.Lock()

    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance._initialize()
        return cls._instance

    def _initialize(self):
        self._pool = []
        self._max_size = 5
        self._in_use = {}
        self._total_created = 0
        self._create_connections(2)

    def _create_connections(self, count):
        for _ in range(count):
            conn_id = f"conn-{self._total_created}"
            self._pool.append(conn_id)
            self._total_created += 1

    def acquire(self, timeout: float = 5.0) -> Optional[str]:
        start = time.time()
        while time.time() - start < timeout:
            with self._pool_lock:
                if self._pool:
                    conn = self._pool.pop()
                    self._in_use[conn] = threading.current_thread().name
                    return conn
                if self._total_created < self._max_size:
                    conn = f"conn-{self._total_created}"
                    self._total_created += 1
                    self._in_use[conn] = threading.current_thread().name
                    return conn
            time.sleep(0.01)
        return None

    def release(self, conn: str):
        with self._pool_lock:
            if conn in self._in_use:
                del self._in_use[conn]
                self._pool.append(conn)

    @property
    def available(self) -> int:
        return len(self._pool)

    @property
    def active(self) -> int:
        return len(self._in_use)


def worker_task(pool, tasks):
    for task in tasks:
        conn = pool.acquire()
        if conn:
            print(f"{threading.current_thread().name} got {conn} for task {task}")
            time.sleep(random.uniform(0.1, 0.3))
            pool.release(conn)
        else:
            print(f"{threading.current_thread().name} TIMEOUT on task {task}")


pool = ConnectionPool()
threads = [
    threading.Thread(target=worker_task, args=(pool, range(3)), name=f"Worker-{i}")
    for i in range(4)
]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(f"Pool available: {pool.available}, active: {pool.active}")
```

```python
# Singleton with dependency injection support
class ServiceRegistry:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._services = {}
            cls._instance._singletons = {}
        return cls._instance

    def register(self, name, factory, singleton=False):
        self._services[name] = {"factory": factory, "singleton": singleton}

    def resolve(self, name, *args, **kwargs):
        service = self._services.get(name)
        if not service:
            raise KeyError(f"Service '{name}' not registered")
        if service["singleton"]:
            if name not in self._singletons:
                self._singletons[name] = service["factory"](*args, **kwargs)
            return self._singletons[name]
        return service["factory"](*args, **kwargs)


registry = ServiceRegistry()

class EmailService:
    def send(self, to, subject):
        print(f"Sending '{subject}' to {to}")

class UserRepository:
    def get_user(self, user_id):
        return {"id": user_id, "name": "Alice"}

registry.register("email", lambda: EmailService(), singleton=True)
registry.register("user_repo", lambda: UserRepository(), singleton=True)

email = registry.resolve("email")
email.send("alice@example.com", "Welcome!")
```

## Advanced Examples

```python
# Metaclass-based singleton registry for multiple subclasses
class SingletonRegistry(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]


class DatabaseEngine(metaclass=SingletonRegistry):
    def __init__(self):
        self.connection = None

    def connect(self, url):
        raise NotImplementedError


class PostgreSQL(DatabaseEngine):
    def connect(self, url):
        self.connection = f"PostgreSQL connected to {url}"
        return self.connection


class MySQL(DatabaseEngine):
    def connect(self, url):
        self.connection = f"MySQL connected to {url}"
        return self.connection


class SQLite(DatabaseEngine):
    def connect(self, url):
        self.connection = f"SQLite connected to {url}"
        return self.connection


pg1 = PostgreSQL()
pg2 = PostgreSQL()
mysql1 = MySQL()

print(pg1 is pg2)                     # True
print(pg1.connect("pg://localhost"))  # PostgreSQL connected to pg://localhost
print(pg2.connect("pg://remote"))     # PostgreSQL connected to pg://localhost
print(mysql1.connect("mysql://localhost"))  # MySQL connected to mysql://localhost
```

```python
# Thread-safe singleton with async initialization
import asyncio
import threading
from typing import Optional, Any


class AsyncSingleton:
    _instance = None
    _lock = asyncio.Lock()
    _init_lock = threading.Lock()
    _initialized = False

    def __new__(cls):
        if cls._instance is None:
            with cls._init_lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance._async_data = None
        return cls._instance

    async def initialize(self):
        if not self._initialized:
            async with self._lock:
                if not self._initialized:
                    print("Initializing singleton asynchronously...")
                    await asyncio.sleep(0.5)
                    self._async_data = {
                        "models": ["gpt-4", "claude-3"],
                        "api_key": "sk-xxx",
                        "rate_limit": 100,
                    }
                    self._initialized = True

    async def get_data(self, key: str) -> Optional[Any]:
        await self.initialize()
        return self._async_data.get(key) if self._async_data else None


async def worker(name):
    singleton = AsyncSingleton()
    data = await singleton.get_data("models")
    print(f"{name}: {data}")
    print(f"{name}: same instance? {singleton is AsyncSingleton()}")


async def main():
    await asyncio.gather(worker("A"), worker("B"), worker("C"))


asyncio.run(main())
```

```python
# Singleton with cleanup / lifecycle management
import weakref


class LifecycleSingleton:
    _instance = None
    _cleanup_handlers = []

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._resources = []
            cls._instance._active = True
        return cls._instance

    def register_resource(self, resource):
        self._resources.append(resource)

    def add_cleanup_handler(self, handler):
        self._cleanup_handlers.append(handler)

    def shutdown(self):
        if self._active:
            print("Shutting down singleton...")
            for handler in self._cleanup_handlers:
                handler()
            for resource in self._resources:
                print(f"  Closing {resource}")
            self._resources.clear()
            self._active = False
            type(self)._instance = None

    @classmethod
    def reset(cls):
        if cls._instance is not None:
            cls._instance.shutdown()
        cls._instance = None


singleton = LifecycleSingleton()
singleton.register_resource("db-connection-1")
singleton.register_resource("cache-redis")
singleton.add_cleanup_handler(lambda: print("  Flushing logs..."))
singleton.shutdown()
print(singleton is LifecycleSingleton())  # False — new instance
```

```python
# Abstract singleton base class
class Singleton(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]


class Config(metaclass=Singleton):
    def __init__(self):
        self._data = {
            "host": "localhost",
            "port": 8080,
            "debug": False,
        }

    def __getitem__(self, key):
        return self._data[key]

    def __setitem__(self, key, value):
        self._data[key] = value

    def __contains__(self, key):
        return key in self._data


cfg = Config()
cfg["host"] = "0.0.0.0"
print(cfg["host"])  # 0.0.0.0
print("port" in cfg)  # True
```

## Real-World Use Cases

```python
# Application-wide logging service
import logging
import sys
from datetime import datetime


class AppLogger(metaclass=SingletonMeta):
    def __init__(self):
        self.logger = logging.getLogger("App")
        self.logger.setLevel(logging.DEBUG)
        handler = logging.StreamHandler(sys.stdout)
        handler.setFormatter(
            logging.Formatter("%(asctime)s [%(levelname)s] %(message)s")
        )
        self.logger.addHandler(handler)
        self._activity_log = []

    def info(self, message):
        self.logger.info(message)
        self._activity_log.append((datetime.now(), "INFO", message))

    def error(self, message):
        self.logger.error(message)
        self._activity_log.append((datetime.now(), "ERROR", message))

    def warn(self, message):
        self.logger.warning(message)
        self._activity_log.append((datetime.now(), "WARN", message))

    def get_activity(self):
        return list(self._activity_log)
```

```python
# Global configuration manager loading from YAML/env
import os
import json
from pathlib import Path


class Settings(metaclass=SingletonMeta):
    def __init__(self):
        self._config = {}
        self._load_defaults()
        self._load_from_env()

    def _load_defaults(self):
        self._config = {
            "database": {
                "host": "localhost",
                "port": 5432,
                "name": "app_db",
                "user": "app",
                "password": "",
            },
            "redis": {"host": "localhost", "port": 6379, "db": 0},
            "logging": {"level": "INFO", "file": "app.log"},
            "api": {"host": "0.0.0.0", "port": 8000, "workers": 4},
        }

    def _load_from_env(self):
        for key in ["DATABASE_URL", "REDIS_URL", "LOG_LEVEL", "API_PORT"]:
            value = os.environ.get(key)
            if value:
                if key == "DATABASE_URL":
                    self._config["database"]["host"] = value
                elif key == "LOG_LEVEL":
                    self._config["logging"]["level"] = value
                elif key == "API_PORT":
                    self._config["api"]["port"] = int(value)

    def get(self, *keys):
        value = self._config
        for key in keys:
            value = value[key]
        return value

    def to_dict(self):
        return dict(self._config)
```

## Common Mistakes

```python
# MISTAKE 1: __init__ running every time
class BadSingleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        self.counter = 0  # Reset every time!


b1 = BadSingleton()
b1.counter = 42
b2 = BadSingleton()
print(b2.counter)  # 0 — __init__ reset it!
```

```python
# MISTAKE 2: Not thread-safe
class UnsafeSingleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            import time
            time.sleep(0.001)
            cls._instance = super().__new__(cls)
        return cls._instance


# Multiple threads may both see _instance is None
```

```python
# MISTAKE 3: Singleton as global state in tests
# Tests become order-dependent and hard to isolate
```

```python
# MISTAKE 4: Using singleton for everything
class UserManager(metaclass=SingletonMeta):
    pass  # What if you need multiple user contexts?
```

```python
# MISTAKE 5: Pickling breaks singletons
import pickle
s = SingletonNew("test")
data = pickle.dumps(s)
s2 = pickle.loads(data)
print(s is s2)  # False — new instance created!
```

## Best Practices

```python
# 1. Use module-level singletons (most Pythonic)
# myapp/config.py
class _Config:
    pass

config = _Config()

# 2. Use metaclass for framework-level singletons
# 3. Always guard __init__ with _initialized flag
# 4. Use locks for thread safety
# 5. Provide a reset method for testing
# 6. Consider dependency injection instead
# 7. Document why a singleton is needed
```

```python
# Testable singleton with reset capability
class TestableSingleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._initialized = False
        return cls._instance

    def __init__(self):
        if not self._initialized:
            self._initialized = True
            self.state = {}

    @classmethod
    def reset(cls):
        cls._instance = None


import unittest

class TestSingleton(unittest.TestCase):
    def setUp(self):
        TestableSingleton.reset()

    def test_singleton(self):
        s1 = TestableSingleton()
        s2 = TestableSingleton()
        self.assertIs(s1, s2)

    def tearDown(self):
        TestableSingleton.reset()
```

## Interview Questions

```python
# Q1: How does __new__ differ from __init__ in singleton implementation?
# __new__ creates the object (class method, returns instance)
# __init__ initializes the object (instance method, returns None)
```

```python
# Q2: How would you make a singleton thread-safe?
# Answer: Use double-checked locking with a threading.Lock
```

```python
# Q3: What is the Borg pattern?
# Answer: Multiple objects sharing the same state via __dict__
# vs Singleton — same identity AND state
```

```python
# Q4: When should you NOT use a singleton?
# - When you need multiple independent instances
# - When it makes testing difficult
# - When it introduces hidden global state
# - When dependency injection would be cleaner
```

```python
# Q5: Implement a singleton that supports inheritance
# Use metaclass SingletonRegistry — each subclass gets its own instance
```

## Coding Challenges

```python
# Challenge 1: Implement a thread-safe singleton cache
# with TTL (time-to-live) for cached items
```

```python
# Challenge 2: Create a singleton event bus
# that allows publish/subscribe across modules
```

```python
# Challenge 3: Implement a singleton database connection
# pool that handles connection failures and retries
```

```python
# Challenge 4: Create a singleton configuration that
# auto-reloads when the config file changes on disk
```

```python
# Challenge 5: Implement a singleton logger that
# supports multiple output handlers with different levels
```

## Summary

The Singleton pattern restricts a class to a single instance and provides global access. Python offers several implementation strategies: `__new__` override, Borg/Monostate pattern, metaclasses, module-level imports, and decorators. Each approach has trade-offs regarding inheritance, thread safety, and testability. The module-level singleton is the most Pythonic, while the metaclass approach offers the best inheritance support. Thread safety requires careful locking, and singletons should be used sparingly as they introduce global state that can complicate testing and maintainability.

## Related Topics

- Factory Pattern: Can provide singleton-scoped objects
- Proxy Pattern: Can control access to singleton objects
- Dependency Injection: Alternative to singletons for shared services
- Metaclasses: Python mechanism enabling singleton metaclass
- Thread Safety: Essential for concurrent singleton access
- Module System: Python modules are natural singletons
