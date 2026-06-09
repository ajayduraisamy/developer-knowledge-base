# Abstraction - ABC, @abstractmethod, interfaces

## Introduction

Abstraction is the OOP principle of hiding implementation details and exposing only the essential features of an object. In Python, abstraction is achieved through Abstract Base Classes (ABCs), the `@abstractmethod` decorator, and interface-like protocols. ABCs define a contract that subclasses must fulfill, ensuring consistent behavior across different implementations while hiding complexity from the consumer.

## ABC class

### What It Is

An Abstract Base Class (ABC) is a class that cannot be instantiated directly. It defines a template of methods and properties that subclasses must implement. ABCs are created by inheriting from `abc.ABC` or using the `ABCMeta` metaclass.

```python
from abc import ABC

class Shape(ABC):
    pass

# s = Shape()  # TypeError: Can't instantiate abstract class
```

### Why It Is Important

ABCs enforce a programming contract. They ensure that all subclasses provide specific functionality, reducing runtime errors and making code more predictable. They also document the expected interface for developers working with the codebase.

### How It Works Internally

The `ABCMeta` metaclass tracks abstract methods defined with `@abstractmethod`. When a class is instantiated, Python checks that all abstract methods have been implemented. If any are missing, it raises a `TypeError`. The registry (`__isabstractmethod__`) marks methods and the class itself as abstract.

```python
from abc import ABCMeta

class MyABC(metaclass=ABCMeta):
    pass
```

### Syntax

```python
from abc import ABC, abstractmethod

class AbstractClass(ABC):
    @abstractmethod
    def required_method(self):
        pass

    def concrete_method(self):
        return "Implemented"
```

### Beginner Examples

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def make_sound(self):
        pass

    def sleep(self):
        return "Zzz..."

class Dog(Animal):
    def make_sound(self):
        return "Woof!"

class Cat(Animal):
    def make_sound(self):
        return "Meow"

animals = [Dog(), Cat()]
for a in animals:
    print(a.make_sound(), a.sleep())
```

### Intermediate Examples

```python
from abc import ABC, abstractmethod
import json, xml.etree.ElementTree as ET

class Serializer(ABC):
    @abstractmethod
    def serialize(self, data: dict) -> str:
        pass

    @abstractmethod
    def deserialize(self, data: str) -> dict:
        pass

    def to_file(self, data: dict, filename: str) -> None:
        serialized = self.serialize(data)
        with open(filename, "w") as f:
            f.write(serialized)

class JSONSerializer(Serializer):
    def serialize(self, data: dict) -> str:
        return json.dumps(data, indent=2)

    def deserialize(self, data: str) -> dict:
        return json.loads(data)

class XMLSerializer(Serializer):
    def serialize(self, data: dict) -> str:
        root = ET.Element("data")
        for key, value in data.items():
            child = ET.SubElement(root, key)
            child.text = str(value)
        return ET.tostring(root, encoding="unicode")

    def deserialize(self, data: str) -> dict:
        root = ET.fromstring(data)
        return {child.tag: child.text for child in root}

def export(serializer: Serializer, data: dict) -> None:
    print(serializer.serialize(data))

export(JSONSerializer(), {"name": "Alice"})
export(XMLSerializer(), {"name": "Alice"})
```

### Advanced Examples

```python
from abc import ABC, abstractmethod
from typing import List, Optional, Generic, TypeVar

T = TypeVar("T")

class Repository(ABC, Generic[T]):
    @abstractmethod
    def get(self, id: int) -> Optional[T]:
        pass

    @abstractmethod
    def get_all(self) -> List[T]:
        pass

    @abstractmethod
    def add(self, entity: T) -> T:
        pass

    @abstractmethod
    def update(self, entity: T) -> T:
        pass

    @abstractmethod
    def delete(self, id: int) -> bool:
        pass

class User:
    def __init__(self, id: int, name: str, email: str):
        self.id = id
        self.name = name
        self.email = email

