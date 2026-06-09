# Type Alias - type keyword, union/intersection in aliases, generic aliases, vs interfaces

## Introduction

Type aliases are a powerful feature that allows creating new names for existing types. They can represent primitive types, unions, intersections, tuples, functions, and generic types. Type aliases are a cornerstone of TypeScript's type composition capabilities, enabling expressive and reusable type definitions.

## type keyword

### What It It

The 	ype keyword creates an alias for any type — primitives, unions, intersections, objects, functions, tuples, or generics. Type aliases don't create new types; they provide a name for an existing type.

### Why It Is Important

Type aliases make complex type expressions readable and reusable. They reduce duplication by defining types once and referencing them by name. They also enable type composition patterns (unions, intersections) that aren't possible with interfaces.

### How It Works Internally

Type aliases are purely a compile-time construct. They are resolved by name substitution — every reference to a type alias is replaced with its definition during type checking. TypeScript evaluates aliases lazily, resolving them only when needed.

### Syntax

`	ypescript
// Primitive alias
type MyString = string;
type Age = number;
type IsActive = boolean;

// Union alias
type Status = "active" | "inactive" | "pending";

// Object alias
type Point2 = {
  x: number;
  y: number;
};

// Function alias
type Callback = (error: Error | null, result?: string) => void;

// Tuple alias
type Pair2 = [string, number];

// Generic alias
type Container<T> = { value: T };
`

### Beginner Examples

`	ypescript
// Type aliases simplify complex types
type UserId = string | number;
type UserRole = "admin" | "editor" | "viewer";

function getUser(id: UserId): void {
  console.log(\Looking up user: \\);
}

function checkAccess(role: UserRole): boolean {
  return role === "admin" || role === "editor";
}

getUser(123);
getUser("user_456");
console.log(checkAccess("admin"));

// Object type alias
type Product3 = {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
};

function formatProduct(p: Product3): string {
  return \\: \$\\;
}
`

### Intermediate Examples

`	ypescript
// Function type alias
type CompareFn<T> = (a: T, b: T) => number;

function sortBy<T>(items: T[], compare: CompareFn<T>): T[] {
  return [...items].sort(compare);
}

// Union of complex types
type AsyncResult<T> =
  | { status: "pending" }
  | { status: "loading"; progress: number }
  | { status: "success"; data: T }
  | { status: "error"; error: string };

// Type alias with template literals
type CSSClass = is-;
type EventName2 = on;

// Alias for complex nested type
type ApiResponse<T> = {
  data: T;
  meta: {
    page: number;
    total: number;
    hasMore: boolean;
  };
};
`

### Advanced Examples

`	ypescript
// Recursive type alias
type JSONValue2 =
  | string
  | number
  | boolean
  | null
  | JSONValue2[]
  | { [key: string]: JSONValue2 };

function stringify(value: JSONValue2): string {
  return JSON.stringify(value);
}

// Conditional type alias
type IsArray<T> = T extends any[] ? true : false;

// Mapped type alias
type Readonly6<T> = {
  readonly [P in keyof T]: T[P];
};

type Optional3<T> = {
  [P in keyof T]?: T[P];
};

// Template literal type alias with inference
type ExtractId<T extends string> = T extends id- ? Id : never;
type UserId2 = ExtractId<"id-42">; // "42"

// Distribution in conditional type alias
type ToArray2<T> = T extends unknown ? T[] : never;
type Result3 = ToArray2<string | number>; // string[] | number[]
`

### Real-World Use Cases

**Domain Modeling**: Creating expressive type aliases for domain concepts.

**API Types**: Type aliases for API request/response shapes.

**Utility Types**: Reusable type transformations.

**Configuration Types**: Complex configuration object types.

### Common Mistakes

**Using type When interface Would Be Better**: For object shapes in public APIs, interfaces provide better error messages and declaration merging.

**Type Alias Name Collisions**: Type aliases can't be redeclared in the same scope.

**Circular References**: Type aliases can be self-referential only for objects and arrays, not primitives.

### Best Practices

1. Use 	ype for unions, intersections, and primitive aliases
2. Use 	ype for utility types and mapped types
3. Use descriptive names for aliases
4. Prefer interface for object types in public APIs
5. Export type aliases from modules for reuse

