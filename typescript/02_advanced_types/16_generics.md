# Generics - Generic functions, generic constraints, generic classes, multiple type params

## Introduction

Generics are a cornerstone of TypeScript's type system, enabling the creation of reusable components that work with a variety of types rather than a single one. A generic is a type parameter—a placeholder for a type that is specified when the component is used. Generics allow you to write functions, classes, and interfaces that are type-safe across different input types while maintaining the relationship between inputs and outputs.

Without generics, you would have to use `any` (losing type safety) or overload every possible type combination (creating code duplication). Generics provide the best of both worlds: reusability with precise type tracking. Understanding generics is essential for writing high-quality TypeScript libraries, utility functions, and type-safe application code.

## Generic functions

### What It Is

A generic function uses one or more type parameters (usually denoted by `<T>`) to capture the type of its arguments and relate it to the return type or other arguments. The type parameter is specified when calling the function, either explicitly or inferred from the arguments.

```typescript
function identity<T>(value: T): T {
  return value;
}
```

### Why It Is Important

Generic functions eliminate code duplication while preserving full type safety. Instead of writing separate functions for `string[]`, `number[]`, and `boolean[]`, you write one generic function that works for all types. The caller gets precise type inference—the return type is tied to the input type.

### How It Works Internally

The compiler infers the type parameter from the arguments at each call site. A fresh type variable is created for each invocation. The compiler checks that all usages of the type parameter within the function body are consistent with the inferred type. The resulting JavaScript is not generic—type parameters are erased.

```typescript
// TypeScript: function identity<T>(value: T): T { return value; }
// JavaScript: function identity(value) { return value; }
```

### Syntax

```typescript
// Generic function declaration
function firstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

// Generic arrow function
const firstElement = <T>(arr: T[]): T | undefined => arr[0];

// Generic function expression
const firstElement: <T>(arr: T[]) => T | undefined = (arr) => arr[0];

// In type alias
type FirstElement = <T>(arr: T[]) => T | undefined;

// In interface
interface FirstElement {
  <T>(arr: T[]): T | undefined;
}

// Multiple calls
function map<T, U>(arr: T[], fn: (item: T) => U): U[] {
  return arr.map(fn);
}
```

### Beginner Examples

```typescript
// Identity function
function identity<T>(value: T): T {
  return value;
}

const num = identity(42); // T inferred as number
const str = identity("hello"); // T inferred as string

// Generic array function
function last<T>(arr: T[]): T | undefined {
  return arr[arr.length - 1];
}

const lastNum = last([1, 2, 3]); // number | undefined
const lastStr = last(["a", "b"]); // string | undefined

// Generic merge function
function merge<T, U>(a: T, b: U): T & U {
  return { ...a, ...b };
}

const merged = merge({ name: "Alice" }, { age: 30 });
// merged: { name: string } & { age: number }

// Generic with array methods
function wrapInArray<T>(value: T): T[] {
  return [value];
}

const arr = wrapInArray("hello"); // string[]
```

### Intermediate Examples

```typescript
// Generic function with type parameter in callback
function transform<T, U>(value: T, fn: (val: T) => U): U {
  return fn(value);
}

const length = transform("hello", (s) => s.length);
// length: number

// Generic promise wrapper
async function fetchJson<T>(url: string): Promise<T> {
  const response = await fetch(url);
  return response.json();
}

const user = await fetchJson<User>("/api/user/1");
// user: User

// Generic key extraction
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const obj = { name: "Alice", age: 30 };
const name = getProperty(obj, "name"); // string
const age = getProperty(obj, "age"); // number

// Generic function with array filter
function filterNonNull<T>(arr: (T | null | undefined)[]): T[] {
  return arr.filter((x): x is T => x != null);
}

const result = filterNonNull([1, null, 2, undefined, 3]); // number[]

// Generic pipe/compose
function pipe<T, U, V>(fn1: (x: T) => U, fn2: (y: U) => V): (x: T) => V {
  return (x) => fn2(fn1(x));
}

const toUpperCase = (s: string) => s.toUpperCase();
const exclaim = (s: string) => s + "!";
const shout = pipe(toUpperCase, exclaim);
// shout: (x: string) => string
```

