# Prototypes

## Introduction

Prototypes are the fundamental mechanism for inheritance and property sharing in JavaScript. Every object has an internal `[[Prototype]]` property that points to another object (or null). When a property is accessed on an object and not found, JavaScript follows the prototype chain to look up the property. Understanding prototypes is essential for mastering object-oriented JavaScript, inheritance, and the language's behavior under the hood.

---

## __proto__ property

### What It Is

`__proto__` is a historical accessor property (getter/setter) on `Object.prototype` that exposes the internal `[[Prototype]]` of an object. It allows reading and setting the prototype of an object. While widely supported, `__proto__` is deprecated in favor of `Object.getPrototypeOf()` and `Object.setPrototypeOf()`.

```javascript
const obj = {};
console.log(obj.__proto__); // Object.prototype
console.log(obj.__proto__ === Object.prototype); // true

const arr = [];
console.log(arr.__proto__); // Array.prototype
console.log(arr.__proto__.__proto__); // Object.prototype
console.log(arr.__proto__.__proto__.__proto__); // null
```

### Why It Is Important

Understanding `__proto__` helps developers:
- Visualize the prototype chain for debugging.
- Understand how property lookup works.
- Recognize the difference between `__proto__` and `prototype`.
- Work with legacy code that uses `__proto__`.
- Set up inheritance chains for testing and exploration.

### How It Works Internally

`__proto__` is defined as a getter/setter on `Object.prototype`. When you access `obj.__proto__`, it calls the getter which returns `[[Prototype]]` (an internal slot). When you set `obj.__proto__ = value`, it calls the setter which changes `[[Prototype]]` to the new value (or throws if the value is not an object or null). The internal `[[Prototype]]` is stored as an immutable (in most cases) link that the engine uses for property resolution.

### Syntax

```javascript
// Reading prototype
const proto = obj.__proto__;

// Setting prototype
const parent = { name: "parent" };
const child = {};
child.__proto__ = parent;
console.log(child.name); // "parent"

// Checking prototype existence
console.log("__proto__" in {}); // true (inherited from Object.prototype)
```

### Beginner Examples

```javascript
// Example 1: Reading the prototype chain
const person = { type: "human" };
const employee = { role: "developer" };
employee.__proto__ = person;

console.log(employee.role); // "developer" (own)
console.log(employee.type); // "human" (from prototype)
console.log(employee.__proto__); // { type: "human" }
console.log(employee.__proto__.__proto__); // Object.prototype

// Example 2: Modifying prototype affects all linked objects
const base = { greeting: "Hello" };
const a = {};
const b = {};
a.__proto__ = base;
b.__proto__ = base;

console.log(a.greeting); // "Hello"
console.log(b.greeting); // "Hello"

base.greeting = "Hi";
console.log(a.greeting); // "Hi"
console.log(b.greeting); // "Hi"

// Example 3: __proto__ of different types
console.log([].__proto__); // Array.prototype
console.log(function () {}.__proto__); // Function.prototype
console.log(new Date().__proto__); // Date.prototype
console.log(/regex/.__proto__); // RegExp.prototype
```

### Intermediate Examples

```javascript
// Example 1: Setting __proto__ to null
const dict = {};
dict.__proto__ = null;
// Now dict has no prototype chain
console.log(dict.toString); // undefined
console.log(dict.__proto__); // undefined (no setter)

// Example 2: Cycling prototype (error)
const a = {};
const b = {};
a.__proto__ = b;
// b.__proto__ = a; // TypeError: Cyclic __proto__ value

// Example 3: __proto__ in object literals
const obj = {
  __proto__: {
    method() {
      return "from proto";
    },
  },
  ownProp: "own",
};
console.log(obj.method()); // "from proto"
console.log(obj.ownProp); // "own"

// Example 4: Performance note - setting __proto__ is slow
// V8 deoptimizes objects when __proto__ is changed
function avoidSetter() {
  // Prefer Object.create() for setting prototypes
  const parent = { x: 1 };
  const child = Object.create(parent);
}
```

### Advanced Examples

