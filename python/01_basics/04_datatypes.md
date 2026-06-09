# Data Types - int, float, complex, str, bool, NoneType

## Introduction

Python provides six fundamental built-in data types: integers (`int`), floating-point numbers (`float`), complex numbers (`complex`), strings (`str`), booleans (`bool`), and the null type (`NoneType`). Each type has specific properties regarding mutability, precision, memory layout, and supported operations. Understanding these types is essential because Python's dynamic typing means operations depend entirely on the types of the operands at runtime.

## int

### What It Is
`int` represents whole numbers with arbitrary precision. Unlike many languages where integers overflow at 2^31 or 2^63, Python integers can grow as large as memory permits. Python 3 unified `int` and `long` from Python 2 into a single `int` type.

### Why It Is Important
Arbitrary-precision integers enable cryptography, big integer arithmetic, and exact financial calculations without overflow concerns. Python's `int` is used for indexing, counting, and as the default numeric type for whole numbers.

### How It Works Internally
CPython's `int` is implemented via `PyLongObject` in `Objects/longobject.c`. Small integers (-5 to 257) are pre-allocated and cached (singleton objects). Larger integers use a sign-magnitude representation with an array of 30-bit "digits" (on 64-bit systems). Operations on small ints avoid allocation by using cached objects; operations on large ints allocate new objects. The `PyLongObject` is variable-length, with `ob_size` indicating the number of digits.

### Syntax
```python
# Integer literals
a = 42            # Decimal
b = 0x2A          # Hexadecimal
c = 0o52          # Octal
d = 0b101010      # Binary
e = 1_000_000     # Underscore separators (Python 3.6+)

# Type conversion
f = int(3.14)     # 3 (truncates toward zero)
g = int("42")     # 42 (base 10 by default)
h = int("FF", 16) # 255 (explicit base)
```

### Beginner Examples
```python
# Basic integer operations
a, b = 15, 4
print(f"Addition: {a + b}")       # 19
print(f"Subtraction: {a - b}")    # 11
print(f"Multiplication: {a * b}") # 60
print(f"Division: {a / b}")       # 3.75 (float!)
print(f"Floor division: {a // b}")# 3
print(f"Modulus: {a % b}")        # 3
print(f"Power: {a ** b}")         # 50625

# Integer size
import sys
small = 42
large = 10 ** 100
print(f"Small int size: {sys.getsizeof(small)} bytes")
print(f"Large int size: {sys.getsizeof(large)} bytes")
print(f"Large int bit length: {large.bit_length()}")
```

### Intermediate Examples
```python
# Arbitrary precision
factorial_100 = 1
for i in range(1, 101):
    factorial_100 *= i
print(f"100! has {len(str(factorial_100))} digits")

# Base conversion
num = 255
print(bin(num))   # 0b11111111
print(oct(num))   # 0o377
print(hex(num))   # 0xff

# Integer methods
print((42).bit_length())          # 6 (binary: 101010)
print((255).bit_count())          # 8 (number of 1 bits)
print((10).to_bytes(4, "big"))    # b'\x00\x00\x00\n'
print(int.from_bytes(b'\x00\n', "big"))  # 10

# Negative integer division
print(f"-7 // 3 = {-7 // 3}")     # -3 (floor division)
print(f"-7 % 3 = {-7 % 3}")       # 2 (modulus always non-negative)
```

### Advanced Examples
```python
# Integer caching and identity
a, b = 256, 256
print(f"256 is 256: {a is b}")  # True (cached)

c, d = 257, 257
print(f"257 is 257: {c is d}")  # False (not cached) — CPython implementation detail

# Variable-length integer internals
import sys

def analyze_int(n: int):
    info = {
        "value": n,
        "size_bytes": sys.getsizeof(n),
        "bit_length": n.bit_length(),
        "is_zero": n == 0,
        "is_negative": n < 0,
        "is_finite": True,
    }
    if n.bit_length() <= 64:
        info["hex"] = hex(n)
    return info

print(analyze_int(2**1000))

# Using int as a bitset
flags = 0
flags |= 1 << 0   # Set bit 0
flags |= 1 << 3   # Set bit 3
print(f"Flags: {flags:08b}")  # 00001001
print(f"Bit 0: {bool(flags & (1 << 0))}")  # True
print(f"Bit 1: {bool(flags & (1 << 1))}")  # False
```

### Real-World Use Cases
- **Database IDs**: Primary keys requiring arbitrary precision
- **Cryptography**: RSA, ECC require large prime integers
- **Bit Flags**: Permission systems, configuration bitmasks
- **Counters**: Pagination, analytics, rate limiting
- **Financial**: Tax calculations, interest computation (with Decimal for precision)

### Common Mistakes
```python
# Mistake 1: Division returning float
result = 10 / 3  # 3.333..., not 3
print(type(result))  # <class 'float'>

# Mistake 2: Integer overflow doesn't exist in Python
# In C: int x = INT_MAX + 1; // overflow
# In Python:
x = 2 ** 1000  # Works fine

# Mistake 3: Confusing int() truncation with rounding
print(int(3.9))  # 3 (truncates, not rounds)
print(round(3.9))  # 4

# Mistake 4: Leading zeros were syntax errors in Python 3
# x = 0012  # SyntaxError
x = 12  # Correct

# Mistake 5: Modulo with negative numbers
print(-7 % 3)  # 2 (not -1!)
```

