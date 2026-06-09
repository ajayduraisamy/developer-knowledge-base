# Type Assertions - as syntax, angle bracket syntax, non-null assertion, const assertions

## Introduction

Type assertions are a mechanism in TypeScript to tell the compiler that you know more about the type of a value than it can infer. Unlike type conversions in other languages, type assertions are purely a compile-time construct—they are erased during compilation and have no runtime behavior. Assertions are essential for working with dynamic data, third-party libraries, and situations where the compiler cannot determine the correct type.

TypeScript provides several forms of assertions: the `as` syntax, angle bracket syntax, non-null assertion (`!`), and `const` assertions (`as const`). Each serves a different purpose and has distinct use cases and safety characteristics.

## as syntax

### What It Is

The `as` syntax is the primary and recommended way to perform type assertions in TypeScript. It tells the compiler to treat a value as a specific type. The syntax is `value as Type`.

```typescript
const value = someFunction() as string;
const input = document.getElementById("root") as HTMLDivElement;
```

### Why It Is Important

Type assertions are unavoidable when working with DOM APIs, JSON parsing, third-party libraries, or any dynamically typed data. The `as` syntax provides an escape hatch from the type system when you have information the compiler doesn't. It is safer than `any` because it at least specifies a target type.

### How It Works Internally

Assertions tell the compiler to ignore its structural compatibility check in one direction. Specifically, `value as T` is allowed when either `T` is assignable to `typeof value` or `typeof value` is assignable to `T` (or both). This prevents completely unrelated type assertions like `42 as string`. The assertion is erased at compile time—no code is generated.

```typescript
// Compile-time only. Generated JS:
// const value = someFunction();
// const input = document.getElementById("root");
```

### Syntax

```typescript
// Basic assertion
const value = getData() as string;

// DOM assertions
const canvas = document.querySelector("canvas") as HTMLCanvasElement;
const input = document.getElementById("email") as HTMLInputElement;

// JSON parsing
const data = JSON.parse(jsonString) as { name: string; age: number };

// Assertion with object literals
const config = { port: 3000, host: "localhost" } as Config;

// Double assertion (unsafe)
const num = (42 as unknown) as string;

// Assertion in JSX (only 'as' syntax works)
const element = <HTMLDivElement>document.getElementById("root"); // Error in .tsx
const element2 = document.getElementById("root") as HTMLDivElement; // OK
```

### Beginner Examples

```typescript
// DOM element selection
const button = document.querySelector(".submit") as HTMLButtonElement;
button.disabled = true;

// JSON parsing
interface User {
  id: number;
  name: string;
  email: string;
}

const response = await fetch("/api/users/1");
const user = (await response.json()) as User;
console.log(user.name);

// Event target
document.addEventListener("click", (e: Event) => {
  const target = e.target as HTMLElement;
  target.style.backgroundColor = "red";
});

// String to enum
enum Direction {
  North = "NORTH",
  South = "SOUTH",
}
const dir = "NORTH" as Direction;
```

### Intermediate Examples

```typescript
// Asserting union narrowings
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function createShape(kind: string): Shape {
  switch (kind) {
    case "circle":
      return { kind: "circle", radius: 10 } as Shape;
    case "square":
      return { kind: "square", side: 10 } as Shape;
    default:
      throw new Error("Unknown shape");
  }
}

// Asserting function parameter types
function processValue(value: unknown): string {
  return (value as { toString: () => string }).toString();
}

// Asserting indexed access
interface StringMap {
  [key: string]: unknown;
}

function getString(map: StringMap, key: string): string {
  return map[key] as string;
}

// Asserting for type narrowing
type Animal = { type: "dog"; bark: () => void } | { type: "cat"; meow: () => void };

function handleAnimal(animal: Animal): void {
  if (animal.type === "dog") {
    (animal as { type: "dog"; bark: () => void }).bark();
  }
}

// Assertion with intersection
const merged = { ...defaults, ...overrides } as Required<Config>;
```

### Advanced Examples

