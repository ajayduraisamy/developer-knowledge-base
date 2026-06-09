# Local Storage - localStorage API, setItem, getItem, removeItem, storage limits

## Introduction

The `localStorage` API is a web storage mechanism that allows JavaScript to store key-value pairs persistently in the browser. Data stored in `localStorage` has no expiration time and persists even after the browser is closed and reopened. It provides a simple synchronous interface for storing up to 5-10 MB of data per origin, making it suitable for caching application state, user preferences, and offline data.

Unlike cookies, `localStorage` data is not automatically sent to the server with HTTP requests, making it more efficient for client-side storage. It is part of the Web Storage API, which also includes `sessionStorage`. Both share the same interface but differ in scope and persistence.

## localStorage API

### What It Is

The `localStorage` API is a key-value storage system available on the `window` object. It stores strings only—all non-string values are automatically converted to strings when stored. The data is organized by origin (scheme + hostname + port), so `https://example.com` and `https://app.example.com` have separate storage.

```javascript
// Store data
localStorage.setItem('theme', 'dark');
localStorage.setItem('user', JSON.stringify({ id: 1, name: 'John' }));

// Retrieve data
const theme = localStorage.getItem('theme'); // 'dark'
const user = JSON.parse(localStorage.getItem('user')); // { id: 1, name: 'John' }

// Remove data
localStorage.removeItem('theme');

// Clear all
localStorage.clear();
```

### Why It Is Important

`localStorage` provides persistent client-side storage with a simple synchronous API. It is essential for saving user preferences (theme, language, layout), caching API responses for offline use, maintaining application state across sessions, and implementing features like draft saving and client-side configuration.

### How It Works Internally

Each browser implements `localStorage` using an internal storage engine. In Chromium, it uses a LevelDB database. In Firefox, it uses SQLite. The data is stored on disk and loaded into memory when the page's origin is first accessed. Each origin gets its own storage area, and the browser enforces quota limits per origin.

```javascript
// The storage key is the origin:
// https://example.com:443 -> separate storage area
// https://api.example.com:443 -> different origin, different storage
// http://example.com:443 -> different scheme, different storage

// Internal operations are synchronous but may involve disk I/O
// The API is synchronous because it works on an in-memory cache
// Changes are flushed to disk asynchronously
```

### Syntax

```javascript
// All localStorage methods
localStorage.setItem(key, value);     // Store a value (value is converted to string)
localStorage.getItem(key);            // Retrieve a value (or null if not found)
localStorage.removeItem(key);         // Remove a specific key
localStorage.clear();                 // Remove all keys for this origin
localStorage.key(index);              // Get the key at the specified index
localStorage.length;                  // Number of stored items

// Property access (alternative to getItem/setItem)
localStorage.theme = 'dark';          // Same as setItem('theme', 'dark')
const theme = localStorage.theme;     // Same as getItem('theme')

// Check if a key exists
localStorage.hasOwnProperty('theme'); // true/false
'theme' in localStorage;              // true/false
```

### Beginner Examples

```javascript
// Example 1: Theme preference
function setTheme(theme) {
  localStorage.setItem('theme', theme);
  document.documentElement.className = `theme-${theme}`;
}

function loadTheme() {
  const saved = localStorage.getItem('theme');
  if (saved) {
    document.documentElement.className = `theme-${saved}`;
  }
}

loadTheme();

// Example 2: Welcome message (first visit detection)
const visitCount = localStorage.getItem('visitCount');
if (visitCount === null) {
  console.log('Welcome first-time visitor!');
  localStorage.setItem('visitCount', '1');
} else {
  const count = parseInt(visitCount, 10) + 1;
  localStorage.setItem('visitCount', String(count));
  console.log(`Welcome back! Visit #${count}`);
}

// Example 3: Form data persistence
const draftKey = 'contact-form-draft';

document.querySelector('#contact-form').addEventListener('input', () => {
  const formData = {
    name: document.querySelector('#name').value,
    email: document.querySelector('#email').value,
    message: document.querySelector('#message').value
  };
  localStorage.setItem(draftKey, JSON.stringify(formData));
});

// Restore on page load
const saved = localStorage.getItem(draftKey);
if (saved) {
  const { name, email, message } = JSON.parse(saved);
  document.querySelector('#name').value = name || '';
  document.querySelector('#email').value = email || '';
  document.querySelector('#message').value = message || '';
}

// Example 4: Simple counter
function incrementCounter() {
  const count = parseInt(localStorage.getItem('counter') || '0', 10) + 1;
  localStorage.setItem('counter', String(count));
  document.querySelector('#counter').textContent = count;
}

incrementCounter();
```

### Intermediate Examples

```javascript
// Example 1: Local cache with expiry
class LocalCache {
  constructor(prefix = 'cache_') {
    this.prefix = prefix;
  }

  set(key, value, ttlMinutes = 60) {
    const item = {
      value,
      expiry: Date.now() + ttlMinutes * 60 * 1000
    };
    localStorage.setItem(this.prefix + key, JSON.stringify(item));
  }

  get(key) {
    const raw = localStorage.getItem(this.prefix + key);
    if (!raw) return null;

    try {
      const item = JSON.parse(raw);
      if (Date.now() > item.expiry) {
        localStorage.removeItem(this.prefix + key);
        return null;
      }
      return item.value;
    } catch {
      return null;
    }
  }

  remove(key) {
    localStorage.removeItem(this.prefix + key);
  }

  clear() {
    const keys = [];
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      if (key.startsWith(this.prefix)) {
        keys.push(key);
      }
    }
    keys.forEach(k => localStorage.removeItem(k));
  }
}

const cache = new LocalCache('api_');

// Cache API responses
async function fetchWithCache(url, ttlMinutes = 5) {
  const cached = cache.get(url);
  if (cached !== null) return cached;

  const response = await fetch(url);
  const data = await response.json();
  cache.set(url, data, ttlMinutes);
  return data;
}

// Example 2: Storage event listener (cross-tab sync)
window.addEventListener('storage', (event) => {
  console.log(`Storage changed: ${event.key}`);
  console.log(`Old value: ${event.oldValue}`);
  console.log(`New value: ${event.newValue}`);
  console.log(`Origin: ${event.url}`);
  console.log(`Storage area: ${event.storageArea === localStorage ? 'localStorage' : 'sessionStorage'}`);

  // React to changes
  if (event.key === 'theme') {
    document.documentElement.className = `theme-${event.newValue}`;
  }
});

