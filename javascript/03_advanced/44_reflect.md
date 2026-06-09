# Reflect

## Introduction

The Reflect API is a built-in object that provides methods for interceptable JavaScript operations. Introduced in ES6, Reflect methods correspond one-to-one with Proxy trap methods and provide a set of sensible defaults for forwarding operations. Reflect makes meta-programming cleaner, more reliable, and less error-prone compared to using older patterns like `Function.prototype.apply.call()`.

---

## Reflect.get() and Reflect.set()

### What It Is

`Reflect.get(target, propertyKey, receiver)` reads a property from an object, akin to `target[propertyKey]`. `Reflect.set(target, propertyKey, value, receiver)` sets a property, akin to `target[propertyKey] = value`. Both accept an optional `receiver` argument that sets the `this` value for getters/setters in the prototype chain.

```javascript
const obj = { x: 1, get y() { return this.x + 1; } };

console.log(Reflect.get(obj, "x")); // 1
console.log(Reflect.get(obj, "y")); // 2

const receiver = { x: 10 };
console.log(Reflect.get(obj, "y", receiver)); // 11 (getter uses receiver as this)

Reflect.set(obj, "x", 42);
console.log(obj.x); // 42
```

### Why It Is Important

`Reflect.get/set` improve upon direct property access:
- They work with the `receiver` argument to control `this` in getters/setters.
- They return a boolean from `set` indicating success/failure.
- They throw fewer exceptions than direct access.
- They are the correct way to forward operations in Proxy traps.
- They behave consistently with the language specification.

### How It Works Internally

`Reflect.get` follows the same internal `[[Get]]` algorithm as the `obj.prop` syntax. It traverses the prototype chain, invokes getters with the receiver as `this`, and returns the value. `Reflect.set` follows `[[Set]]`, checking writability, invoking setters, and returns a boolean. The receiver argument propagates through the chain.

### Syntax

```javascript
Reflect.get(target, propertyKey, receiver);
Reflect.set(target, propertyKey, value, receiver);
// Returns: boolean (true if set succeeded)
```

### Beginner Examples

```javascript
// Example 1: Basic usage
const obj = { a: 1, b: 2 };
console.log(Reflect.get(obj, "a")); // 1
console.log(Reflect.set(obj, "c", 3)); // true
console.log(obj.c); // 3

// Example 2: Non-writable properties
Object.defineProperty(obj, "readonly", { value: 42, writable: false });
console.log(Reflect.set(obj, "readonly", 99)); // false (failed)
console.log(obj.readonly); // 42 (unchanged)

// Example 3: Strict vs non-strict mode
// Reflect.set always returns boolean, no exception on failure
// (unlike direct assignment in strict mode which throws)

// Example 4: Receiver with getters
const parent = { get value() { return this._value; } };
const child = { _value: 5 };
console.log(Reflect.get(parent, "value", child)); // 5 (child is this)
```

### Intermediate Examples

```javascript
// Example 1: Forwarding in proxies
const handler = {
  get(target, prop, receiver) {
    console.log(`Getting ${String(prop)}`);
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, value, receiver) {
    console.log(`Setting ${String(prop)} to ${value}`);
    return Reflect.set(target, prop, value, receiver);
  },
};

const proxy = new Proxy({ x: 1 }, handler);
proxy.x; // "Getting x"
proxy.x = 2; // "Setting x to 2"

// Example 2: With computed names
const key = Symbol("dynamic");
const obj = { [key]: "symbol value" };
console.log(Reflect.get(obj, key)); // "symbol value"

// Example 3: receiver for prototype inheritance
const proto = {
  get greeting() { return `Hello, ${this.name}`; },
};
const instance = Object.create(proto);
instance.name = "Alice";

console.log(Reflect.get(instance, "greeting")); // "Hello, Alice"
// Without receiver, Reflect.get would use instance itself
```

### Advanced Examples

