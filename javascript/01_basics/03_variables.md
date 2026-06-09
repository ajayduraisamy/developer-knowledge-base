# Variables - var, let, const, scope, naming rules

## Introduction
Variables are fundamental building blocks in JavaScript that store data values. Understanding how variable declarations work, their scoping rules, and when to use each type is crucial for writing correct, maintainable code. JavaScript provides three keywords for declaring variables: `var`, `let`, and `const`, each with distinct behaviors.

## var declaration

### What It Is
`var` is the original variable declaration keyword in JavaScript, available since the language's creation. Variables declared with `var` are function-scoped and hoisted to the top of their enclosing function or global scope.

### Why It Is Important
While `var` is largely superseded by `let` and `const` in modern JavaScript, understanding `var` is essential for maintaining legacy codebases, understanding JavaScript's evolution, and grasping fundamental concepts like hoisting and function scope.

### How It Works Internally
When the JavaScript engine encounters a `var` declaration, it hoists the declaration (not the initialization) to the top of the enclosing function scope during the compilation phase. The variable is initialized with `undefined` and gets its assigned value only when the execution reaches the assignment line. Variables declared with `var` outside any function become properties of the global object (`window` in browsers, `global` in Node.js).

### Syntax
```javascript
var variableName = value;

// Multiple declarations
var a = 1, b = 2, c = 3;

// Declaration without initialization
var x;
console.log(x); // undefined

// Re-declaration is allowed
var y = 10;
var y = 20; // No error
```

### Beginner Examples
```javascript
// Basic var usage
var name = "Alice";
console.log(name); // "Alice"

// var hoisting behavior
console.log(hoistedVar); // undefined (not ReferenceError!)
var hoistedVar = "I'm hoisted";

// Function scope example
function example() {
  var inside = "I'm inside the function";
  console.log(inside); // "I'm inside the function"
}
example();
// console.log(inside); // ReferenceError: inside is not defined

// var does NOT have block scope
if (true) {
  var blockVar = "I'm in a block";
}
console.log(blockVar); // "I'm in a block" - accessible outside block!
```

### Intermediate Examples
```javascript
// Hoisting with loops
for (var i = 0; i < 5; i++) {
  setTimeout(function() {
    console.log(i); // Prints 5, 5, 5, 5, 5 (not 0,1,2,3,4)
  }, 100);
}

// var in a function creates a scope boundary
function count() {
  for (var j = 0; j < 3; j++) {
    // j is scoped to count()
  }
  console.log(j); // 3 - accessible after loop
}

// Global scope pollution
var globalVar = "I'm global";
console.log(window.globalVar); // "I'm global" (in browsers)

// Re-declaration in same scope
var message = "Hello";
var message = "World"; // No error, overrides
console.log(message); // "World"

// Undeclared assignment (creates global)
function badPractice() {
  leak = "I become global"; // No var/let/const!
}
badPractice();
console.log(leak); // "I become global"
```

### Advanced Examples
```javascript
// IIFE pattern to create scope (pre-ES6)
var result = (function() {
  var privateVar = "secret";
  return function() {
    return privateVar;
  };
})();

// Self-executing function for module pattern
var Module = {};
(function(exports) {
  var privateData = [];
  exports.add = function(item) {
    privateData.push(item);
  };
  exports.getAll = function() {
    return [...privateData];
  };
})(Module);

// var in switch statements
var value = "b";
switch (value) {
  case "a":
    var msg = "Option A";
    break;
  case "b":
    var msg = "Option B"; // Same variable!
    break;
}
console.log(msg); // "Option B"

// Hoisting order of var declarations
function hoistingOrder() {
  console.log(a); // undefined (hoisted)
  if (false) {
    var a = 100; // Still hoisted despite unreachable code
  }
  console.log(a); // undefined
}
```

### Real-World Use Cases
- Legacy codebases (pre-ES6)
- Older Node.js modules written before ES6 adoption
- Understanding JavaScript fundamentals for interviews
- Global variable declaration in browser scripts
- Teaching how hoisting works conceptually