### Advanced Examples

```typescript
// Generic with variadic tuple types
function concat<T extends readonly unknown[], U extends readonly unknown[]>(
  arr1: T,
  arr2: U
): [...T, ...U] {
  return [...arr1, ...arr2];
}

const result = concat([1, 2] as const, ["a", "b"] as const);
// result: readonly [1, 2, "a", "b"]

// Generic currying
function curry<T, U, V>(fn: (a: T, b: U) => V): (a: T) => (b: U) => V {
  return (a) => (b) => fn(a, b);
}

const add = (a: number, b: number) => a + b;
const curriedAdd = curry(add);
const add5 = curriedAdd(5);
const result2 = add5(3); // 8

// Generic with discriminated union
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

function unwrap<T>(result: Result<T>): T {
  if (result.ok) return result.value;
  throw result.error;
}

// Generic recursive types
type DeepReadonly<T> = T extends object
  ? { readonly [P in keyof T]: DeepReadonly<T[P]> }
  : T;

// Generic function for deep cloning
function deepClone<T>(value: T): T {
  if (typeof value !== "object" || value === null) return value;
  if (Array.isArray(value)) return value.map(deepClone) as T;
  return Object.fromEntries(
    Object.entries(value as Record<string, unknown>).map(([k, v]) => [k, deepClone(v)])
  ) as T;
}

// Generic with inference from multiple arguments
function makePair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const pair = makePair("hello", 42); // [string, number]
```

### Real-World Use Cases

- **Utility functions**: `debounce`, `throttle`, `memoize` — all benefit from generics.
- **API clients**: Generic fetch wrappers that return typed responses.
- **State management**: Generic store and reducer patterns.
- **Data transformers**: Functions that map, filter, reduce with type-safe callbacks.
- **Validation functions**: Generic validators that return typed results.
- **Event emitters**: Generic event handling with typed payloads.

```typescript
// Real-world: Generic memoization
function memoize<T extends (...args: any[]) => any>(fn: T): T {
  const cache = new Map<string, ReturnType<T>>();
  return ((...args: Parameters<T>): ReturnType<T> => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key)!;
    const result = fn(...args);
    cache.set(key, result);
    return result;
  }) as T;
}

const expensiveFn = memoize((x: number, y: number) => x + y);
// expensiveFn has the same signature as the original
```

### Common Mistakes

1. **Unnecessary generics**: Using `<T>` when `any` or a concrete type suffices.
2. **Forgetting to constrain**: `T` could be any type; use `extends` when you need specific properties.
3. **Not inferring types**: Specifying `<string>` explicitly when inference works.
4. **Using generics for every function**: Sometimes a concrete union type is clearer.
5. **Confusing type parameters with values**: Generics are compile-time only; you can't use them at runtime.

### Best Practices

- Let TypeScript infer generic types when possible; only specify them when inference fails.
- Use descriptive names for type parameters (`TItem`, `TResponse`) in complex code.
- Constrain type parameters with `extends` to document what properties are expected.
- Use generic functions instead of function overloads when the overloads differ only in types.
- Prefer multiple simple generics over a single complex one.

### Performance Considerations

Generics are erased at compile time and have zero runtime cost. However, complex generic inferences and deeply nested generic types can slow down compilation. For performance-critical type-level code, keep generics simple and avoid recursive generic types with unbounded depth.

### Interview Questions

1. **Q**: What is the difference between a generic function and a function overload?
   **A**: Overloads define multiple explicit signatures. Generics capture the type relationship between parameters and return value. Generics are more flexible for type-preserving transformations.

