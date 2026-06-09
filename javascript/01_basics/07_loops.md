# Loops - for, while, do-while, for...in, for...of, break/continue

## Introduction
Loops allow repetitive execution of code blocks. JavaScript provides several loop constructs: `for`, `while`, `do-while`, `for...in`, and `for...of`. Each serves different use cases and iteration patterns.

## for loop

### What It Is
The `for` loop is the most traditional loop construct. It repeats a block of code while a specified condition is true, with an initialization step and an iteration update.

### Why It Is Important
The `for` loop provides fine-grained control over iteration with initialization, condition checking, and increment in one line. It is ideal for known iteration counts and index-based array traversal.

### How It Works Internally
The `for` loop executes in four phases: initialization (runs once), condition check (before each iteration), body execution, and iteration update (after each iteration). The engine evaluates the condition before each iteration; if falsy, the loop terminates.

### Syntax
```javascript
for (initialization; condition; iteration) {
  // Loop body
}

// Common pattern
for (let i = 0; i < array.length; i++) {
  console.log(array[i]);
}
```

### Beginner Examples
```javascript
// Basic count
for (let i = 0; i < 5; i++) {
  console.log(i); // 0, 1, 2, 3, 4
}

// Iterating an array
const fruits = ["apple", "banana", "cherry"];
for (let i = 0; i < fruits.length; i++) {
  console.log(fruits[i]);
}

// Summation
let sum = 0;
for (let i = 1; i <= 100; i++) {
  sum += i;
}
console.log(sum); // 5050

// Reverse iteration
for (let i = 10; i >= 0; i--) {
  console.log(i); // 10, 9, ..., 0
}

// Step by 2
for (let i = 0; i < 10; i += 2) {
  console.log(i); // 0, 2, 4, 6, 8
}
```

### Intermediate Examples
```javascript
// Nested for loops (matrix)
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];
for (let i = 0; i < matrix.length; i++) {
  for (let j = 0; j < matrix[i].length; j++) {
    console.log(matrix[i][j]);
  }
}

// Multiple variables
for (let i = 0, j = 10; i < j; i++, j--) {
  console.log(i, j);
}

// Empty statements
let i = 0;
for (; i < 5; ) {
  console.log(i);
  i++;
}

// Loop with early termination using break
for (let i = 0; i < array.length; i++) {
  if (array[i] === target) {
    console.log(`Found at index ${i}`);
    break;
  }
}

// Continue to skip iterations
for (let i = 0; i < 10; i++) {
  if (i % 2 === 0) continue;
  console.log(i); // 1, 3, 5, 7, 9
}
```

### Advanced Examples
```javascript
// Performance-optimized loop (cache length)
const arr = new Array(1000000).fill(0);
for (let i = 0, len = arr.length; i < len; i++) {
  arr[i] = i;
}

// Looping backwards for mutation
const items = [1, 2, 3, 4, 5];
for (let i = items.length - 1; i >= 0; i--) {
  if (items[i] % 2 === 0) {
    items.splice(i, 1);
  }
}
console.log(items); // [1, 3, 5]

// Loop with labeled break/continue
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) {
      break outer; // breaks out of outer loop
    }
    console.log(i, j);
  }
}
// Output: 0 0, 0 1, 0 2, 1 0

// Generating sequences
function range(start, end, step = 1) {
  const result = [];
  for (let i = start; i < end; i += step) {
    result.push(i);
  }
  return result;
}
console.log(range(0, 10, 2)); // [0, 2, 4, 6, 8]
```

### Real-World Use Cases
- Iterating arrays by index
- Matrix operations (nested loops)
- Generating sequences
- Copying/mutating arrays in-place
- Pagination loops
- setTimeout/setInterval repetition
- Canvas pixel manipulation

### Common Mistakes
- Off-by-one errors (using `<=` instead of `<`)
- Infinite loops (forgetting to increment)
- Not caching array length in performance-critical loops
- Modifying array while iterating forward (use reverse for deletion)
- Using `var` instead of `let` for loop variables

### Best Practices
- Use `let` for loop counters (block-scoped per iteration)
- Cache array length: `for (let i = 0, len = arr.length; i < len; i++)`
- Use `break` and `continue` judiciously
- Prefer `for...of` for simple value iteration
- Keep loop bodies small and focused
- Avoid modifying collections during iteration

### Performance Considerations
- `for` loops are generally the fastest iteration construct
- Pre-calculating length improves performance for large arrays
- Reverse iteration can be faster (no length check each time)
- JIT engines optimize simple `for` loops very well
- Avoid heavy computations in the condition or update expressions

