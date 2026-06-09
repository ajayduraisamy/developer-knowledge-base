# Classes and Objects - class, __init__, self, instance/class attributes

## Introduction

A **class** is a blueprint for creating objects. It defines a set of attributes (data) and methods (functions) that characterize any object instantiated from it. An **object** is an instance of a class — a concrete entity that contains actual values and behaves according to the class definition.

Python is an object-oriented language, meaning nearly everything you work with — integers, strings, lists, functions — is an object under the hood.

## Why It Is Important

- **Code reuse**: Classes let you define behaviour once and create many objects from the same blueprint.
- **Organization**: Group related data and functions together in a single unit.
- **Modelling real-world entities**: Represent things like `Car`, `User`, `Invoice` in a natural way.
- **Foundation for OOP**: Classes are the building block for inheritance, polymorphism, encapsulation, and abstraction.
- **Maintainability**: Changes to a class propagate to all instances, reducing duplication.

## Syntax

```python
class ClassName:
    """Optional docstring."""
    class_attribute = value          # shared by all instances

    def __init__(self, param1, param2):
        self.instance_attr1 = param1  # unique to each instance
        self.instance_attr2 = param2

    def method_name(self):
        # self refers to the current instance
        pass
```

Instantiating:

```python
obj = ClassName(arg1, arg2)
```

## Examples

### Minimal class

```python
class Dog:
    species = "Canis familiaris"

    def __init__(self, name, age):
        self.name = name
        self.age = age

    def bark(self):
        return f"{self.name} says woof!"


buddy = Dog("Buddy", 4)
print(buddy.bark())          # Buddy says woof!
print(buddy.species)         # Canis familiaris
print(buddy.age)             # 4
```

## Beginner Examples

### 1. Person class

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Hi, I'm {self.name} and I'm {self.age}."


p = Person("Alice", 30)
print(p.greet())
```

### 2. Counter with class attribute

```python
class Counter:
    total_count = 0

    def __init__(self):
        Counter.total_count += 1
        self.id = Counter.total_count


a = Counter()
b = Counter()
print(b.id)               # 2
print(Counter.total_count)  # 2
```

### 3. BankAccount

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        self.balance += amount
        return self.balance

    def withdraw(self, amount):
        if amount > self.balance:
            return "Insufficient funds"
        self.balance -= amount
        return self.balance


acct = BankAccount("Bob", 100)
print(acct.deposit(50))   # 150
print(acct.withdraw(200)) # Insufficient funds
```

## Intermediate Examples

### 1. Class and instance attribute interaction

```python
class Employee:
    raise_factor = 1.04
    emp_count = 0

    def __init__(self, name, salary):
        self.name = name
        self.salary = salary
        Employee.emp_count += 1

    def apply_raise(self):
        self.salary = int(self.salary * Employee.raise_factor)

    @classmethod
    def from_string(cls, data):
        name, salary = data.split("-")
        return cls(name, int(salary))


e1 = Employee("Alice", 50000)
e2 = Employee.from_string("Bob-60000")
e1.apply_raise()
print(e1.salary)      # 52000
print(e2.salary)      # 60000
print(Employee.emp_count)  # 2
```

### 2. Private name mangling convention

```python
class Student:
    def __init__(self, name, grade):
        self.name = name
        self._grade = grade          # "protected" convention
        self.__id = hash(name)       # name-mangled to _Student__id

    def get_id(self):
        return self.__id


s = Student("Eve", 85)
print(s._grade)        # works but discouraged
print(s.get_id())      # correct usage
# print(s.__id)        # AttributeError
print(s._Student__id)  # name mangling workaround
```

### 3. `__init__` with defaults and type hints

```python
class Point:
    def __init__(self, x: float = 0.0, y: float = 0.0) -> None:
        self.x = x
        self.y = y

    def distance_from_origin(self) -> float:
        return (self.x ** 2 + self.y ** 2) ** 0.5

    def __repr__(self) -> str:
        return f"Point({self.x}, {self.y})"


p = Point(3, 4)
print(p)                    # Point(3, 4)
print(p.distance_from_origin())  # 5.0
```

## Advanced Examples

### 1. Data descriptors with `__set_name__`

```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self.name = f"_{name}"

    def __get__(self, obj, objtype=None):
        return getattr(obj, self.name, 0)

    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError("Must be positive")
        setattr(obj, self.name, value)


class Order:
    quantity = PositiveNumber()
    price = PositiveNumber()

    def __init__(self, quantity, price):
        self.quantity = quantity
        self.price = price

    def total(self):
        return self.quantity * self.price


o = Order(5, 10.0)
print(o.total())  # 50.0
# o.quantity = -1  # ValueError
```

### 2. Slots for memory optimisation

```python
class Point2D:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Point2D({self.x}, {self.y})"


p = Point2D(10, 20)
print(p)          # Point2D(10, 20)
# p.z = 30        # AttributeError: 'Point2D' object has no attribute 'z'
```

### 3. Metaclass-powered class factory

