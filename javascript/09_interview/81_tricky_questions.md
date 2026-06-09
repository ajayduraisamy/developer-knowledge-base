# Tricky JS Questions - Coercion puzzles, async order, edge cases, gotchas

## Introduction

JavaScript is notorious for its quirks and gotchas. Type coercion puzzles, unexpected async execution order, and edge cases in the language specification often appear in technical interviews to gauge a candidate's depth of understanding. These questions separate developers who have memorized syntax from those who understand how JavaScript actually works. This file dissects the trickiest aspects of the language with explanations and practical implications.

## Type Coercion Puzzles

### What It Is

Type coercion is JavaScript's automatic conversion of values from one data type to another, typically when operators are applied to mismatched types. JavaScript has both implicit coercion (automatically performed by the engine) and explicit coercion (deliberately done by the developer via `Number()`, `String()`, `Boolean()`, etc.). The coercion rules follow the Abstract Equality Comparison algorithm for `==` and the ToPrimitive/ToNumber/ToString abstract operations.

### Why It Is Important

Understanding coercion is critical for writing predictable code and debugging mysterious bugs. Coercion is the source of many JavaScript memes and failure points (`[] + []`, `{} + []`, `'5' - 3` vs `'5' + 3`). In interviews, coercion questions test the candidate's knowledge of the specification and their ability to reason about edge cases.

### How It Works Internally

The `==` operator follows the Abstract Equality Comparison algorithm (ES specification section 7.2.14). Key steps:
1. If same type, compare directly (===)
2. If `null == undefined`, return `true`
3. If one is a number and one is a string, convert string to number
4. If one is a boolean, convert boolean to number (true → 1, false → 0)
5. If one is an object and the other is a string/number/symbol, convert object to primitive (ToPrimitive)

The `+` operator: if either operand is a string, do string concatenation; otherwise, do numeric addition.
The `-`, `*`, `/` operators: always convert both operands to numbers.

### Syntax

```javascript
// == vs ===
0 == ''       // true ('' → 0)
0 == '0'      // true ('0' → 0)
false == '0'  // true (false → 0, '0' → 0)
null == undefined // true
[] == ![]     // true (mind-blowing!)
[] == 0       // true
'\t\n' == 0   // true

// + operator
1 + '2'       // '12' (string concat)
'1' + 2       // '12'
1 + 2 + '3'   // '33' (1+2=3, then 3+'3'='33')
'1' + 2 + 3   // '123'
1 - '2'       // -1 (numeric)
'5' * '3'     // 15
'10' / '2'    // 5

// Boolean coercion
!!'false'     // true (non-empty string is truthy)
!!''          // false
!!0           // false
!!'0'         // true
!![]          // true
!!{}          // true

// Object coercion
[1,2] + [3,4] // '1,23,4' (toString then concat)
{} + []       // 0 ({} is empty block, +[] is 0)
[] + {}       // '[object Object]'
```

### Beginner Examples

```javascript
// Common gotchas
console.log(0.1 + 0.2 === 0.3); // false (floating point)
console.log(0.1 + 0.2); // 0.30000000000000004

console.log(typeof NaN); // 'number'
console.log(NaN === NaN); // false (NaN is never equal to itself)
console.log(isNaN('hello')); // true (coerces to number first)
console.log(Number.isNaN('hello')); // false (no coercion)

console.log([] == false); // true
console.log([] == 0);     // true
console.log('' == false); // true
console.log('' == 0);     // true
```

### Intermediate Examples

```javascript
// Falsy values (only 7 in all of JS)
const falsy = [false, 0, -0, 0n, '', null, undefined, NaN];
falsy.forEach(v => console.log(Boolean(v))); // All false

// Truthy gotchas
Boolean('false');   // true (non-empty string)
Boolean(' ');       // true (non-empty string)
Boolean([]);        // true (empty array)
Boolean({});        // true (empty object)
Boolean(new Boolean(false)); // true (object wrapper)

// Coercion chain puzzles
console.log(true + false);          // 1 (1 + 0)
console.log(16 / 4 / 2);            // 2 (not 8)
console.log(1 + 2 + '3' + 4 + 5);   // '3345'

// Array equality
[] == []  // false (different references)
![] == [] // true (![] is false, false == [] → 0 == '' → 0 == 0 → true)

// parseInt quirks
parseInt('08');   // 8 (in modern engines, but 0 in old IE)
parseInt('0x10');  // 16 (hex)
parseInt('10', 2); // 2 (binary)
parseInt('10', 0); // 10 (0 means detect, but might be 10)

// typeof quirks
typeof null       // 'object' (historical bug)
typeof undefined  // 'undefined'
typeof [];        // 'object'
typeof NaN;       // 'number'
```

