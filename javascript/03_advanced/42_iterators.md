# Iterators

## Introduction

Iterators are objects that define a sequence and a mechanism for traversing it. They implement the Iterator protocol by providing a `next()` method that returns `{value, done}` objects. The Iterable protocol, defined by `Symbol.iterator`, allows objects to be iterated using `for...of`, spread, and other language features. Together, they form the foundation for custom iteration in JavaScript.

---

## Symbol.iterator

### What It Is

`Symbol.iterator` is a well-known Symbol that defines the default iterator for an object. When an object implements the `Symbol.iterator` method, it becomes iterable and can be used with `for...of`, spread operator (`...`), `Array.from()`, and destructuring.

```javascript
const iterable = {
  [Symbol.iterator]: function* () {
    yield 1;
    yield 2;
    yield 3;
  },
};

console.log([...iterable]); // [1, 2, 3]
for (const value of iterable) {
  console.log(value); // 1, 2, 3
}
```

### Why It Is Important

`Symbol.iterator` is the bridge between custom objects and JavaScript's iteration protocols:
- It enables custom objects to work with `for...of`, spread, destructuring.
- It is the foundation for making any object iterable.
- Built-in types (Array, Map, Set, String) already implement it.
- It allows lazy evaluation through generator-based implementations.

### How It Works Internally

When JavaScript encounters `for...of`, spread, or `Array.from()` on an object, it:
1. Calls `obj[Symbol.iterator]()` to get an iterator object.
2. Repeatedly calls `iterator.next()` to get `{value, done}`.
3. When `done` is `true`, iteration stops.

The engine caches the iterator method for performance.

### Syntax

```javascript
// Making an object iterable
const obj = {
  [Symbol.iterator]: function () {
    // return an iterator object with next()
    return {
      next: function () {
        return { value: ..., done: boolean };
      },
    };
  },
};

// Using a generator for simplicity
const obj = {
  *[Symbol.iterator]() {
    yield 1;
    yield 2;
  },
};
```

### Beginner Examples

```javascript
// Example 1: Simple iterable range
const range = {
  from: 1,
  to: 5,
  [Symbol.iterator]() {
    let current = this.from;
    const last = this.to;
    return {
      next() {
        if (current <= last) {
          return { value: current++, done: false };
        }
        return { value: undefined, done: true };
      },
    };
  },
};

console.log([...range]); // [1, 2, 3, 4, 5]

// Example 2: Built-in iterables
console.log([..."Hello"]); // ['H', 'e', 'l', 'l', 'o']
console.log([...new Set([1, 2, 3])]); // [1, 2, 3]
console.log([...new Map([['a', 1], ['b', 2]])]); // [['a', 1], ['b', 2]]

// Example 3: Array destructuring with custom iterable
const [first, second] = range;
console.log(first, second); // 1 2
```

### Intermediate Examples

```javascript
// Example 1: Iterable with generator
const fibonacciIterable = {
  [Symbol.iterator]: function* () {
    let a = 0, b = 1;
    while (true) {
      yield a;
      [a, b] = [b, a + b];
    }
  },
};

// Take first 10 fibonacci numbers
let count = 0;
for (const n of fibonacciIterable) {
  if (count++ >= 10) break;
  console.log(n);
}

// Example 2: Paginated API iterable
function createPaginatedIterable(url) {
  return {
    [Symbol.iterator]: () => ({
      page: 1,
      hasMore: true,
      next() {
        if (!this.hasMore) return { done: true };
        const data = fetchData(url, this.page); // sync simulation
        this.page++;
        this.hasMore = data.hasMore;
        return { value: data.items, done: false };
      },
    }),
  };
}

// Example 3: Iterable with cleanup
const resourceIterable = {
  [Symbol.iterator]() {
    let resource = acquire();
    return {
      next() {
        const value = resource.read();
        return value ? { value, done: false } : { done: true };
      },
      return() {
        resource.release();
        return { done: true };
      },
    };
  },
};
```

### Advanced Examples

