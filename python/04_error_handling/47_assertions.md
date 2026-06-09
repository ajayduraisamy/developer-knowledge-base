# Assertions - assert statement, debugging, contract checking

## Introduction

An assertion is a debugging and validation tool that checks whether a condition holds true at a specific point in a program's execution. In Python, the `assert` statement evaluates an expression and raises `AssertionError` if it is `False`. Assertions serve as a form of executable documentation—they declare invariants that must hold, and they actively verify those invariants during development. Unlike regular exception handling, assertions are designed to catch programmer errors (bugs), not runtime or user errors. They are typically disabled in production for performance reasons.

## Why It Is Important

Assertions catch bugs early during development by verifying assumptions about program state. They act as self-checking documentation—anyone reading the code can see what conditions are assumed to be true at that point. Assertions make debugging faster by failing close to the root cause of a problem, rather than letting incorrect state propagate to produce a confusing error later. They are a cornerstone of defensive programming and are heavily used in test-driven development and contract programming. Knowing when and how to use assertions (versus `if`/`raise` for production checks) is a hallmark of an experienced Python developer.

## Syntax

Basic assertion syntax:

```python
assert condition
assert condition, "Optional error message"
```

When the condition is `True`, nothing happens. When `False`, `AssertionError` is raised:

```python
x = -1
assert x >= 0, f"x must be non-negative, got {x}"
# AssertionError: x must be non-negative, got -1
```

Assertions can be disabled globally with the `-O` (optimize) flag:

```bash
python -O script.py
```

When disabled, all `assert` statements are compiled to `None` (no-op). You can check the state with `__debug__`:

```python
if __debug__:
    print("Assertions are enabled")
```

## Examples

```python
# Example 1: Simple assertion
def divide(a, b):
    assert b != 0, "Division by zero"
    return a / b


print(divide(10, 2))  # Works
# print(divide(10, 0))  # AssertionError


# Example 2: Assertion with tuple condition (CAUTION!)
# WRONG - this always passes because a non-empty tuple is truthy:
assert (1 == 2, "This assertion never fails!")
# The condition is the tuple (False, "message"), which is truthy

# CORRECT - use comma carefully or separate:
assert 1 == 2, "This assertion works correctly"


# Example 3: Multiple conditions
def validate_user(user):
    assert user is not None, "User cannot be None"
    assert "name" in user, "User must have a name"
    assert "age" in user, "User must have an age"
    assert user["age"] >= 0, "Age cannot be negative"
    return True


# Example 4: Assertion for data structure invariants
def pop_from_stack(stack):
    assert stack, "Cannot pop from an empty stack"
    return stack.pop()


# Example 5: Assertion for function arguments (development only)
def merge_lists(a, b):
    assert isinstance(a, list), f"Expected list, got {type(a).__name__}"
    assert isinstance(b, list), f"Expected list, got {type(b).__name__}"
    return a + b


# Example 6: Checking __debug__ flag
print(f"__debug__ is {__debug__}")


# Example 7: Assertion in a loop
def process_items(items):
    assert len(items) > 0, "Items list is empty"
    for item in items:
        assert item is not None, "Item cannot be None"
        print(f"Processing {item}")


process_items(["a", "b", None])  # AssertionError on the None item
```

## Beginner Examples

