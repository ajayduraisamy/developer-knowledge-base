# Intersection Types - Intersection syntax (&), combining types, conflicts, vs inheritance

## Introduction

Intersection types in TypeScript allow you to combine multiple types into a single type. A value of an intersection type must satisfy *all* constituent types simultaneously. Syntactically, intersection types use the ampersand (`&`) operator: `A & B` means a value that is both `A` and `B`. Intersection types are a fundamental building block of TypeScript's structural type system, enabling composition-based type design over traditional inheritance hierarchies.

Intersection types are analogous to logical AND at the type level: `A & B` requires all properties from `A` and all properties from `B`. This contrasts with union types (`A | B`), which require at least one. Intersection types are the mechanism behind mixins, type merging, and combining multiple interfaces without creating explicit subtyping relationships.

## Intersection syntax (&)

### What It Is

The intersection type operator `&` combines two or more types into one. The resulting type has all properties of all component types. Intersection types can combine any types—primitives, objects, functions, and even unions.

```typescript
type Combined = { a: string } & { b: number };
// Result: { a: string; b: number }
```

### Why It Is Important

Intersection types enable code reuse through composition rather than inheritance. Instead of creating deep class hierarchies, you can assemble types from smaller, focused pieces. This aligns with the principle of composition over inheritance. Intersection types are also essential for mixins, augmenting third-party types, and modeling cross-cutting concerns.

### How It Works Internally

The TypeScript compiler computes intersection types by merging property declarations. When `A & B` is evaluated, the compiler creates a new type with the union of all properties from `A` and `B`. If both `A` and `B` have a property with the same name, the property type becomes the *intersection* of the individual property types (since the value must satisfy both).

```typescript
// { a: string } & { a: number } → { a: string & number } → { a: never }
// Because a value cannot be both string and number.
```

### Syntax

```typescript
// Basic intersection
type Named = { name: string };
type Aged = { age: number };
type Person = Named & Aged;

// Multiple intersections
type A = { a: string };
type B = { b: number };
type C = { c: boolean };
type ABC = A & B & C;

// With primitive types (nonsensical but valid syntax)
type NeverType = string & number; // never

// With function types
type Loggable = { log: (msg: string) => void };
type Serializable = { serialize: () => string };
type LoggableSerializable = Loggable & Serializable;

// Intersection of unions
type U1 = string | number;
type U2 = number | boolean;
type U3 = U1 & U2; // number
```

### Beginner Examples

```typescript
interface HasName { name: string; }
interface HasAge { age: number; }
type Person = HasName & HasAge;

const person: Person = { name: "Alice", age: 30 };

// Combining multiple interfaces
interface Identifiable { id: string; }
interface Timestampable { createdAt: Date; updatedAt: Date; }
type Entity = Identifiable & Timestampable;

const entity: Entity = {
  id: "123",
  createdAt: new Date(),
  updatedAt: new Date(),
};

// Intersection with existing type
type User = { name: string } & { email: string };
const user: User = { name: "Bob", email: "bob@example.com" };
```

### Intermediate Examples

```typescript
// Intersection with generics
type WithId<T> = T & { id: string };
type NamedEntity = WithId<{ name: string }>;

const entity: NamedEntity = { id: "1", name: "Widget" };

// Intersection of complex types
type RequestConfig = {
  url: string;
  method: "GET" | "POST" | "PUT" | "DELETE";
};
type AuthConfig = {
  token: string;
  refreshToken?: string;
};
type AuthenticatedRequest = RequestConfig & AuthConfig;

const req: AuthenticatedRequest = {
  url: "/api/data",
  method: "GET",
  token: "abc123",
};

// Intersection with arrays
type NumberArray = number[];
type StringArray = string[];
// NumberArray & StringArray → never (contradiction)
type MixedArray = (number | string)[];

// Intersection for function merging
type WithLogging = <T>(fn: () => T) => T;
type WithTiming = <T>(fn: () => T) => { result: T; duration: number };
// Intersection of function types creates an overloaded signature
```

### Advanced Examples

