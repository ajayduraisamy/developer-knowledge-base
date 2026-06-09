# Advanced Topics - Pattern matching, AST, C extensions, PEP deep dives

## Introduction
Python's evolution as a language is driven by PEPs (Python Enhancement Proposals) that introduce new syntax, libraries, and runtime features. This document covers some of the most impactful advanced topics: structural pattern matching (Python 3.10+), the AST module for code analysis and transformation, writing C extensions for maximum performance, and a deep dive into the PEPs that have shaped modern Python.

## Pattern Matching (match/case 3.10+)

### What It Is
Structural pattern matching, introduced in PEP 634–636 (Python 3.10), provides a `match` statement that compares a subject against a series of patterns. Patterns can match literal values, types, sequences, mappings, objects, and combinations of these with guards. It is similar to `switch`/`case` in other languages but much more powerful.

### Why It Is Important
Pattern matching replaces long if-elif chains with declarative, readable code. It is especially useful for parsing structured data (JSON, AST nodes, command-line arguments) and for implementing state machines, protocol handlers, and algebraic data types.

### How It Works Internally
The `match` statement is compiled by CPython's bytecode compiler into a series of specialised opcodes. The compiler analyses patterns and generates efficient comparison code:
1. **Subject evaluation**: the subject expression is evaluated once and stored.
2. **Pattern matching**: each case's pattern is tried in order. For literals, this is an equality check. For class patterns, `isinstance` is used. For sequence patterns, length is checked first.
3. **Guard evaluation**: if a pattern matches and a guard (`if condition`) is present, the guard is evaluated.
4. **Variable binding**: matched sub-patterns bind names in the local scope.
5. **Short-circuit**: once a case matches, no further cases are tried.

### Syntax
```python
match value:
    case 0:
        print("zero")
    case 1 | 2:
        print("one or two")
    case int(n):
        print(f"integer {n}")
    case [x, y]:
        print(f"list with two elements: {x}, {y}")
    case {"key": value}:
        print(f"dict with key: {value}")
    case _:
        print("default")

# Guard
match point:
    case (x, y) if x == y:
        print("on diagonal")
    case (x, y):
        print(f"({x}, {y})")
```

### Beginner Examples
```python
# Simple literal matching
def describe_status(code):
    match code:
        case 200:
            return "OK"
        case 404:
            return "Not Found"
        case 500:
            return "Server Error"
        case _:
            return "Unknown"

# Type matching
def process(value):
    match value:
        case int(n):
            return f"Integer: {n}"
        case float(f):
            return f"Float: {f}"
        case str(s):
            return f"String: {s}"
        case _:
            return "Unknown type"

# Sequence unpacking
def handle_command(cmd):
    match cmd.split():
        case ["quit"]:
            exit()
        case ["hello", name]:
            print(f"Hello, {name}!")
        case ["add", x, y]:
            print(int(x) + int(y))
        case _:
            print("Unknown command")
```

### Intermediate Examples
```python
# Matching nested structures
def parse_config(config):
    match config:
        case {"database": {"host": host, "port": port}}:
            print(f"Connecting to {host}:{port}")
        case {"database": {"host": host}}:
            print(f"Connecting to {host}:5432 (default)")
        case {"logging": {"level": level}} if level in ("DEBUG", "INFO", "ERROR"):
            print(f"Log level: {level}")
        case _:
            print("Invalid config")

# Class pattern matching
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

@dataclass
class Circle:
    center: Point
    radius: float

def describe_shape(shape):
    match shape:
        case Point(x=0, y=0):
            return "Origin"
        case Point(x=x, y=y):
            return f"Point({x}, {y})"
        case Circle(center=Point(x=0, y=0), radius=r):
            return f"Circle at origin, radius {r}"
        case Circle():
            return f"Circle at {shape.center}"
        case _:
            return "Unknown shape"

# Matching with OR patterns
def classify_number(n):
    match n:
        case 0 | 1 | 2:
            return "small"
        case 3 | 4 | 5:
            return "medium"
        case 6 | 7 | 8 | 9:
            return "large"
        case _:
            return "huge"
```

