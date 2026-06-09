# Functional Programming - Pure functions, immutability, currying, composition, monads

## Introduction

Functional programming (FP) is a programming paradigm that treats computation as the evaluation of mathematical functions, avoiding changing state and mutable data. JavaScript supports functional programming through first-class functions, closures, array methods (`map`, `filter`, `reduce`), and newer features like arrow functions. While JavaScript is not a pure functional language (like Haskell), it enables functional patterns that lead to more predictable, testable, and maintainable code.

## Pure Functions

### What It Is

A pure function is a function that, given the same input, always returns the same output and has no side effects. It does not modify any external state, does not perform I/O, does not read external variables (except constants), and does not mutate its arguments.

### Why It Is Important

Pure functions are predictable, testable, and easy to reason about. They don't depend on or modify external state, eliminating entire categories of bugs. In JavaScript, pure functions enable techniques like memoization, lazy evaluation, and parallel execution.

### How It Works Internally

A pure function operates only on its input parameters and returns a value. It doesn't write to the global scope, modify arguments, or perform observable operations like logging. The JavaScript engine can optimize pure functions differently (e.g., constant folding, elimination of unused results).

```javascript
// Pure
function add(a, b) {
  return a + b;
}

// Impure (reads external state)
let taxRate = 0.1;
function calculatePrice(amount) {
  return amount * (1 + taxRate); // Depends on external taxRate
}

// Impure (modifies external state)
let total = 0;
function addToTotal(value) {
  total += value; // Modifies external variable
  return total;
}

// Impure (side effect)
function logAdd(a, b) {
  console.log(`Adding ${a} + ${b}`); // Side effect
  return a + b;
}
```

### Syntax

```javascript
// Pure function examples
const toUpper = (str) => str.toUpperCase();
const reverse = (arr) => [...arr].reverse();
const increment = (x) => x + 1;

// Impure function examples (avoid)
const getRandom = () => Math.random();        // Not deterministic
const getTime = () => Date.now();              // Not deterministic
const pushToArray = (arr, val) => arr.push(val); // Mutates input

// Pure version
const appendToArray = (arr, val) => [...arr, val];

// Refactoring impure to pure
// Impure:
let items = [];
function addItem(item) {
  items.push(item);
  return items;
}

// Pure:
function addItemPure(items, item) {
  return [...items, item];
}
```

## Immutability

### What It Is

Immutability means data cannot be changed after creation. Instead of modifying existing data, you create new data with the desired changes. In JavaScript, primitives are immutable by nature, but objects and arrays are mutable by default. Enforcing immutability requires discipline or libraries (Immer, Immutable.js).

### Why It Is Important

Immutability prevents accidental mutations that cause bugs, especially in concurrent code and state management (Redux requires immutability). It enables referential transparency (same input → same output), simplifies change detection (for UI updates), and makes code easier to reason about.

### How It Works Internally

Non-destructive operations always return new objects/arrays. The spread operator (`...`) and methods like `map`, `filter`, `reduce` create new data. For large data structures, structural sharing (used by Immutable.js) shares unchanged portions between old and new versions.

```javascript
// Mutable (antipattern)
const user = { name: 'Alice', age: 30 };
user.age = 31; // Mutation!

// Immutable pattern
const updatedUser = { ...user, age: 31 };
// user is unchanged, updatedUser is new
```

### Syntax

```javascript
// Object immutability
const user = { name: 'Alice', address: { city: 'NYC' } };

// Shallow copy (spread)
const updated = { ...user, name: 'Bob' };

// Deep copy (for nested objects)
const deepUpdated = JSON.parse(JSON.stringify(user));
// Or structuredClone:
const deepUpdated2 = structuredClone(user);

// Object.assign
const updated3 = Object.assign({}, user, { name: 'Bob' });

// Array immutability
const arr = [1, 2, 3];
const added = [...arr, 4];         // [1,2,3,4]
const removed = arr.filter(x => x !== 2); // [1,3]
const mapped = arr.map(x => x * 2);       // [2,4,6]
const replaced = arr.with(1, 99);         // [1,99,3] (ES2023)
const sorted = [...arr].sort();           // Copy then sort

// Freeze (shallow)
const frozen = Object.freeze({ name: 'Alice' });
// frozen.name = 'Bob'; // TypeError (strict mode) or silently fails
```

### Beginner Examples