```typescript
// Intersection in conditional types
type IsString<T> = T extends string ? true : false;
type Result = IsString<"hello">; // true

// Using intersection to merge mapped types
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};
type Setters<T> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};
type Props<T> = Getters<T> & Setters<T>;

type PersonProps = Props<{ name: string; age: number }>;
// { getName: () => string; setName: (value: string) => void;
//   getAge: () => number; setAge: (value: number) => void; }

// Distributive intersection with conditional types
type UnionToIntersection<U> = 
  (U extends unknown ? (x: U) => void : never) extends 
  (x: infer I) => void ? I : never;

type Test = UnionToIntersection<{ a: string } | { b: number }>;
// { a: string } & { b: number }

// Recursive intersection
type DeepPartial<T> = T extends object
  ? { [P in keyof T]?: DeepPartial<T[P]> }
  : T;

type DeepRequired<T> = T extends object
  ? { [P in keyof T]-?: DeepRequired<T[P]> }
  : T;
```

### Real-World Use Cases

- **Mixins**: Combining behavior from multiple sources into a single type.
- **Redux state**: Combining slice states into a root state.
- **API request configuration**: Merging base config with endpoint-specific config.
- **Plugin systems**: Extending base types with plugin-provided properties.
- **ORM models**: Combining base entity fields with type-specific fields.
- **Middleware chains**: Combining middleware context objects.

```typescript
// Real-world: Mixin pattern
type Constructor<T = {}> = new (...args: any[]) => T;

function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    createdAt = new Date();
    updatedAt = new Date();
  };
}

function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActive = false;
    activate() { this.isActive = true; }
    deactivate() { this.isActive = false; }
  };
}

class User { name: string = ""; }
const ActiveUser = Activatable(Timestamped(User));
// ActiveUser type: User & { createdAt: Date; updatedAt: Date; isActive: boolean; ... }

// Real-world: Express middleware context
type Request = { params: Record<string, string> };
type AuthPayload = { userId: string; roles: string[] };
type AuthenticatedRequest = Request & { user: AuthPayload };

// Real-world: Config merging
type BaseConfig = { env: string; debug: boolean };
type DatabaseConfig = { url: string; poolSize: number };
type ServerConfig = { port: number; host: string };
type AppConfig = BaseConfig & DatabaseConfig & ServerConfig;
```

### Common Mistakes

1. **Intersecting incompatible primitives**: `string & number` results in `never`.
2. **Expecting intersection to merge deep properties**: Intersection only merges top-level properties. Nested objects are intersected recursively only if structurally aligned.
3. **Confusing intersection with union**: `A & B` requires both types; `A | B` requires at least one.
4. **Property name conflicts**: When two intersected types have the same property with incompatible types, the result is `never`.
5. **Overusing intersection for what should be inheritance**: Sometimes `extends` is clearer.

```typescript
// Mistake: incompatible properties
type A = { value: string };
type B = { value: number };
type C = A & B; // { value: never }

// Mistake: deep conflict
type X = { nested: { a: string } };
type Y = { nested: { b: number } };
type Z = X & Y; // { nested: { a: string } & { b: number } } = { nested: { a: string; b: number } }
// This works! But deep merge is not guaranteed for complex types.
```

### Best Practices

- Use intersection types for composition; use `extends` for specialization.
- Prefer `interface extends` when you control all the types being combined.
- Use intersection when combining types you don't control or when you want to merge types from different modules.
- Be aware of property conflicts—they silently produce `never`.
- Combine intersection with mapped and conditional types for powerful type transformations.
- Name your intersection types with `&` in the alias for clarity.

### Performance Considerations

Intersection types are compile-time only. However, very complex intersections (deeply nested, many members) can slow down the TypeScript compiler. The compiler must check each property in each constituent type during assignability checks. For performance-critical code, prefer interfaces with `extends` for type combinations, as they are more efficiently cached.

### Interview Questions

1. **Q**: What happens when you intersect two types with the same property name but different types?
   **A**: The property type becomes the intersection of the individual types. If they are incompatible (e.g., `string & number`), the result is `never`.

