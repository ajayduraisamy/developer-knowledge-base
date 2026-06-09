# Throttling - Throttle implementation, leading/trailing, requestAnimationFrame

## Introduction

Throttling ensures a function is called at most once in a specified time interval. Unlike debouncing (which waits for quiet), throttling guarantees regular execution. Essential for scroll handlers, resize events, and rate-limited APIs.

## Throttle Implementation

### What It Is

A throttled function executes at most once per interval. If called multiple times during the interval, only the first call executes; subsequent calls are ignored until the next interval.

### How It Works Internally

A timestamp-based or timer-based gate controls execution. When the function is called, if enough time has passed since last execution, it runs. Otherwise, the call is either ignored or scheduled for the next interval.

### Syntax

```javascript
// Timer-based throttle
function throttle(fn, interval) {
  let timer = null;
  return function(...args) {
    if (timer) return; // Already waiting
    timer = setTimeout(() => {
      fn.apply(this, args);
      timer = null;
    }, interval);
  };
}

// Timestamp-based throttle
function throttle(fn, interval) {
  let lastCall = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= interval) {
      lastCall = now;
      fn.apply(this, args);
    }
  };
}
```

### Beginner Examples

```javascript
// Scroll handler
function handleScroll() {
  console.log('Scroll position:', window.scrollY);
  // Check if near bottom, lazy load images, etc.
}
const throttledScroll = throttle(handleScroll, 200);
window.addEventListener('scroll', throttledScroll);
// At most every 200ms, regardless of scroll speed

// Button to prevent spam clicks
const throttledClick = throttle(() => {
  console.log('Button clicked at', Date.now());
}, 1000);
button.addEventListener('click', throttledClick);
// Max one click per second

// Mouse move tracking
canvas.addEventListener('mousemove', throttle((e) => {
  drawDot(e.clientX, e.clientY);
}, 50));
// Sample mouse position at ~20fps
```

### Intermediate Examples

```javascript
// Full throttle with leading and trailing options
function throttle(fn, interval, options = {}) {
  const { leading = true, trailing = true } = options;
  let timer = null;
  let lastArgs = null;
  let lastContext = null;
  let lastCall = 0;

  const throttled = function(...args) {
    const now = Date.now();
    const remaining = interval - (now - lastCall);

    lastArgs = args;
    lastContext = this;

    if (remaining <= 0 || remaining > interval) {
      // Enough time has passed
      if (timer) {
        clearTimeout(timer);
        timer = null;
      }
      lastCall = now;
      fn.apply(this, args);
    } else if (!timer && trailing) {
      // Schedule for end of interval
      timer = setTimeout(() => {
        lastCall = leading ? Date.now() : 0;
        timer = null;
        if (trailing) {
          fn.apply(lastContext, lastArgs);
        }
      }, remaining);
    }
  };

  throttled.cancel = function() {
    clearTimeout(timer);
    timer = null;
    lastArgs = null;
    lastContext = null;
    lastCall = 0;
  };

  return throttled;
}

// Usage: scroll position (trailing ensures final position is captured)
const trackScroll = throttle(reportScroll, 500, { leading: true, trailing: true });

// Usage: resize (only leading, no trailing)
const handleResize = throttle(layout, 300, { leading: true, trailing: false });
```

### Advanced Examples

```javascript
// Throttle with maxWait to prevent starvation
function throttleMaxWait(fn, interval, maxWait) {
  let timer = null;
  let lastCall = 0;
  let lastArgs = null;
  let lastContext = null;

  const throttled = function(...args) {
    const now = Date.now();
    const elapsed = now - lastCall;

    lastArgs = args;
    lastContext = this;

    if (elapsed >= interval) {
      // Enough time passed - execute
      if (timer) { clearTimeout(timer); timer = null; }
      lastCall = now;
      fn.apply(this, args);
    } else if (!timer) {
      const remaining = interval - elapsed;
      const wait = maxWait ? Math.min(remaining, maxWait - elapsed) : remaining;
      timer = setTimeout(() => {
        lastCall = Date.now();
        timer = null;
        fn.apply(lastContext, lastArgs);
      }, wait);
    }
  };

  return throttled;
}

// Adaptive throttle (adjusts interval based on frequency)
function adaptiveThrottle(fn, baseInterval, maxInterval) {
  let lastCall = 0;
  let timer = null;
  let callCount = 0;

  return function(...args) {
    callCount++;
    const now = Date.now();
    const rate = callCount / ((now - lastCall) || 1) * 1000;
    const interval = Math.min(baseInterval * rate, maxInterval);

    if (now - lastCall >= interval) {
      lastCall = now;
      callCount = 0;
      fn.apply(this, args);
    }
  };
}

// Throttle wrapper for class methods
function throttleMethod(interval) {
  return function(target, key, descriptor) {
    const original = descriptor.value;
    let lastCall = 0;

    descriptor.value = function(...args) {
      const now = Date.now();
      if (now - lastCall >= interval) {
        lastCall = now;
        return original.apply(this, args);
      }
    };

    return descriptor;
  };
}

class Analytics {
  @throttleMethod(1000)
  track(event) {
    console.log('Tracking:', event);
  }
}
```

## Leading and Trailing Execution

### What It Is

Leading execution fires at the start of each interval. Trailing execution captures the last call in an interval and fires at the end.

### How It Works

- **leading=true, trailing=false**: First call per interval executes immediately; subsequent calls ignored until next interval.

- **leading=false, trailing=true**: Calls are delayed until the end of interval. Only the last call in each interval executes.

