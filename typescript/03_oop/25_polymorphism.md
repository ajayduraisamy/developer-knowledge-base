# Polymorphism - Subtype polymorphism, method overriding, generic polymorphism, parametric

## Introduction
Polymorphism allows objects of different types to be treated uniformly through a common interface. TypeScript supports several forms: subtype polymorphism (inheritance-based), method overriding (dynamic dispatch), generic (parametric) polymorphism (type parameters), and polymorphic `this` types. Polymorphism is essential for writing flexible, reusable, and maintainable code.

## Subtype polymorphism

### What It Is
Subtype polymorphism (also called inclusion polymorphism) allows a derived class instance to be used wherever a base class or interface is expected.

### Why It Is Important
Subtype polymorphism enables the Liskov Substitution Principle — programs can operate on base types and work correctly with any subtype. This is fundamental for design patterns like Strategy, Command, and Template Method.

### How It Works Internally
TypeScript uses structural typing. An object is a subtype if it has all the required members with compatible types. At runtime, JavaScript's prototype chain handles method dispatch.

### Syntax
```typescript
interface Shape {
  area(): number;
}
class Circle implements Shape { /* ... */ }
function printArea(s: Shape): void { console.log(s.area()); }
printArea(new Circle(5));
```

### Beginner Examples
```typescript
interface Animal {
  speak(): string;
}
class Dog implements Animal {
  speak(): string { return "Woof"; }
}
class Cat implements Animal {
  speak(): string { return "Meow"; }
}
function makeSound(animal: Animal): void {
  console.log(animal.speak());
}
const animals: Animal[] = [new Dog(), new Cat()];
animals.forEach(makeSound); // Woof, Meow
```

### Intermediate Examples
```typescript
interface PaymentMethod {
  pay(amount: number): Promise<PaymentResult>;
}
class CreditCard implements PaymentMethod {
  async pay(amount: number): Promise<PaymentResult> {
    // charge credit card
    return { success: true, transactionId: "cc-123" };
  }
}
class PayPal implements PaymentMethod {
  async pay(amount: number): Promise<PaymentResult> {
    // redirect to PayPal
    return { success: true, transactionId: "pp-456" };
  }
}
class CheckoutService {
  async checkout(method: PaymentMethod, amount: number): Promise<PaymentResult> {
    return method.pay(amount);
  }
}
```

### Advanced Examples
```typescript
interface Serializer<T, R> {
  serialize(data: T): R;
  deserialize(raw: R): T;
}
const jsonSerializer: Serializer<User, string> = {
  serialize: (u) => JSON.stringify(u),
  deserialize: (s) => JSON.parse(s),
};
const msgpackSerializer: Serializer<User, Buffer> = {
  serialize: (u) => msgpack.encode(u),
  deserialize: (b) => msgpack.decode(b),
};
function processData<T>(data: T, serializer: Serializer<T, string>): void {
  const serialized = serializer.serialize(data);
  // send over network
}
```

### Real-World Use Cases
- Plugin architectures where plugins conform to a common interface
- Strategy pattern for interchangeable algorithms
- Adapter pattern for integrating disparate systems

### Common Mistakes
- Violating the Liskov Substitution Principle by strengthening preconditions or weakening postconditions in subtypes
- Using subtype polymorphism when parametric polymorphism (generics) would be simpler
- Deep inheritance hierarchies that make subtype relationships confusing

### Best Practices
- Design interfaces from the consumer's perspective
- Keep interfaces small and focused (Interface Segregation)
- Test base type contracts against all subtypes

### Performance Considerations
Dynamic dispatch has a negligible cost. Modern JIT compilers inline polymorphic calls when the type is monomorphic or bimorphic.

### Interview Questions
- Q: What is the difference between subtype polymorphism and parametric polymorphism?
  A: Subtype works through inheritance/interface implementation; parametric uses type parameters (generics).

