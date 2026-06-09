# Web Workers - Worker constructor, postMessage, onmessage, shared workers, transferable

## Introduction

Web Workers enable JavaScript to run in background threads, parallel to the main execution thread. They handle CPU-intensive tasks without blocking UI interactions. Workers have their own global scope and communicate with the main thread via message passing.

## Worker() Constructor

### What It Is

The Worker constructor creates a new worker thread from a JavaScript file. The worker runs in its own global context (DedicatedWorkerGlobalScope), separate from the main window.

### How It Works Internally

The browser spawns a new operating system thread. The worker has its own event loop, its own heap, and no access to DOM, window, or parent objects. Communication is via message passing.

### Syntax

```javascript
// main.js
const worker = new Worker('worker.js');

// worker.js
self.onmessage = function(e) {
  const result = heavyComputation(e.data);
  self.postMessage(result);
};

// Inline worker (Blob URL)
const blob = new Blob([`
  self.onmessage = function(e) {
    self.postMessage(e.data * 2);
  };
`], { type: 'application/javascript' });
const inlineWorker = new Worker(URL.createObjectURL(blob));
```

### Beginner Examples

```javascript
// main.js
const worker = new Worker('fibonacci-worker.js');

worker.postMessage(40); // Request Fibonacci(40)

worker.onmessage = function(e) {
  console.log('Result:', e.data);
  worker.terminate();
};

worker.onerror = function(e) {
  console.error('Worker error:', e.message);
};

// fibonacci-worker.js
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

self.onmessage = function(e) {
  const result = fibonacci(e.data);
  self.postMessage(result);
};

// Image processing worker
// image-worker.js
self.onmessage = function(e) {
  const { imageData, filter } = e.data;
  const pixels = imageData.data;

  // Apply grayscale filter
  for (let i = 0; i < pixels.length; i += 4) {
    const gray = 0.299 * pixels[i] + 0.587 * pixels[i + 1] + 0.114 * pixels[i + 2];
    pixels[i] = gray;     // R
    pixels[i + 1] = gray; // G
    pixels[i + 2] = gray; // B
  }

  self.postMessage(imageData, [imageData.data.buffer]);
};
```

### Intermediate Examples

```javascript
// Pool of workers for parallel processing
class WorkerPool {
  constructor(workerScript, size = navigator.hardwareConcurrency || 4) {
    this.workers = [];
    this.idle = [];
    this.queue = [];
    this.results = new Map();
    this.idCounter = 0;

    for (let i = 0; i < size; i++) {
      const worker = new Worker(workerScript);
      worker.onmessage = (e) => this.handleResult(worker, e);
      worker.onerror = (e) => this.handleError(worker, e);
      this.workers.push(worker);
      this.idle.push(worker);
    }
  }

  exec(data) {
    return new Promise((resolve, reject) => {
      const id = this.idCounter++;
      this.results.set(id, { resolve, reject });

      if (this.idle.length > 0) {
        const worker = this.idle.pop();
        worker.postMessage({ id, data });
      } else {
        this.queue.push({ id, data });
      }
    });
  }

  handleResult(worker, e) {
    const { id, result, error } = e.data;
    const entry = this.results.get(id);

    if (entry) {
      if (error) entry.reject(new Error(error));
      else entry.resolve(result);
      this.results.delete(id);
    }

    if (this.queue.length > 0) {
      const next = this.queue.shift();
      worker.postMessage(next);
    } else {
      this.idle.push(worker);
    }
  }

  handleError(worker, e) {
    console.error('Worker error:', e.message);
    // Replace failed worker
    const idx = this.workers.indexOf(worker);
    this.workers[idx] = new Worker(worker.src);
    this.workers[idx].onmessage = (e) => this.handleResult(this.workers[idx], e);
  }

  terminate() {
    this.workers.forEach(w => w.terminate());
    this.workers = [];
    this.idle = [];
    this.queue = [];
  }
}

// Usage
const pool = new WorkerPool('compute-worker.js');
const promises = tasks.map(task => pool.exec(task));
const results = await Promise.all(promises);
pool.terminate();
```

### Advanced Examples

