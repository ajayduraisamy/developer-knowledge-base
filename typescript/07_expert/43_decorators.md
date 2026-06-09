# Decorators - Class decorators, method decorators, property decorators, parameter decorators, metadata

## Introduction

Decorators are a feature of TypeScript (and an ECMAScript proposal) that allows annotating and modifying classes, methods, properties, and parameters at design time. They provide a declarative way to add behavior (like logging, validation, dependency injection, or memoization) to existing code without modifying its structure. This file covers class decorators, method decorators, property & accessor decorators, parameter decorators, and the Reflect Metadata API.

## Class Decorators

### What It Is

A class decorator is a function that receives a class constructor and can modify or replace it. It is applied to a class declaration and can be used to observe, modify, or replace the class definition.

### Why It Is Important

Class decorators enable meta-programming patterns like adding static methods, registering classes in dependency injection containers, applying mixins, and sealing/freezing class instances.

### How It Works Internally

A class decorator function receives the constructor as its only argument and returns either a modified constructor or `void`. TypeScript compiles decorators to function calls that wrap the constructor. The decorator is evaluated once when the class is defined, not when instantiated.

```typescript
type ClassDecorator = <TFunction extends Function>(target: TFunction) => TFunction | void;
```

### Syntax

```typescript
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class BugReport {
  type = 'report';
  title: string;

  constructor(t: string) {
    this.title = t;
  }
}
```

### Beginner Examples

```typescript
function logClass(constructor: Function) {
  console.log(`Class ${constructor.name} was defined`);
}

@logClass
class Person {
  constructor(public name: string) {}
}
// Logs: "Class Person was defined"

function timestamp<T extends { new (...args: any[]): {} }>(constructor: T) {
  return class extends constructor {
    createdAt = new Date();
  };
}

@timestamp
class Document {
  constructor(public content: string) {}
}

const doc = new Document('Hello');
console.log(doc.createdAt); // Date object
```

### Intermediate Examples

```typescript
// Dependency injection container decorator
const container = new Map<string, any>();

function Injectable(name?: string): ClassDecorator {
  return (target) => {
    const key = name || target.name;
    container.set(key, target);
    console.log(`Registered: ${key}`);
  };
}

@Injectable('UserService')
class UserService {
  getUsers() { return ['Alice', 'Bob']; }
}

// Singleton decorator
function Singleton<T extends { new (...args: any[]): {} }>(constructor: T) {
  let instance: InstanceType<T>;

  return class extends constructor {
    constructor(...args: any[]) {
      if (!instance) {
        super(...args);
        instance = this;
      }
      return instance;
    }
  } as T;
}

@Singleton
class AppConfig {
  apiUrl = 'https://api.example.com';
}

const config1 = new AppConfig();
const config2 = new AppConfig();
console.log(config1 === config2); // true
```

### Advanced Examples

```typescript
// Mixin decorator
function Mixin(...baseClasses: Function[]): ClassDecorator {
  return (target) => {
    baseClasses.forEach(base => {
      Object.getOwnPropertyNames(base.prototype).forEach(name => {
        if (name !== 'constructor') {
          target.prototype[name] = base.prototype[name];
        }
      });
    });
  };
}

class CanEat {
  eat() { return 'Eating'; }
}

class CanSleep {
  sleep() { return 'Sleeping'; }
}

@Mixin(CanEat, CanSleep)
class Cat {}

const cat = new Cat();
console.log((cat as any).eat()); // 'Eating'

// Decorator factory with options
interface ComponentOptions {
  selector: string;
  template?: string;
  styles?: string[];
}

function Component(options: ComponentOptions): ClassDecorator {
  return (target) => {
    Reflect.defineMetadata('component:options', options, target);
    console.log(`Component registered: ${options.selector}`);
  };
}

@Component({
  selector: 'app-root',
  template: '<h1>Hello</h1>',
})
class AppComponent {}
```

### Real-World Use Cases

