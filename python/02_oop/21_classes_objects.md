# Classes and Objects - class, __init__, self, instance/class attributes

## Introduction

Object-Oriented Programming (OOP) is a paradigm that organizes code around objects rather than functions. In Python, everything is an object, and classes are the blueprints for creating objects. Understanding classes and objects is foundational to mastering Python, as they enable encapsulation of data and behavior into reusable, modular components. Python's class system is flexible and powerful, supporting multiple inheritance, metaclasses, and dynamic attribute resolution, yet it remains approachable due to its clean syntax.

## class keyword

### What It Is

The `class` keyword in Python is used to define a new class, which serves as a blueprint for creating objects. A class bundles data (attributes) and functions (methods) that operate on that data into a single logical unit.

```python
class Dog:
    pass
```

### Why It Is Important

The `class` keyword is the fundamental building block of OOP in Python. It allows you to create custom data types that model real-world entities, making code more intuitive, reusable, and organized. Without classes, complex programs would rely solely on primitive data types and standalone functions, leading to scattered, hard-to-maintain code.

### How It Works Internally

When Python encounters a `class` statement, it executes the class body in a new namespace, collects all variable and function definitions, and passes them to the metaclass (usually `type`) to construct the class object. The resulting class object is callable and serves as a factory for instances.

```python
# Internally, this:
class Dog:
    species = "Canis familiaris"
    def bark(self):
        return "Woof!"

# Is roughly equivalent to:
def bark_function(self):
    return "Woof!"
Dog = type("Dog", (), {"species": "Canis familiaris", "bark": bark_function})
```

### Syntax

```python
class ClassName(ParentClass):
    """Optional docstring"""
    class_variable = value

    def __init__(self, params):
        self.instance_variable = params

    def method(self, params):
        pass
```

- `ClassName`: follows standard naming conventions (PascalCase)
- `ParentClass`: optional inheritance specification
- `self`: mandatory first parameter of instance methods

### Beginner Examples

```python
class Car:
    wheels = 4

    def __init__(self, brand, model):
        self.brand = brand
        self.model = model

    def describe(self):
        return f"{self.brand} {self.model} with {self.wheels} wheels"

car1 = Car("Toyota", "Camry")
car2 = Car("Honda", "Civic")
print(car1.describe())  # Toyota Camry with 4 wheels
print(car2.describe())  # Honda Civic with 4 wheels
```

### Intermediate Examples

```python
class BankAccount:
    interest_rate = 0.02
    _total_accounts = 0

    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance
        self._transactions = []
        BankAccount._total_accounts += 1

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self.balance += amount
        self._transactions.append(f"Deposit: +{amount}")

    def withdraw(self, amount):
        if amount > self.balance:
            raise ValueError("Insufficient funds")
        self.balance -= amount
        self._transactions.append(f"Withdrawal: -{amount}")

    def apply_interest(self):
        interest = self.balance * BankAccount.interest_rate
        self.balance += interest
        self._transactions.append(f"Interest: +{interest:.2f}")

    @classmethod
    def total_accounts(cls):
        return cls._total_accounts

class SavingsAccount(BankAccount):
    interest_rate = 0.05

    def withdraw(self, amount):
        if self.balance - amount < 100:
            raise ValueError("Cannot drop below minimum balance of 100")
        super().withdraw(amount)

accounts = [
    BankAccount("Alice", 1000),
    SavingsAccount("Bob", 2000)
]
for acc in accounts:
    acc.apply_interest()
    print(f"{acc.owner}: ${acc.balance:.2f}")
# Alice: $1020.00
# Bob: $2100.00
```

### Advanced Examples

```python
from dataclasses import dataclass
from typing import ClassVar, List, Optional
import uuid

@dataclass
class Product:
    name: str
    price: float
    category: str
    tax_rate: ClassVar[float] = 0.08

    def __post_init__(self):
        self._id = uuid.uuid4()
        if self.price < 0:
            raise ValueError("Price cannot be negative")

    @property
    def total_price(self) -> float:
        return self.price * (1 + self.tax_rate)

    @classmethod
    def from_dict(cls, data: dict) -> "Product":
        return cls(**data)

    def __repr__(self) -> str:
        return f"Product({self.name}, ${self.price:.2f})"


class Inventory:
    def __init__(self):
        self._products: List[Product] = []

    def add_product(self, product: Product) -> None:
        self._products.append(product)

    def total_value(self) -> float:
        return sum(p.total_price for p in self._products)

    def filter_by_category(self, category: str) -> List[Product]:
        return [p for p in self._products if p.category == category]


# Usage with metaclass example
class SingletonMeta(type):
    _instances: dict = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Logger(metaclass=SingletonMeta):
    def log(self, msg: str) -> None:
        print(f"[LOG] {msg}")

logger1 = Logger()
logger2 = Logger()
assert logger1 is logger2  # True
```

### Real-World Use Cases

- **Web frameworks**: Django models (each model class maps to a database table)
- **GUI applications**: Tkinter/ PyQt widgets as class instances
- **Game development**: Player, Enemy, Item as classes with state and behavior
- **Data processing**: Pipeline stages represented as classes with transform methods
- **API clients**: Resource classes (User, Order, Product) mapping to REST endpoints

### Common Mistakes

1. **Forgetting `self` as first method parameter**
```python
class Wrong:
    def method():  # Missing self
        return "hi"
```

2. **Mutating shared class attributes via instance**
```python
class Team:
    members = []  # Shared mutable

team1 = Team()
team1.members.append("Alice")  # Affects all instances
```

3. **Misunderstanding attribute lookup order**: instance → class → parent classes

4. **Using mutable default arguments in `__init__`**
```python
def __init__(self, items=[]):  # Shared across all instances!
    self.items = items
```

### Best Practices

- Use PascalCase for class names
- Keep classes focused (Single Responsibility Principle)
- Document classes with docstrings
- Prefer `@dataclass` for simple data containers
- Use type hints to improve readability
- Avoid overly deep inheritance hierarchies

### Performance Considerations

- Attribute access follows the MRO and is cached in `__dict__` after first lookup
- `__slots__` can reduce memory usage for classes with many instances
- Method calls have overhead; for hot loops, consider extracting method logic inline
- Property access is slower than direct attribute access

### Interview Questions

1. What is the difference between a class and an instance?
2. How does Python's attribute lookup work?
3. What is `__slots__` and when should you use it?
4. Explain the relationship between `type` and metaclasses.
5. What happens when you call `ClassName()`?

### Coding Challenges

1. Implement a `Vector2D` class with `x`, `y` attributes and methods for addition, subtraction, and magnitude calculation.
2. Create a `Library` class that manages `Book` objects with checkout/return functionality.
3. Implement a `Task` class with a priority queue using the `heapq` pattern.

### Related Topics

Inheritance, Magic Methods, Properties, Static and Class Methods, Data Classes, Metaclasses
