# Garbage Collection - Mark-and-sweep, reference counting, V8 GC, weak references

## Introduction

Garbage collection (GC) is automatic memory management that identifies and frees memory that is no longer reachable by the program. JavaScript engines use sophisticated GC algorithms to maintain performance while preventing memory leaks.

## Mark-and-Sweep Algorithm

### What It Is

Mark-and-sweep is the fundamental GC algorithm. It starts from root objects (global object, local variables, stack frames) and traverses all reachable references, marking them as alive. Then it sweeps (frees) all unmarked objects.

### How It Works Internally

1. **Root set**: Identify root references (global object, current stack).
2. **Mark phase**: Trace from roots, marking every reachable object.
3. **Sweep phase**: Iterate all objects; any unmarked object is garbage.
4. **Compact (optional)**: Move surviving objects together to reduce fragmentation.

Modern V8 uses a tri-color marking system:
- **White**: Not visited (candidate for collection)
- **Gray**: Discovered but not yet processed
- **Black**: Processed and all references traced

```javascript
// Conceptual illustration only - GC is automatic
let root = { name: 'root' };
let child1 = { parent: root, data: 'child1' };
let child2 = { parent: root, data: 'child2' };
root.children = [child1, child2];

// All three objects are reachable from root

child1 = null; // child1 object still reachable via root.children[0]
root.children = []; // Now child1 and child2 are unreachable
// Next GC cycle: mark-and-sweep frees both
```

### Coding Challenges

**Challenge 1: Visualize mark-and-sweep**
```javascript
class MarkSweepSimulator {
  constructor() {
    this.objects = new Map();
    this.roots = new Set();
  }
  addObject(id, refs = []) {
    this.objects.set(id, { refs, marked: false });
  }
  addRoot(id) { this.roots.add(id); }
  mark() {
    for (const root of this.roots) this.traverse(root);
  }
  traverse(id) {
    const obj = this.objects.get(id);
    if (!obj || obj.marked) return;
    obj.marked = true;
    for (const ref of obj.refs) this.traverse(ref);
  }
  sweep() {
    for (const [id, obj] of this.objects) {
      if (!obj.marked) this.objects.delete(id);
      else obj.marked = false;
    }
  }
}
```

**Challenge 2: Detect circular reference issue**
```javascript
// Create a circular reference and show why mark-sweep handles it
function createCircular() {
  const a = { name: 'A' };
  const b = { name: 'B' };
  a.ref = b;
  b.ref = a;
  return { a, b }; // Both reachable from root
}
// If we null the root reference, both become unreachable
// Mark-sweep collects both, unlike reference counting
```

## Reference Counting

### What It Is

Reference counting tracks how many references point to each object. When the count reaches zero, the object is immediately freed. This was used in older JavaScript engines but had issues with circular references.

### How It Works

```javascript
let obj1 = { name: 'obj1' }; // ref count: 1
let obj2 = { name: 'obj2' }; // ref count: 1

// Circular reference
obj1.ref = obj2; // obj2 count: 2
obj2.ref = obj1; // obj1 count: 2

obj1 = null; // obj1 count: 1 (still referenced by obj2.ref)
obj2 = null; // obj2 count: 1 (still referenced by obj1.ref)
// Both objects have count > 0, so they're never freed!
// This is why modern JS uses mark-and-sweep, not reference counting
```

### Coding Challenges

**Challenge 1: Implement reference counting**
```javascript
class RefCounted {
  constructor(value) {
    this.value = value;
    this.refCount = 1;
  }
  ref() { this.refCount++; }
  unref() {
    this.refCount--;
    if (this.refCount === 0) {
      console.log(`Freeing: ${this.value}`);
      return true;
    }
    return false;
  }
}
```

**Challenge 2: Demonstrate circular reference leak**
```javascript
function circularLeakDemo() {
  let a = new RefCounted('A');
  let b = new RefCounted('B');
  a.ref = b; b.ref();
  b.ref = a; a.ref();
  a.unref(); // count: 1 (still ref'd by b)
  b.unref(); // count: 1 (still ref'd by a)
  // Both leaked! never freed
}
```

## V8 Garbage Collector

### What It Is

