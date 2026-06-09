# Promises

## Introduction

A Promise is an object representing the eventual completion or failure of an asynchronous operation. Promises provide a cleaner, more composable alternative to callbacks for handling asynchronous code. They were introduced in ES6 and have become the foundation of modern JavaScript async programming, enabling patterns like chaining, parallel execution, and seamless error propagation.

---

## Promise states (pending/fulfilled/rejected)

### What It Is

A Promise is always in one of three mutually exclusive states:
- **pending** — Initial state, neither fulfilled nor rejected.
- **fulfilled** — The operation completed successfully, and the promise has a value.
- **rejected** — The operation failed, and the promise has a reason (error).

Once a promise transitions from pending to either fulfilled or rejected, it is **settled** and its state cannot change again.

```javascript
const promise = new Promise((resolve, reject) => {
  // pending state
  setTimeout(() => {
    resolve("Success!"); // transitions to fulfilled
    // reject(new Error("Fail")); // transitions to rejected
  }, 1000);
});

console.log(promise); // Promise { <pending> }
// After 1 second: Promise { <fulfilled>: "Success!" }
```

### Why It Is Important

Understanding promise states is crucial because:
- It determines how `.then()`, `.catch()`, and `.finally()` handlers behave.
- State immutability ensures that a promise resolves exactly once.
- It enables safe composition — you can attach handlers even after settlement.
- It forms the basis for async/await syntax.
- It allows predictable error propagation through chains.

### How It Works Internally

The Promise constructor receives an executor function `(resolve, reject) => { ... }` which is called synchronously. The executor receives two functions:
- `resolve(value)` — Transitions the promise from pending to fulfilled with the given value. If `value` is another promise, the promise "follows" it (adopts its state).
- `reject(reason)` — Transitions from pending to rejected with the given reason.

Internally, the promise maintains:
- `[[PromiseState]]` — One of `"pending"`, `"fulfilled"`, or `"rejected"`.
- `[[PromiseResult]]` — The fulfilled value or rejection reason (undefined while pending).
- `[[PromiseFulfillReactions]]` — List of `then` handlers queued while pending.
- `[[PromiseRejectReactions]]` — List of `catch` handlers queued while pending.

When a promise settles, all queued handlers are scheduled as microtasks.

### Syntax

```javascript
// Creating a promise
const promise = new Promise((resolve, reject) => {
  // async work
  if (success) {
    resolve(value);
  } else {
    reject(error);
  }
});

// Utility methods that return promises in specific states
const fulfilled = Promise.resolve("immediate value");
const rejected = Promise.reject(new Error("immediate error"));
const pending = new Promise(() => {}); // never settles

// Checking state (not directly possible in user code)
// Use .then() to react to state changes
```

### Beginner Examples

```javascript
// Example 1: Basic promise lifecycle
const greeting = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Hello, world!");
  }, 500);
});

console.log(greeting); // Promise { <pending> }
greeting.then((value) => console.log(value)); // "Hello, world!" (after 500ms)

// Example 2: Immediate settlement
const immediate = Promise.resolve(42);
console.log(immediate); // Promise { <fulfilled>: 42 }
immediate.then((v) => console.log(v)); // 42 (microtask)

// Example 3: Rejection
const failing = Promise.reject(new Error("Something went wrong"));
failing.catch((err) => console.log(err.message)); // "Something went wrong"

// Example 4: State is immutable
const immutable = new Promise((resolve, reject) => {
  resolve("first");
  resolve("second"); // ignored
  reject(new Error("ignored")); // ignored
});
immutable.then(console.log); // "first"
```

### Intermediate Examples

