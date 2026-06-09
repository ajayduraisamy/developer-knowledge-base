# Encapsulation - Public, protected (_), private (__) attributes

## Introduction

**Encapsulation** is the practice of bundling data (attributes) and methods (functions) that operate on that data within a single unit (a class), while restricting direct access to some of the object's internal components. This protects an object's integrity by preventing external code from modifying its internal state in unexpected ways.

Python implements encapsulation through naming conventions — there are no true `private` keywords as in Java or C++. Instead, the language relies on **conventions** and **name mangling** to signal intent.

## Why It Is Important

- **Data protection**: Prevent accidental or malicious modification of internal state.
- **Controlled access**: Provide getters/setters or properties to validate changes.
- **Reduced complexity**: Hide implementation details behind a clean public API.
- **Maintainability**: Internal changes don't affect external code as long as the public interface stays stable.
- **Testing**: Well-encapsulated classes are easier to test in isolation.

## Syntax

```python
class Encapsulated:
    def __init__(self):
        self.public = "anyone can see this"
        self._protected = "subclasses can see this (convention)"
        self.__private = "name-mangled to _Encapsulated__private"

    def get_private(self):
        return self.__private

    def set_private(self, value):
        self.__private = value
```

## Examples

### Access levels demonstration

```python
class Demo:
    def __init__(self):
        self.public = "public"
        self._protected = "protected"
        self.__private = "private"

    def show_private(self):
        return self.__private


d = Demo()
print(d.public)          # public
print(d._protected)      # protected (works, but discouraged)
# print(d.__private)     # AttributeError!
print(d._Demo__private)  # private (name mangling workaround)
print(d.show_private())  # private (proper access)
```

## Beginner Examples

### 1. Bank account with controlled balance

```python
class BankAccount:
    def __init__(self, owner, initial_balance=0):
        self.owner = owner
        self.__balance = initial_balance   # private

    def deposit(self, amount):
        if amount <= 0:
            return "Amount must be positive"
        self.__balance += amount
        return self.__balance

    def withdraw(self, amount):
        if amount <= 0:
            return "Amount must be positive"
        if amount > self.__balance:
            return "Insufficient funds"
        self.__balance -= amount
        return self.__balance

    def get_balance(self):
        return self.__balance


acct = BankAccount("Alice", 1000)
print(acct.get_balance())   # 1000
print(acct.deposit(500))    # 1500
print(acct.withdraw(2000))  # Insufficient funds
# acct.__balance = 999999   # creates new attribute, doesn't affect real balance
# print(acct.get_balance()) # still 1500
```

### 2. Protected attribute in inheritance

```python
class Base:
    def __init__(self):
        self._secret = "base secret"

    def reveal(self):
        return self._secret


class Derived(Base):
    def __init__(self):
        super().__init__()
        self._secret = "derived secret"  # allowed (convention)

    def double_reveal(self):
        return self._secret * 2


d = Derived()
print(d.reveal())         # derived secret
print(d._secret)          # derived secret (accessible but discouraged)
```

### 3. Property-based encapsulation (simple)

```python
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = celsius

    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Below absolute zero!")
        self._celsius = value

    @property
    def fahrenheit(self):
        return self._celsius * 9 / 5 + 32


t = Temperature(100)
print(t.celsius)      # 100
print(t.fahrenheit)   # 212.0
t.celsius = 0
print(t.fahrenheit)   # 32.0
# t.celsius = -300    # ValueError!
```

## Intermediate Examples

### 1. Name mangling to avoid subclass clashes

```python
class Parent:
    def __init__(self):
        self.__value = "parent"

    def get_value(self):
        return self.__value


class Child(Parent):
    def __init__(self):
        super().__init__()
        self.__value = "child"  # _Child__value, doesn't clash!

    def get_child_value(self):
        return self.__value

    def get_parent_value_via_mangling(self):
        return self._Parent__value


c = Child()
print(c.get_value())             # parent
print(c.get_child_value())       # child
print(c.get_parent_value_via_mangling())  # parent
```