```python
# Beginner Example 1: Asserting a math invariant
def factorial(n):
    assert n >= 0, f"Factorial is not defined for negative numbers: {n}"
    if n <= 1:
        return 1
    return n * factorial(n - 1)


print(factorial(5))  # 120
# print(factorial(-3))  # AssertionError


# Beginner Example 2: Assertion with comparison
def is_even(num):
    assert isinstance(num, int), f"Expected int, got {type(num).__name__}"
    return num % 2 == 0


print(is_even(4))  # True
# print(is_even("hello"))  # AssertionError


# Beginner Example 3: Asserting list length
def get_first(items):
    assert len(items) > 0, "Cannot get first element of empty list"
    return items[0]


print(get_first([1, 2, 3]))  # 1
# print(get_first([]))  # AssertionError


# Beginner Example 4: Assertion with string check
def greet(name):
    assert name, "Name cannot be empty"
    return f"Hello, {name}!"


print(greet("Alice"))  # Hello, Alice!
# print(greet(""))  # AssertionError


# Beginner Example 5: Asserting type
def add(x, y):
    assert isinstance(x, (int, float)), "x must be a number"
    assert isinstance(y, (int, float)), "y must be a number"
    return x + y


print(add(3, 4.5))  # 7.5
# print(add("3", 4))  # AssertionError


# Beginner Example 6: Simple assertion in a class
class BankAccount:
    def __init__(self, balance):
        assert balance >= 0, f"Initial balance cannot be negative: {balance}"
        self.balance = balance

    def withdraw(self, amount):
        assert amount > 0, f"Withdrawal amount must be positive: {amount}"
        assert amount <= self.balance, "Insufficient funds"
        self.balance -= amount
        return self.balance


account = BankAccount(100)
account.withdraw(30)
print(account.balance)  # 70
# account.withdraw(200)  # AssertionError: Insufficient funds


# Beginner Example 7: Assertion in a sorting check
def sort_and_assert(numbers):
    sorted_nums = sorted(numbers)
    for i in range(len(sorted_nums) - 1):
        assert sorted_nums[i] <= sorted_nums[i + 1], "Sorting failed"
    return sorted_nums


result = sort_and_assert([3, 1, 4, 1, 5])
print(result)  # [1, 1, 3, 4, 5]
```

## Intermediate Examples

```python
# Intermediate Example 1: Assertions for data integrity
def process_transaction(transaction):
    assert isinstance(transaction, dict), "Transaction must be a dict"
    assert "amount" in transaction, "Transaction missing amount"
    assert "currency" in transaction, "Transaction missing currency"
    assert transaction["amount"] > 0, f"Invalid amount: {transaction['amount']}"
    assert transaction["currency"] in ("USD", "EUR", "GBP"), \
        f"Unsupported currency: {transaction['currency']}"

    # Process transaction...
    return {"status": "completed", "id": 12345}


# transaction = {"amount": -50, "currency": "USD"}
# process_transaction(transaction)  # AssertionError


# Intermediate Example 2: Assertions for post-conditions
import math


def calculate_sqrt(x):
    assert x >= 0, f"Cannot compute sqrt of negative number: {x}"
    result = math.sqrt(x)
    assert result >= 0, f"Post-condition failed: sqrt({x}) = {result} < 0"
    return result


print(calculate_sqrt(9))  # 3.0


# Intermediate Example 3: Asserting immutable state
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

    @property
    def x(self):
        return self._x

    @property
    def y(self):
        return self._y


p = Point(1, 2)
p.move(3, 4)
print(p.x, p.y)  # 4, 6
p.freeze()
# p.move(1, 1)  # AssertionError: Cannot modify a frozen Point


# Intermediate Example 4: Assertions in recursive functions
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


arr = [1, 3, 5, 7, 9, 11]
print(binary_search(arr, 7, 0, len(arr) - 1))  # 3
print(binary_search(arr, 4, 0, len(arr) - 1))  # -1


# Intermediate Example 5: Assertions with custom messages and data
def connect_to_server(host, port, timeout=30):
    assert isinstance(host, str), \
        f"host must be a string, got {type(host).__name__}"
    assert 0 < port < 65536, \
        f"port must be between 1 and 65535, got {port}"
    assert timeout > 0, \
        f"timeout must be positive, got {timeout}"

    # Simulate connection
    return f"Connected to {host}:{port}"


print(connect_to_server("localhost", 8080))
# print(connect_to_server("localhost", 99999))  # AssertionError


# Intermediate Example 6: Assertions for balanced parentheses
def is_balanced(s):
    stack = []
    pairs = {")": "(", "]": "[", "}": "{"}
    for char in s:
        if char in "([{":
            stack.append(char)
        elif char in ")]}":
            assert stack, f"Unmatched closing bracket: {char}"
            top = stack.pop()
            assert pairs[char] == top, \
                f"Mismatched brackets: {top} and {char}"

    assert not stack, f"Unmatched opening brackets remaining: {stack}"
    return True


print(is_balanced("()[]{}"))  # True
print(is_balanced("({[]})"))  # True
# is_balanced("({)}")  # AssertionError: Mismatched brackets: { and )
```

