# Performance Optimization - Profiling, critical rendering path, memory optimization, network optimization

## Introduction

Performance optimization in JavaScript encompasses a wide range of techniques across the entire stack—from writing efficient code and optimizing render paths to reducing network payloads and managing memory. Performance directly impacts user experience, conversion rates, and operational costs. Modern web applications must deliver fast load times, smooth interactions, and efficient resource usage across all devices.

## Profiling Tools (Chrome DevTools)

### What It Is

Profiling is the process of measuring where time and memory are spent in an application. Chrome DevTools provides several profiling tools: the Performance panel (runtime performance), Memory panel (heap snapshots, allocation timelines), Network panel (request timing), and the Performance Monitor (real-time CPU/memory usage).

### Why It Is Important

You cannot optimize what you cannot measure. Profiling reveals the actual bottlenecks in your application, preventing wasted effort on premature optimization. Profiling identifies slow functions, memory leaks, layout thrashing, excessive GC pauses, and network issues.

### How It Works Internally

The Performance panel records a timeline of events: JavaScript execution, rendering, painting, layout, and system-level events. It uses V8's built-in sampling profiler (SIGPROF signal on Linux/macOS) to periodically sample the call stack, producing a statistical view of where time is spent. Flame graphs visualize which functions consume the most CPU time.

### Syntax

```javascript
// Performance API for in-code profiling
const { performance } = require('perf_hooks');

// Mark and measure
performance.mark('start');
// ... code to measure ...
performance.mark('end');
performance.measure('operation', 'start', 'end');

const measurements = performance.getEntriesByType('measure');
console.log(measurements);

// Performance observer
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`${entry.name}: ${entry.duration}ms`);
  }
});
observer.observe({ entryTypes: ['measure', 'function'] });

// User timing API
performance.mark('fetchStart');
await fetch('/api/data');
performance.mark('fetchEnd');
performance.measure('dataFetch', 'fetchStart', 'fetchEnd');

// Performance.now() for high-resolution timing
const start = performance.now();
// ... code ...
const duration = performance.now() - start;
```

### Beginner Examples

```javascript
// Simple timing with console.time
console.time('array operation');
const arr = Array.from({length: 100000}, (_, i) => i);
const sum = arr.reduce((a, b) => a + b, 0);
console.timeEnd('array operation');

// Using performance.now()
const t0 = performance.now();
for (let i = 0; i < 10000; i++) {
  document.querySelector('.element');
}
const t1 = performance.now();
console.log(`Query took ${t1 - t0}ms`);

// Measuring GC pauses
const gcMeasurements = [];
let lastGCTime = performance.now();

const gcObserver = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.entryType === 'gc') {
      gcMeasurements.push({
        duration: entry.duration,
        timestamp: new Date().toISOString()
      });
    }
  }
});
gcObserver.observe({ entryTypes: ['gc'] });

// Long task observation (for main thread blocking)
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.warn(`Long task: ${entry.duration}ms`, entry.attribution);
  }
}).observe({ entryTypes: ['longtask'] });
```

### Intermediate Examples

```javascript
// Custom profiling middleware
function profile(target, name) {
  if (typeof target === 'function') {
    return new Proxy(target, {
      apply(target, thisArg, args) {
        const start = performance.now();
        const result = Reflect.apply(target, thisArg, args);
        const duration = performance.now() - start;
        if (duration > 50) {
          console.warn(`SLOW: ${name || target.name} took ${duration.toFixed(2)}ms`);
        }
        return result;
      }
    });
  }
  
  return new Proxy(target, {
    get(target, prop) {
      const value = target[prop];
      if (typeof value === 'function') {
        return profile(value, `${name}.${String(prop)}`);
      }
      return value;
    }
  });
}

// Usage
const slowObject = {
  compute() {
    let sum = 0;
    for (let i = 0; i < 10000000; i++) sum += i;
    return sum;
  }
};

const profiled = profile(slowObject, 'slowObject');
profiled.compute(); // "SLOW: slowObject.compute took 12.34ms"

// Frame timing (for animations)
let frameCount = 0;
let lastFrameTime = performance.now();

function frameCallback(timestamp) {
  frameCount++;
  if (timestamp - lastFrameTime >= 1000) {
    console.log(`FPS: ${frameCount}`);
    frameCount = 0;
    lastFrameTime = timestamp;
  }
  requestAnimationFrame(frameCallback);
}
requestAnimationFrame(frameCallback);

// Element timing
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`Element ${entry.identifier} rendered at ${entry.renderTime}`);
  }
}).observe({ entryTypes: ['element'] });
```