```javascript
// Example 1: Lazy transformation on iterables
class LazyArray {
  constructor(array) {
    this.array = array;
  }

  map(fn) {
    const self = this;
    return {
      *[Symbol.iterator]() {
        for (const item of self.array) {
          yield fn(item);
        }
      },
    };
  }

  filter(pred) {
    const self = this;
    return {
      *[Symbol.iterator]() {
        for (const item of self.array) {
          if (pred(item)) yield item;
        }
      },
    };
  }

  take(n) {
    const self = this;
    return {
      *[Symbol.iterator]() {
        let count = 0;
        for (const item of self.array) {
          if (count++ >= n) return;
          yield item;
        }
      },
    };
  }
}

const lazy = new LazyArray([1, 2, 3, 4, 5, 6, 7, 8]);
const result = [...lazy.filter(x => x % 2 === 0).map(x => x * 10).take(3)];
console.log(result); // [20, 40, 60]

// Example 2: Matrix iteration (row-major and column-major)
class Matrix {
  constructor(rows, cols, data) {
    this.rows = rows;
    this.cols = cols;
    this.data = data;
  }

  [Symbol.iterator]() {
    return this.rowsMajor();
  }

  *rowsMajor() {
    for (let r = 0; r < this.rows; r++) {
      for (let c = 0; c < this.cols; c++) {
        yield this.data[r * this.cols + c];
      }
    }
  }

  *columnsMajor() {
    for (let c = 0; c < this.cols; c++) {
      for (let r = 0; r < this.rows; r++) {
        yield this.data[r * this.cols + c];
      }
    }
  }
}

// Example 3: Checking if something is iterable
function isIterable(obj) {
  return obj != null && typeof obj[Symbol.iterator] === "function";
}
console.log(isIterable([1,2])); // true
console.log(isIterable("hello")); // true
console.log(isIterable(42)); // false
console.log(isIterable(null)); // false
```

### Real-World Use Cases

- **Database cursors** — Iterating over query results lazily.
- **File reading** — Reading a file line-by-line.
- **Paginated APIs** — Abstracting pagination into iteration.
- **Tree/DOM traversal** — Walking a tree structure.
- **Lazy evaluation** — Computing values only when needed.

### Common Mistakes

- **Not returning an iterator object** — `Symbol.iterator` must return an object with `next()`.
- **Forgetting to return `{ done: true }`** — Infinite loops in `for...of`.
- **Mutating the object during iteration** — Leads to undefined behavior.
- **Assuming all objects are iterable** — Plain objects are NOT iterable by default.
- **Not implementing `return()` for cleanup** — Resource leaks if iteration stops early.

### Best Practices

- Use generators (`*[Symbol.iterator]()`) for simpler iterator implementation.
- Implement `return()` for cleanup (closing files, releasing connections).
- Keep iterators stateless when possible — delegate to generators.
- Use `for...of` instead of manual `.next()` calls for readability.
- Cache iterable checks by storing `Symbol.iterator` lookup.

### Performance Considerations

- Iterator objects are created once per iteration start (cheap).
- Each `.next()` call is a function call — micro-optimization for hot paths.
- Generator-based iterators have slight overhead compared to manual iterators.
- V8 optimizes built-in iterators (Array, Map) heavily.
- Lazy iteration through iterables saves memory for large datasets.

### Interview Questions

1. **What is `Symbol.iterator`?**
   A well-known Symbol that defines the default iterator for an object, making it iterable.

2. **What built-in types have `Symbol.iterator`?**
   Array, String, Map, Set, NodeList, TypedArray, arguments.

3. **What happens if you use `for...of` on a non-iterable object?**
   A TypeError is thrown: "obj is not iterable".

4. **How do you make a plain object iterable?**
   Define a `[Symbol.iterator]` method that returns an iterator.

### Coding Challenges

1. **Make an object iterable that yields its own property names.**
2. **Create an iterable that generates an infinite sequence of even numbers.**
3. **Implement a `take(n)` function that limits iteration.**
4. **Build an iterable wrapper around a WebSocket for message streaming.**

