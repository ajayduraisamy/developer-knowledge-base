# Functions - Function declarations, expressions, arrow functions, parameters

## Introduction
Functions are reusable blocks of code that perform specific tasks. They are first-class citizens in JavaScript, meaning they can be assigned to variables, passed as arguments, returned from other functions, and stored in data structures. This guide covers function declarations, expressions, arrow functions, and parameter handling.

## Function declarations

### What It Is
A function declaration defines a named function using the `function` keyword. It is hoisted, meaning it can be called before its definition in the enclosing scope.

### Why It Is Important
Function declarations provide clear, hoisted definitions that are ideal for module-level functions. They support the `this` binding, can be used as constructors, and have access to the `arguments` object.

### How It Works Internally
During the compilation phase, the engine hoists the function declaration (creates the function object and binds it to the name) before executing any code. This allows calling the function before its lexical position in the source.

### Syntax
```javascript
function functionName(parameter1, parameter2) {
  // Function body
  return value;
}

// Example
function greet(name) {
  return `Hello, ${name}!`;
}

// Calling
greet("Alice"); // "Hello, Alice!"
```

### Beginner Examples
```javascript
// Basic function declaration
function add(a, b) {
  return a + b;
}
console.log(add(3, 4)); // 7

// Function without return (returns undefined)
function logMessage(msg) {
  console.log(msg);
}
const result = logMessage("Hi"); // undefined

// Function with default parameter
function multiply(a, b = 1) {
  return a * b;
}
console.log(multiply(5));    // 5
console.log(multiply(5, 2)); // 10

// Multiple returns (early return pattern)
function divide(a, b) {
  if (b === 0) {
    return "Cannot divide by zero";
  }
  return a / b;
}
```

### Intermediate Examples
```javascript
// Function as a constructor
function Person(name, age) {
  this.name = name;
  this.age = age;
}
Person.prototype.sayHello = function() {
  return `Hi, I'm ${this.name}`;
};
const alice = new Person("Alice", 30);
console.log(alice.sayHello());

// arguments object
function sumAll() {
  let total = 0;
  for (let i = 0; i < arguments.length; i++) {
    total += arguments[i];
  }
  return total;
}
console.log(sumAll(1, 2, 3, 4)); // 10

// Recursion
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}
console.log(factorial(5)); // 120

// Nested function
function outer(x) {
  function inner(y) {
    return x + y;
  }
  return inner(10);
}
console.log(outer(5)); // 15
```

### Advanced Examples
```javascript
// Immediately Invoked Function Expression (IIFE)
(function() {
  const privateVar = "secret";
  console.log("IIFE executed");
})();

// Generator function
function* idGenerator() {
  let id = 1;
  while (true) {
    yield id++;
  }
}
const gen = idGenerator();
console.log(gen.next().value); // 1
console.log(gen.next().value); // 2

// Function with computed property (getter/setter pattern)
function createCounter() {
  let count = 0;
  return {
    increment() { count++; },
    decrement() { count--; },
    get value() { return count; }
  };
}
const counter = createCounter();
counter.increment();
counter.increment();
console.log(counter.value); // 2