### Advanced Examples

```javascript
// Flame graph generation (Node.js)
const inspector = require('inspector');
const fs = require('fs');

async function profileCPU(durationMs = 5000) {
  const session = new inspector.Session();
  session.connect();
  
  await session.post('Profiler.enable');
  await session.post('Profiler.start');
  
  await new Promise(r => setTimeout(r, durationMs));
  
  const { profile } = await session.post('Profiler.stop');
  
  // Convert to flame graph format
  const flameData = {
    name: 'root',
    value: 0,
    children: []
  };
  
  const nodeMap = new Map();
  for (const node of profile.nodes) {
    nodeMap.set(node.id, {
      name: `${node.callFrame.functionName} (${node.callFrame.url}:${node.callFrame.lineNumber})`,
      value: 0,
      children: []
    });
  }
  
  // Aggregate sample counts
  for (const nodeId of profile.samples) {
    const node = nodeMap.get(nodeId);
    if (node) node.value++;
  }
  
  // Build tree
  for (const node of profile.nodes) {
    const parent = nodeMap.get(node.id);
    if (parent) {
      for (const childId of node.children || []) {
        const child = nodeMap.get(childId);
        if (child) parent.children.push(child);
      }
    }
  }
  
  fs.writeFileSync('flamegraph.json', JSON.stringify(flameData));
  console.log('Flame graph saved');
  
  await session.post('Profiler.disable');
  session.disconnect();
}

// Memory allocation tracking
async function trackAllocations() {
  const session = new inspector.Session();
  session.connect();
  
  await session.post('HeapProfiler.enable');
  await session.post('HeapProfiler.startTrackingHeapObjects', { trackAllocations: true });
  
  // ... do some operations ...
  
  // Stop tracking and get allocation data
  await session.post('HeapProfiler.stopTrackingHeapObjects');
  session.disconnect();
}
```

## Critical Rendering Path

### What It Is

The Critical Rendering Path (CRP) is the sequence of steps the browser takes to render a page: HTML parsing, CSS parsing, style calculation, layout, paint, and compositing. Optimizing the CRP means reducing the time from receiving HTML bytes to displaying pixels on screen.

### Why It Is Important

CRP optimization directly impacts Largest Contentful Paint (LCP), First Contentful Paint (FCP), and other Core Web Vitals metrics. Faster rendering improves user experience, SEO rankings (Google uses Core Web Vitals), and conversion rates.

### How It Works Internally

The CRP steps:
1. **DOM construction**: Parse HTML → Tokenizer → Tree construction → DOM tree
2. **CSSOM construction**: Parse CSS → CSSOM tree (blocks rendering)
3. **Render tree**: Combine DOM + CSSOM (only visible elements)
4. **Layout**: Calculate geometry (position, size) for each node
5. **Paint**: Fill pixels for each node (layers)
6. **Composite**: Combine layers and display on screen

JavaScript execution blocks DOM construction (unless async/defer). CSS blocks both rendering and script execution.

### Syntax

```javascript
// CRP optimization patterns

// Defer non-critical JavaScript
// <script defer src="app.js"></script>
// <script async src="analytics.js"></script>

// Preload critical resources
// <link rel="preload" href="critical.css" as="style">
// <link rel="preload" href="font.woff2" as="font" crossorigin>

// Preconnect to origins
// <link rel="preconnect" href="https://api.example.com">

// Inline critical CSS
// <style>/* critical above-the-fold CSS */</style>

// Lazy load non-critical resources
// <img loading="lazy" src="image.jpg" alt="...">
// <iframe loading="lazy" src="..."></iframe>

// Resource hints
async function preloadResource(url, as) {
  const link = document.createElement('link');
  link.rel = 'preload';
  link.href = url;
  link.as = as;
  document.head.appendChild(link);
}
```

