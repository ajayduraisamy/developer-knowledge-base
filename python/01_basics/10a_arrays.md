# Arrays - array() constructor, typecodes, numpy.array()

## Introduction

Arrays in Python provide homogeneous, memory-efficient storage for sequences of elements of a single type. The built-in `array` module bridges the gap between Python's dynamic lists and low-level C arrays, while NumPy arrays offer powerful vectorized operations for numerical computing. Arrays are essential for performance-critical applications, binary data handling, and scientific computing.

## `array.array()` Constructor

### What It Is

`array.array(typecode, initializer)` creates a typed array where all elements share a single C-compatible data type. The `typecode` is a character specifying the element type (e.g., `'i'` for signed int, `'d'` for double). The optional `initializer` can be a list, tuple, range, bytes, or any iterable.

### Why It Is Important

The `array` module provides memory-efficient storage compared to lists (no per-object overhead for each element). It enables direct binary I/O, memory-mapped operations, and interoperability with C libraries. It also enforces type safety — preventing accidental type mixing.

### How It Works Internally

`array_new()` in `arraymodule.c` allocates a `PyArrayObject` containing a contiguous C array of the specified type. The `ob_item` pointer points to a `malloc`'d buffer. `array_ass_slice()` uses `memcpy`/`memmove` for efficient bulk operations. The array tracks its length and allocated capacity separately; it overallocates on growth similarly to lists. Type checking is done at insertion time via `array_ass_item()`.

### Syntax

```python
import array

array.array(typecode)                  # Empty array
array.array(typecode, initializer)     # From iterable
array.array(typecode, bytes_or_buffer) # From bytes/buffer
```

### Beginner Examples

```python
import array

# Creating arrays
ints = array.array('i', [1, 2, 3, 4, 5])
floats = array.array('f', [1.5, 2.5, 3.5])
doubles = array.array('d', [1.1, 2.2, 3.3])
bytes_arr = array.array('B', b'hello')

print(ints)      # array('i', [1, 2, 3, 4, 5])
print(floats)    # array('f', [1.5, 2.5, 3.5])
print(doubles)   # array('d', [1.1, 2.2, 3.3])

# Accessing elements
print(ints[0])       # 1
print(ints[-1])      # 5
print(ints[1:3])     # array('i', [2, 3])

# Modifying
ints[0] = 99
print(ints)          # array('i', [99, 2, 3, 4, 5])

# Appending
ints.append(6)
ints.extend([7, 8])
print(ints)          # array('i', [99, 2, 3, 4, 5, 6, 7, 8])
```

### Intermediate Examples

```python
import array

# Type enforcement
arr = array.array('i', [1, 2, 3])
try:
    arr.append(3.14)  # Wrong type
except TypeError as e:
    print(f"Type error: {e}")

# Convert to/from list
arr = array.array('i', [1, 2, 3])
lst = arr.tolist()
print(lst)  # [1, 2, 3]

# Convert to/from bytes
arr = array.array('i', [1, 2, 3])
data = arr.tobytes()
print(data)  # b'\x01\x00\x00\x00\x02\x00\x00\x00\x03\x00\x00\x00'

restored = array.array('i')
restored.frombytes(data)
print(restored)  # array('i', [1, 2, 3])

# File I/O
arr = array.array('d', [1.0, 2.0, 3.0, 4.0, 5.0])
with open('array_data.bin', 'wb') as f:
    arr.tofile(f)

restored_arr = array.array('d')
with open('array_data.bin', 'rb') as f:
    restored_arr.fromfile(f, 5)
print(restored_arr)  # array('d', [1.0, 2.0, 3.0, 4.0, 5.0])

# Memory efficiency comparison
import sys
lst = list(range(1000))
arr = array.array('i', range(1000))
print(f"List size:  {sys.getsizeof(lst) + sum(sys.getsizeof(i) for i in lst):,} bytes")
print(f"Array size: {sys.getsizeof(arr):,} bytes")
```

### Advanced Examples

