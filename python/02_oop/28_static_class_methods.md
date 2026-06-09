# Static and Class Methods - @staticmethod, @classmethod

## Introduction

Python provides three types of methods within a class: **instance methods** (bound to an instance), **class methods** (bound to the class), and **static methods** (bound to neither). 

- **Instance methods** receive `self` and operate on a specific object.
- **Class methods** receive `cls` and operate on the class itself.
- **Static methods** receive no special first argument — they behave like plain functions namespaced inside the class.

The `@classmethod` and `@staticmethod` decorators make the distinction explicit.

## Why It Is Important

- **Factory methods**: Class methods are ideal for creating objects in alternative ways.
- **Namespace organisation**: Static methods group utility functions with the class they relate to.
- **Polymorphism with factories**: `cls()` in a class method respects inheritance and returns the correct subclass.
- **Code clarity**: Makes the intent clear — "this method belongs to the class concept, not to any single instance."
- **Abstract class methods**: Define interfaces that subclasses must implement.

## Syntax

```python
class MyClass:
    regular_attr = "class-level"

    def instance_method(self):
        """Receives the instance (self)."""
        return self.regular_attr

    @classmethod
    def class_method(cls):
        """Receives the class (cls), not the instance."""
        return cls.regular_attr

    @staticmethod
    def static_method(arg1, arg2):
        """Receives no special first argument."""
        return arg1 + arg2
```

## Examples

### Basic comparison

```python
class Demo:
    @classmethod
    def cm(cls):
        return f"Class method called on {cls.__name__}"

    @staticmethod
    def sm():
        return "Static method called"

    def im(self):
        return f"Instance method called on {self}"


print(Demo.cm())   # Class method called on Demo
print(Demo.sm())   # Static method called
# print(Demo.im()) # TypeError! (no instance)

d = Demo()
print(d.cm())      # Class method called on Demo  (can also be called on instance)
print(d.sm())      # Static method called
print(d.im())      # Instance method called on <__main__.Demo object at ...>
```

## Beginner Examples

### 1. Factory method with classmethod

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    @classmethod
    def from_birth_year(cls, name, birth_year):
        age = 2026 - birth_year
        return cls(name, age)

    @classmethod
    def from_string(cls, data):
        name, age = data.split(",")
        return cls(name.strip(), int(age.strip()))

    def __repr__(self):
        return f"Person({self.name}, {self.age})"


p1 = Person("Alice", 30)
p2 = Person.from_birth_year("Bob", 1990)
p3 = Person.from_string("Charlie, 25")

print(p1)  # Person(Alice, 30)
print(p2)  # Person(Bob, 36)
print(p3)  # Person(Charlie, 25)
```

### 2. Static method as a utility

```python
class MathUtils:
    @staticmethod
    def is_even(n):
        return n % 2 == 0

    @staticmethod
    def factorial(n):
        if n < 0:
            raise ValueError("Negative input")
        result = 1
        for i in range(2, n + 1):
            result *= i
        return result

    @staticmethod
    def gcd(a, b):
        while b:
            a, b = b, a % b
        return a


print(MathUtils.is_even(4))      # True
print(MathUtils.factorial(5))    # 120
print(MathUtils.gcd(12, 8))      # 4
```

### 3. Tracking instances with classmethod

```python
class Animal:
    instances = []

    def __init__(self, name):
        self.name = name
        Animal.instances.append(self)

    @classmethod
    def count(cls):
        return len(cls.instances)

    @classmethod
    def get_names(cls):
        return [a.name for a in cls.instances]


Animal("Rex")
Animal("Luna")
Animal("Max")
print(Animal.count())      # 3
print(Animal.get_names())  # ['Rex', 'Luna', 'Max']
```

## Intermediate Examples

### 1. Classmethod respects subclass (polymorphic factories)

```python
class Shape:
    def __init__(self, name):
        self.name = name

    @classmethod
    def create(cls, *args, **kwargs):
        """Factory method that creates instances of the subclass."""
        return cls(*args, **kwargs)

    def area(self):
        return 0


class Circle(Shape):
    def __init__(self, name, radius):
        super().__init__(name)
        self.radius = radius

    def area(self):
        return 3.14159 * self.radius ** 2


class Square(Shape):
    def __init__(self, name, side):
        super().__init__(name)
        self.side = side

    def area(self):
        return self.side ** 2


# Polymorphic factory — both call their own __init__
c = Circle.create("c1", 5)
s = Square.create("s1", 4)
print(c.area())   # 78.53975
print(s.area())   # 16
```

### 2. Static vs class method in inheritance

```python
class Base:
    @classmethod
    def cm(cls):
        return f"Base.cm called on {cls.__name__}"

    @staticmethod
    def sm():
        return "Base.sm"