// Memoization with function declaration
function fibonacci(n, memo = {}) {
  if (n in memo) return memo[n];
  if (n <= 1) return n;
  memo[n] = fibonacci(n - 1, memo) + fibonacci(n - 2, memo);
  return memo[n];
}
console.log(fibonacci(50)); // 12586269025
```

### Real-World Use Cases
- Module-level utility functions
- Constructors (pre-ES6 class pattern)
- Recursive algorithms (tree traversal, merge sort)
- Generator functions (infinite sequences, async control flow)
- IIFE for isolating scope (older pattern, module pattern)
- Event handlers that need `this` binding

### Common Mistakes
- Forgetting to return a value (returns undefined)
- Declaring functions inside loops (performance and closure issues)
- Using function declaration inside a block (inconsistent across engines)
- Not handling all code paths with return statements
- Confusing function declaration with function expression

### Best Practices
- Use function declarations for top-level functions
- Use descriptive verb-names (`getUser`, `calculateTotal`, `validateInput`)
- Keep functions small and single-purpose (Single Responsibility Principle)
- Use early returns to reduce nesting
- Prefer default parameters over manual `||` checks
- Use TypeScript or JSDoc for parameter/return type documentation

### Performance Considerations
- Function declarations are created once and reused
- Hoisting has no runtime cost
- Recursive functions can cause stack overflow for deep recursion
- Inlined functions can improve JIT optimization

### Interview Questions
1. What is hoisting and how does it affect function declarations?
2. What is the difference between a function declaration and a function expression?
3. How does the `arguments` object work?
4. What is an IIFE and why would you use it?
5. How does recursion work in JavaScript?

### Coding Challenges
1. Implement a recursive `flatten` function using a function declaration.
2. Write a generator function that yields a range of numbers.
3. Create a memoized version of a computationally expensive function.
4. Implement a function that counts the number of arguments passed.

### Related Topics
- Function expressions
- Arrow functions
- Hoisting
- Closures

## Function expressions

### What It Is
A function expression defines a function as part of an expression. It can be named or anonymous. Unlike declarations, function expressions are not hoisted.

### Why It Is Important
Function expressions enable functions as first-class citizens. They are used for callbacks, IIFEs, conditional function definitions, and when a function needs to be assigned to a variable or property.

### How It Works Internally
Function expressions are evaluated at runtime where they appear in the code. They are not hoisted, so they cannot be called before their definition. Named function expressions create a binding only within the function's scope (useful for recursion).

### Syntax
```javascript
// Anonymous function expression
const variable = function(parameters) {
  // body
};

// Named function expression
const variable = function functionName(parameters) {
  // body
};

// Example
const greet = function(name) {
  return `Hello, ${name}!`;
};
```

### Beginner Examples
```javascript
// Assigning to variable
const double = function(n) {
  return n * 2;
};
console.log(double(5)); // 10

// Pass as callback
const numbers = [1, 2, 3];
const doubled = numbers.map(function(n) {
  return n * 2;
});
console.log(doubled); // [2, 4, 6]

// IIFE (Immediately Invoked)
const result = (function(a, b) {
  return a + b;
})(3, 4);
console.log(result); // 7

// Conditional function assignment
let processData;
if (typeof window === "undefined") {
  processData = function(data) {
    return `Server: ${data}`;
  };
} else {
  processData = function(data) {
    return `Client: ${data}`;
  };
}
```

### Intermediate Examples
```javascript
// Named function expression (for recursion in callbacks)
const factorial = function fact(n) {
  if (n <= 1) return 1;
  return n * fact(n - 1);
};
console.log(factorial(5)); // 120

// Object method
const calculator = {
  add: function(a, b) { return a + b; },
  subtract: function(a, b) { return a - b; },
  multiply: function(a, b) { return a * b; }
};

// Array method callback
const numbers = [5, 3, 8, 1, 2];
numbers.sort(function(a, b) {
  return a - b;
});
console.log(numbers); // [1, 2, 3, 5, 8]

// Event listener (DOM)
// button.addEventListener('click', function(event) {
//   console.log('Button clicked', event);
// });
```

### Advanced Examples
```javascript
// Function expression returning a function
const createMultiplier = function(factor) {
  return function(number) {
    return number * factor;
  };
};
const double = createMultiplier(2);
const triple = createMultiplier(3);
console.log(double(5)); // 10
console.log(triple(5)); // 15

// Memoization with function expression
const fib = function fibMemo(n, memo = {}) {
  if (n in memo) return memo[n];
  if (n <= 1) return n;
  memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
  return memo[n];
};

// Immediately invoked method definition
const counter = (function() {
  let count = 0;
  return {
    increment: function() { count++; },
    getCount: function() { return count; }
  };
})();