### Common Mistakes
- Assuming `var` is block-scoped (it's function-scoped)
- Using `var` in loops with closures, causing the classic "loop closure" bug
- Accidentally creating global variables by omitting `var` in assignment
- Re-declaring variables unintentionally in the same scope
- Relying on hoisting behavior for readability

### Best Practices
- Avoid `var` in new code; use `let` and `const` instead
- Declare all variables at the top of their scope if using `var`
- Enable ESLint `no-var` rule to prevent `var` usage
- Use strict mode (`"use strict"`) to prevent accidental globals
- Declare only one variable per statement for clarity

### Performance Considerations
- `var` is generally equally performant to `let` in modern engines
- The hoisting mechanism has negligible overhead
- Global variables from `var` can cause property lookup overhead on `window`
- Function-scoped variables are cleaned up by GC when the function exits

### Interview Questions
1. What is the difference between `var`, `let`, and `const`?
2. Explain hoisting with `var`.
3. What happens when you assign to an undeclared variable?
4. Why does `var` in a for loop behave unexpectedly with closures?
5. What is function scope vs block scope?
6. How does `var` interact with the global object?

### Coding Challenges
1. Fix the classic loop closure bug involving `var`.
2. Write a function that demonstrates hoisting order.
3. Create a module pattern using IIFE and `var`.
4. Implement a counter using `var` that demonstrates closure.

### Related Topics
- let and const declarations
- Hoisting mechanism
- Scope (function vs block)
- IIFE pattern

## let declaration

### What It Is
`let` was introduced in ES6 (2015) as a block-scoped variable declaration. Unlike `var`, `let` is not hoisted in the traditional sense (it has a Temporal Dead Zone), and it cannot be re-declared in the same scope.

### Why It Is Important
`let` is the modern replacement for `var` when reassignment is needed. Its block scoping prevents many common bugs, and the Temporal Dead Zone helps catch errors early. It is the preferred keyword for variables that need to be reassigned.

### How It Works Internally
`let` declarations are hoisted but not initialized. The engine creates the variable binding at the top of the block, but accessing it before declaration throws a `ReferenceError` due to the Temporal Dead Zone (TDZ). The variable is initialized when execution reaches the declaration line. Block scoping is enforced by the engine during variable resolution.

### Syntax
```javascript
let variableName = value;

// Block scoped
{
  let blockScoped = "Only in this block";
}
// console.log(blockScoped); // ReferenceError

// Cannot re-declare in same scope
let x = 10;
// let x = 20; // SyntaxError: Identifier 'x' has already been declared

// Can be reassigned
let y = 10;
y = 20; // OK
```

### Beginner Examples
```javascript
// Basic let usage
let name = "Bob";
console.log(name); // "Bob"

// Block scope demonstration
let global = "global";
{
  let local = "local";
  console.log(global); // "global"
  console.log(local);  // "local"
}
// console.log(local); // ReferenceError

// Temporal Dead Zone (TDZ)
// console.log(tdzVar); // ReferenceError: Cannot access before initialization
let tdzVar = "initialized";
console.log(tdzVar); // "initialized"

// Loop with closure (works correctly with let)
for (let i = 0; i < 5; i++) {
  setTimeout(function() {
    console.log(i); // 0, 1, 2, 3, 4
  }, 100);
}
```

### Intermediate Examples
```javascript
// let in different block types
{
  let a = 1;
  if (true) {
    let a = 2; // Different scope, shadowing outer 'a'
    console.log(a); // 2
  }
  console.log(a); // 1
}

// let in switch/case
switch (x) {
  case 0:
    let value = "zero";
    break;
  case 1:
    // let value = "one"; // SyntaxError: duplicate declaration in same block
    break;
}
// Use block-scoped cases
switch (x) {
  case 0: {
    let value = "zero";
    break;
  }
  case 1: {
    let value = "one";
    break;
  }
}

// let in try/catch
try {
  let temp = riskyOperation();
  console.log(temp);
} catch (error) {
  let message = error.message;
  console.error(message);
}
// console.log(temp); // ReferenceError
// console.log(message); // ReferenceError
```

### Advanced Examples
```javascript
// Temporal Dead Zone edge cases
function tdzEdgeCases() {
  {
    // typeof x; // ReferenceError: Cannot access 'x' before initialization
    let x = 10;
  }
  
  // TDZ with default parameters
  function test(a = b, b = 5) {
    return a;
  }
  // test(); // ReferenceError: Cannot access 'b' before initialization
}

// let in for-in and for-of
const obj = { a: 1, b: 2, c: 3 };
for (let key in obj) {
  setTimeout(() => console.log(key), 0); // a, b, c (proper binding per iteration)
}

// let and the global object
let globalLet = "not global";
console.log(window.globalLet); // undefined (in browsers)

// Redeclaration detection is lexical
function redeclareCheck() {
  let x = 1;
  if (true) {
    let x = 2; // OK, different scope
  }
  // let x = 3; // SyntaxError
}
```

### Real-World Use Cases
- Loop counters in for loops
- Variables that need reassignment (counters, accumulators)
- Temporary values within blocks
- Replacing `var` in all new code where reassignment is needed
- Managing mutable state in algorithms

### Common Mistakes
- Forgetting that `let` creates a new binding in each loop iteration
- Assuming `let` has function scope like `var`
- Trying to access a `let` variable before declaration
- Attempting to re-declare a `let` variable in the same scope
- Using `let` when `const` should be used (immutable references)

### Best Practices
- Prefer `const` by default, use `let` only when reassignment is necessary
- Declare variables as close to their use as possible
- Use `let` for loop counters and accumulator variables
- Avoid shadowing outer scope variables
- Enable ESLint `prefer-const` rule to enforce `const` usage

### Performance Considerations
- `let` may have slightly different optimization characteristics than `var`
- Modern engines optimize `let` as well as `var` in most cases
- Block scoping doesn't add measurable overhead
- The TDZ check has negligible performance impact

### Interview Questions
1. What is the Temporal Dead Zone?
2. How does `let` behave differently from `var` in loops?
3. Can you re-declare a `let` variable in the same scope?
4. Is `let` hoisted?
5. What happens if you use `typeof` on a `let` variable before declaration?

### Coding Challenges
1. Rewrite a `var`-based loop closure problem to use `let`.
2. Create a function demonstrating the TDZ with `let`.
3. Write a code snippet showing block scoping with nested blocks.
4. Fix code that incorrectly uses `var` where `let` would prevent a bug.

### Related Topics
- const declaration
- Temporal Dead Zone
- Block scope
- Hoisting

## const declaration

### What It Is
`const` was introduced in ES6 for declaring variables whose reference cannot be reassigned. The value itself may still be mutable if it is an object or array. `const` is block-scoped like `let` and also has a Temporal Dead Zone.

### Why It Is Important
`const` signals intent that a variable should not be reassigned, making code more predictable and easier to reason about. It prevents accidental reassignment bugs and is the preferred declaration keyword for most variables in modern JavaScript.

### How It Works Internally
`const` behaves identically to `let` at the engine level, except that the engine enforces that no assignment can occur after initialization. The internal slot `[[Writable]]` on the variable binding is set to `false`. For objects and arrays, the variable stores a reference (pointer), and the immutability applies only to the reference, not the underlying data.

### Syntax
```javascript
const CONSTANT_NAME = value;

// Must be initialized at declaration
// const invalid; // SyntaxError: Missing initializer in const declaration

// Cannot reassign
const x = 10;
// x = 20; // TypeError: Assignment to constant variable

// Block scoped
{
  const blockConst = "blocked";
}
// console.log(blockConst); // ReferenceError
```

### Beginner Examples
```javascript
// Basic const usage
const API_BASE_URL = "https://api.example.com/v1";
console.log(API_BASE_URL);

// const with objects (reference is constant, contents are not)
const person = { name: "Alice", age: 30 };
person.age = 31; // OK
person.city = "NYC"; // OK
// person = {}; // TypeError

// const with arrays
const colors = ["red", "green", "blue"];
colors.push("yellow"); // OK
colors[0] = "orange"; // OK
// colors = []; // TypeError

// const in loops - can't reassign, but can use in for...of
const numbers = [1, 2, 3];
for (const num of numbers) {
  console.log(num); // 1, 2, 3
}
```

### Intermediate Examples
```javascript
// const with complex object mutation
const config = {
  server: {
    host: "localhost",
    port: 3000
  },
  database: {
    url: "mongodb://localhost/db",
    pool: { min: 2, max: 10 }
  }
};

// Deep mutation is allowed
config.server.port = 4000;
config.database.pool.max = 20;

// To prevent mutation, use Object.freeze() (shallow)
const frozen = Object.freeze({ a: 1, nested: { b: 2 } });
// frozen.a = 2; // TypeError (in strict mode) or silently fails
frozen.nested.b = 3; // Allowed! (shallow freeze)

// const prevents reassignment but not method calls
const buffer = [];
buffer.push(1, 2, 3); // OK
console.log(buffer.length); // 3
```

### Advanced Examples
```javascript
// const and destructuring
const { name, age } = person; // OK, creates new variables
const [first, ...rest] = numbers; // OK

// const with closure
function createIncrementer() {
  const STEP = 1;
  return function(value) {
    return value + STEP;
  };
}
const increment = createIncrementer();
console.log(increment(5)); // 6

// const in modules - used for exports
// export const PI = 3.14159;

// const with frozen deep object (deep freeze utility)
function deepFreeze(obj) {
  Object.keys(obj).forEach(key => {
    if (typeof obj[key] === 'object' && obj[key] !== null) {
      deepFreeze(obj[key]);
    }
  });
  return Object.freeze(obj);
}
const immutableConfig = deepFreeze({ a: { b: { c: 1 } } });
// immutableConfig.a.b.c = 2; // TypeError

// const with Symbol
const UNIQUE_KEY = Symbol('key');
const registry = {};
registry[UNIQUE_KEY] = "secret value";
```

### Real-World Use Cases
- Configuration constants (API URLs, threshold values)
- Immutable references to objects that will be mutated
- Function references that should not change
- Module exports
- Enum-like constant objects
- Action type strings in Redux
- Mathematical constants (PI, E, etc.)

### Common Mistakes
- Confusing `const` with immutability (it only prevents reassignment)
- Not initializing a `const` at declaration
- Trying to use `const` in a `for` loop with incrementer (use `let` instead)
- Assuming `const` with objects makes all properties read-only
- Using `const` for values that actually need reassignment

### Best Practices
- Use `const` by default for all variables unless reassignment is needed
- Use UPPER_SNAKE_CASE for true constants (values known at compile time)
- Use camelCase for `const` references to objects/functions
- Combine `const` with `Object.freeze()` or TypeScript's `readonly` for immutability
- Always initialize `const` at declaration

### Performance Considerations
- `const` may enable engine optimizations since the binding cannot change
- Engines can potentially inline constant values
- Object mutation performance is identical whether reference is `const` or `let`
- No runtime performance penalty for using `const` over `let`

### Interview Questions
1. What does `const` actually prevent?
2. How do you make an object truly immutable?
3. Can you use `const` in a for loop?
4. What is the difference between `const` and `Object.freeze()`?
5. Why is `const` preferred over `let`?

### Coding Challenges
1. Create a `deepFreeze` utility function.
2. Implement an immutable configuration object using `const`.
3. Fix code that incorrectly tries to reassign a `const`.
4. Write a function that returns a `const` object and demonstrate mutation vs reassignment.

### Related Topics
- let declaration
- Object.freeze()
- Immutability patterns
- Block scope

## Variable naming rules

### What It Is
JavaScript has specific rules and conventions for naming variables. These include syntactic rules (what names are valid) and stylistic conventions (how names should be formatted). Following proper naming conventions improves code readability and maintainability.

### Why It Is Important
Consistent naming conventions make code more readable, predictable, and self-documenting. Following syntactic rules prevents runtime errors, while following stylistic conventions enables team collaboration and reduces cognitive load.

### How It Works Internally
The JavaScript parser validates variable names during the lexing phase. Identifiers must match the Unicode identifier rules defined in the ECMAScript specification. The engine stores variable names as strings in the lexical environment. Reserved words cannot be used as identifiers because they are part of the language grammar.

### Syntax
```javascript
// Valid naming rules
// Must start with: letter (a-z, A-Z), underscore (_), dollar sign ($)
// Can contain: letters, digits, underscores, dollar signs
// Case-sensitive: myVar and MyVar are different

// Valid examples
let firstName = "Alice";
let _private = "hidden";
let $element = document.getElementById('id');
let camelCaseExample = 42;

// Invalid examples
// let 1stName = "Alice";    // Starts with digit
// let my-var = 10;          // Hyphen not allowed
// let let = 5;              // Reserved word
// let class = "math";       // Reserved word

// Case sensitivity
let score = 10;
let Score = 20;
console.log(score); // 10
console.log(Score); // 20
```

### Beginner Examples
```javascript
// camelCase for variables and functions (most common)
let userName = "john_doe";
let itemCount = 42;
let isActive = true;
let getData = () => {};

// PascalCase for classes and constructors
class UserAccount { }
function Person(name) { }

// UPPER_SNAKE_CASE for constants
const MAX_SIZE = 100;
const API_BASE_URL = "https://api.example.com";
const DEFAULT_TIMEOUT_MS = 5000;

// Descriptive names
let totalPrice = items.reduce((sum, item) => sum + item.price, 0);
let averageRating = reviews.reduce((sum, r) => sum + r.rating, 0) / reviews.length;

// Boolean naming
let isLoggedIn = true;
let hasPermission = false;
let canEdit = true;
let shouldRetry = false;
```

### Intermediate Examples
```javascript
// Prefix conventions
// Hungarian notation prefix (rare in JS, but seen)
let strName = "Alice";   // String
let numCount = 42;       // Number
let arrItems = [];       // Array

// Verb prefixes for functions
function getUser(id) { }
function setUser(user) { }
function isValid(data) { }
function fetchData() { }
function transformPayload(payload) { }

// jQuery conventions (older code)
let $header = $('.header');
let $navItems = $('nav li');

// Private variable convention (using underscore)
class Account {
  constructor(balance) {
    this._balance = balance; // Convention: treat as private
  }
  getBalance() {
    return this._balance;
  }
}

// Plural for arrays/collections
let users = [];
let itemList = [];
let nameArray = [];
let idToUserMap = new Map();
```

### Advanced Examples
```javascript
// Destructured naming for clarity
function processUser({ name: userName, age: userAge, role: userRole }) {
  // Explicitly renamed for clarity in function scope
  console.log(userName, userAge, userRole);
}

// Factory function naming
function createUser(name, age) {
  return { name, age, createdAt: new Date() };
}
const user = createUser("Alice", 30);

// Computed property naming
const PREFIX = "user_";
const id = 123;
const dynamicKey = `${PREFIX}${id}`;
const cache = {
  [dynamicKey]: { name: "Alice" }
};

// Descriptive naming for complex logic
function calculateDiscountedPrice(
  originalPrice,
  discountPercentage,
  taxRate,
  applySeasonalAdjustment
) {
  // Clear parameter naming makes the function self-documenting
}

// Abbreviation rules
// Good: url, html, css, id, db, api
// Avoid: usrN, pwd, tmp, arr, obj, val
// OK: i, j, n (only for loop counters)
```

### Real-World Use Cases
- API response properties follow snake_case (backend) to camelCase (JS) mapping
- Database column names to JavaScript variable mapping
- Redux action types use UPPER_SNAKE_CASE
- React component names use PascalCase
- Event handlers use `handle` prefix (handleClick, handleSubmit)
- Custom hooks use `use` prefix (useState, useEffect, useCustomHook)

### Common Mistakes
- Using reserved words as variable names (`class`, `return`, `if`, `for`)
- Using non-descriptive names (`a`, `b`, `c`, `temp`, `data`)
- Inconsistent casing (`user_name` vs `userName` mixed)
- Starting variable names with digits
- Using hyphens in variable names (confused with subtraction)
- Using names that are too similar (`user1`, `user2`, `user3`)

### Best Practices
- Use descriptive, intention-revealing names
- Follow camelCase for variables and functions
- Use PascalCase for classes and constructor functions
- Use UPPER_SNAKE_CASE for true constants
- Avoid abbreviations except for universally known ones
- Use meaningful verb prefixes for functions
- Keep names concise but clear (balance between length and clarity)
- Use singular for single items, plural for collections

### Performance Considerations
- Variable name length has no impact on runtime performance
- Long names are minified during build (Webpack, Terser)
- Property name length affects object property lookup slightly, but negligibly
- Consistent naming reduces debugging time, indirectly improving productivity

### Interview Questions
1. What characters can JavaScript variable names start with?
2. What are reserved words in JavaScript?
3. What is the difference between camelCase and PascalCase?
4. Why is `_` sometimes used as a variable name prefix?
5. Can you use Unicode characters in variable names?

### Coding Challenges
1. Write a validator that checks if a string is a valid JavaScript identifier.
2. Refactor poorly named variables in a given code snippet.
3. Create a function that converts snake_case to camelCase.
4. Write a code review checklist for variable naming.

### Related Topics
- Reserved words in JavaScript
- Unicode identifiers
- Code style guides (Airbnb, Standard, Google)
- Linting rules (ESLint id-length, camelcase)
