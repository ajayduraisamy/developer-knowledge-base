# Arrays - Array() constructor, indexing, methods, iteration

## Introduction
Arrays are ordered, integer-indexed collections of values. They are dynamic, zero-indexed, and can hold elements of any type. Arrays are Objects with special behavior including automatic length tracking and a rich set of built-in methods.

## Array() constructor

### What It Is
The `Array` constructor creates array objects. It can be called with or without `new`, with multiple arguments to create an array of those elements, or with a single numeric argument to pre-allocate length.

### Why It Is Important
Understanding array creation patterns is essential. The `Array()` constructor's behavior differs based on argument count and type, which can lead to bugs if not understood.

### How It Works Internally
When called with a single numeric argument, `Array(n)` sets the length to n but creates no indexed properties (sparse array). When called with multiple arguments, an array containing those arguments is created. The internal `[[DefineOwnProperty]]` is used for index assignment.

### Syntax
```javascript
// Array literal (preferred)
const arr = [1, 2, 3];

// Array constructor
const arr1 = Array(1, 2, 3);   // [1, 2, 3]
const arr2 = Array(3);          // [empty × 3] (sparse)
const arr3 = new Array(3);      // same as Array(3)

// Array.of() (ES6) - avoids single-number behavior
const arr4 = Array.of(3);       // [3]

// Array.from() (ES6) - from iterables
const arr5 = Array.from("hello"); // ["h", "e", "l", "l", "o"]
const arr6 = Array.from({ length: 3 }, (_, i) => i); // [0, 1, 2]
```

### Beginner Examples
```javascript
// Literal syntax (most common)
const fruits = ["apple", "banana", "orange"];
const mixed = [1, "hello", true, null, { a: 1 }];
const empty = [];

// Array() with multiple args
const numbers = Array(1, 2, 3, 4, 5);
console.log(numbers); // [1, 2, 3, 4, 5]

// Array() with single number (creates sparse array)
const sparse = Array(3);
console.log(sparse);        // [empty × 3]
console.log(sparse.length); // 3
console.log(sparse[0]);     // undefined

// Array.of() - single number becomes element
const withOf = Array.of(3);
console.log(withOf); // [3]

// Array.from() with string
const chars = Array.from("abc");
console.log(chars); // ["a", "b", "c"]
```

### Intermediate Examples
```javascript
// Array.from() with map function
const squares = Array.from([1, 2, 3, 4], x => x * x);
console.log(squares); // [1, 4, 9, 16]

// Array.from() with array-like objects
function toArray() {
  return Array.from(arguments);
}
console.log(toArray(1, 2, 3)); // [1, 2, 3]

// Array.from() with { length }
const range = Array.from({ length: 5 }, (_, i) => i + 1);
console.log(range); // [1, 2, 3, 4, 5]

// Creating arrays with holes
const holes = [1, , 3];
console.log(holes);          // [1, empty, 3]
console.log(holes[1]);       // undefined
console.log(0 in holes);     // true (index 0 exists)
console.log(1 in holes);     // false (index 1 doesn't exist)

// Array() for initialization
const zeros = new Array(5).fill(0);
console.log(zeros); // [0, 0, 0, 0, 0]
```

### Advanced Examples
```javascript
// Array.from() with Set for deduplication
const duplicate = [1, 2, 2, 3, 3, 4];
const unique = Array.from(new Set(duplicate));
console.log(unique); // [1, 2, 3, 4]

// Array.from() with Map
const map = new Map([["a", 1], ["b", 2], ["c", 3]]);
const entries = Array.from(map);
console.log(entries); // [["a", 1], ["b", 2], ["c", 3]]

// Creating 2D arrays
const matrix = Array.from({ length: 3 }, () => Array(3).fill(0));
console.log(matrix); // [[0,0,0], [0,0,0], [0,0,0]]

// Performance: Array.from vs spread for Set
const set = new Set([1, 2, 3]);
const arrA = Array.from(set);
const arrB = [...set];
// Array.from is slightly faster for large collections

// Detecting arrays
console.log(Array.isArray([1, 2]));  // true
console.log(Array.isArray({}));       // false
console.log([] instanceof Array);     // true
```

