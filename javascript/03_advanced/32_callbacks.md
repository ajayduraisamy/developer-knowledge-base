# Callbacks

## Introduction

Callbacks are functions passed as arguments to other functions, to be executed later when a certain event occurs or an asynchronous operation completes. They are the foundation of asynchronous programming in JavaScript, predating promises and async/await. While modern JavaScript offers newer patterns, callbacks remain essential for understanding the event-driven nature of the language and are still widely used in Node.js APIs, browser events, and utility functions.

---

## Callback functions

### What It Is

A callback function is a function that is passed as an argument to another function and is invoked by that function at a specific time, typically after some operation completes or when an event occurs. Callbacks can be synchronous (executed immediately within the outer function) or asynchronous (scheduled for later execution).

```javascript
// Synchronous callback
function forEach(arr, callback) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i, arr);
  }
}

// Asynchronous callback
function fetchData(url, callback) {
  const xhr = new XMLHttpRequest();
  xhr.open("GET", url);
  xhr.onload = function () {
    callback(null, xhr.responseText);
  };
  xhr.onerror = function () {
    callback(new Error("Request failed"));
  };
  xhr.send();
}
```

### Why It Is Important

Callbacks are fundamental to JavaScript because:
- JavaScript is single-threaded and event-driven — callbacks enable non-blocking I/O.
- They provide a way to handle continuations — what should happen after an operation.
- They are the basis for all higher-order array methods (`map`, `filter`, `reduce`).
- Many Node.js APIs are built on the error-first callback pattern.
- Browser events (clicks, keypresses, timers) all use callbacks.

### How It Works Internally

When a callback is passed to a function, JavaScript passes a reference to the function object. The receiving function stores the reference and calls it at the appropriate time using the `()` operator. For asynchronous callbacks (like `setTimeout`), the callback is placed into a task queue and will be executed by the event loop when the call stack is empty.

Internally, the JavaScript engine treats callback functions as first-class objects:
1. A reference to the function is created and passed.
2. The host function stores the reference in a variable or closure.
3. When the trigger condition is met, the engine invokes the callback with the appropriate arguments.
4. A new execution context is created for the callback and pushed onto the call stack.

### Syntax

```javascript
// Basic callback syntax
function receiver(callback) {
  // Do some work
  callback(result); // Invoke callback
}

// Anonymous callback
receiver(function (result) {
  console.log(result);
});

// Named callback
function myCallback(result) {
  console.log(result);
}
receiver(myCallback);

// Arrow function callback
receiver((result) => console.log(result));

// Inline callback
receiver(console.log);
```

### Beginner Examples

```javascript
// Example 1: setTimeout callback
setTimeout(function () {
  console.log("This runs after 1 second");
}, 1000);

// Example 2: Array forEach callback
const numbers = [1, 2, 3, 4];
numbers.forEach(function (num, index) {
  console.log(`numbers[${index}] = ${num}`);
});

// Example 3: Event listener callback
const button = document.querySelector("button");
button.addEventListener("click", function (event) {
  console.log("Button clicked at:", event.clientX, event.clientY);
});

// Example 4: Simple synchronous callback
function greet(name, formatter) {
  return formatter(name);
}

function polite(name) {
  return `Dear ${name}`;
}

function casual(name) {
  return `Hey ${name}`;
}

console.log(greet("Alice", polite)); // "Dear Alice"
console.log(greet("Bob", casual));   // "Hey Bob"
```

### Intermediate Examples