2. **Q**: How do intersection and union types relate to each other?
   **A**: They follow distributive laws: `A & (B | C) = (A & B) | (A & C)`. Intersection distributes over union.

3. **Q**: What is the difference between `interface A extends B, C` and `type A = B & C`?
   **A**: Interfaces cannot extend complex types (like mapped or conditional types). Type aliases with intersection can. Interfaces can be declaration-merged; intersection types cannot.

### Coding Challenges

1. Implement a `DeepMerge<T, U>` type that deeply merges two object types.
2. Create a mixin system using intersection types and generic classes.
3. Write a type that takes a union and returns an intersection of its members.

### Related Topics

- Union types
- Interface extends
- Mixins
- Type composition
- `never` type

## Combining object types

### What It Is

Combining object types through intersection creates a new type with all properties from all component types. This is the primary use case for intersection types—composing smaller, focused types into larger, comprehensive ones.

```typescript
type BasicInfo = { name: string; email: string };
type Address = { street: string; city: string; zip: string };
type Contact = BasicInfo & Address;
// { name: string; email: string; street: string; city: string; zip: string }
```

### Why It Is Important

Object type combination via intersection enables separation of concerns in type definitions. Each aspect of a domain object (identity, timestamp, permissions, metadata) can be defined independently and combined as needed. This avoids monolithic type definitions and promotes reusability.

### How It Works Internally

The compiler computes the result of `A & B` for object types by taking the union of property keys from both `A` and `B`. For each key present in both, the property type becomes `A[K] & B[K]`. For keys present in only one, the property type is taken as-is.

```typescript
// A = { a: string; b: number }
// B = { b: string; c: boolean }
// A & B = { a: string; b: number & string; c: boolean }
//        = { a: string; b: never; c: boolean }
```

### Syntax

```typescript
// Simple combination
type WithTimestamp = { createdAt: Date; updatedAt: Date };
type WithVersion = { version: number };
type Entity = WithTimestamp & WithVersion;

// Layered combination
type Base = { id: string };
type Auditable = Base & { createdBy: string; modifiedBy: string };
type SoftDeletable = Base & { deletedAt: Date | null };
type FullEntity = Auditable & SoftDeletable;

// Combining with optional properties
type Config = { debug?: boolean };
type ServerConfig = Config & { port: number; host: string };
```

### Beginner Examples

```typescript
interface Nameable { name: string; }
interface Ageable { age: number; }
interface Emailable { email: string; }

type Person = Nameable & Ageable & Emailable;

const employee: Person = {
  name: "Charlie",
  age: 28,
  email: "charlie@company.com",
};

// Component-based type design
type Identifiable = { id: string };
type Timestamped = { createdAt: Date };
type Describable = { description?: string };

type Product = Identifiable & Timestamped & Describable & { price: number };
```

### Intermediate Examples

```typescript
// Combining with generic parameter
function merge<T, U>(a: T, b: U): T & U {
  return { ...a, ...b };
}

const merged = merge({ name: "Alice" }, { age: 30 });
// merged: { name: string } & { age: number }

// Conditional combination
type AdminFields = { role: "admin"; permissions: string[] };
type UserFields = { role: "user"; department: string };
type Account<IsAdmin extends boolean> = 
  IsAdmin extends true ? AdminFields : UserFields;

// Combining interface and type
interface Car { brand: string; model: string; }
type Electric = { batteryRange: number; rechargeTime: number };
type ElectricCar = Car & Electric;

// Combining intersection with mapped types
type Required<T> = { [P in keyof T]-?: T[P] };
type Optional<T> = { [P in keyof T]+?: T[P] };
type PersonRequired = Required<{ name?: string; age?: number }>;
// { name: string; age: number }
```

### Advanced Examples

