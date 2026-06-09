# Factory - Factory Method, Abstract Factory, dynamic class selection

## Introduction

The Factory pattern provides an interface for creating objects without specifying their concrete classes. It encompasses several related patterns: Simple Factory, Factory Method, and Abstract Factory. These patterns are fundamental creational patterns that promote loose coupling by delegating object creation to specialized factory classes or methods.

## Why It Is Important

Factory patterns abstract the instantiation logic away from client code, making systems more maintainable, testable, and extensible. When object creation involves complex logic, conditional branching, or dependency resolution, factories centralize this logic and reduce code duplication. They enable dynamic class selection at runtime, simplify adding new product types, and support dependency injection frameworks.

## Syntax

Factory patterns rely on Python's dynamic typing and first-class functions. Simple factories are regular functions or classes that return instances. Factory Method uses inheritance with abstract creator classes. Abstract Factory composes multiple factory methods. Python's `__init_subclass__`, `register`, and `functools.singledispatch` can also implement factory-like dispatching.

## Examples

```python
# Simple Factory
from abc import ABC, abstractmethod
from typing import Dict, Type, Optional


class Animal(ABC):
    @abstractmethod
    def speak(self) -> str:
        pass


class Dog(Animal):
    def speak(self) -> str:
        return "Woof!"


class Cat(Animal):
    def speak(self) -> str:
        return "Meow!"


class Duck(Animal):
    def speak(self) -> str:
        return "Quack!"


class AnimalFactory:
    @staticmethod
    def create_animal(animal_type: str) -> Animal:
        animals = {
            "dog": Dog,
            "cat": Cat,
            "duck": Duck,
        }
        animal_class = animals.get(animal_type.lower())
        if not animal_class:
            raise ValueError(f"Unknown animal: {animal_type}")
        return animal_class()


factory = AnimalFactory()
dog = factory.create_animal("dog")
cat = factory.create_animal("cat")
print(dog.speak())  # Woof!
print(cat.speak())  # Meow!
```

```python
# Factory Method Pattern
class Creator(ABC):
    @abstractmethod
    def factory_method(self) -> Animal:
        pass

    def operation(self) -> str:
        animal = self.factory_method()
        return f"Creator: {animal.speak()}"


class DogCreator(Creator):
    def factory_method(self) -> Animal:
        return Dog()


class CatCreator(Creator):
    def factory_method(self) -> Animal:
        return Cat()


def client_code(creator: Creator):
    print(creator.operation())


client_code(DogCreator())  # Creator: Woof!
client_code(CatCreator())  # Creator: Meow!
```

```python
# Abstract Factory Pattern
class GUIFactory(ABC):
    @abstractmethod
    def create_button(self):
        pass

    @abstractmethod
    def create_checkbox(self):
        pass


class WindowsFactory(GUIFactory):
    def create_button(self):
        return WindowsButton()

    def create_checkbox(self):
        return WindowsCheckbox()


class MacFactory(GUIFactory):
    def create_button(self):
        return MacButton()

    def create_checkbox(self):
        return MacCheckbox()


class Button(ABC):
    @abstractmethod
    def render(self):
        pass


class WindowsButton(Button):
    def render(self):
        return "Windows Button"


class MacButton(Button):
    def render(self):
        return "Mac Button"


class Checkbox(ABC):
    @abstractmethod
    def render(self):
        pass


class WindowsCheckbox(Checkbox):
    def render(self):
        return "Windows Checkbox"


class MacCheckbox(Checkbox):
    def render(self):
        return "Mac Checkbox"


def create_ui(factory: GUIFactory):
    button = factory.create_button()
    checkbox = factory.create_checkbox()
    return f"{button.render()} + {checkbox.render()}"


print(create_ui(WindowsFactory()))  # Windows Button + Windows Checkbox
print(create_ui(MacFactory()))      # Mac Button + Mac Checkbox
```

