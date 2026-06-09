# Interview Questions - Core TS questions, advanced type challenges, real-world scenarios, system design

## Introduction

TypeScript interview questions test a candidate's understanding of the type system, practical experience with real-world patterns, and ability to design type-safe systems. This file covers core TypeScript concepts, advanced type challenges, real-world scenarios, and system design questions with TypeScript-specific considerations.

## Core TypeScript Concepts

### What It Is

Core TypeScript concepts include the fundamentals of the type system: primitive types, interfaces, type aliases, unions, intersections, enums, generics, type narrowing, and the differences between `type` and `interface`.

### Why It Is Important

Interviewers ask core concept questions to assess foundational knowledge. A strong grasp of fundamentals indicates the candidate can effectively use TypeScript in day-to-day development without constant documentation lookups.

### How It Works Internally

The TypeScript compiler processes each concept through its type-checking pipeline. Interfaces use structural type checking (duck typing) — compatibility is determined by shape, not declaration. This differs from nominal type systems in C#/Java. The compiler maintains a symbol table and performs type resolution through bidirectional inference during the binding and checking phases.

### Key Questions and Answers

**Q: What is the difference between `interface` and `type`?**

```typescript
// Interface can be extended/merged
interface User { name: string; }
interface User { age: number; } // Merged: User has name and age

// Type cannot be merged, but can use unions, intersections, mapped types
type Status = 'active' | 'inactive';
type AdminUser = User & { role: 'admin' };

// Use interface for public API types (open for extension)
```

**Q: What is the `unknown` type and how is it different from `any`?**

```typescript
let value: any;
value.foo(); // No error - any disables checking

let data: unknown;
// data.foo(); // Error: Object is of type 'unknown'
if (typeof data === 'string') {
  data.toUpperCase(); // OK - narrowed
}

// Use unknown when you don't know the type but want type safety
```

**Q: How does TypeScript's structural typing work?**

```typescript
interface Named { name: string; }
interface Person { name: string; age: number; }

const person: Person = { name: 'Alice', age: 30 };
const named: Named = person; // OK - Person has all Named properties

// TypeScript checks structure, not declaration
```

**Q: What are generics and when would you use them?**

```typescript
// Generic function
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

const num = first([1, 2, 3]); // number
const str = first(['a', 'b']); // string

// Use generics to create reusable components that work with multiple types
```

**Q: Explain type narrowing with examples.**

```typescript
function process(value: string | number | Date) {
  if (typeof value === 'string') {
    value.toUpperCase(); // narrowed to string
  } else if (value instanceof Date) {
    value.getTime(); // narrowed to Date
  } else {
    value.toFixed(2); // narrowed to number
  }
}
```

**Q: What are discriminated unions?**

```typescript
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
```

**Q: Explain the `keyof` and `typeof` operators.**

```typescript
interface User { name: string; age: number; email: string; }

type UserKeys = keyof User; // 'name' | 'age' | 'email'

const config = { host: 'localhost', port: 3000 };
type Config = typeof config; // { host: string; port: number }
```

**Q: What is the `satisfies` operator (TS 4.9+)?**

```typescript
type Color = 'red' | 'green' | 'blue';
type Colors = Record<string, Color>;

const palette = {
  primary: 'blue',
  secondary: 'green',
  // tertiary: 'yellow', // Error with satisfies
} satisfies Colors;

// palette.primary is typed as 'blue' (literal), not Color
```

**Q: Explain `readonly`, `Partial`, `Required`, `Pick`, `Omit`.**

```typescript
interface User { id: string; name: string; email: string; age?: number; }

type ReadonlyUser = Readonly<User>;
type PartialUser = Partial<User>;
type RequiredUser = Required<User>;
type PickUser = Pick<User, 'id' | 'name'>;
type OmitUser = Omit<User, 'email'>;
```

**Q: What is declaration merging and when is it useful?**

