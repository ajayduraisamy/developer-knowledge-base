# Concurrency Model - Single-threaded concurrency, worker threads, shared memory, atomics

## Introduction

JavaScript's concurrency model is unique: it's single-threaded for JavaScript execution but leverages asynchronous I/O and worker threads for parallelism. The event loop handles concurrency for I/O-bound tasks, while worker threads enable true parallelism for CPU-intensive work. Understanding this model is crucial for building performant applications that effectively utilize system resources.

## Single-Threaded Event Loop

### What It Is

JavaScript runs on a single thread (the main thread) for code execution. Concurrency is achieved through the event loop, which processes asynchronous callbacks. This model avoids the complexity of multi-threaded programming (race conditions, deadlocks, shared state) while providing excellent performance for I/O-bound workloads.

### Why It Is Important

The single-threaded model is both a strength and a constraint. It simplifies programming (no locks needed for most code) but requires careful management of CPU-intensive tasks (which block the entire process). Understanding this model helps developers avoid blocking the event loop and design efficient async code.

### How It Works Internally

The event loop processes tasks in order:
1. Execute synchronous code in the current execution context
2. Drain the microtask queue (Promise callbacks, process.nextTick)
3. Process the next macrotask (setTimeout, I/O callback, setImmediate)
4. Repeat

While the main thread is blocked (running CPU-intensive JavaScript), no other code runs—no I/O callbacks, no timer callbacks, no UI updates.

```javascript
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
// Output: 1, 4, 3, 2
// Single thread: all execution happens sequentially on one thread
```

### Syntax

```javascript
// Event loop under the microscope
const start = Date.now();

// Blocking operation (simulated)
function block(ms) {
  const end = Date.now() + ms;
  while (Date.now() < end) {} // Busy wait - blocks event loop!
}

setTimeout(() => console.log('Timeout after block'), 10);
block(100); // Blocks for 100ms
console.log('After block');

// Output:
// After block (immediately after 100ms)
// Timeout after block (right after)

// Non-blocking version
setTimeout(() => console.log('Timeout'), 10);
setTimeout(() => {
  console.log('Non-blocking operation');
}, 100);
console.log('Immediate');
// Output: Immediate, Timeout, Non-blocking operation
```

### Beginner Examples

```javascript
// Why blocking the event loop is bad
const http = require('http');

// BAD: This blocks ALL requests
http.createServer((req, res) => {
  if (req.url === '/compute') {
    const start = Date.now();
    while (Date.now() - start < 5000) {} // Blocks for 5 seconds!
    res.end('Done');
  }
});

// GOOD: Offload CPU work
http.createServer((req, res) => {
  if (req.url === '/compute') {
    // Either use setTimeout to yield:
    const start = Date.now();
    const chunk = () => {
      if (Date.now() - start < 5000) {
        setImmediate(chunk); // Yield to event loop
      } else {
        res.end('Done');
      }
    };
    setImmediate(chunk);
  }
});
// Or better: use Worker Threads
```

### Intermediate Examples

```javascript
// Event loop starvation detection
let lastCheck = Date.now();
setInterval(() => {
  const now = Date.now();
  const lag = now - lastCheck - 1000; // Expected interval
  if (lag > 50) {
    console.warn(`Event loop lag: ${lag}ms`);
  }
  lastCheck = now;
}, 1000);

// CPU-intensive work should NOT run on main thread
function calculatePrimes(limit) {
  const primes = [];
  for (let i = 2; i <= limit; i++) {
    let isPrime = true;
    for (let j = 2; j <= Math.sqrt(i); j++) {
      if (i % j === 0) { isPrime = false; break; }
    }
    if (isPrime) primes.push(i);
  }
  return primes;
}

// Instead of calling calculatePrimes on main thread:
// Move to worker thread or use async scheduling

// Chunked execution to keep event loop responsive
function asyncChunkedPrimes(limit, chunkSize = 1000) {
  return new Promise((resolve) => {
    const primes = [];
    let current = 2;
    
    function processChunk() {
      const end = Math.min(current + chunkSize, limit);
      for (; current < end; current++) {
        let isPrime = true;
        for (let j = 2; j <= Math.sqrt(current); j++) {
          if (current % j === 0) { isPrime = false; break; }
        }
        if (isPrime) primes.push(current);
      }
      
      if (current >= limit) {
        resolve(primes);
      } else {
        setImmediate(processChunk); // Yield to event loop
      }
    }
    
    processChunk();
  });
}
```

