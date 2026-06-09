# Real-World Scenarios - System design, debugging, performance, architecture decisions

## Introduction

Technical interviews increasingly include real-world scenario questions that assess a candidate's ability to apply knowledge to practical problems. These questions cover system design, debugging strategies, performance optimization, and architectural decision-making. Unlike theoretical knowledge questions, scenarios test a developer's experience, judgment, and ability to make tradeoffs. This file covers common scenario-based questions with structured approaches to solving them.

## System Design Discussions

### What It Is

System design questions ask candidates to architect a scalable, reliable, and maintainable system. For JavaScript roles, this typically involves Node.js backend services, frontend applications, or full-stack systems. Candidates are expected to discuss requirements, estimate scale, propose architecture, identify bottlenecks, and justify tradeoffs. Common topics include designing a URL shortener, real-time chat system, rate limiter, or notification service.

### Why It Is Important

System design reveals a candidate's ability to think holistically about software beyond syntax and APIs. It tests understanding of scalability, caching, database design, API design, error handling, monitoring, and deployment. For senior roles, system design is often the most weighted interview section.

### How It Works Internally

A typical system design interview follows this framework:

1. **Clarify requirements**: Functional (what the system does) and non-functional (scale, latency, availability, consistency)
2. **Estimate scale**: Daily active users, requests per second, data storage needs, bandwidth
3. **Design data model**: Schema design, database choice (SQL vs NoSQL)
4. **Design API**: Endpoints, request/response format, authentication
5. **High-level architecture**: Components, services, data flow
6. **Deep dive**: Focus on interesting aspects (caching strategy, real-time updates, sharding)
7. **Address bottlenecks**: Single points of failure, latency optimization, data consistency

### Syntax

```javascript
// Example: URL shortener API design
// POST /api/shorten
// Request: { url: "https://example.com/very/long/url" }
// Response: { shortUrl: "https://short.ly/abc123", expiresAt: "..." }

// GET /:shortCode
// Redirect to original URL with 301 or 302

// API Router
const express = require('express');
const router = express.Router();

router.post('/shorten', validateUrl, rateLimit, async (req, res) => {
  const { url, customAlias, ttl } = req.body;
  
  // Generate short code
  const shortCode = customAlias || nanoid(7);
  
  // Store mapping
  await urlStore.create({
    shortCode,
    originalUrl: url,
    expiresAt: ttl ? Date.now() + ttl : null,
    createdAt: new Date(),
    clicks: 0
  });
  
  // Cache for fast lookups
  await cache.set(shortCode, url, { EX: 3600 });
  
  res.status(201).json({
    shortUrl: `${BASE_URL}/${shortCode}`,
    shortCode
  });
});

router.get('/:shortCode', async (req, res) => {
  const { shortCode } = req.params;
  
  // Check cache first
  let url = await cache.get(shortCode);
  
  if (!url) {
    const record = await urlStore.findByShortCode(shortCode);
    if (!record) return res.status(404).json({ error: 'Not found' });
    
    if (record.expiresAt && Date.now() > record.expiresAt) {
      return res.status(410).json({ error: 'Expired' });
    }
    
    url = record.originalUrl;
    await cache.set(shortCode, url, { EX: 3600 });
  }
  
  // Track click asynchronously
  incrementClick(shortCode).catch(console.error);
  
  res.redirect(301, url);
});
```

### Beginner Examples

```javascript
// Simple rate limiter design (in-memory)
class RateLimiter {
  constructor(windowMs = 60000, maxRequests = 100) {
    this.windowMs = windowMs;
    this.maxRequests = maxRequests;
    this.clients = new Map();
  }
  
  isAllowed(clientId) {
    const now = Date.now();
    if (!this.clients.has(clientId)) {
      this.clients.set(clientId, { count: 1, startTime: now });
      return true;
    }
    
    const record = this.clients.get(clientId);
    if (now - record.startTime > this.windowMs) {
      record.count = 1;
      record.startTime = now;
      return true;
    }
    
    record.count++;
    return record.count <= this.maxRequests;
  }
}

// Design choices:
// - In-memory: fast but not shared across processes
// - Redis-based: shared, persistent, distributed
// - Token bucket vs sliding window vs fixed window
```

### Intermediate Examples

