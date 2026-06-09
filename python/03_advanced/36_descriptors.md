# Descriptors - __get__, __set__, __delete__, descriptor protocol

## Introduction

A descriptor is a Python object that implements at least one of the methods `__get__`, `__set__`, or `__delete__`, known as the **descriptor protocol**. Descriptors control how attribute access works on other objects. When a class attribute is a descriptor, Python binds its special methods to the attribute access on instances. The most common descriptors are `property`, `classmethod`, and `staticmethod`, but custom descriptors enable powerful, reusable attribute behavior.

## Why It Is Important

Descriptors are the underlying mechanism for Python's property system, bound methods, `super()`, and class/static methods. Understanding descriptors is essential for creating reusable attribute validation, computed attributes, lazy evaluation, type checking, and ORM field definitions. They provide fine-grained control over attribute access while maintaining a clean user-facing API.

## Syntax

```python
# Descriptor protocol
class Descriptor:
    def __get__(self, obj, objtype=None):
        return value

    def __set__(self, obj, value):
        # validation, transformation, etc.
        pass

    def __delete__(self, obj):
        # cleanup
        pass

# Using a descriptor
class MyClass:
    attr = Descriptor()

# __set_name__ (Python 3.6+)
class DescriptorWithName:
    def __set_name__(self, owner, name):
        self.name = name
        self.private_name = f"_{name}"
```

## Examples

```python
from typing import Any, Optional, Type
```

### Basic Read-Only Descriptor

```python
class ReadOnly:
    def __init__(self, value: Any) -> None:
        self._value = value

    def __get__(self, obj: Optional[object], objtype: Optional[Type[object]] = None) -> Any:
        if obj is None:
            return self
        return self._value

    def __set__(self, obj: object, value: Any) -> None:
        raise AttributeError("Cannot modify read-only attribute")

class Config:
    VERSION = ReadOnly("1.0.0")
    API_URL = ReadOnly("https://api.example.com")

cfg = Config()
print(cfg.VERSION)
# cfg.VERSION = "2.0"  # Raises AttributeError
```

### Type-Checking Descriptor

```python
class Typed:
    def __init__(self, expected_type: type, default: Any = None) -> None:
        self.expected_type = expected_type
        self.default = default

    def __set_name__(self, owner: Type[object], name: str) -> None:
        self.name = name
        self.private_name = f"_{name}"

    def __get__(self, obj: Optional[object], objtype: Optional[Type[object]] = None) -> Any:
        if obj is None:
            return self
        return getattr(obj, self.private_name, self.default)

    def __set__(self, obj: object, value: Any) -> None:
        if not isinstance(value, self.expected_type):
            raise TypeError(
                f"{self.name} must be {self.expected_type.__name__}, "
                f"got {type(value).__name__}"
            )
        setattr(obj, self.private_name, value)

class Person:
    name = Typed(str)
    age = Typed(int)
    salary = Typed(float)

    def __init__(self, name: str, age: int, salary: float) -> None:
        self.name = name
        self.age = age
        self.salary = salary

p = Person("Alice", 30, 75000.0)
print(p.name, p.age, p.salary)
# p.name = 123  # Raises TypeError
```

## Beginner Examples

### Simple Property-like Descriptor

```python
class PositiveNumber:
    def __set_name__(self, owner: Type[object], name: str) -> None:
        self.name = name
        self.private_name = f"_{name}"

    def __get__(self, obj: Optional[object], objtype: Optional[Type[object]] = None) -> Any:
        if obj is None:
            return self
        return getattr(obj, self.private_name, 0)

    def __set__(self, obj: object, value: float) -> None:
        if value < 0:
            raise ValueError(f"{self.name} must be non-negative, got {value}")
        setattr(obj, self.private_name, value)

class Rectangle:
    width = PositiveNumber()
    height = PositiveNumber()

    def __init__(self, width: float, height: float) -> None:
        self.width = width
        self.height = height

    @property
    def area(self) -> float:
        return self.width * self.height

r = Rectangle(10, 5)
print(r.area)
# r.width = -5  # Raises ValueError
```

### Validation Descriptor

```python
class Validated:
    def __init__(self, validator) -> None:
        self.validator = validator

    def __set_name__(self, owner: Type[object], name: str) -> None:
        self.name = name
        self.private_name = f"_{name}"

    def __get__(self, obj: Optional[object], objtype: Optional[Type[object]] = None) -> Any:
        if obj is None:
            return self
        return getattr(obj, self.private_name)

    def __set__(self, obj: object, value: Any) -> None:
        self.validator(value)
        setattr(obj, self.private_name, value)

def validate_email(value: str) -> None:
    if "@" not in value:
        raise ValueError(f"Invalid email: {value}")

def validate_age(value: int) -> None:
    if not (0 < value < 150):
        raise ValueError(f"Invalid age: {value}")

class User:
    email = Validated(validate_email)
    age = Validated(validate_age)

    def __init__(self, email: str, age: int) -> None:
        self.email = email
        self.age = age

u = User("alice@example.com", 30)
# u = User("invalid", 30)  # Raises ValueError
```

