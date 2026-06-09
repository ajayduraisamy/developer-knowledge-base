# Tuples - Tuple types, labeled tuples, optional elements, rest elements, readonly tuples

## Introduction

Tuples are fixed-length arrays where each element has a specific type. Unlike regular arrays where all elements share the same type, tuples allow precise typing of each position. TypeScript's tuple support has evolved significantly, adding labeled tuples, optional elements, rest elements, and readonly variants.

## Tuple type syntax

### What It Is

A tuple type is defined by specifying the type of each element in order within square brackets: [string, number]. This creates a fixed-length array where the first element must be a string and the second must be a number.

### Why It Is Important

Tuples bridge the gap between arrays and structured data. They're ideal for small, fixed-size collections where each position has meaning — like coordinate pairs, key-value entries, or function return values with multiple parts.

### How It Works Internally

TypeScript represents tuples as Array<T> with numeric index signatures for each position. The type [string, number] is internally similar to { 0: string; 1: number; length: 2 }. This structural representation allows TypeScript to enforce length and per-element type constraints at compile time.

### Syntax

`	ypescript
// Basic tuple
let pair: [string, number] = ["age", 30];

// Access by index
const key: string = pair[0];
const value: number = pair[1];

// Destructuring
const [key2, value2] = pair;

// Function return tuple
function getMinMax(values: number[]): [number, number] {
  return [Math.min(...values), Math.max(...values)];
}
`

### Beginner Examples

`	ypescript
// Coordinate tuple
let point: [number, number] = [10, 20];
const x = point[0];
const y = point[1];

// API response
type ApiResponse = [number, string];
const response: ApiResponse = [200, "OK"];

// CSV row
type CsvRow = [string, number, boolean];
const row: CsvRow = ["Alice", 30, true];

// Destructuring tuples
const [status, message] = response;
console.log(Status : );
`

### Intermediate Examples

`	ypescript
// Tuple with different types
type HttpResult = [number, Record<string, unknown> | string];

const success: HttpResult = [200, { id: 1 }];
const error: HttpResult = [404, "Not Found"];

// Array of tuples
const entries: [string, number][] = [
  ["apple", 5],
  ["banana", 3],
  ["orange", 7],
];

// Tuple transformation
function toRecord<T extends string, U>(
  entries: readonly [T, U][]
): Record<T, U> {
  const result = {} as Record<T, U>;
  for (const [key, value] of entries) {
    result[key] = value;
  }
  return result;
}

const record = toRecord(entries);
// record.apple = 5

// Generic tuples
function swap<T, U>(pair: [T, U]): [U, T] {
  return [pair[1], pair[0]];
}

const swapped = swap(["hello", 42]); // [number, string]
`

### Advanced Examples

`	ypescript
// Variadic tuples (TS 4.0+)
function concat<T extends readonly unknown[], U extends readonly unknown[]>(
  a: [...T],
  b: [...U]
): [...T, ...U] {
  return [...a, ...b];
}

const result = concat([1, 2] as const, ["a", "b"] as const);
// Type: readonly [1, 2, "a", "b"]

// Tuple type inference from arrays
function tuple<T extends unknown[]>(...args: T): T {
  return args;
}

const inferred = tuple(1, "hello", true);
// Type: [number, string, boolean]

// Mapped tuple types
type NullableTuple<T extends readonly unknown[]> = {
  [P in keyof T]: T[P] | null;
};

type Original = [string, number, boolean];
type Nullable = NullableTuple<Original>; // [string | null, number | null, boolean | null]

// Tuple with spread in function parameters
function multiplyTuple<T extends readonly (number | string)[]>(
  ...args: T
): { [P in keyof T]: number } {
  return args.map((a) => (typeof a === "number" ? a * 2 : parseInt(a) * 2)) as any;
}

const doubled = multiplyTuple(1, "5", 3);
// Type: [number, number, number]
`

### Real-World Use Cases

**React useState**: Returns [T, Dispatch<SetStateAction<T>>] — a tuple.

