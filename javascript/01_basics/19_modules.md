# Modules - import, export, default exports, named exports, dynamic imports

## Introduction
Modules are a way to organize JavaScript code into separate files, each with its own scope. ES modules (ESM) are the official standard for JavaScript modules, providing `import` and `export` statements. Understanding modules is essential for building maintainable, scalable applications.

## export statement

### What It Is
The `export` statement is used to expose values (variables, functions, classes, etc.) from a module so they can be imported by other modules.

### Why It Is Important
Exports define the public API of a module. They control what is accessible to consumers and enable code reuse, encapsulation, and separation of concerns.

### How It Works Internally
When the module loader processes a module, it creates a module record with exported bindings. These bindings are live connections, meaning changes in the exporting module are reflected in the importing module. Exports are resolved statically at parse time.

### Syntax
```javascript
// Named exports
export const name = "value";
export function func() {}
export class ClassName {}
export { name, func as alias };

// Default export
export default expression;
export default function() {}
export default class {}

// Re-exports
export { name } from './module.js';
export * from './module.js';
export { default } from './module.js';
```

### Beginner Examples
```javascript
// utils.js
export const PI = 3.14159;
export function double(n) {
  return n * 2;
}
export function square(n) {
  return n * n;
}

// Or export at the end
const PI2 = 3.14159;
function double2(n) { return n * 2; }
function square2(n) { return n * n; }
export { PI2, double2, square2 };

// Default export
export default function greet(name) {
  return `Hello, ${name}!`;
}

// Export with alias
function internalFn() { /* ... */ }
export { internalFn as publicFn };
```

### Intermediate Examples
```javascript
// math.js
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// Re-exporting from another module
export { add, subtract } from './arithmetic.js';

// Aggregating exports
export * from './constants.js';
export { default as Logger } from './logger.js';

// Exporting bindings (live)
export let counter = 0;
export function increment() {
  counter++; // Changes reflected in importers
}

// Exporting types (with JSDoc, or TypeScript)
/** @type {string} */
export const API_URL = "https://api.example.com";
```

### Advanced Examples
```javascript
// barrel exports (index.js)
export { UserService } from './services/user.service.js';
export { AuthService } from './services/auth.service.js';
export { ConfigService } from './services/config.service.js';

// Conditional exports (through re-exports)
// Use package.json exports field instead, but possible:
export { default as PlatformAPI } from './platform/browser.js';
// Dynamic switching is better handled by bundlers

// Exporting instances
export const db = createConnection();
export const cache = new Map();

// Self-initializing export
export const initialized = (() => {
  console.log("Module initializing");
  return true;
})();

// Export with getter
let _value = 0;
export function getValue() { return _value; }
export function setValue(v) { _value = v; }
```

### Real-World Use Cases
- Utility function libraries
- Service modules (API clients, database connections)
- Configuration constants
- Component libraries (React/Vue)
- Plugin systems
- API route handlers
- Shared type definitions
- Middleware collections

### Common Mistakes
- Mixing default and named exports incorrectly
- Forgetting that exports are live bindings (not copies)
- Creating circular dependencies
- Using `export default` with `const`/`let` (SyntaxError)
- Forgetting file extensions in Node.js (require .js)
- Exporting mutable objects from singletons (accidental shared state)

### Best Practices
- Prefer named exports over default exports (better refactoring, tree-shaking)
- Use consistent naming conventions for exports
- Use barrel files (`index.js`) for organizing related modules
- Avoid circular dependencies
- Keep modules focused (single responsibility)
- Use `export` inline to reduce boilerplate
- Document exported API with JSDoc
- Use `export default` for the main export of a module

### Performance Considerations
- Named exports enable better tree-shaking by bundlers
- Default exports may prevent tree-shaking of unused named exports
- Live bindings have no runtime overhead over copies
- Static exports are resolved at module load time
- Re-exports create no additional runtime cost

### Interview Questions
1. What is the difference between named exports and default exports?
2. How do you re-export from another module?
3. Are exports live bindings or copies?
4. What happens if you export a mutable object?
5. Can you use `export default` with `const`?

### Coding Challenges
1. Create a utility module with named exports and a default export.
2. Implement a barrel module that re-exports from multiple submodules.
3. Write a module that exports a singleton instance.
4. Create a module with live bindings (counter pattern).

### Related Topics
- import statement
- Module resolution
- CommonJS vs ESM
- Tree-shaking

