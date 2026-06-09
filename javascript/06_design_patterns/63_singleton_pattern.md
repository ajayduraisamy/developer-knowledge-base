# Singleton Pattern - Single instance, module singleton, instance management

## Introduction

The Singleton Pattern ensures that a class or factory produces only one instance and provides a global point of access to that instance. In JavaScript, singletons manifest in multiple forms: the classic class-based singleton, the module-as-singleton pattern (where ES6 modules naturally behave as singletons), and various instance management strategies.

While controversial in some engineering circles (often criticized for introducing global state), singletons are essential for managing shared resources like configuration objects, database connections, loggers, and caches when used judiciously.

## Single Instance Guarantee

### What It Is

A single instance guarantee means that no matter how many times you request an instance from a constructor or factory, you always receive the same object. The creation logic runs only once; subsequent calls return the cached instance.

### Why It Is Important

Shared resources like database connections, configuration objects, and logging services must maintain a single state across the application. Creating multiple instances of a database connection pool would exhaust server connections. Multiple configuration objects could lead to inconsistent settings. The singleton pattern prevents these issues by enforcing centralized, consistent state.

### How It Works Internally

A singleton typically uses:
1. A private variable to hold the instance (often a closure variable)
2. A conditional check: if the instance exists, return it; otherwise create it
3. Prevention of external instantiation (e.g., throwing on `new` calls)

The instance is stored in the closure or as a static property, surviving for the lifetime of the application (or module).

### Syntax

```javascript
// Classic singleton with closure
const Singleton = (function() {
  let instance;

  function createInstance() {
    return { /* ... */ };
  }

  return {
    getInstance() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

// ES6 class singleton
class SingletonClass {
  static #instance;

  constructor() {
    if (SingletonClass.#instance) {
      return SingletonClass.#instance;
    }
    SingletonClass.#instance = this;
  }

  static getInstance() {
    if (!SingletonClass.#instance) {
      SingletonClass.#instance = new SingletonClass();
    }
    return SingletonClass.#instance;
  }
}
```

### Beginner Examples

```javascript
// Simple counter singleton
function createCounterSingleton() {
  if (createCounterSingleton.instance) {
    return createCounterSingleton.instance;
  }

  let count = 0;

  const instance = {
    increment() { return ++count; },
    decrement() { return --count; },
    getCount() { return count; },
    reset() { count = 0; }
  };

  createCounterSingleton.instance = instance;
  return instance;
}

const counter1 = createCounterSingleton();
const counter2 = createCounterSingleton();
counter1.increment();
counter1.increment();
console.log(counter2.getCount()); // 2 (same instance)
console.log(counter1 === counter2); // true

// Logger singleton
class Logger {
  static #instance;
  #logs = [];

  constructor() {
    if (Logger.#instance) {
      return Logger.#instance;
    }
    Logger.#instance = this;
  }

  static getInstance() {
    if (!Logger.#instance) {
      Logger.#instance = new Logger();
    }
    return Logger.#instance;
  }

  log(level, message) {
    const entry = { level, message, timestamp: new Date().toISOString() };
    this.#logs.push(entry);
    console.log(`[${level}] ${message}`);
  }

  getLogs() { return [...this.#logs]; }
}

const logger1 = Logger.getInstance();
const logger2 = new Logger(); // Returns logger1
logger1.log("INFO", "App started");
console.log(logger2.getLogs().length); // 1
```

### Intermediate Examples

