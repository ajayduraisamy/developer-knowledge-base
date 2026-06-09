# Arrays - array() constructor, typecodes, numpy.array()

## Introduction
Arrays in Python are sequences that store elements of a single data type. Unlike lists, which can hold mixed types, arrays are homogeneous — all elements must be the same type. Python provides the built-in `array` module for memory-efficient arrays, and the popular third-party `numpy` library for scientific computing.

## Why It Is Important
- More memory efficient than lists for large homogeneous datasets
- Faster element-wise operations (especially with NumPy)
- Essential for numerical and scientific computing
- Interoperability with C libraries and hardware interfaces
- Foundation for data science and machine learning workflows

## Syntax
```python
import array
arr = array.array(typecode, initializer)

import numpy as np
np_arr = np.array([1, 2, 3], dtype=np.int32)
```

## Examples

### Beginner Examples

**Built-in array module:**
```python
import array

# 'i' = signed integer, 'f' = float, 'd' = double
int_array = array.array('i', [1, 2, 3, 4, 5])
float_array = array.array('f', [1.5, 2.5, 3.5])

print(f"Integer array: {int_array}")
print(f"Float array: {float_array}")
print(f"Type: {type(int_array)}")

# Access elements like lists
print(f"First: {int_array[0]}")
print(f"Last: {int_array[-1]}")

# Modify elements
int_array[0] = 99
print(f"After modification: {int_array}")

# Append and extend
int_array.append(6)
int_array.extend([7, 8])
print(f"After append/extend: {int_array}")

# Common typecodes
# 'b' signed char, 'B' unsigned char
# 'h' signed short, 'H' unsigned short
# 'i' signed int, 'I' unsigned int
# 'l' signed long, 'L' unsigned long
# 'f' float, 'd' double
```

**Creating arrays from different sources:**
```python
import array

# From list
a1 = array.array('i', [10, 20, 30])

# From tuple
a2 = array.array('f', (1.1, 2.2, 3.3))

# From range
a3 = array.array('i', range(10))

# From bytes
a4 = array.array('B', b'hello')

print(f"From list: {a1}")
print(f"From tuple: {a2}")
print(f"From range: {a3}")
print(f"From bytes: {a4}")
```

### Intermediate Examples

**Array operations and methods:**
```python
import array

arr = array.array('i', [3, 1, 4, 1, 5, 9, 2, 6])

# List-like methods work on arrays
arr.append(7)
arr.extend([8, 9])
arr.insert(0, 0)
print(f"After inserts: {arr}")

arr.pop()
arr.remove(1)  # Removes first occurrence
print(f"After pop/remove: {arr}")

# Index and count
print(f"Index of 5: {arr.index(5)}")
print(f"Count of 1: {arr.count(1)}")

# Slicing
print(f"Slice [2:5]: {arr[2:5]}")
print(f"Reverse: {arr[::-1]}")

# Type-specific arrays enforce type
try:
    arr.append(3.14)  # Raises TypeError — int array can't hold float
except TypeError as e:
    print(f"Type error: {e}")

# Convert to list
as_list = arr.tolist()
print(f"As list: {as_list}")

# Convert to bytes
as_bytes = arr.tobytes()
print(f"As bytes: {as_bytes}")

# Convert from bytes
restored = array.array('i')
restored.frombytes(as_bytes)
print(f"Restored from bytes: {restored}")
```

**NumPy arrays (preview):**
```python
import numpy as np

# Create NumPy array
arr = np.array([1, 2, 3, 4, 5])
print(f"NumPy array: {arr}")
print(f"Shape: {arr.shape}")
print(f"dtype: {arr.dtype}")

# Vectorized operations (much faster than list comprehension)
print(f"Multiply by 2: {arr * 2}")
print(f"Square: {arr ** 2}")
print(f"Sum: {arr.sum()}")
print(f"Mean: {arr.mean()}")
print(f"Max: {arr.max()}")

# Multi-dimensional arrays
matrix = np.array([[1, 2, 3], [4, 5, 6]])
print(f"Matrix:\n{matrix}")
print(f"Shape: {matrix.shape}")
print(f"Transpose:\n{matrix.T}")
```

### Advanced Examples

**Memory comparison — list vs array vs NumPy:**
```python
import sys
import array
import numpy as np

size = 10000

# Compare memory usage
py_list = list(range(size))
py_arr = array.array('i', range(size))
np_arr = np.arange(size, dtype=np.int32)

list_size = sys.getsizeof(py_list) + sum(sys.getsizeof(i) for i in py_list)
arr_size = sys.getsizeof(py_arr)
np_size = np_arr.nbytes + sys.getsizeof(np_arr)

print(f"List memory:   {list_size:,} bytes")
print(f"Array memory:  {arr_size:,} bytes")
print(f"NumPy memory:  {np_size:,} bytes")
print(f"Array saves ~{list_size - arr_size:,} bytes vs list")
```