```typescript
// Deep combination with recursion
type Combine<T, U> = T & U extends infer R
  ? R extends Record<string, unknown>
    ? { [K in keyof R]: R[K] }
    : R
  : never;

// Combining multiple generic types
type RequestShape<Params, Body, Headers> = 
  { params: Params } & { body: Body } & { headers: Headers };

type GetUserRequest = RequestShape<
  { id: string },
  never,
  { authorization: string }
>;

// Intersection with discriminated unions (distributes)
type Shape =
  | ({ kind: "circle" } & { radius: number })
  | ({ kind: "square" } & { side: number })
  | ({ kind: "triangle" } & { base: number; height: number });

// Combining function overloads via intersection
type StringHandler = (x: string) => number;
type NumberHandler = (x: number) => string;
type Overloaded = StringHandler & NumberHandler;
// Equivalent to: (x: string) => number & (x: number) => string
```

### Real-World Use Cases

- **Database models**: Combining base entity fields with specific table fields.
- **API responses**: Wrapping response data with metadata (pagination, errors).
- **UI components**: Combining base component props with theme and styling props.
- **Event systems**: Combining base event fields with type-specific payloads.
- **Plugin architecture**: Merging core types with plugin-provided extensions.

```typescript
// Real-world: Paginated API response
type PaginationMeta = { page: number; pageSize: number; total: number };
type ApiResponse<T> = { data: T } & PaginationMeta;

// Real-world: Form state with metadata
type FormField<T> = { value: T; error?: string; touched: boolean };
type FormState<T> = { [K in keyof T]: FormField<T[K]> } & { isValid: boolean; isDirty: boolean };
```

### Common Mistakes

1. **Forgetting that overlapping properties with incompatible types become `never`**.
2. **Assuming deep merge**: Intersection is shallow—nested objects are intersected recursively only in simple cases.
3. **Using intersection where a union is needed**.
4. **Creating circular references** through self-referencing intersections.

### Best Practices

- Design small, focused interfaces and compose them with `&`.
- Use `type` aliases for intersections; use `interface` for declarations that might be extended.
- Check for property conflicts early by testing your types.
- Prefer `extends` for single-inheritance patterns; use `&` for horizontal composition.

### Performance Considerations

Object type intersection is efficient. The compiler handles property merging quickly. However, intersecting types with hundreds of properties or deeply nested generics can impact compilation speed.

### Interview Questions

1. **Q**: How does TypeScript handle overlapping property types in an intersection?
   **A**: It intersects the property types: if both have `name`, the result is `name: A['name'] & B['name']`. If incompatible, it becomes `never`.

2. **Q**: Can you use intersection with classes?
   **A**: Yes, but class instances have prototypes. Intersection works on the type level, not the runtime level. Using `Object.assign` or spread is the runtime equivalent.

### Coding Challenges

1. Create a type `DeepCombine<T, U>` that recursively merges nested objects.
2. Build an entity factory that composes multiple aspect types (timestamped, auditable, soft-deletable) into a single entity type.

### Related Topics

- Interface merging
- Mixins
- Spread operator
- `Object.assign`

## Property conflicts resolution

### What It Is

When intersecting types share a property name, TypeScript resolves the conflict by intersecting the property types. If both types declare `value: string`, the result is `value: string & string = string`. If they declare `value: string` and `value: number`, the result is `value: string & number = never`.

```typescript
type A = { value: string; x: number };
type B = { value: number; y: number };
type C = A & B; // { value: never; x: number; y: number }
```

### Why It Is Important

Understanding property conflict resolution is crucial for correctly using intersection types. Unintentional conflicts silently produce `never` types, which can make properties unusable. Knowing how conflicts are resolved helps you design types that compose well and avoid subtle bugs.

### How It Works Internally

The compiler's type checker performs a structural merge. For each key `K` present in both operands, the resulting property type is computed as `T[K] & U[K]`. This is not always desirable, especially when the property types are function signatures or complex objects.

```typescript
// For methods: the intersection creates an overloaded type
type A = { method: (x: string) => void };
type B = { method: (x: number) => void };
type C = A & B; // method: ((x: string) => void) & ((x: number) => void)
// This is an overloaded function type
```

### Syntax

