# Declaration Files - .d.ts files, declare keyword, global augmentation, module augmentation, triple-slash directives

## Introduction
Declaration files (`.d.ts`) are TypeScript's way of describing the shape of existing JavaScript code, libraries, and runtime environments. They contain only type information — no implementation. The `declare` keyword, global and module augmentation, and triple-slash directives provide mechanisms for creating, extending, and composing declaration files. Declaration files are essential for using JavaScript libraries in TypeScript, writing type definitions, and extending existing types.

## .d.ts file structure

### What It Is
A `.d.ts` file is a TypeScript declaration file that contains type definitions without implementation. It describes the API surface of a module or script.

### Why It Is Important
`.d.ts` files enable TypeScript's type checking for JavaScript libraries and provide intellisense for any JavaScript code. They are the bridge between typed TypeScript and untyped JavaScript.

### How It Works Internally
The TypeScript compiler parses `.d.ts` files during compilation but does not emit JavaScript for them. They contribute only to the type-checking phase.

### Syntax
```typescript
// Example: lodash.d.ts
declare function chunk<T>(array: T[], size?: number): T[][];
declare function compact<T>(array: T[]): T[];
```

### Beginner Examples
```typescript
// math.d.ts
export function add(a: number, b: number): number;
export function subtract(a: number, b: number): number;
export const PI: number;
```

### Intermediate Examples
```typescript
// shapes.d.ts
export interface Point {
  x: number;
  y: number;
}
export interface Circle {
  center: Point;
  radius: number;
}
export function area(circle: Circle): number;
export function distance(a: Point, b: Point): number;
```

### Advanced Examples
```typescript
// database.d.ts
export interface ConnectionConfig {
  host: string;
  port: number;
  user: string;
  password: string;
  database: string;
}
export interface QueryResult<T = unknown> {
  rows: T[];
  rowCount: number;
  command: string;
}
export declare class Database {
  constructor(config: ConnectionConfig);
  connect(): Promise<void>;
  disconnect(): Promise<void>;
  query<T = Record<string, unknown>>(sql: string, params?: unknown[]): Promise<QueryResult<T>>;
  transaction<T>(fn: (client: Database) => Promise<T>): Promise<T>;
}
export declare function createPool(config: ConnectionConfig, maxSize?: number): Database;
```

### Real-World Use Cases
- Creating type definitions for npm packages that lack built-in types
- Describing the API of internal JavaScript modules during migration
- Defining types for Web Workers, WASM modules, or other non-standard environments

### Common Mistakes
- Putting implementation code in `.d.ts` files
- Forgetting to include `.d.ts` files in tsconfig.json's `include` or `files`
- Using relative paths that don't resolve correctly at compile time

### Best Practices
- Keep `.d.ts` files focused on type declarations only
- Use `export` for module declarations and `declare` for ambient (global) ones
- Name `.d.ts` files after the module they describe (e.g., `lodash.d.ts`)

### Performance Considerations
Many `.d.ts` files slow down compilation. Consider using `skipLibCheck: true` in tsconfig to skip checking all `.d.ts` files, or use incremental builds.

### Interview Questions
- Q: Can a `.d.ts` file contain `import` statements?
  A: Yes, `.d.ts` files can use `import` to reference types from other declaration files.

### Coding Challenges
- Write a `.d.ts` file for a simple math library with functions `add`, `subtract`, `multiply`, and `divide`.
- Create declaration files for a hypothetical `fs-extra` style file system library.

### Related Topics
- declare keyword
- Global augmentation
- Module augmentation

---

## declare keyword

### What It Is
The `declare` keyword tells TypeScript that a variable, function, class, or module exists somewhere (typically in JavaScript) and should not be emitted in the output. It provides type information without implementation.

### Why It Is Important
`declare` is essential for describing the types of external JavaScript code, global variables, and runtime-provided values. It bridges TypeScript's type system with the actual runtime environment.

### How It Works Internally
`declare` statements are completely erased during compilation. They only contribute to the type-checking phase.

### Syntax
```typescript
declare const VERSION: string;
declare function greet(name: string): void;
declare class MyClass { }
```

