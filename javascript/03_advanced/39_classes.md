# Classes

## Introduction

ES6 introduced the `class` syntax, providing a cleaner, more familiar way to create constructor functions and prototype-based inheritance. Classes are syntactic sugar over JavaScript's existing prototypal inheritance model, but they offer additional features like static methods, private fields, and computed property names. Despite being "just sugar," classes have become the standard way to define object blueprints in modern JavaScript.

---

## class declaration

### What It Is

A `class` declaration creates a new class with a given name using prototype-based inheritance. Classes are syntactical sugar over JavaScript's existing prototype-based inheritance, providing a more declarative and familiar syntax for defining object constructors and their methods.

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hello, I'm ${this.name}`;
  }
}

const alice = new Person("Alice");
console.log(alice.greet()); // "Hello, I'm Alice"
```

### Why It Is Important

Class syntax provides:
- A clearer, more concise way to define constructors and prototypes.
- Built-in support for inheritance via `extends`.
- The ability to define static methods, getters/setters, and computed properties.
- Better integration with TypeScript and other transpilers.
- A familiar syntax for developers coming from class-based languages.

### How It Works Internally

Despite the class syntax, JavaScript still uses prototypal inheritance. The class declaration:

1. Creates a function named `Person` (the constructor).
2. Copies all methods defined inside the class body to `Person.prototype`.
3. Static methods are attached to `Person` itself.
4. The class is not hoisted (unlike function declarations).
5. Code inside the class body runs in strict mode automatically.

```javascript
class Person {
  constructor(name) { this.name = name; }
  greet() { return `Hi, ${this.name}`; }
}

// Internally equivalent to:
function Person(name) {
  this.name = name;
}
Person.prototype.greet = function () {
  return `Hi, ${this.name}`;
};
```

### Syntax

```javascript
// Basic class declaration
class ClassName {
  constructor(param) {
    // initialization
  }
  method1() {}
  method2() {}
}

// Class expression (anonymous)
const MyClass = class {
  constructor() {}
};

// Class expression (named)
const MyClass = class MyNamedClass {
  constructor() {}
};
```

### Beginner Examples

```javascript
// Example 1: Basic class
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} makes a sound`;
  }
}

const dog = new Animal("Rex");
console.log(dog.speak()); // "Rex makes a sound"
console.log(dog instanceof Animal); // true

// Example 2: Class with getters/setters
class Temperature {
  constructor(celsius) {
    this._celsius = celsius;
  }
  get fahrenheit() {
    return this._celsius * 9 / 5 + 32;
  }
  set fahrenheit(value) {
    this._celsius = (value - 32) * 5 / 9;
  }
  get celsius() {
    return this._celsius;
  }
}

const temp = new Temperature(25);
console.log(temp.fahrenheit); // 77
temp.fahrenheit = 100;
console.log(temp.celsius); // ~37.78

// Example 3: Computed method names
const methodName = "print";
class Printer {
  [methodName](msg) {
    console.log(msg);
  }
}
new Printer().print("Hello"); // "Hello"
```

### Intermediate Examples

```javascript
// Example 1: Class with computed properties
const SYM = Symbol("secret");
class SecureContainer {
  constructor(secret) {
    this[SYM] = secret;
  }
  getSecret() {
    return this[SYM];
  }
}
const container = new SecureContainer("classified");
console.log(container.getSecret()); // "classified"
console.log(Object.keys(container)); // [] (Symbol keys hidden)

// Example 2: Generator methods
class Range {
  constructor(start, end) {
    this.start = start;
    this.end = end;
  }
  *[Symbol.iterator]() {
    for (let i = this.start; i <= this.end; i++) {
      yield i;
    }
  }
}

const range = new Range(1, 5);
console.log([...range]); // [1, 2, 3, 4, 5]

// Example 3: Class type checking
function isClass(obj, cls) {
  return obj instanceof cls;
}

// Example 4: Class with default values
class Config {
  constructor(options = {}) {
    this.host = options.host || "localhost";
    this.port = options.port || 3000;
    this.debug = options.debug ?? false;
  }
}
```

### Advanced Examples

