# Meta Programming - Proxy, Reflect, Symbol, property descriptors, defineProperty

## Introduction

Metaprogramming is the practice of writing code that operates on other code—inspecting, modifying, or even creating code at runtime. JavaScript supports metaprogramming through several mechanisms: `Proxy` and `Reflect` (intercepting fundamental operations), `Symbol` (unique property keys with protocol-like behavior), and property descriptors (`defineProperty`, `getOwnPropertyDescriptor`). These tools enable frameworks, libraries, and advanced patterns that would otherwise be impossible.

## Proxy for Meta-Programming

### What It Is

A `Proxy` creates a wrapper around an object that intercepts and customizes fundamental operations (property access, assignment, enumeration, function invocation, etc.). The proxy defines "traps" (handler methods) that are called when operations are performed on the proxy, allowing custom behavior to replace or augment the default operation.

### Why It Is Important

Proxy enables capabilities that previously required language-level changes: reactivity (Vue 3), data binding, validation, logging, lazy loading, caching, access control, and API mocking. Proxy is the foundation of modern reactive frameworks and can dramatically simplify cross-cutting concerns.

### How It Works Internally

A `Proxy` wraps a `target` object and a `handler` object. When an operation is performed on the proxy, the engine checks if the handler has a corresponding trap method. If yes, the trap runs with the target and operation details. If no, the operation is forwarded to the target directly.

```javascript
const target = { name: 'Alice' };
const handler = {
  get(target, prop, receiver) {
    console.log(`Getting ${String(prop)}`);
    return Reflect.get(target, prop, receiver);
  }
};

const proxy = new Proxy(target, handler);
console.log(proxy.name); // "Getting name" then "Alice"
```

### Syntax

```javascript
// Basic proxy structure
const target = {};
const handler = {
  get(target, prop, receiver) { /* intercept property reading */ },
  set(target, prop, value, receiver) { /* intercept property writing */ },
  has(target, prop) { /* intercept 'in' operator */ },
  deleteProperty(target, prop) { /* intercept delete */ },
  apply(target, thisArg, args) { /* intercept function calls */ },
  construct(target, args) { /* intercept new */ },
  getPrototypeOf(target) { /* intercept Object.getPrototypeOf */ },
  setPrototypeOf(target, proto) { /* intercept Object.setPrototypeOf */ },
  ownKeys(target) { /* intercept Object.keys/Reflect.ownKeys */ },
  getOwnPropertyDescriptor(target, prop) { /* intercept property descriptor */ },
  defineProperty(target, prop, descriptor) { /* intercept defineProperty */ },
  preventExtensions(target) { /* intercept preventExtensions */ },
  isExtensible(target) { /* intercept isExtensible */ },
  enumerate(target) { /* deprecated, use ownKeys */ }
};

const proxy = new Proxy(target, handler);
```

### Beginner Examples

```javascript
// Logging proxy
const user = { name: 'Alice', age: 30 };

const loggingProxy = new Proxy(user, {
  get(target, prop) {
    console.log(`GET ${String(prop)} = ${target[prop]}`);
    return target[prop];
  },
  set(target, prop, value) {
    console.log(`SET ${String(prop)} = ${value} (was ${target[prop]})`);
    target[prop] = value;
    return true;
  }
});

loggingProxy.name; // "GET name = Alice"
loggingProxy.age = 31; // "SET age = 31 (was 30)"

// Validation proxy
const validator = {
  set(target, prop, value) {
    if (prop === 'age') {
      if (typeof value !== 'number') throw new TypeError('Age must be a number');
      if (value < 0 || value > 150) throw new RangeError('Age out of range');
    }
    target[prop] = value;
    return true;
  }
};

const person = new Proxy({}, validator);
person.age = 30; // OK
// person.age = 'old'; // TypeError
// person.age = 200; // RangeError

// Default values proxy
const withDefaults = new Proxy({}, {
  get(target, prop) {
    if (!(prop in target)) {
      target[prop] = 0;
    }
    return target[prop];
  }
});

withDefaults.count++; // Works even though count didn't exist
console.log(withDefaults.count); // 1
```

