# Lambda Functions - lambda args: expression, map, filter, reduce

## Introduction

Lambda functions are small, anonymous functions defined with the `lambda` keyword. Unlike regular functions defined with `def`, lambdas are single-expression functions that cannot contain statements or annotations. They are used for short, throwaway functions where a full function definition would be unnecessarily verbose. Lambdas are a cornerstone of functional programming in Python, enabling elegant inline function definitions.

## lambda Syntax

### What It Is

A lambda function is an anonymous inline function defined by the syntax `lambda arguments: expression`. The expression is evaluated and returned when the lambda is called. Lambdas can take any number of arguments (including zero) but can only contain a single expression, not statements.

### Why It Is Important

Lambdas allow defining simple functions at the point of use without naming them. This is particularly valuable when passing short callbacks or transformations to higher-order functions like `map()`, `filter()`, and `sorted()`. They make functional-style code more concise and readable by keeping the logic inline.

### How It Works Internally

When Python compiles a lambda expression, it creates a function object — the same kind of object created by `def`. The lambda's bytecode includes a `RETURN_VALUE` opcode for the expression. The main difference is that lambda functions have the name `<lambda>` and cannot contain statements (assignments, loops, `return`, `yield`, etc.). The closure mechanism works identically to regular nested functions.

### Syntax

```python
lambda arguments: expression
# Equivalent to:
def anonymous_function(arguments):
    return expression
```

### Beginner Examples

```python
# Basic lambda
double = lambda x: x * 2
print(double(5))  # 10

# Multiple arguments
add = lambda a, b: a + b
print(add(3, 7))  # 10

# No arguments
constant = lambda: 42
print(constant())  # 42

# Default arguments
power = lambda base, exp=2: base ** exp
print(power(3))     # 9
print(power(3, 3))  # 27
```

### Intermediate Examples

```python
# Lambdas with sorting
students = [
    {"name": "Alice", "grade": 85},
    {"name": "Bob", "grade": 72},
    {"name": "Charlie", "grade": 95}
]
by_grade = sorted(students, key=lambda s: s["grade"], reverse=True)
print(by_grade)

# Lambdas with max/min
words = ["apple", "banana", "cherry", "date"]
longest = max(words, key=lambda w: len(w))
print(longest)  # banana

# Lambdas as dictionary values
operations = {
    "add": lambda a, b: a + b,
    "subtract": lambda a, b: a - b,
    "multiply": lambda a, b: a * b,
    "divide": lambda a, b: a / b if b != 0 else float('inf')
}
print(operations["multiply"](6, 7))  # 42
```

### Advanced Examples

```python
# Lambda with conditional expression
safe_divide = lambda a, b: a / b if b != 0 else None
print(safe_divide(10, 2))   # 5.0
print(safe_divide(10, 0))   # None

# Lambda with tuple unpacking
pairs = [(1, 'one'), (2, 'two'), (3, 'three')]
sorted_by_word = sorted(pairs, key=lambda pair: pair[1])
print(sorted_by_word)  # [(1, 'one'), (3, 'three'), (2, 'two')]

# Lambda returning a lambda (currying)
curried_add = lambda a: lambda b: a + b
add_5 = curried_add(5)
print(add_5(3))  # 8
print(curried_add(10)(20))  # 30

# Lambda with list comprehension equivalent
factors = lambda n: [i for i in range(1, n + 1) if n % i == 0]
print(factors(12))  # [1, 2, 3, 4, 6, 12]
```

### Real-World Use Cases

- **Sorting with custom keys**: `sorted(list, key=lambda x: x.some_attribute)`.
- **Event callbacks**: Define simple event handlers in GUI frameworks.
- **Data transformations**: Short transformations in data processing pipelines.
- **API response formatting**: Transform API responses inline.
- **Temporary key functions**: One-time use key functions for grouping or ordering.

### Common Mistakes

- Trying to use statements inside lambdas (`if`, `for`, `print`, `return`).
- Using lambdas when a named function would be clearer (especially for complex logic).
- Forgetting that lambda captures variables by reference, not by value (late-binding closures).
- Making lambdas too complex (multi-line expressions or nested conditionals).
- Using lambdas just because they're "more functional" — readability matters more.

### Best Practices

- Use lambdas only for simple, obvious operations (single expression, no side effects).
- Prefer `operator` module functions (`operator.add`, `operator.itemgetter`) over lambdas where applicable.
- Assign lambdas to variables only when needed (usually a `def` is better).
- Keep lambdas short — if the logic doesn't fit in one line, use `def`.
- Use lambdas with `sorted()`, `max()`, `min()`, and `filter()` for inline transformations.

