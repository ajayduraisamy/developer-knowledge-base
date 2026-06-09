# CPython Internals - Bytecode, PyObject, interpreter loop

## Introduction

CPython is the reference implementation of Python, written in C. It compiles Python source code to bytecode and executes it via a virtual machine. Understanding CPython internals helps developers write more efficient code and understand Python's behavior at a deeper level.

## Why It Is Important

Knowledge of CPython internals enables developers to write optimized code, understand performance characteristics of different operations, debug complex issues, and contribute to Python itself. It also helps in understanding memory management, threading, and the GIL.

## Syntax

```python
# Using the dis module to inspect bytecode
import dis

def example():
    x = 10
    y = 20
    return x + y

dis.dis(example)

# Getting bytecode as a code object
code = example.__code__
print(code.co_code)
print(code.co_consts)
print(code.co_names)

# Using sys module for internals
import sys
print(sys.implementation)
print(sys.getsizeof([]))
print(sys.getrefcount([]))
```

## Examples

```python
import dis
import sys
import types


def simple_function(a, b):
    c = a + b
    d = c * 2
    return d


dis.dis(simple_function)


def explore_code_object():
    code = simple_function.__code__
    print(f"Code object: {code}")
    print(f"Arg count: {code.co_argcount}")
    print(f"Local vars: {code.co_varnames}")
    print(f"Constants: {code.co_consts}")
    print(f"Names: {code.co_names}")
    print(f"Bytecode: {code.co_code}")


explore_code_object()
```

## Beginner Examples

```python
import dis
import sys


def understand_bytecode():
    def add(a, b):
        return a + b

    print("Bytecode for add function:")
    dis.dis(add)
    print()


def understand_variable_lookup():
    x = 10

    def inner():
        return x

    print("Bytecode for inner (closure):")
    dis.dis(inner)
    print()


def list_vs_tuple_bytecode():
    def make_list():
        return [1, 2, 3]

    def make_tuple():
        return (1, 2, 3)

    print("Bytecode for list literal:")
    dis.dis(make_list)
    print()
    print("Bytecode for tuple literal:")
    dis.dis(make_tuple)
    print()


understand_bytecode()
understand_variable_lookup()
list_vs_tuple_bytecode()


def sys_internals_basics():
    print(f"Python implementation: {sys.implementation}")
    print(f"Version: {sys.version}")
    print(f"Max size: {sys.maxsize}")
    print(f"Int size: {sys.getsizeof(0)}")
    print(f"Float size: {sys.getsizeof(0.0)}")
    print(f"Empty list size: {sys.getsizeof([])}")
    print(f"Empty dict size: {sys.getsizeof({})}")
    print(f"Empty tuple size: {sys.getsizeof(())}")


sys_internals_basics()
```

## Intermediate Examples

```python
import dis
import sys
import struct


def analyze_bytecode_instructions():
    def complex_function(x):
        result = []
        for i in range(x):
            if i % 2 == 0:
                result.append(i ** 2)
            else:
                result.append(i ** 3)
        return sum(result) / len(result)

    print("Bytecode for complex function:")
    dis.dis(complex_function)
    print()
    print("Bytecode with details:")
    dis.dis(complex_function, depth=2)


analyze_bytecode_instructions()


def understand_string_interning():
    a = "hello"
    b = "hello"
    c = "hello!"
    d = "hello!"
    print(f"a is b: {a is b}")
    print(f"c is d: {c is d}")
    print(f"id(a) == id(b): {id(a) == id(b)}")


understand_string_interning()


def small_integer_caching():
    a = 256
    b = 256
    c = 257
    d = 257
    print(f"256 is 256: {a is b}")
    print(f"257 is 257: {c is d}")


small_integer_caching()


def understand_object_sizes():
    class Empty:
        pass

    class WithSlots:
        __slots__ = ('x', 'y')

    e = Empty()
    w = WithSlots()
    print(f"Empty class instance size: {sys.getsizeof(e)}")
    print(f"WithSlots instance size: {sys.getsizeof(w)}")
    print(f"Empty object __dict__ size: {sys.getsizeof(e.__dict__) if hasattr(e, '__dict__') else 'N/A'}")


understand_object_sizes()
```

