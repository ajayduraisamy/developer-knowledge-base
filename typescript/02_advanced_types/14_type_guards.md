# Type Guards - typeof, instanceof, in, user-defined guards, type predicates, asserts

## Introduction

Type guards are expressions that perform runtime type checks and inform the TypeScript compiler about the narrowed type within a code branch. They bridge the gap between runtime and compile-time, allowing TypeScript to understand dynamic type information. Without type guards, the compiler would have no way to know that a `typeof` check eliminates certain union members.

TypeScript provides several built-in type guards (`typeof`, `instanceof`, `in`) and allows you to create user-defined type guards using type predicates (`value is Type`) and assertion functions (`asserts value is Type`). Mastering type guards is essential for writing type-safe code that works with union types, dynamic data, and runtime type checks.

## typeof type guard

### What It Is

The `typeof` operator returns a string indicating the type of its operand at runtime. TypeScript recognizes `typeof` checks as type guards and narrows union types accordingly. It works with primitive types: `"string"`, `"number"`, `"boolean"`, `"symbol"`, `"bigint"`, `"undefined"`, `"object"`, `"function"`.

```typescript
if (typeof value === "string") {
  // value is string here
}
```

### Why It Is Important

`typeof` is the simplest and most common type guard for primitives. It is universally available, works with any value, and has predictable behavior. It is the first line of defense when dealing with union types that include primitives.

### How It Works Internally

The compiler recognizes `typeof x === "string"` and `typeof x !== "string"` patterns. When it encounters such a check, it filters the union type: any member whose runtime `typeof` would not produce the specified string is eliminated. TypeScript knows that `"string"` is the return type of `typeof` for `string`, `"number"` for `number`, etc.

```typescript
// Before: string | number
// After typeof === "string": string
// After typeof === "number": number
```

### Syntax

```typescript
if (typeof value === "string") { }
if (typeof value !== "number") { }
if (typeof value === "object") { }
if (typeof value === "undefined") { }
if (typeof value === "function") { }
```

### Beginner Examples

```typescript
function print(value: string | number): void {
  if (typeof value === "string") {
    console.log(value.toUpperCase());
  } else {
    console.log(value.toFixed(2));
  }
}

function double(value: string | number): string | number {
  if (typeof value === "string") {
    return value + value;
  }
  return value * 2;
}

function isDefined<T>(value: T | undefined): value is T {
  return typeof value !== "undefined";
}
```

### Intermediate Examples

```typescript
function formatDate(value: string | number | Date): string {
  if (typeof value === "string") {
    return new Date(value).toISOString();
  }
  if (typeof value === "number") {
    return new Date(value).toISOString();
  }
  return value.toISOString();
}

function processUnknown(value: unknown): string {
  if (typeof value === "string") return `String: ${value}`;
  if (typeof value === "number") return `Number: ${value}`;
  if (typeof value === "boolean") return `Boolean: ${value}`;
  if (typeof value === "undefined") return "Undefined";
  if (value === null) return "Null";
  return "Complex type";
}

type Primitive = string | number | boolean | null | undefined;
function isPrimitive(value: unknown): value is Primitive {
  switch (typeof value) {
    case "string":
    case "number":
    case "boolean":
    case "undefined":
      return true;
    case "object":
      return value === null;
    default:
      return false;
  }
}
```

### Advanced Examples

```typescript
// typeof with generic constraint
function clone<T>(value: T): T {
  if (typeof value === "object" && value !== null) {
    return JSON.parse(JSON.stringify(value));
  }
  return value;
}

// typeof in type predicates
function isString(value: unknown): value is string {
  return typeof value === "string";
}
function isNumber(value: unknown): value is number {
  return typeof value === "number" && !isNaN(value);
}

// typeof with mapped types
type TypeOfMap = {
  string: string;
  number: number;
  boolean: boolean;
  object: object;
  undefined: undefined;
  function: Function;
};

function createTyped<T extends keyof TypeOfMap>(type: T, value: TypeOfMap[T]): void {}

// typeof narrowing in array filter
const mixed: (string | number)[] = ["a", 1, "b", 2];
const strings: string[] = mixed.filter((x): x is string => typeof x === "string");
const numbers: number[] = mixed.filter((x): x is number => typeof x === "number");
```

