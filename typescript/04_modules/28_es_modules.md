# ES Modules - import/export in TS, type-only imports, re-exports, ambient modules

## Introduction
ES Modules (ESM) are the standard module system for modern JavaScript and TypeScript. They provide a declarative syntax for importing and exporting bindings between files. TypeScript fully supports ESM while extending it with type-only imports, re-export patterns, and ambient module declarations. Modules are the foundation of code organization, dependency management, and encapsulation in TypeScript applications.

## import and export in TypeScript

### What It Is
`import` and `export` are the ESM keywords for sharing code between files. `export` makes declarations available to other modules; `import` brings them into scope.

### Why It Is Important
Modules provide file-level encapsulation, explicit dependency declarations, and enable tree-shaking. They are essential for maintaining large codebases.

### How It Works Internally
TypeScript preserves `import` and `export` statements in the emitted JavaScript. The module resolution determines how import paths map to files. The output format (CommonJS, ESM, etc.) depends on the `module` compiler option.

### Syntax
```typescript
// file: math.ts
export function add(a: number, b: number): number {
  return a + b;
}
// file: app.ts
import { add } from "./math.js";
console.log(add(2, 3));
```

### Beginner Examples
```typescript
// user.ts
export interface User {
  id: string;
  name: string;
}
export function createUser(name: string): User {
  return { id: crypto.randomUUID(), name };
}
export const DEFAULT_USER: User = { id: "0", name: "Guest" };

// app.ts
import { User, createUser, DEFAULT_USER } from "./user.js";
const user = createUser("Alice");
```

### Intermediate Examples
```typescript
// services/database.ts
export class Database {
  async connect(): Promise<void> { }
  async query(sql: string): Promise<unknown[]> { return []; }
}
export type QueryResult<T> = { rows: T[]; count: number };
export const CONNECTION_STRING = "postgres://localhost/db";

// services/index.ts
export { Database } from "./database.js";
export type { QueryResult } from "./database.js";

// app.ts
import { Database, CONNECTION_STRING } from "./services/index.js";
```

### Advanced Examples
```typescript
// default exports
export default class HttpClient {
  async get<T>(url: string): Promise<T> {
    const res = await fetch(url);
    return res.json();
  }
}
// importing default
import HttpClient from "./http-client.js";

// namespace imports
import * as MathUtils from "./math-utils.js";
console.log(MathUtils.add(1, 2));
console.log(MathUtils.PI);

// dynamic imports
async function loadModule(path: string): Promise<void> {
  const mod = await import(path);
  mod.initialize();
}
```

### Real-World Use Cases
- Structuring applications by feature or layer (controllers, services, repositories)
- Sharing types and interfaces across the codebase
- Loading modules lazily for code splitting

### Common Mistakes
- Using incorrect file extensions in import paths (`.ts` vs `.js` in output)
- Forgetting that default exports have different import syntax than named exports
- Circular dependencies between modules

### Best Practices
- Use named exports (not default exports) for better refactoring and tree-shaking
- Keep modules focused on a single responsibility
- Avoid deep import paths — use barrel files (index.ts) to simplify exports

### Performance Considerations
ES modules are statically analyzable, enabling tree-shaking. Dynamic imports enable code splitting. The module resolution strategy affects build time.

### Interview Questions
- Q: What is the difference between named exports and default exports?
  A: Named exports allow multiple exports per module; default exports allow one. Named exports enable better tooling (autocomplete, refactoring).

### Coding Challenges
- Refactor a monolithic file into multiple modules with proper exports and imports.
- Create a barrel file (index.ts) that re-exports from several feature modules.

### Related Topics
- Type-only imports
- Re-exports
- Module resolution

---

## type-only imports (import type)

### What It Is
`import type` is a syntax that imports only the type information from a module, ensuring no JavaScript is emitted for the import at runtime. It can also be used with `export type`.

### Why It Is Important
Type-only imports prevent runtime errors when a module has side effects or when importing types from a package that should not be included in the output. They also enable faster builds and smaller bundles.

