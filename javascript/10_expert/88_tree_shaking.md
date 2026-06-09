# Tree Shaking - Dead code elimination, ES module static analysis, side effects

## Introduction

Tree shaking is a dead code elimination technique that removes unused exports from JavaScript bundles. The term comes from the metaphor of shaking a tree to make dead leaves fall off. Tree shaking relies on ES module's static structure (imports/exports are known at compile time) to determine which code is actually used. It is a critical optimization for modern JavaScript applications, significantly reducing bundle sizes.

## Dead Code Elimination

### What It Is

Dead code elimination (DCE) is the process of removing code that is never executed or never referenced from the final bundle. This includes unused functions, unused exports, unreachable code paths, and variables that are never read. Tree shaking is a form of DCE specialized for module systems, but DCE also includes more general compiler optimizations.

### Why It Is Important

Modern JavaScript applications depend on thousands of packages, but developers typically use only a fraction of each package's API. Without tree shaking, users download every function, class, and utility from every dependency—even the ones they never use. Tree shaking can reduce bundle sizes by 50-90% for utility-heavy libraries like lodash, moment.js, or RxJS.

### How It Works Internally

The bundler (Webpack, Rollup, esbuild) parses the module graph starting from the entry point. It tracks which imports are used and traces them to their source exports. Any export that is not imported or is imported but never used is marked as "dead." The bundler also performs static analysis within modules to identify unused local variables and unreachable code.

The process:
1. Build the dependency graph from entry point
2. For each module, analyze which exports are imported by other modules
3. Mark unused exports as eligible for removal
4. During code generation, omit marked code
5. Additional DCE: remove dead branches (constant folding), unused variables

### Syntax

```javascript
// Without tree shaking: entire module is included
import _ from 'lodash';
_.chunk([1, 2, 3], 2);

// With tree shaking: only chunk function is included
import { chunk } from 'lodash-es';
chunk([1, 2, 3], 2);

// Dead code that gets eliminated
const unused = 'never used';
function unusedFunction() { return 'dead'; }

function usedFunction() {
  if (false) { // Dead branch (constant folding)
    console.log('never executed');
  }
  return 'alive';
}

export { usedFunction };
```

### Beginner Examples

```javascript
// Example: moment.js tree shaking
// Bad (imports entire moment.js, ~230KB gzipped)
import moment from 'moment';
moment().format('YYYY-MM-DD');

// Good (date-fns supports tree shaking, ~18KB for single import)
import { format } from 'date-fns';
format(new Date(), 'yyyy-MM-dd');

// Better (even more targeted)
import format from 'date-fns/format';
format(new Date(), 'yyyy-MM-dd');

// Example: lodash tree shaking
// Bad: imports entire lodash (~70KB)
import _ from 'lodash';
_.map([1, 2, 3], n => n * 2);

// Good: imports specific function
import map from 'lodash/map';
// or
import { map } from 'lodash-es'; // lodash-es supports tree shaking
```

### Intermediate Examples

```javascript
// Barrel files and tree shaking
// Barrel files re-export from multiple modules
// They can defeat tree shaking if the bundler can't trace through them

// utils/index.js (barrel file)
export { formatDate } from './date';
export { calculateTotal } from './math';
export { validateEmail } from './validation';
export { sendEmail } from './email'; // Unused in this bundle

// app.js
import { formatDate } from './utils';
// If tree shaking works, sendEmail and its dependencies are excluded
// BUT: if the barrel file has side effects, nothing can be excluded

// import * pattern - imports everything, prevents tree shaking
import * as utils from './utils';
utils.formatDate(...); // uses formatDate, but all exports are referenced via utils

// Default export vs named exports
// Default imports are harder to tree-shake
import Utils from './utils'; // Unclear what's used

// Named imports enable static analysis
import { formatDate, calculateTotal } from './utils';
```

### Advanced Examples

```javascript
// Dynamic imports and tree shaking
// Dynamic imports create separate chunks, tree-shaken independently
// Only the requested module and its deps are included

import('./math').then(({ add }) => {
  console.log(add(2, 3));
});

// Re-exports with side effects
// Some re-exports simulate side effects, preventing DCE

// index.js
export { default as Button } from './Button';
import './Button.css'; // Side effect! Prevents Button from being tree-shaken away
// Solution: mark css imports as side-effect-free

// Conditional re-exports
// Some libraries conditionally export
let math;
if (process.env.NODE_ENV === 'production') {
  math = require('./math.prod');
} else {
  math = require('./math.dev');
}
// This prevents tree shaking because the bundler can't statically determine
// which exports are used

// Advanced: library author's perspective
// OPTION 1: Individual file imports
// lib/Button.js
export function Button() { ... }

// User imports:
import Button from 'lib/Button';

// OPTION 2: Side-effect-free package configuration
// package.json
{
  "sideEffects": false,
  "module": "lib/index.js"
}

// OPTION 3: Per-file side effects
{
  "sideEffects": [
    "lib/**/*.css",
    "lib/polyfill.js"
  ]
}

// Module level value (live binding) vs tree shaking
// ES module exports are live bindings (not copies)
// This means even if a value is imported, the source module still runs
// if it modifies the value
```

### Coding Challenges

**Challenge 1: Implement a dead code eliminator for a simple module system**
```javascript
// Build a static analyzer that identifies and removes unused exports.

class DeadCodeEliminator {
  constructor() {
    this.modules = new Map();
    this.entryPoints = new Set();
  }

  addModule(name, exports, body) {
    this.modules.set(name, { exports, body, usedExports: new Set(), referenced: new Set() });
  }

  setEntryPoint(name) {
    this.entryPoints.add(name);
  }

  analyze() {
    // Track which exports are referenced across modules
    for (const [name, mod] of this.modules) {
      for (const ref of mod.referenced) {
        // Find which module exports this symbol
        for (const [otherName, otherMod] of this.modules) {
          if (otherMod.exports.includes(ref)) {
            otherMod.usedExports.add(ref);
          }
        }
      }
    }

    // Entry points are always kept
    for (const name of this.entryPoints) {
      const mod = this.modules.get(name);
      if (mod) {
        mod.exports.forEach(e => mod.usedExports.add(e));
      }
    }
  }

  getUnusedExports() {
    this.analyze();
    const unused = [];
    for (const [name, mod] of this.modules) {
      for (const exp of mod.exports) {
        if (!mod.usedExports.has(exp)) {
          unused.push({ module: name, export: exp });
        }
      }
    }
    return unused;
  }

  eliminate() {
    this.analyze();
    const result = [];

    for (const [name, mod] of this.modules) {
      if (mod.exports.length > 0 && mod.usedExports.size === 0 && !this.entryPoints.has(name)) {
        continue;
      }
      const removedExports = mod.exports.filter(e => !mod.usedExports.has(e));
      const keptExports = mod.exports.filter(e => mod.usedExports.has(e));
      result.push({
        module: name,
        kept: keptExports,
        removed: removedExports,
        linesRemoved: removedExports.reduce((sum, exp) => sum + mod.body[exp] || 0, 0)
      });
    }

    return result;
  }
}
```

