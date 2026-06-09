# Observer Pattern - Subject/Observer, event emitters, pub/sub implementation

## Introduction

The Observer Pattern establishes a one-to-many relationship between objects where when one object (the subject) changes state, all its dependents (observers) are notified automatically. This is one of the most widely used patterns in JavaScript, forming the foundation of event systems, reactive programming, state management libraries, and DOM event handling.

This file covers three manifestations of the observer pattern: the classical Subject/Observer relationship, the Event Emitter pattern popularized by Node.js, and the Publish/Subscribe pattern that decouples publishers from subscribers through a message broker.

## Subject/Observer Relationship

### What It Is

The Subject/Observer relationship is a behavioral design pattern where a subject maintains a list of observers and notifies them of state changes. Observers are directly attached to the subject, creating a tight but explicit dependency relationship.

### Why It Is Important

This pattern enables loose coupling between systems while maintaining communication. The subject doesn't need to know the details of its observers — only that they implement a specific notification interface. This makes it easy to add new observers without modifying the subject.

### How It Works Internally

The subject holds a collection of observer references. When its state changes, it iterates over the collection and calls a notification method (typically `update`) on each observer. Observers can be added or removed at runtime. The notification can be synchronous or asynchronous, and can include state data or just a signal.

### Syntax

```javascript
// Basic Subject
class Subject {
  constructor() {
    this.observers = [];
  }

  addObserver(observer) {
    this.observers.push(observer);
  }

  removeObserver(observer) {
    const idx = this.observers.indexOf(observer);
    if (idx > -1) this.observers.splice(idx, 1);
  }

  notify(data) {
    for (const observer of this.observers) {
      observer.update(data);
    }
  }
}

// Basic Observer interface
class Observer {
  update(data) {
    // React to notification
  }
}
```

### Beginner Examples

```javascript
// Weather station example
class WeatherStation {
  constructor() {
    this.temperature = 0;
    this.observers = [];
  }

  addObserver(observer) {
    this.observers.push(observer);
  }

  removeObserver(observer) {
    this.observers = this.observers.filter(o => o !== observer);
  }

  setTemperature(temp) {
    this.temperature = temp;
    this.notifyObservers();
  }

  notifyObservers() {
    for (const observer of this.observers) {
      observer.update(this.temperature);
    }
  }
}

class PhoneDisplay {
  update(temperature) {
    console.log(`Phone Display: Temperature is ${temperature}°C`);
  }
}

class WindowDisplay {
  update(temperature) {
    console.log(`Window Display: Temperature is ${temperature}°C`);
  }
}

const station = new WeatherStation();
const phone = new PhoneDisplay();
const window = new WindowDisplay();

station.addObserver(phone);
station.addObserver(window);
station.setTemperature(25);
// Phone Display: Temperature is 25°C
// Window Display: Temperature is 25°C
```

### Intermediate Examples

```javascript
// Stock market ticker with multiple data points
class StockTicker {
  constructor(symbol) {
    this.symbol = symbol;
    this.price = 0;
    this.previousPrice = 0;
    this.observers = new Set();
  }

  addObserver(observer) {
    this.observers.add(observer);
  }

  removeObserver(observer) {
    this.observers.delete(observer);
  }

  updatePrice(newPrice) {
    this.previousPrice = this.price;
    this.price = newPrice;
    this.notify({
      symbol: this.symbol,
      price: this.price,
      change: this.price - this.previousPrice,
      changePercent: this.previousPrice
        ? ((this.price - this.previousPrice) / this.previousPrice * 100).toFixed(2)
        : 0
    });
  }

  notify(data) {
    for (const observer of this.observers) {
      observer.update(data);
    }
  }
}

class PriceAlert {
  constructor(threshold) {
    this.threshold = threshold;
  }

  update(data) {
    if (data.price > this.threshold) {
      console.log(`ALERT: ${data.symbol} exceeded $${this.threshold}! Current: $${data.price}`);
    }
  }
}

class PortfolioDisplay {
  constructor() {
    this.stocks = {};
  }

  update(data) {
    this.stocks[data.symbol] = data;
    this.render();
  }

  render() {
    console.clear();
    for (const [symbol, data] of Object.entries(this.stocks)) {
      const sign = data.change >= 0 ? "+" : "";
      console.log(`${symbol}: $${data.price} (${sign}${data.change})`);
    }
  }
}

const apple = new StockTicker("AAPL");
const alert = new PriceAlert(200);
const portfolio = new PortfolioDisplay();

apple.addObserver(alert);
apple.addObserver(portfolio);
apple.updatePrice(198);
apple.updatePrice(205);
// ALERT: AAPL exceeded $200! Current: $205
```

### Advanced Examples