### How It Works Internally
`import type` declarations are completely erased in the emitted JavaScript. The type checker sees them during compilation, but no `require()` or `import` call is generated.

### Syntax
```typescript
import type { User } from "./types.js";
import { type Logger, type Config } from "./shared.js";
```

### Beginner Examples
```typescript
// types.ts
export interface User {
  id: string;
  name: string;
}
// service.ts
import type { User } from "./types.js";
export function formatUser(user: User): string {
  return user.name + " (" + user.id + ")";
}
```

### Intermediate Examples
```typescript
// Using inline type imports
import { createLogger, type LoggerConfig } from "./logger.js";
const config: LoggerConfig = { level: "info" };
const logger = createLogger(config);

// Re-exporting types
export type { Logger } from "./logger.js";
export { FileTransport } from "./transports.js";
```

### Advanced Examples
```typescript
export function process<T>(data: T): void {
}

export type {
  Request as ApiRequest,
  Response as ApiResponse,
  Middleware,
} from "./types.js";

// Importing type and value separately
import { EventEmitter } from "events";
import type { EventMap } from "./events.js";
class TypedEmitter<T extends EventMap> extends EventEmitter {
  emit<K extends keyof T>(event: K, ...args: T[K]): boolean {
    return super.emit(event as string, ...args);
  }
}
```

### Real-World Use Cases
- Importing type definitions from packages that are not installed (ambient types)
- Avoiding circular dependencies caused by type imports
- Preventing module side effects from being executed during tests or SSR

### Common Mistakes
- Using `import type` for values (causes compile error)
- Forgetting to use `import type` when only types are needed from a module
- Using `import type` with default exports

### Best Practices
- Always use `import type` when importing only types
- Enable `verbatimModuleSyntax` in tsconfig to enforce this
- Use inline `type` in destructured imports: `import { type A, B } from "./mod"`

### Performance Considerations
Reduces bundle size by eliminating unnecessary imports. Speeds up compilation by reducing the module resolution graph.

### Interview Questions
- Q: What is the difference between `import type` and `import { type }`?
  A: `import type` makes the entire import type-only. `import { type X }` makes only specific bindings type-only.

### Coding Challenges
- Convert a mixed import to use `import type` for the type-only portions.
- Set up a project with `verbatimModuleSyntax` and refactor imports accordingly.

### Related Topics
- import and export
- verbatimModuleSyntax
- Declaration files

---

## Re-exports

### What It Is
Re-exports allow a module to import a binding and then export it again, often under the same or a different name. TypeScript supports aggregation, renaming, and selective re-exporting.

### Why It Is Important
Re-exports enable barrel files (index.ts) that aggregate public APIs, simplify import paths, and decouple internal module structure from the public API surface.

### How It Works Internally
Re-exports are compiled to a combination of import and export statements. For direct re-exports, the module is not actually loaded if only types are re-exported.

### Syntax
```typescript
export { Thing } from "./module.js";
export { Thing as Renamed } from "./module.js";
export * from "./module.js";
```

### Beginner Examples
```typescript
// features/users/index.ts
export { UserService } from "./user-service.js";
export { UserController } from "./user-controller.js";
export type { User, CreateUserDTO } from "./types.js";

// Consumer imports from a clean path
import { UserService, User } from "./features/users/index.js";
```

### Intermediate Examples
```typescript
// Combining and renaming
export { createUser as create, deleteUser as remove } from "./user-service.js";
export type { UserDTO as User } from "./dto.js";

// Star re-export (aggregate all)
export * from "./validators.js";
export * from "./helpers.js";
```

### Advanced Examples
```typescript
// Selective re-export with types
export { Database, type DatabaseConfig } from "./database.js";

// Conditional exports pattern
const isProduction = process.env.NODE_ENV === "production";
export {
  DevLogger as Logger,
  type LoggerConfig,
} from "./dev-logger.js";
export {
  ProdLogger as Logger,
  type LoggerConfig,
} from "./prod-logger.js";

// Re-exporting from multiple levels
export { UserModule } from "./user/index.js";
export { type ApiResponse } from "./common/types.js";
```

