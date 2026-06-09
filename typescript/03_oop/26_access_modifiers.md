# Access Modifiers - public, private, protected, readonly, parameter properties, cross-class access

## Introduction
Access modifiers in TypeScript control the visibility and mutability of class members. They are fundamental to encapsulation, enabling developers to define clear public APIs while hiding implementation details. TypeScript provides `public`, `private`, `protected`, and `readonly` modifiers, along with the ability to combine them in parameter properties. Understanding these modifiers is essential for designing robust, maintainable object-oriented systems.

## public modifier

### What It Is
The `public` modifier makes a class member accessible from anywhere — within the class, in derived classes, and from external code. It is the default access level for all class members.

### Why It Is Important
`public` defines the component's API surface. Every piece of code that interacts with the class does so through its public members. Explicitly marking members as `public` documents the intended interface.

### How It Works Internally
`public` is the default and produces no special compiler behavior. The emitted JavaScript is identical to an unannotated member. It is purely a type-system annotation.

### Syntax
```typescript
class Example {
  public name: string;
  public method(): void {}
}
```

### Beginner Examples
```typescript
class Clock {
  public now: Date;
  public constructor() {
    this.now = new Date();
  }
  public getTime(): string {
    return this.now.toLocaleTimeString();
  }
}
const c = new Clock();
console.log(c.getTime()); // OK
c.now = new Date();        // OK (public write access)
```

### Intermediate Examples
```typescript
class EventBus {
  public listeners = new Map<string, Set<Function>>();
  public on(event: string, handler: Function): void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(handler);
  }
  public emit(event: string, ...args: unknown[]): void {
    const handlers = this.listeners.get(event);
    if (handlers) {
      handlers.forEach(h => h(...args));
    }
  }
  public off(event: string, handler: Function): void {
    this.listeners.get(event)?.delete(handler);
  }
}
```

### Advanced Examples
```typescript
class ObservableState<T extends Record<string, unknown>> {
  public state: T;
  private subscribers = new Set<(state: T) => void>();
  public constructor(initial: T) {
    this.state = { ...initial };
  }
  public subscribe(fn: (state: T) => void): () => void {
    this.subscribers.add(fn);
    return () => this.subscribers.delete(fn);
  }
  public setState(partial: Partial<T>): void {
    this.state = { ...this.state, ...partial };
    this.subscribers.forEach(fn => fn(this.state));
  }
  public getState(): Readonly<T> {
    return Object.freeze({ ...this.state });
  }
}
```

### Real-World Use Cases
- Service methods intended for external consumption
- Component lifecycle hooks in frameworks
- Configuration and initialization interfaces

### Common Mistakes
- Making everything `public` by default, violating encapsulation
- Exposing mutable internal collections as `public` arrays or maps
- Assuming `public` means "unchangeable" — public fields can be reassigned

### Best Practices
- Start with `private` and promote to `public` only when necessary
- Use getters for read-only public access to internal state
- Document the public API thoroughly

### Performance Considerations
No runtime impact. Access modifiers are compile-time only.

### Interview Questions
- Q: What is the default access modifier in TypeScript?
  A: `public` is the default.

### Coding Challenges
- Design a `TaskQueue` class with public methods `enqueue`, `dequeue`, and `size`.
- Create a `Logger` with public methods `info`, `warn`, and `error` that internally route to a private writer.

### Related Topics
- private modifier
- protected modifier
- Parameter properties

---

## private modifier

### What It Is
TypeScript's `private` modifier restricts access to the declaring class only. Subclasses and external code cannot access private members.

### Why It Is Important
`private` enforces encapsulation by hiding implementation details. It prevents external coupling to internal structures, making it safe to refactor internals without breaking consumers.

### How It Works Internally
`private` is enforced at compile time only. The emitted JavaScript has no access control — private properties remain accessible at runtime via bracket notation or property access.

### Syntax
```typescript
class Example {
  private secret: string = "hidden";
  private method(): void {}
}
```

### Beginner Examples
```typescript
class BankAccount {
  private balance: number;
  constructor(initial: number) {
    this.balance = initial;
  }
  public deposit(amount: number): void {
    if (amount <= 0) throw new Error("Invalid amount");
    this.balance += amount;
  }
  public withdraw(amount: number): boolean {
    if (amount > this.balance) return false;
    this.balance -= amount;
    return true;
  }
  public getBalance(): number {
    return this.balance;
  }
}
```

