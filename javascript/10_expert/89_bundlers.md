# Bundlers - Webpack, Rollup, Vite, Parcel, bundling strategies

## Introduction

Module bundlers are tools that resolve JavaScript modules' dependencies and package them into optimized bundles for browser delivery. They handle module formats (ES modules, CommonJS, AMD), transform code (via loaders/plugins), optimize output (minification, tree shaking, code splitting), and manage assets (CSS, images, fonts). Choosing the right bundler and configuration is a critical architectural decision for modern web applications.

## Webpack

### What It Is

Webpack is the most widely used JavaScript bundler, powering major frameworks like React (Create React App) and Next.js. It uses a configuration-driven approach with a vast plugin ecosystem. Webpack treats everything as a module (JavaScript, CSS, images, fonts) and processes them through loaders.

### Why It Is Important

Webpack's dominance means most developers encounter it in production applications. Its flexibility allows it to handle almost any build scenario, but this flexibility comes with complexity. Understanding webpack is essential for debugging build issues and optimizing production bundles.

### How It Works Internally

Webpack builds a dependency graph starting from one or more entry points. Each file is processed through a pipeline: matching loaders transform the file (e.g., TypeScript → JavaScript), and the result is included in the graph. Webpack then outputs bundles based on the configuration (code splitting, multiple entry points, etc.).

Key concepts: Entry, Output, Loaders, Plugins, Module Resolution, Chunks, Assets.

### Syntax

```javascript
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  mode: 'production',
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
    clean: true
  },
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: { presets: ['@babel/preset-env', '@babel/preset-react'] }
        }
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.(png|jpg|gif)$/,
        type: 'asset/resource'
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({ template: './src/index.html' })
  ],
  optimization: {
    splitChunks: { chunks: 'all' },
    usedExports: true,
    sideEffects: true
  }
};
```

### Beginner Examples

```javascript
// Basic webpack configuration for a React app

// Entry: where to start building the graph
// Output: where to put the result
// Loaders: transform files before adding to graph
// Plugins: extend webpack functionality

// Simple configuration
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: __dirname + '/dist'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: 'babel-loader'
      }
    ]
  }
};
```

### Intermediate Examples

```javascript
// Webpack with multiple entry points and code splitting

// webpack.config.js
module.exports = {
  entry: {
    main: './src/index.js',
    admin: './src/admin.js',
    vendor: ['react', 'react-dom']
  },
  output: {
    filename: '[name].[contenthash:8].js',
    chunkFilename: '[name].[contenthash:8].chunk.js',
    path: path.resolve(__dirname, 'dist'),
    publicPath: '/assets/'
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          priority: 10
        },
        common: {
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true
        }
      }
    },
    runtimeChunk: 'single',
    minimize: true,
    minimizer: ['...', new CssMinimizerPlugin()]
  }
};
```

### Advanced Examples

```javascript
// Webpack plugin development - a custom build-time plugin

class BundleStatsPlugin {
  constructor(options) {
    this.options = options;
  }

  apply(compiler) {
    compiler.hooks.emit.tapAsync('BundleStatsPlugin', (compilation, callback) => {
      const stats = {
        entries: {},
        totalSize: 0,
        assetCount: 0
      };

      for (const [filename, asset] of Object.entries(compilation.assets)) {
        if (filename.endsWith('.js')) {
          const size = asset.size();
          stats.totalSize += size;
          stats.assetCount++;
          stats.entries[filename] = {
            size,
            chunks: asset.info?.chunkNames || []
          };
        }
      }

      const statsJson = JSON.stringify(stats, null, 2);
      compilation.assets['bundle-stats.json'] = {
        source: () => statsJson,
        size: () => statsJson.length
      };

      callback();
    });
  }
}

// webpack.config.js
module.exports = {
  plugins: [
    new BundleStatsPlugin({
      output: 'bundle-stats.json'
    })
  ]
};
```

### Coding Challenges

**Challenge 1: Build a minimal webpack-like bundler**
```javascript
// Create a simple bundler that resolves dependencies
// and concatenates modules into a bundle.

class MiniBundler {
  constructor(entryPoint) {
    this.entryPoint = entryPoint;
    this.modules = new Map();
    this.moduleId = 0;
  }

  resolveModule(filename) {
    const fs = require('fs');
    const path = require('path');
    const code = fs.readFileSync(filename, 'utf-8');
    return this.parseModule(filename, code);
  }

  parseModule(filename, code) {
    const id = this.moduleId++;
    const deps = [];
    const importRegex = /require\(['"]([^'"]+)['"]\)/g;
    let match;

    while ((match = importRegex.exec(code)) !== null) {
      const depPath = require.resolve(
        match[1].startsWith('.')
          ? require('path').join(require('path').dirname(filename), match[1])
          : match[1]
      );
      deps.push(depPath);
    }

    this.modules.set(id, { id, filename, code, deps: [] });

    for (const depPath of deps) {
      const depId = this.resolveModule(depPath).id;
      this.modules.get(id).deps.push(depId);
    }

    return { id, deps };
  }

  bundle() {
    this.resolveModule(this.entryPoint);

    let output = '(function(modules) {\n';
    output += '  const installed = {};\n';
    output += '  function require(id) {\n';
    output += '    if (installed[id]) return installed[id].exports;\n';
    output += '    const module = { exports: {} };\n';
    output += '    installed[id] = module;\n';
    output += '    modules[id](module, module.exports, require);\n';
    output += '    return module.exports;\n';
    output += '  }\n';
    output += `  require(0);\n`;
    output += '})({\n';

    for (const [id, mod] of this.modules) {
      const wrapped = mod.code.replace(
        /require\(['"]([^'"]+)['"]\)/g,
        (_, depName) => {
          const depPath = require.resolve(
            depName.startsWith('.')
              ? require('path').join(require('path').dirname(mod.filename), depName)
              : depName
          );
          for (const [otherId, otherMod] of this.modules) {
            if (otherMod.filename === depPath) return `require(${otherId})`;
          }
          return `require("${depName}")`;
        }
      );
      output += `  ${id}: function(module, exports, require) {\n${wrapped}\n  },\n`;
    }

    output += '});\n';
    return output;
  }
}
```

**Challenge 2: Write a webpack config analyzer**
```javascript
// Create a tool that analyzes webpack configurations
// and suggests optimization improvements.

class WebpackConfigAnalyzer {
  constructor(config) {
    this.config = config;
    this.issues = [];
  }

  analyze() {
    this.checkMode();
    this.checkDevtool();
    this.checkOptimization();
    this.checkPerformance();
    this.checkCache();
    return this.issues;
  }

  checkMode() {
    if (!this.config.mode || this.config.mode === 'none') {
      this.addIssue('error', 'No mode set. Set mode to "production" for optimized builds.');
    }
  }

  checkDevtool() {
    const { devtool } = this.config;
    if (devtool && devtool.includes('eval')) {
      this.addIssue('warning', 'Using eval-based source maps (devtool includes "eval") prevents code caching in some browsers.');
    }
  }

  checkOptimization() {
    if (!this.config.optimization) {
      this.addIssue('error', 'No optimization configuration. Add optimization.splitChunks for code splitting.');
      return;
    }

    const { optimization } = this.config;
    if (!optimization.splitChunks) {
      this.addIssue('warning', 'No splitChunks configuration. Large vendor bundles won\'t be split.');
    }

    if (optimization.minimize === false && this.config.mode === 'production') {
      this.addIssue('warning', 'Minimization is disabled in production mode. Enable optimization.minimize.');
    }
  }

  checkPerformance() {
    if (!this.config.performance) {
      this.addIssue('info', 'No performance hints configured. Add performance: { hints: "warning" } for bundle size warnings.');
    }
  }

  checkCache() {
    if (!this.config.cache) {
      this.addIssue('info', 'No cache configuration. Add cache: { type: "filesystem" } for faster rebuilds.');
    }
  }

  addIssue(severity, message) {
    this.issues.push({ severity, message });
  }

  report() {
    console.log('Webpack Config Analysis:\n');
    if (this.issues.length === 0) {
      console.log('No issues found. Configuration looks good!\n');
      return;
    }

    const bySeverity = { error: [], warning: [], info: [] };
    this.issues.forEach(i => bySeverity[i.severity].push(i));

    for (const [sev, items] of Object.entries(bySeverity)) {
      if (items.length === 0) continue;
      console.log(`[${sev.toUpperCase()}]`);
      items.forEach(i => console.log(`  ${i.message}`));
      console.log();
    }
  }
}
```

