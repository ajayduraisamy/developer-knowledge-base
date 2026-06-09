# Lazy Loading - IntersectionObserver, dynamic imports, image lazy loading

## Introduction

Lazy loading defers loading of resources until they are actually needed. This reduces initial page load time, saves bandwidth, and improves perceived performance. Key techniques include IntersectionObserver for visibility-based loading, dynamic imports for code splitting, and native lazy loading for images.

## IntersectionObserver API

### What It Is

IntersectionObserver asynchronously observes changes in the intersection of a target element with an ancestor element or the viewport. It's the modern, performant way to detect element visibility.

### Why It Is Important

Before IntersectionObserver, scroll-based detection required expensive scroll event listeners, getBoundingClientRect calls, and setTimeout throttling. IntersectionObserver is callback-based, runs off the main thread, and provides precise intersection ratios.

### How It Works Internally

The browser internally tracks element positions and fires the callback when the intersection ratio crosses a threshold. No scroll event listeners needed. The observation runs asynchronously.

### Syntax

```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      // Element is visible
      entry.target.src = entry.target.dataset.src; // Load image
      observer.unobserve(entry.target); // Stop observing once loaded
    }
  });
}, { threshold: 0.1 });

document.querySelectorAll('.lazy').forEach(el => observer.observe(el));
```

### Beginner Examples

```javascript
// Image lazy loading with IntersectionObserver
function lazyLoadImages() {
  const images = document.querySelectorAll('img[data-src]');
  if (!images.length) return;

  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const img = entry.target;
        img.src = img.dataset.src;
        img.removeAttribute('data-src');
        observer.unobserve(img);
      }
    });
  });

  images.forEach(img => observer.observe(img));
}

// HTML: <img data-src="image.jpg" alt="Lazy" width="800" height="600">

// Lazy loading iframes
const iframes = document.querySelectorAll('iframe[data-src]');
const iframeObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.src = entry.target.dataset.src;
      iframeObserver.unobserve(entry.target);
    }
  });
});
iframes.forEach(iframe => iframeObserver.observe(iframe));
```

### Intermediate Examples

```javascript
// Generic lazy loader with multiple options
class LazyLoader {
  constructor(options = {}) {
    this.options = {
      root: null,
      rootMargin: '50px',
      threshold: 0,
      once: true,
      ...options
    };
    this.observer = new IntersectionObserver(
      this.handleIntersect.bind(this),
      this.options
    );
    this.pending = new Set();
  }

  observe(element, callback) {
    if (this.options.once && element.dataset._lazyLoaded) return;
    this.pending.add(element);
    element.dataset._lazyPending = 'true';
    this.observer.observe(element);
    // Store callback on element
    element._lazyCallback = callback;
  }

  handleIntersect(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const el = entry.target;
        if (el._lazyCallback) {
          el._lazyCallback(el);
          el.dataset._lazyLoaded = 'true';
        }
        delete el._lazyCallback;
        delete el.dataset._lazyPending;
        this.pending.delete(el);
        if (this.options.once) {
          this.observer.unobserve(el);
        }
      }
    });
  }

  disconnect() {
    this.observer.disconnect();
    this.pending.clear();
  }

  get pendingCount() { return this.pending.size; }
}

// Usage
const loader = new LazyLoader({ rootMargin: '200px' });
document.querySelectorAll('[data-lazy]').forEach(el => {
  loader.observe(el, (element) => {
    if (element.tagName === 'IMG') {
      element.src = element.dataset.src;
    } else if (element.tagName === 'DIV') {
      element.style.backgroundImage = 'url(' + element.dataset.bg + ')';
    }
  });
});

// Infinite scroll with IntersectionObserver
function infiniteScroll(container, loadMore) {
  const sentinel = document.createElement('div');
  sentinel.style.height = '1px';
  container.appendChild(sentinel);

  const observer = new IntersectionObserver(async ([entry]) => {
    if (entry.isIntersecting) {
      const hasMore = await loadMore();
      if (!hasMore) {
        observer.unobserve(sentinel);
        sentinel.textContent = 'No more items';
      }
    }
  }, { rootMargin: '200px' });

  observer.observe(sentinel);
  return () => observer.disconnect();
}
```