## Advanced Examples

```python
import dis
import sys
import types
import opcode
from typing import Any, Dict, List


class BytecodeAnalyzer:
    def __init__(self, func):
        self.func = func
        self.code = func.__code__

    def disassemble(self) -> str:
        return dis.dis(self.func)

    def get_instructions(self) -> List[Dict[str, Any]]:
        instructions = []
        for instr in dis.get_instructions(self.func):
            instructions.append({
                'offset': instr.offset,
                'opname': instr.opname,
                'arg': instr.arg,
                'argval': instr.argval,
            })
        return instructions

    def count_instructions(self) -> Dict[str, int]:
        counts = {}
        for instr in dis.get_instructions(self.func):
            counts[instr.opname] = counts.get(instr.opname, 0) + 1
        return counts

    def complexity_score(self) -> int:
        score = 0
        for instr in dis.get_instructions(self.func):
            opname = instr.opname
            if 'CALL' in opname:
                score += 5
            elif 'LOAD' in opname or 'STORE' in opname:
                score += 1
            elif 'JUMP' in opname:
                score += 3
            elif 'FOR_ITER' in opname:
                score += 4
        return score


def sample_function(n: int) -> int:
    total = 0
    for i in range(n):
        total += i ** 2
    return total


analyzer = BytecodeAnalyzer(sample_function)
print("Instructions:")
for instr in analyzer.get_instructions()[:10]:
    print(f"  {instr['offset']:4d} {instr['opname']:20s} {instr['argval']}")
print(f"\nComplexity score: {analyzer.complexity_score()}")
print(f"Instruction counts: {analyzer.count_instructions()}")


def explore_pyobject():
    print("PyObject structure (C level):")
    print("  typedef struct _object {")
    print("    Py_ssize_t ob_refcnt;")
    print("    PyTypeObject *ob_type;")
    print("  } PyObject;")
    print()
    print("PyVarObject structure:")
    print("  typedef struct {")
    print("    PyObject ob_base;")
    print("    Py_ssize_t ob_size;")
    print("  } PyVarObject;")


explore_pyobject()


def list_internals():
    print("PyListObject internals:")
    print("  PyObject_VAR_HEAD")
    print("  PyObject **ob_item;  // Pointer to items")
    print("  Py_ssize_t allocated; // Allocated slots")
    print()
    print("List resizing strategy:")
    print("  new_allocated = (size_t)newsize + (newsize >> 3) + 3")
    print("  Overallocates by ~12.5% to amortize O(1) appends")


list_internals()


def dict_internals():
    print("PyDictObject internals:")
    print("  PyDictKeysObject *ma_keys;")
    print("  PyObject **ma_values;")
    print()
    print("Dict implemented as hash table with:")
    print("  1. Open addressing with quadratic probing")
    print("  2. Combined table (keys+values) or split table")
    print("  3. 2/3 load factor threshold for resizing")
    print("  4. Compact dict (Python 3.6+) preserves order")


dict_internals()


def free_lists():
    print("CPython free lists (reuse objects):")
    print("  - Small integers (-5 to 256) are cached")
    print("  - Empty tuples are singletons")
    print("  - Free list for list objects")
    print("  - Free list for dict objects")
    print("  - Free list for float objects")
    print("  - Interned strings (auto for identifiers)")


free_lists()
```

## Real-World Use Cases