**Challenge 3: Implement a hot module replacement (HMR) client**
```javascript
// Build a minimal HMR client that accepts module updates
// without full page reload.

class HMRClient {
  constructor(options = {}) {
    this.options = {
      host: options.host || 'localhost',
      port: options.port || 8080,
      path: options.path || '/__webpack_hmr',
      ...options
    };
    this.modules = new Map();
    this.accepting = new Set();
    this.connection = null;
    this.reconnectAttempts = 0;
    this.maxReconnect = 10;
  }

  connect() {
    const protocol = location.protocol === 'https:' ? 'wss:' : 'ws:';
    const url = `${protocol}//${this.options.host}:${this.options.port}${this.options.path}`;

    this.connection = new WebSocket(url);
    this.connection.onopen = () => {
      this.reconnectAttempts = 0;
      console.log('[HMR] Connected');
    };

    this.connection.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleMessage(data);
    };

    this.connection.onclose = () => {
      if (this.reconnectAttempts < this.maxReconnect) {
        this.reconnectAttempts++;
        setTimeout(() => this.connect(), 1000 * this.reconnectAttempts);
      }
    };
  }

  handleMessage(data) {
    switch (data.type) {
      case 'hot':
        console.log(`[HMR] ${data.message || 'Update available'}`);
        break;
      case 'update':
        this.applyUpdate(data.modules);
        break;
      case 'error':
        console.error('[HMR]', data.error);
        break;
      case 'reload':
        window.location.reload();
        break;
    }
  }

  accept(modulePath, callback) {
    this.accepting.add(modulePath);
    this.modules.set(modulePath, { callback });
  }

  applyUpdate(updatedModules) {
    let updated = false;

    for (const modPath of updatedModules) {
      if (this.accepting.has(modPath)) {
        const mod = this.modules.get(modPath);
        if (mod && mod.callback) {
          mod.callback(modPath);
          updated = true;
        }
      }
    }

    if (!updated) {
      console.log('[HMR] No accepting modules found, reloading...');
      window.location.reload();
    }
  }

  disconnect() {
    if (this.connection) {
      this.connection.close();
    }
  }
}
```

## Rollup

### What It Is

Rollup is a module bundler focused on ES modules, tree shaking, and producing clean, optimized output. It pioneered tree shaking and is the preferred bundler for libraries (React, Vue, D3, and many others use Rollup). Rollup produces smaller, more efficient bundles than webpack for library distribution.

### Why It Is Important

Rollup is the standard for JavaScript library bundling because it produces cleaner output with better tree shaking than webpack. Its focus on ES modules makes it ideal for publishing npm packages that should be tree-shakeable by consumers.

### How It Works Internally

Rollup parses ES modules, builds a module graph, and physically removes unused exports during code generation. It uses scope hoisting to combine modules into a single scope, reducing runtime overhead. Rollup's plugin system is simpler than webpack's but powerful.

### Syntax

```javascript
// rollup.config.js
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';
import terser from '@rollup/plugin-terser';

export default {
  input: 'src/index.js',
  output: [
    {
      file: 'dist/bundle.cjs.js',
      format: 'cjs' // CommonJS
    },
    {
      file: 'dist/bundle.esm.js',
      format: 'esm' // ES Module
    },
    {
      file: 'dist/bundle.umd.js',
      format: 'umd',
      name: 'MyLibrary' // UMD for browser
    }
  ],
  plugins: [
    resolve(),      // Resolve node_modules
    commonjs(),     // Convert CJS to ESM
    babel({ presets: ['@babel/preset-env'] }),
    terser()        // Minify
  ],
  external: ['react', 'react-dom'] // Peer dependencies
};
```

### Beginner Examples

```javascript
// Simple Rollup build for a library

// rollup.config.js
export default {
  input: 'src/math.js',
  output: [
    { file: 'dist/math.cjs.js', format: 'cjs' },
    { file: 'dist/math.esm.js', format: 'esm' }
  ]
};

// src/math.js
export function add(a, b) { return a + b; }
export function sub(a, b) { return a - b; }

// Result: dist/math.esm.js
// function add(a, b) { return a + b; }
// function sub(a, b) { return a - b; }
// export { add, sub };
```

### Intermediate Examples

```javascript
// Rollup with TypeScript and multiple outputs

// rollup.config.js
import typescript from '@rollup/plugin-typescript';
import { dts } from 'rollup-plugin-dts';

export default [
  {
    input: 'src/index.ts',
    output: [
      { file: 'dist/index.js', format: 'cjs', exports: 'named' },
      { file: 'dist/index.esm.js', format: 'esm' },
      { file: 'dist/index.umd.js', format: 'umd', name: 'MyLib', globals: { react: 'React' } }
    ],
    plugins: [typescript()],
    external: ['react', 'react-dom']
  },
  {
    input: 'src/index.ts',
    output: { file: 'dist/index.d.ts', format: 'esm' },
    plugins: [dts()]
  }
];
```

### Advanced Examples

```javascript
// Custom Rollup plugin for bundle analysis

function bundleAnalyzer() {
  return {
    name: 'bundle-analyzer',
    generateBundle(_, bundle) {
      const modules = {};
      let totalSize = 0;

      for (const [fileName, chunk] of Object.entries(bundle)) {
        if (chunk.type === 'chunk') {
          const size = Buffer.byteLength(chunk.code, 'utf-8');
          totalSize += size;
          modules[fileName] = {
            size,
            exports: chunk.exports,
            imports: chunk.imports,
            modules: Object.keys(chunk.modules)
          };
        }
      }

      console.log(`\nBundle Analysis:`);
      console.log(`Total size: ${(totalSize / 1024).toFixed(2)} KB`);
      console.log(`Chunks: ${Object.keys(modules).length}\n`);

      for (const [name, info] of Object.entries(modules)) {
        const sizeKB = (info.size / 1024).toFixed(2);
        console.log(`${name}: ${sizeKB} KB`);
        if (info.imports.length > 0) {
          console.log(`  imports: ${info.imports.join(', ')}`);
        }
      }
    }
  };
}
```

### Coding Challenges

**Challenge 1: Build a Rollup-like tree shaker**
```javascript
// Implement the core tree shaking algorithm that Rollup uses:
// trace exports through the module graph and remove unused ones.

class RollupTreeShaker {
  constructor() {
    this.modules = new Map();
  }

  addModule(name, exports, body) {
    this.modules.set(name, {
      exports: exports.map(e => ({ name: e, used: false })),
      body,
      referencedNames: this.extractReferences(body)
    });
  }

  extractReferences(body) {
    const refs = new Set();
    const idPattern = /\b([a-zA-Z_$][\w$]*)\b/g;
    let match;
    const reserved = new Set(['export', 'import', 'function', 'const', 'let', 'var',
      'return', 'if', 'else', 'for', 'while', 'class', 'new', 'this', 'typeof',
      'instanceof', 'void', 'delete', 'throw', 'try', 'catch', 'finally']);

    while ((match = idPattern.exec(body)) !== null) {
      if (!reserved.has(match[1])) {
        refs.add(match[1]);
      }
    }
    return refs;
  }

  trace(entryModule) {
    const queue = [entryModule];
    const visited = new Set();

    while (queue.length > 0) {
      const current = queue.shift();
      if (visited.has(current)) continue;
      visited.add(current);

      const mod = this.modules.get(current);
      if (!mod) continue;

      // Mark all referenced exports as used
      for (const exp of mod.exports) {
        if (mod.referencedNames.has(exp.name)) {
          exp.used = true;
        }
      }
    }

    return { visited: visited.size, total: this.modules.size };
  }