- **leading=true, trailing=true**: First call executes immediately; last call in interval executes at the end. Full coverage.

### Examples

```javascript
const log = (msg) => console.log(new Date().toISOString().slice(11,19), msg);

// Leading only - game shooting
const shoot = throttle(() => log('PEW!'), 500, { leading: true, trailing: false });
// shoot(); shoot(); shoot(); -> 'PEW!' once (first call)

// Trailing only - API rate limit
const callAPI = throttle(() => log('API'), 1000, { leading: false, trailing: true });
// callAPI(); callAPI(); -> 'API' once (after 1s)

// Both - scroll handler needs immediate + final
const onScroll = throttle(() => log('scroll'), 500, { leading: true, trailing: true });
```

## requestAnimationFrame Throttling

### What It Is

Using requestAnimationFrame (rAF) to throttle to the browser's refresh rate (typically 60fps). rAF pauses when the tab is inactive, saving resources.

### Syntax

```javascript
function throttleRAF(fn) {
  let scheduled = false;
  return function(...args) {
    if (scheduled) return;
    scheduled = true;
    requestAnimationFrame(() => {
      scheduled = false;
      fn.apply(this, args);
    });
  };
}

// Usage: smooth scroll animations
const updatePosition = throttleRAF((e) => {
  element.style.transform = 'translate(' + e.clientX + 'px, ' + e.clientY + 'px)';
});
document.addEventListener('mousemove', updatePosition);
```

### Examples

```javascript
// RAF throttled resize handler
const onResize = throttleRAF(() => {
  // Expensive layout calculations
  recalculateLayout();
});
window.addEventListener('resize', onResize);

// RAF for smooth animations
function throttleRAF(fn) {
  let id = null;
  let lastArgs = null;
  let lastContext = null;

  const throttled = function(...args) {
    lastArgs = args;
    lastContext = this;
    if (id === null) {
      id = requestAnimationFrame(() => {
        fn.apply(lastContext, lastArgs);
        id = null;
      });
    }
  };

  throttled.cancel = () => {
    if (id) cancelAnimationFrame(id);
    id = null;
  };

  return throttled;
}

// Comparison: RAF vs setTimeout
const rafThrottle = throttleRAF(updateVisual);
const timeoutThrottle = throttle(updateVisual, 16); // ~60fps
// RAF is better: syncs with vsync, pauses when hidden
```

### Real-World Use Cases

- **Scroll handlers**: 100-200ms throttle for performance
- **Resize handlers**: 200-300ms throttle or RAF
- **Mouse move tracking**: 50ms throttle or RAF for smooth visual
- **API rate limiting**: 1000ms+ throttle to respect rate limits
- **Game input handling**: RAF throttle for smooth controls
- **Progress bar updates**: 100-500ms throttle to avoid excessive DOM writes

### Common Mistakes

```javascript
// Mistake: Throttling with requestAnimationFrame for non-visual tasks
const checkAuth = throttleRAF(checkToken); // RAF pauses when tab hidden!
// Fix: Use setTimeout-based throttle for non-visual operations

// Mistake: Higher interval than needed (adds unnecessary delay)
const search = throttle(searchAPI, 2000); // User waits 2s after types
// Fix: 300-500ms for search

// Mistake: Creating throttle inside render
function List() {
  const handleScroll = throttle(onScroll, 200); // New function each render!
}
// Fix: Create once with useRef/useMemo or module-level
```

### Best Practices

```javascript
// Use RAF for visual updates (animations, scroll effects)
// Use setTimeout for non-visual (API calls, logging, analytics)

// Always provide cancel method
throttled.cancel();

// Test with rapid events
const handler = throttle(fn, 100);
for (let i = 0; i < 100; i++) handler(); // Only ~10 executions

// Document the throttle interval
/** Throttled to prevent excessive API calls (max 1 per second) */
const checkInventory = throttle(fetchStock, 1000);

// Combine with debounce for complex scenarios
const optimized = pipe(throttle(fn, 100), debounce(fn, 500));
```

### Performance Considerations

- Throttle between 16ms (60fps) and 200ms depending on use case.
- RAF-throttled functions don't run in background tabs (saves battery).
- Timestamps are cheaper than timers (no setTimeout overhead).
- Timer-based throttle has minimum ~4ms delay (HTML5 spec).
- RAF fires approximately every 16.67ms (60fps).

### Interview Questions

**Q: What is the difference between throttle and debounce?**
A: Throttle ensures at most one call per interval (rate limiting). Debounce waits for a quiet period before executing (grouping). Use throttle for continuous events (scroll), debounce for discrete bursts (typing).

**Q: When would you use RAF throttle instead of setTimeout?**
A: RAF throttle for visual/rendering updates that need to sync with the browser's paint cycle. setTimeout for non-visual work like API calls, logging, or data processing. RAF also pauses in background tabs, saving CPU/battery.

**Q: How does leading vs trailing throttle affect user experience?**
A: Leading gives immediate response (good for button clicks). Trailing captures the final state (good for scroll position). Both gives coverage but may execute twice per interval.

### Coding Challenges

```javascript
// Challenge 1: Implement throttle that executes on both leading
// and trailing edges, with support for cancel() and flush().

// Challenge 2: Create a throttle decorator for class methods
// that accepts interval in milliseconds.

// Challenge 3: Implement an adaptive throttle that measures
// incoming call frequency and adjusts the interval dynamically.
```

### Related Topics
- Debouncing, requestAnimationFrame, Event loop, Performance optimization