```typescript
// ORM Entity decorator
function Entity(tableName: string): ClassDecorator {
  return (target) => {
    Reflect.defineMetadata('entity:table', tableName, target);
    Reflect.defineMetadata('entity:columns', [], target);
  };
}

function Column(options?: { type?: string; length?: number }): PropertyDecorator {
  return (target, propertyKey) => {
    const columns = Reflect.getMetadata('entity:columns', target.constructor) || [];
    columns.push({ name: propertyKey, ...options });
    Reflect.defineMetadata('entity:columns', columns, target.constructor);
  };
}

@Entity('users')
class User {
  @Column({ type: 'varchar', length: 100 })
  name!: string;

  @Column({ type: 'varchar', length: 255 })
  email!: string;

  @Column({ type: 'int' })
  age!: number;
}
```

### Common Mistakes

```typescript
// MISTAKE: Returning from a class decorator when not needed
// If you don't modify the class, don't return anything

// MISTAKE: Mutating the prototype instead of returning a new class
// Prefer returning a new class over mutating the constructor

// MISTAKE: Forgetting to call super() in the decorated class constructor
```

### Best Practices

- Use class decorators for cross-cutting concerns: logging, DI registration, singleton enforcement.
- Prefer composition (returning a new class) over mutation when modifying behavior.
- Use decorator factories (functions returning decorators) when configuration is needed.
- Use Reflect Metadata to store decorator information for later use.

### Performance Considerations

Class decorators are evaluated once at class definition time, not on every instantiation. The overhead is minimal. However, decorators that wrap the constructor (like Singleton) may add slight per-instantiation overhead.

### Interview Questions

1. What arguments does a class decorator receive?
2. How do you create a configurable class decorator using a factory?
3. What is the difference between mutating the constructor and returning a new class from a decorator?

### Coding Challenges

1. Create a `@LogInstantiation` decorator that logs every time a class is instantiated.
2. Build a `@Deprecated` decorator that logs a warning when a deprecated class is used.

### Related Topics

- Method decorators
- Property decorators
- Reflect metadata

## Method Decorators

### What It Is

A method decorator is a function that receives the target prototype, method name, and property descriptor. It can observe, modify, or replace a method definition. Method decorators are applied to methods of a class.

### Why It Is Important

Method decorators are the most commonly used decorator type. They enable declarative cross-cutting concerns like logging, memoization, rate limiting, retry logic, access control, and timing.

### How It Works Internally

```typescript
type MethodDecorator = <T>(
  target: Object,
  propertyKey: string | symbol,
  descriptor: TypedPropertyDescriptor<T>
) => TypedPropertyDescriptor<T> | void;
```

The decorator receives the prototype (for instance methods) or constructor (for static methods), the method name, and the property descriptor. It can replace the descriptor or modify it.

### Syntax

```typescript
function log(target: Object, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Result:`, result);
    return result;
  };

  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}
```

### Beginner Examples

```typescript
function readonly(target: Object, propertyKey: string, descriptor: PropertyDescriptor) {
  descriptor.writable = false;
  return descriptor;
}

function enumerable(value: boolean) {
  return (target: Object, propertyKey: string, descriptor: PropertyDescriptor) => {
    descriptor.enumerable = value;
    return descriptor;
  };
}

class User {
  constructor(public name: string, public email: string) {}

  @readonly
  @enumerable(false)
  getDisplayName(): string {
    return `${this.name} <${this.email}>`;
  }
}
```

### Intermediate Examples

```typescript
// Memoization decorator
function memoize(target: Object, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  const cache = new Map<string, any>();

  descriptor.value = function (...args: any[]) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = originalMethod.apply(this, args);
    cache.set(key, result);
    return result;
  };

  return descriptor;
}

class Fibonacci {
  @memoize
  calculate(n: number): number {
    if (n <= 1) return n;
    return this.calculate(n - 1) + this.calculate(n - 2);
  }
}