```javascript
// Config singleton with lazy loading and defaults
class AppConfig {
  static #instance;
  #config = {};
  #initialized = false;

  constructor() {
    if (AppConfig.#instance) {
      return AppConfig.#instance;
    }
    AppConfig.#instance = this;
  }

  static getInstance() {
    if (!AppConfig.#instance) {
      AppConfig.#instance = new AppConfig();
    }
    return AppConfig.#instance;
  }

  async init(configPath) {
    if (this.#initialized) return;

    const defaults = {
      port: 3000,
      host: "localhost",
      debug: false,
      logLevel: "info"
    };

    try {
      const response = await fetch(configPath);
      const userConfig = await response.json();
      this.#config = { ...defaults, ...userConfig };
      this.#initialized = true;
    } catch (err) {
      console.warn("Config load failed, using defaults");
      this.#config = defaults;
      this.#initialized = true;
    }
  }

  get(key) {
    return this.#config[key];
  }

  set(key, value) {
    this.#config[key] = value;
  }

  getAll() {
    return { ...this.#config };
  }
}

// Usage
async function bootstrap() {
  const config = AppConfig.getInstance();
  await config.init("/config.json");
  console.log(config.get("port")); // 3000 or loaded value
}

// Database connection singleton
class DatabaseConnection {
  static #instance;
  #connection = null;
  #connected = false;

  constructor() {
    if (DatabaseConnection.#instance) {
      return DatabaseConnection.#instance;
    }
    DatabaseConnection.#instance = this;
  }

  static getInstance() {
    if (!DatabaseConnection.#instance) {
      DatabaseConnection.#instance = new DatabaseConnection();
    }
    return DatabaseConnection.#instance;
  }

  async connect(url, options = {}) {
    if (this.#connected) return this.#connection;

    const { Pool } = require("pg");
    this.#connection = new Pool({ connectionString: url, ...options });
    this.#connected = true;
    console.log("Database connected");
    return this.#connection;
  }

  async query(text, params) {
    if (!this.#connected) throw new Error("Not connected to database");
    return this.#connection.query(text, params);
  }

  async close() {
    if (this.#connected) {
      await this.#connection.end();
      this.#connected = false;
      this.#connection = null;
    }
  }
}
```

### Advanced Examples

```javascript
// Singleton with destruction/recreation support
class EphemeralSingleton {
  static #instance;
  static #destroyed = false;
  #id = Date.now();

  constructor() {
    if (EphemeralSingleton.#instance && !EphemeralSingleton.#destroyed) {
      return EphemeralSingleton.#instance;
    }
    EphemeralSingleton.#instance = this;
    EphemeralSingleton.#destroyed = false;
  }

  static getInstance() {
    if (!EphemeralSingleton.#instance || EphemeralSingleton.#destroyed) {
      new EphemeralSingleton();
    }
    return EphemeralSingleton.#instance;
  }

  getId() { return this.#id; }

  destroy() {
    EphemeralSingleton.#destroyed = true;
    EphemeralSingleton.#instance = null;
  }
}

// Singleton registry for managing multiple singletons
class SingletonRegistry {
  static #instances = new Map();

  static register(name, factory) {
    if (!this.#instances.has(name)) {
      this.#instances.set(name, factory());
    }
    return this.#instances.get(name);
  }

  static get(name) {
    return this.#instances.get(name) || null;
  }

  static remove(name) {
    this.#instances.delete(name);
  }

  static clear() {
    this.#instances.clear();
  }

  static getAll() {
    return new Map(this.#instances);
  }
}

// Usage
const db = SingletonRegistry.register("db", () => ({
  query: async (sql) => console.log(`Executing: ${sql}`)
}));

const cache = SingletonRegistry.register("cache", () => new Map());

// Async initialization singleton
class AsyncSingleton {
  static #instance;
  static #initializing = null;
  #data = null;

  constructor() {
    if (AsyncSingleton.#instance) {
      return AsyncSingleton.#instance;
    }
    AsyncSingleton.#instance = this;
  }

  static async getInstance() {
    if (AsyncSingleton.#instance?.isReady()) {
      return AsyncSingleton.#instance;
    }

    if (!AsyncSingleton.#initializing) {
      AsyncSingleton.#initializing = (async () => {
        const instance = new AsyncSingleton();
        await instance.init();
        return instance;
      })();
    }

    return AsyncSingleton.#initializing;
  }

  async init() {
    // Simulate async initialization
    await new Promise(resolve => setTimeout(resolve, 100));
    this.#data = { initialized: true, timestamp: Date.now() };
  }

  isReady() {
    return this.#data !== null;
  }

  getData() {
    return this.#data;
  }
}
```

