# Tuples - tuple() constructor, immutability, packing/unpacking

## Introduction

Tuples are ordered, immutable sequences in Python, created with parentheses `()`. Unlike lists, tuples cannot be modified after creation (no append, remove, or item assignment). Their immutability makes them hashable (usable as dictionary keys) and suitable for representing fixed collections of data. Tuples support packing, unpacking, and named fields via `namedtuple`.

## Why It Is Important

Tuples are important because:
- They provide data integrity (immutable, can't be accidentally modified)
- They are hashable, so they can be used as dictionary keys and set elements
- They are more memory-efficient than lists
- They enable tuple unpacking for elegant code
- They represent fixed data structures (coordinates, records, etc.)
- They are faster than lists for iteration and access

## Syntax

```python
# Creating tuples
empty = ()
single = (1,)  # Note: comma required for single element
multiple = (1, 2, 3)
without_parens = 1, 2, 3  # Tuple packing

# Tuple constructor
from_list = tuple([1, 2, 3])
from_string = tuple("hello")

# Accessing elements
first = multiple[0]
last = multiple[-1]

# Unpacking
a, b, c = multiple
```

## Examples

### Creating Tuples

```python
empty = ()
print(f"Empty tuple: {empty}, type: {type(empty)}")

single = (5,)  # Tuple with one element
not_a_tuple = (5)  # Just the integer 5
print(f"Single element: {single}, type: {type(single)}")
print(f"Not a tuple: {not_a_tuple}, type: {type(not_a_tuple)}")

coordinates = (10, 20)
rgb = (255, 128, 64)
person = ("Alice", 30, "Engineer")
print(f"Coordinates: {coordinates}")
print(f"RGB: {rgb}")
print(f"Person: {person}")

point = 3, 4
print(f"Packed tuple: {point}, type: {type(point)}")

from_list = tuple([1, 2, 3, 4, 5])
from_string = tuple("Python")
from_range = tuple(range(5))
print(f"From list: {from_list}")
print(f"From string: {from_string}")
print(f"From range: {from_range}")

nested = (1, (2, 3), (4, (5, 6)))
print(f"Nested: {nested}")
```

### Tuple Immutability

```python
t = (1, 2, 3)
print(f"Tuple: {t}")
print(f"First element: {t[0]}")
print(f"Last element: {t[-1]}")

# These would cause errors:
# t[0] = 10     # TypeError
# t.append(4)   # AttributeError

# But tuples can contain mutable objects
t_mutable = (1, [2, 3], "hello")
t_mutable[1].append(4)  # The list INSIDE the tuple can be modified
print(f"After modifying inner list: {t_mutable}")

# Tuple concatenation
t1 = (1, 2, 3)
t2 = (4, 5, 6)
combined = t1 + t2
print(f"Concatenated: {combined}")

# Tuple repetition
print(f"Repeated: {t1 * 3}")
```

## Beginner Examples

```python
# Tuple unpacking
point = (10, 20)
x, y = point
print(f"x={x}, y={y}")

# Swapping variables
a, b = 5, 10
print(f"Before: a={a}, b={b}")
a, b = b, a
print(f"After: a={a}, b={b}")

# Returning multiple values
def get_stats(numbers):
    return min(numbers), max(numbers), sum(numbers) / len(numbers)

values = [3, 7, 1, 9, 4]
minimum, maximum, average = get_stats(values)
print(f"Min: {minimum}, Max: {maximum}, Avg: {average:.2f}")

# Iterating over a tuple
colors = ("red", "green", "blue")
print("Colors:")
for color in colors:
    print(f"  - {color}")

# Checking membership
print(f"'green' in colors: {'green' in colors}")
print(f"'yellow' not in colors: {'yellow' not in colors}")

# Tuple methods
t = (1, 2, 2, 3, 2, 4)
print(f"Count of 2: {t.count(2)}")  # 3
print(f"Index of 3: {t.index(3)}")  # 3
```

## Intermediate Examples

```python
# Extended unpacking (Python 3+)
first, *middle, last = (1, 2, 3, 4, 5)
print(f"First: {first}")
print(f"Middle: {middle}")
print(f"Last: {last}")

# Using tuples as dictionary keys
locations = {
    (40.7128, -74.0060): "New York",
    (51.5074, -0.1278): "London",
    (35.6762, 139.6503): "Tokyo"
}
print(f"Location at NYC: {locations[(40.7128, -74.0060)]}")

# Unpacking in for loops
coordinates = [(1, 2), (3, 4), (5, 6)]
for x, y in coordinates:
    print(f"Point: ({x}, {y}), Sum: {x + y}")

# Enumerate returns tuples
fruits = ("apple", "banana", "cherry")
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")

# Zip returns tuples
names = ("Alice", "Bob", "Charlie")
scores = (85, 92, 78)
for name, score in zip(names, scores):
    print(f"{name}: {score}")

# Converting between list and tuple
my_list = [1, 2, 3]
my_tuple = tuple(my_list)
my_list_converted = list(my_tuple)
print(f"List: {my_list}")
print(f"Tuple: {my_tuple}")

# Sorting a tuple (returns a list)
unsorted = (3, 1, 4, 1, 5, 9)
sorted_tuple = tuple(sorted(unsorted))
print(f"Sorted: {sorted_tuple}")
```

## Advanced Examples

```python
# namedtuple
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])
Student = namedtuple("Student", ["name", "age", "grade"])

p1 = Point(10, 20)
student = Student("Alice", 20, "A")

print(f"Point: {p1}")
print(f"Access by name: p1.x={p1.x}, p1.y={p1.y}")
print(f"Access by index: p1[0]={p1[0]}, p1[1]={p1[1]}")
print(f"Student: {student}")

print(f"Fields: {Point._fields}")
print(f"As dict: {p1._asdict()}")
p3 = p1._replace(x=100)
print(f"Replaced: {p3}")

# Memory comparison
import sys
list_data = list(range(1000))
tuple_data = tuple(range(1000))
print(f"List size: {sys.getsizeof(list_data)} bytes")
print(f"Tuple size: {sys.getsizeof(tuple_data)} bytes")

# Generator expression
gen = (x ** 2 for x in range(10))
print(f"Generator type: {type(gen)}")
print(f"As tuple: {tuple(gen)}")

# Performance comparison
import time
numbers_list = list(range(100000))
numbers_tuple = tuple(range(100000))

start = time.perf_counter()
for _ in range(100):
    for item in numbers_list:
        pass
list_time = time.perf_counter() - start

start = time.perf_counter()
for _ in range(100):
    for item in numbers_tuple:
        pass
tuple_time = time.perf_counter() - start

print(f"List iteration: {list_time:.4f}s")
print(f"Tuple iteration: {tuple_time:.4f}s")
```

## Real-World Use Cases

- **Database Records**: Immutable row data from queries
- **Function Returns**: Returning multiple values safely
- **Dictionary Keys**: Using coordinates as hashable keys
- **Configurations**: Storing immutable configuration values
- **Data Transfer**: Passing fixed data between functions
- **Enumeration**: Defining constant sets of related values

## Common Mistakes

```python
# Mistake 1: Forgetting comma for single-element tuple
single = (5)
print(type(single))  # <class 'int'>
# Correct:
single = (5,)
print(type(single))  # <class 'tuple'>

# Mistake 2: Trying to modify a tuple
t = (1, 2, 3)
# t[0] = 10  # TypeError

# Mistake 3: Confusing parentheses for grouping with tuple
result = (2 + 3) * 4  # 20 (just grouping)

# Mistake 4: Unpacking mismatch
# a, b = (1, 2, 3)  # ValueError: too many values to unpack
# Correct extended unpacking:
a, *rest = (1, 2, 3)

# Mistake 5: Using tuple when list is more appropriate
# If you need to modify the collection, use a list
```

## Best Practices

- Use tuples for fixed, immutable collections
- Use tuples for dictionary keys (lists can't be used)
- Use namedtuples for self-documenting code
- Use tuple unpacking for clean variable assignment
- Use extended unpacking (*) for variable-length tuples
- Use tuples over lists for memory efficiency
- Use parentheses for clarity even when optional

## Interview Questions

1. How are tuples different from lists?
2. Why can tuples be used as dictionary keys but lists cannot?
3. What is tuple unpacking? Give examples.
4. What is a namedtuple and when would you use it?
5. Are tuples always truly immutable? Explain.
6. How do you create a single-element tuple?
7. What is extended tuple unpacking?
8. Compare memory usage of tuples vs lists.
9. How do you convert between tuples and lists?
10. What performance advantages do tuples have?

## Coding Challenges

```python
# Challenge 1: Swap using tuple unpacking
def swap(a, b):
    return b, a

x, y = 5, 10
x, y = swap(x, y)
print(f"x={x}, y={y}")

# Challenge 2: Analyze numbers returning tuple
def analyze_numbers(*nums):
    return min(nums), max(nums), sum(nums)/len(nums)

result = analyze_numbers(3, 7, 1, 9, 4)
print(f"Min: {result[0]}, Max: {result[1]}, Avg: {result[2]:.2f}")

# Challenge 3: namedtuple inventory
Product = namedtuple("Product", ["name", "price", "quantity"])
inventory = [
    Product("Apple", 0.50, 100),
    Product("Banana", 0.30, 150),
    Product("Orange", 0.80, 75),
]
total = sum(p.price * p.quantity for p in inventory)
print(f"Total inventory value: ${total:.2f}")

# Challenge 4: Convert list of tuples to dict
pairs = [("a", 1), ("b", 2), ("c", 3)]
result = dict(pairs)
print(f"Dictionary: {result}")

# Challenge 5: Transpose using zip
columns = [("Alice", "Bob", "Charlie"), (85, 92, 78)]
names, scores = zip(*columns)
print(f"Names: {names}")
print(f"Scores: {scores}")
```

## Summary

Tuples are immutable, ordered sequences ideal for fixed collections. They are hashable (usable as dictionary keys), memory-efficient, and support powerful unpacking features. namedtuple extends tuples with named fields for self-documenting data structures. Choosing tuples over lists when data shouldn't change improves both safety and performance.

## Related Topics

- Lists (mutable alternative)
- namedtuple
- Unpacking and Packing
- Immutability
- Dictionary Keys
- zip, enumerate
- Collections Module