```javascript
// Example 1: Array filter with callback
function filter(arr, predicate) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    if (predicate(arr[i], i, arr)) {
      result.push(arr[i]);
    }
  }
  return result;
}

const nums = [1, 2, 3, 4, 5, 6];
const evens = filter(nums, (n) => n % 2 === 0);
console.log(evens); // [2, 4, 6]

// Example 2: Custom timeout with data
function delayedSum(a, b, delay, callback) {
  setTimeout(function () {
    callback(a + b);
  }, delay);
}
delayedSum(5, 10, 500, function (sum) {
  console.log("Sum is:", sum);
});

// Example 3: Callback with multiple results
function getCoordinates(callback) {
  navigator.geolocation.getCurrentPosition(
    function (position) {
      callback(null, {
        lat: position.coords.latitude,
        lng: position.coords.longitude,
      });
    },
    function (error) {
      callback(error);
    }
  );
}

// Example 4: Transforming data with callbacks
function processUserData(userData, transform, callback) {
  try {
    const transformed = transform(userData);
    callback(null, transformed);
  } catch (err) {
    callback(err);
  }
}
```

### Advanced Examples

```javascript
// Example 1: Composable callbacks (pipe)
function pipe(...fns) {
  return function (initial, callback) {
    let result = initial;
    for (const fn of fns) {
      result = fn(result);
    }
    callback(result);
  };
}

const addTax = (price) => price * 1.2;
const addShipping = (price) => price + 10;
const formatPrice = (price) => `$${price.toFixed(2)}`;

const calculateFinalPrice = pipe(addTax, addShipping, formatPrice);
calculateFinalPrice(100, console.log); // "$130.00"

// Example 2: Throttled callback
function throttle(callback, limit) {
  let waiting = false;
  return function (...args) {
    if (!waiting) {
      callback.apply(this, args);
      waiting = true;
      setTimeout(() => {
        waiting = false;
      }, limit);
    }
  };
}

window.addEventListener("scroll", throttle(function () {
  console.log("Scrolled at:", window.scrollY);
}, 200));

// Example 3: Once callback
function once(callback) {
  let called = false;
  return function (...args) {
    if (called) return;
    called = true;
    return callback.apply(this, args);
  };
}

const initialize = once(function () {
  console.log("Initialized!");
});
initialize(); // "Initialized!"
initialize(); // Nothing (only runs once)
```

### Real-World Use Cases

- **Node.js file system** — `fs.readFile(path, callback)` for async file I/O.
- **Express middleware** — Middleware functions are callbacks with `(req, res, next)`.
- **Database drivers** — MongoDB, Redis, and PostgreSQL clients use callbacks.
- **Animation frames** — `requestAnimationFrame(callback)` for smooth animations.
- **Streams** — Node.js streams use `data`, `end`, `error` event callbacks.

```javascript
// Node.js file read example
const fs = require("fs");
fs.readFile("/path/to/file.txt", "utf8", function (err, data) {
  if (err) {
    console.error("Error reading file:", err);
    return;
  }
  console.log("File content:", data);
});
```

### Common Mistakes

- **Not handling errors within callbacks** — Uncaught errors crash the process.
- **Calling callbacks multiple times** — Leads to duplicate side effects.
- **Forgetting to pass callbacks** — Causes "callback is not a function" errors.
- **Zombie callbacks** — Callbacks that fire after the component is unmounted.
- **Nesting callbacks too deeply** — Leads to callback hell (see Callback hell section).

### Best Practices

- Always check that the callback is a function before calling it (defensive check).
- Call callbacks exactly once (unless document specifically says otherwise).
- Use the error-first pattern for async callbacks consistently.
- Clean up callbacks when they are no longer needed (remove event listeners).
- Use named functions instead of anonymous ones for better stack traces.
- Throttle/debounce callbacks that fire frequently (scroll, resize).

### Performance Considerations

- Creating anonymous functions in loops creates many function objects — prefer named functions or declare once.
- Callbacks that are GC'd after use (like `setTimeout` callbacks) are generally low overhead.
- Event listener callbacks keep their closure scope alive — be mindful of memory.
- Deeply nested callbacks can cause stack overflow if synchronous.
- Each callback invocation adds a frame to the call stack.

### Interview Questions

1. **What is a callback function?**
   A function passed as an argument to another function to be executed later.