## import statement

### What It Is
The `import` statement is used to bring exported values from other modules into the current module's scope.

### Why It Is Important
Imports enable code reuse and modular architecture. They make dependencies explicit and enable static analysis for bundlers and tools.

### How It Works Internally
Imports are hoisted to the top of the module and resolved before the module executes. Imported bindings are live connections to the exporting module. The module loader fetches and parses dependencies in a tree, executing them in dependency order.

### Syntax
```javascript
// Named imports
import { name, func } from './module.js';
import { name as alias } from './module.js';
import * as module from './module.js';

// Default import
import defaultExport from './module.js';

// Mixed imports
import defaultExport, { named1, named2 } from './module.js';

// Side effects only
import './module.js';

// Dynamic import
const module = await import('./module.js');
```

### Beginner Examples
```javascript
// Named imports
import { PI, double, square } from './utils.js';
console.log(PI);          // 3.14159
console.log(double(5));   // 10
console.log(square(3));   // 9

// Default import
import greet from './greeting.js';
console.log(greet("Alice")); // "Hello, Alice!"

// Namespace import
import * as utils from './utils.js';
console.log(utils.PI);
console.log(utils.double(5));
console.log(utils.square(3));

// Import with alias
import { double as twice } from './utils.js';
console.log(twice(4)); // 8

// Import for side effects
import './polyfills.js';
```

### Intermediate Examples
```javascript
// Mixed imports
import defaultLogger, { logLevel, formatMessage } from './logger.js';

// Re-importing from barrel
import { UserService, AuthService } from './services/index.js';

// Conditional import with try/catch
let i18n;
try {
  i18n = await import('./i18n/en.js');
} catch {
  i18n = await import('./i18n/fallback.js');
}

// Import with dynamic specifier
const locale = 'fr';
const translations = await import(`./i18n/${locale}.js`);

// Type imports (TypeScript)
import type { User } from './types.js';
```

### Advanced Examples
```javascript
// Dynamic import for lazy loading
const button = document.getElementById('load');
button.addEventListener('click', async () => {
  const module = await import('./heavy-module.js');
  module.initialize();
});

// Import for module initialization
let initialized = false;
const config = await import('./config.js').then(m => {
  initialized = true;
  return m.default;
});

// Circular dependency handling (import at usage time)
// a.js
export const A = "from A";
// Keep import inside function to avoid circular issue
export function getB() {
  return import('./b.js').then(m => m.B);
}

// Async module initialization
const db = await import('./db.js');
await db.connect();

// Import assertion (import attributes in newer syntax)
import data from './data.json' assert { type: 'json' };
```

### Real-World Use Cases
- Importing library components
- Lazy-loading routes in SPAs (React Router)
- Dynamic i18n loading
- Plugin systems with dynamic imports
- Code splitting with bundlers
- Importing configuration files
- Loading polyfills conditionally
- Environment-specific module loading

### Common Mistakes
- Using `import` inside a function synchronously (use dynamic import)
- Forgetting file extensions in Node.js ESM
- Importing from non-existent paths
- Creating circular imports that cause runtime issues
- Using CommonJS `require` in ESM modules
- Not handling dynamic import promise rejections

### Best Practices
- Place all static imports at the top of the file
- Use named imports for clarity (auto-completion)
- Use namespace imports for convenience with many exports
- Use dynamic imports for lazy loading and code splitting
- Handle dynamic import errors with try/catch
- Use absolute imports for cleaner paths (with bundler config)
- Group imports by type (external, internal, styles)
- Prefer static imports over dynamic for critical dependencies

### Performance Considerations
- Static imports are preloaded (optimized by bundlers)
- Dynamic imports trigger separate network requests (lazy loading)
- Tree-shaking removes unused imports in production builds
- Importing many small modules may cause overhead
- Bundlers can optimize import size (scope hoisting)

### Interview Questions
1. What is the difference between static and dynamic imports?
2. How do you import a default export with a different name?
3. What is a namespace import and when would you use it?
4. How do circular imports work in ES modules?
5. What is tree-shaking and how does it relate to imports?

### Coding Challenges
1. Write an import statement that imports both default and named exports.
2. Implement lazy loading using dynamic import.
3. Create a barrel file that exports multiple modules.
4. Write a function that dynamically imports a module based on a condition.

