# Conditions - if, elif, else, ternary operator

## Introduction

Conditional statements are fundamental control flow structures that allow programs to execute different code paths based on boolean conditions. Python uses `if`, `elif` (short for "else if"), and `else` keywords to create conditional branches. Additionally, the ternary conditional expression provides inline conditionals for simple cases. Python's conditionals leverage truthy/falsy value evaluation and support short-circuit logical operators.

## if Statement

### What It Is
The `if` statement evaluates a condition and executes the associated block only if the condition is truthy. It is the most basic decision-making construct in Python.

### Why It Is Important
The `if` statement is the primary mechanism for conditional execution. It enables programs to respond differently based on input, state, or data characteristics. Understanding Python's truthiness evaluation is essential — any object can be used as a condition.

### How It Works Internally
The `if` statement compiles to bytecode that uses `POP_JUMP_IF_FALSE` (Python 3.11+) or `JUMP_IF_FALSE_OR_POP` (earlier versions). The condition is evaluated on the stack; if falsy (`__bool__()` returns False or `__len__()` returns 0), execution jumps to the next statement after the block. Since Python 3.11, the bytecode uses adaptive instructions for common patterns.

### Syntax
```python
if condition:
    statement(s)

# The condition can be any expression
if True:
    print("Always runs")

if 1:
    print("Also runs (1 is truthy)")

if []:
    print("Never runs (empty list is falsy)")
```

### Beginner Examples
```python
# Basic if statement
age = 20
if age >= 18:
    print("You are an adult")

# Using truthiness
name = input("Enter name: ")
if name:
    print(f"Hello, {name}!")
else:
    print("Name cannot be empty")

# Checking membership
fruits = ["apple", "banana", "cherry"]
if "banana" in fruits:
    print("Banana is in the list")

# Multiple conditions
x = 15
if x > 10 and x < 20:
    print("x is between 10 and 20")
```

### Intermediate Examples
```python
# Using all() and any() in conditions
scores = [85, 92, 78, 95]
if all(s >= 60 for s in scores):
    print("All students passed")

if any(s >= 90 for s in scores):
    print("At least one student got an A")

# Guard pattern with short-circuit
def process_user(user: dict | None):
    if user is None or not user.get("is_active"):
        return None
    
    if "admin" in user.get("roles", []):
        return "admin_panel"
    
    return "user_panel"

# Nested conditions (keep to 2-3 levels max)
def categorize_number(n):
    if n > 0:
        if n % 2 == 0:
            return "positive even"
        return "positive odd"
    elif n < 0:
        return "negative"
    return "zero"
```

### Advanced Examples
```python
# Overriding truthiness with __bool__ and __len__
class Inventory:
    def __init__(self):
        self.items = []
    
    def add(self, item):
        self.items.append(item)
    
    def __bool__(self):
        return len(self.items) > 0
    
    def __len__(self):
        return len(self.items)

inventory = Inventory()
if not inventory:
    print("Inventory is empty")

inventory.add("sword")
if inventory:
    print(f"Inventory has {len(inventory)} item(s)")

# Conditional expressions in comprehensions
numbers = range(-5, 6)
positive_evens = [n for n in numbers if n > 0 and n % 2 == 0]
print(positive_evens)  # [2, 4]

# Sentinel pattern with if
_MISSING = object()

def get_value(data: dict, key: str, default: object = _MISSING):
    if key not in data:
        if default is _MISSING:
            raise KeyError(f"Missing key: {key}")
        return default
    return data[key]

# Try it
data = {"a": 1, "b": 2}
print(get_value(data, "a"))           # 1
print(get_value(data, "c", -1))       # -1
# get_value(data, "c")  # KeyError

# Walrus operator in conditions (Python 3.8+)
import re

text = "Contact: alice@example.com"
if match := re.search(r'[\w.+-]+@[\w-]+\.[\w.]+', text):
    print(f"Found email: {match.group()}")
else:
    print("No email found")
```

### Real-World Use Cases
- **Input Validation**: Checking user input meets requirements
- **Feature Toggles**: Enabling/disabling features based on configuration
- **State Machines**: Transitioning between states based on conditions
- **Data Filtering**: Selecting records matching criteria
- **Error Handling**: Graceful degradation when operations fail