### Advanced Examples

```javascript
// Priority-based lazy loading
class PriorityLazyLoader {
  constructor() {
    this.observer = new IntersectionObserver(this.onIntersect.bind(this), {
      rootMargin: '100px',
      threshold: [0, 0.25, 0.5, 0.75, 1]
    });
    this.queue = [];
  }

  add(element, loadFn, priority = 0) {
    this.queue.push({ element, loadFn, priority, added: Date.now() });
    this.queue.sort((a, b) => b.priority - a.priority || a.added - b.added);
    this.observer.observe(element);
  }

  onIntersect(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting && entry.intersectionRatio > 0.5) {
        this.loadElement(entry.target);
      }
    });
  }

  loadElement(element) {
    const item = this.queue.find(i => i.element === element);
    if (item) {
      item.loadFn();
      this.queue = this.queue.filter(i => i !== item);
      this.observer.unobserve(element);
    }
  }

  preloadAll() {
    this.queue.forEach(item => item.loadFn());
    this.queue = [];
    this.observer.disconnect();
  }
}

// Video lazy loading
class VideoLazyLoader {
  constructor() {
    this.observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        const video = entry.target;
        if (entry.isIntersecting) {
          video.src = video.dataset.src;
          if (video.dataset.autoplay) video.play();
          this.observer.unobserve(video);
        } else if (!video.paused) {
          video.pause();
        }
      });
    }, { rootMargin: '100px' });
  }

  observe(video) { this.observer.observe(video); }
  disconnect() { this.observer.disconnect(); }
}

// Virtual list with IntersectionObserver
class VirtualList {
  constructor(container, itemHeight, totalItems, renderItem) {
    this.container = container;
    this.itemHeight = itemHeight;
    this.totalItems = totalItems;
    this.renderItem = renderItem;
    this.visibleItems = Math.ceil(container.clientHeight / itemHeight) + 5;
    this.setup();
  }

  setup() {
    this.container.style.overflow = 'auto';
    this.container.style.position = 'relative';
    this.contentHeight = document.createElement('div');
    this.contentHeight.style.height = (this.totalItems * this.itemHeight) + 'px';
    this.container.appendChild(this.contentHeight);

    this.viewport = document.createElement('div');
    this.viewport.style.position = 'absolute';
    this.viewport.style.top = '0';
    this.viewport.style.width = '100%';
    this.contentHeight.appendChild(this.viewport);

    this.container.addEventListener('scroll', () => this.update());
    this.update();
  }

  update() {
    const scrollTop = this.container.scrollTop;
    const startIndex = Math.floor(scrollTop / this.itemHeight);
    const endIndex = Math.min(startIndex + this.visibleItems, this.totalItems);

    this.viewport.innerHTML = '';
    this.viewport.style.top = (startIndex * this.itemHeight) + 'px';

    for (let i = startIndex; i < endIndex; i++) {
      this.viewport.appendChild(this.renderItem(i));
    }
  }
}
```

### Real-World Use Cases

- **E-commerce product images**: Lazy load below-the-fold product images.
- **Social media feeds**: Infinite scroll with IntersectionObserver triggers.
- **Single-page applications**: Dynamic imports for route-based code splitting.
- **Media galleries**: Lazy load images/videos as user scrolls.
- **Dashboard widgets**: Lazy load widgets that are below the fold.
- **Chart libraries**: Load heavy charting libs only when chart section is visible.

### Common Mistakes

```javascript
// Mistake: Not providing dimensions (causes layout shift)
<img data-src="image.jpg" /> <!-- No width/height! -->
// Fix: Always set width and height attributes

// Mistake: Lazy loading above-the-fold content
// First-visible content should load eagerly (no lazy)

// Mistake: IntersectionObserver with rootMargin too large
// 1000px rootMargin defeats the purpose of lazy loading

// Mistake: Not unobserving after load
// Observer keeps firing if you don't unobserve

// Mistake: Dynamic importing synchronously
const module = await import('./heavy.js'); // Don't block render!
// Fix: Import in response to user action, not during load
```

