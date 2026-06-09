# Metaclasses - type(), class creation, custom metaclasses

## Introduction

Metaclasses are the "classes of classes" in Python. Just as a class defines how instances behave, a metaclass defines how classes behave. Every class in Python is an instance of a metaclass, most commonly `type`. Metaclasses intercept class creation, allowing you to modify class attributes, add methods, enforce coding standards, implement singleton patterns, and much more. They are one of the most advanced and powerful features of Python's object-oriented system.

## type() Metaclass

### What It Is

`type()` is the default metaclass in Python. When called with one argument, `type(obj)` returns the type of the object. When called with three arguments, `type(name, bases, dict)` dynamically creates a new class. Everything in Python, including classes, is an object, and every object, including classes, has a type. The type of most built-in and user-defined classes is `type`.

### Why It Is Important

Understanding `type()` as a metaclass is fundamental to understanding Python's object model. It reveals that classes are themselves objects that can be created, modified, and manipulated at runtime. This enables dynamic class generation, framework APIs, and the entire metaclass system.

### How It Works Internally

When Python executes a `class` statement, it collects the class name, base classes, and namespace (from the class body), then calls `metaclass(name, bases, namespace)` to create the class object. For a regular class, this is `type(name, bases, namespace)`. The metaclass's `__new__` and `__init__` methods are responsible for creating and initializing the class object.

### Syntax

```python
type(name, bases, dict)
# name: string, the class name
# bases: tuple, the base classes
# dict: dict, the namespace (attributes and methods)
```

### Beginner Examples

```python
# Dynamically creating a class
Person = type('Person', (object,), {
    'species': 'Human',
    '__init__': lambda self, name: setattr(self, 'name', name),
    'greet': lambda self: f"Hello, I'm {self.name}"
})

p = Person("Alice")
print(p.greet())       # Hello, I'm Alice
print(p.species)       # Human
print(type(p))         # <class '__main__.Person'>
print(type(Person))    # <class 'type'>

# Checking metaclasses
print(type(int))       # <class 'type'>
print(type(type))      # <class 'type'> - type is its own metaclass
```

### Intermediate Examples

```python
# Adding methods conditionally
def create_model(name, fields, base=object):
    namespace = {}
    for field_name, field_type in fields.items():
        if field_type == str:
            namespace[field_name] = property(
                lambda self, n=field_name: self.__dict__.get(n, ''),
                lambda self, val, n=field_name: setattr(self, n, str(val))
            )
        elif field_type == int:
            namespace[field_name] = property(
                lambda self, n=field_name: self.__dict__.get(n, 0),
                lambda self, val, n=field_name: setattr(self, n, int(val))
            )
    namespace['__init__'] = lambda self, **kwargs: [setattr(self, k, v) for k, v in kwargs.items()]
    return type(name, (base,), namespace)

Model = create_model('Model', {'name': str, 'age': int})
m = Model(name='Alice', age=30)
print(m.name, m.age)  # Alice 30
```

### Advanced Examples

```python
# Dynamic subclass creation
class Animal:
    def speak(self):
        raise NotImplementedError

def make_animal_sound(sound):
    namespace = {'speak': lambda self: sound}
    return type(f'Animal_{sound}', (Animal,), namespace)

Dog = make_animal_sound('Woof')
Cat = make_animal_sound('Meow')

print(Dog().speak())  # Woof
print(Cat().speak())  # Meow
print(type(Dog))      # <class 'type'>
```

### Real-World Use Cases

- **ORM frameworks** (SQLAlchemy, Django) use metaclasses to convert class definitions into database schemas.
- **Serialization libraries** generate serializers/deserializers from class definitions.
- **Proxy/wrapper generators** dynamically create proxy classes.
- **API client generators** create classes from API schema definitions.
- **Plugin systems** discover and register plugins by class type.

### Common Mistakes

- Using metaclasses when a simpler solution (decorator, inheritance, `__init_subclass__`) would work.
- Forgetting that `type` creates a new class, not an instance.
- Modifying the wrong namespace — `dict` in `type()` is the class namespace, not instance.
- Overcomplicating class creation when `__init_subclass__` suffices.

### Best Practices

- Use metaclasses sparingly — they are powerful but complex.
- Prefer class decorators or `__init_subclass__` for most class customization needs.
- Follow "don't repeat yourself" (DRY) — metaclasses are good for cross-cutting concerns.
- Document metaclass behavior thoroughly for future maintainers.
- Test metaclass-based classes extensively due to their complexity.

### Performance Considerations