```typescript
// Interface declaration merging
interface Request { body: any; }
interface Request { headers: Record<string, string>; }
// Merged: Request has body and headers

// Useful for extending third-party types
declare module 'express' {
  interface Request {
    currentUser?: { id: string };
  }
}
```

### Coding Challenges

1. Write a generic function that returns the first element of an array, typed correctly.
2. Create a type-safe event emitter using generics and discriminated unions.

### Performance Considerations
- Complex conditional types increase compile times; profile with `tsc --generateTrace`
- Large unions cause exponential type-checking in worst-case scenarios
- Prefer mapped types over recursive conditional types for better compiler performance
- Use type tests (expect-type) to verify complex type utilities without runtime cost

### Interview Questions
1. How do you implement a type that extracts the return type of a function?
2. Explain how conditional types distribute over unions.
3. What is the `infer` keyword and how is it used in conditional types?
4. How would you create a deep `Readonly` type using mapped types?
5. Explain variadic tuple types with practical examples.

### Related Topics

- Advanced types
- Utility types
- Type manipulation

## Advanced Type Challenges

### What It Is

Advanced type challenges test a candidate's ability to work with conditional types, template literal types, mapped types, recursive types, and type-level programming.

### Why It Is Important

Advanced type questions differentiate experienced TypeScript developers. They assess the candidate's depth of understanding and ability to leverage the type system for real-world type safety.

### Key Questions and Answers

**Q: Implement a `DeepReadonly<T>` type.**

```typescript
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends Record<string, any>
    ? T[K] extends Function
      ? T[K]
      : DeepReadonly<T[K]>
    : T[K];
};

interface User {
  name: string;
  address: { city: string; zip: string };
}

type DeepUser = DeepReadonly<User>;
// { readonly name: string; readonly address: { readonly city: string; readonly zip: string } }
```

**Q: Create a `PickByValue<T, V>` type that picks properties whose values match `V`.**

```typescript
type PickByValue<T, V> = {
  [K in keyof T as T[K] extends V ? K : never]: T[K];
};

interface Example {
  id: number;
  name: string;
  active: boolean;
  age: number;
}

type NumbersOnly = PickByValue<Example, number>;
// { id: number; age: number }
```

**Q: Implement a type that extracts a property value by a dot-separated path.**

```typescript
type GetFieldType<T, P extends string> =
  P extends `${infer K}.${infer Rest}`
    ? K extends keyof T
      ? GetFieldType<T[K], Rest>
      : never
    : P extends keyof T
      ? T[P]
      : never;

interface Deep {
  user: { profile: { name: string; age: number } };
  meta: { version: string };
}

type Name = GetFieldType<Deep, 'user.profile.name'>; // string
type Version = GetFieldType<Deep, 'meta.version'>; // string
```

**Q: Create a `TupleToUnion<T>` type.**

```typescript
type TupleToUnion<T extends any[]> = T[number];

type Result = TupleToUnion<[string, number, boolean]>;
// string | number | boolean
```

**Q: Implement a `UnionToIntersection<T>` type.**

```typescript
type UnionToIntersection<U> =
  (U extends any ? (k: U) => void : never) extends
  (k: infer I) => void ? I : never;

type Test = UnionToIntersection<{ a: string } | { b: number }>;
// { a: string } & { b: number }
```

**Q: Write a `RequiredByKeys<T, K>` type.**

```typescript
type RequiredByKeys<T, K extends keyof T> = Omit<T, K> & Required<Pick<T, K>>;

interface User { id?: string; name?: string; email?: string; }

type UserWithRequiredId = RequiredByKeys<User, 'id'>;
// { id: string; name?: string; email?: string }
```

**Q: Create a type that converts camelCase to snake_case.**

```typescript
type CamelToSnake<S extends string> =
  S extends `${infer First}${infer Rest}`
    ? First extends Uppercase<First>
      ? `_${Lowercase<First>}${CamelToSnake<Rest>}`
      : `${First}${CamelToSnake<Rest>}`
    : S;

type Test = CamelToSnake<'camelCaseString'>;
// 'camel_case_string'
```

