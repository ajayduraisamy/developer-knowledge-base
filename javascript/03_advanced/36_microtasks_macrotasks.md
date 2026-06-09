# Microtasks and Macrotasks

## Introduction

Microtasks and macrotasks are two categories of asynchronous callbacks in JavaScript that are processed by the event loop at different priorities. Microtasks (Promise `.then()`, `queueMicrotask`, `MutationObserver`) have higher priority and are drained completely after each macrotask. Macrotasks (`setTimeout`, `setInterval`, I/O callbacks) are processed one per event loop iteration. Understanding the distinction is critical for predicting execution order and avoiding subtle bugs.

---

## Microtasks definition

### What It Is

A microtask is a short function that is scheduled to execute after the currently executing script or macrotask completes, but before yielding to the event loop to process the next macrotask or perform rendering. Microtasks include Promise reaction callbacks (`.then()`, `.catch()`, `.finally()`), `queueMicrotask()` callbacks, and `MutationObserver` callbacks.

```javascript
// Sources of microtasks:
Promise.resolve().then(() => console.log("Promise microtask"));
queueMicrotask(() => console.log("queueMicrotask microtask"));

const observer = new MutationObserver(() => console.log("Mutation microtask"));
observer.observe(document.body, { childList: true });
document.body.appendChild(document.createElement("div"));
```

### Why It Is Important

Microtasks ensure that:
- Promise callbacks execute as soon as possible after the current operation.
- DOM mutations (via MutationObserver) are processed before rendering.
- State updates in frameworks (Vue, React) are batched and processed efficiently.
- Code scheduled via `queueMicrotask` runs before timers or I/O.
- The microtask queue provides a "clean slate" for continuation-based async patterns.

### How It Works Internally

The JavaScript engine maintains a microtask queue (called the "job queue" in the ECMAScript spec). After each macrotask completes, the engine checks the microtask queue:

1. If the microtask queue is non-empty, take the first microtask.
2. Execute it (it may queue additional microtasks).
3. Repeat until the microtask queue is empty.
4. Only then proceed to the next macrotask or rendering.

The ECMAScript specification defines the `HostEnqueuePromiseJob` operation and the `Job` abstract operation for handling microtasks.

### Syntax

```javascript
// Promise microtask (most common)
Promise.resolve()
  .then(() => { /* microtask */ })
  .catch(() => { /* microtask */ });

// Explicit microtask scheduling
queueMicrotask(() => {
  /* microtask */
});

// MutationObserver (legacy microtask source)
const observer = new MutationObserver(callback);
observer.observe(node, config);
```

### Beginner Examples

```javascript
// Example 1: Basic microtask behavior
console.log("1");
Promise.resolve().then(() => console.log("2"));
console.log("3");
// Output: 1, 3, 2

// Example 2: queueMicrotask
console.log("A");
queueMicrotask(() => console.log("B"));
console.log("C");
// Output: A, C, B

// Example 3: Microtask vs macrotask
setTimeout(() => console.log("macrotask"), 0);
Promise.resolve().then(() => console.log("microtask"));
// Output: microtask, macrotask
```

### Intermediate Examples

```javascript
// Example 1: Microtask queue is FIFO
queueMicrotask(() => console.log("1"));
queueMicrotask(() => console.log("2"));
queueMicrotask(() => console.log("3"));
// Output: 1, 2, 3

// Example 2: Microtask chaining
Promise.resolve()
  .then(() => {
    console.log("first");
    return Promise.resolve();
  })
  .then(() => console.log("second"))
  .then(() => console.log("third"));
// Output: first, second, third

// Example 3: Microtasks inside microtasks
queueMicrotask(() => {
  console.log("outer");
  queueMicrotask(() => console.log("inner"));
});
// Both run before any macrotask
// Output: outer, inner
```

### Advanced Examples