### Real-World Use Cases

- **Form validation**: Checking the type of user input.
- **API response processing**: Handling different response formats.
- **Configuration loading**: Processing environment variables (always strings) into typed values.
- **Serialization/deserialization**: Converting values based on their runtime type.
- **Logging**: Formatting values differently based on their type.

```typescript
// Real-world: environment variable parser
function parseEnv<T>(value: string | undefined, parser: (s: string) => T, fallback: T): T {
  if (typeof value === "undefined") return fallback;
  try {
    return parser(value);
  } catch {
    return fallback;
  }
}

const port = parseEnv(process.env.PORT, parseInt, 3000);
```

### Common Mistakes

1. **Using `typeof` for `null`**: `typeof null` returns `"object"`, not `"null"`. Always check `value === null` separately.
2. **Using `typeof` for arrays**: `typeof []` returns `"object"`. Use `Array.isArray()` instead.
3. **Forgetting `typeof` returns lowercase strings**: `"string"`, not `"String"`.
4. **Confusing `typeof` type guards with runtime behavior**: The guard narrows the TypeScript type, it doesn't change the value.
5. **Using `typeof` for class instances**: Use `instanceof` instead.

### Best Practices

- Use `typeof` for primitive types only.
- Always check for `null` separately when using `typeof value === "object"`.
- Use `Array.isArray()` for array checks.
- Combine `typeof` with user-defined type predicates for complex logic.
- Prefer `typeof` over `instanceof` for primitives.

### Performance Considerations

`typeof` is one of the fastest runtime operations in JavaScript. The type-narrowing effect is compile-time only with no runtime overhead.

### Interview Questions

1. **Q**: What does `typeof null` return?
   **A**: `"object"`. This is a well-known JavaScript bug. Always check `value === null` separately.

2. **Q**: Can `typeof` distinguish between different object types?
   **A**: No. All objects return `"object"`. Use `instanceof` or property checks for object types.

### Coding Challenges

1. Implement a `deepClone` function that handles primitives, objects, and arrays using `typeof` guards.
2. Write a type-safe `localStorage` wrapper that uses `typeof` to parse stored values.

### Related Topics

- User-defined type predicates
- `instanceof` type guard
- `unknown` type
- Primitive types

## instanceof type guard

### What It Is

The `instanceof` operator checks whether an object is an instance of a particular class or constructor function. TypeScript uses `instanceof` checks as type guards to narrow the type of a variable to the specific class's type.

```typescript
if (value instanceof Date) {
  // value: Date
}
```

### Why It Is Important

`instanceof` is essential for working with class hierarchies and built-in object types. It allows you to safely access class-specific methods and properties after checking the type. It is the standard way to narrow class union types.

### How It Works Internally

TypeScript narrows based on the prototype chain check. When `x instanceof Foo` passes, TypeScript narrows `x` to `Foo` (or a subtype of `Foo`). If a union type contains `Foo` and other types, the incompatible types are removed.

```typescript
// typeof: checks primitive type tag
// instanceof: checks prototype chain
```

### Syntax

```typescript
if (value instanceof Date) { }
if (value instanceof Error) { }
if (value instanceof RegExp) { }
if (value instanceof Map) { }
if (value instanceof Set) { }
if (!(value instanceof Array)) { }
```

### Beginner Examples

```typescript
function getTime(value: Date | string): number {
  if (value instanceof Date) {
    return value.getTime();
  }
  return new Date(value).getTime();
}

function formatError(error: Error | string): string {
  if (error instanceof Error) {
    return `${error.name}: ${error.message}`;
  }
  return error;
}

class APIError extends Error {
  constructor(public statusCode: number, message: string) {
    super(message);
  }
}

function handleError(err: Error): void {
  if (err instanceof APIError) {
    console.log(`API Error ${err.statusCode}: ${err.message}`);
  } else {
    console.log(`Error: ${err.message}`);
  }
}
```

### Intermediate Examples

