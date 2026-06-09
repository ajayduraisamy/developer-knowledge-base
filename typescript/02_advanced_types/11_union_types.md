# Union Types - Union syntax, type narrowing, discriminated unions, exhaustiveness checking

## Introduction

Union types are one of TypeScript's most powerful type-level features. They allow a value to be one of several possible types, enabling flexible APIs and expressive type definitions. A union type is formed using the pipe (`|`) symbol between two or more types. When you write `string | number`, you declare that a value can be either a string or a number. Union types sit at the heart of TypeScript's structural type system, enabling pattern matching–style code through type narrowing and discriminated unions.

Union types are not merely a syntactic convenience—they fundamentally change how we model data. Instead of creating complex class hierarchies or resorting to `any`, union types let the type system precisely describe the shape of data that varies. Combined with control-flow-based type narrowing, they allow the compiler to understand which type is in use at any point in your code, catching errors before runtime.

## Union type syntax (|)

### What It Is

The union type syntax uses the vertical bar (`|`) to declare that a value can be any one of multiple types. For example, `number | boolean` means the value is either a `number` or a `boolean`. Union types can combine primitives, object types, arrays, tuples, or any other valid TypeScript type.

```typescript
type Status = "idle" | "loading" | "success" | "error";
type ID = string | number;
type JSONValue = string | number | boolean | null | JSONValue[] | { [key: string]: JSONValue };
```

### Why It Is Important

Union types eliminate the need for overloaded functions that accept multiple distinct parameter shapes. They make APIs more flexible while maintaining type safety. Without unions, developers would fall back to `any` or cumbersome type-casting. Union types also form the foundation for discriminated unions, which model complex state machines, async states, and protocol messages with full type safety.

### How It Works Internally

Internally, the TypeScript compiler represents union types as a union of type flags or objects. During subtype checking, a type `A` is a subtype of `B | C` if `A` is a subtype of `B` or a subtype of `C`. When checking assignability, the compiler iterates over each member of the union. The compiler also performs union type reduction—removing redundant members and simplifying `never` types within unions.

```typescript
// Internally simplified: never is absorbed
type Reduced = string | never; // string
type Normalized = string | string; // string
```

### Syntax

```typescript
// Basic union syntax
type Result = number | string;

// Union of literal types
type Direction = "north" | "south" | "east" | "west";

// Union of object types
type Shape = 
  | { kind: "circle"; radius: number }
  | { kind: "square"; sideLength: number }
  | { kind: "triangle"; base: number; height: number };

// Union of arrays
type NumberOrStringArray = number[] | string[];

// Union with function types
type AsyncHandler = (data: string) => Promise<void> | void;

// Union with generic types
type Maybe<T> = T | null | undefined;

// Parentheses for grouping
type Complex = (string | number)[] | boolean;
```

### Beginner Examples

```typescript
// Simple union parameter
function printId(id: number | string): void {
  console.log(`ID: ${id}`);
}

printId(101);
printId("ABC-123");

// Union return type
function getLength(value: string | any[]): number {
  return value.length;
}

// Union in variable declaration
let value: string | number = "hello";
value = 42;

// Union with optional
function greet(name: string | undefined): string {
  if (name === undefined) return "Hello, guest";
  return `Hello, ${name}`;
}
```

### Intermediate Examples

```typescript
// Union types with arrays
function firstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

// Union of function signatures
function padLeft(value: string, padding: number | string): string {
  if (typeof padding === "number") {
    return " ".repeat(padding) + value;
  }
  return padding + value;
}

// Union of complex object shapes
interface Cat {
  type: "cat";
  meow: () => string;
}
interface Dog {
  type: "dog";
  bark: () => string;
}
type Pet = Cat | Dog;

// Union with intersection
type Admin = { role: "admin"; permissions: string[] };
type User = { role: "user"; email: string };
type Person = Admin | User;

// Nested unions
type Nested = (string | number)[] | { a: string | boolean };
```

### Advanced Examples