```javascript
// Example 1: Microtask-based batching
class BatchProcessor {
  constructor() {
    this.items = [];
    this.pending = false;
  }

  add(item) {
    this.items.push(item);
    if (!this.pending) {
      this.pending = true;
      queueMicrotask(() => this.flush());
    }
  }

  flush() {
    const batch = this.items.splice(0);
    this.pending = false;
    console.log("Processing batch:", batch);
  }
}

const bp = new BatchProcessor();
bp.add("a");
bp.add("b");
bp.add("c");
// All three are processed in a single microtask
console.log("Items added");
// Output:
// "Items added"
// "Processing batch: ['a', 'b', 'c']"

// Example 2: Microtask-based async state machine
function createTaskRunner() {
  let currentTask = null;
  const tasks = [];

  function runTasks() {
    queueMicrotask(() => {
      while (tasks.length > 0) {
        currentTask = tasks.shift();
        currentTask();
      }
      currentTask = null;
    });
  }

  return {
    add(fn) {
      tasks.push(fn);
      if (!currentTask) runTasks();
    },
  };
}

// Example 3: Counting microtasks
let count = 0;
function trackMicrotasks() {
  queueMicrotask(() => {
    count++;
    if (count < 10) trackMicrotasks();
    else console.log("Processed 10 microtasks");
  });
}
trackMicrotasks();
// All 10 microtasks run before any pending macrotask
```

### Real-World Use Cases

- **React** — Batched state updates inside event handlers use microtasks.
- **Vue** — `nextTick()` uses `queueMicrotask` or Promise.
- **Lit** — Reactive property updates use microtasks.
- **Svelte** — Compiler-generated microtask-based DOM updates.
- **Atomic state management** — Batched subscriber notifications.

### Common Mistakes

- **Assuming `queueMicrotask` is the same as `setTimeout(fn, 0)`** — Microtasks run before macrotasks.
- **Long-running microtasks** — They block rendering and macrotask processing.
- **Recursive microtask scheduling** — Can starve the event loop indefinitely.
- **Not understanding Promise executor sync nature** — The executor runs synchronously, only `.then()` is microtask.

### Best Practices

- Use `queueMicrotask` for lightweight, "run ASAP after current operation" tasks.
- Keep microtask execution time minimal — they block the event loop.
- Use microtasks for batching state updates and DOM change notifications.
- Avoid recursion in microtasks that could lead to starvation.
- Prefer `queueMicrotask` over `Promise.resolve().then()` for clarity when not working with promises.

### Performance Considerations

- Microtask queue processing is synchronous — all microtasks must complete before moving on.
- Very deep microtask chains can cause UI jank (blocking rendering).
- Each microtask has overhead — batching reduces the number of individual microtasks.
- The engine may enforce a microtask depth limit to prevent starvation.
- `MutationObserver` microtasks can be frequent if observing many DOM changes.

### Interview Questions

1. **What is a microtask?**
   A callback scheduled via Promise reactions, `queueMicrotask`, or MutationObserver. Microtasks run after the current macrotask, before the next macrotask.

2. **How is a microtask different from a macrotask?**
   Microtasks have higher priority and the queue is drained completely before processing any macrotask. Macrotasks are processed one per event loop iteration.

3. **What APIs create microtasks?**
   `Promise.then/catch/finally`, `queueMicrotask`, `MutationObserver`, and in Node.js, `process.nextTick` (though it has its own queue that runs before microtasks).

4. **Can microtasks create more microtasks?**
   Yes. The microtask queue is drained until empty, including microtasks added during processing.

### Coding Challenges

1. **Implement `queueMicrotask` using Promises.**
2. **Create a batching utility that coalesces multiple calls into one microtask.**
3. **Build a microtask-based debounce function.**
4. **Write a function that demonstrates microtask starvation.**

### Related Topics

- Event loop
- Macrotasks
- Promises
- `queueMicrotask`
- `MutationObserver`

---

## Macrotasks definition

### What It Is

A macrotask (or just "task") is a callback scheduled via `setTimeout`, `setInterval`, `setImmediate` (Node.js), I/O events, or the event loop's main script execution. Macrotasks are processed one per event loop iteration. After each macrotask, the microtask queue is drained, and rendering may occur.

```javascript
// Sources of macrotasks:
setTimeout(() => console.log("macrotask"), 0);
setInterval(() => console.log("interval macrotask"), 1000);

// Node.js only
setImmediate(() => console.log("check phase macrotask"));

// I/O callbacks
fs.readFile("file.txt", () => console.log("I/O macrotask"));

// UI events (in practice, handled with priority)
button.addEventListener("click", () => console.log("event macrotask"));
```

### Why It Is Important

Macrotasks are the fundamental scheduling unit of the event loop:
- They provide the "tick" rhythm of the event loop.
- They allow rendering to happen between macrotasks.
- They prevent a single long-running task from blocking everything.
- They enable cooperative multitasking via `setTimeout(fn, 0)`.
- They are the lowest priority async mechanism (after microtasks and `nextTick`).