```javascript
// Example 1: Singleton with class
class Singleton {
  constructor() {
    if (Singleton.instance) {
      return Singleton.instance;
    }
    this.data = Math.random();
    Singleton.instance = this;
  }
}
const a = new Singleton();
const b = new Singleton();
console.log(a === b); // true
console.log(a.data === b.data); // true

// Example 2: Abstract-like base class
class AbstractModel {
  constructor() {
    if (new.target === AbstractModel) {
      throw new TypeError("Cannot instantiate abstract class");
    }
  }
  save() {
    if (this.validate()) {
      this._persist();
    }
  }
  validate() {
    throw new Error("Must implement validate()");
  }
  _persist() {
    throw new Error("Must implement _persist()");
  }
}

class User extends AbstractModel {
  constructor(name) {
    super();
    this.name = name;
  }
  validate() {
    return this.name.length > 0;
  }
  _persist() {
    console.log(`Saving user: ${this.name}`);
  }
}

// Example 3: Class as parameter type
function createInstance(ClassType, ...args) {
  return new ClassType(...args);
}
const person = createInstance(Person, "Bob");
console.log(person.greet());

// Example 4: Factory method pattern
class Logger {
  constructor(level) {
    this.level = level;
  }
  static createConsole() {
    return new Logger("info");
  }
  static createFile(path) {
    const logger = new Logger("debug");
    logger.path = path;
    return logger;
  }
}
```

### Real-World Use Cases

- **React components** — `class MyComponent extends React.Component`.
- **Custom elements** — `class MyElement extends HTMLElement`.
- **ORM models** — Mongoose, Sequelize models.
- **Service classes** — API clients, database repositories.
- **Value objects** — DTOs, configuration objects.

### Common Mistakes

- **Forgetting `new` when instantiating** — `Person("Alice")` throws TypeError.
- **Using arrow functions for class methods** — They don't have their own `this` (useful for callbacks, but not for prototype methods).
- **Assuming classes are hoisted** — They are not, unlike function declarations.
- **Creating circular dependencies with class imports.**
- **Putting too much logic in constructors** — Constructors should only initialize properties.

### Best Practices

- Prefer class syntax over manual prototype manipulation.
- Keep constructors simple — only initialize properties.
- Use getters and setters for computed or validated properties.
- Use static methods for factory functions and utility methods.
- Follow the Single Responsibility Principle — each class should have one purpose.

### Performance Considerations

- Classes are optimized by V8 the same way as constructor functions (they are the same internally).
- Methods defined in the class body go on the prototype (shared, memory efficient).
- Arrow functions in class fields create per-instance closures (more memory, but useful for callbacks).
- Class field declarations (ES2022) are own properties, not prototype properties.
- Classes run in strict mode, which allows some engine optimizations.

### Interview Questions

1. **Are JavaScript classes different from classes in Java/C++?**
   Yes. JavaScript classes are syntactic sugar over prototypal inheritance. They use delegation, not classical inheritance.

2. **Are classes hoisted?**
   No. Unlike function declarations, class declarations are not hoisted. They must be defined before use.

3. **What is the difference between `class` and a constructor function?**
   Classes offer cleaner syntax, strict mode enforcement, and features like `extends`, `super`, static methods, and private fields.

4. **Can you use `new` with a class?**
   Yes, that's the only way to instantiate a class. Calling a class without `new` throws a TypeError.

### Coding Challenges

1. **Implement a class-based event emitter.**
2. **Create a class that validates its properties on set.**
3. **Build a generic `Model` class with CRUD operations.**
4. **Implement a stack and queue using classes.**
5. **Create a class that automatically generates unique IDs for instances.**

### Related Topics

- Constructor functions
- Prototype
- `extends`
- `super`
- Static methods
- Private fields

---

## constructor method

### What It Is

The `constructor` method is a special method in a class that is automatically called when an object is instantiated with `new`. It is used to initialize instance properties and perform any setup logic. A class can only have one constructor — if you try to define more than one, it throws a `SyntaxError`.

```javascript
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
    this.createdAt = new Date();
  }
}

const user = new User("Alice", "alice@example.com");
console.log(user.name); // "Alice"
console.log(user.createdAt); // Date object
```

### Why It Is Important

The constructor is the entry point for object initialization:
- It sets up initial property values.
- It validates constructor arguments.
- It performs setup logic (subscriptions, API calls, etc.).
- It calls `super()` in derived classes to initialize the parent class.
- It provides a controlled way to create valid object instances.

### How It Works Internally

When `new ClassName(args)` is called:
1. A new empty object is created.
2. The object's `[[Prototype]]` is set to `ClassName.prototype`.
3. The `constructor` function is called with `this` bound to the new object.
4. If the constructor returns a non-primitive object, that object is returned instead.
5. Otherwise, the newly created object is returned.

If no explicit constructor is defined, the engine provides a default empty constructor.

### Syntax

```javascript
class MyClass {
  constructor(param1, param2) {
    // Initialize properties
    this.prop1 = param1;
    this.prop2 = param2;
  }
}

// Default constructor (if not defined)
class DefaultClass {} // implicitly: constructor() {}

// Constructor with defaults
class Config {
  constructor(options = {}) {
    this.host = options.host || "localhost";
  }
}
```

