# Code Splitting - Dynamic imports, React.lazy, route-based splitting, bundle analysis

## Introduction

Code splitting is a technique that splits your JavaScript bundle into smaller chunks that can be loaded on demand. Instead of downloading the entire application at once, users download only what they need for the current page or interaction.

## Dynamic import() Syntax

### What It Is

Dynamic import() is a function-like expression that loads a module asynchronously. It returns a Promise resolving to the module namespace object. Webpack, Vite, and other bundlers automatically create separate chunks for dynamically imported modules.

### Why It Is Important

Dynamic imports are the foundational mechanism for all code splitting. Without them, the entire application bundle must be downloaded before any code can execute. Dynamic imports enable lazy loading, route-based splitting, conditional polyfill loading, and on-demand feature loading. They directly impact initial load time, time-to-interactive, and bandwidth usage.

### How It Works Internally

The bundler analyzes the import() call and creates a separate chunk. At runtime, the browser fetches the chunk as a script when import() is called. The chunk is cached after first load.

### Syntax

```javascript
// Static import (always bundled together)
import { utils } from './utils.js';

// Dynamic import (creates separate chunk)
const module = await import('./heavy.js');

// Dynamic with default export
const { default: Chart } = await import('./chart.js');

// Dynamic with destructuring
const { render, update } = await import('./dashboard.js');

// Template literal (for variable paths)
const lang = navigator.language;
const messages = await import('./locales/' + lang + '.js');
```

### Beginner Examples

```javascript
// Lazy loading a heavy chart library
async function showChart() {
  const chartContainer = document.getElementById('chart');
  chartContainer.innerHTML = '<p>Loading chart...</p>';

  try {
    const { default: Chart } = await import('./heavy-chart-library.js');
    const chart = new Chart(chartContainer, {
      type: 'bar',
      data: getData()
    });
  } catch (err) {
    chartContainer.innerHTML = '<p>Failed to load chart</p>';
    console.error(err);
  }
}

document.getElementById('show-chart-btn').addEventListener('click', showChart);

// Lazy loading a polyfill
async function ensurePolyfill() {
  if (!window.IntersectionObserver) {
    await import('./intersection-observer-polyfill.js');
  }
  // Now safe to use IntersectionObserver
  new IntersectionObserver(/* ... */);
}
```

### Intermediate Examples

```javascript
// Preloading (fetch when idle, not on interaction)
const preloadedChunks = new Set();

function preloadChunk(name, importFn) {
  if (!preloadedChunks.has(name)) {
    preloadedChunks.add(name);
    // Use requestIdleCallback or setTimeout to defer
    requestIdleCallback(() => importFn());
  }
}

// Preload chart when page is idle
preloadChunk('chart', () => import('./chart.js'));

// Later usage - already cached
button.onclick = async () => {
  const { renderChart } = await import('./chart.js'); // Instant from cache
  renderChart();
};

// Module aggregation with dynamic imports
class LazyService {
  constructor() {
    this.modules = new Map();
  }

  async load(name) {
    if (this.modules.has(name)) {
      return this.modules.get(name);
    }

    const loaders = {
      'analytics': () => import('./services/analytics.js'),
      'auth': () => import('./services/auth.js'),
      'storage': () => import('./services/storage.js'),
      'notifications': () => import('./services/notifications.js')
    };

    const loader = loaders[name];
    if (!loader) throw new Error('Unknown service: ' + name);

    const module = await loader();
    this.modules.set(name, module);
    return module;
  }
}

// Usage
const services = new LazyService();
const auth = await services.load('auth');
auth.login(credentials);
```

### Advanced Examples