## Worker Threads

### What It Is

Worker threads provide true parallelism by running JavaScript on separate operating system threads, each with its own V8 instance, event loop, and memory heap. Workers communicate with the main thread through message passing, avoiding shared-state race conditions.

### Why It Is Important

Worker threads are the correct solution for CPU-intensive operations. They prevent blocking the main event loop for tasks like image processing, data transformation, ML inference, and cryptographic operations. Unlike the cluster module (which creates processes), workers share the same process and can transfer memory efficiently.

### How It Works Internally

Each worker thread:
- Has its own V8 runtime instance (heap, GC, event loop)
- Has its own Node.js environment (require, process)
- Cannot access the main thread's memory directly (no shared state)
- Communicates via message passing (structured clone algorithm)
- Can transfer ArrayBuffers (zero-copy) using Atomics for synchronization

The worker is spawned in a separate OS thread. Thread creation has overhead (~2-5ms), so workers should be long-lived, not created per-task.

### Syntax

```javascript
// main.js
const { Worker } = require('worker_threads');

// Create a worker
const worker = new Worker('./worker.js', {
  workerData: { start: 1, end: 1000000 }
});

// Listen for messages from worker
worker.on('message', (result) => {
  console.log('Result:', result);
});

worker.on('error', (err) => console.error('Worker error:', err));
worker.on('exit', (code) => {
  if (code !== 0) console.error(`Worker exited with code ${code}`);
});

// Send message to worker
worker.postMessage({ command: 'process', data: [1, 2, 3] });

// worker.js
const { parentPort, workerData } = require('worker_threads');

console.log('Worker started with:', workerData);

// Receive messages from main thread
parentPort.on('message', (msg) => {
  if (msg.command === 'process') {
    const result = msg.data.map(x => x * 2);
    parentPort.postMessage(result);
  }
});

// Send result to main thread
parentPort.postMessage({ done: true });
```

### Beginner Examples

```javascript
// Worker for CPU-intensive prime calculation

// prime-worker.js
const { parentPort, workerData } = require('worker_threads');

function calculatePrimes(limit) {
  const primes = [];
  for (let i = 2; i <= limit; i++) {
    let isPrime = true;
    for (let j = 2; j <= Math.sqrt(i); j++) {
      if (i % j === 0) { isPrime = false; break; }
    }
    if (isPrime) primes.push(i);
  }
  return primes;
}

const result = calculatePrimes(workerData.limit);
parentPort.postMessage(result);

// main.js
const { Worker } = require('worker_threads');
const http = require('http');

http.createServer((req, res) => {
  if (req.url === '/primes') {
    const worker = new Worker('./prime-worker.js', {
      workerData: { limit: 5000000 }
    });
    
    worker.on('message', (primes) => {
      res.end(`Found ${primes.length} primes`);
    });
    
    worker.on('error', (err) => {
      res.statusCode = 500;
      res.end(err.message);
    });
  } else {
    res.end('OK');
  }
}).listen(3000);
// Main thread stays responsive even during prime calculation!
```

### Intermediate Examples