### Beginner Examples
```typescript
// Describing a global variable provided by a script tag
declare const API_KEY: string;
declare function trackEvent(name: string, data?: Record<string, unknown>): void;
declare class Analytics {
  constructor(key: string);
  pageview(path: string): void;
}
```

### Intermediate Examples
```typescript
// Describing a library's module
declare module "csv-parser" {
  export interface ParserOptions {
    separator?: string;
    headers?: boolean;
  }
  export function parse(input: string, options?: ParserOptions): Record<string, string>[];
  export function format(data: Record<string, string>[]): string;
}

// Describing ambient values
declare namespace MyGlobalLib {
  function init(config: Record<string, unknown>): void;
  namespace UI {
    class Button { }
    class Modal { }
  }
}
```

### Advanced Examples
```typescript
// Declaring complex types for a WebSocket library
declare module "ws-wrapper" {
  import { EventEmitter } from "events";

  interface Message {
    id: string;
    type: string;
    payload: unknown;
    timestamp: number;
  }

  interface WSOptions {
    url: string;
    protocols?: string[];
    reconnect?: boolean;
    maxRetries?: number;
  }

  declare class WSClient extends EventEmitter {
    constructor(options: WSOptions);
    connect(): Promise<void>;
    disconnect(): Promise<void>;
    send<T = unknown>(type: string, payload: T): Promise<Message>;
    on<T = unknown>(event: "message", handler: (msg: Message & { payload: T }) => void): this;
    on(event: "open" | "close" | "error", handler: () => void): this;
    static readyStates: Readonly<{
      CONNECTING: 0;
      OPEN: 1;
      CLOSING: 2;
      CLOSED: 3;
    }>;
  }

  export { WSClient, WSClient as default, Message, WSOptions };
}
```

### Real-World Use Cases
- Declaring types for browser globals like `process`, `__dirname` in Node.js
- Describing external JavaScript libraries without type definitions
- Declaring environment variables injected at build time

### Common Mistakes
- Adding implementation to `declare` statements (they must have no body)
- Using `declare` for pure TypeScript modules (not needed — just use `export`)
- Forgetting to include `declare` in ambient contexts

### Best Practices
- Use `declare` only for code that exists outside TypeScript's control
- Prefer writing regular TypeScript modules for code you control
- Use `declare global` to add types to the global scope from within a module

### Performance Considerations
`declare` statements are completely erased. They have zero runtime cost.

### Interview Questions
- Q: What is the difference between `declare var` and `declare const`?
  A: `declare const` indicates the value cannot be reassigned; `declare var` allows reassignment.

### Coding Challenges
- Write a `declare` block for a hypothetical `PayPal` global SDK with `createOrder`, `captureOrder`, and `onApprove` methods.
- Create declarations for a `renderMath` function available globally that takes a LaTeX string and returns an HTML element.

### Related Topics
- .d.ts file structure
- Global augmentation
- Module augmentation

---

## Global augmentation

### What It Is
Global augmentation allows adding new members to the global scope from within a module. It uses `declare global { }` to extend global interfaces and declare new global values.

### Why It Is Important
Global augmentation enables extending global types (like `Window`, `NodeJS.Process`, `Array`) without modifying the original declaration files. It is commonly used for polyfills, custom globals, and framework-specific augmentations.

### How It Works Internally
The `declare global` block merges its declarations into the global scope. TypeScript performs declaration merging to combine the augmentations with existing global types.

### Syntax
```typescript
// inside a module
declare global {
  interface Window {
    myApp: { version: string };
  }
}
```

### Beginner Examples
```typescript
// globals.ts
export {};

declare global {
  const APP_VERSION: string;
  function log(message: string): void;
}

// Usage in any module
console.log(APP_VERSION);
log("Application started");
```

### Intermediate Examples
```typescript
// types/window-augmentation.ts
export {};

declare global {
  interface Window {
    __INITIAL_STATE__: Record<string, unknown>;
    gtag: (command: string, ...args: unknown[]) => void;
    dataLayer: unknown[];
  }
  namespace NodeJS {
    interface ProcessEnv {
      MY_CUSTOM_VAR: string;
      NODE_ENV: "development" | "production" | "test";
    }
  }
}

// app.ts
const state = window.__INITIAL_STATE__;
window.gtag("config", "GA-123");
```