```javascript
// Example 1: Promise follows another promise
const inner = new Promise((resolve) =>
  setTimeout(() => resolve("inner value"), 500)
);

const outer = new Promise((resolve) => {
  resolve(inner); // outer adopts inner's state
});

outer.then(console.log); // "inner value" (waits for inner)

// Example 2: State inspection with .then
const p = fetch("https://api.example.com/data");
p.then(
  (response) => console.log("Fulfilled with:", response),
  (error) => console.log("Rejected with:", error)
);

// Example 3: Promise that never settles
const neverSettles = new Promise(() => {});
// This is pending forever - useful for timeouts
function timeout(ms) {
  return new Promise((_, reject) =>
    setTimeout(() => reject(new Error("Timed out")), ms)
  );
}

function withTimeout(promise, ms) {
  return Promise.race([promise, timeout(ms)]);
}
```

### Advanced Examples

```javascript
// Example 1: Deferred pattern
class Deferred {
  constructor() {
    this.promise = new Promise((resolve, reject) => {
      this.resolve = resolve;
      this.reject = reject;
    });
  }
}

// Usage
function createCancellableTask() {
  const deferred = new Deferred();
  const timer = setTimeout(() => {
    deferred.resolve("Task completed");
  }, 5000);

  return {
    promise: deferred.promise,
    cancel: () => {
      clearTimeout(timer);
      deferred.reject(new Error("Task cancelled"));
    },
  };
}

const task = createCancellableTask();
task.promise.then(console.log).catch(console.error);
// Later: task.cancel() to reject

// Example 2: Promise state machine
const STATE = {
  PENDING: "pending",
  FULFILLED: "fulfilled",
  REJECTED: "rejected",
};

function createStateTrackingPromise(executor) {
  let state = STATE.PENDING;
  const promise = new Promise((resolve, reject) => {
    const trackedResolve = (value) => {
      state = STATE.FULFILLED;
      resolve(value);
    };
    const trackedReject = (reason) => {
      state = STATE.REJECTED;
      reject(reason);
    };
    executor(trackedResolve, trackedReject);
  });
  promise.getState = () => state;
  return promise;
}
```

### Real-World Use Cases

- **Fetch API** — `fetch(url)` returns a Promise.
- **Database queries** — MongoDB, PostgreSQL drivers return Promises.
- **File system** — `fs.promises` API returns Promises.
- **Caching** — Memoized async operations return Promises.
- **Rate limiting** — Queue-based operation scheduling.

```javascript
// Fetch API example
fetch("https://api.github.com/users/octocat")
  .then((response) => {
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  })
  .then((data) => console.log(data))
  .catch((err) => console.error("Fetch failed:", err));
```

### Common Mistakes

- **Not returning a promise from a `.then()` handler** — Breaks the chain.
- **Creating a promise that never settles** — Leads to unhandled promise rejection warnings (or silent hangs).
- **Calling `resolve` or `reject` multiple times** — Subsequent calls are ignored.
- **Resolving with a pending promise that never settles** — Makes the outer promise pending forever.
- **Assuming state is observable** — You cannot inspect state synchronously (except in tests with custom wrappers).

### Best Practices

- Always return promises from async functions.
- Avoid the explicit construction antipattern — only use `new Promise` when wrapping callback APIs.
- Prefer `Promise.resolve()` and `Promise.reject()` for immediate values.
- Use `Promise.allSettled` when you need results from all promises regardless of failure.
- Avoid promise chains longer than 3-5 steps — extract into named async functions.

### Performance Considerations

- Creating a Promise has overhead (object allocation). For hot paths, consider caching promises.
- Promise microtasks are higher priority than macrotasks — be mindful of starving the event loop.
- `Promise.all` fails fast — if one rejects, the whole operation rejects immediately.
- `Promise.allSettled` always waits for all promises, which uses more memory for error objects.
- Unhandled promise rejections have detection overhead (engine tracks all rejections).

### Interview Questions

1. **What are the three states of a Promise?**
   Pending (initial), fulfilled (success), rejected (failure). Once settled (fulfilled/rejected), the state is immutable.

2. **Can a Promise be both resolved and rejected?**
   No. Once resolved or rejected, subsequent calls to resolve/reject are ignored.

3. **What happens if you resolve a Promise with another Promise?**
   The outer promise "follows" the inner promise — it adopts its state and waits for it to settle.