2. **What is the difference between synchronous and asynchronous callbacks?**
   Synchronous callbacks execute immediately within the outer function. Asynchronous callbacks are scheduled for later execution on the event loop.

3. **How do you prevent a callback from being called twice?**
   Wrap the callback with a `once()` utility that tracks whether it has been called.

4. **What is the error-first callback pattern?**
   The first argument of the callback is reserved for an error object (null if no error), followed by the result data.

5. **How do you convert a callback-based function to return a promise?**
   Wrap the function call in `new Promise((resolve, reject) => { ... })`.

### Coding Challenges

1. **Implement `map` using a callback pattern.**
2. **Create a `debounce` function that delays invoking a callback.**
3. **Implement a simple Pub/Sub system where subscribers register callbacks.**
4. **Write a function `timeout` that wraps a callback to fail if it takes too long.**
5. **Build a `retry` function that retries a callback-based operation N times.**

### Related Topics

- Higher-order functions
- Event loop
- Asynchronous programming
- Promises
- Async/await
- Observables (RxJS)

---

## Higher-order functions

### What It Is

A higher-order function is a function that does at least one of the following: takes one or more functions as arguments, or returns a function as its result. In JavaScript, functions are first-class citizens, meaning they can be treated like any other value — assigned to variables, passed as arguments, and returned from other functions.

```javascript
// Higher-order function taking a function argument
function operateOnArray(arr, operation) {
  const result = [];
  for (const item of arr) {
    result.push(operation(item));
  }
  return result;
}

// Higher-order function returning a function
function multiplier(factor) {
  return function (value) {
    return value * factor;
  };
}
```

### Why It Is Important

Higher-order functions are a cornerstone of functional programming in JavaScript:
- They enable **abstraction** — general-purpose functions with specific behavior injected.
- They allow **composition** — building complex operations from simple functions.
- They are the basis for **decorators** and **middleware** patterns.
- They enable **declarative** code (what to do) instead of **imperative** (how to do it).
- They reduce code duplication by extracting common patterns.

### How It Works Internally

When a higher-order function receives a function as an argument, the engine stores a reference to that function. When it returns a function, it creates a new function object (and potentially a closure). The engine treats these operations like any other value passing:

1. Function references are stored in variables/parameters.
2. `typeof fn === "function"` can be checked.
3. The returned function has its own `[[Environment]]` (closure).
4. V8 and other engines can inline small functions passed as arguments in optimized code.

### Syntax

```javascript
// Function as argument
function hof(fn, ...args) {
  return fn(...args);
}

// Function as return value
function hof() {
  return function () {
    // ...
  };
}

// Both
function hof(fn) {
  return function (...args) {
    return fn(...args);
  };
}
```

### Beginner Examples

```javascript
// Example 1: setTimeout (built-in HOF)
setTimeout(() => console.log("Hello"), 1000);

// Example 2: Array methods
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((n) => n * 2);
const evens = numbers.filter((n) => n % 2 === 0);
const sum = numbers.reduce((acc, n) => acc + n, 0);

// Example 3: Custom HOF
function withLogging(fn) {
  return function (...args) {
    console.log(`Calling with:`, args);
    const result = fn(...args);
    console.log(`Result:`, result);
    return result;
  };
}

const add = (a, b) => a + b;
const loggedAdd = withLogging(add);
loggedAdd(3, 4); // Logs: Calling with: [3, 4], Result: 7
```

### Intermediate Examples

```javascript
// Example 1: Memoize HOF
function memoize(fn) {
  const cache = new Map();
  return function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// Example 2: Negate predicate HOF
function negate(predicate) {
  return function (...args) {
    return !predicate(...args);
  };
}

const isEven = (n) => n % 2 === 0;
const isOdd = negate(isEven);
console.log([1, 2, 3, 4, 5].filter(isOdd)); // [1, 3, 5]

// Example 3: Compose HOF
function compose(...fns) {
  return function (initial) {
    return fns.reduceRight((acc, fn) => fn(acc), initial);
  };
}

const trim = (s) => s.trim();
const capitalize = (s) => s[0].toUpperCase() + s.slice(1);
const exclaim = (s) => s + "!";

const formatMessage = compose(exclaim, capitalize, trim);
console.log(formatMessage("  hello ")); // "Hello!"
```

