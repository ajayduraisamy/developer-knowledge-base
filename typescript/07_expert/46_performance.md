# Performance - Project references, incremental builds, type-checking performance, const assertions, avoid any

## Introduction

TypeScript performance is crucial for developer productivity, especially in large codebases. Slow compilation and type checking can significantly impact iteration speed. This file covers project references for modular build, incremental builds for faster recompilation, type-checking performance optimization techniques, the role of const assertions, and strategies for avoiding implicit any.

## Project References

### What It Is

Project references are a TypeScript feature that allows dividing a large codebase into smaller, independently compilable projects. Each project can reference other projects, and TypeScript only rebuilds changed projects and their dependents.

### Why It Is Important

Project references dramatically reduce build times in monorepos and large applications by enabling incremental compilation at the project level. They also enforce clear dependency boundaries between code modules.

### How It Works Internally

Each referenced project has its own `tsconfig.json` with `composite: true`. The parent project uses `references` to declare dependencies. TypeScript builds declaration files (`.d.ts`) for referenced projects, which are used for type checking in dependent projects without re-parsing the source.

### Syntax

```json
// packages/core/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src"]
}

// packages/app/tsconfig.json
{
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "references": [
    { "path": "../core" },
    { "path": "../shared" }
  ],
  "include": ["src"]
}
```

### Beginner Examples

```bash
# Build in dependency order
tsc --build packages/core
tsc --build packages/shared
tsc --build packages/app

# Or build all at once (handles dependency ordering)
tsc --build packages/app
```

### Intermediate Examples

```json
// Root tsconfig.json for the monorepo
{
  "files": [],
  "references": [
    { "path": "packages/core" },
    { "path": "packages/shared" },
    { "path": "packages/utils" },
    { "path": "packages/app" }
  ]
}
```

```bash
# Build all projects
tsc --build

# Force rebuild all
tsc --build --force

# Clean all builds
tsc --build --clean

# Watch mode
tsc --build --watch
```

### Advanced Examples

```typescript
// packages/core/src/types.ts
export interface User {
  id: string;
  name: string;
}

// packages/app/src/index.ts
import { User } from '@myorg/core';
// TypeScript resolves this from core's .d.ts output, not the source .ts

const user: User = { id: '1', name: 'Alice' };
```

### Real-World Use Cases

```json
// Monorepo structure:
// packages/
//   types/     - shared type definitions
//   validation - validation logic
//   utils/     - utility functions
//   api/       - API client (depends on types, validation)
//   app/       - main application (depends on all)

// Root tsconfig.json
{
  "files": [],
  "references": [
    { "path": "packages/types" },
    { "path": "packages/validation" },
    { "path": "packages/utils" },
    { "path": "packages/api" },
    { "path": "packages/app" }
  ]
}
```

### Common Mistakes

```typescript
// MISTAKE: Circular references between projects
// packages/a references packages/b, and b references a

// MISTAKE: Not setting composite: true in referenced projects
// Without composite, referenced projects can't be used

// MISTAKE: Importing from source files of referenced projects
// Always import from the project's main export
```

### Best Practices

- Set `composite: true` in all referenced projects.
- Set `declaration: true` and `declarationMap: true` for referenced projects.
- Define a root `tsconfig.json` with references to all projects.
- Use `tsc --build` for building (handles dependency ordering).
- Use `tsc --build --watch` for development.
- Keep referenced projects focused and small.

### Performance Considerations

Project references provide the largest performance gains for large codebases by enabling parallel builds and avoiding re-parsing of unchanged dependencies. The initial build is slightly slower (generating declarations), but incremental builds are much faster.

### Interview Questions

1. What are TypeScript project references and when should you use them?
2. How do project references improve build performance?
3. What is the `composite` option and why is it required?

### Coding Challenges

1. Set up a monorepo with two projects: a shared types library and an application that references it.
2. Configure incremental builds for a project with multiple references.

### Related Topics

- Incremental builds
- tsconfig.json
- Monorepo setup

## Incremental Builds

### What It Is

Incremental builds allow TypeScript to only recompile files that have changed since the last build. TypeScript stores build information in a `.tsbuildinfo` file, which tracks file contents, timestamps, and dependency information.

