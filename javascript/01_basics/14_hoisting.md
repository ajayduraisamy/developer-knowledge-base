# Hoisting - var hoisting, function hoisting, TDZ, let/const hoisting

## Introduction
Hoisting is JavaScript's behavior of moving declarations to the top of their containing scope during the compilation phase. Understanding hoisting is essential for predicting variable availability and avoiding reference errors.

## var hoisting

### What It Is
`var` declarations are hoisted to the top of their enclosing function or global scope. The declaration is hoisted but the initialization remains in place. The variable is initialized with `undefined` until the assignment is reached.

### Why It Is Important
`var` hoisting explains why variables can be referenced before their declaration (returning `undefined` instead of throwing an error). Understanding this behavior is crucial for debugging legacy code and understanding JavaScript's execution model.

### How It Works Internally
During the compilation phase, the engine scans for `var` declarations and creates variable bindings in the current scope, initializing them to `undefined`. During the execution phase, when the assignment is reached, the value is set. This two-phase process gives the illusion that the declaration "moved" to the top.

### Syntax
```javascript
console.log(x); // undefined (not ReferenceError)
var x = 10;
console.log(x); // 10

// The above is interpreted as:
var x; // Hoisted declaration
console.log(x); // undefined
x = 10; // Initialization stays
console.log(x); // 10
```

### Beginner Examples
```javascript
// Basic var hoisting
console.log(hoisted); // undefined
var hoisted = "value";
console.log(hoisted); // "value"

// Hoisting in function scope
function demo() {
  console.log(temp); // undefined
  var temp = "local";
  console.log(temp); // "local"
}
demo();

// Multiple var declarations are combined
var a = 1;
var a = 2; // Re-declaration, same variable
console.log(a); // 2

// Hoisting with conditional
function conditionalHoist() {
  console.log(x); // undefined (hoisted, not assigned)
  if (false) {
    var x = 100; // Still hoisted!
  }
  console.log(x); // undefined (assignment not reached)
}
```

### Intermediate Examples
```javascript
// Hoisting across function boundaries
var globalVar = "initial";
function test() {
  console.log(globalVar); // undefined (not "initial"!)
  var globalVar = "local";
  console.log(globalVar); // "local"
}
test();
// The local var "shadows" the global, and is hoisted to function top

// Hoisting with nested functions
function outer() {
  console.log(inner); // undefined (hoisted)
  var inner = "function-scoped";
  function innerFunc() {
    console.log(inner); // "function-scoped" (from outer scope)
  }
  innerFunc();
}

// Hoisting in loops
function loopHoist() {
  console.log(i); // undefined
  for (var i = 0; i < 3; i++) {
    // loop body
  }
  console.log(i); // 3 (accessible after loop)
}
```

### Advanced Examples
```javascript
// Hoisting and the module pattern
var Module = (function() {
  // All var declarations are hoisted to IIFE scope
  var privateVar = "secret";
  var publicVar = "public";
  
  // This is hoisted
  function privateMethod() {
    return privateVar;
  }
  
  return {
    getSecret: function() {
      return privateMethod();
    },
    publicVar: publicVar
  };
})();

// Multiple declarations and assignments
function hoistingOrder() {
  var a = 1;
  var b = function() { return a; };
  var a = 2; // Re-declares and reassigns
  console.log(b()); // 1 (b captures the variable, not the value)
  console.log(a);   // 2
}

// Hoisting with eval
function evalHoisting() {
  eval("var hoistedByEval = 'from eval'");
  console.log(hoistedByEval); // "from eval" (eval creates vars in current scope)
}

// Hoisting in try/catch
function tryCatchHoisting() {
  console.log(x); // undefined
  try {
    // something
  } catch (e) {
    var x = 10;
  }
  console.log(x); // undefined (assignment in catch didn't run)
}
```

### Real-World Use Cases
- Understanding legacy code that uses `var`
- Debugging "undefined" values from hoisted variables
- Refactoring old codebases to modern syntax
- JavaScript engine internals understanding