// Example 3: Storage usage monitor
function getStorageUsage() {
  let totalBytes = 0;
  const items = [];

  for (let i = 0; i < localStorage.length; i++) {
    const key = localStorage.key(i);
    const value = localStorage.getItem(key);
    const size = new Blob([key, value]).size;
    totalBytes += size;
    items.push({ key, size });
  }

  return {
    items,
    totalBytes,
    totalKB: (totalBytes / 1024).toFixed(2),
    totalMB: (totalBytes / 1024 / 1024).toFixed(4),
    itemCount: localStorage.length
  };
}

// console.log(getStorageUsage());

// Example 4: Namespaced storage
class NamespacedStorage {
  constructor(namespace) {
    this.namespace = namespace + ':';
  }

  set(key, value) {
    localStorage.setItem(this.namespace + key, JSON.stringify(value));
  }

  get(key) {
    const value = localStorage.getItem(this.namespace + key);
    try {
      return value ? JSON.parse(value) : null;
    } catch {
      return value;
    }
  }

  remove(key) {
    localStorage.removeItem(this.namespace + key);
  }

  keys() {
    const result = [];
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      if (key.startsWith(this.namespace)) {
        result.push(key.slice(this.namespace.length));
      }
    }
    return result;
  }

  clear() {
    this.keys().forEach(key => this.remove(key));
  }

  getAll() {
    const result = {};
    for (const key of this.keys()) {
      result[key] = this.get(key);
    }
    return result;
  }
}

const userStorage = new NamespacedStorage('user');
const appStorage = new NamespacedStorage('app');

userStorage.set('preferences', { theme: 'dark', fontSize: 14 });
appStorage.set('lastPage', '/dashboard');
```

### Advanced Examples

```javascript
// Example 1: Storage-backed state management
class StorageStore {
  constructor(key, initialValue = {}) {
    this.key = key;
    this.listeners = new Set();
    this.state = this.load(initialValue);
  }

  load(defaultValue) {
    try {
      const saved = localStorage.getItem(this.key);
      return saved ? { ...defaultValue, ...JSON.parse(saved) } : defaultValue;
    } catch {
      return defaultValue;
    }
  }

  save() {
    localStorage.setItem(this.key, JSON.stringify(this.state));
  }

  get(path) {
    return path.split('.').reduce((obj, key) => obj?.[key], this.state);
  }

  set(path, value) {
    const keys = path.split('.');
    const lastKey = keys.pop();
    const target = keys.reduce((obj, key) => {
      if (!(key in obj)) obj[key] = {};
      return obj[key];
    }, this.state);
    target[lastKey] = value;
    this.save();
    this.notify(path, value);
  }

  subscribe(listener) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }

  notify(path, value) {
    for (const listener of this.listeners) {
      listener(path, value, this.state);
    }
  }

  reset() {
    localStorage.removeItem(this.key);
    this.state = {};
    this.notify('*reset*', null);
  }
}

const store = new StorageStore('app-state', {
  user: null,
  theme: 'light',
  sidebar: { open: true, width: 250 }
});

store.subscribe((path, value, state) => {
  if (path === 'theme') {
    document.documentElement.className = `theme-${value}`;
  }
});

store.set('theme', 'dark');
store.set('sidebar.width', 300);

// Example 2: Offline queue with localStorage
class OfflineQueue {
  constructor(key = 'offline_queue') {
    this.key = key;
    this.queue = this.load();
    this.processing = false;
  }

  load() {
    try {
      const data = localStorage.getItem(this.key);
      return data ? JSON.parse(data) : [];
    } catch {
      return [];
    }
  }

  save() {
    localStorage.setItem(this.key, JSON.stringify(this.queue));
  }

  enqueue(action) {
    this.queue.push({
      ...action,
      id: Date.now() + Math.random(),
      timestamp: Date.now()
    });
    this.save();
    this.process();
  }

  async process() {
    if (this.processing || this.queue.length === 0) return;
    this.processing = true;

    while (this.queue.length > 0) {
      const action = this.queue[0];
      try {
        const response = await fetch(action.url, {
          method: action.method || 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(action.data)
        });

        if (response.ok) {
          this.queue.shift();
          this.save();
          action.onSuccess?.(await response.json());
        } else {
          // Server error, retry later
          break;
        }
      } catch {
        // Network error, retry later
        break;
      }
    }

    this.processing = false;

    if (this.queue.length > 0) {
      // Retry in 30 seconds
      setTimeout(() => this.process(), 30000);
    }
  }

  get length() {
    return this.queue.length;
  }

  clear() {
    this.queue = [];
    localStorage.removeItem(this.key);
  }
}

const queue = new OfflineQueue();

// Use when online/offline
window.addEventListener('online', () => queue.process());
window.addEventListener('offline', () => {
  console.log('Gone offline - actions will be queued');
});

// Example 3: localStorage wrapper with compression
class CompressedStorage {
  constructor(prefix = 'comp_') {
    this.prefix = prefix;
  }

  set(key, value) {
    try {
      const json = JSON.stringify(value);
      const compressed = this.compress(json);
      localStorage.setItem(this.prefix + key, compressed);
    } catch (e) {
      console.warn('Storage quota exceeded for key:', key);
    }
  }

  get(key) {
    const compressed = localStorage.getItem(this.prefix + key);
    if (!compressed) return null;

    try {
      const json = this.decompress(compressed);
      return JSON.parse(json);
    } catch {
      // Try uncompressed (backward compatibility)
      try {
        return JSON.parse(compressed);
      } catch {
        return null;
      }
    }
  }

  compress(str) {
    // Simple run-length encoding for strings
    let result = '';
    let count = 1;

    for (let i = 0; i < str.length; i++) {
      if (str[i] === str[i + 1]) {
        count++;
      } else {
        if (count > 3) {
          result += count + str[i];
        } else {
          result += str[i].repeat(count);
        }
        count = 1;
      }
    }

    return result;
  }

