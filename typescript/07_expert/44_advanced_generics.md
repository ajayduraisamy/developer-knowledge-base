# Advanced Generics - Template literal types with generics, variadic tuple types, recursive conditional types, branded types

## Introduction

Advanced generics in TypeScript enable type-level programming that goes beyond simple parameterized types. Template literal types with generics allow dynamic string type construction. Variadic tuple types enable type-safe operations on tuples of arbitrary length. Recursive conditional types unlock recursive type transformations. Branded types provide nominal-type-like behavior in TypeScript's structural type system. This file explores these advanced patterns with production-grade examples.

## Template Literal Types with Generics

### What It Is

Template literal types allow constructing string types using template literal syntax, where placeholders can be other types (including string literal unions). When combined with generics, they create powerful type transformations over string patterns.

### Why It Is Important

Template literal types enable type-safe event systems, CSS property autocompletion, API route typing, and many other string-based type patterns. They eliminate entire categories of runtime errors by catching invalid string compositions at compile time.

### How It Works Internally

TypeScript expands template literal types by iterating over each placeholder and producing all possible combinations using the `|` (union) type. For example, `` `${'get' | 'set'}${'Name' | 'Age'}` `` produces `'getName' | 'getAge' | 'setAge' | 'setName'`.

### Syntax

```typescript
type EventName<T extends string> = `${T}Changed`;
type NameEvent = EventName<'name'>; // 'nameChanged'

type PropEvent<T extends string, V> = {
  readonly [K in `${T}Changed`]: (value: V) => void;
}[`${T}Changed`];
```

### Beginner Examples

```typescript
type VerticalAlignment = 'top' | 'middle' | 'bottom';
type HorizontalAlignment = 'left' | 'center' | 'right';

type Alignment = `${VerticalAlignment}-${HorizontalAlignment}`;
// 'top-left' | 'top-center' | 'top-right' | 'middle-left' | ...

type CssClass = `${'btn' | 'card' | 'input'}--${'primary' | 'secondary' | 'danger'}`;
// 'btn--primary' | 'btn--secondary' | 'btn--danger' | 'card--primary' | ...

function applyStyle(className: CssClass): void {
  document.querySelector(`.${className}`);
}
```

### Intermediate Examples

```typescript
// Event emitter with typed event names
type EventMap = {
  userLoggedIn: { userId: string };
  userLoggedOut: {};
  errorOccurred: { message: string; code: number };
};

type EventName = keyof EventMap;

type EventHandler<K extends EventName> = (payload: EventMap[K]) => void;

function on<K extends EventName>(event: K, handler: EventHandler<K>): void {
  // Implementation
}

on('userLoggedIn', (payload) => {
  console.log(payload.userId); // typed as string
});

// CSS property builder
type CSSProperty = `${'margin' | 'padding'}${'' | 'Top' | 'Right' | 'Bottom' | 'Left'}`;
type CSSValue = string | number;

const styles: Record<CSSProperty, CSSValue> = {
  margin: '10px',
  paddingTop: '5px',
  marginLeft: 0,
};

// API route builder
type ApiVersion = 'v1' | 'v2';
type Resource = 'users' | 'posts' | 'comments';
type ApiRoute<T extends ApiVersion, R extends Resource> = `/api/${T}/${R}`;

type V1UsersRoute = ApiRoute<'v1', 'users'>; // '/api/v1/users'
```

### Advanced Examples

