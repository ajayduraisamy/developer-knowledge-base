# Static Members - static properties, static methods, static blocks, generic statics

## Introduction
Static members belong to the class itself rather than to instances. They are shared across all instances and are accessed through the class name. TypeScript supports static properties, static methods, static blocks (ES2022+), and patterns for generic static members. Static members are essential for utility functions, factory methods, singleton patterns, and shared configuration.

## static properties

### What It Is
Static properties are variables declared with the `static` keyword that belong to the class itself, not to any instance. All instances share the same static property value.

### Why It Is Important
Static properties are useful for class-level constants, shared counters, default configurations, and memoized data that should be available without creating an instance.

### How It Works Internally
Static properties are attached directly to the constructor function (the class object) in JavaScript. They are not on the prototype and not copied to instances.

### Syntax
```typescript
class Example {
  static count: number = 0;
}
console.log(Example.count);
```

### Beginner Examples
```typescript
class Counter {
  static total: number = 0;
  private id: number;
  constructor() {
    Counter.total++;
    this.id = Counter.total;
  }
  getId(): number {
    return this.id;
  }
}
console.log(Counter.total); // 0
const a = new Counter();
const b = new Counter();
console.log(Counter.total); // 2
```

### Intermediate Examples
```typescript
class AppConfig {
  static readonly DEFAULT_PORT: number = 3000;
  static readonly DEFAULT_HOST: string = "localhost";
  static readonly API_VERSION: string = "v2";
  static readonly SUPPORTED_LOCALES: readonly string[] = ["en", "fr", "de", "ja"];
  private constructor() {} // prevent instantiation
}
console.log(AppConfig.DEFAULT_PORT);
```

### Advanced Examples
```typescript
class Registry {
  private static instances = new Map<string, unknown>();
  static register<T>(key: string, instance: T): void {
    if (Registry.instances.has(key)) {
      throw new Error(`Instance with key "${key}" already registered`);
    }
    Registry.instances.set(key, instance);
  }
  static resolve<T>(key: string): T {
    const instance = Registry.instances.get(key);
    if (!instance) {
      throw new Error(`No instance registered for key "${key}"`);
    }
    return instance as T;
  }
  static has(key: string): boolean {
    return Registry.instances.has(key);
  }
  static clear(): void {
    Registry.instances.clear();
  }
}
// Usage
Registry.register("logger", new ConsoleLogger());
const logger = Registry.resolve<Logger>("logger");
```

### Real-World Use Cases
- Class-level constants and enumerations
- Instance counters or ID generators
- Default configuration values
- Multiton or registry patterns

### Common Mistakes
- Accessing static properties via `this` in static methods (use `ClassName.property` or `this.constructor.property`)
- Assuming static properties are accessible from instances without the class name
- Mutating static readonly arrays or objects (use `ReadonlyArray` or `Readonly`)

### Best Practices
- Use `readonly` for static constants
- Use `private static` for internal class-level state
- Consider using `static` configuration objects instead of passing around config instances

### Performance Considerations
Static property access is slightly faster than instance property access because there's no prototype chain lookup. However, the difference is negligible in practice.

### Interview Questions
- Q: How do you access a static property from an instance method?
  A: Use `ClassName.propertyName` or `this.constructor.propertyName`.

### Coding Challenges
- Create a `UniqueID` class with a static counter and a method that returns a new unique ID string.
- Build a `Theme` class with static properties for colors, fonts, and spacing values.

### Related Topics
- static methods
- Static blocks
- Singleton pattern

---

## static methods

### What It Is
Static methods are functions declared with the `static` keyword that are called on the class itself rather than on instances.

### Why It Is Important
Static methods provide class-level functionality that does not require instance state. They are commonly used for factory methods, utility functions, and operations on static data.

### How It Works Internally
Static methods are added as own properties of the constructor function. They are not on the prototype and are not inherited by instances.

### Syntax
```typescript
class Example {
  static method(): ReturnType {
    // body
  }
}
Example.method();
```

