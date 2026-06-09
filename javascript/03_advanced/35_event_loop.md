# Event Loop

## Introduction

The event loop is the core mechanism that enables JavaScript's non-blocking, asynchronous behavior despite being single-threaded. It continuously checks the call stack and task queues, moving callbacks from queues to the stack when the stack is empty. Understanding the event loop is essential for debugging async code, avoiding race conditions, and optimizing performance.

---

## Call stack

### What It Is

The call stack is a LIFO (Last In, First Out) data structure that tracks function execution. Every time a function is called, a new frame is pushed onto the stack containing the function's arguments, local variables, and return address. When the function returns, its frame is popped off.

```javascript
function first() {
  second();
  console.log("first");
}
function second() {
  third();
  console.log("second");
}
function third() {
  console.log("third");
}
first();
// Stack: [global] -> [first] -> [second] -> [third]
// After third returns: [global] -> [first] -> [second]
// Output: "third", "second", "first"
```

### Why It Is Important

The call stack determines execution order and is the foundation of the event loop:
- JavaScript can only do one thing at a time — the stack must be empty for callbacks to run.
- Stack overflow happens when the stack exceeds its limit (typically ~10,000 frames).
- Recursive functions without base cases cause stack overflow.
- Understanding the stack helps debug errors via stack traces.
- The stack must unwind before async callbacks can execute.

### How It Works Internally

The JavaScript engine (V8, SpiderMonkey) maintains a stack of execution contexts. Each context contains:
- The function being executed
- The `this` binding
- Scope chain (lexical environment)
- Arguments and local variables

When a function calls another function, a new context is pushed. On `return`, it's popped. The stack has a fixed maximum size (e.g., ~984 frames in V8). Exceeding it throws `RangeError: Maximum call stack size exceeded`.

### Syntax

There is no syntax for manipulating the call stack — it's an implicit mechanism:

```javascript
// Function calls push frames
function a() {
  b();
}
function b() {
  c();
}
function c() {
  debugger; // Inspect stack in DevTools
}
a();

// Recursion consumes stack frames
function recurse(depth) {
  if (depth === 0) return;
  recurse(depth - 1);
}
recurse(10000); // Stack overflow
```

### Beginner Examples

```javascript
// Example 1: Visualizing the stack
function greet(name) {
  return `Hello, ${name}!`;
}

function createGreeting() {
  const message = greet("Alice");
  console.log(message);
}

createGreeting();
// Stack: [global] -> [createGreeting] -> [greet]
// greet returns -> stack: [global] -> [createGreeting]
// createGreeting returns -> stack: [global]

// Example 2: Stack overflow
function infinite() {
  return infinite();
}
// infinite(); // Uncaught RangeError: Maximum call stack size exceeded

// Example 3: Stack trace
function level3() { throw new Error("Oops"); }
function level2() { level3(); }
function level1() { level2(); }

try {
  level1();
} catch (err) {
  console.log(err.stack);
  // Shows: level3 -> level2 -> level1 -> (anonymous)
}
```

### Intermediate Examples

```javascript
// Example 1: Async callbacks and stack
function syncFunction() {
  console.log("sync");
}

function asyncFunction() {
  setTimeout(() => console.log("async"), 0);
}

console.log("start");
syncFunction();
asyncFunction();
console.log("end");
// Stack at each point:
// 1. [global] - logs "start"
// 2. [global, syncFunction] - logs "sync"
// 3. [global] - setTimeout registers callback
// 4. [global] - logs "end"
// 5. [] - stack empty
// 6. Callback executes - logs "async"
// Output: "start", "sync", "end", "async"

// Example 2: Stack in recursive async
async function asyncRecurse(n) {
  if (n === 0) return;
  console.log(`Frame: ${n}`);
  await Promise.resolve(); // yields to event loop
  return asyncRecurse(n - 1);
}
// Each await unwinds the stack
```

### Advanced Examples

```javascript
// Example 1: Stack inspection
function getStack() {
  const err = new Error();
  return err.stack
    .split("\n")
    .slice(2) // skip the Error message and getStack line
    .map((line) => line.trim());
}

// Example 2: Safe recursion with trampolining
function trampoline(fn) {
  return function (...args) {
    let result = fn(...args);
    while (typeof result === "function") {
      result = result();
    }
    return result;
  };
}

const safeRecurse = trampoline(function recurse(n, acc = 0) {
  if (n === 0) return acc;
  return () => recurse(n - 1, acc + n);
});

console.log(safeRecurse(100000)); // 5000050000 (no stack overflow)

// Example 3: Async stack traces (async/await)
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

async function loadDashboard() {
  const user = await fetchUser(1);
  const posts = await fetchUserPosts(user.id);
  return { user, posts };
}
// Stack traces in async functions show the full async chain
```

