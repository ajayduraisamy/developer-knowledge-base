# Module Pattern - IIFE, revealing module pattern, ES6 modules vs classic

## Introduction

The Module Pattern is one of the most important design patterns in JavaScript, providing a way to encapsulate private state and expose a controlled public API. Before ES6 modules became native, developers relied on Immediately Invoked Function Expressions (IIFEs) and the revealing module pattern to create modular code. Today, ES6 modules are the standard, but understanding the classic patterns remains essential for maintaining legacy codebases and deeply grasping JavaScript's scoping mechanisms.

This file covers three interrelated concepts: the IIFE pattern that powers classic modules, the revealing module pattern that refines the API surface, and a thorough comparison between ES6 modules and their classic predecessors.

## IIFE (Immediately Invoked Function Expression)

### What It Is

An IIFE is a JavaScript function that is defined and executed immediately after creation. It is a function expression that is invoked right where it is defined, creating an isolated scope that prevents variable leakage into the global environment.

### Why It Is Important

IIFEs are the foundation of the classic module pattern. They allow developers to create private scopes before JavaScript had block-scoped variables (`let`/`const`) or native modules. Libraries like jQuery used IIFEs to avoid polluting the global namespace while exposing a controlled API via `window.$`.

### How It Works Internally

When the JavaScript engine encounters an IIFE, it parses the function expression, creates a new execution context with its own variable environment, and immediately invokes it. The function's scope chain is established, meaning any variables declared inside are not accessible from the outer scope. The parentheses around the function expression tell the parser to treat the `function` keyword as an expression rather than a declaration.

### Syntax

```javascript
// Basic IIFE syntax
(function() {
  // private scope
})();

// With arrow function
(() => {
  // private scope
})();

// With parameters
(function(name) {
  console.log(`Hello, ${name}`);
})("World");

// With a return value
const result = (function(a, b) {
  return a + b;
})(3, 4);
// result = 7
```

### Beginner Examples

```javascript
// Simple IIFE to avoid global variable pollution
(function() {
  const message = "This is private";
  console.log(message);
})();

// Trying to access the variable outside
// console.log(message); // ReferenceError: message is not defined

// IIFE with parameters
const config = (function(env) {
  const apiUrl = env === "production"
    ? "https://api.example.com"
    : "http://localhost:3000";
  return { apiUrl, env };
})("development");

console.log(config.apiUrl); // http://localhost:3000
```

### Intermediate Examples

```javascript
// Creating a module with public and private members
const counterModule = (function() {
  // Private variable
  let count = 0;

  // Private function
  function validate(value) {
    return typeof value === "number" && value >= 0;
  }

  // Public API
  return {
    increment() {
      count++;
      return count;
    },
    decrement() {
      if (count > 0) {
        count--;
      }
      return count;
    },
    getCount() {
      return count;
    },
    reset() {
      count = 0;
    }
  };
})();

console.log(counterModule.increment()); // 1
console.log(counterModule.increment()); // 2
console.log(counterModule.getCount()); // 2
// console.log(counterModule.count); // undefined (private)
```

### Advanced Examples

```javascript
// IIFE with module augmentation (adding features over time)
const store = (function() {
  const state = {};
  const listeners = [];

  return {
    getState() {
      return { ...state };
    },
    setState(key, value) {
      state[key] = value;
      listeners.forEach(fn => fn(state));
    },
    subscribe(fn) {
      listeners.push(fn);
      return () => {
        const idx = listeners.indexOf(fn);
        if (idx > -1) listeners.splice(idx, 1);
      };
    }
  };
})();

// Augmenting the module
const extendedStore = (function(original) {
  const originalSetState = original.setState;

  original.setState = function(key, value) {
    console.log(`State change: ${key} = ${value}`);
    originalSetState.call(this, key, value);
  };

  return original;
})(store);

// Self-contained async IIFE for data initialization
(async () => {
  try {
    const response = await fetch("https://api.example.com/data");
    const data = await response.json();
    console.log("Initialized with data:", data);
  } catch (err) {
    console.error("Initialization failed:", err);
  }
})();
```

### Real-World Use Cases