### Beginner Examples

```javascript
// Example 1: Basic constructor
class Book {
  constructor(title, author, year) {
    this.title = title;
    this.author = author;
    this.year = year;
    this.isAvailable = true;
  }
}

const book = new Book("1984", "George Orwell", 1949);
console.log(book.title); // "1984"
console.log(book.isAvailable); // true

// Example 2: Constructor with validation
class Age {
  constructor(years) {
    if (years < 0 || years > 150) {
      throw new RangeError("Invalid age");
    }
    this.years = years;
  }
}

// Example 3: Default constructor
class NoConstructor {} // has implicit empty constructor
const obj = new NoConstructor();
console.log(obj instanceof NoConstructor); // true
```

### Intermediate Examples

```javascript
// Example 1: Constructor with computed properties
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
    this.area = width * height; // computed in constructor
    this.perimeter = 2 * (width + height);
  }
}

// Example 2: Private initialization via constructor
class Connection {
  constructor(url) {
    this.url = url;
    this._connected = false;
    this._retries = 0;
  }
  async connect() {
    try {
      await fetch(this.url);
      this._connected = true;
    } catch {
      this._retries++;
    }
  }
}

// Example 3: Constructor overloading (simulated)
class Vector {
  constructor(x, y) {
    if (x instanceof Vector) {
      this.x = x.x;
      this.y = x.y;
    } else if (Array.isArray(x)) {
      this.x = x[0];
      this.y = x[1] ?? 0;
    } else {
      this.x = x ?? 0;
      this.y = y ?? 0;
    }
  }
}
```

### Advanced Examples

```javascript
// Example 1: Constructor returning another object
class Singleton {
  constructor() {
    if (Singleton.instance) {
      return Singleton.instance;
    }
    Singleton.instance = this;
    this.timestamp = Date.now();
  }
}

// Example 2: Constructor delegation
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class Circle {
  constructor(center, radius) {
    if (!(center instanceof Point)) {
      center = new Point(center.x ?? 0, center.y ?? 0);
    }
    this.center = center;
    this.radius = radius;
  }
}

// Example 3: Async initialization pattern
class AsyncDataLoader {
  constructor() {
    this.data = null;
    this._initPromise = this._initialize();
  }
  async _initialize() {
    const response = await fetch("/api/data");
    this.data = await response.json();
  }
  async getData() {
    await this._initPromise;
    return this.data;
  }
}

// Usage
(async () => {
  const loader = new AsyncDataLoader();
  const data = await loader.getData();
})();
```

### Real-World Use Cases

- **Dependency injection** — Services passed through constructors.
- **Configuration objects** — Parsing and validating config in constructor.
- **DTOs (Data Transfer Objects)** — Validating and shaping data on construction.
- **Component initialization** — Setting up internal state in UI components.
- **Event setup** — Binding event handlers in constructors.

### Common Mistakes

