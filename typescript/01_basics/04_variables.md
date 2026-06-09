# Variables - let/const, type annotations, type inference, const assertions

## Introduction

Variable declaration is where TypeScript's type system first interacts with your code. How you declare variables — whether with `let` or `const`, with or without type annotations — determines how TypeScript infers and enforces types. This guide covers TypeScript's variable semantics, including the critical distinction between `let` and `const` in type inference and the powerful `as const` assertion for creating deeply immutable, literal types.

## let and const

### What It Is

`let` declares a mutable variable whose value can change over time. `const` declares an immutable binding — the variable cannot be reassigned after initialization. TypeScript treats these differently in its type inference, with `const` preserving literal types and `let` widening to the general type.

### Why It Is Important

Choosing between `let` and `const` affects not just code clarity but also type precision. TypeScript infers narrower, more specific types for `const` declarations, enabling better autocompletion and compile-time checks. `const` also communicates intent clearly — if a value shouldn't change, `const` makes it explicit and lets TypeScript enforce it.

### How It Works Internally

When TypeScript sees a `const` declaration with a primitive value, it infers the literal type (e.g., `"hello"` not `string`). For `let` declarations, it widens to the base type. This is because `let` implies potential reassignment, so the narrowest type would be too restrictive. For objects and arrays, `const` prevents reassignment but does NOT make the contents immutable — that requires `as const` or `Readonly<T>`.

```typescript
// const — literal type
const greeting = "Hello"; // type: "Hello"
const answer = 42; // type: 42
const isReady = true; // type: true

// let — widened type
let greeting2 = "Hello"; // type: string
let answer2 = 42; // type: number
let isReady2 = true; // type: boolean
```

### Syntax

```typescript
// Basic declarations
let mutableValue: string = "can change";
const immutableBinding: string = "cannot reassign";

// No annotation — inference handles it
const literalType = "hello"; // "hello"
let widenedType = "hello"; // string

// const with objects — binding is immutable, content is not
const person = { name: "Alice", age: 30 };
person.age = 31; // OK — object content can change
// person = { name: "Bob", age: 25 }; // Error: Assignment to constant variable
```

### Beginner Examples

```typescript
// Prefer const by default
const maxRetries = 3;
const apiEndpoint = "/api/users";

// Use let only when reassignment is needed
let currentAttempt = 0;
currentAttempt += 1;
currentAttempt += 1;

// TypeScript prevents reassignment with const
const fixedValue = 10;
// fixedValue = 20; // Error: Cannot assign to 'fixedValue' because it is a constant

// let allows reassignment
let mutable = 10;
mutable = 20; // OK

// let with different type
let value: string | number = "hello";
value = 42; // OK — union type allows both
```

### Intermediate Examples

```typescript
// const with block scoping
function processItems(items: string[]) {
  const result: string[] = [];

  for (let i = 0; i < items.length; i++) {
    const item = items[i]; // each iteration gets a new binding
    result.push(item.toUpperCase());
  }

  return result;
}

// Destructuring with const/let
const [first, ...rest] = [1, 2, 3, 4];
let [a, b] = [1, 2];
[a, b] = [b, a]; // Swap — only possible with let

// const in closures captures the value at creation
const functions: (() => void)[] = [];
for (let i = 0; i < 3; i++) {
  functions.push(() => console.log(i)); // let — captures current i
}
functions[0](); // 0
functions[1](); // 1
```

### Advanced Examples

```typescript
// Temporal dead zone with let
function temporalDeadZone() {
  // console.log(x); // ReferenceError: Cannot access 'x' before initialization
  let x = 10;
  console.log(x); // 10
}

// Const with computed property keys
const propKey = "dynamicKey";
const obj = {
  [propKey]: "value",
};

// Const assertions with let/const interplay
const config = {
  api: "https://api.example.com",
  timeout: 5000,
} as const;

// Cannot reassign config, and values are literal types
let timeout = config.timeout; // type: 5000 (literal)
timeout = 3000; // OK — let allows reassignment

// Pattern: const for structure, let for mutable state
interface GameState {
  score: number;
  lives: number;
}

const initialState: GameState = { score: 0, lives: 3 };
let currentState: GameState = { ...initialState };

function updateScore(points: number) {
  currentState = { ...currentState, score: currentState.score + points };
}
```

### Real-World Use Cases