### Advanced Examples
```typescript
// types/extensions.ts
export {};

declare global {
  interface Array<T> {
    groupBy<K extends string | number | symbol>(keyFn: (item: T) => K): Record<K, T[]>;
    distinct(): T[];
    first(): T | undefined;
    last(): T | undefined;
  }
  interface PromiseConstructor {
    delay(ms: number): Promise<void>;
    timeout<T>(promise: Promise<T>, ms: number): Promise<T>;
  }
  interface String {
    capitalize(): string;
    toTitleCase(): string;
    truncate(maxLength: number, suffix?: string): string;
  }
}

// Implementing the augmentations
if (!Array.prototype.groupBy) {
  Array.prototype.groupBy = function groupBy(keyFn) {
    return this.reduce((acc, item) => {
      const key = keyFn(item);
      (acc[key] ??= []).push(item);
      return acc;
    }, {} as Record<string | number | symbol, unknown[]>);
  };
}
```

### Real-World Use Cases
- Adding custom properties to the `Window` object for server-rendered state
- Extending `Express.Request` with custom user properties
- Augmenting `Array` or `String` prototypes with custom methods
- Adding type information for third-party script injected globals

### Common Mistakes
- Forgetting `export {}` at the top of the file (required to make it a module)
- Augmenting types that don't exist (e.g., augmenting a non-existent namespace)
- Overwriting existing properties instead of safely augmenting them

### Best Practices
- Always include `export {}` at the top (or any real export) to make the file a module
- Use global augmentation sparingly — prefer ES module exports when possible
- Place global augmentations in dedicated `.d.ts` files

### Performance Considerations
No runtime cost. Augmentations are purely type-level.

### Interview Questions
- Q: Why do you need `export {}` at the top of a global augmentation file?
  A: To make the file a module rather than a script, so `declare global` is properly scoped.

### Coding Challenges
- Add types to the `Window` interface for custom analytics properties `ga` and `fbq`.
- Augment the `Request` interface from Express to include a `user` property with custom User type.

### Related Topics
- declare keyword
- Module augmentation
- Declaration merging

---

## Module augmentation

### What It Is
Module augmentation allows adding new exports or modifying existing types of an imported module without changing its source code. It uses `declare module "module-name"` inside a module context.

### Why It Is Important
Module augmentation enables extending third-party libraries with custom methods, types, or configuration. It is essential for plugin systems, middleware extensions, and framework customization.

### How It Works Internally
TypeScript merges the declared module contents with the original module's type definitions. This is a form of declaration merging scoped to a specific module.

### Syntax
```typescript
import "some-library";
declare module "some-library" {
  export function newFunction(): void;
}
```

### Beginner Examples
```typescript
// augmentations/lodash-augment.ts
import "lodash";

declare module "lodash" {
  interface LoDashStatic {
    greet(name: string): string;
  }
}

// Usage
import _ from "lodash";
_.greet("Alice");
```

### Intermediate Examples
```typescript
// types/express-augment.d.ts
import "express";

declare module "express" {
  interface Request {
    user?: {
      id: string;
      name: string;
      roles: string[];
    };
    requestId: string;
  }
  interface Response {
    success(data: unknown): void;
    error(message: string, statusCode?: number): void;
  }
}

// Usage in route handler
app.get("/profile", (req, res) => {
  const user = req.user;
  res.success({ profile: user });
});
```

### Advanced Examples
```typescript
// types/knex-augment.d.ts
import "knex";

declare module "knex" {
  namespace Knex {
    interface QueryBuilder<TRecord extends Record<string, unknown> = any, TResult = unknown[]> {
      paginate(page: number, perPage: number): Promise<{
        data: TResult;
        total: number;
        page: number;
        perPage: number;
        totalPages: number;
      }>;
      softDelete(): QueryBuilder<TRecord, number>;
      whereNotDeleted(): QueryBuilder<TRecord, TResult>;
    }
    interface TableNames {
      users: "users";
      posts: "posts";
      comments: "comments";
    }
  }
}

// Usage
const result = await knex("users")
  .where("active", true)
  .paginate(1, 20);

await knex("posts").where("id", 123).softDelete();
```

