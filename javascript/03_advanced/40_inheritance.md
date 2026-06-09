# Inheritance

## Introduction

Inheritance in JavaScript allows a class to derive properties and methods from another class, promoting code reuse and establishing hierarchical relationships. JavaScript uses prototypal inheritance under the hood, but the ES6 class syntax provides a clean extends keyword for setting up inheritance chains. Understanding inheritance is crucial for designing maintainable, DRY object-oriented code.

---

## extends keyword

### What It Is

The extends keyword is used in class declarations or expressions to create a child class that inherits from a parent class. It establishes the prototype chain between the child and parent, making parent methods accessible on child instances.

### Why It Is Important

The extends keyword enables code reuse, hierarchical relationships, polymorphism, cleaner inheritance setup compared to manual prototype manipulation, and integration with super() for parent constructor and method calls.

### How It Works Internally

When class Child extends Parent is evaluated:
1. Child.prototype is set to a new object with its [[Prototype]] pointing to Parent.prototype.
2. Child.__proto__ (the constructor's [[Prototype]]) is set to Parent — this enables static method inheritance.
3. The prototype chain is established: instance → Child.prototype → Parent.prototype → Object.prototype → null.

### Syntax

`javascript
class Child extends Parent {}
class MyArray extends Array {}
class MyError extends Error {}
function mixin(Base) {
  return class extends Base { extraMethod() {} };
}
class Final extends mixin(Parent) {}
`

### Beginner Examples

`javascript
class Vehicle {
  constructor(type) { this.type = type; }
  move() { return ${this.type} is moving; }
}
class Car extends Vehicle {
  constructor(brand) { super("car"); this.brand = brand; }
  honk() { return ${this.brand} honks; }
}
const myCar = new Car("Toyota");
console.log(myCar.move()); // "car is moving"
console.log(myCar instanceof Vehicle); // true

class ValidationError extends Error {
  constructor(message, field) { super(message); this.name = "ValidationError"; this.field = field; }
}
try { throw new ValidationError("Invalid email", "email"); }
catch (err) { console.log(err.field); } // "email"
`

### Intermediate Examples

`javascript
class Person {
  constructor(name, age) { if (age < 0) throw new Error("Invalid age"); this.name = name; this.age = age; }
  describe() { return ${this.name} is  years old; }
}
class Employee extends Person {
  constructor(name, age, title) { super(name, age); if (!title) throw new Error("Title required"); this.title = title; }
  work() { return ${this.name} works as ; }
}

// Computed extends
function withTimestamp(Class) {
  return class extends Class {
    constructor(...args) { super(...args); this.createdAt = new Date(); }
  };
}
const TimestampedPerson = withTimestamp(Person);
const tsPerson = new TimestampedPerson("Bob", 25);
console.log(tsPerson.createdAt); // Date object
`

### Advanced Examples

`javascript
// Extending built-in Array
class ObservableArray extends Array {
  constructor(...args) { super(...args); this.listeners = new Set(); }
  onChange(cb) { this.listeners.add(cb); return () => this.listeners.delete(cb); }
  notify() { this.listeners.forEach(fn => fn(this)); }
  push(...items) { const r = super.push(...items); this.notify(); return r; }
  pop() { const r = super.pop(); this.notify(); return r; }
}
const arr = new ObservableArray(1,2,3);
const unsub = arr.onChange(a => console.log("Changed:", a));
arr.push(4); // "Changed: [1,2,3,4]"
unsub();

// Conditional base class
class Bird extends (Math.random() > 0.5 ? Animal : Object) { fly() { return "Flying"; } }
`

### Real-World Use Cases

React class components (class MyComponent extends React.Component), custom errors, ORM models, custom elements (class MyButton extends HTMLButtonElement), stream types.

### Common Mistakes

Forgetting super() in derived class constructor (ReferenceError), calling super() after accessing 	his, creating deep inheritance hierarchies, assuming private fields are inherited.

### Best Practices

Keep hierarchies shallow (2-3 levels), prefer composition over deep inheritance, always call super() before 	his, use instanceof for checking relationships.

### Performance Considerations

Each level adds a prototype chain lookup step. V8 optimizes shallow chains well. Calling super() is fast as a normal constructor call.

### Interview Questions

1. What does extends do internally? — Sets up the prototype chain.
2. Can you extend built-in classes? — Yes.
3. What happens if you don't call super()? — ReferenceError.
4. Can a class extend a regular function? — Yes, if it's a constructor.

### Coding Challenges

1. Create a class hierarchy for a zoo (Animal → Mammal → Dog).
2. Extend Array to create a Queue class.
3. Build a TimedCache that extends Map with TTL.

### Related Topics

super(), method overriding, prototype chain, mixins, composition vs inheritance

---

## super() function

### What It Is

The super() function calls the parent class constructor. It can also be used as super.methodName() to call parent methods. super() must be called before accessing 	his in a derived class constructor.

### Why It Is Important

super() is essential for properly initializing the parent part of a derived instance, accessing overridden parent methods, avoiding code duplication, and passing constructor arguments to the parent.

### How It Works Internally

When super() is called, the engine creates the 	his binding from the parent constructor, runs the parent constructor with 	his pointing to the new instance, then allows the child constructor to access 	his. super.method() resolves from the parent prototype.

### Syntax

`javascript
class Child extends Parent {
  constructor(...args) { super(...args); }
  method() { super.parentMethod(); }
}
`

### Beginner Examples

`javascript
class Person {
  constructor(firstName, lastName) { this.firstName = firstName; this.lastName = lastName; }
}
class Employee extends Person {
  constructor(firstName, lastName, title) { super(firstName, lastName); this.title = title; }
}
const emp = new Employee("John", "Doe", "Developer");
console.log(emp.firstName); // "John"

class Vehicle {
  start() { return "Engine started"; }
}
class Car extends Vehicle {
  start() { return ${super.start()}. Ready to drive!; }
}
const car = new Car();
console.log(car.start()); // "Engine started. Ready to drive!"
`

### Intermediate Examples

`javascript
// Conditional super call
class Item { constructor(id) { this.id = id; } }
class SpecialItem extends Item {
  constructor(data) {
    if (typeof data === "string") super(data);
    else if (typeof data === "number") super(ITEM-);
    else super("UNKNOWN");
    this.created = new Date();
  }
}

// super with static methods
class Base { static identify() { return "Base"; } }
class Derived extends Base { static identify() { return Derived extends ; } }
console.log(Derived.identify()); // "Derived extends Base"
`

### Advanced Examples

`javascript
// super in mixins
const Timestampable = (Base) => class extends Base {
  constructor(...args) { super(...args); this.createdAt = new Date(); }
  get age() { return Date.now() - this.createdAt.getTime(); }
};
class User { constructor(name) { this.name = name; } }
class TimestampedUser extends Timestampable(User) {
  constructor(name, role) { super(name); this.role = role; }
}

// Multiple level super calls
class A { method() { return "A"; } }
class B extends A { method() { return super.method() + "->B"; } }
class C extends B { method() { return super.method() + "->C"; } }
const c = new C();
console.log(c.method()); // "A->B->C"
`

### Real-World Use Cases

React class components (super(props)), Mongoose models, error subclassing, custom elements, stream subclasses.

### Common Mistakes

Forgetting super() (ReferenceError), using 	his before super(), calling super() outside a derived class constructor, confusing super as an object.

### Best Practices

Always call super() as the first statement in derived constructors. Pass all required arguments. Use super.method() when extending parent methods.

### Performance Considerations

super() calls have minimal overhead. super.method() is slightly slower than 	his.method() due to prototype traversal from the parent.

### Interview Questions

1. What does super() do? — Calls parent constructor and creates 	his binding.
2. Difference between super() and super.method()? — Constructor vs method call.
3. Can you call super() outside a derived constructor? — No.

### Coding Challenges

1. Create a three-level class hierarchy demonstrating super.
2. Build a mixin that requires super() in correct order.
3. Create a class using super to access parent getters/setters.

### Related Topics

extends, method overriding, constructor, mixins

---

## Method overriding

### What It Is

Method overriding occurs when a child class defines a method with the same name as a method in its parent class. The child's version executes instead of the parent's when called on a child instance.

### Why It Is Important

Method overriding enables polymorphism, behavior customization, the template method pattern, code reuse via super.method(), and abstract-like patterns.

### How It Works Internally

Due to the prototype chain, the child method is found first during property resolution and shadows the parent method.

### Syntax

`javascript
class Parent { method() { return "parent"; } }
class Child extends Parent {
  method() {
    const parentResult = super.method();
    return ${parentResult} + child;
  }
}
`

### Beginner Examples

`javascript
class Shape { area() { return 0; } }
class Circle extends Shape {
  constructor(r) { super(); this.r = r; }
  area() { return Math.PI * this.r ** 2; }
}
console.log(new Circle(5).area()); // 78.54

class Logger { log(m) { console.log([] ); } }
class PrefixedLogger extends Logger {
  constructor(p) { super(); this.prefix = p; }
  log(m) { super.log([] ); }
}
`

### Intermediate Examples

`javascript
// Template Method pattern
class DataExporter {
  export() { const data = this.fetchData(); const processed = this.processData(data); const formatted = this.formatData(processed); this.saveData(formatted); }
  fetchData() { throw new Error("Must implement"); }
  processData(d) { return d; }
  formatData(d) { return JSON.stringify(d); }
  saveData(f) { throw new Error("Must implement"); }
}
class JSONFileExporter extends DataExporter {
  fetchData() { return { users: [{id:1, name:"Alice"}] }; }
  saveData(f) { fs.writeFileSync("export.json", f); }
}

// Conditional super call
class Validator {
  validate(v) { if (v == null) throw new Error("Cannot be null"); return true; }
}
class NumberValidator extends Validator {
  validate(v) { super.validate(v); if (typeof v !== "number") throw new Error("Must be number"); if (v < 0) throw new Error("Must be non-negative"); return true; }
}
`

### Real-World Use Cases

React lifecycle methods, error subclassing, test mocking, plugin systems, UI components.

### Common Mistakes

Forgetting super.method() when extending behavior, overriding and changing contract, creating overrides that break method chaining.

### Best Practices

Call super.method() when extending parent behavior. Keep overridden methods consistent with parent contract. Use template method pattern for skeletons.

### Performance Considerations

Overridden methods resolve via prototype chain. Calling super.method() adds one extra lookup. V8 optimizes with polymorphic inline caching.

### Interview Questions

1. What is method overriding? — Child redefines a parent method.
2. How to call overridden parent method? — super.methodName().
3. Can static methods be overridden? — Yes.

### Coding Challenges

1. Override rea() in Circle, Rectangle, Triangle.
2. Build a caching layer overriding get()/set() with TTL.
3. Implement a retry mechanism by overriding an async method.

### Related Topics

extends, super(), polymorphism, template method pattern

---

## Mixins for multiple inheritance

### What It Is

Mixins combine behaviors from multiple sources into a single class. Since JavaScript doesn't support multiple inheritance, mixins provide a way to share functionality across unrelated classes by composing behaviors.

### Why It Is Important

Mixins enable multiple inheritance, horizontal reuse across unrelated hierarchies, composition over inheritance, and flexible behavioral composition.

### How It Works Internally

Mixins copy properties from source objects to the target class's prototype, or use function-based approaches that return extended class expressions.

### Syntax

`javascript
const mixinA = { methodA() {} };
const mixinB = { methodB() {} };
class MyClass {}
Object.assign(MyClass.prototype, mixinA, mixinB);

function withFeature(Base) {
  return class extends Base { featureMethod() {} };
}
class Final extends withFeature(withOtherFeature(Base)) {}
`

### Beginner Examples

`javascript
const Greetable = { greet() { return Hello, !; } };
const Nameable = { setName(n) { this.name = n; return this; } };
class User { constructor(n) { this.name = n; } }
Object.assign(User.prototype, Greetable, Nameable);
const user = new User("Alice");
console.log(user.greet()); // "Hello, Alice!"

function WithTimestamp(Base) {
  return class extends Base {
    constructor(...args) { super(...args); this.createdAt = new Date(); }
    get age() { return Date.now() - this.createdAt.getTime(); }
  };
}
class Message { constructor(content) { this.content = content; } }
const TimestampedMessage = WithTimestamp(Message);
`

### Intermediate Examples

`javascript
function WithLogging(Base) {
  return class extends Base {
    log(op) { console.log([] ); return this; }
  };
}
function WithSerialization(Base) {
  return class extends Base {
    toJSON() { return JSON.stringify(this); }
    static fromJSON(j) { return new this(JSON.parse(j)); }
  };
}
class DataModel { constructor(d) { Object.assign(this, d); } }
class EnhancedModel extends WithSerialization(WithLogging(DataModel)) {
  constructor(d) { super(d); }
}
const model = new EnhancedModel({id:1, name:"test"});
model.log("Created");
console.log(model.toJSON());

// Mixin composition utility
function compose(...mixins) {
  return (Base) => mixins.reduce((acc, m) => m(acc), Base);
}
const ComposedModel = compose(WithLogging, WithSerialization)(DataModel);
`

### Real-World Use Cases

Vue mixins, React HOCs, Angular mixins, Lodash mixins, Express middleware.

### Common Mistakes

Name collisions between mixins, mixin state conflicts, constructor conflicts, over-relying on mixins.

### Best Practices

Use descriptive names (e.g., WithLogging), keep mixins focused on single responsibility, avoid mixing state, document conflicts.

### Performance Considerations

Object.assign on prototype is a one-time cost. Function-based mixins create intermediate class objects (negligible). Method resolution follows the prototype chain.

### Interview Questions

1. What are mixins? — A pattern for combining behaviors from multiple sources.
2. How do you implement mixins? — Object.assign on prototype or function-based class expressions.
3. Difference between mixins and inheritance? — Mixins compose, inheritance derives.

### Coding Challenges

1. Create WithLogging, WithValidation mixins and compose them.
2. Build a mixin that adds event-emitter behavior.
3. Implement a mixin factory with configuration options.

### Related Topics

Composition vs inheritance, extends, higher-order components, traits