- **Library wrappers**: jQuery, Lodash, and many legacy libraries wrap their code in IIFEs to avoid global pollution.
- **Configuration initialization**: Loading environment-specific configuration without leaving temporary variables in scope.
- **Async data initialization**: Running `async` IIFEs at module load time to fetch initial data.
- **Plugin systems**: Creating self-contained plugin scopes that don't interfere with each other.

### Common Mistakes

```javascript
// Mistake 1: Forgetting the invocation parentheses
(function() {
  console.log("This won't run");
}); // Just defines, doesn't execute

// Mistake 2: Missing parentheses around function expression
function() {
  console.log("Syntax error");
}(); // SyntaxError: function statement requires a name

// Mistake 3: Semicolon issues (can cause errors with preceding code)
const x = 1
(function() {
  console.log("Unintended invocation");
})(); // x is treated as 1(function(){...})() - TypeError

// Mistake 4: Not returning the public API
const module = (function() {
  const secret = "hidden";
  // Forgot to return an object - module is undefined
})();
```

### Best Practices

```javascript
// Always use semicolons before IIFEs to avoid automatic semicolon insertion issues
;(function() {
  // safe IIFE
})();

// Use arrow function IIFEs for cleaner syntax
const api = ((baseUrl) => {
  const fetchData = (endpoint) => fetch(`${baseUrl}${endpoint}`);
  return { fetchData };
})("https://api.example.com");

// Use the module pattern consistently with clear public/private separation
// Name the IIFE for better stack traces (optional but helpful)
(function myModule() {
  // named IIFE shows in debugger stack traces
})();
```

### Performance Considerations

- IIFEs create a new function scope and execution context, which has a minimal cost on creation but is negligible for typical usage.
- Modern JavaScript engines optimize IIFEs well; they do not cause memory leaks as long as closures are not holding references to large objects unnecessarily.
- Avoid creating IIFEs inside hot loops; hoist them outside or restructure.

### Interview Questions

**Q: How does an IIFE differ from a regular function declaration?**
A: A function declaration is hoisted and available throughout its containing scope, while an IIFE is an expression that executes immediately and does not pollute the enclosing scope. IIFEs cannot be called later because they don't bind to a name in the scope.

**Q: What problem does the IIFE pattern solve?**
A: It solves the problem of variable scoping before `let`/`const` existed. By creating a new function scope, IIFEs prevent variables from leaking into the global or parent scope, which was critical for avoiding naming collisions in JavaScript applications.

**Q: What is the purpose of wrapping an IIFE in parentheses?**
A: The parentheses tell the JavaScript parser to treat the `function` keyword as a function expression rather than a function declaration. Function declarations require a name and cannot be immediately invoked; wrapping in parentheses converts it to an expression.

### Coding Challenges

```javascript
// Challenge 1: Create a configuration module using IIFE that stores API keys
// and provides methods to get and set them securely.

// Challenge 2: Build a simple event bus using IIFE with on, off, and emit methods.

// Challenge 3: Augment the module from Challenge 2 with a once method
// without modifying the original IIFE code.

// Challenge 4: Create an IIFE that initializes a web app,
// fetches user data, and provides a getUser function.
```

### Related Topics

- Closures (IIFEs use closures to maintain private state)
- Scope and hoisting
- ES6 modules
- Block scoping (`let`/`const`)

## Revealing Module Pattern

### What It Is

The revealing module pattern is a variation of the module pattern where the public API is defined at the return statement by mapping private functions to public names. Instead of scattering `this` references or inline function definitions in the return object, all functions are defined privately and then selectively exposed.

### Why It Is Important

This pattern provides a cleaner, more readable API surface. It makes it immediately obvious which functions are public and which are private. It also allows renaming private functions for the public interface, providing abstraction over implementation details.

### How It Works Internally

The revealing module pattern works exactly like the classic module pattern internally — it uses an IIFE with closure. The difference is purely structural: all functions are defined as private variables, and the return object "reveals" pointers to those private functions under chosen public names.

### Syntax

