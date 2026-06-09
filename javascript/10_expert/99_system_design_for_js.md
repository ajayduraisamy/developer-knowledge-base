# System Design for JS - Architecture patterns, scalability, microservices, state management

## Introduction

System design for JavaScript applications encompasses architectural decisions that affect scalability, maintainability, and reliability. As JavaScript has become a full-stack technology (Node.js, React Native, Electron), understanding how to design systems that scale—from front-end state management to back-end microservice architecture—is essential for senior engineers. This file covers architectural patterns, scalability strategies, microservices with Node.js, and state management approaches.

## Architecture Patterns (MVC, Flux, Redux)

### What It Is

Architecture patterns provide proven structures for organizing code. MVC (Model-View-Controller) separates data, UI, and logic. Flux (and Redux) enforce unidirectional data flow. Each pattern addresses the challenges of growing codebases: coupling, testability, and state management complexity.

### Why It Is Important

Without a clear architecture, applications become "big balls of mud" where changes have unpredictable effects. Architecture patterns provide rules for how code modules interact, making the system predictable, testable, and maintainable as it scales.

### How It Works Internally

**MVC (server-side, traditional)**:
- **Model**: Manages data and business logic
- **View**: Renders UI (presentation)
- **Controller**: Handles input, updates model, selects view

**Flux (client-side, Facebook)**:
- **Dispatcher**: Central hub for all events
- **Stores**: Hold application state
- **Views** (React components): Read state from stores
- **Actions**: Payloads dispatched to stores

**Redux (simplified Flux)**:
- **Store**: Single source of truth (one store)
- **Actions**: Plain objects describing state changes
- **Reducers**: Pure functions that compute new state

### Syntax

```javascript
// MVC Server-side (Node.js/Express)
// Model
class User {
  constructor(data) { Object.assign(this, data); }
  static async find(id) { return db.users.findById(id); }
  async save() { return db.users.update(this); }
}

// Controller
const userController = {
  async show(req, res) {
    const user = await User.find(req.params.id);
    res.render('user/profile', { user });
  },
  async update(req, res) {
    const user = await User.find(req.params.id);
    Object.assign(user, req.body);
    await user.save();
    res.redirect(`/users/${user.id}`);
  }
};

// Routes
app.get('/users/:id', userController.show);
app.post('/users/:id', userController.update);

// Flux
class TodoStore {
  constructor() {
    this.todos = [];
    dispatcher.register((action) => {
      switch (action.type) {
        case 'ADD_TODO':
          this.todos.push(action.payload);
          this.emit('change');
          break;
      }
    });
  }
  getTodos() { return [...this.todos]; }
}

// Redux
const initialState = { todos: [], filter: 'ALL' };
function todoReducer(state = initialState, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return { ...state, todos: [...state.todos, action.payload] };
    case 'TOGGLE_TODO':
      return { ...state, todos: state.todos.map((t, i) =>
        i === action.index ? { ...t, completed: !t.completed } : t
      )};
    case 'SET_FILTER':
      return { ...state, filter: action.filter };
    default:
      return state;
  }
}
```

### Beginner Examples

```javascript
// Simple Redux example
const { createStore } = require('redux');

// Reducer (pure function)
function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT': return state + 1;
    case 'DECREMENT': return state - 1;
    default: return state;
  }
}

// Store
const store = createStore(counter);

// Subscribe to changes
store.subscribe(() => console.log('State:', store.getState()));

// Dispatch actions
store.dispatch({ type: 'INCREMENT' }); // State: 1
store.dispatch({ type: 'INCREMENT' }); // State: 2
store.dispatch({ type: 'DECREMENT' }); // State: 1

// MVC with Express
const express = require('express');
const app = express();

// Model
class Product {
  constructor(data) { this.data = data; }
  static async getAll() { return db.products.find(); }
  static async getById(id) { return db.products.findById(id); }
}

// Controller
const productController = {
  async list(req, res) {
    const products = await Product.getAll();
    const view = req.query.format === 'json' ? 'json' : 'html';
    res.render(`products/${view}`, { products });
  }
};

// View (EJS template)
// views/products/html.ejs
// <ul><% products.forEach(p => { %>
//   <li><%= p.name %> - <%= p.price %></li>
// <% }) %></ul>
```

