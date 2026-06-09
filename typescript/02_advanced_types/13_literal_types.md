# Literal Types - String/number/boolean literals, template literal types, widening

## Introduction

Literal types are types that represent specific, exact values rather than general categories. Instead of `string`, which accepts any string, you can use a literal type like `"hello"` which accepts only the exact string `"hello"`. TypeScript has three primitive literal types: string literals, number literals, and boolean literals. Additionally, TypeScript 4.1 introduced template literal types, which enable powerful string pattern matching at the type level.

Literal types are the foundation for discriminated unions, string enums, and many advanced type patterns. Combined with union types, they enable precise modeling of finite value sets, making impossible states impossible to represent and catching bugs at compile time.

## String literal types

### What It Is

A string literal type is a type that accepts only one specific string value. Written as a string in quotes, like `"success"` or `"GET"`. When combined with union types, string literal types can enumerate all valid values for a parameter.

```typescript
type Method = "GET" | "POST" | "PUT" | "DELETE";
type Status = "idle" | "loading" | "success" | "error";
```

### Why It Is Important

String literal types eliminate entire categories of runtime errors by restricting values to a predefined set. They make APIs self-documenting, enable autocompletion in IDEs, and provide compile-time guarantees that invalid values cannot be passed. They are the mechanism behind discriminated unions, exhaustive switches, and many configuration patterns.

### How It Works Internally

TypeScript represents each string literal as a unique type. When storing in a wider type (like `string`), the literal is *widened* to the broader type. The compiler tracks the exact literal value through assignments and function calls, using it for narrowing and exhaustiveness checking. String literals are subtypes of `string`, so `"hello"` is assignable to `string`, but not vice versa.

```typescript
// "hello" extends string → true
// string extends "hello" → false
```

### Syntax

```typescript
// Single string literal
type Hello = "hello";
const greet: Hello = "hello"; // OK
// const bad: Hello = "hi"; // Error

// Union of string literals
type Direction = "north" | "south" | "east" | "west";
type HTTPMethod = "GET" | "POST" | "PUT" | "PATCH" | "DELETE";
type CSSUnit = "px" | "em" | "rem" | "%" | "vh" | "vw";

// String literal in function parameter
function setAlign(align: "left" | "center" | "right"): void {}

// String literal in property
type Config = {
  mode: "development" | "production" | "test";
  logLevel: "debug" | "info" | "warn" | "error";
};
```

### Beginner Examples

```typescript
type Color = "red" | "green" | "blue";

function getHexColor(color: Color): string {
  switch (color) {
    case "red": return "#FF0000";
    case "green": return "#00FF00";
    case "blue": return "#0000FF";
  }
}

getHexColor("red"); // OK
// getHexColor("yellow"); // Error

type CardSuit = "hearts" | "diamonds" | "clubs" | "spades";
type CardRank = "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" | "10" | "J" | "Q" | "K" | "A";
type Card = `${CardRank} of ${CardSuit}`;

function createDeck(): Card[] {
  const suits: CardSuit[] = ["hearts", "diamonds", "clubs", "spades"];
  const ranks: CardRank[] = ["2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"];
  return suits.flatMap(suit => ranks.map(rank => `${rank} of ${suit}` as Card));
}
```

### Intermediate Examples

```typescript
// String literal as discriminant
type Action =
  | { type: "ADD_TODO"; text: string }
  | { type: "DELETE_TODO"; id: number }
  | { type: "TOGGLE_TODO"; id: number };

function reducer(state: Todo[], action: Action): Todo[] {
  switch (action.type) {
    case "ADD_TODO": return [...state, { id: Date.now(), text: action.text, done: false }];
    case "DELETE_TODO": return state.filter(t => t.id !== action.id);
    case "TOGGLE_TODO": return state.map(t => t.id === action.id ? { ...t, done: !t.done } : t);
  }
}

// String literal union for options
type SortOrder = "asc" | "desc";
type SortField = "name" | "date" | "size";

function sortBy<T>(items: T[], field: keyof T, order: SortOrder): T[] {
  return [...items].sort((a, b) => {
    const cmp = a[field] < b[field] ? -1 : a[field] > b[field] ? 1 : 0;
    return order === "asc" ? cmp : -cmp;
  });
}

// Pattern matching with string literals
type EventType = "click" | "dblclick" | "mouseenter" | "mouseleave";

function handleEvent(event: EventType, handler: () => void): void {
  const validEvents: EventType[] = ["click", "dblclick", "mouseenter", "mouseleave"];
  if (validEvents.includes(event)) {
    handler();
  }
}
```