```javascript
// Reactive state management with computed properties
class ObservableState {
  constructor(initialState = {}) {
    this._state = { ...initialState };
    this._observers = new Map(); // key -> Set of observers
    this._computed = new Map();  // computedKey -> { deps, fn }
    this._computedCache = new Map();
  }

  get(key) {
    return this._state[key];
  }

  set(key, value) {
    const oldValue = this._state[key];
    if (oldValue === value) return;

    this._state[key] = value;
    this._notify(key, value, oldValue);

    // Invalidate and recompute dependent computeds
    for (const [computedKey, computed] of this._computed) {
      if (computed.deps.includes(key)) {
        const newValue = computed.fn(this._state);
        this._computedCache.set(computedKey, newValue);
        this._notify(computedKey, newValue, undefined);
      }
    }
  }

  defineComputed(key, deps, fn) {
    this._computed.set(key, { deps, fn });
    this._computedCache.set(key, fn(this._state));
  }

  getComputed(key) {
    return this._computedCache.get(key);
  }

  observe(key, observer) {
    if (!this._observers.has(key)) {
      this._observers.set(key, new Set());
    }
    this._observers.get(key).add(observer);
    return () => this._observers.get(key)?.delete(observer);
  }

  _notify(key, value, oldValue) {
    const observers = this._observers.get(key);
    if (observers) {
      for (const observer of observers) {
        observer(value, oldValue);
      }
    }
  }
}

// Usage
const state = new ObservableState({ firstName: "John", lastName: "Doe" });
state.defineComputed("fullName", ["firstName", "lastName"],
  (s) => `${s.firstName} ${s.lastName}`
);

const unsubscribe = state.observe("fullName", (name) => {
  console.log(`Name changed to: ${name}`);
});

state.set("firstName", "Jane");
// Name changed to: Jane Doe

// Async observer notification with queue
class AsyncSubject {
  constructor() {
    this.observers = new Set();
    this.queue = [];
    this.processing = false;
  }

  addObserver(observer) {
    this.observers.add(observer);
  }

  removeObserver(observer) {
    this.observers.delete(observer);
  }

  notify(data) {
    this.queue.push(data);
    if (!this.processing) {
      this.processQueue();
    }
  }

  async processQueue() {
    this.processing = true;
    while (this.queue.length > 0) {
      const data = this.queue.shift();
      const promises = [];
      for (const observer of this.observers) {
        promises.push(Promise.resolve(observer.update(data)));
      }
      await Promise.all(promises);
    }
    this.processing = false;
  }
}
```

### Real-World Use Cases

- **Model-View-Controller (MVC)**: Model notifies View of state changes.
- **React state management**: Redux, Zustand, MobX use observer patterns.
- **Angular change detection**: Zone.js patches browser APIs and notifies Angular of changes.
- **Vue reactivity**: Vue 3 uses Proxy-based observation for its reactivity system.
- **Game development**: Entities observe game state for collision, score changes, etc.

### Common Mistakes

```javascript
// Mistake 1: Memory leaks from not removing observers
class LeakyComponent {
  attach() {
    subject.addObserver(this); // Never removed
  }
}
// Solution: Always provide and call a cleanup/unsubscribe function

// Mistake 2: Modifying observer list during iteration
class BadSubject {
  notify(data) {
    for (let i = 0; i < this.observers.length; i++) {
      if (this.observers[i].shouldRemove) {
        this.observers.splice(i, 1); // Modifying during iteration!
      }
      this.observers[i].update(data);
    }
  }
}
// Solution: Use Set with forEach or iterate over a copy

// Mistake 3: Synchronous notification in async contexts
// Can cause unexpected execution order

// Mistake 4: Notifying with mutable state that observers might mutate
```

### Best Practices

```javascript
// Use Set instead of Array for observer collections (prevents duplicates)
class Subject {
  constructor() {
    this.observers = new Set();
  }
}

// Return unsubscribe function from addObserver
addObserver(observer) {
  this.observers.add(observer);
  return () => this.observers.delete(observer);
}

// Freeze or clone data passed to observers
notify(data) {
  const frozenData = Object.freeze({ ...data });
  for (const observer of this.observers) {
    observer.update(frozenData);
  }
}

// Use WeakRef if observers should be garbage collected
addObserver(observer) {
  const ref = new WeakRef(observer);
  this.observers.add(ref);
}
```

### Performance Considerations

- Iterating over observers is O(n) per notification; for very large observer lists, consider batching or priority queues.
- Using `Set` for observer storage provides O(1) addition and removal.
- Copying data before notification adds overhead but prevents mutation bugs.
- Asynchronous notification via microtasks (Promise) can batch updates but adds latency.

### Interview Questions

**Q: What is the difference between the Observer pattern and the Publish/Subscribe pattern?**
A: In the Observer pattern, observers are directly registered with the subject and the subject holds references to observers. In Pub/Sub, publishers and subscribers are decoupled via a message broker/event channel — neither knows about the other. Pub/Sub is more decoupled but Observer is simpler and has less overhead.

