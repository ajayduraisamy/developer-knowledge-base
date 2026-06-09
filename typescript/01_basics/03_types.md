# Types - Primitive types, any/unknown/never/void, type inference, type annotations

## Introduction

TypeScript's type system is built on a foundation of primitive types and special types that control how the compiler checks and infers types. Understanding these fundamentals is crucial before moving to more advanced concepts. This guide covers the primitive types (`string`, `number`, `boolean`), the special types (`any`, `unknown`, `never`, `void`), and the mechanisms of type inference and explicit type annotations.

## Primitive types (string, number, boolean)

### What It Is

TypeScript supports the three primitive types that correspond to JavaScript primitives: `string` for textual data, `number` for numeric values, and `boolean` for true/false values. These types form the basic building blocks of TypeScript programs.

### Why It Is Important

Primitive types are the most commonly used types in any codebase. Correctly understanding how TypeScript handles them — including literal types, type widening, and coercion — prevents entire categories of runtime errors related to value manipulation.

### How It Works Internally

TypeScript's primitive types correspond directly to JavaScript's runtime primitives. The type checker validates operations on these types: arithmetic on numbers, string concatenation, boolean logic. TypeScript also supports literal types, where a specific value (like `"hello"` or `42`) becomes a type. The compiler uses type widening to infer wider types from literal values in mutable contexts.

```typescript
// Literal types
let greeting: "hello" = "hello"; // Only "hello" allowed

// Type widening
let x = "hello"; // Type inferred as string (widened), not "hello"
const y = "hello"; // Type inferred as "hello" (literal, not widened)
```

### Syntax

```typescript
// Explicit primitive type annotations
let username: string = "Alice";
let age: number = 30;
let isActive: boolean = true;

// Literal types
let specificValue: 42 = 42;
let specificString: "ready" = "ready";

// TypeScript also supports:
// bigint (ES2020+)
let big: bigint = 100n;

// symbol
let sym: symbol = Symbol("unique");
```

### Beginner Examples

```typescript
// String operations
let firstName: string = "John";
let lastName: string = "Doe";
let fullName: string = firstName + " " + lastName;
let greeting: string = `Hello, ${fullName}!`;

// Number operations
let count: number = 42;
let price: number = 19.99;
let total: number = count * price;
let negative: number = -10;
let hex: number = 0xff; // 255
let binary: number = 0b1010; // 10
let octal: number = 0o744; // 484

// Boolean operations
let isLoggedIn: boolean = true;
let hasAccess: boolean = isLoggedIn && age >= 18;
let canProceed: boolean = isLoggedIn || hasAccess;
```

### Intermediate Examples

```typescript
// String literal union types
type Direction = "north" | "south" | "east" | "west";
function move(direction: Direction): void {
  console.log(`Moving ${direction}`);
}
move("north"); // OK
// move("up"); // Error: "up" not assignable to Direction

// Numeric literal types
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;
function rollDice(): DiceRoll {
  return (Math.floor(Math.random() * 6) + 1) as DiceRoll;
}

// Template literal types with primitives
type EventName = `on${Capitalize<string>}`;
type ClickHandler = `onClick`;
type ChangeHandler = `onChange`;

// Type narrowing with primitives
function process(value: string | number): string | number {
  if (typeof value === "string") {
    return value.toUpperCase(); // TypeScript knows value is string
  }
  return value.toFixed(2); // TypeScript knows value is number
}
```

### Advanced Examples

```typescript
// Branded primitive types
type Brand<K, T> = K & { __brand: T };
type UserId = Brand<string, "UserId">;
type OrderId = Brand<string, "OrderId">;

function createUserId(id: string): UserId {
  return id as UserId;
}

function createOrderId(id: string): OrderId {
  return id as OrderId;
}

function getUser(userId: UserId): void {
  console.log(`Getting user ${userId}`);
}

const uid = createUserId("user_123");
const oid = createOrderId("order_456");
getUser(uid);
// getUser(oid); // Error: OrderId not assignable to UserId

// Primitive type manipulation via conditional types
type IsString<T> = T extends string ? "yes" : "no";
type Test1 = IsString<"hello">; // "yes"
type Test2 = IsString<42>; // "no"

// Template literal type inference from primitives
type ExtractNumber<T extends string> =
  T extends `${infer Num}` ? (Num extends `${number}` ? Num : never) : never;

type Parsed = ExtractNumber<"123">; // "123"

// Numeric range types (advanced)
type Enumerate<N extends number, Acc extends number[] = []> =
  Acc["length"] extends N
    ? Acc[number]
    : Enumerate<N, [...Acc, Acc["length"]]>;

type Range<F extends number, T extends number> = Exclude<Enumerate<T>, Enumerate<F>>;
type DayOfMonth = Range<1, 32>; // 1 | 2 | ... | 31
```