### Lazy Computation Descriptor

```python
class LazyProperty:
    def __init__(self, func) -> None:
        self.func = func
        self.name = func.__name__
        self.private_name = f"_lazy_{self.name}"

    def __get__(self, obj: Optional[object], objtype: Optional[Type[object]] = None) -> Any:
        if obj is None:
            return self
        if not hasattr(obj, self.private_name):
            value = self.func(obj)
            setattr(obj, self.private_name, value)
        return getattr(obj, self.private_name)

    def __set__(self, obj: object, value: Any) -> None:
        raise AttributeError("Cannot set lazy property")

class Circle:
    def __init__(self, radius: float) -> None:
        self.radius = radius

    @LazyProperty
    def area(self) -> float:
        print("Computing area...")
        return 3.14159 * self.radius ** 2

c = Circle(10)
print(c.area)
print(c.area)  # No recomputation
```

## Intermediate Examples

### Descriptor with Access Logging

```python
class LoggedAccess:
    def __set_name__(self, owner: Type[object], name: str) -> None:
        self.name = name
        self.private_name = f"_{name}"

    def __get__(self, obj: Optional[object], objtype: Optional[Type[object]] = None) -> Any:
        if obj is None:
            return self
        value = getattr(obj, self.private_name)
        print(f"GET {self.name} = {value!r}")
        return value

    def __set__(self, obj: object, value: Any) -> None:
        print(f"SET {self.name} = {value!r}")
        setattr(obj, self.private_name, value)

    def __delete__(self, obj: object) -> None:
        print(f"DELETE {self.name}")
        delattr(obj, self.private_name)

class Settings:
    debug = LoggedAccess()
    timeout = LoggedAccess()

    def __init__(self) -> None:
        self.debug = False
        self.timeout = 30

s = Settings()
s.debug = True
print(s.debug)
del s.debug
```

### Range-Validation Descriptor

```python
class Bounded:
    def __init__(self, minimum: float, maximum: float) -> None:
        self.minimum = minimum
        self.maximum = maximum

    def __set_name__(self, owner: Type[object], name: str) -> None:
        self.name = name
        self.private_name = f"_{name}"

    def __get__(self, obj: Optional[object], objtype: Optional[Type[object]] = None) -> Any:
        if obj is None:
            return self
        return getattr(obj, self.private_name)

    def __set__(self, obj: object, value: float) -> None:
        if not (self.minimum <= value <= self.maximum):
            raise ValueError(
                f"{self.name} must be between {self.minimum} and {self.maximum}, "
                f"got {value}"
            )
        setattr(obj, self.private_name, value)

class Temperature:
    celsius = Bounded(-273.15, 10000)

    def __init__(self, celsius: float) -> None:
        self.celsius = celsius

t = Temperature(25)
print(t.celsius)
# t.celsius = -300  # Raises ValueError
```

### Computed (Derived) Attribute Descriptor

```python
class Computed:
    def __init__(self, func) -> None:
        self.func = func
        self.name = func.__name__

    def __set_name__(self, owner: Type[object], name: str) -> None:
        self.name = name

    def __get__(self, obj: Optional[object], objtype: Optional[Type[object]] = None) -> Any:
        if obj is None:
            return self
        return self.func(obj)

    def __set__(self, obj: object, value: Any) -> None:
        raise AttributeError(f"{self.name} is computed and cannot be set")

class Rectangle:
    def __init__(self, width: float, height: float) -> None:
        self.width = width
        self.height = height

    @Computed
    def area(self) -> float:
        return self.width * self.height

    @Computed
    def perimeter(self) -> float:
        return 2 * (self.width + self.height)

r = Rectangle(10, 5)
print(r.area)
print(r.perimeter)
```

### Unit Conversion Descriptor

