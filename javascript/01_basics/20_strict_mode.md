# Strict Mode - "use strict", restrictions, benefits, module-level strict mode

## Introduction
Strict mode is a way to opt into a restricted variant of JavaScript. It eliminates some silent errors, fixes mistakes that make optimization difficult, and prohibits syntax that may be defined in future versions. Strict mode is enabled by the `"use strict"` directive or automatically in modules and classes.

## "use strict" directive

### What It Is
The `"use strict"` directive is a string literal expression that enables strict mode for the entire script or function scope. It must be the first statement in the scope to take effect.

### Why It Is Important
Strict mode catches common coding errors, prevents unsafe actions, and enables JavaScript engines to optimize code more effectively. It is considered best practice for all modern JavaScript code.

### How It Works Internally
When the parser encounters `"use strict"`, it sets a flag that changes how the engine evaluates code. This affects variable resolution, `this` binding, property access, function behavior, and more. In strict mode, the engine throws errors for actions that were silently ignored in sloppy mode.

### Syntax
```javascript
// Script-level strict mode
"use strict";
// All code in this file runs in strict mode

// Function-level strict mode
function strictFunction() {
  "use strict";
  // Only code in this function is strict
}

// Module strict mode (automatic)
// ES modules are always in strict mode
```

### Beginner Examples
```javascript
// Script level
"use strict";

var x = 10;
let y = 20;
const z = 30;

function test() {
  // Inherits strict mode from script
  console.log("Strict mode function");
}

// Function level
function nonStrictFunction() {
  // Not strict
  var a = 1;
}

function strictFunction2() {
  "use strict";
  // Strict mode only in this function
  var b = 2;
}

// Order matters - "use strict" must be first statement
// This does NOT enable strict mode:
var a = 1;
"use strict"; // Ignored (not first statement)
```

### Intermediate Examples
```javascript
// Strict mode in IIFE
(function() {
  "use strict";
  // Module-like strict scope
  var privateVar = "secret";
})();

// Multiple directives (only first counts)
"use strict";
"use strict"; // Redundant, but harmless

// Strict mode with classes (classes are always strict)
class Example {
  constructor() {
    // Strict mode is automatic
  }
}

// Strict mode with eval
"use strict";
eval("var x = 10");
console.log(x); // ReferenceError (eval vars not leaked in strict mode)

// Checking if strict mode is active
function isStrictMode() {
  return !this; // In strict mode, `this` is undefined in non-method functions
}
console.log(isStrictMode()); // true if script is strict

// Alternative check
const isStrict = (function() { return !this; })();
console.log(isStrict);
```

### Advanced Examples
```javascript
// Strict mode and module interaction
// module.js (always strict)
export function strictFn() {
  // Already strict due to module context
  return "strict";
}

// Conditional strict mode (polyfill patterns)
(function() {
  "use strict";
  
  // Your library code here
  // Always strict regardless of host
  
  if (typeof window !== "undefined") {
    window.MyLib = { /* ... */ };
  }
})();

// Strict mode in dynamically evaluated code
const strictEval = new Function('"use strict"; return !this;');
console.log(strictEval()); // true

// Non-strict eval
const nonStrictEval = new Function('return !this;');
console.log(nonStrictEval()); // false (in non-strict mode, this is global)

// Strict mode detection utility
function isStrictMode2() {
  return (function() { return typeof this === "undefined"; })();
}
```

### Real-World Use Cases
- All new JavaScript projects (always use strict mode)
- Library code wrapped in IIFE with strict mode
- ES modules (automatic strict mode)
- Class declarations (automatic strict mode)
- Node.js modules (CommonJS can use strict mode)
- Legacy code migration to modern practices

### Common Mistakes
- Placing `"use strict"` after other statements (ignored)
- Assuming all code is strict (forgetting function-level scope)
- Using `"use strict"` in concatenated scripts (may break other scripts)
- Expecting `"use strict"` to work on concatenated files (scope matters)
- Not realizing classes and modules are always strict