```javascript
// Worker pool for handling multiple tasks
const { Worker } = require('worker_threads');

class WorkerPool {
  constructor(workerFile, size = 4) {
    this.workers = [];
    this.free = [];
    this.queue = [];
    
    for (let i = 0; i < size; i++) {
      const worker = new Worker(workerFile);
      this.workers.push(worker);
      this.free.push(worker);
      
      worker.on('message', (result) => {
        const resolve = worker._currentResolve;
        delete worker._currentResolve;
        this.free.push(worker);
        resolve(result);
        this.processQueue();
      });
      
      worker.on('error', (err) => {
        // Remove broken worker, create replacement
      });
    }
  }
  
  exec(data) {
    return new Promise((resolve, reject) => {
      const task = { data, resolve, reject };
      if (this.free.length > 0) {
        this.runTask(this.free.pop(), task);
      } else {
        this.queue.push(task);
      }
    });
  }
  
  runTask(worker, task) {
    worker._currentResolve = task.resolve;
    worker.postMessage(task.data);
  }
  
  processQueue() {
    if (this.queue.length > 0 && this.free.length > 0) {
      const task = this.queue.shift();
      this.runTask(this.free.pop(), task);
    }
  }
  
  terminate() {
    this.workers.forEach(w => w.terminate());
  }
}

// Usage
const pool = new WorkerPool('./image-worker.js', 4);
const results = await Promise.all([
  pool.exec({ image: img1 }),
  pool.exec({ image: img2 }),
  pool.exec({ image: img3 })
]);
```

### Advanced Examples

```javascript
// Worker thread with transferable objects
// ArrayBuffers can be transferred (zero-copy)

// main.js
const { Worker } = require('worker_threads');

function processLargeBuffer(size) {
  const buffer = new SharedArrayBuffer(size);
  const view = new Uint8Array(buffer);
  
  // Fill with data
  for (let i = 0; i < size; i++) view[i] = i % 256;
  
  const worker = new Worker('./buffer-worker.js');
  
  // Transfer ownership (zero-copy, main thread loses access)
  worker.postMessage({ buffer }, [buffer]);
  // NOTE: after transfer, view is detached
  
  worker.on('message', (result) => {
    console.log('Processed bytes:', result.processed);
  });
}

// buffer-worker.js
const { parentPort } = require('worker_threads');

parentPort.on('message', (msg) => {
  const buffer = msg.buffer;
  const view = new Uint8Array(buffer);
  
  // Process data in place
  for (let i = 0; i < view.length; i++) {
    view[i] = view[i] * 2;
  }
  
  // Buffer is now modified in place
  parentPort.postMessage({ processed: view.length });
  // NOTE: we must NOT transfer back - main thread lost access
});

// Worker thread lifecycle management
class ManagedWorker {
  constructor(workerFile) {
    this.worker = new Worker(workerFile);
    this.alive = true;
    
    this.worker.on('exit', (code) => {
      this.alive = false;
      if (code !== 0) this.restart();
    });
    
    this.worker.on('error', () => this.restart());
  }
  
  restart() {
    if (!this.alive) return;
    this.worker.terminate();
    this.worker = new Worker(this.worker.options);
    this.alive = true;
  }
  
  async postMessage(data) {
    if (!this.alive) throw new Error('Worker is dead');
    this.worker.postMessage(data);
  }
}
```

## SharedArrayBuffer

### What It Is

`SharedArrayBuffer` is a fixed-length binary data buffer that can be shared between the main thread and worker threads (or between workers). Unlike regular ArrayBuffers (which are transferred and become inaccessible in the sender), SharedArrayBuffers remain accessible to all threads simultaneously.

### Why It Is Important

SharedArrayBuffer enables true shared memory parallelism. Multiple threads can read and write the same memory without copying data. Combined with Atomics, it enables efficient parallel algorithms, zero-copy data sharing between workers, and coordination without message passing overhead.

### How It Works Internally

The SharedArrayBuffer is allocated in shared memory visible to all threads. The operating system maps the same physical memory pages into each thread's address space. Writes by one thread are immediately visible to all other threads (though proper synchronization requires Atomics to avoid CPU cache coherence issues).

### Syntax

```javascript
// SharedArrayBuffer creation
const sab = new SharedArrayBuffer(1024); // 1KB shared memory
const view = new Int32Array(sab);

// In worker:
const sab = new SharedArrayBuffer(1024);
const view = new Int32Array(sab);

// Post the SAB reference (not transferred, both threads keep access)
worker.postMessage({ sharedBuffer: sab }); // NOT transferred, just referenced
```

