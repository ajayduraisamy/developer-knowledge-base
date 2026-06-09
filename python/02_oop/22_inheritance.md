# Inheritance - class Child(Parent), super(), MRO, overriding

## Introduction

Inheritance is a core OOP principle that allows a class to derive properties and behavior from another class. In Python, inheritance enables code reuse, establishes hierarchical relationships, and supports polymorphic behavior. Python's inheritance model is particularly flexible, supporting multiple inheritance where a class can inherit from several parent classes simultaneously.

## class Child(Parent)

### What It Is

The `class Child(Parent)` syntax defines a subclass (child class) that inherits attributes and methods from a base class (parent class). The child can extend or override the behavior inherited from the parent.

```python
class Animal:
    def speak(self):
        return "..."

class Dog(Animal):
    def speak(self):
        return "Woof!"
```

### Why It Is Important

Inheritance promotes code reuse by allowing shared logic to live in a base class while subclasses specialize behavior. It models "is-a" relationships (e.g., a Dog is an Animal), making class hierarchies intuitive and maintainable.

### How It Works Internally

When Python encounters a class definition with a parent, it stores the parent reference in `__bases__`. Attribute lookup traverses the Method Resolution Order (MRO) — a linearized list of ancestor classes — searching each class's `__dict__` until the attribute is found.

```python
class A:
    x = 1

class B(A):
    pass

b = B()
print(b.x)  # 1 — found in A's __dict__
```

### Syntax

```python
class Parent:
    pass

class Child(Parent):         # Single inheritance
    pass

class GrandChild(Child):     # Multi-level inheritance
    pass

class Mixin:
    pass

class ChildMultiple(Parent, Mixin):  # Multiple inheritance
    pass
```

### Beginner Examples

```python
class Vehicle:
    def __init__(self, brand):
        self.brand = brand

    def start(self):
        return f"{self.brand} engine started"

class Car(Vehicle):
    def __init__(self, brand, doors):
        super().__init__(brand)
        self.doors = doors

    def honk(self):
        return "Beep beep!"

car = Car("Toyota", 4)
print(car.start())  # Toyota engine started
print(car.honk())   # Beep beep!
```

### Intermediate Examples

```python
class Logger:
    def log(self, message, level="INFO"):
        print(f"[{level}] {message}")

class FileLogger(Logger):
    def __init__(self, filename):
        self.filename = filename

    def log(self, message, level="INFO"):
        with open(self.filename, "a") as f:
            f.write(f"[{level}] {message}\n")

class ConsoleAndFileLogger(FileLogger):
    def log(self, message, level="INFO"):
        super().log(message, level)
        Logger.log(self, message, level)

logger = ConsoleAndFileLogger("app.log")
logger.log("Application started")
```

### Advanced Examples

```python
# Multiple inheritance with abstract base classes
from abc import ABC, abstractmethod

class JSONSerializable(ABC):
    @abstractmethod
    def to_json(self) -> dict:
        pass

class DatabaseModel(ABC):
    @abstractmethod
    def save(self):
        pass

class User(JSONSerializable, DatabaseModel):
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def to_json(self) -> dict:
        return {"name": self.name, "email": self.email}

    def save(self):
        print(f"Saving {self.name} to database")

# Diamond problem resolution
class A:
    def method(self):
        return "A"

class B(A):
    def method(self):
        return "B"

class C(A):
    def method(self):
        return "C"

class D(B, C):
    pass

d = D()
print(d.method())  # "B" — follows MRO: D → B → C → A
print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

### Real-World Use Cases

- **Django ORM**: model classes inheriting from `django.db.models.Model`
- **Exception hierarchies**: custom exceptions inherit from `Exception`
- **Test frameworks**: test cases inherit from `unittest.TestCase`
- **GUI frameworks**: custom widgets inherit from base widget classes
- **Middleware stacks**: each middleware inherits from a base middleware class

### Common Mistakes

1. **Circular inheritance**: `class A(B)` and `class B(A)` raises `TypeError`
2. **Forgetting `super().__init__()`**: child's `__init__` must explicitly initialize parent
3. **Overusing inheritance**: composition is often more appropriate than deep hierarchies
4. **Diamond problem confusion**: Python's C3 linearization resolves this predictably

### Best Practices

- Favor composition over inheritance when the relationship is "has-a" not "is-a"
- Keep inheritance hierarchies shallow (max 2-3 levels)
- Use mixin classes for cross-cutting concerns
- Always call `super().__init__()` in child `__init__`
- Use abstract base classes to enforce interface contracts

### Performance Considerations

- Deep inheritance slows attribute lookup (MRO traversal)
- `super()` has overhead; cache parent references in hot paths
- Method resolution uses C3 linearization, computed once at class definition
- Property lookups in deep hierarchies add measurable overhead

## super() function

### What It Is

`super()` returns a proxy object that delegates method calls to the next class in the MRO, enabling cooperative multiple inheritance.

### Why It Is Important

Without `super()`, you'd need to hardcode parent class names, which breaks in multiple inheritance scenarios. `super()` ensures each parent in the MRO is called exactly once and in the correct order.

### How It Works Internally

`super()` uses the MRO of the instance's class to determine the next class. `super(C, self).method()` finds C in the MRO, takes the next class, and calls `method` on it.

```python
class A:
    def __init__(self):
        print("A.__init__")