### Best Practices

```javascript
// Use native loading='lazy' for simple cases
<img src="photo.jpg" loading="lazy" width="800" height="600" alt="">

// Use IntersectionObserver for custom behavior
const observer = new IntersectionObserver((entries) => {
  entries.forEach(e => {
    if (e.isIntersecting) {
      // Custom load logic
      observer.unobserve(e.target);
    }
  });
}, { rootMargin: '200px', threshold: 0.01 });

// Combine for progressive enhancement
if ('loading' in HTMLImageElement.prototype) {
  img.loading = 'lazy';
} else {
  observer.observe(img);
}

// Placeholder techniques
// 1. Low-quality image placeholder (LQIP) - tiny blurry image
// 2. Dominant color - solid color background
// 3. SVG skeleton - outline shape
// 4. BlurHash - compact representation of image colors
```

### Coding Challenges

**Challenge 1: Implement a lazy image gallery with error handling**
```javascript
// Create a gallery that loads images lazily as they scroll into view.
// Use IntersectionObserver with rootMargin: '150px'.
// Show a loading spinner while the image loads.
// Display a fallback image if loading fails.

class LazyGallery {
  constructor(container) {
    this.container = container;
    this.observer = new IntersectionObserver(this.onIntersect.bind(this), {
      rootMargin: '150px'
    });
    this.init();
  }

  init() {
    this.container.querySelectorAll('img[data-src]').forEach(img => {
      img.src = 'data:image/svg+xml,...'; // Placeholder
      img.style.opacity = '0';
      img.style.transition = 'opacity 0.3s';
      const spinner = document.createElement('div');
      spinner.className = 'spinner';
      img.parentNode.insertBefore(spinner, img.nextSibling);
      spinner.dataset.for = img.dataset.src;
      this.observer.observe(img);
    });
  }

  onIntersect(entries) {
    entries.forEach(entry => {
      if (!entry.isIntersecting) return;
      const img = entry.target;
      this.observer.unobserve(img);
      const spinner = this.container.querySelector(`[data-for="${img.dataset.src}"]`);
      const temp = new Image();
      temp.onload = () => {
        img.src = img.dataset.src;
        img.onload = () => {
          img.style.opacity = '1';
          if (spinner) spinner.remove();
        };
      };
      temp.onerror = () => {
        img.src = 'fallback.jpg';
        img.style.opacity = '1';
        if (spinner) spinner.remove();
      };
      temp.src = img.dataset.src;
    });
  }

  destroy() { this.observer.disconnect(); }
}
```

**Challenge 2: Build a scroll-spy navigation system**
```javascript
// Implement scroll-spy that highlights nav items based on visible sections.
// Use IntersectionObserver with multiple thresholds.
// Support smooth scrolling to sections on click.

function createScrollSpy(navContainer, sectionSelector, options = {}) {
  const sections = document.querySelectorAll(sectionSelector);
  const navLinks = navContainer.querySelectorAll('a[href^="#"]');
  const activeClass = options.activeClass || 'active';
  const sectionMap = new Map();

  sections.forEach(section => {
    sectionMap.set(section, section.id);
  });

  const observer = new IntersectionObserver((entries) => {
    let activeId = null;
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        activeId = entry.target.id;
      }
    });
    if (activeId) {
      navLinks.forEach(link => {
        link.classList.toggle(activeClass, link.getAttribute('href') === `#${activeId}`);
      });
    }
  }, { threshold: options.threshold || 0.3, rootMargin: options.rootMargin || '-80px 0px 0px 0px' });

  sections.forEach(section => observer.observe(section));

  navLinks.forEach(link => {
    link.addEventListener('click', (e) => {
      e.preventDefault();
      const target = document.querySelector(link.getAttribute('href'));
      if (target) {
        target.scrollIntoView({ behavior: 'smooth' });
      }
    });
  });

  return () => observer.disconnect();
}
```

**Challenge 3: Implement infinite scroll with IntersectionObserver**
```javascript
// Build an infinite scroll component that loads pages of data.
// Use a sentinel element to trigger loading.
// Support loading states, error recovery, and end-of-list detection.