```typescript
class User {
  constructor(public name: string, public email: string) {}
}

class Admin {
  constructor(public name: string, public permissions: string[]) {}
}

function getDisplayName(person: User | Admin): string {
  if (person instanceof User) {
    return `${person.name} (${person.email})`;
  }
  return `${person.name} [Admin: ${person.permissions.join(", ")}]`;
}

// instanceof with built-in types
function cloneValue<T>(value: T): T {
  if (value instanceof Date) return new Date(value.getTime()) as T;
  if (value instanceof Map) return new Map(value) as T;
  if (value instanceof Set) return new Set(value) as T;
  if (Array.isArray(value)) return value.map(cloneValue) as T;
  if (typeof value === "object" && value !== null) {
    return JSON.parse(JSON.stringify(value));
  }
  return value;
}
```

### Advanced Examples

```typescript
// instanceof with generic classes
class Box<T> {
  constructor(public value: T) {}
}

function unbox<T>(box: unknown): T | undefined {
  if (box instanceof Box) {
    return box.value;
  }
  return undefined;
}

// instanceof with mixins
class Timestamped {
  createdAt = new Date();
}

function isTimestamped(obj: unknown): obj is Timestamped {
  return obj instanceof Timestamped;
}

// instanceof for discriminated class hierarchies
abstract class Shape {
  abstract area(): number;
}

class Circle extends Shape {
  constructor(public radius: number) { super(); }
  area(): number { return Math.PI * this.radius ** 2; }
}

class Square extends Shape {
  constructor(public side: number) { super(); }
  area(): number { return this.side ** 2; }
}

function totalArea(shapes: Shape[]): number {
  return shapes.reduce((sum, shape) => {
    if (shape instanceof Circle) {
      return sum + shape.area();
    }
    if (shape instanceof Square) {
      return sum + shape.area();
    }
    return sum;
  }, 0);
}

// instanceof with Symbol.hasInstance
class MyArray<T> extends Array<T> {
  static [Symbol.hasInstance](instance: unknown): boolean {
    return Array.isArray(instance);
  }
}
```

### Real-World Use Cases

- **Error handling**: Differentiating between custom error types.
- **ORM entity hydration**: Checking entity types after database queries.
- **Serialization**: Handling different serialized formats.
- **Plugin systems**: Checking plugin instance types.
- **Command/query objects**: Routing to correct handlers based on class type.

### Common Mistakes

1. **Using `instanceof` with primitives**: `"hello" instanceof String` returns `false`.
2. **Forgetting cross-frame/realm issues**: `instanceof` fails across different iframes or Node.js `vm` contexts because constructors are different.
3. **Using `instanceof` with interfaces**: Interfaces are erased at runtime. Use discriminated unions instead.
4. **Assuming `instanceof` checks the exact class**: It checks the prototype chain, so subclasses also match.

### Best Practices

- Use `instanceof` for class instances, `typeof` for primitives.
- Prefer discriminated unions over `instanceof` for type hierarchies that don't need behavior (methods).
- Be aware of cross-realm issues; use duck typing (`in` guard) as an alternative.
- Combine `instanceof` with custom type predicates for complex checks.

### Performance Considerations

`instanceof` involves walking the prototype chain, which is O(n) in the depth of inheritance. For most hierarchies this is negligible. For performance-critical code with deep hierarchies, consider using a discriminant property instead.

### Interview Questions

1. **Q**: What is the difference between `typeof` and `instanceof`?
   **A**: `typeof` returns a string for primitives and is fast. `instanceof` checks prototype chains and works for class instances.

2. **Q**: Does `instanceof` check the exact class or the prototype chain?
   **A**: It checks the entire prototype chain. `child instanceof Parent` is `true` if `Parent` appears anywhere in the prototype chain.

### Coding Challenges

1. Implement a custom error hierarchy and write a function that formats different error types.
2. Create a type-safe event emitter that uses `instanceof` to route events to correct handlers.

### Related Topics

- Class types
- Prototype chain
- `typeof` type guard
- `in` operator guard

## in operator guard

### What It Is

The `in` operator checks whether a property exists on an object. TypeScript uses `in` checks to narrow union types by property presence. If a property exists only on some members of a union, checking for it narrows the type to those members.

```typescript
if ("swim" in animal) {
  // animal has swim method
}
```

### Why It Is Important

The `in` operator is the primary way to narrow object types that don't have a discriminant property. It enables duck typing—checking for the presence of properties rather than relying on explicit type tags. It is particularly useful when working with third-party types or evolving interfaces.

### How It Works Internally