### Coding Challenges
- Create an `ExportStrategy` interface with `export(data: unknown[]): string`. Implement `CSVExport`, `JSONExport`, and `XMLExport`.
- Build a `Logger` interface and implement `ConsoleLogger`, `FileLogger`, and `RemoteLogger`.

### Related Topics
- Method overriding polymorphism
- Generic (parametric) polymorphism
- Interfaces

---

## Method overriding polymorphism

### What It Is
Method overriding polymorphism occurs when a subclass redefines a method from its superclass, and the correct implementation is resolved at runtime based on the actual object type (dynamic dispatch).

### Why It Is Important
Overriding enables subclasses to specialize behavior while preserving a consistent interface. It is the mechanism through which subtype polymorphism delivers differentiated behavior.

### How It Works Internally
JavaScript looks up methods on the prototype chain. When a method is called, the engine searches the object's own properties, then its prototype, then the prototype's prototype, etc., finding the first match.

### Syntax
```typescript
class Base {
  execute(): void { console.log("Base"); }
}
class Derived extends Base {
  execute(): void { console.log("Derived"); }
}
```

### Beginner Examples
```typescript
class Shape {
  area(): number {
    return 0;
  }
}
class Circle extends Shape {
  constructor(private radius: number) { super(); }
  area(): number {
    return Math.PI * this.radius ** 2;
  }
}
class Rectangle extends Shape {
  constructor(private w: number, private h: number) { super(); }
  area(): number {
    return this.w * this.h;
  }
}
const shapes: Shape[] = [new Circle(5), new Rectangle(4, 6)];
shapes.forEach(s => console.log(s.area()));
```

### Intermediate Examples
```typescript
abstract class ReportGenerator {
  abstract getData(): Promise<unknown[]>;
  abstract format(data: unknown[]): string;
  async generate(): Promise<string> {
    const data = await this.getData();
    return this.format(data);
  }
}
class SalesReport extends ReportGenerator {
  async getData(): Promise<unknown[]> {
    return db.query("SELECT * FROM sales");
  }
  format(data: unknown[]): string {
    return data.map(d => JSON.stringify(d)).join("\n");
  }
}
```

### Advanced Examples
```typescript
abstract class ASTVisitor {
  abstract visitLiteral(node: Literal): void;
  abstract visitBinaryOp(node: BinaryOp): void;
  abstract visitFunctionCall(node: FunctionCall): void;
}
class EvalVisitor extends ASTVisitor {
  visitLiteral(node: Literal): void { /* evaluate */ }
  visitBinaryOp(node: BinaryOp): void { /* evaluate */ }
  visitFunctionCall(node: FunctionCall): void { /* evaluate */ }
}
class PrintVisitor extends ASTVisitor {
  visitLiteral(node: Literal): void { /* print */ }
  visitBinaryOp(node: BinaryOp): void { /* print */ }
  visitFunctionCall(node: FunctionCall): void { /* print */ }
}
```

### Real-World Use Cases
- Visitor pattern for AST processing
- Strategy pattern with overridden execution methods
- Template method pattern with overridden steps

### Common Mistakes
- Forgetting `override` keyword (TS 4.3+) when intended
- Changing the method signature when overriding (use `override` to catch this)
- Not calling `super.method()` when extending behavior

### Best Practices
- Always use the `override` keyword to validate the override
- Keep overridden method signatures compatible with the base
- Consider whether the base method should be abstract or have a default implementation

### Performance Considerations
Method dispatch through `super` calls traverses the prototype chain. Direct overrides without `super` are as fast as any method call.

### Interview Questions
- Q: What happens if a subclass does not override a method that exists in the parent?
  A: The parent's implementation is inherited and used.

### Coding Challenges
- Create a `Character` class with an `attack()` method. Override it in `Warrior`, `Mage`, and `Archer`.
- Build a `FileHandler` base class with `open()`, `read()`, and `close()`. Override for `TextFileHandler` and `BinaryFileHandler`.

