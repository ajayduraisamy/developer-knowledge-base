# Slicing - sequence[start:stop:step], negative indices

## Introduction

Slicing extracts subsequences from ordered sequences (strings, lists, tuples, bytes, ranges) using the concise `[start:stop:step]` syntax. It creates a new sequence containing the specified range without modifying the original. Combined with negative indexing (counting from the end), slicing enables elegant reversal, striding, and partial-copy patterns. Python also provides the `slice()` built-in for programmatic slicing and supports slice assignment on mutable sequences.

## sequence[start:stop:step]

### What It Is

The slice notation `s[start:stop:step]` extracts elements from `start` up to (but not including) `stop`, taking every `step`-th element. All three components are optional. With positive step, default `start` is 0 and default `stop` is the sequence length. Python slices never raise `IndexError` — out-of-bounds indices are silently clamped to the sequence boundaries.

### Why It Is Important

Slicing replaces explicit loops for subsequence extraction, making code more readable and less error-prone. It is used universally in data processing (extracting columns, windows, strides), string parsing, sequence reversal, and pagination.

### How It Works Internally

CPython implements slicing via `PySlice_New()` and the `sq_slice` or `mp_subscript` slot. When Python encounters `s[start:stop:step]`, the compiler generates a `BUILD_SLICE` opcode that creates a `PySliceObject` containing `start`, `stop`, `step` (any can be `None`). The sequence's `__getitem__` method receives this slice object and computes the actual indices using `PySlice_AdjustIndices()` and `PySlice_Unpack()`. For lists and tuples, the result is a new container with copied element references (shallow). For strings, it creates a new string by copying the relevant characters (or using a compact representation in newer Python versions).

### Syntax

```python
s[start:stop]       # Elements from start to stop-1
s[start:]           # From start to end
s[:stop]            # From beginning to stop-1
s[:]                # Full shallow copy
s[::step]           # Every step-th element
s[start:stop:step]  # Strided extraction
```

### Beginner Examples

```python
# Basic string slicing
text = "Python Programming"
print(text[0:6])     # Python
print(text[7:])      # Programming
print(text[:6])      # Python
print(text[:])       # Full copy

# List slicing
nums = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print(nums[2:5])     # [2, 3, 4]
print(nums[:5])      # [0, 1, 2, 3, 4]
print(nums[5:])      # [5, 6, 7, 8, 9]

# With step
print(nums[::2])     # [0, 2, 4, 6, 8]
print(nums[1::2])    # [1, 3, 5, 7, 9]
print(nums[2:8:3])   # [2, 5]

# Reverse
print(nums[::-1])    # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```

### Intermediate Examples

```python
# Dynamic slicing with variables
data = list(range(20))
start, stop, step = 5, 15, 2
print(data[start:stop:step])  # [5, 7, 9, 11, 13]

# Slice assignment on lists
nums = [1, 2, 3, 4, 5]
nums[1:3] = [20, 30]
print(nums)  # [1, 20, 30, 4, 5]

# Replace with different length
nums[1:3] = [100, 200, 300, 400]
print(nums)  # [1, 100, 200, 300, 400, 4, 5]

# Delete via slice
nums[1:5] = []
print(nums)  # [1, 4, 5]

# Insert at position
nums[1:1] = [2, 3]
print(nums)  # [1, 2, 3, 4, 5]

# Slicing tuples
t = (10, 20, 30, 40, 50)
print(t[1:4])    # (20, 30, 40)
print(t[::-1])   # (50, 40, 30, 20, 10)
```

### Advanced Examples

