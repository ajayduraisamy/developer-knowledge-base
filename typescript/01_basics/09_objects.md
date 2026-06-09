# Objects - Object types, readonly properties, index signatures, nested objects, optional properties

## Introduction

Objects are the primary data structure for modeling real-world entities in TypeScript. The type system provides rich capabilities for describing object shapes, including property types, optionality, mutability constraints, index signatures for dynamic properties, and nested object hierarchies.

## Object type syntax

### What It Is

Object types describe the shape of an object — what properties it has and their types. Object types can be defined inline or using interface or 	ype aliases. They use a structural type system where compatibility is determined by shape, not declaration.

### Why It Is Important

Object types are the foundation for modeling data in TypeScript. They ensure that objects have the expected properties with the correct types, catching missing or mis-typed properties at compile time. They serve as contracts between different parts of a codebase.

### How It Works Internally

TypeScript's structural typing means two object types are compatible if one has all the properties of the other (excess property checking applies to object literals). The type checker validates property access, assignment, and method calls against the declared shape.

### Syntax

`	ypescript
// Inline object type
const person: { name: string; age: number } = {
  name: "Alice",
  age: 30,
};

// Type alias for object
type Person = {
  name: string;
  age: number;
};

// Interface for object
interface PersonInterface {
  name: string;
  age: number;
}

// Function with object parameter
function greet(person: { name: string; age: number }): string {
  return \Hello, \! You are \.\;
}

// Object type with methods
type Calculator = {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
};
`

### Beginner Examples

`	ypescript
// Basic object types
const user: { id: number; username: string } = {
  id: 1,
  username: "alice",
};

// Object with mixed types
const product: {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
} = {
  id: 101,
  name: "Laptop",
  price: 999.99,
  inStock: true,
};

// Accessing properties
console.log(product.name);
console.log(product.price);

// Object as function parameter
function displayProduct(p: { id: number; name: string; price: number }): string {
  return \\: \$\\;
}

console.log(displayProduct(product));
`

### Intermediate Examples

`	ypescript
// Object with methods
type Counter = {
  value: number;
  increment(): void;
  decrement(): void;
  reset(): void;
};

function createCounter(initial: number = 0): Counter {
  return {
    value: initial,
    increment() { this.value++; },
    decrement() { this.value--; },
    reset() { this.value = initial; },
  };
}

// Object with call signature
type Greeter = {
  (name: string): string;
  greeting: string;
};

function createGreeter(greeting: string): Greeter {
  const fn = ((name: string) => \\, \!\) as Greeter;
  fn.greeting = greeting;
  return fn;
}

// Generic object types
type Box<T> = {
  value: T;
  get(): T;
  set(value: T): void;
};

function createBox<T>(initial: T): Box<T> {
  let value = initial;
  return {
    get() { return value; },
    set(newValue: T) { value = newValue; },
    value,
  };
}
`

### Advanced Examples

`	ypescript
// Mapped object types
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

type Optional2<T> = {
  [P in keyof T]?: T[P];
};

type Readonly4<T> = {
  readonly [P in keyof T]: T[P];
};

// Conditional object properties
type Conditional<T extends boolean> = T extends true
  ? { enabled: true; value: string }
  : { enabled: false; value?: never };

function processConfig<T extends boolean>(config: Conditional<T>): void {
  if (config.enabled) {
    console.log(config.value);
  }
}

// Intersection object types
type HasName = { name: string };
type HasAge = { age: number };
type Person2 = HasName & HasAge & { email: string };

// Discriminated object unions
type Shape2 =
  | { kind: "circle"; radius: number }
  | { kind: "rectangle"; width: number; height: number }
  | { kind: "triangle"; base: number; height: number };

function area2(shape: Shape2): number {
  switch (shape.kind) {
    case "circle": return Math.PI * shape.radius ** 2;
    case "rectangle": return shape.width * shape.height;
    case "triangle": return (shape.base * shape.height) / 2;
  }
}
`

### Real-World Use Cases

**API Responses**: Typed objects for API request/response bodies.

**Configuration Objects**: Application and component configuration.

**State Management**: Redux/NgRx state objects with typed properties.

**Form Data**: Typed form field objects with validation.

### Common Mistakes

