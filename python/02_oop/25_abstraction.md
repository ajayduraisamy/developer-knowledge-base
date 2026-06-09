# Abstraction - ABC, @abstractmethod, interfaces

## Introduction

**Abstraction** is the concept of hiding complex implementation details and exposing only the essential features of an object or system. It allows you to work with high-level concepts without needing to understand every underlying detail.

In Python, abstraction is primarily achieved through **Abstract Base Classes (ABCs)** — classes that define a blueprint of methods that subclasses must implement. ABCs cannot be instantiated directly; they exist solely to define an interface.

## Why It Is Important

- **Reduced complexity**: Focus on *what* an object does, not *how* it does it.
- **Separation of concerns**: Interface (contract) is separate from implementation.
- **Code reliability**: Enforces that all subclasses implement required methods.
- **Team collaboration**: Multiple developers can implement different backends against the same interface.
- **Testability**: Real implementations can be swapped with mocks or stubs.

## Syntax

```python
from abc import ABC, abstractmethod

class MyInterface(ABC):
    @abstractmethod
    def required_method(self):
        """Must be implemented by subclasses."""
        pass

    def concrete_method(self):
        """Optional — already implemented."""
        return "default behaviour"

class Concrete(MyInterface):
    def required_method(self):
        return "my implementation"
```

## Examples

### Basic ABC enforcement

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

    @abstractmethod
    def perimeter(self):
        pass


class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14159 * self.radius ** 2

    def perimeter(self):
        return 2 * 3.14159 * self.radius


# shape = Shape()           # TypeError!
circle = Circle(5)
print(f"{circle.area():.2f}")  # 78.54
```

## Beginner Examples

### 1. Abstract animal

```python
from abc import ABC, abstractmethod


class Animal(ABC):
    @abstractmethod
    def sound(self):
        ...

    @abstractmethod
    def move(self):
        ...

    def description(self):
        return f"A {self.__class__.__name__}"


class Dog(Animal):
    def sound(self):
        return "Woof"

    def move(self):
        return "Running"


class Fish(Animal):
    def sound(self):
        return "Blub"

    def move(self):
        return "Swimming"


animals = [Dog(), Fish()]
for a in animals:
    print(f"{a.description()}: {a.sound()}, {a.move()}")
```

### 2. Abstract class with concrete methods

```python
from abc import ABC, abstractmethod


class Database(ABC):
    @abstractmethod
    def connect(self):
        ...

    @abstractmethod
    def disconnect(self):
        ...

    def execute(self, query):
        self.connect()
        result = self._run_query(query)
        self.disconnect()
        return result

    @abstractmethod
    def _run_query(self, query):
        ...


class SQLiteDB(Database):
    def connect(self):
        print("SQLite: connected")

    def disconnect(self):
        print("SQLite: disconnected")

    def _run_query(self, query):
        return f"SQLite result for: {query}"


db = SQLiteDB()
print(db.execute("SELECT 1"))
# SQLite: connected
# SQLite: disconnected
# SQLite result for: SELECT 1
```

### 3. Preventing instantiation of abstract class

```python
from abc import ABC, abstractmethod


class Logger(ABC):
    @abstractmethod
    def log(self, message):
        ...

    def info(self, message):
        self.log(f"INFO: {message}")

    def error(self, message):
        self.log(f"ERROR: {message}")


# logger = Logger()  # TypeError: Can't instantiate abstract class

class ConsoleLogger(Logger):
    def log(self, message):
        print(message)


logger = ConsoleLogger()
logger.info("System started")    # INFO: System started
logger.error("Disk full")       # ERROR: Disk full
```

## Intermediate Examples

### 1. Multiple abstract methods with different signatures

```python
from abc import ABC, abstractmethod


class PaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount: float, currency: str) -> dict:
        ...

    @abstractmethod
    def refund(self, transaction_id: str) -> bool:
        ...

    @abstractmethod
    def status(self, transaction_id: str) -> str:
        ...


class StripeGateway(PaymentGateway):
    def charge(self, amount, currency="USD"):
        return {"id": "txn_001", "amount": amount, "currency": currency, "status": "success"}

    def refund(self, transaction_id):
        print(f"Stripe: refunding {transaction_id}")
        return True

    def status(self, transaction_id):
        return "completed"


class PayPalGateway(PaymentGateway):
    def charge(self, amount, currency="USD"):
        return {"id": "PAY-123", "amount": amount, "currency": currency, "status": "pending"}

    def refund(self, transaction_id):
        print(f"PayPal: refunding {transaction_id}")
        return True

    def status(self, transaction_id):
        return "pending"