## Scalability Considerations

### What It Is

Scalability is the ability of a system to handle increasing load by adding resources. Horizontal scaling (adding more servers) is preferred over vertical scaling (bigger servers) because it provides near-linear capacity increases and fault tolerance.

### Why It Is Important

Applications must handle traffic spikes (Black Friday, viral content) without crashing or degrading. Scalability design anticipates growth and distributes load effectively. A non-scalable architecture requires complete rewrites as traffic grows.

### How It Works Internally

Key scalability strategies:

1. **Stateless design**: Servers don't store session state; any request can go to any server
2. **Load balancing**: Distribute requests across servers (round-robin, least connections, consistent hashing)
3. **Caching**: Reduce load on databases and computation (Redis, CDN, in-memory caches)
4. **Database scaling**: Read replicas, sharding, connection pooling
5. **Async processing**: Queue background tasks (message queues, worker processes)
6. **Microservices**: Split monolith into independent services

### Syntax

```javascript
// Stateless session management (store in Redis)
const session = require('express-session');
const RedisStore = require('connect-redis')(session);

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: { secure: true, httpOnly: true, maxAge: 86400000 }
}));

// Load balancing with Node.js cluster
const cluster = require('cluster');
const os = require('os');

if (cluster.isMaster) {
  const numWorkers = os.cpus().length;
  console.log(`Master ${process.pid} spawning ${numWorkers} workers`);
  
  for (let i = 0; i < numWorkers; i++) {
    cluster.fork();
  }
  
  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died, restarting`);
    cluster.fork();
  });
} else {
  // Worker process
  app.listen(3000);
  console.log(`Worker ${process.pid} started`);
}

// Rate limiting for scalability
const rateLimit = require('express-rate-limit');

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP
  standardHeaders: true,
  legacyHeaders: false,
  message: { error: 'Too many requests, try again later' }
});

app.use('/api/', apiLimiter);

// Database connection pooling
const { Pool } = require('pg');
const pool = new Pool({
  max: 20, // Max connections in pool
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000
});

// Query using pool
async function query(text, params) {
  const client = await pool.connect();
  try {
    return await client.query(text, params);
  } finally {
    client.release();
  }
}

// Caching with Redis
const redis = require('redis');
const client = redis.createClient();

async function getOrFetch(key, fetchFn, ttl = 300) {
  const cached = await client.get(key);
  if (cached) return JSON.parse(cached);
  
  const data = await fetchFn();
  await client.setEx(key, ttl, JSON.stringify(data));
  return data;
}
```

### Beginner Examples

```javascript
// Caching strategies

// In-memory cache (simple, single process)
class MemoryCache {
  constructor(ttl = 60000) {
    this.cache = new Map();
    this.ttl = ttl;
  }
  
  get(key) {
    const item = this.cache.get(key);
    if (!item) return undefined;
    if (Date.now() > item.expiry) {
      this.cache.delete(key);
      return undefined;
    }
    return item.value;
  }
  
  set(key, value, ttl = this.ttl) {
    this.cache.set(key, { value, expiry: Date.now() + ttl });
  }
  
  invalidate(key) { this.cache.delete(key); }
}

// Database read replicas
const readPool = new Pool({ host: 'replica.example.com' });
const writePool = new Pool({ host: 'primary.example.com' });

async function readOnly(query, params) {
  return readPool.query(query, params);
}

async function write(query, params) {
  return writePool.query(query, params);
}

// Async task queuing (Bull with Redis)
const Queue = require('bull');
const emailQueue = new Queue('email', 'redis://localhost:6379');

// Producer
app.post('/register', async (req, res) => {
  await createUser(req.body);
  await emailQueue.add({
    type: 'welcome',
    email: req.body.email,
    userId: user.id
  });
  res.status(201).json({ success: true });
});

