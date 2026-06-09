# Symbols

## Introduction

Symbols are a primitive data type introduced in ES6 that serve as unique, immutable identifiers. Unlike strings or numbers, every Symbol is guaranteed to be unique, making them ideal for creating private-like object properties, avoiding name collisions, and defining well-known protocols that the JavaScript language uses internally. Symbols are the foundation for meta-programming in JavaScript.

---

## Symbol() factory

### What It Is

`Symbol()` is a factory function that creates a new unique Symbol value. Each call creates a completely unique Symbol, even if given the same optional description string. Symbols can be used as object property keys, ensuring no property name collisions.

```javascript
const sym1 = Symbol("debug");
const sym2 = Symbol("debug");
console.log(sym1 === sym2); // false (unique)
console.log(sym1.toString()); // "Symbol(debug)"

const obj = {};
obj[sym1] = "value1";
obj[sym2] = "value2";
console.log(obj[sym1]); // "value1"
console.log(obj[sym2]); // "value2"
```

### Why It Is Important

Symbols solve several critical problems:
- **Property uniqueness** — No accidental name collisions when adding metadata to objects.
- **Protocol definition** — Well-known symbols define language-level protocols.
- **Privacy simulation** — Symbols are not enumerable in `for...in` or `Object.keys()`.
- **Metadata annotation** — Adding metadata without polluting string keys.
- **Framework extensibility** — Libraries can add Symbol-keyed properties safely.

### How It Works Internally

Symbols are primitive values (not objects). The engine maintains an internal registry for Symbol descriptions (for debugging). Each Symbol has a unique identity that cannot be replicated. When used as property keys, the engine stores them in a separate internal table from string keys, which is why they are hidden from `Object.keys()` and `for...in`.

### Syntax

```javascript
// Creating symbols
const sym = Symbol();
const symWithDesc = Symbol("description");

// Using as property keys
const obj = {};
obj[sym] = "value";

// Symbols in object literals
const obj = {
  [sym]: "value",
  [Symbol("key")]: "another",
};

// Symbols are not auto-converted to strings
// console.log(sym + ""); // TypeError
console.log(sym.toString()); // "Symbol(description)"
console.log(String(sym)); // "Symbol(description)"
```

### Beginner Examples

```javascript
// Example 1: Basic uniqueness
const id1 = Symbol("id");
const id2 = Symbol("id");
console.log(id1 === id2); // false

// Example 2: Symbols as property keys
const user = {
  name: "Alice",
  [Symbol("id")]: 12345,
  [Symbol("token")]: "abc-def",
};

console.log(user.name); // "Alice"
console.log(Object.keys(user)); // ["name"] (Symbols hidden)
console.log(Object.getOwnPropertySymbols(user)); // [Symbol(id), Symbol(token)]

// Example 3: Symbols are not included in JSON
console.log(JSON.stringify(user)); // {"name":"Alice"}

// Example 4: Symbol descriptions (for debugging)
const s = Symbol("my descriptive text");
console.log(s.description); // "my descriptive text" (ES2019)
```

### Intermediate Examples

```javascript
// Example 1: Constants using Symbols
const Colors = {
  RED: Symbol("red"),
  GREEN: Symbol("green"),
  BLUE: Symbol("blue"),
};

function paint(color) {
  if (color === Colors.RED) return "#FF0000";
  if (color === Colors.GREEN) return "#00FF00";
  if (color === Colors.BLUE) return "#0000FF";
  throw new Error("Unknown color");
}
// No two colors can be equal accidentally
// Unlike string "red" which could collide

// Example 2: Avoiding name collisions in libraries
const libraryA = (() => {
  const key = Symbol("data");
  return {
    addData(obj, data) { obj[key] = data; },
    getData(obj) { return obj[key]; },
  };
})();

const libraryB = (() => {
  const key = Symbol("data"); // Different symbol!
  return {
    addData(obj, data) { obj[key] = data; },
    getData(obj) { return obj[key]; },
  };
})();

const obj = {};
libraryA.addData(obj, "Library A data");
libraryB.addData(obj, "Library B data");
console.log(libraryA.getData(obj)); // "Library A data"
console.log(libraryB.getData(obj)); // "Library B data"

// Example 3: Symbol-keyed method
const speak = Symbol("speak");
class Animal {
  [speak]() {
    return "Animal speaks";
  }
  regularSpeak() {
    return this[speak]();
  }
}
const a = new Animal();
console.log(a.regularSpeak()); // "Animal speaks"
// a.speak is undefined (hidden)
```

