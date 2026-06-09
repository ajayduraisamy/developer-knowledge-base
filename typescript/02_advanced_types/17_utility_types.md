# Utility Types - Partial/Required/Readonly, Pick/Omit, Record/Exclude/Extract, NonNullable

## Introduction

TypeScript provides a set of built-in utility types that facilitate common type transformations. These utility types are generic types that modify existing types in various ways—making properties optional, picking specific properties, excluding types from unions, and more. They are globally available and eliminate the need to write common type transformations manually.

Utility types are implemented using TypeScript's advanced type features (mapped types, conditional types, indexed access types) but are provided as ready-to-use tools. Understanding when and how to use them is essential for writing concise, maintainable TypeScript code.

## Partial and Required

### What It Is

`Partial<T>` converts all properties of `T` to optional. `Required<T>` converts all properties of `T` to required (removes `?`).

```typescript
type Partial<T> = { [P in keyof T]?: T[P] };
type Required<T> = { [P in keyof T]-?: T[P] };
```

### Why It Is Important

These utility types are essential for scenarios where you need to create or update objects incrementally. `Partial` is used for update operations, form states, and configuration merging. `Required` is used to ensure complete objects, especially after validation or initialization.

### How It Works Internally

`Partial` maps over each property of `T` and adds the `?` modifier. `Required` does the opposite, removing the `?` modifier with the `-?` syntax. Both preserve the `readonly` modifier if present.

```typescript
// Partial<{ name: string; age: number }> = { name?: string; age?: number }
// Required<{ name?: string; age?: number }> = { name: string; age: number }
```

### Syntax

```typescript
Partial<T>
Required<T>
```

### Beginner Examples

```typescript
interface User {
  name: string;
  email: string;
  age: number;
}

// Partial: all properties optional
function updateUser(id: number, updates: Partial<User>): void {
  // Only update provided fields
}

updateUser(1, { name: "Alice" }); // OK
updateUser(1, { age: 30 }); // OK
updateUser(1, {}); // OK (empty update)

// Required: all properties required
interface Config {
  host?: string;
  port?: number;
  debug?: boolean;
}

function initServer(config: Required<Config>): void {
  // All properties guaranteed
}

initServer({ host: "localhost", port: 3000, debug: true }); // OK
// initServer({ host: "localhost" }); // Error: missing port and debug

// Partial for default values
function createDefaultUser(): Required<Partial<User>> {
  return { name: "Default", email: "", age: 0 };
}
```

### Intermediate Examples

```typescript
// Deep Partial (recursive)
type DeepPartial<T> = T extends object
  ? { [P in keyof T]?: DeepPartial<T[P]> }
  : T;

interface Nested {
  user: { name: string; address: { city: string; zip: string } };
  settings: { theme: string };
}

const partialNested: DeepPartial<Nested> = {
  user: { name: "Alice" }, // Deeply optional
};

// Partial for form state
interface FormState {
  values: Record<string, string>;
  errors: Record<string, string>;
  touched: Record<string, boolean>;
  isSubmitting: boolean;
}

type PartialFormState = Partial<FormState>;
type InitialFormState = Required<FormState>;

// Required with Pick for specific required fields
type WithRequired<T, K extends keyof T> = Omit<T, K> & Required<Pick<T, K>>;

type UserWithRequiredEmail = WithRequired<User, "email">;
// email is required, name and age stay as-is

// Partial in generic function
function mergeConfig<T extends object>(
  defaults: T,
  overrides: Partial<T>
): Required<T> {
  return { ...defaults, ...overrides } as Required<T>;
}
```

### Advanced Examples

```typescript
// Partial for API update payload
type UpdatePayload<T> = {
  [K in keyof T]?: T[K] extends object ? UpdatePayload<T[K]> : T[K];
};

interface Profile {
  name: string;
  preferences: { theme: string; fontSize: number };
}

// Required with conditional types
type RequiredByTag<T, Tag extends string> = {
  [K in keyof T as T[K] extends { required: true } ? K : never]: T[K];
};

// Partial for builder pattern
class Builder<T extends object> {
  private partial: Partial<T> = {};

  set<K extends keyof T>(key: K, value: T[K]): this {
    this.partial[key] = value;
    return this;
  }

  build(): T {
    return this.partial as T;
  }
}

// Combining Partial and Required for selective optionality
type MakeOptional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;
type MakeRequired<T, K extends keyof T> = Omit<T, K> & Required<Pick<T, K>>;

interface Person {
  id: string;
  name: string;
  email: string;
  phone?: string;
}

type PersonWithOptionalEmail = MakeOptional<Person, "email">;
type PersonWithRequiredPhone = MakeRequired<Person, "phone">;
```

### Real-World Use Cases

