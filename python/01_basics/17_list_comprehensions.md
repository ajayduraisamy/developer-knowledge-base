# List Comprehensions - [expr for item in iterable if condition]

## Introduction

List comprehensions provide a concise, readable syntax for creating lists by applying an expression to each element of an iterable, optionally filtering elements with a condition. Python extends this pattern to set comprehensions (`{...}`), dict comprehensions (`{k: v ...}`), and generator expressions (`(...)`). Comprehensions are more Pythonic and generally faster than equivalent for-loop constructions, but should be used judiciously — complex comprehensions harm readability and should be refactored into regular loops or helper functions.

## Basic comprehensions

### What It Is

A basic list comprehension `[expr for item in iterable]` applies `expr` to each element of an iterable and collects the results into a new list. The expression can be any valid Python expression, including function calls, attribute access, arithmetic, or even nested comprehensions.

### Why It Is Important

Comprehensions replace verbose for-loop + `list.append()` patterns with a single, declarative line. They signal intent clearly: "transform this collection into a new collection." They are also consistently faster than manual loops due to reduced attribute lookups and list-append overhead.

### How It Works Internally

The Python compiler recognizes list comprehension syntax and generates a special code object (a "comprehension" inner scope) with `LIST_APPEND` bytecodes. In CPython, the comprehension creates a new list, iterates over the input using `GET_ITER`/`FOR_ITER`, evaluates the expression, and appends via `LIST_APPEND`. The loop variable in Python 3 is local to the comprehension scope (unlike Python 2 where it leaked).

### Syntax

```python
[expression for item in iterable]
[expression for item in iterable if condition]
[expression if condition else alt for item in iterable]
```

### Beginner Examples

```python
# Square numbers
squares = [x**2 for x in range(10)]
print(squares)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# Convert temperatures
celsius = [0, 10, 20, 30, 40]
fahrenheit = [c * 9/5 + 32 for c in celsius]
print(fahrenheit)  # [32.0, 50.0, 68.0, 86.0, 104.0]

# String operations
fruits = ["apple", "banana", "cherry"]
upper = [f.upper() for f in fruits]
print(upper)  # ['APPLE', 'BANANA', 'CHERRY']

# Extract attribute
people = [{"name": "Alice"}, {"name": "Bob"}, {"name": "Charlie"}]
names = [p["name"] for p in people]
print(names)  # ['Alice', 'Bob', 'Charlie']

# Lengths
words = ["hello", "world", "python"]
lengths = [len(w) for w in words]
print(lengths)  # [5, 5, 6]
```

### Intermediate Examples

```python
# Function call in expression
def double(x):
    return x * 2

nums = [1, 2, 3, 4, 5]
doubled = [double(n) for n in nums]
print(doubled)  # [2, 4, 6, 8, 10]

# Method call
strings = ["hello", "world"]
reversed_strs = [s[::-1] for s in strings]
print(reversed_strs)  # ['olleh', 'dlrow']

# Multiple operations
data = [3, 1, 4, 1, 5, 9, 2, 6]
processed = [n * 2 if n % 2 == 0 else n * 3 for n in data]
print(processed)

# Cartesian product (flat)
colors = ["red", "green"]
sizes = ["S", "M", "L"]
products = [(c, s) for c in colors for s in sizes]
print(products)  # [('red','S'), ('red','M'), ...]

# Unpacking tuples
pairs = [(1, 2), (3, 4), (5, 6)]
sums = [a + b for a, b in pairs]
print(sums)  # [3, 7, 11]
```

### Advanced Examples

```python
# Using walrus operator (Python 3.8+)
import math
def process(x):
    return x ** 2

values = [1, 2, 3, 4, 5]
result = [y for x in values if (y := process(x)) > 10]
print(result)  # [16, 25]

# Side-effect free transform
def safe_int(x):
    try:
        return int(x)
    except (ValueError, TypeError):
        return None

data = ["42", "abc", "3.14", "100", None]
parsed = [safe_int(x) for x in data]
print(parsed)  # [42, None, None, 100, None]

# Comprehension as mapping + filter
items = [1, -2, 3, -4, 5, -6]
positive_doubled = [x * 2 for x in items if x > 0]
print(positive_doubled)  # [2, 6, 10]

# Flat map (map + flatten)
def expand(n):
    return range(n)

expanded = [x for n in [2, 3, 4] for x in expand(n)]
print(expanded)  # [0, 1, 0, 1, 2, 0, 1, 2, 3]
```