### Interview Questions
1. What are the three parts of a `for` loop?
2. What happens if you omit the condition in a `for` loop?
3. How do you iterate over a 2D array?
4. What is the difference between `break` and `continue`?
5. How do labeled loops work?

### Coding Challenges
1. Implement `Array.prototype.map` using a `for` loop.
2. Write a function that creates a multiplication table using nested loops.
3. Implement a function that flattens a nested array using `for` loops.
4. Create a function that finds the longest word in a sentence using a `for` loop.

### Related Topics
- While loop
- For...of loop
- Array iteration methods
- Break and continue

## while loop

### What It Is
The `while` loop executes a block of code as long as a specified condition is truthy. The condition is checked before each iteration.

### Why It Is Important
`while` loops are ideal when the number of iterations is unknown beforehand. They are commonly used for reading streams, waiting for conditions, and indefinite iteration patterns.

### How It Works Internally
The engine evaluates the condition before each iteration. If truthy, the body executes. After execution, control returns to the condition check. If the condition is falsy initially, the body never executes.

### Syntax
```javascript
while (condition) {
  // Loop body
}
```

### Beginner Examples
```javascript
// Count to 5
let count = 0;
while (count < 5) {
  console.log(count);
  count++;
}

// Sum until limit
let sum = 0;
let num = 1;
while (sum < 100) {
  sum += num;
  num++;
}
console.log(sum, num);

// User input validation (pseudo)
let input = "";
while (input !== "quit") {
  // input = prompt("Enter command:");
  console.log(`You entered: ${input}`);
}

// Random until condition
let random;
while (random !== 7) {
  random = Math.floor(Math.random() * 10) + 1;
  console.log(`Rolled: ${random}`);
}
console.log("Got 7!");
```

### Intermediate Examples
```javascript
// Collatz conjecture
function collatz(n) {
  let steps = 0;
  while (n !== 1) {
    if (n % 2 === 0) {
      n /= 2;
    } else {
      n = 3 * n + 1;
    }
    steps++;
  }
  return steps;
}

// Flatten array using stack
function flatten(arr) {
  const result = [];
  const stack = [arr];
  while (stack.length) {
    const item = stack.shift();
    if (Array.isArray(item)) {
      stack.unshift(...item);
    } else {
      result.push(item);
    }
  }
  return result;
}

// Readable stream processing (Node.js)
function processStream(stream) {
  let data = stream.read();
  while (data !== null) {
    // Process data chunk
    console.log(data.length);
    data = stream.read();
  }
}
```

### Advanced Examples
```javascript
// ID generation until unique
function generateUniqueId(existingIds) {
  let id;
  do {
    id = Math.random().toString(36).substr(2, 9);
  } while (existingIds.has(id));
  return id;
}

// Polling with while (async with sleep)
async function pollUntil(url, condition, interval = 1000) {
  let result;
  while (true) {
    const response = await fetch(url);
    result = await response.json();
    if (condition(result)) break;
    await new Promise(r => setTimeout(r, interval));
  }
  return result;
}

// Binary search
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (arr[mid] === target) return mid;
    if (arr[mid] < target) left = mid + 1;
    else right = mid - 1;
  }
  return -1;
}
```

### Real-World Use Cases
- Event loops in game engines
- Stream processing (reading files, network data)
- Polling APIs or databases for status changes
- Retry logic with exponential backoff
- Recursive algorithm simulation (while + stack)
- Tokenization and parsing
- Binary search algorithms

### Common Mistakes
- Forgetting to update the condition variable (infinite loop)
- Using `while (true)` without a `break` path
- Not handling the case where condition is never met
- Using `while` when iteration count is known (use `for`)
- Side effects in condition expression

### Best Practices
- Ensure the condition variable is updated within the loop
- Use `while (true)` with explicit `break` when indefinite
- Prefer `for` when iteration count is known
- Keep the loop condition simple; extract complex logic to a variable
- Consider a maximum iteration guard to prevent infinite loops

### Performance Considerations
- `while` loops are comparable in speed to `for` loops
- Condition checking overhead is minimal
- Infinite loops will crash the environment (stack overflow or hang)
- JIT optimizes predictable while loops

### Interview Questions
1. When would you use `while` instead of `for`?
2. What is an infinite loop and how do you avoid it?
3. How do you implement a polling mechanism with `while`?
4. What happens if the condition is initially false?