### Real-World Use Cases
- Creating test data arrays with `Array.from({ length }, mapFn)`
- Converting Set/Map to arrays for array methods
- Converting NodeList (DOM) to array with `Array.from()`
- Pre-allocating arrays for performance
- Creating initialization templates (grids, matrices)
- Deduplicating with Set + Array.from

### Common Mistakes
- Using `Array(3)` expecting `[3]` instead of sparse array
- Using `new Array(1, 2)` vs `Array(1, 2)` (no difference, but confusing)
- Forgetting that `Array.from()` doesn't modify the original
- Creating sparse arrays unintentionally with trailing commas
- Using `Array.isArray()` vs `instanceof Array` (works across realms)

### Best Practices
- Always use array literal `[]` over `new Array()`
- Use `Array.from()` for converting iterables to arrays
- Use `Array.of()` to avoid the single-number trap
- Use `Array.from({ length: n }, mapFn)` for initialization
- Use `Array.isArray()` for type checking (works across iframes)
- Avoid creating sparse arrays unless intentional

### Performance Considerations
- Array literals are fastest (parsed at compile time)
- `Array(n)` pre-allocation can improve push performance
- `Array.from()` with map function is optimized
- Sparse arrays can cause deoptimization in JIT
- `fill()` is faster than manual initialization loops

### Interview Questions
1. What is the difference between `Array(3)` and `Array.of(3)`?
2. How do you convert a NodeList to an array?
3. What is `Array.from()` and what are its use cases?
4. How do you create an array with n zeros?
5. What is a sparse array and how do you create one?

### Coding Challenges
1. Implement `Array.from()` polyfill.
2. Create a function that generates an array of random numbers.
3. Write a function that creates a 2D grid with a given fill value.
4. Implement a function that deduplicates an array using Array.from.

### Related Topics
- Array literal syntax
- Array methods
- Iterables and array-like objects

## Array indexing

### What It Is
Array indexing refers to accessing elements by their numeric position in the array. JavaScript arrays are zero-indexed, meaning the first element is at index 0.

### Why It Is Important
Indexing is fundamental to array access, modification, and iteration. Understanding bounds checking, negative indexing (via `at()`), and the relationship between length and indices is critical.

### How It Works Internally
Array indices are converted to string property names (`"0"`, `"1"`, etc.). The engine optimizes contiguous integer indices using "elements" storage (separate from properties). Length is automatically updated when elements are added beyond the current length.

### Syntax
```javascript
arr[index];               // Access element
arr[index] = value;        // Set element
arr.at(index);             // Access with negative indexing (ES2022)
arr.length;                // Number of elements
```

### Beginner Examples
```javascript
const fruits = ["apple", "banana", "cherry"];

// Accessing
console.log(fruits[0]);    // "apple"
console.log(fruits[1]);    // "banana"
console.log(fruits[2]);    // "cherry"
console.log(fruits[3]);    // undefined (out of bounds)

// Setting
fruits[1] = "blueberry";
console.log(fruits);       // ["apple", "blueberry", "cherry"]

// Length
console.log(fruits.length); // 3

// Last element
console.log(fruits[fruits.length - 1]); // "cherry"
console.log(fruits.at(-1));             // "cherry" (ES2022)

// Adding beyond length
fruits[5] = "elderberry";
console.log(fruits.length); // 6 (sparse)
console.log(fruits[4]);     // undefined
```