4. **What is the difference between `settled` and `pending`?**
   A settled promise is either fulfilled or rejected. A pending promise is still in progress.

### Coding Challenges

1. **Implement a simple Promise class that tracks state.**
2. **Create a function `isPromise(value)` that checks if something is a Promise.**
3. **Build a `Deferred` utility with external resolve/reject.**
4. **Implement `Promise.resolve` and `Promise.reject` from scratch.**
5. **Write a function that returns a promise's state synchronously (using a wrapper).**

### Related Topics

- Async/await
- Promise chaining
- Error handling
- Microtasks
- `then()` / `catch()` / `finally()`

---

## then() catch() finally()

### What It Is

`then()`, `catch()`, and `finally()` are the three instance methods on Promise objects that allow you to react to settlement. `then()` handles fulfillment and optionally rejection, `catch()` handles rejection only, and `finally()` runs a cleanup function regardless of the outcome.

```javascript
promise
  .then((value) => console.log("Fulfilled:", value))
  .catch((error) => console.error("Rejected:", error))
  .finally(() => console.log("Cleanup"));
```

### Why It Is Important

These methods form the primary API for consuming promises:
- `then()` enables transformation and chaining of results.
- `catch()` provides centralized error handling.
- `finally()` ensures cleanup (closing connections, hiding spinners) regardless of outcome.
- They all return new promises, enabling composable async pipelines.

### How It Works Internally

When a promise settles, the engine creates microtasks from the registered handlers:
- `onFulfilled` handlers from `.then()` are queued for fulfilled promises.
- `onRejected` handlers from `.then()` or `.catch()` are queued for rejected promises.
- `finally()` handlers are always queued regardless of settlement.

Each method returns a new promise. The returned promise is:
- Resolved with the return value of the handler (if handler returns non-promise).
- Follows the returned promise (if handler returns a promise).
- Rejected if the handler throws.

### Syntax

```javascript
// then(onFulfilled, onRejected)
promise.then(
  (value) => { /* handle fulfillment */ },
  (error) => { /* handle rejection */ }
);

// catch(onRejected)
promise.catch((error) => { /* handle rejection */ });

// finally(onFinally)
promise.finally(() => { /* always called */ });
```

### Beginner Examples

```javascript
// Example 1: Using then for fulfillment
Promise.resolve("Hello")
  .then((value) => value + " World")
  .then((value) => console.log(value)); // "Hello World"

// Example 2: Using catch for errors
Promise.reject(new Error("Oops"))
  .catch((err) => console.log("Caught:", err.message)); // "Caught: Oops"

// Example 3: Using finally for cleanup
let loading = true;
fetchData()
  .then((data) => render(data))
  .catch((err) => showError(err))
  .finally(() => {
    loading = false;
  });

// Example 4: then with two callbacks
const p = Math.random() > 0.5
  ? Promise.resolve("win")
  : Promise.reject(new Error("lose"));

p.then(
  (value) => console.log("Success:", value),
  (error) => console.log("Failure:", error.message)
);
```

### Intermediate Examples

```javascript
// Example 1: Error recovery with catch
fetch("/api/data")
  .then((response) => {
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  })
  .catch((err) => {
    console.warn("Fetch failed, using cached data:", err);
    return getCachedData(); // Recovery: return a fallback value
  })
  .then((data) => render(data));

// Example 2: finally doesn't change the promise value
function fetchWithLogging(url) {
  return fetch(url).finally(() => console.log(`Request to ${url} completed`));
}

fetchWithLogging("/api/users")
  .then((res) => res.json())
  .catch((err) => console.error(err));

// Example 3: Re-throwing in catch
Promise.resolve()
  .then(() => {
    throw new Error("Original error");
  })
  .catch((err) => {
    console.log("Logged:", err.message);
    throw err; // Re-throw to propagate
  })
  .catch((err) => {
    console.log("Final handler:", err.message);
  });
// Output:
// "Logged: Original error"
// "Final handler: Original error"
```

### Advanced Examples