### Coding Challenges
1. Implement `Array.prototype.find` using `while`.
2. Write a function that implements the Fibonacci sequence using `while`.
3. Create a number guessing game using `while`.
4. Implement a simple REPL loop using `while`.

### Related Topics
- Do-while loop
- For loop
- Infinite loops

## do-while loop

### What It Is
The `do-while` loop is similar to `while`, but the condition is checked after each iteration execution, guaranteeing at least one execution of the body.

### Why It Is Important
`do-while` ensures the loop body runs at least once even if the condition is initially false. It is useful for input validation, menus, and scenarios where the first iteration must execute before checking a condition.

### How It Works Internally
The engine executes the body first, then evaluates the condition. If truthy, it repeats; if falsy, the loop terminates. This guarantees at least one body execution regardless of the condition.

### Syntax
```javascript
do {
  // Loop body
} while (condition);
```

### Beginner Examples
```javascript
// At least one execution
let i = 0;
do {
  console.log(i); // 0 (runs once even though condition is false)
  i++;
} while (i < 0);

// Menu display (pseudo)
let choice;
do {
  // choice = prompt("Select option (1-3) or 0 to quit:");
  console.log(`You chose: ${choice}`);
} while (choice !== "0");

// Input validation
let input;
do {
  // input = prompt("Enter a positive number:");
  console.log(`Validating: ${input}`);
} while (input <= 0);
```

### Intermediate Examples
```javascript
// Generate until condition met (always run at least once)
function generatePassword(length) {
  let password;
  do {
    password = Array.from({ length }, () =>
      String.fromCharCode(33 + Math.floor(Math.random() * 94))
    ).join('');
  } while (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(password));
  return password;
}

// Retry mechanism
async function fetchWithRetry(url, maxRetries = 3) {
  let attempts = 0;
  let lastError;
  do {
    try {
      const response = await fetch(url);
      if (response.ok) return response.json();
      throw new Error(`HTTP ${response.status}`);
    } catch (error) {
      lastError = error;
      attempts++;
      if (attempts < maxRetries) {
        await new Promise(r => setTimeout(r, 1000 * attempts));
      }
    }
  } while (attempts < maxRetries);
  throw lastError;
}
```

### Real-World Use Cases
- Displaying menu at least once
- Input validation loops
- Retry logic needing at least one attempt
- Game loops (render once before checking game state)
- Simulation initialization

### Common Mistakes
- Forgetting the semicolon after `while (condition)`
- Using `do-while` when a `while` loop would suffice
- Not handling the case where condition is immediately true (works fine)
- Confusing syntax with `while` loops

### Best Practices
- Use `do-while` only when the body must execute at least once
- Always include a condition that will eventually become falsy
- Ensure the body modifies the condition variable
- Use clear variable names that indicate the loop's purpose

### Performance Considerations
- `do-while` performance is essentially identical to `while`
- The guaranteed first iteration has negligible overhead
- Same optimization characteristics as `while` loops

### Interview Questions
1. How does `do-while` differ from `while`?
2. What happens if the condition in `do-while` is always false?
3. When would you use `do-while` over `while`?

### Coding Challenges
1. Implement a number guessing game that shows the prompt at least once.
2. Write a function using `do-while` that asks for confirmation.
3. Create a simple REPL with at least one prompt.

### Related Topics
- While loop
- For loop

## for...in loop

### What It Is
`for...in` iterates over enumerable string-keyed properties of an object, including inherited enumerable properties.

### Why It Is Important
`for...in` is the primary way to iterate over object property keys. It enables inspection of object structure and enumeration of own and inherited properties.

### How It Works Internally
The engine retrieves the object's enumerable properties (both own and inherited) and iterates through them. The order is guaranteed to follow: integer-like keys in ascending order, then string keys in insertion order, then Symbol keys (though `for...in` does not iterate Symbols).

### Syntax
```javascript
for (const key in object) {
  // key is a string key of the object
}
```

### Beginner Examples
```javascript
// Basic object iteration
const person = { name: "Alice", age: 30, city: "NYC" };
for (const key in person) {
  console.log(`${key}: ${person[key]}`);
}
// name: Alice
// age: 30
// city: NYC

// Checking own properties
for (const key in person) {
  if (person.hasOwnProperty(key)) {
    console.log(`Own property: ${key}`);
  }
}

// Iterating arrays (not recommended)
const arr = [10, 20, 30];
for (const index in arr) {
  console.log(index, arr[index]); // "0" 10, "1" 20, "2" 30
}
// Note: index is a string, not a number!
```