```typescript
// Type-safe object path getter
type PathImpl<T, K extends keyof T> =
  K extends string
    ? T[K] extends Record<string, any>
      ? `${K}.${PathImpl<T[K], keyof T[K]>}` | K
      : K
    : never;

type Path<T> = PathImpl<T, keyof T> extends infer P
  ? P extends string
    ? P
    : never
  : never;

type GetValue<T, P extends Path<T>> =
  P extends `${infer K}.${infer Rest}`
    ? K extends keyof T
      ? Rest extends Path<T[K]>
        ? GetValue<T[K], Rest>
        : never
      : never
    : P extends keyof T
      ? T[P]
      : never;

function get<T, P extends Path<T>>(obj: T, path: P): GetValue<T, P> {
  const keys = (path as string).split('.');
  let result: any = obj;
  for (const key of keys) {
    result = result?.[key];
  }
  return result;
}

const obj = {
  user: { name: 'Alice', address: { city: 'NYC', zip: '10001' } },
  meta: { count: 5 },
};

const city = get(obj, 'user.address.city'); // type: string
const count = get(obj, 'meta.count'); // type: number
// get(obj, 'invalid.path'); // Error

// Fluent API with method chaining
type QueryMethods = 'where' | 'orderBy' | 'limit' | 'offset';
type QueryMethod<S extends string> = S extends `${infer M}By${infer F}`
  ? M extends 'order' ? { field: F; direction: 'asc' | 'desc' } : { field: F }
  : Record<string, never>;

function query<T extends Record<string, any>>() {
  return {
    where<K extends keyof T & string>(field: K, value: T[K]) { return this; },
    orderBy<K extends keyof T & string>(field: K, direction: 'asc' | 'desc') { return this; },
    limit(n: number) { return this; },
  };
}

// Usage
query<{ id: number; name: string; age: number }>()
  .where('name', 'Alice')
  .orderBy('age', 'asc')
  .limit(10);
```

### Real-World Use Cases

```typescript
// Redux action type generation
type ActionType<Feature extends string, Action extends string> = `${Feature}/${Action}`;

type UserActions = ActionType<'users', 'fetch' | 'create' | 'update' | 'delete'>;
// 'users/fetch' | 'users/create' | 'users/update' | 'users/delete'

// Event sourcing system
type DomainEvent<Aggregate extends string, Event extends string, Data> = {
  type: `${Aggregate}${Event}`;
  aggregateId: string;
  data: Data;
  timestamp: Date;
};

type OrderCreatedEvent = DomainEvent<'Order', 'Created', { items: string[]; total: number }>;
// { type: 'OrderCreated'; aggregateId: string; data: { items: string[]; total: number }; timestamp: Date }

// CSS class builder
type BEM<Block extends string, Element extends string, Modifier extends string> =
  `${Block}__${Element}--${Modifier}`;

type ButtonModifier = BEM<'button', 'icon', 'large' | 'small'>;
// 'button__icon--large' | 'button__icon--small'
```

### Common Mistakes

```typescript
// MISTAKE: Using template literal types with non-string types
// type Bad = `${42}`; // Error: Type '42' is not assignable to type 'string | number | bigint | boolean | null | undefined'
// CORRECT: type Good = `${42}`; // Actually works - number is converted to string

// MISTAKE: Expecting template literal types to be computed at runtime
// Template literal types are compile-time only

// MISTAKE: Creating overly complex template literal types that are hard to debug
```

### Best Practices

- Use template literal types for type-safe string manipulation at compile time.
- Combine with `infer` for pattern matching on string types.
- Use with generics to create reusable string type transformers.
- Prefer simple unions over complex template literals when possible.
- Test template literal types with simple examples to verify behavior.

### Performance Considerations

Template literal types are resolved at compile time. Very large unions generated from template literals (e.g., combining multiple large unions) can slow down the type checker. Keep template literal combinations manageable.

### Interview Questions

1. How do template literal types with generics work in TypeScript?
2. What is the difference between a template literal type and a regular template literal?
3. How do you extract parts of a string type using `infer` and template literals?

### Coding Challenges

1. Create a type that transforms `camelCase` to `snake_case` using template literal types.
2. Build a type-safe router where URL parameters are inferred from path patterns like `/users/:id/posts/:postId`.

### Related Topics

- Conditional types
- infer keyword
- String literal types

## Variadic Tuple Types

### What It Is

Variadic tuple types allow tuples to have a variable number of elements with varying types. They use rest elements with generic type parameters to capture and transform tuple types of arbitrary length and composition.

### Why It Is Important

Variadic tuple types enable type-safe operations on function parameters, array concatenation, Promise chain typing, and many patterns that require working with sequences of types of unknown length.

### How It Works Internally

Variadic tuple types use `...T extends any[]` to capture the entire tuple. TypeScript can infer, split, and reconstitute tuples using type inference on rest parameters. Key operators include `readonly`, `as const`, and array spread in type positions.

### Syntax