**Key-Value Stores**: Array of [string, V] tuples for Map entries.

**CSV Parsing**: Each row represented as a tuple.

**Database Result Rows**: Typed tuples for query results.

### Common Mistakes

**Exceeding Tuple Length**: Accessing indices beyond the declared length returns undefined.

**Mutating Tuple Length**: Using push or pop on tuples can bypass type safety.

**Confusing Tuples with Arrays**: Tuples have fixed lengths and per-element types; arrays are homogenous.

### Best Practices

1. Use tuples for fixed-size, position-meaningful data
2. Prefer objects/interfaces for data with more than 3-4 elements
3. Use destructuring for clarity
4. Add s const for literal tuple inference

### Performance Considerations

Tuples compile to JavaScript arrays with zero overhead. Length constraints are compile-time only.

### Interview Questions

1. How is a tuple different from an array in TypeScript?
2. How do variadic tuple types work (TS 4.0+)?
3. How do you infer a tuple type from an array literal?

### Coding Challenges

1. Implement a type-safe zip function that returns an array of tuples
2. Create a function that splits an array into two tuples

### Related Topics

- Array Types
- Variadic Tuple Types
- Rest Parameters

## Labeled tuples

### What It Is

Labeled tuples (TS 4.0+) allow naming each element in a tuple type: [name: string, age: number]. Labels improve readability and documentation without affecting the type behavior.

### Why It Is Important

Labels make tuple types self-documenting. They help developers understand what each position represents without needing to look at usage context, especially in function signatures and complex type definitions.

### How It Works Internally

Labels are purely cosmetic in the type system. [name: string, age: number] is structurally identical to [string, number]. Labels appear in IntelliSense and error messages but don't affect assignability.

### Syntax

`	ypescript
// Labeled tuple
type Person = [name: string, age: number];

// Function with labeled tuple parameter
function setPerson(person: [name: string, age: number]): void {
  const [name, age] = person;
  console.log(${name} is  years old);
}

// Labeled return tuple
function getPerson(): [name: string, age: number] {
  return ["Alice", 30];
}

// Destructuring with labels
const [personName, personAge] = getPerson();
`

### Beginner Examples

`	ypescript
// API result with labels
type ApiResult = [statusCode: number, body: string, headers: Record<string, string>];

// Function that returns labeled tuple
function divideAndRemainder(
  dividend: number,
  divisor: number
): [quotient: number, remainder: number] {
  return [Math.floor(dividend / divisor), dividend % divisor];
}

const [q, r] = divideAndRemainder(10, 3);
console.log(Quotient: , Remainder: );

// Range result
function parseRange(input: string): [start: number, end: number] | null {
  const parts = input.split("-").map(Number);
  if (parts.length !== 2 || parts.some(isNaN)) return null;
  return [parts[0], parts[1]];
}
`

### Intermediate Examples

`	ypescript
// Mixed labeled and unlabeled elements
type HttpResponse = [status: number, ...string[]];

function createResponse(
  status: number,
  ...messages: string[]
): HttpResponse {
  return [status, ...messages];
}

// Labeled tuple in generic context
function createPair<K extends string, V>(
  key: K,
  value: V
): [key: K, value: V] {
  return [key, value];
}

const pair = createPair("name", "Alice");
// Type: [key: "name", value: string]

// Labeled tuples in discriminated unions
type Event2 =
  | [type: "click", x: number, y: number]
  | [type: "keypress", key: string]
  | [type: "resize", width: number, height: number];

function handleEvent(event: Event2): void {
  const [type] = event;
  switch (type) {
    case "click": {
      const [, x, y] = event;
      console.log(Clicked at , );
      break;
    }
    case "keypress": {
      const [, key] = event;
      console.log(Pressed );
      break;
    }
    case "resize": {
      const [, width, height] = event;
      console.log(Resized to x);
      break;
    }
  }
}
`

### Advanced Examples