```javascript
// Scalable real-time notification system design
class NotificationSystem {
  constructor() {
    this.channels = new Map(); // userId -> Set<SSE connection>
    this.queue = [];           // Pending notifications
    this.batchInterval = 100;  // Batch every 100ms
    this.setupBatchProcessor();
  }
  
  // Push-based delivery via Server-Sent Events
  subscribe(userId, res) {
    if (!this.channels.has(userId)) {
      this.channels.set(userId, new Set());
    }
    this.channels.get(userId).add(res);
    
    res.on('close', () => {
      this.channels.get(userId)?.delete(res);
    });
    
    // Set SSE headers
    res.writeHead(200, {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive'
    });
  }
  
  // Send notification to user
  async notify(userId, notification) {
    // Store for persistence
    await this.storeNotification(userId, notification);
    
    // Push to connected clients
    const connections = this.channels.get(userId);
    if (connections) {
      connections.forEach(res => {
        res.write(`data: ${JSON.stringify(notification)}\n\n`);
      });
    }
    
    // Push notification (FCM/APNs) for offline users
    if (!connections?.size) {
      await this.sendPushNotification(userId, notification);
    }
  }
  
  setupBatchProcessor() {
    setInterval(() => {
      this.processBatch();
    }, this.batchInterval);
  }
  
  processBatch() {
    // Batch writes to database for efficiency
    // Process notifications in bulk
  }
  
  async storeNotification(userId, notification) {
    // Store in database
    // Index by userId, sort by timestamp
  }
  
  async sendPushNotification(userId, notification) {
    // Via Firebase Cloud Messaging or APNs
  }
}
```

### Advanced Examples

```javascript
// Event sourcing for an order management system
class EventStore {
  constructor(db) {
    this.db = db;
    this.events = [];
    this.subscribers = new Map();
  }
  
  async append(streamId, eventType, data, metadata = {}) {
    const event = {
      id: nanoid(),
      streamId,
      type: eventType,
      data,
      metadata: {
        timestamp: Date.now(),
        version: await this.getNextVersion(streamId),
        ...metadata
      }
    };
    
    await this.db.collection('events').insertOne(event);
    this.events.push(event);
    await this.notifySubscribers(event);
    
    return event;
  }
  
  async getStream(streamId, fromVersion = 0) {
    return this.db.collection('events')
      .find({ streamId, 'metadata.version': { $gt: fromVersion } })
      .sort({ 'metadata.version': 1 })
      .toArray();
  }
  
  async getNextVersion(streamId) {
    const last = await this.db.collection('events')
      .findOne({ streamId }, { sort: { 'metadata.version': -1 } });
    return (last?.metadata?.version || 0) + 1;
  }
  
  // Rebuild current state from events
  async rebuildState(streamId) {
    const events = await this.getStream(streamId);
    let state = initialState;
    
    for (const event of events) {
      const handler = this.eventHandlers[event.type];
      if (handler) {
        state = handler(state, event.data);
      }
    }
    
    return state;
  }
  
  subscribe(eventType, handler) {
    if (!this.subscribers.has(eventType)) {
      this.subscribers.set(eventType, []);
    }
    this.subscribers.get(eventType).push(handler);
  }
  
  async notifySubscribers(event) {
    const handlers = this.subscribers.get(event.type) || [];
    await Promise.all(handlers.map(h => h(event)));
  }
}

// CQRS read model projection
class OrderProjection {
  constructor(eventStore) {
    this.eventStore = eventStore;
    this.eventStore.subscribe('OrderPlaced', this.handleOrderPlaced.bind(this));
    this.eventStore.subscribe('OrderShipped', this.handleOrderShipped.bind(this));
    this.eventStore.subscribe('OrderCancelled', this.handleOrderCancelled.bind(this));
  }
  
  async handleOrderPlaced(event) {
    await db.collection('order_read_model').insertOne({
      orderId: event.streamId,
      userId: event.data.userId,
      items: event.data.items,
      total: event.data.total,
      status: 'placed',
      createdAt: event.metadata.timestamp
    });
  }
  
  async handleOrderShipped(event) {
    await db.collection('order_read_model').updateOne(
      { orderId: event.streamId },
      { $set: { status: 'shipped', shippedAt: event.metadata.timestamp } }
    );
  }
}
```

### Real-World Use Cases