```typescript
// Basic variadic tuple
type Tuple = [string, ...number[], boolean];
// Requires string first, then any number of numbers, then boolean

// Generic variadic tuple
type Push<T extends any[], V> = [...T, V];
type Result = Push<[string, number], boolean>; // [string, number, boolean]

// Inferring tuple from rest parameters
function concat<T extends any[], U extends any[]>(arr1: [...T], arr2: [...U]): [...T, ...U] {
  return [...arr1, ...arr2];
}
```

### Beginner Examples

```typescript
// Concatenating tuples
function concat<T extends any[], U extends any[]>(a: [...T], b: [...U]): [...T, ...U] {
  return [...a, ...b];
}

const result = concat([1, 2] as const, ['a', 'b'] as const);
// type: [1, 2, 'a', 'b']

// Tuple with rest elements
type Mixed = [string, ...number[], boolean];
const valid: Mixed = ['hello', 1, 2, 3, true];

// Prepending to a tuple
type Prepend<T extends any[], V> = [V, ...T];
type PrependResult = Prepend<[number, boolean], string>; // [string, number, boolean]
```

### Intermediate Examples

```typescript
// Typed curry function
type Curried<A extends any[], R> =
  A extends [infer F, ...infer Rest]
    ? (arg: F) => Rest extends []
      ? R
      : Curried<Rest, R>
    : R;

function curry<A extends any[], R>(fn: (...args: A) => R): Curried<A, R> {
  return ((...args: any[]) => {
    if (args.length >= fn.length) {
      return fn(...args as any);
    }
    return curry((...rest: any[]) => fn(...args, ...rest as any)) as any;
  }) as any;
}

const add = (a: number, b: number, c: number) => a + b + c;
const curriedAdd = curry(add);
const result2 = curriedAdd(1)(2)(3); // 6, fully typed

// Type-safe Promise.all
type UnwrapPromises<T extends any[]> = {
  [K in keyof T]: T[K] extends Promise<infer U> ? U : T[K];
};

function promiseAll<T extends any[]>(promises: [...T]): Promise<UnwrapPromises<T>> {
  return Promise.all(promises) as any;
}

const promises = promiseAll([
  Promise.resolve(42),
  Promise.resolve('hello'),
  Promise.resolve(true),
]);
// type: Promise<[number, string, boolean]>
```

### Advanced Examples

```typescript
// Type-safe function composition
type Compose = {
  <A>(...fns: [(arg: A) => A]): (arg: A) => A;
  <A, B>(...fns: [(arg: A) => B]): (arg: A) => B;
  <A, B, C>(...fns: [(arg: A) => B, (arg: B) => C]): (arg: A) => C;
  <A, B, C, D>(...fns: [(arg: A) => B, (arg: B) => C, (arg: C) => D]): (arg: A) => D;
};

function compose<T>(...fns: Function[]): Function {
  return (arg: T) => fns.reduceRight((acc, fn) => fn(acc), arg);
}

// Pipe with variadic tuple types
type Last<T extends any[]> = T extends [...any[], infer L] ? L : never;
type First<T extends any[]> = T extends [infer F, ...any[]] ? F : never;

function pipe<Fns extends [(...args: any[]) => any, ...((arg: any) => any)[]]>(
  ...fns: Fns
): (...args: Parameters<First<Fns>>) => ReturnType<Last<Fns>> {
  return (...args: any[]) => fns.reduce((acc, fn) => fn(acc), ...args);
}

const add1 = (x: number) => x + 1;
const toString = (x: number) => String(x);
const appendExclaim = (x: string) => x + '!';

const piped = pipe(add1, toString, appendExclaim);
const result3 = piped(5); // type: string, value: "6!"

// Zip function with tuple types
function zip<T extends any[][], R extends any[]>(
  ...arrays: T
): { [K in keyof T]: T[K] extends (infer U)[] ? U : never }[][] {
  const maxLength = Math.max(...arrays.map(arr => arr.length));
  return Array.from({ length: maxLength }, (_, i) =>
    arrays.map(arr => arr[i])
  ) as any;
}

const zipped = zip(
  [1, 2, 3] as const,
  ['a', 'b', 'c'] as const,
  [true, false, true] as const,
);
// type: [1, 'a', true][] | [2, 'b', false][] | [3, 'c', true][]

// Tuple filter
type FilterString<T extends any[]> = T extends [infer F, ...infer R]
  ? F extends string
    ? [F, ...FilterString<R>]
    : FilterString<R>
  : [];

type Filtered = FilterString<[1, 'hello', true, 'world', 42]>;
// ['hello', 'world']
```

