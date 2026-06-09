# Factory Pattern - Factory function, object creation, composition over inheritance

## Introduction

The Factory Pattern is a creational design pattern that provides a way to create objects without exposing the instantiation logic to the caller. Instead of using the `new` keyword directly, a factory function or method encapsulates the creation process, returning new instances based on provided parameters. This pattern is particularly powerful in JavaScript due to the language's first-class functions, dynamic typing, and prototypal inheritance model.

This file covers three core concepts: factory functions as a primary object creation mechanism, various object creation patterns available in JavaScript, and the principle of composition over inheritance — a philosophy deeply embedded in JavaScript's design.

## Factory Functions

### What It Is

A factory function is any function that returns a new object without using the `new` keyword. Unlike constructors, factory functions are regular functions that explicitly create and return an object. They can encapsulate object creation logic, apply defaults, and conditionally add properties or methods.

### Why It Is Important

Factory functions provide:
- **Encapsulation**: Private variables and methods via closure
- **Flexibility**: Can return different object types based on conditions
- **Simplicity**: No `this` binding confusion, no `prototype` manipulation needed
- **Composability**: Easy to mix in behaviors from multiple sources

### How It Works Internally

When a factory function is called, it creates a new object literal (or uses `Object.create()`), optionally sets up private state via closures, attaches methods, and returns the object. Each call creates a fresh object with its own copy of the methods unless the methods are shared via a prototype or external object.

### Syntax

```javascript
// Basic factory function
function createPerson(name, age) {
  return {
    name,
    age,
    greet() {
      return `Hi, I'm ${this.name}`;
    }
  };
}

// Factory with private state (closure)
function createCounter(start = 0) {
  let count = start;
  return {
    increment() { return ++count; },
    decrement() { return --count; },
    getCount() { return count; }
  };
}
```

### Beginner Examples

```javascript
// Simple user factory
function createUser(name, role = "user") {
  return {
    name,
    role,
    isAdmin: role === "admin",
    sayHello() {
      return `Hello, I'm ${this.name}`;
    }
  };
}

const alice = createUser("Alice", "admin");
const bob = createUser("Bob");
console.log(alice.sayHello()); // Hello, I'm Alice
console.log(alice.isAdmin); // true
console.log(bob.isAdmin); // false

// Animal factory
function createAnimal(type, name, sound) {
  return {
    type,
    name,
    sound,
    makeSound() {
      return `${name} (${type}) says ${sound}`;
    }
  };
}

const dog = createAnimal("mammal", "Rex", "woof");
const bird = createAnimal("avian", "Tweety", "chirp");
```

### Intermediate Examples

```javascript
// Factory with validation and default values
function createProduct(config) {
  const { name, price, category = "general", tags = [] } = config || {};

  if (!name || typeof name !== "string") {
    throw new Error("Product name is required and must be a string");
  }
  if (typeof price !== "number" || price < 0) {
    throw new Error("Product price must be a non-negative number");
  }

  const id = `prod_${Date.now()}_${Math.random().toString(36).slice(2, 7)}`;
  const createdAt = new Date();

  return {
    id,
    name,
    price,
    category,
    tags,
    createdAt,
    applyDiscount(percent) {
      if (percent < 0 || percent > 100) throw new Error("Invalid discount percent");
      this.price = this.price * (1 - percent / 100);
    },
    toJSON() {
      return { id, name, price, category, tags, createdAt };
    }
  };
}

const laptop = createProduct({
  name: "Laptop Pro",
  price: 1499.99,
  category: "electronics",
  tags: ["laptop", "tech"]
});
console.log(laptop.toJSON());