class B(A):
    def __init__(self):
        print("B.__init__")
        super().__init__()

class C(B):
    def __init__(self):
        print("C.__init__")
        super().__init__()

c = C()
# C.__init__
# B.__init__
# A.__init__
```

### Syntax

```python
super().__init__(args)          # Inside instance method (Python 3+)
super(ClassName, self).method() # Explicit form (Python 2 style)
super().method()                # Preferred implicit form
```

### Beginner Examples

```python
class Parent:
    def __init__(self, name):
        self.name = name

class Child(Parent):
    def __init__(self, name, age):
        super().__init__(name)
        self.age = age

child = Child("Alice", 10)
print(child.name, child.age)
```

### Advanced Examples

```python
class PluginBase:
    def process(self, data):
        return data

class LoggingPlugin(PluginBase):
    def process(self, data):
        print(f"Processing: {data}")
        return super().process(data)

class ValidationPlugin(PluginBase):
    def process(self, data):
        if not data:
            raise ValueError("Empty data")
        return super().process(data)

class EncryptionPlugin(PluginBase):
    def process(self, data):
        return f"encrypted({super().process(data)})"

class Pipeline(LoggingPlugin, ValidationPlugin, EncryptionPlugin):
    pass

p = Pipeline()
result = p.process("hello")
print(result)  # encrypted(hello)
```

## MRO algorithm

### What It Is

The Method Resolution Order (MRO) is the order in which Python searches for attributes and methods in an inheritance hierarchy. Python uses the C3 linearization algorithm to compute a consistent, monotonic MRO.

### Why It Is Important

MRO determines which method gets called when multiple parent classes define the same method. Understanding MRO is essential for debugging complex inheritance hierarchies, especially with multiple inheritance.

### How It Works Internally

C3 linearization merges the MROs of all parent classes, ensuring:
1. Children come before parents
2. Base classes maintain their relative order
3. Monotonicity: if class X appears before Y in one MRO, it does so in all

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass

print(D.__mro__)
# D → B → C → A → object
```

### Syntax

```python
ClassName.__mro__      # Tuple of classes in MRO order
ClassName.mro()        # Same as list
help(ClassName)        # Shows MRO in documentation
```

### Beginner Examples

```python
class Engine:
    def start(self):
        return "Engine started"

class ElectricMotor:
    def start(self):
        return "Motor hummed silently"

class HybridCar(Engine, ElectricMotor):
    pass

car = HybridCar()
print(car.start())  # Engine started (Engine comes first in MRO)
print(HybridCar.__mro__)
```

### Advanced Examples

```python
class A:
    def m(self):
        return "A"

class B(A):
    def m(self):
        return f"B({super().m()})"

class C(A):
    def m(self):
        return f"C({super().m()})"

class D(B, C):
    def m(self):
        return f"D({super().m()})"

d = D()
print(d.m())         # D(B(C(A)))
print(D.__mro__)     # D → B → C → A → object
```