### Common Mistakes
- Expecting a ReferenceError for accessing `var` before declaration
- Assuming `var` is block-scoped (hoisting ignores blocks)
- Not realizing that `var` redeclaration is allowed
- Confusing hoisting of `var` with `let`/`const` temporal dead zone
- Forgetting that assignments are NOT hoisted, only declarations

### Best Practices
- Never rely on `var` hoisting in new code
- Use `let` and `const` exclusively
- Declare variables at the top of their scope if using `var`
- Enable ESLint `no-var` rule to prevent `var` usage
- Enable strict mode to catch undeclared variable assignments

### Performance Considerations
- Hoisting itself has no runtime cost (done at compile time)
- `var` variables are accessible throughout the function scope (may prevent GC)
- Modern engines optimize `var` efficiently
- The TDZ for `let`/`const` has negligible overhead

### Interview Questions
1. Explain hoisting in JavaScript.
2. What is the difference between `var` hoisting and `let`/`const` hoisting?
3. What value does a hoisted `var` have before initialization?
4. Are function expressions hoisted?
5. How does hoisting affect the module pattern?

### Coding Challenges
1. Write a code snippet that demonstrates var hoisting with a function.
2. Predict the output of a function with multiple hoisted vars.
3. Refactor a `var`-based code to use `let`/`const`, explaining hoisting changes.
4. Create a function that demonstrates the difference between hoisting of declarations vs assignments.

### Related Topics
- let and const hoisting
- Function hoisting
- Temporal Dead Zone
- Scope

## Function hoisting

### What It Is
Function declarations are hoisted entirely, meaning both the declaration and the function body are moved to the top of their enclosing scope. This allows calling functions before they appear in the source code.

### Why It Is Important
Function hoisting enables recursive patterns and allows organizing functions in any order within a scope, making code more readable. It differs fundamentally from variable hoisting where only the declaration (not the value) is hoisted.

### How It Works Internally
During the compilation phase, the engine creates the function object and binds it to the identifier before executing any code. The entire function definition is hoisted, not just the name. This is because function declarations produce a binding that includes the function value.

### Syntax
```javascript
// This works because function declarations are hoisted
result = add(5, 3); // 8

function add(a, b) {
  return a + b;
}
```

### Beginner Examples
```javascript
// Calling before declaration
greet(); // "Hello, World!"

function greet() {
  console.log("Hello, World!");
}

// Function hoisting in conditional blocks (behavior varies)
if (true) {
  function sayHi() {
    console.log("Hi");
  }
} else {
  function sayHi() {
    console.log("Hello");
  }
}
// In non-strict mode, last definition wins (browser-dependent)
// In strict mode, block-scoped function declaration

// Hoisting across function boundaries
function outer() {
  inner(); // Works! Hoisted to outer scope
  function inner() {
    console.log("Inner function");
  }
}
outer();
```

### Intermediate Examples
```javascript
// Function hoisting precedence
function hoistingPrecedence() {
  function test() { return "function declaration"; }
  var test = function() { return "function expression"; };
  return test();
}
console.log(hoistingPrecedence()); // "function expression" (var assignment overrides)

// Function vs var hoisting (var wins in assignment)
var foo = "variable";
function foo() { return "function"; }
console.log(foo); // "variable" (var assignment overrides function)

// Named function expression hoisting
// Named function expression creates a binding only inside the function
const func = function namedFunc() {
  console.log(namedFunc); // Works (namedFunc is in-scope here)
};
// console.log(namedFunc); // ReferenceError

// Recursion through hoisting
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2); // fibonacci is hoisted
}
```

### Advanced Examples
```javascript
// Function hoisting with conditionals (strict mode)
"use strict";
function conditionalFunctions() {
  // In strict mode, this is implementation-dependent
  if (false) {
    function neverDefined() {
      return "never";
    }
  }
  // console.log(neverDefined); // ReferenceError in strict mode
}

// Function hoisting and module scope
// In ES modules, function declarations are hoisted to the module scope
helper(); // Works
function helper() {
  return "helper";
}

// Hoisting order: function declarations before var declarations
function hoistOrder() {
  console.log(typeof foo); // "function"
  var foo = "bar";
  function foo() { return "baz"; }
  console.log(typeof foo); // "string"
}

// Generator function hoisting
generatorDemo(); // Works!
function* generatorDemo() {
  yield 1;
  yield 2;
}
```