### Performance Considerations

Type aliases add minimal compilation overhead. Complex conditional types can be expensive — consider simplifying or breaking into smaller aliases.

### Interview Questions

1. What is the difference between 	ype and interface?
2. Can you use 	ype for primitive types?
3. How do recursive type aliases work?

### Coding Challenges

1. Create a type alias for a recursive JSON value
2. Implement a conditional type alias that extracts promise values

### Related Topics

- Interfaces
- Union Types
- Intersection Types
- Mapped Types

## Union and intersection aliases

### What It Is

Union types (|) represent a value that can be one of several types. Intersection types (&) combine multiple types into one. Type aliases can name unions and intersections, enabling reusable type composition.

### Why It Is Important

Unions and intersections are TypeScript's primary type composition tools. Unions model alternatives (this OR that), while intersections model combinations (this AND that). They're essential for discriminated unions, function overloading, mixins, and type-safe state management.

### How It Works Internally

Union types are checked mutually — a value is assignable to a union if it's assignable to at least one member. Intersection types require the value to satisfy all constituent types. TypeScript distributes conditional types over unions and performs excess property checking on intersections.

### Syntax

`	ypescript
// Union type alias
type StringOrNumber = string | number;
type Status2 = "success" | "error" | "loading";
type Nullable3<T> = T | null;

// Intersection type alias
type Named = { name: string };
type Aged = { age: number };
type Person4 = Named & Aged;

// Complex composition
type Employee2 = Person4 & {
  id: number;
  role: string;
};

// Union of object types
type Shape3 =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "triangle"; base: number; height: number };
`

### Beginner Examples

`	ypescript
// Union types
function formatId2(id: string | number): string {
  return \ID: \\;
}

formatId2(123);
formatId2("abc");

// Intersection types
interface HasName2 { name: string; }
interface HasEmail { email: string; }

type Contact = HasName2 & HasEmail;

const contact: Contact = {
  name: "Alice",
  email: "alice@example.com",
};

// Union with literals
type Direction3 = "north" | "south" | "east" | "west";

function move2(dir: Direction3): void {
  console.log(\Moving \\);
}
`

### Intermediate Examples

`	ypescript
// Discriminated union
type Result4<T> =
  | { kind: "ok"; value: T }
  | { kind: "err"; message: string }
  | { kind: "loading"; progress?: number };

function handleResult<T>(result: Result4<T>): string {
  switch (result.kind) {
    case "ok": return \Success: \\;
    case "err": return \Error: \\;
    case "loading": return \Loading... \%\;
  }
}

// Intersection with function types
type Loggable = { log: (msg: string) => void };
type Serializable = { toJSON: () => string };

type Logger = Loggable & Serializable;

const logger: Logger = {
  log(msg: string) { console.log(msg); },
  toJSON() { return JSON.stringify({}); },
};

// Union narrowing with type guards
type Input = string | string[] | null;

function getLength(input: Input): number {
  if (input === null) return 0;
  if (Array.isArray(input)) return input.length;
  return input.length;
}
`

### Advanced Examples

`	ypescript
// Branded union types
type Brand2<K, T> = K & { __brand: T };
type USD2 = Brand2<number, "USD">;
type EUR2 = Brand2<number, "EUR">;

function createUSD(amount: number): USD2 {
  return amount as USD2;
}

function createEUR(amount: number): EUR2 {
  return amount as EUR2;
}

function addMoney(a: USD2, b: USD2): USD2 {
  return (a + b) as USD2;
}

// Distributive conditional with union
type ExtractNames<T> = T extends { name: infer N } ? N : never;
type Names = ExtractNames<{ name: string } | { name: number } | { title: string }>;
// string | number

// Complex intersection with generics
type WithId<T> = T & { id: string };
type WithTimestamps<T> = T & { createdAt: Date; updatedAt: Date };
type Persisted<T> = WithId<WithTimestamps<T>>;

type User5 = Persisted<{
  name: string;
  email: string;
}>;

// Union of tuple types
type Event3 =
  | ["user:created", { id: string; name: string }]
  | ["user:deleted", { id: string }]
  | ["order:placed", { orderId: string; total: number }];

function handleEvent3(event: Event3): void {
  const [type, data] = event;
  switch (type) {
    case "user:created":
      console.log(\Created user \\);
      break;
    case "user:deleted":
      console.log(\Deleted user \\);
      break;
    case "order:placed":
      console.log(\Order \ for \$\\);
      break;
  }
}
`

