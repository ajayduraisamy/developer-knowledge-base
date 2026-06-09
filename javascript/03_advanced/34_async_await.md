# Async/Await

## Introduction

Async/await is syntactic sugar built on top of Promises, introduced in ES2017 (ES8). It allows asynchronous code to be written in a synchronous style, making it more readable and maintainable. The `async` keyword declares a function that returns a Promise, and the `await` keyword pauses execution until a Promise settles, without blocking the main thread.

---

## async function

### What It Is

An `async function` is a function declared with the `async` keyword. It always returns a Promise. If the function returns a non-promise value, it is automatically wrapped in a resolved Promise. If it throws, the returned Promise rejects with the thrown value. Inside an async function, the `await` keyword can be used to pause execution on Promise settlement.

```javascript
async function greet(name) {
  return `Hello, ${name}!`;
}

const result = greet("Alice");
console.log(result); // Promise { <fulfilled>: "Hello, Alice!" }
result.then(console.log); // "Hello, Alice!"
```

### Why It Is Important

Async functions solve several problems with raw Promises:
- **Readability** — Sequential async code looks synchronous.
- **Error handling** — Standard `try/catch` works instead of `.catch()`.
- **Debugging** — Stack traces are clearer than promise chains.
- **Control flow** — Loops and conditionals work naturally.
- **Composability** — Async functions can call other async functions directly.

### How It Works Internally

The JavaScript engine transforms async functions into state machines (similar to generators). Each `await` expression creates a suspension point. The engine:

1. Wraps the function body in a Promise.
2. At each `await`, it creates a `.then()` callback to resume execution.
3. If the awaited promise rejects, control jumps to the nearest `catch`.
4. The final return value resolves the outer promise.

V8 and other engines implement this as a compiler transformation that creates an implicit generator-like state machine.

### Syntax

```javascript
// Async function declaration
async function fetchData() {
  return await somePromise;
}

// Async arrow function
const fetchData = async () => {
  return await somePromise;
};

// Async function expression
const fetchData = async function () {
  return await somePromise;
};

// Async method
const obj = {
  async fetchData() {
    return await somePromise;
  },
};

// Async class method
class MyClass {
  async fetchData() {
    return await somePromise;
  }
}
```

### Beginner Examples

```javascript
// Example 1: Basic async function
async function getMessage() {
  return "Hello World";
}
getMessage().then(console.log); // "Hello World"

// Example 2: Async with await
function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function delayedGreeting() {
  await delay(1000);
  console.log("Hello after 1 second");
  await delay(1000);
  console.log("Hello after 2 seconds");
}

// Example 3: Fetch with async/await
async function getUser(id) {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();
  return data;
}

getUser(1).then(console.log).catch(console.error);
```

### Intermediate Examples

```javascript
// Example 1: Sequential vs parallel
async function sequential() {
  const user = await fetchUser(1);
  const posts = await fetchPosts(user.id); // waits for user
  return { user, posts };
}

async function parallel() {
  const userPromise = fetchUser(1);
  const postsPromise = fetchPosts(1);
  const [user, posts] = await Promise.all([userPromise, postsPromise]);
  return { user, posts };
}

// Example 2: Async with loops
async function processItems(items) {
  const results = [];
  for (const item of items) {
    const result = await processItem(item); // sequential
    results.push(result);
  }
  return results;
}

// Example 3: Async with array methods
async function getAllUsers(ids) {
  // Parallel with map
  const users = await Promise.all(
    ids.map((id) => fetch(`/api/users/${id}`).then((r) => r.json()))
  );
  return users;
}
```

### Advanced Examples

```javascript
// Example 1: Async function as a class method
class Database {
  constructor(connection) {
    this.connection = connection;
  }
  async query(sql, params) {
    const client = await this.connection.connect();
    try {
      const result = await client.query(sql, params);
      return result.rows;
    } finally {
      client.release();
    }
  }
}

// Example 2: Async IIFE
(async () => {
  const data = await fetch("/api/init");
  const config = await data.json();
  initializeApp(config);
})();

// Example 3: Async function with callback interop
function promisify(fn) {
  return function (...args) {
    return new Promise((resolve, reject) => {
      fn(...args, (err, result) => {
        if (err) reject(err);
        else resolve(result);
      });
    });
  };
}

const readFile = promisify(require("fs").readFile);

async function readConfig() {
  const content = await readFile("config.json", "utf8");
  return JSON.parse(content);
}
```