- Design YouTube/Netflix: video streaming, transcoding, CDN, recommendation
- Design Twitter: feed generation, fanout, timeline caching
- Design Uber: ride matching, real-time tracking, surge pricing
- Design e-commerce: product catalog, cart, checkout, payment
- Design collaborative editor: OT/CRDT, WebSocket, diff synchronization

### Common Mistakes

- Jumping to solutions without clarifying requirements
- Ignoring scale estimates and assuming all systems need massive scale
- Not discussing tradeoffs (consistency vs availability, read vs write optimization)
- Forgetting about failure modes, monitoring, and deployment
- Over-engineering (Kubernetes for 100 users)
- Not discussing data consistency and conflict resolution

### Best Practices

- Start with requirements and constraints
- Estimate scale before designing
- Discuss tradeoffs explicitly
- Consider failure modes and recovery
- Use standard patterns (CQRS, event sourcing, pub/sub) when appropriate
- Draw diagrams to communicate architecture
- Focus on the interesting/challenging aspects of the problem

### Performance Considerations

- Caching strategy (CDN, application cache, database cache)
- Database indexing and query optimization
- Connection pooling and resource management
- Asynchronous processing and message queues
- Horizontal vs vertical scaling decisions
- Read replicas for read-heavy workloads

### Interview Questions

**Q: Design a URL shortener. What are the key considerations?**
A: Key areas: hash/ID generation (base62, nanoid, snowflake), storage (key-value store for fast lookups), caching (Redis for hot URLs), redirect strategy (301 for permanent, 302 for analytics), TTL for expired URLs, rate limiting for creation, analytics tracking (clicks, referrers), and scaling (read replicas, CDN for redirects).

**Q: How would you design a real-time chat application?**
A: Key areas: WebSocket for persistent connections, message ordering (server-assigned sequence numbers), presence detection (heartbeats), message persistence (database), delivery guarantees (at-least-once with idempotency), horizontal scaling (Redis pub/sub across nodes), typing indicators (throttled), media sharing (CDN, pre-signed URLs), and offline message sync.

### Coding Challenges

**Challenge 1: Design and implement a distributed rate limiter**
```javascript
// Implement a sliding window rate limiter that works across multiple
// server instances. Use Redis for coordination.

class DistributedRateLimiter {
  constructor(redis, options = {}) {
    this.redis = redis;
    this.windowMs = options.windowMs || 60000;
    this.maxRequests = options.maxRequests || 100;
  }

  async isAllowed(key) {
    const now = Date.now();
    const windowStart = now - this.windowMs;
    const sortedSetKey = `ratelimit:${key}`;

    // Use Redis sorted set with timestamp as score
    // Multi/exec for atomicity
    const lua = `
      redis.call('ZREMRANGEBYSCORE', KEYS[1], 0, ARGV[1])
      local count = redis.call('ZCARD', KEYS[1])
      if count < tonumber(ARGV[2]) then
        redis.call('ZADD', KEYS[1], ARGV[3], ARGV[4])
        redis.call('EXPIRE', KEYS[1], ARGV[5])
        return {1, count + 1, ARGV[3]}
      end
      return {0, count, redis.call('ZRANGE', KEYS[1], 0, 0, 'WITHSCORES')[2]}
    `;

    const result = await this.redis.eval(lua, 1, sortedSetKey,
      windowStart,
      this.maxRequests,
      now,
      `${now}:${Math.random()}`,
      Math.ceil(this.windowMs / 1000) + 1
    );

    return {
      allowed: result[0] === 1,
      current: result[1],
      resetAt: parseInt(result[2]) + this.windowMs
    };
  }

  async getRemaining(key) {
    const count = await this.redis.zcount(
      `ratelimit:${key}`,
      Date.now() - this.windowMs,
      Date.now()
    );
    return Math.max(0, this.maxRequests - count);
  }
}
```

