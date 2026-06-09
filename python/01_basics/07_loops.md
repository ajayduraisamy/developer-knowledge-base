# Loops - for, while, break, continue, else clause

## Introduction

Loops control the repeated execution of code blocks. Python provides two primary loop constructs: `for` (for iterating over sequences, iterators, and generators) and `while` (for repeating while a condition holds). Python also supports `break` for early exit, `continue` to skip the current iteration, and the `else` clause which executes when a loop completes normally (without `break`). 

## for loop

### What It Is
The `for` loop iterates over any iterable (list, tuple, string, dict, set, range, generator, file object). It assigns each element from the iterable to the loop variable and executes the body.

### Why It Is Important
The `for` loop is Python's primary iteration construct, used more frequently than `while`. It works with any iterable, integrates seamlessly with `range()`, `enumerate()`, `zip()`, and comprehensions, and is generally more readable and less error-prone than index-based loops.

### How It Works Internally
The `for` loop calls `iter()` on the iterable to get an iterator, then repeatedly calls `next()` on the iterator. When `StopIteration` is raised, the loop terminates. This is equivalent to:

```python
iterator = iter(iterable)
while True:
    try:
        item = next(iterator)
    except StopIteration:
        break
    # loop body
```

CPython optimizes `for` loops over built-in types (list, tuple, str) with specialized bytecode (Python 3.12+ adaptive interpreter).

### Syntax
```python
# Basic for loop
for item in iterable:
    statement(s)

# With range
for i in range(start, stop, step):
    statement(s)

# With enumerate
for index, value in enumerate(iterable):
    statement(s)

# With zip
for a, b in zip(iterable1, iterable2):
    statement(s)

# With else
for item in iterable:
    statement(s)
else:
    # Runs if no break occurred
    statement(s)
```

### Beginner Examples
```python
# Range-based loop
for i in range(5):
    print(i, end=" ")  # 0 1 2 3 4
print()

# Iterating a list
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(f"I like {fruit}")

# Iterating a string
for char in "Python":
    print(char, end=" ")  # P y t h o n
print()

# Iterating a dictionary
person = {"name": "Alice", "age": 30, "city": "NYC"}
for key, value in person.items():
    print(f"{key}: {value}")

# Using range with step
for i in range(0, 11, 2):
    print(i, end=" ")  # 0 2 4 6 8 10
print()

# Sum using loop
total = 0
for n in range(1, 101):
    total += n
print(f"Sum 1-100: {total}")  # 5050
```

### Intermediate Examples
```python
# Enumerate for index and value
colors = ["red", "green", "blue", "yellow"]
for i, color in enumerate(colors, start=1):
    print(f"{i}. {color}")

# Zip for parallel iteration
names = ["Alice", "Bob", "Charlie"]
scores = [85, 92, 78]
grades = ["B", "A", "C"]
for name, score, grade in zip(names, scores, grades):
    print(f"{name}: {score} ({grade})")

# Reversed iteration
for item in reversed([1, 2, 3, 4, 5]):
    print(item, end=" ")  # 5 4 3 2 1
print()

# Sorted iteration
for fruit in sorted(["banana", "apple", "cherry", "date"]):
    print(fruit, end=" ")
print()

# Unique iteration
for item in set([1, 2, 2, 3, 3, 4]):
    print(item, end=" ")
print()

# Filtering during iteration
numbers = range(20)
for n in numbers:
    if n % 3 != 0:
        continue
    print(n, end=" ")  # 0 3 6 9 12 15 18
print()
```

