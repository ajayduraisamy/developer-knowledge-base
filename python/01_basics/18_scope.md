# Variable Scope - LEGB rule, global, nonlocal

## Introduction

Variable scope determines where a variable can be accessed within a program. Python follows the LEGB rule: Local, Enclosing, Global, Built-in. When you reference a variable, Python searches for it in this order. Understanding scope is crucial for writing correct code, avoiding unintended variable modification, and understanding closures and decorators.

## Why It Is Important

Understanding scope is essential because:
- It determines where variables can be accessed and modified
- It prevents accidental modification of variables in outer scopes
- It enables closures and decorators
- It helps understand the `global` and `nonlocal` keywords
- It explains variable shadowing and related bugs
- It's fundamental to writing modular, maintainable code

## Syntax

```python
# Global variable
global_var = "I'm global"

def outer_function():
    # Enclosing variable
    enclosing_var = "I'm enclosing"
    
    def inner_function():
        # Local variable
        local_var = "I'm local"
        print(local_var)
        print(enclosing_var)  # Can access enclosing
        print(global_var)     # Can access global
    
    inner_function()

# Global keyword
count = 0
def increment():
    global count  # Declare that we're using the global
    count += 1

# Nonlocal keyword
def outer():
    x = 10
    def inner():
        nonlocal x  # Declare that we're using enclosing
        x += 5
    inner()
    print(x)  # 15
```

## Examples

### LEGB Rule in Action

```python
# LEGB: Local, Enclosing, Global, Built-in

# Built-in scope
print(len)  # <built-in function len>

# Global scope
global_var = "global"

def test_scope():
    # Local scope
    local_var = "local"
    print(f"Inside function - local: {local_var}")
    print(f"Inside function - global: {global_var}")

test_scope()
# print(local_var)  # NameError: name 'local_var' is not defined
print(f"Outside function - global: {global_var}")

# Enclosing scope
def outer():
    outer_var = "outer"
    
    def inner():
        inner_var = "inner"
        print(f"Inner function: {inner_var}")
        print(f"Inner function accessing outer: {outer_var}")
        print(f"Inner function accessing global: {global_var}")
    
    inner()
    # print(inner_var)  # NameError

outer()
```

### Variable Shadowing

```python
# Local variable shadows global
x = 10  # Global

def shadow_example():
    x = 20  # Local - shadows global x
    print(f"Inside: x = {x}")

shadow_example()  # Inside: x = 20
print(f"Outside: x = {x}")  # Outside: x = 10

# Shadowing built-in function
# Avoid this!
def len(obj):
    return 42  # Shadows built-in len()

# print(len([1, 2, 3]))  # 42 (not 3!)

# Better to avoid shadowing built-ins
def custom_length(obj):
    return 42

# Parameter shadowing
value = 100

def process(value):  # Parameter shadows global
    print(f"Parameter value: {value}")

process(50)  # Parameter value: 50
print(f"Global value: {value}")  # Global value: 100
```

## Beginner Examples

```python
# Understanding where variables live
name = "Alice"  # Global scope

def greet():
    message = "Hello"  # Local scope
    print(f"{message}, {name}!")  # Can access global

greet()
# print(message)  # NameError

# Modifying global variables
counter = 0

def increment():
    global counter  # Must declare global to modify
    counter += 1

def read_counter():
    print(counter)  # Can read global without declaration

increment()
increment()
read_counter()  # 2

# Local vs global with same name
score = 100

def update_score(new_score):
    score = new_score  # Creates NEW local variable
    print(f"Inside: {score}")  # Local

update_score(200)
print(f"Outside: {score}")  # 100 (unchanged)

# Correct way to modify global
def update_score_global(new_score):
    global score
    score = new_score

update_score_global(200)
print(f"After global update: {score}")  # 200
```

## Intermediate Examples

### The nonlocal Keyword

```python
# nonlocal for nested functions
def make_counter():
    count = 0  # Enclosing scope
    
    def counter():
        nonlocal count  # Declare we're modifying enclosing
        count += 1
        return count
    
    return counter

counter_a = make_counter()
counter_b = make_counter()

print(f"Counter A: {counter_a()}")  # 1
print(f"Counter A: {counter_a()}")  # 2
print(f"Counter B: {counter_b()}")  # 1 (independent)

# Without nonlocal
def make_bad_counter():
    count = 0
    def counter():
        # count += 1  # UnboundLocalError!
        return count  # Can read, but can't assign without nonlocal
    return counter

# Practical nonlocal example
def create_accumulator():
    """Create an accumulator that maintains running total."""
    total = 0
    
    def add(value):
        nonlocal total
        total += value
        return total
    
    return add

accum = create_accumulator()
print(f"Add 5: {accum(5)}")    # 5
print(f"Add 3: {accum(3)}")    # 8
print(f"Add 10: {accum(10)}")  # 18
```

