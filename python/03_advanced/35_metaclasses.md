# Metaclasses - type(), class creation, custom metaclasses

## Introduction

A metaclass is the class of a class — a class that defines how other classes are created. In Python, everything is an object, including classes themselves. Just as ordinary objects are instances of a class, classes are instances of a metaclass. The default metaclass is `type`. When you define a class, Python calls the metaclass to create the class object. Metaclasses intercept class creation, allowing you to modify class attributes, methods, and behavior before the class is actually created.

## Why It Is Important

Metaclasses are the ultimate tool for framework and library authors. They enable powerful patterns like singletons, registration of subclasses (plugin systems), automatic property generation, ORM model definitions (Django, SQLAlchemy), validation of class attributes, and enforcing coding standards at the class level. While rarely needed in day-to-day application code, metaclasses provide essential infrastructure for many Python frameworks.

## Syntax

```python
# Default metaclass: type
class MyClass(metaclass=type):
    pass

# Custom metaclass
class MyMeta(type):
    def __new__(mcs, name, bases, namespace):
        # Modify namespace before class creation
        return super().__new__(mcs, name, bases, namespace)

    def __init__(cls, name, bases, namespace):
        # Post-processing after class creation
        super().__init__(name, bases, namespace)

# Using metaclass in Python 3
class MyClass(metaclass=MyMeta):
    pass
```

## Examples

```python
from typing import Any, Type, ClassVar
import inspect
```

### type() as a Metaclass

```python
# Creating a class dynamically with type()
def __init__(self, name: str, age: int) -> None:
    self.name = name
    self.age = age

def greet(self) -> str:
    return f"Hello, I'm {self.name}"

Person = type('Person', (), {
    '__init__': __init__,
    'greet': greet,
    'species': 'Human'
})

p = Person("Alice", 30)
print(p.greet())
print(p.species)
```

### Basic Custom Metaclass

```python
class CapitalizeMeta(type):
    def __new__(mcs, name: str, bases: tuple[type, ...], namespace: dict[str, Any]) -> type:
        new_namespace = {}
        for attr_name, attr_value in namespace.items():
            if attr_name.startswith('_'):
                new_namespace[attr_name] = attr_value
            else:
                new_namespace[attr_name.capitalize()] = attr_value
        return super().__new__(mcs, name, bases, new_namespace)

class MyClass(metaclass=CapitalizeMeta):
    foo = 1
    bar = 2
    _internal = 3

obj = MyClass()
print(obj.Foo)  # 1
print(obj.Bar)  # 2
print(obj._internal)  # 3
```

### __new__ vs __init__ in Metaclasses

```python
class DebugMeta(type):
    def __new__(mcs, name: str, bases: tuple[type, ...], namespace: dict[str, Any]) -> type:
        print(f"  __new__: Creating class {name}")
        result = super().__new__(mcs, name, bases, namespace)
        print(f"  __new__: Created {result}")
        return result

    def __init__(cls, name: str, bases: tuple[type, ...], namespace: dict[str, Any]) -> None:
        print(f"  __init__: Initializing class {name}")
        super().__init__(name, bases, namespace)

class Base(metaclass=DebugMeta):
    pass

class Derived(Base):
    pass
```

## Beginner Examples

### Adding an Attribute to All Classes

```python
class AddAuthorMeta(type):
    def __new__(mcs, name: str, bases: tuple[type, ...], namespace: dict[str, Any]) -> type:
        namespace['author'] = "Unknown"
        return super().__new__(mcs, name, bases, namespace)

class Book(metaclass=AddAuthorMeta):
    pass

class Article(metaclass=AddAuthorMeta):
    pass

print(Book.author)
print(Article.author)
```

### Preventing Class Instantiation

```python
class NoInstanceMeta(type):
    def __call__(cls, *args: Any, **kwargs: Any) -> None:
        raise TypeError(f"Cannot instantiate {cls.__name__}")

class Constants(metaclass=NoInstanceMeta):
    PI = 3.14159
    E = 2.71828

print(Constants.PI)
# Constants()  # Raises TypeError
```

### Auto-registering Subclasses

```python
class RegistryMeta(type):
    registry: dict[str, type] = {}

    def __new__(mcs, name: str, bases: tuple[type, ...], namespace: dict[str, Any]) -> type:
        cls = super().__new__(mcs, name, bases, namespace)
        if not name.startswith('_'):
            mcs.registry[name] = cls
        return cls

class BasePlugin(metaclass=RegistryMeta):
    pass

class PluginA(BasePlugin):
    pass

class PluginB(BasePlugin):
    pass

class PluginC(BasePlugin):
    pass

print(RegistryMeta.registry)
```