`	ypescript
// Labeled tuples with generics
function useState<T>(
  initial: T
): [value: T, setValue: (newValue: T) => void] {
  let state = initial;
  return [
    state,
    (newValue: T) => {
      state = newValue;
    },
  ];
}

// Async result tuple
type AsyncResult<T, E = Error> = [
  data: T | null,
  error: E | null,
  isLoading: boolean
];

async function useFetch<T>(url: string): Promise<AsyncResult<T>> {
  try {
    const response = await fetch(url);
    const data = await response.json();
    return [data as T, null, false];
  } catch (error) {
    return [null, error as Error, false];
  }
}

// Labeled tuple of functions
type Middleware<T> = [
  name: string,
  handler: (context: T, next: () => void) => void
];

function createPipeline<T>(middlewares: Middleware<T>[]): void {
  for (const [name, handler] of middlewares) {
    console.log(Registering middleware: );
  }
}
`

### Real-World Use Cases

**React Custom Hooks**: Labeled tuples for return values (e.g., [value: T, setter: Dispatch<T>]).

**API Client Responses**: [data: T, error: E | null, status: number].

**Validation Results**: [isValid: boolean, errors: string[]].

### Common Mistakes

**Over-labeling**: Adding labels doesn't change type behavior — they're documentation only.

**Confusing Labels with Properties**: Labels don't create named properties; you can't access person.name, only person[0].

### Best Practices

1. Use labels when the meaning of each position isn't obvious
2. Keep labels concise but descriptive
3. Use labels in public API return types
4. Combine labels with destructuring for readability

### Performance Considerations

Labels are compile-time only — zero runtime cost.

### Interview Questions

1. Do labeled tuples affect type compatibility?
2. How do you destructure a labeled tuple?
3. When would you use labeled tuples?

### Coding Challenges

1. Create a labeled tuple for an API response with status, data, and error
2. Implement a function that returns a labeled tuple with 4+ elements

### Related Topics

- Tuple Types
- Destructuring
- Type Aliases

## Optional and rest elements

### What It Is

Optional tuple elements (TS 3.0+) allow omitting certain trailing elements with ? syntax. Rest elements (TS 4.0+) allow a tuple to contain a variable number of elements of a specific type at a specific position.

### Why It Is Important

Optional elements make tuples flexible for varying data while maintaining position typing. Rest elements enable tuples that capture leading, middle, or trailing variable-length sequences while keeping type safety.

### How It Works Internally

Optional elements have type T | undefined and increase the length range. Rest elements use variadic tuple syntax ...T[] to represent zero or more elements of type T at a specific position in the tuple.

### Syntax

`	ypescript
// Optional elements
type OptionalTuple = [string, number?];
const a: OptionalTuple = ["hello"];
const b: OptionalTuple = ["hello", 42];

// Rest elements
type WithRest = [string, ...number[]];
const c: WithRest = ["hello"];
const d: WithRest = ["hello", 1, 2, 3];

// Rest at start
type StartsWithRest = [...number[], string];
const e: StartsWithRest = [1, 2, 3, "end"];

// Rest in middle
type MiddleRest = [string, ...number[], boolean];
const f: MiddleRest = ["start", true];
const g: MiddleRest = ["start", 1, 2, 3, true];
`

### Beginner Examples

`	ypescript
// Optional element
type Config = [name: string, port?: number];

function createServer(config: Config): void {
  const [name, port = 8080] = config;
  console.log(Server  on port );
}

createServer(["api"]);
createServer(["api", 3000]);

// Rest elements for variadic functions
function logMessages(level: string, ...messages: string[]): void {
  console.log([], ...messages);
}

// Tuple with rest for CSV
type CsvLine = [string, ...(string | number)[]];

const line1: CsvLine = ["Alice"];
const line2: CsvLine = ["Bob", 30, "NYC", true];
`

### Intermediate Examples

