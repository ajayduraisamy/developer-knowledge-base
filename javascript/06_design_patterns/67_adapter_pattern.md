# Adapter Pattern - Interface adaptation, wrapper functions, third-party integration

## Introduction

The Adapter Pattern allows objects with incompatible interfaces to collaborate by wrapping one interface with another that the client expects. It acts as a bridge between two incompatible systems, translating calls from the client format to the format expected by the service.

In JavaScript, the Adapter Pattern is particularly useful for integrating third-party libraries, working with legacy code, and normalizing data from different sources. JavaScript's dynamic typing makes adapters simpler to implement than in statically-typed languages, as adapters can be plain functions, Proxy objects, or duck-typed classes without requiring explicit interface declarations.

## Interface Adaptation

### What It Is

Interface adaptation is the process of converting one interface to another so that classes or objects with incompatible APIs can work together. The adapter wraps the adaptee (the object being adapted) and provides the target interface that the client expects. The client interacts only with the adapter and never directly with the adaptee.

### Why It Is Important

In real-world development, you rarely have complete control over all the interfaces in your system. Third-party libraries have their own APIs. Legacy systems use outdated patterns. Different versions of the same library may have different APIs. The Adapter Pattern lets you make these systems work together without modifying source code. It follows the Open/Closed Principle: you can extend the behavior of existing code without modifying it. Instead of changing the third-party library, you write an adapter that translates between interfaces.

### How It Works Internally

The adapter holds a reference to the adaptee. When the client calls a method on the adapter, the adapter translates that call into one or more calls on the adaptee's interface. The translation can involve renaming methods, changing parameter order or format, combining multiple calls into one, converting data formats, or adding default values for missing parameters. In JavaScript, this is typically done through composition (the adapter contains the adaptee), though it can also be implemented via Proxy traps for dynamic adaptation.

### Syntax

```javascript
class Target {
  request() { return 'Target'; }
}
class Adaptee {
  specificRequest() { return '.eetpadA'; }
}
class Adapter extends Target {
  constructor(adaptee) { super(); this.adaptee = adaptee; }
  request() { return this.adaptee.specificRequest().split('').reverse().join(''); }
}
const adapter = new Adapter(new Adaptee());
console.log(adapter.request());
```

### Beginner Examples

```javascript
class LegacyWeatherAPI {
  getTemperature(c) { return { celsius: 22 }; }
  getHumidity(c) { return { percentage: 65 }; }
}
class WeatherAdapter {
  constructor(api) { this.api = api; }
  fetchWeather(city) {
    const code = this.cityToCode(city);
    return { temp: this.api.getTemperature(code).celsius,
             humidity: this.api.getHumidity(code).percentage };
  }
  cityToCode(n) { return { 'nyc':'NYC','london':'LDN' }[n]||'NYC'; }
}

// Payment gateway adapter
class StripePayment { charge(amount, curr, src) { return { id: 'stripe_'+Date.now(), status: 'success' }; } }
class PayPalPayment { makePayment(data) { return { txId: 'pp_'+Date.now(), state: 'completed' }; } }
class PaymentAdapter {
  constructor(gw) { this.gateway = gw; }
  pay(amount, currency) {
    if (this.gateway instanceof StripePayment) return this.gateway.charge(amount, currency, 'card');
    if (this.gateway instanceof PayPalPayment) return this.gateway.makePayment({ amount, currency });
  }
}
```

### Intermediate Examples

```javascript
// Database adapter pattern
class MySQLAdapter {
  async connect(config) { this.conn = { host: config.host }; }
  async query(sql, params) { return [{ id: 1, name: 'test' }]; }
  async disconnect() { this.conn = null; }
}
class MongoDBAdapter {
  async connect(config) { this.db = { name: config.database }; }
  async query(collection, filter) { return [{ id: 1, name: 'test' }]; }
  async disconnect() { this.db = null; }
}
class DBFactory {
  static create(type) {
    if (type === 'mysql') return new MySQLAdapter();
    if (type === 'mongodb') return new MongoDBAdapter();
    throw new Error('Unsupported type');
  }
}

// Callback to Promise adapter
class CallbackAPI {
  getUser(id, cb) { setTimeout(() => cb(null, { id, name: 'Alice' }), 100); }
}
class PromiseAdapter {
  constructor(api) { this.api = api; }
  getUser(id) { return new Promise((res, rej) => this.api.getUser(id, (e, u) => e ? rej(e) : res(u))); }
}
```

### Advanced Examples