### Real-World Use Cases

- **Express async routes** — `app.get('/api', async (req, res) => { ... })`
- **React useEffect** — `useEffect(() => { async function load() { ... }; load(); }, [])`
- **Data loading hooks** — Custom React hooks that fetch data.
- **CLI tools** — Reading files, making HTTP requests, processing data.
- **Database migrations** — Sequential SQL execution with rollback.

```javascript
// Express async handler
app.get("/api/users/:id", async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) return res.status(404).json({ error: "Not found" });
    res.json(user);
  } catch (err) {
    next(err);
  }
});
```

### Common Mistakes

- **Forgetting that async functions return promises** — `const data = asyncFunc()` gives a Promise, not the value.
- **Not awaiting inside async function** — The promise runs but you lose the result.
- **Using await at the top level** — Not allowed in non-module scripts (use top-level await in modules).
- **Awaiting non-promise values** — Works but is unnecessary overhead.
- **Mixing .then() and await** — Choose one style and be consistent.

### Best Practices

- Always use try/catch for error handling in async functions.
- Return results, don't store in outer variables (side effects make code harder to reason about).
- Use `Promise.all` for parallel operations, not sequential awaits.
- Name async functions descriptively (e.g., `fetchUserData`, `loadConfig`).
- Use async/await over raw promise chains for readability.

### Performance Considerations

- Async functions have minimal overhead vs raw promises (state machine setup).
- Await is slightly slower than `.then()` due to the state machine entry/exit.
- The overhead is negligible for I/O-bound operations.
- Avoid `await` inside hot synchronous loops (use `Promise.all` with mapped promises).
- V8 optimizes async functions aggressively after a few calls (hot path optimization).

### Interview Questions

1. **What does the `async` keyword do?**
   It declares a function that always returns a Promise and allows the use of `await` inside it.

2. **Can you use `await` outside an `async` function?**
   Not in older JavaScript. ES2022 introduced top-level `await` in modules.

3. **What happens if an async function throws?**
   The returned Promise rejects with the thrown error.

4. **Is an async function synchronous or asynchronous?**
   It's synchronous up to the first `await`, then asynchronous.

### Coding Challenges

1. **Create a function that measures the execution time of an async function.**
2. **Implement `Promise.all` using async/await.**
3. **Write an async function that retries an operation N times with backoff.**
4. **Build an async `map` function for arrays with configurable concurrency.**
5. **Create a `sleep` function using async/await.**

### Related Topics

- Promises
- Await keyword
- Error handling
- Concurrency
- Top-level await

---

## await keyword

### What It Is

The `await` keyword can only be used inside `async` functions. It pauses the execution of the async function until the awaited Promise settles. If the Promise fulfills, `await` returns the fulfillment value. If the Promise rejects, `await` throws the rejection reason, which can be caught with try/catch.

```javascript
async function example() {
  const result = await somePromise; // pauses here
  console.log(result); // runs after promise settles
}
```

### Why It Is Important

The `await` keyword is what makes async/await more readable than raw promises:
- It eliminates explicit `.then()` chains.
- It allows natural control flow (loops, conditionals, try/catch).
- It makes async code look synchronous.
- It provides better debugging with clearer stack traces.
- It enables fine-grained control over serial vs parallel execution.

### How It Works Internally

When the engine encounters `await`:

1. It evaluates the expression (which may be a non-promise, which gets wrapped in `Promise.resolve()`).
2. It creates a `.then()` handler that will resume the function.
3. It suspends the function execution and returns control to the caller.
4. When the awaited promise settles, the `.then()` handler is queued as a microtask.
5. When the microtask runs, the function resumes from the `await` point.

The engine creates a "promise reaction" record that captures the function's continuation.

### Syntax

```javascript
// Await a promise
const value = await promise;

// Await a non-promise (auto-wrapped)
const value = await "hello"; // same as await Promise.resolve("hello")

// Await in expressions
const data = (await fetch(url)).json();

// Await in conditionals
if (await userExists(id)) {
  // ...
}

// Await in loops
for await (const item of asyncIterable) {
  // ...
}
```

### Beginner Examples

