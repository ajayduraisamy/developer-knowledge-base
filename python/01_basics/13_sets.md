# Sets - set() constructor, union, intersection, difference

## Introduction

Sets are unordered collections of unique, hashable elements. They are mutable (elements can be added or removed) but cannot contain mutable elements (lists, dicts, or other sets). Sets are optimized for O(1) average-time membership testing, deduplication, and mathematical set operations such as union, intersection, difference, and symmetric difference. Python also provides `frozenset`, an immutable, hashable variant. Sets are fundamental to algorithms that require fast membership queries, uniqueness constraints, and set algebra.

## set() constructor

### What It Is

The `set()` constructor creates a set from any iterable — lists, strings, tuples, ranges, generators, or other iterables. With no argument, it returns an empty set. Note that `{}` creates an empty dictionary, not a set.

### Why It Is Important

The constructor is the primary mechanism for creating sets programmatically. It enables deduplication, type conversion for hashability checks, and transformation of sequence data into an efficiently queryable form.

### How It Works Internally

CPython implements sets as hash tables very similar to dictionaries — a `PySetObject` uses a sparse array of `setentry` structs with open addressing and quadratic probing. When `set()` is called on an iterable, Python iterates the input and inserts each element using the hash function, skipping duplicates. The table resizes when the load factor exceeds 2/3. Unlike dicts, set entries store only keys (no associated values), making them slightly more memory-efficient than dicts with dummy values.

### Syntax

```python
# Empty set
s = set()

# From iterable
s = set([1, 2, 3])
s = set("hello")    # {'h', 'e', 'l', 'o'}
s = set(range(5))
s = set(x**2 for x in range(3))

# Note: {} is empty dict, not set
empty_set = set()      # Correct
empty_dict = {}        # Not a set!
```

### Beginner Examples

```python
# Converting list to set (deduplication)
numbers = [1, 2, 2, 3, 3, 3, 4]
unique = set(numbers)
print(unique)  # {1, 2, 3, 4}

# From string
chars = set("mississippi")
print(chars)  # {'m', 'i', 's', 'p'} (order may vary)

# From range
evens = set(range(0, 10, 2))
print(evens)  # {0, 2, 4, 6, 8}

# Empty set
s = set()
s.add("first")
print(s)  # {'first'}

# From generator
s = set(x**2 for x in range(5))
print(s)  # {0, 1, 4, 9, 16}
```

### Intermediate Examples

```python
# Set from dict keys/values
d = {"a": 1, "b": 2, "c": 3}
keys_set = set(d.keys())
values_set = set(d.values())
print(keys_set)    # {'a', 'b', 'c'}

# Set comprehension vs set()
comp = {x for x in range(10) if x % 2 == 0}
func = set(x for x in range(10) if x % 2 == 0)
print(comp == func)  # True

# Nested iteration to set
matrix = [[1, 2], [2, 3], [3, 4]]
all_values = set(item for row in matrix for item in row)
print(all_values)  # {1, 2, 3, 4}

# Removing duplicates while preserving order (using set for tracking)
def unique_ordered(items):
    seen = set()
    result = []
    for item in items:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result

print(unique_ordered([3, 1, 2, 1, 3, 4]))  # [3, 1, 2, 4]
```

### Advanced Examples

```python
# Frozen set from set
s = set([1, 2, 3])
fs = frozenset(s)
print(hash(fs))  # Hashable

# Set of frozensets (nested sets workaround)
set_of_sets = {frozenset([1, 2]), frozenset([3, 4])}
print(set_of_sets)

# Memory-efficient duplicate detection with set
import sys

def has_duplicates(items):
    seen = set()
    for item in items:
        if item in seen:
            return True
        seen.add(item)
    return False

# Early exit on duplicate
print(has_duplicates([1, 2, 3, 4, 2]))  # True

# Building sets from complex objects
class User:
    def __init__(self, uid, name):
        self.uid = uid
        self.name = name
    def __hash__(self):
        return hash(self.uid)
    def __eq__(self, other):
        return isinstance(other, User) and self.uid == other.uid

users = {User(1, "Alice"), User(2, "Bob"), User(1, "Alicia")}
print(len(users))  # 2 (uid 1 deduplicated)
```