class InfiniteScroll {
  constructor(container, fetcher, options = {}) {
    this.container = container;
    this.fetcher = fetcher;
    this.options = {
      rootMargin: '200px',
      loadingTemplate: () => '<div class="loader">Loading...</div>',
      errorTemplate: (err) => `<div class="error">${err} <button class="retry">Retry</button></div>`,
      ...options
    };
    this.page = 0;
    this.hasMore = true;
    this.isLoading = false;
    this.sentinel = null;
    this.observer = null;
    this.setup();
  }

  setup() {
    this.sentinel = document.createElement('div');
    this.sentinel.className = 'infinite-scroll-sentinel';
    this.container.appendChild(this.sentinel);
    this.observer = new IntersectionObserver(this.onIntersect.bind(this), {
      rootMargin: this.options.rootMargin
    });
    this.observer.observe(this.sentinel);
  }

  async onIntersect(entries) {
    if (!entries[0].isIntersecting || this.isLoading || !this.hasMore) return;
    this.isLoading = true;
    this.sentinel.innerHTML = this.options.loadingTemplate();
    try {
      const { items, hasMore } = await this.fetcher(this.page);
      this.hasMore = hasMore;
      this.page++;
      items.forEach(item => this.container.appendChild(this.renderItem(item)));
      if (!this.hasMore) {
        this.sentinel.innerHTML = '<div class="end">No more results</div>';
        this.observer.unobserve(this.sentinel);
      }
    } catch (err) {
      this.sentinel.innerHTML = this.options.errorTemplate(err.message);
      this.sentinel.querySelector('.retry')?.addEventListener('click', () => {
        this.page--;
        this.isLoading = false;
        this.onIntersect([{ isIntersecting: true }]);
      });
    }
    this.isLoading = false;
  }

  renderItem(item) {
    const div = document.createElement('div');
    div.textContent = JSON.stringify(item);
    return div;
  }

  destroy() {
    this.observer?.disconnect();
    this.sentinel?.remove();
  }
}
```

### Performance Considerations

- Too many IntersectionObserver instances can slow down the page. Batch observations.
- rootMargin of 200-500px provides a good balance (loads before user reaches element).
- Native loading='lazy' has ~300px rootMargin default in Chrome.
- Dynamic imports add network request latency; preload critical chunks.
- Image lazy loading can cause layout shifts if dimensions aren't set.

### Interview Questions

**Q: How does IntersectionObserver differ from scroll event listeners?**
A: IntersectionObserver runs asynchronously off the main thread, provides precise intersection data (thresholds, ratios), and doesn't require scroll event handlers. Scroll-based detection requires getBoundingClientRect calls on the main thread, which can cause jank.

**Q: What is the loading attribute and how does it work?**
A: loading='lazy' tells the browser to defer loading off-screen images/iframes until the user scrolls near them. The browser has built-in prioritization. loading='eager' loads immediately. This is supported in modern Chrome, Firefox, and Safari.

**Q: How would you lazy load a JavaScript library based on user interaction?**
A: Use dynamic import() triggered by the interaction:
button.addEventListener('click', async () => {
  const { editor } = await import('./rich-editor.js');
  editor.init(targetElement);
});
The editor code is only downloaded when the user clicks the button.

### Related Topics

- Code splitting, Dynamic imports, Web performance, Critical rendering path

## Dynamic Imports

### What It Is

Dynamic imports (import()) load JavaScript modules on demand. They return a Promise that resolves with the module namespace object. This is the foundation of code splitting.

### Syntax

```javascript
// Static import (bundled eagerly)
import { heavyModule } from './heavy.js';

// Dynamic import (loaded on demand)
const module = await import('./heavy.js');
module.heavyFunction();