```javascript
// Example 1: Basic await
async function getWeather(city) {
  const response = await fetch(
    `https://api.weather.com/${city}`
  );
  const data = await response.json();
  return data.temperature;
}

// Example 2: Await with setTimeout wrapper
function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function countdown(seconds) {
  for (let i = seconds; i > 0; i--) {
    console.log(i);
    await delay(1000);
  }
  console.log("Go!");
}

// Example 3: Await with error propagation
async function getUser(id) {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return await response.json();
}
```

### Intermediate Examples

```javascript
// Example 1: Await in conditionals
async function getDashboardData(isAdmin) {
  const basic = await fetchBasicData();
  if (isAdmin) {
    const admin = await fetchAdminData();
    return { ...basic, ...admin };
  }
  return basic;
}

// Example 2: Await in loops with dependencies
async function buildDependencyGraph(modules) {
  const graph = {};
  for (const module of modules) {
    const dependencies = await resolveDependencies(module);
    graph[module] = dependencies;
    for (const dep of dependencies) {
      if (!graph[dep]) {
        const subDeps = await resolveDependencies(dep);
        graph[dep] = subDeps;
      }
    }
  }
  return graph;
}

// Example 3: Await with Promise.all for controlled parallelism
async function loadProfile(userId) {
  const [user, posts, friends] = await Promise.all([
    fetchUser(userId),
    fetchPosts(userId),
    fetchFriends(userId),
  ]);
  return { user, posts, friends };
}
```

### Advanced Examples

```javascript
// Example 1: Dynamic await
async function processItem(item, processors) {
  for (const processor of processors) {
    if (await processor.canHandle(item)) {
      return await processor.process(item);
    }
  }
  throw new Error("No suitable processor found");
}

// Example 2: Await with AbortController
async function fetchWithTimeout(url, signal, timeoutMs = 5000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeoutMs);

  try {
    const response = await fetch(url, {
      signal: signal || controller.signal,
    });
    return await response.json();
  } finally {
    clearTimeout(timeoutId);
  }
}

// Example 3: Recursive await
async function fetchPaginated(url, page = 1, accumulator = []) {
  const response = await fetch(`${url}?page=${page}`);
  const data = await response.json();
  const allData = [...accumulator, ...data.items];

  if (data.hasMore) {
    return fetchPaginated(url, page + 1, allData);
  }
  return allData;
}
```

### Real-World Use Cases

- **Data fetching** — Loading data from multiple API endpoints.
- **File I/O** — Reading and writing files sequentially.
- **Database transactions** — Steps that depend on each other.
- **Form validation** — Async validation (checking username availability).
- **Image processing** — Loading, transforming, and saving images.

### Common Mistakes

- **Awaiting in series when parallel is possible** — Unnecessarily slow.
- **Not awaiting inside loops** — Promises fire but results are lost.
- **Awaiting non-promise values** — Works but confuses intent.
- **Forgetting to wrap in try/catch** — Unhandled rejections.
- **Using await in non-async function** — SyntaxError.

### Best Practices

- Only `await` when you need the result before proceeding.
- Use `Promise.all` for independent operations.
- Keep `await` expressions simple — complex expressions are hard to debug.
- Always handle rejected promises (try/catch or .catch).
- Use `await` in loops with `for...of`, not `forEach` (forEach doesn't handle promises).

### Performance Considerations

- Each `await` creates a microtask boundary — minimal overhead.
- Sequential `await` in loops is slow — use `Promise.all` when order doesn't matter.
- V8 can optimize away the promise wrapper for simple `await` expressions.
- Excessive `await` in hot synchronous code adds unnecessary microtask overhead.

### Interview Questions

1. **What does `await` do exactly?**
   It pauses execution of the async function until the promise settles, then returns the fulfillment value or throws the rejection reason.

2. **Can you `await` a non-Promise value?**
   Yes. It's wrapped in `Promise.resolve()` and immediately resumed as a microtask.

3. **What happens if you forget `await` on a promise?**
   You get a Promise object instead of the resolved value.

4. **How does `await` affect the event loop?**
   It yields control back to the event loop, allowing other tasks to run between await expressions.

### Coding Challenges

1. **Create an async `sleep` function.**
2. **Implement a `retry` utility that uses await.**
3. **Build an async `map` that limits concurrency with await.**
4. **Write a function that measures await time vs promise resolution time.**

### Related Topics

- async function
- Promises
- Event loop
- Concurrency
- `for await...of`

---

## Error handling with try/catch

### What It Is

Async/await enables error handling using the familiar `try/catch` syntax instead of `.catch()` chains. Any rejected promise that is `await`ed will throw its rejection reason, which can be caught by a surrounding `try/catch` block. This allows grouping error handling logic for multiple async operations.

```javascript
async function fetchUserData(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return await response.json();
  } catch (err) {
    console.error("Failed to fetch user:", err.message);
    throw err; // re-throw if caller should handle it
  }
}
```

### Why It Is Important

Centralized error handling with try/catch:
- Makes error paths explicit and readable.
- Allows handling errors at the appropriate level of abstraction.
- Enables error recovery and fallback logic.
- Provides clear stack traces with async context.
- Integrates naturally with synchronous error handling patterns.

### How It Works Internally

When `await` encounters a rejected promise, it throws the rejection reason. The engine walks up the call stack (within the async function) to find a matching `catch` block. If no catch is found within the async function, the async function's returned Promise rejects with the error.

The engine uses the same exception handling mechanism as synchronous code, which means:
- Errors are propagated synchronously until the next `await`.
- Multiple `catch` blocks can be used for different error types.
- `finally` blocks always execute for cleanup.

### Syntax

```javascript
// Basic try/catch
async function example() {
  try {
    const result = await riskyOperation();
    return result;
  } catch (error) {
    console.error("Operation failed:", error);
    throw error; // re-throw or return fallback
  }
}