### Beginner Examples
```typescript
class MathUtils {
  static add(a: number, b: number): number {
    return a + b;
  }
  static multiply(a: number, b: number): number {
    return a * b;
  }
  static factorial(n: number): number {
    return n <= 1 ? 1 : n * this.factorial(n - 1);
  }
}
console.log(MathUtils.factorial(5)); // 120
```

### Intermediate Examples
```typescript
class DateTimeHelper {
  static isWeekend(date: Date): boolean {
    const day = date.getDay();
    return day === 0 || day === 6;
  }
  static formatISO(date: Date): string {
    return date.toISOString().split("T")[0];
  }
  static addDays(date: Date, days: number): Date {
    const result = new Date(date);
    result.setDate(result.getDate() + days);
    return result;
  }
  static daysBetween(a: Date, b: Date): number {
    const ms = Math.abs(a.getTime() - b.getTime());
    return Math.floor(ms / (1000 * 60 * 60 * 24));
  }
}
```

### Advanced Examples
```typescript
interface Serializer<T> {
  serialize(data: T): string;
  deserialize(raw: string): T;
}
class JSONSerializer {
  static serialize<T>(data: T): string {
    return JSON.stringify(data, null, 2);
  }
  static deserialize<T>(raw: string): T {
    return JSON.parse(raw) as T;
  }
  static tryDeserialize<T>(raw: string): { success: boolean; data?: T; error?: Error } {
    try {
      const data = JSON.parse(raw);
      return { success: true, data: data as T };
    } catch (error) {
      return { success: false, error: error as Error };
    }
  }
}
```

```typescript
// Factory methods
class User {
  private constructor(
    public readonly id: string,
    public readonly name: string,
    public readonly email: string
  ) {}
  static create(name: string, email: string): User {
    return new User(crypto.randomUUID(), name, email);
  }
  static fromJSON(data: Record<string, string>): User {
    return new User(data.id, data.name, data.email);
  }
  static anonymous(): User {
    return new User("anonymous", "Guest", "guest@localhost");
  }
}
```

### Real-World Use Cases
- Factory methods that encapsulate complex construction logic
- Utility classes with pure functions (e.g., `StringUtils`, `ArrayUtils`)
- Parsers and serializers
- Builder static methods

### Common Mistakes
- Using `this` in a static method to access instance members (not possible)
- Overloading static methods in subclasses (static methods are not polymorphic via `this`)
- Creating static methods that could be standalone functions

### Best Practices
- Use static methods for operations that don't require instance state
- Combine static methods with private constructors for controlled instantiation
- Prefer standalone functions over static utility methods to improve tree-shaking

### Performance Considerations
Static methods are slightly faster to call than instance methods because there is no `this` binding or prototype lookup overhead.

### Interview Questions
- Q: Can static methods be overridden in subclasses?
  A: Shadowed (redeclared), but not truly overridden. Static methods are called on the class, not the instance.

### Coding Challenges
- Create a `StringUtils` class with static methods `capitalize`, `truncate`, `toCamelCase`, and `toSnakeCase`.
- Build a `PaginationHelper` with static methods `getPage`, `totalPages`, and `pageRange`.

### Related Topics
- static properties
- Static blocks
- Factory method pattern

---

## Static blocks (ES2022+)

### What It Is
Static blocks are ECMAScript 2022 feature that allows executing initialization code within a class's static scope. They run once when the class is first evaluated.

### Why It Is Important
Static blocks enable complex static initialization that cannot be expressed with simple field initializers — such as initializing static fields from configuration, handling try-catch, or running loops.

### How It Works Internally
Static blocks are executed immediately when the class definition is evaluated. The compiled JavaScript places the block inside an IIFE or a static initialization block on the constructor function.

### Syntax
```typescript
class Example {
  static data: Map<string, number>;
  static {
    this.data = new Map();
    this.data.set("a", 1);
  }
}
```

### Beginner Examples
```typescript
class ConfigLoader {
  static config: Record<string, string>;
  static {
    try {
      const raw = readFileSync("./config.json", "utf-8");
      ConfigLoader.config = JSON.parse(raw);
    } catch {
      ConfigLoader.config = { mode: "development" };
    }
  }
}
```