### Real-World Use Cases

- **Data transformation**: Converting data formats, units, encodings
- **Field extraction**: Pulling specific keys from lists of dicts
- **Data cleaning**: Normalizing values, trimming whitespace
- **API responses**: Extracting fields from JSON arrays
- **ETL pipelines**: Column transformations in data processing
- **Validation**: Checking all elements satisfy a condition

### Common Mistakes

```python
# Mistake 1: Using comprehension for side effects
[print(x) for x in range(5)]  # Creates [None, None, None, None, None]
# Use for loop instead:
for x in range(5):
    print(x)

# Mistake 2: Forgetting comprehension creates a new object
original = [1, 2, 3]
squared = [x**2 for x in original]  # New list, original unchanged

# Mistake 3: Variable leakage (Python 2 only)
# Python 3: comprehension variable is local
x = "outer"
[x for x in range(5)]
print(x)  # "outer" (Python 3), 4 (Python 2)

# Mistake 4: Confusing order in nested loops
# [expr for a in A for b in B]  -- a outer, b inner
matrix = [[1, 2], [3, 4]]
flat = [x for row in matrix for x in row]  # [1, 2, 3, 4]
```

### Best Practices

- Keep comprehensions simple enough to fit on 1-2 lines
- Use nested comprehensions sparingly (max 2 levels)
- Prefer comprehensions over `map()` + `filter()` for readability
- Never use comprehensions for side effects
- Extract complex expressions into named functions
- Use `[x for x in items if x]` for filtering truthy values
- Use `[x for x in items if x is not None]` for filtering None

### Performance Considerations

Comprehensions are ~20-50% faster than manual for-loops with `append()` because:
- The `LIST_APPEND` bytecode is faster than calling `list.append()`
- No attribute lookup for `.append` each iteration
- The comprehension runs in its own scope (faster local variable access)
For very large iterables, consider generator expressions to avoid building the entire list in memory.

### Interview Questions

1. What is the syntax for a list comprehension with a condition?
2. How is a list comprehension different from a generator expression?
3. Does a list comprehension leak the loop variable in Python 3?
4. How do you flatten a list of lists with a comprehension?
5. Can you use if-else in a list comprehension?
6. What is the performance benefit of comprehensions?
7. How do you write a comprehension that produces a Cartesian product?
8. When should you NOT use a list comprehension?

### Coding Challenges

```python
# Challenge 1: Flatten nested list
nested = [[1, 2], [3, 4, 5], [6], [7, 8, 9]]
flat = [item for sublist in nested for item in sublist]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Challenge 2: Prime numbers
def is_prime(n):
    if n <= 1:
        return False
    return all(n % i != 0 for i in range(2, int(n**0.5) + 1))

primes = [n for n in range(2, 50) if is_prime(n)]
print(primes)  # [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]

# Challenge 3: FizzBuzz with comprehension
fizzbuzz = [
    "FizzBuzz" if n % 15 == 0 else "Fizz" if n % 3 == 0 else "Buzz" if n % 5 == 0 else str(n)
    for n in range(1, 21)
]
print(fizzbuzz)

# Challenge 4: Extract email domains
emails = ["alice@example.com", "bob@test.org", "charlie@company.co.uk"]
domains = [email.split("@")[1] for email in emails]
print(domains)  # ['example.com', 'test.org', 'company.co.uk']
```

### Related Topics

- for loops
- map(), filter() built-ins
- Generator expressions
- Set and dict comprehensions

## Conditional comprehensions

### What It Is

A conditional comprehension includes an `if` clause that filters elements before the expression is evaluated. The `if` can appear at the end (filtering) or as part of a ternary expression (value selection). Multiple `if` conditions can be chained.

### Why It Is Important

Conditional comprehensions combine mapping and filtering into a single expression, eliminating the need for separate filter + map calls or nested if-blocks in loops. They are more readable and efficient than the equivalent `map(filter(...))` chains.