### Intermediate Examples

```javascript
// Reactive proxy (like Vue 3 reactivity)
function reactive(target) {
  const effects = new Map();
  
  function track(prop) {
    if (activeEffect) {
      if (!effects.has(prop)) effects.set(prop, new Set());
      effects.get(prop).add(activeEffect);
    }
  }
  
  function trigger(prop) {
    const deps = effects.get(prop);
    if (deps) deps.forEach(effect => effect());
  }
  
  return new Proxy(target, {
    get(target, prop, receiver) {
      track(prop);
      return Reflect.get(target, prop, receiver);
    },
    set(target, prop, value, receiver) {
      const old = target[prop];
      const result = Reflect.set(target, prop, value, receiver);
      if (old !== value) trigger(prop);
      return result;
    }
  });
}

let activeEffect = null;
function watchEffect(fn) {
  activeEffect = fn;
  fn();
  activeEffect = null;
}

// Usage
const state = reactive({ count: 0, name: 'Alice' });

watchEffect(() => {
  console.log(`Count is: ${state.count}`);
});

state.count++; // "Count is: 1"
state.count = 5; // "Count is: 5"

// Read-only proxy
function readonly(target) {
  return new Proxy(target, {
    get(target, prop) {
      return target[prop];
    },
    set(target, prop, value) {
      console.warn(`Cannot set ${String(prop)} on read-only object`);
      return true; // Silently fail
    },
    deleteProperty(target, prop) {
      console.warn(`Cannot delete ${String(prop)} from read-only object`);
      return true;
    }
  });
}

const config = readonly({ apiKey: 'secret', endpoint: '/api' });
config.apiKey = 'new'; // Warning, doesn't change

// Method chaining proxy
class Calculator {
  constructor() {
    this.value = 0;
  }
  // ... methods
}

const chainable = new Proxy(new Calculator(), {
  get(target, prop) {
    const value = target[prop];
    if (typeof value === 'function') {
      return function(...args) {
        value.apply(target, args);
        return target; // Return the proxy for chaining
      };
    }
    return value;
  }
});

// Negative array indexing
const negativeArray = (arr) => new Proxy(arr, {
  get(target, prop) {
    const index = Number(prop);
    if (index < 0) {
      prop = String(target.length + index);
    }
    return Reflect.get(target, prop);
  }
});

const arr = negativeArray([10, 20, 30, 40]);
console.log(arr[-1]); // 40
console.log(arr[-2]); // 30
```

### Advanced Examples

```javascript
// Virtual properties with Proxy
const person = {
  firstName: 'John',
  lastName: 'Doe'
};

const virtualProxy = new Proxy(person, {
  get(target, prop) {
    if (prop === 'fullName') {
      return `${target.firstName} ${target.lastName}`;
    }
    if (prop === 'initials') {
      return `${target.firstName[0]}.${target.lastName[0]}.`;
    }
    return target[prop];
  }
});

console.log(virtualProxy.fullName); // "John Doe"
console.log(virtualProxy.initials); // "J.D."

// Revocable proxy (temporary access)
const { proxy, revoke } = Proxy.revocable(
  { secret: 'classified' },
  {
    get(target, prop) {
      if (prop === 'secret') {
        console.warn('Accessing secret');
      }
      return target[prop];
    }
  }
);

console.log(proxy.secret); // "classified" with warning
revoke();
// console.log(proxy.secret); // TypeError: Cannot perform 'get' on a proxy that has been revoked

// Auto-pipify proxy (for functional composition)
const pipify = (target) => new Proxy(target, {
  get(target, name) {
    if (name === 'pipe') {
      return (fn) => pipify(fn(target));
    }
    return target[name];
  }
});

const result = pipify(5)
  .pipe(x => x * 2)
  .pipe(x => x + 3)
  .pipe(x => `Result: ${x}`);

console.log(result); // "Result: 13"
// Note: result is still a proxy, use .valueOf() to get the actual value

// Deep proxy for nested reactivity
function deepProxy(target) {
  const cache = new WeakMap();
  
  function createProxy(obj) {
    if (typeof obj !== 'object' || obj === null) return obj;
    if (cache.has(obj)) return cache.get(obj);
    
    const proxy = new Proxy(obj, {
      get(target, prop, receiver) {
        const value = Reflect.get(target, prop, receiver);
        if (typeof value === 'object' && value !== null) {
          return createProxy(value); // Return nested proxy
        }
        return value;
      },
      set(target, prop, value, receiver) {
        console.log(`SET ${String(prop)} = ${value}`);
        return Reflect.set(target, prop, value, receiver);
      }
    });
    
    cache.set(obj, proxy);
    return proxy;
  }
  
  return createProxy(target);
}

// Performance measurement proxy
function measure(target) {
  return new Proxy(target, {
    apply(target, thisArg, args) {
      const start = performance.now();
      const result = Reflect.apply(target, thisArg, args);
      console.log(`${target.name} took ${performance.now() - start}ms`);
      return result;
    }
  });
}

const heavyFn = measure(function compute() {
  let sum = 0;
  for (let i = 0; i < 10000000; i++) sum += i;
  return sum;
});

heavyFn(); // "compute took 12.5ms"
```