### Related Topics

- Iterator protocol (`next()`)
- Iterable protocol
- Generators
- `for...of` loop
- Spread operator
- `Array.from()`
- Destructuring

---

## Iterator protocol (next())

### What It Is

The Iterator protocol defines a standard way to produce a sequence of values. An object is an iterator when it implements a `next()` method that returns an object with two properties: `value` (any value) and `done` (boolean indicating whether iteration is complete).

```javascript
const iterator = {
  current: 0,
  next() {
    if (this.current < 3) {
      return { value: this.current++, done: false };
    }
    return { value: undefined, done: true };
  },
};

console.log(iterator.next()); // { value: 0, done: false }
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

### Why It Is Important

The Iterator protocol is the low-level interface that powers all iteration in JavaScript:
- It standardizes how sequences are produced and consumed.
- It enables lazy (on-demand) value production.
- It is the basis for the Iterable protocol and `for...of`.
- It allows manual control over iteration progress.
- It supports early termination via `return()`.

### How It Works Internally

When `for...of` or spread uses an iterator, the engine:
1. Calls `iterator.next()` to get `result`.
2. If `result.done` is `true`, stops.
3. Otherwise, uses `result.value` and continues.

The engine may also call `iterator.return()` if iteration stops early (break, throw, return).

### Syntax

```javascript
const iterator = {
  next() {
    return { value: any, done: boolean };
  },
  // Optional:
  return() {
    // cleanup
    return { done: true };
  },
  throw(error) {
    throw error;
  },
};
```

### Beginner Examples

```javascript
// Example 1: Manual iteration
function createCounter(max) {
  let count = 0;
  return {
    next() {
      if (count < max) {
        return { value: count++, done: false };
      }
      return { value: undefined, done: true };
    },
  };
}

const counter = createCounter(3);
let result;
while (!(result = counter.next()).done) {
  console.log(result.value); // 0, 1, 2
}

// Example 2: Infinite iterator
const infinite = {
  current: 0,
  next() {
    return { value: this.current++, done: false };
  },
};

// Example 3: Iterator with return for cleanup
const fileReader = {
  file: open("data.txt"),
  next() {
    const line = this.file.readLine();
    if (line === null) return { done: true };
    return { value: line, done: false };
  },
  return() {
    this.file.close();
    return { done: true };
  },
};
```

### Intermediate Examples

```javascript
// Example 1: Iterator with early termination
function createTaskRunner(tasks) {
  let index = 0;
  return {
    next() {
      if (index >= tasks.length) return { done: true };
      const task = tasks[index++];
      return { value: task(), done: false };
    },
    return() {
      console.log(`Cleaning up. ${tasks.length - index} tasks remaining.`);
      index = tasks.length;
      return { done: true };
    },
  };
}

const tasks = [
  () => "Task 1",
  () => "Task 2",
  () => "Task 3",
];

const runner = createTaskRunner(tasks);
console.log(runner.next().value); // "Task 1"
runner.return(); // Cleanup: 2 tasks remaining

// Example 2: Bidirectional iterator
const bidirectional = {
  data: [1, 2, 3, 4, 5],
  index: 0,
  next() {
    if (this.index < this.data.length) {
      return { value: this.data[this.index++], done: false };
    }
    return { done: true };
  },
  prev() {
    if (this.index > 0) {
      return { value: this.data[--this.index], done: false };
    }
    return { done: true };
  },
};

// Example 3: Iterator throwing
const safeIterator = {
  index: 0,
  max: 5,
  next() {
    if (this.index >= this.max) return { done: true };
    if (this.index === 3) throw new Error("Error at index 3");
    return { value: this.index++, done: false };
  },
};
```

### Advanced Examples

```javascript
// Example 1: Composable iterator transformations
function mapIterator(iterator, fn) {
  return {
    next() {
      const result = iterator.next();
      if (result.done) return result;
      return { value: fn(result.value), done: false };
    },
    return() {
      return iterator.return ? iterator.return() : { done: true };
    },
  };
}

