# JS Interview Questions - Core concepts, closures, promises, this, hoisting

## Introduction

JavaScript interviews test foundational language concepts that reveal a candidate's depth of understanding. Core topics like closures, promises, the `this` keyword, and hoisting are universal across frameworks and libraries. Mastering these concepts demonstrates not just syntax knowledge but a genuine understanding of how JavaScript executes. This file covers the most frequently tested interview topics with explanations, examples, and practice questions across experience levels.

## Closures in Interviews

### What It Is

A closure is the combination of a function bundled together with references to its surrounding lexical environment (scope). When a function is defined inside another function, the inner function retains access to the outer function's variables even after the outer function has finished executing. Closures are created every time a function is created, at function creation time.

### Why It Is Important

Closures are fundamental to JavaScript's function-based architecture. They enable data privacy (module pattern), function factories, partial application, currying, callbacks, and event handlers. In interviews, closures test a candidate's understanding of lexical scoping, memory management, and practical JavaScript patterns. The question "What is a closure?" is one of the most common JS interview questions.

### How It Works Internally

When a function executes, an execution context is created with a lexical environment. The lexical environment holds variable bindings and a reference to the outer lexical environment (scope chain). When an inner function is defined, it captures a reference to the current lexical environment. Even after the outer function returns, the inner function's reference to the outer lexical environment prevents garbage collection of those variables, keeping them alive for the closure.

```javascript
function createCounter() {
  let count = 0; // 'count' is in the lexical environment of createCounter
  return function() {
    count++; // inner function closes over 'count'
    return count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
// The 'count' variable persists because counter() holds a reference to it
```

### Syntax

```javascript
// Basic closure pattern
function outer(x) {
  return function inner(y) {
    return x + y;
  };
}

const add5 = outer(5);
console.log(add5(3)); // 8

// Closure with IIFE
const counter = (function() {
  let count = 0;
  return {
    increment: () => ++count,
    decrement: () => --count,
    getValue: () => count
  };
})();

// Closure in loops (pre-ES6 workaround)
for (var i = 0; i < 5; i++) {
  (function(j) {
    setTimeout(() => console.log(j), j * 100);
  })(i);
}
```

### Beginner Examples

```javascript
// Simple closure counter
function makeCounter() {
  let count = 0;
  return function() {
    count += 1;
    return count;
  };
}

const counter1 = makeCounter();
const counter2 = makeCounter();
console.log(counter1()); // 1
console.log(counter1()); // 2
console.log(counter2()); // 1 (separate closure)

// Private variables using closures
function createPerson(name, age) {
  return {
    getName: () => name,
    getAge: () => age,
    setAge: (newAge) => { age = newAge; }
  };
}

const person = createPerson('Alice', 30);
console.log(person.getName()); // 'Alice'
person.setAge(31);
console.log(person.getAge()); // 31
```

### Intermediate Examples

```javascript
// Function factory with closures
function multiply(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = multiply(2);
const triple = multiply(3);
console.log(double(5));  // 10
console.log(triple(5));  // 15

// Memoization with closure
function memoize(fn) {
  const cache = {};
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache[key] !== undefined) {
      console.log('Cache hit:', key);
      return cache[key];
    }
    const result = fn(...args);
    cache[key] = result;
    return result;
  };
}

const factorial = memoize(function(n) {
  return n <= 1 ? 1 : n * factorial(n - 1);
});
console.log(factorial(5)); // 120
console.log(factorial(5)); // Cache hit: [5], 120

// Closure in event listeners
function setupButton(buttonId, message) {
  const button = document.getElementById(buttonId);
  button.addEventListener('click', function() {
    alert(message); // Closes over 'message'
  });
}
```

### Advanced Examples