```typescript
// Assertion for branded types
type Brand<T, B> = T & { __brand: B };
type UserId = Brand<string, "UserId">;
type PostId = Brand<string, "PostId">;

function createUserId(id: string): UserId {
  return id as UserId;
}

function getUser(id: UserId): void {}
function getPost(id: PostId): void {}

const uid = createUserId("user_123");
getUser(uid);
// getPost(uid); // Error: type mismatch

// Asserting complex generics
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Asserting with conditional types
type IsString<T> = T extends string ? true : false;
function checkString<T>(value: T): IsString<T> {
  return (typeof value === "string") as IsString<T>;
}

// Assertion for discriminated unions in maps
type HandlerMap = {
  click: (e: MouseEvent) => void;
  keydown: (e: KeyboardEvent) => void;
  focus: () => void;
};

function createHandler<K extends keyof HandlerMap>(
  event: K,
  handler: HandlerMap[K]
): void {}

// Type assertion with Proxy
function createProxy<T extends object>(target: T): T {
  return new Proxy(target, {
    get(obj, prop) {
      return (obj as any)[prop];
    },
  }) as T;
}

// Asserting function types
type AsyncFn = (...args: any[]) => Promise<any>;
const myFn = (async () => 42) as AsyncFn;

// Nested assertions
const data = JSON.parse(jsonString) as {
  users: Array<{ id: number; name: string }>;
  meta: { total: number; page: number };
};
```

### Real-World Use Cases

- **DOM manipulation**: Casting `document.getElementById` results to specific element types.
- **JSON deserialization**: Typing parsed JSON into interfaces.
- **Third-party library interop**: When library types are missing or incorrect.
- **Testing mocks**: Casting mock objects to their typed interfaces.
- **Migration from JS**: Temporarily asserting types during migration.
- **Form data access**: Typing `FormData` and `EventTarget` values.

```typescript
// Real-world: Form handling
const form = document.querySelector("form") as HTMLFormElement;
form.addEventListener("submit", (e) => {
  e.preventDefault();
  const formData = new FormData(form);
  const email = (formData.get("email") as string);
  const age = parseInt(formData.get("age") as string, 10);
});
```

### Common Mistakes

1. **Overusing assertions**: Using `as` when proper typing or narrowing would suffice.
2. **Asserting unrelated types**: `number as string` is not allowed without double assertion.
3. **Using assertions to silence real type errors**: This defeats the purpose of TypeScript.
4. **Asserting non-null without checking**: Prefer optional chaining or null checks.
5. **Using `as any` as a crutch**: This disables type checking entirely.

```typescript
// Mistake: silencing real errors
function process(input: string | number) {
  (input as string).toUpperCase(); // Unsafe! Could be number
}

// Better: narrow properly
function process2(input: string | number) {
  if (typeof input === "string") {
    input.toUpperCase();
  }
}
```

### Best Practices

- Prefer type narrowing over assertions whenever possible.
- Use assertions only when you have information the compiler doesn't.
- Avoid `as any`—use more specific assertions.
- Prefer the `as` syntax over angle brackets, especially in `.tsx` files.
- Document why the assertion is safe with a comment.
- Use parsing/validation functions instead of raw assertions for untrusted data.

### Performance Considerations

Type assertions have zero runtime cost—they are completely erased during compilation. They do not generate any JavaScript code or affect performance.

### Interview Questions

1. **Q**: What is the difference between type assertion and type casting?
   **A**: Type assertion is compile-time only in TypeScript. Type casting (in other languages) has runtime behavior. TypeScript assertions are erased.

2. **Q**: When is `value as T` not allowed?
   **A**: When neither type is assignable to the other. For example, `42 as string` is not allowed because `number` and `string` are unrelated.

3. **Q**: What is a double assertion and when is it needed?
   **A**: `value as unknown as T`. It's used when the types are completely unrelated. It's unsafe and should be avoided.

### Coding Challenges

1. Write a type-safe JSON parser that uses assertions but validates the structure first.
2. Implement a DOM query wrapper that returns strongly-typed elements without assertions.

### Related Topics

- Type narrowing
- `unknown` type
- `as const`
- Non-null assertion

