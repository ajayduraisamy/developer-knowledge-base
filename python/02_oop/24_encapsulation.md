# Encapsulation - Public, protected (_), private (__) attributes

## Introduction

Encapsulation is the OOP principle of bundling data with the methods that operate on it, while restricting direct access to internal state. Python enforces encapsulation through naming conventions rather than strict access modifiers. Unlike languages like Java or C++, Python trusts developers to respect conventions. Understanding Python's approach to encapsulation is crucial for writing robust, maintainable code that clearly communicates its intended API surface.

## Public attributes

### What It Is

Public attributes in Python are attributes without any leading underscores. They are intended to be part of the class's public API and can be freely accessed and modified from outside the class.

```python
class Car:
    def __init__(self, brand):
        self.brand = brand  # Public attribute
```

### Why It Is Important

Public attributes define the interface that other code interacts with. A well-designed public API is stable, documented, and intuitive. Public attributes should be safe to expose and should not leak implementation details.

### How It Works Internally

Public attributes are stored in the instance's `__dict__` and accessed directly through `obj.attribute`. There is no access control mechanism — any code that can see the object can read or write public attributes.

```python
obj = SomeClass()
obj.public_attr = "new value"  # Always possible
```

### Syntax

```python
class Person:
    def __init__(self, name, age):
        self.name = name       # Public
        self.age = age         # Public

p = Person("Alice", 30)
print(p.name)  # Accessible
p.age = 31     # Modifiable
```

### Beginner Examples

```python
class Book:
    def __init__(self, title, author, pages):
        self.title = title
        self.author = author
        self.pages = pages
        self.current_page = 0

    def read(self, pages):
        self.current_page += pages
        if self.current_page > self.pages:
            self.current_page = self.pages

book = Book("1984", "Orwell", 328)
print(book.title)          # 1984
print(book.current_page)   # 0
book.read(50)
print(book.current_page)   # 50
book.current_page = 999    # Allowed but semantically wrong
```

### Intermediate Examples

```python
class Temperature:
    def __init__(self, celsius=0):
        self.celsius = celsius
        self.fahrenheit = celsius * 9 / 5 + 32

    def __repr__(self):
        return f"{self.celsius}°C / {self.fahrenheit}°F"

t = Temperature(100)
print(t.fahrenheit)  # 212.0
t.celsius = 0        # Does NOT update fahrenheit!
print(t.fahrenheit)  # Still 212.0 — design flaw
```

### Advanced Examples

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class ShoppingCart:
    items: List[str] = field(default_factory=list)
    coupon_code: str = ""

    def add_item(self, item: str) -> None:
        self.items.append(item)

    def total_items(self) -> int:
        return len(self.items)

    def apply_discount(self, code: str) -> None:
        self.coupon_code = code

cart = ShoppingCart()
cart.add_item("Laptop")
cart.add_item("Mouse")
print(cart.items)  # Accessible, but exposing internal list
```

### Real-World Use Cases

- **Configuration objects**: public attributes for settings
- **DTOs (Data Transfer Objects)**: simple public attributes for data
- **API responses**: models with public fields
- **Command objects**: public parameters for commands

### Common Mistakes

1. **Exposing mutable internals**: returning internal lists/ dicts allows external mutation
2. **Breaking invariants**: allowing external attribute changes that violate class constraints
3. **No separation of interface and implementation**: everything is public by default

### Best Practices

- Keep public API minimal and intentional
- Return copies of internal mutable objects or use `@property` for controlled access
- Document public attributes in docstrings
- Use `@dataclass` for simple data containers with public fields

### Performance Considerations

- Public attribute access is the fastest type of attribute access
- No overhead from property decorators or descriptor protocols
- Direct attribute access is optimized in CPython

## Protected attributes (_)

### What It Is

Protected attributes have a single leading underscore (`_attr`). This is a convention indicating that the attribute is intended for internal use by the class and its subclasses, not for external access.

```python
class Worker:
    def __init__(self):
        self._status = "idle"  # Protected
```

### Why It Is Important

Protected attributes signal to other developers that a given attribute is part of the implementation, not the public API. Subclasses are allowed to access and override protected members, but external code should treat them as private implementation details.

### How It Works Internally

Python does not enforce protected access at all. The single underscore is purely a naming convention. The attribute is stored normally in `__dict__` and is fully accessible. Tools like linters and IDEs may warn about accessing protected attributes from outside the class.

```python
obj._protected  # Works fine, but linters will warn
```

### Syntax

```python
class Database:
    def __init__(self):
        self._connection = None   # Protected — subclass may access
        self._is_connected = False

    def connect(self):
        self._connection = "connected"
        self._is_connected = True

    def disconnect(self):
        self._connection = None
        self._is_connected = False

class PostgresDB(Database):
    def query(self, sql):
        if not self._is_connected:  # Subclass accessing protected
            self.connect()
        return f"Executed: {sql}"