V8 (Chrome's JS engine) uses a generational garbage collector with multiple phases. It divides objects into generations based on age: young (nursery) and old generation.

### Generational Hypothesis

Most objects die young. V8 exploits this by optimizing for short-lived objects.

### V8 GC Phases

1. **Minor GC (Scavenge)**: Collects the young generation. Uses semi-space copy algorithm. Fast, pauses typically < 1ms.

2. **Major GC (Mark-Sweep-Compact)**: Collects the entire heap. Uses mark-and-sweep. Slower, pauses can be 10-100ms.

3. **Incremental GC**: Breaks major GC into small steps between JS execution. Reduces pause times.

4. **Concurrent GC**: GC work happens on background threads while JS executes.

### V8 Memory Structure

```javascript
// You can observe V8 heap from Node.js
console.log('Heap:', process.memoryUsage());

// V8 flags (Node.js)
// --max-old-space-size=2048 (increase heap limit)
// --optimize-for-size
// --gc-interval=100

// Chrome DevTools > Memory > Heap Snapshots
// Shows V8's internal object representation
```

### Orinoco (V8 GC Project)

V8's Orinoco project improved GC by adding:
- Parallel compaction (multiple threads)
- Concurrent marking (mark while JS runs)
- Concurrent sweeping
- Result: 70-80% less GC pause time

### Coding Challenges

**Challenge 1: Simulate generational GC**
```javascript
class GenerationalGC {
  constructor() {
    this.nursery = new Set();
    this.oldGen = new Set();
  }
  allocate(obj) { this.nursery.add(obj); }
  collectNursery() {
    for (const obj of this.nursery) {
      if (obj.survived) this.oldGen.add(obj);
    }
    this.nursery.clear();
  }
}
```

**Challenge 2: Monitor GC pauses**
```javascript
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.entryType === 'gc') {
      console.log(`GC pause: ${entry.duration}ms`);
    }
  }
});
observer.observe({ entryTypes: ['gc'] });
```

## WeakMap and WeakSet

### What It Is

WeakMap and WeakSet are collections that hold weak references to their keys. If the key has no other references, the entry can be garbage collected.

### Why It Is Important

WeakMap prevents memory leaks in scenarios where you need to attach metadata to objects without affecting their lifecycle. The canonical use case is caching, DOM element metadata, and private data.

### How It Works Internally

WeakMap holds a weak reference to keys, meaning the GC can collect the key object even while it's in the WeakMap. The entry is automatically removed when the key is collected.

### Syntax

```javascript
const wm = new WeakMap();
let obj = {};
wm.set(obj, 'private data');
console.log(wm.get(obj)); // 'private data'
obj = null; // obj can be GC'd, entry auto-removed

const ws = new WeakSet();
let obj2 = {};
ws.add(obj2);
console.log(ws.has(obj2)); // true
obj2 = null; // obj2 can be GC'd
```

### Beginner Examples

```javascript
// DOM element metadata without leaks
const elementData = new WeakMap();

function attachData(el, data) {
  elementData.set(el, data);
}

function getData(el) {
  return elementData.get(el);
}

// Even if el is removed from DOM and all references dropped
// the WeakMap entry is auto-cleaned

// Private properties
const privateData = new WeakMap();

class Person {
  constructor(name) {
    privateData.set(this, { name });
  }
  getName() {
    return privateData.get(this).name;
  }
}

// When Person instance is GC'd, private data is also freed
```

### Intermediate Examples

```javascript
// Cache with auto-cleanup
const computedCache = new WeakMap();

function expensiveComputation(obj) {
  if (computedCache.has(obj)) {
    return computedCache.get(obj);
  }
  const result = performExpensiveOp(obj);
  computedCache.set(obj, result);
  return result;
}
// When obj is no longer used elsewhere, cache is auto-cleaned

// Multiple DOM element state
const elementStates = new WeakMap();

function setElementState(el, state) {
  elementStates.set(el, state);
}

function getElementState(el) {
  return elementStates.get(el) || 'idle';
}

// All elements in a document
// When elements are removed, their state is auto-freed

// Event handler association
const handlerMap = new WeakMap();

function addManagedListener(el, event, handler) {
  if (!handlerMap.has(el)) handlerMap.set(el, new Map());
  handlerMap.get(el).set(event, handler);
  el.addEventListener(event, handler);
}

function cleanupElement(el) {
  const handlers = handlerMap.get(el);
  if (handlers) {
    for (const [event, handler] of handlers) {
      el.removeEventListener(event, handler);
    }
    handlerMap.delete(el);
  }
}
```

### Advanced Examples

```javascript
// WeakMap-based observable pattern
const observers = new WeakMap();

function observe(obj, fn) {
  if (!observers.has(obj)) observers.set(obj, new Set());
  observers.get(obj).add(fn);
  return () => observers.get(obj)?.delete(fn);
}

function notify(obj, data) {
  observers.get(obj)?.forEach(fn => fn(data));
}
// When obj is GC'd, all its observers are auto-cleaned

// WeakRef (ES2021) - direct weak reference
let target = { message: 'hello' };
const weakRef = new WeakRef(target);
console.log(weakRef.deref()?.message); // 'hello'
target = null;
// After GC: weakRef.deref() returns undefined

// FinalizationRegistry - cleanup callback when object is GC'd
const registry = new FinalizationRegistry((heldValue) => {
  console.log('Object with value', heldValue, 'was collected');
});

let obj = { sensitive: true };
registry.register(obj, 'some-metadata');
obj = null;
// When GC runs, the callback fires

// Practical: pool cleanup
const pool = new FinalizationRegistry((key) => {
  console.log('Returning', key, 'to pool');
  availableSlots.push(key);
});
```

### Real-World Use Cases

- **jQuery.data()**: Stores data on DOM elements without preventing GC.
- **Vue.js reactivity**: WeakMap for reactive dependencies.
- **React Fiber**: WeakMap for component data.
- **Memonization caches**: To avoid preventing GC of unused keys.
- **Private instance data**: Encapsulation pattern.

### Common Mistakes

```javascript
// Mistake: WeakMap keys must be objects
const wm = new WeakMap();
wm.set('string', 'value'); // TypeError: Invalid value used as weak map key

// Mistake: WeakMap is not iterable
for (const [k, v] of wm) {} // TypeError: wm is not iterable

// Mistake: Assuming WeakRef.deref() always works
const ref = new WeakRef({});
// After GC: ref.deref() could be undefined
// Always check: if (ref.deref()) { /* use it */ }

// Mistake: WeakMap doesn't prevent GC of values
// Values can be GC'd if no other references exist (only keys are weak)
```

### Best Practices

```javascript
// Use WeakMap for metadata on DOM elements
// Use WeakMap for private data in classes
// Use WeakSet for tracking without preventing GC
// Use WeakRef for caches where entries can be recreated
// Always check deref() before using WeakRef

// Cache pattern with WeakRef
class WeakCache {
  constructor() { this.cache = new Map(); }
  get(key) {
    const ref = this.cache.get(key);
    if (ref) {
      const val = ref.deref();
      if (val !== undefined) return val;
      this.cache.delete(key); // Clean up collected entry
    }
    return null;
  }
  set(key, value) {
    this.cache.set(key, new WeakRef(value));
  }
}
```

### Performance Considerations

- WeakMap lookups are O(1) but slightly slower than Map due to weak reference handling.
- WeakMap is not iterable (intentionally) for security and performance.
- FinalizationRegistry callbacks run on the microtask queue after GC.
- WeakRef.deref() is synchronous but may trigger a GC if the object was collected.
- Overusing WeakRef can slow down GC since it must track more references.

### Interview Questions

**Q: How does mark-and-sweep solve the circular reference problem?**
A: Mark-and-sweep starts from roots and follows references. Objects in a circular reference are only kept alive if reachable from a root. If no root references the circle, both are collected, unlike reference counting which would leak.

**Q: What is the difference between a WeakMap and a regular Map?**
A: WeakMap holds weak references to its keys, meaning the GC can collect the key object even while it's in the WeakMap. Map holds strong references, preventing GC of keys. WeakMap is not iterable and keys must be objects.

**Q: What are the V8 GC generations?**
A: V8 has young (nursery) and old generations. Most objects are allocated in the nursery. Objects surviving one GC move to the intermediate space. Objects surviving two GCs are promoted to the old generation, which is collected less frequently.

### Related Topics

- Memory management, WeakRef/FinalizationRegistry, V8 internals

### Coding Challenges

**Challenge 1: Implement a WeakMap wrapper with logging**
```javascript
class ObservableWeakMap {
  constructor() {
    this._wm = new WeakMap();
    this._collected = new FinalizationRegistry((key) => {
      console.log(`Key collected: ${key}`);
    });
  }
  set(key, value) {
    this._wm.set(key, value);
    this._collected.register(key, key.description || 'unknown');
  }
  get(key) { return this._wm.get(key); }
}
```

**Challenge 2: Memory-safe DOM element tracker**
```javascript
const tracked = new WeakMap();
function trackElement(el) {
  tracked.set(el, { attached: document.contains(el) });
  return () => {
    if (!document.contains(el)) {
      console.log('Element removed from DOM');
    }
  };
}
```

**Challenge 3: Cache with WeakRef values**
```javascript
class WeakValueMap {
  constructor() { this.cache = new Map(); }
  set(key, value) { this.cache.set(key, new WeakRef(value)); }
  get(key) {
    const ref = this.cache.get(key);
    if (!ref) return undefined;
    const val = ref.deref();
    if (!val) this.cache.delete(key);
    return val;
  }
}
```
