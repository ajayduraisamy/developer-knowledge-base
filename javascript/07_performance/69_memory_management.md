# Memory Management - Memory lifecycle, stack vs heap, memory leaks

## Introduction

Memory management in JavaScript is the process of allocating, using, and releasing memory for program execution. JavaScript engines handle most memory management automatically via garbage collection, but understanding the underlying mechanics is crucial for writing performant, leak-free applications.

## Memory Lifecycle

### What It Is

The memory lifecycle has three phases: allocate, use, and release. JavaScript handles allocation and release automatically, but developers must understand when memory is retained.

### Why It Is Important

Understanding the memory lifecycle helps developers write efficient code that avoids unnecessary allocation and prevents memory leaks. Knowing when memory is allocated (variable declarations, function calls, object creation) and when it is freed (scope exit, reference removal) enables better optimization decisions.

### How It Works Internally

1. **Allocate**: When you declare variables, create objects, or call functions, the engine allocates memory. Primitives are allocated on the stack; objects on the heap.
2. **Use**: Reading/writing variable values, accessing object properties, calling functions.
3. **Release**: The garbage collector identifies memory that is no longer reachable and frees it.

### Syntax

```javascript
// Allocation
let a = 42; // primitive on stack
let obj = { name: 'Alice' }; // object on heap
let arr = [1, 2, 3]; // array on heap

// Use
console.log(a, obj.name, arr[0]);

// Release happens automatically when no longer reachable
obj = null; // Object becomes eligible for GC
```

### Beginner Examples

```javascript
// Function call creates a stack frame
function greet(name) {
  let message = 'Hello, ' + name; // Allocated on call
  return message;
} // message released after return

// Global scope variables persist for app lifetime
let globalCache = {}; // Never released unless set to null

// Block scope with let/const
function process() {
  if (true) {
    let blockVar = 'temp'; // Released after block
    const blockObj = { data: 'temp' }; // Object unreachable after block
  }
}
```

### Intermediate Examples

```javascript
// Closures retain references
function createCounter() {
  let count = 0; // Retained as long as the returned function exists
  return function() { return ++count; };
}
const counter = createCounter(); // count is NOT released
counter(); // 1
counter(); // 2

// Event listeners retain references
function setup() {
  const largeData = new Array(1000000).fill('*');
  element.addEventListener('click', () => {
    console.log('clicked'); // largeData is in closure!
  });
} // largeData cannot be GC'd because listener references it

// WeakMap - keys don't prevent GC
const cache = new WeakMap();
let key = { id: 1 };
cache.set(key, 'expensive data');
key = null; // Entry is automatically removed when key is GC'd
```

### Advanced Examples

```javascript
// Memory profiling utilities
class MemoryTracker {
  constructor() {
    this.snapshots = [];
    this.interval = null;
  }
  start(intervalMs = 5000) {
    this.interval = setInterval(() => {
      if (performance.memory) {
        this.snapshots.push({
          timestamp: Date.now(),
          usedJSHeapSize: performance.memory.usedJSHeapSize,
          totalJSHeapSize: performance.memory.totalJSHeapSize,
          jsHeapSizeLimit: performance.memory.jsHeapSizeLimit
        });
      }
    }, intervalMs);
  }
  stop() { clearInterval(this.interval); }
  getTrend() {
    if (this.snapshots.length < 2) return null;
    const first = this.snapshots[0].usedJSHeapSize;
    const last = this.snapshots[this.snapshots.length - 1].usedJSHeapSize;
    return { growth: last - first, percentage: ((last - first) / first * 100).toFixed(1) + '%' };
  }
}

// Manual memory management pattern (for pools)
class ObjectPool {
  constructor(factory, reset, initialSize = 100) {
    this.factory = factory;
    this.reset = reset;
    this.pool = [];
    this.active = new Set();
    for (let i = 0; i < initialSize; i++) this.pool.push(factory());
  }
  acquire() {
    const obj = this.pool.pop() || this.factory();
    this.active.add(obj);
    return obj;
  }
  release(obj) {
    this.reset(obj);
    this.active.delete(obj);
    this.pool.push(obj);
  }
  get activeCount() { return this.active.size; }
  get poolSize() { return this.pool.length; }
}

// Large object disposal
class LargeDataProcessor {
  constructor() {
    this.buffer = null;
  }
  load() {
    this.buffer = new ArrayBuffer(100 * 1024 * 1024); // 100MB
  }
  process() { /* ... */ }
  dispose() {
    this.buffer = null; // Explicit release
  }
  async [Symbol.asyncDispose]() {
    this.dispose();
  }
}

// Usage with using (ES2025)
// await using processor = new LargeDataProcessor();
// processor.load();
// processor.process();
// Auto-disposed when leaving scope
```

