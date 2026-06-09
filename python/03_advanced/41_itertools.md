# Itertools - count, cycle, chain, groupby, permutations (std lib)

## Introduction

The `itertools` module is a collection of fast, memory-efficient tools for working with iterators. It provides building blocks for creating complex iterator-based data pipelines. The functions in `itertools` are inspired by functional programming languages and Haskell's list comprehensions. They compose together seamlessly, enabling elegant solutions to many data processing problems.

## count and cycle

### What It Is

`itertools.count()` is an infinite iterator that generates consecutive numbers starting from a given value with a configurable step. `itertools.cycle()` is an infinite iterator that cycles through an iterable indefinitely, repeating the sequence forever.

### Why It Is Important

`count` and `cycle` provide infinite sequences that can be used for enumeration, cycling through options, generating unique IDs, and implementing round-robin algorithms. Combined with `zip` or `takewhile`, they can simulate finite sequences with precise control.

### How It Works Internally

`count(start=0, step=1)` creates an iterator that yields `start`, `start + step`, `start + 2*step`, ... indefinitely. At the C level, it uses native arithmetic to generate the next value. `cycle(iterable)` saves a copy of the iterable's elements in a list during the first pass, then repeatedly yields from the saved list.

### Syntax

```python
from itertools import count, cycle

# count
for i in count(10, 2):  # 10, 12, 14, 16, ...
    if i > 20: break

# cycle
colors = cycle(['red', 'green', 'blue'])
# red, green, blue, red, green, blue, ...
```

### Beginner Examples

```python
from itertools import count, cycle, islice

# enumerate with custom start
for i, item in zip(count(1), ['a', 'b', 'c']):
    print(f"{i}: {item}")
# 1: a, 2: b, 3: c

# cycle for alternating values
tasks = ["task1", "task2", "task3"]
workers = cycle(["Alice", "Bob"])
pairs = list(zip(tasks, workers))
print(pairs)  # [('task1', 'Alice'), ('task2', 'Bob'), ('task3', 'Alice')]

# Finite slice of infinite iterator
first_5 = list(islice(count(100, -5), 5))
print(first_5)  # [100, 95, 90, 85, 80]
```

### Intermediate Examples

```python
from itertools import count, cycle, islice, takewhile

# Generating indices with multiples
multiples_of_3_or_5 = (n for n in count(1) if n % 3 == 0 or n % 5 == 0)
first_10 = list(islice(multiples_of_3_or_5, 10))
print(first_10)  # [3, 5, 6, 9, 10, 12, 15, 18, 20, 21]

# Round-robin scheduling with cycle
def round_robin(workers, tasks):
    worker_cycle = cycle(workers)
    assignments = []
    for task, worker in zip(tasks, worker_cycle):
        assignments.append((worker, task))
    return assignments

workers = ["Alice", "Bob", "Charlie"]
tasks = ["T1", "T2", "T3", "T4", "T5"]
print(round_robin(workers, tasks))
# [('Alice', 'T1'), ('Bob', 'T2'), ('Charlie', 'T3'), ('Alice', 'T4'), ('Bob', 'T5')]

# count with takewhile
values = list(takewhile(lambda x: x < 100, count(1, 7)))
print(values)  # [1, 8, 15, 22, 29, 36, 43, 50, 57, 64, 71, 78, 85, 92, 99]
```

### Advanced Examples

```python
from itertools import count, cycle, product, islice

# Matrix coordinate generator
coords = product(count(), count())  # (0,0), (0,1), (0,2), (1,0), ...
first_6 = list(islice(coords, 6))
print(first_6)  # [(0, 0), (0, 1), (0, 2), (1, 0), (1, 1), (1, 2)]

# Auto-incrementing IDs with count
def track_changes():
    id_gen = count(1)
    history = []
    def record(change):
        change_id = next(id_gen)
        history.append((change_id, change))
        return change_id
    return record, history

record, history = track_changes()
record("User created")
record("User updated")
print(history)  # [(1, 'User created'), (2, 'User updated')]

# Cycling with progress indicator
def spinner():
    symbols = cycle(['|', '/', '-', '\\'])
    steps = 10
    for _ in range(steps):
        print(f"\rProcessing {next(symbols)}", end="")
    print("\rDone  ")

# Count with floating point (precision note)
import math
approx_pi = sum(4.0 / (2*i + 1) * (1 if i % 2 == 0 else -1) for i in range(1000000))
print(approx_pi)  # ~3.141591...
```