### Best Practices
- Always enable strict mode in scripts
- Use ES modules (automatic strict mode)
- Use classes which are always strict
- Wrap library code in strict IIFE for safety
- Use linting rules to enforce strict mode
- Do not use `"use strict"` in ES modules (redundant)
- Avoid mixing strict and non-strict code in same project

### Performance Considerations
- Strict mode can improve performance (enables engine optimizations)
- Some strict mode checks may add minimal overhead
- Overall, strict mode is faster than sloppy mode in modern engines
- Optimizations like duplicate property elimination are enabled

### Interview Questions
1. How do you enable strict mode?
2. Where must `"use strict"` be placed to take effect?
3. Are ES modules always in strict mode?
4. How can you detect if strict mode is active?
5. What happens if you place `"use strict"` after a statement?

### Coding Challenges
1. Write a function that detects whether the code is running in strict mode.
2. Create a script that runs in strict mode using an IIFE.
3. Convert a sloppy mode script to strict mode.
4. Write a polyfill that works in both strict and sloppy modes.

### Related Topics
- Strict mode restrictions
- ES modules
- Classes
- this keyword

## Strict mode restrictions

### What It Is
Strict mode imposes several restrictions and changes that make JavaScript more secure, predictable, and optimizable. These restrictions turn previously silent errors into thrown exceptions.

### Why It Is Important
Understanding strict mode restrictions helps developers avoid common bugs and write safer code. The restrictions prevent accidental globals, unsafe mutations, and deprecated features.

### How It Works Internally
The engine checks for strict mode violations at runtime and throws appropriate errors. The parser also rejects some syntax in strict mode that would be valid in sloppy mode.

### Syntax
```javascript
"use strict";

// Variables must be declared
x = 10; // ReferenceError: x is not defined

// Cannot delete undeletable properties
delete Object.prototype; // TypeError

// Duplicate parameter names not allowed
function f(a, a) {} // SyntaxError

// Octal syntax not allowed
// var n = 010; // SyntaxError

// with statement not allowed
// with (obj) { } // SyntaxError

// this is undefined in functions
function show() { console.log(this); }
show(); // undefined instead of global
```

### Beginner Examples
```javascript
"use strict";

// 1. Accidental globals (must declare variables)
// total = 42; // ReferenceError
let total = 42; // OK

// 2. Assignment to non-writable properties
const obj = {};
Object.defineProperty(obj, 'x', { value: 1, writable: false });
// obj.x = 2; // TypeError

// 3. Assignment to getter-only properties
const obj2 = { get x() { return 1; } };
// obj2.x = 2; // TypeError

// 4. this in functions is undefined
function showThis() {
  console.log(this); // undefined
}
showThis();

// 5. Deleting undeletable
// delete Array.prototype; // TypeError
```

### Intermediate Examples
```javascript
"use strict";

// Duplicate property names (allowed in ES6+)
const obj3 = { x: 1, x: 2 }; // OK in strict mode ES6+

// Duplicate parameter names (not allowed)
// function sum(a, a, c) { } // SyntaxError

// Octal literals (not allowed)
// var octal = 010; // SyntaxError
// Use 0o10 syntax instead (ES6+)
var octal = 0o10; // OK

// with statement (not allowed)
// with (Math) { var result = sqrt(25); } // SyntaxError

// eval does not introduce new variables to surrounding scope
eval("var y = 100");
// console.log(y); // ReferenceError (y is not defined)

// arguments.callee (not allowed)
function f() {
  // arguments.callee(); // TypeError
}

// Primitives cannot be assigned properties
"use strict";
// "hello".test = 1; // TypeError
```

### Advanced Examples
```javascript
"use strict";

// arguments does not track parameter changes
function trackChanges(a) {
  a = 100;
  console.log(arguments[0]); // undefined (not 100 in strict mode)
  // In sloppy mode, arguments[0] would be 100
}
trackChanges(5);

// 'this' in event handlers (browser)
// button.addEventListener('click', function() {
//   console.log(this); // The button element (consistent in strict mode)
// });

// 'caller' and 'arguments' on functions
function outer() {
  inner();
}
function inner() {
  // console.log(inner.caller); // TypeError
  // console.log(arguments.callee); // TypeError
}

// Reserved words cannot be used as variable names
// let implements = 1; // SyntaxError (ES3 reserved word)
// You can use these in ES6+ strict mode as property names but not identifiers

// Future reserved words (implements, interface, let, package, private, protected, public, static, yield)
// let static = 1; // SyntaxError in some contexts

// Binding eval or arguments
// var eval = 1; // SyntaxError
// var arguments = 1; // SyntaxError
// function g(eval) {} // SyntaxError
```