### How It Works Internally

Macrotasks are managed by the host environment (browser or Node.js). The event loop:

1. Picks the oldest macrotask from its queue.
2. Pushes it onto the call stack.
3. Executes it to completion.
4. Processes the microtask queue (empty it).
5. Optionally performs rendering.
6. Repeats.

The HTML specification defines the "task queue" as part of the event loop processing model. Each task source (timers, UI events, I/O) has its own queue, and the browser picks from them in a fairness-based manner.

### Syntax

```javascript
// setTimeout
const id = setTimeout(() => console.log("macrotask"), 100);
clearTimeout(id);

// setInterval
const id = setInterval(() => console.log("repeated macrotask"), 1000);
clearInterval(id);

// setImmediate (Node.js)
setImmediate(() => console.log("check phase macrotask"));

// I/O (Node.js)
const fs = require("fs");
fs.readFile("file.txt", () => console.log("I/O macrotask"));
```

### Beginner Examples

```javascript
// Example 1: Basic macrotask
console.log("1");
setTimeout(() => console.log("2"), 0);
console.log("3");
// Output: 1, 3, 2

// Example 2: Macrotask timing is approximate
const start = Date.now();
setTimeout(() => {
  console.log(`Ran after ${Date.now() - start}ms`);
}, 50);
// Usually ~50ms, but can be longer if stack is busy

// Example 3: Multiple macrotasks are FIFO
setTimeout(() => console.log("first"), 0);
setTimeout(() => console.log("second"), 0);
setTimeout(() => console.log("third"), 0);
// Output: first, second, third
```

### Intermediate Examples

```javascript
// Example 1: Macrotask queuing during execution
setTimeout(() => console.log("macrotask 1"), 0);

Promise.resolve().then(() => {
  console.log("microtask");
  setTimeout(() => console.log("macrotask 2"), 0);
});

setTimeout(() => console.log("macrotask 3"), 0);
// Output: microtask, macrotask 1, macrotask 2, macrotask 3

// Example 2: setInterval accumulation
let count = 0;
const interval = setInterval(() => {
  console.log(`Interval ${++count}`);
  if (count >= 5) clearInterval(interval);
}, 100);
// If the callback takes longer than the interval,
// callbacks queue up and run back-to-back when possible

// Example 3: setImmediate vs setTimeout in Node.js
const fs = require("fs");
fs.readFile(__filename, () => {
  setTimeout(() => console.log("timeout"), 0);
  setImmediate(() => console.log("immediate"));
});
// Output: immediate, timeout (always in I/O phase)
```

### Advanced Examples

```javascript
// Example 1: Macrotask-based cooperative scheduling
function processInChunks(items, chunkSize) {
  let index = 0;
  function processChunk() {
    const end = Math.min(index + chunkSize, items.length);
    for (; index < end; index++) {
      heavyProcess(items[index]);
    }
    if (index < items.length) {
      setTimeout(processChunk, 0);
    } else {
      console.log("Done processing all items");
    }
  }
  setTimeout(processChunk, 0);
}

// Example 2: Mimicking macrotask scheduling with MessageChannel
function scheduleMacrotask(callback) {
  const channel = new MessageChannel();
  channel.port1.onmessage = callback;
  channel.port2.postMessage(null);
  return () => { channel.port1.onmessage = null; };
}

// Example 3: Macrotask phases in Node.js
const fs = require("fs");
fs.writeFileSync("test.txt", "data");

// Timer scheduled first
setTimeout(() => console.log("1 - timer"), 0);
// Check scheduled second
setImmediate(() => console.log("2 - check"));
// I/O will run between timer and check phases
fs.readFile("test.txt", () => {
  console.log("3 - I/O");
  // Inside I/O, nextTick runs before immediate
  process.nextTick(() => console.log("4 - nextTick"));
  setImmediate(() => console.log("5 - check from I/O"));
});
```

### Real-World Use Cases

- **UI event handling** — Click, keypress, scroll events are macrotasks.
- **Timer-based polling** — Periodic server health checks.
- **Chunked data processing** — Breaking large datasets into macrotask-sized chunks.
- **Animation frame scheduling** — `requestAnimationFrame` is before rendering, not a pure macrotask.
- **Debouncing/throttling** — `setTimeout` to rate-limit function calls.