// Throttle decorator
function throttle(limit: number, interval: number): MethodDecorator {
  return (target, propertyKey, descriptor) => {
    const originalMethod = descriptor.value;
    const callCount = new Map<string, { count: number; startTime: number }>();

    descriptor.value = function (...args: any[]) {
      const key = JSON.stringify(args);
      const now = Date.now();
      const record = callCount.get(key) || { count: 0, startTime: now };

      if (now - record.startTime > interval) {
        record.count = 0;
        record.startTime = now;
      }

      if (record.count >= limit) {
        throw new Error(`Rate limit exceeded for ${String(propertyKey)}`);
      }

      record.count++;
      callCount.set(key, record);
      return originalMethod.apply(this, args);
    };

    return descriptor;
  };
}
```

### Advanced Examples

```typescript
// Access control decorator
function requireRole(...roles: string[]): MethodDecorator {
  return (target, propertyKey, descriptor) => {
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      const user = (this as any).currentUser;
      if (!user || !roles.some((role) => user.roles?.includes(role))) {
        throw new Error(`Access denied. Required roles: ${roles.join(', ')}`);
      }
      return originalMethod.apply(this, args);
    };

    return descriptor;
  };
}

class AdminController {
  currentUser?: { roles: string[] };

  @requireRole('admin')
  deleteUser(userId: string): void {
    console.log(`User ${userId} deleted`);
  }
}

// Retry decorator
function retry(maxAttempts: number, delayMs: number = 0): MethodDecorator {
  return (target, propertyKey, descriptor) => {
    const originalMethod = descriptor.value as Function;

    descriptor.value = async function (...args: any[]) {
      let lastError: Error | undefined;

      for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
          return await originalMethod.apply(this, args);
        } catch (error) {
          lastError = error as Error;
          console.warn(`Attempt ${attempt}/${maxAttempts} failed: ${error}`);
          if (attempt < maxAttempts && delayMs > 0) {
            await new Promise((resolve) => setTimeout(resolve, delayMs));
          }
        }
      }

      throw lastError;
    };

    return descriptor;
  };
}

class ApiClient {
  @retry(3, 1000)
  async fetchData(url: string): Promise<unknown> {
    const response = await fetch(url);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  }
}
```

### Real-World Use Cases

```typescript
// NestJS-style method decorators
function Get(path: string): MethodDecorator {
  return (target, propertyKey, descriptor) => {
    Reflect.defineMetadata('route:method', 'GET', descriptor.value);
    Reflect.defineMetadata('route:path', path, descriptor.value);
  };
}

function UseGuards(...guards: any[]): MethodDecorator {
  return (target, propertyKey, descriptor) => {
    const existingGuards = Reflect.getMetadata('guards', descriptor.value) || [];
    Reflect.defineMetadata('guards', [...existingGuards, ...guards], descriptor.value);
  };
}

class UserController {
  @Get('/users')
  @UseGuards(AuthGuard)
  getUsers() {
    return ['Alice', 'Bob'];
  }
}
```

### Common Mistakes

```typescript
// MISTAKE: Using arrow functions that capture `this` incorrectly
// descriptor.value = (...args) => originalMethod.apply(this, args);
// CORRECT: Use function expression to preserve dynamic this

// MISTAKE: Not handling the return value properly
// If you modify descriptor.value, you must return the descriptor

// MISTAKE: Applying instance method decorators to static methods
// The target for static methods is the constructor, not the prototype
```

### Best Practices

- Return the modified descriptor from method decorators that change behavior.
- Preserve the original method's signature and return type.
- Use decorator factories for configurable behavior.
- Use `Reflect.metadata` to store decorator metadata for later inspection.
- Keep method decorators focused on a single concern.

### Performance Considerations

Method decorators wrap methods, adding a layer of indirection. For frequently called methods, this adds overhead proportional to the decorator's complexity. Memoization decorators, while adding overhead on first call, save time on subsequent calls with the same arguments.

### Interview Questions

1. What are the three arguments a method decorator receives?
2. How do you modify a method's behavior using a decorator?
3. What is the difference between a `MethodDecorator` and a decorator factory?

### Coding Challenges

1. Create a `@debounce` decorator that delays method execution.
2. Build a `@measure` decorator that logs the execution time of a method.

### Related Topics

- Property decorators
- Parameter decorators
- Reflect metadata

## Property and Accessor Decorators

### What It Is

Property decorators are applied to class properties. Accessor decorators are applied to getter/setter accessors. Property decorators receive the target prototype and property name, but unlike method decorators, they do not receive a property descriptor (in the current proposal).

### Why It Is Important

Property decorators enable patterns like validation, serialization, default values, and dependency injection for class properties. They are widely used in ORMs, form frameworks, and serialization libraries.

### How It Works Internally

```typescript
type PropertyDecorator = (target: Object, propertyKey: string | symbol) => void;
type AccessorDecorator = <T>(
  target: Object,
  propertyKey: string | symbol,
  descriptor: TypedPropertyDescriptor<T>
) => TypedPropertyDescriptor<T> | void;
```

Property decorators do not receive the descriptor, so they cannot modify property behavior directly. Instead, they typically store metadata using `Reflect.defineMetadata`.

### Syntax

```typescript
function format(formatString: string): PropertyDecorator {
  return (target, propertyKey) => {
    Reflect.defineMetadata('format', formatString, target, propertyKey);
  };
}