## Angle bracket syntax (<T>)

### What It Is

The angle bracket syntax (`<Type>value`) is an alternative form of type assertion. It predates the `as` syntax but is functionally identical. It cannot be used in `.tsx` files (React JSX) because it conflicts with JSX syntax.

```typescript
const value = <string>someFunction();
const input = <HTMLInputElement>document.getElementById("email");
```

### Why It Is Important

Some developers prefer the angle bracket syntax for readability, especially those from C# or Java backgrounds. Understanding it is important for reading legacy TypeScript code. However, `as` is the recommended syntax in modern TypeScript.

### How It Works Internally

Identical to `as` syntax. The compiler treats `<T>value` and `value as T` identically. The generated JavaScript is the same.

### Syntax

```typescript
// Basic angle bracket assertion
const length = (<string>value).length;

// DOM assertions
const canvas = <HTMLCanvasElement>document.querySelector("canvas");

// Object assertion
const config = <Config>{ port: 3000 };

// Limitations: cannot be used in .tsx files
// const element = <MyComponent />; // This is JSX, not assertion
```

### Beginner Examples

```typescript
interface Product {
  id: number;
  name: string;
  price: number;
}

const data = JSON.parse('{"id": 1, "name": "Widget", "price": 9.99}');
const product = <Product>data;
console.log(product.name);

// DOM example
const header = <HTMLHeadingElement>document.querySelector("h1");
header.textContent = "New Title";

// Event target
document.addEventListener("click", (e: Event) => {
  const element = <HTMLElement>e.target;
  element.classList.add("clicked");
});
```

### Intermediate Examples

```typescript
// With generics
function getValue<T>(key: string): T {
  return <T>{ /* fetch logic */ };
}

const name = getValue<string>("name");

// Array typing
const items = <Array<{ id: number }>>JSON.parse(jsonString);

// Function type assertion
const handler = <(x: number) => string>((x: number) => x.toString());

// Complex nesting
const appConfig = <Config>(<unknown>{
  apiUrl: "https://api.example.com",
  timeout: 5000,
});
```

### Advanced Examples

```typescript
// Angle bracket with intersection types
const merged = <TypeA & TypeB>{ ...objA, ...objB };

// With conditional types
type Result<T> = T extends string ? StringHandler : NumberHandler;
const handler = <Result<"string">>createHandler();

// In generic constraints
function factory<T extends new (...args: any[]) => any>(
  ctor: T,
  ...args: ConstructorParameters<T>
): InstanceType<T> {
  return <InstanceType<T>>new ctor(...args);
}

// With mapped types
type Readonly<T> = { readonly [P in keyof T]: T[P] };
const config = <Readonly<Config>>{ port: 3000, host: "localhost" };
```

### Real-World Use Cases

- **Legacy codebases**: Older TypeScript projects use angle bracket syntax extensively.
- **Non-JSX projects**: Backend Node.js projects where `.tsx` is not used.
- **Personal preference**: Some teams prefer the angle bracket style.

### Common Mistakes

1. **Using angle brackets in `.tsx` files**: Causes parser errors (confused with JSX tags).
2. **Assuming angle brackets are type parameters**: `<T>value` is an assertion; `value<T>` is a generic call.
3. **Nesting angle brackets**: `<<Type>value>` looks confusing and is unnecessary.

### Best Practices

- Use `as` syntax in `.tsx` files (always required).
- Use `as` syntax for consistency in new code.
- Be able to read angle bracket syntax for legacy code maintenance.
- Don't mix both styles in the same codebase.

### Performance Considerations

Identical to `as` syntax—zero runtime cost.

### Interview Questions

1. **Q**: Why can't you use angle bracket assertions in `.tsx` files?
   **A**: Because `<Type>value` is ambiguous with JSX tags. The compiler interprets it as JSX.

2. **Q**: Is there any functional difference between `<T>value` and `value as T`?
   **A**: No. They are semantically identical. The choice is purely stylistic.

### Coding Challenges