class Child(Base):
    @classmethod
    def cm(cls):
        return f"Child.cm called on {cls.__name__}"

    @staticmethod
    def sm():
        return "Child.sm"


print(Base.cm())     # Base.cm called on Base
print(Child.cm())    # Child.cm called on Child
print(Base.sm())     # Base.sm
print(Child.sm())    # Child.sm — static method inherited but can be overridden
```

### 3. Configurable threshold with classmethod

```python
class Validator:
    min_length = 3
    max_length = 100

    def __init__(self, value):
        self.value = value

    @classmethod
    def set_length_range(cls, min_len, max_len):
        cls.min_length = min_len
        cls.max_length = max_len

    @classmethod
    def create_username(cls, value):
        if len(value) < cls.min_length:
            raise ValueError(f"Too short (min {cls.min_length})")
        if len(value) > cls.max_length:
            raise ValueError(f"Too long (max {cls.max_length})")
        return cls(value)

    def __repr__(self):
        return f"Validator({self.value})"


Validator.set_length_range(4, 20)
# v = Validator.create_username("ab")   # ValueError!
v = Validator.create_username("alice99")
print(v)  # Validator(alice99)
```

## Advanced Examples

### 1. Abstract classmethod and staticmethod

```python
from abc import ABC, abstractmethod


class DataSerializer(ABC):
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


class JSONSerializer(DataSerializer):
    import json

    @classmethod
    def from_string(cls, data):
        return cls(json.loads(data))

    @staticmethod
    def validate(data):
        try:
            import json
            json.loads(data)
            return True
        except (ValueError, TypeError):
            return False

    def serialize(self):
        import json
        return json.dumps(self.__dict__)


# js = DataSerializer()  # TypeError (abstract)
j = JSONSerializer.from_string('{"name": "test"}')
print(j.serialize())      # {"name": "test"}
```

### 2. Singleton via classmethod

```python
class Singleton:
    _instance = None

    def __init__(self):
        if Singleton._instance is not None:
            raise RuntimeError("Use Singleton.get_instance()")
        self.value = 42

    @classmethod
    def get_instance(cls):
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance


# s1 = Singleton()        # RuntimeError on second call
s1 = Singleton.get_instance()
s2 = Singleton.get_instance()
print(s1 is s2)           # True
print(s1.value)           # 42
```

### 3. Registry pattern with `__init_subclass__` and classmethod

```python
class Plugin:
    _registry = {}

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        name = kwargs.get("name", cls.__name__.lower())
        Plugin._registry[name] = cls

    @classmethod
    def get(cls, name):
        if name not in cls._registry:
            raise ValueError(f"Unknown plugin: {name}")
        return cls._registry[name]

    @classmethod
    def list_plugins(cls):
        return list(cls._registry.keys())

    def run(self, data):
        raise NotImplementedError


class UpperPlugin(Plugin, name="upper"):
    def run(self, data):
        return data.upper()


class ReversePlugin(Plugin, name="reverse"):
    def run(self, data):
        return data[::-1]


print(Plugin.list_plugins())          # ['upper', 'reverse']
UpperPlugin.get("upper")
plugin = Plugin.get("reverse")()
print(plugin.run("hello"))           # olleh
```

### 4. Caching with classmethod

```python
class Fibonacci:
    _cache = {0: 0, 1: 1}

    @classmethod
    def get(cls, n):
        if n < 0:
            raise ValueError("n must be non-negative")
        if n not in cls._cache:
            cls._cache[n] = cls.get(n - 1) + cls.get(n - 2)
        return cls._cache[n]

    @classmethod
    def clear_cache(cls):
        cls._cache = {0: 0, 1: 1}

    @classmethod
    def cache_size(cls):
        return len(cls._cache)


print(Fibonacci.get(10))    # 55
print(Fibonacci.get(50))    # 12586269025
print(Fibonacci.cache_size())  # 51
```

## Real-World Use Cases

- **Django models**: `classmethod` used for custom querysets (`Person.objects.filter(...)` is essentially a classmethod returning a `QuerySet`).
- **Dataclass alternatives**: `@classmethod` factory methods like `from_dict`, `from_json`, `from_csv`.
- **Configuration classes**: `@classmethod` to load config from environment variables, YAML files, or dicts.
- **API clients**: `@staticmethod` for utility functions like `build_url()`, `encode_payload()`, `validate_api_key()`.
- **Pydantic**: `@classmethod` validators (`@classmethod` + `@validator`) that operate on the class.

## Common Mistakes

| Mistake | Why it's wrong | Fix |
|---|---|---|
| Using `@staticmethod` when you need `@classmethod` | Can't access class attributes or be overridden polymorphically | Use `@classmethod` if you need `cls` |
| Calling `cls()` inside `@classmethod` incorrectly | Wrong class used in inheritance | `cls()` is correct — it refers to the actual subclass |
| Forgetting that `@staticmethod` can't access `self` or `cls` | `NameError` or unexpected behaviour | Pass needed data as arguments |
| Using `self` instead of `cls` in `@classmethod` | Confusing convention | Use `cls` for consistency |
| Overriding a `@classmethod` with `@staticmethod` | Breaks polymorphism if the caller uses `cls` | Keep the same decorator |
| Expecting `self` in `@staticmethod` | It's a plain function — no automatic instance/class reference | Accept explicit arguments |

```python
# Static method that should have been a classmethod
class Bad:
    default_value = 10

    @staticmethod
    def create(value):
        # Can't access Bad.default_value if called from a subclass
        return Bad(value)  # Always returns Bad, never subclass