### Real-World Use Cases
- Preventing accidental global variables
- Making `this` behavior predictable in non-method functions
- Preventing unsafe property deletion
- Catching duplicate parameter bugs
- Avoiding octal number confusion
- Preventing eval from leaking scope
- Enabling engine optimizations

### Common Mistakes
- Not realizing `arguments` doesn't sync with parameters in strict mode
- Forgetting that `this` is `undefined` in standalone function calls
- Trying to use `with` for convenience (not supported)
- Expecting `delete` to work on variables (SyntaxError)
- Using octal literals without `0o` prefix

### Best Practices
- Always declare variables with `let`, `const`, or `var`
- Use `0o` prefix for octal numbers
- Use arrow functions or `.bind()` for `this` binding
- Avoid `arguments.callee` (use named functions)
- Avoid `eval` entirely (use `JSON.parse` for JSON)
- Use rest parameters instead of `arguments`
- Prefer ES modules over script-level strict mode

### Performance Considerations
- Strict mode restrictions enable better engine optimizations
- Errors are thrown instead of silently failing (runtime cost only on error)
- No performance penalty for using strict mode
- Some optimizations (like scope optimizations) are only possible in strict mode

### Interview Questions
1. What are five restrictions of strict mode?
2. How does strict mode change the behavior of `this`?
3. What happens if you assign to an undeclared variable in strict mode?
4. Why is `with` not allowed in strict mode?
5. How does `arguments` behave differently in strict mode?

### Coding Challenges
1. Write code that demonstrates three different strict mode restrictions.
2. Fix a sloppy-mode function that uses undeclared variables and duplicate parameters.
3. Create a function that works in both strict and sloppy modes using feature detection.
4. Write code that triggers a TypeError in strict mode for each restriction type.

### Related Topics
- use strict directive
- Benefits of strict mode
- Sloppy mode vs strict mode

## Benefits of strict mode

### What It Is
Strict mode provides several benefits for code quality, security, and performance. It transforms silent errors into visible exceptions, prevents unsafe patterns, and enables engine optimizations.

### Why It Is Important
The benefits of strict mode directly improve code quality and developer experience. Catching errors early, preventing common bugs, and enabling optimizations make strict mode essential for production code.

### How It Works Internally
The engine can optimize strict mode code more aggressively because it makes guarantees about variable scope, parameter behavior, and property access. Errors that were silently ignored now throw exceptions, allowing bugs to be caught early.

### Syntax
```javascript
"use strict";

// Benefits manifest as:
// 1. Clear errors instead of silent failures
// 2. Predictable this binding
// 3. Secure eval
// 4. No accidental globals
// 5. Better optimization opportunities
```

### Beginner Examples
```javascript
"use strict";

// Benefit 1: No accidental globals
function calculate() {
  // total = 42; // ReferenceError instead of creating a global
  const total = 42; // Correct
}
calculate();

// Benefit 2: Predictable this
function showContext() {
  console.log(this); // undefined (not window/global)
}
showContext();

// Benefit 3: No silent failures
const config = {};
Object.defineProperty(config, 'apiKey', { value: 'secret', writable: false });
// config.apiKey = 'new'; // TypeError instead of silent failure

// Benefit 4: eval doesn't leak
eval("var secret = 'hidden'");
// console.log(secret); // ReferenceError

// Benefit 5: Duplicate parameter detection
// function add(a, a, b) { } // SyntaxError
```