### Real-World Use Cases

- **Database connection pools**: One pool per database configuration.
- **Application configuration**: Single source of truth for app settings.
- **Logger services**: Centralized logging with consistent formatting and transport.
- **Cache instances**: In-memory caches (like `Map`-based caches) should be shared.
- **State management stores**: Redux store, Zustand store — singletons by design.
- **Service workers**: One service worker registration per scope.

### Common Mistakes

```javascript
// Mistake 1: Singleton that's not thread-safe (relevant for Node.js Worker Threads)
// Not a concern for single-threaded JS, but be aware with SharedArrayBuffer

// Mistake 2: Accidental instance creation via new
class LeakySingleton {
  constructor() {
    // Missing the check - each `new` creates a new instance
  }
}

// Mistake 3: Singletons in tests cause state leakage between tests
// Solution: Provide a reset method or use dependency injection

// Mistake 4: Mutating singleton state unintentionally
// Freeze the object if immutability is desired
const config = Object.freeze({ port: 3000 });

// Mistake 5: Circular dependencies with singletons
// Module A imports Module B singleton, Module B imports Module A singleton
```

### Best Practices

```javascript
// Use sparingly - singletons are global state in disguise
// Only use for truly shared resources

// Provide a reset method for testing
class TestableSingleton {
  static #instance;
  #state = {};

  constructor() {
    if (TestableSingleton.#instance) return TestableSingleton.#instance;
    TestableSingleton.#instance = this;
  }

  static getInstance() {
    if (!TestableSingleton.#instance) new TestableSingleton();
    return TestableSingleton.#instance;
  }

  static reset() {
    TestableSingleton.#instance = null;
  }
}

// Prefer dependency injection over direct singleton access
class ServiceConsumer {
  constructor(logger) {
    this.logger = logger; // Inject logger instead of calling Logger.getInstance()
  }
}

// Make singletons immutable where possible
// Use Object.freeze() for configuration objects
```

### Performance Considerations

- Singleton initialization cost is paid once; all subsequent accesses are O(1) lookups.
- Singletons can become memory leaks if they accumulate data without cleanup.
- Lazy initialization (creating the instance on first use) improves startup time.
- Be mindful of the singleton's lifecycle: if it holds connections or file handles, ensure proper cleanup.

### Interview Questions

**Q: Are ES6 modules singletons?**
A: Yes, ES6 modules are inherently singletons. A module is evaluated only once, and all subsequent imports reference the same module instance. This means any state defined at the module's top level is shared across all importers, making ES6 modules a natural singleton implementation without any extra code.

**Q: How do you test code that uses singletons?**
A: Singletons make testing difficult because they retain state between tests. Solutions include: (1) providing a `reset()` method on the singleton, (2) using dependency injection so the singleton can be mocked, (3) using test doubles that replace the singleton module.

**Q: What is the difference between a singleton and a static class?**
A: A singleton has an instance that can be passed around, implements interfaces, and can be extended. A static class cannot be instantiated, cannot be passed as a reference, and has no instance state. Singletons support lazy initialization; static classes are eagerly loaded.

### Coding Challenges

```javascript
// Challenge 1: Implement a singleton cache that stores key-value pairs
// with a TTL (time-to-live). Ensure all access goes through the same instance.

// Challenge 2: Create a singleton event bus that handles subscriptions and
// can be reset completely between test runs.

// Challenge 3: Build a ConnectionPool singleton that manages up to N database
// connections, implementing acquire() and release() methods.
```

### Related Topics

- Module pattern (singletons via closures)
- Factory pattern (singletons are a special case of factory)
- Dependency injection (alternative to singletons)
- Global state management

## Module as Singleton

### What It Is

In the ES6 module system, every module is evaluated exactly once. When multiple modules import from the same module, they all receive a reference to the same module instance. This makes every ES6 module a natural singleton without any special pattern.