### Related Topics
- export statement
- Module loading
- Dynamic imports
- Bundlers (Webpack, Vite, Rollup)

## Default exports

### What It Is
A default export is a single named export from a module marked as `default`. Each module can have at most one default export.

### Why It Is Important
Default exports provide a convenient way to export the "main" functionality of a module. They allow simpler import syntax and are commonly used for components, classes, and single-function modules.

### How It Works Internally
The default export is stored as a special binding named `default` in the module record. When importing, the imported name is bound directly to this `default` export.

### Syntax
```javascript
// Exporting
export default expression;
export default function() {}
export default class {}

// Importing
import anyName from './module.js';
import defaultExport, { named } from './module.js';
```

### Beginner Examples
```javascript
// greet.js
export default function greet(name) {
  return `Hello, ${name}!`;
}

// app.js
import greet from './greet.js';
// import greetFunction from './greet.js'; // Any name works
console.log(greet("Alice")); // "Hello, Alice!"

// Default class export
export default class User {
  constructor(name) { this.name = name; }
}

// Default expression export
export default {
  env: "development",
  port: 3000
};
```

### Intermediate Examples
```javascript
// Combining default and named exports
// logger.js
export default function log(message) { console.log(message); }
export const LEVELS = { INFO: "info", ERROR: "error" };
export function format(msg) { return `[${new Date().toISOString()}] ${msg}`; }

// app.js
import log, { LEVELS, format } from './logger.js';
log(format("System started"));

// Default anonymous class
export default class {
  constructor(id) { this.id = id; }
  toString() { return `Item(${this.id})`; }
}

// Re-exporting default
export { default as MyComponent } from './MyComponent.js';
```

### Advanced Examples
```javascript
// Named function as default (still exported as default)
function internalName() { return "result"; }
export default internalName; // exported as default, not internalName

// Default export of an arrow function
export default (a, b) => a + b;

// Default export with instantiation
const connection = createConnection();
export default connection;

// Barrel re-export of default
export { default } from './module.js';
export { default as CustomName } from './module.js';

// Dynamic import of default
const module = await import('./module.js');
const defaultExport2 = module.default;
```

### Real-World Use Cases
- React component exports (`export default function App() {}`)
- Single-function utility modules
- Main service class in a module
- Configuration object exports
- Redux reducer exports (`export default reducer`)
- Express router exports

### Common Mistakes
- Having multiple default exports in one module (SyntaxError)
- Confusing `export default` with `export { default }`
- Forgetting that `export default const` is invalid (use `export default` + declaration)
- Mistaking default export name for the actual export name
- Using default exports when named exports would be clearer

### Best Practices
- Use default exports for the primary value of a module
- Use named exports for additional utilities
- Consider using named exports only for better tree-shaking
- Be consistent within a codebase (default vs named preference)
- Name default exports in function/class declarations for better stack traces
- Prefer `export default function` over anonymous default

### Performance Considerations
- Default exports may prevent tree-shaking of unused named siblings
- Bundlers handle default exports efficiently
- No runtime performance difference between default and named
- Dynamic imports of default exports work the same as named

### Interview Questions
1. How many default exports can a module have?
2. What is the syntax for combining default and named imports?
3. How do you re-export a default export?
4. Can you use `export default const`?
5. How do you import a default export with a different name?

### Coding Challenges
1. Create a module with both default and named exports.
2. Import a default export with a different name in the importing module.
3. Re-export a default export from a barrel file.
4. Write a module that exports a class as default and helper functions as named exports.

### Related Topics
- Named exports
- import statement
- Module design patterns

## Named exports

### What It Is
Named exports allow a module to export multiple named values. Each export has a specific name that must be used when importing.

### Why It Is Important
Named exports enable modules to expose multiple functions, constants, or classes. They provide explicit naming that improves code discoverability, auto-completion, and tree-shaking.

### How It Works Internally
Each named export creates a binding with the specified name in the module record. Imports must match the exact export name (or use aliases). The binding is live.

### Syntax
```javascript
// Inline
export const name = value;
export function func() {}
export class Class {}

// At end
const name = value;
function func() {}
export { name, func };
export { name as alias };
```

### Beginner Examples
```javascript
// math.js
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }

// app.js
import { PI, add, subtract } from './math.js';
console.log(add(PI, 1)); // 4.14159

// Alias imports
import { add as sum } from './math.js';
console.log(sum(2, 3)); // 5

// Alias exports
const internalName = "value";
export { internalName as publicName };
```