## Reflect API

### What It Is

The `Reflect` object provides methods for performing JavaScript operations that correspond to the Proxy traps. `Reflect` methods are the default behavior for each Proxy trap—they forward the operation to the target object. Reflect also provides more reliable alternatives to some older JavaScript operations.

### Why It Is Important

Reflect methods are designed to work seamlessly with Proxy traps. They provide a clean, functional API for operations that previously had inconsistent return values or error handling (`delete`, `defineProperty`). Using `Reflect` in Proxy handlers ensures correct behavior for all edge cases, especially with inheritance and `this`.

### How It Works Internally

Each `Reflect` method corresponds to a Proxy trap and performs the default operation. For example, `Reflect.get(target, prop, receiver)` gets the property, and the `receiver` parameter is used for the `this` value in getters, ensuring correct behavior with inherited properties.

### Syntax

```javascript
// Reflect methods and their equivalents
Reflect.get(target, prop, receiver)      // target[prop]
Reflect.set(target, prop, value, receiver) // target[prop] = value
Reflect.has(target, prop)                // prop in target
Reflect.deleteProperty(target, prop)     // delete target[prop]
Reflect.construct(target, args)          // new target(...args)
Reflect.apply(target, thisArg, args)     // target.apply(thisArg, args)
Reflect.ownKeys(target)                  // Object.getOwnPropertyNames + getOwnPropertySymbols
Reflect.getPrototypeOf(target)           // Object.getPrototypeOf(target)
Reflect.setPrototypeOf(target, proto)    // Object.setPrototypeOf(target, proto)
Reflect.defineProperty(target, prop, desc) // Object.defineProperty
Reflect.getOwnPropertyDescriptor(target, prop) // Object.getOwnPropertyDescriptor
Reflect.preventExtensions(target)        // Object.preventExtensions
Reflect.isExtensible(target)             // Object.isExtensible
```

### Beginner Examples

```javascript
// Reflect basics
const obj = { name: 'Alice', age: 30 };

// Get
console.log(Reflect.get(obj, 'name')); // 'Alice'

// Set
Reflect.set(obj, 'age', 31);
console.log(obj.age); // 31

// Has
console.log(Reflect.has(obj, 'name')); // true

// Delete
Reflect.deleteProperty(obj, 'age');
console.log('age' in obj); // false

// Construct
class Person {
  constructor(name) { this.name = name; }
}
const p = Reflect.construct(Person, ['Bob']);
console.log(p.name); // 'Bob'

// Apply
const args = [1, 2, 3];
console.log(Reflect.apply(Math.max, null, args)); // 3

// OwnKeys (both string and symbol keys)
const sym = Symbol('secret');
const data = { a: 1, [sym]: 2 };
console.log(Reflect.ownKeys(data)); // ['a', Symbol(secret)]
```