- **Returning a value from constructor** — Silently ignored (unless it's an object).
- **Doing async work in constructor** — Cannot await in constructor (use init method).
- **Over-complicating constructors with too much logic** — Keep them simple.
- **Forgetting `super()` in derived class constructors** — ReferenceError.
- **Using `this` before calling `super()`** — ReferenceError in derived classes.

### Best Practices

- Keep constructors simple — initialize properties, don't do heavy work.
- Validate constructor arguments and throw early if invalid.
- Use default parameter values for optional configuration.
- Avoid async operations in constructors — use a static factory or init method.
- Consider using object destructuring for constructors with many parameters.

### Performance Considerations

- Constructor code runs once per instantiation — keep it fast.
- Heavy work in constructors slows down object creation.
- V8 inlines simple constructors for performance.
- Property assignments in constructors help V8 create efficient hidden classes.
- Avoid dynamic property names in constructors (they prevent hidden class optimization).

### Interview Questions

1. **What happens if you don't define a constructor?**
   A default empty constructor is added automatically.

2. **Can a class have multiple constructors?**
   No. But you can simulate overloading with conditional logic in one constructor.

3. **What is the return value of `new ClassName()`?**
   The newly created instance, unless the constructor returns a different object.

4. **Can you use `return` in a constructor?**
   Yes, but if it returns a primitive, it's ignored. If it returns an object, that object is used as the instance.

### Coding Challenges

1. **Implement a constructor that validates all arguments.**
2. **Create a factory method that replaces constructor overloading.**
3. **Build a constructor that auto-generates IDs using a static counter.**
4. **Write a class that enforces a maximum number of instances via constructor.**

### Related Topics

- `new` operator
- `super()`
- Instance methods
- Class fields
- Factory pattern

---

## Instance methods

### What It Is

Instance methods are functions defined on the class prototype that operate on individual instances of the class. They have access to instance properties via `this` and are shared across all instances (stored on `ClassName.prototype`, not on each instance).

```javascript
class Calculator {
  constructor(value = 0) {
    this.value = value;
  }
  add(n) {
    this.value += n;
    return this;
  }
  subtract(n) {
    this.value -= n;
    return this;
  }
  getResult() {
    return this.value;
  }
}

const calc = new Calculator(10);
calc.add(5).subtract(3);
console.log(calc.getResult()); // 12
```

### Why It Is Important

Instance methods define the behavior of objects:
- They provide the API for interacting with instances.
- They are shared across all instances (memory efficient).
- They can access and modify instance state via `this`.
- They support method chaining when returning `this`.
- They enable polymorphism through inheritance.

### How It Works Internally

Instance methods defined in the class body are automatically added to `ClassName.prototype`. When a method is called, the engine:

1. Resolves the method through the prototype chain.
2. Sets `this` to the instance the method was called on.
3. Executes the method in the context of the instance.

This is identical to how methods work on constructor function prototypes.

### Syntax

```javascript
class MyClass {
  constructor() { /* ... */ }
  
  // Instance method
  myMethod(arg) {
    // 'this' refers to the instance
    this.property = arg;
  }
  
  // Getter
  get computed() { return this._value * 2; }
  
  // Setter
  set computed(val) { this._value = val / 2; }
  
  // Generator method
  *items() { yield* this._items; }
}
```

### Beginner Examples

```javascript
// Example 1: Basic instance methods
class Dog {
  constructor(name) {
    this.name = name;
  }
  bark() {
    return `${this.name} says woof!`;
  }
  eat(food) {
    return `${this.name} eats ${food}`;
  }
}

const rex = new Dog("Rex");
console.log(rex.bark()); // "Rex says woof!"
console.log(rex.eat("meat")); // "Rex eats meat"

// Example 2: Method chaining
class StringBuilder {
  constructor(str = "") {
    this._str = str;
  }
  append(str) {
    this._str += str;
    return this;
  }
  prepend(str) {
    this._str = str + this._str;
    return this;
  }
  toString() {
    return this._str;
  }
}

const result = new StringBuilder("World")
  .prepend("Hello ")
  .append("!")
  .toString();
console.log(result); // "Hello World!"

// Example 3: Getter and setter
class User {
  constructor(first, last) {
    this.first = first;
    this.last = last;
  }
  get fullName() {
    return `${this.first} ${this.last}`;
  }
  set fullName(name) {
    [this.first, this.last] = name.split(" ");
  }
}
```

### Intermediate Examples

```javascript
// Example 1: Instance method with validation
class BankAccount {
  constructor(balance = 0) {
    this._balance = balance;
    this._transactions = [];
  }
  deposit(amount) {
    if (amount <= 0) throw new Error("Amount must be positive");
    this._balance += amount;
    this._transactions.push({ type: "deposit", amount, date: new Date() });
    return this;
  }
  withdraw(amount) {
    if (amount <= 0) throw new Error("Amount must be positive");
    if (amount > this._balance) throw new Error("Insufficient funds");
    this._balance -= amount;
    this._transactions.push({ type: "withdraw", amount, date: new Date() });
    return this;
  }
  get balance() {
    return this._balance;
  }
  get transactionHistory() {
    return [...this._transactions];
  }
}

// Example 2: Async instance method
class DataService {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
  }
  async get(endpoint) {
    const res = await fetch(`${this.baseUrl}${endpoint}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res.json();
  }
  async post(endpoint, data) {
    const res = await fetch(`${this.baseUrl}${endpoint}`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(data),
    });
    return res.json();
  }
}

// Example 3: Generator instance method
class PaginatedResults {
  constructor(data, pageSize = 10) {
    this.data = data;
    this.pageSize = pageSize;
  }
  *pages() {
    for (let i = 0; i < this.data.length; i += this.pageSize) {
      yield this.data.slice(i, i + this.pageSize);
    }
  }
}
```

### Advanced Examples

```javascript
// Example 1: Methods returning new instances (immutable)
class ImmutablePoint {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    Object.freeze(this);
  }
  move(dx, dy) {
    return new ImmutablePoint(this.x + dx, this.y + dy);
  }
  toString() {
    return `(${this.x}, ${this.y})`;
  }
}