```javascript
const moduleName = (function() {
  // Private variables and functions
  let privateVar = 0;

  function privateFunction() {
    return privateVar;
  }

  function publicFunction() {
    return privateFunction();
  }

  // Reveal public pointers to private functions
  return {
    publicMethod: publicFunction,
    // Can also rename for the API
  };
})();
```

### Beginner Examples

```javascript
const calculator = (function() {
  let result = 0;

  function add(x) {
    result += x;
  }

  function subtract(x) {
    result -= x;
  }

  function multiply(x) {
    result *= x;
  }

  function divide(x) {
    if (x !== 0) result /= x;
  }

  function getResult() {
    return result;
  }

  function clear() {
    result = 0;
  }

  // Reveal public API
  return {
    add,
    subtract,
    multiply,
    divide,
    getResult,
    clear
  };
})();

calculator.add(10);
calculator.subtract(3);
calculator.multiply(2);
console.log(calculator.getResult()); // 14
```

### Intermediate Examples

```javascript
const userStore = (function() {
  // Private data
  const users = new Map();

  // Private validation
  function isValidEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }

  function isDuplicateEmail(email) {
    return Array.from(users.values()).some(u => u.email === email);
  }

  // Private implementations
  function addUserImpl(id, name, email) {
    if (!isValidEmail(email)) {
      throw new Error("Invalid email format");
    }
    if (isDuplicateEmail(email)) {
      throw new Error("Email already exists");
    }
    users.set(id, { id, name, email, createdAt: new Date() });
    return true;
  }

  function getUserImpl(id) {
    return users.get(id) || null;
  }

  function removeUserImpl(id) {
    return users.delete(id);
  }

  function getAllUsersImpl() {
    return Array.from(users.values());
  }

  // Reveal only the intended public API
  return {
    addUser: addUserImpl,
    getUser: getUserImpl,
    removeUser: removeUserImpl,
    getAllUsers: getAllUsersImpl
    // isValidEmail, isDuplicateEmail, users are all private
  };
})();
```

### Advanced Examples

```javascript
const dataService = (function() {
  // Private cache
  const cache = new Map();
  const pendingRequests = new Map();

  // Private helpers
  function generateCacheKey(url, params) {
    return `${url}?${JSON.stringify(params)}`;
  }

  function isCacheExpired(entry, ttl) {
    return Date.now() - entry.timestamp > ttl;
  }

  async function fetchWithRetry(url, options, retries = 3) {
    for (let i = 0; i < retries; i++) {
      try {
        const response = await fetch(url, options);
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return await response.json();
      } catch (err) {
        if (i === retries - 1) throw err;
        await new Promise(r => setTimeout(r, 1000 * (i + 1)));
      }
    }
  }

  // Core implementation
  async function fetchDataImpl(url, params = {}, options = {}) {
    const cacheKey = generateCacheKey(url, params);
    const cached = cache.get(cacheKey);
    const ttl = options.ttl || 60000;

    if (cached && !isCacheExpired(cached, ttl)) {
      return cached.data;
    }

    // Deduplicate concurrent requests for the same URL
    if (pendingRequests.has(cacheKey)) {
      return pendingRequests.get(cacheKey);
    }

    const queryString = Object.entries(params)
      .map(([k, v]) => `${k}=${encodeURIComponent(v)}`)
      .join("&");

    const fullUrl = queryString ? `${url}?${queryString}` : url;
    const promise = fetchWithRetry(fullUrl, {}, options.retries);

    pendingRequests.set(cacheKey, promise);
    const data = await promise;
    pendingRequests.delete(cacheKey);

    cache.set(cacheKey, { data, timestamp: Date.now() });
    return data;
  }

  function invalidateCacheImpl(url, params) {
    const cacheKey = generateCacheKey(url, params);
    cache.delete(cacheKey);
  }

  function clearCacheImpl() {
    cache.clear();
  }

  // Reveal public API
  return {
    fetchData: fetchDataImpl,
    invalidateCache: invalidateCacheImpl,
    clearCache: clearCacheImpl,
    // fetchWithRetry, isCacheExpired, generateCacheKey are private
  };
})();
```

### Real-World Use Cases