// Factory with private methods
function createBankAccount(owner, initialBalance = 0) {
  let balance = initialBalance;
  const transactions = [];

  function logTransaction(type, amount) {
    transactions.push({ type, amount, date: new Date(), balance });
  }

  return {
    owner,
    getBalance() { return balance; },
    deposit(amount) {
      if (amount <= 0) throw new Error("Deposit must be positive");
      balance += amount;
      logTransaction("deposit", amount);
      return balance;
    },
    withdraw(amount) {
      if (amount <= 0) throw new Error("Withdrawal must be positive");
      if (amount > balance) throw new Error("Insufficient funds");
      balance -= amount;
      logTransaction("withdrawal", amount);
      return balance;
    },
    getStatement() {
      return [...transactions];
    }
  };
}
```

### Advanced Examples

```javascript
// Abstract factory pattern
function createUIFactory(theme) {
  return {
    createButton(text) {
      return theme === "dark"
        ? { type: "button", text, style: "background: #333; color: #fff; padding: 8px 16px;" }
        : { type: "button", text, style: "background: #f0f0f0; color: #000; padding: 8px 16px;" };
    },
    createPanel(title) {
      return theme === "dark"
        ? { type: "panel", title, style: "background: #222; border: 1px solid #444;" }
        : { type: "panel", title, style: "background: #fff; border: 1px solid #ccc;" };
    },
    createInput(placeholder) {
      return theme === "dark"
        ? { type: "input", placeholder, style: "background: #444; color: #fff; border: 1px solid #666;" }
        : { type: "input", placeholder, style: "background: #fff; color: #000; border: 1px solid #ccc;" };
    }
  };
}

const darkFactory = createUIFactory("dark");
const lightFactory = createUIFactory("light");

const darkButton = darkFactory.createButton("Submit");
const lightPanel = lightFactory.createPanel("Settings");

// Factory with prototype delegation for memory efficiency
const methodCache = {
  greet() { return `Hi, I'm ${this.name}`; },
  toString() { return `Person: ${this.name}, ${this.age}`; },
  isAdult() { return this.age >= 18; }
};

function createPersonEfficient(name, age) {
  return Object.create(methodCache, {
    name: { value: name, writable: true, enumerable: true },
    age: { value: age, writable: true, enumerable: true }
  });
}

const p1 = createPersonEfficient("Alice", 30);
const p2 = createPersonEfficient("Bob", 15);
console.log(p1.isAdult()); // true
console.log(p2.isAdult()); // false
console.log(Object.getPrototypeOf(p1) === methodCache); // true (shared methods)

// Async factory
async function createDatabaseConnection(config) {
  const connection = await connectToDatabase(config);

  return {
    connection,
    async query(sql, params) {
      return connection.execute(sql, params);
    },
    async close() {
      await connection.disconnect();
      this.connected = false;
    },
    connected: true
  };
}

// Usage: const db = await createDatabaseConnection({ host: "localhost", port: 5432 });
```

### Real-World Use Cases

- **API client factories**: Creating configured HTTP clients with base URLs, headers, and interceptors (like Axios instances).
- **ORM/ODM model factories**: Mongoose models, Sequelize models — wrapping schema definitions into factory functions.
- **UI component factories**: Creating themed UI elements (as shown in the dark/light example).
- **Test data factories**: Generating test objects with sensible defaults and overrides (Factory Bot / Fishery).
- **Connection pool factories**: Creating database connection pools with configuration.

### Common Mistakes

```javascript
// Mistake 1: Returning shared mutable state
const shared = { count: 0 };
function createCounterBad() {
  return {
    increment() { return ++shared.count; }
  };
}
const a = createCounterBad();
const b = createCounterBad();
a.increment(); // shared state - b's counter also affected!

// Mistake 2: Forgetting to handle the case where caller uses `new`
function createPerson(name) {
  return { name };
}
const p = new createPerson("Alice"); // Works but confusing - avoid

// Mistake 3: Not validating inputs
function createUserBad(name, age) {
  return { name, age }; // No validation - age could be a string
}

// Mistake 4: Over-complicating factories when a simple literal suffices
```

### Best Practices

```javascript
// Use descriptive names with "create" prefix
function createShoppingCart() { /* ... */ }

// Always validate inputs
function createTemperature(value, scale = "C") {
  if (!["C", "F", "K"].includes(scale)) throw new Error("Invalid scale");
  // ...
}

// Use closure for truly private state (not _underscore convention)
function createSafe(value) {
  let secret = value;
  return {
    getSecret() { return secret; },
    setSecret(v) { secret = v; }
  };
}