const p1 = new ImmutablePoint(0, 0);
const p2 = p1.move(5, 3);
console.log(p1.toString()); // (0, 0)
console.log(p2.toString()); // (5, 3)

// Example 2: Symbol.iterator instance method
class Range {
  constructor(start, end) {
    this.start = start;
    this.end = end;
  }
  *[Symbol.iterator]() {
    for (let i = this.start; i <= this.end; i++) {
      yield i;
    }
  }
  toArray() {
    return [...this];
  }
}

const range = new Range(1, 4);
console.log(range.toArray()); // [1, 2, 3, 4]

// Example 3: Mixin injection of instance methods
const TimestampMixin = {
  timestamp() {
    return this._createdAt || (this._createdAt = new Date());
  },
  age() {
    return Date.now() - this.timestamp().getTime();
  },
};

class Document {
  constructor(content) {
    this.content = content;
  }
}
Object.assign(Document.prototype, TimestampMixin);

const doc = new Document("Hello");
console.log(doc.age()); // milliseconds since creation
```

### Real-World Use Cases

- **CRUD operations** — `save()`, `update()`, `delete()` on model instances.
- **State manipulation** — `push()`, `pop()` on stack instances.
- **Rendering** — `render()`, `update()` on UI component instances.
- **Validation** — `validate()`, `isValid()` on form instances.
- **Serialization** — `toJSON()`, `toString()` on data instances.

### Common Mistakes

- **Using arrow functions for instance methods (prototype)** — They are not on the prototype, they're per-instance fields.
- **Forgetting `this` context when passing methods as callbacks** — Use arrow function class fields or `.bind()`.
- **Naming collisions** — Overriding built-in methods like `toString()` incorrectly.
- **Mutating shared state in prototype methods** — Accidentally modifying static or class-level state.
- **Creating methods that are too long** — Break into smaller helper methods.

### Best Practices

- Use method chaining (return `this`) for fluent APIs.
- Use getters/setters for computed or validated properties.
- Keep methods focused on a single responsibility.
- Use descriptive method names that reveal intent.
- Document method behavior with JSDoc comments.

### Performance Considerations

- Prototype methods are shared memory — one function per method regardless of instances.
- Arrow function class fields create per-instance functions (more memory, faster property access).
- Method calls via prototype chain are very fast (V8 inline caching).
- Long prototype chains add negligible method lookup cost.
- Methods that are hot paths should be kept small for V8 to inline.

### Interview Questions

1. **Where are instance methods stored?**
   On `ClassName.prototype`, shared across all instances.

2. **What is the difference between a prototype method and an arrow function class field?**
   Prototype methods live on the prototype (shared). Arrow function fields live on each instance (own) and capture `this` lexically.

3. **Can instance methods be chained?**
   Yes, by returning `this` from each method.

4. **How do you make an instance method private?**
   Use the `#` prefix (private class fields) or prefix with `_` (convention only).

### Coding Challenges

1. **Create a class with methods that support immutable operations.**
2. **Implement a `toJSON` method that customizes JSON serialization.**
3. **Build a class that implements `Symbol.toPrimitive` for type coercion.**
4. **Create a fluent validation class with chained `check` methods.**

### Related Topics

- Prototype
- `this` binding
- Getters/setters
- Method chaining
- Symbol methods

---

## Static methods

### What It Is

Static methods are methods defined on the class itself, not on its prototype. They are called directly on the class (e.g., `ClassName.staticMethod()`) and cannot be called on instances. Static methods are used for utility functions, factory methods, and operations that don't require an instance.

```javascript
class MathUtils {
  static add(a, b) {
    return a + b;
  }
  static multiply(a, b) {
    return a * b;
  }
}

console.log(MathUtils.add(5, 3)); // 8
console.log(MathUtils.multiply(4, 2)); // 8
```

### Why It Is Important

Static methods provide:
- Utility functions grouped under a class namespace.
- Factory methods for creating instances.
- Singleton accessors.
- Helper functions that don't need instance state.
- Cleaner API design — separating instance behavior from class-level behavior.

### How It Works Internally

Static methods are added as own properties of the class constructor function (not on `ClassName.prototype`). When a static method is defined, it's essentially the same as:

```javascript
ClassName.staticMethod = function () { /* ... */ };
```

Inherited classes also inherit static methods via the `[[Prototype]]` chain of the class constructor (not the instance prototype chain).

### Syntax

```javascript
class MyClass {
  static myStaticMethod() {
    // 'this' refers to the class itself
  }
  
  static get myStaticGetter() {
    return someValue;
  }
  
  static {
    // Static initialization block (ES2022)
    this.secret = "config";
  }
}

// Calling static methods
MyClass.myStaticMethod();
```

