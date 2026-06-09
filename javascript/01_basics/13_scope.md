# Scope - Global scope, function scope, block scope, lexical scope

## Introduction
Scope determines the accessibility and visibility of variables in your code. JavaScript has several types of scope: global, function, block, and lexical scope. Understanding scope is critical for writing correct, predictable code and avoiding variable collision.

## Global scope

### What It Is
Global scope is the outermost scope. Variables declared in global scope are accessible from anywhere in the program, including inside functions, blocks, and modules.

### Why It Is Important
Global variables are shared across the entire application, making them useful for application-wide constants and configuration. However, excessive global variables cause naming conflicts, make debugging difficult, and are considered bad practice.

### How It Works Internally
In browsers, the global scope is the `window` object. In Node.js, it's `global`. Variables declared with `var` at the top level become properties of the global object. `let` and `const` at the top level create variables in the global lexical environment but don't create global object properties.

### Syntax
```javascript
// Global variable declarations
var globalVar = "accessible everywhere"; // becomes window.globalVar
let globalLet = "also global but not on window";
const globalConst = "constant global";

// Global function declaration
function globalFunction() {
  return "I'm globally accessible";
}
```

### Beginner Examples
```javascript
// Global scope in browser
var message = "Hello";
function greet() {
  console.log(message); // Accesses global variable
}
greet(); // "Hello"

// Global variable implicitly (bad practice)
function badFunction() {
  leak = "I become global"; // No var/let/const!
}
badFunction();
console.log(leak); // "I become global" (accessible globally)

// Accessing global object
console.log(window);      // Browser global object
console.log(global);      // Node.js global object
console.log(globalThis);  // Universal (ES2020)

// Shadowing (do not do)
let value = "global";
function test() {
  let value = "local";
  console.log(value); // "local"
}
test();
console.log(value); // "global"
```

### Intermediate Examples
```javascript
// Global scope pollution
var count = 0;
var count = 1; // Re-declaration (bad with var)
console.log(count); // 1

// let at global scope
let name = "Alice";
// let name = "Bob"; // SyntaxError: can't re-declare

// Global scope in modules
// In ES modules, top-level declarations are scoped to the module, not global

// Global constants
const CONFIG = {
  API_URL: "https://api.example.com",
  TIMEOUT: 5000
};
// CONFIG is global but not on window

// Global function expression
const globalHandler = function() {
  console.log("This is assigned to a global variable");
};
```

### Advanced Examples
```javascript
// Global scope in strict mode
"use strict";
leakedVar = "error in strict mode"; // ReferenceError

// globalThis (ES2020) for universal access
function getGlobalThis() {
  if (typeof globalThis !== "undefined") return globalThis;
  if (typeof self !== "undefined") return self;
  if (typeof window !== "undefined") return window;
  if (typeof global !== "undefined") return global;
  return Function("return this")();
}

// Global variable in different environments
// Browser: var x = 1; → window.x
// Node.js REPL: var x = 1; → global.x
// Node.js module: var x = 1; → module-scoped, not global

// Avoiding globals with IIFE
(function() {
  const privateVar = "I'm not global";
  console.log(privateVar);
})();
// console.log(privateVar); // ReferenceError

// Namespace pattern (before modules)
const MyApp = {};
MyApp.utils = {
  formatDate: function(date) { /* ... */ },
  parseNumber: function(str) { /* ... */ }
};
```

### Real-World Use Cases
- Application-wide configuration constants
- Polyfills that augment built-in prototypes
- Global error handlers (`window.onerror`)
- Feature detection flags
- Third-party script interoperability (carefully)
- Analytics and tracking globals

### Common Mistakes
- Polluting global scope with too many variables
- Accidentally creating global variables by omitting declaration keywords
- Relying on implicit global variables in functions
- Naming conflicts between libraries (solution: modules)
- Assuming module-level declarations are truly global

