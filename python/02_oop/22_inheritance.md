# Inheritance - class Child(Parent), super(), MRO, overriding

## Introduction

**Inheritance** allows a class (child) to derive attributes and methods from another class (parent). The child can reuse, extend, or override the parent's behaviour. This is a core OOP principle that promotes code reuse and establishes a natural hierarchy.

Python supports single inheritance (one parent), multiple inheritance (multiple parents), and multi-level inheritance (chain of parents).

## Why It Is Important

- **Code reuse**: Common logic lives in a base class; subclasses add only what's different.
- **Hierarchical modelling**: Represent "is-a" relationships naturally (e.g., `Dog` is an `Animal`).
- **Extensibility**: Add new functionality by subclassing without modifying existing code.
- **Polymorphism**: Work with many types through a single base-class interface.
- **Framework design**: Base classes define hooks; subclasses provide concrete implementations.

## Syntax

```python
class Parent:
    def method(self):
        ...

class Child(Parent):          # single inheritance
    def method(self):         # override
        super().method()      # call parent version
        ...

class Child(Mixin, Parent):   # multiple inheritance
    pass
```

Key functions:

```python
isinstance(obj, Class)        # True if obj is an instance of Class (or subclass)
issubclass(Sub, Base)         # True if Sub inherits from Base (directly or indirectly)
```

## Examples

### Basic single inheritance

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return "..."


class Dog(Animal):
    def speak(self):
        return f"{self.name} says Woof!"


class Cat(Animal):
    def speak(self):
        return f"{self.name} says Meow!"


animals = [Dog("Rex"), Cat("Luna")]
for a in animals:
    print(a.speak())
# Rex says Woof!
# Luna says Meow!
```

## Beginner Examples

### 1. Extending built-in types

```python
class UpperList(list):
    def append(self, item):
        if isinstance(item, str):
            super().append(item.upper())
        else:
            super().append(item)


ul = UpperList()
ul.append("hello")
ul.append("world")
ul.append(42)
print(ul)  # ['HELLO', 'WORLD', 42]
```

### 2. Multi-level inheritance

```python
class Vehicle:
    def __init__(self, brand):
        self.brand = brand

    def honk(self):
        return "Beep!"


class Car(Vehicle):
    def __init__(self, brand, model):
        super().__init__(brand)
        self.model = model


class ElectricCar(Car):
    def __init__(self, brand, model, battery_kwh):
        super().__init__(brand, model)
        self.battery_kwh = battery_kwh


tesla = ElectricCar("Tesla", "Model 3", 75)
print(tesla.honk())       # Beep!  (inherited all the way up)
print(tesla.brand)        # Tesla
print(tesla.battery_kwh)  # 75
```

### 3. Using `isinstance` and `issubclass`

```python
class Base:
    pass

class Derived(Base):
    pass

obj = Derived()
print(isinstance(obj, Derived))   # True
print(isinstance(obj, Base))      # True (subclass relation)
print(issubclass(Derived, Base))  # True
print(issubclass(Base, Derived))  # False
```

## Intermediate Examples

### 1. Multiple inheritance and MRO

```python
class Flyer:
    def fly(self):
        return "Flying high"

    def action(self):
        return "Flyer action"


class Swimmer:
    def swim(self):
        return "Swimming deep"

    def action(self):
        return "Swimmer action"


class Duck(Flyer, Swimmer):
    def action(self):
        return "Duck action"


d = Duck()
print(d.fly())                  # Flying high
print(d.swim())                 # Swimming deep
print(d.action())               # Duck action (owns override)
print(Duck.__mro__)
# (<class 'Duck'>, <class 'Flyer'>, <class 'Swimmer'>, <class 'object'>)
```

### 2. Calling parent methods with `super()`

```python
class Base:
    def __init__(self, value):
        self.value = value
        print(f"Base.__init__({value})")


class MixinA:
    def __init__(self, *args, **kwargs):
        print("MixinA.__init__")
        super().__init__(*args, **kwargs)


class MixinB:
    def __init__(self, *args, **kwargs):
        print("MixinB.__init__")
        super().__init__(*args, **kwargs)


class Concrete(MixinA, MixinB, Base):
    def __init__(self, value):
        print("Concrete.__init__")
        super().__init__(value)


obj = Concrete(42)
# Concrete.__init__
# MixinA.__init__
# MixinB.__init__
# Base.__init__(42)
# MRO: Concrete -> MixinA -> MixinB -> Base -> object
```

### 3. Method resolution in diamond inheritance

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


print(D().m())      # D(B(C(A)))
print(D.__mro__)
# D -> B -> C -> A -> object
```

## Advanced Examples

### 1. Abstract base class enforcement

```python
from abc import ABC, abstractmethod


class Shape(ABC):
    @abstractmethod
    def area(self):
        ...

    @abstractmethod
    def perimeter(self):
        ...


class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14159 * self.radius ** 2

    def perimeter(self):
        return 2 * 3.14159 * self.radius


# s = Shape()           # TypeError: Can't instantiate abstract class
c = Circle(5)
print(f"{c.area():.2f}")  # 78.54
```

### 2. Dynamic inheritance with `type()`

```python
def make_animal(sound):
    class Animal:
        def speak(self):
            return sound
    return Animal


Dog = make_animal("Woof")
Cat = make_animal("Meow")

d = Dog()
print(d.speak())  # Woof
```