### Best Practices
- Use underscores in large literals: `1_000_000_000`
- Use `//` for integer division, `/` for float division
- Use `bit_length()` for memory analysis of large integers
- Prefer `int(value)` with explicit base for string conversion
- Use `math.isclose()` for float comparison, not int checks
- Use `decimal.Decimal` for financial precision (not raw int)

### Performance Considerations
- Small ints (-5 to 257) are cached; no allocation overhead
- Arithmetic on large ints is O(n) where n is the number of digits
- Integer multiplication uses Karatsuba algorithm for large values (Python 3.11+)
- `sys.getsizeof(int)` is 28 bytes for small ints (64-bit), growing with magnitude
- Bit operations (&, |, <<, >>) on ints are very fast (C-level)

### Interview Questions
1. Are Python integers arbitrary precision? Explain.
2. What integer values are cached in CPython?
3. How does integer division work with negative numbers?
4. What is the difference between `/` and `//` in Python 3?
5. How do you convert a string to an integer with a specific base?
6. What are the bit_length() and bit_count() methods?
7. How do you convert an integer to bytes and back?
8. Why is `255.to_bytes()` invalid syntax? How do you fix it?
9. How does Python represent large integers internally?
10. What is the difference between int and long in Python 2 vs Python 3?

### Coding Challenges
```python
# Challenge 1: Check if integer is palindrome
def is_palindrome_int(n: int) -> bool:
    original = n
    reversed_n = 0
    while n > 0:
        reversed_n = reversed_n * 10 + n % 10
        n //= 10
    return original == reversed_n

print(is_palindrome_int(121))   # True
print(is_palindrome_int(-121))  # False (negative sign)
print(is_palindrome_int(10))    # False

# Challenge 2: Bit population count
def popcount(n: int) -> int:
    """Count set bits (Hamming weight)."""
    return n.bit_count()  # Python 3.8+

def popcount_manual(n: int) -> int:
    count = 0
    while n:
        count += n & 1
        n >>= 1
    return count

print(popcount(0b101010))  # 3
```

### Related Topics
- Numeric Types (float, complex)
- Bitwise Operators
- Type Conversion
- The `decimal` and `fractions` Modules
- The `math` Module

## float

### What It Is
`float` represents double-precision floating-point numbers (64-bit) following the IEEE 754 standard. Python floats correspond to `double` in C. They handle a wide range of values but have limited precision (~15-17 significant decimal digits).

### Why It Is Important
Floats are the primary numeric type for scientific computing, engineering, data science, and any application requiring fractional values. They provide fast hardware-supported arithmetic but require careful handling due to precision limitations.

### How It Works Internally
Python's `float` is implemented as a `PyFloatObject` wrapping a C `double`. In memory, 64 bits are allocated: 1 sign bit, 11 exponent bits (biased by 1023), and 52 mantissa bits (with an implicit leading 1 for normalized numbers). Special values include `inf`, `-inf`, and `nan` (Not a Number). Operations are performed by the CPU's FPU (hardware-accelerated).

### Syntax
```python
# Float literals
a = 3.14
b = .5           # 0.5
c = 1e10         # 10^10 (scientific notation)
d = 1.5e-5       # 0.000015
e = 1_000.5      # Underscore separator
f = float("inf") # Infinity
g = float("nan") # Not a Number

# Type conversion
h = float(42)     # 42.0
i = float("3.14") # 3.14
```

### Beginner Examples
```python
# Basic float operations
a, b = 10.5, 3.2
print(f"{a} + {b} = {a + b}")
print(f"{a} - {b} = {a - b}")
print(f"{a} * {b} = {a * b}")
print(f"{a} / {b} = {a / b}")

# Float precision issue
print(0.1 + 0.2)           # 0.30000000000000004
print(0.1 + 0.2 == 0.3)   # False!

import math
print(math.isclose(0.1 + 0.2, 0.3))  # True

# Special float values
print(float('inf') > 1e308)     # True
print(float('nan') != float('nan'))  # True (NaN is never equal to itself)
```

### Intermediate Examples
```python
# Float formatting and precision
pi = 3.141592653589793
print(f"{pi:.2f}")   # 3.14 (2 decimal places)
print(f"{pi:.10f}")  # 3.1415926536
print(f"{pi:e}")     # 3.141593e+00 (scientific)
print(f"{pi:.0f}")   # 3 (integer rounding)

# Float comparison
import math

def float_equal(a: float, b: float, rel_tol=1e-9, abs_tol=0.0) -> bool:
    return math.isclose(a, b, rel_tol=rel_tol, abs_tol=abs_tol)

# Machine epsilon
eps = sys.float_info.epsilon
print(f"Machine epsilon: {eps}")
print(f"1.0 + {eps} == 1.0: {1.0 + eps == 1.0}")  # False
print(f"1.0 + {eps/2} == 1.0: {1.0 + eps/2 == 1.0}")  # True

import sys

# Float info
info = sys.float_info
print(f"Max float: {info.max}")
print(f"Min float (normalized): {info.min}")
print(f"Max exponent: {info.max_exp}")
print(f"Radix: {info.radix}")
```