```python
# Dynamic class selection with registration
class Plugin:
    _registry: Dict[str, Type["Plugin"]] = {}

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        name = kwargs.get("name", cls.__name__.lower())
        cls._registry[name] = cls

    @classmethod
    def create(cls, name: str, *args, **kwargs):
        plugin_class = cls._registry.get(name)
        if not plugin_class:
            raise ValueError(f"Unknown plugin: {name}")
        return plugin_class(*args, **kwargs)


class JSONExporter(Plugin, name="json"):
    def export(self, data):
        import json
        return json.dumps(data, indent=2)


class CSVExporter(Plugin, name="csv"):
    def export(self, data):
        if not data:
            return ""
        headers = data[0].keys()
        lines = [",".join(headers)]
        for row in data:
            lines.append(",".join(str(row.get(h, "")) for h in headers))
        return "\n".join(lines)


class XMLExporter(Plugin, name="xml"):
    def export(self, data):
        parts = ["<data>"]
        for item in data:
            parts.append("  <item>")
            for k, v in item.items():
                parts.append(f"    <{k}>{v}</{k}>")
            parts.append("  </item>")
        parts.append("</data>")
        return "\n".join(parts)


exporter = Plugin.create("json")
print(exporter.export([{"name": "Alice", "age": 30}]))
# {
#   "name": "Alice",
#   "age": 30
# }
```

## Beginner Examples

```python
# Simple factory for notification system
class Notification:
    def send(self, message):
        pass


class EmailNotification(Notification):
    def send(self, message):
        return f"Sending email: {message}"


class SMSNotification(Notification):
    def send(self, message):
        return f"Sending SMS: {message}"


class PushNotification(Notification):
    def send(self, message):
        return f"Sending push: {message}"


def create_notification(channel):
    factories = {
        "email": EmailNotification,
        "sms": SMSNotification,
        "push": PushNotification,
    }
    cls = factories.get(channel)
    if not cls:
        raise ValueError(f"Unknown channel: {channel}")
    return cls()


notif = create_notification("email")
print(notif.send("Hello!"))
```

```python
# Factory function with lambda factories
def make_document_formatter(format_type):
    formatters = {
        "plain": lambda text: text,
        "upper": lambda text: text.upper(),
        "lower": lambda text: text.lower(),
        "title": lambda text: text.title(),
        "reverse": lambda text: text[::-1],
    }
    formatter = formatters.get(format_type)
    if not formatter:
        raise ValueError(f"Unknown format: {format_type}")
    return formatter


fmt = make_document_formatter("title")
print(fmt("hello world"))  # Hello World
```

## Intermediate Examples

```python
# Factory for creating database connections
import sqlite3
from abc import ABC, abstractmethod
from typing import Any, Dict


class DatabaseConnection(ABC):
    @abstractmethod
    def connect(self):
        pass

    @abstractmethod
    def execute(self, query: str, params: tuple = ()) -> Any:
        pass

    @abstractmethod
    def close(self):
        pass


class SQLiteConnection(DatabaseConnection):
    def __init__(self, db_path: str):
        self.db_path = db_path
        self.conn = None

    def connect(self):
        self.conn = sqlite3.connect(self.db_path)
        return self

    def execute(self, query: str, params: tuple = ()) -> Any:
        cursor = self.conn.cursor()
        cursor.execute(query, params)
        self.conn.commit()
        return cursor

    def close(self):
        if self.conn:
            self.conn.close()


class PostgresConnection(DatabaseConnection):
    def __init__(self, host: str, port: int, db: str, user: str, password: str):
        self.config = {"host": host, "port": port, "db": db, "user": user, "password": password}
        self.conn = None

    def connect(self):
        import psycopg2
        self.conn = psycopg2.connect(**self.config)
        return self

    def execute(self, query: str, params: tuple = ()) -> Any:
        cursor = self.conn.cursor()
        cursor.execute(query, params)
        self.conn.commit()
        return cursor

    def close(self):
        if self.conn:
            self.conn.close()


class MySQLConnection(DatabaseConnection):
    def __init__(self, host: str, port: int, db: str, user: str, password: str):
        self.config = {"host": host, "port": port, "database": db, "user": user, "password": password}
        self.conn = None

    def connect(self):
        import mysql.connector
        self.conn = mysql.connector.connect(**self.config)
        return self

    def execute(self, query: str, params: tuple = ()) -> Any:
        cursor = self.conn.cursor()
        cursor.execute(query, params)
        self.conn.commit()
        return cursor

    def close(self):
        if self.conn:
            self.conn.close()


class DatabaseFactory:
    _connections: Dict[str, type] = {
        "sqlite": SQLiteConnection,
        "postgres": PostgresConnection,
        "mysql": MySQLConnection,
    }

    @classmethod
    def register(cls, name: str, conn_class: type):
        cls._connections[name] = conn_class

    @classmethod
    def create(cls, db_type: str, **kwargs) -> DatabaseConnection:
        conn_class = cls._connections.get(db_type)
        if not conn_class:
            raise ValueError(f"Unsupported database: {db_type}")
        return conn_class(**kwargs)


db = DatabaseFactory.create("sqlite", db_path=":memory:")
db.connect()
result = db.execute("SELECT 1")
print(result.fetchone())
db.close()
```

