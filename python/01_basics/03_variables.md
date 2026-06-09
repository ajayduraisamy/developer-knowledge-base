# Variables - Assigning values with =, dynamic typing, naming rules

## Introduction

Variables in Python are names that refer to objects in memory. Unlike statically-typed languages where variables are memory containers with fixed types, Python variables are best understood as "name tags" attached to objects. The assignment operator `=` binds a name to an object. Python's dynamic typing means a variable can refer to any type of object, and its type can change during execution. Naming rules define what constitutes a valid variable name. Understanding these concepts is fundamental to mastering Python's object model.

## Assigning Values with =

### What It Is
The `=` operator in Python is an assignment operator that binds a name (the variable) to an object (the value). Python evaluates the right-hand side first, then binds the name to the resulting object. Multiple assignment and tuple unpacking allow binding multiple names in a single statement.

### Why It Is Important
Assignment is how data flows through programs. Understanding that Python uses reference semantics (not value semantics) is critical for avoiding bugs with mutable objects. Python's assignment syntax enables elegant patterns like tuple unpacking, swapping without a temporary variable, and star expressions.

### How It Works Internally
Assignment in CPython consists of three operations: (1) the right-hand expression is evaluated via `PyEval_EvalFrame`, producing a reference to a `PyObject`, (2) the name is stored in the current namespace's dictionary (`locals()`) via `PyDict_SetItem`, (3) the reference count of the object is incremented. For mutable objects, in-place modification does not trigger a new assignment — the reference remains the same.

### Syntax
```python
# Basic assignment
name = "Alice"
age = 30

# Multiple assignment (same value)
x = y = z = 0

# Tuple unpacking
a, b, c = 1, 2, 3

# Swapping
a, b = b, a

# Star expression (Python 3.0+)
first, *middle, last = [1, 2, 3, 4, 5]

# Augmented assignment
count += 1   # count = count + 1
value *= 2   # value = value * 2
```

### Beginner Examples
```python
# Basic variable assignment
username = "john_doe"
score = 95
is_active = True

print(f"User: {username}, Score: {score}, Active: {is_active}")

# Variable reassignment
temperature = 25
print(temperature)  # 25
temperature = 30    # Now refers to a different object
print(temperature)  # 30

# Multiple assignment styles
a = b = c = 42
print(a, b, c)  # 42 42 42

# Tuple unpacking
x, y = 10, 20
print(x, y)  # 10 20
```

### Intermediate Examples
```python
# Star expression unpacking
first, *rest = [1, 2, 3, 4, 5]
print(f"First: {first}, Rest: {rest}")  # First: 1, Rest: [2, 3, 4, 5]

head, *body, tail = range(10)
print(f"Head: {head}, Body: {body}, Tail: {tail}")

# Swapping without temp
a, b = 5, 10
a, b = b, a
print(f"Swapped: a={a}, b={b}")  # a=10, b=5

# Unpacking nested structures
data = ("Alice", 30, {"city": "NYC", "zip": "10001"})
name, age, location = data
city = location["city"]
print(f"{name} is {age} and lives in {city}")
```

### Advanced Examples
```python
# Walrus operator (assignment expressions, Python 3.8+)
if (n := len(heavy_computation())) > 0:
    print(f"Processed {n} items")

# Using walrus in list comprehensions
results = [y for x in range(10) if (y := x * 2) > 10]
print(results)  # [12, 14, 16, 18]

# Descriptor-based assignment (properties)
class ValidatedField:
    def __init__(self, validator):
        self.validator = validator
        self.data = {}
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, obj, objtype=None):
        return self.data.get(self.name)
    
    def __set__(self, obj, value):
        if not self.validator(value):
            raise ValueError(f"Invalid value for {self.name}")
        self.data[self.name] = value

class Person:
    age = ValidatedField(lambda v: 0 <= v <= 150)
    name = ValidatedField(lambda v: isinstance(v, str) and len(v) > 0)

p = Person()
p.age = 30  # Works
# p.age = -1  # ValueError
p.name = "Alice"

# __setattr__ hook for custom assignment behavior
class TrackedObject:
    def __init__(self):
        self.changes = []
    
    def __setattr__(self, name, value):
        if name != "changes":
            self.changes.append((name, value))
        super().__setattr__(name, value)

t = TrackedObject()
t.x = 10
t.y = 20
print(f"Changes: {t.changes}")  # [('x', 10), ('y', 20)]
```