### Common Mistakes
```python
# Mistake 1: Assignment in condition
# if x = 5:  # SyntaxError
x = 5
if x == 5:  # Correct
    pass

# Mistake 2: Redundant == True/False
is_valid = True
if is_valid == True:  # Redundant
    pass
if is_valid:  # Correct
    pass

# Mistake 3: Using == for None
value = None
if value == None:  # Works but not idiomatic
    pass
if value is None:  # Correct
    pass

# Mistake 4: Not accounting for falsy 0
count = 0
if count:  # False — but 0 might be a valid count
    print(f"Count is {count}")
# Better:
if count is not None:
    print(f"Count is {count}")

# Mistake 5: Overly complex conditions
# Bad:
if user is not None and user.is_active and user.age >= 18 and user.country == "US":
    pass
# Better:
is_eligible = user is not None and user.is_active
is_adult = user.age >= 18 if user else False
is_domestic = user.country == "US" if user else False
if is_eligible and is_adult and is_domestic:
    pass
```

### Best Practices
- Use truthiness directly: `if items:` not `if len(items) > 0:`
- Use `is None` for None checks, never `== None`
- Extract complex conditions to named variables
- Keep `if` blocks short (consider extracting to functions)
- Avoid deep nesting (3+ levels) — use early returns
- Use `all()` and `any()` for collection conditions
- Use the walrus operator (`:=`) sparingly for conditions

### Performance Considerations
- `if` evaluation is very fast (C-level bytecode)
- `if x is None` is faster than `if x == None`
- Short-circuit `and`/`or` can avoid expensive function calls
- Python 3.11+ has adaptive bytecode that optimizes common `if` patterns
- Using `in` for membership is O(n) for lists, O(1) for sets

### Interview Questions
1. What values are considered falsy in Python?
2. How does Python evaluate conditions internally?
3. What is the difference between `if x:` and `if x is True:`?
4. How do you implement a switch/case in Python (pre-3.10)?
5. What is the walrus operator and how is it used in conditions?
6. How does short-circuit evaluation work in conditions?
7. When should you use `all()` and `any()` in conditions?
8. What is the recommended maximum nesting depth for if statements?
9. How do you customize truthiness for your own classes?
10. What is the guard pattern and how does it reduce nesting?

### Coding Challenges
```python
# Challenge 1: FizzBuzz
for i in range(1, 101):
    if i % 15 == 0:
        print("FizzBuzz")
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)

# Challenge 2: Leap year checker
def is_leap_year(year: int) -> bool:
    return year % 400 == 0 or (year % 4 == 0 and year % 100 != 0)

# Challenge 3: Custom truthiness
class SafeString:
    def __init__(self, value):
        self.value = value
    
    def __bool__(self):
        if self.value is None:
            return False
        return len(self.value.strip()) > 0

print(bool(SafeString("")))       # False
print(bool(SafeString("  ")))     # False
print(bool(SafeString("hello")))  # True
```

### Related Topics
- Boolean Type and Truthiness
- Logical Operators
- Short-Circuit Evaluation
- Match-Case (Structural Pattern Matching)
- The Walrus Operator

## elif Statement

### What It Is
`elif` is Python's contraction of "else if". It follows an `if` statement and allows checking additional conditions when the preceding conditions are falsy. Multiple `elif` blocks can be chained.

### Why It Is Important
`elif` enables mutually exclusive multi-way branching without nested `if` statements. It makes code more readable than `else: if:` nesting and is essential for cascading decision logic like grading systems, menu routing, or state machines.

### How It Works Internally
`elif` is not a separate keyword at the bytecode level — it is syntactic sugar for `else: if:` without additional indentation. The compiler generates a chain of conditional jumps. If a condition is truthy, the corresponding block executes and execution jumps to the end of the entire `if-elif-else` chain.

### Syntax
```python
if condition1:
    statement(s)
elif condition2:
    statement(s)
elif condition3:
    statement(s)
else:
    statement(s)
```

### Beginner Examples
```python
# Grade classification
score = 85
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"
print(f"Score {score} = Grade {grade}")

# Traffic light
light = "green"
if light == "red":
    print("Stop")
elif light == "yellow":
    print("Slow down")
elif light == "green":
    print("Go")
else:
    print("Invalid light color")

# Day of week
day_num = 3
if day_num == 1:
    day = "Monday"
elif day_num == 2:
    day = "Tuesday"
elif day_num == 3:
    day = "Wednesday"
elif day_num == 4:
    day = "Thursday"
elif day_num == 5:
    day = "Friday"
elif day_num == 6:
    day = "Saturday"
elif day_num == 7:
    day = "Sunday"
else:
    day = "Invalid day"
print(day)
```

