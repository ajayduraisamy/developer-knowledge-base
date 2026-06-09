# Closures

## Introduction

Closures are one of the most fundamental and powerful concepts in JavaScript. A closure is created when a function retains access to its lexical scope even after that function has finished executing. This allows functions to "remember" the environment in which they were created, enabling patterns like data encapsulation, factory functions, and module definitions. Understanding closures is essential for mastering JavaScript because they appear in virtually every codebase, from event handlers to callbacks to state management.

---

## Closure definition

### What It Is

A closure is the combination of a function bundled together with references to its surrounding state (the lexical environment). In practice, a closure is created every time a function is defined inside another function and the inner function references variables from the outer function's scope. The inner function "closes over" those variables, meaning it retains access to them even after the outer function has returned.

```javascript
function outer() {
  let count = 0;
  function inner() {
    count++;
    return count;
  }
  return inner;
}

const increment = outer();
console.log(increment()); // 1
console.log(increment()); // 2
```

Here, `inner` closes over the `count` variable from `outer`, maintaining its value across multiple calls.

### Why It Is Important

Closures enable powerful programming patterns that would otherwise be impossible or cumbersome:
- **Data privacy** — Variables can be hidden from the outside scope.
- **State persistence** — Functions can maintain state between invocations.
- **Function factories** — Functions that generate customized functions.
- **Module pattern** — Encapsulating private implementation details.
- **Callback preservation** — Callbacks maintain access to their creation context.

Without closures, JavaScript would lack a native mechanism for true encapsulation and functional composition.

### How It Works Internally

Every function in JavaScript has an internal `[[Environment]]` slot that references the lexical environment where the function was created. When a function is invoked, a new execution context is created, and its outer environment reference is set to the value of `[[Environment]]`. This chain of environment records forms the scope chain.

When a function references a variable that is not in its local scope, the engine walks up the scope chain through the environment records linked by `[[Environment]]`. Even if the outer function has returned, its variable environment is retained because the inner function's `[[Environment]]` still references it. The garbage collector keeps the environment alive as long as any closure references it.

```javascript
// Visual representation of internal references:
// outer() creates LexicalEnvironment LE1
// inner() has [[Environment]] -> LE1
// After outer returns, LE1 is NOT garbage collected
// because inner still references it via [[Environment]]
```

### Syntax

There is no special syntax for creating a closure — it happens automatically whenever a function is defined inside another function and accesses outer variables.

```javascript
// Implicit closure (function inside another function)
function greet(name) {
  return function (message) {
    return `${message}, ${name}!`;
  };
}

// Using a closure in an IIFE
const counter = (function () {
  let count = 0;
  return function () {
    count++;
    return count;
  };
})();
```

### Beginner Examples

```javascript
// Example 1: Basic closure
function createGreeting(greeting) {
  return function (name) {
    console.log(`${greeting}, ${name}!`);
  };
}

const sayHello = createGreeting("Hello");
const sayHi = createGreeting("Hi");
sayHello("Alice"); // Hello, Alice!
sayHi("Bob");      // Hi, Bob!

// Example 2: Counter
function createCounter() {
  let count = 0;
  return {
    increment: function () { count++; },
    decrement: function () { count--; },
    getCount: function () { return count; },
  };
}

const counter = createCounter();
counter.increment();
counter.increment();
console.log(counter.getCount()); // 2
console.log(counter.count);      // undefined (private)
```

### Intermediate Examples

```javascript
// Example 1: Closure with setTimeout
function delayedMessage(msg, delay) {
  setTimeout(function () {
    console.log(msg);
  }, delay);
}
delayedMessage("Hello after 1s", 1000);

// Example 2: Closure in loops (problem with var)
function createButtons() {
  for (var i = 1; i <= 3; i++) {
    (function (index) {
      document
        .getElementById(`btn${index}`)
        .addEventListener("click", function () {
          console.log(`Button ${index} clicked`);
        });
    })(i);
  }
}

// Example 3: Private variables
function createBankAccount(initialBalance) {
  let balance = initialBalance;
  return {
    deposit: function (amount) {
      balance += amount;
      return balance;
    },
    withdraw: function (amount) {
      if (amount > balance) return "Insufficient funds";
      balance -= amount;
      return balance;
    },
    getBalance: function () {
      return balance;
    },
  };
}

const account = createBankAccount(1000);
account.deposit(500);
account.withdraw(200);
console.log(account.getBalance()); // 1300
console.log(account.balance);      // undefined
```