// try/catch/finally
async function example() {
  const resource = await acquireResource();
  try {
    const result = await useResource(resource);
    return result;
  } catch (error) {
    await handleError(error);
    throw error;
  } finally {
    await resource.release();
  }
}

// Multiple catch blocks (via error type checking)
async function example() {
  try {
    return await riskyOp();
  } catch (error) {
    if (error instanceof ValidationError) {
      return handleValidation(error);
    }
    if (error instanceof NetworkError) {
      return handleNetwork(error);
    }
    throw error; // Unknown error, re-throw
  }
}
```

### Beginner Examples

```javascript
// Example 1: Basic try/catch
async function loadConfig() {
  try {
    const response = await fetch("/config.json");
    return await response.json();
  } catch (err) {
    console.error("Config load failed:", err.message);
    return { theme: "default" }; // fallback
  }
}

// Example 2: try/catch with finally
async function saveData(data) {
  showSpinner();
  try {
    const response = await fetch("/api/save", {
      method: "POST",
      body: JSON.stringify(data),
    });
    if (!response.ok) throw new Error("Save failed");
    return await response.json();
  } catch (err) {
    showError(err.message);
    throw err;
  } finally {
    hideSpinner();
  }
}

// Example 3: Catching validation errors
async function registerUser(userData) {
  try {
    const response = await fetch("/api/register", {
      method: "POST",
      body: JSON.stringify(userData),
    });
    const data = await response.json();
    if (!response.ok) throw new Error(data.error);
    return data;
  } catch (err) {
    console.error("Registration failed:", err.message);
    return null;
  }
}
```

### Intermediate Examples

```javascript
// Example 1: Error classification
class AppError extends Error {
  constructor(message, code, statusCode) {
    super(message);
    this.code = code;
    this.statusCode = statusCode;
  }
}

async function apiGet(path) {
  const response = await fetch(path);
  if (response.status === 401) throw new AppError("Unauthorized", "AUTH_FAILED", 401);
  if (response.status === 404) throw new AppError("Not found", "NOT_FOUND", 404);
  if (!response.ok) throw new AppError("Server error", "SERVER_ERROR", response.status);
  return response.json();
}

async function fetchUserProfile(id) {
  try {
    return await apiGet(`/api/users/${id}`);
  } catch (err) {
    if (err instanceof AppError) {
      if (err.code === "NOT_FOUND") return null;
      if (err.code === "AUTH_FAILED") {
        await refreshToken();
        return apiGet(`/api/users/${id}`);
      }
    }
    throw err;
  }
}

// Example 2: Wrapping try/catch into helper
async function safeAsync(asyncFn, fallback) {
  try {
    return await asyncFn();
  } catch (err) {
    console.warn("Async operation failed, using fallback:", err.message);
    return typeof fallback === "function" ? fallback(err) : fallback;
  }
}