```javascript
// Currying with closures
function curry(fn) {
  const arity = fn.length;
  function curried(...args) {
    if (args.length >= arity) {
      return fn(...args);
    }
    return function(...moreArgs) {
      return curried(...args, ...moreArgs);
    };
  }
  return curried;
}

const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);
console.log(curriedAdd(1)(2)(3)); // 6

// Partial application
function partial(fn, ...presetArgs) {
  return function(...laterArgs) {
    return fn(...presetArgs, ...laterArgs);
  };
}

const greet = (greeting, name) => `${greeting}, ${name}!`;
const sayHello = partial(greet, 'Hello');
console.log(sayHello('World')); // "Hello, World!"

// Closure-based module pattern
const Module = (function() {
  const private = new WeakMap();
  
  class ModuleClass {
    constructor(name) {
      const privates = { data: [], name };
      private.set(this, privates);
    }
    
    add(item) {
      private.get(this).data.push(item);
    }
    
    getData() {
      return [...private.get(this).data];
    }
  }
  
  return ModuleClass;
})();

// Debugging closures: common gotcha
function createIncrementers() {
  const functions = [];
  for (var i = 0; i < 3; i++) {
    functions.push(function() { return i; }); // Closes over same 'i'
  }
  return functions;
}
const incrementers = createIncrementers();
console.log(incrementers[0]()); // 3 (not 0!)
// Fix: use let or IIFE
```

### Real-World Use Cases

- **Module pattern**: Creating private state in modules
- **Event handlers**: Maintaining state across DOM events
- **Callbacks and timers**: Preserving context in async operations
- **Debouncing and throttling**: Maintaining timer references
- **React hooks**: `useState`, `useEffect`, `useCallback` all rely on closures
- **Middleware**: Express/Redux middleware chains use closures for composability

### Common Mistakes

- Creating closures in loops with `var` (all iterations share the same variable)
- Memory leaks from unintended closures holding large object references
- Confusing closures with scope (closure is the function + its lexical environment, not just the scope)
- Thinking closures copy values (they reference the actual variables)
- Not understanding that each function call creates a new closure

### Best Practices

- Use `let` or `const` in loops instead of `var` to create per-iteration bindings
- Be mindful of memory: release closure references when no longer needed
- Use IIFEs for creating isolated scopes when needed
- Prefer passing values explicitly when closures cause confusion
- Use `WeakMap` for private data in classes to avoid memory leaks

### Performance Considerations

- Each closure maintains a reference to its lexical environment, consuming memory
- Closure overhead is minimal for typical use cases
- Unintended closures in hot paths (e.g., inside render functions) can cause performance issues
- V8 optimizes closure allocation; creation cost is low
- Memory leaks occur when closures outlive their useful lifetime (e.g., in detached DOM event listeners)

### Interview Questions

**Q: What is a closure and how does it work?**
A: A closure is a function that retains access to its outer lexical scope even after the outer function has returned. It works because JavaScript functions carry a reference to the environment in which they were created, forming a scope chain that persists as long as the function exists.

**Q: What's the difference between a closure and a regular function?**
A: A regular function can only access its own scope and global scope. A closure additionally retains access to the scope where it was defined, even after that scope's execution has finished. All functions in JavaScript are technically closures because they all capture their creation environment, but we typically call them closures when this behavior is observable.

### Coding Challenges

**Challenge 1:** Write a function that creates a counter with increment, decrement, and getValue methods.

```javascript
function createCounter(start = 0) {
  // Your implementation
}
```

**Solution:**

```javascript
function createCounter(start = 0) {
  let count = start;
  return {
    increment: () => ++count,
    decrement: () => --count,
    reset: () => { count = start; },
    getValue: () => count
  };
}
```

## Promises and Async Patterns

### What It Is

A Promise is an object representing the eventual completion or failure of an asynchronous operation. Promises provide a cleaner alternative to callbacks for handling async code, with three states: pending, fulfilled (resolved), and rejected. They support chaining via `.then()`, error handling via `.catch()`, and cleanup via `.finally()`.

### Why It Is Important

Promises are the foundation of modern JavaScript asynchronous programming. They enable sequential async operations without callback nesting (callback hell), parallel async operations via `Promise.all()`, race conditions via `Promise.race()`, and clean error handling. The `async/await` syntax built on promises makes async code read like synchronous code.

### How It Works Internally

A Promise wraps an executor function that receives `resolve` and `reject` callbacks. The Promise maintains internal state (pending -> fulfilled/rejected) and a microtask queue for `.then()` callbacks. When resolved, all `.then()` callbacks are added to the microtask queue. Microtasks run before the next macrotask (like `setTimeout`). This gives promises higher priority than timers in the event loop.

```javascript
const promise = new Promise((resolve, reject) => {
  // Synchronous code runs immediately
  setTimeout(() => resolve('done'), 1000);
});

promise.then(value => console.log(value)); // 'done' after 1 second
```