### Advanced Examples

```javascript
// Example 1: Memoization with closure
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

const factorial = memoize(function (n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
});

console.log(factorial(100)); // Calculated
console.log(factorial(100)); // Cached

// Example 2: Partial application with closure
function partial(fn, ...presetArgs) {
  return function (...laterArgs) {
    return fn(...presetArgs, ...laterArgs);
  };
}

function add(a, b, c) {
  return a + b + c;
}

const add5 = partial(add, 5);
const add5And10 = partial(add, 5, 10);
console.log(add5(3, 2));   // 10
console.log(add5And10(20)); // 35

// Example 3: Lazy evaluation
function lazySum(arr) {
  let sum = 0;
  for (const n of arr) {
    sum += n;
  }
  return function () {
    return sum;
  };
}

const getSum = lazySum([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);
// Computation happened once
console.log(getSum()); // 55
```

### Real-World Use Cases

- **Module bundlers (Webpack, Rollup)** — Use closures to scope modules and prevent global namespace pollution.
- **React hooks (useState, useEffect)** — Rely on closures to persist state across renders.
- **Express/Node.js middleware** — Middleware functions close over configuration data.
- **jQuery** — The `$` function and chaining rely heavily on closures for private state.
- **Redux** — Store `getState()` and `subscribe()` use closures for internal state.

```javascript
// Redux-like minimal store
function createStore(reducer, initialState) {
  let state = initialState;
  const listeners = [];
  return {
    getState: function () {
      return state;
    },
    dispatch: function (action) {
      state = reducer(state, action);
      listeners.forEach((fn) => fn());
    },
    subscribe: function (fn) {
      listeners.push(fn);
      return function () {
        const idx = listeners.indexOf(fn);
        listeners.splice(idx, 1);
      };
    },
  };
}
```

### Common Mistakes

- **Closures in loops with `var`** — All iterations share the same `var` variable.
- **Accidental closure creation** — Creating closures in hot paths can cause memory overhead.
- **Memory leaks** — Holding references to large objects through closures prevents GC.
- **Stale closure values** — Capturing a primitive value vs. a reference changes behavior.
- **Overusing closures** — Sometimes a simpler approach (like a class) is more appropriate.

### Best Practices

- Use `let` or `const` in loops instead of `var` to avoid the loop closure bug.
- Minimize the number of variables captured in closures to reduce memory footprint.
- Explicitly `null` references to large objects when they are no longer needed.
- Prefer closures for true encapsulation needs; use classes when you need inheritance.
- Use closures for factory functions instead of repeating configuration logic.

### Performance Considerations

- Each closure keeps its entire lexical environment alive, which can increase memory usage.
- Accessing variables through closures is slightly slower than local variables because of scope chain traversal.
- Modern JavaScript engines (V8, SpiderMonkey) optimize closures aggressively — they share environments when variables are not referenced.
- Creating many closures inside hot loops can increase GC pressure.
- Use `WeakMap` with closures to avoid preventing GC of objects used as keys.

### Interview Questions

1. **What is a closure and how does it work?**
   A closure is a function that retains access to its lexical scope even after the outer function has returned. It works because functions have an internal `[[Environment]]` reference to the environment where they were created.

2. **What is the difference between a closure and a regular function?**
   A regular function only accesses its own local scope and the global scope. A closure additionally retains access to the scope of its enclosing function(s).

3. **How would you fix the closure loop problem with `var`?**
   Use `let` (block scoping) or an IIFE that captures the loop variable per iteration.

