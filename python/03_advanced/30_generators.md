# Generators - yield, generator expressions, send(), yield from

## Introduction

Generators are a cornerstone of Python's iteration protocol, enabling lazy evaluation and memory-efficient data processing. A generator function looks like a regular function but uses the `yield` keyword to return values one at a time, suspending its state between calls. This allows generators to produce sequences of values without storing them all in memory at once.

Generators are fundamental to Python's approach to streams, infinite sequences, and pipeline-based data processing. They are used extensively in the standard library, web frameworks, and data processing libraries.

## yield Keyword

### What It Is

The `yield` keyword is used in generator functions to produce a value and pause execution. When the generator is iterated over, execution resumes after the last `yield` statement. Unlike `return`, which terminates the function, `yield` suspends the function's state, allowing it to be resumed later.

### Why It Is Important

`yield` enables lazy evaluation — values are produced on-demand rather than computed all at once. This is critical for processing large datasets, infinite sequences, and streaming data where memory constraints make storing all values impractical. It also enables cooperative multitasking through coroutines when combined with `send()`.

### How It Works Internally

When Python encounters `yield` in a function, it creates a generator object instead of executing the function. The generator implements the iterator protocol with `__iter__` and `__next__`. Each call to `__next__` executes the function body until the next `yield`, returns the yielded value, and freezes the frame's local variables, instruction pointer, and stack. On the next call, execution resumes from the frozen state.

### Syntax

```python
def generator_function():
    yield value
    # More code
    yield another_value

gen = generator_function()  # Creates generator object, does NOT execute
value = next(gen)           # Executes until first yield
```

### Beginner Examples

```python
def count_up_to(n):
    count = 1
    while count <= n:
        yield count
        count += 1

counter = count_up_to(5)
for num in counter:
    print(num)  # 1, 2, 3, 4, 5

# Equivalent list (wastes memory for large n)
def count_up_to_list(n):
    return list(range(1, n + 1))
```

### Intermediate Examples

```python
def fibonacci_generator():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

fib = fibonacci_generator()
first_10 = [next(fib) for _ in range(10)]
print(first_10)  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

def read_large_file(file_path):
    with open(file_path) as f:
        for line in f:
            yield line.strip()

# Process lines without loading entire file
for line in read_large_file("huge_log.txt"):
    if "ERROR" in line:
        print(line)
```

### Advanced Examples

```python
def tree_traversal(node):
    """Depth-first traversal of a nested dict tree."""
    yield node
    for child in node.get("children", []):
        yield from tree_traversal(child)

tree = {
    "name": "root",
    "children": [
        {"name": "child1", "children": [{"name": "grandchild1"}]},
        {"name": "child2"}
    ]
}
for n in tree_traversal(tree):
    print(n["name"])
# root, child1, grandchild1, child2

def tail_file(filename, lines=10):
    """Generator that yields last N lines, then follows new lines."""
    with open(filename) as f:
        content = f.readlines()
        for line in content[-lines:]:
            yield line.strip()
        while True:
            line = f.readline()
            if line:
                yield line.strip()
            else:
                import time
                time.sleep(0.1)
```

### Real-World Use Cases

- **Streaming data processing**: Read and process CSV/JSON files line by line without loading into memory.
- **Database cursors**: Fetch rows lazily from database queries.
- **Pipelines**: Chain generators to create Unix-like pipes for data transformation.
- **Infinite sequences**: Generate Fibonacci, prime numbers, or sensor readings indefinitely.
- **Lazy evaluation**: Compute values only when needed in data science workflows.

### Common Mistakes

- Using `return value` in a generator (raises `StopIteration(value)` in Python 3, value is in exception).
- Creating a list from a generator (defeats the purpose of lazy evaluation, though sometimes needed).
- Reusing a generator (generators are exhausted after iteration, must create a new one).
- Mixing `yield` and `return` without understanding that `return` in a generator is equivalent to `raise StopIteration`.

### Best Practices