### Real-World Use Cases

- **Enumerating results**: `zip(count(1), results)` to add line numbers.
- **Round-robin scheduling**: Distribute tasks evenly across worker pool.
- **Progress spinners**: `cycle(['|', '/', '-', '\\'])` for loading animations.
- **Unique ID generation**: `count()` as a source of auto-incrementing IDs.
- **Numeric sequences**: Generate arithmetic progressions with specific step sizes.
- **Paginated API fetching**: `count(1)` for page numbers with `takewhile`.

### Common Mistakes

- Using infinite iterators without limiting them (causes infinite loops).
- Expecting `count()` to work with floating-point without precision issues (use decimal steps carefully).
- Using `cycle()` with a very large iterable (it caches all elements in memory).
- Forgetting to import `itertools` functions individually or as `from itertools import ...`.
- Using `next(count())` directly without understanding it's infinite.

### Best Practices

- Always combine infinite iterators with limiting functions (`islice`, `takewhile`, `zip(range(n), ...)`).
- Use `count(start, step)` instead of `while True: yield value; value += step` for clarity.
- Use `cycle(iterable)` with `zip` for pairing data with repeating patterns.
- Use `islice(count(...), n)` for finite sequences.
- Prefer `count` for arithmetic progressions and `range` for finite ranges.

### Performance Considerations

`count` and `cycle` are implemented in C and are very fast. `count` does one integer add per iteration. `cycle` saves elements to a list during the first iteration (memory proportional to input size). For simple finite numeric ranges, `range()` is faster than `islice(count(), n)`.

### Interview Questions

**Q: How do you create a finite sequence from `itertools.count()`?**

A: Use `itertools.islice(count(start, step), n)` to get exactly n elements, or `itertools.takewhile(predicate, count(...))` to get elements while a condition holds.

**Q: What is the memory behavior of `itertools.cycle()`?**

A: `cycle()` stores a copy of the iterable elements in a list during the first iteration. This means it uses O(n) memory where n is the number of elements in the input iterable.

### Coding Challenges

1. Generate the first 20 triangular numbers using `count()`.
2. Implement a simple traffic light state machine using `cycle()`.
3. Use `count()` to implement a paginated API client that stops when results are empty.

### Related Topics

- `itertools.islice` (limiting infinite iterators)
- `itertools.takewhile` (conditional limiting)
- `itertools.repeat` (repeating a single value)
- `zip` and `zip_longest` (combining with finite iterables)
- Generators (the broader context for itertools)

## chain

### What It Is

`itertools.chain()` takes multiple iterables and returns an iterator that yields elements from the first iterable, then the second, then the third, and so on, as if they were concatenated into a single sequence. `chain.from_iterable()` is a variant that takes a single iterable of iterables.

### Why It Is Important

`chain` eliminates the need to create intermediate lists when combining iterables. Instead of `list1 + list2 + list3` (which creates a new list), `chain(list1, list2, list3)` lazily iterates through each without copying elements.

### How It Works Internally

`chain(*iterables)` creates an iterator that iterates over each iterable in sequence. When one iterable is exhausted, it moves to the next. `chain.from_iterable(iterable_of_iterables)` does the same but takes a single iterable containing sub-iterables. Both are implemented in C for efficiency.

### Syntax

```python
from itertools import chain

# Multiple iterables
for item in chain(iter1, iter2, iter3):
    ...

# From iterable of iterables
for item in chain.from_iterable(iterable_of_iterables):
    ...
```

### Beginner Examples

```python
from itertools import chain

# Concatenating lists lazily
list1 = [1, 2, 3]
list2 = [4, 5]
list3 = [6, 7, 8]
combined = list(chain(list1, list2, list3))
print(combined)  # [1, 2, 3, 4, 5, 6, 7, 8]

# Without chain (creates intermediate list)
combined_copy = list1 + list2 + list3

# chain with different types
result = list(chain("abc", [1, 2, 3], (4, 5)))
print(result)  # ['a', 'b', 'c', 1, 2, 3, 4, 5]
```

### Intermediate Examples