### Real-World Use Cases

- **Debugging** — Reading stack traces to find the source of errors.
- **Profiling** — Analyzing call stack samples in DevTools Performance tab.
- **Trampolining** — Implementing safe recursion for deep traversals.
- **Async stack management** — Understanding why certain code runs before/after others.

### Common Mistakes

- **Assuming async callbacks run on the same stack** — They run on a fresh stack.
- **Stack overflow from deep recursion** — Always have a base case.
- **Blocking the stack** — CPU-intensive operations freeze the UI.
- **Confusing stack frames with event loop iterations** — Different concepts.

### Best Practices

- Avoid deep recursion (over ~1000 frames) — use iteration or trampolining.
- Offload heavy computation to Web Workers.
- Use async/await for better stack traces than raw promises.
- Read stack traces bottom-to-top to find the origin of errors.
- Use `console.trace()` to log current stack during debugging.

### Performance Considerations

- Each function call adds stack overhead — millions of calls add up.
- Inline functions where performance is critical (V8 inlines automatically).
- Recursion is slower than iteration in most engines.
- Async stack traces are more expensive than sync — `--async-stack-traces` flag in Node.js.
- Stack overflow is a hard crash — always guard against infinite recursion.

### Interview Questions

1. **What is the call stack?**
   A LIFO data structure that tracks function execution contexts in the JavaScript engine.

2. **What happens when the call stack overflows?**
   JavaScript throws a `RangeError: Maximum call stack size exceeded`.

3. **What is the relationship between the call stack and the event loop?**
   The event loop can only process callbacks when the call stack is empty.

4. **How does `await` affect the call stack?**
   `await` unwinds the current stack and saves the continuation. When the promise settles, a new stack is created to resume execution.

### Coding Challenges

1. **Implement a function that measures the maximum stack depth.**
2. **Create a trampoline utility for deep recursion.**
3. **Write a function that pretty-prints the current call stack.**
4. **Implement a safe factorial using trampolining.**

### Related Topics

- Execution context
- Event loop
- Task queue
- Microtask queue
- Recursion

---

## Task queue (macrotasks)

### What It Is

The task queue (also called macrotask queue or callback queue) holds callbacks from asynchronous operations like `setTimeout`, `setInterval`, `setImmediate` (Node.js), and I/O events. The event loop picks tasks from this queue only when the call stack is empty, processing them one per iteration.

```javascript
console.log("start");

setTimeout(() => {
  console.log("timeout");
}, 0);

console.log("end");
// Output: "start", "end", "timeout"
```

### Why It Is Important

The task queue is what enables non-blocking asynchronous behavior:
- It defers callback execution until the stack is clear.
- It ensures that long-running synchronous code doesn't starve callbacks indefinitely.
- It provides a scheduling mechanism with minimum delays (not guaranteed times).
- It forms one part of the event loop's priority system.

### How It Works Internally

When `setTimeout(callback, delay)` is called, the Web API (browser) or libuv (Node.js) starts a timer. When the timer expires, the callback is placed into the task queue. The event loop's `tick` does:

1. Check if the call stack is empty.
2. If empty, take the first task from the task queue.
3. Push it onto the call stack.
4. Execute until the stack is empty again.
5. Process microtasks (see next section).
6. Repeat.

```javascript
// Internal flow:
// 1. setTimeout(fn, 0) - Web API starts timer (0ms)
// 2. Timer expires immediately
// 3. fn is queued to task queue
// 4. Event loop checks stack: busy? wait
// 5. Stack empty? Take fn from task queue
// 6. Push fn onto stack, execute
```

### Syntax

```javascript
// setTimeout - schedules a macrotask
setTimeout(() => {
  console.log("runs once after delay");
}, 1000);

// setInterval - schedules repeated macrotasks
const id = setInterval(() => {
  console.log("runs every interval");
}, 1000);
clearInterval(id);

// setImmediate (Node.js) - macrotask after I/O
setImmediate(() => {
  console.log("after I/O callbacks");
});

// I/O events (Node.js)
fs.readFile("file.txt", () => {
  console.log("I/O callback as macrotask");
});
```