```javascript
// Proxy-based dynamic adapter
class DynamicAdapter {
  constructor(adaptee, methodMap) {
    return new Proxy(adaptee, {
      get(target, prop) {
        if (methodMap[prop]) {
          const mapped = methodMap[prop];
          return (...args) => {
            const adapted = mapped(args, target);
            return target[adapted.method](...adapted.args);
          };
        }
        if (typeof target[prop] === 'function') return target[prop].bind(target);
        return target[prop];
      }
    });
  }
}
const legacy = { processData: (type, data) => 'Processing ' + type + ': ' + JSON.stringify(data) };
const adapted = new DynamicAdapter(legacy, {
  run: (args, target) => ({ method: 'processData', args: ['default', args[0]] })
});
console.log(adapted.run({ hello: 'world' }));

// ORM adapter pattern
class ORMAdapter {
  constructor(db) { this.db = db; this.models = new Map(); }
  define(name, schema) {
    const model = {
      name, schema, records: [],
      async create(data) { const r = { id: this.records.length+1, ...data, createdAt: new Date() }; this.records.push(r); return r; },
      async findById(id) { return this.records.find(r => r.id === id) || null; },
      async find(filter) { return this.records.filter(r => Object.entries(filter||{}).every(([k,v]) => r[k]===v)); }
    };
    this.models.set(name, model);
    return model;
  }
}
```

### Real-World Use Cases

- Database abstraction layers (Knex, Sequelize) abstract different SQL dialects
- Payment gateway integration (Stripe, PayPal, Square) unified via adapter
- Cloud storage providers (S3, GCS, Azure Blob) unified interface
- Authentication providers (Passport.js strategies) adapt different OAuth providers
- Caching backends (Memory, Redis, Memcached) unified via adapter
- Testing and mocking with adapter test doubles

### Common Mistakes

```javascript
// Mistake: Leaking adaptee-specific details to client
class LeakyAdapter { getData() { return this.adaptee.getIncompatibleFormat(); } }
// Mistake: Adapter that changes behavior contract
class BadAdapter { getData() { try { return this.api.fetch(); } catch { return null; } } }
// Mistake: Not handling adaptee errors properly
class ErrorAdapter { async getData() { return this.api.fetch(); } }
```

### Best Practices

```javascript
// Keep adapters focused on interface translation only
class UserAdapter {
  constructor(legacy) { this.legacy = legacy; }
  async getUser(id) {
    const u = await this.legacy.fetchUserById(id);
    return { id: u.uid, name: u.full_name, email: u.email_address };
  }
}
// Test adapter independently with mock adaptee
// Use dependency injection for the adaptee
```

### Performance Considerations

- Each adapter call adds a thin wrapping layer, negligible for most use cases
- Proxy-based adapters have slightly higher overhead per property access
- For performance-critical paths, pre-bind adapted methods or use direct delegation
- Streaming adapters may need buffering or batching for high-frequency data

### Interview Questions

**Q: How is Adapter different from Facade?**
A: Adapter converts one interface to another for compatibility between two systems. Facade provides a simplified interface to a complex subsystem. Adapter typically wraps one class; Facade wraps an entire subsystem.

**Q: When would you use object adapter vs class adapter?**
A: JavaScript uses object adapters (composition) since it lacks multiple inheritance. The adapter holds a reference to the adaptee. Class adapters via inheritance are rare in JS but possible if extending both target and adaptee.

**Q: How would you adapt a REST API to match a GraphQL-like interface?**
A: Create an adapter that accepts GraphQL-style queries like `getUser(id, fields)` and translates them to REST calls like `/api/users/:id`, filtering the response to only include requested fields.

### Coding Challenges

```javascript
// Challenge 1: Create an adapter that converts fs.readFile/writeFile
// to work with S3.getObject/S3.putObject maintaining the same interface.
// Challenge 2: Build an adapter wrapping synchronous localStorage
// to match an async IndexedDB API, returning Promises for all operations.
// Challenge 3: Normalize errors from axios, fetch, and node-fetch
// into a unified format with status, message, and code properties.
```

### Related Topics

- Facade pattern, Bridge pattern, Decorator pattern, Proxy pattern, Middleware pattern

## Wrapper Functions

### What It Is

Wrapper functions encapsulate another function, adding behavior before, after, or around the original call. In the Adapter Pattern, wrappers adapt a function's parameters, return type, or error handling to match what the caller expects. They are the simplest form of adaptation requiring no classes or objects.

### Why It Is Important

Wrapper functions are the most lightweight form of the Adapter Pattern. They require no classes, no objects — just a function that calls another function with transformed arguments. This makes them ideal for quick interface adaptation, adding logging, error handling, caching, or retry logic around third-party functions.

### How It Works Internally

A wrapper function accepts the same parameters as the target interface, transforms them as needed, calls the adapted function, transforms the return value if needed, and returns the result. The wrapper can also add error handling, retries, logging, or timing around the call. Wrappers can be composed by chaining multiple wrappers together.

### Syntax

```javascript
function wrap(fn) {
  return function(...args) {
    return fn(...args);
  };
}
const wrapArrow = (fn) => (...args) => fn(...args);
```

### Beginner Examples

```javascript
// Parameter transformation
const createUser = (firstName, lastName) => ({ id: Date.now(), name: firstName + ' ' + lastName });
const createUserFromFullName = (fullName) => {
  const parts = fullName.split(' ');
  return createUser(parts[0], parts.slice(1).join(' ') || '');
};
console.log(createUserFromFullName('Alice Johnson'));

// Adding default parameters
const sendEmail = (to, subject, body) => console.log('Sending to ' + to);
const sendWelcomeEmail = (userEmail) => sendEmail(userEmail, 'Welcome!', 'Thank you for joining!');

// Changing return format
const fetchRawData = async (url) => { const r = await fetch(url); return r.text(); };
const fetchJSON = async (url) => {
  const raw = await fetchRawData(url);
  return { data: JSON.parse(raw), timestamp: Date.now(), source: url };
};
```