**React State**: `const` for component state (even with `useState`, the state variable itself is `const`).
**Configuration Constants**: `const` for application config, environment variables, and magic numbers.
**Reducer State**: `let` for accumulator values in reducers where state is built up.

### Common Mistakes

**Using `let` When `const` Would Work**: This is the most common variable declaration mistake. `const` should be the default.

**Assuming `const` Makes Objects Immutable**: `const` only prevents reassignment of the binding, not mutation of the value.

**Forgetting `const` in For-of Loops**: Use `const` in `for...of` loops since each iteration creates a new binding.

```typescript
// Anti-pattern
let items = [1, 2, 3]; // Should be const
for (let i = 0; i < items.length; i++) {
  // OK — i needs to change
}
```

### Best Practices

1. Declare everything `const` by default; only use `let` when you know reassignment is needed
2. Never use `var` — it has function scoping and can cause bugs
3. Use `const` for destructured values unless reassignment is needed
4. Leverage `const` with literal inference for type safety
5. Prefer `const` with spread operators for "immutable updates"

### Performance Considerations

`const` and `let` have identical performance characteristics in modern JavaScript engines (V8, SpiderMonkey). The choice is purely about semantics and type inference, not runtime speed. TypeScript's `const` literal inference has zero runtime cost.

### Interview Questions

1. What is the difference between `const` and `let` in terms of type inference?
2. Does `const` make objects immutable?
3. What is the temporal dead zone?
4. How does `const` behavior differ in `for` and `for...of` loops?
5. Why prefer `const` over `let`?

### Coding Challenges

1. Rewrite a function to use `const` wherever possible, leaving `let` only for actual reassignment
2. Fix a function that incorrectly uses `let` for values that should be `const`
3. Create an immutable update pattern using `const` and spread operators

### Related Topics

- Variable Declaration in ES6+
- Block Scoping with let and const
- Temporal Dead Zone
- Immutability Patterns in TypeScript

## Type annotations

### What It Is

Type annotations explicitly declare the type of a variable, function parameter, or function return value. They appear after a colon and provide the TypeScript compiler with direct information about what type a value should be.

### Why It Is Important

While TypeScript can infer types, annotations serve as documentation, ensure correctness at declaration sites, and are required in certain positions (function parameters). They also allow TypeScript to check that the assigned value matches the declared type, catching bugs at the point of assignment rather than later usage.

### How It Works Internally

When TypeScript encounters a type annotation, it first checks the annotation for validity (the type must exist and be well-formed). Then it checks that the assigned value is assignable to the annotated type. If the annotation and the inferred type from the value conflict, TypeScript reports an error. Annotations also participate in contextual typing, where the annotated type narrows the expected shape of the assigned expression.

```typescript
// TypeScript checks: value must be assignable to annotation
let name: string = "Alice"; // OK
// let age: string = 30; // Error: Type 'number' is not assignable to type 'string'
```

### Syntax

```typescript
// Variable annotations
let username: string = "Alice";
let age: number = 30;
let isActive: boolean = true;
let data: string[] = ["a", "b", "c"];
let config: { theme: string; volume: number } = { theme: "dark", volume: 75 };

// Function parameter annotations
function greet(name: string, greeting: string): string {
  return `${greeting}, ${name}!`;
}

// Function return type annotation
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function annotations
const multiply = (a: number, b: number): number => a * b;

// Complex type annotations
let callback: (error: Error | null, result?: string) => void;
let handler: (...args: unknown[]) => void;

// Generic annotations
let pairs: Map<string, number> = new Map();
let cache: Record<string, Promise<string>> = {};
```

### Beginner Examples

```typescript
// Annotations catch type mismatches
let count: number = 10;
// count = "ten"; // Error: Type 'string' is not assignable to type 'number'

// Function parameter annotations
function calculateTotal(price: number, quantity: number): number {
  return price * quantity;
}
console.log(calculateTotal(10, 3)); // 30
// console.log(calculateTotal("10", 3)); // Error

// Array annotations
const names: string[] = ["Alice", "Bob", "Charlie"];
const first: string | undefined = names[0];

// Object annotations
const point: { x: number; y: number } = { x: 10, y: 20 };
```

### Intermediate Examples

