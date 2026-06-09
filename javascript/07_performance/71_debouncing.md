# Debouncing - Debounce implementation, leading/trailing, immediate invocation

## Introduction

Debouncing ensures a function is called only after a specified delay since the last invocation. It groups rapid successive calls into a single execution. Essential for handling user input, search-as-you-type, and resize events.

## Debounce Implementation

### What It Is

A debounced function delays execution until after a quiet period. Each new call resets the timer. Only the last call in a burst actually executes.

### How It Works Internally

Each call clears the previous timer and sets a new one. The timer fires only after the delay passes without another call.

### Syntax

```javascript
function debounce(fn, delay) {
  let timer = null;
  return function(...args) {
    const context = this;
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(context, args), delay);
  };
}
```

### Beginner Examples

```javascript
// Search input
const searchInput = document.getElementById('search');

function performSearch(query) {
  console.log('Searching:', query);
  // API call: fetch('/api/search?q=' + query)
}

const debouncedSearch = debounce(performSearch, 300);

searchInput.addEventListener('input', (e) => {
  debouncedSearch(e.target.value);
});
// Only calls performSearch 300ms after user stops typing

// Window resize
function handleResize() {
  console.log('Resized:', window.innerWidth, 'x', window.innerHeight);
}
const debouncedResize = debounce(handleResize, 200);
window.addEventListener('resize', debouncedResize);
```

### Intermediate Examples

```javascript
// Debounce with cancel and flush
function debounce(fn, delay) {
  let timer = null;
  let lastArgs = null;
  let lastContext = null;

  const debounced = function(...args) {
    lastArgs = args;
    lastContext = this;
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(lastContext, lastArgs);
      timer = null;
    }, delay);
  };

  debounced.cancel = function() {
    clearTimeout(timer);
    timer = null;
    lastArgs = null;
    lastContext = null;
  };

  debounced.flush = function() {
    if (timer) {
      clearTimeout(timer);
      fn.apply(lastContext, lastArgs);
      timer = null;
    }
  };

  return debounced;
}

// Usage: cancel pending execution
const debouncedSave = debounce(saveToDatabase, 2000);
debouncedSave(data);
// User navigates away - cancel the save
debouncedSave.cancel();

// Flush: execute immediately
form.addEventListener('submit', (e) => {
  e.preventDefault();
  debouncedSave.flush(); // Execute pending if any
});

// Auto-save in a text editor
class Editor {
  constructor() {
    this.content = '';
    this.autoSave = debounce(this.save.bind(this), 1000);
    this.modified = false;
  }
  updateContent(text) {
    this.content = text;
    this.modified = true;
    this.autoSave();
  }
  async save() {
    if (!this.modified) return;
    await fetch('/api/save', { method: 'POST', body: this.content });
    this.modified = false;
  }
  destroy() {
    this.autoSave.flush(); // Save pending changes
    this.autoSave.cancel();
  }
}
```

### Advanced Examples

```javascript
// Debounce with async function support
function debounceAsync(fn, delay) {
  let timer = null;
  let pendingResolve = null;

  const debounced = function(...args) {
    const context = this;

    if (timer) clearTimeout(timer);

    return new Promise((resolve, reject) => {
      pendingResolve = resolve;
      timer = setTimeout(async () => {
        try {
          const result = await fn.apply(context, args);
          resolve(result);
        } catch (err) {
          reject(err);
        }
        timer = null;
        pendingResolve = null;
      }, delay);
    });
  };

  debounced.cancel = function() {
    clearTimeout(timer);
    timer = null;
    if (pendingResolve) {
      pendingResolve('cancelled');
      pendingResolve = null;
    }
  };

  return debounced;
}

// Debounce with maxWait (guarantee execution within maxWait)
function debounceMaxWait(fn, delay, maxWait) {
  let timer = null;
  let lastCall = 0;
  let lastArgs = null;
  let lastContext = null;

  const debounced = function(...args) {
    lastArgs = args;
    lastContext = this;
    const now = Date.now();

    if (!timer) {
      lastCall = now;
    }

    clearTimeout(timer);

    if (maxWait && (now - lastCall >= maxWait)) {
      fn.apply(lastContext, lastArgs);
      lastCall = now;
      timer = setTimeout(() => {
        timer = null;
      }, delay);
    } else {
      timer = setTimeout(() => {
        fn.apply(lastContext, lastArgs);
        timer = null;
      }, delay);
    }
  };

  return debounced;
}

// Usage: progress report that must fire at least every 5 seconds
const reportProgress = debounceMaxWait(sendProgress, 1000, 5000);
```

## Leading and Trailing Options

### What It Is

Leading execution fires on the first call, trailing fires after the quiet period. Both can be combined: first call fires immediately, last call fires after delay.

### Syntax

```javascript
function debounce(fn, delay, options = {}) {
  const { leading = false, trailing = true } = options;
  let timer = null;
  let lastArgs = null;

  const debounced = function(...args) {
    lastArgs = args;
    const isInvoked = timer === null;

    if (isInvoked && leading) {
      fn.apply(this, args);
    }

    clearTimeout(timer);
    timer = setTimeout(() => {
      if (trailing && !(leading && isInvoked)) {
        fn.apply(this, lastArgs);
      }
      timer = null;
    }, delay);
  };

  return debounced;
}

// Leading: fires immediately, then ignores rapid calls
const saveOnClick = debounce(save, 1000, { leading: true, trailing: false });

// Trailing (default): fires after quiet period
const search = debounce(searchAPI, 300, { leading: false, trailing: true });

// Both: fires immediately and after quiet period
const logActivity = debounce(log, 5000, { leading: true, trailing: true });
// First call logs immediately, last call in burst logs after 5s
```

