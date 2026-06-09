# List Comprehensions - [expr for item in iterable if condition]

## Introduction

List comprehensions provide a concise way to create lists by applying an expression to each element of an iterable, optionally filtering elements with a condition. Python also supports set comprehensions, dict comprehensions, and generator expressions (which use similar syntax but produce values on demand rather than storing them).

## Why It Is Important

Comprehensions are important because:
- They are more concise and readable than equivalent for loops
- They are generally faster than manual loops
- They reduce code duplication
- They support filtering with conditions
- They can be nested for complex transformations
- Generator expressions save memory for large datasets

## Syntax

```python
# List comprehension
[expression for item in iterable]
[expression for item in iterable if condition]
[expression if condition else alt for item in iterable]

# Set comprehension
{expression for item in iterable}

# Dict comprehension
{key_expr: value_expr for item in iterable}

# Generator expression (note: parentheses, not brackets)
(expression for item in iterable)

# Nested comprehension
[expression for outer in outer_iterable for inner in outer]
```

## Examples

### Basic List Comprehensions

```python
# Traditional for loop vs comprehension
numbers = [1, 2, 3, 4, 5]

# For loop
squares_loop = []
for n in numbers:
    squares_loop.append(n ** 2)
print(f"For loop: {squares_loop}")

# List comprehension
squares_comp = [n ** 2 for n in numbers]
print(f"Comprehension: {squares_comp}")

# Both produce the same result

# More examples
cubes = [n ** 3 for n in range(1, 6)]
print(f"Cubes: {cubes}")

even_numbers = [n for n in range(20) if n % 2 == 0]
print(f"Evens: {even_numbers}")

# String operations with comprehension
fruits = ["apple", "banana", "cherry", "date"]
upper_fruits = [f.upper() for f in fruits]
print(f"Upper: {upper_fruits}")

fruit_lengths = [len(f) for f in fruits]
print(f"Lengths: {fruit_lengths}")
```

### Conditional Comprehensions

```python
numbers = range(-5, 6)

# With if condition (filter)
positive = [n for n in numbers if n > 0]
print(f"Positive: {positive}")

# With if-else (transform)
label = ["even" if n % 2 == 0 else "odd" for n in range(10)]
print(f"Labels: {label}")

# Multiple conditions
divisible = [n for n in range(50) if n % 3 == 0 if n % 5 == 0]
print(f"Divisible by 3 and 5: {divisible}")

# Equivalent to: n % 3 == 0 and n % 5 == 0

# Complex condition
result = [
    "high" if n > 5 else "medium" if n > 0 else "low"
    for n in range(-3, 9)
]
print(f"Categories: {result}")
```

## Beginner Examples

```python
# Square numbers
squares = [x ** 2 for x in range(10)]
print(f"Squares: {squares}")

# Even numbers from a list
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
evens = [n for n in numbers if n % 2 == 0]
print(f"Evens: {evens}")

# Convert temperatures
celsius = [0, 10, 20, 30, 40]
fahrenheit = [(c * 9/5) + 32 for c in celsius]
print(f"Fahrenheit: {fahrenheit}")

# Extract names from list of dicts
people = [
    {"name": "Alice", "age": 30},
    {"name": "Bob", "age": 25},
    {"name": "Charlie", "age": 35}
]
names = [p["name"] for p in people]
print(f"Names: {names}")

# Filter out None values
data = [1, None, 3, None, 5, 6, None]
clean = [x for x in data if x is not None]
print(f"Clean: {clean}")

# String transformations
words = ["hello", "world", "python", "programming"]
uppercase = [w.upper() for w in words]
capitalized = [w.capitalize() for w in words]
print(f"Uppercase: {uppercase}")
```

## Intermediate Examples

### Nested Comprehensions

```python
# Flatten a matrix
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flattened = [num for row in matrix for num in row]
print(f"Flattened: {flattened}")

# Equivalent to:
result = []
for row in matrix:
    for num in row:
        result.append(num)

# Transpose a matrix
transposed = [[row[i] for row in matrix] for i in range(len(matrix[0]))]
print(f"Transposed: {transposed}")

# Create a multiplication table
mult_table = [[i * j for j in range(1, 6)] for i in range(1, 6)]
print("Multiplication table:")
for row in mult_table:
    print(f"  {row}")

# Cartesian product
colors = ["red", "green", "blue"]
sizes = ["S", "M", "L"]
products = [(c, s) for c in colors for s in sizes]
print(f"Products: {products}")
```