```javascript
// Example 1: Custom receiver for dependency tracking
function reactive(obj) {
  const deps = new Map();

  return new Proxy(obj, {
    get(target, prop, receiver) {
      if (!deps.has(prop)) deps.set(prop, new Set());
      // Track dependency (if in a tracking context)
      if (currentWatcher) deps.get(prop).add(currentWatcher);
      return Reflect.get(target, prop, receiver);
    },
    set(target, prop, value, receiver) {
      const result = Reflect.set(target, prop, value, receiver);
      if (deps.has(prop)) {
        deps.get(prop).forEach(watcher => watcher());
      }
      return result;
    },
  });
}

// Example 2: Safe property access with default
function getWithDefault(target, prop, defaultValue) {
  const result = Reflect.get(target, prop);
  return result !== undefined ? result : defaultValue;
}

// Example 3: Cross-realm property access
// Reflect.get/set work across realms unlike some direct operations
const iframe = document.createElement("iframe");
document.body.appendChild(iframe);
const crossObj = iframe.contentWindow.eval("({x: 1, y: 2})");
console.log(Reflect.get(crossObj, "x")); // 1 (works cross-realm)
```

### Real-World Use Cases

- **Proxy handlers** — Forwarding operations in get/set traps.
- **Reactive systems** — Dependency tracking in state management.
- **Meta-programming** — Dynamic property access with controlled `this`.
- **Testing** — Verifying property access patterns.
- **Safe object manipulation** — Using booleans instead of exceptions.

### Common Mistakes

- **Forgetting receiver in get/set** — Without it, getters/setters lose context.
- **Assuming Reflect.set always returns true** — Returns false for non-writable properties.
- **Not checking Reflect.set return** — Silent failures on frozen/sealed objects.
- **Confusing Reflect.get with direct access** — They mostly behave the same.

### Best Practices

- Always use `Reflect.get/set` inside Proxy traps (with receiver).
- Check the return value of `Reflect.set` for error handling.
- Use the receiver argument when forwarding through prototype chains.
- Prefer `Reflect.get/set` for meta-programming over computed property access.

### Performance Considerations

- `Reflect.get/set` have similar performance to direct property access.
- V8 optimizes both equally in modern engines.
- Adding receiver adds minimal overhead.
- Proxy forwarding with Reflect is faster than manual implementations.

### Interview Questions

1. **What is the difference between `obj.prop` and `Reflect.get(obj, 'prop')`?**
   Minimal for simple cases. `Reflect.get` allows a receiver argument for getter context.

2. **Why use `Reflect.get` inside a Proxy handler?**
   To properly forward the operation with the correct receiver (usually the proxy, not the target).

3. **What does `Reflect.set` return?**
   A boolean: `true` if the property was set, `false` if it couldn't be set.

4. **How does the receiver argument work?**
   It sets the `this` value for getters/setters found in the prototype chain.

### Coding Challenges

1. **Create a `getWithPath` function using `Reflect.get` for nested property access.**
2. **Implement a simple reactive system using `Reflect.get/set` with tracking.**
3. **Build a safe object updater that reports which properties failed to set.**

### Related Topics

- Reflect API
- Proxy
- Property descriptors
- Getters/setters
- Meta-programming

---

## Reflect.has() and Reflect.deleteProperty()

### What It Is

`Reflect.has(target, propertyKey)` corresponds to the `in` operator, returning a boolean. `Reflect.deleteProperty(target, propertyKey)` corresponds to the `delete` operator, returning a boolean indicating success.

```javascript
const obj = { a: 1, b: 2 };
console.log(Reflect.has(obj, "a")); // true
console.log(Reflect.has(obj, "c")); // false

console.log(Reflect.deleteProperty(obj, "a")); // true
console.log(Reflect.has(obj, "a")); // false

// Deleting non-configurable property
Object.defineProperty(obj, "fixed", { value: 42, configurable: false });
console.log(Reflect.deleteProperty(obj, "fixed")); // false
```

### Why It Is Important

These methods provide:
- A programmatic `in` operator that works with Proxy traps.
- A reliable `delete` that returns boolean (like `delete` in strict mode).
- Consistent behavior with the language specification.
- Safe forwarding in Proxy handlers.

### How It Works Internally

`Reflect.has` follows the internal `[[HasProperty]]` algorithm, traversing the prototype chain. `Reflect.deleteProperty` follows `[[Delete]]`, which checks configurable and strict mode. Both return booleans instead of throwing.