  shake(entryModule) {
    this.trace(entryModule);
    const result = [];

    for (const [name, mod] of this.modules) {
      const unused = mod.exports.filter(e => !e.used).map(e => e.name);
      result.push({
        module: name,
        totalExports: mod.exports.length,
        removed: unused.length,
        kept: mod.exports.length - unused.length,
        names: unused
      });
    }

    return result;
  }

  report(entryModule) {
    const result = this.shake(entryModule);
    console.log('Rollup-style Tree Shaking:\n');

    let totalRemoved = 0;
    let totalExports = 0;

    for (const mod of result) {
      totalRemoved += mod.removed;
      totalExports += mod.totalExports;
      console.log(`${mod.module}:`);
      console.log(`  ${mod.kept}/${mod.totalExports} exports kept (removed ${mod.removed})`);
      if (mod.names.length > 0 && mod.names.length <= 3) {
        console.log(`  removed: ${mod.names.join(', ')}`);
      }
    }

    console.log(`\nTotal: removed ${totalRemoved}/${totalExports} exports`);
  }
}
```

**Challenge 2: Create a Rollup config generator**
```javascript
// Build a tool that generates optimal Rollup configurations
// based on project analysis.

class RollupConfigGenerator {
  constructor(projectInfo) {
    this.project = projectInfo;
    this.plugins = [];
    this.config = {};
  }

  analyze() {
    const { type, hasTypeScript, hasReact, targets, external } = this.project;

    this.config = {
      input: 'src/index.js',
      output: []
    };

    // ESM build (always)
    this.config.output.push({
      file: 'dist/index.esm.js',
      format: 'esm',
      sourcemap: true
    });

    // CJS build (for Node.js)
    if (type === 'library') {
      this.config.output.push({
        file: 'dist/index.cjs.js',
        format: 'cjs',
        exports: 'named',
        sourcemap: true
      });
    }

    // UMD build (for browsers without bundler)
    if (targets?.includes('browser') || targets?.includes('umd')) {
      this.config.output.push({
        file: 'dist/index.umd.js',
        format: 'umd',
        name: this.project.globalName || 'MyLib',
        sourcemap: true,
        globals: this.project.globals || {}
      });
    }

    // TypeScript plugin
    if (hasTypeScript) {
      this.plugins.push('@rollup/plugin-typescript');
      this.config.output.push({
        file: 'dist/index.d.ts',
        format: 'esm',
        plugins: ['rollup-plugin-dts']
      });
    }

    // React plugin
    if (hasReact) {
      this.plugins.push('@rollup/plugin-babel', '@rollup/plugin-node-resolve');
    }

    // External dependencies
    if (external) {
      this.config.external = Array.isArray(external) ? external : [external];
    }

    // Code splitting
    if (this.project.codeSplitting) {
      this.config.output[0].dir = 'dist';
      this.config.output[0].entryFileNames = '[name].esm.js';
      this.config.output[0].chunkFileNames = 'chunks/[name]-[hash].esm.js';
    }

    return this.config;
  }

  generate() {
    this.analyze();
    console.log('Generated Rollup Configuration:\n');

    console.log(`Input: ${this.config.input}`);
    console.log('Outputs:');
    this.config.output.forEach(o => {
      console.log(`  [${o.format}] ${o.file || o.dir}`);
    });

    if (this.plugins.length > 0) {
      console.log('\nRecommended plugins:');
      this.plugins.forEach(p => console.log(`  - ${p}`));
    }

    if (this.config.external) {
      console.log('\nExternal dependencies:');
      this.config.external.forEach(e => console.log(`  - ${e}`));
    }
  }
}
```

**Challenge 3: Compare Rollup's scope hoisting vs Webpack's module wrapper**
```javascript
// Demonstrate the performance difference between Rollup's
// scope hoisting and Webpack's module wrappers.

class ModuleOutputComparator {
  constructor() {
    this.modules = [];
  }

  addModule(name, code) {
    this.modules.push({ name, code });
  }

  webpackOutput() {
    const modules = this.modules.map((m, i) => {
      return `${i}: function(module, exports, require) {\n  ${m.code}\n}`;
    });

    return [
      '(function(modules) {',
      '  function require(id) {',
      '    var module = { exports: {} };',
      '    modules[id](module, module.exports, require);',
      '    return module.exports;',
      '  }',
      '  return require(0);',
      '})({',
      modules.join(',\n'),
      '})'
    ].join('\n');
  }

  rollupOutput() {
    const lines = [];
    for (const m of this.modules) {
      lines.push(`// ${m.name}`);
      lines.push(m.code);
    }
    return lines.join('\n');
  }

  compare() {
    const webpack = this.webpackOutput();
    const rollup = this.rollupOutput();

    console.log('Output Comparison:\n');
    console.log(`Webpack output: ${webpack.length} bytes`);
    console.log(`Rollup output: ${rollup.length} bytes`);
    console.log(`Rollup is ${(webpack.length / rollup.length).toFixed(1)}x smaller\n`);

    console.log('--- Webpack (wrapped output) ---');
    console.log(webpack.slice(0, 300) + '...\n');
    console.log('--- Rollup (scope-hoisted output) ---');
    console.log(rollup.slice(0, 300) + '...');

    return { webpackSize: webpack.length, rollupSize: rollup.length };
  }
}
```

## Vite

### What It Is

Vite (French for "fast") is a next-generation build tool that leverages native ES modules in development and Rollup for production builds. It provides instant server start (no bundling during development) and fast Hot Module Replacement (HMR). Vite is the default bundler for Vue 3 and has gained significant adoption for React, Svelte, and other frameworks.

### Why It Is Important

Vite addresses webpack's main weakness: slow development startup. By serving modules natively in the browser during development, Vite avoids the time-consuming bundling process. Native ESM support in modern browsers makes this approach viable. Vite represents the future of build tooling.

### How It Works Internally

**Development**: Vite serves source files directly to the browser as native ES modules. It transforms files on demand (TypeScript → JavaScript, SFC → JS) and sends them to the browser. HMR is performed over WebSocket for individual modules without full reloads.

**Production**: Vite uses Rollup under the hood for production builds, inheriting Rollup's superior tree shaking and output optimization.

### Syntax

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src')
    }
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          utils: ['./src/utils']
        }
      }
    },
    target: 'es2020',
    minify: 'terser',
    sourcemap: true
  },
  server: {
    port: 3000,
    proxy: {
      '/api': 'http://localhost:8080'
    }
  }
});
```

### Beginner Examples

```javascript
// Basic Vite setup for a Vanilla JS project

// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  root: '.',
  build: {
    outDir: 'dist',
    sourcemap: true
  }
});

// index.html (Vite automatically injects the script tag)
// <script type="module" src="/src/main.js"></script>

// src/main.js (import directly - no bundling in dev)
import { createApp } from './app.js';
// In dev: browser loads createApp as a separate HTTP request

document.querySelector('#app').innerHTML = '<h1>Hello Vite!</h1>';
```

### Intermediate Examples

```javascript
// Vite with environment variables and multi-page setup

// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';

// Environment variable usage
export default defineConfig(({ command, mode }) => {
  const isProduction = mode === 'production';

  return {
    plugins: [react()],
    define: {
      __APP_VERSION__: JSON.stringify('1.0.0')
    },
    resolve: {
      alias: {
        '@': resolve(__dirname, 'src'),
        '@components': resolve(__dirname, 'src/components')
      }
    },
    build: {
      rollupOptions: {
        input: {
          main: resolve(__dirname, 'index.html'),
          admin: resolve(__dirname, 'admin.html')
        },
        output: {
          manualChunks(id) {
            if (id.includes('node_modules')) {
              if (id.includes('react')) return 'vendor-react';
              if (id.includes('lodash')) return 'vendor-utils';
              return 'vendor';
            }
          }
        }
      },
      minify: isProduction ? 'terser' : false,
      sourcemap: !isProduction
    }
  };
});
```

### Advanced Examples