const user = await safeAsync(
  () => fetchUser(1),
  { id: 0, name: "Guest" }
);

// Example 3: Batch error handling
async function loadDashboard() {
  const errors = [];
  const results = {};

  for (const [key, promise] of Object.entries({
    user: fetchUser(),
    posts: fetchPosts(),
    friends: fetchFriends(),
  })) {
    try {
      results[key] = await promise;
    } catch (err) {
      errors.push({ key, error: err.message });
      results[key] = null;
    }
  }

  if (errors.length > 0) {
    console.warn("Partial dashboard load:", errors);
  }
  return results;
}
```

### Advanced Examples

```javascript
// Example 1: Retry with exponential backoff
async function fetchWithRetry(url, options = {}) {
  const { maxRetries = 3, baseDelay = 1000 } = options;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      return await response.json();
    } catch (err) {
      if (attempt === maxRetries) throw err;
      console.warn(`Attempt ${attempt} failed, retrying...`);
      await new Promise((r) =>
        setTimeout(r, baseDelay * Math.pow(2, attempt - 1))
      );
    }
  }
}

// Example 2: Circuit breaker pattern
class CircuitBreaker {
  constructor(fn, options = {}) {
    this.fn = fn;
    this.failureCount = 0;
    this.threshold = options.threshold || 3;
    this.timeout = options.timeout || 10000;
    this.state = "CLOSED";
  }

  async call(...args) {
    if (this.state === "OPEN") {
      if (Date.now() - this.lastFailure >= this.timeout) {
        this.state = "HALF_OPEN";
      } else {
        throw new Error("Circuit breaker is OPEN");
      }
    }

    try {
      const result = await this.fn(...args);
      if (this.state === "HALF_OPEN") {
        this.state = "CLOSED";
        this.failureCount = 0;
      }
      return result;
    } catch (err) {
      this.failureCount++;
      this.lastFailure = Date.now();
      if (this.failureCount >= this.threshold) {
        this.state = "OPEN";
      }
      throw err;
    }
  }
}

// Example 3: Global error boundary
async function errorBoundary(asyncFn, context) {
  try {
    return { data: await asyncFn(), error: null };
  } catch (err) {
    console.error(`Error in ${context}:`, err);
    reportError(err, context);
    return { data: null, error: err };
  }
}
```

### Real-World Use Cases

- **React Query/useQuery** — Built-in error handling for data fetching.
- **Express async middleware** — Wrapping route handlers to catch errors.
- **Sentry error tracking** — Reporting caught errors with context.
- **Form submission** — Handling network errors during form save.
- **File processing** — Cleaning up resources in finally blocks.

### Common Mistakes

- **Catching errors but not handling them** — Empty catch blocks swallow errors.
- **Forgetting to re-throw after logging** — Downstream code doesn't know about failures.
- **Using try/catch around every await** — Too granular; catch at a higher level.
- **Not using finally for cleanup** — Resources leak on errors.
- **Catching non-Error values** — Strings or objects thrown lose stack traces.

### Best Practices

- Catch errors at the level where you can meaningfully handle them.
- Re-throw errors you can't handle (with `throw err`).
- Use `finally` for cleanup regardless of success or failure.
- Extend `Error` for custom error types with additional properties.
- Log errors with sufficient context for debugging.
- Don't use try/catch for control flow — avoid throwing for non-error conditions.

### Performance Considerations

- try/catch is optimized by V8 and has negligible overhead when no error is thrown.
- Throwing errors is expensive (stack trace construction) — don't use exceptions for regular control flow.
- Deep async stacks with try/catch are resolved efficiently.
- Error objects with stack traces consume memory proportional to stack depth.

### Interview Questions

1. **How does error handling with async/await differ from promise chains?**
   async/await uses try/catch (familiar and synchronous-looking). Promise chains use .catch() method chaining.

2. **Can you use try/catch around multiple awaits?**
   Yes. A single try/catch can wrap multiple await expressions, catching the first rejection.

3. **What is the difference between not catching and catching in an async function?**
   Uncaught rejections cause the returned promise to reject. Caught errors are handled locally.

4. **How do errors propagate in nested async functions?**
   Errors propagate up the async call stack just like synchronous exceptions, until caught by a try/catch or reaching the top level.

### Coding Challenges

1. **Create an `assert` function for async preconditions.**
2. **Build a `withRetry` wrapper that uses try/catch internally.**
3. **Implement a `timeout` function using try/catch with AbortController.**
4. **Write a utility that converts any async function to return `[data, error]` tuples.**
5. **Create a circuit breaker with automatic recovery.**

### Related Topics

- Promises (.catch)
- Error objects
- Custom error types
- Error logging
- Defensive programming

---

## Concurrent async operations

### What It Is

Concurrent async operations involve running multiple asynchronous tasks simultaneously and coordinating their results. JavaScript provides several patterns: `Promise.all` for wait-for-all, `Promise.race` for first-settled, `Promise.allSettled` for wait-for-all-with-results, and `Promise.any` for first-fulfilled. async/await makes these patterns clean and readable.

```javascript
async function loadAll() {
  const [users, posts, comments] = await Promise.all([
    fetch("/api/users").then((r) => r.json()),
    fetch("/api/posts").then((r) => r.json()),
    fetch("/api/comments").then((r) => r.json()),
  ]);
  return { users, posts, comments };
}
```

### Why It Is Important

Concurrency is critical for performance in I/O-bound applications:
- Fetching multiple resources in parallel is much faster than sequentially.
- Coordinating results from multiple sources is a common requirement.
- Different patterns suit different needs (all, race, any, allSettled).
- Understanding concurrency prevents accidental serialization bottlenecks.
- Proper concurrency control prevents resource exhaustion.

### How It Works Internally

When using `Promise.all`, all promises are started immediately (they are created synchronously). The engine schedules their `.then()` handlers as microtasks. When all have fulfilled, `Promise.all` resolves with the results array. `Promise.race` resolves/rejects with the first settlement. The promises themselves run concurrently via the event loop — the engine doesn't create additional threads, but I/O operations are non-blocking.

### Syntax

```javascript
// Parallel execution
const [a, b, c] = await Promise.all([p1, p2, p3]);