### Advanced Examples
```python
# Float rounding modes and the Decimal alternative
from decimal import Decimal, ROUND_HALF_EVEN

# Banker's rounding (round half to even)
print(round(2.5))  # 2 (not 3!)
print(round(3.5))  # 4
print(round(2.675, 2))  # 2.67 (not 2.68 — binary representation issue!)

# Using Decimal for precise rounding
d = Decimal("2.675").quantize(Decimal("0.01"), rounding=ROUND_HALF_EVEN)
print(d)  # 2.68

# Float internals
import struct

def float_to_hex(f: float) -> str:
    return f.hex()

def hex_to_float(h: str) -> float:
    return float.fromhex(h)

def float_to_bits(f: float) -> str:
    """Convert float to its IEEE 754 binary representation."""
    packed = struct.pack(">d", f)
    bits = format(struct.unpack(">Q", packed)[0], "064b")
    sign = bits[0]
    exponent = bits[1:12]
    mantissa = bits[12:]
    return f"Sign: {sign}, Exponent: {exponent}, Mantissa: {mantissa[:16]}..."

print(float_to_bits(3.14))
print(float_to_hex(3.14))
print(hex_to_float("0x1.91eb851eb851fp+1"))  # Approx 3.14

# Kahan summation for reduced error
def kahan_sum(values):
    total = 0.0
    compensation = 0.0
    for value in values:
        y = value - compensation
        t = total + y
        compensation = (t - total) - y
        total = t
    return total

# Compare naive vs Kahan
nums = [0.1] * 10
print(f"Naive sum: {sum(nums)}")        # 0.9999999999999999
print(f"Kahan sum: {kahan_sum(nums)}")  # 1.0
```

### Real-World Use Cases
- **Scientific Computing**: Simulation, numerical analysis
- **Graphics**: 3D transformations, camera coordinates
- **Machine Learning**: Model weights, gradients (typically float32/float64)
- **Physics Engines**: Collision detection, force calculations
- **Statistics**: Mean, variance, correlation computations

### Common Mistakes
```python
# Mistake 1: Float equality checks
x = 0.1 + 0.2
if x == 0.3:  # False!
    pass

# Correct:
if math.isclose(x, 0.3):
    pass

# Mistake 2: Accumulating float errors
total = 0.0
for _ in range(1000000):
    total += 0.1
print(total)  # 100000.0000013328 (not 100000.0)

# Mistake 3: Comparing NaN
x = float('nan')
print(x == x)    # False!
print(x is x)    # True (identity check, not value check)

# Mistake 4: Overflow to infinity
print(1e308 * 10)  # inf (not error)

# Mistake 5: Losing precision with very large numbers
print(1e16 + 1)  # 1e16 (1 is lost!)
```

### Best Practices
- Use `math.isclose()` for float equality comparisons
- Use `Decimal` for financial/monetary calculations
- Use `fractions.Fraction` for exact rational arithmetic
- Be aware of float limitations: ~15-17 significant digits
- Use `f"{value:.{precision}f}"` for controlled formatting
- Avoid accumulating small errors in long loops
- Use `float.hex()` for exact float representation debugging

### Performance Considerations
- Float operations are hardware-accelerated (CPU FPU)
- Float arithmetic is typically as fast as integer arithmetic
- `math.isclose()` is slower than direct comparison but essential for correctness
- Converting between float and Decimal is expensive
- NumPy's float32 is 2x faster than Python float for vectorized operations

### Interview Questions
1. How are floats represented in Python (IEEE 754)?
2. Why is `0.1 + 0.2 != 0.3`? Explain the precision issue.
3. What is `math.isclose()` and how does it work?
4. What are `float('inf')` and `float('nan')`? How do you check for them?
5. What is machine epsilon in Python?
6. How do you round floats in Python? What is banker's rounding?
7. What is the difference between float and Decimal?
8. How do you format floats to a specific number of decimal places?
9. What happens when a float operation overflows?
10. How does `float.hex()` help with debugging?

### Coding Challenges
```python
# Challenge 1: Float-aware sum
import math

def precise_sum(values, tolerance=1e-12):
    """Sum values with tolerance for float errors."""
    result = sum(values)
    if math.isclose(result, round(result), abs_tol=tolerance):
        return round(result)
    return result

print(precise_sum([0.1] * 10))  # 1.0

# Challenge 2: Determine float representation
import struct

def float_binary(f: float) -> dict:
    packed = struct.pack('>d', f)
    bits = format(struct.unpack('>Q', packed)[0], '064b')
    return {
        "sign": "+" if bits[0] == "0" else "-",
        "exponent": int(bits[1:12], 2) - 1023,
        "mantissa_bits": bits[12:],
        "hex": f.hex()
    }

print(float_binary(3.14))
```

### Related Topics
- IEEE 754 Standard
- The `math` Module
- The `decimal` Module
- The `fractions` Module
- Float Formatting

## complex

### What It Is
`complex` represents complex numbers with real and imaginary parts, both stored as Python floats. Complex numbers are written as `a + bj` where `a` is the real part and `b` is the imaginary part (using `j` instead of `i` as in mathematics).

### Why It Is Important
Complex numbers are essential for scientific and engineering domains including signal processing, quantum mechanics, control theory, and electrical engineering. Python is one of the few general-purpose languages with built-in complex number support.

### How It Works Internally
`PyComplexObject` stores two C `double` values: `real` and `imag`. Arithmetic operations (addition, subtraction, multiplication, division, power) are implemented in C, providing performance close to native code. Complex numbers support all standard math operations and additional methods like `conjugate()`.

