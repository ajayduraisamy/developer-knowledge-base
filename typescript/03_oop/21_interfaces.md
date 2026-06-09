# Interfaces - interface declaration, optional/readonly, extends, hybrid types, vs type

## Introduction
TypeScript interfaces define the shape of values without providing implementation. They serve as contracts that objects, classes, and functions can adhere to. Interfaces are a cornerstone of TypeScript's structural type system, enabling duck typing with compile-time safety. They promote decoupled, testable code by allowing consumers to depend on abstractions rather than concrete implementations.

## interface declaration

### What It Is
An interface declaration introduces a named type using the `interface` keyword, describing the structure that conforming values must have.

### Why It Is Important
Interfaces allow you to document the expected shape of objects, enable IDE autocompletion, and catch type mismatches at compile time without runtime overhead.

### How It Works Internally
TypeScript interfaces exist only at compile time. They have no runtime representation in the emitted JavaScript. The type checker uses interfaces during structural compatibility checks.

### Syntax
```typescript
interface InterfaceName {
  property: Type;
  method(param: Type): ReturnType;
}
```

### Beginner Examples
```typescript
interface Person {
  name: string;
  age: number;
}
const alice: Person = { name: "Alice", age: 30 };
```

### Intermediate Examples
```typescript
interface HTTPResponse {
  statusCode: number;
  headers: Record<string, string>;
  body: string;
  toJSON<T>(): T;
}
```

### Advanced Examples
```typescript
interface GenericRepository<T, K extends keyof T> {
  find(id: K): Promise<T | null>;
  findAll(): Promise<T[]>;
  create(data: Omit<T, K>): Promise<T>;
}
```

### Real-World Use Cases
- Defining payload types for API responses
- Describing configuration objects
- Contracting data-access layers
- Modeling domain events

### Common Mistakes
- Using interfaces when `type` aliases are more appropriate (e.g., union types)
- Adding methods to an interface that should be value types
- Over-engineering with many tiny interfaces prematurely

### Best Practices
- Keep interfaces focused on a single responsibility
- Use descriptive property names
- Prefer interfaces over `type` for object shapes that may be extended

### Performance Considerations
Zero runtime overhead. Interfaces are erased at compile time.

### Interview Questions
- Q: Can an interface extend a class?
  A: Yes, an interface can extend a class, inheriting the class's member signatures but not its implementation.

### Coding Challenges
- Define an interface `Comparable<T>` with a `compareTo(other: T): number` method.
- Create interfaces for a simple e-commerce domain: `Product`, `CartItem`, `Order`.

### Related Topics
- Optional and readonly properties
- `extends` for interfaces
- `interface` vs `type`

---

## Optional and readonly properties

### What It Is
Optional properties are denoted with `?` and may be absent when the value is created. `readonly` properties can be assigned only during initial object creation and are immutable thereafter.

### Why It Is Important
Optional properties model fields that are not always required, reducing the need for `undefined`. Readonly properties enforce immutability at the type level, preventing accidental mutations.

### How It Works Internally
`?` adds `| undefined` to the type and makes property assignment optional. `readonly` is a compile-time constraint — JavaScript does not enforce it at runtime.

### Syntax
```typescript
interface Config {
  name: string;
  debug?: boolean;
  readonly id: string;
}
```

### Beginner Examples
```typescript
interface UserProfile {
  displayName: string;
  bio?: string;
  readonly createdAt: Date;
}
const profile: UserProfile = {
  displayName: "Alice",
  createdAt: new Date(),
};
```

### Intermediate Examples
```typescript
interface APIOptions {
  baseURL: string;
  timeout?: number;
  retries?: number;
  readonly apiKey: string;
}
```

### Advanced Examples
```typescript
interface ReadonlyState<T> {
  readonly [K in keyof T]: ReadonlyState<T[K]>;
}
interface AppState {
  readonly user: ReadonlyState<User>;
  readonly ui: { readonly theme: string };
}
```

### Real-World Use Cases
- Configuration objects where some fields are optional
- Immutable event payloads in event sourcing
- API responses with computed read-only identifiers

### Common Mistakes
- Forgetting that `readonly` only prevents assignment to the property, not mutation of nested objects
- Using `readonly` on arrays without `ReadonlyArray<T>`
- Making properties optional when they should be required but nullable

### Best Practices
- Use `readonly` for identifiers and timestamps
- Combine `readonly` with `Readonly<T>` for deeply immutable types
- Document when optional properties have fallback defaults