### Real-World Use Cases

**State Management**: Discriminated unions for application state.

**API Responses**: Union of success/error/loading states.

**Form Validation**: Union of valid/invalid states with error details.

**Event Systems**: Discriminated union of event types with typed payloads.

### Common Mistakes

**Intersection with Conflicts**: Intersecting types with conflicting properties results in 
ever.

**Not Narrowing Unions**: Using a union type without narrowing restricts available operations.

**Excess Properties with Intersections**: Intersection of object literals may have unexpected excess property checking.

### Best Practices

1. Use discriminated unions for state machines
2. Narrow unions with type guards before accessing properties
3. Avoid intersections when types have conflicting properties
4. Use branded types for type-safe IDs and currencies
5. Prefer union over overloads for simple type variations

### Performance Considerations

Unions and intersections are compile-time constructs with no runtime cost. Large unions (100+ members) can slow type checking.

### Interview Questions

1. What is a discriminated union and when would you use it?
2. How does intersection resolve conflicting property types?
3. How do conditional types distribute over unions?

### Coding Challenges

1. Create a branded type for currency conversion
2. Implement a discriminated union for a multistep form state

### Related Topics

- Type Guards
- Discriminated Unions
- Branded Types

## Generic type aliases

### What It It

Generic type aliases are type aliases that accept type parameters. They enable reusable type definitions that work with any type, adapting their structure based on the provided type arguments.

### Why It Is Important

Generics eliminate type duplication by allowing a single type definition to work with many types. They are essential for utility types (Partial, Readonly, Pick, etc.), container types (Box, Result, Maybe), and type-safe data structures.

### How It Works Internally

Generic type aliases create parameterized types. When a type argument is supplied, TypeScript substitutes the parameter throughout the alias definition. Generic aliases support constraints (extends), default types, and conditional evaluation.

### Syntax

`	ypescript
// Simple generic alias
type Box2<T> = {
  value: T;
};

const numberBox: Box2<number> = { value: 42 };
const stringBox: Box2<string> = { value: "hello" };

// Generic with constraint
type HasLength2<T extends { length: number }> = T;

// Generic with default
type Result5<T, E = Error> = { ok: true; value: T } | { ok: false; error: E };

// Multiple type parameters
type Pair3<T, U> = { first: T; second: U };

// Generic conditional
type IsString2<T> = T extends string ? "yes" : "no";
`

### Beginner Examples

`	ypescript
// Generic container
type Container2<T> = {
  value: T;
  isEmpty: boolean;
};

const numContainer: Container2<number> = { value: 42, isEmpty: false };
const strContainer2: Container2<string> = { value: "hello", isEmpty: false };

// Generic function type
type Transformer<T, U> = (input: T) => U;

const toString: Transformer<number, string> = (n) => n.toString();

// Generic response type
type ApiResponse4<T> = {
  success: boolean;
  data: T;
  message?: string;
};

const userResponse: ApiResponse4<{ id: number; name: string }> = {
  success: true,
  data: { id: 1, name: "Alice" },
};
`

### Intermediate Examples

`	ypescript
// Generic with constraint
type Identifiable<T extends { id: string | number }> = T & {
  createdAt: Date;
};

type User6 = Identifiable<{ id: number; name: string }>;

// Generic utility types
type Nullable4<T> = T | null;
type Optional4<T> = T | undefined;
type Nullish4<T> = T | null | undefined;

// Generic mapper
type Mapped<T, V> = {
  [P in keyof T]: V;
};

type NumberProps<T> = Mapped<T, number>;

// Generic with keyof constraint
type Pluck<T, K extends keyof T> = T[K];

interface User7 {
  id: number;
  name: string;
  email: string;
}

type UserName2 = Pluck<User7, "name">; // string
type UserId2 = Pluck<User7, "id">; // number
`

### Advanced Examples