### Examples

```javascript
// Leading: prevent button double-click
const submit = debounce(() => {
  console.log('Submitting...');
}, 1000, { leading: true, trailing: false });

button.addEventListener('click', submit);
// First click fires immediately; subsequent clicks ignored for 1s

// Trailing: search-as-you-type
const search = debounce((query) => {
  fetch('/api/search?q=' + query);
}, 300);
// Fires 300ms after user stops typing

// Both: game save (immediate feedback + eventual save)
const saveGame = debounce(() => {
  localStorage.setItem('save', JSON.stringify(gameState));
}, 5000, { leading: true, trailing: true });
// Shows 'Saving...' immediately, performs actual save after 5s quiet
```

## Immediate Invocation

### What It Is

Immediate invocation (leading edge) fires the function on the first call in a burst, then prevents further calls for the delay period. Equivalent to leading=true, trailing=false.

### Examples

```javascript
// Classic immediate debounce (lodash _.debounce with leading=true)
function debounceImmediate(fn, delay) {
  let timer = null;
  let ready = true;

  return function(...args) {
    if (ready) {
      fn.apply(this, args);
      ready = false;
    }

    clearTimeout(timer);
    timer = setTimeout(() => {
      ready = true;
    }, delay);
  };
}

// Immediate: API rate limiting
const callAPI = debounceImmediate(() => {
  console.log('API call at', Date.now());
}, 1000);

callAPI(); // Fires immediately
callAPI(); // Ignored
callAPI(); // Ignored
// After 1s: ready to fire again
```

### Real-World Use Cases

- **Search autocomplete**: Trailing debounce, 300-500ms delay
- **Button double-click prevention**: Leading debounce, 1000ms delay
- **Window resize handlers**: Trailing debounce, 200ms delay
- **Auto-save forms**: Trailing debounce, 2000ms delay
- **Rate-limited API calls**: Leading debounce with immediate execution
- **Scroll position tracking**: Leading debounce, 100ms delay
- **Payment submission**: Leading debounce with both edges

### Common Mistakes

```javascript
// Mistake: Not preserving this context
input.addEventListener('input', debounce(function(e) {
  console.log(this.value); // 'this' is window, not input!
}, 300));
// Fix: Use function (not arrow) in debounce wrapper with fn.apply

// Mistake: Debouncing with zero delay
const bad = debounce(fn, 0); // Still uses setTimeout(fn, 0)
// Use synchronous throttling instead

// Mistake: Creating debounced function inside render loop
function Component() {
  // New debounced function every render!
  const handleSearch = debounce(search, 300);
}
// Fix: Create once (useRef or module level)

// Mistake: Not handling async errors
const debouncedFetch = debounce(async () => {
  const res = await fetch(url);
  // Unhandled promise rejection if fails
}, 1000);
```

### Best Practices

```javascript
// Store debounced function reference (don't recreate)
const search = debounce(handleSearch, 300);

// Cancel on unmount/cleanup
useEffect(() => {
  return () => search.cancel();
}, []);

// Use maxWait for guaranteed execution
const update = debounceMaxWait(fn, 1000, 5000);

// Type the debounced function
/** @param {(...args) => void} fn */
function debounce(fn, delay) { /* ... */ }
```

### Performance Considerations

- setTimeout has 4ms minimum clamping in browsers.
- Each debounced call allocates a timer object; frequent calls create timer management overhead.
- requestAnimationFrame can be better for visual updates (60fps sync).
- Node.js: setImmediate for 0-delay debouncing (vs setTimeout(fn, 0)).
- Debouncing adds latency (delay amount); balance responsiveness with performance.

### Interview Questions

**Q: What is the difference between debounce and throttle?**
A: Debounce delays execution until after a quiet period. Throttle ensures at most one execution per interval. Debounce groups a burst into one call; throttle spreads calls evenly over time.

**Q: How would you implement a debounce that fires on both leading and trailing edges?**
A: Set leading=true and trailing=true. On first call, execute immediately and start timer. On subsequent calls, reset timer. When timer fires, execute with last arguments. The leading call gives immediate feedback; the trailing call captures the final state.

**Q: Why would you use debounce with a maxWait option?**
A: maxWait guarantees execution within a maximum interval, even if calls keep coming. This prevents the function from never executing (starvation) when calls are continuous. Useful for progress reports, real-time updates, and auto-save scenarios.

### Coding Challenges

```javascript
// Challenge 1: Implement a debounce with leading and trailing options,
// plus a maxWait option. Test with rapid calls.

// Challenge 2: Implement a debounced function that calls Promise
// and returns a Promise that resolves when the debounced call executes.

// Challenge 3: Implement a debounced input that calls an API
// but cancels the previous request if a new one comes in.
```

### Related Topics
- Throttling, requestAnimationFrame, Event loop timing, Async patterns
