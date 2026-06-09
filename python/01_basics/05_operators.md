# Operators - Arithmetic (+, -, *, /, //, %, **), comparison, logical

## Introduction

Operators are symbols or keywords that perform operations on operands (values). Python provides a rich set of operators including arithmetic for mathematical computation, comparison for relational checks, and logical for boolean combination. Each operator has defined precedence and associativity that determines evaluation order in expressions. Python also supports operator overloading, allowing custom types to define operator behavior.

## Arithmetic Operators

### What It Is
Arithmetic operators perform mathematical computations on numeric operands. Python provides `+` (addition), `-` (subtraction), `*` (multiplication), `/` (division), `//` (floor division), `%` (modulus), and `**` (exponentiation). The behavior varies by operand type (int, float, complex, Decimal, Fraction).

### Why It Is Important
Arithmetic operations are the foundation of all numerical computing. Understanding the nuances — especially the difference between `/` and `//`, modulus with negative numbers, and type promotion — is essential for correct mathematical code.

### How It Works Internally
Each arithmetic operator corresponds to a special method on the operand objects: `__add__`, `__sub__`, `__mul__`, `__truediv__`, `__floordiv__`, `__mod__`, `__pow__`. Python first tries the left operand's method, then the right operand's reflected method (e.g., `__radd__`). For mixed types, Python follows a numeric hierarchy: `int` < `float` < `complex`. The result type follows the "wider" operand type.

### Syntax
```python
# Basic arithmetic
a + b   # Addition
a - b   # Subtraction
a * b   # Multiplication
a / b   # Division (always returns float)
a // b  # Floor division
a % b   # Modulus (remainder)
a ** b  # Exponentiation

# Augmented assignment
a += b  # a = a + b
a -= b  # a = a - b
a *= b  # a = a * b
a /= b  # a = a / b
a //= b # a = a // b
a %= b  # a = a % b
a **= b # a = a ** b
```

### Beginner Examples
```python
# All arithmetic operators
a, b = 15, 4
print(f"a = {a}, b = {b}")
print(f"a + b = {a + b}")      # 19
print(f"a - b = {a - b}")      # 11
print(f"a * b = {a * b}")      # 60
print(f"a / b = {a / b}")      # 3.75
print(f"a // b = {a // b}")    # 3
print(f"a % b = {a % b}")      # 3
print(f"a ** b = {a ** b}")    # 50625

# Integer vs float division
print(10 / 3)   # 3.3333333333333335
print(10 // 3)  # 3

# Exponentiation
print(2 ** 3)   # 8
print(9 ** 0.5) # 3.0 (square root)
print(2 ** -1)  # 0.5

# Order of operations (PEMDAS)
print(2 + 3 * 4)    # 14 (multiplication first)
print((2 + 3) * 4)  # 20 (parenthesis override)
```

### Intermediate Examples
```python
# Modulus with negative numbers
print(7 % 3)    # 1
print(-7 % 3)   # 2 (not -1!)
print(7 % -3)   # -2
print(-7 % -3)  # -1

# Floor division with negatives
print(7 // 3)    # 2
print(-7 // 3)   # -3 (rounds toward negative infinity)
print(7 // -3)   # -3
print(-7 // -3)  # 2

# Type promotion
result = 10 + 3.14       # float (13.14)
result = 10 + 3 + 2j     # complex (13+2j)
result = 10 + True       # int (11, True == 1)

# Powerful one-liners with operators
def is_even(n): return n % 2 == 0
def is_odd(n): return n % 2 == 1
def is_power_of_two(n): return n > 0 and (n & (n - 1)) == 0
def clamp(value, low, high): return max(low, min(value, high))

print(is_power_of_two(16))  # True
print(clamp(25, 0, 10))     # 10
```