```python
# Slice object for programmatic slicing
s = slice(2, 8, 2)
nums = list(range(10))
print(nums[s])  # [2, 4, 6]
print(s.start, s.stop, s.step)  # 2 8 2

# Custom class with slicing support
class Frame:
    def __init__(self, data):
        self.data = data
    def __getitem__(self, idx):
        if isinstance(idx, slice):
            return Frame(self.data[idx])
        return self.data[idx]
    def __repr__(self):
        return f"Frame({self.data})"

f = Frame([1, 2, 3, 4, 5, 6])
print(f[1:4])    # Frame([2, 3, 4])
print(f[::-1])   # Frame([6, 5, 4, 3, 2, 1])

# Chunking with slicing
def chunks(lst, n):
    return [lst[i:i+n] for i in range(0, len(lst), n)]

print(chunks(list(range(10)), 3))  # [[0,1,2], [3,4,5], [6,7,8], [9]]

# Moving average with slicing
def moving_avg(data, window):
    return [sum(data[i:i+window])/window for i in range(len(data)-window+1)]

prices = [10, 12, 13, 15, 18, 20, 22]
print(moving_avg(prices, 3))  # [11.67, 13.33, 15.33, 17.67, 20.0]

# Palindrome check
def is_palindrome(s):
    s = s.lower().replace(" ", "")
    return s == s[::-1]

print(is_palindrome("racecar"))  # True
```

### Real-World Use Cases

- **Data processing**: Extracting columns from matrix rows
- **String parsing**: Extracting substrings from fixed-format text
- **Pagination**: Slicing API results into pages: `data[page*size:(page+1)*size]`
- **Signal processing**: Windowing and strided convolution
- **Image processing**: Cropping 2D regions via row/col slices
- **Sequence reversal**: Algorithmic problems requiring reversal
- **Data sampling**: Taking every nth element from a dataset

### Common Mistakes

```python
# Mistake 1: Off-by-one on stop index
text = "Python"
print(text[0:5])   # "Pytho" (not "Python"!)
print(text[0:6])   # "Python" (correct)

# Mistake 2: Slicing creates a copy (for lists)
original = [1, 2, 3]
sliced = original[1:2]
sliced[0] = 99
print(original)  # [1, 2, 3] (unchanged)

# Mistake 3: stop < start with positive step
print(nums[5:2])   # [] (empty!)
print(nums[5:2:-1])  # [5, 4, 3] (use negative step)

# Mistake 4: step=0
# nums[::0]  # ValueError: slice step cannot be zero

# Mistake 5: Slicing beyond bounds is silent
nums = [1, 2, 3]
print(nums[1:100])  # [2, 3] (no error, stops at end)

# Mistake 6: String slice assignment
# s = "hello"
# s[0:1] = "H"  # TypeError: 'str' object does not support item assignment
```

### Best Practices

- Remember stop is exclusive; use negative indices for end-relative access
- Use `[::-1]` for reversing any sequence
- Use `[:]` for shallow copies
- Use slice objects for reusable/computed slices
- Prefer slicing over manual loops for subsequence extraction
- Use slice assignment for efficient list modification
- Combine with `len()` for dynamic end-relative slicing

### Performance Considerations

Slicing is O(k) where k is the length of the result. String slicing creates a new string (O(k) character copy). List and tuple slicing creates a new container copying element references (O(k), shallow). Slice assignment on lists is O(n + k) because elements may need to be shifted. Using `s[:] = iterable` is an efficient way to replace a list's contents without creating a new object.

### Interview Questions

1. What is the syntax for slicing and what do the defaults mean?
2. Does slicing ever raise IndexError?
3. How do you reverse a sequence with slicing?
4. What is the difference between slice assignment on lists (mutable) and strings (immutable)?
5. How does the slice object work?
6. What happens with `s[start:stop:-1]` when start < stop?
7. How do you implement `__getitem__` for a custom class that supports slicing?
8. What is the time complexity of list slicing?

### Coding Challenges

```python
# Challenge 1: Rotate list using slicing
def rotate(lst, k):
    if not lst:
        return lst
    k = k % len(lst)
    return lst[-k:] + lst[:-k]

print(rotate([1, 2, 3, 4, 5], 2))   # [4, 5, 1, 2, 3]
print(rotate([1, 2, 3, 4, 5], -1))  # [2, 3, 4, 5, 1]

# Challenge 2: Extract every nth element
def every_nth(data, n, offset=0):
    return data[offset::n]

print(every_nth([1, 2, 3, 4, 5, 6, 7, 8], 3))      # [1, 4, 7]
print(every_nth([1, 2, 3, 4, 5, 6, 7, 8], 3, 1))   # [2, 5, 8]

# Challenge 3: Get submatrix via slicing
def submatrix(matrix, r_start, r_end, c_start, c_end):
    return [row[c_start:c_end] for row in matrix[r_start:r_end]]

matrix = [[1, 2, 3, 4], [5, 6, 7, 8], [9, 10, 11, 12]]
print(submatrix(matrix, 0, 2, 1, 3))  # [[2, 3], [6, 7]]

# Challenge 4: Sliding window generator
def sliding_window(data, n):
    for i in range(len(data) - n + 1):
        yield data[i:i+n]

for w in sliding_window([1, 2, 3, 4, 5], 3):
    print(w)  # [1,2,3] [2,3,4] [3,4,5]
```

