# Encapsulation - Public/private/protected, ECMAScript private fields (#), readonly, visibility

## Introduction
Encapsulation is the object-oriented principle of bundling data with the methods that operate on that data while restricting direct access to internal state. TypeScript provides multiple mechanisms for encapsulation: access modifiers (`public`, `private`, `protected`), ECMAScript private fields (`#`), and the `readonly` modifier. These tools enable developers to enforce boundaries, protect invariants, and expose only well-defined interfaces.

## public default

### What It Is
In TypeScript, all class members are `public` by default. Public members are accessible from anywhere — within the class, in derived classes, and from external code.

### Why It Is Important
The public interface defines the contract that external consumers interact with. Explicitly marking members as public documents the intended API surface.

### How It Works Internally
`public` is the default access; omitting it produces identical emitted JavaScript. The compiler does not enforce restrictions beyond what is visible.

### Syntax
```typescript
class Example {
  public name: string; // explicit
  age: number;         // implicit public
  public greet(): void { console.log("Hello"); }
}
```

### Beginner Examples
```typescript
class Person {
  public name: string;
  public constructor(name: string) {
    this.name = name;
  }
  public sayHello(): string {
    return `Hi, I'm ${this.name}`;
  }
}
const p = new Person("Alice");
console.log(p.name);      // OK
console.log(p.sayHello()); // OK
```

### Intermediate Examples
```typescript
class ApiService {
  public baseURL: string;
  public timeout: number;
  constructor(baseURL: string, timeout = 5000) {
    this.baseURL = baseURL;
    this.timeout = timeout;
  }
  public async fetch<T>(path: string): Promise<T> {
    const res = await fetch(`${this.baseURL}${path}`, {
      signal: AbortSignal.timeout(this.timeout),
    });
    return res.json();
  }
}
```

### Advanced Examples
```typescript
class Observable<T> {
  public subscribers = new Set<(value: T) => void>();
  public get value(): T {
    return this._value;
  }
  public set value(v: T) {
    this._value = v;
    this.subscribers.forEach(fn => fn(v));
  }
  private _value: T;
  constructor(initial: T) {
    this._value = initial;
  }
  public subscribe(fn: (value: T) => void): () => void {
    this.subscribers.add(fn);
    return () => this.subscribers.delete(fn);
  }
}
```

### Real-World Use Cases
- Public API methods on service classes
- Component input/output properties in frameworks
- Configuration properties on utility classes

### Common Mistakes
- Marking everything `public` by default without considering encapsulation
- Assuming `public` members are the only way to expose functionality
- Exposing mutable internal state as `public` arrays or objects

### Best Practices
- Prefer `private` by default; make members `public` only when necessary
- Use getters for controlled access to internal state
- Document the public API with explicit `public` annotations

### Performance Considerations
No runtime impact. Access modifiers are strictly compile-time constructs.

### Interview Questions
- Q: If you omit the access modifier, what is the default?
  A: `public` is the default access modifier in TypeScript.

### Coding Challenges
- Create a `BankAccount` class with public `deposit` and `withdraw` methods but keep the balance encapsulated.
- Design a `Timer` class with a public `start`, `stop`, and `reset` interface.

### Related Topics
- private (TS)
- protected
- ECMAScript private fields (#)

---

## private (TS)

### What It Is
TypeScript's `private` modifier marks a member as accessible only within the class that declares it. It is a compile-time construct that does not prevent access at runtime.

### Why It Is Important
`private` enforces encapsulation by preventing external code and derived classes from accessing internal implementation details, reducing coupling and protecting invariants.

### How It Works Internally
TypeScript's `private` is a compile-time check only. The emitted JavaScript retains the property as a normal property. At runtime, the property is accessible. TypeScript also supports `private` with `--strictPropertyInitialization`.

### Syntax
```typescript
class Example {
  private secret: string;
  constructor(s: string) {
    this.secret = s;
  }
  private reveal(): string {
    return this.secret;
  }
}
```

### Beginner Examples
```typescript
class Wallet {
  private balance: number;
  constructor(initial: number) {
    this.balance = initial;
  }
  public deposit(amount: number): void {
    this.balance += amount;
  }
  public getBalance(): number {
    return this.balance;
  }
}
const w = new Wallet(100);
// w.balance; // Error: private
console.log(w.getBalance()); // 100
```

### Intermediate Examples
```typescript
class Cache {
  private store = new Map<string, { value: unknown; expiresAt: number }>();
  private defaultTTL = 60_000;
  public set(key: string, value: unknown, ttl?: number): void {
    this.store.set(key, {
      value,
      expiresAt: Date.now() + (ttl ?? this.defaultTTL),
    });
  }
  public get<T>(key: string): T | undefined {
    const entry = this.store.get(key);
    if (!entry) return undefined;
    if (Date.now() > entry.expiresAt) {
      this.store.delete(key);
      return undefined;
    }
    return entry.value as T;
  }
  public clear(): void {
    this.store.clear();
  }
}
```

### Advanced Examples
```typescript
class EncryptedStore {
  private static encryptionKey: Buffer;
  private data = new Map<string, Buffer>();
  private static deriveKey(password: string): Buffer {
    return crypto.createHash("sha256").update(password).digest();
  }
  public static initialize(password: string): void {
    this.encryptionKey = this.deriveKey(password);
  }
  public save(id: string, plaintext: string): void {
    const cipher = crypto.createCipheriv("aes-256-gcm", EncryptedStore.encryptionKey, null!);
    this.data.set(id, Buffer.concat([cipher.update(plaintext), cipher.final()]));
  }
}
```

### Real-World Use Cases
- Hiding database implementation details in repositories
- Encapsulating internal counters, caches, and state machines
- Protecting internal API keys and configuration

### Common Mistakes
- Assuming TypeScript `private` provides runtime security
- Using `private` when `protected` is needed for subclasses
- Forgetting that properties are still enumerable or accessible via `obj['prop']`

### Best Practices
- Use `private` for internal implementation details
- Consider using ECMAScript `#` private fields for true runtime privacy
- Combine `private` with `readonly` for immutable internal state

