# ES Modules - import/export, static vs dynamic, module resolution, circular dependencies

## Introduction

ES Modules (ESM) are the official standard module system for JavaScript, introduced in ES2015 (ES6). They provide a static module structure that enables tree shaking, deterministic module resolution, and cyclic dependency handling. ESM has become the universal module format, supported natively in browsers, Node.js, Deno, and Bun.

## Static import/export

### What It Is

Static imports and exports are the primary syntax of ES modules. `import` declarations load bindings from other modules, and `export` declarations expose bindings to other modules. Both are static: they must be at the top level of the module (not inside functions or conditionals), and the module specifier must be a string literal (not a computed expression).

### Why It Is Important

Static structure enables powerful optimizations. Bundlers can analyze imports/exports without executing code, enabling tree shaking, dead code elimination, and deterministic circular dependency resolution. The static nature also enables "live bindings" where exported values automatically reflect changes in the exporting module.

### How It Works Internally

When a module is loaded, the JavaScript engine first parses it to build a module record containing all exported names. Module linking resolves imports by connecting each import binding to its corresponding export binding. Unlike CommonJS (which copies values), ESM creates live bindings: importing modules see the current value of the exported variable in the source module.

Modules go through three phases:
1. **Parsing**: Parse source, detect errors, build module records
2. **Linking**: Match imports to exports, create bindings
3. **Evaluation**: Execute module bodies in order

### Syntax

```javascript
// Named exports
export const PI = 3.14159;
export function multiply(a, b) { return a * b; }
export class Calculator { /* ... */ }

// Named imports
import { PI, multiply, Calculator } from './math.js';

// Default export
export default function greet(name) {
  return `Hello, ${name}!`;
}

// Default import
import greet from './greeting.js';

// Mixing named and default
import greet, { PI, multiply } from './utils.js';

// Renaming imports/exports
export { internalName as publicName };
import { publicName as localName } from './module.js';

// Re-exporting
export { multiply } from './math.js';
export * from './utils.js'; // Wildcard re-export

// Export namespace
import * as math from './math.js';
math.multiply(2, 3);
```

### Beginner Examples

```javascript
// math.js
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
const internal = 'not exported'; // Private to this module
export default class Calculator {
  static run(operation, a, b) {
    return this[operation](a, b);
  }
}

// app.js
// Named imports (exact names from math.js)
import { add, subtract } from './math.js';

// Default import
import Calculator from './math.js';

console.log(add(5, 3)); // 8
console.log(Calculator.run('add', 5, 3)); // 8
// console.log(internal); // ReferenceError: not accessible

// Renaming imports
import { add as sum, subtract as diff } from './math.js';
console.log(sum(10, 5)); // 15
```

### Intermediate Examples

```javascript
// Live bindings (EVM vs CJS)
// module.js
export let count = 0;
export function increment() {
  count++;
}

// main.js
import { count, increment } from './module.js';
console.log(count); // 0
increment();
console.log(count); // 1 (live binding reflects change!)

// Compare with CommonJS:
// main.cjs
const { count, increment } = require('./module.cjs');
console.log(count); // 0
increment();
console.log(count); // 0 (copy, not binding!)

// Re-export patterns
// index.js (barrel file)
export { default as Button } from './Button.js';
export { default as Input } from './Input.js';
export { validateEmail, formatDate } from './utils.js';

// App.js
import { Button, Input, validateEmail } from './components/index.js';

// namespace import
import * as Components from './components/index.js';
console.log(Components.Button);
console.log(Components.Input);

// import for side effects only
import './polyfill.js'; // Runs the module for its side effects
import './styles.css'; // CSS modules (with bundler)

// import.meta
console.log(import.meta.url); // File URL of current module
console.log(import.meta.resolve('./other.js')); // Resolve specifier
```

### Advanced Examples

```javascript
// Conditional exports (package.json)
// package.json
{
  "exports": {
    ".": {
      "import": "./dist/index.esm.js",
      "require": "./dist/index.cjs.js"
    },
    "./feature": {
      "import": "./dist/feature.esm.js",
      "require": "./dist/feature.cjs.js"
    }
  }
}

// User imports:
import { feature } from 'my-package/feature';
// Resolves to dist/feature.esm.js

// Self-referencing imports
// A package can import from itself using its own name
// package name: "my-lib"
// src/index.js
import { helper } from 'my-lib/helper.js';
// This resolves within the package itself

// import assertions and attributes (import JSON, CSS)
import data from './data.json' assert { type: 'json' };
// import styles from './styles.css' assert { type: 'css' };

// WebAssembly modules (import)
import { memory, add } from './math.wasm';
console.log(add(5, 3)); // 8

// Preserving export order
// The order of imports/exports doesn't matter for value resolution
// But evaluation order follows dependency graph (topological)
```

### Coding Challenges

**Challenge 1: Build a static import validator**
```javascript
// Create a tool that validates static import/export statements
// and detects common issues.

class StaticImportValidator {
  constructor(source) {
    this.source = source;
    this.lines = source.split('\n');
    this.issues = [];
    this.exports = new Map();
    this.imports = [];
  }

  parse() {
    for (const line of this.lines) {
      const trimmed = line.trim();

      // Track named exports
      const namedExport = trimmed.match(/export\s+(?:const|let|var|function|class)\s+(\w+)/);
      if (namedExport) {
        this.exports.set(namedExport[1], { type: 'named', line: this.lines.indexOf(line) + 1 });
      }

      // Track default export
      if (trimmed.startsWith('export default')) {
        this.exports.set('default', { type: 'default', line: this.lines.indexOf(line) + 1 });
      }

      // Track imports
      const importMatch = trimmed.match(/import\s+(?:\{([^}]+)\}|\*\s+as\s+(\w+)|\w+)\s+from\s+['"]([^'"]+)['"]/);
      if (importMatch) {
        this.imports.push({
          names: importMatch[1] ? importMatch[1].split(',').map(s => s.trim()) : [importMatch[2] || 'default'],
          source: importMatch[3],
          line: this.lines.indexOf(line) + 1
        });
      }

      // Check for CommonJS mixed usage
      if (trimmed.startsWith('module.exports') || trimmed.startsWith('exports.')) {
        this.addIssue('error', `Line ${this.lines.indexOf(line) + 1}: Mixed CommonJS/ESM syntax. Use export instead.`);
      }

      // Check for dynamic require
      if (trimmed.includes('require(') && trimmed.includes('import')) {
        this.addIssue('warning', `Line ${this.lines.indexOf(line) + 1}: Mixing require and import in the same module.`);
      }
    }
  }

  addIssue(severity, message) {
    this.issues.push({ severity, message });
  }

  validate() {
    this.parse();

    // Check for unused imports
    for (const imp of this.imports) {
      for (const name of imp.names) {
        if (name === 'default') continue;
        if (!this.source.includes(name + '(') && !this.source.includes(name + '.') && !this.source.includes(name + ' ')) {
          this.addIssue('warning', `Line ${imp.line}: Import '${name}' from '${imp.source}' appears unused.`);
        }
      }
    }

    // Check for export without matching import
    for (const [name, info] of this.exports) {
      let found = false;
      for (const imp of this.imports) {
        if (imp.names.includes(name)) { found = true; break; }
      }
    }

    return this.issues;
  }

  report() {
    this.validate();
    if (this.issues.length === 0) {
      console.log('No import/export issues found.\n');
      return;
    }

    console.log(`Found ${this.issues.length} issue(s):\n`);
    this.issues.forEach(i => {
      const icon = i.severity === 'error' ? '×' : '!';
      console.log(`  ${icon} [${i.severity.toUpperCase()}] ${i.message}`);
    });
  }
}
```