### Real-World Use Cases
- Creating public API surfaces for libraries
- Barrel files that aggregate module exports
- Renaming exports to avoid naming conflicts

### Common Mistakes
- Re-exporting conflicting names (use `as` to resolve)
- Star re-exports that unintentionally expose internal modules
- Deep re-export chains that make debugging difficult

### Best Practices
- Use barrel files at the feature or directory level
- Be explicit about re-exports rather than using `export *`
- Keep re-export depth to 1-2 levels

### Performance Considerations
Re-exports do not duplicate code in the output. They are resolved during module linking and have negligible runtime impact.

### Interview Questions
- Q: Can you re-export a renamed default export?
  A: Yes: `export { default as MyDefault } from "./module.js"`.

### Coding Challenges
- Create a barrel index.ts that aggregates exports from 3 feature modules.
- Build a shared library that re-exports types and functions with renamed identifiers.

### Related Topics
- import and export
- Barrel files
- Module resolution

---

## Ambient module declarations

### What It Is
Ambient module declarations use `declare module "module-name"` to describe the shape of external modules that have no type definitions.

### Why It Is Important
Ambient modules allow TypeScript to provide type checking for JavaScript libraries or non-TypeScript modules that do not include their own type definitions.

### How It Works Internally
Ambient module declarations are not compiled to JavaScript. They exist only in `.d.ts` files or in `declare` blocks. The type checker uses them during compilation.

### Syntax
```typescript
declare module "my-library" {
  export function doSomething(): void;
  export const VERSION: string;
}
```

### Beginner Examples
```typescript
// globals.d.ts
declare module "csv-parse" {
  export function parse(input: string): Record<string, string>[];
}

// app.ts
import { parse } from "csv-parse";
const data = parse("a,b\n1,2");
```

### Intermediate Examples
```typescript
// types/env.d.ts
declare module "virtual:config" {
  export const API_URL: string;
  export const APP_NAME: string;
  export function getEnv(key: string): string | undefined;
}

// types/svg.d.ts
declare module "*.svg" {
  const content: string;
  export default content;
}
declare module "*.css" {
  const classes: Record<string, string>;
  export default classes;
}
```

### Advanced Examples
```typescript
// types/plugin-system.d.ts
declare module "@my-company/plugin-*" {
  interface PluginManifest {
    name: string;
    version: string;
    initialize(): Promise<void>;
    destroy(): Promise<void>;
  }
  const manifest: PluginManifest;
  export default manifest;
  export function getDependencies(): string[];
}

// types/extended-package.d.ts
declare module "existing-package" {
  export interface ExistingType {
    newMethod(): void;
  }
  export function extraHelper(): void;
}
```

### Real-World Use Cases
- Type definitions for CSS/SCSS modules in webpack
- Ambient declarations for Vite virtual modules
- Adding types to JS-only npm packages
- Declaring types for asset imports (images, fonts)

### Common Mistakes
- Placing ambient module declarations in `.ts` files (should be `.d.ts`)
- Forgetting to include the ambient file in the tsconfig includes list
- Using relative paths in ambient module names

### Best Practices
- Keep ambient declarations in `.d.ts` files
- Place them in a `types/` directory at the project root
- Use clear naming conventions for custom ambient module names

### Performance Considerations
Ambient declarations have no runtime cost. They are purely compile-time.

### Interview Questions
- Q: What is the difference between an ambient module declaration and a normal module?
  A: Ambient modules declare types for modules that exist externally; they do not emit JavaScript.

### Coding Challenges
- Write an ambient declaration for a hypothetical `@company/utils` package.
- Create ambient declarations for image imports (`*.png`, `*.jpg`) and CSS modules.

### Related Topics
- Declaration files
- Triple-slash directives
- Global augmentation