**Challenge 2: Build a tree shaking visualizer**
```javascript
// Create a tool that shows the impact of tree shaking
// by comparing bundle sizes before and after elimination.

class TreeShakingVisualizer {
  constructor() {
    this.modules = new Map();
    this.importGraph = new Map();
  }

  addModule(name, size, exports) {
    this.modules.set(name, { size, exports, imported: 0 });
  }

  addImport(from, to, symbols) {
    if (!this.importGraph.has(from)) {
      this.importGraph.set(from, []);
    }
    this.importGraph.get(from).push({ to, symbols });
  }

  simulateShaking() {
    let totalBefore = 0;
    let totalAfter = 0;
    const stats = [];

    for (const [name, mod] of this.modules) {
      totalBefore += mod.size;
      const usedExports = new Set();

      // Find all imports of this module
      for (const [, imports] of this.importGraph) {
        for (const imp of imports) {
          if (imp.to === name) {
            imp.symbols.forEach(s => usedExports.add(s));
          }
        }
      }

      const ratio = mod.exports.length > 0
        ? usedExports.size / mod.exports.length
        : 1;
      const keptSize = Math.round(mod.size * Math.max(ratio, 0.1));
      totalAfter += keptSize;

      stats.push({
        module: name,
        exports: mod.exports.length,
        used: usedExports.size,
        keptSize,
        percentKept: Math.round(Math.max(ratio, 0.1) * 100)
      });
    }

    return {
      before: totalBefore,
      after: totalAfter,
      saved: totalBefore - totalAfter,
      percentReduction: Math.round((1 - totalAfter / totalBefore) * 100),
      modules: stats
    };
  }

  report() {
    const result = this.simulateShaking();
    console.log('Tree Shaking Report:\n');
    console.log(`Total before: ${result.before} bytes`);
    console.log(`Total after: ${result.after} bytes`);
    console.log(`Saved: ${result.saved} bytes (${result.percentReduction}% reduction)\n`);

    console.log('Module Breakdown:');
    console.log('Module'.padEnd(25), 'Exports'.padEnd(10), 'Used'.padEnd(10), 'Kept %'.padEnd(10), 'Size After');
    console.log('-'.repeat(65));
    for (const m of result.modules) {
      console.log(
        m.module.slice(0, 23).padEnd(25),
        String(m.exports).padEnd(10),
        String(m.used).padEnd(10),
        `${m.percentKept}%`.padEnd(10),
        `${m.keptSize}B`
      );
    }
  }
}
```

**Challenge 3: Detect code that defeats tree shaking**
```javascript
// Create a linter that identifies patterns which prevent
// effective dead code elimination.

class TreeShakingLinter {
  constructor(source) {
    this.source = source;
    this.lines = source.split('\n');
    this.issues = [];
  }

  lint() {
    // Pattern 1: Side effects in modules
    for (let i = 0; i < this.lines.length; i++) {
      const line = this.lines[i];
      const trimmed = line.trim();

      // Top-level console.log
      if (/^console\.(log|warn|error)\(/.test(trimmed) && !trimmed.includes('//')) {
        this.addIssue(i, 'Top-level console.* prevents tree shaking (has side effects)');
      }

      // Prototype patching
      if (/\w+\.prototype\.\w+\s*=/.test(trimmed)) {
        this.addIssue(i, 'Prototype extension creates a side effect');
      }

      // Global variable modification
      if (/^(window|global|globalThis)\.\w+\s*=/.test(trimmed)) {
        this.addIssue(i, 'Global variable assignment creates a side effect');
      }

      // Dynamic require
      if (/require\(['"`]/.test(trimmed) && /\$\{/.test(trimmed)) {
        this.addIssue(i, 'Dynamic require() with template literal prevents static analysis');
      }

      // re-export barrel with many exports
      if (/export\s+\*\s+from/.test(trimmed)) {
        this.addIssue(i, 'Wildcard re-export (* from) may include unnecessary modules');
      }
    }

    // Pattern 2: CommonJS imports
    const cjsImports = this.source.match(/const\s+\w+\s*=\s*require\(/g);
    if (cjsImports && cjsImports.length > 3) {
      this.addIssue(0, `Found ${cjsImports.length} CommonJS require() calls - switch to ES imports for tree shaking`);
    }

    // Pattern 3: No sideEffects flag
    if (!this.source.includes('"sideEffects"') && !this.source.includes("'sideEffects'")) {
      this.addIssue(0, 'No sideEffects flag found in package.json - bundler may not eliminate unused modules');
    }

    return this.report();
  }

  addIssue(line, message) {
    this.issues.push({ line: line + 1, message, severity: line === 0 ? 'warning' : 'error' });
  }

  report() {
    if (this.issues.length === 0) {
      console.log('No tree shaking issues found.\n');
      return [];
    }

    console.log(`Tree shaking analysis: ${this.issues.length} issue(s) found\n`);
    for (const issue of this.issues) {
      const icon = issue.severity === 'error' ? '!' : '>';
      console.log(`  ${icon} Line ${issue.line}: ${issue.message}`);
    }
    return this.issues;
  }
}
```

## ES Module Static Analysis

### What It Is

ES module static analysis is the ability to determine import and export relationships without executing the code. Because `import` and `export` statements are syntactically rigid (top-level, no conditionals), bundlers can parse them statically to build a complete dependency graph. This enables tree shaking, dead code elimination, and deterministic module ordering.

### Why It Is Important

Static analysis is what makes tree shaking possible. CommonJS's `require()` is dynamic (can be conditional, inside functions, with computed paths), making static analysis impossible for arbitrary code. ES modules' static structure gives bundlers the visibility needed to safely remove unused code.

### How It Works Internally

The bundler's parser identifies all `import` and `export` statements. For each module, it builds a mapping of:

- **Exports** provided: names and the local bindings they reference
- **Imports** consumed: names from which modules
- **Re-exports**: `export { x } from './other'`

This forms a module graph. The bundler then traces from entry points: a module's export is "live" if any entry point or live module imports it. Modules with no live imports (but with side effects) are kept. Modules with no live imports and no side effects are removed.

```javascript
// Static vs dynamic analysis