**Challenge 2: Implement a re-export optimizer**
```javascript
// Build a tool that analyzes barrel files (index.js that re-exports)
// and suggests optimization strategies.

class ReExportOptimizer {
  constructor() {
    this.barrels = new Map();
    this.directImports = new Map();
  }

  addModule(name, code) {
    const reExports = [];
    const lines = code.split('\n');

    for (const line of lines) {
      const reExport = line.match(/export\s+\{([^}]+)\}\s+from\s+['"]([^'"]+)['"]/);
      if (reExport) {
        const names = reExport[1].split(',').map(s => s.trim());
        reExports.push({ names, source: reExport[2] });
      }

      const exportAll = line.match(/export\s+\*\s+from\s+['"]([^'"]+)['"]/);
      if (exportAll) {
        reExports.push({ names: ['*'], source: exportAll[1] });
      }
    }

    this.barrels.set(name, reExports);
  }

  addDirectImport(from, to, names) {
    if (!this.directImports.has(to)) {
      this.directImports.set(to, []);
    }
    this.directImports.get(to).push({ from, names });
  }

  analyze() {
    const results = [];

    for (const [name, reExports] of this.barrels) {
      const total = reExports.reduce((s, r) => s + (r.names.includes('*') ? 20 : r.names.length), 0);
      const used = (this.directImports.get(name) || [])
        .reduce((s, i) => s + i.names.length, 0);

      results.push({
        module: name,
        reExported: total,
        directlyUsed: used,
        wasteRatio: total > 0 ? ((total - used) / total * 100).toFixed(0) : '0',
        suggestion: total > used * 2
          ? 'Consider direct imports instead of barrel file'
          : 'Barrel file is reasonable'
      });
    }

    return results;
  }

  report() {
    const results = this.analyze();
    console.log('Re-export Optimization Report:\n');

    for (const r of results) {
      const icon = r.wasteRatio > 50 ? '!' : 'OK';
      console.log(`[${icon}] ${r.module}`);
      console.log(`  Re-exports: ${r.reExported} symbols`);
      console.log(`  Directly used: ${r.directlyUsed} symbols`);
      console.log(`  Waste: ${r.wasteRatio}%`);
      console.log(`  Suggestion: ${r.suggestion}\n`);
    }
  }
}
```

**Challenge 3: Create a tool that converts CommonJS to ES modules**
```javascript
// Build a CJS-to-ESM converter that handles common patterns.

class CJSToESMConverter {
  constructor(code) {
    this.code = code;
    this.exports = new Map();
    this.imports = new Map();
    this.esmCode = '';
  }

  convert() {
    let result = this.code;

    // Convert require() to import
    result = result.replace(
      /const\s+(\w+)\s*=\s*require\(['"]([^'"]+)['"]\)/g,
      (_, name, module) => `import ${name} from '${module}'`
    );

    // Convert destructured require
    result = result.replace(
      /const\s+\{([^}]+)\}\s*=\s*require\(['"]([^'"]+)['"]\)/g,
      (_, names, module) => `import { ${names.trim()} } from '${module}'`
    );

    // Convert module.exports = { ... } to named exports
    result = result.replace(
      /module\.exports\s*=\s*\{([^}]+)\}/g,
      (_, exports) => {
        return exports.split(',').map(e => {
          const [name, value] = e.trim().split(':').map(s => s.trim());
          if (value && value !== name) {
            return `export const ${name} = ${value}`;
          }
          return `export { ${name} }`;
        }).join('\n');
      }
    );

    // Convert module.exports = single value
    result = result.replace(
      /module\.exports\s*=\s*(\w+)/g,
      'export default $1'
    );

    // Convert exports.name = value
    result = result.replace(
      /exports\.(\w+)\s*=\s*(.+)/g,
      'export const $1 = $2'
    );

    // Handle __dirname and __filename (ESM doesn't have these)
    result = result.replace(
      /__dirname/g,
      "import.meta.url ? new URL('.', import.meta.url).pathname : '.'"
    );
    result = result.replace(
      /__filename/g,
      "import.meta.url ? new URL(import.meta.url).pathname : __filename"
    );

    // Handle require.resolve
    result = result.replace(
      /require\.resolve\(['"]([^'"]+)['"]\)/g,
      (_, module) => {
        return `new URL('${module}', import.meta.url).href`;
      }
    );

    this.esmCode = result;
    return this.esmCode;
  }

  getStats() {
    const cjsPatterns = (this.code.match(/require/g) || []).length;
    const esmPatterns = (this.esmCode.match(/import /g) || []).length +
                        (this.esmCode.match(/export /g) || []).length;

    return {
      originalStatements: cjsPatterns,
      convertedStatements: esmPatterns,
      linesOriginal: this.code.split('\n').length,
      linesConverted: this.esmCode.split('\n').length,
      convertSuccess: cjsPatterns > 0 && esmPatterns > 0
    };
  }
}
```

## Dynamic import()

### What It Is

Dynamic `import()` is a function-like expression that loads a module asynchronously at runtime. Unlike static imports, it can be used conditionally, within functions, or with computed paths. It returns a Promise that resolves to the module's namespace object. Dynamic imports enable code splitting and lazy loading.

### Why It Is Important

Dynamic imports are the primary mechanism for code splitting in modern web applications. They allow splitting the bundle into smaller chunks that are loaded on demand, reducing initial page load time. They also enable conditional feature loading, internationalization, and polyfill loading.

### How It Works Internally

When the engine encounters `import()`, it initiates a new module load: fetching the module, parsing it, linking it, and evaluating it. This happens asynchronously. The returned promise resolves to a module namespace object (like `import * as` would produce). The loaded module is cached, so subsequent `import()` of the same specifier returns the cached module.

Bundlers detect `import()` calls and create separate chunks for the dynamically loaded modules and their dependencies.

### Syntax

```javascript
// Basic dynamic import
const mathModule = await import('./math.js');
console.log(mathModule.add(2, 3));

// Dynamic module specifier
const locale = getUserLocale();
const i18n = await import(`./locales/${locale}.js`);

// Conditional loading
if (condition) {
  const { heavyLibrary } = await import('./heavy.js');
  heavyLibrary.process(data);
}

// With error handling
try {
  const module = await import('./feature.js');
  module.initialize();
} catch (err) {
  console.error('Failed to load feature:', err);
}

// Default exports with dynamic import
const { default: Calculator } = await import('./math.js');
const calc = new Calculator();
```

### Beginner Examples

```javascript
// Route-based code splitting (React + React Router)
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./pages/Dashboard.js'));
const Settings = lazy(() => import('./pages/Settings.js'));
const Profile = lazy(() => import('./pages/Profile.js'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
        <Route path="/profile" element={<Profile />} />
      </Routes>
    </Suspense>
  );
}

// Feature gating
async function loadFeature(featureName) {
  try {
    const feature = await import(`./features/${featureName}.js`);
    feature.enable();
  } catch {
    console.log(`${featureName} not available`);
  }
}

// On-demand polyfill loading
if (!('IntersectionObserver' in window)) {
  await import('intersection-observer');
}
```

### Intermediate Examples

```javascript
// Dynamic import with webpack chunk naming
const Chart = React.lazy(() => import(/* webpackChunkName: "chart" */ './Chart'));
const DataTable = React.lazy(() => import(/* webpackChunkName: "table" */ './DataTable'));

// Parallel dynamic imports
const [chart, table, map] = await Promise.all([
  import('./Chart.js'),
  import('./DataTable.js'),
  import('./Map.js')
]);

// Preloading dynamic imports
const link = document.createElement('link');
link.rel = 'modulepreload';
link.href = moduleUrl;
document.head.appendChild(link);
// Later
const module = await import(moduleUrl); // Already cached

// Retry logic for dynamic imports
async function importWithRetry(specifier, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await import(specifier);
    } catch (err) {
      if (i === retries - 1) throw err;
      await new Promise(r => setTimeout(r, 1000 * (i + 1)));
    }
  }
}

// Dynamic import with context/replacement (webpack require.context)
// Not available in standard import() - bundler specific
```

