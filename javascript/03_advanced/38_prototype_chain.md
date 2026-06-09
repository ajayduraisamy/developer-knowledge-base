# Prototype Chain

## Introduction

The prototype chain is JavaScript's inheritance mechanism. When a property is accessed on an object, the engine first looks for the property on the object itself. If not found, it follows the `[[Prototype]]` link to the parent object, then the parent's parent, and so on until either the property is found or the chain ends at `null`. This chain of linked prototypes forms the backbone of property sharing, method inheritance, and memory efficiency in JavaScript.

---

## Prototype chain resolution

### What It Is

Prototype chain resolution is the process by which JavaScript resolves property access on an object. When you access `obj.prop`, the engine:

1. Checks if `obj` has an own property named `prop`.
2. If not, gets the prototype of `obj` (`[[Prototype]]`).
3. Checks if the prototype has `prop`.
4. Repeats step 2-3 up the chain.
5. If the chain ends (`null`), returns `undefined`.

```javascript
const grandparent = { a: 1 };
const parent = Object.create(grandparent);
parent.b = 2;
const child = Object.create(parent);
child.c = 3;

console.log(child.c); // 3 (own)
console.log(child.b); // 2 (from parent)
console.log(child.a); // 1 (from grandparent)
console.log(child.d); // undefined (not found)
```

### Why It Is Important

Prototype chain resolution is fundamental to:
- How inheritance works — child objects access parent methods.
- How built-in methods are shared — `[].map()` comes from `Array.prototype`.
- How property shadowing works — own properties override prototype properties.
- How memory is saved — methods live on the prototype, not on each instance.
- How dynamic dispatch works — adding methods to prototype affects all objects.

### How It Works Internally

The engine's property lookup uses inline caching (IC) for performance. When a property is accessed:

1. The engine checks the object's "shape" (hidden class) for the property.
2. If found, it returns the value directly (megamorphic/monomorphic caching).
3. If not found, it walks the `[[Prototype]]` chain.
4. Each prototype lookup also checks shapes.

V8 optimizes common prototype chain lookups. If a prototype chain hasn't changed, the engine caches the result of the lookup at each access site.

### Syntax

Resolution is automatic — no special syntax is needed:

```javascript
// Property access triggers chain resolution
const value = obj.property;

// Method call triggers chain resolution
obj.method();

// 'in' operator checks the entire chain
console.log("toString" in {}); // true (found on Object.prototype)

// Property assignment does NOT traverse the chain
obj.property = value; // always sets on obj (unless setter on prototype)
```

### Beginner Examples

```javascript
// Example 1: Simple chain
const animal = { eats: true };
const rabbit = Object.create(animal);
rabbit.jumps = true;

console.log(rabbit.jumps); // true (own)
console.log(rabbit.eats); // true (from prototype)
console.log(animal.isPrototypeOf(rabbit)); // true

// Example 2: Shadowing
const parent = { value: 10 };
const child = Object.create(parent);
child.value = 20;

console.log(child.value); // 20 (own property shadows prototype)
console.log(parent.value); // 10 (unchanged)

// Example 3: Built-in chain
const arr = [1, 2, 3];
console.log(arr.toString()); // Array.prototype.toString -> Object.prototype.toString
console.log(arr.__proto__); // Array.prototype
console.log(arr.__proto__.__proto__); // Object.prototype
console.log(arr.__proto__.__proto__.__proto__); // null
```

### Intermediate Examples

```javascript
// Example 1: Method resolution
const vehicle = {
  drive() {
    return "Vroom!";
  },
};
const car = Object.create(vehicle);
car.honk = function () {
  return "Beep!";
};

console.log(car.drive()); // "Vroom!" (from vehicle)
console.log(car.honk()); // "Beep!" (own)

// Method override (shadowing)
car.drive = function () {
  return "Zoom!";
};
console.log(car.drive()); // "Zoom!" (own overrides prototype)

// Example 2: Constructor prototype chain
function Person(name) {
  this.name = name;
}
Person.prototype.sayHi = function () {
  return `Hi, I'm ${this.name}`;
};

