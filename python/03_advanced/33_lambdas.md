# Lambda Functions - lambda args: expression, map, filter, reduce

## Introduction

A lambda is a small, anonymous function defined inline using the `lambda` keyword. Unlike regular functions defined with `def`, lambdas are limited to a single expression and cannot contain statements or annotations. They are typically used for short, throwaway operations where defining a full function would be unnecessarily verbose, especially as arguments to higher-order functions like `map()`, `filter()`, `sorted()`, and `reduce()`.

## Why It Is Important

Lambdas enable functional programming patterns in Python by allowing concise function definitions without cluttering the namespace. They are essential for callback-based APIs, sorting with custom keys, data transformation pipelines, and GUI event handlers. Their compact syntax makes code more readable when the logic is simple and localized, though they should be avoided for complex operations.

## Syntax

```python
lambda arguments: expression

# Examples:
lambda x: x * 2
lambda a, b: a + b
lambda x, y: x if x > y else y
lambda *args: sum(args)
lambda **kwargs: max(kwargs.values())
```

## Examples

```python
from typing import Any, Callable, Sequence, Iterable
from functools import reduce
import math
```

### Basic Lambda

```python
double = lambda x: x * 2
print(double(5))  # 10

# Equivalent to:
def double_func(x: int) -> int:
    return x * 2
```

### Lambda with Multiple Arguments

```python
add = lambda a, b: a + b
print(add(3, 7))  # 10

max_val = lambda a, b: a if a > b else b
print(max_val(10, 20))  # 20
```

### Lambda with Default Arguments

```python
greet = lambda name, greeting="Hello": f"{greeting}, {name}!"
print(greet("Alice"))
print(greet("Bob", "Hi"))
```

### Immediately Invoked Lambda

```python
result = (lambda x: x ** 2)(9)
print(result)  # 81
```

## Beginner Examples

### Using map() with Lambda

```python
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x ** 2, numbers))
print(squared)  # [1, 4, 9, 16, 25]

celsius = [0, 10, 20, 30, 40]
fahrenheit = list(map(lambda c: c * 9 / 5 + 32, celsius))
print(fahrenheit)
```

### Using filter() with Lambda

```python
numbers = range(1, 21)
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]

words = ["cat", "elephant", "dog", "giraffe", "ant"]
long_words = list(filter(lambda w: len(w) > 3, words))
print(long_words)  # ['elephant', 'giraffe']
```

### Sorting with key=lambda

```python
students = [
    {"name": "Alice", "grade": 85},
    {"name": "Bob", "grade": 72},
    {"name": "Charlie", "grade": 93},
    {"name": "Diana", "grade": 88},
]

by_grade = sorted(students, key=lambda s: s["grade"], reverse=True)
print(by_grade)

by_name = sorted(students, key=lambda s: s["name"])
print(by_name)
```

### Using reduce() with Lambda

```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]
product = reduce(lambda a, b: a * b, numbers)
print(product)  # 120

max_value = reduce(lambda a, b: a if a > b else b, numbers)
print(max_value)  # 5
```

## Intermediate Examples

### Sorting Complex Objects

```python
class Person:
    def __init__(self, name: str, age: int, salary: float) -> None:
        self.name = name
        self.age = age
        self.salary = salary

    def __repr__(self) -> str:
        return f"Person({self.name}, {self.age}, {self.salary})"

people = [
    Person("Alice", 30, 70000),
    Person("Bob", 25, 85000),
    Person("Charlie", 35, 65000),
    Person("Diana", 28, 90000),
]

by_salary = sorted(people, key=lambda p: p.salary)
print(by_salary)

by_age_desc = sorted(people, key=lambda p: p.age, reverse=True)
print(by_age_desc)

# Multi-level sort: by age, then by name
multi = sorted(people, key=lambda p: (p.age, p.name))
print(multi)
```

### Lambda in Dictionary for Switch/Case

```python
operations = {
    "+": lambda a, b: a + b,
    "-": lambda a, b: a - b,
    "*": lambda a, b: a * b,
    "/": lambda a, b: a / b if b != 0 else float('inf'),
    "^": lambda a, b: a ** b,
}

def calculate(op: str, x: float, y: float) -> float:
    func = operations.get(op)
    if func is None:
        raise ValueError(f"Unknown operation: {op}")
    return func(x, y)

print(calculate("+", 10, 5))
print(calculate("^", 2, 3))
```

