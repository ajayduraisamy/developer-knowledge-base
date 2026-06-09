# Runtime Internals - Node.js runtime, Deno, Bun, event-driven I/O, libuv

## Introduction

JavaScript runtimes provide the environment in which JavaScript code executes outside the browser. Node.js, Deno, and Bun are the three major server-side JavaScript runtimes. Each implements the JavaScript specification (V8 or JavaScriptCore) but differs in APIs, module systems, security models, and tooling. Understanding runtime internals—especially the event loop and libuv—is crucial for writing performant server-side code.

## Node.js Runtime Architecture

### What It Is

Node.js is an open-source, cross-platform JavaScript runtime built on V8 and libuv. It provides an event-driven, non-blocking I/O model that makes it efficient for building scalable network applications. Node.js's architecture consists of V8 (JavaScript execution), libuv (I/O, event loop, threading), and C++ bindings that bridge JavaScript to system APIs.

### Why It Is Important

Node.js dominates server-side JavaScript development. Understanding its architecture helps developers write efficient code, diagnose performance issues, and work with its asynchronous model effectively.

### How It Works Internally

Node.js layers:
1. **JavaScript layer**: Your code, Node.js core libraries (fs, http, path, etc.)
2. **Node.js C++ bindings**: Bridge between JS and C++ (using N-API/napi)
3. **V8**: JavaScript engine (compilation, GC, memory management)
4. **libuv**: Async I/O, event loop, thread pool, file system, DNS, signals
5. **Other C/C++ dependencies**: OpenSSL (TLS), c-ares (DNS), http-parser, zlib

```
┌─────────────────────────────────────┐
│         JavaScript (user code)      │
├─────────────────────────────────────┤
│     Node.js Core (JS + C++)         │
│  fs, http, crypto, stream, etc.     │
├─────────────────────────────────────┤
│           V8 JavaScript Engine      │
├─────────────────────────────────────┤
│              libuv                   │
│  Event Loop │ Thread Pool │ Async IO │
├─────────────────────────────────────┤
│           Operating System          │
└─────────────────────────────────────┘
```

### Syntax

```javascript
// Node.js internal architecture example
// A simple file read reveals the architecture layers:

const fs = require('fs');
const crypto = require('crypto');

// JavaScript layer: your code calls fs.readFile
fs.readFile('/path/to/file.txt', 'utf8', (err, data) => {
  // This callback runs on the event loop
  console.log(data);
});

// What happens internally:
// 1. JS layer: fs.readFile → binding → C++ layer
// 2. C++ layer: creates a request, passes to libuv
// 3. libuv: submits to thread pool (file I/O uses thread pool)
// 4. Thread pool: performs blocking readFile in background thread
// 5. Completion: libuv posts completion to event loop
// 6. Event loop: calls your callback (JS layer)
```

### Beginner Examples

```javascript
// Understanding Node.js process
const process = require('process');

// Process info
console.log(process.version);      // Node.js version
console.log(process.arch);         // Architecture (x64, arm64)
console.log(process.platform);     // Platform (win32, linux, darwin)
console.log(process.pid);          // Process ID
console.log(process.cwd());        // Current working directory

// Environment variables
console.log(process.env.NODE_ENV);

// Process lifecycle events
process.on('beforeExit', () => console.log('Process about to exit'));
process.on('exit', (code) => console.log(`Exit code: ${code}`));

// Next tick (microtask)
process.nextTick(() => console.log('next tick'));
```

### Intermediate Examples

```javascript
// Event loop phases and setImmediate vs nextTick
const fs = require('fs');

// Order of execution:
fs.readFile(__filename, () => {
  console.log('1. poll phase (I/O callback)');
});

setImmediate(() => {
  console.log('2. check phase (setImmediate)');
});

setTimeout(() => {
  console.log('3. timers phase');
}, 0);

process.nextTick(() => {
  console.log('4. next tick (microtask between phases)');
});

Promise.resolve().then(() => {
  console.log('5. promise microtask');
});

// Output order:
// 4. next tick
// 5. promise microtask
// 3. timers phase
// 1. poll phase (I/O callbacks)
// 2. check phase

// Node.js specific APIs
const { performance } = require('perf_hooks');
const start = performance.now();
setTimeout(() => {
  const end = performance.now();
  console.log(`Timer took ${end - start}ms`);
}, 100);

// Worker threads (for CPU-intensive work)
const { Worker } = require('worker_threads');
const worker = new Worker(`
  const { parentPort } = require('worker_threads');
  parentPort.postMessage(compute());