  decompress(str) {
    let result = '';
    let i = 0;

    while (i < str.length) {
      if (/\d/.test(str[i])) {
        let count = '';
        while (i < str.length && /\d/.test(str[i])) {
          count += str[i];
          i++;
        }
        if (i < str.length) {
          result += str[i].repeat(parseInt(count, 10));
          i++;
        }
      } else {
        result += str[i];
        i++;
      }
    }

    return result;
  }

  remove(key) {
    localStorage.removeItem(this.prefix + key);
  }
}

const compressed = new CompressedStorage();
compressed.set('largeData', Array(1000).fill('test data'));

// Example 4: Transactional storage operations
class TransactionalStorage {
  constructor() {
    this.backup = null;
  }

  begin() {
    // Snapshot current state
    this.backup = {};
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      this.backup[key] = localStorage.getItem(key);
    }
  }

  commit() {
    this.backup = null;
  }

  rollback() {
    if (!this.backup) return;

    // Restore backup
    localStorage.clear();
    for (const [key, value] of Object.entries(this.backup)) {
      localStorage.setItem(key, value);
    }
    this.backup = null;
  }

  setItem(key, value) {
    if (!this.backup) this.begin();
    localStorage.setItem(key, value);
  }

  removeItem(key) {
    if (!this.backup) this.begin();
    localStorage.removeItem(key);
  }
}

const txn = new TransactionalStorage();

try {
  txn.setItem('config1', 'value1');
  txn.setItem('config2', 'value2');
  // If something goes wrong:
  // txn.rollback();
  txn.commit();
} catch (error) {
  txn.rollback();
}
```

### Real-World Use Cases

- **User Preferences**: Theme, language, font size, layout options
- **Session Tokens**: JWT tokens for authentication (with security considerations)
- **Application Cache**: Cached API responses for offline functionality
- **Draft Storage**: Auto-saved form drafts, editor content
- **Shopping Cart**: Persistent cart across sessions (non-authenticated)
- **Game State**: Save game progress locally
- **Feature Flags**: Client-side feature toggles
- **Performance Data**: Cached computations (e.g., search index)
- **A/B Testing**: Stored variant assignments
- **Onboarding State**: Track which onboarding steps a user has completed

### Common Mistakes

```javascript
// Mistake 1: Storing objects without JSON serialization
localStorage.setItem('user', { id: 1 }); // Stores '[object Object]'
localStorage.setItem('user', JSON.stringify({ id: 1 })); // Correct

// Mistake 2: Not handling quota exceeded errors
try {
  localStorage.setItem('key', largeValue);
} catch (e) {
  if (e.name === 'QuotaExceededError' || e.code === 22) {
    console.warn('Storage quota exceeded');
    // Clear old data or notify user
  }
}

// Mistake 3: Assuming synchronous access is safe
// localStorage is synchronous but may cause performance issues
// in the main thread if accessing very large datasets frequently

// Mistake 4: Storing sensitive data
localStorage.setItem('password', 'mySecret123'); // NEVER store passwords/ tokens in plain text

// Mistake 5: Not handling private browsing mode
if (typeof localStorage === 'undefined') {
  console.warn('localStorage not available');
}

// Mistake 6: Forgetting that values are strings
localStorage.setItem('count', 0);
const count = localStorage.getItem('count');
console.log(count + 1); // '01' (string concatenation)
console.log(Number(count) + 1); // 1
```

### Best Practices

- Always serialize objects with `JSON.stringify` and parse with `JSON.parse`
- Handle `QuotaExceededError` gracefully
- Validate and sanitize data when reading from localStorage
- Use namespaced keys to avoid collisions (e.g., `appName_keyName`)
- Never store sensitive data (passwords, tokens, PII) in localStorage
- Check for availability before using (private browsing may block it)
- Implement data expiry for cached items
- Use the `storage` event to sync across tabs
- Limit reads/writes to avoid main thread blocking
- Clear unused data to stay under quota

### Performance Considerations

`localStorage` operations are synchronous and can block the main thread. While individual operations are fast (microseconds), reading or writing large amounts of data can cause noticeable jank. For large datasets, consider using IndexedDB instead.

```javascript
// Benchmark localStorage performance
console.time('localStorage write (1000 items)');
for (let i = 0; i < 1000; i++) {
  localStorage.setItem(`key_${i}`, `value_${i}`);
}
console.timeEnd('localStorage write (1000 items)'); // ~5-15ms

console.time('localStorage read (1000 items)');
for (let i = 0; i < 1000; i++) {
  const value = localStorage.getItem(`key_${i}`);
}
console.timeEnd('localStorage read (1000 items)'); // ~3-10ms
```

### Interview Questions

1. What is the difference between localStorage and cookies?
2. How much data can localStorage store?
3. Is localStorage synchronous or asynchronous?
4. How do you handle the quota exceeded error?
5. Can localStorage be used in service workers?
6. How does localStorage handle different origins?
7. What happens to localStorage data when the browser cache is cleared?
8. How can you observe localStorage changes across tabs?
9. Is localStorage secure for storing sensitive data?
10. How would you implement a TTL (time-to-live) for localStorage items?

### Coding Challenges

1. **Expiring Cache**: Implement a localStorage wrapper that supports TTL for items
2. **Storage Quota Monitor**: Build a utility that tracks and reports storage usage
3. **Offline Queue**: Create an action queue that stores failed API requests and retries them
4. **Cross-Tab Sync**: Build a system that syncs state across browser tabs using the storage event
5. **Persistent State Management**: Implement a simple state management library backed by localStorage

### Related Topics

- [sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [StorageEvent](https://developer.mozilla.org/en-US/docs/Web/API/StorageEvent)
- [Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache)
- [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)

## setItem() and getItem()

### What It Is

`setItem()` stores a value in localStorage under a specified key. `getItem()` retrieves the value associated with a key. Both methods work with strings—non-string values are converted to strings. `getItem()` returns `null` if the key doesn't exist.

```javascript
localStorage.setItem('key', 'value');
const value = localStorage.getItem('key'); // 'value' or null
```

### Why It Is Important

These are the primary methods for interacting with localStorage. Understanding their behavior—especially string coercion, null handling, and error handling—is essential for correct usage.

### How It Works Internally

`setItem` serializes the value to a string and stores it in the origin's storage area. If the key already exists, it overwrites the previous value. If the storage area is full, it throws a `QuotaExceededError`. `getItem` looks up the key in the storage area and returns the string value, or `null` if the key is not found.

```javascript
// String coercion behavior
localStorage.setItem('number', 42);     // Stores "42"
localStorage.setItem('bool', true);     // Stores "true"
localStorage.setItem('obj', {});        // Stores "[object Object]" (useless!)
localStorage.setItem('null', null);     // Stores "null"
localStorage.setItem('undef', undefined); // Stores "undefined"
```

### Syntax

```javascript
// setItem
localStorage.setItem(keyName, keyValue);
// keyName: string
// keyValue: string (non-strings are coerced)
// Returns: undefined
// Throws: QuotaExceededError if storage is full