### Syntax

```javascript
// Promise creation
const p = new Promise((resolve, reject) => {
  if (success) resolve(value);
  else reject(error);
});

// Chaining
fetch('/api/data')
  .then(res => res.json())
  .then(data => process(data))
  .catch(err => handleError(err))
  .finally(() => cleanup());

// Static methods
Promise.all([p1, p2, p3]);
Promise.allSettled([p1, p2, p3]);
Promise.race([p1, p2, p3]);
Promise.any([p1, p2, p3]);

// async/await
async function getData() {
  try {
    const res = await fetch('/api/data');
    return await res.json();
  } catch (err) {
    handleError(err);
  }
}
```

### Beginner Examples

```javascript
// Converting callback to promise
function readFilePromise(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, 'utf8', (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}

readFilePromise('file.txt')
  .then(data => console.log(data))
  .catch(err => console.error(err));

// Simple Promise chaining
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

delay(1000)
  .then(() => console.log('1 second'))
  .then(() => delay(1000))
  .then(() => console.log('2 seconds'));
```

### Intermediate Examples

```javascript
// Promise.all with error handling
async function fetchMultipleUrls(urls) {
  try {
    const promises = urls.map(url => 
      fetch(url).then(res => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
    );
    return await Promise.all(promises);
  } catch (err) {
    console.error('One or more requests failed:', err);
    throw err;
  }
}

// Promise chain with value transformation
fetchUser(id)
  .then(user => fetchPosts(user.id))
  .then(posts => posts.filter(p => p.published))
  .then(published => published.map(p => p.title))
  .then(titles => titles.join('\n'))
  .catch(err => console.error('Failed:', err));

// Sequential vs parallel execution
async function processItems(items) {
  // Sequential (slow but controlled)
  const results = [];
  for (const item of items) {
    results.push(await process(item));
  }
  
  // Parallel (fast but resource intensive)
  const parallelResults = await Promise.all(
    items.map(item => process(item))
  );
  
  // Batched (controlled parallelism)
  const batchResults = [];
  for (let i = 0; i < items.length; i += 3) {
    const batch = items.slice(i, i + 3);
    batchResults.push(...await Promise.all(
      batch.map(item => process(item))
    ));
  }
}
```

### Advanced Examples

```javascript
// Custom Promise implementation with retry
function retryPromise(fn, maxRetries = 3, delay = 1000) {
  return new Promise((resolve, reject) => {
    const attempt = (retriesLeft) => {
      fn()
        .then(resolve)
        .catch(err => {
          if (retriesLeft === 0) {
            reject(err);
            return;
          }
          console.log(`Retrying... ${retriesLeft} attempts left`);
          setTimeout(() => attempt(retriesLeft - 1), delay);
        });
    };
    attempt(maxRetries);
  });
}

// Promise pool for concurrency control
async function promisePool(items, concurrency, fn) {
  const results = [];
  const executing = new Set();
  
  for (const [index, item] of items.entries()) {
    const p = fn(item, index, items);
    results.push(p);
    executing.add(p);
    
    const clean = () => executing.delete(p);
    p.then(clean, clean);
    
    if (executing.size >= concurrency) {
      await Promise.race(executing);
    }
  }
  
  return Promise.all(results);
}

// Async generator with Promises
async function* paginate(url, maxPages = Infinity) {
  let page = 1;
  let hasNext = true;
  
  while (hasNext && page <= maxPages) {
    const res = await fetch(`${url}?page=${page}`);
    const data = await res.json();
    hasNext = data.next !== null;
    page++;
    yield data.items;
  }
}

// Usage
for await (const items of paginate('/api/data')) {
  console.log('Page:', items);
}

// Promisify utility
function promisify(fn) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      fn(...args, (err, result) => {
        if (err) reject(err);
        else resolve(result);
      });
    });
  };
}

const readFile = promisify(require('fs').readFile);
```

### Real-World Use Cases

- API calls and data fetching (`fetch`, `axios`, `graphql-request`)
- Database operations (Mongoose, Sequelize, Prisma)
- File I/O in Node.js (readFile, writeFile)
- User authentication flows (login, token refresh)
- Image/video loading and processing
- Stream processing and backpressure handling

### Common Mistakes