function filterIterator(iterator, pred) {
  return {
    next() {
      let result = iterator.next();
      while (!result.done) {
        if (pred(result.value)) return result;
        result = iterator.next();
      }
      return result;
    },
    return() {
      return iterator.return ? iterator.return() : { done: true };
    },
  };
}

const base = createCounter(10);
const mapped = mapIterator(base, x => x * 2);
const filtered = filterIterator(mapped, x => x > 10);

console.log(filtered.next().value); // 12
console.log(filtered.next().value); // 14

// Example 2: Async-safe callback to iterator
function eventStream(element, eventName) {
  let handlers = [];
  let paused = [];

  element.addEventListener(eventName, handler);

  function handler(event) {
    if (paused.length > 0) {
      paused.shift()(event);
    } else {
      handlers.push(event);
    }
  }

  return {
    next() {
      if (handlers.length > 0) {
        return { value: handlers.shift(), done: false };
      }
      return new Promise(resolve => {
        paused.push(event => resolve({ value: event, done: false }));
      });
    },
    return() {
      element.removeEventListener(eventName, handler);
      handlers = [];
      paused = [];
      return { done: true };
    },
  };
}
```

### Real-World Use Cases

- **Stream processing** — Transform streams using iterators.
- **Database queries** — Iterate over query result sets.
- **Data serialization** — Iterate over parsed tokens.
- **Event processing** — Process events sequentially.
- **Custom data structures** — Tree, graph, linked list traversal.

### Common Mistakes

- **Returning `{ done: true }` without `value`** — Works but `value` is undefined.
- **Not resetting state between iterations** — Same iterator can't be reused (create a new one).
- **Side effects in `next()`** — Makes iteration non-deterministic.
- **Forgetting `return()` for cleanup** — Resources leak.
- **Returning non-object from `next()`** — Must return an object.

### Best Practices

- Always return an object with `value` and `done` properties.
- Implement `return()` for cleanup (files, connections, listeners).
- Create a new iterator per iteration (for iterables).
- Use generators instead of manual iterators when possible.
- Handle errors gracefully inside `next()`.

### Performance Considerations

- Each `next()` call is a function call — very fast.
- V8 can inline small `next()` functions in hot paths.
- Creating a new iterator object per iteration start is cheap.
- `return()` should be fast — avoid heavy cleanup that blocks iteration.
- Iterators with many `next()` calls (millions) should be optimized.

### Interview Questions

1. **What must an object implement to be an iterator?**
   A `next()` method that returns `{ value: any, done: boolean }`.

2. **What is the difference between an iterator and an iterable?**
   An iterator has `next()`. An iterable has `Symbol.iterator` that returns an iterator.

3. **What is the `return()` method for?**
   Optional cleanup when iteration stops early (break, throw, return).

4. **Can an iterator also be iterable?**
   Yes, by also implementing `Symbol.iterator` that returns `this`.

### Coding Challenges

1. **Create an iterator for a linked list.**
2. **Implement an iterator that yields values from two arrays in alternating order.**
3. **Build a `zip` iterator that combines two iterators.**
4. **Create an iterator that splits a string by a delimiter.**

### Related Topics

- `Symbol.iterator`
- Iterable protocol
- Generators
- `for...of`
- `return()`/`throw()` methods
- Async iterators

---

## Iterable protocol

### What It Is

The Iterable protocol allows objects to define their iteration behavior. An object is iterable if it implements the `[Symbol.iterator]()` method that returns an iterator. Built-in iterables include Array, String, Map, Set, and TypedArray. Custom objects can be made iterable by implementing this protocol.

```javascript
const iterable = {
  data: [10, 20, 30],
  [Symbol.iterator]() {
    let index = 0;
    const data = this.data;
    return {
      next() {
        if (index < data.length) {
          return { value: data[index++], done: false };
        }
        return { done: true };
      },
    };
  },
};

