# Cython - Static types, .pyx files, C extensions, compilation

## Introduction
Cython is a superset of Python that compiles Python-like code into C extensions, achieving performance close to hand-written C. By adding static type declarations to Python code, Cython bypasses the CPython interpreter overhead and generates optimised C code that calls directly into the CPython C API or external C libraries. It is widely used in scientific computing, data processing, and for wrapping native libraries.

## Static Type Declarations

### What It Is
Cython extends Python's syntax with C-level type annotations using `cdef` and `cpdef` keywords. Variables, function parameters, return types, and structs can be annotated with C types (`int`, `double`, `float`, `Py_ssize_t`, pointers, etc.), which causes Cython to generate C code that operates on raw C values instead of Python `PyObject*` pointers.

### Why It Is Important
Python's dynamic typing forces every operation to go through the interpreter loop and PyObject protocol. Static typing allows Cython to compile arithmetic, loops, and function calls into native C instructions — typically 10–100x faster for compute-intensive code.

### How It Works Internally
Cython's compiler (`cythonize`) parses `.pyx` files, resolves type annotations, and emits a `.c` file. For a typed variable `cdef int x`, the generated C code declares `int x;` and uses `x = PyLong_AsLong(py_obj);` for assignments. Arithmetic on typed variables becomes raw C `+`, `-`, `*`, `/` operations. Function calls with `cdef` become direct C function calls (no Python call overhead).

### Syntax
```cython
# Pure Python style with cython annotation comments
import cython

@cython.cfunc
@cython.returns(cython.int)
@cython.locals(a=cython.int, b=cython.int)
def add(a, b):
    return a + b

# .pyx style
# cdef int add(int a, int b):
#     return a + b
```

### Beginner Examples
```python
# compute.pyx
def compute_pure(n):
    total = 0
    for i in range(n):
        total += i * i
    return total

# With static types
def compute_typed(int n):
    cdef int i
    cdef long long total = 0
    for i in range(n):
        total += i * i
    return total

# setup.py
from setuptools import setup
from Cython.Build import cythonize

setup(ext_modules=cythonize('compute.pyx'))
```

### Intermediate Examples
```cython
# math_ops.pyx
cimport cython

@cython.boundscheck(False)
@cython.wraparound(False)
def dot_product(double[:] a, double[:] b):
    cdef:
        int i
        int n = a.shape[0]
        double total = 0.0

    for i in range(n):
        total += a[i] * b[i]
    return total

# Enums and structs
cdef struct Point:
    double x
    double y

cdef double distance(Point p1, Point p2):
    cdef double dx = p1.x - p2.x
    cdef double dy = p1.y - p2.y
    return (dx * dx + dy * dy) ** 0.5

def compute_distances(double[:] xs, double[:] ys):
    cdef:
        int n = xs.shape[0]
        int i, j
        list result = []
        Point p1, p2

    for i in range(n):
        p1 = Point(xs[i], ys[i])
        for j in range(i + 1, n):
            p2 = Point(xs[j], ys[j])
            result.append(distance(p1, p2))
    return result
```

### Advanced Examples
```cython
# advanced_typed.pyx
from libc.math cimport sin, cos, sqrt
from libc.stdlib cimport malloc, free

cdef class ComplexArray:
    """Array of complex numbers with C-level storage."""
    cdef:
        double *real
        double *imag
        int size

    def __init__(self, int size):
        self.size = size
        self.real = <double*>malloc(size * sizeof(double))
        self.imag = <double*>malloc(size * sizeof(double))

    def __dealloc__(self):
        free(self.real)
        free(self.imag)

    cpdef void set(self, int idx, double r, double i):
        self.real[idx] = r
        self.imag[idx] = i

    cpdef double magnitude(self, int idx):
        return sqrt(self.real[idx] * self.real[idx] + self.imag[idx] * self.imag[idx])

    @cython.boundscheck(False)
    cpdef void multiply_all(self, double factor):
        cdef int i
        for i in range(self.size):
            self.real[i] *= factor
            self.imag[i] *= factor

# Template-like fused types
ctypedef fused numeric:
    float
    double
    int
    long long

cdef numeric square(numeric x):
    return x * x

def apply_square(numeric[:] arr):
    cdef int i
    cdef int n = arr.shape[0]
    for i in range(n):
        arr[i] = square(arr[i])
```