### Real-World Use Cases

- **Data deduplication**: Removing duplicate rows, IDs, or emails
- **Membership testing**: Checking if a user is in an allowed set
- **Tag/flag systems**: Tracking unique tags across documents
- **Crawling**: Tracking visited URLs in web scrapers
- **Access control**: Maintaining sets of authorized tokens
- **Analytics**: Finding unique visitors across time periods

### Common Mistakes

```python
# Mistake 1: Using {} for empty set
s = {}  # This is a dict!
print(type(s))  # <class 'dict'>
# Correct:
s = set()

# Mistake 2: Adding unhashable types
s = set()
# s.add([1, 2])  # TypeError: unhashable type: 'list'
s.add((1, 2))  # Correct (tuple is hashable)

# Mistake 3: Assuming set() preserves order
s = set([3, 1, 2])
print(s)  # Likely {1, 2, 3} but NOT guaranteed!

# Mistake 4: set() on list of lists
# set([[1, 2], [3, 4]])  # TypeError
set_of_tuples = {(1, 2), (3, 4)}  # Correct

# Mistake 5: Forgetting set() is O(n) with hash collisions
# In worst-case (all hashes collide), set() becomes O(n^2)
```

### Best Practices

- Use `set()` for empty sets (never `{}`)
- Use set comprehensions for clarity when transforming elements
- Use `frozenset()` when hashability is required
- Prefer `set(iterable)` over manual `for` loops with `.add()`
- Be explicit about hash/eq for custom objects in sets

### Performance Considerations

`set()` construction is O(n) average, O(n^2) worst-case (hash collisions). Sets consume ~72 bytes + 8 bytes per entry (CPython). For large datasets, pre-allocating capacity is not directly supported; Python resizes automatically. Repeated `add()` calls are amortized O(1). Sets are significantly faster than lists for membership testing (O(1) vs O(n)).

### Interview Questions

1. Why does `set()` exist when `{}` creates dicts?
2. What happens when you pass a generator to `set()`?
3. How does CPython implement sets internally?
4. What is the time complexity of `set(iterable)`?
5. Can you create a set of sets? How?
6. How does `set()` interact with user-defined hash functions?
7. What is the difference between `set()` and `frozenset()`?

### Coding Challenges

```python
# Challenge 1: Find duplicates using set
def find_duplicates(items):
    seen = set()
    duplicates = set()
    for item in items:
        if item in seen:
            duplicates.add(item)
        else:
            seen.add(item)
    return duplicates

print(find_duplicates([1, 2, 3, 2, 4, 3, 5]))  # {2, 3}

# Challenge 2: Unique elements across multiple lists
def unique_elements(*lists):
    all_set = set()
    duplicate_set = set()
    for lst in lists:
        for item in lst:
            if item in all_set:
                duplicate_set.add(item)
            all_set.add(item)
    return all_set - duplicate_set

print(unique_elements([1, 2], [2, 3], [3, 4]))  # {1, 4}

# Challenge 3: Set-based Jaccard similarity
def jaccard(a, b):
    if not a and not b:
        return 1.0
    return len(a & b) / len(a | b)

a = set([1, 2, 3, 4])
b = set([3, 4, 5, 6])
print(f"{jaccard(a, b):.2f}")  # 0.33
```

### Related Topics

- Hashable types and __hash__
- frozenset
- set comprehensions
- CPython hash table implementation
- dict keys (similar hash table)

## Union

### What It Is

Union combines elements from two or more sets, returning a new set containing all unique elements from all input sets. In Python, union is performed with the `|` operator or the `.union()` method.

### Why It Is Important

Union is fundamental to set algebra and appears in data merging, tag aggregation, permission combination, and any scenario where you need the complete set of unique items across multiple collections.

### How It Works Internally

The `|` operator calls `PySet_Union()` internally. CPython creates a new empty set, then copies all elements from the first set, then iterates the second set and inserts elements not already present. The `|` operator is more efficient than `a.union(b)` when both operands are sets because it can optimize capacity pre-allocation based on the sum of sizes.