### Real-World Use Cases
- **Configuration**: Loading environment variables into named variables
- **Data Processing**: Unpacking database query results into named fields
- **API Responses**: Destructuring JSON responses into Python variables
- **State Management**: Tracking application state with boolean flags
- **Configuration**: Using walrus operator to avoid repeated calculations

### Common Mistakes
```python
# Mistake 1: Using = instead of == in conditions
# if x = 5:  # SyntaxError: invalid syntax
# Correct:
x = 5
if x == 5: pass

# Mistake 2: Unintentional reference sharing
original = [1, 2, 3]
reference = original  # Not a copy!
reference.append(4)
print(original)  # [1, 2, 3, 4] — modified!

# Correct: use .copy() or [:]
copy = original.copy()
copy.append(5)
print(original)  # [1, 2, 3, 4]

# Mistake 3: Mismatched unpacking
# a, b = [1, 2, 3]  # ValueError: too many values to unpack
a, *b = [1, 2, 3]  # Correct with star expression

# Mistake 4: Assignment to a slice modifies the list
items = [1, 2, 3, 4, 5]
items[1:3] = [100]  # Replaces items[1] and items[2] with one element
print(items)  # [1, 100, 4, 5]
```

### Best Practices
- Use meaningful variable names (not single letters except in loops)
- Use tuple unpacking for cleaner multiple assignments
- Use the walrus operator sparingly; prioritize readability
- Prefer immutable assignments where possible
- Use `copy()` explicitly for mutable objects to signal intent
- Keep assignment and mutation separate concepts in your mind

### Performance Considerations
- Attribute access (`obj.attr = value`) is slower than local variable assignment
- Local variable lookup is ~10x faster than global variable lookup
- The walrus operator avoids redundant computation
- Star expression unpacking creates list objects (memory overhead for large sequences)
- Assignment to `__slots__` attributes is faster than `__dict__` assignment

### Interview Questions
1. How does Python handle variable assignment (pass-by-object-reference)?
2. What is the difference between assignment and mutation?
3. How does tuple unpacking work in Python?
4. What is the walrus operator (Python 3.8+)?
5. What happens when you assign a variable that doesn't exist?
6. How does augmented assignment (`+=`, `*=`) differ for mutable vs immutable types?
7. Can you unpack an iterator? What are the memory implications?
8. What is `__setattr__` and how does it affect assignment?
9. How are default parameter values bound in function definitions?
10. What is the difference between `a = b = []` and `a = []; b = []`?

### Coding Challenges
```python
# Challenge 1: Swap three variables in one line
a, b, c = 1, 2, 3
a, b, c = c, a, b
print(f"a={a}, b={b}, c={c}")  # a=3, b=1, c=2

# Challenge 2: Nested unpacking
data = {"users": [{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]}
for user in data["users"]:
    name, age = user["name"], user["age"]
    print(f"{name} is {age}")

# Challenge 3: Validate assignment with descriptor
class PositiveNumber:
    def __set_name__(self, owner, name):
        self.name = name
    def __get__(self, obj, objtype=None):
        return getattr(obj, f"_{self.name}", None)
    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError(f"{self.name} must be positive")
        object.__setattr__(obj, f"_{self.name}", value)

class Order:
    quantity = PositiveNumber()
    price = PositiveNumber()

order = Order()
order.quantity = 5
order.price = 29.99
# order.quantity = -1  # ValueError
```