**Q: How do you prevent memory leaks with the Observer pattern?**
A: Always return an unsubscribe function from the subscription method. In frameworks, clean up subscriptions in lifecycle hooks (e.g., `useEffect` cleanup, `componentWillUnmount`). Consider using `WeakRef` for observers or `FinalizationRegistry` for automatic cleanup.

**Q: How does Vue.js implement its reactivity system using observers?**
A: Vue 3 uses JavaScript Proxy objects to intercept get/set operations on data objects. When a component reads data during render, it registers the component as a dependency (observer). When the data changes via set, Vue notifies all dependent components to re-render.

### Coding Challenges

```javascript
// Challenge 1: Implement an ObservableArray that notifies observers
// when items are added, removed, or the array is sorted/filtered.

// Challenge 2: Create a simple reactive programming library where
// computed values automatically update when their dependencies change.
// UseObserver pattern to track dependencies at access time.

// Challenge 3: Build a model syncing system where changes to a
// JavaScript object are automatically synced to a server and
// to other connected clients via WebSocket.
```

### Related Topics

- Event Emitter pattern
- Publish/Subscribe pattern
- Reactive programming (RxJS)
- Proxy and Reflect APIs
- WeakRef and FinalizationRegistry

## Event Emitter Pattern

### What It Is

The Event Emitter pattern is a JavaScript-specific implementation of the Observer pattern where a central object emits named events, and listeners register to receive specific events. It was popularized by Node.js's `EventEmitter` class and is used throughout the Node.js ecosystem.

### Why It Is Important

Event Emitters provide a clean, decoupled way to handle asynchronous operations and communicate between components. They form the backbone of Node.js's I/O model, where streams, servers, and request objects all emit events. The pattern is intuitive for developers and maps well to DOM event handling.

### How It Works Internally

An Event Emitter maintains a map of event names to arrays of listener functions. When `emit` is called with an event name, the emitter looks up the listeners for that event and invokes them in order, passing along any additional arguments. Listeners can be added (`on`), added for one-time use (`once`), or removed (`off`).

### Syntax

```javascript
// Node.js EventEmitter
const EventEmitter = require("events");

const emitter = new EventEmitter();

emitter.on("data", (payload) => {
  console.log("Received:", payload);
});

emitter.once("init", () => {
  console.log("This runs only once");
});

emitter.emit("data", { id: 1 });
emitter.emit("init");
emitter.emit("init"); // Ignored (once)

// Custom EventEmitter implementation
class SimpleEventEmitter {
  constructor() {
    this._events = {};
  }

  on(event, listener) {
    if (!this._events[event]) this._events[event] = [];
    this._events[event].push(listener);
    return () => this.off(event, listener);
  }

  off(event, listener) {
    if (!this._events[event]) return;
    this._events[event] = this._events[event].filter(l => l !== listener);
  }

  emit(event, ...args) {
    if (!this._events[event]) return false;
    for (const listener of [...this._events[event]]) {
      listener(...args);
    }
    return true;
  }

  once(event, listener) {
    const wrapper = (...args) => {
      this.off(event, wrapper);
      listener(...args);
    };
    this.on(event, wrapper);
  }
}
```

### Beginner Examples

```javascript
// Simple event-driven user registration
const EventEmitter = require("events");

class UserService extends EventEmitter {
  async registerUser(userData) {
    this.emit("user:beforeRegister", userData);

    const user = await saveToDatabase(userData);

    this.emit("user:registered", user);

    return user;
  }
}

const userService = new UserService();

userService.on("user:beforeRegister", (data) => {
  console.log("Validating user:", data.email);
});

userService.on("user:registered", (user) => {
  console.log("Welcome email sent to:", user.email);
  analytics.track("User Registered", { userId: user.id });
});

userService.on("user:registered", (user) => {
  console.log("Cache invalidated for user:", user.id);
});

// Order tracker
class OrderTracker extends EventEmitter {
  constructor() {
    super();
    this.orders = new Map();
  }

  createOrder(items) {
    const order = { id: Date.now(), items, status: "pending", createdAt: new Date() };
    this.orders.set(order.id, order);
    this.emit("order.created", order);
    return order;
  }

  processOrder(orderId) {
    const order = this.orders.get(orderId);
    if (!order) return this.emit("error", new Error("Order not found"));

    order.status = "processing";
    this.emit("order.processing", order);

    setTimeout(() => {
      order.status = "shipped";
      this.emit("order.shipped", order);
    }, 2000);
  }
}

const tracker = new OrderTracker();

tracker.on("order.created", (order) => {
  console.log(`Order #${order.id} created with ${order.items.length} items`);
});

tracker.on("order.shipped", (order) => {
  console.log(`Order #${order.id} has been shipped!`);
});
```

### Intermediate Examples

```javascript
// Custom event emitter with wildcards and namespaces
class AdvancedEventEmitter {
  constructor() {
    this._listeners = new Map();
  }

