# Spread and Rest - Spread operator (...), rest parameters, use cases

## Introduction
The spread operator and rest parameters both use the `...` syntax but serve different purposes. The spread operator expands iterables into individual elements. Rest parameters collect multiple elements into an array. Understanding both is essential for modern JavaScript.

## Spread operator (...)

### What It Is
The spread operator (`...`) allows an iterable (array, string, object) to be expanded into individual elements in contexts like function calls, array literals, and object literals.

### Why It Is Important
The spread operator provides a concise way to copy, merge, and manipulate arrays and objects. It replaces older methods like `concat`, `slice`, and `Object.assign` with more readable syntax.

### How It Works Internally
For arrays and iterables, the spread operator uses the `[Symbol.iterator]()` protocol to iterate over elements. For objects, it uses own enumerable property access. The spread operator creates a shallow copy.

### Syntax
```javascript
// Array spread
const newArray = [...iterable];

// Object spread
const newObj = { ...obj };

// Function call spread
func(...iterable);
```

### Beginner Examples
```javascript
// Array spread - copy
const original = [1, 2, 3];
const copy = [...original];
console.log(copy); // [1, 2, 3]
console.log(copy === original); // false (new array)

// Array spread - merge
const arr1 = [1, 2];
const arr2 = [3, 4];
const merged = [...arr1, ...arr2];
console.log(merged); // [1, 2, 3, 4]

// Array spread - insert
const midInsert = [0, ...arr1, 5, ...arr2];
console.log(midInsert); // [0, 1, 2, 5, 3, 4]

// Object spread - copy
const user = { name: "Alice", age: 30 };
const userCopy = { ...user };
console.log(userCopy); // { name: "Alice", age: 30 }

// Object spread - merge
const details = { email: "alice@example.com" };
const fullUser = { ...user, ...details };
console.log(fullUser); // { name: "Alice", age: 30, email: "alice@example.com" }

// Function call spread
const numbers = [5, 2, 9, 1, 7];
console.log(Math.max(...numbers)); // 9
console.log(Math.min(...numbers)); // 1
```

### Intermediate Examples
```javascript
// Override properties with spread
const defaults = { theme: "light", locale: "en", debug: false };
const config = { ...defaults, theme: "dark", debug: true };
console.log(config); // { theme: "dark", locale: "en", debug: true }

// Spread with string (iterable)
const chars = [..."hello"];
console.log(chars); // ["h", "e", "l", "l", "o"]

// Spread with Set
const set = new Set([1, 2, 2, 3, 3, 4]);
const unique = [...set];
console.log(unique); // [1, 2, 3, 4]

// Spread with Map
const map = new Map([["a", 1], ["b", 2]]);
const entries = [...map];
console.log(entries); // [["a", 1], ["b", 2]]

// Spread in array literal with conditionals
const items = [
  "always",
  ...(condition ? ["conditional"] : []),
  "always too"
];

// Spread with typed arrays
const uint8 = new Uint8Array([1, 2, 3]);
const regular = [...uint8];
console.log(regular); // [1, 2, 3]
```

### Advanced Examples
```javascript
// Deep clone limitation (shallow only)
const original2 = { a: 1, nested: { b: 2 } };
const shallow = { ...original2 };
shallow.nested.b = 99;
console.log(original2.nested.b); // 99 (shared reference!)

// Immutable update pattern
const state = { count: 0, items: [] };
const newState = { ...state, count: state.count + 1 };
console.log(newState); // { count: 1, items: [] }

// Spread with getters (evaluates them)
const objWithGetter = {
  _value: 10,
  get value() { return this._value * 2; }
};
const spreaded = { ...objWithGetter };
console.log(spreaded.value); // 20 (getter evaluated)

// Spread to remove properties (via destructuring)
const user3 = { name: "Alice", password: "secret", email: "alice@example.com" };
const { password, ...safeUser } = user3;
console.log(safeUser); // { name: "Alice", email: "alice@example.com" }

// Spread in React state (common pattern)
// this.setState(prevState => ({ ...prevState, count: prevState.count + 1 }));

// Spread with NodeList
const paragraphs = document.querySelectorAll("p");
const paragraphArray = [...paragraphs];
// paragraphArray.map(p => p.textContent);

// Multiple spreads with later overriding
const result3 = { a: 1, b: 2, ...{ b: 3, c: 4 }, ...{ c: 5 } };
console.log(result3); // { a: 1, b: 3, c: 5 }
```