**Challenge 2: Design a real-time collaborative document editor**
```javascript
// Implement the core CRDT (Conflict-free Replicated Data Type)
// for real-time collaborative editing.

class CharCRDT {
  constructor(clientId) {
    this.clientId = clientId;
    this.clock = 0;
    this.chars = [];
  }

  insert(afterIndex, char) {
    this.clock++;
    const position = this.generatePosition(afterIndex);
    const op = {
      type: 'insert',
      char,
      position,
      clientId: this.clientId,
      clock: this.clock
    };
    this.applyInsert(op);
    return op;
  }

  delete(index) {
    if (index < 0 || index >= this.chars.length) return null;
    const op = {
      type: 'delete',
      id: this.chars[index].id,
      clientId: this.clientId,
      clock: ++this.clock
    };
    this.chars[index].deleted = true;
    return op;
  }

  generatePosition(afterIndex) {
    if (afterIndex < 0) {
      return { before: null, after: this.chars[0]?.id || null };
    }
    if (afterIndex >= this.chars.length - 1) {
      return { before: this.chars[this.chars.length - 1]?.id || null, after: null };
    }
    return {
      before: this.chars[afterIndex].id,
      after: this.chars[afterIndex + 1].id
    };
  }

  applyInsert(op) {
    const char = {
      id: `${op.clientId}:${op.clock}`,
      char: op.char,
      deleted: false
    };

    if (!op.position.after) {
      this.chars.push(char);
    } else {
      const idx = this.chars.findIndex(c => c.id === op.position.after);
      if (idx >= 0) {
        this.chars.splice(idx, 0, char);
      }
    }
  }

  applyOp(op) {
    if (op.type === 'insert') this.applyInsert(op);
    else if (op.type === 'delete') {
      const char = this.chars.find(c => c.id === op.id);
      if (char) char.deleted = true;
    }
  }

  getText() {
    return this.chars.filter(c => !c.deleted).map(c => c.char).join('');
  }
}
```

**Challenge 3: Design a real-time analytics dashboard backend**
```javascript
// Build an event processing pipeline that aggregates analytics
// events in real-time and serves dashboard data.

class AnalyticsPipeline {
  constructor(redis, db) {
    this.redis = redis;
    this.db = db;
    this.buffer = [];
    this.flushInterval = setInterval(() => this.flush(), 5000);
  }

  async track(event) {
    event.timestamp = event.timestamp || Date.now();
    this.buffer.push(event);

    // Real-time counters (Redis)
    const key = `analytics:${event.type}:${this.getMinute(event.timestamp)}`;
    await this.redis.hincrby(key, 'count', 1);
    await this.redis.expire(key, 86400);

    // Unique users
    const userKey = `analytics:unique:${event.type}:${this.getDay(event.timestamp)}`;
    await this.redis.pfadd(userKey, event.userId);
    await this.redis.expire(userKey, 86400 * 7);
  }

  async flush() {
    if (this.buffer.length === 0) return;
    const batch = this.buffer.splice(0);
    try {
      await this.db.collection('analytics_events').insertMany(batch);
    } catch (err) {
      console.error('Failed to flush analytics:', err);
      this.buffer.unshift(...batch);
    }
  }

  async getStats(type, since) {
    const pipeline = this.redis.pipeline();
    const now = Date.now();
    let ts = since;
    while (ts < now) {
      pipeline.hgetall(`analytics:${type}:${this.getMinute(ts)}`);
      ts += 60000;
    }
    const results = await pipeline.exec();
    return results
      .filter(r => r[1])
      .map(([, data]) => ({ count: parseInt(data.count || 0) }))
      .reduce((sum, r) => sum + r.count, 0);
  }

  getMinute(ts) { return Math.floor(ts / 60000); }
  getDay(ts) { return Math.floor(ts / 86400000); }

  destroy() { clearInterval(this.flushInterval); this.flush(); }
}
```

### Related Topics

- System design patterns (CQRS, Event Sourcing, Saga)
- CAP theorem and tradeoffs
- Caching strategies (CDN, Redis, in-memory)
- Database design (SQL vs NoSQL, sharding, replication)
- Real-time communication (WebSocket, SSE, long polling)
- Microservices and service mesh
- Container orchestration (Docker, Kubernetes)
- API design (REST, GraphQL, gRPC)
- Monitoring and observability

## Debugging Scenarios

### What It Is

Debugging scenario questions present a broken application and ask the candidate to diagnose and fix the issue. Common scenarios include memory leaks, race conditions, production performance degradation, incorrect state updates, and unexpected behavior.

### Why It Is Important

Debugging is a core software engineering skill. These questions assess systematic problem-solving, tool proficiency, and understanding of how systems fail.

### How It Works Internally

A structured debugging approach:

1. **Reproduce the issue**: Understand the exact conditions
2. **Isolate the cause**: Binary search, divide and conquer
3. **Form a hypothesis**: Based on observations
4. **Test the hypothesis**: Add logging, inspect state
5. **Fix and verify**: Apply fix, confirm resolution
6. **Prevent recurrence**: Add tests, monitoring, guardrails

