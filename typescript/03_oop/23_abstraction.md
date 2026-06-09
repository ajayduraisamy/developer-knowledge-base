# Abstraction - abstract classes, abstract methods, abstract vs interfaces

## Introduction
Abstraction is an object-oriented principle that hides implementation details and exposes only essential features. TypeScript provides `abstract` classes and methods to define incomplete blueprints that subclasses must complete. Abstraction reduces complexity, enforces consistent APIs, and decouples high-level policy from low-level implementation.

## abstract class

### What It Is
An abstract class is a class declared with the `abstract` keyword that cannot be instantiated directly. It serves as a base class that may contain both implemented methods and abstract method declarations.

### Why It Is Important
Abstract classes provide partial implementation that derived classes share while forcing them to implement specific behaviors. They capture commonality and mandate variation points.

### How It Works Internally
TypeScript compiles abstract classes to regular JavaScript constructor functions. The `abstract` keyword is erased, but the compiler prevents instantiation. Abstract methods are declared but not emitted — only their implementations in subclasses appear in the output.

### Syntax
```typescript
abstract class AbstractClass {
  abstract abstractMethod(): void;
  concreteMethod(): void {
    // implementation
  }
}
```

### Beginner Examples
```typescript
abstract class Animal {
  abstract makeSound(): void;
  move(): void {
    console.log("Moving...");
  }
}
// const a = new Animal(); // Error: cannot instantiate abstract class
class Dog extends Animal {
  makeSound(): void {
    console.log("Woof!");
  }
}
const d = new Dog();
d.makeSound(); // "Woof!"
d.move(); // "Moving..."
```

### Intermediate Examples
```typescript
abstract class Shape {
  constructor(public color: string) {}
  abstract area(): number;
  describe(): string {
    return `A ${this.color} shape with area ${this.area()}`;
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
abstract class TaskRunner {
  protected abstract preProcess(): Promise<void>;
  protected abstract process(): Promise<void>;
  protected abstract postProcess(): Promise<void>;
  async run(): Promise<void> {
    await this.preProcess();
    await this.process();
    await this.postProcess();
  }
}
class DataPipeline extends TaskRunner {
  protected async preProcess(): Promise<void> {
    // validate input data
  }
  protected async process(): Promise<void> {
    // transform data
  }
  protected async postProcess(): Promise<void> {
    // persist results
  }
}
```

### Real-World Use Cases
- Framework base classes (e.g., Angular `Directive`, NestJS `Guard`)
- Template method pattern for workflow engines
- Plugin architecture with shared lifecycle hooks

### Common Mistakes
- Trying to instantiate an abstract class directly
- Forgetting to implement all abstract methods in a concrete subclass
- Making classes abstract when they don't actually have unimplemented methods

### Best Practices
- Name abstract classes with a descriptive prefix or suffix (e.g., `BaseService`, `AbstractController`)
- Keep abstract classes focused — one abstraction per class
- Implement as much shared logic as possible to reduce subclass boilerplate

### Performance Considerations
Abstract classes have the same performance as regular classes. Virtual method dispatch through abstract methods is slightly slower than direct calls but optimized well by JIT compilers.

### Interview Questions
- Q: Can an abstract class have a constructor?
  A: Yes, abstract classes can have constructors that are called via `super()` from derived classes.

### Coding Challenges
- Create an abstract `DatabaseDriver` with `connect`, `disconnect`, and `query` methods. Implement `PostgresDriver` and `SQLiteDriver`.
- Build an abstract `ReportGenerator` with `fetchData`, `format`, and abstract `export`.

### Related Topics
- Abstract methods
- Abstract class vs interface
- Template method pattern

---

## abstract methods

### What It Is
An abstract method is a method declared with the `abstract` keyword within an abstract class. It has no implementation body and must be implemented by any concrete subclass.

### Why It Is Important
Abstract methods define a contract that subclasses must fulfill. They enforce a consistent API across a family of related classes while allowing each subclass to provide its own implementation.

### How It Works Internally
Abstract methods are erased during compilation. The compiler checks that all abstract methods are implemented in concrete classes at compile time.

### Syntax
```typescript
abstract class Base {
  abstract methodName(param: Type): ReturnType;
}
```

### Beginner Examples
```typescript
abstract class Logger {
  abstract log(message: string): void;
}
class ConsoleLogger extends Logger {
  log(message: string): void {
    console.log(message);
  }
}
class FileLogger extends Logger {
  log(message: string): void {
    // write to file
  }
}
```