```typescript
// Identity conflict
type S1 = { id: string };
type S2 = { id: string };
type Combined = S1 & S2; // { id: string } (string & string = string)

// Type conflict → never
type C1 = { id: string };
type C2 = { id: number };
type Conflict = C1 & C2; // { id: never }

// Method overloading (desirable)
type M1 = { handle: (e: MouseEvent) => void };
type M2 = { handle: (e: KeyboardEvent) => void };
type MCombined = M1 & M2;
// handle: ((e: MouseEvent) => void) & ((e: KeyboardEvent) => void)

// Optional vs required conflict
type R1 = { name: string };
type R2 = { name?: string };
type RCombined = R1 & R2; // { name: string } (string & (string | undefined) = string)
```

### Beginner Examples

```typescript
// Same type, same property
interface A { title: string; }
interface B { title: string; }
type Result = A & B; // { title: string }

// Different types, conflict
interface X { count: number; }
interface Y { count: string; }
type Z = X & Y; // { count: never }

// Avoiding conflict by renaming
interface Person { name: string; }
interface Employee { employeeName: string; }
type Worker = Person & Employee; // No conflict

// Function properties
interface Logger { log: (msg: string) => void; }
interface Profiler { log: (duration: number) => void; }
type LoggerProfiler = Logger & Profiler;
// log: ((msg: string) => void) & ((duration: number) => void)
```

### Intermediate Examples

```typescript
// Deep property conflict
type A = { nested: { value: string } };
type B = { nested: { value: number } };
type C = A & B; // { nested: { value: string & number } } = { nested: { value: never } }

// Partial overlap
type D = { config: { color: string; size: number } };
type E = { config: { color: string; theme: "dark" | "light" } };
type F = D & E;
// { config: { color: string; size: number; theme: "dark" | "light" } }

// Generic property conflict
type Wrapper<T> = { data: T; timestamp: Date };
type SW = Wrapper<string>;
type NW = Wrapper<number>;
type CombinedWrapper = SW & NW; // { data: never; timestamp: Date }

// Resolving via type alias
type Merge<T, U> = Omit<T, keyof U> & U;
type Fixed = Merge<{ id: string }, { id: number }>; // { id: number }
```

### Advanced Examples

```typescript
// Using Omit to resolve conflicts
type Base = { id: string; name: string };
type Override = { id: number };
type Resolved = Omit<Base, "id"> & Override;
// { name: string; id: number }

// Generic conflict resolver
type SafeMerge<T, U> = {
  [K in keyof (T & U)]: K extends keyof T
    ? K extends keyof U
      ? T[K] | U[K]  // Union instead of intersection
      : T[K]
    : K extends keyof U
      ? U[K]
      : never;
};

type Safe = SafeMerge<{ id: string }, { id: number }>;
// { id: string | number }

// Conflict resolution with conditional types
type ResolveConflict<T, U> = {
  [K in keyof (T & U)]: K extends keyof T & keyof U
    ? [T[K], U[K]] extends [object, object]
      ? ResolveConflict<T[K], U[K]>
      : T[K] | U[K]
    : K extends keyof T
      ? T[K]
      : U[K];
};
```

### Real-World Use Cases

- **Versioned API clients**: When API versions change property types, use `Omit` & intersection to override.
- **Plugin systems**: Plugins may override base properties with more specific types.
- **Configuration merging**: User config overrides default config, requiring conflict resolution.
- **ORM entity extension**: Adding custom fields to base entity definitions.

```typescript
// Real-world: Default config merging
type DefaultConfig = { port: number; host: string; protocol: "http" | "https" };
type UserConfig = { port: string };
// Conflict! Resolve:
type MergedConfig = Omit<DefaultConfig, "port"> & { port: string | number };
```

### Common Mistakes

1. **Not realizing conflicts exist** until a property becomes `never`.
2. **Assuming conflict resolution creates a union**—it creates an intersection.
3. **Not using `Omit` to exclude conflicting properties** before intersecting.
4. **Relying on deep merge** for nested conflicts.

### Best Practices