### Why It Is Important

Incremental builds reduce compilation time by an order of magnitude in large projects. Instead of recompiling every file on every build, TypeScript only processes changed files and their dependents.

### How It Works Internally

When `incremental: true` is set, TypeScript generates a `.tsbuildinfo` file that caches:
- File contents hashes
- Source file dependencies
- Emit outputs
- Type information

On subsequent builds, TypeScript compares current file hashes with cached ones and skips unchanged files.

### Syntax

```json
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./.tsbuildinfo",
    "outDir": "./dist"
  }
}
```

### Beginner Examples

```bash
# First build (full)
tsc

# Second build (incremental - faster)
tsc

# Force full rebuild
tsc --force
```

### Intermediate Examples

```json
// tsconfig.json with incremental + project references
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./.tsbuildinfo",
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true
  },
  "include": ["src"]
}
```

### Advanced Examples

```typescript
// Programmatic API with incremental builds
import * as ts from 'typescript';

const compilerOptions: ts.CompilerOptions = {
  incremental: true,
  tsBuildInfoFile: '.tsbuildinfo',
  outDir: './dist',
  target: ts.ScriptTarget.ES2020,
  module: ts.ModuleKind.CommonJS,
};

const host = ts.createIncrementalCompilerHost(compilerOptions);
const program = ts.createIncrementalProgram({
  rootNames: ['src/index.ts'],
  options: compilerOptions,
  host,
});

const result = program.emit();
```

### Real-World Use Cases

```bash
# CI/CD pipeline with incremental builds
# Cache the .tsbuildinfo file between builds

# GitHub Actions example
- name: Cache TypeScript build info
  uses: actions/cache@v3
  with:
    path: |
      **/.tsbuildinfo
      **/dist
    key: ${{ runner.os }}-tsc-${{ hashFiles('src/**/*.ts') }}
    restore-keys: |
      ${{ runner.os }}-tsc-

- name: Build
  run: npx tsc --build
```

### Common Mistakes

```bash
# MISTAKE: Not caching .tsbuildinfo in CI
# Without caching, every CI build starts fresh

# MISTAKE: Using incremental without outDir
# Incremental builds require a stable output directory

# MISTAKE: Cleaning dist without cleaning .tsbuildinfo
# Stale build info can cause incorrect incremental builds
```

### Best Practices

- Always enable `incremental: true` in development.
- Cache `.tsbuildinfo` files in CI.
- Use `tsBuildInfoFile` to control where build info is stored.
- Use `tsc --force` occasionally to ensure clean builds.
- Combine incremental with project references for maximum performance.

### Performance Considerations

Incremental builds can be 2-10x faster than full builds depending on project size and scope of changes. The `.tsbuildinfo` file grows with project size but remains efficient. Initial build time is slightly increased due to writing build info.

### Interview Questions

1. How does TypeScript's incremental compilation work?
2. What is the `.tsbuildinfo` file and why is it important?
3. How do incremental builds interact with project references?

### Coding Challenges

1. Measure the build time difference between incremental and non-incremental builds for a project.
2. Set up a CI pipeline that caches `.tsbuildinfo` files for faster builds.

### Related Topics

- Project references
- TypeScript compiler options
- Build performance

## Type-Checking Performance Optimization

### What It Is

Type-checking performance optimization involves configuring TypeScript and structuring code to minimize the time spent on type checking. This includes compiler options, code patterns, and project structure decisions.

### Why It Is Important

Slow type checking breaks developer flow and reduces productivity. Optimizing type-checking performance is essential for large codebases where type checking can take minutes.

### How It Works Internally

TypeScript's type checker performs complex constraint solving, especially with advanced types (unions, intersections, conditional types). The complexity can grow exponentially with deeply nested generics, large union types, and recursive types.

### Syntax

```json
{
  "compilerOptions": {
    "skipLibCheck": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true
  }
}
```

### Beginner Examples

```json
// Fastest safe configuration
{
  "compilerOptions": {
    "skipLibCheck": true,
    "skipDefaultLibCheck": true,
    "strict": true,
    "target": "ES2020",
    "module": "commonjs"
  }
}
```