### Related Topics
- Subtype polymorphism
- Inheritance
- Abstract classes

---

## Generic (parametric) polymorphism

### What It Is
Generic polymorphism (parametric polymorphism) uses type parameters to define functions, classes, and interfaces that can work with any type while maintaining type safety.

### Why It Is Important
Generics enable code reuse across types without sacrificing type safety. They allow you to write algorithms and data structures that work uniformly over a variety of types.

### How It Works Internally
TypeScript's generics are erased during compilation (like Java). The compiler checks type constraints at compile time but emits no generic type information in JavaScript.

### Syntax
```typescript
function identity<T>(arg: T): T {
  return arg;
}
class Box<T> {
  constructor(public value: T) {}
}
```

### Beginner Examples
```typescript
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}
const num = first([1, 2, 3]); // number
const str = first(["a", "b"]); // string
```

### Intermediate Examples
```typescript
interface Repository<T, K extends string | number> {
  find(id: K): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<void>;
  delete(id: K): Promise<void>;
}
class InMemoryRepo<T, K extends string | number> implements Repository<T, K> {
  private items = new Map<K, T>();
  async find(id: K): Promise<T | null> { return this.items.get(id) ?? null; }
  async findAll(): Promise<T[]> { return Array.from(this.items.values()); }
  async save(entity: T & { id: K }): Promise<void> { this.items.set(entity.id, entity); }
  async delete(id: K): Promise<void> { this.items.delete(id); }
}
```

### Advanced Examples
```typescript
function mapObject<T, R>(
  obj: Record<string, T>,
  fn: (value: T, key: string) => R
): Record<string, R> {
  const result: Record<string, R> = {};
  for (const [key, value] of Object.entries(obj)) {
    result[key] = fn(value, key);
  }
  return result;
}
const doubled = mapObject({ a: 1, b: 2 }, (x) => x * 2); // { a: 2, b: 4 }
```

```typescript
// Generic constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
const user = { name: "Alice", age: 30 };
const name = getProperty(user, "name"); // string
// getProperty(user, "invalid"); // Error
```

```typescript
// Higher-kinded type simulation with generic factories
interface Functor<F> {
  map<T, R>(fn: (value: T) => R, container: F<T>): F<R>;
}
class ArrayFunctor implements Functor<Array> {
  map<T, R>(fn: (value: T) => R, arr: T[]): R[] {
    return arr.map(fn);
  }
}
```

### Real-World Use Cases
- Generic data structures (e.g., `Map<K, V>`, `Set<T>`, `Promise<T>`)
- Generic service classes that work with any entity type
- API client functions typed by response schema

### Common Mistakes
- Over-constraining generic parameters (requiring more than needed)
- Using `any` when a generic would maintain type safety
- Not providing type arguments when they cannot be inferred

### Best Practices
- Use descriptive type parameter names (`T`, `K`, `V`, `R` for common; `TEntity`, `TId` for clarity)
- Constrain generics with `extends` only when necessary
- Prefer generic functions over function overloads

### Performance Considerations
Generics are erased at compile time — zero runtime overhead. However, excessive generic instantiation can slow compilation.

### Interview Questions
- Q: What is a generic constraint?
  A: A restriction on a type parameter using the `extends` keyword (e.g., `<T extends HasId>`).

### Coding Challenges
- Implement a generic `Stack<T>` class with `push`, `pop`, and `peek`.
- Write a generic `deepClone<T>(obj: T): T` function.
- Create a generic `EventEmitter<T extends Record<string, unknown[]>>` with typed events.

### Related Topics
- Subtype polymorphism
- Polymorphic `this` types
- Generic constraints

---

## Polymorphic this types

### What It Is
A polymorphic `this` type represents the type of the current class or interface. It is used in method return types to enable fluent interfaces and method chaining in derived classes.