### Advanced Examples

```javascript
// Example 1: Async HOF with retry
function withRetry(fn, maxRetries = 3, delay = 1000) {
  return async function (...args) {
    let lastError;
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        return await fn(...args);
      } catch (err) {
        lastError = err;
        if (attempt < maxRetries) {
          await new Promise((r) => setTimeout(r, delay * attempt));
        }
      }
    }
    throw lastError;
  };
}

// Example 2: Middleware pipeline HOF
function createMiddlewarePipeline(middlewares) {
  return function (context) {
    let idx = 0;
    function next() {
      if (idx < middlewares.length) {
        const middleware = middlewares[idx++];
        middleware(context, next);
      }
    }
    next();
  };
}

// Example 3: Transducer-like HOF
function mapping(transform) {
  return function (reducer) {
    return function (acc, val) {
      return reducer(acc, transform(val));
    };
  };
}
```

### Real-World Use Cases

- **Redux middleware** — `applyMiddleware` is a HOF that composes middleware.
- **React higher-order components** — Functions that take a component and return an enhanced component.
- **Express middleware** — `app.use(middleware)` where middleware is a function.
- **Jest `describe`/`it`** — Tests are functions passed to test runner HOFs.
- **Lodash/Underscore** — Utility libraries are collections of HOFs.

```javascript
// React HOC example
function withLogging(WrappedComponent) {
  return function EnhancedComponent(props) {
    console.log(`Rendering ${WrappedComponent.name} with`, props);
    return <WrappedComponent {...props} />;
  };
}
```

### Common Mistakes

- **Passing a function call result instead of a function reference** — `setTimeout(foo(), 1000)` instead of `setTimeout(foo, 1000)`.
- **Losing `this` context** — When passing object methods as callbacks.
- **Creating closures unintentionally inside HOFs** — Can cause memory issues.
- **Over-abstraction** — Too many HOFs can make code hard to read.
- **Not handling variadic arguments** — Using `fn(...args)` vs `fn(arg)` inconsistently.

### Best Practices

- Prefer arrow functions for inline callbacks to avoid `this` confusion.
- Use descriptive names for HOFs (e.g., `withLogging`, `withRetry`).
- Keep HOFs small and focused on a single concern.
- Use TypeScript to type higher-order function signatures.
- Document what callbacks should receive and return.

### Performance Considerations

- Each HOF call creates new function wrappers — not an issue for typical usage.
- Inline callbacks in hot performance paths can prevent engine optimizations.
- Lodash/deeply optimized HOFs use lazy evaluation for performance.
- Compose chains should not be excessively long (more than 10 functions).
- The `arguments` object in HOFs is slower than rest parameters.

### Interview Questions

1. **What makes a function "higher-order"?**
   A function that takes a function as an argument and/or returns a function.

2. **What is the difference between a callback and a higher-order function?**
   A callback is the function passed as an argument; a higher-order function is the function that receives or returns it.

3. **How does `Array.prototype.reduce` serve as a HOF?**
   `reduce` takes a callback function (the reducer) and returns the accumulated value.

4. **Explain how `compose` works as a HOF.**
   `compose` takes multiple functions and returns a new function that applies them right-to-left.

### Coding Challenges

1. **Implement `Array.prototype.map` as a standalone HOF.**
2. **Create a `pipe` function that chains functions left-to-right.**
3. **Build a `compose` function that handles async functions.**
4. **Implement a `curry` HOF that transforms a function to be partially applied.**
5. **Create a `debounce` HOF that delays invocation.**

### Related Topics

- Callback functions
- Functional programming
- Function composition
- Closures
- First-class functions

---

## Callback hell

### What It Is