```python
from itertools import chain

# Flattening a list of lists
nested = [[1, 2], [3, 4, 5], [6]]
flat = list(chain.from_iterable(nested))
print(flat)  # [1, 2, 3, 4, 5, 6]

# Equivalent list comprehension
flat2 = [item for sublist in nested for item in sublist]

# Merging multiple data sources
def read_all_files(filenames):
    def generate_lines():
        for filename in filenames:
            with open(filename) as f:
                yield f.readlines()
    return chain.from_iterable(generate_lines())

# Using chain to process different iterable types
data = chain(
    [1, 2, 3],
    range(4, 7),
    (x * 10 for x in [7, 8, 9])
)
print(list(data))  # [1, 2, 3, 4, 5, 6, 70, 80, 90]
```

### Advanced Examples

```python
from itertools import chain

# Building an iterator over multiple sources
class MultiSourceReader:
    def __init__(self, *sources):
        self._chain = chain(*sources)

    def __iter__(self):
        return self

    def __next__(self):
        return next(self._chain)

# Reading chunks from multiple files
def read_chunks(files, chunk_size=1024):
    def file_chunks():
        for file in files:
            while True:
                chunk = file.read(chunk_size)
                if not chunk:
                    break
                yield chunk
    return chain.from_iterable(file_chunks())

# chain with map for nested structure processing
data = [
    {"items": [1, 2]},
    {"items": [3, 4, 5]},
    {"items": [6]}
]
all_items = list(chain.from_iterable(
    d["items"] for d in data
))
print(all_items)  # [1, 2, 3, 4, 5, 6]

# chain for merging sorted sequences (use heapq.merge for sorted)
import heapq
list1 = [1, 3, 5, 7]
list2 = [2, 4, 6, 8]
merged_sorted = list(heapq.merge(list1, list2))
print(merged_sorted)  # [1, 2, 3, 4, 5, 6, 7, 8]
```

### Real-World Use Cases

- **Processing multiple files**: Iterate over lines from multiple files as if one stream.
- **Data pipeline assembly**: Combine multiple transformations into a single iterator.
- **API response aggregation**: Merge paginated API responses into a single stream.
- **Configuration merging**: Combine configuration from multiple sources.
- **Lazy concatenation**: Avoid copying large lists when combining data sources.

### Common Mistakes

- Using `chain([a, b, c])` when `chain(a, b, c)` is intended (first one chains one iterable of three items).
- Using `chain.from_iterable(nested)` when `chain(*nested)` works (both work, from_iterable is more explicit).
- Expecting `chain` to sort or deduplicate (it just concatenates; use `heapq.merge` for sorted merge).
- Creating an intermediate list to flatten when `chain.from_iterable` works lazily.

### Best Practices

- Use `chain` instead of `+` for combining iterables to avoid creating intermediate lists.
- Use `chain.from_iterable` for flattening a single iterable of iterables.
- Use `heapq.merge` for combining sorted iterables in order.
- Use `chain` to combine different data sources in data pipeline patterns.
- Chain generator expressions for complex lazy transformations.

### Performance Considerations

`chain` is O(1) memory — it doesn't create a combined copy. Each `next()` call retrieves the next element from the current sub-iterator. The overhead per element is one C function call. For concatenating many iterables, the overhead of switching between them is negligible.

### Interview Questions

**Q: What is the difference between `chain` and `chain.from_iterable`?**

A: `chain(*iterables)` takes multiple iterables as positional arguments. `chain.from_iterable(iterable)` takes a single iterable that yields iterables. Use `chain` when you know the iterables at the call site; use `from_iterable` when the iterables are computed or come from another iterator.

**Q: How is `chain` different from concatenating lists with `+`?**

A: `+` creates a new list with all elements copied. `chain` creates a lazy iterator that yields from each input iterable sequentially. `chain` uses O(1) extra memory regardless of input sizes, while `+` uses O(n) memory.

### Coding Challenges

1. Use `chain.from_iterable` to flatten a deeply nested list structure (list of lists of lists).
2. Implement a concatenation of multiple CSV files into a single stream of rows using `chain`.
3. Use `chain` with `map` to read and combine multiple data streams.

### Related Topics

- `itertools.chain.from_iterable`
- `itertools.islice` (slicing chained iterables)
- `heapq.merge` (sorted merge)
- List concatenation (`+` operator)
- Generator expressions (often combined with chain)

## groupby

### What It Is

`itertools.groupby()` returns an iterator that produces consecutive keys and groups from an iterable. It groups elements based on a key function, but only consecutive elements with the same key are placed in the same group. It is similar to the Unix `uniq` command and SQL's `GROUP BY` but requires sorted input for meaningful aggregation.