```javascript
// Example 1: __proto__ in class inheritance
class Animal {
  speak() {
    return "Animal speaks";
  }
}
class Dog extends Animal {
  bark() {
    return "Woof!";
  }
}

const dog = new Dog();
console.log(dog.__proto__); // Dog.prototype
console.log(dog.__proto__.__proto__); // Animal.prototype
console.log(dog.__proto__.__proto__.__proto__); // Object.prototype

// Example 2: Disabling __proto__ via Object.create(null)
const noProto = Object.create(null);
console.log(noProto.__proto__); // undefined (no Object.prototype)
// noProto is truly prototype-free

// Example 3: __proto__ getter/setter emulation
const protoAccessor = {
  getProto() {
    return Object.getPrototypeOf(this);
  },
  setProto(obj) {
    Object.setPrototypeOf(this, obj);
  },
};
```

### Real-World Use Cases

- **Debugging** — Inspecting prototype chains in DevTools console.
- **Polyfilling** — Setting up prototype chains for older browsers.
- **Object.create(null)** — Creating dictionaries without prototype pollution.
- **Prototype manipulation in tests** — Mocking and stubbing methods.
- **Learning and exploration** — Understanding JavaScript's inheritance model.

### Common Mistakes

- **Confusing `__proto__` with `prototype`** — `__proto__` is on instances, `prototype` is on constructors.
- **Setting `__proto__` on primitive values** — Primitives don't have `__proto__` (their wrapper objects do).
- **Using `__proto__` for performance-critical code** — It's slower than `Object.create()`.
- **Assuming `__proto__` is available everywhere** — It may not exist in strict engine modes or older JS environments.

### Best Practices

- Don't use `__proto__` in production code. Use `Object.getPrototypeOf()` and `Object.setPrototypeOf()`.
- Never use `__proto__` as a property key in objects that serve as dictionaries (use `Object.create(null)`).
- Prefer `Object.create()` for setting prototypes at object creation time.
- Use `__proto__` only for debugging and learning.

### Performance Considerations

- Accessing `__proto__` is a getter call — slightly slower than property access.
- Setting `__proto__` is expensive — it triggers deoptimization in V8.
- Objects with mutable `[[Prototype]]` are slower than objects with fixed prototypes.
- `Object.create()` at creation time is the most performant way to set prototypes.

### Interview Questions

1. **What is `__proto__`?**
   A getter/setter on `Object.prototype` that exposes the internal `[[Prototype]]` of an object.

2. **What is the difference between `__proto__` and `prototype`?**
   `__proto__` is the actual prototype of an instance. `prototype` is a property on constructor functions that becomes the `__proto__` of instances created via `new`.

3. **Is `__proto__` deprecated?**
   Yes, it's deprecated in favor of `Object.getPrototypeOf()` and `Object.setPrototypeOf()`, though it remains widely supported.

4. **Can you set `__proto__` to any value?**
   Only objects or null. Setting it to a primitive is silently ignored or throws in strict mode.

### Coding Challenges

1. **Implement a function that prints the full prototype chain of an object using `__proto__`.**
2. **Create a function that checks if two objects share the same prototype.**
3. **Build a `Object.create()` polyfill that uses `__proto__`.**
4. **Write a function that detects if `__proto__` is supported.**

### Related Topics

- `prototype` property
- `Object.getPrototypeOf()`
- `Object.setPrototypeOf()`
- Prototype chain
- Object.create()

---

## prototype property

### What It Is

The `prototype` property is a special property on constructor functions (and classes) that becomes the `[[Prototype]]` (i.e., `__proto__`) of all objects created via `new` with that constructor. It is NOT the constructor's own prototype — `Constructor.prototype` is an object that will be assigned as the prototype of instances.

```javascript
function Person(name) {
  this.name = name;
}
Person.prototype.sayHello = function () {
  return `Hello, I'm ${this.name}`;
};

const alice = new Person("Alice");
console.log(alice.sayHello()); // "Hello, I'm Alice"
console.log(alice.__proto__ === Person.prototype); // true
console.log(Person.prototype.constructor === Person); // true
```

### Why It Is Important

The `prototype` property is the foundation of JavaScript's constructor-based inheritance:
- It defines shared methods and properties for all instances of a constructor.
- It enables memory-efficient method sharing (one function, not per-instance copies).
- It is the basis for class syntax in ES6.
- It allows dynamic addition of methods to all instances, even after creation.
- It is essential for understanding how JavaScript's OOP model works.

### How It Works Internally

When a function is created, the engine automatically gives it a `prototype` property — an object with a `constructor` property pointing back to the function. When `new F()` is called:

1. A new object is created.
2. Its `[[Prototype]]` is set to `F.prototype`.
3. The constructor function runs with `this` bound to the new object.
4. The new object is returned (unless the constructor returns another object).

The `prototype` property is only meaningful on constructor functions — regular functions have one but it's unused unless called with `new`.

### Syntax

```javascript
// Constructor function with prototype
function Constructor(param) {
  this.prop = param;
}
Constructor.prototype.method = function () {};