## Intermediate Examples

### Singleton Metaclass

```python
class SingletonMeta(type):
    _instances: dict[type, object] = {}

    def __call__(cls, *args: Any, **kwargs: Any) -> Any:
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self) -> None:
        print("Initializing Database connection...")
        self.connected = True

db1 = Database()
db2 = Database()
print(db1 is db2)  # True
```

### Validation Metaclass

```python
class ValidateAttributesMeta(type):
    def __new__(mcs, name: str, bases: tuple[type, ...], namespace: dict[str, Any]) -> type:
        annotations = namespace.get('__annotations__', {})
        for attr_name, attr_type in annotations.items():
            if attr_name.startswith('_'):
                continue
            default = namespace.get(attr_name)
            if default is not None and not isinstance(default, attr_type):
                raise TypeError(
                    f"Default value for '{attr_name}' must be {attr_type.__name__}, "
                    f"got {type(default).__name__}"
                )
        return super().__new__(mcs, name, bases, namespace)

    def __call__(cls, *args: Any, **kwargs: Any) -> Any:
        instance = super().__call__(*args, **kwargs)
        annotations = getattr(cls, '__annotations__', {})
        for attr_name, attr_type in annotations.items():
            if hasattr(instance, attr_name):
                value = getattr(instance, attr_name)
                if not isinstance(value, attr_type):
                    raise TypeError(
                        f"Attribute '{attr_name}' must be {attr_type.__name__}, "
                        f"got {type(value).__name__}"
                    )
        return instance

class Person(metaclass=ValidateAttributesMeta):
    name: str
    age: int

    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age

p = Person("Alice", 30)  # OK
# p = Person("Alice", "30")  # Raises TypeError
```

### Metaclass for Adding Methods

```python
class AddMethodsMeta(type):
    def __new__(mcs, name: str, bases: tuple[type, ...], namespace: dict[str, Any]) -> type:
        # Add a to_dict method
        def to_dict(self) -> dict[str, Any]:
            result: dict[str, Any] = {}
            for key, value in self.__dict__.items():
                if not key.startswith('_'):
                    result[key] = value
            return result
        namespace['to_dict'] = to_dict

        # Add a from_dict class method
        @classmethod
        def from_dict(cls, data: dict[str, Any]) -> Any:
            return cls(**data)
        namespace['from_dict'] = from_dict

        return super().__new__(mcs, name, bases, namespace)

class Point(metaclass=AddMethodsMeta):
    def __init__(self, x: float, y: float) -> None:
        self.x = x
        self.y = y

p = Point(3, 4)
print(p.to_dict())
p2 = Point.from_dict({"x": 5, "y": 6})
print(p2.to_dict())
```

### Metaclass for Logging Method Calls

```python
class LoggingMeta(type):
    def __new__(mcs, name: str, bases: tuple[type, ...], namespace: dict[str, Any]) -> type:
        for attr_name, attr_value in namespace.items():
            if callable(attr_value) and not attr_name.startswith('_'):
                original = attr_value
                def make_logged(name: str, func: Any) -> Any:
                    def logged(self, *args: Any, **kwargs: Any) -> Any:
                        print(f"Calling {name} on {self.__class__.__name__}")
                        result = func(self, *args, **kwargs)
                        print(f"{name} returned {result}")
                        return result
                    return logged
                namespace[attr_name] = make_logged(attr_name, original)
        return super().__new__(mcs, name, bases, namespace)

class Calculator(metaclass=LoggingMeta):
    def add(self, a: int, b: int) -> int:
        return a + b

    def multiply(self, a: int, b: int) -> int:
        return a * b

calc = Calculator()
calc.add(3, 4)
calc.multiply(5, 6)
```

## Advanced Examples

### ORM-style Field Definition (like Django/SQLAlchemy)