```python
class AutoStr:
    """Metaclass that auto-generates __str__ from attributes."""

    def __new__(cls, name, bases, namespace):
        if "__str__" not in namespace and "_fields" in namespace:
            fields = namespace["_fields"]

            def __str__(self):
                parts = [f"{f}={getattr(self, f)!r}" for f in fields]
                return f"{name}({', '.join(parts)})"

            namespace["__str__"] = __str__
        return super().__new__(cls, name, bases, namespace)


class Product(metaclass=AutoStr):
    _fields = ("name", "price")

    def __init__(self, name, price):
        self.name = name
        self.price = price


p = Product("Widget", 9.99)
print(p)  # Product(name='Widget', price=9.99)
```

## Real-World Use Cases

- **Web frameworks**: Django models are classes mapping to database tables.
- **GUI applications**: Each window, button, or widget is an object from a class.
- **Game development**: Entities such as `Player`, `Enemy`, `Bullet` are classes with shared and unique state.
- **Configuration handlers**: Classes encapsulate loading, validation, and access to settings.
- **API clients**: A `Client` class holds authentication tokens, base URLs, and HTTP methods.

## Common Mistakes

| Mistake | Why it's wrong | Fix |
|---|---|---|
| Forgetting `self` as first parameter | Python will silently treat the method as a regular function | Always include `self` (or `cls`) |
| Mutating class attribute via instance | Creates a new instance attribute instead of updating the class attribute | Use `ClassName.attr = value` |
| Using mutable default arguments in `__init__` | Default is shared across all instances | Use `None` and create a new mutable inside |
| Naming instance and local variables the same | Shadows the instance attribute | Use different names or prefix with `self.` |
| Forgetting to call `super().__init__()` | Parent initialisation is skipped | Call `super().__init__(...)` when inheriting |

```python
# Mutable default — BAD
class Bad:
    def __init__(self, items=[]):
        self.items = items

a = Bad()
b = Bad()
a.items.append(1)
print(b.items)  # [1]  — oops!


# Mutable default — GOOD
class Good:
    def __init__(self, items=None):
        self.items = items if items is not None else []

a = Good()
b = Good()
a.items.append(1)
print(b.items)  # []
```

## Best Practices

- Use **PascalCase** for class names.
- Keep `__init__` simple — only assign attributes, avoid heavy logic.
- Prefer **instance attributes** for per-object data; use **class attributes** for constants or shared state.
- Type-annotate `__init__` parameters and return `None`.
- Document the class with a docstring explaining its purpose.
- Use properties (`@property`) instead of bare getter/setter methods.
- Keep classes **small** and **single-responsibility** — if a class does too much, split it.
- Favour composition over inheritance.

## Interview Questions

1. **What is the difference between a class attribute and an instance attribute?**
   *Class attributes are shared by all instances; instance attributes belong to a specific object.*

2. **Explain the role of `self` in Python classes.**
   *`self` is the reference to the current instance; it is passed implicitly when calling a method on an object.*

3. **How does Python handle attribute lookup?**
   *It follows the C3 linearization: instance → class → parent classes → `__getattr__` → `AttributeError`.*

4. **What is the purpose of `__init__`? Is it a constructor?**
   *It is an initialiser, not a constructor (`__new__` creates the object). `__init__` sets up initial state.*

5. **Can you change a class attribute after instantiation?**
   *Yes, via `ClassName.attr = new_value`. Changing it via an instance creates a new instance attribute that shadows the class one.*

6. **What are `__slots__` and when should you use them?**
   *`__slots__` restricts which attributes an instance can have and saves memory by eliminating `__dict__`. Use in memory-sensitive applications with many instances.*

## Coding Challenges

1. **Library System**: Create a `Book` class with `title`, `author`, `isbn`. Add a `Library` class that can add, remove, and search books.

2. **Banking System**: Implement `Account` (base), `SavingsAccount` (with interest), `CheckingAccount` (with overdraft). Track all transactions.

3. **Vector Class**: Build a 2D `Vector` class with `__add__`, `__sub__`, `__mul__` (dot product), `__abs__` (magnitude), and `__repr__`.

4. **Task Tracker**: Create a `Task` class with status, priority, due date. Build a `Project` class that manages a collection of tasks and can filter by status/priority.

5. **Event System**: Implement `Event` and `EventEmitter` classes. The emitter stores listeners and notifies them when the event fires.

## Summary

- A **class** is a blueprint; an **object** is an instance of that blueprint.
- `__init__` initialises instance attributes on creation.
- `self` always refers to the current instance.
- **Class attributes** are shared; **instance attributes** are per-object.
- Attribute lookup follows MRO: instance → class → parents.
- Use `__slots__` to save memory for large numbers of objects.

## Related Topics

- Inheritance — reusing and extending class behaviour
- Polymorphism — same interface, different implementations
- Encapsulation — hiding internal state
- Abstraction — defining interfaces via ABCs
- Magic methods — customising object behaviour
- Properties — computed attributes with getter/setter logic
- Static and class methods — methods that don't operate on instances