### Advanced Examples
```python
# Operator overloading for custom types
class Vector:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y
    
    def __add__(self, other):
        if isinstance(other, Vector):
            return Vector(self.x + other.x, self.y + other.y)
        return NotImplemented
    
    def __sub__(self, other):
        if isinstance(other, Vector):
            return Vector(self.x - other.x, self.y - other.y)
        return NotImplemented
    
    def __mul__(self, scalar):
        if isinstance(scalar, (int, float)):
            return Vector(self.x * scalar, self.y * scalar)
        return NotImplemented
    
    def __rmul__(self, scalar):
        return self.__mul__(scalar)
    
    def __neg__(self):
        return Vector(-self.x, -self.y)
    
    def __abs__(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5
    
    def __eq__(self, other):
        if isinstance(other, Vector):
            return self.x == other.x and self.y == other.y
        return NotImplemented
    
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(2, 3)
v2 = Vector(1, 4)
print(v1 + v2)   # Vector(3, 7)
print(v1 - v2)   # Vector(1, -1)
print(v1 * 3)    # Vector(6, 9)
print(3 * v1)    # Vector(6, 9) (__rmul__)
print(-v1)       # Vector(-2, -3)
print(abs(v1))   # 3.605...
print(v1 == Vector(2, 3))  # True

# The @ operator (matrix multiplication, Python 3.5+)
import numpy as np

A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])
print(A @ B)  # Matrix multiplication

# Decorator for operator timing
import time
import functools

def timed_op(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__}: {elapsed:.9f}s")
        return result
    return wrapper

large = 10 ** 100000
@timed_op
def large_power():
    return large ** 2  # ~10ms
```

### Real-World Use Cases
- **Financial Calculations**: Interest rates, amortization schedules
- **Graphics Programming**: Vector transformations, matrix operations
- **Game Physics**: Velocity, acceleration, collision detection
- **Statistics**: Mean, variance, standard deviation computations
- **Cryptography**: Modular exponentiation, prime number generation
- **Data Science**: Normalization, scaling, feature engineering

### Common Mistakes
```python
# Mistake 1: Using / instead of // for integer division
# In Python 2, 10/3 was 3. In Python 3, it's 3.333...
# Always use // for integer division

# Mistake 2: Forgetting operator precedence
result = 2 ** 3 ** 2  # 512, not 64!
# ** is right-associative: 2 ** (3 ** 2) = 2 ** 9 = 512

# Mistake 3: Modulus with negative numbers
balance = -50
payment = 30
remainder = balance % payment  # 10 (not -20)
# Use: abs(balance) % payment for expected behavior

# Mistake 4: Integer overflow concern (doesn't exist in Python)
# x = 2 ** 100000  # Works fine in Python

# Mistake 5: Type errors from mixed arithmetic
# result = "count: " + 10  # TypeError
# Correct: result = "count: " + str(10)
```

### Best Practices
- Use `//` for integer division and `/` when float is needed
- Use parentheses to clarify complex expressions
- Use `math.isclose()` for float equality after arithmetic
- Use `divmod()` to get both quotient and remainder: `q, r = divmod(a, b)`
- For large integer exponentiation, `pow(base, exp, mod)` is more efficient
- Use `abs()` for absolute value
- Consider `decimal.Decimal` for financial precision

### Performance Considerations
- Integer arithmetic is very fast (C implementation)
- Float arithmetic uses hardware FPU (also very fast)
- Large integer operations are O(n) where n is the number of digits
- `pow(x, y, z)` is much faster than `(x ** y) % z` for large exponents
- Augmented assignment (`a += 1`) is slightly faster than `a = a + 1`
- `/` and `//` differ in cost: floor division for large ints is more complex

### Interview Questions
1. What is the difference between `/` and `//` in Python 3?
2. How does the modulus operator work with negative numbers in Python?
3. What is operator associativity? Give an example with `**`.
4. How does type promotion work in mixed-type arithmetic?
5. What are augmented assignment operators?
6. How do you overload an operator in a custom class?
7. What is the `@` operator used for (Python 3.5+)?
8. How does Python's `pow(x, y, z)` three-argument form work?
9. What is the difference between `__add__` and `__radd__`?
10. How does floor division behave with negative numbers?

### Coding Challenges
```python
# Challenge 1: Vector arithmetic class
class Vector2D:
    def __init__(self, x, y):
        self.x, self.y = x, y
    
    def __add__(self, other):
        return Vector2D(self.x + other.x, self.y + other.y)
    
    def __sub__(self, other):
        return Vector2D(self.x - other.x, self.y - other.y)
    
    def __mul__(self, scalar):
        return Vector2D(self.x * scalar, self.y * scalar)
    
    def dot(self, other):
        return self.x * other.x + self.y * other.y
    
    def __repr__(self):
        return f"V({self.x}, {self.y})"

v1 = Vector2D(3, 4)
v2 = Vector2D(1, 2)
print(v1 + v2)        # V(4, 6)
print(v1.dot(v2))     # 11

# Challenge 2: Matrix multiplication using @
import numpy as np

def matrix_power(M: np.ndarray, n: int) -> np.ndarray:
    result = np.eye(M.shape[0], dtype=int)
    base = M.copy()
    while n > 0:
        if n & 1:
            result = result @ base
        base = base @ base
        n >>= 1
    return result

# Fibonacci using matrix exponentiation
F = np.array([[1, 1], [1, 0]], dtype=object)
print(matrix_power(F, 10))  # Includes F(11)
```