// Provide sensible defaults using destructuring with defaults
function createConfig({ host = "localhost", port = 8080, ssl = false } = {}) {
  return { host, port, ssl };
}
```

### Performance Considerations

- Factory functions create new method objects on every invocation unless methods are shared via prototype delegation (`Object.create`).
- For performance-critical code creating thousands of objects, use prototype-based factories or classes.
- Closures in factories keep references to the enclosing scope, which can prevent garbage collection if the factory objects live long. Be mindful of large scopes captured by closures.

### Interview Questions

**Q: What is the difference between a factory function and a constructor function?**
A: A factory function is a regular function that returns a new object explicitly. A constructor function is called with `new`, automatically creates `this`, and implicitly returns it. Factory functions avoid `this` confusion, don't require `new`, and can easily encapsulate private state via closures.

**Q: When would you choose a factory function over a class?**
A: Factory functions are preferable when: (1) You need private state via closures. (2) The creation logic is complex or conditional. (3) You want to compose behaviors from multiple sources. (4) You don't need `instanceof` checks. (5) You want to avoid `this` binding issues.

**Q: How do you implement the abstract factory pattern in JavaScript?**
A: An abstract factory is a factory that returns factories. You define an interface (implicitly, since JS has no interfaces) that multiple concrete factories implement, each creating a family of related objects. The top-level function takes a parameter (like `theme` or `platform`) and returns the appropriate concrete factory.

### Coding Challenges

```javascript
// Challenge 1: Implement a NotificationFactory that creates different
// notification types (email, SMS, push) based on a type parameter.
// Each type should have a send(message) method.

// Challenge 2: Create a createQueryBuilder factory that builds SQL queries
// using method chaining: query.select("*").from("users").where("age > 18");

// Challenge 3: Implement an EntityFactory with a registry pattern that
// allows registering entity types and creating them by type name.
```

### Related Topics

- Constructor functions and the `new` keyword
- `Object.create()` and prototypal inheritance
- Composition vs inheritance
- Dependency injection (factories enable DI)

## Object Creation Patterns

### What It Is

Object creation patterns encompass all the ways JavaScript provides to create objects: object literals, constructor functions, `Object.create()`, classes, and factory functions. Each pattern has different characteristics regarding inheritance, performance, and memory usage.

### Why It Is Important

Choosing the right object creation pattern affects code maintainability, performance, memory consumption, and team readability. Understanding all patterns allows developers to make informed decisions based on the specific requirements of each situation.

### How It Works Internally

JavaScript's object creation always goes through the `[[Prototype]]` chain. Whether using literals, constructors, or `Object.create()`, the engine sets up the object's internal prototype link. Factory functions are unique in that they don't use the prototype chain by default (though they can via `Object.create`).

### Syntax

```javascript
// 1. Object literal
const obj = { key: "value" };

// 2. Constructor function
function Person(name) { this.name = name; }
const p = new Person("Alice");

// 3. Object.create()
const proto = { greet() { return "Hi"; } };
const obj = Object.create(proto);

// 4. Class (ES6)
class Animal {
  constructor(name) { this.name = name; }
}

// 5. Factory function
function createX() { return { key: "value" }; }
```

### Beginner Examples

```javascript
// Object literal - simplest
const book = {
  title: "JavaScript: The Good Parts",
  author: "Douglas Crockford",
  getSummary() {
    return `${this.title} by ${this.author}`;
  }
};

// Constructor function
function Car(make, model, year) {
  this.make = make;
  this.model = model;
  this.year = year;
  this.getAge = function() {
    return new Date().getFullYear() - this.year;
  };
}
Car.prototype.getInfo = function() {
  return `${this.year} ${this.make} ${this.model}`;
};
const myCar = new Car("Honda", "Civic", 2020);

// Class syntax
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }
  get area() { return this.width * this.height; }
  get perimeter() { return 2 * (this.width + this.height); }
}
const rect = new Rectangle(10, 5);
```

### Intermediate Examples

```javascript
// Comparing memory usage: methods on prototype vs on instance
function InstanceMethod(name) {
  this.name = name;
  this.greet = function() { return `Hi, ${this.name}`; };
  // Each instance gets its own greet function - wasteful
}