## Method overriding

### What It Is

Method overriding occurs when a subclass defines a method with the same name as a method in its parent class, replacing or extending the parent's implementation.

### Why It Is Important

Overriding enables polymorphic behavior — different subclasses can respond to the same method call differently. This is the foundation of many design patterns (Template Method, Strategy, etc.).

### How It Works Internally

When Python looks up an attribute on an instance, it searches the MRO. The first class in the MRO that defines the method "wins." The overridden parent method can still be accessed via `super()`.

```python
class Base:
    def greet(self):
        return "Hello"

class Child(Base):
    def greet(self):
        return "Hi"

Child().greet()  # "Hi" (Child's version wins)
```

### Syntax

```python
class Parent:
    def method(self):
        return "parent"

class Child(Parent):
    def method(self):
        parent_result = super().method()
        return f"child({parent_result})"
```

### Beginner Examples

```python
class Shape:
    def area(self):
        return 0

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14159 * self.radius ** 2

class Square(Shape):
    def __init__(self, side):
        self.side = side

    def area(self):
        return self.side ** 2

shapes = [Circle(5), Square(4)]
for shape in shapes:
    print(shape.area())  # 78.53975, 16
```

### Advanced Examples

```python
# Template Method pattern
class DataProcessor:
    def process(self, data):
        data = self._clean(data)
        data = self._transform(data)
        return self._save(data)

    def _clean(self, data):
        return data.strip()

    def _transform(self, data):
        return data.upper()

    def _save(self, data):
        raise NotImplementedError

class FileProcessor(DataProcessor):
    def __init__(self, filename):
        self.filename = filename

    def _save(self, data):
        with open(self.filename, "w") as f:
            f.write(data)
        return f"Saved to {self.filename}"

class DatabaseProcessor(DataProcessor):
    def _save(self, data):
        return f"INSERT INTO table VALUES ('{data}')"

class CustomProcessor(DataProcessor):
    def _clean(self, data):
        # Override just the cleaning step
        return super()._clean(data.replace("\n", " "))

    def _transform(self, data):
        # Override to add special formatting
        return f"[{super()._transform(data)}]"
```

### Real-World Use Cases

- **Django**: overriding `save()`, `delete()`, `clean()` on models
- **unittest**: overriding `setUp()`, `tearDown()` for test fixtures
- **Custom exceptions**: overriding `__str__` for error formatting
- **Serialization**: overriding `to_dict()`, `from_dict()` in data models

### Common Mistakes

1. **Signature mismatch**: overriding but changing parameters breaks polymorphism
2. **Forgetting `super()`**: completely replacing instead of extending behavior
3. **Accidental override**: adding a method that inadvertently shadows a parent method
4. **Inconsistent return types**: violating Liskov Substitution Principle

### Best Practices

- Always call `super().method()` when extending behavior unless intentionally replacing
- Maintain consistent method signatures (Liskov Substitution)
- Use abstract base classes to define override contracts
- Document which methods are intended to be overridden

### Performance Considerations

- Overridden methods add one extra `super()` call at minimum
- Dynamic dispatch overhead is negligible for typical use
- Deep hierarchies with many overrides increase lookup time

### Interview Questions

1. What is the C3 linearization algorithm and why is it used?
2. How does `super()` work with multiple inheritance?
3. What is the diamond problem and how does Python resolve it?
4. How can you view the MRO of a class?
5. What happens if you don't call `super().__init__()`?

### Coding Challenges

1. Implement a class hierarchy for vehicles (Vehicle → Car, Truck, Motorcycle) with a `calculate_fuel_efficiency` method.
2. Create a logging system with multiple inheritance: `ConsoleLogger`, `FileLogger`, `TimestampMixin`.
3. Solve the "diamond problem" by designing a `Student`, `Employee`, `TeachingAssistant` hierarchy.

### Related Topics

Polymorphism, Abstract Base Classes, Mixins, Composition, Object class
