# CPython Internals - Bytecode, PyObject, interpreter loop

## Introduction
CPython is the reference implementation of Python, written in C. Understanding its internals — bytecode, the `PyObject` structure, and the interpreter eval loop — is invaluable for writing high-performance Python, debugging tricky issues, and developing C extensions. This document explains how Python source code is compiled to bytecode, how objects are represented in memory, and how the interpreter executes those instructions.

## Bytecode (dis module)

### What It Is
Bytecode is an intermediate, platform-independent representation of Python source code. The compiler (`compile()` builtin or `py_compile`) translates source into a `code` object containing a sequence of 2-byte opcodes and optional arguments. The `dis` module disassembles these code objects into human-readable mnemonics.

### Why It Is Important
Bytecode reveals exactly what the interpreter does with your code. Understanding bytecode helps you:
- Spot unnecessary operations (e.g., repeated global lookups).
- Understand why certain constructs are faster (e.g., local variable access vs. global).
- Debug obscure bugs that only appear at the instruction level.

### How It Works Internally
1. **Compilation**: `Python/compile.c` visits the AST and emits `struct instr` sequences stored in a `PyCodeObject`.
2. **Code object**: Contains `co_code` (bytecode bytes), `co_consts`, `co_names`, `co_varnames`, `co_lnotab` (line number table), and `co_stacksize`.
3. **Opcodes**: Each opcode is one byte (0–255). Some opcodes (e.g., `LOAD_FAST`, `LOAD_CONST`) are followed by a one-byte or two-byte argument.
4. **Bytecode is immutable**: Once compiled, the bytecode of a function does not change until the function is redefined.

### Syntax
```python
import dis

# Disassemble a function
def f(a, b):
    return a + b

dis.dis(f)

# Disassemble a code object
code = compile('x = 1 + 2', '<string>', 'exec')
dis.dis(code)

# Show bytecode bytes
print(f.__code__.co_code)

# Show detailed info
dis.show_code(f)
```

### Beginner Examples
```python
import dis

def greet(name):
    greeting = f"Hello, {name}!"
    return greeting

dis.dis(greet)

# Output shows:
# LOAD_FAST     name
# FORMAT_VALUE  0
# LOAD_CONST    'Hello, '
# BUILD_STRING  2
# STORE_FAST    greeting
# LOAD_FAST     greeting
# RETURN_VALUE
```

### Intermediate Examples
```python
import dis

# Compare local vs global variable access
global_var = 42

def use_global():
    return global_var

def use_local():
    local_var = 42
    return local_var

print("Global access:")
dis.dis(use_global)
print("\nLocal access:")
dis.dis(use_local)

# Disassembling comprehensions
def comprehension():
    return [x**2 for x in range(10)]

dis.dis(comprehension)

# Analysing with dis.COMPILER_FLAG
import sys
print(f'Optimization flags: {sys.flags.optimize}')

# Using dis for optimisation insight
def slow_loop(n):
    result = 0
    for i in range(n):
        result += i ** 2 + i * 3
    return result

dis.dis(slow_loop)
```

### Advanced Examples
```python
import dis
import sys
import types

# Modifying bytecode at runtime (advanced)
def add_one(x):
    return x + 1

# Get bytecode bytes
original_code = add_one.__code__
print(f'Original bytecode: {original_code.co_code.hex()}')

# Creating a new code object with modified constants
new_consts = list(original_code.co_consts)
for i, c in enumerate(new_consts):
    if c == 1:
        new_consts[i] = 100

new_code = types.CodeType(
    original_code.co_argcount,
    original_code.co_posonlyargcount,
    original_code.co_kwonlyargcount,
    original_code.co_nlocals,
    original_code.co_stacksize,
    original_code.co_flags,
    original_code.co_code,
    tuple(new_consts),           # modified constants
    original_code.co_names,
    original_code.co_varnames,
    original_code.co_filename,
    original_code.co_name,
    original_code.co_firstlineno,
    original_code.co_lnotab,
    original_code.co_freevars,
    original_code.co_cellvars
)

add_one.__code__ = new_code
print(add_one(5))  # 105

# Instruction-level analysis
def count_ops(func, opname):
    """Count occurrences of a specific opcode in a function."""
    import opcode
    op = opcode.opmap[opname]
    code = func.__code__.co_code
    count = 0
    i = 0
    while i < len(code):
        if code[i] == op:
            count += 1
        i += 2 if code[i] >= dis.HAVE_ARGUMENT else 1
    return count

def example():
    x = 1
    y = 2
    z = x + y
    return z

print(count_ops(example, 'LOAD_CONST'))
```

