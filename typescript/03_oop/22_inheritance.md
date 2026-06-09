# Inheritance - extends keyword, super(), method overriding, abstract classes

## Introduction
Inheritance is a fundamental object-oriented principle that allows a derived class to reuse, extend, and specialize the behavior of a base class. TypeScript implements classical single inheritance through the `extends` keyword, supporting method overriding, constructor chaining via `super()`, and abstract class patterns. This mechanism enables code reuse, hierarchical type relationships, and polymorphic behavior.

## extends keyword

### What It Is
The `extends` keyword establishes an inheritance relationship where a child class derives from a parent class, inheriting its members and behavior.

### Why It Is Important
`extends` enables code reuse by letting child classes leverage existing implementation. It models real-world "is-a" relationships (e.g., a `Dog` is an `Animal`).

### How It Works Internally
TypeScript compiles the `extends` clause into JavaScript's `class extends` or a manual prototype chain for older targets. The child's `prototype.__proto__` is set to the parent's prototype.

### Syntax
```typescript
class Child extends Parent {
  // additional or overridden members
}
```

### Beginner Examples
```typescript
class Animal {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
  move(distance: number): void {
    console.log(`${this.name} moved ${distance}m.`);
  }
}
class Dog extends Animal {
  bark(): void {
    console.log("Woof!");
  }
}
const dog = new Dog("Rex");
dog.bark();   // "Woof!"
dog.move(10); // "Rex moved 10m."
```

### Intermediate Examples
```typescript
class Shape {
  constructor(public color: string) {}
  describe(): string {
    return `A ${this.color} shape`;
  }
}
class Circle extends Shape {
  constructor(color: string, public radius: number) {
    super(color);
  }
  area(): number {
    return Math.PI * this.radius ** 2;
  }
}
```

### Advanced Examples
```typescript
class PluginBase {
  name: string;
  version: string;
  constructor(name: string, version: string) {
    this.name = name;
    this.version = version;
  }
  initialize(): void {
    console.log(`Plugin ${this.name} v${this.version} initialized`);
  }
}
class AuthPlugin extends PluginBase {
  constructor() {
    super("auth", "1.0.0");
  }
}
```

### Real-World Use Cases
- Extending framework base classes (e.g., Angular `Component`, React `Component`)
- Implementing domain entity hierarchies (`Person` → `Employee` → `Manager`)
- Creating specialized exception types (`Error` → `NetworkError` → `TimeoutError`)

### Common Mistakes
- Using `extends` when composition would be more appropriate
- Extending concrete classes in deep hierarchies, leading to fragile code
- Forgetting to call `super()` in the derived class constructor

### Best Practices
- Prefer shallow hierarchies (1–2 levels deep)
- Use `extends` for true "is-a" relationships only
- Consider composition over inheritance for code reuse

### Performance Considerations
Property lookup traverses the prototype chain; deep inheritance trees can slow property access. Method calls on the prototype are slightly slower than own methods.

### Interview Questions
- Q: Can a class extend multiple classes in TypeScript?
  A: No, TypeScript supports single inheritance. Use interfaces or mixins for multiple inheritance simulation.

### Coding Challenges
- Build an `Employee` class extending `Person` with an additional `employeeId` and `department`.
- Create a class hierarchy: `Media` → `Book` | `Video` | `Podcast`.

### Related Topics
- `super()` calls
- Method overriding
- Abstract classes

---

## super() calls

### What It Is
`super()` in the constructor invokes the parent class constructor. `super.methodName()` calls a parent class method from the child class.

### Why It Is Important
The child constructor must call `super()` before accessing `this` to properly initialize the parent's state. Calling parent methods enables extending behavior without replacing it.

### How It Works Internally
`super()` compiles to `_this = _super.call(this)` or a `Reflect.construct` call in ES6 output. `super.method()` compiles to `_super.prototype.method.call(this)`.

### Syntax
```typescript
class Child extends Parent {
  constructor(...args) {
    super(...args); // call parent constructor
    // child initialization
  }
  method(): void {
    super.method(); // call parent method
    // child extension
  }
}
```

### Beginner Examples
```typescript
class Vehicle {
  constructor(public make: string, public model: string) {}
  getInfo(): string {
    return `${this.make} ${this.model}`;
  }
}
class Car extends Vehicle {
  constructor(make: string, model: string, public doors: number) {
    super(make, model); // required
  }
  getInfo(): string {
    return `${super.getInfo()} with ${this.doors} doors`;
  }
}
```