### Advanced Examples
```python
# Custom iterator
class FibonacciIterator:
    def __init__(self, limit: int):
        self.limit = limit
        self.a, self.b = 0, 1
        self.count = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.count >= self.limit:
            raise StopIteration
        self.count += 1
        self.a, self.b = self.b, self.a + self.b
        return self.a

for fib in FibonacciIterator(10):
    print(fib, end=" ")  # 1 1 2 3 5 8 13 21 34 55
print()

# Generator-based iteration (memory efficient)
def read_large_file(filepath: str):
    with open(filepath, "r") as f:
        for line in f:
            yield line.strip()

# For loop with itertools
from itertools import chain, cycle, accumulate

# Chain multiple iterables
for item in chain([1, 2, 3], ["a", "b", "c"]):
    print(item, end=" ")  # 1 2 3 a b c
print()

# Cycle indefinitely
count = 0
for item in cycle(["red", "green", "blue"]):
    print(item, end=" ")
    count += 1
    if count >= 6:
        break  # red green blue red green blue
print()

# Accumulate (running total)
for total in accumulate([1, 2, 3, 4, 5]):
    print(total, end=" ")  # 1 3 6 10 15
print()

# Performance measurement
import time

numbers = range(1_000_000)
start = time.perf_counter()
squares = [n ** 2 for n in numbers]  # List comprehension (fast C-level loop)
list_time = time.perf_counter() - start

start = time.perf_counter()
squares_loop = []
for n in numbers:
    squares_loop.append(n ** 2)
loop_time = time.perf_counter() - start

print(f"List comp: {list_time:.4f}s, Loop: {loop_time:.4f}s")
```

### Real-World Use Cases
- **Data Processing**: Iterating through CSV rows, JSON arrays, database results
- **File I/O**: Reading files line by line
- **Web Scraping**: Looping through pages of results
- **API Calls**: Paginating through API responses
- **Batch Processing**: Processing records in chunks
- **Machine Learning**: Iterating over training epochs and batches

### Common Mistakes
```python
# Mistake 1: Modifying list while iterating
numbers = [1, 2, 3, 4, 5]
for n in numbers:
    if n % 2 == 0:
        numbers.remove(n)  # Skips elements!
print(numbers)  # [1, 3, 5] — but actually [1, 3, 5, ...] unstable

# Correct: iterate over a copy
for n in numbers[:]:
    if n % 2 == 0:
        numbers.remove(n)

# Or use list comprehension
numbers[:] = [n for n in numbers if n % 2 != 0]

# Mistake 2: Forgetting that dict iteration gives keys
d = {"a": 1, "b": 2}
for item in d:
    print(item)  # "a", "b" (keys, not values)
# Correct:
for key, value in d.items(): pass

# Mistake 3: Range off-by-one
for i in range(0, 5):  # 0,1,2,3,4 (not 5!)
    pass

# Mistake 4: Loop variable leak (Python 3.x: fixed for list comp, not for loops)
for i in range(5):
    pass
print(i)  # 4 (loop variable leaks!)

# Mistake 5: Using for where while is better
# When number of iterations is unknown, use while
```

### Best Practices
- Use `for` loop over `while` when iterating over sequences
- Use `enumerate()` instead of manual index tracking
- Use `zip()` for parallel iteration
- Use `reversed()` / `sorted()` as needed
- Prefer list comprehensions over `for` loops for simple transformations
- Use generator expressions for memory-efficient iteration
- Avoid modifying the iterable during iteration
- Use descriptive loop variable names (`for student in students:`)

### Performance Considerations
- `for` loops over lists/tuples are optimized with direct iteration (no index lookup)
- List comprehensions are ~1.5-2x faster than equivalent `for` loops (C-level iteration)
- `range()` is lazy (generates values on demand, not all at once)
- Iterating over dict items with `.items()` is more efficient than iterating keys and looking up
- Generator expressions use less memory than list comprehensions
- Python 3.12+ adaptive interpreter optimizes common for-loop patterns

### Interview Questions
1. How does a `for` loop work internally in Python?
2. What is the difference between `range()` in Python 2 and Python 3?
3. How do you iterate over a dictionary? What are the different methods?
4. How do you get both index and value in a for loop?
5. How do you iterate over multiple iterables in parallel?
6. Why should you avoid modifying a list while iterating?
7. What is the difference between a list comprehension and a for loop?
8. How do you reverse iterate in Python?
9. What is the `else` clause on a for loop?
10. How can you create a custom iterable for use in for loops?

### Coding Challenges
```python
# Challenge 1: Flatten nested list
def flatten(nested):
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item

print(list(flatten([1, [2, [3, 4], 5], 6])))  # [1, 2, 3, 4, 5, 6]

# Challenge 2: Sieve of Eratosthenes
def primes_upto(limit: int):
    sieve = [True] * (limit + 1)
    sieve[0] = sieve[1] = False
    for i in range(2, int(limit ** 0.5) + 1):
        if sieve[i]:
            for j in range(i * i, limit + 1, i):
                sieve[j] = False
    return [i for i, is_prime in enumerate(sieve) if is_prime]

print(primes_upto(50))
```