```typescript
// Recursive union types
type JSONValue =
  | string
  | number
  | boolean
  | null
  | JSONValue[]
  | { [key: string]: JSONValue };

function stringifyJSON(value: JSONValue): string {
  return JSON.stringify(value);
}

// Union type with template literal types
type EventName = `on${Capitalize<string>}`;
type MouseEvent = `mouse${"enter" | "leave" | "move"}`;

// Branded union types for nominal typing
type Brand<T, B> = T & { __brand: B };
type UserID = Brand<string, "UserID">;
type PostID = Brand<string, "PostID">;

function getUser(id: UserID): void {}
function getPost(id: PostID): void {}

declare const uid: UserID;
declare const pid: PostID;

getUser(uid);
getPost(pid);
// getUser(pid); // Error: Type 'PostID' is not assignable to type 'UserID'

// Union types with mapped and conditional types
type Nullable<T> = T | null | undefined;
type NonNull<T> = T extends null | undefined ? never : T;

// Union filtering
type ExtractString<T> = T extends string ? T : never;
type OnlyStrings = ExtractString<string | number | boolean>; // string
```

### Real-World Use Cases

- **API response states**: Modeling loading, success, and error states with a discriminated union.
- **Redux actions**: Each action in a reducer is a member of a union type.
- **Configuration objects**: Allowing config values to be specified as strings or pre-parsed objects.
- **Event handlers**: Accepting both callback functions and event emitter objects.
- **Database queries**: Parameters that can be a string ID or a filter object.
- **UI component props**: Accepting multiple input shapes for flexible components.
- **Protocol parsing**: Representing different message types in network protocols.

```typescript
// Real-world: API state machine
type AsyncState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };

// Real-world: Redux action
type Action =
  | { type: "ADD_TODO"; payload: { text: string } }
  | { type: "DELETE_TODO"; payload: { id: number } }
  | { type: "TOGGLE_TODO"; payload: { id: number } };

// Real-world: Component props
type InputProps =
  | { variant: "text"; placeholder: string; maxLength?: number }
  | { variant: "number"; min: number; max: number }
  | { variant: "select"; options: string[] };
```

### Common Mistakes

1. **Forgetting to narrow the type** before accessing type-specific properties.
2. **Using `any` instead of a union** when multiple types are acceptable.
3. **Overly broad unions** like `string | any` (the `any` absorbs everything).
4. **Not handling all union members** in switch statements (missing exhaustiveness).
5. **Confusing union (`|`) with intersection (`&`)**.
6. **Assuming order matters**—union members are unordered in the type system.
7. **Using union types when overloads are clearer** for distinct parameter lists.

```typescript
// Mistake: no narrowing
function handle(value: string | number) {
  // value.toFixed(2); // Error!
}

// Mistake: any in union
function bad(value: string | any) {} // any dominates

// Mistake: confusing | and &
type A = { a: string } | { b: number }; // either a or b
type B = { a: string } & { b: number }; // both a and b
```

### Best Practices

- Always narrow with `typeof`, `instanceof`, `in`, or discriminated properties before accessing type-specific members.
- Use discriminated unions with a literal `type` or `kind` property for complex state machines.
- Prefer union types over function overloads when only the return type differs.
- Use `never` for exhaustiveness checking in switch/default branches.
- Keep union types focused—avoid unions of more than 5-6 large object types.
- Use type aliases (`type`) to name unions for reuse rather than inline unions.

### Performance Considerations

Union types themselves have zero runtime cost—they are erased during compilation. However, large union types can slow down the TypeScript compiler's type-checking performance. Unions with hundreds of members (e.g., `string | "literal1" | "literal2" | ...`) can cause significant compilation slowdowns. The compiler must check each member during assignability and narrowing, so prefer discriminated unions with few variants over tagged unions with dozens.

```typescript
// Expensive for the compiler (many string literals)
type Slow = "a" | "b" | "c" | ... | "z" | ...;

// More efficient: use a discriminated structure
type Fast = { kind: "letters"; value: string };
```

### Interview Questions

1. **Q**: What is the difference between a union type and an intersection type?
   **A**: A union type (`A | B`) means the value can be `A` *or* `B`. An intersection type (`A & B`) means the value must be *both* `A` and `B`.

2. **Q**: How does TypeScript narrow union types?
   **A**: TypeScript uses control flow analysis. Within conditional branches with `typeof`, `instanceof`, `in`, or equality checks, the compiler narrows the type to the compatible union members.

3. **Q**: What is an absorption law in union types?
   **A**: `never` is absorbed: `T | never` simplifies to `T`. Also, `any` absorbs everything: `T | any` simplifies to `any`.

4. **Q**: How do you perform exhaustiveness checking with union types?
   **A**: Use a `default` branch in a switch statement that assigns the value to a variable of type `never`. If any union member is unhandled, the assignment fails.