### Related Topics
- The `operator` Module
- Operator Overloading
- The `math` Module
- The `decimal` and `fractions` Modules
- Augmented Assignment

## Comparison Operators

### What It Is
Comparison operators evaluate the relationship between two operands and return a boolean (`True` or `False`). Python provides `==` (equal), `!=` (not equal), `<` (less than), `>` (greater than), `<=` (less than or equal), and `>=` (greater than or equal).

### Why It Is Important
Comparisons enable decision-making, sorting, filtering, and validation. Python's unique chained comparisons (`a < b < c`) provide mathematical naturalness. Understanding the difference between `==` (value equality) and `is` (identity equality) is critical.

### How It Works Internally
The `==` operator calls `__eq__()` on the left operand, with the right operand as argument. If `__eq__` returns `NotImplemented`, Python falls back to `__eq__()` on the right operand. For ordering (`<`, `>`, etc.), Python calls `__lt__`, `__gt__`, `__le__`, `__ge__`. Chained comparisons like `a < b < c` are evaluated as `a < b and b < c` (with `b` evaluated only once).

### Syntax
```python
# Basic comparisons
a == b   # Equal
a != b   # Not equal
a < b    # Less than
a > b    # Greater than
a <= b   # Less than or equal
a >= b   # Greater than or equal

# Chained comparisons (Python-specific)
a < b < c     # Equivalent to: a < b and b < c
a <= b < c    # Mixed chaining
a == b == c   # All equal (not a == b and b == c)

# Identity comparison
a is b     # Same object?
a is not b # Different objects?
```

### Beginner Examples
```python
# Numeric comparisons
x, y = 10, 20
print(f"x == y: {x == y}")   # False
print(f"x != y: {x != y}")   # True
print(f"x < y: {x < y}")     # True
print(f"x > y: {x > y}")     # False
print(f"x <= y: {x <= y}")   # True
print(f"x >= y: {x >= y}")   # False

# Chained comparisons
age = 25
print(18 <= age <= 65)  # True (working age)
print(0 < age < 120)    # True (valid age)

# String comparison (lexicographic)
print("apple" < "banana")  # True
print("Hello" < "hello")   # True (H < h in ASCII)

# List/tuple comparison (element-by-element)
print([1, 2, 3] < [1, 2, 4])  # True
print((1, 2) == (1, 2))       # True

# None comparison
value = None
print(value is None)      # True (always use is)
print(value == None)      # True (works but not idiomatic)
```

### Intermediate Examples
```python
# Custom comparison for objects
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __eq__(self, other):
        if not isinstance(other, Person):
            return NotImplemented
        return self.name == other.name and self.age == other.age
    
    def __lt__(self, other):
        if not isinstance(other, Person):
            return NotImplemented
        # Sort by age, then name
        return (self.age, self.name) < (other.age, other.name)
    
    def __repr__(self):
        return f"Person({self.name}, {self.age})"

people = [
    Person("Alice", 30),
    Person("Bob", 25),
    Person("Charlie", 30),
    Person("Alice", 25),
]

print(sorted(people))
# [Person(Alice, 25), Person(Bob, 25), Person(Alice, 30), Person(Charlie, 30)]

# Comparing different types
print((1, 2, 3) < "hello")  # TypeError in Python 3
# Python 2 allowed this (arbitrary ordering), Python 3 does not

# The `cmp_to_key` pattern (for sorting with custom comparators)
from functools import cmp_to_key

def compare_by_parity(a, b):
    """Sort evens first, then odds."""
    if a % 2 == b % 2:
        return -1 if a < b else 1 if a > b else 0
    return -1 if a % 2 == 0 else 1

numbers = [3, 1, 4, 1, 5, 9, 2, 6]
print(sorted(numbers, key=cmp_to_key(compare_by_parity)))
```