```javascript
// Vite plugin development - transform custom file types

function svgAsComponentPlugin() {
  return {
    name: 'svg-component',
    enforce: 'pre',
    transform(code, id) {
      if (id.endsWith('.svg')) {
        const componentName = id.split('/').pop().replace('.svg', 'Svg');
        return {
          code: `
import React from 'react';
function ${componentName}(props) {
  return ${code.replace(/<svg/, '<svg {...props}')};
}
export default ${componentName};
          `,
          map: null
        };
      }
    },
    // Development: serve transformed files
    configureServer(server) {
      server.middlewares.use((req, res, next) => {
        if (req.url.endsWith('.svg')) {
          res.setHeader('Content-Type', 'application/javascript');
        }
        next();
      });
    }
  };
}

// Custom Vite plugin for build-time code injection
function buildInfoPlugin() {
  return {
    name: 'build-info',
    config(config) {
      console.log(`Building for ${config.mode || 'production'}`);
    },
    buildStart() {
      console.log(`Build started at ${new Date().toISOString()}`);
    },
    closeBundle() {
      console.log(`Build completed at ${new Date().toISOString()}`);
    }
  };
}
```

### Coding Challenges

**Challenge 1: Build a Vite-compatible dev server**
```javascript
// Create a minimal dev server that serves ES modules natively
// like Vite does in development mode.

class MiniViteServer {
  constructor(root = '.') {
    this.root = root;
    this.plugins = [];
    this.transformCache = new Map();
    this.server = null;
  }

  use(plugin) {
    this.plugins.push(plugin);
    return this;
  }

  async start(port = 3000) {
    const http = require('http');
    const fs = require('fs');
    const path = require('path');

    this.server = http.createServer(async (req, res) => {
      const url = new URL(req.url, `http://localhost:${port}`);
      let filePath = path.join(this.root, url.pathname);

      // Handle root
      if (url.pathname === '/') {
        filePath = path.join(this.root, 'index.html');
      }

      // Handle bare module imports (node_modules)
      if (!url.pathname.startsWith('/') || url.pathname.startsWith('/@modules/')) {
        return this.resolveBareModule(url, res);
      }

      try {
        let code = fs.readFileSync(filePath, 'utf-8');

        // Run plugins' transform hooks
        for (const plugin of this.plugins) {
          if (plugin.transform) {
            const result = await plugin.transform(code, filePath);
            if (result) code = result.code || result;
          }
        }

        // Transform bare imports in ESM
        code = this.transformImports(code);

        res.writeHead(200, { 'Content-Type': this.getContentType(filePath) });
        res.end(code);
      } catch (err) {
        res.writeHead(404, { 'Content-Type': 'text/plain' });
        res.end(`Not found: ${filePath}`);
      }
    });

    this.server.listen(port, () => {
      console.log(`Mini Vite server running at http://localhost:${port}`);
    });
  }

  transformImports(code) {
    return code.replace(
      /from\s+['"]([^'"]+)['"]/g,
      (match, specifier) => {
        if (specifier.startsWith('.')) return match;
        return `from '/@modules/${specifier}'`;
      }
    );
  }

  getContentType(filePath) {
    const ext = filePath.split('.').pop();
    const types = {
      js: 'application/javascript',
      ts: 'application/javascript',
      css: 'text/css',
      html: 'text/html',
      json: 'application/json',
      svg: 'image/svg+xml'
    };
    return types[ext] || 'application/octet-stream';
  }

  resolveBareModule(url, res) {
    const specifier = url.pathname.replace('/@modules/', '');
    const path = require.resolve(specifier, { paths: [this.root] });
    const fs = require('fs');
    const code = fs.readFileSync(path, 'utf-8');
    res.writeHead(200, { 'Content-Type': 'application/javascript' });
    res.end(code);
  }

  async stop() {
    if (this.server) {
      this.server.close();
    }
  }
}
```

**Challenge 2: Implement Vite-style HMR for a dev server**
```javascript
// Build a hot module replacement system inspired by Vite's approach.

class ViteStyleHMR {
  constructor(server) {
    this.server = server;
    this.clients = new Set();
    this.hmrBoundary = null;
  }

  injectClient(indexHtml) {
    const hmrClient = `
<script type="module">
const hmr = new HMRClient('ws://localhost:${this.server.address().port}/__hmr');
window.__HMR_CLIENT__ = hmr;
</script>`;

    return indexHtml.replace('</head>', `${hmrClient}\n</head>`);
  }

  setupWebSocket() {
    const WebSocket = require('ws');
    const wss = new WebSocket.Server({ server: this.server });

    wss.on('connection', (ws) => {
      this.clients.add(ws);
      console.log('[HMR] Client connected');

      ws.on('close', () => {
        this.clients.delete(ws);
        console.log('[HMR] Client disconnected');
      });
    });

    this.wss = wss;
  }

  watchFiles(root) {
    const chokidar = require('chokidar');
    const watcher = chokidar.watch(root, {
      ignored: /node_modules|\.git/,
      persistent: true
    });

    watcher.on('change', (filePath) => {
      console.log(`[HMR] File changed: ${filePath}`);
      this.notifyClients({
        type: 'update',
        path: '/' + filePath.replace(/\\/g, '/'),
        timestamp: Date.now()
      });
    });

    return watcher;
  }

  notifyClients(data) {
    const message = JSON.stringify(data);
    for (const client of this.clients) {
      if (client.readyState === 1) {
        client.send(message);
      }
    }
  }

  // Client-side HMR handler (runs in browser)
  static hmrClientScript() {
    return `
class HMRClient {
  constructor(url) {
    this.url = url;
    this.modules = new Map();
    this.connect();
  }

  connect() {
    this.ws = new WebSocket(this.url);
    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleUpdate(data);
    };
    this.ws.onclose = () => {
      setTimeout(() => this.connect(), 3000);
    };
  }

  accept(path, callback) {
    this.modules.set(path, callback);
  }

  handleUpdate(data) {
    if (data.type === 'update') {
      const callback = this.modules.get(data.path);
      if (callback) {
        callback(data.path);
      } else {
        window.location.reload();
      }
    }
  }
}

window.HMRClient = HMRClient;
    `;
  }
}
```

**Challenge 3: Create a Vite plugin for automatic code splitting**
```javascript
// Build a Vite plugin that automatically creates optimal
// code split points based on import analysis.

function autoSplitPlugin(options = {}) {
  const { threshold = 50 * 1024, maxChunks = 10 } = options;

  return {
    name: 'auto-split',
    enforce: 'post',

    // Analyze module sizes
    buildStart() {
      this.moduleSizes = new Map();
    },

    transform(code, id) {
      this.moduleSizes.set(id, Buffer.byteLength(code, 'utf-8'));
      return null;
    },

    // Generate split points
    generateBundle(_, bundle) {
      const largeModules = [];

      for (const [id, size] of this.moduleSizes) {
        if (size > threshold) {
          largeModules.push({ id, size });
        }
      }

      largeModules.sort((a, b) => b.size - a.size);
      const toSplit = largeModules.slice(0, maxChunks);

      if (toSplit.length > 0) {
        console.log(`\n[Auto Split] Found ${toSplit.length} large module(s) to split:`);
        toSplit.forEach(m => {
          console.log(`  ${m.id}: ${(m.size / 1024).toFixed(1)} KB`);
        });

        console.log('\n[Auto Split] Suggested manual chunks:');
        toSplit.forEach(m => {
          const name = m.id.split('/').pop().replace(/\.\w+$/, '');
          console.log(`  '${name}': () => import('${m.id}')`);
        });
      }
    }
  };
}
```

## Parcel

### What It Is

Parcel is a zero-configuration bundler that aims to "just work." It automatically detects file types and applies appropriate transforms without configuration. Parcel supports a wide range of languages (JavaScript, TypeScript, CSS, SCSS, JSX, etc.) out of the box.

### Why It Is Important

Parcel demonstrates that bundling can be simple. Its zero-config approach is ideal for small projects, prototypes, and developers who want minimal setup. Parcel's performance is competitive with Vite for development.

### How It Works Internally

Parcel uses worker threads for parallel processing, making builds fast. It caches compiled assets for subsequent builds. Parcel's scope hoisting enables tree shaking. It automatically handles asset hashing, code splitting, and HMR.

### Syntax

```javascript
// Parcel requires minimal configuration
// Most configuration is done via .browserslistrc and tsconfig.json