### How It Works Internally

The `if` clause is compiled into a `JUMP_IF_FALSE_OR_POP` bytecode — each iteration evaluates the condition; if falsy, it jumps to the next `FOR_ITER` without executing the expression. Multiple `if` clauses compile to multiple conditional jumps. The ternary `if` inside the expression (`x if cond else y`) is a different construct — it always executes the expression but selects which value to produce.

### Syntax

```python
# Filter at the end
[expr for item in iterable if condition]

# Multiple conditions (AND)
[expr for item in iterable if cond1 if cond2]

# Ternary in expression
[expr_true if condition else expr_false for item in iterable]

# Filter + ternary combined
[expr_true if c1 else expr_false for item in iterable if condition]
```

### Beginner Examples

```python
# Filter even numbers
evens = [n for n in range(20) if n % 2 == 0]
print(evens)  # [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

# Filter strings by length
words = ["hi", "hello", "world", "python", "code"]
long_words = [w for w in words if len(w) >= 5]
print(long_words)  # ['hello', 'world', 'python']

# Filter None values
data = [1, None, 3, None, 5, 6, None]
clean = [x for x in data if x is not None]
print(clean)  # [1, 3, 5, 6]

# Ternary: label numbers
labels = ["even" if n % 2 == 0 else "odd" for n in range(10)]
print(labels)  # ['even', 'odd', 'even', 'odd', ...]

# Filter + transform combined
nums = [1, -2, 3, -4, 5, -6]
absolute_positive = [abs(x) for x in nums if x > 0]
print(absolute_positive)  # [1, 3, 5]
```

### Intermediate Examples

```python
# Multiple conditions (AND)
divisible = [n for n in range(100) if n % 3 == 0 if n % 5 == 0]
print(divisible[:10])  # [0, 15, 30, 45, 60, 75, 90]

# Equivalent single if:
[n for n in range(100) if n % 3 == 0 and n % 5 == 0]

# Complex ternary with multiple conditions
categories = [
    "high" if n > 5 else "medium" if n > 0 else "low"
    for n in range(-3, 9)
]
print(categories)

# Filtering with index (using enumerate)
items = ["a", "b", "c", "d", "e"]
at_even_indices = [val for idx, val in enumerate(items) if idx % 2 == 0]
print(at_even_indices)  # ['a', 'c', 'e']

# Filter by type
mixed = [1, "hello", 3.14, "world", 42, "python"]
strings = [x for x in mixed if isinstance(x, str)]
numbers = [x for x in mixed if isinstance(x, (int, float))]
print(strings)  # ['hello', 'world', 'python']
print(numbers)  # [1, 3.14, 42]

# Filter with function
def is_valid(item):
    return item is not None and item > 0

valid = [x for x in [1, 0, None, -1, 5, 10] if is_valid(x)]
print(valid)  # [1, 5, 10]
```

### Advanced Examples

```python
# Conditional in nested comprehension
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
evens_flat = [x for row in matrix for x in row if x % 2 == 0]
print(evens_flat)  # [2, 4, 6, 8]

# Filter with walrus operator (Python 3.8+)
import re
texts = ["hello 42", "world", "python 3.14"]
with_numbers = [t for t in texts if (m := re.search(r"\d+", t))]
print(with_numbers)  # ['hello 42', 'python 3.14']

# Multiple conditions with different operations
data = range(-5, 6)
result = [
    x * 10 if x > 0 else x * -1 if x < 0 else 0
    for x in data
    if x != 0  # Skip zero
]
print(result)  # [5, 4, 3, 2, 1, 10, 20, 30, 40, 50]

# Filter by membership in set
valid_tokens = {"foo", "bar", "baz"}
tokens = ["foo", "qux", "bar", "quux", "baz"]
filtered = [t for t in tokens if t in valid_tokens]
print(filtered)  # ['foo', 'bar', 'baz']

# Filtering with try/except (via helper)
def try_int(x):
    try:
        return int(x), True
    except (ValueError, TypeError):
        return None, False

values = ["42", "abc", "3.14", "100"]
parsed = [v for v in values if (x, ok := try_int(v)) and ok]

# Equivalent: use helper that returns Optional
def safe_int(x):
    try:
        return int(x)
    except (ValueError, TypeError):
        return None

parsed2 = [safe_int(x) for x in values if safe_int(x) is not None]
print(parsed2)  # [42, 100]
```