### Beginner Examples

```javascript
// Shared counter between main thread and worker

// main.js
const { Worker } = require('worker_threads');

const sharedBuffer = new SharedArrayBuffer(4); // 4 bytes for one int32
const counter = new Int32Array(sharedBuffer);
counter[0] = 0;

const worker = new Worker('./worker.js');
worker.postMessage({ sharedBuffer });

// Also increment from main thread
setInterval(() => {
  Atomics.add(counter, 0, 1);
  console.log('Main incremented, counter:', counter[0]);
}, 1000);

// worker.js
const { parentPort } = require('worker_threads');

parentPort.on('message', (msg) => {
  const counter = new Int32Array(msg.sharedBuffer);
  
  setInterval(() => {
    Atomics.add(counter, 0, 1);
    console.log('Worker incremented, counter:', counter[0]);
  }, 1500);
});
```

## Atomics API

### What It Is

The Atomics API provides atomic operations for shared memory access, ensuring that operations on SharedArrayBuffer are thread-safe. Atomics guarantees that reads and writes are not subject to race conditions and provides wait/notify for synchronization between threads.

### Why It Is Important

Without Atomics, shared memory access is unsafe due to CPU caching and instruction reordering. One thread might not see another thread's writes, or might see them in a different order. Atomics provides the memory ordering guarantees needed for correct parallel algorithms.

### How It Works Internally

Atomic operations ensure:
1. **Atomicity**: The operation completes without interruption (no torn reads/writes)
2. **Visibility**: The result is immediately visible to all threads
3. **Ordering**: Memory operations before/after the atomic are properly ordered (sequentially consistent)

The CPU uses special instructions (like `LOCK CMPXCHG` on x86) for atomic operations, which invalidate other cores' cache lines.

### Syntax

```javascript
const sab = new SharedArrayBuffer(16);
const view = new Int32Array(sab);

// Atomic read and write
Atomics.store(view, 0, 42);
const value = Atomics.load(view, 0);

// Atomic modify
Atomics.add(view, 0, 5);    // view[0] += 5
Atomics.sub(view, 0, 3);    // view[0] -= 3
Atomics.and(view, 0, 0xFF); // view[0] &= 0xFF
Atomics.or(view, 0, 0x80);  // view[0] |= 0x80
Atomics.xor(view, 0, 0x0F); // view[0] ^= 0x0F

// Atomic exchange
const old = Atomics.exchange(view, 0, 100);
const expected = Atomics.compareExchange(view, 0, 100, 200); // If 100, set 200

// Wait and notify (for synchronization)
// In worker A:
Atomics.store(view, 0, 1);
Atomics.notify(view, 0, 1); // Wake up 1 waiter

// In worker B:
Atomics.wait(view, 0, 0); // Wait until view[0] !== 0
```

### Beginner Examples

```javascript
// Producer-consumer with Atomics

// producer-worker.js
const { parentPort } = require('worker_threads');

parentPort.on('message', ({ buffer, items }) => {
  const view = new Int32Array(buffer);
  const CAPACITY = view.length - 2; // Reserve 2 slots for head/tail
  
  for (const item of items) {
    // Wait until there's space
    let tail, head;
    do {
      tail = Atomics.load(view, 0);
      head = Atomics.load(view, 1);
    } while ((tail + 1) % (CAPACITY + 2) === head);
    
    // Write item
    Atomics.store(view, tail, item);
    Atomics.store(view, 0, (tail + 1) % (CAPACITY + 2));
    Atomics.notify(view, 1, 1); // Signal consumer
  }
  
  // Signal done
  Atomics.store(view, -1, -1);
  Atomics.notify(view, 1, 1);
});

// consumer-worker.js
const { parentPort } = require('worker_threads');

parentPort.on('message', ({ buffer }) => {
  const view = new Int32Array(buffer);
  
  while (true) {
    let head = Atomics.load(view, 1);
    const tail = Atomics.load(view, 0);
    
    if (head === tail) {
      // Check if producer is done
      if (Atomics.load(view, -1) === -1) break;
      Atomics.wait(view, 1, head); // Wait for signal
      continue;
    }
    
    const item = Atomics.load(view, head);
    Atomics.store(view, 1, (head + 1) % view.length);
    console.log('Consumed:', item);
  }
  
  console.log('Consumer done');
});
```