`	ypescript
// Multiple optional elements
type Header = [name: string, value?: string, priority?: number];

const h1: Header = ["Content-Type"];
const h2: Header = ["Content-Type", "application/json"];
const h3: Header = ["Content-Type", "application/json", 1];

// Rest with generics
function createTuple<T extends unknown[]>(
  first: T[0],
  ...rest: T
): [T[0], ...T] {
  return [first, ...rest];
}

// Partial application with tuples
type PartialArgs<T extends readonly unknown[], U extends readonly unknown[]> =
  T extends [...infer First, ...U] ? First : never;

// Rest with spread in function parameters
function mergeTuples<T extends readonly unknown[], U extends readonly unknown[]>(
  a: [...T],
  b: [...U]
): [...T, ...U] {
  return [...a, ...b];
}

const merged = mergeTuples([1, 2] as const, ["a"] as const);
// Type: readonly [1, 2, "a"]
`

### Advanced Examples

`	ypescript
// Conditional optional elements
type HttpRequest<
  Method extends "GET" | "POST" | "PUT" | "DELETE"
> = Method extends "GET"
  ? [url: string]
  : [url: string, body: unknown, headers?: Record<string, string>];

function makeRequest<M extends "GET" | "POST">(
  method: M,
  ...args: HttpRequest<M>
): void {
  if (method === "GET") {
    const [url] = args;
    console.log(GET );
  } else {
    const [url, body] = args;
    console.log(POST , body);
  }
}

makeRequest("GET", "/api/users");
makeRequest("POST", "/api/users", { name: "Alice" });

// String parsing with tuple rest
function parsePath(path: string): [prefix: string, ...segments: string[]] {
  const parts = path.split("/").filter(Boolean);
  return [parts[0], ...parts.slice(1)];
}

const [prefix, ...rest] = parsePath("/api/v1/users/123");

// Variadic tuple for function composition
type Composer<T extends readonly ((...args: any[]) => any)[]> = {
  [P in keyof T]: T[P];
};

function pipe<T extends readonly ((...args: any[]) => any)[]>(
  ...fns: T
): (...args: Parameters<T[0]>) => ReturnType<T[number]> {
  return (initial: any) =>
    fns.reduce((acc, fn) => fn(acc), initial);
}
`

### Real-World Use Cases

**HTTP Request Builder**: Tuples with optional body/headers based on method.

**Configuration Tuples**: Required name with optional settings.

**CSV Parsing**: Variable-length rows with optional cells.

### Common Mistakes

**Placing Optional Before Required**: Optional elements must come after required elements (except rest).

**Using Rest Before Optional**: Rest elements can't be followed by optional elements.

### Best Practices

1. Use optional elements for truly optional trailing data
2. Use rest elements for variable-length middle sections
3. Consider using objects for tuples with many optional elements

### Performance Considerations

Optional and rest elements are compile-time only — no runtime overhead.

### Interview Questions

1. How do optional tuple elements differ from | undefined in unions?
2. Can rest elements appear at any position in a tuple?
3. How do optional elements affect the length property type?

### Coding Challenges

1. Create a tuple with both optional and rest elements
2. Implement a function that accepts a variable number of arguments using rest tuple

### Related Topics

- Variadic Tuple Types
- Optional Parameters
- Rest Parameters

## Readonly tuples

### What It Is

Readonly tuples prevent modification of tuple elements and the tuple itself. They're created with eadonly [T, U] syntax or by applying s const to tuple literals.

### Why It Is Important

Readonly tuples enforce immutability at the type level, preventing accidental mutations. They're essential when tuples should remain constant — like configuration values, coordinate constants, or function arguments that must not be modified.

### How It Works Internally

eadonly [T, U] is shorthand for Readonly<[T, U]>, making all numeric indices readonly and preventing methods like push, pop, and ill.

### Syntax

`	ypescript
// Readonly tuple
const point: readonly [number, number] = [10, 20];
// point[0] = 5; // Error: readonly
// point.push(30); // Error: push doesn't exist on readonly

// as const creates readonly tuples
const config = ["localhost", 3000] as const;
// Type: readonly ["localhost", 3000]

// Function parameter
function processPair(pair: readonly [string, number]): void {
  const [key, value] = pair;
  console.log(pair[0]);
  // pair[0] = "new"; // Error
}
`

### Beginner Examples