- **API PATCH endpoints**: `Partial<Entity>` for partial updates.
- **Form state management**: `Partial<FormValues>` for incomplete form data.
- **Configuration merging**: `Partial<Config>` for user overrides.
- **Initialization states**: `Required<Config>` after validation.
- **Feature flags**: `Partial<Features>` for enabling specific features.

```typescript
// Real-world: Express PATCH handler
interface UserEntity {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

async function patchUser(id: number, updates: Partial<UserEntity>): Promise<UserEntity> {
  // Only update provided fields
  const user = await db.findUser(id);
  const updated = { ...user, ...updates };
  return db.saveUser(updated);
}
```

### Common Mistakes

1. **Using `Partial` when you should use optional properties directly**.
2. **Not realizing `Partial` is shallow**—nested objects remain as-is.
3. **Using `Required` on types that already have all properties required** (no effect).
4. **Assuming `Partial` makes nested properties partial**—use `DeepPartial` instead.

### Best Practices

- Use `Partial` for update/merge scenarios where you don't need all properties.
- Use `Required` after validation/initialization to guarantee completeness.
- Implement `DeepPartial` for deeply nested object updates.
- Combine `Partial` with `Pick` for fine-grained optionality control.
- Prefer specific optional properties over blanket `Partial` in interface design.

### Performance Considerations

Utility types are computed at compile time and have no runtime cost. They may slightly increase type-checking time but the effect is negligible.

### Interview Questions

1. **Q**: What is the difference between `Partial<T>` and `T` with optional properties?
   **A**: `Partial<T>` makes all properties optional. If `T` already has optional properties, `Partial<T>` adds `?` to properties that were already required.

2. **Q**: How do you implement a deep `Partial`?
   **A**: Recursively: `type DeepPartial<T> = T extends object ? { [P in keyof T]?: DeepPartial<T[P]> } : T;`

### Coding Challenges

1. Implement a generic `updateObject` function that uses `Partial<T>` for updates.
2. Create a `DeepRequired` utility type that makes all nested properties required.

### Related Topics

- Mapped types
- `Pick` and `Omit`
- Optional properties
- `-?` modifier

## Readonly

### What It Is

`Readonly<T>` makes all properties of `T` readonly, preventing reassignment after initialization.

```typescript
type Readonly<T> = { readonly [P in keyof T]: T[P] };
```

### Why It Is Important

`Readonly<T>` is essential for creating immutable data structures, configuration objects that should not change, and ensuring data integrity in functional programming patterns. It provides compile-time enforcement of immutability.

### How It Works Internally

The compiler adds the `readonly` modifier to each property. Assignment to a readonly property produces a compile-time error. Readonly is a compile-time concept—at runtime, the properties are still mutable.

```typescript
// Readonly<{ name: string; age: number }> = { readonly name: string; readonly age: number }
```

### Syntax

```typescript
Readonly<T>
```

### Beginner Examples

```typescript
interface Point {
  x: number;
  y: number;
}

const origin: Readonly<Point> = { x: 0, y: 0 };
// origin.x = 10; // Error: Cannot assign to readonly property

// Configuration object
const CONFIG: Readonly<{
  apiUrl: string;
  timeout: number;
  retries: number;
}> = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
};

// CONFIG.apiUrl = "new-url"; // Error

// Readonly array
const names: ReadonlyArray<string> = ["Alice", "Bob"];
// names.push("Charlie"); // Error
// names[0] = "Eve"; // Error
```

### Intermediate Examples

```typescript
// Deep Readonly
type DeepReadonly<T> = T extends object
  ? { readonly [P in keyof T]: DeepReadonly<T[P]> }
  : T;

interface Config {
  database: { host: string; port: number };
  server: { port: number; ssl: boolean };
}

const config: DeepReadonly<Config> = {
  database: { host: "localhost", port: 5432 },
  server: { port: 8080, ssl: true },
};

// config.database.host = "new-host"; // Error: deeply readonly

// Readonly in function parameters
function processConfig(config: Readonly<Config>): void {
  // config.server.port = 9000; // Error
}

// Readonly with class properties
class ImmutableState {
  constructor(public readonly state: Readonly<AppState>) {}
}

// Readonly as return type
function fetchConfig(): Readonly<Config> {
  return Object.freeze({ ...defaultConfig });
}
```

### Advanced Examples