### Best Practices
- Minimize global variables; prefer modules and local scope
- Use `const` for global constants
- Use `let` sparingly at global scope, never `var`
- Use modules to encapsulate code
- Use IIFE for immediate execution without pollution
- Use namespace objects (e.g., `const APP = {}`) if globals are necessary
- Enable strict mode to catch accidental globals

### Performance Considerations
- Global variables are always in scope (slowest lookup)
- Engines optimize global variable access less than local
- Module-scoped variables are faster than true globals
- Each global increases the likelihood of name collisions

### Interview Questions
1. What is the global object in browsers vs Node.js?
2. How do you create a global variable without using `var`/`let`/`const`?
3. What is `globalThis`?
4. Why should you avoid global variables?
5. How are ES module top-level variables different from script-level globals?

### Coding Challenges
1. Write a function that detects the global object in any environment.
2. Convert a global-polluted script to use modular scope.
3. Create a namespace object that reduces global footprint.
4. Implement a function that counts how many global properties exist.

### Related Topics
- Function scope
- Block scope
- Global object
- Module scope

## Function scope

### What It Is
Function scope means variables declared inside a function are only accessible within that function. Each function creates its own scope. This is the original scoping mechanism in JavaScript (for `var`).

### Why It Is Important
Function scope enables encapsulation and information hiding. Variables inside functions don't interfere with outer scopes, enabling modular code organization.

### How It Works Internally
When a function is called, the engine creates a new execution context with a new lexical environment. Variables declared with `var` are hoisted to the top of this function. Parameters are also part of the function's scope.

### Syntax
```javascript
function outer() {
  var functionScoped = "only inside outer";
  function inner() {
    var innerScoped = "only inside inner";
    console.log(functionScoped); // Accessible (closure)
  }
  // console.log(innerScoped); // ReferenceError
  inner();
}
outer();
```

### Beginner Examples
```javascript
// Function scope basics
function demo() {
  var localVar = "I'm local to demo()";
  console.log(localVar); // Works
}
demo();
// console.log(localVar); // ReferenceError

// var ignores blocks
if (true) {
  var blockVar = "I'm function-scoped, not block-scoped";
}
console.log(blockVar); // "I'm function-scoped..."

// Function scope with let
function test() {
  let x = 10;
  if (true) {
    let x = 20; // Different scope (block)
    console.log(x); // 20
  }
  console.log(x); // 10
}

// Nested function scope
function outer(a) {
  function inner(b) {
    return a + b;
  }
  return inner;
}
const add5 = outer(5);
console.log(add5(3)); // 8
```

### Intermediate Examples
```javascript
// Hoisting within function scope
function hoistingDemo() {
  console.log(x); // undefined (hoisted, not initialized)
  var x = 10;
  console.log(x); // 10
}

// Function parameters are scoped to the function
function sum(...args) {
  // rest parameter creates a real array in function scope
  return args.reduce((a, b) => a + b, 0);
}

// Function declaration in function scope
function parent() {
  function child() {
    return "child";
  }
  return child(); // Works
}
// child(); // ReferenceError: not in scope

// var in for loop (function scope issue)
function loopClosure() {
  var funcs = [];
  for (var i = 0; i < 3; i++) {
    funcs.push(function() { return i; });
  }
  return funcs.map(f => f()); // [3, 3, 3] (all share same i)
}
```

### Advanced Examples
```javascript
// Function scope and closures
function createCounter() {
  let count = 0; // count is in createCounter's scope
  return {
    increment() { count++; },
    decrement() { count--; },
    getValue() { return count; }
  };
}
const counter = createCounter();
counter.increment();
counter.increment();
console.log(counter.getValue()); // 2
// console.log(count); // ReferenceError

// IIFE creates function scope
const singleton = (function() {
  let instance;
  return {
    getInstance() {
      if (!instance) instance = { id: Math.random() };
      return instance;
    }
  };
})();

// Function scope in recursion
function factorial(n) {
  // Each recursive call has its own n parameter
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}

// Function scope with try/catch
function safeParse(json) {
  try {
    const parsed = JSON.parse(json);
    return parsed;
  } catch (e) {
    // e is scoped to catch block
    return null;
  }
  // console.log(parsed); // ReferenceError
}
```