### Advanced Examples

```javascript
// Example 1: Simulating private methods
const _private = Symbol("private");
class SecureStore {
  constructor(secret) {
    this[_private] = secret;
  }

  getSecret() {
    return this[_private];
  }

  // Still accessible via Object.getOwnPropertySymbols
  // but hidden from casual inspection
}

// Example 2: Custom behavior with Symbol properties
const serialized = Symbol("serialized");
class User {
  constructor(name, age) {
    this.name = name;
    this.age = age;
    this[serialized] = `${name}:${age}`;
  }

  toJSON() {
    return this[serialized];
  }
}

// Example 3: Symbol for dependency injection
const DI = Symbol("di");
class ServiceContainer {
  constructor() {
    this[DI] = new Map();
  }

  register(name, instance) {
    this[DI].set(name, instance);
  }

  get(name) {
    return this[DI].get(name);
  }
}

// Example 4: Well-known Symbol usage
class CustomCollection {
  constructor(items) {
    this.items = items;
  }

  [Symbol.iterator]() {
    return this.items[Symbol.iterator]();
  }

  [Symbol.toPrimitive](hint) {
    if (hint === "string") return `Collection(${this.items.length})`;
    if (hint === "number") return this.items.length;
    return null;
  }

  get [Symbol.toStringTag]() {
    return "CustomCollection";
  }
}
```

### Real-World Use Cases

- **Framework metadata** — React's `$$typeof` uses a Symbol to mark React elements.
- **Protocol definitions** — Iterators, async iterators, and other protocols.
- **Library clean APIs** — Adding internal metadata without polluting the API surface.
- **Enum alternatives** — Unique values for switch/case patterns.
- **ORM metadata** — Storing schema information on model instances.

### Common Mistakes

- **Assuming Symbols are private** — They are still accessible via `Object.getOwnPropertySymbols()`.
- **Trying to use `new Symbol()`** — Symbol is not a constructor, it's a factory function.
- **Auto-converting Symbols to strings** — `String(sym)` works but `sym + ''` throws.
- **Using Symbols for truly sensitive data** — They are not private, just non-enumerable.
- **Forgetting that Symbols are unique** — Two identical descriptions still create different Symbols.

### Best Practices

- Use Symbols for protocol-level property keys (well-known symbols).
- Use Symbols to avoid name collisions in library code.
- Use Symbols for enum-like constants.
- Use `Symbol.for()` for shared/cross-realm Symbols (see Symbol registry).
- Use `.description` for debugging, not for equality.

### Performance Considerations

- Creating Symbols is very cheap (similar to creating a string).
- Property access with Symbol keys is similar to string key access.
- V8 optimizes Symbol-keyed properties the same as string-keyed.
- Using too many distinct Symbols per object is not recommended.
- Symbol-keyed properties cannot be accessed via dot notation (always bracket notation).

### Interview Questions

1. **What is a Symbol and why is it useful?**
   A unique, immutable primitive value used as property keys to avoid name collisions.

2. **How do you create a Symbol?**
   Using `Symbol()` or `Symbol("description")`.

3. **Can Symbols be converted to strings automatically?**
   No. You must use explicit `.toString()` or `String()`.

4. **Are Symbols truly private?**
   No. They are hidden from `for...in` and `Object.keys()`, but accessible via `Object.getOwnPropertySymbols()`.

### Coding Challenges

