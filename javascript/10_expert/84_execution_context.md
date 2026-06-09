# Execution Context - Global context, function context, creation/execution phases, scope chain

## Introduction

Execution context is the abstract environment in which JavaScript code is evaluated and executed. Every time JavaScript code runs, it does so within an execution context. Understanding execution contexts is fundamental to mastering closures, the `this` keyword, hoisting, and scope—concepts that underpin all JavaScript development. There are three types: global execution context (GEC), function execution context (FEC), and eval execution context.

## Global Execution Context

### What It Is

The Global Execution Context is the default context created when JavaScript code first starts running. It represents the global scope—the outermost environment where code that is not inside any function executes. In browsers, the global context creates the `window` object; in Node.js, it creates the `global` object. There is only one global execution context throughout the lifetime of a JavaScript program.

### Why It Is Important

The global execution context is the root of the scope chain. All other execution contexts (function, eval) are created within this context. Variables declared in the global scope are accessible everywhere. Understanding the global context helps developers avoid polluting the global namespace, which can cause conflicts and memory leaks.

### How It Works Internally

When the JavaScript engine starts, it creates the global execution context in two phases:

1. **Creation phase**: The engine creates the global object (`window`/`global`), sets up `this` to reference the global object, and allocates memory for function declarations (fully hoisted) and variable declarations (`var` hoisted with `undefined`, `let`/`const` in TDZ).

2. **Execution phase**: Code is executed line by line. Variable assignments occur, functions are called, and the scope chain is traversed for identifier resolution.

### Syntax

```javascript
// Global scope
const globalVar = 'accessible everywhere';

function globalFunction() {
  console.log('Also global');
}

console.log(this); // Window (browser) or global (Node.js)
console.log(globalVar); // 'accessible everywhere'
```

### Beginner Examples

```javascript
// Global context in browser
console.log(this === window); // true

var globalVar = 'I am global';
let globalLet = 'I am also global'; // Not on window!

function globalFunc() {
  console.log('I am a hoisted function');
}

console.log(window.globalVar); // 'I am global' (var creates property)
console.log(window.globalLet); // undefined (let does not)
```

### Intermediate Examples

```javascript
// Understanding global vs module scope (Node.js)
// In Node.js, each module has its own scope, not the global scope

// module.js
const moduleVar = 'module scoped'; // NOT global

global.shared = 'truly global'; // Explicitly on global object

// In another module:
console.log(global.shared); // 'truly global'
console.log(moduleVar); // ReferenceError: not accessible

// The global object in different environments
// Browser: window (or globalThis)
// Node.js: global
// Web Worker: self
// Universal: globalThis (ES2020)
```

### Advanced Examples

```javascript
// Global context pollution and its effects
// Polluting the global object is dangerous

// Bad practice
var setTimeout = 'oops'; // Overwrites window.setTimeout!
// TypeError: setTimeout is not a function

// Strict mode prevents some issues
'use strict';
mistypeVaraible = 17; // ReferenceError (not creating global property)

// Implicit global creation (non-strict)
function setGlobal() {
  implicitGlobal = 'created on window'; // No var/let/const!
}
setGlobal();
console.log(window.implicitGlobal); // 'created on window'

// GlobalThis for universal reference
console.log(globalThis); // Works everywhere
```

### Real-World Use Cases

- Configuring global error handlers (`window.onerror`, `process.on('uncaughtException')`)
- Setting up polyfills and feature detection
- Third-party script integration (careful not to pollute)
- Global configuration and constants
- Implementing the Module pattern via IIFE to avoid global scope pollution

### Common Mistakes

- Polluting the global scope with too many variables
- Forgetting `var`/`let`/`const` and accidentally creating globals
- Assuming Node.js modules share the global scope like browser scripts
- Using `var` in the global scope (creates window properties)
- Confusing the global object with the global scope

### Best Practices

- Minimize global variables; use modules instead
- Use `const` and `let` instead of `var`
- Enable strict mode to catch accidental globals
- Use `globalThis` for cross-environment global access
- Wrap code in IIFEs or modules for encapsulation
- Use the Module pattern for namespacing

### Performance Considerations

- Global variables are slower to access than local variables because the scope chain must be traversed to the outermost level
- Each global `var` declaration creates a property on the global object, which can cause hidden class transitions on the global object itself
- Unused global variables are never garbage collected as long as the global context exists (until page unload)
- Implicit globals (assigning to undeclared variables) are 2-3x slower than declared local variables due to scope chain traversal and global object property creation
- The global execution context persists for the entire application lifetime, so closures referencing it never have their scope freed

### Interview Questions

**Q: How does the global execution context differ from function execution contexts?**
A: There is only one global execution context for the entire program lifetime, created when the script starts. Function execution contexts are created and destroyed with each function call. The global context creates the global object (`window`/`global`), sets `this` to it, and is the root of the scope chain. Function contexts have their own variable environment, `arguments` object, and `this` binding determined by how the function is called.

**Q: What is the difference between `var`, `let`, and `const` in the global context?**
A: `var` declarations at the global level create properties on the global object (`window`/`global`), making them accessible via both the variable name and as properties of the global object. `let` and `const` create variables in the global scope but do NOT create properties on the global object. Additionally, `let`/`const` are subject to the Temporal Dead Zone (TDZ), while `var` is hoisted and initialized as `undefined`.

**Q: How do you avoid polluting the global scope?**
A: Use ES modules (each module has its own scope), use IIFEs to create closure scopes, minimize use of global variables, use `const`/`let` instead of `var`, enable strict mode, use the Module pattern or Revealing Module pattern, and prefer passing dependencies explicitly rather than accessing them globally.

**Q: What is `globalThis` and why is it useful?**
A: `globalThis` is a standard ES2020 property that returns the global object regardless of the environment. In browsers it returns `window`, in Node.js it returns `global`, in Web Workers it returns `self`. Before `globalThis`, developers had to use environment detection (`typeof window !== 'undefined' ? window : global`) to access the global object.

**Q: How does the global execution context work in Node.js vs browsers?**
A: In browsers, the global execution context creates the `window` object, and top-level `var` declarations become properties on `window`. In Node.js, each file is treated as a module with its own scope, so top-level declarations are module-scoped, not global. To create truly global variables in Node.js, you must explicitly assign to `global` or `globalThis`.

### Coding Challenges