```javascript
// Webpack magic comments for chunk naming and prefetching
const Chart = () => import(/* webpackChunkName: "chart" */ /* webpackPrefetch: true */ './Chart');

// Dynamic import with retry
async function importWithRetry(importFn, retries = 3) {
  for (let i = 0; i <= retries; i++) {
    try {
      return await importFn();
    } catch (err) {
      if (i === retries) throw err;
      console.warn('Import failed, retrying...', err);
      await new Promise(r => setTimeout(r, 1000 * (i + 1)));
    }
  }
}

const Table = React.lazy(() => importWithRetry(() => import('./Table')));

// Conditional polyfill loading
async function loadPolyfills() {
  const polyfills = [];

  if (!('fetch' in window)) {
    polyfills.push(import('whatwg-fetch'));
  }
  if (!('IntersectionObserver' in window)) {
    polyfills.push(import('intersection-observer'));
  }
  if (!('Object.fromEntries' in Object)) {
    polyfills.push(import('core-js/stable/object/from-entries'));
  }

  if (polyfills.length > 0) {
    await Promise.all(polyfills);
    console.log('Polyfills loaded');
  }
}

// Worker chunk (separate thread)
const worker = new Worker(new URL('./heavy-worker.js', import.meta.url));
```

### Coding Challenges

**Challenge 1: Build a lazy module loader with dependency resolution**
```javascript
// Create a module loader that resolves dependencies between dynamic chunks.
// If module A depends on module B, ensure B is loaded before A.
// Support parallel loading of independent modules.

class DependencyLoader {
  constructor() {
    this.loaded = new Map();
    this.loading = new Map();
  }

  define(name, deps, factory) {
    this.deps = this.deps || new Map();
    this.deps.set(name, { deps, factory });
  }

  async load(name) {
    if (this.loaded.has(name)) return this.loaded.get(name);
    if (this.loading.has(name)) return this.loading.get(name);

    const promise = (async () => {
      const definition = this.deps.get(name);
      if (!definition) throw new Error(`Module ${name} not defined`);

      const depModules = await Promise.all(
        definition.deps.map(dep => this.load(dep))
      );

      const result = definition.factory(...depModules);
      this.loaded.set(name, result);
      return result;
    })();

    this.loading.set(name, promise);
    return promise;
  }

  async loadDynamic(importFn, name) {
    if (this.loaded.has(name)) return this.loaded.get(name);
    const module = await importFn();
    this.loaded.set(name, module);
    return module;
  }
}
```

**Challenge 2: Implement predictive preloading based on user behavior**
```javascript
// Build a system that predicts which chunks the user will need next
// and preloads them in the background. Track mouse movement and hover.

class PredictivePreloader {
  constructor() {
    this.chunks = new Map();
    this.hoverTimers = new Map();
    this.loaded = new Set();
  }

  register(selector, importFn, chunkName) {
    this.chunks.set(selector, { importFn, chunkName });
    const elements = document.querySelectorAll(selector);
    elements.forEach(el => {
      el.addEventListener('mouseenter', () => this.onHoverStart(el, selector));
      el.addEventListener('mouseleave', () => this.onHoverEnd(el, selector));
    });
  }

  onHoverStart(element, selector) {
    const chunk = this.chunks.get(selector);
    if (!chunk || this.loaded.has(chunk.chunkName)) return;

    this.hoverTimers.set(element, setTimeout(() => {
      chunk.importFn().then(() => {
        this.loaded.add(chunk.chunkName);
        element.dataset.preloaded = 'true';
      }).catch(() => {});
    }, 200));
  }

  onHoverEnd(element) {
    const timer = this.hoverTimers.get(element);
    if (timer) {
      clearTimeout(timer);
      this.hoverTimers.delete(element);
    }
  }

  destroy() {
    this.hoverTimers.forEach(t => clearTimeout(t));
    this.hoverTimers.clear();
    this.chunks.clear();
  }
}
```

