# Descriptors - __get__, __set__, __delete__, descriptor protocol

## Introduction

Descriptors are Python objects that define how attribute access is managed on other objects. By implementing `__get__`, `__set__`, and/or `__delete__` methods, descriptor objects can intercept attribute lookup, assignment, and deletion. Descriptors are the mechanism behind properties, methods, `@staticmethod`, `@classmethod`, and `__slots__`. They are a fundamental part of Python's object-oriented implementation.

## __get__ Method

### What It Is

`__get__` is a method that defines what happens when the descriptor's attribute is accessed (read). It is called when you retrieve the descriptor value from an instance or a class. The method receives the instance (`obj`) and the class (`objtype`) as arguments.

### Why It Is Important

The `__get__` method enables computed attributes, lazy evaluation, validation on read, and different behavior for instance vs. class access. It is the core of Python's property mechanism and is essential for creating reusable attribute management patterns.

### How It Works Internally

When Python encounters `instance.attr`, it looks up `attr` on the instance's class. If the class has a descriptor for `attr` (an object with `__get__`), Python calls `descriptor.__get__(instance, type(instance))`. If the descriptor also has `__set__` or `__delete__`, it's a data descriptor and takes priority over instance `__dict__`. If it only has `__get__` (non-data descriptor), the instance `__dict__` takes priority.

### Syntax

```python
class Descriptor:
    def __get__(self, obj, objtype=None):
        return value
```

### Beginner Examples

```python
class PositiveNumber:
    def __get__(self, obj, objtype=None):
        return obj.__dict__.get(self.name, 0)

    def __set_name__(self, owner, name):
        self.name = name

class Order:
    price = PositiveNumber()
    quantity = PositiveNumber()

    def __init__(self, price, quantity):
        self.price = price
        self.quantity = quantity

order = Order(10, 5)
print(order.price)  # 10
print(Order.price)  # <__main__.PositiveNumber object>
```

### Intermediate Examples

```python
class LazyProperty:
    def __init__(self, func):
        self.func = func
        self.name = func.__name__

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        value = self.func(obj)
        obj.__dict__[self.name] = value  # Cache in instance dict
        return value

class DataProcessor:
    def __init__(self, data):
        self.data = data

    @LazyProperty
    def processed(self):
        print("Processing data...")
        return [x * 2 for x in self.data]

dp = DataProcessor([1, 2, 3])
print(dp.processed)  # Processing data... [2, 4, 6]
print(dp.processed)  # [2, 4, 6] (from cache, no processing)
```

### Advanced Examples

```python
class ValidatedAttribute:
    def __init__(self, validator):
        self.validator = validator
        self.name = None

    def __set_name__(self, owner, name):
        self.name = f'_{name}'

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.name, None)

    def __set__(self, obj, value):
        self.validator(value)
        setattr(obj, self.name, value)

class Aged:
    age = ValidatedAttribute(lambda v: (
        isinstance(v, int) and 0 <= v <= 150
    ) or (_ for _ in ()).throw(ValueError(f"Invalid age: {v}")))

    def __init__(self, age):
        self.age = age

a = Aged(25)
print(a.age)  # 25
# a.age = 200  # ValueError: Invalid age: 200
```

### Real-World Use Cases

- **ORM field descriptors**: SQLAlchemy and Django use descriptors to track field access.
- **Lazy loading**: Load expensive attributes only when first accessed.
- **Validation**: Validate attribute values on read.
- **Unit conversion**: Automatically convert units when reading attributes.
- **Access logging**: Log every read of sensitive attributes.

### Common Mistakes

- Forgetting to handle `obj is None` (class-level access) in `__get__`.
- Caching in `__get__` without considering that `obj.__dict__` might already have the key.
- Implementing `__get__` without `__set__` (non-data descriptor) — instance dict will shadow it.
- Expecting `__get__` to be called for every access when `__set__` is not defined.

### Best Practices

- Always handle `obj is None` in `__get__` to return the descriptor itself for class access.
- Use `__set_name__` to capture the attribute name from the class definition (Python 3.6+).
- Use `__get__` for read-only computed attributes. Combine with `__set__` for mutable descriptors.
- Prefer `property` for simple cases; use descriptors for reusable, parameterized behavior.

### Performance Considerations

Descriptor lookups involve one extra Python function call per attribute access. This is fast but can add up in tight loops. For hot paths, consider inlining access or caching the descriptor result. The `__get__` method's performance depends on its implementation complexity.

### Interview Questions

**Q: What is the difference between a data descriptor and a non-data descriptor?**

