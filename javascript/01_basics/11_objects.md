# Objects - Object literal, constructor, properties, methods, this

## Introduction
Objects are the fundamental data structure in JavaScript, representing collections of key-value pairs. Almost everything in JavaScript is an object or can behave like one. Understanding objects is central to mastering the language.

## Object literal syntax

### What It Is
The object literal syntax (`{}`) is the simplest and most common way to create objects. It allows inline definition of properties and methods using a concise, readable syntax.

### Why It Is Important
Object literals are used extensively for data structures, configuration objects, namespacing, and modeling real-world entities. They are the foundation of JSON and data exchange.

### How It Works Internally
When the parser encounters `{}`, it creates a new Object instance with `Object.prototype` as its prototype. Property assignments use the internal `[[DefineOwnProperty]]` method. The engine may optimize object representations using "hidden classes" for fast property access.

### Syntax
```javascript
const obj = {
  property: value,
  method() {},
  get getterName() {},
  set setterName(value) {},
  [computedKey]: value
};
```

### Beginner Examples
```javascript
// Basic object literal
const person = {
  firstName: "Alice",
  lastName: "Johnson",
  age: 30,
  greet() {
    console.log(`Hi, I'm ${this.firstName}`);
  }
};

console.log(person.firstName); // "Alice"
console.log(person["lastName"]); // "Johnson"
person.greet(); // "Hi, I'm Alice"

// Shorthand properties (ES6)
const name = "Bob";
const age = 25;
const user = { name, age };
console.log(user); // { name: "Bob", age: 25 }

// Computed property names
const key = "dynamicKey";
const obj = {
  [key]: "dynamic value",
  ["prop_" + 1]: "also dynamic"
};
```

### Intermediate Examples
```javascript
// Method shorthand
const calculator = {
  value: 0,
  add(n) { this.value += n; return this; },
  subtract(n) { this.value -= n; return this; },
  multiply(n) { this.value *= n; return this; },
  getResult() { return this.value; }
};
calculator.add(5).multiply(2).subtract(3);
console.log(calculator.getResult()); // 7

// Getter and setter
const temperature = {
  _celsius: 0,
  get fahrenheit() {
    return this._celsius * 9 / 5 + 32;
  },
  set fahrenheit(value) {
    this._celsius = (value - 32) * 5 / 9;
  },
  get celsius() { return this._celsius; },
  set celsius(value) { this._celsius = value; }
};
temperature.celsius = 25;
console.log(temperature.fahrenheit); // 77

// Nested objects
const company = {
  name: "Tech Corp",
  address: {
    street: "123 Main St",
    city: "San Francisco",
    zip: "94105"
  },
  employees: [
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" }
  ]
};
```

### Advanced Examples
```javascript
// Computed keys for dynamic behavior
const handlers = {};
["click", "hover", "focus"].forEach(event => {
  handlers[event] = function(data) {
    console.log(`Handling ${event} with`, data);
  };
});

// Property value shorthand with destructuring
function createUser(name, email, role = "user") {
  return { name, email, role, createdAt: new Date() };
}

// Object spread
const defaults = { host: "localhost", port: 3000, ssl: false };
const config = { ...defaults, port: 8080, ssl: true };
console.log(config); // { host: "localhost", port: 8080, ssl: true }

// Short-circuit evaluation in object literals
const isAdmin = false;
const userData = {
  name: "Alice",
  ...(isAdmin && { adminPanel: true, permissions: ["all"] })
};