1. Migrate a legacy codebase from angle bracket syntax to `as` syntax.
2. Write a codemod that converts angle bracket assertions to `as` syntax.

### Related Topics

- `as` syntax
- JSX/TSX
- Type assertions

## Non-null assertion (!)

### What It Is

The non-null assertion operator (`!`) is a postfix expression that tells TypeScript to exclude `null` and `undefined` from the type of a value. It does not perform any runtime check—it is purely a compile-time assertion.

```typescript
const name = user!.name; // Asserts user is not null
element!.style.display = "none";
```

### Why It Is Important

The non-null assertion is useful when you know through program logic that a value cannot be `null` or `undefined` at a specific point, but TypeScript cannot infer this. Common scenarios include DOM element existence after guard checks, values set in callbacks, and post-initialization guarantees.

### How It Works Internally

The compiler removes `null` and `undefined` from the type. If `x` is `string | null | undefined`, `x!` is typed as `string`. No JavaScript code is generated—the `!` is erased.

```typescript
// TypeScript: x! → removes null/undefined
// JavaScript: x → no change
```

### Syntax

```typescript
// Non-null assertion on variables
value!.property
value!.method()

// Non-null assertion on function calls
getValue()!.toString()

// Non-null assertion on optional chaining
obj?.prop! // Note: the ! applies after the optional chain

// Non-null assertion in destructuring
const { name! } = obj;

// Non-null assertion with definite assignment
class Example {
  value!: string; // Assigned outside constructor
}
```

### Beginner Examples

```typescript
// DOM element (we know it exists)
const root = document.getElementById("root")!;
root.innerHTML = "<h1>Hello</h1>";

// Array access (we know index exists)
const first = arr[0]!;
console.log(first.toUpperCase());

// After null check (redundant but sometimes used)
function process(value: string | null): void {
  if (value === null) return;
  const upper = value!.toUpperCase(); // value is already narrowed
}

// Environment variables
const apiKey = process.env.API_KEY!;

// Callback parameter
function setup(callback?: () => void): void {
  callback!(); // We know callback is provided
}
```

### Intermediate Examples

```typescript
// Definite assignment assertion
class UserManager {
  private user!: User; // Assigned in init method, not constructor

  init(userData: unknown): void {
    this.user = parseUser(userData);
  }

  getName(): string {
    return this.user.name; // Safe after init is called
  }
}

// Post guard assertion
function findUser(id: string): User | undefined {
  // Fetch logic
}

function displayUser(id: string): void {
  const user = findUser(id);
  if (!user) throw new Error("User not found");
  // user is narrowed, but ! is sometimes used for emphasis:
  const name = user!.name;
}

// Array filter + assertion
const items: (string | null)[] = ["a", null, "b"];
const filtered: string[] = items.filter(Boolean) as string[];
// Or with non-null assertion:
const mapped = items.map(x => x!);

// Conditional non-null
function getFirst<T>(arr: T[]): T | undefined {
  return arr.length > 0 ? arr[0]! : undefined;
}
```

### Advanced Examples

```typescript
// Non-null assertion with mapped types
type NonNullProperties<T> = {
  [P in keyof T]-?: NonNullable<T[P]>;
};

function ensureNonNull<T extends object>(obj: T): NonNullProperties<T> {
  return obj as NonNullProperties<T>;
}

// Non-null assertion in generic context
function assertDefined<T>(value: T | null | undefined): T {
  if (value == null) throw new Error("Unexpected null/undefined");
  return value!; // Redundant because of narrowing
}

// Non-null in promise chains
fetch("/api/data")
  .then(r => r.json())
  .then(data => data.users!)
  .then(users => users.forEach(u => console.log(u.name)));

// Class with delayed initialization
class LazyLoader<T> {
  private _value!: T;
  private loaded = false;

  load(value: T): void {
    this._value = value;
    this.loaded = true;
  }

  get value(): T {
    if (!this.loaded) throw new Error("Not loaded");
    return this._value!;
  }
}

// Non-null assertion with intersection types
type NonNull<T> = T & {};
const value: string | null = getValue();
const safe: string = value!;
// The intersection with {} removes null/undefined
```

