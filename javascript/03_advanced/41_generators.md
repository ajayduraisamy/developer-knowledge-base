# Generators

## Introduction

Generators are special functions that can pause execution and resume later, maintaining their state between pauses. They are defined with `function*` and use `yield` to produce values. Generators provide a powerful way to work with sequences of data, implement iterators, manage asynchronous flows, and create infinite sequences.

---

## function* declaration

### What It Is

The `function*` declaration defines a generator function that returns a Generator object. Unlike regular functions, generator functions can be paused in the middle of execution using the `yield` keyword and resumed later, making them ideal for lazy evaluation and asynchronous control flow.

### Why It Is Important

Generator functions enable lazy evaluation (values computed on demand), infinite sequences (no memory limit), custom iteration logic, asynchronous flow control (before async/await), and cooperative multitasking via yielding control.

### How It Works Internally

When a generator function is called, it does not execute the body. Instead, it returns a Generator object that conforms to the iterator protocol. The generator body executes only when `.next()` is called. Each `yield` suspends execution and saves the state. The engine maintains the generator's execution context in a suspended state until the next `.next()` call.

### Syntax

```javascript
function* generatorName() { yield value1; yield value2; return finalValue; }
const gen = function* () { yield 1; };
const obj = { *genMethod() { yield 1; } };
class MyClass { *generator() { yield 1; } }
```

### Beginner Examples

```javascript
function* simpleGenerator() {
  yield 1; yield 2; yield 3;
}
const gen = simpleGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }
for (const value of simpleGenerator()) { console.log(value); }
console.log([...simpleGenerator()]); // [1, 2, 3]
```

### Intermediate Examples

```javascript
function* range(start, end, step = 1) {
  for (let i = start; i <= end; i += step) yield i;
}
console.log([...range(1, 10, 2)]); // [1, 3, 5, 7, 9]

function* fibonacci(n) {
  let a = 0, b = 1, count = 0;
  while (count < n) { yield a; [a, b] = [b, a + b]; count++; }
}
console.log([...fibonacci(10)]); // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

### Advanced Examples

```javascript
function* treeTraversal(node) {
  yield node.value;
  if (node.left) yield* treeTraversal(node.left);
  if (node.right) yield* treeTraversal(node.right);
}