### Advanced Examples

```typescript
// Generic with string literal constraint
function createApi<T extends string>(endpoint: T) {
  return {
    get: () => fetch(`/api/${endpoint}`),
    post: <B>(body: B) => fetch(`/api/${endpoint}`, { method: "POST", body: JSON.stringify(body) }),
  };
}

const usersApi = createApi("users"); // T is "users"

// Branded string literals
type Brand<T, B> = T & { __brand: B };
type Email = Brand<string, "Email">;
type Phone = Brand<string, "Phone">;

function sendEmail(to: Email, message: string): void {}
const myEmail = "user@example.com" as Email;
sendEmail(myEmail, "Hello");

// String literal to enum mapping
type StatusCode = 200 | 201 | 204 | 400 | 401 | 403 | 404 | 500;
type StatusMessage = {
  [K in StatusCode]: string;
};
const statusMessages: StatusMessage = {
  200: "OK",
  201: "Created",
  204: "No Content",
  400: "Bad Request",
  401: "Unauthorized",
  403: "Forbidden",
  404: "Not Found",
  500: "Internal Server Error",
};

// Conditional type with string literal
type EventHandler<E extends string> = E extends "click"
  ? (e: MouseEvent) => void
  : E extends "keydown"
    ? (e: KeyboardEvent) => void
    : (e: Event) => void;

type ClickHandler = EventHandler<"click">; // (e: MouseEvent) => void
```

### Real-World Use Cases

- **API route definitions**: Type-safe route parameters and methods.
- **State machines**: Finite states as string literals.
- **Configuration objects**: Allowed modes, levels, and options.
- **CSS-in-JS**: Type-safe unit and property values.
- **Event systems**: Type-safe event names in event emitters.
- **Database query builders**: Type-safe operators and sort orders.
- **Form validation**: Predefined validation rule names.

```typescript
// Real-world: Route builder
type Route = "/users" | "/posts" | "/comments";
type Method = "GET" | "POST" | "PUT" | "DELETE";
type ApiCall = `${Method} ${Route}`;
// "GET /users" | "GET /posts" | ... | "DELETE /comments"
```

### Common Mistakes

1. **Using `string` when a literal union would be safer**.
2. **Assuming a `const` variable automatically narrows**—it does: `const x = "hello";` is typed as `"hello"`, not `string`.
3. **Forgetting that `let` variables widen**: `let x = "hello"` is typed as `string`.
4. **Creating too-large literal unions**: thousands of literals slow the compiler.
5. **Mixing case-sensitive literals**: `"GET"` and `"get"` are different types.

### Best Practices

- Use string literal unions for parameters with a fixed set of values.
- Combine with discriminated unions for state machines.
- Use `const` assertions (`as const`) to preserve literal types.
- Prefer template literal types for string patterns.
- Use enums only when you need numeric values; otherwise, use string literal unions.

### Performance Considerations

Each string literal is a unique type. A union of 100 string literals creates 100 distinct types, which the compiler must track and check. For very large sets (100+), consider using branded types or runtime validation instead. Template literal types with large unions can cause combinatorial explosion.

### Interview Questions

1. **Q**: What is the difference between `const x = "hello"` and `let x = "hello"` in terms of literal types?
   **A**: `const x` is inferred as `"hello"` (literal type). `let x` is inferred as `string` (widened type).

2. **Q**: How do string literal types enable discriminated unions?
   **A**: Each union member has a property with a unique string literal type (e.g., `{ kind: "circle" }`), which TypeScript uses for narrowing in switch/if statements.

3. **Q**: Can a string literal type be wider than a single string?
   **A**: No, a single string literal type is exactly one string. Use unions to represent multiple strings.

### Coding Challenges

1. Implement a type-safe event emitter with string literal event names.
2. Create a configuration system where all configuration keys are validated as string literals at compile time.
3. Build a URL router that extracts path parameters using string literal patterns.

### Related Topics

- Literal widening
- Template literal types
- Discriminated unions
- `as const`
- Enums

## Number and boolean literals

### What It Is

Number literal types represent exact numeric values, and boolean literal types represent the exact values `true` or `false`. Like string literals, they can be composed into unions to represent finite sets of valid values.