### Advanced Examples

```javascript
// Deep coercion chain
const x = {
  valueOf: () => 1,
  toString: () => 'custom'
};

console.log(x + 2);   // 3 (uses valueOf for addition)
console.log(`${x}`);  // 'custom' (toString for template literal)
console.log(x == '1'); // true ('1' == 1, x valueOf is 1)

// Custom coercion with Symbol.toPrimitive
const obj = {
  [Symbol.toPrimitive](hint) {
    if (hint === 'number') return 42;
    if (hint === 'string') return 'answer';
    return 'default';
  }
};

console.log(+obj);     // 42
console.log(`${obj}`); // 'answer'
console.log(obj + ''); // 'default'

// The infamous WTF array
const arr = [1, 2, 3];
arr.valueOf = () => [4, 5, 6];
console.log(arr + 1);    // '1,2,31' (valueOf returns object, uses toString)
console.log(+arr);       // NaN (cannot convert [1,2,3] to number)

// True coercion of dates
const d = new Date();
console.log(d + 1);     // Date string + 1
console.log(+d);        // Timestamp number
console.log(d - 0);     // Timestamp number

// Negative zero
const negZero = -0;
console.log(negZero === 0); // true
console.log(1 / negZero);   // -Infinity
console.log(Object.is(negZero, 0)); // false

// Array destructuring with coercion
const [a, b] = 'hello';
console.log(a, b); // 'h' 'e' (strings are iterable)

// Comparing NaN
Object.is(NaN, NaN); // true (unlike ===)
```

### Real-World Use Cases

- API response handling (coercing string numbers to actual numbers)
- Form validation (empty string vs 0 vs null)
- Config merging default values (truthy/falsy checks)
- Dynamic property access (object to string conversion for keys)
- Sorting algorithms (handling mixed types)
- Serialization/deserialization (JSON)

### Common Mistakes

- Using `==` instead of `===` to avoid coercion surprises
- Checking for array emptiness with `if (arr)` (empty array is truthy)
- Using `isNaN()` instead of `Number.isNaN()` (isNaN coerces)
- Assuming `0` is falsy in all contexts (it is, but `'0'` is truthy)
- Comparing floating point numbers directly
- Confusing `null` vs `undefined` comparisons

### Best Practices

- Always use `===` and `!==` (except when explicitly checking for `null`/`undefined`)
- Use explicit coercion: `String()`, `Number()`, `Boolean()`
- Use `Number.isNaN()` instead of `isNaN()` for NaN checks
- Use `Object.is()` for strict equality including -0 and NaN
- Use epsilon comparisons for floating point: `Math.abs(a - b) < Number.EPSILON`
- Avoid `==` in production code (enable ESLint `eqeqeq` rule)

### Performance Considerations

- `===` is faster than `==` because it skips the coercion algorithm
- Explicit coercion has predictable performance
- String concatenation with `+` is slower than template literals for complex cases
- Number conversion via `+str` is faster than `Number(str)` or `parseInt(str)`
- Coercion during comparisons is optimized in V8 but still slower than strict equality

### Interview Questions

**Q: Why does `[] == ![]` evaluate to `true`?**
A: `![]` is `false` (because `[]` is truthy). So we have `[] == false`. `false` is coerced to `0`. Then `[]` is coerced to `''` via `toString()`, and `'' == 0` is `true` because `''` is coerced to `0`. The chain: `[] == ![]` → `[] == false` → `[] == 0` → `'' == 0` → `0 == 0` → `true`.

**Q: What is the difference between `==` and `===`?** A: `===` (strict equality) compares both value and type without coercion. `==` (abstract equality) performs type coercion before comparison according to the Abstract Equality Comparison algorithm. Always prefer `===` to avoid unexpected coercion behavior.