### Coding Challenges

1. Implement a type-safe event emitter that uses union types for event names and payloads.
2. Write a function that parses a JSON string and returns a union type of all possible JSON value types.
3. Create a discriminated union representing a user's authentication state (unauthenticated, loading, authenticated, error).
4. Implement a deep `Pick` utility using union types and template literals.

### Related Topics

- Intersection types
- Type guards
- Discriminated unions
- Exhaustiveness checking
- Conditional types
- Template literal types

## Type narrowing with typeof/in

### What It Is

Type narrowing is the process by which TypeScript refines a union type to a more specific type within a code branch. The compiler analyzes control flow and uses type guards—expressions that narrow types—to determine which union members are possible at any point. The `typeof` operator and the `in` operator are two primary built-in type guards.

```typescript
function narrow(value: string | number | { name: string }) {
  if (typeof value === "string") {
    // value: string
  }
  if (typeof value === "number") {
    // value: number
  }
  if ("name" in value) {
    // value: { name: string }
  }
}
```

### Why It Is Important

Without type narrowing, you could not safely access type-specific properties on union types. Narrowing is what makes union types practical—it gives you the flexibility of dynamic typing with the safety of static typing. TypeScript's narrowing is smart enough to handle `typeof`, `in`, `instanceof`, equality checks, truthiness checks, and user-defined type predicates.

### How It Works Internally

TypeScript uses a control flow graph (CFG) to track types at each program point. When the compiler encounters a type guard expression like `typeof x === "string"`, it computes the *narrowed type* by filtering the original union. For `typeof`, it recognizes the specific return values: `"string"`, `"number"`, `"boolean"`, `"symbol"`, `"bigint"`, `"undefined"`, `"object"`, `"function"`. The `in` operator narrows based on property existence.

```typescript
// Compiler tracks: after typeof check, removes incompatible types
// Before: string | number
// After typeof === "string": string
// After typeof === "number": number
```

### Syntax

```typescript
// typeof narrowing
if (typeof value === "string") { }
if (typeof value !== "number") { }

// in narrowing
if ("property" in value) { }
if (!("property" in value)) { }

// Truthiness narrowing
if (value) { }

// Equality narrowing
if (value === "specific-value") { }

// Discriminated union narrowing
if (value.kind === "circle") { }
```

### Beginner Examples

```typescript
function describe(value: string | number | boolean): string {
  if (typeof value === "string") {
    return `String of length ${value.length}`;
  }
  if (typeof value === "number") {
    return `Number: ${value.toFixed(2)}`;
  }
  return `Boolean: ${value}`;
}

function printCoordinates(p: { x: number; y: number } | { lat: number; lng: number }) {
  if ("x" in p) {
    console.log(p.x, p.y);
  } else {
    console.log(p.lat, p.lng);
  }
}
```

### Intermediate Examples

```typescript
interface Fish { swim: () => void; layEggs: () => void; }
interface Bird { fly: () => void; layEggs: () => void; }

function move(pet: Fish | Bird) {
  if ("swim" in pet) {
    return pet.swim();
  }
  return pet.fly();
}

// Combining typeof with truthiness
function getLength(value: string | null | undefined): number {
  if (typeof value === "string") {
    return value.length;
  }
  return 0;
}

// Narrowing with else if chains
function classify(value: unknown): string {
  if (typeof value === "string") return "string";
  else if (typeof value === "number") return "number";
  else if (typeof value === "boolean") return "boolean";
  else if (value === null) return "null";
  else if (Array.isArray(value)) return "array";
  else if (typeof value === "object") return "object";
  return "unknown";
}
```

### Advanced Examples

```typescript
// Nested narrowing with type predicates
function isString(value: unknown): value is string {
  return typeof value === "string";
}

// Exhaustive narrowing with never
function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${value}`);
}

type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "triangle"; base: number; height: number };

function area(shape: Shape): number {
  if (shape.kind === "circle") return Math.PI * shape.radius ** 2;
  if (shape.kind === "square") return shape.side ** 2;
  if (shape.kind === "triangle") return (shape.base * shape.height) / 2;
  return assertNever(shape);
}

// Combining typeof with conditional types
type IsString<T> = T extends string ? true : false;
type Check = IsString<typeof "hello">; // true
```

### Real-World Use Cases

- **Form validation**: Narrowing form field values based on field type.
- **API response handling**: Differentiating between error and success responses.
- **Configuration merging**: Handling both partial and complete config objects.
- **Event handling**: Differentiating between mouse, keyboard, and touch events.
- **Database queries**: Handling different query operators and filters.

```typescript
// Real-world: API response
type APIResponse<T> =
  | { status: "success"; data: T }
  | { status: "error"; message: string };