function* safeGenerator() {
  try { yield 1; yield 2; } catch (err) { console.error("Caught:", err); }
  yield 3;
}
const gen = safeGenerator();
gen.next();
gen.throw(new Error("Oops"));
```

### Real-World Use Cases

Custom iterators, pagination (yield pages on demand), infinite sequences, async flow control, Redux Saga middleware.

### Common Mistakes

Forgetting `*` in function declaration, trying to `yield` inside regular functions, calling `next()` on exhausted generator.

### Best Practices

Use generators for lazy sequences, combine with `for...of`, use `yield*` for delegation, handle early termination with `try/finally`.

### Performance Considerations

Generators have memory overhead for suspended state but are more memory-efficient than arrays for large sequences.

### Interview Questions

1. What is a generator function? — Function that can pause and resume, returning a Generator.
2. Difference between `yield` and `return`? — `yield` pauses and produces, `return` ends.
3. Can generators receive values? — Yes, via `gen.next(value)`.

### Coding Challenges

1. Implement an infinite ID generator.
2. Create a generator for paginated API data.
3. Build a generator that yields prime numbers.

### Related Topics

`yield` keyword, `next()` method, `yield*` delegation, async generators, iterators

---

## yield keyword

### What It Is

The `yield` keyword pauses generator execution and returns a value to the caller. It can also receive values from the caller when used with `gen.next(value)`.

### Why It Is Important

`yield` enables two-way communication between the generator and caller, lazy value production, and cooperative multitasking.

### How It Works Internally

When `yield expr` is executed, the expression is evaluated, the value is wrapped in `{value, done}`, execution is suspended, and control returns to the caller.

### Syntax

```javascript
function* gen() {
  const received = yield "sent";
  yield received;
}
const g = gen();
console.log(g.next()); // { value: "sent", done: false }
console.log(g.next("hello")); // { value: "hello", done: false }
```

### Beginner Examples

```javascript
function* counter() {
  let count = 0;
  while (true) {
    const increment = yield count;
    if (increment !== undefined) count += increment;
    else count++;
  }
}
const c = counter();
console.log(c.next().value); // 0
console.log(c.next().value); // 1
console.log(c.next(5).value); // 6
```

### Intermediate Examples

```javascript
function* interactive() {
  const name = yield "What is your name?";
  const age = yield `Hello ${name}, how old are you?`;
  yield `${name} is ${age} years old`;
}
const g = interactive();
console.log(g.next().value);
console.log(g.next("Alice").value);
console.log(g.next(30).value);
```

### Real-World Use Cases

Interactive command prompts, coroutines, data processing pipelines, state machines.

### Common Mistakes

Not capturing the return value of `yield`, confusing `yield` with `return`, using `yield` inside callbacks.

### Best Practices

Use `yield` for producing sequence values, capture yielded values for two-way communication, use `yield*` for delegation.

### Performance Considerations

Each `yield` creates a new `{value, done}` object. Minimal overhead for typical usage.

### Interview Questions

1. What does `yield` do? — Pauses execution and produces a value.
2. Can `yield` receive values? — Yes, via `gen.next(value)`.
3. Difference between `yield` and `yield*`? — `yield*` delegates to another generator.

### Coding Challenges

1. Create a two-way communication generator for a quiz.
2. Build an interactive REPL using generators.

### Related Topics

`function*`, `next()`, `yield*`, generator delegation

---

## next() method

### What It Is

The `next()` method is called on a Generator object to resume its execution. It returns an object with `value` (the yielded value) and `done` (boolean). An optional argument becomes the result of the last `yield`.

### Why It Is Important

`next()` is the interface for interacting with generators, controlling execution flow and passing values.

### How It Works Internally

Calling `.next(arg)` resumes the generator from the suspension point. The `arg` replaces the `yield` expression. The generator runs until the next `yield`, `return`, or completion.

### Syntax

```javascript
const gen = generatorFunction();
const result1 = gen.next();
const result2 = gen.next(argument);
```

### Beginner Examples

```javascript
function* numbers() { yield 1; yield 2; return 3; }
const gen = numbers();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: true }
```

### Intermediate Examples

```javascript
function* runningTotal() {
  let total = 0;
  while (true) {
    const value = yield total;
    total += value ?? 0;
  }
}
const rt = runningTotal();
rt.next();
console.log(rt.next(5).value); // 5
console.log(rt.next(10).value); // 15
```

### Common Mistakes

Calling `next()` without priming the generator first, ignoring the `done` flag.

### Best Practices

Check `done` in manual iteration, use `for...of` when you don't need two-way communication.

### Performance Considerations

Each `.next()` call is a function call with minimal overhead.

### Interview Questions

1. What does `next()` return? — `{ value: any, done: boolean }`.
2. Can you pass a value to the first `next()` call? — Yes, but it is ignored if no `yield` has been reached.

### Coding Challenges

1. Create a generator state machine driven by `next()`.
2. Implement a coroutine system using generators.

### Related Topics

`yield`, generator iteration, `return()` and `throw()` methods

---

## yield* delegation

### What It Is

The `yield*` expression delegates iteration to another generator or iterable object, yielding each value before continuing.

### Why It Is Important

`yield*` enables generator composition, recursive algorithms, flattening nested generators, and reusing generator logic.

### How It Works Internally

`yield*` opens the delegated iterator and yields each of its values, forwarding `next()`, `return()`, and `throw()` calls.

### Syntax

```javascript
function* inner() { yield 1; yield 2; return 3; }
function* outer() { const result = yield* inner(); yield result; }
```

### Beginner Examples

```javascript
function* a() { yield 1; yield 2; }
function* b() { yield 3; yield 4; }
function* combined() { yield* a(); yield* b(); }
console.log([...combined()]); // [1, 2, 3, 4]
```

### Intermediate Examples

```javascript
function* flatten(...arrays) { for (const arr of arrays) yield* arr; }
console.log([...flatten([1,2], [3,4], [5,6])]); // [1,2,3,4,5,6]