### Real-World Use Cases
- Copying arrays and objects (shallow)
- Merging configuration objects
- Adding/removing items immutably
- Converting NodeList to Array
- Passing array elements as function arguments
- React/Redux immutable state updates
- Combining multiple data sources
- Removing properties from objects (with destructuring)

### Common Mistakes
- Assuming spread creates a deep copy
- Using spread on non-iterables with arrays (TypeError)
- Spreading undefined or null in object spread (works in modern JS)
- Forgetting that object spread only copies own enumerable properties
- Using spread with large arrays causing memory issues
- Expecting spread to preserve getter behavior

### Best Practices
- Use spread for shallow copies instead of `Object.assign`
- Use spread for merging objects instead of manual assignment
- Use spread for immutable updates to arrays/objects
- Use `[...iterable]` to convert iterables to arrays
- Use spread with destructuring to omit properties
- Be aware of shallow copy limitations
- Use spread over `apply` for function arguments

### Performance Considerations
- Spread creates a new array/object with O(n) time/space
- For large arrays, `concat` may be faster than spread
- Object spread iterates over own properties (fast)
- Multiple nested spreads in hot paths can slow down
- Spread in object literals is well-optimized by modern engines
- For immutable updates, spread is generally fast enough

### Interview Questions
1. What does the spread operator do?
2. Is the spread operator a deep or shallow copy?
3. What types can you spread into an array?
4. How do you use spread to remove a property from an object?
5. What is the difference between spread in arrays vs objects?

### Coding Challenges
1. Implement a function that merges multiple objects using spread.
2. Write a function that removes a property from an object immutably using spread.
3. Create an array deduplication function using Set and spread.
4. Implement a shallow array comparison function using spread.
5. Write a function that adds an item to an array at a specific position using spread.

### Related Topics
- Rest parameters
- Destructuring
- Immutability patterns

## Rest parameters

### What It Is
Rest parameters allow a function to accept an indefinite number of arguments as an array. The syntax uses `...` followed by the parameter name, and it must be the last parameter.

### Why It Is Important
Rest parameters replace the `arguments` object with a real array, enabling array methods and clearer function signatures. They are essential for variadic functions, variadic function composition, and wrapper functions.

### How It Works Internally
When a function with a rest parameter is called, any arguments beyond the named parameters are collected into a new array bound to the rest parameter. The rest parameter is always a real Array instance.

### Syntax
```javascript
function func(param1, param2, ...restParams) {
  // restParams is an array of remaining arguments
}

// Arrow functions support rest
const func = (...args) => args;
```

### Beginner Examples
```javascript
// Basic rest parameter
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}
console.log(sum(1, 2, 3, 4)); // 10
console.log(sum(5));           // 5
console.log(sum());            // 0

// Rest with named parameters
function log(level, ...messages) {
  console.log(`[${level}]:`, ...messages);
}
log("INFO", "Server started", "on port", 3000);
// [INFO]: Server started on port 3000

// Rest captures remaining arguments
function multiply(multiplier, ...numbers) {
  return numbers.map(n => n * multiplier);
}
console.log(multiply(2, 1, 2, 3)); // [2, 4, 6]

// Empty rest
function noExtra(first, ...rest) {
  console.log(rest.length); // 0 if no extra args
}
noExtra(1);
```

### Intermediate Examples
```javascript
// Rest with destructuring
function processConfig(baseURL, ...middlewares) {
  return middlewares.reduce((config, middleware) => {
    return { ...config, ...middleware(config) };
  }, { baseURL });
}

// Rest to collect specific arguments
function formatString(format, ...args) {
  return format.replace(/%[sd]/g, () => args.shift() || "");
}
console.log(formatString("Hello %s, you have %d messages", "Alice", 5));

// Rest with default parameters
function createUser(name, ...roles) {
  return {
    name,
    roles: roles.length ? roles : ["user"],
    createdAt: new Date()
  };
}
console.log(createUser("Alice"));       // { name: "Alice", roles: ["user"] }
console.log(createUser("Bob", "admin", "editor")); // { name: "Bob", roles: ["admin", "editor"] }

// Rest in arrow functions
const average = (...numbers) => {
  if (numbers.length === 0) return 0;
  return numbers.reduce((a, b) => a + b) / numbers.length;
};
console.log(average(1, 2, 3, 4, 5)); // 3

// Rest with constructor
class EventEmitter {
  on(event, ...handlers) {
    this._handlers = this._handlers || {};
    this._handlers[event] = [...(this._handlers[event] || []), ...handlers];
  }
}
```