### Advanced Examples
```python
# AST pattern matching (Python 3.10+)
import ast

def simplify_binary_op(node):
    match node:
        case ast.BinOp(left=ast.Constant(value=x), op=ast.Add(), right=ast.Constant(value=y)):
            return ast.Constant(value=x + y)
        case ast.BinOp(left=ast.Constant(value=x), op=ast.Mult(), right=ast.Constant(value=y)):
            return ast.Constant(value=x * y)
        case ast.BinOp(left=left, op=op, right=right):
            return ast.BinOp(left=simplify_binary_op(left), op=op, right=simplify_binary_op(right))
        case _:
            return node

# Matching with capture and wildcard
def analyze_http_request(method, path, headers):
    match (method, path, headers):
        case ("GET", path, _) if "auth" in headers:
            return f"Authenticated GET {path}"
        case ("POST", "/api/data", {"Content-Type": "application/json"}):
            return "JSON POST to /api/data"
        case ("GET", _, _):
            return "Anonymous GET"
        case _:
            return "Unknown request"

# State machine implementation
class StateMachine:
    def __init__(self):
        self.state = "idle"

    def transition(self, event):
        match (self.state, event):
            case ("idle", "start"):
                self.state = "running"
            case ("running", "pause"):
                self.state = "paused"
            case ("running", "stop"):
                self.state = "stopped"
            case ("paused", "resume"):
                self.state = "running"
            case ("paused", "stop"):
                self.state = "stopped"
            case _:
                raise ValueError(f"Invalid transition from {self.state} on {event}")

# JSON parsing with match
def extract_values(data):
    match data:
        case {"type": "user", "name": str(name), "id": int(id)}:
            return {"kind": "user", "name": name, "id": id}
        case {"type": "group", "members": [*names]}:
            return {"kind": "group", "count": len(names)}
        case {"type": "message", "text": str(text), **rest}:
            return {"kind": "message", "length": len(text)}
        case _:
            return None
```

### Real-World Use Cases
- **HTTP router**: match request method and path against defined routes.
- **JSON transformation**: pattern-match JSON structures for ETL pipelines.
- **AST rewriting**: match and transform AST nodes in code linters and transpilers.
- **Protocol parsing**: parse binary or text protocols with sequence patterns.

### Common Mistakes
- Forgetting that `_` is a wildcard but also a valid variable name — it does not bind.
- Using `|` (OR) inside a class pattern incorrectly — `case Point(0 | 1, 0 | 1):` matches (0,0), (0,1), (1,0), (1,1).
- Assuming patterns are evaluated left-to-right with short-circuit — OR patterns try each alternative.
- Matching against a variable that shadows an existing name — use dotted names for constants.

### Best Practices
- Use `match` for complex branching on shape/structure, not for simple integer switches (an `if` is clearer).
- Use guards (`if condition`) for additional constraints that cannot be expressed in the pattern.
- Prefer class patterns over raw `isinstance` checks for cleaner type dispatch.
- Use `case _` as the default case to avoid accidentally falling through.

### Performance Considerations
- `match` compiles to efficient bytecode — typically as fast as an equivalent if-elif chain.
- Complex nested patterns may generate more bytecode than manual checks but are rarely the bottleneck.
- The subject expression is evaluated only once, even across multiple cases.

### Interview Questions
- **Q**: How does pattern matching differ from a simple switch statement?  
  **A**: Pattern matching supports destructuring, type checking, guards, and nested patterns — not just literal equality.
- **Q**: What is the difference between `case [x, y]` and `case (x, y)`?  
  **A**: `[x, y]` matches a sequence of exactly two elements (list, tuple, etc.), while `(x, y)` matches the same. Both are sequence patterns and behave identically.

### Coding Challenges
- Implement a JSON validator that uses `match` to check the structure of a JSON object against a schema.
- Write a simple expression evaluator that matches AST nodes (Number, Add, Subtract, Multiply, Divide) and computes the result.