### Intermediate Examples
```typescript
class DatabaseConnectionManager {
  static pool: ConnectionPool;
  static maxConnections: number;
  static {
    const env = process.env.NODE_ENV ?? "development";
    DatabaseConnectionManager.maxConnections = env === "production" ? 100 : 5;
    DatabaseConnectionManager.pool = new ConnectionPool({
      max: DatabaseConnectionManager.maxConnections,
      idleTimeoutMillis: 30000,
    });
    console.log(`Connection pool initialized (max: ${DatabaseConnectionManager.maxConnections})`);
  }
  static async query(sql: string): Promise<unknown[]> {
    const conn = await DatabaseConnectionManager.pool.acquire();
    try {
      return await conn.query(sql);
    } finally {
      await DatabaseConnectionManager.pool.release(conn);
    }
  }
}
```

### Advanced Examples
```typescript
interface PluginDescriptor {
  name: string;
  version: string;
  factory: () => Plugin;
}
class PluginRegistry {
  static plugins: Map<string, PluginDescriptor> = new Map();
  static loaded: Set<string> = new Set();
  static {
    // Register built-in plugins
    PluginRegistry.register({
      name: "logger",
      version: "1.0.0",
      factory: () => new ConsolePlugin(),
    });
    PluginRegistry.register({
      name: "metrics",
      version: "1.0.0",
      factory: () => new MetricsPlugin(),
    });
  }
  static register(descriptor: PluginDescriptor): void {
    PluginRegistry.plugins.set(descriptor.name, descriptor);
  }
  static load(name: string): Plugin {
    const desc = PluginRegistry.plugins.get(name);
    if (!desc) throw new Error(`Plugin "${name}" not found`);
    if (!PluginRegistry.loaded.has(name)) {
      const plugin = desc.factory();
      plugin.initialize();
      PluginRegistry.loaded.add(name);
      return plugin;
    }
    throw new Error(`Plugin "${name}" already loaded`);
  }
  static {
    // Dynamic loading from environment
    const pluginsToLoad = (process.env.PLUGINS ?? "").split(",").filter(Boolean);
    for (const name of pluginsToLoad) {
      PluginRegistry.load(name.trim());
    }
  }
}
```

```typescript
class CachePolicy {
  static policies: Map<string, { ttl: number; maxSize: number }>;
  static defaultTTL: number;
  static {
    CachePolicy.policies = new Map();
    CachePolicy.policies.set("short", { ttl: 60_000, maxSize: 100 });
    CachePolicy.policies.set("medium", { ttl: 300_000, maxSize: 1000 });
    CachePolicy.policies.set("long", { ttl: 3_600_000, maxSize: 10000 });
    CachePolicy.defaultTTL = CachePolicy.policies.get("medium")!.ttl;
  }
}
```

### Real-World Use Cases
- Loading configuration from files or environment variables
- Initializing static maps or caches with complex data
- Registration patterns (plugins, formatters, validators)
- Running validation logic at class load time

### Common Mistakes
- Assuming static blocks run per-instance (they run once per class)
- Using `await` inside static blocks (static blocks are synchronous)
- Accessing instance members from static blocks (not possible)

### Best Practices
- Use static blocks for complex static initialization that cannot be expressed with field initializers
- Keep static blocks focused on a single initialization concern
- Handle errors gracefully within static blocks

### Performance Considerations
Static blocks execute once at class definition time. The cost is incurred when the module is first imported. Avoid expensive synchronous operations in static blocks.

### Interview Questions
- Q: When are static blocks executed?
  A: When the class definition is evaluated, typically when the module is first imported.

### Coding Challenges
- Create a `FeatureFlags` class that reads from environment variables in a static block.
- Build a `ValidationRegistry` that registers built-in validators in a static block.

### Related Topics
- static properties
- static methods
- Class initialization

---

## Generic static member patterns

### What It Is
Generic static members involve static properties and methods that use the class's type parameters. TypeScript has restrictions on this, but there are patterns to work around them.

### Why It Is Important
Generic static members enable type-safe factory methods, singleton registries, and polymorphic static utilities that work across different generic instantiations.

