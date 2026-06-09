# Design Patterns - Singleton, Factory, Builder, Strategy in TS, dependency injection

## Introduction

Design patterns are reusable solutions to common software design problems. TypeScript's type system enhances traditional design patterns by adding type safety, making them more robust and self-documenting. This file covers implementing Singleton, Factory, Builder, and Strategy patterns in TypeScript, as well as a simple dependency injection container.

## Singleton Pattern in TS

### What It Is

The Singleton pattern ensures a class has only one instance and provides a global point of access to it. TypeScript's type system helps enforce the pattern at compile time.

### Why It Is Important

Singletons are useful for managing shared resources like database connections, configuration objects, logger instances, and caches. TypeScript ensures that the singleton contract is enforced and that the single instance is correctly typed.

### How It Works Internally

The private constructor prevents external instantiation. A static method or property holds the single instance. TypeScript's `private` constructor keyword enforces this at compile time.

### Syntax

```typescript
class Singleton {
  private static instance: Singleton;

  private constructor() {}

  static getInstance(): Singleton {
    if (!Singleton.instance) {
      Singleton.instance = new Singleton();
    }
    return Singleton.instance;
  }
}

const instance1 = Singleton.getInstance();
const instance2 = Singleton.getInstance();
console.log(instance1 === instance2); // true
```

### Beginner Examples

```typescript
class Logger {
  private static instance: Logger;
  private logs: string[] = [];

  private constructor() {}

  static getInstance(): Logger {
    if (!Logger.instance) {
      Logger.instance = new Logger();
    }
    return Logger.instance;
  }

  log(message: string): void {
    this.logs.push(`[${new Date().toISOString()}] ${message}`);
  }

  getLogs(): string[] {
    return [...this.logs];
  }
}

const logger = Logger.getInstance();
logger.log('Application started');
```

### Intermediate Examples

```typescript
// Generic singleton
class SingletonFactory<T> {
  private static instances = new Map<string, any>();

  static getInstance<T>(key: string, factory: () => T): T {
    if (!this.instances.has(key)) {
      this.instances.set(key, factory());
    }
    return this.instances.get(key)!;
  }
}

const dbConnection = SingletonFactory.getInstance('db', () => ({
  host: 'localhost',
  port: 5432,
  connect: () => console.log('Connected'),
}));

// Singleton with initialization
class ConfigManager {
  private static instance: ConfigManager;
  private config: Record<string, unknown> = {};

  private constructor() {}

  static async initialize(configPath: string): Promise<ConfigManager> {
    if (ConfigManager.instance) {
      throw new Error('ConfigManager already initialized');
    }
    const instance = new ConfigManager();
    instance.config = await loadConfig(configPath);
    ConfigManager.instance = instance;
    return instance;
  }

  static getInstance(): ConfigManager {
    if (!ConfigManager.instance) {
      throw new Error('ConfigManager not initialized. Call initialize() first.');
    }
    return ConfigManager.instance;
  }

  get<T>(key: string): T {
    return this.config[key] as T;
  }
}
```

### Advanced Examples

```typescript
// Singleton with module pattern (alternative approach)
class DatabasePool {
  private static instance: DatabasePool;
  private pool: any[];
  private readonly maxSize: number;

  private constructor(maxSize: number = 10) {
    this.maxSize = maxSize;
    this.pool = [];
  }

  static getInstance(maxSize?: number): DatabasePool {
    if (!DatabasePool.instance) {
      DatabasePool.instance = new DatabasePool(maxSize);
    }
    return DatabasePool.instance;
  }

  async acquire(): Promise<any> {
    if (this.pool.length > 0) {
      return this.pool.pop();
    }
    return this.createConnection();
  }

  release(connection: any): void {
    if (this.pool.length < this.maxSize) {
      this.pool.push(connection);
    }
  }

  private async createConnection(): Promise<any> {
    return { id: Math.random(), query: (sql: string) => console.log(sql) };
  }
}

// Module-level singleton (simpler alternative)
const dbPool = new (class {
  private connections: any[] = [];
  acquire() { return Promise.resolve({}); }
  release(conn: any) { this.connections.push(conn); }
})();

export default dbPool;
```

### Real-World Use Cases