### Performance Considerations
No runtime cost. The property exists on the instance normally at runtime.

### Interview Questions
- Q: How can you access a `private` property at runtime in JavaScript?
  A: Using bracket notation (`obj['privateProp']`) or by accessing the property on the instance directly.

### Coding Challenges
- Create a `Counter` class where `count` is private and only `increment()`, `decrement()`, and `getValue()` are public.
- Build a `SessionManager` with private token storage and public `login()`, `logout()`, and `isAuthenticated()` methods.

### Related Topics
- public default
- protected
- ECMAScript private fields (#)

---

## protected

### What It Is
The `protected` modifier allows access to a member within the declaring class and its subclasses, but not from external code.

### Why It Is Important
`protected` enables base classes to expose helper methods and state to derived classes while hiding them from the public API.

### How It Works Internally
Like `private`, `protected` is a compile-time construct. The emitted JavaScript has no protection. However, the compiler enforces access restrictions based on the modifier.

### Syntax
```typescript
class Base {
  protected helper(): void { /* ... */ }
}
class Derived extends Base {
  method(): void {
    this.helper(); // OK
  }
}
```

### Beginner Examples
```typescript
class Animal {
  protected name: string;
  constructor(name: string) {
    this.name = name;
  }
}
class Dog extends Animal {
  public speak(): string {
    return `${this.name} says Woof!`; // OK: protected
  }
}
const d = new Dog("Rex");
// d.name; // Error: protected
```

### Intermediate Examples
```typescript
abstract class BaseService {
  protected httpClient: HttpClient;
  protected baseURL: string;
  constructor(baseURL: string) {
    this.baseURL = baseURL;
    this.httpClient = new HttpClient();
  }
  protected async request<T>(method: string, path: string): Promise<T> {
    const res = await this.httpClient.fetch(method, `${this.baseURL}${path}`);
    return res.json();
  }
}
class UserService extends BaseService {
  constructor() { super("https://api.example.com"); }
  async getById(id: string): Promise<User> {
    return this.request<User>("GET", `/users/${id}`);
  }
}
```

### Advanced Examples
```typescript
class AbstractUIComponent {
  protected element: HTMLElement;
  constructor(tag: string) {
    this.element = document.createElement(tag);
  }
  protected setStyles(styles: Partial<CSSStyleDeclaration>): void {
    Object.assign(this.element.style, styles);
  }
  public mount(target: HTMLElement): void {
    target.appendChild(this.element);
    this.onMounted();
  }
  protected onMounted(): void {
    // hook for subclasses
  }
}
class Button extends AbstractUIComponent {
  constructor(label: string) {
    super("button");
    this.element.textContent = label;
  }
  protected onMounted(): void {
    this.setStyles({ backgroundColor: "blue", color: "white" });
  }
}
```

### Real-World Use Cases
- Template method pattern with protected steps
- Framework base classes with protected lifecycle hooks
- Shared utility methods in class hierarchies

### Common Mistakes
- Marking everything `protected` — prefer `private` when subclasses don't need access
- Accessing protected members from sibling classes that don't share the base
- Relying on `protected` for runtime safety

### Best Practices
- Use `protected` only when derived classes genuinely need access
- Document the protected API contract for subclass authors
- Consider making protected properties `readonly` to prevent accidental mutation

### Performance Considerations
No runtime cost. Compile-time only.

### Interview Questions
- Q: Can a derived class access a protected member from a different instance of the base class?
  A: Yes, as long as the access occurs within the derived class.

### Coding Challenges
- Create a `Vehicle` base class with a protected `fuelLevel` and a `public refuel()`. Extend with `Car` and `Truck`.
- Build an abstract `Report` with protected `formatHeader()` and `formatFooter()`, then implement `PDFReport` and `CSVReport`.

### Related Topics
- private (TS)
- public default
- Inheritance

---

## ECMAScript private fields (#)

### What It Is
ECMAScript private fields use the `#` prefix to create truly private class fields, enforced by the JavaScript runtime. They are a native JavaScript feature fully supported in TypeScript since version 3.8.

### Why It Is Important
Unlike TypeScript's `private` modifier (compile-time only), `#` private fields provide true hard privacy at runtime — inaccessible even with reflection or bracket notation.

### How It Works Internally
`#` fields use JavaScript's native private field semantics via `WeakMap`-based storage downlevel or native private fields in modern engines. They are not accessible via `Object.keys()`, `for...in`, or `obj['#field']`.

### Syntax
```typescript
class Example {
  #secret: string;
  constructor(s: string) {
    this.#secret = s;
  }
}
```

### Beginner Examples
```typescript
class Counter {
  #count = 0;
  increment(): number {
    this.#count += 1;
    return this.#count;
  }
  get count(): number {
    return this.#count;
  }
}
const c = new Counter();
// c.#count; // SyntaxError: private field
console.log(c.increment()); // 1
```

### Intermediate Examples
```typescript
class APIHandler {
  #apiKey: string;
  #baseURL: string;
  constructor(apiKey: string, baseURL: string) {
    this.#apiKey = apiKey;
    this.#baseURL = baseURL;
  }
  async #request<T>(path: string): Promise<T> {
    const res = await fetch(`${this.#baseURL}${path}`, {
      headers: { Authorization: `Bearer ${this.#apiKey}` },
    });
    return res.json();
  }
  async getUsers(): Promise<User[]> {
    return this.#request<User[]>("/users");
  }
}
```

### Advanced Examples
```typescript
class SecureVault {
  #data = new WeakMap<object, unknown>();
  #lock: boolean = false;
  set(key: object, value: unknown): void {
    if (this.#lock) throw new Error("Vault is locked");
    if (typeof key !== "object" || key === null) {
      throw new Error("Key must be an object");
    }
    this.#data.set(key, value);
  }
  get(key: object): unknown {
    return this.#data.get(key);
  }
  lock(): void {
    this.#lock = true;
  }
  unlock(): void {
    this.#lock = false;
  }
}
class UserVault extends SecureVault {
  storeUser(user: User): void {
    this.set(user, { token: user.token });
  }
}
```

### Real-World Use Cases
- Storing security-sensitive data (API keys, tokens)
- Library implementations where internal state must not be exposed
- Strict encapsulation for framework internals

### Common Mistakes
- Mixing `#` private and `private` in the same class (they behave differently)
- Assuming `#` fields are accessible in subclasses (they are not)
- Using `#` fields in libraries that target older JavaScript engines without proper downleveling

### Best Practices
- Prefer `#` private fields for true runtime encapsulation
- Use `#` for internal state that must not be touched by external code
- Be aware that `#` fields cannot be accessed in subclasses — use `protected` if subclass access is needed

### Performance Considerations
Native `#` private fields are fast in modern engines. Downleveled `#` fields (using `WeakMap`) have a small performance overhead for creation and access.

### Interview Questions
- Q: Can you access a `#` private field from outside the class?
  A: No, `#` private fields are truly private at the JavaScript language level.

### Coding Challenges
- Create a `SecretKeeper` class with a `#secret` field and methods to `setSecret` and `revealSecret` with a password check.
- Build a `LinkedListNode` class using `#` fields for `value`, `next`, and `prev`.

### Related Topics
- private (TS)
- protected
- readonly modifier

---

## readonly modifier

### What It Is
The `readonly` modifier marks a property as immutable after initialization. It can only be assigned in the declaration or in the constructor.

### Why It Is Important
`readonly` prevents accidental reassignment after construction, making the code safer and documenting intent. It is key to building immutable data structures.

### How It Works Internally
`readonly` is a compile-time constraint. The emitted JavaScript has no enforcement, but the compiler catches reassignment attempts.

### Syntax
```typescript
class Example {
  readonly id: string;
  constructor(id: string) {
    this.id = id;
  }
}
```

### Beginner Examples
```typescript
class User {
  readonly id: string;
  readonly createdAt: Date;
  public name: string;
  constructor(id: string, name: string) {
    this.id = id;
    this.createdAt = new Date();
    this.name = name;
  }
}
const u = new User("u1", "Alice");
// u.id = "u2"; // Error: readonly
```

### Intermediate Examples
```typescript
class Configuration {
  readonly appName: string;
  readonly version: string;
  readonly debug: boolean;
  constructor(env: Record<string, string | undefined>) {
    this.appName = env.APP_NAME ?? "MyApp";
    this.version = env.APP_VERSION ?? "1.0.0";
    this.debug = env.NODE_ENV === "development";
  }
}
```

### Advanced Examples
```typescript
class ImmutablePoint {
  readonly x: number;
  readonly y: number;
  readonly z: number;
  constructor(x: number, y: number, z: number = 0) {
    this.x = x;
    this.y = y;
    this.z = z;
  }
  withX(x: number): ImmutablePoint {
    return new ImmutablePoint(x, this.y, this.z);
  }
  withY(y: number): ImmutablePoint {
    return new ImmutablePoint(this.x, y, this.z);
  }
}
```

### Real-World Use Cases
- Entity identifiers and timestamps
- Configuration values loaded at startup
- Value objects in Domain-Driven Design

### Common Mistakes
- Forgetting that `readonly` only prevents property reassignment, not mutation of nested objects
- Using `readonly` on methods (they should not be reassigned anyway)
- Combining `readonly` with setter — a property cannot have both `readonly` and a setter

### Best Practices
- Use `readonly` for any property that should not change after construction
- Combine `readonly` with `private` or `#` for immutable internal state
- For deep immutability, use `Readonly<T>` utility type or `as const`

### Performance Considerations
No runtime cost. Compile-time only.

### Interview Questions
- Q: Can you assign a `readonly` property in a method other than the constructor?
  A: No, `readonly` properties can only be assigned at declaration or in the constructor.

### Coding Challenges
- Create an immutable `Money` class with `readonly amount` and `readonly currency`, and methods `add` and `subtract` that return new instances.
- Build a `readonly` version of a `ShoppingCart` where adding items returns a new cart.

### Related Topics
- public default
- private (TS)
- Parameter properties