// Object with Symbol keys
const ID = Symbol("id");
const obj2 = {
  [ID]: 12345,
  name: "Secret Agent"
};
console.log(obj2[ID]); // 12345
```

### Real-World Use Cases
- Data transfer objects (DTOs) for API communication
- Configuration objects
- State objects in applications
- Module namespaces
- Enum-like structures
- Factory function returns
- React component state and props (plain objects)
- JSON data structures

### Common Mistakes
- Trailing commas in JSON (valid in JS objects, invalid in JSON)
- Using reserved words as property keys without quotes
- Forgetting comma between properties
- Confusing `=` with `:` for property assignment
- Accidentally creating global variables in object literals (no `var`/`let`)

### Best Practices
- Use shorthand property names when variable matches key
- Use method shorthand instead of `function` keyword
- Use computed property names for dynamic keys
- Use getters/setters for computed properties
- Use object spread for merging/cloning (shallow)
- Keep object literals readable with consistent formatting
- Use `const` for objects that won't be reassigned

### Performance Considerations
- Object literals are highly optimized (hidden classes)
- Shorthand properties have same performance as explicit
- Computed property names prevent some optimizations
- Spread operator creates shallow copies (O(n))
- Getters/setters have small overhead
- Accessing deep paths repeatedly should be cached

### Interview Questions
1. What is the difference between dot notation and bracket notation?
2. How do computed property names work?
3. What is property value shorthand?
4. How do getters and setters work in object literals?
5. Can you use spread operator with objects?

### Coding Challenges
1. Create a nested object literal representing a library catalog.
2. Write a function that merges multiple objects using spread.
3. Create an object with computed property names for a dynamic form builder.
4. Implement a chainable calculator using method shorthand.

### Related Topics
- Object() constructor
- Properties and methods
- JSON

## Object() constructor

### What It Is
The `Object()` constructor creates object wrappers or delegates to other constructors based on the argument type. When called without arguments or with `null`/`undefined`, it returns an empty object.

### Why It Is Important
Understanding the Object constructor helps in type conversion, object creation patterns, and understanding JavaScript's object model. It is also the basis for all objects in JavaScript.

### How It Works Internally
`Object(value)` performs ToObject conversion: primitives become wrapper objects, and objects are returned as-is. `new Object(value)` creates an object that wraps the value. The `Object` constructor is also the root of the prototype chain.

### Syntax
```javascript
Object();          // {}
Object(null);      // {}
Object(undefined); // {}
Object(value);     // Object wrapper or the value itself

new Object();      // {}
new Object(value); // Wrapper object
```

### Beginner Examples
```javascript
// Empty object
const obj1 = new Object();
const obj2 = Object();
const obj3 = {};
// All three create essentially the same thing

// Converting primitives
console.log(Object(42));          // Number { 42 }
console.log(Object("hello"));     // String { "hello" }
console.log(Object(true));        // Boolean { true }

// Object() with existing object (returns same reference)
const original = { a: 1 };
const copy = Object(original);
console.log(original === copy); // true

// Object() is a no-op for objects
const arr = [1, 2, 3];
console.log(Object(arr) === arr); // true
```

### Intermediate Examples
```javascript
// Object() for type coercion
function ensureObject(value) {
  return Object(value);
}
console.log(ensureObject(null));        // {}
console.log(ensureObject(undefined));   // {}
console.log(ensureObject("test"));      // String("test")
console.log(ensureObject(42));          // Number(42)

// Object constructor vs literal performance
// Literal {} is faster and preferred

// Object.create (alternative)
const proto = { greet() { return "Hello"; } };
const obj = Object.create(proto);
console.log(obj.greet()); // "Hello"
console.log(Object.getPrototypeOf(obj) === proto); // true

// Object.assign for merging
const target = { a: 1 };
const source1 = { b: 2 };
const source2 = { c: 3 };
Object.assign(target, source1, source2);
console.log(target); // { a: 1, b: 2, c: 3 }
```

### Advanced Examples
```javascript
// Object.fromEntries (reverse of Object.entries)
const entries = [["name", "Alice"], ["age", 30]];
const fromEntries = Object.fromEntries(entries);
console.log(fromEntries); // { name: "Alice", age: 30 }

// Object with null prototype
const nullProto = Object.create(null);
nullProto.key = "value";
console.log(nullProto.toString); // undefined (no prototype)
console.log(nullProto.key);      // "value"

// Property descriptors with Object.defineProperties
const obj = Object.defineProperties({}, {
  name: { value: "Alice", writable: true, enumerable: true },
  id: { value: 12345, writable: false, enumerable: false },
  greeting: {
    get() { return `Hi, ${this.name}`; },
    enumerable: true
  }
});