```python
import array
import struct

# Memoryview for zero-copy operations
arr = array.array('i', [0x01020304, 0x05060708])
view = memoryview(arr)

# Interpret as bytes
as_bytes = view.cast('B')
print(list(as_bytes))
# [4, 3, 2, 1, 8, 7, 6, 5] (little-endian)

# Interpret as unsigned shorts
as_shorts = view.cast('H')
print(list(as_shorts))
# [0x0304, 0x0102, 0x0708, 0x0506] (little-endian)

# Byte swapping
arr = array.array('i', [1, 2, 3, 4])
arr.byteswap()
print(arr)  # array('i', [16777216, 33554432, 50331648, 67108864])

# Packing/unpacking with struct
packed = struct.pack('iii', 1, 2, 3)
arr = array.array('i')
arr.frombytes(packed)
print(arr)  # array('i', [1, 2, 3])

# Using array as ring buffer
class ArrayRingBuffer:
    def __init__(self, typecode, capacity):
        self._arr = array.array(typecode, [0]) * capacity
        self._capacity = capacity
        self._start = 0
        self._size = 0

    def append(self, value):
        idx = (self._start + self._size) % self._capacity
        self._arr[idx] = value
        if self._size < self._capacity:
            self._size += 1
        else:
            self._start = (self._start + 1) % self._capacity

    def __getitem__(self, index):
        if not -self._size <= index < self._size:
            raise IndexError
        if index < 0:
            index += self._size
        return self._arr[(self._start + index) % self._capacity]

    def __repr__(self):
        return f"ArrayRingBuffer({list(self._arr)})"

buf = ArrayRingBuffer('d', 3)
buf.append(1.0); buf.append(2.0); buf.append(3.0); buf.append(4.0)
print(buf[0], buf[1], buf[2])  # 2.0 3.0 4.0
```

### Real-World Use Cases

- **Binary Protocol Handling**: Building/parsing network packets
- **Audio Processing**: Sample buffers as `'h'` (short) or `'f'` (float) arrays
- **Image Processing**: Pixel data as `'B'` (unsigned byte) arrays
- **Serialization**: Efficient binary serialization for data transfer
- **Memory-Mapped Files**: `mmap` + array for large dataset access
- **Embedded Systems**: Interfacing with hardware registers and C structures

### Common Mistakes

```python
import array

# Mistake 1: Wrong typecode for data
try:
    arr = array.array('i', [1.5, 2.5])  # 'i' doesn't accept floats
except TypeError as e:
    print(f"Mistake 1: {e}")

# Mistake 2: Overflow with small typecode
try:
    arr = array.array('b', [128])  # 'b' range: -128 to 127
except OverflowError as e:
    print(f"Mistake 2: {e}")

# Mistake 3: Using array where list is simpler
# If you don't need type safety or memory efficiency, use a list

# Mistake 4: Confusing array.array with numpy.array
# import array as np  # Don't do this!

# Mistake 5: Forgetting arrays are mutable (unlike bytes)
```

### Best Practices

- Use the smallest typecode that fits your data range
- Use `memoryview` for zero-copy data reinterpretation
- Use `tofile()`/`fromfile()` for efficient binary file I/O
- Use `tolist()` for interoperability with list-based APIs
- Use `byteswap()` when reading data with different endianness
- Profile before choosing array over list — lists are often fast enough
- Prefer `array('B')` for raw byte manipulation (more convenient than `bytes`)

### Performance Considerations

- Arrays store values inline (no per-object overhead) — ~75% memory savings vs lists for `'i'`
- Access time is comparable to lists (O(1) with direct pointer arithmetic)
- Appending has amortized O(1) with overallocation strategy
- `tolist()` creates a new list of Python objects (O(n), memory-heavy)
- `tobytes()` is O(n) but produces a compact representation
- Binary I/O with `tofile()`/`fromfile()` is very fast (C-level write/read)
- Python 3.12+ has improved array methods for small arrays

### Interview Questions

1. What typecodes does the `array` module support?
2. How does `array.array` store data differently from a list?
3. What is the memory advantage of arrays over lists?
4. How do you convert between arrays and bytes?
5. How does `memoryview` work with arrays?
6. What is the difference between `'i'`, `'h'`, and `'l'` typecodes?
7. How do you read an array from a binary file?
8. Can arrays be multidimensional?
9. How does the array module handle endianness?
10. When should you use `array.array` vs `numpy.array` vs a list?

### Coding Challenges