### Why It Is Important

The module-as-singleton pattern eliminates boilerplate for creating singletons. You simply define state at the module level, export the public API, and the module system handles the singleton guarantee. This is the idiomatic way to create singletons in modern JavaScript applications.

### How It Works Internally

The JavaScript engine maintains a module map. When a module is imported:
1. The engine checks the module map for the requested module URL.
2. If not found, it fetches, parses, and evaluates the module.
3. The module's exports are cached.
4. Subsequent imports of the same URL return the cached exports.

This evaluation happens once, ensuring all consumers share the same module scope and state.

### Syntax

```javascript
// config.js - Module as singleton
let config = {
  port: 3000,
  host: "localhost"
};

export function getConfig() {
  return { ...config };
}

export function setConfig(key, value) {
  config[key] = value;
}

// app.js - Importing the singleton
import { getConfig, setConfig } from "./config.js";

setConfig("port", 8080);
const cfg = getConfig(); // { port: 8080, host: "localhost" }

// anotherModule.js - Same singleton state
import { getConfig } from "./config.js";
console.log(getConfig().port); // 8080 (same instance)
```

### Beginner Examples

```javascript
// counter.js - Module as singleton
let count = 0;

export function increment() { count++; }
export function decrement() { count--; }
export function getCount() { return count; }
export function reset() { count = 0; }

// app.js
import { increment, getCount } from "./counter.js";
import { increment as inc2, getCount as get2 } from "./counter.js";

increment();
increment();
console.log(getCount()); // 2
console.log(get2()); // 2 (same module state)
console.log(increment === inc2); // true (same function reference)

// theme.js - Theme singleton
let currentTheme = "light";
const listeners = new Set();

export function getTheme() {
  return currentTheme;
}

export function setTheme(theme) {
  currentTheme = theme;
  listeners.forEach(fn => fn(theme));
}

export function onThemeChange(fn) {
  listeners.add(fn);
  return () => listeners.delete(fn);
}
```

### Intermediate Examples

```javascript
// store.js - Simple state management singleton
const state = new Map();
const subscribers = new Map();

export function getState(key) {
  return state.get(key);
}

export function setState(key, value) {
  const oldValue = state.get(key);
  state.set(key, value);
  const subs = subscribers.get(key) || [];
  subs.forEach(fn => fn(value, oldValue));
}

export function subscribe(key, fn) {
  if (!subscribers.has(key)) {
    subscribers.set(key, new Set());
  }
  subscribers.get(key).add(fn);
  return () => subscribers.get(key).delete(fn);
}

export function getAllState() {
  return Object.fromEntries(state);
}

// api.js - API client singleton
const BASE_URL = "https://api.example.com/v1";
let authToken = null;
let requestCount = 0;

async function request(endpoint, options = {}) {
  requestCount++;
  const headers = {
    "Content-Type": "application/json",
    ...(authToken ? { Authorization: `Bearer ${authToken}` } : {}),
    ...options.headers
  };

  const response = await fetch(`${BASE_URL}${endpoint}`, {
    ...options,
    headers
  });

  if (!response.ok) {
    throw new ApiError(response.status, await response.text());
  }

  return response.json();
}

export function setToken(token) {
  authToken = token;
}

export function getRequestCount() {
  return requestCount;
}

export const api = {
  get: (endpoint) => request(endpoint),
  post: (endpoint, body) => request(endpoint, { method: "POST", body: JSON.stringify(body) }),
  put: (endpoint, body) => request(endpoint, { method: "PUT", body: JSON.stringify(body) }),
  delete: (endpoint) => request(endpoint, { method: "DELETE" })
};

class ApiError extends Error {
  constructor(status, message) {
    super(message);
    this.status = status;
    this.name = "ApiError";
  }
}
```

### Advanced Examples