### Intermediate Examples
```typescript
class TokenManager {
  private tokens = new Set<string>();
  private blacklist = new Set<string>();
  public generate(userId: string): string {
    const token = crypto.randomUUID();
    this.tokens.add(token);
    return token;
  }
  public validate(token: string): boolean {
    return this.tokens.has(token) && !this.blacklist.has(token);
  }
  public revoke(token: string): void {
    this.blacklist.add(token);
  }
  private cleanup(): void {
    this.blacklist.clear();
  }
}
```

### Advanced Examples
```typescript
class ConnectionPool {
  private static instance: ConnectionPool;
  private connections: Map<string, DatabaseConnection> = new Map();
  private maxConnections: number;
  private constructor(maxConnections = 10) {
    this.maxConnections = maxConnections;
  }
  public static getInstance(max?: number): ConnectionPool {
    if (!ConnectionPool.instance) {
      ConnectionPool.instance = new ConnectionPool(max);
    }
    return ConnectionPool.instance;
  }
  private async createConnection(id: string): Promise<DatabaseConnection> {
    const conn = new DatabaseConnection(id);
    await conn.connect();
    return conn;
  }
  public async acquire(id: string): Promise<DatabaseConnection> {
    if (!this.connections.has(id)) {
      if (this.connections.size >= this.maxConnections) {
        throw new Error("Connection pool exhausted");
      }
      this.connections.set(id, await this.createConnection(id));
    }
    return this.connections.get(id)!;
  }
}
```

### Real-World Use Cases
- Hiding internal data structures (caches, maps, queues)
- Encapsulating implementation details in service classes
- Protecting internal state from external mutation

### Common Mistakes
- Assuming `private` provides runtime security
- Using `private` when `protected` is needed for subclass access
- Declaring private fields that shadow parent class private fields

### Best Practices
- Use `private` by default; only promote to `protected` or `public` when needed
- Prefer ECMAScript `#` private fields for true runtime privacy
- Combine `private` with `readonly` for immutable internal state

### Performance Considerations
No runtime cost. Compile-time enforcement only.

### Interview Questions
- Q: What is the difference between TypeScript `private` and ECMAScript `#` private fields?
  A: TypeScript `private` is compile-time only; `#` fields are enforced at the JavaScript runtime level.

### Coding Challenges
- Implement a `Cache` class with private `Map` storage and public `get`, `set`, and `clear` methods.
- Create a `RateLimiter` with private tracking of request timestamps per user.