// Function expression with bind
module.exports = function(options) {
  this.options = options;
  return this;
}.bind({});
```

### Real-World Use Cases
- Callbacks for async operations (setTimeout, fetch, promises)
- Array method callbacks (map, filter, reduce)
- Event handlers in browser/DOM
- Module pattern with IIFE
- Conditional function definitions based on environment
- Factory functions returning functions

### Common Mistakes
- Forgetting function expressions are not hoisted (calling before definition)
- Creating functions inside loops (creates new function each iteration)
- Not handling `this` binding correctly in function expressions
- Using anonymous functions in stack traces (harder to debug)
- Confusing function declaration with expression syntax

### Best Practices
- Use function expressions when assigning to a variable or property
- Name function expressions for better stack traces
- Use arrow functions for short callbacks (unless `this` binding is needed)
- Avoid creating function expressions inside loops when possible
- Use IIFE sparingly (module pattern is obsolete with ES modules)

### Performance Considerations
- Function expressions create a new function object each time evaluated
- Inside loops, function expressions create many function objects
- Modern engines optimize this well, but be mindful in hot paths
- Named function expressions don't add performance overhead

### Interview Questions
1. What is the difference between a function declaration and a function expression?
2. What is an IIFE and why would you use one?
3. How does hoisting apply to function expressions?
4. What is the purpose of named function expressions?
5. Can you call a function expression before it's defined?

### Coding Challenges
1. Write an IIFE that creates a counter module.
2. Create a function expression that implements currying.
3. Write a function that accepts a callback and calls it with specific arguments.
4. Create a factory function using function expressions.

### Related Topics
- Function declarations
- Arrow functions
- IIFE pattern
- Callbacks

## Arrow functions

### What It Is
Arrow functions (`=>`) are a concise syntax for writing function expressions, introduced in ES6. They have lexical `this` binding and cannot be used as constructors.

### Why It Is Important
Arrow functions provide concise syntax, especially for short callbacks. They solve the "lost `this`" problem common in traditional functions by capturing `this` from the enclosing lexical scope.

### How It Works Internally
Arrow functions do not have their own `this`, `arguments`, `super`, or `new.target`. They inherit these from the enclosing scope. They are always anonymous but can be assigned to variables. They cannot be used with `new` because they lack the `[[Construct]]` internal method.

### Syntax
```javascript
// Basic syntax
(param1, param2) => { /* body */ }
// Single parameter (no parentheses needed)
param => { /* body */ }
// Expression body (implicit return)
(param1, param2) => expression
// No parameters
() => { /* body */ }

// Examples
const add = (a, b) => a + b;
const square = x => x * x;
const greet = () => console.log("Hello");
```

### Beginner Examples
```javascript
// Concise body (implicit return)
const double = n => n * 2;
console.log(double(5)); // 10

// Block body (explicit return needed)
const sum = (a, b) => {
  const result = a + b;
  return result;
};

// Array callbacks
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);
const evens = numbers.filter(n => n % 2 === 0);
const sum2 = numbers.reduce((acc, n) => acc + n, 0);

// No parameters
const getRandom = () => Math.random();

// Returning an object literal
const createUser = (name, age) => ({ name, age, id: Date.now() });
```

### Intermediate Examples
```javascript
// Lexical this binding
function Timer() {
  this.seconds = 0;
  setInterval(() => {
    this.seconds++; // `this` refers to Timer instance
    console.log(this.seconds);
  }, 1000);
}
// new Timer();

// Arrow function with default parameters
const greet = (name = "Guest", greeting = "Hello") =>
  `${greeting}, ${name}!`;

// Rest parameters
const sumAll = (...numbers) => numbers.reduce((a, b) => a + b, 0);

// Arrow function as object method (cautious!)
const obj = {
  value: 42,
  getValue: () => obj.value, // Not recommended for methods
  getValueProper() {
    return this.value; // Method shorthand is better
  }
};

// Chaining with arrow functions
const result2 = [1, 2, 3, 4, 5]
  .filter(n => n % 2 === 0)
  .map(n => n * 10)
  .reduce((a, b) => a + b, 0);
```

### Advanced Examples
```javascript
// Arrow function in higher-order functions
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);