### Real-World Use Cases

```typescript
// Redux middleware typing
type MiddlewareAPI<S, A> = { getState: () => S; dispatch: (action: A) => void };
type Middleware<S, A> = (api: MiddlewareAPI<S, A>) => (next: (action: A) => void) => (action: A) => void;

// Express middleware chain
type MiddlewareChain<T extends any[], R> = T extends [infer F, ...infer Rest]
  ? F extends (...args: any[]) => any
    ? (arg: ReturnType<F>) => MiddlewareChain<Rest, R>
    : never
  : R;

// React hook-like composition
function useChain<T extends (() => any)[]>(...hooks: T): { [K in keyof T]: ReturnType<T[K]> } {
  return hooks.map(hook => hook()) as any;
}

const [user, posts, notifications] = useChain(
  () => ({ id: 1, name: 'Alice' }),
  () => [{ id: 1, title: 'Post' }],
  () => 5,
);
// user: { id: number; name: string }
// posts: { id: number; title: string }[]
// notifications: number
```

### Common Mistakes

```typescript
// MISTAKE: Using variadic tuple types with mutable arrays
// const arr = [1, 2, 3]; // number[], not [number, number, number]
// CORRECT: const arr = [1, 2, 3] as const; // readonly [1, 2, 3]

// MISTAKE: Expecting TypeScript to infer literal tuple types without `as const`

// MISTAKE: Overcomplicating with variadic tuples when a simpler type suffices
```

### Best Practices

- Use `as const` to preserve tuple types when passing arrays to variadic functions.
- Combine variadic tuples with `infer` for type extraction.
- Use mapped types over tuple indices for element-wise transformations.
- Prefer simpler overloads when variadic tuples become too complex.
- Test type inference with concrete examples.

### Performance Considerations

Variadic tuple types are resolved at compile time. Deeply recursive type instantiation with large tuples can slow down the type checker. Keep tuples reasonably sized in type-level computations.

### Interview Questions

1. What are variadic tuple types and how do they differ from regular tuples?
2. How do you concatenate two tuples at the type level?
3. How does `as const` affect tuple type inference?

### Coding Challenges

1. Create a `zip` function that correctly types the zipped output.
2. Build a type-safe `compose` function using variadic tuple types.

### Related Topics

- Tuple types
- Rest parameters
- Conditional types

## Recursive Conditional Types

### What It Is

Recursive conditional types are conditional types that reference themselves. They enable type-level recursion for processing nested structures, tree transformations, and operations on types of arbitrary depth.

### Why It Is Important

Recursive conditional types unlock deep type transformations that would otherwise require massive union types or manual expansion. They are essential for type-safe deep partial, deep readonly, JSON types, and path-based type accessors.

### How It Works Internally

TypeScript supports recursion in conditional types up to a configurable depth (default 50). Each recursive step creates a new type instantiation. TypeScript ensures termination by tracking recursion depth.

### Syntax

```typescript
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends Record<string, any>
    ? T[K] extends Function
      ? T[K]
      : DeepReadonly<T[K]>
    : T[K];
};

type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends Record<string, any>
    ? T[K] extends Function
      ? T[K]
      : DeepPartial<T[K]>
    : T[K];
};
```

### Beginner Examples

```typescript
type JSONValue =
  | string
  | number
  | boolean
  | null
  | JSONValue[]
  | { [key: string]: JSONValue };

function parseJSON(data: string): JSONValue {
  return JSON.parse(data);
}

// Deep mutable/immutable
type DeepMutable<T> = {
  -readonly [K in keyof T]: T[K] extends ReadonlyArray<infer U>
    ? Array<DeepMutable<U>>
    : T[K] extends Record<string, any>
      ? DeepMutable<T[K]>
      : T[K];
};
```

### Intermediate Examples

