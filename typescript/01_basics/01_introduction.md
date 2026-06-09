# Introduction to TypeScript - What is TS?, Benefits over JS, Compilation, Type system overview

## Introduction

TypeScript is a strict syntactical superset of JavaScript developed and maintained by Microsoft. It adds optional static typing, classes, interfaces, and other powerful features on top of JavaScript, enabling developers to catch errors at compile time rather than runtime. Since its initial release in 2012, TypeScript has become one of the most widely adopted languages in the web development ecosystem, powering everything from small hobby projects to enterprise-scale applications used by millions of users.

## What is TypeScript?

### What It Is

TypeScript is an open-source programming language that builds upon JavaScript by adding static type definitions. It compiles to plain JavaScript, meaning any valid JavaScript code is also valid TypeScript code. This relationship is often described as TypeScript being a "superset" of JavaScript — everything you know about JavaScript carries over, and TypeScript adds additional capabilities on top.

The core innovation of TypeScript is its type system, which allows developers to specify the shapes of data, the signatures of functions, and the contracts between different parts of a codebase. These type annotations are entirely optional, making TypeScript approachable for JavaScript developers who want to adopt it incrementally.

```typescript
// Valid JavaScript is valid TypeScript
function greet(name) {
  return `Hello, ${name}!`;
}

// TypeScript adds type annotations
function greetTyped(name: string): string {
  return `Hello, ${name}!`;
}
```

### Why It Is Important

TypeScript addresses the fundamental challenges that arise when building large-scale JavaScript applications. JavaScript's dynamic nature, while flexible, becomes a liability as codebases grow — refactoring becomes dangerous, API contracts become unclear, and bugs that could have been caught statically slip into production.

TypeScript shifts error detection from runtime to compile time, catching entire categories of bugs before they ever reach users. Studies have shown that TypeScript catches 15-20% of bugs at compile time that would otherwise be discovered during testing or in production. In large codebases with many contributors, the type system serves as living documentation, making it immediately clear what shape data should take and what functions expect.

Beyond bug prevention, TypeScript dramatically improves developer productivity through better tooling. Modern IDEs leverage TypeScript's type information to provide accurate autocompletion, inline documentation, refactoring tools, and "go to definition" navigation that would be impossible with plain JavaScript.

### How It Works Internally

TypeScript operates through a multi-phase compilation pipeline:

1. **Parsing**: The TypeScript compiler (`tsc`) first parses the source code into a syntax tree (`SourceFile` AST node). Unlike JavaScript parsers that stop at the AST, TypeScript's parser also produces line maps and position tracking.

2. **Binder**: The binder creates `Symbols` for each named entity (variables, functions, types) and links declarations to their corresponding symbols. This phase produces the `SymbolTable` that connects all references to declarations across files.

3. **Type Checking**: The type checker walks the AST and evaluates type expressions, performing unification and constraint solving. It uses a bidirectional type inference algorithm — inferring types from usage patterns while also checking explicit annotations against usage.

4. **Transformation**: Once type checking is complete, TypeScript strips all type annotations and transforms newer ECMAScript features to the target version specified in `tsconfig.json`. This phase uses a series of transformers (e.g., `ClassTransformer`, `ES2015forOfTransformer`) that map TypeScript AST nodes to JavaScript AST nodes.

5. **Emission**: The transformed AST is printed back to source code, producing `.js` output files, and optionally `.d.ts` declaration files and `.js.map` source maps.

```typescript
// TypeScript source
interface User {
  id: number;
  name: string;
}

function formatUser(user: User): string {
  return `${user.name} (${user.id})`;
}

// After compilation (target: ES2015)
// function formatUser(user) {
//   return user.name + " (" + user.id + ")";
// }
// Note: interface User is erased entirely — no runtime cost!
```

### Syntax