### Syntax

```javascript
Reflect.has(target, propertyKey);
// Returns: boolean

Reflect.deleteProperty(target, propertyKey);
// Returns: boolean (true if deleted or property didn't exist)
```

### Beginner Examples

```javascript
// Example 1: has vs in operator
const obj = { x: 1 };
console.log("x" in obj); // true
console.log(Reflect.has(obj, "x")); // true
console.log("toString" in obj); // true
console.log(Reflect.has(obj, "toString")); // true (prototype chain)

// Example 2: delete vs deleteProperty
const obj = { a: 1, b: 2 };
console.log(Reflect.deleteProperty(obj, "a")); // true
console.log(Reflect.deleteProperty(obj, "c")); // true (non-existent is success)

// Example 3: Non-configurable property
Object.defineProperty(obj, "fixed", { value: 42, configurable: false });
console.log(Reflect.deleteProperty(obj, "fixed")); // false
```

### Intermediate Examples

```javascript
// Example 1: Forwarding in Proxy
const handler = {
  has(target, prop) {
    console.log(`Checking ${String(prop)}`);
    return Reflect.has(target, prop);
  },
  deleteProperty(target, prop) {
    console.log(`Deleting ${String(prop)}`);
    return Reflect.deleteProperty(target, prop);
  },
};

const proxy = new Proxy({ x: 1, y: 2 }, handler);
console.log("x" in proxy); // "Checking x" true
delete proxy.y; // "Deleting y" true

// Example 2: has in prototype chain
const parent = { inherited: true };
const child = Object.create(parent);
child.own = true;

console.log(Reflect.has(child, "own")); // true
console.log(Reflect.has(child, "inherited")); // true (from prototype)
console.log(Reflect.has(child, "nonexistent")); // false

// Example 3: delete with frozen objects
const frozen = Object.freeze({ a: 1 });
console.log(Reflect.deleteProperty(frozen, "a")); // false (frozen)
```

### Advanced Examples

```javascript
// Example 1: Validation with has
const schema = {
  name: { required: true, type: "string" },
  age: { required: false, type: "number" },
};

function createValidatedObject(data, schema) {
  const obj = { ...data };
  return new Proxy(obj, {
    has(target, prop) {
      if (!Reflect.has(schema, prop)) return false;
      return Reflect.has(target, prop) || schema[prop]?.required;
    },
    get(target, prop) {
      if (Reflect.has(target, prop)) {
        return Reflect.get(target, prop);
      }
      if (schema[prop]?.required) {
        return undefined; // or throw
      }
      return undefined;
    },
  });
}

// Example 2: Soft delete tracking
function withSoftDelete(obj) {
  const deleted = new Set();
  return new Proxy(obj, {
    deleteProperty(target, prop) {
      if (Reflect.has(target, prop)) {
        deleted.add(prop);
        return true;
      }
      return false;
    },
    has(target, prop) {
      if (deleted.has(prop)) return false;
      return Reflect.has(target, prop);
    },
    get(target, prop) {
      if (deleted.has(prop)) return undefined;
      return Reflect.get(target, prop);
    },
  });
}
```

### Real-World Use Cases

- **ORM delete guards** — Preventing deletion of protected fields.
- **Access control** — Conditional `has` for hidden properties.
- **Versioned objects** — Soft delete with `deleteProperty` tracking.
- **Schema validation** — Checking property existence against a schema.
- **Proxy forwarding** — Default handlers for `has` and `deleteProperty` traps.

### Common Mistakes

- **Confusing `Reflect.has` with `Object.hasOwn`** — `has` checks the prototype chain.
- **Assuming `deleteProperty` works on non-configurable properties** — Returns false.
- **Not checking return value** — Deleting from frozen objects silently fails.
- **Using `deleteProperty` on primitives** — May throw.

### Best Practices

- Use `Reflect.has` instead of the `in` operator in meta-programming.
- Use `Reflect.deleteProperty` instead of the `delete` operator in Proxy handlers.
- Always check the return boolean for error handling.
- Use `Object.hasOwn` when you need own-property-only checks.

### Performance Considerations