**Challenge 1: Build a sandbox that prevents global scope pollution**
```javascript
// Create a sandbox function that executes code in an isolated scope,
// preventing accidental global variable creation.

function sandbox(code) {
  const sandboxGlobals = new Proxy({}, {
    has: () => true, // Trap all 'in' checks
    get: (target, key) => {
      if (key === Symbol.unscopables) return undefined;
      // Allow access to safe globals
      const safe = ['console', 'Math', 'JSON', 'Array', 'Object',
        'String', 'Number', 'Boolean', 'Date', 'RegExp', 'Map', 'Set',
        'Promise', 'Error', 'parseInt', 'parseFloat', 'isNaN', 'setTimeout',
        'clearTimeout', 'setInterval', 'clearInterval'];
      if (safe.includes(key) || typeof key === 'symbol') {
        return globalThis[key];
      }
      if (key in globalThis) {
        console.warn(`Sandbox: Access to global "${String(key)}" is restricted`);
        return undefined;
      }
      return globalThis[key];
    },
    set: (target, key) => {
      console.warn(`Sandbox: Preventing global assignment to "${String(key)}"`);
      return true; // Silently ignore
    }
  });

  const sandboxedEval = new Function('sandbox', `with(sandbox) { ${code} }`);
  try {
    sandboxedEval(sandboxGlobals);
  } catch (err) {
    console.error('Sandbox error:', err.message);
  }
}

// Usage:
// sandbox('var x = 10; console.log(x);'); // Works
// sandbox('accidentalGlobal = "leak";'); // Silently prevented
// sandbox('console.log(fetch);'); // Warning about restricted global

// Alternative using IIFE:
function createSandboxedModule() {
  const module = {};
  (function(exports) {
    // All vars here are scoped to this function
    'use strict';
    const privateVar = 'truly private';
    exports.getSecret = () => privateVar;
    // Without 'use strict', assigning to undeclared var would create global
  })(module);
  return module;
}
```

**Challenge 2: Implement a cross-environment global access library**
```javascript
// Build a utility that provides consistent global access across
// browsers, Node.js, Web Workers, and other JS runtimes.

class CrossEnvironment {
  static getGlobalObject() {
    if (typeof globalThis !== 'undefined') return globalThis;
    if (typeof window !== 'undefined') return window;
    if (typeof self !== 'undefined') return self;
    if (typeof global !== 'undefined') return global;
    if (typeof this !== 'undefined') return this;
    throw new Error('Could not determine global object');
  }

  static getGlobal(name) {
    const globalObj = this.getGlobalObject();
    return globalObj[name];
  }

  static setGlobal(name, value) {
    const globalObj = this.getGlobalObject();
    globalObj[name] = value;
  }

  static hasGlobal(name) {
    const globalObj = this.getGlobalObject();
    return name in globalObj;
  }

  static isBrowser() {
    return typeof window !== 'undefined' && typeof window.document !== 'undefined';
  }

  static isNode() {
    return typeof process !== 'undefined' &&
      process.versions != null &&
      process.versions.node != null;
  }

  static isWebWorker() {
    return typeof self !== 'undefined' &&
      typeof WorkerGlobalScope !== 'undefined' &&
      self instanceof WorkerGlobalScope;
  }

  static safeGlobal(name, fallback) {
    try {
      const value = this.getGlobal(name);
      return value !== undefined ? value : fallback;
    } catch {
      return fallback;
    }
  }

  static getGlobalNames() {
    const globalObj = this.getGlobalObject();
    return Object.getOwnPropertyNames(globalObj)
      .filter(name => !name.startsWith('__'));
  }

  static countGlobals() {
    const before = Object.keys(this.getGlobalObject()).length;
    return {
      total: before,
      ownProperties: Object.getOwnPropertyNames(this.getGlobalObject()).length
    };
  }
}
```

**Challenge 3: Debug the global context pollution in a third-party script**
```javascript
// Given a page with third-party scripts, identify and mitigate
// global scope pollution.

class GlobalPollutionDetector {
  constructor() {
    this.baseline = null;
    this.pollution = new Map();
  }

  captureBaseline() {
    this.baseline = this.getGlobalSnapshot();
    console.log(`Baseline: ${this.baseline.size} global properties`);
  }

  getGlobalSnapshot() {
    const snapshot = new Map();
    const g = typeof window !== 'undefined' ? window : global;
    Object.getOwnPropertyNames(g).forEach(key => {
      try {
        const value = g[key];
        snapshot.set(key, typeof value);
      } catch {
        snapshot.set(key, 'inaccessible');
      }
    });
    return snapshot;
  }

  detectPollution() {
    if (!this.baseline) {
      this.captureBaseline();
      return;
    }

    const current = this.getGlobalSnapshot();
    this.pollution.clear();

    for (const [key, type] of current) {
      if (!this.baseline.has(key)) {
        this.pollution.set(key, {
          type,
          source: this.guessSource(key),
          risk: this.assessRisk(key, type)
        });
      }
    }

    return this.generateReport();
  }

  guessSource(key) {
    // Common third-party prefixes
    if (key.startsWith('ga') || key.startsWith('_ga')) return 'Google Analytics';
    if (key.startsWith('fbq') || key.startsWith('_fb')) return 'Facebook Pixel';
    if (key.startsWith('gtag')) return 'Google Tag Manager';
    if (key.startsWith('__utm')) return 'Google Analytics (legacy)';
    if (key.startsWith('_paq')) return 'Matomo/Piwik';
    if (key.startsWith('ya')) return 'Yandex Metrica';
    return 'Unknown third-party script';
  }

  assessRisk(key, type) {
    // High risk: overwriting built-ins or functions
    const builtins = ['document', 'window', 'fetch', 'XMLHttpRequest',
      'setTimeout', 'setInterval', 'addEventListener', 'Promise'];
    if (builtins.includes(key)) return 'critical';
    if (type === 'function' && key.length < 10) return 'high';
    if (type === 'object') return 'medium';
    return 'low';
  }

  generateReport() {
    if (this.pollution.size === 0) {
      return { message: 'No global pollution detected', pollution: [] };
    }

    const entries = Array.from(this.pollution.entries())
      .map(([name, info]) => ({ name, ...info }))
      .sort((a, b) => {
        const riskOrder = { critical: 0, high: 1, medium: 2, low: 3 };
        return riskOrder[a.risk] - riskOrder[b.risk];
      });

    return {
      totalPollutants: entries.length,
      byRisk: {
        critical: entries.filter(e => e.risk === 'critical').length,
        high: entries.filter(e => e.risk === 'high').length,
        medium: entries.filter(e => e.risk === 'medium').length,
        low: entries.filter(e => e.risk === 'low').length
      },
      entries
    };
  }

  suggestCleanup(report) {
    const suggestions = [];
    if (report.byRisk.critical > 0) {
      suggestions.push('Critical globals overwritten - investigate script load order');
    }
    if (report.totalPollutants > 20) {
      suggestions.push('Consider iframe sandboxing for third-party scripts');
    }
    suggestions.push('Use ES modules to isolate third-party code');
    suggestions.push('Wrap legacy scripts in IIFEs');
    return suggestions;
  }
}
```

## Function Execution Context

### What It Is

A function execution context is created each time a function is called (or invoked). Each function call gets its own execution context, which is placed on the call stack. The function execution context has its own variable environment, `this` binding, and outer environment reference (scope chain).

### Why It Is Important

Function execution contexts are the mechanism that enables closures, lexical scoping, and dynamic `this` binding. Each function call creates a new scope with its own variables, allowing for encapsulation and recursion. Understanding function execution contexts is essential for debugging scope-related bugs.

### How It Works Internally

When a function is called, the engine creates a new function execution context in two phases:

**Creation phase**:
1. Creates the `arguments` object (for non-arrow functions)
2. Sets up the `this` binding based on how the function was called
3. Creates a new variable environment (scope object)
4. Allocates memory for local variables (hoisting)
5. Sets the outer environment reference (to the parent scope)
6. Creates the scope chain (current variable environment + outer references)