### Why It Is Important
`this` types ensure that method chaining works correctly with inheritance. Without `this` types, chaining from a derived class returns the base class type, losing derived-specific methods.

### How It Works Internally
`this` is treated as an implicit type parameter. When a method returns `this`, the return type is determined by the actual type of the instance at the call site.

### Syntax
```typescript
class Base {
  setA(value: number): this {
    // ...
    return this;
  }
}
```

### Beginner Examples
```typescript
class StringBuilder {
  private parts: string[] = [];
  append(str: string): this {
    this.parts.push(str);
    return this;
  }
  build(): string {
    return this.parts.join("");
  }
}
const sb = new StringBuilder();
const result = sb.append("Hello").append(" ").append("World").build();
// "Hello World"
```

### Intermediate Examples
```typescript
class QueryBuilder<T> {
  protected filters: string[] = [];
  protected selectFields: string[] = ["*"];
  where(condition: string): this {
    this.filters.push(condition);
    return this;
  }
  select(...fields: string[]): this {
    this.selectFields = fields;
    return this;
  }
  build(): string {
    let sql = `SELECT ${this.selectFields.join(", ")} FROM ${this.getTable()}`;
    if (this.filters.length > 0) {
      sql += ` WHERE ${this.filters.join(" AND ")}`;
    }
    return sql;
  }
  protected getTable(): string {
    return (this as any).constructor.name;
  }
}
class UserQuery extends QueryBuilder<User> {
  whereName(name: string): this {
    return this.where(`name = '${name}'`);
  }
}
const query = new UserQuery()
  .select("id", "name")
  .whereName("Alice")
  .where("active = true")
  .build();
// Works because select() returns UserQuery (this), not QueryBuilder
```

### Advanced Examples
```typescript
abstract class Builder<T> {
  protected config: Partial<T> = {};
  set<K extends keyof T>(key: K, value: T[K]): this {
    this.config[key] = value;
    return this;
  }
  abstract build(): T;
}
interface UserConfig {
  name: string;
  age: number;
  email: string;
}
class UserBuilder extends Builder<UserConfig> {
  build(): UserConfig {
    return this.config as UserConfig;
  }
}
const user = new UserBuilder()
  .set("name", "Alice")
  .set("age", 30)
  .set("email", "alice@example.com")
  .build();
```

```typescript
// Fluent repository pattern
class FluentRepo<T, K extends string | number> {
  protected query: Partial<T> = {};
  protected limitCount?: number;
  where<F extends keyof T>(field: F, value: T[F]): this {
    this.query[field] = value;
    return this;
  }
  limit(n: number): this {
    this.limitCount = n;
    return this;
  }
  async execute(): Promise<T[]> {
    // execute query
    return [];
  }
}
```

### Real-World Use Cases
- Fluent API / builder pattern implementations
- Query builder DSLs (SQL, MongoDB aggregation)
- Configuration builders with chained methods

### Common Mistakes
- Returning `this` when the method should return the base class type
- Using `this` as a type in contexts where it doesn't refer to the instance (e.g., static methods)
- Assuming `this` types work with interfaces that don't declare them

### Best Practices
- Use `this` return types for all fluent chain methods
- Combine with generics for maximum flexibility
- Avoid state mutation that would make chaining confusing

### Performance Considerations
No runtime cost. TypeScript resolves `this` to the concrete instance type at compile time.

### Interview Questions
- Q: How does `this` type differ from a generic type parameter?
  A: `this` is implicitly determined by the instance's actual type; generics are explicitly provided or inferred.

### Coding Challenges
- Create an `HTTPRequestBuilder` with methods `setURL`, `setMethod`, `setHeader`, `setBody`, all returning `this`.
- Build a `Pipeline<T>` class with `pipe(step: (val: T) => T): this` for chaining transformations.

### Related Topics
- Generic (parametric) polymorphism
- Method chaining
- Fluent interface pattern