class InMemoryUserRepository(Repository[User]):
    def __init__(self):
        self._store: dict[int, User] = {}

    def get(self, id: int) -> Optional[User]:
        return self._store.get(id)

    def get_all(self) -> List[User]:
        return list(self._store.values())

    def add(self, entity: User) -> User:
        self._store[entity.id] = entity
        return entity

    def update(self, entity: User) -> User:
        if entity.id not in self._store:
            raise ValueError(f"User {entity.id} not found")
        self._store[entity.id] = entity
        return entity

    def delete(self, id: int) -> bool:
        if id in self._store:
            del self._store[id]
            return True
        return False

# The interface is clearly defined — switch implementations freely
repo: Repository[User] = InMemoryUserRepository()
repo.add(User(1, "Alice", "alice@example.com"))
print(repo.get(1).name)
```

### Real-World Use Cases

- **Django model fields**: `Field` base class with abstract methods
- **Plugin systems**: define plugin interface via ABCs
- **Database adapters**: abstract connection/ query interface
- **Web framework middleware**: abstract middleware classes
- **Testing**: mock objects implement same ABC as real objects

### Common Mistakes

1. **Forgetting `@abstractmethod`**: methods in ABCs aren't automatically abstract
2. **Calling `super().__init__()` without ABC's `__init__`**: ABCs can have `__init__`
3. **Creating non-abstract ABCs**: unnecessary abstraction adds complexity without benefit
4. **Over-abstracting**: not every class needs an ABC — YAGNI principle

### Best Practices

- Use ABCs when you need to define a contract for multiple implementations
- Keep ABCs focused on a single responsibility
- ABCs can (and often should) include concrete methods
- Name ABCs to convey their abstract nature (e.g., `BaseSerializer`, `AbstractRepository`)
- Use `@abstractmethod` on properties, classmethods, and staticmethods too

### Performance Considerations

- ABCs add negligible overhead at class definition time
- `isinstance` checks with ABCs are fast (cached registry)
- Method dispatch through ABCs is identical to normal method calls
- Abstract method checking only occurs at instantiation, not during normal execution

## @abstractmethod decorator

### What It Is

`@abstractmethod` is a decorator that marks a method as abstract — it must be overridden by any concrete subclass. It can be applied to instance methods, class methods, static methods, and properties.

```python
from abc import abstractmethod

class Base:
    @abstractmethod
    def doit(self):
        pass
```

### Why It Is Important

Without `@abstractmethod`, Python's ABC mechanism cannot enforce implementation. The decorator is the mechanism that makes the contract binding — subclasses that don't implement the method cannot be instantiated.

### How It Works Internally

`@abstractmethod` sets `__isabstractmethod__ = True` on the function object. `ABCMeta.__call__` checks `__isabstractmethod__` on the class and all its members before allowing instantiation.

```python
# Decorating at different levels
@abstractmethod
def method(self): pass          # Instance method

@classmethod
@abstractmethod
def cmethod(cls): pass          # Class method (order matters!)

@staticmethod
@abstractmethod
def smethod(): pass             # Static method

@property
@abstractmethod
def prop(self): pass            # Abstract property
```

### Syntax

```python
from abc import ABC, abstractmethod

class Database(ABC):
    @abstractmethod
    def connect(self):
        pass

    @abstractmethod
    def disconnect(self):
        pass

    @classmethod
    @abstractmethod
    def create_pool(cls, size: int):
        pass

    @property
    @abstractmethod
    def is_connected(self) -> bool:
        pass
```

### Beginner Examples

```python
from abc import ABC, abstractmethod

class PaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount):
        pass

    @abstractmethod
    def refund(self, transaction_id):
        pass

    def get_status(self, transaction_id):
        return f"Checking status of {transaction_id}"

class Stripe(PaymentGateway):
    def charge(self, amount):
        return f"Stripe charged ${amount}"

    def refund(self, transaction_id):
        return f"Stripe refunded {transaction_id}"

class PayPal(PaymentGateway):
    def charge(self, amount):
        return f"PayPal processed ${amount}"

    def refund(self, transaction_id):
        return f"PayPal reversed {transaction_id}"

gateway = Stripe()
print(gateway.charge(50))
print(gateway.get_status("txn_123"))
```

### Intermediate Examples

```python
from abc import ABC, abstractmethod