for (const item of iterable) {
  console.log(item); // 10, 20, 30
}
```

### Why It Is Important

The Iterable protocol is the standard interface that the JavaScript language uses for:
- `for...of` loops
- Spread operator (`[...iterable]`)
- `Array.from(iterable)`
- Destructuring (`[a, b] = iterable`)
- `Promise.all(iterable)` and `Promise.race(iterable)`
- `Map`, `Set` constructors
- `yield*` delegation

### How It Works Internally

When JavaScript encounters an iterable, it:
1. Calls `iterable[Symbol.iterator]()` to obtain an iterator.
2. Calls `iterator.next()` repeatedly.
3. Stops when `done` is `true`.
4. If iteration stops early, calls `iterator.return()` if available.

The `Symbol.iterator` method must return a fresh iterator each time it is called.

### Syntax

```javascript
// Object is iterable if it has [Symbol.iterator]
const iterable = {
  [Symbol.iterator]() {
    return iteratorObject;
  },
};

// Generator function shorthand
const iterable = {
  *[Symbol.iterator]() {
    yield 1;
    yield 2;
  },
};
```

### Beginner Examples

```javascript
// Example 1: Simple range iterable
class Range {
  constructor(start, end) {
    this.start = start;
    this.end = end;
  }

  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    return {
      next() {
        if (current <= end) {
          return { value: current++, done: false };
        }
        return { done: true };
      },
    };
  }
}

for (const n of new Range(1, 5)) {
  console.log(n); // 1, 2, 3, 4, 5
}
console.log([...new Range(1, 5)]); // [1, 2, 3, 4, 5]

// Example 2: String is iterable
for (const char of "hello") {
  console.log(char); // 'h', 'e', 'l', 'l', 'o'
}

// Example 3: Set and Map are iterable
const set = new Set([1, 2, 3]);
console.log([...set]); // [1, 2, 3]

const map = new Map([['a', 1], ['b', 2]]);
for (const [key, value] of map) {
  console.log(key, value);
}
```

### Intermediate Examples

```javascript
// Example 1: Iterable class with multiple iteration strategies
class Matrix {
  constructor(data) {
    this.data = data; // [[1,2],[3,4]]
  }

  // Default iteration (row-major)
  [Symbol.iterator]() {
    return this.rowMajor();
  }

  *rowMajor() {
    for (const row of this.data) {
      for (const cell of row) {
        yield cell;
      }
    }
  }

  *columnMajor() {
    for (let col = 0; col < this.data[0].length; col++) {
      for (let row = 0; row < this.data.length; row++) {
        yield this.data[row][col];
      }
    }
  }
}

const mat = new Matrix([[1, 2], [3, 4]]);
console.log([...mat]); // [1, 2, 3, 4]
console.log([...mat.columnMajor()]); // [1, 3, 2, 4]

// Example 2: Chainable iterable operations
class ChainableIterable {
  constructor(iterable) {
    this.iterable = iterable;
  }

  [Symbol.iterator]() {
    return this.iterable[Symbol.iterator]();
  }

  map(fn) {
    const self = this;
    return new ChainableIterable({
      *[Symbol.iterator]() {
        for (const item of self) {
          yield fn(item);
        }
      },
    });
  }

  filter(pred) {
    const self = this;
    return new ChainableIterable({
      *[Symbol.iterator]() {
        for (const item of self) {
          if (pred(item)) yield item;
        }
      },
    });
  }

  take(n) {
    const self = this;
    return new ChainableIterable({
      *[Symbol.iterator]() {
        let count = 0;
        for (const item of self) {
          if (count++ >= n) return;
          yield item;
        }
      },
    });
  }
}

const chain = new ChainableIterable([1, 2, 3, 4, 5, 6]);
const result = chain
  .filter(x => x % 2 === 0)
  .map(x => x * 10)
  .take(2);

console.log([...result]); // [20, 40]
```

### Advanced Examples

```javascript
// Example 1: Async iterable
const asyncIterable = {
  data: [1, 2, 3],
  [Symbol.asyncIterator]() {
    let index = 0;
    const data = this.data;
    return {
      async next() {
        await new Promise(r => setTimeout(r, 100));
        if (index < data.length) {
          return { value: data[index++], done: false };
        }
        return { done: true };
      },
    };
  },
};