```python
# Challenge 1: Array-based statistical summary
import array
import statistics

def arr_summary(arr):
    return {
        "length": len(arr),
        "min": min(arr),
        "max": max(arr),
        "sum": sum(arr),
        "mean": statistics.mean(arr),
        "median": statistics.median(arr),
        "stdev": statistics.stdev(arr) if len(arr) > 1 else 0.0,
    }

data = array.array('d', [1.5, 2.7, 3.2, 4.1, 5.0])
for k, v in arr_summary(data).items():
    print(f"{k}: {v}")

# Challenge 2: Hex dump utility
def hex_dump(data: array.array, width=16):
    result = []
    for i in range(0, len(data), width):
        chunk = data[i:i + width]
        hex_part = " ".join(f"{b:02x}" for b in chunk)
        ascii_part = "".join(chr(b) if 32 <= b < 127 else "." for b in chunk)
        result.append(f"{i:04x}  {hex_part:<48}  {ascii_part}")
    return "\n".join(result)

data = array.array('B', range(256))
print(hex_dump(data[:64]))

# Challenge 3: Sliding window average
def sliding_avg(values: array.array, window: int):
    result = array.array('d')
    for i in range(len(values) - window + 1):
        avg = sum(values[i:i + window]) / window
        result.append(avg)
    return result

vals = array.array('i', [1, 2, 3, 4, 5, 6, 7, 8])
avgs = sliding_avg(vals, 3)
print(list(avgs))  # [2.0, 3.0, 4.0, 5.0, 6.0, 7.0]
```

### Related Topics

- `struct` Module
- `memoryview` Built-in
- `mmap` Module
- `bytes` and `bytearray`
- `ctypes` Module
- `numpy` (for advanced array operations)

## Typecodes

### What It Is

Typecodes are single-character strings that specify the C data type of elements in an `array.array`. Each typecode maps to a specific C type with known size, signedness, and range. Common typecodes include `'b'`, `'B'`, `'h'`, `'H'`, `'i'`, `'I'`, `'l'`, `'L'`, `'f'`, `'d'`.

### Why It Is Important

Choosing the correct typecode is critical for memory efficiency, preventing overflow, and ensuring correct data representation. The typecode determines storage size (1-8 bytes per element), range of representable values, and binary representation. Using the wrong typecode can cause silent data corruption or overflow errors.

### How It Works Internally

Each typecode maps to a C `typedef` in `arraymodule.c`:
- `'b'` → `signed char` (1 byte)
- `'B'` → `unsigned char` (1 byte)
- `'h'` → `signed short` (2 bytes)
- `'H'` → `unsigned short` (2 bytes)
- `'i'` → `signed int` (4 bytes typically, platform-dependent)
- `'I'` → `unsigned int` (4 bytes)
- `'l'` → `signed long` (4 or 8 bytes, platform-dependent)
- `'L'` → `unsigned long` (4 or 8 bytes)
- `'q'` → `signed long long` (8 bytes, Python 3.3+)
- `'Q'` → `unsigned long long` (8 bytes, Python 3.3+)
- `'f'` → `float` (4 bytes, IEEE 754)
- `'d'` → `double` (8 bytes, IEEE 754)

### Syntax

```python
import array

arr = array.array(typecode_char, [values])
```

| Typecode | C Type | Size | Range |
|----------|--------|------|-------|
| `'b'` | signed char | 1 | -128 to 127 |
| `'B'` | unsigned char | 1 | 0 to 255 |
| `'h'` | signed short | 2 | -32,768 to 32,767 |
| `'H'` | unsigned short | 2 | 0 to 65,535 |
| `'i'` | signed int | 4 | -2^31 to 2^31-1 |
| `'I'` | unsigned int | 4 | 0 to 2^32-1 |
| `'l'` | signed long | 4 or 8 | platform-dependent |
| `'L'` | unsigned long | 4 or 8 | platform-dependent |
| `'q'` | long long | 8 | -2^63 to 2^63-1 |
| `'Q'` | unsigned long long | 8 | 0 to 2^64-1 |
| `'f'` | float | 4 | ~1.2e-38 to 3.4e38 |
| `'d'` | double | 8 | ~2.2e-308 to 1.8e308 |
| `'u'` | wchar_t | 2 or 4 | Unicode (deprecated) |

### Beginner Examples