### Syntax
```python
# Complex number creation
z1 = 3 + 4j
z2 = complex(3, 4)  # Same as 3 + 4j
z3 = 5j             # Pure imaginary

# Accessing parts
z = 3 + 4j
print(z.real)  # 3.0
print(z.imag)  # 4.0

# Conjugate
print(z.conjugate())  # 3 - 4j
```

### Beginner Examples
```python
# Basic complex operations
a = 3 + 4j
b = 1 - 2j

print(f"{a} + {b} = {a + b}")  # 4 + 2j
print(f"{a} - {b} = {a - b}")  # 2 + 6j
print(f"{a} * {b} = {a * b}")  # 11 - 2j
print(f"{a} / {b} = {a / b}")  # -1 + 2j

# Magnitude and phase
import cmath
z = 3 + 4j
print(f"Magnitude: {abs(z)}")          # 5.0
print(f"Phase: {cmath.phase(z)}")      # 0.927... radians
print(f"Polar: {cmath.polar(z)}")      # (5.0, 0.927...)
```

### Intermediate Examples
```python
# Complex number operations
import cmath

# Polar to rectangular
r, theta = 5, cmath.pi / 4
z = cmath.rect(r, theta)
print(f"Rectangular: {z}")  # (3.5355+3.5355j)

# Exponential and logarithmic
z = 1 + 1j
print(f"exp(z): {cmath.exp(z)}")       # e^(1+j)
print(f"log(z): {cmath.log(z)}")       # Natural log
print(f"sqrt(z): {cmath.sqrt(z)}")      # Square root

# Trigonometric functions
print(f"sin(z): {cmath.sin(z)}")
print(f"cos(z): {cmath.cos(z)}")
print(f"phase(z): {cmath.phase(z)}")   # in radians

# Complex number equality
print((3+4j).conjugate())  # (3-4j)
print((3+4j).real)         # 3.0
print((3+4j).imag)         # 4.0
```

### Advanced Examples
```python
# Mandelbrot set computation
def mandelbrot(c: complex, max_iter: int = 100) -> int:
    z = 0j
    for n in range(max_iter):
        if abs(z) > 2:
            return n
        z = z * z + c
    return max_iter

# Complex numbers in signal processing
import cmath
import math

def dft(signal: list[complex]) -> list[complex]:
    """Discrete Fourier Transform (naive O(n^2))."""
    n = len(signal)
    result = []
    for k in range(n):
        s = 0j
        for t in range(n):
            angle = -2j * cmath.pi * t * k / n
            s += signal[t] * cmath.exp(angle)
        result.append(s)
    return result

# Using complex for 2D vector operations
class Vector2D:
    def __init__(self, x: float, y: float):
        self._c = complex(x, y)
    
    @property
    def x(self): return self._c.real
    @property
    def y(self): return self._c.imag
    
    def __add__(self, other):
        return Vector2D(self.x + other.x, self.y + other.y)
    
    def rotate(self, angle: float):
        rotation = cmath.rect(1, angle)
        result = self._c * rotation
        return Vector2D(result.real, result.imag)
    
    def magnitude(self) -> float:
        return abs(self._c)
    
    def __repr__(self):
        return f"Vector2D({self.x}, {self.y})"

v = Vector2D(1, 0)
rotated = v.rotate(math.pi / 2)
print(f"Rotated 90°: {rotated}")  # ~Vector2D(0, 1)
```

### Real-World Use Cases
- **Signal Processing**: FFT, audio analysis, filter design
- **Quantum Computing**: Quantum state vectors are complex
- **Electrical Engineering**: AC circuit analysis with phasors
- **Control Theory**: Transfer functions in the s-plane
- **Fractal Generation**: Mandelbrot and Julia sets
- **Image Processing**: Frequency domain filtering

### Common Mistakes
```python
# Mistake 1: Using 'i' instead of 'j' for imaginary unit
# z = 3 + 4i  # SyntaxError

# Mistake 2: Forgetting parentheses in arithmetic
print(3 + 4j * 2)  # 3 + 8j (4j * 2 = 8j)

# Mistake 3: Comparing complex numbers for ordering
# if z1 < z2:  # TypeError
# Complex numbers don't have a natural total order

# Mistake 4: Using int division with complex
# 10 // (3+4j)  # TypeError: can't take floor of complex number

# Mistake 5: Expecting round() to work
# round(3+4j, 2)  # TypeError: type complex doesn't define __round__
```

### Best Practices
- Use `cmath` module (not `math`) for complex number functions
- Use `abs(z)` for magnitude, `cmath.phase(z)` for angle
- Prefer `complex(real, imag)` constructor for clarity
- Avoid ordering comparisons with complex numbers
- Use `numpy.complex128` for vectorized complex operations
- Convert complex to polar form with `cmath.polar()`

### Performance Considerations
- Complex arithmetic is slightly slower than float (2x operations)
- NumPy's complex arrays are significantly faster for large datasets
- `abs(z)` involves a square root (computationally expensive)
- Complex multiplication involves 4 float multiplications and 2 additions
- Python's complex numbers are passed by value (stack-allocated in C)