### Beginner Examples

```javascript
// Measuring CRP
// Paint Timing API
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`${entry.name}: ${entry.startTime}ms`);
  }
}).observe({ entryTypes: ['paint'] });

// First Paint, First Contentful Paint are reported

// Largest Contentful Paint
new PerformanceObserver((list) => {
  const entries = list.getEntries();
  const last = entries[entries.length - 1];
  console.log(`LCP: ${last.startTime}ms`, last.element);
}).observe({ entryTypes: ['largest-contentful-paint'] });

// Layout shifts (Cumulative Layout Shift)
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`CLS impact: ${entry.value}`, entry.sources);
  }
}).observe({ entryTypes: ['layout-shift'] });

// First Input Delay
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`FID: ${entry.processingStart - entry.startTime}ms`);
  }
}).observe({ entryTypes: ['first-input'] });
```

### Intermediate Examples

```javascript
// DOM size optimization
function measureDOMPerformance() {
  const body = document.body;
  const totalElements = document.getElementsByTagName('*').length;
  const totalNodes = document.getElementsByTagName('*').length + 
    document.querySelectorAll('text, comment, cdata').length;
  
  console.log({
    totalElements,
    totalNodes,
    bodyDepth: getMaxDepth(body),
    elementLimit: totalElements > 1500 ? 'WARNING' : 'OK'
  });
  
  // Find deepest node
  function getMaxDepth(node, depth = 0) {
    let max = depth;
    for (let child = node.firstElementChild; child; child = child.nextElementSibling) {
      const childDepth = getMaxDepth(child, depth + 1);
      if (childDepth > max) max = childDepth;
    }
    return max;
  }
}

// Layout thrashing detection
// Bad: forces multiple layouts
function badLayout() {
  for (let i = 0; i < 100; i++) {
    const el = document.getElementById(`item-${i}`);
    el.style.width = `${el.offsetWidth + 10}px`; // Read then write
  }
}

// Good: batch reads, then batch writes
function goodLayout() {
  const widths = [];
  const elements = [];
  
  for (let i = 0; i < 100; i++) {
    const el = document.getElementById(`item-${i}`);
    elements.push(el);
    widths.push(el.offsetWidth); // Batch reads
  }
  
  for (let i = 0; i < elements.length; i++) {
    elements[i].style.width = `${widths[i] + 10}px`; // Batch writes
  }
}

// Virtual scrolling for large lists
class VirtualScroller {
  constructor(container, items, itemHeight, renderItem) {
    this.container = container;
    this.items = items;
    this.itemHeight = itemHeight;
    this.renderItem = renderItem;
    this.visibleItems = Math.ceil(container.clientHeight / itemHeight) + 2;
    
    this.container.addEventListener('scroll', () => this.update());
    this.update();
  }
  
  update() {
    const scrollTop = this.container.scrollTop;
    const startIndex = Math.floor(scrollTop / this.itemHeight);
    const endIndex = Math.min(startIndex + this.visibleItems, this.items.length);
    
    this.container.innerHTML = '';
    this.container.style.height = `${this.items.length * this.itemHeight}px`;
    
    for (let i = startIndex; i < endIndex; i++) {
      const item = this.renderItem(this.items[i], i);
      item.style.position = 'absolute';
      item.style.top = `${i * this.itemHeight}px`;
      this.container.appendChild(item);
    }
  }
}
```

## Memory Optimization

### What It Is

Memory optimization involves reducing memory usage, preventing leaks, and minimizing garbage collection (GC) pauses. In long-running applications (Single Page Apps, Node.js servers), memory management is critical for performance and stability.

### Why It Is Important

Memory leaks cause gradual performance degradation and eventually crashes (out-of-memory). Frequent GC pauses cause jank in animations and unresponsive interfaces. High memory usage leads to OOM kills, swap thrashing, and increased infrastructure costs.