1. **Create a constant enum using Symbols.**
2. **Implement a `HiddenField` mixin that uses Symbols for private-like fields.**
3. **Build a `MetadataStore` that uses Symbols to attach data to objects.**
4. **Create a class that implements multiple well-known Symbols.**

### Related Topics

- Well-known symbols
- Symbol registry
- Property keys
- `Object.getOwnPropertySymbols()`
- Symbol.toPrimitive

---

## Well-known symbols (Symbol.iterator, Symbol.toStringTag)

### What It Is

Well-known symbols are built-in Symbol values on the Symbol constructor that define language-level protocols. They allow custom objects to integrate with JavaScript's built-in behavior for iteration, string conversion, type coercion, instanceof checks, and more.

```javascript
class MyCollection {
  constructor(items) { this.items = items; }
  *[Symbol.iterator]() { yield* this.items; }
  get [Symbol.toStringTag]() { return "MyCollection"; }
  [Symbol.toPrimitive](hint) {
    if (hint === "number") return this.items.length;
    return `[${this.items.join(",")}]`;
  }
}
```

### Why It Is Important

Well-known symbols define the contract between custom code and the JavaScript runtime:
- `Symbol.iterator` — Makes objects iterable with `for...of`.
- `Symbol.toStringTag` — Customizes `Object.prototype.toString.call(obj)`.
- `Symbol.toPrimitive` — Controls type coercion behavior.
- `Symbol.hasInstance` — Customizes `instanceof` behavior.
- `Symbol.species` — Controls which constructor is used for derived objects.

### How It Works Internally

The engine uses these symbols internally. For example, when `for...of` runs, it calls `obj[Symbol.iterator]()`. When `Object.prototype.toString()` runs, it checks for `Symbol.toStringTag`. When `instanceof` runs, it checks for `Symbol.hasInstance`.

### Syntax

```javascript
// Most commonly used well-known symbols:
Symbol.iterator
Symbol.toStringTag
Symbol.toPrimitive
Symbol.hasInstance
Symbol.species
Symbol.match / Symbol.replace / Symbol.search / Symbol.split
Symbol.unscopables
Symbol.asyncIterator
Symbol.isConcatSpreadable
```

### Beginner Examples

```javascript
// Example 1: Symbol.iterator
class NumberRange {
  constructor(start, end) { this.start = start; this.end = end; }
  *[Symbol.iterator]() {
    for (let i = this.start; i <= this.end; i++) yield i;
  }
}

for (const n of new NumberRange(1, 5)) {
  console.log(n); // 1, 2, 3, 4, 5
}

// Example 2: Symbol.toStringTag
class Database {
  get [Symbol.toStringTag]() { return "Database"; }
}
const db = new Database();
console.log(Object.prototype.toString.call(db)); // "[object Database]"
console.log(String(db)); // "[object Database]"

// Example 3: Symbol.toPrimitive
class Temperature {
  constructor(celsius) { this.celsius = celsius; }
  [Symbol.toPrimitive](hint) {
    if (hint === "number") return this.celsius;
    if (hint === "string") return `${this.celsius}°C`;
    return this.celsius * 9/5 + 32; // default: fahrenheit
  }
}
const temp = new Temperature(25);
console.log(+temp); // 25 (number hint)
console.log(`${temp}`); // "25°C" (string hint)
console.log(temp + 10); // 77 (default hint, fahrenheit + 10)
```

### Intermediate Examples

```javascript
// Example 1: Symbol.hasInstance
class PositiveNumber {
  static [Symbol.hasInstance](value) {
    return typeof value === "number" && value > 0;
  }
}
console.log(5 instanceof PositiveNumber); // true
console.log(-3 instanceof PositiveNumber); // false
console.log("hello" instanceof PositiveNumber); // false

// Example 2: Symbol.species
class MyArray extends Array {
  static get [Symbol.species]() { return Array; }
  // When map/filter/fill etc return a new array,
  // it will be a plain Array, not MyArray
}

const myArr = new MyArray(1, 2, 3);
const mapped = myArr.map(x => x * 2);
console.log(mapped instanceof MyArray); // false (uses Array)
console.log(mapped instanceof Array); // true

// Example 3: Symbol.isConcatSpreadable
const specialArray = {
  length: 2,
  0: "a",
  1: "b",
  [Symbol.isConcatSpreadable]: true,
};
console.log(["start"].concat(specialArray)); // ["start", "a", "b"]

// Without the Symbol:
const normalObj = { length: 2, 0: "x", 1: "y" };
console.log(["start"].concat(normalObj)); // ["start", { length:2, 0:"x", 1:"y" }]
```