### Intermediate Examples
```javascript
// Re-exporting named exports
export { add, subtract } from './arithmetic.js';
export * from './constants.js';

// Re-export with rename
export { add as arithmeticAdd } from './arithmetic.js';

// Grouped exports
const VERSION = "1.0.0";
const API_URL = "https://api.example.com";
const TIMEOUT = 5000;
export { VERSION, API_URL, TIMEOUT };

// Selective re-export (excluding some)
export { add, subtract } from './math.js';
// Not exporting multiply

// Exporting multiple constants
export const HTTP_STATUS = {
  OK: 200,
  NOT_FOUND: 404,
  SERVER_ERROR: 500
};
```

### Advanced Examples
```javascript
// Exporting live bindings
export let counter = 0;
export function increment() {
  counter++; // All importers see the updated value
}

// Dynamic re-export pattern
// This exports are determined at module load
const services = {
  user: UserService,
  auth: AuthService
};
export const getService = (name) => services[name];

// Exporting functions with consistent naming
export const formatDate = (date) => { /* ... */ };
export const formatNumber = (num) => { /* ... */ };
export const formatCurrency = (amount) => { /* ... */ };

// Named export of default import
export { default as MyComponent } from './MyComponent.js';
export { default as MyComponent, helper } from './MyComponent.js';

// Converting CommonJS to ESM (with warnings)
// export { default } from 'cjs-module'; // May not work as expected
```

### Real-World Use Cases
- Utility libraries (lodash-style)
- API service modules
- Constants and configuration
- Helper functions
- Type definitions
- Component sets
- React hook libraries
- Redux action creators and selectors

### Common Mistakes
- Not importing with the correct name
- Forgetting to destructure named imports (using whole object)
- Confusing named imports with default import syntax
- Creating too many named exports in a single module
- Not using aliases when names conflict
- Exporting mutable objects thinking they're immutable

### Best Practices
- Prefer named exports for most use cases
- Keep module focused (3-7 exports is reasonable)
- Use consistent naming conventions (camelCase)
- Use re-exports for organizing larger libraries
- Use barrel files to aggregate named exports
- Document each export's purpose with JSDoc
- Use aliases for external-facing API names
- Avoid `export *` as it may export unexpected names

### Performance Considerations
- Named exports enable precise tree-shaking
- Unused named exports are eliminated by bundlers
- No runtime overhead compared to default exports
- Re-exports via `export *` may prevent tree-shaking
- Barrel files should re-export explicitly for best optimization

### Interview Questions
1. How do you rename a named export?
2. What is the difference between `export { a, b }` and `export * from './module'`?
3. Can you re-export a default export as a named export?
4. How does tree-shaking benefit from named exports?
5. What happens if you import a named export that doesn't exist?

### Coding Challenges
1. Create a module with multiple named exports and re-export them from a barrel.
2. Write a module that exports live bindings (counter pattern).
3. Implement a utility module with named exports for string formatting.
4. Create a module that re-exports specific named exports from another module.

### Related Topics
- Default exports
- import statement
- Module aggregation
- Tree-shaking

## Dynamic imports

### What It Is
Dynamic imports use the `import()` function-like expression to load modules asynchronously at runtime. Unlike static imports, dynamic imports can be conditional and lazy.

### Why It Is Important
Dynamic imports enable code splitting, lazy loading, and conditional module loading. They are essential for reducing initial bundle sizes in web applications and loading modules based on runtime conditions.

### How It Works Internally
`import()` returns a Promise that resolves to a module namespace object (containing all exports). The browser/Node.js fetches and evaluates the module on demand. Dynamic imports trigger separate network requests in browsers.

### Syntax
```javascript
// Basic dynamic import
const module = await import('./module.js');

// With expression
const name = 'module';
const module = await import(`./${name}.js`);

// Destructuring
const { namedExport, default: defaultExport } = await import('./module.js');

// Promise style
import('./module.js').then(module => { /* ... */ });
```

### Beginner Examples
```javascript
// Basic dynamic import
async function loadModule() {
  const utils = await import('./utils.js');
  console.log(utils.double(5));
  console.log(utils.square(3));
}

// Conditional loading
if (condition) {
  const module = await import('./feature.js');
  module.initialize();
}

// Dynamic default import
const { default: greet } = await import('./greet.js');
console.log(greet("Alice"));

// With error handling
try {
  const config = await import('./config.js');
  applyConfig(config.default);
} catch (error) {
  console.error("Failed to load config", error);
}
```