### Performance Considerations

Lambda functions have the same performance characteristics as regular functions — they create a function object at definition time. The overhead of calling a lambda is identical to calling a `def` function. However, using a lambda inside a loop (defined repeatedly) creates many function objects, which can be slower than defining a named function outside the loop.

### Interview Questions

**Q: What is the difference between a lambda and a regular function?**

A: Lambdas are anonymous, single-expression functions that cannot contain statements. Regular functions use `def`, can have multiple expressions and statements, have a name, support annotations, and can include docstrings.

**Q: Can a lambda contain a return statement?**

A: No. The expression after the colon is implicitly returned. Using `return` inside a lambda causes a `SyntaxError`.

**Q: What is the late-binding closure problem with lambdas in loops?**

A: When lambdas capture loop variables, they capture the variable reference (not the value at creation time). By the time the lambdas are called, all of them see the final value of the loop variable. This is commonly seen in list comprehensions of lambdas.

### Coding Challenges

1. Write a lambda that checks if a string is a palindrome.
2. Use a lambda with `sorted()` to sort a list of tuples by the second element, then the first.
3. Create a lambda that computes the nth Fibonacci number.
4. Implement a lambda-based currying function that takes a function and partial arguments.

### Related Topics

- First-class functions
- Higher-order functions (`map`, `filter`, `reduce`)
- `operator` module
- Closures (lambdas close over their enclosing scope)
- `functools.partial` (alternative to lambdas for partial application)

## map() Function

### What It Is

`map()` is a built-in function that applies a given function to every item of an iterable and returns an iterator yielding the results. It transforms each element of the input iterable(s) according to the provided function, enabling clean, functional-style data transformations without explicit loops.

### Why It Is Important

`map()` abstracts the iteration pattern of applying a transformation to each element, making code more declarative and often more readable. It pairs well with lambdas for simple transformations and avoids the need for temporary lists. It is also lazy — it produces results on-demand, making it memory-efficient for large datasets.

### How It Works Internally

`map()` creates a map object (an iterator) that stores the function and iterable(s). Each call to `__next__` fetches the next item(s) from the iterable(s), passes them to the function, and yields the result. When the shortest iterable is exhausted, iteration stops. The implementation is in C, making it faster than an equivalent Python `for` loop for most cases.

### Syntax

```python
map(func, iterable)              # Single iterable
map(func, iterable1, iterable2)  # Multiple iterables (func must accept that many args)
```

### Beginner Examples

```python
# Basic mapping
numbers = [1, 2, 3, 4, 5]
squared = map(lambda x: x ** 2, numbers)
print(list(squared))  # [1, 4, 9, 16, 25]

# With a named function
def celsius_to_fahrenheit(c):
    return (c * 9/5) + 32

temps_c = [0, 20, 30, 100]
temps_f = list(map(celsius_to_fahrenheit, temps_c))
print(temps_f)  # [32.0, 68.0, 86.0, 212.0]
```

### Intermediate Examples

```python
# Multiple iterables
a = [1, 2, 3, 4]
b = [10, 20, 30, 40]
summed = map(lambda x, y: x + y, a, b)
print(list(summed))  # [11, 22, 33, 44]

# Different lengths (stops at shortest)
c = [1, 2, 3]
d = [10, 20, 30, 40, 50]
result = map(lambda x, y: (x, y), c, d)
print(list(result))  # [(1, 10), (2, 20), (3, 30)]

# map with None function (identity transform)
nested = [[1, 2], [3, 4], [5, 6]]
flattened = sum(map(list, nested), [])
print(flattened)  # [1, 2, 3, 4, 5, 6]

# Converting types
str_nums = ["1", "2", "3", "4", "5"]
int_nums = list(map(int, str_nums))
print(int_nums)  # [1, 2, 3, 4, 5]
```

### Advanced Examples