2. **Q**: How does TypeScript infer generic type parameters?
   **A**: It looks at the types of the arguments passed and uses type inference to determine the type parameters. For multiple candidates, it picks the most specific type.

### Coding Challenges

1. Implement a generic `pipe` function that composes N functions with correct types.
2. Write a generic `groupBy` function that groups an array by a key selector.
3. Implement a type-safe event emitter using generics.

### Related Topics

- Generic constraints
- Generic classes
- Type parameter inference
- Utility types

## Generic constraints (extends)

### What It Is

Generic constraints restrict what types can be used for a type parameter. The `extends` keyword specifies that the type parameter must be assignable to a certain type. This allows you to access properties and methods of the constraint type within the generic function.

```typescript
function getLength<T extends { length: number }>(value: T): number {
  return value.length;
}
```

### Why It Is Important

Constraints make generics practical by guaranteeing that certain properties or methods exist on the type parameter. Without constraints, you can only access properties that exist on all types (essentially nothing useful). Constraints document the contract that callers must satisfy.

### How It Works Internally

When a type parameter is constrained with `extends`, the compiler checks that the inferred/concrete type satisfies the constraint. Within the generic body, the type parameter is treated as a subtype of the constraint, giving access to its properties.

```typescript
// T extends { length: number } → T has a length property of type number
```

### Syntax

```typescript
// Basic constraint
function longest<T extends { length: number }>(a: T, b: T): T {
  return a.length >= b.length ? a : b;
}

// Constraint with interface
interface HasLength { length: number; }
function longest<T extends HasLength>(a: T, b: T): T {
  return a.length >= b.length ? a : b;
}

// Multiple constraints (using intersection)
function process<T extends { name: string } & { age: number }>(obj: T): void {}

// Self-referencing constraint
function compare<T extends { compareTo: (other: T) => number }>(a: T, b: T): number {
  return a.compareTo(b);
}

// Keyof constraint
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

### Beginner Examples

```typescript
// Length constraint
function longest<T extends { length: number }>(a: T, b: T): T {
  return a.length >= b.length ? a : b;
}

const longerStr = longest("hello", "world"); // string
const longerArr = longest([1, 2], [1, 2, 3]); // number[]

// Name constraint
interface Named { name: string; }

function greet<T extends Named>(entity: T): string {
  return `Hello, ${entity.name}`;
}

greet({ name: "Alice", age: 30 }); // OK
// greet({ age: 30 }); // Error: no name property

// Minimum constraint
function minLength<T extends { length: number }>(value: T, min: number): T {
  if (value.length < min) throw new Error("Too short");
  return value;
}

minLength("hello", 3); // OK
minLength([1, 2, 3], 5); // Error (runtime)
```

### Intermediate Examples

```typescript
// Constraint with constructor signature
function createInstance<T>(ctor: new (...args: any[]) => T, ...args: any[]): T {
  return new ctor(...args);
}

class User { constructor(public name: string) {} }
const user = createInstance(User, "Alice"); // User

// Constrained generic with type mapping
function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] {
  return items.map(item => item[key]);
}

const users = [{ name: "Alice", age: 30 }, { name: "Bob", age: 25 }];
const names = pluck(users, "name"); // string[]
const ages = pluck(users, "age"); // number[]

// Constraint with index signature
function toRecord<T extends string>(keys: T[], value: number): Record<T, number> {
  const result = {} as Record<T, number>;
  keys.forEach(key => { result[key] = value; });
  return result;
}

const record = toRecord(["a", "b", "c"], 1);
// record: Record<"a" | "b" | "c", number>

// Constraint with callback
function withLogging<T extends (...args: any[]) => any>(
  fn: T,
  ...args: Parameters<T>
): ReturnType<T> {
  console.log(`Calling with ${args}`);
  return fn(...args);
}
```

### Advanced Examples

```typescript
// Constraint for type-safe builder pattern
class QueryBuilder<T extends Record<string, unknown>> {
  private conditions: string[] = [];

