# Type Inference - Contextual typing, best common type, widening, literal inference, type flow analysis

## Introduction

Type inference is the mechanism by which TypeScript determines types without explicit annotations. It is one of the language's most powerful features, enabling concise code while maintaining type safety. This file covers contextual typing (types inferred from context), best common type (for arrays and conditional expressions), type widening (literal to primitive), literal type inference (const vs let), and control flow analysis (type narrowing through code paths).

## Contextual Typing

### What It Is

Contextual typing (also called "bidirectional inference") is when TypeScript infers the types of expressions based on their expected type from the surrounding context. For example, when you pass a callback to `Array.prototype.map`, TypeScript infers the callback's parameter types from the array element type.

### Why It Is Important

Contextual typing makes TypeScript feel less verbose by eliminating redundant type annotations. It enables powerful patterns like typed event handlers, generic callback functions, and type-safe builder patterns where the context determines the types.

### How It Works Internally

The type checker performs bidirectional inference. In addition to inferring types bottom-up (from expressions to their parents), it also infers top-down (from expected types to expressions). The expected type is called the "contextual type." The checker attempts to reconcile both directions.

### Syntax

```typescript
// Contextual typing infers the parameter types from the callback signature
const numbers = [1, 2, 3];
const strings = numbers.map((num) => num.toString());
// num is inferred as number from the array type

// Event handlers
document.addEventListener('click', (event) => {
  // event is inferred as MouseEvent
  console.log(event.clientX, event.clientY);
});
```

### Beginner Examples

```typescript
// Array methods
const items = ['apple', 'banana', 'cherry'];
items.forEach((item, index) => {
  // item: string, index: number
  console.log(`${index}: ${item}`);
});

// Promise chains
Promise.resolve(42)
  .then((value) => {
    // value: number
    return value.toString();
  })
  .then((str) => {
    // str: string
    console.log(str.length);
  });
```

### Intermediate Examples

```typescript
// Generic function with contextual typing
function createPair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const pair = createPair('hello', 42);
// Type: [string, number]

// Builder pattern
class QueryBuilder<T> {
  constructor(private table: string) {}

  where<K extends keyof T>(field: K, value: T[K]): this {
    console.log(`WHERE ${String(field)} = ${value}`);
    return this;
  }
}

interface User {
  id: number;
  name: string;
  email: string;
}

const query = new QueryBuilder<User>('users');
query.where('name', 'Alice'); // OK: name is keyof User, 'Alice' is string
// query.where('age', 30); // Error: age is not a property of User
```

### Advanced Examples

```typescript
// Contextual typing with overloads
function process(value: string): string[];
function process(value: number): number[];
function process(value: string | number): (string | number)[] {
  if (typeof value === 'string') {
    return value.split('');
  }
  return [value];
}

// Context determines which overload is used
const strResult = process('hello'); // string[]
const numResult = process(42); // number[]

// Contextual typing in object literals
interface Config<T> {
  url: string;
  transform: (data: unknown) => T;
}

const config: Config<number> = {
  url: '/api/data',
  transform: (data) => {
    // Return type must be number based on Config<number>
    return Number(data);
  },
};
```

### Real-World Use Cases

```typescript
// React event handlers with contextual typing
import React from 'react';

const Form = () => {
  const [value, setValue] = React.useState('');

  const handleChange: React.ChangeEventHandler<HTMLInputElement> = (event) => {
    // event.target.value is inferred as string
    setValue(event.target.value);
  };

  return <input value={value} onChange={handleChange} />;
};

// Express route handlers
import { RequestHandler } from 'express';

const getUser: RequestHandler<{ id: string }> = (req, res) => {
  // req.params.id is inferred as string
  const id = req.params.id;
  res.json({ id });
};
```

### Common Mistakes

```typescript
// MISTAKE: Expecting contextual typing to work without type annotation on the target
// const items = [].map(x => x.toString()); // Error: x is implicitly any
// CORRECT: const items = [1,2,3].map(x => x.toString());

// MISTAKE: Breaking contextual typing with intermediate variables
// const handler = (event) => { event.clientX }; // Error: event is any
// CORRECT: Keep inline or annotate the parameter
```