### Advanced Examples

```javascript
// import() in Node.js for conditional module loading
// ESM file that conditionally loads native addon
async function loadNativeModule() {
  try {
    return await import('./native-bindings.node');
  } catch {
    return await import('./js-fallback.js');
  }
}

// Micro-frontend loading
const appRegistry = new Map();

async function registerApp(name, url) {
  const module = await import(url);
  appRegistry.set(name, module);
  return module;
}

async function loadApp(name) {
  let app = appRegistry.get(name);
  if (!app) {
    app = await import(`./apps/${name}/index.js`);
    appRegistry.set(name, app);
  }
  return app;
}

// Dynamic import with module evaluation control
// Some bundlers support import() with custom chunk behavior

// Virtual modules (rollup/vite plugin pattern)
// import(`/__virtual/${moduleName}`);
// Bundler plugin intercepts and provides module content

// Worker imports (Vite)
const worker = new Worker(new URL('./worker.js', import.meta.url));
// This creates a separate chunk for the worker

// Dynamic import in service workers
self.addEventListener('install', (event) => {
  event.waitUntil(
    import('./cache-manager.js').then(({ initCache }) => initCache())
  );
});
```

### Real-World Use Cases

- Route-based code splitting (SPA frameworks)
- On-demand feature loading (admin panels, dashboards)
- Internationalization (lazy-loading locale files)
- Polyfill loading (load only if needed)
- A/B testing (load experiment scripts)
- Plugin/module systems (load extensions at runtime)

### Common Mistakes

- Using dynamic imports for everything (defeats static analysis)
- Not handling loading states (Suspense, loading spinners)
- Forgetting error boundaries for failed imports
- Importing the same module multiple times dynamically
- Not preloading critical dynamic chunks
- Using dynamic imports in server-rendered code without checks

### Best Practices

- Use static imports for critical, immediately needed code
- Use dynamic imports for code splitting and lazy loading
- Implement proper loading states and error boundaries
- Preload important dynamic chunks with `<link rel="modulepreload">`
- Use bundle analyzers to verify chunk sizes
- Avoid dynamic expressions in import() when possible (bundlers analyze statically)
- Name chunks explicitly for debugging: `import(/* webpackChunkName: "admin" */ './admin')`

### Performance Considerations

- Dynamic imports create a separate network request, adding latency to the first use
- Each dynamic import() call resolves the module specifier at runtime, adding overhead
- Browser preload scanners cannot discover dynamically imported modules
- Module duplication can occur when the same module is dynamically imported in multiple chunks
- Caching dynamic imports requires careful Service Worker configuration (cache-first strategy)
- Chunk size matters: chunks under 50KB may not be worth splitting due to request overhead
- HTTP/2 multiplexing reduces the penalty of multiple chunk requests
- Preload critical dynamic chunks with `<link rel="modulepreload">` to hide network latency
- Dynamic imports with template literals (`${}`) defeat bundler static analysis entirely
- Use `/* webpackPreload: true */` or `/* webpackPrefetch: true */` for priority hints

### Coding Challenges

**Challenge 1: Build a dynamic import performance tracker**
```javascript
// Create a tool that monitors dynamic import timing and
// reports loading performance metrics.

class DynamicImportTracker {
  constructor() {
    this.imports = new Map();
    this.observers = new Set();
  }

  async load(specifier, name) {
    const key = name || specifier;
    const startTime = performance.now();

    try {
      const module = await import(specifier);
      const loadTime = performance.now() - startTime;

      this.imports.set(key, {
        specifier,
        loadTime,
        success: true,
        timestamp: Date.now(),
        size: null
      });

      this.notifyObservers({ key, loadTime, success: true });

      // Estimate chunk size (if supported)
      if (performance.getEntriesByType) {
        const entries = performance.getEntriesByType('resource');
        const match = entries.find(e => e.name.includes(specifier));
        if (match) {
          this.imports.get(key).size = match.transferSize || match.encodedBodySize;
        }
      }

      return module;
    } catch (error) {
      const loadTime = performance.now() - startTime;
      this.imports.set(key, {
        specifier,
        loadTime,
        success: false,
        error: error.message,
        timestamp: Date.now()
      });
      this.notifyObservers({ key, loadTime, success: false, error: error.message });
      throw error;
    }
  }

  onLoad(callback) {
    this.observers.add(callback);
  }

  notifyObservers(data) {
    this.observers.forEach(cb => cb(data));
  }

  generateReport() {
    console.log('Dynamic Import Performance Report\n');
    console.log('Module'.padEnd(35), 'Time'.padEnd(10), 'Size'.padEnd(10), 'Status');
    console.log('-'.repeat(70));

    let totalTime = 0;
    let totalSize = 0;
    let successCount = 0;
    let failCount = 0;

    for (const [key, data] of this.imports) {
      const status = data.success ? 'OK' : 'FAIL';
      const sizeStr = data.size ? `${(data.size / 1024).toFixed(1)}KB` : 'N/A';
      const timeStr = `${data.loadTime.toFixed(1)}ms`;

      console.log(
        key.slice(0, 33).padEnd(35),
        timeStr.padEnd(10),
        sizeStr.padEnd(10),
        status
      );

      if (data.success) {
        totalTime += data.loadTime;
        totalSize += data.size || 0;
        successCount++;
      } else {
        failCount++;
      }
    }

    console.log('-'.repeat(70));
    console.log(
      'Total'.padEnd(35),
      `${totalTime.toFixed(1)}ms`.padEnd(10),
      `${(totalSize / 1024).toFixed(1)}KB`.padEnd(10),
      `${successCount}/${successCount + failCount} successful`
    );

    if (failCount > 0) {
      console.log(`\nWARNING: ${failCount} dynamic import(s) failed.`);
      console.log('Add error boundaries and retry logic for robustness.');
    }
  }
}
```

**Challenge 2: Implement a smart preloading system**
```javascript
// Build a system that predicts which dynamic imports will be
// needed and preloads them proactively.

class SmartPreloader {
  constructor() {
    this.patterns = new Map();
    this.preloaded = new Set();
    this.stats = { predicted: 0, correct: 0, wrong: 0 };
  }

  learnPattern(route, imports) {
    this.patterns.set(route, {
      imports,
      count: (this.patterns.get(route)?.count || 0) + 1
    });
  }

  predict(route) {
    const pattern = this.patterns.get(route);
    if (!pattern) return [];

    const threshold = Math.max(2, pattern.count * 0.5);
    const predictions = [];

    for (const imp of pattern.imports) {
      if (pattern.count >= threshold) {
        predictions.push(imp);
      }
    }

    return predictions;
  }

  async preload(specifier) {
    if (this.preloaded.has(specifier)) return;

    this.preloaded.add(specifier);
    this.stats.predicted++;

    try {
      // Use link rel=preload for module scripts
      const link = document.createElement('link');
      link.rel = 'modulepreload';
      link.href = specifier;
      document.head.appendChild(link);

      // Also fetch and cache the module
      const module = await import(specifier);
      this.stats.correct++;
      return module;
    } catch {
      this.stats.wrong++;
    }
  }

  async navigate(route, getImports) {
    const predictions = this.predict(route);

    // Start preloading predictions
    const preloadPromises = predictions.map(p => this.preload(p));

    // Load actual imports
    const imports = await getImports();

    // Learn from actual imports
    this.learnPattern(route, Object.keys(imports));

    await Promise.all(preloadPromises);
    return imports;
  }

  report() {
    console.log('Smart Preloader Report:\n');
    const accuracy = this.stats.predicted > 0
      ? (this.stats.correct / this.stats.predicted * 100).toFixed(0)
      : 'N/A';

    console.log(`Predictions: ${this.stats.predicted}`);
    console.log(`Correct: ${this.stats.correct}`);
    console.log(`Wrong: ${this.stats.wrong}`);
    console.log(`Accuracy: ${accuracy}%`);
    console.log(`Preloaded modules: ${this.preloaded.size}`);
    console.log(`Learned patterns: ${this.patterns.size}`);

    if (accuracy !== 'N/A' && Number(accuracy) < 50) {
      console.log('\nLow accuracy. Consider increasing the count threshold or');
      console.log('using explicit preload hints instead of prediction.');
    }
  }
}
```