```python
class Field:
    def __init__(self, field_type: type, default: Any = None) -> None:
        self.field_type = field_type
        self.default = default
        self.name: str = ""

class ModelMeta(type):
    def __new__(mcs, name: str, bases: tuple[type, ...], namespace: dict[str, Any]) -> type:
        fields: dict[str, Field] = {}
        for attr_name, attr_value in namespace.items():
            if isinstance(attr_value, Field):
                attr_value.name = attr_name
                fields[attr_name] = attr_value
        namespace['_fields'] = fields
        return super().__new__(mcs, name, bases, namespace)

class BaseModel(metaclass=ModelMeta):
    def __init__(self, **kwargs: Any) -> None:
        for field_name, field in self._fields.items():
            value = kwargs.get(field_name, field.default)
            if value is not None and not isinstance(value, field.field_type):
                raise TypeError(
                    f"Field '{field_name}' expects {field.field_type.__name__}, "
                    f"got {type(value).__name__}"
                )
            setattr(self, field_name, value)

    def __repr__(self) -> str:
        fields = ", ".join(
            f"{n}={getattr(self, n, None)}"
            for n in self._fields
        )
        return f"{self.__class__.__name__}({fields})"

class User(BaseModel):
    name = Field(str)
    age = Field(int, default=0)
    email = Field(str, default="")

u = User(name="Alice", age=30, email="alice@example.com")
print(u)
print(u._fields)
```

### Metaclass for Property Generation

```python
class AutoPropertyMeta(type):
    def __new__(mcs, name: str, bases: tuple[type, ...], namespace: dict[str, Any]) -> type:
        annotations = namespace.get('__annotations__', {})
        for attr_name in list(annotations.keys()):
            if attr_name.startswith('_'):
                continue
            private_name = f"_{attr_name}"
            namespace[private_name] = namespace.pop(attr_name, None)

            def make_property(name: str, private: str) -> property:
                def getter(self) -> Any:
                    return getattr(self, private)
                def setter(self, value: Any) -> None:
                    setattr(self, private, value)
                return property(getter, setter)

            namespace[attr_name] = make_property(attr_name, private_name)

        return super().__new__(mcs, name, bases, namespace)

class Person(metaclass=AutoPropertyMeta):
    name: str
    age: int

    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age

p = Person("Alice", 30)
print(p.name)
p.name = "Bob"
print(p.name)
```

### Metaclass for Input Validation at Construction

```python
class ConstrainedMeta(type):
    constraints: dict[str, Any] = {}

    def __new__(mcs, name: str, bases: tuple[type, ...], namespace: dict[str, Any]) -> type:
        constraints = namespace.get('__constraints__', {})
        original_init = namespace.get('__init__')

        if original_init:
            @staticmethod
            def make_constrained_init(orig: Any, cons: dict[str, Any]) -> Any:
                def constrained_init(self, *args: Any, **kwargs: Any) -> None:
                    orig(self, *args, **kwargs)
                    for attr_name, validator in cons.items():
                        if hasattr(self, attr_name):
                            value = getattr(self, attr_name)
                            if not validator(value):
                                raise ValueError(
                                    f"Validation failed for {attr_name}: {value}"
                                )
                return constrained_init
            namespace['__init__'] = make_constrained_init(original_init, constraints)

        return super().__new__(mcs, name, bases, namespace)

class PositiveInteger:
    @staticmethod
    def validate(value: int) -> bool:
        return isinstance(value, int) and value > 0

class Person(metaclass=ConstrainedMeta):
    __constraints__ = {
        'age': PositiveInteger.validate,
    }

    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age

p = Person("Alice", 25)  # OK
# p = Person("Bob", -5)    # Raises ValueError
```

### Metaclass for Abstract Method Enforcement at Construction

```python
class InterfaceMeta(type):
    def __new__(mcs, name: str, bases: tuple[type, ...], namespace: dict[str, Any]) -> type:
        abstract_methods = set()
        for base in bases:
            abstract_methods.update(getattr(base, '_abstract_methods', set()))

        for attr_name, attr_value in namespace.items():
            if getattr(attr_value, '_is_abstract', False):
                abstract_methods.add(attr_name)

        # Check if this is a concrete implementation
        if not getattr(namespace.get('__is_abstract__'), '_is_abstract', False):
            concrete_class = super().__new__(mcs, name, bases, namespace)
            missing = abstract_methods - set(concrete_class.__dict__.keys())
            if missing:
                raise TypeError(
                    f"Can't instantiate {name}: missing abstract methods: {missing}"
                )
            return concrete_class

        namespace['_abstract_methods'] = abstract_methods
        return super().__new__(mcs, name, bases, namespace)

def abstract(func: Any) -> Any:
    func._is_abstract = True
    return func

# Usage:
# class Shape(metaclass=InterfaceMeta):
#     __is_abstract__ = True
#
#     @abstract
#     def area(self):
#         pass
#
# class Circle(Shape):
#     def __init__(self, r):
#         self.r = r
#     def area(self):
#         return 3.14 * self.r ** 2
#
# c = Circle(5)  # OK
```