### Real-World Use Cases
- **Performance analysis**: compare bytecode for `list comprehension` vs `map` with lambda to see why one is faster.
- **Bytecode patching**: frameworks like `pytest` and `coverage` modify bytecode at import time to inject instrumentation.
- **Reverse engineering**: decompile obfuscated Python bytecode for forensic analysis.

### Common Mistakes
- Assuming `dis.dis` output is deterministic across Python versions — opcodes change between 3.x releases.
- Modifying `co_code` directly without updating `co_lnotab` — causes traceback line number errors.
- Forgetting that `dis.HAVE_ARGUMENT` determines whether an opcode consumes an argument byte.

### Best Practices
- Use `dis.dis` with `adaptive=True` on Python 3.13+ to see specialised bytecode.
- When optimising hot loops, examine the bytecode for repeated `LOAD_GLOBAL` or `LOAD_ATTR` and replace them with local bindings.
- Study the bytecode of built-in functions (`map`, `filter`, `sorted`) to understand why they are faster than equivalent Python loops.

### Performance Considerations
- Bytecode dispatch in the eval loop is a `switch` statement — it is fast, but each opcode dispatch costs ~5–10 CPU cycles.
- `LOAD_FAST` (local variable) is a single array index; `LOAD_GLOBAL` involves a dict lookup — 2–3x slower.
- Bytecode specialisation in Python 3.11+ (`adaptive` and `specialising` opcodes) can double execution speed for hot functions.

### Interview Questions
- **Q**: What is the difference between `LOAD_FAST`, `LOAD_GLOBAL`, and `LOAD_DEREF`?  
  **A**: `LOAD_FAST` accesses a local variable by index in `co_varnames`. `LOAD_GLOBAL` does a dict lookup in the global namespace. `LOAD_DEREF` accesses a closure/free variable from the cell object.
- **Q**: How does Python 3.11's "zero-cost exception handling" change the bytecode?  
  **A**: Python 3.11 deferred exception table entries, removing `SETUP_FINALLY` from the hot path and reducing bytecode size.

### Coding Challenges
- Write a function that takes a Python source line and returns the number of bytecode instructions it compiles to.
- Implement a simple bytecode optimiser that replaces `LOAD_CONST 0; LOAD_CONST 1; BINARY_ADD` with a single `LOAD_CONST (0+1)` when both constants are literals.