- Use generators for large or infinite sequences to save memory.
- Document whether a function returns a generator or a sequence.
- Use `yield from` to delegate to sub-generators when composing generators.
- Close generators explicitly with `.close()` when breaking early from infinite generators.
- Use generator expressions for simple transformations instead of full generator functions.

### Performance Considerations

Generators have lower memory footprint than lists because they produce items on-the-fly. However, per-item access is slightly slower due to the overhead of resuming the generator frame. For CPU-bound computations, consider whether lazy evaluation outweighs the overhead. Repeated iteration over a generator is impossible — materialize to a list if replay is needed.

### Interview Questions

**Q: What is the difference between `return` and `yield`?**

A: `return` terminates the function and sends a value to the caller. `yield` produces a value, suspends the function state, and allows resumption later. A function with `yield` is a generator function.

**Q: Can a generator have multiple `yield` statements?**

A: Yes. Each `yield` produces the next value. The generator resumes execution from the last `yield` when `next()` is called again.

**Q: What happens when a generator is garbage collected while still active?**

A: The generator's `close()` method is called by the garbage collector, which raises `GeneratorExit` inside the generator, allowing cleanup code in `finally` blocks to run.

### Coding Challenges

1. Write a generator that yields prime numbers indefinitely using the Sieve of Eratosthenes.
2. Create a `chunked(iterable, n)` generator that yields lists of `n` items from the iterable.
3. Implement a generator that produces a sliding window over a sequence.
4. Build a generator-based pipeline that reads a file, filters lines, transforms them, and writes output.

### Related Topics

- Iterators (generators implement the iterator protocol)
- Generator expressions (compact generator syntax)
- `yield from` (delegating to sub-generators)
- Coroutines (generators used with `send()` for two-way communication)
- Async generators (`async for` with `async def` and `yield`)

## Generator Expressions

### What It Is

Generator expressions are a compact syntax for creating generators without defining a function. They use parentheses `()` with the same syntax as list comprehensions but produce items lazily. A generator expression creates an anonymous generator function that yields values on demand.

### Why It Is Important

Generator expressions provide a memory-efficient alternative to list comprehensions when you don't need all elements at once. They are ideal for passing to functions that consume iterables, such as `sum()`, `min()`, `max()`, and `any()`, since they avoid creating intermediate lists.

### How It Works Internally

A generator expression `(x for x in iterable)` is compiled into a generator object. The bytecode creates a code object similar to a generator function's code, with a `YIELD_VALUE` opcode at each iteration step. The generator object is an iterator that evaluates the expression lazily for each item.

### Syntax

```python
gen_expr = (expression for item in iterable if condition)
# Equivalent to:
def gen_func():
    for item in iterable:
        if condition:
            yield expression
gen_expr = gen_func()
```

### Beginner Examples

```python
squares = (x * x for x in range(10))
print(type(squares))  # <class 'generator'>

# Compare memory
import sys
list_squares = [x * x for x in range(1000)]
gen_squares = (x * x for x in range(1000))
print(sys.getsizeof(list_squares))  # ~8856 bytes
print(sys.getsizeof(gen_squares))   # ~112 bytes (constant)

# Use with sum()
total = sum(x * x for x in range(1000000))  # No list created
```

### Intermediate Examples

```python
data = [
    {"name": "Alice", "score": 85},
    {"name": "Bob", "score": 92},
    {"name": "Charlie", "score": 78}
]

# Lazy filtering and transformation
high_scores = (
    f"{p['name']}: {p['score']}"
    for p in data
    if p['score'] >= 80
)

print(", ".join(high_scores))  # Alice: 85, Bob: 92

# Nested generator expressions
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = (item for row in matrix for item in row)
print(list(flat))  # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### Advanced Examples

```python
import itertools

# Generator expression with itertools for efficient combinatorics
cards = list(itertools.product("♠♥♦♣", ["A","2","3","4","5","6","7","8","9","10","J","Q","K"]))
deck = (f"{rank}{suit}" for suit, rank in cards)

# Lazy prime generation with generator expression
def is_prime(n):
    return n > 1 and all(n % i != 0 for i in range(2, int(n**0.5) + 1))