```typescript
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;
type HTTPStatus = 200 | 201 | 204 | 400 | 401 | 403 | 404 | 500;
type BooleanValue = true | false;
type Truthy = true;
type Falsy = false;
```

### Why It Is Important

Number literal types enable type-safe numeric constants and flags. They are essential for modeling status codes, enum-like numeric sets, and precise numeric configuration. Boolean literal types are crucial for type predicates, branded types, and discriminated unions where the discriminant is a boolean.

### How It Works Internally

Number literals are represented as subtypes of `number`. The compiler tracks the exact numeric value through constant folding and assignments. Boolean literals are represented as subtypes of `boolean`, where `boolean` is actually a union type `true | false` internally.

```typescript
// Internally: boolean = true | false
// So `true` is a literal subtype of `boolean`
```

### Syntax

```typescript
// Number literal
type Zero = 0;
type One = 1;
type Two = 2;

// Union of number literals
type SmallPositive = 1 | 2 | 3 | 4 | 5;
type Weekday = 0 | 1 | 2 | 3 | 4 | 5 | 6;
type Month = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12;

// Boolean literal
type IsActive = true;
type IsInactive = false;

// Union with boolean
type Result = true | false; // Same as boolean
type MaybeDone = true | false | undefined;

// Negative numbers
type Temperature = -10 | -5 | 0 | 5 | 10;
```

### Beginner Examples

```typescript
function rollDice(): number {
  return (Math.floor(Math.random() * 6) + 1) as 1 | 2 | 3 | 4 | 5 | 6;
}

function getHTTPStatusMessage(status: 200 | 201 | 400 | 404 | 500): string {
  switch (status) {
    case 200: return "OK";
    case 201: return "Created";
    case 400: return "Bad Request";
    case 404: return "Not Found";
    case 500: return "Internal Server Error";
  }
}

function isActive(value: true): boolean {
  return value;
}

// Boolean literal in type predicate
function isString(value: unknown): value is string {
  return typeof value === "string";
}
```

### Intermediate Examples

```typescript
// Number literal for exact arithmetic
type Unit = 1;
type Zero = 0;
type Negative = -1;

// Using number literal in array length
type ArrayOfLength<N extends number, T> = N extends 0
  ? []
  : N extends 1
    ? [T]
    : N extends 2
      ? [T, T]
      : T[];

type Pair = ArrayOfLength<2, string>; // [string, string]

// Boolean literal for function overloading
function process<T>(value: T, flag: true): string;
function process<T>(value: T, flag: false): number;
function process<T>(value: T, flag: boolean): string | number {
  return flag ? JSON.stringify(value) : (value as any).length;
}

// Number literal for bitmask
type Permission = 0 | 1 | 2 | 4 | 8;
const READ: Permission = 1;
const WRITE: Permission = 2;
const EXECUTE: Permission = 4;
const DELETE: Permission = 8;

function hasPermission(userPerms: number, required: Permission): boolean {
  return (userPerms & required) === required;
}
```

### Advanced Examples

```typescript
// Negative number literals for offsets
type Offset = -1 | 0 | 1;

function move(direction: Offset): void {
  // -1 = left/up, 0 = stay, 1 = right/down
}

// Number literal for tuple index access
type Tuple = [string, number, boolean];
type First = Tuple[0]; // string
type Second = Tuple[1]; // number
type Third = Tuple[2]; // boolean

// Number literal with template literal (string to number)
type NumericString = "0" | "1" | "2" | "3" | "4" | "5";

// Boolean literal as discriminant
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: Error };

function handleResult<T>(result: Result<T>): T {
  if (result.success) return result.data;
  throw result.error;
}

// Phantom boolean types for state machines
type DoorState = true; // open
type DoorClosed = false; // closed
type Door<State extends boolean> = { isOpen: State };

const openDoor: Door<true> = { isOpen: true };
const closedDoor: Door<false> = { isOpen: false };

// Only openDoor can pass
function walkThrough(door: Door<true>): void {}
walkThrough(openDoor); // OK
// walkThrough(closedDoor); // Error
```

### Real-World Use Cases

- **HTTP status codes**: Type-safe status code handling.
- **Configuration flags**: `true`/`false`/`undefined` for feature flags.
- **Bitmask permissions**: Using number literals for permission bits.
- **Array dimensions**: Fixed-length arrays and tuple types.
- **Game development**: Dice, card values, and coordinate offsets.
- **Financial systems**: Exact currency amounts and rounding modes.