```javascript
// Example 1: Fluent retry with then/catch
function fetchWithRetry(url, retries = 3) {
  return fetch(url).catch((err) => {
    if (retries <= 0) throw err;
    console.log(`Retrying... (${retries} left)`);
    return fetchWithRetry(url, retries - 1);
  });
}

// Example 2: Tap operator (side effects without modifying value)
function tap(fn) {
  return (value) => {
    fn(value);
    return value;
  };
}

Promise.resolve("data")
  .then(tap(console.log))
  .then((value) => value.toUpperCase())
  .then(tap(console.log));
// "data"
// "DATA"

// Example 3: Using finally for resource cleanup
class DatabaseConnection {
  async query(sql) {
    try {
      const result = await this._execute(sql);
      return result;
    } finally {
      this._release(); // Always release the connection
    }
  }
}

// Example 4: Conditional chaining
function getUserData(userId) {
  return fetch(`/api/users/${userId}`)
    .then((res) => res.json())
    .then((user) => {
      if (user.role === "admin") {
        return fetch(`/api/admin/users`).then((r) => r.json());
      }
      return user;
    });
}
```

### Real-World Use Cases

- **Axios interceptors** — `then`/`catch` for request/response transformation.
- **React Query** — Uses promise chains for data fetching with retry.
- **Express async middleware** — Wrapping errors in catch.
- **Service workers** — Promise chains for caching strategies.
- **Stream processing** — Chaining transforms on data pipelines.

### Common Mistakes

- **Forgetting to return a promise from then** — Breaks the chain.
- **Not returning the catch —** Errors become unhandled.
- **Misunderstanding finally's return value** — Finally does NOT change the resolved value unless it throws.
- **Swallowing errors in catch without re-throwing** — The chain becomes resolved with undefined.
- **Nesting then instead of chaining** — Creates pyramid-like structure.

### Best Practices

- Always return a value from `then` handlers (even `undefined`) to avoid confusion.
- Put `catch` at the end of chains to catch all errors.
- Use `finally` for cleanup, not for value transformation.
- Avoid using the second argument to `then` — use `catch` instead for clarity.
- Keep handler functions small — extract named functions for complex logic.

### Performance Considerations

- Each `then`/`catch`/`finally` creates a new Promise object — moderate overhead.
- Chaining 100+ promises is fine; the overhead is microseconds per link.
- `.finally()` is slightly more expensive than `.then()` because it must handle both states.
- Promise handlers are always called asynchronously (microtask), even for already-settled promises.
- Using async/await compiles to promise chains with similar performance characteristics.

### Interview Questions

1. **How is `promise.then(fn).catch(fn)` different from `promise.then(fn, fn)`?**
   `.then(fn).catch(fn)` — If the first `fn` throws, the `catch` catches it. `.then(fn, fn)` — The second `fn` only catches errors from the original promise, not from the first `fn`.

2. **Does `finally()` receive the resolved value or rejection reason?**
   No. `finally()` receives no arguments. It simply runs cleanup.

3. **Can `finally()` change the resolved value?**
   No, unless it throws (which causes rejection) or returns a pending promise (which delays settlement).

4. **What happens if you don't return a value from `.then()`?**
   The next `.then()` receives `undefined`.

### Coding Challenges

1. **Implement `Promise.prototype.then`, `catch`, and `finally` from scratch.**
2. **Create a `tap` utility for side effects in promise chains.**
3. **Build a retry mechanism using recursive then/catch.**
4. **Implement a timeout decorator for promises.**

### Related Topics

- Promise chaining
- Error handling
- Promise combinators
- Microtask scheduling

---

## Promise.all() and Promise.race()

### What It Is

`Promise.all()` and `Promise.race()` are static methods on the Promise constructor that operate on iterables of promises. `Promise.all()` waits for all promises to fulfill and returns an array of fulfillment values. If any promise rejects, it immediately rejects with that reason. `Promise.race()` settles as soon as the first promise settles (either fulfills or rejects), adopting its value or reason.