### Best Practices

- Leverage contextual typing for callbacks, event handlers, and generic functions.
- Avoid breaking the inference chain with unnecessary intermediate variables.
- Use type annotations on function signatures (parameters and return types), let contextual typing handle the internals.
- In React, use event handler types like `React.ChangeEventHandler` for contextual typing.

### Performance Considerations

Contextual typing adds complexity to the type inference algorithm but has no runtime cost. In rare cases, deeply nested contextual typing can slow down the type checker.

### Interview Questions

1. What is contextual typing and how does it differ from regular type inference?
2. How does contextual typing work with generic functions?
3. Why does `[].map(x => x.toString())` fail without a type annotation?

### Coding Challenges

1. Create a typed event emitter that uses contextual typing for event handlers based on event names.
2. Build a generic `createStore` function where the reducer's state type is inferred from the initial state.

### Related Topics

- Type inference
- Best common type
- Generic type inference

## Best Common Type

### What It Is

Best common type is the algorithm TypeScript uses to infer the type of an array literal or conditional expression when the elements/branches have different types. It finds the most specific common supertype that all elements can be assigned to.

### Why It Is Important

Best common type enables TypeScript to type arrays and conditionals without explicit annotations, while still catching type inconsistencies. It balances specificity with permissiveness.

### How It Works Internally

TypeScript iterates over the candidate types and finds a type that all candidates are assignable to. It prefers the most specific type. For `[1, 2, 3]`, the best common type is `number`. For `[1, 'hello', true]`, the best common type is `string | number | boolean`.

### Syntax

```typescript
// Array literal inference
const arr1 = [1, 2, 3]; // number[]
const arr2 = [1, 'hello', true]; // (string | number | boolean)[]

// Conditional expression
const x = Math.random() > 0.5 ? 'yes' : 42; // string | number
```

### Beginner Examples

```typescript
// Mixed arrays
const mixed = [10, 'hello', true];
// Type: (string | number | boolean)[]

// Nested arrays
const nested = [[1, 2], [3, 4]]; // number[][]
const mixedNested = [[1, 2], ['a', 'b']]; // (string | number)[][]

// Conditional types
const result = someCondition
  ? { type: 'success', data: 'ok' }
  : { type: 'error', message: 'fail' };
// Type: { type: string; data?: string; message?: string }
```

### Intermediate Examples

```typescript
// Best common type with objects
const shapes = [
  { kind: 'circle', radius: 5 },
  { kind: 'square', side: 10 },
];
// Type: ({ kind: string; radius: number } | { kind: string; side: number })[]

// To get a tighter type, use a union or widening
interface Circle { kind: 'circle'; radius: number; }
interface Square { kind: 'square'; side: number; }

const typedShapes: (Circle | Square)[] = [
  { kind: 'circle', radius: 5 },
  { kind: 'square', side: 10 },
];

// Best common type with tuple inference
const tuple = [1, 'hello'] as const;
// Type: readonly [1, 'hello']
```

### Advanced Examples

```typescript
// Best common type and discriminated unions
type Result<T> = { success: true; data: T } | { success: false; error: string };

function processResults<T>(items: T[]): Result<T>[] {
  return items.map(item => {
    try {
      const data = processItem(item);
      return { success: true, data };
    } catch (err) {
      return { success: false, error: String(err) };
    }
  });
}

// The return type is correctly inferred as Result<T>[]

// Best common type with conditional returns
function create(defaultValue: string | number) {
  if (typeof defaultValue === 'string') {
    return { type: 'string', value: defaultValue };
  }
  return { type: 'number', value: defaultValue };
}
// Return type: { type: string; value: string | number }
```

### Real-World Use Cases

```typescript
// API response handling
function parseResponses(responses: unknown[]) {
  return responses.map(r => {
    if (typeof r === 'string') {
      return JSON.parse(r);
    }
    return r;
  });
}
// Return type: unknown[] (best common type of JSON.parse return and unknown)

// Form validation errors
const errors = [
  { field: 'name', message: 'Required' },
  { field: 'email', message: 'Invalid format' },
];
// Type: { field: string; message: string }[]
```

