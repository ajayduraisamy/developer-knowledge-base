# Numba - @jit decorator, vectorize, GPU acceleration

## Introduction

Numba is a Just-In-Time (JIT) compiler for Python that translates a subset of Python and NumPy code into fast machine code using LLVM. It provides decorators like @jit, @vectorize, and @cuda to accelerate numerical computations.

## Why It Is Important

Numba enables near C or Fortran performance for numerical Python code with minimal changes. It is especially powerful for array operations, mathematical computations, and loops that would be slow in pure Python. No separate compilation step is required.

## Syntax

```python
# Basic @jit decorator
from numba import jit
import numpy as np

@jit(nopython=True)
def sum_squares(n):
    total = 0.0
    for i in range(n):
        total += i ** 2
    return total

# @vectorize for element-wise operations
from numba import vectorize

@vectorize(['float64(float64, float64)'], target='cpu')
def add(x, y):
    return x + y

# @cuda for GPU computing
from numba import cuda

@cuda.jit
def kernel_function(arr):
    idx = cuda.grid(1)
    if idx < arr.size:
        arr[idx] = arr[idx] * 2

# @njit shortcut for nopython=True
from numba import njit

@njit
def fast_function(x):
    return x ** 2
```

## Examples

```python
import numpy as np
import time
import math


def pure_python_loop(n: int) -> float:
    total = 0.0
    for i in range(n):
        total += math.sqrt(i)
    return total


start = time.perf_counter()
pure_python_loop(1000000)
print(f"Pure Python: {time.perf_counter() - start:.4f}s")


def numba_example():
    print("Numba JIT compilation example:")
    print("""
    from numba import jit
    import math

    @jit(nopython=True)
    def numba_loop(n):
        total = 0.0
        for i in range(n):
            total += math.sqrt(i)
        return total
    """)


numba_example()
```

## Beginner Examples

```python
import time
import math
import numpy as np


def jit_basics():
    print("Numba JIT Basics:")
    print("1. @jit(nopython=True) for best performance")
    print("2. @njit is shorthand for @jit(nopython=True)")
    print("3. First call compiles, subsequent calls are fast")
    print("4. Works best with NumPy arrays and loops")
    print("5. Supports most math functions")


jit_basics()


def simple_numba_comparison():
    n = 1000000

    def python_sum():
        total = 0.0
        for i in range(n):
            total += i ** 0.5
        return total

    start = time.perf_counter()
    python_sum()
    py_time = time.perf_counter() - start
    print(f"Python time: {py_time:.4f}s")

    print("""
    Numba version:
    from numba import njit

    @njit
    def numba_sum(n):
        total = 0.0
        for i in range(n):
            total += i ** 0.5
        return total

    First call (compilation + execution): ~0.5s
    Subsequent calls: ~0.01s (50-100x faster)
    """)


simple_numba_comparison()


def vectorize_example():
    print("\n@vectorize decorator:")
    print("""
    @vectorize(['float64(float64, float64)'])
    def multiply(x, y):
        return x * y

    a = np.array([1.0, 2.0, 3.0])
    b = np.array([4.0, 5.0, 6.0])
    result = multiply(a, b)  # [4.0, 10.0, 18.0]
    """)


vectorize_example()
```

## Intermediate Examples

```python
import time
import math
import numpy as np
from typing import Tuple


class NumbaPatterns:
    def __init__(self):
        pass

    def jit_variants(self):
        print("JIT decorator variants:")
        print("""
    @jit                    # Lazy compilation, object mode fallback
    @jit(nopython=True)     # Strict compilation, best performance
    @njit                   # Shorthand for @jit(nopython=True)
    @jit(parallel=True)     # Automatic parallelization
    @jit(fastmath=True)     # Enable fast math optimizations
    @jit(cache=True)        # Cache compilation results
    @jit(nogil=True)        # Release GIL during execution
        """)

    def supported_features(self):
        print("Supported Python features in nopython mode:")
        print("  - if/elif/else, for/while loops")
        print("  - Basic math: +, -, *, /, **, %")
        print("  - math functions: sqrt, sin, cos, exp, log")
        print("  - NumPy: arrays, slicing, np.sum, np.mean")
        print("  - Tuples, namedtuples, enums")
        print("  - Classes (limited)")
        print("  - Random number generation")
        print()
        print("NOT supported:")
        print("  - try/except, with statements")
        print("  - sets, dictionaries")
        print("  - Variable number of arguments")
        print("  - Most of the Python standard library")
        print("  - Arbitrary Python objects")

    def compilation_caching(self):
        print("Compilation caching:")
        print("""
    @jit(cache=True)
    def cached_function(x):
        return x ** 2

    # Compiled version saved to disk
    # No recompilation on next run
        """)


patterns = NumbaPatterns()
patterns.jit_variants()
patterns.supported_features()
patterns.compilation_caching()


def practical_jit_example():
    n = 1000000

    def python_black_scholes(S, K, T, r, sigma):
        d1 = (math.log(S / K) + (r + 0.5 * sigma ** 2) * T) / (sigma * math.sqrt(T))
        d2 = d1 - sigma * math.sqrt(T)
        call = S * 0.5 * (1 + math.erf(d1 / math.sqrt(2))) - K * math.exp(-r * T) * 0.5 * (1 + math.erf(d2 / math.sqrt(2)))
        return call

    def compute_many_python():
        results = []
        for i in range(n):
            S = 100 + i * 0.01
            results.append(python_black_scholes(S, 100, 1, 0.05, 0.2))
        return results

    start = time.perf_counter()
    result_sample = python_black_scholes(105, 100, 1, 0.05, 0.2)
    print(f"Sample result: {result_sample:.4f}")
    print("Python version would be slow for 1M iterations")
    print("Numba @njit would make it ~100x faster")


practical_jit_example()
```