### Syntax

```python
a = {1, 2, 3}
b = {3, 4, 5}

# Operator
result = a | b

# Method
result = a.union(b)

# Multiple sets
result = a | b | {7, 8}
result = a.union(b, {6, 7})

# In-place update
a |= b  # a becomes a | b
a.update(b)  # Same as |=
```

### Beginner Examples

```python
a = {1, 2, 3}
b = {3, 4, 5}

union_op = a | b
union_method = a.union(b)
print(union_op)      # {1, 2, 3, 4, 5}
print(union_method)  # {1, 2, 3, 4, 5}

# Strings as sets
a = set("abc")
b = set("bcd")
print(a | b)  # {'a', 'b', 'c', 'd'}

# Empty set union
empty = set()
print(empty | {1, 2})  # {1, 2}

# Union of identical sets
a = {1, 2, 3}
print(a | a)  # {1, 2, 3} (no duplicates)
```

### Intermediate Examples

```python
# Multiple set union
a = {1, 2}
b = {3, 4}
c = {5, 6}
result = a | b | c
print(result)  # {1, 2, 3, 4, 5, 6}

# Union with generators
def generate_numbers():
    yield from range(3)

s = {10, 20}.union(generate_numbers())
print(s)  # {0, 1, 2, 10, 20}

# Union for tag aggregation
user1_tags = {"python", "data-science", "ml"}
user2_tags = {"python", "web-dev", "devops"}
all_tags = user1_tags | user2_tags
print(all_tags)

# Performance comparison
import time
a = set(range(100000))
b = set(range(50000, 150000))

start = time.perf_counter()
c = a | b
print(f"Operator: {time.perf_counter() - start:.4f}s")

start = time.perf_counter()
c = a.union(b)
print(f"Method: {time.perf_counter() - start:.4f}s")
```

### Advanced Examples

```python
# Union of set of frozensets
set_of_frozen = {frozenset([1, 2]), frozenset([3, 4])}
unioned = set()
for fs in set_of_frozen:
    unioned |= fs
print(unioned)  # {1, 2, 3, 4}

# Reducer pattern for union
from functools import reduce
sets = [{1, 2}, {2, 3}, {3, 4}, {4, 5}]
result = reduce(lambda a, b: a | b, sets, set())
print(result)  # {1, 2, 3, 4, 5}

# Tag-based content recommendation
def recommended_content(user_tags, content_tags):
    overlapping = user_tags & content_tags
    total = user_tags | content_tags
    if not total:
        return 0.0
    return len(overlapping) / len(total)

user = {"python", "ml", "data"}
content = {"python", "web", "api"}
print(f"Relevance: {recommended_content(user, content):.2f}")  # 0.20
```

### Real-World Use Cases

- **Permission aggregation**: Combining permissions from multiple roles
- **Tag expansion**: Collecting all tags across a document set
- **User base merge**: Combining user lists from multiple sources
- **Feature sets**: Union of feature sets in ML pipelines
- **Query expansion**: Union of related search terms in information retrieval
- **Access control**: Union of allowed IP ranges

### Common Mistakes

```python
# Mistake 1: Using | with non-set types
# {1, 2} | [3, 4]  # TypeError: unsupported operand type(s) for |: 'set' and 'list'
{1, 2}.union([3, 4])  # Works -- method accepts any iterable

# Mistake 2: Forgetting union creates a new set
a = {1, 2}
b = {2, 3}
c = a | b
print(a)  # {1, 2} -- unchanged
print(c)  # {1, 2, 3} -- new set

# Use |= for in-place:
a |= b
print(a)  # {1, 2, 3}

# Mistake 3: Assuming union preserves original order
# Sets are unordered -- don't rely on element order
```

### Best Practices

- Prefer `|` operator over `.union()` when both operands are sets (more readable)
- Use `.union()` when one argument is a non-set iterable
- Use `|=` for in-place updates
- Use `functools.reduce` for union of many sets
- Chain unions with `|` for readability

### Performance Considerations