### How It Works Internally
TypeScript does not allow static members to reference class type parameters directly (static members belong to the class constructor, which is shared across all instantiations). Workarounds include using the class as a namespace or using separate generic functions.

### Syntax
```typescript
// Direct reference to class type param is NOT allowed:
// class Container<T> {
//   static default: T; // Error!
// }
```

### Beginner Examples
```typescript
// Workaround: use a separate function
class Factory<T> {
  create(value: T): { value: T } {
    return { value };
  }
  static make<T>(value: T): { value: T } {
    return { value };
  }
}
const result = Factory.make(42); // { value: 42 }
```

### Intermediate Examples
```typescript
interface Deserializable<T> {
  fromJSON(json: string): T;
}
class Model<T> {
  constructor(public data: T) {}
  static create<T>(data: T): Model<T> {
    return new Model(data);
  }
  static fromJSON<T>(json: string): Model<T> {
    return new Model(JSON.parse(json));
  }
  static arrayFromJSON<T>(json: string): Model<T>[] {
    return JSON.parse(json).map((d: T) => new Model(d));
  }
}
const user = Model.create({ name: "Alice" });
const users = Model.arrayFromJSON<{ name: string }>('[{"name":"Bob"}]');
```

### Advanced Examples
```typescript
// Registry pattern with generic types
interface Type<T> {
  new (...args: unknown[]): T;
}

class DI {
  private static registry = new Map<string, { type: Type<unknown>; singleton: boolean }>();
  private static instances = new Map<string, unknown>();

  static register<T>(token: string, type: Type<T>, singleton = false): void {
    DI.registry.set(token, { type, singleton });
    if (singleton) {
      DI.instances.set(token, new type());
    }
  }

  static resolve<T>(token: string): T {
    if (DI.instances.has(token)) {
      return DI.instances.get(token) as T;
    }
    const entry = DI.registry.get(token);
    if (!entry) throw new Error(`No registration for "${token}"`);
    const instance = new entry.type() as T;
    if (entry.singleton) {
      DI.instances.set(token, instance);
    }
    return instance;
  }

  static has(token: string): boolean {
    return DI.registry.has(token);
  }

  static clear(): void {
    DI.registry.clear();
    DI.instances.clear();
  }
}

// Usage
class LoggerImpl {
  log(msg: string) { console.log(msg); }
}
DI.register<LoggerImpl>("logger", LoggerImpl, true);
const logger = DI.resolve<LoggerImpl>("logger");
```

```typescript
// Generic factory with type-safe builders
class Builder<T> {
  private props: Partial<T> = {};
  set<K extends keyof T>(key: K, value: T[K]): this {
    this.props[key] = value;
    return this;
  }
  build(): T {
    return this.props as T;
  }
  static create<T>(): Builder<T> {
    return new Builder<T>();
  }
}

interface UserData {
  name: string;
  age: number;
  email: string;
}

const user = Builder.create<UserData>()
  .set("name", "Alice")
  .set("age", 30)
  .set("email", "alice@example.com")
  .build();
```

### Real-World Use Cases
- Generic dependency injection containers
- Type-safe factory registries
- Generic serializers/deserializers with static methods
- Builder pattern with static factory methods

### Common Mistakes
- Trying to reference class type parameters in static members directly
- Assuming static methods on generic classes are generic over the class type param
- Forgetting to add type parameters to static methods when needed

### Best Practices
- Add explicit type parameters to static methods on generic classes
- Use the class itself as a namespace for related generic functions
- Consider non-static alternatives when type parameter information is needed

### Performance Considerations
Generic static patterns have no runtime overhead — generics are erased. The patterns themselves are compile-time conveniences.

### Interview Questions
- Q: Why can't static members reference class type parameters?
  A: Static members exist on the class constructor, which is shared across all concrete type instantiations.

### Coding Challenges
- Create a generic `Serializer<T>` with a static method `serialize` and `deserialize` that infers types.
- Build a generic `RepositoryFactory` that creates type-safe repository instances from a static registry.

### Related Topics
- Generic (parametric) polymorphism
- static methods
- Factory pattern
