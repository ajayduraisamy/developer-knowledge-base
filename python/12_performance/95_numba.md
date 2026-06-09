# Numba - @jit decorator, vectorize, GPU acceleration

## Introduction
Numba is a just-in-time (JIT) compiler for Python that translates a subset of Python and NumPy into fast machine code using LLVM. By decorating functions with `@jit`, you can achieve speeds approaching C or Fortran without leaving Python. Numba also provides `@vectorize` for NumPy ufunc-style operations and GPU backends (CUDA and ROCm) for parallel acceleration. It is widely used in scientific computing, data science, and quantitative finance.

## @jit decorator

### What It Is
The `@jit` decorator instructs Numba to compile a function at call time. On the first invocation, Numba infers the types of arguments, generates LLVM IR, optimises it, and compiles to native machine code. Subsequent calls reuse the cached compiled version. The resulting function runs without Python interpreter overhead.

### Why It Is Important
`@jit` is the simplest way to accelerate Python numeric code. A tight loop with arithmetic operations typically runs 50–200x faster with `@jit` than in pure CPython. It requires minimal code changes — just add a decorator.

### How It Works Internally
1. **Type inference**: Numba analyses the bytecode of the decorated function and determines the concrete types of all variables based on the input argument types.
2. **Numba IR**: The bytecode is translated into Numba's internal intermediate representation (SSA form).
3. **LLVM IR generation**: The Numba IR is lowered to LLVM IR, with type-specialised operations.
4. **LLVM optimisation and code generation**: LLVM applies standard optimisations (loop unrolling, inlining, auto-vectorisation) and emits native machine code.
5. **Caching**: The compiled function is stored in an in-memory cache keyed by argument types.

### Syntax
```python
from numba import jit
import numpy as np

@jit(nopython=True)          # nopython mode: no Python objects
def my_function(x):
    return x ** 2 + 1

@jit(nopython=True, parallel=True)
def parallel_sum(arr):
    total = 0
    for i in numba.prange(len(arr)):
        total += arr[i]
    return total

@jit(nopython=True, cache=True)
def cached_function(x):
    return x ** 0.5

# Lazy compilation — types inferred at call time
# Also eager compilation with explicit signatures
@jit('float64(float64, float64)', nopython=True)
def add(a, b):
    return a + b
```

### Beginner Examples
```python
import numba
import numpy as np
import time

@numba.jit(nopython=True)
def sum_squares(n):
    total = 0
    for i in range(n):
        total += i * i
    return total

# First call compiles, subsequent calls are fast
start = time.perf_counter()
result = sum_squares(10_000_000)
jit_time = time.perf_counter() - start

def pure_python(n):
    total = 0
    for i in range(n):
        total += i * i
    return total

start = time.perf_counter()
result2 = pure_python(10_000_000)
py_time = time.perf_counter() - start

print(f'JIT: {jit_time:.3f}s, Python: {py_time:.3f}s, Speedup: {py_time/jit_time:.1f}x')
```

### Intermediate Examples
```python
import numba
import numpy as np
from numba import jit, prange

@jit(nopython=True, parallel=True)
def monte_carlo_pi(n):
    count = 0
    for i in prange(n):
        x = np.random.random()
        y = np.random.random()
        if x * x + y * y <= 1.0:
            count += 1
    return 4.0 * count / n

@jit(nopython=True)
def mandelbrot(c, max_iter):
    z = 0j
    for n in range(max_iter):
        if abs(z) > 2:
            return n
        z = z * z + c
    return max_iter

@jit(nopython=True, parallel=True)
def mandelbrot_set(xmin, xmax, ymin, ymax, width, height, max_iter):
    x_vals = np.linspace(xmin, xmax, width)
    y_vals = np.linspace(ymin, ymax, height)
    result = np.zeros((height, width), dtype=np.int32)
    for i in prange(height):
        for j in range(width):
            result[i, j] = mandelbrot(complex(x_vals[j], y_vals[i]), max_iter)
    return result
```