### Intermediate Examples
```python
# Multi-condition elif chain
def shipping_cost(weight: float, destination: str, express: bool) -> float:
    base = 0.0
    if weight <= 0.5:
        base = 5.0
    elif weight <= 2.0:
        base = 8.0
    elif weight <= 10.0:
        base = 12.0
    else:
        base = 20.0
    
    if destination == "international":
        base *= 2.5
    elif destination == "express":
        base *= 1.5
    
    if express:
        base *= 1.75
    
    return base

print(shipping_cost(1.5, "domestic", False))  # 8.0
print(shipping_cost(5.0, "international", True))  # 52.5

# Using elif with isinstance
def process_value(value):
    if isinstance(value, int):
        return f"Integer: {value * 2}"
    elif isinstance(value, float):
        return f"Float: {value:.2f}"
    elif isinstance(value, str):
        return f"String: {value.upper()}"
    elif isinstance(value, (list, tuple)):
        return f"Sequence: {len(value)} items"
    elif value is None:
        return "None"
    else:
        return f"Unknown type: {type(value).__name__}"

print(process_value(42))       # Integer: 84
print(process_value(3.14))     # Float: 3.14
print(process_value("hello"))  # String: HELLO
print(process_value(None))     # None
```

### Advanced Examples
```python
# Dynamic dispatch with elif (alternative to match-case)
import json

def handle_event(event_type: str, data: dict) -> str:
    if event_type == "user.created":
        return f"Created user: {data.get('name')}"
    elif event_type == "user.updated":
        return f"Updated user {data.get('id')}: {data.get('changes')}"
    elif event_type == "user.deleted":
        return f"Deleted user: {data.get('id')}"
    elif event_type == "order.placed":
        return f"Order {data.get('order_id')} placed for ${data.get('total')}"
    elif event_type == "order.shipped":
        return f"Order {data.get('order_id')} shipped"
    elif event_type == "payment.received":
        return f"Payment of ${data.get('amount')} received"
    else:
        return f"Unknown event: {event_type}"

# elif chain as a strategy pattern
class DiscountCalculator:
    def calculate(self, customer_type: str, amount: float) -> float:
        if customer_type == "regular":
            return amount  # No discount
        elif customer_type == "member":
            return amount * 0.95  # 5% off
        elif customer_type == "vip":
            return amount * 0.90  # 10% off
        elif customer_type == "employee":
            return amount * 0.80  # 20% off
        else:
            return amount

calc = DiscountCalculator()
print(calc.calculate("vip", 100.0))  # 90.0

# elif with ranges and binary search
import bisect

def get_rating(score: float) -> str:
    """Map score to rating using bisect (fast for many thresholds)."""
    breakpoints = [60, 70, 80, 90, 100]
    ratings = ["F", "D", "C", "B", "A"]
    index = bisect.bisect_left(breakpoints, score)
    return ratings[index] if index < len(ratings) else "A+"

# vs elif:
def get_rating_elif(score: float) -> str:
    if score >= 90:
        return "A"
    elif score >= 80:
        return "B"
    elif score >= 70:
        return "C"
    elif score >= 60:
        return "D"
    else:
        return "F"
```

### Real-World Use Cases
- **HTTP Routing**: Mapping URL paths to handlers
- **Menu Systems**: Processing user menu choices
- **Validation**: Cascading validation rules with specific error messages
- **Configuration**: Environment-specific settings (dev/staging/prod)
- **Data Transformation**: Type-specific processing of heterogeneous data

### Common Mistakes
```python
# Mistake 1: Using multiple if instead of elif
x = 10
if x > 5:
    print(">5")
if x > 8:  # Should be elif — both print!
    print(">8")
# Output: >5 \n >8 (both execute!)

# Mistake 2: Wrong order of conditions (most specific first)
def categorize(score):
    if score >= 60:  # Matches first for 85!
        return "Pass"
    elif score >= 90:
        return "Excellent"
    return "Fail"
# Fix: put 90 check first

# Mistake 3: Missing final else
def get_status(code):
    if code == 200:
        return "OK"
    elif code == 404:
        return "Not Found"
    elif code == 500:
        return "Server Error"
    # Missing else — implicitly returns None

# Mistake 4: Overlapping conditions
def test(n):
    if n > 0:
        return "Positive"
    elif n >= 0:  # Never reached (first catches all positive)
        return "Zero"
    return "Negative"
```

