# Lists - list() constructor, indexing, methods, comprehensions

## Introduction

Lists are ordered, mutable collections that can store items of different data types. They are one of Python's most versatile and commonly used data structures. Lists are created with square brackets `[]`, support indexing, slicing, and numerous built-in methods for manipulation. Lists can contain any type of object, including other lists (nested lists).

## Why It Is Important

Lists are fundamental because:
- They store and organize collections of data
- They are mutable (can be changed in place)
- They support heterogeneous items (different types)
- They provide extensive methods for manipulation
- They are essential for iteration and data processing
- They form the basis for many algorithms and data structures

## Syntax

```python
# Creating a list
empty_list = []
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True, None]

# List with list() constructor
chars = list("hello")  # ['h', 'e', 'l', 'l', 'o']

# List comprehension
squares = [x**2 for x in range(10)]

# Accessing elements
first = numbers[0]
last = numbers[-1]

# Slicing
subset = numbers[1:4]

# Modifying
numbers[0] = 10
numbers.append(6)
```

## Examples

### Creating Lists

```python
# Different ways to create lists
empty = []
numbers = [1, 2, 3, 4, 5]
mixed = [1, "apple", 3.14, True, [5, 6]]
print(f"Empty: {empty}")
print(f"Numbers: {numbers}")
print(f"Mixed types: {mixed}")

# Using list() constructor
from_range = list(range(5))  # [0, 1, 2, 3, 4]
from_string = list("hello")  # ['h', 'e', 'l', 'l', 'o']
from_tuple = list((1, 2, 3))  # [1, 2, 3]
print(f"From range: {from_range}")
print(f"From string: {from_string}")

# List with repeated items
zeros = [0] * 5  # [0, 0, 0, 0, 0]
print(f"Repeated: {zeros}")

# Nested lists
nested = [[1, 2], [3, 4], [5, 6]]
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
print(f"Matrix: {matrix}")
print(f"Element [1][1]: {matrix[1][1]}")  # 5
```

### Indexing and Slicing

```python
fruits = ["apple", "banana", "cherry", "date", "elderberry"]
print(f"Fruits: {fruits}")

# Indexing
print(f"First: {fruits[0]}")       # apple
print(f"Last: {fruits[-1]}")       # elderberry
print(f"Second: {fruits[1]}")      # banana
print(f"Second to last: {fruits[-2]}")  # date

# Slicing [start:stop:step]
print(f"fruits[0:3]: {fruits[0:3]}")       # ['apple', 'banana', 'cherry']
print(f"fruits[:3]: {fruits[:3]}")         # ['apple', 'banana', 'cherry']
print(f"fruits[2:]: {fruits[2:]}")         # ['cherry', 'date', 'elderberry']
print(f"fruits[::2]: {fruits[::2]}")       # ['apple', 'cherry', 'elderberry']
print(f"fruits[::-1]: {fruits[::-1]}")     # Reversed

# Modifying via slice
numbers = [1, 2, 3, 4, 5]
numbers[1:3] = [20, 30]  # Replace elements
print(f"After slice assignment: {numbers}")

numbers[1:3] = [100]  # Replace with fewer elements
print(f"After replacement: {numbers}")

numbers[1:1] = [50, 60]  # Insert at position (empty slice)
print(f"After insertion: {numbers}")
```

## Beginner Examples

```python
# Basic list operations
numbers = []

# Adding elements
numbers.append(10)
numbers.append(20)
numbers.append(30)
print(f"After append: {numbers}")

# Insert at specific position
numbers.insert(1, 15)
print(f"After insert: {numbers}")

# Extending with another list
numbers.extend([40, 50])
print(f"After extend: {numbers}")

# Removing elements
numbers.remove(30)  # Remove by value
print(f"After remove: {numbers}")

popped = numbers.pop()  # Remove and return last
print(f"Popped: {popped}, Remaining: {numbers}")

popped_at = numbers.pop(1)  # Remove at index
print(f"Popped at index 1: {popped_at}, Remaining: {numbers}")

# Checking membership
print(f"20 in numbers: {20 in numbers}")
print(f"99 not in numbers: {99 not in numbers}")

# List length
print(f"Length: {len(numbers)}")

# Iteration
print("Iterating:")
for num in numbers:
    print(f"  {num}")

# Finding index
colors = ["red", "green", "blue", "green"]
print(f"Index of 'blue': {colors.index('blue')}")
print(f"Count of 'green': {colors.count('green')}")
```

## Intermediate Examples

