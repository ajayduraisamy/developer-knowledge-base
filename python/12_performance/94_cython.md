# Cython - Static types, .pyx files, C extensions, compilation

## Introduction

Cython is a programming language that makes writing C extensions for Python as easy as Python itself. It allows you to write Python code that is translated into C extensions, providing near C-level performance while maintaining Python-like syntax.

## Why It Is Important

Cython bridges the gap between Python's ease of use and C's performance. It enables significant speedups (often 10-100x) for numerical computations, tight loops, and operations that would be slow in pure Python. It also allows direct interaction with C libraries.

## Syntax

```python
# Cython .pyx file syntax
# cdef for C-level functions and variables
# cpdef for functions callable from both C and Python
# def for pure Python functions

# Static type declarations
cdef int i
cdef double x
cdef list items
cdef dict mapping

# Memory views for array access
cdef int[:] arr_view = arr

# C imports
# from libc.math cimport sin, cos
# from libc.stdlib cimport malloc, free

# Function declarations
cdef int square(int x):
    return x * x

cpdef double average(list data):
    cdef int n = len(data)
    cdef double total = 0.0
    for i in range(n):
        total += data[i]
    return total / n
```

## Examples

```python
import math
import time


def pure_python_sum_squares(n: int) -> float:
    total = 0.0
    for i in range(n):
        total += math.sqrt(i ** 2)
    return total


start = time.perf_counter()
result = pure_python_sum_squares(1000000)
print(f"Pure Python: {time.perf_counter() - start:.4f}s")


def demonstrate_cython_syntax():
    print("Cython equivalent would look like:")
    print("""
    cimport cython
    from libc.math cimport sqrt

    @cython.boundscheck(False)
    @cython.wraparound(False)
    cpdef double cython_sum_squares(int n):
        cdef int i
        cdef double total = 0.0
        for i in range(n):
            total += sqrt(i * i)
        return total
    """)


demonstrate_cython_syntax()
```

## Beginner Examples

```python
import time
import math


def cython_basics_intro():
    print("Cython Basics:")
    print("1. Cython files use .pyx extension")
    print("2. Use cdef for C variables")
    print("3. Use cdef for C-only functions")
    print("4. Use cpdef for dual C/Python functions")
    print("5. Use def for Python-only functions")
    print("6. Static typing gives major speedups")


cython_basics_intro()


def compare_loop_speed():
    n = 1000000

    def python_loop():
        total = 0
        for i in range(n):
            total += i
        return total

    start = time.perf_counter()
    python_loop()
    py_time = time.perf_counter() - start
    print(f"Python loop: {py_time:.4f}s")

    print("""
    Cython version would be:
    cpdef int cython_loop(int n):
        cdef int i
        cdef int total = 0
        for i in range(n):
            total += i
        return total
    Expected: ~10-50x faster
    """)


compare_loop_speed()


def setup_py_example():
    print("setup.py for building Cython extension:")
    print("""
    from setuptools import setup
    from Cython.Build import cythonize

    setup(
        ext_modules = cythonize("example.pyx")
    )
    """)
    print("Build with: python setup.py build_ext --inplace")


setup_py_example()
```

## Intermediate Examples

```python
import time
import math
from typing import List


class CythonConcepts:
    def __init__(self):
        pass

    def static_declarations(self):
        print("Static type declarations in Cython:")
        print("  cdef int i               # C int")
        print("  cdef float f              # C float")
        print("  cdef double d             # C double")
        print("  cdef char c               # C char")
        print("  cdef long long ll         # C long long")
        print("  cdef int[10] arr          # C array")
        print("  cdef int* ptr             # C pointer")
        print("  cdef list py_list         # Python list")
        print("  cdef dict py_dict         # Python dict")
        print("  cdef str s                # Python string")

    def cdef_vs_cpdef(self):
        print("\ncdef functions:")
        print("  - Only callable from Cython")
        print("  - Fastest, no Python overhead")
        print("  - Return type must be C type")
        print("\ncpdef functions:")
        print("  - Callable from both C and Python")
        print("  - Slightly slower than cdef")
        print("  - Python wrapper generated")
        print("\ndef functions:")
        print("  - Pure Python, no speedup")
        print("  - Callable from Python")

    def interacting_with_c(self):
        print("\nInteracting with C libraries:")
        print("  from libc.math cimport sin, cos, sqrt")
        print("  from libc.stdlib cimport malloc, free")
        print("  from libc.string cimport strlen, memcpy")
        print("  from libc.stdio cimport printf")


concepts = CythonConcepts()
concepts.static_declarations()
concepts.cdef_vs_cpdef()
concepts.interacting_with_c()


def manual_optimization_pattern():
    print("\nCython optimization decorators:")
    print("  @cython.boundscheck(False)  # Skip bounds checks")
    print("  @cython.wraparound(False)   # Skip negative indexing")
    print("  @cython.nonecheck(False)    # Skip None checks")
    print("  @cython.cdivision(True)     # C division semantics")
    print("  @cython.infer_types(True)   # Infer types automatically")


manual_optimization_pattern()


def cdef_class_example():
    print("\nCython extension types (cdef class):")
    print("""
    cdef class Point:
        cdef public double x, y

        def __init__(self, double x, double y):
            self.x = x
            self.y = y

        cpdef double distance(self, Point other):
            return sqrt((self.x - other.x) ** 2 + (self.y - other.y) ** 2)
    """)


cdef_class_example()
```