```typescript
class AuthService {
  private static instance: AuthService;
  private currentUser: { id: string; token: string } | null = null;

  private constructor() {}

  static getInstance(): AuthService {
    if (!AuthService.instance) {
      AuthService.instance = new AuthService();
    }
    return AuthService.instance;
  }

  async login(email: string, password: string): Promise<void> {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password }),
    });
    this.currentUser = await response.json();
  }

  logout(): void {
    this.currentUser = null;
  }

  getToken(): string | null {
    return this.currentUser?.token ?? null;
  }

  isAuthenticated(): boolean {
    return this.currentUser !== null;
  }
}
```

### Common Mistakes

```typescript
// MISTAKE: Not making the constructor private
// class BadSingleton { static instance: BadSingleton; constructor() {} }
// Anyone can create instances

// MISTAKE: Thread-unsafe lazy initialization (relevant in concurrent environments)
// Use Node.js module caching instead for safe singletons

// MISTAKE: Overusing singletons (makes testing difficult)
```

### Best Practices

- Use `private constructor` to prevent external instantiation.
- Use `static getInstance()` method for controlled access.
- Consider using module-level instances (ES module caching) as a simpler alternative.
- Make singletons testable by allowing instance reset in tests.
- Avoid overusing singletons; prefer dependency injection for testability.

### Performance Considerations

Singleton pattern adds minimal overhead (one extra function call). The instance is created once and reused. Module-level singletons (ES modules) are thread-safe due to Node.js module caching.

### Interview Questions

1. How do you implement a Singleton in TypeScript?
2. What are the alternatives to the Singleton pattern in TypeScript?
3. How do you make a Singleton testable?

### Coding Challenges

1. Implement a thread-safe Singleton cache with TTL support.
2. Create a Singleton event bus that can be used across modules.

### Related Topics

- Factory pattern
- Dependency injection
- Module pattern

## Factory Pattern in TS

### What It Is

The Factory pattern provides an interface for creating objects without specifying their concrete classes. TypeScript's discriminated unions and generics make factories type-safe and expressive.

### Why It Is Important

Factories encapsulate object creation logic, making code more maintainable and testable. TypeScript ensures that factory methods return correctly typed objects and that creation parameters are validated.

### How It Works Internally

A factory function or class abstracts the instantiation logic. It typically takes configuration parameters and returns an instance of a common interface or type.

### Syntax

```typescript
interface Product {
  operation(): string;
}

class ConcreteProductA implements Product {
  operation() { return 'Product A'; }
}

class ConcreteProductB implements Product {
  operation() { return 'Product B'; }
}

class Creator {
  static create(type: 'A' | 'B'): Product {
    switch (type) {
      case 'A': return new ConcreteProductA();
      case 'B': return new ConcreteProductB();
    }
  }
}
```

### Beginner Examples

```typescript
type LoggerType = 'console' | 'file' | 'memory';

interface ILogger {
  log(message: string): void;
}

class ConsoleLogger implements ILogger {
  log(message: string) { console.log(message); }
}

class FileLogger implements ILogger {
  log(message: string) { /* write to file */ }
}

class LoggerFactory {
  static createLogger(type: LoggerType): ILogger {
    switch (type) {
      case 'console': return new ConsoleLogger();
      case 'file': return new FileLogger();
      case 'memory': return { log: (msg: string) => { /* store in memory */ } };
    }
  }
}
```

### Intermediate Examples

```typescript
// Generic factory with registration
type Constructor<T> = new (...args: any[]) => T;

class Factory<T> {
  private registry = new Map<string, Constructor<T>>();

  register(name: string, ctor: Constructor<T>): void {
    this.registry.set(name, ctor);
  }

  create(name: string, ...args: any[]): T {
    const ctor = this.registry.get(name);
    if (!ctor) throw new Error(`No constructor registered for ${name}`);
    return new ctor(...args);
  }
}

interface Shape {
  draw(): void;
}

class Circle implements Shape {
  constructor(private radius: number) {}
  draw() { console.log(`Circle with radius ${this.radius}`); }
}

class Square implements Shape {
  constructor(private side: number) {}
  draw() { console.log(`Square with side ${this.side}`); }
}

const shapeFactory = new Factory<Shape>();
shapeFactory.register('circle', Circle);
shapeFactory.register('square', Square);

const circle = shapeFactory.create('circle', 5);
```

### Advanced Examples