function PrototypeMethod(name) {
  this.name = name;
}
PrototypeMethod.prototype.greet = function() {
  return `Hi, ${this.name}`;
};
// All instances share the same greet function

// Object.create with property descriptors
const animal = {
  breathe() { return "breathing"; }
};

const dog = Object.create(animal, {
  bark: {
    value: function() { return "woof!"; },
    writable: true,
    enumerable: false,
    configurable: true
  },
  name: {
    value: "Rex",
    writable: true,
    enumerable: true,
    configurable: false
  }
});

// Mixin pattern with object spread
const canEat = {
  eat() { return `${this.name} is eating`; }
};

const canWalk = {
  walk() { return `${this.name} is walking`; }
};

function createDog(name) {
  return {
    name,
    bark() { return "woof!"; },
    ...canEat,
    ...canWalk
  };
}
```

### Advanced Examples

```javascript
// Fluent API with builder pattern
class QueryBuilder {
  constructor() {
    this.clauses = [];
    this.params = [];
  }

  select(fields) {
    this.fields = fields;
    return this;
  }

  from(table) {
    this.table = table;
    return this;
  }

  where(condition, ...args) {
    this.clauses.push(`WHERE ${condition}`);
    this.params.push(...args);
    return this;
  }

  orderBy(field, direction = "ASC") {
    this.clauses.push(`ORDER BY ${field} ${direction}`);
    return this;
  }

  limit(count) {
    this.clauses.push(`LIMIT ${count}`);
    return this;
  }

  build() {
    let sql = `SELECT ${this.fields || "*"} FROM ${this.table}`;
    if (this.clauses.length) sql += " " + this.clauses.join(" ");
    return { sql, params: this.params };
  }
}

const query = new QueryBuilder()
  .select("id, name, email")
  .from("users")
  .where("age > ?", 18)
  .orderBy("name")
  .limit(10)
  .build();

// Pool object pattern (object reuse for performance)
class ParticlePool {
  constructor(size) {
    this.pool = new Array(size).fill(null).map(() => ({
      x: 0, y: 0, vx: 0, vy: 0, active: false
    }));
    this.activeCount = 0;
  }

  create(x, y, vx, vy) {
    if (this.activeCount >= this.pool.length) return null;
    const p = this.pool[this.activeCount++];
    p.x = x; p.y = y; p.vx = vx; p.vy = vy; p.active = true;
    return p;
  }

  release() {
    if (this.activeCount > 0) {
      this.pool[--this.activeCount].active = false;
    }
  }
}
```

### Real-World Use Cases

- **ORM entities**: Sequelize/Mongoose models use class instances to represent database rows.
- **State management**: Redux actions and state objects are plain objects (literal pattern).
- **Configuration objects**: Deeply nested configs use object literals with defaults.
- **Game development**: Object pools for particles, bullets, enemies (pool pattern).

### Common Mistakes

```javascript
// Mistake 1: Forgetting `new` with constructors
function Person(name) {
  this.name = name;
}
const p = Person("Alice"); // `this` refers to global/window, no error in non-strict mode
console.log(global.name); // "Alice" - leaked to global!

// Mistake 2: Returning non-primitive from constructor
function BadConstructor() {
  return { custom: "object" }; // Overrides `this` - returns this object instead
}
const b = new BadConstructor();
console.log(b instanceof BadConstructor); // false!

// Mistake 3: Shared mutable state on prototype
function SharedState() {}
SharedState.prototype.items = [];
const a = new SharedState();
const b = new SharedState();
a.items.push("hello");
console.log(b.items); // ["hello"] - shared!
```

### Best Practices

```javascript
// Prefer object literals for simple data containers
const point = { x: 10, y: 20 };