function* treeTraversal(tree) {
  if (tree.left) yield* treeTraversal(tree.left);
  yield tree.value;
  if (tree.right) yield* treeTraversal(tree.right);
}
```

### Advanced Examples

```javascript
function* map(iterable, fn) { for (const v of iterable) yield fn(v); }
function* filter(iterable, pred) { for (const v of iterable) if (pred(v)) yield v; }
function* take(iterable, n) { let c = 0; for (const v of iterable) { if (c >= n) return; yield v; c++; } }
const result = [...take(filter(map([1,2,3,4,5], x => x*2), x => x > 5), 3)];
console.log(result); // [6, 8, 10]
```

### Real-World Use Cases

Lazy data transformation pipelines, tree/graph traversal, composing generators.

### Common Mistakes

Forgetting `*` in `yield*`, not handling returned value, deep recursion exceeding stack.

### Best Practices

Use `yield*` for composing generators and recursive algorithms. Capture return values when needed.

### Performance Considerations

Each `yield*` adds a delegation layer but is highly optimized in modern engines.

### Interview Questions

1. What is `yield*`? — Delegates iteration to another generator/iterable.
2. Can `yield*` be used with arrays? — Yes, any iterable.

### Coding Challenges

1. Create a lazy `map`/`filter`/`reduce` pipeline with `yield*`.
2. Implement a directory tree walker using recursive `yield*`.

### Related Topics

`yield`, iterators, `Symbol.iterator`, generator composition

---

## Async generators

### What It Is

Async generators combine `async function*` with `yield`. They produce promises and are consumed with `for await...of`. Each value may require waiting for an async operation.

### Why It Is Important

Async generators enable streaming async data, lazy async sequences, paginated API consumption, real-time data processing, and async data pipelines.

### How It Works Internally

Each `yield` in an async generator returns a Promise. The `for await...of` loop awaits each promise. The generator suspends between yields.

### Syntax

```javascript
async function* asyncGen() {
  const r1 = await fetch("/api/1"); yield r1;
  const r2 = await fetch("/api/2"); yield r2;
}
for await (const v of asyncGen()) { console.log(v); }
```

### Beginner Examples

```javascript
async function* generatePages(url, total) {
  for (let p = 1; p <= total; p++) {
    const res = await fetch(`${url}?page=${p}`);
    yield await res.json();
  }
}
for await (const page of generatePages("/api/users", 3)) { console.log(page); }
```

### Intermediate Examples

```javascript
async function* streamLines(url) {
  const res = await fetch(url);
  const reader = res.body.getReader();
  const decoder = new TextDecoder();
  let buffer = "";
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    buffer += decoder.decode(value, { stream: true });
    const lines = buffer.split("\n");
    buffer = lines.pop() || "";
    for (const line of lines) yield line;
  }
  if (buffer) yield buffer;
}
for await (const line of streamLines("/data.txt")) { console.log(line); }
```

### Real-World Use Cases

Streaming file processing, paginated API consumption, real-time data feeds, CSV/JSON streaming parsers.

### Common Mistakes

Forgetting `async` before `function*`, using `for...of` instead of `for await...of`.

### Best Practices

Use `try/catch` inside async generators, combine with AbortController for cancellation.

### Performance Considerations

Each iteration awaits a promise. I/O-bound, not CPU-bound. Does not block the event loop.

### Interview Questions

1. What is an async generator? — An async function that yields promises, consumed with `for await...of`.
2. Can you use `yield*` in async generators? — Yes, with async iterables.

### Coding Challenges

1. Create an async generator for paginated API data.
2. Build a CSV file streaming parser.
3. Implement an async data transformation pipeline.

### Related Topics

Async/await, iterators, `for await...of`, streams, `Symbol.asyncIterator`