### 2. Read-only attribute via property

```python
class User:
    def __init__(self, username, email):
        self._username = username
        self._email = email

    @property
    def username(self):
        return self._username

    @property
    def email(self):
        return self._email

    @email.setter
    def email(self, value):
        if "@" not in value:
            raise ValueError("Invalid email")
        self._email = value


u = User("alice", "alice@example.com")
print(u.username)       # alice
# u.username = "bob"    # AttributeError: can't set attribute
u.email = "alice@new.com"
print(u.email)          # alice@new.com
```

### 3. Validation via name mangling

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.__price = 0
        self.price = price  # use setter

    @property
    def price(self):
        return self.__price

    @price.setter
    def price(self, value):
        if value < 0:
            raise ValueError("Price cannot be negative")
        self.__price = value

    def apply_discount(self, percent):
        self.__price *= (1 - percent / 100)


p = Product("Widget", 49.99)
print(p.price)            # 49.99
p.apply_discount(10)
print(f"{p.price:.2f}")   # 44.99
```

## Advanced Examples

### 1. Property with lazy initialisation

```python
class DatabaseConnection:
    def __init__(self, connection_string):
        self._connection_string = connection_string
        self._connection = None

    @property
    def connection(self):
        if self._connection is None:
            print("Establishing connection...")
            self._connection = self._connect()
        return self._connection

    def _connect(self):
        # Simulate connection
        return f"Connected to {self._connection_string}"

    def query(self, sql):
        conn = self.connection
        return f"Executing '{sql}' on {conn}"


db = DatabaseConnection("postgres://localhost/mydb")
print(db.query("SELECT 1"))
# Establishing connection...
# Executing 'SELECT 1' on Connected to postgres://localhost/mydb
print(db.query("SELECT 2"))
# (no "Establishing" — already connected)
```

### 2. Descriptor-based validation

```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self.private_name = f"_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name, 0)

    def __set__(self, obj, value):
        if not isinstance(value, (int, float)):
            raise TypeError("Must be a number")
        if value <= 0:
            raise ValueError("Must be positive")
        setattr(obj, self.private_name, value)


class Order:
    quantity = PositiveNumber()
    unit_price = PositiveNumber()

    def __init__(self, quantity, unit_price):
        self.quantity = quantity
        self.unit_price = unit_price

    def total(self):
        return self.quantity * self.unit_price


o = Order(5, 19.99)
print(o.total())     # 99.95
# o.quantity = -1    # ValueError: Must be positive
```

### 3. Property with caching (functools.cached_property)

```python
from functools import cached_property
import time


class ExpensiveReport:
    def __init__(self, data):
        self.data = data
        self._generated = False

    @cached_property
    def summary(self):
        print("Generating summary (expensive)...")
        time.sleep(0.1)  # simulate computation
        result = {"count": len(self.data), "sum": sum(self.data)}
        self._generated = True
        return result

    @property
    def generated(self):
        return self._generated


r = ExpensiveReport([1, 2, 3, 4, 5])
print(r.summary)    # Generating summary... then shows dict
print(r.summary)    # Instant — cached!
print(r.generated)  # True
```

## Real-World Use Cases

- **ORM models**: Django models use `_` prefix for internal fields; properties expose related objects.
- **Configuration classes**: Settings are loaded once and exposed via read-only properties.
- **API clients**: API keys stored as private attributes, never logged.
- **GUI components**: Internal widget state (position, size) protected; public methods guarantee valid transitions.
- **Financial systems**: Account balances hidden behind transactions that validate every change.

## Common Mistakes

| Mistake | Why it's wrong | Fix |
|---|---|---|
| Using double underscore for every attribute | Creates unnecessary name mangling; breaks subclasses | Use single `_` for protected; `__` only to avoid subclass collisions |
| Expecting `__` to be truly private | It's name-mangled, not private; still accessible | Accept Python's convention-based approach |
| Writing explicit getters/setters instead of `@property` | Verbose, not Pythonic | Use `@property` and `@attr.setter` |
| Exposing mutable internal objects directly | Caller can modify internal state | Return a copy or a read-only proxy |
| Forgetting to call `super().__init__()` | Private attributes from parent namespace may not be set up | Always call `super().__init__()` |

```python
# Exposing mutable internals — BAD
class BadConfig:
    def __init__(self):
        self._settings = {"debug": True}

    def get_settings(self):
        return self._settings  # caller can mutate!