### Common Mistakes

```typescript
// MISTAKE: Expecting discriminated union from untyped object literals
// const items = [{ kind: 'a', val: 1 }, { kind: 'b', val: 'x' }];
// items[0].val // string | number, not narrowed by kind

// MISTAKE: Assuming best common type will use a named interface
// TypeScript infers structural types, not named ones
```

### Best Practices

- Use explicit type annotations when you need a specific union or interface type for arrays.
- Use `as const` to preserve literal types in arrays.
- Use discriminated unions explicitly with type annotations for object arrays with different shapes.
- Be aware that best common type may produce broader types than desired.

### Performance Considerations

Best common type computation is efficient for most cases. For very large union types (hundreds of members), it can slow down type checking.

### Interview Questions

1. What is the best common type algorithm in TypeScript?
2. How does TypeScript determine the type of `[1, 'hello', true]`?
3. How can `as const` affect best common type inference?

### Coding Challenges

1. Write a function that accepts an array of mixed types and returns a properly typed discriminated union.
2. Create a type-safe `merge` function that finds the best common type of two arrays.

### Related Topics

- Contextual typing
- Type widening
- Union types

## Type Widening

### What It Is

Type widening is the process by which TypeScript expands literal types to their broader primitive types. For example, when you declare `let x = 'hello'`, TypeScript infers `x` as `string`, not `'hello'`. This allows reassignment while maintaining type safety.

### Why It Is Important

Widening strikes a balance between precision and practicality. Without widening, `let x = 'hello'` would be typed as the literal `'hello'`, preventing any reassignment to `'world'`. With widening, the variable can be reassigned within the primitive type.

### How It Works Internally

TypeScript widens literal types (`'hello'`, `42`, `true`) to their base types (`string`, `number`, `boolean`) for `let`, `var`, and mutable object properties. `const` declarations are not widened because they cannot be reassigned. The `as const` assertion prevents widening.

### Syntax

```typescript
// const declarations preserve literal types
const greeting = 'hello'; // type: 'hello'

// let declarations widen literals
let name = 'world'; // type: string

// as const prevents widening
const config = {
  url: 'https://api.example.com' as const,
  method: 'GET',
};
// config.url: 'https://api.example.com' (not widened)
```

### Beginner Examples

```typescript
// Literal widening
const pi = 3.14159; // type: 3.14159
let radius = 5; // type: number

// Boolean widening
const isProduction = true; // type: true
let isDebug = false; // type: boolean

// Null/undefined widening
const empty = null; // type: null
let maybe = undefined; // type: undefined (not widened to any)
```

### Intermediate Examples

```typescript
// Object property widening
const user = {
  name: 'Alice', // type: string (widened from 'Alice')
  age: 30,       // type: number (widened from 30)
};
// TypeScript infers: { name: string; age: number }

// Preventing widening with explicit type
const user2: { name: 'Alice'; age: 30 } = {
  name: 'Alice',
  age: 30,
};

// Array widening
const arr = [1, 2, 3]; // type: number[] (widened from [1, 2, 3])

// Preserving literal types in arrays
const strict = [1, 2, 3] as const; // type: readonly [1, 2, 3]
```

### Advanced Examples

```typescript
// Widening and discriminated unions
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';

function makeRequest(url: string, method: HttpMethod) {
  // Implementation
}

// Without widening control, this breaks:
const myMethod = 'GET'; // widened to string
// makeRequest('/api', myMethod); // Error: string not assignable to HttpMethod

// Fix with const assertion:
const myMethod2 = 'GET' as const; // type: 'GET'
makeRequest('/api', myMethod2); // OK

const config = {
  method: 'GET' as const,
};
makeRequest('/api', config.method); // OK

// Widening with function parameters
function setTheme(theme: 'light' | 'dark') { }

const preferred = 'dark'; // widened to string
// setTheme(preferred); // Error!

// Fix: annotate or use as const
const preferred2: 'light' | 'dark' = 'dark';
setTheme(preferred2);

// Class property widening
class Config {
  readonly method = 'GET'; // type: 'GET' (readonly prevents widening)
  mode = 'production'; // type: string (mutable, widened)
}
```