def process_payment(gateway: PaymentGateway, amount: float):
    result = gateway.charge(amount, "USD")
    print(f"Payment {result['id']}: {result['status']}")
    return result


process_payment(StripeGateway(), 100.0)
process_payment(PayPalGateway(), 200.0)
```

### 2. Abstract properties

```python
from abc import ABC, abstractmethod


class Employee(ABC):
    def __init__(self, name):
        self.name = name

    @property
    @abstractmethod
    def salary(self):
        ...

    @property
    @abstractmethod
    def role(self):
        ...


class FullTimeEmployee(Employee):
    @property
    def salary(self):
        return 50000

    @property
    def role(self):
        return "Full-Time"


class Contractor(Employee):
    @property
    def salary(self):
        return 80000

    @property
    def role(self):
        return "Contractor"


emps = [FullTimeEmployee("Alice"), Contractor("Bob")]
for e in emps:
    print(f"{e.name} ({e.role}): ${e.salary}")
```

### 3. ABC with classmethod and staticmethod

```python
from abc import ABC, abstractmethod


class Serializer(ABC):
    @classmethod
    @abstractmethod
    def from_string(cls, data: str):
        ...

    @staticmethod
    @abstractmethod
    def validate(data: str) -> bool:
        ...

    @abstractmethod
    def serialize(self) -> str:
        ...


class JSONSerializer(Serializer):
    import json

    @classmethod
    def from_string(cls, data):
        return cls(json.loads(data))

    @staticmethod
    def validate(data):
        try:
            json.loads(data)
            return True
        except ValueError:
            return False

    def serialize(self):
        return json.dumps(self.__dict__)


# Note: order of decorators matters — @abstractmethod must be innermost
```

## Advanced Examples

### 1. Virtual subclass registration

```python
from abc import ABC


class MySequence(ABC):
    @abstractmethod
    def __getitem__(self, index):
        ...

    @abstractmethod
    def __len__(self):
        ...


# Register an existing class as a virtual subclass
@MySequence.register
class MyList:
    def __init__(self, items):
        self._items = items

    def __getitem__(self, index):
        return self._items[index]

    def __len__(self):
        return len(self._items)


print(issubclass(MyList, MySequence))  # True
print(isinstance(MyList([1, 2, 3]), MySequence))  # True
```

### 2. Abstract context manager

```python
from abc import ABC, abstractmethod


class AbstractDatabase(ABC):
    @abstractmethod
    def __enter__(self):
        ...

    @abstractmethod
    def __exit__(self, exc_type, exc_val, exc_tb):
        ...

    @abstractmethod
    def query(self, sql):
        ...


class PostgresDB(AbstractDatabase):
    def __enter__(self):
        print("Postgres: connected")
        return self

    def __exit__(self, *args):
        print("Postgres: disconnected")

    def query(self, sql):
        return f"Result: {sql}"


with PostgresDB() as db:
    print(db.query("SELECT 1"))
# Postgres: connected
# Result: SELECT 1
# Postgres: disconnected
```

### 3. Factory using ABC registration

```python
from abc import ABC, abstractmethod


class Plugin(ABC):
    _registry = {}

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        name = kwargs.get("name", cls.__name__.lower())
        Plugin._registry[name] = cls

    @abstractmethod
    def process(self, data):
        ...

    @classmethod
    def create(cls, name, **kwargs):
        if name not in cls._registry:
            raise ValueError(f"Unknown plugin: {name}")
        return cls._registry[name](**kwargs)


class UpperPlugin(Plugin, name="upper"):
    def process(self, data):
        return data.upper()


class ReversePlugin(Plugin, name="reverse"):
    def process(self, data):
        return data[::-1]


plugin = Plugin.create("upper")
print(plugin.process("hello"))      # HELLO