### Intermediate Examples
```javascript
// Prototype chain iteration
function Animal(type) { this.type = type; }
Animal.prototype.eat = function() { console.log("Eating"); };

const dog = new Animal("mammal");
dog.name = "Rex";

for (const key in dog) {
  console.log(key); // "type", "name", "eat" (inherited)
}

// Filtering with hasOwnProperty
for (const key in dog) {
  if (Object.prototype.hasOwnProperty.call(dog, key)) {
    console.log(`Own: ${key}`);
  }
}

// Enumerating objects with getters
const obj = {
  _value: 10,
  get value() { return this._value; },
  set value(v) { this._value = v; }
};
for (const key in obj) console.log(key); // "_value", "value"
```

### Advanced Examples
```javascript
// Using for...in for deep copy
function deepCopy(obj) {
  if (typeof obj !== "object" || obj === null) return obj;
  const copy = Array.isArray(obj) ? [] : {};
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      copy[key] = deepCopy(obj[key]);
    }
  }
  return copy;
}

// Collecting own enumerable keys
function getOwnKeys(obj) {
  const keys = [];
  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      keys.push(key);
    }
  }
  return keys;
}
// Equivalent to Object.keys()

// Merging objects with inherited properties
function mergeWithPrototype(target, source) {
  for (const key in source) {
    target[key] = source[key];
  }
  return target;
}
```

### Real-World Use Cases
- Debugging/inspecting object properties
- Shallow cloning objects
- Serialization of plain objects
- Legacy code iteration patterns
- Working with custom prototypes
- Dynamic property access delegation

### Common Mistakes
- Using `for...in` on arrays (iterates over indices as strings and any other enumerable properties)
- Not using `hasOwnProperty` check and getting unexpected inherited properties
- Assuming keys are iterated in insertion order (integer keys come first)
- Forgetting that `for...in` does not iterate Symbol properties

### Best Practices
- Always use `hasOwnProperty` check (or `Object.hasOwn()`) when using `for...in`
- Use `for...of` or `forEach` for arrays instead of `for...in`
- Use `Object.keys()`, `Object.values()`, `Object.entries()` for own properties
- Avoid adding enumerable properties to prototypes in modern code
- Use `Object.hasOwn(obj, key)` (ES2022) over `obj.hasOwnProperty(key)`

### Performance Considerations
- `for...in` is slower than `for...of` or `forEach` for arrays
- Has to traverse the entire prototype chain (unless filtered)
- Optimized for object property enumeration, not arrays
- JIT can optimize `for...in` on objects with consistent shapes

### Interview Questions
1. What does `for...in` iterate over?
2. Why should you avoid `for...in` for arrays?
3. How do you filter out inherited properties in `for...in`?
4. What is the iteration order of `for...in`?
5. Does `for...in` iterate over Symbol properties?

### Coding Challenges
1. Write a function using `for...in` that counts own properties of an object.
2. Implement a function that copies only own enumerable properties from source to target.
3. Create a deep comparison function that uses `for...in` to compare object properties.
4. Write a custom `Object.keys()` implementation using `for...in`.

### Related Topics
- Object.keys/values/entries
- Property enumeration
- Prototypes and inheritance
- For...of loop

## for...of loop

### What It Is
`for...of` iterates over iterable objects (arrays, strings, Maps, Sets, etc.) and executes the body for each element's value.

### Why It Is Important
`for...of` provides a simple, modern way to iterate over values of any iterable without managing indices. It works with all built-in iterables and custom iterables.

### How It Works Internally
The engine calls the `[Symbol.iterator]()` method of the iterable to get an iterator object. It then calls `next()` on the iterator in each iteration until `done` is true. The value property of the result is assigned to the loop variable.

### Syntax
```javascript
for (const value of iterable) {
  // Loop body
}
```

### Beginner Examples
```javascript
// Array iteration
const colors = ["red", "green", "blue"];
for (const color of colors) {
  console.log(color); // "red", "green", "blue"
}

// String iteration
const str = "hello";
for (const char of str) {
  console.log(char); // "h", "e", "l", "l", "o"
}

// Set iteration
const unique = new Set([1, 2, 3, 2, 1]);
for (const num of unique) {
  console.log(num); // 1, 2, 3
}

// Map iteration
const map = new Map([["a", 1], ["b", 2]]);
for (const [key, value] of map) {
  console.log(key, value); // "a" 1, "b" 2
}

// Using const for the iteration variable
const numbers = [1, 2, 3];
for (const num of numbers) {
  console.log(num * 2); // 2, 4, 6
}
```