`	ypescript
// Immutable coordinates
const origin: readonly [number, number] = [0, 0];
const dimensions: readonly [number, number] = [1920, 1080];

// Using as const
const STATUS_CODE = [200, "OK"] as const;
type StatusCode = typeof STATUS_CODE;
// readonly [200, "OK"]

// Readonly tuple in function
function calculateDistance(
  a: readonly [number, number],
  b: readonly [number, number]
): number {
  const dx = a[0] - b[0];
  const dy = a[1] - b[1];
  return Math.sqrt(dx * dx + dy * dy);
}

calculateDistance([0, 0], [3, 4]);
`

### Intermediate Examples

`	ypescript
// Readonly array of readonly tuples
const COLORS: readonly [string, string][] = [
  ["primary", "#3498db"],
  ["secondary", "#2ecc71"],
  ["danger", "#e74c3c"],
] as const;

// Readonly spread
function mergePoints(
  a: readonly [number, number],
  b: readonly [number, number]
): readonly [number, number, number, number] {
  return [...a, ...b];
}

// Class with readonly tuple
class Rectangle {
  constructor(
    private readonly topLeft: readonly [number, number],
    private readonly bottomRight: readonly [number, number]
  ) {}

  get width(): number {
    return this.bottomRight[0] - this.topLeft[0];
  }

  get height(): number {
    return this.bottomRight[1] - this.topLeft[1];
  }
}
`

### Advanced Examples

`	ypescript
// Deep readonly tuple
type DeepReadonlyTuple<T extends readonly unknown[]> = {
  readonly [P in keyof T]: T[P] extends object
    ? DeepReadonly<T[P]>
    : T[P];
};

type DeepReadonly<T> = T extends readonly any[]
  ? DeepReadonlyTuple<T>
  : T extends object
    ? { readonly [P in keyof T]: DeepReadonly<T[P]> }
    : T;

// Readonly branded tuple
type BrandedTuple<T extends readonly unknown[], Brand extends string> =
  T & { readonly __brand: Brand };

type UserId = BrandedTuple<readonly [string, number], "User">;

function createUser(name: string, id: number): UserId {
  return [name, id] as UserId;
}

const user = createUser("Alice", 1);

// Readonly tuple mapping
type MapTuple<T extends readonly unknown[], Fn extends (item: any) => any> = {
  readonly [P in keyof T]: Fn extends (item: T[P]) => infer R ? R : never;
};

type Stringified = MapTuple<readonly [1, true, "hello"], (x: any) => string>;
// readonly [string, string, string]

// Readonly discriminated union tuple
type StateTransition = readonly [
  from: string,
  to: string,
  action: string
];

const transitions: readonly StateTransition[] = [
  ["pending", "confirmed", "confirm"],
  ["confirmed", "shipped", "ship"],
  ["shipped", "delivered", "deliver"],
] as const;
`

### Real-World Use Cases

**Configuration Constants**: Immutable tuples for app configuration.

**Coordinate Systems**: Points, vectors, and dimensions as readonly tuples.

**State Machine Transitions**: Readonly tuples defining valid state transitions.

### Common Mistakes

**Assuming Spread Creates a Mutable Copy**: Spreading a readonly tuple into a mutable array gives a mutable result — explicit typing may be needed.

**Forgetting s const for Literal Tuples**: Without s const, ["hello", 42] is (string | number)[], not a tuple.

### Best Practices

1. Use eadonly tuples for function parameters that should not be mutated
2. Use s const for tuple literal inference
3. Prefer eadonly [T, U] over [T, U] by default
4. Use readonly tuples for constants and configuration

### Performance Considerations

Zero runtime cost — readonly is purely a compile-time constraint.

### Interview Questions

1. How do readonly tuples differ from regular tuples?
2. What does s const do to a tuple literal?
3. Can you spread a readonly tuple into a mutable array?

### Coding Challenges

1. Implement a deep readonly tuple type
2. Create a function that safely merges two readonly tuples

### Related Topics

- ReadonlyArray
- as const
- Immutability in TypeScript