// package.json
{
  "scripts": {
    "dev": "parcel src/index.html",
    "build": "parcel build src/index.html"
  }
}

// For custom config, use .parcelrc:
// {
//   "extends": "@parcel/config-default",
//   "transforms": {
//     "*.svg": ["@parcel/transformer-svg"]
//   }
// }

// Browserslist determines transpilation targets
// .browserslistrc
// > 1%
// last 2 versions
// not dead
```

### Beginner Examples

```javascript
// Parcel offers truly zero-config development

// Install: npm install --save-dev parcel

// package.json
{
  "scripts": {
    "dev": "parcel src/index.html",
    "build": "parcel build src/index.html"
  },
  "browserslist": "> 0.5%, last 2 versions"
}

// src/index.html - Just reference your files
// <link rel="stylesheet" href="./styles.css">
// <script type="module" src="./app.js"></script>

// Parcel automatically detects TypeScript, SCSS, JSX, etc.
// No configuration needed for standard setups.
```

### Intermediate Examples

```javascript
// Parcel with custom transformers and named pipelines

// .parcelrc - configure named pipelines
{
  "extends": "@parcel/config-default",
  "transforms": {
    "*.svg": ["@parcel/transformer-svg"],
    "*.{mjs,js}": ["@parcel/transformer-babel"]
  },
  "resolvers": ["@parcel/resolver-glob", "..."]
}

// Use .env files for environment variables
// .env
// API_URL=https://api.example.com
// .env.production
// API_URL=https://api.prod.example.com

// In your code:
// console.log(process.env.API_URL);
// Parcel replaces at build time (not runtime)

// Parcel's content hashing and code splitting
// Dynamic import automatically creates split points
const adminModule = import('./admin.js');

// Parcel handles this automatically:
// - Creates a separate chunk for admin.js
// - Adds content hash to filename
// - Generates correct import paths
```

### Advanced Examples

```javascript
// Parcel plugin development

// Custom Parcel transformer plugin
const { Transformer } = require('@parcel/plugin');

module.exports = new Transformer({
  async transform({ asset }) {
    // Get the source code
    let code = await asset.getCode();

    // Transform the code
    code = code.replace(/process\.env\.APP_VERSION/g, `'${require('./package.json').version}'`);

    // Update asset
    asset.setCode(code);
    asset.addIncludedFile('./package.json');

    return [asset];
  }
});

// Custom Parcel resolver plugin
const { Resolver } = require('@parcel/plugin');

module.exports = new Resolver({
  async resolve({ specifier, options }) {
    // Intercept theme imports
    if (specifier.startsWith('theme:')) {
      const themeName = specifier.replace('theme:', '');
      return {
        filePath: options.projectRoot + `/src/themes/${themeName}.css`
      };
    }
    return null; // Let other resolvers handle it
  }
});

// Parallel builds with Parcel's worker pool
// Parcel automatically uses all available CPU cores
// For large projects, this provides significant speedup
// over single-threaded bundlers like Rollup (without parallel plugin)
```

### Coding Challenges

**Challenge 1: Build a Parcel-compatible zero-config bundler**
```javascript
// Create a minimal zero-config bundler that auto-detects
// file types and applies transforms without configuration.

class ZeroConfigBundler {
  constructor(root) {
    this.root = root;
    this.handlers = new Map();
    this.registerDefaultHandlers();
  }

  registerDefaultHandlers() {
    // JavaScript/TypeScript
    this.handlers.set('.js', this.transformJS.bind(this));
    this.handlers.set('.jsx', this.transformJSX.bind(this));
    this.handlers.set('.ts', this.transformTS.bind(this));

    // Styles
    this.handlers.set('.css', this.transformCSS.bind(this));
    this.handlers.set('.scss', this.transformSCSS.bind(this));

    // Assets
    this.handlers.set('.svg', this.transformAsset.bind(this));
    this.handlers.set('.png', this.transformAsset.bind(this));
    this.handlers.set('.jpg', this.transformAsset.bind(this));
  }

  detectType(filePath) {
    const ext = filePath.split('.').pop();
    if (ext === 'jsx' || ext === 'tsx') return '.jsx';
    if (ext === 'ts') return '.ts';
    if (ext === 'scss' || ext === 'sass') return '.scss';
    return '.' + ext;
  }