### Advanced Examples
```python
# Partial ordering (objects that are not totally ordered)
from functools import total_ordering

@total_ordering
class Version:
    def __init__(self, major, minor, patch=0):
        self.major = major
        self.minor = minor
        self.patch = patch
    
    def __eq__(self, other):
        if not isinstance(other, Version):
            return NotImplemented
        return (self.major, self.minor, self.patch) == (
            other.major, other.minor, other.patch)
    
    def __lt__(self, other):
        if not isinstance(other, Version):
            return NotImplemented
        return (self.major, self.minor, self.patch) < (
            other.major, other.minor, other.patch)

# @total_ordering supplies __le__, __gt__, __ge__
v1 = Version(3, 10, 0)
v2 = Version(3, 12, 0)
print(v1 < v2)   # True
print(v1 <= v2)  # True
print(v1 >= v2)  # False

# Rich comparison with numpy arrays
import numpy as np

arr = np.array([1, 2, 3, 4, 5])
print(arr > 2)    # [False False  True  True  True]
print(arr % 2 == 0)  # [False  True False  True False]

# Comparison chaining edge cases
a = [1, 2]
b = [1, 2]
c = [1, 2]
print(a == b == c)  # True (a == b and b == c)

# But watch for side effects
def get_middle():
    print("Middle evaluated!")
    return 5

print(3 < get_middle() < 7)  # "Middle evaluated!" once, then True
```

### Real-World Use Cases
- **Sorting & Ranking**: Custom ordering for leaderboards
- **Data Filtering**: SQL-like WHERE clause comparisons
- **Validation**: Range checks, boundary verification
- **Version Comparison**: Checking minimum required versions
- **Permission Systems**: Hierarchical access levels
- **Search**: Fuzzy matching, range queries

### Common Mistakes
```python
# Mistake 1: Using = instead of ==
# if x = 5:  # SyntaxError

# Mistake 2: Using == when is is appropriate (or vice versa)
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)  # True (same values)
print(a is b)  # False (different objects)

# Mistake 3: Floating-point comparison
print(0.1 + 0.2 == 0.3)  # False!
import math
print(math.isclose(0.1 + 0.2, 0.3))  # True

# Mistake 4: Chained comparison misunderstanding
x = 5
print(3 < x < 10)    # True (chained)
print(3 < x and x < 10)  # Same, more explicit

# Mistake 5: Comparing incompatible types
# print(42 < "hello")  # TypeError in Python 3
```

### Best Practices
- Use `is` for comparing with `None`, `True`, `False`; use `==` for values
- Use `math.isclose()` for float comparisons
- Leverage chained comparisons: `if 0 <= score <= 100:`
- Use `sorted()` with `key=` for custom ordering (not `cmp_to_key`)
- Use `@total_ordering` to minimize boilerplate for custom classes
- Avoid comparing objects of incompatible types

### Performance Considerations
- `is` is faster than `==` (pointer comparison vs method call)
- Chained comparisons avoid double evaluation of middle operand
- `==` fallback to `is` for CPython for some built-in types (e.g., small ints)
- `sorted()` with `key=` is faster than with `cmp` (decorate-sort-undecorate)
- Comparison of short-circuit (`a < b < c`) stops at first False

### Interview Questions
1. What is the difference between `==` and `is`?
2. How do chained comparisons work in Python?
3. Can you compare objects of different types in Python 3?
4. How do you implement custom comparison for a class?
5. What is the `@total_ordering` decorator?
6. How does lexicographic ordering work for sequences?
7. What is the `NotImplemented` singleton and when is it used?
8. How do you handle floating-point comparisons safely?
9. What is the `cmp_to_key` function and when is it useful?
10. How does Python determine if two objects are equal by default?

### Coding Challenges
```python
# Challenge 1: Implement __eq__ for a custom class
class Book:
    def __init__(self, title, author, isbn):
        self.title = title
        self.author = author
        self.isbn = isbn
    
    def __eq__(self, other):
        if not isinstance(other, Book):
            return NotImplemented
        return self.isbn == other.isbn
    
    def __hash__(self):
        return hash(self.isbn)

# Challenge 2: Multi-field sorting
students = [
    {"name": "Alice", "grade": 90, "age": 20},
    {"name": "Bob", "grade": 85, "age": 22},
    {"name": "Charlie", "grade": 90, "age": 19},
    {"name": "Diana", "grade": 85, "age": 21},
]

# Sort by grade descending, then age ascending
sorted_students = sorted(
    students,
    key=lambda s: (-s["grade"], s["age"])
)
for s in sorted_students:
    print(f"{s['name']}: grade={s['grade']}, age={s['age']}")
```

### Related Topics
- `is` vs `==`
- `__eq__`, `__lt__`, `__hash__`
- The `@total_ordering` Decorator
- `functools.cmp_to_key`
- The `operator` Module
- `math.isclose()`