### Lambda for Data Transformation Pipelines

```python
data = ["  hello  ", "  WORLD  ", "  Python  "]

pipeline = lambda items: list(
    map(lambda s: s.strip().upper(),
        filter(lambda s: len(s.strip()) > 0, items))
)

print(pipeline(data))

# More granular:
cleaned = list(map(lambda x: x.strip().capitalize(), data))
print(cleaned)
```

### Lambda with max() / min()

```python
strings = ["apple", "banana", "cherry", "date"]
longest = max(strings, key=lambda s: len(s))
print(longest)  # banana

# Finding the dictionary with max value
sales = [
    {"product": "A", "revenue": 1200},
    {"product": "B", "revenue": 3400},
    {"product": "C", "revenue": 2100},
]
top_seller = max(sales, key=lambda s: s["revenue"])
print(top_seller)
```

### Using Lambdas in List.sort() with Multiple Criteria

```python
records = [
    ("John", 25, "Engineer"),
    ("Alice", 30, "Designer"),
    ("Bob", 25, "Engineer"),
    ("Diana", 28, "Manager"),
]

# Sort by age ascending, then by name alphabetically
records.sort(key=lambda r: (r[1], r[0]))
print(records)
```

## Advanced Examples

### Lambda in Decorators

```python
def log_result(func: Callable) -> Callable:
    return lambda *args, **kwargs: (
        print(f"{func.__name__} -> {result}") 
        for result in [func(*args, **kwargs)]
    ).__next__()

@log_result
def add(a: int, b: int) -> int:
    return a + b

add(3, 4)
```

### Lambda in Functional Composition

```python
def compose(*functions: Callable) -> Callable:
    """Compose multiple functions: compose(f, g)(x) = f(g(x))"""
    return reduce(lambda f, g: lambda x: f(g(x)), functions)

def add_one(x: int) -> int:
    return x + 1

def double(x: int) -> int:
    return x * 2

def square(x: int) -> int:
    return x ** 2

pipeline = compose(square, double, add_one)
print(pipeline(5))  # square(double(add_one(5))) = square(double(6)) = square(12) = 144
```

### Lambda for Lazy Evaluation

```python
class Lazy:
    def __init__(self, func: Callable) -> None:
        self._func = func
        self._value: Any = None
        self._evaluated = False

    def get(self) -> Any:
        if not self._evaluated:
            self._value = self._func()
            self._evaluated = True
        return self._value

lazy_value = Lazy(lambda: sum(range(10_000_000)))
print("Lazy value created (not computed yet)")
print(f"Computed: {lazy_value.get()}")
```

### Lambda in Event Handlers (GUI Simulation)

```python
class Button:
    def __init__(self, label: str) -> None:
        self.label = label
        self._handlers: list[Callable] = []

    def on_click(self, handler: Callable) -> None:
        self._handlers.append(handler)

    def click(self) -> None:
        for handler in self._handlers:
            handler()

buttons = [
    Button("Save"),
    Button("Delete"),
    Button("Cancel"),
]

for btn in buttons:
    btn.on_click(lambda b=btn: print(f"{b.label} clicked"))

buttons[0].click()
buttons[2].click()
```

### Lambda with Partial Application (Currying)

```python
def curry(func: Callable) -> Callable:
    def curried(*args: Any, **kwargs: Any) -> Any:
        if len(args) + len(kwargs) >= func.__code__.co_argcount:
            return func(*args, **kwargs)
        return lambda *more_args, **more_kwargs: curried(
            *args, *more_args, **{**kwargs, **more_kwargs}
        )
    return curried

@curry
def greet(greeting: str, name: str) -> str:
    return f"{greeting}, {name}!"

say_hello = greet("Hello")
print(say_hello("Alice"))
print(say_hello("Bob"))

say_hi = greet("Hi")
print(say_hi("Charlie"))
```

### Using Lambdas in DefaultDict Factories

```python
from collections import defaultdict

# Create nested defaultdict with lambda
nested_dict = defaultdict(lambda: defaultdict(list))
nested_dict["users"]["alice"].append(100)
nested_dict["users"]["bob"].append(200)
print(dict(nested_dict))

# Counter with lambda default
counter = defaultdict(lambda: 0)
for ch in "abracadabra":
    counter[ch] += 1
print(dict(counter))
```

