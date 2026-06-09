# Session Storage - sessionStorage API, differences from localStorage, tab-scoped

## Introduction

`sessionStorage` is a web storage mechanism similar to `localStorage` but with a crucial difference: data persists only for the duration of the page session. Once the browser tab or window is closed, the data is automatically deleted. Session storage is scoped to the current tab or window, meaning data is not shared between tabs, even if they load the same page.

The `sessionStorage` API shares the same interface as `localStorage` (`setItem`, `getItem`, `removeItem`, `clear`, `key`, `length`), making it easy to switch between the two. However, the differences in persistence and scoping make each suitable for different use cases.

## sessionStorage API

### What It Is

`sessionStorage` is a key-value storage system available on the `window` object that persists data for the duration of a page session. A page session lasts as long as the browser tab is open and survives page reloads and restores. Opening a page in a new tab or window creates a new session, which is separate from the original.

```javascript
// Store data (same API as localStorage)
sessionStorage.setItem('pageViewCount', '1');
sessionStorage.setItem('formData', JSON.stringify({ step: 2, draft: '...' }));

// Retrieve data
const count = sessionStorage.getItem('pageViewCount');
const formData = JSON.parse(sessionStorage.getItem('formData'));

// Remove data
sessionStorage.removeItem('temporaryFlag');
sessionStorage.clear();
```

### Why It Is Important

`sessionStorage` is essential for storing transient state that should not persist beyond the current browsing session. It is ideal for wizard forms (multi-step data), temporary UI state, page-specific caches, and any data that should be cleared when the user closes the tab. Its tab-scoped nature makes it useful for isolating state between different instances of the same application.

### How It Works Internally

Each browser tab maintains its own `sessionStorage` data store. The data exists in memory and is optionally backed by disk to survive page refreshes. When the tab is closed, the data store is discarded. Unlike `localStorage`, there is no sharing between tabs—each tab has its own isolated storage area.

```javascript
// Tab isolation:
// Tab 1: sessionStorage.setItem('key', 'value1')
// Tab 2: sessionStorage.getItem('key') // null (different session)

// Session persistence:
// Page reload: sessionStorage data survives
// Same-page navigation: data survives
// Link to another origin: data is lost
// Close tab: data is permanently deleted
```

### Syntax

```javascript
// All methods (identical to localStorage)
sessionStorage.setItem(key, value);
sessionStorage.getItem(key);        // Returns string or null
sessionStorage.removeItem(key);
sessionStorage.clear();
sessionStorage.key(index);          // Get key at index
sessionStorage.length;              // Number of items

// Property access (alternative)
sessionStorage.myKey = 'value';
const val = sessionStorage.myKey;
```

### Beginner Examples

```javascript
// Example 1: Page view counter per session
function incrementPageView() {
  const count = parseInt(sessionStorage.getItem('viewCount') || '0', 10) + 1;
  sessionStorage.setItem('viewCount', String(count));
  return count;
}

console.log(`You've viewed ${incrementPageView()} pages in this session`);

// Example 2: Multi-step form wizard
const form = document.querySelector('#wizard-form');
const stepKey = 'wizard_step';

// Save current step
function saveStep(step, data) {
  sessionStorage.setItem(stepKey, JSON.stringify({ step, data }));
}

// Resume from saved step
function resumeWizard() {
  const saved = sessionStorage.getItem(stepKey);
  if (saved) {
    const { step, data } = JSON.parse(saved);
    goToStep(step);
    populateForm(data);
  }
}

form.addEventListener('submit', (e) => {
  e.preventDefault();
  const currentStep = getCurrentStep();
  const formData = new FormData(form);

  if (currentStep < totalSteps) {
    saveStep(currentStep + 1, Object.fromEntries(formData));
    goToStep(currentStep + 1);
  } else {
    // Final submission - clear session storage
    sessionStorage.removeItem(stepKey);
    submitForm(formData);
  }
});

resumeWizard();

// Example 3: Tab-specific UI state
document.querySelector('.accordion').addEventListener('click', (e) => {
  const panel = e.target.closest('.accordion-panel');
  if (!panel) return;

  const id = panel.dataset.panelId;
  const isOpen = panel.classList.toggle('open');

  // Save accordion state for this tab only
  sessionStorage.setItem(`accordion_${id}`, isOpen ? 'open' : 'closed');
});

// Restore accordion state
document.querySelectorAll('.accordion-panel').forEach(panel => {
  const id = panel.dataset.panelId;
  const state = sessionStorage.getItem(`accordion_${id}`);
  if (state === 'open') {
    panel.classList.add('open');
  }
});

// Example 4: Scroll position restoration
const scrollKey = 'scroll_position_' + window.location.pathname;

window.addEventListener('scroll', () => {
  sessionStorage.setItem(scrollKey, String(window.scrollY));
});

window.addEventListener('load', () => {
  const saved = sessionStorage.getItem(scrollKey);
  if (saved) {
    window.scrollTo(0, parseInt(saved, 10));
  }
});
```

### Intermediate Examples

```javascript
// Example 1: Session-based undo/redo
class SessionUndoRedo {
  constructor(maxSteps = 20) {
    this.key = 'undo_stack';
    this.maxSteps = maxSteps;
    this.stack = this.load() || { undo: [], redo: [] };
  }

  load() {
    try {
      return JSON.parse(sessionStorage.getItem(this.key));
    } catch {
      return null;
    }
  }

  save() {
    sessionStorage.setItem(this.key, JSON.stringify(this.stack));
  }

  push(action) {
    this.stack.undo.push(action);
    this.stack.redo = []; // Clear redo on new action
    if (this.stack.undo.length > this.maxSteps) {
      this.stack.undo.shift();
    }
    this.save();
  }

  undo() {
    const action = this.stack.undo.pop();
    if (action) {
      this.stack.redo.push(action);
      this.save();
      return action;
    }
    return null;
  }