```typescript
// Deep pick - pick properties at any depth
type DeepPick<T, Path extends string> =
  Path extends `${infer K}.${infer Rest}`
    ? K extends keyof T
      ? { [P in K]: DeepPick<T[K], Rest> }
      : never
    : Path extends keyof T
      ? { [P in Path]: T[Path] }
      : never;

interface DeepNested {
  user: {
    profile: {
      name: string;
      age: number;
    };
    settings: {
      theme: 'light' | 'dark';
    };
  };
}

type PickedName = DeepPick<DeepNested, 'user.profile.name'>;
// { user: { profile: { name: string } } }

// Deep omit
type DeepOmit<T, Path extends string> =
  Path extends `${infer K}.${infer Rest}`
    ? K extends keyof T
      ? { [P in keyof T]: P extends K ? DeepOmit<T[P], Rest> : T[P] }
      : T
    : Path extends keyof T
      ? Omit<T, Path>
      : T;

// Type-level JSON stringify
type JSONStringify<T> =
  T extends string ? `"${T}"` :
  T extends number | boolean | null ? `${T}` :
  T extends (infer U)[] ? `[${{ [K in keyof T]: JSONStringify<T[K]> }[number]}]` :
  T extends Record<string, any> ? `{${ { [K in keyof T & string]: `"${K}":${JSONStringify<T[K]>}` }[keyof T & string] }}` :
  never;
```

### Advanced Examples

```typescript
// DeepNonNullable - recursively remove null/undefined
type DeepNonNullable<T> =
  T extends (infer U)[]
    ? DeepNonNullable<U>[]
    : T extends Record<string, any>
      ? { [K in keyof T]-?: DeepNonNullable<NonNullable<T[K]>> }
      : NonNullable<T>;

type NullableNested = {
  a: string | null;
  b: { c: number | undefined; d: { e: boolean | null }[] | null };
};

type Cleaned = DeepNonNullable<NullableNested>;
// { a: string; b: { c: number; d: { e: boolean }[] } }

// Recursive type for merging
type MergeDeep<T, U> = T extends Record<string, any>
  ? U extends Record<string, any>
    ? {
        [K in keyof T | keyof U]: K extends keyof T
          ? K extends keyof U
            ? MergeDeep<T[K], U[K]>
            : T[K]
          : K extends keyof U
            ? U[K]
            : never;
      }
    : U
  : U;

type A = { a: number; shared: { x: string } };
type B = { b: string; shared: { y: number } };
type Merged = MergeDeep<A, B>;
// { a: number; b: string; shared: { x: string; y: number } }

// Type-level flatten
type Flatten<T> = T extends any[] ? T[number] extends infer U
  ? U extends Record<string, any>
    ? { [K in keyof U]: U[K] }
    : U
  : never : T;

type NestedArray = [{ id: number }, { name: string }];
type Flattened = Flatten<NestedArray>;
// { id: number } | { name: string }
```

### Real-World Use Cases

```typescript
// Immutable state tree
type DeepFreeze<T> = {
  readonly [K in keyof T]: T[K] extends Record<string, any>
    ? T[K] extends Function
      ? T[K]
      : DeepFreeze<T[K]>
    : T[K];
};

function freeze<T>(obj: T): DeepFreeze<T> {
  return Object.freeze(obj) as any;
}

const state = freeze({
  user: { name: 'Alice', address: { city: 'NYC' } },
  items: [{ id: 1, name: 'Item' }],
});

// state.user.name = 'Bob'; // Error: readonly
// state.user.address.city = 'LA'; // Error: readonly

// Recursive validation type
type ValidationErrors<T> = {
  [K in keyof T]?: T[K] extends Record<string, any>
    ? T[K] extends (infer U)[]
      ? ValidationErrors<U>[]
      : ValidationErrors<T[K]>
    : string[];
};

interface UserForm {
  name: string;
  email: string;
  address: {
    street: string;
    city: string;
  };
  tags: string[];
}

type FormErrors = ValidationErrors<UserForm>;
// {
//   name?: string[];
//   email?: string[];
//   address?: {
//     street?: string[];
//     city?: string[];
//   };
//   tags?: string[][];
// }
```

### Common Mistakes

```typescript
// MISTAKE: Creating infinite recursive types
// type Infinite<T> = T extends number ? Infinite<T> : never; // Error: Type instantiation is excessively deep

// MISTAKE: Not handling the base case in recursive conditional types
// Every recursion must have a termination condition

// MISTAKE: Forgetting to handle primitives and functions
// typeof 'hello' extends Record<string, any> -> true! So check for primitives first
```

### Best Practices