class DataSource(ABC):
    @abstractmethod
    def fetch(self, query: str) -> list:
        pass

    @abstractmethod
    def count(self) -> int:
        pass

    @classmethod
    @abstractmethod
    def from_config(cls, config: dict) -> "DataSource":
        pass

class APIDataSource(DataSource):
    def __init__(self, base_url: str):
        self.base_url = base_url
        self._cache = {}

    def fetch(self, query: str) -> list:
        import requests
        response = requests.get(f"{self.base_url}/{query}")
        return response.json()

    def count(self) -> int:
        return len(self._cache) if self._cache else 0

    @classmethod
    def from_config(cls, config: dict) -> "APIDataSource":
        return cls(config["base_url"])

    # Can still have abstract-ish methods with default behavior
    def clear_cache(self):
        self._cache.clear()

class FileDataSource(DataSource):
    def __init__(self, filepath: str):
        self.filepath = filepath

    def fetch(self, query: str) -> list:
        import json
        with open(self.filepath) as f:
            data = json.load(f)
        return data.get(query, [])

    def count(self) -> int:
        import json
        with open(self.filepath) as f:
            return len(json.load(f))

    @classmethod
    def from_config(cls, config: dict) -> "FileDataSource":
        return cls(config["filepath"])

sources: list[DataSource] = [
    APIDataSource("https://api.example.com"),
    FileDataSource("data.json")
]
```

### Advanced Examples

```python
from abc import ABC, abstractmethod
from typing import Any, Protocol, runtime_checkable

class Plugin(ABC):
    """Plugin system using abstract methods."""

    @abstractmethod
    def name(self) -> str:
        pass

    @abstractmethod
    def version(self) -> str:
        pass

    @abstractmethod
    def execute(self, context: dict) -> dict:
        pass

    @abstractmethod
    def validate(self, config: dict) -> bool:
        pass

    # Template method — concrete method using abstract steps
    def run(self, context: dict, config: dict) -> dict:
        if not self.validate(config):
            raise ValueError(f"Invalid config for plugin {self.name()}")
        result = self.execute(context)
        result["_plugin"] = self.name()
        result["_version"] = self.version()
        return result

class LoggingPlugin(Plugin):
    def name(self) -> str:
        return "logging"

    def version(self) -> str:
        return "1.0.0"

    def execute(self, context: dict) -> dict:
        message = context.get("message", "")
        print(f"[LOG] {message}")
        return {"logged": True, "message": message}

    def validate(self, config: dict) -> bool:
        return "log_level" in config

class MetricsPlugin(Plugin):
    def name(self) -> str:
        return "metrics"

    def version(self) -> str:
        return "2.1.0"

    def execute(self, context: dict) -> dict:
        return {
            "duration": context.get("duration", 0),
            "memory": context.get("memory", 0)
        }

    def validate(self, config: dict) -> bool:
        return True

# Now we can use isinstance checks
def is_valid_plugin(obj: Any) -> bool:
    return isinstance(obj, Plugin)

plugins = [LoggingPlugin(), MetricsPlugin()]
for p in plugins:
    result = p.run({"message": "hello"}, {"log_level": "info"})
    print(result)
```

### Real-World Use Cases

- **Python's `collections.abc`**: `Iterable`, `Mapping`, `Sequence` ABCs
- **Django REST Framework**: serializers with abstract `create()`/`update()` methods
- **PyTest fixtures**: abstract fixture factories
- **Machine learning pipelines**: abstract `fit()`/`predict()` methods

### Common Mistakes

1. **Wrong decorator order**: `@abstractmethod` must be the innermost decorator
2. **Missing `ABC` base class**: ABC won't enforce without `ABC` metaclass
3. **Abstract `__init__`**: `__init__` can't be abstract (no `super().__init__` call issue)
4. **Instantiating abstract class**: subclasses must implement all abstract methods

### Best Practices

- Always put `@abstractmethod` as the innermost decorator
- Provide concrete default implementations when possible
- Use `@abstractmethod` on properties for abstract attributes
- Keep abstract method signatures consistent across implementations
- Document what each abstract method should do

### Performance Considerations

- `@abstractmethod` has zero runtime cost on successfully instantiated objects
- The check only happens at `__init__` time in the metaclass
- Proper use can reduce debugging time by catching missing implementations early

## Interfaces in Python

### What It Is

Python doesn't have a formal `interface` keyword like Java or Go. Instead, interfaces are implemented using ABCs (with only abstract methods) or Protocols (structural subtyping). An interface defines a set of methods that a class must implement, without providing any implementation itself.

```python
# Interface using ABC (formal interface)
class Readable(ABC):
    @abstractmethod
    def read(self) -> str:
        pass

    @abstractmethod
    def close(self) -> None:
        pass