  async bundle(entry) {
    const fs = require('fs');
    const path = require('path');
    const entryPath = path.resolve(this.root, entry);
    const queue = [entryPath];
    const processed = new Set();
    const chunks = [];

    while (queue.length > 0) {
      const filePath = queue.shift();
      if (processed.has(filePath)) continue;
      processed.add(filePath);

      let code = fs.readFileSync(filePath, 'utf-8');
      const type = this.detectType(filePath);
      const handler = this.handlers.get(type);

      if (handler) {
        code = await handler(code, filePath);
      }

      chunks.push({ filePath, code, type });

      // Find dependencies via imports
      const deps = code.match(/(?:import\s+(?:.*?\s+from\s+)?['"])([^'"]+)/g);
      if (deps) {
        for (const dep of deps) {
          const specifier = dep.replace(/(?:import\s+(?:.*?\s+from\s+)?['"])/, '');
          if (specifier.startsWith('.')) {
            const depPath = path.resolve(path.dirname(filePath), specifier);
            const extVariants = ['', '.js', '.jsx', '.ts', '.tsx', '/index.js', '/index.ts'];
            for (const ext of extVariants) {
              const candidate = depPath + ext;
              if (fs.existsSync(candidate)) {
                queue.push(candidate);
                break;
              }
            }
          }
        }
      }
    }

    return this.assemble(chunks);
  }

  transformJS(code) { return code; }

  transformJSX(code) {
    const babel = require('@babel/core');
    const result = babel.transformSync(code, {
      presets: ['@babel/preset-react'],
      plugins: ['@babel/plugin-transform-modules-commonjs']
    });
    return result.code;
  }

  transformTS(code) {
    const ts = require('typescript');
    const result = ts.transpileModule(code, {
      compilerOptions: { module: ts.ModuleKind.ESNext, target: ts.ScriptTarget.ES2020 }
    });
    return result.outputText;
  }

  transformCSS(code) {
    return `(function() { const s = document.createElement('style'); s.textContent = ${JSON.stringify(code)}; document.head.appendChild(s); })();`;
  }

  transformSCSS(code, filePath) {
    const sass = require('sass');
    const result = sass.compile(filePath);
    return this.transformCSS(result.css);
  }

  transformAsset(code, filePath) {
    const fs = require('fs');
    const data = fs.readFileSync(filePath);
    const base64 = data.toString('base64');
    const ext = filePath.split('.').pop();
    const mimeTypes = { svg: 'image/svg+xml', png: 'image/png', jpg: 'image/jpeg' };
    return `export default 'data:${mimeTypes[ext] || 'application/octet-stream'};base64,${base64}'`;
  }

  assemble(chunks) {
    let bundle = '';
    const moduleMap = new Map();
    let id = 0;

    for (const chunk of chunks) {
      moduleMap.set(chunk.filePath, id++);
    }

    for (const chunk of chunks) {
      const modId = moduleMap.get(chunk.filePath);
      if (chunk.type === '.css') {
        bundle += chunk.code + '\n';
      } else {
        let code = chunk.code;
        code = code.replace(/import\s+(.*?)\s+from\s+['"]([^'"]+)['"]/g, (_, imports, specifier) => {
          if (specifier.startsWith('.')) {
            const depId = moduleMap.get(require('path').resolve(require('path').dirname(chunk.filePath), specifier));
            if (depId !== undefined) {
              return `const ${imports} = __modules[${depId}]`;
            }
          }
          return `// external: ${specifier}`;
        });
        bundle += `__modules[${modId}] = function() {\n${code}\n};\n`;
      }
    }

    return `const __modules = {};\n${bundle}\n__modules[0]();`;
  }
}
```

**Challenge 2: Implement Parcel's automatic cache invalidation**
```javascript
// Build a caching system that automatically invalidates
// cached builds when files or dependencies change.

class ParcelCache {
  constructor(cacheDir = '.parcel-cache') {
    this.cacheDir = cacheDir;
    this.cache = new Map();
    this.fileHashes = new Map();
  }

  async get(key) {
    return this.cache.get(key) || null;
  }

  async set(key, value, deps = []) {
    const hash = await this.computeHash(value);

    // Store dependency tracking
    const depHashes = {};
    for (const dep of deps) {
      depHashes[dep] = await this.getFileHash(dep);
    }

    this.cache.set(key, { value, hash, depHashes, timestamp: Date.now() });
    return true;
  }

  async isStale(key) {
    const entry = this.cache.get(key);
    if (!entry) return true;

    // Check if any dependencies changed
    for (const [dep, oldHash] of Object.entries(entry.depHashes)) {
      const currentHash = await this.getFileHash(dep);
      if (currentHash !== oldHash) return true;
    }

    return false;
  }

  async getFileHash(filePath) {
    if (this.fileHashes.has(filePath)) {
      return this.fileHashes.get(filePath);
    }

    const fs = require('fs');
    const crypto = require('crypto');
    const content = fs.readFileSync(filePath);
    const hash = crypto.createHash('md5').update(content).digest('hex');
    this.fileHashes.set(filePath, hash);
    return hash;
  }

  async computeHash(data) {
    const crypto = require('crypto');
    return crypto.createHash('md5').update(JSON.stringify(data)).digest('hex');
  }

  invalidate(patterns) {
    for (const key of this.cache.keys()) {
      if (patterns.some(p => key.includes(p))) {
        this.cache.delete(key);
      }
    }
  }

  getStats() {
    return {
      entries: this.cache.size,
      stale: [...this.cache.keys()].filter(k => this.isStale(k)).length,
      fresh: [...this.cache.keys()].filter(k => !this.isStale(k)).length
    };
  }

  clear() {
    this.cache.clear();
    this.fileHashes.clear();
  }
}
```

**Challenge 3: Create a build time comparison tool**
```javascript
// Build a benchmark that compares build times across bundlers
// for different project sizes and configurations.

class BuildTimeComparator {
  constructor() {
    this.results = new Map();
    this.bundlers = {};
  }

  register(name, buildFn) {
    this.bundlers[name] = buildFn;
  }

  async benchmark(name, projectSize, buildFn) {
    // Warmup
    for (let i = 0; i < 3; i++) {
      buildFn(projectSize);
    }

    const times = [];
    for (let i = 0; i < 5; i++) {
      const start = Date.now();
      buildFn(projectSize);
      times.push(Date.now() - start);
    }

    const avg = times.reduce((a, b) => a + b, 0) / times.length;
    const min = Math.min(...times);
    const max = Math.max(...times);

    return { avg, min, max, samples: times };
  }

  async runAll() {
    const sizes = [100, 1000, 10000];

    console.log('Build Time Comparison:\n');
    console.log('Bundler'.padEnd(15), '100 modules'.padEnd(20), '1000 modules'.padEnd(20), '10000 modules'.padEnd(20));
    console.log('-'.repeat(75));

    for (const [name, buildFn] of Object.entries(this.bundlers)) {
      const row = [name.padEnd(15)];

      for (const size of sizes) {
        const result = await this.benchmark(name, size, buildFn);
        const label = `${result.avg.toFixed(0)}ms (${(1000 / result.avg).toFixed(1)} mod/s)`;
        row.push(label.padEnd(20));
        this.results.set(`${name}-${size}`, result);
      }

      console.log(row.join(''));
    }

    return this.results;
  }
}
```

## Bundling Strategies and Comparisons

### What It Is

Different bundlers excel at different use cases. Choosing the right bundler depends on project type (application vs library), team needs, performance requirements, and ecosystem compatibility. This section compares bundlers and provides strategies for different scenarios.

### Why It Is Important

The wrong bundler choice can lead to slow development cycles, difficult debugging, and unnecessarily large bundles. Understanding the tradeoffs helps teams make informed architectural decisions.

### Comparison

| Feature | Webpack | Rollup | Vite | Parcel |
|---------|---------|--------|------|--------|
| **Dev Server** | slow (full bundle) | limited | fast (native ESM) | fast |
| **HMR** | good | limited | excellent | good |
| **Tree Shaking** | good | excellent (pioneer) | excellent (via Rollup) | good |
| **Code Splitting** | excellent | good | excellent | good |
| **Config** | complex | moderate | minimal | minimal |
| **Plugin Ecosystem** | largest | large | growing | moderate |
| **Library Bundling** | okay | best (default choice) | okay (via Rollup) | limited |
| **CSS/Assets** | excellent | moderate | excellent | good |
| **WASM Support** | good | limited | good | good |
| **Startup Time** | slow | fast | instant | fast (cache) |

### Syntax

```javascript
// Strategy 1: Library author → Use Rollup
// rollup.config.js for a React component library
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';
import terser from '@rollup/plugin-terser';
import peerDepsExternal from 'rollup-plugin-peer-deps-external';

export default {
  input: 'src/index.js',
  output: [
    { file: 'dist/index.cjs.js', format: 'cjs', exports: 'named' },
    { file: 'dist/index.esm.js', format: 'esm' },
    { file: 'dist/index.umd.js', format: 'umd', name: 'MyLib' }
  ],
  plugins: [
    peerDepsExternal(),
    resolve(),
    commonjs(),
    babel({ presets: ['@babel/preset-env', '@babel/preset-react'] }),
    terser()
  ]
};

// Strategy 2: Large application → Use webpack or Vite
// webpack with optimized code splitting
module.exports = {
  entry: { main: './src/index.js' },
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          priority: 10
        },
        common: {
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true
        }
      }
    },
    runtimeChunk: 'single'
  }
};

// Strategy 3: Rapid prototyping → Use Vite or Parcel
// Vite with minimal config
import { defineConfig } from 'vite';
export default defineConfig({
  plugins: [], // Just need framework plugin
  server: { port: 3000 }
});
```

### Beginner Examples

```javascript
// When to use which bundler

// Use Vite for:
// - New projects (React, Vue, Svelte)
// - Fast development experience
// - Modern browsers target
// - Production optimization is important

// Use Webpack for:
// - Large legacy projects
// - Complex configuration needs
// - Extensive custom build steps
// - Enterprise applications requiring fine control

// Use Rollup for:
// - Publishing npm packages/libraries
// - When output size matters most
// - Tree-shakeable library distribution
// - SDK or widget development

// Use Parcel for:
// - Quick prototypes
// - Small projects with standard setups
// - Developers who want minimal config
```

### Intermediate Examples

```javascript
// Strategy: Hybrid bundling for large applications
// Use multiple bundlers for different parts of the app

// 1. Vite for development (fast HMR)
// 2. Rollup for library packages (clean ESM output)
// 3. Webpack for specific legacy integrations (if needed)

// Monorepo strategy with Turborepo
// turbo.json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"],
      "inputs": ["src/**"]
    },
    "test": {
      "dependsOn": ["build"]
    }
  }
}

// Each package chooses its bundler:
// packages/ui/     -> Rollup (library)
// packages/web/    -> Vite (app)
// packages/admin/  -> Vite (app)
// packages/legacy/ -> Webpack (enterprise integration)

// Shared config for consistent output
// packages/shared/bundler-base.js
export const sharedBuildConfig = {
  sourcemap: true,
  minify: process.env.NODE_ENV === 'production',
  target: ['es2020', 'chrome60', 'firefox60', 'safari12', 'edge18']
};
```

### Advanced Examples

```javascript
// Strategy: Build-time vs runtime optimization tradeoffs

// 1. Build-time optimization (slower build, faster runtime)
// - Tree shaking removes dead code
// - Scope hoisting reduces module overhead
// - Minification reduces parse time
// - Code splitting reduces initial load