### Beginner Examples

```javascript
// Example 1: Basic macrotask scheduling
console.log("A");
setTimeout(() => console.log("B"), 0);
console.log("C");
// Output: A, C, B

// Example 2: Multiple timeouts
setTimeout(() => console.log("first"), 100);
setTimeout(() => console.log("second"), 50);
setTimeout(() => console.log("third"), 0);
// Output: third (0ms), second (50ms), first (100ms)

// Example 3: setInterval macrotasks
let count = 0;
const interval = setInterval(() => {
  console.log(`Tick ${++count}`);
  if (count >= 3) clearInterval(interval);
}, 500);
// Output: Tick 1 (500ms), Tick 2 (1000ms), Tick 3 (1500ms)
```

### Intermediate Examples

```javascript
// Example 1: Macrotask starvation demonstration
const start = Date.now();
setTimeout(() => {
  console.log(`Timeout ran after ${Date.now() - start}ms`);
}, 10);

// Block the stack for 100ms
while (Date.now() - start < 100) {
  // busy wait
}
// The timeout callback runs after ~100ms, not 10ms
// because the stack was blocked

// Example 2: Task queue ordering
function scheduleTasks() {
  setTimeout(() => console.log("Task 1"), 0);
  setTimeout(() => console.log("Task 2"), 0);
  setTimeout(() => console.log("Task 3"), 0);
}
scheduleTasks();
// Output: Task 1, Task 2, Task 3 (FIFO order)

// Example 3: Macrotask in Node.js (setImmediate vs setTimeout)
// In Node.js event loop phases:
// timers -> I/O -> idle/prepare -> poll -> check -> close
// setImmediate callbacks run in the "check" phase
// setTimeout callbacks run in the "timers" phase

setTimeout(() => console.log("timeout"), 0);
setImmediate(() => console.log("immediate"));
// Order depends on phase of event loop at start
```

### Advanced Examples

```javascript
// Example 1: Macrotask scheduling for UI rendering
// Browser renders between macrotask executions
function animate() {
  let pos = 0;
  function step() {
    pos += 1;
    element.style.transform = `translateX(${pos}px)`;
    if (pos < 100) {
      setTimeout(step, 16); // ~60fps
    }
  }
  setTimeout(step, 0);
}

// Example 2: Cooperative multitasking with macrotasks
function processLargeArray(array) {
  let index = 0;
  function chunk() {
    const start = Date.now();
    while (index < array.length && Date.now() - start < 50) {
      processItem(array[index]);
      index++;
    }
    if (index < array.length) {
      setTimeout(chunk, 0); // yield to event loop
    } else {
      console.log("Processing complete");
    }
  }
  chunk();
}
// Prevents blocking the UI for large arrays

// Example 3: MessageChannel for macrotask scheduling
function scheduleMacrotask(callback) {
  const channel = new MessageChannel();
  channel.port1.onmessage = callback;
  channel.port2.postMessage(null);
}
// More reliable than setTimeout(fn, 0) for macrotask scheduling
```

### Real-World Use Cases

- **Animations** — `setTimeout`/`setInterval` for frame scheduling.
- **Loading indicators** — Show spinner before heavy computation.
- **Chunked processing** — Break large tasks into smaller chunks.
- **Debouncing** — `setTimeout` to delay execution.
- **Polling** — `setInterval` for periodic server checks.

### Common Mistakes

- **Assuming setTimeout(fn, 0) runs immediately** — It runs after the stack empties.
- **Using setInterval for precise timing** — Delays accumulate if callbacks take too long.
- **Blocking the task queue** — Long-running synchronous code delays all pending tasks.
- **Forgetting to clear intervals/timeouts** — Memory leaks and unwanted callbacks.
- **Relying on setTimeout precision** — Minimum delay is 4ms for nested timeouts (HTML spec).

### Best Practices

- Use `setTimeout(fn, 0)` to defer work until the stack is clear.
- Always clear intervals with `clearInterval` when no longer needed.
- Prefer `requestAnimationFrame` for visual updates.
- Use `setImmediate` (Node.js) over `setTimeout(fn, 0)` for better performance.
- Chunk heavy synchronous work with `setTimeout` to keep the UI responsive.

### Performance Considerations