## Logical Operators

### What It Is
Logical operators combine boolean expressions. Python provides `and` (conjunction), `or` (disjunction), and `not` (negation). These operators support short-circuit evaluation and return one of the operands (not necessarily `True`/`False`), enabling elegant conditional patterns.

### Why It Is Important
Logical operators enable complex decision-making by combining multiple conditions. Short-circuit evaluation can prevent expensive or error-prone operations. Understanding the truthy/falsy nature of Python's logical operators unlocks concise and Pythonic code patterns.

### How It Works Internally
`and` and `or` are implemented in the bytecode evaluator (`ceval.c`). `and` evaluates the left operand; if it's falsy, returns it without evaluating the right side. If truthy, evaluates and returns the right operand. `or` evaluates the left operand; if truthy, returns it without evaluating the right side. If falsy, evaluates and returns the right operand. `not` calls `__bool__()` on the operand and inverts the result.

### Syntax
```python
# Logical operators
a and b   # True if both true
a or b    # True if either true
not a     # True if a is false

# Short-circuit behavior
# and: returns first falsy value, or last value
# or:  returns first truthy value, or last value

# In conditions
if condition1 and condition2:
    pass

if condition1 or condition2:
    pass

if not condition:
    pass
```

### Beginner Examples
```python
# Basic logical operators
a, b = True, False
print(f"True and False: {a and b}")  # False
print(f"True or False: {a or b}")    # True
print(f"not True: {not a}")          # False

# Logical operators with non-boolean values
print(0 and 42)       # 0 (falsy, short-circuits)
print(1 and 42)       # 42 (last truthy value)
print(0 or 42)        # 42 (first truthy value)
print(1 or 42)        # 1 (truthy, short-circuits)

# Common patterns
name = input("Enter name: ") or "Guest"
print(f"Hello, {name}!")

# Short-circuit in conditions
def check_user(user):
    return user is not None and "admin" in user.get("roles", [])

# Combining conditions
x = 15
if x > 0 and x < 100:
    print(f"{x} is between 0 and 100")
```

### Intermediate Examples
```python
# Short-circuit for safeguarding
items = None
# items.append(1)  # AttributeError!

# Short-circuit prevents error:
if items is not None and len(items) > 0:
    print(f"First item: {items[0]}")
else:
    print("Items is None or empty")

# Default values with or
def greet(name=None):
    name = name or "Guest"
    return f"Hello, {name}!"

print(greet())        # Hello, Guest!
print(greet("Bob"))   # Hello, Bob!
# But: greet(0) would give "Hello, Guest!" (0 is falsy)

# Better default with None check:
def greet_safe(name=None):
    if name is None:
        name = "Guest"
    return f"Hello, {name}!"

# Chained logical operations
def can_access(user, resource):
    return (
        user.is_authenticated
        and not user.is_banned
        and (user.is_admin or resource.is_public)
    )

# Using any() and all() for logical operations over iterables
scores = [85, 92, 78, 95, 88]
print(f"Any > 90: {any(s > 90 for s in scores)}")     # True
print(f"All > 70: {all(s > 70 for s in scores)}")     # True
print(f"All passed: {all(s >= 60 for s in scores)}")  # True
```

### Advanced Examples
```python
# Short-circuit for lazy evaluation
import time

def expensive_check():
    print("Expensive check running...")
    time.sleep(0.5)
    return True

# Short-circuit avoids expensive check
if False and expensive_check():
    print("This won't print")

if True or expensive_check():
    print("Expensive check skipped")

# Short-circuit in function arguments
def process(data: list | None, transform: bool = False):
    # Only call upper() if data is truthy and transform is True
    result = data and transform and [x.upper() for x in data]
    return result or data

print(process(["a", "b"], True))   # ['A', 'B']
print(process(["a", "b"], False))  # ['a', 'b']
print(process(None, True))         # None (stops at data check)

# Custom short-circuit with __bool__ and __len__
class LazyList:
    def __init__(self, generator_func):
        self._items = None
        self._generator = generator_func
    
    def _realize(self):
        if self._items is None:
            self._items = list(self._generator())
        return self._items
    
    def __bool__(self):
        # Only realize first element to check truthiness
        return len(self._realize()) > 0
    
    def __len__(self):
        return len(self._realize())
    
    def __getitem__(self, index):
        return self._realize()[index]

def generate_large():
    yield from range(1000000)

lazy = LazyList(generate_large)
if lazy:  # Only realizes first element... actually realizes all (simplified)
    print(f"Has {len(lazy)} items")

# Exclusive or (XOR) — Python doesn't have a logical XOR operator
def xor(a, b):
    return bool(a) != bool(b)

# Or using ^ for booleans
print(True ^ True)    # False (^ is bitwise but works for bool)
print(True ^ False)   # True
```

