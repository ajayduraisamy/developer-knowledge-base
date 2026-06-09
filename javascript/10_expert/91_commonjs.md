# CommonJS - require(), module.exports, runtime resolution, vs ES modules

## Introduction

CommonJS (CJS) is the module system originally designed for server-side JavaScript and popularized by Node.js. It uses `require()` for importing and `module.exports` or `exports` for exporting. Before ES modules became standardized, CommonJS was the de facto standard for Node.js packages. Understanding CommonJS is essential for maintaining legacy Node.js applications and understanding the npm ecosystem.

## require() Function

### What It Is

`require()` is Node.js's synchronous function for importing modules. It takes a module identifier (path or package name) and returns the exported value of the target module. `require()` is synchronous because it reads files from disk, evaluates them, and caches the result.

### Why It Is Important

Most npm packages still use CommonJS or provide CommonJS builds. Understanding `require()` is essential for debugging import issues and understanding module caching behavior. The synchronous nature of `require()` has significant implications for application startup and circular dependency handling.

### How It Works Internally

When `require()` is called:
1. **Resolve**: Convert the module specifier to an absolute file path
2. **Check cache**: If the module has been loaded before, return the cached exports
3. **Create module**: Create a new `Module` instance
4. **Cache module**: Store in cache before evaluation (to handle circular deps)
5. **Evaluate**: Read the file, wrap it in a function, execute it with `exports`, `require`, `module`, `__filename`, `__dirname`
6. **Return**: Return `module.exports`

```javascript
// Simplified require() implementation
function require(modulePath) {
  const resolvedPath = Module._resolveFilename(modulePath);
  
  if (Module._cache[resolvedPath]) {
    return Module._cache[resolvedPath].exports;
  }
  
  const module = new Module(resolvedPath);
  Module._cache[resolvedPath] = module;
  
  module.load(resolvedPath); // Reads, wraps, and evaluates
  
  return module.exports;
}
```

### Syntax

```javascript
// Basic require
const fs = require('fs');
const path = require('path');
const express = require('express');
const myModule = require('./my-module');
const utils = require('../utils/helper');

// require with destructuring
const { readFile, writeFile } = require('fs');

// require JSON files
const config = require('./config.json');

// require with full path
const module = require('/absolute/path/to/module.js');

// require directory (loads index.js)
const models = require('./models');

// require and execute immediately
const result = require('./script')(); // If it exports a function
```

### Beginner Examples

```javascript
// Basic module usage

// my-module.js
const message = 'Hello from module!';
function greet(name) {
  return `${message} Hi, ${name}!`;
}
// Set what require() returns
module.exports = greet;

// app.js
const greet = require('./my-module.js');
console.log(greet('Alice')); // 'Hello from module! Hi, Alice!'

// Multiple exports with object
module.exports = {
  greet,
  message,
  farewell: (name) => `Goodbye, ${name}!`
};

// app.js
const myModule = require('./my-module.js');
console.log(myModule.greet('Alice'));
console.log(myModule.farewell('Bob'));
```

### Intermediate Examples

```javascript
// Module caching
// Module is only evaluated once
const dep1 = require('./dependency'); // Evaluated
const dep2 = require('./dependency'); // Cached (same object)
console.log(dep1 === dep2); // true

// First require evaluates the module
// Subsequent requires return cached exports

// Avoiding caching (fresh evaluation)
delete require.cache[require.resolve('./dependency')];
const fresh = require('./dependency'); // Re-evaluated

// Circular dependency with require
// a.js
console.log('a: start');
const b = require('./b');
console.log('a: b loaded', b);
exports.aValue = 'from a';
exports.getB = () => b;
console.log('a: end');

// b.js
console.log('b: start');
const a = require('./a');
console.log('b: a loaded', a);
exports.bValue = 'from b';
exports.getA = () => a;
console.log('b: end');

// Output:
// a: start
// b: start
// b: a loaded { aValue: 'from a', getB: [Function] }
// b: end
// a: b loaded { bValue: 'from b', getA: [Function] }
// a: end

// Note: a is partially loaded when b requires it
// But both values are accessible after full evaluation
```

### Advanced Examples