  on(pattern, listener) {
    const parts = pattern.split(":");
    const key = pattern;

    if (!this._listeners.has(key)) {
      this._listeners.set(key, []);
    }
    this._listeners.get(key).push(listener);

    // Add to wildcard patterns
    if (parts.length > 1) {
      const wildcard = parts.slice(0, -1).join(":") + ":*";
      if (!this._listeners.has(wildcard)) {
        this._listeners.set(wildcard, []);
      }
      this._listeners.get(wildcard).push(listener);
    }

    return () => this.off(pattern, listener);
  }

  off(pattern, listener) {
    const listeners = this._listeners.get(pattern);
    if (listeners) {
      const idx = listeners.indexOf(listener);
      if (idx > -1) listeners.splice(idx, 1);
    }
  }

  emit(event, ...args) {
    // Exact match
    const exact = this._listeners.get(event);
    if (exact) {
      for (const listener of [...exact]) {
        listener(...args);
      }
    }

    // Wildcard match (e.g., "user:*" matches "user:created", "user:updated")
    const parts = event.split(":");
    if (parts.length > 1) {
      const wildcard = parts.slice(0, -1).join(":") + ":*";
      const wildcardListeners = this._listeners.get(wildcard);
      if (wildcardListeners) {
        for (const listener of [...wildcardListeners]) {
          listener(...args);
        }
      }
    }

    // Global wildcard
    const globalListeners = this._listeners.get("*");
    if (globalListeners) {
      for (const listener of [...globalListeners]) {
        listener(event, ...args);
      }
    }
  }

  removeAllListeners(pattern) {
    if (pattern) {
      this._listeners.delete(pattern);
    } else {
      this._listeners.clear();
    }
  }

  listenerCount(pattern) {
    return (this._listeners.get(pattern) || []).length;
  }
}

// Usage
const emitter = new AdvancedEventEmitter();
emitter.on("user:*", (data) => console.log("User event:", data));
emitter.on("*", (event, data) => console.log(`Global catch: ${event}`, data));
emitter.emit("user:created", { id: 1 });
emitter.emit("order:paid", { id: 42 });

// Event emitter with async support and error handling
class AsyncEventEmitter {
  constructor() {
    this._listeners = new Map();
  }

  on(event, listener) {
    if (!this._listeners.has(event)) {
      this._listeners.set(event, []);
    }
    this._listeners.get(event).push(listener);
    return () => this.off(event, listener);
  }

  off(event, listener) {
    const listeners = this._listeners.get(event);
    if (listeners) {
      const idx = listeners.indexOf(listener);
      if (idx > -1) listeners.splice(idx, 1);
    }
  }

  async emit(event, ...args) {
    const listeners = this._listeners.get(event);
    if (!listeners || listeners.length === 0) return;

    const errors = [];
    for (const listener of listeners) {
      try {
        await Promise.resolve(listener(...args));
      } catch (err) {
        errors.push(err);
      }
    }
    if (errors.length > 0) {
      const errorEvent = this._listeners.get("error");
      if (errorEvent) {
        for (const handler of errorEvent) {
          handler(new AggregateError(errors, `Errors in ${event} handlers`));
        }
      }
    }
  }

  once(event, listener) {
    const wrapper = (...args) => {
      this.off(event, wrapper);
      return listener(...args);
    };
    this.on(event, wrapper);
  }
}
```

### Advanced Examples

```javascript
// Typed event emitter with TypeScript-like patterns in JSDoc
/** @template {Record<string, any[]>} T */
class TypedEventEmitter {
  /** @type {Map<keyof T, Set<(...args: any[]) => void>>} */
  #listeners = new Map();

  /** @param {keyof T} event */
  on(event, listener) {
    if (!this.#listeners.has(event)) {
      this.#listeners.set(event, new Set());
    }
    this.#listeners.get(event).add(listener);
    return () => this.#listeners.get(event).delete(listener);
  }

  /** @param {keyof T} event */
  emit(event, ...args) {
    const listeners = this.#listeners.get(event);
    if (listeners) {
      for (const listener of listeners) {
        listener(...args);
      }
    }
  }

  /** @param {keyof T} event */
  once(event, listener) {
    const wrapper = (...args) => {
      this.#listeners.get(event).delete(wrapper);
      listener(...args);
    };
    this.on(event, wrapper);
  }

  removeAllListeners(event) {
    if (event) {
      this.#listeners.delete(event);
    } else {
      this.#listeners.clear();
    }
  }
}

// Event emitter with middleware support
class MiddlewareEventEmitter extends EventEmitter {
  constructor() {
    super();
    this._middlewares = new Map();
  }

  use(event, middleware) {
    if (!this._middlewares.has(event)) {
      this._middlewares.set(event, []);
    }
    this._middlewares.get(event).push(middleware);
    return this;
  }

  emit(event, ...args) {
    const middlewares = this._middlewares.get(event) || [];

    if (middlewares.length === 0) {
      return super.emit(event, ...args);
    }

    // Run middleware chain
    let idx = 0;
    const next = () => {
      if (idx < middlewares.length) {
        const middleware = middlewares[idx++];
        middleware(args, next);
      } else {
        super.emit(event, ...args);
      }
    };
    next();
  }
}