```typescript
// Real-world: HTTP status handler
type StatusCode = 100 | 200 | 201 | 204 | 301 | 302 | 400 | 401 | 403 | 404 | 500 | 502 | 503;
function handleStatus(code: StatusCode): void {
  if (code >= 200 && code < 300) console.log("Success");
  else if (code >= 300 && code < 400) console.log("Redirect");
  else if (code >= 400 && code < 500) console.log("Client Error");
  else console.log("Server Error");
}
```

### Common Mistakes

1. **Using `number` when a literal union would prevent invalid values**.
2. **Not knowing that `boolean` is a union of `true | false`**.
3. **Assuming arithmetic changes literal types**: `const x: 5 = 5; const y = x + 1; // number, not 6`.
4. **Forgetting that number literals are compared by value, not identity**.

### Best Practices

- Use number literal unions for finite numeric sets.
- Use `boolean` for simple flags; use `true | false | undefined` for optional flags.
- Combine number literals with `const` for constants.
- Prefer discriminated unions with string literals over boolean discriminants for complex state.

### Performance Considerations

Number literal types are efficient. However, creating a union of every integer from 0 to 1000 is slow and rarely useful. Use `number` with runtime validation for large ranges. Boolean literal unions are always fast.

### Interview Questions

1. **Q**: What is the internal representation of `boolean` in TypeScript?
   **A**: `boolean` is a union type `true | false`.

2. **Q**: Can you have a literal type of `0.5`?
   **A**: Yes. Any numeric literal can be a type, including decimals, negatives, and scientific notation: `-1`, `0.5`, `1e3`, etc.

### Coding Challenges

1. Implement a type-safe permissions system using number literal bitmasks.
2. Create a door/gate state machine using boolean literal types.

### Related Topics

- String literal types
- Enums
- Bitmask patterns
- Boolean type predicates

## Template literal types

### What It Is

Template literal types, introduced in TypeScript 4.1, allow you to construct string types using template literal syntax. They can reference other types in `${...}` placeholders, enabling powerful string pattern matching and validation at the type level.

```typescript
type Greeting = `Hello, ${string}!`;
type EventName = `on${Capitalize<string>}`;
type CSSProperty = `--${string}`;
```

### Why It Is Important

Template literal types enable type-safe string manipulation for CSS, HTML, routing, API endpoints, and any domain involving string patterns. They eliminate runtime validation for string formats, provide autocomplete for complex string patterns, and enable new patterns like type-safe CSS-in-JS and URL builders.

### How It Works Internally

The compiler evaluates template literal types by expanding each placeholder. When a placeholder contains a union type, the compiler computes the *cartesian product* of all possible string combinations. For example, `` `${"a" | "b"}${"1" | "2"}` `` expands to `"a1" | "a2" | "b1" | "b2"`. This can cause combinatorial explosion if unions are large.

```typescript
// Cartesian product: 2 × 2 = 4 strings
type A = `${"a" | "b"}${"1" | "2"}`;
// "a1" | "a2" | "b1" | "b2"
```

### Syntax

```typescript
// Basic template literal
type Greeting = `Hello, ${string}!`;
type URL = `https://${string}.com`;

// With union placeholders
type Suffix = "er" | "est";
type Word = `fast${Suffix}`; // "faster" | "fastest"

// Nested template literals
type Color = "red" | "green" | "blue";
type CSSClass = `bg-${Color}-${100 | 200 | 300}`;

// With intrinsic string types
type Capitalized = `get${Capitalize<string>}`;

// Multiple placeholders
type Route = `/api/${string}/${string}`;
type Query = `?${string}=${string}`;
```

### Beginner Examples

```typescript
// Simple validation
type Email = `${string}@${string}.${string}`;
function sendEmail(to: Email): void {}
sendEmail("user@example.com"); // OK
// sendEmail("invalid"); // Error

// Event handlers
type EventType = "click" | "focus" | "blur" | "change";
type EventHandler = `on${Capitalize<EventType>}`;
// "onClick" | "onFocus" | "onBlur" | "onChange"

// CSS properties
type CSSUnit = "px" | "em" | "rem" | "%";
type CSSValue = `${number}${CSSUnit}`;
// "42px" | "2em" | "1.5rem" | "100%" | ...