### Intermediate Examples

```javascript
// With retry wrapper
const withRetry = (fn, opts = {}) => {
  const max = opts.maxRetries || 3;
  const backoff = opts.backoff || ((n) => Math.pow(2, n) * 100);
  return async (...a) => {
    let lastErr;
    for (let i = 0; i <= max; i++) {
      try { return await fn(...a); }
      catch(e) { lastErr = e; if (i < max) await new Promise(r => setTimeout(r, backoff(i))); }
    }
    throw lastErr;
  };
};

// With caching wrapper
const withCache = (fn, opts = {}) => {
  const c = new Map(); const ttl = opts.ttl || 60000;
  return async (...a) => {
    const k = JSON.stringify(a); const h = c.get(k);
    if (h && Date.now()-h.t < ttl) return h.d;
    const r = await fn(...a); c.set(k, {d:r, t:Date.now()}); return r;
  };
};

// With logging and timing
const withLogging = (fn, name) => {
  return async (...a) => {
    console.log('Calling ' + name);
    const start = performance.now();
    try { const r = await fn(...a); console.log(name + ' took ' + (performance.now()-start).toFixed(2) + 'ms'); return r; }
    catch(e) { console.error(name + ' failed:', e.message); throw e; }
  };
};
```

### Advanced Examples

```javascript
// Composable wrapper factory
function createWrapper(...middlewares) {
  return (originalFn) => middlewares.reduceRight((fn, mw) => mw(fn), originalFn);
}
const withValidation = (validator) => (fn) => async (...args) => {
  const valid = validator(...args);
  if (valid !== true) throw new Error(valid || 'Validation failed');
  return fn(...args);
};
const processOrder = async (data) => ({ success: true, orderId: Date.now() });
const validatedProcess = createWrapper(withLogging, withRetry, withValidation(
  (data) => { if (!data.items?.length) return 'Must have items'; return true; }
))(processOrder);

// Partial application adapter
const partial = (fn, ...preset) => (...later) => fn(...preset, ...later);
```

### Real-World Use Cases

- Express middleware wraps route handlers with auth, logging, parsing
- Redux applyMiddleware wraps dispatch with logging, thunks, sagas
- Axios interceptors wrap HTTP calls for auth tokens, error handling
- Rate limiting wraps API calls to enforce request limits

### Common Mistakes

```javascript
// Mistake: Losing this context - always use fn.apply(this, args)
// Mistake: Not handling async functions - check for Promise
// Mistake: Forgetting to return the result from wrapper
// Mistake: Over-wrapping with too many layers making debugging difficult
```

### Performance Considerations

- Each wrapper adds nanosecond-level function call overhead in V8
- Multiple wrappers add stack depth; minimize nesting in hot paths
- Async wrappers add microtask overhead; sync wrappers are free
- Object spread in wrappers creates shallow copies

## Third-Party Library Integration

### What It Is

Third-party library integration using the Adapter Pattern involves creating an abstraction layer between your application code and external libraries. The adapter encapsulates the library-specific code, providing a clean interface that your application uses. This insulates your code from library changes and makes it easy to swap libraries.

### Why It Is Important

Direct dependency on third-party libraries creates tight coupling. If the library's API changes (major version upgrade), the library is deprecated, or you need to switch to a competitor, the changes can ripple through your entire codebase. An adapter layer isolates these concerns, making library changes localized to the adapter.

### How It Works Internally

The adapter imports and uses the third-party library internally. It exposes only the methods your application needs, translated to your domain language. Your application code never directly imports the third-party library — it always goes through the adapter. This creates a seam for testing, mocking, and library replacement.

### Beginner Examples

```javascript
class HttpClientAdapter {
  constructor(url) { this.baseURL = url; this.ints = []; }
  use(fn) { this.ints.push(fn); }
  async get(e, p) { return this.req('GET', e, null, p); }
  async post(e, d) { return this.req('POST', e, d); }
  async req(m, e, d, p) {
    const url = new URL(this.baseURL + e);
    if (p) Object.entries(p).forEach(([k,v]) => url.searchParams.set(k,v));
    const opts = { method: m, headers: {'Content-Type':'application/json'} };
    if (d) opts.body = JSON.stringify(d);
    let cfg = { url: url.toString(), options: opts };
    this.ints.forEach(fn => { cfg = fn(cfg) || cfg; });
    const res = await fetch(cfg.url, cfg.options);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    return res.json();
  }
}
```

### Best Practices

```javascript
// Always expose cleanup/dispose methods
// Keep adapter interface minimal - only what your app needs
// Document which library is being adapted
// Write unit tests for the adapter with mock library
// Consider performance implications of the translation layer
```

### Related Topics
- Facade, Bridge, Decorator, Proxy patterns
- Higher-order functions, Function composition