const alice = new Person("Alice");
console.log(alice.sayHi()); // "Hi, I'm Alice"
// Resolution: alice -> alice.__proto__ (Person.prototype) -> found sayHi

// Example 3: Multiple levels
const level1 = { x: 1 };
const level2 = Object.create(level1);
level2.y = 2;
const level3 = Object.create(level2);
level3.z = 3;

console.log(level3.x); // 1 (3 levels up)
console.log(level3.y); // 2 (2 levels up)
console.log(level3.z); // 3 (own)

// Deleting shadow property reveals prototype
delete level3.z;
console.log(level3.z); // undefined
delete level3.y;
console.log(level3.y); // 2 (from level2, not own)
```

### Advanced Examples

```javascript
// Example 1: getters on the prototype chain
const proto = {
  get fullName() {
    return `${this.first} ${this.last}`;
  },
};

const user = Object.create(proto);
user.first = "John";
user.last = "Doe";

console.log(user.fullName); // "John Doe"
// The getter runs with 'this' set to the original receiver (user)

// Example 2: Setters on the prototype chain
const protoConfig = {
  set name(val) {
    this._name = val.toUpperCase();
  },
  get name() {
    return this._name;
  },
};

const config = Object.create(protoConfig);
config.name = "database";
console.log(config.name); // "DATABASE"
console.log(config.hasOwnProperty("_name")); // true

// Example 3: Prototype chain with Symbol properties
const sym = Symbol("unique");
const obj1 = { [sym]: "first" };
const obj2 = Object.create(obj1);
console.log(obj2[sym]); // "first" (Symbol properties follow the chain too)

// Example 4: Non-enumerable properties in the chain
Object.defineProperty(Array.prototype, "custom", {
  value: "custom",
  enumerable: false,
});
const arr = [];
console.log(arr.custom); // "custom" (found on prototype)
console.log(Object.keys(arr)); // [] (non-enumerable, not shown)
// Clean up: delete Array.prototype.custom;
```

### Real-World Use Cases

- **Object-oriented code** — All class instances use prototype chains for methods.
- **Framework mixins** — Combining behavior from multiple prototypes.
- **Polyfills** — Adding methods to `Array.prototype` or `Object.prototype`.
- **Feature detection** — Checking if a method exists on the prototype chain.
- **Proxy fallbacks** — Using prototype chain for default values.

### Common Mistakes

- **Assuming `for...in` only iterates own properties** — It iterates enumerable properties on the entire chain.
- **Confusing property access with assignment** — `obj.prop = val` always sets on `obj` (unless there's a setter on the prototype).
- **Shadowing methods unintentionally** — Creating an own property with the same name as a prototype method.
- **Modifying built-in prototypes** — Causes conflicts, especially with `Object.prototype`.
- **Assuming `null` prototype objects have standard methods** — They don't.

### Best Practices

- Use `hasOwnProperty()` to distinguish own vs inherited properties.
- Use `Object.keys()` for own enumerable properties only.
- Avoid modifying built-in prototypes in library code.
- Prefer ES6 class syntax which makes prototype intention clear.
- Use `for...of` instead of `for...in` for arrays (avoids prototype properties).

### Performance Considerations

- Prototype chain lookup is slightly slower than own property access.
- V8 optimizes lookups with inline caching — repeated access to the same property is fast.
- Deep chains (5+ levels) add marginal overhead.
- Properties on `Object.prototype` are the slowest to resolve (end of chain).
- Adding properties to `Object.prototype` affects ALL objects and slows down lookups.

### Interview Questions

1. **How does JavaScript resolve `obj.property`?**
   It checks `obj` for an own property. If not found, it follows the `[[Prototype]]` chain until the property is found or the chain ends at `null`.

2. **What is the difference between own properties and inherited properties?**
   Own properties are directly on the object. Inherited properties are accessed via the prototype chain.

3. **What happens when you assign `obj.prop = value` and `prop` exists on the prototype?**
   The assignment creates or updates an own property on `obj`, shadowing the prototype.

4. **Does `delete obj.prop` affect the prototype chain?**
   No. `delete` only removes own properties. If `prop` exists on the prototype, it becomes visible again after deletion.

### Coding Challenges

1. **Write a function that fully traces the property resolution path.**
2. **Implement `Object.prototype.hasOwnProperty` as a standalone function.**
3. **Create a deep clone that preserves prototype chains.**
4. **Build a function that checks if a property exists anywhere in the chain.**

### Related Topics

- `hasOwnProperty()`
- Prototype chain
- Inheritance
- `instanceof`
- Property descriptors

---

## Inheritance via prototype chain

### What It Is

Inheritance via the prototype chain allows objects to share behavior through linked prototypes. Instead of class-based inheritance (copying behavior), JavaScript uses delegation — objects delegate property lookups to their prototypes. This is known as "prototypal inheritance" or "behavior delegation."

```javascript
const animal = {
  eat() {
    return `${this.name} eats`;
  },
};

