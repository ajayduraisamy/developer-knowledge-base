# Assertions - assert statement, debugging, contract checking

## Introduction

Assertions are a debugging and validation tool that checks whether a condition holds true at a specific point in a program's execution. Python's `assert` statement evaluates an expression and raises `AssertionError` if it is `False`. Assertions serve as executable documentation—they declare invariants that must hold and actively verify them during development. Unlike regular exception handling, assertions are designed to catch programmer errors (bugs), not runtime or user errors. They are typically disabled in production for performance.

## assert statement

### What It Is

The `assert` statement is a built-in Python statement that verifies a condition. If the condition is `True`, execution continues normally. If `False`, `AssertionError` is raised with an optional error message. When Python runs with optimization flags (`-O` or `-OO`), all `assert` statements are removed at compile time.

### Why It Is Important

Assertions catch bugs early by verifying assumptions about program state. They act as self-checking documentation—anyone reading the code sees what conditions are assumed to be true. Assertions make debugging faster by failing close to the root cause, rather than letting incorrect state propagate. They are a cornerstone of defensive programming and design-by-contract methodologies.

### How It Works Internally

The `assert` statement is compiled to bytecode that checks `__debug__` at runtime. When `__debug__` is `True` (the default), the condition is evaluated and `AssertionError` is raised if false. With the `-O` flag, `__debug__` is `False` and `assert` statements are compiled to `None` (no-op). This means the condition expression is NEVER evaluated when assertions are disabled, including any side effects.

### Syntax

```python
assert condition
assert condition, "Optional error message"
```

```python
x = -1
assert x >= 0, f"x must be non-negative, got {x}"
# AssertionError: x must be non-negative, got -1
```

```bash
# Disable assertions
python -O script.py
python -OO script.py  # Also removes docstrings
```

```python
# Check if assertions are enabled
if __debug__:
    print("Assertions are enabled")
```

### Beginner Examples

```python
# Simple assertion
def divide(a, b):
    assert b != 0, "Division by zero"
    return a / b

print(divide(10, 2))  # Works
# print(divide(10, 0))  # AssertionError

# Assertion with multiple conditions
def validate_user(user):
    assert user is not None, "User cannot be None"
    assert "name" in user, "User must have a name"
    assert "age" in user, "User must have an age"
    assert user["age"] >= 0, "Age cannot be negative"
    return True

# Assertion for data structure invariants
def pop_from_stack(stack):
    assert stack, "Cannot pop from an empty stack"
    return stack.pop()

# Assertion for function arguments
def merge_lists(a, b):
    assert isinstance(a, list), f"Expected list, got {type(a).__name__}"
    assert isinstance(b, list), f"Expected list, got {type(b).__name__}"
    return a + b

# Assertion in a loop
def process_items(items):
    assert len(items) > 0, "Items list is empty"
    for item in items:
        assert item is not None, "Item cannot be None"
        print(f"Processing {item}")

# Asserting math invariants
def factorial(n):
    assert n >= 0, f"Factorial not defined for negative: {n}"
    if n <= 1:
        return 1
    return n * factorial(n - 1)

# Assertion in a class
class BankAccount:
    def __init__(self, balance):
        assert balance >= 0, f"Initial balance cannot be negative: {balance}"
        self.balance = balance

    def withdraw(self, amount):
        assert amount > 0, f"Withdrawal amount must be positive: {amount}"
        assert amount <= self.balance, "Insufficient funds"
        self.balance -= amount
        return self.balance
```

### Intermediate Examples