```python
# map with method calls
words = ["hello", "world", "python"]
upper_words = list(map(str.upper, words))
print(upper_words)  # ['HELLO', 'WORLD', 'PYTHON']

# map with operator module
import operator
vectors = [(1, 2), (3, 4), (5, 6)]
dot_products = list(map(operator.mul, *zip(*vectors)))
print(dot_products)  # [5, 11, 17] (1*5, 2*6 ... wait, let me recalculate)
# Actually: zip(*vectors) = [(1,3,5), (2,4,6)], operator.mul gives element-wise
dot_products = list(map(lambda a, b: a * b, *zip(*vectors)))
print(dot_products)  # [1*3*5=15, 2*4*6=48] - this is product not dot

# Proper dot product
dot = sum(map(operator.mul, (1,2,3), (4,5,6)))
print(dot)  # 1*4 + 2*5 + 3*6 = 32

# map with itertools for infinite sequences
import itertools
count = itertools.count(1)
squared_inf = map(lambda x: x**2, count)
first_10 = list(itertools.islice(squared_inf, 10))
print(first_10)  # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

### Real-World Use Cases

- **Data cleaning**: `map(str.strip, data)` to clean whitespace from strings.
- **Type conversion**: `map(int, csv_row)` to parse numeric strings.
- **Bulk normalization**: `map(normalize, records)` for data preprocessing.
- **Formatting**: `map(format_row, rows)` to prepare data for export.
- **API responses**: `map(transform, api_results)` to reshape API data.

### Common Mistakes

- Forgetting to consume the map object with `list()` or iteration.
- Using `map()` with a lambda when a list comprehension would be clearer.
- Expecting `map()` with multiple iterables to pad shorter ones (it stops at the shortest).
- Using `map()` with side-effect functions (use a `for` loop instead).
- Thinking `map()` is always faster than comprehensions (often comparable).

### Best Practices

- Use `map()` when you already have a named function — it's cleaner than a comprehension.
- Use list comprehensions when the transformation is complex or involves filtering.
- Use `map()` with `operator` module functions instead of lambdas for common operations.
- Consume map results lazily when processing large datasets.
- Prefer `map()` over explicit `for` loops for simple transformations.

### Performance Considerations

`map()` is implemented in C and is generally faster than an equivalent Python `for` loop. However, list comprehensions are also highly optimized and often match or exceed `map()` performance, especially when combined with `if` filters. For very large datasets, `map()`'s lazy evaluation saves memory.

### Interview Questions

**Q: What is the difference between `map()` and a list comprehension?**

A: Both apply a transformation. `map()` is lazy and works best with named functions. List comprehensions support filtering (`if`) and are often more readable for complex expressions. Performance is similar in most cases.

**Q: What happens when `map()` is given `None` as the function?**

A: It acts as the identity function, combining iterables into tuples. `map(None, a, b)` is equivalent to `zip(a, b)`.

### Coding Challenges

1. Use `map()` to convert a list of hexadecimal strings to integers.
2. Use `map()` with multiple iterables to compute element-wise Euclidean distance.
3. Implement a function that uses `map()` to concatenate corresponding strings from two lists with a separator.

### Related Topics

- List comprehensions (alternative to map/filter)
- `filter()` function
- `functools.reduce` (for cumulative operations)
- Generator expressions (similar laziness)
- `itertools.starmap` (for unpacking arguments)

## filter() Function

### What It Is

`filter()` is a built-in function that constructs an iterator from elements of an iterable for which a function returns `True`. It selects items that satisfy a condition, providing a functional way to filter data without explicit loops or conditional statements.

### Why It Is Important

`filter()` makes selection logic declarative and composable. Combined with lambdas, it enables concise inline filtering that reads naturally. Like `map()`, it is lazy and memory-efficient, processing items one at a time without creating intermediate collections.

### How It Works Internally

`filter()` creates a filter object (an iterator). Each call to `__next__` retrieves the next item from the iterable that makes the predicate function return `True`. Items for which the predicate returns `False` are skipped. `StopIteration` is raised when no more matching items exist. If the predicate is `None`, it uses the identity function (removing falsy values).

### Syntax

```python
filter(predicate, iterable)
# Returns iterator yielding items where predicate(item) is True
```

### Beginner Examples

```python
# Basic filtering
numbers = range(1, 21)
even = filter(lambda x: x % 2 == 0, numbers)
print(list(even))  # [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]

# filter with None (remove falsy values)
mixed = [0, 1, "", "hello", [], [1, 2], None, True, False]
truthy = list(filter(None, mixed))
print(truthy)  # [1, 'hello', [1, 2], True]
```

### Intermediate Examples

```python
# Filtering objects
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

people = [
    Person("Alice", 25), Person("Bob", 17),
    Person("Charlie", 30), Person("Diana", 16)
]
adults = filter(lambda p: p.age >= 18, people)
adult_names = [p.name for p in adults]
print(adult_names)  # ['Alice', 'Charlie']

# String filtering
words = ["apple", "banana", "avocado", "cherry", "apricot"]
a_words = list(filter(lambda w: w.startswith("a"), words))
print(a_words)  # ['apple', 'avocado', 'apricot']