// Use classes when you need methods and encapsulation
class Temperature {
  #celsius; // true private fields
  constructor(celsius) { this.#celsius = celsius; }
  get fahrenheit() { return this.#celsius * 9/5 + 32; }
}

// Use Object.create() when you need fine-grained property control
const frozen = Object.create(null); // no prototype - true dictionary
```

### Performance Considerations

- **Object literals** are fastest for creation and have the smallest memory footprint.
- **Classes** (and constructors with prototypes) are memory-efficient for many instances because methods are shared.
- **Factory functions with closures** create new function objects per instance, using more memory.
- **Object pools** reduce GC pressure in high-frequency object creation scenarios.

### Interview Questions

**Q: What are the pros and cons of using Object.create(null)?**
A: `Object.create(null)` creates an object with no prototype, meaning it has no `toString`, `hasOwnProperty`, or other inherited methods. This is useful for pure dictionaries where you want to avoid property name collisions with the prototype chain. The con is that you lose all Object.prototype methods.

**Q: How do you implement the singleton pattern using object creation patterns?**
A: Using a factory that caches and returns the same instance: `const getInstance = (() => { let instance; return () => instance || (instance = createSomething()); })();`

**Q: What is the difference between `Object.create(proto)` and `new Constructor()`?**
A: `Object.create(proto)` creates a new object with `proto` as its prototype, but does not run any constructor code. `new Constructor()` creates a new object with `Constructor.prototype` as its prototype AND runs the constructor function to initialize instance properties.

### Coding Challenges

```javascript
// Challenge 1: Implement a LoggerFactory that returns loggers with different
// transport methods (console, file, HTTP). Use Object.create for prototype sharing.

// Challenge 2: Create a Builder pattern class for constructing HTTP requests
// with method chaining: new RequestBuilder().url("/api").method("POST").body(data).headers({"Content-Type": "application/json"});

// Challenge 3: Implement an inflatable object pool for WebGL vertex data
// that pre-allocates objects and reuses them to avoid garbage collection.
```

### Related Topics

- `new` keyword and constructor behavior
- `Object.create()` and `Object.assign()`
- ES6 class syntax
- Prototypal inheritance
- Property descriptors and `Object.defineProperty()`

## Composition Over Inheritance

### What It Is

Composition over inheritance is a design principle where you build objects by composing smaller, focused behaviors rather than inheriting from a deep class hierarchy. Instead of an "is-a" relationship (a Dog is an Animal), composition uses "has-a" or "behaves-as" relationships (a Dog has the ability to eat, walk, bark).

### Why It Is Important

Deep inheritance hierarchies become fragile and rigid. Changes in base classes can unexpectedly break derived classes (the "fragile base class problem"). Composition provides greater flexibility, easier testing, better code reuse, and avoids the diamond problem of multiple inheritance.

### How It Works Internally

JavaScript's prototypal inheritance is inherently more compositional than classical inheritance. The prototype chain forms a flexible delegation mechanism, and JavaScript's dynamic nature allows objects to be extended at runtime. Composition can be implemented via:
- **Mixin functions**: Copying methods from one object to another
- **Object.assign()**: Composing objects from multiple sources
- **Function composition**: Pipe/compose patterns
- **Dependency injection**: Passing capabilities as parameters

### Syntax

```javascript
// Mixin composition
const canEat = { eat() { console.log("Eating"); } };
const canWalk = { walk() { console.log("Walking"); } };
const canBark = { bark() { console.log("Woof!"); } };

const dog = Object.assign({}, canEat, canWalk, canBark);

// Factory composition
function createDog(name) {
  return {
    name,
    ...canEat,
    ...canWalk,
    ...canBark
  };
}

// Functional composition
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);
```

### Beginner Examples

```javascript
// Inheritance approach (problematic)
class Animal {
  constructor(name) { this.name = name; }
  eat() { return `${this.name} eats`; }
  breathe() { return `${this.name} breathes`; }
}

class Mammal extends Animal {
  constructor(name) { super(name); }
  walk() { return `${this.name} walks`; }
}

class Bird extends Animal {
  constructor(name) { super(name); }
  fly() { return `${this.name} flies`; }
}

// What if we need a flying mammal? Bat? Pegasus?
// Deep hierarchy becomes awkward quickly.

// Composition approach (flexible)
const eater = (state) => ({
  eat() { return `${state.name} eats`; }
});

const breather = (state) => ({
  breathe() { return `${state.name} breathes`; }
});

const walker = (state) => ({
  walk() { return `${state.name} walks`; }
});

const flyer = (state) => ({
  fly() { return `${state.name} flies`; }
});

function createMammal(name) {
  const state = { name };
  return { ...eater(state), ...breather(state), ...walker(state) };
}

function createBat(name) {
  const state = { name };
  return { ...eater(state), ...breather(state), ...walker(state), ...flyer(state) };
}

const bat = createBat("Bruce");
console.log(bat.fly()); // Bruce flies
console.log(bat.eat()); // Bruce eats
console.log(bat.walk()); // Bruce walks
```

### Intermediate Examples

```javascript
// More robust mixin pattern with conflict resolution
function mixin(target, ...sources) {
  for (const source of sources) {
    for (const [key, value] of Object.entries(source)) {
      if (target.hasOwnProperty(key)) continue; // Don't override existing
      Object.defineProperty(target, key, Object.getOwnPropertyDescriptor(source, key));
    }
  }
  return target;
}

const withLogging = {
  log(level, message) {
    console.log(`[${level.toUpperCase()}] ${this.name}: ${message}`);
  }
};

const withTimestamps = {
  getTimestamp() {
    return new Date().toISOString();
  }
};

const withValidation = (rules) => ({
  validate(data) {
    for (const [field, validator] of Object.entries(rules)) {
      if (data[field] !== undefined && !validator(data[field])) {
        throw new Error(`Validation failed for ${field}`);
      }
    }
    return true;
  }
});

// Creating a service with composed behaviors
function createUserService() {
  const state = { name: "UserService" };
  return mixin(
    {},
    withLogging,
    withTimestamps,
    withValidation({
      email: (v) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v),
      age: (v) => v >= 0 && v <= 150
    })
  );
}