## Advanced Examples

```python
import time
import math
import random
from typing import List, Tuple


class AdvancedCython:
    def __init__(self):
        pass

    def memory_views(self):
        print("Memory views for efficient array access:")
        print("""
    cdef double[:, :] view_2d = numpy_array
    cdef int[:] view_1d = array

    # Access without Python overhead
    for i in range(shape[0]):
        for j in range(shape[1]):
            view_2d[i, j] = view_2d[i, j] * 2.0
        """)

    def fused_types(self):
        print("\nFused types (templates):")
        print("""
    cimport cython

    @cython.fused_type
    ctypedef fused my_type:
        int
        double
        float

    cpdef my_type max(my_type a, my_type b):
        return a if a > b else b
        """)

    def parallel_support(self):
        print("\nOpenMP parallel support:")
        print("""
    from cython.parallel import prange, parallel

    @cython.boundscheck(False)
    @cython.wraparound(False)
    cpdef double parallel_sum(double[:] arr):
        cdef int i
        cdef double total = 0.0
        for i in prange(arr.shape[0], nogil=True):
            total += arr[i]
        return total
        """)

    def callback_to_python(self):
        print("\nCalling Python from Cython:")
        print("""
    cpdef void apply_func(list data, object func):
        cdef int i
        for i in range(len(data)):
            data[i] = func(data[i])  # Still Python overhead
        """)


adv = AdvancedCython()
adv.memory_views()
adv.fused_types()
adv.parallel_support()
adv.callback_to_python()


def performance_comparison_detailed():
    print("\nDetailed performance comparison:")

    def pure_python(n: int) -> float:
        total = 0.0
        for i in range(n):
            total += math.sin(i) * math.cos(i)
        return total

    n = 500000
    start = time.perf_counter()
    pure_python(n)
    t1 = time.perf_counter() - start

    print(f"Pure Python: {t1:.4f}s")
    print("""
    Expected Cython performance:
    - With cdef types: {:.4f}s (estimated)
    - With @boundscheck(False): {:.4f}s (estimated)
    - With nogil parallel: {:.4f}s (estimated)
    """.format(t1 / 20, t1 / 30, t1 / 40))


performance_comparison_detailed()


def compilation_process():
    print("Compilation process:")
    print("1. Cython compiles .pyx to .c")
    print("2. C compiler compiles .c to .so/.pyd")
    print("3. Extension is importable in Python")
    print()
    print("Build methods:")
    print("  - setup.py with cythonize()")
    print("  - pyximport for on-the-fly compilation")
    print("  - Cython.Build.cythonize()")
    print("  - Manual: cython -3 file.pyx")
    print()


compilation_process()
```

## Real-World Use Cases