**Q: Implement a `DeepPartial<T>` that recursively makes properties optional.**

```typescript
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends Record<string, any>
    ? T[K] extends Function
      ? T[K]
      : DeepPartial<T[K]>
    : T[K];
};
```

**Q: Write a `Mutable<T>` type that removes readonly.**

```typescript
type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};

interface ReadonlyUser {
  readonly id: string;
  readonly name: string;
}

type MutableUser = Mutable<ReadonlyUser>;
// { id: string; name: string }
```

**Q: Create a type that extracts the return type of a function.**

```typescript
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type Fn = (x: number) => string;
type Result = MyReturnType<Fn>; // string
```

### Coding Challenges

1. Implement a type-safe `_.get(obj, path)` function using template literal types.
2. Create a validation type that ensures an object has no extra properties.

### Performance Considerations
- Conditional types with recursive inference can hit TypeScript's instantiation depth limit (50 by default, adjustable)
- Template literal types are parse-heavy; avoid extremely long string patterns
- Mapped types over large unions (1000+ members) degrade compile performance
- Use `type` tests in separate files to keep production code compilation fast

### Interview Questions
1. How do variadic tuple types improve type inference for function parameters?
2. Explain how to create a type that deeply converts all properties to optional.
3. What is the difference between `Exclude` and `Omit`?
4. How does TypeScript resolve contradictory conditional types?
5. Implement a `DeepPartial<T>` using mapped and conditional types.

### Related Topics

- Conditional types
- Template literal types
- Mapped types

## Real-World TypeScript Scenarios

### What It Is

Real-world scenario questions present practical problems that TypeScript developers face daily, such as migrating from JavaScript, typing third-party libraries, handling complex state, and building type-safe APIs.

### Why It Is Important

These questions assess practical experience and problem-solving ability. They reveal whether a candidate can apply TypeScript knowledge to real production challenges.

### Key Questions and Answers

**Q: How do you type a Redux slice with createSlice?**

```typescript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface CounterState {
  value: number;
  status: 'idle' | 'loading' | 'failed';
}

const initialState: CounterState = {
  value: 0,
  status: 'idle',
};

const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increment: (state) => { state.value += 1; },
    incrementByAmount: (state, action: PayloadAction<number>) => {
      state.value += action.payload;
    },
  },
});
```

**Q: How do you type an Express request with custom properties?**

```typescript
// types/express.d.ts
declare namespace Express {
  interface Request {
    currentUser?: { id: string; roles: string[] };
    requestId: string;
  }
}

// Middleware
app.use((req, res, next) => {
  req.requestId = uuid();
  // req.currentUser is now accessible
  next();
});
```

**Q: How do you type a React context safely?**

```typescript
interface AuthContextType {
  user: { id: string; name: string } | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

function useAuth(): AuthContextType {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
}
```

**Q: How do you type a custom React hook?**

```typescript
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T | ((prev: T) => T)) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    const item = localStorage.getItem(key);
    return item ? JSON.parse(item) : initialValue;
  });

  const setValue = (value: T | ((prev: T) => T)) => {
    const valueToStore = value instanceof Function ? value(storedValue) : value;
    setStoredValue(valueToStore);
    localStorage.setItem(key, JSON.stringify(valueToStore));
  };

  return [storedValue, setValue];
}
```

**Q: How do you type a generic API response?**

```typescript
interface ApiResponse<T> {
  data: T;
  meta?: { total: number; page: number; limit: number };
  error?: { message: string; code: string };
}

async function fetchApi<T>(url: string): Promise<ApiResponse<T>> {
  const response = await fetch(url);
  return response.json();
}

// Usage
const users = await fetchApi<User[]>('/api/users');
// users.data is typed as User[]
```

**Q: How do you migrate a JavaScript project to TypeScript?**