```typescript
// Readonly mapped type with exceptions
type ReadonlyExcept<T, K extends keyof T> = {
  readonly [P in Exclude<keyof T, K>]: T[P];
} & { [P in K]: T[P] };

// Readonly with conditional types
type Immutable<T> = T extends Function
  ? T
  : T extends object
    ? { readonly [P in keyof T]: Immutable<T[P]> }
    : T;

// Readonly in generic constraints
function freeze<T extends object>(obj: T): Readonly<T> {
  return Object.freeze(obj);
}

const frozen = freeze({ a: 1, b: 2 });
// frozen.a = 3; // Error

// Combining Readonly with Required
type ImmutableConfig = Readonly<Required<Config>>;

// Readonly for tuple types
type ImmutableTuple = readonly [string, number, boolean];
const tuple: ImmutableTuple = ["hello", 42, true];
// tuple[0] = "world"; // Error
```

### Real-World Use Cases

- **Redux state**: The entire store is read-only; reducers return new objects.
- **Configuration objects**: Application config that should not change after initialization.
- **API responses**: Data received from APIs should not be mutated.
- **Constants**: Mathematical constants, lookup tables, enums.
- **Shared state**: Objects passed to multiple consumers that should not be modified.

```typescript
// Real-world: Redux-style state
interface AppState {
  readonly user: Readonly<User>;
  readonly todos: ReadonlyArray<Todo>;
  readonly ui: Readonly<UIState>;
}
```

### Common Mistakes

1. **Assuming `Readonly` provides runtime immutability**—it's compile-time only. Use `Object.freeze` for runtime.
2. **Using `Readonly` on shallow objects**—nested properties can still be mutated.
3. **Forgetting that `Readonly` applies to the property, not the value**—`Readonly<{ arr: number[] }>` prevents reassigning `arr` but allows `arr.push()`.
4. **Overusing readonly** when mutation is acceptable and desired.

### Best Practices

- Use `Readonly<T>` for function parameters that should not be modified.
- Use `DeepReadonly` for complete immutability guarantees.
- Combine with `Object.freeze` for runtime enforcement.
- Use `ReadonlyArray<T>` instead of `T[]` for immutable arrays.
- Prefer `readonly` in interface definitions for immutable data models.

### Performance Considerations

`Readonly` is compile-time only with no runtime cost. `Object.freeze` has a runtime cost and can impact performance for large objects.

### Interview Questions

1. **Q**: How is `Readonly<T>` different from using `const`?
   **A**: `const` prevents reassignment of the variable; `Readonly<T>` prevents reassignment of properties. `const obj = { a: 1 }; obj.a = 2;` is allowed. `Readonly<{ a: number }>` prevents `obj.a = 2`.

2. **Q**: Does `Readonly<T>` make nested properties readonly?
   **A**: No, only the top-level properties. Use `DeepReadonly` for nested immutability.

### Coding Challenges

1. Implement a `DeepReadonly` utility type.
2. Write a function that takes a `Readonly<Config>` and merges it with overrides (returning a new object).

### Related Topics

- `readonly` modifier
- `const`
- `Object.freeze`
- Immutability patterns

## Pick and Omit

### What It Is

`Pick<T, K>` creates a type with only the specified keys `K` from `T`. `Omit<T, K>` creates a type with all keys from `T` except those in `K`.

```typescript
type Pick<T, K extends keyof T> = { [P in K]: T[P] };
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

### Why It Is Important

These utility types are fundamental for type manipulation. `Pick` allows you to extract a subset of properties, which is useful for creating focused interfaces from larger ones. `Omit` allows you to exclude properties, which is useful for modifying types without changing the original.

### How It Works Internally

`Pick` uses a mapped type with a constrained key set. `Omit` is defined in terms of `Pick` and `Exclude`: it excludes the specified keys and picks the rest.

```typescript
// Pick<User, "name" | "email"> → { name: string; email: string }
// Omit<User, "password"> → { id: number; name: string; email: string }
```

### Syntax

```typescript
Pick<T, K extends keyof T>
Omit<T, K extends keyof any>
```

### Beginner Examples

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

// Pick: select specific fields
type PublicUser = Pick<User, "id" | "name" | "email">;
// { id: number; name: string; email: string }

// Omit: exclude specific fields
type UserWithoutPassword = Omit<User, "password">;
// { id: number; name: string; email: string; createdAt: Date }

type UserCredentials = Pick<User, "email" | "password">;
// { email: string; password: string }

function createPublicUser(user: User): PublicUser {
  const { id, name, email } = user;
  return { id, name, email };
}

function createUser(data: Omit<User, "id" | "createdAt">): User {
  return { id: Date.now(), createdAt: new Date(), ...data };
}
```

### Intermediate Examples