```typescript
// Async factory with validation
interface UserInput {
  name: string;
  email: string;
  role: 'admin' | 'user';
}

interface User {
  id: string;
  name: string;
  email: string;
  role: string;
  createdAt: Date;
}

class UserFactory {
  static async create(data: UserInput): Promise<User> {
    const validated = await UserFactory.validate(data);
    return {
      id: crypto.randomUUID(),
      ...validated,
      createdAt: new Date(),
    };
  }

  private static async validate(data: UserInput): Promise<UserInput> {
    if (!data.email.includes('@')) throw new Error('Invalid email');
    if (data.name.length < 2) throw new Error('Name too short');
    return data;
  }
}

// Factory with builder pattern
class QueryFactory {
  static select<T extends Record<string, any>>(table: string): QueryBuilder<T> {
    return new QueryBuilder<T>(table);
  }
}

class QueryBuilder<T> {
  private conditions: string[] = [];

  constructor(private table: string) {}

  where<K extends keyof T>(field: K, value: T[K]): this {
    this.conditions.push(`${String(field)} = ${value}`);
    return this;
  }

  build(): string {
    const where = this.conditions.length > 0
      ? ` WHERE ${this.conditions.join(' AND ')}`
      : '';
    return `SELECT * FROM ${this.table}${where}`;
  }
}
```

### Real-World Use Cases

```typescript
// Notification system
type NotificationType = 'email' | 'sms' | 'push';

interface NotificationPayload {
  to: string;
  body: string;
  subject?: string;
}

interface INotificationService {
  send(payload: NotificationPayload): Promise<boolean>;
}

class EmailService implements INotificationService {
  async send(payload: NotificationPayload) {
    console.log(`Email sent to ${payload.to}: ${payload.subject}`);
    return true;
  }
}

class SMSService implements INotificationService {
  async send(payload: NotificationPayload) {
    console.log(`SMS sent to ${payload.to}: ${payload.body}`);
    return true;
  }
}

class PushService implements INotificationService {
  async send(payload: NotificationPayload) {
    console.log(`Push sent to ${payload.to}`);
    return true;
  }
}

class NotificationFactory {
  private static services: Record<NotificationType, new () => INotificationService> = {
    email: EmailService,
    sms: SMSService,
    push: PushService,
  };

  static create(type: NotificationType): INotificationService {
    return new this.services[type]();
  }
}
```

### Common Mistakes

```typescript
// MISTAKE: Using factory when a simple constructor suffices
// Factory pattern adds indirection; use it only when creation logic is complex

// MISTAKE: Not typing the return type of the factory
// Always return a typed interface, not concrete types

// MISTAKE: Factory method with too many parameters
// Use builder pattern or parameter objects instead
```

### Best Practices

- Always return an interface type from factory methods.
- Use factory registration pattern for extensible factories.
- Use async factories for creation logic involving I/O or validation.
- Combine factory with builder for complex object construction.
- Keep factory methods focused; avoid feature envy.

### Performance Considerations

Factory pattern adds minimal overhead (one extra function call). Registration-based factories use Map lookups, which are O(1). Async factories add Promise overhead.

### Interview Questions

1. What is the Factory pattern and when would you use it?
2. How do you implement a factory with registration in TypeScript?
3. How does the Factory pattern differ from the Builder pattern?

### Coding Challenges

1. Create a factory for database connections that returns different driver instances based on configuration.
2. Build a pluggable parser factory that registers parsers for different file formats.

### Related Topics

- Builder pattern
- Strategy pattern
- Dependency injection

## Builder Pattern in TS

### What It Is

The Builder pattern separates the construction of a complex object from its representation. It allows the same construction process to create different representations.

### Why It Is Important

The Builder pattern is ideal for objects with many optional parameters, complex validation, or multi-step construction. TypeScript ensures that builder methods return the correct types and that the built object is complete.

### How It Works Internally

A builder class exposes chainable methods that accumulate state. A `build()` method validates the accumulated state and creates the final object.

### Syntax

```typescript
class ProductBuilder {
  private product: { partA?: string; partB?: string; partC?: string } = {};

  setPartA(value: string): this {
    this.product.partA = value;
    return this;
  }

  setPartB(value: string): this {
    this.product.partB = value;
    return this;
  }

  build(): { partA: string; partB: string } {
    if (!this.product.partA) throw new Error('partA is required');
    return { partA: this.product.partA, partB: this.product.partB || '' };
  }
}
```

### Beginner Examples