### Intermediate Examples

```json
{
  "compilerOptions": {
    // Performance optimizations
    "skipLibCheck": true,
    "skipDefaultLibCheck": true,
    "incremental": true,

    // Type checking strictness (still keep these for safety)
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true,
    "noUncheckedIndexedAccess": true,

    // Exclude from type checking
    "exclude": ["node_modules", "dist", "**/*.test.ts"]
  }
}
```

### Advanced Examples

```typescript
// Performance anti-patterns to avoid:

// AVOID: Large union types
type Status = 'a' | 'b' | 'c' | ...  // 100+ members

// AVOID: Deeply nested conditional types
type DeepNested<T> = T extends { a: infer U }
  ? U extends { b: infer V }
    ? V extends { c: infer W } ? W : never
    : never
  : never;

// AVOID: Complex recursive types without limits
// type Recursive<T> = T extends any[] ? Recursive<T[0]> : T;

// AVOID: Using template literal types with huge unions
type Huge = `${'a'|'b'|...100...}${'x'|'y'|...100...}`;
// Generates 10,000+ union members

// PREFER: Simple, flat types
type SimpleStatus = string;
type SimpleNested<T> = T extends { a: { b: { c: infer W } } } ? W : never;
```

### Real-World Use Cases

```json
// Production tsconfig for a large application
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "incremental": true,
    "tsBuildInfoFile": "./node_modules/.cache/tsbuildinfo",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "exclude": ["node_modules", "dist", "coverage"]
}
```

### Common Mistakes

```typescript
// MISTAKE: Using `any` to speed up type checking
// This defeats the purpose of TypeScript and can hide errors

// MISTAKE: Disabling strict mode for performance
// The performance gain is minimal, and the loss in safety is significant

// MISTAKE: Not using skipLibCheck
// Skipping node_modules type checking provides massive speedup
```

### Best Practices

- Enable `skipLibCheck: true` for all projects.
- Use `exactOptionalPropertyTypes` and `noUncheckedIndexedAccess` instead of runtime guards.
- Avoid large union types (>50 members); use branded types or interfaces instead.
- Break large files into smaller, focused modules.
- Use interfaces over type aliases for object types (slightly faster checking).
- Prefer simple generics over complex conditional types.
- Use `@ts-expect-error` sparingly; each one adds type checking overhead.

### Performance Considerations

Performance bottlenecks typically come from:
- Large union types (exponential checking)
- Deeply nested conditional types
- Recursive types with high instantiation depth
- Template literal types with large unions
- Files with many exported types (checking happens for each consumer)

### Interview Questions

1. What compiler options can improve type-checking performance?
2. How do large union types affect type-checking performance?
3. What is `skipLibCheck` and why is it important?

### Coding Challenges

1. Profile a TypeScript project's type-checking time and identify the slowest files.
2. Refactor a file with large union types to reduce type-checking time.

### Related Topics

- Incremental builds
- Project references
- TypeScript compiler options

## const assertions and Performance

### What It Is

Const assertions (`as const`) tell TypeScript to infer the narrowest possible type for a value. For literal types, arrays, and objects, this preserves exact values rather than widening them to their general types.

### Why It Is Important

Const assertions enable precise type inference that can actually improve type-checking performance by reducing the need for complex narrowing and widening operations. They also make discriminated unions more efficient.

### How It Works Internally

When you write `as const`, TypeScript:
- Infers literal types instead of base types (`'hello'` instead of `string`)
- Infers readonly tuples instead of mutable arrays
- Makes all object properties `readonly`

This reduces the type space that TypeScript needs to consider during checking.

### Syntax

```typescript
const colors = ['red', 'green', 'blue'] as const;
// type: readonly ['red', 'green', 'blue']

const config = { host: 'localhost', port: 3000 } as const;
// type: { readonly host: 'localhost'; readonly port: 3000 }
```

### Beginner Examples