**Challenge 3: Create a dynamic import manager with progress tracking**
```javascript
// Build a manager that tracks the loading progress of dynamic imports,
// supports prioritization, and provides progress callbacks.

class ImportManager {
  constructor() {
    this.queue = [];
    this.loading = new Set();
    this.cache = new Map();
    this.maxConcurrent = 3;
  }

  enqueue(name, importFn, priority = 0) {
    return new Promise((resolve, reject) => {
      this.queue.push({ name, importFn, priority, resolve, reject });
      this.queue.sort((a, b) => b.priority - a.priority);
      this.processQueue();
    });
  }

  async processQueue() {
    if (this.loading.size >= this.maxConcurrent) return;

    while (this.queue.length > 0 && this.loading.size < this.maxConcurrent) {
      const item = this.queue.shift();
      this.loading.add(item.name);
      this.execute(item).finally(() => {
        this.loading.delete(item.name);
        this.processQueue();
      });
    }
  }

  async execute(item) {
    if (this.cache.has(item.name)) {
      item.resolve(this.cache.get(item.name));
      return;
    }
    try {
      const total = this.queue.length + this.loading.size;
      const completed = this.cache.size;
      const module = await item.importFn();
      this.cache.set(item.name, module);
      item.resolve(module);
    } catch (err) {
      item.reject(err);
    }
  }

  getProgress() {
    const total = this.queue.length + this.loading.size + this.cache.size;
    return {
      loaded: this.cache.size,
      loading: this.loading.size,
      queued: this.queue.length,
      total,
      percent: total > 0 ? Math.round((this.cache.size / total) * 100) : 100
    };
  }

  clearCache() { this.cache.clear(); }
}
```

## React.lazy and Suspense

### What It Is

React.lazy enables dynamic imports for React components. Combined with Suspense, it shows a fallback UI while the component chunk loads.

### Why It Is Important

React.lazy integrates code splitting directly into React's component model, making it straightforward to split applications at the component level. Without React.lazy, developers would need to manage loading states manually for every lazy-loaded component. Suspense provides a declarative way to show fallback UIs, handle multiple lazy components, and coordinate loading states across the component tree.

### Syntax

```jsx
import React, { Suspense } from 'react';

const HeavyComponent = React.lazy(() => import('./HeavyComponent'));
const Dashboard = React.lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
```

### Examples

```jsx
// Route-based splitting with React Router
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import React, { Suspense } from 'react';

const Home = React.lazy(() => import('./pages/Home'));
const About = React.lazy(() => import('./pages/About'));
const Dashboard = React.lazy(() => import('./pages/Dashboard'));
const Settings = React.lazy(() => import('./pages/Settings'));

function Loading() {
  return <div className="page-loader"><Spinner /></div>;
}

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<Loading />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}

// Named exports with React.lazy
const LazyComponent = React.lazy(() =>
  import('./components').then(module => ({
    default: module.NamedComponent
  }))
);

// Error boundary for lazy loading failures
class LazyErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() { return { hasError: true }; }

  render() {
    if (this.state.hasError) {
      return <div>Failed to load component. <button onClick={() => window.location.reload()}>Retry</button></div>;
    }
    return this.props.children;
  }
}

// Usage
<LazyErrorBoundary>
  <Suspense fallback={<Spinner />}>
    <Dashboard />
  </Suspense>
</LazyErrorBoundary>
```

### Coding Challenges

**Challenge 1: Build a lazy-loading image gallery with Suspense**
```jsx
// Create a gallery that uses React.lazy and Suspense to load
// image viewer components on demand. Show skeleton loaders.

import React, { Suspense, useState } from 'react';

const Lightbox = React.lazy(() => import('./Lightbox'));
const ImageEditor = React.lazy(() => import('./ImageEditor'));

function Gallery({ images }) {
  const [selected, setSelected] = useState(null);
  const [editing, setEditing] = useState(null);

  return (
    <div className="gallery">
      {images.map(img => (
        <div key={img.id} className="gallery-item">
          <img src={img.thumbnail} alt={img.title}
            onClick={() => setSelected(img)} />
          <button onClick={() => setEditing(img)}>Edit</button>
        </div>
      ))}

      {selected && (
        <Suspense fallback={<div className="lightbox-skeleton" />}>
          <Lightbox image={selected} onClose={() => setSelected(null)} />
        </Suspense>
      )}

      {editing && (
        <Suspense fallback={<div className="editor-skeleton" />}>
          <ImageEditor image={editing} onClose={() => setEditing(null)} />
        </Suspense>
      )}
    </div>
  );
}
```