```typescript
// Union type annotations
let id: string | number;
id = "abc123"; // OK
id = 456; // OK
// id = true; // Error

// Optional property annotations
interface UserConfig {
  name: string;
  age?: number;
  email: string;
}

const config: UserConfig = {
  name: "Alice",
  email: "alice@example.com",
  // age is optional — no error
};

// Function type annotations
type ClickHandler = (event: MouseEvent) => void;
const handleClick: ClickHandler = (event) => {
  console.log(event.clientX, event.clientY);
};

// Generic type annotations
function firstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}
const first: string | undefined = firstElement(["a", "b", "c"]);
```

### Advanced Examples

```typescript
// Conditional type annotations
type NumberOrString<T> = T extends number ? number : string;
let value: NumberOrString<typeof 42>; // number
let value2: NumberOrString<"hello">; // string

// Indexed access type annotations
interface APIResponse {
  status: number;
  data: {
    user: {
      id: number;
      name: string;
    };
  };
}
type UserType = APIResponse["data"]["user"];
const user: UserType = { id: 1, name: "Alice" };

// Mapped type annotations with as const
type ReadonlyConfig<T> = {
  readonly [P in keyof T]: T[P];
};

interface AppConfig {
  port: number;
  host: string;
  debug: boolean;
}

type ImmutableConfig = ReadonlyConfig<AppConfig>;
const config2: ImmutableConfig = { port: 3000, host: "localhost", debug: true };
// config2.port = 4000; // Error: readonly

// Template literal type annotations
type CSSValue = string | number;
type CSSProperty = `--${string}`;
const customProp: CSSProperty = "--primary-color";

// Satisfies with annotations
type Palette = Record<string, string>;
const colors = {
  primary: "#3498db",
  secondary: "#2ecc71",
} satisfies Palette;
// colors.primary — type is string, not just keyof typeof colors
```

### Real-World Use Cases

**API Response Typing**: Annotating API response variables ensures the data matches expected shapes.

**Configuration Objects**: Annotating config objects catches missing or misspelled properties.

**Callback Signatures**: Annotating callback types ensures correct parameter usage in event handlers and async operations.

### Common Mistakes

**Redundant Annotations**: Adding annotations that TypeScript can trivially infer adds noise without benefit.

```typescript
// Redundant
const name: string = "Alice"; // Inferred as "Alice" (literal), annotation widens to string
// Better:
const name = "Alice"; // "Alice" literal type

// Useful annotation
const apiUrl: string = "https://api.example.com"; // If you want string, not literal
```

**Too Narrow Annotations**: Annotating too restrictively can prevent legitimate values.

```
interface Settings {
  volume: number;
}
const settings: Settings = { volume: 75 }; // OK but if you wanted literal type...
const settings2 = { volume: 75 }; // { volume: number }
const settings3 = { volume: 75 } as const; // { readonly volume: 75 }
```

### Best Practices

1. Annotate function parameters — they can't be inferred from usage
2. Annotate function return types for public API functions
3. Let inference handle local variables
4. Use annotations to document intent (e.g., `const id: string | number = ...`)
5. Prefer `interface` or `type` aliases for complex annotations instead of inline types
6. Use `satisfies` for validation without type widening

### Performance Considerations

Type annotations are erased at compile time — zero runtime cost. More annotations mean more AST nodes for the compiler to process, but the impact is negligible in practice.

### Interview Questions

1. When would you use a type annotation instead of relying on inference?
2. What happens when a type annotation conflicts with an inferred type?
3. How do annotations affect contextual typing?
4. What is the difference between `let x: string = "hello"` and `const x = "hello"`?
5. Why can't TypeScript always infer function parameter types?

### Coding Challenges

1. Given a complex nested object, write the type annotation for it
2. Create a function with properly annotated parameters and return type
3. Fix a code snippet with incorrect type annotations

### Related Topics

- Type Inference in TypeScript
- Type Alias vs Interface
- Union and Intersection Types
- Contextual Typing

## Type inference

### What It Is

Type inference is TypeScript's ability to automatically deduce the type of a variable, expression, or function return value without explicit annotations. TypeScript uses sophisticated inference algorithms including contextual typing, flow-based narrowing, and generic type parameter inference.

### Why It Is Important

Type inference balances type safety with conciseness. Without it, every variable, expression, and function call would require an explicit type annotation, making TypeScript as verbose as Java. Inference keeps code readable while maintaining type safety.

### How It Works Internally

TypeScript's inference engine operates in several modes:

1. **Declaration inference**: From the initializer value of a variable
2. **Return type inference**: From the return statements in a function body
3. **Contextual inference**: From the expected type at the position of an expression
4. **Generic inference**: From the arguments passed to a generic function
5. **Flow-based inference**: Narrowing types based on control flow constructs