const add1 = x => x + 1;
const double2 = x => x * 2;
const add1ThenDouble = pipe(add1, double2);
console.log(add1ThenDouble(5)); // 12

// Arrow function for currying
const curry = fn => (...args) =>
  args.length >= fn.length
    ? fn(...args)
    : curry(fn.bind(null, ...args));

const add3 = (a, b, c) => a + b + c;
const curriedAdd = curry(add3);
console.log(curriedAdd(1)(2)(3)); // 6

// Arrow function with tagged template literal
const html = (strings, ...values) =>
  strings.reduce((acc, str, i) => acc + str + (values[i] || ""), "");

const name2 = "World";
const greeting2 = html`<h1>Hello, ${name2}!</h1>`;
console.log(greeting2); // <h1>Hello, World!</h1>

// Arrow function in async context
const fetchData = async () => {
  const response = await fetch('https://api.example.com/data');
  return response.json();
};
```

### Real-World Use Cases
- Array method callbacks (map, filter, reduce, forEach)
- Promise chains (then/catch callbacks)
- Event listeners (when `this` binding is not needed)
- React functional components
- Redux reducers and action creators
- Short utility functions
- Composing functions (functional programming)

### Common Mistakes
- Using arrow function for object methods (lexical `this` from outer scope)
- Using arrow function as constructor (throws TypeError)
- Using arrow function in dynamic `this` scenarios (event handlers needing `this`)
- Forgetting parentheses when returning object literal
- Assuming arrow functions have `arguments` object

### Best Practices
- Use arrow functions for short callbacks and utility functions
- Use method shorthand syntax for object methods
- Avoid arrow functions when you need dynamic `this`
- Use expression body for simple one-liners
- Use block body for multi-line logic
- Use arrow functions in React hooks and callbacks to avoid binding

### Performance Considerations
- Arrow functions have similar performance to anonymous function expressions
- No performance penalty for lexical `this` binding
- Slightly faster in some engines due to simpler internal structure
- Creating many arrow functions in hot paths can impact memory (same as any function)

### Interview Questions
1. How does `this` work differently in arrow functions?
2. Can you use arrow functions as constructors?
3. What is the syntax for returning an object literal from an arrow function?
4. How do arrow functions handle the `arguments` object?
5. When should you avoid arrow functions?

### Coding Challenges
1. Rewrite a set of traditional functions as arrow functions.
2. Create a compose function using arrow functions.
3. Fix a `this` binding issue by converting to arrow functions.
4. Write a currying function using arrow functions.

### Related Topics
- Function expressions
- this keyword
- Lexical scoping
- Functional programming

## Parameters and arguments

### What It Is
Parameters are the named variables listed in a function's definition. Arguments are the actual values passed when calling the function. JavaScript provides flexible parameter handling with default parameters, rest parameters, destructuring, and the `arguments` object.

### Why It Is Important
Understanding parameter handling is crucial for writing flexible, reusable functions. Default parameters prevent undefined errors, rest parameters handle variable-length inputs, and destructuring simplifies object/array argument handling.

### How It Works Internally
When a function is called, the engine creates a new execution context with parameter variables bound to the passed arguments. If fewer arguments are passed than parameters, remaining parameters are `undefined`. Default parameters are evaluated at call time (not definition time). Rest parameters collect remaining arguments into a real array.

### Syntax
```javascript
// Default parameters
function greet(name = "Guest") {
  return `Hello, ${name}`;
}

// Rest parameters
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

// Destructuring parameters
function processUser({ name, age, role = "user" }) {
  console.log(name, age, role);
}
```

### Beginner Examples
```javascript
// Basic parameters
function add(a, b) {
  return a + b;
}
console.log(add(3, 4)); // a=3, b=4
console.log(add(3));    // a=3, b=undefined → NaN

// Default parameters
function multiply(a, b = 1) {
  return a * b;
}
console.log(multiply(5));    // 5
console.log(multiply(5, 2)); // 10

// Default with previous parameter
function createUrl(base, path, protocol = "https") {
  return `${protocol}://${base}/${path}`;
}