```python
import array
import sys

# Signed integer types
b_arr = array.array('b', [-128, 0, 127])    # signed char
h_arr = array.array('h', [-32768, 0, 32767])  # signed short
i_arr = array.array('i', [-2**31, 0, 2**31-1]) # signed int

# Unsigned integer types
B_arr = array.array('B', [0, 128, 255])    # unsigned char
H_arr = array.array('H', [0, 32768, 65535])  # unsigned short

# Floating point types
f_arr = array.array('f', [3.14, 2.718, 1.618])  # float (32-bit)
d_arr = array.array('d', [3.14, 2.718, 1.618])  # double (64-bit)

# Size comparison
print(f"'b' size: {b_arr.itemsize} byte")
print(f"'h' size: {h_arr.itemsize} bytes")
print(f"'i' size: {i_arr.itemsize} bytes")
print(f"'f' size: {f_arr.itemsize} bytes")
print(f"'d' size: {d_arr.itemsize} bytes")

# Itemsize determines serialized size
print(f"Memory for 'b' array: {sys.getsizeof(b_arr)} bytes")
print(f"Memory for 'd' array: {sys.getsizeof(d_arr)} bytes")
```

### Intermediate Examples

```python
import array

# Typecode overflow behavior
try:
    arr = array.array('b', [200])  # OverflowError: 'b' range is -128..127
except OverflowError as e:
    print(f"Overflow: {e}")

# 'I' (unsigned int) doesn't accept negative values
try:
    arr = array.array('I', [-1])
except OverflowError as e:
    print(f"Overflow: {e}")

# Precision loss with float vs double
f_arr = array.array('f', [1.23456789012345])
d_arr = array.array('d', [1.23456789012345])
print(f"float (32-bit):  {f_arr[0]:.15f}")   # 1.234567880630493 (precision loss)
print(f"double (64-bit): {d_arr[0]:.15f}")   # 1.234567890123450 (full precision)

# Typecode conversion via bytes
import struct
i_arr = array.array('i', [0x01020304])
# Reinterpret bytes as unsigned char
view = memoryview(i_arr).cast('B')
print(list(view))  # [4, 3, 2, 1] (little-endian on x86)

# Finding the right typecode for a value
def best_typecode(value):
    typecodes = [
        ('b', -128, 127),
        ('B', 0, 255),
        ('h', -32768, 32767),
        ('H', 0, 65535),
        ('i', -2**31, 2**31 - 1),
        ('I', 0, 2**32 - 1),
    ]
    for code, lo, hi in typecodes:
        if isinstance(value, int) and lo <= value <= hi:
            return code
    return 'q' if isinstance(value, int) else 'd'

print(best_typecode(42))     # 'b'
print(best_typecode(200))    # 'B' (unsigned char)
print(best_typecode(40000))  # 'H' (unsigned short)
```

### Advanced Examples

```python
import array
import platform
import struct

# Platform-dependent size of 'l' and 'L'
print(f"Platform: {platform.architecture()}")
print(f"Size of 'l': {array.array('l').itemsize} bytes")
print(f"Size of 'q': {array.array('q').itemsize} bytes")

# Detecting typecode from struct format characters
type_map = {
    'b': 'b', 'B': 'B',
    'h': 'h', 'H': 'H',
    'i': 'i', 'I': 'I',
    'l': 'l', 'L': 'L',
    'q': 'q', 'Q': 'Q',
    'f': 'f', 'd': 'd',
}

# Pack array into struct-compatible bytes
def array_to_struct_format(arr):
    """Return struct format string for the array."""
    size_map = {1: 'b', 2: 'h', 4: 'i', 8: 'q'}
    for arr_code in arr.typecode:
        size = arr.itemsize
        if size in size_map:
            return f"{len(arr)}{size_map[size]}"
    raise ValueError(f"Unsupported size {size}")

# Typecode-aware serialization
class TypedArraySerializer:
    @staticmethod
    def serialize(arr):
        code = arr.typecode.encode()
        data = arr.tobytes()
        return struct.pack('H', len(code)) + code + struct.pack('I', len(data)) + data

    @staticmethod
    def deserialize(buffer):
        offset = 0
        code_len = struct.unpack_from('H', buffer, offset)[0]
        offset += 2
        code = buffer[offset:offset + code_len].decode()
        offset += code_len
        data_len = struct.unpack_from('I', buffer, offset)[0]
        offset += 4
        result = array.array(code)
        result.frombytes(buffer[offset:offset + data_len])
        return result

arr = array.array('d', [1.0, 2.0, 3.0])
serialized = TypedArraySerializer.serialize(arr)
restored = TypedArraySerializer.deserialize(serialized)
print(restored)  # array('d', [1.0, 2.0, 3.0])
print(restored == arr)  # True
```

### Real-World Use Cases