### Related Topics
- [PyObject structure](#pyobject-structure)
- [Interpreter loop](#interpreter-loop)
- [Cython](#cython---static-types-pyx-files-c-extensions-compilation)

---

## PyObject Structure

### What It Is
`PyObject` is the fundamental C struct underlying every Python object. All Python objects — integers, strings, functions, classes, modules — are pointers to structures that begin with the `PyObject_HEAD` macro. This header contains a reference count and a type pointer, giving every object its identity, type, and lifecycle.

### Why It Is Important
Understanding `PyObject` clarifies:
- Why every Python value is a pointer (memory overhead).
- How `type()` and `isinstance()` work (by comparing `ob_type`).
- How the memory allocator (`PyMem_Malloc`) manages object lifetime.
- How to write correct C extensions that manage reference counts.

### How It Works Internally
```c
// Minimal PyObject
typedef struct _object {
    Py_ssize_t ob_refcnt;      // reference count
    PyTypeObject *ob_type;     // pointer to type object
} PyObject;

// PyObject_HEAD expands to the above
// Variable-length objects add:
typedef struct {
    PyObject ob_base;          // PyObject_HEAD
    Py_ssize_t ob_size;        // number of items
} PyVarObject;

// Example for PyLongObject (int):
struct _longobject {
    PyObject_HEAD
    Py_ssize_t ob_digit[1];    // variable-length digit array
};

// Example for PyListObject:
typedef struct {
    PyObject_VAR_HEAD
    PyObject **ob_item;        // pointer to array of PyObject*
    Py_ssize_t allocated;      // allocated capacity
} PyListObject;
```

### Syntax
PyObject is a C concept and is not directly visible in Python, but the following inspection tools reveal its layout:

```python
import sys
import ctypes

# Size of different objects
print(f'Int size: {sys.getsizeof(42)}')          # 28 bytes (32-bit digits)
print(f'List size (empty): {sys.getsizeof([])}')  # 56 bytes
print(f'Dict size (empty): {sys.getsizeof({})}')  # 72 bytes
print(f'Tuple size (empty): {sys.getsizeof(())}') # 40 bytes

# Object identity via id() — the memory address
a = object()
print(f'Memory address: {id(a):#x}')

# Using ctypes to read refcount
def refcount(obj):
    return ctypes.c_long.from_address(id(obj)).value

x = []
print(f'Refcount: {refcount(x)}')
```

### Beginner Examples
```python
import sys
import ctypes

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(1, 2)

# Every object has a refcount and a type
print(f'Type: {type(p)}')
print(f'Refcount: {sys.getrefcount(p) - 1}')  # subtract 1 for getrefcount arg
print(f'Size: {sys.getsizeof(p)} bytes')
print(f'Instance dict size: {sys.getsizeof(p.__dict__)} bytes')
```

### Intermediate Examples
```python
import sys
import ctypes
import gc

# Inspect object layout
def object_layout(obj):
    addr = id(obj)
    print(f'Address: 0x{addr:x}')
    print(f'Size: {sys.getsizeof(obj)}')

    # Read refcount via ctypes (64-bit)
    refcount = ctypes.c_ssize_t.from_address(addr).value
    print(f'Refcount: {refcount}')

    # Read type pointer (second 8 bytes on 64-bit)
    type_ptr = ctypes.c_void_p.from_address(addr + 8).value
    print(f'Type: {type(obj)} at 0x{type_ptr:x}')

# Compare mutable vs immutable
a = 42
b = a
print("Small int sharing:")
print(f'a is b: {a is b}')
print(f'Refcount of 42: {sys.getrefcount(42)}')

# Slots vs dict
class WithSlots:
    __slots__ = ('x', 'y')
    def __init__(self, x, y):
        self.x = x
        self.y = y

class NoSlots:
    def __init__(self, x, y):
        self.x = x
        self.y = y

print(f'WithSlots size: {sys.getsizeof(WithSlots(1, 2))}')
print(f'NoSlots size: {sys.getsizeof(NoSlots(1, 2))}')
```

### Advanced Examples
```python
import sys
import ctypes
import gc

# Creating a read-only ctypes view into PyObject
class PyObjectView:
    def __init__(self, obj):
        self.obj = obj
        self.addr = id(obj)

    @property
    def refcount(self):
        return ctypes.c_ssize_t.from_address(self.addr).value

    @property
    def type_ptr(self):
        return ctypes.c_void_p.from_address(self.addr + 8).value

    @property
    def type_name(self):
        return type(self.obj).__name__

    def dump(self):
        print(f'PyObject@{self.addr:#x}')
        print(f'  ob_refcnt = {self.refcount}')
        print(f'  ob_type   = 0x{self.type_ptr:x} ({self.type_name})')

# Demonstrate interning
view1 = PyObjectView("hello")
view2 = PyObjectView("hello")
print(f"Same string literal? {view1.addr == view2.addr}")
print(f"Refcount of 'hello': {view1.refcount}")

# Track object creation via gc
class Tracked:
    def __init__(self):
        self.created = id(self)
        self.view = PyObjectView(self)

t = Tracked()
print(f'Tracked object at: {hex(t.created)}')
print(f'Current refcount: {t.view.refcount}')
```

### Real-World Use Cases
- **Memory profiling**: knowing that each `int` in a list costs 28 bytes + 8 bytes pointer helps estimate memory budgets.
- **C extension development**: when writing extensions, you must allocate `PyObject` structs correctly — misuse causes segfaults.
- **Performance tuning**: using `__slots__` changes the object from having `__dict__` (a PyDictObject) to `PyObject**` offsets, saving 40+ bytes per instance.

### Common Mistakes
- Treating `sys.getsizeof` as total memory — it only measures the struct itself, not referenced objects.
- Forgetting that `PyObject*` is a pointer; storing Python objects in a C array requires `PyObject**`, not `PyObject[]`.
- Assuming `id()` is unique across the process lifetime — CPython reuses memory addresses after deallocation.

### Best Practices
- Use `__slots__` in classes with millions of instances to reduce memory by 50–70%.
- Understand that small integers (-5 to 256) are pre-allocated singletons in CPython.
- For large homogeneous arrays, prefer `array.array` or `numpy.ndarray` — they store C primitives directly, not `PyObject*` pointers.

### Performance Considerations
- Each `PyObject` has a minimum overhead of 16 bytes (32-bit) or 32 bytes (64-bit, due to alignment).
- Pointer chasing: accessing an attribute goes `PyObject* → ob_type → tp_getattro → dict lookup`, which can be 5–10 indirections.
- Object allocation via `PyObject_Malloc` is fast (free-list based for small objects), but millions of allocations still add up.

### Interview Questions
- **Q**: What is stored in the `PyObject` header?  
  **A**: `ob_refcnt` (reference count, `Py_ssize_t`) and `ob_type` (pointer to `PyTypeObject`).
- **Q**: How does `isinstance` work at the C level?  
  **A**: It follows `ob_type` and walks the MRO linked list (`tp_mro`), comparing each entry to the target type.

### Coding Challenges
- Use `ctypes` to traverse an object's `__dict__` by reading `tp_dictoffset` from the type and dereferencing the pointer.
- Estimate the total memory footprint of a nested data structure (list of dicts of ints) using `sys.getsizeof` recursively.

### Related Topics
- [Reference counting](#reference-counting)
- [Bytecode (dis module)](#bytecode-dis-module)
- [Cython](#cython---static-types-pyx-files-c-extensions-compilation)

---

## Interpreter Loop

### What It Is
The interpreter loop (the "eval loop") is the heart of CPython, located in `Python/ceval.c`. It is a C `switch` statement that repeatedly fetches the next bytecode instruction, decodes it, and dispatches to the corresponding C handler. This loop is what actually executes your Python code.

### Why It Is Important
The interpreter loop is the bottleneck for all Python code. Every Python operation — addition, function call, attribute access — goes through this loop. Understanding its structure helps you write code that minimises bytecode dispatches and leverage CPython's internal optimisations.

### How It Works Internally
The main function is `_PyEval_EvalFrameDefault`. Simplified pseudocode:

```c
PyObject*
_PyEval_EvalFrameDefault(PyFrameObject *f, int throwflag) {
    PyObject *retval = NULL;
    PyCodeObject *co = f->f_code;
    unsigned char *next_instr = co->co_code + f->f_lasti;

    for (;;) {
        unsigned char opcode = *next_instr++;
        Py_ssize_t oparg = 0;
        if (opcode >= HAVE_ARGUMENT) {
            oparg = *next_instr++;
            oparg |= (*next_instr++) << 8;
        }

        switch (opcode) {
            case NOP:
                break;
            case LOAD_FAST:
                Py_INCREF(fastlocals[oparg]);
                PUSH(fastlocals[oparg]);
                break;
            case BINARY_ADD:
                PyObject *right = POP();
                PyObject *left = TOP();
                PyObject *sum = PyNumber_Add(left, right);
                SET_TOP(sum);
                Py_DECREF(left); Py_DECREF(right);
                break;
            case RETURN_VALUE:
                retval = POP();
                goto exit;
            // ... 100+ more cases
        }
    }

exit:
    return retval;
}
```

Key components:
- **Frame**: `PyFrameObject` holds the local namespace, global namespace, instruction pointer, and evaluation stack.
- **Value stack**: a contiguous C array (`f->f_valuestack`) used for intermediate results.
- **Fast locals**: `f->f_localsplus` caches local variables in an array indexed by `LOAD_FAST` oparg.
- **PEP 523**: added a frame evaluation API allowing JIT compilers (like Pyjion and Cinder) to replace the eval loop.

### Syntax
```python
import sys

# Frame objects are visible during debugging
import inspect

def inner():
    return inspect.currentframe()

def outer():
    return inner()

frame = outer()
print(f'Function: {frame.f_code.co_name}')
print(f'Line number: {frame.f_lineno}')
print(f'Locals: {frame.f_locals}')
print(f'Instruction pointer: {frame.f_lasti}')
```

### Beginner Examples
```python
import sys
import dis

def simple_function(a, b):
    c = a + b
    return c

# Trace execution with settrace
def trace_calls(frame, event, arg):
    if event == 'call':
        print(f'Calling {frame.f_code.co_name}')
    elif event == 'line':
        print(f'Line {frame.f_lineno}: {dis.Bytecode(frame.f_code).dis()}')
    elif event == 'return':
        print(f'Return from {frame.f_code.co_name}')
    return trace_calls

sys.settrace(trace_calls)
simple_function(3, 4)
sys.settrace(None)
```

### Intermediate Examples
```python
import sys
import dis

# Counting bytecode instructions
def count_instructions(func):
    code = func.__code__
    bytecode = dis.Bytecode(code)
    instructions = list(bytecode)
    print(f'{func.__name__}: {len(instructions)} instructions')
    for instr in instructions[:10]:
        print(f'  {instr.opname}')

def fast():
    x = 1
    y = 2
    return x + y

def slow():
    return globals()['x'] + globals()['y']

x, y = 1, 2
count_instructions(fast)
count_instructions(slow)

# Frame reuse (Python 3.11+)
def recursive_factorial(n):
    if n <= 1:
        return 1
    return n * recursive_factorial(n - 1)

for depth in [1, 10, 100]:
    f = recursive_factorial.__code__
    print(f'Depth {depth}: co_stacksize = {f.co_stacksize}')
```

### Advanced Examples
```python
import sys
import types
import dis

# Creating a custom frame with types.FrameType (Python 3.11+)
# Note: FrameType constructor is internal and version-specific

# Profiling eval loop overhead
import time

def measure_dispatch_overhead():
    """Measure the overhead of bytecode dispatch vs native C."""
    # Empty loop in Python
    def py_loop(n):
        i = 0
        while i < n:
            i += 1

    start = time.perf_counter()
    py_loop(10_000_000)
    t_py = time.perf_counter() - start

    # Each iteration is roughly 3-4 bytecode dispatches
    dispatches = 10_000_000 * 4
    ns_per_dispatch = (t_py / dispatches) * 1e9
    print(f'{ns_per_dispatch:.1f} ns per dispatch')
    print(f'Loop overhead is primarily dispatch, not arithmetic')

measure_dispatch_overhead()

# Understanding the effect of inlining (Python 3.12+)
def hot_function():
    total = 0
    for i in range(1000):
        total += i * 2
    return total

def cold_wrapper():
    return hot_function()

# Python 3.11+ inlines some common opcodes
print("Hot function bytecode:")
dis.dis(hot_function)
```

### Real-World Use Cases
- **Performance optimisation**: knowing the eval loop helps you write code that reduces the number of dispatches — e.g., moving a `len()` call outside a loop.
- **Custom interpreters**: projects like `pyston` and `cinder` replace the eval loop with a JIT to achieve speedups of 20–50%.
- **Debugging tools**: frame introspection is used by `pdb`, `faulthandler`, `sentry`, and `signal` handlers to produce accurate stack traces.

### Common Mistakes
- Assuming that the eval loop is the same in PyPy (it's not — PyPy is a tracing JIT with no bytecode dispatch loop).
- Writing Python code that creates many short-lived frames (deep recursion or many small function calls) — frame creation is not free.
- Using `sys.settrace` in production — it forces the interpreter into "debug mode", disabling all optimisation and slowing execution by 10–100x.

### Best Practices
- Inline small functions manually when profiling shows high frame-creation overhead.
- Use `functools.lru_cache` on pure functions to bypass the eval loop entirely for repeated calls.
- For numerical hot spots, use NumPy or Numba to push execution into native C code, completely escaping the eval loop.

### Performance Considerations
- Each bytecode dispatch costs ~40–80 CPU cycles in the eval loop switch statement.
- Python 3.11's "adaptive" opcodes (e.g., `BINARY_ADD_ADAPTIVE`) can specialise to type-specific handlers, reducing dispatch overhead.
- The eval loop is bound by the GIL for CPU-bound code; multiprocessing avoids the GIL but creates separate interpreter processes.

### Interview Questions
- **Q**: What is the "value stack" in the interpreter loop?  
  **A**: It is a C array where intermediate results are pushed and popped during bytecode execution; distinct from the C call stack.
- **Q**: How did Python 3.11 reduce eval loop overhead compared to 3.10?  
  **A**: Python 3.11 introduced zero-cost exception handling (removed `SETUP_FINALLY` from bytecode), frame reuse (fewer allocations), and specialised opcodes for common operations.

### Coding Challenges
- Implement a minimal bytecode interpreter for a tiny subset of Python (push, pop, add, return) in C.
- Write a Python function that, given a `.pyc` file, reads the code object, disassembles it, and prints the instruction count per line.

### Related Topics
- [Bytecode (dis module)](#bytecode-dis-module)
- [PyObject structure](#pyobject-structure)
- [Numba](#numba---jit-decorator-vectorize-gpu-acceleration)