  redo() {
    const action = this.stack.redo.pop();
    if (action) {
      this.stack.undo.push(action);
      this.save();
      return action;
    }
    return null;
  }

  clear() {
    sessionStorage.removeItem(this.key);
    this.stack = { undo: [], redo: [] };
  }
}

// Example 2: Session-scoped cache with request deduplication
class SessionCache {
  constructor() {
    this.pendingRequests = new Map();
  }

  async fetch(url, options = {}) {
    const cacheKey = `cache_${url}`;
    const ttl = options.ttl || 30000; // 30 seconds default

    // Check cache
    const cached = this.get(cacheKey);
    if (cached !== null) return cached;

    // Deduplicate in-flight requests
    if (this.pendingRequests.has(url)) {
      return this.pendingRequests.get(url);
    }

    // Fetch
    const promise = (async () => {
      try {
        const response = await fetch(url);
        const data = await response.json();
        this.set(cacheKey, data, ttl);
        return data;
      } finally {
        this.pendingRequests.delete(url);
      }
    })();

    this.pendingRequests.set(url, promise);
    return promise;
  }

  set(key, value, ttlMs = 30000) {
    sessionStorage.setItem(key, JSON.stringify({
      data: value,
      expiry: Date.now() + ttlMs
    }));
  }

  get(key) {
    const raw = sessionStorage.getItem(key);
    if (!raw) return null;

    try {
      const item = JSON.parse(raw);
      if (Date.now() > item.expiry) {
        sessionStorage.removeItem(key);
        return null;
      }
      return item.data;
    } catch {
      return null;
    }
  }

  invalidate(pattern) {
    const regex = new RegExp(pattern);
    for (let i = sessionStorage.length - 1; i >= 0; i--) {
      const key = sessionStorage.key(i);
      if (key.startsWith('cache_') && regex.test(key)) {
        sessionStorage.removeItem(key);
      }
    }
  }

  clear() {
    for (let i = sessionStorage.length - 1; i >= 0; i--) {
      const key = sessionStorage.key(i);
      if (key.startsWith('cache_')) {
        sessionStorage.removeItem(key);
      }
    }
  }
}

const sessionCache = new SessionCache();
sessionCache.fetch('/api/data', { ttl: 60000 });

// Example 3: Session isolation for multiple instances
class SessionManager {
  constructor(prefix = 'app') {
    this.prefix = prefix;
    this.instanceId = this.getInstanceId();
  }

  getInstanceId() {
    let id = sessionStorage.getItem(`${this.prefix}_instance_id`);
    if (!id) {
      id = `${Date.now()}_${Math.random().toString(36).slice(2, 8)}`;
      sessionStorage.setItem(`${this.prefix}_instance_id`, id);
    }
    return id;
  }

  set(key, value) {
    sessionStorage.setItem(`${this.prefix}_${this.instanceId}_${key}`, JSON.stringify(value));
  }

  get(key) {
    const value = sessionStorage.getItem(`${this.prefix}_${this.instanceId}_${key}`);
    try {
      return value ? JSON.parse(value) : null;
    } catch {
      return value;
    }
  }

  remove(key) {
    sessionStorage.removeItem(`${this.prefix}_${this.instanceId}_${key}`);
  }

  clearInstance() {
    const prefix = `${this.prefix}_${this.instanceId}_`;
    for (let i = sessionStorage.length - 1; i >= 0; i--) {
      const key = sessionStorage.key(i);
      if (key.startsWith(prefix)) {
        sessionStorage.removeItem(key);
      }
    }
  }
}

const session1 = new SessionManager('app');
session1.set('theme', 'dark');
console.log(session1.get('theme')); // 'dark'
```

### Advanced Examples

```javascript
// Example 1: Session storage with expiry and events
class AdvancedSessionStorage {
  constructor() {
    this.changeListeners = new Map();
  }

  setItem(key, value, options = {}) {
    const item = {
      value,
      timestamp: Date.now(),
      expiry: options.ttl ? Date.now() + options.ttl : null
    };

    sessionStorage.setItem(key, JSON.stringify(item));
    this.notify(key, value, null);
  }

  getItem(key) {
    const raw = sessionStorage.getItem(key);
    if (!raw) return null;

    try {
      const item = JSON.parse(raw);

      if (item.expiry && Date.now() > item.expiry) {
        sessionStorage.removeItem(key);
        this.notify(key, null, 'expired');
        return null;
      }

      return item.value;
    } catch {
      return null;
    }
  }

  removeItem(key) {
    sessionStorage.removeItem(key);
    this.notify(key, null, 'removed');
  }

  clear() {
    sessionStorage.clear();
    this.notify('*', null, 'cleared');
  }

  onChange(key, callback) {
    if (!this.changeListeners.has(key)) {
      this.changeListeners.set(key, []);
    }
    this.changeListeners.get(key).push(callback);
    return () => {
      const handlers = this.changeListeners.get(key);
      if (handlers) {
        const idx = handlers.indexOf(callback);
        if (idx >= 0) handlers.splice(idx, 1);
      }
    };
  }

  notify(key, value, reason) {
    const handlers = this.changeListeners.get(key) || [];
    const wildcardHandlers = this.changeListeners.get('*') || [];

    for (const handler of [...handlers, ...wildcardHandlers]) {
      handler(key, value, reason);
    }
  }

  get size() {
    return sessionStorage.length;
  }

  keys() {
    const keys = [];
    for (let i = 0; i < sessionStorage.length; i++) {
      keys.push(sessionStorage.key(i));
    }
    return keys;
  }
}

const ss = new AdvancedSessionStorage();

const unsub = ss.onChange('theme', (key, value, reason) => {
  if (value) {
    document.documentElement.className = `theme-${value}`;
  }
});

ss.setItem('theme', 'dark', { ttl: 3600000 }); // 1 hour

// Example 2: Session storage synchronization across iframes
class CrossFrameSession {
  constructor() {
    this.channel = new BroadcastChannel('session-sync');
    this.setup();
  }