A: A data descriptor implements `__set__` or `__delete__` (or both). A non-data descriptor only implements `__get__`. Data descriptors take precedence over instance `__dict__`; non-data descriptors do not.

**Q: What happens when `__get__` returns `self`?**

A: When `obj` is `None` (class-level access), `__get__` can return `self` to allow class-level attribute access. This is how unbound methods work — `Class.method` returns the function, not a bound method.

### Coding Challenges

1. Implement a `ReadOnly` descriptor that prevents setting an attribute after initialization.
2. Create a `LoggedAttribute` descriptor that logs every read and write with timestamps.
3. Implement a `Default` descriptor that returns a default value if the attribute hasn't been set.
4. Build a `Computed` descriptor that recomputes its value when dependencies change.

### Related Topics

- `property` built-in (function-based descriptor)
- `__set__` and `__delete__` methods
- `__set_name__` protocol
- `__getattribute__` vs `__getattr__`
- Slots (`__slots__` uses descriptors internally)

## __set__ Method

### What It Is

`__set__` is the method that defines what happens when a value is assigned to a descriptor-managed attribute. It intercepts assignment operations, allowing validation, type checking, transformation, logging, or rejection of the assigned value.

### Why It Is Important

`__set__` enables controlled attribute mutation, which is essential for data validation, type enforcement, change notification, and immutability patterns. Combined with `__get__`, it provides full control over attribute lifecycle.

### How It Works Internally

When `instance.attr = value` is executed, Python checks if `attr` is a data descriptor on the class (has `__set__`). If so, it calls `descriptor.__set__(instance, value)`. If not, it stores the value directly in `instance.__dict__`. This check happens during attribute setting, before any instance dictionary lookup.

### Syntax

```python
class Descriptor:
    def __set__(self, obj, value):
        # Validation, transformation, etc.
        obj.__dict__[self.name] = value
```

### Beginner Examples

```python
class IntegerField:
    def __set_name__(self, owner, name):
        self.name = f'_{name}'

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.name, 0)

    def __set__(self, obj, value):
        if not isinstance(value, int):
            raise TypeError(f"Expected int, got {type(value).__name__}")
        setattr(obj, self.name, value)

class Product:
    price = IntegerField()
    stock = IntegerField()

    def __init__(self, price, stock):
        self.price = price
        self.stock = stock

p = Product(10, 100)
print(p.price)   # 10
# p.price = "expensive"  # TypeError: Expected int, got str
```

### Intermediate Examples

```python
class RangeField:
    def __init__(self, min_val, max_val):
        self.min_val = min_val
        self.max_val = max_val

    def __set_name__(self, owner, name):
        self.name = f'_{name}'

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.name, self.min_val)

    def __set__(self, obj, value):
        if not isinstance(value, (int, float)):
            raise TypeError(f"Expected number, got {type(value).__name__}")
        if not self.min_val <= value <= self.max_val:
            raise ValueError(f"Value {value} outside range [{self.min_val}, {self.max_val}]")
        setattr(obj, self.name, value)

class Temperature:
    celsius = RangeField(-273.15, 10000)

    def __init__(self, celsius=0):
        self.celsius = celsius

t = Temperature(25)
print(t.celsius)  # 25
# t.celsius = -300  # ValueError: Value -300 outside range [-273.15, 10000]
```

### Advanced Examples

```python
class ObservableField:
    def __init__(self, callback=None):
        self.callback = callback

    def __set_name__(self, owner, name):
        self.name = f'_{name}'
        self.public_name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.name, None)

    def __set__(self, obj, value):
        old_value = self.__get__(obj)
        setattr(obj, self.name, value)
        if self.callback:
            self.callback(obj, self.public_name, old_value, value)

class Person:
    name = ObservableField()
    age = ObservableField()

    def __init__(self, name, age):
        self.name = name
        self.age = age

def on_change(obj, attr, old, new):
    print(f"{type(obj).__name__}.{attr} changed: {old} -> {new}")

Person.name = ObservableField(on_change)
Person.age = ObservableField(on_change)

p = Person("Alice", 30)
p.name = "Bob"   # Person.name changed: Alice -> Bob
p.age = 31       # Person.age changed: 30 -> 31
```

### Real-World Use Cases

- **ORM fields**: Track dirty state for database updates.
- **Validated config**: Ensure configuration values are within allowed ranges.
- **Data synchronization**: Trigger UI updates or network sync when data changes.
- **History/undo tracking**: Record attribute changes for undo support.
- **Immutable fields**: Raise `AttributeError` on modification attempts.