### Using Functions in Comprehensions

```python
# With lambda
numbers = [1, 2, 3, 4, 5]
doubled = list(map(lambda x: x * 2, numbers))
print(f"Doubled: {doubled}")

# With custom function
def is_prime(n):
    if n <= 1:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True

primes = [n for n in range(2, 50) if is_prime(n)]
print(f"Primes: {primes}")

# Method calls
strings = ["hello", "world", "python"]
reversed_strings = [s[::-1] for s in strings]
print(f"Reversed: {reversed_strings}")

# Multiple operations
data = [3, 1, 4, 1, 5, 9, 2, 6]
processed = [
    n * 2 if n % 2 == 0 else n * 3
    for n in data
    if n > 2
]
print(f"Processed: {processed}")
```

## Advanced Examples

### Dict Comprehensions

```python
# Basic dict comprehension
squares = {x: x ** 2 for x in range(6)}
print(f"Squares: {squares}")

# From two lists
keys = ["name", "age", "city"]
values = ["Alice", 30, "NYC"]
person = {k: v for k, v in zip(keys, values)}
print(f"Person: {person}")

# Conditional dict comprehension
numbers = range(10)
even_squares = {n: n**2 for n in numbers if n % 2 == 0}
print(f"Even squares: {even_squares}")

# Inverting a dictionary
original = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in original.items()}
print(f"Inverted: {inverted}")

# Transforming values
prices = {"apple": 0.50, "banana": 0.30, "orange": 0.80}
with_tax = {item: price * 1.1 for item, price in prices.items()}
print(f"With tax: {with_tax}")

# Filtering a dictionary
grades = {"Alice": 85, "Bob": 92, "Charlie": 78, "Diana": 95}
honors = {name: grade for name, grade in grades.items() if grade >= 90}
print(f"Honors: {honors}")
```

### Set Comprehensions

```python
# Basic set comprehension
squares = {x ** 2 for x in range(10)}
print(f"Squares set: {squares}")

# Removing duplicates with comprehension
numbers = [1, 2, 3, 2, 4, 3, 5, 1, 6]
unique_squares = {n ** 2 for n in numbers}
print(f"Unique squares: {unique_squares}")

# Conditional set comprehension
even_set = {x for x in range(20) if x % 2 == 0}
print(f"Even set: {even_set}")

# Set of character frequencies
text = "mississippi"
char_set = {c for c in text}
print(f"Unique chars: {char_set}")
```

### Generator Expressions

```python
# Generator expression (memory efficient)
import sys

# List comprehension - creates full list in memory
list_squares = [x ** 2 for x in range(1000000)]
print(f"List size: {sys.getsizeof(list_squares)} bytes")

# Generator expression - produces values on demand
gen_squares = (x ** 2 for x in range(1000000))
print(f"Generator size: {sys.getsizeof(gen_squares)} bytes")

# Using generator expression
gen = (x ** 2 for x in range(5))
print(f"Generator type: {type(gen)}")
for val in gen:
    print(f"  {val}")

# Generator as function argument
total = sum(x ** 2 for x in range(100))
print(f"Sum of squares: {total}")

# Generator with condition
avg_even = sum(x for x in range(100) if x % 2 == 0) / 50
print(f"Average of evens: {avg_even}")

# Generator expression vs list comprehension
def process_with_list(items):
    return [x * 2 for x in items if x > 0]

def process_with_gen(items):
    return (x * 2 for x in items if x > 0)

data = [1, -2, 3, -4, 5]
print(f"List result: {process_with_list(data)}")
gen = process_with_gen(data)
print(f"Generator result: {list(gen)}")
```

## Real-World Use Cases

- **Data Transformation**: Converting, filtering, and reshaping data
- **Data Cleaning**: Removing None values, normalizing formats
- **API Responses**: Extracting specific fields from JSON
- **Configuration Parsing**: Building dictionaries from config files
- **Feature Engineering**: Creating derived features in ML pipelines
- **Log Analysis**: Extracting patterns from log files
- **Database Queries**: Formatting query results

## Common Mistakes