### Interview Questions
1. Why does Python use `j` instead of `i` for the imaginary unit?
2. How do you access the real and imaginary parts of a complex number?
3. What is the difference between `math` and `cmath` modules?
4. Can you sort complex numbers? Why or why not?
5. How do you compute the magnitude and phase of a complex number?
6. What is complex conjugation?
7. How do you convert between rectangular and polar forms?
8. What operations are not supported by complex numbers?
9. How are complex numbers stored internally in CPython?
10. When would you use complex numbers in real-world applications?

### Coding Challenges
```python
# Challenge 1: Mandelbrot set membership
def is_in_mandelbrot(c: complex, max_iter: int = 256) -> bool:
    z = 0j
    for _ in range(max_iter):
        z = z * z + c
        if abs(z) > 2:
            return False
    return True

# Challenge 2: Quadratic equation solver with complex roots
import cmath

def solve_quadratic(a: float, b: float, c: float) -> tuple[complex, complex]:
    discriminant = b*b - 4*a*c
    sqrt_d = cmath.sqrt(complex(discriminant, 0))
    x1 = (-b + sqrt_d) / (2*a)
    x2 = (-b - sqrt_d) / (2*a)
    return x1, x2

print(solve_quadratic(1, 2, 5))  # (-1+2j, -1-2j)
```

### Related Topics
- The `cmath` Module
- The `math` Module
- NumPy Complex Numbers
- Signal Processing (FFT)
- NumPy for Vectorized Operations

## str

### What It Is
`str` represents immutable sequences of Unicode code points. Strings are one of the most used types in Python, supporting rich text processing, formatting, and manipulation operations.

### Why It Is Important
Strings are fundamental to all I/O, user interaction, data processing, and web development. Python's str type provides comprehensive Unicode support (Python 3), extensive method libraries, and multiple formatting options (f-strings, format(), %-formatting).

### How It Works Internally
In CPython, `str` is stored as `PyUnicodeObject`. The internal representation depends on the maximum code point in the string: Latin-1 (1 byte per char), UCS-2 (2 bytes), or UCS-4 (4 bytes) — this is called "flexible string representation" (since Python 3.3, PEP 393). Strings are immutable; every operation returns a new string. The `intern()` mechanism caches short strings for faster identity comparisons.

### Syntax
```python
# String literals
s1 = 'single quotes'
s2 = "double quotes"
s3 = '''multi-line
string'''
s4 = """also multi-line"""
s5 = r"raw\nstring"  # Backslashes are literal
s6 = f"value = {42}" # f-string (Python 3.6+)
s7 = b"bytes"        # bytes literal (not str)

# str() constructor
s = str(42)          # "42"
s = str(3.14)        # "3.14"
s = str(True)        # "True"
```

### Beginner Examples
```python
# String operations
text = "Hello, Python!"
print(text.upper())       # "HELLO, PYTHON!"
print(text.lower())       # "hello, python!"
print(text.title())       # "Hello, Python!"
print(len(text))          # 14

# Concatenation and repetition
greeting = "Hello" + " " + "World"
print(greeting)           # "Hello World"
print("Ha" * 3)           # "HaHaHa"

# Membership
print("Python" in text)   # True
print("Java" not in text) # True

# Indexing and slicing
print(text[0])            # "H"
print(text[-1])           # "!"
print(text[0:5])          # "Hello"
print(text[::-1])         # "!nohtyP ,olleH"
```

### Real-World Use Cases
- **Web Scraping**: Extracting text from HTML/XML responses
- **Natural Language Processing**: Tokenization, stemming, sentiment analysis
- **Data Serialization**: JSON, XML, CSV generation and parsing
- **Logging**: Formatting log messages with timestamps and context
- **User Interfaces**: CLI output formatting, GUI text rendering

### Common Mistakes
```python
# Mistake 1: String concatenation in loops (O(n^2))
items = ["a", "b", "c", "d"]
result = ""
for item in items:
    result += item + ","  # Creates new string each iteration
# Correct:
result = ",".join(items)

# Mistake 2: Forgetting strings are immutable
text = "Hello"
text.upper()  # Doesn't modify text!
# Correct:
text = text.upper()

# Mistake 3: Confusing b"" (bytes) with "" (str)
data = b"hello"
# data + "world"  # TypeError: can't concat str to bytes

# Mistake 4: Encoding issues
with open("file.txt", "w") as f:  # Uses locale encoding
    f.write("café")
# Better:
with open("file.txt", "w", encoding="utf-8") as f:
    f.write("café")
```

### Best Practices
- Use f-strings for formatting (Python 3.6+)
- Use `join()` for concatenating multiple strings
- Use raw strings `r""` for regex patterns and Windows paths
- Always specify `encoding="utf-8"` for file I/O
- Use `str.casefold()` for case-insensitive comparisons
- Prefer string methods over regex for simple operations

### Performance Considerations
- f-strings are the fastest formatting method (Python 3.6+)
- `str.join()` is O(n); `+=` in a loop is O(n^2)
- String slicing creates new strings (O(k) for slice of length k)
- Interned strings save memory for repeated identical values
- The flexible string representation (PEP 393) saves memory for ASCII strings