### Real-World Use Cases

**Form Validation**: String literal types for field names, boolean for dirty/touched state tracking.

**Configuration Objects**: String literal unions for allowed configuration values.

**API Response Handling**: Type-safe extraction of values from API responses using primitive type guards.

### Common Mistakes

**Confusing `number` and `Number`**: The lowercase `number` is the TypeScript type; `Number` is the JavaScript object wrapper. Always use lowercase.

**Assuming `string`, `number`, `boolean` Are Objects**: These are primitives and don't have methods unless auto-boxed by JavaScript.

**Overly Wide Literal Types**: Using `let` for values that should be `const` widens literal types unnecessarily.

### Best Practices

1. Prefer `const` for literal values to get narrower types
2. Use template literal types for type-safe string formatting
3. Leverage brand types for type-safe IDs
4. Avoid `Number`, `String`, `Boolean` constructor types
5. Use `typeof` checks for primitive type narrowing

### Performance Considerations

Primitive type annotations are erased at compile time — zero runtime cost. Branded types using intersection also erase at compile time. Template literal types can be complex for the compiler but are resolved at compile time only.

### Interview Questions

1. What are literal types and how do they differ from primitive types?
2. How does TypeScript widen types and when does it not widen?
3. What is the difference between `string` and `String` in TypeScript?
4. How do you create a type that represents a specific number range?
5. What are branded types and why are they useful?

### Coding Challenges

1. Create a `DayOfWeek` type using string literals
2. Implement a type-safe `parseInt` function
3. Create a branded type for `Email` that validates at the type level
4. Implement a `StringToUnion<T extends string>` type that splits a string into a union of characters

### Related Topics

- TypeScript Handbook — Basic Types
- Literal Types
- Type Widening and Narrowing
- Template Literal Types

## any and unknown

### What It Is

`any` and `unknown` are TypeScript's top types — every other type is assignable to them. However, they serve opposite purposes: `any` opts out of type checking entirely, while `unknown` requires type checking before use.

### Why It Is Important

`any` is a dangerous escape hatch that disables type checking. `unknown` provides a type-safe alternative for values whose type is genuinely unknown, forcing type narrowing before use. Using `unknown` over `any` is a key best practice for maintaining type safety.

### How It Works Internally

- `any`: Disables all type checking on the value. No property access, method call, or assignment is validated. `any` propagates — any expression involving `any` becomes `any`.
- `unknown`: Represents a value that could be anything. Unlike `any`, you cannot perform any operation on an `unknown` value without first narrowing its type (via `typeof`, `instanceof`, type guards, etc.).

```typescript
declare const value: any;
value.foo.bar.baz(); // No error — all type checking disabled

declare const value2: unknown;
// value2.foo; // Error: Object is of type 'unknown'
```

### Syntax

```typescript
// any — disables type checking
let flexible: any = 42;
flexible = "hello";
flexible = true;
flexible.doSomething(); // No type checking
flexible[100]; // No type checking

// unknown — requires narrowing
let safe: unknown = 42;
safe = "hello";

if (typeof safe === "string") {
  console.log(safe.toUpperCase()); // OK — narrowed to string
}
```

### Beginner Examples

```typescript
// When migrating JS to TS, any helps incrementally
function migrateMe(data: any): any {
  // Temporarily any during migration
  return data.process();
}

// unknown for parse results
function parseJSON(json: string): unknown {
  return JSON.parse(json);
}

const result = parseJSON('{"name":"Alice"}');
// result.name; // Error: Object is of type 'unknown'

if (typeof result === "object" && result !== null) {
  // Still need to narrow further
  if ("name" in result) {
    console.log((result as { name: string }).name);
  }
}
```