### Scope in Classes

```python
# Class and instance scope
class Student:
    school = "Python Academy"  # Class variable (class scope)
    
    def __init__(self, name):
        self.name = name  # Instance variable (instance scope)
    
    def introduce(self):
        # Local variable
        greeting = f"Hi, I'm {self.name} from {Student.school}"
        return greeting

s1 = Student("Alice")
s2 = Student("Bob")
print(s1.introduce())
print(s2.introduce())

# Class variable is shared
Student.school = "Code School"
print(s1.introduce())  # Now from "Code School"

# Instance variable is separate
s1.name = "Alicia"
print(s1.introduce())  # Alicia
print(s2.introduce())  # Bob

# Scope in comprehensions (Python 3)
x = "outer"
list_comp = [x for x in range(5)]
print(f"x after comprehension: {x}")  # "outer" (not 4)

# But in Python 2:
# x would be 4! (leaked from comprehension)
```

## Advanced Examples

### Closures

```python
# Closure: function that remembers its enclosing scope
def make_multiplier(factor):
    """Create a function that multiplies by factor."""
    def multiplier(x):
        return x * factor  # factor is from enclosing scope
    return multiplier

double = make_multiplier(2)
triple = make_multiplier(3)

print(f"Double 5: {double(5)}")  # 10
print(f"Triple 5: {triple(5)}")  # 15

# Closure with multiple values
def make_config(prefix, suffix):
    """Create a string formatter."""
    def format_value(value):
        return f"{prefix}{value}{suffix}"
    return format_value

wrap_parens = make_config("(", ")")
wrap_brackets = make_config("[", "]")
wrap_braces = make_config("{", "}")

print(wrap_parens("hello"))   # (hello)
print(wrap_brackets(42))      # [42]
print(wrap_braces("world"))   # {world}

# Closure for delayed execution
def make_logger(prefix):
    def logger(message):
        from datetime import datetime
        timestamp = datetime.now().isoformat()
        print(f"[{timestamp}] {prefix}: {message}")
    return logger

info_logger = make_logger("INFO")
error_logger = make_logger("ERROR")

info_logger("Application started")
error_logger("Connection failed")
```

### Decorator Pattern (Advanced Scope Usage)

```python
import functools

def require_authentication(func):
    """Decorator that checks authentication (scope demo)."""
    authenticated_user = {"name": "Alice", "role": "admin"}
    
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # Accessing authenticated_user from enclosing scope
        if not authenticated_user:
            return "Access denied: Not authenticated"
        print(f"Authenticated as: {authenticated_user['name']}")
        return func(*args, **kwargs)
    
    return wrapper

@require_authentication
def view_profile(user_id):
    return f"Profile data for user {user_id}"

print(view_profile(42))

# Multiple levels of scope
def make_validator(rules):
    """Create a validator with custom rules."""
    def validate(data):
        errors = []
        for rule_name, rule_func in rules.items():
            if not rule_func(data):
                errors.append(f"Failed: {rule_name}")
        return errors
    
    return validate

# Define validation rules
rules = {
    "not_empty": lambda d: len(d.get("name", "")) > 0,
    "min_age": lambda d: d.get("age", 0) >= 18,
}

validator = make_validator(rules)
print(validator({"name": "Alice", "age": 20}))  # []
print(validator({"name": "", "age": 15}))  # ['not_empty', 'min_age']
```

## Real-World Use Cases

- **Closures**: Creating function factories, partial application
- **Decorators**: Adding functionality to functions without modification
- **Configuration**: Creating functions with preset parameters
- **Caching**: Maintaining state across function calls (memoization)
- **Logging**: Creating loggers with different prefixes
- **Validation**: Creating validation functions with custom rules
- **Lazy Evaluation**: Delaying computation with closures

## Common Mistakes