When TypeScript sees `"prop" in x`, it narrows `x` by removing union members where `prop` is not a known property. If all members have `prop` (possibly optional), the narrowing effect is limited. The compiler uses the type's structural information to determine which members have the property.

```typescript
type A = { a: string };
type B = { b: number };
// "a" in value → narrows to A
// "b" in value → narrows to B
```

### Syntax

```typescript
if ("propertyName" in value) { }
if (!("propertyName" in value)) { }

// With computed property
const key = "name";
if (key in value) { }

// In ternary
const type = "fly" in bird ? "flying" : "non-flying";
```

### Beginner Examples

```typescript
interface Fish { swim: () => void; }
interface Bird { fly: () => void; }

function move(animal: Fish | Bird): void {
  if ("swim" in animal) {
    animal.swim();
  } else {
    animal.fly();
  }
}

interface Car { drive: () => void; }
interface Boat { sail: () => void; }

function operate(vehicle: Car | Boat): void {
  if ("drive" in vehicle) {
    vehicle.drive();
  } else {
    vehicle.sail();
  }
}

// Checking optional properties
interface Config { url: string; timeout?: number; }
interface OtherConfig { endpoint: string; retries?: number; }

function process(config: Config | OtherConfig): void {
  if ("url" in config) {
    console.log(config.url);
  } else {
    console.log(config.endpoint);
  }
}
```

### Intermediate Examples

```typescript
interface Circle { kind: "circle"; radius: number; }
interface Square { kind: "square"; side: number; }
type Shape = Circle | Square;

function getArea(shape: Shape): number {
  // 'in' works with discriminant too
  if ("radius" in shape) {
    return Math.PI * shape.radius ** 2;
  }
  return shape.side ** 2;
}

// 'in' with generic types
function hasProperty<T extends object, K extends string>(
  obj: T,
  prop: K
): obj is T & Record<K, unknown> {
  return prop in obj;
}

interface User { name: string; email: string; }
interface Admin extends User { role: "admin"; }

function isAdmin(user: User): user is Admin {
  return "role" in user;
}

// Dynamic property checking
function getNestedValue(obj: Record<string, unknown>, path: string[]): unknown {
  let current: unknown = obj;
  for (const key of path) {
    if (typeof current === "object" && current !== null && key in current) {
      current = (current as Record<string, unknown>)[key];
    } else {
      return undefined;
    }
  }
  return current;
}
```

### Advanced Examples

```typescript
// 'in' with mapped types
type WithRequiredProperty<T, K extends keyof T> = T & { [P in K]-?: T[P] };

function ensureProperty<T, K extends keyof T>(
  obj: T,
  key: K
): WithRequiredProperty<T, K> {
  if (!(key in obj)) {
    (obj as any)[key] = undefined;
  }
  return obj as WithRequiredProperty<T, K>;
}

// 'in' for exhaustive checking
function exhaustiveCheck(value: never): never {
  throw new Error(`Unhandled type: ${JSON.stringify(value)}`);
}

// Combining 'in' with template literal types
type EventHandlers = {
  onClick: () => void;
  onHover: () => void;
  onFocus: () => void;
};

function hasHandler<T extends string>(
  handlers: Partial<EventHandlers>,
  name: T
): name is T & keyof EventHandlers {
  return name in handlers;
}

// 'in' with union of intersection types
type RequestBase = { url: string; method: string };
type GetRequest = RequestBase & { method: "GET" };
type PostRequest = RequestBase & { method: "POST"; body: unknown };

function isPostRequest(req: GetRequest | PostRequest): req is PostRequest {
  return "body" in req;
}
```

### Real-World Use Cases

- **Duck typing**: Checking interfaces without explicit discriminants.
- **Dynamic configuration**: Checking which config options are set.
- **Plugin systems**: Detecting plugin capabilities by property presence.
- **Form validation**: Checking which fields have errors.
- **API response shaping**: Determining response structure by property presence.

```typescript
// Real-world: Duck typing for plugin capabilities
interface LoggerPlugin { log: (msg: string) => void; }
interface ProfilerPlugin { start: () => void; end: () => number; }
interface CachePlugin { get: (key: string) => unknown; set: (key: string, value: unknown) => void; }

type Plugin = Record<string, unknown>;

function hasCapability<T extends string>(
  plugin: Plugin,
  capability: T
): plugin is Plugin & Record<T, (...args: any[]) => any> {
  return capability in plugin && typeof plugin[capability] === "function";
}
```