## Advanced Examples

```python
# Advanced Example 1: Contract programming with assertions
class Contract:
    """Simple contract programming decorator using assertions."""

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

    @staticmethod
    def invariant(cond_func):
        def decorator(cls):
            original_init = cls.__init__

            def new_init(self, *args, **kwargs):
                original_init(self, *args, **kwargs)
                assert cond_func(self), \
                    f"Invariant failed after __init__ for {cls.__name__}"

            cls.__init__ = new_init

            for attr_name in dir(cls):
                attr = getattr(cls, attr_name)
                if callable(attr) and not attr_name.startswith("_"):
                    original_method = attr

                    def make_wrapper(method):
                        def wrapper(self, *args, **kwargs):
                            result = method(self, *args, **kwargs)
                            assert cond_func(self), \
                                f"Invariant failed after {method.__name__} " \
                                f"for {cls.__name__}"
                            return result
                        return wrapper

                    setattr(cls, attr_name, make_wrapper(original_method))
            return cls
        return decorator


@Contract.invariant(lambda self: self.balance >= 0)
class BankAccount:
    def __init__(self, balance):
        self.balance = balance

    @Contract.precondition(lambda self, amount: amount > 0)
    @Contract.postcondition(lambda result: result >= 0)
    def withdraw(self, amount):
        self.balance -= amount
        return self.balance

    @Contract.precondition(lambda self, amount: amount > 0)
    def deposit(self, amount):
        self.balance += amount
        return self.balance


account = BankAccount(100)
account.deposit(50)
print(f"Balance after deposit: {account.balance}")  # 150
account.withdraw(30)
print(f"Balance after withdrawal: {account.balance}")  # 120
# account.withdraw(200)  # AssertionError: Invariant failed


# Advanced Example 2: Assertion-based state machine validation
class StateMachine:
    def __init__(self):
        self.state = "IDLE"
        self._allowed_transitions = {
            "IDLE": ["RUNNING", "STOPPED"],
            "RUNNING": ["PAUSED", "STOPPED"],
            "PAUSED": ["RUNNING", "STOPPED"],
            "STOPPED": ["IDLE"],
        }

    def transition(self, new_state):
        allowed = self._allowed_transitions.get(self.state, [])
        assert new_state in allowed, \
            f"Illegal transition: {self.state} -> {new_state}. " \
            f"Allowed: {allowed}"
        old_state = self.state
        self.state = new_state
        print(f"{old_state} -> {new_state}")

    def start(self): self.transition("RUNNING")
    def pause(self): self.transition("PAUSED")
    def resume(self): self.transition("RUNNING")
    def stop(self): self.transition("STOPPED")
    def reset(self): self.transition("IDLE")


sm = StateMachine()
sm.start()   # IDLE -> RUNNING
sm.pause()   # RUNNING -> PAUSED
sm.resume()  # PAUSED -> RUNNING
sm.stop()    # RUNNING -> STOPPED
# sm.pause()  # AssertionError: Illegal transition: STOPPED -> PAUSED


# Advanced Example 3: Using __debug__ for expensive checks
import time


class DataProcessor:
    def __init__(self, data):
        self.data = data
        if __debug__:
            self._validate_data()

    def _validate_data(self):
        print("Running expensive data validation...")
        time.sleep(0.1)  # Expensive check
        assert isinstance(self.data, list), "Data must be a list"
        assert all(isinstance(x, (int, float)) for x in self.data), \
            "All elements must be numbers"
        assert len(self.data) > 0, "Data cannot be empty"
        print("Validation passed")

    def process(self):
        return [x * 2 for x in self.data]


# With -O flag, _validate_data is never called
dp = DataProcessor([1, 2, 3])
print(dp.process())

# When assertions are disabled:
# python -O -c "
# exec(open('script.py').read())
# "


# Advanced Example 4: Abstract class enforcement with assertions
class AbstractShape:
    def area(self):
        raise NotImplementedError

    def perimeter(self):
        raise NotImplementedError


class Rectangle(AbstractShape):
    def __init__(self, width, height):
        assert width > 0, f"Width must be positive: {width}"
        assert height > 0, f"Height must be positive: {height}"
        self.width = width
        self.height = height

    def area(self):
        result = self.width * self.height
        assert result > 0, f"Area invariant violated: {result}"
        return result

    def perimeter(self):
        result = 2 * (self.width + self.height)
        assert result > 0, f"Perimeter invariant violated: {result}"
        return result


rect = Rectangle(3, 4)
print(f"Area: {rect.area()}, Perimeter: {rect.perimeter()}")
# rect = Rectangle(-1, 5)  # AssertionError: Width must be positive


# Advanced Example 5: Assertion-based debug assertions like C's assert.h
def debug_only(func):
    """Decorator that only runs the function when __debug__ is True."""
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
            assert heap[i] <= heap[left], \
                f"Heap invariant violated at index {i}"
        if right < len(heap):
            assert heap[i] <= heap[right], \
                f"Heap invariant violated at index {i}"


class MinHeap:
    def __init__(self):
        self.data = []

    def push(self, value):
        self.data.append(value)
        self._sift_up(len(self.data) - 1)
        if __debug__:
            check_heap_invariant(self.data)

    def pop(self):
        assert self.data, "Cannot pop from empty heap"
        result = self.data[0]
        if len(self.data) > 1:
            self.data[0] = self.data.pop()
            self._sift_down(0)
        else:
            self.data.pop()
        if __debug__:
            check_heap_invariant(self.data)
        return result

    def _sift_up(self, idx):
        while idx > 0:
            parent = (idx - 1) // 2
            if self.data[idx] >= self.data[parent]:
                break
            self.data[idx], self.data[parent] = self.data[parent], self.data[idx]
            idx = parent

    def _sift_down(self, idx):
        n = len(self.data)
        while True:
            smallest = idx
            left = 2 * idx + 1
            right = 2 * idx + 2
            if left < n and self.data[left] < self.data[smallest]:
                smallest = left
            if right < n and self.data[right] < self.data[smallest]:
                smallest = right
            if smallest == idx:
                break
            self.data[idx], self.data[smallest] = \
                self.data[smallest], self.data[idx]
            idx = smallest


heap = MinHeap()
for v in [5, 3, 7, 1, 9, 2]:
    heap.push(v)
print(f"Heap: {heap.data}")
print(heap.pop())  # 1
print(heap.pop())  # 2
print(heap.pop())  # 3
```