### Common Mistakes

- Not storing the value in the instance dict (must do `obj.__dict__[name] = value` or similar).
- Forgetting to convert or validate the value before storing.
- Raising exceptions in `__set__` that are inappropriate for the application context.
- Using `obj.attr = value` inside `__set__` (infinite recursion).
- Not handling the case where `__set__` is called before any `__get__`.

### Best Practices

- Always use `__set_name__` to store the attribute name to avoid hardcoded names.
- Store values in `obj.__dict__` or via `object.__setattr__` to avoid recursive calls.
- Validate early in `__set__` to fail fast with clear error messages.
- Consider calling a callback or notifying observers in `__set__` for reactive patterns.

### Performance Considerations

`__set__` adds function call overhead to every attribute assignment. For performance-critical code, minimize descriptor use in hot paths. The overhead is one extra function call per assignment — significant only in tight loops with millions of iterations.

### Interview Questions

**Q: How does a data descriptor differ from a property?**

A: `property` is a built-in data descriptor implemented in C. Custom descriptors are Python classes implementing `__get__`/`__set__`/`__delete__`. Properties are simpler for single-use; descriptors are reusable across classes.

**Q: Can a descriptor prevent setting an attribute?**

A: Yes. `__set__` can raise `AttributeError` to prevent setting, acting as a read-only descriptor. However, users can still bypass it via `object.__setattr__`.

### Coding Challenges

1. Implement an `ImmutableAfterInit` descriptor that prevents changes after `__init__`.
2. Create a `HistoryField` descriptor that keeps a list of all previous values.
3. Build a `SyncedField` descriptor that sends updates to a server when a value changes.
4. Implement a `EncryptedField` descriptor that encrypts values on set and decrypts on get.

### Related Topics

- `__get__` method (the read counterpart)
- `__delete__` method (the delete counterpart)
- `property` (built-in descriptor for common cases)
- `__set_name__` (naming protocol)
- `__slots__` (descriptor-based attribute restriction)

## __delete__ Method

### What It Is

`__delete__` is the method that defines what happens when a descriptor-managed attribute is deleted with the `del` statement. It intercepts deletion, allowing custom cleanup, logging, or prevention of deletion.

### Why It Is Important

`__delete__` provides control over attribute removal, which is important for cleanup operations, resource management, and enforcing invariants. Without it, deleting a descriptor-managed attribute would either fail or silently remove it from the instance dictionary, bypassing any management logic.

### How It Works Internally

When `del instance.attr` is executed, Python checks if `attr` is a data descriptor on the class. If it has `__delete__`, Python calls `descriptor.__delete__(instance)`. If not, it removes `attr` from `instance.__dict__` (raising `AttributeError` if not found).

### Syntax

```python
class Descriptor:
    def __delete__(self, obj):
        # Cleanup logic
        del obj.__dict__[self.name]
```

### Beginner Examples

```python
class ProtectedField:
    def __set_name__(self, owner, name):
        self.name = f'_{name}'

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.name, None)

    def __set__(self, obj, value):
        setattr(obj, self.name, value)

    def __delete__(self, obj):
        raise AttributeError(f"Cannot delete {self.name[1:]}")

class Config:
    api_key = ProtectedField()

    def __init__(self, api_key):
        self.api_key = api_key

cfg = Config("secret-123")
print(cfg.api_key)  # secret-123
# del cfg.api_key    # AttributeError: Cannot delete api_key
```

### Intermediate Examples

```python
class LoggedDeletion:
    def __set_name__(self, owner, name):
        self.name = f'_{name}'
        self.public_name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.name, None)

    def __set__(self, obj, value):
        setattr(obj, self.name, value)

    def __delete__(self, obj):
        old_value = self.__get__(obj)
        print(f"Deleting {self.public_name} (was: {old_value})")
        setattr(obj, self.name, None)

class Session:
    user = LoggedDeletion()
    token = LoggedDeletion()

    def __init__(self, user, token):
        self.user = user
        self.token = token

s = Session("Alice", "abc123")
del s.user     # Deleting user (was: Alice)
del s.token    # Deleting token (was: abc123)
```

### Advanced Examples

```python
class ResourceField:
    def __set_name__(self, owner, name):
        self.name = f'_{name}'

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.name, None)

    def __set__(self, obj, value):
        old = self.__get__(obj)
        if old is not None and hasattr(old, 'close'):
            old.close()
        setattr(obj, self.name, value)

    def __delete__(self, obj):
        value = self.__get__(obj)
        if value is not None:
            if hasattr(value, 'close'):
                value.close()
            setattr(obj, self.name, None)

class ConnectionManager:
    connection = ResourceField()

    def __init__(self, conn):
        self.connection = conn

# cm = ConnectionManager(open("file.txt"))
# del cm.connection  # Automatically closes the file
```

