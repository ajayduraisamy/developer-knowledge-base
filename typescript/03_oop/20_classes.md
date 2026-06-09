# Classes - class syntax, constructor, fields, methods, parameter properties, implements

## Introduction
Classes in TypeScript build upon JavaScript's ES2015+ class syntax by adding type annotations, visibility modifiers, parameter properties, and interface implementation contracts. They provide a blueprint for creating objects with shared structure and behavior, and they sit at the heart of object-oriented programming in the language. TypeScript compiles classes down to JavaScript, offering a superset of features that make large-scale application development more robust and self-documenting.

## class declaration

### What It Is
A class declaration defines a new class using the `class` keyword followed by a name and an optional body enclosed in curly braces. The body contains properties (fields), constructors, and methods.

### Why It Is Important
The class declaration is the entry point for creating object blueprints. Without it, developers would have to rely on plain objects or factory functions, which lack the structural discipline, inheritance chain, and `instanceof` checks that classes provide.

### How It Works Internally
TypeScript compiles a class declaration into a JavaScript constructor function with a prototype. The declared name becomes the constructor identifier, and method definitions become prototype properties. TypeScript also emits metadata for decorators and tracks the class's type shape for the type system.

### Syntax
```typescript
class ClassName {
  // fields
  // constructor
  // methods
}
```

### Beginner Examples
```typescript
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}
const p = new Person("Alice");
console.log(p.name); // "Alice"
```

### Intermediate Examples
```typescript
class Counter {
  count: number;
  constructor(initial = 0) {
    this.count = initial;
  }
  increment(): number {
    this.count += 1;
    return this.count;
  }
}
const c = new Counter(5);
console.log(c.increment()); // 6
```

### Advanced Examples
```typescript
class Registry<T> {
  private items: Map<string, T> = new Map();
  register(key: string, value: T): void {
    this.items.set(key, value);
  }
  get(key: string): T | undefined {
    return this.items.get(key);
  }
}
const reg = new Registry<number>();
reg.register("x", 42);
```

### Real-World Use Cases
- Domain models in DDD (e.g., `Order`, `User`, `Product`)
- Service classes encapsulating business logic
- Data transfer objects (DTOs) with validation methods
- UI component classes in Angular or Aurelia

### Common Mistakes
- Forgetting to declare field types or initialize fields
- Using `this` in static methods expecting instance state
- Mutating class fields without declaring them

### Best Practices
- Declare all fields explicitly with their types
- Use concise constructor parameter properties when appropriate
- Prefer interfaces for public contracts, classes for implementation

### Performance Considerations
Class instances have a small overhead compared to plain objects. Property access on the prototype chain may be marginally slower than own properties. In hot code paths, consider caching method references.

### Interview Questions
- Q: What is the difference between a class and a constructor function?
  A: A class is syntactic sugar over constructor functions with stricter semantics (e.g., methods are non-enumerable, `use strict` is implied).

### Coding Challenges
- Implement a `Queue` class with `enqueue`, `dequeue`, and `peek` using generic types.
- Create a `Vector` class with `add`, `subtract`, and `dotProduct` methods.

### Related Topics
- Constructor and fields
- Methods
- Parameter properties
- `implements` clause

---

## Constructor and fields

### What It Is
The constructor is a special method named `constructor` that runs when a new instance is created. Fields are properties declared on the class body, optionally with initializers.

### Why It Is Important
The constructor initializes instance state, validates arguments, and sets up the object's invariant. Fields define the shape of the data each instance holds.

### How It Works Internally
TypeScript transpiles field declarations into assignments in the constructor. If a field has an initializer, it becomes an assignment after `super()` calls. The constructor itself compiles to the function body of the generated constructor function.

### Syntax
```typescript
class Example {
  field1: string;
  field2: number = 0;
  constructor(param: string) {
    this.field1 = param;
  }
}
```

### Beginner Examples
```typescript
class Book {
  title: string;
  year: number;
  constructor(title: string, year: number) {
    this.title = title;
    this.year = year;
  }
}
```

### Intermediate Examples
```typescript
class Temperature {
  celsius: number;
  constructor(celsius: number) {
    if (celsius < -273.15) throw new Error("Below absolute zero");
    this.celsius = celsius;
  }
  get fahrenheit(): number {
    return this.celsius * 9 / 5 + 32;
  }
}
```