- `Reflect.has` is as fast as the `in` operator.
- `Reflect.deleteProperty` is as fast as the `delete` operator.
- Both are optimized inline by V8.
- Using them inside proxys adds minimal overhead.

### Interview Questions

1. **What does `Reflect.has` correspond to?** The `in` operator.
2. **What does `Reflect.deleteProperty` correspond to?** The `delete` operator.
3. **Why use `Reflect.deleteProperty` instead of `delete`?** Returns a boolean and works well with Proxy forwarding.
4. **Does `Reflect.has` check the prototype chain?** Yes, like the `in` operator.

### Coding Challenges

1. **Create a `safeDelete` function using `Reflect.deleteProperty` that reports failures.**
2. **Implement a proxy that hides properties with `Reflect.has`.**
3. **Build a read-only view of an object using `Reflect.has` and `Reflect.get`.**

### Related Topics

- `in` operator
- `delete` operator
- Proxy `has` and `deleteProperty` traps
- `Object.hasOwn()`

---

## Reflect.construct() and Reflect.apply()

### What It Is

`Reflect.construct(target, args, newTarget)` calls a constructor with the given arguments, equivalent to `new target(...args)`. The optional `newTarget` sets the `new.target` value. `Reflect.apply(target, thisArgument, argumentsList)` calls a function with a given `this` value and arguments array, equivalent to `target.apply(thisArg, args)`.

```javascript
function Person(name) { this.name = name; }
const alice = Reflect.construct(Person, ["Alice"]);
console.log(alice.name); // "Alice"
console.log(alice instanceof Person); // true

function greet(greeting) { return `${greeting}, ${this.name}`; }
console.log(Reflect.apply(greet, alice, ["Hello"])); // "Hello, Alice"
```

### Why It Is Important

These methods provide:
- A cleaner way to call constructors dynamically than spread+new.
- Control over `new.target` for inheritance scenarios.
- A more reliable alternative to `Function.prototype.apply.call`.
- Consistent forwarding in Proxy `construct` and `apply` traps.
- No need to use `bind` or temporary objects for function calls.

### How It Works Internally

`Reflect.construct` follows the `[[Construct]]` internal method. With `newTarget`, it changes what `new.target` resolves to in the constructor body. `Reflect.apply` follows `[[Call]]`, setting `this` and spreading the arguments.

### Syntax

```javascript
Reflect.construct(target, args, newTarget);
// Returns: new instance

Reflect.apply(target, thisArgument, argumentsList);
// Returns: function result
```

### Beginner Examples

```javascript
// Example 1: Dynamic construction
const types = { Person, Car, Animal };
function create(type, ...args) {
  const Constructor = types[type];
  if (!Constructor) throw new Error("Unknown type");
  return Reflect.construct(Constructor, args);
}

// Example 2: Apply with array
const numbers = [1, 5, 3, 9, 2];
console.log(Reflect.apply(Math.max, null, numbers)); // 9

// Example 3: Apply with this
const obj = { multiplier: 2 };
function multiply(arr) {
  return arr.map(x => x * this.multiplier);
}
console.log(Reflect.apply(multiply, obj, [[1, 2, 3]])); // [2, 4, 6]
```

### Intermediate Examples

```javascript
// Example 1: new.target with construct
class Parent {
  constructor() {
    console.log(new.target.name);
  }
}
class Child extends Parent {}

// Normally new Child() would set new.target = Child
// With Reflect.construct we can override:
Reflect.construct(Parent, [], Child); // Logs "Child"

// Example 2: Avoiding the spread operator limitation
function sum(a, b, c) { return a + b + c; }
const args = [1, 2, 3];
console.log(Reflect.apply(sum, null, args)); // 6
// VS the old way:
console.log(Function.prototype.apply.call(sum, null, args)); // 6

// Example 3: Practical apply for array methods on array-like
const arrayLike = { 0: "a", 1: "b", length: 2 };
const result = Reflect.apply(Array.prototype.map, arrayLike, [
  x => x.toUpperCase(),
]);
console.log(result); // ["A", "B"]
```

### Advanced Examples