**Working with binary data:**
```python
import array
import struct

# Arrays are ideal for binary protocol handling
packet = array.array('B', [0x00, 0x01, 0x02, 0xFF, 0xFE])

# Parse as different types using memoryview
view = memoryview(packet)
as_short = view.cast('H')  # Interpret as unsigned shorts
print(f"Bytes: {list(packet)}")
print(f"As unsigned shorts: {list(as_short)}")

# Write to binary file
with open('data.bin', 'wb') as f:
    packet.tofile(f)

# Read back from binary file
restored = array.array('B')
with open('data.bin', 'rb') as f:
    restored.fromfile(f, 5)  # Read 5 elements
print(f"Restored from file: {list(restored)}")
```

## Real-World Use Cases
- Audio processing (sample buffers)
- Image processing (pixel data as byte arrays)
- Network packet construction and parsing
- Memory-mapped file I/O
- Scientific computing (NumPy arrays)
- Machine learning datasets
- Database query result buffers

## Common Mistakes

```python
import array

# Mistake 1: Wrong typecode for data
try:
    arr = array.array('i', [1.5, 2.5])  # 'i' doesn't accept floats
except TypeError as e:
    print(f"Mistake 1: {e}")

# Mistake 2: Mixing types in a single array
arr = array.array('i', [1, 2, 3])
# arr.append("hello")  # TypeError — can't add string to int array

# Mistake 3: Forgetting arrays are typed
arr = array.array('i', [1, 2, 3])
# arr[0] = 3.14  # TypeError — truncation or conversion error

# Mistake 4: Not choosing right typecode
# 'b' (signed char) range: -128 to 127
small = array.array('b', [100, 127])
# small.append(128)  # OverflowError
print(f"Mistake 4 prevention: use 'h' for larger ranges")

# Mistake 5: Confusing array.array with numpy.array
# import array as np  # Don't do this!
```

## Best Practices
- Use `array.array` for simple typed sequences where memory matters
- Use NumPy arrays for numerical/matrix operations
- Choose the smallest typecode that fits your data range
- Use `memoryview` for zero-copy operations on array data
- Convert to/from bytes for serialization and network transmission
- Profile before optimizing — lists are often fast enough
- Use `array('B')` for byte-level operations and binary protocols

## Interview Questions

1. What is the difference between a list and an array?
2. What typecodes does the `array` module support?
3. When would you use `array.array` vs `numpy.array`?
4. How do you convert an array to bytes and back?
5. What is the memory advantage of arrays over lists?
6. How do you find the length of an array?
7. Can arrays be multidimensional?

## Coding Challenges

```python
# Challenge 1: Array statistics
import array
import statistics

def array_stats(arr):
    """Calculate basic statistics on a numeric array."""
    return {
        'min': min(arr),
        'max': max(arr),
        'sum': sum(arr),
        'mean': statistics.mean(arr),
        'median': statistics.median(arr),
        'std': statistics.stdev(arr) if len(arr) > 1 else 0
    }

data = array.array('d', [1.5, 2.7, 3.2, 4.1, 5.0])
stats = array_stats(data)
for key, value in stats.items():
    print(f"{key}: {value:.2f}")

# Challenge 2: Byte-level hex dump
def hex_dump(data_bytes, width=16):
    """Create a hex dump of binary data."""
    result = []
    for i in range(0, len(data_bytes), width):
        chunk = data_bytes[i:i + width]
        hex_part = ' '.join(f'{b:02x}' for b in chunk)
        ascii_part = ''.join(chr(b) if 32 <= b < 127 else '.' for b in chunk)
        result.append(f"{i:04x}  {hex_part:<48}  {ascii_part}")
    return '\n'.join(result)

data = array.array('B', range(256))
print(hex_dump(data[:64]))

# Challenge 3: Running average with array
def running_average(values, window):
    """Calculate running average with a sliding window."""
    import array
    result = array.array('d')
    for i in range(len(values) - window + 1):
        avg = sum(values[i:i + window]) / window
        result.append(avg)
    return result

values = array.array('i', [1, 2, 3, 4, 5, 6, 7, 8])
avg = running_average(values, 3)
print(f"Running average (window=3): {list(avg)}")
```

## Summary
Arrays provide type-safe, memory-efficient storage for homogeneous data. Python's built-in `array` module is ideal for simple typed sequences, while NumPy arrays offer powerful vectorized operations for numerical computing. Arrays bridge the gap between Python's dynamic lists and low-level C data structures.

## Related Topics
- 10_lists.md — the more flexible, untyped counterpart
- 04_datatypes.md — type system and numeric types
- 48_file_operations.md — reading/writing binary data
- 95_numba.md — JIT compilation for array operations