```javascript
// Custom require resolution
const Module = require('module');
const originalResolve = Module._resolveFilename;

Module._resolveFilename = function(request, parent) {
  // Custom resolution logic
  if (request.startsWith('@my-scope/')) {
    // Map to custom directory
    const mappedPath = request.replace('@my-scope/', '/opt/shared-modules/');
    return mappedPath;
  }
  return originalResolve.call(this, request, parent);
};

// Module._resolveLookupPaths for custom paths
const path = require('path');
Module._resolveLookupPaths = function(request, parent) {
  // Add custom lookup paths
  const original = Module._resolveLookupPaths(request, parent) || [];
  return [...original, '/opt/shared-modules'];
};

// require.resolve for checking existence
const resolvedPath = require.resolve('some-package');
console.log('Package found at:', resolvedPath);

// Conditional requires
let driver;
try {
  driver = require('pg');
} catch (e) {
  driver = require('sqlite3');
}

// Require with node_modules depth
// Node.js searches node_modules from current directory upward
// Stops at root node_modules or when package found
```

## module.exports and exports

### What It Is

`module.exports` is the object that `require()` returns. Initially, it's an empty object `{}`. `exports` is a shorthand reference to `module.exports`. Setting properties on `exports` modifies `module.exports`, but reassigning `exports` breaks the reference.

### Why It Is Important

The distinction between `exports` and `module.exports` is a common source of confusion. Understanding the reference relationship prevents bugs where a module appears to export nothing.

### How It Works Internally

Node.js wraps each module in a function:
```javascript
(function(exports, require, module, __filename, __dirname) {
  // Module code here
});
```

Initially, `exports` and `module.exports` reference the same object. When you assign a property to `exports`, it adds to the shared object. But if you assign a new value to `exports`, it no longer points to `module.exports`, and the new object is ignored by `require()`.

### Syntax

```javascript
// Works: setting properties on exports
exports.add = (a, b) => a + b;
exports.subtract = (a, b) => a - b;

// Equivalent to:
module.exports.add = (a, b) => a + b;
module.exports.subtract = (a, b) => a - b;

// Does NOT work: reassigning exports
exports = (a, b) => a + b; // Broken! Module.exports still is {}

// Works: reassigning module.exports
module.exports = (a, b) => a + b;

// Common pattern: export single function
module.exports = function greet(name) {
  return `Hello, ${name}!`;
};

// Export constructor
module.exports = class Person {
  constructor(name) { this.name = name; }
};

// Export instance
module.exports = new Person('Alice');
```

### Beginner Examples

```javascript
// Various export patterns

// Pattern 1: Add to exports object
// calculator.js
exports.add = (a, b) => a + b;
exports.subtract = (a, b) => a - b;
exports.multiply = (a, b) => a * b;

// main.js
const calc = require('./calculator');
console.log(calc.add(2, 3)); // 5

// Pattern 2: Replace module.exports entirely
// greet.js
module.exports = function(name) {
  return `Hello, ${name}!`;
};

// main.js
const greet = require('./greet');
console.log(greet('Alice')); // 'Hello, Alice!'

// Pattern 3: Export multiple with object literal
// utils.js
module.exports = {
  formatDate: (d) => d.toISOString().split('T')[0],
  capitalize: (s) => s.charAt(0).toUpperCase() + s.slice(1),
  range: (n) => Array.from({length: n}, (_, i) => i)
};
```

### Intermediate Examples

```javascript
// Exports reassignment trap
// BUG: This does NOT export
exports = {
  value: 42,
  method() { return this.value; }
};
// module.exports is still {}!

// FIX: Assign to module.exports
module.exports = {
  value: 42,
  method() { return this.value; }
};

// Combining exports and module.exports
exports.publicApi = function() {
  return privateHelper();
};

function privateHelper() {
  return 'private'; // Not exported
}

module.exports = {
  ...module.exports, // Include exports properties
  extra: 'also exported'
};

// exports shorthand utility
// When you use exports.fn = ... it's the same as module.exports.fn = ...
// Both are fine; choose one style consistently

// Checking what gets exported
// counter.js
let count = 0;
exports.increment = () => ++count;
exports.decrement = () => --count;
exports.getCount = () => count;

// The count variable is private (not exported)
// Only the three methods are accessible via require
```

### Advanced Examples