```typescript
class UserBuilder {
  private user: {
    name: string;
    email?: string;
    age?: number;
    phone?: string;
    address?: string;
  } = { name: '' };

  setName(name: string): this {
    this.user.name = name;
    return this;
  }

  setEmail(email: string): this {
    this.user.email = email;
    return this;
  }

  setAge(age: number): this {
    this.user.age = age;
    return this;
  }

  build(): { name: string; email?: string; age?: number } {
    if (!this.user.name) throw new Error('Name is required');
    return { ...this.user };
  }
}

const user = new UserBuilder()
  .setName('Alice')
  .setEmail('alice@test.com')
  .setAge(30)
  .build();
```

### Intermediate Examples

```typescript
// Query builder
class QueryBuilder<T extends Record<string, any>> {
  private selects: (keyof T)[] = [];
  private whereClauses: string[] = [];
  private orderByField: keyof T | null = null;
  private orderDirection: 'ASC' | 'DESC' = 'ASC';
  private limitCount: number | null = null;

  select(...fields: (keyof T)[]): this {
    this.selects = fields;
    return this;
  }

  where(field: keyof T, value: T[keyof T]): this {
    this.whereClauses.push(`${String(field)} = ${JSON.stringify(value)}`);
    return this;
  }

  orderBy(field: keyof T, direction: 'ASC' | 'DESC' = 'ASC'): this {
    this.orderByField = field;
    this.orderDirection = direction;
    return this;
  }

  limit(count: number): this {
    this.limitCount = count;
    return this;
  }

  build(): string {
    const select = this.selects.length > 0
      ? this.selects.map(String).join(', ')
      : '*';
    let query = `SELECT ${select} FROM ${this.table}`;
    if (this.whereClauses.length > 0) {
      query += ` WHERE ${this.whereClauses.join(' AND ')}`;
    }
    if (this.orderByField) {
      query += ` ORDER BY ${String(this.orderByField)} ${this.orderDirection}`;
    }
    if (this.limitCount !== null) {
      query += ` LIMIT ${this.limitCount}`;
    }
    return query;
  }

  constructor(private table: string) {}
}
```

### Advanced Examples

```typescript
// Type-safe builder with discriminated build result
class ResponseBuilder<T> {
  private statusCode: number = 200;
  private headers: Record<string, string> = {};
  private body: T | null = null;

  status(code: number): this {
    this.statusCode = code;
    return this;
  }

  header(name: string, value: string): this {
    this.headers[name] = value;
    return this;
  }

  json(data: T): this {
    this.body = data;
    this.header('Content-Type', 'application/json');
    return this;
  }

  build(): { statusCode: number; headers: Record<string, string>; body: T | null } {
    return {
      statusCode: this.statusCode,
      headers: this.headers,
      body: this.body,
    };
  }
}

// Fluent builder with generics
class ConfigBuilder<T extends Record<string, any>> {
  private config: Partial<T> = {};

  set<K extends keyof T>(key: K, value: T[K]): this {
    this.config[key] = value;
    return this;
  }

  merge(other: Partial<T>): this {
    this.config = { ...this.config, ...other };
    return this;
  }

  build(): T {
    return this.config as T;
  }
}

interface AppConfig {
  host: string;
  port: number;
  debug: boolean;
  apiKey: string;
}

const config = new ConfigBuilder<AppConfig>()
  .set('host', 'localhost')
  .set('port', 3000)
  .set('debug', true)
  .set('apiKey', 'abc123')
  .build();
```

### Real-World Use Cases

```typescript
// HTTP request builder
class RequestBuilder {
  private method: string = 'GET';
  private url: string = '';
  private headers: Record<string, string> = {};
  private body: unknown = null;
  private queryParams: Record<string, string> = {};

  get(url: string): this {
    this.method = 'GET';
    this.url = url;
    return this;
  }

  post(url: string): this {
    this.method = 'POST';
    this.url = url;
    return this;
  }

  header(name: string, value: string): this {
    this.headers[name] = value;
    return this;
  }

  query(name: string, value: string): this {
    this.queryParams[name] = value;
    return this;
  }

  send<T>(data: T): this {
    this.body = data;
    return this;
  }

  async execute<T>(): Promise<T> {
    const params = new URLSearchParams(this.queryParams).toString();
    const fullUrl = params ? `${this.url}?${params}` : this.url;
    const response = await fetch(fullUrl, {
      method: this.method,
      headers: this.headers,
      body: this.body ? JSON.stringify(this.body) : undefined,
    });
    return response.json() as Promise<T>;
  }
}

// Usage
const result = await new RequestBuilder()
  .post('https://api.example.com/users')
  .header('Authorization', 'Bearer token')
  .header('Content-Type', 'application/json')
  .send({ name: 'Alice', email: 'alice@test.com' })
  .execute<{ id: string }>();
```