`);
worker.on('message', result => console.log(result));
```

### Advanced Examples

```javascript
// N-API and native addons
// Creating a native addon (.node file)
// Requires C++ knowledge and node-gyp

// binding.gyp
// {
//   "targets": [{
//     "target_name": "myaddon",
//     "sources": ["src/myaddon.cc"]
//   }]
// }

// myaddon.cc (simplified)
// #include <napi.h>
// Napi::String Method(const Napi::CallbackInfo& info) {
//   return Napi::String::New(info.Env(), "Hello from C++!");
// }
// Napi::Object Init(Napi::Env env, Napi::Object exports) {
//   exports.Set(Napi::String::New(env, "hello"),
//               Napi::Function::New(env, Method));
//   return exports;
// }

// VM module for isolated execution
const vm = require('vm');
const sandbox = { x: 10, console };
vm.createContext(sandbox);
const code = 'x * 5';
const result = vm.runInContext(code, sandbox);
console.log(result); // 50

// V8 heap management
const v8 = require('v8');
console.log(v8.getHeapStatistics());
v8.setFlagsFromString('--max-old-space-size=4096');
```

## Deno Runtime

### What It Is

Deno is a modern JavaScript/TypeScript runtime created by Ryan Dahl (original creator of Node.js). It addresses several design regrets of Node.js: centralized package management (npm), legacy APIs, security model, and TypeScript support. Deno uses V8 (like Node.js) but is written in Rust instead of C++.

### Why It Is Important

Deno represents a modern rethinking of server-side JavaScript. It offers built-in TypeScript support, URL-based imports (no npm required), secure-by-default permissions, web-standard APIs (fetch, WebSocket, EventTarget), and modern tooling (formatter, linter, bundler built in).

### How It Works Internally

Deno's architecture:
1. **V8**: JavaScript engine
2. **Tokio**: Rust async runtime (replaces libuv)
3. **Rust core**: File system, networking, crypto, etc.
4. **TypeScript compiler**: Built-in, fast compilation via SWC
5. **Deno APIs**: Secure wrappers around system calls

Key architectural differences from Node.js:
- Single binary (no npm, no node_modules)
- Permission system (file, network, env access must be explicitly allowed)
- ES modules only (no CommonJS)
- URL-based imports (import from http/https/github URLs)
- Web-standard APIs (uses browser-compatible APIs)

### Syntax

```javascript
// Deno imports (URL-based)
import { serve } from 'https://deno.land/std@0.200.0/http/server.ts';
import { assertEquals } from 'https://deno.land/std@0.200.0/testing/asserts.ts';

// Deno security model
// Run with: deno run --allow-net --allow-read app.ts
// Permissions are granular:
// --allow-net: network access
// --allow-read: file read access
// --allow-write: file write access
// --allow-env: environment variables
// --allow-run: subprocess execution

// Deno namespace
console.log(Deno.version);
console.log(Deno.build);
console.log(Deno.cwd());
console.log(Deno.env.get('HOME'));

// Web-standard APIs (node-compatible)
const response = await fetch('https://api.example.com');
const data = await response.json();

// Deno filesystem
const content = await Deno.readTextFile('./data.json');
await Deno.writeTextFile('./output.txt', 'Hello Deno');
```

### Beginner Examples

```javascript
// Basic Deno HTTP server
// save as server.ts, run: deno run --allow-net server.ts

import { serve } from 'https://deno.land/std@0.200.0/http/server.ts';

const handler = (request) => {
  const url = new URL(request.url);
  
  if (url.pathname === '/hello') {
    return new Response('Hello, World!', {
      headers: { 'content-type': 'text/plain' }
    });
  }
  
  return new Response('Not Found', { status: 404 });
};

serve(handler, { port: 3000 });
console.log('Server running on http://localhost:3000');

// Deno file operations
const decoder = new TextDecoder('utf-8');
const data = await Deno.readFile('./file.txt');
console.log(decoder.decode(data));

// Deno test
import { assertEquals } from 'https://deno.land/std/testing/asserts.ts';

Deno.test('test addition', () => {
  assertEquals(1 + 2, 3);
});
```