// Static (ES modules - analyzable):
import { helper } from './utils';
export const result = helper();

// Dynamic (CommonJS - not statically analyzable):
const utils = require('./utils');
if (condition) {
  module.exports = utils.fn;
}
```

### Beginner Examples

```javascript
// ES Module static structure
// ALL imports/exports must be at the top level

// Valid (static):
import { a } from './module';
export const b = a + 1;

// Invalid (dynamic - SyntaxError):
if (condition) {
  import { a } from './module'; // Error: import must be top-level
}

// Dynamic import() is allowed but creates a separate module graph
// import() returns a Promise, can't be statically traced

// Named exports must have known names
export { myFunction }; // Static, analyzable

// Dynamic names not allowed
const name = 'myFunction';
export { [name]: value }; // Not valid ES module syntax
```

### Intermediate Examples

```javascript
// Live bindings vs copies
// ES modules export live bindings (connections to variables)

// module.js
export let count = 0;
export function increment() {
  count++;
}

// main.js
import { count, increment } from './module.js';
console.log(count); // 0
increment();
console.log(count); // 1 (live binding!)

// CommonJS would be:
// var count = 0;
// exports.count = count; // Copy, not binding
// increment would update the local count, not exports.count

// Renaming exports
const internalName = 'value';
export { internalName as publicName };

// Aggregating modules
export { value } from './other-module';
// This re-exports without creating a local binding
// The bundler can trace through re-exports

// import.meta and meta properties
console.log(import.meta.url); // Static at module level
```

### Advanced Examples

```javascript
// Module graph analysis for tree shaking
// Bundler traces all import/export chains

// entry.js
import { analyze } from './analyzer';
console.log(analyze(10));

// analyzer.js
import { compute } from './compute';
import { format } from './format'; // UNUSED import (not referenced)
export function analyze(n) {
  return compute(n) * 2;
}

// compute.js
import { sqrt } from './math';
export function compute(n) {
  return sqrt(n);
}

// math.js
export function sqrt(n) { return n * n; }
export function cube(n) { return n * n * n; } // UNUSED export
export function exp(n) { return Math.exp(n); } // UNUSED export

// format.js
export function format(n) { return `Result: ${n}`; } // ENTIRE MODULE UNUSED
// Since nothing imports from format.js and it has no side effects,
// the entire module is removed

// Side effect detection algorithm
// A module has side effects if:
// 1. It runs code at the top level (not in a function)
// 2. It modifies a global/shared value
// 3. It patches prototypes or globals
// 4. It imports CSS or other assets (unless configured)

// Example of side effect detection
// pure.js - no side effects (safe to remove if unused)
export function add(a, b) { return a + b; }
export function sub(a, b) { return a - b; }

// impure.js - has side effect (must always be included)
console.log('Module loaded!'); // Top-level side effect
Array.prototype.customMethod = function() {}; // Prototype patching
```

### Coding Challenges

**Challenge 1: Build a static import/export dependency graph analyzer**
```javascript
// Create a tool that parses ES module imports/exports and
// builds a dependency graph for analysis.

class DependencyGraphAnalyzer {
  constructor() {
    this.nodes = new Map();
    this.edges = [];
  }

  addModule(name, code) {
    const imports = [];
    const exports = [];
    const lines = code.split('\n');

    for (const line of lines) {
      const trimmed = line.trim();

      // Static imports
      const importMatch = trimmed.match(/import\s+(?:\{[^}]+\}|\*\s+as\s+\w+|\w+)\s+from\s+['"]([^'"]+)['"]/);
      if (importMatch) {
        imports.push({ source: importMatch[1], statement: trimmed });
      }

      // Static exports
      const exportMatch = trimmed.match(/export\s+(?:default\s+)?(?:const|let|var|function|class)\s+(\w+)/);
      if (exportMatch) {
        exports.push(exportMatch[1]);
      }

      // Re-exports
      const reExportMatch = trimmed.match(/export\s+\{[^}]+\}\s+from\s+['"]([^'"]+)['"]/);
      if (reExportMatch) {
        imports.push({ source: reExportMatch[1], statement: trimmed, reExport: true });
      }
    }

    this.nodes.set(name, { name, imports, exports, code });
    return this;
  }

  buildGraph() {
    this.edges = [];
    for (const [name, node] of this.nodes) {
      for (const imp of node.imports) {
        this.edges.push({
          from: name,
          to: imp.source,
          statement: imp.statement,
          isReExport: imp.reExport || false
        });
      }
    }
  }

  findUnusedModules(entryPoint) {
    this.buildGraph();
    const visited = new Set();
    const queue = [entryPoint];

    while (queue.length > 0) {
      const current = queue.shift();
      if (visited.has(current)) continue;
      visited.add(current);

      const outgoing = this.edges.filter(e => e.from === current);
      for (const edge of outgoing) {
        if (this.nodes.has(edge.to)) {
          queue.push(edge.to);
        }
      }
    }

    const allModules = new Set(this.nodes.keys());
    const unused = [...allModules].filter(m => !visited.has(m));
    return {
      entryPoint,
      totalModules: allModules.size,
      usedModules: visited.size,
      unusedModules: unused,
      unusedPercent: Math.round((unused.length / allModules.size) * 100)
    };
  }

  report() {
    this.buildGraph();
    console.log('Dependency Graph Report:\n');
    console.log('Nodes:');
    for (const [name, node] of this.nodes) {
      console.log(`  ${name} (${node.exports.length} exports, ${node.imports.length} imports)`);
    }
    console.log(`\nEdges: ${this.edges.length}`);
    for (const edge of this.edges) {
      console.log(`  ${edge.from} -> ${edge.to}${edge.isReExport ? ' (re-export)' : ''}`);
    }
  }
}
```

**Challenge 2: Detect circular dependencies through static analysis**
```javascript
// Build a circular dependency detector using DFS on the
// static import graph.