### How It Works Internally

V8's garbage collector uses a generational approach. The young generation (new space) is collected frequently with a fast scavenge algorithm. Survivors are promoted to the old generation, which uses mark-sweep and mark-compact (slower but less frequent). GC pauses can be minimized by reducing allocation rate and avoiding reference retention.

### Syntax

```javascript
// Memory leak detection patterns

// 1. Detached DOM elements (common SPA leak)
function createDetachedLeak() {
  let elements = [];
  return function create() {
    for (let i = 0; i < 1000; i++) {
      const el = document.createElement('div');
      elements.push(el); // Retained reference
      // el not attached to DOM, but referenced
    }
  };
}

// 2. Closures retaining large data
function createClosureLeak() {
  const largeData = new Array(1000000).fill('x');
  return function() {
    console.log(largeData.length); // Closure keeps largeData alive
  };
}

// 3. Event listener leaks
class Component {
  constructor(element) {
    this.handler = () => {
      console.log(this.constructor.name);
    };
    element.addEventListener('click', this.handler);
    // Missing: removeEventListener when component is destroyed
  }
  
  destroy(element) {
    element.removeEventListener('click', this.handler);
  }
}

// 4. Cache without eviction
class LeakyCache {
  constructor() {
    this.cache = new Map();
  }
  
  get(key) {
    return this.cache.get(key);
  }
  
  set(key, value) {
    this.cache.set(key, value); // Never cleaned!
  }
}

// Fixed cache with eviction
class LRUCache {
  constructor(maxSize = 100) {
    this.maxSize = maxSize;
    this.cache = new Map();
  }
  
  get(key) {
    if (!this.cache.has(key)) return undefined;
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value); // Move to end (most recently used)
    return value;
  }
  
  set(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey); // Evict least recently used
    }
    this.cache.set(key, value);
  }
}
```

### Beginner Examples

```javascript
// Object pooling to reduce GC pressure
class VectorPool {
  constructor(size = 1000) {
    this.pool = [];
    for (let i = 0; i < size; i++) {
      this.pool.push({ x: 0, y: 0, z: 0 });
    }
  }
  
  acquire(x = 0, y = 0, z = 0) {
    let vec = this.pool.pop();
    if (!vec) vec = { x, y, z };
    else { vec.x = x; vec.y = y; vec.z = z; }
    return vec;
  }
  
  release(vec) {
    this.pool.push(vec);
  }
}

// Avoiding memory leaks with WeakMap
const elementData = new WeakMap();

function attachData(element, data) {
  elementData.set(element, data);
  // When element is removed from DOM, data is automatically GC'd
}

// String concatenation optimization
// Bad: creates many intermediate strings
let str = '';
for (let i = 0; i < 10000; i++) {
  str += i; // New string each iteration
}

// Good: use array join
const parts = [];
for (let i = 0; i < 10000; i++) {
  parts.push(String(i));
}
const result = parts.join('');

// Even better: use template literals when possible

// Avoiding large retains in closures
// Bad:
function processItems(items) {
  const hugeArray = new Array(1000000);
  items.forEach(item => {
    // hugeArray is retained in this closure
    doSomething(item, hugeArray.length);
  });
}

// Good:
function processItems(items) {
  const hugeArraySize = new Array(1000000).length;
  items.forEach(item => {
    doSomething(item, hugeArraySize);
  });
}
```

### Intermediate Examples