### Best Practices
- Order conditions from most specific to most general
- Always include an `else` clause (even if just a comment or assertion)
- Use `elif` instead of nested `else: if:` for readability
- Limit elif chains to ~5-7 branches; consider dict dispatch for longer chains
- Use match-case (Python 3.10+) for pattern matching over long elif chains
- Put the most common case first for performance

### Performance Considerations
- Each condition in an elif chain is evaluated sequentially
- Worst case: all conditions checked (O(n) where n = number of branches)
- Put the most likely condition first to minimize evaluations
- For long chains, consider dict dispatch (O(1)) or bisect (O(log n))
- elif chains are still very fast for <10 branches

### Interview Questions
1. What is the difference between `elif` and `else: if:` ?
2. How does Python execute an elif chain internally?
3. When should you use elif vs a dictionary dispatch?
4. Should you always include an else clause? Why?
5. How do you optimize a long elif chain?
6. What happens if no elif condition matches and there's no else?
7. How do elif and match-case compare (Python 3.10+)?
8. What is the recommended maximum length for an elif chain?
9. How do you handle overlapping conditions in elif chains?
10. How does Python determine which elif block to execute?

### Coding Challenges
```python
# Challenge 1: HTTP status code handler using elif
def http_status_message(code: int) -> str:
    if 100 <= code < 200:
        return "Informational"
    elif 200 <= code < 300:
        return "Success"
    elif 300 <= code < 400:
        return "Redirection"
    elif 400 <= code < 500:
        return "Client Error"
    elif 500 <= code < 600:
        return "Server Error"
    else:
        return "Unknown"

# Challenge 2: BMI calculator with weight categories
def bmi_category(weight_kg: float, height_m: float) -> str:
    bmi = weight_kg / (height_m ** 2)
    if bmi < 18.5:
        return "Underweight"
    elif bmi < 25:
        return "Normal"
    elif bmi < 30:
        return "Overweight"
    else:
        return "Obese"

print(bmi_category(70, 1.75))  # Normal
```

### Related Topics
- `if` Statement
- Multi-way Branching
- Dictionary Dispatch
- Match-Case Statement
- Boolean Logic

## else Statement

### What It Is
The `else` clause in a conditional statement defines a block of code that executes when the preceding `if` (and all `elif`) conditions are falsy. It serves as the default or fallback path.

### Why It Is Important
`else` ensures completeness in conditional logic — every possible input has a defined outcome. It prevents unexpected None returns and makes code more robust. `else` is also available on `for` and `while` loops and `try` blocks, though with different semantics.

### How It Works Internally
`else` generates a `JUMP_FORWARD` instruction at the end of the preceding code block, skipping over the `else` block when any preceding condition is true. If no condition is true, execution falls through to the `else` block. The `else` on loops executes after the loop completes normally (no `break`). The `else` on `try` executes if no exception occurred.

### Syntax
```python
# if-else
if condition:
    statement(s)
else:
    statement(s)

# For-else
for item in iterable:
    if condition:
        break
else:
    # Runs if no break occurred
    statement(s)

# While-else
while condition:
    if break_condition:
        break
else:
    # Runs if no break occurred
    statement(s)

# Try-else
try:
    dangerous_code()
except SomeError:
    handle_error()
else:
    # Runs if no exception occurred
    statement(s)
```

### Beginner Examples
```python
# Basic if-else
age = 16
if age >= 18:
    print("Can vote")
else:
    print("Cannot vote")

# For-else: search with notification
def find_item(items, target):
    for item in items:
        if item == target:
            print(f"Found: {item}")
            break
    else:
        print(f"{target} not found")

find_item([1, 2, 3], 2)  # Found: 2
find_item([1, 2, 3], 5)  # 5 not found

# While-else
def find_prime(limit):
    n = limit
    while n > 1:
        if all(n % i != 0 for i in range(2, int(n ** 0.5) + 1)):
            print(f"Found prime: {n}")
            break
        n -= 1
    else:
        print("No prime found")

find_prime(10)  # Found prime: 7
```