// getItem
const value = localStorage.getItem(keyName);
// keyName: string
// Returns: string | null
```

### Beginner Examples

```javascript
// Example 1: Basic storage and retrieval
localStorage.setItem('username', 'john_doe');
const username = localStorage.getItem('username');
console.log(username); // 'john_doe'

// Example 2: Checking if a key exists
const email = localStorage.getItem('user_email');
if (email !== null) {
  console.log('Found email:', email);
} else {
  console.log('No email stored');
}

// Example 3: Storing and reading JSON
const config = {
  theme: 'dark',
  fontSize: 14,
  sidebar: true
};

localStorage.setItem('config', JSON.stringify(config));

const savedConfig = JSON.parse(localStorage.getItem('config'));
console.log(savedConfig.theme); // 'dark'

// Example 4: Safe getItem with default value
function getStorageItem(key, defaultValue = null) {
  const value = localStorage.getItem(key);
  return value !== null ? value : defaultValue;
}

const name = getStorageItem('name', 'Anonymous');
```

### Intermediate Examples

```javascript
// Example 1: Batch operations
function batchSet(items) {
  for (const [key, value] of Object.entries(items)) {
    localStorage.setItem(key, JSON.stringify(value));
  }
}

function batchGet(keys) {
  const result = {};
  for (const key of keys) {
    const value = localStorage.getItem(key);
    if (value !== null) {
      try {
        result[key] = JSON.parse(value);
      } catch {
        result[key] = value;
      }
    }
  }
  return result;
}

batchSet({
  user: { id: 1, name: 'John' },
  preferences: { theme: 'dark' },
  lastVisit: Date.now()
});

const data = batchGet(['user', 'preferences', 'lastVisit']);

// Example 2: Safe setItem with fallback
function safeSetItem(key, value) {
  try {
    localStorage.setItem(key, value);
    return true;
  } catch (e) {
    if (e.name === 'QuotaExceededError') {
      // Free up space by removing oldest items
      removeOldestItems();
      try {
        localStorage.setItem(key, value);
        return true;
      } catch {
        return false;
      }
    }
    return false;
  }
}

function removeOldestItems(count = 5) {
  const items = [];
  for (let i = 0; i < localStorage.length; i++) {
    items.push({
      key: localStorage.key(i),
      value: localStorage.getItem(localStorage.key(i))
    });
  }
  // Remove first 'count' items
  items.slice(0, count).forEach(item => {
    localStorage.removeItem(item.key);
  });
}

// Example 3: Atomic set with conditional
function setIfNotExists(key, value) {
  if (localStorage.getItem(key) === null) {
    localStorage.setItem(key, value);
    return true;
  }
  return false;
}

function setIfExists(key, value) {
  if (localStorage.getItem(key) !== null) {
    localStorage.setItem(key, value);
    return true;
  }
  return false;
}

// Example 4: Versioned storage
function setWithVersion(key, value, version = 1) {
  localStorage.setItem(key, JSON.stringify({
    version,
    data: value,
    timestamp: Date.now()
  }));
}

function getWithVersion(key) {
  const raw = localStorage.getItem(key);
  if (!raw) return null;

  try {
    const item = JSON.parse(raw);
    if (item && typeof item === 'object' && 'version' in item) {
      return item.data;
    }
    // Legacy unversioned data
    return item;
  } catch {
    return raw;
  }
}
```

### Advanced Examples

```javascript
// Example 1: Observable storage
function createObservableStorage() {
  const handlers = new Map();

  const storage = {
    setItem(key, value) {
      const oldValue = localStorage.getItem(key);
      localStorage.setItem(key, value);

      const keyHandlers = handlers.get(key) || [];
      keyHandlers.forEach(fn => fn(value, oldValue, key));
    },

    getItem(key) {
      return localStorage.getItem(key);
    },

    onChange(key, handler) {
      if (!handlers.has(key)) {
        handlers.set(key, []);
      }
      handlers.get(key).push(handler);
      return () => {
        const list = handlers.get(key);
        if (list) {
          const idx = list.indexOf(handler);
          if (idx >= 0) list.splice(idx, 1);
        }
      };
    }
  };

  return storage;
}

const storage = createObservableStorage();

const unsubscribe = storage.onChange('theme', (newVal, oldVal) => {
  document.documentElement.className = `theme-${newVal}`;
});

storage.setItem('theme', 'dark');

// Example 2: Storage migration system
class StorageMigration {
  constructor(version = 1) {
    this.version = version;
  }

  migrate() {
    const currentVersion = parseInt(localStorage.getItem('_storage_version') || '0', 10);

    if (currentVersion < this.version) {
      // Run migrations sequentially
      for (let v = currentVersion + 1; v <= this.version; v++) {
        this[`migrateToV${v}`]?.();
        localStorage.setItem('_storage_version', String(v));
      }
    }
  }

  migrateToV1() {
    // Initial migration: prefix all keys with 'app_'
    const keysToMigrate = [];
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      if (!key.startsWith('app_') && !key.startsWith('_storage')) {
        keysToMigrate.push(key);
      }
    }

    keysToMigrate.forEach(key => {
      const value = localStorage.getItem(key);
      localStorage.setItem(`app_${key}`, value);
      localStorage.removeItem(key);
    });
  }

  migrateToV2() {
    // Migrate specific data structures
    const config = localStorage.getItem('app_config');
    if (config) {
      try {
        const parsed = JSON.parse(config);
        // Transform config structure
        parsed.version = 2;
        parsed.updatedAt = Date.now();
        localStorage.setItem('app_config', JSON.stringify(parsed));
      } catch {
        // Invalid JSON, skip
      }
    }
  }
}

