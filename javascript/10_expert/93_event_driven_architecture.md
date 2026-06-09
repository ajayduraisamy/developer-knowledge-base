# Event-Driven Architecture - Event emitters, event loops, pub/sub, event sourcing

## Introduction

Event-driven architecture (EDA) is a software design pattern where components communicate through events rather than direct method calls. In JavaScript, the event-driven model is deeply embedded—from DOM events in the browser to EventEmitters in Node.js to pub/sub systems in distributed applications. Understanding EDA is essential for building decoupled, scalable, and reactive systems.

## Event Emitter Pattern

### What It Is

The Event Emitter pattern allows objects to emit named events that other objects can listen to. It implements the Observer pattern: a subject (the emitter) maintains a list of dependents (observers) and notifies them of state changes. Node.js's `EventEmitter` class is a canonical implementation, and the pattern is widely used in browsers (DOM events) and frontend frameworks.

### Why It Is Important

Event emitters enable loose coupling between components. The emitter doesn't need to know who or how many listeners exist. This makes code more modular, testable, and scalable. Event emitters are the foundation of Node.js's stream system, HTTP module, and many userland libraries.

### How It Works Internally

An EventEmitter maintains a map of event names to arrays of listener functions. When `emit(eventName, ...args)` is called, it looks up the listener array and calls each listener with the provided arguments. Key internal operations: adding listeners (`on`, `addListener`, `once`), removing listeners (`off`, `removeListener`, `removeAllListeners`), and emitting events.

```javascript
// Simplified EventEmitter implementation
class EventEmitter {
  constructor() {
    this._events = new Map();
  }
  
  on(name, listener) {
    if (!this._events.has(name)) this._events.set(name, []);
    this._events.get(name).push(listener);
    return this;
  }
  
  emit(name, ...args) {
    const listeners = this._events.get(name);
    if (!listeners) return false;
    listeners.forEach(listener => listener(...args));
    return true;
  }
  
  removeListener(name, listener) {
    const listeners = this._events.get(name);
    if (!listeners) return;
    const idx = listeners.indexOf(listener);
    if (idx >= 0) listeners.splice(idx, 1);
  }
}
```

### Syntax

```javascript
const EventEmitter = require('events');

// Creating an emitter
const emitter = new EventEmitter();

// Listening to events
emitter.on('data', (chunk) => {
  console.log('Received:', chunk);
});

emitter.once('close', () => {
  console.log('Closed (will only fire once)');
});

// Emitting events
emitter.emit('data', 'Hello');
emitter.emit('close');
emitter.emit('close'); // Nothing happens (once listener removed)

// Remove listeners
const handler = () => console.log('handler');
emitter.on('event', handler);
emitter.off('event', handler); // Remove specific listener
emitter.removeAllListeners('event'); // Remove all listeners for event

// Error handling
emitter.on('error', (err) => console.error('Error:', err.message));
emitter.emit('error', new Error('Something went wrong'));
```

### Beginner Examples

```javascript
// Custom event emitter
const EventEmitter = require('events');

class Logger extends EventEmitter {
  log(message) {
    const timestamp = new Date().toISOString();
    this.emit('message', { timestamp, message });
  }
}

const logger = new Logger();
logger.on('message', (data) => {
  console.log(`[${data.timestamp}] ${data.message}`);
});

logger.log('Application started');
logger.log('User logged in');

// Once listener
logger.once('initialized', () => {
  console.log('This runs only once');
});
logger.emit('initialized');
logger.emit('initialized'); // Ignored

// Counting listeners
const EventEmitter = require('events');
console.log(logger.listenerCount('message')); // 1
console.log(logger.eventNames()); // ['message', 'initialized']
```

### Intermediate Examples