### Real-World Use Cases

- **Protected configuration**: Prevent deletion of critical configuration values.
- **Resource cleanup**: Automatically release resources (files, connections) when deleted.
- **Reference counting**: Track when attributes are removed for cache invalidation.
- **State machines**: Prevent deletion of attributes in invalid states.
- **Database relationships**: Cascade delete or nullify related records.

### Common Mistakes

- Forgetting to call `super().__delete__()` in subclassed descriptors.
- Deleting from `obj.__dict__` directly without considering `__slots__`.
- Not checking if the attribute exists before trying to delete it.
- Raising `AttributeError` in `__delete__` when the attribute doesn't exist (confusing).

### Best Practices

- Implement `__delete__` only when deletion needs special handling (cleanup, protection, logging).
- For simple read-only attributes, raise `AttributeError` with a clear message.
- Always clean up external resources (file handles, connections) in `__delete__`.
- Document whether an attribute can be deleted and what side effects deletion has.

### Performance Considerations

`__delete__` is called once per `del` operation. Its performance is dominated by whatever cleanup logic it contains. For simple operations (setting to None), it's equivalent to a regular attribute delete.

### Interview Questions

**Q: What happens if you `del` an attribute managed by a descriptor without `__delete__`?**

A: If the descriptor is a non-data descriptor, `del` deletes from `instance.__dict__`. If it's a data descriptor without `__delete__`, Python raises `AttributeError`.

**Q: How do you implement a one-time-set descriptor?**

A: Implement `__set__` to raise `AttributeError` after the first set, and `__delete__` to reset the flag, allowing re-setting.

### Coding Challenges

1. Implement a `TemporaryField` descriptor that warns when deleted and allows it only once.
2. Create a `CascadeDelete` descriptor that deletes related objects when the attribute is deleted.
3. Build a `UndoableField` descriptor that supports undo through delete.

### Related Topics

- `__get__` and `__set__` (the other descriptor methods)
- `del` statement
- `__delattr__` magic method
- `__slots__` (uses descriptors internally)

## Descriptor Protocol

### What It Is

The descriptor protocol is the specification that defines how Python objects can intercept attribute access. Any object implementing one or more of `__get__`, `__set__`, and `__delete__` is called a descriptor. The protocol determines attribute lookup priority, binding behavior, and the interaction between descriptors and instance dictionaries.

### Why It Is Important

The descriptor protocol is the foundation of Python's attribute access system. Properties, methods, `@staticmethod`, `@classmethod`, and `super()` all rely on descriptors. Understanding the protocol is essential for mastering Python's OOP model and for creating reusable attribute management abstractions.

### How It Works Internally

The descriptor protocol is implemented in CPython's `object.__getattribute__`. For attribute access `obj.attr`:

1. Look for `attr` in the class MRO. If found and it's a data descriptor (has `__set__` or `__delete__`), call `descriptor.__get__(obj, type(obj))`.
2. If not a data descriptor, check `obj.__dict__` for `attr`.
3. If not in `obj.__dict__`, look in the class MRO. If found and it's a non-data descriptor (has only `__get__`), call it. Otherwise return the value directly.
4. If not found, raise `AttributeError`.

### Syntax

```python
class Descriptor:
    def __get__(self, obj, objtype=None): ...
    def __set__(self, obj, value): ...
    def __delete__(self, obj): ...
```

### Beginner Examples

```python
# Demonstrating the priority rules
class NonDataDescriptor:
    """Only has __get__ - non-data descriptor."""
    def __get__(self, obj, objtype=None):
        return "NON-DATA DESCRIPTOR"

class DataDescriptor:
    """Has __get__ and __set__ - data descriptor."""
    def __get__(self, obj, objtype=None):
        return "DATA DESCRIPTOR"
    def __set__(self, obj, value):
        raise AttributeError("Read-only")

class Demo:
    non_data = NonDataDescriptor()
    data = DataDescriptor()

d = Demo()
print(d.non_data)  # NON-DATA DESCRIPTOR
print(d.data)      # DATA DESCRIPTOR

# Instance dict overrides non-data descriptor
d.non_data = "instance value"
print(d.non_data)  # instance value (instance dict wins)

# Instance dict does NOT override data descriptor
try:
    d.data = "instance value"
except AttributeError:
    print("Data descriptor prevents set")

# Delete instance dict entry to reveal descriptor again
del d.non_data
print(d.non_data)  # NON-DATA DESCRIPTOR
```

