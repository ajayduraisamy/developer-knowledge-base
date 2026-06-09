# Conditional Types - T extends U ? X : Y, infer keyword, distributive conditional types, real-world patterns

## Introduction

Conditional types are one of TypeScript's most powerful advanced type features. They allow you to create types that depend on a condition, expressed as `T extends U ? X : Y`. This is analogous to a ternary expression at the type level: if type `T` is assignable to type `U`, the result is `X`; otherwise, it's `Y`.

Conditional types enable dynamic type transformations, type-level programming, and the creation of utility types that adapt to their inputs. Combined with the `infer` keyword, they can decompose and analyze types, extracting inner types from complex structures. Understanding conditional types is essential for mastering TypeScript's type system and writing sophisticated type utilities.

## Conditional type syntax

### What It Is

A conditional type has the form `T extends U ? X : Y`. TypeScript evaluates whether `T` is assignable to `U`. If true, the type resolves to `X`; otherwise, to `Y`.

```typescript
type IsString<T> = T extends string ? true : false;
type A = IsString<"hello">; // true
type B = IsString<42>; // false
```

### Why It Is Important

Conditional types enable type-level decision making. They are the foundation for all built-in utility types, type filtering, and type transformations. Without them, TypeScript's type system would be limited to structural relationships and could not conditionally change types based on their shape.

### How It Works Internally

The compiler checks assignability of `T` to `U`. If the check is positive, the true branch is used; otherwise, the false branch. When `T` is a generic type parameter, the evaluation is deferred until the type parameter is resolved. When `T` is a union, the conditional type *distributes* over each member (see distributive conditional types).

```typescript
// Deferred evaluation:
// type IsString<T> = T extends string ? true : false;
// At call site: IsString<number> → false
// At call site: IsString<"hello"> → true
```

### Syntax

```typescript
// Basic conditional
type Truthy<T> = T extends false | 0 | "" | null | undefined ? false : true;

// Nested conditional
type TypeName<T> =
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  T extends undefined ? "undefined" :
  T extends Function ? "function" :
  "object";

// Conditional with generic
type IsArray<T> = T extends readonly unknown[] ? true : false;

// Inline conditional
type Result = 42 extends number ? "yes" : "no"; // "yes"

// Conditional with never
type IfNever<T> = [T] extends [never] ? true : false;
```

### Beginner Examples

```typescript
// Simple type check
type IsNumber<T> = T extends number ? true : false;

type Check1 = IsNumber<42>; // true
type Check2 = IsNumber<"hello">; // false
type Check3 = IsNumber<number>; // true

// Type name extraction
function describeType<T>(value: T): TypeName<T> {
  if (typeof value === "string") return "string" as TypeName<T>;
  if (typeof value === "number") return "number" as TypeName<T>;
  if (typeof value === "boolean") return "boolean" as TypeName<T>;
  return "object" as TypeName<T>;
}

// Conditional return type
type Result<T> = T extends string
  ? { success: true; value: T }
  : { success: false; error: string };

function process<T extends string | number>(input: T): Result<T> {
  if (typeof input === "string") {
    return { success: true, value: input } as Result<T>;
  }
  return { success: false, error: "Expected string" } as Result<T>;
}

// Conditional for filtering
type Nullable<T> = T | null | undefined;
type RemoveNull<T> = T extends null | undefined ? never : T;

type Clean = RemoveNull<string | null>; // string
```

### Intermediate Examples

```typescript
// Conditional with indexed access
type ArrayElement<T> = T extends (infer U)[] ? U : never;
type Element = ArrayElement<string[]>; // string

// Conditional for function types
type ReturnOf<T> = T extends (...args: any[]) => infer R ? R : never;
type ParamOf<T> = T extends (arg: infer P) => any ? P : never;

type Fn = (x: string) => number;
type R = ReturnOf<Fn>; // number
type P = ParamOf<Fn>; // string

// Conditional for promise unwrapping
type Unwrap<T> = T extends Promise<infer U> ? U : T;
type Wrapped = Unwrap<Promise<string>>; // string
type NotWrapped = Unwrap<number>; // number

// Conditional with multiple branches
type PrimitiveOrComplex<T> =
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  "complex";

// Conditional for nullish checks
type IsNullable<T> = T extends null | undefined ? true : false;
type IsStringNullable = IsNullable<string | null>; // true? No! This distributes
```

