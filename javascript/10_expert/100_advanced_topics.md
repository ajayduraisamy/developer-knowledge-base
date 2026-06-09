# Advanced Topics - Decorators, WeakRef, Temporal API, pipeline operator, records/tuples

## Introduction

JavaScript continues to evolve with new features and proposals that extend the language's capabilities. Advanced topics like decorators, WeakRef, the Temporal API, the pipeline operator, and records/tuples represent the cutting edge of JavaScript development. Understanding these features prepares developers for the future of the language and enables more expressive, safer, and performant code.

## Decorators Proposal

### What It Is

Decorators are a proposal for JavaScript that allows annotating and modifying classes, methods, properties, and parameters at design time. Decorators are functions that wrap or modify the decorated element. The proposal has been through several iterations (the old TC39 Stage 2 proposal differs from the newer Stage 3 proposal). TypeScript's experimental decorators are based on the older proposal.

### Why It Is Important

Decorators enable metaprogramming patterns common in frameworks: dependency injection (Angular), data validation (class-validator), logging, memoization, and authorization. They provide a declarative syntax for cross-cutting concerns that would otherwise require repetitive boilerplate.

### How It Works Internally

A decorator is a function that receives the decorated element (class, method, property, accessor, or parameter) and optionally returns a replacement. In the Stage 3 proposal, decorators can:

- **Class decorators**: Receive and return a constructor function
- **Method decorators**: Receive the target, context object with kind, name, and accessor
- **Property decorators**: Receive undefined and context
- **Accessor decorators**: Receive the getter/setter and context
- **Auto-accessor decorators**: For auto-accessor (`accessor`) syntax

### Syntax

```javascript
// Stage 3 decorator proposal syntax

// Simple method decorator
function logged(target, context) {
  const methodName = context.name;
  
  function replacementMethod(...args) {
    console.log(`Calling ${methodName} with`, args);
    const result = target.call(this, ...args);
    console.log(`Called ${methodName}, returned`, result);
    return result;
  }
  
  return replacementMethod;
}

class Calculator {
  @logged
  add(a, b) {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(2, 3);
// "Calling add with [2, 3]"
// "Called add, returned 5"

// Accessor decorator
function configurable(value) {
  return function(target, context) {
    return {
      get() { return target.get.call(this); },
      set(v) { 
        console.log(`Setting ${String(context.name)} to`, v);
        target.set.call(this, v);
      },
      configurable: value
    };
  };
}

class Config {
  @configurable(false)
  accessor apiKey = '';
}

// Auto-accessor
class Person {
  @required
  accessor name = '';
  
  @range(0, 150)
  accessor age = 0;
}

// Class decorator
function sealed(target, context) {
  Object.seal(target);
  Object.seal(target.prototype);
  return target;
}

@sealed
class ApiService {
  fetch() { /* ... */ }
}
```

### Beginner Examples

```javascript
// Basic decorator use cases

// Logging decorator
function log(target, context) {
  if (context.kind === 'method') {
    return function(...args) {
      console.log(`LOG: ${String(context.name)} called`);
      return target.call(this, ...args);
    };
  }
}

class MyClass {
  @log
  doSomething(value) {
    return value * 2;
  }
}

// Binding decorator
function bound(target, context) {
  if (context.kind === 'method') {
    return function(...args) {
      return target.call(this, ...args);
    }.bind(this); // Simplified; actual implementation is more nuanced
  }
}

class Component {
  constructor(name) { this.name = name; }
  
  @bound
  handleClick() {
    console.log(this.name);
  }
}

// Deprecation decorator
function deprecated(message) {
  return function(target, context) {
    return function(...args) {
      console.warn(`DEPRECATED: ${String(context.name)} - ${message}`);
      return target.call(this, ...args);
    };
  };
}

class OldAPI {
  @deprecated('Use newMethod instead')
  oldMethod() { return 'legacy result'; }
  
  newMethod() { return 'modern result'; }
}
```

### Intermediate Examples