### Intermediate Examples

```javascript
// Using Reflect in Proxy handlers (correct this binding)
const parent = {
  get value() { return this._value; },
  set value(v) { this._value = v; }
};

const child = new Proxy({ _value: 0 }, {
  get(target, prop, receiver) {
    // Without using receiver, 'this' in getters would be target, not receiver
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, value, receiver) {
    return Reflect.set(target, prop, value, receiver);
  }
});

Object.setPrototypeOf(child, parent);
child.value = 42;
console.log(child.value); // 42

// Without Reflect (WRONG):
// get: return target[prop] → this would be target, not child
// set: target[prop] = value → property set on target, not child

// Better than try/catch
// Old way:
try {
  Object.defineProperty(obj, 'prop', { value: 1 });
} catch (e) {
  // handle
}

// Reflect way:
if (Reflect.defineProperty(obj, 'prop', { value: 1 })) {
  console.log('Success');
} else {
  console.log('Failed');
}

// Checking property existence
// Old: prop in obj (throws if obj is null/undefined)
// New: Reflect.has(obj, prop) (safe)
```

### Advanced Examples

```javascript
// Forwarding proxies (exact default behavior)
function createPassthroughProxy(target) {
  return new Proxy(target, {
    get: Reflect.get,
    set: Reflect.set,
    has: Reflect.has,
    deleteProperty: Reflect.deleteProperty,
    ownKeys: Reflect.ownKeys,
    getOwnPropertyDescriptor: Reflect.getOwnPropertyDescriptor,
    defineProperty: Reflect.defineProperty,
    getPrototypeOf: Reflect.getPrototypeOf,
    setPrototypeOf: Reflect.setPrototypeOf,
    preventExtensions: Reflect.preventExtensions,
    isExtensible: Reflect.isExtensible,
    apply: Reflect.apply,
    construct: Reflect.construct
  });
}

// Custom handler that extends Reflect behavior
const smartHandler = {
  get(target, prop, receiver) {
    console.log(`Getting ${String(prop)}`);
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, value, receiver) {
    if (typeof value === 'string') {
      value = value.trim();
    }
    return Reflect.set(target, prop, value, receiver);
  },
  has(target, prop) {
    console.log(`Checking ${String(prop)}`);
    return Reflect.has(target, prop);
  },
  deleteProperty(target, prop) {
    console.log(`Deleting ${String(prop)}`);
    if (prop === 'id') {
      console.warn('Cannot delete id');
      return false;
    }
    return Reflect.deleteProperty(target, prop);
  }
};

// Reflect.construct with variable arguments
class Database {
  constructor(host, port, name) {
    this.connection = `${host}:${port}/${name}`;
  }
}

const args = ['localhost', 5432, 'mydb'];
const db = Reflect.construct(Database, args);
```

## Symbol Well-Known Symbols

### What It Is

Symbols are unique, immutable primitive values used as property keys. Well-known symbols (like `Symbol.iterator`, `Symbol.toStringTag`) are built-in symbols that allow customization of JavaScript's internal behaviors. They are the protocol mechanism of JavaScript—implementing a well-known symbol changes how an object behaves with language features.

### Why It Is Important

Well-known symbols enable objects to integrate with JavaScript's built-in protocols. `Symbol.iterator` makes objects iterable. `Symbol.toStringTag` customizes `Object.prototype.toString`. `Symbol.hasInstance` customizes `instanceof`. They provide a way to hook into language-level operations without conflicting with other property names.

### How It Works Internally

When JavaScript performs internal operations (like `for...of`, `instanceof`, `String()`), it checks for well-known symbols on the object. If the symbol property exists, the engine uses the provided function/value instead of the default behavior.

### Syntax