### Real-World Use Cases

```typescript
// Redux action types
const ADD_TODO = 'ADD_TODO' as const;
const REMOVE_TODO = 'REMOVE_TODO' as const;

type Action =
  | { type: typeof ADD_TODO; payload: string }
  | { type: typeof REMOVE_TODO; payload: string };

// Event handler map
const handlers = {
  click: (e: MouseEvent) => {},
  keydown: (e: KeyboardEvent) => {},
} as const;

type EventName = keyof typeof handlers;
// 'click' | 'keydown' (preserved, not string)
```

### Common Mistakes

```typescript
// MISTAKE: Forgetting that let widens literals
let status = 'active'; // string, not 'active'
// status = 'inactive'; // OK, but loses type info

// MISTAKE: Expecting const objects to have literal properties
// const obj = { name: 'test' }; // obj.name is string, not 'test'

// MISTAKE: Not using as const when literal types are needed
// Especially in discriminated unions and tuple patterns
```

### Best Practices

- Use `const` for values that should never change (preserves literal types).
- Use `as const` for objects and arrays that need literal type preservation.
- Use explicit type annotations when you need specific literal types.
- Be mindful of widening when using discriminated unions with `let` variables.
- Prefer `const` over `let` when reassignment is not needed.

### Performance Considerations

Widening decisions are made at compile time with no runtime impact. The `as const` assertion adds no runtime overhead (it is erased).

### Interview Questions

1. What is type widening and when does it occur?
2. Why does `let x = 'hello'` infer as `string` but `const x = 'hello'` infers as `'hello'`?
3. How does `as const` prevent type widening?

### Coding Challenges

1. Write a function that accepts a string literal type and uses `as const` to preserve the literal in returned objects.
2. Create a type-safe event system where event names are string literals and handlers are typed accordingly.

### Related Topics

- Literal type inference
- as const
- Type narrowing

## Literal Type Inference

### What It Is

Literal type inference is TypeScript's ability to infer literal types (specific values like `'hello'`, `42`, `true`) rather than their broader base types. This happens primarily with `const` declarations and `as const` assertions.

### Why It Is Important

Literal types enable precise type checking for discriminated unions, function overloads, and configuration objects. They allow TypeScript to catch mismatches between expected and actual values.

### How It Works Internally

When a value is declared with `const`, TypeScript infers the most specific type possible. For `const x = 'hello'`, the type is `'hello'` (a string literal type). For `const y = 42`, the type is `42` (a numeric literal type).

### Syntax

```typescript
// const declarations get literal types
const name = 'TypeScript'; // type: 'TypeScript'
const version = 5.3; // type: 5.3
const isAwesome = true; // type: true

// let/var declarations get widened types
let name2 = 'TypeScript'; // type: string

// as const assertion
const config = { key: 'value' } as const;
// config.key: 'value' (literal)
```

### Beginner Examples

```typescript
// String literal types
const SUCCESS = 'success';
const ERROR = 'error';
type Status = typeof SUCCESS | typeof ERROR;
// 'success' | 'error'

// Numeric literal types
const MAX_RETRIES = 3;
type RetryCount = typeof MAX_RETRIES;
// 3

// Boolean literal types
const ENABLED = true;
type FeatureFlag = typeof ENABLED;
// true
```

### Intermediate Examples

```typescript
// Literal types in discriminated unions
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; side: number }
  | { kind: 'triangle'; base: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'circle': return Math.PI * shape.radius ** 2;
    case 'square': return shape.side ** 2;
    case 'triangle': return (shape.base * shape.height) / 2;
  }
}

// Literal types for tuple inference
const position = [10, 20] as const;
// type: readonly [10, 20]
const x = position[0]; // type: 10, not number
```

### Advanced Examples

```typescript
// Template literal types with inference
type EventName<T extends string> = `${T}Changed`;
type NameEvent = EventName<'name'>; // 'nameChanged'

// Literal type from function parameter
function setSize<T extends number>(size: T): T {
  return size;
}
const size = setSize(42); // type: 42

// Mapping over literal types
type Color = 'red' | 'green' | 'blue';
type ColorMap = { [K in Color]: string };
// { red: string; green: string; blue: string }
```

