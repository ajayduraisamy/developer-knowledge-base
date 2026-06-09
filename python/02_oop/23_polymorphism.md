# Polymorphism - Duck typing, method overriding, operator overloading

## Introduction

**Polymorphism** (Greek: "many forms") is the ability of different object types to respond to the same interface or method call in their own way. In Python, polymorphism is primarily achieved through **duck typing** — "If it walks like a duck and quacks like a duck, it's a duck" — meaning the object's behaviour matters, not its explicit type.

Python supports several forms: duck typing, method overriding, operator overloading, and abstract base classes.

## Why It Is Important

- **Flexibility**: Write code that works with any object providing the expected methods.
- **Extensibility**: Add new types without modifying existing code (open/closed principle).
- **Reusability**: Generic functions and algorithms work across unrelated classes.
- **Testability**: Easily swap real implementations with mocks or stubs.
- **Reduced coupling**: Code depends on behaviour (interfaces), not concrete types.

## Syntax

```python
# Duck typing — no explicit interface needed
def make_sound(animal):
    return animal.speak()

# Method overriding
class Base:
    def method(self):
        return "base"

class Child(Base):
    def method(self):
        return "child"

# Operator overloading
class Vector:
    def __add__(self, other):
        ...

# Abstract base class
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        ...
```

## Examples

### Duck typing in action

```python
class Duck:
    def speak(self):
        return "Quack!"

class Person:
    def speak(self):
        return "Hello!"

def make_sound(entity):
    print(entity.speak())

make_sound(Duck())    # Quack!
make_sound(Person())  # Hello!
```

## Beginner Examples

### 1. len() polymorphism

```python
print(len("hello"))     # 5  (str)
print(len([1, 2, 3]))   # 3  (list)
print(len({"a": 1}))    # 1  (dict)
print(len(range(10)))   # 10 (range)
```

### 2. Method overriding in inheritance

```python
class Animal:
    def move(self):
        return "Moving"

class Fish(Animal):
    def move(self):
        return "Swimming"

class Bird(Animal):
    def move(self):
        return "Flying"

class Snake(Animal):
    pass  # uses parent move()

animals = [Fish(), Bird(), Snake()]
for a in animals:
    print(a.move())
# Swimming
# Flying
# Moving
```

### 3. Iteration protocol

```python
class Countdown:
    def __init__(self, start):
        self.start = start

    def __iter__(self):
        self.n = self.start
        return self

    def __next__(self):
        if self.n < 0:
            raise StopIteration
        val = self.n
        self.n -= 1
        return val


for x in Countdown(3):
    print(x, end=" ")  # 3 2 1 0
```

## Intermediate Examples