```javascript
// Validation decorators
function validate(target, context) {
  if (context.kind === 'method') {
    return function(...args) {
      const validations = context.metadata?.validations || [];
      for (const validation of validations) {
        validation(...args);
      }
      return target.call(this, ...args);
    };
  }
}

function notEmpty(parameterIndex) {
  return function(target, context) {
    const existing = context.metadata?.validations || [];
    existing.push((...args) => {
      if (args[parameterIndex] === '' || args[parameterIndex] == null) {
        throw new Error(`${parameterIndex} must not be empty`);
      }
    });
    context.metadata = { ...context.metadata, validations: existing };
  };
}

class UserService {
  @validate
  createUser(
    @notEmpty(0) username,
    @notEmpty(1) email
  ) {
    return { username, email };
  }
}

// Caching/Memoization decorator
function memoize(target, context) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = target.call(this, ...args);
    cache.set(key, result);
    return result;
  };
}

class Fibonacci {
  @memoize
  compute(n) {
    if (n <= 1) return n;
    return this.compute(n - 1) + this.compute(n - 2);
  }
}

// Throttle decorator
function throttle(delay = 100) {
  return function(target, context) {
    let lastCall = 0;
    return function(...args) {
      const now = Date.now();
      if (now - lastCall >= delay) {
        lastCall = now;
        return target.call(this, ...args);
      }
    };
  };
}

class SearchAPI {
  @throttle(500)
  search(query) {
    return fetch(`/api/search?q=${query}`);
  }
}
```

## WeakRef and FinalizationRegistry

### What It Is

`WeakRef` (Weak Reference) holds a reference to an object without preventing it from being garbage collected. `FinalizationRegistry` allows registering callbacks that run after an object has been garbage collected. These are advanced memory management features.

### Why It Is Important

WeakRef enables building caches and data structures that don't prevent garbage collection. You can cache expensive computations without memory leaks. The FinalizationRegistry enables cleanup of external resources (file handles, connections) associated with JS objects.

### How It Works Internally

A WeakRef holds a "weak" reference. If the object is still alive, `deref()` returns it; otherwise, `deref()` returns `undefined`. The FinalizationRegistry calls a cleanup callback when an object is collected.

Important: You cannot rely on WeakRef or FinalizationRegistry for critical cleanup. GC behavior is non-deterministic. They're intended for optimization, not resource management.

### Syntax

```javascript
// Basic WeakRef usage
let obj = { data: 'valuable' };
const weakRef = new WeakRef(obj);

// Later
const ref = weakRef.deref();
if (ref) {
  console.log('Object still alive:', ref.data);
} else {
  console.log('Object was collected');
}

obj = null; // Now eligible for GC

// FinalizationRegistry
const registry = new FinalizationRegistry((heldValue) => {
  console.log(`Object with ${heldValue} was collected`);
});

function createResource(name) {
  const resource = { name, handle: openHandle(name) };
  registry.register(resource, name);
  return resource;
}
```

### Beginner Examples

```javascript
// WeakRef for caching
class ImageCache {
  constructor() {
    this.cache = new Map(); // Map<string, WeakRef<ImageData>>
  }
  
  get(url) {
    const weakRef = this.cache.get(url);
    if (weakRef) {
      const image = weakRef.deref();
      if (image) return image; // Cache hit
      this.cache.delete(url); // Clean up dead entry
    }
    return null;
  }
  
  set(url, imageData) {
    this.cache.set(url, new WeakRef(imageData));
  }
}

// Using FinalizationRegistry for cleanup
class FileManager {
  constructor() {
    this.registry = new FinalizationRegistry((filePath) => {
      console.log(`Cleanup: ${filePath}`);
      cleanupFileHandle(filePath);
    });
  }
  
  openFile(path) {
    const handle = { path, fd: openFileDescriptor(path) };
    this.registry.register(handle, path);
    return handle;
  }
}
```

### Intermediate Examples