class CircularDependencyDetector {
  constructor() {
    this.adjacency = new Map();
  }

  addEdge(from, to) {
    if (!this.adjacency.has(from)) {
      this.adjacency.set(from, []);
    }
    this.adjacency.get(from).push(to);
  }

  addModule(name, imports) {
    for (const imp of imports) {
      this.addEdge(name, imp);
    }
  }

  detect() {
    const visited = new Set();
    const recStack = new Set();
    const cycles = [];

    const dfs = (node, path) => {
      visited.add(node);
      recStack.add(node);
      path.push(node);

      const neighbors = this.adjacency.get(node) || [];
      for (const neighbor of neighbors) {
        if (!visited.has(neighbor)) {
          if (dfs(neighbor, [...path])) return true;
        } else if (recStack.has(neighbor)) {
          const cycle = path.slice(path.indexOf(neighbor));
          cycle.push(neighbor);
          cycles.push(cycle);
          return true;
        }
      }

      recStack.delete(node);
      return false;
    };

    for (const node of this.adjacency.keys()) {
      if (!visited.has(node)) {
        dfs(node, []);
      }
    }

    return cycles;
  }

  report() {
    const cycles = this.detect();
    if (cycles.length === 0) {
      console.log('No circular dependencies found.\n');
      return;
    }

    console.log(`Found ${cycles.length} circular dependenc(ies):\n`);
    cycles.forEach((cycle, i) => {
      console.log(`Cycle ${i + 1}: ${cycle.join(' -> ')}`);
      console.log(`  Length: ${cycle.length - 1} modules`);
      console.log('  Impact: May cause runtime errors with undefined imports');
      console.log('  Fix: Extract shared logic to a third module\n');
    });
  }
}
```

**Challenge 3: Implement a tree shaker using static module analysis**
```javascript
// Build a minimal tree shaker that uses static import/export
// analysis to remove unused code.

class StaticTreeShaker {
  constructor() {
    this.modules = new Map();
  }

  register(name, code) {
    const parsed = this.parseModule(code);
    this.modules.set(name, { ...parsed, code });
  }

  parseModule(code) {
    const exports = [];
    const lines = code.split('\n');

    // Find export declarations
    for (const line of lines) {
      const exp = line.match(/export\s+(?:default\s+)?(?:const|let|var|function|class)\s+(\w+)/);
      if (exp) exports.push({ name: exp[1], isDefault: line.includes('export default') });
    }

    // Find used names (non-export references)
    const allNames = code.match(/\b(\w+)\b/g) || [];
    const exportNames = new Set(['default', ...exports.map(e => e.name)]);
    const usedNames = new Set(
      allNames.filter(n => !exportNames.has(n) && !['const', 'let', 'var', 'function', 'return', 'if', 'else', 'for', 'while'].includes(n))
    );

    return { exports, usedNames };
  }

  shake() {
    const results = [];

    for (const [name, mod] of this.modules) {
      // Check if any export is used externally
      const externalUse = [...this.modules.values()]
        .some(other => other !== mod && other.usedNames.has(name));

      const unusedExports = mod.exports.filter(exp =>
        ![...this.modules.values()]
          .some(other => other !== mod && other.usedNames.has(exp.name))
      );

      results.push({
        module: name,
        totalSize: mod.code.length,
        exports: mod.exports.length,
        unusedExports: unusedExports.length,
        canRemove: mod.exports.length > 0 && unusedExports.length === mod.exports.length && !externalUse,
        unusedExportNames: unusedExports.map(e => e.name)
      });
    }

    return {
      results,
      totalBefore: results.reduce((s, r) => s + r.totalSize, 0),
      totalAfter: results
        .filter(r => !r.canRemove)
        .reduce((s, r) => s + r.totalSize, 0)
    };
  }

  report() {
    const result = this.shake();
    console.log('Tree Shaking Analysis:\n');
    console.log(`Total before: ${result.totalBefore}B`);
    console.log(`Total after: ${result.totalAfter}B`);
    console.log(`Saved: ${result.totalBefore - result.totalAfter}B\n`);

    for (const mod of result.results) {
      const status = mod.canRemove ? 'REMOVED' : 'KEPT';
      console.log(`[${status}] ${mod.module} (${mod.totalSize}B, ${mod.unusedExports}/${mod.exports} unused)`);
      if (mod.unusedExportNames.length > 0) {
        console.log(`       Unused: ${mod.unusedExportNames.join(', ')}`);
      }
    }
  }
}
```

## sideEffects Flag

### What It Is

The `sideEffects` field in `package.json` tells bundlers whether a package's modules have side effects when imported. If a package has `"sideEffects": false`, the bundler knows it can safely remove entire modules from the bundle if none of their exports are used. If `"sideEffects": ["*.css"]`, only CSS files have side effects.

### Why It Is Important

Without the `sideEffects` flag, bundlers must assume every module may have side effects and cannot remove any module completely—even if no exports are used. The flag enables safe, aggressive tree shaking of entire libraries. This is critical for utility libraries, UI component libraries, and polyfill packages.

### How It Works Internally

When a bundler processes a module, it checks if all imported names from that module are used elsewhere. If none are used, the bundler checks the module's `sideEffects` configuration:
- If `sideEffects: false`: remove the module entirely
- If `sideEffects: true`: keep the module (may have side effects)
- If `sideEffects: ["pattern"]`: keep if file matches pattern, remove otherwise

The bundler uses the filepath relative to the package root to match against the patterns.

### Syntax

```json
// package.json
{
  "name": "my-library",
  "version": "1.0.0",
  "sideEffects": false, // No modules in this package have side effects
  "module": "dist/index.mjs" // ES module entry point
}
```

```json
// With CSS side effects
{
  "sideEffects": ["**/*.css", "**/*.scss"]
}
```

```json
// Per-file configuration
{
  "sideEffects": [
    "./src/setup.js",     // This file has side effects
    "**/polyfill.js",     // All polyfills have side effects
    "**/*.css"            // All CSS files have side effects
  ]
}
```

### Beginner Examples

```javascript
// How sideEffects affects bundling