`|` is O(len(a) + len(b)) average. CPython optimizes the operation by pre-allocating the result set to the size of a + b. `.union(*many_sets)` is O(total elements across all sets). The `|=` operator modifies in-place and may avoid reallocation if the target set has sufficient capacity.

### Interview Questions

1. What is the difference between `|` and `.union()`?
2. Can `|` work with non-set types?
3. How does Python compute union internally?
4. What is the time complexity of set union?
5. How do you union more than two sets efficiently?
6. Does union mutate the original sets?

### Coding Challenges

```python
# Challenge 1: Union of variable number of sets
def multi_union(*sets):
    result = set()
    for s in sets:
        result |= s
    return result

print(multi_union({1}, {2, 3}, {3, 4, 5}))  # {1, 2, 3, 4, 5}

# Challenge 2: Symmetric aggregation
def symmetric_union(a, b):
    return (a - b) | (b - a)  # Equivalent to a ^ b

print(symmetric_union({1, 2, 3}, {2, 3, 4}))  # {1, 4}

# Challenge 3: Union-based spell checker
def spell_check(word, dictionary):
    letters = set("abcdefghijklmnopqrstuvwxyz")
    candidates = set()
    # One-character edits: union of all possible edits
    for i in range(len(word)):
        candidates |= {word[:i] + c + word[i+1:] for c in letters}
    return candidates & dictionary

dictionary = {"hello", "hallo", "helly", "world"}
print(spell_check("hello", dictionary))
```

### Related Topics

- set intersection (&)
- set difference (-)
- set symmetric difference (^)
- update methods (|=, &=, -=, ^=)
- frozenset operations

## Intersection

### What It Is

Intersection returns a new set containing only elements present in all input sets. In Python, intersection uses the `&` operator or `.intersection()` method.

### Why It Is Important

Intersection is essential for finding commonality — common tags, shared users, overlapping permissions, mutual connections, and any scenario requiring elements that satisfy multiple criteria simultaneously.

### How It Works Internally

The `&` operator calls `PySet_Intersection()`. CPython iterates over the smaller set and checks membership in the other set(s), yielding only elements found in all sets. This "iterate smallest first" strategy minimizes membership tests. The result set is pre-allocated to the size of the smallest input set.

### Syntax

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

# Operator
result = a & b

# Method
result = a.intersection(b)

# Multiple sets
result = a & b & {4, 5}
result = a.intersection(b, {2, 4})

# In-place
a &= b  # a becomes a & b
a.intersection_update(b)  # Same as &=
```

### Beginner Examples

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

common = a & b
print(common)  # {3, 4}

# With strings
a = set("hello")
b = set("world")
print(a & b)  # {'l', 'o'}

# Empty intersection
a = {1, 2}
b = {3, 4}
print(a & b)  # set()

# Single set
print(a & a)  # {1, 2}

# Check disjoint sets
def are_disjoint(a, b):
    return len(a & b) == 0

print(are_disjoint({1, 2}, {3, 4}))  # True
```

### Intermediate Examples

```python
# Three-way intersection
a = {1, 2, 3, 4}
b = {2, 3, 4, 5}
c = {3, 4, 5, 6}
print(a & b & c)  # {3, 4}

# Intersection with non-set iterable
a = {1, 2, 3, 4}
print(a.intersection([3, 4, 5]))  # {3, 4}

# Use case: common friends
alice_friends = {"bob", "charlie", "diana"}
bob_friends = {"alice", "charlie", "eve"}
mutual = alice_friends & bob_friends
print(mutual)  # {'charlie'}

# Intersection as filter
valid_ids = {101, 102, 103, 104}
selected = {102, 104, 106}
valid_selected = valid_ids & selected
print(valid_selected)  # {102, 104}

# Intersection for word analysis
def common_letters(words):
    if not words:
        return set()
    result = set(words[0])
    for word in words[1:]:
        result &= set(word)
    return result

print(common_letters(["hello", "world", "help"]))  # {'l'}
```

### Advanced Examples