### Related Topics
- Object References and Mutability
- Namespaces and Scope
- Augmented Assignment
- The Walrus Operator (`:=`)
- Descriptors and `__setattr__`

## Dynamic Typing

### What It Is
Dynamic typing means variables do not have fixed types in Python. A variable is simply a name bound to an object; the object has a type, but the name does not. The same name can refer to an integer, a string, a list, or any other type at different points in the program.

### Why It Is Important
Dynamic typing enables rapid prototyping, duck typing, and polymorphism without explicit interfaces. It allows writing generic functions that work with any type supporting the required operations. However, it shifts type error detection from compile-time to runtime, requiring thorough testing.

### How It Works Internally
Every Python object has a `PyObject` header containing `ob_refcnt` (reference count) and `ob_type` (pointer to the type object). When a name is assigned, CPython stores the name-object mapping in the namespace's dictionary. The name has no type information — only the object does. Type checking at runtime uses `isinstance()` which walks the MRO (Method Resolution Order) or checks `ob_type` directly. In CPython, `Py_TYPE(obj)` efficiently retrieves an object's type.

### Syntax
```python
# Same variable can hold different types
value = 42
print(type(value))  # <class 'int'>

value = "hello"
print(type(value))  # <class 'str'>

value = [1, 2, 3]
print(type(value))  # <class 'list'>

# Type checking
isinstance(value, list)   # True
type(value) is list       # True (but doesn't handle subclasses)

# Duck typing
def process(item):
    return item.upper()  # Works for any object with .upper()

process("hello")  # 'HELLO'
# process(42)     # AttributeError — no .upper() on int
```

### Beginner Examples
```python
# Dynamic typing in action
x = 10
print(f"x = {x}, type = {type(x).__name__}")

x = "Python"
print(f"x = {x}, type = {type(x).__name__}")

x = [1, 2, 3]
print(f"x = {x}, type = {type(x).__name__}")

x = lambda a: a * 2
print(f"x = {x}, type = {type(x).__name__}")

# Type changes implicitly based on operations
result = 10 + 5       # int
result = 10 + 5.0     # float (implicit conversion)
result = "count: " + str(10)  # str (explicit conversion needed)
```

### Intermediate Examples
```python
# Duck typing in practice
class Dog:
    def speak(self):
        return "Woof!"

class Cat:
    def speak(self):
        return "Meow!"

class Car:
    def honk(self):
        return "Beep!"

def make_sound(animal):
    return animal.speak()  # Only requires .speak() method

print(make_sound(Dog()))  # Woof!
print(make_sound(Cat()))  # Meow!
# print(make_sound(Car()))  # AttributeError - no .speak()

# Type checking patterns
def safe_divide(a, b):
    if not isinstance(b, (int, float)):
        return "Division not supported"
    if b == 0:
        return "Cannot divide by zero"
    return a / b

# Using type annotations (Python 3.5+)
from typing import Union

def process_value(value: Union[int, str, list]) -> str:
    if isinstance(value, int):
        return f"Number: {value}"
    elif isinstance(value, str):
        return f"Text: {value.upper()}"
    elif isinstance(value, list):
        return f"List with {len(value)} items"
    return "Unknown type"
```

### Advanced Examples
```python
# Protocol classes for structural typing (Python 3.8+)
from typing import Protocol, runtime_checkable

@runtime_checkable
class Drawable(Protocol):
    def draw(self) -> str: ...

class Circle:
    def draw(self) -> str:
        return "Drawing circle"

class Square:
    def draw(self) -> str:
        return "Drawing square"

class NotDrawable:
    pass

def render(item: Drawable):
    print(item.draw())

render(Circle())  # OK
render(Square())  # OK
print(isinstance(NotDrawable(), Drawable))  # False

# Type narrowing with type guards
def process(data: int | str | list) -> str:
    match data:
        case int() as n:
            return f"int: {n * 2}"
        case str() as s:
            return f"str: {s[::-1]}"
        case list() as lst:
            return f"list: {sum(lst)}"
        case _:
            return "unknown"

print(process(42))       # int: 84
print(process("hello"))  # str: olleh
print(process([1,2,3]))  # list: 6

# TypeVar for generics
from typing import TypeVar, Sequence

T = TypeVar('T')

def first(items: Sequence[T]) -> T:
    return items[0]

print(first([1, 2, 3]))        # 1 (int)
print(first("hello"))          # 'h' (str)
print(first((True, False)))    # True (bool)
```