### Performance Considerations
No runtime impact. These are purely type-level constraints.

### Interview Questions
- Q: How do you make all properties of an interface readonly?
  A: Use the `Readonly<T>` utility type or a mapped type: `type Immutable<T> = { readonly [K in keyof T]: T[K] }`.

### Coding Challenges
- Create an interface `ImmutableUser` where all properties are readonly and `name` is required but `nickname` is optional.
- Write a generic utility type `Writable<T>` that removes readonly from all properties.

### Related Topics
- interface declaration
- `Readonly<T>` utility type
- Required and Partial utility types

---

## interface extends

### What It Is
Interface inheritance allows one interface to extend one or more other interfaces, inheriting all their members and optionally adding new ones.

### Why It Is Important
Extending interfaces promotes code reuse and models hierarchical type relationships (e.g., a `AdminUser` extends `User`). It enables compositional type design.

### How It Works Internally
The compiler merges the members of all extended interfaces into the extending interface. If two parent interfaces declare the same property with incompatible types, a compile error occurs.

### Syntax
```typescript
interface Base {
  id: string;
}
interface Derived extends Base {
  extra: number;
}
```

### Beginner Examples
```typescript
interface Named {
  name: string;
}
interface Aged {
  age: number;
}
interface Person extends Named, Aged {
  email: string;
}
```

### Intermediate Examples
```typescript
interface Timestamped {
  createdAt: Date;
  updatedAt: Date;
}
interface SoftDeletable {
  deletedAt: Date | null;
}
interface AuditableEntity extends Timestamped, SoftDeletable {
  createdBy: string;
  updatedBy: string;
}
```

### Advanced Examples
```typescript
interface JSONSerializable {
  toJSON(): Record<string, unknown>;
}
interface Loggable {
  log(level: string): void;
}
interface ServiceContract extends JSONSerializable, Loggable {
  execute(): Promise<unknown>;
}
```

### Real-World Use Cases
- Building layered type hierarchies in domain models
- Composing cross-cutting concerns (auditable, soft-deletable, versionable)
- Extending third-party library types in declaration merging

### Common Mistakes
- Extending too deeply (deep inheritance chains are hard to reason about)
- Extending interfaces that have conflicting property types
- Using `extends` when intersection types (`&`) would be simpler

### Best Practices
- Prefer shallow inheritance (one level of extension)
- Use interface extension over intersection for intentional type relationships
- Favor composition of small interfaces over monolithic hierarchies

### Performance Considerations
No runtime cost. All extension is resolved at compile time.

### Interview Questions
- Q: Can an interface extend a type alias?
  A: Yes, if the type alias resolves to an object type. It cannot extend union types.

### Coding Challenges
- Define a type hierarchy: `Entity` (id), `AuditableEntity` (extends Entity, adds timestamps), `User` (extends AuditableEntity).
- Create interfaces for a plugin system: `Plugin` (name, version), `ConfigurablePlugin` (extends Plugin, adds configure method).

### Related Topics
- interface declaration
- Intersection types
- Declaration merging

---

## Hybrid types

### What It Is
Hybrid types are interfaces that describe both callable signatures (functions) and static properties. They model objects that are functions but also carry additional data.

### Why It Is Important
Some JavaScript patterns (e.g., jQuery, `regExp.test`, counter functions) create objects callable as functions with extra properties. Hybrid types let TypeScript model these patterns with full type safety.

### How It Works Internally
The compiler resolves the call signature for function calls and the property signatures for member access. The emitted JavaScript is unchanged.

### Syntax
```typescript
interface Hybrid {
  (param: Type): ReturnType;
  property: Type;
}
```

### Beginner Examples
```typescript
interface Greeter {
  (name: string): void;
  defaultName: string;
}
function createGreeter(): Greeter {
  const fn = ((name: string) => {
    console.log(`Hello, ${name}!`);
  }) as Greeter;
  fn.defaultName = "World";
  return fn;
}
const greet = createGreeter();
greet("Alice");
console.log(greet.defaultName);
```

### Intermediate Examples
```typescript
interface Counter {
  (start: number): string;
  interval: number;
  reset(): void;
}
function createCounter(): Counter {
  const counter = ((start: number): string => {
    return `Count: ${start}`;
  }) as Counter;
  counter.interval = 1000;
  counter.reset = () => { counter.interval = 1000; };
  return counter;
}
```