// Adding multiple methods at once
Constructor.prototype = {
  method1() {},
  method2() {},
  // Don't forget to set constructor
  constructor: Constructor,
};

// With classes (syntactic sugar over prototype)
class MyClass {
  method() {} // goes to MyClass.prototype
}
```

### Beginner Examples

```javascript
// Example 1: Shared method via prototype
function Animal(type) {
  this.type = type;
}
Animal.prototype.describe = function () {
  return `This is a ${this.type}`;
};

const cat = new Animal("cat");
const dog = new Animal("dog");
console.log(cat.describe()); // "This is a cat"
console.log(dog.describe()); // "This is a dog"
console.log(cat.describe === dog.describe); // true (same function)

// Example 2: Adding methods after instantiation
function Counter() {
  this.count = 0;
}
const c1 = new Counter();

// Add method later
Counter.prototype.increment = function () {
  this.count++;
};

c1.increment();
console.log(c1.count); // 1 (method added after object creation)

// Example 3: constructor property
function Car(make) {
  this.make = make;
}
const myCar = new Car("Toyota");
console.log(myCar.constructor); // Car function
console.log(myCar.constructor === Car); // true
```

### Intermediate Examples

```javascript
// Example 1: Overriding prototype (replaces all methods)
function Widget(name) {
  this.name = name;
}
// First prototype setup
Widget.prototype.render = function () {
  return `Widget: ${this.name}`;
};
const w1 = new Widget("A");
console.log(w1.render()); // "Widget: A"

// Replace prototype entirely
Widget.prototype = {
  constructor: Widget,
  render() {
    return `Rendered: ${this.name}`;
  },
  destroy() {
    this.name = null;
  },
};
const w2 = new Widget("B");
console.log(w1.render()); // "Widget: A" (old prototype)
console.log(w2.render()); // "Rendered: B" (new prototype)

// Example 2: Inheritance via prototype
function Shape(color) {
  this.color = color;
}
Shape.prototype.getColor = function () {
  return this.color;
};

function Circle(radius, color) {
  Shape.call(this, color); // call parent constructor
  this.radius = radius;
}
// Set up inheritance
Circle.prototype = Object.create(Shape.prototype);
Circle.prototype.constructor = Circle;

Circle.prototype.getArea = function () {
  return Math.PI * this.radius ** 2;
};

const circle = new Circle(5, "red");
console.log(circle.getColor()); // "red" (from Shape.prototype)
console.log(circle.getArea()); // 78.54 (from Circle.prototype)

// Example 3: Checking method origin
console.log(circle.hasOwnProperty("getColor")); // false
console.log("getColor" in circle); // true
console.log(Circle.prototype.hasOwnProperty("getColor")); // false
console.log(Shape.prototype.hasOwnProperty("getColor")); // true
```

### Advanced Examples

```javascript
// Example 1: prototype and class syntax
class Vehicle {
  constructor(type) {
    this.type = type;
  }
  move() {
    return `${this.type} is moving`;
  }
  // Static method - on constructor, not prototype
  static classify() {
    return "Vehicle classification";
  }
}

console.log(Vehicle.prototype.move); // function
console.log(Vehicle.prototype.constructor); // Vehicle
console.log(Vehicle.classify); // function (static)
console.log(Vehicle.prototype.classify); // undefined (static not on proto)

// Example 2: Multiple inheritance-like pattern
const canWalk = {
  walk() {
    return `${this.name} is walking`;
  },
};
const canSwim = {
  swim() {
    return `${this.name} is swimming`;
  },
};

function Animal(name) {
  this.name = name;
}
Object.assign(Animal.prototype, canWalk, canSwim);

const duck = new Animal("Duck");
console.log(duck.walk()); // "Duck is walking"
console.log(duck.swim()); // "Duck is swimming"