# Use copy — GOOD
class GoodConfig:
    def __init__(self):
        self._settings = {"debug": True}

    def get_settings(self):
        return self._settings.copy()

    def get(self, key):
        return self._settings.get(key)
```

## Best Practices

- Follow the naming convention: **public** → `attr`, **protected** → `_attr`, **private** → `__attr`.
- Use `@property` instead of explicit `get_attr()` / `set_attr()` methods.
- Keep public API as **small** as possible — expose only what's necessary.
- Create **read-only properties** by omitting the setter.
- Return **copies or immutable proxies** of internal mutable objects.
- Do **not** overuse `__name_mangling`; it complicates debugging and inheritance.
- Validate data in setters/properties, not in the calling code.

## Interview Questions

1. **How does Python implement encapsulation compared to Java?**
   *Python uses naming conventions (`_` and `__` prefix) rather than access modifier keywords. It's based on trust and convention rather than compiler enforcement.*

2. **What is name mangling and when would you use it?**
   *Python transforms `__attr` to `_ClassName__attr` to avoid accidental overrides in subclasses. Use it when designing a class intended for inheritance and you want to avoid attribute name clashes.*

3. **What is the difference between `_protected` and `__private`?**
   *`_protected` is a convention meaning "internal use only." `__private` triggers name mangling and is intended to avoid subclass attribute collisions.*

4. **Explain the property decorator and its use in encapsulation.**
   *`@property` turns a method into a read-only attribute accessor. It allows computation, validation, and lazy loading while keeping the interface looking like a simple attribute.*

5. **Can you make an attribute truly private in Python?**
   *No — all attributes can be accessed if someone knows the mangled name. Python relies on convention and trust.*

6. **What is the difference between `cached_property` and `property`?**
   *`cached_property` computes the value once and caches it for the lifetime of the instance. `property` recomputes on every access unless you implement caching yourself.*

## Coding Challenges

1. **Secure Voting System**: `VotingBooth` with private `_votes` dict. Only allows `vote(candidate)` and `get_results()`. No direct access to vote data.

2. **Immutable Point**: `ImmutablePoint` with `x` and `y` as read-only properties (only settable via `__init__`). Implement `with_x(new_x)` that returns a new instance.

3. **Password Manager**: `PasswordVault` stores passwords in a private dict. Provides `get_password(service)`, `set_password(service, password)`. Log all access. Never expose the raw dict.

4. **Validation Decorator**: Create a `@validated` decorator for properties that accepts a validation function. Apply to a `Person` class with `age` (0–150) and `email` (must contain `@`).

5. **Proxy Pattern**: `ProtectedDict` wraps a dict and only allows access through `get(key)` and `set(key, value)`. Log all read/write operations. Prevent deletion.

## Summary

- Encapsulation bundles data and methods; it restricts direct access to an object's internals.
- **No true `private`** in Python — conventions and name mangling signal intent.
- `_attr` = protected (internal + subclass use), `__attr` = name-mangled (avoids subclass clashes).
- **Properties** (`@property`) are the Pythonic way to implement getters/setters with validation.
- Return **copies** of mutable internal objects to prevent accidental modification.
- Keep the **public interface minimal** and well-documented.

## Related Topics

- Classes and Objects — the encapsulation unit
- Properties — Pythonic getters/setters
- Inheritance — name mangling exists primarily for inheritance safety
- Polymorphism — encapsulated internals behind a polymorphic interface
- Abstraction — hiding implementation details behind abstract interfaces
- Magic Methods — dunder methods often access private attributes