### Common Mistakes

1. **Using `in` with primitives**: `"length" in "hello"` throws an error (primitives don't work with `in` unless wrapped).
2. **Assuming `in` checks own properties**: `in` checks the entire prototype chain.
3. **Forgetting that `in` with a non-literal string doesn't narrow**: `key in obj` only works when `key` is a string literal.
4. **Using `in` with `null` or `undefined`**: This throws a runtime error. Always check for nullish first.

### Best Practices

- Use `in` for duck typing when you don't control the types.
- Prefer discriminated unions with literal discriminants over `in` for types you control.
- Always guard against `null`/`undefined` before using `in`.
- Use `in` with string literals, not variables, for narrowing.

### Performance Considerations

The `in` operator checks property existence via the prototype chain. It's O(depth of prototype chain). For flat objects this is O(1). The narrowing effect is compile-time only.

### Interview Questions

1. **Q**: Does `in` check own properties or inherited ones?
   **A**: Both. `in` checks the entire prototype chain. Use `hasOwnProperty` for own properties only.

2. **Q**: When would you prefer `in` over a discriminated union?
   **A**: When you don't control the types (e.g., third-party libraries) or when adding a discriminant would be intrusive.

### Coding Challenges

1. Write a function that deep-merges two objects using `in` to check property existence.
2. Implement a type-safe plugin system that uses `in` to check plugin capabilities.

### Related Topics

- `hasOwnProperty`
- Discriminated unions
- Duck typing
- Type narrowing

## User-defined type predicates

### What It Is

A user-defined type predicate is a function that returns a boolean and tells TypeScript that, when it returns `true`, the argument is of a specific type. The syntax is `parameterName is Type` as the return type annotation.

```typescript
function isString(value: unknown): value is string {
  return typeof value === "string";
}
```

### Why It Is Important

User-defined type predicates allow you to encapsulate complex type-checking logic in reusable functions. They extend TypeScript's narrowing capabilities beyond built-in guards, enabling type-safe validation for custom interfaces, complex structures, and domain-specific types.

### How It Works Internally

When the compiler encounters `if (isString(x))`, it checks whether `isString` has a type predicate return type (`x is string`). If so, it narrows the type of `x` in the true branch to the specified type. In the false branch, it removes that type. The compiler trusts the predicate function—it does not verify the implementation.

```typescript
// The compiler takes your word for it.
// Always ensure your implementation is correct.
```

### Syntax

```typescript
// Basic predicate
function isString(value: unknown): value is string {
  return typeof value === "string";
}

// Predicate with specific type
interface User { name: string; }
function isUser(value: unknown): value is User {
  return typeof value === "object" && value !== null && "name" in value;
}

// Multiple predicates
function isStringOrNumber(value: unknown): value is string | number {
  return typeof value === "string" || typeof value === "number";
}
```

### Beginner Examples

```typescript
function isNumber(value: unknown): value is number {
  return typeof value === "number" && !isNaN(value);
}

function isDefined<T>(value: T | null | undefined): value is NonNullable<T> {
  return value !== null && value !== undefined;
}

interface Cat { meow: () => void; }
interface Dog { bark: () => void; }

function isCat(pet: Cat | Dog): pet is Cat {
  return (pet as Cat).meow !== undefined;
}

function makeSound(pet: Cat | Dog): void {
  if (isCat(pet)) {
    pet.meow();
  } else {
    pet.bark();
  }
}

// Using with Array.filter
const items: (string | number)[] = ["a", 1, "b", 2];
const onlyStrings: string[] = items.filter(isString);
const onlyNumbers: number[] = items.filter(isNumber);
```

### Intermediate Examples

```typescript
// Generic predicate
function isOfType<T>(value: unknown, prop: string): value is T {
  return typeof value === "object" && value !== null && prop in value;
}

// Predicate for discriminated unions
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function isCircle(shape: Shape): shape is { kind: "circle"; radius: number } {
  return shape.kind === "circle";
}

function isSquare(shape: Shape): shape is { kind: "square"; side: number } {
  return shape.kind === "square";
}

// Predicate factory
function createArrayFilter<T>() {
  return function filter(arr: unknown[]): arr is T[] {
    return arr.every(item => typeof item === "string"); // Simplified
  };
}

const filterStrings = createArrayFilter<string>();

// Compound predicates
interface HasEmail { email: string; }
interface HasPhone { phone: string; }

function hasEmail(value: unknown): value is HasEmail {
  return typeof value === "object" && value !== null && "email" in value;
}

function hasPhone(value: unknown): value is HasPhone {
  return typeof value === "object" && value !== null && "phone" in value;
}

function isContact(value: unknown): value is HasEmail & HasPhone {
  return hasEmail(value) && hasPhone(value);
}
```

### Advanced Examples

```typescript
// Recursive type predicate
type JSONValue = string | number | boolean | null | JSONValue[] | { [key: string]: JSONValue };

function isJSONValue(value: unknown): value is JSONValue {
  if (value === null) return true;
  if (typeof value === "string" || typeof value === "number" || typeof value === "boolean") return true;
  if (Array.isArray(value)) return value.every(isJSONValue);
  if (typeof value === "object") {
    return Object.values(value as Record<string, unknown>).every(isJSONValue);
  }
  return false;
}

// Predicate with brand
type Brand<T, B> = T & { __brand: B };
type UserId = Brand<string, "UserId">;

function isUserId(value: string): value is UserId {
  return /^user_\d+$/.test(value);
}

// Type guard with multiple type parameters
function isInstanceOf<T extends new (...args: any[]) => any>(
  value: unknown,
  cls: T
): value is InstanceType<T> {
  return value instanceof cls;
}

// Predicate narrowing in switch
type ActionResult<T> =
  | { status: "success"; data: T }
  | { status: "error"; error: Error }
  | { status: "loading" };

function isSuccess<T>(result: ActionResult<T>): result is { status: "success"; data: T } {
  return result.status === "success";
}

function isError<T>(result: ActionResult<T>): result is { status: "error"; error: Error } {
  return result.status === "error";
}

// Composing predicates with logical operators
function isStringOrArray(value: unknown): value is string | unknown[] {
  return typeof value === "string" || Array.isArray(value);
}
```

### Real-World Use Cases

- **API response validation**: Checking the structure of API responses.
- **Database query results**: Type-safe result set processing.
- **Configuration validation**: Ensuring config objects have the right shape.
- **Form validation**: Validating form field values and narrowing types.
- **Serialization/deserialization**: Validating parsed JSON structures.
- **Plugin capability detection**: Checking if a plugin implements a specific interface.

```typescript
// Real-world: API response validation
interface SuccessResponse { ok: true; data: unknown; }
interface ErrorResponse { ok: false; error: string; }

type APIResponse = SuccessResponse | ErrorResponse;

function isSuccessResponse(response: APIResponse): response is SuccessResponse {
  return response.ok === true;
}

async function fetchData<T>(url: string): Promise<T> {
  const response = await fetch(url);
  const json: APIResponse = await response.json();
  if (!isSuccessResponse(json)) {
    throw new Error(json.error);
  }
  return json.data as T;
}
```

### Common Mistakes

1. **Writing incorrect predicate implementations**: The compiler trusts you. A buggy predicate causes incorrect narrowing.
2. **Not checking all necessary properties**: Missing a check allows wrong values through.
3. **Overly broad predicates**: `value is object` catches too much.
4. **Forgetting that predicates only narrow in the true branch**: The false branch removes the type.
5. **Not using predicates with `Array.filter`**: Without the predicate type, `.filter()` returns the original union type.

### Best Practices

- Keep predicate functions simple and focused on one type check.
- Always test predicate functions thoroughly—the compiler doesn't verify them.
- Use predicates with `Array.filter` for type-safe array filtering.
- Prefer built-in guards over predicates when possible.
- Combine predicates with generic constraints for reusable type checks.

### Performance Considerations

Predicate functions have runtime cost proportional to their implementation complexity. For performance-critical code, minimize checks inside predicates. The compile-time narrowing has no runtime overhead.

### Interview Questions

1. **Q**: How is a type predicate different from a regular boolean return?
   **A**: A type predicate (`value is Type`) tells the compiler to narrow the type. A regular boolean return does not narrow.

2. **Q**: Can you use type predicates with `Array.filter`?
   **A**: Yes. `.filter(isString)` returns `string[]` instead of `(string | number)[]` when the predicate has the right signature.

### Coding Challenges

1. Write a type predicate for a recursive data structure (like a binary tree).
2. Implement a type-safe `deepClone` function that uses predicates to handle different types.

### Related Topics

- `asserts` keyword
- Type narrowing
- `unknown` type
- Discriminated unions

## asserts keyword

### What It Is

The `asserts` keyword creates assertion functions that throw if a condition is not met. After a successful (non-throwing) call, TypeScript narrows the type of the asserted value. Unlike type predicates, assertion functions don't return a boolean—they throw on failure.

```typescript
function assertString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error("Expected string");
  }
}
```

### Why It Is Important

Assertion functions provide a way to validate data and narrow types in one step. They are particularly useful for input validation, API boundary checks, and parsing untrusted data. They combine runtime validation with compile-time narrowing, making the developer experience seamless.

### How It Works Internally

When the compiler sees a call to an assertion function like `assertString(x)`, it checks whether the function has an `asserts` return type. If so, after the call (assuming no exception), the compiler narrows `x` to the asserted type. The compiler trusts the assertion function and does not verify the implementation.

```typescript
// Before: value: unknown
assertString(value);
// After: value: string (if no exception thrown)
```

### Syntax

```typescript
// Basic assertion
function assertString(value: unknown): asserts value is string {
  if (typeof value !== "string") throw new Error();
}

// Assertion without type narrowing (just asserts truthiness)
function assert(condition: boolean): asserts condition {
  if (!condition) throw new Error("Assertion failed");
}

// Assertion with custom error
function assertDefined<T>(value: T): asserts value is NonNullable<T> {
  if (value === null || value === undefined) {
    throw new Error("Value is null or undefined");
  }
}
```

### Beginner Examples

```typescript
function assertNumber(value: unknown): asserts value is number {
  if (typeof value !== "number" || isNaN(value)) {
    throw new TypeError("Expected a valid number");
  }
}

function double(value: unknown): number {
  assertNumber(value);
  return value * 2; // value is number here
}

function assertArray(value: unknown): asserts value is unknown[] {
  if (!Array.isArray(value)) {
    throw new Error("Expected an array");
  }
}

function firstElement(value: unknown): unknown {
  assertArray(value);
  return value[0];
}

function assertIsError(err: unknown): asserts err is Error {
  if (!(err instanceof Error)) {
    throw new Error("Expected an Error instance");
  }
}

function processError(err: unknown): string {
  assertIsError(err);
  return err.message; // err is Error
}
```

### Intermediate Examples

```typescript
// Assertion with generic
function assertInstanceOf<T extends new (...args: any[]) => any>(
  value: unknown,
  cls: T
): asserts value is InstanceType<T> {
  if (!(value instanceof cls)) {
    throw new TypeError(`Expected instance of ${cls.name}`);
  }
}

class User { constructor(public name: string) {} }

function greet(value: unknown): string {
  assertInstanceOf(value, User);
  return `Hello, ${value.name}`;
}

// Multiple assertions
function assertStringRecord(value: unknown): asserts value is Record<string, string> {
  if (typeof value !== "object" || value === null) {
    throw new Error("Expected an object");
  }
  for (const [k, v] of Object.entries(value)) {
    if (typeof k !== "string" || typeof v !== "string") {
      throw new Error("Expected string key-value pairs");
    }
  }
}

// Assertion with discriminated union
type Shape = 
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function assertCircle(shape: Shape): asserts shape is { kind: "circle"; radius: number } {
  if (shape.kind !== "circle") {
    throw new Error("Expected a circle");
  }
}

function getRadius(shape: Shape): number {
  assertCircle(shape);
  return shape.radius;
}
```

### Advanced Examples

```typescript
// Assertion function factories
function createTypeAssertion<T>() {
  return function assert(value: unknown, validator: (v: unknown) => boolean): asserts value is T {
    if (!validator(value)) {
      throw new TypeError("Type assertion failed");
    }
  };
}

const assertString = createTypeAssertion<string>();

// Assertion with complex validation
interface EmailConfig {
  host: string;
  port: number;
  user: string;
  pass: string;
}

function assertEmailConfig(value: unknown): asserts value is EmailConfig {
  if (typeof value !== "object" || value === null) throw new Error("Expected object");
  const obj = value as Record<string, unknown>;
  if (typeof obj.host !== "string") throw new Error("host must be a string");
  if (typeof obj.port !== "number") throw new Error("port must be a number");
  if (typeof obj.user !== "string") throw new Error("user must be a string");
  if (typeof obj.pass !== "string") throw new Error("pass must be a string");
}

// Combining asserts with type predicates
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function assertString(value: unknown): asserts value is string {
  if (!isString(value)) throw new Error("Expected string");
}

// Assertion with error discrimination
class ValidationError extends Error {
  constructor(message: string, public readonly field: string) {
    super(message);
  }
}

function assertField<T>(value: unknown, field: string): asserts value is T {
  if (typeof value === "undefined" || value === null) {
    throw new ValidationError(`${field} is required`, field);
  }
}

// Using asserts in class methods
class Parser {
  parse(data: unknown): Record<string, unknown> {
    this.assertRecord(data);
    return data;
  }

  private assertRecord(value: unknown): asserts value is Record<string, unknown> {
    if (typeof value !== "object" || value === null) {
      throw new Error("Expected a record");
    }
  }
}
```

### Real-World Use Cases

- **Input validation at API boundaries**: Asserting request body shapes.
- **Environment variable validation**: Ensuring required env vars are present and typed.
- **Configuration loading**: Validating config files against expected schemas.
- **Database result processing**: Ensuring query results have expected shapes.
- **Testing utilities**: Custom assertion helpers in test suites.
- **Deserialization safety**: Validating parsed JSON against expected types.

```typescript
// Real-world: Environment variable validation
function assertEnvVar(value: string | undefined, name: string): asserts value is string {
  if (typeof value !== "string" || value.length === 0) {
    throw new Error(`Environment variable ${name} is required`);
  }
}

function loadConfig(): Config {
  assertEnvVar(process.env.DB_HOST, "DB_HOST");
  assertEnvVar(process.env.DB_PORT, "DB_PORT");
  assertEnvVar(process.env.DB_USER, "DB_USER");
  assertEnvVar(process.env.DB_PASS, "DB_PASS");
  return {
    host: process.env.DB_HOST,
    port: parseInt(process.env.DB_PORT, 10),
    user: process.env.DB_USER,
    pass: process.env.DB_PASS,
  };
}
```

### Common Mistakes

1. **Forgetting to throw**: The assertion must throw on failure, or the narrowing is invalid.
2. **Using `asserts` without a type**: `asserts condition` without `is Type` only narrows the condition variable.
3. **Not handling the exception**: Callers must wrap assertion calls in try-catch.
4. **Overusing assertions**: For recoverable errors, prefer type predicates.
5. **Asserting without checking**: The implementation must actually check the type.

### Best Practices

- Use `asserts` for non-recoverable validation failures.
- Use type predicates for recoverable checks (where returning `false` is a valid outcome).
- Provide clear error messages in assertions.
- Combine assertions with configuration loading and input validation.
- Use `asserts` at module boundaries (API, file loading, config).
- Never use assertions for control flow—they should only fail on programmer error.

### Performance Considerations

Assertion functions have runtime cost proportional to the validation logic. The narrowing effect is compile-time only. If assertions are called frequently, validate with a type predicate first, then assert.

### Interview Questions

1. **Q**: What is the difference between `value is Type` (predicate) and `asserts value is Type` (assertion)?
   **A**: A predicate returns `boolean` and narrows in the `true` branch. An assertion throws on failure and narrows after the call if no exception is thrown.

2. **Q**: Can an assertion function have a return type other than `void`?
   **A**: No. Assertion functions must return `void` or `never`. They cannot return a meaningful value.

3. **Q**: How does the compiler know an assertion function succeeded?
   **A**: It assumes that if the function returns normally (no exception), the assertion passed, and it narrows accordingly.

### Coding Challenges

1. Implement an `assertShape` function that validates a discriminated union and narrows to the correct variant.
2. Write a configuration loader that uses assertions to validate each configuration field.

### Related Topics

- Type predicates
- Exception handling
- Narrowing
- `never` type