### Intermediate Examples
```typescript
abstract class Validator {
  abstract validate(value: unknown): boolean;
  abstract getErrorMessage(): string;
}
class EmailValidator extends Validator {
  validate(value: unknown): boolean {
    return typeof value === "string" && value.includes("@");
  }
  getErrorMessage(): string {
    return "Invalid email address";
  }
}
```

### Advanced Examples
```typescript
abstract class Middleware<T extends Record<string, unknown>> {
  abstract handle(context: T, next: () => Promise<void>): Promise<void>;
}
class AuthMiddleware extends Middleware<{ user?: User }> {
  async handle(context: { user?: User }, next: () => Promise<void>): Promise<void> {
    if (!context.user) throw new Error("Unauthorized");
    await next();
  }
}
```

### Real-World Use Cases
- Strategy pattern where abstract method defines the algorithm interface
- Visitor pattern with abstract `accept` method
- Template method pattern with abstract steps

### Common Mistakes
- Declaring an abstract method outside an abstract class
- Providing a body for an abstract method
- Forgetting to implement all abstract methods in a concrete subclass

### Best Practices
- Keep abstract method signatures stable — changing them breaks all subclasses
- Document the expected behavior of abstract methods with comments
- Keep the number of abstract methods small (ideally 1–3)

### Performance Considerations
No runtime overhead for the abstract declaration itself. Dynamic dispatch through the prototype chain is the same as for regular overridden methods.

### Interview Questions
- Q: Can an abstract method be private?
  A: No, abstract methods must be `public` or `protected` so subclasses can implement them.

### Coding Challenges
- Define an abstract class `SortStrategy` with an abstract `sort<T>(items: T[]): T[]` method. Implement `BubbleSort` and `QuickSort`.
- Create an abstract `NotificationSender` with abstract `send(recipient: string, message: string): Promise<boolean>`.

### Related Topics
- abstract class
- Method overriding
- abstract class vs interface

---

## abstract class vs interface

### What It Is
Both abstract classes and interfaces define contracts, but abstract classes can provide implementation, constructors, and access modifiers, while interfaces are purely structural.

### Why It Is Important
Choosing between abstract class and interface affects the design's flexibility, reusability, and future evolution. Understanding the trade-offs is essential for sound architecture.

### How It Works Internally
Abstract classes compile to constructor functions with prototype methods. Interfaces are completely erased. Abstract classes support runtime checks with `instanceof`; interfaces do not.

### Syntax
```typescript
// interface — pure contract
interface Drivable {
  drive(): void;
}
// abstract class — contract + implementation
abstract class Vehicle {
  abstract drive(): void;
  refuel(): void { console.log("Refueling..."); }
}
```

### Beginner Examples
```typescript
// Use interface when no implementation is needed
interface Flyable {
  fly(): void;
}
class Bird implements Flyable {
  fly(): void { console.log("Flying"); }
}

// Use abstract class when shared implementation is needed
abstract class Bird {
  abstract fly(): void;
  eat(): void { console.log("Eating"); }
}
class Sparrow extends Bird {
  fly(): void { console.log("Sparrow flying"); }
}
```

### Intermediate Examples
```typescript
// interface for unrelated types that share behavior
interface Serializable {
  toJSON(): unknown;
}
class User implements Serializable { /* ... */ }
class Order implements Serializable { /* ... */ }

// abstract class for related types with shared state
abstract class Animal {
  constructor(protected name: string) {}
  abstract sound(): string;
  introduce(): string { return `I am ${this.name}, ${this.sound()}`; }
}
```

### Advanced Examples
```typescript
// combining both
interface Repository<T> {
  find(id: string): Promise<T | null>;
  save(entity: T): Promise<void>;
}
abstract class BaseRepository<T> implements Repository<T> {
  protected abstract getCollection(): string;
  async find(id: string): Promise<T | null> {
    const db = await getDB();
    return db.query(this.getCollection()).findById(id);
  }
  async save(entity: T): Promise<void> {
    const db = await getDB();
    await db.query(this.getCollection()).insert(entity);
  }
}
```

### Real-World Use Cases
- Use interfaces for cross-cutting contracts (e.g., `Serializable`, `Clonable`)
- Use abstract classes for framework base classes that provide infrastructure
- Use interfaces when you need multiple inheritance (a class can implement several interfaces but extend only one abstract class)