### Advanced Examples
```typescript
interface CurrencyFormatter {
  (amount: number, currency: string): string;
  locale: string;
  setLocale(locale: string): void;
  currencies: readonly string[];
}
function createFormatter(defaultLocale = "en-US"): CurrencyFormatter {
  const fmt = ((amount: number, currency: string) => {
    return new Intl.NumberFormat(fmt.locale, {
      style: "currency",
      currency,
    }).format(amount);
  }) as CurrencyFormatter;
  fmt.locale = defaultLocale;
  fmt.setLocale = (l: string) => { fmt.locale = l; };
  fmt.currencies = ["USD", "EUR", "GBP"];
  return fmt;
}
```

### Real-World Use Cases
- jQuery-style libraries where the main function has utility properties
- Redux middleware with a callable signature and custom methods
- Validation libraries exposing a validator function with configuration

### Common Mistakes
- Forgetting to cast the function object with `as InterfaceType`
- Trying to use class instances as hybrid types (classes produce instances, not callable objects)
- Not accounting for the `this` context within the callable part

### Best Practices
- Use hybrid types sparingly — prefer classes or separate objects when possible
- Assert the type explicitly using `as` when constructing hybrid objects
- Document the function-property duality clearly

### Performance Considerations
Hybrid objects have the same performance as plain functions with properties. No TypeScript-specific overhead.

### Interview Questions
- Q: Can a hybrid type interface extend another interface?
  A: Yes, hybrid interfaces can extend other interfaces, adding both callable and property members.

### Coding Challenges
- Create a hybrid type `Logger` that is callable with a message and has `level`, `setLevel`, and `format` properties.
- Build a hybrid `TaskRunner` that can be called with a task ID and has `queue`, `running`, and `cancelAll` members.

### Related Topics
- Call signatures
- interface declaration
- Type assertions

---

## interface vs type

### What It Is
Both `interface` and `type` can define object shapes, but they have distinct capabilities: `interface` supports declaration merging and extends more naturally; `type` can represent unions, intersections, tuples, and primitives.

### Why It Is Important
Choosing the right construct affects code maintainability, extensibility, and clarity. The TypeScript community has conventions for when to use each.

### How It Works Internally
Both are erased at runtime. Interfaces are evaluated lazily and support merging; type aliases are computed eagerly and cannot merge.

### Syntax
```typescript
// interface
interface Foo { a: string; }
// type
type Foo = { a: string; };
```

### Beginner Examples
```typescript
interface User {
  name: string;
}
type User = {
  name: string;
};
// Both work identically for simple object types
```

### Intermediate Examples
```typescript
// interface can merge
interface Request {
  body: string;
}
interface Request {
  headers: Record<string, string>;
}
// merged: { body: string; headers: Record<string, string>; }

// type can define unions
type Status = "idle" | "loading" | "success" | "error";
type Result<T> = { data: T } | { error: Error };
```

### Advanced Examples
```typescript
// interface extends
interface BaseEntity { id: string; }
interface User extends BaseEntity { name: string; }

// type intersection
type BaseEntity = { id: string; };
type User = BaseEntity & { name: string; };

// type mapped types
type Readonly<T> = { readonly [K in keyof T]: T[K] };
// interface cannot do mapped types directly
```

### Real-World Use Cases
- Use `interface` for public API contracts and library type definitions (supports extension)
- Use `type` for computed types, unions, and complex transformations
- Use `interface` when you want consumers to be able to augment your types (declaration merging)

### Common Mistakes
- Assuming `type` and `interface` are interchangeable in all contexts (interface required for declaration merging)
- Using `type` when an `interface` would provide better error messages
- Trying to use `interface` for primitive type aliases or tuples

### Best Practices
- Prefer `interface` for object shapes that may be extended
- Prefer `type` for computed types (mapped, conditional, intersections of unions)
- Be consistent within a codebase

### Performance Considerations
Interfaces are slightly faster for the compiler to check in some cases due to lazy evaluation. In practice, the difference is negligible.

### Interview Questions
- Q: Can `type` alias be used in `implements` clause?
  A: Yes, if the type alias is an object type (not a union).

### Coding Challenges
- Model a `Result<T, E>` type using discriminated union with `type`.
- Define an `ApiResponse` interface that consumers can extend with custom fields.

### Related Topics
- Type aliases
- Declaration merging
- Union and intersection types