### Lambda in Properties

```python
class Rectangle:
    def __init__(self, width: float, height: float) -> None:
        self.width = width
        self.height = height

    # Using lambda-like getter
    area = property(lambda self: self.width * self.height)
    perimeter = property(lambda self: 2 * (self.width + self.height))

rect = Rectangle(10, 5)
print(rect.area)
print(rect.perimeter)
```

## Real-World Use Cases

- **Sorting data** with custom keys in `sorted()`, `list.sort()`, `max()`, `min()`.
- **Data transformation** with `map()` and `filter()`.
- **GUI callback functions** (tkinter, PyQt) where simple actions are needed.
- **Key functions in itertools** (e.g., `itertools.groupby()`).
- **Default factory functions** in `defaultdict`.
- **Pandas apply()** with simple row/column transformations.
- **Web framework route parameters** (e.g., Flask URL converters).
- **Configuration files** where callable defaults are needed.
- **Property definitions** for simple computed attributes.

## Common Mistakes

- Trying to use statements inside lambdas (assignments, `return`, `for`, `if/else` blocks).
- Using lambdas where a regular function would be more readable (complex logic > 1 expression).
- Forgetting that variables in lambdas are late-bound — they capture the variable, not its value.
- Overusing lambdas when `operator` module functions would suffice (e.g., `operator.add`).
- Using lambdas for functions that are reused multiple times (define a `def` instead).
- Expecting lambdas to support type annotations or docstrings.

### Late Binding Pitfall

```python
# This creates 10 lambdas that all reference the same 'i'
funcs = [lambda: i for i in range(10)]
print([f() for f in funcs])  # [9, 9, 9, ...]

# Fix: use default argument to capture current value
funcs = [lambda i=i: i for i in range(10)]
print([f() for f in funcs])  # [0, 1, 2, ...]
```

## Best Practices

- Keep lambdas simple — one expression only, preferably fitting on one line.
- Use lambdas only when the operation is trivial; define a named function for anything complex.
- Prefer `operator` module functions (`operator.add`, `operator.itemgetter`) over lambdas.
- Use default arguments in lambdas to avoid late-binding issues.
- Use type hints with a `Callable` type alias if the lambda is stored in a variable.
- Consider using `functools.partial` instead of lambda for partial function application.
- For sorting by attributes, use `operator.attrgetter`; for items, use `operator.itemgetter`.

## Interview Questions

1. What is a lambda function and how is it different from a regular function?
2. What are the limitations of lambda functions in Python?
3. How do you use lambda with `map()`, `filter()`, and `reduce()`?
4. Explain the late-binding problem with lambdas in loops and how to fix it.
5. How can you use lambda for custom sorting?
6. What's the difference between `lambda` and `functools.partial`?
7. Can a lambda function be recursive?
8. How can you use lambda with `defaultdict`?
9. What is a closure and how does it relate to lambdas?
10. How would you implement a simple switch/case using lambdas?

## Coding Challenges

1. **String Sort**: Sort a list of strings by their last character using a lambda.
2. **Filter by Length**: Use lambda with `filter()` to keep words longer than 5 characters.
3. **Lambda Calculator**: Implement a calculator using a dictionary of lambda functions.
4. **Key Extractor**: Given a list of tuples `(name, age, score)`, sort by score descending using lambda.
5. **Nested Sort**: Sort a list of dictionaries by a specified key using a lambda.
6. **Compose**: Write a `compose` function that chains multiple lambdas.
7. **FizzBuzz with Lambdas**: Implement FizzBuzz using `map()` and lambdas.
8. **Data Pipeline**: Build a transformation pipeline using multiple lambdas in sequence.

## Summary

Lambda functions are concise, anonymous functions limited to a single expression. They excel in functional programming patterns like `map()`, `filter()`, `sorted()`, and `reduce()`, where simple inline functions are needed. While powerful, they should be used judiciously — simple operations only — and regular functions should be preferred for any non-trivial logic.

## Related Topics

- Functional programming (map, filter, reduce)
- Higher-order functions
- Closures
- operator module
- functools module (partial, reduce)
- Sorting algorithms
- Defaultdict factories