**Excess Property Checking on Object Literals**: Assigning an object literal directly to a typed variable checks for extra properties.

`	ypescript
interface Point { x: number; y: number; }
// const p: Point = { x: 10, y: 20, z: 30 }; // Error: z does not exist
`

**Mutable Defaults in Destructuring**: Object destructuring with defaults can lead to shared mutable state.

### Best Practices

1. Prefer interfaces for object shapes
2. Use type aliases for unions and intersections
3. Use readonly for immutable properties
4. Use optional properties for optional fields
5. Use index signatures for dynamic key objects

### Performance Considerations

Object type annotations are erased at compile time — zero runtime cost. Complex mapped types can increase compilation time for large interfaces.

### Interview Questions

1. What is structural typing and how does it apply to objects?
2. What is excess property checking?
3. How do object types differ between type and interface?

### Coding Challenges

1. Create a generic object type that deeply makes all properties optional
2. Implement a type-safe object merge function

### Related Topics

- Interfaces
- Type Aliases
- Structural Typing

## Readonly and optional properties

### What It Is

Readonly properties (eadonly modifier) prevent reassignment after object creation. Optional properties (? modifier) allow the property to be absent or undefined. Both modifiers constrain how properties can be used.

### Why It Is Important

Readonly ensures data integrity by preventing unintended mutation. Optional properties make object types flexible for scenarios where some data may not be available. Together they model real-world data more accurately.

### How It Works Internally

Readonly is a compile-time constraint — attempting to assign to a readonly property produces a type error. Optional properties add | undefined to the type and make the property not required in object literals. Both are erased at runtime.

### Syntax

`	ypescript
// Readonly property
interface Config {
  readonly apiKey: string;
  readonly endpoint: string;
}

const config: Config = {
  apiKey: "abc123",
  endpoint: "https://api.example.com",
};
// config.apiKey = "new"; // Error: readonly

// Optional property
interface User {
  id: number;
  name: string;
  email?: string; // optional
}

const user1: User = { id: 1, name: "Alice" };
const user2: User = { id: 2, name: "Bob", email: "bob@example.com" };

// Combined
interface Settings {
  readonly id: string;
  readonly createdAt: Date;
  label: string;
  description?: string;
}
`

### Beginner Examples

`	ypescript
// Optional properties
interface Product2 {
  name: string;
  price: number;
  description?: string;
  category?: string;
}

const basic: Product2 = { name: "Widget", price: 9.99 };
const full: Product2 = {
  name: "Gadget",
  price: 29.99,
  description: "A fancy gadget",
  category: "electronics",
};

// Accessing optional properties
function getDescription(product: Product2): string {
  return product.description ?? "No description available";
}

// Readonly for configuration
interface DatabaseConfig {
  readonly host: string;
  readonly port: number;
  readonly username: string;
  readonly password: string;
}

const dbConfig: DatabaseConfig = {
  host: "localhost",
  port: 5432,
  username: "admin",
  password: "secret",
};

// dbConfig.port = 5433; // Error
`

### Intermediate Examples

`	ypescript
// Readonly array of objects with readonly properties
interface User2 {
  readonly id: number;
  name: string;
  readonly email: string;
}

const users: readonly User2[] = [
  { id: 1, name: "Alice", email: "alice@example.com" },
  { id: 2, name: "Bob", email: "bob@example.com" },
];

// users[0].id = 5; // Error: readonly
// users[0] = { id: 3, name: "Charlie", email: "c@example.com" }; // Error: readonly array
users[0].name = "Alicia"; // OK — name is not readonly

// Optional with nested objects
interface Address {
  street: string;
  city: string;
  zipCode?: string;
}

interface Employee {
  id: number;
  name: string;
  address?: Address;
  phone?: string;
}

// Spread with optional
function updateEmployee(
  emp: Employee,
  updates: Partial<Employee>
): Employee {
  return { ...emp, ...updates };
}

// Readonly class property
class ApiClient {
  constructor(readonly baseUrl: string) {}
  // baseUrl is set once in constructor and readonly thereafter
}
`

### Advanced Examples