### Coding Challenges

**Challenge 1:** Implement a function that safely compares two values, handling NaN and -0.

```javascript
function safeEqual(a, b) {
  // Your implementation
}
```

**Solution:**

```javascript
function safeEqual(a, b) {
  if (Object.is(a, b)) return true;  // Handles NaN and -0
  return false;
}

// Or using the specification algorithm:
function safeEqual(a, b) {
  if (Number.isNaN(a) && Number.isNaN(b)) return true;
  if (Object.is(a, -0) && Object.is(b, -0)) return true;
  if (Object.is(a, -0)) return 1 / a === 1 / b;
  return a === b;
}
```

## Async Execution Order

### What It Is

JavaScript's async execution order is governed by the event loop, which manages the execution of synchronous code, microtasks (Promises, MutationObserver, queueMicrotask), and macrotasks (setTimeout, setInterval, I/O, UI rendering). Understanding the precise order in which these queues drain is essential for predicting async behavior.

### Why It Is Important

Async execution order questions are among the most common and revealing interview questions. They test understanding of the event loop, microtask vs macrotask priority, and the Promise lifecycle. Misunderstanding these leads to race conditions, timing bugs, and incorrect assumptions about async flow.

### How It Works Internally

The event loop processes in this order:
1. Execute all synchronous code (current macrotask)
2. Drain the entire microtask queue (Promise callbacks, queueMicrotask)
3. Execute the next macrotask (setTimeout callback, I/O)
4. Repeat

Between macrotasks, the browser may repaint/render. Microtasks are processed before the next macrotask, which means Promise callbacks run before setTimeout callbacks, even if the setTimeout delay is 0.

```javascript
console.log(1);           // Sync
setTimeout(() => console.log(2), 0); // Macrotask
Promise.resolve().then(() => console.log(3)); // Microtask
console.log(4);           // Sync
// Output: 1, 4, 3, 2
```

### Syntax

```javascript
// Microtask creation
Promise.resolve().then(() => {});
queueMicrotask(() => {});
// MutationObserver callbacks also run as microtasks

// Macrotask creation
setTimeout(() => {}, 0);
setInterval(() => {}, 0);
// I/O callbacks (fs.readFile, etc.)
// requestAnimationFrame (special: runs before paint)

// Async/await internally uses microtasks
async function foo() {
  await bar(); // The code after await runs as a microtask
}
```

### Beginner Examples

```javascript
// Basic event loop order
console.log('A');
setTimeout(() => console.log('B'), 0);
Promise.resolve().then(() => console.log('C'));
console.log('D');
// Output: A, D, C, B

// Nested promises
Promise.resolve()
  .then(() => console.log('1'))
  .then(() => console.log('2'));

Promise.resolve()
  .then(() => console.log('3'))
  .then(() => console.log('4'));
// Output: 1, 3, 2, 4

// setTimeout vs Promise
console.log('start');
setTimeout(() => console.log('timeout'), 0);
Promise.resolve('promise').then(v => console.log(v));
console.log('end');
// Output: start, end, promise, timeout
```

### Intermediate Examples

```javascript
// Mixing microtasks and macrotasks
setTimeout(() => console.log('timeout1'), 0);
Promise.resolve().then(() => {
  console.log('promise1');
  setTimeout(() => console.log('timeout2'), 0);
});
Promise.resolve().then(() => console.log('promise2'));
console.log('sync');
// Output: sync, promise1, promise2, timeout1, timeout2

// Async/await microtask behavior
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end'); // This is a microtask
}

async function async2() {
  console.log('async2');
}

console.log('script start');
setTimeout(() => console.log('setTimeout'), 0);
async1();
new Promise(resolve => {
  console.log('promise1');
  resolve();
}).then(() => console.log('promise2'));
console.log('script end');

// Output: script start, async1 start, async2, promise1, script end,
//         async1 end, promise2, setTimeout

// Promise constructor is synchronous
const p = new Promise(resolve => {
  console.log('inside promise'); // Run synchronously
  resolve('resolved');
});
p.then(v => console.log(v)); // Microtask
console.log('after promise');
// Output: inside promise, after promise, resolved
```

### Advanced Examples