### Intermediate Examples
```javascript
"use strict";

// Performance optimization (engines can assume variable rules)
function optimizedSum() {
  "use strict";
  let total = 0;
  for (let i = 0; i < arguments.length; i++) {
    total += arguments[i];
  }
  return total;
}

// Security: preventing this coercion
function safeConstructor(value) {
  "use strict";
  // If called without `new`, `this` is undefined
  if (typeof this === "undefined") {
    return new safeConstructor(value);
  }
  this.value = value;
}

// Predictable delete behavior
var obj4 = { x: 1, y: 2 };
// delete obj4; // SyntaxError (cannot delete variables)
delete obj4.x; // OK (deleting property)

// Better error reporting
function processData(data) {
  "use strict";
  // Object.freeze prevents modifications
  const frozen = Object.freeze(data);
  // frozen.name = "test"; // TypeError with clear message
}

// Secure eval (eval in own scope)
function secureEval(code) {
  "use strict";
  return eval(code); // Can't leak to enclosing scope
}
```

### Advanced Examples
```javascript
"use strict";

// Optimization: Engines can avoid prototype chain checks
// In strict mode, `this` is never implicitly wrapped as an object

// Optimization: Property access is faster
// No need to check for getters/setters in some cases

// Optimization: Variable access
// Engines optimize lexical environments better in strict mode

// Security: Preventing function stack manipulation
function secureFunction() {
  "use strict";
  // arguments.callee and .caller are disabled
  // Prevents untrusted code from traversing call stack
}

// Consistency across environments
function strictOrSloppy() {
  "use strict";
  // Behavior is identical across all browsers
  // No environmental quirks
}

// Migration helper: converting sloppy to strict
function migrateExample() {
  "use strict";
  // Hoisting still works, but TDZ catches issues
  // console.log(x); // ReferenceError (TDZ)
  let x = 42;
  
  // No octal confusion
  // var year = 010; // SyntaxError (not 8)
  var year = 0o10; // Explicit octal (8)
}
```

### Real-World Use Cases
- All production JavaScript code
- Library development (ensures compatibility)
- Team projects (catches errors across developers)
- Large codebases (prevents common bugs)
- Performance-critical code (enables optimizations)
- Security-sensitive applications (prevents dangerous patterns)
- Cross-platform code (consistent behavior)

### Common Mistakes
- Thinking strict mode is optional (should always be used)
- Not using strict mode in new projects
- Assuming strict mode has no effect on performance (it improves it)
- Migrating to strict mode without testing all code paths
- Not realizing that concatenated scripts may break with strict mode

### Best Practices
- Use strict mode in all code by default
- Use ES modules (automatic strict mode)
- Use TypeScript (compiles to strict mode)
- Enable ESLint strict rules
- Test thoroughly when migrating code to strict mode
- Use build tools that add `"use strict"` automatically
- Prefer classes which are always strict

### Performance Considerations
- Strict mode enables V8 optimization tricks
- Some functions compile faster in strict mode
- Property access can be optimized better
- Variable lookups are faster in strict mode
- Overall, strict mode code runs faster than equivalent sloppy code

### Interview Questions
1. What are the main benefits of strict mode?
2. How does strict mode improve security?
3. How does strict mode help with debugging?
4. Can strict mode improve performance? How?
5. How does strict mode make eval safer?

### Coding Challenges
1. Write a comparison of the same code in strict and sloppy mode showing differences.
2. Create a function that works correctly in strict mode but would fail silently in sloppy mode.
3. Migrate a legacy function to strict mode and document changes.
4. Write a performance test comparing strict vs sloppy mode operations.

### Related Topics
- Strict mode restrictions
- use strict directive
- Module strict mode

## Module-level strict mode

### What It Is
ES modules (files using `import`/`export`) are always in strict mode automatically, regardless of whether `"use strict"` is specified. This includes all code within the module, including function bodies and class definitions.

### Why It Is Important
Module-level strict mode means that all modern JavaScript using modules automatically gets the benefits of strict mode without explicit directives. This improves security and performance for module-based applications.

### How It Works Internally
The parser detects that a file is a module (by presence of `import`/`export` or by `type="module"` in HTML or `.mjs` extension) and automatically sets the strict mode flag for the entire module.

