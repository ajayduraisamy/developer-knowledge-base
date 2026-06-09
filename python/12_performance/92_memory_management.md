# Memory Management - Garbage collection, reference counting, weakref

## Introduction

Python manages memory automatically through a combination of reference counting and a generational garbage collector. The CPython interpreter handles memory allocation, deallocation, and optimization internally. Understanding these mechanisms helps developers write more efficient code and avoid common memory pitfalls.

## Why It Is Important

Memory management affects application performance, scalability, and reliability. Poor memory management leads to memory leaks, high memory usage, and degraded performance. Understanding Python's memory model helps developers optimize memory usage, detect leaks, and build more efficient applications.

## Syntax

```python
# Reference counting basics
import sys
x = []
print(sys.getrefcount(x))  # Returns reference count

# Garbage collection module
import gc
gc.enable()
gc.disable()
gc.collect()  # Manual collection
gc.get_objects()  # Get all tracked objects
gc.get_stats()  # Get GC statistics

# Weak references
import weakref
class MyClass:
    pass
obj = MyClass()
ref = weakref.ref(obj)
print(ref())  # Returns object if alive
del obj
print(ref())  # Returns None

# __slots__ for memory optimization
class WithSlots:
    __slots__ = ('x', 'y')
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Memory profiling
# pip install memory_profiler
# python -m memory_profiler script.py
```

## Examples

```python
import sys
import gc
import weakref
from typing import List, Optional


class Node:
    def __init__(self, value):
        self.value = value
        self.next: Optional['Node'] = None


def demonstrate_reference_counting():
    x = []
    ref_count = sys.getrefcount(x) - 1
    print(f"Reference count for empty list: {ref_count}")
    y = x
    print(f"After assignment y = x: {sys.getrefcount(x) - 1}")
    del y
    print(f"After del y: {sys.getrefcount(x) - 1}")


demonstrate_reference_counting()


def demonstrate_cyclic_reference():
    a = Node(1)
    b = Node(2)
    a.next = b
    b.next = a
    print(f"GC is tracking: {gc.is_tracked(a)}")
    print(f"Collected objects: {gc.collect()}")


demonstrate_cyclic_reference()
```

## Beginner Examples

```python
import sys
import gc


def reference_count_basics():
    a = [1, 2, 3]
    print(f"Refcount after creation: {sys.getrefcount(a) - 1}")
    b = a
    print(f"Refcount after b = a: {sys.getrefcount(a) - 1}")
    c = a
    print(f"Refcount after c = a: {sys.getrefcount(a) - 1}")
    del b
    print(f"Refcount after del b: {sys.getrefcount(a) - 1}")
    del c
    print(f"Refcount after del c: {sys.getrefcount(a) - 1}")


reference_count_basics()


def understanding_gc():
    print(f"Garbage collector enabled: {gc.isenabled()}")
    print(f"GC thresholds: {gc.get_threshold()}")
    gc.set_threshold(700, 10, 10)
    print(f"New thresholds: {gc.get_threshold()}")
    gc.set_threshold(700, 10, 5)


understanding_gc()


def create_and_collect_garbage():
    for _ in range(100):
        x = [i for i in range(1000)]
    print(f"Collected: {gc.collect()} objects")
    print(f"GC stats: {gc.get_stats()}")


create_and_collect_garbage()
```

## Intermediate Examples

```python
import sys
import gc
import weakref
from typing import Any, Dict, List


class CircularReference:
    def __init__(self, name: str):
        self.name = name
        self.other: 'CircularReference' = None

    def __del__(self):
        print(f"Deleting {self.name}")


def create_cyclic_garbage():
    a = CircularReference('A')
    b = CircularReference('B')
    c = CircularReference('C')
    a.other = b
    b.other = c
    c.other = a
    print(f"Objects tracked: {len(gc.get_objects())}")
    del a, b, c
    print(f"Collected: {gc.collect()}")
    print(f"Objects tracked after: {len(gc.get_objects())}")


create_cyclic_garbage()


class CacheWithWeakRef:
    def __init__(self):
        self._cache: Dict[str, weakref.ref] = {}

    def add(self, key: str, value: Any):
        self._cache[key] = weakref.ref(value)

    def get(self, key: str) -> Any:
        ref = self._cache.get(key)
        if ref is not None:
            return ref()
        return None

    def cleanup(self):
        dead_keys = [k for k, v in self._cache.items() if v() is None]
        for k in dead_keys:
            del self._cache[k]


def demonstrate_weakref_cache():
    cache = CacheWithWeakRef()
    obj = {'data': 'important'}
    cache.add('key1', obj)
    print(f"Cache get: {cache.get('key1')}")
    del obj
    print(f"Cache get after delete: {cache.get('key1')}")
    cache.cleanup()
    print(f"Cache size after cleanup: {len(cache._cache)}")


demonstrate_weakref_cache()


def weakref_callbacks():
    class Data:
        def __init__(self, value):
            self.value = value

    def on_delete(ref):
        print(f"Object with ref {ref} was deleted")

    data = Data(42)
    ref = weakref.ref(data, on_delete)
    print(f"Object alive: {ref() is not None}")
    del data
    print(f"Object alive after delete: {ref() is not None}")


weakref_callbacks()
```