(async () => {
  for await (const value of asyncIterable) {
    console.log(value); // 1, 2, 3 (with 100ms delay each)
  }
})();

// Example 2: Iterable that wraps another iterable with logging
function loggedIterable(iterable) {
  return {
    [Symbol.iterator]() {
      const iterator = iterable[Symbol.iterator]();
      return {
        next() {
          const result = iterator.next();
          console.log(`Iterated: ${JSON.stringify(result)}`);
          return result;
        },
        return() {
          console.log("Iteration ended early");
          return iterator.return ? iterator.return() : { done: true };
        },
      };
    },
  };
}

// Example 3: Infinite iterable with break
function randomNumbers(seed) {
  let state = seed;
  return {
    [Symbol.iterator]() {
      return {
        next() {
          state = (state * 1103515245 + 12345) & 0x7fffffff;
          return { value: state, done: false };
        },
      };
    },
  };
}

let count = 0;
for (const n of randomNumbers(42)) {
  console.log(n);
  if (count++ > 5) break;
}
```

### Real-World Use Cases

- **Custom collection classes** — Making custom data structures work with language features.
- **Data loading** — Iterating over paginated API results.
- **File processing** — Reading a file as lines via an iterable.
- **Event streams** — Converting DOM events into iterables.
- **ORM query results** — Iterating over database result sets.

### Common Mistakes

- **Not returning a fresh iterator each time** — Iterable can only be iterated once.
- **Forgetting `Symbol.iterator` returns an iterator** — It returns an object, not the iterator directly.
- **Using plain objects as iterables** — They are not iterable by default.
- **Modifying the iterable during iteration** — Leads to undefined behavior.
- **Not implementing `return()` for cleanup** — Iteration may stop early.

### Best Practices

- Use generators for implementing `[Symbol.iterator]()`.
- Return a fresh iterator each time `[Symbol.iterator]()` is called.
- Implement cleanup in `return()` if iteration can stop early.
- Make your own collection classes iterable to integrate with the language.
- Document whether iteration produces copies or references.

### Performance Considerations

- Each `for...of` loop calls `Symbol.iterator()` once (creates one iterator object).
- V8 optimizes built-in iterables (Array, Map) with fast paths.
- Generator-based iterables have slight overhead over manual iterators.
- Lazy iteration via iterables saves memory for large sequences.
- `...spread` on large iterables creates an array — memory intensive.

### Interview Questions

1. **What makes an object iterable?**
   Implementing the `[Symbol.iterator]()` method that returns an iterator.

2. **What is the difference between an iterable and an iterator?**
   An iterable has `Symbol.iterator` that returns an iterator. An iterator has `next()` that returns `{value, done}`.

3. **Can you iterate over a plain object?**
   No, but you can use `Object.keys()`, `Object.values()`, or `Object.entries()` to get iterable arrays.

4. **What happens if `Symbol.iterator` returns a non-object?**
   A TypeError is thrown.

### Coding Challenges

1. **Make a `LinkedList` class iterable.**
2. **Create an iterable that produces the Collatz sequence.**
3. **Implement a `zip` function that combines two iterables into one.**
4. **Build an iterable that memoizes values from a generator.**
5. **Create an iterable wrapper that adds progress logging.**

### Related Topics

- `Symbol.iterator`
- Iterator protocol (`next()`)
- `for...of` loop
- Generators
- `Symbol.asyncIterator`
- `Array.from()`

---

## Custom iterables

### What It Is

Custom iterables are user-defined objects that implement the Iterable protocol. They allow developers to create objects that can be used with JavaScript's iteration constructs (`for...of`, spread, destructuring) for domain-specific traversal logic.

### Why It Is Important

Custom iterables allow you to:
- Integrate your data structures with the language's iteration syntax.
- Provide domain-specific traversal (tree walking, graph traversal, pagination).
- Implement lazy evaluation for performance.
- Create fluent, chainable APIs.
- Abstract away complex iteration logic.

### How It Works Internally

Same as the Iterable protocol — define `[Symbol.iterator]()` on your object. The engine calls it to get an iterator, then calls `next()` on the iterator. You have full control over the iteration logic.

### Syntax

```javascript
// Using an object
const custom = {
  [Symbol.iterator]() {
    // return iterator
  },
};