### Intermediate Examples
```python
# For-else for validation
def validate_all_passwords(users):
    for user in users:
        if len(user["password"]) < 8:
            print(f"Error: {user['name']} has weak password")
            break
    else:
        print("All passwords strong")

users = [
    {"name": "Alice", "password": "secure123"},
    {"name": "Bob", "password": "weak"},
    {"name": "Charlie", "password": "strongPass1"},
]
validate_all_passwords(users)  # Error: Bob has weak password

# Try-else-except-finally
def safe_divide(a, b):
    try:
        result = a / b
    except ZeroDivisionError:
        print("Cannot divide by zero")
        return None
    else:
        print(f"Division succeeded: {result}")
        return result
    finally:
        print("Cleanup performed")

safe_divide(10, 2)  # Division succeeded: 5.0, Cleanup performed
safe_divide(10, 0)  # Cannot divide by zero, Cleanup performed
```

### Advanced Examples
```python
# Loop else with nested break
def find_pythagorean_triplet(limit: int) -> tuple | None:
    """Find first Pythagorean triplet with sum <= limit."""
    for a in range(1, limit):
        for b in range(a, limit):
            c = (a ** 2 + b ** 2) ** 0.5
            if c.is_integer() and a + b + int(c) <= limit:
                return (a, b, int(c))
    else:
        return None

print(find_pythagorean_triplet(20))  # (3, 4, 5)
print(find_pythagorean_triplet(5))   # None

# else clause for sentinel-based iteration
def find_in_matrix(matrix, target):
    """Find target in 2D matrix, returning (row, col)."""
    for i, row in enumerate(matrix):
        for j, value in enumerate(row):
            if value == target:
                print(f"Found at ({i}, {j})")
                break
        else:
            continue  # Inner loop didn't find — keep searching
        break  # Inner loop found — stop outer
    else:
        print(f"{target} not found")

matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]
find_in_matrix(matrix, 5)  # Found at (1, 1)
find_in_matrix(matrix, 10)  # 10 not found

# else with context managers
import contextlib

@contextlib.contextmanager
def database_transaction():
    print("BEGIN TRANSACTION")
    try:
        yield
    except Exception:
        print("ROLLBACK")
        raise
    else:
        print("COMMIT")
    finally:
        print("CLOSE CONNECTION")

with database_transaction():
    print("Executing queries...")
    # If this raises, rollback; otherwise commit
```

### Real-World Use Cases
- **Search Algorithms**: Loop else to signal "not found" (more readable than flags)
- **Transaction Management**: Try-else for commit on success
- **Validation**: For-else to ensure all items pass validation
- **Game Logic**: While-else for "no valid moves" detection
- **Resource Cleanup**: Finally (not else) for guaranteed cleanup

### Common Mistakes
```python
# Mistake 1: Confusing else on if vs else on loops
# if-else: runs when condition is False
# for-else: runs when no break occurred
# while-else: runs when no break occurred

# Mistake 2: For-else with no break — else always runs
for i in range(3):
    print(i)
else:
    print("Always runs!")  # No break possible, always executes

# Mistake 3: Not understanding try-else timing
try:
    print("try")
except:
    print("except")
else:
    print("else")  # Runs only if no exception
finally:
    print("finally")  # Always runs

# Mistake 4: Using else when a simple fallback is clearer
if condition:
    result = value
else:
    result = default
# Better:
result = value if condition else default
```

### Best Practices
- Use for-else for search operations instead of boolean flags
- Use try-else for code that should run only on success
- Keep else blocks short and focused
- Always consider adding a comment explaining when the else runs
- Use `else` on loops sparingly — not all Python developers know this feature
- Prefer `else` over setting and checking boolean flags in loops

### Performance Considerations
- `else` on loops has zero overhead when the loop exits via `break`
- `else` on `try` is slightly slower than code after the `try-except` (extra bytecode)
- Using `else` instead of boolean flags reduces variable assignments
- No performance difference between `if-else` and `if: return; else: return`
- `else` on `for` adds only one extra bytecode instruction