### Real-World Use Cases

- Memory profiling and leak detection in long-running SPAs
- Optimizing server-side memory in Node.js applications handling thousands of requests
- Game development where consistent frame rates require controlled allocation
- Data-intensive applications (processing large datasets, video, images)
- Embedded JavaScript environments with limited memory (IoT)

### Common Mistakes

```javascript
// Mistake: Assuming block scope frees all variables
function process() {
  const largeData = new Array(1000000);
  if (true) {
    const temp = largeData; // Reference retained!
  }
  // largeData still referenced, not freed
  // Fix: nullify when done
}

// Mistake: Not understanding closure retention
function createHandler() {
  const heavy = new Array(100000).fill('x');
  return function() {
    console.log('click');
    // heavy is kept alive even though not used here
  };
}

// Mistake: Holding references to DOM elements
const elements = [];
document.querySelectorAll('.item').forEach(el => {
  elements.push(el);
});
// Even after DOM removal, elements persist
```

### Coding Challenges

**Challenge 1: Detect memory leak pattern**
```javascript
// Given this code, identify and fix the leak
function setupLogger() {
  const logs = [];
  setInterval(() => {
    logs.push({ time: Date.now(), message: 'tick' });
  }, 1000);
  return () => console.log('Logger running');
}
// Fix: limit log size or use weak references
function setupLoggerFixed() {
  const logs = [];
  const MAX_LOGS = 100;
  setInterval(() => {
    logs.push({ time: Date.now(), message: 'tick' });
    if (logs.length > MAX_LOGS) logs.shift();
  }, 1000);
  return () => console.log('Logger running');
}
```

**Challenge 2: Implement an LRU cache**
```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }
  get(key) {
    if (!this.cache.has(key)) return -1;
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }
  put(key, value) {
    if (this.cache.has(key)) this.cache.delete(key);
    else if (this.cache.size >= this.capacity) {
      this.cache.delete(this.cache.keys().next().value);
    }
    this.cache.set(key, value);
  }
}
```

**Challenge 3: Object pool for vector operations**
```javascript
class VectorPool {
  constructor(size = 100) {
    this.pool = Array.from({ length: size }, () => ({ x: 0, y: 0 }));
  }
  acquire(x = 0, y = 0) {
    let v = this.pool.pop();
    if (!v) v = { x, y };
    else { v.x = x; v.y = y; }
    return v;
  }
  release(v) {
    this.pool.push(v);
  }
}
```

## Stack vs Heap

### What It Is

The stack stores primitives and function call frames. The heap stores objects, arrays, and closures. Stack memory is automatically managed with function calls. Heap memory is managed by the garbage collector.

### Why It Is Important

Knowing what goes on the stack vs heap helps developers optimize memory usage. Stack allocation is extremely fast but limited in size. Heap allocation is more flexible but slower and requires GC. Choosing the right memory region is critical for performance.

### How It Works Internally

**Stack**: LIFO data structure. Each function call creates a stack frame with local variables, return address, and parameters. Frames are pushed on call and popped on return. Very fast, fixed-size per thread.

**Heap**: Dynamically allocated memory. Objects of any size can be allocated. Managed by the garbage collector. Slower allocation but more flexible. Subject to fragmentation.

### Examples