### Interview Questions
1. What is the difference between str and bytes in Python 3?
2. Are strings mutable or immutable? How does this affect performance?
3. What is string interning in CPython?
4. How does flexible string representation work (PEP 393)?
5. What are the different ways to format strings in Python?
6. How do you efficiently concatenate many strings?
7. What is Unicode normalization and how do you perform it?
8. How does slicing work with strings?
9. What is the difference between `str.find()` and `str.index()`?
10. How do you check if a string contains only digits/letters/alphanumeric?

### Coding Challenges
(*See file 09_strings.md for detailed coverage of str() constructor, slicing, f-strings, and string methods.*)

### Related Topics
- String Methods
- String Formatting (f-strings, format(), %)
- Unicode and Encoding
- Regular Expressions
- The `re` Module

## bool

### What It Is
`bool` is a subtype of `int` with exactly two values: `True` and `False`. Booleans represent truth values and are the result of comparison and logical operations.

### Why It Is Important
Booleans enable conditional logic, loop control, and flag-based programming. They are the foundation of decision-making in programs and are used implicitly in all `if`, `while`, and logical operations through truthiness testing.

### How It Works Internally
`bool` is a subclass of `int`. `True` is implemented as `Py_True` (a singleton with integer value 1), and `False` as `Py_False` (integer value 0). The `bool()` constructor checks the object's `__bool__()` method first; if absent, falls back to `__len__()`. If neither exists, the value is always `True`. The `bool` type has `ob_refcnt` preventing deallocation (singleton pattern).

### Syntax
```python
# Boolean literals
is_active = True
is_complete = False

# Boolean as integer
print(True + True)    # 2
print(True * 10)      # 10
print(False + 5)      # 5

# Conversion
print(bool(1))        # True
print(bool(0))        # False
print(bool("hello"))  # True
print(bool(""))       # False
print(bool([]))       # False
print(bool([1, 2]))   # True
print(bool(None))     # False
```

### Beginner Examples
```python
# Truthy and falsy values
falsy = [False, None, 0, 0.0, "", [], (), {}, set(), range(0)]
for val in falsy:
    print(f"{repr(val):12} -> {bool(val)}")

truthy = [True, 1, -1, "text", [0], (1,), {1: 2}, {1}]
for val in truthy:
    print(f"{repr(val):12} -> {bool(val)}")

# Boolean operators
a, b = True, False
print(f"True and False: {a and b}")  # False
print(f"True or False: {a or b}")    # True
print(f"not True: {not a}")          # False

# Short-circuit evaluation
def side_effect():
    print("Called!")
    return True

# False and side_effect() -> side_effect NOT called (short-circuit)
print(False and side_effect())  # False
```

### Intermediate Examples
```python
# Truthiness in conditions
def process_items(items):
    if not items:  # Instead of: if len(items) == 0
        return "Empty"
    if items:      # Instead of: if len(items) > 0
        return f"Processing {len(items)} items"

print(process_items([]))       # Empty
print(process_items([1, 2]))   # Processing 2 items

# Truthiness pitfalls
def check_value(value):
    if value:  # This catches None, 0, "", [], but might not be intended
        return f"Got: {value}"
    return "No value"

print(check_value(0))        # "No value" — was 0 intentional?
print(check_value(""))       # "No value"
print(check_value(False))    # "No value"

# Explicit check for None
def check_none(value):
    if value is None:
        return "Got None"
    return f"Got: {value}"

# Boolean is subclass of int
print(issubclass(bool, int))   # True
print(isinstance(True, int))   # True
print(type(True) is bool)      # True
print(type(True) is int)       # False (type is specifically bool)
```

### Advanced Examples
```python
# Boolean array operations for performance
from array import array

class FlagSet:
    def __init__(self, size: int):
        self._flags = array('B', [0]) * size  # Byte array for compact storage
    
    def set(self, index: int):
        self._flags[index] = 1
    
    def clear(self, index: int):
        self._flags[index] = 0
    
    def __getitem__(self, index: int) -> bool:
        return bool(self._flags[index])
    
    def any(self) -> bool:
        return any(self._flags)
    
    def all(self) -> bool:
        return all(self._flags)

flags = FlagSet(10)
flags.set(3)
flags.set(5)
print(f"Flag 3: {flags[3]}")   # True
print(f"Flag 4: {flags[4]}")   # False
print(f"Any set: {flags.any()}")  # True

# Boolean short-circuit for validation
def validate_user(user: dict | None) -> bool:
    return (
        user is not None
        and user.get("is_active", False)
        and "admin" in user.get("roles", [])
    )

# Using bool for sentinel values
_SENTINEL = object()

def get_config(key: str, default: bool | object = _SENTINEL) -> bool:
    config = {"debug": True, "verbose": False}
    value = config.get(key, _SENTINEL)
    if value is _SENTINEL:
        if default is _SENTINEL:
            raise KeyError(f"Missing config key: {key}")
        return bool(default)
    return bool(value)

print(get_config("debug"))    # True
print(get_config("missing", False))  # False
```

### Real-World Use Cases
- **Feature Flags**: Boolean configuration toggles
- **Access Control**: Permission checks returning bool
- **Validation**: Form and input validation functions
- **State Machines**: Boolean flags for application state
- **Search Filters**: Combining multiple boolean criteria