// Conditional import
if (condition) {
  const { feature } = await import('./feature.js');
  feature.init();
}
```

### Examples

```javascript
// Route-based lazy loading (SPA)
const routes = {
  '/dashboard': () => import('./pages/Dashboard.js'),
  '/settings': () => import('./pages/Settings.js'),
  '/profile': () => import('./pages/Profile.js')
};

async function navigate(path) {
  const loader = routes[path];
  if (loader) {
    const module = await loader();
    module.renderPage();
  }
}

// Lazy loading a heavy library
async function formatDate(date, formatStr) {
  if (!formatDate.cache) {
    const moment = await import('moment');
    formatDate.cache = moment;
  }
  return formatDate.cache(date).format(formatStr);
}
```

### Coding Challenges

**Challenge 1: Build a lazy-loaded tab component**
```javascript
// Create a tab component where each tab's content is loaded dynamically.
// Import a module per tab. Show a loading indicator. Handle errors.
// Preload the adjacent tab content on hover.

class LazyTabs {
  constructor(container, tabs) {
    this.container = container;
    this.tabs = tabs;
    this.cache = new Map();
    this.activeIndex = -1;
    this.render();
  }

  render() {
    const nav = document.createElement('nav');
    this.tabs.forEach((tab, i) => {
      const btn = document.createElement('button');
      btn.textContent = tab.label;
      btn.addEventListener('click', () => this.activate(i));
      btn.addEventListener('mouseenter', () => this.preload(i));
      nav.appendChild(btn);
    });
    this.content = document.createElement('div');
    this.content.className = 'tab-content';
    this.container.append(nav, this.content);
  }

  async activate(index) {
    if (index === this.activeIndex) return;
    this.content.innerHTML = '<div class="spinner">Loading...</div>';
    this.activeIndex = index;
    const tab = this.tabs[index];
    try {
      let module = this.cache.get(tab.label);
      if (!module) {
        module = await tab.loader();
        this.cache.set(tab.label, module);
      }
      this.content.innerHTML = '';
      module.render(this.content);
    } catch (err) {
      this.content.innerHTML = `<div class="error">Failed to load: ${err.message}</div>`;
    }
  }

  async preload(index) {
    const tab = this.tabs[index];
    if (!this.cache.has(tab.label)) {
      try {
        const module = await tab.loader();
        this.cache.set(tab.label, module);
      } catch {}
    }
  }

  destroy() {
    this.container.innerHTML = '';
    this.cache.clear();
  }
}
```

**Challenge 2: Implement a feature-flag system with dynamic imports**
```javascript
// Build a system that conditionally loads features based on flags.
// Features are lazy-loaded modules. Support A/B testing.

class FeatureManager {
  constructor() {
    this.features = new Map();
    this.running = new Map();
  }

  async init(flagEndpoint) {
    const response = await fetch(flagEndpoint);
    const flags = await response.json();
    for (const [name, config] of Object.entries(flags)) {
      this.features.set(name, config);
    }
  }

  isEnabled(name) {
    return this.features.get(name)?.enabled === true;
  }

  getVariant(name) {
    return this.features.get(name)?.variant || 'control';
  }

  async loadFeature(name) {
    if (!this.isEnabled(name)) {
      return { enabled: false };
    }
    if (this.running.has(name)) {
      return this.running.get(name);
    }
    const variant = this.getVariant(name);
    const promise = (async () => {
      try {
        const module = variant === 'control'
          ? await import(`./features/${name}/v1.js`)
          : await import(`./features/${name}/v2.js`);
        const instance = module.default || module;
        return { enabled: true, instance, variant };
      } catch (err) {
        console.error(`Failed to load feature ${name}:`, err);
        return { enabled: false, error: err };
      }
    })();
    this.running.set(name, promise);
    return promise;
  }

  async runFeature(name, ...args) {
    const result = await this.loadFeature(name);
    if (result.enabled && result.instance.run) {
      return result.instance.run(...args);
    }
  }
}
```

**Challenge 3: Create a code-split data visualization dashboard**
```javascript
// Build a dashboard that loads visualization modules on demand.
// Each chart type is a separate chunk. Support adding new charts.

class Dashboard {
  constructor(root) {
    this.root = root;
    this.widgets = new Map();
    this.registry = new Map();
  }