```typescript
// Pick from nested type
interface APIResponse {
  data: { user: User; token: string };
  status: number;
  message: string;
}

type ResponseData = Pick<APIResponse, "data">;
// { data: { user: User; token: string } }

// Omit multiple keys
type UserWithoutTimestamps = Omit<User, "createdAt" | "updatedAt">;

// Pick with optional keys
type OptionalUserFields = Pick<User, "phone" | "avatar">;

// Omit with union of keys
type SensitiveFields = "password" | "ssn" | "creditCard";
type SafeUser = Omit<User, SensitiveFields>;

// Pick and Omit in generic
function selectFields<T, K extends keyof T>(obj: T, ...keys: K[]): Pick<T, K> {
  const result = {} as Pick<T, K>;
  for (const key of keys) {
    result[key] = obj[key];
  }
  return result;
}

const user = { id: 1, name: "Alice", email: "alice@test.com" };
const subset = selectFields(user, "id", "name"); // Pick<User, "id" | "name">
```

### Advanced Examples

```typescript
// Pick with conditional types
type FunctionProperties<T> = Pick<T, {
  [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T]>;

// Omit with deep filtering
type OmitByType<T, Type> = {
  [K in keyof T as T[K] extends Type ? never : K]: T[K];
};

type WithoutFunctions<T> = OmitByType<T, Function>;

// Pick + Partial for optional subset
type PartialPick<T, K extends keyof T> = Partial<Pick<T, K>> & Omit<T, K>;

interface Task {
  id: number;
  title: string;
  description: string;
  completed: boolean;
  dueDate: Date;
}

function updateTask(id: number, updates: PartialPick<Task, "title" | "description" | "completed">): void {
  // Only the picked fields are optional
}

// Omit + Required for required subset
type RequiredOmit<T, K extends keyof T> = Required<Omit<T, K>> & Pick<T, K>;

// Pick with template literal keys
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<Pick<User, "name" | "email">>;
// { getName: () => string; getEmail: () => string }

// Nested Pick with path
type NestedPick<T, Path extends string> = Path extends `${infer K}.${infer Rest}`
  ? K extends keyof T
    ? { [P in K]: NestedPick<T[K], Rest> }
    : never
  : Path extends keyof T
    ? Pick<T, Path>
    : never;
```

### Real-World Use Cases

- **API response shaping**: Excluding sensitive fields from responses.
- **Form state**: Picking only editable fields from a model.
- **DTOs**: Creating Data Transfer Objects with specific fields.
- **Component props**: Extracting specific props for sub-components.
- **Testing**: Creating mock objects with specific fields.
- **Permissions**: Generating views with field-level access control.

```typescript
// Real-world: API response cleanup
class UserController {
  async getUser(id: number): Promise<Pick<User, "id" | "name" | "email">> {
    const user = await db.findUser(id);
    return { id: user.id, name: user.name, email: user.email };
  }

  async createUser(data: Omit<User, "id" | "createdAt">): Promise<User> {
    return db.createUser(data);
  }
}
```

### Common Mistakes

1. **Omit on a type with index signature**: `Omit<{ [key: string]: number }, "specificKey">` may produce unexpected results.
2. **Omit with union types**: `Omit` works on object types, not union types.
3. **Not constraining `K` with `extends keyof T` in `Pick`**.
4. **Using `Pick` when you want `Required<Pick<T, K>>`**—picked properties keep their original optionality.

### Best Practices

- Use `Pick` to create focused, minimal interfaces from larger types.
- Use `Omit` to exclude fields rather than redefining the interface.
- Combine `Pick` and `Omit` with `Partial` and `Required` for fine-grained control.
- Prefer `Omit` over `Pick` when you're excluding fewer properties than you're including.
- Use descriptive type aliases for common pick/omit patterns.

### Performance Considerations

`Pick` and `Omit` are efficient utility types. Complex chains (`Omit<Pick<T, K1>, K2>`) add compile-time overhead but are generally fine.

### Interview Questions

1. **Q**: What is the difference between `Pick<T, K>` and `Omit<T, K>`?
   **A**: `Pick` includes only the specified keys; `Omit` includes all keys except the specified ones.

2. **Q**: How is `Omit` implemented?
   **A**: `type Omit<T, K> = Pick<T, Exclude<keyof T, K>>`

### Coding Challenges

1. Implement a `strictPick` that makes picked properties required.
2. Write a type-safe function that removes specified fields from an object.

### Related Topics

- `Exclude` and `Extract`
- `Partial` and `Required`
- Mapped types

## Record

### What It Is

`Record<K, T>` creates an object type with keys `K` and values `T`. It's a concise way to define dictionary-like types.

```typescript
type Record<K extends keyof any, T> = { [P in K]: T };
```

### Why It Is Important

`Record` is the go-to utility type for mapping one set of values to another. It's more concise than writing out the indexed type manually and clearly expresses intent. It's commonly used for lookup tables, caches, configuration maps, and enum-based property maps.

### How It Works Internally

`Record` uses a mapped type over the keys `K`. Each key `P in K` gets the value type `T`. If `K` is a union type, each member becomes a key.