# Filtering with regex
import re
emails = ["alice@example.com", "bob@test", "charlie@domain.org", "invalid"]
valid_emails = list(filter(
    lambda e: re.match(r"^[\w.]+@[\w.]+\.\w+$", e) is not None,
    emails
))
print(valid_emails)  # ['alice@example.com', 'charlie@domain.org']
```

### Advanced Examples

```python
# Filter and map combination
numbers = range(1, 31)
result = list(map(
    lambda x: x ** 2,
    filter(lambda x: x % 3 == 0, numbers)
))
print(result)  # [9, 36, 81, 144, 225, 324, 441, 576, 729, 900]

# Sieve of Eratosthenes with filter
def sieve(n):
    numbers = list(range(2, n + 1))
    primes = []
    while numbers:
        prime = numbers[0]
        primes.append(prime)
        numbers = list(filter(lambda x, p=prime: x % p != 0, numbers))
    return primes

print(sieve(50))  # [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]

# Filtering with partial application
from functools import partial

def greater_than(threshold, value):
    return value > threshold

above_10 = partial(greater_than, 10)
data = [5, 12, 3, 18, 7, 20]
filtered = list(filter(above_10, data))
print(filtered)  # [12, 18, 20]
```

### Real-World Use Cases

- **Data validation**: Filter out invalid records from a dataset.
- **Search/filtering UI**: Apply user-selected filters to a list of items.
- **Log filtering**: Select log entries matching specific criteria.
- **Data cleaning**: Remove `None`, empty strings, or sentinel values.
- **Input sanitization**: Filter allowed characters from user input.

### Common Mistakes

- Using `filter()` when a comprehension with `if` is more readable.
- Expecting `filter()` to modify the original iterable (it doesn't).
- Forgetting that `filter()` returns an iterator, not a list.
- Using `filter()` on infinite iterables without limiting results.
- Passing a predicate that returns non-boolean values (truthy/falsy works but can be confusing).

### Best Practices

- Use `filter()` when the predicate is a named function or simple lambda.
- Use list comprehensions when filtering involves complex conditions or you also need transformation.
- Combine `filter()` with `map()` in a pipeline for read–filter–transform sequences.
- Use `itertools.compress()` for filtering based on a separate selector iterable.
- Consume filter results lazily for memory efficiency with large data.

### Performance Considerations

`filter()` is lazy and memory-efficient, processing one item at a time. It is faster than a `for` loop with `if` for simple predicates. For complex filtering logic, a list comprehension is often as fast and more readable. Benchmarks show `filter()` is 10-30% faster than the equivalent comprehension in CPython for simple integer operations.

### Interview Questions

**Q: What is the difference between `filter()` and a list comprehension with `if`?**

A: `filter()` is lazy and typically has less overhead for simple predicates. List comprehensions with `if` are more flexible (can combine filtering and transformation) and are often preferred for readability.

**Q: What happens when `filter(None, iterable)` is used?**

A: It removes all falsy values — `None`, `False`, `0`, `0.0`, `""`, `[]`, `{}`, `()`. All truthy values pass through.

### Coding Challenges

1. Use `filter()` to extract all prime numbers from a list of integers.
2. Filter a list of strings to keep only valid email addresses (combination of `filter` and regex).
3. Use `filter()` to remove duplicate consecutive elements from a list (keep first of each run).
4. Implement a function that uses `filter()` to find all leap years in a range.

### Related Topics

- `map()` function
- List comprehensions with `if`
- `itertools.filterfalse` (inverse of filter)
- `itertools.compress` (selector-based filtering)
- `any()` and `all()` (predicate testing on iterables)

## reduce() Function

### What It Is

`reduce()` is a function from the `functools` module (moved from built-ins in Python 3) that applies a function of two arguments cumulatively to the items of an iterable, reducing the iterable to a single value. It is the functional programming equivalent of folding or accumulating a sequence.

### Why It Is Important

`reduce()` expresses cumulative operations (sum, product, maximum, concatenation) in a declarative way. While simple operations like sum and product have dedicated functions, `reduce()` is the general tool for any binary reduction, enabling clean implementations of complex accumulations like finding the Nth Fibonacci number or computing GCD across a sequence.

### How It Works Internally

`reduce()` works by applying the function to the first two items, then to the result and the next item, and so on. If an initializer is provided, it's used as the first argument and the first item of the iterable as the second. If the iterable has only one item and no initializer, that item is returned directly without calling the function.

### Syntax

```python
from functools import reduce

reduce(function, iterable)              # No initializer
reduce(function, iterable, initializer) # With initializer
```

### Beginner Examples

```python
from functools import reduce

# Sum (same as sum())
numbers = [1, 2, 3, 4, 5]
total = reduce(lambda a, b: a + b, numbers)
print(total)  # 15

# Product
product = reduce(lambda a, b: a * b, range(1, 6))
print(product)  # 120 (5!)