```javascript
// Memory monitoring
class MemoryMonitor {
  constructor(threshold = 100 * 1024 * 1024) {
    this.threshold = threshold;
    this.measurements = [];
    this.checkInterval = setInterval(() => this.check(), 10000);
    this.setupWarning();
  }
  
  check() {
    if (!performance.memory) return; // Chrome only
    
    const { usedJSHeapSize, totalJSHeapSize, jsHeapSizeLimit } = performance.memory;
    
    this.measurements.push({
      timestamp: Date.now(),
      used: usedJSHeapSize,
      total: totalJSHeapSize,
      limit: jsHeapSizeLimit
    });
    
    const usagePercent = (usedJSHeapSize / jsHeapSizeLimit) * 100;
    
    if (usedJSHeapSize > this.threshold) {
      console.warn(`Memory warning: ${formatBytes(usedJSHeapSize)} used`);
    }
    
    return {
      used: formatBytes(usedJSHeapSize),
      total: formatBytes(totalJSHeapSize),
      percent: usagePercent.toFixed(1) + '%'
    };
  }
  
  setupWarning() {
    const origWarn = console.warn;
    console.warn = (...args) => {
      this.check();
      origWarn.apply(console, args);
    };
  }
  
  getGrowth() {
    if (this.measurements.length < 2) return 0;
    const first = this.measurements[0].used;
    const last = this.measurements[this.measurements.length - 1].used;
    return last - first;
  }
  
  cleanup() {
    clearInterval(this.checkInterval);
  }
}

function formatBytes(bytes) {
  const units = ['B', 'KB', 'MB', 'GB'];
  let i = 0;
  while (bytes >= 1024 && i < units.length - 1) {
    bytes /= 1024;
    i++;
  }
  return `${bytes.toFixed(1)} ${units[i]}`;
}

// Manual GC triggering (for debugging, not production)
if (global.gc) {
  global.gc();
  console.log('Manual GC triggered');
}

// Node.js memory management
// --max-old-space-size=4096
// --optimize-for-size
// --expose-gc for debugging
```

## Network Optimization

### What It Is

Network optimization reduces the time and resources needed to transfer data between server and client. Techniques include minimizing payload sizes, reducing round trips, leveraging caching, using CDNs, and optimizing resource loading order.

### Why It Is Important

Network time dominates page load time, especially on mobile networks. Every additional kilobyte and round trip delays page rendering. Core Web Vitals (LCP, FCP, TTFB) are heavily influenced by network performance.

### How It Works Internally

Key techniques:
- **Minification**: Remove whitespace, comments, shorten variable names
- **Compression**: Gzip/Brotli compress text resources (60-90% reduction)
- **Tree shaking**: Remove unused code
- **Code splitting**: Load only what's needed
- **Caching**: Cache-Control, ETag, Service Workers
- **CDN**: Distribute content geographically
- **HTTP/2**: Multiplexing, server push
- **Resource hints**: Preload, preconnect, prefetch
- **Image optimization**: WebP, AVIF, responsive images, lazy loading

### Syntax

```javascript
// Service Worker caching strategies
// Cache-First (for stable assets)
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request).then((networkResponse) => {
        const cache = await caches.open('dynamic');
        cache.put(event.request, networkResponse.clone());
        return networkResponse;
      });
    })
  );
});

// Network-First (for dynamic content)
self.addEventListener('fetch', (event) => {
  event.respondWith(
    fetch(event.request).then((response) => {
      const cache = await caches.open('api');
      cache.put(event.request, response.clone());
      return response;
    }).catch(() => caches.match(event.request))
  );
});

// Stale-While-Revalidate
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((cached) => {
      const fetchPromise = fetch(event.request).then((networkResponse) => {
        const cache = await caches.open('dynamic');
        cache.put(event.request, networkResponse.clone());
        return networkResponse;
      });
      return cached || fetchPromise;
    })
  );
});
```

### Beginner Examples

```javascript
// Resource loading optimization

// Preload critical resources
// <link rel="preload" href="/styles/critical.css" as="style">
// <link rel="preload" href="/fonts/inter.woff2" as="font" crossorigin>

// Preconnect to third-party origins
// <link rel="preconnect" href="https://api.example.com">
// <link rel="dns-prefetch" href="https://api.example.com">

// Lazy loading images
document.querySelectorAll('img[data-src]').forEach(img => {
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        img.src = img.dataset.src;
        observer.unobserve(img);
      }
    });
  });
  observer.observe(img);
});

// Resource hint API
const hints = [
  { rel: 'preload', href: '/critical.js', as: 'script' },
  { rel: 'preconnect', href: 'https://fonts.googleapis.com' }
];

hints.forEach(({ rel, href, as }) => {
  const link = document.createElement('link');
  link.rel = rel;
  link.href = href;
  if (as) link.as = as;
  document.head.appendChild(link);
});
```