## Advanced Examples

```python
import sys
import gc
import weakref
import objgraph
from typing import Any, Dict, List, Optional


class MemoryProfiler:
    def __init__(self):
        self._snapshots: List[Dict[str, Any]] = []

    def take_snapshot(self, label: str = ''):
        gc.collect()
        snapshot = {
            'label': label,
            'tracked_objects': len(gc.get_objects()),
            'object_counts': self._count_types(),
        }
        self._snapshots.append(snapshot)
        return snapshot

    def _count_types(self) -> Dict[str, int]:
        counts = {}
        for obj in gc.get_objects():
            t = type(obj).__name__
            counts[t] = counts.get(t, 0) + 1
        return dict(sorted(counts.items(), key=lambda x: -x[1])[:10])

    def diff(self, label1: str, label2: str) -> Dict[str, int]:
        snap1 = next(s for s in self._snapshots if s['label'] == label1)
        snap2 = next(s for s in self._snapshots if s['label'] == label2)
        diff = {}
        all_keys = set(snap1['object_counts']) | set(snap2['object_counts'])
        for k in all_keys:
            v1 = snap1['object_counts'].get(k, 0)
            v2 = snap2['object_counts'].get(k, 0)
            if v1 != v2:
                diff[k] = v2 - v1
        return diff


profiler = MemoryProfiler()
profiler.take_snapshot('start')


class LargeObject:
    __slots__ = ('id', 'name', 'data')

    def __init__(self, id: int, name: str):
        self.id = id
        self.name = name
        self.data = [0] * 1000


class RegularObject:
    def __init__(self, id: int, name: str):
        self.id = id
        self.name = name
        self.data = [0] * 1000


def compare_slots_vs_dict():
    large_objects = [LargeObject(i, f'obj_{i}') for i in range(1000)]
    regular_objects = [RegularObject(i, f'obj_{i}') for i in range(1000)]
    size_large = sys.getsizeof(large_objects[0])
    size_regular = sys.getsizeof(regular_objects[0])
    print(f"LargeObject (__slots__) size: {size_large} bytes")
    print(f"RegularObject (__dict__) size: {size_regular} bytes")
    print(f"Memory saved per object: {size_regular - size_large} bytes")
    print(f"Total saved for 1000 objects: {(size_regular - size_large) * 1000} bytes")


compare_slots_vs_dict()


def detect_memory_leak():
    leaked = []
    class Leaky:
        def __init__(self):
            self.data = [i for i in range(10000)]

    for i in range(100):
        leaked.append(Leaky())

    profiler.take_snapshot('after_leak')
    print(f"Tracked objects at start: {profiler._snapshots[0]['tracked_objects']}")
    print(f"Tracked objects after leak: {profiler._snapshots[1]['tracked_objects']}")
    diff = profiler.diff('start', 'after_leak')
    print(f"Diff counts: {diff}")


detect_memory_leak()


def objgraph_usage():
    print("objgraph usage examples:")
    print("  import objgraph")
    print("  objgraph.show_refs([obj], filename='refs.png')")
    print("  objgraph.show_backrefs([obj], filename='backrefs.png')")
    print("  objgraph.count('list')  # Count list objects")
    print("  objgraph.typestats()     # Show type statistics")
    print("  objgraph.show_growth()   # Show growing objects")


objgraph_usage()
```

## Real-World Use Cases