Metaclasses run at class definition time, not instance creation time. The overhead is paid once per class, not per instance. However, metaclass `__call__` can affect instance creation speed. Avoid heavy computation in metaclass `__init__` or `__new__` that runs at class definition time.

### Interview Questions

**Q: What is the difference between a metaclass and a base class?**

A: A base class defines behavior inherited by subclasses. A metaclass defines how classes themselves are constructed. A class inherits from a base class; a class is an instance of a metaclass.

**Q: What is the `__call__` method in a metaclass used for?**

A: The `__call__` method on a metaclass is invoked when an instance of the class is created. It can customize instance creation, for example implementing singletons or object pooling.

### Coding Challenges

1. Create a `SingletonMeta` metaclass that implements the singleton pattern.
2. Implement a `ValidateAttributesMeta` that ensures all methods have docstrings.
3. Build a metaclass that automatically adds property accessors for class attributes starting with `_`.

### Related Topics

- `__init_subclass__` (simpler alternative to metaclasses)
- Class decorators (often simpler than metaclasses)
- `__new__` vs `__init__` in metaclasses
- `type()` and class creation
- ABCMeta (abstract base classes)

## Class Creation Process

### What It Is

The class creation process in Python is the sequence of steps Python goes through when executing a `class` statement. This process involves determining the metaclass, preparing the namespace, executing the class body, and calling the metaclass to construct the class object.

### Why It Is Important

Understanding the class creation process demystifies how classes work in Python. It explains how metaclasses can intercept and modify class creation, how `__init_subclass__` hooks work, and how frameworks like Django and SQLAlchemy derive schema from class definitions.

### How It Works Internally

The class creation process follows these steps:
1. Determine the metaclass: Python looks for a `metaclass` keyword argument, then the first base class's metaclass, then `type`.
2. Prepare the namespace: Calls `metaclass.__prepare__(name, bases, **kwargs)` to get the namespace (a dict-like object).
3. Execute the class body: The class body is executed in the prepared namespace.
4. Call the metaclass: `metaclass(name, bases, namespace, **kwargs)` creates the class object.
5. Install the class: The class object is bound to its name in the enclosing scope.

### Syntax

```python
class MyClass(MyBase, metaclass=MyMeta, **kwargs):
    # Class body
    x = 10
```

### Beginner Examples

```python
class SimpleClass:
    """A simple class for understanding creation."""
    x = 10
    def method(self):
        return self.x

# What Python actually does:
# 1. Determine metaclass -> type
# 2. Prepare namespace -> {}
# 3. Execute body -> {'x': 10, 'method': <function>, ...}
# 4. Create class -> type('SimpleClass', (), namespace)

# We can inspect the steps
print(SimpleClass.__name__)   # SimpleClass
print(SimpleClass.__bases__)  # (<class 'object'>,)
print(SimpleClass.__dict__)   # {'x': 10, 'method': <function>, ...}
```

### Intermediate Examples

```python
# Custom namespace preparation
class OrderedNamespace(dict):
    def __init__(self):
        super().__init__()
        self._order = []

    def __setitem__(self, key, value):
        if key not in self:
            self._order.append(key)
        super().__setitem__(key, value)

class OrderedMeta(type):
    @classmethod
    def __prepare__(mcs, name, bases):
        return OrderedNamespace()

    def __new__(mcs, name, bases, namespace):
        namespace['_field_order'] = namespace._order[:]
        return super().__new__(mcs, name, bases, dict(namespace))

class OrderedClass(metaclass=OrderedMeta):
    z = 1
    a = 2
    m = 3
    b = 4

print(OrderedClass._field_order)  # ['z', 'a', 'm', 'b']
```

### Advanced Examples

```python
# Full class creation hook with __init_subclass__
class PluginBase:
    plugins = []

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        cls.plugins.append(cls)

class PluginA(PluginBase):
    pass

class PluginB(PluginBase):
    pass

print(PluginBase.plugins)  # [<class 'PluginA'>, <class 'PluginB'>]

# Metaclass controlling instance creation
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Singleton(metaclass=SingletonMeta):
    def __init__(self, value):
        self.value = value

s1 = Singleton(1)
s2 = Singleton(2)
print(s1 is s2)    # True
print(s1.value)    # 1 (not 2)
```

### Real-World Use Cases

- **Django models**: `class User(models.Model):` triggers metaclass to create database schema.
- **SQLAlchemy ORM**: `__tablename__`, `__table_args__` are processed by metaclass.
- **Enum support**: The `Enum` metaclass processes enum members.
- **Abstract base classes**: `ABCMeta` registers abstract methods.
- **Protocol classes**: `typing.Protocol` uses metaclass for structural subtyping.