  setup() {
    // Listen for session changes from other frames
    this.channel.addEventListener('message', (event) => {
      const { type, key, value } = event.data;
      if (type === 'session-set') {
        this.handleRemoteSet(key, value);
      } else if (type === 'session-remove') {
        this.handleRemoteRemove(key);
      }
    });
  }

  setItem(key, value) {
    sessionStorage.setItem(key, value);
    this.channel.postMessage({ type: 'session-set', key, value });
  }

  removeItem(key) {
    sessionStorage.removeItem(key);
    this.channel.postMessage({ type: 'session-remove', key });
  }

  handleRemoteSet(key, value) {
    // Only apply if local value doesn't exist or is older
    const local = sessionStorage.getItem(key);
    if (local === null) {
      sessionStorage.setItem(key, value);
    }
  }

  handleRemoteRemove(key) {
    // Respect remote removal
    sessionStorage.removeItem(key);
  }
}

// Example 3: Session-aware reactive state
class SessionReactiveState {
  constructor(name, initial = {}) {
    this.name = name;
    this.subscribers = new Set();
    this.state = this.load(initial);
  }

  load(initial) {
    try {
      const saved = sessionStorage.getItem(`state_${this.name}`);
      return saved ? { ...initial, ...JSON.parse(saved) } : initial;
    } catch {
      return initial;
    }
  }

  save() {
    sessionStorage.setItem(`state_${this.name}`, JSON.stringify(this.state));
  }

  get(key) {
    return key ? this.state[key] : this.state;
  }

  set(key, value) {
    if (typeof key === 'object') {
      Object.assign(this.state, key);
    } else {
      this.state[key] = value;
    }
    this.save();
    this.notify(key, value);
  }

  subscribe(fn) {
    this.subscribers.add(fn);
    return () => this.subscribers.delete(fn);
  }

  notify(key, value) {
    for (const fn of this.subscribers) {
      fn(key, value, this.state);
    }
  }

  reset() {
    sessionStorage.removeItem(`state_${this.name}`);
    this.state = {};
    this.notify('*reset*', null);
  }
}

const pageState = new SessionReactiveState('page', {
  scrollPosition: 0,
  activeTab: 'home',
  filters: {}
});

pageState.subscribe((key, value) => {
  if (key === 'activeTab') {
    updateTabUI(value);
  }
});

pageState.set('activeTab', 'settings');

// Example 4: Session key-value store with versioning
class VersionedSessionStore {
  constructor(version = 1) {
    this.version = version;
    this.metaKey = '_store_meta';
    this.ensureVersion();
  }

  ensureVersion() {
    const meta = this.getMeta();
    if (meta.version < this.version) {
      this.migrate(meta.version, this.version);
      meta.version = this.version;
      this.setMeta(meta);
    }
  }

  getMeta() {
    try {
      return JSON.parse(sessionStorage.getItem(this.metaKey)) || { version: 1, created: Date.now() };
    } catch {
      return { version: 1, created: Date.now() };
    }
  }

  setMeta(meta) {
    sessionStorage.setItem(this.metaKey, JSON.stringify(meta));
  }

  setItem(key, value) {
    sessionStorage.setItem(key, JSON.stringify({
      value,
      version: this.version,
      updated: Date.now()
    }));
  }

  getItem(key) {
    const raw = sessionStorage.getItem(key);
    if (!raw) return null;

    try {
      const item = JSON.parse(raw);
      return item.value;
    } catch {
      return null;
    }
  }

  migrate(fromVersion, toVersion) {
    for (let v = fromVersion; v < toVersion; v++) {
      const migration = this[`migrateV${v}toV${v + 1}`];
      if (migration) migration();
    }
  }

  // Example migration
  migrateV1toV2() {
    // Prefix all existing keys with 'v2_'
    for (let i = sessionStorage.length - 1; i >= 0; i--) {
      const key = sessionStorage.key(i);
      if (key !== this.metaKey) {
        const value = sessionStorage.getItem(key);
        sessionStorage.removeItem(key);
        sessionStorage.setItem(`v2_${key}`, value);
      }
    }
  }

  clear() {
    const meta = this.getMeta();
    sessionStorage.clear();
    this.setMeta({ ...meta, version: this.version, cleared: Date.now() });
  }
}
```

### Real-World Use Cases

- **Wizard/Stepped Forms**: Store partially completed form data across page reloads
- **Tab-Specific UI State**: Accordion states, tab selections, expand/collapse states
- **Transient Notifications**: Track which notifications the user has seen this session
- **Page Navigation History**: Keep a stack of visited pages within a tab for custom back navigation
- **Scroll Position Memory**: Remember scroll position when navigating back
- **Temporary Caching**: Cache API responses that are only relevant for the current session
- **Analytics Session Tracking**: Generate and track a unique session ID
- **Multi-Step Checkout**: Store checkout progress across page reloads
- **Undo/Redo Stacks**: Maintain per-tab undo/redo history
- **A/B Test Assignments**: Store test variant assignments for the current session

### Common Mistakes

```javascript
// Mistake 1: Expecting sessionStorage to persist across tabs
sessionStorage.setItem('key', 'value');
// Opening a new tab will NOT have this data

// Mistake 2: Using sessionStorage for data that needs to persist after tab close
// sessionStorage data is lost when the tab is closed

// Mistake 3: Assuming sessionStorage survives browser restart
// Session storage is cleared when the browser process ends

// Mistake 4: Confusing sessionStorage with localStorage
// They share the same API but have different lifetimes