```javascript
// Exporting ES6 classes
class Database {
  constructor(options) { /* ... */ }
  async query(sql) { /* ... */ }
  async connect() { /* ... */ }
}

module.exports = Database;
// Can also add static methods
Database.create = async function(options) {
  const db = new Database(options);
  await db.connect();
  return db;
};

// Prototype extension through exports
exports.ArrayUtils = {};
exports.ArrayUtils.first = (arr) => arr[0];
exports.ArrayUtils.last = (arr) => arr[arr.length - 1];
exports.ArrayUtils.pluck = (arr, key) => arr.map(item => item[key]);

// Singleton pattern with module caching
// config.js
class Config {
  constructor() {
    this.settings = {};
  }
  get(key) { return this.settings[key]; }
  set(key, value) { this.settings[key] = value; }
}

module.exports = new Config(); // Singleton!
// All require('./config') return the same instance

// Replacing exports with function + properties
module.exports = createConnection;
module.exports.defaults = { host: 'localhost', port: 3000 };
module.exports.close = (conn) => conn.end();

function createConnection(options) {
  return { ...module.exports.defaults, ...options };
}
```

## Runtime Module Resolution

### What It Is

CommonJS module resolution determines which file a `require()` call loads. The resolution algorithm searches `node_modules` directories, handles file extensions, and resolves directories to `index.js` files. Resolution is synchronous and happens at runtime (when `require()` is called).

### Why It Is Important

Understanding resolution helps debug "module not found" errors, configure custom resolution, and manage dependency versions. The resolution algorithm is also why npm's nested dependency tree works.

### How It Works Internally

1. If the specifier starts with `'/'`, it's an absolute path
2. If it starts with `'./'` or `'../'`, it's a relative path
3. Otherwise, it's a "bare" specifier resolved from `node_modules`

For relative/absolute paths, Node.js tries:
- Exact file with extension
- `.js`, `.json`, `.node` extensions appended
- Directory with `index.js`, `index.json`, `index.node`

For bare specifiers, Node.js searches:
- Current module's `node_modules`
- Parent directory's `node_modules`
- ...up to root

### Syntax

```javascript
// Resolution order for require('./utils')
// 1. ./utils (as file)
// 2. ./utils.js
// 3. ./utils.json
// 4. ./utils.node
// 5. ./utils/index.js
// 6. ./utils/index.json
// 7. ./utils/index.node

// Resolution order for require('lodash')
// 1. ./node_modules/lodash
// 2. ../node_modules/lodash
// 3. ../../node_modules/lodash
// (continues upward)

// package.json fields checked for bare specifiers
// 1. "exports" (Node.js 12+)
// 2. "main" (traditional)
// 3. "index.js" (if no main)
```

### Beginner Examples

```javascript
// Understanding resolution paths
const modulePath = require.resolve('lodash');
console.log(modulePath); // Full path to lodash/index.js

// Check which file resolves
console.log(require.resolve('./utils'));
// cwd/src/utils.js

// Adding module search paths
// In app.js
const path = require('path');
require.main.paths.push(path.join(__dirname, 'custom_modules'));
// Now require('my-module') will also look in ./custom_modules

// NODE_PATH environment variable
// export NODE_PATH=/usr/lib/node_modules
// Node checks NODE_PATH paths after node_modules
```

### Intermediate Examples

```javascript
// Package resolution with conditional exports (Node.js 12+)
// my-package/package.json
{
  "main": "./dist/index.cjs.js",
  "exports": {
    ".": {
      "require": "./dist/index.cjs.js",
      "node": "./dist/index.node.js",
      "default": "./dist/index.js"
    },
    "./feature": {
      "require": "./dist/feature.cjs.js",
      "default": "./dist/feature.js"
    }
  }
}

// Resolving subpath exports
const feature = require('my-package/feature');
// Uses exports["./feature"].require → dist/feature.cjs.js

// require.resolve.paths
console.log(module.paths);
// Shows all node_modules paths that will be searched

// Package resolution with main field
// package.json resolution order:
// 1. exports (highest priority)
// 2. main (fallback)
// 3. index.js in package root (last resort)
```

### Advanced Examples