### Intermediate Examples
```javascript
// Negative indexing with at()
const numbers = [10, 20, 30, 40, 50];
console.log(numbers.at(-1));   // 50
console.log(numbers.at(-2));   // 40
console.log(numbers.at(0));    // 10
console.log(numbers.at(10));   // undefined

// Array index characteristics (they are strings internally)
const arr = ["a", "b"];
console.log(arr["0"]);         // "a" (coerced to string)
console.log("0" in arr);       // true
console.log(0 in arr);         // true (0 coered to "0")

// Length manipulation
arr.length = 5;
console.log(arr);             // ["a", "b", empty × 3]
arr.length = 1;
console.log(arr);             // ["a"] (truncated)

// Multi-dimensional indexing
const matrix = [[1, 2], [3, 4], [5, 6]];
console.log(matrix[1][0]);    // 3
console.log(matrix[2][1]);    // 6
```

### Advanced Examples
```javascript
// Non-contiguous indices (sparse arrays)
const sparse = [];
sparse[0] = "first";
sparse[100] = "last";
console.log(sparse.length);      // 101
console.log(sparse[50]);         // undefined
console.log(Object.keys(sparse)); // ["0", "100"]

// Index-based deletion
const arr = ["a", "b", "c", "d"];
delete arr[1]; // Don't do this!
console.log(arr);       // ["a", empty, "c", "d"]
console.log(arr[1]);    // undefined
console.log(1 in arr);  // false (hole)

// Proper deletion with splice
arr.splice(1, 1);
console.log(arr); // ["a", "c", "d"]

// Negative index polyfill for at()
function at(arr, index) {
  if (index >= 0) return arr[index];
  return arr[arr.length + index];
}

// Typed arrays (Uint8Array, etc.) work with indexing
const uint8 = new Uint8Array([10, 20, 30]);
console.log(uint8[1]); // 20
uint8[1] = 255;
console.log(uint8[1]); // 255
```

### Real-World Use Cases
- Accessing specific data in collections
- Implementing stacks and queues
- Matrix operations in math/graphics
- Pagination (current page data)
- Character access in strings (using array indexing on strings)
- Data table row/column access

### Common Mistakes
- Off-by-one errors: accessing `arr[arr.length]` (always undefined)
- Assuming out-of-bounds access throws an error (returns undefined)
- Using `delete arr[i]` instead of `splice` (creates sparse array)
- Confusing string index `"0"` with numeric `0` (same thing internally)
- Not accounting for sparse arrays in iteration

### Best Practices
- Use `arr[arr.length - 1]` or `arr.at(-1)` for last element
- Use `.splice()` for removing elements (not `delete`)
- Always validate index before access: `if (i >= 0 && i < arr.length)`
- Use `.at()` for cleaner negative indexing
- Avoid sparse arrays (use `undefined` values instead of holes)
- Prefer `.fill()` over setting individual indices for initialization

### Performance Considerations
- Contiguous indexed arrays are highly optimized (use "elements" storage)
- Sparse arrays fall back to dictionary mode (slower)
- Accessing out-of-bounds indices has no error but returns undefined
- Sequential index access is faster than random access due to cache locality
- Typed arrays (Uint8Array, etc.) are faster for numeric data

### Interview Questions
1. What happens when you access an array index that doesn't exist?
2. How do you get the last element of an array without knowing its length?
3. What is `arr.at()` and how does it differ from bracket notation?
4. How does setting `arr.length` affect the array?
5. What is a sparse array?

### Coding Challenges
1. Implement a function that safely accesses nested array indices.
2. Write a function that rotates array elements by index.
3. Create a function that swaps two array elements by index.
4. Implement a matrix multiplication using array indexing.

### Related Topics
- Array length property
- at() method
- Typed arrays
- Sparse arrays

## Array methods (push, pop, shift, unshift, map, filter, reduce)

### What It Is
Array methods are built-in functions that perform common operations. They are broadly categorized into mutating methods (modify the original array) and non-mutating methods (return new arrays).

### Why It Is Important
Array methods enable declarative data processing. They reduce boilerplate, improve readability, and are optimized by JavaScript engines. Mastery of these methods is essential for productive JavaScript development.

### How It Works Internally
Mutating methods modify the array's internal indexed properties and update length. Non-mutating methods create a new array by iterating and applying operations. All modern methods are implemented in native code for performance.