// Without sideEffects: false
// If you import { Button } from 'my-library',
// the bundler must include ALL modules from my-library
// because any module might have top-level side effects

// With sideEffects: false
// If you import { Button } from 'my-library',
// only the Button module and its dependencies are included

// Enabling side effects in your project
// webpack.config.js
{
  module: {
    rules: [{
      test: /\.js$/,
      sideEffects: false // Mark all JS as side-effect-free
    }]
  }
}

// Rollup (no configuration needed - analyzes tree shaking automatically)
// But respects sideEffects in package.json
```

### Intermediate Examples

```javascript
// Common side effect pitfalls

// 1. Importing from barrel that has side effects
// Buttons/index.js
import './button.css'; // Side effect!
export { Button } from './Button';
export { ButtonGroup } from './ButtonGroup';

// User imports:
import { Button } from './Buttons';
// button.css is still included (side effect)
// ButtonGroup is excluded (no side effect, not used)

// 2. Glob imports with side effects
// Sometimes bundlers can't trace glob imports
const modules = import.meta.glob('./modules/*.js');
// All matching modules are included (considered side effects)

// 3. Dynamic require
// CommonJS require inside an ES module
if (process.env.NODE_ENV === 'test') {
  require('./test-setup'); // Side effect!
}

// 4. CSS modules
import styles from './Button.module.css';
// CSS imports are side effects (they add styles to the page)
```

### Advanced Examples

```javascript
// Setting up a library for tree shaking

// Package structure:
// my-library/
//   package.json
//   src/
//     index.js
//     Button.js
//     Input.js
//     utils/
//       format.js
//       validate.js

// package.json
{
  "name": "my-library",
  "version": "1.0.0",
  "main": "dist/index.cjs.js",     // CommonJS entry
  "module": "dist/index.esm.js",    // ES module entry
  "sideEffects": false,             // All modules are side-effect-free
  "files": ["dist", "src"]          // Allow deep imports
}

// src/index.js
export { Button } from './Button';
export { Input } from './Input';
export { formatDate } from './utils/format';
export { validateEmail } from './utils/validate';

// Users can import selectively:
// import { Button } from 'my-library';
// This only includes Button.js and its dependencies
// Input.js, format.js, validate.js are excluded

// Or deep import for maximum tree shaking:
// import Button from 'my-library/src/Button';

// Testing side effects configuration
// Use webpack's stats or Rollup's output to verify
// that unused exports are actually removed

// webpack.config.js
{
  optimization: {
    usedExports: true,  // Marks unused exports
    sideEffects: true,  // Respects sideEffects flag
    providedExports: true
  }
}
```

### Coding Challenges

**Challenge 1: Build a side effect analyzer for package.json validation**
```javascript
// Create a tool that validates a package's sideEffects configuration
// and detects actual side effects in modules.

class SideEffectAnalyzer {
  constructor(packageJson) {
    this.pkg = typeof packageJson === 'string' ? JSON.parse(packageJson) : packageJson;
    this.modules = new Map();
    this.issues = [];
  }

  addModule(name, code) {
    this.modules.set(name, { code, sideEffects: this.detectSideEffects(code) });
  }