### Related Topics
- [AST module](#ast-module)
- [C extensions](#c-extensions)
- [Important PEPs](#important-peps)

---

## AST Module

### What It Is
The `ast` module provides facilities for parsing Python source code into an Abstract Syntax Tree (AST), traversing and transforming the tree, and compiling it back to bytecode. The AST is a tree representation of the syntactic structure of Python code.

### Why It Is Important
The AST module is the foundation for:
- **Static analysis**: linters (flake8, pylint), type checkers (mypy), and security scanners (bandit).
- **Code transformation**: automatic refactoring, transpilation (e.g., Cython's pure Python mode), and instrumentation (coverage.py).
- **Metaprogramming**: generating and executing code from ASTs, building DSLs.

### How It Works Internally
1. **Parsing**: `ast.parse(source)` calls CPython's `PyParser_ParseString` to produce a CST (Concrete Syntax Tree).
2. **AST generation**: the CST is converted to an AST by `Python/ast.c`, producing `AST` node objects (e.g., `ast.Module`, `ast.FunctionDef`, `ast.BinOp`).
3. **Tree traversal**: node visitors walk the tree using the visitor pattern (`ast.NodeVisitor`).
4. **Transformation**: `ast.NodeTransformer` allows in-place modification of nodes.
5. **Compilation**: `compile(ast_node, filename, 'exec')` converts the AST to a code object.

### Syntax
```python
import ast

# Parse code
tree = ast.parse('x = 1 + 2', mode='exec')

# Dump the tree
print(ast.dump(tree, indent=2))

# Walk the tree
for node in ast.walk(tree):
    print(type(node).__name__)

# Compile and execute
code = compile(tree, '<string>', 'exec')
exec(code)
```

### Beginner Examples
```python
import ast

# Simple AST inspection
source = """
def greet(name):
    return f"Hello, {name}!"
"""

tree = ast.parse(source)
print(ast.dump(tree, indent=2))

# Counting function definitions
class FunctionCounter(ast.NodeVisitor):
    def __init__(self):
        self.count = 0
    def visit_FunctionDef(self, node):
        self.count += 1
        self.generic_visit(node)

counter = FunctionCounter()
counter.visit(tree)
print(f'Found {counter.count} functions')

# Extracting all variable names
class VariableExtractor(ast.NodeVisitor):
    def __init__(self):
        self.vars = set()
    def visit_Name(self, node):
        self.vars.add(node.id)

extractor = VariableExtractor()
extractor.visit(tree)
print(extractor.vars)
```

### Intermediate Examples
```python
import ast
import astor  # third-party for AST-to-source

# Simple arithmetic simplifier
class SimplifyArithmetic(ast.NodeTransformer):
    def visit_BinOp(self, node):
        self.generic_visit(node)
        if isinstance(node.left, ast.Constant) and isinstance(node.right, ast.Constant):
            if isinstance(node.op, ast.Add):
                return ast.Constant(value=node.left.value + node.right.value)
            if isinstance(node.op, ast.Sub):
                return ast.Constant(value=node.left.value - node.right.value)
            if isinstance(node.op, ast.Mult):
                return ast.Constant(value=node.left.value * node.right.value)
        return node

# Code lint: detect unused imports
class UnusedImportChecker(ast.NodeVisitor):
    def __init__(self):
        self.imports = {}
        self.used_names = set()

    def visit_Import(self, node):
        for alias in node.names:
            self.imports[alias.asname or alias.name] = node.lineno
        self.generic_visit(node)

    def visit_Name(self, node):
        self.used_names.add(node.id)
        self.generic_visit(node)

    def report_unused(self):
        for name, lineno in self.imports.items():
            if name not in self.used_names:
                print(f'Unused import: {name} at line {lineno}')

# Inlining simple functions
class Inliner(ast.NodeTransformer):
    def __init__(self):
        self.functions = {}

    def visit_FunctionDef(self, node):
        if len(node.body) == 1 and isinstance(node.body[0], ast.Return):
            self.functions[node.name] = node.body[0].value
        return node
```

### Advanced Examples
```python
import ast
import inspect
import textwrap

# Decorator that records function bytecode via AST
def instrument(func):
    source = textwrap.dedent(inspect.getsource(func))
    tree = ast.parse(source)

    class Instrumenter(ast.NodeTransformer):
        def visit_FunctionDef(self, node):
            # Add logging at function entry and exit
            log_start = ast.Expr(
                ast.Call(
                    func=ast.Attribute(ast.Name('print', ast.Load()), 'print', ast.Load()),
                    args=[ast.Constant(f'Entering {node.name}')],
                    keywords=[]
                )
            )
            node.body.insert(0, log_start)
            return node

    tree = Instrumenter().visit(tree)
    ast.fix_missing_locations(tree)
    code = compile(tree, func.__code__.co_filename, 'exec')

    # Extract the modified function
    local_ns = {}
    exec(code, func.__globals__, local_ns)
    return local_ns[func.__name__]

# Parser for a simple DSL
class DSLParser:
    """Parse a simple arithmetic DSL into Python AST."""

    def parse(self, source):
        tokens = source.split()
        return self._parse_expression(tokens, 0)[0]

    def _parse_expression(self, tokens, pos):
        if tokens[pos].isdigit():
            return ast.Constant(value=int(tokens[pos])), pos + 1
        if tokens[pos] == '(':
            left, pos = self._parse_expression(tokens, pos + 1)
            op_token = tokens[pos]
            op_map = {'+': ast.Add(), '-': ast.Sub(), '*': ast.Mult(), '/': ast.Div()}
            right, pos = self._parse_expression(tokens, pos + 1)
            assert tokens[pos] == ')'
            return ast.BinOp(left=left, op=op_map[op_token], right=right), pos + 1
        raise ValueError(f'Unexpected token: {tokens[pos]}')

# Mypy-style type annotation extraction
class TypeAnnotationExtractor(ast.NodeVisitor):
    def __init__(self):
        self.type_map = {}

    def visit_FunctionDef(self, node):
        if node.returns:
            self.type_map[f'{node.name}_return'] = ast.dump(node.returns)
        for arg in node.args.args:
            if arg.annotation:
                self.type_map[f'{node.name}.{arg.arg}'] = ast.dump(arg.annotation)
        self.generic_visit(node)
```

### Real-World Use Cases
- **pylint / flake8**: parse source files and apply lint rules via AST visitors.
- **coverage.py**: instrument AST nodes with line-number tracking to report which lines executed.
- **Black (formatter)**: parse source to AST, then output formatted code.
- **Sphinx autodoc**: extract docstrings and signatures from AST without importing the module.
- **Cython pure Python mode**: uses AST to translate typed Python to C.

### Common Mistakes
- Modifying AST nodes without calling `ast.fix_missing_locations()` — missing `lineno` and `col_offset` cause compilation errors.
- Using `ast.literal_eval` on untrusted input — it only evaluates literals but can still consume unbounded resources.
- Forgetting that AST node types change between Python versions — code that works on 3.10 may fail on 3.8.

### Best Practices
- Use `ast.NodeVisitor` for read-only traversal and `ast.NodeTransformer` for modifications.
- Always call `ast.fix_missing_locations()` after transforming the tree.
- Use `ast.literal_eval` for safely evaluating Python literals from untrusted sources.
- Test AST transformations with both valid and invalid inputs to ensure robustness.

### Performance Considerations
- Parsing large source files is O(n) in source length; for massive codebases, incremental parsing is needed.
- Transforming the AST with `NodeTransformer` creates a new tree; for large trees, this can be memory-intensive.
- `ast.dump` with `indent` is helpful for debugging but slow on large trees.

### Interview Questions
- **Q**: What is the difference between `ast.NodeVisitor` and `ast.NodeTransformer`?  
  **A**: `NodeVisitor` walks the tree without modifying it. `NodeTransformer` walks and can replace or delete nodes.
- **Q**: How does `ast.literal_eval` work and why is it safer than `eval`?  
  **A**: It only accepts literal expressions (strings, numbers, tuples, lists, dicts, booleans, None), rejecting function calls and attribute access.

### Coding Challenges
- Write an AST transformer that converts all `while` loops to `for` loops with a range (if possible).
- Implement a simple obfuscator that renames all local variables to random names using AST transformation.

### Related Topics
- [Pattern matching](#pattern-matching-matchcase-310)
- [C extensions](#c-extensions)
- [Important PEPs](#important-peps)

---

## C Extensions

### What It Is
C extensions are shared libraries written in C that can be imported as Python modules. They interact with the Python interpreter through the C API (`Python.h`). C extensions allow Python code to call C functions directly, access system-level APIs, and achieve performance comparable to hand-written C.

### Why It Is Important
C extensions are the ultimate performance optimisation for Python. They bypass the interpreter loop entirely, operate on raw C data types, and can interface with existing C/C++ libraries. Core Python modules (`json`, `io`, `collections`, `socket`, `struct`) are C extensions.

### How It Works Internally
A C extension module implements `PyMODINIT_FUNC PyInit_<name>(void)`. The function:
1. Defines methods in a `PyMethodDef` array.
2. Defines types in `PyTypeObject` structs.
3. Calls `PyModule_Create` to register the module.
4. Python's import mechanism loads the shared library (`.so`/`.pyd`) using `dlopen`/`LoadLibrary` and calls `PyInit_*`.

Reference counting is manual: `Py_INCREF` and `Py_DECREF` manage object lifetimes. The GIL must be released for long-running C code.

### Syntax
```c
#include <Python.h>

static PyObject* my_function(PyObject *self, PyObject *args) {
    int x;
    if (!PyArg_ParseTuple(args, "i", &x))
        return NULL;
    return PyLong_FromLong(x * 2);
}

static PyMethodDef MyMethods[] = {
    {"double_it", my_function, METH_VARARGS, "Double an integer"},
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
```c
// simplest_ext.c
#include <Python.h>

static PyObject* add(PyObject *self, PyObject *args) {
    int a, b;
    if (!PyArg_ParseTuple(args, "ii", &a, &b))
        return NULL;
    return PyLong_FromLong(a + b);
}

static PyObject* hello(PyObject *self, PyObject *args) {
    return PyUnicode_FromString("Hello from C!");
}

static PyMethodDef Methods[] = {
    {"add", add, METH_VARARGS, "Add two integers"},
    {"hello", hello, METH_NOARGS, "Return a greeting"},
    {NULL, NULL, 0, NULL}
};

static struct PyModuleDef extmodule = {
    PyModuleDef_HEAD_INIT, "simple_ext", NULL, -1, Methods
};

PyMODINIT_FUNC PyInit_simple_ext(void) {
    return PyModule_Create(&extmodule);
}

// Build with: python setup.py build_ext --inplace
// setup.py:
// from setuptools import setup, Extension
// setup(ext_modules=[Extension('simple_ext', ['simple_ext.c'])])
```

### Intermediate Examples
```c
// Custom type: 2D Vector
#include <Python.h>
#include <structmember.h>

typedef struct {
    PyObject_HEAD
    double x;
    double y;
} VectorObject;

static void Vector_dealloc(VectorObject *self) {
    Py_TYPE(self)->tp_free((PyObject*)self);
}

static PyObject* Vector_new(PyTypeObject *type, PyObject *args, PyObject *kwds) {
    VectorObject *self = (VectorObject*)type->tp_alloc(type, 0);
    if (self) {
        self->x = 0.0;
        self->y = 0.0;
    }
    return (PyObject*)self;
}

static int Vector_init(VectorObject *self, PyObject *args, PyObject *kwds) {
    static char *kwlist[] = {"x", "y", NULL};
    if (!PyArg_ParseTupleAndKeywords(args, kwds, "|dd", kwlist, &self->x, &self->y))
        return -1;
    return 0;
}

static PyObject* Vector_repr(VectorObject *self) {
    return PyUnicode_FromFormat("Vector(%g, %g)", self->x, self->y);
}

static PyObject* Vector_add(PyObject *a, PyObject *b) {
    if (!PyObject_TypeCheck(a, &VectorType) || !PyObject_TypeCheck(b, &VectorType)) {
        Py_RETURN_NOTIMPLEMENTED;
    }
    VectorObject *va = (VectorObject*)a;
    VectorObject *vb = (VectorObject*)b;
    return PyObject_CallFunction((PyObject*)&VectorType, "dd", va->x + vb->x, va->y + vb->y);
}

static PyNumberMethods Vector_as_number = {
    .nb_add = Vector_add,
};

static PyMemberDef Vector_members[] = {
    {"x", T_DOUBLE, offsetof(VectorObject, x), 0, "x coordinate"},
    {"y", T_DOUBLE, offsetof(VectorObject, y), 0, "y coordinate"},
    {NULL}
};

static PyTypeObject VectorType = {
    PyVarObject_HEAD_INIT(NULL, 0)
    .tp_name = "vector.Vector",
    .tp_basicsize = sizeof(VectorObject),
    .tp_itemsize = 0,
    .tp_dealloc = (destructor)Vector_dealloc,
    .tp_repr = (reprfunc)Vector_repr,
    .tp_as_number = &Vector_as_number,
    .tp_flags = Py_TPFLAGS_DEFAULT,
    .tp_doc = "2D Vector type",
    .tp_members = Vector_members,
    .tp_init = (initproc)Vector_init,
    .tp_new = Vector_new,
};
```

### Advanced Examples
```c
// Buffer protocol for zero-copy
static int Buffer_getbuf(MyObject *self, Py_buffer *view, int flags) {
    return PyBuffer_FillInfo(view, (PyObject*)self,
                             self->buffer, self->size,
                             1,  // readonly
                             flags);
}

static void Buffer_relbuf(MyObject *self, Py_buffer *view) {
    // No-op for simple buffer
}

static PyBufferProcs BufferProcs = {
    .bf_getbuffer = Buffer_getbuf,
    .bf_releasebuffer = Buffer_relbuf,
};

// GIL handling for long-running operations
static PyObject* compute(PyObject *self, PyObject *args) {
    long iterations;
    if (!PyArg_ParseTuple(args, "l", &iterations))
        return NULL;

    // Release GIL for long computation
    Py_BEGIN_ALLOW_THREADS
    for (long i = 0; i < iterations; i++) {
        // CPU-intensive work
        volatile double x = 0.0;
        for (int j = 0; j < 1000; j++) {
            x += j * 0.001;
        }
    }
    Py_END_ALLOW_THREADS

    return PyLong_FromLong(iterations);
}

// Exception handling
static PyObject* risky_divide(PyObject *self, PyObject *args) {
    int a, b;
    if (!PyArg_ParseTuple(args, "ii", &a, &b))
        return NULL;
    if (b == 0) {
        PyErr_SetString(PyExc_ZeroDivisionError, "division by zero in C extension");
        return NULL;
    }
    return PyLong_FromLong(a / b);
}

// Module-level data structures
static PyObject* get_counter(PyObject *self, PyObject *args) {
    static long counter = 0;
    counter++;
    return PyLong_FromLong(counter);
}
```

### Real-World Use Cases
- **NumPy**: core array operations implemented in C for performance.
- **Pillow (PIL)**: image codec wrappers for libjpeg, libpng, etc.
- **psycopg2**: PostgreSQL database driver using libpq C library.
- **lxml**: fast XML/HTML parser wrapping libxml2 and libxslt.

### Common Mistakes
- Forgetting to return `NULL` after setting an exception — returning a valid object with `PyErr_Occurred()` non-NULL causes undefined behaviour.
- Not handling reference counts correctly — leads to memory leaks (missing `Py_DECREF`) or crashes (extra `Py_DECREF`).
- Calling blocking C code without releasing the GIL — stalls all Python threads.
- Using `PyArg_ParseTuple` with wrong format specifiers — silent corruption of the stack.

### Best Practices
- Always use `Py_BEGIN_ALLOW_THREADS` / `Py_END_ALLOW_THREADS` for I/O or CPU-bound work.
- Use `Py_INCREF` and `Py_DECREF` carefully; consider `Py_XINCREF`/`Py_XDECREF` for nullable pointers.
- Use `PyCapsule` to wrap raw C pointers for passing through Python code.
- Prefix all public functions and types with the module name to avoid symbol clashes.
- Use `setuptools.Extension` for building; consider `cibuildwheel` for cross-platform wheels.

### Performance Considerations
- C extensions bypass the interpreter entirely — function calls are ~50x faster than equivalent Python.
- Memory allocation in C (`malloc`/`free`) is faster than Python object allocation.
- Buffer protocol enables zero-copy data sharing between Python and C (used by NumPy).
- The GIL release is not free — re-acquiring the GIL takes ~1 microsecond.

### Interview Questions
- **Q**: How do you release the GIL in a C extension and why is it necessary?  
  **A**: Use `Py_BEGIN_ALLOW_THREADS` / `Py_END_ALLOW_THREADS`. It allows other Python threads to run while the C code blocks on I/O or computation.
- **Q**: What is `PyCapsule` and when would you use it?  
  **A**: `PyCapsule` wraps a `void*` pointer. It is used to safely pass C resources (allocated memory, library handles) through Python APIs without leaking.

### Coding Challenges
- Write a C extension that provides a `sum_of_squares(n)` function that computes the sum of squares from 0 to n-1, and compare its performance to `sum(i*i for i in range(n))`.
- Implement a C extension type `IntArray` that supports `__len__`, `__getitem__`, and `__setitem__` with a contiguous C array backing.

### Related Topics
- [Cython](#cython---static-types-pyx-files-c-extensions-compilation)
- [CPython Internals](#cpython-internals---bytecode-pyobject-interpreter-loop)
- [Important PEPs](#important-peps)

---

## Important PEPs

### What It Is
Python Enhancement Proposals (PEPs) are the design documents that guide Python's evolution. Some PEPs introduce major language features (type hints, async/await, pattern matching), while others establish coding standards (PEP 8), packaging guidelines (PEP 517), or version release schedules (PEP 602).

### Why It Is Important
Understanding key PEPs helps you write idiomatic, modern Python. Each PEP represents hours of community debate and expert design. Knowing why features exist (the motivation sections of PEPs) helps you choose the right tool for the job.

### How It Works Internally
PEPs follow a standardised template:
1. **PEP number and title**
2. **Abstract**: brief summary.
3. **Motivation**: the problem being solved.
4. **Rationale**: why this approach over alternatives.
5. **Specification**: detailed technical description.
6. **Backwards compatibility**: migration strategy.
7. **Reference implementation**: often linked to a CPython PR.

PEPs are accepted by the Python Steering Council after community discussion on python-dev and Discourse.

### Key PEPs Summary

```python
# PEP 8 — Style Guide for Python Code
# - 4 spaces per indentation level
# - 79 characters per line (99 for docstrings/comments)
# - snake_case for functions and variables
# - CamelCase for classes
# - UPPER_CASE for constants
# Imports: standard library, third-party, local (separated by blank lines)

# PEP 20 — The Zen of Python
import this
# Beautiful is better than ugly.
# Explicit is better than implicit.
# Simple is better than complex.
# ...

# PEP 257 — Docstring Conventions
# Triple-quoted strings for docstrings
# One-line docstrings: """Summary."""
# Multi-line: """Summary.\n\nDetailed description."""

# PEP 3107 / 484 — Function Annotations / Type Hints
def greet(name: str) -> str:
    return f"Hello, {name}"

from typing import List, Optional, Dict, Union
def process(items: List[int], config: Optional[Dict[str, int]] = None) -> Union[int, str]:
    ...

# PEP 343 — With Statement
with open('file.txt') as f:
    content = f.read()

# PEP 380 — yield from (delegating to subgenerator)
def flatten(nested):
    for sublist in nested:
        yield from sublist

# PEP 492 — Coroutines with async/await
async def fetch_data(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            return await resp.json()

# PEP 498 — f-strings
name = "Alice"
print(f"Hello, {name}!")  # Python 3.6+

# PEP 526 — Variable Annotations
name: str = "Alice"
scores: List[int] = []

# PEP 557 — Data Classes
from dataclasses import dataclass
@dataclass
class Point:
    x: float
    y: float

# PEP 572 — Assignment Expressions (walrus operator)
if (n := len(data)) > 10:
    print(f"Long list with {n} items")

# PEP 585 — Type Hinting Generics in Standard Collections
# Python 3.9+: use list[int] instead of List[int]
def process(items: list[int]) -> list[str]:
    ...

# PEP 604 — Union Types Syntax
# Python 3.10+: use int | str instead of Union[int, str]
def parse(value: int | str) -> int | None:
    ...

# PEP 634-636 — Structural Pattern Matching
# Python 3.10+
match command.split():
    case ["quit"]: exit()
    case ["hello", name]: print(f"Hello {name}")
    case _: print("Unknown")

# PEP 649 — Deferred Evaluation of Annotations (Python 3.12+)
# Annotations are stored as strings and evaluated lazily
from __future__ import annotations

# PEP 684 — Per-Interpreter GIL (Python 3.12+)
# Sub-interpreters with isolated GILs for true parallelism
```

### Beginner Examples
```python
# PEP 8 example: well-formatted code
def calculate_average(numbers: list[float]) -> float:
    """Calculate the arithmetic mean of a list of numbers.

    Args:
        numbers: A list of numeric values.

    Returns:
        The arithmetic mean.
    """
    if not numbers:
        raise ValueError("Cannot calculate average of empty list")
    return sum(numbers) / len(numbers)

# PEP 257: proper docstrings
class Rectangle:
    """A class representing a rectangle."""

    def __init__(self, width: float, height: float) -> None:
        self.width = width
        self.height = height

    @property
    def area(self) -> float:
        """Calculate and return the area of the rectangle."""
        return self.width * self.height
```

### Intermediate Examples
```python
# PEP 557: Data classes with validation
from dataclasses import dataclass, field
from typing import List

@dataclass(order=True)
class Student:
    name: str
    grade: float
    subjects: List[str] = field(default_factory=list)

    def __post_init__(self):
        if not 0 <= self.grade <= 100:
            raise ValueError("Grade must be between 0 and 100")

# PEP 572: Walrus operator in comprehensions
def process_items(items):
    return [y for x in items if (y := x.strip())]

# PEP 585/604: Modern type hints
from __future__ import annotations

def merge(
    dict1: dict[str, int | str],
    dict2: dict[str, int | str],
) -> dict[str, int | str]:
    return {**dict1, **dict2}

# PEP 343: Custom context manager
from contextlib import contextmanager

@contextmanager
def timed_block(name: str):
    import time
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        print(f'{name}: {elapsed:.3f}s')
```

### Advanced Examples
```python
# PEP 649: Deferred annotation evaluation
from __future__ import annotations
import dataclasses

@dataclasses.dataclass
class Node:
    value: int
    children: list[Node]  # Works without forward reference hack

# PEP 684: Sub-interpreters (Python 3.12+)
import _xxsubinterpreters as interpreters
import threading

def run_in_subinterpreter(code: str) -> None:
    interp = interpreters.create()
    interpreters.run_string(interp, code)

# PEP 484 typing protocol (structural subtyping)
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

def render(obj: Drawable) -> None:
    obj.draw()

# PEP 570 (positional-only parameters)
def greet(name, /, greeting="Hello"):
    """PEP 570: / marks positional-only params."""
    print(f"{greeting}, {name}!")

# PEP 563 (from __future__ import annotations) impact
from __future__ import annotations

class Tree:
    def __init__(self, left: Tree | None = None, right: Tree | None = None):
        self.left = left
        self.right = right
    # No need for string annotations!

# PEP 604 alternatives vs Union
def parse_config_v1() -> dict[str, int | list[int]]:
    return {"port": 8080, "backends": [1, 2, 3]}

# PEP 318 (decorators) evolution
@dataclass
@functools.lru_cache(maxsize=128)
class MemoizedPoint:
    x: int
    y: int
```

### Real-World Use Cases
- **PEP 517/518 (build system)**: every `pip install` from source uses pyproject.toml build hooks.
- **PEP 8**: style guides integrated into linters (flake8, pylint, black).
- **PEP 484**: mypy, pyright, and pyre use type hints for static analysis in large codebases.
- **PEP 557**: Django and Pydantic use data classes or similar patterns for model definitions.
- **PEP 634**: used in HTTP routers (e.g., `pyramid`), JSON parsers, and state machines.

### Common Mistakes
- Ignoring PEP 8 — leads to inconsistent, hard-to-read code in collaborative projects.
- Using old-style type hints (`List[int]`, `Dict[str, int]`) on Python 3.9+ instead of `list[int]`, `dict[str, int]`.
- Using `Union[str, int]` instead of the cleaner `str | int` syntax (Python 3.10+).
- Not using `from __future__ import annotations` when defining recursive type annotations.

### Best Practices
- Follow PEP 8 consistently — use `black` or `ruff` for automatic formatting.
- Leverage modern type hint syntax (PEP 585, 604, 649) on supported Python versions.
- Use data classes (PEP 557) for simple value objects instead of manually defining `__init__`, `__repr__`, `__eq__`.
- Adopt structural pattern matching (PEP 634) for complex branching on data shapes.
- Keep up with PEPs by reading the monthly "Python Developer's Guide" updates.

### Performance Considerations
- PEP 585/604 type hints have zero runtime cost — they are evaluated at definition time and discarded.
- PEP 563 (`from __future__ import annotations`) avoids evaluating annotations at runtime, improving import speed.
- PEP 557 data classes generate efficient `__init__` and `__repr__` in pure Python (no C acceleration).
- PEP 572 (walrus operator) can reduce duplicated expressions, saving computation.

### Interview Questions
- **Q**: What does PEP 8 recommend for line length and why?  
  **A**: 79 characters for code, 72 for docstrings. This ensures readability on small screens and side-by-side diffs.
- **Q**: What is the difference between PEP 563 and PEP 649 for annotation evaluation?  
  **A**: PEP 563 (Python 3.7+) stores annotations as strings; PEP 649 (Python 3.12+) stores them as deferred expressions that are evaluated on demand, fixing issues with forward references and runtime introspection.

### Coding Challenges
- Write a script that checks whether a given Python file follows PEP 8 line length and naming conventions.
- Create a decorator that uses `inspect` and PEP 484 type annotations to validate argument types at runtime.

### Related Topics
- [Pattern matching](#pattern-matching-matchcase-310)
- [AST module](#ast-module)
- [C extensions](#c-extensions)
- [Type hints (conceptual)](#-typing-and-type-hints)