### Syntax
```javascript
// Mutating methods (modify original)
arr.push(...items)     // Add to end, returns new length
arr.pop()              // Remove from end, returns element
arr.unshift(...items)  // Add to start, returns new length
arr.shift()            // Remove from start, returns element
arr.splice(start, deleteCount, ...items) // General purpose
arr.sort(compareFn)    // Sort in place
arr.reverse()          // Reverse in place
arr.fill(value, start, end) // Fill with value

// Non-mutating methods (return new array)
arr.map(fn)            // Transform each element
arr.filter(fn)         // Keep elements where fn returns true
arr.reduce(fn, init)   // Reduce to single value
arr.concat(...arrays)  // Merge arrays
arr.slice(start, end)  // Extract portion
arr.flat(depth)        // Flatten nested arrays
arr.flatMap(fn)        // Map then flat(1)
```

### Beginner Examples
```javascript
// push/pop (stack operations)
const stack = [];
stack.push(1);        // [1]
stack.push(2, 3);     // [1, 2, 3]
const top = stack.pop(); // 3, stack is [1, 2]
console.log(stack, top);

// shift/unshift (queue operations)
const queue = [];
queue.push("a");      // ["a"]
queue.push("b");      // ["a", "b"]
const first = queue.shift(); // "a", queue is ["b"]
queue.unshift("z");   // ["z", "b"]

// map - transform
const numbers = [1, 2, 3, 4];
const doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8]

// filter - select
const evens = numbers.filter(n => n % 2 === 0);
console.log(evens); // [2, 4]

// reduce - accumulate
const sum = numbers.reduce((acc, n) => acc + n, 0);
console.log(sum); // 10
```

### Intermediate Examples
```javascript
// Chaining methods
const data = [1, 2, 3, 4, 5, 6];
const result = data
  .filter(n => n % 2 === 0)    // [2, 4, 6]
  .map(n => n * 10)             // [20, 40, 60]
  .reduce((acc, n) => acc + n, 0); // 120

// Complex reduce
const items = [
  { category: "fruit", name: "apple" },
  { category: "fruit", name: "banana" },
  { category: "veg", name: "carrot" }
];
const grouped = items.reduce((acc, item) => {
  (acc[item.category] = acc[item.category] || []).push(item);
  return acc;
}, {});
// { fruit: [{...}, {...}], veg: [{...}] }

// splice - add/remove anywhere
const arr = ["a", "b", "c", "d"];
arr.splice(1, 1);             // remove 1 at index 1 → ["a", "c", "d"]
arr.splice(1, 0, "x", "y");  // insert → ["a", "x", "y", "c", "d"]
arr.splice(2, 1, "z");       // replace → ["a", "x", "z", "c", "d"]

// flat and flatMap
const nested = [1, [2, [3, [4]]]];
console.log(nested.flat(2));     // [1, 2, 3, [4]]
console.log(nested.flat(Infinity)); // [1, 2, 3, 4]

const sentences = ["hello world", "foo bar"];
const words = sentences.flatMap(s => s.split(" "));
console.log(words); // ["hello", "world", "foo", "bar"]
```

### Advanced Examples
```javascript
// Reduce for complex transformations
const orders = [
  { id: 1, items: [{ price: 10 }, { price: 20 }] },
  { id: 2, items: [{ price: 30 }] },
  { id: 3, items: [{ price: 5 }, { price: 15 }, { price: 25 }] }
];

const stats = orders.reduce((acc, order) => {
  const orderTotal = order.items.reduce((s, item) => s + item.price, 0);
  acc.totalRevenue += orderTotal;
  acc.orderCount++;
  acc.maxOrder = Math.max(acc.maxOrder, orderTotal);
  acc.allItems.push(...order.items);
  return acc;
}, { totalRevenue: 0, orderCount: 0, maxOrder: 0, allItems: [] });

// Sort with custom comparator
const people = [
  { name: "Alice", age: 30 },
  { name: "Bob", age: 25 },
  { name: "Charlie", age: 35 }
];
people.sort((a, b) => a.age - b.age);
console.log(people); // [Bob(25), Alice(30), Charlie(35)]

// CopyWithin
const arr = [1, 2, 3, 4, 5];
arr.copyWithin(0, 3, 5); // [4, 5, 3, 4, 5]

// Group (ES2024)
const inventory = [
  { name: "asparagus", type: "vegetables" },
  { name: "bananas", type: "fruit" },
  { name: "broccoli", type: "vegetables" }
];
const grouped2 = Object.groupBy(inventory, item => item.type);
// { vegetables: [{asparagus}, {broccoli}], fruit: [{bananas}] }
```

