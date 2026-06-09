# Memory Management - Garbage collection, reference counting, weakref

## Introduction
Python manages memory automatically through a combination of reference counting and a cyclic garbage collector. Every object carries an integer refcount; when it reaches zero, the object is deallocated immediately. The garbage collector (the `gc` module) handles the one situation reference counting cannot: cycles of objects that reference each other but are otherwise unreachable. The `weakref` module allows references that do not increment the refcount, enabling caches and observers without preventing object deallocation. Understanding these three mechanisms is essential for writing memory-efficient, leak-free Python applications.

## Garbage Collection

### What It Is
Python's garbage collector (GC) detects and collects *unreachable* object cycles — groups of objects whose reference counts are positive only because they reference each other in a cycle, with no external references. The GC lives in the `gc` module and runs as part of the interpreter's memory allocation routine (`PyMem_Malloc`). It is generational (three generations) and runs automatically when the number of allocations minus deallocations exceeds a per-generation threshold.

### Why It Is Important
Without a cycle collector, Python would leak memory whenever container objects (lists, dicts, instances of user-defined classes) formed reference cycles and became unreachable. The GC reclaims this memory, preventing unbounded growth in long-running processes such as web servers and background workers.

### How It Works Internally
The GC is a stop-the-world mark-and-sweep collector. When triggered:
1. **Tracking**: The GC maintains a linked list of all tracked container objects (objects that *can* participate in cycles, such as `list`, `dict`, `set`, and instances of user classes).
2. **Generations**: Objects start in generation 0. If they survive a collection, they move to generation 1, then generation 2. Thresholds are `(700, 10, 10)` by default. Collections become progressively less frequent in higher generations.
3. **Mark phase**: Starting from a set of root objects (globals, stack frames), the GC follows references, marking reachable objects. The `gc.garbage` list holds objects with `__del__` methods that cannot be collected safely.
4. **Sweep phase**: Unmarked objects are deallocated by calling `tp_dealloc`.

```python
import gc
gc.get_threshold()   # (700, 10, 10) - gen0, gen1, gen2
gc.get_count()       # current counters
```

### Syntax
```python
import gc

gc.enable()           # enable automatic GC (default on)
gc.disable()          # disable automatic GC
gc.collect()          # perform a full collection
gc.collect(1)         # collect only generation <= 1
gc.set_debug(gc.DEBUG_STATS)  # print collection stats
gc.get_objects()      # list of all tracked objects
gc.is_tracked(obj)    # check if obj is tracked by GC
```

### Beginner Examples
```python
import gc

class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

# Create a cycle
a = Node(1)
b = Node(2)
a.next = b
b.next = a

# Delete external references
del a, b

# Force collection and check
collected = gc.collect()
print(f'Collected {collected} objects')
```

### Intermediate Examples
```python
import gc
import sys

class Resource:
    def __init__(self, name):
        self.name = name
        self.other = None

    def __del__(self):
        print(f'Deleting {self.name}')

gc.set_debug(gc.DEBUG_STATS)
gc.set_debug(gc.DEBUG_LEAK)

# Manually triggering collection
for i in range(1000):
    r = Resource(f'R{i}')
    r.other = Resource(f'R{i}_other')
    r.other.other = r     # cycle

gc.collect()
print(f'Uncollectable garbage: {len(gc.garbage)}')
```

### Advanced Examples
```python
import gc
import weakref
import objgraph

class HeavyObject:
    def __init__(self, data):
        self.data = data
        self.ref = None

# Find root cause of memory leak
def find_leak():
    gc.collect()
    objgraph.show_most_common_types(limit=20)

    # Find all reference chains to a specific type
    chains = objgraph.find_backref_chain(
        some_instance,
        objgraph.is_proper_module
    )
    objgraph.show_chain(chains)

# Custom GC strategy for real-time systems
class RealTimeGC:
    def __init__(self, interval_ms=50):
        self.interval = interval_ms
        gc.disable()

    def tick(self):
        if gc.get_count()[0] >= 1400:  # double default threshold
            gc.collect(0)
        if gc.get_count()[1] >= 20:
            gc.collect(1)

# Debugging with gc.callbacks
def on_gc_stage(phase, info):
    if phase == 'start':
        print(f'GC {info["generation"]} starting, collections: {gc.get_count()}')

gc.callbacks.append(on_gc_stage)
```