```javascript
// Custom module resolution with proxyquire (testing)
const proxyquire = require('proxyquire');

const stubs = {
  'fs': {
    readFile: (path, cb) => cb(null, 'mocked content')
  }
};

const myModule = proxyquire('./my-module', stubs);
// my-module's require('fs') returns stubbed version

// require with symlinks
// By default, Node.js resolves through symlinks
// --preserve-symlinks flag for alternative behavior

// Resolving with self-referencing
// If package name is "my-package"
// my-package/src/index.js:
const utils = require('my-package/src/utils');
// Resolves within the same package

// Custom resolveAlgorithm (hooking Module._resolveFilename)
const Module = require('module');
const original = Module._resolveFilename;
Module._resolveFilename = function(request, parent, isMain) {
  if (request.startsWith('@my-internal/')) {
    const mapped = request.replace('@my-internal/', '/opt/internal/');
    return mapped;
  }
  return original.call(this, request, parent, isMain);
};

// Checking what a module resolves to
function resolvePath(specifier) {
  try {
    return require.resolve(specifier);
  } catch (e) {
    return `Not found: ${e.message}`;
  }
}
```

## CommonJS vs ES Modules

### What It Is

CommonJS and ES Modules are two different module systems with distinct syntax, semantics, and behavior. CommonJS is synchronous, runtime-based, and dynamic. ES Modules are asynchronous, static, and optimized for static analysis. The two systems can interoperate in Node.js under certain conditions.

### Why It Is Important

The ecosystem is transitioning from CommonJS to ES Modules. Most new packages offer both formats. Understanding the differences helps developers choose the right system for their project and debug interop issues.

### Key Differences

| Feature | CommonJS | ES Modules |
|---------|----------|------------|
| Syntax | `require()`, `module.exports` | `import`, `export` |
| Loading | Synchronous | Asynchronous (static) |
| Resolution | Runtime | Static (analyzed at parse time) |
| File extension | `.js` (default), `.cjs` | `.mjs` or `.js` with `"type": "module"` |
| Top-level await | No | Yes |
| Live bindings | No (value copies) | Yes (reference bindings) |
| Tree shaking | Not possible | Supported |
| Circular deps | Partial exports | Live bindings handle better |
| `this` at top level | `exports` object | `undefined` |

### Syntax

```javascript
// CommonJS
const fs = require('fs');
const { readFile } = require('fs');
module.exports = myFunction;
exports.helper = helper;

// ES Modules
import fs from 'fs';
import { readFile } from 'fs';
export default myFunction;
export const helper = helper;

// Interop
// ESM can import CJS (default import = module.exports)
import pkg from 'cjs-package';
// CJS can import ESM with dynamic import() (async)
async function load() {
  const mod = await import('esm-package');
}
```

### Beginner Examples

```javascript
// File extension conventions

// .cjs = always CommonJS
// my-module.cjs
module.exports = { value: 42 };

// .mjs = always ESM
// my-module.mjs
export const value = 42;

// .js = depends on package.json "type" field
// package.json
{
  "type": "module"  // All .js files are ESM
}
// Or:
{
  "type": "commonjs" // All .js files are CJS (default)
}

// Mixing CJS and ESM in same project
// Use .cjs for CJS files when "type": "module"
// Use .mjs for ESM files when "type": "commonjs"
```

### Intermediate Examples

```javascript
// ESM importing CJS
// cjs-lib/index.js (CommonJS)
module.exports = {
  name: 'CJS Lib',
  version: '1.0.0',
  greet() { return 'Hello!'; }
};
module.exports.extra = 'extra value';

// app.mjs (ESM)
import cjsLib from 'cjs-lib';
console.log(cjsLib.name); // 'CJS Lib'
console.log(cjsLib.greet()); // 'Hello!'
// module.exports becomes the default export

// Named imports from CJS (limited)
// Only works for static named exports (not all bundlers)
import { name, greet } from 'cjs-lib';
// May work in Node.js (named exports detection)

// CJS importing ESM (async only)
// esm-lib/index.mjs
export const value = 42;
export function help() { return 'helping'; }
export default 'default export';

// app.cjs (CommonJS)
async function loadEsm() {
  const esmLib = await import('esm-lib');
  console.log(esmLib.value); // 42
  console.log(esmLib.help()); // 'helping'
  console.log(esmLib.default); // 'default export'
}
loadEsm();
```

### Advanced Examples