  register(type, loader) {
    this.registry.set(type, loader);
  }

  async addWidget(type, config) {
    if (this.widgets.has(config.id)) {
      throw new Error(`Widget ${config.id} already exists`);
    }
    const loader = this.registry.get(type);
    if (!loader) throw new Error(`Unknown widget type: ${type}`);

    const el = document.createElement('div');
    el.className = 'widget loading';
    el.innerHTML = '<div class="spinner"></div>';
    this.root.appendChild(el);

    try {
      const module = await loader();
      const Widget = module.default || module;
      const widget = new Widget(el, config);
      this.widgets.set(config.id, { type, widget, el, config });
      el.classList.remove('loading');
      return widget;
    } catch (err) {
      el.innerHTML = `<div class="error">Failed: ${err.message}</div>`;
      el.classList.add('error');
      throw err;
    }
  }

  async removeWidget(id) {
    const entry = this.widgets.get(id);
    if (!entry) return;
    if (entry.widget.destroy) entry.widget.destroy();
    entry.el.remove();
    this.widgets.delete(id);
  }

  async refreshAll() {
    for (const [id, entry] of this.widgets) {
      await this.removeWidget(id);
      await this.addWidget(entry.type, entry.config);
    }
  }
}
```

## Image Lazy Loading

### What It Is

Native lazy loading uses the loading="lazy" attribute on img and iframe elements. The browser defers loading until the element is near the viewport.

### Syntax

```html
<img src="image.jpg" loading="lazy" alt="description" />
<iframe src="video.html" loading="lazy"></iframe>
```

### Examples

```javascript
// Progressive image loading (blur-up technique)
class ProgressiveImage {
  constructor(imgElement) {
    this.img = imgElement;
    this.setup();
  }

  setup() {
    const fullSrc = this.img.dataset.src;
    const thumbSrc = this.img.dataset.thumb;

    // Show tiny thumbnail first
    if (thumbSrc) {
      this.img.src = thumbSrc;
      this.img.style.filter = 'blur(10px)';
    }

    // Load full image
    const fullImage = new Image();
    fullImage.onload = () => {
      this.img.src = fullSrc;
      this.img.style.filter = 'none';
      this.img.classList.add('loaded');
    };
    fullImage.src = fullSrc;
  }
}

// IntersectionObserver + progressive loading
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      new ProgressiveImage(entry.target);
      observer.unobserve(entry.target);
    }
  });
}, { rootMargin: '200px' });

document.querySelectorAll('img[data-src]').forEach(img => observer.observe(img));
```

### Real-World Use Cases

- **E-commerce product images**: Lazy load below-the-fold product images.
- **Social media feeds**: Infinite scroll with IntersectionObserver triggers.
- **Single-page applications**: Dynamic imports for route-based code splitting.
- **Media galleries**: Lazy load images/videos as user scrolls.
- **Dashboard widgets**: Lazy load widgets that are below the fold.
- **Chart libraries**: Load heavy charting libs only when chart section is visible.

### Common Mistakes

```javascript
// Mistake: Not providing dimensions (causes layout shift)
<img data-src="image.jpg" /> <!-- No width/height! -->
// Fix: Always set width and height attributes

// Mistake: Lazy loading above-the-fold content
// First-visible content should load eagerly (no lazy)

// Mistake: IntersectionObserver with rootMargin too large
// 1000px rootMargin defeats the purpose of lazy loading

// Mistake: Not unobserving after load
// Observer keeps firing if you don't unobserve

// Mistake: Dynamic importing synchronously
const module = await import('./heavy.js'); // Don't block render!
// Fix: Import in response to user action, not during load
```

### Best Practices

```javascript
// Use native loading='lazy' for simple cases
<img src="photo.jpg" loading="lazy" width="800" height="600" alt="">

// Use IntersectionObserver for custom behavior
const observer = new IntersectionObserver((entries) => {
  entries.forEach(e => {
    if (e.isIntersecting) {
      // Custom load logic
      observer.unobserve(e.target);
    }
  });
}, { rootMargin: '200px', threshold: 0.01 });