### Advanced Examples
```javascript
// Rest parameters for variadic currying
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn(...args);
    }
    return (...moreArgs) => curried(...args, ...moreArgs);
  };
}
const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);
console.log(curriedAdd(1)(2)(3)); // 6

// Rest for function composition
const compose = (...fns) => (x) => fns.reduceRight((v, f) => f(v), x);
const pipe = (...fns) => (x) => fns.reduce((v, f) => f(v), x);

const add1 = x => x + 1;
const double2 = x => x * 2;
const add1ThenDouble = pipe(add1, double2);
console.log(add1ThenDouble(5)); // 12

// Rest for tap/debug
function tap(...args) {
  console.log(...args);
  return args.length === 1 ? args[0] : args;
}

// Rest in setter
class Stack {
  push(...items) {
    this._items = [...(this._items || []), ...items];
    return this._items.length;
  }
  pop() {
    return this._items?.pop();
  }
}

// Rest parameters with method borrowing
function joinWith(separator, ...strings) {
  return strings.join(separator);
}
console.log(joinWith(" - ", "a", "b", "c")); // "a - b - c"
```

### Real-World Use Cases
- Math utilities (sum, average, max, min)
- Logging functions with variable arguments
- Event systems with multiple handlers
- Middleware patterns (Express, Koa, Redux)
- Function composition utilities
- Currying and partial application
- Wrapper/decorator functions
- Format functions (printf-style)

### Common Mistakes
- Using rest parameter in the middle of parameters (SyntaxError)
- Confusing rest parameters with the `arguments` object
- Expecting rest to work in setter functions (not supported)
- Using rest parameter with default values (rest cannot have defaults)
- Forgetting that rest creates a new array each call

### Best Practices
- Use rest parameters over `arguments` object in modern code
- Rest parameter should be the last parameter
- Use descriptive names for rest parameters (`...args`, `...items`)
- Use rest for variadic functions (unknown number of arguments)
- Use rest with spread for forwarding arguments
- Keep functions with rest parameters focused
- Document the expected structure of rest arguments

### Performance Considerations
- Rest parameters create a new array each call (minor overhead)
- For performance-critical code, consider named parameters
- JIT engines optimize rest parameter allocation
- Rest is slightly faster than `arguments` object
- Avoid deep copying rest parameters unnecessarily

### Interview Questions
1. What is the difference between rest parameters and the `arguments` object?
2. Can a rest parameter have a default value?
3. Can there be multiple rest parameters in a function?
4. How do rest parameters work with arrow functions?
5. What is the difference between rest and spread (same syntax, different context)?

### Coding Challenges
1. Implement a function that accepts any number of arguments and returns their product.
2. Create a logging function that prefixes a log level and accepts multiple messages.
3. Write a pipeline/compose function using rest parameters.
4. Implement a function that merges multiple arrays using rest.
5. Create a curried function using rest parameters.

### Related Topics
- Spread operator
- Arguments object
- Default parameters
- Function composition

## Combining spread and rest

### What It Is
Spread and rest can be combined to create powerful patterns for data transformation, function composition, and argument forwarding. They are complementary sides of the same syntax.

### Why It Is Important
Combining spread and rest enables elegant solutions for common patterns: forwarding arguments, creating flexible APIs, and transforming data structures concisely.

### How It Works Internally
When used together, rest collects elements into an array, and spread expands them. The engine handles both through the iterator protocol and array/object property enumeration.

### Syntax
```javascript
// Forwarding arguments
function wrapper(...args) {
  return originalFunc(...args);
}

// Partial application
function partial(fn, ...preset) {
  return (...later) => fn(...preset, ...later);
}

// Array manipulation
const [first, ...rest] = array;
const newArray = [first, ...rest];
```

### Beginner Examples
```javascript
// Argument forwarding
function logger(fn) {
  return (...args) => {
    console.log(`Calling with:`, ...args);
    return fn(...args);
  };
}
const wrappedSum = logger((a, b) => a + b);
console.log(wrappedSum(3, 4)); // Logs: Calling with: 3 4 \n 7

// Gather first, spread rest
const [head, ...tail] = [1, 2, 3, 4];
const reconstituted = [head, ...tail];
console.log(reconstituted); // [1, 2, 3, 4]

// Partial application
function partial(fn, ...presetArgs) {
  return function(...laterArgs) {
    return fn(...presetArgs, ...laterArgs);
  };
}
const add = (a, b, c) => a + b + c;
const add5 = partial(add, 5);
console.log(add5(3, 2)); // 10
```