```javascript
// Package dual packaging (CJS + ESM)
// my-package/package.json
{
  "name": "my-package",
  "main": "./dist/index.cjs.js",
  "module": "./dist/index.esm.js",
  "exports": {
    ".": {
      "import": "./dist/index.esm.js",
      "require": "./dist/index.cjs.js"
    }
  },
  "type": "module"
}

// Build both formats (Rollup example)
// rollup.config.js
export default [
  { input: 'src/index.js', output: { format: 'cjs', file: 'dist/index.cjs.js' } },
  { input: 'src/index.js', output: { format: 'esm', file: 'dist/index.esm.js' } }
];

// Conditional require/import
// In CJS:
let mod;
try {
  mod = require('esm-lib');
} catch {
  // Fallback
}

// In ESM:
import { createRequire } from 'module';
const require = createRequire(import.meta.url);
const cjsMod = require('cjs-lib');

// Package type scoping
// package.json
{
  "type": "module",
  "exports": {
    ".": "./index.js",     // ESM (because type: module)
    "./legacy": {
      "require": "./legacy.cjs"  // Explicit CJS
    }
  }
}
```

### Real-World Use Cases

- Developing dual-format npm packages
- Migrating CJS codebase to ESM gradually
- Using ESM-only libraries in CJS projects (via dynamic import)
- Maintaining legacy Node.js applications
- Creating reusable modules for both Node.js and browsers

### Common Mistakes

- Using `require()` in ESM (throws ReferenceError)
- Using `import` at top level in CJS (syntax error)
- Assuming ESM can `require()` synchronously
- Forgetting file extensions in ESM imports
- Not configuring `"type": "module"` in package.json
- Mixing up `exports` and `module.exports` in CJS
- Expecting tree shaking to work with CJS

### Best Practices

- Use ESM for new projects (it's the standard)
- Provide both CJS and ESM for published packages
- Use `.cjs` and `.mjs` extensions for clarity
- Configure `"exports"` in package.json for Node.js 12+
- Use `createRequire` for selective CJS usage in ESM
- Avoid `require()` in hot paths (synchronous I/O blocks event loop)
- Use dynamic `import()` in ESM for conditional loading

### Interview Questions

**Q: What is the difference between `exports` and `module.exports`?** A: `exports` is a shorthand reference to `module.exports`. Initially both point to the same object. Setting `exports.prop = value` modifies both, but reassigning `exports = newValue` breaks the reference—only `module.exports` determines what `require()` returns.

**Q: Why can't CommonJS `require()` be used in ES modules?** A: ESM is designed for static analysis with asynchronous module loading. `require()` is synchronous and dynamic. The module resolution and loading model of ESM (phases: parse → link → evaluate) is fundamentally different from CJS (synchronous, immediate evaluation).


### Performance Considerations
- require() is synchronous and blocks the event loop; avoid in performance-critical paths
- Module caching reduces repeated I/O but increases memory usage
- Circular dependencies cause partial module exports which can return undefined
- Dynamic require() prevents static analysis and tree-shaking
- CommonJS modules cannot be tree-shaken by bundlers due to dynamic nature


### Coding Challenges
```javascript
// Challenge 1: Implement a simple require() function
function myRequire(modulePath) {
  const fs = require('fs');
  const path = require('path');
  const code = fs.readFileSync(modulePath, 'utf8');
  const module = { exports: {} };
  const wrapper = new Function('module', 'exports', 'require', '__dirname', '__filename', code);
  wrapper(module, module.exports, myRequire, path.dirname(modulePath), modulePath);
  return module.exports;
}

// Challenge 2: Demonstrate circular dependency issue
// a.js
// module.exports.a = 1;
// const b = require('./b');
// module.exports.a2 = 2;

// b.js
// module.exports.b = 1;
// const a = require('./a'); // Gets incomplete export { a: 1 }
// module.exports.b2 = a.a + 1; // Works, but a.a2 is undefined

// Challenge 3: Module caching demonstration
const mod1 = require('./my-module');
const mod2 = require('./my-module');
console.log(mod1 === mod2); // true - same cached instance
```

### Related Topics

- ES module system (import/export)
- Node.js module architecture
- Package.json exports field
- Module caching and singleton patterns
- Dual package hazard
- Dynamic import() in ESM
- Module bundling and tree shaking
- Node.js --experimental-modules history
- require.resolve and module resolution
- Interop between CJS and ESM in TypeScript