```python
# Assertions for pre-conditions
import math

def calculate_sqrt(x):
    assert x >= 0, f"Cannot compute sqrt of negative number: {x}"
    result = math.sqrt(x)
    assert result >= 0, f"Post-condition failed: sqrt({x}) = {result} < 0"
    return result

# Assertions for immutable state
class Point:
    def __init__(self, x, y):
        self._x = x
        self._y = y
        self._frozen = False

    def freeze(self):
        self._frozen = True

    def move(self, dx, dy):
        assert not self._frozen, "Cannot modify a frozen Point"
        self._x += dx
        self._y += dy

p = Point(1, 2)
p.move(3, 4)
p.freeze()
# p.move(1, 1)  # AssertionError: Cannot modify a frozen Point

# Assertions in recursive functions
def binary_search(arr, target, low, high):
    assert low <= high, f"Invalid bounds: low={low}, high={high}"
    assert arr == sorted(arr), "Array must be sorted for binary search"

    if low > high:
        return -1
    mid = (low + high) // 2
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        return binary_search(arr, target, mid + 1, high)
    else:
        return binary_search(arr, target, low, mid - 1)

# Assertions for balanced parentheses
def is_balanced(s):
    stack = []
    pairs = {")": "(", "]": "[", "}": "{"}
    for char in s:
        if char in "([{":
            stack.append(char)
        elif char in ")]}":
            assert stack, f"Unmatched closing bracket: {char}"
            top = stack.pop()
            assert pairs[char] == top, f"Mismatched: {top} and {char}"
    assert not stack, f"Unmatched opening brackets: {stack}"
    return True
```

### Advanced Examples

```python
# Contract programming decorator
class Contract:
    @staticmethod
    def precondition(cond_func):
        def decorator(func):
            def wrapper(*args, **kwargs):
                assert cond_func(*args, **kwargs), \
                    f"Precondition failed for {func.__name__}"
                return func(*args, **kwargs)
            return wrapper
        return decorator

    @staticmethod
    def postcondition(cond_func):
        def decorator(func):
            def wrapper(*args, **kwargs):
                result = func(*args, **kwargs)
                assert cond_func(result), \
                    f"Postcondition failed for {func.__name__}"
                return result
            return wrapper
        return decorator

@Contract.precondition(lambda self, amount: amount > 0)
@Contract.postcondition(lambda result: result >= 0)
def withdraw(self, amount):
    self.balance -= amount
    return self.balance

# State machine validation
class StateMachine:
    def __init__(self):
        self.state = "IDLE"
        self._allowed = {
            "IDLE": ["RUNNING", "STOPPED"],
            "RUNNING": ["PAUSED", "STOPPED"],
            "PAUSED": ["RUNNING", "STOPPED"],
            "STOPPED": ["IDLE"],
        }

    def transition(self, new_state):
        allowed = self._allowed.get(self.state, [])
        assert new_state in allowed, \
            f"Illegal: {self.state} -> {new_state}. Allowed: {allowed}"
        self.state = new_state

# Using __debug__ for expensive checks
class DataProcessor:
    def __init__(self, data):
        self.data = data
        if __debug__:
            self._validate_data()

    def _validate_data(self):
        assert isinstance(self.data, list), "Data must be a list"
        assert all(isinstance(x, (int, float)) for x in self.data), \
            "All elements must be numbers"
        assert len(self.data) > 0, "Data cannot be empty"

# Decorator for debug-only functions
def debug_only(func):
    def wrapper(*args, **kwargs):
        if __debug__:
            return func(*args, **kwargs)
    return wrapper

@debug_only
def check_heap_invariant(heap):
    for i in range(len(heap)):
        left = 2 * i + 1
        right = 2 * i + 2
        if left < len(heap):
            assert heap[i] <= heap[left], f"Heap invariant at {i}"
        if right < len(heap):
            assert heap[i] <= heap[right], f"Heap invariant at {i}"
```

## Debugging usage

### What It Is

Assertions are primarily a debugging tool. They verify assumptions during development and testing, catching bugs as early as possible. When used effectively, assertions reduce debugging time by ensuring that incorrect state is detected at the point where it's introduced, not where it causes symptoms.

### Why It Is Important

Without assertions, incorrect state silently propagates through the program, causing confusing failures far from the actual bug. Assertions fail at the source of the problem, making bugs easier to find and fix. They also serve as active documentation—tests that run on every execution.

### Beginner Examples

```python
# Debugging assumptions
def process_order(order):
    assert order is not None
    assert "items" in order
    assert len(order["items"]) > 0
    # Process order...

# Sanity check after computation
def sort_numbers(nums):
    result = sorted(nums)
    for i in range(len(result) - 1):
        assert result[i] <= result[i + 1], "Sorting failed"
    return result

# Invariant checking in loops
total = 0
for i, item in enumerate(items):
    total += item.price
    assert total >= 0, "Total went negative!"
    assert i < len(items), "Loop index exceeded!"
```

### Advanced Examples