const dog = Object.create(animal);
dog.bark = function () {
  return `${this.name} barks`;
};

const rex = Object.create(dog);
rex.name = "Rex";

console.log(rex.eat()); // "Rex eats" (from animal)
console.log(rex.bark()); // "Rex barks" (from dog)
```

### Why It Is Important

Prototypal inheritance is:
- More flexible than class inheritance — objects can inherit from any object.
- Memory efficient — methods are shared, not copied.
- Dynamic — changes to prototypes are reflected by all inheriting objects.
- The foundation of ES6 class syntax (which is syntactic sugar over prototypes).
- Natural for JavaScript — "objects inheriting from objects" is the language's design.

### How It Works Internally

When you create an object with `Object.create(parent)`:
1. A new object is allocated.
2. Its `[[Prototype]]` is set to `parent`.
3. When `child.method()` is called, the engine resolves `method` through the chain.

For constructor functions with `new`:
1. A new object is created.
2. Its `[[Prototype]]` is set to `Constructor.prototype`.
3. The constructor runs.
4. The new object is returned.

The "inheritance" is purely delegation — there is no copying. The child object simply doesn't have the method; it finds it on the prototype when accessed.

### Syntax

```javascript
// Method 1: Object.create (direct prototypal inheritance)
const parent = { method() {} };
const child = Object.create(parent);

// Method 2: Constructor prototype assignment
function Parent() {}
function Child() {}
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;

// Method 3: ES6 class (syntactic sugar)
class Parent {}
class Child extends Parent {}