// Usage: Middleware that logs and validates
const app = new MiddlewareEventEmitter();
app.use("request", (args, next) => {
  console.log(`[${new Date().toISOString()}] Request:`, args[0].method, args[0].url);
  next();
});
app.use("request", (args, next) => {
  if (!args[0].headers.authorization) {
    args[0].error = "Missing auth token";
  }
  next();
});

// Event emitter with replay (recent events replayed to new listeners)
class ReplayEventEmitter {
  constructor(maxHistory = 10) {
    this._listeners = new Map();
    this._history = [];
    this._maxHistory = maxHistory;
  }

  on(event, listener) {
    if (!this._listeners.has(event)) {
      this._listeners.set(event, new Set());
    }
    this._listeners.get(event).add(listener);

    // Replay recent events
    for (const entry of this._history) {
      if (entry.event === event || event === "*") {
        listener(...entry.args);
      }
    }

    return () => this._listeners.get(event).delete(listener);
  }

  emit(event, ...args) {
    this._history.push({ event, args, timestamp: Date.now() });
    if (this._history.length > this._maxHistory) {
      this._history.shift();
    }

    const listeners = this._listeners.get(event);
    if (listeners) {
      for (const listener of listeners) {
        listener(...args);
      }
    }
  }
}
```

### Real-World Use Cases

- **Node.js core**: `http.Server`, `stream.Readable`, `child_process` all extend EventEmitter.
- **WebSocket libraries**: Socket.IO, ws use event emitters for message handling.
- **ORM lifecycle hooks**: Sequelize, Mongoose emit events on model operations.
- **Build tools**: Webpack, Gulp emit events during build lifecycle.
- **Browser events**: DOM event system (`addEventListener`) follows the same pattern.

### Common Mistakes

```javascript
// Mistake 1: Exceeding maximum listeners (Node.js warning at 10)
emitter.setMaxListeners(50); // Increase if needed

// Mistake 2: Not handling errors
emitter.on("error", (err) => {
  console.error("Caught error:", err);
});

// Mistake 3: Memory leaks from forgetting to remove listeners
function setup() {
  const data = getData();
  emitter.on("update", () => {
    // Closure over large `data` object prevents GC
  });
  // Listener is never removed
}

// Mistake 4: Using arrow functions with `once` when removal is needed
// Arrow functions can't be referenced for removal
```

### Best Practices

```javascript
// Always handle 'error' events to prevent crashes
emitter.on("error", console.error);

// Use event naming conventions (namespace:action)
emitter.emit("user:created");
emitter.emit("order:status:changed");

// Return unsubscribe function from on()
function subscribe(emitter, event, handler) {
  emitter.on(event, handler);
  return () => emitter.off(event, handler);
}

// Limit listener count in development
if (process.env.NODE_ENV === "development") {
  emitter.setMaxListeners(100);
}

// Use once() for one-shot events
emitter.once("database:connected", startServer);
```

### Performance Considerations

- Each listener is a function call per emit; many listeners cause O(n) overhead.
- Node.js EventEmitter is highly optimized in C++ — prefer it over custom implementations.
- Wildcard matching adds overhead for each emit; avoid in hot paths.
- Asynchronous error handling in emitters can add microtask overhead.

### Interview Questions

**Q: What is the difference between `on` and `once` in EventEmitter?**
A: `on` registers a listener that fires every time the event is emitted. `once` registers a listener that fires only once, after which it is automatically removed. Internally, `once` wraps the listener in a function that calls `this.off()` before invoking the original listener.

**Q: How does Node.js EventEmitter handle errors?**
A: If an 'error' event is emitted and there are no listeners for it, Node.js throws the error, which can crash the process. It's a best practice to always register an 'error' listener on emitters. If there are listeners, the error is passed to them like any other event data.

**Q: Can EventEmitter be used for cross-component communication in a browser?**
A: Yes. You can create a global EventEmitter or use a pub/sub implementation for cross-component communication. This is common in vanilla JS applications and micro-frontends. In React, you'd typically use Context or a state management library, but EventEmitter can work well for event-driven architectures.

### Coding Challenges

```javascript
// Challenge 1: Implement a RetryEventEmitter that re-emits events
// if listeners throw errors, with configurable retry count and backoff.

// Challenge 2: Build a message bus using EventEmitter that supports
// message filtering, transformation middleware, and guaranteed delivery.