```python
# Debug assertions in data structures
class MinHeap:
    def __init__(self):
        self.data = []

    def push(self, value):
        self.data.append(value)
        self._sift_up(len(self.data) - 1)
        if __debug__:
            self._check_invariant()

    def pop(self):
        assert self.data, "Cannot pop from empty heap"
        result = self.data[0]
        if len(self.data) > 1:
            self.data[0] = self.data.pop()
            self._sift_down(0)
        else:
            self.data.pop()
        if __debug__:
            self._check_invariant()
        return result

    def _check_invariant(self):
        for i in range(len(self.data)):
            left = 2 * i + 1
            right = 2 * i + 2
            if left < len(self.data):
                assert self.data[i] <= self.data[left]
            if right < len(self.data):
                assert self.data[i] <= self.data[right]
```

## Contract checking

### What It Is

Design-by-contract uses assertions to enforce formal contracts: preconditions (what callers must guarantee), postconditions (what the function guarantees), and invariants (what must always be true). Assertions are the primary tool for implementing contracts in Python.

### Why It Is Important

Contracts make interfaces explicit and verifiable. They document requirements in executable form, catching contract violations immediately. In large systems with many components, contracts catch integration bugs early and make API boundaries clear.

### Beginner Examples

```python
# Precondition: what callers must provide
def connect_to_server(host, port):
    assert isinstance(host, str), f"host must be string, got {type(host).__name__}"
    assert 0 < port < 65536, f"port must be 1-65535, got {port}"
    return f"Connected to {host}:{port}"

# Postcondition: what function guarantees
def compute_average(numbers):
    assert len(numbers) > 0, "Cannot average empty list"
    result = sum(numbers) / len(numbers)
    assert min(numbers) <= result <= max(numbers), \
        "Average must be between min and max"
    return result

# Invariant: object state always valid
class Rectangle:
    def __init__(self, width, height):
        assert width > 0 and height > 0
        self.width = width
        self.height = height

    def area(self):
        result = self.width * self.height
        assert result > 0, f"Area invariant violated: {result}"
        return result
```

### Advanced Examples

```python
# Full contract decorators
def preconditions(*checks):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for check in checks:
                assert check(*args, **kwargs), \
                    f"Precondition failed for {func.__name__}"
            return func(*args, **kwargs)
        return wrapper
    return decorator

# Invariant class decorator
def invariant(check_func):
    def decorator(cls):
        original_init = cls.__init__
        def new_init(self, *args, **kwargs):
            original_init(self, *args, **kwargs)
            assert check_func(self), f"Invariant failed after init"
        cls.__init__ = new_init

        for name in dir(cls):
            attr = getattr(cls, name)
            if callable(attr) and not name.startswith("_"):
                original = attr
                def make_wrapper(method, name=name):
                    def wrapper(self, *args, **kwargs):
                        result = method(self, *args, **kwargs)
                        assert check_func(self), \
                            f"Invariant failed after {name}"
                        return result
                    return wrapper
                setattr(cls, name, make_wrapper(original))
        return cls
    return decorator

@invariant(lambda self: self.balance >= 0)
class BankAccount:
    def __init__(self, balance):
        self.balance = balance

    def withdraw(self, amount):
        assert amount > 0, "Amount must be positive"
        self.balance -= amount
        return self.balance
```

### Real-World Use Cases

```python
# API response validation
def call_api(endpoint, expected_status=200):
    response = {"status": 200, "body": {"users": []}}
    assert isinstance(response, dict), "Response must be dict"
    assert "status" in response, "Missing status"
    assert response["status"] == expected_status, \
        f"Expected {expected_status}, got {response['status']}"
    return response["body"]

# ETL assertions
def etl_pipeline(source_data):
    assert source_data is not None, "Source cannot be None"
    assert len(source_data) > 0, "Source is empty"

    transformed = [row for row in source_data if row.get("active")]
    assert len(transformed) <= len(source_data), "Transform should not add rows"
    assert all("id" in row for row in transformed), "All rows must have ID"

    for row in transformed:
        assert row["id"] not in (None, ""), "ID cannot be empty"
        print(f"Loading row {row['id']}")

    return transformed

# Config validation
def load_config(config_path):
    assert config_path.endswith(".json"), f"Must be JSON: {config_path}"
    import json
    with open(config_path) as f:
        config = json.load(f)
    assert "database" in config, "Missing database section"
    assert "host" in config["database"], "Missing host"
    assert 0 < config["database"]["port"] < 65536, "Invalid port"
    return config
```