### Interview Questions
1. When does the `else` clause on a `for` loop execute?
2. What is the difference between `else` on `if` vs `else` on `for`?
3. How does `try-except-else-finally` execution order work?
4. Why would you use `for-else` instead of a boolean flag?
5. Can you use `else` with `while`? When does it execute?
6. What happens if you have both `break` and `else` in a nested loop?
7. Is `else` on loops widely known? When is it appropriate?
8. How does `else` interact with `continue` in loops?
9. What's a common use case for `try-except-else`?
10. How do you implement for-else behavior without the else clause?

### Coding Challenges
```python
# Challenge 1: Prime number check with for-else
def is_prime(n: int) -> bool:
    if n <= 1:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    else:
        return True  # Only runs if no divisor found

for n in [2, 3, 4, 5, 17, 25]:
    print(f"{n}: {is_prime(n)}")

# Challenge 2: Connection retry with while-else
import time
import random

def connect_with_retry(max_retries: int = 3) -> bool:
    attempts = 0
    while attempts < max_retries:
        attempts += 1
        print(f"Connection attempt {attempts}...")
        if random.random() > 0.7:  # Simulate random success
            print("Connected!")
            return True
        time.sleep(0.5)
    else:
        print("All connection attempts failed")
        return False

connect_with_retry(3)
```

### Related Topics
- `if` Statement
- `elif` Statement
- `for` Loop
- `while` Loop
- `try`/`except`/`finally`
- Loop Control (`break`, `continue`)

## Ternary Operator

### What It Is
The ternary operator (conditional expression) is a concise way to write simple if-else logic in a single expression. Its syntax is `value_if_true if condition else value_if_false`. Unlike C's `? :`, Python's ternary reads like plain English.

### Why It Is Important
The ternary operator makes simple conditionals more readable by reducing boilerplate. It is essential for conditional expressions in list comprehensions, lambda functions, and return statements where a full if-else block would be unnecessarily verbose.

### How It Works Internally
The ternary expression is compiled to bytecode that evaluates the condition first, then jumps to either the true-branch or false-branch expression. Unlike `and`/`or` short-circuit patterns, the ternary operator explicitly evaluates only one branch. In CPython, it uses `JUMP_IF_FALSE_OR_POP` and related instructions.

### Syntax
```python
# Basic ternary
value = true_value if condition else false_value

# Nested ternary (use sparingly)
value = a if cond1 else b if cond2 else c

# In list comprehensions
result = [x if x > 0 else -x for x in numbers]

# In lambda
f = lambda x: "even" if x % 2 == 0 else "odd"

# In f-strings
print(f"Status: {'active' if is_active else 'inactive'}")
```

### Beginner Examples
```python
# Basic ternary usage
age = 20
status = "Adult" if age >= 18 else "Minor"
print(status)  # Adult

# In print statements
score = 85
print("Pass" if score >= 60 else "Fail")

# Simple assignment
x = -5
absolute = x if x >= 0 else -x
print(absolute)  # 5

# Max of two values (without max())
a, b = 10, 20
max_val = a if a > b else b
print(max_val)  # 20

# In return statements
def is_even(n):
    return True if n % 2 == 0 else False
# Even simpler: return n % 2 == 0
```

### Intermediate Examples
```python
# Ternary in list comprehensions
numbers = range(-5, 6)
signs = ["positive" if n > 0 else "negative" if n < 0 else "zero" for n in numbers]
print(signs)

# Nested ternary (clear with parentheses)
def categorize(x):
    return (
        "positive" if x > 0
        else "negative" if x < 0
        else "zero"
    )

# Ternary with and/or (older style, not recommended)
value = condition and true_value or false_value  # Fragile!

# Ternary for default values
def greet(name=None):
    return f"Hello, {name if name is not None else 'Guest'}!"

print(greet())         # Hello, Guest!
print(greet("Alice"))  # Hello, Alice!

# Ternary with method calls
data = {"count": 0}
display = data["count"] if "count" in data else "N/A"

# For conditional arguments
import math
x = -4.7
rounded = math.floor(x) if x < 0 else math.ceil(x)
print(rounded)  # -5 (floor for negative)
```