- When you know conflicts will occur, use `Omit<T, keyof U> & U` to let `U` win.
- Use `SafeMerge` or a utility type that unions conflicting properties.
- Test your intersection types with the `never` check.
- Avoid intersecting types with overlapping property names unless you intend the intersection behavior.

### Performance Considerations

The conflict resolution itself is fast. However, deeply nested conflicts in large types can increase the compiler's work. Using `Omit` to resolve conflicts adds a minor overhead.

### Interview Questions

1. **Q**: What happens if two intersected interfaces have a property with the same name but different types?
   **A**: The resulting property type is `T[K] & U[K]`. If incompatible, it becomes `never`.

2. **Q**: How do you override a property in an intersection type?
   **A**: Use `Omit<T, keyof U> & U` to exclude the conflicting property from `T` before intersecting.

### Coding Challenges

1. Implement a `Override<T, U>` type where properties in `U` override those in `T`.
2. Create a type that detects property conflicts in an intersection and reports them.

### Related Topics

- `Omit` utility type
- `never` type
- Type merging
- Declaration merging

## Intersection vs inheritance

### What It Is

TypeScript provides two main mechanisms for combining types: intersection types (`&`) and inheritance (`extends`). Both allow you to build composite types, but they have different characteristics, capabilities, and use cases. Intersection types work with any type; inheritance works only with interfaces and classes.

```typescript
// Intersection
type Admin = Person & { role: string; permissions: string[] };

// Inheritance
interface Admin extends Person {
  role: string;
  permissions: string[];
}
```

### Why It Is Important

Choosing between intersection and inheritance affects code readability, type-relationship clarity, and extensibility. Understanding the trade-offs helps you design type systems that are maintainable, expressive, and idiomatic.

### How It Works Internally

The compiler handles both mechanisms similarly at the type level—both produce merged property sets. However, interfaces with `extends` support declaration merging (multiple declarations of the same interface are automatically merged), which intersection types do not. Interfaces also preserve the declared structure for display in IDE tooltips, while intersection types can produce more complex-looking types.

```typescript
// Declaration merging (interface only)
interface User { name: string; }
interface User { age: number; } // Auto-merged

// Intersection type cannot be re-opened
type User = { name: string; };
// Cannot redeclare User
```

### Syntax

```typescript
// Inheritance with interface
interface Animal { name: string; }
interface Dog extends Animal { breed: string; }

// Multiple inheritance
interface Mammal { warmBlooded: boolean; }
interface Pet { owner: string; }
interface Dog extends Mammal, Pet { breed: string; }

// Intersection equivalent
type Animal = { name: string };
type Dog = Animal & { breed: string };
type Mammal = { warmBlooded: boolean };
type PetType = { owner: string };
type DogType = Mammal & PetType & { breed: string };

// Class inheritance
class BaseEntity { id: string = ""; }
class User extends BaseEntity { name: string = ""; }

// Intersection with class type
type UserType = BaseEntity & { name: string };
```

### Beginner Examples

```typescript
// Interface inheritance
interface Shape {
  color: string;
  area(): number;
}
interface Circle extends Shape {
  radius: number;
}

// Type intersection equivalent
type ShapeType = {
  color: string;
  area(): number;
};
type CircleType = ShapeType & { radius: number };

// Both produce similar types
const c1: Circle = { color: "red", radius: 5, area: () => Math.PI * 25 };
const c2: CircleType = { color: "blue", radius: 3, area: () => Math.PI * 9 };

// Readability difference
interface Employee extends Person {
  jobTitle: string;
  salary: number;
}
type EmployeeType = Person & { jobTitle: string; salary: number };
```

### Intermediate Examples

```typescript
// Intersection with complex types (not possible with extends)
type MappedType<T> = { [K in keyof T]: T[K] };
type Conditional<T> = T extends string ? { value: T } : { value: number };

// This works:
type Combined = MappedType<{ a: string }> & Conditional<string>;
// This does NOT work (interface cannot extend complex types):
// interface Bad extends Conditional<string> { }
// Error: An interface can only extend an object type or intersection of object types

// Intersection allows mixing with primitives
type StringOrNumber = string | number;
// Can't "extend" a union with interface

// Intersection with tuple types
type Point = [number, number];
type NamedPoint = Point & { name: string };
// { 0: number; 1: number; name: string; length: 2 }

// extends with generic constraints
function identity<T extends { length: number }>(arg: T): T {
  return arg;
}
```