### Related Topics
- `while` Loop
- `range()` Function
- `enumerate()` Function
- `zip()` Function
- Iterators and Iterables
- Generators (`yield`)
- List Comprehensions
- The `itertools` Module

## while loop

### What It Is
The `while` loop repeatedly executes a block of code as long as a condition remains truthy. It is ideal for situations where the number of iterations is not known in advance.

### Why It Is Important
`while` loops handle indefinite iteration, user input loops, polling operations, and algorithms that converge to a result. They are essential for interactive programs, game loops, and state-based processing.

### How It Works Internally
The `while` loop compiles to bytecode that evaluates the condition, uses `POP_JUMP_IF_FALSE` to exit if falsy, executes the body, then jumps back to re-evaluate the condition. The condition is re-evaluated before each iteration. If the condition never becomes falsy, the loop runs indefinitely (infinite loop).

### Syntax
```python
while condition:
    statement(s)

while condition:
    statement(s)
else:
    # Runs if condition becomes false (no break)
    statement(s)

# Sentinel pattern
while True:
    item = get_next()
    if not item:
        break
    process(item)
```

### Beginner Examples
```python
# Countdown
count = 5
while count > 0:
    print(count, end=" ")
    count -= 1
print("Blast off!")  # 5 4 3 2 1 Blast off!

# Sum until negative
total = 0
while True:
    num = int(input("Enter number (negative to stop): "))
    if num < 0:
        break
    total += num
print(f"Sum: {total}")

# User input validation
while True:
    password = input("Enter password (min 8 chars): ")
    if len(password) >= 8:
        print("Password accepted!")
        break
    print("Too short, try again.")

# Guessing game
import random
target = random.randint(1, 100)
guess = -1
attempts = 0
while guess != target:
    guess = int(input("Guess (1-100): "))
    attempts += 1
    if guess < target:
        print("Too low!")
    elif guess > target:
        print("Too high!")
print(f"Correct! Got it in {attempts} attempts")
```

### Intermediate Examples
```python
# Newton's method for square root
def sqrt_newton(n: float, tolerance: float = 1e-10) -> float:
    if n < 0:
        raise ValueError("No real sqrt for negative")
    if n == 0:
        return 0
    x = n
    while True:
        next_x = (x + n / x) / 2
        if abs(next_x - x) < tolerance:
            return next_x
        x = next_x

print(sqrt_newton(25))    # 5.0
print(sqrt_newton(2))     # 1.4142135623...

# While with else
def find_power_of_two(n: int) -> int | None:
    """Find the exponent such that 2^exp >= n."""
    exp = 0
    power = 1
    while power < n:
        power *= 2
        exp += 1
    else:
        return exp

print(find_power_of_two(10))  # 4 (2^4 = 16 >= 10)

# Polling pattern
import time

def wait_for_condition(check_func, timeout: float = 5.0, interval: float = 0.1) -> bool:
    start = time.time()
    while time.time() - start < timeout:
        if check_func():
            return True
        time.sleep(interval)
    return False

# Usage:
# wait_for_condition(lambda: data_ready(), timeout=10.0)
```

### Advanced Examples
```python
# Rate limiter with while
import time
from collections import deque

class RateLimiter:
    def __init__(self, max_calls: int, window: float):
        self.max_calls = max_calls
        self.window = window
        self.calls = deque()
    
    def acquire(self) -> bool:
        now = time.time()
        # Remove old calls outside the window
        while self.calls and self.calls[0] < now - self.window:
            self.calls.popleft()
        
        if len(self.calls) >= self.max_calls:
            return False
        
        self.calls.append(now)
        return True

limiter = RateLimiter(3, 1.0)
for _ in range(5):
    print(f"Acquired: {limiter.acquire()}")
    time.sleep(0.3)

# Producer-consumer with while
from collections import deque

class TaskQueue:
    def __init__(self):
        self.queue = deque()
    
    def produce(self, item):
        self.queue.append(item)
    
    def consume(self) -> int | None:
        """Consumer with polling."""
        while self.queue:
            yield self.queue.popleft()
        return None

queue = TaskQueue()
queue.produce(1)
queue.produce(2)
queue.produce(3)

for task in queue.consume():
    print(f"Processing: {task}")

# Event loop pattern
class EventLoop:
    def __init__(self):
        self.tasks = []
    
    def add_task(self, coro):
        self.tasks.append(coro)
    
    def run(self):
        while self.tasks:
            task = self.tasks.pop(0)
            try:
                next(task)
                self.tasks.append(task)
            except StopIteration:
                pass

def countdown(n):
    while n > 0:
        print(f"Count: {n}")
        yield
        n -= 1

loop = EventLoop()
loop.add_task(countdown(3))
loop.add_task(countdown(2))
loop.run()
```