first_100_primes = (n for n in range(2, 1000) if is_prime(n))
print(list(itertools.islice(first_100_primes, 10)))  # First 10 primes
```

### Real-World Use Cases

- Pass a lazy generator to `sum()`, `any()`, `all()`, `min()`, `max()`.
- Read and process CSV/JSON lines with generator expressions in data pipelines.
- Lazy data validation: `all(validate(x) for x in huge_dataset)`.
- SQL-like querying: `(transform(row) for row in rows if filter(row))`.

### Common Mistakes

- Using square brackets instead of parentheses (creates a list, not a generator).
- Iterating over a generator expression multiple times (it gets exhausted).
- Expecting a generator expression to have methods like `.append()` or `.sort()`.
- Using generator expressions with side effects in debugging (they are lazy — won't execute until consumed).

### Best Practices

- Use generator expressions for simple, single-use iteration logic.
- Use generator functions for complex logic or multi-step transformations.
- Pass generator expressions directly to consuming functions to avoid intermediate lists.
- For long-running generator expressions, consider breaking into named generator functions for readability.

### Performance Considerations

Generator expressions have O(1) memory usage regardless of input size, making them ideal for large datasets. They are slightly faster than equivalent `for` loops with `yield` due to reduced bytecode, but slower than list comprehensions for small-to-medium datasets where you need all values.

### Interview Questions

**Q: What is the difference between a list comprehension and a generator expression?**

A: A list comprehension `[x for x in ...]` creates the entire list in memory. A generator expression `(x for x in ...)` creates a generator that yields items lazily. Generator expressions use less memory but have per-item access overhead.

**Q: When would you use a generator expression over a list comprehension?**

A: When working with large datasets; when passing to functions that consume iterables (`sum`, `any`, `all`); when you only need to iterate once; or when memory is constrained.

### Coding Challenges

1. Write a generator expression that yields all palindromic numbers under 10000.
2. Use a generator expression with `any()` to check if a large file contains a specific pattern.
3. Create a generator expression that yields the running average of a stream of numbers.

### Related Topics

- List comprehensions (eager counterpart)
- Set and dict comprehensions (other comprehension forms)
- itertools (often used alongside generator expressions)
- Generator functions (for more complex logic)

## send() Method

### What It Is

The `send()` method allows two-way communication with a generator. While `next()` pushes values from the generator to the caller, `send()` allows the caller to push values back into the generator, which becomes the value of the `yield` expression inside the generator body.

### Why It Is Important

`send()` transforms generators from simple iterators into coroutines, enabling bidirectional data flow. This pattern is useful for implementing state machines, data sinks, streaming protocols, and cooperative multitasking. It was the foundation of Python's original coroutine and async support before `async`/`await` syntax.

### How It Works Internally

When `gen.send(value)` is called, the value becomes the result of the `yield` expression that paused the generator. The generator resumes execution until the next `yield`, then returns the yielded value back to the caller. The first call must be `next(gen)` or `gen.send(None)` because there is no `yield` expression to receive a value before the generator starts.

### Syntax

```python
gen = generator_function()
next(gen)           # Prime the generator (or gen.send(None))
result = gen.send(value)  # Send value in, get next yielded value
```

### Beginner Examples

```python
def echo():
    """A generator that echoes back what it receives."""
    print("Starting echo generator")
    while True:
        received = yield
        print(f"Echo: {received}")

gen = echo()
next(gen)           # "Starting echo generator"
gen.send("Hello")   # "Echo: Hello"
gen.send("World")   # "Echo: World"
gen.close()
```

### Intermediate Examples

```python
def running_average():
    """Generator that computes running average."""
    total = 0.0
    count = 0
    average = None
    while True:
        value = yield average
        if value is None:
            continue
        total += value
        count += 1
        average = total / count