function setWidth(width: CSSValue): void {}
setWidth("100px");
setWidth("50%");
// setWidth("abc"); // Error
```

### Intermediate Examples

```typescript
// Template literal with inference (pattern matching)
type ExtractName<T extends string> = T extends `${infer Name}.${string}`
  ? Name
  : never;

type FileName = ExtractName<"report.pdf">; // "report"
type NoExt = ExtractName<"readme">; // never

// Deeply nested template literals
type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
type Route = "/users" | "/posts" | "/comments";
type APICall = `${HTTPMethod} ${Route}`;
// "GET /users" | "POST /users" | ... | "DELETE /comments"

// Chained template literals
type Chain<T extends string> = `${T}.${T}`;
type Doubled = Chain<"hello">; // "hello.hello"

// Template literal with conditional types
type IsPrefixed<T extends string> = T extends `prefix-${string}` ? true : false;
type Check1 = IsPrefixed<"prefix-value">; // true
type Check2 = IsPrefixed<"value">; // false
```

### Advanced Examples

```typescript
// Template literal key remapping (mapped types)
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};
type Person = Getters<{ name: string; age: number }>;
// { getName: () => string; getAge: () => number }

// Parsing URL parameters
type ParseRoute<R extends string> =
  R extends `${string}:${infer Param}/${infer Rest}`
    ? { [K in Param | keyof ParseRoute<Rest>]: string }
    : R extends `${string}:${infer Param}`
      ? { [K in Param]: string }
      : {};

type UserRoute = ParseRoute<"/users/:id">; // { id: string }
type PostRoute = ParseRoute<"/posts/:postId/comments/:commentId">;
// { postId: string; commentId: string }

// Type-safe CSS builder
type CSSProperty = "color" | "background" | "font-size" | "margin" | "padding";
type CSSValue = `${number}px` | `${number}em` | string;
type CSSDeclaration = `${CSSProperty}: ${CSSValue}`;
type CSSRule = `${CSSDeclaration};`;

const style: CSSRule = "color: red;";

// Template literal with recursive types
type Join<T extends string[], Separator extends string> =
  T extends [infer First extends string, ...infer Rest extends string[]]
    ? Rest extends []
      ? First
      : `${First}${Separator}${Join<Rest, Separator>}`
    : "";

type Path = Join<["users", ":id", "posts"], "/">;
// "users/:id/posts"

// Capitalize, Uncapitalize, Uppercase, Lowercase intrinsic types
type CamelToSnake<S extends string> =
  S extends `${infer First}${infer Rest}`
    ? First extends Uppercase<First>
      ? `_${Lowercase<First>}${CamelToSnake<Rest>}`
      : `${First}${CamelToSnake<Rest>}`
    : S;

type SnakeCase = CamelToSnake<"myVariableName">; // "my_variable_name"
```

### Real-World Use Cases

- **Type-safe CSS**: Enforcing valid CSS property/value pairs.
- **URL routing**: Parsing route parameters at the type level.
- **API clients**: Type-safe endpoint construction with methods and paths.
- **Database queries**: Type-safe column names and table references.
- **Event emitters**: Enforcing `on` prefix conventions for event names.
- **GraphQL**: Type-safe field selection strings.
- **i18n**: Type-safe translation key patterns with parameter interpolation.

```typescript
// Real-world: Type-safe API client
type APIRoutes = {
  users: { list: "GET"; create: "POST" };
  posts: { list: "GET"; byId: "GET" };
};

type BuildPath<Module extends keyof APIRoutes, Action extends keyof APIRoutes[Module]> =
  `/${string & Module}/${string & Action}`;