### Syntax

```javascript
// Common debugging patterns

// Memory leak: retained references
class LeakyComponent {
  constructor() {
    this.cache = new Map();
  }
  
  addToCache(key, value) {
    this.cache.set(key, value); // Never cleared!
  }
}

// Race condition: shared state
let counter = 0;
async function increment() {
  const current = counter; // Read
  await someAsyncOperation();
  counter = current + 1;  // Write (based on stale read)
}

// Debugging with Chrome DevTools
// console.table, console.time, console.trace
// Performance tab, Memory tab (heap snapshots)
// Network tab (waterfall)
```

### Beginner Examples

```javascript
// Scenario: Function returns undefined unexpectedly
function getConfig(key) {
  const config = { theme: 'dark', lang: 'en' };
  return config.key; // Bug: should be config[key]
}

// Debugging approach:
// 1. Add logging: console.log(config, key)
// 2. Check property access pattern
// 3. Fix: return config[key]

// Scenario: Event listener firing multiple times
function setupButton() {
  const btn = document.getElementById('button');
  btn.addEventListener('click', handleClick); // Called multiple times!
}

// Fix: Remove old listeners first
function setupButton() {
  const btn = document.getElementById('button');
  btn.removeEventListener('click', handleClick);
  btn.addEventListener('click', handleClick);
}

// Or use { once: true } if single click is expected
```

### Intermediate Examples

```javascript
// Scenario: Memory leak from closures
function createWidget(container) {
  const data = fetchLargeDataset(); // 10MB
  
  container.addEventListener('click', function onClick() {
    // This closure keeps 'data' alive
    // If container is removed from DOM, 'data' persists
    // because onClick is still referenced
    processData(data);
  });
  
  // Fix: Add cleanup mechanism
  return {
    destroy() {
      container.removeEventListener('click', onClick);
    }
  };
}

// Scenario: Async race condition in search
const searchResults = [];
input.addEventListener('input', async () => {
  const results = await fetch(`/search?q=${input.value}`);
  searchResults.push(results);
  displayResults(searchResults[searchResults.length - 1]);
  // Bug: if request 1 completes after request 2,
  // results[1] displays request 1's data (stale)
});

// Fix: Use cancellation token
let abortController;
input.addEventListener('input', () => {
  abortController?.abort();
  abortController = new AbortController();
  const query = input.value;
  
  fetch(`/search?q=${query}`, { signal: abortController.signal })
    .then(res => res.json())
    .then(results => {
      // Only display if query still matches
      if (input.value === query) {
        displayResults(results);
      }
    })
    .catch(err => {
      if (err.name !== 'AbortError') handleError(err);
    });
});
```

### Advanced Examples

```javascript
// Scenario: Production performance degradation (memory leak)
// Symptoms: Heap grows over time, GC pauses increase
// Debugging approach:

// 1. Take heap snapshot (Chrome DevTools / Node.js --inspect)
// 2. Compare snapshots to identify growing objects
// 3. Look for detached DOM elements, closures, caches

// Example leak: growing cache
class DataService {
  constructor() {
    this.cache = new Map();
    this.maxSize = 1000;
  }
  
  async getData(key) {
    if (this.cache.has(key)) {
      return this.cache.get(key);
    }
    const data = await fetch(`/api/${key}`);
    this.cache.set(key, data);
    
    // BUG: No eviction policy
    // Fix: Add LRU eviction
    if (this.cache.size > this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey); // Simple FIFO eviction
      // Better: implement actual LRU
    }
    
    return data;
  }
}

// Scenario: Debugging a deadlock in async code
class ResourceManager {
  constructor() {
    this.lock1 = new Mutex();
    this.lock2 = new Mutex();
  }
  
  async methodA() {
    await this.lock1.acquire();
    await delay(100);
    await this.lock2.acquire(); // Deadlock if methodB runs simultaneously
    // ...
    this.lock2.release();
    this.lock1.release();
  }
  
  async methodB() {
    await this.lock2.acquire();
    await delay(100);
    await this.lock1.acquire(); // Deadlock!
    // ...
    this.lock1.release();
    this.lock2.release();
  }
}

// Fix: Always acquire locks in the same order
// Use a lock hierarchy

// Scenario: Debugging infinite re-render in React
function MyComponent() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    setCount(count + 1); // Triggers re-render → effect runs again → infinite loop
  }); // Missing dependency array!
  
  return <div>{count}</div>;
}
```