// Race
const first = await Promise.race([p1, p2]);

// All settled
const results = await Promise.allSettled([p1, p2]);

// Any (first fulfilled)
const firstSuccess = await Promise.any([p1, p2]);
```

### Beginner Examples

```javascript
// Example 1: Parallel fetches
async function getUsers(ids) {
  const promises = ids.map((id) =>
    fetch(`/api/users/${id}`).then((r) => r.json())
  );
  return Promise.all(promises);
}

// Example 2: Race between fetch and timeout
async function fetchWithTimeout(url, ms) {
  const result = await Promise.race([
    fetch(url).then((r) => r.json()),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error("Timeout")), ms)
    ),
  ]);
  return result;
}

// Example 3: Sequential vs parallel comparison
async function sequential(ids) {
  const results = [];
  for (const id of ids) {
    results.push(await fetchUser(id));
  }
  return results;
}

async function parallel(ids) {
  return Promise.all(ids.map((id) => fetchUser(id)));
}
// parallel is significantly faster for I/O-bound operations
```

### Intermediate Examples

```javascript
// Example 1: Concurrency limited
async function concurrencyLimited(tasks, limit) {
  const results = [];
  const executing = new Set();

  for (const task of tasks) {
    const promise = task().then((result) => {
      executing.delete(promise);
      return result;
    });
    results.push(promise);
    executing.add(promise);

    if (executing.size >= limit) {
      await Promise.race(executing);
    }
  }

  return Promise.all(results);
}

const tasks = urls.map((url) => () => fetch(url).then((r) => r.json()));
const data = await concurrencyLimited(tasks, 3);

// Example 2: Promise.allSettled with async/await
async function loadCriticalAndNonCritical() {
  const critical = await Promise.all([
    fetchUser(),
    fetchPosts(),
  ]).catch((err) => {
    throw new Error("Critical data failed: " + err.message);
  });

  // Non-critical can fail partially
  const nonCritical = await Promise.allSettled([
    fetchRecommendations(),
    fetchAds(),
    fetchTrending(),
  ]);

  const extra = nonCritical
    .filter((r) => r.status === "fulfilled")
    .map((r) => r.value);

  return { critical, extra };
}