```python
# Factory with configuration-driven creation
import json
from pathlib import Path


class Task:
    def run(self):
        pass


class DataIngestionTask(Task):
    def __init__(self, config):
        self.source = config.get("source")
        self.destination = config.get("destination")

    def run(self):
        return f"Ingesting data from {self.source} to {self.destination}"


class DataTransformTask(Task):
    def __init__(self, config):
        self.transform_type = config.get("type", "default")

    def run(self):
        return f"Applying {self.transform_type} transformation"


class DataExportTask(Task):
    def __init__(self, config):
        self.format = config.get("format", "csv")
        self.target = config.get("target")

    def run(self):
        return f"Exporting to {self.format} at {self.target}"


class TaskFactory:
    _tasks = {
        "ingest": DataIngestionTask,
        "transform": DataTransformTask,
        "export": DataExportTask,
    }

    @classmethod
    def from_config(cls, config: dict) -> Task:
        task_type = config.get("type")
        task_class = cls._tasks.get(task_type)
        if not task_class:
            raise ValueError(f"Unknown task: {task_type}")
        return task_class(config.get("params", {}))

    @classmethod
    def from_json(cls, json_str: str) -> list:
        configs = json.loads(json_str)
        return [cls.from_config(cfg) for cfg in configs]


pipeline_config = """
[
    {"type": "ingest", "params": {"source": "s3://bucket", "destination": "/tmp/data"}},
    {"type": "transform", "params": {"type": "normalize"}},
    {"type": "export", "params": {"format": "parquet", "target": "s3://output"}}
]
"""

tasks = TaskFactory.from_json(pipeline_config)
for task in tasks:
    print(task.run())
```

## Advanced Examples

```python
# Abstract Factory with dependency injection container
from typing import Dict, Type, Callable, Any
from functools import partial


class ServiceContainer:
    def __init__(self):
        self._factories: Dict[str, Callable] = {}
        self._singletons: Dict[str, Any] = {}

    def register(self, name: str, factory: Callable, singleton: bool = False):
        self._factories[name] = (factory, singleton)

    def resolve(self, name: str, **overrides) -> Any:
        if name not in self._factories:
            raise KeyError(f"Service '{name}' not registered")
        factory, is_singleton = self._factories[name]
        if is_singleton and name in self._singletons:
            return self._singletons[name]
        instance = factory(container=self, **overrides)
        if is_singleton:
            self._singletons[name] = instance
        return instance


class Repository(ABC):
    @abstractmethod
    def find(self, id: int):
        pass


class UserRepository(Repository):
    def __init__(self, container: ServiceContainer = None):
        self.db = container.resolve("database") if container else None

    def find(self, id: int):
        return {"id": id, "name": "Alice", "email": "alice@example.com"}


class DatabaseService:
    def __init__(self, container: ServiceContainer = None):
        self.connected = False

    def connect(self):
        self.connected = True
        return "Connected to database"


class UserService:
    def __init__(self, container: ServiceContainer = None):
        self.repo = container.resolve("user_repository")

    def get_user(self, user_id: int):
        user = self.repo.find(user_id)
        return f"User: {user['name']} ({user['email']})"


container = ServiceContainer()
container.register("database", lambda container: DatabaseService(), singleton=True)
container.register("user_repository", lambda container: UserRepository(container))
container.register("user_service", lambda container: UserService(container))

service = container.resolve("user_service")
print(service.get_user(1))
```