  where<K extends keyof T>(key: K, value: T[K]): this {
    this.conditions.push(`${String(key)} = ${value}`);
    return this;
  }

  build(): string {
    return `SELECT * FROM table WHERE ${this.conditions.join(" AND ")}`;
  }
}

interface User { id: number; name: string; email: string; }
const query = new QueryBuilder<User>()
  .where("id", 1)
  .where("name", "Alice")
  .build();

// Constraint with template literal types
function createApiEndpoint<T extends string>(base: T) {
  return {
    get: <R>(id: string): Promise<R> => fetch(`/api/${base}/${id}`).then(r => r.json()),
    list: <R>(): Promise<R[]> => fetch(`/api/${base}`).then(r => r.json()),
  };
}

const usersApi = createApiEndpoint("users");

// Constraint for discriminated union narrowing
type Shape<T extends string> = { kind: T };

function getKind<T extends string>(shape: Shape<T>): T {
  return shape.kind;
}

// Conditional constraints
type StringKeys<T> = {
  [K in keyof T]: T[K] extends string ? K : never;
}[keyof T];

function getStringValues<T>(obj: T): Pick<T, StringKeys<T>> {
  const result = {} as Pick<T, StringKeys<T>>;
  for (const key of Object.keys(obj as any)) {
    if (typeof (obj as any)[key] === "string") {
      (result as any)[key] = (obj as any)[key];
    }
  }
  return result;
}

// Constraint for type-level arithmetic
type AtLeast<T, K extends keyof T> = Partial<T> & Pick<T, K>;
type Required<T, K extends keyof T> = Omit<T, K> & { [P in K]-?: T[P] };

function ensure<T, K extends keyof T>(obj: Partial<T>, key: K): asserts obj is Required<T, K> {
  if (!(key in obj)) throw new Error(`Missing ${String(key)}`);
}
```

### Real-World Use Cases

- **ORM query builders**: Constraining generic types to entity fields.
- **Form validation**: Generic validators constrained to field types.
- **API response parsing**: Generic fetch with type-level endpoint matching.
- **Configuration loaders**: Generic config readers constrained to expected shapes.
- **Plugin systems**: Generic plugin interfaces with constrained capabilities.

### Common Mistakes

1. **Not constraining when accessing properties**: `T.length` fails without `extends { length: number }`.
2. **Over-constraining**: Requiring more than necessary limits reusability.
3. **Using `any` in constraints**: `extends any` is meaningless.
4. **Constraining to primitive types unnecessarily**: `T extends string` works but may be too restrictive.

### Best Practices

- Constrain as loosely as possible while still accessing the properties you need.
- Use `extends` to document the minimum required interface.
- Prefer composing small constraint interfaces over a single large one.
- Use `keyof T` constraints for property accessors.
- Avoid constraining to concrete types when interfaces suffice.

### Performance Considerations

Constraints add no runtime overhead. They may slightly increase compile-time checking but are generally negligible.

### Interview Questions

1. **Q**: How do you constrain a generic type to have a specific property?
   **A**: Use `T extends { propertyName: PropertyType }`.

2. **Q**: Can a type parameter be constrained by another type parameter?
   **A**: Yes: `function compare<T, U extends T>(a: T, b: U) {}`.

### Coding Challenges

1. Implement a generic `pick` function that returns a new object with only the specified keys.
2. Write a generic `hasProperty` type guard that is constrained by property name.

### Related Topics

- `keyof` operator
- Conditional types
- Generic parameter defaults
- Generic inference

## Generic classes and interfaces

### What It Is

Generic classes and interfaces allow you to define reusable type structures where one or more types are parameterized. They enable creating type-safe containers, collections, and abstractions that work with any type.

```typescript
class Box<T> {
  constructor(public value: T) {}
}