- `setTimeout(fn, 0)` has a minimum delay of 4ms for nested calls (≥ 5th level).
- Task queue processing happens once per event loop iteration — too many tasks can starve rendering.
- The task queue is FIFO — fairness is maintained.
- Macrotask scheduling has overhead — creating thousands of timers is expensive.
- `setImmediate` is faster than `setTimeout(fn, 0)` in Node.js (no timer overhead).

### Interview Questions

1. **What is the task queue?**
   A FIFO queue holding callbacks from async operations (timers, I/O). The event loop processes them when the call stack is empty.

2. **What is the difference between a task and a microtask?**
   Tasks come from the task queue (timers, I/O). Microtasks come from promises, mutation observers, and `queueMicrotask`. Microtasks have higher priority.

3. **Does setTimeout(fn, 0) run immediately after the current function?**
   No. It runs after all currently queued microtasks and when the call stack is next empty.

4. **How many tasks are processed per event loop iteration?**
   One task per iteration. After processing a task, the event loop processes all microtasks before the next iteration.

### Coding Challenges

1. **Implement a cooperative scheduler using setTimeout.**
2. **Create a `scheduleAfterStackClears` utility.**
3. **Build a polling function using setTimeout recursion.**
4. **Write a function that measures the minimum setTimeout delay.**

### Related Topics

- Event loop
- Microtask queue
- `setTimeout` / `setInterval`
- `setImmediate`
- `requestAnimationFrame`

---

## Microtask queue

### What It Is

The microtask queue (also called the job queue) holds callbacks from Promise reactions (`.then()`, `.catch()`, `.finally()`), `queueMicrotask()`, and `MutationObserver`. Microtasks are processed after each macrotask, and the microtask queue is drained completely before the next macrotask is picked up.

```javascript
console.log("start");

setTimeout(() => console.log("timeout"), 0);

Promise.resolve().then(() => console.log("microtask"));

console.log("end");
// Output: "start", "end", "microtask", "timeout"
```

### Why It Is Important

Microtasks have higher priority than macrotasks:
- Promises resolve asynchronously but before timers.
- They enable "as soon as possible, but not synchronously" semantics.
- They are essential for the correct operation of Promise-based code.
- They prevent microtask-based callbacks from being starved by macrotasks.
- They ensure DOM updates are batched and processed efficiently.

### How It Works Internally

The event loop processes microtasks at specific checkpoints:
1. After each macrotask completes.
2. After each callback from the task queue.
3. After each script execution completes.

The microtask queue is processed until it is empty. If microtasks queue additional microtasks, those are also processed before yielding to macrotasks. This means microtasks can starve the task queue if they continually enqueue more microtasks.

```javascript
// Internal flow:
// 1. Execute macrotask (e.g., script tag)
// 2. Empty the microtask queue (process all queued microtasks)
// 3. Render (if needed)
// 4. Pick next macrotask
// 5. Repeat
```

### Syntax

```javascript
// Promise reactions are microtasks
Promise.resolve().then(() => console.log("microtask"));
Promise.reject().catch(() => console.log("microtask"));

// queueMicrotask API
queueMicrotask(() => {
  console.log("explicit microtask");
});

// MutationObserver callbacks are microtasks
const observer = new MutationObserver(() => console.log("mutation"));
observer.observe(element, { childList: true });
```

### Beginner Examples

```javascript
// Example 1: Microtask vs macrotask order
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
// Output: 1, 4, 3, 2

// Example 2: Promise microtask
function demo() {
  console.log("A");
  new Promise((resolve) => {
    console.log("B"); // executor runs synchronously
    resolve();
  }).then(() => console.log("C"));
  console.log("D");
}
demo();
// Output: A, B, D, C

// Example 3: queueMicrotask
console.log("first");
queueMicrotask(() => console.log("second"));
console.log("third");
// Output: first, third, second
```

### Intermediate Examples

```javascript
// Example 1: Microtask queue is drained completely
Promise.resolve()
  .then(() => {
    console.log("microtask 1");
    // Queue another microtask
    queueMicrotask(() => console.log("microtask 2"));
  })
  .then(() => {
    console.log("microtask 3");
  });

setTimeout(() => console.log("macrotask"), 0);
// Output: microtask 1, microtask 3, microtask 2, macrotask
// Note: microtask 3 comes from the chain, microtask 2 was queued inside

// Example 2: Microtask starvation of macrotasks
function microtaskFlood() {
  queueMicrotask(() => {
    console.log("microtask");
    microtaskFlood(); // never lets macrotasks run
  });
}
// microtaskFlood(); // DON'T RUN - starves the event loop

// Example 3: Microtasks and rendering
button.addEventListener("click", () => {
  Promise.resolve().then(() => {
    // This runs BEFORE the browser renders
    console.log("microtask before render");
  });
  setTimeout(() => {
    // This runs AFTER the browser renders
    console.log("macrotask after render");
  }, 0);
});
```