### Real-World Use Cases
- **NumPy/SciPy internals**: many NumPy operations are backed by Cython-generated C code that processes arrays element-by-element without Python overhead.
- **Game physics engines**: Cython particle systems and collision detectors run at native speed while keeping game logic in Python.
- **High-frequency trading**: latency-critical order book processing in Cython bridges Python risk infrastructure with C++ matching engines.

### Common Mistakes
- Declaring a variable `cdef int x` but forgetting to initialise it — Cython does not zero-initialise.
- Using `def` instead of `cdef` or `cpdef` for internal functions — every call still goes through Python.
- Accessing Python objects (lists, dicts) inside typed loops — each access re-wraps the value in a `PyObject*`.

### Best Practices
- Use `@cython.boundscheck(False)` and `@cython.wraparound(False)` on performance-critical loops.
- Annotate loop variables (`cdef int i`) to keep loops entirely in C.
- Use `cpdef` for methods that are called from both Cython and Python — they generate both a C-fast-path and a Python wrapper.
- Profile before and after adding types — the first 10% of type declarations yield 90% of the speedup.

### Performance Considerations
- Typed loops can be 50–200x faster than CPython for pure arithmetic.
- Function call overhead drops from ~50 ns (Python) to ~1 ns (C) when using `cdef` functions.
- Type conversions (e.g., Python `int` to C `int`) add overhead — minimise crossing the Python/C boundary in hot loops.

### Interview Questions
- **Q**: What is the difference between `def`, `cdef`, and `cpdef` in Cython?  
  **A**: `def` is a Python-visible function (always Python call overhead). `cdef` is a C-only function (fast, invisible from Python). `cpdef` generates both paths — a C function for internal calls and a Python wrapper for external calls.
- **Q**: How does `@cython.boundscheck(False)` improve performance?  
  **A**: It skips the index-out-of-range check for each array access, turning a conditional branch into a raw memory load.

### Coding Challenges
- Convert a pure Python Mandelbrot set computation to Cython with static types and measure the speedup.
- Write a Cython function that computes the pairwise Euclidean distance matrix for two 2D arrays and compare performance with SciPy's `cdist`.