### Intermediate Examples

```javascript
// Parallel sum computation with Atomics
const { Worker } = require('worker_threads');

async function parallelSum(array, threadCount = 4) {
  const chunkSize = Math.ceil(array.length / threadCount);
  const sharedBuffer = new SharedArrayBuffer(4); // For result
  const result = new Int32Array(sharedBuffer);
  result[0] = 0;
  
  const workers = [];
  for (let i = 0; i < threadCount; i++) {
    const start = i * chunkSize;
    const end = Math.min(start + chunkSize, array.length);
    const chunk = array.slice(start, end);
    
    const worker = new Worker('./sum-worker.js');
    worker.postMessage({ data: chunk, sharedBuffer });
    workers.push(worker);
  }
  
  // Wait for all workers to finish
  await Promise.all(workers.map(w => new Promise(r => w.on('exit', r))));
  
  return result[0];
}

// sum-worker.js
const { parentPort } = require('worker_threads');

parentPort.on('message', ({ data, sharedBuffer }) => {
  const result = new Int32Array(sharedBuffer);
  let sum = 0;
  
  for (const val of data) {
    sum += val;
  }
  
  // Atomically add to shared result
  Atomics.add(result, 0, sum);
  
  // Exit worker
  process.exit(0);
});

// Spin-lock with Atomics (use with caution!)
class SpinLock {
  constructor(sharedBuffer) {
    this.lock = new Int32Array(sharedBuffer);
  }
  
  acquire() {
    while (true) {
      // Try to set from 0 to 1 atomically
      const old = Atomics.compareExchange(this.lock, 0, 0, 1);
      if (old === 0) return; // Acquired!
      
      // Wait for notification (more efficient than spinning)
      Atomics.wait(this.lock, 0, 1, 100);
    }
  }
  
  release() {
    Atomics.store(this.lock, 0, 0);
    Atomics.notify(this.lock, 0, 1);
  }
}
```

### Advanced Examples

```javascript
// Lock-free ring buffer with Atomics
class LockFreeQueue {
  constructor(capacity) {
    this.buffer = new SharedArrayBuffer((capacity + 2) * 4);
    this.view = new Int32Array(this.buffer);
    // Index 0: head (read position)
    // Index 1: tail (write position)
    // Indices 2+: data slots
    this.capacity = capacity;
    Atomics.store(this.view, 0, 2); // head starts at index 2
    Atomics.store(this.view, 1, 2); // tail starts at index 2
  }
  
  enqueue(value) {
    while (true) {
      const tail = Atomics.load(this.view, 1);
      const head = Atomics.load(this.view, 0);
      const nextTail = tail === this.capacity + 1 ? 2 : tail + 1;
      
      if (nextTail === head) return false; // Queue full
      
      const old = Atomics.compareExchange(this.view, 1, tail, nextTail);
      if (old === tail) {
        Atomics.store(this.view, tail, value);
        return true;
      }
    }
  }
  
  dequeue() {
    while (true) {
      const head = Atomics.load(this.view, 0);
      const tail = Atomics.load(this.view, 1);
      
      if (head === tail) return null; // Queue empty
      
      const nextHead = head === this.capacity + 1 ? 2 : head + 1;
      const old = Atomics.compareExchange(this.view, 0, head, nextHead);
      
      if (old === head) {
        return Atomics.load(this.view, head);
      }
    }
  }
}

// Parallel merge sort with Atomics
async function parallelMergeSort(array, threadCount = 4) {
  const sab = new SharedArrayBuffer(array.length * 4);
  const shared = new Int32Array(sab);
  shared.set(array);
  
  // Divide and conquer across workers
  const chunkSize = Math.ceil(array.length / threadCount);
  // ... implementation continues with worker management
}

// Memory ordering considerations
// Atomics guarantees sequential consistency by default
// For performance-critical code, weaker orderings possible:
// Atomics.store(view, 0, 42)  // Full barrier
// Atomics.load(view, 0)       // Full barrier
```