// Method 4: Object.setPrototypeOf (dynamic, slow)
const obj = {};
Object.setPrototypeOf(obj, parent);
```

### Beginner Examples

```javascript
// Example 1: Animal-Dog inheritance
function Animal(name) {
  this.name = name;
}
Animal.prototype.eat = function () {
  return `${this.name} eats`;
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

const buddy = new Dog("Buddy", "Golden Retriever");
console.log(buddy.eat()); // "Buddy eats"
console.log(buddy.bark()); // "Buddy barks"
console.log(buddy instanceof Dog); // true
console.log(buddy instanceof Animal); // true

// Example 2: Direct prototypal inheritance
const base = {
  init(name) {
    this.name = name;
    return this;
  },
  greet() {
    return `Hello, ${this.name}`;
  },
};

const user = Object.create(base).init("Alice");
console.log(user.greet()); // "Hello, Alice"

// Example 3: Override parent method
const shape = {
  area() {
    return 0;
  },
};
const square = Object.create(shape);
square.init = function (side) {
  this.side = side;
  return this;
};
square.area = function () {
  return this.side * this.side;
};

const box = Object.create(square).init(5);
console.log(box.area()); // 25 (overrides shape.area)
```

### Intermediate Examples

```javascript
// Example 1: Calling parent methods
function Vehicle(type) {
  this.type = type;
}
Vehicle.prototype.move = function () {
  return `${this.type} is moving`;
};

function Car(brand) {
  Vehicle.call(this, "car");
  this.brand = brand;
}
Car.prototype = Object.create(Vehicle.prototype);
Car.prototype.constructor = Car;
Car.prototype.move = function () {
  // Call parent method
  const parentMove = Vehicle.prototype.move.call(this);
  return `${parentMove} at 100km/h`;
};
Car.prototype.honk = function () {
  return `${this.brand} honks`;
};

const myCar = new Car("Toyota");
console.log(myCar.move()); // "car is moving at 100km/h"
console.log(myCar.honk()); // "Toyota honks"

// Example 2: Multiple inheritance via mixins
const canFly = {
  fly() { return `${this.name} flies`; },
};
const canSwim = {
  swim() { return `${this.name} swims`; },
};

function Bird(name) {
  this.name = name;
}
// Mix in behaviors
Object.assign(Bird.prototype, canFly, canSwim);

const duck = new Bird("Duck");
console.log(duck.fly()); // "Duck flies"
console.log(duck.swim()); // "Duck swims"

// Example 3: Prototype chain with intermediate objects
const a = { a: 1 };
const b = Object.create(a);
b.b = 2;
const c = Object.create(b);
c.c = 3;

// c inherits from b which inherits from a
console.log(Object.getPrototypeOf(c) === b); // true
console.log(Object.getPrototypeOf(b) === a); // true
```

### Advanced Examples

```javascript
// Example 1: Selective inheritance with factory functions
function createBeing(name) {
  return { name };
}

const withLegs = {
  walk() { return `${this.name} walks`; },
};

const withWings = {
  fly() { return `${this.name} flies`; },
};

function createDog(name) {
  const dog = createBeing(name);
  Object.assign(dog, withLegs);
  dog.bark = function () { return `${this.name} barks`; };
  return dog;
}

function createBird(name) {
  const bird = createBeing(name);
  Object.assign(bird, withWings, withLegs);
  bird.chirp = function () { return `${this.name} chirps`; };
  return bird;
}

// Example 2: Abstract prototype chain
const abstractAnimal = {
  speak() {
    throw new Error("Must implement speak()");
  },
  eat() {
    return `${this.name} eats`;
  },
};

function createCat(name) {
  const cat = Object.create(abstractAnimal);
  cat.name = name;
  cat.speak = function () { return "Meow"; };
  return cat;
}

const garfield = createCat("Garfield");
console.log(garfield.eat()); // "Garfield eats"
console.log(garfield.speak()); // "Meow"

// Example 3: Super-call pattern in prototypal inheritance
const parent = {
  method() {
    return "parent";
  },
};

const child = Object.create(parent);
child.method = function () {
  return `${this.__proto__.method()} -> child`;
};
// Better approach using Object.getPrototypeOf
child.method = function () {
  const parentMethod = Object.getPrototypeOf(this).method.call(this);
  return `${parentMethod} -> child`;
};
```

### Real-World Use Cases

- **Mongoose models** — Schema methods are inherited via prototype.
- **Express middleware** — Each middleware layer extends `req`/`res`.
- **Backbone.js** — Models, Views, and Collections use prototypal inheritance.
- **React (pre-ES6)** — `createReactClass` used mixins with prototypal inheritance.
- **Custom element prototypes** — Web Components extend `HTMLElement.prototype`.

### Common Mistakes

- **Forgetting to call parent constructor** — `Parent.call(this, args)` in child constructor.
- **Not setting `constructor` after replacing prototype** — Breaks `instance.constructor`.
- **Creating overly deep prototype chains** — Increases lookup time and complexity.
- **Confusing `class` inheritance with class-based languages** — JavaScript uses delegation, not copying.
- **Sharing objects instead of methods on prototype** — Accidental mutation across instances.

### Best Practices

- Use ES6 `class` and `extends` for most inheritance scenarios.
- Keep prototype chains shallow (3-4 levels max).
- Use composition over inheritance when possible (mixins, object.assign).
- Always call the parent constructor from the child constructor.
- Always set `constructor` when replacing `prototype`.

### Performance Considerations

- Each level of prototype chain adds a lookup cost.
- V8 optimizes shallow chains well; deep chains (5+) may be slower.
- Mixing in many methods via `Object.assign` on a prototype is fast.
- Methods defined in constructor (`this.method = ...`) are NOT shared — each instance has its own copy.
- `Object.create()` is faster and more optimized than `Object.setPrototypeOf()`.

### Interview Questions

1. **How does prototypal inheritance differ from classical inheritance?**
   Prototypal inheritance uses delegation — objects inherit from objects. Classical inheritance uses classes and copies behavior.

2. **What is the difference between `Object.create()` and the `new` keyword?**
   Both set the prototype. `new` also runs the constructor function. `Object.create()` just creates the object with the specified prototype.

3. **How do you call a parent method from a child method?**
   `Parent.prototype.method.call(this)` or using `super.method()` in class syntax.

4. **What is the "prototype chain" and how deep should it be?**
   The chain of `[[Prototype]]` links. It should be kept shallow — 3-4 levels maximum.

### Coding Challenges

1. **Implement classical inheritance using only constructor functions and prototypes.**
2. **Create a `mixin` function that merges multiple prototype behaviors.**
3. **Build a multiple inheritance solution using prototype chains.**
4. **Write a function that implements `super()` calls in non-class code.**
5. **Create an inheritance chain and measure property lookup performance at each level.**

### Related Topics

- Prototype chain resolution
- `hasOwnProperty()`
- `instanceof`
- Class `extends`
- Mixins
- Composition vs inheritance

---

## hasOwnProperty()

### What It Is

`hasOwnProperty()` is a method available on all objects (inherited from `Object.prototype`) that returns `true` if the specified property is a direct (own) property of the object, and `false` if it is inherited via the prototype chain.

```javascript
const parent = { inherited: true };
const child = Object.create(parent);
child.own = true;

console.log(child.hasOwnProperty("own")); // true
console.log(child.hasOwnProperty("inherited")); // false
console.log("inherited" in child); // true (in operator checks chain)
```

### Why It Is Important

`hasOwnProperty()` is essential for:
- Distinguishing own properties from inherited ones.
- Safely iterating over object keys without picking up prototype properties.
- Implementing `for...in` loops correctly (should be combined with `hasOwnProperty`).
- JSON serialization — `JSON.stringify` only serializes own enumerable properties.
- Defensive programming — checking for direct property ownership before access.

### How It Works Internally

`hasOwnProperty()` checks the object's own property storage (the "properties backing store" or "dictionary" for that object). It does NOT traverse the prototype chain. Internally, the engine looks up the property in the object's property map — if it's found as an own property (including non-enumerable), it returns `true`.

### Syntax

```javascript
obj.hasOwnProperty(prop);

// prop must be a string or Symbol
const sym = Symbol("test");
obj[sym] = "value";
obj.hasOwnProperty(sym); // true

// Called safely on objects with null prototype
Object.prototype.hasOwnProperty.call(obj, prop);
```

### Beginner Examples

```javascript
// Example 1: Own vs inherited
const obj = { ownProp: "value" };
console.log(obj.hasOwnProperty("ownProp")); // true
console.log(obj.hasOwnProperty("toString")); // false (inherited)
console.log("toString" in obj); // true (exists in chain)

// Example 2: hasOwnProperty in loops
const parent = { inherited: "parent" };
const child = Object.create(parent);
child.own = "child";

for (const key in child) {
  if (child.hasOwnProperty(key)) {
    console.log(`Own: ${key}`); // "Own: own"
  } else {
    console.log(`Inherited: ${key}`); // "Inherited: inherited"
  }
}

// Example 3: Non-enumerable own properties
const obj = {};
Object.defineProperty(obj, "hidden", {
  value: "secret",
  enumerable: false,
});
console.log(obj.hasOwnProperty("hidden")); // true (own but non-enumerable)
console.log(Object.keys(obj)); // [] (non-enumerable not listed)
```

### Intermediate Examples

```javascript
// Example 1: Safe call on null-prototype objects
const dict = Object.create(null);
dict.key = "value";

// This would throw: dict.hasOwnProperty("key") (no hasOwnProperty)
// Safe approach:
console.log(Object.prototype.hasOwnProperty.call(dict, "key")); // true
console.log(Object.prototype.hasOwnProperty.call(dict, "toString")); // false

// Example 2: hasOwnProperty with Symbol keys
const sym = Symbol("unique");
const obj = { [sym]: "symbol value", regular: "regular value" };
console.log(obj.hasOwnProperty(sym)); // true
console.log(obj.hasOwnProperty("regular")); // true

// Example 3: hasOwnProperty vs Object.hasOwn (ES2022)
const obj = { prop: 1 };
console.log(obj.hasOwnProperty("prop")); // true
console.log(Object.hasOwn(obj, "prop")); // true (ES2022, safer for null-prototype)

// Object.hasOwn is the modern replacement
```

### Advanced Examples

```javascript
// Example 1: Overriding hasOwnProperty
const malicious = {
  hasOwnProperty() {
    return false; // always returns false
  },
  data: "secret",
};
// This bypasses the overridden method
console.log(Object.prototype.hasOwnProperty.call(malicious, "data")); // true

// Example 2: Checking arrays
const arr = [1, 2, 3];
arr.customProp = "custom";
console.log(arr.hasOwnProperty(0)); // true (index 0)
console.log(arr.hasOwnProperty("length")); // true (length is own)
console.log(arr.hasOwnProperty("customProp")); // true
console.log(arr.hasOwnProperty("map")); // false (from Array.prototype)

// Example 3: Filtering out inherited methods
function getOwnMethods(obj) {
  const methods = [];
  for (const key in obj) {
    if (
      obj.hasOwnProperty(key) &&
      typeof obj[key] === "function"
    ) {
      methods.push(key);
    }
  }
  return methods;
}

// Example 4: hasOwnProperty with Object.keys precision
const data = { a: 1, b: 2 };
Object.defineProperty(data, "c", { value: 3, enumerable: false });
console.log(Object.keys(data)); // ["a", "b"]
console.log(Object.getOwnPropertyNames(data)); // ["a", "b", "c"]
```

### Real-World Use Cases

- **Lodash's `_.has`** — Uses `hasOwnProperty` internally.
- **React's `hasOwnProperty` checks** — Used in prop validation.
- **Object iteration utilities** — Libraries use it to avoid prototype traversal.
- **Secure coding** — Defensive checks against prototype pollution.
- **JSON serialization** — Only own properties are serialized.

### Common Mistakes

- **Calling `hasOwnProperty` on null-prototype objects** — Throws TypeError.
- **Using `hasOwnProperty` with `for...in` without a check** — `for...in` includes inherited properties.
- **Overriding `hasOwnProperty` in objects** — Breaks property checking (use `Object.hasOwn`).
- **Confusing `hasOwnProperty` with `in`** — `in` checks the prototype chain too.
- **Using `hasOwnProperty` for arrays** — Works fine, but `Array.isArray` is better for type checking.

### Best Practices

- Use `Object.hasOwn(obj, prop)` (ES2022) instead of `obj.hasOwnProperty(prop)` — it's safer.
- Always use `hasOwnProperty` inside `for...in` loops to filter inherited properties.
- When working with unknown objects, use `Object.prototype.hasOwnProperty.call(obj, prop)`.
- Use `Object.keys()` for own enumerable properties (no iteration needed).
- Use `getOwnPropertyNames()` for all own properties including non-enumerable.

### Performance Considerations

- `hasOwnProperty()` is very fast — it's a simple property map lookup.
- `Object.hasOwn()` is similarly fast in modern engines.
- `for...in` with `hasOwnProperty` check is slower than `Object.keys()` for own properties only.
- Avoid calling `hasOwnProperty` in hot inner loops — cache the result if possible.

### Interview Questions

1. **What is the difference between `hasOwnProperty` and the `in` operator?**
   `hasOwnProperty` checks only own properties. `in` checks the entire prototype chain.

2. **Why is `obj.hasOwnProperty('toString')` false?**
   Because `toString` is inherited from `Object.prototype`, not an own property of `obj`.

3. **What is `Object.hasOwn()` and how is it different?**
   `Object.hasOwn()` (ES2022) is a static method that calls `hasOwnProperty` safely, even on objects with null prototypes or overridden `hasOwnProperty`.

4. **How would you safely check for an own property on an unknown object?**
   Use `Object.hasOwn(obj, prop)` or `Object.prototype.hasOwnProperty.call(obj, prop)`.

### Coding Challenges

1. **Implement `Object.hasOwn()` as a polyfill.**
2. **Create a function that lists only own properties (enumerable and non-enumerable).**
3. **Build a deep property checker that distinguishes own from inherited at all levels.**
4. **Write a `for...in` alternative that only iterates own properties.**
5. **Implement a safe `hasOwnProperty` that works on null-prototype objects.**

### Related Topics

- Prototype chain
- `in` operator
- `Object.keys()`
- `Object.getOwnPropertyNames()`
- `Object.hasOwn()`
- `for...in`

---

## instanceof operator

### What It Is

The `instanceof` operator tests whether the `prototype` property of a constructor appears anywhere in the prototype chain of an object. It returns `true` if the constructor's prototype is found in the object's prototype chain, `false` otherwise.

```javascript
function Animal() {}
function Dog() {}
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

const rex = new Dog();
console.log(rex instanceof Dog); // true
console.log(rex instanceof Animal); // true
console.log(rex instanceof Object); // true
console.log(rex instanceof Array); // false
```

### Why It Is Important

`instanceof` is essential for:
- Type checking objects at runtime.
- Implementing polymorphic behavior.
- Checking inheritance relationships.
- Validating object types in libraries and frameworks.
- Implementing custom type guards in TypeScript.

### How It Works Internally

The `instanceof` operator works as follows:

1. Get `right.prototype` (the constructor's `prototype` property).
2. Get `left.__proto__` (the object's `[[Prototype]]`).
3. Walk the prototype chain of `left`:
   - If `left.__proto__ === right.prototype`, return `true`.
   - Otherwise, go to `left.__proto__.__proto__`.
4. If chain ends (null), return `false`.

Note: `instanceof` checks the `prototype` property of the right-hand side, which is different from the `[[Prototype]]` of the left-hand side.

### Syntax

```javascript
object instanceof Constructor;

// Returns boolean
[1,2,3] instanceof Array; // true
[1,2,3] instanceof Object; // true
[1,2,3] instanceof RegExp; // false

// Also works with classes
class MyClass {}
const instance = new MyClass();
instance instanceof MyClass; // true
```

### Beginner Examples

```javascript
// Example 1: Basic instanceof
const arr = [1, 2, 3];
console.log(arr instanceof Array); // true
console.log(arr instanceof Object); // true (Array extends Object)
console.log(arr instanceof Function); // false

// Example 2: Custom constructors
function Car(make) {
  this.make = make;
}
const myCar = new Car("Toyota");
console.log(myCar instanceof Car); // true
console.log(myCar instanceof Object); // true

// Example 3: Primitives
console.log("hello" instanceof String); // false (primitives are not objects)
console.log(new String("hello") instanceof String); // true (wrapper objects)
```

### Intermediate Examples

```javascript
// Example 1: instanceof with modified prototype
function Animal() {}
function Dog() {}
Dog.prototype = Object.create(Animal.prototype);

const rex = new Dog();
console.log(rex instanceof Dog); // true
console.log(rex instanceof Animal); // true

// After changing Dog.prototype:
const otherProto = {};
Dog.prototype = otherProto;
const fido = new Dog();
console.log(fido instanceof Dog); // true (fido created after change)
console.log(rex instanceof Dog); // false (rex still has old prototype)

// Example 2: instanceof with Object.create
const animal = { eats: true };
const rabbit = Object.create(animal);
console.log(rabbit instanceof Object); // true
// But: console.log(rabbit instanceof ???) - no constructor

// Example 3: Cross-frame instanceof issues
// If objects come from different iframes/windows,
// instanceof may fail because constructors differ
// Solution: use duck typing or Symbol.hasInstance
```

### Advanced Examples

```javascript
// Example 1: Symbol.hasInstance custom behavior
class MyArray {
  static [Symbol.hasInstance](instance) {
    return Array.isArray(instance);
  }
}

console.log([] instanceof MyArray); // true
console.log({} instanceof MyArray); // false

// Example 2: instanceof with primitive wrappers
console.log(1 instanceof Number); // false
console.log(new Number(1) instanceof Number); // true

// Custom primitive check
function isNumber(value) {
  return typeof value === "number" || value instanceof Number;
}

// Example 3: instanceof in inheritance trees
class A {}
class B extends A {}
class C extends B {}

const obj = new C();
console.log(obj instanceof A); // true
console.log(obj instanceof B); // true
console.log(obj instanceof C); // true

const obj2 = new B();
console.log(obj2 instanceof C); // false
console.log(obj2 instanceof A); // true

// Example 4: Checking for function
function isFunction(value) {
  return typeof value === "function" || value instanceof Function;
}
```

### Real-World Use Cases

- **Type checking in libraries** — Mongoose checks if objects are Model instances.
- **Error handling** — `err instanceof SyntaxError`, `err instanceof TypeError`.
- **Polymorphic dispatch** — Different behavior based on instance type.
- **Validation** — Ensuring function arguments are the expected type.
- **Serialization frameworks** — Checking object types for custom serialization.

```javascript
// Error handling with instanceof
try {
  JSON.parse("invalid json");
} catch (err) {
  if (err instanceof SyntaxError) {
    console.log("Invalid JSON format");
  } else {
    throw err;
  }
}
```

### Common Mistakes

- **Using instanceof with primitives** — Primitives are not objects, so `instanceof` returns `false`.
- **Cross-realm instanceof** — Objects from different iframes/contexts have different constructors.
- **Assuming instanceof checks the constructor** — It checks the `prototype` property, which is different.
- **Changing `Constructor.prototype` after creating instances** — Existing instances may fail instanceof.
- **Forgetting that `instanceof` checks the chain** — `obj instanceof Object` is `true` for almost everything.

### Best Practices

- Use `typeof` for primitive type checks, `instanceof` for object type checks.
- For cross-realm scenarios, use duck typing or `Symbol.hasInstance`.
- Avoid changing `Constructor.prototype` after creating instances.
- Use `Array.isArray()` instead of `arr instanceof Array` for cross-frame safety.
- Use `Object.prototype.toString.call(obj)` for robust type detection across realms.

### Performance Considerations

- `instanceof` walks the prototype chain — it's fast for shallow chains.
- V8 optimizes `instanceof` for built-in types (Array, RegExp, etc.).
- Custom `Symbol.hasInstance` may be slower if it does complex checks.
- Changing `Constructor.prototype` invalidates V8's inline caches.
- `instanceof` is generally very fast (nanoseconds) for most use cases.

### Interview Questions

1. **How does `instanceof` work?**
   It checks if `Constructor.prototype` is in the prototype chain of the object. It walks the chain and compares each prototype.

2. **What is the difference between `typeof` and `instanceof`?**
   `typeof` returns a string for primitive types. `instanceof` checks object inheritance against constructors.

3. **Can `instanceof` return false positives?**
   Yes, if `Constructor.prototype` has been tampered with or if the object is from a different realm.

4. **What is `Symbol.hasInstance`?**
   A well-known Symbol that allows a constructor to customize `instanceof` behavior.

### Coding Challenges

1. **Implement `instanceof` operator using `Object.getPrototypeOf()`.**
2. **Create a cross-realm-safe `instanceof` function.**
3. **Build a `getPrototypeChain` function that shows all constructor names.**
4. **Implement custom `Symbol.hasInstance` for a utility class.**
5. **Write a type-checking utility that combines `typeof` and `instanceof`.**

### Related Topics

- Prototype chain
- `Symbol.hasInstance`
- `typeof` operator
- Duck typing
- Type checking
- Cross-realm objects