### Intermediate Examples

```javascript
// Bundle size analysis
import { getBundleSize } from './bundler-stats';

async function analyzeBundle() {
  // Use webpack-bundle-analyzer or vite-bundle-visualizer
  // In code:
  const modules = await getBundleSize();
  
  const totalSize = modules.reduce((sum, m) => sum + m.size, 0);
  const largeModules = modules
    .sort((a, b) => b.size - a.size)
    .slice(0, 10);
  
  console.log('Top 10 largest modules:');
  largeModules.forEach(m => {
    console.log(`${m.name}: ${(m.size / 1024).toFixed(1)}KB`);
  });
}

// Network waterfall analysis
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.entryType === 'resource') {
      console.log({
        name: entry.name,
        duration: `${entry.duration.toFixed(0)}ms`,
        dns: `${entry.domainLookupEnd - entry.domainLookupStart}ms`,
        tcp: `${entry.connectEnd - entry.connectStart}ms`,
        tls: `${entry.secureConnectionStart ? entry.connectEnd - entry.secureConnectionStart : 0}ms`,
        ttfb: `${entry.responseStart - entry.requestStart}ms`,
        download: `${entry.responseEnd - entry.responseStart}ms`,
        size: `${(entry.transferSize / 1024).toFixed(1)}KB`
      });
    }
  }
}).observe({ entryTypes: ['resource'] });

// Compression for API responses (Node.js)
const zlib = require('zlib');
const compression = require('compression');
app.use(compression({
  filter: (req, res) => {
    if (req.headers['x-no-compression']) return false;
    return compression.filter(req, res);
  },
  level: 6 // Default compression level
}));

// HTTP/2 server push (use with caution)
// Better to use preload links than push
```

### Advanced Examples

```javascript
// Adaptive loading based on network conditions
const connection = navigator.connection || navigator.mozConnection;

function getNetworkQuality() {
  if (!connection) return 'unknown';
  
  const { effectiveType, rtt, downlink, saveData } = connection;
  
  if (saveData) return 'slow'; // Data saver mode
  
  if (effectiveType === 'slow-2g' || effectiveType === '2g') return 'slow';
  if (effectiveType === '3g' && rtt > 200) return 'medium';
  return 'fast';
}

// Load resources based on network
async function loadOptimalResources() {
  const quality = getNetworkQuality();
  
  switch (quality) {
    case 'slow':
      // Load minimal version, no images
      await import('./core.min.js');
      loadStyle('/styles/core.css');
      break;
    case 'medium':
      await import('./medium.js');
      loadLazyImages();
      break;
    case 'fast':
      await import('./full.js');
      loadHighQualityImages();
      break;
  }
}

// Connection change listener
if (connection) {
  connection.addEventListener('change', () => {
    console.log('Network changed:', connection.effectiveType);
    // Adjust quality
  });
}

// Memory-adaptive image loading
function loadAppropriateImage(imgElement, sources) {
  const memory = performance.memory;
  
  let quality = 'high';
  
  if (memory) {
    const usedRatio = memory.usedJSHeapSize / memory.jsHeapSizeLimit;
    if (usedRatio > 0.8) quality = 'low';
    else if (usedRatio > 0.5) quality = 'medium';
  }
  
  imgElement.src = sources[quality] || sources.high;
  imgElement.loading = quality === 'low' ? 'lazy' : 'eager';
}

// Resource prioritization API (Chrome)
// <link rel="preload" href="..." as="...">
// <link rel="prefetch" href="..."> (for next page)
// <link rel="prerender" href="..."> (full page prefetch)

// Priority hints (Chrome 101+)
// <img src="hero.jpg" fetchpriority="high">
// <img src="below-fold.jpg" fetchpriority="low">
```

### Real-World Use Cases