`	ypescript
// Recursive generic type
type DeepPartial2<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial2<T[P]> : T[P];
};

// Generic conditional
type IfEquals<T, U, Y, N> =
  (<G>() => G extends T ? 1 : 2) extends <G>() => G extends U ? 1 : 2 ? Y : N;

type IsExact<T, U> = IfEquals<T, U, true, false>;

// Filter properties by type
type PickByType<T, V> = {
  [P in keyof T as T[P] extends V ? P : never]: T[P];
};

interface Mixed {
  name: string;
  age: number;
  email: string;
  id: number;
}

type StringProps = PickByType<Mixed, string>; // { name: string; email: string }

// Generic with variadic tuples
type TuplePush<T extends readonly unknown[], V> = [...T, V];
type TuplePop<T extends readonly unknown[]> =
  T extends readonly [...infer U, any] ? U : never;

type Original2 = readonly [1, 2, 3];
type Pushed = TuplePush<Original2, 4>; // [1, 2, 3, 4]
type Popped = TuplePop<Original2>; // [1, 2]

// Generic template literal
type CreateApiPath<Resource extends string, Version extends number> =
  \/api/v\/\\;

type UsersPath = CreateApiPath<"users", 2>; // "/api/v2/users"

// Constrained generic with conditional
type Arrayify<T> = T extends any[] ? T : T[];
type TestArray = Arrayify<string>; // string[]
type TestArray2 = Arrayify<number[]>; // number[]
`

### Real-World Use Cases

**Repository Pattern**: Generic repository types for database operations.

**State Management**: Generic state and action types for state managers.

**API Clients**: Generic HTTP client with typed request/response types.

**Form State**: Generic form field types with validation.

### Common Mistakes

**Missing Constraints**: Generic types without constraints accept any type — add extends for safety.

**Overly Complex Generics**: Multiple nested generics reduce readability.

**Using ny Instead of Generic**: Prefer generic over ny for type-safe reusable code.

### Best Practices

1. Use descriptive type parameter names (e.g., TItem not T)
2. Add constraints with extends for type safety
3. Provide defaults for type parameters when sensible
4. Keep generics simple — extract complex logic into helper types
5. Use generics for reusable utilities, not every type definition

### Performance Considerations

Generic type alias evaluation adds compilation overhead, especially for deep conditional types. Consider caching complex types with helper aliases.

### Interview Questions

1. How do generic type constraints work?
2. What is the difference between generic type alias and generic interface?
3. How do default type parameters work?

### Coding Challenges

1. Implement PickByType<T, V> utility type
2. Create a generic ApiResponse<T> with pagination support
3. Implement a generic TupleToUnion<T> type

### Related Topics

- Generic Interfaces
- Generic Functions
- Conditional Types
- Template Literal Types

## type vs interface

### What It It

Both 	ype and interface can define object shapes, but they have important differences. Interfaces support declaration merging and can only describe objects. Type aliases can represent any type and support mapped types, conditional types, and other advanced features.

### Why It Is Important

Choosing between 	ype and interface affects code readability, extensibility, and maintainability. The decision often comes down to whether you need declaration merging (use interface) or type composition features (use 	ype). Understanding the differences helps teams establish consistent conventions.

### How It Works Internally

Both create structural types — the runtime is identical. The key internal differences are:

- Interfaces create a named type that can be augmented through declaration merging
- Type aliases create a name for an existing type without creating a new type
- Interfaces can be extendsed or implementsed in classes
- Type aliases handle mapped types, conditional types, and other advanced patterns

### Syntax

`	ypescript
// Interface (object only)
interface Person8 {
  name: string;
  age: number;
}

// Type alias (any type)
type Person9 = {
  name: string;
  age: number;
};

// Interface with extends
interface Employee3 extends Person8 {
  role: string;
}

// Type with intersection
type Employee4 = Person8 & { role: string };

// Interface merging
interface Person8 {
  email?: string;
}
// Person8 now has name, age, and email

// Type union (not possible with interface)
type Status3 = "active" | "inactive" | "pending";

// Type mapped (not possible with interface)
type Readonly7<T> = {
  readonly [P in keyof T]: T[P];
};
`

### Beginner Examples