**Challenge 3: Create a chunk naming consistency checker**
```javascript
// Build a tool that validates dynamic import chunk naming
// conventions and detects inconsistencies.

class ChunkNamingChecker {
  constructor() {
    this.chunks = [];
    this.namingPatterns = [];
  }

  addChunk(name, specifier, options = {}) {
    this.chunks.push({
      name: name || this.guessName(specifier),
      specifier,
      size: options.size || 0,
      isEntry: options.isEntry || false,
      dependencies: options.dependencies || []
    });
  }

  guessName(specifier) {
    const parts = specifier.split('/');
    return parts[parts.length - 1].replace(/\.\w+$/, '');
  }

  addNamingPattern(pattern) {
    this.namingPatterns.push(pattern);
  }

  check() {
    const issues = [];

    for (const chunk of this.chunks) {
      // Check naming consistency
      if (this.namingPatterns.length > 0) {
        const matches = this.namingPatterns.some(p => p.test(chunk.name));
        if (!matches) {
          issues.push({
            severity: 'warning',
            chunk: chunk.name,
            message: `Chunk name '${chunk.name}' does not match naming patterns: ${this.namingPatterns.map(p => p.toString()).join(', ')}`
          });
        }
      }

      // Check for generic names
      const genericNames = ['chunk', 'module', 'index', 'main'];
      if (genericNames.includes(chunk.name)) {
        issues.push({
          severity: 'info',
          chunk: chunk.name,
          message: `Generic chunk name '${chunk.name}'. Use more descriptive names for debugging.`
        });
      }

      // Check size
      if (chunk.size > 0 && chunk.size > 200 * 1024) {
        issues.push({
          severity: 'warning',
          chunk: chunk.name,
          message: `Large chunk: ${(chunk.size / 1024).toFixed(0)}KB. Consider further splitting.`
        });
      }

      // Check for duplicate names
      const duplicates = this.chunks.filter(c => c.name === chunk.name);
      if (duplicates.length > 1 && chunk === duplicates[0]) {
        issues.push({
          severity: 'error',
          chunk: chunk.name,
          message: `Duplicate chunk name '${chunk.name}' used ${duplicates.length} times. Each chunk needs a unique name.`
        });
      }
    }

    return issues;
  }

  suggestNamingConvention() {
    console.log('Recommended Naming Conventions:\n');
    console.log('  Route-based: routes/admin, routes/dashboard/settings');
    console.log('  Component-based: components/DataTable, components/Chart');
    console.log('  Vendor-based: vendors/react, vendors/lodash');
    console.log('  Feature-based: features/search, features/checkout');
    console.log('\nExample webpack config:');
    console.log('  import(/* webpackChunkName: "routes/admin" */ "./routes/admin")');
    console.log('  import(/* webpackChunkName: "vendors/react" */ "react")');
  }

  report() {
    const issues = this.check();
    console.log('Chunk Naming Consistency Report:\n');

    console.log(`Total chunks: ${this.chunks.length}`);
    console.log(`Naming patterns: ${this.namingPatterns.length > 0 ? this.namingPatterns.map(p => p.toString()).join(', ') : 'none'}\n`);

    if (issues.length === 0) {
      console.log('No naming issues found.');
      return;
    }

    console.log(`${issues.length} issue(s) found:\n`);
    for (const issue of issues) {
      const icon = issue.severity === 'error' ? '×' : issue.severity === 'warning' ? '!' : '>';
      console.log(`  ${icon} [${issue.severity.toUpperCase()}] ${issue.chunk}: ${issue.message}`);
    }
  }
}
```

## Module Resolution Algorithm

### What It Is

Module resolution is the algorithm that determines which file a module specifier (like `'./utils'` or `'lodash'`) refers to. Node.js has a well-defined resolution algorithm for both CommonJS and ESM. ES module resolution differs from CommonJS in several important ways, including file extension requirements and package exports.

### Why It Is Important

Module resolution is a common source of build errors and confusion. Understanding the algorithm helps developers correctly specify imports and configure their projects for successful resolution.

### How It Works Internally

Node.js ESM resolution (ES2020+):

1. **Relative specifier** (`'./'` or `'../'`): Resolved relative to the importing file's location
   - Must include file extension: `import './utils.js'` (not `'./utils'`)
   - Or use an `imports` map in package.json

2. **Bare specifier** (`'lodash'` or `'@scope/package'`): Resolved from `node_modules` directories
   - Uses the `"exports"` field in the package's `package.json`
   - Falls back to `"main"` and `"module"` fields
   - Node.js checks the directory of the importing module, then parent directories

3. **Absolute specifier**: File URL (`file:///path/to/module.js`)

4. **Package imports** (`#internal`): Using the `"imports"` field in package.json

```javascript
// Resolution examples

// Relative (must include extension)
import { helper } from './utils/helper.js';
import data from '/data/config.json' assert { type: 'json' };

// Bare specifier
import express from 'express';
import { chunk } from 'lodash-es';

// Package exports field
// If package.json has:
// "exports": { "feature": "./dist/feature.js" }
import { feature } from 'my-package/feature';
```

### Beginner Examples

```javascript
// Node.js ESM resolution rules

// Correct: with extension
import { add } from './math.js';

// Incorrect: missing extension (works in bundlers, not in Node.js ESM)
// import { add } from './math'; // Error in Node.js ESM

// Package entry resolution
// If package.json has:
// {
//   "main": "dist/index.cjs.js",
//   "module": "dist/index.esm.js",
//   "exports": {
//     ".": {
//       "import": "./dist/index.esm.js",
//       "require": "./dist/index.cjs.js"
//     }
//   }
// }

// Node.js uses exports.import for ESM importers
// Node.js uses exports.require for CJS require()
// Bundlers check module field first, then main
```

### Intermediate Examples

```javascript
// Package exports subpath patterns
// package.json
{
  "exports": {
    ".": "./index.js",
    "./feature": "./feature.js",
    "./utils/*": "./utils/*.js",     // Wildcard
    "./internal/*": null              // Block direct access
  }
}

// Valid imports:
import { feature } from 'my-package/feature';     // -> feature.js
import { helper } from 'my-package/utils/helper'; // -> utils/helper.js

// Invalid (blocked):
// import { internal } from 'my-package/internal/secret'; // Error!

// Conditional exports
{
  "exports": {
    ".": {
      "types": "./types/index.d.ts",
      "import": "./esm/index.js",
      "require": "./cjs/index.js",
      "browser": "./browser/index.js"
    }
  }
}
// Conditions checked in order: types → import/require → browser
// Custom conditions can be added with --conditions flag

// Import maps (package.json imports field)
{
  "imports": {
    "#utils": "./src/utils/index.js",
    "#components/*": "./src/components/*.jsx"
  }
}
// import { format } from '#utils';
// import Button from '#components/Button';
```

### Advanced Examples