### Common Mistakes

- **Assuming macrotask ordering is deterministic** — Between `setTimeout` and `setImmediate`, it depends on event loop phase.
- **Using `setInterval` for precise timing** — Callback duration affects when the next interval fires.
- **Blocking macrotask processing** — Synchronous CPU-heavy work delays all pending macrotasks.
- **Not clearing timers** — Causes memory leaks and unintended callbacks.
- **Creating too many macrotasks** — Can starve rendering if macrotasks keep the stack busy.

### Best Practices

- Use `setTimeout(fn, 0)` to defer work to the next event loop iteration.
- For repeated work, prefer `setTimeout` recursion over `setInterval` (avoids overlapping calls).
- Always store and clear timer IDs when the component unmounts.
- Chunk heavy synchronous work across multiple macrotasks to keep UI responsive.
- Use `setImmediate` in Node.js for "after I/O, before next timer" scheduling.

### Performance Considerations

- Creating a `setTimeout` with 0 delay has overhead — not free.
- Minimum delay for nested `setTimeout` calls (≥5th level) is 4ms (HTML spec).
- Macrotask queues can grow large if callbacks are slower than the interval.
- Each macrotask transition has overhead (context switching).
- `setImmediate` is more performant than `setTimeout(fn, 0)` in Node.js (no timer data structure).

### Interview Questions

1. **What is a macrotask?**
   A callback from `setTimeout`, `setInterval`, I/O, or UI events. Macrotasks are processed one per event loop iteration.

2. **How does a macrotask differ from a microtask?**
   Macrotasks are lower priority. Multiple macrotasks can be queued, but only one is processed per iteration before microtasks are drained.

3. **What is the minimum delay for `setTimeout`?**
   In modern browsers, 0ms for non-nested calls, but 4ms for nested calls (≥5th nesting level).

4. **Does `setTimeout(fn, 0)` guarantee execution after all promises?**
   Yes, because microtasks are processed before the next macrotask.

### Coding Challenges

1. **Implement a cooperative scheduler that splits work across macrotasks.**
2. **Create a `scheduleAfterRender` utility.**
3. **Build a recursive `setTimeout` clock with drift compensation.**
4. **Write a function that measures the actual `setTimeout` minimum delay.**

### Related Topics

- Event loop
- Microtasks
- `setTimeout` / `setInterval`
- Rendering
- `requestAnimationFrame`

---

## queueMicrotask()

### What It Is

`queueMicrotask()` is a method that explicitly schedules a function to run as a microtask. It provides a standard way to queue microtasks without creating a Promise object. It was introduced to formalize the microtask queuing pattern that developers had been using via `Promise.resolve().then()`.

```javascript
queueMicrotask(() => {
  console.log("This runs as a microtask");
});
```

### Why It Is Important

`queueMicrotask()` provides:
- A direct, explicit API for queuing microtasks without Promise overhead.
- Clear intention — "run this as soon as possible, but not synchronously."
- Better performance than `Promise.resolve().then()` in some engines.
- Standardization of the microtask queuing pattern across environments.
- A way to run code after the current operation but before rendering.

### How It Works Internally

The `queueMicrotask()` function takes a callback and enqueues it in the microtask queue. The engine processes it during the microtask checkpoint, which occurs after each macrotask or synchronous script execution. The callback is called with no arguments and does not create a Promise — it directly interacts with the job queue mechanism defined in the ECMAScript specification.

### Syntax

```javascript
// Basic usage
queueMicrotask(() => {
  console.log("microtask callback");
});

// With error handling
queueMicrotask(() => {
  throw new Error("Uncaught error in microtask");
});
// Errors are reported as unhandled rejections or 'error' events
```

### Beginner Examples

```javascript
// Example 1: Basic microtask scheduling
console.log("1");
queueMicrotask(() => console.log("2"));
console.log("3");
// Output: 1, 3, 2

// Example 2: Multiple microtasks
queueMicrotask(() => console.log("first"));
queueMicrotask(() => console.log("second"));
queueMicrotask(() => console.log("third"));
// Output: first, second, third

// Example 3: queueMicrotask vs setTimeout
queueMicrotask(() => console.log("microtask"));
setTimeout(() => console.log("macrotask"), 0);
// Output: microtask, macrotask
```

### Intermediate Examples