```javascript
// Mutable vs immutable practice

// Mutable array methods (mutate in place):
arr.push(4);     // Mutates
arr.pop();       // Mutates
arr.splice(1, 1); // Mutates
arr.sort();      // Mutates
arr.reverse();   // Mutates

// Immutable alternatives:
[...arr, 4];                    // Add
[...arr.slice(0, -1)];          // Pop
arr.filter((_, i) => i !== 1);  // Remove
[...arr].sort();                // Sort
[...arr].reverse();             // Reverse

// Mutable object:
function updateName(user, newName) {
  user.name = newName; // Mutation!
  return user;
}

// Immutable:
function updateNameImmutable(user, newName) {
  return { ...user, name: newName };
}

// React/Redux requires immutability
// Reducer example:
function userReducer(state, action) {
  switch (action.type) {
    case 'UPDATE_NAME':
      return { ...state, name: action.payload };
    case 'UPDATE_EMAIL':
      return { ...state, email: action.payload };
    default:
      return state;
  }
}
```

## Currying and Partial Application

### What It Is

Currying transforms a function that takes multiple arguments into a sequence of functions that each take a single argument. `add(a, b, c)` becomes `add(a)(b)(c)`. Partial application is similar but transforms a function by pre-filling some arguments, producing a function with fewer parameters.

### Why It Is Important

Currying and partial application enable function composition, code reuse, and the creation of specialized functions from general ones. They are fundamental to functional programming and are used extensively in libraries like Lodash, Ramda, and Redux.

### How It Works Internally

Currying works through closures. Each call returns a new function that captures the previously provided arguments and waits for the remaining ones. When enough arguments are supplied, the original function is called.

### Syntax

```javascript
// Manual currying
function add(a) {
  return function(b) {
    return a + b;
  };
}
const add5 = add(5);
console.log(add5(3)); // 8
console.log(add(5)(3)); // 8

// Arrow function currying
const add = a => b => a + b;

// General curry function
function curry(fn) {
  const arity = fn.length;
  return function curried(...args) {
    if (args.length >= arity) {
      return fn(...args);
    }
    return (...moreArgs) => curried(...args, ...moreArgs);
  };
}

const curriedAdd = curry((a, b, c) => a + b + c);
console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
console.log(curriedAdd(1, 2, 3)); // 6
```

### Beginner Examples

```javascript
// Partial application with bind
function multiply(a, b) { return a * b; }
const double = multiply.bind(null, 2);
console.log(double(5)); // 10

// Partial application function
function partial(fn, ...args) {
  return (...moreArgs) => fn(...args, ...moreArgs);
}

const add = (a, b, c) => a + b + c;
const add5 = partial(add, 5);
console.log(add5(10, 20)); // 5 + 10 + 20 = 35

// Lodash's curry
const _ = require('lodash');
const curried = _.curry((a, b, c) => a + b + c);
console.log(curried(1)(2)(3)); // 6

// Real-world: formatting functions
const format = (locale, options, date) => 
  new Intl.DateTimeFormat(locale, options).format(date);

const formatUS = partial(format, 'en-US');
const formatShortUS = partial(formatUS, { dateStyle: 'short' });
console.log(formatShortUS(new Date())); // "1/1/24"
```

### Intermediate Examples

```javascript
// Currying for configuration
const fetchWithOptions = (baseUrl) => (endpoint) => (params) =>
  fetch(`${baseUrl}${endpoint}?${new URLSearchParams(params)}`);

const apiV1 = fetchWithOptions('https://api.example.com/v1');
const getUsers = apiV1('/users');
const getPosts = apiV1('/posts');

const activeUsers = await getUsers({ status: 'active' });

// Curried validation functions
const validate = (predicate) => (message) => (value) =>
  predicate(value) ? null : message;

const isEmail = (v) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v);
const isRequired = (v) => v != null && v !== '';

const emailValidator = validate(isEmail)('Invalid email');
const requiredValidator = validate(isRequired)('This field is required');

console.log(emailValidator('test@example.com')); // null
console.log(emailValidator('invalid')); // 'Invalid email'

// Tracing/Logging curried function
const trace = (label) => (value) => {
  console.log(`${label}: ${value}`);
  return value;
};

const double = x => x * 2;
const triple = x => x * 3;

const result = pipe(
  double,
  trace('after double'),
  triple,
  trace('after triple')
)(5);
```

## Function Composition

### What It Is

Function composition is the process of combining two or more functions to produce a new function. The output of one function becomes the input of the next. Compose (right-to-left) and pipe (left-to-right) are the two common composition patterns.

### Why It Is Important

Composition is the fundamental pattern for building complex operations from simple, reusable functions. It enables a declarative programming style where you describe _what_ to do rather than _how_ to do it. Composition is more modular and composable than method chaining.