```javascript
// Creating symbols
const sym1 = Symbol('description');
const sym2 = Symbol('description');
console.log(sym1 === sym2); // false (unique)

// Symbol as property key
const obj = {
  [sym1]: 'secret value'
};
console.log(obj[sym1]); // 'secret value'
console.log(obj.sym1); // undefined (not string-keyed)

// Well-known symbols
Symbol.iterator    // Makes objects iterable (for...of, spread)
Symbol.toStringTag // Customizes Object.prototype.toString
Symbol.hasInstance // Customizes instanceof behavior
Symbol.species     // Controls derived class constructors
Symbol.match       // Customizes String.prototype.match
Symbol.replace     // Customizes String.prototype.replace
Symbol.search      // Customizes String.prototype.search
Symbol.split       // Customizes String.prototype.split
Symbol.toPrimitive // Customizes type coercion
Symbol.unscopables // Prevents property binding in with statements
Symbol.isConcatSpreadable // Controls Array.prototype.concat spread
Symbol.asyncIterator // Makes objects async iterable
Symbol.dispose     // Explicit resource management (ES2024)
```

### Beginner Examples

```javascript
// Symbol.iterator - make object iterable
const range = {
  start: 1,
  end: 5,
  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    return {
      next() {
        if (current <= end) {
          return { value: current++, done: false };
        }
        return { done: true };
      }
    };
  }
};

console.log([...range]); // [1, 2, 3, 4, 5]
for (const n of range) console.log(n); // 1, 2, 3, 4, 5

// Symbol.toStringTag
const myObj = {
  [Symbol.toStringTag]: 'MyCustomType'
};
console.log(Object.prototype.toString.call(myObj)); // '[object MyCustomType]'

// Symbol.hasInstance
class MyArray {
  static [Symbol.hasInstance](instance) {
    return Array.isArray(instance);
  }
}

console.log([] instanceof MyArray); // true
console.log({} instanceof MyArray); // false
```

### Intermediate Examples

```javascript
// Symbol.toPrimitive - custom type coercion
const temperature = {
  value: 30,
  unit: 'C',
  [Symbol.toPrimitive](hint) {
    if (hint === 'number') return this.value + 273.15; // Kelvin
    if (hint === 'string') return `${this.value}°${this.unit}`;
    return this.value; // default
  }
};

console.log(+temperature);    // 303.15 (number hint)
console.log(`${temperature}`); // '30°C' (string hint)
console.log(temperature + ''); // '30' (default hint)

// Symbol.species - control derived class constructors
class MyArray extends Array {
  static get [Symbol.species]() { return Array; }
}

const myArr = new MyArray(1, 2, 3);
const mapped = myArr.map(x => x * 2);
console.log(mapped instanceof MyArray); // false
console.log(mapped instanceof Array);   // true

// Symbol.match/replace/search/split
class UpperCaseSearch {
  [Symbol.search](str) {
    return str.toLowerCase().indexOf(this.toString().toLowerCase());
  }
}

const searchTerm = new UpperCaseSearch('WORLD');
console.log('Hello World'.search(searchTerm)); // 6

// Symbol.unscopables
const obj = { x: 1, y: 2, z: 3 };
obj[Symbol.unscopables] = { x: true };

with (obj) {
  console.log(y); // 2
  console.log(z); // 3
  // console.log(x); // ReferenceError (unscopable)
}
```

### Advanced Examples