### Real-World Use Cases
- **Data Pipelines**: Processing heterogeneous data from CSV/JSON
- **Web Frameworks**: Flask/FastAPI route handlers accept multiple types
- **Serialization**: JSON serialization works with any basic Python type
- **Configuration**: Loading typed values from config files
- **Dependency Injection**: Passing different implementations to functions

### Common Mistakes
```python
# Mistake 1: Assuming type based on variable name
value = 42
# ... 100 lines later ...
value = "text"  # Legal but may surprise readers

# Mistake 2: Not validating types in public APIs
def process(items):
    for item in items:  # Assumes items is iterable
        pass

# process(None)  # TypeError: 'NoneType' object is not iterable

# Mistake 3: Over-relying on type() vs isinstance()
class MyInt(int): pass

x = MyInt(5)
print(type(x) is int)       # False! (type is MyInt)
print(isinstance(x, int))   # True

# Mistake 4: Type narrowing with == instead of isinstance
x = 10
if type(x) == int:  # Works but doesn't handle subclasses
    pass
if isinstance(x, int):  # Better
    pass
```

### Best Practices
- Use type hints for public API functions
- Use `isinstance()` over `type()` for type checking (handles subclasses)
- Check for the required behavior, not the type (duck typing) when possible
- Use `typing.Protocol` for structural subtyping (Python 3.8+)
- Use `match` statement for pattern matching (Python 3.10+)
- Avoid changing variable types in a single scope (confusing)
- Validate types at boundaries (function inputs, API endpoints)