### Intermediate Examples
```javascript
// Iterating with index (use entries)
const items = ["a", "b", "c"];
for (const [index, value] of items.entries()) {
  console.log(index, value);
}

// Iterating over arguments (array-like)
function sum() {
  let total = 0;
  for (const num of arguments) {
    total += num;
  }
  return total;
}
console.log(sum(1, 2, 3)); // 6

// Iterating over NodeList (DOM)
const paragraphs = document.querySelectorAll("p");
for (const p of paragraphs) {
  p.classList.add("highlight");
}

// Iterating generator output
function* fibonacci(n) {
  let a = 0, b = 1;
  while (n-- > 0) {
    yield a;
    [a, b] = [b, a + b];
  }
}
for (const num of fibonacci(10)) {
  console.log(num); // 0, 1, 1, 2, 3, 5, 8, 13, 21, 34
}
```

### Advanced Examples
```javascript
// Custom iterable object
const rangeIterable = {
  from: 0,
  to: 5,
  [Symbol.iterator]() {
    let current = this.from;
    const end = this.to;
    return {
      next() {
        return current <= end
          ? { value: current++, done: false }
          : { value: undefined, done: true };
      }
    };
  }
};
for (const num of rangeIterable) {
  console.log(num); // 0, 1, 2, 3, 4, 5
}

// Async iteration
async function* asyncGenerator() {
  for (let i = 0; i < 3; i++) {
    await new Promise(r => setTimeout(r, 100));
    yield i;
  }
}
(async () => {
  for await (const num of asyncGenerator()) {
    console.log(num); // 0, 1, 2 (with delays)
  }
})();

// Destructuring in iteration
const users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" }
];
for (const { id, name } of users) {
  console.log(`${id}: ${name}`);
}

// Break and continue in for...of
for (const num of [1, 2, 3, 4, 5]) {
  if (num === 3) continue;
  if (num === 5) break;
  console.log(num); // 1, 2, 4
}
```

### Real-World Use Cases
- Iterating over API responses (arrays of objects)
- DOM NodeList manipulation
- String character processing
- Map/Set data structure iteration
- Generator function consumption
- Streaming data processing (Node.js streams with async iteration)
- Working with typed arrays (Uint8Array, etc.)

### Common Mistakes
- Trying to use `for...of` on plain objects (not iterable)
- Forgetting that `for...of` gives values, not indices
- Using `for...of` when you need index access (use `.entries()`)
- Modifying the iterable during iteration (undefined behavior for some types)
- Assuming `for...of` works with array-like objects without Symbol.iterator

### Best Practices
- Use `for...of` as the default iteration method for iterables
- Use `.entries()` when you need both index and value
- Use `for...of` over `forEach` when you need `break`/`continue`
- Use `for...of` with `const` for the iteration variable
- Use `for await...of` for async iterables
- Prefer `for...of` over `for...in` for arrays and iterables

### Performance Considerations
- `for...of` is slightly slower than indexed `for` loop for arrays
- Performance difference is negligible for most use cases
- For large arrays, a traditional `for` loop may be marginally faster
- JIT optimization of `for...of` is improving in modern engines
- Generator-based `for...of` has more overhead

### Interview Questions
1. What types can `for...of` iterate over?
2. How does `for...of` differ from `for...in`?
3. How do you get the index in a `for...of` loop?
4. What is the `Symbol.iterator` protocol?
5. How do you use `for...of` with async generators?

### Coding Challenges
1. Implement `Array.prototype.forEach` using `for...of`.
2. Create a custom iterable object that generates prime numbers.
3. Write a function that merges multiple arrays using `for...of`.
4. Implement a pagination iterator using `for...of` and generators.

### Related Topics
- Iterators and iterables
- Symbol.iterator
- Generators
- For...in loop

## break and continue

### What It Is
`break` terminates the current loop or switch statement completely. `continue` skips the rest of the current iteration and proceeds to the next iteration.

### Why It Is Important
These statements provide fine-grained control over loop execution. `break` enables early exit when a condition is met, saving unnecessary iterations. `continue` allows skipping specific iterations without breaking the loop structure.

### How It Works Internally
For `break`, the engine jumps to the first statement after the loop or switch. For `continue`, the engine jumps to the loop's update expression (for `for` loops) or condition check (for `while`/`do-while`). Both can be used with loop labels to break/continue outer loops.