```javascript
const promises = [
  fetch("/api/users"),
  fetch("/api/posts"),
  fetch("/api/comments"),
];

Promise.all(promises)
  .then(([users, posts, comments]) => {
    console.log("All data loaded");
  })
  .catch((err) => console.error("One or more failed:", err));

Promise.race([
  fetch("/api/data"),
  timeout(5000),
]).then(console.log).catch(console.error);
```

### Why It Is Important

These combinators solve common async coordination problems:
- `Promise.all` — Run multiple async operations in parallel and wait for all to complete.
- `Promise.race` — Implement timeouts, fallbacks, or race conditions.
- `Promise.allSettled` — Wait for all operations regardless of failure (ES2020).
- `Promise.any` — Wait for the first successful fulfillment (ES2021).

### How It Works Internally

`Promise.all` creates an array to collect results. It iterates the input, subscribing to each promise with `.then()`. Each fulfillment stores the result at the correct index. When the count of fulfilled promises reaches the input length, it resolves the outer promise with the results array. If any input rejects, the outer promise is immediately rejected.

`Promise.race` subscribes to all input promises and resolves/rejects the outer promise with the first settlement value.

### Syntax

```javascript
// Promise.all(iterable)
const allPromise = Promise.all([promise1, promise2, promise3]);

// Promise.race(iterable)
const racePromise = Promise.race([promise1, promise2]);

// Promise.allSettled(iterable)
const settledPromise = Promise.allSettled([promise1, promise2]);

// Promise.any(iterable)
const anyPromise = Promise.any([promise1, promise2]);
```

### Beginner Examples

```javascript
// Example 1: Promise.all - parallel fetch
const urls = [
  "https://api.github.com/users/octocat",
  "https://api.github.com/users/torvalds",
  "https://api.github.com/users/gaearon",
];

Promise.all(urls.map((url) => fetch(url).then((r) => r.json())))
  .then((users) => {
    users.forEach((user) => console.log(user.login));
  })
  .catch((err) => console.error("One request failed:", err));

// Example 2: Promise.race - timeout
const fetchWithTimeout = (url, ms) =>
  Promise.race([
    fetch(url),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error(`Request timed out after ${ms}ms`)), ms)
    ),
  ]);

fetchWithTimeout("https://api.example.com/data", 3000)
  .then((res) => res.json())
  .catch((err) => console.error(err.message));

// Example 3: Promise.all with non-promise values
Promise.all([1, Promise.resolve(2), 3]).then(console.log); // [1, 2, 3]
```

### Intermediate Examples

```javascript
// Example 1: Promise.all with error handling
async function loadUserProfile(userId) {
  const promises = [
    fetch(`/api/users/${userId}`).then((r) => r.json()),
    fetch(`/api/users/${userId}/posts`).then((r) => r.json()),
    fetch(`/api/users/${userId}/settings`).then((r) => r.json()),
  ];
  try {
    const [user, posts, settings] = await Promise.all(promises);
    return { user, posts, settings };
  } catch (err) {
    console.error("Failed to load profile:", err);
    throw err;
  }
}

// Example 2: Promise.allSettled
async function loadNonCriticalData() {
  const results = await Promise.allSettled([
    fetch("/api/recommendations").then((r) => r.json()),
    fetch("/api/ads").then((r) => r.json()),
    fetch("/api/trending").then((r) => r.json()),
  ]);

  const data = {};
  results.forEach((result, index) => {
    if (result.status === "fulfilled") {
      data[`source${index}`] = result.value;
    } else {
      console.warn(`Source ${index} failed:`, result.reason);
      data[`source${index}`] = null;
    }
  });
  return data;
}

// Example 3: Promise.any for fallback servers
const servers = [
  "https://us-east.api.example.com",
  "https://eu-west.api.example.com",
  "https://ap-southeast.api.example.com",
];

Promise.any(
  servers.map((server) =>
    fetch(`${server}/health`)
      .then((r) => {
        if (!r.ok) throw new Error(`Server ${server} unhealthy`);
        return server;
      })
  )
).then((fastestHealthy) => {
  console.log(`Using server: ${fastestHealthy}`);
}).catch((err) => {
  console.error("All servers failed:", err);
});
```

