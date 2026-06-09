# Sets - set() constructor, union, intersection, difference

## Introduction

Sets are unordered collections of unique, hashable elements. They are mutable (can add/remove elements) but cannot contain mutable elements (like lists or dicts). Sets are optimized for membership testing, deduplication, and mathematical set operations like union, intersection, and difference. Python also provides `frozenset`, an immutable version of set.

## Why It Is Important

Sets are important because:
- They provide O(1) average-time membership testing
- They automatically eliminate duplicates
- They support mathematical set operations efficiently
- They are useful for data cleaning and validation
- They enable fast intersection/union operations on collections

## Syntax

```python
# Creating sets
empty_set = set()  # {} creates an empty dict, not set!
numbers = {1, 2, 3, 4, 5}
mixed = {1, "hello", (1, 2)}  # Hashable elements only

# From other iterables
from_list = set([1, 2, 2, 3])  # {1, 2, 3}
from_string = set("hello")  # {'h', 'e', 'l', 'o'}

# Set comprehension
squares = {x**2 for x in range(5)}  # {0, 1, 4, 9, 16}

# Frozenset (immutable)
fs = frozenset([1, 2, 3])
```

## Examples

### Creating Sets

```python
# Different creation methods
empty = set()  # Must use set(), not {}
print(f"Empty set: {empty}, type: {type(empty)}")

# Literal syntax
numbers = {1, 2, 3, 4, 5}
print(f"Numbers: {numbers}")

# Eliminating duplicates
duplicates = {1, 2, 2, 3, 3, 3, 4}
print(f"Duplicates removed: {duplicates}")

# From various iterables
from_list = set([3, 1, 4, 1, 5, 9])
from_string = set("mississippi")
from_range = set(range(5))
print(f"From list: {from_list}")
print(f"From string 'mississippi': {from_string}")
print(f"From range: {from_range}")

# Mixed types (all must be hashable)
mixed = {1, "hello", (1, 2), 3.14}
print(f"Mixed set: {mixed}")

# Frozenset
fs = frozenset([1, 2, 3, 3, 4])
print(f"Frozenset: {fs}")
print(f"Frozenset is hashable: {hash(fs)}")
```

### Basic Set Operations

```python
# Adding elements
fruits = {"apple", "banana"}
fruits.add("cherry")
print(f"After add: {fruits}")

# Removing elements
fruits.discard("banana")  # No error if missing
print(f"After discard: {fruits}")
fruits.remove("apple")  # KeyError if missing
# fruits.remove("grape")  # KeyError!

# Pop (remove and return arbitrary element)
numbers = {1, 2, 3}
popped = numbers.pop()
print(f"Popped: {popped}, Remaining: {numbers}")

# Clear
numbers.clear()
print(f"After clear: {numbers}")

# Membership testing (fast, O(1))
fruits = {"apple", "banana", "cherry"}
print(f"'banana' in fruits: {'banana' in fruits}")  # True
print(f"'grape' not in fruits: {'grape' not in fruits}")  # True

# Length
print(f"Number of fruits: {len(fruits)}")

# Iteration
for fruit in fruits:
    print(f"  {fruit}")
```

## Beginner Examples

```python
# Removing duplicates from a list
numbers = [1, 2, 3, 2, 4, 3, 5, 1, 6]
unique = list(set(numbers))
print(f"Original: {numbers}")
print(f"Unique: {unique}")

# But note: set() does not preserve order
# To preserve order while removing duplicates:
def unique_ordered(items):
    seen = set()
    result = []
    for item in items:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result

numbers = [3, 1, 2, 1, 3, 4, 3, 5]
print(f"Unique ordered: {unique_ordered(numbers)}")

# Finding common elements between two lists
list_a = [1, 2, 3, 4, 5]
list_b = [4, 5, 6, 7, 8]
common = set(list_a) & set(list_b)
print(f"Common elements: {common}")

# Finding unique elements
only_in_a = set(list_a) - set(list_b)
only_in_b = set(list_b) - set(list_a)
print(f"Only in A: {only_in_a}")
print(f"Only in B: {only_in_b}")

# Checking if all items are unique
def all_unique(items):
    return len(items) == len(set(items))

print(f"[1,2,3] all unique: {all_unique([1, 2, 3])}")  # True
print(f"[1,2,2] all unique: {all_unique([1, 2, 2])}")  # False

# Counting vowels
def count_vowels(text):
    vowels = set("aeiouAEIOU")
    return sum(1 for char in text if char in vowels)

print(f"Vowels in 'Hello World': {count_vowels('Hello World')}")
```

## Intermediate Examples