### Real-World Use Cases
- API response transformation (map/filter)
- Data aggregation and statistics (reduce)
- Search/filter functionality (filter)
- Shopping cart totals (reduce)
- List rendering in React (map)
- Data deduplication (filter with indexOf)
- Sorting and ordering (sort)
- Tree/flat data conversion (flatMap)
- Pagination (slice)

### Common Mistakes
- Not returning a value from map/filter/reduce callback
- Mutating original array when expecting immutability
- Forgetting reduce needs an initial value (crashes on empty array)
- Using sort without comparator (sorts as strings!)
- Chaining too many methods affecting readability
- Using splice instead of slice (mutating vs non-mutating)

### Best Practices
- Prefer non-mutating methods (map, filter, reduce, slice)
- Always provide initial value for reduce
- Use descriptive callback names: `items.map(item => item.price)`
- Chain methods for readable data pipelines
- Use `flatMap` instead of `map` + `flat(1)`
- Use `for...of` instead of `forEach` when needing `break`/`continue`
- Extract complex callbacks to named functions

### Performance Considerations
- `for` loops are faster than array methods for large arrays
- Chaining creates intermediate arrays (but is usually fine)
- `reduce` is slightly slower than specialized methods for sum
- `sort` is O(n log n); avoid sorting large arrays frequently
- `flat(Infinity)` is slower than iterative flattening
- Modern engines heavily optimize array method chains

### Interview Questions
1. What is the difference between `map` and `forEach`?
2. How does `reduce` work and what are its use cases?
3. What is the difference between `splice` and `slice`?
4. How do you remove duplicates from an array?
5. How does `sort` work without a comparator?
6. What is `flatMap` used for?

### Coding Challenges
1. Implement `Array.prototype.map` using `reduce`.
2. Write a function that deep flattens an array.
3. Create a groupBy function using reduce.
4. Implement a ranking algorithm using sort and map.
5. Write an array intersection function using filter.

### Related Topics
- Array iteration
- Functional programming
- Immutability patterns

## Array iteration

### What It Is
Array iteration refers to the various ways to loop through array elements. JavaScript provides many iteration methods beyond the traditional `for` loop.

### Why It Is Important
Choosing the right iteration method makes code more readable, idiomatic, and less error-prone. Different methods serve different purposes and have different performance characteristics.

### How It Works Internally
Most iteration methods (forEach, map, filter, etc.) accept a callback that is called for each element. The engine's JIT may inline these calls for optimization. `for...of` uses the iterator protocol.

### Syntax
```javascript
// Traditional for loop
for (let i = 0; i < arr.length; i++) { }

// for...of (values)
for (const value of arr) { }

// for...in (indices as strings)
for (const index in arr) { }

// forEach (method)
arr.forEach((value, index, array) => { });

// Array methods (map, filter, reduce)
arr.map(fn);
arr.filter(fn);
arr.reduce(fn, init);

// entries, keys, values
for (const [index, value] of arr.entries()) { }
for (const index of arr.keys()) { }
for (const value of arr.values()) { }
```

### Beginner Examples
```javascript
const fruits = ["apple", "banana", "cherry"];

// for...of (recommended for values)
for (const fruit of fruits) {
  console.log(fruit);
}

// forEach
fruits.forEach((fruit, index) => {
  console.log(`${index}: ${fruit}`);
});

// for with index
for (let i = 0; i < fruits.length; i++) {
  console.log(fruits[i]);
}

// entries() for index/value pairs
for (const [i, fruit] of fruits.entries()) {
  console.log(i, fruit);
}
```