function handleResponse<T>(response: APIResponse<T>): T | never {
  if (response.status === "success") return response.data;
  throw new Error(response.message);
}
```

### Common Mistakes

1. **Using `typeof` for objects**: `typeof null` returns `"object"`, and `typeof []` also returns `"object"`. Use `Array.isArray()` or `=== null`.
2. **Forgetting that `typeof` returns a string literal**: The comparison must be with the exact string: `typeof x === "object"`.
3. **Using `in` with primitives**: The `in` operator can be used with primitives but checks the prototype chain.
4. **Relying on `typeof` for classes**: Use `instanceof` instead.
5. **Not narrowing before accessing properties**: Accessing `.length` on `string | number` without a guard is an error.

### Best Practices

- Prefer discriminated unions with a literal property over `typeof`/`in` when possible—they are more explicit.
- Use `typeof` for primitives, `instanceof` for classes, `in` for object properties.
- Combine narrowing with early returns to reduce nesting.
- Use `unknown` instead of `any` when you need to narrow from a totally unknown type.
- Always handle the `never` case in exhaustive switches.

### Performance Considerations

Type narrowing is purely a compile-time concept with zero runtime cost. However, the compiler's narrowing analysis has complexity proportional to the number of branches and union members. Very complex nested narrowing (many levels or large unions) can slow down compilation. For runtime performance, type guards like `typeof` and `in` are fast operations, but user-defined type predicates may involve expensive computation.

### Interview Questions

1. **Q**: What is the difference between `typeof` and `instanceof` for narrowing?
   **A**: `typeof` works with primitive types and returns a string. `instanceof` checks the prototype chain and works with class instances.

2. **Q**: How does TypeScript narrow in a `switch` statement?
   **A**: TypeScript narrows the type in each `case` branch based on the case expression. For discriminated unions with a literal property, it narrows to the specific variant.

3. **Q**: Can narrowing be undone?
   **A**: Yes, if you assign a value of the wider type to the variable, or if control flow merges (e.g., after an if-else), the type reverts to the union.

### Coding Challenges

1. Write a `deepClone` function that narrows correctly for arrays, objects, and primitives.
2. Implement a type-safe router that narrows URL parameters based on the route pattern.
3. Create a function that processes a union of different database query types.

### Related Topics

- Type guards
- Discriminated unions
- Control flow analysis
- User-defined type predicates
- `asserts` keyword

## Discriminated unions

### What It Is

A discriminated union (also called a tagged union or algebraic data type) is a pattern where each member of a union type shares a common property—the discriminant—with a literal type. TypeScript uses this discriminant to narrow the type when you check it, giving you access to the specific member's properties.

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; sideLength: number }
  | { kind: "triangle"; base: number; height: number };
```

### Why It Is Important

Discriminated unions are the standard pattern for modeling data that can take multiple distinct forms. They enable exhaustive pattern matching, make impossible states impossible to represent, and provide excellent developer experience with autocompletion and type safety. They are the bread and butter of state management, protocol handling, and form logic.

### How It Works Internally

The compiler identifies discriminated unions by looking for union types where all members have a property of the same name with a literal type. When you check this discriminant (e.g., `shape.kind === "circle"`), the compiler computes the narrowed type by intersecting the original type with `{ kind: "circle" }`, effectively removing incompatible members.

```typescript
// TypeScript internally:
// Shape & { kind: "circle" } = { kind: "circle"; radius: number }
```

### Syntax

```typescript
// Basic discriminated union
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string };

// Using different property name
type RequestState =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: unknown }
  | { status: "error"; error: unknown };

// Multiple discriminants
type Event =
  | { type: "click"; x: number; y: number }
  | { type: "keypress"; key: string; ctrlKey: boolean }
  | { type: "focus" };

// String literal vs enum discriminant
enum State { IDLE, LOADING, SUCCESS, ERROR }
type AsyncState = 
  | { state: State.IDLE }
  | { state: State.LOADING }
  | { state: State.SUCCESS; data: unknown };
```

### Beginner Examples