// Consumer (worker process)
emailQueue.process(async (job) => {
  const { type, email, userId } = job.data;
  if (type === 'welcome') {
    await sendWelcomeEmail(email, userId);
  }
});
```

## Microservices with Node.js

### What It Is

Microservices architecture decomposes an application into small, independent services that communicate over the network. Each service owns its data, can be developed and deployed independently, and uses a well-defined API.

### Why It Is Important

Microservices enable team autonomy, independent scaling, technology diversity, and faster deployments. However, they introduce complexity: network latency, data consistency, distributed tracing, and service discovery.

### How It Works Internally

Microservice communication patterns:
- **HTTP/REST**: Synchronous request-response (simple but coupled)
- **Message queue**: Async event-driven (RabbitMQ, Kafka, NATS)
- **gRPC**: High-performance binary protocol
- **Event sourcing**: State changes as events
- **API Gateway**: Single entry point routing to services

Service discovery: Services register with a registry (Consul, etcd, Kubernetes DNS).
Circuit breakers: Prevent cascading failures when a service is down.

### Syntax

```javascript
// Microservice with Express (User Service)
const express = require('express');
const app = express();

// API endpoints
app.get('/api/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) return res.status(404).json({ error: 'Not found' });
  res.json(user);
});

app.post('/api/users', async (req, res) => {
  const user = await User.create(req.body);
  // Publish event for other services
  await eventBus.publish('user.created', { id: user.id });
  res.status(201).json(user);
});

// Circuit breaker pattern
const CircuitBreaker = require('opossum');

async function fetchOrderService(userId) {
  const response = await fetch(`http://order-service/api/orders?userId=${userId}`);
  return response.json();
}

const breaker = new CircuitBreaker(fetchOrderService, {
  timeout: 3000,
  errorThresholdPercentage: 50,
  resetTimeout: 30000
});

breaker.fallback(() => ({ orders: [] }));

app.get('/api/users/:id/orders', async (req, res) => {
  try {
    const orders = await breaker.fire(req.params.id);
    res.json(orders);
  } catch (err) {
    res.status(503).json({ error: 'Order service unavailable' });
  }
});

// Event bus (RabbitMQ)
const amqp = require('amqplib');

class EventBus {
  async connect(url) {
    this.conn = await amqp.connect(url);
    this.channel = await this.conn.createChannel();
    await this.channel.assertExchange('events', 'topic', { durable: true });
  }
  
  async publish(event, data) {
    this.channel.publish('events', event, Buffer.from(JSON.stringify(data)));
  }
  
  async subscribe(eventPattern, handler) {
    const q = await this.channel.assertQueue('', { exclusive: true });
    await this.channel.bindQueue(q.queue, 'events', eventPattern);
    this.channel.consume(q.queue, (msg) => {
      const data = JSON.parse(msg.content.toString());
      handler(data, msg.fields.routingKey);
      this.channel.ack(msg);
    });
  }
}
```

### Beginner Examples

```javascript
// API Gateway pattern
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');

const gateway = express();

// Route to microservices
gateway.use('/api/users', createProxyMiddleware({
  target: 'http://user-service:3001',
  changeOrigin: true
}));

gateway.use('/api/orders', createProxyMiddleware({
  target: 'http://order-service:3002',
  changeOrigin: true
}));

gateway.use('/api/products', createProxyMiddleware({
  target: 'http://product-service:3003',
  changeOrigin: true
}));

// Health check
gateway.get('/health', async (req, res) => {
  const services = ['user-service', 'order-service', 'product-service'];
  const results = await Promise.allSettled(
    services.map(s => fetch(`http://${s}/health`))
  );
  
  res.json({
    status: results.every(r => r.status === 'fulfilled') ? 'ok' : 'degraded',
    services: services.map((s, i) => ({
      name: s,
      status: results[i].status === 'fulfilled' ? 'up' : 'down'
    }))
  });
});

// Service discovery with environment variables
const SERVICES = {
  users: process.env.USER_SERVICE_URL || 'http://localhost:3001',
  orders: process.env.ORDER_SERVICE_URL || 'http://localhost:3002',
  products: process.env.PRODUCT_SERVICE_URL || 'http://localhost:3003'
};