class User {
  @format('yyyy-MM-dd')
  birthDate!: Date;
}

function capitalize(target: Object, propertyKey: string): void {
  let value: string;

  Object.defineProperty(target, propertyKey, {
    get: () => value,
    set: (newValue: string) => {
      value = newValue.charAt(0).toUpperCase() + newValue.slice(1);
    },
  });
}

class Person {
  @capitalize
  name!: string;
}

const p = new Person();
p.name = 'alice';
console.log(p.name); // 'Alice'
```

### Beginner Examples

```typescript
// Default value decorator
function Default(value: any): PropertyDecorator {
  return (target, propertyKey) => {
    const metadataKey = `default:${String(propertyKey)}`;
    Reflect.defineMetadata(metadataKey, value, target, propertyKey);

    Object.defineProperty(target, propertyKey, {
      get: () => {
        const stored = Reflect.getMetadata(metadataKey, target, propertyKey);
        return stored !== undefined ? stored : value;
      },
      set: (newValue) => {
        Reflect.defineMetadata(metadataKey, newValue, target, propertyKey);
      },
      enumerable: true,
      configurable: true,
    });
  };
}

class Config {
  @Default(3000)
  port!: number;

  @Default('localhost')
  host!: string;
}

// Non-enumerable decorator
function NonEnumerable(target: Object, propertyKey: string | symbol) {
  Object.defineProperty(target, propertyKey, {
    enumerable: false,
  });
}
```

### Intermediate Examples

```typescript
// Validation decorators
function Required(target: Object, propertyKey: string | symbol) {
  const requiredProps = Reflect.getMetadata('validation:required', target) || [];
  requiredProps.push(propertyKey);
  Reflect.defineMetadata('validation:required', requiredProps, target);
}

function validate(instance: any): boolean {
  const target = Object.getPrototypeOf(instance);
  const requiredProps = Reflect.getMetadata('validation:required', target) || [];

  for (const prop of requiredProps) {
    if (instance[prop] === null || instance[prop] === undefined || instance[prop] === '') {
      throw new Error(`${String(prop)} is required`);
    }
  }
  return true;
}

class Product {
  @Required
  name!: string;

  @Required
  price!: number;
}

const product = new Product();
// validate(product); // Throws: name is required

// Serialization decorator
function Expose(name?: string): PropertyDecorator {
  return (target, propertyKey) => {
    const metadataKey = 'serialization:exposed';
    const exposed = Reflect.getMetadata(metadataKey, target) || {};
    exposed[propertyKey] = name || propertyKey;
    Reflect.defineMetadata(metadataKey, exposed, target);
  };
}

function Exclude(target: Object, propertyKey: string | symbol) {
  const metadataKey = 'serialization:excluded';
  const excluded = Reflect.getMetadata(metadataKey, target) || [];
  excluded.push(propertyKey);
  Reflect.defineMetadata(metadataKey, excluded, target);
}

class Employee {
  @Expose()
  id!: number;

  @Expose('full_name')
  name!: string;

  @Exclude
  salary!: number;
}
```

### Advanced Examples

```typescript
// Reactive property decorator (like Vue's reactive)
function Reactive(target: Object, propertyKey: string | symbol) {
  const internalKey = Symbol(`__${String(propertyKey)}`);
  const subscriptions = new Map<string | symbol, Set<Function>>();

  Object.defineProperty(target, propertyKey, {
    get() {
      return (this as any)[internalKey];
    },
    set(newValue) {
      const oldValue = (this as any)[internalKey];
      (this as any)[internalKey] = newValue;
      const subs = subscriptions.get(propertyKey);
      if (subs) {
        subs.forEach((cb) => cb(newValue, oldValue));
      }
    },
    enumerable: true,
    configurable: true,
  });
}