### Common Mistakes

```typescript
// MISTAKE: Not returning `this` from builder methods
// setPartA(value: string) { this.part = value; } // Breaks chaining

// MISTAKE: Not validating in build()
// build() should check that required fields are set

// MISTAKE: Mutable builder state shared across builds
// Each build() should create a new instance or reset state
```

### Best Practices

- Return `this` from all setter methods for chaining.
- Validate required fields in the `build()` method.
- Use generics for type-safe builder results.
- Make builders immutable (return new builder state) for complex scenarios.
- Name methods clearly: `withProperty`, `setProperty`, `addProperty`.

### Performance Considerations

Builder pattern adds object creation overhead. For performance-critical code, reuse builders or use simple factory functions instead. The pattern is most valuable for complex object construction.

### Interview Questions

1. How does the Builder pattern differ from the Factory pattern?
2. How do you implement method chaining in TypeScript builders?
3. How do you validate the built object in the Builder pattern?

### Coding Challenges

1. Create an HTML element builder that constructs HTML strings with attributes and children.
2. Build a SQL query builder with type-safe table and column references.

### Related Topics

- Factory pattern
- Fluent interfaces
- Method chaining

## Strategy Pattern

### What It Is

The Strategy pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable. The strategy object can change the algorithm used independent of the client that uses it.

### Why It Is Important

The Strategy pattern enables selecting algorithms at runtime, promoting the Open/Closed principle. TypeScript's union types and interfaces make strategies type-safe and easy to compose.

### How It Works Internally

A strategy interface defines the algorithm contract. Concrete strategy classes implement this interface. A context class uses a strategy instance and can switch strategies at runtime.

### Syntax

```typescript
interface Strategy {
  execute(data: unknown): unknown;
}

class ConcreteStrategyA implements Strategy {
  execute(data: unknown): unknown {
    return `Processed by A: ${data}`;
  }
}

class ConcreteStrategyB implements Strategy {
  execute(data: unknown): unknown {
    return `Processed by B: ${data}`;
  }
}

class Context {
  constructor(private strategy: Strategy) {}

  setStrategy(strategy: Strategy): void {
    this.strategy = strategy;
  }

  executeStrategy(data: unknown): unknown {
    return this.strategy.execute(data);
  }
}
```

### Beginner Examples

```typescript
interface PaymentStrategy {
  pay(amount: number): Promise<boolean>;
}

class CreditCardPayment implements PaymentStrategy {
  constructor(private cardNumber: string, private cvv: string) {}

  async pay(amount: number): Promise<boolean> {
    console.log(`Paid $${amount} with credit card ${this.cardNumber.slice(-4)}`);
    return true;
  }
}

class PayPalPayment implements PaymentStrategy {
  constructor(private email: string) {}

  async pay(amount: number): Promise<boolean> {
    console.log(`Paid $${amount} via PayPal (${this.email})`);
    return true;
  }
}

class CryptoPayment implements PaymentStrategy {
  async pay(amount: number): Promise<boolean> {
    console.log(`Paid $${amount} with cryptocurrency`);
    return true;
  }
}

class Checkout {
  constructor(private strategy: PaymentStrategy) {}

  async processOrder(amount: number): Promise<void> {
    const success = await this.strategy.pay(amount);
    if (success) console.log('Order completed');
  }
}
```

### Intermediate Examples