```javascript
// Example 1: queueMicrotask for batching DOM updates
let pendingUpdate = false;
let updateData = null;

function scheduleUpdate(data) {
  updateData = { ...updateData, ...data };
  if (!pendingUpdate) {
    pendingUpdate = true;
    queueMicrotask(() => {
      pendingUpdate = false;
      applyUpdate(updateData);
      updateData = null;
    });
  }
}

// Multiple synchronous calls are batched
scheduleUpdate({ x: 1 });
scheduleUpdate({ y: 2 });
scheduleUpdate({ z: 3 });
// Only one microtask runs, with { x: 1, y: 2, z: 3 }

// Example 2: queueMicrotask with async functions
queueMicrotask(async () => {
  const data = await fetch("/api/data");
  console.log(await data.json());
});
// Note: the async function itself is the microtask;
// the await inside creates additional microtasks

// Example 3: Error handling in queueMicrotask
queueMicrotask(() => {
  try {
    riskyOperation();
  } catch (err) {
    console.error("Microtask error caught:", err.message);
  }
});
// Without try/catch, errors become unhandled rejections
```

### Advanced Examples

```javascript
// Example 1: Promise-less microtask scheduling
function yieldToMicrotasks() {
  return new Promise((resolve) => queueMicrotask(resolve));
}

async function processWithYielding(items) {
  for (const item of items) {
    process(item);
    await yieldToMicrotasks();
  }
}

// Example 2: Microtask priority queue
const microtaskQueue = {
  high: [],
  low: [],
  scheduled: false,
  add(task, priority = "low") {
    this[priority].push(task);
    if (!this.scheduled) {
      this.scheduled = true;
      queueMicrotask(() => this.drain());
    }
  },
  drain() {
    this.scheduled = false;
    const allTasks = [...this.high, ...this.low];
    this.high.length = 0;
    this.low.length = 0;
    allTasks.forEach((task) => task());
  },
};

// Example 3: queueMicrotask polyfill
if (!window.queueMicrotask) {
  window.queueMicrotask = function (callback) {
    Promise.resolve()
      .then(callback)
      .catch((e) =>
        setTimeout(() => {
          throw e;
        }, 0)
      );
  };
}
```

### Real-World Use Cases

- **Framework reactivity systems** — Vue's `nextTick`, Svelte's batching.
- **Lit's reactive updates** — Batched property changes.
- **Web component lifecycle** — `updateComplete` promise.
- **Observable subscriptions** — Coalescing subscriber notifications.
- **State management** — Batching store update notifications.

### Common Mistakes

- **Using queueMicrotask when you should use setTimeout** — Microtasks block rendering, macrotasks don't.
- **Not handling errors inside queueMicrotask** — Errors become unhandled rejections.
- **Assuming queueMicrotask runs before all macrotasks** — It does, but macrotasks queued before the microtask will still run after.
- **Using queueMicrotask for heavy computation** — Blocks the event loop.
- **Creating recursive microtask chains** — Can starve the task queue.

### Best Practices

- Use `queueMicrotask` for lightweight, high-priority deferred work.
- Handle errors with try/catch inside the microtask callback.
- Use `queueMicrotask` to batch multiple synchronous state changes.
- Use `setTimeout` instead for work that can tolerate a frame delay.
- Prefer `queueMicrotask` over `Promise.resolve().then()` for non-promise-microtask needs.

### Performance Considerations

- `queueMicrotask` is slightly more performant than `Promise.resolve().then()` (no Promise object).
- Microtasks are processed in batches — queue multiple calls for efficiency.
- Each microtask callback has call overhead.
- Deep microtask chains can cause microtask queue overflow (engine-dependent limit).
- `queueMicrotask` is available in all modern environments (Node.js, browsers, Deno).

### Interview Questions

1. **What is `queueMicrotask` and when should you use it?**
   It schedules a callback as a microtask. Use it when you need to run code after the current operation but before rendering or macrotasks.

2. **What is the difference between `queueMicrotask(fn)` and `Promise.resolve().then(fn)`?**
   Functionally similar, but `queueMicrotask` is more explicit and may be slightly faster (no Promise wrapper).

3. **Does `queueMicrotask` work with error handling?**
   Yes, but uncaught errors in microtasks become unhandled promise rejections (or fire the `error` event in browsers).