```javascript
// WeakRef-based event emitter (no manual cleanup needed)
class WeakEventEmitter {
  constructor() {
    this.listeners = new Map(); // event -> Set<WeakRef>
    this.cleanupTimer = setInterval(() => this.cleanup(), 60000);
  }
  
  on(event, handler) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event).add(new WeakRef(handler));
    
    return () => this.off(event, handler);
  }
  
  off(event, handler) {
    const handlers = this.listeners.get(event);
    if (!handlers) return;
    
    for (const ref of handlers) {
      if (ref.deref() === handler) {
        handlers.delete(ref);
        break;
      }
    }
  }
  
  emit(event, ...args) {
    const handlers = this.listeners.get(event);
    if (!handlers) return;
    
    for (const ref of handlers) {
      const handler = ref.deref();
      if (handler) handler(...args);
    }
  }
  
  cleanup() {
    for (const [event, handlers] of this.listeners) {
      for (const ref of handlers) {
        if (!ref.deref()) handlers.delete(ref);
      }
      if (handlers.size === 0) this.listeners.delete(event);
    }
  }
  
  destroy() {
    clearInterval(this.cleanupTimer);
    this.listeners.clear();
  }
}

// WeakRef for DOM element tracking
class DOMObserver {
  constructor() {
    this.elements = new Set(); // Set<WeakRef<Element>>
    this.observer = new MutationObserver(() => this.check());
    this.observer.observe(document.body, { childList: true, subtree: true });
  }
  
  track(element) {
    this.elements.add(new WeakRef(element));
  }
  
  check() {
    for (const ref of this.elements) {
      if (!ref.deref()) {
        this.elements.delete(ref);
        console.log('Element was removed, cleaning up');
      }
    }
  }
}
```

## Temporal API

### What It Is

The Temporal API is a modern replacement for the `Date` object, providing comprehensive date and time handling. It includes types for plain dates, times, date-times, time zones, durations, and calendars. Temporal addresses the many shortcomings of `Date` (mutable, timezone confusion, poor arithmetic).

### Why It Is Important

JavaScript's `Date` object has well-known problems: confusing API (months are 0-indexed), no timezone support beyond UTC/local, mutability, and inconsistent parsing. Temporal provides a clean, immutable, and comprehensive API for all date/time operations.

### How It Works Internally

Temporal provides several types:

- `Temporal.PlainDate`: A date without time (2024-06-15)
- `Temporal.PlainTime`: A time without date (14:30:00)
- `Temporal.PlainDateTime`: Date + Time without timezone
- `Temporal.ZonedDateTime`: Date + Time with timezone
- `Temporal.Instant`: An exact point in time (UTC timestamp)
- `Temporal.Duration`: A length of time (5 days, 3 hours)
- `Temporal.Now`: Methods to get current time in various forms
- `Temporal.TimeZone`: Timezone information
- `Temporal.Calendar`: Calendar system

### Syntax

```javascript
// Temporal API examples (proposal stage 3)

// Current time
const now = Temporal.Now.instant();
const zonedNow = Temporal.Now.zonedDateTimeISO();
const plainDate = Temporal.Now.plainDateISO();
const plainTime = Temporal.Now.plainTimeISO();

// Creating dates
const date = Temporal.PlainDate.from('2024-06-15');
const date2 = Temporal.PlainDate.from({ year: 2024, month: 6, day: 15 });
const dateTime = Temporal.PlainDateTime.from('2024-06-15T14:30:00');
const zoned = Temporal.ZonedDateTime.from('2024-06-15T14:30:00[America/New_York]');

// Date arithmetic
const tomorrow = date.add({ days: 1 });
const lastWeek = date.subtract({ weeks: 1 });
const nextMonth = date.add({ months: 1 });
const duration = Temporal.Duration.from({ days: 5, hours: 3 });
const result = date.add(duration);

// Comparison
date.equals(date2); // true/false
date.compare(date, date2); // -1, 0, or 1

// Formatting
date.toLocaleString('en-US'); // "6/15/2024"
date.toLocaleString('en-GB'); // "15/06/2024"
```