// Mistake 5: Expecting storage event to fire for sessionStorage changes
// The storage event only fires for localStorage, not sessionStorage
```

### Best Practices

- Use `sessionStorage` for transient tab-specific state only
- Always clear session data when it is no longer needed
- Consider sessionStorage for form drafts (they should clear when tab closes)
- Use sessionStorage for undo/redo stacks (session-scoped)
- Don't rely on sessionStorage for critical data that must persist
- Be aware that sessionStorage is isolated per tab, not per window
- Use `JSON.stringify`/`JSON.parse` for objects and arrays

### Performance Considerations

`sessionStorage` performance is identical to `localStorage`—operations are synchronous and fast for typical use cases. Since sessionStorage typically stores less data than localStorage, performance issues are rare. The isolation per tab means each tab uses its own memory, so many tabs with large sessionStorage data can add up.

### Interview Questions

1. How does `sessionStorage` differ from `localStorage`?
2. When does `sessionStorage` data get deleted?
3. Is `sessionStorage` shared between browser tabs?
4. Can `sessionStorage` survive a page refresh?
5. What happens to `sessionStorage` when you duplicate a tab?
6. Does the storage event fire for sessionStorage changes?
7. How much data can sessionStorage hold?
8. Is sessionStorage isolated per tab or per window?

### Coding Challenges

1. **Session-Based Form Recovery**: Build a form that auto-saves to sessionStorage and recovers on crash
2. **Tab-Specific Preferences**: Implement a dashboard where each tab has its own layout state
3. **Session Undo/Redo**: Implement undo/redo using sessionStorage
4. **Cross-Frame Sync**: Sync sessionStorage across iframes in the same page

### Related Topics

- [localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
- [StorageEvent](https://developer.mozilla.org/en-US/docs/Web/API/StorageEvent)

## Differences from localStorage

### What It Is

While `sessionStorage` and `localStorage` share the same API, they differ fundamentally in lifetime, scope, and use cases. Understanding these differences is critical for choosing the right storage mechanism for a given task.

```javascript
// Same API, different behavior
localStorage.setItem('key', 'persistent');    // Survives tab close
sessionStorage.setItem('key', 'transient');    // Lost on tab close

// localStorage is origin-scoped
// sessionStorage is tab-scoped within an origin
```

### Why It Is Important

Choosing the wrong storage type leads to data persistence issues. Using `localStorage` for session-only data pollutes long-term storage and can lead to stale data. Using `sessionStorage` for data that needs to persist across sessions causes data loss. Understanding the trade-offs ensures the right choice for each use case.

### How It Works Internally

`localStorage` writes data to disk and persists it across browser sessions. `sessionStorage` keeps data in memory (with optional disk backing for tab restoration features). The browser maintains separate storage areas for each type, each with its own quota.

```javascript
// Internal storage areas (conceptual):
// Origin: https://example.com
//   ├── localStorage: { theme: 'dark', user: {...} }  (on disk)
//   └── sessionStorage (Tab 1): { draft: '...' }      (in memory)
//   └── sessionStorage (Tab 2): { draft: '...' }      (separate, in memory)
//   └── sessionStorage (Tab 3): { draft: '...' }      (separate, in memory)
```

### Syntax

```javascript
// Both use the exact same interface
// Only the backing store differs

// localStorage - persistent across sessions
window.localStorage.setItem('key', 'value');
window.localStorage.getItem('key');

// sessionStorage - persists only for current session
window.sessionStorage.setItem('key', 'value');
window.sessionStorage.getItem('key');

// Availability check (same for both)
function isStorageAvailable(type) {
  try {
    const storage = window[type];
    const test = '__test__';
    storage.setItem(test, test);
    storage.removeItem(test);
    return true;
  } catch {
    return false;
  }
}
```

### Beginner Examples

```javascript
// Example 1: Lifetime demonstration
// localStorage persists across browser restarts
localStorage.setItem('lastLogin', Date.now());

// sessionStorage is lost when tab is closed
sessionStorage.setItem('sessionStart', Date.now());

// On page load:
document.addEventListener('DOMContentLoaded', () => {
  const lastLogin = localStorage.getItem('lastLogin');
  const sessionStart = sessionStorage.getItem('sessionStart');

  if (lastLogin) {
    console.log('Welcome back! Last login:', new Date(parseInt(lastLogin)));
  }

  if (sessionStart) {
    console.log('Session started at:', new Date(parseInt(sessionStart)));
  } else {
    // New session (or session was restored)
    sessionStorage.setItem('sessionStart', Date.now());
  }
});

// Example 2: Choosing the right storage
// Use localStorage for:
localStorage.setItem('theme', 'dark'); // User preference (persist)

// Use sessionStorage for:
sessionStorage.setItem('draft_id', '123'); // Form draft (transient)
sessionStorage.setItem('current_step', '3'); // Wizard progress

// Example 3: Migration from session to local
function promoteToLocal(key) {
  const value = sessionStorage.getItem(key);
  if (value !== null) {
    localStorage.setItem(key, value);
    sessionStorage.removeItem(key);
  }
}

// Promote session data to persistent storage on "Save for later"
document.querySelector('#save-for-later').addEventListener('click', () => {
  const draft = sessionStorage.getItem('checkout_draft');
  if (draft) {
    localStorage.setItem('saved_draft', draft);
    sessionStorage.removeItem('checkout_draft');
    showToast('Draft saved for later');
  }
});
```

### Intermediate Examples

```javascript
// Example 1: Hybrid storage with fallback
class HybridStorage {
  constructor(key) {
    this.key = key;
  }

  set(value, persistent = false) {
    // Store in both, but control persistence via the flag
    const serialized = JSON.stringify(value);

    if (persistent) {
      localStorage.setItem(this.key, serialized);
    }
    sessionStorage.setItem(this.key, serialized);
  }

  get() {
    // Try session first, then local
    let raw = sessionStorage.getItem(this.key);
    if (raw !== null) {
      try { return JSON.parse(raw); } catch { return raw; }
    }

    raw = localStorage.getItem(this.key);
    if (raw !== null) {
      try { return JSON.parse(raw); } catch { return raw; }
    }

    return null;
  }

  promoteToPersistent() {
    const value = this.get();
    if (value !== null) {
      localStorage.setItem(this.key, JSON.stringify(value));
    }
  }