Callback hell, also known as "pyramid of doom," is a pattern of deeply nested callback functions that makes code difficult to read, maintain, and debug. It occurs when asynchronous operations depend on each other, requiring callbacks within callbacks, leading to excessive indentation and tangled control flow.

```javascript
// Classic callback hell
getUser(function (user) {
  getPosts(user.id, function (posts) {
    getComments(posts[0].id, function (comments) {
      getLikes(comments[0].id, function (likes) {
        console.log(likes);
      });
    });
  });
});
```

### Why It Is Important

Callback hell represents the failure point of the callback pattern for complex async workflows. Understanding it is important because:
- It motivates the need for better async patterns (Promises, async/await).
- It demonstrates why code structure matters for readability.
- It highlights the difficulty of error propagation in nested callbacks.
- It shows why maintainability degrades with async complexity.
- Most senior developers have experienced or inherited callback hell code.

### How It Works Internally

Callback hell is not a language feature but a code organization problem. Internally, each callback creates a new execution context when invoked. Deep nesting creates deep call stacks (though not simultaneously, since callbacks execute asynchronously). The real issue is:

1. **Control flow becomes non-linear** — difficult to trace the sequence of operations.
2. **Error handling is duplicated** — each level needs its own error handler.
3. **Variable shadowing** — inner callbacks can accidentally shadow outer variables.
4. **Code is T-shaped** — horizontal growth from nesting, vertical growth from steps.

### Syntax

The "syntax" of callback hell is characterized by:

```javascript
asyncOp1(function (err, result1) {
  if (err) handleError(err);
  asyncOp2(result1, function (err, result2) {
    if (err) handleError(err);
    asyncOp3(result2, function (err, result3) {
      if (err) handleError(err);
      asyncOp4(result3, function (err, result4) {
        if (err) handleError(err);
        // Do something with result4
      });
    });
  });
});
```

### Beginner Examples

```javascript
// Example 1: Three nested setTimeout
setTimeout(function () {
  console.log("Step 1");
  setTimeout(function () {
    console.log("Step 2");
    setTimeout(function () {
      console.log("Step 3");
    }, 1000);
  }, 1000);
}, 1000);

// Example 2: File operations (Node.js)
const fs = require("fs");
fs.readFile("a.txt", "utf8", function (err, dataA) {
  if (err) console.error(err);
  fs.writeFile("b.txt", dataA, function (err) {
    if (err) console.error(err);
    fs.readFile("c.txt", "utf8", function (err, dataC) {
      if (err) console.error(err);
      fs.writeFile("d.txt", dataA + dataC, function (err) {
        if (err) console.error(err);
        console.log("Done");
      });
    });
  });
});
```

### Intermediate Examples

```javascript
// Example 1: Authentication flow
function authenticate(username, password) {
  findUser(username, function (err, user) {
    if (err) return console.error(err);
    verifyPassword(password, user.hash, function (err, matched) {
      if (err) return console.error(err);
      if (!matched) return console.error("Wrong password");
      generateToken(user, function (err, token) {
        if (err) return console.error(err);
        saveSession(token, user.id, function (err) {
          if (err) return console.error(err);
          console.log("Authenticated successfully");
        });
      });
    });
  });
}

// Example 2: Web scraping with dependencies
request("https://site.com/page", function (err, html) {
  if (err) return console.error(err);
  const links = extractLinks(html);
  request(links[0], function (err, html2) {
    if (err) return console.error(err);
    const data = extractData(html2);
    saveToDatabase(data, function (err) {
      if (err) return console.error(err);
      request(links[1], function (err, html3) {
        // Deeper and deeper...
      });
    });
  });
});
```

### Advanced Examples