### Beginner Examples

```javascript
// Temporal vs Date comparison

// Date (old way)
const d = new Date();
d.setFullYear(2024);
d.setMonth(5); // June (0-indexed!)
d.setDate(15);
console.log(d.toISOString().split('T')[0]); // Hacky formatting

// Temporal (new way)
const date = Temporal.PlainDate.from({ year: 2024, month: 6, day: 15 });
console.log(date.toString()); // "2024-06-15"

// Date arithmetic (Date has no built-in arithmetic)
const date1 = Temporal.PlainDate.from('2024-01-01');
const date2 = Temporal.PlainDate.from('2024-12-25');
const diff = date1.until(date2);
console.log(diff.total('days')); // 359

// Timezone handling
const meeting = Temporal.ZonedDateTime.from(
  '2024-06-15T14:00:00[America/New_York]'
);
console.log(meeting.toLocaleString('en-US', { timeZone: 'Europe/London' }));
// "2024-06-15T19:00:00[Europe/London]"

// PlainTime for business hours
const open = Temporal.PlainTime.from('09:00');
const close = Temporal.PlainTime.from('17:00');
const nowTime = Temporal.Now.plainTimeISO();
const isOpen = nowTime.compare(open) >= 0 && nowTime.compare(close) < 0;
```

### Intermediate Examples

```javascript
// Working with business days
function addBusinessDays(startDate, days) {
  let current = startDate;
  let added = 0;
  
  while (added < days) {
    current = current.add({ days: 1 });
    const dayOfWeek = current.dayOfWeek;
    if (dayOfWeek >= 1 && dayOfWeek <= 5) { // Monday to Friday
      added++;
    }
  }
  
  return current;
}

const start = Temporal.PlainDate.from('2024-06-03'); // Monday
const result = addBusinessDays(start, 5);
console.log(result.toString()); // "2024-06-10" (skips weekend)

// Duration formatting
function formatDuration(duration) {
  const d = Temporal.Duration.from(duration);
  const parts = [];
  
  if (d.years) parts.push(`${d.years}y`);
  if (d.months) parts.push(`${d.months}m`);
  if (d.days) parts.push(`${d.days}d`);
  if (d.hours) parts.push(`${d.hours}h`);
  if (d.minutes) parts.push(`${d.minutes}m`);
  if (d.seconds) parts.push(`${d.seconds}s`);
  
  return parts.join(' ');
}

console.log(formatDuration({ days: 5, hours: 3, minutes: 30 }));
// "5d 3h 30m"

// Recurring events (using Temporal arithmetic)
function* monthlyRecurrence(startDate, count) {
  let current = startDate;
  for (let i = 0; i < count; i++) {
    yield current;
    current = current.add({ months: 1 });
  }
}

const start = Temporal.PlainDate.from('2024-01-15');
const meetings = [...monthlyRecurrence(start, 3)];
console.log(meetings.map(d => d.toString()));
// ["2024-01-15", "2024-02-15", "2024-03-15"]
```

## Pipeline Operator (|>)

### What It Is

The pipeline operator (`|>`) is a Stage 2 proposal that provides a syntactic sugar for chaining function calls in a readable, left-to-right manner. Instead of nesting function calls (right-to-left reading), the pipeline operator lets you write operations sequentially.

### Why It Is Important

The pipeline operator eliminates deeply nested function calls (`f(g(h(x)))`), making code more readable by following the natural left-to-right execution flow. It's inspired by pipelines in functional languages (Elixir, F#, Haskell) and Unix shell pipes.

### How It Works Internally

`expression |> function` evaluates `expression`, then calls `function` with the result as its argument. `expression |> function(args...)` calls `function` with `expression` as the first argument, followed by `args`. The pipeline is just syntactic sugar—it has no runtime overhead beyond normal function calls.

### Syntax