### Advanced Examples
```typescript
class Config {
  readonly env: string;
  version: string;
  debug: boolean = false;
  constructor(env: string, version: string) {
    this.env = env;
    this.version = version;
    if (env === "development") this.debug = true;
  }
}
```

### Real-World Use Cases
- Initializing database connections in service classes
- Setting up dependency injections through constructor parameters
- Validating constructor arguments before assignment

### Common Mistakes
- Defining a field and a constructor parameter with the same name without using parameter properties
- Calling `this` before `super()` in a derived class
- Forgetting the `readonly` modifier for immutable fields

### Best Practices
- Initialize fields with sensible defaults when possible
- Keep constructors simple — delegate complex setup to factory methods
- Use `readonly` for fields that should not change after construction

### Performance Considerations
Field initializers run during construction; avoid expensive operations in hot-path constructors. Consider lazy initialization for computationally heavy fields.

### Interview Questions
- Q: How does TypeScript handle field declarations with initializers differently from those without?
  A: Initializers become assignments in the constructor; non-initialized fields rely on explicit assignment during construction.

### Coding Challenges
- Write a `Matrix` class with a constructor accepting rows, columns, and an optional default value.
- Create a `Range` class with `start`, `end`, and a `step` field with default 1.

### Related Topics
- class declaration
- Methods
- Parameter properties
- `readonly` modifier

---

## Methods

### What It Is
Methods are functions defined on a class body. They operate on instance data via `this` and can be inherited, overridden, and called on class instances.

### Why It Is Important
Methods encapsulate behavior alongside data, providing a clean API for interacting with object state.

### How It Works Internally
Methods are assigned to the class's prototype, shared across all instances. TypeScript ensures type-safe `this` context and supports method overload signatures.

### Syntax
```typescript
class Example {
  methodName(param: Type): ReturnType {
    // body
  }
}
```

### Beginner Examples
```typescript
class Greeter {
  greet(name: string): string {
    return `Hello, ${name}!`;
  }
}
const g = new Greeter();
console.log(g.greet("Bob"));
```

### Intermediate Examples
```typescript
class Stack<T> {
  private items: T[] = [];
  push(item: T): void {
    this.items.push(item);
  }
  pop(): T | undefined {
    return this.items.pop();
  }
  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }
}
```

### Advanced Examples
```typescript
class Calculator {
  add(a: number, b: number): number;
  add(a: string, b: string): string;
  add(a: any, b: any): any {
    return a + b;
  }
}
```

### Real-World Use Cases
- Service methods making API calls (e.g., `UserService.getById(id)`)
- Repository methods for data access (`findAll`, `save`, `delete`)
- Utility methods on value objects (`toJSON`, `equals`)

### Common Mistakes
- Using fat-arrow functions for methods when prototype sharing is needed
- Mismatching return types due to missing annotations
- Calling methods without proper `this` binding in callbacks

### Best Practices
- Prefer regular method syntax (not arrow functions) for prototype methods
- Use method overloading for different call signatures
- Always annotate method return types for public APIs

### Performance Considerations
Arrow function methods create a new function per instance, costing memory. Regular prototype methods are shared. Use arrow functions only when you need lexical `this` binding.

### Interview Questions
- Q: What is the difference between a method and a function property with an arrow function?
  A: Methods live on the prototype and are shared; arrow function properties are own properties, each instance has its own copy.

### Coding Challenges
- Build a `DoublyLinkedList` class with `insert`, `remove`, and `traverse` methods.
- Create an `EventEmitter` class with `on`, `emit`, and `off` methods.

### Related Topics
- class declaration
- Inheritance (method overriding)
- `this` typing

---

## Parameter properties (public constructor shorthand)

### What It Is
Parameter properties allow declaring and initializing class fields directly in the constructor signature by prefixing a parameter with a visibility modifier (`public`, `private`, `protected`, `readonly`).

### Why It Is Important
Parameter properties eliminate boilerplate code where you would otherwise declare a field and then assign the constructor parameter to it. This leads to more concise and readable class definitions.

### How It Works Internally
TypeScript desugars the parameter property into a field declaration and an assignment in the constructor body. The emitted JavaScript includes the field assignment just as if it were written manually.

### Syntax
```typescript
class Example {
  constructor(public name: string, private age: number) {}
}
```

### Beginner Examples
```typescript
class User {
  constructor(public username: string, public email: string) {}
}
const u = new User("alice", "alice@example.com");
console.log(u.username); // "alice"
```