### Common Mistakes

- Forgetting that `__prepare__` must return a mapping, not necessarily a dict.
- Assuming `metaclass` keyword is passed to `__init__` (it's consumed by class creation).
- Modifying the namespace after class creation and expecting metaclass to see changes.
- Expecting `__init_subclass__` to be called when the parent class is created (it's for subclasses).

### Best Practices

- Use `__init_subclass__` instead of metaclasses when you only need to customize subclass creation.
- Override `__prepare__` only when you need ordered or filtered namespaces.
- Use `super()` properly in metaclass methods to maintain MRO compatibility.
- Keep metaclass logic simple and focused on a single transformation.

### Performance Considerations

The class creation process runs once per class at definition time. `__prepare__` adds slight overhead for each class definition. The namespace dictionary and class body execution are as fast as normal code. Metaclass hooks that run at class creation time don't affect instance creation or method call performance.

### Interview Questions

**Q: What is `__prepare__` in a metaclass?**

A: `__prepare__` is a class method on the metaclass that returns the namespace dict used for class body execution. It's called before the class body executes and can return any mapping. It's useful for ordered namespaces or filtered namespaces.

**Q: How does Python determine which metaclass to use?**

A: Python checks (1) the `metaclass` keyword argument in the class definition, (2) the metaclass of the first base class, (3) `type` if neither is specified. Metaclasses must be subclasses of each other to avoid `TypeError`.

### Coding Challenges

1. Create a metaclass that records the order of method definitions in a class.
2. Implement a metaclass that automatically adds `__repr__` based on `__init__` parameters.
3. Build a metaclass that validates attribute types at class definition time.

### Related Topics

- `type()` function
- Metaclass `__new__` vs `__init__`
- `__init_subclass__` hook
- Class decorators (alternative approach)
- MRO (Method Resolution Order)

## Custom Metaclasses

### What It Is

A custom metaclass is a class that inherits from `type` (or another metaclass) and overrides methods like `__new__`, `__init__`, or `__call__` to customize class or instance creation. Custom metaclasses are defined with `class Meta(type):` and applied to classes via the `metaclass` keyword argument.

### Why It Is Important

Custom metaclasses provide ultimate control over class behavior. They are used by frameworks to implement automatic registration, validation, transformation, and APIs that would be impossible with regular inheritance. While advanced, they solve problems that have no other clean solution.

### How It Works Internally

A custom metaclass inherits from `type`. It overrides `__new__` (called before class creation, receives `mcs`, `name`, `bases`, `namespace`), `__init__` (called after class creation for initialization), and/or `__call__` (called when the class is instantiated). The metaclass is applied by specifying `metaclass=MyMeta` in the class definition.

### Syntax

```python
class CustomMeta(type):
    def __new__(mcs, name, bases, namespace):
        # Modify namespace before class creation
        return super().__new__(mcs, name, bases, namespace)

    def __init__(cls, name, bases, namespace):
        # Initialize the newly created class
        super().__init__(name, bases, namespace)

    def __call__(cls, *args, **kwargs):
        # Called when an instance is created
        return super().__call__(*args, **kwargs)

class MyClass(metaclass=CustomMeta):
    pass
```

### Beginner Examples

```python
class AutoPropertyMeta(type):
    def __new__(mcs, name, bases, namespace):
        for key, value in list(namespace.items()):
            if key.startswith('_') and isinstance(value, (str, int, float)):
                prop_name = key.lstrip('_')
                namespace[prop_name] = property(
                    lambda self, k=key: getattr(self, k),
                    lambda self, v, k=key: setattr(self, k, v)
                )
        return super().__new__(mcs, name, bases, namespace)

class Person(metaclass=AutoPropertyMeta):
    _name = "Unknown"
    _age = 0

p = Person()
print(p.name)  # Unknown
p.name = "Alice"
print(p.name)  # Alice
```

### Intermediate Examples

```python
class RegistryMeta(type):
    registry = {}

    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        if not name.startswith('Base'):
            mcs.registry[name] = cls
        return cls

class BaseHandler(metaclass=RegistryMeta):
    pass

class UserHandler(BaseHandler):
    def handle(self):
        return "User"

class OrderHandler(BaseHandler):
    def handle(self):
        return "Order"

print(RegistryMeta.registry)
# {'UserHandler': <class 'UserHandler'>, 'OrderHandler': <class 'OrderHandler'>}


class LoggerMeta(type):
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        original_init = cls.__init__

        def logged_init(self, *args, **kwargs):
            print(f"Creating {name} instance with args={args}, kwargs={kwargs}")
            original_init(self, *args, **kwargs)

        cls.__init__ = logged_init
        return cls

class Product(metaclass=LoggerMeta):
    def __init__(self, name, price):
        self.name = name
        self.price = price

p = Product("Widget", 9.99)
# Creating Product instance with args=('Widget', 9.99), kwargs={}
```

### Advanced Examples

```python
class ValidatedMeta(type):
    def __new__(mcs, name, bases, namespace):
        annotations = namespace.get('__annotations__', {})
        validators = {}

        for attr_name, attr_type in annotations.items():
            def make_validator(aname, atype):
                def validate(self, value):
                    if not isinstance(value, atype):
                        raise TypeError(f"{aname} must be {atype.__name__}, got {type(value).__name__}")
                    return value
                return validate

            private_name = f'_{attr_name}'

            def make_getter(pname):
                return lambda self: getattr(self, pname, None)

            def make_setter(pname, aname, atype):
                validator = make_validator(aname, atype)
                def setter(self, value):
                    setattr(self, pname, validator(value))
                return setter

            namespace[attr_name] = property(
                make_getter(private_name),
                make_setter(private_name, attr_name, attr_type)
            )

            if '__init__' not in namespace:
                def make_init(attrs):
                    def __init__(self, **kwargs):
                        for k, v in kwargs.items():
                            setattr(self, k, v)
                    return __init__
                namespace['__init__'] = make_init(list(annotations.keys()))

        return super().__new__(mcs, name, bases, namespace)

class User(metaclass=ValidatedMeta):
    name: str
    age: int

u = User(name="Alice", age=30)
print(u.name)  # Alice
# u.name = 42  # TypeError: name must be str, got int
```

### Real-World Use Cases

- **Django's ModelBase metaclass**: Converts `class Meta` inner class and field definitions into database schema.
- **SQLAlchemy's DeclarativeMeta**: Maps class attributes to database columns.
- **ABC (Abstract Base Classes)**: `ABCMeta` registers abstract methods and prevents instantiation.
- **Enum metaclass**: Processes enum members and ensures uniqueness.
- **Singleton metaclasses**: Ensure only one instance exists per class.

### Common Mistakes

- Inheriting from `type` but forgetting `super().__new__()` with correct arguments.
- Using metaclasses for simple tasks achievable with class decorators or `__init_subclass__`.
- Creating metaclasses that don't compose well with other metaclasses (metaclass conflicts).
- Modifying the namespace dict in `__new__` after calling `super().__new__()`.
- Expecting `__init__` to receive the same `kwargs` as the class definition (it doesn't get `metaclass`).

### Best Practices

- Prefer class decorators or `__init_subclass__` over metaclasses when possible.
- If using metaclasses, follow the template method pattern: override `__new__` for creation-time modifications and `__init__` for post-creation setup.
- Document metaclass behavior with clear examples — metaclasses are hard to debug.
- Handle metaclass conflicts by creating a combined metaclass when multiple metaclasses are needed.
- Keep metaclass logic independent and testable in isolation.

### Performance Considerations

Custom metaclass overhead occurs at class definition time. Each `__new__` and `__init__` call happens once per class. The `__call__` method, when overridden, adds overhead to every instance creation. Measure instance creation performance if the metaclass's `__call__` does expensive operations.

### Interview Questions

**Q: When would you use a metaclass instead of a class decorator?**

A: When you need to control class creation at the point where the class is defined (before the class object exists), when you need to modify the class namespace before the class object is built, or when you want the customization to be inherited without an explicit decorator on every subclass.

**Q: What is metaclass conflict and how do you resolve it?**

A: A metaclass conflict occurs when a class inherits from multiple base classes with different (unrelated) metaclasses. Python raises `TypeError`. It's resolved by creating a new metaclass that inherits from all conflicting metaclasses, or by restructuring the inheritance hierarchy.

### Coding Challenges

1. Implement a `FinalMeta` metaclass that prevents a class from being subclassed.
2. Create a `TimerMeta` that automatically logs execution time of all methods in a class.
3. Build a `SerializableMeta` that adds `to_dict()` and `from_dict()` methods based on `__init__` parameters.
4. Implement an `InterfaceMeta` that ensures all methods defined in a base "interface" class are implemented in subclasses.

### Related Topics

- `type()` built-in function
- `__new__` method
- `__init_subclass__` hook
- Class decorators
- ABC (Abstract Base Classes)
- MRO (Method Resolution Order)