```python
class UnitConversion:
    def __init__(self, factor: float) -> None:
        self.factor = factor

    def __set_name__(self, owner: Type[object], name: str) -> None:
        self.name = name
        self.internal_name = f"_in_meters"

    def __get__(self, obj: Optional[object], objtype: Optional[Type[object]] = None) -> Any:
        if obj is None:
            return self
        return getattr(obj, self.internal_name, 0.0) / self.factor

    def __set__(self, obj: object, value: float) -> None:
        setattr(obj, self.internal_name, value * self.factor)

class Distance:
    meters = UnitConversion(1.0)
    kilometers = UnitConversion(1000.0)
    miles = UnitConversion(1609.34)
    feet = UnitConversion(0.3048)

    def __init__(self, meters: float = 0.0) -> None:
        self.meters = meters

d = Distance()
d.meters = 1000
print(f"{d.meters:.2f} m = {d.kilometers:.2f} km = {d.miles:.2f} miles")
```

## Advanced Examples

### Implementing property() with Descriptors

```python
class Property:
    """Reimplementation of the built-in property descriptor."""
    def __init__(self, fget=None, fset=None, fdel=None, doc=None) -> None:
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        if doc is None and fget is not None:
            doc = fget.__doc__
        self.__doc__ = doc

    def __get__(self, obj: Optional[object], objtype: Optional[Type[object]] = None) -> Any:
        if obj is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(obj)

    def __set__(self, obj: object, value: Any) -> None:
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(obj, value)

    def __delete__(self, obj: object) -> None:
        if self.fdel is None:
            raise AttributeError("can't delete attribute")
        self.fdel(obj)

    def setter(self, fset):
        return Property(self.fget, fset, self.fdel)

    def deleter(self, fdel):
        return Property(self.fget, self.fset, fdel)

class Person:
    def __init__(self, name: str) -> None:
        self._name = name

    @Property
    def name(self) -> str:
        return self._name

    @name.setter
    def name(self, value: str) -> None:
        if not isinstance(value, str):
            raise TypeError("Name must be a string")
        self._name = value

p = Person("Alice")
print(p.name)
p.name = "Bob"
print(p.name)
```

### Descriptor for Cached Properties with Invalidation

```python
class CachedProperty:
    def __init__(self, func) -> None:
        self.func = func
        self.name = func.__name__
        self.cache_name = f"_cache_{self.name}"

    def __get__(self, obj: Optional[object], objtype: Optional[Type[object]] = None) -> Any:
        if obj is None:
            return self
        cache = getattr(obj, self.cache_name, None)
        if cache is None:
            value = self.func(obj)
            setattr(obj, self.cache_name, value)
            return value
        return cache

    def __set__(self, obj: object, value: Any) -> None:
        raise AttributeError("Cannot set cached property")

    def invalidate(self, obj: object) -> None:
        if hasattr(obj, self.cache_name):
            delattr(obj, self.cache_name)

class DataProcessor:
    def __init__(self, data: list[int]) -> None:
        self.data = data

    @CachedProperty
    def processed(self) -> list[int]:
        print("Processing data...")
        return [x ** 2 for x in self.data]

    @CachedProperty
    def summary(self) -> dict[str, Any]:
        print("Computing summary...")
        processed = self.processed
        return {
            "min": min(processed),
            "max": max(processed),
            "sum": sum(processed),
        }

dp = DataProcessor([1, 2, 3, 4, 5])
print(dp.summary)
print(dp.summary)  # Cached
```

### Boolean Flag Descriptor

```python
class BooleanFlag:
    def __set_name__(self, owner: Type[object], name: str) -> None:
        self.name = name
        self.private_name = f"_{name}"

    def __get__(self, obj: Optional[object], objtype: Optional[Type[object]] = None) -> bool:
        if obj is None:
            return self
        return getattr(obj, self.private_name, False)

    def __set__(self, obj: object, value: Any) -> None:
        setattr(obj, self.private_name, bool(value))

    def __delete__(self, obj: object) -> None:
        setattr(obj, self.private_name, False)

class FeatureFlags:
    dark_mode = BooleanFlag()
    beta_features = BooleanFlag()
    analytics = BooleanFlag()

    def __init__(self) -> None:
        self.dark_mode = True
        self.beta_features = False

flags = FeatureFlags()
print(flags.dark_mode)
print(flags.beta_features)
del flags.dark_mode
print(flags.dark_mode)
```

### Descriptor for Enum-like Attributes

```python
class EnumChoice:
    def __init__(self, *valid_values: str) -> None:
        self.valid_values = valid_values

    def __set_name__(self, owner: Type[object], name: str) -> None:
        self.name = name
        self.private_name = f"_{name}"

    def __get__(self, obj: Optional[object], objtype: Optional[Type[object]] = None) -> Any:
        if obj is None:
            return self
        return getattr(obj, self.private_name)

    def __set__(self, obj: object, value: str) -> None:
        if value not in self.valid_values:
            raise ValueError(
                f"{self.name} must be one of {self.valid_values}, got '{value}'"
            )
        setattr(obj, self.private_name, value)

class Order:
    status = EnumChoice("pending", "shipped", "delivered", "cancelled")

    def __init__(self, status: str = "pending") -> None:
        self.status = status

o = Order("shipped")
print(o.status)
# o.status = "unknown"  # Raises ValueError
```