4. **Can closures cause memory leaks?**
   Yes, if a closure holds references to large objects that are no longer needed, they cannot be garbage collected.

5. **Explain how closures are used in the module pattern.**
   An IIFE returns an object with methods that close over private variables, exposing only the public API.

### Coding Challenges

1. **Create a function `once` that ensures a given function can only be called once.**
2. **Implement a `throttle` function that limits how often a function can be called.**
3. **Build a simple Pub/Sub (EventEmitter) using closures.**
4. **Create a function `curry` that transforms a multi-argument function into a sequence of unary functions.**
5. **Implement a `compose` function that composes multiple functions right-to-left using closures.**

### Related Topics

- Lexical scoping
- Higher-order functions
- IIFE (Immediately Invoked Function Expression)
- Function factories
- Module pattern
- Currying and partial application
- Memoization
- Garbage collection and memory management

---

## Lexical environment

### What It Is

A lexical environment is a specification type defined by ECMAScript that holds the mapping of identifiers (variable names) to their values within a specific scope. Each execution context has an associated lexical environment that consists of an environment record (where variable bindings are stored) and a reference to the outer lexical environment (forming the scope chain).

```javascript
// Lexical environments in nested functions
const globalEnv = { a: 1 };
function outer() {
  // outer's lexical environment: { b: 2 } -> outer: globalEnv
  const b = 2;
  function inner() {
    // inner's lexical environment: { c: 3 } -> inner: outer's env
    const c = 3;
    return a + b + c;
  }
  return inner();
}
```

### Why It Is Important

Lexical environments determine variable visibility and scope resolution at the time code is written (lexical time), not at runtime. This predictability is why JavaScript is statically scoped. Understanding lexical environments helps developers reason about:
- Which variables are accessible where
- Why closures work the way they do
- The behavior of `var`, `let`, and `const`
- The temporal dead zone
- How nested functions resolve variable names

### How It Works Internally

When the JavaScript engine executes a script, it creates a global lexical environment. Each function call creates a new lexical environment. Each block statement (`{ }`) for `let`/`const` also creates a new lexical environment.

A lexical environment has two components:
1. **Environment Record** — Stores variable and function declarations.
2. **Outer Reference** — Points to the parent lexical environment.

Variable resolution follows the scope chain: the engine looks in the current environment record first, then the outer, then the next outer, until it reaches the global environment.

```javascript
// Internal representation (pseudocode):
// GlobalEnv = {
//   record: { x: 10, outer: <function> },
//   outer: null
// }
// outerEnv = {
//   record: { y: 20 },
//   outer: GlobalEnv
// }
// innerEnv = {
//   record: { z: 30 },
//   outer: outerEnv
// }
// Resolving 'x' in innerEnv: innerEnv -> outerEnv -> GlobalEnv -> found
```

### Syntax

Lexical environments are created implicitly by the engine. The syntax that influences their structure includes:

```javascript
// Global environment
var a = 1;
let b = 2;
const c = 3;

// Function environment
function myFunction() {
  var x = 10;
  let y = 20;
}

// Block environment (let/const only)
if (true) {
  let blockVar = "block scope";
  const blockConst = "also block scope";
}

// Nested function environments
function outer() {
  let outerVar = "outer";
  function inner() {
    let innerVar = "inner";
    return outerVar + innerVar;
  }
  return inner;
}
```

### Beginner Examples

```javascript
// Example 1: Variable shadowing
let value = "global";
function example() {
  let value = "function";
  if (true) {
    let value = "block";
    console.log(value); // "block"
  }
  console.log(value); // "function"
}
console.log(value); // "global"

// Example 2: Scope chain resolution
const animal = "cat";
function speak() {
  const sound = "meow";
  function announce() {
    console.log(`The ${animal} says ${sound}`);
  }
  announce();
}
speak(); // "The cat says meow"

// Example 3: let vs var environment
function demo() {
  console.log(a); // undefined (hoisted)
  console.log(b); // ReferenceError: temporal dead zone
  var a = 1;
  let b = 2;
}
```