```javascript
// db-pool.js - Database pool singleton with connection lifecycle
import { createPool } from "mysql2/promise";

let pool = null;
let config = null;
const poolState = {
  isConnected: false,
  activeConnections: 0,
  totalQueries: 0
};

export async function initializePool(dbConfig) {
  if (pool) {
    throw new Error("Database pool is already initialized");
  }
  config = dbConfig;
  pool = createPool({
    host: dbConfig.host,
    user: dbConfig.user,
    password: dbConfig.password,
    database: dbConfig.database,
    connectionLimit: dbConfig.poolSize || 10,
    waitForConnections: true,
    queueLimit: 0
  });
  poolState.isConnected = true;
  return pool;
}

export async function query(sql, params) {
  if (!pool) throw new Error("Database not initialized");
  poolState.totalQueries++;
  poolState.activeConnections++;
  try {
    const [rows] = await pool.execute(sql, params);
    return rows;
  } finally {
    poolState.activeConnections--;
  }
}

export async function closePool() {
  if (pool) {
    await pool.end();
    pool = null;
    poolState.isConnected = false;
  }
}

export function getPoolStatus() {
  return { ...poolState, config: config ? { ...config, password: "***" } : null };
}

// Feature flag singleton module
const flags = new Map();
const flagChangeListeners = new Set();

async function loadFlags() {
  try {
    const response = await fetch("/api/flags");
    const data = await response.json();
    flags.clear();
    Object.entries(data).forEach(([key, value]) => flags.set(key, value));
    flagChangeListeners.forEach(fn => fn(flags));
  } catch (err) {
    console.error("Failed to load feature flags", err);
  }
}

export function isEnabled(flagName) {
  return flags.get(flagName) === true;
}

export function enableFlag(flagName) {
  flags.set(flagName, true);
  flagChangeListeners.forEach(fn => fn(flags));
}

export function disableFlag(flagName) {
  flags.set(flagName, false);
  flagChangeListeners.forEach(fn => fn(flags));
}

export function getFlags() {
  return Object.fromEntries(flags);
}

export function onFlagChange(fn) {
  flagChangeListeners.add(fn);
  return () => flagChangeListeners.delete(fn);
}

// Auto-initialize
loadFlags();
```

### Real-World Use Cases

- **Redux store**: A single store object is created and shared across the app.
- **Axios instances**: `axios.create()` returns a singleton-like instance per configuration.
- **Prisma client**: `PrismaClient` is instantiated once and shared.
- **Express app**: The Express application object is a singleton configured during startup.
- **i18n/locale state**: Translation strings and locale settings are module-level singletons.

### Common Mistakes

```javascript
// Mistake 1: Assuming module state is fresh on each import
// Module state persists - importing doesn't reset state

// Mistake 2: Mutating imported objects thinking they're local copies
import { config } from "./config.js";
config.port = 8080; // This mutates the singleton! Bad practice

// Mistake 3: Circular imports with module-level initialization
// a.js - import { b } from "./b.js"; export const a = b + 1;
// b.js - import { a } from "./a.js"; export const b = a + 1;
// One will be undefined due to circular dependency

// Mistake 4: Not handling module initialization order
export const value = someFunction(); // Runs when module is first imported
```

### Best Practices

```javascript
// Export functions that encapsulate state rather than exporting state directly
// Bad:
export let config = { port: 3000 };

// Good:
let config = { port: 3000 };
export function getConfig() { return { ...config }; }
export function updateConfig(update) { config = { ...config, ...update }; }

// Use Object.freeze for truly immutable singleton exports
export const IMMUTABLE_CONFIG = Object.freeze({ port: 3000, host: "localhost" });

// Keep module initialization side-effect free or idempotent
// Avoid auto-initialization that could fail
// Instead, export an async init() function that consumers call

// For testability, export a reset function in development
if (process.env.NODE_ENV === "test") {
  export function _resetState() {
    state.clear();
    subscribers.clear();
  }
}
```

### Performance Considerations

- Module singletons have zero overhead — they are the natural behavior of the ES6 module system.
- Module-level variables persist for the application's lifetime, so they don't add GC pressure.
- However, accumulated state in module singletons can grow indefinitely if not managed.