### Real-World Use Cases

- **Angular/React lifecycle**: Values set in lifecycle methods (e.g., `ngOnInit`, `useEffect`) but not in the constructor.
- **DOM references**: Elements that are guaranteed to exist in the DOM.
- **Testing**: Mock values that are always provided.
- **Configuration**: Values loaded at startup that are always present after initialization.
- **Event handlers**: Event object properties that are present for specific event types.

```typescript
// Real-world: React refs
function MyComponent() {
  const inputRef = useRef<HTMLInputElement>(null!);

  useEffect(() => {
    inputRef.current!.focus(); // We know it's mounted
  }, []);

  return <input ref={inputRef} />;
}

// Real-world: Angular lifecycle
class AppComponent implements OnInit {
  data!: Data[]; // Set in ngOnInit

  ngOnInit(): void {
    this.data = this.service.getData();
  }
}
```

### Common Mistakes

1. **Overusing `!` instead of proper null checking**: `user!.name` throws if `user` is null.
2. **Using `!` in TypeScript's strict null check mode without justification**.
3. **Forgetting that `!` does not provide runtime safety**: It's just a compile-time hint.
4. **Using `!` on optional properties that might genuinely be `undefined`**.
5. **Confusing `!` with `?` (optional chaining)**: `x?.prop` is safe; `x!.prop` is not.

```typescript
// Bad: assumes always non-null
function getLength(value?: string): number {
  return value!.length; // Runtime error if undefined
}

// Good: handle null/undefined properly
function getLengthSafe(value?: string): number {
  return value?.length ?? 0;
}
```

### Best Practices

- Only use `!` when you are certain the value is non-null and have programmatic proof.
- Prefer optional chaining (`?.`) and nullish coalescing (`??`) over `!`.
- Use `!` sparingly—each usage is a potential runtime error.
- Document why the assertion is safe.
- In classes, consider if the definite assignment assertion is truly needed or if the design should change.
- For DOM elements, check existence first or use optional chaining.

### Performance Considerations

Zero runtime cost. The `!` operator is erased at compile time.

### Interview Questions

1. **Q**: What does `value!` do in TypeScript?
   **A**: It's a non-null assertion that removes `null` and `undefined` from the type of `value`. It has no runtime effect.

2. **Q**: How is `!` different from `?.`?
   **A**: `?.` (optional chaining) short-circuits and returns `undefined` if the value is nullish. `!` asserts the value is non-null and performs no runtime check.

3. **Q**: What is the definite assignment assertion (`!:`)?
   **A**: It tells TypeScript that a class property will be assigned before use, even if not in the constructor. Example: `name!: string`.

### Coding Challenges

1. Refactor a codebase that heavily uses `!` to use proper null handling patterns.
2. Implement a class with delayed initialization that safely avoids `!` assertions.

### Related Topics

- Strict null checks
- Optional chaining
- Nullish coalescing
- Definite assignment

## const assertions

### What It Is

A `const` assertion (`as const`) tells TypeScript to infer the narrowest possible type for a value. For primitives, it infers literal types. For objects, it makes all properties `readonly` and infers literal property types. For arrays, it infers `readonly` tuples with literal element types.

```typescript
const x = "hello" as const; // x: "hello"
const y = { name: "Alice" } as const; // { readonly name: "Alice" }
const z = [1, 2, 3] as const; // readonly [1, 2, 3]
```

### Why It Is Important

`as const` enables precise type inference for constant values, especially in objects and arrays. It is essential for creating type-safe enum alternatives, defining route maps, configuration objects, and any scenario where literal types need to be preserved through object or array literals.

### How It Works Internally

The compiler applies three transformations: (1) literal types are inferred instead of widened types, (2) object properties become `readonly`, (3) arrays become `readonly` tuples. These changes are purely at the type level.

```typescript
// Without as const:
const a = { name: "Alice" }; // { name: string }
const b = [1, 2]; // number[]

// With as const:
const c = { name: "Alice" } as const; // { readonly name: "Alice" }
const d = [1, 2] as const; // readonly [1, 2]
```