### Advanced Examples

```javascript
// Example 1: Symbol.match for custom pattern matching
class StartsWith {
  constructor(str) { this.str = str; }
  [Symbol.match](target) {
    return target.startsWith(this.str) ? [this.str] : null;
  }
}

console.log("Hello World".match(new StartsWith("Hello"))); // ["Hello"]
console.log("Goodbye".match(new StartsWith("Hello"))); // null

// Example 2: Complete well-known symbol integration
class SmartArray {
  constructor(items = []) { this.items = [...items]; }

  // Iterator
  *[Symbol.iterator]() { yield* this.items; }

  // String tag
  get [Symbol.toStringTag]() { return "SmartArray"; }

  // Primitive coercion
  [Symbol.toPrimitive](hint) {
    if (hint === "number") return this.items.length;
    return `[${this.items.join(", ")}]`;
  }

  // Concat spread
  get [Symbol.isConcatSpreadable]() { return true; }
}

const smart = new SmartArray([1, 2, 3]);
console.log([...smart]); // [1, 2, 3]
console.log(Object.prototype.toString.call(smart)); // "[object SmartArray]"
console.log(+smart); // 3
console.log([0].concat(smart)); // [0, 1, 2, 3]
```

### Real-World Use Cases

- **Custom collections** — Implementing `Symbol.iterator` for custom data structures.
- **Serialization** — Customizing `Symbol.toStringTag` for better debugging.
- **Type coercion** — `Symbol.toPrimitive` for custom type conversion logic.
- **Pattern matching** — `Symbol.match`, `Symbol.replace` for custom string operations.
- **Framework interop** — Using `Symbol.species` for derived classes.

### Common Mistakes

- **Forgetting `*` on `[Symbol.iterator]`** — Makes it a generator method.
- **Returning wrong type from `Symbol.toPrimitive`** — Must return a primitive, not object.
- **Not using `get` for `Symbol.toStringTag`** — It should be a getter.
- **Shadowing well-known symbols unintentionally** — Adding a property with the same name.

### Best Practices

- Always implement `Symbol.iterator` on custom collection classes.
- Implement `Symbol.toStringTag` for better debugging output.
- Use `Symbol.toPrimitive` when custom type coercion is needed.
- Use `Symbol.species` carefully — it can cause unexpected behavior.
- Document which well-known symbols your class implements.

### Performance Considerations

- Well-known symbols are cached by the engine — fast property access.
- Classes implementing many well-known symbols may have more complex shape.
- Well-known symbol lookups are inlined by V8.
- Most well-known symbols are only accessed during specific operations.

### Interview Questions

1. **What are well-known symbols?**
   Built-in Symbols on the Symbol constructor that define language-level protocols.

2. **Name three well-known symbols and their purposes.**
   `Symbol.iterator` (iteration), `Symbol.toStringTag` (string tag), `Symbol.toPrimitive` (type coercion).

3. **How does `Symbol.hasInstance` work?**
   It customizes the `instanceof` operator. If defined statically, it's called with the object to check.

4. **What is `Symbol.species` used for?**
   It controls which constructor is used when creating derived objects from array methods.

### Coding Challenges

1. **Create a class that implements `Symbol.iterator`, `Symbol.toStringTag`, and `Symbol.toPrimitive`.**
2. **Implement a custom `Symbol.hasInstance` that validates email addresses.**
3. **Build a `Symbol.match`-based string matcher.**
4. **Create an array subclass with custom `Symbol.species`.**