// Example 3: prototype-free constructors
function Plain() {
  return { a: 1 }; // returns object, not this
}
Plain.prototype.method = function () {
  return "method";
};
const p = new Plain();
console.log(p.method); // undefined (p is not an instance of Plain)
```

### Real-World Use Cases

- **Mongoose (MongoDB ODM)** — Schema methods are added to prototype.
- **Express.js** — `req` and `res` objects inherit from prototypes with utility methods.
- **jQuery** — `$.fn` is jQuery's prototype for plugin methods.
- **React (pre-classes)** — `React.createClass` used prototype-based inheritance.
- **Prototype-based frameworks** — Backbone.js uses prototype extensively.

### Common Mistakes

- **Forgetting to set `constructor` after replacing prototype** — Breaks `instance.constructor`.
- **Using arrow functions in prototype** — Arrow functions don't have their own `this`.
- **Overwriting prototype with an object that doesn't extend the original.**
- **Assuming `prototype` is the function's own prototype** — It's not; `__proto__` is.
- **Modifying built-in prototypes (`Array.prototype`, `Object.prototype`)** — Bad practice, causes conflicts.

### Best Practices

- Use ES6 `class` syntax instead of manual prototype manipulation.
- When using constructor functions, always set up prototype methods outside the constructor.
- If you replace the prototype object, reassign `constructor` to the original function.
- Avoid modifying built-in prototypes like `Array.prototype` or `Object.prototype`.
- Use `Object.getPrototypeOf()` to read prototypes, not `__proto__`.

### Performance Considerations

- Methods on the prototype are shared across all instances — memory efficient.
- Property lookup on the prototype chain is slightly slower than own properties.
- Deep prototype chains increase lookup time (though modern engines optimize this).
- Replacing `Constructor.prototype` after objects are created breaks existing instances.
- V8 creates "hidden classes" based on property structure — changing prototypes disrupts optimization.

### Interview Questions

1. **What is the `prototype` property?**
   A property on constructor functions that becomes the `[[Prototype]]` of instances created via `new`.

2. **What is the difference between `prototype` and `__proto__`?**
   `prototype` is a property on constructors. `__proto__` is the actual prototype of any object (instance).

3. **What does `Constructor.prototype.constructor` point to?**
   The constructor function itself. It's a back-reference from the prototype to the constructor.

4. **What happens if you add a method to the prototype after creating instances?**
   Existing instances immediately have access to the new method via the prototype chain.

### Coding Challenges

1. **Create a constructor function with methods on its prototype.**
2. **Implement inheritance by setting up prototype chains manually.**
3. **Build a `mixin` function that copies methods to a constructor's prototype.**
4. **Write a function that lists all methods available on an object (own and inherited).**

### Related Topics

- `__proto__`
- `Object.getPrototypeOf()`
- Constructor functions
- `new` operator
- Class syntax
- `instanceof`

---

## Object.getPrototypeOf()

### What It Is

`Object.getPrototypeOf(obj)` is the standard, non-deprecated method for retrieving the `[[Prototype]]` of an object. It returns the prototype (an object or null) of the specified object. It is the recommended alternative to `__proto__`.

```javascript
const proto = { shared: true };
const obj = Object.create(proto);
console.log(Object.getPrototypeOf(obj)); // { shared: true }
console.log(Object.getPrototypeOf(obj) === proto); // true
```

### Why It Is Important

`Object.getPrototypeOf()` is:
- The standards-compliant way to read prototypes.
- Available in all modern environments (ES5+).
- Safer than `__proto__` (no accidental property access issues).
- Essential for writing library code that needs to traverse the prototype chain.
- Used internally by the `instanceof` operator and other language features.

### How It Works Internally

`Object.getPrototypeOf()` reads the internal `[[Prototype]]` slot of the object directly, without triggering any getter (unlike `__proto__` which goes through `Object.prototype`). It is a direct, low-level access to the engine's internal prototype pointer.

### Syntax

```javascript
Object.getPrototypeOf(obj);

// Returns null for prototype-less objects
const nullProto = Object.create(null);
console.log(Object.getPrototypeOf(nullProto)); // null

// Returns same as obj.__proto__
const obj = {};
console.log(Object.getPrototypeOf(obj) === obj.__proto__); // true
```

### Beginner Examples

```javascript
// Example 1: Getting prototype of different objects
const obj = {};
console.log(Object.getPrototypeOf(obj) === Object.prototype); // true

const arr = [];
console.log(Object.getPrototypeOf(arr) === Array.prototype); // true

function fn() {}
console.log(Object.getPrototypeOf(fn) === Function.prototype); // true