- Not returning promises from `.then()` chains (breaks chaining)
- Forgetting to catch promise rejections ("unhandled promise rejection")
- Nesting promises instead of chaining (promise pyramid)
- Mixing sync `try/catch` with promise chains
- Not understanding that `Promise.all` fails fast (rejects on first rejection)
- Creating promise anti-patterns (wrapping existing promises unnecessarily)

### Best Practices

- Always return promises from `.then()` to maintain the chain
- Use `async/await` for cleaner asynchronous code
- Handle errors with `.catch()` or `try/catch` in async functions
- Use `Promise.allSettled()` when you need all results regardless of failure
- Use `Promise.any()` for first successful fulfillment
- Avoid the explicit promise construction antipattern when wrapping APIs
- Add `.catch()` at the end of every promise chain
- Use `finally()` for cleanup operations

### Performance Considerations

- Promise creation is cheap (microtask queue addition)
- `.then()` callbacks run as microtasks, higher priority than `setTimeout`
- Unhandled rejections can crash Node.js (deprecation warning)
- `Promise.all()` runs promises concurrently but can overwhelm resources
- Promise chains create intermediate promise objects (minimal overhead)
- 1000s of concurrent promises can cause memory pressure

### Interview Questions

**Q: What is the difference between `Promise.all()` and `Promise.allSettled()`?**
A: `Promise.all()` rejects immediately when any input promise rejects (fail-fast). `Promise.allSettled()` waits for all promises to settle (resolve or reject) and returns results with status and value/reason. Use `all` when all must succeed; use `allSettled` when you need results regardless.

**Q: How does `async/await` relate to Promises?**
A: `async/await` is syntactic sugar over Promises. An `async` function always returns a Promise. The `await` keyword pauses execution of the async function until the awaited Promise settles, without blocking the main thread. Under the hood, the compiler transforms the function into a state machine using Promises and generators.

### Coding Challenges

**Challenge 1:** Implement `Promise.all()` from scratch.

```javascript
function promiseAll(promises) {
  // Your implementation
}
```

**Solution:**

```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    if (!promises || typeof promises[Symbol.iterator] !== 'function') {
      reject(new TypeError('Expected an iterable'));
      return;
    }
    
    const results = [];
    let completed = 0;
    const iterable = Array.from(promises);
    
    if (iterable.length === 0) {
      resolve(results);
      return;
    }
    
    iterable.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = value;
          completed++;
          if (completed === iterable.length) {
            resolve(results);
          }
        })
        .catch(reject);
    });
  });
}
```

## This Keyword Questions

### What It Is

The `this` keyword in JavaScript refers to the execution context of a function—the object that the function is operating on. Unlike other languages where `this` always refers to the class instance, JavaScript's `this` is determined by how a function is called (call-site), not where it's defined. This dynamic binding is a common source of confusion and a favorite interview topic.

### Why It Is Important

Understanding `this` is essential for working with object methods, event handlers, constructors, and many JavaScript patterns. Misunderstanding `this` leads to bugs that are hard to diagnose. Frameworks like React rely heavily on correct `this` binding for class components, event handlers, and lifecycle methods.

### How It Works Internally

When a function is called, a new execution context is created. The `this` value is determined by the call pattern:

1. **Default binding**: Non-strict mode: `this` is the global object (`window`/`global`). Strict mode: `this` is `undefined`.
2. **Implicit binding**: When called as `obj.method()`, `this` is `obj`.
3. **Explicit binding**: Using `.call()`, `.apply()`, or `.bind()`, `this` is the explicitly passed value.
4. **New binding**: When called with `new`, `this` is the newly created object.
5. **Arrow functions**: `this` is lexically bound to the enclosing scope (not call-site determined).

```javascript
function showThis() {
  console.log(this);
}

const obj = { method: showThis };
showThis();        // Default: global/undefined (strict mode)
obj.method();      // Implicit: obj
showThis.call(obj); // Explicit: obj
new showThis();    // New: new instance
```

### Syntax