```typescript
// Basic type annotations
let username: string = "Alice";
let age: number = 30;
let isActive: boolean = true;

// Function with typed parameters and return
function add(a: number, b: number): number {
  return a + b;
}

// Interface defining object shape
interface Person {
  firstName: string;
  lastName: string;
  age?: number; // optional property
}

// Type assertion (when you know more than TS)
const input = document.getElementById("name") as HTMLInputElement;
input.value; // TypeScript knows this is HTMLInputElement

// Generic function
function identity<T>(arg: T): T {
  return arg;
}
```

### Beginner Examples

```typescript
// Hello World with types
function sayHello(name: string): void {
  console.log(`Hello, ${name}!`);
}
sayHello("TypeScript"); // Hello, TypeScript!

// Type inference in action
let message = "Hello!"; // TypeScript infers `string`
// message = 42; // Error: Type 'number' is not assignable to type 'string'

// Simple typed function
function square(x: number): number {
  return x * x;
}
console.log(square(5)); // 25

// Optional parameter
function greetFull(firstName: string, lastName?: string): string {
  if (lastName) {
    return `Hello, ${firstName} ${lastName}!`;
  }
  return `Hello, ${firstName}!`;
}
console.log(greetFull("Alice")); // Hello, Alice!
console.log(greetFull("Bob", "Smith")); // Hello, Bob Smith!
```

### Intermediate Examples

```typescript
// Union types
function formatId(id: string | number): string {
  return `ID: ${id}`;
}

// Type guards
function processValue(value: string | string[]): string {
  if (Array.isArray(value)) {
    return value.join(", ");
  }
  return value;
}

// Generics with constraints
interface HasLength {
  length: number;
}
function logLength<T extends HasLength>(item: T): T {
  console.log(item.length);
  return item;
}
logLength("hello"); // 5
logLength([1, 2, 3]); // 3
// logLength(123); // Error: number doesn't have length

// Mapped types
type Readonly2<T> = {
  readonly [P in keyof T]: T[P];
};

interface Point {
  x: number;
  y: number;
}
type ReadonlyPoint = Readonly2<Point>;
// { readonly x: number; readonly y: number; }
```

### Advanced Examples

```typescript
// Conditional types
type IsString<T> = T extends string ? "yes" : "no";
type Test1 = IsString<string>; // "yes"
type Test2 = IsString<number>; // "no"

// Template literal types
type EventName = `on${Capitalize<string>}`;
// Represents "on" followed by any capitalized string

// Discriminated unions
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rectangle"; width: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
  }
}

// Branded types for type safety
type Brand<K, T> = K & { __brand: T };
type USD = Brand<number, "USD">;
type EUR = Brand<number, "EUR">;

function createUSD(amount: number): USD {
  return amount as USD;
}

function createEUR(amount: number): EUR {
  return amount as EUR;
}

function buy(usd: USD, eur: EUR): string {
  return `Bought with ${usd} USD and ${eur} EUR`;
}

const dollars = createUSD(100);
const euros = createEUR(200);
buy(dollars, euros);
// buy(euros, dollars); // Error: EUR not assignable to USD
```

### Real-World Use Cases

**React Applications**: TypeScript is extensively used in React projects for props typing, state management, and custom hooks. The `@types/react` package provides comprehensive type definitions for React's API.

```typescript
interface ButtonProps {
  label: string;
  variant: "primary" | "secondary";
  onClick: () => void;
  disabled?: boolean;
}

const Button: React.FC<ButtonProps> = ({
  label,
  variant,
  onClick,
  disabled = false,
}) => {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
};
```

**Node.js Backend Services**: TypeScript is widely adopted in backend development with Express, NestJS, or Fastify. The type system ensures request/response handling is type-safe.

**API Clients and SDKs**: TypeScript declaration files allow library authors to provide type-safe APIs. Popular libraries like Axios, Prisma, and Apollo Client ship with first-class TypeScript support.

### Common Mistakes

**Overusing `any`**: The `any` type disables type checking entirely, defeating TypeScript's purpose. Prefer `unknown` when the type is genuinely unknown.