### Metaclass for Automatic __repr__ and __str__

```python
class AutoReprMeta(type):
    def __new__(mcs, name: str, bases: tuple[type, ...], namespace: dict[str, Any]) -> type:
        if '__repr__' not in namespace:
            def __repr__(self) -> str:
                cls = self.__class__
                attrs = ", ".join(
                    f"{k}={v!r}"
                    for k, v in self.__dict__.items()
                    if not k.startswith('_')
                )
                return f"{cls.__name__}({attrs})"
            namespace['__repr__'] = __repr__
        return super().__new__(mcs, name, bases, namespace)

class Point(metaclass=AutoReprMeta):
    def __init__(self, x: int, y: int) -> None:
        self.x = x
        self.y = y

p = Point(10, 20)
print(p)
```

## Real-World Use Cases

- **Django ORM**: Model classes are defined using metaclasses to create database table mappings.
- **SQLAlchemy ORM**: Declarative base uses metaclasses for table definitions.
- **Python's ABC (abc.ABCMeta)**: Enables abstract base classes with `@abstractmethod`.
- **Singleton pattern**: Ensuring a class has only one instance.
- **Plugin/registry systems**: Auto-registering subclasses of a base class.
- **Serialization frameworks**: Automatically generating serialization/deserialization code.
- **Validation frameworks**: Enforcing type and value constraints on class attributes.
- **Aspect-oriented programming**: Adding logging, tracing, or security checks automatically.

## Common Mistakes

- Using metaclasses when a simpler solution (decorator, inheritance, `__init_subclass__`) would work.
- Forgetting to call `super().__new__()` or `super().__init__()` in metaclass methods.
- Confusing `__new__` (called before class creation) with `__init__` (called after).
- Overusing metaclasses — they add complexity and can make code hard to debug.
- Modifying class attributes in ways that break `isinstance()` and `issubclass()` checks.
- Trying to use metaclass syntax from Python 2 (`__metaclass__`) in Python 3.
- Not understanding that metaclass methods operate on classes, not instances.

## Best Practices

- Prefer `__init_subclass__`, decorators, or inheritance over metaclasses when possible.
- Keep metaclass logic focused on class creation only.
- Use descriptive names for metaclasses (suffix with `Meta` or `MetaClass`).
- Document clearly what your metaclass does and why it's needed.
- Always call `super()` to maintain the MRO chain.
- Avoid metaclasses in libraries meant for public consumption unless absolutely necessary.
- Use `__new__` for modifying the class before creation; use `__init__` for post-processing.

## Interview Questions

1. What is a metaclass in Python?
2. How do you create a custom metaclass?
3. What is the difference between `type.__new__` and `type.__init__` in a metaclass?
4. How do metaclasses relate to class creation?
5. What is the Singleton pattern and how do you implement it with a metaclass?
6. How do Django and SQLAlchemy use metaclasses?
7. What is the MRO and how do metaclasses affect it?
8. How does `__init_subclass__` differ from metaclasses?
9. What is the difference between a class decorator and a metaclass?
10. How do you dynamically create a class using `type()`?

## Coding Challenges

1. **Singleton Metaclass**: Create a metaclass that ensures only one instance of a class exists.
2. **Attribute Validation**: Build a metaclass that validates types of all annotated attributes.
3. **Plugin Registry**: Implement a metaclass that auto-registers all subclasses.
4. **Auto-Properties**: Create a metaclass that converts annotated attributes into properties with getters/setters.
5. **Logging Metaclass**: Build a metaclass that logs every method call on instances.
6. **AbstractInterface**: Create a metaclass that enforces abstract method implementation.
7. **ORM Fields**: Implement a simple ORM-style field system with a metaclass.
8. **Auto-Repr**: Write a metaclass that automatically generates `__repr__` and `__str__` methods.

## Summary

Metaclasses are classes that create classes, intercepting the class creation process to modify or enhance the resulting class. They are the deepest level of Python's metaprogramming system and power many frameworks (Django, SQLAlchemy, ABC). While powerful, they should be used sparingly; simpler alternatives like decorators and `__init_subclass__` often suffice.

## Related Topics

- type() and dynamic class creation
- __new__ vs __init__
- Class decorators
- __init_subclass__
- Abstract base classes (abc module)
- Descriptors
- Decorators
- Class creation protocol