**Challenge 2: Implement a code-split wizard/multi-step form**
```jsx
// Build a multi-step form where each step is a lazy-loaded component.
// Preload the next step when the current step is being filled out.

import React, { Suspense, useState, lazy } from 'react';

const steps = {
  1: lazy(() => import('./steps/PersonalInfo')),
  2: lazy(() => import('./steps/Address')),
  3: lazy(() => import('./steps/Payment')),
  4: lazy(() => import('./steps/Review'))
};

function Wizard() {
  const [step, setStep] = useState(1);
  const [formData, setFormData] = useState({});
  const StepComponent = steps[step];

  const goNext = (data) => {
    const next = step + 1;
    setFormData(prev => ({ ...prev, ...data }));
    // Preload the step after next
    if (steps[next + 1]) {
      steps[next + 1]().catch(() => {});
    }
    setStep(next);
  };

  const goBack = () => setStep(s => Math.max(1, s - 1));

  return (
    <div className="wizard">
      <div className="steps-indicator">
        {Object.keys(steps).map(s => (
          <div key={s} className={`step ${+s <= step ? 'active' : ''}`} />
        ))}
      </div>
      <Suspense fallback={<div className="step-loader">Loading step...</div>}>
        <StepComponent
          data={formData}
          onNext={goNext}
          onBack={goBack}
          isFirst={step === 1}
          isLast={step === Object.keys(steps).length}
        />
      </Suspense>
    </div>
  );
}
```

**Challenge 3: Create a priority-based component preloader**
```jsx
// Build a system that preloads React components based on user interaction
// signals like hover, scroll direction, and idle time.

import React, { useEffect, useRef } from 'react';

function useComponentPreloader(preloadMap) {
  const preloaded = useRef(new Set());

  useEffect(() => {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const componentName = entry.target.dataset.component;
          const preloader = preloadMap[componentName];
          if (preloader && !preloaded.current.has(componentName)) {
            preloaded.current.add(componentName);
            preloader().catch(() => {});
          }
        }
      });
    }, { rootMargin: '500px' });

    const elements = document.querySelectorAll('[data-component]');
    elements.forEach(el => observer.observe(el));

    return () => observer.disconnect();
  }, [preloadMap]);

  return preloaded.current;
}

// Usage in a route component:
function Dashboard() {
  useComponentPreloader({
    'chart': () => import('./Chart'),
    'data-table': () => import('./DataTable'),
    'map': () => import('./MapView')
  });

  return (
    <div>
      <ChartComponent />
    </div>
  );
}
```

## Route-Based Splitting

### What It Is

Splitting bundles by route so each page loads only its own code. The most effective splitting strategy since users typically interact with one page at a time.

### Why It Is Important

Route-based splitting provides the best cost-benefit ratio of any code splitting strategy. Users on the home page should never download code for the settings page or admin dashboard. Since routes represent distinct user journeys, each route's code is independent. This is the first splitting strategy teams should implement—it typically yields the largest bundle size reduction with minimal architectural changes.

### Examples

```javascript
// Plain JS route splitting
const routes = {
  '/': {
    load: () => import('./pages/Home'),
    render: (module) => module.renderHome()
  },
  '/products': {
    load: () => import('./pages/Products'),
    render: (module) => module.renderProducts(getPageParam())
  },
  '/checkout': {
    load: () => import('./pages/Checkout'),
    render: (module) => module.renderCheckout()
  }
};

async function router() {
  const path = window.location.pathname;
  const route = routes[path];

  if (route) {
    const module = await route.load();
    route.render(module);
  }
}

window.addEventListener('popstate', router);

// Vue.js route splitting
// const routes = [
//   { path: '/', component: () => import('./pages/Home.vue') },
//   { path: '/about', component: () => import('./pages/About.vue') }
// ];

// Angular route splitting
// const routes: Routes = [
//   { path: '', loadComponent: () => import('./home/home.component') },
//   { path: 'admin', loadChildren: () => import('./admin/admin.module') }
// ];
```

### Coding Challenges