### How It Works Internally

Compose/pipe creates a closure that calls each function in sequence, passing the result of each as the argument to the next. The implementation is typically a `reduce` operation over the function array.

```javascript
// Compose: right-to-left
const compose = (...fns) => (x) => fns.reduceRight((acc, fn) => fn(acc), x);

// Pipe: left-to-right (more common in JS)
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);
```

### Syntax

```javascript
// Manual composition
const trim = (s) => s.trim();
const toLower = (s) => s.toLowerCase();
const capitalize = (s) => s[0].toUpperCase() + s.slice(1);

const formatName = (name) => capitalize(toLower(trim(name)));
console.log(formatName('  ALICE  ')); // 'Alice'

// Using pipe
const formatName2 = pipe(trim, toLower, capitalize);
console.log(formatName2('  ALICE  ')); // 'Alice'

// Using compose (reverse order)
const formatName3 = compose(capitalize, toLower, trim);
console.log(formatName3('  ALICE  ')); // 'Alice'

// Multiple arguments
const pipeMulti = (...fns) => fns.reduce((f, g) => (...args) => g(f(...args)));

const add = (a, b) => a + b;
const double = (x) => x * 2;

const addAndDouble = pipe(add, double);
console.log(addAndDouble(3, 5)); // 16
```

### Beginner Examples

```javascript
// Processing pipelines
const users = [
  { name: 'Alice', age: 25, active: true },
  { name: 'Bob', age: 17, active: true },
  { name: 'Charlie', age: 30, active: false },
  { name: 'Diana', age: 22, active: true }
];

// Without composition (nested, hard to read)
const result1 = users
  .filter(u => u.active && u.age >= 18)
  .map(u => u.name.toUpperCase())
  .sort();

// With composition
const filterActive = (users) => users.filter(u => u.active);
const filterAdults = (users) => users.filter(u => u.age >= 18);
const getNames = (users) => users.map(u => u.name);
const toUpper = (str) => typeof str === 'string' ? str.toUpperCase() : str.map(toUpper);
const uppercase = (items) => items.map(toUpper);
const sort = (items) => [...items].sort();

const getActiveAdultNames = pipe(
  filterActive,
  filterAdults,
  getNames,
  uppercase,
  sort
);

console.log(getActiveAdultNames(users)); // ['ALICE', 'DIANA']

// Composition with data transformation
const processOrder = pipe(
  validateOrder,
  calculateTotal,
  applyDiscount,
  generateInvoice,
  sendConfirmation
);
```

## Functors and Monads

### What It Is

A Functor is a data type that implements a `map` method that applies a function to the value(s) inside the functor while preserving the structure. A Monad is a type that implements `flatMap` (or `chain`, `bind`) and `of` (or `return`) methods, enabling sequential composition of computations with context.

### Why It Is Important

Functors and monads provide a principled way to handle computations with context: optional values (Maybe), errors (Either), async (Promise), or state. Monads are used in functional programming to sequence operations while managing side effects. Promise is a monad for async computations. Optional chaining (`?.`) is a practical application of the Maybe monad.

### How It Works Internally

**Functor**: `F.map(fn)` returns a new F with the function applied. Array's `.map()` is a functor operation: `[1,2,3].map(x => x * 2)` returns a new array preserving the array structure.

**Monad**: `M.flatMap(fn)` applies a function that returns a monad, then flattens the result. Promise's `.then()` is a monadic operation because `Promise.resolve(1).then(x => Promise.resolve(x * 2))` returns `Promise<2>`, not `Promise<Promise<2>>`.

### Syntax

```javascript
// Array as Functor
const arr = [1, 2, 3];
const doubled = arr.map(x => x * 2); // [2, 4, 6] (structure preserved)

// Promise as Monad
Promise.resolve(5)
  .then(x => Promise.resolve(x * 2)) // Returns Promise<10>, not Promise<Promise<10>>
  .then(console.log); // 10

// Optional/Maybe pattern
function findUser(id) { /* maybe returns user or null */ }

// Without Maybe:
const user = findUser(1);
const city = user ? user.address?.city : undefined;
// Still ugly with nested ternaries

// Simple Maybe implementation
class Maybe {
  constructor(value) {
    this._value = value;
  }
  
  static of(value) {
    return new Maybe(value);
  }
  
  map(fn) {
    return this._value == null 
      ? Maybe.of(null) 
      : Maybe.of(fn(this._value));
  }
  
  flatMap(fn) {
    return this._value == null 
      ? Maybe.of(null) 
      : fn(this._value);
  }
  
  getOrElse(defaultValue) {
    return this._value != null ? this._value : defaultValue;
  }
}

// Usage
const city = Maybe.of(findUser(1))
  .map(user => user.address)
  .map(address => address.city)
  .getOrElse('Unknown');
```