### Advanced Examples

```javascript
// Example 1: Microtask queue flushing
function flushMicrotasks() {
  return new Promise((resolve) => resolve());
}
// await flushMicrotasks() - yields control until microtasks are processed

// Example 2: Check if currently in microtask
let inMicrotask = false;
queueMicrotask(() => {
  inMicrotask = true;
  Promise.resolve().then(() => {
    console.log("Still in microtask drain:", true);
  });
});

// Example 3: Microtask-based batching
class Batcher {
  constructor() {
    this.queue = [];
    this.scheduled = false;
  }

  add(item) {
    this.queue.push(item);
    if (!this.scheduled) {
      this.scheduled = true;
      queueMicrotask(() => this.flush());
    }
  }

  flush() {
    const batch = this.queue.splice(0);
    this.scheduled = false;
    batch.forEach((item) => process(item));
  }
}

const batcher = new Batcher();
batcher.add("a");
batcher.add("b");
batcher.add("c");
// All three are processed in a single microtask flush
```

### Real-World Use Cases

- **Vue.js reactivity** — Uses microtasks for batched DOM updates.
- **React state batching** — `setState` calls are batched using microtasks.
- **Promise libraries** — All `.then()` callbacks use microtasks.
- **MutationObserver** — DOM change callbacks are microtasks.
- **Database transactions** — Promise-based transaction completion.

### Common Mistakes

- **Assuming microtasks run "instantly"** — They run after the current script, not during.
- **Microtask starvation** — Infinite microtask recursion blocks macrotasks and rendering.
- **Confusing microtasks with macrotasks** — Different priority levels.
- **Not understanding Promise execution order** — The executor is synchronous, `.then()` is microtask.
- **queueMicrotask vs setTimeout(fn, 0)** — Microtask runs before any pending macrotask.

### Best Practices

- Use `queueMicrotask` for "deferred but ASAP" operations.
- Don't rely on microtask timing for user-visible interactions (use `requestAnimationFrame`).
- Be aware that microtasks block rendering — keep them short.
- Use Promise-based APIs — they naturally use microtasks.
- Avoid recursive microtask scheduling that could starve macrotasks.

### Performance Considerations

- Microtask queue processing is synchronous within the event loop — it blocks further processing.
- Deep microtask queues can delay rendering and user interactions.
- Each microtask is a function call — thousands are fine, millions are not.
- Microtask creation has similar overhead to Promise creation.
- The engine may limit microtask recursion depth to prevent starvation.

### Interview Questions

1. **What is a microtask?**
   A callback scheduled via Promise reactions, `queueMicrotask`, or `MutationObserver`. Microtasks are processed after each macrotask, before the next macrotask.

2. **What is the difference between a microtask and a macrotask?**
   Macrotasks come from timers and I/O. Microtasks come from promises and `queueMicrotask`. Microtasks have higher priority and are drained completely before the next macrotask.

3. **How does the event loop handle the microtask queue?**
   After processing a macrotask, the event loop processes all microtasks (including those added during microtask processing) before moving to the next macrotask.

4. **Can microtasks starve macrotasks?**
   Yes. If microtasks continuously queue more microtasks, the macrotask queue never gets processed.

### Coding Challenges

1. **Implement a simple event loop with microtask processing.**
2. **Create a utility that measures how many microtasks can be queued per frame.**
3. **Build a batcher that uses `queueMicrotask` for coalescing updates.**
4. **Write a function that demonstrates microtask starvation.**
5. **Implement `Promise.resolve().then()` using only `queueMicrotask`.**

### Related Topics

- Event loop
- Task queue
- Macrotasks
- Promises
- `queueMicrotask`
- `MutationObserver`

---

## Event loop phases

### What It Is

The event loop is not a single monolithic concept — it operates in phases. In the browser, the event loop processes macrotasks, microtasks, and rendering in a specific order. In Node.js, the event loop has more granular phases: timers, I/O callbacks, idle/prepare, poll, check (setImmediate), and close callbacks.