```typescript
// Validation strategies
interface ValidationStrategy<T> {
  validate(value: T): { valid: boolean; errors: string[] };
}

class EmailValidation implements ValidationStrategy<string> {
  validate(value: string) {
    const errors: string[] = [];
    if (!value.includes('@')) errors.push('Must contain @');
    if (value.length < 5) errors.push('Too short');
    return { valid: errors.length === 0, errors };
  }
}

class PasswordValidation implements ValidationStrategy<string> {
  validate(value: string) {
    const errors: string[] = [];
    if (value.length < 8) errors.push('Must be at least 8 characters');
    if (!/[A-Z]/.test(value)) errors.push('Must contain uppercase letter');
    if (!/[0-9]/.test(value)) errors.push('Must contain number');
    return { valid: errors.length === 0, errors };
  }
}

class Validator<T> {
  private strategies: ValidationStrategy<T>[] = [];

  addStrategy(strategy: ValidationStrategy<T>): void {
    this.strategies.push(strategy);
  }

  validate(value: T): string[] {
    return this.strategies.flatMap(s => s.validate(value).errors);
  }
}

// Sorting strategies
interface SortStrategy<T> {
  sort(data: T[]): T[];
}

class QuickSort<T> implements SortStrategy<T> {
  sort(data: T[]): T[] {
    if (data.length <= 1) return data;
    const pivot = data[0];
    const left = data.slice(1).filter(x => x < pivot);
    const right = data.slice(1).filter(x => x >= pivot);
    return [...this.sort(left), pivot, ...this.sort(right)];
  }
}

class BubbleSort<T> implements SortStrategy<T> {
  sort(data: T[]): T[] {
    const arr = [...data];
    for (let i = 0; i < arr.length; i++) {
      for (let j = 0; j < arr.length - i - 1; j++) {
        if (arr[j] > arr[j + 1]) {
          [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        }
      }
    }
    return arr;
  }
}
```

### Advanced Examples

```typescript
// Strategy with discriminated union
type CompressionStrategy = 'gzip' | 'deflate' | 'brotli';

interface ICompressor {
  compress(data: Buffer): Promise<Buffer>;
  decompress(data: Buffer): Promise<Buffer>;
}

class GzipCompressor implements ICompressor {
  async compress(data: Buffer): Promise<Buffer> {
    return data; // Simplified
  }
  async decompress(data: Buffer): Promise<Buffer> {
    return data;
  }
}

class DeflateCompressor implements ICompressor {
  async compress(data: Buffer): Promise<Buffer> {
    return data;
  }
  async decompress(data: Buffer): Promise<Buffer> {
    return data;
  }
}

class CompressorFactory {
  private static compressors: Record<CompressionStrategy, new () => ICompressor> = {
    gzip: GzipCompressor,
    deflate: DeflateCompressor,
    brotli: GzipCompressor, // Simplified
  };

  static create(type: CompressionStrategy): ICompressor {
    return new this.compressors[type]();
  }
}

// Strategy with configuration
interface RetryStrategy {
  shouldRetry(attempt: number, error: Error): boolean;
  getDelay(attempt: number): number;
}

class FixedRetry implements RetryStrategy {
  constructor(private maxRetries: number, private delay: number) {}

  shouldRetry(attempt: number): boolean {
    return attempt < this.maxRetries;
  }

  getDelay(attempt: number): number {
    return this.delay;
  }
}

class ExponentialBackoff implements RetryStrategy {
  constructor(
    private maxRetries: number,
    private baseDelay: number,
    private maxDelay: number = 30000
  ) {}

  shouldRetry(attempt: number): boolean {
    return attempt < this.maxRetries;
  }

  getDelay(attempt: number): number {
    return Math.min(this.baseDelay * Math.pow(2, attempt), this.maxDelay);
  }
}

class RetryableOperation {
  constructor(private strategy: RetryStrategy) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    let lastError: Error | undefined;
    for (let attempt = 0; attempt <= 10; attempt++) {
      try {
        return await fn();
      } catch (error) {
        lastError = error as Error;
        if (!this.strategy.shouldRetry(attempt, lastError)) break;
        await new Promise(resolve => setTimeout(resolve, this.strategy.getDelay(attempt)));
      }
    }
    throw lastError;
  }
}
```

### Real-World Use Cases

```typescript
// Routing strategy
interface RouteStrategy {
  calculateRoute(start: string, end: string): RouteResult;
}

interface RouteResult {
  distance: number;
  duration: number;
  steps: string[];
}

class CarRoute implements RouteStrategy {
  calculateRoute(start: string, end: string): RouteResult {
    return { distance: 100, duration: 60, steps: ['Drive on highway'] };
  }
}

class BikeRoute implements RouteStrategy {
  calculateRoute(start: string, end: string): RouteResult {
    return { distance: 30, duration: 90, steps: ['Use bike lane'] };
  }
}

class WalkingRoute implements RouteStrategy {
  calculateRoute(start: string, end: string): RouteResult {
    return { distance: 20, duration: 120, steps: ['Walk on sidewalk'] };
  }
}

class Navigator {
  private strategy: RouteStrategy;

  constructor(strategy: RouteStrategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy: RouteStrategy): void {
    this.strategy = strategy;
  }

  getRoute(start: string, end: string): RouteResult {
    return this.strategy.calculateRoute(start, end);
  }
}
```