```javascript
// Promise vs process.nextTick (Node.js)
// process.nextTick runs before Promise microtasks in Node.js
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));
setTimeout(() => console.log('timeout'), 0);
console.log('sync');
// Node.js Output: sync, nextTick, promise, timeout

// Multiple microtask queue entries
setTimeout(() => console.log('t1'));
setTimeout(() => {
  console.log('t2');
  Promise.resolve().then(() => console.log('t2-promise'));
});
setTimeout(() => console.log('t3'));
// Output: t1, t2, t2-promise, t3

// Promise resolution with thenables
const thenable = {
  then(resolve) {
    console.log('thenable called');
    resolve('from thenable');
  }
};

Promise.resolve(thenable).then(v => console.log(v));
// The thenable is unwrapped asynchronously

// Async generator execution
async function* gen() {
  yield await Promise.resolve(1);
  yield await Promise.resolve(2);
}

(async () => {
  for await (const val of gen()) {
    console.log(val);
  }
})();

// Unhandled rejection timing
const p = Promise.reject(new Error('test'));
setTimeout(() => {
  p.catch(() => console.log('caught'));
}, 100);
// Unhandled rejection is detected after the microtask queue drains

// Nested async/await with timing
const start = Date.now();
async function test() {
  console.log('start', Date.now() - start);
  await null; // This creates a microtask boundary
  console.log('after await null', Date.now() - start);
  await Promise.resolve();
  console.log('after await promise', Date.now() - start);
  await new Promise(resolve => setTimeout(resolve, 10));
  console.log('after 10ms', Date.now() - start);
}
test();
```

### Real-World Use Cases

- Correctly sequencing API calls with authentication token refresh
- Animations and frame scheduling with requestAnimationFrame
- React's state batching and async rendering
- Database transaction ordering in Node.js
- Ensuring UI updates happen after state changes
- Race condition prevention in concurrent code

### Common Mistakes

- Assuming `setTimeout(fn, 0)` runs immediately (it runs after microtasks)
- Not realizing `await` creates a microtask boundary
- Thinking Promise constructor runs asynchronously (it runs synchronously)
- Forgetting that `.then()` creates a new Promise (forgetting to return)
- Mixed async patterns causing unpredictable order

### Best Practices

- Use `await` in async functions rather than `.then()` chains
- Be explicit about execution order with `queueMicrotask` when needed
- Understand that `await` pauses until the awaited promise settles
- Avoid relying on microtask vs macrotask ordering in production code
- Use `Promise.all()` for parallel operations with predictable resolution
- Use synchronization primitives (mutex, semaphore) for complex ordering

### Performance Considerations

- Microtasks are processed continuously until the queue is empty (can starve rendering)
- Long microtask chains can block UI updates
- setTimeout(fn, 0) has a minimum delay of 4ms for nested calls (HTML spec)
- Promise resolution adds minimal overhead (microtask)
- V8 optimizes Promise creation and chaining heavily
- Excessive microtask queuing can cause "microtask starvation"

### Interview Questions

**Q: Why does `setTimeout(fn, 0)` not execute immediately?**
A: `setTimeout` queues a macrotask that runs only after the current synchronous code and all microtasks complete. Even with 0ms delay, the minimum delay is enforced (0ms for top-level, 4ms for nested), and the callback must wait for the event loop to process all pending microtasks first.

**Q: What is the difference between microtasks and macrotasks?**
A: Microtasks (Promise callbacks, `queueMicrotask`, `MutationObserver`) are processed immediately after the current macrotask completes and before the next macrotask. All microtasks are drained before the next macrotask begins. Macrotasks (setTimeout, setInterval, I/O) are processed one per event loop iteration, with rendering potentially happening between them.

### Coding Challenges

**Challenge 1:** Write the output order of this code:

```javascript
async function f1() {
  console.log('f1 start');
  await f2();
  console.log('f1 end');
}

async function f2() {
  console.log('f2');
}

console.log('start');
setTimeout(() => console.log('timeout'), 0);
f1();
new Promise(r => (console.log('promise'), r()))
  .then(() => console.log('then'));
console.log('end');
```

**Solution:**

```
start, f1 start, f2, promise, end, f1 end, then, timeout
```

## Edge Case Gotchas

### What It Is

