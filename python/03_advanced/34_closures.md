# Closures - Nested functions, __closure__, nonlocal variables

## Introduction

A closure is a function object that remembers values in enclosing scopes even when those scopes are no longer present in memory. In Python, closures occur when a nested function references variables from its enclosing function's scope. This concept enables powerful patterns like data hiding, function factories, and decorators. Understanding closures is essential for mastering Python's functional programming capabilities and is a prerequisite for understanding decorators, partial functions, and callback-based patterns.

## Nested Functions

### What It Is

A nested function is a function defined inside another function. The inner function can access variables from the outer function's scope, but not vice versa. Nested functions are local to the outer function and cannot be called directly from outside unless returned or assigned to an external variable.

### Why It Is Important

Nested functions enable encapsulation — they hide helper functions from the global namespace, preventing name collisions and keeping implementation details private. Combined with closures, they allow the inner function to retain access to the outer function's state, enabling function factories and stateful callbacks.

### How It Works Internally

When Python executes a function definition, it compiles the function body into a code object. For nested functions, the bytecode includes a `LOAD_DEREF` instruction for accessing variables from the enclosing scope. The function object stores a reference to the enclosing scope's variables through the `__closure__` attribute. Each nested function gets its own scope that is linked to the enclosing scope via the `__closure__` tuple of cells.

### Syntax

```python
def outer_function(outer_param):
    # Outer scope
    def inner_function(inner_param):
        # Inner scope, can access outer_param and inner_param
        return outer_param + inner_param
    return inner_function
```

### Beginner Examples

```python
def make_greeting(greeting):
    def greet(name):
        return f"{greeting}, {name}!"
    return greet

say_hello = make_greeting("Hello")
say_hi = make_greeting("Hi")

print(say_hello("Alice"))   # "Hello, Alice!"
print(say_hi("Bob"))        # "Hi, Bob!"

# Inner function without closure (no outer variables used)
def outer():
    def inner():
        return "Just a nested function"
    return inner

func = outer()
print(func())  # "Just a nested function"
```

### Intermediate Examples

```python
# Inner function with multiple levels of nesting
def factory(x):
    def middle(y):
        def inner(z):
            return x * y + z
        return inner
    return middle

calc = factory(10)(5)
print(calc(3))  # 10 * 5 + 3 = 53

# Nested functions for encapsulation
def process_data(initial_data):
    data = initial_data[:]

    def add_item(item):
        data.append(item)

    def remove_item(item):
        data.remove(item)

    def get_data():
        return data[:]  # Return a copy

    return {"add": add_item, "remove": remove_item, "get": get_data}

manager = process_data([1, 2, 3])
manager["add"](4)
manager["remove"](1)
print(manager["get"]())  # [2, 3, 4]
```

### Advanced Examples

```python
# Nested functions as callback dispatchers
def create_event_handler(event_type):
    handlers = {}

    def register(callback):
        handlers[callback.__name__] = callback
        return callback

    def dispatch(*args, **kwargs):
        for name, handler in handlers.items():
            handler(*args, **kwargs)

    def get_handler(name):
        return handlers.get(name)

    return {"register": register, "dispatch": dispatch, "get": get_handler}

handler_system = create_event_handler("click")
handler_system["register"](lambda: print("Clicked!"))
handler_system["dispatch"]()  # Clicked!

# Nested generator with closure
def make_counter(start=0, step=1):
    count = start

    def counter():
        nonlocal count
        current = count
        count += step
        return current

    return counter

counter = make_counter(10, 5)
print(counter())  # 10
print(counter())  # 15
print(counter())  # 20
```

### Real-World Use Cases

- **Encapsulation**: Hide implementation details of helper functions.
- **Function factories**: Create specialized functions with shared configuration.
- **Callback registration systems**: Bundle related callbacks with shared state.
- **Partial application**: Bind some arguments of a function (alternative to `functools.partial`).
- **Lazy initialization**: Compute and cache a value on first access.

### Common Mistakes