```typescript
// Record<"a" | "b" | "c", number> = { a: number; b: number; c: number }
```

### Syntax

```typescript
Record<K, T>
// K: union of keys
// T: value type
```

### Beginner Examples

```typescript
// Simple record
type PageInfo = Record<string, string>;
const pages: PageInfo = { home: "/", about: "/about", contact: "/contact" };

// Record with literal keys
type UserRole = "admin" | "editor" | "viewer";
type RolePermissions = Record<UserRole, string[]>;

const permissions: RolePermissions = {
  admin: ["create", "read", "update", "delete"],
  editor: ["create", "read", "update"],
  viewer: ["read"],
};

// Record with number values
type StatusCode = Record<number, string>;
const httpStatuses: StatusCode = {
  200: "OK",
  404: "Not Found",
  500: "Internal Server Error",
};

// Record for enum mapping
enum Color {
  Red = "RED",
  Green = "GREEN",
  Blue = "BLUE",
}

type ColorHex = Record<Color, string>;
const hexValues: ColorHex = {
  [Color.Red]: "#FF0000",
  [Color.Green]: "#00FF00",
  [Color.Blue]: "#0000FF",
};
```

### Intermediate Examples

```typescript
// Record with constraints
function createRecord<K extends string, V>(keys: K[], value: V): Record<K, V> {
  const result = {} as Record<K, V>;
  keys.forEach(key => { result[key] = value; });
  return result;
}

const defaults = createRecord(["port", "host", "debug"], "not-set");
// defaults: Record<"port" | "host" | "debug", string>

// Record with complex values
type ConfigMap = Record<string, { enabled: boolean; value: string }>;
const config: ConfigMap = {
  featureX: { enabled: true, value: "on" },
  featureY: { enabled: false, value: "off" },
};

// Nested Record
type NestedRecord = Record<string, Record<string, number>>;
const matrix: NestedRecord = {
  row1: { col1: 1, col2: 2 },
  row2: { col1: 3, col2: 4 },
};

// Record with function values
type EventHandlers = Record<string, (...args: any[]) => void>;
const handlers: EventHandlers = {
  click: () => console.log("clicked"),
  focus: () => console.log("focused"),
};
```

### Advanced Examples

```typescript
// Record with template literal keys
type EventName = `on${Capitalize<string>}`;
type EventHandlerMap = Record<EventName, (e: Event) => void>;

// Record + Partial for optional values
type OptionalConfig = Partial<Record<string, string>>;

// Record in generic constraints
function mapValues<T, U>(obj: Record<string, T>, fn: (value: T) => U): Record<string, U> {
  const result: Record<string, U> = {};
  for (const key in obj) {
    result[key] = fn(obj[key]);
  }
  return result;
}

const lengths = mapValues({ a: "hello", b: "world" }, s => s.length);
// lengths: Record<string, number>

// Record with union discrimination
type HandlerMap = Record<string, (data: unknown) => void>;

function handleAll(handlers: HandlerMap, data: Record<string, unknown>): void {
  for (const key in data) {
    handlers[key]?.(data[key]);
  }
}

// Record with Readonly for constant maps
const HTTP_STATUSES: Readonly<Record<number, string>> = {
  200: "OK",
  201: "Created",
  400: "Bad Request",
  404: "Not Found",
  500: "Internal Server Error",
} as const;

// Record with Pick for partial mapping
type PartialRecord<K extends keyof any, T> = Partial<Record<K, T>>;
function createPartialRecord<K extends string, V>(keys: K[]): PartialRecord<K, V> {
  return {} as PartialRecord<K, V>;
}
```

### Real-World Use Cases

- **Lookup tables**: Error codes, status messages, translation maps.
- **Configuration maps**: Feature flags, environment configs.
- **Event handlers**: Mapping event names to handler functions.
- **Cache stores**: Key-value caches with typed values.
- **Permissions matrices**: Role-to-permission mappings.
- **Form state**: Key-value form field values.

```typescript
// Real-world: Error code lookup
type ErrorCode = "NOT_FOUND" | "UNAUTHORIZED" | "VALIDATION_ERROR" | "INTERNAL_ERROR";
type ErrorMessages = Record<ErrorCode, { message: string; status: number }>;

const errorMessages: ErrorMessages = {
  NOT_FOUND: { message: "Resource not found", status: 404 },
  UNAUTHORIZED: { message: "Unauthorized", status: 401 },
  VALIDATION_ERROR: { message: "Validation failed", status: 400 },
  INTERNAL_ERROR: { message: "Internal server error", status: 500 },
};
```

### Common Mistakes