const migration = new StorageMigration(2);
migration.migrate();

// Example 3: Compressed setItem/getItem for large data
async function setItemCompressed(key, value) {
  const json = JSON.stringify(value);
  const encoder = new TextEncoder();
  const data = encoder.encode(json);

  // Compress using CompressionStream if available
  if ('CompressionStream' in window) {
    const cs = new CompressionStream('gzip');
    const writer = cs.writable.getWriter();
    writer.write(data);
    writer.close();

    const reader = cs.readable.getReader();
    const chunks = [];
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      chunks.push(value);
    }

    // Calculate total length
    const totalLength = chunks.reduce((acc, chunk) => acc + chunk.byteLength, 0);
    const compressed = new Uint8Array(totalLength);
    let offset = 0;
    for (const chunk of chunks) {
      compressed.set(chunk, offset);
      offset += chunk.byteLength;
    }

    // Store as base64
    const base64 = btoa(String.fromCharCode(...compressed));
    localStorage.setItem(key, base64);
  } else {
    localStorage.setItem(key, json);
  }
}

async function getItemCompressed(key) {
  const value = localStorage.getItem(key);
  if (!value) return null;

  try {
    // Try parsing as JSON first (uncompressed)
    return JSON.parse(value);
  } catch {
    // Assume it's compressed base64
    try {
      const binary = atob(value);
      const bytes = new Uint8Array(binary.length);
      for (let i = 0; i < binary.length; i++) {
        bytes[i] = binary.charCodeAt(i);
      }

      const ds = new DecompressionStream('gzip');
      const writer = ds.writable.getWriter();
      writer.write(bytes);
      writer.close();

      const reader = ds.readable.getReader();
      const chunks = [];
      while (true) {
        const { done, value } = await reader.read();
        if (done) break;
        chunks.push(value);
      }

      const totalLength = chunks.reduce((acc, chunk) => acc + chunk.byteLength, 0);
      const decompressed = new Uint8Array(totalLength);
      let offset = 0;
      for (const chunk of chunks) {
        decompressed.set(chunk, offset);
        offset += chunk.byteLength;
      }

      const decoder = new TextDecoder();
      return JSON.parse(decoder.decode(decompressed));
    } catch {
      return null;
    }
  }
}
```

### Real-World Use Cases

- **User Authentication**: Storing JWT tokens for session management
- **Application Settings**: Saving user preferences between sessions
- **Cart Management**: Persistent shopping cart for unauthenticated users
- **Form Drafts**: Auto-saving form progress
- **Game Saves**: Storing game state locally
- **Feature Flags**: Client-side feature gating
- **A/B Testing Variants**: Storing user's test group assignment
- **Last Visited State**: Remembering the last page or scroll position

### Common Mistakes

```javascript
// Mistake 1: Not parsing JSON on retrieval
localStorage.setItem('data', JSON.stringify([1, 2, 3]));
const data = localStorage.getItem('data');
console.log(Array.isArray(data)); // false (it's a string)
const parsed = JSON.parse(data);
console.log(Array.isArray(parsed)); // true

// Mistake 2: Not handling null from getItem
const value = localStorage.getItem('nonexistent');
value.prop = 1; // TypeError: Cannot read property 'prop' of null

// Mistake 3: Silent failures in private browsing
try {
  localStorage.setItem('test', 'value');
} catch {
  // localStorage may be unavailable
}
```

### Best Practices

- Always use `JSON.stringify`/`JSON.parse` for non-string data
- Check for `null` return from `getItem`
- Wrap `setItem` in try-catch for quota handling
- Use `removeItem` to clean up unused keys
- Prefer `getItem(key)` over `localStorage.key` to avoid prototype pollution

### Performance Considerations

Individual `setItem` and `getItem` calls are fast (< 1ms for typical data). However, for large datasets or frequent operations, the synchronous nature can cause jank. Batch operations and caching in memory can mitigate this.

### Interview Questions

1. What does `getItem` return if the key doesn't exist?
2. What happens if you store a non-string value with `setItem`?
3. How do you safely store and retrieve objects?
4. What error does `setItem` throw when storage is full?
5. Is there a way to get all keys at once?

### Coding Challenges

1. **Safe Storage Wrapper**: Create a wrapper that handles serialization, errors, and defaults
2. **Batch Operations**: Implement efficient batch read/write for localStorage
3. **Observable Storage**: Create storage with change notification callbacks

### Related Topics

- [JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)
- [JSON.parse](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)
- [QuotaExceededError](https://developer.mozilla.org/en-US/docs/Web/API/Storage/setItem)

## removeItem() and clear()

### What It Is

`removeItem()` deletes a specific key-value pair from localStorage. `clear()` removes all key-value pairs for the current origin. Both methods are irreversible and operate synchronously.

```javascript
localStorage.removeItem('temporary_data');
localStorage.clear();
```

### Why It Is Important

Proper cleanup of localStorage data is essential for managing storage quota and maintaining data hygiene. `removeItem` allows targeted deletion of specific data, while `clear` is useful for complete resets (e.g., user logout).

### How It Works Internally

`removeItem` deletes the entry from the origin's storage map and queues a disk write. If the key doesn't exist, the call is a no-op. `clear` removes all entries for the origin. Both methods dispatch a `StorageEvent` to other tabs/windows of the same origin.

### Syntax

```javascript
localStorage.removeItem(key);
localStorage.clear();
```

### Beginner Examples

```javascript
// Example 1: Removing specific keys
localStorage.removeItem('temporary_token');
localStorage.removeItem('session_cache');

// Example 2: Clear all on logout
function logout() {
  localStorage.removeItem('auth_token');
  localStorage.removeItem('user_profile');
  localStorage.removeItem('user_preferences');
  // Or simply: localStorage.clear();
  window.location.href = '/login';
}

// Example 3: Clear with confirmation
function clearAllData() {
  if (confirm('This will remove all saved data. Continue?')) {
    localStorage.clear();
    updateUI();
  }
}