```python
import dis
import sys
import time
from typing import List


def optimize_with_bytecode_knowledge():
    print("Real-world optimization based on CPython internals:")

    def version_1(data: List[int]) -> int:
        total = 0
        for x in data:
            total += x
        return total

    def version_2(data: List[int]) -> int:
        return sum(data)

    data = list(range(1000000))
    start = time.perf_counter()
    for _ in range(100):
        version_1(data)
    t1 = time.perf_counter() - start

    start = time.perf_counter()
    for _ in range(100):
        version_2(data)
    t2 = time.perf_counter() - start

    print(f"Manual loop: {t1:.4f}s, sum(): {t2:.4f}s")
    print("sum() is faster because it's implemented in C")


optimize_with_bytecode_knowledge()


def method_lookup_optimization():
    print("Method lookup optimization:")
    print("  - Local variables are fastest (LOAD_FAST)")
    print("  - Global variables are slower (LOAD_GLOBAL)")
    print("  - Attribute access is slowest (LOAD_ATTR)")
    print()
    print("Optimization: cache methods in local variables")
    print("  Instead of: list.append(x) in a loop")
    print("  Use: append = list.append; append(x)")


method_lookup_optimization()


def dict_order_guarantee():
    print("Python 3.7+ dict maintains insertion order")
    print("This is due to the compact dict implementation")
    print("which stores values in a separate array")


dict_order_guarantee()
```

## Common Mistakes

```python
import dis
import sys


def mistake_1_mutable_default_args():
    print("Mistake 1: Mutable default arguments")
    print("Default args are evaluated once at function definition")

    def append_to_list(value, lst=[]):
        lst.append(value)
        return lst

    print(f"First call: {append_to_list(1)}")
    print(f"Second call: {append_to_list(2)}")
    print("Fix: Use None and create new list each call")


def mistake_2_chaining_comparisons():
    print("Mistake 2: Not using chained comparisons")

    def check_range_verbose(x):
        return x > 5 and x < 10

    def check_range_chained(x):
        return 5 < x < 10

    print("Chained comparisons are more efficient and readable")


def mistake_3_string_concatenation_in_loops():
    print("Mistake 3: String concatenation in loops")
    print("Strings are immutable, += creates new string each time")
    print("Use ''.join(list) instead")


def mistake_4_slow_attr_access_in_loops():
    print("Mistake 4: Repeated attribute access in loops")
    print("Each .lookup requires dict lookup")
    print("Cache in local variable for speed")


mistake_1_mutable_default_args()
mistake_2_chaining_comparisons()
mistake_3_string_concatenation_in_loops()
mistake_4_slow_attr_access_in_loops()
```

## Best Practices

```python
import dis
import sys


def best_practice_1_use_locals():
    print("Best Practice 1: Use local variables for speed")

    def fast_example(items):
        append = items.append
        for i in range(1000):
            append(i)
        return items

    dis.dis(fast_example)


def best_practice_2_understand_bytecode():
    print("Best Practice 2: Use dis to understand performance")
    print("Compare bytecode of different implementations")


def best_practice_3_leveraging_c_implementations():
    print("Best Practice 3: Use C implementations")
    print("  - Use built-in functions (sum, map, filter)")
    print("  - Use list comprehensions over manual loops")
    print("  - Use itertools for iteration patterns")


def best_practice_4_understand_memory_layout():
    print("Best Practice 4: Understand memory layout")
    print("  - Lists are arrays of PyObject* pointers")
    print("  - Tuples are fixed arrays (faster than lists)")
    print("  - dict uses open addressing hash table")


def best_practice_5_avoid_global_lookups():
    print("Best Practice 5: Avoid global lookups in tight loops")
    print("  - Use local variable bindings")
    print("  - Use default arguments to bind globals")


best_practice_1_use_locals()
best_practice_2_understand_bytecode()
best_practice_3_leveraging_c_implementations()
best_practice_4_understand_memory_layout()
best_practice_5_avoid_global_lookups()
```

## Interview Questions