- **Embedded Systems**: Choosing `'B'` or `'H'` for register values (known bit width)
- **Audio**: `'h'` (16-bit signed PCM samples) or `'f'` (32-bit float samples)
- **Image Processing**: `'B'` for grayscale pixel values (0-255)
- **Network Protocols**: `'H'` for 16-bit ports, `'I'` for 32-bit addresses
- **Scientific Data**: `'d'` for high-precision measurements
- **Memory-Constrained Systems**: `'b'` or `'h'` to minimize memory footprint

### Common Mistakes

```python
import array

# Mistake 1: Assuming fixed size for 'l' across platforms
# 'l' is 4 bytes on Windows, 8 bytes on 64-bit Linux
# Use 'i' or 'q' for portable code

# Mistake 2: Overflow with signed/unsigned mismatch
try:
    arr = array.array('B', [-1])  # Unsigned can't hold negative
except OverflowError:
    pass
try:
    arr = array.array('b', [128])  # Signed char max is 127
except OverflowError:
    pass

# Mistake 3: Precision loss with 'f' for large integers
arr = array.array('f', [16777217])  # float can't represent this exactly
print(arr[0])  # 16777216.0 (rounded!)

# Mistake 4: Using 'u' (deprecated, removed in Python 4)
# Use array('B') + encoding or just use a string

# Mistake 5: Not checking itemsize for platform portability
```

### Best Practices

- Use `'i'` (4-byte int) for general-purpose integer arrays (portable, sufficient range)
- Use `'d'` (8-byte double) for general-purpose float arrays
- Use `'B'` for raw bytes (compatible with `bytes` and `bytearray`)
- Use `'q'`/`'Q'` when you need 64-bit integers (portable, unambiguous)
- Use `itemsize` attribute to check actual type size at runtime
- Avoid `'l'`/`'L'` in cross-platform code (platform-dependent size)
- Test overflow behavior on the target platform

### Performance Considerations

- Smaller typecodes = less memory = better cache utilization
- `'b'` arrays are fastest for memory bandwidth-bound operations
- `'d'` operations may be slower than `'f'` on some hardware (memory bandwidth)
- `'i'` is typically the native word size and fastest for integer operations
- Type conversion at insertion time adds overhead — pre-convert values
- Memoryview casting avoids copies but depends on hardware endianness
- Aligned access (native sizes) is faster than misaligned on some architectures

### Interview Questions

1. What typecodes does the `array` module support?
2. How do you choose the right typecode for your data?
3. What is the difference between `'b'` and `'B'`?
4. Why is `'l'` typecode size platform-dependent?
5. What happens when you insert a value outside the typecode's range?
6. How does `itemsize` relate to memory layout?
7. How do you convert an array from one typecode to another?
8. What is the precision difference between `'f'` and `'d'`?
9. How does endianness affect array serialization?
10. What typecode should you use for cross-platform data exchange?

### Coding Challenges

```python
# Challenge 1: Auto-detect optimal typecode
import array
import math

def optimal_typecode(values):
    """Find the smallest typecode that can hold all values."""
    if all(isinstance(v, int) for v in values):
        min_v, max_v = min(values), max(values)
        if max_v <= 255 and min_v >= 0:
            return 'B'
        if max_v <= 127 and min_v >= -128:
            return 'b'
        if max_v <= 65535 and min_v >= 0:
            return 'H'
        if max_v <= 32767 and min_v >= -32768:
            return 'h'
        if max_v <= 2**32 - 1 and min_v >= 0:
            return 'I'
        if max_v <= 2**31 - 1 and min_v >= -2**31:
            return 'i'
        return 'q'
    else:
        return 'd'

test = [10, 20, 30]
code = optimal_typecode(test)
arr = array.array(code, test)
print(f"Optimal typecode for {test}: '{code}' ({arr.itemsize} bytes/elem)")

# Challenge 2: Cross-platform array serialization
def serialize_portable(arr: array.array) -> bytes:
    """Serialize array with endianness marker."""
    import struct
    endian = '<' if struct.pack('H', 1)[0] == 1 else '>'
    header = endian.encode() + arr.typecode.encode()
    return header + arr.tobytes()

def deserialize_portable(data: bytes) -> array.array:
    endian = data[0:1].decode()
    typecode = data[1:2].decode()
    arr = array.array(typecode)
    if (endian == '<' and struct.pack('H', 1)[0] != 1) or \
       (endian == '>' and struct.pack('H', 1)[0] == 1):
        arr.frombytes(data[2:])
        arr.byteswap()
    else:
        arr.frombytes(data[2:])
    return arr

# Challenge 3: Memory usage calculator
def array_memory_estimate(typecode, length):
    """Estimate total memory for an array of given type and length."""
    arr = array.array(typecode, [0]) * length
    return {
        'typecode': typecode,
        'length': length,
        'itemsize': arr.itemsize,
        'element_bytes': length * arr.itemsize,
        'overhead_bytes': arr.__sizeof__() - length * arr.itemsize,
        'total_bytes': arr.__sizeof__(),
    }

for code in ['b', 'h', 'i', 'q', 'f', 'd']:
    info = array_memory_estimate(code, 1000)
    print(f"'{code}': {info['total_bytes']:,} bytes for {info['length']} elements")
```