### Advanced Examples

```typescript
// Recursive conditional type
type DeepReadonly<T> = T extends object
  ? { readonly [P in keyof T]: DeepReadonly<T[P]> }
  : T;

// Conditional with variadic tuples
type TupleSplit<T extends any[], N extends number, O extends any[] = []> =
  O["length"] extends N
    ? [O, T]
    : T extends [infer F, ...infer R]
      ? TupleSplit<R, N, [...O, F]>
      : [O, T];

type Split = TupleSplit<[1, 2, 3, 4], 2>;
// [[1, 2], [3, 4]]

// Conditional for branded types
type Brand<T, B extends string> = T & { __brand: B };
type IsBranded<T> = T extends { __brand: string } ? true : false;

// Conditional with string checks
type IsLiteralString<T> = T extends string
  ? string extends T
    ? false
    : true
  : false;

type Test1 = IsLiteralString<"hello">; // true
type Test2 = IsLiteralString<string>; // false

// Nested conditionals for complex logic
type CompareTypes<T, U> =
  T extends U
    ? U extends T
      ? "equal"
      : "T extends U"
    : U extends T
      ? "U extends T"
      : "incomparable";

type C1 = CompareTypes<string, string>; // "equal"
type C2 = CompareTypes<"hello", string>; // "T extends U"
type C3 = CompareTypes<string, number>; // "incomparable"
```

### Real-World Use Cases

- **API response types**: Conditional success/error response based on HTTP status.
- **Component props**: Conditional prop types based on a variant/type prop.
- **State machines**: Conditional state transition types.
- **Validation**: Conditional return types based on validation result.
- **Serialization**: Different serialization logic based on input type.
- **Plugin systems**: Conditional configuration types based on plugin type.

```typescript
// Real-world: Conditional API response
type APIResponse<T, E = Error> =
  T extends void
    ? { success: true }
    : { success: true; data: T }
  | { success: false; error: E };

async function callAPI<T>(url: string): Promise<APIResponse<T>> {
  try {
    const response = await fetch(url);
    const data = await response.json();
    return { success: true, data: data as T };
  } catch (e) {
    return { success: false, error: e as Error };
  }
}
```

### Common Mistakes

1. **Not understanding distributivity**: When `T` is a union, the conditional applies to each member.
2. **Using `never` incorrectly**: `never` in a union disappears, which affects distributivity.
3. **Creating infinite recursion**: Recursive conditional types must have a base case.
4. **Forgetting that conditionals are evaluated lazily**: Generic type parameters defer evaluation.
5. **Overcomplicating**: Sometimes a simpler mapped type or overload suffices.

### Best Practices

- Always have a base case in recursive conditional types.
- Use `[T] extends [U]` to prevent distribution over unions when needed.
- Name conditional types clearly to indicate what they test.
- Combine conditionals with `infer` for type extraction.
- Keep conditional types focused—one transformation per type.
- Use conditional types at type aliases, not inline in complex expressions.

### Performance Considerations

Conditional types can be expensive for the compiler, especially deeply nested ones or those that distribute over large unions. Recursive conditional types have a recursion limit (typically 50 levels). For production code, keep conditional types shallow and avoid distributing over unions with thousands of members.

### Interview Questions

1. **Q**: What is a conditional type in TypeScript?
   **A**: A type that takes the form `T extends U ? X : Y`. It evaluates to `X` if `T` is assignable to `U`, otherwise `Y`.

2. **Q**: How do you prevent a conditional type from distributing over a union?
   **A**: Wrap both sides in square brackets: `[T] extends [U] ? X : Y`. This treats `T` as a single unit.

### Coding Challenges