## Bun Runtime

### What It Is

Bun is a fast all-in-one JavaScript runtime built from scratch using Apple's JavaScriptCore (Safari's engine) instead of V8. Written in Zig, Bun aims to be a drop-in replacement for Node.js while dramatically improving performance for package installation, script execution, and build tasks.

### Why It Is Important

Bun promises 10x+ performance improvements over Node.js for many operations. Its native bundler, transpiler, and package manager eliminate the need for separate tools (webpack, TypeScript compiler, npm). Bun aims to be the fastest runtime while maintaining Node.js compatibility.

### How It Works Internally

Bun's architecture:
1. **JavaScriptCore**: Apple's JavaScript engine (typically faster startup than V8)
2. **Zig**: Systems programming language (performance and safety)
3. **SQLite**: Built-in database (no separate installation)
4. **Native APIs**: Written in Zig for maximum performance
5. **Node.js compatibility**: Polyfills for Node.js APIs (fs, http, crypto, etc.)

Key features:
- `bun run`: 10x faster than `npm run`
- `bun install`: 10x faster than `npm install`
- `bun test`: Jest-compatible test runner
- `bun build`: Native bundler and transpiler
- Built-in TypeScript/JSX support (no config)
- Node.js API compatibility for most packages

### Syntax

```javascript
// Bun APIs (Node.js compatible + Bun-specific)

// Bun serves files (fast static file serving)
Bun.serve({
  port: 3000,
  async fetch(request) {
    const url = new URL(request.url);
    if (url.pathname === '/') {
      return new Response(Bun.file('./index.html'));
    }
    return new Response('Not Found', { status: 404 });
  }
});

// Bun file I/O (extremely fast)
const file = Bun.file('./data.json');
const json = await file.json();
const text = await file.text();

// Bun write
await Bun.write('./output.txt', 'Hello Bun');

// Bun package.json resolution (fast)
const pkg = Bun.resolveSync('lodash', process.cwd());

// Built-in SQLite
import { Database } from 'bun:sqlite';
const db = new Database('mydb.sqlite');
db.exec('CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)');
const query = db.prepare('INSERT INTO users (name) VALUES ($name)');
query.run({ $name: 'Alice' });
```

### Beginner Examples

```javascript
// Bun HTTP server
// save as server.js, run: bun run server.js

Bun.serve({
  port: 3000,
  fetch(request) {
    return new Response(`Hello from Bun! Requested: ${request.url}`);
  }
});

// Bun file operations
const text = await Bun.file('./package.json').text();
console.log(text);

// Bun shell commands
const output = await Bun.$`echo "hello from shell"`.text();
console.log(output);

// Bun test
import { test, expect } from 'bun:test';

test('2 + 2 = 4', () => {
  expect(2 + 2).toBe(4);
});
```

## libuv and Event-Driven I/O

### What It Is

libuv is a multi-platform C library that provides the event loop, asynchronous I/O, and thread pool for Node.js. It abstracts platform-specific APIs (epoll on Linux, kqueue on macOS, IOCP on Windows) into a unified interface. libuv handles file system operations, networking, DNS, signal handling, and child processes.

### Why It Is Important

libuv is the backbone of Node.js's asynchronous I/O model. It enables non-blocking operations in a single-threaded JavaScript environment by using system-level async APIs and a thread pool for blocking operations. Understanding libuv's event loop phases is essential for predicting code execution order.

### How It Works Internally

libuv's event loop runs in phases, in order:

1. **Timers**: Execute setTimeout/setInterval callbacks whose time has elapsed
2. **Pending callbacks**: Execute I/O callbacks deferred to the next iteration
3. **Idle, prepare**: Internal use (also setImmediate prepares in this phase)
4. **Poll**: Retrieve new I/O events, execute their callbacks, or wait
5. **Check**: Execute setImmediate callbacks
6. **Close callbacks**: Execute close event callbacks (socket.on('close'))