```python
import gc
import weakref
import sys
from typing import Any, Dict, List, Optional, Set


class ConnectionPool:
    def __init__(self, max_size: int = 10):
        self._pool: List[weakref.ref] = []
        self._max_size = max_size
        self._active: Set[int] = set()

    def acquire(self) -> Optional[Any]:
        self._cleanup()
        if self._pool:
            ref = self._pool.pop()
            conn = ref()
            if conn is not None:
                self._active.add(id(conn))
                return conn
        if len(self._active) < self._max_size:
            conn = self._create_connection()
            self._active.add(id(conn))
            return conn
        return None

    def release(self, conn: Any) -> None:
        conn_id = id(conn)
        if conn_id in self._active:
            self._active.remove(conn_id)
            self._pool.append(weakref.ref(conn))

    def _cleanup(self):
        self._pool = [ref for ref in self._pool if ref() is not None]

    def _create_connection(self) -> dict:
        return {'buffer': [], 'connected': True}


pool = ConnectionPool(max_size=3)
conn1 = pool.acquire()
conn2 = pool.acquire()
pool.release(conn1)
conn3 = pool.acquire()
print(f"Active connections: {pool._active}")


def web_app_memory_management():
    print("Web app memory best practices:")
    print("1. Use connection pooling with weakref")
    print("2. Clear request-scoped caches after each request")
    print("3. Set max size for caches (LRU)")
    print("4. Monitor memory usage with gc.get_objects()")
    print("5. Use __slots__ for dataclass-like objects")
    print("6. Avoid circular references in request handlers")


web_app_memory_management()
```

## Common Mistakes

```python
import gc
import sys
from typing import List


def mistake_1_ignoring_cycles():
    print("Mistake 1: Creating circular references without cleanup")

    class Parent:
        def __init__(self):
            self.children = []

        def __del__(self):
            print("Parent deleted")

    class Child:
        def __init__(self, parent):
            self.parent = parent

        def __del__(self):
            print("Child deleted")

    p = Parent()
    c = Child(p)
    p.children.append(c)
    print(f"Objects before delete: {len(gc.get_objects())}")
    del p, c
    print(f"Objects after delete: {len(gc.get_objects())}")
    gc.collect()
    print("After gc.collect(), objects should be freed")


def mistake_2_holding_references_in_caches():
    print("Mistake 2: Caches holding references indefinitely")
    print("Solution: Use weakref or set max cache size")


def mistake_3_not_using_slots():
    print("Mistake 3: Not using __slots__ for many small objects")
    print("Each object has __dict__ overhead (~40+ bytes)")


def mistake_4_modifying_list_while_iterating():
    print("Mistake 4: Modifying list while iterating")
    items = [1, 2, 3, 4, 5]
    for item in items[:]:
        if item % 2 == 0:
            items.remove(item)
    print(f"Items after safe removal: {items}")


def mistake_5_global_variables():
    print("Mistake 5: Global variables holding large data")
    print("Globals persist for the entire program lifetime")
    print("Use function-local or context-managed resources")


mistake_1_ignoring_cycles()
mistake_2_holding_references_in_caches()
mistake_3_not_using_slots()
mistake_4_modifying_list_while_iterating()
mistake_5_global_variables()
```

## Best Practices

```python
import gc
import sys
import weakref
from typing import Any, Dict


def best_practice_1_use_slots():
    print("Best Practice 1: Use __slots__ for data-heavy classes")

    class Point:
        __slots__ = ('x', 'y')
        def __init__(self, x, y):
            self.x = x
            self.y = y

    p = Point(1, 2)
    print(f"Point size: {sys.getsizeof(p)} bytes")


def best_practice_2_weakref_for_caches():
    print("Best Practice 2: Use weakref for caches and caches")
    cache: Dict[str, weakref.ref] = {}
    obj = {'data': 'test'}
    cache['key'] = weakref.ref(obj)
    print(f"Cache hit: {cache['key']() is not None}")
    del obj
    print(f"Cache miss: {cache['key']() is None}")


def best_practice_3_manual_gc_collection():
    print("Best Practice 3: Manual GC collection at idle times")
    gc.set_debug(gc.DEBUG_LEAK)
    gc.collect()
    gc.set_debug(0)


def best_practice_4_use_generators():
    print("Best Practice 4: Use generators for large data")
    print("Generators yield one item at a time")
    print("vs. creating full lists in memory")


def best_practice_5_monitor_memory():
    print("Best Practice 5: Monitor memory regularly")
    import tracemalloc
    tracemalloc.start()
    snapshot = tracemalloc.take_snapshot()
    top_stats = snapshot.statistics('lineno')
    for stat in top_stats[:5]:
        print(stat)


def best_practice_6_pool_objects():
    print("Best Practice 6: Object pooling for expensive objects")
    print("Reuse objects instead of creating new ones")


best_practice_1_use_slots()
best_practice_2_weakref_for_caches()
best_practice_3_manual_gc_collection()
best_practice_4_use_generators()
best_practice_5_monitor_memory()
best_practice_6_pool_objects()
```