```python
import time
import math
import random


def scientific_computing():
    print("Real-world: Scientific Computing")

    def compute_pi_python(n: int) -> float:
        total = 0.0
        step = 1.0 / n
        for i in range(n):
            x = (i + 0.5) * step
            total += 4.0 / (1.0 + x * x)
        return total * step

    n = 1000000
    start = time.perf_counter()
    result = compute_pi_python(n)
    t = time.perf_counter() - start
    print(f"Pi (Python): {result:.6f}, Time: {t:.4f}s")
    print("Cython version would be 50-100x faster")


def game_development():
    print("Real-world: Game Development")
    print("""
    cdef class Vector3:
        cdef double x, y, z

        cpdef Vector3 add(Vector3 other):
            return Vector3(self.x + other.x,
                          self.y + other.y,
                          self.z + other.z)

        cpdef double magnitude(self):
            return sqrt(self.x * self.x +
                       self.y * self.y +
                       self.z * self.z)
    """)


def image_processing():
    print("Real-world: Image Processing")
    print("""
    @cython.boundscheck(False)
    @cython.wraparound(False)
    cpdef void grayscale(unsigned char[:, :, :] img):
        cdef int i, j
        cdef unsigned char r, g, b, gray
        for i in range(img.shape[0]):
            for j in range(img.shape[1]):
                r = img[i, j, 0]
                g = img[i, j, 1]
                b = img[i, j, 2]
                gray = <unsigned char>(0.299 * r + 0.587 * g + 0.114 * b)
                img[i, j, 0] = gray
                img[i, j, 1] = gray
                img[i, j, 2] = gray
    """)


def wrapping_c_library():
    print("Real-world: Wrapping C Libraries")
    print("""
    cdef extern from "sodium.h":
        int crypto_box_easy(
            unsigned char *ciphertext,
            const unsigned char *plaintext,
            unsigned long long mlen,
            const unsigned char *n,
            const unsigned char *pk,
            const unsigned char *sk
        )

    def encrypt(bytes plaintext, bytes nonce, bytes pk, bytes sk):
        cdef unsigned char *ciphertext
        cdef int result
        # ... allocate and call C function
    """)


scientific_computing()
game_development()
image_processing()
wrapping_c_library()
```

## Common Mistakes

```python
def mistake_1_wrong_type_declarations():
    print("Mistake 1: Wrong or missing type declarations")
    print("Without types, Cython is no faster than Python")
    print("Always declare types for loop variables")


def mistake_2_python_overhead_in_hot_loops():
    print("Mistake 2: Python calls inside tight loops")
    print("Avoid calling Python functions in hot loops")
    print("Use C math library instead of math module")


def mistake_3_no_bounds_check_disable():
    print("Mistake 3: Not disabling bounds checks")
    print("@cython.boundscheck(False) for hot loops")
    print("Only disable after verifying correctness")


def mistake_4_using_list_append():
    print("Mistake 4: Using Python list append in loops")
    print("Pre-allocate with cdef list arr = [0] * n")
    print("Or use memory views for arrays")


def mistake_5_not_using_nogil():
    print("Mistake 5: Not using 'nogil' where possible")
    print("Release GIL for pure C operations")
    print("with nogil: # pure C operations here")


def mistake_6_wrong_build_config():
    print("Mistake 6: Incorrect build configuration")
    print("Use proper compiler flags for optimization")
    print("Add -O3 flag for maximum optimization")


mistake_1_wrong_type_declarations()
mistake_2_python_overhead_in_hot_loops()
mistake_3_no_bounds_check_disable()
mistake_4_using_list_append()
mistake_5_not_using_nogil()
mistake_6_wrong_build_config()
```

## Best Practices

```python
import math


def best_practice_1_profile_first():
    print("Best Practice 1: Profile before converting to Cython")
    print("Only convert the hot paths that need optimization")


def best_practice_2_incremental_conversion():
    print("Best Practice 2: Convert incrementally")
    print("1. Profile to find bottlenecks")
    print("2. Add type declarations")
    print("3. Convert to cdef/cpdef functions")
    print("4. Add optimization decorators")
    print("5. Add nogil blocks")


def best_practice_3_use_memory_views():
    print("Best Practice 3: Use memory views for arrays")
    print("Memory views provide zero-overhead access")
    print("Works with NumPy, array, bytes objects")


def best_practice_4_minimize_python_interaction():
    print("Best Practice 4: Minimize Python interaction")
    print("Each Python call adds overhead")
    print("Batch Python operations when possible")


def best_practice_5_compile_with_optimization():
    print("Best Practice 5: Compile with optimization flags")
    print("  setup.py: extra_compile_args=['-O3']")
    print("  Consider LTO and architecture-specific flags")


def best_practice_6_test_thoroughly():
    print("Best Practice 6: Test thoroughly")
    print("Cython can introduce subtle bugs")
    print("Test both Python and Cython versions")


best_practice_1_profile_first()
best_practice_2_incremental_conversion()
best_practice_3_use_memory_views()
best_practice_4_minimize_python_interaction()
best_practice_5_compile_with_optimization()
best_practice_6_test_thoroughly()
```