avg_gen = running_average()
next(avg_gen)  # Prime the generator
print(avg_gen.send(10))   # 10.0
print(avg_gen.send(20))   # 15.0
print(avg_gen.send(30))   # 20.0
```

### Advanced Examples

```python
def state_machine():
    """Finite state machine using send()."""
    state = "IDLE"
    while True:
        event = yield state
        if state == "IDLE":
            if event == "START":
                state = "RUNNING"
            elif event == "STOP":
                state = "STOPPED"
        elif state == "RUNNING":
            if event == "PAUSE":
                state = "PAUSED"
            elif event == "STOP":
                state = "STOPPED"
        elif state == "PAUSED":
            if event == "RESUME":
                state = "RUNNING"
            elif event == "STOP":
                state = "STOPPED"
        elif state == "STOPPED":
            pass  # Terminal state

fsm = state_machine()
next(fsm)               # Prime
print(fsm.send("START"))  # RUNNING
print(fsm.send("PAUSE"))  # PAUSED
print(fsm.send("RESUME")) # RUNNING
print(fsm.send("STOP"))   # STOPPED
```

### Real-World Use Cases

- **Data pipelines**: Send data into a processing pipeline stage and get results back.
- **State machines**: Implement protocol handlers, game state managers, or workflow engines.
- **Cooperative multitasking**: Coroutine scheduler that sends control between tasks.
- **Streaming parsers**: Feed data to a parser generator that yields parsed results.
- **Actor model**: Generators can act as lightweight actors that process messages sent via `send()`.

### Common Mistakes

- Forgetting to prime the generator with `next(gen)` or `gen.send(None)` first.
- Trying to send a value to an exhausted generator (raises `StopIteration`).
- Not handling `GeneratorExit` when the generator is closed externally.
- Expecting `send()` to work like `next()` without priming.

### Best Practices

- Always prime generators that use `send()` with `next(gen)` or `gen.send(None)`.
- Use `try/finally` in generators to clean up resources on close.
- Document whether a generator expects values via `send()`.
- Consider using `async`/`await` for new coroutine-based code instead of `send()`.

### Performance Considerations

`send()` has similar performance to `next()` since both resume the generator frame. The overhead comes from the function call and frame resumption, not from the data transfer itself. For high-frequency send operations, consider batching values.

### Interview Questions

**Q: What happens if you call `send()` without priming the generator?**

A: It raises `TypeError: can't send non-None value to a just-started generator`. The first call to a generator must be `next(gen)` or `gen.send(None)`.

**Q: How is `send()` related to coroutines?**

A: `send()` allows two-way communication with generators, effectively making them coroutines. Before `async`/`await` (PEP 492), generators with `send()` were the primary way to implement coroutines in Python.

### Coding Challenges

1. Implement a `task_queue` generator that accepts tasks via `send()` and processes them in order.
2. Create a generator-based logger that accepts log messages and writes them to a file, with configurable log level filtering.
3. Build a simple text adventure game engine using a generator state machine with `send()`.

### Related Topics

- Coroutines (generator-based coroutines vs. native coroutines)
- `throw()` method (inject exceptions into generators)
- `close()` method (clean up generator resources)
- `yield from` (delegating send/throw/close to sub-generators)

## yield from

### What It Is

`yield from` is a keyword introduced in Python 3.3 that allows a generator to delegate part of its operation to another generator or iterable. It yields all values from a sub-iterator, and importantly, it establishes a full bidirectional channel between the caller and the sub-generator, including `send()`, `throw()`, and `close()`.

### Why It Is Important

`yield from` simplifies generator composition by eliminating boilerplate code. Without it, you would need to manually iterate and yield each value from sub-generators, and bidirectional communication (through `send()`) would require complex forwarding logic. It is essential for refactoring generators into reusable components.

### How It Works Internally

When a generator does `yield from subgen`, Python suspends the current generator and forwards all iteration operations to `subgen`. The caller interacts directly with `subgen` through the delegation channel. When `subgen` is exhausted (raises `StopIteration`), the value from the exception becomes the value of the `yield from` expression, and the current generator resumes.

### Syntax

```python
def delegating_generator():
    yield from sub_generator  # Delegate to sub_generator
    # Or:
    yield from iterable       # Delegate to any iterable
```

### Beginner Examples