`	ypescript
// Deep readonly
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

interface NestedConfig {
  server: {
    host: string;
    port: number;
    ssl: {
      enabled: boolean;
      cert: string;
    };
  };
  database: {
    url: string;
    pool: {
      min: number;
      max: number;
    };
  };
}

type ImmutableConfig = DeepReadonly<NestedConfig>;

// Conditional optional properties
type ApiResponse2<T> = {
  success: true;
  data: T;
  meta?: {
    total: number;
    page: number;
    pageSize: number;
  };
} | {
  success: false;
  error: string;
  details?: unknown;
};

// Readonly with branded types
type BrandedId<T extends string> = string & { readonly __brand: T };
type UserId = BrandedId<"User">;
type OrderId = BrandedId<"Order">;

// Optional property elimination
type RequiredExcept<T, K extends keyof T> = Omit<T, K> & {
  [P in K]?: T[P];
};

type PartialConfig = RequiredExcept<NestedConfig, "database">;
`

### Real-World Use Cases

**Immutable State**: Redux state with readonly properties prevents accidental mutation.

**Configuration Objects**: Readonly ensures configuration isn't changed at runtime.

**Optional Fields**: API responses where some fields may not be present.

**DTOs**: Data transfer objects with readonly for sent data.

### Common Mistakes

**Confusing readonly with Immutability**: Readonly is a shallow constraint — nested objects can still be mutated.

**Optional Property Checking**: Accessing an optional property without null check or default.

**Readonly in Interfaces vs Classes**: Readonly in interfaces affects usage, while in classes it also affects assignment.

### Best Practices

1. Use eadonly for properties that should not change after initialization
2. Use optional ? for genuinely optional fields
3. Prefer Readonly<T> mapped type over manual readonly
4. Use s const for deeply immutable literals
5. Always provide defaults or guards for optional properties

### Performance Considerations

Both modifiers are compile-time only — zero runtime overhead.

### Interview Questions

1. What is the difference between readonly in an interface vs a class?
2. How do you make all properties of a type readonly?
3. What happens when accessing an optional property without checking?

### Coding Challenges

1. Implement a DeepReadonly mapped type
2. Create a type that makes specific properties optional while keeping others required

### Related Topics

- Readonly Type
- Partial Type
- Required Type

## Index signatures

### What It Is

Index signatures allow objects with dynamic property names to be typed. They define the type for properties accessed by a dynamic key, using [key: string]: T or [key: number]: T syntax.

### Why It Is Important

Index signatures model objects used as dictionaries, maps, or dynamic records where property names aren't known at compile time. They enable type-safe access to dynamic properties while allowing known properties alongside dynamic ones.

### How It Works Internally

An index signature tells TypeScript that any property access with a matching key type will return the specified value type. TypeScript checks that any explicitly declared properties are compatible with the index signature's value type.

### Syntax

`	ypescript
// String index signature
type Dictionary<T> = {
  [key: string]: T;
};

const dict: Dictionary<number> = {
  apple: 5,
  banana: 3,
};
dict.orange = 7;
dict["grape"] = 2;

// Number index signature
type NumericDictionary<T> = {
  [index: number]: T;
};

const arr: NumericDictionary<string> = {
  0: "zero",
  1: "one",
  2: "two",
};

// Combined with known properties
interface Config2 {
  [key: string]: string | number;
  port: number;
  host: string;
  // name: string[]; // Error: not assignable to string | number
}
`

### Beginner Examples

`	ypescript
// Simple dictionary
type StringMap = {
  [key: string]: string;
};

const translations: StringMap = {
  hello: "hola",
  goodbye: "adiós",
  thank_you: "gracias",
};

translations.please = "por favor";

// Accessing dynamic keys
function getTranslation(key: string, dict: StringMap): string {
  return dict[key] ?? \No translation for "\"\;
}

// Cache
type Cache = {
  [key: string]: unknown;
};

const apiCache: Cache = {};
apiCache["/users"] = { id: 1, name: "Alice" };
apiCache["/products"] = [{ id: 1, name: "Laptop" }];
`

### Intermediate Examples