### Real-World Use Cases

- Memory leak from detached DOM event listeners
- Race condition in database update operations
- Stale closure in React hooks (useEffect, useCallback)
- CORS issues in API integration
- Throttling/debouncing issues in search autocomplete
- Authentication token expiration race conditions
- WebSocket reconnection storms

### Common Mistakes

- Not reproducing the issue consistently before debugging
- Making changes without forming a hypothesis
- Fixing symptoms instead of root causes
- Not adding regression tests after fixing
- Ignoring error logs and monitoring data
- Overlooking the simplest explanations

### Best Practices

- Use systematic approach: reproduce → isolate → hypothesize → test → fix
- Leverage devtools: breakpoints, heap snapshots, performance recording
- Add structured logging (correlation IDs, timing)
- Use source maps for debugging minified code
- Write automated tests that reproduce the bug
- Monitor production with APM tools (Datadog, New Relic, Sentry)
- Use feature flags for safe rollouts and rollbacks

### Performance Considerations

- Heap snapshots are expensive (pause the runtime)
- Performance recordings can affect timing-dependent bugs
- Logging can mask timing bugs (Heisenbug)
- Production debugging should use minimal instrumentation
- Use CPU and heap profiling with caution in production

### Interview Questions

**Q: A Node.js application has a memory leak that grows over 24 hours. How do you debug it?** A: 1) Use `--inspect` flag and take heap snapshots at intervals. 2) Compare snapshots to identify growing objects. 3) Look for global caches without eviction, closures retaining large data, event listeners on detached DOM, or unclosed connections. 4) Add `heapdump` module for automated snapshots. 5) Use Chrome DevTools Memory tab to analyze retainers.

**Q: How do you debug a race condition that only happens in production?** A: 1) Add structured logging with correlation IDs and timestamps. 2) Implement distributed tracing. 3) Add metrics for concurrent operations. 4) Use feature flags to narrow down conditions. 5) Consider using formal verification for critical async workflows. 6) Reproduce in staging with production-like load. 7) Add runtime assertions to detect unexpected state.

### Coding Challenges

**Challenge 1: Debug a production memory leak**
```javascript
// Given the following code, identify why memory grows over time and fix it.

class EventBus {
  constructor() {
    this.listeners = new Map();
    this.eventHistory = [];
  }

  on(event, callback) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event).add(callback);
    // Bug: listeners are never removed when components unmount
  }

  emit(event, data) {
    this.eventHistory.push({ event, data, timestamp: Date.now() });
    // Bug: eventHistory grows unbounded
    const callbacks = this.listeners.get(event);
    if (callbacks) {
      callbacks.forEach(cb => cb(data));
    }
  }

  off(event, callback) {
    const callbacks = this.listeners.get(event);
    if (callbacks) {
      callbacks.delete(callback);
      if (callbacks.size === 0) this.listeners.delete(event);
    }
  }
}

// Debugging steps:
// 1. Take heap snapshot, observe Map and Array growth
// 2. Check retainers of eventHistory
// 3. Verify components call off() on unmount
// Fix: Add maxHistory limit and ensure cleanup

class FixedEventBus extends EventBus {
  constructor(maxHistory = 1000) {
    super();
    this.maxHistory = maxHistory;
  }

  emit(event, data) {
    this.eventHistory.push({ event, data, timestamp: Date.now() });
    if (this.eventHistory.length > this.maxHistory) {
      this.eventHistory.splice(0, this.eventHistory.length - this.maxHistory);
    }
    super.emit(event, data);
  }

  // Add weak reference option
  onWeak(event, callback) {
    const weakCallback = (data) => {
      if (typeof callback === 'function') callback(data);
    };
    this.on(event, weakCallback);
  }
}
```