```javascript
// Example 1: Conditional branching with callbacks
function processOrder(orderId) {
  getOrder(orderId, function (err, order) {
    if (err) return handleError(err);
    if (order.status === "cancelled") {
      cancelPayment(order.paymentId, function (err) {
        if (err) return handleError(err);
        notifyUser("Order cancelled", function (err) {
          if (err) handleError(err);
        });
      });
    } else if (order.status === "shipped") {
      getTracking(order.id, function (err, tracking) {
        if (err) return handleError(err);
        sendTrackingEmail(order.userId, tracking, function (err) {
          if (err) handleError(err);
        });
      });
    } else {
      // More branches...
    }
  });
}

// Example 2: Parallel callbacks with shared state (callback hell hybrid)
function loadDashboard(userId) {
  let user, posts, notifications;
  getUser(userId, function (err, u) {
    if (err) return handleError(err);
    user = u;
    if (posts && notifications) renderDashboard(user, posts, notifications);
  });
  getPosts(userId, function (err, p) {
    if (err) return handleError(err);
    posts = p;
    if (user && notifications) renderDashboard(user, posts, notifications);
  });
  getNotifications(userId, function (err, n) {
    if (err) return handleError(err);
    notifications = n;
    if (user && posts) renderDashboard(user, posts, notifications);
  });
}
```

### Real-World Use Cases

- **Legacy Node.js codebases** (pre-2015) are full of callback hell.
- **Complex ETL pipelines** where each step depends on the previous.
- **Multi-step API orchestration** where endpoints are called sequentially.
- **File processing chains** (read → transform → write → read next).
- **Legacy browser code** with nested XHR requests.

### Common Mistakes

- **Not using Promises** when they are available.
- **Mixing synchronous and asynchronous callbacks** — leads to race conditions.
- **Forgetting `return` statements** — causes callbacks to execute twice.
- **Shadowing variables** across nested callbacks.
- **Not extracting named functions** — inline anonymous functions make debugging harder.

### Best Practices

- **Use Promises or async/await** — convert callbacks to Promises with `util.promisify`.
- **Name your callbacks** — use named functions to flatten the pyramid.
- **Extract logic into functions** — break large callback chains into smaller pieces.
- **Use control flow libraries** — `async.js` (Node.js) for parallel/series waterflow.
- **Flatten the structure** — early returns reduce nesting.

```javascript
// Flattened with named functions
getUser(function onUser(err, user) {
  if (err) return handleError(err);
  getPosts(user.id, onPosts);
});

function onPosts(err, posts) {
  if (err) return handleError(err);
  getComments(posts[0].id, onComments);
}

function onComments(err, comments) {
  if (err) return handleError(err);
  console.log(comments);
}
```

### Performance Considerations

- Deep nesting doesn't directly impact performance, but the code is harder to optimize by the engine.
- Callback hell often leads to duplicated error handlers — more code, more memory.
- Refactoring to Promises may introduce slight overhead from Promise object creation.
- The readability and maintainability benefits of flattening far outweigh minimal performance differences.

### Interview Questions

1. **What is callback hell and why is it a problem?**
   Deeply nested callbacks that make code unreadable and hard to maintain. Problems include poor readability, difficult error handling, and tangled control flow.

2. **How can you avoid callback hell?**
   Use Promises, async/await, extract named functions, flatten the structure, or use control flow libraries.

3. **What is the pyramid of doom?**
   The visual pattern of nested callbacks that creates a triangular indentation shape.

4. **How did developers handle async code before Promises?**
   Using libraries like `async.js` (waterfall, series, parallel) or manually flattening with named functions.

### Coding Challenges

1. **Convert a deeply nested callback chain to use `async.waterfall`.**
2. **Refactor a callback hell example into named functions.**
3. **Rewrite a callback hell chain using Promises and then async/await.**
4. **Create a utility that automatically flattens callback chains.**

### Related Topics

- Promises
- Async/await
- Control flow
- Error handling patterns
- `util.promisify`

---

## Error-first callback pattern

### What It Is

The error-first callback pattern (also called "Node.js callback convention") is a standard convention where the first argument of a callback is reserved for an error object. If the operation succeeded, the error argument is `null` or `undefined`, and subsequent arguments contain the result data. This pattern is the de facto standard for Node.js asynchronous APIs.