class Good:
    default_value = 10

    @classmethod
    def create(cls, value=None):
        if value is None:
            value = cls.default_value
        return cls(value)  # Returns correct subclass
```

## Best Practices

- Use `@classmethod` for **factory methods** and **alternative constructors**.
- Use `@staticmethod` for **utility functions** that are conceptually related to the class but don't need access to the class or instance.
- Use `@classmethod` when the method needs to be **polymorphic** (overridden in subclasses).
- Use `@classmethod` to access or modify **class-level state**.
- Always use `cls` (not `self`) as the first parameter of `@classmethod`.
- Always use `cls()` inside `@classmethod` to create instances — this guarantees the correct subclass is returned.
- Document static methods clearly — since they don't receive `self`/`cls`, readers need to understand what they do from the docstring.

## Interview Questions

1. **What is the difference between `@classmethod` and `@staticmethod`?**
   *`@classmethod` receives `cls` (the class) and can access/modify class state and create instances. `@staticmethod` receives no special first argument — it's a plain function inside the class namespace.*

2. **Can you call a classmethod on an instance?**
   *Yes — both classmethods and staticmethods can be called on instances, though they are usually called on the class.*

3. **When would you use a classmethod instead of an instance method?**
   *When the method doesn't need instance data but needs the class — e.g., factory methods, class-level configuration, or alternative constructors.*

4. **Can a staticmethod be overridden in a subclass?**
   *Yes, but it's not polymorphic like a classmethod — the subclass's version must be explicitly called.*

5. **What is the decorator order for `@abstractmethod` and `@classmethod`?**
   *`@classmethod @abstractmethod` or `@staticmethod @abstractmethod` — the `@abstractmethod` must be the innermost decorator.*

6. **How does `cls()` in a classmethod behave with inheritance?**
   *`cls()` calls the `__init__` of the actual class the method was called on, not the class where it was defined. This makes factory methods polymorphic.*

## Coding Challenges

1. **Shape Factory with Registry**: Build a `Shape` base class with `__init_subclass__` that auto-registers. Add a `@classmethod create(name, **kwargs)` factory. Implement `Circle`, `Square`, `Triangle`.

2. **Configuration Loader**: `AppConfig` with `@classmethod` methods: `from_json(path)`, `from_env()`, `from_dict(data)`. Each returns a new `AppConfig` instance.

3. **Logger with Class-Level Configuration**: `Logger` with `@classmethod set_level(level)` and `set_format(fmt)`. Instance methods `info()`, `error()`, `debug()` respect the class-level settings.

4. **Polymorphic Serializer**: `Serializer` ABC with `@classmethod from_string(data)`, `@staticmethod validate(data)`, instance method `serialize()`. Implement `JSONSerializer` and `XMLSerializer`.

5. **ID Generator**: `IDGenerator` that keeps a class-level counter. `@classmethod next_id()` increments and returns. `@classmethod reset()` sets the counter back. `@staticmethod is_valid_id(id)` checks format.

## Summary

- **Instance methods** receive `self` and work with instance data.
- **Class methods** (`@classmethod`) receive `cls` and work with class-level data; they are ideal for factory methods and alternative constructors.
- **Static methods** (`@staticmethod`) receive no special first argument; they are plain functions namespaced in the class.
- Use `cls()` in classmethods for **polymorphic factories** that return the correct subclass.
- Use `@classmethod` for **class-level state** management and **configuration**.
- Use `@staticmethod` for **utility functions** closely related to the class.
- `@abstractmethod` can be combined with both — the abstractmethod decorator must be innermost.

## Related Topics

- Classes and Objects — methods are defined within classes
- Inheritance — classmethods are inherited and can be overridden
- Polymorphism — classmethod factories are inherently polymorphic
- Encapsulation — classmethods can control access to class-level state
- Abstraction — abstract classmethods and staticmethods define interfaces
- Properties — property methods are a different kind of method descriptor