### Intermediate Examples

```python
# Implementing function binding with descriptors
class Function:
    """Simplified function descriptor (like Python's function)."""
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self  # Class access, return unbound
        # Instance access, return bound method
        def bound(*args, **kwargs):
            return self(obj, *args, **kwargs)
        return bound

# Actually Python's method binding is more sophisticated:
class MyClass:
    def method(self):
        return "called"

obj = MyClass()
print(MyClass.method)   # <function MyClass.method at ...>
print(obj.method)       # <bound method ...>

# The property built-in is a data descriptor
class WithProperty:
    _value = 10

    @property
    def value(self):
        return self._value

    @value.setter
    def value(self, val):
        self._value = val

    @value.deleter
    def value(self):
        del self._value
```

### Advanced Examples

```python
# Descriptor that delegates to a method
class cached_property:
    """Simplified cached_property implementation."""
    def __init__(self, func):
        self.func = func
        self.name = func.__name__

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        value = self.func(obj)
        obj.__dict__[self.name] = value
        return value

    def __set__(self, obj, value):
        raise AttributeError("Cannot set cached property")

    def __delete__(self, obj):
        obj.__dict__.pop(self.name, None)

class DataAnalyzer:
    def __init__(self, data):
        self.data = data

    @cached_property
    def statistics(self):
        import time
        time.sleep(0.1)  # Expensive computation
        return {
            'mean': sum(self.data) / len(self.data),
            'max': max(self.data),
            'min': min(self.data)
        }

da = DataAnalyzer([1, 2, 3, 4, 5])
print(da.statistics)  # Computed
print(da.statistics)  # From cache (no delay)
del da.statistics
print(da.statistics)  # Re-computed
```

### Real-World Use Cases

- **Property system**: `@property` is the most common descriptor usage.
- **Method binding**: Functions are descriptors — `obj.method` creates a bound method.
- **Static/class methods**: `staticmethod` and `classmethod` are descriptor classes.
- **ORM systems**: SQLAlchemy, Django use descriptors for field tracking.
- **Validation frameworks**: Pydantic uses descriptors for data validation.
- **Slots**: `__slots__` creates descriptors for each slot name.

### Common Mistakes

- Confusing data and non-data descriptors (instance dict shadowing behavior differs).
- Forgetting that `__get__` is called for class-level access too (check `obj` for `None`).
- Implementing descriptors without using `__set_name__` (Python 3.6+) to get the attribute name.
- Not understanding that `staticmethod` and `classmethod` are descriptors.
- Creating descriptors that are unaware of the descriptor protocol's lookup priority.

### Best Practices

- Use `property` for simple attribute management; use custom descriptors for reusable patterns.
- Always implement `__set_name__` to capture the attribute name.
- Understand the data vs. non-data descriptor distinction before deciding which to implement.
- Use descriptors for cross-cutting attribute concerns (logging, validation, type checking).
- Document whether a descriptor is meant to be a data or non-data descriptor.

### Performance Considerations

The descriptor protocol's overhead comes from the attribute lookup chain (`__getattribute__`). Each descriptor access involves at least one extra Python function call. CPython optimizes common descriptors (property, function) at the C level. Custom Python-level descriptors are slower but still fast enough for most applications.

### Interview Questions

**Q: Explain the attribute lookup priority in Python.**

A: (1) Data descriptors from the class MRO. (2) Instance `__dict__`. (3) Non-data descriptors from the class MRO. (4) Class `__dict__` values. (5) If not found, `__getattr__` is called (if defined). If still not found, `AttributeError`.

**Q: How do `@staticmethod` and `@classmethod` use the descriptor protocol?**

A: Both are descriptor classes. `staticmethod.__get__` returns the underlying function unchanged (no binding). `classmethod.__get__` binds the method to the class (not the instance), passing `cls` as the first argument.

### Coding Challenges

1. Implement a simplified `property` descriptor from scratch.
2. Create a `typed_field(type)` descriptor that only accepts values of a specific type.
3. Build a `history_tracker` descriptor that records every change to the attribute.
4. Implement a `default_factory` descriptor that calls a function if the attribute hasn't been set.

### Related Topics

- `property` built-in function
- `__getattribute__` and `__getattr__`
- `__set_name__` protocol
- Function descriptors and bound methods
- `staticmethod` and `classmethod`
- `__slots__` (uses per-name descriptors)