```typescript
function calculateArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    case "triangle":
      return (shape.base * shape.height) / 2;
  }
}

type PaymentMethod =
  | { type: "credit"; cardNumber: string; cvv: string }
  | { type: "paypal"; email: string }
  | { type: "bank"; accountNumber: string; routingNumber: string };

function processPayment(method: PaymentMethod): void {
  if (method.type === "credit") {
    // method.cardNumber available
  } else if (method.type === "paypal") {
    // method.email available
  }
}
```

### Intermediate Examples

```typescript
// Generic discriminated union
type AsyncOp<T> =
  | { tag: "pending" }
  | { tag: "success"; value: T }
  | { tag: "failure"; error: Error };

function mapAsync<T, U>(op: AsyncOp<T>, fn: (t: T) => U): AsyncOp<U> {
  if (op.tag === "success") {
    return { tag: "success", value: fn(op.value) };
  }
  return op;
}

// Nested discriminated unions
type NetworkEvent =
  | { type: "connection"; status: "connected" | "disconnected" }
  | { type: "message"; payload: string }
  | { type: "error"; code: number; message: string };

// Discriminated union with optional discriminant
type Action =
  | { type: "increment"; amount: number }
  | { type: "decrement"; amount: number }
  | { type: "reset"; value?: number };
```

### Advanced Examples

```typescript
// Polymorphic discriminated union with generic discriminant
type Schema<T extends string> = { type: T; validate: (value: unknown) => value is infer; };
// Cannot infer across union; use discriminated pattern:

type StringSchema = { type: "string"; minLength?: number; maxLength?: number };
type NumberSchema = { type: "number"; min?: number; max?: number };
type BooleanSchema = { type: "boolean" };
type Schema = StringSchema | NumberSchema | BooleanSchema;

// Redux-style reducer with discriminated union
type CounterAction =
  | { type: "increment"; payload: number }
  | { type: "decrement"; payload: number }
  | { type: "reset" };

interface CounterState { count: number }

function counterReducer(state: CounterState, action: CounterAction): CounterState {
  switch (action.type) {
    case "increment":
      return { count: state.count + action.payload };
    case "decrement":
      return { count: state.count - action.payload };
    case "reset":
      return { count: 0 };
  }
}

// Builder pattern with discriminated union
type QueryBuilder =
  | { op: "select"; fields: string[] }
  | { op: "where"; field: string; operator: "eq" | "gt" | "lt"; value: unknown }
  | { op: "limit"; count: number }
  | { op: "orderBy"; field: string; direction: "asc" | "desc" };
```

### Real-World Use Cases

- **State machines**: UI states (idle, loading, success, error).
- **Form validation**: Each field validator returns a discriminated result.
- **Command pattern**: Different commands in a CQRS system.
- **WebSocket messages**: Different message types on a socket connection.
- **Compiler AST nodes**: Each node type has a distinct structure.
- **Database query builders**: Chaining different query operations.

```typescript
// Real-world: WebSocket message handler
type WSMessage =
  | { event: "join"; room: string; user: string }
  | { event: "leave"; room: string; user: string }
  | { event: "message"; room: string; user: string; text: string }
  | { event: "typing"; room: string; user: string };

function handleMessage(ws: WebSocket, msg: WSMessage): void {
  switch (msg.event) {
    case "join": ws.join(msg.room); break;
    case "leave": ws.leave(msg.room); break;
    case "message": broadcast(msg.room, msg); break;
    case "typing": notifyTyping(msg.room, msg.user); break;
  }
}
```

### Common Mistakes

1. **Using non-literal types as discriminants**—the discriminant property must have a literal type.
2. **Missing the discriminant property** in some union members.
3. **Using optional discriminant properties**—they should be required.
4. **Not using `switch`/exhaustive checking** and missing new variants.
5. **Mutating the discriminant** after object creation.

```typescript
// Mistake: non-literal discriminant
type Bad = { type: string; data: string } | { type: string; value: number };
// type is just string, not a literal → no narrowing

// Mistake: optional discriminant
type Bad2 = { type?: "a"; x: number } | { type?: "b"; y: number };
```

### Best Practices

- Use a single well-named discriminant property (convention: `kind`, `type`, `tag`, `status`).
- Make the discriminant required and a literal string type.
- Use `switch` statements for exhaustiveness checking.
- Keep the number of variants manageable (3–7 is ideal).
- Combine with `never` for the default case to get compile-time safety.

### Performance Considerations