```javascript
// Browser event loop phases (simplified):
// 1. Execute a macrotask (from task queue)
// 2. Process all microtasks (drain microtask queue)
// 3. Perform rendering (if needed)
// 4. Repeat

// Node.js event loop phases:
// 1. Timers (setTimeout, setInterval)
// 2. I/O callbacks
// 3. Idle/Prepare (internal)
// 4. Poll (I/O polling)
// 5. Check (setImmediate)
// 6. Close callbacks
```

### Why It Is Important

Understanding the event loop phases is crucial for:
- Knowing exactly when your code will execute relative to other operations.
- Debugging timing-dependent issues.
- Optimizing performance (when to use `setTimeout` vs `setImmediate` vs `process.nextTick`).
- Understanding Promise/timer/rendering ordering.
- Writing predictable asynchronous code.

### How It Works Internally

**Browser Event Loop:**
1. Pick the oldest macrotask from the task queue.
2. Execute it (run the callback).
3. Check the microtask queue — process all microtasks until empty.
4. Check if rendering is needed (update animations, styles, layout, paint).
5. If there's a `requestAnimationFrame` callback, run it before rendering.
6. Repeat from step 1.

**Node.js Event Loop (libuv):**
The event loop iterates through phases. Each phase has its own queue. Between phases, microtasks and `process.nextTick` callbacks are processed.

### Syntax

```javascript
// Browser - observe event loop order
console.log("script start");

setTimeout(() => console.log("macrotask - timeout"), 0);

Promise.resolve().then(() => console.log("microtask - promise"));

requestAnimationFrame(() => console.log("before render"));

console.log("script end");
// Typical output:
// script start
// script end
// microtask - promise
// before render
// macrotask - timeout

// Node.js - observe phases
setTimeout(() => console.log("timers phase"), 0);
setImmediate(() => console.log("check phase"));

// In main module, the order depends on event loop state
// Usually: timers phase (setTimeout), then check phase (setImmediate)
// But inside I/O callback, setImmediate always runs first
```

### Beginner Examples

```javascript
// Example 1: Browser event loop order
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
// Stack: script runs -> microtask queue drains -> macrotask queue
// Output: 1, 4, 3, 2

// Example 2: Node.js timer vs I/O
const fs = require("fs");

// In the main module:
setTimeout(() => console.log("timeout"), 0);
setImmediate(() => console.log("immediate"));
// Output varies: both can be first

// But inside an I/O callback:
fs.readFile(__filename, () => {
  setTimeout(() => console.log("timeout"), 0);
  setImmediate(() => console.log("immediate"));
});
// Output: immediate, timeout (always)
// Because I/O phase -> check phase -> timers phase (next loop)

// Example 3: process.nextTick (Node.js)
// nextTick callbacks run between phases, before microtasks
console.log("start");
process.nextTick(() => console.log("nextTick"));
Promise.resolve().then(() => console.log("promise"));
console.log("end");
// Output: start, end, nextTick, promise
```

### Intermediate Examples

```javascript
// Example 1: Full event loop interaction
console.log("1 - sync");

setTimeout(() => {
  console.log("2 - timeout");
  Promise.resolve().then(() => console.log("3 - timeout microtask"));
}, 0);

Promise.resolve().then(() => {
  console.log("4 - microtask");
  setTimeout(() => console.log("5 - microtask timeout"), 0);
});

setTimeout(() => console.log("6 - second timeout"), 0);

console.log("7 - sync");
// Output: 1, 7, 4, 2, 3, 6, 5 (browser)

// Let's trace:
// Script: logs 1, 7. Schedules timeout1, microtask1, timeout2
// Microtask queue: [microtask1] -> logs 4, schedules timeout3
// Microtask queue empty
// Macrotask queue: [timeout1, timeout2, timeout3]
// timeout1: logs 2, schedules microtask2
// Microtask queue: [microtask2] -> logs 3
// Macrotask: timeout2: logs 6
// Microtask queue empty
// Macrotask: timeout3: logs 5

// Example 2: requestAnimationFrame timing
// rAF fires before style/layout/paint, after microtasks
button.addEventListener("click", () => {
  requestAnimationFrame(() => {
    console.log("before render");
  });
  Promise.resolve().then(() => console.log("microtask"));
  setTimeout(() => console.log("macrotask"), 0);
});
// Click handler: runs synchronously, schedules: rAF, microtask, timeout
// After handler: microtask runs ("microtask")
// Before render: rAF runs ("before render")
// Render happens
// Next macrotask: timeout runs ("macrotask")
```