### Real-World Use Cases

- **Data filtering**: Remove invalid/outlier rows from datasets
- **Value normalization**: Transform while filtering out-of-range values
- **Type-specific processing**: Process only strings from mixed lists
- **Threshold application**: Apply formatting only to values above a threshold
- **Pagination filtering**: Select specific result pages
- **Log filtering**: Extract log entries matching criteria
- **Validation**: Collect errors/valid items separately

### Common Mistakes

```python
# Mistake 1: Confusing filter-if with ternary-if
# Filter:  [x for x in items if x > 0]     -- skips items
# Ternary: [x if x > 0 else 0 for x in items]  -- transforms all items

# Mistake 2: if after variable use (must be after for clause)
# Wrong: [x if x > 0 for x in items]  -- SyntaxError
# Correct: [x for x in items if x > 0]

# Mistake 3: Using filter when you mean map
# [x**2 for x in nums if x > 0]  -- filters THEN squares
# [x**2 if x > 0 else x for x in nums]  -- squares positive, keeps rest

# Mistake 4: Complex conditions that hurt readability
# Bad:
result = [a*b for a in range(10) for b in range(10) if a > b if a % 2 == 0 if b % 3 == 0]
# Better:
result = [a*b for a in range(10) for b in range(10) if a > b and a % 2 == 0 and b % 3 == 0]
# Best as nested loop:
result = []
for a in range(10):
    for b in range(10):
        if a > b and a % 2 == 0 and b % 3 == 0:
            result.append(a * b)
```

### Best Practices

- Place `if` at the end of the comprehension for filtering
- Use ternary (`x if cond else y`) for value selection (always produces n results)
- Chain `if` conditions with `and` instead of multiple `if` clauses
- Keep conditions simple; extract complex predicates into functions
- Prefer readability over compactness for conditional logic
- Filter before mapping to avoid unnecessary work

### Performance Considerations

Filter `if` prevents the expression from being evaluated for rejected items — more efficient than mapping then filtering. Multiple `if` clauses have the same cost as a single `and`. Ternary expressions are always evaluated (the condition check is fast, but both branches are not — only the selected one executes). For large datasets, consider generator expressions if you don't need the entire list at once.

### Interview Questions

1. What is the difference between `[x for x in items if cond]` and `[x if cond else y for x in items]`?
2. How do you filter by type in a comprehension?
3. Can you have multiple if conditions in a comprehension?
4. How do you use enumerate in a filtered comprehension?
5. What is the walrus operator and how does it relate to comprehensions?

### Coding Challenges

```python
# Challenge 1: Separate even and odd
nums = range(1, 21)
evens = [n for n in nums if n % 2 == 0]
odds = [n for n in nums if n % 2 == 1]
print(evens)
print(odds)

# Challenge 2: Filter strings containing substring
words = ["cat", "caterpillar", "dog", "catalog", "bird"]
cat_words = [w for w in words if "cat" in w]
print(cat_words)  # ['cat', 'caterpillar', 'catalog']

# Challenge 3: Conditional square or cube
nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
result = [x**2 if x % 2 == 0 else x**3 for x in nums]
print(result)

# Challenge 4: Filter and transform dict values
data = {"a": 10, "b": 0, "c": -5, "d": 20, "e": -1}
positive = {k: v for k, v in data.items() if v > 0}
print(positive)  # {'a': 10, 'd': 20}
```

### Related Topics

- filter() built-in
- Ternary expressions (x if cond else y)
- Walrus operator (:=)
- Boolean operators (and, or, not)

## Nested comprehensions

### What It Is

Nested comprehensions have multiple `for` clauses, creating a Cartesian product or flattening nested structures. They follow the same ordering as nested for-loops: the leftmost `for` is the outermost loop. Nested comprehensions can include conditions at any level.

### Why It Is Important