## Real-World Use Cases

Assertions are used extensively in:
- **Test suites** — verifying expected outcomes (though `unittest.TestCase.assertEqual()` etc. are preferred over bare `assert`)
- **Build and CI pipelines** — with `-O` disabled, assertions validate invariants during testing
- **Scientific computing** — asserting matrix dimensions, probability distributions, conservation laws
- **Game development** — verifying entity states, collision geometry, resource constraints
- **Network protocol implementations** — checking packet structure, sequence numbers, checksums
- **Embedded/real-time systems** — timing constraints and memory bounds

```python
# Real-world: API response validation with assertions
import json


def call_api(endpoint, expected_status=200):
    # Simulate API call
    response = {"status": 200, "body": {"users": [{"id": 1, "name": "Alice"}]}}

    assert isinstance(response, dict), "Response must be a dict"
    assert "status" in response, "Response missing status code"
    assert response["status"] == expected_status, \
        f"Expected status {expected_status}, got {response['status']}"
    assert "body" in response, "Response missing body"

    return response["body"]


# Real-world: Data pipeline assertions
def etl_pipeline(source_data):
    # Extract
    assert source_data is not None, "Source data cannot be None"
    assert len(source_data) > 0, "Source data is empty"

    # Transform
    transformed = [row for row in source_data if row.get("active")]

    assert len(transformed) <= len(source_data), \
        "Transform should not increase row count"
    assert all("id" in row for row in transformed), \
        "All transformed rows must have an ID"

    # Load
    for row in transformed:
        assert row["id"] not in (None, ""), "ID cannot be empty"
        print(f"Loading row {row['id']}")

    return transformed


data = [{"id": 1, "active": True}, {"id": 2, "active": False}]
etl_pipeline(data)


# Real-world: Configuration validation
def load_config(config_path):
    assert config_path.endswith(".json"), \
        f"Config must be JSON file: {config_path}"

    with open(config_path) as f:
        config = json.load(f)

    assert "database" in config, "Config missing 'database' section"
    assert "host" in config["database"], "Database config missing host"
    assert "port" in config["database"], "Database config missing port"
    assert 0 < config["database"]["port"] < 65536, \
        f"Invalid port: {config['database']['port']}"

    return config
```