**Execution phase**:
1. Executes code line by line
2. Assigns values to variables
3. Nested function calls create new execution contexts
4. Returns value or `undefined`, pops context from stack

### Syntax

```javascript
function example(a, b) {
  const c = a + b;
  return c;
}

example(1, 2);
// When called, creates a new execution context with:
// - arguments: { 0: 1, 1: 2, length: 2 }
// - this: depends on call pattern
// - variables: c (initially undefined, then 3)
// - outer scope: global scope
```

### Beginner Examples

```javascript
// Basic function execution context
function greet(name) {
  const message = `Hello, ${name}!`;
  console.log(message);
  return message;
}

greet('Alice'); // Creates FEC for greet
greet('Bob');   // Creates NEW FEC for greet

// Each call is independent
function counter() {
  let count = 0;
  return function() {
    count++;
    return count;
  };
}

const c1 = counter(); // FEC with own count
const c2 = counter(); // Different FEC, different count
console.log(c1()); // 1
console.log(c1()); // 2
console.log(c2()); // 1 (separate execution context)
```

### Intermediate Examples

```javascript
// Arguments object
function showArgs(a, b) {
  console.log(arguments); // [Arguments] { '0': 1, '1': 2 }
  console.log(arguments.length); // 2
  console.log(typeof arguments); // 'object'
  console.log(Array.isArray(arguments)); // false
  console.log(Array.from(arguments)); // [1, 2]
  
  // arguments is array-like but not an array
}

showArgs(1, 2);

// Default parameters create their own scope
function withDefault(x = 5) {
  let x = 10; // SyntaxError: Identifier 'x' already declared
}
// Default parameters are in a separate scope from the function body

// Rest parameters (no arguments object needed)
function withRest(a, b, ...rest) {
  console.log(rest); // Proper array
}

// Recursion creates multiple execution contexts
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}
// Each recursive call creates a new FEC on the stack
factorial(5); // Stack: factorial(5) → factorial(4) → ... → factorial(1)
```

### Advanced Examples

```javascript
// Execution context and closures - how outer reference works
function outer(x) {
  function inner(y) {
    return x + y; // inner's FEC has outer's environment as outer reference
  }
  return inner;
}

const add5 = outer(5);
// outer's FEC should be gone, but inner's closure keeps reference
// inner's FEC's outer environment reference points to outer's variable env
console.log(add5(3)); // 8

// Execution context with this binding demonstration
const obj = {
  name: 'Object',
  method() {
    console.log(this.name);
  }
};

const extracted = obj.method;
extracted(); // undefined (this = global/undefined)
obj.method(); // 'Object' (this = obj)

// Arrow function context
const arrowObj = {
  name: 'Arrow',
  regular: function() {
    console.log(this.name); // own this
  },
  arrow: () => {
    console.log(this.name); // parent's this (lexical)
  }
};

arrowObj.regular(); // 'Arrow'
arrowObj.arrow(); // undefined (this = global or enclosing)

// Execution context with getters/setters
const person = {
  _age: 30,
  get age() {
    return this._age; // this is the object accessed through
  }
};
```

### Real-World Use Cases

- Recursive algorithms (tree traversal, fibonacci, factorial)
- Closure-based patterns (memoization, currying, partial application)
- Method chaining (`this` returning)
- Callback contexts (ensuring correct `this`)
- Functional programming (function composition)

### Common Mistakes

- Assuming `this` inside a function always refers to the function's own scope
- Forgetting that arrow functions don't have their own `this` or `arguments`
- Passing methods as callbacks without binding
- Creating closures in loops with `var`
- Not understanding that default parameters have their own scope

### Best Practices

- Use arrow functions when you want lexical `this`
- Use method shorthand syntax for object methods
- Bind `this` explicitly when passing methods as callbacks
- Use rest parameters (`...args`) instead of `arguments`
- Keep functions pure when possible (no side effects on outer scope)

### Performance Considerations

- Each function call allocates a new execution context, which has overhead (memory and CPU). Avoid creating unnecessary nested functions in hot paths
- Arrow functions are slightly more performant than regular functions in some engines because they skip creating their own `arguments` object and `this` binding
- Deep recursion creates many execution contexts and can exhaust stack memory. For deep recursion, use iterative approaches or trampolining
- Default parameters create a nested scope, adding an extra layer to the scope chain. This adds marginal overhead for each function call with defaults
- Functions with many parameters (>100) may degrade performance as the engine must allocate larger `arguments` objects and parameter slots

### Interview Questions

**Q: What happens when a function is called in JavaScript?**
A: When a function is called, the engine: (1) creates a new function execution context, (2) enters the creation phase (sets up `this`, creates `arguments` object, allocates memory for local variables, establishes the scope chain), (3) enters the execution phase (executes code line by line), and (4) when the function returns, the context is popped from the call stack and destroyed (unless preserved by a closure).

**Q: How does `this` get determined in a function execution context?**
A: `this` is determined by how the function is called, not where it's defined: (1) regular function call: `this` is the global object (or `undefined` in strict mode), (2) method call: `this` is the object the method is called on, (3) constructor with `new`: `this` is the newly created instance, (4) `.call()`/`.apply()`/`.bind()`: `this` is explicitly set, (5) arrow functions: `this` is lexically inherited from the enclosing context.

**Q: How do closures relate to function execution contexts?**
A: A closure is formed when an inner function references variables from an outer function's execution context. Normally, when a function returns, its execution context is popped from the stack and garbage collected. But if an inner function holds a reference to the outer context's variables (via the scope chain), the outer context's variable environment is preserved in memory. The closure keeps the scope chain alive.

**Q: What is the difference between `arguments` and rest parameters?**
A: `arguments` is an array-like object available in all non-arrow functions. It contains all passed arguments regardless of declared parameters. Rest parameters (`...args`) create a real Array containing only the excess parameters beyond the named ones. Rest parameters are preferred because they are real arrays and don't prevent V8 optimizations.

**Q: How does strict mode affect function execution contexts?**
A: Strict mode changes several aspects: (1) `this` is `undefined` in regular function calls instead of the global object, (2) assigning to undeclared variables throws a ReferenceError instead of creating a global, (3) the `arguments` object does not track parameter changes, (4) `arguments.callee` and `arguments.caller` throw errors, (5) duplicate parameter names are not allowed.

### Coding Challenges