1. **Using `Record<string, any>` too broadly**—defeats the purpose of typing.
2. **Not constraining `K`**: `Record` automatically constrains `K` to `string | number | symbol`.
3. **Using `Record` when a tuple or array would be more appropriate**.
4. **Forgetting that `Record` creates a type, not a runtime value**.

### Best Practices

- Use `Record<K, T>` for dictionary/map-like structures.
- Combine with `Partial` for optionally-present values.
- Combine with `Readonly` for constant maps.
- Use `as const` with `Record` for literal value types.
- Prefer `Record<string, T>` over `{ [key: string]: T }` for readability.

### Performance Considerations

`Record` is efficient. Large records with many keys (100+) slightly increase type-checking time but this is rarely an issue.

### Interview Questions

1. **Q**: What is the constraint on `K` in `Record<K, T>`?
   **A**: `K extends keyof any`, which is `string | number | symbol`.

2. **Q**: How is `Record<K, T>` different from `{ [key: string]: T }`?
   **A**: `Record<K, T>` works with any key union (not just string). It's also more readable and clearly expresses intent.

### Coding Challenges

1. Implement a type-safe `groupBy` function that returns a `Record<string, T[]>`.
2. Create a `Record`-based event system with typed event names and payloads.

### Related Topics

- Mapped types
- Index signatures
- `keyof` operator
- `Partial<Record<K, T>>`

## Exclude and Extract

### What It Is

`Exclude<T, U>` removes types from `T` that are assignable to `U`. `Extract<T, U>` extracts types from `T` that are assignable to `U`. Both work with union types.

```typescript
type Exclude<T, U> = T extends U ? never : T;
type Extract<T, U> = T extends U ? T : never;
```

### Why It Is Important

These utility types are fundamental for manipulating union types. `Exclude` is used to remove unwanted members from unions. `Extract` is used to filter unions to specific members. They are the building blocks for more complex conditional type transformations.

### How It Works Internally

Both use conditional types that distribute over unions. For each member of `T`, `Exclude` checks if it extends `U`. If so, it's replaced with `never` (removed). `Extract` does the opposite—only members extending `U` are kept.

```typescript
// Exclude<"a" | "b" | "c", "a" | "c"> = "b"
// Extract<"a" | "b" | "c", "a" | "c"> = "a" | "c"
```

### Syntax

```typescript
Exclude<T, U>
Extract<T, U>
```

### Beginner Examples

```typescript
// Exclude string literals
type Colors = "red" | "green" | "blue" | "yellow";
type PrimaryColors = Exclude<Colors, "yellow">; // "red" | "green" | "blue"

// Exclude types from union
type Mixed = string | number | boolean | null;
type StringOrNumber = Exclude<Mixed, boolean | null>; // string | number

// Extract specific types
type NumbersAndStrings = string | number;
type OnlyNumbers = Extract<NumbersAndStrings, number>; // number

// Extract by kind
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "triangle"; base: number; height: number };

type Circles = Extract<Shape, { kind: "circle" }>;
// { kind: "circle"; radius: number }

// Exclude null/undefined
type NonNullable<T> = Exclude<T, null | undefined>;
type Maybe = string | null | undefined;
type Definitely = NonNullable<Maybe>; // string
```

### Intermediate Examples

```typescript
// Exclude with function types
type Callbacks = (() => void) | ((x: number) => void) | string;
type OnlyFunctions = Exclude<Callbacks, string>; // (() => void) | ((x: number) => void)

// Extract with specific signatures
type EventHandlers = 
  | ((e: MouseEvent) => void)
  | ((e: KeyboardEvent) => void)
  | ((e: TouchEvent) => void);

type MouseHandlers = Extract<EventHandlers, (e: MouseEvent) => void>;
// ((e: MouseEvent) => void)

// Exclude with object types
type Base = { id: string; name: string };
type WithAge = { age: number };
type WithEmail = { email: string };
type Person = Base & (WithAge | WithEmail);

// Extract from generic parameter
function filterByType<T, U>(arr: T[], type: new (...args: any[]) => U): Extract<T, U>[] {
  return arr.filter((item): item is Extract<T, U> => item instanceof type);
}

// Exclude + keyof for property filtering
type WithoutStringValues<T> = {
  [K in Exclude<keyof T, { [P in keyof T]: T[P] extends string ? P : never }[keyof T]>]: T[K];
};
```

### Advanced Examples