```javascript
// Worker with dependency imports (ES Module workers)
// main.js
const worker = new Worker('./worker.js', { type: 'module' });

// worker.js (ES Module worker)
// import { processData } from './utils.js';
//
// self.onmessage = async function(e) {
//   const result = await processData(e.data);
//   self.postMessage(result);
// };

// Worker thread lifecycle management
class ManagedWorker {
  constructor(script, options = {}) {
    this.script = script;
    this.options = options;
    this.worker = null;
    this.restartCount = 0;
    this.maxRestarts = options.maxRestarts || 3;
    this.messageHandlers = new Map();
    this.idCounter = 0;
  }

  start() {
    this.worker = new Worker(this.script, this.options);

    this.worker.onmessage = (e) => {
      const { type, id, data } = e.data;
      if (id && this.messageHandlers.has(id)) {
        const { resolve, reject } = this.messageHandlers.get(id);
        this.messageHandlers.delete(id);
        if (type === 'error') reject(new Error(data));
        else resolve(data);
      }
      if (this.options.onMessage) {
        this.options.onMessage(data);
      }
    };

    this.worker.onerror = (e) => {
      console.error('Worker error:', e.message, e.filename, e.lineno);
      if (this.restartCount < this.maxRestarts) {
        this.restartCount++;
        console.log('Restarting worker... attempt', this.restartCount);
        this.start();
      }
    };

    this.restartCount = 0;
  }

  postMessage(data) {
    return new Promise((resolve, reject) => {
      const id = this.idCounter++;
      this.messageHandlers.set(id, { resolve, reject });

      if (this.worker) {
        this.worker.postMessage({ id, data });
      } else {
        reject(new Error('Worker not started'));
      }
    });
  }

  terminate() {
    if (this.worker) {
      this.worker.terminate();
      this.worker = null;
    }
    this.messageHandlers.clear();
  }
}

// Progressive enhancement: Worker or fallback
function createBackgroundTask(fn) {
  if (window.Worker) {
    // Use Worker
    const blob = new Blob([`
      self.onmessage = async function(e) {
        try {
          const result = (${fn.toString()})(e.data);
          self.postMessage({ success: true, result });
        } catch (err) {
          self.postMessage({ success: false, error: err.message });
        }
      };
    `], { type: 'application/javascript' });

    return (data) => {
      return new Promise((resolve, reject) => {
        const worker = new Worker(URL.createObjectURL(blob));
        worker.onmessage = (e) => {
          if (e.data.success) resolve(e.data.result);
          else reject(new Error(e.data.error));
          worker.terminate();
        };
        worker.onerror = (e) => {
          reject(new Error(e.message));
          worker.terminate();
        };
        worker.postMessage(data);
      });
    };
  } else {
    // Fallback: run on main thread
    return (data) => Promise.resolve(fn(data));
  }
}

// Usage
const heavyTask = createBackgroundTask((data) => {
  let sum = 0;
  for (let i = 0; i < data.iterations; i++) sum += i;
  return sum;
});

const result = await heavyTask({ iterations: 1000000000 });
```

## postMessage() and onmessage

### What It Is

postMessage() sends data to the worker. onmessage receives data from the worker. Data is copied (structured clone algorithm), not shared by default. Transferable objects can be moved without copying.

### Syntax

```javascript
// Main thread to worker
worker.postMessage(data);
worker.postMessage(data, [transferable]); // With transferable objects

// Worker to main thread
self.postMessage(data);
self.postMessage(data, [transferable]);

// Receiving messages
worker.onmessage = function(e) { console.log(e.data); };
worker.addEventListener('message', (e) => console.log(e.data));
```

### Examples

```javascript
// Structured clone: objects, arrays, Maps, Sets, Date, RegExp, Blob, etc.
worker.postMessage({
  user: { name: 'Alice', age: 30 },
  items: [1, 2, 3],
  date: new Date(),
  map: new Map([['key', 'value']])
});

// What CANNOT be sent: Functions, DOM elements, Promises, WeakMaps

// Message batching
self.onmessage = function(e) {
  const results = [];
  for (const item of e.data.items) {
    results.push(processItem(item));
  }
  self.postMessage({ type: 'batch', results });
};

// Request-response pattern
worker.onmessage = function(e) {
  if (e.data.type === 'progress') {
    updateProgressBar(e.data.percent);
  } else if (e.data.type === 'result') {
    showResult(e.data.data);
  } else if (e.data.type === 'error') {
    showError(e.data.message);
  }
};
```

## Shared Workers

### What It Is

SharedWorkers can be accessed by multiple scripts from the same origin. Useful for shared state, cross-tab communication, and centralized resources.

### Syntax

```javascript
// main.js (multiple tabs can connect)
const worker = new SharedWorker('shared-worker.js');
worker.port.start();
worker.port.postMessage({ type: 'connect', tab: getTabId() });
worker.port.onmessage = (e) => {
  console.log('Message:', e.data);
};

// shared-worker.js
const connections = new Map();

self.onconnect = function(e) {
  const port = e.ports[0];
  const id = generateId();
  connections.set(id, port);

  port.onmessage = function(e) {
    const data = e.data;

    switch (data.type) {
      case 'broadcast':
        connections.forEach((p, pid) => {
          if (pid !== id) p.postMessage({ from: id, data: data.payload });
        });
        break;
      case 'getState':
        port.postMessage({ type: 'state', data: currentState });
        break;
      case 'updateState':
        currentState = { ...currentState, ...data.payload };
        break;
    }
  };

  port.start();
  port.postMessage({ type: 'connected', id });
};
```

## Transferable Objects

### What It Is