**Challenge 1: Build a function call tracer that logs execution contexts**
```javascript
// Create a wrapper that intercepts function calls and logs
// execution context details: arguments, this, scope chain depth.

function createContextTracer(target, name) {
  const stackDepth = new Error().stack?.split('\n').length || 0;

  return new Proxy(target, {
    apply(fn, thisArg, args) {
      const depth = new Error().stack?.split('\n').length - stackDepth || 0;
      const indent = '  '.repeat(depth);

      console.log(`${indent}→ ${name}()`);
      console.log(`${indent}  this:`, thisArg);
      console.log(`${indent}  args:`, args);

      // Track scope chain (simulate by checking parent frames)
      const scopeVars = Object.keys(thisArg || {}).slice(0, 3);
      if (scopeVars.length > 0) {
        console.log(`${indent}  scope vars:`, scopeVars);
      }

      const start = Date.now();
      try {
        const result = fn.apply(thisArg, args);
        console.log(`${indent}← ${name}() =>`, result, `(${Date.now() - start}ms)`);
        return result;
      } catch (err) {
        console.log(`${indent}✕ ${name}() threw:`, err.message);
        throw err;
      }
    },

    construct(fn, args) {
      console.log(`→ new ${name}() with args:`, args);
      const instance = new fn(...args);
      console.log(`← new ${name}() =>`, instance);
      return instance;
    }
  });
}

// Usage:
// class Calculator {
//   add(a, b) { return a + b; }
// }
// const traced = createContextTracer(Calculator.prototype.add, 'Calculator.add');
// traced.call({}, 2, 3);
```

**Challenge 2: Implement a mini-debugger that inspects execution contexts**
```javascript
// Build a debugging utility that can inspect the current execution
// context's variable bindings, scope chain, and this value.

class ExecutionContextInspector {
  constructor() {
    this.frames = [];
    this.maxFrames = 50;
  }

  captureFrame(label) {
    const stack = new Error().stack?.split('\n').slice(2) || [];
    const frame = {
      label: label || 'unknown',
      timestamp: Date.now(),
      depth: stack.length,
      caller: stack[0]?.trim() || 'top level',
      stack: stack.slice(0, 10),
      capturedVars: this.captureVariables()
    };
    this.frames.push(frame);
    if (this.frames.length > this.maxFrames) {
      this.frames.shift();
    }
    return frame;
  }

  captureVariables() {
    // In a real debugger, this would use Debugger API
    // Here we simulate by checking what's accessible
    try {
      const vars = {};
      // Check for common patterns that indicate local variables
      for (const key of Object.keys(globalThis)) {
        if (key.startsWith('_')) continue;
      }
      return vars;
    } catch {
      return {};
    }
  }

  formatStackFrame(frame) {
    return [
      `[${frame.label}]`,
      `  Depth: ${frame.depth}`,
      `  Caller: ${frame.caller}`,
      `  Time: ${new Date(frame.timestamp).toISOString()}`,
      `  Stack:`,
      ...frame.stack.map(s => `    ${s}`)
    ].join('\n');
  }

  printCallStack() {
    console.log('=== Execution Context Trace ===');
    this.frames.forEach((frame, i) => {
      console.log(`\nFrame ${i + 1}:`);
      console.log(this.formatStackFrame(frame));
    });
  }

  analyzeDepth() {
    if (this.frames.length < 2) return;
    const depths = this.frames.map(f => f.depth);
    const max = Math.max(...depths);
    const min = Math.min(...depths);
    console.log(`Stack depth range: ${min} - ${max}`);
    if (max > 100) {
      console.warn('Warning: Deep call stack detected (>100 frames)');
    }
  }
}
```

**Challenge 3: Create a function memoizer that respects execution context**
```javascript
// Build a memoizer that caches function results but accounts for
// different execution contexts (this binding and arguments).

class ContextAwareMemoizer {
  constructor() {
    this.caches = new WeakMap(); // thisArg -> Map(args -> result)
    this.hits = 0;
    this.misses = 0;
  }

  memoize(fn) {
    const memoizer = this;
    return function(...args) {
      return memoizer._execute(fn, this, args);
    };
  }

  _execute(fn, thisArg, args) {
    // Get or create cache for this execution context
    if (!this.caches.has(thisArg)) {
      this.caches.set(thisArg, new Map());
    }
    const contextCache = this.caches.get(thisArg);

    // Create cache key from serialized arguments
    const key = this._serializeArgs(args);

    if (contextCache.has(key)) {
      this.hits++;
      return contextCache.get(key);
    }

    this.misses++;
    const result = fn.apply(thisArg, args);

    // Only cache primitive/immutable results to avoid stale references
    if (this._isCacheable(result)) {
      contextCache.set(key, result);
    }

    return result;
  }

  _serializeArgs(args) {
    try {
      return JSON.stringify(Array.from(args));
    } catch {
      return args.map(a => String(a)).join('::');
    }
  }

  _isCacheable(value) {
    return value === null ||
      value === undefined ||
      typeof value === 'string' ||
      typeof value === 'number' ||
      typeof value === 'boolean';
  }

  clearContext(thisArg) {
    this.caches.delete(thisArg);
  }

  clearAll() {
    this.caches = new WeakMap();
    this.hits = 0;
    this.misses = 0;
  }

  stats() {
    return {
      contexts: this.caches.size,
      hits: this.hits,
      misses: this.misses,
      hitRate: this.hits + this.misses > 0
        ? `${(this.hits / (this.hits + this.misses) * 100).toFixed(1)}%`
        : 'N/A'
    };
  }
}

// Usage:
// const memoizer = new ContextAwareMemoizer();
// const obj = { multiplier: 2 };
// const memoized = memoizer.memoize(function(n) { return n * this.multiplier; });
// console.log(memoizer.stats());
```

## Creation and Execution Phases

### What It Is

Each execution context goes through two distinct phases: the creation phase (where the environment is set up) and the execution phase (where code runs). The creation phase is where hoisting occurs—variables and functions are registered in memory before any code executes.

### Why It Is Important

The two-phase process explains why functions can be called before their declaration (hoisting), why `var` variables are `undefined` before assignment, and why `let`/`const` throw ReferenceError before initialization (TDZ). Understanding these phases is essential for debugging scope and timing issues.

### How It Works Internally

**Creation Phase**:
1. Create the variable object (VO): stores variables, function declarations, and function parameters
2. Create the scope chain: links the current VO to parent VOs
3. Determine the value of `this`
4. For function contexts: create the `arguments` object
5. Hoist function declarations (entire function body)
6. Hoist `var` declarations (initialize as `undefined`)
7. `let`/`const` declarations are hoisted but uninitialized (TDZ)

**Execution Phase**:
1. Execute code statements line by line
2. Assign values to variables
3. Evaluate expressions
4. Create new execution contexts for function calls
5. Build and return values

### Syntax

```javascript
// The creation phase explained
function demo() {
  console.log(a); // undefined (var hoisted)
  console.log(b); // ReferenceError (let in TDZ)
  console.log(fn); // Function (fully hoisted)
  
  var a = 1;
  let b = 2;
  
  function fn() { return 'hoisted'; }
  
  const c = 3;
  
  // What the creation phase looks like conceptually:
  // VO = {
  //   a: undefined,        // var hoisted
  //   fn: <function ref>,  // function fully hoisted
  //   b: <uninitialized>,  // let TDZ
  //   c: <uninitialized>   // const TDZ
  // }
}
```

### Beginner Examples