### Advanced Examples
```python
import numba
import numpy as np
from numba import jit, cfunc, types, carray

# Eager compilation with explicit signature
@jit(types.float64(types.float64[:], types.int64), nopython=True)
def cumulative_sum(arr, start):
    result = np.empty_like(arr)
    acc = start
    for i in range(len(arr)):
        acc += arr[i]
        result[i] = acc
    return result

# Structured array support
mype = np.dtype([('x', np.float64), ('y', np.float64), ('val', np.float64)])

@jit(nopython=True)
def process_points(points):
    total = 0.0
    for i in range(len(points)):
        total += points[i].x * points[i].y * points[i].val
    return total

# Callback function with cfunc (for C-callable functions)
@cfunc(types.float64(types.float64))
def square(x):
    return x * x

# Using with NumPy functions
@jit(nopython=True)
def black_scholes(S, K, T, r, sigma):
    """Black-Scholes pricing with Numba."""
    from numba import exp, log, sqrt
    d1 = (log(S / K) + (r + 0.5 * sigma ** 2) * T) / (sigma * sqrt(T))
    d2 = d1 - sigma * sqrt(T)
    from scipy.special import ndtr
    # Note: scipy.special functions may not be available in nopython mode
    # Use Numba's built-in erf instead
    from numba import erf
    norm_cdf = lambda x: 0.5 * (1 + erf(x / sqrt(2)))
    return S * norm_cdf(d1) - K * exp(-r * T) * norm_cdf(d2)

# Object mode fallback (avoid in hot paths)
@jit
def flexible(x):
    if isinstance(x, int):
        return x * 2
    elif isinstance(x, float):
        return x ** 2
    return 0
```

### Real-World Use Cases
- **Quantitative finance**: price thousands of options with the Black-Scholes formula or run Monte Carlo simulations for VaR calculations.
- **Image processing**: apply convolution kernels, colour space conversions, and filtering with per-pixel operations.
- **Physics simulations**: N-body gravitational simulations, molecular dynamics, and lattice Boltzmann methods.

### Common Mistakes
- Using Python objects inside `nopython=True` (e.g., `list.append` of mixed types, `dict` with non-homogeneous values).
- Forgetting that `nopython=True` falls back to object mode silently unless explicitly set — always include `nopython=True`.
- Calling unsupported NumPy functions — check Numba's supported NumPy subset.
- Expecting compilation to be free — the first call is slower than pure Python; use `cache=True` or eager compilation for production.

### Best Practices
- Always use `nopython=True` — it guarantees full compilation and maximum speed.
- Use `parallel=True` and `prange` for embarrassingly parallel loops.
- Use `cache=True` to persist compiled functions to disk across sessions (avoids recompilation at startup).
- Use `@jit(signature)` for eager compilation when you know the input types at development time.

### Performance Considerations
- First call overhead: compilation takes 0.1–2 seconds depending on function complexity.
- Inline costs: small functions are inlined by LLVM; very large functions may increase compilation time without proportional benefit.
- Memory bandwidth: for array-heavy operations, Numba is often limited by memory bandwidth, not CPU speed.

### Interview Questions
- **Q**: What is the difference between `nopython=True` and `object` mode in Numba?  
  **A**: `nopython=True` compiles everything to machine code with no Python objects, giving maximum speed. Object mode (default fallback) leaves some operations as Python calls, which are slower.
- **Q**: How does `prange` differ from `range` in Numba?  
  **A**: `prange` is Numba's parallel loop construct — iterations are distributed across CPU threads, unlike `range` which runs serially.

### Coding Challenges
- Write a Numba-accelerated `sieve_of_eratosthenes` and compare its performance to a pure Python and a NumPy implementation.
- Implement a 2D convolution kernel with `@jit(nopython=True, parallel=True)` and benchmark against SciPy's `ndimage.convolve`.