```python
# Sorting
numbers = [3, 1, 4, 1, 5, 9, 2, 6]

# In-place sorting
numbers.sort()
print(f"Ascending: {numbers}")

numbers.sort(reverse=True)
print(f"Descending: {numbers}")

# Sorted function (returns new list)
numbers = [3, 1, 4, 1, 5, 9]
sorted_asc = sorted(numbers)
sorted_desc = sorted(numbers, reverse=True)
print(f"Original: {numbers}")
print(f"Sorted asc: {sorted_asc}")
print(f"Sorted desc: {sorted_desc}")

# Custom sort with key
words = ["banana", "apple", "cherry", "date", "elderberry"]
words.sort(key=len)
print(f"Sorted by length: {words}")

# Sort by last character
words.sort(key=lambda x: x[-1])
print(f"Sorted by last char: {words}")

# Reversing
numbers = [1, 2, 3, 4, 5]
numbers.reverse()
print(f"Reversed: {numbers}")

# Copying lists (important!)
original = [1, 2, 3]

# Shallow copy methods
copy1 = original.copy()
copy2 = original[:]
copy3 = list(original)

# Nested list copy
nested = [[1, 2], [3, 4]]
shallow = nested.copy()
shallow[0][0] = 99  # Also modifies nested!
print(f"Nested: {nested}")
print(f"Shallow: {shallow}")

# Deep copy
import copy
deep = copy.deepcopy(nested)
deep[0][0] = 100
print(f"Nested after deep copy change: {nested}")
print(f"Deep: {deep}")

# List operations
list1 = [1, 2, 3]
list2 = [4, 5, 6]
combined = list1 + list2
print(f"Concatenation: {combined}")
print(f"Repetition: {list1 * 3}")
```

## Advanced Examples

```python
# List comprehensions
numbers = [1, 2, 3, 4, 5]
squares = [n ** 2 for n in numbers]
print(f"Squares: {squares}")

# Conditional comprehensions
even_squares = [n ** 2 for n in numbers if n % 2 == 0]
print(f"Even squares: {even_squares}")

# Nested list comprehensions
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flattened = [num for row in matrix for num in row]
print(f"Flattened matrix: {flattened}")

# Using lists as stacks (LIFO)
stack = []
stack.append(1)
stack.append(2)
stack.append(3)
print(f"\nStack: {stack}")
print(f"Pop: {stack.pop()}")
print(f"Pop: {stack.pop()}")
print(f"Stack after pops: {stack}")

# Using lists as queues (FIFO) - use collections.deque for efficiency
from collections import deque
queue = deque(["Alice", "Bob", "Charlie"])
queue.append("Diana")
print(f"\nQueue: {list(queue)}")
print(f"Popleft: {queue.popleft()}")
print(f"Queue after: {list(queue)}")

# List as array (for homogeneous numeric data)
import array
arr = array.array('i', [1, 2, 3, 4, 5])
print(f"\nArray: {arr}")

# List mapping and filtering
def double(x):
    return x * 2

numbers = [1, 2, 3, 4, 5]
doubled = list(map(double, numbers))
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(f"Mapped: {doubled}")
print(f"Filtered: {evens}")

# Zipping and unzipping lists
names = ["Alice", "Bob", "Charlie"]
scores = [85, 92, 78]
combined = list(zip(names, scores))
print(f"\nZipped: {combined}")

names_back, scores_back = zip(*combined)
print(f"Unzipped names: {names_back}")
print(f"Unzipped scores: {scores_back}")

# Partitioning a list
def partition(lst, predicate):
    """Split list into two lists based on predicate."""
    true_items = []
    false_items = []
    for item in lst:
        if predicate(item):
            true_items.append(item)
        else:
            false_items.append(item)
    return true_items, false_items

numbers = [1, 2, 3, 4, 5, 6]
evens, odds = partition(numbers, lambda x: x % 2 == 0)
print(f"\nEvens: {evens}, Odds: {odds}")

# Chunking a list
def chunk_list(lst, chunk_size):
    """Split a list into chunks of specified size."""
    return [lst[i:i + chunk_size] for i in range(0, len(lst), chunk_size)]

big_list = list(range(12))
chunks = chunk_list(big_list, 4)
print(f"Chunks: {chunks}")
```

## Real-World Use Cases

- **Data Storage**: Storing records, rows from databases
- **Queue/Stack Implementation**: Task queues, undo history
- **Matrix Operations**: 2D grids for games, image processing
- **Dynamic Collections**: Items that grow/shrink during runtime
- **Sorting and Searching**: Organizing and finding data
- **Buffer Management**: Temporary storage for data processing
- **Configuration**: List of settings, allowed values

## Common Mistakes