```python
# Mathematical set operations
a = {1, 2, 3, 4, 5}
b = {4, 5, 6, 7, 8}

print(f"A: {a}")
print(f"B: {b}")

# Union | : elements in A or B
print(f"Union (A | B): {a | b}")           # {1, 2, 3, 4, 5, 6, 7, 8}
print(f"Union (A.union(B)): {a.union(b)}")

# Intersection & : elements in both A and B
print(f"Intersection (A & B): {a & b}")    # {4, 5}
print(f"Intersection (A.intersection(B)): {a.intersection(b)}")

# Difference - : elements in A but not B
print(f"Difference (A - B): {a - b}")      # {1, 2, 3}
print(f"Difference (B - A): {b - a}")      # {6, 7, 8}
print(f"Difference (A.difference(b)): {a.difference(b)}")

# Symmetric difference ^ : elements in A or B but not both
print(f"Symmetric diff (A ^ B): {a ^ b}")  # {1, 2, 3, 6, 7, 8}
print(f"Symmetric diff: {a.symmetric_difference(b)}")

# Subset / Superset
x = {1, 2, 3}
y = {1, 2, 3, 4, 5}
print(f"X: {x}, Y: {y}")
print(f"X is subset of Y: {x.issubset(y)}")       # True
print(f"Y is superset of X: {y.issuperset(x)}")    # True
print(f"X is proper subset of Y: {x < y}")         # True (strict)

# Disjoint sets (no common elements)
c = {10, 20, 30}
print(f"X and C are disjoint: {x.isdisjoint(c)}")  # True

# Update operations (modify in-place)
a = {1, 2, 3}
b = {3, 4, 5}
a.update(b)  # Add all elements from b to a
print(f"After update: {a}")  # {1, 2, 3, 4, 5}

a.intersection_update({1, 2, 3, 6})
print(f"After intersection_update: {a}")  # {1, 2, 3}

a.difference_update({1})
print(f"After difference_update: {a}")  # {2, 3}

a.symmetric_difference_update({3, 4, 5})
print(f"After symmetric_difference_update: {a}")  # {2, 4, 5}
```

## Advanced Examples

```python
# Set comprehensions
squares = {x**2 for x in range(10)}
print(f"Squares: {squares}")

# Conditional set comprehension
evens = {x for x in range(20) if x % 2 == 0}
print(f"Even numbers: {evens}")

# Set comprehension with multiple loops
pairs = {(a, b) for a in range(3) for b in range(3) if a != b}
print(f"Pairs: {pairs}")

# Using sets for graph algorithms (adjacency)
graph = {
    "A": {"B", "C"},
    "B": {"A", "D", "E"},
    "C": {"A", "F"},
    "D": {"B"},
    "E": {"B", "F"},
    "F": {"C", "E"}
}

def bfs_shortest_path(graph, start, goal):
    """Find shortest path using sets for visited tracking."""
    if start == goal:
        return [start]
    visited = {start}
    queue = [[start]]
    while queue:
        path = queue.pop(0)
        node = path[-1]
        for neighbor in graph[node] - visited:
            if neighbor == goal:
                return path + [neighbor]
            visited.add(neighbor)
            queue.append(path + [neighbor])
    return None

print(f"Path A to F: {bfs_shortest_path(graph, 'A', 'F')}")

# Using frozenset as dictionary keys
# Sets can't be dict keys, but frozensets can
teams = {
    frozenset({"Alice", "Bob"}): "Team Alpha",
    frozenset({"Charlie", "Diana"}): "Team Beta"
}
team_members = frozenset({"Alice", "Bob"})
print(f"Team: {teams[team_members]}")

# Finding unique elements across multiple lists using sets
def unique_across(*lists):
    """Return elements that appear in exactly one list."""
    all_elements = set()
    duplicates = set()
    for lst in lists:
        for item in lst:
            if item in all_elements:
                duplicates.add(item)
            all_elements.add(item)
    return all_elements - duplicates

list1 = [1, 2, 3, 4]
list2 = [3, 4, 5, 6]
list3 = [5, 6, 7, 8]
print(f"Unique across lists: {unique_across(list1, list2, list3)}")

# Anagram groups using frozenset of character counts
from collections import Counter
def anagram_groups(words):
    groups = {}
    for word in words:
        # Use frozenset of (char, count) as key
        signature = frozenset(Counter(word).items())
        groups.setdefault(signature, []).append(word)
    return list(groups.values())

words = ["eat", "tea", "tan", "ate", "nat", "bat"]
print(f"Anagram groups: {anagram_groups(words)}")
```

## Real-World Use Cases

- **Data Deduplication**: Removing duplicates from large datasets
- **Membership Testing**: Checking if items exist in a collection
- **Access Control**: Tracking user permissions and roles
- **Graph Algorithms**: Tracking visited nodes in BFS/DFS
- **Data Analysis**: Finding common/unique elements across datasets
- **Database Operations**: Implementing set-based SQL operations
- **Spam Filtering**: Maintaining sets of known spam addresses