```javascript
// EventEmitter with async listeners
const { EventEmitter } = require('events');

class AsyncEmitter extends EventEmitter {
  async emitAsync(event, ...args) {
    const listeners = this.listeners(event);
    const results = [];
    for (const listener of listeners) {
      results.push(await Promise.resolve(listener(...args)));
    }
    return results;
  }
}

const emitter = new AsyncEmitter();
emitter.on('data', async (value) => {
  await new Promise(r => setTimeout(r, 100));
  console.log('Processed:', value);
  return value * 2;
});

emitter.on('data', async (value) => {
  console.log('Logged:', value);
  return value;
});

// Emit and wait for all listeners
emitter.emitAsync('data', 42).then(results => {
  console.log('All results:', results);
});

// Error handling in EventEmitter
emitter.on('error', (err) => {
  console.error('Global error handler:', err.message);
});

// Memory leak warning
// By default, EventEmitter warns if >10 listeners for same event
emitter.setMaxListeners(25); // Increase limit

// Event emitter inheritance
class Database extends EventEmitter {
  async connect() {
    this.emit('connecting');
    try {
      await connectToDB();
      this.emit('connected');
    } catch (err) {
      this.emit('error', err);
      this.emit('connectionFailed');
    }
  }
  
  async query(sql) {
    this.emit('query', sql);
    // ... process
    this.emit('result', data);
  }
}

const db = new Database();
db.on('connected', () => console.log('DB connected'));
db.on('query', (sql) => console.log('Query:', sql));
db.on('error', (err) => console.error('DB error:', err));
```

### Advanced Examples

```javascript
// Event delegation with async iterators
const { on, once } = require('events');
const EventEmitter = require('events');

const emitter = new EventEmitter();

// Event as async iterator
async function readEvents() {
  for await (const [data] of on(emitter, 'data')) {
    console.log('Event stream:', data);
    if (data === 'done') break;
  }
}

readEvents();
emitter.emit('data', 'chunk1');
emitter.emit('data', 'chunk2');
emitter.emit('data', 'done');

// Promise-based event waiting
async function waitForEvent(emitter, event, timeout = 5000) {
  return Promise.race([
    once(emitter, event),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error(`Timeout waiting for ${event}`)), timeout)
    )
  ]);
}

// Usage
await waitForEvent(db, 'connected', 30000);

// Event deduplication
class DedupEmitter extends EventEmitter {
  constructor() {
    super();
    this._lastEmit = new Map();
  }
  
  dedupEmit(event, data, interval = 100) {
    const key = `${event}:${JSON.stringify(data)}`;
    const last = this._lastEmit.get(event);
    if (last && Date.now() - last < interval) return;
    this._lastEmit.set(event, Date.now());
    this.emit(event, data);
  }
}

// Custom propagation control (stop propagation)
class PropagatingEmitter extends EventEmitter {
  emit(event, ...args) {
    const listeners = this.rawListeners(event);
    for (const listener of listeners) {
      const result = listener(...args);
      if (result === false) break; // Stop propagation
    }
    return true;
  }
}
```

### Real-World Use Cases

- Node.js HTTP server (request, response events)
- Stream processing (data, end, error events)
- WebSocket and real-time communication
- Plugin systems and middleware hooks
- Game engine event systems
- UI component communication (without coupling)

### Common Mistakes

- Not handling `error` events (unhandled errors crash process)
- Memory leaks from forgetting to remove listeners
- Adding too many listeners (warns at >10)
- Using `EventEmitter` for simple callbacks (over-engineering)
- Mutating listener array during emission (unexpected behavior)
- Not using `once` for one-time events

### Best Practices

- Always attach an `error` listener to every emitter
- Remove listeners when no longer needed (`emitter.off()`)
- Use `once()` for one-time events
- Set `maxListeners` appropriately for your use case
- Use descriptive event names (namespaced, like `user:login`)
- Consider `Symbol` event names for internal events
- Extend EventEmitter class for complex emitters
- Use `eventNames()` and `listenerCount()` for debugging

## Event Loop Integration

### What It Is

Event emitter integration with the event loop involves understanding how emitted events interact with synchronous code, microtasks, and macrotasks. Event emitters can be used to schedule work on different phases of the event loop.

### Why It Is Important