### Related Topics
- protected modifier
- ECMAScript private fields (#)
- Encapsulation

---

## protected modifier

### What It Is
The `protected` modifier allows access within the declaring class and any subclass, but prevents access from external code.

### Why It Is Important
`protected` enables base classes to expose internal helpers to subclasses without making them part of the public API. It supports the Template Method pattern and other inheritance-based designs.

### How It Works Internally
Like `private`, `protected` is compile-time enforced. The emitted JavaScript has no access restrictions.

### Syntax
```typescript
class Base {
  protected helper(): void {}
}
class Derived extends Base {
  useHelper(): void { this.helper(); } // OK
}
```

### Beginner Examples
```typescript
class Document {
  protected content: string;
  constructor(content: string) {
    this.content = content;
  }
  protected format(): string {
    return this.content;
  }
}
class PDFDocument extends Document {
  public export(): string {
    return this.format(); // OK: protected access
  }
}
// const doc = new PDFDocument("hello");
// doc.content; // Error: protected
```

### Intermediate Examples
```typescript
abstract class HttpHandler {
  protected abstract handleRequest(req: Request): Promise<Response>;
  protected logRequest(req: Request): void {
    console.log(`[${req.method}] ${req.url}`);
  }
  protected handleError(error: Error): Response {
    return new Response(error.message, { status: 500 });
  }
  public async handle(req: Request): Promise<Response> {
    try {
      this.logRequest(req);
      return await this.handleRequest(req);
    } catch (error) {
      return this.handleError(error as Error);
    }
  }
}
class UserHandler extends HttpHandler {
  protected async handleRequest(req: Request): Promise<Response> {
    const user = await getUser(req.params.id);
    return new Response(JSON.stringify(user));
  }
}
```

### Advanced Examples
```typescript
abstract class Plugin {
  protected abstract name: string;
  protected abstract version: string;
  protected metadata: Record<string, unknown> = {};
  protected setMetadata(key: string, value: unknown): void {
    this.metadata[key] = value;
  }
  public abstract initialize(): Promise<void>;
  public abstract execute(context: PluginContext): Promise<unknown>;
}
class AuthPlugin extends Plugin {
  protected name = "auth";
  protected version = "1.0.0";
  protected async initialize(): Promise<void> {
    this.setMetadata("provider", "oauth2");
    // initialization logic
  }
  public async execute(context: PluginContext): Promise<unknown> {
    // auth logic
    return { authenticated: true };
  }
}
```

### Real-World Use Cases
- Template method pattern with protected steps
- Base service classes with shared helpers
- Framework component lifecycle hooks

### Common Mistakes
- Using `protected` when `private` is sufficient
- Overriding protected methods with incompatible signatures
- Accessing protected members via unrelated sibling classes

### Best Practices
- Prefer `private` and only use `protected` when subclass access is explicitly needed
- Document the protected API contract for subclass implementors
- Consider whether a protected method should be abstract

### Performance Considerations
No runtime cost. Compile-time only.

### Interview Questions
- Q: Can a derived class access a protected member of the base class through an instance of the base class?
  A: Yes, TypeScript allows this as long as the access occurs within the derived class.

### Coding Challenges
- Create a `Report` base class with protected `buildHeader()`, `buildBody()`, and `buildFooter()` methods. Implement `HTMLReport` and `CSVReport`.
- Build a `Validator` base class with `protected addError()` and `public validate(): boolean`.

### Related Topics
- private modifier
- public modifier
- Parameter properties

---

## readonly modifier

### What It Is
The `readonly` modifier marks a property as immutable after initialization. It can be assigned only at declaration or in the constructor.

### Why It Is Important
`readonly` prevents accidental reassignment, documents intent, and supports immutable design patterns. It is essential for value objects and configuration classes.

### How It Works Internally
`readonly` is a compile-time constraint. The emitted JavaScript has no runtime enforcement. Properties can still be mutated via `Object.defineProperty` or direct assignment in non-strict mode.

### Syntax
```typescript
class Example {
  readonly id: string;
  readonly createdAt: Date = new Date();
  constructor(id: string) {
    this.id = id;
  }
}
```

### Beginner Examples
```typescript
class User {
  readonly id: string;
  readonly registeredAt: Date;
  public name: string;
  constructor(id: string, name: string) {
    this.id = id;
    this.name = name;
    this.registeredAt = new Date();
  }
}
const u = new User("u1", "Alice");
// u.id = "u2"; // Error: readonly
```

### Intermediate Examples
```typescript
class AppConfig {
  readonly port: number;
  readonly host: string;
  readonly databaseUrl: string;
  readonly debug: boolean;
  constructor(env: NodeJS.ProcessEnv) {
    this.port = parseInt(env.PORT ?? "3000", 10);
    this.host = env.HOST ?? "localhost";
    this.databaseUrl = env.DATABASE_URL ?? "sqlite://:memory:";
    this.debug = env.NODE_ENV === "development";
  }
}
```

### Advanced Examples
```typescript
class ImmutableVector3 {
  readonly x: number;
  readonly y: number;
  readonly z: number;
  constructor(x: number, y: number, z: number = 0) {
    this.x = x;
    this.y = y;
    this.z = z;
  }
  add(v: ImmutableVector3): ImmutableVector3 {
    return new ImmutableVector3(this.x + v.x, this.y + v.y, this.z + v.z);
  }
  scale(factor: number): ImmutableVector3 {
    return new ImmutableVector3(this.x * factor, this.y * factor, this.z * factor);
  }
  dot(v: ImmutableVector3): number {
    return this.x * v.x + this.y * v.y + this.z * v.z;
  }
  magnitude(): number {
    return Math.sqrt(this.x ** 2 + this.y ** 2 + this.z ** 2);
  }
  normalize(): ImmutableVector3 {
    const mag = this.magnitude();
    return mag === 0 ? this : this.scale(1 / mag);
  }
}
```

### Real-World Use Cases
- Domain entity identifiers and timestamps
- Application configuration loaded at startup
- Value objects (Money, Point, DateRange)
- Builder product results

### Common Mistakes
- Confusing `readonly` with immutability — `readonly` only prevents property reassignment, not mutation of referenced objects
- Using `readonly` with arrays (use `ReadonlyArray<T>` or `readonly T[]`)
- Assuming `readonly` properties are enforced at runtime

### Best Practices
- Use `readonly` for all properties that should not change post-construction
- Combine with parameter properties: `constructor(public readonly id: string) {}`
- For deep immutability, use `ReadonlyDeep<T>` or `as const`

### Performance Considerations
No runtime cost. Compile-time enforcement.

### Interview Questions
- Q: Can you assign a readonly property in a method other than the constructor?
  A: No, readonly properties can only be assigned in their declaration or in the constructor.

### Coding Challenges
- Create an `ImmutableDateRange` class with readonly `start` and `end`, methods `overlaps()`, `contains()`, and `duration()`.
- Build a `ReadonlyConfig` class that wraps a config object and exposes only readonly access.

### Related Topics
- public modifier
- private modifier
- Parameter properties

---

## Parameter properties with modifiers

### What It Is
Parameter properties combine constructor parameter declaration with property declaration in one syntax by adding an access modifier to a constructor parameter.

### Why It Is Important
Parameter properties eliminate boilerplate by automatically creating and initializing class fields from constructor parameters. They make class definitions more concise and readable.

### How It Works Internally
TypeScript desugars each parameter property into a field declaration and an assignment statement in the constructor body. The emitted JavaScript includes both.

### Syntax
```typescript
class Example {
  constructor(
    public name: string,
    private id: number,
    protected readonly config: Config
  ) {}
}
```

### Beginner Examples
```typescript
class Task {
  constructor(
    public title: string,
    public completed: boolean = false,
    private readonly createdAt: Date = new Date()
  ) {}
}
const task = new Task("Buy milk");
console.log(task.title); // "Buy milk"
// task.createdAt; // Error: private
```

### Intermediate Examples
```typescript
class ApiClient {
  constructor(
    private readonly baseURL: string,
    private readonly timeout: number = 5000,
    private readonly headers: Record<string, string> = {},
    public readonly name: string = "ApiClient"
  ) {}
  async get<T>(path: string): Promise<T> {
    const res = await fetch(`${this.baseURL}${path}`, {
      headers: this.headers,
      signal: AbortSignal.timeout(this.timeout),
    });
    return res.json();
  }
}
```

### Advanced Examples
```typescript
interface IDatabaseConfig {
  host: string;
  port: number;
  username: string;
  password: string;
  database: string;
}
class DatabaseService {
  private pool: ConnectionPool;
  constructor(
    private readonly config: IDatabaseConfig,
    private readonly maxPoolSize: number = 10,
    private readonly logger: Logger = new ConsoleLogger(),
    public readonly name: string = "DatabaseService"
  ) {
    this.pool = new ConnectionPool({
      host: this.config.host,
      port: this.config.port,
      user: this.config.username,
      password: this.config.password,
      database: this.config.database,
      max: this.maxPoolSize,
    });
  }
  async query(sql: string, params?: unknown[]): Promise<unknown[]> {
    this.logger.info(`Executing query: ${sql}`);
    const conn = await this.pool.acquire();
    try {
      return await conn.query(sql, params);
    } finally {
      await this.pool.release(conn);
    }
  }
  async close(): Promise<void> {
    await this.pool.drain();
    this.logger.info("Database connection closed");
  }
}
```

```typescript
// Combining parameter properties with dependency injection
interface Logger {
  log(level: string, message: string): void;
}
interface CacheService {
  get<T>(key: string): Promise<T | null>;
  set<T>(key: string, value: T, ttl?: number): Promise<void>;
}
class UserService {
  constructor(
    private readonly logger: Logger,
    private readonly cache: CacheService,
    private readonly db: DatabaseService,
    public readonly basePath: string = "/api/users"
  ) {}
  async getUser(id: string): Promise<User | null> {
    const cached = await this.cache.get<User>(`user:${id}`);
    if (cached) {
      this.logger.log("debug", `Cache hit for user ${id}`);
      return cached;
    }
    const user = await this.db.query<User>(
      "SELECT * FROM users WHERE id = $1", [id]
    );
    if (user) {
      await this.cache.set(`user:${id}`, user, 300);
    }
    return user;
  }
}
```

### Real-World Use Cases
- Angular/NestJS service classes with dependency injection
- DTO and entity classes with many constructor parameters
- Configuration classes that combine injection with defaults

### Common Mistakes
- Mixing regular constructor parameters with parameter properties — non-modifier parameters are not automatically assigned
- Using `private` when the property needs to be accessed in tests or by subclasses
- Forgetting that parameter properties create instance properties that may shadow parent class properties

### Best Practices
- Use parameter properties consistently across a project
- Use `private readonly` for injected dependencies
- Use `public readonly` for externally readable config values
- Avoid parameter properties when constructor logic beyond simple assignment is needed

### Performance Considerations
Identical to manually declaring fields and assigning them. No performance impact.

### Interview Questions
- Q: Can you combine parameter properties with regular constructor parameters?
  A: Yes. Constructor parameters without modifiers are regular parameters and must be manually assigned.

### Coding Challenges
- Define a `ConfigService` class using parameter properties for port, host, database URL, and log level.
- Create a `Repository<T>` class that takes a `DatabaseService` and `CacheService` as parameter properties.

### Related Topics
- public modifier
- private modifier
- readonly modifier
- Constructor and fields