```javascript
// The two-phase process
// Creation phase for global context:
// 1. window/global object created
// 2. this = window
// 3. function declarations hoisted: foo
// 4. var declarations hoisted: bar = undefined

// Execution phase:
console.log(foo); // [Function: foo]
console.log(bar); // undefined (not ReferenceError!)

function foo() { return 'hello'; }
var bar = 'world';

console.log(bar); // 'world' (assigned during execution)

// Demonstrating the phases with a function
function test() {
  console.log('Phase 1: Creation');
  // At this point: x = undefined, y = uninitialized (TDZ), z = uninitialized (TDZ)
  
  var x = 10;
  let y = 20;
  const z = 30;
  
  console.log('Phase 2: Execution');
  // x = 10, y = 20, z = 30
}
```

### Intermediate Examples

```javascript
// TDZ in practice
function tdzDemo() {
  console.log(typeof x); // 'undefined' (var is hoisted and defined)
  console.log(typeof y); // ReferenceError! (let in TDZ, cannot access)
  
  var x = 1;
  let y = 2;
}

// TDZ with temporal dead zone and typeof
// typeof is safe for undeclared variables but NOT for TDZ
console.log(typeof undeclaredVar); // 'undefined' (safe)
console.log(typeof tdzVar); // ReferenceError!
let tdzVar = 10;

// Creation phase with nested functions
function outer() {
  function inner() {
    return 'inner';
  }
  var outerVar = 'outer';
  return inner(); // inner's creation phase happened when outer's creation phase ran
}

// Multiple var declarations with same name
var duplicate = 1;
function test() {
  console.log(duplicate); // undefined (local var hoisted, shadows global)
  var duplicate = 2;
}
test();
```

### Advanced Examples

```javascript
// Execution phase in detail with the call stack
function first() {
  console.log('first: starts');
  second();
  console.log('first: ends');
}

function second() {
  console.log('second: starts');
  third();
  console.log('second: ends');
}

function third() {
  console.log('third: runs');
}

first();
// Call stack during execution:
// 1. Global EC (creation) → Global EC (execution)
// 2. first() FEC (creation) → first() FEC (execution) → calls second
// 3. second() FEC (creation) → second() FEC (execution) → calls third
// 4. third() FEC (creation) → third() FEC (execution) → returns
// 5. second() continues → returns
// 6. first() continues → returns
// 7. Global EC continues

// Variable hoisting with block scopes (let/const)
function blockScoping() {
  var x = 1;
  
  if (true) {
    var x = 2; // Same variable (var is function-scoped)
    let y = 3; // Block-scoped
    console.log(x); // 2
    console.log(y); // 3
  }
  
  console.log(x); // 2 (var was overwritten)
  // console.log(y); // ReferenceError
}

// Creation phase optimization: not all variables are eagerly created
// Modern engines analyze code and only create needed variables

// The scope chain during creation phase
const globalVar = 'global';
function parent() {
  const parentVar = 'parent';
  function child() {
    const childVar = 'child';
    // Scope chain: child → parent → global
    console.log(globalVar); // Found in global scope
    console.log(parentVar); // Found in parent scope
    console.log(childVar);  // Found in own scope
  }
  child();
}
```

### Performance Considerations

- The creation phase runs once per execution context. For frequently-called functions, the overhead of setting up the variable environment, scope chain, and `this` binding can add measurable latency in hot paths
- Function declarations are fully hoisted (entire function body stored in memory) during creation, which uses more memory than variable hoisting but makes the function immediately available regardless of where it appears in the code
- `let`/`const` TDZ checks add runtime overhead: every time a variable in TDZ is accessed, the engine checks if initialization has occurred. This is why `let`/`const` can be slightly slower than `var` in some engines
- Block-scoped variables (for loops with `let`) create a new lexical environment for each iteration, adding allocation overhead. However, this enables closures in loops to capture the correct value
- Engines optimize by not eagerly creating variable bindings for unused variables detected during static analysis, reducing creation phase overhead

### Interview Questions

**Q: Explain the creation and execution phases of an execution context.**
A: The creation phase sets up the environment: creates the variable object, establishes the scope chain, determines `this`, hoists function declarations (full function body), hoists `var` declarations (initialized as `undefined`), and places `let`/`const` in the Temporal Dead Zone (uninitialized but hoisted). The execution phase then runs code line by line, assigning values, executing expressions, and creating new contexts for function calls.

**Q: What is the Temporal Dead Zone (TDZ) and when does it occur?**
A: The TDZ is the period between when a `let` or `const` variable enters scope (during creation phase) and when it is initialized (during execution phase). Accessing the variable during the TDZ throws a ReferenceError. The TDZ exists because these variables are hoisted (they exist in the scope) but not initialized (unlike `var` which is initialized as `undefined`).

**Q: How does hoisting work differently for function declarations vs function expressions?**
A: Function declarations are fully hoisted—the entire function body is stored in memory during the creation phase, so the function can be called before its declaration line. Function expressions (assigned to variables) follow the hoisting behavior of their assignment variable: `var fnExpr = function() {}` is hoisted as `undefined`, so calling it before the line throws TypeError; `let/const fnExpr = function() {}` is in the TDZ and throws ReferenceError.

**Q: What is the difference between the variable object and the activation object?**
A: In ES3 terminology, the Variable Object (VO) stores variables, function declarations, and function parameters. The Activation Object (AO) is the specific VO created for a function execution context (including the `arguments` object). In modern JavaScript (ES5+), these concepts are formalized as the VariableEnvironment and LexicalEnvironment, which handle both function-scoped (`var`) and block-scoped (`let`/`const`) declarations separately.

**Q: How does `eval` affect execution contexts?**
A: `eval()` creates an eval execution context that has access to the calling context's scope chain. In non-strict mode, `eval` can introduce new variables into the calling scope. In strict mode, `eval` has its own scope (like a nested function). `eval` is considered harmful because it prevents V8 from performing optimizations on the containing function—the engine cannot statically analyze code that may be introduced dynamically.

### Coding Challenges