```javascript
// Example 1: Proxy forwarding
const handler = {
  apply(target, thisArg, args) {
    console.log(`Called with:`, args);
    return Reflect.apply(target, thisArg, args);
  },
  construct(target, args, newTarget) {
    console.log(`Constructed with:`, args);
    return Reflect.construct(target, args, newTarget);
  },
};

const ProxyFn = new Proxy(function (x) { return x * 2; }, handler);
console.log(ProxyFn(5)); // "Called with: [5]" then 10

const ProxyClass = new Proxy(Person, handler);
const p = new ProxyClass("Bob");
console.log(p.name); // "Bob"

// Example 2: Abstract factory with construct
class ServiceFactory {
  constructor(services) {
    this.services = services;
  }

  create(name, ...args) {
    const Service = this.services[name];
    if (!Service) throw new Error(`Service ${name} not found`);
    return Reflect.construct(Service, args);
  }
}

// Example 3: Using construct to bypass new.target restrictions
class Base {
  constructor() {
    if (new.target === Base) {
      throw new Error("Cannot instantiate abstract class");
    }
  }
}
// This bypasses the restriction:
// Reflect.construct(Base, []) // throws
class Concrete extends Base {}
const instance = Reflect.construct(Base, [], Concrete);
// Works because new.target is Concrete, not Base
```

### Real-World Use Cases

- **Dependency injection** — Creating instances dynamically.
- **Proxy-based middleware** — Intercepting function calls and constructions.
- **Polymorphic factories** — Choosing constructors at runtime.
- **Method borrowing** — Calling array methods on array-like objects.
- **Testing** — Mocking constructors and function calls.

### Common Mistakes

- **Forgetting `newTarget` in inheritance context** — Breaks `new.target` based logic.
- **Not spreading arguments for apply** — `Reflect.apply` expects an array.
- **Using `Reflect.apply` where direct call suffices** — Adds unnecessary complexity.
- **Assuming `Reflect.construct` works on non-constructors** — Throws TypeError.

### Best Practices

- Use `Reflect.apply` over `.call()` or `.apply()` in Proxy handlers.
- Use `Reflect.construct` for dynamic instantiation.
- Pass `newTarget` in `construct` traps for correct inheritance.
- Prefer direct function calls over Reflect.apply for simple cases.

### Performance Considerations

- `Reflect.apply` is as fast as `.apply()` and faster than `Function.prototype.apply.call`.
- `Reflect.construct` is comparable to `new` operator.
- V8 has optimized paths for both.
- Using them inside Proxy handlers adds minimal overhead.

### Interview Questions

1. **What is the difference between `Reflect.apply` and `fn.apply()`?**
   `Reflect.apply` doesn't require the function to have an `apply` method (works with Proxies).

2. **What is the `newTarget` argument in `Reflect.construct`?**
   It sets the value of `new.target` inside the constructor, enabling correct inheritance.

3. **When would you use `Reflect.construct` over the `new` operator?**
   When you need to control `new.target` or when the constructor is determined dynamically.

4. **How do you forward `apply` and `construct` in Proxy handlers?**
   Use `Reflect.apply(target, thisArg, args)` and `Reflect.construct(target, args, newTarget)`.

### Coding Challenges

1. **Create a function `constructFromArray(className, args)` that dynamically constructs instances.**
2. **Implement a Proxy that logs all function calls and constructor invocations.**
3. **Build a method `borrowMethod` that uses `Reflect.apply` to borrow array methods.**
4. **Create an abstract factory using `Reflect.construct`.**

### Related Topics

- `new` operator
- Function apply/call/bind
- Proxy `apply` and `construct` traps
- `new.target`
- Class inheritance

---

## Comparison with Proxy

### What It Is

Reflect and Proxy are complementary APIs introduced in ES6. Proxy defines traps (interceptors) for object operations. Reflect provides default implementations for those traps. Together, they enable clean, complete meta-programming: Proxy intercepts, Reflect forwards.

```javascript
const handler = {
  get(target, prop, receiver) {
    // Custom logic
    return Reflect.get(target, prop, receiver); // Forward
  },
};

const proxy = new Proxy(target, handler);
```

### Why It Is Important