```typescript
// Declaration inference
let x = 5; // x: number

// Return type inference
function add(a: number, b: number) {
  return a + b; // return type: number
}

// Contextual inference
const names: string[] = [];
names.forEach((name) => {
  // name is inferred as string from context
});
```

### Syntax

```typescript
// Variables — type inferred from initializer
const name = "Alice"; // "Alice"
let age = 30; // number
const items = [1, 2, 3]; // number[]

// Destructuring inference
const [first, second] = [1, 2]; // first: number, second: number
const { id, title } = { id: 1, title: "TypeScript" }; // id: number, title: string

// Function return type inference
function createUser(name: string, age: number) {
  return { name, age, createdAt: new Date() };
  // Return type: { name: string; age: number; createdAt: Date }
}

// Generic inference
function identity<T>(arg: T): T {
  return arg;
}
const result = identity("hello"); // T inferred as string, result: string
```

### Beginner Examples

```typescript
// Basic inference
let message = "Hello, TypeScript!"; // message: string
let count = 42; // count: number
let isReady = false; // isReady: boolean

// Array inference
const fruits = ["apple", "banana", "orange"]; // fruits: string[]
const mixed = [1, "two", 3]; // mixed: (string | number)[]

// Object inference
const user = {
  name: "Alice",
  age: 30,
}; // user: { name: string; age: number }

// Function return inference
function sum(numbers: number[]) {
  return numbers.reduce((acc, n) => acc + n, 0);
  // Return type: number
}
```

### Intermediate Examples

```typescript
// Best common type inference
const array = [0, 1, null]; // (number | null)[]

// Contextual typing with callbacks
const numbers = [1, 2, 3, 4, 5];

// filter knows the callback type from Array<T>.filter
const evens = numbers.filter((n) => n % 2 === 0); // n: number

// Inference with generic constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = { name: "Alice", age: 30 };
const personName = getProperty(person, "name"); // string
const personAge = getProperty(person, "age"); // number

// Inference from multiple return paths
function format(input: string | number) {
  if (typeof input === "string") {
    return input.trim(); // return string
  }
  return input.toFixed(2); // return string (toFixed returns string)
}
// Return type: string
```

### Advanced Examples

```typescript
// Recursive inference
function deepClone<T>(obj: T): T {
  if (obj === null || typeof obj !== "object") return obj;
  if (Array.isArray(obj)) {
    return obj.map(deepClone) as unknown as T;
  }
  const cloned: Record<string, unknown> = {};
  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      cloned[key] = deepClone((obj as Record<string, unknown>)[key]);
    }
  }
  return cloned as T;
}

// Mapped type inference
type Optional<T> = {
  [P in keyof T]?: T[P];
};

function makeOptional<T>(obj: T): Optional<T> {
  const result: Record<string, unknown> = {};
  for (const key in obj) {
    result[key] = obj[key];
  }
  return result as Optional<T>;
}

const config = makeOptional({ port: 3000, host: "localhost" });
// Return type: { port?: number; host?: string }

// Conditional type inference
type ExtractReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
function example(): string {
  return "hello";
}
type ExampleReturn = ExtractReturnType<typeof example>; // string

// Template literal inference
function createEndpoint<T extends string>(path: T): `/api/${T}` {
  return `/api/${path}`;
}
const endpoint = createEndpoint("users"); // "/api/users"
```

### Real-World Use Cases

**React Hooks**: TypeScript infers the types from `useState`, `useReducer`, and custom hooks, reducing boilerplate.

**ORMs (Prisma, Drizzle)**: Database queries have their return types inferred from the query builder chain.

**API Clients (tRPC, TanStack Query)**: Endpoints and responses are fully inferred from the server definition.

### Common Mistakes

**Assuming Inference is Always Correct**: Complex operations (like `reduce`) may not infer the intended type — provide explicit generics when needed.

```typescript
// Wrong inference
const numbers = [1, 2, 3];
const sum = numbers.reduce((acc, n) => acc + n, 0); // OK: number
const wrongSum = numbers.reduce((acc, n) => acc + n); // Error: acc is number | undefined
// Fix:
const fixedSum = numbers.reduce((acc, n) => acc + n, 0);
```

**Over-relying on Inference for Public API Types**: Even if TypeScript infers the return type of a function, annotating it explicitly serves as documentation.