```

### Beginner Examples

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self._balance = balance  # Protected — use methods to interact
        self._transaction_count = 0

    def deposit(self, amount):
        if amount > 0:
            self._balance += amount
            self._transaction_count += 1

    def get_balance(self):
        return self._balance

class SavingsAccount(BankAccount):
    def apply_interest(self, rate):
        interest = self._balance * rate  # Subclass accessing protected
        self.deposit(interest)

acct = SavingsAccount("Alice", 1000)
acct.apply_interest(0.05)
print(acct.get_balance())  # 1050
print(acct._balance)       # Works but frowned upon
```

### Intermediate Examples

```python
class FileHandler:
    def __init__(self, filename):
        self.filename = filename
        self._file = None
        self._position = 0

    def open(self):
        self._file = open(self.filename, "r")

    def _read_chunk(self, size=1024):
        if self._file:
            data = self._file.read(size)
            self._position += len(data)
            return data
        return ""

    def close(self):
        if self._file:
            self._file.close()
            self._file = None

class BufferedFileHandler(FileHandler):
    def __init__(self, filename, buffer_size=4096):
        super().__init__(filename)
        self._buffer_size = buffer_size
        self._buffer = ""

    def read_line(self):
        while "\n" not in self._buffer:
            chunk = self._read_chunk(self._buffer_size)  # Using protected method
            if not chunk:
                break
            self._buffer += chunk
        line, self._buffer = self._buffer.split("\n", 1) if "\n" in self._buffer else (self._buffer, "")
        return line
```

### Advanced Examples

```python
from abc import ABC, abstractmethod
from typing import Optional

class DataPipeline(ABC):
    def __init__(self, config: dict):
        self.config = config
        self._data: Optional[dict] = None
        self._errors: list = []

    def _validate_config(self) -> bool:
        return all(k in self.config for k in ["source", "destination"])

    @abstractmethod
    def _extract(self) -> dict:
        pass

    @abstractmethod
    def _transform(self, data: dict) -> dict:
        pass

    @abstractmethod
    def _load(self, data: dict) -> bool:
        pass

    def run(self) -> bool:
        if not self._validate_config():
            self._errors.append("Invalid configuration")
            return False
        try:
            self._data = self._extract()
            self._data = self._transform(self._data)
            return self._load(self._data)
        except Exception as e:
            self._errors.append(str(e))
            return False

class CSVPipeline(DataPipeline):
    def _extract(self) -> dict:
        return {"rows": [{"id": 1, "name": "Alice"}]}

    def _transform(self, data: dict) -> dict:
        data["processed"] = True
        return data

    def _load(self, data: dict) -> bool:
        print(f"Loaded {len(data.get('rows', []))} rows")
        return True

pipeline = CSVPipeline({"source": "file.csv", "destination": "db"})
pipeline.run()
```

### Real-World Use Cases

- **Framework internals**: Django's `_meta`, `_state` attributes on models
- **Plugin systems**: protected methods that plugins can override
- **Template Method pattern**: abstract protected methods for subclasses to implement
- **Caching**: `_cache` dictionary for internal caching logic

### Common Mistakes

1. **Treating `_` as real protection**: external code can still access it
2. **Over-exposing internals**: marking too many things as protected when they should be private
3. **Subclass violating contract**: accessing protected attributes meant for internal use only
4. **Name collision with double underscore**: confusing `_` and `__` usage

### Best Practices

- Use `_` for attributes that are implementation details
- Document which protected attributes subclasses may access
- Prefer `@property` over direct protected attribute access for public APIs
- Don't rely on `_` for security — it's a convention, not enforcement

### Performance Considerations

- Protected attributes have the same access speed as public attributes
- No overhead from name mangling (unlike double underscore)
- Property wrapping of protected attributes adds method call overhead

## Private attributes (__)

### What It Is

Private attributes have a double leading underscore (`__attr`). Python applies name mangling to these attributes, making them harder to accidentally access from outside the class.

```python
class Secret:
    __key = "abc123"  # Becomes _Secret__key
```

### Why It Is Important

Name mangling prevents accidental overrides in subclasses and reduces the risk of external code depending on implementation details. It's Python's strongest form of encapsulation, though it's still not true privacy — the mangled name can still be accessed deliberately.

### How It Works Internally

When the compiler encounters a double-underscore attribute inside a class definition, it transforms the name to `_ClassName__attribute`. This mangling happens at compile time, not runtime.

```python
class A:
    def __init__(self):
        self.__secret = 42    # Becomes _A__secret

class B(A):
    def __init__(self):
        super().__init__()
        self.__secret = 99    # Becomes _B__secret, doesn't override A's

obj = B()
print(obj.__secret)        # AttributeError
print(obj._A__secret)      # 42
print(obj._B__secret)      # 99
```

### Syntax

```python
class Counter:
    def __init__(self):
        self.__count = 0   # Private — name mangled to _Counter__count

    def increment(self):
        self.__count += 1

    def get_count(self):
        return self.__count

c = Counter()
c.increment()
# print(c.__count)         # AttributeError
print(c.get_count())       # 1
print(c._Counter__count)   # 1 (name mangling can be bypassed)
```