### 1. Operator overloading

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Point(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        return Point(self.x - other.x, self.y - other.y)

    def __mul__(self, scalar):
        return Point(self.x * scalar, self.y * scalar)

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __repr__(self):
        return f"Point({self.x}, {self.y})"


p1 = Point(1, 2)
p2 = Point(3, 4)
print(p1 + p2)    # Point(4, 6)
print(p2 - p1)    # Point(2, 2)
print(p1 * 3)     # Point(3, 6)
print(p1 == Point(1, 2))  # True
```

### 2. Abstract base class with registered subclasses

```python
from abc import ABC, abstractmethod


class Printable(ABC):
    @abstractmethod
    def format(self) -> str:
        ...


class Document(Printable):
    def __init__(self, content):
        self.content = content

    def format(self):
        return f"Document: {self.content}"


# Registering an existing class without modifying it
@Printable.register
class LegacyReport:
    def format(self):
        return "Legacy Report"


print(isinstance(LegacyReport(), Printable))  # True
```

### 3. Polymorphic functions with protocol hints

```python
from typing import Protocol


class Speakable(Protocol):
    def speak(self) -> str:
        ...


def greet(entity: Speakable) -> None:
    print(f"Greetings: {entity.speak()}")


class Robot:
    def speak(self) -> str:
        return "Beep boop"


class Human:
    def speak(self) -> str:
        return "Hello there"


greet(Robot())   # Greetings: Beep boop
greet(Human())   # Greetings: Hello there
```

## Advanced Examples

### 1. Generic visitor pattern with singledispatch

```python
from functools import singledispatch


@singledispatch
def render(obj):
    raise TypeError(f"No renderer for {type(obj)}")


@render.register(str)
def _(obj):
    return f"String: {obj}"


@render.register(int)
def _(obj):
    return f"Integer: {obj * 2}"


@render.register(list)
def _(obj):
    return f"List: {', '.join(str(x) for x in obj)}"


@render.register(dict)
def _(obj):
    return f"Dict: {len(obj)} keys"


print(render("hello"))      # String: hello
print(render(21))           # Integer: 42
print(render([1, 2, 3]))    # List: 1, 2, 3
print(render({"a": 1}))     # Dict: 1 keys
```

### 2. Custom numeric type with full operator support

```python
class Complex:
    def __init__(self, real, imag):
        self.real = real
        self.imag = imag

    def __add__(self, other):
        if isinstance(other, (int, float)):
            return Complex(self.real + other, self.imag)
        return Complex(self.real + other.real, self.imag + other.imag)

    def __radd__(self, other):
        return self.__add__(other)

    def __neg__(self):
        return Complex(-self.real, -self.imag)

    def __abs__(self):
        return (self.real ** 2 + self.imag ** 2) ** 0.5

    def __eq__(self, other):
        if isinstance(other, Complex):
            return self.real == other.real and self.imag == other.imag
        return NotImplemented

    def __repr__(self):
        sign = "+" if self.imag >= 0 else "-"
        return f"{self.real} {sign} {abs(self.imag)}i"


c1 = Complex(3, 4)
c2 = Complex(1, -2)
print(c1 + c2)      # 4 + 2i
print(5 + c1)       # 8 + 4i
print(-c1)          # -3 - 4i
print(abs(c1))      # 5.0
print(c1 == Complex(3, 4))  # True
```

### 3. Pluggable backend system

```python
from abc import ABC, abstractmethod


class StorageBackend(ABC):
    @abstractmethod
    def save(self, key: str, data: bytes) -> None:
        ...

    @abstractmethod
    def load(self, key: str) -> bytes:
        ...


class DiskStorage(StorageBackend):
    def save(self, key, data):
        with open(f"{key}.bin", "wb") as f:
            f.write(data)
        print(f"Disk: saved {key}")

    def load(self, key):
        with open(f"{key}.bin", "rb") as f:
            return f.read()


class S3Storage(StorageBackend):
    def save(self, key, data):
        print(f"S3: saved {key} (simulated)")

    def load(self, key):
        print(f"S3: loaded {key} (simulated)")
        return b"data"


class StorageClient:
    def __init__(self, backend: StorageBackend):
        self._backend = backend

    def store(self, key, data):
        self._backend.save(key, data)

    def retrieve(self, key):
        return self._backend.load(key)


client = StorageClient(DiskStorage())
client.store("test", b"hello world")
```

## Real-World Use Cases

- **Plugin architectures**: Load different plugins that all follow the same interface.
- **ORM systems**: A `QuerySet` works the same way whether the backend is PostgreSQL, SQLite, or MySQL.
- **Middleware pipelines**: Each middleware implements the same `process_request` / `process_response` methods.
- **Serialization**: `json.dumps()`, `pickle.dumps()`, `yaml.dump()` all accept objects that follow the serialization protocol.
- **Strategy pattern**: Different algorithms (sorting, compression, encryption) are interchangeable.

## Common Mistakes

| Mistake | Why it's wrong | Fix |
|---|---|---|
| Checking `type()` instead of using duck typing | Breaks polymorphism and Liskov substitution | Use `hasattr()` or `try-except` or `Protocol` |
| Not returning `NotImplemented` in overloaded operators | Falls back to the wrong operation | Return `NotImplemented` to let Python try the reverse |
| Forgetting that `__radd__` exists | `5 + obj` fails | Implement `__radd__` for symmetric operators |
| Overriding without keeping the same signature | Violates Liskov substitution principle | Keep compatible method signatures |
| Using ABCs when duck typing would suffice | Unnecessary ceremony | Use ABCs only when you need to enforce an interface |

## Best Practices

- Prefer **duck typing** over explicit type checks — "ask for forgiveness, not permission."
- Use `@abstractmethod` when you need to **enforce** that subclasses implement a method.
- Use `typing.Protocol` for static type checking of duck-typed interfaces.
- Override operators only when the operation is **natural** for the class (e.g., `+` for vectors).
- Always return `NotImplemented` from operator overloads when the operation is not supported for the given type.
- Keep polymorphic interfaces **small** — 1–3 methods is ideal.

## Interview Questions

1. **What is duck typing in Python?**
   *An object's suitability is determined by the presence of methods/properties, not by its type. "If it walks like a duck, it's a duck."*

2. **How does Python achieve polymorphism without interfaces?**
   *Through duck typing and dynamic dispatch — any object that implements the expected methods can be used.*

3. **What is the difference between method overloading and method overriding?**
   *Overriding: a child class redefines a parent method (runtime polymorphism). Overloading: the same method name with different parameters; Python doesn't support compile-time overloading (use default args or `singledispatch`).*

4. **Explain the Liskov Substitution Principle.**
   *Subtypes must be substitutable for their base types without altering the correctness of the program.*

5. **How does `functools.singledispatch` work?**
   *It creates a generic function that dispatches on the type of the first argument, allowing polymorphic behaviour without classes.*

6. **What is the difference between `NotImplemented` and `NotImplementedError`?**
   *`NotImplemented` is a singleton returned by operator overloads to signal the operation isn't supported (triggers reverse op). `NotImplementedError` is an exception raised for abstract methods that must be overridden.*

## Coding Challenges

1. **Shape Area Calculator**: Create `Circle`, `Rectangle`, `Triangle` classes, each with an `area()` method. Write a function `total_area(shapes)` that sums areas polymorphically.

2. **Custom Container**: Implement a `SparseList` that stores only non-default values. Support `__getitem__`, `__setitem__`, `__len__`, `__iter__`, and `__add__`.

3. **Plugin System**: Define a `Plugin` ABC with `process(data)` method. Implement `UpperPlugin`, `ReversePlugin`, `CompressPlugin`. Create a `Pipeline` that runs all plugins.

4. **Polymorphic Serializer**: Build `JSONSerializer`, `XMLSerializer`, `YAMLSerializer`, all implementing `serialize(obj)` and `deserialize(data)`.

5. **Expression Evaluator**: Create `Number`, `Add`, `Subtract`, `Multiply` classes, each with an `eval()` method. Build an AST and evaluate it.

## Summary

- **Polymorphism** means "many forms" — the same interface works with different types.
- **Duck typing** is Python's native approach: if an object has the method, call it.
- **Method overriding** allows a child class to replace parent methods.
- **Operator overloading** customises how operators work with user-defined classes.
- **ABCs** and **Protocols** provide enforcement and static type checking.
- The **Liskov Substitution Principle** ensures polymorphic code remains correct.

## Related Topics

- Classes and Objects — the objects that participate in polymorphic behaviour
- Inheritance — method overriding is a key form of polymorphism
- Encapsulation — polymorphic methods often access internal state
- Abstraction — ABCs define polymorphic interfaces
- Magic Methods — operator overloading uses dunder methods
- Static and Class Methods — can also be overridden polymorphically