- **API service layers**: Wrapping HTTP requests with caching, retry logic, and authentication.
- **State management stores**: Redux-like stores where the state is private and only actions are exposed.
- **Configuration managers**: Reading from environment variables and exposing only necessary config.
- **Logger utilities**: Providing `info`, `warn`, `error` while hiding log level internals.

### Common Mistakes

```javascript
// Mistake 1: Mutating private values from outside
const module = (function() {
  const private = { data: "sensitive" };
  return {
    getData: () => private
  };
})();
module.getData().data = "hacked"; // Works! Avoid returning references to mutable objects without copying

// Mistake 2: Inconsistent casing between private and public API
const bad = (function() {
  function GET_DATA() { /* ... */ }
  return {
    getData: GET_DATA // Inconsistent naming
  };
})();

// Mistake 3: Forgetting that functions are references
function addListener(fn) {
  listeners.push(fn);
}
// If you later reassign `addListener`, the original reference in listeners is unaffected
```

### Best Practices

```javascript
// Always return copies of mutable objects to prevent external mutation
const safeModule = (function() {
  const state = { data: "important" };
  return {
    getState: () => ({ ...state }),
    setState: (newData) => { state.data = newData; }
  };
})();

// Keep private and public naming consistent for readability
// Use descriptive names for private implementations
function computeUserPermissionsImpl(user) { /* ... */ }
return {
  computeUserPermissions: computeUserPermissionsImpl
};
```

### Performance Considerations

- The revealing module pattern has the same performance characteristics as the classic module pattern — zero runtime overhead once initialized.
- There is a one-time cost at module load for function creation, but this is negligible.
- Object spreading in getters (`{ ...state }`) creates shallow copies; for performance-critical paths, consider providing immutable data structures or selective accessors.

### Interview Questions

**Q: How is the revealing module pattern different from the classic module pattern?**
A: In the classic module pattern, public methods are defined inline in the return object. In the revealing module pattern, all methods are defined as private variables, and the return object only contains references to them. This makes the public API more explicit and easier to read.

**Q: What are the advantages of the revealing module pattern?**
A: (1) Consistent syntax — all functions are defined the same way. (2) Clear distinction between public and private. (3) Easy to rename or alias functions for the public API. (4) Better readability at the return statement.

**Q: What is a disadvantage of the revealing module pattern?**
A: If a private function references another private function, and you override the public version later, the private reference still points to the original function. This can lead to unexpected behavior if the module is augmented or patched.

### Coding Challenges

```javascript
// Challenge 1: Create a shopping cart module using revealing module pattern.
// It should have addItem, removeItem, getTotal, and checkout methods,
// while hiding the items array and discount logic.

// Challenge 2: Build a logger module that supports log levels (info, warn, error)
// and can have the log level changed at runtime. Keep the level storage private.

// Challenge 3: Create a rate limiter module that tracks API call frequency
// and blocks calls exceeding the limit. Expose canCall(url) and recordCall(url).
// Hide the internal call tracking Map.
```

### Related Topics

- Module pattern
- Closures (enables private state)
- Object property descriptors (for stricter encapsulation)
- ES6 modules (modern alternative)

## ES6 Modules Comparison

### What It Is

ES6 (ES2015) introduced native module support to JavaScript via `import` and `export` keywords. ES6 modules are statically analyzable, have their own scope, and are executed in strict mode by default. They provide a standardized, built-in module system that replaces the need for IIFE-based patterns.

### Why It Is Important

ES6 modules solved the long-standing problem of dependency management in JavaScript. They provide:
- Static analysis for tree shaking
- Cyclic dependency handling
- Named and default exports
- Top-level `await`
- Better tooling support (IDE autocompletion, type checking)

### How It Works Internally

ES6 modules are parsed before execution, allowing the JavaScript engine to build a dependency graph. The engine determines imports and exports statically (not at runtime), which enables optimizations like dead code elimination. Modules are singletons — they are only executed once, and subsequent imports receive the same instance.

The module resolution process involves:
1. **Module resolution**: The specifier string is resolved to a file URL.
2. **Parsing**: The module is parsed, and all `import`/`export` statements are identified.
3. **Graph building**: A dependency graph is constructed.
4. **Evaluation**: Modules are executed in dependency order (depth-first).