  remove() {
    sessionStorage.removeItem(this.key);
    localStorage.removeItem(this.key);
  }
}

const draft = new HybridStorage('document_draft');
draft.set({ content: 'Hello' }); // Session only
draft.set({ content: 'Hello World' }, true); // Session + Local
draft.promoteToPersistent(); // Promote session data to local

// Example 2: Storage type detection
function getStorageType() {
  const testKey = '_storage_type_test_';
  const testValue = 'test';

  localStorage.setItem(testKey, testValue);
  sessionStorage.setItem(testKey, testValue);

  const localBefore = localStorage.getItem(testKey);
  const sessionBefore = sessionStorage.getItem(testKey);

  // Simulate session end (we can only clear sessionStorage programmatically)
  sessionStorage.removeItem(testKey);

  const localAfter = localStorage.getItem(testKey);
  const sessionAfter = sessionStorage.getItem(testKey);

  localStorage.removeItem(testKey);

  return {
    localStoragePersists: localAfter === testValue, // Should be true
    sessionStorageIsolated: sessionAfter === null    // Should be true
  };
}

// console.log(getStorageType());

// Example 3: Synchronizing between storage types
class StorageSync {
  constructor(keys = []) {
    this.keys = keys;
    this.setupListener();
  }

  setupListener() {
    window.addEventListener('storage', (event) => {
      if (this.keys.includes(event.key)) {
        // localStorage changed in another tab, sync to this session
        if (event.newValue !== null) {
          sessionStorage.setItem(event.key, event.newValue);
        } else {
          sessionStorage.removeItem(event.key);
        }
      }
    });
  }

  set(key, value, persistent = false) {
    if (persistent) {
      localStorage.setItem(key, JSON.stringify(value));
    }
    sessionStorage.setItem(key, JSON.stringify(value));
  }

  get(key) {
    let raw = sessionStorage.getItem(key);
    if (raw === null) {
      raw = localStorage.getItem(key);
    }
    if (raw !== null) {
      try { return JSON.parse(raw); } catch { return raw; }
    }
    return null;
  }
}

const sync = new StorageSync(['theme', 'language']);
sync.set('theme', 'dark', true); // Store in both
```

### Advanced Examples

```javascript
// Example 1: Unified storage abstraction
class StorageAbstraction {
  constructor(type = 'auto') {
    this.type = type;
    this.stores = {
      local: typeof localStorage !== 'undefined' ? localStorage : null,
      session: typeof sessionStorage !== 'undefined' ? sessionStorage : null
    };
  }

  getStore(persistent) {
    if (this.type === 'local') return this.stores.local;
    if (this.type === 'session') return this.stores.session;

    // Auto mode: use local for persistent, session for transient
    return persistent ? this.stores.local : this.stores.session;
  }

  setItem(key, value, options = {}) {
    const store = this.getStore(options.persistent);
    if (!store) return false;

    const item = {
      value,
      type: typeof value,
      timestamp: Date.now(),
      persistent: options.persistent || false
    };

    try {
      store.setItem(key, JSON.stringify(item));
      return true;
    } catch {
      return false;
    }
  }

  getItem(key) {
    // Try session first, then local
    for (const store of [this.stores.session, this.stores.local]) {
      if (!store) continue;
      const raw = store.getItem(key);
      if (raw !== null) {
        try {
          const item = JSON.parse(raw);
          return item.value;
        } catch {
          return raw;
        }
      }
    }
    return null;
  }

  removeItem(key) {
    if (this.stores.local) this.stores.local.removeItem(key);
    if (this.stores.session) this.stores.session.removeItem(key);
  }

  clear() {
    if (this.stores.local) this.stores.local.clear();
    if (this.stores.session) this.stores.session.clear();
  }

  get length() {
    // Combined unique keys count
    const keys = new Set();
    for (const store of [this.stores.session, this.stores.local]) {
      if (!store) continue;
      for (let i = 0; i < store.length; i++) {
        keys.add(store.key(i));
      }
    }
    return keys.size;
  }
}

const storage = new StorageAbstraction('auto');
storage.setItem('preference', 'dark', { persistent: true }); // Goes to localStorage
storage.setItem('session_data', { step: 3 }, { persistent: false }); // Goes to sessionStorage

// Example 2: Storage profiling tool
class StorageProfiler {
  constructor() {
    this.results = { local: {}, session: {} };
  }

  analyze(type) {
    const store = window[`${type}Storage`];
    if (!store) return;

    let totalSize = 0;
    let largestKey = '';
    let largestSize = 0;
    const itemCount = store.length;
    const keys = [];

    for (let i = 0; i < store.length; i++) {
      const key = store.key(i);
      const value = store.getItem(key);
      const size = key.length + value.length;
      totalSize += size;

      keys.push({ key, size });

      if (size > largestSize) {
        largestSize = size;
        largestKey = key;
      }
    }

    this.results[type] = {
      itemCount,
      totalSize,
      totalKB: (totalSize / 1024).toFixed(2),
      largestKey,
      largestSizeKB: (largestSize / 1024).toFixed(2),
      keys: keys.sort((a, b) => b.size - a.size)
    };
  }

  compare() {
    this.analyze('local');
    this.analyze('session');

    console.log('=== Storage Comparison ===');
    console.log('localStorage:', this.results.local);
    console.log('sessionStorage:', this.results.session);

    if (this.results.local.totalSize > this.results.session.totalSize) {
      console.log('localStorage is more utilized');
    } else {
      console.log('sessionStorage is more utilized');
    }
  }
}

// new StorageProfiler().compare();

// Example 3: Automatic storage demotion/promotion
class SmartStorage {
  constructor(options = {}) {
    this.maxSessionAge = options.maxSessionAge || 3600000; // 1 hour
    this.maxLocalAge = options.maxLocalAge || 86400000 * 7; // 7 days
  }