```javascript
// Error-first callback signature
function (error, result) {
  if (error) {
    // Handle error
    console.error("Operation failed:", error);
    return;
  }
  // Handle success
  console.log("Result:", result);
}
```

### Why It Is Important

The error-first pattern solves several problems:
- **Consistency** — All Node.js APIs follow the same convention.
- **Error handling** — Errors are always the first parameter, easy to check.
- **No exception swallowing** — Errors are surfaced explicitly rather than thrown.
- **Universal pattern** — Works across callback-based libraries and utilities.
- **Tooling friendly** — `util.promisify` and other tools rely on this convention.

### How It Works Internally

There is no language-level enforcement of this pattern — it is purely a convention. However, internally:

1. The async function performs its work.
2. If an error occurs, it calls `callback(error)` or `callback(new Error(...), null)`.
3. If successful, it calls `callback(null, result)` or `callback(undefined, result)`.
4. The caller checks the first argument to determine success or failure.
5. The pattern prevents thrown exceptions from being lost in async contexts.

### Syntax

```javascript
// Standard error-first callback signature
function asyncOperation(param1, param2, callback) {
  // callback signature: (error, result) => void
}

// Proper invocation pattern
function myCallback(err, data) {
  if (err) {
    // Error path
    console.error("Error:", err.message);
    return;
  }
  // Success path
  console.log("Data:", data);
}

// Calling the async function
asyncOperation("param", myCallback);
```

### Beginner Examples

```javascript
// Example 1: fs.readFile (Node.js)
const fs = require("fs");

fs.readFile("config.json", "utf8", function (err, data) {
  if (err) {
    console.error("Failed to read config:", err.message);
    return;
  }
  try {
    const config = JSON.parse(data);
    console.log("Config loaded:", config);
  } catch (parseErr) {
    console.error("Invalid JSON:", parseErr.message);
  }
});

// Example 2: Simple implementation
function divideAsync(a, b, callback) {
  setTimeout(function () {
    if (b === 0) {
      callback(new Error("Division by zero"));
      return;
    }
    callback(null, a / b);
  }, 100);
}

divideAsync(10, 2, function (err, result) {
  if (err) return console.error(err);
  console.log(result); // 5
});

divideAsync(10, 0, function (err, result) {
  if (err) return console.error(err.message); // "Division by zero"
});
```

### Intermediate Examples

```javascript
// Example 1: Multiple callbacks with error-first
function fetchUserData(userId, callback) {
  getUserFromDB(userId, function (err, user) {
    if (err) return callback(err);
    getPosts(user.id, function (err, posts) {
      if (err) return callback(err);
      getProfile(userId, function (err, profile) {
        if (err) return callback(err);
        callback(null, { user, posts, profile });
      });
    });
  });
}

// Example 2: Wrapping non-callback APIs
function readFilePromise(path, encoding) {
  return new Promise(function (resolve, reject) {
    fs.readFile(path, encoding, function (err, data) {
      if (err) reject(err);
      else resolve(data);
    });
  });
}

// Example 3: Parallel operations with error-first
function parallelOps(callback) {
  let completed = 0;
  const results = [];
  let hasErrored = false;

  function onDone(index) {
    return function (err, result) {
      if (hasErrored) return;
      if (err) {
        hasErrored = true;
        callback(err);
        return;
      }
      results[index] = result;
      completed++;
      if (completed === 3) {
        callback(null, results);
      }
    };
  }

  asyncOp1(onDone(0));
  asyncOp2(onDone(1));
  asyncOp3(onDone(2));
}
```

### Advanced Examples