## Advanced Examples

```python
import time
import math
import numpy as np
from typing import Tuple, Optional


class AdvancedNumba:
    def __init__(self):
        pass

    def parallel_jit(self):
        print("Parallel JIT with @jit(parallel=True):")
        print("""
    from numba import prange

    @jit(nopython=True, parallel=True)
    def parallel_sum(arr):
        total = 0.0
        for i in prange(len(arr)):
            total += arr[i]
        return total
        """)

    def cuda_programming(self):
        print("CUDA GPU programming:")
        print("""
    from numba import cuda
    import numpy as np

    @cuda.jit
    def vector_add(a, b, c):
        idx = cuda.grid(1)
        if idx < a.size:
            c[idx] = a[idx] + b[idx]

    # Launch kernel
    threads_per_block = 256
    blocks_per_grid = (a.size + threads_per_block - 1) // threads_per_block
    vector_add[blocks_per_grid, threads_per_block](a, b, c)
        """)

    def vectorize_targets(self):
        print("Vectorize with different targets:")
        print("""
    @vectorize(['float64(float64, float64)'], target='cpu')
    def cpu_func(x, y):
        return x + y

    @vectorize(['float64(float64, float64)'], target='parallel')
    def parallel_func(x, y):
        return x + y

    @vectorize(['float64(float64, float64)'], target='cuda')
    def cuda_func(x, y):
        return x + y
        """)

    def guvectorize(self):
        print("Generalized UFuncs (guvectorize):")
        print("""
    from numba import guvectorize

    @guvectorize(['(float64[:], float64[:])'], '(n)->(n)')
    def moving_avg(inp, out):
        for i in range(len(out)):
            total = 0.0
            for j in range(3):
                if i + j < len(inp):
                    total += inp[i + j]
            out[i] = total / 3.0
        """)


adv = AdvancedNumba()
adv.parallel_jit()
adv.cuda_programming()
adv.vectorize_targets()
adv.guvectorize()


def stencil_example():
    print("Stencil computations (for image processing):")
    print("""
    from numba import stencil

    @stencil
    def blur_kernel(a):
        return (a[-1, -1] + a[-1, 0] + a[-1, 1] +
                a[0, -1]  + a[0, 0]  + a[0, 1] +
                a[1, -1]  + a[1, 0]  + a[1, 1]) / 9.0

    result = blur_kernel(image)
    """)


stencil_example()


def record_array_example():
    print("Numba Record Arrays (structured arrays):")
    print("""
    import numpy as np
    from numba import njit
    from numba import types
    from numba.typed import Dict

    @njit
    def process_records():
        # Use typed Dict for dictionary-like functionality
        d = Dict.empty(
            key_type=types.unicode_type,
            value_type=types.float64
        )
        d['value'] = 42.0
        return d
    """)


record_array_example()
```

## Real-World Use Cases