### Related Topics

- `Symbol()` factory
- Symbol registry
- Iterators
- Type coercion
- `instanceof` operator
- `Object.prototype.toString()`

---

## Symbol registry (for()/keyFor())

### What It Is

The Symbol registry is a global, cross-realm Symbol store. `Symbol.for(key)` returns an existing Symbol for the given key or creates a new one. `Symbol.keyFor(sym)` returns the key string for a registered Symbol. This enables Symbol sharing across different JavaScript realms (iframes, service workers, etc.).

```javascript
const sym1 = Symbol.for("app.unique");
const sym2 = Symbol.for("app.unique");
console.log(sym1 === sym2); // true (same Symbol)

const key = Symbol.keyFor(sym1);
console.log(key); // "app.unique"
```

### Why It Is Important

The Symbol registry solves the cross-realm and cross-module Symbol sharing problem:
- Ensures the same Symbol is used across iframes, workers, and modules.
- Allows libraries to share well-known Symbols.
- Enables runtime protocol registration.
- Avoids the need to pass Symbol references explicitly.

### How It Works Internally

The engine maintains a global Symbol registry (a map from string keys to Symbols). When `Symbol.for(key)` is called, it looks up the key in the registry. If found, returns the existing Symbol. If not, creates a new Symbol and registers it. `Symbol.keyFor(sym)` checks if the Symbol is in the registry and returns its key. The registry is per-realm in some engines but shared in modern environments.

### Syntax

```javascript
// Register or retrieve a Symbol
const sym = Symbol.for("key");

// Get the key for a registered Symbol
const key = Symbol.keyFor(sym);

// Unregistered Symbols return undefined for keyFor
const localSym = Symbol("test");
console.log(Symbol.keyFor(localSym)); // undefined
```

### Beginner Examples

```javascript
// Example 1: Cross-module Symbol sharing
// module-a.js
const SHARED_KEY = Symbol.for("app.cacheKey");
class CacheA {
  set(obj, value) { obj[SHARED_KEY] = value; }
  get(obj) { return obj[SHARED_KEY]; }
}

// module-b.js (same Symbol)
const SHARED_KEY = Symbol.for("app.cacheKey");
class CacheB {
  has(obj) { return SHARED_KEY in obj; }
}

// Both use the same Symbol!

// Example 2: keyFor
const mySym = Symbol.for("myKey");
console.log(Symbol.keyFor(mySym)); // "myKey"

const notRegistered = Symbol("test");
console.log(Symbol.keyFor(notRegistered)); // undefined

// Example 3: Cross-realm usage
// If you have an iframe:
// const iframeSym = iframe.contentWindow.Symbol.for("shared");
// const localSym = Symbol.for("shared");
// console.log(iframeSym === localSym); // true
```

### Intermediate Examples

```javascript
// Example 1: Protocol registration pattern
const Protocols = {
  HTTP: Symbol.for("protocol.http"),
  HTTPS: Symbol.for("protocol.https"),
  WS: Symbol.for("protocol.ws"),
};

class RequestHandler {
  constructor(protocol) {
    this.protocol = protocol;
  }

  static register(symbol, handler) {
    RequestHandler.handlers[symbol] = handler;
  }

  static handlers = {};

  handle(request) {
    const handler = RequestHandler.handlers[this.protocol];
    if (!handler) throw new Error("Unknown protocol");
    return handler(request);
  }
}

// Register handlers (even from different modules)
RequestHandler.register(Protocols.HTTP, (req) => `HTTP: ${req.url}`);
RequestHandler.register(Protocols.HTTPS, (req) => `HTTPS: ${req.url}`);

// Example 2: Plugin system with Symbol.for
const PluginAPI = {
  INIT: Symbol.for("plugin.init"),
  DESTROY: Symbol.for("plugin.destroy"),
  GET_DATA: Symbol.for("plugin.getData"),
};

class PluginManager {
  constructor() {
    this.plugins = new Set();
  }

  register(plugin) {
    if (typeof plugin[PluginAPI.INIT] === "function") {
      this.plugins.add(plugin);
      plugin[PluginAPI.INIT]();
    }
  }

  unregister(plugin) {
    if (typeof plugin[PluginAPI.DESTROY] === "function") {
      plugin[PluginAPI.DESTROY]();
    }
    this.plugins.delete(plugin);
  }
}

// Example 3: Avoiding collisions with namespaced keys
const MY_LIB = Symbol.for("mylib.internal");
// Use descriptive, namespaced keys like "libname.purpose"
```