// Example 2: Object.create
const parent = { value: 42 };
const child = Object.create(parent);
console.log(Object.getPrototypeOf(child) === parent); // true
console.log(child.value); // 42

// Example 3: null prototype
const noProto = Object.create(null);
console.log(Object.getPrototypeOf(noProto)); // null
console.log(noProto.toString); // undefined
```

### Intermediate Examples

```javascript
// Example 1: Traversing the prototype chain
function getAllPrototypes(obj) {
  const protos = [];
  let current = obj;
  while (current) {
    const proto = Object.getPrototypeOf(current);
    if (proto) protos.push(proto);
    current = proto;
  }
  return protos;
}

console.log(getAllPrototypes([]));
// [Array.prototype, Object.prototype]

// Example 2: Checking if an object has a specific prototype
function isInstanceOf(obj, constructor) {
  let proto = Object.getPrototypeOf(obj);
  while (proto) {
    if (proto === constructor.prototype) return true;
    proto = Object.getPrototypeOf(proto);
  }
  return false;
}

console.log(isInstanceOf([], Array)); // true
console.log(isInstanceOf([], Object)); // true
console.log(isInstanceOf([], RegExp)); // false

// Example 3: With classes
class Base {}
class Derived extends Base {}
const instance = new Derived();
console.log(Object.getPrototypeOf(instance) === Derived.prototype); // true
console.log(Object.getPrototypeOf(Derived.prototype) === Base.prototype); // true
```

### Advanced Examples

```javascript
// Example 1: Object.getPrototypeOf for method borrowing
function prototypeOf(obj) {
  return Object.getPrototypeOf(obj);
}

const arrayProto = prototypeOf([]);
// Borrow methods from Array.prototype for array-like objects
const arrayLike = { 0: "a", 1: "b", length: 2 };
Array.prototype.forEach.call(arrayLike, (item) => console.log(item));

// Example 2: Safe object inspection
function isPlainObject(obj) {
  if (obj === null || typeof obj !== "object") return false;
  const proto = Object.getPrototypeOf(obj);
  return proto === null || proto === Object.prototype;
}

console.log(isPlainObject({})); // true
console.log(isPlainObject([])); // false
console.log(isPlainObject(Object.create(null))); // true

// Example 3: Extracting prototype chain for debugging
function debugPrototypeChain(obj) {
  const chain = [];
  let current = obj;
  while (current) {
    const proto = Object.getPrototypeOf(current);
    chain.push({
      value: current,
      proto: proto,
      constructor: proto?.constructor?.name,
    });
    current = proto;
  }
  return chain;
}
```

### Real-World Use Cases

- **Utility libraries** — Lodash's `isPlainObject` uses `Object.getPrototypeOf`.
- **Polyfill detection** — Checking if a built-in prototype has native methods.
- **Debugging tools** — Analyzing prototype chains in developer tools.
- **Framework internals** — Vue's reactivity system uses prototype checks.
- **Type checking** — Custom `instanceof`-like checks.

### Common Mistakes

- **Using `Object.getPrototypeOf` on primitives** — Primitives are coerced to objects, but the prototype of the wrapper is returned.
- **Assuming all objects have `Object.prototype` in their chain** — `Object.create(null)` doesn't.
- **Confusing `Object.getPrototypeOf` with `Object.getOwnPropertyDescriptor`** — Different purposes.

### Best Practices

- Always use `Object.getPrototypeOf()` instead of `__proto__`.
- Use `Object.create()` to set prototypes at creation time.
- Use `Object.setPrototypeOf()` sparingly (performance impact).
- Use `Object.getPrototypeOf()` for prototype chain traversal utilities.

### Performance Considerations

- `Object.getPrototypeOf()` is a fast, direct internal read.
- It is significantly faster than `obj.__proto__` (which is a getter call).
- Traversing long prototype chains with repeated calls is still fast.
- V8 optimizes this to a single pointer dereference.

### Interview Questions

1. **What does `Object.getPrototypeOf()` return?**
   The internal `[[Prototype]]` of an object (or null for prototype-free objects).

2. **How is `Object.getPrototypeOf` different from `__proto__`?**
   `Object.getPrototypeOf` is a direct API call that reads `[[Prototype]]` internally. `__proto__` is a getter/setter on `Object.prototype`.

3. **Can `Object.getPrototypeOf` return `null`?**
   Yes, for objects created with `Object.create(null)`.

4. **Is `Object.getPrototypeOf` available in ES5?**
   Yes, it was introduced in ES5.

### Coding Challenges

1. **Implement `Object.getPrototypeOf` without using the native method.**
2. **Write a function that checks if an object inherits from a given prototype.**
3. **Build a prototype chain visualizer that uses `Object.getPrototypeOf`.**
4. **Create a `getRootPrototype` function that follows the chain to the end.**

### Related Topics

- `__proto__`
- `Object.setPrototypeOf()`
- `Object.create()`
- Prototype chain
- `instanceof`

---

## Setting prototypes

### What It Is

Setting prototypes refers to defining or changing the `[[Prototype]]` of an object. The primary methods are `Object.create()` (creates a new object with a specified prototype) and `Object.setPrototypeOf()` (mutates an existing object's prototype). The `__proto__` setter can also be used but is deprecated.

```javascript
// Object.create - create with prototype
const parent = { greet() { return "Hello"; } };
const child = Object.create(parent);
console.log(child.greet()); // "Hello"