Understanding the relationship between Reflect and Proxy matters because:
- Reflect methods are the standard way to forward Proxy traps.
- Without Reflect, forwarding requires complex manual implementation.
- Reflect does not require a Proxy — it works on any object.
- Proxy without Reflect can miss edge cases (proxied prototypes, receivers).
- Together they form the meta-programming foundation of modern JavaScript.

### How It Works Internally

Every Proxy trap corresponds to a Reflect method. When you call `Reflect.get(target, prop, receiver)`, it executes the same internal `[[Get]]` algorithm that the engine uses. When intercepting with a Proxy, calling `Reflect.get(target, prop, receiver)` inside the `get` trap ensures the operation is forwarded correctly with the right receiver.

### Syntax

```javascript
const handler = {
  // Each trap corresponds to a Reflect method
  get: Reflect.get,
  set: Reflect.set,
  has: Reflect.has,
  deleteProperty: Reflect.deleteProperty,
  apply: Reflect.apply,
  construct: Reflect.construct,
  getPrototypeOf: Reflect.getPrototypeOf,
  setPrototypeOf: Reflect.setPrototypeOf,
  isExtensible: Reflect.isExtensible,
  preventExtensions: Reflect.preventExtensions,
  getOwnPropertyDescriptor: Reflect.getOwnPropertyDescriptor,
  defineProperty: Reflect.defineProperty,
  ownKeys: Reflect.ownKeys,
};
```

### Beginner Examples

```javascript
// Example 1: Default forwarding handler
const handler = {
  get: Reflect.get,
  set: Reflect.set,
  has: Reflect.has,
  deleteProperty: Reflect.deleteProperty,
};

const proxy = new Proxy({ a: 1, b: 2 }, handler);
// Works exactly like the target

// Example 2: Adding logging via Proxy + Reflect
const logHandler = {
  get(target, prop, receiver) {
    console.log(`GET ${String(prop)}`);
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, value, receiver) {
    console.log(`SET ${String(prop)} = ${value}`);
    return Reflect.set(target, prop, value, receiver);
  },
};

// Example 3: Validation with Proxy + Reflect
const validateHandler = {
  set(target, prop, value) {
    if (prop === "age" && (typeof value !== "number" || value < 0)) {
      throw new TypeError("Age must be a positive number");
    }
    return Reflect.set(target, prop, value);
  },
};
```

### Intermediate Examples

```javascript
// Example 1: Revocable proxy with Reflect forwarding
const { proxy, revoke } = Proxy.revocable(target, {
  get: Reflect.get,
  set: Reflect.set,
});

// Example 2: Transparent proxying
function createPassthroughProxy(obj) {
  return new Proxy(obj, {
    get: Reflect.get,
    set: Reflect.set,
    has: Reflect.has,
    deleteProperty: Reflect.deleteProperty,
    ownKeys: Reflect.ownKeys,
    getOwnPropertyDescriptor: Reflect.getOwnPropertyDescriptor,
    defineProperty: Reflect.defineProperty,
  });
}

// Example 3: Proxy with receiver forwarding (correct)
const handler = {
  get(target, prop, receiver) {
    // Bad: return target[prop]; // Ignores receiver
    // Good:
    return Reflect.get(target, prop, receiver);
  },
};

const proto = { get value() { return this.x; } };
const obj = Object.create(proto);
obj.x = 10;
const proxy = new Proxy(obj, handler);
console.log(proxy.value); // 10 (works because receiver is forwarded)
```

### Advanced Examples