### Why It Is Important

`groupby` provides a functional way to group consecutive elements, which is useful for run-length encoding, aggregating sorted data, and processing consecutive duplicates. It is memory-efficient (only stores the current group) and integrates well with sorting for complete grouping functionality.

### How It Works Internally

`groupby(iterable, key=None)` scans the iterable sequentially. When the key function's return value changes, it yields the previous key and a sub-iterator over the elements that shared that key. The sub-iterator shares the underlying iterator with the main `groupby`, so it must be consumed before moving to the next group.

### Syntax

```python
from itertools import groupby

for key, group in groupby(iterable, key_function):
    # key: the shared key value
    # group: iterator over elements with this key
```

### Beginner Examples

```python
from itertools import groupby

# Basic consecutive grouping
data = "AAABBBCCAA"
for key, group in groupby(data):
    print(f"{key}: {''.join(group)}")
# A: AAA, B: BBB, C: CC, A: AA

# Without sorting, non-consecutive duplicates are separate
data = [1, 1, 2, 2, 1, 1]
for key, group in groupby(data):
    print(f"{key}: {list(group)}")
# 1: [1, 1], 2: [2, 2], 1: [1, 1]

# With sorting first
data = [3, 1, 2, 1, 3, 2]
sorted_data = sorted(data)
for key, group in groupby(sorted_data):
    print(f"{key}: {list(group)}")
# 1: [1, 1], 2: [2, 2], 3: [3, 3]
```

### Intermediate Examples

```python
from itertools import groupby
from operator import itemgetter

# Grouping records by a field
records = [
    {"department": "Engineering", "name": "Alice"},
    {"department": "Engineering", "name": "Bob"},
    {"department": "Sales", "name": "Charlie"},
    {"department": "Engineering", "name": "Diana"},
    {"department": "Sales", "name": "Eve"},
]

# Sort by the grouping key first
records.sort(key=itemgetter("department"))

for dept, group in groupby(records, key=itemgetter("department")):
    names = [r["name"] for r in group]
    print(f"{dept}: {', '.join(names)}")
# Engineering: Alice, Bob, Diana
# Sales: Charlie, Eve

# Run-length encoding
def rle_encode(data):
    return [(key, len(list(group))) for key, group in groupby(data)]

def rle_decode(encoded):
    return ''.join(key * count for key, count in encoded)

original = "AAABBCCCCD"
encoded = rle_encode(original)
print(encoded)  # [('A', 3), ('B', 2), ('C', 4), ('D', 1)]
decoded = rle_decode(encoded)
print(decoded)  # "AAABBCCCCD"
```

### Advanced Examples

```python
from itertools import groupby
from datetime import datetime, timedelta
import random

# Grouping dates by week
dates = [datetime(2024, 1, 1) + timedelta(days=random.randint(0, 60)) for _ in range(20)]
dates.sort()
for week_start, group in groupby(dates, key=lambda d: d - timedelta(days=d.weekday())):
    print(f"Week of {week_start.date()}: {len(list(group))} items")

# Aggregating with groupby
transactions = [
    {"user": "Alice", "amount": 100},
    {"user": "Bob", "amount": 50},
    {"user": "Alice", "amount": 200},
    {"user": "Charlie", "amount": 75},
    {"user": "Bob", "amount": 150},
]
transactions.sort(key=lambda t: t["user"])

for user, group in groupby(transactions, key=lambda t: t["user"]):
    amounts = [t["amount"] for t in group]
    print(f"{user}: total={sum(amounts)}, count={len(amounts)}, avg={sum(amounts)/len(amounts):.1f}")
# Alice: total=300, count=2, avg=150.0
# Bob: total=200, count=2, avg=100.0
# Charlie: total=75, count=1, avg=75.0

# Nested grouping
def group_and_count(data, key_fn, value_fn=lambda x: 1):
    data_sorted = sorted(data, key=key_fn)
    result = {}
    for key, group in groupby(data_sorted, key=key_fn):
        result[key] = sum(value_fn(item) for item in group)
    return result
```

### Real-World Use Cases

- **Log analysis**: Group log entries by error type or timestamp.
- **Data aggregation**: Group sales records by region, product, or salesperson.
- **Run-length encoding**: Compress repeating sequences in data.
- **Time series analysis**: Group events by hour, day, or week.
- **Sequence analysis**: Find consecutive duplicates or patterns in sequences.

### Common Mistakes