### Beginner Examples

```javascript
// Example 1: Utility static methods
class StringUtils {
  static capitalize(str) {
    return str.charAt(0).toUpperCase() + str.slice(1);
  }
  static truncate(str, maxLength) {
    return str.length > maxLength ? str.slice(0, maxLength) + "..." : str;
  }
  static isEmpty(str) {
    return str.trim().length === 0;
  }
}

console.log(StringUtils.capitalize("hello")); // "Hello"
console.log(StringUtils.truncate("Long text here", 5)); // "Long ..."

// Example 2: Factory static method
class User {
  constructor(name, role) {
    this.name = name;
    this.role = role;
  }
  static createAdmin(name) {
    return new User(name, "admin");
  }
  static createGuest() {
    return new User("Guest", "guest");
  }
}

const admin = User.createAdmin("Alice");
console.log(admin.role); // "admin"

// Example 3: Static property
class Config {
  static appName = "MyApp";
  static version = "1.0.0";
}
console.log(Config.appName); // "MyApp"
```

### Intermediate Examples

```javascript
// Example 1: Singleton with static method
class Database {
  constructor(url) {
    this.url = url;
  }
  static getInstance(url) {
    if (!Database._instance) {
      Database._instance = new Database(url);
    }
    return Database._instance;
  }
  query(sql) {
    return `Executing: ${sql} on ${this.url}`;
  }
}

const db1 = Database.getInstance("mysql://localhost");
const db2 = Database.getInstance("mysql://localhost");
console.log(db1 === db2); // true

// Example 2: Static method with instance registry
class Shape {
  static registry = new Map();
  
  static register(name, shapeClass) {
    Shape.registry.set(name, shapeClass);
  }
  static create(name, ...args) {
    const shapeClass = Shape.registry.get(name);
    if (!shapeClass) throw new Error(`Unknown shape: ${name}`);
    return new shapeClass(...args);
  }
}

class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }
  area() { return Math.PI * this.radius ** 2; }
}
Shape.register("circle", Circle);

const circle = Shape.create("circle", 5);
console.log(circle.area()); // 78.54

// Example 3: Static initialization block
class AppConfig {
  static config;
  
  static {
    try {
      const raw = fs.readFileSync("config.json", "utf8");
      this.config = JSON.parse(raw);
    } catch {
      this.config = { mode: "development" };
    }
  }
}
```

### Advanced Examples

```javascript
// Example 1: Static inheritance
class Parent {
  static greet() {
    return "Hello from Parent";
  }
}

class Child extends Parent {}

console.log(Child.greet()); // "Hello from Child"
// Static methods are inherited via the constructor's [[Prototype]]

// Example 2: Static method polymorphism
class Serializer {
  static serialize(value) {
    if (value === null) return "null";
    if (typeof value === "string") return `"${value}"`;
    if (typeof value === "number") return String(value);
    if (Array.isArray(value)) return this.serializeArray(value);
    if (typeof value === "object") return this.serializeObject(value);
    return String(value);
  }
  static serializeArray(arr) {
    return `[${arr.map((v) => this.serialize(v)).join(",")}]`;
  }
  static serializeObject(obj) {
    const parts = Object.entries(obj).map(
      ([k, v]) => `"${k}":${this.serialize(v)}`
    );
    return `{${parts.join(",")}}`;
  }
}

console.log(Serializer.serialize({ a: 1, b: [2, 3] }));
// {"a":1,"b":[2,3]}

// Example 3: Static method for caching
class ExpensiveOperation {
  static cache = new WeakMap();
  
  static compute(obj) {
    if (this.cache.has(obj)) return this.cache.get(obj);
    const result = this._heavyComputation(obj);
    this.cache.set(obj, result);
    return result;
  }
  static _heavyComputation(obj) {
    // Simulated heavy work
    return Object.keys(obj).reduce((acc, k) => acc + obj[k], 0);
  }
}
```

### Real-World Use Cases

- **Factory methods** — `Promise.resolve()`, `Array.from()`.
- **Utility classes** — `Math` is essentially a static class.
- **Singleton access** — `Database.getInstance()`.
- **Builder patterns** — `Builder.create().withX().build()`.
- **Helper functions** — `Date.now()`, `Object.keys()`.

### Common Mistakes

- **Trying to call static methods on instances** — `instance.staticMethod()` is `undefined`.
- **Using `this` in static methods assuming it's an instance** — `this` refers to the class.
- **Forgetting that static methods are inherited** — Can cause unexpected behavior.
- **Overusing static methods for everything** — Not all utilities need to be static.
- **Mutating static state from instance methods** — Shared state issues.