// 2. Runtime optimization (faster build, runtime cost)
// - Lazy loading with dynamic imports
// - Runtime code generation (eval - avoid this)
// - Just-in-time compilation (V8 handles this)
// - Service worker caching

// Benchmark different strategies:
function benchmarkBuildStrategy(strategy) {
  const metrics = {
    buildTime: 0,
    bundleSize: 0,
    parseTime: 0,
    executionTime: 0,
    cacheHitRate: 0
  };

  const strategies = {
    aggressive: {
      buildTime: 120,     // seconds
      bundleSize: 150,     // KB
      parseTime: 50,       // ms
      executionTime: 100,  // ms
      cacheHitRate: 0.9
    },
    balanced: {
      buildTime: 45,
      bundleSize: 220,
      parseTime: 80,
      executionTime: 130,
      cacheHitRate: 0.7
    },
    fast: {
      buildTime: 15,
      bundleSize: 350,
      parseTime: 120,
      executionTime: 160,
      cacheHitRate: 0.5
    }
  };

  return strategies[strategy] || strategies.balanced;
}

// Analyze first-load vs cached-load performance
function analyzeLoadingStrategy(strategy) {
  const s = benchmarkBuildStrategy(strategy);
  const firstLoad = s.parseTime + s.executionTime;
  const cachedLoad = s.executionTime; // Service worker caches parsed code
  const savings = ((firstLoad - cachedLoad) / firstLoad * 100).toFixed(0);

  console.log(`${strategy.toUpperCase()} Strategy:`);
  console.log(`  Build time: ${s.buildTime}s`);
  console.log(`  Bundle: ${s.bundleSize}KB`);
  console.log(`  First load: ${firstLoad}ms`);
  console.log(`  Cached load: ${cachedLoad}ms`);
  console.log(`  Cache savings: ${savings}%`);
  console.log();
}
```

### Coding Challenges

**Challenge 1: Build a bundler decision tree**
```javascript
// Create a decision-making tool that recommends the best bundler
// based on project requirements.

class BundlerDecisionTree {
  constructor() {
    this.rules = [
      {
        condition: (p) => p.type === 'library' && p.distribution === 'npm',
        recommendation: 'Rollup',
        reason: 'Best output quality for npm packages, superior tree shaking, clean ESM/CJS/UMD output.',
        alternatives: ['esbuild', 'tsup']
      },
      {
        condition: (p) => p.type === 'application' && p.priority === 'developer-experience',
        recommendation: 'Vite',
        reason: 'Fastest dev server startup and HMR, native ESM in development, Rollup-powered production builds.',
        alternatives: ['Parcel', 'Webpack']
      },
      {
        condition: (p) => p.type === 'application' && p.legacyIntegration === true,
        recommendation: 'Webpack',
        reason: 'Largest plugin ecosystem, handles complex configuration, best for enterprise with custom build needs.',
        alternatives: ['Vite (with legacy plugin)']
      },
      {
        condition: (p) => p.type === 'application' && p.simplicity === 'critical',
        recommendation: 'Parcel',
        reason: 'True zero-config, auto-detects file types, good for small projects and prototypes.',
        alternatives: ['Vite', 'esbuild']
      },
      {
        condition: (p) => p.type === 'component-library',
        recommendation: 'Rollup + Storybook',
        reason: 'Rollup for clean library output, Storybook for component development and documentation.',
        alternatives: ['Vite + Histoire', 'Webpack + Storybook']
      },
      {
        condition: (p) => p.platform === 'node',
        recommendation: 'esbuild / tsup',
        reason: 'Fastest builds for Node.js packages, excellent TypeScript support, minimal configuration.',
        alternatives: ['tsc', 'Rollup']
      }
    ];
  }

  recommend(project) {
    console.log('Bundler Recommendation:\n');

    for (const rule of this.rules) {
      if (rule.condition(project)) {
        console.log(`Recommended: ${rule.recommendation}`);
        console.log(`Why: ${rule.reason}`);
        if (rule.alternatives.length > 0) {
          console.log(`Alternatives: ${rule.alternatives.join(', ')}`);
        }
        console.log();

        // Show confidence based on how many rules match
        const matchingRules = this.rules.filter(r => r.condition(project));
        const confidence = Math.min(matchingRules.length / this.rules.length * 100, 100);
        console.log(`Confidence: ${Math.round(confidence)}%`);
        return { recommendation: rule.recommendation, confidence };
      }
    }

    // Fallback
    console.log('Recommendation: Vite');
    console.log('Why: Best overall choice for most projects.');
    console.log('Confidence: 50% (project requirements unclear)');
    return { recommendation: 'Vite', confidence: 50 };
  }
}
```

**Challenge 2: Implement a monorepo build orchestrator**
```javascript
// Build a tool that manages builds across multiple packages,
// caching results and rebuilding only changed packages.

class MonorepoBuildOrchestrator {
  constructor() {
    this.packages = new Map();
    this.dependencyGraph = new Map();
    this.buildCache = new Map();
  }

  addPackage(name, config) {
    this.packages.set(name, {
      ...config,
      lastBuild: null,
      hash: null
    });
    this.dependencyGraph.set(name, config.dependencies || []);
  }

  setDependencies(pkgName, deps) {
    this.dependencyGraph.set(pkgName, deps);
  }

  getBuildOrder() {
    const visited = new Set();
    const order = [];

    const dfs = (pkg) => {
      if (visited.has(pkg)) return;
      visited.add(pkg);

      const deps = this.dependencyGraph.get(pkg) || [];
      for (const dep of deps) {
        if (this.packages.has(dep)) {
          dfs(dep);
        }
      }

      order.push(pkg);
    };

    for (const pkg of this.packages.keys()) {
      dfs(pkg);
    }

    return order;
  }

  computeHash(pkgName) {
    const pkg = this.packages.get(pkgName);
    if (!pkg) return '';

    const crypto = require('crypto');
    const hashInput = JSON.stringify({
      src: pkg.sourceHash,
      config: pkg.buildConfig,
      deps: (this.dependencyGraph.get(pkgName) || [])
        .map(d => this.packages.get(d)?.hash || '')
        .join(',')
    });

    return crypto.createHash('md5').update(hashInput).digest('hex');
  }

  needsRebuild(pkgName) {
    const pkg = this.packages.get(pkgName);
    if (!pkg) return false;

    const newHash = this.computeHash(pkgName);
    const oldHash = this.buildCache.get(pkgName);

    return newHash !== oldHash;
  }

  build(force = false) {
    const order = this.getBuildOrder();
    const results = [];

    console.log('Build Order:', order.join(' -> '), '\n');

    for (const pkgName of order) {
      const needsBuild = force || this.needsRebuild(pkgName);

      if (!needsBuild) {
        results.push({ package: pkgName, status: 'cached', time: 0 });
        console.log(`[CACHED] ${pkgName}`);
        continue;
      }

      const start = Date.now();
      console.log(`[BUILD] ${pkgName}...`);

      // Simulate build
      const buildTime = Math.floor(Math.random() * 3000) + 500;

      const pkg = this.packages.get(pkgName);
      const hash = this.computeHash(pkgName);
      this.buildCache.set(pkgName, hash);

      results.push({
        package: pkgName,
        status: 'built',
        time: buildTime,
        hash: hash.slice(0, 8)
      });

      console.log(`[DONE] ${pkgName} (${buildTime}ms, hash: ${hash.slice(0, 8)})`);
    }

    const totalTime = results.reduce((s, r) => s + r.time, 0);
    const builtCount = results.filter(r => r.status === 'built').length;
    const cachedCount = results.filter(r => r.status === 'cached').length;

    console.log(`\nSummary: ${builtCount} built, ${cachedCount} cached (${totalTime}ms total)`);
    return results;
  }

  invalidate(pkgName) {
    this.buildCache.delete(pkgName);

    // Invalidate downstream dependents
    for (const [pkg, deps] of this.dependencyGraph) {
      if (deps.includes(pkgName)) {
        this.invalidate(pkg);
      }
    }
  }
}
```

**Challenge 3: Create a bundle optimization suggestion engine**
```javascript
// Build an AI-like tool that analyzes a bundle and suggests
// specific optimizations based on patterns.