interface Repository<T> {
  getById(id: string): Promise<T>;
  getAll(): Promise<T[]>;
  create(item: T): Promise<T>;
}
```

### Why It Is Important

Generic classes and interfaces are the foundation for type-safe data structures, service abstractions, and state management. They enable code reuse without sacrificing type safety, and they make APIs self-documenting about what types they work with.

### How It Works Internally

The compiler stores the type parameter at the class/interface level. When the class is instantiated or the interface is implemented, the type parameter is resolved. The type information is erased at runtime.

```typescript
// TypeScript: class Box<T> { constructor(public value: T) {} }
// JavaScript: class Box { constructor(value) { this.value = value; } }
```

### Syntax

```typescript
// Generic class
class Stack<T> {
  private items: T[] = [];

  push(item: T): void { this.items.push(item); }
  pop(): T | undefined { return this.items.pop(); }
  peek(): T | undefined { return this.items[this.items.length - 1]; }
  get length(): number { return this.items.length; }
}

// Generic interface
interface Comparator<T> {
  compare(a: T, b: T): number;
}

interface Serializer<T, U = string> {
  serialize(value: T): U;
  deserialize(value: U): T;
}

// Generic class with constraint
class Collection<T extends { id: string }> {
  private items = new Map<string, T>();

  add(item: T): void { this.items.set(item.id, item); }
  get(id: string): T | undefined { return this.items.get(id); }
}

// Generic abstract class
abstract class BaseEntity<T> {
  abstract validate(): boolean;
  abstract toDTO(): T;
}
```

### Beginner Examples

```typescript
// Generic Pair class
class Pair<T, U> {
  constructor(public first: T, public second: U) {}

  swap(): Pair<U, T> {
    return new Pair(this.second, this.first);
  }
}

const pair = new Pair("hello", 42);
const swapped = pair.swap(); // Pair<number, string>

// Generic Queue class
class Queue<T> {
  private data: T[] = [];

  enqueue(item: T): void { this.data.push(item); }
  dequeue(): T | undefined { return this.data.shift(); }
  isEmpty(): boolean { return this.data.length === 0; }
}

const queue = new Queue<number>();
queue.enqueue(1);
queue.enqueue(2);
const first = queue.dequeue(); // number | undefined

// Generic interface
interface KeyValuePair<K, V> {
  key: K;
  value: V;
}

const entry: KeyValuePair<string, number> = { key: "age", value: 30 };
```

### Intermediate Examples

```typescript
// Generic repository interface
interface Repository<T, ID = string> {
  findById(id: ID): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: ID): Promise<void>;
}

class UserRepository implements Repository<User, number> {
  async findById(id: number): Promise<User | null> { /* ... */ return null; }
  async findAll(): Promise<User[]> { return []; }
  async save(entity: User): Promise<User> { return entity; }
  async delete(id: number): Promise<void> { }
}

// Generic singleton pattern
class Singleton<T> {
  private static instances = new Map<string, any>();

  static getInstance<T>(key: string, factory: () => T): T {
    if (!this.instances.has(key)) {
      this.instances.set(key, factory());
    }
    return this.instances.get(key) as T;
  }
}

const db = Singleton.getInstance("db", () => new Database());

// Generic builder pattern
class Builder<T extends Record<string, unknown>> {
  private entity: Partial<T> = {};

  set<K extends keyof T>(key: K, value: T[K]): this {
    this.entity[key] = value;
    return this;
  }

  build(): T {
    return this.entity as T;
  }
}

interface User { id: number; name: string; email: string; }
const userBuilder = new Builder<User>()
  .set("id", 1)
  .set("name", "Alice")
  .set("email", "alice@example.com")
  .build();
```

### Advanced Examples

```typescript
// Generic class with computed property types
class TypedMap<T extends Record<string, unknown>> {
  private data: T;

  constructor(initial: T) {
    this.data = { ...initial };
  }

  get<K extends keyof T>(key: K): T[K] {
    return this.data[key];
  }

  set<K extends keyof T>(key: K, value: T[K]): void {
    this.data[key] = value;
  }

  toJSON(): T {
    return { ...this.data };
  }
}