### Intermediate Examples

```javascript
// Example 1: Block-level lexical environment
function processItems(items) {
  for (let i = 0; i < items.length; i++) {
    setTimeout(function () {
      console.log(items[i]); // undefined because i is block-scoped
    }, 100);
  }
}
// Each iteration gets its own 'i' binding

// Example 2: Nested lexical environments
const factory = "Acme";
function createWidget(type) {
  const factory = "Internal"; // shadows outer
  return function widget(part) {
    const serial = `${factory}-${type}-${part}`;
    return serial;
  };
}
const makeBolt = createWidget("bolt");
console.log(makeBolt("001")); // "Internal-bolt-001"

// Example 3: Switch creates block environments
let x = 1;
switch (x) {
  case 1: {
    let msg = "one";
    console.log(msg);
    // 'msg' is trapped in this block
  }
}
```

### Advanced Examples

```javascript
// Example 1: Multiple closure environments sharing
function createMultipliers() {
  const multipliers = [];
  for (let i = 0; i < 3; i++) {
    multipliers.push(function (val) {
      return val * (i + 1); // each closure captures its own 'i'
    });
  }
  return multipliers;
}
const [doubler, tripler, quadrupler] = createMultipliers();
console.log(doubler(5));   // 5
console.log(tripler(5));   // 10
console.log(quadrupler(5)); // 15

// Example 2: Using eval to inspect environments
function inspectScope() {
  const secret = 42;
  return eval("secret"); // eval runs in current lexical environment
}

// Example 3: with statement (deprecated) affects environment
const obj = { a: 1, b: 2 };
function withDemo() {
  const a = 99;
  with (obj) {
    console.log(a); // 1 (from obj), not 99
    console.log(b); // 2
  }
}
```

### Real-World Use Cases

- **Babel/TypeScript transpilation** — Understanding lexical environments is essential for correctly transforming ES6+ to ES5.
- **ESLint scope analysis** — The `no-unused-vars` and `block-scoped-var` rules rely on lexical environment analysis.
- **Debuggers (Chrome DevTools)** — The Scope panel shows the lexical environment chain at each breakpoint.
- **Module loaders** — SystemJS and others use lexical environments to isolate module scopes.

### Common Mistakes

- **Confusing `let` with `var` hoisting** — Both are hoisted, but `let` enters the temporal dead zone.
- **Assuming block scopes don't affect functions** — Function declarations in blocks behave differently across strict/non-strict modes.
- **Shadowing variables unintentionally** — A similarly-named variable in an inner scope hides the outer one.
- **Misunderstanding the global environment** — `var` creates properties on `window`/`globalThis`, `let` does not.

### Best Practices

- Prefer `const` by default, then `let` when reassignment is needed.
- Never use `var` in modern codebases.
- Avoid shadowing outer variables — rename inner variables to be more specific.
- Keep functions shallow to reduce complex environment nesting.
- Use block scopes (`{}`) to contain temporary variables.

### Performance Considerations

- Lexical environment lookup is very fast — modern engines optimize with inline caching.
- Deeply nested scopes can increase lookup time marginally, but this is rarely a bottleneck.
- Each new lexical environment has some memory overhead — avoid unnecessary block scopes in hot paths.
- The global environment is always present and is the slowest to resolve (prototype chain for global properties).

### Interview Questions

1. **What is the difference between lexical environment and execution context?**
   An execution context contains the lexical environment, variable environment, and `this` binding. The lexical environment is part of the execution context.

2. **How does the temporal dead zone relate to lexical environments?**
   `let` and `const` variables are hoisted but uninitialized — they exist in the lexical environment but cannot be accessed until the declaration is reached.

3. **What happens to the lexical environment when a function returns?**
   If the function created no closures, the environment is eligible for garbage collection. If closures exist, the environment persists via `[[Environment]]` references.