`	ypescript
// Interface — preferred for objects
interface Animal {
  name: string;
  species: string;
  age: number;
}

// Type — preferred for unions
type AnimalKind = "mammal" | "bird" | "reptile" | "fish";

// Both can describe functions
interface GreetFn {
  (name: string): string;
}

type GreetFn2 = (name: string) => string;

// Interface supports extension
interface Dog extends Animal {
  breed: string;
}

// Type supports intersection
type Cat = Animal & { furColor: string };

// Interface supports declaration merging
interface Animal {
  isDomestic?: boolean;
}
// Animal now has name, species, age, and optional isDomestic
`

### Intermediate Examples

`	ypescript
// Interface for class implementation
interface Serializable2 {
  serialize(): string;
}

class User8 implements Serializable2 {
  constructor(public name: string, public age: number) {}
  serialize(): string {
    return JSON.stringify({ name: this.name, age: this.age });
  }
}

// Type for complex unions
type RequestState<T> =
  | { type: "idle" }
  | { type: "loading" }
  | { type: "success"; data: T }
  | { type: "error"; error: string };

// Interface for public API — better error messages
interface UserService {
  getUser(id: string): Promise<User8>;
  createUser(data: Omit<User8, "id">): Promise<User8>;
  updateUser(id: string, data: Partial<User8>): Promise<User8>;
}

// Type for internal utilities
type DeepPartial3<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial3<T[P]> : T[P];
};

// Interface with generic
interface ApiResponse5<T> {
  data: T;
  status: number;
  message: string;
}

// Type with generic and constraints
type ApiResult<T extends { id: string | number }> = {
  data: T;
  links: {
    self: string;
    related?: string;
  };
};
`

### Advanced Examples

`	ypescript
// Interface can extend from types
type Named2 = { name: string };
interface Employee5 extends Named2 {
  role: string;
}

// Type can intersect interfaces
interface HasId { id: string; }
interface HasTimestamp { createdAt: Date; }
type Entity = HasId & HasTimestamp;

// Conditional types only work with type aliases
type IsArray2<T> = T extends any[] ? true : false;

// Mapped types only work with type aliases
type Getters3<T> = {
  [P in keyof T as \get\\]: () => T[P];
};

// Interface can be used in ambient declarations
declare global {
  interface Window {
    myApp: {
      version: string;
      config: Record<string, unknown>;
    };
  }
}

// Type alias for template literal
type ColorName2 = "red" | "green" | "blue";
type HexColor2 = Record<ColorName2, string>;

// Interface with call and construct signatures
interface CallableConstructable {
  (name: string): string;
  new (name: string): { name: string };
}
`

### Real-World Use Cases

**Public Library API**: Use interfaces for exported types — they support declaration merging and give better error messages.

**Internal Utility Types**: Use type aliases for mapped types, conditional types, and type transformations.

**Class Contracts**: Use interfaces for implements — classes can implement multiple interfaces.

**State Management**: Use discriminated unions (type aliases) for state types.

### Common Mistakes

**Using type When interface Works Better**: For objects in public APIs, interfaces are preferred.

**Assuming type Can Be Merged**: Type aliases cannot be redeclared or merged.

**Not Knowing the Differences**: Teams should agree on conventions — mixing arbitrarily causes confusion.

### Best Practices

1. Prefer interface for public API object types
2. Prefer 	ype for unions, intersections, mapped types, and conditional types
3. Use interface for declaration merging (augmenting external types)
4. Use 	ype for functions and primitives
5. Be consistent across the codebase

### Performance Considerations

Interfaces generally provide slightly better compiler performance than type aliases for object types. Type aliases with complex conditional types can be slower. For most use cases, the difference is negligible.

### Interview Questions

1. What are the key differences between type and interface?
2. Can you use type and interface interchangeably?
3. When would you choose interface over type?
4. What is declaration merging and which one supports it?

### Coding Challenges

1. Refactor a codebase that uses type for objects to use interface where appropriate
2. Create a scenario where you MUST use interface (declaration merging)
3. Create a scenario where you MUST use type (mapped types, conditional types)

### Related Topics

- Declaration Merging
- Mapped Types
- Conditional Types
- Interfaces Deep Dive