// Challenge 3: Create an EventEmitter that supports 'before' and 'after'
// hooks for each event, allowing listeners to intercept and modify event data.
```

### Related Topics

- Node.js EventEmitter API
- Streams (Stream extends EventEmitter)
- DOM CustomEvent API
- Reactive programming
- Middleware pattern

## Publish/Subscribe Implementation

### What It Is

The Publish/Subscribe (Pub/Sub) pattern extends the Observer pattern by introducing a message broker or event channel between publishers and subscribers. Publishers emit messages to the broker, which routes them to interested subscribers. Publishers and subscribers have no direct knowledge of each other.

### Why It Is Important

Pub/Sub provides maximum decoupling. Publishers don't need to know which subscribers exist, and subscribers don't need to know which publishers exist. This enables highly scalable, maintainable architectures where components can be added, removed, and modified independently.

### How It Works Internally

A Pub/Sub system maintains a message broker that keeps a registry of topics and their subscribers. When a publisher sends a message to a topic, the broker iterates over all subscribers of that topic and delivers the message. The broker can provide additional features like message filtering, delivery guarantees, and message persistence.

### Syntax

```javascript
// Minimal Pub/Sub implementation
class PubSub {
  constructor() {
    this.topics = new Map();
  }

  subscribe(topic, handler) {
    if (!this.topics.has(topic)) {
      this.topics.set(topic, new Set());
    }
    this.topics.get(topic).add(handler);
    return () => this.topics.get(topic)?.delete(handler);
  }

  publish(topic, data) {
    const handlers = this.topics.get(topic);
    if (handlers) {
      for (const handler of handlers) {
        handler(data);
      }
    }
  }

  unsubscribe(topic, handler) {
    this.topics.get(topic)?.delete(handler);
  }
}
```

### Beginner Examples

```javascript
// Simple notification system
const bus = new PubSub();

// Subscribers
const unsub1 = bus.subscribe("order:placed", (order) => {
  sendConfirmationEmail(order);
});

const unsub2 = bus.subscribe("order:placed", (order) => {
  updateInventory(order.items);
});

const unsub3 = bus.subscribe("order:cancelled", (orderId) => {
  processRefund(orderId);
});

// Publisher
function placeOrder(items) {
  const order = { id: generateId(), items, date: new Date() };
  // Business logic...
  bus.publish("order:placed", order);
  return order;
}

function cancelOrder(orderId) {
  // Business logic...
  bus.publish("order:cancelled", orderId);
}

// Cleanup
function cleanup() {
  unsub1();
  unsub2();
  unsub3();
}

// Chat room example
const chat = new PubSub();

chat.subscribe("chat:message", ({ user, message }) => {
  console.log(`${user}: ${message}`);
});

chat.subscribe("chat:join", ({ user }) => {
  console.log(`${user} joined the chat`);
});

chat.subscribe("chat:leave", ({ user }) => {
  console.log(`${user} left the chat`);
});

// Publisher
function sendMessage(user, message) {
  chat.publish("chat:message", { user, message, timestamp: Date.now() });
}

sendMessage("Alice", "Hello everyone!");
// Alice: Hello everyone!
```

### Intermediate Examples

```javascript
// Pub/Sub with message filtering
class FilteredPubSub {
  constructor() {
    this.subscriptions = new Map();
  }

  subscribe(topic, handler, filter = null) {
    if (!this.subscriptions.has(topic)) {
      this.subscriptions.set(topic, []);
    }
    const sub = { handler, filter };
    this.subscriptions.get(topic).push(sub);
    return () => {
      const subs = this.subscriptions.get(topic);
      if (subs) {
        const idx = subs.indexOf(sub);
        if (idx > -1) subs.splice(idx, 1);
      }
    };
  }

  publish(topic, data) {
    const subs = this.subscriptions.get(topic);
    if (!subs) return;

    for (const { handler, filter } of subs) {
      if (!filter || filter(data)) {
        handler(data);
      }
    }
  }
}

// Usage
const bus = new FilteredPubSub();

bus.subscribe("transaction", (tx) => {
  console.log(`High-value transaction: $${tx.amount}`);
}, (tx) => tx.amount > 10000);

bus.subscribe("transaction", (tx) => {
  console.log(`International transaction: ${tx.from} -> ${tx.to}`);
}, (tx) => tx.from.country !== tx.to.country);

// Pub/Sub with wildcard topics
class WildcardPubSub {
  constructor() {
    this.subscriptions = [];
  }

  subscribe(pattern, handler) {
    const regex = new RegExp(
      "^" + pattern.replace(/\./g, "\\.").replace(/\*/g, "[^.]+").replace(/#/g, ".+") + "$"
    );
    const sub = { pattern, regex, handler };
    this.subscriptions.push(sub);
    return () => {
      const idx = this.subscriptions.indexOf(sub);
      if (idx > -1) this.subscriptions.splice(idx, 1);
    };
  }

  publish(topic, data) {
    for (const { regex, handler } of this.subscriptions) {
      if (regex.test(topic)) {
        handler(topic, data);
      }
    }
  }
}

// Usage (MQTT-like topic matching)
const mqtt = new WildcardPubSub();
mqtt.subscribe("sensors/+/temperature", (topic, data) => {
  console.log(`Temperature reading from ${topic}: ${data.value}°C`);
});
mqtt.subscribe("sensors/#", (topic, data) => {
  console.log(`Any sensor data: ${topic} =`, data);
});

mqtt.publish("sensors/room1/temperature", { value: 22.5 });
mqtt.publish("sensors/room2/humidity", { value: 60 });
```

### Advanced Examples

```javascript
// Full-featured Pub/Sub with delivery guarantees and async support
class ReliablePubSub {
  constructor() {
    this.topics = new Map();
    this.pendingQueue = [];
    this.isProcessing = false;
    this.maxRetries = 3;
  }