Understanding event loop integration helps prevent unexpected execution order and ensures that events are processed at the right time. It also enables patterns like deferred emission, async event processing, and scheduling.

### How It Works Internally

When `emit()` is called, listeners are invoked synchronously in the current execution context. If you want asynchronous execution, you must schedule it explicitly (using `setImmediate`, `process.nextTick`, `Promise.resolve().then()`, etc.).

```javascript
const EventEmitter = require('events');
const emitter = new EventEmitter();

emitter.on('event', () => console.log('A: sync listener'));
emitter.on('event', () => {
  setImmediate(() => console.log('C: scheduled'));
  console.log('B: sync listener');
});

console.log('1');
emitter.emit('event');
console.log('2');
// Output: 1, A, B, 2, C
```

### Syntax

```javascript
// Deferred event emission
class DeferredEmitter extends EventEmitter {
  emitAsync(event, ...args) {
    return new Promise(resolve => {
      setImmediate(() => {
        this.emit(event, ...args);
        resolve();
      });
    });
  }
}

// Batch emission (coalesce events)
class BatchEmitter extends EventEmitter {
  constructor(delay = 10) {
    super();
    this._pending = new Map();
    this._timer = null;
    this._delay = delay;
  }
  
  emitBatched(event, data) {
    if (!this._pending.has(event)) {
      this._pending.set(event, []);
    }
    this._pending.get(event).push(data);
    
    if (!this._timer) {
      this._timer = setTimeout(() => this._flush(), this._delay);
    }
  }
  
  _flush() {
    this._timer = null;
    for (const [event, datas] of this._pending) {
      this.emit(event, datas.length === 1 ? datas[0] : datas);
    }
    this._pending.clear();
  }
}
```

### Best Practices

- Emit synchronously for same-tick notifications
- Use `setImmediate` to break long listener chains
- Batch rapid events to avoid event loop pressure
- Use `async_hooks` for tracking async context across listeners

## Pub/Sub Systems

### What It Is

Publish/Subscribe (pub/sub) is a messaging pattern where publishers emit events/messages without knowledge of subscribers, and subscribers express interest in specific events without knowledge of publishers. A message broker or event bus sits between them. This is an evolution of the EventEmitter pattern for distributed systems.

### Why It Is Important

Pub/sub enables loose coupling at scale. In distributed systems, services can communicate through events without direct dependencies. This is essential for microservices, event-driven architectures, and real-time applications.

### How It Works Internally

In a pub/sub system:
- Publishers send messages to topics/channels
- Subscribers register interest in topics
- A broker routes messages from publishers to subscribers
- Messages may be persistent (stored for later delivery) or transient

```javascript
// In-memory pub/sub broker
class PubSub {
  constructor() {
    this.topics = new Map();
  }
  
  subscribe(topic, handler) {
    if (!this.topics.has(topic)) this.topics.set(topic, new Set());
    this.topics.get(topic).add(handler);
    return () => this.topics.get(topic)?.delete(handler);
  }
  
  publish(topic, message) {
    const handlers = this.topics.get(topic);
    if (!handlers) return;
    handlers.forEach(handler => {
      try { handler(message); } catch (e) { console.error(e); }
    });
  }
}
```

### Syntax

```javascript
// Redis pub/sub example
const Redis = require('ioredis');
const publisher = new Redis();
const subscriber = new Redis();

// Subscribe
subscriber.subscribe('order:created', (err, count) => {
  console.log(`Subscribed to ${count} channels`);
});

subscriber.on('message', (channel, message) => {
  const order = JSON.parse(message);
  console.log(`New order: ${order.id}`);
});

// Publish
const order = { id: 123, items: ['item1'], total: 50 };
publisher.publish('order:created', JSON.stringify(order));

// Unsubscribe
subscriber.unsubscribe('order:created');

// RabbitMQ with amqplib
const amqp = require('amqplib');

async function setup() {
  const conn = await amqp.connect('amqp://localhost');
  const channel = await conn.createChannel();
  
  const exchange = 'orders';
  await channel.assertExchange(exchange, 'topic', { durable: true });
  
  // Publish
  channel.publish(exchange, 'order.created', Buffer.from(JSON.stringify(order)));
  
  // Subscribe
  const q = await channel.assertQueue('', { exclusive: true });
  await channel.bindQueue(q.queue, exchange, 'order.*');
  channel.consume(q.queue, (msg) => {
    console.log('Received:', msg.content.toString());
    channel.ack(msg);
  });
}
```