### Syntax
```javascript
// Basic break and continue
for (let i = 0; i < 10; i++) {
  if (i === 3) break;     // terminates loop at i=3
  if (i === 1) continue;  // skips i=1
  console.log(i);         // 0, 2
}

// Labeled break/continue
outerLoop: for (let i = 0; i < 3; i++) {
  innerLoop: for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) break outerLoop;
    console.log(i, j);
  }
}
```

### Beginner Examples
```javascript
// Break: find first even number
const numbers = [1, 3, 5, 6, 7, 8];
for (const num of numbers) {
  if (num % 2 === 0) {
    console.log(`Found even: ${num}`);
    break;
  }
}

// Continue: skip odd numbers
for (let i = 0; i < 10; i++) {
  if (i % 2 !== 0) continue;
  console.log(i); // 0, 2, 4, 6, 8
}

// Break in while loop
let sum = 0;
let i = 1;
while (true) {
  sum += i;
  if (sum > 100) break;
  i++;
}
```

### Intermediate Examples
```javascript
// Continue with label
skipRow: for (let row = 0; row < 4; row++) {
  for (let col = 0; col < 4; col++) {
    if (col === 2) continue skipRow; // skip rest of row
    console.log(row, col);
  }
}

// Finding multiple results with break
function findFirstTwo(arr, predicate) {
  const results = [];
  for (const item of arr) {
    if (predicate(item)) {
      results.push(item);
      if (results.length === 2) break;
    }
  }
  return results;
}

// Skipping with continue in filter-like pattern
function removeDuplicates(arr) {
  const seen = new Set();
  const result = [];
  for (const item of arr) {
    if (seen.has(item)) continue;
    seen.add(item);
    result.push(item);
  }
  return result;
}
```

### Advanced Examples
```javascript
// Nested loop with labeled break
function findInMatrix(matrix, target) {
  let result = null;
  search: for (let i = 0; i < matrix.length; i++) {
    for (let j = 0; j < matrix[i].length; j++) {
      if (matrix[i][j] === target) {
        result = { row: i, col: j };
        break search;
      }
    }
  }
  return result;
}

// Continue with early filtering
function processPayloads(payloads) {
  for (const payload of payloads) {
    if (!payload.isValid) continue;
    if (payload.size > MAX_SIZE) continue;
    if (payload.type === "deprecated") continue;
    // Process valid payload
    process(payload);
  }
}

// Break with debounced observer pattern
function observeUntil(observable, predicate) {
  let unsubscribe;
  const result = new Promise((resolve, reject) => {
    unsubscribe = observable.subscribe(value => {
      if (predicate(value)) {
        resolve(value);
      }
    });
  });
  return result.finally(() => unsubscribe?.());
}
```

### Real-World Use Cases
- Early exit when target found (search algorithms)
- Skipping invalid items in data processing
- Breaking out of nested loops in game logic
- Pagination: stop when no more data
- Filtering with continue for cleaner code
- Validation loops with break for first error
- Timeout-based loops with break

### Common Mistakes
- Using `break` outside of a loop or switch (syntax error)
- Using `continue` in a `switch` statement (acts on surrounding loop)
- Forgetting that `continue` in `while` loop re-evaluates condition (may cause infinite loop)
- Not using labels when breaking outer loops from deeply nested code
- Overusing `break`/`continue` making code harder to follow

### Best Practices
- Use `break` for early exit when further iteration is unnecessary
- Use `continue` to skip iterations, not to hide complex logic
- Prefer extracting loop body to a function over using `continue` excessively
- Use labeled `break` sparingly; consider extracting to a function with return
- Ensure all code paths are covered when using `break` early
- Use `break` in infinite loops (`while (true)`) with clear exit conditions

### Performance Considerations
- `break` can improve performance by reducing unnecessary iterations
- `continue` has negligible overhead
- Labeled break/continue don't add runtime cost
- JIT optimizes predictable loop flows

### Interview Questions
1. What is the difference between `break` and `continue`?
2. How do labeled breaks work in nested loops?
3. Can you use `break` in a `switch` statement inside a loop?
4. What happens if you use `continue` in a `while` loop without updating the condition?
5. When is it appropriate to use `break`?

### Coding Challenges
1. Write a function that finds the first index of an element using `break`.
2. Implement a function that skips every nth element using `continue`.
3. Create a nested loop that uses labeled break to exit completely when a condition is met.
4. Write a data processing pipeline that uses `continue` for filtering and `break` for early termination.

### Related Topics
- For loop
- While loop
- Switch statement
- Loop labels