### Intermediate Examples
```javascript
// Middleware pattern
function createMiddleware(...middlewares) {
  return (handler) => {
    return (...args) => {
      const chain = middlewares.reduceRight((next, middleware) => {
        return middleware(next);
      }, handler);
      return chain(...args);
    };
  };
}

// Variadic merge with rest and spread
function mergeArrays(...arrays) {
  return arrays.reduce((acc, arr) => [...acc, ...arr], []);
}
console.log(mergeArrays([1, 2], [3, 4], [5, 6])); // [1, 2, 3, 4, 5, 6]

// Object transformation
function pick(obj, ...keys) {
  return keys.reduce((acc, key) => {
    if (key in obj) acc[key] = obj[key];
    return acc;
  }, {});
}
function omit(obj, ...keys) {
  const keysSet = new Set(keys);
  return Object.fromEntries(
    Object.entries(obj).filter(([key]) => !keysSet.has(key))
  );
}

// Fluent API
class QueryBuilder {
  constructor(...conditions) {
    this.conditions = conditions;
  }
  where(...clauses) {
    return new QueryBuilder(...this.conditions, ...clauses);
  }
  build() {
    return this.conditions.join(" AND ");
  }
}
```

### Advanced Examples
```javascript
// Decorator pattern
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

function throttle(fn, interval) {
  let lastTime = 0;
  return (...args) => {
    const now = Date.now();
    if (now - lastTime >= interval) {
      lastTime = now;
      fn(...args);
    }
  };
}

// Memoization with rest
function memoize(fn) {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

// Redux connect-like selector
function createSelector(...fns) {
  const dependencies = fns.slice(0, -1);
  const compute = fns[fns.length - 1];
  let lastArgs = null;
  let lastResult;
  
  return (...args) => {
    const depValues = dependencies.map(fn => fn(...args));
    if (lastArgs && depValues.every((v, i) => v === lastArgs[i])) {
      return lastResult;
    }
    lastArgs = depValues;
    lastResult = compute(...depValues);
    return lastResult;
  };
}

// Fluent builder with spread/rest
class URLBuilder {
  constructor(base = "") {
    this.base = base;
    this.params = {};
  }
  addParams(entries) {
    this.params = { ...this.params, ...entries };
    return this;
  }
  build() {
    const query = Object.entries(this.params)
      .map(([k, v]) => `${k}=${encodeURIComponent(v)}`)
      .join("&");
    return query ? `${this.base}?${query}` : this.base;
  }
}
```

### Real-World Use Cases
- Function decorators (debounce, throttle, memoize)
- API client wrappers
- Logging/telemetry wrappers
- Data transformation pipelines
- Configuration merging
- State management (Redux reducers)
- Builder pattern implementations
- Middleware stacks (Express, Redux)

### Common Mistakes
- Confusing which side is rest vs spread (context determines)
- Using rest in object destructuring with spread (both possible)
- Creating deeply nested spread/rest chains (readability issues)
- Not handling empty rest arrays in composition
- Performance overhead of repeatedly spreading in hot paths

### Best Practices
- Use rest to collect arguments, spread to expand them
- Combine for clean argument forwarding
- Use for decorator/wrapper patterns
- Keep composition chains reasonable in length
- Use rest in destructuring to collect remaining properties
- Use spread to merge and override
- Prefer readability over clever combinations

### Performance Considerations
- Each spread creates a new collection
- Composition chains should be mindful of allocation
- `...args` in forwarding is optimized by JIT
- Deeply nested rest/spread combinations may cause GC pressure
- For hot paths, benchmark and consider manual alternatives

### Interview Questions
1. How do you forward all arguments from one function to another?
2. How do you implement a partial application function using rest and spread?
3. What is the difference between `...args` in a function definition vs function call?
4. How do you combine rest parameters with destructuring?
5. How would you implement a compose function using rest and spread?

### Coding Challenges
1. Implement a `debounce` function that forwards arguments.
2. Create a `compose` function using rest parameters and spread.
3. Write a function that merges multiple objects and arrays using spread.
4. Implement a simple Redux-like `combineReducers` using rest/spread.
5. Create a validator function that accepts multiple validation functions and forwards data.

### Related Topics
- Rest parameters
- Spread operator
- Destructuring
- Function composition