4. **Can you cancel a queued microtask?**
   No. Unlike `setTimeout`, `queueMicrotask` does not return a cancellation token.

### Coding Challenges

1. **Implement `queueMicrotask` without using Promises (using `MutationObserver`).**
2. **Create a batched microtask scheduler.**
3. **Build a `nextTick` function similar to Node.js using `queueMicrotask`.**
4. **Write a utility that yields to the microtask queue.**

### Related Topics

- Microtasks
- Event loop
- Promises
- `MutationObserver`
- `process.nextTick`

---

## Execution order examples

### What It Is

Execution order examples demonstrate how the interplay between synchronous code, microtasks, and macrotasks determines the sequence in which callbacks run. These examples are essential for developing an intuition about the event loop and avoiding timing-dependent bugs.

```javascript
// Classic order demonstration
console.log("1 - sync");
setTimeout(() => console.log("2 - macrotask"), 0);
Promise.resolve().then(() => console.log("3 - microtask"));
console.log("4 - sync");
// Output: 1, 4, 3, 2
```

### Why It Is Important

Understanding execution order is critical because:
- It helps debug async code that seems "random" or "race-condition-like."
- It's essential knowledge for working with frameworks (React, Vue, Angular).
- It determines when DOM updates appear to users.
- It affects resource cleanup and initialization ordering.
- It's a common interview topic for senior positions.

### How It Works Internally

The execution order follows these rules:
1. Synchronous code runs first (current macrotask).
2. Microtask queue is drained (all pending microtasks).
3. Browser may render.
4. Next macrotask is picked from the queue.
5. Microtask queue is drained again.
6. Repeat.

When a macrotask callback runs, it may queue more microtasks and macrotasks. Those microtasks are processed before the next macrotask.

### Syntax

The order is implicit — determined by the event loop processing model:

```javascript
// Template for testing order
function orderTest() {
  console.log("sync start");

  setTimeout(() => console.log("timeout"), 0);

  Promise.resolve().then(() => console.log("promise"));

  queueMicrotask(() => console.log("microtask"));

  console.log("sync end");
}
```

### Beginner Examples

```javascript
// Example 1: Basic order
setTimeout(() => console.log("1"), 0);
console.log("2");
Promise.resolve().then(() => console.log("3"));
console.log("4");
// Output: 2, 4, 3, 1

// Example 2: Promise executor is sync
console.log("start");
const p = new Promise((resolve) => {
  console.log("executor");
  resolve("resolved");
});
p.then(console.log);
console.log("end");
// Output: start, executor, end, resolved

// Example 3: Multiple promises
Promise.resolve("a").then((v) => console.log(v));
Promise.resolve("b").then((v) => console.log(v));
Promise.resolve("c").then((v) => console.log(v));
console.log("sync");
// Output: sync, a, b, c
```

### Intermediate Examples

```javascript
// Example 1: Mixed scheduling
console.log("1");

setTimeout(() => {
  console.log("2");
  Promise.resolve().then(() => console.log("3"));
}, 0);

Promise.resolve().then(() => {
  console.log("4");
  setTimeout(() => console.log("5"), 0);
});

setTimeout(() => console.log("6"), 0);

console.log("7");
// Output: 1, 7, 4, 2, 3, 6, 5

// Let's trace step by step:
// 1. Sync: logs "1", schedules timeout1, schedules microtask1, schedules timeout2
// 2. Sync: logs "7"
// 3. Microtask: [microtask1] -> logs "4", schedules timeout3
// 4. Macrotask: [timeout1] -> logs "2", schedules microtask2
// 5. Microtask: [microtask2] -> logs "3"
// 6. Macrotask: [timeout2] -> logs "6"
// 7. Macrotask: [timeout3] -> logs "5"

// Example 2: Nested microtasks
queueMicrotask(() => {
  console.log("outer microtask");
  queueMicrotask(() => console.log("inner microtask"));
});
setTimeout(() => console.log("timeout"), 0);
// Output: outer microtask, inner microtask, timeout
// Both microtasks run before the macrotask

// Example 3: await creates microtask boundaries
async function test() {
  console.log("async start");
  await Promise.resolve();
  console.log("async after await");
}
test();
console.log("sync");
// Output: async start, sync, async after await
// The part after await is a microtask
```

### Advanced Examples