Edge cases in JavaScript refer to unexpected behaviors that arise from language design decisions, specification subtleties, and historical artifacts. These include `NaN` behavior, floating-point precision, `null` vs `undefined`, array holes, sparse arrays, `undefined` vs undeclared variables, and the nuances of `delete`, `in`, and property enumeration.

### Why It Is Important

Edge cases are where production bugs hide. Knowledge of edge cases distinguishes senior developers who can anticipate and prevent subtle bugs. Interviewers use edge case questions to evaluate a candidate's experience with real-world debugging and their depth of language knowledge beyond surface-level usage.

### How It Works Internally

Many edge cases arise from the ECMAScript specification's abstract operations. For example, `ToNumber` converts various inputs to numbers with specific rules. `ToPrimitive` converts objects to primitives. The `[[Delete]]` internal method handles property removal. Understanding these abstract operations helps explain seemingly bizarre behavior.

### Syntax

```javascript
// NaN
NaN === NaN          // false
Object.is(NaN, NaN)  // true
Math.min() > Math.max() // true (Math.min() = Infinity, Math.max() = -Infinity)

// Array edge cases
[, ,]                 // Length 2 (trailing comma is ignored)
[1, , 3]             // [1, empty, 3] - sparse array
[1, , 3].length      // 3
[1, , 3][1]          // undefined
delete [1,2,3][1];    // [1, empty, 3]

// null vs undefined
null == undefined    // true
null === undefined   // false
typeof null          // 'object'
typeof undefined     // 'undefined'

// parseInt with leading zeros
parseInt('08')       // 8 (ES5+), 0 (old browsers with octal)
parseInt('0x10')     // 16

// Floating point
0.1 + 0.2 !== 0.3   // true
Number.EPSILON       // 2.220446049250313e-16
```

### Beginner Examples

```javascript
// The global object
var globalVar = 'test';
console.log(window.globalVar); // 'test' (var creates global property)

let letVar = 'test';
console.log(window.letVar); // undefined (let does not create global property)

// Function arguments
function test(a, b) {
  console.log(arguments.length); // 1
  console.log(arguments[0], arguments[1]); // 'first', undefined
  a = 'changed';
  console.log(arguments[0]); // 'changed' (in non-strict mode)
}
test('first');

// Sort by default is lexicographic
[1, 3, 20, 100].sort(); // [1, 100, 20, 3]

// instanceof with primitives
'hello' instanceof String  // false
new String('hello') instanceof String // true

// typeof
typeof null          // 'object'
typeof function(){}  // 'function'
typeof class{}       // 'function'
typeof [1,2,3]       // 'object'
```

### Intermediate Examples

```javascript
// Array methods don't visit holes
const arr = [1, , 3];
arr.forEach(v => console.log(v)); // 1, 3 (skips hole)
arr.map(v => v * 2);              // [2, empty, 6]
arr.filter(v => true);            // [1, 3] (filters out holes)

// delete doesn't change array length
const a = [1, 2, 3];
delete a[1];
console.log(a.length); // 3
console.log(a);        // [1, empty, 3]

// Object property enumeration
const obj = { a: 1, b: 2 };
Object.prototype.c = 3;
for (const key in obj) {
  console.log(key); // 'a', 'b', 'c' (inherited!)
}
console.log(Object.keys(obj)); // ['a', 'b'] (own only)

// Property descriptor defaults
Object.getOwnPropertyDescriptor({}, 'x'); // undefined
Object.getOwnPropertyDescriptor({x:1}, 'x');
// { value: 1, writable: true, enumerable: true, configurable: true }

// String indexing
'hello'[0];          // 'h'
'hello'[-1];         // undefined
'hello'['0'];        // 'h' (coerced to number)
'hello'.length;      // 5
'hello'.length = 3;  // 5 (strings are immutable)

// + vs Number()
+'';                 // 0
+' ';                // 0
+'\t\n';             // 0
+'0';                // 0
+'abc';              // NaN
```

### Advanced Examples