// Combine for progressive enhancement
if ('loading' in HTMLImageElement.prototype) {
  img.loading = 'lazy';
} else {
  observer.observe(img);
}

// Placeholder techniques
// 1. Low-quality image placeholder (LQIP) - tiny blurry image
// 2. Dominant color - solid color background
// 3. SVG skeleton - outline shape
// 4. BlurHash - compact representation of image colors
```

### Coding Challenges

**Challenge 1: Implement responsive image selection with device detection**
```javascript
// Build a responsive lazy loader that picks the optimal image size
// based on viewport width, device pixel ratio, and network speed.

class ResponsiveImageLoader {
  constructor(options = {}) {
    this.options = {
      container: document,
      ...options
    };
    this.observer = new IntersectionObserver(this.onIntersect.bind(this), {
      rootMargin: '200px'
    });
    this.init();
  }

  init() {
    const images = this.options.container.querySelectorAll('img[data-srcset]');
    images.forEach(img => {
      img.dataset.dpr = window.devicePixelRatio || 1;
      if (navigator.connection) {
        img.dataset.effectiveType = navigator.connection.effectiveType;
      }
      this.observer.observe(img);
    });
  }

  onIntersect(entries) {
    entries.forEach(entry => {
      if (!entry.isIntersecting) return;
      const img = entry.target;
      this.observer.unobserve(img);
      const sources = JSON.parse(img.dataset.srcset);
      const dpr = parseFloat(img.dataset.dpr);
      const viewportWidth = window.innerWidth;
      const connection = img.dataset.effectiveType;
      const maxWidth = connection && connection.includes('2g') ? viewportWidth * 0.5 : viewportWidth;
      const neededWidth = Math.ceil(maxWidth * dpr);
      const best = sources
        .filter(s => s.width >= neededWidth || s === sources[sources.length - 1])
        .sort((a, b) => a.width - b.width)[0];
      img.src = best.url;
      img.srcset = sources.map(s => `${s.url} ${s.width}w`).join(', ');
      img.sizes = `${Math.ceil(viewportWidth)}px`;
    });
  }

  destroy() { this.observer.disconnect(); }
}
```

**Challenge 2: Build a blurred placeholder generator**
```javascript
// Implement a progressive image loader that generates and shows
// a blurred, low-resolution placeholder before loading the full image.

class BlurHashLoader {
  constructor(container) {
    this.container = container;
    this.observer = new IntersectionObserver(this.onIntersect.bind(this), {
      rootMargin: '150px'
    });
    this.process();
  }

  process() {
    this.container.querySelectorAll('img[data-blurhash]').forEach(img => {
      const canvas = document.createElement('canvas');
      canvas.width = 32;
      canvas.height = 32;
      canvas.className = 'blur-placeholder';
      canvas.style.position = 'absolute';
      canvas.style.top = '0';
      canvas.style.left = '0';
      canvas.style.width = '100%';
      canvas.style.height = '100%';
      canvas.style.filter = 'blur(20px)';
      canvas.style.transform = 'scale(1.1)';
      img.style.position = 'relative';
      img.parentNode.style.position = 'relative';
      img.parentNode.insertBefore(canvas, img);
      this.decodeBlurHash(img.dataset.blurhash, canvas);
      this.observer.observe(img);
    });
  }

  decodeBlurHash(hash, canvas) {
    // Simplified blurhash decoding - real implementation uses
    // the blurhash npm package to decode to pixels
    const ctx = canvas.getContext('2d');
    const imageData = ctx.createImageData(32, 32);
    for (let i = 0; i < imageData.data.length; i += 4) {
      imageData.data[i] = 128;
      imageData.data[i + 1] = 128;
      imageData.data[i + 2] = 128;
      imageData.data[i + 3] = 255;
    }
    ctx.putImageData(imageData, 0, 0);
  }