```javascript
// Stack: primitives and references
let x = 10; // Stored on stack
let y = x; // Copied on stack
x = 20;
console.log(y); // 10 (copy)

// Heap: objects
let obj1 = { value: 10 }; // Reference on stack, object on heap
let obj2 = obj1; // Reference copied on stack, same heap object
obj1.value = 20;
console.log(obj2.value); // 20 (same object)

// Stack overflow (recursion)
function recurse() { recurse(); }
// RangeError: Maximum call stack size exceeded

// Heap fragmentation
let arr = [];
for (let i = 0; i < 10000; i++) {
  arr.push({ id: i, data: new Array(100).fill('x') });
}
// Creates many heap objects, potentially causing fragmentation
```

### Real-World Use Cases

- Stack: function calls, local primitives, recursion (tree traversal)
- Heap: large datasets, dynamic objects, shared state
- Hybrid: closures capture stack variables into heap when function escapes
- Optimizing: moving heap allocations to stack when possible (escape analysis)

### Common Mistakes

```javascript
// Mistake: Creating too many objects in tight loops (heap pressure)
for (let i = 0; i < 100000; i++) {
  const temp = { value: i }; // New heap allocation each iteration
}
// Fix: reuse objects or use primitives

// Mistake: Deep recursion causing stack overflow
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}
// For n = 100000, stack overflow
// Fix: use iteration or tail recursion
```

### Coding Challenges

**Challenge 1: Determine stack vs heap allocation**
```javascript
function analyzeAllocation() {
  const a = 42; // stack
  const b = 'hello'; // stack (interned)
  const c = { value: 42 }; // heap
  const d = [1, 2, 3]; // heap
  const e = () => {}; // heap (closure)
  console.log({ a, b, c, d, e });
}
```

**Challenge 2: Safe recursion with trampolining**
```javascript
function trampoline(fn) {
  return (...args) => {
    let result = fn(...args);
    while (typeof result === 'function') result = result();
    return result;
  };
}

const sum = trampoline(function f(n, acc = 0) {
  return n <= 0 ? acc : () => f(n - 1, acc + n);
});

console.log(sum(100000)); // 5000050000
```

**Challenge 3: Detect stack overflow depth**
```javascript
function measureStackDepth(depth = 0) {
  try {
    return measureStackDepth(depth + 1);
  } catch {
    return depth;
  }
}
console.log('Max depth:', measureStackDepth());
```

## Common Memory Leaks

### What It Is

Memory leaks occur when memory that is no longer needed is not released, causing the application to consume increasing amounts of memory over time.

### Types of Leaks

1. **Accidental globals**: Assigning to undeclared variables creates global properties.
2. **Forgotten timers/intervals**: setInterval callbacks holding references.
3. **Detached DOM nodes**: Removing DOM elements but keeping JS references.
4. **Closures retaining large objects**: Callbacks referencing large data.
5. **Event listener leaks**: Adding listeners without removal.
6. **Cache without bounds**: Maps and objects growing unbounded.
7. **Circular references**: Objects referencing each other (less of an issue with modern GC).

### Leak Examples

```javascript
// Leak 1: Accidental globals
function leak() {
  leaked = 'global variable'; // No let/const/var - becomes global!
}

// Leak 2: Forgotten interval
function startTimer() {
  const data = new Array(1000000);
  setInterval(() => {
    console.log(data.length); // data never released!
  }, 1000);
}
// Fix: clearInterval, don't reference large data unnecessarily

// Leak 3: Detached DOM nodes
let detachedNodes = [];
document.querySelectorAll('.item').forEach(el => {
  detachedNodes.push(el);
  el.remove(); // Removed from DOM but still referenced
});
// Fix: Clear the array when done

// Leak 4: Growing cache
const userCache = new Map();
function cacheUser(user) {
  userCache.set(user.id, user); // Never cleaned up!
}
// Fix: Limit cache size or use WeakMap

// Leak 5: Closure retaining large scope
function outer() {
  const largeArray = new Array(1000000).fill('data');
  return function inner() {
    // Only needs a small part, but holds entire largeArray
    console.log('hello');
  };
}
// Fix: Only reference what you need
```

### Detection Tools

```javascript
// Chrome DevTools: Performance > Memory
// Chrome DevTools: Memory > Heap Snapshots
// Node.js: --inspect flag for Chrome DevTools
// Node.js: process.memoryUsage()
console.log(process.memoryUsage());
// { rss, heapTotal, heapUsed, external, arrayBuffers }

// Node.js heap dump
// require('v8').writeHeapSnapshot();

// Performance Observer API
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.entryType === 'measure') {
      console.log(`${entry.name}: ${entry.duration}ms`);
    }
  }
});
observer.observe({ entryTypes: ['measure', 'resource'] });
```