// Using a class
class CustomCollection {
  [Symbol.iterator]() {
    // return iterator
  }
}

// Using a factory function
function createCustomIterable(...args) {
  return {
    [Symbol.iterator]() {
      // return iterator
    },
  };
}
```

### Beginner Examples

```javascript
// Example 1: Custom iterable for a deck of cards
const suits = ['♠', '♥', '♦', '♣'];
const ranks = ['A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K'];

const deck = {
  [Symbol.iterator]: function* () {
    for (const suit of suits) {
      for (const rank of ranks) {
        yield `${rank}${suit}`;
      }
    }
  },
};

console.log([...deck].length); // 52
for (const card of deck) {
  console.log(card);
  break; // Just one card
}

// Example 2: Custom iterable for a binary tree
class TreeNode {
  constructor(value, left = null, right = null) {
    this.value = value;
    this.left = left;
    this.right = right;
  }

  *[Symbol.iterator]() {
    if (this.left) yield* this.left;
    yield this.value;
    if (this.right) yield* this.right;
  }
}

const tree = new TreeNode(2,
  new TreeNode(1),
  new TreeNode(3)
);
console.log([...tree]); // [1, 2, 3]
```

### Intermediate Examples

```javascript
// Example 1: Custom iterable for paginated data
function createPaginatedFetcher(url) {
  return {
    [Symbol.iterator]() {
      let page = 1;
      let hasMore = true;
      let cache = [];

      return {
        async next() {
          if (cache.length > 0) {
            return { value: cache.shift(), done: false };
          }
          if (!hasMore) return { done: true };

          const response = await fetch(`${url}?page=${page}`);
          const data = await response.json();
          page++;
          hasMore = data.hasMore;
          cache = data.items.slice(1);
          return { value: data.items[0], done: false };
        },
      };
    },
  };
}

// Example 2: Custom iterable with multiple iteration modes
class TextDocument {
  constructor(text) {
    this.text = text;
  }

  // Characters
  [Symbol.iterator]() {
    return this.characters();
  }

  *characters() {
    for (const char of this.text) {
      yield char;
    }
  }

  *words() {
    const words = this.text.split(/\s+/);
    for (const word of words) {
      yield word;
    }
  }

  *lines() {
    const lines = this.text.split('\n');
    for (const line of lines) {
      yield line;
    }
  }
}

const doc = new TextDocument("Hello world\nFoo bar");
console.log([...doc]); // ['H','e','l','l','o',' ','w','o','r','l','d','\n','F','o','o',' ','b','a','r']
console.log([...doc.words()]); // ['Hello', 'world', 'Foo', 'bar']
console.log([...doc.lines()]); // ['Hello world', 'Foo bar']

// Example 3: Custom iterable with cycle support
class CyclingIterable {
  constructor(iterable) {
    this.iterable = iterable;
  }

  [Symbol.iterator]() {
    const data = [...this.iterable];
    return {
      next() {
        return { value: data[Math.floor(Math.random() * data.length)], done: false };
      },
    };
  }
}
```

### Advanced Examples

```javascript
// Example 1: Observable-like iterable
class Subject {
  constructor() {
    this.subscribers = [];
    this.completed = false;
  }

  next(value) {
    if (!this.completed) {
      for (const sub of this.subscribers) {
        sub.next(value);
      }
    }
  }

  complete() {
    this.completed = true;
    for (const sub of this.subscribers) {
      sub.complete();
    }
    this.subscribers = [];
  }