```python
# Mistake 1: Modifying list while iterating
numbers = [1, 2, 3, 4, 5, 6]
# Wrong:
for num in numbers:
    if num % 2 == 0:
        numbers.remove(num)  # Skips elements!
print(numbers)  # Incorrect result

# Correct:
numbers = [1, 2, 3, 4, 5, 6]
numbers[:] = [num for num in numbers if num % 2 != 0]
print(numbers)  # [1, 3, 5]

# Mistake 2: Confusing append and extend
a = [1, 2, 3]
a.append([4, 5])  # Adds as single element
print(a)  # [1, 2, 3, [4, 5]]

a = [1, 2, 3]
a.extend([4, 5])  # Adds each element
print(a)  # [1, 2, 3, 4, 5]

# Mistake 3: Using = instead of .copy()
a = [1, 2, 3]
b = a  # Both reference the same list
b.append(4)
print(a)  # [1, 2, 3, 4] (a was modified too!)

# Correct:
a = [1, 2, 3]
b = a.copy()
b.append(4)
print(a)  # [1, 2, 3]

# Mistake 4: Index out of range
# empty = []
# empty[0] = 10  # IndexError

# Mistake 5: Forgetting that list.sort() returns None
numbers = [3, 1, 2]
result = numbers.sort()  # None! (in-place)
print(result)  # None

# Use sorted() instead:
numbers = [3, 1, 2]
sorted_nums = sorted(numbers)
print(sorted_nums)  # [1, 2, 3]

# Mistake 6: Creating a list of mutable objects incorrectly
# Wrong:
matrix = [[0] * 3] * 3  # Creates references to same list!
matrix[0][0] = 1
print(matrix)  # [[1, 0, 0], [1, 0, 0], [1, 0, 0]] (all rows changed!)

# Correct:
matrix = [[0] * 3 for _ in range(3)]
matrix[0][0] = 1
print(matrix)  # [[1, 0, 0], [0, 0, 0], [0, 0, 0]]
```

## Best Practices

- Use list comprehensions for simple transformations (more readable and faster)
- Use `join()` for converting lists to strings
- Use `copy()` or `[:]` to create independent copies
- Use `deepcopy()` for nested mutable structures
- Prefer `extend()` over multiple `append()` calls in loops
- Use `collections.deque` for queue operations (faster than list)
- Avoid modifying lists while iterating over them
- Use `enumerate()` when you need both index and value
- Use `zip()` to iterate over multiple lists in parallel
- Use `any()` / `all()` for boolean checks on list elements
- Use `sorted()` instead of `.sort()` when you need the original list
- Consider `array.array` for homogeneous numeric data

## Interview Questions

1. What is the difference between `append()` and `extend()`?
2. How do you reverse a list in Python?
3. What's the difference between `list.sort()` and `sorted()`?
4. How do you create a shallow copy vs a deep copy of a list?
5. Explain list slicing with the `[start:stop:step]` syntax.
6. How do you remove duplicates from a list while preserving order?
7. What is the time complexity of `append()`, `insert()`, and `pop()`?
8. How do you flatten a nested list?
9. Explain list comprehensions with a conditional expression.
10. How do you find the most common element in a list?

## Coding Challenges

1. Write a function that removes duplicates from a list while preserving order.
2. Create a function that merges two sorted lists into a single sorted list.
3. Write a function that rotates a list by k positions.
4. Create a function that finds all pairs in a list that sum to a target.
5. Write a function that transposes a matrix (list of lists).

```python
# Challenge 1: Remove duplicates preserving order
def remove_duplicates(lst):
    seen = set()
    result = []
    for item in lst:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result

items = [3, 1, 2, 1, 3, 4, 2, 5]
print(f"Original: {items}")
print(f"No duplicates: {remove_duplicates(items)}")

# Challenge 2: Merge sorted lists
def merge_sorted(list1, list2):
    result = []
    i = j = 0
    while i < len(list1) and j < len(list2):
        if list1[i] <= list2[j]:
            result.append(list1[i])
            i += 1
        else:
            result.append(list2[j])
            j += 1
    result.extend(list1[i:])
    result.extend(list2[j:])
    return result

a = [1, 3, 5, 7]
b = [2, 4, 6, 8]
print(f"Merged: {merge_sorted(a, b)}")

# Challenge 3: Rotate list by k positions
def rotate(lst, k):
    if not lst:
        return lst
    k = k % len(lst)
    return lst[-k:] + lst[:-k]

nums = [1, 2, 3, 4, 5]
print(f"Rotated by 2: {rotate(nums, 2)}")   # [4, 5, 1, 2, 3]
print(f"Rotated by -1: {rotate(nums, -1)}") # [2, 3, 4, 5, 1]

# Challenge 4: Find pairs that sum to target
def find_pairs(nums, target):
    seen = set()
    pairs = []
    for num in nums:
        complement = target - num
        if complement in seen:
            pairs.append((complement, num))
        seen.add(num)
    return pairs

nums = [2, 4, 3, 7, 5, 1, 6]
print(f"Pairs summing to 8: {find_pairs(nums, 8)}")

# Challenge 5: Matrix Transpose
def transpose(matrix):
    rows = len(matrix)
    cols = len(matrix[0])
    return [[matrix[i][j] for i in range(rows)] for j in range(cols)]

matrix = [
    [1, 2, 3],
    [4, 5, 6],
]
print("Original:")
for row in matrix:
    print(row)
print("Transposed:")
for row in transpose(matrix):
    print(row)
```

## Summary

Lists are versatile, mutable sequences that can store heterogeneous items. They support indexing, slicing, and a comprehensive set of methods for adding, removing, searching, and sorting elements. List comprehensions provide a concise way to create and transform lists. Understanding when and how to use lists, as well as their performance characteristics, is essential for effective Python programming.

## Related Topics

- Tuples (immutable alternative)
- List Comprehensions
- Iterators and Iterables
- Collections Module (deque, defaultdict, Counter)
- Slicing
- Mutability and References
- Sorting (sorted, sort, key functions)