Nested comprehensions replace multi-level nested for-loops with concise expressions. They are essential for flattening matrices, computing Cartesian products, and applying operations across multiple dimensions.

### How It Works Internally

Each additional `for` clause in a comprehension compiles to one more `GET_ITER`/`FOR_ITER` pair, nested inside the previous. The `LIST_APPEND` operation always happens at the innermost level. For `[expr for a in A for b in B]`, CPython iterates `A`, then for each `a`, iterates `B`, evaluates `expr`, and appends the result.

### Syntax

```python
# Flattening
[item for outer in nested for item in outer]

# Cartesian product
[(a, b) for a in A for b in B]

# Multi-level with conditions
[expr for a in A if cond1 for b in B if cond2]
```

### Beginner Examples

```python
# Flatten a matrix
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [num for row in matrix for num in row]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Cartesian product
colors = ["red", "green"]
sizes = ["S", "M", "L"]
products = [(c, s) for c in colors for s in sizes]
print(products)
# [('red','S'), ('red','M'), ('red','L'), ('green','S'), ('green','M'), ('green','L')]

# Multiplication table
table = [i * j for i in range(1, 6) for j in range(1, 6)]
# [1,2,3,4,5, 2,4,6,8,10, 3,6,9,12,15, ...]

# Flatten with string join
words = [["hello", "world"], ["python", "rocks"]]
all_words = [word for group in words for word in group]
print(all_words)  # ['hello', 'world', 'python', 'rocks']
```

### Intermediate Examples

```python
# Matrix transposition
matrix = [[1, 2, 3], [4, 5, 6]]
transposed = [[row[i] for row in matrix] for i in range(3)]
print(transposed)  # [[1, 4], [2, 5], [3, 6]]

# Filtered flatten
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
evens = [x for row in matrix for x in row if x % 2 == 0]
print(evens)  # [2, 4, 6, 8]

# Deep flatten (2 levels)
deep = [[[1, 2], [3, 4]], [[5, 6], [7, 8]]]
flat = [z for outer in deep for inner in outer for z in inner]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8]

# Nested comprehension with condition on outer
pairs = [(a, b) for a in range(5) if a % 2 == 0 for b in range(5) if b % 2 == 1]
print(pairs)  # [(0,1), (0,3), (2,1), (2,3), (4,1), (4,3)]

# 2D comprehension to create matrix
matrix = [[i * j for j in range(1, 4)] for i in range(1, 4)]
print(matrix)  # [[1, 2, 3], [2, 4, 6], [3, 6, 9]]
```

### Advanced Examples

```python
# Flatten arbitrary nesting recursively
def deep_flatten(lst):
    result = []
    for item in lst:
        if isinstance(item, list):
            result.extend(deep_flatten(item))
        else:
            result.append(item)
    return result

nested = [1, [2, [3, 4], 5], [6, 7], 8]
print(deep_flatten(nested))  # [1, 2, 3, 4, 5, 6, 7, 8]

# Pythagorean triplets
triplets = [
    (a, b, c)
    for a in range(1, 20)
    for b in range(a, 20)
    for c in range(b, 20)
    if a**2 + b**2 == c**2
]
print(triplets)  # [(3, 4, 5), (5, 12, 13), (6, 8, 10), (8, 15, 17), (9, 12, 15)]

# Cross product with filtering
A = [1, 2, 3]
B = [4, 5, 6]
pairs = [(a, b, a*b) for a in A for b in B if a * b > 10]
print(pairs)  # [(2, 6, 12), (3, 4, 12), (3, 5, 15), (3, 6, 18)]

# Nested dict comprehension
matrix = [[1, 2], [3, 4]]
labeled = {f"row_{i}": {f"col_{j}": val for j, val in enumerate(row)} for i, row in enumerate(matrix)}
print(labeled)  # {'row_0': {'col_0': 1, 'col_1': 2}, 'row_1': {'col_0': 3, 'col_1': 4}}

# Transpose with zip vs nested comprehension
matrix = [[1, 2, 3], [4, 5, 6]]
t1 = list(zip(*matrix))  # [(1, 4), (2, 5), (3, 6)]
t2 = [[row[i] for row in matrix] for i in range(len(matrix[0]))]
print(t1 == t2)  # True

# Flatten dict of lists
dict_of_lists = {"a": [1, 2], "b": [3, 4], "c": [5, 6]}
flattened = [(k, v) for k, lst in dict_of_lists.items() for v in lst]
print(flattened)  # [('a',1), ('a',2), ('b',3), ('b',4), ('c',5), ('c',6)]
```