// Preventing modification
const frozen = Object.freeze({ a: 1 });
const sealed = Object.seal({ a: 1, b: 2 });
const extensible = Object.preventExtensions({ a: 1 });
```

### Real-World Use Cases
- Creating objects with specific prototypes (Object.create)
- Merging configurations (Object.assign)
- Converting key-value pairs to objects (Object.fromEntries)
- Defining non-enumerable or read-only properties
- Creating objects without prototype (for safe dictionaries)

### Common Mistakes
- Using `new Object()` instead of `{}` (unnecessary)
- Assuming `Object()` clones objects (returns same reference)
- Forgetting that `Object.assign` does shallow merge
- Mutating the target object unintentionally with Object.assign
- Confusing `Object.create` with `new Object`

### Best Practices
- Always use `{}` over `new Object()` for empty objects
- Use `Object.assign` for shallow merging (or spread operator)
- Use `Object.create(null)` for dictionary/map objects
- Use `Object.freeze` for constants
- Use `Object.defineProperties` for fine-grained property control
- Use `Object.fromEntries` for transforming Map/entries to object

### Performance Considerations
- `{}` is faster than `new Object()`
- `Object.create(null)` has slightly faster property access (no prototype chain)
- `Object.assign` is slower than spread for small objects
- `Object.freeze` objects can be optimized by JIT
- Property descriptors add slight overhead

### Interview Questions
1. What is the difference between `{}` and `new Object()`?
2. How does `Object.create()` work?
3. What is `Object.assign()` used for?
4. What is the difference between `Object.freeze()` and `Object.seal()`?
5. How do you create an object with no prototype?

### Coding Challenges
1. Implement a simple `Object.assign` polyfill.
2. Create a function using `Object.create` for prototypal inheritance.
3. Write a deep merge function using Object.assign or similar.
4. Create an immutable configuration object using Object.freeze.

### Related Topics
- Object literal syntax
- Object static methods
- Prototypes and inheritance

## Properties and methods

### What It Is
Properties are key-value pairs stored on objects. Values can be primitives, objects, or functions (methods). Properties have descriptors controlling their behavior (writable, enumerable, configurable).

### Why It Is Important
Understanding property descriptors, property access, enumeration, and ownership is crucial for metaprogramming, serialization, and object manipulation.

### How It Works Internally
Each property has a property descriptor with attributes: `value`, `writable`, `enumerable`, `configurable` (data properties) or `get`, `set`, `enumerable`, `configurable` (accessor properties). The engine uses hidden classes for property lookup optimization.

### Syntax
```javascript
// Property access
obj.property;        // Dot notation
obj["property"];     // Bracket notation

// Property assignment
obj.property = value;

// Delete
delete obj.property;

// Property descriptors
Object.getOwnPropertyDescriptor(obj, prop);
Object.defineProperty(obj, prop, descriptor);
Object.defineProperties(obj, props);
Object.getOwnPropertyDescriptors(obj);

// Property inspection
Object.keys(obj);     // Own enumerable string keys
Object.values(obj);   // Own enumerable values
Object.entries(obj);  // Own enumerable [key, value] pairs
Object.getOwnPropertyNames(obj); // All own string keys
Object.getOwnPropertySymbols(obj); // All own symbol keys
```

### Beginner Examples
```javascript
const user = {
  name: "Alice",
  age: 30
};

// Accessing properties
console.log(user.name);     // "Alice" (dot notation)
console.log(user["age"]);   // 30 (bracket notation)

// Setting properties
user.email = "alice@example.com";
user["phone"] = "555-0123";

// Dynamic property access
const propName = "name";
console.log(user[propName]); // "Alice"

// Delete
delete user.phone;
console.log(user.phone); // undefined

// Check existence
console.log("name" in user);    // true
console.log(user.hasOwnProperty("name")); // true
console.log(user.toString !== undefined); // true (inherited)

// Enumerate
for (const key in user) console.log(key); // name, age, email
Object.keys(user).forEach(k => console.log(k)); // name, age, email
```

### Intermediate Examples
```javascript
// Property descriptors
const obj = {};
Object.defineProperty(obj, "readOnly", {
  value: 42,
  writable: false,
  enumerable: true,
  configurable: false
});
// obj.readOnly = 100; // TypeError in strict mode

Object.defineProperty(obj, "hidden", {
  value: "secret",
  enumerable: false
});
console.log(Object.keys(obj));   // ["readOnly"] (hidden excluded)

// Accessor properties
const person = {
  _name: "Alice",
  get name() { return this._name; },
  set name(value) {
    if (!value) throw new Error("Name required");
    this._name = value;
  }
};