### Advanced Examples

```javascript
// Example 1: Dynamic protocol symbol
function defineHook(name) {
  return Symbol.for(`hook.${name}`);
}

const beforeRender = defineHook("beforeRender");
const afterRender = defineHook("afterRender");

class Renderer {
  constructor() {
    this.hooks = new Map();
  }

  addHook(symbol, fn) {
    if (!this.hooks.has(symbol)) this.hooks.set(symbol, []);
    this.hooks.get(symbol).push(fn);
  }

  render() {
    this.runHooks(beforeRender);
    // ... rendering ...
    this.runHooks(afterRender);
  }

  runHooks(symbol) {
    const hooks = this.hooks.get(symbol) || [];
    hooks.forEach(fn => fn());
  }
}

// Example 2: Testing Symbol registration
function isRegisteredSymbol(sym) {
  return Symbol.keyFor(sym) !== undefined;
}

// Example 3: Weak reference to registered Symbols
const cache = new WeakMap();
function getOrCreateSymbol(key) {
  if (!cache.has(key)) {
    cache.set(key, Symbol.for(key));
  }
  return cache.get(key);
}
```

### Real-World Use Cases

- **Cross-realm communication** — Symbols shared between iframe and parent window.
- **Plugin/extension systems** — Well-known hooks identified by Symbol.for.
- **Protocol definitions** — Shared identifiers for loosely coupled modules.
- **Dependency injection** — Identifiers for services.
- **Framework extensibility** — Vue's reactivity system uses Symbol.for internally.

### Common Mistakes

- **Using `Symbol()` instead of `Symbol.for()` for sharing** — Regular Symbols are unique per call.
- **Assuming key collisions are impossible** — Use namespaced keys like `"lib.purpose"`.
- **Using `Symbol.keyFor` on non-registered Symbols** — Returns `undefined`.
- **Overusing the global registry** — Can become a maintenance burden.

### Best Practices

- Use `Symbol.for()` for Symbols that need to be shared across modules/realm.
- Use `Symbol()` for private, module-scoped Symbols.
- Use namespaced keys: `"libname.purpose"` to avoid collisions.
- Document registered Symbol keys in a central location.
- Avoid using the registry for transient/internal-only Symbols.

### Performance Considerations

- `Symbol.for()` lookup is a hash map operation (fast).
- `Symbol.keyFor()` is also fast.
- Registered Symbols are never garbage collected (they live forever).
- Heavy use of the global registry for dynamically generated keys can cause memory growth.
- Prefer private `Symbol()` for internal module use to avoid registry pollution.

### Interview Questions

1. **What is the difference between `Symbol()` and `Symbol.for()`?**
   `Symbol()` creates a new unique Symbol. `Symbol.for(key)` returns a registered Symbol or creates one, ensuring uniqueness by key.

2. **What is `Symbol.keyFor()` used for?**
   It retrieves the key string for a Symbol from the global registry.

3. **When would you use `Symbol.for()`?**
   When you need to share a Symbol across modules or JavaScript realms.

4. **Can `Symbol.keyFor()` return a key for any Symbol?**
   No, only Symbols created with `Symbol.for()` are in the registry. `Symbol.keyFor()` returns `undefined` for regular Symbols.

### Coding Challenges