- Start with `allowJs: true` and `checkJs: false` for gradual migration.
- Add `// @ts-check` incrementally to individual files.
- Rename files from `.js` to `.ts` one by one.
- Use `any` types as a temporary escape hatch during migration.
- Enable strict mode gradually (`noImplicitAny`, `strictNullChecks`, etc.).
- Add type definitions for external libraries (`@types/*`).
- Use the migration tooling: `ts-migrate` for automated migration.

**Q: How do you handle third-party libraries without types?**

```typescript
// Create a declaration file
// declarations/untyped-library.d.ts
declare module 'untyped-library' {
  export function doSomething(input: string): Result;
  export interface Result { id: number; name: string; }
  export default function main(): void;
}

// Or use a wildcard declaration
declare module 'untyped-library' {
  const content: any;
  export default content;
}
```

**Q: How do you type a complex form state with validation?**

```typescript
type FormState<T extends Record<string, unknown>> = {
  [K in keyof T]: { value: T[K]; error: string | null; touched: boolean; };
};

type FormAction<T extends Record<string, unknown>> =
  | { type: 'SET_FIELD'; field: keyof T; value: T[keyof T] }
  | { type: 'SET_ERROR'; field: keyof T; error: string | null }
  | { type: 'TOUCH_FIELD'; field: keyof T }
  | { type: 'RESET'; state: FormState<T> };

function formReducer<T extends Record<string, unknown>>(
  state: FormState<T>,
  action: FormAction<T>
): FormState<T> {
  switch (action.type) {
    case 'SET_FIELD':
      return { ...state, [action.field]: { ...state[action.field], value: action.value, error: null } };
    case 'SET_ERROR':
      return { ...state, [action.field]: { ...state[action.field], error: action.error } };
    case 'TOUCH_FIELD':
      return { ...state, [action.field]: { ...state[action.field], touched: true } };
    case 'RESET':
      return action.state;
  }
}
```

**Q: How do you type a builder pattern in TypeScript?**

```typescript
class QueryBuilder<T extends Record<string, any>> {
  private conditions: string[] = [];

  where<K extends keyof T>(field: K, value: T[K]): this {
    this.conditions.push(`${String(field)} = ${JSON.stringify(value)}`);
    return this;
  }

  build(): string {
    return `SELECT * FROM table WHERE ${this.conditions.join(' AND ')}`;
  }
}
```

**Q: How do you handle exhaustive type checking?**

```typescript
type Color = 'red' | 'green' | 'blue';

function getHex(color: Color): string {
  switch (color) {
    case 'red': return '#FF0000';
    case 'green': return '#00FF00';
    case 'blue': return '#0000FF';
    default:
      const _exhaustive: never = color;
      throw new Error(`Unhandled color: ${color}`);
  }
}
```

### Coding Challenges

1. Type a WebSocket message handler that dispatches to different handlers based on message type.
2. Create a type-safe dependency injection container.

### Performance Considerations
- Type-safe Redux stores with deeply nested reducers increase type instantiation depth
- Zod validation schema types can generate large, complex inferred types
- Decorator-based validation (class-validator) adds reflection overhead at runtime
- Prefer plain TypeScript types over runtime validation libraries for compile-time-only concerns

### Interview Questions
1. How do you type a React component that accepts generically typed props?
2. How would you handle typing for a complex form state with dynamic fields?
3. Explain how to migrate a JavaScript project to TypeScript incrementally.
4. How do you type an API response that could have different shapes?
5. What strategies do you use for typing third-party libraries without type definitions?

### Related Topics

- Design patterns
- React TypeScript
- Express TypeScript

## TypeScript System Design

### What It Is

TypeScript system design questions assess a candidate's ability to architect large-scale TypeScript applications, including project structure, type strategy, build configuration, and team workflow considerations.

### Why It Is Important

System design questions for TypeScript evaluate architectural thinking and experience with production TypeScript at scale. They cover monorepo setup, incremental compilation, type coverage strategies, and integration with build pipelines.

### Key Questions and Answers