### Best Practices

- Use static methods for operations that don't require instance state.
- Use static factory methods to provide multiple construction paths.
- Use static initialization blocks for complex static setup.
- Avoid overusing static methods — they can make testing harder.
- Document whether a method is static or instance in comments/TypeScript.

### Performance Considerations

- Static methods have no performance difference from regular functions.
- Static property access is slightly faster than instance property access (no prototype chain).
- Static methods defined in class body are created once.
- Static method inheritance creates a small prototype chain for the class constructor.

### Interview Questions

1. **What is the difference between a static method and an instance method?**
   Static methods are defined on the class itself and called on the class. Instance methods are defined on the prototype and called on instances.

2. **Are static methods inherited?**
   Yes, by derived classes (via the constructor's prototype chain).

3. **What does `this` refer to in a static method?**
   The class itself.

4. **Can static methods be async?**
   Yes: `static async fetchData() { return await fetch(...); }`.

### Coding Challenges

1. **Create a `DateUtils` class with static methods for date formatting.**
2. **Implement a `Cache` class with static methods for memoization.**
3. **Build a class registry using static methods.**
4. **Create a `Validator` class with static validation methods.**
5. **Implement a static factory that creates different instance types.**

### Related Topics

- Instance methods
- Class inheritance
- Static properties
- Factory pattern
- Singleton pattern

---

## Private fields (#)

### What It Is

Private fields are class properties that are truly private — they can only be accessed within the class body. They are denoted by the `#` prefix and were introduced in ES2022. Unlike the convention of prefixing with `_`, private fields are enforced by the JavaScript engine and cannot be accessed from outside the class.

```javascript
class Person {
  #name;
  #age;
  
  constructor(name, age) {
    this.#name = name;
    this.#age = age;
  }
  
  greet() {
    return `Hi, I'm ${this.#name} and I'm ${this.#age}`;
  }
}

const alice = new Person("Alice", 30);
console.log(alice.greet()); // "Hi, I'm Alice and I'm 30"
console.log(alice.#name); // SyntaxError: Private field '#name' must be declared
```

### Why It Is Important

Private fields provide:
- True encapsulation — privacy enforced by the engine, not by convention.
- No risk of external code depending on internal implementation details.
- Cleaner public API — only public methods are visible.
- Safer refactoring — internal changes don't affect external consumers.
- Better tooling support — IDEs can distinguish public vs private.

### How It Works Internally

Private fields are stored in an internal slot that is not accessible via property access or `Object.getOwnPropertyNames()`. The engine uses a "brand check" — when accessing `this.#field`, the engine verifies that the object is an instance of the class that declared the field. This check is done at runtime and cannot be bypassed.

Private fields are not part of the prototype — they are own properties stored in a special internal storage.

### Syntax

```javascript
class MyClass {
  // Field declarations (ES2022)
  #privateField;
  #privateFieldWithDefault = "default";
  
  constructor(value) {
    this.#privateField = value;
    // this.#undeclaredField = 1; // SyntaxError
  }
  
  #privateMethod() {
    return this.#privateField;
  }
  
  getPrivate() {
    return this.#privateMethod();
  }
}
```

### Beginner Examples

```javascript
// Example 1: Basic private fields
class Counter {
  #count = 0;
  
  increment() {
    this.#count++;
  }
  decrement() {
    this.#count--;
  }
  get value() {
    return this.#count;
  }
}

const counter = new Counter();
counter.increment();
counter.increment();
console.log(counter.value); // 2
console.log(counter.#count); // SyntaxError

// Example 2: Private fields with constructor
class User {
  #password;
  
  constructor(username, password) {
    this.username = username;
    this.#password = password;
  }
  
  checkPassword(attempt) {
    return this.#password === attempt;
  }
}

const user = new User("alice", "secret123");
console.log(user.checkPassword("secret123")); // true
console.log(user.#password); // SyntaxError

// Example 3: Private methods
class Logger {
  #logs = [];
  
  log(message) {
    this.#logs.push(message);
    this.#writeToConsole(message);
  }
  
  #writeToConsole(msg) {
    console.log(`[LOG]: ${msg}`);
  }
}
```

### Intermediate Examples

```javascript
// Example 1: Private field with getter
class Temperature {
  #celsius;
  
  constructor(celsius) {
    this.#celsius = celsius;
  }
  
  get fahrenheit() {
    return this.#celsius * 9 / 5 + 32;
  }
  
  set fahrenheit(value) {
    this.#celsius = (value - 32) * 5 / 9;
  }
}

const temp = new Temperature(25);
console.log(temp.fahrenheit); // 77
console.log(temp.#celsius); // SyntaxError

// Example 2: Private static fields
class Config {
  static #instance;
  static #secretKey = "default-key";
  
  static {
    this.#instance = new Config();
  }
  
  static getInstance() {
    return this.#instance;
  }
  
  static #hash(value) {
    return value.split("").reverse().join("");
  }
  
  static encrypt(value) {
    return this.#hash(value) + this.#secretKey;
  }
}

// Example 3: Private fields in inheritance
class Animal {
  #name;
  
  constructor(name) {
    this.#name = name;
  }
  
  getName() {
    return this.#name;
  }
}

class Dog extends Animal {
  #breed;
  
  constructor(name, breed) {
    super(name);
    this.#breed = breed;
  }
  
  getInfo() {
    return `${this.getName()} is a ${this.#breed}`;
    // Cannot access this.#name directly from subclass
  }
}
```

### Advanced Examples

```javascript
// Example 1: Private field for internal state machine
class Connection {
  #state = "disconnected";
  #handlers = new Map();
  