### Intermediate Examples
```javascript
// Lazy loading in event handlers
button.addEventListener('click', async () => {
  const { showModal } = await import('./modal.js');
  showModal({ title: 'Hello' });
});

// Dynamic import with template string
const locale = navigator.language.split('-')[0];
try {
  const messages = await import(`./i18n/${locale}.js`);
  applyTranslations(messages.default);
} catch {
  const messages = await import('./i18n/en.js');
  applyTranslations(messages.default);
}

// Dynamic import with cache busting
const module = await import(`./module.js?v=${Date.now()}`);

// Importing multiple modules in parallel
const [moduleA, moduleB, moduleC] = await Promise.all([
  import('./a.js'),
  import('./b.js'),
  import('./c.js')
]);
```

### Advanced Examples
```javascript
// Dynamic plugin system
class PluginManager {
  constructor() {
    this.plugins = new Map();
  }
  
  async loadPlugin(name) {
    try {
      const plugin = await import(`./plugins/${name}.js`);
      const instance = plugin.default;
      await instance.initialize(this);
      this.plugins.set(name, instance);
      return instance;
    } catch (error) {
      console.error(`Failed to load plugin ${name}:`, error);
      throw error;
    }
  }
  
  async loadAll(pluginNames) {
    return Promise.all(pluginNames.map(name => this.loadPlugin(name)));
  }
}

// Route-based code splitting (React-like)
const routes = {
  '/home': () => import('./pages/Home.js'),
  '/about': () => import('./pages/About.js'),
  '/contact': () => import('./pages/Contact.js')
};

async function loadRoute(path) {
  const loader = routes[path];
  if (!loader) throw new Error(`Route not found: ${path}`);
  const module = await loader();
  return module.default;
}

// Prefetching
const prefetched = import('./heavily-used-module.js'); // Start loading early
// ... later:
const module = await prefetched; // Already loaded

// Import with timeout
function importWithTimeout(specifier, timeout = 5000) {
  return Promise.race([
    import(specifier),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error(`Import timeout: ${specifier}`)), timeout)
    )
  ]);
}
```

### Real-World Use Cases
- Route-based code splitting in SPAs (React Router, Vue Router)
- Lazy loading heavy components (charts, editors, maps)
- Loading polyfills conditionally based on browser support
- i18n translation loading based on user locale
- Plugin architectures
- Feature flags (load features on demand)
- Loading configuration dynamically
- Server-side module loading (Node.js)
- A/B testing module variants

### Common Mistakes
- Using dynamic imports for critical path dependencies (use static)
- Not handling dynamic import rejections
- Creating too many small dynamic imports (network overhead)
- Using `import()` in non-async contexts without `.then()`
- Forgetting that dynamic imports return a module namespace object
- Overusing dynamic imports when static would work

### Best Practices
- Use dynamic imports for code splitting (routes, heavy components)
- Handle errors with try/catch
- Use `Promise.all` for parallel dynamic imports
- Consider prefetching critical dynamic imports
- Use dynamic imports for conditional/optional dependencies
- Avoid dynamic imports for critical application startup code
- Use bundler-specific comments for chunk naming: `import(/* webpackChunkName: "chart" */ './chart.js')`
- Monitor dynamic import performance in production

### Performance Considerations
- Dynamic imports add network round-trips
- Each `import()` triggers a separate HTTP request (in browsers)
- Large dynamic modules should be prefetched
- Use preload/prefetch hints for important dynamic chunks
- Dynamic imports can cause layout shifts if not handled well
- Parallel dynamic imports reduce total load time
- Bundle splitting granularity affects caching efficiency

### Interview Questions
1. How does `import()` differ from static `import`?
2. How do you handle errors with dynamic imports?
3. How do dynamic imports enable code splitting?
4. How do you import both default and named exports dynamically?
5. Can you use dynamic imports in service workers?

### Coding Challenges
1. Implement lazy loading for a heavy module based on user interaction.
2. Create a simple route-based code splitting system.
3. Write a function that loads a module with a timeout.
4. Implement a plugin system using dynamic imports.
5. Create a prefetch utility that preloads modules without blocking.

### Related Topics
- Static imports
- Code splitting
- Lazy loading
- Module bundlers