```javascript
// Example 1: Complex scenario
console.log("1 - sync");

setTimeout(() => {
  console.log("2 - timeout1");
  Promise.resolve().then(() => {
    console.log("3 - timeout1 microtask");
  });
}, 0);

Promise.resolve()
  .then(() => {
    console.log("4 - promise1");
    setTimeout(() => {
      console.log("5 - promise1 timeout");
      queueMicrotask(() => console.log("6 - promise1 timeout microtask"));
    }, 0);
  })
  .then(() => console.log("7 - promise2"));

setTimeout(() => {
  console.log("8 - timeout2");
  queueMicrotask(() => console.log("9 - timeout2 microtask"));
}, 0);

console.log("10 - sync end");
// Output: 1, 10, 4, 7, 2, 3, 8, 9, 5, 6

// Example 2: Event loop with async/await
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}

async function async2() {
  console.log("async2");
}

console.log("script start");
setTimeout(() => console.log("setTimeout"), 0);
async1();
new Promise((resolve) => {
  console.log("promise1");
  resolve();
}).then(() => console.log("promise2"));
console.log("script end");
// Output:
// script start
// async1 start
// async2
// promise1
// script end
// async1 end
// promise2
// setTimeout

// Example 3: Node.js specific order
const fs = require("fs");

setTimeout(() => console.log("timeout"), 0);
setImmediate(() => console.log("immediate"));
process.nextTick(() => console.log("nextTick"));
Promise.resolve().then(() => console.log("promise"));

fs.writeFileSync("temp.txt", "test");
fs.readFile("temp.txt", () => {
  console.log("I/O callback");
  process.nextTick(() => console.log("I/O nextTick"));
  setImmediate(() => console.log("I/O immediate"));
  setTimeout(() => console.log("I/O timeout"), 0);
});

console.log("sync end");
// Likely output: sync end, nextTick, promise, timeout, immediate, I/O callback, I/O nextTick, I/O immediate, I/O timeout
// (timeout/immediate order in main module may vary)
```

### Real-World Use Cases

- **React concurrent mode** — Scheduling priorities affect rendering order.
- **Database transaction sequencing** — Ensuring operations run in the intended order.
- **Animation sequencing** — Firing animations after state updates.
- **Test assertions** — Understanding when assertions run relative to async operations.
- **Middleware ordering** — Ensuring middleware runs in the correct sequence.

### Common Mistakes

- **Assuming `setTimeout(fn, 0)` means "run next"** — Microtasks always run before it.
- **Not accounting for microtasks in async test assertions** — Tests may pass/fail nondeterministically.
- **Expecting rendering between promise chains** — Microtasks block rendering.
- **Confusing `process.nextTick` with `queueMicrotask` order** — `nextTick` runs before microtasks in Node.js.

### Best Practices

- Use execution order diagrams when debugging complex async flows.
- Test async code with explicit order expectations.
- Use `await` and yield control points explicitly rather than relying on implicit ordering.
- Keep the number of microtask sources small and predictable.
- Use `Promise.all` and well-structured async functions instead of relying on tricky interleaving.

### Performance Considerations

- Synchronous code is the fastest — minimize async scheduling in hot paths.
- Each async boundary (await, microtask) adds overhead.
- Deep nesting of microtasks and macrotasks can cause order-dependent performance issues.
- The event loop processing model is well-defined but varies slightly between environments.
- Engine optimizations (like microtask batching) affect exact timing.

### Interview Questions

1. **What is the output of `console.log('a'); setTimeout(() => console.log('b'), 0); Promise.resolve().then(() => console.log('c')); console.log('d');`**
   a, d, c, b

2. **What happens first: a microtask or a macrotask?**
   A microtask. Microtasks are processed after the current macrotask completes, before the next macrotask.

3. **How does `await` affect execution order?**
   The code after `await` is scheduled as a microtask. The calling code continues synchronously up to the await.

4. **What is the difference between `process.nextTick` and `queueMicrotask` in Node.js?**
   `process.nextTick` runs between each phase of the event loop, before microtasks. `queueMicrotask` runs after the current macrotask.

### Coding Challenges

1. **Predict the output of a given async code snippet.**
2. **Write a function that forces a specific execution order.**
3. **Create a visualizer that logs event loop phases.**
4. **Build a test helper that asserts execution order.**

### Related Topics

- Event loop phases
- Call stack
- Microtask queue
- Macrotask queue
- `process.nextTick`
- Event loop visualization tools