```typescript
// Bad
function process(data: any): any {
  return data.name; // no type safety
}

// Good
function process<T extends { name: string }>(data: T): string {
  return data.name;
}
```

**Ignoring Strict Mode**: Running TypeScript without `strict: true` in `tsconfig.json` disables many valuable checks like strict null checking.

**Using Type Assertions Incorrectly**: Type assertions should confirm what you know, not override the compiler's safety.

```typescript
// Risky — could be null
const input = document.getElementById("name") as HTMLInputElement;

// Safer
const input = document.getElementById("name");
if (input instanceof HTMLInputElement) {
  input.value;
}
```

### Best Practices

1. Enable `strict: true` in tsconfig.json from the start
2. Prefer `interface` for public API shapes and `type` for unions/intersections
3. Use `unknown` instead of `any` for values of unknown type
4. Leverage type inference for simple variable declarations
5. Write explicit return types on function declarations for clarity
6. Use branded types for domain modeling with strong guarantees
7. Keep type definitions close to their usage

### Performance Considerations

TypeScript's type system has computational limits. Complex conditional types, deeply recursive types, and large union types can slow down compilation significantly. The `--noUnusedLocals` and `--noUnusedParameters` flags add checking overhead but improve code quality. For large projects, consider using project references (`composite: true`) to enable incremental builds and faster type checking. TypeScript 5.x introduced `--explainFiles` and `--traceResolution` for diagnosing slow compilation.

### Interview Questions

1. What is the difference between `interface` and `type`?
2. How does TypeScript's structural type system differ from nominal typing (e.g., Java)?
3. Explain the concept of type narrowing and give three examples of type guards.
4. What are mapped types and how do they work?
5. How does the `--strict` flag affect TypeScript's behavior?
6. What is a discriminated union and when would you use it?
7. Explain the difference between `any`, `unknown`, and `never`.
8. How do you merge declarations across files in TypeScript?
9. What are conditional types and how do they enable `ReturnType<T>`?
10. How does TypeScript handle module resolution?

### Coding Challenges

1. Implement a type-safe `get` function that retrieves nested properties with proper typing
2. Create a `DeepReadonly<T>` mapped type that makes all nested properties readonly
3. Implement a `PickByType<T, U>` utility type that picks properties matching a type
4. Write a type-safe event emitter with typed event names and payloads
5. Create a branded types implementation for currency conversion

### Related Topics