## Common Mistakes

1. **Using assertions for data validation in production** — assertions can be disabled with `-O`; use `if`/`raise` for input validation that must always run.
2. **Using assertions with side effects** — when assertions are disabled, the side effect never happens.
3. **Parentheses trap** — `assert (condition, message)` is always truthy because it's a tuple.
4. **Over-relying on assertions** — they are for catching bugs, not for handling expected error conditions.
5. **Expensive operations in assertions** — if the expensive operation is disabled along with assertions, you mask performance issues; if it stays, you waste CPU.
6. **Not providing descriptive messages** — bare `assert x` gives no context when it fails.

```python
import logging

# Mistake 1: Using assert for user input validation
def get_user_age():
    age = int(input("Enter age: "))
    assert 0 <= age <= 150, "Invalid age"  # BAD: can be disabled!
    return age


# CORRECT: Use if/raise for production validation
def get_user_age_correct():
    age = int(input("Enter age: "))
    if not (0 <= age <= 150):
        raise ValueError(f"Invalid age: {age}")
    return age


# Mistake 2: Side effect in assertion
def pop_item(stack):
    # BAD: if assertions are disabled, the pop never happens
    assert stack.pop() is not None, "Stack item was None"
    return stack


stack = [1, 2, 3]
# When -O is used, stack.pop() is never called!


# Mistake 3: The parentheses trap
assert (1 == 2, "This message is in a tuple")
# The tuple (False, "message") is truthy, so this NEVER fails!

# Use a string literal instead of a tuple
assert 1 == 2, "This message is a string"
# This correctly raises AssertionError


# Mistake 4: Using assert for type checking in public APIs
def process(data):
    assert isinstance(data, list), "data must be a list"  # BAD
    # Use type hints + if/raise instead
    ...


# Mistake 5: Silence swallowing
try:
    assert False, "This should fail"
except AssertionError:
    pass  # BAD: silently swallowing assertion errors hides bugs


# Mistake 6: Assertion with side effect in expression
items = []
# BAD: if assertions disabled, append never happens
assert items.append("new_item") is None, "Append failed"
print(items)  # [] when -O is used
```

## Best Practices

1. **Use assertions for internal invariants, not user input validation** — use `if`/`raise ValueError()` for user-facing checks.
2. **Always provide a descriptive error message** — makes debugging much faster.
3. **Use assertions to document and enforce preconditions, postconditions, and invariants** — especially in complex algorithms and data structures.
4. **Never use assertions with side effects** — the side effect disappears when `-O` is used.
5. **Avoid the parentheses trap** — `assert (x, "msg")` is a tuple; write `assert x, "msg"`.
6. **Use `__debug__` guard for expensive debug-only checks** — wrap heavy validation in `if __debug__:` blocks.
7. **Don't catch `AssertionError` in production code** — if an assertion fires, the program is in an unexpected state and should fail loudly.
8. **Use assertions in tests** — but prefer dedicated test assertions (`self.assertEqual()`, etc.) for clearer failure messages.
9. **Run tests with `-O` flag occasionally** — to ensure no critical code depends on assertion side effects.
10. **Layer assertions with type hints** — type hints provide static checking; assertions provide runtime verification.