### Real-World Use Cases
- **Interactive Input**: Menu systems, command interpreters
- **Game Loops**: Update-render cycles
- **Network Clients**: Connection retry, keep-alive polling
- **Data Processing**: Reading streams until exhausted
- **Algorithmic Convergence**: Newton's method, gradient descent
- **Rate Limiting**: Token bucket algorithms

### Common Mistakes
```python
# Mistake 1: Infinite loop (forgetting to update condition)
i = 0
while i < 10:
    print(i)
    # Missing: i += 1 — infinite loop!

# Mistake 2: Using while where for is more appropriate
i = 0
while i < len(items):
    print(items[i])
    i += 1
# Better: for item in items:

# Mistake 3: Confusing = and == in condition
# while x = 5:  # SyntaxError or infinite assignment
while x == 5:  # Correct

# Mistake 4: While True without break path
while True:
    pass  # No break — infinite!

# Mistake 5: Not handling exceptions inside while
while True:
    try:
        data = get_data()
        if not data:
            break
    except ConnectionError:
        continue  # OK, but need to avoid infinite retry loop
```

### Best Practices
- Prefer `for` loops when iterating over a known sequence
- Ensure loop condition will eventually become falsy
- Use `while True` with `break` for sentinel patterns
- Add a timeout or max iterations guard for safety
- Keep the body short; extract complex logic to functions
- Use `continue` to skip iterations gracefully
- Consider `for` with `itertools.takewhile` for some while patterns

### Performance Considerations
- `while` loops are slightly slower than `for` loops for sequence iteration
- Condition evaluation happens every iteration (keep conditions simple)
- Infinite loops consume 100% CPU per core (use `time.sleep()` in polling)
- Python 3.12+ adaptive interpreter optimizes common while patterns
- Use `while` with simple conditions for best performance

### Interview Questions
1. When would you use a `while` loop instead of a `for` loop?
2. How do you avoid infinite loops in Python?
3. What is the `else` clause on a `while` loop?
4. How do you implement a do-while loop in Python?
5. How do you poll for a condition with a timeout?
6. What is the difference between `while` in Python and do-while in C/Java?
7. How do you implement a rate limiter using while?
8. What happens to the condition variable scope in a while loop?
9. How do you handle retry logic with backoff using while?
10. What are the performance implications of while vs for loops?

### Coding Challenges
```python
# Challenge 1: Collatz conjecture
def collatz_steps(n: int) -> int:
    steps = 0
    while n != 1:
        n = n // 2 if n % 2 == 0 else 3 * n + 1
        steps += 1
    return steps

print(collatz_steps(27))  # 111

# Challenge 2: Exponential backoff retry
import time
import random

def retry_with_backoff(func, max_retries: int = 3, base_delay: float = 0.5):
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            delay = base_delay * (2 ** attempt) + random.uniform(0, 0.1)
            print(f"Attempt {attempt + 1} failed, retrying in {delay:.2f}s")
            time.sleep(delay)

# Usage:
# retry_with_backoff(lambda: risky_operation(), max_retries=5)
```

### Related Topics
- `for` Loop
- Loop Control (`break`, `continue`)
- `else` Clause on Loops
- Infinite Loops and Polling
- Exponential Backoff

## break

### What It Is
`break` immediately terminates the innermost enclosing loop (`for` or `while`), transferring execution to the statement following the loop body.

### Why It Is Important
`break` enables early exit from loops based on conditions, making code more efficient by avoiding unnecessary iterations. It is essential for search loops (found item → stop), sentinel-based loops, and nested loop control.