**Challenge 1: Implement a SPA router with route-based code splitting**
```javascript
// Build a client-side router that dynamically loads page modules.
// Support lazy loading, preloading, scroll restoration, and navigation guards.

class SPA {
  constructor(routes, options = {}) {
    this.routes = new Map();
    this.cache = new Map();
    this.currentPath = null;
    this.guards = [];
    options.container = options.container || document.getElementById('app');

    for (const [path, config] of Object.entries(routes)) {
      this.routes.set(path, config);
    }

    window.addEventListener('popstate', () => this.resolve());
  }

  guard(fn) { this.guards.push(fn); }

  async navigate(path, data = {}) {
    for (const guard of this.guards) {
      const result = await guard(path, this.currentPath);
      if (result === false) return;
    }
    history.pushState(data, '', path);
    await this.resolve();
  }

  async resolve() {
    const path = window.location.pathname;
    const route = this.routes.get(path);

    if (!route) {
      this.renderNotFound();
      return;
    }

    this.currentPath = path;
    const container = document.getElementById('app');
    container.innerHTML = route.loading || '<div>Loading...</div>';

    try {
      let module = this.cache.get(path);
      if (!module) {
        module = await route.load();
        this.cache.set(path, module);
      }
      container.innerHTML = '';
      route.render(module, container);
      if (route.afterRender) route.afterRender();
    } catch (err) {
      container.innerHTML = route.error
        ? route.error(err)
        : `<div>Error loading page: ${err.message}</div>`;
    }
  }

  async preload(path) {
    const route = this.routes.get(path);
    if (route && !this.cache.has(path)) {
      try {
        const module = await route.load();
        this.cache.set(path, module);
      } catch {}
    }
  }

  renderNotFound() {
    document.getElementById('app').innerHTML = '<h1>404 - Page Not Found</h1>';
  }
}
```

**Challenge 2: Create a route preloader based on link visibility**
```javascript
// Preload route chunks when links become visible in the viewport.
// Use IntersectionObserver to detect visible links.

class RoutePreloader {
  constructor() {
    this.observer = new IntersectionObserver(this.onIntersect.bind(this), {
      rootMargin: '100px'
    });
    this.preloaded = new Set();
    this.init();
  }

  init() {
    // Observe all internal navigation links
    document.querySelectorAll('a[href^="/"]').forEach(link => {
      if (link.dataset.route) {
        this.observer.observe(link);
      }
    });

    // Observe mouse hover for additional preloading hints
    document.addEventListener('mouseover', (e) => {
      const link = e.target.closest('a[href^="/"]');
      if (link?.dataset.route && !this.preloaded.has(link.dataset.route)) {
        this.preloadRoute(link.dataset.route);
      }
    }, { passive: true });
  }

  onIntersect(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const link = entry.target;
        if (link.dataset.route && !this.preloaded.has(link.dataset.route)) {
          this.preloadRoute(link.dataset.route);
        }
        this.observer.unobserve(link);
      }
    });
  }

  async preloadRoute(routeName) {
    this.preloaded.add(routeName);
    const routes = window.__ROUTES__;
    if (routes && routes[routeName]) {
      try {
        await routes[routeName].load();
        console.log(`Preloaded route: ${routeName}`);
      } catch {}
    }
  }

  destroy() { this.observer.disconnect(); }
}
```

**Challenge 3: Implement layout-based code splitting with nested routes**
```javascript
// Build a route system that splits layouts and pages independently.
// Layout chunks are shared across pages within the same section.

class LayoutRouter {
  constructor() {
    this.layouts = new Map();
    this.currentLayout = null;
    this.currentPage = null;
  }

  registerLayout(name, loader) {
    this.layouts.set(name, { loader, instance: null });
  }

  async resolve(path) {
    const match = this.matchRoute(path);
    if (!match) {
      await this.renderLayout('public', 'not-found');
      return;
    }

    const { layout, page, params } = match;

    if (layout !== this.currentLayout) {
      await this.loadLayout(layout);
    }

    await this.loadPage(layout, page, params);
  }

  async loadLayout(name) {
    const layout = this.layouts.get(name);
    if (!layout) throw new Error(`Layout ${name} not found`);

    if (!layout.instance) {
      const module = await layout.loader();
      layout.instance = module.default || module;
    }

    document.getElementById('layout-outlet').innerHTML = '';
    const el = layout.instance.render();
    document.getElementById('layout-outlet').appendChild(el);
    this.currentLayout = name;
  }

  async loadPage(layout, page, params) {
    const module = await import(`./pages/${layout}/${page}.js`);
    const Page = module.default || module;
    const container = document.getElementById('page-outlet');
    container.innerHTML = '';
    const el = Page.render(params);
    container.appendChild(el);
    this.currentPage = { layout, page };
  }

  matchRoute(path) {
    const routes = {
      '/': { layout: 'public', page: 'home' },
      '/login': { layout: 'public', page: 'login' },
      '/dashboard': { layout: 'authenticated', page: 'dashboard' },
      '/settings': { layout: 'authenticated', page: 'settings' }
    };
    return routes[path];
  }
}
```