### Beginner Examples

```python
class User:
    def __init__(self, username, password):
        self.username = username
        self.__password = password  # Private
        self.__login_attempts = 0

    def check_password(self, candidate):
        self.__login_attempts += 1
        return self.__password == candidate

    def get_login_attempts(self):
        return self.__login_attempts

user = User("alice", "s3cret")
print(user.username)            # alice
# print(user.__password)        # AttributeError
print(user.check_password("wrong"))  # False
print(user.get_login_attempts())     # 1
```

### Intermediate Examples

```python
class Cache:
    def __init__(self, maxsize=128):
        self.__store = {}
        self.__maxsize = maxsize
        self.__hits = 0
        self.__misses = 0

    def get(self, key):
        if key in self.__store:
            self.__hits += 1
            return self.__store[key]
        self.__misses += 1
        return None

    def set(self, key, value):
        if len(self.__store) >= self.__maxsize:
            self.__evict()
        self.__store[key] = value

    def __evict(self):
        import time
        self.__store.pop(next(iter(self.__store)))

    def stats(self):
        total = self.__hits + self.__misses
        hit_rate = self.__hits / total * 100 if total else 0
        return {
            "hits": self.__hits,
            "misses": self.__misses,
            "hit_rate": f"{hit_rate:.1f}%",
            "size": len(self.__store)
        }

class TimedCache(Cache):
    def __init__(self, maxsize=128, ttl=60):
        super().__init__(maxsize)
        self.__ttl = ttl  # _TimedCache__ttl — separate from Cache's __
        self.__timestamps = {}

    def set(self, key, value):
        super().set(key, value)
        self.__timestamps[key] = __import__("time").time()

# Note: Cache.__evict is _Cache__evict — subclass can't access it
```

### Advanced Examples

```python
class ConnectionPool:
    def __init__(self, min_size=5, max_size=20):
        self.__min_size = min_size
        self.__max_size = max_size
        self.__connections = []
        self.__in_use = set()
        self.__total_created = 0
        self.__initialize_pool()

    def __initialize_pool(self):
        for _ in range(self.__min_size):
            conn = self.__create_connection()
            self.__connections.append(conn)
            self.__total_created += 1

    def __create_connection(self):
        return f"conn-{self.__total_created + 1}"

    def acquire(self):
        if self.__connections:
            conn = self.__connections.pop()
            self.__in_use.add(conn)
            return conn
        if self.__total_created < self.__max_size:
            conn = self.__create_connection()
            self.__total_created += 1
            self.__in_use.add(conn)
            return conn
        raise RuntimeError("No available connections")

    def release(self, conn):
        self.__in_use.discard(conn)
        self.__connections.append(conn)

    @property
    def stats(self):
        return {
            "available": len(self.__connections),
            "in_use": len(self.__in_use),
            "total_created": self.__total_created
        }

pool = ConnectionPool()
conn = pool.acquire()
pool.release(conn)
print(pool.stats)
# Can still access internals if truly needed:
# print(pool._ConnectionPool__connections)
```

### Real-World Use Cases

- **Authentication tokens**: passwords, API keys stored as private
- **Internal state machines**: private attributes for state tracking
- **Caching implementations**: private cache dicts
- **Sanitized public API**: hiding implementation details from consumers
- **ORM internals**: private fields for database connection state

### Common Mistakes

1. **Thinking `__` is true security**: name mangling is trivially bypassed
2. **Overusing `__`**: unnecessary name mangling complicates debugging and testing
3. **Accessing `__` attributes from subclasses**: `__private` in parent becomes `_Parent__private`
4. **Name mangling with `__method__`**: double underscores on both sides are magic methods, not private
5. **Using `__` for attributes accessed by properties**: direct attribute access in setters requires same mangling

### Best Practices

- Use `__` sparingly — only when you need to avoid accidental override in subclasses
- Prefer `_` by default for most internal implementation details
- Use `__` for values that should not be accidentally exposed in subclass APIs
- Document private attributes with the mangled name
- Never use `__` for security; store secrets in environment variables

### Performance Considerations

- Name mangling happens at compile time, so there's no runtime cost
- Access speed is identical to public attributes after name resolution
- `__slots__` works with mangled names — specify the mangled name in slots
- Introspection tools can see mangled names (e.g., `vars(obj)` shows them)

### Interview Questions

1. What is name mangling and when does it happen?
2. How does `_` differ from `__` in Python?
3. Can you access a private attribute from outside the class? How?
4. Why might a library author use `__` for internal attributes?
5. How does name mangling interact with inheritance?

### Coding Challenges

1. Create a `SafeDict` class with private internal storage, public `get`/`set`/`keys` methods.
2. Implement a `TemperatureSensor` with protected `_reading` and validation in the public API.
3. Build an `Ecosystem` with private state that prevents direct mutation of internal data structures.

### Related Topics

Properties, Name Mangling, Descriptors, `__slots__`, Data Classes