### Real-World Use Cases
- **Web server memory leak detection**: run `gc.collect()` after each request and compare `gc.get_count()` deltas to identify endpoints that leak.
- **Game loop memory management**: disable automatic GC in a game loop and call `gc.collect(0)` manually every N frames to avoid unpredictable pauses.
- **Data science pipelines**: call `gc.collect()` explicitly after large matrix operations to return memory to the OS sooner.

### Common Mistakes
- Forgetting that `__del__` methods create cycles that the GC cannot collect — they end up in `gc.garbage`.
- Disabling GC globally without understanding that the application creates cycles — leads to unbounded memory growth.
- Assuming `gc.collect()` returns memory to the OS — Python's allocator retains free arenas for reuse.

### Best Practices
- Leave the GC enabled unless you have a specific real-time constraint.
- Profile object allocations with `gc.get_objects()` and `objgraph` when investigating leaks.
- Use `gc.DEBUG_SAVEALL` during development to print every collected object.

### Performance Considerations
- Full GC (generation 2) can take milliseconds and cause latency spikes; for latency-sensitive applications, manually control collection frequency.
- Disabling the GC improves throughput by 5–15% on allocation-heavy workloads but risks memory leaks if cycles exist.
- Each tracked object carries a hidden `PyGC_Head` (approximately 32 bytes) overhead.

### Interview Questions
- **Q**: How does Python detect and collect reference cycles?  
  **A**: The GC uses a generational mark-and-sweep algorithm starting from root objects, marking reachable containers, and sweeping unmarked ones.
- **Q**: Why does `gc.garbage` exist and when does it fill up?  
  **A**: Objects with `__del__` methods that form cycles are moved to `gc.garbage` because the GC cannot determine a safe deallocation order.

### Coding Challenges
- Implement a cycle detector that walks object references from a given root and returns the set of all reachable objects.
- Write a context manager that temporarily disables GC, runs a block, then forces collection and logs how many objects were collected.