Between each phase, libuv processes:
- `process.nextTick` queue (entirely drained)
- Promise microtask queue (entirely drained)

```javascript
// Event loop phases (simplified)
while (isAlive()) {
  // 1. Timers: check for expired setTimeout/setInterval
  // 2. Pending callbacks: process completed I/O
  // 3. Idle/Prepare: internal preparation
  
  // 4. Poll: I/O polling + callbacks
  //    (blocks here if nothing pending)
  
  // 5. Check: setImmediate callbacks
  // 6. Close: cleanup callbacks
  
  // Between phases:
  // - process.nextTick() queue (drained entirely)
  // - Promise microtasks (drained entirely)
}
```

### Syntax

```javascript
// Event loop phase demonstrations

// Timer phase
setTimeout(() => console.log('timer'), 0);

// Check phase
setImmediate(() => console.log('check'));

// I/O callback (poll phase)
const fs = require('fs');
fs.readFile(__filename, () => {
  console.log('I/O callback (poll)');
  
  // Nesting inside poll shows order clearly
  setTimeout(() => console.log('timer inside poll'), 0);
  setImmediate(() => console.log('check inside poll'));
  process.nextTick(() => console.log('nextTick inside poll'));
});

// Output:
// I/O callback (poll)
// nextTick inside poll
// check inside poll
// timer inside poll
```

### Beginner Examples

```javascript
// libuv thread pool
const crypto = require('crypto');

// crypto.pbkdf2 is CPU-intensive and runs on libuv thread pool
const start = Date.now();
crypto.pbkdf2('password', 'salt', 100000, 64, 'sha512', (err, key) => {
  console.log(`1: ${Date.now() - start}ms`);
});

// Default thread pool size is 4
// Increase with: process.env.UV_THREADPOOL_SIZE = 8

// libuv blocking vs non-blocking
// Non-blocking: fs.readFile (uses thread pool or async system calls)
// Blocking: fs.readFileSync (blocking system call, blocks event loop)

// Never use sync versions in production servers
const data = fs.readFile('/large-file.txt', (err, data) => {
  // Non-blocking: event loop continues during I/O
  processData(data);
});

// Blocking version - stops everything
// const data = fs.readFileSync('/large-file.txt'); // BAD!
```

### Intermediate Examples

```javascript
// Event loop starvation
function starveEventLoop() {
  // Synchronous CPU-bound operation blocks event loop
  // setTimeout, I/O, Promise callbacks cannot run
  while (Date.now() % 1000000 !== 0) {}
  console.log('Finally done blocking');
}

// To avoid starvation, use async scheduling
async function safeOperation() {
  const chunk = (arr) => {
    const size = 10;
    for (let i = 0; i < arr.length; i += size) {
      processChunk(arr.slice(i, i + size));
      // Yield to event loop every chunk
      await new Promise(r => setImmediate(r));
    }
  };
}

// Understanding thread pool vs event loop
// Network I/O (TCP, UDP) uses kernel async facilities (epoll/kqueue)
// → No thread pool used → Very efficient

// File I/O and DNS use thread pool
// → Can block thread pool threads → Pool exhaustion possible

// Custom async operations with libuv
// Node.js addons can schedule work on libuv thread pool
// using napi_async_work APIs
```

### Advanced Examples

```javascript
// Observing libuv behavior
const uv = process.binding('uv'); // Internal API

// libuv handle types
console.log('UV handles:', Object.keys(uv));

// Monitor active handles
function printActiveHandles() {
  const handles = process._getActiveHandles();
  const requests = process._getActiveRequests();
  console.log(`Active handles: ${handles.length}, Requests: ${requests.length}`);
}

setInterval(printActiveHandles, 1000);

// libuv resource limits
// The poll phase has a timeout (max time waiting for I/O)
// Controlled by the event loop based on nearest timer

// Understanding uv_run
// uv_run(UV_RUN_DEFAULT): runs loop until no more handles
// uv_run(UV_RUN_ONCE): runs one iteration (blocks if necessary)
// uv_run(UV_RUN_NOWAIT): runs one iteration without blocking

// Thread pool sizing
// Each thread pool operation takes a slot
// If 4 file reads are running, 5th waits
// Adjust with UV_THREADPOOL_SIZE
process.env.UV_THREADPOOL_SIZE = '8';
// Higher value: more concurrent I/O but more thread overhead

// Custom events through libuv
// Use async_hooks to observe async resource lifecycle
const async_hooks = require('async_hooks');

const hook = async_hooks.createHook({
  init(asyncId, type, triggerAsyncId) {
    console.log(`Init: ${type}[${asyncId}] triggered by ${triggerAsyncId}`);
  },
  destroy(asyncId) {
    console.log(`Destroy: ${asyncId}`);
  }
});
hook.enable();
```