### Syntax

```typescript
// Primitive as const
const x = 42 as const; // 42
const y = true as const; // true

// Object as const
const obj = { a: 1, b: "hello" } as const;
// { readonly a: 1; readonly b: "hello" }

// Array as const
const arr = [1, "two", true] as const;
// readonly [1, "two", true]

// Nested as const
const config = {
  api: { url: "https://api.example.com", timeout: 5000 },
} as const;
// { readonly api: { readonly url: "https://api.example.com"; readonly timeout: 5000 } }

// Mixed: partial as const
const partial = { a: "hello", b: [1, 2] as const };
// { a: string; b: readonly [1, 2] }
```

### Beginner Examples

```typescript
// Enum alternative
const Colors = {
  Red: "#FF0000",
  Green: "#00FF00",
  Blue: "#0000FF",
} as const;

type ColorName = keyof typeof Colors; // "Red" | "Green" | "Blue"
type ColorHex = typeof Colors[ColorName]; // "#FF0000" | "#00FF00" | "#0000FF"

function getColor(name: ColorName): ColorHex {
  return Colors[name];
}

// Tuple preservation
const point = [10, 20] as const;
// point: readonly [10, 20]
const x = point[0]; // 10 (not number)
const y = point[1]; // 20 (not number)

// Route definitions
const ROUTES = {
  HOME: "/",
  ABOUT: "/about",
  CONTACT: "/contact",
} as const;

type Route = typeof ROUTES[keyof typeof ROUTES];
// "/" | "/about" | "/contact"

function navigate(route: Route): void {
  window.location.href = route;
}
```

### Intermediate Examples

```typescript
// Type-safe event emitter
const Events = {
  UserCreated: "user:created",
  UserUpdated: "user:updated",
  UserDeleted: "user:deleted",
} as const;

type EventName = typeof Events[keyof typeof Events];
// "user:created" | "user:updated" | "user:deleted"

type EventPayload = {
  [Events.UserCreated]: { id: number; name: string };
  [Events.UserUpdated]: { id: number; changes: Record<string, unknown> };
  [Events.UserDeleted]: { id: number };
};

function emit<K extends EventName>(event: K, payload: EventPayload[K]): void {}

emit(Events.UserCreated, { id: 1, name: "Alice" });

// Configuration objects
const CONFIG = {
  development: { apiUrl: "http://localhost:3000", debug: true },
  production: { apiUrl: "https://api.example.com", debug: false },
  test: { apiUrl: "https://test-api.example.com", debug: true },
} as const;

type Environment = keyof typeof CONFIG; // "development" | "production" | "test"
type EnvConfig = typeof CONFIG[Environment];

function loadConfig(env: Environment): EnvConfig {
  return CONFIG[env];
}

// Discriminated union from const object
const ActionTypes = {
  AddTodo: "ADD_TODO",
  DeleteTodo: "DELETE_TODO",
  ToggleTodo: "TOGGLE_TODO",
} as const;

type Action =
  | { type: typeof ActionTypes.AddTodo; text: string }
  | { type: typeof ActionTypes.DeleteTodo; id: number }
  | { type: typeof ActionTypes.ToggleTodo; id: number };
```

### Advanced Examples

```typescript
// as const with function return types
function createConfig() {
  return {
    port: 3000,
    host: "localhost",
    ssl: false,
  } as const;
}

const config = createConfig();
// config: { readonly port: 3000; readonly host: "localhost"; readonly ssl: false }

// as const with template literals
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type Route = "/users" | "/posts" | "/comments";
type ApiCall = `${HttpMethod} ${Route}`;

const apiCalls = [
  "GET /users",
  "POST /users",
  "GET /posts",
] as const;

type ValidApiCall = typeof apiCalls[number];
// "GET /users" | "POST /users" | "GET /posts"

// Nested const assertions
const NestedConfig = {
  database: {
    host: "localhost",
    port: 5432,
    credentials: {
      user: "admin",
      password: "secret",
    },
  },
  server: {
    port: 8080,
    ssl: {
      enabled: true,
      cert: "/path/to/cert",
    },
  },
} as const;

type DeepConfig = typeof NestedConfig;
// All properties are deeply readonly with literal types

// as const with arrays for tuple types
const rgb = ["red", "green", "blue"] as const;
type RGB = typeof rgb[number]; // "red" | "green" | "blue"

const directions = ["north", "south", "east", "west"] as const;
type Direction = typeof directions[number]; // "north" | "south" | "east" | "west"

// Combining as const with satisfies
const Palette = {
  primary: "#007bff",
  secondary: "#6c757d",
  success: "#28a745",
} as const satisfies Record<string, `#${string}`>;