  set(key, value) {
    const item = {
      value,
      created: Date.now(),
      lastAccess: Date.now()
    };

    sessionStorage.setItem(key, JSON.stringify(item));
  }

  get(key) {
    let raw = sessionStorage.getItem(key);

    // Promote from local if not in session
    if (raw === null) {
      raw = localStorage.getItem(key);
      if (raw !== null) {
        try {
          const item = JSON.parse(raw);
          const age = Date.now() - item.lastAccess;

          if (age < this.maxLocalAge) {
            // Promote to session storage
            item.lastAccess = Date.now();
            sessionStorage.setItem(key, JSON.stringify(item));
            localStorage.setItem(key, JSON.stringify(item));
            return item.value;
          } else {
            // Expired
            localStorage.removeItem(key);
            return null;
          }
        } catch {
          return null;
        }
      }
      return null;
    }

    try {
      const item = JSON.parse(raw);
      const age = Date.now() - item.lastAccess;

      if (age > this.maxSessionAge) {
        // Demote to localStorage for longer-term
        item.lastAccess = Date.now();
        localStorage.setItem(key, JSON.stringify(item));
        sessionStorage.removeItem(key);
        return item.value;
      }

      item.lastAccess = Date.now();
      sessionStorage.setItem(key, JSON.stringify(item));
      return item.value;
    } catch {
      return null;
    }
  }

  remove(key) {
    sessionStorage.removeItem(key);
    localStorage.removeItem(key);
  }
}
```

### Real-World Use Cases

- **localStorage for**: User preferences, authentication tokens, cached API data, offline data
- **sessionStorage for**: Form drafts, wizard progress, tab-specific UI state, temporary filters, undo stacks
- **Both**: Use sessionStorage as a fast cache, localStorage as persistent fallback

### Common Mistakes

```javascript
// Mistake 1: Using localStorage when sessionStorage is more appropriate
localStorage.setItem('search_query', 'term'); // Should this persist for days?

// Mistake 2: Expecting storage event for sessionStorage
window.addEventListener('storage', (e) => {
  // Only fires for localStorage changes, not sessionStorage
});

// Mistake 3: Not clearing session data when appropriate
// sessionStorage auto-clears on tab close, but explicit cleanup is still good practice

// Mistake 4: Mixing keys between storage types
localStorage.setItem('config', 'value1');
sessionStorage.setItem('config', 'value2');
// Both exist independently, no conflict
```

### Best Practices

- Use `localStorage` for data that should survive browser restart
- Use `sessionStorage` for data that should only survive page refreshes
- Never store sensitive data in either (no encryption)
- Be explicit about which storage type you need
- Consider using `sessionStorage` + `localStorage` together with a promotion strategy
- Clear sessionStorage when data is explicitly committed (e.g., form submitted)

### Performance Considerations

Both storage types have identical performance characteristics for individual operations. `sessionStorage` may be slightly faster for reads in some browsers because it's memory-backed. Since sessionStorage data is transient, it typically stores less data and has fewer performance concerns.

### Interview Questions

1. What are the key differences between sessionStorage and localStorage?
2. When would you use sessionStorage instead of localStorage?
3. Can sessionStorage and localStorage share data?
4. What happens to sessionStorage when a page is restored from the browser's tab restore feature?
5. Does sessionStorage survive browser crashes?
6. What is the storage limit for each?
7. How do you check which storage type is available?
8. Can you listen for changes in sessionStorage across tabs?

### Coding Challenges

1. **Smart Hybrid Storage**: Build a storage system that promotes session data to local on demand
2. **Storage Migrator**: Create a tool that migrates data between session and local storage
3. **Performance Comparator**: Build a benchmark that compares read/write speeds of both storage types
4. **Session-Aware Cache**: Implement a cache that uses sessionStorage with localStorage fallback

### Related Topics

- [localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache)

## Tab-Scoped Storage

### What It Is

Tab-scoped storage refers to the isolation of `sessionStorage` data within a single browser tab or window. Each tab has its own independent `sessionStorage` area that is not accessible from other tabs, even if they load the exact same page from the same origin. This isolation extends to iframes within the same tab.

```javascript
// Tab 1 opens example.com
sessionStorage.setItem('tabId', 'Tab 1');

// Tab 2 opens example.com
sessionStorage.getItem('tabId'); // null (different tab, different session)
sessionStorage.setItem('tabId', 'Tab 2');

// Both tabs have independent 'tabId' values
```

### Why It Is Important

Tab-scoped storage is essential for applications where users may have multiple instances open simultaneously. Without it, different tabs would interfere with each other's state. This isolation enables multi-tab workflows where each tab independently manages its own state.

### How It Works Internally

The browser assigns each tab a unique internal identifier. When `sessionStorage` is accessed, the browser routes the request to the storage area associated with that tab's identifier. This identifier is not accessible from JavaScript. When a tab is duplicated (e.g., via Ctrl+click or middle-click), some browsers may copy the sessionStorage from the source tab to the new tab.

```javascript
// Tab duplication behavior:
// Chrome: sessionStorage is copied to new tab when duplicating
// Firefox: sessionStorage is copied to new tab when duplicating
// Safari: sessionStorage is NOT copied (new tab gets empty storage)
// Manual Ctrl+C / Ctrl+T: sessionStorage is NOT copied (new empty session)
```

### Syntax

```javascript
// No special syntax - sessionStorage is automatically tab-scoped
sessionStorage.setItem('tabData', 'isolated');

// In a different tab, this data is not accessible
// Even if the same origin and same page

// Detect if we're in a new tab vs duplicated tab
function isNewSession() {
  return sessionStorage.getItem('_session_init') === null;
}

function initSession() {
  sessionStorage.setItem('_session_init', 'true');
}
```

### Beginner Examples

```javascript
// Example 1: Tab-specific counter
let tabCounter = parseInt(sessionStorage.getItem('tabCounter') || '0', 10);
tabCounter++;
sessionStorage.setItem('tabCounter', String(tabCounter));
console.log(`This tab has been refreshed ${tabCounter} times`);