```typescript
// Without const assertion (widened)
const colors = ['red', 'green', 'blue'];
// type: string[]

// With const assertion (exact)
const colorsExact = ['red', 'green', 'blue'] as const;
// type: readonly ['red', 'green', 'blue']

// Benefits in discriminated unions
type Action =
  | { type: 'increment' }
  | { type: 'decrement' };

const action = { type: 'increment' } as const;
// action.type is 'increment', not string
```

### Intermediate Examples

```typescript
// Exhaustive switch with const assertions
function handleAction(action: Action) {
  switch (action.type) {
    case 'increment':
      return action; // Narrowed correctly
    case 'decrement':
      return action;
  }
}

// Object literals for config
const HTTP_STATUS = {
  OK: 200,
  NOT_FOUND: 404,
  INTERNAL_ERROR: 500,
} as const;

type HttpStatus = typeof HTTP_STATUS[keyof typeof HTTP_STATUS];
// type: 200 | 404 | 500 (not number)
```

### Advanced Examples

```typescript
// Const assertions with function returns
function createConfig<T extends Record<string, any>>(config: T): Readonly<T> {
  return Object.freeze({ ...config }) as Readonly<T>;
}

const appConfig = createConfig({
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3,
} as const);

// appConfig.apiUrl is 'https://api.example.com' (literal, not string)

// Complex nested config
const featureFlags = {
  dashboard: {
    enabled: true,
    version: 2,
  },
  reporting: {
    enabled: false,
    formats: ['pdf', 'csv'] as const,
  },
} as const;

type FeatureFlags = typeof featureFlags;
// Deeply readonly with literal types
```

### Real-World Use Cases

```typescript
// Effect on performance: reducing union sizes
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';

// Without as const
const methods = ['GET', 'POST', 'PUT', 'DELETE'];
// methods is string[] - can't be assigned to HttpMethod[]

// With as const
const methodsTyped = ['GET', 'POST', 'PUT', 'DELETE'] as const;
// methodsTyped is readonly ['GET', 'POST', 'PUT', 'DELETE']
// Can be used where readonly HttpMethod[] is expected

// Enum alternative using const assertion
const Direction = {
  Up: 'UP',
  Down: 'DOWN',
  Left: 'LEFT',
  Right: 'RIGHT',
} as const;

type Direction = typeof Direction[keyof typeof Direction];
// 'UP' | 'DOWN' | 'LEFT' | 'RIGHT'
```

### Common Mistakes

```typescript
// MISTAKE: Using as const on mutable data
// const data = getMutableData() as const; // Dangerous! Data might change

// MISTAKE: Overusing as const on large objects
// Each property becomes readonly, which may be inconvenient

// MISTAKE: as const on computed properties
// const x = { [computeKey()]: 'value' } as const; // Key type is still string
```

### Best Practices

- Use `as const` for configuration objects, enums, and constant arrays.
- Use `as const` on action creators and discriminated union discriminants.
- Combine `as const` with `satisfies` for runtime-validated constants.
- Prefer `as const` over `enum` for better tree-shaking and simpler types.

### Performance Considerations

Const assertions can improve performance by reducing type widening and the need for type narrowing. They also enable TypeScript to skip certain type compatibility checks because the types are more precise and readonly.

### Interview Questions

1. What does `as const` do to type inference?
2. How can `as const` improve type-checking performance?
3. When would you use `as const` vs `enum`?

### Coding Challenges

1. Refactor an enum-based configuration to use `as const` and measure type-checking time.
2. Create a type-safe HTTP client that uses `as const` for method and header definitions.

### Related Topics

- Type widening
- Literal types
- Readonly types

## Avoiding Implicit Any

### What It Is

Implicit `any` occurs when TypeScript cannot determine a type and defaults to `any`. This bypasses type checking entirely, defeating TypeScript's purpose. Avoiding implicit `any` ensures comprehensive type coverage.

### Why It Is Important

Every `any` type creates a blind spot in type checking. The `noImplicitAny` compiler option (included in `strict: true`) catches cases where TypeScript would infer `any`, forcing explicit annotations and ensuring complete type safety.

### How It Works Internally

When TypeScript encounters an untyped parameter, variable, or return value, it normally infers the type. If it can't determine a specific type, it would default to `any` (with `noImplicitAny: false`) or report an error (with `noImplicitAny: true`).