4. **Explain how `eval` interacts with lexical environments.**
   `eval` creates new bindings in the current lexical environment (in non-strict mode) or creates its own lexical environment (in strict mode).

### Coding Challenges

1. **Write a function that determines if a variable is accessible at a given point in code (no cheating with try/catch).**
2. **Create a function that prints all variables in the current lexical environment (use `eval` or `Reflect`).**
3. **Implement a function that measures scope depth by counting lexical environments from a given function.**

### Related Topics

- Execution context
- Scope chain
- Hoisting
- Temporal dead zone
- `with` statement (deprecated)
- `eval` function

---

## Practical closure uses (factories, modules)

### What It Is

Beyond simple variable capture, closures enable practical patterns that are the foundation of many JavaScript tools and libraries. Two of the most significant patterns are **factory functions** (functions that return other functions or objects) and **the module pattern** (encapsulating private state with a public API).

```javascript
// Factory function example
function createGreeter(language) {
  return function (name) {
    if (language === "en") return `Hello, ${name}!`;
    if (language === "es") return `¡Hola, ${name}!`;
    if (language === "fr") return `Bonjour, ${name}!`;
    return `Hello, ${name}!`;
  };
}
```

### Why It Is Important

The factory and module patterns, powered by closures, provide:
- **Encapsulation** without the need for classes
- **Composition** over inheritance
- **Stateless to stateful** transformations
- **Dependency injection** at function level
- **Unit testability** through mockable dependencies

These patterns were essential to JavaScript before ES6 classes and remain idiomatic even in modern code.

### How It Works Internally

Factory functions work by defining inner functions that capture the factory's parameters and local variables in their `[[Environment]]`. Each call to the factory creates a new lexical environment, so each returned closure has its own private copy of the captured state.

The module pattern leverages an IIFE (Immediately Invoked Function Expression) to create a single lexical environment. The IIFE returns an object whose methods are closures over that environment. This creates a singleton with private and public members.

### Syntax

```javascript
// Factory function pattern
function createPerson(name, age) {
  return {
    getName: function () { return name; },
    getAge: function () { return age; },
    setAge: function (newAge) { age = newAge; },
  };
}

// Revealing module pattern
const CounterModule = (function () {
  let count = 0;
  function increment() { count++; }
  function decrement() { count--; }
  function getCount() { return count; }
  return { increment, decrement, getCount };
})();
```

### Beginner Examples

```javascript
// Example 1: Factory for DOM elements
function createElement(tag) {
  return function (text, className) {
    const el = document.createElement(tag);
    el.textContent = text;
    if (className) el.className = className;
    return el;
  };
}
const makeDiv = createElement("div");
const makeSpan = createElement("span");
const myDiv = makeDiv("Hello", "greeting");

// Example 2: Simple module
const Logger = (function () {
  const logs = [];
  return {
    log: function (msg) {
      logs.push(msg);
      console.log(`[LOG]: ${msg}`);
    },
    getLogs: function () {
      return [...logs];
    },
    clear: function () {
      logs.length = 0;
    },
  };
})();

Logger.log("App started");
Logger.log("User logged in");
console.log(Logger.getLogs()); // ["App started", "User logged in"]
```

### Intermediate Examples

```javascript
// Example 1: Configuration factory
function createApiClient(baseURL) {
  let token = null;
  return {
    setToken: function (t) { token = t; },
    get: function (endpoint) {
      return fetch(`${baseURL}${endpoint}`, {
        headers: token ? { Authorization: `Bearer ${token}` } : {},
      });
    },
    post: function (endpoint, data) {
      return fetch(`${baseURL}${endpoint}`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          ...(token ? { Authorization: `Bearer ${token}` } : {}),
        },
        body: JSON.stringify(data),
      });
    },
  };
}
const api = createApiClient("https://api.example.com");
api.setToken("abc123");
api.get("/users").then((res) => res.json());

// Example 2: State machine factory
function createStateMachine(initialState, transitions) {
  let state = initialState;
  return {
    getState: function () { return state; },
    transition: function (action) {
      const nextState = transitions[state]?.[action];
      if (!nextState) throw new Error(`Invalid transition: ${state} -> ${action}`);
      state = nextState;
      return state;
    },
    reset: function () {
      state = initialState;
    },
  };
}

const lightFSM = createStateMachine("red", {
  red: { next: "green" },
  green: { next: "yellow" },
  yellow: { next: "red" },
});
console.log(lightFSM.transition("next")); // green
```