### Real-World Use Cases

- **Matrix operations**: Transposition, flattening, reshaping
- **Database joins**: Simulating cross-product of query results
- **Configuration expansion**: Generating all combinations of settings
- **Test parameterization**: Generating test cases from parameter tuples
- **Coordinate generation**: Grid coordinates for image processing
- **Combinatorics**: Generating combinations for algorithm testing
- **Multi-dimensional data**: Processing tensors and arrays

### Common Mistakes

```python
# Mistake 1: Wrong order of for clauses
# [x for x in row for row in matrix]  -- NameError: row not defined
# Correct: [x for row in matrix for x in row]

# Mistake 2: Over-nesting reduces readability
# Too many levels:
result = [a*b for a in X for b in Y for c in Z if a > b if b > c if a + c > 10]
# Use loops instead:
for a in X:
    for b in Y:
        for c in Z:
            if a > b and b > c and a + c > 10:
                result.append(a * b)

# Mistake 3: Forgetting that inner loop iterates fully for each outer item
# With 100 outer and 100 inner, the expression runs 10000 times

# Mistake 4: Unnecessary nested comprehension for simple tasks
# Instead of [[x for x in row] for row in mtx]
# Use: [list(row) for row in mtx]
```

### Best Practices

- Limit nesting to 2 levels for readability
- Use `zip()` or `itertools.product()` for complex multi-iteration
- Break deep nesting into regular for-loops
- Name the loop variables clearly
- Consider readability for other developers first
- For 3+ levels, prefer explicit loops over comprehensions
- Use `itertools.chain.from_iterable()` for flattening

### Performance Considerations

Nested comprehensions are O(n * m * ...) where each level multiplies the iteration count. They are typically as fast as the equivalent nested loops, but readability degrades quickly. For large Cartesian products, consider `itertools.product()` which produces tuples lazily. For flattening, `itertools.chain.from_iterable()` uses less memory than a comprehension that creates an intermediate list.

### Interview Questions

1. How do you flatten a 2D matrix using a comprehension?
2. What is the order of evaluation in nested comprehensions?
3. How do you transpose a matrix with nested comprehensions?
4. When should you avoid nested comprehensions?
5. How does `itertools.product()` compare to nested comprehensions?
6. How do you flatten a list of lists of lists?

### Coding Challenges

```python
# Challenge 1: Flatten irregular nesting
def deep_flatten_comprehension(nested):
    return [x for item in nested for sublist in (item if isinstance(item, list) else [item]) for x in (sublist if isinstance(sublist, list) else [sublist])]

# Simpler with recursion:
def flatten(lst):
    result = []
    for item in lst:
        if isinstance(item, list):
            result.extend(flatten(item))
        else:
            result.append(item)
    return result

nested = [1, [2, [3, 4], 5], [6, 7], 8]
print(flatten(nested))

# Challenge 2: Generate all pairs summing to target
def sum_pairs(nums, target):
    return [(a, b) for a in nums for b in nums if a + b == target and a <= b]

print(sum_pairs([1, 2, 3, 4, 5], 6))  # [(1,5), (2,4), (3,3)]

# Challenge 3: Create identity matrix
def identity(n):
    return [[1 if i == j else 0 for j in range(n)] for i in range(n)]

print(identity(4))  # [[1,0,0,0], [0,1,0,0], [0,0,1,0], [0,0,0,1]]

# Challenge 4: Word frequency matrix
sentences = ["hello world", "python is great", "hello python"]
words = list(set(w for s in sentences for w in s.split()))
matrix = [[s.split().count(w) for w in words] for s in sentences]
print(words)
print(matrix)
```

### Related Topics

- itertools.product()
- itertools.chain()
- Nested for loops
- Matrix transposition
- List flattening techniques
- zip() and zip(*)