- Always include a base case for non-object types.
- Check for `Function` types separately, as they extend `Record<string, any>`.
- Limit recursion depth by handling nested structures at known depths.
- Use `infer` to extract and recurse on element types of arrays.
- Test recursive types with small examples to verify termination.

### Performance Considerations

Recursive conditional types have a default recursion limit of 50. Deeply nested types can significantly slow down type checking. For very deep structures, consider flattening or using simpler types.

### Interview Questions

1. How do recursive conditional types work in TypeScript?
2. What is the base case in a recursive conditional type and why is it important?
3. How does TypeScript prevent infinite recursion in types?

### Coding Challenges

1. Create a `DeepRequired<T>` type that makes all properties (nested too) required.
2. Build a `DeepPick<T, Path>` that picks a property by dot-separated path.

### Related Topics

- Conditional types
- Template literal types
- Mapped types

## Branded/Nominal Types

### What It Is

Branded types (also called nominal types or opaque types) simulate nominal typing within TypeScript's structural type system. By adding a unique "brand" property to a type, values of structurally identical types become distinguishable.

### Why It Is Important

Branded types prevent mixing semantically different values that happen to have the same structure. For example, preventing passing a `UserId` (string) where a `ProductId` (string) is expected, even though both are strings.

### How It Works Internally

A branded type adds a phantom property that doesn't exist at runtime but forces TypeScript to treat two otherwise identical types as different. The brand is typically `__brand` or a Symbol key.

### Syntax

```typescript
type Brand<T, B extends string> = T & { __brand: B };

type UserId = Brand<string, 'UserId'>;
type ProductId = Brand<string, 'ProductId'>;

function getUser(id: UserId): void {}
function getProduct(id: ProductId): void {}

const userId = 'user-123' as UserId;
const productId = 'prod-456' as ProductId;

getUser(userId); // OK
// getUser(productId); // Error: ProductId not assignable to UserId
```

### Beginner Examples

```typescript
// Email and Phone are both strings, but not interchangeable
type Email = Brand<string, 'Email'>;
type PhoneNumber = Brand<string, 'Phone'>;

function sendEmail(to: Email, body: string): void {
  console.log(`Sending to ${to}`);
}

function sendSMS(to: PhoneNumber, message: string): void {
  console.log(`Texting ${to}`);
}

const email = 'alice@test.com' as Email;
const phone = '+1234567890' as PhoneNumber;

sendEmail(email, 'Hello'); // OK
// sendEmail(phone, 'Hello'); // Error: PhoneNumber not assignable to Email

// Numeric IDs
type OrderId = Brand<number, 'OrderId'>;
type CustomerId = Brand<number, 'CustomerId'>;

function findOrder(id: OrderId): void {}
function findCustomer(id: CustomerId): void {}
```

### Intermediate Examples

```typescript
// Branded types with validation
type UserId = Brand<string, 'UserId'>;

function createUserId(raw: string): UserId {
  if (!raw.match(/^user_[a-f0-9]{24}$/)) {
    throw new Error('Invalid user ID format');
  }
  return raw as UserId;
}

// Currency amounts
type USD = Brand<number, 'USD'>;
type EUR = Brand<number, 'EUR'>;
type JPY = Brand<number, 'JPY'>;

function addUSD(a: USD, b: USD): USD {
  return (a + b) as USD;
}

const price1 = 29.99 as USD;
const price2 = 19.99 as USD;
const total = addUSD(price1, price2); // OK: USD

const euroPrice = 25.00 as EUR;
// addUSD(price1, euroPrice); // Error: EUR not assignable to USD

// Branded types with interfaces
interface User {
  id: UserId;
  name: string;
  email: Email;
}

function createUser(name: string, email: Email): User {
  return { id: createUserId('user_' + generateHex()), name, email };
}
```

### Advanced Examples