plugin2 = Plugin.create("reverse")
print(plugin2.process("hello"))     # olleh
```

## Real-World Use Cases

- **Web framework routers**: Abstract `Router` class with `resolve(path)` — Flask, Django, FastAPI each implement it differently.
- **Database backends**: Abstract `Database` with `connect()`, `execute()`, `disconnect()` — SQLite, PostgreSQL, MySQL each have a concrete implementation.
- **Caching backends**: Abstract `Cache` with `get(key)`, `set(key, value, ttl)` — Redis, Memcached, and in-memory caches.
- **Notification systems**: Abstract `Notifier` with `send(message)` — EmailNotifier, SMSNotifier, PushNotifier.
- **File storage**: Abstract `Storage` with `save(path, data)`, `load(path)` — LocalStorage, S3Storage, GCSStorage.

## Common Mistakes

| Mistake | Why it's wrong | Fix |
|---|---|---|
| Forgetting `@abstractmethod` decorator | Subclass isn't forced to implement | Add `@abstractmethod` to all methods that must be overridden |
| Instantiating ABC | `TypeError` raised | Only instantiate concrete subclasses |
| Mixing `@abstractmethod` and `@staticmethod`/`@classmethod` wrong order | `@abstractmethod` must be innermost | `@classmethod` / `@staticmethod` goes outermost |
| Defining `@abstractmethod` but giving it a body | Body is never called directly | Body is callable via `super().method()`, keep it if useful |
| Creating deep ABC hierarchies | Overcomplicates the design | Prefer composition or flat ABC structures |
| Using ABC when duck typing is sufficient | Unnecessary ceremony | Use ABC only when you need to **enforce** an interface |

```python
# Wrong decorator order
class Bad(ABC):
    @abstractmethod
    @staticmethod       # Wrong!
    def method():
        ...

# Correct decorator order
class Good(ABC):
    @staticmethod
    @abstractmethod     # abstractmethod is innermost
    def method():
        ...
```

## Best Practices

- Use ABCs when you need to **guarantee** that subclasses implement certain methods.
- Keep ABCs **small** — 1–5 abstract methods is a good range.
- Provide **default implementations** in the ABC where it makes sense.
- Use `__init_subclass__` for automatic registration of subclasses.
- Document the **contract** clearly in the ABC's docstring and abstract method docstrings.
- Prefer **duck typing** + `Protocol` for static type checking; use ABCs for runtime enforcement.
- Name ABCs with an `Abstract` prefix or use the `ABC` suffix (e.g., `AbstractRepository`).

## Interview Questions

1. **What is the difference between an abstract class and an interface?**
   *In Python, ABCs can have both abstract and concrete methods; there is no separate interface keyword. An "interface" is an ABC with only abstract methods.*

2. **Can you instantiate an abstract class?**
   *No — Python raises `TypeError: Can't instantiate abstract class...` if any abstract methods remain unimplemented.*

3. **What is `abc.ABC` and how does it work?**
   *`ABC` is a helper class that inherits from `ABCMeta`. It sets `__metaclass__ = ABCMeta`, which tracks abstract methods and prevents instantiation until all are implemented.*

4. **Explain `__init_subclass__` vs `ABCMeta`.**
   *`__init_subclass__` is called when a subclass is created; it's useful for registration. `ABCMeta` enforces that all abstract methods are implemented before instantiation.*

5. **What are virtual subclasses?**
   *Virtual subclasses are classes registered with an ABC via `register()` that `issubclass` and `isinstance` recognise, even though they don't inherit from the ABC.*

6. **How does `@abstractmethod` interact with `@property`?**
   *`@property @abstractmethod` creates an abstract property that must be overridden as a property in subclasses.*

## Coding Challenges

1. **Transport System**: Define `Vehicle` ABC with `start()`, `stop()`, `fuel_type()`. Implement `Car`, `Bicycle`, `Plane`. The ABC should also have `description()` that returns the class name.

2. **Data Pipeline**: Create `Stage` ABC with `process(data)`. Implement `FilterStage`, `TransformStage`, `AggregateStage`. Build a `Pipeline` that chains multiple stages.

3. **Cache Interface**: Design a `Cache` ABC with `get(key)`, `set(key, value, ttl)`, `delete(key)`, `clear()`. Implement `InMemoryCache` and `FileCache`.

4. **Plugin Discovery**: Build an ABC that auto-registers subclasses. Subclasses must implement `run()`. Add a `list_plugins()` and `run_plugin(name)` class method.

5. **Validator Framework**: Create a `Validator` ABC with `validate(value)` returning `(bool, error_message)`. Implement `EmailValidator`, `PhoneValidator`, `AgeValidator`. Build a `CompositeValidator` that runs multiple validators.

## Summary

- **Abstraction** hides implementation details behind a clean interface.
- **ABCs** (from `abc` module) define required methods via `@abstractmethod`.
- ABCs **cannot be instantiated** — only concrete subclasses can.
- `register()` creates **virtual subclasses** without inheritance.
- `__init_subclass__` enables **auto-registration** of subclasses.
- Use ABCs for **enforcement**, `Protocol` for **static typing**, and duck typing for **flexibility**.

## Related Topics

- Classes and Objects — the building blocks of abstraction
- Inheritance — ABCs rely on the inheritance mechanism
- Polymorphism — ABCs define a polymorphic interface
- Encapsulation — abstraction hides internal details
- Magic Methods — abstract classes can declare abstract dunder methods
- Properties — abstract properties enforce a property interface