### Intermediate Examples
```typescript
class BaseService {
  constructor(protected baseURL: string) {}
  protected log(method: string, path: string): void {
    console.log(`[${method}] ${this.baseURL}${path}`);
  }
}
class UserService extends BaseService {
  constructor() {
    super("https://api.example.com");
  }
  async getById(id: string): Promise<User> {
    super.log("GET", `/users/${id}`);
    const res = await fetch(`${this.baseURL}/users/${id}`);
    return res.json();
  }
}
```

### Advanced Examples
```typescript
abstract class Validator {
  abstract validate(value: unknown): boolean;
  sanitize(value: string): string {
    return value.trim();
  }
}
class EmailValidator extends Validator {
  validate(value: unknown): boolean {
    const str = super.sanitize(String(value));
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(str);
  }
}
```

### Real-World Use Cases
- Extending form validation: call `super.validate()` then add specific rules
- Extending UI components: call `super.render()` then add child elements
- Extending error classes: call `super(message)` to set `this.message`

### Common Mistakes
- Forgetting `super()` in the constructor leads to runtime `ReferenceError`
- Passing wrong arguments to `super()`
- Accessing `this` before `super()` in derived constructor

### Best Practices
- Always call `super()` before any `this` access
- Match `super()` argument types to the parent constructor parameter types
- Use `super.methodName()` when extending inherited methods rather than fully overriding

### Performance Considerations
`super()` calls are direct function calls with negligible overhead. `super.method()` requires prototype chain traversal but is generally fast.

### Interview Questions
- Q: What happens if you omit `super()` in a derived class?
  A: TypeScript emits a compile error, and JavaScript throws a `ReferenceError` at runtime.

### Coding Challenges
- Create a `BaseEntity` with `id` and `createdAt`, then extend it with `SoftDeletableEntity` that adds `deletedAt` and calls `super()`.
- Build a `Logger` base class and a `TimedLogger` that prepends timestamps using `super.log()`.

### Related Topics
- extends keyword
- Constructor and fields
- Abstract classes

---

## Method overriding

### What It Is
Method overriding occurs when a child class defines a method with the same name and signature as a parent method, replacing or extending the parent's behavior.

### Why It Is Important
Overriding allows subclasses to specialize behavior for their own context while maintaining a consistent interface. It is the mechanism behind polymorphic method dispatch.

### How It Works Internally
JavaScript's prototype chain determines which method executes based on the object's actual type. The child's method is found first on its prototype, shadowing the parent's.

### Syntax
```typescript
class Parent {
  method(): void { /* parent logic */ }
}
class Child extends Parent {
  method(): void {
    // override — optionally call super.method()
  }
}
```

### Beginner Examples
```typescript
class Bird {
  sound(): string {
    return "Chirp";
  }
}
class Duck extends Bird {
  sound(): string {
    return "Quack";
  }
}
const bird: Bird = new Duck();
console.log(bird.sound()); // "Quack" — polymorphic
```

### Intermediate Examples
```typescript
class DataParser {
  parse(raw: string): unknown {
    return JSON.parse(raw);
  }
}
class CustomParser extends DataParser {
  parse(raw: string): Record<string, unknown> {
    const parsed = super.parse(raw) as Record<string, unknown>;
    if (typeof parsed !== "object" || parsed === null) {
      throw new Error("Expected an object");
    }
    return parsed;
  }
}
```

### Advanced Examples
```typescript
class Renderer {
  render(vnode: VNode): HTMLElement {
    const el = document.createElement(vnode.tag);
    for (const [key, val] of Object.entries(vnode.props ?? {})) {
      el.setAttribute(key, String(val));
    }
    for (const child of vnode.children ?? []) {
      el.appendChild(
        typeof child === "string"
          ? document.createTextNode(child)
          : this.render(child)
      );
    }
    return el;
  }
}
class SVGTRenderer extends Renderer {
  override render(vnode: VNode): SVGElement {
    const el = document.createElementNS(
      "http://www.w3.org/2000/svg",
      vnode.tag
    );
    for (const [key, val] of Object.entries(vnode.props ?? {})) {
      el.setAttributeNS(null, key, String(val));
    }
    for (const child of vnode.children ?? []) {
      el.appendChild(
        typeof child === "string"
          ? document.createTextNode(child)
          : this.render(child)
      );
    }
    return el;
  }
}
```

### Real-World Use Cases
- Overriding lifecycle hooks in UI frameworks (e.g., `ngOnInit`, `componentDidMount`)
- Custom serialization in data classes
- Specializing error handling in derived service classes