- Not sorting before `groupby` when full grouping (not just consecutive) is desired.
- Saving the group iterator without consuming it (it points to shared, advancing data).
- Using `list(group)` inside a loop that iterates `groupby` (consuming the group is fine, but storing for later is risky).
- Expecting `groupby` to work like SQL GROUP BY (it doesn't — SQL aggregates across all rows with the same key).
- Forgetting that the group iterator is only valid while the current key is active.

### Best Practices

- Always sort by the key function before `groupby` unless you specifically want consecutive grouping.
- Consume each group immediately within the loop (e.g., convert to `list(group)`).
- Use `operator.itemgetter` or `operator.attrgetter` as key functions for clarity.
- Combine `groupby` with `sorted` for complete grouping functionality.
- Use `groupby` for run-length encoding and consecutive duplicate detection.

### Performance Considerations

`groupby` is O(n) — it scans the iterable once. Sorting before grouping adds O(n log n). The groups are sub-iterators that share the underlying iterator — consuming a group advances the main iterator. Memory usage is O(1) for the groupby itself, but `list(group)` inside the loop uses memory proportional to the largest group.

### Interview Questions

**Q: Why must data be sorted before using `groupby` for complete grouping?**

A: `groupby` only groups consecutive elements with the same key. If elements with the same key are scattered throughout the iterable, they will appear in multiple groups. Sorting by the key brings all same-key elements together.

**Q: What happens if you don't consume a group iterator before moving to the next key?**

A: The group iterator shares the underlying iterator with `groupby`. If you don't consume the group, the next `next()` call on the groupby advances the underlying iterator, potentially skipping elements or producing empty groups.

### Coding Challenges

1. Use `groupby` to implement a run-length decoder that expands `[('A',3), ('B',2)]` to `AAABB`.
2. Group a list of birthdays by month and count how many birthdays are in each month.
3. Find the longest consecutive run of the same character in a string using `groupby`.
4. Implement a log parser that groups log lines by their timestamp buckets (e.g., hourly).

### Related Topics

- `sorted()` function (prerequisite for full groupby)
- `operator.itemgetter` (common key function)
- `itertools.compress` (filtering with selectors)
- Run-length encoding patterns
- SQL GROUP BY equivalent

## permutations and combinations

### What It Is

`itertools.permutations()` generates all possible orderings of items from an iterable. `itertools.combinations()` generates all possible subsets of a given length from an iterable, without regard to order. `itertools.combinations_with_replacement()` generates combinations where elements can be chosen multiple times.

### Why It Is Important

These combinatoric generators are essential for solving problems in probability, optimization, scheduling, cryptography, and game AI. They provide memory-efficient iteration over combinatoric spaces, avoiding the need to generate all possibilities at once.

### How It Works Internally

`permutations(iterable, r)` generates tuples of length r in lexicographic order of the input positions. It uses a backtracking algorithm to generate permutations. `combinations(iterable, r)` generates tuples in lexicographic order using a similar backtracking approach. Both generate results lazily, yielding one tuple per iteration.

### Syntax

```python
from itertools import permutations, combinations, combinations_with_replacement

permutations(iterable, r=None)  # r defaults to len(iterable)
combinations(iterable, r)
combinations_with_replacement(iterable, r)
```

### Beginner Examples

```python
from itertools import permutations, combinations

# Permutations (order matters)
items = ['A', 'B', 'C']
perms = list(permutations(items, 2))
print(perms)  # [('A','B'), ('A','C'), ('B','A'), ('B','C'), ('C','A'), ('C','B')]

# Permutations of all items
perms_full = list(permutations(items))
print(perms_full)  # 6 permutations

# Combinations (order doesn't matter)
combs = list(combinations(items, 2))
print(combs)  # [('A','B'), ('A','C'), ('B','C')]

# Combinations with replacement
combs_r = list(combinations_with_replacement(items, 2))
print(combs_r)  # [('A','A'), ('A','B'), ('A','C'), ('B','B'), ('B','C'), ('C','C')]
```

### Intermediate Examples

```python
from itertools import permutations, combinations, combinations_with_replacement
import math

# Verifying counts
items = [1, 2, 3, 4, 5]
r = 3
print(f"Permutations: {len(list(permutations(items, r)))} = {math.perm(len(items), r)}")
print(f"Combinations: {len(list(combinations(items, r)))} = {math.comb(len(items), r)}")

# Generating all subsets (power set)
def power_set(iterable):
    s = list(iterable)
    for r in range(len(s) + 1):
        yield from combinations(s, r)

ps = list(power_set([1, 2, 3]))
print(ps)  # [(), (1,), (2,), (3,), (1,2), (1,3), (2,3), (1,2,3)]

# Word unscrambler
word = "cat"
scrambles = [''.join(p) for p in permutations(word)]
print(scrambles)  # ['cat', 'cta', 'act', 'atc', 'tca', 'tac']
```

### Advanced Examples

```python
from itertools import permutations, combinations, product

# Traveling Salesman Problem (brute force)
def tsp_bruteforce(cities, distances):
    best_route = None
    best_distance = float('inf')
    for perm in permutations(cities[1:]):
        route = [cities[0]] + list(perm)
        total = sum(distances[route[i]][route[i+1]] for i in range(len(route)-1))
        if total < best_distance:
            best_distance = total
            best_route = route
    return best_route, best_distance

cities = ['A', 'B', 'C', 'D']
distances = {
    'A': {'B': 10, 'C': 15, 'D': 20},
    'B': {'A': 10, 'C': 35, 'D': 25},
    'C': {'A': 15, 'B': 35, 'D': 30},
    'D': {'A': 20, 'B': 25, 'C': 30}
}
route, dist = tsp_bruteforce(cities, distances)
print(f"Best route: {route}, distance: {dist}")

# Cartesian product (like nested for loops)
suits = ['♠', '♥', '♦', '♣']
ranks = ['A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K']
deck = list(product(ranks, suits))
print(f"Deck has {len(deck)} cards")  # 52

# Combinations for lottery probability
lottery_numbers = list(range(1, 50))
possible_combinations = math.comb(49, 6)
print(f"Lottery combinations: {possible_combinations:,}")  # 13,983,816
```

### Real-World Use Cases

- **Password cracking**: Generate all possible password combinations.
- **Scheduling**: Assign tasks to workers in all possible ways.
- **Game AI**: Evaluate all possible moves in turn-based games.
- **Optimization**: Brute-force search for small solution spaces.
- **Genetics**: Generate all possible gene sequence combinations.
- **Testing**: Generate all combinations of test parameters (use `product` for parameter grids).

### Common Mistakes

- Using permutations when combinations are needed (order doesn't matter) — 10x more results.
- Using `list(permutations(...))` for large inputs (memory explosion).
- Forgetting that `r` defaults to `len(iterable)` for permutations.
- Using `combinations_with_replacement` when simple `combinations` is needed.
- Expecting `permutations` to work with duplicate input values (it treats positions uniquely, not values).

### Best Practices

- Use `combinations` when order doesn't matter, `permutations` when it does.
- Use `product` for Cartesian products (nested loop replacement).
- Use `math.comb` and `math.perm` to check counts before generating.
- Use `islice` to limit combinatoric generation for large spaces.
- Use `combinations_with_replacement` for multiset combinations.

### Performance Considerations

Combinatoric generators produce O(n! / (n-r)!) permutations and O(n choose r) combinations. For n=20, permutations(20) has 2.4 quintillion results. Always estimate the size before generating. The generators themselves are memory-efficient (O(r) storage), but converting to `list()` can exhaust memory.

### Interview Questions

**Q: What is the difference between `combinations` and `combinations_with_replacement`?**

A: `combinations` does not allow repeated elements — each element can appear at most once. `combinations_with_replacement` allows elements to be chosen multiple times. For example, `combinations('ABC', 2)` gives AB, AC, BC, while `combinations_with_replacement('ABC', 2)` adds AA, BB, CC.

**Q: How many permutations does `permutations([1..n])` produce?**

A: `n!` (n factorial). For n elements where r=n, `permutations` produces n! results. For `permutations(iterable, r)`, it produces n! / (n-r)! results.

### Coding Challenges

1. Generate all possible poker hands (5-card combinations) from a standard deck.
2. Use `permutations` to find all possible ways to schedule 4 meetings in 5 time slots.
3. Implement a brute-force solution to the knapsack problem using `combinations`.
4. Generate all possible valid parentheses combinations for n pairs using `itertools`.

### Related Topics

- `itertools.product` (Cartesian product)
- `itertools.compress` (filtering combinations)
- `math.comb` and `math.perm` (count calculations)
- `random.sample` (random selection without generation)
- Combinatorics mathematics