async function callService(name, endpoint, options) {
  const baseUrl = SERVICES[name];
  const response = await fetch(`${baseUrl}${endpoint}`, options);
  if (!response.ok) throw new Error(`Service ${name} returned ${response.status}`);
  return response.json();
}
```

## State Management Strategies

### What It Is

State management controls how application state is stored, accessed, and modified. In front-end applications, this determines how UI reflects data changes. In back-end systems, it determines how request context and session data are handled.

### Why It Is Important

Poor state management leads to inconsistent UI, difficult debugging, and race conditions. As applications grow, direct state manipulation becomes unmanageable. Patterns like Redux, Zustand, and React Query provide predictable state handling.

### How It Works Internally

State management approaches:

1. **Local state**: useState (React), component-scoped state
2. **Server state**: React Query, SWR, Apollo Client (cache-first, syncs with server)
3. **Global state**: Redux, Zustand, Jotai, Recoil (shared across components)
4. **URL state**: Router state (query parameters, path)
5. **Persisted state**: AsyncStorage, localStorage (survives page reload)

### Syntax

```javascript
// Local state (React hooks)
const [user, setUser] = useState(null);
const [loading, setLoading] = useState(true);

useEffect(() => {
  fetchUser().then(u => { setUser(u); setLoading(false); });
}, []);

// Global state with Zustand
import { create } from 'zustand';

const useStore = create((set) => ({
  user: null,
  posts: [],
  loading: false,
  
  setUser: (user) => set({ user }),
  setPosts: (posts) => set({ posts }),
  setLoading: (loading) => set({ loading }),
  
  fetchUser: async (id) => {
    set({ loading: true });
    const user = await api.getUser(id);
    set({ user, loading: false });
  },
  
  addPost: (post) => set((state) => ({
    posts: [...state.posts, post]
  }))
}));

// Component
function UserProfile() {
  const { user, loading, fetchUser } = useStore();
  
  useEffect(() => { fetchUser(1); }, []);
  if (loading) return <Spinner />;
  return <div>{user?.name}</div>;
}

// Server state with TanStack Query
import { useQuery, useMutation, QueryClient } from '@tanstack/react-query';

function UserPosts({ userId }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['posts', userId],
    queryFn: () => fetch(`/api/users/${userId}/posts`).then(r => r.json()),
    staleTime: 30000 // 30 seconds before refetch
  });
  
  const mutation = useMutation({
    mutationFn: (newPost) => fetch('/api/posts', {
      method: 'POST',
      body: JSON.stringify(newPost)
    }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    }
  });
  
  if (isLoading) return <Spinner />;
  return (
    <div>
      {data?.map(post => <Post key={post.id} post={post} />)}
      <button onClick={() => mutation.mutate({ title: 'New' })}>
        Add Post
      </button>
    </div>
  );
}

// Middleware-based state (Redux with Redux Toolkit)
import { configureStore, createSlice } from '@reduxjs/toolkit';

const authSlice = createSlice({
  name: 'auth',
  initialState: { user: null, token: null },
  reducers: {
    login(state, action) {
      state.user = action.payload.user;
      state.token = action.payload.token;
    },
    logout(state) {
      state.user = null;
      state.token = null;
    }
  }
});