### Common Mistakes

```typescript
// MISTAKE: Strategy interface that is too specific
// interface Strategy { execute(a: number, b: number): number }
// BETTER: interface Strategy<T, R> { execute(data: T): R }

// MISTAKE: Not allowing runtime strategy changes
// Context should allow setStrategy at any time

// MISTAKE: Putting strategy selection logic in the client
// Strategy selection can be delegated to a factory
```

### Best Practices

- Define a clean strategy interface with generic types.
- Keep strategies stateless (pass data through execute).
- Use strategy selection via factory or configuration.
- Allow runtime strategy switching.
- Document what each strategy does differently.

### Performance Considerations

Strategy pattern adds one level of indirection (virtual call). For most applications, this is negligible. For performance-critical code, consider if-else chains. Strategy objects can be cached and reused.

### Interview Questions

1. What is the Strategy pattern and when is it useful?
2. How does the Strategy pattern relate to the Open/Closed principle?
3. How do you implement strategy selection in TypeScript?

### Coding Challenges

1. Create a text formatter that uses different strategies for plain text, HTML, and Markdown.
2. Build an authentication system that supports multiple auth strategies (JWT, OAuth, API Key).

### Related Topics

- Factory pattern
- State pattern
- Open/Closed principle

## Dependency Injection

### What It Is

Dependency Injection (DI) is a design pattern where dependencies are provided to a class from the outside rather than created internally. TypeScript's decorators and metadata reflection enable sophisticated DI containers.

### Why It Is Important

DI promotes loose coupling, testability, and maintainability. TypeScript's type system ensures that injected dependencies satisfy the required contracts at compile time.

### How It Works Internally

A DI container maps abstract types (tokens) to concrete implementations. When a class requests a dependency, the container creates the appropriate instance, resolving its dependencies recursively.

### Syntax

```typescript
class Container {
  private dependencies = new Map<string, any>();

  register<T>(token: string, implementation: new (...args: any[]) => T): void {
    this.dependencies.set(token, implementation);
  }

  resolve<T>(token: string): T {
    const Implementation = this.dependencies.get(token);
    if (!Implementation) throw new Error(`Dependency ${token} not found`);
    return new Implementation();
  }
}
```

### Beginner Examples

```typescript
interface LoggerInterface {
  log(message: string): void;
}

class ConsoleLoggerDI implements LoggerInterface {
  log(message: string) { console.log(message); }
}

class UserServiceDI {
  constructor(private logger: LoggerInterface) {}

  createUser(name: string): void {
    this.logger.log(`Creating user: ${name}`);
  }
}

const container = new Container();
container.register('logger', ConsoleLoggerDI);
const logger = container.resolve<LoggerInterface>('logger');
const userService = new UserServiceDI(logger);
```

### Intermediate Examples

```typescript
// Simple DI container with constructor injection
class DIContainer {
  private instances = new Map<string, any>();
  private factories = new Map<string, () => any>();

  register<T>(token: string, factory: () => T): void {
    this.factories.set(token, factory);
  }

  registerSingleton<T>(token: string, factory: () => T): void {
    this.factories.set(token, () => {
      if (!this.instances.has(token)) {
        this.instances.set(token, factory());
      }
      return this.instances.get(token);
    });
  }

  resolve<T>(token: string): T {
    const factory = this.factories.get(token);
    if (!factory) throw new Error(`No registration for ${token}`);
    return factory() as T;
  }
}

// Usage
interface Database {
  query(sql: string): Promise<any[]>;
}

class PostgresDatabase implements Database {
  async query(sql: string): Promise<any[]> {
    console.log(`Querying: ${sql}`);
    return [];
  }
}

class UserRepository {
  constructor(private db: Database) {}

  async findAll() {
    return this.db.query('SELECT * FROM users');
  }
}

const di = new DIContainer();
di.registerSingleton('db', () => new PostgresDatabase());
di.register('userRepository', () => new UserRepository(di.resolve<Database>('db')));
```

### Advanced Examples