### Advanced Examples

```javascript
// Example 1: Promise.all with concurrency control
async function parallelLimit(tasks, limit) {
  const results = [];
  const executing = new Set();

  for (const [index, task] of tasks.entries()) {
    const promise = Promise.resolve().then(() => task());
    results[index] = promise;
    executing.add(promise);

    const clean = () => executing.delete(promise);
    promise.then(clean, clean);

    if (executing.size >= limit) {
      await Promise.race(executing);
    }
  }

  return Promise.all(results);
}

const tasks = Array.from({ length: 10 }, (_, i) => () =>
  fetch(`/api/page/${i}`)
);

parallelLimit(tasks, 3).then((pages) => console.log("All pages loaded"));

// Example 2: Race with multiple timeouts (exponential backoff)
function raceWithBackoff(url, maxAttempts = 3) {
  const attempts = [];
  for (let i = 0; i < maxAttempts; i++) {
    const delay = Math.pow(2, i) * 1000;
    attempts.push(
      new Promise((resolve, reject) => {
        setTimeout(() => {
          fetch(url).then(resolve).catch(reject);
        }, delay);
      })
    );
  }
  return Promise.race(attempts);
}

// Example 3: Promise.all for batch processing with progress
async function batchWithProgress(items, batchSize, processor) {
  const results = [];
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map((item) => processor(item))
    );
    results.push(...batchResults);
    console.log(`Progress: ${Math.min(i + batchSize, items.length)}/${items.length}`);
  }
  return results;
}
```

### Real-World Use Cases

- **Microservice orchestration** — `Promise.all` gathers data from multiple services.
- **Preloading assets** — Load images, fonts, and scripts in parallel.
- **Database batch operations** — Insert multiple records concurrently.
- **Circuit breaker** — `Promise.race` to detect slow responses.
- **Feature flags** — Load multiple config sources and use fastest response.

### Common Mistakes

- **Using Promise.all when operations should be sequential** — Wastes resources on dependencies.
- **Not handling rejection in Promise.all** — One failure loses all results.
- **Using Promise.race for timeouts without cleanup** — The slow promise still runs (and may cause side effects).
- **Assuming Promise.race cancels losers** — Promises are not cancellable; race just ignores later settlements.
- **Passing non-iterable to Promise.all** — TypeErrors.

### Best Practices

- Use `Promise.allSettled` instead of `Promise.all` when you need all results regardless of errors.
- Use `Promise.any` instead of `Promise.race` when you want the first success, not first settlement.
- Limit concurrency with `Promise.all` to avoid overwhelming resources.
- Always catch errors from `Promise.all` to get meaningful error messages.
- Use `Promise.all` with timeouts using `Promise.race` for network requests.

### Performance Considerations

- `Promise.all` runs all promises concurrently — memory scales with the number of promises.
- `Promise.race` doesn't cancel losers — they continue executing and consuming resources.
- Large `Promise.all` calls (thousands of promises) can cause microtask queue overflow.
- `Promise.allSettled` collects all results before resolving, using memory proportional to input size.
- Consider chunking for massive parallel operations (100+ promises).

### Interview Questions

1. **What is the difference between Promise.all and Promise.allSettled?**
   `Promise.all` rejects immediately on any rejection. `Promise.allSettled` waits for all promises to settle and returns their status/value/reason.

2. **What is the difference between Promise.race and Promise.any?**
   `Promise.race` settles on the first settled promise (fulfilled or rejected). `Promise.any` settles on the first fulfilled promise, rejecting only if all reject.

3. **Can Promise.all accept non-promise values?**
   Yes. Non-thenable values are wrapped in `Promise.resolve()`.

4. **What happens if you pass an empty array to Promise.all?**
   It resolves immediately with an empty array.

### Coding Challenges