### Real-World Use Cases
- Organizing helper functions at the bottom of a file
- Recursive function patterns
- Callback function definitions before their usage
- Legacy code organization patterns
- Module pattern with private helper functions

### Common Mistakes
- Assuming function expressions are hoisted (they are not)
- Using function declarations inside `if` blocks (inconsistent behavior)
- Confusing function hoisting order with variable hoisting
- Expecting `class` declarations to be hoisted (they are not)
- Relying on function hoisting in conditional blocks

### Best Practices
- Use function declarations for top-level functions (benefit from hoisting)
- Define functions before use for clarity (despite hoisting)
- Avoid function declarations inside conditional blocks
- Use `const` with arrow functions for function expressions (no hoisting)
- Use function declarations for recursive functions

### Performance Considerations
- Function hoisting has no runtime performance impact
- Function declarations create the function once at compile time
- Function expressions inside functions create new functions each execution
- Hoisting allows better JIT optimization in some cases

### Interview Questions
1. What is the difference between hoisting of function declarations and function expressions?
2. Can you call a function expression before it's defined?
3. How does function hoisting interact with var hoisting?
4. Are class declarations hoisted?
5. What happens when you declare a function inside an if/else block?

### Coding Challenges
1. Write a recursive function that benefits from hoisting.
2. Demonstrate hoisting precedence between function declarations and var declarations.
3. Refactor code that relies on conditional function declarations to be clear and correct.
4. Create a function that calls itself recursively before its declaration.

### Related Topics
- Function declarations
- var hoisting
- let and const hoisting
- TDZ

## Temporal Dead Zone (TDZ)

### What It Is
The Temporal Dead Zone (TDZ) is the period between entering a scope and the actual declaration of a `let` or `const` variable where accessing the variable throws a ReferenceError.

### Why It Is Important
The TDZ prevents accessing variables before initialization, catching bugs early. It replaces the confusing `undefined` behavior of `var` hoisting with clear errors.

### How It Works Internally
When scope is entered, `let` and `const` variables are registered in the lexical environment but marked as "uninitialized." Any access to an uninitialized binding throws a ReferenceError. When the declaration statement is reached, the variable is initialized with the assigned value or `undefined`.

### Syntax
```javascript
{
  // TDZ starts
  // console.log(x); // ReferenceError: Cannot access 'x' before initialization
  const x = 10; // TDZ ends
  console.log(x); // 10
}
```

### Beginner Examples
```javascript
// Basic TDZ
{
  // TDZ for `x`
  // console.log(x); // ReferenceError
  let x = 5;
  console.log(x); // 5
}

// TDZ with const
{
  // console.log(y); // ReferenceError
  const y = 10;
  console.log(y); // 10
}

// TDZ in functions
function tdzDemo() {
  // console.log(z); // ReferenceError (TDZ)
  let z = 20;
  console.log(z); // 20
}

// TDZ with typeof
{
  // console.log(typeof notDeclared); // "undefined" (safe)
  // console.log(typeof a); // ReferenceError! (TDZ for `a`)
  let a = 1;
}
```

### Intermediate Examples
```javascript
// TDZ in conditional blocks
function tdzConditional(condition) {
  if (condition) {
    // console.log(value); // ReferenceError (TDZ)
    let value = 42;
    return value;
  }
}

// TDZ with default parameters
function defaultParam(a = b, b = 1) {
  // a tries to access b which is in TDZ
  return a;
}
// console.log(defaultParam()); // ReferenceError: Cannot access 'b' before initialization

// TDZ in for loops
function tdzFor() {
  // console.log(i); // ReferenceError (TDZ)
  for (let i = 0; i < 3; i++) {
    console.log(i); // Works
  }
}

// TDZ and closure
function tdzClosure() {
  const funcs = [];
  for (let i = 0; i < 3; i++) {
    funcs.push(() => i); // Each closure captures a different `i`
  }
  return funcs;
}
```