### Related Topics

- C Data Types
- Struct Module
- Platform Module
- Endianness
- `memoryview`
- `ctypes` and C Interop

## `numpy.array()`

### What It Is

`numpy.array()` creates an ndarray (n-dimensional array) from any array-like input. NumPy arrays support homogeneous typed data with optional shape, strides, and memory layout control. They are the foundation of scientific computing in Python.

### Why It Is Important

NumPy arrays provide vectorized operations that execute at C speed, significantly outperforming Python loops for numerical computations. They support broadcasting, slicing, reshaping, linear algebra, FFT, random number generation, and seamless integration with C/Fortran libraries. NumPy is the backbone of the Python data science ecosystem.

### How It Works Internally

`numpy.core.multiarray.array()` creates a `PyArrayObject` containing:
- A pointer to a contiguous (or strided) C array of data
- Shape tuple (dimensions)
- Strides tuple (bytes between elements along each axis)
- Dtype descriptor (element type, byte order, alignment)
- Flags (C-contiguous, Fortran-contiguous, writable, etc.)

Data is stored in a single `malloc`'d block (for contiguous arrays) using the buffer protocol. Vectorized operations use SIMD instructions where available (NumPy 1.20+ uses SIMD intrinsics extensively). Operations like `a + b` dispatch to C-level ufuncs (universal functions) that iterate with SIMD-optimized inner loops.

### Syntax

```python
import numpy as np

np.array(object, dtype=None, copy=True, order='K', ndmin=0)
np.zeros(shape, dtype=float)
np.ones(shape, dtype=float)
np.empty(shape, dtype=float)
np.arange(start, stop, step, dtype=None)
np.linspace(start, stop, num, dtype=None)
```

### Beginner Examples

```python
import numpy as np

# Creating arrays
arr = np.array([1, 2, 3, 4, 5])
print(arr)         # [1 2 3 4 5]
print(arr.shape)   # (5,)
print(arr.dtype)   # int64 (or int32 on some platforms)
print(arr.ndim)    # 1

# Multi-dimensional
matrix = np.array([[1, 2, 3], [4, 5, 6]])
print(matrix)
print(matrix.shape)  # (2, 3)

# Special arrays
zeros = np.zeros((2, 3))
ones = np.ones((3,))
identity = np.eye(3)
rng = np.arange(0, 10, 2)  # [0, 2, 4, 6, 8]
lin = np.linspace(0, 1, 5)  # [0.0, 0.25, 0.5, 0.75, 1.0]

# Vectorized operations
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
print(a + b)   # [5 7 9]
print(a * 2)   # [2 4 6]
print(a ** 2)  # [1 4 9]
print(np.sqrt(a))  # [1.0 1.414 1.732]

# Indexing and slicing
print(arr[0])       # 1
print(arr[-1])      # 5
print(arr[1:4])     # [2 3 4]
print(matrix[0, 1]) # 2
print(matrix[:, 1]) # [2 5] (second column)
```

### Intermediate Examples