### Advanced Examples

```javascript
// Example 1: Node.js phase order demonstration
const fs = require("fs");

// Create a file to read
fs.writeFileSync("test.txt", "hello");

// Track execution order
const order = [];

order.push("1 - start");

setTimeout(() => order.push("2 - timer"), 0);
setImmediate(() => order.push("3 - immediate"));

fs.readFile("test.txt", () => {
  order.push("4 - I/O callback");
  
  setTimeout(() => order.push("5 - timer in I/O"), 0);
  setImmediate(() => order.push("6 - immediate in I/O"));
  process.nextTick(() => order.push("7 - nextTick in I/O"));
});

process.nextTick(() => order.push("8 - nextTick"));

order.push("9 - end");

process.on("exit", () => {
  console.log("Order:", order);
});
// Typical output: 1, 9, 8, 2, 3, 4, 7, 6, 5 (or 2/3 swapped)
// or: 1, 9, 8, 3, 2, 4, 7, 6, 5

// Example 2: Render blocking with microtasks
function blockRendering() {
  // This queueMicrotask chain blocks rendering indefinitely
  function loop() {
    queueMicrotask(loop);
  }
  loop();
}
// blockRendering(); // DON'T RUN - UI freezes

// Example 3: Cooperative rendering scheduling
async function yieldToRender() {
  return new Promise((resolve) => {
    setTimeout(resolve, 0); // yields to rendering after macrotask
  });
}

async function updateUI() {
  for (let i = 0; i < 100; i++) {
    element.textContent = `Progress: ${i}%`;
    await yieldToRender(); // allows browser to render
  }
}
```

### Real-World Use Cases

- **Performance profiling** — Understanding which phase is slow.
- **Server-side rendering** — Managing async data fetching order.
- **Game loops** — `requestAnimationFrame` for frame synchronization.
- **CLI tools** — Node.js event loop phases affect file I/O timing.
- **Database drivers** — Connection pool management tied to event loop.

### Common Mistakes

- **Assuming `setTimeout` and `setImmediate` have deterministic order** — They depend on event loop state.
- **Blocking the event loop with synchronous CPU work** — Affects all phases.
- **Not understanding that microtasks block the current phase** — All microtasks must complete before moving on.
- **Confusing browser and Node.js event loops** — They have different phase structures.
- **Creating microtask recursion** — Starves all other phases.

### Best Practices

- Use `requestAnimationFrame` for UI updates (browser).
- Use `setImmediate` for "after I/O but asap" (Node.js).
- Use `process.nextTick` sparingly — it has the highest priority.
- Keep microtask callbacks short to avoid blocking next macrotask.
- Profile with Chrome DevTools Performance tab to see event loop phases.

### Performance Considerations

- Each event loop iteration processes one macrotask and all microtasks.
- Rendering happens at most once per macrotask (usually ~16ms for 60fps).
- Node.js phases are bounded — if a phase takes too long, the loop is delayed.
- `process.nextTick` can starve I/O if called recursively.
- `requestAnimationFrame` is the best signal for frame-bounded work.

### Interview Questions

1. **What are the phases of the browser event loop?**
   Execute macrotask → Process all microtasks → Render (if needed) → Repeat.

2. **What are the phases of the Node.js event loop?**
   Timers → I/O callbacks → Idle/Prepare → Poll → Check (setImmediate) → Close callbacks.

3. **What is the difference between `process.nextTick` and `queueMicrotask`?**
   `process.nextTick` runs before microtasks, between each phase. `queueMicrotask` runs after the current macrotask. `nextTick` has higher priority.

4. **When does `requestAnimationFrame` run in the event loop?**
   After microtasks and before rendering (style, layout, paint).

### Coding Challenges

1. **Create a diagram/function that demonstrates the browser event loop order.**
2. **Build a utility that yields control back to the event loop for rendering.**
3. **Write a Node.js script that demonstrates all six phase orders.**
4. **Implement a scheduler that lets you queue tasks at specific phases.**
5. **Create a function that measures how long each event loop phase takes.**

### Related Topics

- Call stack
- Task queue (macrotasks)
- Microtask queue
- `requestAnimationFrame`
- `setImmediate` / `process.nextTick`
- libuv (Node.js)