## Bundle Analysis

### What It Is

Bundle analysis examines your production bundles to identify large dependencies, duplicate code, and opportunities for splitting.

### Why It Is Important

Without bundle analysis, teams are blind to what their users are downloading. A single large dependency (like moment.js at 230KB) can account for more bundle size than all application code combined. Bundle analysis reveals: which packages are the largest, which code is duplicated across chunks, what's being imported but never used, and how code splitting changes affect bundle sizes. Regular bundle analysis should be part of every CI pipeline.

### Tools

- **webpack-bundle-analyzer**: Visualizes bundle contents as an interactive treemap.
- **source-map-explorer**: Shows code distribution in source files.
- **Vite bundle visualizer**: Similar for Vite projects.
- **Chrome DevTools Coverage**: Shows used vs unused code.
- **import-cost**: VS Code extension showing imported module sizes.

### Configuration Examples

```javascript
// webpack.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      reportFilename: 'bundle-report.html',
      openAnalyzer: false
    })
  ]
};

// package.json script
// "analyze": "ANALYZE=true webpack --mode production"

// Vite configuration
// vite.config.js
import { visualizer } from 'rollup-plugin-visualizer';

export default {
  plugins: [
    visualizer({
      filename: 'dist/stats.html',
      open: true
    })
  ]
};
```

### Analysis Patterns

```javascript
// 1. Check for duplicate packages
// Run: npx madge --circular src/

// 2. Identify large dependencies
// webpack-bundle-analyzer output shows ~gzipped sizes

// 3. Check code coverage
// Chrome DevTools > Coverage > Record

// 4. Monitor bundle size over time
// Use bundlesize or size-limit packages

// package.json
// "size-limit": [
//   {
//     "path": "dist/index.js",
//     "limit": "170 KB"
//   },
//   {
//     "path": "dist/pages/*.js",
//     "limit": "50 KB"
//   }
// ]
```

### Coding Challenges

**Challenge 1: Build a bundle size budget checker**
```javascript
// Create a tool that enforces bundle size budgets in CI.
// Parse webpack stats JSON, check each chunk against limits,
// and fail the build if any chunk exceeds its budget.

class BundleBudgetChecker {
  constructor(statsPath, budgets) {
    this.stats = require(statsPath);
    this.budgets = budgets;
  }

  check() {
    const errors = [];
    const warnings = [];

    for (const asset of this.stats.assets || []) {
      const budget = this.budgets.find(b => asset.name.match(b.pattern));
      if (!budget) continue;

      const sizeKB = asset.size / 1024;
      const maxKB = this.parseBudget(budget.maxSize);

      if (sizeKB > maxKB) {
        const entry = sizeKB > maxKB * 1.1 ? errors : warnings;
        entry.push({
          asset: asset.name,
          size: `${sizeKB.toFixed(1)} KB`,
          limit: `${maxKB.toFixed(1)} KB`,
          diff: `+${(sizeKB - maxKB).toFixed(1)} KB`,
          severity: sizeKB > maxKB * 1.1 ? 'error' : 'warning'
        });
      }
    }

    return { errors, warnings, passed: errors.length === 0 };
  }

  parseBudget(size) {
    const match = size.match(/^(\d+(?:\.\d+)?)\s*(KB|MB)$/);
    if (!match) throw new Error(`Invalid budget format: ${size}`);
    const value = parseFloat(match[1]);
    return match[2] === 'MB' ? value * 1024 : value;
  }

  report() {
    const result = this.check();
    if (result.warnings.length) {
      console.warn('Bundle size warnings:');
      result.warnings.forEach(w => console.warn(`  ${w.asset}: ${w.size} (limit: ${w.limit})`));
    }
    if (result.errors.length) {
      console.error('Bundle size errors:');
      result.errors.forEach(e => console.error(`  ${e.asset}: ${e.size} (limit: ${e.limit}, ${e.diff})`));
    }
    return result.passed;
  }
}
```