```python
# Optimized multi-set intersection
def fast_intersection(*sets):
    if not sets:
        return set()
    smallest = min(sets, key=len)
    return smallest.intersection(*[s for s in sets if s is not smallest])

sets = [set(range(1000)), set(range(500, 1500)), set(range(300, 800))]
result = fast_intersection(*sets)
print(len(result))  # 300

# Intersection of dict key views
d1 = {"a": 1, "b": 2, "c": 3}
d2 = {"b": 4, "c": 5, "d": 6}
common_keys = d1.keys() & d2.keys()
print(common_keys)  # {'b', 'c'}

# Tag-based filtering
class Document:
    def __init__(self, title, tags):
        self.title = title
        self.tags = set(tags)

docs = [
    Document("Py", ["python", "programming"]),
    Document("ML", ["python", "ml", "data"]),
    Document("Web", ["python", "web", "api"]),
]

required = {"python", "ml"}
matching = [d for d in docs if d.tags & required]
print([d.title for d in matching])  # ['Py', 'ML']

# Graph intersection for recommendation
def shared_interests(user1_tags, user2_tags):
    return len(user1_tags & user2_tags) / len(user1_tags | user2_tags)

print(f"Similarity: {shared_interests({'a', 'b', 'c'}, {'b', 'c', 'd'}):.2f}")
```

### Real-World Use Cases

- **Database JOINs**: Finding matching rows across tables
- **Social networks**: Mutual friends or common interests
- **E-commerce**: Product filtering by multiple criteria
- **Access control**: Users with all required permissions
- **Analytics**: Customers who bought multiple products
- **Search**: Results matching all query terms
- **Tags**: Content matching multiple tag filters

### Common Mistakes

```python
# Mistake 1: & with non-set types
# {1, 2} & [2, 3]  # TypeError
{1, 2}.intersection([2, 3])  # Works

# Mistake 2: Confusing intersection with union
a = {1, 2}
b = {2, 3}
print(a & b)  # {2} (not {1, 2, 3}!)

# Mistake 3: Forgetting & creates new set
a = {1, 2, 3}
b = {2, 3, 4}
c = a & b
print(a)  # {1, 2, 3} -- unchanged
a &= b
print(a)  # {2, 3} -- modified

# Mistake 4: Assuming intersection of many sets is O(n*m)
# It's optimized: iterates smallest set first
```

### Best Practices

- Use `&` over `.intersection()` for set-set operations
- Use `.intersection()` when including non-set iterables
- Use `&=` for in-place intersection update
- Prefer `a & b & c` over chained `.intersection()` calls
- Use `min(sets, key=len).intersection(*others)` for many sets
- Use dict view intersection for common-key analysis

### Performance Considerations

`&` is O(min(len(a), len(b))) average for two sets. For multiple sets, iteration order is optimized to start with the smallest set. The result set is pre-allocated to the smallest input size. Dict view intersection (`.keys() & .keys()`) is as efficient as set intersection because dict keys are backed by a hash table.

### Interview Questions

1. How does CPython optimize set intersection?
2. Why is `a & b & c` efficient?
3. Can you intersect more than two sets?
4. How does dict view intersection work?
5. What is the time complexity of intersection?
6. What happens when intersecting with an empty set?

### Coding Challenges

```python
# Challenge 1: Multi-set intersection reducer
def intersect_all(sets):
    if not sets:
        return set()
    return set.intersection(*sets)

sets = [{1, 2, 3}, {2, 3, 4}, {3, 4, 5}]
print(intersect_all(sets))  # {3}

# Challenge 2: Intersection-based tag search
def search_docs(docs, required_tags):
    required = set(required_tags)
    return [doc for doc in docs if required & set(doc["tags"]) == required]

docs = [
    {"title": "A", "tags": ["python", "ml"]},
    {"title": "B", "tags": ["python", "web"]},
    {"title": "C", "tags": ["python", "ml", "data"]},
]
print([d["title"] for d in search_docs(docs, ["python", "ml"])])

# Challenge 3: Common element across all sublists
def common_in_all(lists):
    if not lists:
        return []
    common = set(lists[0])
    for lst in lists[1:]:
        common &= set(lst)
    return list(common)

data = [[1, 2, 3, 4], [2, 3, 4, 5], [3, 4, 5, 6]]
print(common_in_all(data))  # [3, 4]
```