1. Implement a `DeepPartial` using conditional types.
2. Create a conditional type that extracts the resolved value from a nested Promise.
3. Write a conditional type that checks if two types are exactly equal.

### Related Topics

- `infer` keyword
- Distributive conditional types
- Mapped types
- Recursive types

## infer keyword

### What It Is

The `infer` keyword is used within conditional types to declare a type variable that captures a type from within a larger type. It appears within the `extends` clause of a conditional type and allows you to extract and use sub-types.

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type A = ReturnType<() => string>; // string
```

### Why It Is Important

`infer` unlocks the ability to decompose types—extracting return types, parameter types, array element types, and more. It enables the implementation of utility types like `ReturnType`, `Parameters`, `ConstructorParameters`, and `InstanceType` directly in the type system.

### How It Works Internally

When the compiler evaluates `T extends SomeType<infer X>`, it matches `T` against `SomeType` and infers the type argument for `X` from the structure of `T`. This inference follows the same rules as generic inference in functions. `infer` can only appear within a conditional type's `extends` clause.

```typescript
// T = () => string
// (...args: any[]) => infer R matches () => string
// R is inferred as string
```

### Syntax

```typescript
// Infer return type
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

// Infer parameter type
type MyParameter<T> = T extends (arg: infer P) => any ? P : never;

// Infer array element type
type ElementType<T> = T extends (infer U)[] ? U : never;

// Infer promise value
type PromiseValue<T> = T extends Promise<infer U> ? U : never;

// Infer with multiple positions
type FirstArg<T> = T extends (first: infer F, ...rest: any[]) => any ? F : never;

// Infer constructor type
type InstanceType<T> = T extends new (...args: any[]) => infer R ? R : never;
```

### Beginner Examples

```typescript
// Extract element type from array
type ItemType<T extends readonly unknown[]> = T extends readonly (infer U)[] ? U : never;

type Numbers = ItemType<number[]>; // number
type Strings = ItemType<readonly string[]>; // string
type Mixed = ItemType<[string, number]>; // string | number

// Extract return type from function
function greet(name: string): string {
  return `Hello, ${name}`;
}

type GreetReturn = ReturnType<typeof greet>; // string

// Extract promise result
async function fetchUser(): Promise<{ id: number; name: string }> {
  return { id: 1, name: "Alice" };
}

type UserData = PromiseValue<ReturnType<typeof fetchUser>>;
// { id: number; name: string }

// Extract first parameter
type FirstParam<T> = T extends (first: infer F, ...rest: any[]) => any ? F : never;

type Fn = (name: string, age: number) => void;
type Name = FirstParam<Fn>; // string
```

### Intermediate Examples

```typescript
// Infer from method signatures
interface User {
  getName(): string;
  setName(name: string): void;
}

type MethodReturn<T, M extends keyof T> = T[M] extends (...args: any[]) => infer R ? R : never;
type GetNameReturn = MethodReturn<User, "getName">; // string

// Infer from complex generics
interface Box<T> { value: T; get(): T; set(val: T): void; }

type BoxValue<T> = T extends Box<infer U> ? U : never;
type StringBoxValue = BoxValue<Box<string>>; // string

// Infer in recursive types
type Flatten<T> = T extends Array<infer U> ? Flatten<U> : T;
type Flat = Flatten<[[[number]]]>; // number

// Infer with template literals
type ExtractName<T extends string> = T extends `${infer Name}.${string}` ? Name : never;
type FileName = ExtractName<"report.pdf">; // "report"

// Infer from tuple
type Tail<T extends any[]> = T extends [infer F, ...infer R] ? R : never;
type T1 = Tail<[1, 2, 3]>; // [2, 3]
type T2 = Tail<[1]>; // []
```

### Advanced Examples

```typescript
// Infer from deeply nested types
type DeepPromiseValue<T> = T extends Promise<infer U> ? DeepPromiseValue<U> : T;
type Deep = DeepPromiseValue<Promise<Promise<string>>>; // string