// Example 4: Remove keys matching a pattern
function removeMatching(pattern) {
  const regex = new RegExp(pattern);
  const keysToRemove = [];

  for (let i = 0; i < localStorage.length; i++) {
    const key = localStorage.key(i);
    if (regex.test(key)) {
      keysToRemove.push(key);
    }
  }

  keysToRemove.forEach(key => localStorage.removeItem(key));
  return keysToRemove.length;
}

const removedCount = removeMatching('^cache_');
console.log(`Removed ${removedCount} cache entries`);
```

### Intermediate Examples

```javascript
// Example 1: Remove oldest items when quota is low
function pruneStorage(maxItems = 100) {
  if (localStorage.length <= maxItems) return;

  const items = [];
  for (let i = 0; i < localStorage.length; i++) {
    const key = localStorage.key(i);
    const value = localStorage.getItem(key);
    // Assume items have a timestamp in JSON
    try {
      const parsed = JSON.parse(value);
      items.push({ key, timestamp: parsed.timestamp || 0 });
    } catch {
      items.push({ key, timestamp: 0 });
    }
  }

  // Sort by timestamp ascending (oldest first)
  items.sort((a, b) => a.timestamp - b.timestamp);

  // Remove oldest items until under limit
  const toRemove = items.slice(0, items.length - maxItems);
  toRemove.forEach(item => localStorage.removeItem(item.key));
}

// Example 2: Garbage collection for expired items
function garbageCollect() {
  let removedCount = 0;

  for (let i = localStorage.length - 1; i >= 0; i--) {
    const key = localStorage.key(i);
    const value = localStorage.getItem(key);

    try {
      const parsed = JSON.parse(value);
      if (parsed.expiry && Date.now() > parsed.expiry) {
        localStorage.removeItem(key);
        removedCount++;
      }
    } catch {
      // Not JSON or no expiry, skip
    }
  }

  return removedCount;
}

// Run GC on page load
document.addEventListener('DOMContentLoaded', () => {
  const removed = garbageCollect();
  if (removed > 0) {
    console.log(`Cleaned up ${removed} expired items`);
  }
});

// Example 3: Selective clear with whitelist
function clearExcept(whitelist = []) {
  const keysToKeep = new Set(whitelist);
  const preserved = {};

  // Save whitelisted items
  for (const key of keysToKeep) {
    const value = localStorage.getItem(key);
    if (value !== null) {
      preserved[key] = value;
    }
  }

  // Clear everything
  localStorage.clear();

  // Restore whitelisted items
  for (const [key, value] of Object.entries(preserved)) {
    localStorage.setItem(key, value);
  }
}

clearExcept(['theme', 'language', 'auth_token']);
```

### Advanced Examples

```javascript
// Example 1: Storage recovery system
class StorageRecovery {
  constructor() {
    this.backupKey = '_storage_backup';
  }

  backup() {
    const data = {};
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      data[key] = localStorage.getItem(key);
    }
    localStorage.setItem(this.backupKey, JSON.stringify(data));
  }

  restore() {
    const backup = localStorage.getItem(this.backupKey);
    if (!backup) return false;

    try {
      const data = JSON.parse(backup);
      for (const [key, value] of Object.entries(data)) {
        if (key !== this.backupKey) {
          localStorage.setItem(key, value);
        }
      }
      return true;
    } catch {
      return false;
    }
  }

  clear() {
    localStorage.clear();
  }
}

const recovery = new StorageRecovery();
recovery.backup();
// ... later, if data gets corrupted:
// recovery.restore();

// Example 2: undoable remove
class UndoableStorage {
  constructor() {
    this.undoStack = [];
    this.maxUndo = 10;
  }

  removeItem(key) {
    const value = localStorage.getItem(key);
    if (value !== null) {
      this.undoStack.push({ type: 'remove', key, value });
      if (this.undoStack.length > this.maxUndo) {
        this.undoStack.shift();
      }
      localStorage.removeItem(key);
    }
  }

  clear() {
    const snapshot = {};
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      snapshot[key] = localStorage.getItem(key);
    }
    this.undoStack.push({ type: 'clear', snapshot });
    localStorage.clear();
  }

  undo() {
    const action = this.undoStack.pop();
    if (!action) return false;

    if (action.type === 'remove') {
      localStorage.setItem(action.key, action.value);
    } else if (action.type === 'clear') {
      for (const [key, value] of Object.entries(action.snapshot)) {
        localStorage.setItem(key, value);
      }
    }

    return true;
  }
}

// Example 3: Atomic batch delete with rollback
function batchDelete(keys) {
  const backup = {};

  for (const key of keys) {
    const value = localStorage.getItem(key);
    if (value !== null) {
      backup[key] = value;
    }
  }

  // Perform deletion
  for (const key of keys) {
    localStorage.removeItem(key);
  }

  // Return rollback function
  return function rollback() {
    for (const [key, value] of Object.entries(backup)) {
      localStorage.setItem(key, value);
    }
  };
}

const rollback = batchDelete(['temp_data', 'cache_response']);
// If something goes wrong:
// rollback();
```

### Real-World Use Cases

- **User Logout**: Clear authentication tokens and user data
- **Cache Invalidation**: Remove stale cached data
- **Data Migration**: Remove old data after migration to new format
- **Storage Cleanup**: Remove expired items during garbage collection
- **Factory Reset**: Clear all application state
- **Privacy Compliance**: Delete user data on request
- **Testing**: Clear state between test runs

### Common Mistakes

```javascript
// Mistake 1: Not removing old keys when updating data structure
localStorage.setItem('user_v1', data);
// After migration to v2:
localStorage.setItem('user_v2', data);
// User v1 data still exists! Remove it.

// Mistake 2: Using clear() when only specific keys need removal
localStorage.clear(); // Removes ALL data including user preferences
localStorage.removeItem('specific_key'); // Better