- [TypeScript Handbook — Basic Types](https://www.typescriptlang.org/docs/handbook/basic-types.html)
- [TypeScript Compiler Options](https://www.typescriptlang.org/tsconfig)
- [TypeScript with React](https://react-typescript-cheatsheet.netlify.app/)
- [TypeScript Design Patterns](https://github.com/torokmark/design_patterns_in_typescript)
- [TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html)

## Benefits over JavaScript

### What It Is

TypeScript's benefits over plain JavaScript encompass three major categories: static type checking, enhanced tooling, and improved developer experience. These benefits compound as project size and team size increase, making TypeScript the standard choice for professional JavaScript development.

### Why It Is Important

JavaScript was designed as a scripting language for browser interactivity — type safety was never a design goal. As JavaScript applications grew from simple form validation to complex single-page applications, the absence of static types became the primary source of runtime errors and reduced developer productivity. TypeScript systematically addresses these pain points without abandoning the JavaScript ecosystem.

### How It Works Internally

TypeScript's type checking is structural (duck typing) rather than nominal. When checking assignability, TypeScript compares the shapes of types rather than their names. This means a type `{ name: string; age: number }` is compatible with any object having the same structure, regardless of declared interface name.

The type checker uses a control-flow-based analysis that narrows types within conditional branches. This allows TypeScript to understand that after a `typeof x === "string"` check, `x` is definitely a string in the truthy branch.

```typescript
function process(value: string | null): string {
  if (value === null) {
    return "default";
  }
  // TypeScript knows value is string here
  return value.toUpperCase();
}
```

### Syntax

```typescript
// JavaScript — no type checking
function divide(a, b) {
  return a / b;
}
divide("10", 0); // Inf, no error

// TypeScript — catches errors
function divide(a: number, b: number): number {
  return a / b;
}
// divide("10", 0); // Error: Argument of type 'string' is not assignable to 'number'
```

### Beginner Examples

```typescript
// JavaScript issue: no protection against wrong types
function calculateTotal(price, quantity) {
  return price * quantity;
}
console.log(calculateTotal("$10", 5)); // NaN — bug!

// TypeScript catches it
function calculateTotalTS(price: number, quantity: number): number {
  return price * quantity;
}
// calculateTotalTS("$10", 5); // Compile error

// Better autocompletion in IDEs
interface User {
  id: number;
  name: string;
  email: string;
}

function sendEmail(user: User): void {
  // IDE provides autocomplete for user.id, user.name, user.email
  console.log(`Sending to ${user.email}`);
}
```

### Intermediate Examples

```typescript
// TypeScript catches refactoring errors
interface User {
  id: number;
  name: string;
  email: string;
}

// Rename `email` to `emailAddress` — TypeScript finds all usages
function displayUser(user: User): string {
  return `${user.name} <${user.email}>`;
}

// API responses are self-documenting
interface ApiResponse<T> {
  data: T;
  error: string | null;
  status: number;
}

async function fetchUser(id: number): Promise<ApiResponse<User>> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
  // Return type ensures you handle error and status
}
```

### Advanced Examples

```typescript
// Template literal types for type-safe string manipulation
type HexDigit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" | "a" | "b" | "c" | "d" | "e" | "f";
type HexColor = `#${HexDigit}${HexDigit}${HexDigit}${HexDigit}${HexDigit}${HexDigit}`;

// Type-safe validation
type ValidEmail = `${string}@${string}.${string}`;

function sendToEmail(email: ValidEmail): void {
  console.log(`Sending to ${email}`);
}
sendToEmail("user@example.com"); // OK
// sendToEmail("invalid"); // Error

// Types enforce business logic at compile time
type OrderState = "pending" | "confirmed" | "shipped" | "delivered" | "cancelled";
type ValidTransition = {
  pending: "confirmed" | "cancelled";
  confirmed: "shipped" | "cancelled";
  shipped: "delivered";
  delivered: never;
  cancelled: never;
};

function transitionOrder(
  current: OrderState,
  next: ValidTransition[OrderState]
): OrderState {
  return next;
}
// TypeScript prevents invalid state transitions at compile time
```

### Real-World Use Cases

**Large-Scale Refactoring**: Companies like Airbnb, Google, and Microsoft have successfully migrated large JavaScript codebases to TypeScript. During refactoring, TypeScript catches all callers of modified functions and ensures the new API is used correctly everywhere.

**Library Development**: Popular libraries like Angular, Vue 3, and Svelte are built with TypeScript, providing first-class type definitions to their users.

**Enterprise Applications**: Banking, healthcare, and e-commerce applications rely on TypeScript to enforce data integrity and reduce production incidents.

### Common Mistakes

**Thinking TypeScript Replaces Testing**: TypeScript catches type errors, not logic errors. Types don't ensure correct business logic.

**Over-engineering Types**: Not everything needs a complex generic type. Sometimes simple types are more maintainable.

**Ignoring `strictNullChecks`**: Without it, `null` and `undefined` are assignable to any type, defeating null safety.

### Best Practices

1. Always use `strict: true`
2. Write tests alongside types — they catch different kinds of bugs
3. Use `readonly` for immutable data structures
4. Leverage `as const` for literal type inference
5. Use `satisfies` operator (TS 4.9+) for type validation without widening

### Performance Considerations

TypeScript's compilation adds a build step. For development, use `tsc --noEmit` for just type checking and a separate bundler (esbuild, swc) for faster compilation. Project references enable incremental builds where only changed files are rechecked.

### Interview Questions

1. What specific advantages does TypeScript provide over JavaScript in a team setting?
2. How does TypeScript's structural type system enable more flexible code than Java's nominal system?
3. Can TypeScript completely eliminate runtime errors?
4. Why do many companies migrate large JS codebases to TypeScript?

### Coding Challenges

1. Create a type-safe EventEmitter that enforces event names and payloads
2. Implement a type-safe query builder for SQL
3. Build a typed state machine for an order processing system

### Related Topics

- TypeScript vs Flow (Facebook's type checker)
- Using TypeScript with Babel
- TypeScript in Deno vs Node.js
- The `satisfies` operator (TS 4.9+)

## Compilation process

### What It Is

The TypeScript compilation process transforms `.ts` and `.tsx` files into JavaScript while optionally emitting type declaration files (`.d.ts`) and source maps (`.js.map`). This compilation is handled by the TypeScript compiler (`tsc`), which performs parsing, type checking, and code generation in a sophisticated pipeline.

### Why It Is Important

Understanding the compilation process is crucial for configuring build pipelines, debugging issues, and optimizing development workflows. Unlike Babel or SWC, TypeScript also performs type checking during compilation, which catches errors before runtime.

### How It Works Internally

The TypeScript compilation pipeline consists of several phases:

1. **Scanner/Tokenizer**: Converts source text into a stream of tokens (keywords, identifiers, operators, etc.)
2. **Parser**: Converts tokens into an Abstract Syntax Tree (AST) using a recursive descent parser
3. **Binder**: Creates symbols and connects declarations to their scopes
4. **Type Resolver**: Resolves type references and evaluates type expressions
5. **Type Checker**: Performs type checking against the resolved types
6. **Emitter**: Transforms the AST to the target JavaScript version and generates output files

```typescript
// Input TypeScript
const greeting: string = "Hello";
console.log(greeting);

// Parsing -> Token stream:
// const, greeting, :, string, =, "Hello", ;
// console, ., log, (, greeting, ), ;

// After type checking -> JavaScript emission
// "use strict";
// const greeting = "Hello";
// console.log(greeting);
```

### Syntax

```bash
# Basic compilation
tsc index.ts

# Watch mode
tsc --watch

# With custom config
tsc -p tsconfig.prod.json

# Emit declaration files only
tsc --declaration --emitDeclarationOnly

# Project references build mode
tsc --build
```

### Beginner Examples

```typescript
// File: math.ts
export function add(a: number, b: number): number {
  return a + b;
}

// Command: tsc math.ts
// Output: math.js
// "use strict";
// Object.defineProperty(exports, "__esModule", { value: true });
// function add(a, b) {
//   return a + b;
// }
// exports.add = add;
```

### Intermediate Examples

```typescript
// File: user.ts
interface User {
  readonly id: number;
  name: string;
  email?: string;
}

class UserRepository {
  private users: Map<number, User> = new Map();

  add(user: User): void {
    this.users.set(user.id, user);
  }

  find(id: number): User | undefined {
    return this.users.get(id);
  }
}

export { User, UserRepository };

// Compiled output (ES2020 target):
// class UserRepository {
//   constructor() {
//     this.users = new Map();
//   }
//   add(user) {
//     this.users.set(user.id, user);
//   }
//   find(id) {
//     return this.users.get(user.id);
//   }
// }
```

### Advanced Examples

```typescript
// Using tsconfig paths and project references
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@models/*": ["src/models/*"],
      "@utils/*": ["src/utils/*"]
    },
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/utils" }
  ]
}

// TypeScript resolves imports using path mapping
import { User } from "@models/User";
// Resolved to: src/models/User.ts (or .d.ts)

// Declaration file output (User.d.ts)
// export interface User {
//   readonly id: number;
//   name: string;
//   email?: string;
// }
```

### Real-World Use Cases

**Monorepo Setup with Project References**: Large projects use TypeScript project references to enable incremental builds. Each package in the monorepo references its dependencies, allowing TypeScript to build only changed packages.

**Bundler Integration**: Modern bundlers (Vite, esbuild, Webpack with ts-loader) integrate TypeScript compilation. Vite uses esbuild for fast transpilation and `tsc` for type checking in a separate process.

**CI/CD Pipelines**: TypeScript compilation is integrated into CI/CD to catch type errors before deployment. `tsc --noEmit` is often run as a linting step.

### Common Mistakes

**Confusing Transpilation with Type Checking**: `tsc` does both, but bundlers like esbuild only strip types without checking them. Always run `tsc --noEmit` separately.

**Not Using `tsc --build` for Multi-file Projects**: In project references mode, `--build` handles dependency order automatically.

### Best Practices

1. Use `tsc --noEmit` in CI for fast type checking
2. Configure `outDir` and `rootDir` for clean output structure
3. Use `declaration: true` for library projects
4. Enable `sourceMap: true` for debugging
5. Use `tsc --build` for monorepos with project references

### Performance Considerations

- Large monorepos benefit from `incremental: true` and `tsBuildInfoFile`
- File watching (`--watch`) uses efficient incremental parsing
- Project references enable parallel builds
- Consider using `swc` or `esbuild` for transpilation alongside `tsc` for type checking only

### Interview Questions

1. What is the difference between `tsc` and a bundler like Webpack?
2. How does TypeScript's `--build` mode differ from regular `tsc`?
3. What are declaration files (`.d.ts`) and why are they important?
4. How does TypeScript resolve modules?
5. What is the purpose of `tsconfig.json`'s `paths` configuration?

### Coding Challenges

1. Set up a monorepo with two packages using project references
2. Configure a CI pipeline that runs type checking and transpilation separately
3. Create a custom TypeScript transformer plugin

### Related Topics

- tsconfig.json options reference
- TypeScript compiler API
- AST viewer (ts-ast-viewer.com)
- TypeScript custom transformers

## Type system overview

### What It Is

TypeScript's type system is a structural (duck typing), gradual type system that optionally enforces type constraints on values. It features generics, union and intersection types, conditional types, mapped types, and template literal types — providing rich expressiveness for encoding program invariants.

### Why It Is Important

The type system is the foundation of TypeScript's value proposition. Understanding its design principles (structural typing, type erasure, gradual typing) helps developers write safer, more expressive code and leverage advanced patterns like discriminated unions and branded types.

### How It Works Internally

TypeScript uses a structural type system where two types are compatible if their structures match. The type checker performs unification on types, handling:

- **Subtype checking**: Is type A assignable to type B?
- **Type inference**: What type does this expression have?
- **Control flow analysis**: How does the type narrow in different branches?

TypeScript types are erasable — they exist only at compile time and produce no runtime code. This means TypeScript adds zero runtime overhead while providing compile-time safety.

```typescript
interface Point {
  x: number;
  y: number;
}

interface LabeledPoint {
  x: number;
  y: number;
  label: string;
}

// Structural compatibility: LabeledPoint is a subtype of Point
// because it has all properties of Point
const labeled: LabeledPoint = { x: 1, y: 2, label: "A" };
const point: Point = labeled; // OK — structural typing
```

### Syntax

```typescript
// Primitive types
const isDone: boolean = false;
const decimal: number = 6;
const color: string = "blue";

// Array types
const list: number[] = [1, 2, 3];
const list2: Array<number> = [1, 2, 3];

// Tuple types
const pair: [string, number] = ["age", 30];

// Enum types
enum Color { Red, Green, Blue }
const c: Color = Color.Green;

// Function types
const myFunc: (x: number) => number = (x) => x * 2;

// Object types
const obj: { name: string; age: number } = { name: "Alice", age: 30 };
```

### Beginner Examples

```typescript
// Type inference
let x = 3; // x: number
let y = "hello"; // y: string
let z = true; // z: boolean

// Union type
let id: string | number = 123;
id = "abc"; // OK

// Array with types
const names: string[] = ["Alice", "Bob", "Charlie"];

// Enum
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}
const dir: Direction = Direction.Up;
```

### Intermediate Examples

```typescript
// Generic constraints
interface HasId {
  id: number;
}

function findById<T extends HasId>(items: T[], id: number): T | undefined {
  return items.find((item) => item.id === id);
}

// Conditional types
type NonNullable2<T> = T extends null | undefined ? never : T;
type Result = NonNullable2<string | null>; // string

// Mapped types
type Optional<T> = {
  [P in keyof T]?: T[P];
};

type Readonly3<T> = {
  readonly [P in keyof T]: T[P];
};
```

### Advanced Examples

```typescript
// Recursive conditional types
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends Record<string, unknown>
    ? DeepReadonly<T[P]>
    : T[P];
};

interface Config {
  name: string;
  nested: {
    value: number;
    deeper: {
      flag: boolean;
    };
  };
}

type ReadonlyConfig = DeepReadonly<Config>;
// All properties deeply readonly

// Distributive conditional types
type ToArray<T> = T extends unknown ? T[] : never;
type Result2 = ToArray<string | number>; // string[] | number[]

// Template literal types with inference
type ExtractName<T extends string> =
  T extends `${infer Name}@${infer Domain}` ? Name : never;

type UserName = ExtractName<"alice@example.com">; // "alice"

// Key remapping via as clause
type Getters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};

interface Person {
  name: string;
  age: number;
}
// type PersonGetters = {
//   getName: () => string;
//   getAge: () => number;
// }
```

### Real-World Use Cases

**API Response Typing**: Full type coverage for REST API responses, including error states, pagination, and nested resources.

**State Machines**: Discriminated unions encode every possible state and transition of a finite state machine.

**Form Validation**: Conditional types and template literals create type-safe form validators where field names are checked at compile time.

**ORM Integration**: Prisma uses TypeScript's template literal types and mapped types to generate fully typed database queries.

### Common Mistakes

**Over-complicating Types**: Simple types are better than clever types most of the time.

**Forgetting Type Erasure**: Types don't exist at runtime. Type guards must use JavaScript constructs (`typeof`, `instanceof`, property checks).

**Misunderstanding Distributive Conditional Types**: Conditional types distribute over unions when the checked type is a bare type parameter.

### Best Practices

1. Prefer explicit return types on functions for documentation
2. Use `readonly` for immutable interfaces
3. Leverage `satisfies` for type validation without widening
4. Use discriminated unions for complex state management
5. Keep generic type parameter names descriptive (`TItem` not just `T`)

### Performance Considerations

Deeply recursive types (e.g., `DeepReadonly` on large interfaces) can slow down type checking. TypeScript 5.x introduced performance improvements for recursive types but complex transformations still have cost. Use `type` aliases to cache complex type computations.

### Interview Questions

1. Explain structural typing versus nominal typing with examples.
2. What are the four categories of TypeScript types?
3. How do conditional types distribute over unions?
4. What is the difference between `keyof` and `in` in mapped types?
5. How does TypeScript's `never` type behave in unions and intersections?

### Coding Challenges

1. Implement `DeepPartial<T>` that makes all nested properties optional
2. Create `PickByValue<T, V>` that picks keys whose values match type `V`
3. Implement a `TupleToUnion<T extends readonly any[]>` type
4. Create a type-safe `Object.keys` that returns only known keys
5. Implement `FunctionPropertyNames<T>` that extracts property names with function values

### Related Topics

- TypeScript Handbook — Types
- TypeScript Type Relationships
- Advanced Types (union, intersection, conditional)
- Template Literal Types
- Mapped Types Deep Dive