### Intermediate Examples
```javascript
// Performance comparison of iteration methods
const largeArray = new Array(1000000).fill(0).map((_, i) => i);

// for loop (fastest)
let sum1 = 0;
for (let i = 0; i < largeArray.length; i++) {
  sum1 += largeArray[i];
}

// for...of
let sum2 = 0;
for (const val of largeArray) {
  sum2 += val;
}

// forEach
let sum3 = 0;
largeArray.forEach(val => { sum3 += val; });

// Early termination with for...of
function findFirstEven(arr) {
  for (const num of arr) {
    if (num % 2 === 0) return num;
  }
}

// Iterating only owned indices
const sparse = [1, , , 4];
sparse.forEach((v, i) => console.log(i, v)); // 0 1, 3 4 (skips holes)
for (const v of sparse) console.log(v); // 1, undefined, undefined, 4
```

### Advanced Examples
```javascript
// Async iteration with for-await-of
async function processItems(items) {
  for await (const item of items) {
    await processItem(item);
  }
}

// Custom iterator for arrays (Symbol.iterator)
const arr = [10, 20, 30];
const iterator = arr[Symbol.iterator]();
console.log(iterator.next()); // { value: 10, done: false }
console.log(iterator.next()); // { value: 20, done: false }
console.log(iterator.next()); // { value: 30, done: false }
console.log(iterator.next()); // { value: undefined, done: true }

// Reverse iteration
for (let i = arr.length - 1; i >= 0; i--) {
  console.log(arr[i]);
}

// Breaking out of forEach (use for...of instead)
function findFirst(arr, predicate) {
  for (const item of arr) {
    if (predicate(item)) return item;
  }
  return undefined;
}

// Some and every (short-circuit iteration)
const hasEven = [1, 3, 5, 7, 8].some(n => n % 2 === 0); // true (stops at 8)
const allPositive = [1, 2, 3, -4, 5].every(n => n > 0); // false (stops at -4)
```

### Real-World Use Cases
- Rendering lists in UI frameworks (React map)
- Data transformation pipelines
- Search/filter operations
- Validation (every, some)
- Async batch processing
- CSV/JSON data processing

### Common Mistakes
- Using `for...in` for arrays (iterates enumerable properties too)
- Using `forEach` when you need `break`/`continue`
- Forgetting that `forEach` doesn't work with async/await (use `for...of`)
- Modifying array during iteration (unpredictable results)
- Not accounting for sparse arrays in iteration

### Best Practices
- Use `for...of` as default iteration method
- Use `map` for transformation, `filter` for selection, `reduce` for accumulation
- Use `forEach` only for side effects with no early termination needed
- Use `some`/`every` for boolean checks with short-circuit
- Use `for` loop when index access or performance is critical
- Use `for...of` with `entries()` when both index and value needed

### Performance Considerations
- Traditional `for` loops are fastest across all engines
- `for...of` is slightly slower than `for` but more readable
- `forEach` is slower than `for...of` for large arrays
- `map`, `filter`, `reduce` create intermediate arrays
- Chaining multiple methods has overhead but is usually negligible
- Short-circuit methods (some, every) can be much faster for early exits

### Interview Questions
1. What is the difference between `for...of` and `for...in`?
2. How do you break out of a `forEach` loop?
3. What iteration method would you use for async operations?
4. How do you iterate an array backwards?
5. What is the fastest way to iterate an array in JavaScript?

### Coding Challenges
1. Implement a custom `forEach` function with early termination support.
2. Write a function that iterates two arrays in parallel.
3. Create a lazy iterator that generates array values on demand.
4. Implement `Array.prototype.groupBy` using iteration.
5. Write a batched iteration function that processes items in chunks.

### Related Topics
- For loops
- For...of loop
- Iterator protocol
- Async iteration