### Intermediate Examples

```javascript
// Typed event bus with TypeScript
interface EventMap {
  'user:created': { id: string; name: string };
  'user:deleted': { id: string };
  'order:placed': { orderId: string; amount: number };
}

type EventHandler<K extends keyof EventMap> = (data: EventMap[K]) => void;

class EventBus {
  private handlers = new Map<string, Set<Function>>();
  
  on<K extends keyof EventMap>(event: K, handler: EventHandler<K>): () => void {
    if (!this.handlers.has(event)) this.handlers.set(event, new Set());
    this.handlers.get(event)!.add(handler);
    return () => this.handlers.get(event)?.delete(handler);
  }
  
  emit<K extends keyof EventMap>(event: K, data: EventMap[K]) {
    this.handlers.get(event)?.forEach(handler => {
      try { handler(data); } catch (e) { console.error(e); }
    });
  }
  
  removeAll(event?: keyof EventMap) {
    if (event) this.handlers.delete(event);
    else this.handlers.clear();
  }
}

// Wildcard subscriptions
class WildcardEventBus {
  private handlers = new Map<string, Set<Function>>();
  
  on(pattern: string, handler: Function) {
    if (!this.handlers.has(pattern)) this.handlers.set(pattern, new Set());
    this.handlers.get(pattern)!.add(handler);
    return () => this.handlers.get(pattern)?.delete(handler);
  }
  
  emit(event: string, data: any) {
    for (const [pattern, handlers] of this.handlers) {
      if (this.match(pattern, event)) {
        handlers.forEach(h => { try { h(data, event); } catch(e) {} });
      }
    }
  }
  
  private match(pattern: string, event: string): boolean {
    const regex = new RegExp('^' + pattern.replace(/\*/g, '.*') + '$');
    return regex.test(event);
  }
}
```

## Event Sourcing Concepts

### What It Is

Event sourcing is a pattern where state changes are stored as a sequence of events, rather than the current state. Instead of storing the current order status, you store every event that happened to the order (OrderPlaced, PaymentReceived, ItemShipped, etc.). The current state is derived by replaying the event sequence.

### Why It Is Important

Event sourcing provides a complete audit trail, enables time travel (reconstructing past states), facilitates event-driven integrations (other services can subscribe to events), and naturally supports CQRS (Command Query Responsibility Segregation).

### How It Works Internally

Events are stored in an append-only event store. Each event has a type, timestamp, and data payload. To get current state, events are replayed in order through reducer functions (like Redux). Events are immutable—they are never modified or deleted, only appended.

### Syntax

```javascript
// Event sourcing for an order system
class EventStore {
  constructor() {
    this.events = [];
  }
  
  append(aggregateId, type, data) {
    const event = {
      id: this.events.length + 1,
      aggregateId,
      type,
      data,
      timestamp: new Date().toISOString(),
      version: this.getNextVersion(aggregateId)
    };
    this.events.push(event);
    return event;
  }
  
  getEvents(aggregateId) {
    return this.events.filter(e => e.aggregateId === aggregateId);
  }
  
  getNextVersion(aggregateId) {
    const events = this.getEvents(aggregateId);
    return events.length + 1;
  }
}

// Event handlers (reducers)
const reducers = {
  OrderPlaced: (state, event) => ({
    ...state,
    status: 'placed',
    items: event.data.items,
    total: event.data.total
  }),
  PaymentReceived: (state, event) => ({
    ...state,
    status: 'paid',
    paymentId: event.data.paymentId
  }),
  OrderShipped: (state, event) => ({
    ...state,
    status: 'shipped',
    trackingNumber: event.data.trackingNumber
  }),
  OrderDelivered: (state, event) => ({
    ...state,
    status: 'delivered',
    deliveredAt: event.timestamp
  })
};

// Rebuild state from events
function rebuildState(events) {
  return events.reduce((state, event) => {
    const reducer = reducers[event.type];
    return reducer ? reducer(state, event) : state;
  }, { id: events[0]?.aggregateId, status: 'pending', createdAt: events[0]?.timestamp });
}

// Usage
const store = new EventStore();
store.append('order-1', 'OrderPlaced', { items: ['widget'], total: 100 });
store.append('order-1', 'PaymentReceived', { paymentId: 'pay-123' });
store.append('order-1', 'OrderShipped', { trackingNumber: '1Z999AA' });

const currentState = rebuildState(store.getEvents('order-1'));
console.log(currentState);
// { id: 'order-1', status: 'shipped', items: ['widget'], total: 100, ... }
```