// Mistake 3: Assuming removeItem returns success/failure
removeItem always returns undefined regardless of whether the key existed
```

### Best Practices

- Remove individual items instead of clearing all data when possible
- Clean up old data when migrating to new data formats
- Use pattern-matching removal for bulk cleanup
- Always provide a way for users to clear their data (privacy)
- Implement garbage collection for expired cached items

### Performance Considerations

`removeItem` and `clear` are O(1) and O(n) operations respectively. For large storage, `clear` is faster than removing items one by one because it's a single operation in the browser's storage engine.

### Interview Questions

1. What is the difference between `removeItem` and `clear`?
2. What happens if you call `removeItem` on a non-existent key?
3. Does `clear` remove data from all origins?
4. How do you remove multiple keys that match a pattern?
5. Can you undo a `clear` operation?

### Coding Challenges

1. **Storage Cleaner**: Build a utility that removes expired items from localStorage
2. **Undoable Storage**: Create a storage wrapper with undo capability for destructive operations
3. **Pattern Cleaner**: Implement a function that deletes all keys matching a regex pattern

### Related Topics

- [Storage.clear()](https://developer.mozilla.org/en-US/docs/Web/API/Storage/clear)
- [Storage.removeItem()](https://developer.mozilla.org/en-US/docs/Web/API/Storage/removeItem)
- [StorageEvent](https://developer.mozilla.org/en-US/docs/Web/API/StorageEvent)

## Storage Limits and Security

### What It Is

localStorage has storage limits that vary by browser, typically 5-10 MB per origin. Security considerations include the synchronous API, lack of encryption, exposure to XSS attacks, and the fact that data persists indefinitely until explicitly removed.

```javascript
// Checking approximate available space
function checkStorageLimit() {
  const testKey = '_storage_test_';
  let total = 0;

  try {
    // Fill storage until quota exceeded
    while (true) {
      localStorage.setItem(testKey, 'a'.repeat(1024 * 100)); // 100KB chunks
      total += 100;
    }
  } catch (e) {
    localStorage.removeItem(testKey);
    return total; // Approximate MB
  }
}

// console.log(`Approximate storage limit: ${checkStorageLimit()} MB`);
```

### Why It Is Important

Understanding storage limits helps prevent data loss from quota exceeded errors. Security awareness prevents common vulnerabilities like XSS-based data theft. The persistence model affects application design decisions about what to store and when to clean up.

### How It Works Internally

Each browser enforces storage quotas at the origin level. Chromium uses a pool of disk space shared across all origins (typically 10% of available disk, up to ~10MB per origin). Firefox uses 10 MB per origin. Safari uses 5 MB per origin. In private browsing mode, localStorage may be completely unavailable or cleared when the browser is closed.

```javascript
// Browser storage limits (approximate):
// Chrome: 10 MB per origin
// Firefox: 10 MB per origin
// Safari: 5 MB per origin
// IE/Edge: 10 MB per origin
// iOS Safari: 5 MB per origin
// Private browsing: May block localStorage entirely
```

### Syntax

```javascript
// Check availability
function isLocalStorageAvailable() {
  try {
    const test = '__storage_test__';
    localStorage.setItem(test, test);
    localStorage.removeItem(test);
    return true;
  } catch {
    return false;
  }
}

// Estimate remaining space
function estimateRemainingSpace() {
  const testKey = '__space_test__';
  const chunkSize = 1024 * 1024; // 1MB chunks
  let totalStored = 0;

  try {
    while (true) {
      localStorage.setItem(testKey + totalStored, 'x'.repeat(chunkSize));
      totalStored++;
    }
  } catch {
    // Clean up
    for (let i = 0; i < totalStored; i++) {
      localStorage.removeItem(testKey + i);
    }
    return totalStored; // MB
  }
}
```

### Beginner Examples

```javascript
// Example 1: Checking localStorage availability
if (typeof Storage !== 'undefined') {
  // localStorage is available
  localStorage.setItem('test', 'value');
} else {
  console.warn('localStorage is not supported');
  // Fallback to cookies or in-memory storage
}

// Example 2: Quota exceeded handling
function safeStorageSet(key, value) {
  try {
    localStorage.setItem(key, value);
    return true;
  } catch (error) {
    if (error.name === 'QuotaExceededError') {
      console.warn('Storage quota exceeded');
      // Option 1: Remove oldest items
      pruneOldItems();
      // Option 2: Notify user
      notifyUser('Storage is full. Please clear some data.');
    }
    return false;
  }
}

// Example 3: Privacy - clearing data on logout
document.querySelector('#logout').addEventListener('click', () => {
  if (confirm('Logout and clear local data?')) {
    localStorage.clear();
    window.location.href = '/login';
  }
});
```

### Intermediate Examples

```javascript
// Example 1: Storage quota monitor
class StorageQuotaMonitor {
  constructor(warningThreshold = 0.8) {
    this.warningThreshold = warningThreshold;
    this.limit = 5 * 1024 * 1024; // Assume 5MB limit
  }

  getUsedBytes() {
    let total = 0;
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      const value = localStorage.getItem(key);
      total += key.length + value.length;
    }
    return total;
  }

  getUsagePercent() {
    return (this.getUsedBytes() / this.limit) * 100;
  }

  isNearLimit() {
    return this.getUsagePercent() > (this.warningThreshold * 100);
  }

  monitor() {
    if (this.isNearLimit()) {
      console.warn(`Storage at ${this.getUsagePercent().toFixed(1)}% capacity`);
      this.suggestCleanup();
    }
  }

  suggestCleanup() {
    const largeItems = [];
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      const value = localStorage.getItem(key);
      const size = key.length + value.length;
      if (size > 1024 * 100) { // > 100KB
        largeItems.push({ key, size });
      }
    }

    if (largeItems.length > 0) {
      console.table(largeItems.sort((a, b) => b.size - a.size));
      console.log('Consider removing large cached items');
    }
  }
}

const monitor = new StorageQuotaMonitor();
monitor.monitor();

// Example 2: Secure storage wrapper (encryption)
class SecureStorage {
  constructor(secretKey) {
    this.secretKey = secretKey;
  }

  // Simple XOR-based obfuscation (NOT cryptographically secure)
  // For real security, use Web Crypto API
  encrypt(text) {
    let result = '';
    for (let i = 0; i < text.length; i++) {
      const code = text.charCodeAt(i) ^ this.secretKey.charCodeAt(i % this.secretKey.length);
      result += String.fromCharCode(code);
    }
    return btoa(result);
  }