### Related Topics

- set union (a | b)
- set difference (a - b)
- set symmetric difference (a ^ b)
- dict view intersection
- Boolean algebra
- Venn diagrams

## Difference

### What It Is

Difference returns elements present in one set but not in another. Python provides two forms: difference (`-` or `.difference()`) returns elements in the left set but not the right, and symmetric difference (`^` or `.symmetric_difference()`) returns elements in either set but not both.

### Why It Is Important

Difference is essential for exclusion filtering, finding unique elements, change detection (A - B shows what was removed), and data reconciliation. Symmetric difference is crucial for identifying elements that are exclusive to each set.

### How It Works Internally

The `-` operator calls `PySet_Difference()`. CPython iterates over the left set and checks membership in the right set, collecting elements not found in the right. The result is pre-allocated to the size of the left set. `^` (symmetric difference) is implemented as `(a - b) | (b - a)` internally, or directly as `PySet_SymmetricDifference()`, which iterates both sets and tracks elements that appear in only one.

### Syntax

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

# Difference
result = a - b      # Elements in a but not b
result = a.difference(b)

# Symmetric difference
result = a ^ b      # Elements in a or b but not both
result = a.symmetric_difference(b)

# In-place
a -= b               # a becomes a - b
a.difference_update(b)

a ^= b               # a becomes a ^ b
a.symmetric_difference_update(b)
```

### Beginner Examples

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

# Difference
print(a - b)  # {1, 2}
print(b - a)  # {5, 6}

# Symmetric difference
print(a ^ b)  # {1, 2, 5, 6}

# With strings
a = set("hello")
b = set("world")
print(a - b)  # {'h', 'e'}
print(a ^ b)  # {'h', 'e', 'w', 'r', 'd'}

# Empty difference
print({1, 2} - {1, 2})  # set()

# Single set
print(a - a)  # set()
```

### Intermediate Examples

```python
# Finding unique elements
all_users = {"alice", "bob", "charlie", "diana"}
active_users = {"alice", "charlie"}
inactive = all_users - active_users
print(inactive)  # {'bob', 'diana'}

# Change detection
old_permissions = {"read", "write", "delete", "admin"}
new_permissions = {"read", "write", "export"}
added = new_permissions - old_permissions
removed = old_permissions - new_permissions
print(f"Added: {added}, Removed: {removed}")  # Added: {'export'}, Removed: {'delete', 'admin'}

# Symmetric difference for comparison
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
exclusive = a ^ b
print(exclusive)  # {1, 2, 5, 6}

# Difference with non-set
print(a.difference([3, 4, 5]))  # {1, 2}

# Chaining differences
a = {1, 2, 3, 4, 5}
b = {2, 3}
c = {4}
result = a - b - c
print(result)  # {1, 5}
```

### Advanced Examples

```python
# Symmetric difference of multiple sets
def symmetric_diff_all(*sets):
    if not sets:
        return set()
    result = sets[0]
    for s in sets[1:]:
        result ^= s
    return result

sets = [{1, 2}, {2, 3}, {3, 4}]
print(symmetric_diff_all(*sets))  # {1, 4}

# Difference with dict views
d1 = {"a": 1, "b": 2, "c": 3}
d2 = {"b": 4, "c": 5, "d": 6}
only_in_d1 = d1.keys() - d2.keys()
only_in_d2 = d2.keys() - d1.keys()
print(only_in_d1, only_in_d2)  # {'a'} {'d'}

# Symmetric difference for data validation
def validate_sets(expected, actual):
    missing = expected - actual
    extra = actual - expected
    report = {}
    if missing:
        report["missing"] = missing
    if extra:
        report["extra"] = extra
    return report

expected = {"required": 1, "optional": 2}
actual = {"required": 1, "extra_field": 3}
print(validate_sets(set(expected), set(actual)))

# Exclusive content recommendation
user1 = {"python", "ml", "data"}
user2 = {"python", "web", "devops"}
exclusive_to_user1 = user1 - user2
exclusive_to_user2 = user2 - user1
print(f"Recommend to user2: {exclusive_to_user1}")
print(f"Recommend to user1: {exclusive_to_user2}")
```