## Common Mistakes

```python
# Mistake 1: Using {} to create empty set
empty = {}  # This is a dict, not a set!
print(type(empty))  # <class 'dict'>

# Correct:
empty = set()
print(type(empty))  # <class 'set'>

# Mistake 2: Adding mutable elements
# s = set()
# s.add([1, 2])  # TypeError: unhashable type: 'list'
# Use tuple instead:
s.add((1, 2))

# Mistake 3: Assuming sets are ordered
s = {3, 1, 2}
print(s)  # May not be {1, 2, 3} - sets are unordered!

# Mistake 4: Forgetting that set operations return new sets
a = {1, 2, 3}
b = {3, 4, 5}
result = a.union(b)  # Returns new set, does NOT modify a
print(a)  # Still {1, 2, 3}

# For in-place modification:
a.update(b)
print(a)  # {1, 2, 3, 4, 5}

# Mistake 5: Using set on list of lists
# set([[1, 2], [3, 4]])  # TypeError
# Correct:
set_of_tuples = {(1, 2), (3, 4)}

# Mistake 6: Set.pop() removes arbitrary element
# Don't rely on order when using pop()
```

## Best Practices

- Use `set()` for creating empty sets
- Use sets for O(1) membership testing instead of lists
- Use `frozenset` for hashable, immutable sets
- Prefer set operators (`|`, `&`, `-`, `^`) over method calls
- Use set comprehensions for clarity
- Use `.discard()` instead of `.remove()` when unsure about existence
- Convert to list with `list()` if you need ordered output
- Use `isdisjoint()` instead of `len(a & b) == 0`
- Use `<=` and `>=` for subset/superset checks
- Use `all_unique = len(items) == len(set(items))` for dup check

## Interview Questions

1. What is the difference between set and list?
2. How does membership testing differ between sets and lists?
3. What is a frozenset and when would you use it?
4. Explain the different set operations (union, intersection, etc.)
5. What are the time complexities of set operations?
6. Can a set contain a set? How can you achieve nested sets?
7. How do you remove duplicates from a list while preserving order?
8. What is a set comprehension?
9. How are sets implemented in Python (internally)?
10. When should you use a set instead of a list?

## Coding Challenges

```python
# Challenge 1: Find duplicates in a list
def find_duplicates(items):
    seen = set()
    duplicates = set()
    for item in items:
        if item in seen:
            duplicates.add(item)
        else:
            seen.add(item)
    return list(duplicates)

print(find_duplicates([1, 2, 3, 2, 4, 3, 5]))  # [2, 3]

# Challenge 2: Check if two strings are anagrams
def are_anagrams(s1, s2):
    return set(s1.lower()) == set(s2.lower()) and len(s1) == len(s2)

print(are_anagrams("listen", "silent"))  # True
print(are_anagrams("hello", "world"))    # False

# Challenge 3: Jaccard similarity between sets
def jaccard_similarity(a, b):
    if not a and not b:
        return 1.0
    return len(a & b) / len(a | b)

a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
print(f"Jaccard similarity: {jaccard_similarity(a, b):.2f}")

# Challenge 4: Find symmetric difference of three sets
def symmetric_diff_three(a, b, c):
    return (a ^ b) ^ c

a = {1, 2, 3}
b = {2, 3, 4}
c = {3, 4, 5}
print(f"Symmetric diff of three: {symmetric_diff_three(a, b, c)}")

# Challenge 5: Set-based spell checker
def spell_check(word, dictionary):
    """Find words in dictionary that are 1 edit away."""
    candidates = set()
    # Deletes
    for i in range(len(word)):
        candidates.add(word[:i] + word[i+1:])
    # Replaces
    for i in range(len(word)):
        for c in "abcdefghijklmnopqrstuvwxyz":
            candidates.add(word[:i] + c + word[i+1:])
    # Inserts
    for i in range(len(word) + 1):
        for c in "abcdefghijklmnopqrstuvwxyz":
            candidates.add(word[:i] + c + word[i:])
    return candidates & dictionary

dictionary = {"hello", "world", "help", "held", "helm", "hell"}
print(f"Suggestions for 'helo': {spell_check('helo', dictionary)}")
```

## Summary

Sets are unordered collections of unique, hashable elements with fast membership testing and mathematical set operations. They support union, intersection, difference, and symmetric difference operations. Frozensets provide immutable, hashable sets. Sets excel at deduplication, membership checking, and set-based algorithms. Using them appropriately can significantly improve code clarity and performance.

## Related Topics

- Dictionaries (set-like keys)
- Hashable Types
- Frozenset
- Set Comprehensions
- Collections Module
- Graph Algorithms
- Boolean Algebra