### Advanced Examples

```typescript
// extends constraint on generic parameters
function merge<T extends object, U extends object>(a: T, b: U): T & U {
  return { ...a, ...b };
}

// Intersection used inside extends
interface WithTimestamps {
  createdAt: Date;
  updatedAt: Date;
}
interface Product extends WithTimestamps {
  name: string;
  price: number;
}

// Combining extends with intersection for advanced patterns
interface AdminPermissions {
  canDelete: boolean;
  canBan: boolean;
}
type FullAdmin = Product & AdminPermissions;
class AdminProduct implements FullAdmin {
  name = "";
  price = 0;
  createdAt = new Date();
  updatedAt = new Date();
  canDelete = true;
  canBan = true;
}

// Recursive inheritance vs intersection
interface TreeNode<T> {
  value: T;
  children: TreeNode<T>[];
}
type TreeNodeType<T> = { value: T; children: TreeNodeType<T>[] };
```

### Real-World Use Cases

- **Use `extends`**: When modeling "is-a" relationships (e.g., `Circle extends Shape`).
- **Use `&`**: When modeling "has-a" relationships or mixing concerns (e.g., `Timestamped & Versioned & Auditable`).
- **Use `extends`**: When you want declaration merging and re-opening interfaces.
- **Use `&`**: When you need to combine types that include mapped types, conditional types, or type aliases.

```typescript
// Real-world: extends for hierarchy
interface Entity { id: string; }
interface User extends Entity { name: string; }
interface AdminUser extends User { permissions: string[]; }

// Real-world: intersection for cross-cutting concerns
type Loggable = { log: (msg: string) => void };
type Serializable = { toJSON: () => object };
type Cacheable = { cacheKey: string; ttl: number };
type Service = Loggable & Serializable & Cacheable & { execute(): Promise<void> };
```

### Common Mistakes

1. **Using intersection where inheritance is semantically clearer** (e.g., `Cat & Animal` vs `Cat extends Animal`).
2. **Trying to `extends` a type alias that isn't an object type**.
3. **Assuming intersection and inheritance are interchangeable**—interfaces support declaration merging, types do not.
4. **Forgetting that `extends` in generics is a constraint, not an intersection**.

### Best Practices

- Use `interface extends` for type hierarchies and is-a relationships.
- Use `type &` for composition, mixins, and combining independent concerns.
- Use `interface` when you need declaration merging (e.g., extending global types).
- Use `type` when working with mapped types, conditional types, or unions.
- Prefer `interface` for public API types for better display in tooltips and documentation.

### Performance Considerations

Both mechanisms have similar performance characteristics. Interfaces with `extends` are slightly more efficient because the compiler can cache and reuse interface types more effectively than intersection types. For very large type compositions, prefer interfaces.

### Interview Questions

1. **Q**: What are the key differences between `interface extends` and `type &`?
   **A**: Interfaces support declaration merging; types don't. Interfaces can only extend object types; intersection can combine any types. Interfaces are preferred for public API types; intersection is more flexible.

2. **Q**: Can you extend a type alias with an interface?
   **A**: Yes, if the type alias is an object type: `interface Foo extends Bar {}` where `Bar` is a type alias for an object type. No, if the alias is anything else (union, intersection of non-objects, etc.).

3. **Q**: When would you choose intersection over inheritance?
   **A**: When combining types from different modules, when working with mapped/conditional types, or when you need to override specific properties.

### Coding Challenges

1. Refactor a deep class hierarchy into a composition of intersection types.
2. Create a type that uses both `extends` constraints and `&` intersection to model a complex domain.

### Related Topics

- Interface extends
- Type aliases
- Declaration merging
- Mixins
- Structural typing