### Intermediate Examples

```typescript
// Type-safe JSON.parse wrapper
function safeParse<T>(json: string): { success: true; data: T } | { success: false; error: string } {
  try {
    const parsed: unknown = JSON.parse(json);
    return { success: true, data: parsed as T };
  } catch (e) {
    return { success: false, error: (e as Error).message };
  }
}

// Generic unknown handler
function handleUnknown(value: unknown): string {
  if (value === null) return "null";
  if (value === undefined) return "undefined";

  switch (typeof value) {
    case "string":
      return `String: ${value}`;
    case "number":
      return `Number: ${value}`;
    case "boolean":
      return `Boolean: ${value}`;
    case "object":
      return `Object: ${JSON.stringify(value)}`;
    case "function":
      return `Function: ${value.name}`;
    default:
      return `Unknown: ${String(value)}`;
  }
}

// API response handler
interface ApiResponse {
  status: number;
  data: unknown;
}

function processResponse(response: ApiResponse): void {
  if (response.status >= 200 && response.status < 300) {
    if (Array.isArray(response.data)) {
      // Handle array
    } else if (typeof response.data === "object" && response.data !== null) {
      // Handle object
    }
  }
}
```

### Advanced Examples

```typescript
// Type predicate narrowing from unknown
function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === "object" && value !== null && !Array.isArray(value);
}

function isStringRecord(value: unknown): value is Record<string, string> {
  if (!isRecord(value)) return false;
  return Object.values(value).every((v) => typeof v === "string");
}

// Safe access with unknown
function getNested(obj: unknown, path: string[]): unknown {
  let current: unknown = obj;
  for (const key of path) {
    if (typeof current !== "object" || current === null) return undefined;
    if (!(key in current)) return undefined;
    current = (current as Record<string, unknown>)[key];
  }
  return current;
}

// Any escape hatch with documentation
// eslint-disable-next-line @typescript-eslint/no-explicit-any
function debugLog(...args: any[]): void {
  // Only use any for truly dynamic logging
  console.log(new Date().toISOString(), ...args);
}
```

### Real-World Use Cases

**Migration Strategy**: When migrating JavaScript to TypeScript, adding types gradually means using `any` temporarily. Track `any` usage with ESLint rules to eliminate over time.

**Third-party Library Integration**: When TypeScript definitions are incomplete, `unknown` provides a bridge until proper types are available.

**Dynamic Plugin Systems**: Plugin architectures where the shape of data is genuinely unknown at compile time.

### Common Mistakes

**Using `any` When `unknown` Would Work**: `any` should be extremely rare in well-typed codebases.

**Not Narrowing `unknown` Before Use**: `unknown` without narrowing is unusable — that's the point.

**Cascading `any`**: Once `any` infects a type, it spreads. One `any` return value makes all consumers unchecked.

### Best Practices

1. Never use `any` for new code unless absolutely necessary
2. Use `unknown` for values of genuinely unknown type
3. Always narrow `unknown` with type guards before use
4. Add ESLint rule `@typescript-eslint/no-explicit-any` to prevent any
5. Document any legitimate `any` usage with comments explaining why

### Performance Considerations

`any` and `unknown` have no runtime cost. However, excessive `any` can cause the compiler to skip optimizations that rely on type information. `unknown` maintains type safety without performance cost.

### Interview Questions

1. What is the difference between `any` and `unknown`?
2. When would you use `unknown` instead of generics?
3. How does `any` propagate through type checking?
4. Why is `unknown` considered type-safe?
5. How do you safely narrow `unknown` to a specific type?

### Coding Challenges

1. Write a type-safe `JSON.parse` that returns `unknown`
2. Create a function that deeply converts `unknown` to a typed Record
3. Implement a type-safe event emitter using `unknown` for event payloads

### Related Topics

- TypeScript Handbook — Basic Types
- Type Guards and Narrowing
- TypeScript Migration Guide
- ESLint TypeScript Rules

## never and void