### Advanced Examples
```javascript
// TDZ and class inheritance
class Parent {
  constructor() {
    this.value = 42;
  }
}
class Child extends Parent {
  constructor() {
    super(); // Must call super before accessing `this`
    this.childValue = this.value;
  }
}
// Accessing `this` before super() is like a TDZ

// TDZ with destructuring
function tdzDestructuring() {
  // let { x, y } = { x: a, y: b }; // TDZ applies here too
  let a = 1, b = 2;
  let { x, y } = { x: a, y: b };
}

// TDZ in switch cases
function tdzSwitch(val) {
  switch (val) {
    case 0:
      // let x = 10; // TDZ for `x` in case 1
      break;
    case 1:
      // console.log(x); // ReferenceError if case 0 didn't run
      let x = 20;
      break;
  }
}

// TDZ and block-level function declarations (strict mode)
"use strict";
{
  // console.log(f()); // ReferenceError (TDZ for block-scoped function)
  function f() { return "ok"; }
  console.log(f()); // "ok"
}
```

### Real-World Use Cases
- Catching variable access errors during development
- Enforcing proper initialization order
- Safe data access patterns
- Class inheritance ensuring super() is called
- Module-level variable initialization order

### Common Mistakes
- Expecting `typeof` to return "undefined" for variables in TDZ (throws error)
- Assuming `let` is not hoisted at all (it is, but with TDZ)
- Creating circular dependencies with default parameters
- Forgetting to call `super()` before accessing `this` in class constructors
- Using variables in their own initialization

### Best Practices
- Declare `let` and `const` at the top of their scope
- Never access a variable before its declaration
- Use `const` by default to signal initialization happens at declaration
- Use `let` only when reassignment is needed
- Always call `super()` before accessing `this` in derived classes
- Avoid complex initialization dependencies between variables

### Performance Considerations
- TDZ checks have negligible runtime cost
- The error is thrown during execution, not compilation
- Modern engines optimize TDZ checks away in many cases
- No performance difference between TDZ and non-TDZ access after initialization

### Interview Questions
1. What is the Temporal Dead Zone?
2. Which declarations are affected by TDZ?
3. Why does `typeof` throw an error for variables in TDZ but not undeclared variables?
4. How does TDZ affect function default parameters?
5. How does TDZ relate to class inheritance?

### Coding Challenges
1. Write code that demonstrates TDZ in a block scope.
2. Create a scenario where TDZ causes an error with default parameters.
3. Refactor code to avoid TDZ errors.
4. Implement a function that catches and handles TDZ errors.

### Related Topics
- let and const
- Hoisting
- var vs let/const
- Class inheritance

## let and const hoisting

### What It Is
`let` and `const` are hoisted (their declarations are moved to the top of their block), but unlike `var`, they are not initialized with `undefined`. They remain in the Temporal Dead Zone until the declaration is reached.

### Why It Is Important
Understanding that `let` and `const` are hoisted helps explain the TDZ behavior. The variable exists in the scope from the beginning but cannot be accessed until initialization.

### How It Works Internally
During the compilation phase, the engine creates bindings for `let` and `const` in the lexical environment. These bindings are initialized with a special "uninitialized" sentinel value. Runtime access checks detect this state and throw ReferenceError.

### Syntax
```javascript
{
  // x exists here (hoisted) but is uninitialized (TDZ)
  // console.log(x); // ReferenceError
  let x = 10;
  console.log(x); // 10
}

{
  // y exists here but is uninitialized
  // console.log(y); // ReferenceError
  const y = 20;
  console.log(y); // 20
}
```