// Infer from function with multiple parameters
type Parameters<T> = T extends (...args: infer P) => any ? P : never;
type FnParams = Parameters<(a: string, b: number) => void>; // [string, number]

// Infer with variance
type PropType<T, K extends keyof T> = T[K] extends infer R ? R : never;
type NameType = PropType<{ name: string }, "name">; // string

// Infer from conditional types
type ExtractArray<T> = T extends { [K: string]: infer U } ? U : never;
type ObjValues = ExtractArray<{ a: 1; b: 2 }>; // 1 | 2

// Infer from overloaded functions (last signature wins)
type Overloaded = {
  (x: string): number;
  (x: number): string;
};
type OverloadReturn = ReturnType<Overloaded>; // number (last overload)

// Infer with intersection
type IntersectInfer<T> = T extends { a: infer A } & { b: infer B } ? [A, B] : never;
type AB = IntersectInfer<{ a: string; b: number }>; // [string, number]

// String parsing with infer
type ParsePath<T extends string> =
  T extends `${infer Start}/${infer Rest}`
    ? [Start, ...ParsePath<Rest>]
    : [T];

type Path = ParsePath<"a/b/c">; // ["a", "b", "c"]
```

### Real-World Use Cases

- **Utility library development**: Implementing `ReturnType`, `Parameters`, `ConstructorParameters`.
- **API client generators**: Extracting response types from function signatures.
- **Form validation**: Inferring field types from validator functions.
- **Event systems**: Extracting payload types from event handler signatures.
- **Plugin systems**: Inferring configuration types from plugin definitions.
- **Database ORM**: Inferring entity types from query builders.

```typescript
// Real-world: Type-safe event emitter
class TypedEmitter<T extends Record<string, (...args: any[]) => void>> {
  on<K extends keyof T>(event: K, handler: T[K]): void {}
  emit<K extends keyof T>(event: K, ...args: Parameters<T[K]>): void {}
}

type Events = {
  data: (payload: { id: number }) => void;
  error: (err: Error) => void;
};

const emitter = new TypedEmitter<Events>();
emitter.on("data", (payload) => console.log(payload.id));
emitter.emit("data", { id: 1 });
```

### Common Mistakes

1. **Using `infer` outside a conditional type**: It's only valid in the `extends` clause.
2. **Multiple `infer` declarations**: You can have multiple `infer` variables, but they must be unambiguous.
3. **Confusing `infer` with generic parameters**: `infer` is used for extraction, generics for parameterization.
4. **Not handling the case where inference fails**: Always provide a `never` fallback.
5. **Inferring from non-specific types**: The pattern must be specific enough for the compiler to match.

### Best Practices

- Always provide a fallback (`never` or a sensible default) when inference fails.
- Use `infer` for extracting types from generics, functions, and complex structures.
- Combine `infer` with recursive conditional types for deeply nested extraction.
- Name your inferred type variables descriptively.
- Prefer built-in utility types when they meet your needs.

### Performance Considerations

`infer` adds to the compiler's inference workload. Deeply recursive inference (like `DeepPromiseValue`) can hit recursion limits. For most practical uses, `infer` is efficient.

### Interview Questions

1. **Q**: What is the `infer` keyword used for?
   **A**: It declares a type variable within a conditional type's `extends` clause to capture and use a sub-type from a larger type.

2. **Q**: Where can `infer` appear?
   **A**: Only in the `extends` clause of a conditional type.

3. **Q**: How would you implement `ReturnType<T>` using `infer`?
   **A**: `type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;`

### Coding Challenges

1. Implement a `ConstructorParameters` utility type using `infer`.
2. Write a type that extracts the resolved type from a deeply nested `Promise<Promise<...>>`.
3. Create a type that infers the key-value types from a `Map<K, V>`.

### Related Topics

- Conditional types
- Distributive conditional types
- ReturnType, Parameters utility types
- Template literal type inference

## Distributive conditional types

### What It Is

Distributive conditional types are conditional types that automatically expand over union types. When a conditional type is applied to a union type `T`, the condition is evaluated for each member of the union individually, and the results are combined into a new union.

```typescript
type ToArray<T> = T extends unknown ? T[] : never;
type Result = ToArray<string | number>;
// string[] | number[] (NOT (string | number)[])
```

### Why It Is Important

Distributivity is what makes conditional types useful for filtering and transforming unions. Without it, a conditional type applied to `string | number` would check if the entire union extends the condition, which it usually doesn't. Distributivity enables union filtering (`Exclude`, `Extract`) and per-member transformation.

### How It Works Internally

When `T` is a generic type parameter and is a union, the compiler splits the union into individual members, evaluates the conditional for each, and joins the results back into a union. Distribution only happens for *naked* type parameters—type parameters used directly in the `extends` clause, not wrapped in a tuple, array, or other type.

```typescript
// Distributive: T extends U ? X : Y
// Non-distributive: [T] extends [U] ? X : Y (prevents distribution)
// Non-distributive: T[] extends U[] ? X : Y (prevents distribution)
```

### Syntax

```typescript
// Distributive (naked T)
type Filter<T, U> = T extends U ? T : never;