```javascript
// Example 1: util.promisify internals
function promisify(fn) {
  return function (...args) {
    return new Promise((resolve, reject) => {
      fn(...args, function (err, result) {
        if (err) reject(err);
        else resolve(result);
      });
    });
  };
}

const readFile = promisify(fs.readFile);
readFile("config.json", "utf8")
  .then(console.log)
  .catch(console.error);

// Example 2: Auto-handling errors in middleware
function errorWrappingCallback(callback) {
  return function (err, result) {
    if (err) {
      // Log, transform, or re-wrap the error
      console.error(`[${new Date().toISOString()}] Error:`, err.message);
      callback(err);
      return;
    }
    callback(null, result);
  };
}

// Example 3: Converting error-first to async/await
const { promisify } = require("util");
const readdir = promisify(fs.readdir);
const stat = promisify(fs.stat);

async function listFilesWithSizes(dir) {
  const files = await readdir(dir);
  const entries = await Promise.all(
    files.map(async (file) => {
      const stats = await stat(file);
      return { file, size: stats.size };
    })
  );
  return entries;
}
```

### Real-World Use Cases

- **Node.js core modules** — `fs`, `child_process`, `dns`, `crypto`, `net`, `http`, `dgram`.
- **Redis client** (node_redis) — All commands return error-first callbacks.
- **MongoDB driver** — `collection.find().toArray(callback)`.
- **Express/Connect middleware** — `next(err)` passes errors.
- **Gulp.js** — Task functions use error-first callbacks.

```javascript
// Express error handling
app.get("/user/:id", function (req, res, next) {
  User.findById(req.params.id, function (err, user) {
    if (err) return next(err); // Pass to Express error handler
    if (!user) return next(new Error("User not found"));
    res.json(user);
  });
});
```

### Common Mistakes

- **Not checking the error** — Processing `result` even when `err` is truthy.
- **Ignoring the error** — `callback(err)` but not returning, so code continues.
- **Calling callback twice** — Once with error, once with success.
- **Not returning after error** — Code continues past the error check.
- **Passing non-Error objects** — Strings or plain objects instead of `Error` instances.

### Best Practices

- Always check `err` first thing in the callback.
- Return immediately after calling `callback(err)` to prevent double invocation.
- Always pass `Error` objects (with stack traces) rather than strings.
- Use `typeof callback === "function"` check for optional callbacks.
- Always call the callback — even if the function is aborted.
- Document the callback signature in JSDoc.

```javascript
// Proper pattern
function safeAsyncOp(callback) {
  const cb = typeof callback === "function" ? callback : noop;

  asyncWork(function (err, result) {
    if (err) {
      cb(new Error(`Operation failed: ${err.message}`));
      return;
    }
    cb(null, result);
  });
}
```

### Performance Considerations

- Error objects creation has overhead — consider reusing error objects in hot paths.
- Checking `err` is a negligible cost.
- Creating callbacks dynamically in hot loops can be optimized by declaring named functions.
- `util.promisify` adds minimal overhead per call.
- Error-first callbacks are generally more performant than Promises due to less object allocation.

### Interview Questions

1. **Why does Node.js use error-first callbacks?**
   For consistency, explicit error handling, and to prevent errors from being silently swallowed.

2. **What happens if you don't check the error argument?**
   The callback processes potentially undefined/invalid data, which may cause unexpected behavior or crashes.

3. **How does `util.promisify` work with error-first callbacks?**
   It wraps a function returning it as a Promise-based function, checking the error argument to decide whether to resolve or reject.

4. **What is the difference between error-first callbacks and throwing exceptions?**
   Errors are passed as arguments rather than thrown, which is necessary in async contexts where try/catch cannot catch async errors.

### Coding Challenges

1. **Implement your own `promisify` function for error-first callbacks.**
2. **Create a function `callbackify` that converts a Promise-returning function to error-first callback style.**
3. **Write a `timeout` wrapper for error-first callbacks that fails after a specified duration.**
4. **Build an error-first `retry` utility that retries on failure.**
5. **Implement a utility that transforms error-first callbacks into a stream of events.**

### Related Topics

- Node.js callback convention
- `util.promisify`
- Error handling patterns
- Asynchronous programming
- Promise construction