### Advanced Examples

```javascript
// Example 1: Dependency injection module
const ServiceContainer = (function () {
  const services = new Map();
  const instances = new Map();
  return {
    register: function (name, factory) {
      services.set(name, factory);
    },
    get: function (name) {
      if (!instances.has(name)) {
        const factory = services.get(name);
        if (!factory) throw new Error(`Service not found: ${name}`);
        instances.set(name, factory());
      }
      return instances.get(name);
    },
    reset: function () {
      instances.clear();
    },
  };
})();

// Example 2: Currying with factory pattern
function curry(fn, arity = fn.length) {
  return (function currier(args) {
    return function (...nextArgs) {
      const combined = [...args, ...nextArgs];
      if (combined.length >= arity) {
        return fn(...combined);
      }
      return currier(combined);
    };
  })([]);
}

function sum(a, b, c) {
  return a + b + c;
}

const curriedSum = curry(sum);
console.log(curriedSum(1)(2)(3)); // 6

// Example 3: Fluent API with closures
function createQuery() {
  let table = "";
  const conditions = [];
  let orderByField = "";
  let limitCount = null;
  return {
    from: function (t) { table = t; return this; },
    where: function (field, op, value) {
      conditions.push({ field, op, value });
      return this;
    },
    orderBy: function (field) { orderByField = field; return this; },
    limit: function (n) { limitCount = n; return this; },
    build: function () {
      let sql = `SELECT * FROM ${table}`;
      if (conditions.length) {
        sql +=
          " WHERE " +
          conditions.map((c) => `${c.field} ${c.op} ${c.value}`).join(" AND ");
      }
      if (orderByField) sql += ` ORDER BY ${orderByField}`;
      if (limitCount) sql += ` LIMIT ${limitCount}`;
      return sql;
    },
  };
}
const query = createQuery()
  .from("users")
  .where("age", ">", 18)
  .where("status", "=", "'active'")
  .orderBy("name")
  .limit(10)
  .build();
```

### Real-World Use Cases

- **Vue's `data()` function** — Returns a fresh state object each time (factory pattern).
- **Express router** — `express.Router()` uses closures to isolate middleware stacks.
- **Jest's `describe`/`it`** — Each test block has its own closure scope.
- **Axios instances** — `axios.create({ baseURL })` returns a configured instance via closures.
- **Knex.js query builder** — Fluent API backed by closures for query construction.

### Common Mistakes

- **Forgetting that factory calls create new state** — Each factory invocation is independent.
- **Trying to use `this` in factory functions without understanding context.**
- **Over-engineering modules** — Not everything needs to be in a revealing module.
- **Mutating shared state in modules** — Multiple consumers affect each other.

### Best Practices

- Use factory functions when you need multiple independent instances.
- Use the module pattern (IIFE) for singletons.
- Prefer closures over classes for simple state encapsulation.
- Return immutable views of internal state to prevent unintended mutation.
- Name factory functions clearly (e.g., `createStore`, `makeBuilder`).

### Performance Considerations

- Factory functions create closures per instance — hundreds of thousands may cause GC pressure.
- The module pattern creates one closure — negligible overhead.
- Using `Object.freeze()` on returned objects prevents mutation but has runtime cost.
- Closures captured in hot loops should be minimized or hoisted outside.

### Interview Questions

1. **What is the difference between a factory function and a constructor function?**
   Factory functions return an object literal (no `new` required), while constructors use `new` and set properties on `this`.

2. **How does the module pattern provide privacy?**
   Variables inside the IIFE are not accessible from the outside; only the returned object's methods (closures) can access them.