- Core Web Vitals optimization (LCP, FID, CLS)
- E-commerce conversion rate optimization
- Mobile web performance (3G/4G networks)
- Real-time application responsiveness
- Large data visualization performance
- Game and animation framerate optimization
- Microservices API response times

### Common Mistakes

- Premature optimization without profiling
- Ignoring memory leaks (assumed GC handles everything)
- Over-optimizing images (loss of quality for minimal size gain)
- Using too many resource hints (preloading everything defeats the purpose)
- Optimizing for desktop only (ignoring mobile constraints)
- Not measuring the actual user experience (Real User Monitoring)
- Aggressively caching without proper invalidation

### Best Practices

- Profile before optimizing (use DevTools, Lighthouse, WebPageTest)
- Measure with Real User Monitoring (RUM) data
- Optimize the Critical Rendering Path (inline critical CSS, async JS)
- Implement proper code splitting (route-based, component-based)
- Use appropriate caching strategies (service workers, HTTP caching)
- Optimize images (WebP, AVIF, responsive sizes, lazy loading)
- Minimize main thread work (offload to Web Workers)
- Use HTTP/2 or HTTP/3 for multiplexing
- Monitor bundle sizes with CI checks

### Performance Considerations

- Each optimization has tradeoffs (memory vs CPU, size vs quality)
- Mobile devices have slower CPUs, less memory, slower networks
- Core Web Vitals targets: LCP < 2.5s, FID < 100ms, CLS < 0.1
- First meaningful paint goals: under 1s
- JavaScript bundle size budgets: < 200KB (critical), < 500KB (total)
- Time to Interactive: under 5s on 3G

### Interview Questions

**Q: What is the Critical Rendering Path and how do you optimize it?** A: The CRP is the sequence: HTML → DOM, CSS → CSSOM, Render Tree, Layout, Paint, Composite. Optimize by: inlining critical CSS, deferring non-critical CSS/JS, using async/defer for scripts, preloading hero images, minimizing render-blocking resources, and reducing DOM/CSSOM size.

**Q: How do you identify and fix JavaScript memory leaks?** A: Use Chrome DevTools Memory panel to take heap snapshots. Compare two snapshots (before/after an operation). Look for growing object counts, detached DOM trees, and unexpected retained sizes. Common fixes: remove event listeners, nullify references, use WeakMap for caches, avoid closures retaining large data, and implement proper cleanup in component lifecycle methods.


### Coding Challenges
```javascript
// Challenge 1: Memory leak detector
function detectMemoryLeak() {
  const items = [];
  setInterval(() => { items.push(new Array(1000).fill('leak')); }, 100);
  console.warn('Potential leak: items array grows unbounded');
}

// Challenge 2: Profile a function's execution time
function profile(fn, label = fn.name) {
  return function(...args) {
    const start = performance.now();
    const result = fn.apply(this, args);
    const end = performance.now();
    console.log(`${label}: ${end - start}ms`);
    return result;
  };
}
const slowSum = profile((n) => { let s = 0; for(let i=0; i<n; i++) s += i; return s; }, 'slowSum');
console.log(slowSum(1000000));

// Challenge 3: Implement virtual scrolling (simplified)
function virtualScroll(container, items, itemHeight, visibleCount) {
  const startIndex = Math.floor(container.scrollTop / itemHeight);
  const endIndex = startIndex + visibleCount;
  return items.slice(startIndex, endIndex).map((item, i) => ({
    ...item,
    index: startIndex + i,
    style: { position: 'absolute', top: (startIndex + i) * itemHeight + 'px' }
  }));
}
```

### Related Topics

- V8 garbage collection internals
- Web Vitals (LCP, FID, CLS, INP)
- Service Worker and caching strategies
- HTTP/2 and HTTP/3
- CDN configuration and edge computing
- Image optimization (responsive, formats)
- CSS performance (containment, will-change, GPU acceleration)
- Build tool optimization (tree shaking, code splitting)
- Performance budgets and CI enforcement
- Real User Monitoring (RUM) vs Synthetic Monitoring