### Real-World Use Cases

- Building high-performance HTTP servers and APIs
- Real-time applications (chat, gaming, collaboration)
- Streaming services (video, audio)
- Command-line tools and build systems
- IoT backends with many concurrent connections
- Serverless functions

### Common Mistakes

- Blocking the event loop with CPU-intensive synchronous operations
- Setting UV_THREADPOOL_SIZE too high (causes thread contention)
- Not considering thread pool exhaustion for file I/O
- Misunderstanding event loop phase ordering
- Using synchronous APIs in production servers

### Best Practices

- Always use async APIs (fs.promises, etc.) for I/O
- Offload CPU-intensive work to Worker Threads
- Use streams for large data processing
- Monitor event loop lag (blocked time)
- Set max listeners for EventEmitter (avoid memory leaks)
- Use `setImmediate()` to break up synchronous work
- Keep `process.nextTick()` usage minimal (can starve I/O)
- Profile with `clinic.js` or `0x` for flame graphs

### Interview Questions

**Q: What is libuv and what role does it play in Node.js?** A: libuv is a C library that provides the event loop and asynchronous I/O for Node.js. It abstracts platform-specific async operations (epoll, kqueue, IOCP) into a unified API. It manages the event loop phases (timers, I/O callbacks, poll, check, close), the thread pool for blocking operations (file I/O, DNS, crypto), and handles signals, timers, and child processes.

**Q: How does Node.js handle asynchronous I/O without blocking?** A: For network I/O, Node.js uses the operating system's native async facilities (epoll on Linux, kqueue on macOS, IOCP on Windows) through libuv. These allow a single thread to monitor thousands of sockets simultaneously. For file I/O and DNS, libuv uses a thread pool (default 4 threads) so the main JavaScript thread isn't blocked. When operations complete, callbacks are queued on the event loop.


### Performance Considerations
- libuv's thread pool has default size of 4; increase with UV_THREADPOOL_SIZE for I/O-heavy workloads
- Event-driven I/O outperforms thread-per-connection for concurrent connections
- Deno's permission system adds security overhead on first access
- Bun's JavaScriptCore engine startup is faster than V8 for short-lived processes
- Node.js module loading is synchronous and blocks event loop startup


### Coding Challenges
```javascript
// Challenge 1: Measure event loop lag
function measureEventLoopLag() {
  const start = Date.now();
  setTimeout(() => {
    const lag = Date.now() - start - 100;
    console.log(`Event loop lag: ${lag}ms`);
  }, 100);
}
measureEventLoopLag();

// Challenge 2: Check runtime environment
function detectRuntime() {
  if (typeof process !== 'undefined' && process.release?.name === 'node') return 'Node.js';
  if (typeof Deno !== 'undefined') return 'Deno';
  if (typeof Bun !== 'undefined') return 'Bun';
  return 'Browser';
}
console.log(`Running on: ${detectRuntime()}`);

// Challenge 3: Set libuv thread pool size
process.env.UV_THREADPOOL_SIZE = '8';
const crypto = require('crypto');
const start = Date.now();
Promise.all(Array(8).fill().map(() => 
  new Promise(resolve => crypto.pbkdf2('pass', 'salt', 100000, 64, 'sha512', resolve))
)).then(() => console.log(`Done in ${Date.now() - start}ms`));
```

### Related Topics

- Event loop in detail (phases, microtasks, macrotasks)
- Worker threads for CPU-intensive work
- libuv API and configuration
- V8 engine architecture
- Node.js vs Deno vs Bun comparison
- Cluster module for multi-process
- Async hooks and diagnostics
- Performance monitoring and profiling
- Streams and backpressure
- Buffer and TypedArray internals