  subscribe(topic, handler, options = {}) {
    const { filter, retries = 0, priority = 0 } = options;
    if (!this.topics.has(topic)) {
      this.topics.set(topic, []);
    }
    const subscription = { handler, filter, retries, priority, failedCount: 0 };
    this.topics.get(topic).push(subscription);
    // Sort by priority (higher = first)
    this.topics.get(topic).sort((a, b) => b.priority - a.priority);
    return () => {
      const subs = this.topics.get(topic);
      if (subs) {
        const idx = subs.indexOf(subscription);
        if (idx > -1) subs.splice(idx, 1);
      }
    };
  }

  publish(topic, data) {
    this.pendingQueue.push({ topic, data, retryCount: 0 });
    this.processQueue();
  }

  async processQueue() {
    if (this.isProcessing) return;
    this.isProcessing = true;

    while (this.pendingQueue.length > 0) {
      const message = this.pendingQueue.shift();
      await this.deliver(message);
    }

    this.isProcessing = false;
  }

  async deliver(message) {
    const { topic, data } = message;
    const subscriptions = this.topics.get(topic);
    if (!subscriptions) return;

    const deliveryPromises = [];

    for (const sub of subscriptions) {
      if (sub.filter && !sub.filter(data)) continue;

      const promise = (async () => {
        try {
          await Promise.resolve(sub.handler(data));
          sub.failedCount = 0;
        } catch (err) {
          sub.failedCount++;
          if (sub.retries > 0 && sub.failedCount <= sub.retries) {
            this.pendingQueue.push({
              topic,
              data,
              retryCount: message.retryCount + 1
            });
          }
          console.error(`Handler failed for ${topic}:`, err);
        }
      })();

      deliveryPromises.push(promise);
    }

    await Promise.allSettled(deliveryPromises);
  }

  // Message queuing with persistence
  async publishWithPersistence(topic, data) {
    await this.saveToDatabase({ topic, data, timestamp: Date.now() });
    this.publish(topic, data);
  }

  async replayHistorical(fromDate, toDate, topic) {
    const messages = await this.getMessagesFromDB(fromDate, toDate, topic);
    for (const msg of messages) {
      this.publish(msg.topic, msg.data);
    }
  }

  async saveToDatabase(message) { /* ... */ }
  async getMessagesFromDB(from, to, topic) { /* ... */ }
}

// Distributed Pub/Sub with WebSocket bridge
class DistributedPubSub {
  constructor(wsUrl) {
    this.local = new PubSub();
    this.topics = new Set();
    this.ws = null;
    this.wsUrl = wsUrl;
  }

  async connect() {
    this.ws = new WebSocket(this.wsUrl);
    this.ws.onmessage = (event) => {
      const { topic, data } = JSON.parse(event.data);
      this.local.publish(topic, data);
    };
    await new Promise((resolve) => this.ws.onopen = resolve);
  }

  subscribe(topic, handler) {
    const unsub = this.local.subscribe(topic, handler);
    if (!this.topics.has(topic)) {
      this.topics.add(topic);
      this.ws.send(JSON.stringify({ type: "subscribe", topic }));
    }
    return unsub;
  }

  publish(topic, data) {
    this.local.publish(topic, data);
    this.ws.send(JSON.stringify({ type: "publish", topic, data }));
  }

  disconnect() {
    if (this.ws) {
      for (const topic of this.topics) {
        this.ws.send(JSON.stringify({ type: "unsubscribe", topic }));
      }
      this.ws.close();
    }
  }
}

// Pub/Sub with saga pattern support (distributed transactions)
class SagaPubSub extends PubSub {
  constructor() {
    super();
    this.sagas = new Map();
  }

  beginSaga(sagaId, steps) {
    this.sagas.set(sagaId, {
      steps,
      currentStep: 0,
      compensations: [],
      state: {}
    });
    return sagaId;
  }

  async executeStep(sagaId, topic, compensationTopic) {
    const saga = this.sagas.get(sagaId);
    if (!saga) throw new Error(`Saga ${sagaId} not found`);

    return new Promise((resolve, reject) => {
      const unsub = this.subscribe(topic, (result) => {
        saga.currentStep++;
        saga.state = { ...saga.state, ...result };
        unsub();
        resolve(result);
      });

      const unsubComp = this.subscribe(compensationTopic, (error) => {
        this.compensate(sagaId);
        unsubComp();
        reject(error);
      });

      saga.compensations.push(() => {
        this.publish(compensationTopic, { sagaId });
      });
    });
  }