```

### Why It Is Important

Interfaces decouple specification from implementation. They allow different implementations to be swapped without changing client code. In large codebases, interfaces define clear boundaries between components.

### How It Works Internally

ABC-based interfaces work the same as any ABC — the metaclass checks for abstract method implementations. Protocol-based interfaces (Python 3.8+) use structural subtyping — a class satisfies a Protocol if it has the required methods, even without explicit inheritance.

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> str: ...

class Circle:
    def draw(self) -> str:
        return "○"

def render(obj: Drawable) -> None:
    print(obj.draw())

render(Circle())  # Works — Circle satisfies Drawable structurally
```

### Syntax

```python
# ABC-based interface (nominal)
from abc import ABC, abstractmethod

class Flyable(ABC):
    @abstractmethod
    def fly(self) -> str:
        pass

# Protocol-based interface (structural)
from typing import Protocol

class Swimmable(Protocol):
    def swim(self) -> str:
        ...

# Mixing both
class Duck(Flyable):
    def fly(self) -> str:
        return "Flying"
    def swim(self) -> str:
        return "Swimming"
```

### Beginner Examples

```python
from abc import ABC, abstractmethod

class Sortable(ABC):
    @abstractmethod
    def sort(self, items):
        pass

class BubbleSort(Sortable):
    def sort(self, items):
        n = len(items)
        for i in range(n):
            for j in range(0, n - i - 1):
                if items[j] > items[j + 1]:
                    items[j], items[j + 1] = items[j + 1], items[j]
        return items

class QuickSort(Sortable):
    def sort(self, items):
        if len(items) <= 1:
            return items
        pivot = items[0]
        left = [x for x in items[1:] if x <= pivot]
        right = [x for x in items[1:] if x > pivot]
        return self.sort(left) + [pivot] + self.sort(right)

def process(sorter: Sortable, data):
    return sorter.sort(data.copy())

print(process(BubbleSort(), [3, 1, 4, 1, 5]))
print(process(QuickSort(), [3, 1, 4, 1, 5]))
```

### Intermediate Examples

```python
from typing import Protocol, List, runtime_checkable

@runtime_checkable
class IterableByYear(Protocol):
    def years(self) -> List[int]:
        ...

    def items_for_year(self, year: int) -> List[str]:
        ...

class MovieDatabase:
    def __init__(self):
        self._data = {
            2020: ["Parasite", "1917"],
            2021: ["Nomadland", "CODA"],
            2022: ["Everything Everywhere All at Once"],
        }

    def years(self) -> List[int]:
        return list(self._data.keys())

    def items_for_year(self, year: int) -> List[str]:
        return self._data.get(year, [])

class MusicLibrary:
    def __init__(self):
        self._data = {
            "2020": ["After Hours", "Folklore"],
            "2021": ["Happier Than Ever", "Planet Her"],
        }

    def years(self) -> List[int]:
        return [int(y) for y in self._data.keys()]

    def items_for_year(self, year: int) -> List[str]:
        return self._data.get(str(year), [])

def browse(source: IterableByYear, year: int) -> None:
    if year in source.years():
        print(f"Items from {year}: {source.items_for_year(year)}")

browse(MovieDatabase(), 2020)
browse(MusicLibrary(), 2020)
print(isinstance(MovieDatabase(), IterableByYear))  # True (with @runtime_checkable)
```

### Advanced Examples