3. **When would you choose a factory over a class?**
   When you need simple encapsulation without inheritance, or when you need to compose behaviors.

### Coding Challenges

1. **Build a rate limiter factory that limits how many times a function can be called per interval.**
2. **Create a simple Pub/Sub module using the revealing module pattern.**
3. **Write a factory that produces functions for retrying async operations with exponential backoff.**

### Related Topics

- IIFE
- Factory pattern
- Module pattern
- Revealing module pattern
- Dependency injection
- Fluent API design

---

## Memory implications

### What It Is

Closures have direct memory implications because they prevent the garbage collector from reclaiming the lexical environments they reference. Every variable captured by a closure remains in memory for as long as the closure exists, potentially keeping large data structures alive much longer than necessary.

```javascript
function createLargeClosure() {
  const largeData = new Array(1000000).fill("data");
  const important = "I need this";
  return function () {
    return important; // only 'important' is used
    // but 'largeData' is kept alive too
  };
}
```

### Why It Is Important

Understanding the memory implications of closures helps developers:
- Avoid accidental memory leaks in long-running applications
- Optimize memory usage in performance-critical code
- Debug issues with growing heap sizes
- Design APIs that are memory-efficient
- Understand garbage collection behavior in event listeners and callbacks

Memory leaks from closures are among the most common JavaScript production issues.

### How It Works Internally

When a closure is created, the engine must decide which variables from the outer scope to keep alive. Modern engines (V8) apply **optimized variable analysis**: they determine exactly which variables are referenced by inner closures and only keep those alive. However, if any inner closure references any outer variable, the entire scope object may still be retained in some cases, especially with `eval`, `with`, or `new Function()`.

```javascript
// V8 optimization: only 'b' is kept alive
function outer() {
  const a = new Array(1000000); // May be GC'd
  const b = "small";
  const c = new Array(1000000); // May be GC'd
  return function () {
    return b;
  };
}
// However, in practice, the entire environment may be retained
// if the engine cannot prove which variables are not referenced.
```

### Syntax

Memory is managed implicitly. The concern is in how closures are created and used:

```javascript
// Problematic: keeps entire scope alive
function addHandler() {
  const bigData = new Array(100000).fill("*");
  element.addEventListener("click", function () {
    console.log("clicked");
    // bigData is NOT used here, but is kept alive
  });
}

// Better: reference only what's needed
function addHandlerFixed() {
  const bigData = new Array(100000).fill("*");
  // Explicitly null the reference after use
  element.addEventListener("click", function () {
    console.log("clicked");
  });
  bigData = null; // Now GC can collect it
}
```

### Beginner Examples

```javascript
// Example 1: Closure preventing GC
function leakExample() {
  const heavy = new Array(1000000).join("*");
  return function () {
    console.log("I don't use heavy, but keep it alive");
  };
}

// Example 2: Subtle memory leak with DOM
function setupButton(button) {
  const data = { id: button.id, heavy: new Array(100000) };
  button.addEventListener("click", function () {
    console.log(data.id); // data.heavy is also preserved
  });
}
// When the button is removed from DOM, the event listener
// and data are NOT GC'd because the listener closure
// still references data.
```

### Intermediate Examples

```javascript
// Example 1: Accumulating closure references
function createPollingService() {
  const results = [];
  return {
    addResult: function (result) {
      results.push(result);
    },
    getResults: function () {
      return results;
    },
    clearResults: function () {
      results.length = 0; // Proper cleanup
    },
  };
}

// Example 2: Closure cache without cleanup
function createExpensiveCache() {
  const cache = new Map();
  return {
    get: function (key) {
      if (cache.has(key)) return cache.get(key);
      const value = performExpensiveComputation(key);
      cache.set(key, value);
      return value;
    },
    // Missing: cache eviction or size limit
  };
}

// Example 3: Event listener leak
class Component {
  constructor(element) {
    this.element = element;
    this.handleClick = () => {
      console.log(this.data); // closes over `this`
    };
    element.addEventListener("click", this.handleClick);
  }
  destroy() {
    // Must remove listener for GC
    this.element.removeEventListener("click", this.handleClick);
  }
}
```