  async compensate(sagaId) {
    const saga = this.sagas.get(sagaId);
    if (!saga) return;

    for (const compensation of saga.compensations.reverse()) {
      try {
        compensation();
      } catch (err) {
        console.error(`Compensation failed for saga ${sagaId}:`, err);
      }
    }
    this.sagas.delete(sagaId);
  }
}
```

### Real-World Use Cases

- **Microservices communication**: Services publish events that other services consume asynchronously.
- **Event Sourcing**: Storing events as the source of truth, rebuilding state from event replay.
- **Real-time dashboards**: UI subscribes to data topics that backend services publish.
- **WebSocket message routing**: Routing incoming WebSocket messages to appropriate handlers.
- **Plugin systems**: Plugins subscribe to lifecycle events published by the host application.

### Common Mistakes

```javascript
// Mistake 1: Synchronous blocking in async subscribers
bus.subscribe("data", (data) => {
  heavyComputation(data); // Blocks the broker!
});

// Mistake 2: Subscribing without unsubscribe leads to memory leaks
class Component {
  mount() {
    this.unsub = bus.subscribe("update", this.handleUpdate);
  }
  // Missing unmount cleanup!
}

// Mistake 3: Overusing global pub/sub (makes debugging difficult)
// Use localized buses where possible

// Mistake 4: Publishing mutable objects that subscribers modify
const data = { count: 0 };
bus.publish("update", data); // Subscribers can mutate data
```

### Best Practices

```javascript
// Always return unsubscribe functions from subscribe
function createPubSub() {
  const subs = new Map();
  return {
    subscribe(topic, handler) {
      // ...
      return () => { /* cleanup */ };
    }
  };
}

// Freeze published data to prevent mutation
publish(topic, data) {
  for (const handler of this.getHandlers(topic)) {
    handler(Object.freeze({ ...data }));
  }
}

// Use typed topics (prefix namespaces)
const Topics = {
  USER: { CREATED: "user:created", UPDATED: "user:updated" },
  ORDER: { PLACED: "order:placed", CANCELLED: "order:cancelled" }
};

// Provide a way to unsubscribe all listeners for a given context
function createScopedSubscriber(context) {
  const unsubs = [];
  return {
    subscribe(topic, handler) {
      unsubs.push(bus.subscribe(topic, handler));
    },
    unsubscribeAll() {
      unsubs.forEach(fn => fn());
      unsubs.length = 0;
    }
  };
}
```

### Performance Considerations

- Each publish iterates over all subscribers of the topic — O(n) per publish.
- Wildcard/pattern matching adds regex overhead; avoid in hot paths.
- Async delivery with `await` can slow the broker; consider batching or microtask scheduling.
- Message serialization/deserialization (for distributed Pub/Sub) adds significant overhead.
- Using `Object.freeze` for every message adds cost; use for sensitive data only.

### Comparison: Observer vs Pub/Sub

| Aspect | Observer | Pub/Sub |
|--------|----------|---------|
| Coupling | Subject knows observers | Fully decoupled via broker |
| Scale | Single subject, many observers | Many publishers, many subscribers |
| Complexity | Simple | More complex (broker overhead) |
| Use case | Single state source | Cross-system communication |
| Implementation | Observers register with subject | Subscribers register with broker |
| Message routing | Direct | Topic-based |
| State management | Subject holds state | Messages are ephemeral |

### Interview Questions

**Q: When would you choose Pub/Sub over the Observer pattern?**
A: Choose Pub/Sub when you need maximum decoupling, when multiple publishers can emit the same type of event, or when building cross-system communication (microservices, distributed systems). Choose Observer when you have a single subject with known observers and want simplicity and performance.

**Q: How do you handle message ordering guarantees in Pub/Sub?**
A: In single-threaded JS, messages are processed in FIFO order. For distributed systems, use message sequencing (sequence numbers), Kafka-style partitions per key, or idempotent consumers that can handle out-of-order messages.

**Q: What is the "callback hell" problem with Pub/Sub and how do you mitigate it?**
A: Complex Pub/Sub chains can become hard to follow and debug. Mitigations include: using message schemas, logging all publications, using a centralized debugger/visualizer, preferring async/await patterns, and keeping subscription handlers simple.

### Coding Challenges

```javascript
// Challenge 1: Implement a Pub/Sub system with guaranteed delivery.
// If a handler throws, retry the delivery up to 3 times with exponential backoff.
// If all retries fail, publish to a "dead letter" topic.

// Challenge 2: Build a time-travel debugger for Pub/Sub that records
// all messages and allows replaying them from any point in time.

// Challenge 3: Create a priority-based Pub/Sub where subscribers
// can specify priority levels, and higher priority subscribers
// receive messages before lower priority ones.
```

### Related Topics

- Message queues (RabbitMQ, Kafka)
- Event-driven architecture
- CQRS (Command Query Responsibility Segregation)
- Event Sourcing
- ReactiveX (RxJS)