### Related Topics

- Negative indexing
- slice() built-in
- List slice assignment
- __getitem__ and __setitem__
- Sequence protocol
- Shallow vs deep copy

## Negative indices

### What It Is

Negative indices count from the end of a sequence: `-1` is the last element, `-2` is the second-to-last, and so on. In slicing, negative start/stop/step values can create sophisticated extraction patterns, especially when combined with negative step for reversed traversal.

### Why It Is Important

Negative indices eliminate the need for `len(s) - 1` boilerplate, making end-relative access concise and natural. They are essential for extracting suffixes, trailing elements, and implementing reverse-aware slicing.

### How It Works Internally

When Python's `PySlice_AdjustIndices()` processes a slice, negative indices are converted to positive by adding the sequence length. For example, `s[-2:]` becomes `s[len(s)-2:]`. The adjustment logic clamps values to `[0, length]`. For negative step, the default start becomes `-1` (last element) and default stop becomes `0` (before first element), enabling reverse traversal.

### Syntax

```python
s[-1]        # Last element
s[-3:]       # Last 3 elements
s[:-2]       # All but last 2
s[-5:-2]     # From 5th-last to 2nd-last
s[::-1]      # Reverse
s[-2:0:-1]   # From 2nd-last down to (but not including) first
```

### Beginner Examples

```python
text = "Python"
print(text[-1])     # n (last char)
print(text[-2])     # o (second last)
print(text[-3:])    # hon (last 3)
print(text[:-2])    # Pyth (all but last 2)
print(text[-4:-1])  # tho (4th-last to last-1)

nums = [10, 20, 30, 40, 50]
print(nums[-1])     # 50
print(nums[-3:])    # [30, 40, 50]
print(nums[:-2])    # [10, 20, 30]

# Removing suffix
filename = "document.txt"
print(filename[:-4])  # document
```

### Intermediate Examples

```python
# Last N elements
data = list(range(10))
last_three = data[-3:]
print(last_three)  # [7, 8, 9]

# All but last N
all_but_last_two = data[:-2]
print(all_but_last_two)  # [0, 1, 2, 3, 4, 5, 6, 7]

# Middle segment from end
print(data[-7:-2])  # [3, 4, 5, 6, 7]

# Negative step from end
print(data[-2:0:-1])  # [8, 7, 6, 5, 4, 3, 2, 1]

# Skip last N
print(data[:-3])  # [0, 1, 2, 3, 4, 5, 6]

# Practical: URL parsing
url = "https://example.com/page"
domain = url.split("//")[1].split("/")[0]
print(domain)  # example.com

# Extraction from the end
log_line = "2024-01-15 ERROR: Connection failed [42]"
error_code = log_line[-4:-1]
print(error_code)  # [42]

# Negative indexing with step
seq = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print(seq[-1::-2])  # [9, 7, 5, 3, 1] (reversed, every other)
```

### Advanced Examples

```python
# Tail extraction (like Unix tail)
def tail(seq, n=10):
    return seq[-n:] if n > 0 else seq[:]

lines = list(range(100))
print(tail(lines, 3))  # [97, 98, 99]

# Negative index edge cases
empty = []
# print(empty[-1])  # IndexError: list index out of range

# When abs(index) > length
# s = "hi"
# print(s[-10])  # IndexError

# But slicing handles it gracefully:
print([1, 2, 3][-10:])   # [1, 2, 3] (clamped to start)
print([1, 2, 3][:-10])   # [] (clamped before start)

# Negative start with negative step
s = list(range(10))
print(s[-1:-5:-1])   # [9, 8, 7, 6]
print(s[-5:-1:1])    # [5, 6, 7, 8]

# Symmetric padding extraction
def extract_center(seq, window):
    mid = len(seq) // 2
    half = window // 2
    return seq[mid-half:mid+half+1]

print(extract_center(list(range(10)), 3))  # [4, 5, 6]

# Boundary conditions
def safe_last(seq, n=1):
    return seq[-n:] if seq else []

print(safe_last([1, 2, 3], 2))  # [2, 3]
print(safe_last([], 2))         # []

# Nested negative indexing
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
print(matrix[-1][-1])  # 9 (last row, last col)
```