function Watch(property: string): MethodDecorator {
  return (target, propertyKey, descriptor) => {
    const watches = Reflect.getMetadata('reactive:watchers', target) || [];
    watches.push({ property, method: propertyKey });
    Reflect.defineMetadata('reactive:watchers', watches, target);
  };
}

class ViewModel {
  @Reactive
  name = '';

  @Reactive
  age = 0;

  @Watch('name')
  onNameChanged(newValue: string, oldValue: string) {
    console.log(`Name changed from ${oldValue} to ${newValue}`);
  }
}
```

### Real-World Use Cases

```typescript
// TypeORM-style column decorator
function Column(type?: string): PropertyDecorator {
  return (target, propertyKey) => {
    const columns = Reflect.getMetadata('orm:columns', target.constructor) || [];
    columns.push({ name: propertyKey, type: type || 'varchar' });
    Reflect.defineMetadata('orm:columns', columns, target.constructor);
  };
}

function PrimaryGeneratedColumn(): PropertyDecorator {
  return (target, propertyKey) => {
    const columns = Reflect.getMetadata('orm:columns', target.constructor) || [];
    columns.push({ name: propertyKey, type: 'int', primary: true, generated: true });
    Reflect.defineMetadata('orm:columns', columns, target.constructor);
  };
}

@Entity('posts')
class Post {
  @PrimaryGeneratedColumn()
  id!: number;

  @Column('varchar')
  title!: string;

  @Column('text')
  content!: string;

  @Column('timestamp')
  createdAt!: Date;
}
```

### Common Mistakes

```typescript
// MISTAKE: Assuming property decorators receive a descriptor
// Property decorators only get target and propertyKey

// MISTAKE: Trying to modify the property descriptor in a property decorator
// Use accessor decorators for setter/getter modifications

// MISTAKE: Not using Reflect.defineMetadata correctly
// Always pass the target prototype as the receiver
```

### Best Practices

- Use property decorators for metadata and side effects, not for modifying behavior.
- Use accessor decorators when you need to control getter/setter behavior.
- Use `Reflect.defineMetadata` to associate data with decorated properties.
- Combine property decorators with class decorators that process the metadata.
- Validate properties at construction or serialization time, not in the decorator itself.

### Performance Considerations

Property decorators (without descriptor modification) have negligible overhead. Accessor decorators that modify getters/setters add indirection on every property access. Use them judiciously for frequently accessed properties.

### Interview Questions

1. What is the difference between a property decorator and an accessor decorator?
2. Why don't property decorators receive a property descriptor?
3. How do you store metadata for a decorated property?

### Coding Challenges

1. Create a `@MinLength` and `@MaxLength` validation decorator for string properties.
2. Build a `@Transient` decorator that excludes properties from JSON serialization.

### Related Topics

- Property descriptors
- Reflect metadata
- Validation decorators

## Parameter Decorators

### What It Is

A parameter decorator is applied to a method parameter. It receives the target prototype, method name, and parameter index. Parameter decorators are typically used in conjunction with other decorators to provide metadata about method parameters.

### Why It Is Important

Parameter decorators enable dependency injection frameworks (like Angular and NestJS) to resolve constructor parameters, and they enable parameter validation, transformation, and documentation generation.

### How It Works Internally

```typescript
type ParameterDecorator = (target: Object, propertyKey: string | symbol, parameterIndex: number) => void;
```

Parameter decorators cannot modify the parameter directly; they only store metadata. This metadata is typically consumed by a method or class decorator.

### Syntax

```typescript
import 'reflect-metadata';

function Inject(token: any): ParameterDecorator {
  return (target, propertyKey, parameterIndex) => {
    const existingParams = Reflect.getMetadata('design:paramtypes', target, propertyKey) || [];
    const metadataKey = `inject:${String(propertyKey)}`;
    const params = Reflect.getMetadata(metadataKey, target) || {};
    params[parameterIndex] = token;
    Reflect.defineMetadata(metadataKey, params, target);
  };
}