// Merging properties
const source = { a: 1, b: 2 };
const target = { b: 3, c: 4 };
Object.assign(target, source);
console.log(target); // { b: 1, c: 4, a: 1 }

// Property enumeration order
const mixed = { b: 2, a: 1, 0: "zero", 2: "two" };
console.log(Object.keys(mixed));
// ["0", "2", "b", "a"] (integer keys first, then insertion)
```

### Advanced Examples
```javascript
// Property cloning with descriptors
function cloneWithDescriptors(obj) {
  return Object.defineProperties(
    {},
    Object.getOwnPropertyDescriptors(obj)
  );
}

// Proxy for custom property behavior
const handler = {
  get(target, property) {
    if (property in target) {
      console.log(`Reading ${property}`);
      return target[property];
    }
    return "Property not found";
  },
  set(target, property, value) {
    console.log(`Setting ${property} to ${value}`);
    target[property] = value;
    return true;
  }
};
const proxy = new Proxy({}, handler);
proxy.name = "Alice"; // "Setting name to Alice"
console.log(proxy.name); // "Reading name" \n "Alice"

// Computed property access
function getProperty(obj, path) {
  return path.split(".").reduce((current, key) =>
    current ? current[key] : undefined, obj
  );
}
const data = { user: { profile: { name: "Alice" } } };
console.log(getProperty(data, "user.profile.name")); // "Alice"
```

### Real-World Use Cases
- Serialization/deserialization (JSON)
- Dynamic property access for form data
- Property validation with setters
- Creating immutable properties with descriptors
- Property enumeration for UI rendering
- State management (React setState merges)
- API response transformation

### Common Mistakes
- Using `for...in` without `hasOwnProperty` check (gets inherited properties)
- Deleting array elements with `delete` (creates sparse array)
- Confusing `hasOwnProperty` with `in` operator
- Mutating `Object.prototype` (never do this!)
- Assuming property enumeration order is always insertion order

### Best Practices
- Use `Object.keys()` for own enumerable properties
- Use `for...in` only with `hasOwnProperty` check
- Use `Object.hasOwn(obj, prop)` (ES2022) over `obj.hasOwnProperty(prop)`
- Use `Object.defineProperty` for controlled property creation
- Use getters/setters for computed or validated properties
- Avoid deleting properties (set to undefined or use Map)
- Prefer bracket notation for dynamic keys

### Performance Considerations
- Dot notation is faster than bracket notation
- Property lookup is O(1) on average (hash table or hidden class)
- Deleting properties deoptimizes hidden classes
- `Object.keys()` returns a new array each call
- Property enumeration creates array allocations
- Accessor (getter/setter) methods have function call overhead

### Interview Questions
1. What is the difference between dot and bracket property access?
2. How do property descriptors work?
3. What is the `in` operator vs `hasOwnProperty`?
4. How do you make a property non-writable?
5. What is the difference between enumerable and non-enumerable properties?

### Coding Challenges
1. Implement a function that gets nested object property safely.
2. Create a function that deep clones an object preserving property descriptors.
3. Write a diff function that finds added/removed/changed properties.
4. Implement a property validator using setters.

### Related Topics
- Object static methods
- Property descriptors
- Proxy

## this keyword in objects

### What It Is
The `this` keyword refers to the object that is executing the current function. In object methods, `this` typically refers to the object on which the method was called.

### Why It Is Important
Understanding `this` is essential for object-oriented JavaScript. It enables methods to access and modify the object's properties dynamically. Incorrect `this` binding is one of the most common bug sources.

### How It Works Internally
The value of `this` is determined by the execution context. In method calls, `this` is set to the object before the dot. The engine passes `this` as an implicit argument. Arrow functions capture `this` lexically from the enclosing scope.

### Syntax
```javascript
const obj = {
  value: 42,
  method() {
    console.log(this.value); // `this` is obj
  },
  arrow: () => {
    console.log(this.value); // `this` is outer scope
  }
};
```

### Beginner Examples
```javascript
const user = {
  name: "Alice",
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  }
};
user.greet(); // "Hello, I'm Alice"

// this depends on call context
const greet = user.greet;
greet(); // "Hello, I'm undefined" (this is global/window)