### Best Practices

```javascript
// 1. Always clean up
function setup() {
  const handler = () => { /* ... */ };
  window.addEventListener('resize', handler);
  return () => window.removeEventListener('resize', handler);
}

// 2. Limit caches
class LRUCache {
  constructor(max = 100) { this.max = max; this.cache = new Map(); }
  get(k) {
    if (!this.cache.has(k)) return null;
    const v = this.cache.get(k);
    this.cache.delete(k);
    this.cache.set(k, v);
    return v;
  }
  set(k, v) {
    this.cache.delete(k);
    this.cache.set(k, v);
    if (this.cache.size > this.max) {
      this.cache.delete(this.cache.keys().next().value);
    }
  }
}

// 3. Use WeakMap/WeakSet for caches
const widgetData = new WeakMap();
// Keys are objects; entries auto-removed when key is GC'd

// 4. Avoid global state accumulation
function processData(items) {
  // Local scope - cleaned up after function returns
  const results = items.map(expensiveOp);
  return results;
}
```

### Performance Considerations

- Stack allocation is nanoseconds; heap allocation is microseconds.
- Frequent GC pauses affect frame rate and UX.
- Object pools reduce GC pressure for frequently created objects.
- Memory fragmentation can cause out-of-memory errors even with available space.
- ArrayBuffers and TypedArrays bypass GC but must be manually managed.

### Interview Questions

**Q: What's the difference between stack and heap memory?**
A: Stack stores primitives and function call frames in LIFO order. Fast, fixed-size, automatically managed. Heap stores objects in dynamically allocated memory. Slower, garbage-collected, can hold any size.

**Q: How do closures affect memory management?**
A: Closures retain references to their outer scope variables. If a closure outlives the outer function, all variables in the closure's scope chain are retained, preventing GC. This is a common source of memory leaks.

**Q: How would you debug a memory leak in production?**
A: (1) Take heap snapshots at different times to identify growing objects. (2) Use performance.memory to track usage trends. (3) Use Chrome DevTools' allocation instrumentation on timelines. (4) For Node.js, use --inspect with Chrome DevTools or heap dump analysis.

### Related Topics

- Garbage collection, WeakRef/WeakMap, Object pools, Stack traces

### Coding Challenges

**Challenge 1: Fix the memory leak**
```javascript
function createLeakyComponent(container) {
  const data = fetchLargeData();
  container.addEventListener('click', function handler() {
    console.log(data.length);
  });
  // Fix: allow cleanup
  return () => container.removeEventListener('click', handler);
}
```

**Challenge 2: Implement a self-cleaning cache**
```javascript
class SelfCleaningCache {
  constructor(ttl = 60000) {
    this.cache = new Map();
    this.ttl = ttl;
  }
  set(key, value) {
    this.cache.set(key, { value, expiry: Date.now() + this.ttl });
  }
  get(key) {
    const entry = this.cache.get(key);
    if (!entry) return undefined;
    if (Date.now() > entry.expiry) {
      this.cache.delete(key);
      return undefined;
    }
    return entry.value;
  }
}
```

**Challenge 3: Detect circular references in an object**
```javascript
function hasCircularReference(obj) {
  const seen = new WeakSet();
  function detect(value) {
    if (typeof value === 'object' && value !== null) {
      if (seen.has(value)) return true;
      seen.add(value);
      for (const key of Object.keys(value)) {
        if (detect(value[key])) return true;
      }
    }
    return false;
  }
  return detect(obj);
}
```

**Challenge 4: Implement a proper cleanup pattern**
```javascript
class ResourceManager {
  constructor() {
    this.resources = new Set();
  }
  register(resource) {
    this.resources.add(resource);
    return () => this.resources.delete(resource);
  }
  cleanup() {
    for (const res of this.resources) {
      if (res.dispose) res.dispose();
    }
    this.resources.clear();
  }
}
```