```typescript
// DI with decorators and metadata
import 'reflect-metadata';

const INJECTABLE_KEY = Symbol('injectable');
const INJECT_KEY = Symbol('inject');

function Injectable(): ClassDecorator {
  return (target) => {
    Reflect.defineMetadata(INJECTABLE_KEY, true, target);
  };
}

function Inject(token: string): ParameterDecorator {
  return (target, propertyKey, parameterIndex) => {
    const existingInjections = Reflect.getOwnMetadata(INJECT_KEY, target) || [];
    existingInjections.push({ parameterIndex, token });
    Reflect.defineMetadata(INJECT_KEY, existingInjections, target);
  };
}

class AdvancedDIContainer {
  private registrations = new Map<string, { cls: any; singleton: boolean; instance?: any }>();

  register(token: string, cls: any, options?: { singleton?: boolean }): void {
    this.registrations.set(token, {
      cls,
      singleton: options?.singleton ?? false,
    });
  }

  resolve<T>(token: string): T {
    const registration = this.registrations.get(token);
    if (!registration) throw new Error(`No registration for ${token}`);

    if (registration.singleton && registration.instance) {
      return registration.instance;
    }

    const injections = Reflect.getOwnMetadata(INJECT_KEY, registration.cls.prototype) || [];
    const paramTypes = Reflect.getMetadata('design:paramtypes', registration.cls) || [];

    const args = paramTypes.map((_: any, index: number) => {
      const injection = injections.find((i: any) => i.parameterIndex === index);
      const token = injection?.token || paramTypes[index]?.name;
      return this.resolve(token);
    });

    const instance = new registration.cls(...args);
    if (registration.singleton) {
      registration.instance = instance;
    }
    return instance;
  }
}

// Usage
@Injectable()
class DatabaseService {
  query(sql: string) { return `Result: ${sql}`; }
}

@Injectable()
class LoggerService {
  log(msg: string) { console.log(msg); }
}

@Injectable()
class AppService {
  constructor(
    @Inject('DatabaseService') private db: DatabaseService,
    @Inject('LoggerService') private logger: LoggerService
  ) {}

  run() {
    this.logger.log(this.db.query('SELECT 1'));
  }
}

const advancedDI = new AdvancedDIContainer();
advancedDI.register('DatabaseService', DatabaseService, { singleton: true });
advancedDI.register('LoggerService', LoggerService, { singleton: true });
advancedDI.register('AppService', AppService);

const app = advancedDI.resolve<AppService>('AppService');
app.run();
```

### Real-World Use Cases

```typescript
// Module-based DI for feature toggles
interface FeatureService {
  isEnabled(feature: string): boolean;
}

class RemoteFeatureService implements FeatureService {
  private features = new Map<string, boolean>();

  async load(): Promise<void> {
    const response = await fetch('/api/features');
    const data = await response.json();
    Object.entries(data).forEach(([key, value]) => this.features.set(key, value as boolean));
  }

  isEnabled(feature: string): boolean {
    return this.features.get(feature) ?? false;
  }
}

class LocalFeatureService implements FeatureService {
  isEnabled(feature: string): boolean {
    return localStorage.getItem(`feature:${feature}`) === 'true';
  }
}

// Depending on environment, register different implementation
if (process.env.NODE_ENV === 'production') {
  di.registerSingleton('featureService', () => new RemoteFeatureService());
} else {
  di.registerSingleton('featureService', () => new LocalFeatureService());
}
```

### Common Mistakes

```typescript
// MISTAKE: Service locator antipattern (using container as a global)
// Use constructor injection instead of resolving inside methods

// MISTAKE: Circular dependencies
// DI containers typically throw errors on circular deps

// MISTAKE: Over-engineering with DI for simple projects
// DI is valuable for medium-to-large applications
```

### Best Practices

- Use constructor injection as the primary DI method.
- Register dependencies early, resolve at the composition root.
- Use singleton scope for stateless services.
- Avoid the service locator pattern; inject dependencies explicitly.
- Use DI containers for managing object lifetimes (transient, singleton, scoped).

### Performance Considerations

DI container resolution happens once per registration at application startup. Property injection and method injection add slight overhead. Constructor injection has the least overhead and is preferred.

### Interview Questions

1. What is dependency injection and why is it useful?
2. How do you implement a simple DI container in TypeScript?
3. What is the composition root in DI?

### Coding Challenges

1. Build a DI container that supports transient, singleton, and scoped lifetimes.
2. Create a decorator-based DI system that auto-wires constructor parameters.

### Related Topics

- Decorators
- Inversion of Control
- Strategy pattern