```python
# Factory with classmethod inheritance pattern
class Serializer(ABC):
    @classmethod
    @abstractmethod
    def from_data(cls, data):
        pass

    @abstractmethod
    def serialize(self) -> str:
        pass


class JSONSerializer(Serializer):
    def __init__(self, obj):
        self.obj = obj

    @classmethod
    def from_data(cls, data):
        return cls(data)

    def serialize(self):
        import json
        return json.dumps(self.obj, indent=2)


class YAMLSerializer(Serializer):
    def __init__(self, obj):
        self.obj = obj

    @classmethod
    def from_data(cls, data):
        return cls(data)

    def serialize(self):
        lines = []
        for k, v in self.obj.items():
            if isinstance(v, dict):
                lines.append(f"{k}:")
                for sk, sv in v.items():
                    lines.append(f"  {sk}: {sv}")
            else:
                lines.append(f"{k}: {v}")
        return "\n".join(lines)


class SerializerFactory:
    _serializers: Dict[str, Type[Serializer]] = {}

    @classmethod
    def register(cls, name: str, serializer_class: Type[Serializer]):
        cls._serializers[name] = serializer_class

    @classmethod
    def create(cls, format: str, data: dict) -> Serializer:
        serializer_class = cls._serializers.get(format)
        if not serializer_class:
            raise ValueError(f"Unknown format: {format}")
        return serializer_class.from_data(data)


SerializerFactory.register("json", JSONSerializer)
SerializerFactory.register("yaml", YAMLSerializer)

data = {"name": "Alice", "age": 30, "address": {"city": "NYC", "zip": "10001"}}
serializer = SerializerFactory.create("json", data)
print(serializer.serialize())
```

```python
# Factory with Lazy Initialization and Caching
import time
from typing import Type, Dict, Any


class Model(ABC):
    @abstractmethod
    def predict(self, input_data):
        pass


class LinearRegressionModel(Model):
    def __init__(self):
        time.sleep(0.5)  # Simulate loading
        self.coefficients = [0.5, -0.2, 1.0]

    def predict(self, input_data):
        return sum(c * x for c, x in zip(self.coefficients, input_data))


class RandomForestModel(Model):
    def __init__(self):
        time.sleep(1.0)  # Simulate loading
        self.trees = 100

    def predict(self, input_data):
        return sum(input_data) / len(input_data) + 0.5


class NeuralNetworkModel(Model):
    def __init__(self):
        time.sleep(2.0)  # Simulate loading
        self.layers = [64, 32, 1]

    def predict(self, input_data):
        return sum(input_data) * 1.2


class LazyModelFactory:
    _models: Dict[str, Type[Model]] = {
        "linear": LinearRegressionModel,
        "random_forest": RandomForestModel,
        "neural_network": NeuralNetworkModel,
    }
    _cache: Dict[str, Model] = {}

    @classmethod
    def get_model(cls, name: str) -> Model:
        if name not in cls._cache:
            model_class = cls._models.get(name)
            if not model_class:
                raise ValueError(f"Unknown model: {name}")
            print(f"Loading {name} model...")
            cls._cache[name] = model_class()
        return cls._cache[name]


start = time.time()
model1 = LazyModelFactory.get_model("linear")
model2 = LazyModelFactory.get_model("linear")
elapsed = time.time() - start
print(f"Time: {elapsed:.2f}s (second call should be instant)")
print(model1 is model2)  # True
```