### Real-World Use Cases

- Parallel image/video processing (filters, encoding)
- Large dataset processing (ETL pipelines)
- Real-time audio processing (DSP)
- Scientific computing and simulation
- Game engines (physics, AI)
- Cryptographic operations (hashing, encryption)
- Database query processing (parallel aggregation)

### Common Mistakes

- Blocking the main thread with CPU work (use Workers!)
- Creating a worker per request (overhead is too high)
- Forgetting to terminate workers (memory leak)
- Not handling worker errors (crashes and exit codes)
- Race conditions without proper Atomics usage
- Transferring an ArrayBuffer after it's already been transferred
- Using SharedArrayBuffer without Atomics for synchronization

### Best Practices

- Use workers for CPU-intensive tasks (image processing, data transformation)
- Use a worker pool instead of creating workers per task
- Design workers to be long-lived (amortize startup cost)
- Use SharedArrayBuffer + Atomics for high-throughput data sharing
- Always use Atomics for shared memory access (never direct read/write)
- Prefer message passing (structured clone) over shared memory when performance is acceptable
- Handle worker errors and restarts gracefully
- Terminate workers when no longer needed
- Profile before optimizing (message passing may be sufficient)

### Interview Questions

**Q: How does Node.js achieve concurrency with a single thread?** A: Node.js uses an event-driven, non-blocking I/O model. While JavaScript executes on a single thread, I/O operations (network, file system) are delegated to libuv's thread pool or the OS's async facilities. When I/O completes, callbacks are queued on the event loop. This allows handling thousands of concurrent connections without multi-threading overhead.

**Q: When should you use Worker Threads instead of the main thread?** A: Worker threads should be used for CPU-intensive synchronous operations that would block the event loop. Examples: image processing, data transformation, cryptographic operations, JSON parsing of large files, complex calculations. For I/O-bound operations, the existing async APIs are sufficient and more efficient. Workers have startup overhead (~2-5ms) so they should be long-lived, not created per task.


### Performance Considerations
- Worker threads have startup overhead (~5-10ms); reuse workers for small tasks
- SharedArrayBuffer requires cross-origin isolation headers (COOP/COEP)
- Atomics operations are faster than mutex locks but require careful ordering
- PostMessage between threads serializes data via structured clone (slow for large objects)
- Multiple workers improve CPU-bound tasks but don't help I/O-bound tasks


### Coding Challenges
```javascript
// Challenge 1: Create a worker thread pool
const { Worker } = require('worker_threads');
function runWorker(workerData) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./worker.js', { workerData });
    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', code => code !== 0 && reject(new Error(`Exit ${code}`)));
  });
}

// Challenge 2: Use SharedArrayBuffer for counter
const buffer = new SharedArrayBuffer(4);
const counter = new Int32Array(buffer);
Atomics.add(counter, 0, 1);
console.log(Atomics.load(counter, 0)); // 1

// Challenge 3: Atomics-based spin lock
class SpinLock {
  constructor() { this.lock = new Int32Array(new SharedArrayBuffer(4)); }
  acquire() { while (Atomics.compareExchange(this.lock, 0, 0, 1) !== 0); }
  release() { Atomics.store(this.lock, 0, 0); }
}
```

### Related Topics

- Node.js event loop in depth
- libuv thread pool
- Cluster module (multi-process)
- Child processes
- Web Workers (browser equivalent)
- Atomics and SharedArrayBuffer specification
- Lock-free data structures
- Parallel algorithms and patterns
- CPU vs I/O bound operations
- Thread safety and synchronization primitives