### Real-World Use Cases

- **File extensions**: `filename[-3:]` for extension extraction
- **Log analysis**: `log_line[-50:]` for recent entries
- **Paginated APIs**: `data[-page_size:]` for most recent page
- **Circular buffers**: Using negative indices for wrap-around access
- **URL routing**: Path suffix extraction for routing
- **Data trimming**: Removing first/last N elements from arrays
- **Recent history**: Accessing most recent N items

### Common Mistakes

```python
# Mistake 1: IndexError on empty sequence
# empty = []
# print(empty[-1])  # IndexError
# Check first: if empty: ...

# Mistake 2: Confusing -1 indexing with slicing
s = [1, 2, 3]
print(s[-1])     # 3 (element)
print(s[-1:])    # [3] (list slice)

# Mistake 3: Out-of-bounds negative index on access
# s = [1, 2]
# print(s[-10])  # IndexError (but slicing works!)

# Mistake 4: Negative step with wrong start/stop
s = [0, 1, 2, 3, 4, 5]
print(s[4:1:-1])   # [4, 3, 2] (correct)
print(s[-2:-5:-1]) # [4, 3, 2] (negative indices with negative step)

# Mistake 5: Forgetting that -0 is still 0
s = [1, 2, 3]
print(s[-0])  # 1 (not 3!)
```

### Best Practices

- Use `s[-1]` for last element instead of `s[len(s)-1]`
- Use `s[-n:]` to get the last n elements
- Use `s[:-n]` to drop the last n elements
- Always check for empty sequences before negative indexing (single element)
- For slicing, negative indices beyond bounds are clamped safely
- Combine negative indices with positive step for suffix extraction

### Performance Considerations

Negative index adjustment is O(1) — it's just an addition. There is no performance penalty compared to positive indices. The adjustment happens once during the slice or access operation. Python's internals handle negative indices at the C level with no Python-level overhead.

### Interview Questions

1. How does Python handle negative indices internally?
2. What is the difference between `s[-1]` and `s[-1:]`?
3. What happens when you use an out-of-bounds negative index in a slice vs direct access?
4. How do you get the last N elements of a sequence?
5. How do you remove the last N elements using slicing?
6. Can you use negative step with negative indices?
7. What is `-0` in Python indexing?
8. How do you safely handle negative indexing on empty sequences?

### Coding Challenges

```python
# Challenge 1: Tail function
def tail(seq, n=10):
    return seq[-max(0, n):] if seq else []

print(tail([1, 2, 3, 4, 5], 3))  # [3, 4, 5]
print(tail([1], 5))               # [1]

# Challenge 2: Drop last N
def drop_last(seq, n=1):
    return seq[:-n] if n < len(seq) else type(seq)()

print(drop_last([1, 2, 3, 4, 5], 2))  # [1, 2, 3]

# Challenge 3: Alternating from end
def alternate_from_end(seq):
    return seq[-1::-2]

print(alternate_from_end([1, 2, 3, 4, 5, 6]))  # [6, 4, 2]

# Challenge 4: Safe negative accessor
class SafeSeq:
    def __init__(self, data):
        self.data = data
    def get(self, idx):
        if not self.data:
            raise IndexError("empty sequence")
        return self.data[idx]

s = SafeSeq([10, 20, 30])
print(s.get(-1))  # 30

# Challenge 5: Circular negative index
def circular_get(seq, idx):
    return seq[idx % len(seq)] if seq else None

print(circular_get([1, 2, 3], -1))  # 3
print(circular_get([1, 2, 3], -4))  # 3
```

### Related Topics

- Positive indexing (0-based)
- slice() built-in
- Slice start/stop/step defaults
- Sequence protocol
- IndexError handling
- Circular buffers