## Interview Questions

```python
def interview_q1():
    print("Q: How does Python manage memory?")
    print("A: CPython uses reference counting + generational GC.")
    print("   Objects are freed when refcount reaches 0.")


def interview_q2():
    print("Q: What is the Garbage Collection algorithm?")
    print("A: Generational GC: young, middle, old generations.")
    print("   Objects that survive collection move to older gen.")


def interview_q3():
    print("Q: What are cyclic references and how are they handled?")
    print("A: When objects reference each other, refcount never")
    print("   reaches 0. The GC detects and collects these.")


def interview_q4():
    print("Q: What is weakref and when to use it?")
    print("A: weakref holds a reference that doesn't increase")
    print("   refcount. Used for caches and parent references.")


def interview_q5():
    print("Q: How does __slots__ save memory?")
    print("A: It eliminates __dict__ per object, saving ~40+ bytes")
    print("   and speeds up attribute access.")


def interview_q6():
    print("Q: How do you detect memory leaks in Python?")
    print("A: Use gc.get_objects(), tracemalloc, memory_profiler,")
    print("   objgraph for visualization of reference graphs.")


def interview_q7():
    print("Q: What is tracemalloc?")
    print("A: A module to trace memory allocations. It tracks")
    print("   which line of code allocated which memory.")


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
import gc
import weakref
import sys


def challenge_1_detect_cycle():
    print("Challenge 1: Find and fix the cycle in this code")

    class Node:
        def __init__(self, name):
            self.name = name
            self.neighbor = None

    a = Node('A')
    b = Node('B')
    a.neighbor = b
    b.neighbor = a
    print(f"GC collected: {gc.collect()}")
    print("Fix: Use weakref for one direction")


def challenge_2_optimize_memory():
    print("Challenge 2: Optimize this class with __slots__")

    class Point:
        __slots__ = ('x', 'y', 'z')
        def __init__(self, x, y, z):
            self.x = x
            self.y = y
            self.z = z

    points = [Point(i, i * 2, i * 3) for i in range(1000)]
    print(f"Point size: {sys.getsizeof(points[0])} bytes")


def challenge_3_implement_weak_cache():
    print("Challenge 3: Implement a weak reference cache")

    class WeakCache:
        def __init__(self):
            self._data = {}

        def set(self, key, value):
            self._data[key] = weakref.ref(value)

        def get(self, key):
            ref = self._data.get(key)
            return ref() if ref else None

        def cleanup(self):
            dead = [k for k, v in self._data.items() if v() is None]
            for k in dead:
                del self._data[k]

    cache = WeakCache()
    obj = [1, 2, 3]
    cache.set('list1', obj)
    print(f"Before delete: {cache.get('list1')}")
    del obj
    print(f"After delete: {cache.get('list1')}")
    cache.cleanup()
    print(f"Cache size: {len(cache._data)}")


def challenge_4_profile_memory_usage():
    print("Challenge 4: Profile memory usage of different data structures")
    print("Compare list vs tuple vs array.array for storing 10000 integers")
    print("Use memory_profiler or sys.getsizeof()")


challenge_1_detect_cycle()
challenge_2_optimize_memory()
challenge_3_implement_weak_cache()
challenge_4_profile_memory_usage()
```

## Summary

Python's memory management combines reference counting with generational garbage collection. Key concepts include understanding cyclic references, using weakref appropriately, optimizing with __slots__, detecting memory leaks via gc module and tracemalloc, and profiling memory with tools like memory_profiler and objgraph.

## Related Topics

- Profiling (91_profiling.md)
- CPython Internals (93_cpython_internals.md)
- Caching (96_caching.md)
- Data Structures (97_data_structures.md)