### Beginner Examples

```javascript
// Reality in JavaScript: Promise as monad
// Promise satisfies monad laws:
// of: Promise.resolve
// flatMap: .then (when callback returns Promise)

// Left identity: of(x).flatMap(f) === f(x)
Promise.resolve(5).then(x => Promise.resolve(x * 2)) // Promise<10>
// vs
((x) => Promise.resolve(x * 2))(5); // Same result

// Right identity: m.flatMap(of) === m
Promise.resolve(5).then(Promise.resolve); // Promise<5>

// Associativity: m.flatMap(f).flatMap(g) === m.flatMap(x => f(x).flatMap(g))
// Promise chaining satisfies this

// Array as monad
// of: Array.of
// flatMap: Array.flatMap (or arr.map(...).flat())

Array.prototype.flatMap = function(fn) {
  return this.reduce((acc, x) => acc.concat(fn(x)), []);
};

const result = [1, 2, 3]
  .flatMap(x => [x, -x]); // [1, -1, 2, -2, 3, -3]

// Nested arrays flattened automatically
const nested = [[1], [2, 3], [4]];
console.log(nested.flatMap(x => x)); // [1, 2, 3, 4]
```

### Intermediate Examples

```javascript
// Either monad for error handling
class Either {
  constructor(value) {
    this._value = value;
  }
  
  static Right(value) {
    return new Right(value);
  }
  
  static Left(value) {
    return new Left(value);
  }
  
  static tryCatch(fn) {
    try {
      return Either.Right(fn());
    } catch (e) {
      return Either.Left(e);
    }
  }
  
  isRight() { return false; }
  isLeft() { return false; }
}

class Right extends Either {
  map(fn) { return Either.Right(fn(this._value)); }
  flatMap(fn) { return fn(this._value); }
  getOrElse(_) { return this._value; }
  get() { return this._value; }
}

class Left extends Either {
  map(_) { return this; }
  flatMap(_) { return this; }
  getOrElse(defaultValue) { return defaultValue; }
  get() { throw new Error('Cannot get Left value'); }
}

// Usage
function parseJson(str) {
  try {
    return Either.Right(JSON.parse(str));
  } catch (e) {
    return Either.Left(e);
  }
}

function getAge(json) {
  const age = json.age;
  return age != null ? Either.Right(age) : Either.Left(new Error('No age'));
}

const result = parseJson('{"name":"Alice","age":30}')
  .flatMap(getAge)
  .map(age => age + 1)
  .getOrElse(0);

console.log(result); // 31

// IO Monad (lazy computation)
class IO {
  constructor(effect) {
    this.effect = effect;
  }
  
  static of(value) {
    return new IO(() => value);
  }
  
  map(fn) {
    return new IO(() => fn(this.effect()));
  }
  
  flatMap(fn) {
    return new IO(() => fn(this.effect()).run());
  }
  
  run() {
    return this.effect();
  }
}

// Usage
const readLine = new IO(() => prompt('Enter name:'));
const writeLine = (msg) => new IO(() => console.log(msg));

const program = readLine
  .map(name => `Hello, ${name}!`)
  .flatMap(writeLine);

// Nothing executes until .run()
// program.run(); // Only now does the side effect occur
```

### Advanced Examples