### What It Is

`void` represents the absence of a value (returned by functions that don't return anything useful). `never` represents values that never occur — functions that throw exceptions, infinite loops, or impossible type states.

### Why It Is Important

`void` is essential for typing functions that perform side effects without returning values. `never` is crucial for exhaustiveness checking in discriminated unions and for representing unreachable code paths in the type system.

### How It Works Internally

- `void`: In JavaScript, a function that doesn't explicitly return a value returns `undefined`. TypeScript uses `void` to represent this pattern. `void` is assignable from `undefined` but not from other types.
- `never`: A type with no values. The bottom type in TypeScript's type system. Every type is assignable to `never`? No — `never` is assignable to every type, but no type (except `never` itself) is assignable to `never`. It represents a logical impossibility.

```typescript
// void return type
function log(message: string): void {
  console.log(message);
  // No return statement — returns undefined
}

// never return type — function never completes
function throwError(message: string): never {
  throw new Error(message);
}

// never in union types — disappears in unions
type Test = string | never; // string
type Test2 = never | number; // number
```

### Syntax

```typescript
// void return type
function doSomething(): void {
  console.log("Did something");
}

// void as a variable type (unusual but valid)
let unusable: void = undefined;
// unusable = null; // Error if strictNullChecks is on

// never return type
function fail(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {
    // infinite
  }
}

// never in exhaustiveness checks
type Shape = "circle" | "square" | "triangle";
function assertNever(x: never): never {
  throw new Error(`Unexpected value: ${x}`);
}
```

### Beginner Examples

```typescript
// void — no return value
function greet(name: string): void {
  console.log(`Hello, ${name}!`);
}

// void accepts undefined
const result: void = undefined;

// never — throws exception
function panic(): never {
  throw new Error("Panic!");
}

// never — infinite loop
function forever(): never {
  while (true) {
    console.log("Running...");
  }
}

// void for callbacks
function forEach<T>(items: T[], callback: (item: T) => void): void {
  for (const item of items) {
    callback(item);
  }
}
```

### Intermediate Examples

```typescript
// Exhaustive switch with never
type Animal = "dog" | "cat" | "bird";

function makeSound(animal: Animal): string {
  switch (animal) {
    case "dog":
      return "woof";
    case "cat":
      return "meow";
    case "bird":
      return "tweet";
    default:
      // If a new animal is added without a case, this line errors
      return assertNever(animal);
  }
}

function assertNever(x: never): never {
  throw new Error(`Unexpected value: ${x}`);
}

// Adding 'fish' to Animal would cause a compile error
// type Animal = "dog" | "cat" | "bird" | "fish";
// Error in switch: Argument of type 'string' not assignable to 'never'

// never in conditional types
type NonNullable2<T> = T extends null | undefined ? never : T;
type T1 = NonNullable2<string | null>; // string
type T2 = NonNullable2<number | undefined | null>; // number

// void in callback contexts
const arr = [1, 2, 3];
const mapped: void[] = arr.map((x) => {
  console.log(x);
}); // void[] — forEach equivalent
```

### Advanced Examples

```typescript
// Type-level never filtering
type FilterNever<T> = {
  [K in keyof T as T[K] extends never ? never : K]: T[K];
};

type Test = FilterNever<{ a: string; b: never; c: number }>;
// { a: string; c: number }

// never in promise resolution
type Result<T> = T extends Promise<infer U> ? U : T;
type R1 = Result<Promise<string>>; // string
type R2 = Result<string>; // string

// Void in mapped types for filtering
type Methods<T> = {
  [K in keyof T as T[K] extends (...args: any[]) => any ? K : never]: T[K];
};

interface Service {
  name: string;
  start(): void;
  stop(): void;
  config: Record<string, unknown>;
}

type ServiceMethods = Methods<Service>;
// { start: () => void; stop: () => void }

// Never in tuple manipulation
type Push<T extends any[], V> = [...T, V];
type Pop<T extends any[]> = T extends [...infer U, any] ? U : never;
type Shift<T extends any[]> = T extends [any, ...infer U] ? U : never;

type Tuple = [1, 2, 3];
type Popped = Pop<Tuple>; // [1, 2]
type Shifted = Shift<Tuple>; // [2, 3]

// Void in function overloads
function handler(callback: () => void): void;
function handler(callback: () => Promise<void>): void;
function handler(callback: (() => void) | (() => Promise<void>)): void {
  const result = callback();
  if (result instanceof Promise) {
    result.catch(console.error);
  }
}
```

### Real-World Use Cases

**Exhaustiveness Checking**: When adding new cases to discriminated unions, `never` ensures every case is handled — if you forget a case, TypeScript flags the error.

**Error Handling**: Functions that always throw (like assertion utilities) return `never`, signaling to the compiler that subsequent code is unreachable.

**Redux Reducers**: The `default` case in reducers uses `never` to ensure all action types are handled.

### Common Mistakes

**Confusing `void` with `undefined`**: While similar, `void` return type means the return value is ignored, not that it's `undefined`.

**Using `void` for Variables**: `void` as a variable type is rarely useful — use `undefined` instead.

**Forgetting `never` in Unions**: `never` disappears in unions (`string | never` = `string`), which is useful for filtering but can be surprising.

### Best Practices

1. Use `void` for functions with side effects that don't return a value
2. Use `never` for exhaustiveness checks in discriminated unions
3. Never assign to `never` variables — it indicates unreachable code
4. Use `never` in conditional types to filter out cases
5. Prefer `void` over `undefined` for function return types

### Performance Considerations

Both `void` and `never` are compile-time only concepts with no runtime representation. They don't affect bundle size or runtime performance.

### Interview Questions

1. What is the difference between `void` and `undefined`?
2. What is the `never` type and when would you use it?
3. How does `never` behave in union and intersection types?
4. How do you perform exhaustiveness checking in TypeScript?
5. Can a function returning `void` actually return a value?

### Coding Challenges

1. Create an exhaustive type checker for a 5-case discriminated union
2. Implement a `FilterNullable<T>` type that removes `null` and `undefined` from a union
3. Write a function that returns `never` and demonstrate its use in control flow analysis

### Related Topics

- TypeScript Control Flow Analysis
- Discriminated Unions
- Conditional Types
- Function Return Types

## Type inference vs annotations

### What It Is

Type inference is TypeScript's ability to automatically determine the type of a value based on its usage and context. Type annotations are explicit declarations where the developer specifies the type. TypeScript balances both approaches, encouraging inference for simple cases and annotations for complex ones.

### Why It Is Important

Understanding when to rely on inference and when to use annotations is essential for writing idiomatic TypeScript. Over-annotating creates noise; under-annotating can lead to type errors or confusing code. The right balance improves readability and type safety.

### How It Works Internally

TypeScript uses bidirectional type inference:

1. **Bottom-up inference**: Types are inferred from their values. `let x = 5` infers `x: number`
2. **Top-down (contextual) inference**: Types are inferred from how values are used. `const fn: (x: number) => void = (x) => { ... }` infers `x` as `number`
3. **Flow-based inference**: Types are refined based on control flow. After a `typeof` check, the type narrows

```typescript
// Bottom-up
let x = 5; // Type: number

// Top-down (contextual typing)
const names: string[] = [];
names.forEach((name) => {
  // name is inferred as string from context
  console.log(name.toUpperCase());
});

// Flow-based
function example(x: string | number) {
  if (typeof x === "string") {
    // x is string here
    return x.length;
  }
  // x is number here
  return x * 2;
}
```

### Syntax

```typescript
// Type inference (no annotation)
let name = "Alice"; // inferred: string
let count = 42; // inferred: number
let items = ["a", "b", "c"]; // inferred: string[]
let config = { theme: "dark", volume: 75 }; // inferred: { theme: string; volume: number }

// Type annotations (explicit)
let username: string = "Alice";
let itemCount: number = 42;
let configExplicit: { theme: string; volume: number } = { theme: "dark", volume: 75 };

// Const context — narrower inference
const greeting = "Hello"; // inferred: "Hello" (literal type, not string)
let greeting2 = "Hello"; // inferred: string (widened)
```

### Beginner Examples

```typescript
// Let inference handle simple cases
const name = "Alice"; // const -> literal type "Alice"
let age = 30; // let -> number
const items = [1, 2, 3]; // number[]

// Annotate function signatures
function add(a: number, b: number): number {
  return a + b;
}

// Inference works in return position
function multiply(a: number, b: number) {
  return a * b; // return type inferred as number
}

// Object literal inference
const person = {
  name: "Alice",
  age: 30,
};
// Type: { name: string; age: number }
```

### Intermediate Examples

```typescript
// Function parameter inference with callbacks
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((n) => n * 2); // n inferred as number

// Complex object inference
function createUser(name: string, age: number) {
  return {
    id: Math.random().toString(36),
    name,
    age,
    createdAt: new Date(),
  };
}
// Return type inferred as:
// { id: string; name: string; age: number; createdAt: Date }

// Generic inference
function firstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

const first = firstElement([1, 2, 3]); // T inferred as number, first: number | undefined
const firstStr = firstElement(["a", "b"]); // T inferred as string
```

### Advanced Examples

```typescript
// Conditional return type inference
function createContainer<T extends string | number>(value: T): T extends string ? { text: string } : { value: number } {
  if (typeof value === "string") {
    return { text: value } as any;
  }
  return { value } as any;
}
const strContainer = createContainer("hello"); // { text: string }
const numContainer = createContainer(42); // { value: number }

// Template literal inference
function createEndpoint<T extends string>(resource: T): `/${T}` {
  return `/${resource}`;
}
const usersEndpoint = createEndpoint("users"); // "/users"

// Inferred generic constraint
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
const obj = { name: "Alice", age: 30 };
const val = getProperty(obj, "name"); // string

// Mapped type inference
type Readonly2<T> = {
  readonly [P in keyof T]: T[P];
};
interface Point {
  x: number;
  y: number;
}
const point: Readonly2<Point> = { x: 10, y: 20 };
// point.x = 5; // Error: readonly

// Satisfies operator (TS 4.9+) — type validation without widening
interface ColorConfig {
  primary: string;
  secondary: string;
}

const palette = {
  primary: "#3498db",
  secondary: "#2ecc71",
} satisfies ColorConfig;
// Palette type is still { primary: string; secondary: string }, not widened to ColorConfig
// But satisfies ensures it matches ColorConfig
```

### Real-World Use Cases

**API Client Functions**: Function return types are often inferred from the implementation, while parameters are annotated.

**React Hooks**: Custom hooks use inference for return values and annotations for parameters.

**Configuration Objects**: The `satisfies` operator allows type validation without widening, preserving literal types for autocomplete.

### Common Mistakes

**Over-annotating Variables**: `let name: string = "Alice"` is redundant — TypeScript infers `string` from the value.

**Not Annotating Function Parameters**: Parameters lack contextual inference in most cases — they must be annotated.

**Assuming Inference Works Everywhere**: Complex operations (like conditional returns) may require explicit annotations.

**Object Widening**: Without `satisfies` or `as const`, objects can have their types widened unexpectedly.

### Best Practices

1. Let inference handle local variable types
2. Always annotate function parameter types
3. Annotate function return types for public APIs
4. Use `satisfies` for type validation without widening
5. Use `as const` to preserve literal types in objects and arrays
6. Annotate complex generic constraints explicitly

### Performance Considerations

Inference has minimal performance cost. The TypeScript compiler infers types during the type-checking phase regardless of annotations. Annotations can actually slow compilation slightly by adding more AST nodes to process, but the difference is negligible.

### Interview Questions

1. When should you use type annotations instead of relying on inference?
2. How does contextual typing work in TypeScript?
3. What is the difference between `const` and `let` in type inference?
4. How does the `satisfies` operator differ from type annotations?
5. How does TypeScript infer generic type parameters?

### Coding Challenges

1. Write a function where the return type is inferred based on a discriminated parameter
2. Create a generic function where the type parameter is inferred from usage
3. Implement a curried function using type inference and contextual typing
4. Use `satisfies` to validate an object type while preserving literal types

### Related Topics

- TypeScript Type Inference
- Contextual Typing
- Type Widening and Narrowing
- The `satisfies` Operator
- Generic Type Inference