// Example 2: Independent tab state
document.querySelectorAll('.filter').forEach(filter => {
  filter.addEventListener('change', () => {
    // Each tab maintains its own filter state
    const filters = getFilters();
    sessionStorage.setItem('activeFilters', JSON.stringify(filters));
    applyFilters(filters);
  });
});

// Example 3: Preventing duplicate tabs
if (sessionStorage.getItem('isActive') === 'true') {
  console.warn('This tab is already open in another tab');
  // Option: focus the other tab via BroadcastChannel
} else {
  sessionStorage.setItem('isActive', 'true');
  window.addEventListener('beforeunload', () => {
    sessionStorage.removeItem('isActive');
  });
}

// Note: This doesn't actually prevent duplicates since each tab has its own session
// Use BroadcastChannel or SharedWorker for cross-tab communication
```

### Intermediate Examples

```javascript
// Example 1: Cross-tab communication despite isolation
class CrossTabChannel {
  constructor(channelName = 'app-channel') {
    this.channel = new BroadcastChannel(channelName);
    this.listeners = new Map();
    this.setup();
  }

  setup() {
    this.channel.addEventListener('message', (event) => {
      const handlers = this.listeners.get(event.data.type) || [];
      for (const handler of handlers) {
        handler(event.data.payload, event.data.source);
      }
    });
  }

  send(type, payload) {
    this.channel.postMessage({
      type,
      payload,
      source: {
        tabId: sessionStorage.getItem('tabId'),
        timestamp: Date.now()
      }
    });
  }

  on(type, handler) {
    if (!this.listeners.has(type)) {
      this.listeners.set(type, []);
    }
    this.listeners.get(type).push(handler);
    return () => {
      const handlers = this.listeners.get(type);
      if (handlers) {
        const idx = handlers.indexOf(handler);
        if (idx >= 0) handlers.splice(idx, 1);
      }
    };
  }

  close() {
    this.channel.close();
  }
}

// Example 2: Assigning unique tab IDs
function generateTabId() {
  let tabId = sessionStorage.getItem('tabId');
  if (!tabId) {
    tabId = `tab_${Date.now()}_${Math.random().toString(36).slice(2, 8)}`;
    sessionStorage.setItem('tabId', tabId);
  }
  return tabId;
}

// Share tab ID with the page title for debugging
document.title = `${document.title} [${generateTabId().slice(-6)}]`;

// Example 3: Tab-scoped session logging
class TabScopedLogger {
  constructor() {
    this.tabId = generateTabId();
    this.logKey = `logs_${this.tabId}`;
    this.logs = this.load();
  }

  load() {
    try {
      return JSON.parse(sessionStorage.getItem(this.logKey)) || [];
    } catch {
      return [];
    }
  }

  save() {
    sessionStorage.setItem(this.logKey, JSON.stringify(this.logs));
  }

  log(level, message, data = {}) {
    const entry = {
      timestamp: Date.now(),
      level,
      message,
      data,
      tabId: this.tabId
    };

    this.logs.push(entry);
    if (this.logs.length > 100) {
      this.logs.shift();
    }
    this.save();

    console.log(`[Tab ${this.tabId}] ${level}: ${message}`, data);
  }

  debug(msg, data) { this.log('DEBUG', msg, data); }
  info(msg, data) { this.log('INFO', msg, data); }
  warn(msg, data) { this.log('WARN', msg, data); }
  error(msg, data) { this.log('ERROR', msg, data); }

  getLogs() {
    return [...this.logs];
  }

  clear() {
    this.logs = [];
    sessionStorage.removeItem(this.logKey);
  }
}

const logger = new TabScopedLogger();
logger.info('Page loaded', { url: window.location.href });
```

### Advanced Examples

```javascript
// Example 1: Tab-synced session state via BroadcastChannel
class TabSyncState {
  constructor(namespace = 'app') {
    this.namespace = namespace;
    this.tabId = generateTabId();
    this.state = {};
    this.listeners = new Map();
    this.channel = new BroadcastChannel(`${namespace}-sync`);
    this.setup();
  }

  setup() {
    this.channel.addEventListener('message', (event) => {
      const { type, tabId, key, value, timestamp } = event.data;

      if (tabId === this.tabId) return; // Ignore own messages

      if (type === 'STATE_UPDATE') {
        this.state[key] = { value, timestamp };
        this.notify(key, value, 'remote');
      } else if (type === 'STATE_DELETE') {
        delete this.state[key];
        this.notify(key, null, 'deleted');
      } else if (type === 'STATE_REQUEST') {
        // Another tab requests current state
        this.channel.postMessage({
          type: 'STATE_RESPONSE',
          tabId: this.tabId,
          state: this.getPublicState(),
          timestamp: Date.now()
        });
      } else if (type === 'STATE_RESPONSE') {
        // Merge state from other tab
        for (const [key, { value, timestamp }] of Object.entries(event.data.state)) {
          if (!this.state[key] || timestamp > this.state[key].timestamp) {
            this.state[key] = { value, timestamp };
            this.notify(key, value, 'synced');
          }
        }
      }
    });

    // Request state from other tabs on connect
    this.channel.postMessage({
      type: 'STATE_REQUEST',
      tabId: this.tabId,
      timestamp: Date.now()
    });
  }

  set(key, value) {
    this.state[key] = { value, timestamp: Date.now() };
    sessionStorage.setItem(`${this.namespace}_${key}`, JSON.stringify(value));
    this.notify(key, value, 'local');

    this.channel.postMessage({
      type: 'STATE_UPDATE',
      tabId: this.tabId,
      key,
      value,
      timestamp: Date.now()
    });
  }