**Challenge 2: Create a duplicate dependency detector**
```javascript
// Build a tool that finds duplicate modules in the bundle.
// Multiple versions of the same package increase bundle size unnecessarily.

class DuplicateDetector {
  constructor(statsData) {
    this.modules = statsData.modules || [];
  }

  findDuplicates() {
    const modulesByName = new Map();

    for (const mod of this.modules) {
      if (!mod.name) continue;
      // Extract package name from module path
      const pkgMatch = mod.name.match(/node_modules\/(@[^/]+\/[^/]+|[^/]+)/);
      if (!pkgMatch) continue;
      const pkg = pkgMatch[1];
      const versionMatch = mod.name.match(/node_modules\/[^/]+\/[^/]+\/([^/]+)/);
      const version = this.getPackageVersion(pkg);

      if (!modulesByName.has(pkg)) {
        modulesByName.set(pkg, []);
      }
      modulesByName.get(pkg).push({
        version,
        size: mod.size,
        path: mod.name
      });
    }

    const duplicates = [];
    for (const [pkg, instances] of modulesByName) {
      if (instances.length > 1) {
        const versions = [...new Set(instances.map(i => i.version))];
        if (versions.length > 1) {
          duplicates.push({
            package: pkg,
            versions,
            instances: instances.length,
            totalSize: instances.reduce((s, i) => s + i.size, 0),
            wasteSize: instances.slice(1).reduce((s, i) => s + i.size, 0)
          });
        }
      }
    }

    return duplicates.sort((a, b) => b.wasteSize - a.wasteSize);
  }

  getPackageVersion(pkg) {
    try {
      return require(`${pkg}/package.json`).version;
    } catch {
      return 'unknown';
    }
  }
}
```

**Challenge 3: Build a bundle diff tool for CI**
```javascript
// Create a tool that compares two bundle stats and reports
// what changed between releases. Integrate with PR comments.

class BundleDiff {
  constructor(beforeStats, afterStats) {
    this.before = this.normalize(beforeStats);
    this.after = this.normalize(afterStats);
  }

  normalize(stats) {
    return (stats.assets || []).map(a => ({
      name: a.name,
      size: a.size,
      chunks: a.chunks
    }));
  }

  diff() {
    const beforeMap = new Map(this.before.map(a => [a.name, a]));
    const afterMap = new Map(this.after.map(a => [a.name, a]));
    const allNames = new Set([...beforeMap.keys(), ...afterMap.keys()]);

    const changes = [];
    for (const name of allNames) {
      const before = beforeMap.get(name);
      const after = afterMap.get(name);

      if (!before) {
        changes.push({ type: 'added', name, size: after.size });
      } else if (!after) {
        changes.push({ type: 'removed', name, size: -before.size });
      } else {
        const diff = after.size - before.size;
        if (Math.abs(diff) > 100) {
          changes.push({
            type: diff > 0 ? 'increased' : 'decreased',
            name,
            size: after.size,
            diff,
            percent: ((diff / before.size) * 100).toFixed(1)
          });
        }
      }
    }

    const totalBefore = this.before.reduce((s, a) => s + a.size, 0);
    const totalAfter = this.after.reduce((s, a) => s + a.size, 0);

    return {
      changes,
      totalBefore,
      totalAfter,
      totalDiff: totalAfter - totalBefore,
      summary: {
        added: changes.filter(c => c.type === 'added').length,
        removed: changes.filter(c => c.type === 'removed').length,
        increased: changes.filter(c => c.type === 'increased').length,
        decreased: changes.filter(c => c.type === 'decreased').length,
        totalChange: totalAfter - totalBefore
      }
    };
  }

  formatMarkdown() {
    const result = this.diff();
    let md = `## Bundle Size Report\n\n`;
    md += `| Metric | Before | After | Change |\n`;
    md += `|--------|--------|-------|--------|\n`;
    md += `| **Total** | ${(result.totalBefore / 1024).toFixed(1)} KB | ${(result.totalAfter / 1024).toFixed(1)} KB | ${result.totalDiff > 0 ? '+' : ''}${(result.totalDiff / 1024).toFixed(1)} KB |\n\n`;

    if (result.changes.length > 0) {
      md += `### Changes\n\n`;
      md += `| Asset | Change | Size | Δ |\n`;
      md += `|-------|--------|------|-----|\n`;
      for (const change of result.changes) {
        const icon = change.type === 'added' ? '🟢' : change.type === 'removed' ? '🔴' : change.type === 'increased' ? '⚠️' : '✅';
        md += `| ${icon} ${change.name} | ${change.type} | ${(change.size / 1024).toFixed(1)} KB | ${change.diff ? `${change.diff > 0 ? '+' : ''}${(change.diff / 1024).toFixed(1)} KB` : '-'} |\n`;
      }
    }

    return md;
  }
}
```

### Real-World Use Cases

- **E-commerce**: Product page loads on demand; checkout page has own chunk.
- **SaaS dashboards**: Admin panels lazy-loaded; public pages eager.
- **Document editors**: Rich text editor only loads when user creates/edits a doc.
- **Multi-language apps**: Locale files dynamically imported per user language.
- **Mobile-specific features**: Code for mobile-only features loaded on mobile detection.

### Common Mistakes

```javascript
// Mistake: Every component as lazy (over-splitting)
// Network overhead of many small requests can be worse