class BundleOptimizer {
  constructor() {
    this.optimizations = [];
    this.libraryAlternatives = {
      'moment': {
        alternatives: ['date-fns', 'dayjs', 'luxon'],
        sizeReduction: '50-95%',
        reason: 'moment.js imports entire locale bundle, alternatives are tree-shakeable'
      },
      'lodash': {
        alternatives: ['lodash-es', 'lodash/* direct imports'],
        sizeReduction: '70-90%',
        reason: 'Import only needed functions instead of entire library'
      },
      'jquery': {
        alternatives: ['vanilla JS', 'Alpine.js'],
        sizeReduction: '100%',
        reason: 'Modern browsers have native DOM APIs that replace jQuery'
      },
      'axios': {
        alternatives: ['native fetch', 'ky', 'got'],
        sizeReduction: '50-80%',
        reason: 'Native fetch is built into browsers, no library needed'
      }
    };
  }

  analyzeBundle(bundle) {
    this.optimizations = [];

    // Check for oversized libraries
    if (bundle.modules) {
      this.findLargeLibraries(bundle);
    }

    // Check for missing code splitting
    this.checkCodeSplitting(bundle);

    // Check for duplicate dependencies
    this.checkDuplicates(bundle);

    // Check for unused polyfills
    this.checkPolyfills(bundle);

    return this.optimizations;
  }

  findLargeLibraries(bundle) {
    for (const mod of bundle.modules || []) {
      for (const [lib, info] of Object.entries(this.libraryAlternatives)) {
        if (mod.name.includes(lib) && mod.size > 5000) {
          this.addSuggestion({
            type: 'library-replacement',
            severity: 'high',
            current: lib,
            alternatives: info.alternatives,
            estimatedSaving: info.sizeReduction,
            reason: info.reason,
            currentSize: mod.size
          });
        }
      }

      // Flag large individual modules
      if (mod.size > 100000) { // > 100KB
        this.addSuggestion({
          type: 'large-module',
          severity: 'medium',
          module: mod.name,
          size: mod.size,
          suggestion: 'Consider lazy loading this module with dynamic import()',
          estimatedSaving: `${mod.size}B on initial load`
        });
      }
    }
  }

  checkCodeSplitting(bundle) {
    if (bundle.totalSize > 500000 && !bundle.hasCodeSplitting) {
      this.addSuggestion({
        type: 'code-splitting',
        severity: 'high',
        suggestion: 'Enable code splitting with React.lazy() or dynamic import()',
        estimatedSaving: '30-70% initial load size reduction',
        reason: `Bundle is ${(bundle.totalSize / 1024).toFixed(0)}KB, should be under 200KB for optimal loading`
      });
    }
  }

  checkDuplicates(bundle) {
    if (bundle.duplicates && bundle.duplicates.length > 0) {
      for (const dup of bundle.duplicates) {
        this.addSuggestion({
          type: 'duplicate-dependency',
          severity: 'high',
          dependency: dup.name,
          versions: dup.versions,
          suggestion: 'Deduplicate with yarn-deduplicate or npm dedupe',
          estimatedSaving: `${dup.extraSize}B`
        });
      }
    }
  }

  checkPolyfills(bundle) {
    const polyfillPatterns = ['core-js', 'regenerator-runtime', 'babel-polyfill'];
    for (const mod of bundle.modules || []) {
      if (polyfillPatterns.some(p => mod.name.includes(p))) {
        this.addSuggestion({
          type: 'polyfill-audit',
          severity: 'low',
          module: mod.name,
          suggestion: 'Audit target browser list to include only necessary polyfills. Use @babel/preset-env useBuiltIns.',
          estimatedSaving: '10-40%'
        });
      }
    }
  }

  addSuggestion(suggestion) {
    this.optimizations.push(suggestion);
  }

  generateReport(bundle) {
    this.analyzeBundle(bundle);
    console.log('Bundle Optimization Report:\n');

    console.log(`Bundle: ${(bundle.totalSize / 1024).toFixed(1)} KB (${bundle.modules?.length || 0} modules)\n`);

    if (this.optimizations.length === 0) {
      console.log('No optimizations found. Bundle is well-configured!');
      return;
    }

    const bySeverity = { high: [], medium: [], low: [] };
    this.optimizations.forEach(o => bySeverity[o.severity]?.push(o));

    for (const [severity, items] of Object.entries(bySeverity)) {
      if (items.length === 0) continue;
      console.log(`[${severity.toUpperCase()}] ${items.length} suggestion(s):\n`);
      items.forEach(item => {
        if (item.type === 'library-replacement') {
          console.log(`  Replace ${item.current} with ${item.alternatives.join(' / ')}`);
          console.log(`  Reason: ${item.reason}`);
          console.log(`  Savings: ~${item.estimatedSaving}\n`);
        } else {
          console.log(`  ${item.suggestion}`);
          console.log(`  Savings: ${item.estimatedSaving}\n`);
        }
      });
    }

    const totalSavings = this.optimizations
      .filter(o => o.currentSize)
      .reduce((s, o) => s + o.currentSize, 0);
    if (totalSavings > 0) {
      console.log(`Estimated total savings: ${(totalSavings / 1024).toFixed(1)} KB`);
    }
  }
}
```

### Real-World Use Cases

- **Webpack**: Large applications with complex build pipelines (legacy enterprise apps)
- **Rollup**: npm package publishing (React, Vue, D3, Three.js)
- **Vite**: Modern web applications (recommended by Vue, Svelte, Astro)
- **Parcel**: Small to medium static sites, prototypes
- **esbuild**: Build toolchain component (used by Vite, Snowpack, Remix)

### Common Mistakes

- Over-configuring webpack (too many plugins, unnecessary complexity)
- Not using webpack's `optimization.splitChunks` for large apps
- Publishing CommonJS-only npm packages (not tree-shakeable)
- Not configuring `sideEffects` for tree shaking
- Using development mode for production builds (no minification)
- Not leveraging code splitting for large applications
- Ignoring bundle analysis tools

### Best Practices

- Use Vite for new applications (fast dev, good production builds)
- Use Rollup for library publishing
- Use Webpack for complex enterprise applications needing deep customization
- Always analyze bundles with `webpack-bundle-analyzer` or `vite-bundle-visualizer`
- Configure proper code splitting for large applications
- Set up tree shaking with `sideEffects: false`
- Use content hashing for long-term caching
- Minimize plugin usage to reduce build time complexity
- Keep build tooling up to date for performance improvements

### Performance Considerations

- Build time: Vite < esbuild < Rollup < Parcel < Webpack
- Development HMR: Vite ≥ Parcel > Webpack (with cache) > Rollup
- Production bundle size: Rollup ≥ Webpack > Vite > Parcel
- Memory usage: Webpack (highest) → Vite → Rollup → esbuild (lowest)
- Cache effectiveness: Vite (excellent) → Webpack (good) → Parcel (good) → Rollup (basic)

### Interview Questions

**Q: What are the differences between Webpack and Vite?** A: Webpack bundles the entire application during development, causing slow startup. Vite serves native ES modules during development, enabling instant startup and fast HMR. For production, Vite uses Rollup under the hood. Webpack has a larger plugin ecosystem and more configuration options. Vite is simpler to configure and faster for development.

**Q: Why would you use Rollup instead of Webpack for a library?** A: Rollup produces cleaner, more optimized output for libraries. It has better tree shaking (physically removes unused code) and produces smaller bundles. Rollup's output formats (ESM, CJS, UMD) are cleaner and more standards-compliant. Rollup doesn't add runtime overhead. Most major libraries (React, Vue, D3) use Rollup for distribution.

### Related Topics

- Module formats (ESM, CJS, AMD, UMD, IIFE)
- Code splitting and lazy loading
- Tree shaking and dead code elimination
- Hot Module Replacement (HMR)
- Build tool performance optimization
- Package distribution and npm publishing
- Monorepo build strategies (Turborepo, Nx)
- ESBuild for build tooling
- SWC and Rust-based build tools