### Advanced Examples

```javascript
// Example 1: WeakMap for zero-overhead caching
const memo = new WeakMap();
function processObject(obj) {
  if (memo.has(obj)) return memo.get(obj);
  const result = heavyComputation(obj);
  memo.set(obj, result);
  return result;
}
// When obj is GC'd, the cache entry disappears automatically.

// Example 2: Closure scope inspection with DevTools
// In Chrome DevTools, Memory tab -> Take Heap Snapshot
// Look for "closure" in the constructor list
// to find unexpected retained objects.

// Example 3: Self-healing through dereferencing
function createSelfCleaningClosure() {
  let data = loadLargeData();
  const fn = function () {
    const result = data.process();
    data = null; // Release after first use
    return result;
  };
  return fn;
}
```

### Real-World Use Cases

- **React useEffect cleanup** — Returning a cleanup function ensures closures are properly released.
- **Angular change detection** — Expressions are evaluated in closures that must be cleaned up.
- **Single Page Applications** — Navigation between pages must destroy old closures (event listeners, subscriptions).
- **WebSocket connections** — Message handlers hold closures over large data structures.
- **Intersection/Mutation Observers** — Observer callbacks keep their captured scope alive.

```javascript
// React example of proper cleanup
useEffect(() => {
  const subscription = dataSource.subscribe(handleData);
  return () => subscription.unsubscribe(); // Cleanup
}, []);
```

### Common Mistakes

- **Creating closures inside loops that accumulate references** — Arrays growing unbounded.
- **Not removing event listeners** — The most common closure leak in frontend apps.
- **Circular references between DOM and closures** — The classic IE6-7 leak, still possible.
- **Storing large data in module-level closures** — Data lives for the entire app lifetime.
- **Assuming `null` assignment is enough** — If other references exist, `null` won't help.

### Best Practices

- Always remove event listeners when the element is removed (use a `destroy()` method).
- Use `WeakMap`/`WeakSet` for caches to avoid preventing GC of keys.
- Limit the scope of captured variables — capture only what you need.
- Use `AbortController` for fetch/signal-based cleanup.
- Profile memory with Chrome DevTools Heap Snapshots to find unexpected closures.
- Implement cache eviction policies (LRU, TTL) for closure-based caches.

### Performance Considerations

- Closures keep their entire (optimized) scope alive — each closure adds memory overhead.
- DOM event listener closures can keep detached DOM subtrees alive.
- Modern engines optimize by splitting environments, but `eval`/`with` disable these optimizations.
- The cost of creating a closure is small, but millions of closures add up.
- Use `FinalizationRegistry` (advanced) to observe when objects are reclaimed.

### Interview Questions

1. **How can closures cause memory leaks in JavaScript?**
   Closures retain references to their lexical environments. If a closure is kept alive (e.g., as an event listener), all variables in its captured scope remain in memory.

2. **How does V8 optimize closure memory?**
   V8 analyzes which variables are actually referenced by closures and creates "context" objects containing only those variables, allowing unrelated variables to be GC'd.

3. **What tools can you use to find closure memory leaks?**
   Chrome DevTools Memory tab (Heap Snapshots), Performance monitor, and `performance.memory` API.

4. **How does WeakMap help with closure memory issues?**
   WeakMap keys are garbage collected when no other references exist, so closures using WeakMap don't prevent GC.

### Coding Challenges

1. **Create a closure-based cache that automatically evicts entries after a TTL.**
2. **Build a function that detects whether a given closure retains unnecessary variables.**
3. **Write an EventEmitter that doesn't leak memory when listeners are removed.**

### Related Topics

- Garbage collection
- Memory management
- WeakMap and WeakSet
- Event listener lifecycle
- Detached DOM nodes
- `FinalizationRegistry`