```javascript
// Custom resolution with bundlers

// Webpack resolve configuration
// webpack.config.js
module.exports = {
  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx'],
    alias: {
      '@': path.resolve(__dirname, 'src')
    },
    modules: ['node_modules', path.resolve(__dirname, 'src')],
    mainFields: ['browser', 'module', 'main']
  }
};

// TypeScript path mapping
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"]
    }
  }
}

// Monorepo resolution (workspaces)
// package.json
{
  "workspaces": ["packages/*"]
}
// Workspace packages are symlinked in node_modules
// import { awesome } from '@my/lib/awesome'; // Resolves to packages/awesome

// Conditional export resolution in bundlers
// Most bundlers support condition exports but may differ from Node.js
// Vite: uses resolve.conditions
// Webpack: uses resolve.conditionNames
```

### Performance Considerations

- Module resolution is a blocking operation at module load time — deep node_modules trees add latency
- Node.js caches resolved paths after first resolution within the same process
- The `exports` field in package.json provides faster resolution than `main` by reducing search paths
- Symlink resolution (monorepos, workspaces) adds overhead compared to direct file resolution
- TypeScript's path mapping (`paths` in tsconfig) requires runtime resolution, adding additional lookups
- Self-referencing packages (`import from 'pkg-name'` within the package itself) add resolution complexity
- Conditional exports are evaluated at resolution time; excessive conditions slow loading
- Bundlers resolve modules at build time (one-time cost), while Node.js resolves at runtime (per-request)
- Using deep import paths (`import 'lodash/map'`) bypasses exports validation but resolves faster
- Package managers (pnpm's symlinked structure) can affect resolution performance vs npm/yarn flat structure

### Coding Challenges

**Challenge 1: Build a module resolution simulator**
```javascript
// Create a tool that simulates Node.js module resolution
// algorithm to help understand how imports are resolved.

class ModuleResolver {
  constructor(options = {}) {
    this.options = {
      extensions: options.extensions || ['.js', '.mjs', '.cjs', '.json', '.jsx', '.ts'],
      mainFields: options.mainFields || ['exports', 'module', 'main'],
      nodeModulesPaths: options.nodeModulesPaths || ['node_modules'],
      ...options
    };
    this.cache = new Map();
    this.resolutionLog = [];
  }

  addPackage(name, packageJson) {
    this.cache.set(name, packageJson);
  }

  resolve(specifier, fromFile) {
    this.resolutionLog = [];
    this.resolutionLog.push(`Resolving '${specifier}' from '${fromFile}'\n`);

    if (specifier.startsWith('.')) {
      return this.resolveRelative(specifier, fromFile);
    } else if (specifier.startsWith('#')) {
      return this.resolveInternal(specifier);
    } else {
      return this.resolveBare(specifier, fromFile);
    }
  }

  resolveRelative(specifier, fromFile) {
    const path = require('path');
    const fs = require('fs');
    const dir = path.dirname(fromFile);
    const resolved = path.resolve(dir, specifier);

    this.resolutionLog.push(`1. Relative specifier, starting from ${dir}\n`);
    this.resolutionLog.push(`2. Full path candidate: ${resolved}\n`);

    // Try exact file
    if (fs.existsSync(resolved) && fs.statSync(resolved).isFile()) {
      this.resolutionLog.push(`3. Exact file match: ${resolved}\n`);
      return resolved;
    }

    // Try with extensions
    for (const ext of this.options.extensions) {
      const withExt = resolved + ext;
      if (fs.existsSync(withExt)) {
        this.resolutionLog.push(`3. Found with extension '${ext}': ${withExt}\n`);
        return withExt;
      }
    }

    // Try as directory with index file
    const dirCandidates = ['index', ...this.options.extensions.flatMap(e => ['index' + e])];
    for (const candidate of dirCandidates) {
      const indexPath = path.join(resolved, candidate);
      if (fs.existsSync(indexPath)) {
        this.resolutionLog.push(`3. Found as directory index: ${indexPath}\n`);
        return indexPath;
      }
    }

    throw new Error(`Cannot resolve '${specifier}' from '${fromFile}'`);
  }

  resolveBare(specifier, fromFile) {
    const path = require('path');
    const fs = require('fs');
    const [packageName] = specifier.split('/').length > 1 && specifier.startsWith('@')
      ? specifier.split('/').slice(0, 2).join('/')
      : [specifier.split('/')[0]];

    this.resolutionLog.push(`1. Bare specifier, package: '${packageName}'\n`);

    // Search node_modules up the directory tree
    let currentDir = path.dirname(fromFile);
    while (currentDir !== path.dirname(currentDir)) {
      for (const nmPath of this.options.nodeModulesPaths) {
        const packageDir = path.join(currentDir, nmPath, packageName);

        const pkgPath = path.join(packageDir, 'package.json');
        if (fs.existsSync(pkgPath)) {
          const pkg = JSON.parse(fs.readFileSync(pkgPath, 'utf-8'));
          this.resolutionLog.push(`2. Found package at ${packageDir}\n`);

          // Check exports field
          if (pkg.exports && this.options.mainFields.includes('exports')) {
            const subPath = specifier.replace(packageName, '') || '.';
            const exportTarget = pkg.exports[subPath] || pkg.exports['.'];
            if (exportTarget) {
              const resolved = path.resolve(packageDir, exportTarget);
              this.resolutionLog.push(`3. Using exports field: ${exportTarget} -> ${resolved}\n`);
              return resolved;
            }
          }

          // Fall back to main
          for (const field of this.options.mainFields) {
            if (pkg[field]) {
              const resolved = path.resolve(packageDir, pkg[field]);
              this.resolutionLog.push(`3. Using '${field}' field: ${pkg[field]} -> ${resolved}\n`);
              return resolved;
            }
          }
        }
      }
      currentDir = path.dirname(currentDir);
    }

    throw new Error(`Cannot find package '${specifier}'`);
  }

  resolveInternal(specifier) {
    this.resolutionLog.push(`1. Internal/imports specifier: ${specifier}\n`);
    this.resolutionLog.push(`2. Requires package.json 'imports' field\n`);
    return specifier;
  }

  getTrace() {
    return this.resolutionLog.join('');
  }
}
```

**Challenge 2: Implement Node.js ESM resolution algorithm**
```javascript
// Build the full ESM resolution algorithm as specified by Node.js,
// including package exports patterns and conditional exports.

class ESMResolver {
  constructor(options = {}) {
    this.packages = new Map();
    this.conditions = options.conditions || ['import', 'node', 'default'];
    this.extensions = options.extensions || ['.js', '.mjs', '.cjs', '.json', '.wasm'];
  }

  register(name, pkg) {
    this.packages.set(name, pkg);
  }

  resolve(specifier, fromPackage) {
    if (specifier.startsWith('.')) {
      return this.resolveRelative(specifier, fromPackage);
    }
    if (specifier.startsWith('#')) {
      return this.resolvePackageImport(specifier, fromPackage);
    }
    return this.resolvePackageExport(specifier);
  }

  matchPattern(pattern, target) {
    // Handle * wildcards in export patterns
    if (pattern.includes('*')) {
      const [prefix, suffix] = pattern.split('*');
      if (target.startsWith(prefix) && target.endsWith(suffix)) {
        const match = target.slice(prefix.length, -suffix.length || undefined);
        return { matched: true, wildcard: match };
      }
      return { matched: false };
    }
    return { matched: pattern === target, wildcard: null };
  }

  resolvePackageExport(specifier) {
    const [scope, name] = specifier.startsWith('@')
      ? [specifier.split('/')[0] + '/' + specifier.split('/')[1], specifier.split('/').slice(2).join('/')]
      : [specifier.split('/')[0], specifier.split('/').slice(1).join('/')];

    const pkg = this.packages.get(scope);
    if (!pkg) throw new Error(`Package '${scope}' not found`);

    if (!pkg.exports) {
      return this.resolveLegacy(scope, name, pkg);
    }

    const subpath = name || '.';

    // Direct match
    if (typeof pkg.exports === 'string') {
      return this.resolveFile(pkg.exports, scope);
    }

    // Exports map
    const bestMatch = this.findBestMatch(pkg.exports, subpath);
    if (!bestMatch) {
      throw new Error(`Package '${scope}' has no export matching '${subpath}'`);
    }

    // Resolve conditional target
    const target = this.resolveConditions(bestMatch);
    if (!target) {
      throw new Error(`No matching condition for '${subpath}' in package '${scope}'`);
    }

    // Replace wildcard
    const finalTarget = bestMatch.wildcard !== null
      ? target.replace('*', bestMatch.wildcard)
      : target;

    return this.resolveFile(finalTarget, scope);
  }

  findBestMatch(exports, subpath) {
    // Try exact match first
    if (exports[subpath]) {
      return { target: exports[subpath], wildcard: null };
    }

    // Try wildcard patterns sorted by specificity
    const wildcardKeys = Object.keys(exports)
      .filter(k => k.includes('*'))
      .sort((a, b) => b.length - a.length);

    for (const key of wildcardKeys) {
      const { matched, wildcard } = this.matchPattern(key, subpath);
      if (matched) {
        return { target: exports[key], wildcard };
      }
    }

    return null;
  }

  resolveConditions(target) {
    if (typeof target === 'string') return target;

    for (const condition of this.conditions) {
      if (target[condition]) {
        const value = target[condition];
        return typeof value === 'string' ? value : this.resolveConditions(value);
      }
    }

    if (target.default) {
      return typeof target.default === 'string' ? target.default : null;
    }

    return null;
  }

  resolveFile(filePath, packageName) {
    // Try exact, then with extensions, then /index
    return filePath;
  }

  resolveLegacy(scope, name, pkg) {
    const mainField = pkg.module || pkg.main || 'index.js';
    return this.resolveFile(mainField, scope);
  }

  resolvePackageImport(specifier, fromPackage) {
    const pkg = this.packages.get(fromPackage);
    if (!pkg || !pkg.imports) {
      throw new Error(`No imports found in package '${fromPackage}'`);
    }

    const match = this.findBestMatch(pkg.imports, specifier);
    if (!match) throw new Error(`No import match for '${specifier}'`);

    const target = this.resolveConditions(match.target);
    return target;
  }

  resolveRelative(specifier, fromPackage) {
    // File path resolution
    return specifier;
  }
}
```

**Challenge 3: Create a resolution path tracer**
```javascript
// Build a debugging tool that shows the full resolution path
// taken by Node.js when resolving a module specifier.

class ResolutionTracer {
  constructor() {
    this.trace = [];
    this.startTime = 0;
  }

  begin(specifier, from) {
    this.startTime = Date.now();
    this.trace = [];
    this.trace.push({
      step: 0,
      action: 'START',
      detail: `Resolve '${specifier}' from '${from}'`,
      time: 0
    });
  }

  step(action, detail) {
    this.trace.push({
      step: this.trace.length,
      action,
      detail,
      time: Date.now() - this.startTime
    });
  }

  end(result) {
    this.trace.push({
      step: this.trace.length,
      action: 'END',
      detail: `Resolved to: ${result}`,
      time: Date.now() - this.startTime
    });
  }

  error(err) {
    this.trace.push({
      step: this.trace.length,
      action: 'ERROR',
      detail: err.message || String(err),
      time: Date.now() - this.startTime
    });
  }

  // Simulate Node.js resolution steps
  simulateResolution(specifier) {
    this.begin(specifier, '/project/src/index.js');

    if (specifier.startsWith('.')) {
      this.step('RELATIVE', `Detected relative path specifier: '${specifier}'`);
      this.step('CWD', `Resolving relative to: /project/src/`);

      const candidates = [
        specifier,
        specifier + '.js',
        specifier + '.mjs',
        specifier + '/index.js',
        specifier + '/index.mjs'
      ];

      for (const c of candidates) {
        this.step('TRY', `Trying: /project/src/${c}`);
      }

      this.step('FOUND', `Found: /project/src/${specifier}.js`);
      this.step('LOAD', `Loading module: /project/src/${specifier}.js`);
      this.end(`/project/src/${specifier}.js`);

    } else {
      const packageName = specifier.split('/').slice(0, specifier.startsWith('@') ? 2 : 1).join('/');
      const subpath = specifier.replace(packageName, '') || '.';

      this.step('BARE', `Detected bare specifier: '${specifier}'`);
      this.step('PACKAGE', `Looking for package: '${packageName}'`);

      const searchPaths = [
        '/project/node_modules/' + packageName,
        '/node_modules/' + packageName
      ];

      for (const p of searchPaths) {
        this.step('SEARCH', `Checking: ${p}/package.json`);
      }

      this.step('FOUND', `Found package: /project/node_modules/${packageName}`);

      // Check exports field
      this.step('EXPORTS', `Checking 'exports' field in package.json`);

      if (subpath === '.') {
        this.step('MAIN', `Using 'main' field: dist/index.js`);
        this.end(`/project/node_modules/${packageName}/dist/index.js`);
      } else {
        this.step('SUBPATH', `Resolving subpath: '${subpath}'`);
        this.step('MATCH', `Pattern match: './dist/${subpath}.js'`);
        this.end(`/project/node_modules/${packageName}/dist/${subpath}.js`);
      }
    }

    return this;
  }

  generateReport() {
    console.log('Module Resolution Trace:\n');
    console.log('Step'.padEnd(8), 'Action'.padEnd(15), 'Time'.padEnd(10), 'Detail');
    console.log('-'.repeat(90));

    for (const t of this.trace) {
      console.log(
        String(t.step).padEnd(8),
        t.action.padEnd(15),
        `${t.time}ms`.padEnd(10),
        t.detail
      );
    }

    const lastStep = this.trace[this.trace.length - 1];
    if (lastStep.action === 'END') {
      console.log(`\nTotal resolution time: ${lastStep.time}ms`);
    } else if (lastStep.action === 'ERROR') {
      console.log(`\nResolution failed: ${lastStep.detail}`);
    }
  }
}
```

## Circular Dependency Handling

### What It Is

Circular (or cyclic) dependencies occur when two or more modules depend on each other directly or indirectly (A imports B, B imports A). Both CommonJS and ES modules handle circular dependencies, but they handle them differently. ES modules are more resilient due to live bindings.

### Why It Is Important

Circular dependencies are common in real-world codebases, especially in model/relationship definitions, ORM schemas, and component hierarchies. Understanding how different module systems handle cycles prevents subtle bugs like undefined imports.

### How It Works Internally

**ES modules**: During the linking phase, all imports and exports are connected as live bindings before any module body is evaluated. When a cycle is detected, the module that is still being evaluated has its exports all set but the exported bindings may still be undefined (if the exporting module's body hasn't executed yet).

**CommonJS**: Modules are evaluated synchronously. If module A requires module B, and B requires A, module A's `module.exports` at the time B requires it may be incomplete (only part of its exports are set). This leads to the "partial exports" problem.

```javascript
// Circular dependency with ES modules (works correctly)

// a.js
import { b } from './b.js';
export const a = 'A';
console.log('a.js:', b); // 'B' (b is initialized when a runs)

// b.js
import { a } from './a.js';
export const b = 'B';
console.log('b.js:', a); // 'A' (a is already initialized)

// Output:
// b.js: A
// a.js: B
```

### Beginner Examples

```javascript
// CommonJS circular dependency (problematic)

// a.cjs
const b = require('./b.cjs');
exports.a = 'A';
console.log('a.cjs:', b); // {}

// b.cjs
const a = require('./a.cjs');
exports.b = 'B';
console.log('b.cjs:', a); // {} (a is empty when b requires it!)

// Main
const a = require('./a.cjs');
console.log('main:', a); // { a: 'A', b: 'B' }

// ES modules handle this better:
// a.mjs
import { b } from './b.mjs';
export const a = 'A';
console.log('a.mjs:', b); // 'B'

// b.mjs
import { a } from './a.mjs';
export const b = 'B';
console.log('b.mjs:', a); // 'A' (no undefined!)
```

### Intermediate Examples

```javascript
// Circular dependency with default exports

// a.js
import b from './b.js';
const a = { name: 'A', friend: b?.name };
export default a;
console.log('a:', a); // { name: 'A', friend: 'B' }

// b.js
import a from './a.js';
const b = { name: 'B', friend: a?.name };
export default b;
console.log('b:', b); // { name: 'B', friend: 'A' }

// Works because live bindings ensure values are available
// But order depends on which module is evaluated first

// Circular dependency with classes
// types.js
import { Dragon } from './creatures.js';
export class Monster {
  static types = [Dragon];
}

// creatures.js
import { Monster } from './types.js';
export class Dragon extends Monster {
  constructor() { super(); }
}
// This works in ESM because class declarations are hoisted
// during the linking phase
```

### Advanced Examples

```javascript
// Detecting circular dependencies
class CircularDependencyDetector {
  constructor() {
    this.visited = new Set();
    this.stack = new Set();
    this.cycles = [];
  }
  
  detect(moduleName, moduleGraph) {
    this.visited.add(moduleName);
    this.stack.add(moduleName);
    
    const dependencies = moduleGraph[moduleName] || [];
    for (const dep of dependencies) {
      if (!this.visited.has(dep)) {
        this.detect(dep, moduleGraph);
      } else if (this.stack.has(dep)) {
        this.cycles.push([...this.stack]);
      }
    }
    
    this.stack.delete(moduleName);
  }
  
  getCycles(moduleGraph) {
    for (const moduleName of Object.keys(moduleGraph)) {
      if (!this.visited.has(moduleName)) {
        this.detect(moduleName, moduleGraph);
      }
    }
    return this.cycles;
  }
}

// Breaking circular dependencies
// Strategy 1: Extract shared dependency
// Before:
// A -> B -> A
// After:
// A -> C <- B

// Strategy 2: Use dependency injection
// a.js
export class A {
  constructor(b) { this.b = b; }
}

// b.js
export class B {
  constructor(a) { this.a = a; }
}

// main.js
import { A } from './a.js';
import { B } from './b.js';
const b = new B();
const a = new A(b);
b.a = a; // Set reference after construction

// Strategy 3: Lazy require (use import() for the problematic module)
// a.js
let _B;
export class A {
  get b() {
    if (!_B) _B = require('./b.js').B;
    return new _B();
  }
}

// Strategy 4: Interface/type extraction (TypeScript)
// types.ts
export interface IRepository { /* ... */ }

// a.ts
import type { IRepository } from './types.js';
export class A { constructor(repo: IRepository) {} }

// b.ts
import { A } from './a.js';
import type { IRepository } from './types.js';
export class B implements IRepository {}
```

### Real-World Use Cases

- ORM entity relationships (User has Posts, Post belongs to User)
- Plugin/extension architecture
- Model graph with bidirectional references
- Middleware chains (Express, Redux)
- Component trees in UI frameworks
- Messaging/event systems

### Common Mistakes

- Creating circular dependencies through barrel files
- Expecting CommonJS to handle cycles like ESM
- Using `require()` inside ESM which behaves differently
- Not restructuring code when circular dependencies form
- Assuming all bundlers handle cycles the same way
- Forgetting that module evaluation order affects cycle behavior

### Best Practices

- Avoid circular dependencies when possible (extract shared module)
- Use ES modules for better circular dependency handling
- Restructure code to extract shared dependencies
- Use dependency injection to break hard cycles
- Use interfaces/types for circular type references
- Use lazy imports (`import()`) to break runtime cycles
- Run dependency cycle detection in CI

### Performance Considerations

- Circular dependencies prevent dead code elimination and tree shaking (all exports in a cycle are "used")
- Module evaluation order in cycles affects initialization time (modules must wait for their dependencies)
- Each cycle adds latency to module loading (modules can't complete evaluation until dependents are ready)
- CJS cycles are more expensive than ESM cycles due to value copying vs live bindings
- Cycles across async boundaries (dynamic import) increase complexity and potential for race conditions
- Deeply nested cycles (5+ modules) can trigger stack-like evaluation delays
- Bundlers may fail to code-split modules involved in circular dependencies
- TypeScript type-only imports (`import type`) don't create cycles but the bundler may not know this
- Dependency cycle detection tools add build-time overhead proportional to graph complexity
- Restructuring to break cycles often improves module initialization order and overall load speed

### Coding Challenges

**Challenge 1: Build a circular dependency detector for ES modules**
```javascript
// Create a tool that uses DFS to detect circular dependencies
// in module graphs, reporting the full cycle path.

class CircularDependencyDetector {
  constructor() {
    this.graph = new Map();
  }

  addModule(name, imports) {
    this.graph.set(name, imports);
  }

  detect() {
    const WHITE = 0, GRAY = 1, BLACK = 2;
    const colors = new Map();
    const parent = new Map();
    const cycles = [];

    for (const node of this.graph.keys()) {
      colors.set(node, WHITE);
    }

    const dfs = (node) => {
      colors.set(node, GRAY);

      const neighbors = this.graph.get(node) || [];
      for (const neighbor of neighbors) {
        if (!this.graph.has(neighbor)) continue;

        if (colors.get(neighbor) === GRAY) {
          // Found cycle - reconstruct the path
          const cycle = [];
          let current = node;
          while (current !== neighbor) {
            cycle.unshift(current);
            current = parent.get(current);
            if (!current) break;
          }
          cycle.unshift(neighbor);
          cycle.push(neighbor); // Close the cycle
          cycles.push(cycle);
        } else if (colors.get(neighbor) === WHITE) {
          parent.set(neighbor, node);
          dfs(neighbor);
        }
      }

      colors.set(node, BLACK);
    };

    for (const node of this.graph.keys()) {
      if (colors.get(node) === WHITE) {
        dfs(node);
      }
    }

    return cycles;
  }

  addEdge(from, to) {
    if (!this.graph.has(from)) this.graph.set(from, []);
    this.graph.get(from).push(to);
  }

  getCycleImpact(cycle) {
    // Calculate how many modules are affected by this cycle
    const affected = new Set(cycle);
    const queue = [...cycle];

    while (queue.length > 0) {
      const mod = queue.shift();
      const imports = this.graph.get(mod) || [];
      for (const imp of imports) {
        if (!affected.has(imp)) {
          affected.add(imp);
          queue.push(imp);
        }
      }
    }

    return affected.size;
  }

  report() {
    const cycles = this.detect();

    console.log('Circular Dependency Analysis:\n');
    console.log(`Total modules: ${this.graph.size}\n`);

    if (cycles.length === 0) {
      console.log('No circular dependencies found.');
      return;
    }

    console.log(`Found ${cycles.length} circular dependenc(ies):\n`);

    cycles.forEach((cycle, i) => {
      const unique = [...new Set(cycle)];
      const impact = this.getCycleImpact(unique);
      console.log(`Cycle ${i + 1}: ${unique.join(' -> ')}`);
      console.log(`  Length: ${unique.length - 1} modules`);
      console.log(`  Affected: ${impact} modules (${(impact / this.graph.size * 100).toFixed(0)}% of graph)`);
      console.log(`  Severity: ${impact > this.graph.size * 0.5 ? 'HIGH' : impact > this.graph.size * 0.25 ? 'MEDIUM' : 'LOW'}`);
      console.log(`  Suggestion: ${this.suggestFix(unique)}\n`);
    });
  }

  suggestFix(cycle) {
    if (cycle.length <= 3) {
      return `Extract the shared dependency into a separate module that both can import.`;
    }
    return `Consider merging modules ${cycle[0]} and ${cycle[1]} into a single module, or use dependency injection.`;
  }
}
```

**Challenge 2: Implement live binding semantics for cycle-safe modules**
```javascript
// Create a module system that correctly implements ES module
// live bindings to safely handle circular dependencies.

class LiveBindingModuleSystem {
  constructor() {
    this.modules = new Map();
    this.exports = new Map();
    this.evaluated = new Set();
    this.evaluating = new Set();
  }

  define(name, factory) {
    this.modules.set(name, { factory, evaluated: false });
    this.exports.set(name, {});
    return this;
  }

  async import(name) {
    if (this.evaluated.has(name)) {
      return this.exports.get(name);
    }

    if (this.evaluating.has(name)) {
      // Circular import detected - return partial exports
      // ESM live bindings mean the caller will see the value
      // when it's eventually assigned
      return this.exports.get(name);
    }

    this.evaluating.add(name);

    const mod = this.modules.get(name);
    if (!mod) throw new Error(`Module '${name}' not found`);

    // Create the exports object (live bindings)
    const moduleExports = {};

    // Wrap factory to capture exports
    const exportFn = (key, value) => {
      Object.defineProperty(moduleExports, key, {
        get: () => value,
        enumerable: true,
        configurable: true
      });
    };

    const importFn = async (depName) => {
      const depExports = await this.import(depName);
      return depExports;
    };

    // Execute the module factory
    await mod.factory(exportFn, importFn);

    this.exports.set(name, moduleExports);
    this.evaluated.add(name);
    this.evaluating.delete(name);

    return moduleExports;
  }

  // Example usage: A <-> B circular dependency
  setupExample() {
    this.define('A', (exportFn, importFn) => {
      exportFn('name', 'Module A');

      importFn('B').then(bExports => {
        // Even though B imports A, A can safely access B's exports
        // because of live bindings
        console.log('A sees B.name:', bExports.name);
      });

      exportFn('getValue', () => {
        return 'Value from A';
      });
    });

    this.define('B', (exportFn, importFn) => {
      exportFn('name', 'Module B');

      importFn('A').then(aExports => {
        // B can access A's exports even though A imports B
        // This is the key benefit of ESM live bindings
        console.log('B sees A.name:', aExports.name);
      });

      exportFn('getValue', () => {
        return 'Value from B';
      });
    });

    return this;
  }

  compareWithCJS() {
    console.log('ESM vs CJS Circular Dependency Behavior:\n');

    console.log('ESM (Live Bindings):');
    console.log('  - All modules are linked before evaluation');
    console.log('  - Exports are live references (not copies)');
    console.log('  - Circular imports return partial exports immediately');
    console.log('  - Values update as modules complete evaluation');
    console.log('  - Works correctly: function hoisting + live bindings\n');

    console.log('CommonJS (Value Copies):');
    console.log('  - Modules execute synchronously in order');
    console.log('  - module.exports is an object that gets populated over time');
    console.log('  - If B requires A before A finishes, B gets an incomplete object');
    console.log('  - Values are copies at time of require()');
    console.log('  - Fails silently: B gets undefined for exports A hasn\'t set yet\n');
  }
}
```

**Challenge 3: Create a cycle-breaking refactoring assistant**
```javascript
// Build a tool that suggests how to refactor code to eliminate
// or mitigate circular dependencies.

class CycleRefactorAssistant {
  constructor() {
    this.cycles = [];
    this.strategies = new Map();
    this.initStrategies();
  }

  initStrategies() {
    this.strategies.set('extract-shared', {
      name: 'Extract Shared Module',
      description: 'Extract the common dependency into a separate module that both can import.',
      difficulty: 'low',
      appliesTo: (cycle) => cycle.length === 3
    });

    this.strategies.set('merge-modules', {
      name: 'Merge Modules',
      description: 'Merge the two cyclically-dependent modules into a single module.',
      difficulty: 'medium',
      appliesTo: (cycle) => cycle.length === 3
    });

    this.strategies.set('dependency-injection', {
      name: 'Dependency Injection',
      description: 'Pass dependencies as parameters instead of importing them directly.',
      difficulty: 'high',
      appliesTo: (cycle) => cycle.length >= 3
    });

    this.strategies.set('lazy-import', {
      name: 'Lazy Dynamic Import',
      description: 'Replace one static import with a dynamic import() inside the function that needs it.',
      difficulty: 'medium',
      appliesTo: (cycle) => cycle.length >= 3
    });

    this.strategies.set('interface-extraction', {
      name: 'Interface/Type Extraction',
      description: 'Extract interfaces or types into a separate module (no runtime dependency).',
      difficulty: 'low',
      appliesTo: (cycle) => cycle.length >= 3
    });

    this.strategies.set('event-emitter', {
      name: 'Event Emitter Pattern',
      description: 'Replace direct imports with an event-based communication pattern.',
      difficulty: 'high',
      appliesTo: () => true
    });
  }

  analyzeCycle(cycle) {
    const suggestions = [];

    for (const [key, strategy] of this.strategies) {
      if (strategy.appliesTo(cycle)) {
        suggestions.push(strategy);
      }
    }

    return suggestions;
  }

  analyze(code) {
    const imports = [];
    const lines = code.split('\n');

    for (const line of lines) {
      const imp = line.match(/import\s+.*\s+from\s+['"]([^'"]+)['"]/);
      if (imp) {
        imports.push({ specifier: imp[1], line: lines.indexOf(line) + 1 });
      }
    }

    return imports;
  }

  generateRefactoringPlan(cycle, modules) {
    console.log('Refactoring Plan for Circular Dependency:\n');
    console.log(`Cycle: ${cycle.join(' -> ')}\n`);

    console.log('Recommended Strategies (sorted by ease):\n');
    const suggestions = this.analyzeCycle(cycle);
    suggestions.sort((a, b) => {
      const diff = { low: 0, medium: 1, high: 2 };
      return diff[a.difficulty] - diff[b.difficulty];
    });

    suggestions.forEach((s, i) => {
      console.log(`${i + 1}. ${s.name} (${s.difficulty})`);
      console.log(`   ${s.description}\n`);
    });

    const cycleSet = new Set(cycle);
    const circularImports = modules.filter(m =>
      module.exports && cycleSet.has(m)
    );

    if (circularImports.length > 0) {
      console.log('Modules to modify:\n');
      circularImports.forEach(m => {
        console.log(`  - ${m}`);
      });
    }

    return suggestions;
  }
}
```

### Interview Questions

**Q: How do ES modules handle circular dependencies differently from CommonJS?** A: ES modules use live bindings (references to exported values) rather than value copies. During the linking phase, all imports and exports are connected before any module is evaluated. This means that even if A imports B and B imports A, both modules will see the correct values because the bindings are live. CommonJS evaluates modules synchronously, so if A requires B before finishing its exports, B sees an incomplete module.

**Q: What is the difference between `import` and `import()`?** A: `import` is a static declaration that must be at the top level of a module. It enables static analysis and tree shaking. `import()` is a function-like expression that loads modules asynchronously at runtime, enabling code splitting and conditional loading. `import()` returns a Promise resolving to the module namespace.

### Related Topics

- CommonJS module system (require, module.exports)
- Package.json exports field
- Node.js module resolution
- Code splitting strategies
- Tree shaking and static analysis
- Module bundler internals
- Import maps and URL imports
- TypeScript module resolution
- Deno module system (URL imports)
- WebAssembly modules