  decrypt(encoded) {
    const text = atob(encoded);
    let result = '';
    for (let i = 0; i < text.length; i++) {
      const code = text.charCodeAt(i) ^ this.secretKey.charCodeAt(i % this.secretKey.length);
      result += String.fromCharCode(code);
    }
    return result;
  }

  setItem(key, value) {
    const serialized = JSON.stringify(value);
    const encrypted = this.encrypt(serialized);
    localStorage.setItem(key, encrypted);
  }

  getItem(key) {
    const encrypted = localStorage.getItem(key);
    if (!encrypted) return null;
    try {
      const decrypted = this.decrypt(encrypted);
      return JSON.parse(decrypted);
    } catch {
      return null;
    }
  }

  removeItem(key) {
    localStorage.removeItem(key);
  }

  clear() {
    localStorage.clear();
  }
}

// Note: True security requires the Web Crypto API
async function securelyStore(key, value) {
  const encoder = new TextEncoder();
  const data = encoder.encode(JSON.stringify(value));

  // Generate a key (in production, derive from user password)
  const cryptoKey = await crypto.subtle.generateKey(
    { name: 'AES-GCM', length: 256 },
    true,
    ['encrypt', 'decrypt']
  );

  const iv = crypto.getRandomValues(new Uint8Array(12));
  const encrypted = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv },
    cryptoKey,
    data
  );

  // Store encrypted data and IV
  const exportKey = await crypto.subtle.exportKey('raw', cryptoKey);
  localStorage.setItem(key + '_key', btoa(String.fromCharCode(...new Uint8Array(exportKey))));
  localStorage.setItem(key + '_iv', btoa(String.fromCharCode(...iv)));
  localStorage.setItem(key, btoa(String.fromCharCode(...new Uint8Array(encrypted))));
}

// Example 3: Fallback storage chain
class FallbackStorage {
  constructor() {
    this.stores = [];
    this.init();
  }

  init() {
    // Priority: localStorage > sessionStorage > in-memory
    if (this.isAvailable('localStorage')) {
      this.stores.push({ name: 'localStorage', api: localStorage });
    }
    if (this.isAvailable('sessionStorage')) {
      this.stores.push({ name: 'sessionStorage', api: sessionStorage });
    }
    // In-memory fallback
    this.memoryStore = new Map();
    this.stores.push({ name: 'memory', api: null });
  }

  isAvailable(type) {
    try {
      const storage = window[type];
      const test = '__storage_test__';
      storage.setItem(test, test);
      storage.removeItem(test);
      return true;
    } catch {
      return false;
    }
  }

  setItem(key, value) {
    const serialized = JSON.stringify(value);

    for (const store of this.stores) {
      if (store.api) {
        try {
          store.api.setItem(key, serialized);
          return;
        } catch {
          continue; // Try next store
        }
      } else {
        this.memoryStore.set(key, serialized);
      }
    }
  }

  getItem(key) {
    for (const store of this.stores) {
      let value = null;

      if (store.api) {
        try {
          value = store.api.getItem(key);
        } catch {
          continue;
        }
      } else {
        value = this.memoryStore.get(key) || null;
      }

      if (value !== null) {
        try {
          return JSON.parse(value);
        } catch {
          return value;
        }
      }
    }

    return null;
  }

  removeItem(key) {
    for (const store of this.stores) {
      if (store.api) {
        try { store.api.removeItem(key); } catch {}
      } else {
        this.memoryStore.delete(key);
      }
    }
  }

  clear() {
    for (const store of this.stores) {
      if (store.api) {
        try { store.api.clear(); } catch {}
      } else {
        this.memoryStore.clear();
      }
    }
  }
}

const storage = new FallbackStorage();
storage.setItem('theme', 'dark');
console.log(storage.getItem('theme')); // 'dark'
```

### Real-World Use Cases

- **Quota Monitoring**: Alert users when storage is nearly full
- **Data Cleanup**: Automatic cleanup of old cached data
- **Secure Storage**: Encrypting sensitive data before storing (though limited)
- **Fallback Chains**: Graceful degradation when localStorage is unavailable
- **Privacy Compliance**: GDPR/CCPA data deletion mechanisms
- **Cross-Browser Compatibility**: Handling different storage limits gracefully

### Common Mistakes

```javascript
// Mistake 1: Assuming localStorage is always available
// In private browsing or some security contexts, it may throw

// Mistake 2: Storing sensitive data without encryption
localStorage.setItem('credit_card', '4111-1111-1111-1111'); // NEVER

// Mistake 3: Not handling quota exceeded errors
// Always wrap setItem in try-catch

// Mistake 4: Assuming data is isolated per subdomain
// localStorage.example.com and app.example.com share storage
// (same registered domain)
```

### Best Practices

- Always check localStorage availability before use
- Handle quota exceeded errors gracefully
- Never store sensitive data (passwords, tokens, PII) without encryption
- Implement automatic cleanup for cached/expired data
- Provide user controls to clear stored data
- Use fallback storage mechanisms for private browsing modes
- Consider using the Web Crypto API for truly secure storage needs
- Monitor storage usage and warn users when approaching limits

### Performance Considerations

Storage operations block the main thread. For large data, consider IndexedDB. The synchronous nature of localStorage means that storage operations in tight loops can cause jank. Memory-mapped implementations in modern browsers make this less of an issue for typical use cases.

### Interview Questions

1. What is the storage limit for localStorage?
2. What happens when you exceed the localStorage quota?
3. Is localStorage secure? Why or why not?
4. How can you check if localStorage is available?
5. What happens to localStorage in private browsing mode?
6. How do different browsers handle storage limits?
7. Can you encrypt data in localStorage?
8. What is the difference between origin and subdomain isolation?

### Coding Challenges

1. **Quota Monitor**: Build a storage quota monitoring and warning system
2. **Secure Storage**: Implement client-side encryption for localStorage values
3. **Fallback Chain**: Create a storage system that falls back from localStorage to IndexedDB to memory
4. **Storage Cleanup Scheduler**: Build an automatic cleanup system that removes old data when quota is low
5. **Cross-Origin Storage Checker**: Determine storage limits across different browsers

### Related Topics

- [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API)
- [Cookies vs localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
- [Storage Quota Estimation API](https://developer.mozilla.org/en-US/docs/Web/API/StorageManager/estimate)