```python
import time
import math
import numpy as np


def financial_computing():
    print("Real-world: Financial Computing")

    def monte_carlo_pi_python(n: int) -> float:
        count = 0
        for _ in range(n):
            x, y = np.random.random(), np.random.random()
            if x * x + y * y <= 1.0:
                count += 1
        return 4.0 * count / n

    n = 1000000
    start = time.perf_counter()
    result = monte_carlo_pi_python(n)
    t = time.perf_counter() - start
    print(f"Monte Carlo Pi: {result:.6f}, Time: {t:.4f}s")
    print("Numba @njit would be ~100x faster")


def image_processing():
    print("Real-world: Image Processing")
    print("""
    @njit(parallel=True)
    def grayscale_image(image):
        result = np.empty_like(image)
        for i in prange(image.shape[0]):
            for j in range(image.shape[1]):
                r, g, b = image[i, j]
                gray = 0.299 * r + 0.587 * g + 0.114 * b
                result[i, j] = gray
        return result
    """)


def machine_learning():
    print("Real-world: Machine Learning Kernels")
    print("""
    @njit
    def euclidean_distances(X, Y):
        m = X.shape[0]
        n = Y.shape[0]
        D = np.empty((m, n), dtype=np.float64)
        for i in range(m):
            for j in range(n):
                total = 0.0
                for k in range(X.shape[1]):
                    diff = X[i, k] - Y[j, k]
                    total += diff * diff
                D[i, j] = math.sqrt(total)
        return D
    """)


def odes_simulation():
    print("Real-world: ODE Simulation")
    print("""
    @njit
    def lorenz_system(state, sigma, rho, beta, dt, steps):
        trajectory = np.empty((steps, 3))
        x, y, z = state
        for i in range(steps):
            dx = sigma * (y - x)
            dy = x * (rho - z) - y
            dz = x * y - beta * z
            x += dx * dt
            y += dy * dt
            z += dz * dt
            trajectory[i] = [x, y, z]
        return trajectory
    """)


financial_computing()
image_processing()
machine_learning()
odes_simulation()
```

## Common Mistakes

```python
def mistake_1_unsupported_python_features():
    print("Mistake 1: Using unsupported Python features")
    print("  - try/except blocks in nopython mode")
    print("  - Variable-length arguments")
    print("  - Arbitrary Python objects")
    print("  - Most of Python standard library")


def mistake_2_not_using_nopython():
    print("Mistake 2: Forgetting nopython=True")
    print("Without it, Numba falls back to object mode")
    print("Object mode is often slower than pure Python")


def mistake_3_compilation_overhead():
    print("Mistake 3: Not accounting for compilation time")
    print("First call compiles the function")
    print("Use sample call for warmup before timing")


def mistake_4_calling_numba_funcs_from_python():
    print("Mistake 4: Too many small Numba calls")
    print("Each call has overhead from Python/Numba boundary")
    print("Batch work into larger Numba functions")


def mistake_5_list_and_dict_usage():
    print("Mistake 5: Using Python lists/dicts in nopython")
    print("Use numpy arrays instead of lists")
    print("Use numba.typed.Dict instead of dict")


def mistake_6_wrong_dtypes():
    print("Mistake 6: Type inference issues")
    print("Use explicit types when inference fails")
    print("Check with numba.typeof() for debugging")


mistake_1_unsupported_python_features()
mistake_2_not_using_nopython()
mistake_3_compilation_overhead()
mistake_4_calling_numba_funcs_from_python()
mistake_5_list_and_dict_usage()
mistake_6_wrong_dtypes()
```

## Best Practices

```python
import time
import numpy as np


def best_practice_1_warmup():
    print("Best Practice 1: Warm up the JIT compiler")
    print("Call the function once to trigger compilation")
    print("Then measure execution time")


def best_practice_2_use_arrays():
    print("Best Practice 2: Use NumPy arrays")
    print("Numba works best with homogeneous arrays")
    print("Avoid Python lists in hot paths")


def best_practice_3_batch_operations():
    print("Best Practice 3: Batch operations")
    print("Minimize calls between Python and Numba")
    print("Do more work inside each Numba call")


def best_practice_4_check_numba_type():
    print("Best Practice 4: Verify types with numba.typeof")
    print("from numba import typeof")
    print("typeof(my_variable)  # See inferred type")


def best_practice_5_use_parallel():
    print("Best Practice 5: Use parallel=True when possible")
    print("Use prange for loop parallelization")
    print("Ensure no cross-iteration dependencies")


def best_practice_6_cache_compilation():
    print("Best Practice 6: Cache compiled functions")
    print("@jit(cache=True) saves .pyc cache files")
    print("No recompilation on subsequent runs")


best_practice_1_warmup()
best_practice_2_use_arrays()
best_practice_3_batch_operations()
best_practice_4_check_numba_type()
best_practice_5_use_parallel()
best_practice_6_cache_compilation()
```

## Interview Questions