```javascript
// Basic pipeline
value |> fn           // fn(value)
value |> fn(args)     // fn(value, ...args)

// Pipeline with function calls
const result = 5
  |> (x => x * 2)
  |> (x => x + 1)
  |> (x => `Result: ${x}`);

console.log(result); // "Result: 11"

// Without pipeline (nested, right-to-left):
const result2 = `Result: ${(5 * 2) + 1}`;

// Compared to chaining
// Chain:
[1, 2, 3]
  .map(x => x * 2)
  .filter(x => x > 2)
  .reduce((a, b) => a + b, 0);

// Pipeline (when methods aren't available):
const data = [1, 2, 3];
const result3 = data
  |> (arr => arr.map(x => x * 2))
  |> (arr => arr.filter(x => x > 2))
  |> (arr => arr.reduce((a, b) => a + b, 0));

// Pipeline with partial application (proposal)
// value |> fn(%) // placeholder for value
```

### Beginner Examples

```javascript
// Data transformation pipeline
function double(x) { return x * 2; }
function increment(x) { return x + 1; }
function toString(x) { return `Value: ${x}`; }

// Without pipeline:
const r1 = toString(increment(double(5)));
// Reads: apply double, then increment, then toString (right-to-left)

// With pipeline:
const r2 = 5
  |> double     // 10
  |> increment  // 11
  |> toString;  // "Value: 11"
// Reads top-to-bottom, left-to-right

// Practical: string processing
const name = '  alice  ';
const formatted = name
  |> (s => s.trim())
  |> (s => s.charAt(0).toUpperCase() + s.slice(1));

console.log(formatted); // "Alice"

// Array processing pipeline
const numbers = [5, 2, 8, 1, 9, 3];
const pipeline = numbers
  |> (arr => arr.sort((a, b) => a - b))
  |> (arr => arr.filter(n => n % 2 === 0))
  |> (arr => arr.reduce((sum, n) => sum + n, 0));

console.log(pipeline); // 10 (2 + 8)
```

### Intermediate Examples

```javascript
// Pipeline with well-known functions
import { pipe } from 'lodash/fp';

// Without pipeline operator (using lodash pipe):
const processOrder = pipe(
  validateOrder,
  calculateTotal,
  applyDiscount,
  formatResponse
);

// With pipeline operator (someday):
function handleOrder(order) {
  return order
    |> validateOrder
    |> calculateTotal
    |> applyDiscount
    |> formatResponse;
}

// Error handling in pipelines
function safeParseJSON(json) {
  try {
    return JSON.parse(json);
  } catch {
    return null;
  }
}

function getField(data) {
  return data?.field;
}

const result = '{"field":"value"}'
  |> safeParseJSON
  |> getField;

console.log(result); // "value"

// Async pipelines (with await)
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

function extractName(user) {
  return user?.name;
}

function capitalize(str) {
  return str?.charAt(0).toUpperCase() + str?.slice(1);
}

// Today:
const userData = await fetchUser(1);
const userName = extractName(userData);
const formattedName = capitalize(userName);

// With pipeline + await (hypothetical):
// const name = await fetchUser(1)
//   |> extractName
//   |> capitalize;
```

## Records and Tuples Proposal

### What It Is

Records and Tuples are a Stage 2 proposal for immutable, deeply equal data structures in JavaScript. Records (`#{ ... }`) are immutable objects, and tuples (`#[ ... ]`) are immutable arrays. They are compared by value (not reference) and are deeply immutable.

### Why It Is Important

Records and Tuples solve the fundamental JavaScript problem of mutable objects and reference-based comparison. They enable:
- **Value equality**: `#{a: 1} === #{a: 1}` is `true`
- **Immutability by default**: No accidental mutations
- **Optimization**: Structurally identical records can share memory
- **Better state management**: Predictable state updates

### How It Works Internally

Records and tuples are primitive-like values (like strings and numbers). They are compared by their contents, not by identity. They are deeply immutable—you cannot set properties on a record or indexes on a tuple. Nested objects must be records/tuples for deep equality.