```javascript
// Symbol.asyncIterator
const asyncRange = {
  start: 1,
  end: 5,
  delay: 100,
  [Symbol.asyncIterator]() {
    let current = this.start;
    const end = this.end;
    const delay = this.delay;
    return {
      async next() {
        await new Promise(r => setTimeout(r, delay));
        if (current <= end) {
          return { value: current++, done: false };
        }
        return { done: true };
      }
    };
  }
};

(async () => {
  for await (const n of asyncRange) {
    console.log(n); // 1, 2, 3, 4, 5 (with 100ms delay each)
  }
})();

// Symbol.dispose (ES2024 - Explicit Resource Management)
class FileHandle {
  constructor(path) { this.path = path; this.opened = true; }
  read() { /* ... */ }
  [Symbol.dispose]() {
    console.log(`Closing file: ${this.path}`);
    this.opened = false;
  }
}

// using (proposal)
// using file = new FileHandle('data.txt');
// file.read();
// // Automatically disposed when scope exits

// Symbol.isConcatSpreadable
const notSpreadable = { length: 2, 0: 'a', 1: 'b', [Symbol.isConcatSpreadable]: false };
console.log([1, 2].concat(notSpreadable)); // [1, 2, {0:'a', 1:'b', length: 2}]

const spreadable = { length: 2, 0: 'a', 1: 'b', [Symbol.isConcatSpreadable]: true };
console.log([1, 2].concat(spreadable)); // [1, 2, 'a', 'b']

// Global symbols (not unique)
const globalSym = Symbol.for('shared');
const sameGlobalSym = Symbol.for('shared');
console.log(globalSym === sameGlobalSym); // true

console.log(Symbol.keyFor(globalSym)); // 'shared'
```

## Property Descriptors and defineProperty

### What It Is

Property descriptors are objects that describe the configuration of a property on an object. They define whether a property is writable, enumerable, configurable, and either its value or getter/setter. `Object.defineProperty()` and `Object.defineProperties()` create or modify properties with fine-grained control.

### Why It Is Important

Property descriptors give developers precise control over object behavior. They enable:
- Read-only properties (writable: false)
- Hidden properties (enumerable: false)
- Non-deletable properties (configurable: false)
- Computed properties via getters/setters
- Preventing property changes (freezing/sealing)

### How It Works Internally

Every property has a descriptor with one of two forms:
- **Data descriptor**: `value`, `writable`, `enumerable`, `configurable`
- **Accessor descriptor**: `get`, `set`, `enumerable`, `configurable`

When you access a property, the engine checks its descriptor. For data properties, it returns `value` directly. For accessor properties, it calls the `get` function with `this` set to the object.

```javascript
// Internal representation
{
  value: 42,           // Data descriptor
  writable: true,      // Can be changed
  enumerable: true,    // Shows up in for...in
  configurable: true   // Can be deleted or redefined
}
// or
{
  get: function() {},  // Accessor descriptor
  set: function(v) {},
  enumerable: true,
  configurable: true
}
```

### Syntax

```javascript
// Define a data property
Object.defineProperty(obj, 'prop', {
  value: 42,
  writable: false,
  enumerable: true,
  configurable: false
});

// Define an accessor property
Object.defineProperty(obj, 'fullName', {
  get() { return `${this.first} ${this.last}`; },
  set(value) { [this.first, this.last] = value.split(' '); },
  enumerable: true,
  configurable: true
});

// Define multiple properties
Object.defineProperties(obj, {
  x: { value: 0, writable: true },
  y: { value: 0, writable: true },
  distance: {
    get() { return Math.sqrt(this.x ** 2 + this.y ** 2); }
  }
});

// Get property descriptor
const desc = Object.getOwnPropertyDescriptor(obj, 'prop');

// Shortcut methods
Object.freeze(obj);  // writable: false, configurable: false
Object.seal(obj);    // configurable: false
Object.preventExtensions(obj); // No new properties
```

### Beginner Examples

```javascript
// Read-only properties
const CONFIG = {};
Object.defineProperty(CONFIG, 'API_KEY', {
  value: 'abc123',
  writable: false,    // Cannot be changed
  configurable: false // Cannot be deleted
});

// CONFIG.API_KEY = 'new'; // Silently fails in non-strict, TypeError in strict

// Non-enumerable properties
const user = { name: 'Alice' };
Object.defineProperty(user, 'password', {
  value: 'secret123',
  enumerable: false,  // Hidden from Object.keys/for...in
  writable: true
});

console.log(Object.keys(user)); // ['name']
console.log(user.password); // 'secret123' (still accessible)
for (const key in user) console.log(key); // 'name' only

// Getters and setters
const temperature = {
  _celsius: 0,
  get fahrenheit() {
    return this._celsius * 9/5 + 32;
  },
  set fahrenheit(f) {
    this._celsius = (f - 32) * 5/9;
  }
};

temperature.fahrenheit = 212;
console.log(temperature._celsius); // 100
console.log(temperature.fahrenheit); // 212
```