```python
# Mistake 1: Using comprehension for side effects
# Bad (comprehension used for side effects):
[print(x) for x in range(5)]  # Creates a list [None, None, None, None, None]
# Good:
for x in range(5):
    print(x)

# Mistake 2: Overly complex comprehensions
# Hard to read:
result = [a * b for a in range(10) for b in range(10) 
          if a > b if a % 2 == 0 if b % 3 == 0]
# Better as loops:
result = []
for a in range(10):
    for b in range(10):
        if a > b and a % 2 == 0 and b % 3 == 0:
            result.append(a * b)

# Mistake 3: Forgetting that comprehension creates a new object
original = [1, 2, 3]
squared = [x ** 2 for x in original]  # New list, original unchanged

# Mistake 4: Using list comprehension when generator is better
# sum([x**2 for x in range(10**7)])  # Uses lots of memory
# Better:
# sum(x**2 for x in range(10**7))  # Generator, memory efficient

# Mistake 5: Accidental variable leakage (Python 2, fixed in Python 3)
# In Python 3, the loop variable in comprehensions is local
x = "outer"
result = [x for x in range(5)]
print(x)  # Still "outer" in Python 3

# Mistake 6: Confusing comprehension with generator
comp = [x**2 for x in range(5)]  # List comprehension
gen = (x**2 for x in range(5))   # Generator expression
print(type(comp))  # <class 'list'>
print(type(gen))   # <class 'generator'>
```

## Best Practices

- Use comprehensions for simple transformations (fit on 1-2 lines)
- Avoid nested comprehensions with more than 2 levels
- Use generator expressions for large sequences to save memory
- Use dict comprehensions for key-value transformations
- Use set comprehensions for deduplication
- Prefer comprehensions over `map()` and `filter()` for readability
- Don't use comprehensions for side effects (use for loops instead)
- Use `if` at the end for filtering, not at the beginning
- Use parentheses for generator expressions in function calls
- Keep expressions simple; extract complex logic to functions

## Interview Questions

1. What is a list comprehension and how does it differ from a for loop?
2. What is the difference between list comprehensions and generator expressions?
3. How do you write a conditional list comprehension?
4. How do you create a dict comprehension?
5. How do you flatten a nested list using comprehension?
6. What are the performance implications of comprehensions vs loops?
7. Can you use else in a list comprehension?
8. How do you write a set comprehension?
9. When should you use a generator expression instead of list comprehension?
10. What are the limitations of comprehensions?

## Coding Challenges

```python
# Challenge 1: Find all numbers divisible by both 3 and 5
result = [n for n in range(1, 101) if n % 3 == 0 and n % 5 == 0]
print(f"Divisible by 3 and 5: {result}")

# Challenge 2: Flatten a list of lists
nested = [[1, 2], [3, 4, 5], [6], [7, 8, 9]]
flat = [item for sublist in nested for item in sublist]
print(f"Flattened: {flat}")

# Challenge 3: Create a dictionary mapping words to their lengths
sentence = "the quick brown fox jumps over the lazy dog"
word_lengths = {word: len(word) for word in sentence.split()}
print(f"Word lengths: {word_lengths}")

# Challenge 4: Transpose a matrix using comprehension
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
transposed = [[row[i] for row in matrix] for i in range(3)]
print(f"Transposed: {transposed}")

# Challenge 5: Prime numbers with comprehension
def is_prime(n):
    if n <= 1:
        return False
    return all(n % i != 0 for i in range(2, int(n ** 0.5) + 1))

primes = [n for n in range(2, 100) if is_prime(n)]
print(f"Primes up to 100: {primes}")

# Challenge 6: FizzBuzz with comprehension
fizzbuzz = [
    "FizzBuzz" if n % 15 == 0 
    else "Fizz" if n % 3 == 0 
    else "Buzz" if n % 5 == 0 
    else str(n)
    for n in range(1, 21)
]
print(f"FizzBuzz: {fizzbuzz}")

# Challenge 7: Extract email domains
emails = ["alice@example.com", "bob@test.org", "charlie@company.co.uk"]
domains = [email.split("@")[1] for email in emails]
print(f"Domains: {domains}")

# Challenge 8: Pythagorean triplets
triplets = [
    (a, b, c) 
    for a in range(1, 20) 
    for b in range(a, 20) 
    for c in range(b, 20) 
    if a**2 + b**2 == c**2
]
print(f"Pythagorean triplets: {triplets}")
```

## Summary

List comprehensions provide concise syntax for creating lists from iterables with optional filtering. Set and dict comprehensions extend this pattern to other collection types. Generator expressions use similar syntax but produce values lazily, saving memory. Comprehensions are more readable and faster than equivalent loops for simple transformations. Complex comprehensions should be replaced with regular loops for clarity.

## Related Topics

- Lists
- Dictionaries
- Sets
- Generators and Yield
- Lambda Functions
- map, filter, reduce
- Iterators and Iterables
- Memory Management