### Related Topics
- [.pyx files](#pyx-files)
- [C extensions](#c-extensions)
- [Compilation](#compilation)

---

## .pyx files

### What It Is
A `.pyx` file is the source file for Cython. It contains Python code extended with Cython-specific syntax: `cdef`, `cpdef`, `cimport`, `cdef class`, `cdef struct`, `cdef enum`, and `cimport` declarations. The `.pyx` file is processed by `cythonize` to produce a `.c` file, which is then compiled by a C compiler into a shared library (`.pyd` on Windows, `.so` on Linux/macOS).

### Why It Is Important
The `.pyx` file is where Cython's performance transformation happens. It is the developer's interface for deciding which parts of the code are Python and which are C. The `.pxd` file (analogous to `.h` in C) provides declarations for sharing C-level definitions across modules.

### How It Works Internally
The Cython compiler (`cython.py`) parses the `.pyx` file into an AST, resolves inferred and declared types, and performs code generation. Key stages:
1. **Tokenisation and parsing** (similar to CPython's `parser`).
2. **Type analysis and inference**: resolves `cdef`, `cpdef`, `ctypedef`, and implicit Python ↔ C conversions.
3. **C code generation**: emits a `.c` file that uses CPython C API calls where Python interaction is needed and raw C where types are static.
4. **Module initialisation**: generates `PyMODINIT_FUNC` for `PyInit_<module>`.

### Syntax
```cython
# mymodule.pyx
cdef extern from "some_c_lib.h":
    int c_function(int x)

def py_wrapper(x):
    return c_function(x)

# mymodule.pxd (declaration file)
cdef int helper(int x)

# cimport from another Cython module
from libc.math cimport sin, cos
```

### Beginner Examples
```cython
# hello.pyx
def say_hello(name):
    return f"Hello, {name}!"

# Use from Python:
# >>> from hello import say_hello
# >>> say_hello("World")
```

### Intermediate Examples
```cython
# utils.pyx
cimport cython
from libc.math cimport exp, log, sqrt

# Public Python function
def softmax(double[:] arr):
    cdef:
        int i
        int n = arr.shape[0]
        double total = 0.0
        double max_val = -1e300

    for i in range(n):
        if arr[i] > max_val:
            max_val = arr[i]

    for i in range(n):
        total += exp(arr[i] - max_val)

    for i in range(n):
        arr[i] = exp(arr[i] - max_val) / total

# utils.pxd
cdef double _internal_scale(double x, double factor)

# Multiple files
# fastmath.pyx
cimport utils

def compute(double[:] data):
    utils.softmax(data)
```

### Advanced Examples
```cython
# complex_module.pyx
from libcpp.vector cimport vector
from libcpp.string cimport string
from libc.string cimport memcpy

# Cython with C++ STL
cdef class Polynomial:
    cdef:
        vector[double] coeffs

    def __init__(self, list coeffs):
        self.coeffs = coeffs

    cpdef double evaluate(self, double x):
        cdef:
            double result = 0.0
            double power = 1.0
            int i

        for i in range(self.coeffs.size()):
            result += self.coeffs[i] * power
            power *= x
        return result

    cpdef Polynomial derivative(self):
        cdef Polynomial deriv = Polynomial.__new__(Polynomial)
        cdef int n = self.coeffs.size()
        if n <= 1:
            deriv.coeffs = [0.0]
        else:
            cdef vector[double] new_coeffs
            new_coeffs.resize(n - 1)
            for i in range(1, n):
                new_coeffs[i - 1] = self.coeffs[i] * i
            deriv.coeffs = new_coeffs
        return deriv

# Conditional compilation for platform-specific code
IF UNAME_SYSNAME == "Windows":
    cdef extern from "Windows.h":
        void Sleep(int ms)
ELSE:
    cdef extern from "unistd.h":
        void usleep(int microseconds)
```

### Real-World Use Cases
- **Splitting Python monoliths**: create `.pyx` files for performance-critical modules (e.g., image processing, JSON parsing) while keeping I/O and config in pure Python.
- **Cross-language projects**: `.pyx` files with `cdef extern from` headers allow direct integration of C/C++ libraries without a separate `ctypes` or `cffi` layer.
- **Scientific computing packages**: many PyPI packages include `.pyx` files that are compiled at install time via `setuptools` and `cythonize`.

### Common Mistakes
- Mixing `.py` and `.pyx` files without proper `# distutils: language = c++` or `# cython: language_level=3` header comments.
- Forgetting to include a `.pxd` file for shared declarations — each `.pyx` sees only its own declarations.
- Using Python features that Cython does not support (e.g., `yield from` in older Cython versions, certain metaclass patterns).

### Best Practices
- Keep `.pyx` files focused on hot loops and data structures; leave orchestration and I/O in `.py`.
- Use `cimport` to share C-level declarations between `.pyx` files and avoid duplication.
- Include `# cython: language_level=3, boundscheck=False, wraparound=False` as module-level comments for consistent behaviour.

### Performance Considerations
- The `.pyx` compilation pipeline adds incremental build time (seconds to minutes for large projects).
- Cython-generated `.c` files can be 10–100x larger than the original `.pyx`, increasing compilation time.
- Runtime speed of compiled `.pyx` code is comparable to hand-written C, minus the Python→C boundary crossing.

### Interview Questions
- **Q**: What is the purpose of the `.pxd` file in a Cython project?  
  **A**: The `.pxd` file provides C-level declarations (function signatures, type definitions) that can be shared across multiple `.pyx` files, analogous to a C header file.
- **Q**: Can you mix Python and Cython in the same project?  
  **A**: Yes, `.py` and `.pyx` files coexist. `.pyx` files are compiled to `.so`/`.pyd` files, and pure `.py` files are interpreted normally.

### Coding Challenges
- Create a `.pyx` module that wraps a simple C library (e.g., a Fibonacci calculator) and call it from Python.
- Structure a multi-file Cython project with `.pxd` declarations and build it with `setuptools`.

### Related Topics
- [Static type declarations](#static-type-declarations)
- [C extensions](#c-extensions)
- [Compilation](#compilation)

---

## C Extensions

### What It Is
C extensions are shared libraries written in C (or generated by Cython) that expose Python-callable functions and types. They are loaded by CPython via `import` and interact with the interpreter through the Python C API (`Python.h`). Every Cython-compiled `.so`/`.pyd` file is a C extension.

### Why It Is Important
C extensions provide the ultimate escape hatch for performance: they run at full native speed, can call any C library, and bypass the CPython interpreter entirely. Many core Python modules (`json`, `itertools`, `collections`, `socket`) are implemented as C extensions.

### How It Works Internally
A C extension module must implement `PyMODINIT_FUNC PyInit_<modulename>(void)` which:
1. Defines module-level functions as `PyMethodDef` arrays.
2. Defines types as `PyTypeObject` structs with custom `tp_*` slots.
3. Calls `PyModule_Create` to register the module in CPython's module cache.
4. The `PyMethodDef` maps Python function names to C function pointers of type `PyCFunction`.

### Syntax
```c
// mymodule.c
#include <Python.h>

static PyObject* my_add(PyObject *self, PyObject *args) {
    int a, b;
    if (!PyArg_ParseTuple(args, "ii", &a, &b))
        return NULL;
    return PyLong_FromLong(a + b);
}

static PyMethodDef MyMethods[] = {
    {"add", my_add, METH_VARARGS, "Add two integers"},
    {NULL, NULL, 0, NULL}
};

static struct PyModuleDef mymodule = {
    PyModuleDef_HEAD_INIT,
    "mymodule",
    NULL,
    -1,
    MyMethods
};

PyMODINIT_FUNC PyInit_mymodule(void) {
    return PyModule_Create(&mymodule);
}
```

### Beginner Examples
```cython
# Using Cython to create a C extension
# simple_ext.pyx
def greet(name: str) -> str:
    return f"Hello, {name}!"

# setup.py
from setuptools import setup, Extension
from Cython.Build import cythonize

setup(
    ext_modules=cythonize("simple_ext.pyx"),
)
```

### Intermediate Examples
```c
// A custom type in C
#include <Python.h>
#include <structmember.h>

typedef struct {
    PyObject_HEAD
    double x;
    double y;
} PointObject;

static int Point_init(PointObject *self, PyObject *args, PyObject *kwds) {
    if (!PyArg_ParseTuple(args, "dd", &self->x, &self->y))
        return -1;
    return 0;
}

static PyObject* Point_repr(PointObject *self) {
    return PyUnicode_FromFormat("Point(%g, %g)", self->x, self->y);
}

static PyMemberDef Point_members[] = {
    {"x", T_DOUBLE, offsetof(PointObject, x), 0, "x coordinate"},
    {"y", T_DOUBLE, offsetof(PointObject, y), 0, "y coordinate"},
    {NULL}
};

static PyTypeObject PointType = {
    PyVarObject_HEAD_INIT(NULL, 0)
    .tp_name = "mymodule.Point",
    .tp_basicsize = sizeof(PointObject),
    .tp_itemsize = 0,
    .tp_flags = Py_TPFLAGS_DEFAULT,
    .tp_init = (initproc)Point_init,
    .tp_members = Point_members,
    .tp_repr = (reprfunc)Point_repr,
};
```

### Advanced Examples
```c
// Buffer protocol for zero-copy array access
static int MyBuffer_getbuf(MyObject *self, Py_buffer *view, int flags) {
    return PyBuffer_FillInfo(view, (PyObject*)self,
                             self->data, self->size,
                             0, flags);
}

static PyBufferProcs MyBuffer_as_buffer = {
    (getbufferproc)MyBuffer_getbuf,
    (releasebufferproc)MyBuffer_releasebuf
};

// Exception handling
static PyObject* divide(PyObject *self, PyObject *args) {
    int a, b;
    if (!PyArg_ParseTuple(args, "ii", &a, &b))
        return NULL;
    if (b == 0) {
        PyErr_SetString(PyExc_ZeroDivisionError, "division by zero");
        return NULL;
    }
    return PyLong_FromLong(a / b);
}

// Memory management in extensions
static PyObject* create_array(PyObject *self, PyObject *args) {
    int size;
    if (!PyArg_ParseTuple(args, "i", &size))
        return NULL;

    double *arr = (double*)PyMem_Malloc(size * sizeof(double));
    if (!arr) return PyErr_NoMemory();

    PyObject *result = PyCapsule_New(arr, "my_array", free_capsule);
    return result;
}

static void free_capsule(PyObject *capsule) {
    double *arr = (double*)PyCapsule_GetPointer(capsule, "my_array");
    if (arr) PyMem_Free(arr);
}
```

### Real-World Use Cases
- **NumPy**: implemented primarily in C with a Python frontend; the C extension handles all array operations without interpreter overhead.
- **Database drivers**: `psycopg2` and `mysql-connector` are C extensions that talk directly to PostgreSQL/MySQL wire protocols.
- **Image processing**: `Pillow` uses C extensions to call libjpeg, libpng, etc., decoding images at C speed.

### Common Mistakes
- Forgetting to increment/decrement reference counts — causes crashes or memory leaks.
- Ignoring the Python GIL — calling blocking C code while holding the GIL stalls all Python threads.
- Using `PyArg_ParseTuple` without `&` before pointer arguments — silent corruption.

### Best Practices
- Use `PyMem_Malloc`/`PyMem_Free` instead of `malloc`/`free` for memory that might interact with Python objects.
- Release the GIL with `Py_BEGIN_ALLOW_THREADS` / `Py_END_ALLOW_THREADS` around long-running C code.
- Wrap all Python-facing APIs in `try`/`except` at the Python level and return `NULL` with `PyErr_SetString` on errors.

### Performance Considerations
- C extensions avoid the interpreter loop entirely — calls from Python cost only the `PyCFunction` dispatch (~30 ns).
- Crossing the Python↔C boundary has overhead from argument parsing and type conversion; batch work in large chunks.
- C extensions are compiled to native machine code and are fully optimised by the C compiler (-O2 or -O3).

### Interview Questions
- **Q**: What is `PyCapsule` and when would you use it?  
  **A**: `PyCapsule` wraps a `void*` pointer for safely passing C resources (allocated memory, library handles) through Python code without leaking.
- **Q**: How does a C extension avoid holding the GIL during a long computation?  
  **A**: By using `Py_BEGIN_ALLOW_THREADS` and `Py_END_ALLOW_THREADS` macros around the computation, releasing the GIL temporarily.

### Coding Challenges
- Write a minimal C extension that computes the Fibonacci sequence and returns a Python list.
- Create a C extension type that implements `__len__`, `__getitem__`, and `__setitem__` for a fixed-size integer array.

### Related Topics
- [Static type declarations](#static-type-declarations)
- [Compilation](#compilation)
- [Advanced Topics (C extensions)](#advanced-topics---pattern-matching-ast-c-extensions-pep-deep-dives)

---

## Compilation

### What It Is
Cython compilation is the process of translating `.pyx` (or `.py` with annotations) files into C code and then compiling that C code into a native shared library. The build is typically orchestrated by `setuptools` via `cythonize()` or a `Makefile`.

### Why It Is Important
Proper compilation is essential for using Cython effectively. Different platforms require different compiler flags, and the build system must know where to find Python headers and libraries. The compilation mode also controls optimisation, debugging symbols, and linking.

### How It Works Internally
1. **Cython (frontend)**: The `cython` command or `Cython.Build.cythonize` reads `.pyx` files, applies the Cython compiler, and generates `.c` files.
2. **C compiler (backend)**: The C compiler (`gcc`, `clang`, `MSVC`) compiles the `.c` file into a `.o` object file and links it against `libpython` to produce a `.pyd`/`.so`.
3. **Import**: When Python imports the module, CPython calls `PyInit_<module>`, which registers functions and types.

### Syntax
```python
# setup.py
from setuptools import setup, Extension
from Cython.Build import cythonize
import numpy as np

extensions = [
    Extension(
        "fastmath",
        ["fastmath.pyx"],
        include_dirs=[np.get_include()],
        libraries=["m"],
        define_macros=[("CYTHON_CLINE_IN_TRACEBACK", "0")]
    ),
]

setup(
    ext_modules=cythonize(
        extensions,
        language_level="3",
        annotate=True,           # produces HTML annotation
        compiler_directives={
            "boundscheck": False,
            "wraparound": False,
        }
    ),
)

# Build command:
# python setup.py build_ext --inplace
```

### Beginner Examples
```bash
# Simplest compilation
cythonize -i mymodule.pyx

# With annotations
cythonize -i -a mymodule.pyx

# Using pyximport (for quick prototyping)
import pyximport
pyximport.install()
import mymodule
```

### Intermediate Examples
```python
# build_ext.py
from setuptools import setup, Extension
from Cython.Build import cythonize
import sys

def get_extra_compile_args():
    if sys.platform == 'win32':
        return ['/O2', '/openmp']
    return ['-O3', '-march=native', '-fopenmp']

ext = Extension(
    "fast_processor",
    sources=["fast_processor.pyx"],
    extra_compile_args=get_extra_compile_args(),
    extra_link_args=[],
)

setup(
    ext_modules=cythonize(
        [ext],
        language_level="3",
        annotate=True,
        gdb_debug=False,
    )
)
```

### Advanced Examples
```python
# build.py - Full build pipeline
from Cython.Build import cythonize
from Cython.Compiler import Options
from setuptools import setup, Extension
import os
import sysconfig

# Global Cython options
Options.docstrings = False

# Per-module directives
compiler_directives = {
    'binding': False,
    'boundscheck': False,
    'wraparound': False,
    'initializedcheck': False,
    'cdivision': True,
    'embedsignature': False,
}

modules = [
    Extension(
        "engine.core",
        ["engine/core.pyx"],
        include_dirs=[
            sysconfig.get_config_var('INCLUDEPY'),
            os.path.join('vendor', 'include')
        ],
        libraries=['pthread'],
    ),
    Extension(
        "engine.io",
        ["engine/io.pyx"],
    ),
]

ext_modules = cythonize(
    modules,
    language_level="3",
    compiler_directives=compiler_directives,
    nthreads=4,              # parallel compilation
    force=True,              # rebuild even if .pyx is older
)

setup(
    name='engine',
    ext_modules=ext_modules,
    zip_safe=False,
)

# Using Cython with pkg-config
import subprocess
flags = subprocess.check_output(
    ['pkg-config', '--cflags', '--libs', 'opencv4']
).decode().strip().split()

cv_ext = Extension(
    "vision.processor",
    ["vision/processor.pyx"],
    extra_compile_args=flags[:len(flags)//2],
    extra_link_args=flags[len(flags)//2:],
)
```

### Real-World Use Cases
- **pip-installable packages**: specify `cythonize` in `setup.py` so users get compiled extensions when they run `pip install`.
- **Windows + MSVC builds**: ensure `vcvarsall.bat` is loaded so `setuptools` finds the compiler; use `/Ox` for optimisation.
- **Cross-compilation** (e.g., Raspberry Pi): compile `.pyx` to `.c` on a build server, then cross-compile `.c` to ARM `.so`.

### Common Mistakes
- Not installing `Cython` before running `setup.py` — `cythonize` is not available.
- Forgetting `language_level="3"` — defaults to Python 2 in older Cython versions.
- Using `pyximport` in production — it compiles at import time, slowing startup and requiring a compiler on the target machine.

### Best Practices
- Use `annotate=True` in early development to identify yellow lines (Python interaction) that need typing.
- Keep Cython compilation in a separate `build` step for CI — don't rely on `pyximport`.
- Set `CYTHON_TRACE=1` and `CYTHON_TRACE_NOGIL=1` macros only for profile builds; leave them off in release builds.
- Pin Cython version in `setup.py` with `setup_requires=['cython>=3.0']`.

### Performance Considerations
- Compiled extensions are as fast as hand-written C for compute-heavy loops.
- Startup time increases slightly because loading a `.pyd` requires dynamic linking.
- The `.so`/`.pyd` files are platform-specific — distribute wheels for each platform (use `cibuildwheel` or manylinux).

### Interview Questions
- **Q**: What is the purpose of the `-a` flag in `cythonize -a`?  
  **A**: It produces an HTML annotation file that colours each line by how much it interacts with Python (white = pure C, yellow = Python API calls).
- **Q**: What does `cythonize` do with `compiler_directives={'boundscheck': False}`?  
  **A**: It disables runtime bounds checks on array indexing for all functions in the module, trading safety for speed.

### Coding Challenges
- Set up a minimal Cython project with `setup.py`, one `.pyx` file, and a `.pxd` declaration file; build and import it.
- Write a `Makefile` that compiles a Cython module with optimisations (`-O3 -march=native`) and measures the speedup over the pure Python equivalent.

### Related Topics
- [Static type declarations](#static-type-declarations)
- [.pyx files](#pyx-files)
- [C extensions](#c-extensions)