### Real-World Use Cases

- **Data reconciliation**: Finding differences between two datasets
- **Permission diffing**: Determining added/removed permissions
- **Sync operations**: Identifying files to add/delete in backups
- **A/B testing**: Finding unique users in test vs control groups
- **Schema migration**: Detecting added/removed columns
- **Access control**: Users with exclusive access to resources
- **Inventory**: Items in one warehouse but not another

### Common Mistakes

```python
# Mistake 1: Forgetting difference is not commutative
a = {1, 2, 3}
b = {2, 3, 4}
print(a - b)  # {1}
print(b - a)  # {4} (different!)

# Mistake 2: Using - with non-set types
# a - [3, 4]  # TypeError
a.difference([3, 4])  # Works

# Mistake 3: Confusing - with ^
a = {1, 2}
b = {2, 3}
print(a - b)  # {1}
print(a ^ b)  # {1, 3}

# Mistake 4: Forgetting -= creates in-place modification
a = {1, 2, 3}
b = {2}
c = a - b
print(a)  # {1, 2, 3} -- unchanged
a -= b
print(a)  # {1, 3} -- modified

# Mistake 5: Using difference with overlapping in-place updates
a = {1, 2}
b = {2, 3}
a -= b
# a is now {1}
# If you need both a and b after, use a - b first, then assign
```

### Best Practices

- Use `-` for set-set difference (more readable)
- Use `.difference()` when the right operand is not a set
- Use `^` for symmetric difference (exclusive elements)
- Use `-=` and `^=` for in-place updates
- Use dict key view difference (`d1.keys() - d2.keys()`)
- Use difference for change detection patterns
- Combine symmetric difference with intersection for full set comparison

### Performance Considerations

`-` is O(len(a)) average — CPython iterates a and checks membership in b. `^` is O(len(a) + len(b)). Dict key view difference is as efficient as set difference. Pre-allocating the result set avoids resizing overhead. For very large sets, difference is more efficient than union followed by filter.

### Interview Questions

1. What is the difference between `-` and `.difference()`?
2. What is symmetric difference and when is it useful?
3. How does CPython implement set difference?
4. Is set difference commutative?
5. What is the time complexity of `a - b`?
6. How do you compute symmetric difference of more than 2 sets?
7. How can dict key views be used with difference?

### Coding Challenges

```python
# Challenge 1: Change tracker
def detect_changes(old, new):
    added = new - old
    removed = old - new
    unchanged = old & new
    return {"added": added, "removed": removed, "unchanged": unchanged}

old = {"read", "write", "delete"}
new = {"read", "write", "export"}
print(detect_changes(old, new))

# Challenge 2: Exclusive items
def exclusive_items(a, b):
    return {"only_a": a - b, "only_b": b - a, "both": a & b}

a = {"alice", "bob", "charlie"}
b = {"bob", "diana", "eve"}
print(exclusive_items(a, b))

# Challenge 3: Set diff for backup sync
def backup_sync(local, remote):
    to_upload = local - remote
    to_download = remote - local
    return to_upload, to_download

local = {"f1.txt", "f2.txt", "f3.txt"}
remote = {"f1.txt", "f4.txt"}
upload, download = backup_sync(local, remote)
print(f"Upload: {upload}, Download: {download}")

# Challenge 4: Symmetric difference for A/B test exclusivity
def ab_exclusive(users_a, users_b):
    return {"only_a": len(users_a - users_b), "only_b": len(users_b - users_a)}

a = set(range(1000))
b = set(range(800, 1800))
print(ab_exclusive(a, b))  # only_a: 800, only_b: 800

# Challenge 5: Set difference for tag-based unsubscription
def recommend_unsubscribes(user_tags, content_tags):
    return content_tags - user_tags

user = {"python", "data"}
content = {"python", "web", "ml"}
print(recommend_unsubscribes(user, content))  # {'web', 'ml'}
```

### Related Topics

- set union
- set intersection
- symmetric difference
- frozenset operations
- Dict key views
- Boolean algebra
- Change detection patterns