```javascript
// Example 1: Full transparent proxy (all traps)
function createFullProxy(target) {
  const handler = {};
  for (const method of Object.getOwnPropertyNames(Reflect)) {
    if (typeof Reflect[method] === "function") {
      handler[method] = Reflect[method];
    }
  }
  // Map trap names to Reflect methods
  const trapMap = {
    getPrototypeOf: "getPrototypeOf",
    setPrototypeOf: "setPrototypeOf",
    isExtensible: "isExtensible",
    preventExtensions: "preventExtensions",
    getOwnPropertyDescriptor: "getOwnPropertyDescriptor",
    defineProperty: "defineProperty",
    has: "has",
    get: "get",
    set: "set",
    deleteProperty: "deleteProperty",
    ownKeys: "ownKeys",
    apply: "apply",
    construct: "construct",
  };
  const finalHandler = {};
  for (const [trap, reflectMethod] of Object.entries(trapMap)) {
    finalHandler[trap] = Reflect[reflectMethod];
  }
  return new Proxy(target, finalHandler);
}

// Example 2: Proxy + Reflect for immutable objects
function immutable(obj) {
  return new Proxy(obj, {
    set() { return false; },
    deleteProperty() { return false; },
    defineProperty() { return false; },
    setPrototypeOf() { return false; },
    preventExtensions() { return true; },
    get: Reflect.get,
    has: Reflect.has,
    ownKeys: Reflect.ownKeys,
    getOwnPropertyDescriptor: Reflect.getOwnPropertyDescriptor,
  });
}

const frozen = immutable({ x: 1 });
console.log(frozen.x); // 1
// frozen.x = 2; // silently fails

// Example 3: Computed properties via Proxy + Reflect
function computed(obj, compute) {
  return new Proxy(obj, {
    get(target, prop, receiver) {
      if (prop in compute) {
        return Reflect.apply(compute[prop], receiver, []);
      }
      return Reflect.get(target, prop, receiver);
    },
    has(target, prop) {
      return prop in compute || Reflect.has(target, prop);
    },
    ownKeys(target) {
      return [...Reflect.ownKeys(target), ...Reflect.ownKeys(compute)];
    },
  });
}

const user = computed(
  { firstName: "John", lastName: "Doe" },
  {
    fullName() { return `${this.firstName} ${this.lastName}`; },
  }
);
console.log(user.fullName); // "John Doe"
```

### Real-World Use Cases

- **State management** (MobX, Vue reactivity) — Proxy + Reflect for reactive objects.
- **Validation libraries** — Runtime schema validation via Proxy.
- **ORM lazy loading** — Proxies that lazily fetch related data.
- **API mocking** — Creating mock objects that record all interactions.
- **Sandboxing** — Restricting access to properties via Proxy.

### Common Mistakes

- **Not forwarding receiver** — Breaks getters in the prototype chain.
- **Using Proxy without Reflect** — Manually reimplementing [[Get]] is bug-prone.
- **Assuming all Proxy traps need custom logic** — Many handlers just need `Reflect.*`.
- **Forgetting that Reflect methods work without Proxy** — They are independent utilities.
- **Over-engineering with full proxies** — Simple cases don't need all traps.

### Best Practices

- Always use `Reflect.*` inside Proxy handlers (don't reimplement).
- Use the simplest handler that meets your needs.
- Forward the `receiver` argument in `get` and `set` traps.
- Use `Proxy.revocable()` when the proxy should have a limited lifespan.
- Prefer `Reflect.*` over manual alternatives for meta-programming.

### Performance Considerations

- Proxy adds overhead to every intercepted operation.
- Reflect methods inside Proxy handlers add negligible overhead vs manual forwarding.
- V8 optimizes common Proxy + Reflect patterns.
- Full transparent proxies (all traps) are slower than minimal proxies.
- Proxy-heavy code should be measured and optimized if needed.

### Interview Questions

1. **How do Reflect and Proxy relate to each other?**
   Proxy defines interception points (traps). Reflect provides default forwarding implementations for those traps.

2. **Why should you use Reflect inside Proxy handlers?**
   To correctly forward the operation with the proper receiver, preserving getter/setter context.

3. **Can you use Reflect without Proxy?**
   Yes. Reflect methods work on any object and are useful independently.

4. **What would happen if you didn't use Reflect in a get trap?**
   You would lose the receiver, which breaks prototype getter context.

### Coding Challenges

1. **Create a Proxy that logs all operations using Reflect forwarding.**
2. **Implement a `readonly` function using Proxy + Reflect.**
3. **Build a `deepProxy` that proxies nested objects using Reflect.**
4. **Create a `deprecated` proxy that warns when accessing certain properties.**
5. **Implement a `timed` proxy that measures access times using Reflect.**

### Related Topics

- Proxy
- Meta-programming
- Traps
- Property descriptors
- Receiver forwarding
- `Proxy.revocable()`