### Real-World Use Cases
- **Validation**: Multi-field form validation chaining
- **Access Control**: Combining role, permission, and status checks
- **Feature Flags**: Enabling features only when multiple conditions met
- **Data Filtering**: Combining filter criteria dynamically
- **API Rate Limiting**: Checking multiple throttle conditions
- **Configuration**: Override cascading (env > config > default)

### Common Mistakes
```python
# Mistake 1: Confusing and/or with &/|
# and/or are logical (short-circuit)
# &/| are bitwise (no short-circuit)
print(2 and 3)  # 3 (logical and — returns last truthy)
print(2 & 3)    # 2 (bitwise and)
print(2 or 3)   # 2 (logical or — returns first truthy)
print(2 | 3)    # 3 (bitwise or)

# Mistake 2: Using `and`/`or` for value when None check is better
default_value = 0
result = default_value or 42  # 42 (because 0 is falsy!)
# Better:
result = default_value if default_value is not None else 42

# Mistake 3: Not using parentheses for complex conditions
# if a or b and c:  # ambiguous — and has higher precedence
# Better:
if (a or b) and c: pass

# Mistake 4: Forgetting that `not` has lowest precedence
# if not a > 5:  # Means: if (not a) > 5 — WRONG
# Better:
if not (a > 5): pass

# Mistake 5: Using == True/False redundantly
if is_active == True:  # Redundant
    pass
if is_active:  # Better
    pass
```

### Best Practices
- Use `and`/`or`/`not` (not `&`/`|`/`~`) for logical operations
- Use parentheses for clarity in complex logical expressions
- Leverage short-circuit for validation (`x and x.method()`)
- Prefer `any()`/`all()` over chained `or`/`and` for iterables
- Use `is None` check instead of `or` default when None is possible
- Keep conditions simple; extract complex logic into named variables
- Use `all()` for "every item must satisfy" conditions

### Performance Considerations
- Short-circuit evaluation can avoid expensive operations
- `any()` and `all()` short-circuit (stop at first match/non-match)
- `not` is very fast (just inverts the boolean result)
- `and`/`or` are implemented at the C level (bytecode instructions)
- Logical operators with conditions are faster than if/else for simple cases
- Profile: `value = cond and X or Y` vs `value = X if cond else Y`

### Interview Questions
1. How does short-circuit evaluation work in Python?
2. What is the return value of `and` and `or` (not just True/False)?
3. What is the difference between `and`/`or` and `&`/`|`?
4. How do you implement XOR in Python?
5. What is operator precedence between `not`, `and`, and `or`?
6. How does `any()` and `all()` work internally?
7. What is the standard way to provide default values?
8. How do you safely access nested attributes with short-circuit?
9. What happens if you chain multiple `or` operators?
10. How does `bool()` interact with `and`/`or`?

### Coding Challenges
```python
# Challenge 1: Safe attribute access chain
def getattr_chain(obj, *attrs, default=None):
    """Safely access nested attributes."""
    current = obj
    for attr in attrs:
        current = getattr(current, attr, None)
        if current is None:
            return default
    return current

class Address:
    def __init__(self, city=None, zip_code=None):
        self.city = city
        self.zip_code = zip_code

class User:
    def __init__(self, address=None):
        self.address = address

user = User(Address("NYC", "10001"))
print(getattr_chain(user, "address", "city"))       # NYC
print(getattr_chain(user, "address", "zip_code"))   # 10001
print(getattr_chain(user, "nonexistent", "attr"))   # None

# Challenge 2: Multi-condition filter builder
class Filter:
    def __init__(self, *conditions):
        self.conditions = conditions
    
    def evaluate(self, item) -> bool:
        return all(cond(item) for cond in self.conditions)

even = lambda n: n % 2 == 0
positive = lambda n: n > 0
less_than_10 = lambda n: n < 10

f = Filter(even, positive, less_than_10)
numbers = [-2, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
print([n for n in numbers if f.evaluate(n)])  # [2, 4, 6, 8]
```

### Related Topics
- Boolean Type and Truthiness
- Short-Circuit Evaluation
- The `any()` and `all()` Functions
- Operator Precedence
- Conditional Expressions (Ternary)