### Syntax

```json
{
  "compilerOptions": {
    "noImplicitAny": true,
    "strict": true
  }
}
```

```typescript
// This would error with noImplicitAny
function process(data) {
  // Error: Parameter 'data' implicitly has 'any' type
  return data.value;
}

// Fix with explicit type
function process(data: { value: string }) {
  return data.value;
}
```

### Beginner Examples

```typescript
// BAD: Implicit any
function logItems(items) {
  items.forEach(item => console.log(item));
}

// GOOD: Explicit type
function logItems(items: string[]) {
  items.forEach(item => console.log(item));
}

// BAD: Implicit any in callback
const result = someArray.map(item => item.value);

// GOOD: Typed callback
const result = someArray.map((item: { value: number }) => item.value);
```

### Intermediate Examples

```typescript
// BAD: Implicit any from destructuring
function processConfig({ host, port }) {
  // host: any, port: any
  return `http://${host}:${port}`;
}

// GOOD: Typed destructuring
function processConfig({ host, port }: { host: string; port: number }) {
  return `http://${host}:${port}`;
}

// BAD: Implicit return type
function createUser(name, email) {
  return { name, email, createdAt: new Date() };
}

// GOOD: Explicit parameter types and inferred return
function createUser(name: string, email: string) {
  return { name, email, createdAt: new Date() };
}
```

### Advanced Examples

```typescript
// BAD: any in arrays
const items = []; // any[]
items.push('hello'); // OK but unsafe

// GOOD: Typed array
const items: string[] = [];
items.push('hello'); // OK and type-safe

// BAD: Event callback with any
button.addEventListener('click', (event) => {
  // event: any
});

// GOOD: Typed event
button.addEventListener('click', (event: MouseEvent) => {
  console.log(event.clientX);
});

// BAD: any from JSON.parse
function getConfig() {
  const data = JSON.parse(fs.readFileSync('config.json', 'utf-8'));
  return data; // any
}

// GOOD: Type-safe JSON parsing
interface AppConfig {
  host: string;
  port: number;
}

function getConfig(): AppConfig {
  const data = JSON.parse(fs.readFileSync('config.json', 'utf-8'));
  return data as AppConfig;
}
```

### Real-World Use Cases

```typescript
// Migrating from JS to TS: Use precise types, not any

// BAD: Loose typing
function fetchData(url: string): any {
  return fetch(url).then(r => r.json());
}

const data = fetchData('/api/users');
console.log(data.name); // Unsafe - no type checking

// GOOD: Precise typing
interface User {
  id: string;
  name: string;
  email: string;
}

async function fetchUsers(url: string): Promise<User[]> {
  const response = await fetch(url);
  return response.json() as Promise<User[]>;
}

const users = await fetchUsers('/api/users');
console.log(users[0].name); // Safe - full type checking
```

### Common Mistakes

```typescript
// MISTAKE: Using any to suppress errors
function process(data: any): any {
  return data.value;
}

// MISTAKE: not handling JSON.parse return type
// JSON.parse always returns any

// MISTAKE: Using any[] instead of a specific array type
// Prefer: string[], number[], Array<{ id: string }>
```

### Best Practices

- Always enable `noImplicitAny` (included in `strict`).
- Prefer `unknown` over `any` when you truly don't know the type.
- Use type assertions (`as Type`) as a last resort, not a first choice.
- Use Zod or similar for validating external data.
- Use `@ts-expect-error` for legitimate exceptions, not to suppress type errors.

### Performance Considerations

Avoiding `any` has a positive impact on type-checking performance in the long run. While explicit types require more code, they reduce the search space for the type checker. Each `any` creates a potential performance bottleneck as it widens all downstream types.

### Interview Questions

1. What is implicit `any` and why should you avoid it?
2. How does `noImplicitAny` help with type safety?
3. What is the difference between `any` and `unknown`?

### Coding Challenges

1. Refactor a codebase that uses `any` extensively to use precise types.
2. Create a utility type that replaces `any` with `unknown` while maintaining usability.

### Related Topics

- Strict mode
- unknown type
- Type assertions