### Syntax

```javascript
// Named exports - moduleA.js
export const PI = 3.14159;
export function double(x) { return x * 2; }
export class Calculator { /* ... */ }

// Or export separately
const E = 2.71828;
function triple(x) { return x * 3; }
export { E, triple };

// Default export - moduleB.js
export default function greet(name) {
  return `Hello, ${name}`;
}

// Named import
import { PI, double } from "./moduleA.js";

// Default import
import greet from "./moduleB.js";

// Mixed import
import greet, { PI, double } from "./modules.js";

// Namespace import
import * as math from "./moduleA.js";
// math.PI, math.double(5)

// Re-exporting
export { double as duplicate } from "./moduleA.js";
export * from "./moduleA.js";
```

### Beginner Examples

```javascript
// math.js
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
export const VERSION = "1.0.0";

// app.js
import { add, subtract, VERSION } from "./math.js";
console.log(add(5, 3)); // 8
console.log(VERSION); // 1.0.0

// default example
// utils.js
export default function formatDate(date) {
  return date.toISOString().split("T")[0];
}

// app.js
import formatDate from "./utils.js";
console.log(formatDate(new Date())); // 2026-06-09
```

### Intermediate Examples

```javascript
// api.js - Re-exporting and aggregation
export { default as createUser } from "./services/userService.js";
export { default as authMiddleware } from "./middleware/auth.js";
export { validateEmail, validatePassword } from "./validators.js";
export { config } from "./config.js";

// Dynamic imports (lazy loading)
// dashboard.js
export function loadDashboard() {
  return import("./modules/dashboard.js").then(mod => {
    mod.initDashboard();
  });
}

// Circular dependency handling
// a.js
import { b } from "./b.js";
export function a() {
  console.log("a called");
  b();
}

// b.js
import { a } from "./a.js";
export function b() {
  console.log("b called");
  a(); // Works fine with ES6 modules (not with CommonJS)
}
```

### Advanced Examples

```javascript
// Dynamic import with error handling and fallback
async function loadLocale(locale) {
  try {
    const messages = await import(`./locales/${locale}.js`);
    return messages.default;
  } catch (err) {
    console.warn(`Locale ${locale} not found, falling back to en`);
    const fallback = await import("./locales/en.js");
    return fallback.default;
  }
}

// Top-level await
// config.js
const response = await fetch("/api/config");
export const config = await response.json();

// Importing with side effects
import "./polyfills.js"; // Runs the module for its side effects only

// Module aggregation with renaming
// index.js - Barrel file
export {
  UserService as UserServiceInterface,
  AuthService
} from "./services/index.js";

export {
  createValidator,
  validate as validateInput
} from "./validation/index.js";

// Using import.meta for module metadata
if (import.meta.url.includes("localhost")) {
  console.log("Running in development mode");
}

// Live bindings example
// counter.js
export let count = 0;
export function increment() {
  count++; // The change is reflected in all importing modules
}

// app.js
import { count, increment } from "./counter.js";
console.log(count); // 0
increment();
console.log(count); // 1 (live binding)
```

### Real-World Use Cases

- **Micro-frontends**: Each micro-frontend is a self-contained module loaded dynamically.
- **Library publishing**: NPM packages use ES6 modules for tree-shaking compatibility (via `"module"` field in `package.json`).
- **Lazy loading**: Route-based code splitting in frameworks like React and Vue.
- **Feature flags**: Conditionally importing different implementations based on environment.

### Common Mistakes

```javascript
// Mistake 1: Trying to use import/export inside a condition
if (condition) {
  import("./module.js"); // SyntaxError: 'import' and 'export' may only appear at the top level
}

// Mistake 2: Forgetting .js extension in Node.js
import { something } from "./module"; // May fail - Node requires extensions for ES modules
// Correct:
import { something } from "./module.js";

// Mistake 3: Mutating imported bindings
import { config } from "./config.js";
config = { new: "value" }; // TypeError: Assignment to constant variable

// But mutation through methods works
import { obj } from "./module.js";
obj.key = "new value"; // This works but is often bad practice

// Mistake 4: Mixing default and named exports inconsistently
export default function myFunc() {}
export myFunc; // Multiple default exports - SyntaxError

// Mistake 5: Circular dependency with named exports in CommonJS
// Not an issue with ES6 modules, but common mistake when migrating
```