class UserService {
  constructor(@Inject('Database') private db: any) {}
}
```

### Beginner Examples

```typescript
function LogParameter(target: Object, propertyKey: string | symbol, parameterIndex: number) {
  const metadataKey = `log:${String(propertyKey)}`;
  const indices = Reflect.getMetadata(metadataKey, target) || [];
  indices.push(parameterIndex);
  Reflect.defineMetadata(metadataKey, indices, target);
}

class Calculator2 {
  add(@LogParameter a: number, @LogParameter b: number): number {
    return a + b;
  }
}
```

### Intermediate Examples

```typescript
// Validation parameter decorator
function Validate(target: Object, propertyKey: string | symbol, parameterIndex: number) {
  const metadataKey = `validate:${String(propertyKey)}`;
  const validations = Reflect.getMetadata(metadataKey, target) || [];
  validations.push(parameterIndex);
  Reflect.defineMetadata(metadataKey, validations, target);
}

function validateParameters(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  const metadataKey = `validate:${propertyKey}`;
  const validationIndices = Reflect.getMetadata(metadataKey, target) || [];

  descriptor.value = function (...args: any[]) {
    for (const index of validationIndices) {
      if (args[index] === null || args[index] === undefined) {
        throw new Error(`Parameter at index ${index} is required`);
      }
    }
    return originalMethod.apply(this, args);
  };
}

class Service {
  @validateParameters
  process(@Validate data: string, @Validate id: number): void {
    console.log(`Processing ${data} with id ${id}`);
  }
}
```

### Advanced Examples

```typescript
// Dependency injection with parameter decorators
type Token = string | symbol | { new (...args: any[]): any };

const injector = new Map<Token, any>();

function register(token: Token, implementation: any) {
  injector.set(token, implementation);
}

function Inject(token: Token): ParameterDecorator {
  return (target, propertyKey, parameterIndex) => {
    const metadataKey = `di:${String(propertyKey)}`;
    const params = Reflect.getMetadata(metadataKey, target) || {};
    params[parameterIndex] = token;
    Reflect.defineMetadata(metadataKey, params, target);
  };
}

function Injectable(): ClassDecorator {
  return (target) => {
    const paramTypes = Reflect.getMetadata('design:paramtypes', target) || [];
    const metadataKey = `di:constructor`;
    const customTokens = Reflect.getMetadata(metadataKey, target) || {};

    register(target.name, (...args: any[]) => {
      const resolvedParams = paramTypes.map((type: any, index: number) => {
        const token = customTokens[index] || type;
        return injector.get(token);
      });
      return new target(...resolvedParams);
    });
  };
}

class Database {
  query(sql: string) { return `Result of: ${sql}`; }
}

class Logger {
  log(msg: string) { console.log(msg); }
}

@Injectable()
class UserService2 {
  constructor(
    @Inject(Database) private db: Database,
    @Inject(Logger) private logger: Logger
  ) {}

  getUsers() {
    this.logger.log('Fetching users');
    return this.db.query('SELECT * FROM users');
  }
}
```

### Real-World Use Cases

```typescript
// Express route parameter injection
function Param(paramName: string): ParameterDecorator {
  return (target, propertyKey, parameterIndex) => {
    const metadataKey = `route:params:${String(propertyKey)}`;
    const params = Reflect.getMetadata(metadataKey, target) || {};
    params[parameterIndex] = paramName;
    Reflect.defineMetadata(metadataKey, params, target);
  };
}

class UserController2 {
  getUser(@Param('id') id: string) {
    return `User ${id}`;
  }
}
```

### Common Mistakes

```typescript
// MISTAKE: Expecting parameter decorators to modify parameter behavior
// They only provide metadata; other decorators must consume that metadata