### Real-World Use Cases

```typescript
// HTTP status codes
type HttpStatus = 200 | 201 | 204 | 400 | 401 | 403 | 404 | 500;

function isSuccess(status: HttpStatus): boolean {
  return status >= 200 && status < 300;
}

// Route paths with literal types
type Route = '/' | '/about' | '/contact' | '/users/:id';

function navigate(route: Route): void {
  window.history.pushState({}, '', route);
}

// CSS unit literals
type CssUnit = 'px' | 'em' | 'rem' | '%' | 'vh' | 'vw';

function toCss(value: number, unit: CssUnit): string {
  return `${value}${unit}`;
}
```

### Common Mistakes

```typescript
// MISTAKE: Using let when you need literal types
// let status = 'active'; // string, not 'active'
// CORRECT: const status = 'active'; // 'active'

// MISTAKE: Forgetting as const for object literals
// const config = { theme: 'dark' };
// config.theme // string, not 'dark'

// MISTAKE: Overly broad types for function parameters
// function greet(prefix: string) // too broad
// function greet(prefix: 'Mr.' | 'Ms.' | 'Dr.') // better
```

### Best Practices

- Use `const` declarations to preserve literal types.
- Use `as const` for objects, arrays, and function return types.
- Use literal types in discriminated unions for exhaustive checking.
- Use template literal types with literal types for type-safe string manipulation.
- Combine literal types with `keyof` for type-safe property access.

### Performance Considerations

Literal types are more precise and can increase the complexity of type checking in very large unions. However, for typical usage, the performance impact is negligible.

### Interview Questions

1. When does TypeScript infer a literal type vs a widened type?
2. What is the difference between `const` and `let` for type inference?
3. How does `as const` work with nested objects?

### Coding Challenges

1. Create a function that takes a literal string type and returns a template literal type.
2. Build a type-safe router that uses literal types for route definitions.

### Related Topics

- Type widening
- as const
- Template literal types

## Control Flow Analysis

### What It Is

Control flow analysis (CFA) is TypeScript's ability to narrow the type of a variable based on code flow constructs like `if`, `switch`, `typeof`, `instanceof`, and custom type guards. It tracks how types change through different code paths.

### Why It Is Important

CFA enables safe access to properties that are only available after certain checks. It reduces the need for type assertions and makes code both safer and more readable by eliminating redundant null checks and type casts.

### How It Works Internally

TypeScript builds a control flow graph and tracks the type of each variable at each point in the graph. When a type guard condition is evaluated, TypeScript narrows the type in the truthy branch and (optionally) in the falsy branch. The analysis handles loops, early returns, and thrown exceptions.

### Syntax

```typescript
function process(value: string | null): string {
  if (value === null) {
    return 'default';
  }
  // value is narrowed to string here
  return value.toUpperCase();
}

// typeof type guard
function format(input: string | number): string {
  if (typeof input === 'string') {
    return input.trim();
  }
  return input.toFixed(2);
}
```

### Beginner Examples

```typescript
// Null checking
function getLength(value: string | null): number {
  if (value === null) {
    return 0;
  }
  return value.length; // value is string
}

// Truthiness narrowing
function getFirst(items: string[] | undefined): string | undefined {
  if (!items) {
    return undefined;
  }
  return items[0]; // items is string[]
}

// Equality narrowing
function compare(a: string | number, b: string | boolean) {
  if (a === b) {
    // Both are narrowed to string
    console.log(a.toUpperCase());
  }
}
```

### Intermediate Examples

```typescript
// instanceof narrowing
class Dog { bark() { return 'Woof!'; } }
class Cat { meow() { return 'Meow!'; } }

function makeSound(animal: Dog | Cat): string {
  if (animal instanceof Dog) {
    return animal.bark(); // animal is Dog
  }
  return animal.meow(); // animal is Cat
}

// in operator narrowing
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ('swim' in animal) {
    animal.swim(); // animal is Fish
  } else {
    animal.fly(); // animal is Bird
  }
}

// Discriminated union narrowing
type Shape2 =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; side: number };

function area2(shape: Shape2): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2; // shape is circle
    case 'square':
      return shape.side ** 2; // shape is square
  }
}
```