```javascript
// Default binding
function foo() { return this; }
console.log(foo() === global); // true (non-strict)

// Implicit binding
const user = {
  name: 'Alice',
  greet() { return `Hello, ${this.name}`; }
};
console.log(user.greet()); // 'Hello, Alice'

// Explicit binding
function introduce(verb) {
  return `${this.name} ${verb}`;
}
console.log(introduce.call(user, 'speaks'));  // 'Alice speaks'
console.log(introduce.apply(user, ['walks'])); // 'Alice walks'
const bound = introduce.bind(user, 'jumps');
console.log(bound()); // 'Alice jumps'

// New binding
function Person(name) {
  this.name = name;
}
const p = new Person('Bob');
console.log(p.name); // 'Bob'

// Arrow function (lexical this)
const obj2 = {
  name: 'Obj2',
  regular: function() { return this.name; },
  arrow: () => this.name
};
console.log(obj2.regular()); // 'Obj2'
console.log(obj2.arrow());   // undefined or global name
```

### Beginner Examples

```javascript
// Common 'this' mistake
const user = {
  name: 'Alice',
  greet() {
    console.log(`Hello, ${this.name}`);
  },
  greetLater() {
    setTimeout(function() { // Regular function loses 'this'
      console.log(`Hello, ${this.name}`); // this is global/undefined
    }, 1000);
  },
  greetLaterFixed() {
    setTimeout(() => { // Arrow function captures 'this'
      console.log(`Hello, ${this.name}`); // this is user
    }, 1000);
  }
};

// Event handler 'this'
button.addEventListener('click', function() {
  console.log(this); // The button element
});

button.addEventListener('click', () => {
  console.log(this); // The enclosing context (not the button!)
});
```

### Intermediate Examples

```javascript
// Method borrowing
const car = {
  brand: 'Tesla',
  start() { return `${this.brand} started`; }
};

const bike = { brand: 'Ducati' };
console.log(car.start.call(bike)); // 'Ducati started'

// 'this' in nested functions (pre-ES6 workaround)
const obj = {
  data: [1, 2, 3],
  process() {
    const self = this; // Capture this
    this.data.forEach(function(item) {
      console.log(self.data); // Use self instead of this
    });
  }
};

// Constructor with and without new
function Create(name) {
  if (!new.target) {
    return new Create(name); // Auto-invoke with new
  }
  this.name = name;
}

const c1 = new Create('Alice');
const c2 = Create('Bob'); // Also works due to safety check
```

### Advanced Examples

```javascript
// Function.prototype.bind implementation
Function.prototype.myBind = function(thisArg, ...boundArgs) {
  const original = this;
  return function(...args) {
    return original.call(thisArg, ...boundArgs, ...args);
  };
};

// 'this' in class fields vs prototype methods
class MyClass {
  name = 'instance';
  arrowField = () => this.name; // Lexical this
  
  prototypeMethod() {
    return this.name; // Dynamic this
  }
}

const obj = new MyClass();
const { arrowField, prototypeMethod } = obj;
console.log(arrowField());      // 'instance' (lexical this = obj)
console.log(prototypeMethod()); // 'instance' or undefined if detached

// 'this' with getters/setters
const counter = {
  _count: 0,
  get count() { return this._count; },
  set count(v) { this._count = v; },
  increment() { this._count++; }
};

// Proxy with 'this'
const target = { name: 'target' };
const handler = {
  get(t, prop, receiver) {
    console.log(`Getting ${prop}, this is:`, this); // handler, not target
    return Reflect.get(t, prop, receiver);
  }
};

const proxy = new Proxy(target, handler);
```

### Real-World Use Cases

- React class components (method binding in JSX)
- DOM event listeners (this = element)
- jQuery style chaining (returning this from methods)
- Method extraction (passing object methods as callbacks)
- Constructor functions and classes
- Middleware patterns (Express, Passport)

### Common Mistakes

- Losing `this` when passing methods as callbacks: `setTimeout(obj.method, 100)` — `this` will be wrong
- Forgetting to bind event handlers in React class components
- Using arrow functions as object methods (lexical `this` may not refer to the object)
- Confusing `this` in regular functions vs arrow functions in the same code
- Not understanding strict mode effect on `this` (undefined vs global)

### Best Practices

- Use arrow functions when you need lexical `this` (callbacks, event handlers)
- Use regular functions (method shorthand) for object methods
- Bind methods explicitly in constructors (class components) or use class field arrows
- Avoid storing `this` in a variable (`self`, `that`) — use arrow functions instead
- Use `.call()`, `.apply()`, or `.bind()` explicitly for dynamic `this` control
- Enable strict mode to avoid accidental global `this` references

### Performance Considerations