### Real-World Use Cases
- Encapsulating private state (closure pattern)
- Module pattern with IIFE
- Function-level constants and helpers
- Preventing variable leakage to global scope
- Creating domain-specific functions within a scope
- Temporary variables that should not persist

### Common Mistakes
- Assuming `var` is block-scoped (it's function-scoped)
- Creating multiple functions in a loop with `var` (closure bug)
- Forgetting that function parameters are scoped to the function
- Declaring functions inside loops (performance)
- Not understanding nested function scope creates closures

### Best Practices
- Use `let` and `const` for block scoping within functions
- Keep functions small (single responsibility)
- Use closure patterns for private state
- Avoid deeply nested functions (extract to named functions)
- Prefer `const` by default within function scope
- Use IIFE for immediate execution without pollution

### Performance Considerations
- Function scope creation has minor overhead
- Each function call creates a new lexical environment
- Closure variables persist in memory until references are removed
- Modern engines optimize simple functions aggressively

### Interview Questions
1. What is the difference between function scope and block scope?
2. How does `var` behave inside a function?
3. What is a closure and how does it relate to function scope?
4. How do IIFEs use function scope?
5. What happens to function scope in recursion?

### Coding Challenges
1. Create a module using function scope (IIFE pattern).
2. Fix the classic loop closure bug by understanding function scope.
3. Implement a function that generates unique IDs using closure.
4. Write a function that demonstrates nested function scope.

### Related Topics
- Block scope
- Closures
- Hoisting
- IIFE

## Block scope (let/const)

### What It Is
Block scope limits variable access to the block in which they are declared. A block is defined by curly braces `{}`. `let` and `const` are block-scoped; `var` is not.

### Why It Is Important
Block scope provides finer-grained control over variable lifetime than function scope. It prevents accidental variable sharing between blocks (especially in loops and conditionals) and aligns with other C-family languages.

### How It Works Internally
When the engine encounters a block (`if`, `for`, `while`, bare `{}`), it creates a new lexical environment for `let` and `const` declarations within that block. Variables are hoisted to the top of the block but remain in the Temporal Dead Zone until declaration.

### Syntax
```javascript
{
  let blockScoped = "only in this block";
  const alsoBlockScoped = "also only in this block";
}
// console.log(blockScoped); // ReferenceError

if (true) {
  let message = "block-scoped to if";
}
for (let i = 0; i < 3; i++) {
  // i is block-scoped to each iteration
}
```

### Beginner Examples
```javascript
// Block scope with if
if (true) {
  let x = 10;
  const y = 20;
  console.log(x, y); // 10, 20
}
// console.log(x); // ReferenceError
// console.log(y); // ReferenceError

// Block scope with for
for (let i = 0; i < 3; i++) {
  console.log(i); // 0, 1, 2
}
// console.log(i); // ReferenceError

// Bare block
{
  let private = "secret";
  var notBlockScoped = "visible outside";
}
console.log(notBlockScoped); // "visible outside"
// console.log(private); // ReferenceError

// Block scope in switch
switch (x) {
  case 0: {
    let result = "zero";
    break;
  }
  case 1: {
    let result = "one"; // OK, different block
    break;
  }
}
```

### Intermediate Examples
```javascript
// Block scope in for loop (correct closure behavior)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0); // 0, 1, 2
}

// Block scope with const
{
  const API_KEY = "abc123";
  // Use API_KEY within this block
  fetchData(API_KEY);
}
// API_KEY not accessible here

// Block scope prevents temporal issues
{
  // console.log(temp); // ReferenceError (TDZ)
  const temp = "initialized";
  console.log(temp); // "initialized"
}

// Nested blocks
{
  let a = 1;
  {
    let a = 2; // Different variable (shadowing)
    console.log(a); // 2
  }
  console.log(a); // 1
}
```

### Advanced Examples
```javascript
// Block scope in try/catch
try {
  throw new Error("fail");
} catch (error) {
  const message = error.message;
  console.log(message); // "fail"
}
// console.log(message); // ReferenceError
// console.log(error); // ReferenceError

// Block scope with labels
blockLabel: {
  const data = fetchData();
  if (!data.valid) break blockLabel;
  processData(data);
  console.log("Data processed");
}

// Block scope for resource management
{
  const connection = createConnection();
  try {
    connection.query("SELECT * FROM users");
  } finally {
    connection.close();
  }
}
// connection is not accessible here

// Block scope in arrow functions
const fn = () => {
  let x = 1;
  {
    let x = 2; // OK, block-scoped
    return x;
  }
};

// IIFE vs block (modern)
// Old IIFE pattern:
(function() {
  const temp = "value";
  console.log(temp);
})();

// Modern block pattern:
{
  const temp = "value";
  console.log(temp);
}
```

### Real-World Use Cases
- Loop counters isolated to the loop
- Temporary variables inside conditionals
- Inline blocks for resource management
- Preventing variable name collisions
- Isolating `const` declarations in switch cases
- Scoping temporary variables in complex algorithms
- Replacing IIFE patterns for simple scope isolation

### Common Mistakes
- Assuming `var` has block scope
- Forgetting that bare blocks are valid and create scope
- Re-declaring `let`/`const` in the same block (SyntaxError)
- Trying to access `let`/`const` outside their block
- Not wrapping switch cases in blocks for unique declarations

### Best Practices
- Use `let` and `const` exclusively (never `var` in new code)
- Declare variables as close to their use as possible
- Use blocks to limit variable scope in long functions
- Use block scope in switch cases for unique variable declarations
- Prefer `const` by default; use `let` only for reassignment
- Avoid using blocks for side effects (use functions instead)

### Performance Considerations
- Block scope creation has negligible overhead
- Engines optimize block-scoped variables efficiently
- Each `let`/`const` declaration in a block creates a binding
- Loop iteration with `let` creates a new binding per iteration (minor cost)
- Modern JIT handles block scope well

### Interview Questions
1. What is block scope and which keywords support it?
2. How does block scope fix the loop closure problem?
3. What is the difference between block scope and function scope?
4. Can you create a block scope without a control structure?
5. How does `let`/`const` in blocks interact with hoisting?

### Coding Challenges
1. Fix a `var` loop closure bug by switching to `let`.
2. Create a block-scoped variable that shadows an outer variable.
3. Implement a switch statement where each case has its own block scope.
4. Write a function that demonstrates TDZ within a block.

### Related Topics
- Function scope
- let and const
- Temporal Dead Zone
- Hoisting

## Lexical scope

### What It Is
Lexical scope (also called static scope) means that the scope of a variable is determined by its position in the source code. Inner functions can access variables from outer scopes where they are defined.

### Why It Is Important
Lexical scope is the foundation of closures, one of JavaScript's most powerful features. It enables callbacks, event handlers, and functional programming patterns.

### How It Works Internally
Each execution context has a reference to its outer lexical environment. When a variable is not found in the current scope, the engine walks up the scope chain (outer environment references) until it finds the variable or reaches the global scope.

### Syntax
```javascript
const outer = "global";

function outerFunction() {
  const middle = "outer function";
  
  function innerFunction() {
    const inner = "inner function";
    console.log(inner);  // From own scope
    console.log(middle); // From outerFunction scope (lexical)
    console.log(outer);  // From global scope (lexical)
  }
  
  innerFunction();
}
outerFunction();
```

### Beginner Examples
```javascript
// Basic lexical scoping
const name = "Global";
function sayName() {
  console.log(name); // "Global" (from outer scope)
}
sayName();

// Nested lexical scope
function outer() {
  const message = "Hello from outer";
  function inner() {
    console.log(message); // "Hello from outer" (lexical access)
  }
  inner();
}
outer();

// Scope chain
const a = 1;
function first() {
  const b = 2;
  function second() {
    const c = 3;
    console.log(a, b, c); // 1, 2, 3 (walks scope chain)
  }
  second();
}
first();
```

### Intermediate Examples
```javascript
// Lexical scope in callbacks
function createCallback() {
  const name = "Alice";
  return function() {
    console.log(name); // Captures `name` from lexical scope
  };
}
const callback = createCallback();
callback(); // "Alice" (closure)

// Lexical scope with multiple levels
function configure(prefix) {
  return function(middle) {
    return function(suffix) {
      return `${prefix}-${middle}-${suffix}`;
    };
  };
}
const withPrefix = configure("PRE");
const withMiddle = withPrefix("MID");
console.log(withMiddle("SUF")); // "PRE-MID-SUF"

// Lexical scope preserved in async callbacks
function delayedGreet(name) {
  setTimeout(() => {
    console.log(`Hello, ${name}!`); // `name` from lexical scope
  }, 1000);
}
delayedGreet("Alice"); // "Hello, Alice!" (after 1s)

// Lexical scope with let
function createCounters() {
  const counters = [];
  for (let i = 0; i < 3; i++) {
    counters.push(() => i); // Each gets its own `i`
  }
  return counters;
}
const [c0, c1, c2] = createCounters();
console.log(c0(), c1(), c2()); // 0, 1, 2
```

### Advanced Examples
```javascript
// Lexical scope in modules
// module.js
const privateVar = "module-private";
export function getPrivate() {
  return privateVar; // Lexical access to module scope
}

// Lexical scope in class methods
class Container {
  constructor(secret) {
    this.secret = secret;
  }
  getSecret() {
    return this.secret; // `this` is dynamic, not lexical
  }
  getArrowSecret = () => {
    return this.secret; // Arrow captures `this` lexically
  }
}

// Lexical scope chain depth
function scopeDepth() {
  const level1 = 1;
  function level2() {
    const level2Var = 2;
    function level3() {
      const level3Var = 3;
      function level4() {
        const level4Var = 4;
        return level1 + level2Var + level3Var + level4Var;
      }
      return level4();
    }
    return level3();
  }
  return level2();
}
console.log(scopeDepth()); // 10

// Lexical scope with eval (avoid eval)
function evalScope() {
  const x = 10;
  eval("console.log(x)"); // 10 (eval uses lexical scope)
}
```

### Real-World Use Cases
- Event handlers accessing component state
- Callback functions in async operations
- Functional programming (currying, composition)
- Creating custom hooks in React
- Middleware patterns (Express, Redux)
- Memoization with private cache
- Factory functions with shared state
- Partial application and currying

### Common Mistakes
- Confusing lexical scope with dynamic scope (`this`)
- Expecting `eval()` to have access to local scope (it does, but avoid it)
- Not realizing that closures capture variables, not values
- Creating closures in loops without understanding binding
- Assuming `this` is lexically scoped (use arrow functions for lexical `this`)

### Best Practices
- Understand that functions capture their lexical environment
- Use closures for private state (module pattern)
- Be mindful of memory usage with long-lived closures
- Prefer explicit parameter passing over closure capture when clear
- Use arrow functions when lexical `this` is needed
- Use block-scoped variables to limit captured scope
- Consider using `WeakMap` for large data in closures

### Performance Considerations
- Lexical scope chain lookup is fast (usually O(1) after optimization)
- Each closure retains reference to its outer scope (memory)
- Long scope chains are not a performance concern
- The outer variables in a closure prevent GC of the scope
- Modern engines optimize scope chain lookups aggressively

### Interview Questions
1. What is lexical scope?
2. How does the scope chain work?
3. What is the difference between lexical scope and dynamic scope?
4. How do closures relate to lexical scope?
5. What happens when a closure references a variable from an outer scope?

### Coding Challenges
1. Implement a function that demonstrates the scope chain across 4 levels.
2. Create a curried function that uses lexical scope at each level.
3. Write a function that returns an object with methods that access lexical scope.
4. Implement a memoization function using lexical scope for the cache.
5. Create a simple dependency injection using lexical scope.

### Related Topics
- Closures
- Scope chain
- Lexical environment
- Execution context