```python
from abc import ABC, abstractmethod
from typing import Protocol, TypeVar, Generic, List

T = TypeVar("T")

# Formal interface (ABC-based)
class RepositoryInterface(ABC):
    @abstractmethod
    def find_by_id(self, id: int):
        pass

    @abstractmethod
    def find_all(self) -> list:
        pass

    @abstractmethod
    def save(self, entity) -> None:
        pass

    @abstractmethod
    def delete(self, id: int) -> None:
        pass

# Structural interface (Protocol)
class CacheInterface(Protocol):
    def get(self, key: str):
        ...

    def set(self, key: str, value, ttl: int = 300) -> None:
        ...

    def invalidate(self, pattern: str) -> None:
        ...

# Implementation
class PostgresRepository(RepositoryInterface):
    def __init__(self, connection_string: str):
        self.conn_string = connection_string

    def find_by_id(self, id: int):
        return {"id": id, "name": f"User {id}"}

    def find_all(self) -> list:
        return [{"id": 1, "name": "Alice"}]

    def save(self, entity) -> None:
        print(f"Saving {entity}")

    def delete(self, id: int) -> None:
        print(f"Deleting {id}")

class RedisCache:
    def get(self, key: str):
        print(f"Redis get {key}")
        return None

    def set(self, key: str, value, ttl: int = 300) -> None:
        print(f"Redis set {key}={value} (ttl={ttl})")

    def invalidate(self, pattern: str) -> None:
        print(f"Redis invalidate {pattern}")

    # Additional methods not in interface
    def flush_all(self) -> None:
        print("Redis flush all")

class Service:
    def __init__(self, repo: RepositoryInterface, cache: CacheInterface):
        self.repo = repo
        self.cache = cache

    def get_user(self, id: int):
        cached = self.cache.get(f"user:{id}")
        if cached:
            return cached
        user = self.repo.find_by_id(id)
        self.cache.set(f"user:{id}", user)
        return user

service = Service(PostgresRepository("pg://localhost"), RedisCache())
print(service.get_user(1))
```

### Real-World Use Cases

- **Dependency injection**: interfaces define contract for injectable services
- **Repository pattern**: abstract data access behind an interface
- **Strategy pattern**: interchangeable algorithms behind a common interface
- **Adapter pattern**: adapt different systems to a common interface
- **API versioning**: newer API implementations satisfy the same interface

### Common Mistakes

1. **Making interfaces too large**: violating Interface Segregation Principle
2. **Using ABCs when Protocols are more appropriate**: nominal vs structural typing
3. **Implementing interfaces partially**: leads to `TypeError` at instantiation
4. **Not using `@runtime_checkable`**: Protocols don't support `isinstance` by default
5. **Over-relying on interfaces**: unnecessary abstraction adds complexity

### Best Practices

- Prefer Protocols for duck-typing style interfaces (structural)
- Prefer ABCs when you want to provide some default implementation
- Keep interfaces small and focused (ISP: Interface Segregation Principle)
- Name interfaces clearly: `Readable`, `Writable`, `Serializable`
- Use `@runtime_checkable` sparingly (it has overhead for Protocols)

### Performance Considerations

- Interface dispatch through ABCs is identical to normal method dispatch
- `isinstance` with `@runtime_checkable` Protocols has overhead (checks at runtime)
- Static Protocol checks in mypy have no runtime cost
- ABC registration (`register()`) is fast and enables `isinstance` checks without inheritance

### Interview Questions

1. What's the difference between ABC and Protocol in Python?
2. How does `@abstractmethod` work under the hood?
3. Can you have abstract `__init__` methods? Why or why not?
4. What is the Interface Segregation Principle and how does it apply to Python?
5. How would you design a plugin system using ABCs?
6. What's the difference between nominal and structural typing?

### Coding Challenges

1. Design an abstract `NotificationSender` with `send()` method, then implement `EmailSender`, `SMSSender`, `PushSender`.
2. Create a `PersistenceInterface` with `save`/`load`/`delete` and implement file-based and database-backed versions.
3. Build a `SortStrategy` interface with multiple implementations (bubble, merge, quick) and a context class that uses any strategy.

### Related Topics

ABC, @abstractmethod, Protocols, Duck Typing, Dependency Injection, Design Patterns