const map = new TypedMap({ name: "Alice", age: 30 });
const name = map.get("name"); // string
map.set("age", 31);

// Generic class with method chaining
class Query<T extends Record<string, unknown>> {
  private conditions: Array<[keyof T, string, unknown]> = [];

  where<K extends keyof T>(key: K, op: "eq" | "gt" | "lt", value: T[K]): this {
    this.conditions.push([key, op, value]);
    return this;
  }

  orWhere<K extends keyof T>(key: K, op: "eq" | "gt" | "lt", value: T[K]): this {
    // Implementation
    return this;
  }

  execute(): Promise<T[]> {
    // Execute query
    return Promise.resolve([]);
  }
}

// Generic interface with indexing
interface Indexed<T> {
  [key: string]: T;
}

// Generic class with static factory
class Result<T, E = Error> {
  private constructor(
    private readonly isOk: boolean,
    private readonly value?: T,
    private readonly error?: E
  ) {}

  static ok<T>(value: T): Result<T, never> {
    return new Result(true, value);
  }

  static fail<E>(error: E): Result<never, E> {
    return new Result(false, undefined, error);
  }

  unwrap(): T {
    if (!this.isOk) throw this.error;
    return this.value!;
  }

  map<U>(fn: (value: T) => U): Result<U, E> {
    if (!this.isOk) return this as Result<never, E> as Result<U, E>;
    return Result.ok(fn(this.value!));
  }
}

const ok = Result.ok(42);
const mapped = ok.map(x => x.toString()); // Result<string, never>
```

### Real-World Use Cases

- **Data structures**: Custom collections (Stack, Queue, Tree, Graph) with type-safe elements.
- **Repository pattern**: Database access abstractions parameterized by entity type.
- **State management**: Generic store and reducer classes.
- **API clients**: HTTP client classes parameterized by response types.
- **Validation**: Generic validator classes for form fields and data models.
- **Caching**: Generic cache implementations with typed keys and values.

```typescript
// Real-world: Generic cache with TTL
class Cache<T> {
  private store = new Map<string, { value: T; expiresAt: number }>();

  constructor(private ttlMs: number = 60000) {}

  get(key: string): T | undefined {
    const entry = this.store.get(key);
    if (entry && entry.expiresAt > Date.now()) return entry.value;
    this.store.delete(key);
    return undefined;
  }

  set(key: string, value: T): void {
    this.store.set(key, { value, expiresAt: Date.now() + this.ttlMs });
  }

  invalidate(key: string): void {
    this.store.delete(key);
  }
}
```

### Common Mistakes

1. **Not specifying the type parameter when creating an instance**: `new Stack()` defaults to `Stack<unknown>`.
2. **Trying to use generics at runtime**: Type parameters are erased; you can't do `T === String`.
3. **Overly complex generic class hierarchies**: Keep generics in classes focused.
4. **Mixing instance and static generic parameters**: Static members cannot use the class's type parameter.

### Best Practices

- Infer the type parameter from constructor arguments when possible.
- Use generic interfaces for contracts and generic classes for implementations.
- Keep type parameter lists short (1-2 parameters per class).
- Use descriptive names for type parameters (`TEntity`, `TId`, `TResponse`).
- Consider whether a generic interface + concrete implementations is cleaner than a generic class.

### Performance Considerations

Generic classes have no runtime overhead compared to non-generic ones. The JavaScript output is identical. Type checking is done at compile time.

### Interview Questions

1. **Q**: Can a static method in a generic class use the class's type parameter?
   **A**: No. Static members are per-class, not per-instance. They cannot access the instance type parameter.

2. **Q**: How do you create a new instance of a generic type parameter?
   **A**: You can't directly (`new T()` is not possible). Pass a constructor function as an argument: `function create<T>(ctor: new () => T): T { return new ctor(); }`.

### Coding Challenges

1. Implement a generic `EventEmitter` class with typed events and payloads.
2. Build a generic `Tree<T>` class with type-safe traversal methods.
3. Create a generic `Validator<T>` interface with implementations for different types.

### Related Topics

- Generic functions
- Generic constraints
- Mixins
- Factory pattern

## Multiple type parameters

### What It Is

Generics can accept multiple type parameters, separated by commas. Each type parameter can have its own constraints and defaults. Multiple type parameters allow functions and types to relate different types to each other.

```typescript
function map<T, U>(arr: T[], fn: (item: T) => U): U[] {
  return arr.map(fn);
}
```

### Why It Is Important

Multiple type parameters are essential for functions that transform types, combine different types, or relate inputs to outputs. They enable type-safe mapping, zipping, merging, and other operations that work with two or more independent type dimensions.

### How It Works Internally

Each type parameter is independently inferred or specified. The compiler tracks all type parameters and their relationships. Inference can propagate between parameters (e.g., inferring `U` from the return type of a callback that receives `T`).

### Syntax

```typescript
// Two type parameters
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