### Descriptor with Custom Serialization

```python
import json

class JSONSerializable:
    def __set_name__(self, owner: Type[object], name: str) -> None:
        self.name = name
        self.private_name = f"_{name}"

    def __get__(self, obj: Optional[object], objtype: Optional[Type[object]] = None) -> Any:
        if obj is None:
            return self
        return getattr(obj, self.private_name)

    def __set__(self, obj: object, value: Any) -> None:
        if isinstance(value, str):
            try:
                value = json.loads(value)
            except json.JSONDecodeError:
                pass
        if not isinstance(value, (list, dict)):
            raise TypeError(f"{self.name} must be a list or dict (or JSON string)")
        setattr(obj, self.private_name, value)

class Configuration:
    data = JSONSerializable()

    def __init__(self, data: Any = None) -> None:
        self.data = data or {}

config = Configuration({"key": "value"})
print(config.data)
config.data = '{"a": 1, "b": 2}'
print(config.data)
```

## Real-World Use Cases

- **@property**: The built-in property decorator is implemented as a descriptor.
- **@classmethod** and **@staticmethod**: Both are descriptors that modify method binding.
- **Django model fields**: Each field (CharField, IntegerField) is a descriptor.
- **SQLAlchemy ORM**: Column descriptors handle database column mapping.
- **Form validation**: WTForms and Django forms use descriptors for field validation.
- **Pydantic**: Field validation and type coercion use descriptors.
- **Pyramid's @reify**: A lazy property descriptor.
- **Traits library (Enthought)**: Extensive use of descriptors for type checking and validation.

## Common Mistakes

- Forgetting that descriptors are class-level, not instance-level attributes.
- Not handling the case when `obj` is `None` in `__get__` (returning `self` for class access).
- Using mutable default values in descriptors — they are shared across instances.
- Not calling `__set_name__` correctly (available only in Python 3.6+).
- Confusing data descriptors (define `__set__` or `__delete__`) with non-data descriptors (only `__get__`).
- Not understanding that `__set__` overrides instance attribute assignment due to descriptor precedence.
- Creating descriptor instances that are shared between different attribute names.

## Best Practices

- Always implement `__set_name__` to store the attribute name for error messages.
- Use the `private_name = f"_{name}"` pattern to store instance data with name mangling.
- Handle `obj is None` in `__get__` to support class-level access.
- Clearly document whether your descriptor is a data descriptor (has `__set__`) or non-data.
- Use `__slots__` in the host class if you want to prevent regular attribute addition.
- Prefer `property` for simple cases; use descriptors only for reusable attribute patterns.
- Consider using `__init_subclass__` or decorators if descriptors become too complex.

## Interview Questions

1. What is the descriptor protocol in Python?
2. What is the difference between a data descriptor and a non-data descriptor?
3. How does `property` work as a descriptor?
4. What is `__set_name__` and when is it called?
5. What is the precedence of attribute lookup involving descriptors?
6. How does `@classmethod` work as a descriptor?
7. How would you implement a type-checking descriptor?
8. What is `super()` and how does it relate to descriptors?
9. How do descriptors interact with `__slots__`?
10. Implement a lazy evaluation descriptor.

## Coding Challenges

1. **TypeChecked**: Create a generic descriptor that validates attribute types.
2. **RangeChecked**: Build a descriptor that validates numeric ranges.
3. **CachedProperty**: Implement a descriptor that caches the result of a method call.
4. **ReadOnly**: Create a descriptor that makes an attribute read-only after initialization.
5. **UnitConverter**: Build descriptors that automatically convert between units.
6. **AccessLogger**: Implement a descriptor that logs all reads/writes to an attribute.
7. **DefaultAttr**: Create a descriptor that returns a default value if the attribute is not set.
8. **ValidatedString**: Build a descriptor that validates string patterns (regex).

## Summary

Descriptors are objects that define custom behavior for attribute access via `__get__`, `__set__`, and `__delete__`. They power Python's `property`, `classmethod`, and `staticmethod`, and are essential for creating reusable attribute validation, lazy computation, and ORM field systems. Understanding the descriptor protocol is key to mastering Python's attribute access model.

## Related Topics

- Properties
- __getattr__ vs __getattribute__
- Attribute access precedence
- Metaclasses
- Property decorator
- classmethod and staticmethod
- __slots__
- Python's data model