### Common Mistakes

```python
# Mistake 1: Using assert for user input validation
def get_user_age():
    age = int(input("Enter age: "))
    assert 0 <= age <= 150, "Invalid age"  # BAD: disabled with -O!
    return age

# CORRECT:
def get_user_age_correct():
    age = int(input("Enter age: "))
    if not (0 <= age <= 150):
        raise ValueError(f"Invalid age: {age}")
    return age

# Mistake 2: Side effects in assertions
def pop_item(stack):
    assert stack.pop() is not None, "Item was None"  # BAD
    return stack
# With -O, stack.pop() is never called!

# Mistake 3: The parentheses trap
assert (1 == 2, "This message is in a tuple")
# The tuple (False, "message") is truthy, so this NEVER fails!

assert 1 == 2, "This message is a string"  # Correct

# Mistake 4: Using assert for type checking in public APIs
def process(data):
    assert isinstance(data, list), "data must be a list"  # BAD
    # Use type hints + if/raise instead

# Mistake 5: Silently swallowing assertion errors
try:
    assert False, "This should fail"
except AssertionError:
    pass  # BAD: hides bugs

# Mistake 6: Expensive operations in assertions
assert expensive_validation(data), "Validation failed"
# With -O, expensive_validation is never called (maybe good or bad)
```

### Best Practices

- Use assertions for internal invariants, not user input validation
- Always provide descriptive error messages
- Assert preconditions, postconditions, and invariants
- Never use assertions with side effects
- Avoid the parentheses trap: `assert x, "msg"` not `assert (x, "msg")`
- Use `if __debug__:` for expensive debug-only checks
- Don't catch `AssertionError` in production code
- Use assertions in tests, but prefer `self.assertEqual()` for clearer messages
- Run tests with `-O` flag occasionally to check for side-effect dependencies
- Layer assertions with type hints (static + runtime checking)

### Performance Considerations

Assertions have zero cost when disabled (the expression is never evaluated). When enabled, the condition evaluation cost depends on the expression. In performance-critical code:
- Avoid expensive function calls in assertions (they run every time)
- Use `if __debug__:` for multi-statement validation blocks
- Profile hot paths with assertions enabled vs disabled
- The `assert` bytecode is very fast for simple comparisons

### Interview Questions

1. What is the difference between `assert` and `raise`?

   `assert` is for catching programmer errors during development and can be disabled with `-O`. `raise` is for handling expected error conditions and always executes.

2. How do you disable assertions in production?

   Run Python with `-O` flag: `python -O script.py`. This sets `__debug__` to `False` and removes all `assert` statements at compile time.

3. What is the parentheses trap with `assert`?

   `assert (condition, message)` creates a tuple which is always truthy, so the assertion never fails. Correct: `assert condition, message`.

4. Why avoid side effects in assertions?

   When assertions are disabled with `-O`, the side effect never executes, potentially changing program behavior silently.

5. When should you use `if __debug__:` instead of `assert`?

   When the check involves expensive computation or multiple statements. The `if __debug__:` block allows complex debug-only logic.

6. How do assertions relate to contract programming?

   Assertions implement design-by-contract: preconditions (assert on input), postconditions (assert on output), and invariants (assert on object state).

### Coding Challenges

1. Implement bubble sort with assertions that the array is sorted after each pass.

2. Create a bounded stack with assertions for: push fails if full, pop fails if empty, internal array length never exceeds capacity.

3. Write a matrix multiplication function with assertions that dimensions match and result dimensions are correct.

4. Implement a binary search tree with assertions that check the BST invariant after every insert/delete.

5. Build a test runner that executes functions with `assert` statements, reports pass/fail, and continues after individual assertion failures.

### Related Topics

- `43_exceptions.md` - Exception raising and handling
- `44_try_except.md` - Try/except blocks
- `45_custom_exceptions.md` - Custom exceptions
- `46_logging.md` - Logging for debugging
- Python's `-O` and `-OO` optimization flags
- `unittest.TestCase` assertion methods
- `pytest` assertion introspection
- Design by contract (DbC) methodology
- Type hints for static contract checking