```python
def interview_q1():
    print("Q: What is the Python interpreter loop?")
    print("A: The main loop in ceval.c that fetches, decodes,")
    print("   and executes bytecode instructions.")


def interview_q2():
    print("Q: How are Python objects represented in C?")
    print("A: Every object is a PyObject with ob_refcnt and")
    print("   ob_type. Variable-size objects use PyVarObject.")


def interview_q3():
    print("Q: What is string interning?")
    print("A: Reusing the same string object for identical")
    print("   strings to save memory. Auto for identifiers.")


def interview_q4():
    print("Q: How does Python's list resizing work?")
    print("A: Overallocates by ~12.5% to amortize O(1) append.")
    print("   Formula: new_alloc = newsize + (newsize >> 3) + 3")


def interview_q5():
    print("Q: What is the small object cache in CPython?")
    print("A: Free lists for small objects (< 256 bytes)")
    print("   to avoid frequent system allocator calls.")


def interview_q6():
    print("Q: What are free lists?")
    print("A: Pre-allocated pools of objects for reuse.")
    print("   Examples: int free list, float free list,")
    print("   list free list, tuple free list.")


def interview_q7():
    print("Q: How does the compact dict implementation work?")
    print("A: Separate keys (hash table) and values (array).")
    print("   Values stored in insertion order.")


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
import dis
import sys
import types


def challenge_1_analyze_bytecode():
    print("Challenge 1: Analyze bytecode of different functions")

    def func_a():
        return sum(i for i in range(100))

    def func_b():
        return sum([i for i in range(100)])

    def func_c():
        total = 0
        for i in range(100):
            total += i
        return total

    print("func_a (generator expression):")
    dis.dis(func_a)
    print("\nfunc_b (list comprehension):")
    dis.dis(func_b)
    print("\nfunc_c (manual loop):")
    dis.dis(func_c)


def challenge_2_understand_interning():
    print("Challenge 2: Predict which strings are interned")

    def test_interning():
        a = "hello"
        b = "hello"
        c = "hello world"
        d = "hello world"
        e = "hello " + "world"
        f = "hello" + " world"
        print(f"a is b: {a is b}")
        print(f"c is d: {c is d}")
        print(f"c is e: {c is e}")
        print(f"c is f: {c is f}")

    test_interning()


def challenge_3_optimize_with_locals():
    print("Challenge 3: Optimize this function using local bindings")

    import math

    def unoptimized(data):
        result = []
        for x in data:
            result.append(math.sin(x) + math.cos(x))
        return result

    sin = math.sin
    cos = math.cos

    def optimized(data):
        result = []
        append = result.append
        for x in data:
            append(sin(x) + cos(x))
        return result

    data = [i * 0.1 for i in range(1000)]
    print(f"Unoptimized result length: {len(unoptimized(data))}")
    print(f"Optimized result length: {len(optimized(data))}")
    print("Disassembly comparison:")
    print("Unoptimized:")
    dis.dis(unoptimized)
    print("Optimized:")
    dis.dis(optimized)


def challenge_4_explore_code_object():
    print("Challenge 4: Explore a function's code object")

    def my_function(x, y=10):
        z = x * y
        return z

    code = my_function.__code__
    print(f"co_argcount: {code.co_argcount}")
    print(f"co_nlocals: {code.co_nlocals}")
    print(f"co_stacksize: {code.co_stacksize}")
    print(f"co_flags: {code.co_flags}")
    print(f"co_code: {code.co_code}")
    print(f"co_consts: {code.co_consts}")
    print(f"co_names: {code.co_names}")
    print(f"co_varnames: {code.co_varnames}")
    print(f"co_filename: {code.co_filename}")
    print(f"co_name: {code.co_name}")
    print(f"co_lnotab: {code.co_lnotab}")
    print(f"co_freevars: {code.co_freevars}")
    print(f"co_cellvars: {code.co_cellvars}")


challenge_1_analyze_bytecode()
challenge_2_understand_interning()
challenge_3_optimize_with_locals()
challenge_4_explore_code_object()
```

## Summary

CPython internals encompass the interpreter loop, bytecode compilation and execution, object representation (PyObject/PyVarObject), memory management, and optimizations like string interning, small object caching, and free lists. Understanding these internals helps write more efficient Python code.

## Related Topics

- Memory Management (92_memory_management.md)
- Profiling (91_profiling.md)
- C Extensions (100_advanced_topics.md)
- Data Structures (97_data_structures.md)