```python
def interview_q1():
    print("Q: How does Numba JIT compilation work?")
    print("A: Numba uses LLVM to compile Python bytecode")
    print("   to machine code at runtime.")


def interview_q2():
    print("Q: What is nopython mode?")
    print("A: Strict mode where all type inference is done")
    print("   at compile time. Best performance.")


def interview_q3():
    print("Q: What is the @vectorize decorator?")
    print("A: Creates NumPy ufuncs from scalar functions.")
    print("   Supports CPU, parallel, and CUDA targets.")


def interview_q4():
    print("Q: What Python features are supported?")
    print("A: Loops, conditionals, math, NumPy arrays,")
    print("   basic types, tuples. No exception handling.")


def interview_q5():
    print("Q: How does Numba compare to Cython?")
    print("A: Numba: JIT, no compilation step, NumPy focus.")
    print("   Cython: Compile-time, broader C integration.")


def interview_q6():
    print("Q: What is prange in Numba?")
    print("A: Parallel range for loop parallelization.")
    print("   Used with @jit(parallel=True).")


def interview_q7():
    print("Q: What are Numba's limitations?")
    print("A: Limited Python features, no dict/set natively,")
    print("   compilation overhead on first call.")


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
import numpy as np


def challenge_1_mandelbrot():
    print("Challenge 1: Implement Mandelbrot set with @njit")
    print("""
    @njit
    def mandelbrot(cr, ci, max_iter):
        zr, zi = 0.0, 0.0
        for n in range(max_iter):
            zr2, zi2 = zr * zr, zi * zi
            if zr2 + zi2 > 4.0:
                return n
            zi = 2.0 * zr * zi + ci
            zr = zr2 - zi2 + cr
        return max_iter

    @njit
    def mandelbrot_set(xmin, xmax, ymin, ymax, width, height, max_iter):
        result = np.empty((height, width), dtype=np.int32)
        dx = (xmax - xmin) / width
        dy = (ymax - ymin) / height
        for i in range(height):
            for j in range(width):
                result[i, j] = mandelbrot(xmin + j * dx, ymin + i * dy, max_iter)
        return result
    """)


def challenge_2_black_scholes():
    print("Challenge 2: Black-Scholes option pricing")
    n = 1000000
    S = np.random.uniform(80, 120, n)
    K = 100.0
    T = 1.0
    r = 0.05
    sigma = 0.2

    def python_black_scholes_array(S, K, T, r, sigma):
        d1 = (np.log(S / K) + (r + 0.5 * sigma ** 2) * T) / (sigma * np.sqrt(T))
        d2 = d1 - sigma * np.sqrt(T)
        call = S * 0.5 * (1 + np.vectorize(math.erf)(d1 / np.sqrt(2))) - K * np.exp(-r * T) * 0.5 * (1 + np.vectorize(math.erf)(d2 / np.sqrt(2)))
        return call

    print("NumPy vectorized version is already fast")
    print("Numba @vectorize would add GPU support")



def challenge_3_ray_tracing():
    print("Challenge 3: Simple ray tracer with @njit")
    print("""
    @njit
    def ray_sphere_intersect(ray_origin, ray_dir, sphere_center, sphere_radius):
        oc = ray_origin - sphere_center
        a = np.dot(ray_dir, ray_dir)
        b = 2.0 * np.dot(oc, ray_dir)
        c = np.dot(oc, oc) - sphere_radius * sphere_radius
        discriminant = b * b - 4 * a * c
        if discriminant < 0:
            return -1.0
        t = (-b - math.sqrt(discriminant)) / (2.0 * a)
        return t
    """)


def challenge_4_kmeans():
    print("Challenge 4: K-Means clustering")
    print("""
    @njit
    def kmeans(X, centroids, max_iter=100):
        n, k = X.shape[0], centroids.shape[0]
        labels = np.empty(n, dtype=np.int64)
        for _ in range(max_iter):
            # Assignment step
            for i in range(n):
                min_dist = np.inf
                for j in range(k):
                    dist = np.sum((X[i] - centroids[j]) ** 2)
                    if dist < min_dist:
                        min_dist = dist
                        labels[i] = j
            # Update step
            for j in range(k):
                mask = labels == j
                if np.sum(mask) > 0:
                    centroids[j] = np.mean(X[mask], axis=0)
        return labels, centroids
    """)


challenge_1_mandelbrot()
challenge_2_black_scholes()
challenge_3_ray_tracing()
challenge_4_kmeans()
```

## Summary

Numba provides JIT compilation for Python, accelerating numerical code by 10-100x with minimal changes. Key features include @jit for function acceleration, @vectorize for ufuncs, and @cuda for GPU programming. Best suited for NumPy-heavy code, loops, and mathematical computations.

## Related Topics

- Cython (94_cython.md)
- Profiling (91_profiling.md)
- Data Structures (97_data_structures.md)
- Algorithms (98_algorithms.md)