```javascript
// Property accessor edge cases
const obj = { a: 1 };
console.log(obj['a']);  // 1
console.log(obj[a]);    // ReferenceError: a is not defined
const b = 'a';
console.log(obj[b]);    // 1

// parseInt differences
parseInt(0.0000008);   // 8 (stringified as '8e-7', parseInt stops at non-digit)
parseInt(1000000000000000000000); // 1 (stringified as '1e+21')

// Array constructor trap
new Array(3);           // [empty × 3]
new Array(3, 4);        // [3, 4]
Array.of(3);            // [3]

// toString and valueOf interaction
const obj2 = {
  toString() { return '2'; },
  valueOf() { return 1; }
};
console.log(obj2 + 1);  // 2 (uses valueOf)
console.log(`${obj2}`); // '2' (uses toString)

// RegExp with global flag and exec
const re = /a/g;
console.log(re.exec('aba')); // ['a', index: 0]
console.log(re.exec('aba')); // ['a', index: 2]
console.log(re.exec('aba')); // null (lastIndex reset)

// Object.is for edge cases
Object.is(NaN, NaN);     // true
Object.is(-0, 0);        // false
Object.is(-0, -0);       // true

// Async function return value
const asyncFn = async () => {};
asyncFn();  // Promise<undefined>

const syncReturn = () => {};
syncReturn(); // undefined

// setTimeout this behavior
setTimeout(function() {
  console.log(this); // global (or undefined in strict)
}, 0);

const obj3 = { name: 'test' };
setTimeout(obj3.method, 0); // this is NOT obj3 (method extracted)

// Semicolon insertion gotchas
function returns() {
  return
  {
    value: 42
  };
}
console.log(returns()); // undefined (ASI inserts ; after return)
```

### Real-World Use Cases

- Serialization: handling `NaN`, `Infinity`, `-0` in JSON (they become `null` or are lost)
- CSV parsing: `parseInt` with leading zeros (dates, codes)
- Form validation: distinguishing empty string, 0, null, and undefined
- Array operations: sparse arrays from `delete` or array literal holes
- Configuration merging: handling `null` vs `undefined` defaults
- Authorization: checking for property existence with `in` vs `hasOwnProperty`

### Common Mistakes

- Using `==` to check for `null`/`undefined` (works but confusing)
- Assuming array methods always traverse all indices (they skip holes)
- Comparing `NaN` directly (always false)
- Using `delete` on array elements (creates holes, doesn't change length)
- Forgetting that `sort()` sorts lexicographically by default
- Using `typeof` to check for arrays (use `Array.isArray()`)
- Confusing `undefined` vs undeclared variables

### Best Practices

- Use `Array.isArray()` for array checks
- Use `Object.is()` for precise comparisons including NaN and -0
- Use `Number.isNaN()` (not `isNaN()`) for NaN checks
- Use `.fill()` or `Array.from()` to avoid sparse arrays
- Provide compare function for `.sort()`: `arr.sort((a, b) => a - b)`
- Use `let`/`const` instead of `var` to avoid global property creation
- Use `Number.isInteger()` for integer checks
- Always use strict mode: `'use strict'`

### Performance Considerations

- Non-contiguous (sparse) arrays are slower in V8 than dense arrays
- Property lookup on objects with deleted properties may trigger deoptimization
- `Object.is` is slightly slower than `===` but necessary for NaN/-0
- `Number.isNaN` is faster than `isNaN` (no coercion)
- Engine optimizations for monomorphic property access (hidden classes)
- Sparse arrays use a dictionary-based representation (slower)

### Interview Questions

**Q: Why does `typeof null` return `'object'`?** A: This is a historical bug from the first version of JavaScript. The type tag for objects was 0, and `null` was represented as the null pointer (0x00 in most implementations), which was incorrectly classified as an object type tag. It cannot be fixed because existing code depends on this behavior.

**Q: What is the difference between sparse arrays and dense arrays?** A: Dense arrays have contiguous indices from 0 to length-1. Sparse arrays have gaps (holes) where indices are missing. Sparse arrays are created via `new Array(n)`, `delete`, or literal holes `[1,,3]`. Sparse array operations are slower because V8 uses a dictionary representation instead of a contiguous backing store.

### Related Topics

- JavaScript specification abstract operations
- V8 hidden classes and object representation
- Array internals (elements kinds in V8)
- Event loop deep dive
- Coercion in the specification
- Strict mode differences
- JSON serialization quirks
- ES2024+ proposals addressing edge cases
- Testing strategies for edge cases