```python
from typing import List, Optional


def binary_search(arr: List[int], target: int) -> Optional[int]:
    # Precondition: arr must be sorted
    assert all(arr[i] <= arr[i + 1] for i in range(len(arr) - 1)), \
        "Array must be sorted for binary search"

    low, high = 0, len(arr) - 1
    while low <= high:
        mid = (low + high) // 2
        if arr[mid] == target:
            # Postcondition: result is in valid range
            assert 0 <= mid < len(arr), f"Index {mid} out of bounds"
            return mid
        elif arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1

    return None


# Test with assertions enabled
result = binary_search([1, 3, 5, 7, 9], 5)
print(f"Found at index: {result}")

# When running with -O:
# python -O -c "from script import *; print(binary_search([1,3,5,7,9], 5))"
# The assertion is removed but the function still works correctly
```

## Interview Questions

**Q1: What is the difference between `assert` and `raise`?**
A: `assert` is for catching programmer errors during development and can be disabled with `-O`. `raise` is for handling expected error conditions (invalid input, unavailable resources) and always executes.

**Q2: How do you disable assertions in production?**
A: Run Python with the `-O` (optimize) flag: `python -O script.py`. This sets `__debug__` to `False` and removes all `assert` statements at compile time.

**Q3: What is the parentheses trap with `assert`?**
A: `assert (condition, message)` creates a tuple `(condition, message)` which is always truthy, so the assertion never fails. The correct syntax is `assert condition, message` without parentheses wrapping both arguments.

**Q4: Why should you avoid side effects in assertions?**
A: When assertions are disabled with `-O`, the side effect never executes, potentially changing program behavior. For example, `assert my_list.pop() is not None` would silently stop mutating the list.

**Q5: When should you use `if __debug__:` instead of `assert`?**
A: When the check involves expensive computation or multiple statements. The `if __debug__:` block allows complex debug-only logic, while `assert` is for single-condition checks.

**Q6: How do assertions relate to contract programming?**
A: Assertions are the primary tool for implementing design by contract in Python. Preconditions (assert on input), postconditions (assert on output), and invariants (assert on object state) can all be expressed with assertions, ensuring a function's contract is honored.

## Coding Challenges

**Challenge 1: Sorting Algorithm with Invariant Checking**
Implement bubble sort with assertions that the array is sorted after each pass. Use assertions for preconditions (list of comparable items) and postconditions (result is sorted, same length, same elements).

**Challenge 2: Stack with Overflow/Underflow Guards**
Create a bounded stack with assertions for: `push` fails if full, `pop` fails if empty, internal array length never exceeds capacity, and peek always returns the top element.

**Challenge 3: Matrix Multiplication with Dimension Checks**
Write a matrix multiplication function with assertions that: both inputs are 2D lists, columns of A equal rows of B, all rows have consistent lengths, and the result dimensions match expectations.

**Challenge 4: Binary Tree Invariant Validator**
Implement a binary search tree with assertions that check the BST invariant after every insert and delete operation, verifying that all left descendants are less than the node and all right descendants are greater.

**Challenge 5: Assertion-Based Test Runner**
Write a simple test runner that executes functions containing `assert` statements, reports which assertions passed/failed, and counts total assertions checked. Wrap this so that even if one assertion fails, remaining assertions in the same test still run.

## Summary

Assertions are a lightweight, compile-time-removable mechanism for verifying program invariants during development. The `assert` statement checks a condition and raises `AssertionError` with an optional message when the condition is false. Unlike `if`/`raise` constructs, assertions are for detecting programmer errors, not handling runtime or user errors. They can be globally disabled with the `-O` command-line flag, making them zero-cost in production. Proper use of assertions follows the design-by-contract paradigm: preconditions, postconditions, and invariants. The key to effective assertions is knowing when to use them (internal invariants, debug checks, self-documenting code) and when to use proper exception handling (user input, external resources, expected failure modes).

## Related Topics

- Python's `-O` and `-OO` command-line optimization flags
- The `__debug__` global constant
- Design by contract (DbC) methodology
- `unittest.TestCase` methods (`assertEqual`, `assertTrue`, `assertRaises`, etc.)
- `pytest` assertion introspection (rewrites assert statements for better reporting)
- Exception handling (`try`/`except`/`finally`/`raise`)
- Type hints (`typing` module) for static assertion-like contracts
- `doctest` for inline test/documentation assertions
- Property-based testing (`hypothesis` library) for automated invariant discovery