// Functional mixin approach
function withAbility(state, ability) {
  return { ...state, ...ability };
}

const canSwim = (state) => ({
  swim() { return `${state.name} swims`; }
});

const duck = [eater, swimer, flyer, walker].reduce(
  (obj, mixin) => ({ ...obj, ...mixin(obj) }),
  { name: "Daffy" }
);
```

### Advanced Examples

```javascript
// Full functional composition with pipe
const pipe = (...fns) => (x) => fns.reduce((v, f) => f(v), x);

const addLogging = (target) => ({
  ...target,
  log: (msg) => console.log(`[${target.name}] ${msg}`)
});

const addPersistence = (target) => ({
  ...target,
  save: () => localStorage.setItem(target.id, JSON.stringify(target.state())),
  load: () => JSON.parse(localStorage.getItem(target.id))
});

const addValidation = (target, schema) => ({
  ...target,
  validate: (data) => {
    const errors = [];
    for (const [key, rules] of Object.entries(schema)) {
      for (const [rule, param] of Object.entries(rules)) {
        if (rule === "required" && !data[key]) {
          errors.push(`${key} is required`);
        }
        if (rule === "minLength" && data[key]?.length < param) {
          errors.push(`${key} must be at least ${param} characters`);
        }
      }
    }
    return errors.length ? errors : null;
  }
});

// Base object factory, then compose
function createBaseStore(name) {
  return {
    name,
    _data: {},
    state() { return { ...this._data }; },
    set(key, value) { this._data[key] = value; },
    get(key) { return this._data[key]; }
  };
}

const createEnhancedStore = (name, validationSchema) => pipe(
  addLogging,
  (s) => addPersistence(s),
  (s) => addValidation(s, validationSchema || {})
)(createBaseStore(name));

// Strategy pattern using composition
const sortingStrategies = {
  byName: (a, b) => a.name.localeCompare(b.name),
  byPrice: (a, b) => a.price - b.price,
  byDate: (a, b) => new Date(b.date) - new Date(a.date)
};

function withSorting(base, strategy) {
  return {
    ...base,
    sort(data) {
      return [...data].sort(strategy);
    }
  };
}

const priceSorter = withSorting({ name: "PriceSorter" }, sortingStrategies.byPrice);
```

### Real-World Use Cases

- **React Hooks**: Hooks are a form of composition, composing state and effects into functional components.
- **Express middleware**: Middleware functions compose to form request processing pipelines.
- **Redux middleware**: `applyMiddleware` composes middleware functions around the dispatch function.
- **Angular services**: Services are composed via dependency injection rather than inheritance.
- **Mongoose plugins**: Plugins mix in functionality to schemas.

### Common Mistakes

```javascript
// Mistake 1: Mutating shared mixin objects
const sharedMixin = { items: [] };
const a = { ...sharedMixin };
const b = { ...sharedMixin };
a.items.push("x"); // [ 'x' ] - Wait, spread creates a shallow copy
// Actually with spread, each gets its own items array since spread creates a shallow copy
// But with Object.assign or direct reference, this can be an issue