  onIntersect(entries) {
    entries.forEach(entry => {
      if (!entry.isIntersecting) return;
      const img = entry.target;
      this.observer.unobserve(img);
      const temp = new Image();
      temp.onload = () => {
        img.src = img.dataset.src;
        img.onload = () => {
          const placeholder = img.parentNode.querySelector('.blur-placeholder');
          if (placeholder) {
            placeholder.style.transition = 'opacity 0.5s';
            placeholder.style.opacity = '0';
            setTimeout(() => placeholder.remove(), 500);
          }
        };
      };
      temp.src = img.dataset.src;
    });
  }

  destroy() { this.observer.disconnect(); }
}
```

**Challenge 3: Create an intersection-aware video player**
```javascript
// Build a video player that automatically plays/pauses based on visibility.
// Use IntersectionObserver to detect when the video enters/exits the viewport.
// Support muted autoplay, progress tracking, and lazy source loading.

class VisibilityVideoPlayer {
  constructor(videoElements) {
    this.videos = new Map();
    this.observer = new IntersectionObserver(this.onVisibilityChange.bind(this), {
      threshold: [0, 0.25, 0.5, 0.75, 1]
    });
    videoElements.forEach(video => this.setupVideo(video));
  }

  setupVideo(video) {
    video.controls = true;
    video.preload = 'none';
    const state = {
      element: video,
      wasPlaying: false,
      visibility: 0,
      playTime: 0,
      observer: new IntersectionObserver(([entry]) => {
        state.visibility = entry.intersectionRatio;
      }, { threshold: [0, 0.5] })
    };
    state.observer.observe(video);
    video.dataset.src = video.querySelector('source')?.src || video.src;
    video.removeAttribute('src');
    video.load();
    this.videos.set(video, state);
    this.observer.observe(video);
  }

  onVisibilityChange(entries) {
    entries.forEach(entry => {
      const state = this.videos.get(entry.target);
      if (!state) return;
      if (entry.isIntersecting && entry.intersectionRatio > 0.5) {
        if (!state.element.dataset.loaded) {
          state.element.src = state.element.dataset.src;
          state.element.load();
          state.element.dataset.loaded = 'true';
        }
        state.element.muted = true;
        state.element.play().catch(() => {});
        state.element.dataset.visible = 'true';
      } else {
        if (state.element.dataset.visible) {
          state.element.pause();
          delete state.element.dataset.visible;
        }
      }
    });
  }

  getStats() {
    const stats = {};
    for (const [video, state] of this.videos) {
      stats[video.id || 'unknown'] = {
        visibility: state.visibility,
        loaded: !!video.dataset.loaded,
        paused: video.paused,
        currentTime: video.currentTime
      };
    }
    return stats;
  }

  destroy() {
    this.observer.disconnect();
    for (const [, state] of this.videos) {
      state.observer.disconnect();
    }
    this.videos.clear();
  }
}
```

### Performance Considerations

- Too many IntersectionObserver instances can slow down the page. Batch observations.
- rootMargin of 200-500px provides a good balance (loads before user reaches element).
- Native loading='lazy' has ~300px rootMargin default in Chrome.
- Dynamic imports add network request latency; preload critical chunks.
- Image lazy loading can cause layout shifts if dimensions aren't set.

### Interview Questions

**Q: How does IntersectionObserver differ from scroll event listeners?**
A: IntersectionObserver runs asynchronously off the main thread, provides precise intersection data (thresholds, ratios), and doesn't require scroll event handlers. Scroll-based detection requires getBoundingClientRect calls on the main thread, which can cause jank.

**Q: What is the loading attribute and how does it work?**
A: loading='lazy' tells the browser to defer loading off-screen images/iframes until the user scrolls near them. The browser has built-in prioritization. loading='eager' loads immediately. This is supported in modern Chrome, Firefox, and Safari.

**Q: How would you lazy load a JavaScript library based on user interaction?**
A: Use dynamic import() triggered by the interaction:
button.addEventListener('click', async () => {
  const { editor } = await import('./rich-editor.js');
  editor.init(targetElement);
});
The editor code is only downloaded when the user clicks the button.

### Related Topics

- Code splitting, Dynamic imports, Web performance, Critical rendering path