  on(event, handler) {
    this.#handlers.set(event, handler);
    return this;
  }
  
  connect() {
    this.#transition("connecting");
    setTimeout(() => {
      this.#transition("connected");
    }, 100);
  }
  
  disconnect() {
    this.#transition("disconnected");
  }
  
  #transition(newState) {
    const oldState = this.#state;
    this.#state = newState;
    const handler = this.#handlers.get(newState);
    handler?.(oldState, newState);
  }
}

// Example 2: Private fields for immutable internal state
class SafeArray {
  #items;
  
  constructor(items = []) {
    this.#items = [...items]; // Copy to prevent external mutation
  }
  
  get length() {
    return this.#items.length;
  }
  
  at(index) {
    return this.#items[index];
  }
  
  toArray() {
    return [...this.#items]; // Return a copy
  }
}

const safe = new SafeArray([1, 2, 3]);
console.log(safe.length); // 3
console.log(safe.toArray()); // [1, 2, 3]

// Example 3: Private field with WeakMap polyfill pattern
// Before private fields, this was the pattern using WeakMap
const _private = new WeakMap();
class Widget {
  constructor() {
    _private.set(this, { state: "initial" });
  }
  getState() {
    return _private.get(this).state;
  }
  setState(state) {
    _private.get(this).state = state;
  }
}
```

### Real-World Use Cases

- **Framework internals** — React's fiber nodes use private-like patterns.
- **Service classes** — API keys, tokens stored as private fields.
- **State management** — Internal store state with controlled access.
- **Security-sensitive data** — Passwords, tokens, encryption keys.
- **Library development** — Exposing clean public API while hiding internals.

### Common Mistakes

- **Forgetting to declare private fields** — Must be declared in the class body.
- **Assuming subclasses can access parent private fields** — They cannot.
- **Using `#` on the prototype** — Private fields are instance-specific.
- **Checking private fields with `in`** — `#field in obj` works but only inside the class.
- **Trying to access private fields after object transformation** — `Object.assign` or spread won't copy private fields.

### Best Practices

- Use private fields for internal implementation details.
- Use `#` instead of `_` for true privacy in modern code.
- Declare all private fields at the top of the class body.
- Use private methods for internal helper functions.
- Remember that private fields are not cloned by spread or `Object.assign`.

### Performance Considerations

- Private fields have similar performance to regular fields (slightly slower due to brand checks).
- V8 optimizes private field access with inline caching.
- Private methods are also optimized.
- The brand check adds a tiny overhead per access (nanoseconds).
- Private fields cannot be accessed via `Object.getOwnPropertyNames()` or similar.

### Interview Questions

1. **How are private fields different from `_` prefixed fields?**
   Private fields (`#`) are truly private and enforced by the engine. `_` is just a convention with no enforcement.

2. **Can subclasses access parent private fields?**
   No. Private fields are only accessible within the class that declares them.

3. **Are private fields cloned by `Object.assign` or spread?**
   No. They are not enumerable or accessible via property access, so they are not copied.

4. **Can you use `this.#field in obj` to check for a private field?**
   Yes, but only inside the class that declares the field.

### Coding Challenges

1. **Create a class with a private `#data` array and public methods for CRUD.**
2. **Implement a bank account class with a private `#balance` field.**
3. **Build a reactive state class with private fields for subscribers.**
4. **Create a class that uses private methods for internal validation.**
5. **Implement a simple Linked List with private node internals.**

### Related Topics

- Class fields
- Private methods
- Encapsulation
- WeakMap for privacy
- TypeScript `private` keyword