### Advanced Examples

```typescript
// User-defined type guards
interface Car { drive: () => void; fuel: 'gas' | 'electric'; }
interface Boat { sail: () => void; fuel: 'diesel'; }

function isCar(vehicle: Car | Boat): vehicle is Car {
  return 'drive' in vehicle;
}

function operate(vehicle: Car | Boat) {
  if (isCar(vehicle)) {
    vehicle.drive(); // vehicle is Car
  } else {
    vehicle.sail(); // vehicle is Boat
  }
}

// Assertion functions
function assertNonNull<T>(value: T): asserts value is NonNullable<T> {
  if (value === null || value === undefined) {
    throw new Error('Value must not be null or undefined');
  }
}

function processUser(user: { name: string } | null) {
  assertNonNull(user);
  user.name; // user is narrowed to { name: string }
}

// Control flow with complex conditions
interface Response2 {
  status: number;
  data?: unknown;
  error?: { message: string };
}

function handleResponse(response: Response2) {
  if (response.status >= 200 && response.status < 300 && response.data) {
    return response.data; // data is truthy
  }
  if (response.error) {
    throw new Error(response.error.message);
  }
  throw new Error(`Unexpected status: ${response.status}`);
}

// Type narrowing with array methods
const items = [1, null, 3, undefined, 5];
const validItems = items.filter((item): item is number => item !== null && item !== undefined);
// validItems: number[]
```

### Real-World Use Cases

```typescript
// Form state management
type FormState<T> =
  | { status: 'idle' }
  | { status: 'filling'; data: Partial<T> }
  | { status: 'submitting'; data: T }
  | { status: 'success'; data: T; response: unknown }
  | { status: 'error'; data: T; error: string };

function FormComponent<T>(state: FormState<T>) {
  switch (state.status) {
    case 'idle':
      return { message: 'Start filling' };
    case 'filling':
    case 'submitting':
      return { data: state.data }; // both have data
    case 'success':
      return { data: state.data, result: state.response };
    case 'error':
      return { error: state.error };
  }
}

// API response handling
type ApiResult<T> =
  | { success: true; data: T; meta?: { total: number } }
  | { success: false; error: string; code?: number };

function handleApiResult<T>(result: ApiResult<T>) {
  if (result.success) {
    return { data: result.data, meta: result.meta };
  }
  throw new Error(`${result.code}: ${result.error}`);
}
```

### Common Mistakes

```typescript
// MISTAKE: Assuming narrowing persists after reassignment
let value: string | number = 'hello';
if (typeof value === 'string') {
  value = 42; // value is now number
  value.toFixed(2); // OK
}

// MISTAKE: Not handling all cases in discriminated unions
// TypeScript's exhaustiveness check helps here

// MISTAKE: Over-relying on type assertions instead of proper narrowing
// (x as string).toUpperCase(); // Unsafe
// Prefer: if (typeof x === 'string') x.toUpperCase();
```

### Best Practices

- Use discriminated unions with exhaustive switch statements.
- Prefer type guards (`is`) over type assertions for reusable narrowing logic.
- Use assertion functions (`asserts`) for validation-style narrowing.
- Let TypeScript's CFA do the work; avoid unnecessary type annotations.
- Use `never` in the default case of exhaustive switches.

### Performance Considerations

Control flow analysis is a compile-time operation with no runtime cost. However, complex control flow with many branches can increase type-checking time. User-defined type guards add runtime overhead (the guard function executes).

### Interview Questions

1. How does TypeScript narrow types in conditional branches?
2. What is a user-defined type guard and how do you create one?
3. How does TypeScript track type changes across reassignments?
4. What is the `never` type and how is it used in exhaustive checking?

### Coding Challenges

1. Create a discriminated union for a video player state and write a function that uses CFA to handle each state.
2. Write a custom type guard that checks if a value is a valid `Date` object.
3. Implement an assertion function that validates an API response shape.

### Related Topics

- Discriminated unions
- Type guards
- Assertion functions
- Exhaustiveness checking