`	ypescript
// Index signature with union value type
type EventBus = {
  [event: string]: ((...args: any[]) => void)[];
};

const eventBus: EventBus = {
  click: [() => console.log("clicked")],
  hover: [(e: MouseEvent) => console.log(e)],
};

// Index signature with template literal
type CssProperties = {
  [key: --]: string;
};

const theme: CssProperties = {
  "--primary-color": "#3498db",
  "--secondary-color": "#2ecc71",
  "--font-size": "16px",
};

// Index signature with specific value constraints
type ScoreMap = {
  [player: string]: number;
};

const scores: ScoreMap = {
  Alice: 95,
  Bob: 87,
  Charlie: 92,
};

function getAverageScore(scores: ScoreMap): number {
  const values = Object.values(scores);
  return values.reduce((a, b) => a + b, 0) / values.length;
}
`

### Advanced Examples

`	ypescript
// Index signature with generics
class SafeMap<K extends string, V> {
  private data: { [key in K]?: V } = {};

  get(key: K): V | undefined {
    return this.data[key];
  }

  set(key: K, value: V): void {
    this.data[key] = value;
  }

  getAll(): { [key in K]?: V } {
    return { ...this.data };
  }
}

// Nested index signatures
type DeepDictionary = {
  [key: string]: string | DeepDictionary;
};

const nested: DeepDictionary = {
  level1: {
    level2: {
      value: "deep",
    },
    other: "value",
  },
  flat: "value",
};

// Conditional index signature
type IndexType<T, K extends string | number | symbol> = {
  [P in K]: T;
};

// Record utility with index signature
type UserCache = Record<string, { id: number; name: string }>;

const cache: UserCache = {
  user_1: { id: 1, name: "Alice" },
  user_2: { id: 2, name: "Bob" },
};

// Index signature with readonly
type ReadonlyDict<T> = {
  readonly [key: string]: T;
};

const config3: ReadonlyDict<string> = {
  apiKey: "secret",
  endpoint: "https://api.example.com",
};
// config3.apiKey = "new"; // Error
`

### Real-World Use Cases

**Configuration Maps**: Feature flags, environment variables.

**Caching Layers**: API response caches, computed value caches.

**Translation Dictionaries**: i18n translation maps.

**Plugin Systems**: Dynamically registered plugin handlers.

### Common Mistakes

**Type Safety with Object.keys**: Object.keys returns string[], not (keyof T)[] for index signatures.

**Mixing Index Signatures with Specific Keys**: Specific keys must match the index signature's value type.

**Using ny as Value Type**: Prefer unknown when the value type is truly unknown.

### Best Practices

1. Prefer Map<K, V> over index signatures for mutable dictionaries
2. Use Record<K, V> utility for simple dictionary types
3. Be specific with value types — avoid ny
4. Use eadonly modifier for immutable dictionaries
5. Add known properties alongside index signatures when appropriate

### Performance Considerations

Index signatures have no runtime cost. At runtime they're plain JavaScript objects. For large dictionaries, Map has better performance for frequent additions/deletions.

### Interview Questions

1. What is an index signature and when would you use it?
2. How do index signatures interact with explicit property declarations?
3. What is the difference between Record<K, V> and an index signature?

### Coding Challenges

1. Implement a type-safe event bus using index signatures
2. Create a generic dictionary class with CRUD operations

### Related Topics

- Record Type
- Map vs Object
- Indexed Access Types

## Nested object types

### What It Is

Nested object types describe objects that contain other objects as properties. TypeScript supports deeply nested type definitions, allowing accurate modeling of complex data structures like API responses, configuration hierarchies, and document data.

### Why It Is Important

Real-world data is rarely flat. Nested object types enable precise modeling of hierarchical data, ensuring type safety at every level of nesting. They're essential for working with JSON APIs, form data, and complex configurations.

### How It Works Internally

TypeScript recursively checks nested object types. Each access into a nested property is checked against the corresponding nested type. TypeScript's structural typing applies at every level — two deeply nested types are compatible if their shapes match at all levels.

### Syntax

`	ypescript
// Simple nesting
interface Address2 {
  street: string;
  city: string;
  country: string;
}

interface Person3 {
  name: string;
  address: Address2;
}

const person3: Person3 = {
  name: "Alice",
  address: {
    street: "123 Main St",
    city: "New York",
    country: "USA",
  },
};

// Deep nesting
interface Company {
  name: string;
  headquarters: Address2;
  departments: {
    engineering: {
      manager: Person3;
      employees: Person3[];
    };
    sales: {
      manager: Person3;
      employees: Person3[];
    };
  };
}
`