**Challenge 1: Build a hoisting simulator that shows creation vs execution**
```javascript
// Create a simulator that traces the two-phase execution context process.
// Show what happens during creation vs execution phase step by step.

class HoistingSimulator {
  constructor(code) {
    this.code = code;
    this.creationPhase = [];
    this.executionPhase = [];
    this.scope = {};
    this.tdz = new Set();
  }

  simulate() {
    console.log('=== Hoisting Simulator ===\n');
    console.log('Code:', this.code, '\n');

    // Phase 1: Creation
    console.log('--- Creation Phase ---');
    this._parseDeclarations();
    this._printScope('After creation');

    // Phase 2: Execution
    console.log('\n--- Execution Phase ---');
    this._simulateExecution();
    this._printScope('After execution');

    return { creation: this.creationPhase, execution: this.executionPhase };
  }

  _parseDeclarations() {
    // Find function declarations
    const funcRegex = /function\s+(\w+)\s*\(/g;
    let match;
    while ((match = funcRegex.exec(this.code)) !== null) {
      this.scope[match[1]] = `<function ${match[1]}>`;
      this.creationPhase.push({
        type: 'function declaration',
        name: match[1],
        value: '<hoisted>'
      });
    }

    // Find var declarations
    const varRegex = /var\s+(\w+)/g;
    while ((match = varRegex.exec(this.code)) !== null) {
      this.scope[match[1]] = undefined;
      this.creationPhase.push({
        type: 'var hoisting',
        name: match[1],
        value: 'undefined'
      });
    }

    // Find let/const declarations (TDZ)
    const letConstRegex = /(let|const)\s+(\w+)/g;
    while ((match = letConstRegex.exec(this.code)) !== null) {
      this.tdz.add(match[2]);
      this.creationPhase.push({
        type: `${match[1]} declaration`,
        name: match[2],
        value: '<uninitialized> (TDZ)'
      });
    }
  }

  _simulateExecution() {
    const lines = this.code.split('\n');
    for (const line of lines) {
      const trimmed = line.trim();
      if (!trimmed) continue;

      // Handle assignments
      const assignMatch = trimmed.match(/(?:var|let|const)\s+(\w+)\s*=\s*(.+);?$/);
      if (assignMatch) {
        const name = assignMatch[1];
        const value = assignMatch[2];
        this.scope[name] = this._evaluate(value);
        this.tdz.delete(name);
        this.executionPhase.push({
          type: 'assignment',
          name,
          value: this.scope[name]
        });
        continue;
      }

      // Handle reassignments
      const reassignMatch = trimmed.match(/(\w+)\s*=\s*(.+);?$/);
      if (reassignMatch && reassignMatch[1] in this.scope) {
        const name = reassignMatch[1];
        this.scope[name] = this._evaluate(reassignMatch[2]);
        this.executionPhase.push({
          type: 'reassignment',
          name,
          value: this.scope[name]
        });
      }
    }
  }

  _evaluate(expr) {
    expr = expr.replace(/['"]/g, '').trim();
    if (!isNaN(expr)) return Number(expr);
    if (expr === 'true') return true;
    if (expr === 'false') return false;
    return expr;
  }

  _printScope(label) {
    console.log(`\n${label}:`);
    for (const [key, value] of Object.entries(this.scope)) {
      const tdzNote = this.tdz.has(key) ? ' (TDZ!)' : '';
      console.log(`  ${key} = ${value}${tdzNote}`);
    }
  }
}
```

**Challenge 2: Implement a TDZ-aware variable evaluator**
```javascript
// Create a variable wrapper that throws ReferenceError like the TDZ
// when accessed before initialization.

class TemporalDeadZone {
  constructor() {
    this.values = new Map();
    this.initialized = new WeakSet();
  }

  declare(name) {
    if (this.values.has(name)) {
      throw new SyntaxError(`Identifier '${name}' has already been declared`);
    }
    const symbol = Symbol(name);
    this.values.set(name, {
      symbol,
      initialized: false,
      value: undefined
    });
    return symbol;
  }

  initialize(name, value) {
    const binding = this.values.get(name);
    if (!binding) throw new ReferenceError(`Cannot access '${name}' before initialization`);
    binding.initialized = true;
    binding.value = value;
  }

  get(name) {
    const binding = this.values.get(name);
    if (!binding) throw new ReferenceError(`${name} is not defined`);
    if (!binding.initialized) {
      throw new ReferenceError(
        `Cannot access '${name}' before initialization (TDZ)`
      );
    }
    return binding.value;
  }

  set(name, value) {
    const binding = this.values.get(name);
    if (!binding) throw new ReferenceError(`${name} is not defined`);
    if (!binding.initialized) {
      throw new ReferenceError(
        `Cannot access '${name}' before initialization (TDZ)`
      );
    }
    binding.value = value;
  }

  has(name) {
    return this.values.has(name);
  }

  scope(fn) {
    // Create a mini-scope where variables have TDZ semantics
    return (...args) => {
      const saved = new Map(this.values);
      try {
        return fn(...args);
      } finally {
        this.values = saved;
      }
    };
  }
}

// Usage:
// const tdz = new TemporalDeadZone();
// tdz.declare('x');
// tdz.declare('y');
// // tdz.get('x'); // Would throw TDZ ReferenceError
// tdz.initialize('x', 10);
// console.log(tdz.get('x')); // 10
```

**Challenge 3: Create a visual scope chain debugger**
```javascript
// Build a tool that visually displays the scope chain at any point
// during code execution, showing variable values at each level.

class ScopeChainDebugger {
  constructor() {
    this.scopes = [];
    this.depth = 0;
  }

  enterScope(label, variables = {}) {
    this.depth++;
    const scope = {
      label,
      depth: this.depth,
      variables: { ...variables },
      timestamp: Date.now(),
      children: []
    };

    if (this.scopes.length > 0) {
      const parent = this.scopes[this.scopes.length - 1];
      parent.children.push(scope);
    }

    this.scopes.push(scope);
    return scope;
  }

  leaveScope() {
    const scope = this.scopes.pop();
    this.depth--;
    return scope;
  }

  updateVariable(name, value) {
    if (this.scopes.length > 0) {
      const current = this.scopes[this.scopes.length - 1];
      current.variables[name] = value;
    }
  }

  trace() {
    console.log('=== Scope Chain ===\n');
    this._printScope(this.scopes[0], 0);
  }

  _printScope(scope, indent) {
    if (!scope) return;
    const prefix = '  '.repeat(indent);
    console.log(`${prefix}┌─ ${scope.label} (depth: ${scope.depth})`);
    console.log(`${prefix}│  Variables:`);
    for (const [key, value] of Object.entries(scope.variables)) {
      const val = typeof value === 'function' ? '<function>' :
        typeof value === 'object' ? JSON.stringify(value) : String(value);
      console.log(`${prefix}│  • ${key} = ${val}`);
    }
    console.log(`${prefix}└──────────`);
    for (const child of scope.children) {
      this._printScope(child, indent + 1);
    }
  }

  findVariable(name) {
    // Walk scope chain from innermost to outermost
    for (let i = this.scopes.length - 1; i >= 0; i--) {
      const scope = this.scopes[i];
      if (name in scope.variables) {
        return {
          value: scope.variables[name],
          scope: scope.label,
          depth: scope.depth
        };
      }
    }
    return null;
  }

  clear() {
    this.scopes = [];
    this.depth = 0;
  }
}
```

## Scope Chain Formation

### What It Is

The scope chain is the hierarchical chain of variable environments that JavaScript uses for identifier resolution. When code accesses a variable, the engine traverses the scope chain from the current execution context outward to the global context, stopping at the first match. The scope chain is formed during the creation phase of each execution context.

### Why It Is Important

The scope chain is the mechanism that makes closures work, enables lexical scoping, and governs variable visibility. Understanding the scope chain helps developers predict which variables are accessible where, debug "undefined" variable errors, and design clean module boundaries.

### How It Works Internally

Each execution context has a reference to its outer environment (the parent scope). When a variable is accessed, the engine:
1. Looks in the current execution context's variable environment
2. If not found, follows the outer environment reference
3. Continues until the global context is reached
4. If still not found, throws `ReferenceError` (in strict mode) or returns `undefined`

The scope chain is defined by the lexical nesting of functions, not the call stack. This is called lexical scoping (or static scoping).

### Syntax