// Non-distributive (wrapped T)
type NonDistributive<T, U> = [T] extends [U] ? true : false;

// Using distribution for union filtering
type OnlyString<T> = T extends string ? T : never;
type Strings = OnlyString<string | number | boolean>; // string

// Distribution with mapping
type Wrap<T> = T extends unknown ? { value: T } : never;
type Wrapped = Wrap<string | number>;
// { value: string } | { value: number }
```

### Beginner Examples

```typescript
// Filtering a union
type RemoveNumbers<T> = T extends number ? never : T;
type WithoutNumbers = RemoveNumbers<string | number | boolean>;
// string | boolean

// Extracting from a union
type ExtractStrings<T> = T extends string ? T : never;
type OnlyStrings = ExtractStrings<string | number | symbol>;
// string

// Mapping each member
type Nullify<T> = T extends unknown ? T | null : never;
type Nullable = Nullify<string | number>;
// string | null | number | null → string | number | null

// Check if any member matches
type ContainsString<T> = T extends string ? true : false;
type HasString = ContainsString<string | number>;
// true | false → boolean
```

### Intermediate Examples

```typescript
// Preventing distribution (when you need the whole union)
type IsUnion<T, U = T> = T extends U ? [U] extends [T] ? false : true : never;
type CheckUnion = IsUnion<string | number>; // true
type CheckNot = IsUnion<string>; // false

// Distribution with multiple type parameters
type FilterByType<T, U> = T extends U ? T : never;
type NumbersAndStrings = string | number;
type Filtered = FilterByType<NumbersAndStrings, number | boolean>;
// number (both number and boolean? number extends, boolean doesn't → number)

// Using distribution for discriminated union filtering
type Action =
  | { type: "FETCH_START" }
  | { type: "FETCH_SUCCESS"; data: string }
  | { type: "FETCH_ERROR"; error: Error };

type WithData<T> = T extends { data: any } ? T : never;
type FetchSuccess = WithData<Action>;
// { type: "FETCH_SUCCESS"; data: string }

// Distribution with mapped types
type KeysOfUnion<T> = T extends unknown ? keyof T : never;
type Keys = KeysOfUnion<{ a: number } | { b: string }>;
// "a" | "b"
```

### Advanced Examples

```typescript
// Union to intersection (using distributivity + infer)
type UnionToIntersection<U> =
  (U extends unknown ? (x: U) => void : never) extends
  (x: infer I) => void ? I : never;

type Intersected = UnionToIntersection<{ a: string } | { b: number }>;
// { a: string } & { b: number }

// Distribution with functional types
type FunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];

type MethodNames = FunctionPropertyNames<{ a(): void; b: number }>; // "a"

// Non-distributive pattern for exact type comparison
type ExactMatch<T, U> = [T] extends [U]
  ? [U] extends [T]
    ? true
    : false
  : false;