### Intermediate Examples

```javascript
// Computed properties with caching
class DataFetcher {
  constructor() {
    this._cache = new Map();
  }
  
  getData(key) {
    if (!this._cache.has(key)) {
      console.log(`Fetching ${key}...`);
      this._cache.set(key, `data for ${key}`);
    }
    return this._cache.get(key);
  }
}

// Using defineProperty for lazy computation
function lazy(obj, prop, compute) {
  let computed = false;
  let value;
  
  Object.defineProperty(obj, prop, {
    get() {
      if (!computed) {
        value = compute.call(this);
        computed = true;
        // Reconfigure to a simple value property
        Object.defineProperty(this, prop, {
          value,
          writable: true,
          enumerable: true,
          configurable: true
        });
      }
      return value;
    },
    configurable: true,
    enumerable: true
  });
}

const obj = {};
lazy(obj, 'expensive', () => {
  console.log('Computing...');
  return 42;
});

console.log(obj.expensive); // 'Computing...' then 42
console.log(obj.expensive); // 42 (no recomputation)

// Property validation with defineProperty
function defineValidated(obj, prop, validator) {
  let value;
  Object.defineProperty(obj, prop, {
    get() { return value; },
    set(newVal) {
      if (!validator(newVal)) {
        throw new Error(`Invalid value for ${String(prop)}`);
      }
      value = newVal;
    },
    enumerable: true,
    configurable: true
  });
}

const person = {};
defineValidated(person, 'age', (v) => Number.isInteger(v) && v >= 0 && v < 150);
person.age = 30; // OK
// person.age = -5; // Error!
```

### Advanced Examples

```javascript
// Property inheritance with getters
class Shape {
  constructor() { this._color = 'black'; }
  
  get color() { return this._color; }
  set color(c) { this._color = c; }
  
  get description() { return `A ${this.color} shape`; }
}

class Circle extends Shape {
  constructor(radius) {
    super();
    this._radius = radius;
  }
  
  get radius() { return this._radius; }
  get area() { return Math.PI * this._radius ** 2; }
  get description() { return `A ${this.color} circle with radius ${this._radius}`; }
}

const c = new Circle(5);
c.color = 'red';
console.log(c.description); // 'A red circle with radius 5'

// Object.defineProperties with mixed descriptors
function createEnum(values) {
  const obj = {};
  for (const value of values) {
    Object.defineProperty(obj, value, {
      value: Symbol(value),
      writable: false,
      enumerable: true,
      configurable: false
    });
  }
  Object.freeze(obj);
  return obj;
}

const Color = createEnum(['RED', 'GREEN', 'BLUE']);
console.log(Color.RED); // Symbol(RED)
// Color.RED = 'other'; // TypeError (frozen + non-writable)

// Plumbing getter/setter through prototype
const proto = {
  get full() { return `${this.first} ${this.last}`; },
  set full(value) { [this.first, this.last] = value.split(' '); }
};

const person = Object.create(proto, {
  first: { value: '', writable: true, enumerable: true },
  last: { value: '', writable: true, enumerable: true }
});

person.full = 'John Doe';
console.log(person.first); // 'John'

// Object.getOwnPropertyDescriptors for copying accessors
const source = {
  _val: 0,
  get value() { return this._val; },
  set value(v) { this._val = v; }
};

// Shallow copy with Object.assign loses getters!
const badCopy = Object.assign({}, source);
console.log(badCopy.value); // undefined (copied _val but not getter)

// Correct copy using descriptors:
const goodCopy = Object.defineProperties(
  {}, Object.getOwnPropertyDescriptors(source)
);
console.log(goodCopy.value); // 0
goodCopy.value = 5;
console.log(goodCopy.value); // 5
```

