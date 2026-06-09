# Slicing - sequence[start:stop:step], negative indices

## Introduction

Slicing is a powerful feature in Python that allows you to extract portions of sequences (strings, lists, tuples, bytes, etc.) using the `[start:stop:step]` syntax. Slicing creates a new sequence containing the specified elements without modifying the original. It supports negative indexing and can reverse sequences.

## Why It Is Important

Slicing is important because:
- It provides concise syntax for extracting subsequences
- It supports negative indexing for accessing from the end
- It enables reversing sequences easily
- It works on strings, lists, tuples, and other sequences
- It is more readable and efficient than manual loops
- The `slice()` object allows programmatic slicing

## Syntax

```python
# Basic slicing
sequence[start:stop]     # Elements from start to stop-1
sequence[start:]         # Elements from start to end
sequence[:stop]          # Elements from beginning to stop-1
sequence[:]              # Full copy (shallow)

# With step
sequence[start:stop:step]  # Every step-th element

# Negative indices
sequence[-1]              # Last element
sequence[-3:-1]           # Elements from third-last to second-last

# Reverse
sequence[::-1]            # Reversed copy

# Slice object
s = slice(start, stop, step)
sequence[s]
```

## Examples

### String Slicing

```python
text = "Python Programming"
print(f"String: '{text}'")
print(f"Length: {len(text)}")

# Basic slicing
print(f"text[0:6]: '{text[0:6]}'")      # Python
print(f"text[7:]: '{text[7:]}'")        # Programming
print(f"text[:6]: '{text[:6]}'")        # Python
print(f"text[:]: '{text[:]}'")          # Full copy

# With step
print(f"text[::2]: '{text[::2]}'")      # Pto rgamn (every other char)
print(f"text[1::2]: '{text[1::2]}'")    # yhnPormig

# Reverse
print(f"text[::-1]: '{text[::-1]}'")    # gnimmargorP nohtyP

# Negative indexing
print(f"text[-1]: '{text[-1]}'")        # g
print(f"text[-11:]: '{text[-11:]}'")    # Programming
print(f"text[:-7]: '{text[:-7]}'")      # Python
```

### List Slicing

```python
numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print(f"Original: {numbers}")

# Basic slicing
print(f"numbers[2:5]: {numbers[2:5]}")     # [2, 3, 4]
print(f"numbers[:5]: {numbers[:5]}")       # [0, 1, 2, 3, 4]
print(f"numbers[5:]: {numbers[5:]}")       # [5, 6, 7, 8, 9]
print(f"numbers[:]: {numbers[:]}")         # Full copy

# With step
print(f"numbers[::2]: {numbers[::2]}")     # [0, 2, 4, 6, 8]
print(f"numbers[1::2]: {numbers[1::2]}")   # [1, 3, 5, 7, 9]
print(f"numbers[2:8:3]: {numbers[2:8:3]}") # [2, 5]

# Reverse
print(f"numbers[::-1]: {numbers[::-1]}")   # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]

# Negative step from middle
print(f"numbers[7:2:-1]: {numbers[7:2:-1]}")  # [7, 6, 5, 4, 3]
```

## Beginner Examples

```python
# Extracting substrings
email = "user@example.com"

# Get username (everything before @)
username = email[:email.index("@")]
print(f"Username: {username}")

# Get domain (everything after @)
domain = email[email.index("@") + 1:]
print(f"Domain: {domain}")

# Extract parts of a date
date = "2024-01-15"
year = date[:4]
month = date[5:7]
day = date[8:]
print(f"Year: {year}, Month: {month}, Day: {day}")

# Get first/last N characters
word = "PythonProgramming"
print(f"First 6: {word[:6]}")     # Python
print(f"Last 11: {word[-11:]}")   # Programming

# Skip first and last
print(f"Middle: {word[1:-1]}")    # ythonProgrammin

# Every other element
items = [10, 20, 30, 40, 50, 60]
print(f"Even indices: {items[::2]}")   # [10, 30, 50]
print(f"Odd indices: {items[1::2]}")   # [20, 40, 60]

# Using slicing to split a list
data = [1, 2, 3, 4, 5, 6, 7, 8]
mid = len(data) // 2
first_half = data[:mid]
second_half = data[mid:]
print(f"First half: {first_half}")
print(f"Second half: {second_half}")
```

## Intermediate Examples

### Slicing with Variables

```python
text = "Hello, World!"

# Dynamic slicing
start = 7
end = 12
print(f"text[{start}:{end}]: '{text[start:end]}'")  # World

# Slicing with computed positions
sentence = "The quick brown fox"
words = sentence.split()
word_lengths = [len(w) for w in words]
positions = []
pos = 0
for length in word_lengths:
    positions.append((pos, pos + length))
    pos += length + 1  # +1 for space

for start, end in positions:
    print(f"  word: '{sentence[start:end]}'")
```