### Common Mistakes
```python
# Mistake 1: == True / == False comparisons
is_active = True
if is_active == True:  # Redundant
    pass
# Better:
if is_active:
    pass

# Mistake 2: Using is with True/False
value = True
if value is True:  # Works but fragile (value could be 1)
    pass
# Better:
if value:
    pass

# Mistake 3: Truthiness with numeric 0
count = 0
if count:  # False! But count=0 might be valid
    print(f"Count is {count}")
# Better:
if count is not None:
    print(f"Count is {count}")

# Mistake 4: Using 'and'/'or' when '&'/'|' is intended
# and/or are logical operators (short-circuit)
# &/| are bitwise operators (no short-circuit)

# Mistake 5: bool(container) checks emptiness but not membership
my_list = [0]
if my_list:  # True (non-empty)
    pass
```

### Best Practices
- Use implicit truthiness with `if items:` not `if len(items) > 0:`
- Use `is None` for None checks, not truthiness
- Use `is` for comparing with True/False only in specific cases
- Prefer named boolean variables for complex conditions
- Use `any()` and `all()` for iterable boolean checks
- Return `bool` from validation functions, not strings/int

### Performance Considerations
- `bool()` is very fast (checks `__bool__` / `__len__` only)
- Short-circuit `and`/`or` can avoid expensive function calls
- Boolean `True`/`False` are singletons — no allocation
- `any()` and `all()` short-circuit on first match
- `bool(value)` and `not not value` are equivalent in speed

### Interview Questions
1. Is bool a subclass of int in Python?
2. What values are considered falsy in Python?
3. What is the difference between `and`/`or` and `&`/`|`?
4. How does short-circuit evaluation work with `and` and `or`?
5. How does `bool()` determine the truth value of an object?
6. What is the difference between `if x:` and `if x is True:`?
7. Can you create a custom truthiness behavior for your class?
8. How do `any()` and `all()` work?
9. What does `bool([])` return? What about `bool([[]])`?
10. How does Python handle boolean logic in list comprehensions?

### Coding Challenges
```python
# Challenge 1: Boolean truth table generator
def truth_table():
    ops = [
        ("and", lambda a, b: a and b),
        ("or", lambda a, b: a or b),
        ("xor", lambda a, b: a != b),
        ("nand", lambda a, b: not (a and b)),
    ]
    for name, op in ops:
        print(f"\n{name}:")
        for a in [True, False]:
            for b in [True, False]:
                print(f"  {a} {name} {b} = {op(a, b)}")

# Challenge 2: Custom truthiness
class TruthyList:
    def __init__(self, items, always_truthy=False):
        self.items = items
        self.always_truthy = always_truthy
    
    def __bool__(self):
        if self.always_truthy:
            return True
        return len(self.items) > 0

t = TruthyList([], always_truthy=True)
print(bool(t))  # True
```

### Related Topics
- Truthiness and Falsy Values
- Boolean Operators
- Short-Circuit Evaluation
- The `all()` and `any()` Functions
- Comparison Operators

## NoneType

### What It Is
`NoneType` is the type of the single value `None`, representing the absence of a value. `None` is Python's null value, similar to `null` in Java, `nil` in Ruby, or `NULL` in C.

### Why It Is Important
`None` serves as a sentinel value for missing data, optional parameters, function returns with no explicit return, and default arguments. It is essential for indicating "no value" in a way that is distinct from zero, empty string, or empty container.

### How It Works Internally
`None` is a singleton — only one instance exists per interpreter session. It is implemented as `Py_None` in CPython, a global `PyObject` with a reference count that is never zeroed. `None` is a keyword in Python 3 (not just a built-in name). Its `__bool__()` method returns `False`, making it falsy.

### Syntax
```python
# Assignment
result = None

# Checking for None (always use 'is')
if result is None:
    print("No result")

if result is not None:
    print(f"Result: {result}")

# Functions implicitly return None
def no_return():
    pass

print(no_return())  # None

# Type annotation
from typing import Optional
value: Optional[int] = None
```

### Beginner Examples
```python
# None as default for function parameters
def find_user(user_id: int) -> str | None:
    users = {1: "Alice", 2: "Bob"}
    return users.get(user_id)  # Returns None if not found

user = find_user(3)
if user is None:
    print("User not found")
else:
    print(f"Found: {user}")

# None vs empty string
def format_name(first: str, last: str, middle: str | None = None) -> str:
    if middle is None:
        return f"{first} {last}"
    return f"{first} {middle} {last}"

print(format_name("John", "Doe"))          # John Doe
print(format_name("John", "Doe", "M."))    # John M. Doe

# None in containers
data = {"name": "Alice", "email": None}
if data["email"] is None:
    print("Email not provided")
```

### Intermediate Examples
```python
# None as sentinel
_MISSING = object()

def get_config(key: str, default: str | None = _MISSING) -> str | None:
    config = {"host": "localhost", "port": "8080"}
    if key not in config:
        if default is _MISSING:
            raise KeyError(f"Missing config: {key}")
        return default
    return config[key]

# get_config("host")           # "localhost"
# get_config("missing")       # KeyError
# get_config("missing", None) # None

# None propagation
class User:
    def __init__(self, name: str, email: str | None = None):
        self.name = name
        self.email = email
    
    def get_email_domain(self) -> str | None:
        if self.email is None:
            return None
        return self.email.split("@")[1]

user = User("Alice")
print(user.get_email_domain())  # None

# None in type hints (Python 3.10+)
def process(value: int | None = None) -> str:
    if value is None:
        return "No value provided"
    return f"Value: {value}"

# The null object pattern
class NullLogger:
    def debug(self, msg): pass
    def info(self, msg): pass
    def warning(self, msg): pass
    def error(self, msg): pass

class Application:
    def __init__(self, logger=None):
        self.logger = logger or NullLogger()
    
    def run(self):
        self.logger.info("Running...")

# Uses NullLogger (no AttributeError)
app = Application()
app.run()
```