1. **Implement your own `Promise.all` from scratch.**
2. **Implement `Promise.allSettled`.**
3. **Implement `Promise.race` from scratch.**
4. **Build a `parallelLimit` utility that runs N async functions with a max concurrency.**
5. **Create a `timeout` function that wraps any promise with a timeout.**

### Related Topics

- Promise combinators
- Concurrency control
- Error propagation
- Async iteration
- `Promise.allSettled`, `Promise.any`

---

## Promise chaining

### What It Is

Promise chaining is the pattern of connecting multiple `.then()` calls sequentially, where each handler can return a value (which becomes the next promise's fulfillment) or a new promise (which is flattened into the chain). This allows expressing sequential asynchronous operations in a linear fashion, avoiding callback hell.

```javascript
fetchUser(userId)
  .then((user) => fetchPosts(user.id))
  .then((posts) => fetchComments(posts[0].id))
  .then((comments) => renderComments(comments))
  .catch((error) => console.error("Failed:", error));
```

### Why It Is Important

Chaining is the primary mechanism for composing asynchronous operations:
- It creates a clear, linear sequence of operations.
- Errors propagate automatically down the chain.
- Each step can transform the result.
- It enables flat structure instead of nested callbacks.
- It allows conditional async logic within the chain.

### How It Works Internally

Each `.then()`, `.catch()`, and `.finally()` call returns a new Promise. When a handler function returns:
- A non-thenable value → The new promise is resolved with that value.
- A thenable/promise → The new promise adopts that promise's state (flattened).
- A thrown value → The new promise is rejected with that thrown value.

The chain stores the previous promise and connects it to the new one via reaction records. When the previous promise settles, it schedules the handler as a microtask.

### Syntax

```javascript
// Basic chain
promise
  .then(step1)
  .then(step2)
  .then(step3)
  .catch(handleError);

// Chain with conditional branching
promise
  .then((value) => {
    if (condition) return asyncOp1(value);
    return asyncOp2(value);
  })
  .then((result) => processResult(result));
```

### Beginner Examples

```javascript
// Example 1: Simple transformation chain
Promise.resolve(1)
  .then((n) => n + 1)
  .then((n) => n * 2)
  .then((n) => n.toString())
  .then(console.log); // "4"

// Example 2: Fetch chain
fetch("https://api.github.com/users/octocat")
  .then((response) => {
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  })
  .then((data) => {
    return fetch(data.repos_url);
  })
  .then((response) => response.json())
  .then((repos) => {
    console.log(`Found ${repos.length} repositories`);
  })
  .catch((err) => console.error(err));

// Example 3: Returning promises from then
function step1() {
  return new Promise((resolve) => setTimeout(() => resolve("A"), 100));
}
function step2(value) {
  return new Promise((resolve) => setTimeout(() => resolve(value + "B"), 100));
}

step1()
  .then(step2)
  .then(console.log); // "AB" after ~200ms
```

### Intermediate Examples

```javascript
// Example 1: Error recovery in chain
fetch("/api/data")
  .then((res) => {
    if (!res.ok) throw new Error(`Status ${res.status}`);
    return res.json();
  })
  .catch((err) => {
    if (err.message.includes("404")) return { default: true };
    throw err; // Re-throw other errors
  })
  .then((data) => processData(data))
  .catch((err) => console.error("Unrecoverable:", err));

// Example 2: Branching chain
function getData(type) {
  return fetch(`/api/${type}`)
    .then((res) => res.json())
    .then((data) => {
      if (type === "users") {
        return data.map((u) => u.name);
      }
      if (type === "posts") {
        return data.map((p) => p.title);
      }
      return data;
    });
}

// Example 3: Chain with finally for cleanup
let isLoading = true;
showSpinner();

fetchData()
  .then((data) => render(data))
  .catch((err) => showError(err))
  .finally(() => {
    isLoading = false;
    hideSpinner();
  });
```

### Advanced Examples

```javascript
// Example 1: Dynamic chain building
function buildPipeline(steps) {
  return function (initial) {
    return steps.reduce(
      (chain, step) => chain.then((data) => step(data)),
      Promise.resolve(initial)
    );
  };
}

const pipeline = buildPipeline([
  (data) => validate(data),
  (data) => transform(data),
  (data) => enrich(data),
  (data) => persist(data),
]);

pipeline(input)
  .then((result) => console.log("Pipeline complete:", result))
  .catch((err) => console.error("Pipeline failed:", err));

// Example 2: Retry chain
function retry(fn, times) {
  return fn().catch((err) => {
    if (times <= 1) throw err;
    console.log(`Retrying... (${times - 1} left)`);
    return retry(fn, times - 1);
  });
}

// Example 3: Chain with async context passing
fetchUser(id)
  .then((user) => {
    return Promise.all([user, fetchPosts(user.id)]);
  })
  .then(([user, posts]) => {
    return Promise.all([user, posts, fetchFriends(user.id)]);
  })
  .then(([user, posts, friends]) => {
    return renderDashboard(user, posts, friends);
  })
  .catch(handleError);

// Example 4: Sequential array processing with reduce
const asyncOps = [op1, op2, op3, op4, op5];

asyncOps
  .reduce((chain, op) => {
    return chain.then((results) => {
      return op().then((result) => [...results, result]);
    });
  }, Promise.resolve([]))
  .then((allResults) => console.log(allResults));
```

### Real-World Use Cases

- **Express async middleware** — Chain of middleware functions.
- **Knex.js query builder** — `.select().where().orderBy()` returns promises.
- **AWS SDK** — Chaining `.promise()` calls for async operations.
- **GraphQL resolvers** — Resolving data from multiple sources in sequence.
- **File processing** — Read → transform → write → email pipeline.

```javascript
// File processing chain
const fs = require("fs").promises;

fs.readFile("input.txt", "utf8")
  .then((content) => content.toUpperCase())
  .then((content) => fs.writeFile("output.txt", content))
  .then(() => console.log("Processing complete"))
  .catch((err) => console.error("Processing failed:", err));
```

### Common Mistakes

- **Not returning the promise from then** — Breaks the chain.
- **Nesting promises instead of chaining** — Creates callback hell.
- **Throwing non-Error values** — Makes debugging harder.
- **Not handling errors in the first then's rejection handler** — Use catch instead.
- **Forgetting that async functions return promises** — Mixing sync return with async.

### Best Practices

- Always `return` promises from within `.then()` handlers.
- Prefer `catch` at the end over the second argument to `then`.
- Use async/await for complex chains (they are syntactic sugar over promises).
- Keep chains short — extract reusable functions.
- Name your promise variables (`userPromise`, `postsPromise`) for readability.

### Performance Considerations

- Each `.then()` adds microtasks — chains are efficient.
- Long chains (50+ steps) may cause microtask queue buildup.
- Intermediate values (like in the `[user, posts]` pattern) consume memory.
- Flat chains are easier for engines to optimize than nested promise trees.
- Using async/await compiles to promise chains with identical performance.

### Interview Questions

1. **How does promise chaining differ from callback nesting?**
   Chaining is flat and linear, with automatic error propagation. Nesting creates pyramids with manual error handling at each level.

2. **What happens if you return a promise from within a `.then()`?**
   The outer promise adopts the inner promise's state, effectively flattening it.

3. **How does error propagation work in a chain?**
   If any step rejects or throws, the chain skips subsequent `.then()` handlers until a `.catch()` is found.

4. **Can you branch a promise chain?**
   Yes, by storing intermediate promises in variables — `const a = p.then(...); const b = p.then(...);`

### Coding Challenges

1. **Implement a `chain` function that sequences async operations.**
2. **Build a `pipeline` utility similar to compose but for async operations.**
3. **Create a function that processes an array sequentially using promise chaining.**
4. **Implement a retry mechanism in a promise chain.**
5. **Write a debounce function that returns a chainable promise.**

### Related Topics

- Async/await
- Promise composition
- Control flow
- Sequential vs parallel execution
- `reduce` with promises