**Contextual Inference Surprises**: Sometimes inference from context gives unexpected types.

### Best Practices

1. Rely on inference for local variables
2. Add return type annotations to exported functions
3. Use `noImplicitAny: true` to ensure TypeScript errors when it can't infer
4. Provide explicit generic parameters when inference doesn't give the desired type
5. Prefer `const` over `let` for narrower type inference

### Performance Considerations

Type inference adds to compilation time, especially for complex expressions and generic type inference. In practice, the overhead is minimal. TypeScript 5.x introduced optimizations for type inference performance.

### Interview Questions

1. How does TypeScript determine the "best common type" for arrays?
2. What is contextual typing?
3. How does TypeScript infer generic type parameters?
4. When does inference fail and require explicit annotations?
5. How does control flow affect type inference?

### Coding Challenges

1. Create a function where TypeScript infers a complex return type
2. Write a generic function where the type parameter is inferred from usage
3. Fix a code snippet where inference gives the wrong type

### Related Topics

- TypeScript Type Inference
- Contextual Typing
- Best Common Type Algorithm
- Generic Type Inference

## const assertions (as const)

### What It Is

`as const` is a type assertion that tells TypeScript to infer the narrowest possible types for a value. For primitive literals, it infers the literal type. For objects, it makes all properties `readonly` and infers literal types for values. For arrays, it infers a tuple type with literal elements.

### Why It Is Important

`as const` enables precise, immutable type definitions without verbose annotations. It's essential for discriminated unions, configuration objects, and creating constant values with literal types that don't get widened to their general types.

### How It Works Internally

When `as const` is applied to a value, TypeScript applies three transformations:
1. **Literal inference**: All primitive values become their literal types (`"hello"` not `string`)
2. **Readonly members**: All object properties become `readonly`
3. **Tuple inference**: Arrays become tuple types with literal element types

```typescript
// Without as const
const config = { theme: "dark", retries: 3 };
// Type: { theme: string; retries: number }
// config.theme = "light"; // OK — not readonly

// With as const
const configFrozen = { theme: "dark", retries: 3 } as const;
// Type: { readonly theme: "dark"; readonly retries: 3 }
// configFrozen.theme = "light"; // Error: readonly property
```

### Syntax

```typescript
// Primitive literal with as const
const greeting = "Hello" as const; // type: "Hello" (narrowest possible)
// const greeting: "Hello" = "Hello" is equivalent but more verbose

// Object with as const
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
} as const;
// Type: { readonly apiUrl: "https://api.example.com"; readonly timeout: 5000 }

// Array with as const
const fruits = ["apple", "banana", "orange"] as const;
// Type: readonly ["apple", "banana", "orange"] (tuple)
// Not: string[]

// Nested as const
const nested = {
  outer: {
    inner: "deep",
  },
} as const;
// Type: { readonly outer: { readonly inner: "deep" } }
```

### Beginner Examples

```typescript
// Without as const — types are widened
const COLORS = {
  primary: "#3498db",
  secondary: "#2ecc71",
};
// COLORS.primary: string — loses the literal value

// With as const — types are literal
const COLORS_FROZEN = {
  primary: "#3498db",
  secondary: "#2ecc71",
} as const;
// COLORS_FROZEN.primary: "#3498db" — preserves literal

// Array as const creates a tuple
const positions = [10, 20, 30] as const; // readonly [10, 20, 30]
// const x = positions[0]; // type: 10 (not number)

// String literals with as const
const UP = "UP" as const; // "UP"
const DOWN = "DOWN" as const; // "DOWN"
// type Direction = typeof UP | typeof DOWN; // "UP" | "DOWN"
```

### Intermediate Examples

```typescript
// Discriminated union with as const
type Action =
  | { type: "ADD_TODO"; payload: string }
  | { type: "TOGGLE_TODO"; payload: number }
  | { type: "REMOVE_TODO"; payload: number };

// Using as const to create actions
const addTodoAction = {
  type: "ADD_TODO" as const,
  payload: "Learn TypeScript",
};

const toggleTodoAction = {
  type: "TOGGLE_TODO" as const,
  payload: 1,
};

// as const for configuration objects
type Env = "development" | "staging" | "production";

const envConfigs = {
  development: {
    apiUrl: "http://localhost:3000",
    debug: true,
  },
  staging: {
    apiUrl: "https://staging.example.com",
    debug: false,
  },
  production: {
    apiUrl: "https://api.example.com",
    debug: false,
  },
} as const;

type EnvConfig = (typeof envConfigs)[Env];
// { readonly apiUrl: string; readonly debug: boolean }

// as const with function returns
function createConfig() {
  return {
    port: 3000,
    host: "localhost",
  } as const;
}
const serverConfig = createConfig();
// Type: { readonly port: 3000; readonly host: "localhost" }
```