### Interview Questions

**Q: How does an ES6 module achieve singleton behavior without any special code?**
A: ES6 modules are evaluated once per URL. The JavaScript engine caches the module's exports in a module map. When another module imports from the same URL, the engine returns the cached exports, ensuring all consumers share the same state.

**Q: Can module singletons be reset for testing?**
A: By default, no — module state persists across imports. However, you can export a `reset()` function (often guarded by `process.env.NODE_ENV !== "production"`) that clears the module state. Alternatively, use dynamic `import()` which may re-evaluate the module if uncached.

**Q: What's the difference between defining state at module level vs using a class singleton?**
A: Module-level state is simpler and doesn't require any class boilerplate. Class singletons provide more structure, support inheritance, and can implement interfaces. Module-level state is idiomatic in functional JavaScript, while class singletons are more common in OOP-style code.

### Coding Challenges

```javascript
// Challenge 1: Create a module singleton that manages WebSocket connections.
// It should maintain a single connection URL, reconnect logic, and message handlers.
// Export connect(), disconnect(), send(), and onMessage() functions.

// Challenge 2: Build a module-level performance metrics collector singleton.
// Export recordMetric(name, duration), getMetrics(name), getAllMetrics(), and clear().
// The module should collect metrics across all importing files.

// Challenge 3: Create a session management module singleton that stores
// user session data, handles token refresh, and notifies listeners on auth changes.
```

### Related Topics

- ES6 module system
- Module caching and evaluation order
- Dynamic `import()`
- Circular dependencies

## Instance Management

### What It Is

Instance management encompasses strategies for controlling how and when singleton instances are created, stored, accessed, and destroyed. This includes lazy vs eager initialization, instance lifecycle management, thread safety considerations, and cleanup strategies.

### Why It Is Important

Poor instance management leads to resource leaks, stale state, memory bloat, and testing difficulties. Proper instance management ensures that singletons initialize correctly, use resources efficiently, and can be reset between tests or during application lifecycle events.

### How It Works Internally

Instance management is typically implemented via:
- **Closure variables**: Store instance in a function's closure
- **Static class properties**: `static #instance` in ES6 classes
- **Module-level variables**: In ES6 module singletons
- **WeakMap or Map registries**: For tracking instances without preventing GC
- **Proxy-based lazy initialization**: Using `Proxy` to defer creation

### Syntax

```javascript
// Lazy initialization (standard)
const getInstance = (() => {
  let instance;
  return () => {
    if (!instance) instance = createExpensiveObject();
    return instance;
  };
})();

// Eager initialization
const instance = createExpensiveObject(); // Created at module load

// Instance registry (multiple named singletons)
const registry = new Map();
function getOrCreate(name, factory) {
  if (!registry.has(name)) {
    registry.set(name, factory());
  }
  return registry.get(name);
}
```

### Beginner Examples

```javascript
// Eager vs lazy initialization
// Eager: Created immediately when module is loaded
const eagerLogger = {
  log(msg) { console.log(msg); }
};

// Lazy: Created only on first access
function getLazyLogger() {
  if (!getLazyLogger.instance) {
    getLazyLogger.instance = {
      log(msg) { console.log(msg); }
    };
  }
  return getLazyLogger.instance;
}

// Instance with TTL (time-to-live)
function createTTLSingleton(ttl = 60000) {
  let instance = null;
  let lastAccessed = 0;

  return function() {
    const now = Date.now();
    if (!instance || (now - lastAccessed) > ttl) {
      instance = { createdAt: now };
    }
    lastAccessed = now;
    return instance;
  };
}

const getInstance = createTTLSingleton(5000);
```

### Intermediate Examples