  detectSideEffects(code) {
    const effects = [];
    const lines = code.split('\n');

    for (let i = 0; i < lines.length; i++) {
      const line = lines[i].trim();

      // Top-level function calls (not in a function declaration)
      if (/^(?!function|const|let|var|class|import|export|if|for|while)\w+\s*\(/.test(line)) {
        effects.push({ line: i + 1, type: 'top-level-call', detail: line });
      }

      // Prototype patching
      if (/\w+\.prototype\.\w+\s*=/.test(line)) {
        effects.push({ line: i + 1, type: 'prototype-patch', detail: line });
      }

      // Global modifications
      if (/(window|global|globalThis|document)\.[\w.]+\s*=/.test(line) ||
          /Object\.defineProperty\(/.test(line)) {
        effects.push({ line: i + 1, type: 'global-mutation', detail: line });
      }

      // DOM operations
      if (/(document|window)\.(addEventListener|querySelector|getElement)/.test(line)) {
        effects.push({ line: i + 1, type: 'dom-side-effect', detail: line });
      }

      // Property definitions
      if (/Object\.(defineProperty|defineProperties)\(/.test(line)) {
        effects.push({ line: i + 1, type: 'property-definition', detail: line });
      }

      // Polyfills
      if (/if\s*\(!\w+\[/.test(line) || /if\s*\(typeof\s+\w+\s*===\s*['"]undefined['"]/.test(line)) {
        effects.push({ line: i + 1, type: 'polyfill-check', detail: line });
      }
    }

    return effects;
  }

  validate() {
    const { sideEffects } = this.pkg;

    // Check if sideEffects field exists
    if (sideEffects === undefined) {
      this.issues.push({ severity: 'warning', message: 'No sideEffects field in package.json. Bundlers assume all files have side effects.' });
    }

    // Check all modules against declared side effects
    for (const [name, mod] of this.modules) {
      if (mod.sideEffects.length > 0) {
        // If sideEffects is false but module has effects
        if (sideEffects === false) {
          this.issues.push({
            severity: 'error',
            module: name,
            message: `Declared sideEffects:false but found ${mod.sideEffects.length} side effect(s)`
          });
        }

        // If sideEffects is an array, check if this module is listed
        if (Array.isArray(sideEffects)) {
          const isListed = sideEffects.some(pattern => {
            if (pattern.endsWith('*')) {
              const prefix = pattern.slice(0, -1);
              return name.startsWith(prefix);
            }
            return pattern === name || name.endsWith(pattern);
          });

          if (!isListed) {
            this.issues.push({
              severity: 'error',
              module: name,
              message: `Module has side effects but is not listed in sideEffects array`
            });
          }
        }
      }
    }

    return this.issues;
  }

  generateReport() {
    this.validate();
    console.log('Side Effects Analysis Report:\n');
    console.log(`Package: ${this.pkg.name || 'unknown'}@${this.pkg.version || '0.0.0'}\n`);

    console.log('Module Side Effects:');
    for (const [name, mod] of this.modules) {
      if (mod.sideEffects.length > 0) {
        console.log(`\n  ${name} (${mod.sideEffects.length} side effect(s)):`);
        mod.sideEffects.forEach(se => {
          console.log(`    Line ${se.line}: [${se.type}] ${se.detail.slice(0, 60)}`);
        });
      }
    }

    console.log('\nValidation Issues:');
    if (this.issues.length === 0) {
      console.log('  No issues found.');
    } else {
      this.issues.forEach(i => {
        const sev = i.severity === 'error' ? 'ERROR' : 'WARN';
        const mod = i.module ? `[${i.module}] ` : '';
        console.log(`  ${sev}: ${mod}${i.message}`);
      });
    }
  }
}
```

**Challenge 2: Compare bundle sizes with different sideEffects configurations**
```javascript
// Create a benchmark that measures how different sideEffects
// configurations affect final bundle size.

class SideEffectsBenchmark {
  constructor() {
    this.scenarios = [];
    this.results = [];
  }

  addScenario(name, modulesConfig) {
    this.scenarios.push({ name, modulesConfig });
  }

  simulate(config) {
    let totalSize = 0;
    let usedSize = 0;
    let unusedModules = 0;

    for (const [modName, mod] of Object.entries(config)) {
      const { size, usedExports, totalExports, sideEffects } = mod;
      totalSize += size;

      if (usedExports === 0 && !sideEffects) {
        unusedModules++;
        // Module can be completely removed
        continue;
      }

      if (sideEffects) {
        usedSize += size;
      } else if (totalExports > 0) {
        const ratio = usedExports / totalExports;
        usedSize += Math.ceil(size * ratio);
      }
    }

    return {
      totalSize,
      bundleSize: usedSize,
      saved: totalSize - usedSize,
      reductionPercent: Math.round((1 - usedSize / totalSize) * 100),
      unusedModules
    };
  }

  run() {
    console.log('Side Effects Configuration Benchmark:\n');
    console.log('Scenario'.padEnd(35), 'Total'.padEnd(10), 'Bundle'.padEnd(10), 'Saved'.padEnd(10), 'Reduction');
    console.log('-'.repeat(80));

    for (const scenario of this.scenarios) {
      const result = this.simulate(scenario.modulesConfig);
      console.log(
        scenario.name.slice(0, 33).padEnd(35),
        `${result.totalSize}KB`.padEnd(10),
        `${result.bundleSize}KB`.padEnd(10),
        `${result.saved}KB`.padEnd(10),
        `${result.reductionPercent}%`
      );
    }

    console.log('\nNote: Setting sideEffects:false enables the most aggressive');
    console.log('tree shaking but requires actual side-effect-free code.');
    console.log('Use per-file sideEffects arrays for hybrid scenarios.');
  }
}
```

**Challenge 3: Build a linter that detects missing sideEffects declarations**
```javascript
// Create a linter that checks if library packages properly
// declare their sideEffects in package.json.

class SideEffectsLinter {
  constructor() {
    this.packages = new Map();
    this.rules = [
      {
        id: 'missing-sideEffects',
        check: (pkg) => pkg.sideEffects === undefined,
        message: 'Missing sideEffects field. Add "sideEffects": false if package has no side effects.'
      },
      {
        id: 'non-functional-libraries',
        check: (pkg) => {
          const isCSSLib = pkg.name?.includes('css') || pkg.name?.includes('style');
          const isUtility = !pkg.main?.includes('react') && !pkg.main?.includes('angular');
          return isUtility && !isCSSLib && pkg.sideEffects === undefined;
        },
        message: 'Utility library without sideEffects field. Set to false for optimal tree shaking.'
      },
      {
        id: 'style-imports',
        check: (pkg) => {
          return pkg.sideEffects === false && pkg.main?.includes('css');
        },
        message: 'Package has CSS imports but sideEffects:false. CSS files have side effects.'
      },
      {
        id: 'wrong-case',
        check: (pkg) => {
          return typeof pkg.sideEffects === 'string' &&
            pkg.sideEffects !== 'false' &&
            pkg.sideEffects !== '';
        },
        message: 'sideEffects should be boolean or array of file patterns.'
      },
      {
        id: 'no-module-field',
        check: (pkg) => {
          return !pkg.module && !pkg.exports;
        },
        message: 'Missing "module" or "exports" field. ES module entry point required for tree shaking.'
      }
    ];
  }

  addPackage(name, pkg) {
    this.packages.set(name, pkg);
  }

  lint() {
    const issues = [];

    for (const [name, pkg] of this.packages) {
      for (const rule of this.rules) {
        if (rule.check(pkg)) {
          issues.push({
            package: name,
            rule: rule.id,
            message: rule.message
          });
        }
      }
    }

    return issues;
  }

  report() {
    const issues = this.lint();
    console.log('Side Effects Linter Report:\n');
    console.log(`Checked ${this.packages.size} packages.\n`);

    if (issues.length === 0) {
      console.log('All packages properly configured.');
      return;
    }

    const grouped = new Map();
    for (const issue of issues) {
      if (!grouped.has(issue.package)) grouped.set(issue.package, []);
      grouped.get(issue.package).push(issue);
    }

    console.log(`Found ${issues.length} issue(s):\n`);
    for (const [pkg, pkgIssues] of grouped) {
      console.log(`${pkg}:`);
      pkgIssues.forEach(i => console.log(`  - ${i.message}`));
      console.log();
    }
  }
}
```

## Tree Shaking with Bundlers

### What It Is

Tree shaking implementation varies across bundlers. Rollup pioneered tree shaking and has the most thorough implementation. Webpack added tree shaking later and has improved significantly. esbuild has fast but less aggressive tree shaking. Parcel handles some tree shaking but focuses on zero-config approach.

### Why It Is Important

Choosing the right bundler and configuring it correctly determines how effectively your bundle is optimized. Understanding each bundler's strengths and limitations helps achieve the smallest possible bundle size.

### How It Works Internally

**Rollup**: Static analysis + live code bindings. Rollup analyzes the module graph, tracks which exports are used, and physically removes unused code. It produces cleaner output with minimal overhead.

**Webpack**: Uses `optimization.usedExports` (marks unused) + terser for removal. Webpack's tree shaking works through "side effects" awareness. Webpack marks exports as unused and lets the minifier remove them.

**esbuild**: Fast but conservative tree shaking. esbuild removes dead code within modules but does not do deep cross-module tree shaking.

**Parcel**: Automatic tree shaking based on scope hoisting. Parcel's scope hoisting bundles modules into a single scope, enabling better dead code elimination.

### Syntax

```javascript
// Rollup (most effective tree shaking)
// rollup.config.js
export default {
  input: 'src/main.js',
  output: { file: 'dist/bundle.js', format: 'esm' },
  treeshake: {
    moduleSideEffects: false,
    propertyReadSideEffects: false,
    tryCatchDeoptimization: false,
    unknownGlobalSideEffects: false
  }
};

// Webpack
// webpack.config.js
module.exports = {
  mode: 'production',
  optimization: {
    usedExports: true,
    sideEffects: true,
    concatenateModules: true,
    minimize: true,
    minimizer: [new TerserPlugin({ extractComments: false })]
  }
};

// esbuild (built-in tree shaking)
// esbuild.config.js
require('esbuild').build({
  entryPoints: ['src/main.js'],
  bundle: true,
  minify: true,
  treeShaking: true,
  outfile: 'dist/bundle.js'
});
```

### Beginner Examples

```javascript
// Practical tree shaking examples

// Example 1: Only import what you need
// Instead of:
import * as lodash from 'lodash';
lodash.map(...);
lodash.filter(...);

// Do:
import map from 'lodash/map';
import filter from 'lodash/filter';

// Example 2: Use library-specific ES module builds
// Lodash: lodash-es (ES module version)
// Moment: use date-fns or dayjs (tree-shakeable)
// RxJS: v6+ supports tree shaking

// Example 3: Avoid default imports for libraries
// Bad (harder to tree shake):
import React from 'react';
React.useEffect(...);

// Good (named imports):
import { useEffect } from 'react';

// Example 4: Configure jQuery-style plugins carefully
// Without tree shaking:
import 'jquery';
import 'jquery-ui'; // Sets up global jQuery plugins

// With tree shaking: mark these as side effects
// sideEffects: ["jquery-ui"]
```

### Real-World Use Cases

- UI component libraries (Ant Design, Material-UI) - users import only needed components
- Utility libraries (lodash-es, date-fns, RxJS) - import only needed functions
- Internal shared libraries - modularize for tree shaking
- Large applications with many dependencies

### Coding Challenges

**Challenge 1: Implement a bundler comparison tool**
```javascript
// Create a tool that compares tree shaking effectiveness
// across different bundlers (Webpack, Rollup, esbuild).

class BundlerComparison {
  constructor() {
    this.bundlers = {};
    this.scenarios = [];
  }

  defineBundler(name, shakeFn) {
    this.bundlers[name] = shakeFn;
  }

  addScenario(name, modules) {
    this.scenarios.push({ name, modules });
  }

  run() {
    console.log('Bundler Tree Shaking Comparison:\n');
    const headers = ['Scenario', ...Object.keys(this.bundlers), 'Best'];
    console.log(headers.map(h => h.padEnd(15)).join(''));
    console.log('-'.repeat(headers.length * 15));

    for (const scenario of this.scenarios) {
      const results = [];
      for (const [name, shakeFn] of Object.entries(this.bundlers)) {
        const size = shakeFn(scenario.modules);
        results.push({ name, size });
      }
      const best = results.reduce((min, r) => r.size < min.size ? r : min, results[0]);
      console.log(
        scenario.name.padEnd(15) +
        results.map(r => `${r.size}B`.padEnd(15)).join('') +
        best.name.padEnd(15)
      );
    }

    return this.scenarios.reduce((report, s) => {
      report[s.name] = {};
      for (const [name, fn] of Object.entries(this.bundlers)) {
        report[s.name][name] = fn(s.modules);
      }
      return report;
    }, {});
  }
}

// Simple tree shaker simulation (Rollup-like)
function rollupShaker(modules) {
  let total = 0;
  const used = new Set(Object.keys(modules).filter(k => modules[k].imported));
  for (const [name, mod] of Object.entries(modules)) {
    if (used.has(name) || mod.sideEffects) {
      if (mod.exports.length > 0) {
        const usedExports = mod.exports.filter(e => mod.usedExportNames?.includes(e));
        const ratio = usedExports.length / Math.max(mod.exports.length, 1);
        total += Math.ceil(mod.size * Math.max(ratio, 0.1));
      } else {
        total += mod.size;
      }
    }
  }
  return total;
}

// Webpack-like (less aggressive)
function webpackShaker(modules) {
  let total = 0;
  for (const mod of Object.values(modules)) {
    const keep = mod.imported || mod.sideEffects || mod.exports.length === 0;
    if (keep) {
      const usedExports = mod.exports.filter(e => mod.usedExportNames?.includes(e));
      const ratio = usedExports.length / Math.max(mod.exports.length, 1);
      total += Math.ceil(mod.size * Math.max(ratio, 0.3));
    }
  }
  return total;
}

// esbuild-like (fast, minimal)
function esbuildShaker(modules) {
  let total = 0;
  for (const mod of Object.values(modules)) {
    if (mod.imported || mod.sideEffects) {
      total += mod.size;
    }
  }
  return total;
}
```

**Challenge 2: Build a tree shaking audit tool**
```javascript
// Create a bundle auditor that reports which imports could
// be further optimized by more aggressive tree shaking.

class TreeShakingAuditor {
  constructor() {
    this.sources = [];
  }

  addSource(name, code) {
    this.sources.push({ name, code });
  }

  audit() {
    console.log('Tree Shaking Audit Report:\n');
    const issues = [];

    for (const source of this.sources) {
      // Check for namespace imports
      const namespaceImports = source.code.match(/import\s+\*\s+as\s+(\w+)\s+from\s+['"]([^'"]+)['"]/g);
      if (namespaceImports) {
        issues.push({
          severity: 'warning',
          file: source.name,
          message: `Namespace import prevents tree shaking of specific exports`,
          detail: namespaceImports[0]
        });
      }

      // Check for default imports from libraries
      const defaultImports = source.code.match(/import\s+(\w+)\s+from\s+['"]([^'"]+)['"]/g);
      if (defaultImports) {
        for (const imp of defaultImports) {
          const libMatch = imp.match(/from\s+['"]([^'"]+)['"]/);
          if (libMatch && !libMatch[1].startsWith('.')) {
            issues.push({
              severity: 'info',
              file: source.name,
              message: `Default import of '${libMatch[1]}' may import unused code. Use named imports instead.`,
              detail: imp
            });
          }
        }
      }

      // Check for barrel imports
      const barrelImports = source.code.match(/import\s+\{[^}]+\}\s+from\s+['"]([^'"]+\/index)['"]/);
      if (barrelImports) {
        issues.push({
          severity: 'warning',
          file: source.name,
          message: `Barrel import from '${barrelImports[1]}' may import many unused modules`,
          detail: barrelImports[0]
        });
      }
    }

    const bySeverity = { error: [], warning: [], info: [] };
    issues.forEach(i => bySeverity[i.severity].push(i));

    for (const [severity, items] of Object.entries(bySeverity)) {
      if (items.length === 0) continue;
      console.log(`[${severity.toUpperCase()}] ${items.length} issue(s)`);
      items.forEach(i => console.log(`  ${i.file}: ${i.message}`));
      console.log();
    }

    if (issues.length === 0) {
      console.log('No tree shaking issues found.');
    }

    return issues;
  }
}
```

**Challenge 3: Compare build times vs tree shaking effectiveness**
```javascript
// Create a benchmark that measures the tradeoff between
// build time and bundle size reduction.

class BuildTimeBenchmark {
  constructor() {
    this.configs = [];
    this.results = [];
  }

  addConfig(name, treeShakeOptions) {
    this.configs.push({ name, options: treeShakeOptions, simulatedSize: 0, simulatedTime: 0 });
  }

  simulateModule(size, exports, usedExports, sideEffects) {
    const ratio = usedExports.length / Math.max(exports, 1);
    const kept = sideEffects ? size : Math.ceil(size * Math.max(ratio, 0.05));
    const eliminated = size - kept;
    const analysisTime = Math.ceil(exports * 0.1);
    return { kept, eliminated, analysisTime };
  }

  run(modules) {
    console.log('Build Time vs Tree Shaking Effectiveness:\n');
    console.log('Configuration'.padEnd(35), 'Size (KB)'.padEnd(15), 'Time (ms)'.padEnd(15), 'Effectiveness');
    console.log('-'.repeat(85));

    for (const config of this.configs) {
      let totalSize = 0;
      let keptSize = 0;
      let totalTime = 0;

      for (const mod of modules) {
        const aggressive = config.options.level === 'aggressive';
        const result = this.simulateModule(
          mod.size,
          mod.exports,
          aggressive ? mod.usedExportNames : mod.usedExportNames.slice(0, Math.ceil(mod.exports / 2)),
          mod.sideEffects && !config.options.removeSideEffects
        );

        keptSize += result.kept;
        totalSize += mod.size;
        totalTime += result.analysisTime;
      }

      const effectiveness = Math.round((1 - keptSize / totalSize) * 100);
      config.simulatedSize = keptSize;
      config.simulatedTime = totalTime;

      console.log(
        config.name.slice(0, 33).padEnd(35),
        `${Math.round(keptSize / 1024)}KB`.padEnd(15),
        `${totalTime}ms`.padEnd(15),
        `${effectiveness}%`
      );
    }

    console.log('\nTradeoff Analysis:');
    const fastest = this.results.length > 0 ? this.results.reduce((a, b) => a.time < b.time ? a : b) : null;
    const smallest = this.results.length > 0 ? this.results.reduce((a, b) => a.size < b.size ? a : b) : null;
    if (fastest) console.log(`Fastest build: ${fastest.name} (${fastest.time}ms)`);
    if (smallest) console.log(`Smallest bundle: ${smallest.name} (${Math.round(smallest.size / 1024)}KB)`);
  }
}
```

### Common Mistakes

- Using CommonJS modules (require/module.exports) - not tree-shakeable
- Creating barrel files that re-export everything (can defeat tree shaking)
- Not configuring `sideEffects` for CSS or other non-JS imports
- Using `import * as X` (makes all exports "used" via namespace)
- Not checking if a library provides ES module build (check `"module"` field)
- Importing from the wrong path (dist vs src)

### Best Practices

- Use ES modules throughout your project
- Prefer named imports over default imports for tree-shakeable libraries
- Configure `sideEffects: false` in your own packages
- Use direct imports for library functions: `import map from 'lodash/map'`
- Check library support: look for `"module"` field in package.json
- Verify tree shaking with bundle analyzers (webpack-bundle-analyzer)
- Avoid barrel files that re-export many modules
- Use `import()` for code splitting (lazy loading)
- Configure CSS imports as side effects

### Performance Considerations

- Tree shaking is done at build time, zero runtime cost
- More aggressive tree shaking may increase build time
- Bundle analysis with source maps shows tree shaking effectiveness
- Smaller bundles = faster download, parse, and execute
- Tree shaking doesn't affect runtime performance of remaining code

### Interview Questions

**Q: How does tree shaking work and why does it require ES modules?** A: Tree shaking analyzes the module graph statically to determine which exports are actually imported and used. ES modules have static, top-level import/export syntax that can be analyzed without executing code. CommonJS's dynamic `require()` prevents this analysis because module dependencies can be conditional or computed.

**Q: What is the `sideEffects` flag and why is it important?** A: The `sideEffects` flag in `package.json` tells bundlers whether importing a module has side effects beyond what it exports. If `false`, bundlers can safely remove entire modules if none of their exports are used. Without this flag, bundlers must assume every module could have side effects and cannot remove any unused module entirely.

### Related Topics

- ES module specification
- CommonJS vs ES modules
- Code splitting and lazy loading
- Bundle analysis and optimization
- Module bundler architecture
- Minification (Terser, UglifyJS)
- webpack/rollup configuration
- Package.json module field
- Library authoring best practices