type PaletteColor = keyof typeof Palette; // "primary" | "secondary" | "success"
type PaletteValue = typeof Palette[PaletteColor]; // `#${string}`
```

### Real-World Use Cases

- **Redux action types**: Defining action type constants with inferred literal types.
- **Route maps**: Type-safe route strings for navigation.
- **Configuration objects**: Environment-specific configs with exact types.
- **Enum alternatives**: Object-based enums with full type safety.
- **API endpoint definitions**: Type-safe endpoint paths and methods.
- **Translation keys**: Type-safe i18n key definitions.
- **Color/font/theme definitions**: Design tokens with exact value types.

```typescript
// Real-world: Redux Toolkit-style actions
const CounterActions = {
  increment: "counter/increment",
  decrement: "counter/decrement",
  reset: "counter/reset",
} as const;

type CounterAction = {
  [K in keyof typeof CounterActions]: {
    type: typeof CounterActions[K];
    payload?: K extends "reset" ? undefined : number;
  };
}[keyof typeof CounterActions];

// Real-world: Type-safe API client
const API = {
  users: { list: "GET", create: "POST" },
  posts: { list: "GET", create: "POST", delete: "DELETE" },
} as const;

type APIEndpoints = {
  [Module in keyof typeof API]: {
    [Action in keyof typeof API[Module]]: (
      ...args: Action extends "delete" ? [id: number] : []
    ) => Promise<unknown>;
  };
};
```

### Common Mistakes

1. **Using `as const` on mutable data**: The resulting type is readonly, which may conflict with APIs that expect mutable types.
2. **Forgetting that `as const` makes everything deeply readonly**.
3. **Overusing `as const` when widening is acceptable**.
4. **Assuming `as const` works with class instances**—it's for literal values only.
5. **Not using `as const` when extracting union types from objects/arrays**.

### Best Practices

- Use `as const` on `const` objects to preserve literal types for all properties.
- Use `as const` with arrays when you need a tuple type with literal elements.
- Use `as const` for enum alternatives (object-based enums).
- Combine `as const` with `satisfies` for type validation + literal inference.
- Use `keyof typeof obj` and `typeof obj[key]` with `as const` objects for type-safe value unions.
- Avoid `as const` in library API boundaries if consumers need mutable types.

### Performance Considerations

`as const` adds no runtime cost—it is purely a compile-time construct. However, deeply `readonly` types with many literal properties can increase type-checking time slightly compared to widened types. The benefit of improved type safety usually outweighs this minor cost.

### Interview Questions

1. **Q**: What does `as const` do to an object literal?
   **A**: It makes all properties `readonly` and infers their types as literals instead of the widened base types.

2. **Q**: How do you create a union type from an `as const` array?
   **A**: Use `typeof arr[number]`. For example, `const arr = ["a", "b"] as const` gives you `"a" | "b"`.

3. **Q**: Can you use `as const` with a variable that is not a literal?
   **A**: No. `as const` only works with literal values (primitives, object literals, array literals). It cannot be used with variables or expressions that aren't immediately known.

### Coding Challenges

1. Build a type-safe i18n system using `as const` objects for translation keys.
2. Implement a Redux-style reducer using `as const` action types.
3. Create a type-safe event emitter with events defined in an `as const` object.

### Related Topics

- Literal types
- `readonly` modifier
- `satisfies` keyword
- `const` vs `as const`
- Type widening