  [Symbol.iterator]() {
    const self = this;
    let buffer = [];
    let resolve = null;
    const subscriber = {
      next(value) { buffer.push(value); resolve?.(); resolve = null; },
      complete() { self.completed = true; resolve?.(); resolve = null; },
    };
    this.subscribers.push(subscriber);

    return {
      next() {
        if (buffer.length > 0) {
          return { value: buffer.shift(), done: false };
        }
        if (self.completed) return { done: true };
        return new Promise(r => {
          resolve = () => r({ value: buffer.shift(), done: false });
        });
      },
      return() {
        const idx = self.subscribers.indexOf(subscriber);
        if (idx >= 0) self.subscribers.splice(idx, 1);
        return { done: true };
      },
    };
  }
}

// Example 2: Iterator that can be reused
class ReusableIterable {
  constructor(data) {
    this.data = data;
  }

  [Symbol.iterator]() {
    let index = 0;
    const data = this.data;
    return {
      next() {
        if (index < data.length) {
          return { value: data[index++], done: false };
        }
        return { done: true };
      },
    };
  }

  // Method to reset a specific iterator
  reset() {
    // Reset logic if needed
  }
}

// Example 3: Sliding window iterable
class SlidingWindow {
  constructor(iterable, size) {
    this.data = [...iterable];
    this.size = size;
  }

  [Symbol.iterator]() {
    const data = this.data;
    const size = this.size;
    let index = 0;
    return {
      next() {
        if (index + size <= data.length) {
          return { value: data.slice(index++, index + size - 1), done: false };
        }
        return { done: true };
      },
    };
  }
}

const window = new SlidingWindow([1, 2, 3, 4, 5], 3);
console.log([...window]); // [[1,2,3], [2,3,4], [3,4,5]]
```

### Real-World Use Cases

- **ORM query builders** — Iterating over query result sets.
- **File parsers** — CSV/JSON streaming parsers as iterables.
- **Message queues** — Consuming messages as an iterable.
- **Data generators** — Test data, mock data, random data as iterables.
- **Graph algorithms** — BFS/DFS traversal as iterables.

### Common Mistakes

- **Single-use iteration** — Forgetting to create a new iterator per call.
- **Side effects during iteration** — Mutating the underlying data.
- **Memory leaks** — Not implementing `return()` for cleanup.
- **Too complex** — Over-engineering when an array would suffice.
- **Not handling errors** — Errors during iteration should propagate cleanly.

### Best Practices

- Always return a fresh iterator from `[Symbol.iterator]()`.
- Use generators for simple iteration logic.
- Implement `return()` for resource cleanup.
- Consider making your iterable reusable by caching the data or implementing reset.
- Document whether iteration is lazy or eager.

### Performance Considerations

- Creating custom iterators is cheap.
- For large datasets, lazy iteration saves memory.
- Spreading a custom iterable creates an array — O(n) memory.
- Generator-based iterables may have overhead for simple patterns.
- V8 can optimize custom iterables if they follow monomorphic shapes.

### Interview Questions

1. **How would you make a custom collection iterable?**
   Implement `[Symbol.iterator]()` that returns an iterator with `next()`.

2. **What are the benefits of making a custom class iterable?**
   Integration with `for...of`, spread, destructuring, `Array.from()`, and other language features.

3. **How do you handle cleanup in a custom iterable?**
   Implement `return()` on the iterator object.

4. **Can a custom iterable be used with `Promise.all`?**
   Yes, `Promise.all` accepts any iterable.

### Coding Challenges

1. **Create an iterable `Fibonacci` class that generates Fibonacci numbers.**
2. **Implement a `Range` class that supports `start`, `end`, `step` parameters.**
3. **Build a `takeWhile` iterable that yields values until a condition fails.**
4. **Create a `cycle` iterable that repeats an iterable indefinitely.**
5. **Implement an iterable for a 2D grid that supports row, column, and diagonal traversal.**

### Related Topics

- `Symbol.iterator`
- Iterator protocol
- Generators
- `for...of`
- `yield*`
- Async iterators (`Symbol.asyncIterator`)