```python
# Mistake 1: UnboundLocalError
x = 10
def increment():
    # x += 1  # UnboundLocalError
    # Python thinks x is local because we're assigning to it
    # Fix:
    global x
    x += 1

# Mistake 2: Forgetting global declaration
count = 0
def increment():
    # count += 1  # UnboundLocalError
    global count
    count += 1

# Mistake 3: Modifying enclosing variable without nonlocal
def outer():
    x = 10
    def inner():
        # x += 5  # UnboundLocalError
        nonlocal x
        x += 5
    inner()
    print(x)

outer()

# Mistake 4: Unintended variable shadowing
# Avoid using same names in nested scopes unnecessarily

# Mistake 5: Assuming loop variable persists
for i in range(5):
    pass
print(i)  # 4 (i is accessible after loop! No block scope)

# Mistake 6: Closure capturing loop variable
funcs = []
for i in range(3):
    funcs.append(lambda: i)  # All functions reference same i

for f in funcs:
    print(f())  # Prints 2, 2, 2 (all see final value of i)

# Fix using default argument:
funcs = []
for i in range(3):
    funcs.append(lambda x=i: x)  # Captures current value

for f in funcs:
    print(f())  # 0, 1, 2
```

## Best Practices

- Minimize use of global variables; pass parameters instead
- Use `global` sparingly and only when necessary
- Use `nonlocal` for nested function state
- Avoid variable shadowing (same name in nested scopes)
- Keep functions small with limited local scope
- Use closures for stateful functions instead of global variables
- Use class attributes for shared state when appropriate
- Use `_` prefix for internal/private variables (convention)
- Be aware that loops don't create new scope in Python
- Use `functools.wraps()` when creating decorators
- Use default arguments to capture loop variables in closures

## Interview Questions

1. What is the LEGB rule in Python?
2. What is the difference between `global` and `nonlocal`?
3. What is variable shadowing and why should you avoid it?
4. What is a closure and how does scope relate to it?
5. How does Python determine variable scope?
6. What is the `UnboundLocalError` and what causes it?
7. How do you modify a global variable from within a function?
8. How do closures capture variables from enclosing scopes?
9. What happens to loop variables after the loop ends?
10. How do you create a private variable in Python?

## Coding Challenges

```python
# Challenge 1: Counter using closures
def create_counter(start=0):
    count = start
    def counter():
        nonlocal count
        count += 1
        return count - 1
    return counter

counter = create_counter(10)
print(f"Count: {counter()}")  # 10
print(f"Count: {counter()}")  # 11
print(f"Count: {counter()}")  # 12

# Challenge 2: Function timing decorator
import time
import functools

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} took {end - start:.6f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(0.1)
    return "Done"

slow_function()

# Challenge 3: Limited call decorator
def max_calls(max_calls_count):
    def decorator(func):
        calls = 0
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            nonlocal calls
            calls += 1
            if calls > max_calls_count:
                return f"Max calls ({max_calls_count}) exceeded"
            return func(*args, **kwargs)
        return wrapper
    return decorator

@max_calls(3)
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))  # Hello, Alice!
print(greet("Bob"))    # Hello, Bob!
print(greet("Charlie")) # Hello, Charlie!
print(greet("Diana"))  # Max calls (3) exceeded

# Challenge 4: Variable scope analyzer
def analyze_scope():
    local_var = "local"
    print(f"Local: {local_var}")
    
    def inner():
        inner_var = "inner"
        print(f"Inner: {inner_var}")
    
    inner()
    
analyze_scope()

# Challenge 5: Cache with closures
def make_cache():
    cache = {}
    
    def get_or_compute(key, func):
        if key not in cache:
            cache[key] = func()
        return cache[key]
    
    def clear():
        cache.clear()
    
    def stats():
        return {
            "size": len(cache),
            "keys": list(cache.keys())
        }
    
    # Return multiple functions accessing same cache
    return get_or_compute, clear, stats

get, clear, stats = make_cache()
import time

result1 = get("expensive", lambda: sum(range(10**6)))
print(f"Result: {result1}")
print(f"Cache stats: {stats()}")

result2 = get("expensive", lambda: sum(range(10**6)))  # From cache
print(f"Cached result: {result2}")

clear()
print(f"After clear: {stats()}")  # size: 0
```

## Summary

Python's scope follows the LEGB rule (Local, Enclosing, Global, Built-in). Variables can be read from outer scopes but must be explicitly declared with `global` or `nonlocal` to be modified. Understanding scope is essential for closures, decorators, and writing bug-free code. Python does not have block-level scope (loops, if statements don't create new scope).

## Related Topics

- Functions and Closures
- Decorators
- Global and Nonlocal Keywords
- Namespace and Module Scope
- Variable Shadowing
- Function Definitions
- Lambda Functions
- LEGB Rule