### Best Practices

```javascript
// Prefer named exports over default exports for better refactoring and tree shaking
// Good
export { Button, Input, Select };

// Avoid
export default Button;

// Use barrel files (index.js) for clean public API surfaces
// Group related exports and re-export from a central file

// Use dynamic imports for code splitting, not as a replacement for static imports
// Static imports should be the default

// Keep modules small and focused (single responsibility)
// A module should export one primary concern

// Use import.meta.env or env variables instead of process.env for browser compatibility
```

### Performance Considerations

- **ES6 modules enable tree shaking**: Tools like Webpack and Rollup can eliminate unused exports because `import`/`export` are statically analyzable.
- **Dynamic imports** split code into separate chunks, reducing initial bundle size.
- **Module evaluation** happens once per module. Subsequent imports from the same module receive cached exports (live bindings).
- **HTTP/2 multiplexing** makes many small modules more efficient than a few large bundles.
- **Side effect flags**: Use `"sideEffects": false` in `package.json` to enable deeper tree shaking.

### Comparison: Classic Module Pattern vs ES6 Modules

| Feature | Classic Module (IIFE/Revealing) | ES6 Modules |
|---------|--------------------------------|-------------|
| Scope | Function scope | Module scope |
| Strict mode | Optional | Always strict |
| Static analysis | No | Yes |
| Tree shaking | No | Yes |
| Cyclic dependencies | Not supported | Supported |
| Top-level await | Not supported | Supported |
| Live bindings | No (copies) | Yes (live references) |
| Async loading | Manual | Native via dynamic `import()` |
| Browser support | All browsers | Modern browsers only |
| Tooling support | Limited | Full (autocomplete, type checking) |

```javascript
// Classic pattern - export is a snapshot copy
const classicModule = (function() {
  let value = 1;
  return {
    getValue: () => value,
    increment: () => ++value
  };
})();
console.log(classicModule.getValue()); // 1
classicModule.increment();
console.log(classicModule.getValue()); // 2

// ES6 module - export is a live binding
// counter.js
export let value = 1;
export function increment() { value++; }

// app.js
import { value, increment } from "./counter.js";
console.log(value); // 1
increment();
console.log(value); // 2 (live binding)
```

### Interview Questions

**Q: What are the key differences between ES6 modules and CommonJS?**
A: ES6 modules are static (imports/exports are determined at parse time), support cyclic dependencies natively, have live bindings, and are always in strict mode. CommonJS is dynamic (requires are runtime), detects cyclic dependencies at runtime, copies values, and is not strict by default.

**Q: How do ES6 modules handle circular dependencies?**
A: ES6 modules handle circular dependencies through live bindings. When module A imports from module B, and B imports from A, the engine provides a reference to the binding that is updated as evaluation progresses. This allows both modules to see each other's exports even if evaluation hasn't completed.

**Q: What is tree shaking and how do ES6 modules enable it?**
A: Tree shaking is dead code elimination where unused exports are removed from the final bundle. ES6 modules enable this because `import` and `export` are static (determined at parse time), allowing tools to trace which exports are actually used and safely remove unused ones.

### Coding Challenges

```javascript
// Challenge 1: Create an ES6 module that exports a function to debounce API calls,
// a function to cancel pending requests, and a configuration object.
// Import and use it in a main module.

// Challenge 2: Set up a barrel file (index.js) that re-exports from three
// sub-modules: validation.js, formatting.js, and storage.js.
// Each sub-module should export at least two named exports.

// Challenge 3: Create a module that uses dynamic imports to load
// different renderers (svgRenderer.js, canvasRenderer.js, webGLRenderer.js)
// based on a config value. Include error handling for unsupported renderers.
```

### Related Topics

- CommonJS modules (Node.js require/exports)
- AMD/RequireJS (historical module systems)
- Dynamic `import()`
- Module bundlers (Webpack, Rollup, ESBuild)
- `package.json` module resolution