```javascript
// Records are created with #{} syntax
const record = #{ x: 1, y: 2, name: "point" };

// Tuples are created with #[] syntax
const tuple = #[1, 2, 3, "hello"];

// Equality is by value
#{ a: 1 } === #{ a: 1 }; // true
#[1, 2, 3] === #[1, 2, 3]; // true
```

### Syntax

```javascript
// Creating records
const point = #{ x: 10, y: 20 };
const nested = #{ a: #{ b: 2 } };

// Creating tuples
const list = #[1, 2, 3];
const mixed = #["hello", 42, #{ value: true }];

// Accessing values
point.x; // 10
list[0]; // 1

// Updating (returns new record/tuple, doesn't mutate)
const moved = #{ ...point, x: 30 };
const extended = #[...list, 4, 5];

// Spreading into objects
const obj = { ...point, z: 30 }; // Converts record to object
```

### Beginner Examples

```javascript
// Value equality
const a = #{ x: 1, y: 2 };
const b = #{ x: 1, y: 2 };
console.log(a === b); // true (same content)

// Object vs Record
const objA = { x: 1, y: 2 };
const objB = { x: 1, y: 2 };
console.log(objA === objB); // false (different references)

// Using records as Map keys
const scores = new Map();
scores.set(#{ name: "Alice", game: "chess" }, 1500);
scores.set(#{ name: "Bob", game: "chess" }, 1400);
console.log(scores.get(#{ name: "Alice", game: "chess" })); // 1500

// Nested records
const user = #{
  id: 1,
  name: "Alice",
  address: #{
    city: "NYC",
    zip: "10001"
  }
};

// user.address.city = "LA"; // TypeError: cannot mutate record
const moved = #{ ...user, address: #{ ...user.address, city: "LA" } };
```

### Intermediate Examples

```javascript
// Using tuples for React hooks (stable references)
function useForm(initialValues) {
  const [values, setValues] = React.useState(initialValues);
  
  // Stable reference when values haven't changed
  const stableValues = #{ ...values };
  
  return [stableValues, setValues];
}

// State management with records
function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return #{ ...state, count: state.count + 1 };
    case 'reset':
      return #{ count: 0 };
    default:
      return state;
  }
}

// Deep updates
const state = #{
  user: #{
    profile: #{
      name: "Alice",
      settings: #{
        theme: "dark",
        notifications: true
      }
    }
  }
};

const newState = #{
  ...state,
  user: #{
    ...state.user,
    profile: #{
      ...state.user.profile,
      settings: #{
        ...state.user.profile.settings,
        theme: "light"
      }
    }
  }
};

// Records as Redux actions
const action = #{ type: "UPDATE_USER", payload: #{ id: 1, name: "Bob" } };

// Comparing in selectors
const memoizedSelector = (state) => {
  const key = #{
    userId: state.userId,
    filter: state.filter
  };
  // If userId and filter haven't changed, key === previous key
  return cache.getOrCreate(key, () => computeExpensiveResult(state));
};
```

### Advanced Examples

```javascript
// Destructuring records
const #{ x, y, z = 0 } = #{ x: 10, y: 20 };
console.log(x, y, z); // 10, 20, 0

// Pattern matching (proposal) with records
// match (point) {
//   when (#{ x: 0, y: 0 }) -> "origin";
//   when (#{ x: _, y: 0 }) -> "on x-axis";
//   when (#{ x: 0, y: _ }) -> "on y-axis";
//   when (#{ x: _, y: _ }) -> `point ${point.x},${point.y}`;
// }

// Converting objects to records
function toRecord(obj) {
  const entries = Object.entries(obj).map(([key, value]) => {
    if (value && typeof value === 'object' && !Array.isArray(value)) {
      return [key, toRecord(value)];
    }
    if (Array.isArray(value)) {
      return [key, toTuple(value)];
    }
    return [key, value];
  });
  return #{ ...Object.fromEntries(entries) };
}

function toTuple(arr) {
  return #[...arr.map(v => 
    v && typeof v === 'object' ? toRecord(v) : v
  )];
}

// Usage with API data
const apiUser = { id: 1, name: "Alice", tags: ["admin", "premium"] };
const record = toRecord(apiUser);
console.log(record === toRecord(apiUser)); // true (if same data)

// Records for frozen configuration
const CONFIG = #{
  api: #{
    baseUrl: "https://api.example.com",
    timeout: 5000,
    retries: 3
  },
  features: #{
    darkMode: true,
    beta: false
  }
};

// CONFIG.api.timeout = 10000; // TypeError!
// Create new config instead
const newConfig = #{ ...CONFIG, api: #{ ...CONFIG.api, timeout: 10000 } };
```