```typescript
// Exclude from mapped type keys
type User = { id: number; name: string; email: string; password: string };
type NonSensitiveKeys = Exclude<keyof User, "password">;
// "id" | "name" | "email"

// Extract with conditional types
type ExtractPromise<T> = T extends Promise<infer U> ? U : never;

// Exclude by value type
type StringKeys<T> = Extract<keyof T, { [K in keyof T]: T[K] extends string ? K : never }[keyof T]>;
type UserStringKeys = StringKeys<User>; // "name" | "email"

// Exclude/Extract for discriminated unions
type Action =
  | { type: "FETCH_START" }
  | { type: "FETCH_SUCCESS"; data: unknown }
  | { type: "FETCH_ERROR"; error: Error };

type FetchActions = Extract<Action, { type: `FETCH_${string}` }>;
// All three actions

type SuccessActions = Extract<Action, { type: "FETCH_SUCCESS" }>;
// { type: "FETCH_SUCCESS"; data: unknown }

// Exclude with never
type RemoveNever<T> = T extends never ? never : T;

// Complex filtering
type FilterByType<T, Condition> = {
  [K in keyof T]: T[K] extends Condition ? T[K] : never;
}[keyof T];

// Using Extract for function overload filtering
type Overloaded = {
  (x: string): number;
  (x: number): string;
};
type StringOverload = Extract<Overloaded, (x: string) => any>;
```

### Real-World Use Cases

- **Action filtering**: Extract specific actions from a discriminated union.
- **Type cleanup**: Exclude `null` and `undefined` from types.
- **Key filtering**: Exclude specific keys from `keyof`.
- **API response handling**: Extract success/error types from response unions.
- **Configuration**: Filter configuration options by type.
- **Event handling**: Filter event handlers by event type.

```typescript
// Real-world: Redux action filtering
type AppAction =
  | { type: "USER_LOGIN"; payload: { userId: number } }
  | { type: "USER_LOGOUT" }
  | { type: "UPDATE_PROFILE"; payload: { name: string } }
  | { type: "DELETE_ACCOUNT" };

type UserActions = Extract<AppAction, { type: `USER_${string}` }>;
// { type: "USER_LOGIN"; ... } | { type: "USER_LOGOUT" }

type ActionsWithPayload = Extract<AppAction, { payload: unknown }>;
// { type: "USER_LOGIN"; payload: ... } | { type: "UPDATE_PROFILE"; payload: ... }
```

### Common Mistakes

1. **Using `Exclude` on non-union types**: It still works but may not do what you expect.
2. **Forgetting that `Exclude` and `Extract` distribute over unions**.
3. **Using `Exclude` with `never`**: `Exclude<never, T>` is `never` (empty set).
4. **Confusing `Exclude` (removes) with `Extract` (keeps)**.

### Best Practices

- Use `Exclude` to remove unwanted members from union types.
- Use `Extract` to filter unions to specific members.
- Combine with `keyof T` for property-level filtering.
- Use `NonNullable<T>` (built-in) instead of `Exclude<T, null | undefined>`.
- Prefer `Exclude`/`Extract` over manual conditional type distribution.

### Performance Considerations

Both are implemented as conditional types that distribute over unions. They are efficient for typical union sizes (up to dozens of members). Very large unions (100+ members) may impact compile time.

### Interview Questions

1. **Q**: How do `Exclude` and `Extract` handle union types?
   **A**: They distribute over unions. Each member of `T` is individually tested against `U`.

2. **Q**: What is the difference?
   **A**: `Exclude<T, U>` removes from `T` all types assignable to `U`. `Extract<T, U>` keeps only the types from `T` that are assignable to `U`.

### Coding Challenges

1. Implement `NonNullable<T>` using `Exclude`.
2. Write a type that extracts all function types from a union.

### Related Topics

- Conditional types
- Distributive conditional types
- `NonNullable`
- `keyof` operator

## NonNullable

### What It Is

`NonNullable<T>` removes `null` and `undefined` from a type `T`.

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;
```

### Why It Is Important

`NonNullable` is essential for cleaning up types after filtering or processing steps. It's commonly used in array filtering, data transformation pipelines, and any scenario where null/undefined values have been eliminated.

### How It Works Internally

It's implemented as `Exclude<T, null | undefined>`. The conditional type distributes over unions and removes any member that is `null`, `undefined`, or `null | undefined`.

```typescript
// NonNullable<string | null | undefined> = string
```

### Syntax

```typescript
NonNullable<T>
```

### Beginner Examples

```typescript
type Maybe = string | null | undefined;
type Definitely = NonNullable<Maybe>; // string

// Array filtering
const values: (string | null | undefined)[] = ["a", null, "b", undefined];
const filtered: string[] = values.filter((x): x is NonNullable<typeof x> => x != null);

// Function parameters
function process<T>(value: T): NonNullable<T> {
  if (value == null) throw new Error("Value is null");
  return value as NonNullable<T>;
}

const result = process<string | null>("hello"); // string