Zero runtime cost—discriminant checks are just property comparisons. However, be mindful that very large discriminated unions (10+ members) can cause compiler slowdowns. In performance-critical paths, consider flattening deeply nested discriminated unions.

### Interview Questions

1. **Q**: What makes a union "discriminated"?
   **A**: A union is discriminated when all members share a common property of a literal type (e.g., `{ kind: "circle" }` vs `{ kind: "square" }`).

2. **Q**: Can a discriminated union have more than one discriminant?
   **A**: Yes, but only one property is typically used for narrowing. Multiple literal properties can be used for more specific narrowing.

3. **Q**: How is a discriminated union different from `switch` on a string?
   **A**: Without discriminated unions, TypeScript cannot narrow the type within each case. With discriminated unions, each case branch knows the exact type.

### Coding Challenges

1. Model a traffic light system as a discriminated union with 3 states.
2. Implement a reducer for a shopping cart using discriminated unions.
3. Create a discriminated union for a WebSocket message protocol with at least 5 message types.

### Related Topics

- Union types
- Exhaustiveness checking
- `never` type
- Pattern matching (proposal)
- Algebraic data types

## Exhaustiveness checking with never

### What It Is

Exhaustiveness checking ensures that every variant of a union type is handled. In TypeScript, the `never` type is used for this: you assign the value to a variable of type `never` in the default branch. If any union member is unhandled, TypeScript raises a type error.

```typescript
function assertNever(value: never): never {
  throw new Error(`Unhandled variant: ${value}`);
}
```

### Why It Is Important

Exhaustiveness checking prevents a class of bugs where new union members are added but existing switch statements or conditional chains are not updated. Without it, adding a new variant silently leads to runtime errors. With it, the compiler immediately points to every location that needs updating.

### How It Works Internally

TypeScript's control flow analysis determines the type of a variable at a given program point. In a `switch` statement, after all `case` branches, the type of the switch expression in the default branch is the union of members not yet handled. If all members are handled, this type is `never`. If you assign a `never`-typed value to another `never` variable, it succeeds. If any member remains, the type is not `never`, and the assignment fails.

```typescript
// Internally: after switch on Shape
// If all cases handled: type is never
// If "triangle" not handled: type is { kind: "triangle"; ... }
```

### Syntax

```typescript
// Default exhaustiveness function
function exhaustive(value: never): never {
  throw new Error(`Exhaustive check failed: ${value}`);
}

// Usage in switch
function area(s: Shape): number {
  switch (s.kind) {
    case "circle": return Math.PI * s.radius ** 2;
    case "square": return s.side ** 2;
    case "triangle": return (s.base * s.height) / 2;
    default: return exhaustive(s);
  }
}

// Usage with if/else
function area2(s: Shape): number {
  if (s.kind === "circle") return Math.PI * s.radius ** 2;
  if (s.kind === "square") return s.side ** 2;
  if (s.kind === "triangle") return (s.base * s.height) / 2;
  return exhaustive(s);
}

// Inline exhaustiveness
function area3(s: Shape): number {
  switch (s.kind) {
    case "circle": return Math.PI * s.radius ** 2;
    case "square": return s.side ** 2;
    case "triangle": return (s.base * s.height) / 2;
    default: {
      const _: never = s;
      return _;
    }
  }
}
```

### Beginner Examples

```typescript
type Color = "red" | "green" | "blue";

function getHex(color: Color): string {
  switch (color) {
    case "red": return "#FF0000";
    case "green": return "#00FF00";
    case "blue": return "#0000FF";
    default: {
      const _exhaustive: never = color;
      return _exhaustive;
    }
  }
}

type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";

function handleRequest(method: HttpMethod): string {
  if (method === "GET") return "fetching";
  if (method === "POST") return "creating";
  if (method === "PUT") return "updating";
  if (method === "DELETE") return "deleting";
  throw new Error(`Unknown method: ${method}`);
  // Without the throw, TypeScript warns this returns undefined
}
```

### Intermediate Examples

```typescript
// Exhaustiveness with generic discriminated union
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

function unwrap<T, E>(result: Result<T, E>): T {
  if (result.ok) return result.value;
  throw result.error;
}

// Multiple variants needing exhaustive handling
type Op =
  | { op: "add"; a: number; b: number }
  | { op: "subtract"; a: number; b: number }
  | { op: "multiply"; a: number; b: number }
  | { op: "divide"; a: number; b: number };

function calculate(op: Op): number {
  switch (op.op) {
    case "add": return op.a + op.b;
    case "subtract": return op.a - op.b;
    case "multiply": return op.a * op.b;
    case "divide": return op.a / op.b;
  }
}
```