### Performance Considerations
- Dynamic dispatch (method lookup) is slower than in statically-typed languages
- Python 3.12+ has specialized bytecode for common type patterns
- `isinstance()` is fast (O(1) for most cases with type caching)
- Type hints have zero runtime cost (they're not enforced at runtime)
- Using `__slots__` can speed up attribute access and reduce memory
- Mypy/pyright can catch type errors at development time

### Interview Questions
1. What is dynamic typing and how does it differ from static typing?
2. What is duck typing in Python?
3. How does Python handle type checking at runtime?
4. What is the difference between `type()` and `isinstance()`?
5. How do type hints affect runtime behavior?
6. What are the advantages and disadvantages of dynamic typing?
7. Explain structural typing with Protocols (Python 3.8+).
8. How does Python handle type coercion in operations?
9. What is the `typing` module and when should you use it?
10. How do you handle multiple return types in Python?

### Coding Challenges
```python
# Challenge 1: Type-safe function through dynamic checking
def ensure_type(value, target_type, default=None):
    """Return value if it matches target_type, else default."""
    if isinstance(value, target_type):
        return value
    try:
        return target_type(value)
    except (ValueError, TypeError):
        return default

print(ensure_type("42", int))       # 42
print(ensure_type("abc", int))      # None
print(ensure_type("3.14", float))   # 3.14

# Challenge 2: Duck typing check
def quack(obj):
    """Check if object can 'quack' like a duck."""
    return hasattr(obj, "quack") and callable(getattr(obj, "quack"))

class Duck:
    def quack(self):
        return "Quack!"
    
class Person:
    def quack(self):
        return "I'm pretending to be a duck"

print(quack(Duck()))    # True
print(quack(Person()))  # True
print(quack(42))        # False
```

### Related Topics
- Duck Typing vs Nominal Typing
- Type Hints and Annotations
- The `typing` Module
- Protocols (Structural Subtyping)
- `isinstance()` and `type()`
- Pattern Matching (match/case)

## Naming Rules

### What It Is
Python has specific rules for valid identifiers (names for variables, functions, classes, etc.) and widely-adopted conventions for naming different kinds of objects. The rules define what's syntactically allowed; the conventions define what's idiomatic and readable.

### Why It Is Important
Consistent naming conventions improve code readability, maintainability, and team collaboration. PEP 8 (Python's style guide) provides standardized naming conventions that virtually all Python projects follow. Correct naming avoids conflicts with keywords and built-in names.

### How It Works Internally
Python's lexer (`tokenizer.c`) validates identifiers during lexical analysis. An identifier must match the regex `[a-zA-Z_][a-zA-Z0-9_]*`. Unicode characters (including letters from non-Latin scripts) are allowed in Python 3. Reserved keywords are stored in a frozen set and cannot be used as identifiers. Name resolution follows the LEGB (Local, Enclosing, Global, Built-in) rule through nested namespaces.

### Syntax
```python
# Valid identifiers
variable = 1
_variable = 2          # Internal use convention
variable_ = 3          # Avoids keyword conflict
VAR_NAME = 4           # Constant convention
camelCase = 5          # Not Pythonic but valid
unícode_ñames = 6      # Unicode allowed (Python 3+)
__double_underscore = 7 # Name mangling in classes

# Invalid identifiers
# 1variable = 1        # Starts with digit
# my-var = 1           # Contains hyphen
# class = 1            # Reserved keyword
# for = 1              # Reserved keyword
# import = 1           # Reserved keyword
```

### Beginner Examples
```python
# Good naming — descriptive and readable
user_name = "Alice"
total_price = 49.99
is_logged_in = True
max_retries = 3

# Bad naming — cryptic
u = "Alice"        # What is 'u'?
tp = 49.99         # What is 'tp'?
l = True           # Looks like '1'
mxr = 3            # Unreadable

# Constants (by convention)
MAX_CONNECTIONS = 100
DEFAULT_TIMEOUT = 30
PI = 3.14159

# Internal use (by convention)
_private_helper = "not exported by module"
__internal = "name mangled in classes"
```

### Intermediate Examples
```python
# Name mangling with double underscore
class Base:
    def __init__(self):
        self.__secret = "hidden"
    
    def get_secret(self):
        return self.__secret

class Derived(Base):
    def __init__(self):
        super().__init__()
        self.__secret = "overridden?"  # Different attribute!

b = Base()
print(b.get_secret())  # "hidden"
print(b._Base__secret)  # "hidden" (name mangled to _Base__secret)

d = Derived()
print(d.get_secret())           # "hidden" (parent's still accessible)
print(d._Derived__secret)       # "overridden?"
print(d._Base__secret)          # "hidden"

# Using trailing underscore to avoid keyword conflicts
def process_items(items, class_=None):
    """Process items, optionally filtering by class."""
    if class_ is not None:
        items = [i for i in items if isinstance(i, class_)]
    return items

# Using _ for throwaway variables
for _ in range(5):
    print("Hello", end=" ")

a, _, b = (1, 2, 3)  # _ gets the value 2 (ignored)
```

### Advanced Examples
```python
# Naming conventions for special methods (dunder methods)
class CustomContainer:
    def __init__(self, items):
        self._items = list(items)
    
    def __len__(self):
        return len(self._items)
    
    def __getitem__(self, index):
        return self._items[index]
    
    def __iter__(self):
        return iter(self._items)
    
    def __repr__(self):
        return f"CustomContainer({self._items})"

# Name resolution order (LEGB)
x = "global"

def outer():
    x = "enclosing"
    
    def inner():
        x = "local"
        print(x)  # "local"
    
    inner()

outer()

# Checking identifier validity
import keyword

def is_valid_identifier(name: str) -> bool:
    """Check if a string is a valid Python identifier."""
    return name.isidentifier() and not keyword.iskeyword(name)

names = ["var", "class", "2var", "my-var", "_private", "var_1"]
for n in names:
    print(f"{n:15} valid={is_valid_identifier(n)}")

# Dynamic attribute names with getattr/setattr
class DynamicObject:
    pass

obj = DynamicObject()
attr_name = "user_score"
value = 95
setattr(obj, attr_name, value)
print(getattr(obj, attr_name))  # 95
```

### Real-World Use Cases
- **API Development**: Naming REST endpoints with snake_case
- **Data Science**: DataFrame column names use snake_case
- **Configuration**: Environment variables use UPPER_CASE
- **ORM Models**: Database models use PascalCase for classes
- **Testing**: Test functions use descriptive snake_case names

### Common Mistakes
```python
# Mistake 1: Shadowing built-in names
list = [1, 2, 3]     # Overwrites built-in list()
print(list("hello"))  # TypeError: 'list' object is not callable

# Correct:
my_list = [1, 2, 3]

# Mistake 2: Using 'l' (lowercase L) or 'O' (uppercase O) as names
l = [1, 2, 3]  # l looks like 1 in many fonts
O = "oxygen"   # O looks like 0

# Mistake 3: Name collision in class inheritance
class Parent:
    def __init__(self):
        self.value = 42

class Child(Parent):
    def __init__(self):
        self.value = "text"  # Hides Parent's value!

# Mistake 4: Using misleading names
data = "hello"     # 'data' is too vague
temp = "result"    # 'temp' is misleading
```

### Best Practices
- Follow PEP 8: `snake_case` for variables/functions, `PascalCase` for classes, `UPPER_CASE` for constants
- Use descriptive names: `user_count` over `uc` or `count`
- Use leading underscore `_` for internal/private attributes
- Use trailing underscore to avoid keyword conflicts: `class_`
- Use double underscore `__` only for name mangling in class hierarchies
- Use `_` for throwaway variables
- Never shadow built-in names (`list`, `str`, `dict`, `int`, `print`)
- Keep names concise but unambiguous (3-4 words max)

### Performance Considerations
- Name resolution follows LEGB rules; local variables are fastest
- Global variable lookup is slower than local (dictionary lookup vs array indexing)
- Using `__slots__` in classes eliminates `__dict__`, speeding attribute access
- `getattr()` and `setattr()` are ~2x slower than direct attribute access
- Python 3.12 optimizes common attribute access patterns

### Interview Questions
1. What are Python's naming rules for identifiers?
2. What is PEP 8 and what naming conventions does it recommend?
3. What is the difference between `_var`, `__var`, and `var_`?
4. How does name mangling work with `__var` in classes?
5. What is the LEGB rule for name resolution?
6. Why should you avoid shadowing built-in names?
7. What is a "throwaway" variable and how do you use it?
8. Are Unicode characters allowed in Python identifiers?
9. What happens if you use a reserved keyword as a name?
10. How do you check if a string is a valid Python identifier?

### Coding Challenges
```python
# Challenge 1: Identifier validator
import keyword
import re

def validate_identifiers(code: str) -> list[dict]:
    """Find valid and invalid identifiers in Python code."""
    tokens = re.findall(r'\b[a-zA-Z_]\w*\b', code)
    results = []
    seen = set()
    for token in tokens:
        if token not in seen:
            seen.add(token)
            results.append({
                "name": token,
                "valid": token.isidentifier() and not keyword.iskeyword(token),
                "is_keyword": keyword.iskeyword(token),
                "is_builtin": token in dir(__builtins__),
            })
    return results

code_snippet = """
def my_function(x, y, z):
    for item in range(x):
        print(item)
    return y + z
"""
for r in validate_identifiers(code_snippet):
    status = "✓" if r["valid"] else "✗"
    print(f"{status} {r['name']}")
```

### Related Topics
- PEP 8 Style Guide
- Reserved Keywords
- Built-in Names
- Scope (LEGB Rule)
- Name Mangling
- The `keyword` Module