## Interview Questions

```python
def interview_q1():
    print("Q: What is Cython and how does it improve performance?")
    print("A: Cython compiles Python to C extensions.")
    print("   Static typing removes interpreter overhead.")


def interview_q2():
    print("Q: What is the difference between cdef and cpdef?")
    print("A: cdef: C-only function, faster, no Python call overhead.")
    print("   cpdef: Both C and Python callable.")


def interview_q3():
    print("Q: How do you compile a Cython file?")
    print("A: Using cythonize in setup.py or pyximport.")
    print("   Output: .c file -> .so/.pyd extension.")


def interview_q4():
    print("Q: What are memory views in Cython?")
    print("A: Efficient array access without Python overhead.")
    print("   Typed memory views: int[:], double[:, :]")


def interview_q5():
    print("Q: What is nogil and when should it be used?")
    print("A: Releases the GIL for parallel execution.")
    print("   Only use with pure C operations.")


def interview_q6():
    print("Q: How does Cython compare to Numba?")
    print("A: Cython: compile-time optimization, broader scope.")
    print("   Numba: JIT runtime, best for NumPy operations.")


def interview_q7():
    print("Q: What are fused types in Cython?")
    print("A: Cython's template mechanism for generic functions.")
    print("   Similar to C++ templates or Java generics.")


interview_q1()
interview_q2()
interview_q3()
interview_q4()
interview_q5()
interview_q6()
interview_q7()
```

## Coding Challenges

```python
import time
import math


def challenge_1_vector_dot_product():
    print("Challenge 1: Implement dot product in Cython")

    def dot_product_python(a, b):
        total = 0.0
        for i in range(len(a)):
            total += a[i] * b[i]
        return total

    n = 1000000
    a = [i * 0.5 for i in range(n)]
    b = [i * 0.3 for i in range(n)]
    start = time.perf_counter()
    result = dot_product_python(a, b)
    t = time.perf_counter() - start
    print(f"Python dot product: {result:.2f}, Time: {t:.4f}s")
    print("Cython version would be ~50x faster")


def challenge_2_mandelbrot_set():
    print("Challenge 2: Implement Mandelbrot set computation")
    print("""
    cpdef int mandelbrot(double cr, double ci, int max_iter):
        cdef double zr = 0.0, zi = 0.0
        cdef double zr2, zi2
        cdef int n = 0
        while n < max_iter:
            zr2 = zr * zr
            zi2 = zi * zi
            if zr2 + zi2 > 4.0:
                return n
            zi = 2.0 * zr * zi + ci
            zr = zr2 - zi2 + cr
            n += 1
        return max_iter
    """)


def challenge_3_matrix_multiplication():
    print("Challenge 3: Implement matrix multiplication")
    print("""
    @cython.boundscheck(False)
    @cython.wraparound(False)
    cpdef void matrix_multiply(double[:, :] A, double[:, :] B, double[:, :] C):
        cdef int i, j, k
        cdef double total
        for i in range(A.shape[0]):
            for j in range(B.shape[1]):
                total = 0.0
                for k in range(A.shape[1]):
                    total += A[i, k] * B[k, j]
                C[i, j] = total
    """)


def challenge_4_implement_reduce():
    print("Challenge 4: Implement parallel sum reduction")
    print("""
    from cython.parallel import prange

    @cython.boundscheck(False)
    @cython.wraparound(False)
    cpdef double parallel_sum(double[:] arr):
        cdef int i
        cdef double total = 0.0
        for i in prange(arr.shape[0], nogil=True):
            total += arr[i]
        return total
    """)


challenge_1_vector_dot_product()
challenge_2_mandelbrot_set()
challenge_3_matrix_multiplication()
challenge_4_implement_reduce()
```

## Summary

Cython bridges Python and C by compiling Python-like code to C extensions. It offers significant performance gains through static typing, C-level functions, memory views, and parallel processing. Best for numerical computing, game development, image processing, and wrapping C libraries.

## Related Topics

- Numba (95_numba.md)
- CPython Internals (93_cpython_internals.md)
- Profiling (91_profiling.md)
- Advanced Topics (100_advanced_topics.md)