### Common Mistakes
- Changing the return type in a way that violates the Liskov Substitution Principle
- Forgetting to call `super.method()` when extending behavior
- Not using the `override` keyword (TypeScript 4.3+) to catch signature mismatches

### Best Practices
- Use the `override` keyword to catch subtle bugs when parent signatures change
- Favor calling `super.method()` to extend rather than replace when appropriate
- Keep overridden methods faithful to the parent's contract (LSP)

### Performance Considerations
Method dispatch is dynamic but highly optimized by V8 and other engines. Override depth has negligible impact on call speed.

### Interview Questions
- Q: What is the difference between method overriding and method overloading?
  A: Overriding replaces a parent method; overloading provides multiple signatures for the same function.

### Coding Challenges
- Create a `Shape` class with `area(): number`, then override it in `Circle`, `Rectangle`, and `Triangle`.
- Build a `Serializer` base class with `serialize` method, then override in `JSONSerializer` and `XMLSerializer`.

### Related Topics
- extends keyword
- Polymorphism
- `super()` calls

---

## Member visibility in inheritance

### What It Is
Access modifiers (`public`, `protected`, `private`) control which inherited members are visible and accessible in derived classes and external code.

### Why It Is Important
Visibility modifiers enforce encapsulation boundaries even within inheritance hierarchies. `protected` members are accessible in derived classes but not publicly, providing a controlled API for subclass authors.

### How It Works Internally
TypeScript's access modifiers are compile-time only. The emitted JavaScript does not enforce visibility, but the compiler prevents access violations.

### Syntax
```typescript
class Base {
  public a = 1;
  protected b = 2;
  private c = 3;
}
class Derived extends Base {
  method(): void {
    console.log(this.a); // OK
    console.log(this.b); // OK
    console.log(this.c); // Error: private
  }
}
```

### Beginner Examples
```typescript
class Account {
  public owner: string;
  protected balance: number;
  private accountNumber: string;
  constructor(owner: string, balance: number, accountNumber: string) {
    this.owner = owner;
    this.balance = balance;
    this.accountNumber = accountNumber;
  }
}
class SavingsAccount extends Account {
  constructor(owner: string, balance: number) {
    super(owner, balance, "SAV-" + Date.now());
  }
  getBalance(): number {
    return this.balance; // OK: protected
  }
}
```

### Intermediate Examples
```typescript
class BaseRepository {
  protected db: Database;
  private connectionString: string;
  constructor(connectionString: string) {
    this.connectionString = connectionString;
    this.db = new Database(connectionString);
  }
  protected async query(sql: string): Promise<unknown[]> {
    return this.db.execute(sql);
  }
}
class UserRepository extends BaseRepository {
  constructor() {
    super("postgres://localhost/mydb");
  }
  async findAll(): Promise<User[]> {
    return this.query("SELECT * FROM users") as Promise<User[]>;
  }
}
```

### Advanced Examples
```typescript
class AbstractCache {
  private store = new Map<string, unknown>();
  protected abstract getKey(...args: unknown[]): string;
  public get<T>(...args: unknown[]): T | undefined {
    return this.store.get(this.getKey(...args)) as T | undefined;
  }
  protected set(key: string, value: unknown): void {
    this.store.set(key, value);
  }
}
class UserCache extends AbstractCache {
  protected getKey(userId: string): string {
    return `user:${userId}`;
  }
  public setUser(userId: string, data: User): void {
    // Access to protected set is OK
    this.set(this.getKey(userId), data);
  }
}
```

### Real-World Use Cases
- Template method pattern where base class exposes protected steps
- Framework base classes with protected lifecycle hooks
- Repository pattern with protected query methods

### Common Mistakes
- Overexposing internal state with `protected` when it should be truly encapsulated
- Assuming `private` members are truly private at runtime
- Changing visibility from `protected` to `public` in derived classes accidentally

### Best Practices
- Use `protected` sparingly — prefer `private` by default
- Document the contract that derived classes must follow
- Prefix `protected` methods with descriptive names (e.g., `doValidate`, `onChange`)

### Performance Considerations
Visibility modifiers have zero runtime impact — they are compile-time constructs.

### Interview Questions
- Q: Can a derived class override the visibility of an inherited member?
  A: A derived class cannot narrow visibility (e.g., make `public` → `private`), but can widen it (e.g., `protected` → `public`).

### Coding Challenges
- Create a `Document` base class with `public title`, `protected content`, and `private metadata`. Extend it with `PDFDocument` and `WordDocument`.
- Build an `AbstractLogger` with `protected formatMessage` and `private write` methods.

### Related Topics
- Access modifiers
- Encapsulation
- Abstract classes