```javascript
// Task monad (like Promise but lazy and pure)
class Task {
  constructor(fork) {
    this.fork = fork;
  }
  
  static of(value) {
    return new Task((reject, resolve) => resolve(value));
  }
  
  static rejected(error) {
    return new Task((reject) => reject(error));
  }
  
  map(fn) {
    return new Task((reject, resolve) => 
      this.fork(reject, (value) => resolve(fn(value)))
    );
  }
  
  flatMap(fn) {
    return new Task((reject, resolve) =>
      this.fork(reject, (value) => fn(value).fork(reject, resolve))
    );
  }
  
  mapRejection(fn) {
    return new Task((reject, resolve) =>
      this.fork((error) => resolve(fn(error)), resolve)
    );
  }
}

// Usage
const fetchUser = (id) => new Task((reject, resolve) => {
  fetch(`/api/users/${id}`)
    .then(r => r.json())
    .then(resolve)
    .catch(reject);
});

const processUser = fetchUser(1)
  .map(user => ({ ...user, processed: true }))
  .mapRejection(err => ({ error: err.message }));

// Nothing executes until .fork is called
// processUser.fork(console.error, console.log);

// Monad laws (simple validation)
class Identity {
  constructor(value) { this.value = value; }
  static of(value) { return new Identity(value); }
  map(fn) { return Identity.of(fn(this.value)); }
  flatMap(fn) { return fn(this.value); }
}

// Laws:
// 1. Left identity: of(x).flatMap(f) = f(x)
Identity.of(5).flatMap(x => Identity.of(x * 2)); // Identity(10)
((x) => Identity.of(x * 2))(5); // Identity(10)

// 2. Right identity: m.flatMap(of) = m
Identity.of(5).flatMap(Identity.of); // Identity(5)

// 3. Associativity: m.flatMap(f).flatMap(g) = m.flatMap(x => f(x).flatMap(g))
const f = x => Identity.of(x + 1);
const g = x => Identity.of(x * 2);
Identity.of(5).flatMap(f).flatMap(g); // Identity(12)
Identity.of(5).flatMap(x => f(x).flatMap(g)); // Identity(12)
```

### Real-World Use Cases

- Redux reducers (pure functions + immutability)
- RxJS observables (functor/monad patterns)
- React functional components (composable, pure)
- Lodash/fp (functional utilities)
- Sanctuary, Folktale (functional libraries)
- Express middleware (composition)
- Promises (monad for async)

### Common Mistakes

- Mutating function arguments (side effects)
- Confusing currying with partial application
- Over-engineering with monads in simple cases
- Not handling null/undefined in functional pipelines
- Combining too many functions in a pipe (readability suffers)
- Performance concerns with deep recursion (functional recursion)

### Best Practices

- Use pure functions whenever possible
- Keep functions small and single-purpose
- Prefer `map`, `filter`, `reduce` over imperative loops
- Use `pipe`/`compose` for building processing pipelines
- Start simple (just use pure functions + composition)
- Use libraries (Ramda, Lodash/fp) for advanced FP patterns
- Avoid deep recursion; use Tail Call Optimization or iteration
- Use TypeScript for better functional type safety

### Performance Considerations

- Immutable operations create new objects (GC pressure)
- Spread operator on large objects/arrays is O(n)
- Immutable.js uses structural sharing for performance
- Currying adds closure overhead per call
- Lodash/fp functions have overhead but are well-optimized
- Pure functions enable compiler optimizations (V8)
- Memoization with pure functions can dramatically improve perf

### Interview Questions

**Q: What are the benefits of pure functions?** A: Pure functions are deterministic (same input = same output), have no side effects, are easy to test (no mocking needed), are easier to reason about, support memoization, enable referential transparency, and can be parallelized safely.

**Q: Explain the difference between currying and partial application.** A: Currying transforms a function taking n arguments into n nested functions each taking 1 argument: `f(a,b,c) → f(a)(b)(c)`. Partial application pre-fills some arguments, returning a function with fewer parameters. Currying always produces unary functions; partial application can produce functions with any remaining arity.


### Coding Challenges
```javascript
// Challenge 1: Implement compose and pipe
const compose = (...fns) => x => fns.reduceRight((acc, fn) => fn(acc), x);
const pipe = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);

const add1 = x => x + 1;
const double = x => x * 2;
const add1ThenDouble = pipe(add1, double);
console.log(add1ThenDouble(5)); // 12

// Challenge 2: Implement curry
function curry(fn) {
  return function curried(...args) {
    return args.length >= fn.length
      ? fn.apply(this, args)
      : (...more) => curried(...args, ...more);
  };
}
const add = curry((a, b, c) => a + b + c);
console.log(add(1)(2)(3)); // 6

// Challenge 3: Implement a simple Maybe monad
class Maybe {
  constructor(value) { this._value = value; }
  static of(value) { return new Maybe(value); }
  map(fn) { return this._value == null ? Maybe.of(null) : Maybe.of(fn(this._value)); }
  getOrElse(defaultVal) { return this._value ?? defaultVal; }
}
const result = Maybe.of(5).map(x => x * 2).map(x => x + 1).getOrElse(0);
console.log(result); // 11
```

### Related Topics

- Higher-order functions
- Function composition patterns
- Immutable data structures
- Monad transformers
- Reactive programming (RxJS)
- Category theory fundamentals
- Algebraic data types
- Lazy evaluation
- Referential transparency
- Side effect management