### How It Works Internally
`break` compiles to a `JUMP_ABSOLUTE` bytecode instruction that jumps to the address after the loop's end. In nested loops, `break` only exits the innermost loop. The presence of `break` affects the `else` clause — if `break` is executed, the `else` is skipped.

### Syntax
```python
for item in iterable:
    if condition:
        break  # Exit loop immediately
# Execution continues here after break

while condition:
    if exit_condition:
        break
```

### Beginner Examples
```python
# Search with break
numbers = [3, 7, 1, 9, 4, 6, 8, 2]
target = 9

for i, num in enumerate(numbers):
    if num == target:
        print(f"Found {target} at index {i}")
        break

# Early exit on condition
for i in range(10):
    if i == 5:
        break
    print(i, end=" ")  # 0 1 2 3 4
print("(stopped at 5)")

# Input sentinel
while True:
    command = input("Enter command (quit to exit): ")
    if command == "quit":
        break
    print(f"Executing: {command}")

# Break with else
def find_prime_above(limit):
    """Find first prime number above limit."""
    n = limit + 1
    while True:
        if all(n % i != 0 for i in range(2, int(n ** 0.5) + 1)):
            print(f"Found prime: {n}")
            break
        n += 1

find_prime_above(20)  # Found prime: 23
```

### Related Topics
- `continue` — Skip current iteration, not exit loop
- `else` Clause — Executes if no `break` occurred
- Loop Control Statements

## continue

### What It Is
`continue` skips the rest of the current iteration and proceeds to the next iteration of the innermost loop. In a `for` loop, it advances to the next item; in a `while` loop, it re-evaluates the condition.

### Why It Is Important
`continue` simplifies loop logic by allowing early skip of unwanted cases without deeply nested `if` statements. It makes code more readable by reducing indentation levels.

### How It Works Internally
`continue` compiles to a `JUMP_ABSOLUTE` that jumps back to the loop's start (for `for`, to the iterator's `next()` call; for `while`, to the condition check). It does not affect the `else` clause (unlike `break`).

### Syntax
```python
for item in iterable:
    if skip_condition:
        continue  # Skip to next iteration
    process(item)

while condition:
    if skip_condition:
        continue
    process(item)
```

### Beginner Examples
```python
# Skip even numbers
for i in range(10):
    if i % 2 == 0:
        continue
    print(i, end=" ")  # 1 3 5 7 9
print()

# Skip comments and empty lines
lines = ["data", "# comment", "", "more data", "# another"]
for line in lines:
    if not line or line.startswith("#"):
        continue
    print(f"Process: {line}")

# Skip invalid items
data = [1, None, 2, "invalid", 3, None, 4]
for item in data:
    if item is None or not isinstance(item, (int, float)):
        continue
    print(f"Valid: {item}")
```

### Related Topics
- `break` — Exit loop entirely
- `else` Clause
- Loop Control Flow

## else Clause

### What It Is
The `else` clause on loops (`for` and `while`) executes when the loop terminates normally — i.e., when the condition becomes falsy (`while`) or the iterable is exhausted (`for`), but NOT when the loop is exited via `break`.

### Why It Is Important
The `else` clause provides a clean way to detect "not found" or "completed without interruption" scenarios without maintaining boolean flags. It is especially useful for search operations and validation loops.

### How It Works Internally
The `else` clause on loops is compiled to bytecode that places the else block after the loop's exit path. If a `break` occurs, execution jumps past the else block as well. If no `break` occurs, execution falls through to the else block.

### Syntax
```python
# For-else
for item in iterable:
    if condition:
        break
else:
    # Runs only if no break occurred

# While-else
while condition:
    if exit_condition:
        break
else:
    # Runs only if condition became false (no break)
```

### Beginner Examples
```python
# For-else: search notification
def find_user(users, target_id):
    for user in users:
        if user["id"] == target_id:
            print(f"Found: {user['name']}")
            break
    else:
        print(f"User {target_id} not found")

users = [
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"},
]
find_user(users, 2)  # Found: Bob
find_user(users, 3)  # User 3 not found

# While-else
def gcd(a, b):
    """Greatest common divisor using Euclid's algorithm."""
    while b:
        a, b = b, a % b
    else:
        return a

print(gcd(48, 18))  # 6
```

### Related Topics
- `break` Statement
- `for` Loop
- `while` Loop
- `try`/`except`/`else`/`finally`