const store = configureStore({
  reducer: {
    auth: authSlice.reducer,
    posts: postsSlice.reducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(thunkMiddleware)
});
```

### Beginner Examples

```javascript
// Simple state management with React Context
const AppContext = createContext(null);

function AppProvider({ children }) {
  const [state, dispatch] = useReducer(appReducer, initialState);
  
  const actions = useMemo(() => ({
    login: (user) => dispatch({ type: 'LOGIN', payload: user }),
    logout: () => dispatch({ type: 'LOGOUT' }),
    addNotification: (n) => dispatch({ type: 'ADD_NOTIFICATION', payload: n })
  }), []);
  
  return (
    <AppContext.Provider value={{ state, ...actions }}>
      {children}
    </AppContext.Provider>
  );
}

function useApp() {
  const context = useContext(AppContext);
  if (!context) throw new Error('useApp must be in AppProvider');
  return context;
}

// Atomic state with Jotai
import { atom, useAtom } from 'jotai';

const countAtom = atom(0);
const doubleAtom = atom((get) => get(countAtom) * 2);
const incrementAtom = atom(null, (get, set) => set(countAtom, get(countAtom) + 1));

function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const [double] = useAtom(doubleAtom);
  const [, increment] = useAtom(incrementAtom);
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Double: {double}</p>
      <button onClick={increment}>+1</button>
    </div>
  );
}
```

### Real-World Use Cases

- E-commerce platform (product catalog + cart + checkout as microservices)
- Social media feed (real-time updates, event-driven architecture)
- Real-time collaboration (CRDT-based state synchronization)
- Dashboard application (pulling from multiple microservices)
- Banking application (strong consistency requirements)

### Common Mistakes

- Premature microservices (distributed monolith—more complex without benefits)
- Shared databases between services (coupling defeats microservices purpose)
- Synchronous communication chains (service A calls B calls C—fragile)
- Over-centralized state (everything in Redux even if local state would work)
- Underestimating data consistency challenges in distributed systems
- Not implementing circuit breakers (cascading failures)
- State management without proper typing (runtime errors)

### Best Practices

- Start with a monolith; extract microservices when justified
- Use message queues for inter-service communication
- Implement circuit breakers and retries with exponential backoff
- Use distributed tracing (OpenTelemetry) for debugging
- Each service owns its data store (no shared databases)
- Design for failure (graceful degradation, fallbacks)
- Use feature flags for gradual rollouts
- Implement comprehensive monitoring and alerting

### Performance Considerations

- Network calls between services add latency (2-10ms per call)
- Serialization/deserialization overhead (JSON vs binary protocols)
- Database calls are typically the bottleneck
- Caching at multiple levels reduces load (CDN, API gateway, application, database)
- State updates trigger re-renders; optimize with selectors and memoization
- Event-driven architectures have eventual consistency (trade latency for throughput)

### Interview Questions

**Q: When would you choose microservices over a monolith?** A: Microservices are appropriate when: (1) teams need to deploy independently, (2) different parts of the system have different scaling requirements, (3) technology diversity is needed, (4) the monolith has grown too large to maintain. Start with a monolith and extract services when the monolith's complexity outweighs microservices' overhead.

**Q: How do you handle state synchronization in a distributed system?** A: Approaches depend on consistency requirements. Strong consistency: use distributed transactions (Saga pattern) or consensus algorithms (Raft). Eventual consistency: use event sourcing with idempotent handlers. For user-facing features, optimistic UI updates combined with background synchronization provide good UX. For critical data (payments), implement compensating transactions.


### Coding Challenges
```javascript
// Challenge 1: Simple state management (mini-Redux)
function createStore(reducer, initialState) {
  let state = initialState;
  const listeners = [];
  return {
    getState: () => state,
    dispatch: action => { state = reducer(state, action); listeners.forEach(l => l()); },
    subscribe: listener => { listeners.push(listener); return () => listeners.splice(listeners.indexOf(listener), 1); }
  };
}
const counterReducer = (state = 0, action) => {
  switch(action.type) { case 'INCREMENT': return state + 1; case 'DECREMENT': return state - 1; default: return state; }
};
const store = createStore(counterReducer, 0);
store.subscribe(() => console.log('State:', store.getState()));
store.dispatch({ type: 'INCREMENT' });

// Challenge 2: LRU Cache for high-performance systems
class LRUCache {
  constructor(capacity) { this.capacity = capacity; this.cache = new Map(); }
  get(key) { if (!this.cache.has(key)) return -1; const val = this.cache.get(key); this.cache.delete(key); this.cache.set(key, val); return val; }
  put(key, value) { if (this.cache.has(key)) this.cache.delete(key); this.cache.set(key, value); if (this.cache.size > this.capacity) this.cache.delete(this.cache.keys().next().value); }
}
const cache = new LRUCache(2);
cache.put(1, 1); cache.put(2, 2); console.log(cache.get(1));
```

### Related Topics

- Monorepo vs multi-repo (code organization)
- Containerization (Docker, Kubernetes)
- API design (REST, GraphQL, gRPC)
- Event-driven architecture
- CQRS (Command Query Responsibility Segregation)
- Saga pattern for distributed transactions
- 12-Factor App methodology
- Observability (logging, metrics, tracing)
- CI/CD for microservices
- Front-end architecture (micro-frontends)