# Max
values = [3, 7, 2, 9, 1, 5]
maximum = reduce(lambda a, b: a if a > b else b, values)
print(maximum)  # 9
```

### Intermediate Examples

```python
from functools import reduce

# Concatenate strings
words = ["Hello", " ", "World", "!"]
sentence = reduce(lambda a, b: a + b, words)
print(sentence)  # "Hello World!"

# Union of sets
sets = [{1, 2}, {2, 3}, {3, 4}, {4, 5}]
union = reduce(lambda a, b: a | b, sets)
print(union)  # {1, 2, 3, 4, 5}

# Intersection of sets
sets = [{1, 2, 3}, {2, 3, 4}, {3, 4, 5}]
intersection = reduce(lambda a, b: a & b, sets)
print(intersection)  # {3}

# GCD of multiple numbers
import math
numbers = [48, 36, 72, 24]
gcd = reduce(math.gcd, numbers)
print(gcd)  # 12
```

### Advanced Examples

```python
from functools import reduce

# Flatten a list of lists
nested = [[1, 2], [3, 4, 5], [6]]
flat = reduce(lambda a, b: a + b, nested)
print(flat)  # [1, 2, 3, 4, 5, 6]

# Factorial with reduce
factorial = lambda n: reduce(lambda a, b: a * b, range(1, n + 1), 1)
print(factorial(5))  # 120

# Running maximum with reduce
from functools import reduce
data = [3, 1, 4, 1, 5, 9, 2, 6]
running_max = reduce(
    lambda acc, x: acc + [max(acc[-1], x)],
    data[1:],
    [data[0]]
)
print(running_max)  # [3, 3, 4, 4, 5, 9, 9, 9]

# Pipeline composition
pipeline = [
    lambda s: s.strip(),
    lambda s: s.upper(),
    lambda s: s.replace(" ", "_")
]
compose = lambda funcs, arg: reduce(lambda acc, f: f(acc), funcs, arg)
result = compose(pipeline, "  Hello World  ")
print(result)  # "HELLO_WORLD"
```

### Real-World Use Cases

- **Summing/producing numbers** across a sequence.
- **Computing statistical aggregates** (mean, variance) in one pass.
- **Chaining operations** in a pipeline (function composition).
- **Parsing expressions** (reduce can build syntax trees from tokens).
- **Merging dictionaries or configurations** with custom merge logic.
- **Converting between data structures** (list of dicts to nested dict).

### Common Mistakes

- Using `reduce()` when a loop is more readable (especially for non-trivial operations).
- Forgetting the initializer when the iterable might be empty (raises `TypeError`).
- Using `reduce()` for operations that have built-in functions (`sum()`, `any()`, `all()`).
- Applying `reduce()` to operations that are not associative (order matters, can produce wrong results).
- Expecting `reduce()` to be faster than a loop (it's usually the same or slower).

### Best Practices

- Use built-in reduction functions (`sum`, `min`, `max`, `any`, `all`) when available.
- Always provide an initializer for clarity and to handle empty iterables.
- Use `reduce()` only when the operation is clearly a reduction and no built-in exists.
- Consider using `functools.reduce` with `operator` module functions for common operations.
- Document what the function does — reduce can be cryptic.

### Performance Considerations

`reduce()` is implemented in C and has similar performance to an explicit `for` loop. The main cost is the function call per element. For tight loops, an explicit `for` loop with local variable bindings can be faster. The initializer adds no overhead when the iterable has at least one element.

### Interview Questions

**Q: Why was `reduce()` moved from built-ins to `functools`?**

A: Guido van Rossum, Python's creator, felt that `reduce()` was overused and harder to understand than explicit loops. It was moved to `functools` in Python 3 to discourage its use for simple operations while keeping it available for genuinely useful reductions.

**Q: What is the difference between `reduce()` and `accumulate()` from itertools?**

A: `reduce()` returns a single accumulated value. `itertools.accumulate()` returns an iterator yielding each intermediate accumulated value.

### Coding Challenges

1. Use `reduce()` to compute the mean and standard deviation of a list of numbers in one pass.
2. Implement a function `compose(*funcs)` that uses `reduce()` to compose multiple functions right-to-left.
3. Use `reduce()` to convert a list of digits to an integer: `[1, 2, 3]` -> `123`.
4. Implement `group_by` using `reduce()` on a list of key-value pairs.

### Related Topics

- `map()` and `filter()` (the trio of functional programming)
- `itertools.accumulate` (intermediate reduction values)
- `operator` module (provides functions for reduce)
- `functools` module
- List comprehensions (alternative to map/filter/reduce chains)