### Advanced Examples

```typescript
// Exhaustiveness with conditional types
type IsNever<T> = [T] extends [never] ? true : false;

// Type-safe event emitter with exhaustiveness
type EventMap = {
  click: { x: number; y: number };
  focus: void;
  blur: void;
  keydown: { key: string };
};

type EventName = keyof EventMap;

function emit<K extends EventName>(name: K, payload: EventMap[K]): void {
  // Implementation
}

// Exhaustive handler builder
type HandlerMap<T extends string> = { [K in T]: () => void };

function createHandler<T extends string>(
  handlers: HandlerMap<T>
): (name: T) => void {
  return (name) => handlers[name]();
}

// Phantom type for exhaustive matching
type MatchResult<T, R> = { [K in keyof T]: R }[keyof T];

function match<T extends Record<string, unknown>, R>(
  value: T[keyof T],
  handlers: { [K in keyof T]: (v: Extract<T[keyof T], T[K]>) => R }
): R {
  // Advanced pattern matching
}
```

### Real-World Use Cases

- **Adding new features**: When you add a new variant to a backend's response type, exhaustiveness checking shows every frontend handler that needs updating.
- **Protocol evolution**: When a WebSocket protocol gains a new message type, the compiler flags all message handlers.
- **Form field types**: Adding a new field type forces you to implement validation, rendering, and parsing for it.
- **Database migration**: Adding a new entity type with a discriminated union shows every place that needs migration logic.

```typescript
// Real-world: Form field system
type Field =
  | { kind: "text"; value: string }
  | { kind: "number"; value: number }
  | { kind: "checkbox"; checked: boolean }
  | { kind: "select"; value: string; options: string[] };

function renderField(field: Field): string {
  switch (field.kind) {
    case "text": return `<input type="text" value="${field.value}" />`;
    case "number": return `<input type="number" value="${field.value}" />`;
    case "checkbox": return `<input type="checkbox" ${field.checked ? "checked" : ""} />`;
    case "select":
      return `<select>${field.options.map(o =>
        `<option ${o === field.value ? "selected" : ""}>${o}</option>`
      ).join("")}</select>`;
  }
}
```

### Common Mistakes

1. **Not having a default/else branch** with the `never` assertion.
2. **Using `return` in every case** but forgetting the default—TypeScript might infer `undefined` in the return type.
3. **Throwing a generic error** instead of using `never`—you lose the compile-time safety.
4. **Forgetting to handle `undefined` or `null`** in the union.
5. **Adding a new variant but the function already has a default** that returns a fallback—the bug goes undetected.

### Best Practices

- Always use `assertNever` or an inline `never` assignment in the default branch.
- Use `switch` statements (not if/else chains) for exhaustive matching—they are more readable and the compiler handles them better.
- Keep the exhaustive check as the last resort, not a fallback value.
- When adding a new variant, the compiler will point to every place that needs updating. Let it guide you.
- Name your exhaustive function clearly, e.g., `assertNever` or `exhaustiveCheck`.

### Performance Considerations

The `never` type is erased at runtime. The `assertNever` function is never called if your code is correct. If it *is* called, it throws—which is the intended failure behavior. There is no performance overhead for using this pattern.

### Interview Questions

1. **Q**: How does TypeScript know that a switch statement is exhaustive?
   **A**: After the last `case`, the compiler checks the type of the value in the default branch. If all union members were handled, the type is `never`. Otherwise it's the remaining members.

2. **Q**: What happens if you skip the default branch entirely?
   **A**: TypeScript won't error, but the function might implicitly return `undefined` if it doesn't return in every case. The exhaustiveness guarantee is lost.

3. **Q**: Can you use exhaustiveness checking with if/else chains?
   **A**: Yes, but it's more verbose. You need a final else branch with the `never` assertion. Switch is preferred.

### Coding Challenges

1. Add a new variant to `Shape` (e.g., `rectangle`) and find all the exhaustive checks that break.
2. Write a generic `exhaustiveMatch` function that takes a value and a handler map and returns the correct type.
3. Create a system where adding a new user permission requires updating all permission checks.

### Related Topics

- `never` type
- Discriminated unions
- Control flow analysis
- Switch statements
- Pattern matching