Transferable objects (ArrayBuffer, MessagePort, ImageBitmap, OffscreenCanvas) can be transferred between threads without copying. Ownership is transferred, making the transfer zero-copy.

### Why It Is Important

Transferring avoids copying large data. A 100MB ArrayBuffer would take ~100ms to copy; transferring takes <1ms. The source loses access after transfer.

### Syntax

```javascript
// main.js
const buffer = new ArrayBuffer(1024 * 1024 * 100); // 100MB

console.log(buffer.byteLength); // 104857600
worker.postMessage({ buffer }, [buffer]); // Transfer ownership
console.log(buffer.byteLength); // 0 - ownership transferred!

// worker.js
self.onmessage = function(e) {
  const buffer = e.data.buffer;
  console.log(buffer.byteLength); // 104857600
  // Process buffer...
  self.postMessage({ result: 'done' });
  // buffer is automatically freed when worker terminates
};

// Multiple transfers
const buffer1 = new ArrayBuffer(1024);
const buffer2 = new ArrayBuffer(2048);
worker.postMessage({ buf1: buffer1, buf2: buffer2 }, [buffer1, buffer2]);

// OffscreenCanvas transfer
const canvas = document.getElementById('canvas');
const offscreen = canvas.transferControlToOffscreen();
worker.postMessage({ canvas: offscreen }, [offscreen]);
```

### Real-World Use Cases

- **Image processing**: Filters, compression, color correction off the main thread.
- **Data parsing**: JSON parsing, CSV parsing, XML parsing of large files.
- **Cryptography**: Encryption/decryption without blocking UI.
- **Game logic**: Physics calculations, pathfinding in background.
- **Code compilation**: Syntax highlighting, bundling, transpilation.
- **Real-time data**: WebSocket message processing in worker.
- **Machine learning**: Model inference in background thread.

### Common Mistakes

```javascript
// Mistake: Trying to access DOM in worker
self.onmessage = function() {
  document.getElementById('output').textContent = 'done'; // Error!
};

// Mistake: Not handling worker errors
const worker = new Worker('worker.js');
worker.postMessage(data);
// No onerror handler - crashes silently!

// Mistake: Creating too many workers
for (let i = 0; i < 100; i++) {
  new Worker('worker.js'); // Each has ~5MB memory overhead
}
// Fix: Use a worker pool

// Mistake: Sending functions to workers
worker.postMessage(() => { /* ... */ }); // Error!

// Mistake: Using closed SharedWorker port
worker.port.close();
worker.port.postMessage(data); // Silently fails
```

### Best Practices

```javascript
// Use a worker pool instead of creating workers per task
// Handle errors with onerror
// Terminate workers when done (pool.terminate())
// Use Transferable objects for large binary data
// Keep worker messages small and frequent (not large and rare)
// Use SharedWorker for cross-tab state
// Prefer ES Module workers for clarity
// Consider Service Workers for network caching

// Performance monitoring
worker.onmessage = function(e) {
  if (e.data.type === 'progress') {
    console.log('Progress:', e.data.percent);
  }
};
```

### Performance Considerations

- Creating a worker takes ~5-10ms and ~5MB memory.
- Message passing has ~10microsecond overhead for small messages.
- Transferable objects: zero-copy for binary data.
- Structured clone: O(n) for large objects.
- Too many workers = thread contention; use CPU core count.
- Workers don't share memory (no race conditions by default).
- Parallel computation scales with available CPU cores.

### Interview Questions

**Q: What can and cannot be sent via postMessage?**
A: Can send: primitives, objects, arrays, Map, Set, Date, RegExp, Blob, ArrayBuffer, ImageBitmap. Cannot send: functions, DOM elements, Promises, WeakMap/WeakSet, Error objects.

**Q: What is the difference between a Worker and a SharedWorker?**
A: A Worker is dedicated to one script context. A SharedWorker can be accessed by multiple scripts (tabs, iframes) from the same origin. SharedWorker uses ports for communication. Workers are for background tasks; SharedWorkers for cross-context communication.

**Q: What are Transferable objects and when would you use them?**
A: Transferable objects (ArrayBuffer, MessagePort, OffscreenCanvas) can be moved between threads without copying. Use them for large binary data (images, audio, video frames, large datasets) to avoid the O(n) cost of structured cloning. The source loses access after transfer.

### Coding Challenges

```javascript
// Challenge 1: Implement a parallel prime number finder
// using a WorkerPool. Distribute ranges across workers
// and collect results.

// Challenge 2: Create an OffscreenCanvas-based image filter
// that runs filters (grayscale, sepia, blur) in a worker.

// Challenge 3: Build a real-time data processing pipeline
// using SharedWorker that multiple tabs connect to.
// The worker processes WebSocket data and broadcasts results.
```

### Related Topics
- Service Workers, Event loop parallelism, Atomics/SharedArrayBuffer