// Object.setPrototypeOf - mutate existing object
const obj = {};
Object.setPrototypeOf(obj, parent);
console.log(obj.greet()); // "Hello"
```

### Why It Is Important

Setting prototypes is how JavaScript implements inheritance:
- `Object.create()` is the canonical way to establish prototype chains.
- `Object.setPrototypeOf()` allows changing an object's prototype dynamically.
- Framework code (e.g., transpilers for class inheritance) uses these internally.
- Understanding prototype setting is essential for debugging inheritance issues.
- It enables object composition and mixin patterns.

### How It Works Internally

`Object.create(proto)`:
1. Creates a new empty object.
2. Sets its `[[Prototype]]` to `proto`.
3. Returns the new object.

`Object.setPrototypeOf(obj, proto)`:
1. Checks if `obj` is extensible.
2. Changes the `[[Prototype]]` of `obj` to `proto`.
3. This triggers deoptimization in V8 (the object's hidden class changes).

The engine must update internal data structures when prototypes change, which is why `Object.setPrototypeOf` is slow.

### Syntax

```javascript
// Object.create
Object.create(proto);
Object.create(proto, descriptorsObject);

// Object.setPrototypeOf
Object.setPrototypeOf(obj, proto);

// Using __proto__ (deprecated)
obj.__proto__ = proto;
```

### Beginner Examples

```javascript
// Example 1: Object.create
const animal = {
  init(name) {
    this.name = name;
    return this;
  },
  speak() {
    return `${this.name} makes a sound`;
  },
};

const dog = Object.create(animal).init("Rex");
console.log(dog.speak()); // "Rex makes a sound"
console.log(Object.getPrototypeOf(dog) === animal); // true

// Example 2: Object.setPrototypeOf
const base = { baseMethod() { return "base"; } };
const obj = { ownMethod() { return "own"; } };
Object.setPrototypeOf(obj, base);
console.log(obj.baseMethod()); // "base"
console.log(obj.ownMethod()); // "own"

// Example 3: Setting null prototype
const dict = Object.create(null);
dict.key = "value";
console.log(dict.toString); // undefined (clean dictionary)
```

### Intermediate Examples

```javascript
// Example 1: Prototype chain with Object.create
const level0 = { a: 0 };
const level1 = Object.create(level0);
level1.b = 1;
const level2 = Object.create(level1);
level2.c = 2;

console.log(level2.c); // 2 (own)
console.log(level2.b); // 1 (from level1)
console.log(level2.a); // 0 (from level0)

// Example 2: Property descriptors with Object.create
const obj = Object.create(
  { inherited: 1 },
  {
    own: {
      value: 2,
      writable: true,
      enumerable: true,
      configurable: true,
    },
  }
);
console.log(obj.inherited); // 1
console.log(obj.own); // 2

// Example 3: Classical inheritance with Object.create
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function () {
  return `${this.name} speaks`;
};

function Dog(name, breed) {
  Animal.call(this, name);
  this.breed = breed;
}
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
Dog.prototype.bark = function () {
  return `${this.name} barks`;
};