1. **Implement a plugin system using `Symbol.for()` for lifecycle hooks.**
2. **Create a cross-realm event bus using registered Symbols.**
3. **Build a dependency injection container that uses Symbol.for keys.**
4. **Implement a `Symbol.keyFor` polyfill.**

### Related Topics

- `Symbol()` factory
- Cross-realm Symbols
- Module system
- Plugin architecture
- Global registry

---

## Symbol use cases

### What It Is

Symbols have a wide range of practical use cases beyond basic property keys. They enable meta-programming, protocol definition, safe extension of third-party objects, and implementation of custom language-level behaviors.

### Why It Is Important

Understanding Symbol use cases allows developers to:
- Write safer libraries that don't collide with user code.
- Implement custom iteration, string coercion, and type checking.
- Create extensible frameworks with well-defined extension points.
- Add metadata to objects without breaking existing code.
- Leverage JavaScript's meta-programming capabilities.

### How It Works Internally

Each use case leverages different Symbol properties:
- `Symbol.iterator` — Enables `for...of` integration.
- `Symbol.toPrimitive` — Controls type coercion.
- `Symbol.for()` — Enables cross-module Symbol sharing.
- Unique property keys — Ensures no collisions with string keys.

### Syntax

```javascript
// Various Symbol use cases
const sym = Symbol("unique"); // Private-like property
const shared = Symbol.for("shared"); // Shared across modules
const obj = {
  [Symbol.iterator]: function* () {}, // Custom iteration
  [Symbol.toPrimitive](hint) {}, // Type coercion
};
```

### Beginner Examples

```javascript
// Use Case 1: Safe metadata on DOM elements
const clickCount = Symbol("clickCount");
const button = document.querySelector("button");
button[clickCount] = 0;
button.addEventListener("click", () => {
  button[clickCount]++;
  console.log(`Clicked ${button[clickCount]} times`);
});

// Use Case 2: Enum-like constants
const Status = {
  PENDING: Symbol("pending"),
  APPROVED: Symbol("approved"),
  REJECTED: Symbol("rejected"),
};

function getStatusMessage(status) {
  switch (status) {
    case Status.PENDING: return "Waiting for review";
    case Status.APPROVED: return "Approved!";
    case Status.REJECTED: return "Rejected";
    default: return "Unknown";
  }
}

// Use Case 3: Custom toString
class Point {
  constructor(x, y) { this.x = x; this.y = y; }
  get [Symbol.toStringTag]() { return "Point"; }
  [Symbol.toPrimitive](hint) {
    if (hint === "string") return `(${this.x}, ${this.y})`;
    if (hint === "number") return Math.sqrt(this.x**2 + this.y**2);
    return null;
  }
}
```

### Intermediate Examples

```javascript
// Use Case 1: Avoiding property name clashes in mixins
const mixinId = Symbol("mixinId");
function TimestampMixin(Base) {
  return class extends Base {
    constructor(...args) {
      super(...args);
      this[mixinId] = Date.now();
    }
    get createdAt() { return this[mixinId]; }
  };
}

// Use Case 2: Hidden internal state
const _state = Symbol("state");
class FSM {
  constructor(initial) {
    this[_state] = initial;
  }
  get state() { return this[_state]; }
  transition(newState) {
    const old = this[_state];
    this[_state] = newState;
    this.onTransition?.(old, newState);
  }
}

// Use Case 3: Custom serialization hints
const serializationKey = Symbol("serialize");
class User {
  constructor(name, email, password) {
    this.name = name;
    this.email = email;
    this[serializationKey] = { name, email }; // exclude password
  }
  toJSON() { return this[serializationKey]; }
}
```

### Advanced Examples