// MISTAKE: Forgetting to install reflect-metadata
// Parameter decorators often rely on Reflect.defineMetadata to store metadata
```

### Best Practices

- Use parameter decorators to annotate parameters with metadata.
- Combine parameter decorators with method/class decorators that consume the metadata.
- Use `reflect-metadata` polyfill for `Reflect.defineMetadata`.
- Keep parameter decorators focused on a single concern (injection, validation, etc.).

### Performance Considerations

Parameter decorators execute once at class definition time. They add no per-call overhead. The `reflect-metadata` polyfill adds minimal overhead for metadata operations.

### Interview Questions

1. What arguments does a parameter decorator receive?
2. Why can't parameter decorators modify the parameter directly?
3. How do parameter decorators work with dependency injection?

### Coding Challenges

1. Create a `@Required` parameter decorator that throws if the parameter is undefined.
2. Build a parameter decorator system for automatic type conversion of method parameters.

### Related Topics

- Method decorators
- Reflect metadata
- Dependency injection

## Reflect Metadata

### What It Is

Reflect Metadata is a proposal for adding metadata to objects through decorators. It provides a way to attach and retrieve metadata at runtime using `Reflect.defineMetadata`, `Reflect.getMetadata`, and related functions. The `reflect-metadata` polyfill enables this in TypeScript.

### Why It Is Important

Reflect Metadata is the foundation for advanced decorator patterns, including dependency injection, validation, serialization, and ORM mapping. It allows decorators to store information that can be later read by other decorators, frameworks, or runtime code.

### How It Works Internally

Metadata is stored as a WeakMap of targets to metadata maps. Each metadata map is keyed by the metadata key and optionally scoped to a property key. The `design:paramtypes` and `design:type` metadata keys are automatically emitted by TypeScript when `emitDecoratorMetadata` is enabled.

```typescript
// TypeScript emits these automatically:
// design:type - the type of a property
// design:paramtypes - parameter types of a method
// design:returntype - return type of a method
```

### Syntax

```typescript
import 'reflect-metadata';

const metadataKey = Symbol('myMetadata');

// Define metadata
Reflect.defineMetadata(metadataKey, { version: 1 }, target);
Reflect.defineMetadata(metadataKey, { format: 'json' }, target, 'methodName');

// Get metadata
const meta = Reflect.getMetadata(metadataKey, target);
const methodMeta = Reflect.getMetadata(metadataKey, target, 'methodName');

// Check if metadata exists
const hasMeta = Reflect.hasMetadata(metadataKey, target);
```

### Beginner Examples

```typescript
import 'reflect-metadata';

function Mark(marker: string): PropertyDecorator {
  return (target, propertyKey) => {
    Reflect.defineMetadata('marker', marker, target, propertyKey);
  };
}

class Document2 {
  @Mark('important')
  title!: string;

  @Mark('optional')
  description?: string;
}

const doc = new Document2();
const titleMarker = Reflect.getMetadata('marker', Object.getPrototypeOf(doc), 'title');
console.log(titleMarker); // 'important'
```

### Intermediate Examples

```typescript
// Using design-time metadata
function logTypes(target: Object, propertyKey: string, descriptor: PropertyDescriptor) {
  const paramTypes = Reflect.getMetadata('design:paramtypes', target, propertyKey);
  const returnType = Reflect.getMetadata('design:returntype', target, propertyKey);

  console.log(`Method: ${String(propertyKey)}`);
  console.log(`Parameter types: ${paramTypes?.map((t: any) => t.name).join(', ')}`);
  console.log(`Return type: ${returnType?.name}`);

  const originalMethod = descriptor.value;
  descriptor.value = function (...args: any[]) {
    args.forEach((arg, index) => {
      const expectedType = paramTypes?.[index];
      if (expectedType && arg !== null && arg !== undefined) {
        if (arg.constructor !== expectedType) {
          console.warn(`Parameter ${index} expected ${expectedType.name}, got ${arg.constructor.name}`);
        }
      }
    });
    return originalMethod.apply(this, args);
  };
}

class MathService {
  @logTypes
  add(a: number, b: number): number {
    return a + b;
  }
}
```

### Advanced Examples

```typescript
// Metadata-based serialization
import 'reflect-metadata';

const SERIALIZE_KEY = Symbol('serialize');

function Serialize(name?: string): PropertyDecorator {
  return (target, propertyKey) => {
    const existing = Reflect.getMetadata(SERIALIZE_KEY, target) || [];
    existing.push({ key: propertyKey, name: name || String(propertyKey) });
    Reflect.defineMetadata(SERIALIZE_KEY, existing, target);
  };
}