### Common Mistakes
- Choosing abstract class over interface when no shared implementation exists
- Adding default methods to an interface (available in TS but can lead to fragile designs)
- Inheriting an abstract class from a non-abstract base that has conflicting state

### Best Practices
- Prefer interfaces for public APIs and libraries (easier for consumers to implement)
- Use abstract classes when constructors, state, or access modifiers are needed
- Follow "program to an interface, not an implementation"

### Performance Considerations
Interfaces have zero cost. Abstract classes have the same cost as any class. `instanceof` checks with abstract classes are slightly slower than concrete ones but rarely a bottleneck.

### Interview Questions
- Q: Can an abstract class implement an interface?
  A: Yes, abstract classes can implement interfaces. The abstract class may leave some interface methods abstract.

### Coding Challenges
- Model a payment system: define an `PaymentProcessor` interface, then an `AbstractPaymentProcessor` abstract class that implements it and adds logging.
- Define both an `ICache` interface and an `AbstractCache` abstract class; implement `RedisCache` extending the abstract class.

### Related Topics
- interface declaration
- abstract class
- Inheritance

---

## Design implications

### What It Is
Design implications refer to the architectural consequences of choosing abstraction mechanisms — how abstract classes and interfaces affect coupling, testing, evolution, and team collaboration.

### Why It Is Important
Abstraction choices ripple through the entire codebase. Good abstraction reduces coupling and improves testability; poor abstraction creates unnecessary indirection and maintenance burden.

### How It Works Internally
The TypeScript compiler does not enforce design principles, but the type system enables techniques like dependency inversion and interface segregation.

### Beginner Examples
```typescript
// Tight coupling — hard to test
class OrderService {
  private db = new MySQLDatabase();
  save(order: Order): void {
    this.db.insert(order);
  }
}

// Loose coupling — testable
class OrderService {
  constructor(private db: Database) {}
  save(order: Order): void {
    this.db.insert(order);
  }
}
```

### Intermediate Examples
```typescript
// Dependency inversion via abstraction
interface NotificationService {
  send(recipient: string, message: string): Promise<void>;
}
class EmailService implements NotificationService {
  async send(recipient: string, message: string): Promise<void> {
    // send email
  }
}
class SMSService implements NotificationService {
  async send(recipient: string, message: string): Promise<void> {
    // send SMS
  }
}
class AlertSystem {
  constructor(private notifier: NotificationService) {}
  async alert(msg: string): Promise<void> {
    await this.notifier.send("admin", msg);
  }
}
```

### Advanced Examples
```typescript
// Abstract factory pattern
interface Widget {
  render(): HTMLElement;
}
interface ThemeFactory {
  createButton(): Widget;
  createInput(): Widget;
}
class DarkThemeFactory implements ThemeFactory {
  createButton(): Widget {
    return new DarkButton();
  }
  createInput(): Widget {
    return new DarkInput();
  }
}
class LightThemeFactory implements ThemeFactory {
  createButton(): Widget {
    return new LightButton();
  }
  createInput(): Widget {
    return new LightInput();
  }
}
class App {
  constructor(private theme: ThemeFactory) {}
  buildUI(): void {
    const btn = this.theme.createButton();
    const input = this.theme.createInput();
    // ...
  }
}
```

### Real-World Use Cases
- Hexagonal architecture (ports and adapters) using interfaces
- Plugin systems with abstract base classes
- Strategy pattern for interchangeable algorithms

### Common Mistakes
- Premature abstraction — creating interfaces or abstract classes before there are multiple implementations
- Leaky abstractions that expose implementation details
- Over-abstraction making the codebase difficult to navigate

### Best Practices
- Follow the "Rule of Three" — don't abstract until there are three concrete cases
- Design abstractions from the consumer's perspective (interface segregation)
- Prefer composition over inheritance for flexibility

### Performance Considerations
Abstraction itself adds no runtime cost in TypeScript. However, indirection can make code harder for the optimizer to inline. Profile before optimizing away abstractions.

### Interview Questions
- Q: What is the "Abstraction Principle"?
  A: Each significant piece of functionality should be behind a single abstraction that hides its implementation.

### Coding Challenges
- Refactor a tightly coupled class into one using dependency injection through an interface.
- Design an abstraction for a `FileStorage` that supports `LocalStorage`, `S3Storage`, and `AzureBlobStorage`.

### Related Topics
- abstract class vs interface
- SOLID principles
- Dependency injection