### Intermediate Examples
```typescript
class Product {
  constructor(
    public readonly id: string,
    public name: string,
    private _price: number
  ) {}
  get price(): number {
    return this._price;
  }
  set price(value: number) {
    if (value < 0) throw new Error("Price cannot be negative");
    this._price = value;
  }
}
```

### Advanced Examples
```typescript
class Service {
  constructor(
    private readonly httpClient: HttpClient,
    @Inject() private readonly logger: Logger,
    public baseUrl: string
  ) {}
  async fetch(path: string): Promise<Response> {
    return this.httpClient.get(`${this.baseUrl}${path}`);
  }
}
```

### Real-World Use Cases
- Angular service classes injecting dependencies
- React state management stores
- DTO and entity classes in NestJS

### Common Mistakes
- Using parameter properties with primitive types when you need validation logic in the constructor
- Combining explicit field declarations with parameter properties for the same field
- Forgetting that `private` parameter properties create actual JavaScript properties (they are not truly private at runtime)

### Best Practices
- Use parameter properties for simple field assignments
- Avoid parameter properties when constructor logic beyond assignment is needed
- Combine `readonly` with parameter properties for immutable fields

### Performance Considerations
Parameter properties generate the same JavaScript as manual field declarations, so there is no runtime performance difference. They purely affect developer ergonomics.

### Interview Questions
- Q: Can you use parameter properties with accessor decorators?
  A: Yes, parameter properties work with parameter decorators, as seen in Angular/DI frameworks.

### Coding Challenges
- Refactor a verbose class with 5+ fields into a concise version using parameter properties.
- Create a `ConfigLoader` class using parameter properties for all injected dependencies.

### Related Topics
- Access modifiers
- Constructor and fields
- Dependency injection patterns

---

## implements clause

### What It Is
The `implements` clause allows a class to declare that it satisfies one or more interfaces. The TypeScript compiler enforces that the class provides the required members.

### Why It Is Important
Implements contracts enforce structural compliance at compile time, enabling polymorphic substitution and self-documenting code.

### How It Works Internally
The `implements` clause does not affect the emitted JavaScript; it is purely a compile-time check. The compiler verifies that the class has all the required members with compatible types.

### Syntax
```typescript
interface Drivable {
  start(): void;
  stop(): void;
}
class Car implements Drivable {
  start(): void { /* ... */ }
  stop(): void { /* ... */ }
}
```

### Beginner Examples
```typescript
interface Identifiable {
  id: string;
}
class Entity implements Identifiable {
  constructor(public id: string) {}
}
```

### Intermediate Examples
```typescript
interface Serializable {
  toJSON(): Record<string, unknown>;
}
interface Loggable {
  log(): void;
}
class User implements Serializable, Loggable {
  constructor(public name: string) {}
  toJSON(): Record<string, unknown> {
    return { name: this.name };
  }
  log(): void {
    console.log(this.toJSON());
  }
}
```

### Advanced Examples
```typescript
interface Repository<T> {
  find(id: string): Promise<T | null>;
  save(entity: T): Promise<void>;
  delete(id: string): Promise<void>;
}
class InMemoryUserRepository implements Repository<User> {
  private store = new Map<string, User>();
  async find(id: string): Promise<User | null> {
    return this.store.get(id) ?? null;
  }
  async save(entity: User): Promise<void> {
    this.store.set(entity.id, entity);
  }
  async delete(id: string): Promise<void> {
    this.store.delete(id);
  }
}
```

### Real-World Use Cases
- Implementing repository interfaces in data-access layers
- Strategy pattern with interchangeable algorithm classes
- Adapter pattern where classes implement target interfaces

### Common Mistakes
- Expecting `implements` to inherit implementation (it only checks shape)
- Creating classes that partially implement an interface and cause compilation errors
- Implementing an interface that changes frequently, requiring constant class updates

### Best Practices
- Favor composition over inheritance; use `implements` for contracts
- Keep interfaces small and focused (Interface Segregation Principle)
- Combine `implements` with `extends` for robust class hierarchies

### Performance Considerations
No runtime cost. The `implements` check is purely compile-time.

### Interview Questions
- Q: Can a class implement multiple interfaces?
  A: Yes, separated by commas: `class Foo implements A, B`.

### Coding Challenges
- Define a `SortStrategy` interface with a `sort` method, then implement `QuickSort` and `MergeSort` classes.
- Create a `Plugin` interface and implement two plugins for a hypothetical event system.

### Related Topics
- Interfaces
- Inheritance
- Type compatibility