### Slice Assignment (Mutable Sequences)

```python
# List slice assignment
numbers = [1, 2, 3, 4, 5]
print(f"Original: {numbers}")

# Replace a slice
numbers[1:3] = [20, 30]
print(f"After replacement: {numbers}")

# Replace with different length
numbers[1:3] = [100, 200, 300, 400]
print(f"After longer replacement: {numbers}")

# Delete a slice
numbers[1:3] = []
print(f"After deletion: {numbers}")

# Insert without removing
numbers[1:1] = [50, 60]  # Insert at index 1
print(f"After insertion: {numbers}")

# Extend at end
numbers[len(numbers):] = [999]
print(f"After extend: {numbers}")
```

### Slice Object

```python
# Creating slice objects
s = slice(2, 8, 2)
numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print(f"numbers[{s.start}:{s.stop}:{s.step}]: {numbers[s]}")

# Slice object attributes
print(f"start: {s.start}")
print(f"stop: {s.stop}")
print(f"step: {s.step}")

# Using slice for data extraction
def extract_sections(data, section_size):
    """Extract sections of equal size from data."""
    sections = []
    for i in range(0, len(data), section_size):
        section = data[i:i + section_size]
        sections.append(section)
    return sections

data = list(range(20))
sections = extract_sections(data, 5)
for i, section in enumerate(sections):
    print(f"Section {i}: {section}")
```

## Advanced Examples

### Custom Class with Slicing Support

```python
class CustomList:
    """A custom class that supports slicing."""
    def __init__(self, items):
        self._items = list(items)
    
    def __getitem__(self, index):
        if isinstance(index, slice):
            return CustomList(self._items[index])
        return self._items[index]
    
    def __setitem__(self, index, value):
        if isinstance(index, slice):
            self._items[index] = list(value)
        else:
            self._items[index] = value
    
    def __repr__(self):
        return f"CustomList({self._items})"
    
    def __len__(self):
        return len(self._items)

cl = CustomList([1, 2, 3, 4, 5, 6, 7, 8])
print(f"Original: {cl}")
print(f"cl[2:5]: {cl[2:5]}")
print(f"cl[::-1]: {cl[::-1]}")
cl[1:4] = [20, 30, 40]
print(f"After assignment: {cl}")
```

### Advanced Slicing Patterns

```python
# Palindrome check with slicing
def is_palindrome(s):
    s = s.lower().replace(" ", "")
    return s == s[::-1]

print(f"racecar: {is_palindrome('racecar')}")
print(f"A man a plan a canal Panama: {is_palindrome('A man a plan a canal Panama')}")

# Rotating a list with slicing
def rotate_left(lst, k):
    if not lst:
        return lst
    k = k % len(lst)
    return lst[k:] + lst[:k]

def rotate_right(lst, k):
    if not lst:
        return lst
    k = k % len(lst)
    return lst[-k:] + lst[:-k]

nums = [1, 2, 3, 4, 5]
print(f"Original: {nums}")
print(f"Rotate left by 2: {rotate_left(nums, 2)}")
print(f"Rotate right by 2: {rotate_right(nums, 2)}")

# Sliding window with slicing
def sliding_window(data, window_size):
    """Generate all windows of given size."""
    for i in range(len(data) - window_size + 1):
        yield data[i:i + window_size]

data = [1, 2, 3, 4, 5, 6]
print("Sliding windows of size 3:")
for window in sliding_window(data, 3):
    print(f"  {window}")

# Chunking with slicing
def chunk_list(lst, chunk_size):
    """Split a list into chunks."""
    return [lst[i:i + chunk_size] for i in range(0, len(lst), chunk_size)]

data = list(range(10))
print(f"Chunks of 3: {chunk_list(data, 3)}")

# Strided convolution-like access
matrix = [
    [1, 2, 3, 4],
    [5, 6, 7, 8],
    [9, 10, 11, 12],
    [13, 14, 15, 16]
]

# Extract 2x2 submatrix
row_slice = slice(1, 3)
col_slice = slice(1, 3)
submatrix = [row[col_slice] for row in matrix[row_slice]]
print(f"2x2 submatrix: {submatrix}")
```

## Real-World Use Cases

- **Data Processing**: Extracting specific columns from datasets
- **String Parsing**: Extracting substrings from formatted text
- **Sequence Reversal**: Reversing lists, strings for algorithms
- **Data Sampling**: Taking every nth element from a dataset
- **Signal Processing**: Windowing, strided convolution
- **Image Processing**: Cropping, region-of-interest extraction
- **Paginated Results**: Slicing API results into pages

## Common Mistakes