type UserListPath = BuildPath<"users", "list">; // "/users/list"
```

### Common Mistakes

1. **Causing combinatorial explosion**: `${"a"|"b"|"c"}${"x"|"y"|"z"}` creates 9 types. Large unions create millions.
2. **Forgetting that template literals match at the type level, not runtime**.
3. **Using `string` placeholders too broadly**: `${string}` matches any string, negating the benefit.
4. **Not knowing about intrinsic string types**: `Capitalize`, `Uncapitalize`, `Uppercase`, `Lowercase`.
5. **Assuming template literals can parse string content**—they match structure, not runtime values.

### Best Practices

- Keep union sizes small in placeholders to avoid combinatorial explosion.
- Use `infer` for pattern matching with template literals.
- Combine template literal types with mapped types for key remapping.
- Use `Capitalize`/`Lowercase` etc. for naming convention transformations.
- Prefer template literal types for string validation over runtime regex checks.

### Performance Considerations

Combinatorial explosion is the primary performance concern. A template like `${A}${B}${C}` where each has 10 members creates 1000 types. TypeScript has a recursion limit (typically 50) and a union size limit (100,000). For complex string patterns, consider runtime validation instead.

### Interview Questions

1. **Q**: What happens when you put a union type in a template literal placeholder?
   **A**: The compiler computes the Cartesian product, creating a union of all possible string combinations.

2. **Q**: How do you extract parts of a string using template literal types?
   **A**: Using `infer` in a conditional type: `T extends `${infer A}-${infer B}` ? A : never`.

3. **Q**: What are the four intrinsic string types in TypeScript?
   **A**: `Capitalize<S>`, `Uncapitalize<S>`, `Uppercase<S>`, `Lowercase<S>`.

### Coding Challenges

1. Build a type that converts camelCase to kebab-case using template literal types.
2. Create a type-safe URL router that extracts path parameters at the type level.
3. Implement a type-safe CSS-in-JS system using template literal types.

### Related Topics

- Conditional types with `infer`
- Mapped type key remapping
- Intrinsic string types
- Template literal strings (ES6)

## Literal widening

### What It Is

Literal widening is the process by which TypeScript expands a literal type to its broader base type under certain conditions. The most common example is `let x = "hello"`, which TypeScript infers as `string`, not `"hello"`. This prevents overly narrow types from causing assignment errors in mutable variables.

```typescript
let x = "hello"; // x: string (widened)
const y = "hello"; // y: "hello" (not widened)
```

### Why It Is Important

Literal widening strikes a balance between precision and practicality. Without widening, every `let` assignment would fix a variable to a single literal value, making reassignment to a different value of the same base type impossible. Widening ensures that mutable variables have practical, flexible types.

### How It Works Internally

TypeScript's type inference applies widening based on the mutability context. `const` declarations are not widened (the value cannot change, so the literal is safe). `let` and `var` declarations are widened to their base type. Parameters, object properties, and array elements are also widened unless annotated or inferred from a `const` context.

```typescript
// Widening occurs for:
let a = "hello"; // string
var b = 42; // number
let c = true; // boolean

// Widening does NOT occur for:
const d = "hello"; // "hello"
let e: "hello" = "hello"; // "hello" (explicit annotation)
```

### Syntax

```typescript
// Automatic widening
let str = "hello"; // string
let num = 42; // number
let bool = true; // boolean

// Preventing widening with explicit type annotation
let str2: "hello" = "hello"; // "hello"
let num2: 42 = 42; // 42

// Preventing widening with as const
let str3 = "hello" as const; // "hello"
let arr = [1, 2, 3] as const; // readonly [1, 2, 3]

// Widening in objects
const obj = { name: "Alice" }; // { name: string } (property widened)
const obj2 = { name: "Alice" as const }; // { name: "Alice" } (literal preserved)

// Widening in arrays
const arr1 = [1, 2, 3]; // number[]
const arr2 = [1, 2, 3] as const; // readonly [1, 2, 3]
```

### Beginner Examples

```typescript
// Widening in variables
let status = "active"; // string
status = "inactive"; // OK
// status = 42; // Error: Type 'number' not assignable to 'string'

// const preserves literals
const port = 3000; // 3000
// port = 4000; // Error: Cannot assign to 'const'

// Function parameters are widened
function setStatus(s: "active" | "inactive") {}
const myStatus = "active"; // myStatus: "active"
setStatus(myStatus); // OK

// let parameter widened
let myStatus2 = "active"; // myStatus2: string
// setStatus(myStatus2); // Error: string not assignable to "active" | "inactive"
```

### Intermediate Examples

```typescript
// Widening in object literals
const config = {
  mode: "development", // string (widened)
  port: 8080, // number (widened)
};

// Use as const to preserve literals
const strictConfig = {
  mode: "development",
  port: 8080,
} as const;
// { readonly mode: "development"; readonly port: 8080 }

// Widening with tuples
let tuple = [1, "two"]; // (string | number)[]
const tuple2 = [1, "two"] as const; // readonly [1, "two"]