// Example 3: Batching with concurrency
async function processBatch(items, batchSize, processor) {
  const results = [];
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map((item) => processor(item))
    );
    results.push(...batchResults);
  }
  return results;
}
```

### Advanced Examples

```javascript
// Example 1: Concurrent queue with priorities
class PriorityQueue {
  constructor(concurrency = 3) {
    this.concurrency = concurrency;
    this.running = 0;
    this.queue = [];
  }

  async add(task, priority = 0) {
    return new Promise((resolve, reject) => {
      this.queue.push({ task, resolve, reject, priority });
      this.queue.sort((a, b) => b.priority - a.priority);
      this._process();
    });
  }

  async _process() {
    if (this.running >= this.concurrency || this.queue.length === 0) return;
    this.running++;
    const { task, resolve, reject } = this.queue.shift();
    try {
      resolve(await task());
    } catch (err) {
      reject(err);
    } finally {
      this.running--;
      this._process();
    }
  }
}

// Example 2: Async pool with dynamic scheduling
async function* asyncPool(tasks, concurrency) {
  const results = [];
  const executing = new Set();

  for (const [index, task] of tasks.entries()) {
    const promise = Promise.resolve().then(() => task());
    results[index] = promise;
    executing.add(promise);

    promise.finally(() => executing.delete(promise));

    if (executing.size >= concurrency) {
      await Promise.race(executing);
    }
  }

  for (const result of results) {
    yield await result;
  }
}

// Usage
for await (const data of asyncPool(tasks, 3)) {
  process(data);
}

// Example 3: Abortable concurrent operations
async function fetchWithAbort(urls, signal) {
  const promises = urls.map((url) =>
    fetch(url, { signal }).then((r) => r.json())
  );

  try {
    return await Promise.all(promises);
  } catch (err) {
    if (err.name === "AbortError") {
      console.log("Fetches were aborted");
    }
    throw err;
  }
}
```

### Real-World Use Cases

- **Microservice aggregation** — Combine results from multiple backend services.
- **Image gallery** — Load multiple images in parallel.
- **Dashboard widgets** — Each widget fetches independently.
- **Bulk database operations** — Insert/update many records.
- **Web scraping** — Scrape multiple pages concurrently with limits.

### Common Mistakes

- **Sequential awaits when parallel is possible** — The most common performance mistake.
- **No concurrency limit** — Thousands of parallel requests overwhelm the network/database.
- **Ignoring errors in Promise.all** — One failure loses all results.
- **Using forEach with async/await** — forEach doesn't await promises.
- **Assuming Promise.race cancels losers** — All promises run to completion.

### Best Practices

- Default to parallel with `Promise.all` for independent operations.
- Always limit concurrency when dealing with external resources.
- Use `Promise.allSettled` when you need partial results.
- Use `Promise.any` for fallback strategies.
- Consider using async iterators for streaming concurrent results.
- Use AbortController to cancel in-flight requests when no longer needed.

### Performance Considerations

- Parallel execution is I/O-bound and uses negligible CPU for waiting.
- Too many concurrent operations can exhaust file descriptors or database connections.
- `Promise.all` has O(1) overhead per promise — fine for hundreds, consider batching for thousands.
- Memory usage scales with the number of concurrent promises (each keeps its result).
- V8 limits microtask queue depth; extremely deep concurrency can cause backpressure issues.

### Interview Questions

1. **How do you run multiple async operations in parallel?**
   Create all promises first, then use `await Promise.all([...])`.

2. **What's the difference between sequential and concurrent execution?**
   Sequential runs one at a time, each waiting for the previous. Concurrent starts all at once and waits for all to finish.

3. **How do you limit concurrency?**
   Use a semaphore pattern — track running tasks and wait when hitting the limit.

4. **Can you cancel concurrent operations?**
   Not directly. Use `AbortController` with fetch, or ignore results of operations that are no longer needed.

### Coding Challenges

1. **Implement `Promise.all` with async/await.**
2. **Build a concurrency limiter that runs N tasks at a time.**
3. **Create an async pool that yields results as they complete.**
4. **Implement a rate limiter for concurrent API calls.**
5. **Write a function that batches large arrays for parallel processing.**

### Related Topics

- Promise combinators (all, race, allSettled, any)
- Event loop
- Microtask scheduling
- AbortController
- Async iterators
