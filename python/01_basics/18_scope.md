# Variable Scope - LEGB rule, global, nonlocal

## Introduction

Variable scope determines where a name can be accessed within a program. Python resolves names using the LEGB rule: Local, Enclosing, Global, Built-in. When you reference a name, Python searches these four scopes in order and uses the first match. Unlike many languages, Python does not have block-level scope — loops, conditionals, and `with` blocks do NOT create a new scope. Only functions, classes, modules, and comprehensions (in Python 3) create new scopes. Understanding scope is essential for closures, decorators, and avoiding subtle bugs.

## LEGB rule

### What It Is

The LEGB rule defines the order Python searches for variable names: **L**ocal (inside the current function), **E**nclosing (in outer/nested functions), **G**lobal (at the module level), **B**uilt-in (Python's predefined names like `len`, `print`, `range`). Search stops at the first match.

### Why It Is Important

LEGB determines which variable you're actually accessing when multiple variables share the same name. Misunderstanding LEGB leads to `UnboundLocalError`, unintended variable shadowing, and bugs in nested functions and closures.

### How It Works Internally

When Python compiles a function, it scans the body and classifies each name as local, global, or built-in using the `co_varnames`, `co_freevars`, `co_cellvars`, and `co_names` tuples on the code object. At runtime, name lookup follows:
1. `LOAD_FAST` — local variables (from `co_varnames`, stored in the frame's `f_locals`)
2. `LOAD_DEREF` — free/cell variables (closures, stored in the frame's `f_localsplus`)
3. `LOAD_GLOBAL` — global and built-in names (from module dict and builtins)
If a name is assigned anywhere in a function, Python considers it local for the entire function (unless declared `global` or `nonlocal`).

### Syntax

```python
# LEGB search order
built_in = len  # Built-in scope
global_var = "global"  # Global scope

def outer():
    enclosing_var = "enclosing"  # Enclosing scope
    
    def inner():
        local_var = "local"  # Local scope
        print(local_var)     # Local
        print(enclosing_var) # Enclosing
        print(global_var)    # Global
        print(len)           # Built-in
    
    inner()
```

### Beginner Examples

```python
# Built-in scope
print(len)  # <built-in function len>

# Global scope
x = 10

def show_x():
    print(x)  # Accesses global x

show_x()  # 10

# Local scope shadows global
x = 10
def local_x():
    x = 20  # New local variable, does NOT modify global
    print(f"Inside: {x}")

local_x()   # Inside: 20
print(x)    # 10 (global unchanged)

# Enclosing scope
def outer():
    msg = "Hello from outer"
    
    def inner():
        print(msg)  # Accesses enclosing (non-local)
    
    inner()

outer()

# Enclosing + local
def outer():
    val = "outer"
    def inner():
        val = "inner"  # New local, shadows enclosing
        print(val)
    inner()
    print(val)  # "outer" (unchanged)

outer()
```

### Intermediate Examples

```python
# LEGB in action with multiple levels
z = "global"

def level1():
    z = "level1"
    
    def level2():
        z = "level2"
        
        def level3():
            z = "level3"
            print(z)  # level3 (local)
        
        level3()
        print(z)  # level2 (enclosing)
    
    level2()
    print(z)  # level1 (enclosing)

level1()
print(z)  # global (global)

# Variable classification determines behavior
x = 10

def test():
    print(x)  # Would it work or error?
    # x = 20  # If this line exists, x is local, print(x) raises UnboundLocalError

test()  # 10 (x is global, no local assignment)

# But:
def test_error():
    # print(x)  # UnboundLocalError!
    x = 20  # Python sees assignment and classifies x as local

# test_error()  # Would fail if uncommented

# Comprehension scope (Python 3)
x = "outer"
comp = [x for x in range(3)]
print(x)  # "outer" (comprehension variable is localized in Python 3)

# Generator expression scope
x = "outer"
gen = (x for x in range(3))
print(x)  # "outer"
```

### Advanced Examples

```python
# Code object inspection
def demo():
    x = 10
    y = 20
    def inner():
        z = 30
        return x + y + z
    return inner

print(demo.__code__.co_varnames)    # ('x', 'y', 'inner')
print(demo.__code__.co_cellvars)    # ('x', 'y')
inner_func = demo()
print(inner_func.__code__.co_freevars)  # ('x', 'y')
print(inner_func.__code__.co_varnames)  # ('z',)

# Dynamic scope inspection
import sys

def inspect_scope():
    frame = sys._getframe(0)
    print(f"Local: {list(frame.f_locals.keys())}")
    print(f"Global: {list(frame.f_globals.keys())[:5]}...")

inspect_scope()

# Simulating scope resolution manually
class ScopeResolver:
    def __init__(self):
        self._locals = {}
        self._globals = globals()
        self._builtins = __builtins__
    
    def resolve(self, name):
        if name in self._locals:
            return f"Local: {self._locals[name]}"
        if name in self._globals:
            return f"Global: {self._globals[name]}"
        if hasattr(self._builtins, name):
            return "Built-in"
        return "Not found"

r = ScopeResolver()
print(r.resolve("len"))    # Built-in
print(r.resolve("__name__"))  # Global
```

### Real-World Use Cases

- **Debugging**: Understanding why a variable has an unexpected value
- **Closures**: Capturing enclosing scope for function factories
- **Decorators**: Wrapping functions while preserving scope relationships
- **Configuration**: Module-level globals for shared settings
- **Libraries**: Ensuring internal names don't leak to user scope
- **Recursive patterns**: Managing state across nested function calls

### Common Mistakes

```python
# Mistake 1: UnboundLocalError
x = 10
def increment():
    # x += 1  # UnboundLocalError
    # Python sees assignment (x += 1) and classifies x as local
    # Fix:
    global x
    x += 1

# Mistake 2: Assuming block-level scope
for i in range(5):
    pass
print(i)  # 4 (i persists after loop! Python has no block scope)

if True:
    y = 42
print(y)  # 42 (no block scope for conditionals)

# Mistake 3: Closure capturing loop variable
funcs = []
for i in range(3):
    funcs.append(lambda: i)  # All reference the SAME i

for f in funcs:
    print(f())  # 2, 2, 2 (all see final value of i)

# Fix: capture by default argument
funcs = []
for i in range(3):
    funcs.append(lambda x=i: x)  # Captures current i

for f in funcs:
    print(f())  # 0, 1, 2

# Mistake 4: Assuming comprehension leaks (Python 3)
[x for x in range(5)]
print(x)  # NameError in Python 3! (x is localized)
```

### Best Practices

- Avoid using the same name in multiple scopes (shadowing)
- Use `global` and `nonlocal` sparingly — prefer passing parameters
- Be aware of loop variable leakage (no block scope)
- Use default arguments to capture loop variables in closures
- Keep functions small with limited local scope
- Use `_` prefix for internal/private variables (convention)
- Use class attributes for shared state when appropriate

### Performance Considerations

Local variable access (`LOAD_FAST`) is faster than global (`LOAD_GLOBAL`) and built-in (`LOAD_GLOBAL` with dict lookup). Local variables are stored in a fixed-size array in the frame, indexed by integer offset. Global lookups require a dict lookup. For performance-critical inner loops, copy globals to local variables:

```python
# Slow: global lookup each iteration
import math
def slow(values):
    return [math.sqrt(x) for x in values]

# Fast: local reference
def fast(values):
    sqrt = math.sqrt
    return [sqrt(x) for x in values]
```

### Interview Questions

1. What is the LEGB rule and what does each letter stand for?
2. What causes UnboundLocalError and how do you fix it?
3. Does Python have block-level scope? Explain.
4. How do comprehension variables behave differently in Python 2 vs 3?
5. How does Python determine if a name is local or global?
6. Why do all closures in a loop capture the final value?
7. How can you inspect code objects to see their scope classification?
8. What is the scope of a class definition?

### Coding Challenges

```python
# Challenge 1: Fix the counter
def make_counter():
    count = 0
    def counter():
        nonlocal count
        count += 1
        return count
    return counter

c = make_counter()
print(c())  # 1
print(c())  # 2

# Challenge 2: Multi-level counter
def make_multi_counter():
    counts = {}
    def counter(name):
        nonlocal counts
        counts[name] = counts.get(name, 0) + 1
        return counts[name]
    return counter

mc = make_multi_counter()
print(mc("a"))  # 1
print(mc("b"))  # 1
print(mc("a"))  # 2

# Challenge 3: Variable scope analyzer
import inspect

def analyze_var(name):
    frame = inspect.currentframe().f_back
    for level in range(10):
        if frame is None:
            break
        if name in frame.f_locals:
            return f"Local (depth {level}): {frame.f_locals[name]}"
        if name in frame.f_globals:
            return f"Global: {frame.f_globals[name]}"
        frame = frame.f_back
    return "Not found"

x = 42
def test():
    y = 10
    print(analyze_var("y"))  # Local
    print(analyze_var("x"))  # Global
```

### Related Topics

- global and nonlocal keywords
- Closures
- Decorators
- Namespace and module scope
- Frame objects and inspection
- Local vs global performance

## global keyword

### What It Is

The `global` statement declares that a name inside a function refers to a module-level (global) variable. Without `global`, any assignment to a name in a function creates a local variable — even if a global with that name exists.

### Why It Is Important

`global` is necessary because Python's name-binding rules otherwise make all assigned names local. Without `global`, modifying a module-level variable from inside a function is impossible. It also allows reading a global before assigning to a local with the same name without causing UnboundLocalError.

### How It Works Internally

The `global` declaration causes the compiler to use `LOAD_GLOBAL` and `STORE_GLOBAL` bytecodes instead of `LOAD_FAST`/`STORE_FAST` for the declared names. The name is stored in the function's `co_names` tuple and accessed via the module's `__dict__`. The declaration must appear before any use of the name in the function body.

### Syntax

```python
# Declaration
global name
global name1, name2  # Multiple names

# Usage
counter = 0

def increment():
    global counter  # Declare intent to use module-level counter
    counter += 1
```

### Beginner Examples

```python
# Modifying a global variable
count = 0

def increment():
    global count
    count += 1

increment()
increment()
print(count)  # 2

# Reading global without declaration (no assignment)
name = "Alice"

def greet():
    print(f"Hello, {name}!")  # Can read global without declaration

greet()  # Hello, Alice!

# Shadowing: local assignment without global
value = 100

def set_local():
    value = 200  # Creates local, doesn't touch global

set_local()
print(value)  # 100 (unchanged)

# Global with same name as parameter (don't do this)
x = 10
def bad(param):
    global x  # Shadows parameter? No -- global and param are different
    x = param  # Modifies global
    return x

print(bad(20))  # 20
print(x)  # 20
```

### Intermediate Examples

```python
# Multiple globals
app_name = "MyApp"
version = "1.0"
debug = True

def configure():
    global app_name, version, debug
    app_name = "ProductionApp"
    version = "2.0"
    debug = False

configure()
print(app_name, version, debug)

# Global in nested functions
result = []

def add_item(item):
    global result
    result.append(item)

def clear():
    global result
    result = []  # Reassigns global (not modifying the list)

add_item("apple")
add_item("banana")
print(result)  # ['apple', 'banana']
clear()
print(result)  # []

# Global for initialization pattern
_cache = None

def get_cache():
    global _cache
    if _cache is None:
        _cache = {}
    return _cache

cache = get_cache()
cache["key"] = "value"
print(get_cache())  # {'key': 'value'}
```

### Advanced Examples

```python
# Global across modules (cross-module state)
# config.py
DEBUG = False
DATABASE_URL = "sqlite:///default.db"

# app.py (conceptual)
# from config import DEBUG, DATABASE_URL
# 
# def setup():
#     global DEBUG, DATABASE_URL
#     DEBUG = True
#     DATABASE_URL = "postgresql://localhost/mydb"

# The above works because DEBUG and DATABASE_URL are imported names
# but they're local to app.py's module scope
# To modify the actual config module:
import config

def setup():
    config.DEBUG = True
    config.DATABASE_URL = "postgresql://localhost/mydb"

setup()
print(config.DEBUG)  # True

# Global variables and threads
import threading

counter = 0
lock = threading.Lock()

def thread_safe_increment():
    global counter
    with lock:
        counter += 1

threads = [threading.Thread(target=thread_safe_increment) for _ in range(100)]
for t in threads: t.start()
for t in threads: t.join()
print(counter)  # 100

# Global inspection
x = 10
def get_globals():
    global x
    return globals()  # Returns module __dict__

print(get_globals()["x"])  # 10
```

### Real-World Use Cases

- **Module-level configuration**: App-wide settings modified by setup functions
- **Caching**: Module-level cache that gets populated on first call
- **Counters**: Global counters for analytics (use with caution)
- **Logging**: Configuring loggers at module level
- **Registration patterns**: Plugin/component registration systems
- **State flags**: Global flags (debug mode, test mode)

### Common Mistakes

```python
# Mistake 1: Forgetting global when assigning
count = 0
def increment():
    # count += 1  # UnboundLocalError!
    global count
    count += 1

# Mistake 2: Declaring global after use
x = 10
def bad():
    print(x)  # UnboundLocalError if global declared below
    global x  # Too late! Python already classified x as local
    x = 20

# Mistake 3: Using global when a parameter is better
# Instead of:
result = 0
def add_bad(a, b):
    global result
    result = a + b

# Better:
def add_good(a, b):
    return a + b

# Mistake 4: Global with mutable objects
items = []
def add_item(item):
    global items
    items.append(item)  # Works, but global is not needed here!
    # items.append(item)  # Also works without global when only mutating

# Mistake 5: Global in library code
# Library functions should accept parameters, not rely on globals
```

### Best Practices

- Use `global` sparingly — prefer parameters and return values
- Never use `global` in library functions (leads to coupling)
- Use module-level constants (ALL_CAPS) without `global`
- For shared state, prefer classes or closures over globals
- If you must use global, keep it in a single module
- Document all global variables clearly
- Consider using `threading.Lock` with mutable globals
- Use `globals()` dict for dynamic global access

### Performance Considerations

Global variable access is slower than local access (dict lookup vs array index). Repeated global access in loops should be cached locally. `STORE_GLOBAL` is slower than `STORE_FAST`. For performance-critical code, copy globals to locals and use parameters.

### Interview Questions

1. When is the `global` keyword necessary?
2. What happens if you assign to a global variable without `global`?
3. Can you use `global` in nested functions?
4. What is the difference between modifying a mutable global and reassigning it?
5. Why is global considered bad practice?
6. How does `global` affect bytecode generation?

### Coding Challenges

```python
# Challenge 1: Global counter with reset
counter = 0

def increment():
    global counter
    counter += 1

def reset():
    global counter
    counter = 0

def get_count():
    global counter
    return counter

increment()
increment()
print(get_count())  # 2
reset()
print(get_count())  # 0

# Challenge 2: Global configuration manager
_config = {
    "host": "localhost",
    "port": 8080,
    "debug": False,
}

def get_config(key, default=None):
    return _config.get(key, default)

def set_config(key, value):
    global _config
    _config[key] = value

def update_config(**kwargs):
    global _config
    _config.update(kwargs)

set_config("port", 9090)
update_config(debug=True)
print(get_config("port"))    # 9090
print(get_config("debug"))   # True

# Challenge 3: Module-level singleton
_initialized = False

def init_app():
    global _initialized
    if _initialized:
        return
    _initialized = True
    print("Initializing...")

init_app()  # Initializing...
init_app()  # (no output, already initialized)
```

### Related Topics

- nonlocal keyword
- Module scope
- globals() built-in
- Constants convention (ALL_CAPS)
- Thread safety with globals

## nonlocal keyword

### What It Is

The `nonlocal` statement declares that a name in a nested function refers to a variable in the nearest enclosing function scope (excluding global scope). It allows assignment to variables in enclosing scopes without making them global. Unlike `global`, `nonlocal` cannot refer to the module-level scope.

### Why It Is Important

`nonlocal` is essential for closures that maintain state across calls — counters, accumulators, caches, and decorators. Without `nonlocal`, assigning to an enclosing variable would create a new local variable, breaking the closure pattern.

### How It Works Internally

When `nonlocal` is declared, the compiler emits `LOAD_DEREF` and `STORE_DEREF` bytecodes instead of `LOAD_FAST`/`STORE_FAST`. The variable is stored as a cell object in the function's `__closure__` tuple. The enclosing function stores the variable as a cell (co_cellvars), and the nested function references it as a free variable (co_freevars). Changes propagate to all closures sharing the same cell.

### Syntax

```python
def outer():
    x = 10
    
    def inner():
        nonlocal x  # x refers to enclosing x
        x += 5
    
    inner()
    print(x)  # 15
    
# Multiple variables
def outer():
    a = 1
    b = 2
    def inner():
        nonlocal a, b
        a += 1
        b += 1
    inner()
```

### Beginner Examples

```python
# Counter using nonlocal
def make_counter():
    count = 0
    
    def counter():
        nonlocal count
        count += 1
        return count
    
    return counter

c = make_counter()
print(c())  # 1
print(c())  # 2
print(c())  # 3

# Each closure has its own state
c1 = make_counter()
c2 = make_counter()
print(c1())  # 1
print(c1())  # 2
print(c2())  # 1 (independent)

# Without nonlocal, this fails:
def bad_make_counter():
    count = 0
    def counter():
        # count += 1  # UnboundLocalError!
        return count  # Can read, can't assign
    return counter

# Accumulator pattern
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
```

### Intermediate Examples

```python
# Nonlocal with multiple levels
def level1():
    x = 1
    
    def level2():
        x = 2  # Shadows level1's x
        
        def level3():
            nonlocal x  # Refers to level2's x, NOT level1's
            x += 1
            return x
        
        print(level3())  # 3
        print(x)  # 3
        
    level2()
    print(x)  # 1 (level1's x unchanged)

level1()

# Nonlocal with list state
def make_tracker():
    history = []
    
    def track(value):
        nonlocal history
        history.append(value)
        return history
    
    def clear():
        nonlocal history
        history = []  # Reassign, not just mutate
    
    return track, clear

track, clear = make_tracker()
track("a")
track("b")
print(track("c"))  # ['a', 'b', 'c']
clear()
print(track("x"))  # ['x']

# Limited call decorator
def max_calls(n):
    def decorator(func):
        calls = 0
        
        def wrapper(*args, **kwargs):
            nonlocal calls
            calls += 1
            if calls > n:
                raise RuntimeError(f"Max {n} calls exceeded")
            return func(*args, **kwargs)
        
        return wrapper
    return decorator

@max_calls(3)
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))   # Hello, Alice!
print(greet("Bob"))     # Hello, Bob!
print(greet("Charlie")) # Hello, Charlie!
# print(greet("Diana"))  # RuntimeError!
```

### Advanced Examples

```python
# Multiple closures sharing state
def make_shared_state():
    data = {}
    
    def setter(key, value):
        nonlocal data
        data[key] = value
    
    def getter(key):
        return data.get(key)
    
    def deleter(key):
        nonlocal data
        if key in data:
            del data[key]
    
    def clear():
        nonlocal data
        data = {}
    
    return setter, getter, deleter, clear

setter, getter, deleter, clear = make_shared_state()
setter("name", "Alice")
print(getter("name"))  # Alice
deleter("name")
print(getter("name"))  # None

# nonlocal with initialization guard
def lazy_init():
    _cache = None
    
    def get_or_compute(key, factory):
        nonlocal _cache
        if _cache is None:
            _cache = {}
        if key not in _cache:
            _cache[key] = factory()
        return _cache[key]
    
    def stats():
        nonlocal _cache
        return {"initialized": _cache is not None}
    
    return get_or_compute, stats

compute, stats = lazy_init()
print(stats())  # {'initialized': False}
result = compute("expensive", lambda: sum(range(10**6)))
print(stats())  # {'initialized': True}

# nonlocal with class emulation
def make_account(initial_balance=0):
    balance = initial_balance
    transactions = []
    
    def deposit(amount):
        nonlocal balance
        if amount <= 0:
            raise ValueError("Amount must be positive")
        balance += amount
        transactions.append(("deposit", amount))
        return balance
    
    def withdraw(amount):
        nonlocal balance
        if amount <= 0:
            raise ValueError("Amount must be positive")
        if amount > balance:
            raise ValueError("Insufficient funds")
        balance -= amount
        transactions.append(("withdraw", amount))
        return balance
    
    def get_balance():
        return balance
    
    def get_transactions():
        return list(transactions)
    
    return deposit, withdraw, get_balance, get_transactions

dep, wd, bal, txns = make_account(100)
dep(50)    # 150
wd(30)     # 120
print(bal())  # 120
print(txns())  # [('deposit', 50), ('withdraw', 30)]
```

### Real-World Use Cases

- **Closures with state**: Counters, accumulators, caches
- **Decorators**: `@wraps`, timing, logging, rate limiting
- **Lazy initialization**: Module-level singletons with state
- **State encapsulation**: Emulating private state (before dataclasses)
- **Function factories**: Creating functions with persistent configuration
- **Callback handlers**: Callbacks that maintain state across invocations

### Common Mistakes

```python
# Mistake 1: nonlocal at global scope
# nonlocal x  # SyntaxError: no enclosing scope

# Mistake 2: nonlocal with non-existent variable
def outer():
    def inner():
        # nonlocal x  # SyntaxError: no binding for nonlocal 'x' found
        x = 10
    inner()

# Mistake 3: Forgetting nonlocal when assigning
def make_counter():
    count = 0
    def counter():
        # count += 1  # UnboundLocalError!
        nonlocal count
        count += 1
        return count
    return counter

# Mistake 4: nonlocal naming conflict
x = 10
def outer():
    x = 20  # Enclosing
    def inner():
        nonlocal x  # Refers to outer's x, NOT global
        x = 30
    inner()
    print(x)  # 30

# Mistake 5: Using nonlocal when global is needed
x = 10
def outer():
    def inner():
        # nonlocal x  # No enclosing x!
        global x  # Must use global
        x = 20
    inner()

outer()
print(x)  # 20

# Mistake 6: Multiple levels of nesting with same name
def a():
    x = 1
    def b():
        x = 2
        def c():
            nonlocal x  # Refers to b's x
            x = 3
        c()
        print(x)  # 3
    b()
a()
```

### Best Practices

- Use `nonlocal` only when you need to reassign an enclosing variable
- For mutable state, you can mutate without `nonlocal` (e.g., `list.append`)
- Use `nonlocal` sparingly — consider classes for complex state
- Prefer passing state via parameters when possible
- Use `nonlocal` for simple counting/accumulation in closures
- Document what the nonlocal variable represents

### Performance Considerations

`nonlocal` variable access uses `LOAD_DEREF` (cell variable), which is slightly slower than `LOAD_FAST` (local) but faster than `LOAD_GLOBAL` (dict lookup). Cell variables involve indirection through a cell object. In performance-critical inner loops, consider pulling nonlocal values into locals at the top of the function.

### Interview Questions

1. What is the difference between `global` and `nonlocal`?
2. Can you use `nonlocal` at the module level?
3. How does `nonlocal` interact with closures?
4. What happens if the enclosing scope variable doesn't exist?
5. How do you share state between multiple closures?
6. What bytecodes are generated for `nonlocal`?
7. Can `nonlocal` reference a global variable? Why or why not?

### Coding Challenges

```python
# Challenge 1: State machine
def make_state_machine():
    state = "idle"
    
    def transition(event):
        nonlocal state
        transitions = {
            ("idle", "start"): "running",
            ("running", "pause"): "paused",
            ("paused", "resume"): "running",
            ("running", "stop"): "idle",
            ("paused", "stop"): "idle",
        }
        next_state = transitions.get((state, event))
        if next_state:
            state = next_state
            return f"{state}"
        return f"Invalid transition from {state} via {event}"
    
    def current_state():
        return state
    
    return transition, current_state

trans, cur = make_state_machine()
print(trans("start"))  # running
print(trans("pause"))  # paused
print(cur())           # paused

# Challenge 2: Ratelimited function
def ratelimit(max_per_second):
    last_calls = []
    
    def decorator(func):
        import time
        def wrapper(*args, **kwargs):
            nonlocal last_calls
            now = time.time()
            last_calls = [t for t in last_calls if now - t < 1]
            if len(last_calls) >= max_per_second:
                raise RuntimeError("Rate limit exceeded")
            last_calls.append(now)
            return func(*args, **kwargs)
        return wrapper
    return decorator

# Challenge 3: Memoized function
def memoize():
    cache = {}
    
    def decorator(func):
        def wrapper(*args):
            nonlocal cache
            key = args
            if key not in cache:
                cache[key] = func(*args)
            return cache[key]
        return wrapper
    return decorator

memo = memoize()

@memo
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)

print(fib(100))  # 354224848179261915075
```

### Related Topics

- global keyword
- Closures
- Cell variables (__closure__)
- Decorators
- Variable scope (LEGB)
- Function attributes (func.attr pattern)