**Q: How would you structure a large monorepo TypeScript project?**

```json
// Root tsconfig.json with project references
{
  "files": [],
  "references": [
    { "path": "packages/types" },
    { "path": "packages/validation" },
    { "path": "packages/domain" },
    { "path": "packages/api" },
    { "path": "packages/web-app" },
    { "path": "packages/mobile-api" }
  ]
}
```

Structure:
```
packages/
  types/          - Shared interfaces and types
  validation/     - Zod schemas derived from types
  domain/         - Business logic (depends on types, validation)
  api/            - Express REST API (depends on domain)
  web-app/        - React frontend (depends on api client)
  mobile-api/     - Mobile backend (shares types with web)
```

**Q: How do you ensure type safety across a monorepo?**

- Use project references for strict dependency boundaries.
- Derive all types from a single source of truth (Zod schemas in `types`).
- Run `tsc --noEmit` in CI to catch type errors across all packages.
- Use `barrelsby` or manual index files to control public API surfaces.
- Enforce `strict: true` across all packages.
- Use ESLint with `@typescript-eslint` for additional type linting.

**Q: How would you design a type-safe API client?**

```typescript
// Shared types package
interface ApiEndpoint<TReq, TRes> {
  method: 'GET' | 'POST' | 'PUT' | 'DELETE';
  path: string;
  request: TReq;
  response: TRes;
}

// Type-safe API client
class ApiClient {
  async call<TReq, TRes>(
    endpoint: ApiEndpoint<TReq, TRes>,
    data?: TReq
  ): Promise<TRes> {
    const response = await fetch(endpoint.path, {
      method: endpoint.method,
      headers: { 'Content-Type': 'application/json' },
      body: data ? JSON.stringify(data) : undefined,
    });
    return response.json();
  }
}

// Endpoint definitions
const getUserEndpoint: ApiEndpoint<{ id: string }, User> = {
  method: 'GET',
  path: '/api/users/:id',
  request: { id: '' },
  response: { id: '', name: '', email: '' },
};
```

**Q: How do you approach type coverage and quality metrics?**

- Use `type-coverage` package to measure type coverage percentage.
- Track `any` usage with ESLint rules (`@typescript-eslint/no-explicit-any`).
- Monitor build times and set performance budgets.
- Use `ts-prune` to find unused exports and types.
- Set CI gates: no new `any` types, maintain >95% type coverage.
- Run `tsc --noEmit` as part of pre-commit hooks.

**Q: How would you design a type-safe state management system?**

```typescript
// Action types
type Action<T extends string, P = undefined> = {
  type: T;
  payload: P;
};

// Reducer with exhaustive checking
type Reducer<S, A extends { type: string }> = (state: S, action: A) => S;

// Generic store
class Store<S, A extends { type: string }> {
  private state: S;
  private reducer: Reducer<S, A>;

  constructor(reducer: Reducer<S, A>, initialState: S) {
    this.reducer = reducer;
    this.state = initialState;
  }

  dispatch(action: A): void {
    this.state = this.reducer(this.state, action);
  }

  getState(): Readonly<S> {
    return this.state;
  }
}

// Usage with discriminated unions
type CounterAction =
  | { type: 'INCREMENT'; payload?: undefined }
  | { type: 'DECREMENT'; payload?: undefined }
  | { type: 'SET'; payload: number };
```

**Q: How do you handle TypeScript in a microservices architecture?**

- Create a shared types package published as an npm package.
- Use protocol buffers or OpenAPI for cross-service contracts.
- Generate TypeScript types from protobuf definitions or OpenAPI specs.
- Each service has its own `tsconfig.json` with project references to shared packages.
- Use `tsc --build` for efficient incremental builds across services.
- Consider using GraphQL with code generation for strongly typed APIs.

**Q: How do you design a type-safe plugin system?**