- Defining a nested function but never returning or referencing it outside (it's just a local variable, not a closure unless referenced).
- Expecting the outer function to access inner function variables (impossible).
- Creating nested functions inside loops without capturing the loop variable by value (late-binding problem).
- Forgetting that nested functions are redefined each time the outer function is called.
- Making nested functions that are too deep (more than 2-3 levels hurts readability).

### Best Practices

- Use nested functions for simple helpers that are only relevant within the outer function.
- Keep nesting depth to at most 2-3 levels.
- Consider extracting deeply nested functions into separate top-level functions with clear prefixes.
- Use nested functions with closures for function factories (the classic use case).
- Document the expected closure behavior clearly for maintainers.

### Performance Considerations

Each call to the outer function that returns a nested function creates a new function object. This is fast but not free — avoid creating many closure functions in tight loops. Accessing closure variables (`LOAD_DEREF`) is slightly slower than accessing locals (`LOAD_FAST`) but faster than globals (`LOAD_GLOBAL`). The `__closure__` tuple keeps referenced variables alive, preventing garbage collection of the enclosing scope.

### Interview Questions

**Q: What is the difference between a nested function and a closure?**

A: A nested function is a function defined inside another function. A closure is a nested function that references variables from the enclosing scope. Not all nested functions are closures (only those that capture variables).

**Q: Are nested functions hoisted in Python?**

A: No. Unlike JavaScript, Python does not hoist nested functions. A nested function must be defined before it's used within the outer function, just like any other variable.

### Coding Challenges

1. Implement a `make_timer()` function that returns a closure returning the time elapsed since the factory was called.
2. Create a nested function pattern that caches expensive computations.
3. Implement a simple middleware chain using nested functions and closures.
4. Build a function factory that creates mathematical operation functions with a configurable precision.

### Related Topics

- Closures (closures build on nested functions)
- `nonlocal` keyword
- `__closure__` attribute
- Decorators (closures in action)
- `functools.partial` (alternative closure pattern)

## __closure__ Attribute

### What It Is

The `__closure__` attribute is a tuple of cell objects on a function object that captures variables from the enclosing scope. Each cell object has a `cell_contents` attribute that holds the current value of the captured variable. If a function does not capture any variables, `__closure__` is `None`.

### Why It Is Important

`__closure__` provides introspection into which variables a closure captures and their current values. This is crucial for debugging closure-related bugs (especially the late-binding problem), for implementing serialization of closure-based functions, and for understanding how closures work internally.

### How It Works Internally

When the outer function is called, Python creates cell objects for each variable referenced by the inner function. These cells live on the heap (not the stack), allowing the inner function to access them even after the outer function returns. The `__closure__` attribute stores these cells in the order the variables are accessed in the inner function's code.

### Syntax

```python
def outer(x):
    def inner(y):
        return x + y
    return inner

f = outer(10)
print(f.__closure__)       # (<cell at 0x...: int object at 0x...>,)
print(f.__closure__[0].cell_contents)  # 10
```

### Beginner Examples

```python
def multiplier(factor):
    def multiply(x):
        return x * factor
    return multiply

double = multiplier(2)
triple = multiplier(3)

print(double.__closure__)  # (<cell at 0x...: int object at 0x...>,)
print(double.__closure__[0].cell_contents)  # 2
print(triple.__closure__[0].cell_contents)  # 3

# Non-closure function
def regular_func(x):
    return x * 2

print(regular_func.__closure__)  # None
```

### Intermediate Examples

```python
def analyze_closure():
    a = 1
    b = "hello"
    c = [1, 2, 3]
    d = 42  # Not captured

    def inner():
        return a + b[0] + c[0]  # Captures a, b, c. Does NOT capture d.

    return inner

func = analyze_closure()
for i, cell in enumerate(func.__closure__):
    print(f"Cell {i}: {cell.cell_contents} (type: {type(cell.cell_contents).__name__})")
# Cell 0: 1 (type: int)
# Cell 1: hello (type: str)
# Cell 2: [1, 2, 3] (type: list)

# Modifying cell contents (via the cell, not recommended)
func.__closure__[0].cell_contents = 100
print(func())  # 101 + h + 1 (potentially broken)
```

### Advanced Examples

```python
import pickle

def make_closure(value):
    def inner():
        return value
    return inner

# Serializing closures is tricky
c1 = make_closure(42)

# We can manually serialize and deserialize if we know the structure
import types

def rebuild_closure(value):
    def inner():
        return value
    return inner

state = {
    "cell_value": c1.__closure__[0].cell_contents
}
deserialized = rebuild_closure(state["cell_value"])
print(deserialized())  # 42

# Introspection to get all captured variables
def inspect_closure(func):
    if func.__closure__ is None:
        return {}
    code = func.__code__
    # co_freevars contains the names of captured variables
    return dict(zip(
        code.co_freevars,
        [cell.cell_contents for cell in func.__closure__]
    ))

def outer(a, b):
    c = "captured"
    def inner():
        return a + b + c
    return inner

f = outer(1, 2)
print(inspect_closure(f))  # {'a': 1, 'b': 2, 'c': 'captured'}
```

### Real-World Use Cases

- **Debugging**: Check what values a closure is capturing (especially for late-binding issues).
- **Testing**: Verify that closures capture the expected variables.
- **Serialization**: When pickling closures, understanding `__closure__` helps implement custom serialization.
- **Dynamic code generation**: Analyse and modify captured variables in generated closures.
- **Profiling**: Track which closures keep large objects alive (memory debugging).

### Common Mistakes

- Modifying `cell_contents` directly (it works but can lead to inconsistent state — avoid in production).
- Expecting `__closure__` to be present on regular functions (it's `None` for non-closures).
- Forgetting that `__closure__` order corresponds to `co_freevars` order, not variable definition order.
- Relying on `__closure__` to check if a function is a closure (need to check it's not None and not empty).

### Best Practices

- Use `__closure__` mainly for debugging and introspection, not in production logic.
- Access `__closure__` through the function object after it's been returned from the outer function.
- Pair `__closure__` with `func.__code__.co_freevars` to get both names and values.
- Avoid modifying `cell_contents` in production code — it breaks the encapsulation that closures provide.
- Use proper debugging tools (breakpoints, logging) before resorting to `__closure__` inspection.

### Performance Considerations

Accessing `__closure__` is cheap (attribute lookup). Iterating over cells and accessing `cell_contents` is also fast. However, using it in hot paths is unnecessary since you can access the values through normal function calls. The closure mechanism itself adds minimal overhead (one level of indirection per captured variable per function call).

### Interview Questions

**Q: What is the difference between `__closure__` and `__code__.co_freevars`?**

A: `__closure__` contains the actual cell objects with values. `co_freevars` contains the names of the captured variables as a tuple of strings. They are parallel — `co_freevars[i]` is the name of the variable captured in `__closure__[i]`.

**Q: Can a closure have an empty `__closure__` tuple?**

A: Technically no — `__closure__` is `None` if no variables are captured, or a tuple of cells for captured variables. An empty tuple is not possible because if a variable is referenced, a cell is created.

### Coding Challenges

1. Write a function that takes a closure and returns a dictionary of all captured variable names and values.
2. Implement a `compare_closures(f1, f2)` function that checks if two closures capture the same variables with the same values.
3. Create a function that serializes a simple closure to JSON (captured values only) and a function to restore it.

### Related Topics

- `nonlocal` keyword
- `global` keyword
- `__code__.co_freevars`
- `__code__.co_cellvars`
- `inspect` module for closure introspection

## nonlocal Variables

### What It Is

The `nonlocal` keyword allows a nested function to assign to variables defined in the enclosing (but non-global) scope. Without `nonlocal`, assignment in a nested function creates a new local variable, shadowing the outer one. `nonlocal` is the closure equivalent of `global`, but for enclosing function scopes.

### Why It Is Important

`nonlocal` enables closures to maintain mutable state across calls. Without it, closures could only read outer variables — any assignment would create a new local variable, breaking the closure. This is essential for patterns like counters, accumulators, caches, and state machines implemented as closures.

### How It Works Internally

When `nonlocal var` is declared, the compiler marks `var` for lookup in enclosing scopes using cell objects. The `STORE_DEREF` bytecode instruction is used for assignments to `nonlocal` variables, which writes to the cell object shared with the enclosing scope. Without `nonlocal`, assignment uses `STORE_FAST` (local) or `STORE_GLOBAL`.

### Syntax

```python
def outer():
    count = 0
    def inner():
        nonlocal count  # Declare count as nonlocal
        count += 1      # Now modifies outer's count
        return count
    return inner
```

### Beginner Examples

```python
# Counter with nonlocal
def make_counter(start=0):
    count = start
    def counter():
        nonlocal count
        count += 1
        return count
    return counter

counter_a = make_counter()
counter_b = make_counter(100)

print(counter_a())  # 1
print(counter_a())  # 2
print(counter_b())  # 101
print(counter_a())  # 3

# Without nonlocal (broken)
def make_counter_broken(start=0):
    count = start
    def counter():
        # This creates a NEW local variable 'count'
        count += 1  # UnboundLocalError!
        return count
    return counter
```

### Intermediate Examples

```python
# Accumulator with nonlocal
def make_accumulator():
    total = 0
    def add(value):
        nonlocal total
        total += value
        return total
    return add

acc = make_accumulator()
print(acc(5))   # 5
print(acc(3))   # 8
print(acc(10))  # 18

# Cache with nonlocal
def make_cached(func):
    cache = {}
    def wrapper(*args):
        nonlocal cache  # Not actually needed if we mutate dict
        key = args
        if key not in cache:
            cache[key] = func(*args)
        return cache[key]
    return wrapper

# Multiple nonlocal variables
def make_stats():
    count = 0
    total = 0.0
    def update(value):
        nonlocal count, total
        count += 1
        total += value
    def average():
        nonlocal count, total
        return total / count if count else 0
    return update, average

update, avg = make_stats()
update(10)
update(20)
update(30)
print(avg())  # 20.0
```

### Advanced Examples

```python
# State machine with nonlocal
def make_state_machine():
    state = "idle"
    data = None

    def transition(event, value=None):
        nonlocal state, data
        if state == "idle":
            if event == "start":
                state = "running"
                data = value
                return f"Started with {value}"
        elif state == "running":
            if event == "process":
                data = data + value if isinstance(data, int) else value
                return f"Processed, data={data}"
            elif event == "stop":
                state = "stopped"
                result = data
                data = None
                return f"Stopped with final={result}"
        elif state == "stopped":
            return "Machine is stopped"
        return f"Invalid transition from {state}"

    def get_state():
        nonlocal state
        return state

    return transition, get_state

trans, state = make_state_machine()
print(trans("start", 10))  # Started with 10
print(trans("process", 5)) # Processed, data=15
print(trans("stop"))       # Stopped with final=15
print(state())             # stopped

# Nonlocal for caching and lazy initialization
def make_lazy_property(compute_func):
    value = None
    computed = False

    def getter():
        nonlocal value, computed
        if not computed:
            value = compute_func()
            computed = True
        return value

    def reset():
        nonlocal computed
        computed = False

    return getter, reset

import random
expensive = lambda: random.randint(1, 1000000)
get_val, reset_val = make_lazy_property(expensive)

print(get_val())  # Computes and caches
print(get_val())  # Returns cached value
reset_val()
print(get_val())  # Recomputes
```

### Real-World Use Cases

- **Counters**: Create independent counters for tracking events.
- **Accumulators**: Aggregate values across multiple calls (analytics, logging).
- **Caches**: Implement memoization and lazy initialization in closures.
- **State machines**: Maintain state across transitions in a closure-based FSM.
- **Throttling/Debouncing**: Track timestamps across calls for rate limiting.
- **Configuration accumulation**: Build configuration objects incrementally.

### Common Mistakes

- Forgetting `nonlocal` and getting `UnboundLocalError` when trying to assign.
- Using `nonlocal` for mutable objects (like lists) when only mutation is needed (no reassignment needed).
- Thinking `nonlocal` searches the global scope (use `global` for that).
- Declaring `nonlocal` for a variable that doesn't exist in any enclosing scope (`SyntaxError`).
- Using `nonlocal` at module level (only valid inside nested functions).

### Best Practices

- Use `nonlocal` only when you need to reassign a captured variable (rebind to a new object).
- For mutable objects (lists, dicts, sets), mutation doesn't require `nonlocal` — only reassignment does.
- Keep the number of nonlocal variables small — too many indicate the closure is doing too much.
- Consider converting complex closures with many nonlocal variables to a class with methods.
- Document which variables are being modified via `nonlocal` for clarity.

### Performance Considerations

`nonlocal` variable access uses `LOAD_DEREF`/`STORE_DEREF` instructions, which are slightly slower than local variable access but faster than globals. The overhead is negligible for most use cases. Each `nonlocal` variable creates a cell object that persists as long as the closure exists, which can prevent memory from being freed.

### Interview Questions

**Q: What is the difference between `nonlocal` and `global`?**

A: `global` refers to the module-level scope. `nonlocal` refers to the nearest enclosing function scope (not global). `global` works at any nesting level, while `nonlocal` only works inside nested functions and skips the global scope.

**Q: What happens if you try to assign to an outer variable without `nonlocal`?**

A: Python creates a new local variable with the same name in the inner function's scope, shadowing the outer variable. If you try to read before assignment, you get `UnboundLocalError` because Python detected the assignment and treats it as local throughout.

### Coding Challenges

1. Implement a `make_timer` function using `nonlocal` that returns the time since it was last called.
2. Create a `make_ratelimiter(max_calls, period)` using `nonlocal` to track call timestamps.
3. Implement a once-callable wrapper using `nonlocal` that ensures a function is only called once.
4. Build a closure-based debouncer that delays function execution until calls stop for N milliseconds.

### Related Topics

- `global` keyword
- Closures (nonlocal is for mutable closure state)
- Function attributes (alternative: attach state to the function object itself)
- Class-based state (class with `__call__` as an alternative to closures with nonlocal)
- `__closure__` attribute (how nonlocal variables are stored)