type E1 = ExactMatch<string, string>; // true
type E2 = ExactMatch<string, "hello">; // false
type E3 = ExactMatch<{ a: string }, { a: string; b: number }>; // false

// Distribution with template literals
type EventName = `on${Capitalize<string>}`;
type DistributedEvents<T extends string> = T extends string ? `on${Capitalize<T>}` : never;
type Events = DistributedEvents<"click" | "focus" | "blur">;
// "onClick" | "onFocus" | "onBlur"

// Distribution for exhaustive matching
type Match<T, Cases extends Record<string, unknown>> = {
  [K in keyof Cases]: T extends K ? Cases[K] : never;
}[keyof Cases];

type Result = Match<"a", { a: 1; b: 2; c: 3 }>; // 1
```

### Real-World Use Cases

- **Union filtering**: Removing or extracting specific types from unions.
- **Action handling**: Extracting specific action types in Redux reducers.
- **Event filtering**: Filtering events by type in event-driven systems.
- **Type guards**: Creating type-level conditionals for union types.
- **API response handling**: Differentiating success and error response types.
- **Configuration merging**: Combining configuration types with distribution.

```typescript
// Real-world: Redux action extraction
type ActionMap = {
  ADD_TODO: { text: string };
  DELETE_TODO: { id: number };
  TOGGLE_TODO: { id: number };
};

type Action = {
  [K in keyof ActionMap]: { type: K } & ActionMap[K];
}[keyof ActionMap];

type ExtractAction<T extends Action["type"]> = Action extends { type: T }
  ? Action
  : never;

type AddTodoAction = ExtractAction<"ADD_TODO">;
// { type: "ADD_TODO"; text: string }
```

### Common Mistakes

1. **Not realizing distribution is happening**: When `T` is a generic `string | number`, the conditional applies to each member.
2. **Forgetting how to prevent distribution**: Wrap with `[T]` to treat it as a single type.
3. **Distributing over `never`**: `T extends U ? X : Y` when `T = never` results in `never` (the empty union).
4. **Assuming distribution happens with non-generic types**: Distribution only occurs for generic type parameters.
5. **Overlooking that `boolean` is `true | false`**: `boolean` distributes into `true` and `false`.

### Best Practices

- Use distribution intentionally for union filtering and transformation.
- Use `[T] extends [U]` when you need to check the entire union as one.
- Remember that `never` in distribution results in `never` (empty).
- Test distribution behavior with simple examples before using in complex utilities.
- Document whether a conditional type is meant to be distributive or not.

### Performance Considerations

Distributive conditional types can cause exponential type-checking time when applied to large unions. Each member is independently evaluated, and with deeply nested conditionals, this multiplies. Keep unions small when using distributive types in performance-sensitive code.

### Interview Questions

1. **Q**: What is a distributive conditional type?
   **A**: It's a conditional type that, when applied to a union type parameter, distributes over each member of the union, evaluating the condition separately for each and combining the results.

2. **Q**: How do you prevent distribution?
   **A**: Wrap the type parameter in square brackets: `[T] extends [U] ? X : Y`.

3. **Q**: What happens when a distributive conditional type is applied to `never`?
   **A**: `never` is an empty union. Distribution over an empty union yields `never`.

### Coding Challenges

1. Implement `Exclude<T, U>` and `Extract<T, U>` using distributive conditional types.
2. Write a type that takes a union of objects and returns the union of their keys.
3. Create a type that converts a union to its last member.

### Related Topics

- Conditional types
- `never` type
- Union types
- `Exclude` and `Extract`

## Real-world conditional patterns (ReturnType, Parameters)

### What It Is

Several built-in utility types are implemented using conditional types with `infer`. The most notable are `ReturnType<T>` (extracts the return type of a function), `Parameters<T>` (extracts the parameter types as a tuple), `ConstructorParameters<T>` (extracts constructor parameter types), and `InstanceType<T>` (extracts instance type from a constructor).

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type Parameters<T> = T extends (...args: infer P) => any ? P : never;
type ConstructorParameters<T> = T extends abstract new (...args: infer P) => any ? P : never;
type InstanceType<T> = T extends abstract new (...args: any[]) => infer R ? R : never;
```