### Real-World Use Cases

- Vue.js reactivity system (Proxy-based in Vue 3)
- MobX observables
- TypeScript's decorators
- Form validation libraries
- ORM/ODM libraries (Mongoose, Sequelize)
- API mocking tools
- Logging and debugging frameworks
- Access control systems
- Immutable data libraries

### Common Mistakes

- Using `Object.assign()` to copy objects with getters (loses accessors)
- Modifying `Symbol` properties via `for...in` or `Object.keys()` (they're hidden from enumeration)
- Expecting `Proxy` to work transparently with `===` (strict equality checks the proxy, not target)
- Using `Proxy` in performance-critical hot paths (trap overhead)
- Not forwarding `receiver` in Proxy get/set traps (breaks inheritance)
- Forgetting that `Object.defineProperty` defaults to `false` for all descriptor attributes

### Best Practices

- Use `Reflect` methods consistently in Proxy handlers
- Forward `receiver` parameter to `Reflect.get` and `Reflect.set`
- Prefer `Proxy` for intercepting operations on existing objects
- Use `defineProperty` for creating non-standard property behaviors
- Use `Symbol` for protocol-like interfaces (iteration, coercion)
- Use `Object.getOwnPropertyDescriptors` for correctly copying objects with accessors
- Store symbol keys in a variable; don't recreate them

### Performance Considerations

- Proxy traps add overhead to every intercepted operation
- V8 optimizes Proxy traps but they're still slower than direct access
- `Object.defineProperty` has one-time cost (configuration)
- Symbols have minimal overhead as property keys
- Property descriptors don't affect runtime performance after creation (except getters/setters)
- Proxy in hot paths can cause 10-100x slowdown; measure before optimizing

### Interview Questions

**Q: What is the difference between a Proxy and Object.defineProperty?** A: `Object.defineProperty` intercepts access to a single property on an existing object. It can be configured as a data or accessor property. `Proxy` can intercept any operation on an object (property access, deletion, function calls, constructor calls, etc.) and can work with any number of properties dynamically. `Proxy` is more powerful but has runtime overhead.

**Q: How does Vue 3's reactivity differ from Vue 2's?** A: Vue 2 used `Object.defineProperty` to convert each property to getter/setter pairs. This required knowing all properties upfront and couldn't detect property addition/deletion. Vue 3 uses `Proxy` to intercept all property operations dynamically, supporting property addition, deletion, and array index changes without special handling.


### Coding Challenges
```javascript
// Challenge 1: Implement validation with Proxy
function createValidator(obj, schema) {
  return new Proxy(obj, {
    set(target, key, value) {
      if (schema[key] && !schema[key](value)) {
        throw new Error(`Invalid value for ${key}`);
      }
      target[key] = value;
      return true;
    }
  });
}
const user = createValidator({}, {
  age: v => typeof v === 'number' && v > 0
});
user.age = 25; // OK
// user.age = -5; // Error

// Challenge 2: Use defineProperty for computed properties
const person = { firstName: 'John', lastName: 'Doe' };
Object.defineProperty(person, 'fullName', {
  get() { return `${this.firstName} ${this.lastName}`; },
  set(value) { [this.firstName, this.lastName] = value.split(' '); }
});
console.log(person.fullName);
person.fullName = 'Jane Smith';
console.log(person.firstName);

// Challenge 3: Autobind decorator with Reflect
function autobind(target, key, descriptor) {
  const fn = descriptor.value;
  return {
    configurable: true,
    get() {
      const bound = fn.bind(this);
      Object.defineProperty(this, key, { value: bound });
      return bound;
    }
  };
}
```

### Related Topics

- Functional programming and metaprogramming
- Decorators (TC39 proposal)
- Iterators and generators
- Object capability model (security)
- Framework internals (Vue, MobX, Immer)
- AST manipulation (Babel plugins)
- Reflection and introspection
- Dynamic code evaluation (eval, Function constructor)
- Object.observe (deprecated)
- Accessor properties vs data properties