### Advanced Examples
```python
# Ternary in class definitions
class Config:
    def __init__(self, debug: bool = False):
        self.log_level = "DEBUG" if debug else "INFO"
        self.connection_pool = 10 if debug else 100

# Ternary for conditional lambda
compare = lambda a, b: a if a > b else b

# Ternary with type narrowing
def process(value: int | str) -> str:
    return value.upper() if isinstance(value, str) else str(value * 2)

# Ternary in decorators
def conditional_decorator(use_decorator: bool):
    def decorator(func):
        def wrapper(*args, **kwargs):
            return func(*args, **kwargs)
        return wrapper
    return decorator if use_decorator else lambda f: f

# Ternary for sentinel defaults
_MISSING = object()

def get_config(key: str, default: str = None):
    return config_value if (config_value := config.get(key, _MISSING)) is not _MISSING else default

# Elegant conditional chaining
def discount(price: float, customer_type: str) -> float:
    return (
        price * 0.9 if customer_type == "member"
        else price * 0.85 if customer_type == "vip"
        else price * 0.8 if customer_type == "employee"
        else price
    )
```

### Real-World Use Cases
- **Default Values**: `name if name else "Guest"`
- **Conditional Formatting**: `f"${amount:.2f}" if amount else "Free"`
- **Data Cleaning**: `value if value is not None else "N/A"`
- **CSS-like Conditional Classes**: `"active" if is_selected else "inactive"`
- **API Responses**: `{"error": msg} if failed else {"data": result}`
- **Conditional Key in Dict**: `key if include_key else None`

### Common Mistakes
```python
# Mistake 1: Using ternary when if-else is clearer
result = (a if cond1 else b) if cond2 else c  # Hard to read
# Better:
if cond2:
    result = a if cond1 else b
else:
    result = c

# Mistake 2: Overly complex ternary expressions
# value = a if cond1 else b if cond2 else c if cond3 else d
# Fine for 2 levels, confusing for more

# Mistake 3: Misplaced parentheses
result = a if cond else b + c  # Means: a if cond else (b + c)
# If you wanted: (a if cond else b) + c
result = (a if cond else b) + c

# Mistake 4: Side effects in ternary branches
# result = print("true") if cond else print("false")  # Returns None

# Mistake 5: Using ternary for assignment that could be if-else
# Either works, but if-else is often clearer for statements
```

### Best Practices
- Use ternary only for simple, single-expression conditionals
- Limit to one level; extract nested ternaries to functions
- Prefer parentheses for clarity around the condition
- Use ternary in list comprehensions, lambda, f-strings
- If it doesn't fit on one line, use a regular if-else
- Never put complex expressions or function calls with side effects in ternaries

### Performance Considerations
- Ternary is as fast as equivalent if-else (both compile to similar bytecode)
- Ternary avoids variable reassignment in some patterns
- In list comprehensions, ternary can be faster than if-else blocks
- No performance benefit from using ternary over if-else for readability
- Python 3.12+ optimizes common ternary patterns

### Interview Questions
1. What is the syntax of the Python ternary operator?
2. How is the ternary operator different from C's `? :` operator?
3. When should you use a ternary vs an if-else?
4. How do you nest ternary operators? Is it recommended?
5. Can you use a ternary inside an f-string?
6. How does the ternary operator work in list comprehensions?
7. What are the alternatives to the ternary operator?
8. Can a ternary expression have side effects?
9. How do you debug a complex ternary expression?
10. What is the Pythonic alternative to C's `? :` operator?

### Coding Challenges
```python
# Challenge 1: Conditional function application
def apply_if(value, condition, func):
    return func(value) if condition else value

print(apply_if("hello", True, str.upper))   # HELLO
print(apply_if("hello", False, str.upper))  # hello

# Challenge 2: Min/Max without built-in functions
def min_max(a, b):
    return (a if a <= b else b, a if a >= b else a)

print(min_max(10, 20))  # (10, 20)
print(min_max(30, 5))   # (5, 30)

# Challenge 3: Ternary-based FizzBuzz
def fizzbuzz(n):
    return (
        "FizzBuzz" if n % 15 == 0
        else "Fizz" if n % 3 == 0
        else "Buzz" if n % 5 == 0
        else str(n)
    )

for i in range(1, 16):
    print(fizzbuzz(i), end=" ")
# 1 2 Fizz 4 Buzz Fizz 7 8 Fizz Buzz 11 Fizz 13 14 FizzBuzz
```

### Related Topics
- Boolean Logic
- List Comprehensions
- Lambda Functions
- Short-Circuit Evaluation (`and`/`or`)
- Conditional Expressions in f-strings