function serialize(instance: any): Record<string, unknown> {
  const target = Object.getPrototypeOf(instance);
  const properties = Reflect.getMetadata(SERIALIZE_KEY, target) || [];
  const result: Record<string, unknown> = {};

  properties.forEach(({ key, name }: { key: string | symbol; name: string }) => {
    result[name] = (instance as any)[key];
  });

  return result;
}

function deserialize<T>(cls: new () => T, data: Record<string, unknown>): T {
  const instance = new cls();
  const target = Object.getPrototypeOf(instance);
  const properties = Reflect.getMetadata(SERIALIZE_KEY, target) || [];

  properties.forEach(({ key, name }: { key: string | symbol; name: string }) => {
    if (data[name] !== undefined) {
      (instance as any)[key] = data[name];
    }
  });

  return instance;
}

class User3 {
  @Serialize('user_id')
  id!: number;

  @Serialize('full_name')
  name!: string;

  @Serialize()
  email!: string;
}

const user = new User3();
user.id = 1;
user.name = 'Alice';
user.email = 'alice@test.com';

const serialized = serialize(user);
console.log(serialized); // { user_id: 1, full_name: 'Alice', email: 'alice@test.com' }

const deserialized = deserialize(User3, serialized);
console.log(deserialized instanceof User3); // true
```

### Real-World Use Cases

```typescript
// Framework metadata system
const ROUTE_KEY = Symbol('route:definitions');

function Controller(basePath: string): ClassDecorator {
  return (target) => {
    Reflect.defineMetadata(ROUTE_KEY, { basePath, routes: [] }, target);
  };
}

function Get(path: string): MethodDecorator {
  return (target, propertyKey, descriptor) => {
    const routes = Reflect.getMetadata(ROUTE_KEY, target.constructor) || { basePath: '', routes: [] };
    routes.routes.push({ method: 'GET', path, handler: propertyKey });
    Reflect.defineMetadata(ROUTE_KEY, routes, target.constructor);
  };
}

function registerRoutes(controller: any) {
  const metadata = Reflect.getMetadata(ROUTE_KEY, controller);
  if (!metadata) return;

  const instance = new controller();
  metadata.routes.forEach((route: any) => {
    const fullPath = metadata.basePath + route.path;
    console.log(`Registered: ${route.method} ${fullPath} -> ${String(route.handler)}`);
  });
}

@Controller('/api/users')
class UserController3 {
  @Get('/')
  getUsers() { return ['Alice', 'Bob']; }

  @Get('/:id')
  getUserById() { return { id: 1, name: 'Alice' }; }
}

registerRoutes(UserController3);
```

### Common Mistakes

```typescript
// MISTAKE: Forgetting to import 'reflect-metadata' once at the app entry
// Without it, Reflect.defineMetadata and friends are undefined

// MISTAKE: Not enabling emitDecoratorMetadata in tsconfig
// Without this, design-time types are not emitted

// MISTAKE: Using wrong target object for metadata
// For instance members, use prototype; for static members, use constructor
```

### Best Practices

- Import `reflect-metadata` once at your application's entry point.
- Enable `emitDecoratorMetadata` and `experimentalDecorators` in tsconfig.
- Use `Symbol` keys for metadata to avoid naming collisions.
- Use prototype (`target`) for instance member metadata.
- Use constructor for class-level and static member metadata.
- Combine `defineMetadata` in decorators with `getMetadata` in framework code.

### Performance Considerations

Reflect Metadata operations are relatively fast, using WeakMap lookups. However, storing large amounts of metadata can increase memory usage. Design-time metadata is computed once at class definition.

### Interview Questions

1. What is Reflect Metadata and why is it used with decorators?
2. What metadata does TypeScript emit automatically when `emitDecoratorMetadata` is enabled?
3. How do you define and retrieve custom metadata for a class property?

### Coding Challenges

1. Create a metadata-driven validation system where property decorators add validation rules and a `validate` function reads and applies them.
2. Build a serialization framework using Reflect Metadata that can handle nested objects.

### Related Topics

- Class decorators
- Method decorators
- Property decorators
- Parameter decorators