```python
def generator_a():
    yield 1
    yield 2

def generator_b():
    yield 3
    yield from generator_a()
    yield 4

print(list(generator_b()))  # [3, 1, 2, 4]

# Equivalent without yield from
def generator_b_manual():
    yield 3
    for val in generator_a():
        yield val
    yield 4
```

### Intermediate Examples

```python
def flatten(nested):
    """Flatten arbitrarily nested iterables."""
    for item in nested:
        if isinstance(item, (list, tuple)):
            yield from flatten(item)
        else:
            yield item

nested = [1, [2, [3, 4], 5], 6]
print(list(flatten(nested)))  # [1, 2, 3, 4, 5, 6]

# Recursive directory traversal
import os
def walk_files(path):
    for entry in os.scandir(path):
        if entry.is_dir():
            yield from walk_files(entry.path)
        else:
            yield entry.path

for file in walk_files("."):
    print(file)
```

### Advanced Examples

```python
def final_value_demo():
    """yield from captures the return value of the sub-generator."""
    def sub_gen():
        total = 0
        for i in range(5):
            received = yield i
            if received is not None:
                total += received
        return total

    result = yield from sub_gen()
    print(f"Sub-generator returned: {result}")
    yield result

gen = final_value_demo()
print(next(gen))          # 0
print(gen.send(10))       # 1 (received=10)
print(gen.send(20))       # 2 (received=20)
print(gen.send(30))       # 3 (received=30)
print(gen.send(40))       # 4 (received=40)
# "Sub-generator returned: 100"
# print(next(gen)) -> 100 (the final yielded value)
```

### Real-World Use Cases

- **Recursive generators** for tree data structures (AST walking, JSON traversal).
- **Generator refactoring** — extracting reusable generator parts while maintaining full coroutine channels.
- **Pipeline composition** — chaining processing stages where `send()`/`throw()` need to propagate.
- **Streaming parsers** where sub-generators handle specific protocol layers.
- **Async generators** (`yield from` was the precursor to `await` in async contexts).

### Common Mistakes

- Forgetting that `yield from` creates a direct channel — the caller sees the sub-generator directly.
- Expecting `yield from` without a return value to work as a simple for-loop (it does, but communication flows through).
- Using `yield from` with non-iterables (raises `TypeError`).
- Not understanding that `yield from` delegates `send()`, `throw()`, and `close()` to the sub-generator.

### Best Practices

- Use `yield from` instead of manual `for` loops when delegating to sub-generators.
- Prefer `yield from` for recursive generator patterns.
- Combine `yield from` with `return value` to pass final results from sub-generators.
- Document the sub-generator chain clearly for maintainability.

### Performance Considerations

`yield from` is slightly faster than manually iterating and yielding from a sub-generator because it optimizes the delegation at the C level. For deep chains of `yield from`, there is a cumulative overhead, but it is negligible for typical use cases.

### Interview Questions

**Q: What is the relationship between `yield from` and `await`?**

A: `await` in Python 3.5+ is syntactically and semantically similar to `yield from`. In fact, `await` was built on the same forwarding mechanics as `yield from`. In CPython, `await` uses the `YIELD_FROM` opcode, just like `yield from`.

**Q: How does `yield from` handle `send()` and `throw()`?**

A: `yield from` establishes a full bidirectional channel. When the caller calls `send(value)` on the delegating generator, the value is forwarded directly to the sub-generator's `send()`. Similarly, `throw()` is forwarded. The delegating generator only resumes when the sub-generator is exhausted.

### Coding Challenges

1. Write a `yield from`-based tree serializer that converts a nested dict structure to JSON-like output lazily.
2. Implement a `merge(*iterables)` generator that merges sorted iterables using `yield from` and `heapq.merge`.
3. Create a generator-based parser that uses `yield from` to delegate to sub-parsers for different data types.
4. Build a `zip_longest` generator using `yield from` and `itertools.zip_longest`.

### Related Topics

- Coroutines (the foundation for async/await)
- `asyncio` (uses `yield from` pattern with `await`)
- Generator delegation patterns
- `itertools.chain` (eager alternative to `yield from` for chaining iterables)
- Recursive algorithms (DFS, tree traversal with `yield from`)