```javascript
// Use Case 1: Metaprogramming with Symbols
const methods = Object.getOwnPropertySymbols(Math);
console.log(methods); // Symbols on Math (none by default)

// Use Case 2: Aspect-oriented programming with Symbols
const beforeHooks = Symbol("before");
const afterHooks = Symbol("after");

function hookMethod(obj, methodName, hook, when = "after") {
  const original = obj[methodName];
  const hookStore = when === "before" ? beforeHooks : afterHooks;

  if (!obj[hookStore]) obj[hookStore] = new Map();
  if (!obj[hookStore].has(methodName)) {
    obj[hookStore].set(methodName, []);
    obj[methodName] = function (...args) {
      (obj[beforeHooks]?.get(methodName) || []).forEach(h => h.apply(this, args));
      const result = original.apply(this, args);
      (obj[afterHooks]?.get(methodName) || []).forEach(h => h.call(this, result));
      return result;
    };
  }
  obj[hookStore].get(methodName).push(hook);
}

// Use Case 3: Symbol-based dependency injection
const INJECT = Symbol.for("di.inject");
class Container {
  constructor() {
    this.services = new Map();
  }
  register(token, factory) {
    this.services.set(token, factory);
  }
  resolve(target) {
    const deps = target[INJECT] || [];
    const args = deps.map(dep => {
      const factory = this.services.get(dep);
      if (!factory) throw new Error(`Dependency not found: ${dep.toString()}`);
      return factory();
    });
    return new target(...args);
  }
}

const DB_TOKEN = Symbol.for("db");
const LOGGER_TOKEN = Symbol.for("logger");

class UserService {
  static [INJECT] = [DB_TOKEN, LOGGER_TOKEN];
  constructor(db, logger) {
    this.db = db;
    this.logger = logger;
  }
}
```

### Real-World Use Cases

- **React** — Uses `Symbol.for('react.element')` for element type identification.
- **Redux** — Uses Symbols for internal action types.
- **Iteration protocols** — All `for...of` usage depends on `Symbol.iterator`.
- **Serialization** — Custom `toJSON` via `Symbol.toStringTag`.
- **Testing frameworks** — Internal Symbol keys for test metadata.

### Common Mistakes

- **Overusing Symbols** — Not everything needs to be a Symbol; strings are fine for most APIs.
- **Expecting true privacy** — Symbols are not private; they just hide from common enumeration.
- **Forgetting cross-realm issues** — `Symbol()` from different realms are not equal.
- **Using Symbols for serializable data** — JSON.stringify ignores Symbol keys.
- **Creating too many unique Symbols** — Can degrade performance in hot paths.

### Best Practices

- Use `Symbol()` for internal/collision-free property keys.
- Use `Symbol.for()` for shared protocols and cross-realm scenarios.
- Use well-known symbols for integrating with language features.
- Avoid Symbols for APIs that need to be serialized.
- Document Symbol usage in your library's API.

### Performance Considerations

- Symbol-keyed properties are fast to access.
- `Object.getOwnPropertySymbols()` is O(n) where n is number of Symbol keys.
- Symbols cannot be accessed with dot notation (always bracket).
- V8 stores Symbol properties separately from string properties.
- Using thousands of unique Symbols as keys on a single object is not recommended.

### Interview Questions

1. **What are practical use cases for Symbols?**
   Avoiding property collisions, defining protocols (iterator, toStringTag), enum-like constants, and cross-realm shared symbols.

2. **When would you NOT use a Symbol?**
   When you need serialization (JSON), when the key should be enumerable, or for simple property access that doesn't need uniqueness.

3. **Can Symbols be used as keys in WeakMap?**
   Yes, Symbols can be WeakMap keys (since ES2023).

4. **How do Symbols help with library design?**
   They allow libraries to attach metadata to user objects without fear of name collisions.

### Coding Challenges

1. **Create a simple ORM that uses Symbols for internal model metadata.**
2. **Implement a caching system using Symbols as cache keys.**
3. **Build a plugin architecture using `Symbol.for()` for lifecycle hooks.**
4. **Create a serialization system that handles Symbol-keyed properties.**
5. **Implement an event emitter that uses Symbols for event types.**

### Related Topics

- `Symbol()` factory
- Well-known symbols
- Symbol registry
- Property descriptors
- `Object.getOwnPropertySymbols()`
- Metaprogramming