### Beginner Examples

`	ypescript
// Blog post with nested author and comments
interface Author {
  id: number;
  name: string;
  avatar?: string;
}

interface Comment {
  id: number;
  text: string;
  author: Author;
  createdAt: Date;
}

interface BlogPost {
  id: number;
  title: string;
  content: string;
  author: Author;
  comments: Comment[];
  tags: string[];
}

const post: BlogPost = {
  id: 1,
  title: "TypeScript Tips",
  content: "Lorem ipsum...",
  author: { id: 1, name: "Alice" },
  comments: [
    { id: 1, text: "Great post!", author: { id: 2, name: "Bob" }, createdAt: new Date() },
  ],
  tags: ["typescript", "programming"],
};

// Accessing nested properties
console.log(post.author.name);
console.log(post.comments[0].text);
`

### Intermediate Examples

`	ypescript
// Self-referencing nested types
interface TreeNode<T> {
  value: T;
  children: TreeNode<T>[];
}

const tree: TreeNode<number> = {
  value: 1,
  children: [
    {
      value: 2,
      children: [
        { value: 4, children: [] },
        { value: 5, children: [] },
      ],
    },
    {
      value: 3,
      children: [
        { value: 6, children: [] },
      ],
    },
  ],
};

// Nested optional types
interface DeepConfig {
  server?: {
    host?: string;
    port?: number;
    ssl?: {
      enabled?: boolean;
      certPath?: string;
    };
  };
  database?: {
    url?: string;
    credentials?: {
      username?: string;
      password?: string;
    };
  };
}

function getPort(config: DeepConfig): number {
  return config.server?.port ?? 3000;
}
`

### Advanced Examples

`	ypescript
// Generic nested type transformation
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

type DeepRequired<T> = {
  [P in keyof T]-?: T[P] extends object ? DeepRequired<T[P]> : T[P];
};

type DeepReadonly5<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly5<T[P]> : T[P];
};

// Nested type with discriminated union
type ApiResponse3<T> = {
  status: number;
  data: T;
  meta?: {
    pagination?: {
      page: number;
      pageSize: number;
      total: number;
    };
    links?: {
      self: string;
      next?: string;
      prev?: string;
    };
  };
};

// Recursive nested type
type JSONValue =
  | string
  | number
  | boolean
  | null
  | JSONValue[]
  | { [key: string]: JSONValue };

// Type-safe deep clone
function deepClone<T>(obj: T): T {
  if (obj === null || typeof obj !== "object") return obj;
  if (Array.isArray(obj)) return obj.map(deepClone) as unknown as T;
  const cloned: Record<string, unknown> = {};
  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      cloned[key] = deepClone((obj as Record<string, unknown>)[key]);
    }
  }
  return cloned as T;
}
`

### Real-World Use Cases

**API Response Structures**: Nested JSON responses with user, posts, comments.

**Document Databases**: MongoDB-like document structures with nested arrays and objects.

**Form Wizard Data**: Multi-step form data with nested sections.

**Configuration Files**: Deeply nested JSON/YAML configuration.

### Common Mistakes

**Excessively Deep Nesting**: Beyond 3-4 levels, consider flattening or using different data modeling.

**Not Handling Optional Deep Paths**: Each optional level requires ?. for safe access.

**Mutable Nested Properties**: Deeply nested objects can be mutated at any level — use DeepReadonly for immutability.

### Best Practices

1. Keep nesting to 3-4 levels maximum
2. Use optional chaining for optional nested properties
3. Prefer flat structures when possible
4. Use type utilities (DeepPartial, DeepReadonly) for nested types
5. Extract nested types into named interfaces for clarity

### Performance Considerations

Nested types add compilation time proportional to depth. Deep conditional types can be expensive — consider breaking into smaller interfaces.

### Interview Questions

1. How do you type a recursive nested object?
2. What is DeepPartial and how would you implement it?
3. How does optional chaining work with nested optional properties?

### Coding Challenges

1. Implement DeepPartial, DeepRequired, and DeepReadonly
2. Create a type-safe function to safely access deeply nested optional properties

### Related Topics

- Recursive Types
- Conditional Types in Depth
- Optional Chaining