### Why It Is Important

These utility types enable type-safe reflection on functions and constructors. They are essential for higher-order functions, decorators, dependency injection, and any pattern that wraps or transforms functions. They eliminate the need to manually specify types that the compiler already knows.

### How It Works Internally

These types use `infer` within conditional types to match function signatures and extract specific parts. `ReturnType` matches `(...args: any[]) => infer R` and captures `R`. `Parameters` captures the entire parameter tuple as `infer P`. The fallback is `never` for non-function types.

```typescript
// ReturnType<() => string> → infer R matches string → string
// Parameters<(a: number, b: string) => void> → infer P matches [number, string]
```

### Syntax

```typescript
ReturnType<T>       // Extract return type of function T
Parameters<T>       // Extract parameters tuple of function T
ConstructorParameters<T> // Extract constructor parameters
InstanceType<T>     // Extract instance type from constructor type
```

### Beginner Examples

```typescript
// ReturnType
function createUser(name: string): { id: number; name: string } {
  return { id: Date.now(), name };
}

type CreateUserReturn = ReturnType<typeof createUser>;
// { id: number; name: string }

// Parameters
type CreateUserParams = Parameters<typeof createUser>;
// [string]

function fetchData(url: string, options?: RequestInit): Promise<Response> {
  return fetch(url, options);
}

type FetchParams = Parameters<typeof fetchData>;
// [string, RequestInit | undefined]

// InstanceType
class Database {
  constructor(private url: string, private poolSize: number) {}
  connect(): void {}
}

type DBInstance = InstanceType<typeof Database>;
// Database

// ConstructorParameters
type DBConstructorParams = ConstructorParameters<typeof Database>;
// [string, number]
```

### Intermediate Examples

```typescript
// Using ReturnType for wrapped functions
function withLogging<T extends (...args: any[]) => any>(
  fn: T,
  ...args: Parameters<T>
): ReturnType<T> {
  console.log(`Calling ${fn.name} with`, args);
  return fn(...args);
}

const result = withLogging(createUser, "Alice");

// Using Parameters for type-safe event emission
function emit<T extends (...args: any[]) => any>(
  event: string,
  handler: T,
  ...args: Parameters<T>
): ReturnType<T> {
  return handler(...args);
}

// Extracting method types
class Calculator {
  add(a: number, b: number): number { return a + b; }
  subtract(a: number, b: number): number { return a - b; }
}

type CalcMethod = typeof Calculator.prototype.add;
type AddParams = Parameters<CalcMethod>; // [number, number]
type AddReturn = ReturnType<CalcMethod>; // number

// Combining ReturnType with Promise
async function fetchJSON<T>(url: string): Promise<T> {
  const res = await fetch(url);
  return res.json();
}

type FetchJSONReturn = ReturnType<typeof fetchJSON>;
// Promise<unknown> (generic, so inference is limited)
```

### Advanced Examples

```typescript
// Unwrapping return types through promises
type UnwrappedReturn<T extends (...args: any[]) => any> =
  ReturnType<T> extends Promise<infer U> ? U : ReturnType<T>;

async function loadUser(): Promise<{ name: string }> {
  return { name: "Alice" };
}

type UserData = UnwrappedReturn<typeof loadUser>;
// { name: string }

// Matching function overloads (last overload wins)
function transform(x: string): number;
function transform(x: number): string;
function transform(x: string | number): string | number {
  return x;
}

type TransformReturn = ReturnType<typeof transform>;
// string (last overload's return type)

// Parameters with this parameter
type MethodParams<T> = T extends (this: any, ...args: infer P) => any ? P : never;
type AddParams2 = MethodParams<Calculator["add"]>; // [number, number]

// Using ConstructorParameters for factory functions
function createInstance<T extends new (...args: any[]) => any>(
  ctor: T,
  ...args: ConstructorParameters<T>
): InstanceType<T> {
  return new ctor(...args);
}

const db = createInstance(Database, "localhost", 10);
// db: Database

// Generic constraints with ReturnType and Parameters
function memoize<T extends (...args: any[]) => any>(
  fn: T
): (...args: Parameters<T>) => ReturnType<T> {
  const cache = new Map<string, ReturnType<T>>();
  return (...args: Parameters<T>): ReturnType<T> => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key)!;
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

const memoizedAdd = memoize((a: number, b: number) => a + b);

// Chained type extraction
type Fn = (x: string) => Promise<number>;
type FnReturn = ReturnType<Fn>; // Promise<number>
type Unwrapped = FnReturn extends Promise<infer U> ? U : never; // number
```