const rex = new Dog("Rex", "German Shepherd");
console.log(rex.speak()); // "Rex speaks"
console.log(rex.bark()); // "Rex barks"
console.log(rex instanceof Dog); // true
console.log(rex instanceof Animal); // true
```

### Advanced Examples

```javascript
// Example 1: Dynamic prototype swapping
function withLogging(obj) {
  const loggingProto = {
    ...Object.getPrototypeOf(obj),
    get(key) {
      console.log(`Getting ${key}`);
      return this[key];
    },
    set(key, value) {
      console.log(`Setting ${key} to ${value}`);
      this[key] = value;
    },
  };
  Object.setPrototypeOf(loggingProto, Object.getPrototypeOf(obj));
  Object.setPrototypeOf(obj, loggingProto);
  return obj;
}

const data = { x: 1, y: 2 };
withLogging(data);
console.log(data.x); // "Getting x" then 1

// Example 2: Prototype chain for behavior delegation
const canEat = {
  eat() { return `${this.name} eats`; },
};
const canSleep = {
  sleep() { return `${this.name} sleeps`; },
};
const canFly = {
  fly() { return `${this.name} flies`; },
};

function createCreature(name, abilities) {
  const creature = { name };
  // Build prototype chain through abilities
  let current = creature;
  abilities.forEach((ability) => {
    Object.setPrototypeOf(current, ability);
    current = ability;
  });
  return creature;
}

const bat = createCreature("Bat", [canEat, canSleep, canFly]);
console.log(bat.eat()); // "Bat eats"

// Example 3: Preventing prototype changes
const sealed = Object.seal({});
// Object.setPrototypeOf(sealed, {}); // TypeError: non-extensible
```

### Real-World Use Cases

- **Transpiled classes** — Babel/TypeScript use `Object.create` to implement `extends`.
- **Framework internals** — Vue 2 used prototype manipulation for reactivity.
- **ORM libraries** — Sequelize, Mongoose use prototype chains for model instances.
- **Type checking utilities** — Building custom `instanceof`-like checks.
- **Non-extensible objects** — Security-sensitive code uses `Object.create(null)` for clean dictionaries.

### Common Mistakes

- **Using `Object.setPrototypeOf` in performance-critical code** — Deoptimizes the object.
- **Creating cyclic prototype chains** — Leads to errors.
- **Setting prototype to a primitive** — Silently ignored or throws.
- **Forgetting to set `constructor` after prototype assignment** — Breaks instanceof-like checks.
- **Using `Object.setPrototypeOf` on frozen or sealed objects** — Throws TypeError.

### Best Practices

- Use `Object.create()` for setting prototypes at creation time, not `Object.setPrototypeOf()`.
- Avoid `Object.setPrototypeOf()` in hot paths — it destroys engine optimizations.
- If you must change prototypes dynamically, do it once and early in the object's life.
- Use `Object.create(null)` for dictionaries to avoid prototype pollution.
- Prefer ES6 `class` and `extends` over manual prototype manipulation.

### Performance Considerations

- `Object.create()` is fast — it creates a new object with the specified prototype.
- `Object.setPrototypeOf()` is slow — it changes the prototype of an existing object, forcing V8 to deoptimize and re-optimize.
- Objects with mutable prototypes cannot use V8's inline cache optimizations.
- Prototype chain traversal is fast regardless of how the chain was set up.
- Never call `Object.setPrototypeOf()` in hot loops or on frequently accessed objects.

### Interview Questions

1. **What is the difference between `Object.create()` and `Object.setPrototypeOf()`?**
   `Object.create()` creates a new object with the specified prototype. `Object.setPrototypeOf()` mutates an existing object's prototype.

2. **Why is `Object.setPrototypeOf()` considered slow?**
   It changes the prototype of an existing object, forcing V8 to deoptimize hidden class optimizations.

3. **How do you create an object without a prototype?**
   `Object.create(null)` — useful for dictionaries.

4. **What happens if you set a prototype to `null`?**
   The object has no prototype chain, so inherited methods like `toString` and `hasOwnProperty` are unavailable.

### Coding Challenges

1. **Implement `Object.create()` as a polyfill.**
2. **Build a function that merges multiple prototypes using `Object.setPrototypeOf`.**
3. **Create a class-free inheritance pattern using only `Object.create()`.**
4. **Write a function that copies prototype methods from one object to another.**
5. **Implement a simple ORM that uses prototype chains for model instances.**

### Related Topics

- `Object.create()`
- `Object.setPrototypeOf()`
- `__proto__`
- `new` operator
- Class `extends`
- Prototype chain