// Object properties
interface Config {
  host?: string;
  port?: number;
}
type NonNullableConfig = {
  [K in keyof Config]: NonNullable<Config[K]>;
};
```

### Intermediate Examples

```typescript
// NonNullable in generic constraints
function assertDefined<T>(value: T): asserts value is NonNullable<T> {
  if (value == null) throw new Error("Expected non-null value");
}

// NonNullable for promise resolution
async function firstDefined<T>(promises: Promise<T | null | undefined>[]): Promise<NonNullable<T>> {
  const results = await Promise.all(promises);
  const defined = results.find((x): x is NonNullable<T> => x != null);
  if (!defined) throw new Error("No defined value found");
  return defined;
}

// NonNullable in mapped types
type DefinedFields<T> = {
  [P in keyof T]-?: NonNullable<T[P]>;
};

// NonNullable with array methods
function compact<T>(arr: (T | null | undefined)[]): NonNullable<T>[] {
  return arr.filter((x): x is NonNullable<T> => x != null);
}

const cleaned = compact([1, null, 2, undefined, 3]); // number[]

// NonNullable for discriminated unions
type NullableResult<T> = { ok: true; value: T } | { ok: false };
type SuccessResult<T> = Extract<NullableResult<T>, { ok: true }>;
type ResultValue<T> = NonNullable<NullableResult<T>["value"]>;
```

### Advanced Examples

```typescript
// Deep NonNullable
type DeepNonNullable<T> = T extends object
  ? { [P in keyof T]: DeepNonNullable<NonNullable<T[P]>> }
  : NonNullable<T>;

interface DeepNullable {
  a: string | null;
  b: { c: number | null; d: string | undefined } | null;
}

type Clean = DeepNonNullable<DeepNullable>;
// { a: string; b: { c: number; d: string } }

// NonNullable with function types
type NonNullableFunction<T extends (...args: any[]) => any> = 
  (...args: Parameters<T>) => NonNullable<ReturnType<T>>;

function safe<T, R>(fn: (...args: T[]) => R | null | undefined): (...args: T[]) => NonNullable<R> {
  return (...args) => {
    const result = fn(...args);
    if (result == null) throw new Error("Function returned null");
    return result as NonNullable<R>;
  };
}

// NonNullable in object value extraction
type DefinedValues<T> = NonNullable<T[keyof T]>;

interface Data {
  name: string | null;
  age: number | undefined;
  email: string;
}

type AllDefined = DefinedValues<Data>; // string | number

// NonNullable with union filtering
type FilterNullish<T> = T extends null | undefined ? never : T;
type CleanUnion = FilterNullish<string | null | undefined | number>;
// string | number
```

### Real-World Use Cases

- **Array filtering**: Removing null/undefined from mixed arrays.
- **API response processing**: Handling optional fields after validation.
- **Function return types**: Ensuring functions return defined values.
- **Configuration defaults**: Resolving optional config to required config.
- **Database queries**: Transforming nullable database results to defined types.

```typescript
// Real-world: Database result transformation
interface DbUser {
  name: string | null;
  email: string | null;
  avatar_url: string | null;
}

interface CleanUser {
  name: NonNullable<DbUser["name"]>;
  email: NonNullable<DbUser["email"]>;
  avatar_url: NonNullable<DbUser["avatar_url"]>;
}

function cleanUser(dbUser: DbUser): CleanUser {
  return {
    name: dbUser.name ?? "Anonymous",
    email: dbUser.email!,
    avatar_url: dbUser.avatar_url ?? "/default-avatar.png",
  };
}
```

### Common Mistakes

1. **Using `NonNullable` after a runtime null check that already narrows the type**.
2. **Forgetting that `NonNullable` only removes `null` and `undefined`**, not other falsy values.
3. **Using `NonNullable` on non-union types** (no effect).
4. **Not using `NonNullable` with `Array.filter` type predicates**.

### Best Practices

- Use `NonNullable` as the return type for filtering functions.
- Combine with `Array.filter` for type-safe array cleaning.
- Use in generic constraints to guarantee non-null values.
- Prefer over manual `Exclude<T, null | undefined>`.
- Use `NonNullable` to document that a value has been validated as non-null.

### Performance Considerations

`NonNullable` is a simple conditional type and is very efficient.

### Interview Questions

1. **Q**: How is `NonNullable<T>` implemented?
   **A**: `type NonNullable<T> = T extends null | undefined ? never : T;`

2. **Q**: What is the difference between `NonNullable<T>` and `Required<T>`?
   **A**: `NonNullable` removes `null`/`undefined` from a type union. `Required` removes the optional `?` modifier from properties.

### Coding Challenges

1. Write a `compact` function that returns `NonNullable<T>[]` from a nullable array.
2. Implement `DeepNonNullable<T>` that recursively removes null/undefined.

### Related Topics

- `Exclude`
- Conditional types
- Null and undefined types
- Strict null checks