- `.bind()` creates a new function wrapper (minor overhead)
- Arrow functions have slightly different internal handling but comparable performance
- Property lookup for `this` is fast (it's a lexical reference, not a property lookup)
- Repeated `.bind()` calls in render methods (React) should be avoided; bind once

### Interview Questions

**Q: How is the value of `this` determined in a regular function vs an arrow function?**
A: In regular functions, `this` is determined by the call-site (how the function is called): implicit binding, explicit binding, new binding, or default binding. In arrow functions, `this` is lexically bound: it inherits `this` from the enclosing scope at the time of definition, regardless of how the arrow function is called.

**Q: What will `this` be inside a `setTimeout` callback?**
A: In non-strict mode with a regular function, `this` will be the global object (`window`/`global`). In strict mode, `this` will be `undefined`. If an arrow function is used, `this` will be the enclosing context where the arrow function was defined.

### Coding Challenges

**Challenge 1:** Implement `.bind()` manually.

```javascript
function myBind(fn, context, ...boundArgs) {
  // Your implementation
}
```

**Solution:**

```javascript
function myBind(fn, context, ...boundArgs) {
  return function(...args) {
    return fn.apply(context, [...boundArgs, ...args]);
  };
}

// Or as a prototype method
if (!Function.prototype.myBind) {
  Function.prototype.myBind = function(context, ...boundArgs) {
    const fn = this;
    return function(...args) {
      return fn.apply(context, [...boundArgs, ...args]);
    };
  };
}
```

## Hoisting and Scope Questions

### What It Is

Hoisting is JavaScript's behavior of moving variable and function declarations to the top of their containing scope during the compilation phase, before execution. This means that `var` declarations and function declarations are accessible before the line they appear in the source code. However, `let` and `const` declarations are hoisted but not initialized (temporal dead zone), and `class` declarations are not hoisted.

### Why It Is Important

Hoisting is a fundamental JavaScript behavior that affects code organization and readability. Understanding hoisting helps developers avoid bugs from accessing variables before declaration. It explains why functions can be called before their definition, why `var` variables are `undefined` before assignment, and why `let`/`const` throw `ReferenceError` before initialization.

### How It Works Internally

During the compilation phase, the JavaScript engine performs a first pass that scans for declarations. Function declarations are fully hoisted (both name and body). `var` declarations are hoisted with initialization to `undefined`. `let` and `const` are hoisted but remain uninitialized in the temporal dead zone (TDZ) until the actual declaration line is reached.

```javascript
console.log(foo); // undefined (var hoisted with undefined)
console.log(bar); // ReferenceError: Cannot access before initialization

var foo = 'foo';
let bar = 'bar';
```

### Syntax

```javascript
// Function declaration hoisting
sayHello(); // "Hello!" — function is hoisted
function sayHello() { console.log('Hello!'); }

// var hoisting
console.log(x); // undefined
var x = 5;

// let/const hoisting (TDZ)
console.log(y); // ReferenceError
let y = 10;

// Function expression hoisting
sayHi(); // TypeError: sayHi is not a function
var sayHi = function() { console.log('Hi'); };
```

### Beginner Examples

```javascript
// What gets hoisted where
console.log(a); // undefined
var a = 1;

console.log(b); // ReferenceError
let b = 2;

// Function declaration example
walk(); // 'Walking...'
function walk() { console.log('Walking...'); }

// Function expression example
run(); // TypeError: run is not a function (var is hoisted, but assignment isn't)
var run = function() { console.log('Running...'); };

// Block scoping
if (true) {
  var x = 1;  // Function-scoped, leaks outside block
  let y = 2;  // Block-scoped, stays in block
}
console.log(x); // 1
console.log(y); // ReferenceError
```

### Intermediate Examples

```javascript
// Hoisting order within a function
function example() {
  console.log(a); // undefined
  console.log(b); // ReferenceError: Cannot access 'b' before initialization
  console.log(c); // undefined (function hoisting happens first)
  
  var a = 1;
  let b = 2;
  
  function c() {}
  
  console.log(a); // 1
  console.log(b); // 2
}

// Multiple var declarations with same name
var x = 1;
function example2() {
  console.log(x); // undefined (local x hoisted, shadows global)
  var x = 2;
  console.log(x); // 2
}

// The temporal dead zone in practice
let x = 'outer';
{
  console.log(x); // ReferenceError: Cannot access 'x' before initialization
  let x = 'inner';
}

// Class hoisting
const c = new MyClass(); // ReferenceError
class MyClass {}

// Hoisting with import
console.log(add(1, 2)); // Works! Import hoisting
import { add } from './math';
```

### Advanced Examples

```javascript
// Interaction between var hoisting and closures
function process() {
  var results = [];
  for (var i = 0; i < 3; i++) {
    results.push(function() { return i; });
  }
  return results;
}
// All functions return 3 because var i is hoisted to function scope

// let in loops creates per-iteration bindings
function processFixed() {
  const results = [];
  for (let i = 0; i < 3; i++) {
    results.push(function() { return i; });
  }
  return results;
}
// Each function returns its own i (0, 1, 2)

// IIFE and hoisting
(function() {
  console.log(a); // undefined
  var a = 10;
})();

// Module scope hoisting (ES modules have their own scope)
// a.js
export const shared = 'value';
// b.js
console.log(shared); // Works due to static module resolution

// Understanding temporal dead zone with typeof
typeof undeclared; // 'undefined'
typeof x; // ReferenceError (TDZ)
let x = 1;

// Function vs var hoisting priority
function foo() { return 'function'; }
var foo = 'string';
console.log(foo); // 'string' (var assignment overwrites function after hoisting)
```

### Real-World Use Cases

- Understanding where to place function declarations (they can be anywhere)
- Debugging "undefined" variable errors (likely var hoisting)
- Debugging ReferenceError with let/const (likely TDZ issue)
- Code organization: knowing that function declarations can be called before definition
- Module imports: understanding that imports are hoisted

### Common Mistakes

- Assuming `let` and `const` are not hoisted (they are, but with TDZ)
- Relying on `var` hoisting for code clarity (it makes code harder to understand)
- Accessing `let`/`const` before declaration in the same block
- Confusing function declarations with function expressions regarding hoisting
- Assuming blocks create scope for `var` (they don't)

### Best Practices

- Declare all variables at the top of their scope (or close to usage)
- Use `const` by default, `let` when reassignment is needed, never `var`
- Avoid relying on hoisting for code organization
- Declare functions before using them (even though they can be hoisted)
- Enable linting rules that catch TDZ violations (`no-use-before-define`)
- Group all `import` statements at the top of modules

### Performance Considerations

- Hoisting is a compile-time operation with zero runtime cost
- TDZ checks add minimal overhead (V8 handles this efficiently)
- Function hoisting allows the engine to optimize function allocation
- Module import hoisting enables static analysis for tree shaking
- No runtime performance difference between var, let, and const (after initialization)

### Interview Questions

**Q: What is the temporal dead zone (TDZ) with `let` and `const`?**
A: The TDZ is the time between entering a scope (block) and the actual declaration of a `let` or `const` variable. During this period, accessing the variable throws a `ReferenceError`. The variable is hoisted (the engine knows about it) but is uninitialized. Once the declaration line is reached, the variable becomes accessible.

**Q: How does hoisting differ between `var`, `function`, `let`, `const`, and `class` declarations?**
A: `function` declarations: fully hoisted (name + body). `var`: hoisted and initialized to `undefined`. `let`/`const`: hoisted but uninitialized (TDZ). `class`: hoisted but uninitialized (TDZ) like let/const. `import`: hoisted (available throughout module). Arrow functions and function expressions are not hoisted because they are assigned to variables.

### Coding Challenges

**Challenge 1:** Predict the output and explain the behavior.

```javascript
console.log(a);
var a = 1;

console.log(b);
let b = 2;

function test() {
  console.log(c);
  var c = 3;
  console.log(d);
  let d = 4;
}
test();
```

**Solution:**

```javascript
// Output:
// undefined (var a hoisted, initialized to undefined before assignment)
// ReferenceError: Cannot access 'b' before initialization (TDZ)
// undefined (var c hoisted within function)
// ReferenceError: Cannot access 'd' before initialization (TDZ)
```

### Related Topics

- Execution context (covered in detail in expert section)
- Scope chain and lexical environment
- `let`, `const`, and `var` comparison
- Temporal Dead Zone in depth
- `this` binding and its relationship to scope
- Module system and import hoisting
- IIFE patterns and scope isolation