  get(key) {
    if (this.state[key]) return this.state[key].value;

    // Try session storage
    const raw = sessionStorage.getItem(`${this.namespace}_${key}`);
    if (raw !== null) {
      try {
        const value = JSON.parse(raw);
        this.state[key] = { value, timestamp: 0 };
        return value;
      } catch {
        return raw;
      }
    }

    return null;
  }

  getPublicState() {
    const result = {};
    for (const [key, { value, timestamp }] of Object.entries(this.state)) {
      result[key] = { value, timestamp };
    }
    return result;
  }

  onChange(key, handler) {
    if (!this.listeners.has(key)) {
      this.listeners.set(key, []);
    }
    this.listeners.get(key).push(handler);
    return () => {
      const handlers = this.listeners.get(key);
      if (handlers) {
        const idx = handlers.indexOf(handler);
        if (idx >= 0) handlers.splice(idx, 1);
      }
    };
  }

  notify(key, value, source) {
    const handlers = this.listeners.get(key) || [];
    const wildcardHandlers = this.listeners.get('*') || [];
    for (const handler of [...handlers, ...wildcardHandlers]) {
      handler(key, value, source);
    }
  }

  close() {
    this.channel.close();
  }
}

// Example usage in multiple tabs
const syncState = new TabSyncState('dashboard');

syncState.on('theme', (key, value, source) => {
  document.documentElement.className = `theme-${value}`;
  console.log(`Theme changed to ${value} via ${source}`);
});

syncState.set('theme', 'dark');

// Example 2: Tab-scoped authorization
class TabAuth {
  constructor() {
    this.tabId = generateTabId();
    this.authKey = `auth_${this.tabId}`;
  }

  setToken(token) {
    sessionStorage.setItem(this.authKey, token);
  }

  getToken() {
    return sessionStorage.getItem(this.authKey);
  }

  clearToken() {
    sessionStorage.removeItem(this.authKey);
  }

  isAuthenticated() {
    return this.getToken() !== null;
  }

  getAuthHeaders() {
    const token = this.getToken();
    return token ? { Authorization: `Bearer ${token}` } : {};
  }
}

const tabAuth = new TabAuth();
tabAuth.setToken('jwt_token_here');
console.log(tabAuth.isAuthenticated()); // true

// Example 3: Tab-scoped media playback state
class TabMediaPlayer {
  constructor() {
    this.stateKey = 'media_state';
    this.restore();
  }

  save() {
    sessionStorage.setItem(this.stateKey, JSON.stringify({
      currentTrack: this.currentTrack,
      position: this.audio?.currentTime || 0,
      volume: this.audio?.volume || 1,
      muted: this.audio?.muted || false,
      playlist: this.playlist
    }));
  }

  restore() {
    const raw = sessionStorage.getItem(this.stateKey);
    if (raw) {
      try {
        const state = JSON.parse(raw);
        this.currentTrack = state.currentTrack;
        this.playlist = state.playlist;
        // Don't restore position - each tab starts fresh
      } catch {
        this.init();
      }
    } else {
      this.init();
    }
  }

  init() {
    this.currentTrack = null;
    this.playlist = [];
  }

  setTrack(track) {
    this.currentTrack = track;
    this.save();
  }

  addToPlaylist(track) {
    this.playlist.push(track);
    this.save();
  }

  clear() {
    sessionStorage.removeItem(this.stateKey);
    this.init();
  }
}
```

### Real-World Use Cases

- **Multi-Tab Dashboards**: Each tab maintains independent filter/sort state
- **Code Editors**: Each tab has its own undo history and unsaved changes
- **Media Players**: Each tab has independent playlist and playback state
- **Chat Applications**: Each tab handles its own connection and message queue
- **Admin Panels**: Different tabs for editing different records without interference
- **Data Comparison Tools**: Side-by-side data views with independent navigation
- **Form Drafts**: Multiple forms in different tabs without cross-contamination

### Common Mistakes

```javascript
// Mistake 1: Assuming sessionStorage is shared across tabs
// It is NOT - each tab has its own isolated sessionStorage

// Mistake 2: Using sessionStorage for cross-tab communication
// Use BroadcastChannel, SharedWorker, or localStorage (with storage event)

// Mistake 3: Not handling tab duplication
// When a tab is duplicated, sessionStorage may be copied (browser-dependent)

// Mistake 4: Confusing tab scope with window scope
// Opening a new window also creates a new session (same as a new tab)
```

### Best Practices

- Use `sessionStorage` when you need per-tab isolation
- Use `BroadcastChannel` for cross-tab communication
- Assign unique tab IDs for debugging and state tracking
- Handle tab duplication gracefully (expect state may or may not carry over)
- Clean up sessionStorage when data is committed (e.g., form submitted)
- Don't rely on sessionStorage for critical cross-tab coordination

### Performance Considerations

Since each tab has its own sessionStorage, the total memory used scales with the number of open tabs. Applications with large sessionStorage data in many tabs can consume significant memory. Consider using `localStorage` for data that doesn't need tab isolation.

### Interview Questions

1. How is sessionStorage scoped differently from localStorage?
2. Can two tabs of the same origin share sessionStorage?
3. What happens to sessionStorage when you duplicate a tab?
4. How would you communicate between two tabs?
5. Is sessionStorage scoped per tab or per window?
6. What happens to an iframe's sessionStorage?
7. How do different browsers handle tab duplication with sessionStorage?
8. How can you give each tab a unique ID?

### Coding Challenges

1. **Multi-Tab State Sync**: Build a system that syncs state across tabs while respecting tab isolation
2. **Tab Manager**: Create a tab-scoped state management system with unique tab IDs
3. **Cross-Tab Channel**: Implement a communication system using BroadcastChannel that respects sessionStorage
4. **Duplicate Tab Detector**: Build a system that detects when a tab is duplicated and syncs appropriate state

### Related Topics

- [BroadcastChannel API](https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel)
- [SharedWorker](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker)
- [StorageEvent](https://developer.mozilla.org/en-US/docs/Web/API/StorageEvent)
- [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