### Syntax
```javascript
// No "use strict" needed in modules

export function example() {
  // Already in strict mode
  this; // undefined in standalone calls
}

// Equivalent to:
"use strict";
export function example() {
  // ...
}
```

### Beginner Examples
```javascript
// module.js - no "use strict" needed
export const PI = 3.14159;

// Already strict: implicit globals throw
// total = 42; // ReferenceError (would be caught)

export function calculate(radius) {
  // Strict mode is active
  return PI * radius * radius;
}

// app.js
import { calculate } from './module.js';
console.log(calculate(5));

// this at module top level is undefined
console.log(this); // undefined (not window)
```

### Intermediate Examples
```javascript
// strict-module.mjs (.mjs extension indicates module)
export class StrictClass {
  constructor(value) {
    // Class body is always strict
    this.value = value;
  }
  
  getValue() {
    // Strict mode active
    return this.value;
  }
}

// Dynamic import inherits strict mode
// (Dynamic imports load ES modules, which are strict)

// CommonJS vs ESM strict behavior
// CommonJS (require) is NOT automatically strict
// ESM (import) IS automatically strict

// Module-imported functions remain strict
import { strictFn } from './module.js';
function sloppy() {
  // This function in a non-module script is NOT automatically strict
  // But if called from a module, it still follows its own strictness
}
```

### Advanced Examples
```javascript
// Module-level strict mode demonstration
// module-helper.js
export function checkModuleStrict() {
  // Returns true because modules are always strict
  return (function() { return typeof this === "undefined"; })();
}

// strict-vs-sloppy.js (module - strict)
export function createStrictFn() {
  "use strict"; // Redundant but harmless
  return function() {
    return !this; // true (strict)
  };
}

// Interaction with non-module (script) code
// script.js (non-module, no "use strict")
// var x = 10; // This is global
// <script src="module.js" type="module"></script>
// Module code is strict regardless of surrounding scripts

// Checking if a module is in strict mode
export const isStrict = (() => typeof this === "undefined")();
// Note: module top-level `this` is undefined, so isStrict is technically true
// but `this` check doesn't work at top level of modules

// Better check
export const isDefinitelyStrict = (() => {
  "use strict";
  return !function() { return this; }();
})();
```

### Real-World Use Cases
- All ES module-based applications
- Front-end frameworks (React, Vue, Angular with ESM)
- Node.js ESM applications (type: "module" in package.json)
- Build tool configurations using ESM
- Any code using `import`/`export` syntax
- Web components using ES modules
- Library authors writing ESM packages
- Code-split applications using dynamic imports

### Common Mistakes
- Adding `"use strict"` in ES modules unnecessarily
- Assuming CommonJS files are automatically strict (they are not)
- Forgetting that top-level `this` in modules is `undefined`
- Mixing module and script type code without understanding strictness
- Not realizing that `import`ed functions run in strict mode (if the imported module uses it)

### Best Practices
- Use ES modules for all new code (automatic strict mode)
- Use `type="module"` in HTML scripts
- Use `type: "module"` in package.json for Node.js
- Use `.mjs` extension for Node.js modules
- Do not add `"use strict"` in module files (redundant)
- Use build tools that emit ESM for modern browsers
- Prefer ESM over CommonJS for new projects

### Performance Considerations
- Module strict mode enables same optimizations as explicit strict mode
- Modules are deferred by default (improves initial load)
- Module scripts are cached (better performance)
- Static analysis of imports enables better tree-shaking
- Module scope is isolated (reduces global scope pollution)

### Interview Questions
1. Are ES modules always in strict mode?
2. Do you need `"use strict"` in module files?
3. What is the value of `this` at the top level of a module?
4. How does module strict mode differ from script strict mode?
5. What happens when a module imports a CommonJS module (regarding strictness)?

### Coding Challenges
1. Create an ES module and demonstrate automatic strict mode.
2. Compare `this` behavior in a module vs a script at the top level.
3. Write a module that exports a function and verify it runs in strict mode.
4. Create a hybrid project with both strict (module) and sloppy (script) files.

### Related Topics
- ES modules
- use strict directive
- Strict mode benefits
- Type="module" in HTML