### Real-World Use Cases

- **Decorators**: Framework annotations (Angular, NestJS), logging, validation
- **WeakRef**: Large caches, DOM element tracking, event emitter cleanup
- **Temporal API**: Scheduling, date arithmetic, timezone conversion, logging
- **Pipeline operator**: Data transformation chains, processing pipelines
- **Records/Tuples**: State management (Redux), memoization, configuration

### Current Status and Adoption

- **Decorators**: Stage 3 (2023-03), TypeScript 5.0+ supports 3rd proposal
- **WeakRef/FinalizationRegistry**: Stage 4 (shipped in ES2021)
- **Temporal API**: Stage 3, available via polyfill
- **Pipeline operator**: Stage 2, evolving design
- **Records/Tuples**: Stage 2, awaiting implementation feedback

### Common Mistakes

- Using WeakRef for critical cleanup (GC is non-deterministic)
- Assuming Temporal API is available without polyfill
- Expecting decorators to work like TypeScript experimental decorators
- Creating deeply nested record updates without helper functions
- Using pipeline operator in production without transpiler support

### Best Practices

- Use polyfills for advanced proposals (Temporal, Records/Tuples)
- Babel plugins for decorators, pipeline operator
- Use WeakRef only for non-critical caching optimizations
- Prefer Temporal over Date for all new date/time code
- Use helper functions like `updateIn` for nested record updates
- Combine pipeline operator with other functional patterns


### Performance Considerations
- Decorators can impact performance due to wrapper function creation; prefer simple decorators over complex ones
- WeakRef cleanup is non-deterministic and should not be relied upon for critical resource management
- Temporal API operations are generally faster than manual Date arithmetic
- Pipeline operator (|>) avoids intermediate variable allocation, improving memory usage
- Records/Tuples use structural equality which is O(n) compared to reference equality O(1)


### Interview Questions
1. How do you implement a simple decorator in JavaScript?
2. What are the use cases for WeakRef and FinalizationRegistry?
3. How does the Temporal API differ from the Date object?
4. Explain the pipeline operator proposal and its benefits.
5. What are the advantages of Records and Tuples over plain objects/arrays?


### Coding Challenges
```javascript
// Challenge 1: Implement a @log decorator
function log(target, name, descriptor) {
  const original = descriptor.value;
  descriptor.value = function(...args) {
    console.log(`Calling ${name} with`, args);
    return original.apply(this, args);
  };
  return descriptor;
}

class Calculator {
  @log
  add(a, b) { return a + b; }
}

const calc = new Calculator();
console.log(calc.add(3, 4));

// Challenge 2: Use Temporal API for date arithmetic
const { Temporal } = require('@js-temporal/polyfill');
const today = Temporal.Now.plainDateISO();
const future = today.add({ days: 30 });
console.log(`Today: ${today}, Future: ${future}`);

// Challenge 3: WeakRef to track object lifecycle
let target = { data: "important" };
const ref = new WeakRef(target);
console.log(ref.deref()?.data);
target = null; // May be GC'd later
setTimeout(() => console.log(ref.deref()), 1000);
```

### Related Topics

- TC39 proposal process (Stage 0-4)
- Babel and transpilation
- Polyfills and core-js
- Immutable data structures
- Functional programming patterns
- Date/Time handling best practices
- Metaprogramming in JavaScript
- State management architectures