### Real-World Use Cases

- **Higher-order functions**: Wrapping functions with logging, caching, error handling.
- **Dependency injection**: Resolving constructor dependencies.
- **Mocking/testing**: Creating type-safe mock functions.
- **Event systems**: Extracting event payload types from handler signatures.
- **API client generation**: Deriving types from API function signatures.
- **Form validation**: Inferring field types from validator functions.
- **Redux middleware**: Typing middleware with function parameters.

```typescript
// Real-world: Dependency injection container
class Container {
  private registry = new Map<string, new (...args: any[]) => any>();

  register<T extends new (...args: any[]) => any>(
    token: string,
    ctor: T
  ): void {
    this.registry.set(token, ctor);
  }

  resolve<T extends new (...args: any[]) => any>(
    token: string
  ): InstanceType<T> {
    const ctor = this.registry.get(token) as T;
    if (!ctor) throw new Error(`No registration for ${token}`);
    // Simplified: resolve constructor params recursively
    return new ctor();
  }
}

// Real-world: Type-safe Redux middleware
type Middleware<Dispatch extends (...args: any[]) => any> = (
  api: { dispatch: Dispatch; getState: () => unknown }
) => (next: Dispatch) => Dispatch;

type AppDispatch = (action: { type: string; payload?: unknown }) => void;
const logger: Middleware<AppDispatch> = (api) => (next) => (action) => {
  console.log("dispatching", action);
  return next(action);
};
```

### Common Mistakes

1. **Using `ReturnType` with generic functions**: The result is the generic return type, not a specific instantiation.
2. **Not handling async functions**: `ReturnType<AsyncFn>` is `Promise<T>`, not `T`.
3. **Using `typeof` with class methods**: Remember `typeof Class.prototype.method` for instance methods.
4. **Confusing `Parameters` (tuple) with individual parameters**.
5. **Forgetting that overloaded functions return the last overload's types**.

### Best Practices

- Use `ReturnType` and `Parameters` for higher-order function typing.
- Combine `ReturnType` with `Promise<infer U>` to unwrap async return types.
- Use `ConstructorParameters` and `InstanceType` for factory functions and DI.
- Prefer these utility types over manually writing function type signatures.
- Remember that these utilities work best with concrete (not generic) functions.

### Performance Considerations

These utility types are highly optimized in the compiler. They are among the most efficient conditional types because they match simple, common patterns.

### Interview Questions

1. **Q**: How is `ReturnType<T>` implemented?
   **A**: `type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;`

2. **Q**: What does `Parameters<T>` return?
   **A**: A tuple type of the function's parameters. For `(a: string, b: number) => void`, it returns `[string, number]`.

3. **Q**: Can `ReturnType` extract the return type of an async function?
   **A**: Yes, but it returns `Promise<T>`, not `T`. You need to unwrap with `ReturnType<T> extends Promise<infer U> ? U : ReturnType<T>`.

### Coding Challenges

1. Implement an `AwaitedReturnType<T>` that returns the resolved type of an async function.
2. Write a type-safe `debounce` function that preserves parameters and return types.
3. Create a `compose` function that infers the types through a chain of function compositions.

### Related Topics

- `infer` keyword
- Conditional types
- Function types
- Higher-order functions
- Overloads