```python
# Factory with function registry and singledispatch
from functools import singledispatch


@singledispatch
def create_formatter(data):
    raise TypeError(f"No formatter for {type(data)}")


@create_formatter.register(dict)
def _format_dict(data):
    return "\n".join(f"{k}: {v}" for k, v in data.items())


@create_formatter.register(list)
def _format_list(data):
    return "\n".join(f"- {item}" for item in data)


@create_formatter.register(str)
def _format_string(data):
    return data.upper()


@create_formatter.register(int)
def _format_int(data):
    return f"Number: {data}"


@create_formatter.register(type(None))
def _format_none(data):
    return "NULL"


print(create_formatter({"name": "Alice", "age": 30}))
print(create_formatter(["apple", "banana", "cherry"]))
print(create_formatter("hello"))
print(create_formatter(42))
print(create_formatter(None))
```

## Real-World Use Cases

```python
# Web framework HTTP handler factory
class HTTPHandler(ABC):
    @abstractmethod
    def handle(self, request):
        pass


class GetHandler(HTTPHandler):
    def handle(self, request):
        return f"GET {request['path']} — 200 OK"


class PostHandler(HTTPHandler):
    def handle(self, request):
        return f"POST {request['path']} — 201 Created"


class PutHandler(HTTPHandler):
    def handle(self, request):
        return f"PUT {request['path']} — 200 Updated"


class DeleteHandler(HTTPHandler):
    def handle(self, request):
        return f"DELETE {request['path']} — 204 No Content"


class Router:
    def __init__(self):
        self._routes = {}

    def route(self, method: str, path: str, handler_class: Type[HTTPHandler]):
        self._routes[(method.upper(), path)] = handler_class

    def dispatch(self, method: str, path: str):
        handler_class = self._routes.get((method.upper(), path))
        if not handler_class:
            return f"{method} {path} — 404 Not Found"
        handler = handler_class()
        return handler.handle({"method": method, "path": path})


router = Router()
router.route("GET", "/users", GetHandler)
router.route("POST", "/users", PostHandler)
router.route("GET", "/users/1", GetHandler)

print(router.dispatch("GET", "/users"))
print(router.dispatch("POST", "/users"))
print(router.dispatch("DELETE", "/users"))
```

```python
# Payment gateway factory
class PaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount: float, currency: str) -> dict:
        pass

    @abstractmethod
    def refund(self, transaction_id: str) -> dict:
        pass


class StripeGateway(PaymentGateway):
    def charge(self, amount, currency):
        return {"success": True, "provider": "stripe", "amount": amount, "currency": currency}

    def refund(self, transaction_id):
        return {"success": True, "provider": "stripe", "transaction": transaction_id}


class PayPalGateway(PaymentGateway):
    def charge(self, amount, currency):
        return {"success": True, "provider": "paypal", "amount": amount, "currency": currency}

    def refund(self, transaction_id):
        return {"success": True, "provider": "paypal", "transaction": transaction_id}


class SquareGateway(PaymentGateway):
    def charge(self, amount, currency):
        return {"success": True, "provider": "square", "amount": amount, "currency": currency}

    def refund(self, transaction_id):
        return {"success": True, "provider": "square", "transaction": transaction_id}


class PaymentFactory:
    _gateways = {
        "stripe": StripeGateway,
        "paypal": PayPalGateway,
        "square": SquareGateway,
    }

    @classmethod
    def create(cls, provider: str) -> PaymentGateway:
        gateway = cls._gateways.get(provider.lower())
        if not gateway:
            raise ValueError(f"Unknown payment provider: {provider}")
        return gateway()


def process_payment(provider: str, amount: float, currency: str = "USD"):
    gateway = PaymentFactory.create(provider)
    return gateway.charge(amount, currency)


print(process_payment("stripe", 49.99))
print(process_payment("paypal", 29.99, "EUR"))
```