```python
import numpy as np

# Broadcasting
a = np.array([[1, 2, 3], [4, 5, 6]])
b = np.array([10, 20, 30])
print(a + b)  # [[11 22 33], [14 25 36]]  — b broadcast across rows

# Boolean masking
arr = np.array([1, 2, 3, 4, 5, 6])
mask = arr > 3
print(mask)        # [False False False  True  True  True]
print(arr[mask])   # [4 5 6]
print(arr[arr % 2 == 0])  # [2 4 6]

# Reshaping
arr = np.arange(12)
reshaped = arr.reshape(3, 4)
print(reshaped)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

# Flattening/Raveling
print(reshaped.flatten())  # Copy
print(reshaped.ravel())    # View (if possible)

# Aggregations
data = np.random.randn(1000)
print(data.mean())   # ~0.0
print(data.std())    # ~1.0
print(data.min())    # minimum
print(data.max())    # maximum
print(data.sum())    # sum

# Linear algebra
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])
print(np.dot(A, B))       # Matrix multiplication
print(A @ B)              # Same (Python 3.5+)
print(np.linalg.inv(A))   # Inverse
print(np.linalg.eig(A))   # Eigenvalues/vectors

# dtype control
arr = np.array([1, 2, 3], dtype=np.float32)
print(arr.dtype)   # float32
print(arr.itemsize)  # 4
```

### Advanced Examples

```python
import numpy as np

# Strides and memory layout
arr = np.arange(12).reshape(3, 4)
print(f"Strides: {arr.strides}")  # (32, 8) for float64
print(f"C-contiguous: {arr.flags.c_contiguous}")
print(f"F-contiguous: {arr.flags.f_contiguous}")

# Fortran-order (column-major)
arr_f = np.asfortranarray(arr)
print(f"F-contiguous: {arr_f.flags.f_contiguous}")

# Memory views (no copy)
view = arr[:2, :2]
view[0, 0] = 99
print(arr)  # Modified!

# Copy creation
copy = arr[:2, :2].copy()
copy[0, 0] = -1
print(arr)  # Not modified

# Advanced indexing (fancy indexing)
indices = np.array([0, 2, 3])
print(arr[indices])  # Rows 0, 2, 3

# Where clause
x = np.array([1, 2, 3, 4, 5])
result = np.where(x > 3, x * 10, x * -1)
print(result)  # [-1 -2 -3 40 50]

# Vectorized string operations
names = np.array(["alice", "bob", "charlie"])
print(np.char.upper(names))     # ['ALICE' 'BOB' 'CHARLIE']
print(np.char.capitalize(names))  # ['Alice' 'Bob' 'Charlie']

# Structured arrays
dtype = [('name', 'U10'), ('age', 'i4'), ('salary', 'f8')]
data = np.array([
    ('Alice', 30, 75000.0),
    ('Bob', 25, 62000.0),
], dtype=dtype)
print(data['name'])   # ['Alice' 'Bob']
print(data[data['age'] > 28])  # Filter

# Ufuncs and reduction
def my_ufunc(x):
    return np.sin(x) + np.cos(x)

arr = np.linspace(0, np.pi, 1000)
result = my_ufunc(arr)
print(f"Max: {result.max():.4f}")  # Vectorized!

# Out-of-core processing with memmap
fp = np.memmap('large_data.dat', dtype='float64', mode='w+', shape=(1000, 1000))
fp[:, :] = np.random.randn(1000, 1000)  # Written to disk
fp.flush()
```

### Real-World Use Cases

- **Machine Learning**: Feature matrices, model weights, training data (default tensor type in sklearn, PyTorch)
- **Image Processing**: Multi-dimensional pixel arrays (height × width × channels)
- **Signal Processing**: Time-series data, FFT, filtering
- **Scientific Simulation**: Grid-based computations (fluid dynamics, weather modeling)
- **Financial Analysis**: Time-series price data, covariance matrices, portfolio optimization
- **Computer Graphics**: Transformation matrices, vertex data, texture arrays

### Common Mistakes

```python
import numpy as np

# Mistake 1: Confusing copy vs view
arr = np.array([1, 2, 3, 4, 5])
slice_view = arr[1:4]  # View, not copy
slice_view[0] = 99
print(arr)  # [1 99 3 4 5] — original modified!

# Mistake 2: Assuming Python list semantics for comparisons
a = np.array([1, 2, 3])
b = np.array([1, 2, 3])
# if a == b:  # ValueError! (element-wise comparison)
print(np.array_equal(a, b))  # True

# Mistake 3: Modifying array while iterating
for val in np.nditer(arr, op_flags=['readwrite']):
    val[...] = val * 2  # Must use [...] for in-place modification

# Mistake 4: Integer overflow
arr = np.array([2**62], dtype=np.int64)
# print(arr * 2)  # Overflow warning, wraps around

# Mistake 5: Slow Python loops instead of vectorized operations
# Bad:
result = np.zeros_like(arr)
for i in range(len(arr)):
    result[i] = np.sin(arr[i])  # Slow!
# Good:
result = np.sin(arr)  # Fast (vectorized)
```