### Related Topics
- [Reference counting](#reference-counting)
- [weakref module](#weakref-module)
- [memory_profiler](#memory_profiler)

---

## Reference Counting

### What It Is
Reference counting is Python's primary memory management mechanism. Every Python object (`PyObject`) has an `ob_refcnt` field — an integer that tracks how many references point to that object. When `ob_refcnt` reaches zero, the object's memory is deallocated immediately. This is deterministic, predictable, and requires no separate collection pass.

### Why It Is Important
Reference counting guarantees that most objects are freed the moment they become unreachable, without waiting for a GC cycle. This makes Python suitable for scripting and automation where predictable teardown (file handles, locks, sockets) is critical.

### How It Works Internally
The `PyObject` struct starts with `PyObject_HEAD`:
```c
typedef struct _object {
    Py_ssize_t ob_refcnt;
    PyTypeObject *ob_type;
} PyObject;
```
Operations that modify the refcount:
- `Py_INCREF(op)` — increments `ob_refcnt` (when a new reference is taken: assignment, passing to a function, appending to a list).
- `Py_DECREF(op)` — decrements `ob_refcnt` and calls `tp_dealloc` if it reaches zero.
- `Py_XINCREF` / `Py_XDECREF` — same but accept `NULL` safely.

When `tp_dealloc` is called, the object calls `PyMem_FREE` to release its memory. For containers, `tp_dealloc` first iterates child objects and calls `Py_DECREF` on each, which can trigger cascading deallocations.

### Syntax
```python
import sys

# Get reference count
obj = []
sys.getrefcount(obj)   # returns 2+ (1 for local var, 1 for argument)

# The refcount increases with each new reference
a = obj
b = a
c = [obj]
print(sys.getrefcount(obj))  # typically 5+

# Deletion decreases refcount
del a
print(sys.getrefcount(obj))  # typically 4+

# Container membership
d = {}
d['key'] = obj
print(sys.getrefcount(obj))
```

### Beginner Examples
```python
import sys

def show_refcounts():
    x = "hello"
    print(f'Initial refcount: {sys.getrefcount(x)}')

    y = x
    print(f'After y = x: {sys.getrefcount(x)}')

    lst = [x]
    print(f'After list: {sys.getrefcount(x)}')

    d = {1: x}
    print(f'After dict: {sys.getrefcount(x)}')

    del y, lst, d
    print(f'After cleanup: {sys.getrefcount(x)}')

show_refcounts()
```

### Intermediate Examples
```python
import sys
import ctypes

class MutableString:
    def __init__(self, value):
        self.value = value

def refcount_address(obj):
    """Get refcount via ctypes for demonstration."""
    return ctypes.c_long.from_address(id(obj)).value

a = MutableString("test")
print(f'Refcount via sys: {sys.getrefcount(a)}')
print(f'Refcount via ctypes: {refcount_address(a)}')

b = a
print(f'After b = a: {refcount_address(a)}')

# Interned strings
s1 = "hello_world_intern"
s2 = "hello_world_intern"
print(f'Interned string refcount: {sys.getrefcount(s1)}')

# Small integers are cached
x = 256
y = 256
print(f'Small int (same object): {x is y}')
print(f'Refcount of 256: {sys.getrefcount(x)}')
```

### Advanced Examples
```python
import sys
import ctypes
import gc

# Observing deallocation with weakref callbacks
import weakref

class Watched:
    def __init__(self, name):
        self.name = name

    def __repr__(self):
        return f'Watched({self.name})'

def on_delete(ref):
    print(f'{ref} was deleted!')

obj = Watched('test')
ref = weakref.ref(obj, on_delete)

print(f'Refcount before delete: {sys.getrefcount(obj)}')
del obj
print('After del obj')

# Understanding temporary references
def get_count():
    return sys.getrefcount(some_global)

some_global = []
print(f'Refcount inside call (includes temp): {get_count()}')

# Container refcount mechanics
parent = []
child = [parent]        # parent refcount += 1 (from child list)
parent.append(child)    # child refcount += 1 (from parent list)

print(f'Parent refcount: {sys.getrefcount(parent)}')
print(f'Child refcount: {sys.getrefcount(child)}')
```

### Real-World Use Cases
- **Resource cleanup**: file objects, socket connections, and lock objects are freed immediately when their last reference is dropped.
- **C extension debugging**: if you write C extensions, mismanaging `Py_INCREF`/`Py_DECREF` causes segfaults or memory leaks; `sys.getrefcount` helps debug.
- **High-frequency trading**: deterministic deallocation ensures that memory pressure never accumulates between ticks.

### Common Mistakes
- Using `sys.getrefcount(obj)` and forgetting that passing `obj` as an argument increments the refcount by 1.
- Assuming `del x` always calls `tp_dealloc` — it only decrements the refcount; if other references exist, the object lives on.
- Creating cycles of container objects without understanding that the GC is required to free them.

### Best Practices
- For long-lived objects, use `weakref` for caches and back-references to avoid artificially inflating refcounts.
- Use context managers (`with`) for resources instead of relying on `__del__` — `__del__` may be delayed by the GC.
- Never manually call `ctypes` to modify refcounts — that violates memory safety.

### Performance Considerations
- Refcount updates are the most frequent operation in CPython — every assignment, function call, and attribute access touches `ob_refcnt`.
- The GIL serialises refcount updates, so multithreaded programs do not need atomic refcount operations.
- PyPy and other non-CPython implementations replace refcounting with a tracing GC to avoid this overhead.

### Interview Questions
- **Q**: Why doesn't CPython use pure garbage collection instead of reference counting?  
  **A**: Reference counting provides deterministic deallocation, which is important for resource cleanup. Also, it fits CPython's C heritage where explicit memory management is familiar.
- **Q**: What happens when two container objects reference each other and all external references are deleted?  
  **A**: Their refcounts never reach zero (each points to the other), so the GC must collect them via mark-and-sweep.

### Coding Challenges
- Implement a simple reference-counted wrapper in Python using `ctypes` that prints a message when the object is destroyed.
- Write a script that detects reference cycles in arbitrary object graphs by simulating a naive reference count subtractor.

### Related Topics
- [Garbage collection](#garbage-collection)
- [weakref module](#weakref-module)
- [CPython Internals](#cpython-internals---bytecode-pyobject-interpreter-loop)

---

## weakref module

### What It Is
The `weakref` module creates references to objects that do *not* increment the reference count. When the last strong reference to an object is deleted, all weak references become `None` (or invoke a callback). The module provides `ref`, `proxy`, `WeakValueDictionary`, `WeakKeyDictionary`, `WeakSet`, and `finalize`.

### Why It Is Important
Weak references are essential for caches, event listeners, and graph structures with back-references. Without weakrefs, these patterns would keep objects alive indefinitely, causing memory leaks. Weak references also enable `weakref.finalize` for reliable cleanup callbacks.

### How It Works Internally
A `weakref.ref` object stores a raw `PyObject*` pointer (*not* incremented) to the target object. The `PyObject` struct includes a `PyObject* ob_weaklist` member — a linked list of all live weak references to that object. When `tp_dealloc` is called (refcount reaches zero), CPython iterates `ob_weaklist`, sets each weak reference's pointer to `NULL`, and invokes any registered callbacks. The weak reference object itself is deallocated separately when its own refcount reaches zero.

### Syntax
```python
import weakref

# Basic weak reference
ref = weakref.ref(target)
obj = ref()            # returns target or None if dead

# Weak proxy (acts as transparent proxy)
proxy = weakref.proxy(target)
proxy.method()         # raises ReferenceError if dead

# Weak value dictionary (keys are strongly referenced)
d = weakref.WeakValueDictionary()
d[key] = value         # value may be GC'd if no strong refs

# Weak key dictionary (keys are weakly referenced)
d = weakref.WeakKeyDictionary()
d[obj] = value         # entry removed when obj is GC'd

# Weak set
s = weakref.WeakSet()
s.add(obj)             # obj removed when GC'd

# Finalize (callback on object death)
weakref.finalize(obj, callback, *args, **kwargs)
```

### Beginner Examples
```python
import weakref

class ExpensiveData:
    def __init__(self, name):
        self.name = name
        self.data = [0] * 10_000_000

def on_death(ref):
    print(f'Object {ref} has been garbage collected')

data = ExpensiveData('test')
ref = weakref.ref(data, on_death)
proxy = weakref.proxy(data)

print(f'Via ref: {ref().name}')
print(f'Via proxy: {proxy.name}')

del data
print(f'After delete ref: {ref()}')     # None
```

### Intermediate Examples
```python
import weakref

class Cache:
    def __init__(self):
        self._cache = weakref.WeakValueDictionary()

    def get(self, key):
        return self._cache.get(key)

    def set(self, key, value):
        self._cache[key] = value

class DataLoader:
    def __init__(self, cache: Cache):
        self._cache = cache

    def load(self, key):
        cached = self._cache.get(key)
        if cached is not None:
            return cached
        data = self._fetch_from_db(key)
        self._cache.set(key, data)
        return data

    def _fetch_from_db(self, key):
        return {'key': key, 'value': 'expensive'}

# Using finalize for resource cleanup
import tempfile
import os

class TempResource:
    def __init__(self):
        self.tmpfile = tempfile.NamedTemporaryFile(delete=False)
        weakref.finalize(self, self._cleanup, self.tmpfile.name)

    @staticmethod
    def _cleanup(path):
        if os.path.exists(path):
            os.unlink(path)
            print(f'Cleaned up {path}')
```

### Advanced Examples
```python
import weakref
import gc

# Observer pattern without preventing deallocation
class EventEmitter:
    def __init__(self):
        self._listeners = weakref.WeakSet()

    def add_listener(self, listener):
        self._listeners.add(listener)

    def emit(self, event):
        dead = []
        for listener in self._listeners:
            try:
                listener(event)
            except ReferenceError:
                dead.append(listener)
        for listener in dead:
            self._listeners.discard(listener)

class MyListener:
    def __call__(self, event):
        print(f'Got event: {event}')

emitter = EventEmitter()
listener = MyListener()
emitter.add_listener(listener)
emitter.emit('test')

del listener
gc.collect()
emitter.emit('after_delete')  # no output, listener auto-removed

# Ref graph introspection
class Node:
    def __init__(self):
        self.parent = None
        self.children = []

    def add_child(self, child):
        child.parent = weakref.ref(self)  # avoid cycle
        self.children.append(child)

root = Node()
child = Node()
root.add_child(child)

# WeakKeyDictionary for metadata
metadata = weakref.WeakKeyDictionary()

class Widget:
    def __init__(self, name):
        self.name = name

w = Widget('button')
metadata[w] = {'x': 10, 'y': 20}
print(metadata[w])
```

### Real-World Use Cases
- **Object-relational mappers (SQLAlchemy)**: use weak references for back-references between related model instances to avoid cycles.
- **GUI frameworks (tkinter, PyQt)**: store widget callbacks as weak references so that disconnecting a widget does not leak memory.
- **LRU caches**: combine `WeakValueDictionary` with a strong-reference LRU list to create a cache that evicts when memory is needed.

### Common Mistakes
- Calling `ref()` and not checking for `None` — causes `AttributeError` if the target was collected.
- Assuming `WeakValueDictionary` entries are evicted immediately on `del value` — they are removed only during GC or dictionary mutation.
- Using `weakref.proxy` and catching `ReferenceError` instead of using `ref()` with an explicit `None` check (proxies have overhead).

### Best Practices
- Use `WeakValueDictionary` for caches where the cache should not keep values alive.
- Use `WeakKeyDictionary` for metadata attached to objects you do not own.
- Use `weakref.finalize` instead of `__del__` for cleanup — it is more predictable and does not interact with GC cycles.
- Always check `ref() is not None` before dereferencing.

### Performance Considerations
- Creating a weak reference incurs a small allocation (the `weakref.ref` object) plus an insertion into the target's weak list.
- Dereferencing a live weak reference is only slightly slower than a strong reference (one pointer indirection).
- `WeakValueDictionary` scans its internal dict for dead refs on mutation — frequent mutations on large dictionaries are O(n).

### Interview Questions
- **Q**: How does CPython implement weak references at the C level?  
  **A**: Each `PyObject` has an `ob_weaklist` field; `PyWeakref_NewRef` adds to this list. On deallocation, CPython walks the list, nulls pointers, and fires callbacks.
- **Q**: What is the difference between `weakref.ref` and `weakref.proxy`?  
  **A**: `ref` returns the object or `None`; `proxy` provides transparent attribute access but raises `ReferenceError` if the object is dead.

### Coding Challenges
- Implement a simple `WeakSet` using `weakref.ref` and a regular `set`.
- Design a thread-safe `WeakCache` that supports TTL and automatic eviction when memory pressure exceeds a threshold.

### Related Topics
- [Reference counting](#reference-counting)
- [Garbage collection](#garbage-collection)
- [Caching](#caching---lru-cache-redis-memoization-cache-invalidation-strategies)