```typescript
// Fluent branded type creation
class Branded<T extends string> {
  static of<T extends string>() {
    return {
      make: (value: string): Brand<string, T> => value as Brand<string, T>,
      validate: (value: string, validator: (v: string) => boolean): Brand<string, T> => {
        if (!validator(value)) throw new Error(`Invalid ${T}`);
        return value as Brand<string, T>;
      },
    };
  }
}

const UserId = Branded.of<'UserId'>();
const validUserId = UserId.validate('user_abc123', (v) => v.startsWith('user_'));

// Generic branded utilities
type BrandedType<T, B extends string> = T & { readonly _brand: B };

function brand<T, B extends string>(value: T, _brand: B): BrandedType<T, B> {
  return value as BrandedType<T, B>;
}

const userId = brand('123', 'UserId');
const orderId = brand('456', 'OrderId');

// Branded type with class
class SafeString<T extends string> {
  private readonly __brand!: T;
  constructor(public readonly value: string) {}
}

type SafeUserId = SafeString<'UserId'>;
type SafeEmail = SafeString<'Email'>;

function createSafeUserId(id: string): SafeUserId {
  return new SafeString(id) as SafeUserId;
}

// Branded types in discriminated unions
type AbsolutePath = Brand<string, 'AbsolutePath'>;
type RelativePath = Brand<string, 'RelativePath'>;

function isAbsolute(path: string): path is AbsolutePath {
  return path.startsWith('/');
}

function normalizePath(path: AbsolutePath): string {
  return path.replace(/\/+/g, '/');
}

// Phantom type pattern (no runtime overhead)
interface Phantom<T> {
  __phantom?: T;
}

type Metric<T extends string> = number & Phantom<T>;
type Kilometers = Metric<'km'>;
type Miles = Metric<'mi'>;

function toMiles(km: Kilometers): Miles {
  return (km * 0.621371) as Miles;
}
```

### Real-World Use Cases

```typescript
// Financial domain
type AccountId = Brand<string, 'AccountId'>;
type TransactionId = Brand<string, 'TransactionId'>;
type Money = Brand<number, 'Money'>;

interface Transaction {
  id: TransactionId;
  from: AccountId;
  to: AccountId;
  amount: Money;
  timestamp: Date;
}

function transfer(from: AccountId, to: AccountId, amount: Money): Transaction {
  if (from === to) throw new Error('Cannot transfer to same account');
  return { id: crypto.randomUUID() as TransactionId, from, to, amount, timestamp: new Date() };
}

// API response types
type Validated<T, V extends string> = T & { readonly __validated: V };

type SanitizedHtml = Validated<string, 'sanitized'>;
type EscapedSql = Validated<string, 'escaped'>;

function sanitize(input: string): SanitizedHtml {
  return input.replace(/</g, '&lt;').replace(/>/g, '&gt;') as SanitizedHtml;
}

function render(html: SanitizedHtml): void {
  document.body.innerHTML = html; // Safe: only accepts sanitized strings
}

// Unit conversions
type Meters = Brand<number, 'Meters'>;
type Seconds = Brand<number, 'Seconds'>;
type MetersPerSecond = Brand<number, 'MetersPerSecond'>;

function speed(distance: Meters, time: Seconds): MetersPerSecond {
  if (time <= 0) throw new Error('Time must be positive');
  return (distance / time) as MetersPerSecond;
}
```

### Common Mistakes

```typescript
// MISTAKE: Forgetting that branded types are compile-time only
// At runtime, a branded string is just a string
// const id = '123' as UserId;
// typeof id === 'string' // true

// MISTAKE: Using a brand property name that conflicts with real properties
// const obj = { __brand: 'real' };
// const branded = obj as Brand<string, 'MyBrand'>; // Conflict!

// MISTAKE: Not providing factory functions for creating branded types
// Relying on `as` casts everywhere defeats the purpose
```

### Best Practices

- Provide factory functions (e.g., `createUserId()`) instead of manual `as` casts.
- Use unique brand names across the project to prevent accidental assignability.
- Consider using `interface` instead of `type` for more explicit branding.
- Validate input when creating branded types from external data.
- Use branded types at module boundaries (API, database, external interfaces).

### Performance Considerations

Branded types are completely erased at runtime with zero runtime cost. The brand property exists only at the type level. Factory functions add minimal overhead for validation.

### Interview Questions

1. What are branded types and why are they useful in TypeScript?
2. How do branded types differ from nominal types in other languages?
3. How do you create and use a branded type?

### Coding Challenges

1. Create a `Branded` utility type and factory functions for `Email`, `Phone`, and `SSN`.
2. Build a type-safe currency converter that prevents mixing different currencies.

### Related Topics

- Structural vs nominal typing
- Intersection types
- Opaque types