### 3. Cooperative multiple inheritance with kwargs

```python
class Base:
    def __init__(self, **kwargs):
        self.value = kwargs.pop("value", None)
        super().__init__(**kwargs)


class LogMixin:
    def __init__(self, **kwargs):
        self.log = kwargs.pop("log", None)
        if self.log:
            print(f"Log: creating instance with {kwargs}")
        super().__init__(**kwargs)


class ValidatedMixin:
    def __init__(self, **kwargs):
        if "value" in kwargs and kwargs["value"] < 0:
            raise ValueError("Value must be non-negative")
        super().__init__(**kwargs)


class MyClass(LogMixin, ValidatedMixin, Base):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)


obj = MyClass(value=10, log=True)
print(obj.value)  # 10
```

## Real-World Use Cases

- **Django models**: `class Product(models.Model)` — ORM inheritance provides table generation, query sets, and relationship fields.
- **UI frameworks**: `class Button(Widget)` where `Widget` provides event handling, rendering, and layout.
- **Exception hierarchy**: `class ValidationError(ValueError)` — custom exceptions inherit from standard ones for proper `except` matching.
- **Middleware stacks**: Each middleware inherits a base and calls `super()` to form a processing pipeline.
- **Plugin systems**: Base class defines the plugin API; third-party code subclasses it.

## Common Mistakes

| Mistake | Why it's wrong | Fix |
|---|---|---|
| Forgetting `super().__init__()` | Parent's initialisation is skipped | Always call `super().__init__(...)` |
| Diamond problem without cooperative MRO | `super()` calls may skip classes or be duplicated | Use `**kwargs` pattern and always call `super()` |
| Deep inheritance chains | Hard to follow and maintain | Prefer composition over deep hierarchies |
| Overriding without calling parent | Parent behaviour is lost | Call `super().method()` if you need parent logic |
| Misunderstanding MRO in multiple inheritance | Wrong method is called | Check `ClassName.__mro__` |

## Best Practices

- Favour **composition** over inheritance where possible.
- Keep inheritance trees **shallow** — no more than 2–3 levels.
- Use `super()` for cooperative multiple inheritance with consistent **kwargs signatures.
- Always call `super().__init__()` in child classes unless you intentionally want to skip initialisation.
- Prefer **mixin classes** (single-purpose, no `__init__`) for sharing behaviour across unrelated hierarchies.
- Document the MRO expectations when using multiple inheritance.
- Use `isinstance()` sparingly — prefer polymorphism (duck typing).

## Interview Questions

1. **What is the difference between `super()` and calling the parent class directly?**
   *`super()` follows the MRO dynamically and supports cooperative multiple inheritance; direct parent calls are static and break MRO.*

2. **Explain Python's MRO (C3 linearization).**
   *MRO determines the order in which base classes are searched when resolving a method. It ensures that a class always appears before its parents and that the order is monotonic.*

3. **How does multiple inheritance work in Python?**
   *A class can inherit from multiple base classes. Method resolution follows the MRO, which is a linearisation respecting all inheritance graphs.*

4. **What is a mixin and when would you use one?**
   *A mixin is a small class intended to add a specific behaviour to other classes via multiple inheritance. It is not meant to stand alone.*

5. **Can you change the MRO at runtime?**
   *No, MRO is computed at class creation and stored in `__mro__`. You cannot modify it, but you can redefine the class.*

6. **What is the difference between `isinstance` and `issubclass`?**
   *`isinstance(obj, Cls)` checks if `obj` is an instance of `Cls` or a subclass thereof; `issubclass(Sub, Base)` checks if `Sub` is a subclass of `Base`.*

## Coding Challenges

1. **Employee Hierarchy**: Build `Employee` (base with `name`, `salary`), `Manager` (with `team` list), `Developer` (with `programming_languages`). Calculate total team salary.

2. **Library System with Inheritance**: `Item` (title, year), `Book(Item)` (author, pages), `DVD(Item)` (director, runtime). Implement a `search_by_year` method.

3. **Shape Hierarchy**: `Shape` → `Circle`, `Rectangle`, `Triangle`. Each must implement `area()` and `perimeter()`. Add a `ColoredShape` mixin.

4. **Logging Framework**: Create `Logger` base, `FileLogger`, `ConsoleLogger`, `CompositeLogger` (sends to multiple loggers). Use MRO for formatting mixins.

5. **Virtual File System**: `Entry` (name), `File(Entry)` (content), `Directory(Entry)` (children). Implement recursive size calculation and tree display.

## Summary

- Inheritance models an **"is-a"** relationship between classes.
- `super()` delegates to the next class in the **MRO**.
- **Single inheritance** is straightforward; **multiple inheritance** requires understanding C3 linearization.
- `isinstance` and `issubclass` introspect the inheritance hierarchy.
- Prefer **composition** over deep inheritance trees.
- Use **mixins** to share small, focused behaviours across unrelated classes.

## Related Topics

- Classes and Objects — the foundation for inheritance
- Polymorphism — inheriting to override and extend behaviour
- Encapsulation — controlling access to inherited attributes
- Abstraction — using ABCs to define required interfaces for subclasses
- Magic Methods — overriding dunder methods inherited from `object`
- Properties — computed attribute access in class hierarchies