### Real-World Use Cases

- Auditing and compliance systems (financial transactions)
- E-commerce order processing
- Collaborative applications (Google Docs)
- Banking and accounting systems
- Version control systems (Git is event-sourced)
- Temporal debugging and replay

### Common Mistakes

- Using event sourcing when simple CRUD suffices
- Storing the entire current state alongside events (defeats purpose)
- Mutating events after storage (break replay)
- Not versioning event schemas (evolving data structures)
- Ignoring event store performance (queries require replaying all events)
- Complex projection management

### Best Practices

- Keep events small and focused (single responsibility per event)
- Version your event schemas
- Use event versioning for backward compatibility
- Implement snapshots to avoid full replay for large streams
- Store events in append-only data stores
- Use projections (read models) for query performance
- Ensure idempotency in event handlers
- Implement event upcasting for schema evolution


### Performance Considerations
- Event emitters with many listeners cause linear dispatch time; limit listeners with setMaxListeners()
- Memory leaks occur when listeners are not removed (use once() for one-shot events)
- Event sourcing requires efficient storage (event stores) and replay mechanisms
- Pub/sub systems can bottleneck on message serialization/deserialization
- Asynchronous event handling prevents blocking the event loop


### Interview Questions
1. Explain the difference between Event Emitter and Pub/Sub patterns.
2. How would you implement an event-driven microservice communication?
3. What is event sourcing and when would you use it?
4. How do you prevent memory leaks with event emitters?
5. Compare event-driven architecture with request-response architecture.


### Coding Challenges
```javascript
// Challenge 1: Implement a simple EventEmitter
class EventEmitter {
  constructor() { this.events = {}; }
  on(event, listener) {
    if (!this.events[event]) this.events[event] = [];
    this.events[event].push(listener);
    return () => this.off(event, listener);
  }
  emit(event, ...args) {
    if (!this.events[event]) return;
    this.events[event].forEach(l => l(...args));
  }
  off(event, listener) {
    if (!this.events[event]) return;
    this.events[event] = this.events[event].filter(l => l !== listener);
  }
}

const emitter = new EventEmitter();
const unsub = emitter.on('data', d => console.log('Received:', d));
emitter.emit('data', { id: 1 });

// Challenge 2: Implement a simple pub/sub
const pubsub = { subscribers: {} };
pubsub.subscribe = (event, fn) => {
  if (!pubsub.subscribers[event]) pubsub.subscribers[event] = [];
  pubsub.subscribers[event].push(fn);
};
pubsub.publish = (event, data) => {
  (pubsub.subscribers[event] || []).forEach(fn => fn(data));
};
pubsub.subscribe('user:created', user => console.log('New user:', user.name));
pubsub.publish('user:created', { name: 'Alice' });
```

### Related Topics

- Observable pattern and reactive programming
- CQRS (Command Query Responsibility Segregation)
- Stream processing and event streaming
- Apache Kafka and message brokers
- Redux architecture (event sourcing for UI state)
- Domain-Driven Design (DDD) aggregates
- Eventual consistency
- Saga pattern for distributed transactions
- Change Data Capture (CDC)