```javascript
// Scope chain: inner → middle → outer → global
const globalVar = 1;

function outer() {
  const outerVar = 2;
  
  function middle() {
    const middleVar = 3;
    
    function inner() {
      const innerVar = 4;
      // Can access: innerVar, middleVar, outerVar, globalVar
    }
    
    // Can access: middleVar, outerVar, globalVar
    // Cannot access: innerVar
  }
  
  // Can access: outerVar, globalVar
  // Cannot access: middleVar, innerVar
}
```

### Beginner Examples

```javascript
// Simple scope chain
let a = 'global';
function first() {
  let b = 'first';
  console.log(a); // 'global' (from outer scope)
  console.log(b); // 'first' (own scope)
  
  function second() {
    let c = 'second';
    console.log(a); // 'global' (scope chain traversal)
    console.log(b); // 'first' (scope chain)
    console.log(c); // 'second' (own scope)
  }
  
  second();
}

first();

// Shadowing
let x = 1;
function shadow() {
  let x = 2; // Shadows global x
  console.log(x); // 2
  function inner() {
    let x = 3; // Shadows the outer x
    console.log(x); // 3
  }
  inner();
  console.log(x); // 2
}
shadow();
console.log(x); // 1
```

### Intermediate Examples

```javascript
// Scope chain determines what variables are captured in closures
function createCounter(n) {
  let count = n;
  return {
    increment: function() { count++; },
    decrement: function() { count--; },
    getCount: function() { return count; }
  };
}
// All three returned functions share the SAME scope chain
// pointing to the SAME count variable

// Scope chain is lexical, not dynamic
const value = 'global';
function wrapper() {
  const value = 'wrapper';
  function test() {
    console.log(value);
  }
  return test;
}

wrapper()(); // 'wrapper' (lexical scope of test is wrapper, not call site)

// Dynamic scope vs lexical scope
function outer() {
  const x = 'outer';
  function inner() {
    console.log(x);
  }
  middle(inner);
}

function middle(fn) {
  const x = 'middle';
  fn(); // 'outer' (lexical scope wins, not call-site scope)
}
```

### Advanced Examples

```javascript
// Scope chain with block scopes (let/const create block scopes)
function blockScopeChain() {
  if (true) {
    let blockVar = 'block';
    var functionVar = 'function';
    const blockConst = 'also block';
    
    function insideBlock() {
      console.log(blockVar);   // 'block' (scope chain)
      console.log(functionVar); // 'function' (function scope)
      console.log(blockConst);  // 'also block'
    }
    insideBlock();
  }
  console.log(functionVar); // 'function' (var is function-scoped)
  // console.log(blockVar); // ReferenceError
}

// Scope chain with catch blocks
try {
  throw new Error('test');
} catch (err) {
  // err is only accessible in catch block
  console.log(err.message);
}
// console.log(err); // ReferenceError

// With statement (considered harmful, disables optimizations)
const obj = { a: 1, b: 2 };
with (obj) {
  console.log(a); // 1 (obj properties become variables in scope)
}

// Eval creates its own scope chain
function evalScope() {
  const x = 'function scope';
  eval('var y = "eval scope"; console.log(x);'); // 'function scope'
  console.log(y); // 'eval scope' (var in eval leaks to enclosing function)
}

// Strict mode eval
function strictEval() {
  'use strict';
  const x = 'function scope';
  eval('var y = "eval scope";');
  console.log(y); // ReferenceError (eval has its own scope in strict mode)
}
```

### Real-World Use Cases

- Module pattern (creating private state via scope chain)
- Framework internals (Angular's zone.js, React's fiber)
- Error handling (try/catch scope chains)
- Testing (mocking via scope manipulation)
- Secure sandboxing (controlling scope chain)

### Common Mistakes