## Common Mistakes

```python
# MISTAKE 1: Factory method that returns pre-created instances
class BadFactory:
    def create(self, type_):
        if type_ == "a":
            return A()  # OK
        elif type_ == "b":
            return B()  # OK
        return BadSharedInstance()  # Shared mutable state!

```

```python
# MISTAKE 2: Too many conditional branches in factory
def create(type_):
    if type_ == "a": return A()
    elif type_ == "b": return B()
    elif type_ == "c": return C()
    elif type_ == "d": return D()
    elif type_ == "e": return E()
    # Use a registry/dict instead!
```

```python
# MISTAKE 3: Factory returning None instead of a valid object
def create(type_):
    if type_ == "a":
        return A()
    # Returns None for unknown types — causes AttributeError downstream

```

```python
# MISTAKE 4: Violating LSP with incompatible subtypes
class Vehicle:
    def start_engine(self): pass

class Bicycle(Vehicle):
    def start_engine(self):  # Bicycle has no engine!
        raise NotImplementedError
```

## Best Practices

```python
# 1. Use registration pattern instead of if/elif chains
# 2. Return abstract types, not concrete implementations
# 3. Validate parameters early in factory methods
# 4. Use classmethod/staticmethod for factory methods
# 5. Consider functools.singledispatch for simple cases
# 6. Document the factory's interface clearly
# 7. Keep factory logic simple — no business logic
```

```python
# Best practice: Typed factory with TypeVar
from typing import TypeVar, Type

T = TypeVar("T", bound=Animal)

def create_animal(cls: Type[T], *args, **kwargs) -> T:
    return cls(*args, **kwargs)


dog = create_animal(Dog)
cat = create_animal(Cat)
```

## Interview Questions

```python
# Q1: Difference between Simple Factory, Factory Method, and Abstract Factory?
# Simple Factory: single function/class creating objects
# Factory Method: subclasses decide which class to instantiate
# Abstract Factory: family of related product factories
```

```python
# Q2: How does the Factory pattern support the Open/Closed Principle?
# New product types can be added without modifying existing
# client code — just register a new factory class
```

```python
# Q3: When would you use Abstract Factory vs Factory Method?
# Abstract Factory: creating families of related products
# Factory Method: single product, subclasses decide type
```

```python
# Q4: How to implement factory in Python without switch/case?
# Use dict-based registry, or __init_subclass__ registration
```

## Coding Challenges

```python
# Challenge 1: Implement a factory for creating different
# types of parsers (JSON, XML, CSV, YAML) with registration
```

```python
# Challenge 2: Build an Abstract Factory for UI components
# (light theme vs dark theme) with buttons, inputs, cards
```

```python
# Challenge 3: Create a plugin system using factory + registration
# where plugins auto-register via __init_subclass__
```

```python
# Challenge 4: Implement a factory that uses lazy loading
# and caches created instances by type
```

```python
# Challenge 5: Build a dependency injection container
# that uses factory methods for service resolution
```

## Summary

The Factory pattern family simplifies object creation by centralizing instantiation logic. Simple Factory uses a single function or class. Factory Method delegates creation to subclasses. Abstract Factory composes factories for product families. Python's dynamic features enable elegant implementations using registration dicts, classmethod factories, singledispatch, and __init_subclass__ hooks. These patterns improve code maintainability, support the Open/Closed Principle, and make systems more extensible.

## Related Topics

- Singleton Pattern: Factories can create singleton-scoped objects
- Strategy Pattern: Similar structure but different intent
- Dependency Injection: Often uses factory patterns
- Builder Pattern: Alternative for complex object construction
- Abstract Base Classes: Used to define factory interfaces
- Plugin Architecture: Frequently uses factory + registration