// Mistake: Not handling loading errors
const LazyComp = React.lazy(() => import('./Comp'));
// Without error boundary, network failure crashes the app

// Mistake: Lazy loading above-the-fold content
// Critical initial content should be eagerly loaded

// Mistake: Creating new import() each render
function bad() {
  // New chunk request every render!
  const chart = import('./chart').then(m => m.render());
}

// Mistake: Not preloading anticipated routes
// User hovering a link = preload opportunity
```

### Best Practices

```javascript
// 1. Split by routes (most effective)
// 2. Lazy heavy libraries (charts, editors, PDF)
// 3. Prefetch likely next pages
const LazyPage = React.lazy(() => import(/* webpackPrefetch: true */ './Page'));

// 4. Use consistent chunk naming
// webpackChunkName: 'pages-[request]'

// 5. Monitor bundle sizes
// Add size-limit or bundlesize to CI

// 6. Split vendor chunks (node_modules)
// webpack: optimization.splitChunks.cacheGroups

// 7. Consider network conditions
// Adaptive loading: serve smaller bundles on 3G
```

### Performance Considerations

- Target chunk size: 20-50KB per chunk (gzipped).
- Too many chunks = excessive HTTP requests (HTTP/2 helps).
- Preload critical chunks, prefetch anticipated chunks.
- Server-side rendering solves the flash-of-loading problem.
- Module federation enables micro-frontend code sharing.

### Interview Questions

**Q: What is the difference between code splitting and tree shaking?**
A: Code splitting creates separate bundles loaded on demand. Tree shaking eliminates unused exports within a bundle. Both reduce bundle size but work differently: splitting is strategic (what to load when), shaking is analytical (what code is actually used).

**Q: How do you decide what to code split?**
A: (1) Start with route-level splitting (every route gets its own chunk). (2) Identify heavy libraries loaded early but used late (charts, rich text editors). (3) Split large dependencies into vendor chunks. (4) Use bundle analysis to find large modules that can be deferred.

**Q: How does webpack determine where to split?**
A: Webpack creates a split point at each dynamic import() call. It analyzes the static dependency graph, finds all dynamic imports, and creates separate chunks. Magic comments control chunk naming, prefetching, and preloading behavior.

### Related Topics

- Dynamic imports, Lazy loading, Tree shaking, Bundle optimization