// this in nested methods
const counter = {
  count: 0,
  increment() {
    this.count++;
  },
  reset() {
    this.count = 0;
  }
};
counter.increment();
counter.increment();
console.log(counter.count); // 2
```

### Intermediate Examples
```javascript
// this in prototype methods
function Person(name) {
  this.name = name;
}
Person.prototype.sayHello = function() {
  return `Hello, I'm ${this.name}`;
};
const alice = new Person("Alice");
console.log(alice.sayHello()); // "Hello, I'm Alice"

// Losing this context
const button = {
  label: "Click me",
  click() {
    console.log(`${this.label} clicked`);
  }
};
// setTimeout(button.click, 100); // "undefined clicked" (this lost)

// Fixing this with arrow function
const button2 = {
  label: "Click me",
  click() {
    setTimeout(() => {
      console.log(`${this.label} clicked`); // arrow captures `this`
    }, 100);
  }
};

// Fixing this with bind
const boundClick = button.click.bind(button);
// setTimeout(boundClick, 100); // "Click me clicked"
```

### Advanced Examples
```javascript
// Method chaining with this
const chainable = {
  value: 0,
  add(n) { this.value += n; return this; },
  multiply(n) { this.value *= n; return this; },
  subtract(n) { this.value -= n; return this; },
  log() { console.log(this.value); return this; }
};
chainable.add(5).multiply(2).subtract(3).log(); // 7

// this in getters/setters
const person = {
  _name: "Alice",
  get name() { return this._name; },
  set name(value) {
    this._name = value.trim();
  }
};

// Context binding in event handlers
class UIComponent {
  constructor() {
    this.element = document.createElement("button");
    this.element.textContent = "Click";
    this.element.addEventListener("click", this.handleClick.bind(this));
    // Or use: this.element.addEventListener("click", (e) => this.handleClick(e));
  }
  handleClick(event) {
    console.log("Clicked:", this.element.textContent);
  }
}

// call, apply, bind for explicit this
function updateInfo(role, department) {
  this.role = role;
  this.department = department;
}
const employee = { name: "Bob" };
updateInfo.call(employee, "developer", "engineering");
updateInfo.apply(employee, ["designer", "creative"]);
const boundUpdate = updateInfo.bind(employee, "manager");
boundUpdate("operations");
```

### Real-World Use Cases
- Object-oriented class methods
- Event handlers in DOM
- React class component methods (binding in constructor)
- Method chaining (jQuery-like APIs)
- Prototype method definitions
- Builder pattern implementations
- Mixin and trait implementations

### Common Mistakes
- Losing `this` when passing method as callback
- Using arrow functions for object methods (wrong `this`)
- Forgetting `new` with constructor functions (`this` becomes global)
- Assuming `this` inside nested functions refers to outer object
- Not binding methods in React class components
- Confusing `this` in event handlers vs callback functions

### Best Practices
- Use arrow functions for callbacks within methods (captures `this`)
- Use `.bind()` or arrow functions for event handlers
- Use class syntax instead of constructor functions (clearer `this`)
- Avoid nested function declarations within methods (use arrow functions)
- Document expected `this` context in JSDoc
- Use `this` only in methods, not in standalone functions
- Use `Function.prototype.bind()` to create bound method references

### Performance Considerations
- `this` resolution is fast (bound by call context)
- `.bind()` creates a new function (minor overhead)
- Arrow functions don't have their own `this` (no binding cost)
- Method calls are optimized by JIT (inline caching)
- Repeated `.bind()` calls create multiple function objects

### Interview Questions
1. How is `this` determined in a method call?
2. How do you preserve `this` in a callback?
3. What is the difference between `call`, `apply`, and `bind`?
4. How does `this` work in arrow functions?
5. What happens when you use `this` in a constructor without `new`?

### Coding Challenges
1. Create an object with chainable methods using `this`.
2. Fix a `this` reference issue in a setTimeout callback.
3. Implement a simple event emitter that uses `this` for context.
4. Write a function that works as both a regular function and a constructor (checking `this`).
5. Create a mixin pattern that merges methods with proper `this` binding.

### Related Topics
- this keyword (comprehensive guide)
- call, apply, bind
- Arrow functions
- Classes and constructors