- Confusing scope chain with call stack (lexical vs dynamic)
- Assuming `this` follows the scope chain (it does not; `this` is a binding, not a variable)
- Creating functions inside loops and expecting each to capture a different variable (use `let`)
- Thinking `var` creates block scope (it doesn't)
- Not understanding that inner functions capture references, not values

### Best Practices

- Use `const` by default, `let` when needed, never `var`
- Keep functions small to limit scope chain depth
- Prefer passing values explicitly over relying on scope chain for "implicit" parameters
- Avoid deep nesting (pyramid of doom)
- Use modules for explicit dependency declaration
- Avoid `with` and `eval` (performance and security issues)

### Performance Considerations

- Each level of scope chain traversal adds lookup overhead. Deeply nested scopes (5+ levels) can slow down variable access by 2-3x compared to local variables
- Closure scope chains persist in memory as long as any function in the chain remains referenced. Each closure retains the entire chain, not just the variables it uses
- The with statement and try/catch block create dynamic scopes that cannot be optimized by V8. Functions containing `with` are never optimized by TurboFan
- Block-scoped variables (`let`/`const` in blocks) add additional scope objects to the chain. In practice, the overhead is negligible for normal use
- Shadowing (declaring a variable with the same name in an inner scope) forces the engine to stop scope chain traversal earlier, which can be a slight optimization

### Interview Questions

**Q: What is the difference between the scope chain and the call stack?**
A: The scope chain is based on lexical nesting (where functions are defined in the source code) and determines variable access. The call stack is based on the order of function calls (at runtime). The scope chain is static (determined at parse time), while the call stack is dynamic (changes during execution). A function's scope chain never changes, but it can appear at different positions on the call stack.

**Q: How does the scope chain enable closures?**
A: When a function is defined inside another function, the inner function captures a reference to the outer function's scope chain. Even after the outer function has returned and its execution context is popped from the stack, the inner function's scope chain keeps the outer function's variable environment alive. This preserved scope chain is what allows the inner function to access the outer function's variables later.

**Q: What is variable shadowing and when would you use it?**
A: Variable shadowing occurs when an inner scope declares a variable with the same name as one in an outer scope. The inner declaration "shadows" the outer one—code in the inner scope can only access the inner variable. This is occasionally useful for intentional name reuse in a limited scope, but it can also cause bugs if the developer forgets about the outer variable.

**Q: How does `let` in a for loop create separate scope for each iteration?**
A: In a `for (let i = 0; i < n; i++)` loop, the engine creates a new lexical environment for each iteration. Each environment has its own binding of `i` with the current value. This is why closures created inside the loop capture the correct value of `i`. With `var`, there's only one binding shared across all iterations, so all closures see the final value.

**Q: What is the "with" statement and why is it deprecated?**
A: `with (obj) { ... }` extends the scope chain by adding the properties of `obj` as variables. It's deprecated because: (1) it makes code ambiguous (you can't tell if `x` refers to a property or a variable), (2) it prevents V8 from optimizing the containing function (the engine can't statically determine the scope), (3) it's slower than explicit property access, and (4) it can cause security issues in sandbox environments.

### Coding Challenges

**Challenge 1: Implement a lexical scope analyzer**
```javascript
// Build a tool that analyzes code and reports the scope chain
// for each identifier reference.

class LexicalScopeAnalyzer {
  constructor(code) {
    this.code = code;
    this.scopes = [];
    this.references = [];
  }

  analyze() {
    const lines = this.code.split('\n');
    this.scopes = [{ name: 'global', variables: new Set(), children: [], parent: null }];

    lines.forEach((line, lineNum) => {
      const trimmed = line.trim();

      // Detect function declarations (new scope)
      const funcMatch = trimmed.match(/function\s+(\w+)/);
      if (funcMatch) {
        const scope = {
          name: funcMatch[1],
          variables: new Set(),
          children: [],
          parent: this.scopes[this.scopes.length - 1]
        };
        this.scopes[this.scopes.length - 1].children.push(scope);
        this.scopes.push(scope);
      }

      // Detect variables declared in this scope
      const varMatches = trimmed.matchAll(/(?:let|const|var)\s+(\w+)/g);
      for (const m of varMatches) {
        this.scopes[this.scopes.length - 1].variables.add(m[1]);
      }

      // Detect identifier references
      const refMatches = trimmed.matchAll(/\b([a-zA-Z_$]\w*)\b/g);
      for (const m of refMatches) {
        if (!['function', 'let', 'const', 'var', 'return', 'if', 'else',
              'for', 'while', 'true', 'false', 'null', 'undefined',
              'this', 'new', 'typeof', 'instanceof', 'delete', 'void',
              'throw', 'try', 'catch', 'finally'].includes(m[1])) {
          this.references.push({
            name: m[1],
            line: lineNum + 1,
            column: m.index
          });
        }
      }

      // Detect end of function
      if (trimmed === '}') {
        this.scopes.pop();
      }
    });

    return {
      scopes: this.scopes[0], // Root scope tree
      references: this.resolveReferences()
    };
  }

  resolveReferences() {
    return this.references.map(ref => {
      const resolved = this.findBinding(ref.name);
      return {
        ...ref,
        resolved: resolved || null,
        kind: resolved ? resolved.kind : 'global'
      };
    });
  }

  findBinding(name) {
    // Walk from innermost scope outward
    for (let i = this.scopes.length - 1; i >= 0; i--) {
      const scope = this.scopes[i];
      if (scope.variables.has(name)) {
        return { scope: scope.name, kind: 'local' };
      }
    }
    return null;
  }

  printScopeTree() {
    const print = (scope, indent = 0) => {
      const prefix = '  '.repeat(indent);
      console.log(`${prefix}└─ ${scope.name} (${scope.variables.size} vars)`);
      scope.variables.forEach(v => console.log(`${prefix}    • ${v}`));
      scope.children.forEach(child => print(child, indent + 1));
    };
    print(this.scopes[0]);
  }
}
```

**Challenge 2: Build a closure-based state management system**
```javascript
// Create a simple state management library using closures and scope chains.
// Each store has private state accessible only through defined actions.

function createStore(initialState, reducer) {
  // Private state - only accessible via scope chain
  let state = { ...initialState };
  const listeners = new Set();

  function getState() {
    return state;
  }

  function dispatch(action) {
    const prevState = state;
    state = reducer(state, action);

    if (state !== prevState) {
      listeners.forEach(listener => listener(state, prevState));
    }
  }

  function subscribe(listener) {
    listeners.add(listener);
    return () => listeners.delete(listener);
  }

  function createSelector(selectorFn) {
    let lastResult;
    let lastState;

    return function() {
      if (state !== lastState) {
        lastResult = selectorFn(state);
        lastState = state;
      }
      return lastResult;
    };
  }

  function createMiddleware(middleware) {
    const originalDispatch = dispatch;
    dispatch = (action) => {
      middleware({
        getState,
        dispatch: originalDispatch
      })(originalDispatch)(action);
    };
  }

  return { getState, dispatch, subscribe, createSelector, createMiddleware };
}

// Usage:
// const store = createStore({ count: 0, todos: [] }, (state, action) => {
//   switch (action.type) {
//     case 'INCREMENT':
//       return { ...state, count: state.count + 1 };
//     default:
//       return state;
//   }
// });
// const unsubscribe = store.subscribe((state) => console.log('State:', state));
// store.dispatch({ type: 'INCREMENT' });
```

**Challenge 3: Build a scope chain visualizer using Proxy**
```javascript
// Use Proxy to intercept variable access and log the scope chain traversal.

class ScopeVisualizer {
  constructor(depth = 3) {
    this.depth = depth;
    this.scopes = [];
    this.hits = new Map();
  }

  createScope(name, parentScope = null) {
    const scope = {
      name,
      variables: {},
      parent: parentScope,
      children: []
    };

    const handler = {
      get: (target, prop) => {
        if (typeof prop === 'string' && !prop.startsWith('_')) {
          this.trackAccess(name, prop, 'get');
        }
        return target[prop];
      },
      set: (target, prop, value) => {
        if (typeof prop === 'string' && !prop.startsWith('_')) {
          this.trackAccess(name, prop, 'set');
        }
        target[prop] = value;
        return true;
      }
    };

    const proxy = new Proxy(scope.variables, handler);
    scope.proxy = proxy;

    if (parentScope) {
      parentScope.children.push(scope);
    }
    this.scopes.push(scope);

    // Create the scope chain lookup
    const scopeChain = this.buildScopeChain(scope);
    const chainHandler = {
      get: (target, prop) => {
        if (prop === '_addVariable') return (name, value) => { proxy[name] = value; };
        for (const s of scopeChain) {
          if (prop in s.variables) {
            this.trackAccess(s.name, prop, 'get-chain');
            return s.variables[prop];
          }
        }
        return undefined;
      },
      set: (target, prop, value) => {
        // Set on nearest scope that has the variable, or current scope
        for (const s of scopeChain) {
          if (prop in s.variables) {
            s.variables[prop] = value;
            this.trackAccess(s.name, prop, 'set-chain');
            return true;
          }
        }
        proxy[prop] = value;
        return true;
      }
    };

    return new Proxy({}, chainHandler);
  }

  buildScopeChain(scope) {
    const chain = [];
    let current = scope;
    for (let i = 0; i < this.depth && current; i++) {
      chain.push(current);
      current = current.parent;
    }
    return chain;
  }

  trackAccess(scopeName, varName, type) {
    const key = `${scopeName}:${varName}`;
    if (!this.hits.has(key)) {
      this.hits.set(key, { scope: scopeName, variable: varName, reads: 0, writes: 0 });
    }
    const record = this.hits.get(key);
    if (type === 'get' || type === 'get-chain') record.reads++;
    else record.writes++;
  }

  printAccessReport() {
    console.log('=== Scope Chain Access Report ===\n');
    const sorted = Array.from(this.hits.values())
      .sort((a, b) => b.reads + b.writes - a.reads - a.writes);

    console.log('Variable'.padEnd(25), 'Scope'.padEnd(15), 'Reads'.padEnd(8), 'Writes');
    console.log('-'.repeat(60));
    sorted.forEach(r => {
      console.log(
        r.variable.padEnd(25),
        r.scope.padEnd(15),
        String(r.reads).padEnd(8),
        r.writes
      );
    });
  }
}
```

### Related Topics

- Call stack and execution context order
- Lexical environment and variable environment
- `this` binding (distinct from scope)
- Hoisting in creation phase
- Closure formation (function + scope chain)
- Module scope (different from function scope)
- Block scoping with let/const
- Global object and globalThis
- Temporal Dead Zone