### Best Practices

- Use vectorized operations instead of Python loops for numerical code
- Use `dtype=np.float32` for memory-constrained applications
- Use `np.memmap` for datasets larger than RAM
- Use `np.savez`/`np.load` for efficient array persistence
- Use `out=` parameter in ufuncs to avoid intermediate allocations
- Use `view()` cautiously — know when you're sharing memory
- Use `np.einsum` for complex tensor contractions
- Profile with `%timeit` to compare vectorized vs loop performance
- Use `np.random.default_rng()` (NumPy 1.17+) instead of legacy `np.random`

### Performance Considerations

- Vectorized operations are 10-100x faster than Python loops (C-level execution)
- Memory layout matters — C-contiguous is fastest for row-wise operations
- NumPy 1.22+ uses SIMD for many ufuncs (add, multiply, sqrt, etc.)
- Broadcasting avoids data copies but has computational overhead
- `@` operator (matmul) uses BLAS libraries (MKL, OpenBLAS) for near-peak performance
- Fancy indexing creates a copy (not a view) and is slower than slicing
- `np.empty` vs `np.zeros` — `empty` is faster but contains garbage values
- First call to any NumPy function includes import overhead (~0.1-0.5s)

### Interview Questions

1. What is the difference between a NumPy array and a Python list?
2. How does broadcasting work in NumPy?
3. What is the difference between `view` and `copy` in NumPy?
4. How do strides work in NumPy arrays?
5. How does NumPy implement vectorized operations internally?
6. What is the difference between C-contiguous and Fortran-contiguous order?
7. How does NumPy handle data types (dtypes)?
8. What is a ufunc and how does it work?
9. How does NumPy compare to `array.array`?
10. How do you handle large datasets that don't fit in RAM with NumPy?

### Coding Challenges

```python
# Challenge 1: Matrix operations without loops
def matrix_operations(A, B):
    return {
        "sum": A + B,
        "product": A @ B,
        "element_multiply": A * B,
        "transpose": A.T,
        "determinant_A": np.linalg.det(A),
    }

A = np.array([[1, 2], [3, 4]], dtype=float)
B = np.array([[5, 6], [7, 8]], dtype=float)
ops = matrix_operations(A, B)
for k, v in ops.items():
    print(f"{k}:\n{v}\n")

# Challenge 2: Image processing with NumPy
def rgb_to_grayscale(img):
    """Convert RGB image to grayscale using broadcasting."""
    weights = np.array([0.2989, 0.5870, 0.1140])
    return np.dot(img[..., :3], weights)

# Simulate RGB image
img = np.random.randint(0, 256, (100, 100, 3), dtype=np.uint8)
gray = rgb_to_grayscale(img)
print(f"Input shape: {img.shape}, Output shape: {gray.shape}")

# Challenge 3: Custom rolling window
def rolling_window(arr, window):
    """Create rolling window view of 1D array (no copy)."""
    shape = (arr.shape[0] - window + 1, window)
    strides = (arr.strides[0], arr.strides[0])
    return np.lib.stride_tricks.as_strided(arr, shape=shape, strides=strides)

data = np.array([1, 2, 3, 4, 5, 6])
windows = rolling_window(data, 3)
print(windows)
# [[1 2 3]
#  [2 3 4]
#  [3 4 5]
#  [4 5 6]]
print(windows.mean(axis=1))  # [2. 3. 4. 5.]

# Challenge 4: Vectorized string processing
def vectorize_text(names, prefix="Mr/Ms"):
    """Vectorized string operations on NumPy string arrays."""
    names = np.char.strip(names)
    names = np.char.capitalize(names)
    return np.char.add(f"{prefix} ", names)

names = np.array([" alice ", "BOB", " charlie "])
print(vectorize_text(names, "Dr."))
# ['Dr. Alice' 'Dr. Bob' 'Dr. Charlie']
```

### Related Topics

- `array.array` module
- Scientific Python Ecosystem (SciPy, pandas, matplotlib)
- BLAS and LAPACK Libraries
- SIMD Instructions
- `__array_interface__` Protocol
- `__buffer__` Protocol
- `numba` JIT Compilation
- `cupy` GPU Arrays