// Widening in function return types
function createMode() {
  return "development"; // Widened to string
}
// Use explicit return type or as const
function createModeStrict() {
  return "development" as const; // "development"
}

// Widening in generic inference
function identity<T>(value: T): T { return value; }
const result = identity("hello"); // T inferred as "hello" (no widening in generic inference)
```

### Advanced Examples

```typescript
// Controlling widening with custom types
type NoWiden<T> = T extends string
  ? T
  : T extends number
    ? T
    : T extends boolean
      ? T
      : never;

class Literal<T extends string> {
  constructor(public value: NoWiden<T>) {}
}

const lit = new Literal("hello");
// lit.value: "hello"

// Widening in conditional types
type IsWidened<T> = T extends string
  ? string extends T ? true : false
  : false;

type Test1 = IsWidened<"hello">; // false (literal, not string)
type Test2 = IsWidened<string>; // true

// Preserving literals through functions
function tuple<T extends readonly unknown[]>(...args: T): T {
  return args;
}

const result2 = tuple("hello", 42); // readonly ["hello", 42]

// Deep literal preservation
function deepFreeze<T>(obj: T): Readonly<T> {
  return Object.freeze(obj);
}

const frozen = deepFreeze({ name: "Alice" });
// frozen: Readonly<{ name: string }> (widened)
// Use as const at the call site

// as const with arrays of literals
const directions = ["north", "south", "east", "west"] as const;
type Direction = typeof directions[number]; // "north" | "south" | "east" | "west"
```

### Real-World Use Cases

- **Configuration objects**: Using `as const` to preserve literal types in configs.
- **Enum alternatives**: Using `const` objects with `as const` for type-safe constants.
- **Tuple inference**: Preserving element types in function results.
- **Route definitions**: Ensuring route strings are literals for type-safe routing.
- **CSS class names**: Preserving exact class name literals.

```typescript
// Real-world: Enum alternative with as const
const Colors = {
  Red: "#FF0000",
  Green: "#00FF00",
  Blue: "#0000FF",
} as const;
type ColorName = keyof typeof Colors; // "Red" | "Green" | "Blue"
type ColorHex = typeof Colors[ColorName]; // "#FF0000" | "#00FF00" | "#0000FF"

// Real-world: Route definitions
const ROUTES = {
  HOME: "/",
  USERS: "/users",
  USER_DETAIL: "/users/:id",
} as const;
type Route = typeof ROUTES[keyof typeof ROUTES];
// "/" | "/users" | "/users/:id"
```

### Common Mistakes

1. **Forgetting that `let` variables widen** and passing them to functions expecting literal unions.
2. **Not using `as const`** when you need to preserve literal types in objects and arrays.
3. **Overusing `as const`** where widening is acceptable, making types overly rigid.
4. **Assuming properties of `const` objects are literals**—they widen unless `as const` is used.
5. **Not knowing that function return values widen** even when the expression is a literal.

### Best Practices

- Use `const` instead of `let` whenever possible to preserve literal types.
- Use `as const` after object literals and arrays to preserve all nested literals.
- Add explicit type annotations when you need to prevent widening on `let` variables.
- Prefer `as const` over manual type annotations for complex literal objects.
- Remember that `as const` makes all properties `readonly`.

### Performance Considerations

Widening reduces type complexity, which improves compiler performance. A widened type (`string`) is simpler than a literal type (`"hello"`). However, the widening decision itself has negligible cost. Using `as const` preserves more type information, which can slightly increase compile-time type-checking but improves safety.

### Interview Questions

1. **Q**: When does TypeScript widen literal types?
   **A**: When a value is declared with `let` or `var`, when it's a property of an object literal (unless `as const`), and when it's an element of a mutable array.

2. **Q**: How do you prevent literal widening?
   **A**: Use `const`, use an explicit type annotation with the literal, or use `as const`.

3. **Q**: Does `as const` affect runtime behavior?
   **A**: No, `as const` is a compile-time-only construct. It tells the TypeScript type checker to infer the narrowest possible type. At runtime, it's erased.

### Coding Challenges

1. Write a function that takes a readonly tuple of strings and returns a union of the tuple's literal types.
2. Create a configuration system where all keys are preserved as literal types without using `as const`.
3. Implement a `deepFreeze` function that preserves literal types through `Object.freeze`.

### Related Topics

- `const` assertions
- Type inference
- `Readonly` utility type
- `as const` vs `const`
- Widening vs non-widening