```python
# Mistake 1: Off-by-one errors with stop index
text = "Python"
print(text[0:5])   # "Pytho" (NOT "Python"!)
print(text[0:6])   # "Python" (correct)

# Mistake 2: Forgetting that slicing creates a copy
original = [1, 2, 3]
sliced = original[1:2]  # Creates a new list
sliced[0] = 99
print(original)  # [1, 2, 3] (unchanged)

# Mistake 3: Slicing with stop < start and positive step
nums = [0, 1, 2, 3, 4, 5]
print(nums[5:2])   # [] (empty - need negative step)
print(nums[5:2:-1])  # [5, 4, 3]

# Mistake 4: Using step=0
# nums[::0]  # ValueError: slice step cannot be zero

# Mistake 5: Modifying string via slice assignment
# s = "hello"
# s[0] = "H"  # TypeError: 'str' object does not support item assignment
# s[0:1] = "H"  # Also TypeError

# Mistake 6: Confusing slice bounds
text = "hello"
print(text[-4:4])  # 'ell' (from index -4 to index 3)

# Mistake 7: Slicing beyond bounds doesn't error
nums = [1, 2, 3]
print(nums[1:100])  # [2, 3] (no IndexError, just stops at end)
```

## Best Practices

- Use negative indexing for accessing from the end
- Use `[::-1]` for reversing sequences
- Use slice objects when programmatically building slices
- Remember stop index is exclusive
- Use `[:]` for shallow copies of sequences
- Use slicing for readable subsequence extraction
- Avoid modifying mutable sequences while slicing them
- Use `step` for sampling every nth element
- Use slice assignment for efficient list modifications
- Combine slicing with `len()` for dynamic extraction

## Interview Questions

1. What is the syntax for slicing in Python?
2. How does negative indexing work in slicing?
3. How do you reverse a string or list using slicing?
4. What is the difference between `list[1:5]` and `list[1:5:2]`?
5. Can you slice a tuple? What about a string?
6. How does slice assignment work on lists?
7. What is a slice object and how do you use it?
8. What happens if `start` > `stop` with a positive step?
9. How do you create a shallow copy of a list using slicing?
10. Can slicing raise an IndexError?

## Coding Challenges

```python
# Challenge 1: Custom substring function
def substring(text, start, end=None, step=1):
    """Extract substring with default end."""
    if end is None:
        end = len(text)
    return text[start:end:step]

print(substring("Python Programming", 0, 6))    # Python
print(substring("Python Programming", 7))        # Programming
print(substring("Python Programming", ::-1))     # Reversed

# Challenge 2: Rotate list using slicing
def rotate(lst, k):
    if not lst:
        return lst
    k = k % len(lst)
    if k < 0:
        k += len(lst)
    return lst[-k:] + lst[:-k]

print(rotate([1, 2, 3, 4, 5], 2))   # [4, 5, 1, 2, 3]
print(rotate([1, 2, 3, 4, 5], -1))  # [2, 3, 4, 5, 1]

# Challenge 3: Extract every nth element
def every_nth(data, n, offset=0):
    return data[offset::n]

print(every_nth([1, 2, 3, 4, 5, 6, 7, 8], 3))       # [1, 4, 7]
print(every_nth([1, 2, 3, 4, 5, 6, 7, 8], 3, 1))    # [2, 5, 8]

# Challenge 4: Windowed average
def moving_average(data, window_size):
    return [sum(data[i:i+window_size])/window_size 
            for i in range(len(data)-window_size+1)]

prices = [10, 12, 13, 15, 18, 20, 22, 25]
averages = moving_average(prices, 3)
print(f"Prices: {prices}")
print(f"3-day averages: {[f'{a:.1f}' for a in averages]}")

# Challenge 5: Matrix slicing
def get_submatrix(matrix, row_start, row_end, col_start, col_end):
    return [row[col_start:col_end] for row in matrix[row_start:row_end]]

matrix = [
    [1, 2, 3, 4],
    [5, 6, 7, 8],
    [9, 10, 11, 12],
    [13, 14, 15, 16]
]
sub = get_submatrix(matrix, 1, 3, 1, 3)
print(f"Submatrix: {sub}")  # [[6, 7], [10, 11]]
```

## Summary

Slicing provides a concise syntax for extracting portions of sequences using `[start:stop:step]`. It supports negative indexing, defaults for omitted values, and works on strings, lists, tuples, and other sequences. Slicing creates new copies for immutable types and can be used for assignment on mutable types. Understanding slicing is essential for efficient Python programming.

## Related Topics

- Strings (Indexing and Slicing)
- Lists (Indexing and Slicing)
- Tuples
- Sequence Types
- Slice Objects
- Negative Indexing
- List Comprehensions