```typescript
// Augmenting a component library
import "antd";

declare module "antd/lib/button" {
  interface ButtonProps {
    loading?: boolean;
    icon?: React.ReactNode;
    shape?: "circle" | "round" | "default";
  }
}
```

### Real-World Use Cases
- Adding custom methods to ORM query builders (Knex, TypeORM)
- Extending Express Request/Response with custom properties
- Adding plugin methods to libraries (Lodash, jQuery)
- Extending UI component library prop types

### Common Mistakes
- Not importing the module before augmenting it
- Augmenting a module that doesn't have declaration files (use ambient module instead)
- Augmenting without matching the exact module name used in imports

### Best Practices
- Place module augmentations in separate files under `types/` directory
- Import the target module at the top of the augmentation file
- Use `declare module "exact-module-name"` with the exact import path

### Performance Considerations
No runtime cost. Module augmentations are type-level only.

### Interview Questions
- Q: Can you augment a module that has no existing type declarations?
  A: Yes, you can use `declare module "name"` to create ambient types for untyped modules.

### Coding Challenges
- Augment the `axios` module to add a `cancelAll` static method.
- Augment the `dayjs` module to add a `fromNow` method that returns a localized relative time string.

### Related Topics
- declare keyword
- Global augmentation
- Declaration merging

---

## Triple-slash directives

### What It Is
Triple-slash directives are single-line comments that contain XML tags. They serve as compiler directives for referencing other files, specifying library dependencies, and controlling module resolution.

### Why It Is Important
Before ES modules became standard, triple-slash directives were the primary mechanism for referencing dependencies between files. They are still used in declaration files for referencing other type definitions.

### How It Works Internally
The TypeScript compiler processes triple-slash directives during the preprocessing phase. They cause the compiler to include additional files in the compilation context.

### Syntax
```typescript
/// <reference path="..." />
/// <reference types="..." />
/// <reference lib="..." />
/// <amd-module name="..." />
```

### Beginner Examples
```typescript
/// <reference types="node" />

// Now Node.js types are available
const fs = require("fs");
const path = require("path");
```

### Intermediate Examples
```typescript
// types/globals.d.ts
/// <reference types="vite/client" />
/// <reference lib="dom" />
/// <reference lib="esnext" />

// Now Vite's client types, DOM APIs, and ESNext features are available
const meta = import.meta.env;
const el = document.getElementById("app");
```

### Advanced Examples
```typescript
// types/complex-refs.d.ts
/// <reference path="./base-types.d.ts" />
/// <reference path="./extensions.d.ts" />
/// <reference lib="es2022.full" />
/// <reference types="jest" />
/// <reference types="cypress" />

// index.ts — using all referenced types
import { BaseType } from "./some-module";
```

```typescript
// Legacy module declaration with AMD
/// <amd-module name="myapp/utils" />
export function helper(): void { }
```

### Real-World Use Cases
- Referencing DOM or ESNext library types in non-standard environments
- Including type definitions in legacy projects without module systems
- Declaration files that depend on other declaration files
- Setting up test environments with Jest/Cypress type references

### Common Mistakes
- Using triple-slash directives in ES module files (prefer `import`)
- Forgetting the triple slash (`///`) must be at the top of the file
- Using `reference path` when `reference types` is more appropriate

### Best Practices
- Avoid triple-slash directives in modern TypeScript code — use `import` instead
- Use `reference types` for npm package type dependencies
- Use `reference lib` for standard library targets
- Place all directives at the very top of the file before any other code

### Performance Considerations
Triple-slash directives cause the compiler to include additional files, which can slow compilation. Use them sparingly.

### Interview Questions
- Q: What is the difference between `reference path` and `reference types`?
  A: `path` references a specific file; `types` references a package's type declarations from `node_modules/@types`.

### Coding Challenges
- Create a `project.d.ts` that references base types, DOM lib, and a third-party type package.
- Write a legacy declaration file that uses triple-slash directives to compose three separate type files.

### Related Topics
- .d.ts file structure
- Module resolution
- tsconfig.json