### Advanced Examples

```typescript
// as const for literal union derivation
const HTTP_METHODS = ["GET", "POST", "PUT", "DELETE", "PATCH"] as const;
type HttpMethod = (typeof HTTP_METHODS)[number];
// "GET" | "POST" | "PUT" | "DELETE" | "PATCH"

const STATUS_CODES = {
  SUCCESS: 200,
  CREATED: 201,
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  NOT_FOUND: 404,
  SERVER_ERROR: 500,
} as const;

type StatusCode = (typeof STATUS_CODES)[keyof typeof STATUS_CODES];
// 200 | 201 | 400 | 401 | 404 | 500

// as const with template literal types
const ROUTES = {
  users: "/users",
  posts: "/posts",
  comments: "/comments",
} as const;

type Route = (typeof ROUTES)[keyof typeof ROUTES];
// "/users" | "/posts" | "/comments"

type RouteParams = {
  [K in keyof typeof ROUTES]: (typeof ROUTES)[K];
};
// { readonly users: "/users"; readonly posts: "/posts"; readonly comments: "/comments" }

// as const with branded types
type Brand<K, T> = K & { __brand: T };
const createBrand = <T extends string>(value: T) => value as Brand<T, typeof value>;

const userId = createBrand("user_123");
// type: "user_123" & { __brand: "user_123" }

// as const for exhaustive mapping
const ColorMap = {
  primary: "#3498db",
  danger: "#e74c3c",
  success: "#2ecc71",
} as const;

type ColorName = keyof typeof ColorMap; // "primary" | "danger" | "success"
type ColorHex = (typeof ColorMap)[ColorName];
// "#3498db" | "#e74c3c" | "#2ecc71"

function getColor(name: ColorName): ColorHex {
  return ColorMap[name];
}
```

### Real-World Use Cases

**Configuration Constants**: Use `as const` for application config to get literal types for autocomplete and type safety.

```typescript
const APP_CONFIG = {
  apiBaseUrl: "https://api.example.com/v2",
  defaultPageSize: 20,
  maxRetries: 3,
  features: {
    darkMode: true,
    betaFeatures: false,
  },
} as const;

// APP_CONFIG.features.darkMode is typed as `true`, not `boolean`
```

**Enum Alternatives**: `as const` objects provide type-safe enum-like constructs without runtime costs.

**State Machines**: `as const` enables precise state definitions for finite state machines.

### Common Mistakes

**Forgetting `as const` on Array Literals**: Without `as const`, `["a", "b"]` is `string[]`, not `readonly ["a", "b"]`.

**Applying `as const` Too Broadly**: `as const` on large objects makes all properties readonly, which may not be desired.

**Using `as const` on Variables That Should Be Mutable**: `as const` creates deeply immutable types — if you need to change values later, don't use it.

### Best Practices

1. Use `as const` for configuration objects and constant definitions
2. Use `as const` to derive union types from arrays and objects
3. Use `as const` when you need literal types (e.g., discriminated union string literals)
4. Use `as const` for tuple inference from array literals
5. Prefer `as const` over `readonly` annotations for deep immutability

### Performance Considerations

`as const` is a compile-time construct with zero runtime cost. The inferred types can be more complex (deeply nested `readonly` types) but the compiler handles them efficiently. Deeply nested `as const` on very large objects can increase compilation time slightly.

### Interview Questions

1. What does `as const` do when applied to objects versus arrays?
2. How does `as const` differ from `Object.freeze`?
3. When would you use `as const` instead of an enum?
4. How does `as const` interact with `typeof`?
5. Can you reverse the effects of `as const`?

### Coding Challenges

1. Derive a union type from an array of strings using `as const`
2. Create a type-safe configuration object using `as const`
3. Implement a discriminated union of actions using `as const`
4. Create an enum-like construct using `as const` that's type-safe and tree-shakeable

### Related Topics

- Type Assertions
- Literal Types
- Readonly Types
- `const` vs `as const`
- Discriminated Unions via as const