```javascript
// Instance pool (limited number of instances, not just one)
class InstancePool {
  constructor(factory, maxSize = 5) {
    this.factory = factory;
    this.maxSize = maxSize;
    this.instances = [];
    this.inUse = new Set();
  }

  acquire() {
    // Try to find a free instance
    const available = this.instances.find(i => !this.inUse.has(i));
    if (available) {
      this.inUse.add(available);
      return available;
    }

    // Create new if under max
    if (this.instances.length < this.maxSize) {
      const instance = this.factory();
      this.instances.push(instance);
      this.inUse.add(instance);
      return instance;
    }

    // Wait for one to be released (simplified - would use a queue in production)
    throw new Error("No available instances");
  }

  release(instance) {
    this.inUse.delete(instance);
  }

  get activeCount() {
    return this.inUse.size;
  }

  get totalCount() {
    return this.instances.length;
  }

  drain() {
    if (this.inUse.size > 0) {
      throw new Error("Cannot drain: instances still in use");
    }
    this.instances = [];
  }
}

// Auto-resetting singleton (resets on certain conditions)
function createAutoResettingSingleton() {
  let instance = null;
  let refCount = 0;
  const RESET_TIMEOUT = 300000; // 5 minutes

  let resetTimer = null;

  function scheduleReset() {
    if (resetTimer) clearTimeout(resetTimer);
    resetTimer = setTimeout(() => {
      if (refCount === 0) {
        instance = null;
        console.log("Singleton auto-reset");
      }
    }, RESET_TIMEOUT);
  }

  return {
    getInstance() {
      refCount++;
      if (resetTimer) clearTimeout(resetTimer);
      if (!instance) {
        instance = { id: Math.random(), createdAt: new Date() };
      }
      return instance;
    },
    release() {
      refCount = Math.max(0, refCount - 1);
      if (refCount === 0) {
        scheduleReset();
      }
    },
    forceReset() {
      instance = null;
      refCount = 0;
      if (resetTimer) clearTimeout(resetTimer);
    }
  };
}
```

### Advanced Examples

```javascript
// Proxy-based lazy singleton with monitoring
function createMonitoredSingleton(factory) {
  let instance = null;
  let accessCount = 0;
  let creationTime = null;

  const handler = {
    get(target, prop) {
      accessCount++;
      if (prop === "__stats") {
        return { accessCount, creationTime, isCreated: instance !== null };
      }
      return Reflect.get(target, prop);
    },
    apply(target, thisArg, args) {
      accessCount++;
      return Reflect.apply(target, thisArg, args);
    }
  };

  return new Proxy({}, {
    get(target, prop) {
      if (prop === "__stats") {
        return { accessCount, creationTime, isCreated: instance !== null };
      }
      if (!instance) {
        instance = factory();
        creationTime = Date.now();
      }
      return instance[prop];
    }
  });
}

// Multi-tenant singleton registry (one singleton per tenant)
class TenantSingletonRegistry {
  #instances = new Map();
  #factories = new Map();

  register(type, factory) {
    this.#factories.set(type, factory);
  }

  getForTenant(tenantId, type) {
    const key = `${tenantId}:${type}`;
    if (!this.#instances.has(key)) {
      const factory = this.#factories.get(type);
      if (!factory) throw new Error(`No factory for type: ${type}`);
      this.#instances.set(key, factory(tenantId));
    }
    return this.#instances.get(key);
  }

  removeForTenant(tenantId) {
    for (const [key] of this.#instances) {
      if (key.startsWith(`${tenantId}:`)) {
        this.#instances.delete(key);
      }
    }
  }

  clearAll() {
    this.#instances.clear();
  }
}

// WeakRef-based singleton (allows GC when no other references exist)
class WeakSingleton {
  static #ref = null;
  static #finalizationGroup = new FinalizationRegistry(() => {
    WeakSingleton.#ref = null;
  });

  constructor() {
    if (WeakSingleton.#ref?.deref()) {
      return WeakSingleton.#ref.deref();
    }
    this.createdAt = Date.now();
    WeakSingleton.#ref = new WeakRef(this);
    WeakSingleton.#finalizationGroup.register(this, this);
  }
}
```

### Real-World Use Cases