// Three type parameters
function transform<A, B, C>(
  value: A,
  fn1: (a: A) => B,
  fn2: (b: B) => C
): C {
  return fn2(fn1(value));
}

// With defaults
function create<T, U = string>(first: T, second?: U): [T, U] {
  return [first, second as U];
}

// With constraints
function merge<T extends object, U extends object>(a: T, b: U): T & U {
  return { ...a, ...b };
}

// Partial inference
function createPair<T, U = T>(first: T, second?: U): [T, U] {
  return [first, second as U];
}
```

### Beginner Examples

```typescript
// Key-value pair
function createEntry<K, V>(key: K, value: V): { key: K; value: V } {
  return { key, value };
}

const entry = createEntry("id", 1);
// entry: { key: string; value: number }

// Zip two arrays
function zip<T, U>(arr1: T[], arr2: U[]): [T, U][] {
  const length = Math.min(arr1.length, arr2.length);
  const result: [T, U][] = [];
  for (let i = 0; i < length; i++) {
    result.push([arr1[i], arr2[i]]);
  }
  return result;
}

const zipped = zip(["a", "b", "c"], [1, 2, 3]);
// zipped: [string, number][]

// Object from entries
function fromEntries<K extends string, V>(entries: [K, V][]): Record<K, V> {
  const result = {} as Record<K, V>;
  for (const [key, value] of entries) {
    result[key] = value;
  }
  return result;
}
```

### Intermediate Examples

```typescript
// Generic with index access
function getPair<T, K1 extends keyof T, K2 extends keyof T>(
  obj: T,
  key1: K1,
  key2: K2
): [T[K1], T[K2]] {
  return [obj[key1], obj[key2]];
}

const obj = { a: 1, b: "hello", c: true };
const pair = getPair(obj, "a", "b"); // [number, string]

// Bivariate function
function compare<T, U extends T>(a: T, b: U): boolean {
  return JSON.stringify(a) === JSON.stringify(b);
}

// Type-safe event emitter
class EventEmitter<Events extends Record<string, unknown>> {
  private listeners = new Map<keyof Events, Array<(data: any) => void>>();

  on<K extends keyof Events>(event: K, listener: (data: Events[K]) => void): void {
    if (!this.listeners.has(event)) this.listeners.set(event, []);
    this.listeners.get(event)!.push(listener);
  }

  emit<K extends keyof Events>(event: K, data: Events[K]): void {
    this.listeners.get(event)?.forEach(listener => listener(data));
  }
}

interface AppEvents {
  userLogin: { userId: number; timestamp: Date };
  error: { message: string; code: number };
  logout: void;
}

const emitter = new EventEmitter<AppEvents>();
emitter.on("userLogin", (data) => console.log(data.userId));
emitter.emit("userLogin", { userId: 1, timestamp: new Date() });
```

### Advanced Examples

```typescript
// Multiple type parameters with variadic tuples
function concat<T extends readonly unknown[], U extends readonly unknown[]>(
  arr1: T,
  arr2: U
): [...T, ...U] {
  return [...arr1, ...arr2];
}