// Undefined triggers default, null does not
function test(x = "default") {
  return x;
}
console.log(test(undefined)); // "default"
console.log(test(null));      // null
```

### Intermediate Examples
```javascript
// Rest parameters
function logAll(...args) {
  args.forEach(arg => console.log(arg));
}
logAll(1, 2, 3, "hello"); // 1, 2, 3, "hello"

// Rest with regular parameters
function introduce(greeting, ...names) {
  return names.map(name => `${greeting}, ${name}!`);
}
console.log(introduce("Hello", "Alice", "Bob", "Charlie"));

// arguments vs rest
function oldSchool() {
  // arguments is array-like, not a real array
  const args = Array.from(arguments);
  return args.join(", ");
}

// Destructuring objects in parameters
function configure({ host = "localhost", port = 3000, ssl = false } = {}) {
  console.log(host, port, ssl);
}
configure({ port: 8080 }); // "localhost", 8080, false
configure(); // "localhost", 3000, false (with empty default)

// Destructuring arrays in parameters
function firstAndRest([first, ...rest]) {
  return { first, rest };
}
console.log(firstAndRest([1, 2, 3, 4])); // { first: 1, rest: [2,3,4] }
```

### Advanced Examples
```javascript
// Parameter destructuring with computed keys
function renameKeys(obj, { renameMap = {}, prefix = "" } = {}) {
  const result = {};
  for (const [key, value] of Object.entries(obj)) {
    const newKey = renameMap[key] || `${prefix}${key}`;
    result[newKey] = value;
  }
  return result;
}

// Required parameter validation
function required(paramName) {
  throw new Error(`Parameter "${paramName}" is required`);
}

function createUser(
  name = required("name"),
  email = required("email"),
  role = "user"
) {
  return { name, email, role };
}

// Variadic function with typed rest
function combineArrays(...arrays) {
  return arrays.reduce((acc, arr) => {
    if (!Array.isArray(arr)) throw new TypeError("Expected array");
    return acc.concat(arr);
  }, []);
}

// Default parameters with destructuring and nested defaults
function fetchData({
  url,
  method = "GET",
  headers = {},
  timeout = 5000,
  retries = 3,
  onProgress = null
} = {}) {
  // Implementation
}
```

### Real-World Use Cases
- Configuration objects with defaults
- API functions with many options
- Event handlers with event objects
- Currying and partial application
- Higher-order functions with callbacks
- Variadic functions like `Math.max()`, `Object.assign()`
- Middleware patterns (Express/Koa)

### Common Mistakes
- Not handling missing arguments (leads to undefined errors)
- Using `arguments` object in arrow functions (not available)
- Mutating the `arguments` object (causes optimization deopt)
- Forgetting that default parameters are evaluated each call
- Destructuring `null` or `undefined` (throws TypeError)
- Confusing rest parameters with the `arguments` object

### Best Practices
- Use default parameters instead of `||` or `typeof` checks
- Use object destructuring for functions with many parameters
- Always provide a default empty object for destructured parameters: `({} = {})`
- Use rest parameters instead of `arguments` in modern code
- Keep the number of parameters small (<=3, or use config object)
- Document destructured parameters with JSDoc or TypeScript

### Performance Considerations
- Rest parameters create a new array each call (minor overhead)
- `arguments` object is slower than rest parameters
- Default parameters have negligible overhead
- Destructuring parameters has minimal performance impact
- Engines optimize parameter handling aggressively

### Interview Questions
1. What is the difference between parameters and arguments?
2. How do default parameters work?
3. What is the rest parameter syntax and how does it differ from `arguments`?
4. How does destructuring work in function parameters?
5. Can you use default parameters before required ones?

### Coding Challenges
1. Implement a function that accepts an object parameter with defaults.
2. Write a variadic `sum` function using rest parameters.
3. Create a function that validates required parameters.
4. Write a function using destructuring to extract specific properties from an object.
5. Implement a function that merges multiple objects using rest parameters.

### Related Topics
- Default parameters
- Rest parameters
- Destructuring assignment
- Spread operator