- **Connection pooling**: Database connection pools that acquire/release connections.
- **Tenant isolation**: Multi-tenant SaaS apps that maintain one singleton per tenant.
- **Resource cleanup**: Singletons that auto-cleanup after periods of inactivity.
- **Mock/service injection**: Test frameworks that swap singleton implementations.
- **Feature flag caching**: Singleton caches that invalidate on a schedule.

### Common Mistakes

```javascript
// Mistake 1: Not cleaning up singleton resources
class LeakySingleton {
  #connections = [];
  addConnection(conn) {
    this.#connections.push(conn); // Never removed - memory leak
  }
}

// Mistake 2: Singletons in serverless environments
// Lambda functions may reuse containers; singletons persist across invocations
// Solution: Check and reset state per invocation

// Mistake 3: Assuming singleton is thread-safe (Node.js Worker Threads)
// Each thread has its own module instance, but shared memory requires locks

// Mistake 4: Storing request-specific data in singletons
// This leaks data between requests in web servers
```

### Best Practices

```javascript
// Implement IDisposable-style cleanup
class ManagedSingleton {
  static #instance;
  #resources = [];
  #disposed = false;

  constructor() {
    if (ManagedSingleton.#instance) return ManagedSingleton.#instance;
    ManagedSingleton.#instance = this;
  }

  async dispose() {
    if (this.#disposed) return;
    this.#disposed = true;
    for (const resource of this.#resources) {
      await resource.close();
    }
    this.#resources = [];
    ManagedSingleton.#instance = null;
  }

  async [Symbol.asyncDispose]() {
    await this.dispose();
  }
}

// Use WeakRef for caches that should not prevent GC
const cache = new Map();
export function getCached(key, factory) {
  if (!cache.has(key) || !cache.get(key).deref()) {
    cache.set(key, new WeakRef(factory()));
  }
  return cache.get(key).deref();
}

// Prefer factory functions with dependency injection over hardcoded singletons
export function createService(deps) {
  return { /* service implementation using deps */ };
}
// Let the composition root create and manage the singleton
```

### Performance Considerations

- Lazy initialization improves startup time but adds latency on first access.
- Instance pools reduce allocation/deallocation overhead for frequently created objects.
- WeakRef-based singletons can be garbage collected under memory pressure.
- Registry-based singletons add O(1) lookup overhead.
- TTL-based singletons add timer management overhead.

### Interview Questions

**Q: What are the trade-offs between eager and lazy singleton initialization?**
A: Eager initialization is simpler and catches initialization errors at startup, but increases memory usage and startup time. Lazy initialization defers the cost until first use and is useful for resources that may not be needed, but can cause unexpected latency on first access and may fail late.

**Q: How do you handle singleton cleanup in a long-running application?**
A: Implement a `dispose()` or `close()` method that releases resources and nullifies the instance. Use the singleton's lifecycle hooks (process.on('exit'), signal handlers) to trigger cleanup. In modern JS, implement `Symbol.asyncDispose` for `using` declarations.

**Q: How do you prevent singletons from causing issues in a serverless environment?**
A: In serverless (AWS Lambda, etc.), containers may be reused across invocations. Check and reinitialize singletons per invocation, or use the handler's scope to create fresh instances per request while caching expensive resources that are safe to reuse (like DB connections).

### Coding Challenges

```javascript
// Challenge 1: Implement a singleton with connection pooling for an API client.
// The singleton should maintain up to 5 concurrent connections,
// queue requests when all connections are busy, and timeout after 30 seconds.

// Challenge 2: Create a singleton registry that supports:
// - Named instances with factory functions
// - Lazy initialization
// - TTL-based eviction
// - Event emission on creation and eviction

// Challenge 3: Build a multi-tenant configuration singleton where each tenant
// gets its own configuration instance, and instances are evicted after 1 hour of inactivity.
```

### Related Topics

- Object lifecycle management
- Resource pooling
- WeakRef and FinalizationRegistry
- Serverless architecture patterns
- Dependency injection containers