// Mistake 2: Method name collisions
const mixinA = { save: () => "saving A" };
const mixinB = { save: () => "saving B" };
const obj = { ...mixinA, ...mixinB }; // mixinA.save is overwritten!

// Mistake 3: Composition overkill - not everything needs to be composed
// Simple objects don't need mixins
```

### Best Practices

```javascript
// Prefer small, focused mixins/modules over large, monolithic ones
// Each behavior should have a single responsibility

// Use factory functions with composition for most cases
function createValidator(rules) {
  return {
    validate: (data) => {
      // use rules
    }
  };
}

// Name mixins clearly (withAdject, withVerb naming)
// Prefer composition at the same level of abstraction

// Test mixins independently
// Composed objects should be testable by testing their component parts

// Use TypeScript interfaces to define mixin contracts
/*
interface Loggable {
  log(level: string, message: string): void;
}
*/

// Only use inheritance when there's a genuine "is-a" relationship
// AND the subclass doesn't need to override much behavior
```

### Performance Considerations

- Object spread (`...`) creates shallow copies, which has O(n) complexity per composition.
- For performance-critical paths, consider mutation-based composition (e.g., `Object.assign(target, source)`) instead of spread.
- Functional composition (pipe/compose) creates new functions without allocating objects.
- Deeply nested compositions can create long prototype chains or large objects; keep composition flat.

### Comparison: Inheritance vs Composition

| Aspect | Inheritance | Composition |
|--------|-------------|-------------|
| Relationship | "is-a" | "has-a" / "behaves-as" |
| Code reuse | Through base class | Through mixins/delegation |
| Flexibility | Rigid hierarchy | Flexible combinations |
| Encapsulation | Breaks encapsulation (subclasses depend on parent) | Preserves encapsulation |
| Testing | Harder (must test full hierarchy) | Easier (test components independently) |
| Override | Easy via method override | Requires explicit delegation |
| Memory | Shared via prototype | Copies methods unless shared |
| JS idioms | Classes, `extends` | Factories, mixins, `Object.assign` |

### Interview Questions

**Q: Why is composition preferred over inheritance in JavaScript?**
A: JavaScript is a prototypal language designed for delegation, not classical inheritance. Deep inheritance hierarchies are rigid and fragile, while composition provides flexibility through JavaScript's dynamic object system. Composition also aligns better with JavaScript's functional capabilities and avoids the fragile base class problem.

**Q: How do you handle the "diamond problem" in JavaScript?**
A: JavaScript doesn't have classical multiple inheritance (the diamond problem). With composition, you explicitly merge behaviors, and conflicts are resolved by ordering (last-in wins with spread/assign). This explicit conflict resolution is simpler and more predictable.

**Q: What is the difference between object.assign and spread operator for composition?**
A: `Object.assign(target, ...sources)` mutates the target object and returns it. `{ ...source1, ...source2 }` creates a new object and never mutates existing ones. Both perform shallow copies and resolve conflicts by later sources overriding earlier ones.

### Coding Challenges

```javascript
// Challenge 1: Create a composable behavior system where you can create
// game entities with mixins like withHealth, withMana, withPosition, withAI.
// Compose a "Dragon" that has all of them, and a "Tree" that only has position and health.

// Challenge 2: Implement a pipe function that composes middleware
// in the style of Express/Koa, where each middleware calls next().
// The composed function should accept (request, response) and return a promise.

// Challenge 3: Build a logging system using composition where you can mix
// and match destinations (console, file, HTTP, database) and formatters
// (JSON, plain text, HTML). Use functional mixins to compose loggers.
```

### Related Topics

- Mixin pattern
- Functional programming (function composition, currying)
- Dependency injection
- SOLID principles
- React Hooks (composition in UI)