**Challenge 2: Debug a race condition in async request handling**
```javascript
// The following search component has a race condition.
// Identify and fix it.

class SearchComponent {
  constructor(input, resultsContainer) {
    this.input = input;
    this.results = resultsContainer;
    this.pendingRequest = 0;
    input.addEventListener('input', () => this.handleInput());
  }

  async handleInput() {
    const query = this.input.value;
    this.pendingRequest++;
    const requestId = this.pendingRequest;

    try {
      const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`);
      const data = await response.json();

      // Bug: if a newer request already completed, this overwrites correct results
      // Bug: if requests complete out of order, stale data shows
      if (requestId === this.pendingRequest) {
        this.renderResults(data);
      }
    } catch (err) {
      // Bug: error from stale request can show error state even if newer request succeeded
      if (requestId === this.pendingRequest) {
        this.renderError(err);
      }
    }
  }

  renderResults(data) { this.results.innerHTML = data.map(d => `<div>${d.title}</div>`).join(''); }
  renderError(err) { this.results.innerHTML = `<div class="error">${err.message}</div>`; }
}

// Fixed version:
class FixedSearchComponent {
  constructor(input, resultsContainer) {
    this.input = input;
    this.results = resultsContainer;
    this.abortController = null;
    this.input.addEventListener('input', () => this.handleInput());
  }

  async handleInput() {
    this.abortController?.abort();
    this.abortController = new AbortController();
    const query = this.input.value;
    const controller = this.abortController;

    if (query.length < 2) {
      this.results.innerHTML = '';
      return;
    }

    this.results.innerHTML = '<div class="loading">Searching...</div>';

    try {
      const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`, {
        signal: controller.signal
      });
      const data = await response.json();

      // Only update if this request is still the latest
      if (controller === this.abortController) {
        this.renderResults(data);
      }
    } catch (err) {
      if (err.name !== 'AbortError' && controller === this.abortController) {
        this.renderError(err);
      }
    }
  }

  renderResults(data) { this.results.innerHTML = data.map(d => `<div>${d.title}</div>`).join(''); }
  renderError(err) { this.results.innerHTML = `<div class="error">${err.message}</div>`; }
}
```

**Challenge 3: Debug an infinite re-render in a React application**
```jsx
// Find and fix the infinite re-render bugs in this React component.

import React, { useState, useEffect, useCallback, useMemo } from 'react';

function UserDashboard({ userId }) {
  const [user, setUser] = useState(null);
  const [settings, setSettings] = useState(null);
  const [count, setCount] = useState(0);

  // Bug 1: Missing dependency - runs on every render
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }); // Missing [userId]

  // Bug 2: Object recreated every render, causing infinite effect loop
  const defaultSettings = { theme: 'light', notifications: true };

  useEffect(() => {
    setSettings(defaultSettings);
  }, [defaultSettings]); // defaultSettings is new every render!

  // Bug 3: useCallback with stale closure
  const handleClick = useCallback(() => {
    setCount(count + 1); // count is stale
  }, []); // Missing count dependency

  // Bug 4: Expensive computation on every render
  const processedData = processData(user);
  // Should use useMemo

  return (
    <div onClick={handleClick}>
      <UserProfile user={user} settings={settings} />
      <DataView data={processedData} />
    </div>
  );
}

// Fixed version:
function FixedUserDashboard({ userId }) {
  const [user, setUser] = useState(null);
  const [settings, setSettings] = useState(null);
  const [count, setCount] = useState(0);

  // Fix 1: Add dependency
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);

  // Fix 2: Memoize default settings
  const defaultSettings = useMemo(() => ({
    theme: 'light',
    notifications: true
  }), []);

  useEffect(() => {
    setSettings(defaultSettings);
  }, [defaultSettings]);

  // Fix 3: Use functional update
  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []);

  // Fix 4: Memoize computation
  const processedData = useMemo(() => processData(user), [user]);

  return (
    <div onClick={handleClick}>
      <UserProfile user={user} settings={settings} />
      <DataView data={processedData} />
    </div>
  );
}
```

### Related Topics

- Debugging tools (Chrome DevTools, Node.js inspector)
- Performance profiling and optimization
- APM and observability (Datadog, OpenTelemetry)
- State machines and workflow engines
- Feature flags and A/B testing
- Incident response and post-mortems

## Related Topics

- System design patterns (CQRS, Event Sourcing, Saga)
- CAP theorem and tradeoffs
- Debugging tools (Chrome DevTools, Node.js inspector)
- Performance profiling and optimization
- APM and observability (Datadog, OpenTelemetry)
- Distributed systems (consensus, replication, partitioning)
- State machines and workflow engines
- Feature flags and A/B testing
- Incident response and post-mortems