// Type-safe Redux reducer
type Reducer<S, A> = (state: S, action: A) => S;

function combineReducers<S extends Record<string, unknown>, A>(
  reducers: {
    [K in keyof S]: Reducer<S[K], A>;
  }
): Reducer<S, A> {
  return (state, action) => {
    const nextState = {} as S;
    for (const key in reducers) {
      nextState[key] = reducers[key](state[key], action);
    }
    return nextState;
  };
}

// Generic with type parameter for options
function createSelector<T, R, P extends any[] = []>(
  selector: (state: T, ...params: P) => R
): (state: T, ...params: P) => R {
  let lastState: T | undefined;
  let lastResult: R | undefined;

  return (state: T, ...params: P): R => {
    if (state !== lastState) {
      lastResult = selector(state, ...params);
      lastState = state;
    }
    return lastResult!;
  };
}

// Multiple type parameters with recursive types
type DeepMap<T, U> = T extends object
  ? { [K in keyof T]: DeepMap<T[K], U> }
  : U;

type Stringified = DeepMap<{ a: number; b: { c: boolean } }, string>;
// { a: string; b: { c: string } }

// Function with type parameter defaulting
function makeArray<T, U = T>(first: T, second?: U): (T | U)[] {
  return second !== undefined ? [first, second] : [first];
}

const arr1 = makeArray(1); // number[]
const arr2 = makeArray(1, "hello"); // (number | string)[]
```

### Real-World Use Cases

- **React components**: Generic props with multiple type parameters for data and actions.
- **Redux**: `Reducer<S, A>` with state and action type parameters.
- **API clients**: `ApiClient<Req, Res>` with request and response types.
- **Database queries**: `Query<T, R>` with input and result types.
- **Form state**: `FormState<T, E>` with data and error type parameters.
- **Validation rules**: `Validator<T, R>` with input and result types.

```typescript
// Real-world: Generic React component (simplified)
interface ListProps<T, U> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  onItemSelect?: (item: T, meta: U) => void;
  meta?: U;
}

function List<T, U>(props: ListProps<T, U>): React.ReactElement {
  return <div>{props.items.map((item, i) => props.renderItem(item, i))}</div>;
}
```

### Common Mistakes

1. **Too many type parameters**: More than 3-4 type parameters can make the code hard to read.
2. **Not ordering parameters logically**: Put required parameters first, optional/defaulted ones last.
3. **Forgetting that type parameters are independent**: Each is inferred separately unless constrained.
4. **Not using partial inference**: Sometimes you need to specify some parameters and let others be inferred.

### Best Practices

- Limit to 1-3 type parameters most of the time.
- Place type parameters with defaults after those without.
- Use the order that makes partial inference practical (parameters that can be inferred first).
- Consider whether separate overloads or a union type would be clearer.
- Name type parameters meaningfully: `TInput`, `TOutput`, `TError`.

### Performance Considerations

Multiple type parameters have no runtime cost. They can increase compile-time checking proportional to the number of combinations the compiler evaluates. This is typically negligible.

### Interview Questions

1. **Q**: How do you specify only some type parameters and let others be inferred?
   **A**: You can't partially specify type parameters in a function call. You must either specify all or let all be inferred. Workarounds include currying or using an options object.

2. **Q**: Can you have variadic type parameters (arbitrary number)?
   **A**: Yes, with variadic tuple types: `function concat<T extends readonly unknown[]>(...arrays: T): T`.

### Coding Challenges

1. Implement a generic `memoize` function that works with functions of multiple arguments.
2. Build a type-safe dependency injection container using multiple generic type parameters.
3. Create a generic finite state machine with typed states and transitions.

### Related Topics

- Variadic tuple types
- Function overloads
- Generic defaults
- Type inference