```typescript
interface Plugin<TConfig = Record<string, unknown>> {
  name: string;
  version: string;
  initialize(config: TConfig): Promise<void>;
  hooks: Partial<PluginHooks>;
}

interface PluginHooks {
  beforeRequest: (req: Request) => Request | Promise<Request>;
  afterResponse: (res: Response) => Response | Promise<Response>;
  onError: (error: Error) => void;
}

class PluginManager {
  private plugins: Map<string, Plugin<any>> = new Map();

  register<TConfig>(plugin: Plugin<TConfig>, config: TConfig): void {
    this.plugins.set(plugin.name, plugin);
    plugin.initialize(config);
  }

  getPlugin<T extends Plugin<any>>(name: string): T | undefined {
    return this.plugins.get(name) as T | undefined;
  }
}
```

**Q: How do you structure types for a large team?**

- Establish a type convention document (style guide for types).
- Use consistent naming: `I` prefix for interfaces (optional), `T` prefix for generics, `Type` suffix for complex types.
- Create a shared types package with all domain types.
- Derive API types from Zod schemas (single source of truth).
- Use barrel exports (`index.ts`) for clean public APIs.
- Avoid circular dependencies with strict import rules.
- Use ESLint `import/no-cycle` to enforce dependency direction.

**Q: How do you scale TypeScript compilation for large projects?**

- Use project references to enable parallel builds.
- Enable `incremental: true` and cache `.tsbuildinfo` files.
- Use `skipLibCheck: true` to skip `node_modules` type checking.
- Use `ts-loader` with `transpileOnly: true` in webpack (type checking in separate process).
- Use `fork-ts-checker-webpack-plugin` for parallel type checking.
- Consider `swc` or `esbuild` for transpilation (without type checking).
- Use `isolatedModules: true` for faster single-file transpilation.
- Partition the codebase into independently buildable modules.

**Q: How do you handle TypeScript migration for a legacy codebase?**

```
1. Audit current codebase (JS files, dependencies, patterns)
2. Add tsconfig.json with allowJs: true, strict: false
3. Install @types for external libraries
4. Add type-checking file by file (rename .js to .ts)
5. Fix any type errors introduced
6. Enable strict options incrementally:
   a. noImplicitAny
   b. strictNullChecks
   c. strictFunctionTypes
   d. strictBindCallApply
   e. strictPropertyInitialization
   f. noImplicitThis
   g. alwaysStrict
7. Remove allowJs when migration is complete
8. Add CI checks to prevent new JS files
```

### Key System Design Principles

1. **Single source of truth**: Derive types from a single definition (Zod schemas, protobuf).
2. **Strict boundaries**: Use project references to enforce package dependencies.
3. **Incremental adoption**: Enable strict mode gradually.
4. **Performance first**: Configure build for speed; use project references and incremental builds.
5. **Tooling integration**: ESLint, Prettier, Husky, CI type checking.
6. **Type coverage**: Measure and enforce type coverage metrics.
7. **Documentation**: TypeDoc or inline documentation for public APIs.
8. **Testing**: Use typed testing utilities (ts-jest, vitest).

### Performance Considerations
- Project references dramatically reduce full recompilation times in monorepos
- Incremental builds use `.tsbuildinfo` to skip unchanged files — essential for CI pipelines
- Type coverage enforcement with `type-coverage` package adds build time; run as a separate CI step
- Monorepo design affects compilation strategy: build orchestrators (Nx, Turborepo) add overhead but enable caching

### Interview Questions
1. How would you structure a large TypeScript monorepo for performance?
2. Explain how project references improve build times in a multi-package architecture.
3. How do you handle type sharing between frontend and backend in a full-stack TypeScript app?
4. What considerations would you make for TypeScript in a microservices architecture?
5. How do you enforce type safety across service boundaries in distributed systems?

### Coding Challenges

1. Design a type-safe event sourcing system with event types derived from a schema registry.
2. Architect a monorepo with three packages (types, server, client) with project references and incremental builds.

### Related Topics

- Project references
- Monorepo management
- Build performance