### Related Topics
- [@vectorize](#vectorize)
- [GPU acceleration](#gpu-acceleration)
- [Cython](#cython---static-types-pyx-files-c-extensions-compilation)

---

## @vectorize

### What It Is
`@numba.vectorize` creates NumPy universal functions (ufuncs) that operate element-wise on arrays. A function decorated with `@vectorize` is compiled to operate on scalar values, but Numba generates the broadcasting and iteration logic automatically, so it can be applied to entire arrays as `my_ufunc(arr1, arr2)`.

### Why It Is Important
`@vectorize` eliminates the need to write explicit loops when applying scalar functions to arrays. It is faster than NumPy's `np.vectorize` (which is a Python loop) and often competitive with hand-written NumPy expressions because Numba can fuse operations and avoid intermediate arrays.

### How It Works Internally
1. **Scalar kernel**: The user writes a function operating on scalar values.
2. **Type specialisation**: Numba generates specialised versions for each combination of input types (int32, float64, complex128, etc.).
3. **Loop generation**: LLVM generates the broadcasting loop, handling strides, shapes, and output allocation.
4. **SIMD vectorisation**: LLVM auto-vectorises the inner loop using SSE/AVX instructions when possible.
5. **Parallel execution** (with `target='parallel'`): iterations are split across CPU threads.

### Syntax
```python
from numba import vectorize, float64, int64
import numpy as np

# Simple vectorize (CPU, single-threaded)
@vectorize([float64(float64, float64)])
def add(x, y):
    return x + y

# Target-specific vectorize
@vectorize([float64(float64, float64)], target='parallel')
def add_parallel(x, y):
    return x + y

# No signature — Numba infers types (slower compilation)
@vectorize
def multiply(x, y):
    return x * y

# Multiple signatures
@vectorize([
    float64(float64, float64),
    int64(int64, int64)
])
def add_generic(x, y):
    return x + y
```

### Beginner Examples
```python
import numba
import numpy as np

@numba.vectorize([numba.float64(numba.float64, numba.float64)])
def hypotenuse(a, b):
    return (a ** 2 + b ** 2) ** 0.5

x = np.random.randn(10_000_000)
y = np.random.randn(10_000_000)

result = hypotenuse(x, y)

# Compare to manual loop
@numba.jit(nopython=True)
def manual_hyp(a, b):
    result = np.empty_like(a)
    for i in range(len(a)):
        result[i] = (a[i] ** 2 + b[i] ** 2) ** 0.5
    return result

result2 = manual_hyp(x, y)
print(f'Max diff: {np.max(np.abs(result - result2))}')
```

### Intermediate Examples
```python
import numba
import numpy as np
from numba import vectorize, float64, boolean

# Custom activation function
@vectorize([float64(float64)], target='parallel')
def swish(x):
    return x / (1.0 + np.exp(-x))

# Complex step with guards
@vectorize([float64(float64, float64, float64)])
def clamped_power(x, exp, clamp):
    if x < -clamp:
        return -clamp ** exp
    elif x > clamp:
        return clamp ** exp
    else:
        return x ** exp

# Boolean ufunc
@vectorize([boolean(float64, float64)])
def is_close(a, b):
    return abs(a - b) < 1e-8

# Custom reduction-like operation
@vectorize([float64(float64, float64)])
def smoothstep(edge0, edge1, x):
    t = max(0.0, min(1.0, (x - edge0) / (edge1 - edge0)))
    return t * t * (3.0 - 2.0 * t)
```

### Advanced Examples
```python
import numba
import numpy as np
from numba import vectorize, cuda, float64, complex128

# Multiple outputs with guvectorize
from numba import guvectorize

@guvectorize([(float64[:], float64[:], float64[:])], '(n),(n)->(n)')
def add_vectors(a, b, out):
    for i in range(a.shape[0]):
        out[i] = a[i] + b[i]

# Ufunc with complex numbers
@vectorize([complex128(complex128, float64)])
def phase_shift(z, phi):
    return z * np.exp(1j * phi)

# Gather operation with vectorize
@vectorize([float64(float64[:], int64)])
def gather(arr, idx):
    return arr[idx]

# Using vectorize with structured dtypes (Numba >= 0.57)
point_dtype = np.dtype([('x', np.float64), ('y', np.float64)])
point_array = np.zeros(1000, dtype=point_dtype)

@vectorize([float64(float64, float64)])
def magnitude(x, y):
    return np.sqrt(x * x + y * y)

result = magnitude(point_array['x'], point_array['y'])

# Performance comparison
import time

def numpy_version(x, y):
    return np.sqrt(x**2 + y**2)

numba_ufunc = add_parallel if 'add_parallel' in locals() else add

x = np.random.randn(10_000_000)
y = np.random.randn(10_000_000)

t0 = time.perf_counter()
r1 = numpy_version(x, y)
t1 = time.perf_counter()

t2 = time.perf_counter()
r2 = add_parallel(x, y)
t3 = time.perf_counter()

print(f'NumPy: {t1-t0:.3f}s, Numba vectorize: {t3-t2:.3f}s')
```

### Real-World Use Cases
- **Element-wise image operations**: gamma correction, colour space conversion, thresholding.
- **Signal processing**: apply window functions, filter kernels, and normalisation to large sensor arrays.
- **Machine learning feature engineering**: create custom transformations that are not available in scikit-learn.

### Common Mistakes
- Using `@vectorize` for operations that are already implemented as NumPy ufuncs (redundant).
- Forgetting that `@vectorize` works element-wise; for operations that need neighbour access, use `@jit` with explicit loops or `@stencil`.
- Specifying too many type signatures — use a generic signature or let Numba infer.

### Best Practices
- Use `target='parallel'` for large arrays ( > 1 million elements) to utilise all CPU cores.
- Use `@guvectorize` for operations that reduce dimensions or need multiple output arrays.
- Benchmark against NumPy's native ufuncs — many NumPy operations are already implemented in C and may be faster for simple operations.

### Performance Considerations
- `@vectorize` with `target='parallel'` can saturate all CPU cores but has thread-launch overhead (~10 µs).
- For small arrays (< 10k elements), the overhead of Numba compilation and dispatch outweighs the speed benefit.
- Memory bandwidth is often the bottleneck for `@vectorize` — CPU cache blocking helps, but Numba does not automatically tile.

### Interview Questions
- **Q**: How does `@vectorize` differ from `@jit` when applied to arrays?  
  **A**: `@jit` compiles explicit loops; `@vectorize` takes a scalar function and generates the broadcasting loop automatically, creating a proper NumPy ufunc.
- **Q**: What is `@guvectorize` and when would you use it?  
  **A**: `@guvectorize` generalises `@vectorize` to arbitrary input/output shapes (including reductions), specified via a layout signature like `(n),(n)->()`. Use it for operations like dot products or reductions.

### Coding Challenges
- Implement a Gaussian blur using `@vectorize` with a sliding window (hint: use `@guvectorize` or `@jit` with loops instead).
- Create a `@vectorize`-based implementation of the softmax function that operates on 2D arrays row-wise.

### Related Topics
- [@jit decorator](#jit-decorator)
- [GPU acceleration](#gpu-acceleration)
- [Cython](#cython---static-types-pyx-files-c-extensions-compilation)

---

## GPU Acceleration

### What It Is
Numba provides GPU acceleration through the `numba.cuda` (NVIDIA CUDA) and `numba.roc` (AMD ROCm) backends. You write kernel functions in a subset of Python, and Numba compiles them to GPU machine code (PTX for CUDA, HSAIL for ROCm). Host-side code can launch kernels, copy data to/from the device, and manage GPU memory.

### Why It Is Important
GPUs contain thousands of cores that can execute the same instruction on different data simultaneously (SIMT). For data-parallel workloads — matrix multiplication, image convolution, particle simulations — GPU acceleration can provide 10–100x speedup over multi-core CPUs.

### How It Works Internally
1. **Kernel compilation**: `@cuda.jit` compiles the kernel function (with special thread hierarchy variables `threadIdx`, `blockIdx`, `blockDim`, `gridDim`) to PTX.
2. **Data transfer**: `cuda.to_device` copies NumPy arrays to GPU global memory.
3. **Kernel launch**: `kernel_function[blocks_per_grid, threads_per_block](args)` launches the kernel on the GPU.
4. **Synchronisation**: `cuda.synchronize()` waits for kernel completion; `cuda.stream` allows asynchronous operations.
5. **Result transfer**: `cuda.copy_to_host` copies results back to CPU memory.

### Syntax
```python
from numba import cuda
import numpy as np

@cuda.jit
def vector_add(a, b, out):
    idx = cuda.grid(1)           # 1D global thread index
    if idx < out.shape[0]:
        out[idx] = a[idx] + b[idx]

# Data preparation
n = 1_000_000
a = cuda.to_device(np.random.randn(n))
b = cuda.to_device(np.random.randn(n))
out = cuda.device_array_like(a)

# Kernel launch
threads_per_block = 256
blocks_per_grid = (n + threads_per_block - 1) // threads_per_block
vector_add[blocks_per_grid, threads_per_block](a, b, out)

# Retrieve result
result = out.copy_to_host()
```

### Beginner Examples
```python
import numpy as np
from numba import cuda

@cuda.jit
def saxpy(alpha, x, y, out):
    i = cuda.grid(1)
    if i < out.size:
        out[i] = alpha * x[i] + y[i]

n = 100_000
x = cuda.to_device(np.random.randn(n).astype(np.float32))
y = cuda.to_device(np.random.randn(n).astype(np.float32))
out = cuda.device_array_like(x)

threads = 256
blocks = (n + threads - 1) // threads
saxpy[blocks, threads](2.0, x, y, out)

result = out.copy_to_host()
print(f'Saxpy result (first 5): {result[:5]}')
```

### Intermediate Examples
```python
import numpy as np
from numba import cuda

# 2D kernel with shared memory
TPB = 16

@cuda.jit
def matmul(A, B, C):
    row, col = cuda.grid(2)
    if row >= C.shape[0] or col >= C.shape[1]:
        return

    tile_A = cuda.shared.array((TPB, TPB), dtype=np.float32)
    tile_B = cuda.shared.array((TPB, TPB), dtype=np.float32)

    acc = 0.0
    for tile in range(0, A.shape[1], TPB):
        if row < A.shape[0] and tile + cuda.threadIdx.x < A.shape[1]:
            tile_A[cuda.threadIdx.y, cuda.threadIdx.x] = A[row, tile + cuda.threadIdx.x]
        if col < B.shape[1] and tile + cuda.threadIdx.y < B.shape[0]:
            tile_B[cuda.threadIdx.y, cuda.threadIdx.x] = B[tile + cuda.threadIdx.y, col]
        cuda.syncthreads()

        for k in range(TPB):
            acc += tile_A[cuda.threadIdx.y, k] * tile_B[k, cuda.threadIdx.x]
        cuda.syncthreads()

    C[row, col] = acc

# Reduction kernel
@cuda.jit
def reduce_sum(arr, result):
    tid = cuda.threadIdx.x
    bid = cuda.blockIdx.x
    bdim = cuda.blockDim.x
    i = bid * bdim + tid

    shared = cuda.shared.array((TPB,), dtype=np.float32)
    shared[tid] = arr[i] if i < arr.size else 0.0
    cuda.syncthreads()

    s = bdim // 2
    while s > 0:
        if tid < s:
            shared[tid] += shared[tid + s]
        cuda.syncthreads()
        s //= 2

    if tid == 0:
        result[bid] = shared[0]
```

### Advanced Examples
```python
import numpy as np
from numba import cuda
import math

# Streams for overlapping copy and compute
@cuda.jit
def process_chunk(data, out):
    i = cuda.grid(1)
    if i < data.size:
        out[i] = math.sin(data[i]) * math.cos(data[i])

def pipelined_processing(data):
    n = len(data)
    chunk_size = n // 4
    streams = [cuda.stream() for _ in range(4)]
    d_data_chunks = []
    d_out_chunks = []
    results = []

    for i in range(4):
        start = i * chunk_size
        end = start + chunk_size if i < 3 else n
        chunk = data[start:end]
        d_chunk = cuda.to_device(chunk, stream=streams[i])
        d_out = cuda.device_array_like(chunk, stream=streams[i])
        d_data_chunks.append(d_chunk)
        d_out_chunks.append(d_out)
        threads = 256
        blocks = (len(chunk) + threads - 1) // threads
        process_chunk[blocks, threads, streams[i]](d_chunk, d_out)

    for i in range(4):
        results.append(d_out_chunks[i].copy_to_host(stream=streams[i]))

    cuda.synchronize()
    return np.concatenate(results)

# Texture memory for read-only data
from numba.cuda import texref

@cuda.jit
def texture_lookup(tex, positions, out):
    i = cuda.grid(1)
    if i < positions.shape[0]:
        x, y = positions[i, 0], positions[i, 1]
        out[i] = tex[x, y]

# Dynamic parallelism (CUDA >= 2.0)
@cuda.jit
def child_kernel(data):
    i = cuda.grid(1)
    if i < data.size:
        data[i] *= 2.0

@cuda.jit
def parent_kernel(data):
    i = cuda.grid(1)
    if i == 0:
        child_kernel[1, 128](data)
```

### Real-World Use Cases
- **Deep learning**: Numba-compiled custom CUDA kernels for novel activation functions or layers that are not in PyTorch/TensorFlow.
- **Computational fluid dynamics**: Lattice Boltzmann simulations running on GPU with 50x speedup over CPU.
- **Real-time image processing**: live video stream processing at 60+ FPS using GPU kernels for edge detection and colour correction.

### Common Mistakes
- Launching kernels with insufficient threads to cover the data — leads to silent partial computation.
- Forgetting `cuda.synchronize()` before reading results — reads stale data from host memory.
- Using Python objects inside GPU kernels — only NumPy scalar types and Numba-supported operations are allowed.
- Not considering memory transfer overhead — copying data to/from GPU can dominate runtime for small arrays.

### Best Practices
- Coalesce global memory accesses: adjacent threads should access adjacent memory addresses.
- Use shared memory for data reused within a block (e.g., tiled matrix multiplication).
- Overlap data transfers with kernel execution using streams.
- Profile with `nvidia-smi` and Nvidia Nsight to identify occupancy bottlenecks.

### Performance Considerations
- Memory bandwidth is often the limiting factor; optimise for coalesced, aligned accesses.
- Register pressure reduces occupancy — use fewer local variables in kernels.
- PCIe transfer latency (~5–10 µs) plus bandwidth (up to 32 GB/s on PCIe 4.0) means only large arrays (>1 MB) benefit from GPU offload.

### Interview Questions
- **Q**: What is the difference between `threadIdx`, `blockIdx`, `blockDim`, and `gridDim` in CUDA?  
  **A**: `threadIdx` is the thread's index within its block; `blockIdx` is the block's index in the grid; `blockDim` is the number of threads per block; `gridDim` is the number of blocks per grid.
- **Q**: Why is shared memory important in GPU kernels?  
  **A**: Shared memory is on-chip SRAM that is ~100x faster than global memory. It allows threads in the same block to share data without repeated global memory reads.

### Coding Challenges
- Implement a GPU-accelerated Sobel edge detection kernel using `numba.cuda`.
- Write a GPU reduction kernel that finds the maximum value in an array and compare its performance with `np.max`.

### Related Topics
- [@jit decorator](#jit-decorator)
- [@vectorize](#vectorize)
- [Cython](#cython---static-types-pyx-files-c-extensions-compilation)