### Beginner Examples
```javascript
// let is hoisted with TDZ
{
  console.log(typeof notDeclared); // "undefined" (no variable)
  // console.log(typeof x); // ReferenceError (x exists but in TDZ)
  let x = 10;
}

// const is hoisted with TDZ
function constHoisting() {
  // console.log(PI); // ReferenceError
  const PI = 3.14159;
  console.log(PI); // 3.14159
}

// Redeclaration detection
function redeclareTest() {
  let a = 1;
  // let a = 2; // SyntaxError: Identifier 'a' has already been declared
}

// Hoisting across function boundaries (not possible)
function scopeTest() {
  if (true) {
    let blockVar = "block";
    console.log(blockVar); // "block"
  }
  // console.log(blockVar); // ReferenceError (not hoisted to function scope)
}
```

### Intermediate Examples
```javascript
// let in for loops (new binding per iteration)
function forLetHoisting() {
  const funcs = [];
  for (let i = 0; i < 3; i++) {
    // Each iteration has its own `i` (lexical per-iteration binding)
    funcs.push(() => i);
  }
  console.log(funcs.map(f => f())); // [0, 1, 2]
}

// const requires initialization
function constInit() {
  // const VALUE; // SyntaxError: Missing initializer in const declaration
  const VALUE = 42;
  console.log(VALUE);
}

// TDZ with temporal order
function tdzOrder() {
  function foo() {
    console.log(bar); // Can access bar (not in TDZ at call time?)
  }
  let bar = 10;
  foo(); // 10 (bar is already initialized when foo runs)
}

// typeof behavior difference
{
  // console.log(typeof undeclared); // "undefined"
  // console.log(typeof declared); // ReferenceError
  let declared = 1;
}
```

### Advanced Examples
```javascript
// Hoisting with multiple let declarations in same scope
function multiLet() {
  // console.log(a); // ReferenceError
  // console.log(b); // ReferenceError
  let a = 1;
  let b = 2;
  console.log(a + b); // 3
}

// let in switch - TDZ between cases
function switchTDZ(value) {
  switch (value) {
    case 0:
      let x = 10;
      break;
    case 1:
      // TDZ for x here if case 0 didn't run!
      // console.log(x); // ReferenceError
      break;
  }
}

// Block-scoped function declarations (strict mode)
"use strict";
{
  // console.log(f); // ReferenceError (TDZ)
  function f() { return 1; }
  console.log(f()); // 1
}

// Combining var and let in same scope
function mixedHoisting() {
  var x = 1;
  let x = 2; // SyntaxError: Identifier 'x' has already been declared
}
```

### Real-World Use Cases
- Understanding why `let`/`const` cannot be accessed before declaration
- Migrating from `var` to `let`/`const` in legacy codebases
- Properly structuring initialization order in complex functions
- Class constructor initialization order

### Common Mistakes
- Assuming `let` is NOT hoisted (it is, but with TDZ)
- Expecting `typeof` to work on `let`/`const` before declaration
- Confusing hoisting of `var` (initialized with undefined) with `let` (TDZ)
- Declaring `let` after usage in the same scope
- Forgetting `const` must be initialized at declaration

### Best Practices
- Declare all `let` and `const` at the top of their scope
- Use `const` by default, `let` only when reassignment is needed
- Never rely on accessing variables before their declaration
- Use block scoping consistently
- Group variable declarations by usage proximity
- Use linting rules (ESLint `no-use-before-define`) to catch TDZ issues

### Performance Considerations
- TDZ checks are minimal overhead
- Engine optimizes away TDZ when it can prove safety
- `let` and `const` in loops create per-iteration bindings (minor overhead)
- The safety of TDZ is worth the negligible performance cost

### Interview Questions
1. Are `let` and `const` hoisted? Explain.
2. What is the difference between `var` hoisting and `let`/`const` hoisting?
3. Why is `typeof` not safe for `let`/`const` before declaration?
4. How does the TDZ prevent certain bugs?
5. What happens if you try to access a `let` variable in a switch case before its definition?

### Coding Challenges
1. Write code that demonstrates the TDZ difference between `var` and `let`.
2. Create a scenario where `let` hoisting with TDZ prevents a bug.
3. Refactor a `var`-based code snippet to `let`/`const` and explain hoisting changes.
4. Implement a function that catches when a `let` variable is in TDZ.

### Related Topics
- TDZ
- var hoisting
- let and const
- Block scope