### Advanced Examples
```python
# Optional type with None semantics
from typing import Optional

class Node:
    def __init__(self, value: int, next_node: Optional["Node"] = None):
        self.value = value
        self.next = next_node
    
    def traverse(self) -> list[int]:
        values = []
        current: Optional[Node] = self
        while current is not None:
            values.append(current.value)
            current = current.next
        return values

# Build linked list: 1 -> 2 -> 3 -> None
head = Node(1, Node(2, Node(3)))
print(head.traverse())  # [1, 2, 3]

# None-aware attribute access (Python 3.x doesn't have ?. operator)
class SafeDict(dict):
    def get_nested(self, *keys):
        current = self
        for key in keys:
            if current is None:
                return None
            current = current.get(key)
        return current

data = {"a": {"b": {"c": 42}}}
sd = SafeDict(data)
print(sd.get_nested("a", "b", "c"))  # 42
print(sd.get_nested("a", "x", "y"))  # None

# The Optional type vs None union
from typing import Union, Optional

# These are equivalent in Python 3.10+:
value1: Optional[int]  # Optional[int] -> int | None (Python 3.10+)
value2: int | None     # Preferred style in Python 3.10+

# None in dataclasses
from dataclasses import dataclass, field

@dataclass
class Config:
    host: str = "localhost"
    port: int = 8080
    database_url: str | None = None
    
    def get_connection_string(self) -> str | None:
        if self.database_url is None:
            return None
        return f"{self.database_url}?host={self.host}&port={self.port}"
```

### Real-World Use Cases
- **Database Operations**: NULL column values map to None
- **API Responses**: Optional fields as None
- **Configuration**: Default None for optional settings
- **Caching**: None for uncached values
- **Error Handling**: Function returns None on failure instead of raising
- **Linked Lists / Trees**: None for leaf nodes or end markers

### Common Mistakes
```python
# Mistake 1: Using == instead of is for None
value = None
if value == None:  # Works but wrong (can be overridden)
    pass
# Correct:
if value is None:
    pass

# Mistake 2: Truthiness check with None
value = None
if not value:  # True! But also true for 0, "", []
    pass
# Better:
if value is None:
    pass

# Mistake 3: Mutable default using None incorrectly
def append(item, target=None):
    if target is None:
        target = []
    target.append(item)
    return target

# Mistake 4: Using None as a dictionary key
# d = {None: "value"}  # Works but confusing

# Mistake 5: Returning None when an exception is better
def divide(a, b):
    if b == 0:
        return None  # Better to raise ValueError
    return a / b
```

### Best Practices
- Always use `is None` and `is not None` for comparison
- Use `Optional[Type]` or `Type | None` in type hints (Python 3.10+)
- Use None as sentinel for "value not provided" in optional parameters
- Don't use truthiness checks when None is a legitimate value
- Prefer returning None vs raising exceptions for expected missing data
- Use the Null Object pattern instead of None for polymorphic behavior
- Document when a function can return None

### Performance Considerations
- `None` is a singleton; identity checks are extremely fast (pointer comparison)
- `value is None` is faster than `value == None`
- `if x:` is slightly faster than `if x is not None:` but semantically different
- None in containers adds a pointer overhead (8 bytes on 64-bit)
- Using None as a default argument is the standard pattern (no comparison overhead)

### Interview Questions
1. What is None in Python and how is it implemented?
2. Why should you use `is None` instead of `== None`?
3. Can you subclass NoneType?
4. What is the difference between None and NaN?
5. How do you use None as a sentinel value?
6. What is the Null Object pattern and how does it relate to None?
7. How does None work with type hints?
8. What functions implicitly return None?
9. How do you handle None in nested data structures?
10. What is the Optional type and when should you use it?

### Coding Challenges
```python
# Challenge 1: Deep None-safe accessor
def deep_get(obj, *keys, default=None):
    """Safely access nested dictionary values."""
    current = obj
    for key in keys:
        if current is None:
            return default
        current = current.get(key) if isinstance(current, dict) else default
        if current is None:
            return default
    return current

nested = {"a": {"b": {"c": 42}}}
print(deep_get(nested, "a", "b", "c"))  # 42
print(deep_get(nested, "a", "x"))       # None

# Challenge 2: Maybe/Option monad pattern (simplified)
class Maybe:
    def __init__(self, value):
        self._value = value
    
    @classmethod
    def of(cls, value):
        return cls(value)
    
    def map(self, func):
        if self._value is None:
            return Maybe(None)
        return Maybe(func(self._value))
    
    def get_or_else(self, default):
        return self._value if self._value is not None else default

result = Maybe.of("hello").map(str.upper).map(len).get_or_else(0)
print(result)  # 5

result = Maybe.of(None).map(str.upper).get_or_else("default")
print(result)  # "default"
```

### Related Topics
- The `Optional` Type
- The Null Object Pattern
- Sentinel Values
- `is` vs `==` Comparison
- None in Databases and APIs
