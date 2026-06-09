# Enums - Numeric enums, string enums, const enums, reverse mappings, enum best practices

## Introduction

Enums are a feature that allows developers to define a set of named constants. TypeScript offers both numeric and string enums, along with specialized variants like const enums. Enums help make code more readable by replacing magic numbers and strings with descriptive names.

## Numeric enums

### What It Is

A numeric enum is an enum where each member is assigned a numeric value. By default, members auto-increment starting from 0. Numeric enums also support reverse mapping — you can look up a member name by its value.

### Why It Is Important

Numeric enums provide a readable way to represent related numeric constants. They're useful for status codes, flags, and categories where numeric values have specific meanings. Auto-incrementing reduces boilerplate and ensures uniqueness.

### How It Works Internally

Numeric enums compile to JavaScript objects with both forward (name-to-value) and reverse (value-to-name) mappings. The compiled code creates an object where each member name is a key mapping to its value, and each value is also a key mapping back to its name.

`	ypescript
// TypeScript
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right  // 3
}

// Compiled JavaScript
// var Direction;
// (function (Direction) {
//   Direction[Direction["Up"] = 0] = "Up";
//   Direction[Direction["Down"] = 1] = "Down";
//   Direction[Direction["Left"] = 2] = "Left";
//   Direction[Direction["Right"] = 3] = "Right";
// })(Direction || (Direction = {}));
`

### Syntax

`	ypescript
// Basic numeric enum
enum Direction {
  Up,
  Down,
  Left,
  Right,
}

// With custom start value
enum StatusCode {
  OK = 200,
  Created = 201,
  BadRequest = 400,
  Unauthorized = 401,
  NotFound = 404,
  InternalServerError = 500,
}

// With computed values
enum FileAccess {
  None = 0,
  Read = 1 << 0,
  Write = 1 << 1,
  ReadWrite = Read | Write,
}

// Access
const dir: Direction = Direction.Up;
const status: StatusCode = StatusCode.OK;
`

### Beginner Examples

`	ypescript
enum Color {
  Red,
  Green,
  Blue,
}

function getColorHex(color: Color): string {
  switch (color) {
    case Color.Red:
      return "#FF0000";
    case Color.Green:
      return "#00FF00";
    case Color.Blue:
      return "#0000FF";
  }
}

console.log(getColorHex(Color.Red)); // #FF0000
console.log(Color.Red); // 0
console.log(Color[0]); // "Red"

enum Weekday {
  Monday,
  Tuesday,
  Wednesday,
  Thursday,
  Friday,
  Saturday,
  Sunday,
}

function isWeekend(day: Weekday): boolean {
  return day === Weekday.Saturday || day === Weekday.Sunday;
}
`

### Intermediate Examples

`	ypescript
// Heterogeneous enum (mix of initialized and auto)
enum Response {
  No = 0,
  Yes = 1,
  Maybe = 2,
}

// Flags with bitwise operators
enum Permission {
  None = 0,
  Read = 1,
  Write = 2,
  Execute = 4,
  All = Read | Write | Execute,
}

function hasPermission(userPerms: Permission, required: Permission): boolean {
  return (userPerms & required) === required;
}

const userPerms = Permission.Read | Permission.Write;
console.log(hasPermission(userPerms, Permission.Read)); // true
console.log(hasPermission(userPerms, Permission.Execute)); // false

// Enum as type
function moveCharacter(direction: Direction, distance: number): void {
  console.log(Moving  by  units);
}

moveCharacter(Direction.Up, 10);
`

### Advanced Examples

`	ypescript
// Enum with methods (using namespace merging)
enum HttpStatus {
  OK = 200,
  Created = 201,
  BadRequest = 400,
  NotFound = 404,
  InternalServerError = 500,
}

namespace HttpStatus {
  export function isSuccess(status: HttpStatus): boolean {
    return status >= 200 && status < 300;
  }

  export function isClientError(status: HttpStatus): boolean {
    return status >= 400 && status < 500;
  }

  export function isServerError(status: HttpStatus): boolean {
    return status >= 500 && status < 600;
  }
}

console.log(HttpStatus.isSuccess(HttpStatus.OK)); // true
console.log(HttpStatus.isClientError(HttpStatus.NotFound)); // true

// Enum as discriminated union member
type Result =
  | { kind: HttpStatus.OK; data: unknown }
  | { kind: HttpStatus.Created; id: string }
  | { kind: HttpStatus.NotFound; message: string };

// Enum with computed members
enum MathConstants {
  PI = 3.14159,
  E = 2.71828,
  PHI = 1.61803,
}

// Enum iteration
function getAllEnumValues<T extends Record<string, string | number>>(
  enumObj: T
): T[keyof T][] {
  return Object.values(enumObj).filter(
    (v) => typeof v === "number"
  ) as T[keyof T][];
}

const directions = getAllEnumValues(Direction);
// [0, 1, 2, 3]
`

### Real-World Use Cases

**HTTP Status Codes**: Enum for all standard HTTP status codes.

**User Roles**: enum Role { Admin, Editor, Viewer }.

**Order States**: enum OrderState { Pending, Confirmed, Shipped, Delivered, Cancelled }.

### Common Mistakes

**Assuming Type Safety Across Enum Types**: Different enums with the same value are still incompatible.

**Mutable Enum Objects**: Enum objects can be modified at runtime — a potential bug source.

**Using Negative or Float Values**: TypeScript allows them but they can be confusing.

### Best Practices

1. Use PascalCase for enum names and members
2. Explicitly initialize the first value if you need a specific start
3. Use const enum when you don't need reverse mappings
4. Avoid heterogeneous enums (mixing strings and numbers)
5. Consider union types as an alternative for simple cases

### Performance Considerations

Numeric enums generate runtime objects with both forward and reverse mappings. For performance-critical code, consider const enum which inlines values. Reverse mapping adds to bundle size.

### Interview Questions

1. How does TypeScript implement reverse mapping for numeric enums?
2. What's the default starting value for numeric enums?
3. Can you use computed values in numeric enums?

### Coding Challenges

1. Create a bitwise flag enum for file permissions
2. Implement an enum with associated metadata using namespace merging

### Related Topics

- String Enums
- Const Enums
- Union Types as Enum Alternatives

## String enums

### What It Is

A string enum is an enum where each member is initialized with a string literal. Unlike numeric enums, string enums don't have auto-incrementing behavior or reverse mapping.

### Why It Is Important

String enums provide meaningful runtime values (the actual strings) compared to numeric enums where the value is just a number. They're useful when the enum value needs to be serialized, logged, or used in API communication.

### How It Works Internally

String enums compile to JavaScript objects with only forward mapping (name-to-value). They don't generate reverse mapping. Each member must be initialized with a string literal or another string enum member.

### Syntax

`	ypescript
// Basic string enum
enum Direction2 {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}

// String enum with related strings
enum LogLevel {
  Error = "error",
  Warn = "warn",
  Info = "info",
  Debug = "debug",
}

// Access
const dir2: Direction2 = Direction2.Up; // "UP"
const level: LogLevel = LogLevel.Info; // "info"
`

### Beginner Examples

`	ypescript
enum Color2 {
  Red = "RED",
  Green = "GREEN",
  Blue = "BLUE",
}

function getColorName(color: Color2): string {
  return The color is ;
}

console.log(getColorName(Color2.Red)); // "The color is RED"

// Comparing string enums
function isPrimaryColor(color: Color2): boolean {
  return color === Color2.Red || color === Color2.Green || color === Color2.Blue;
}

// Using enum values in objects
const colorMap: Record<Color2, string> = {
  [Color2.Red]: "#FF0000",
  [Color2.Green]: "#00FF00",
  [Color2.Blue]: "#0000FF",
};
`

### Intermediate Examples

`	ypescript
// String enum for API actions
enum ApiAction {
  Create = "CREATE",
  Read = "READ",
  Update = "UPDATE",
  Delete = "DELETE",
}

function performAction(action: ApiAction, resource: string): void {
  switch (action) {
    case ApiAction.Create:
      console.log(Creating );
      break;
    case ApiAction.Read:
      console.log(Reading );
      break;
    case ApiAction.Update:
      console.log(Updating );
      break;
    case ApiAction.Delete:
      console.log(Deleting );
      break;
  }
}

// String enum serialization
enum Environment {
  Development = "development",
  Staging = "staging",
  Production = "production",
}

const config = {
  env: Environment.Production as string,
  apiUrl: "https://api.example.com",
};

// Type-safe string enum parsing
function parseEnvironment(value: string): Environment | undefined {
  return Object.values(Environment).includes(value as Environment)
    ? (value as Environment)
    : undefined;
}
`

### Advanced Examples

`	ypescript
// String enum with custom utility
enum EventType {
  UserCreated = "user:created",
  UserUpdated = "user:updated",
  UserDeleted = "user:deleted",
  OrderPlaced = "order:placed",
  OrderShipped = "order:shipped",
}

namespace EventType {
  export function fromString(value: string): EventType | null {
    const entries = Object.values(EventType) as string[];
    return entries.includes(value) ? (value as EventType) : null;
  }

  export function getCategory(event: EventType): "user" | "order" {
    return event.startsWith("user:") ? "user" : "order";
  }
}

// String enum with generic handler
type EventHandler<T extends EventType> = T extends EventType.UserCreated
  ? (user: { id: string; name: string }) => void
  : T extends EventType.OrderPlaced
    ? (order: { id: string; total: number }) => void
    : (data: unknown) => void;

const handlers: Record<EventType, EventHandler<EventType>> = {
  [EventType.UserCreated]: (user) => console.log(User created: ),
  [EventType.UserUpdated]: (data) => console.log("User updated", data),
  [EventType.UserDeleted]: (data) => console.log("User deleted", data),
  [EventType.OrderPlaced]: (order) => console.log(Order: ),
  [EventType.OrderShipped]: (data) => console.log("Order shipped", data),
};
`

### Real-World Use Cases

**API Action Types**: CRUD operations with descriptive string values.

**Event Names**: Emitted event names with namespaced strings.

**Configuration Keys**: Environment names, feature flags.

**Localization Keys**: i18n keys using string enums.

### Common Mistakes

**Missing Initializer**: String enum members must be initialized — they don't auto-increment.

**Assuming Reverse Mapping**: String enums don't have reverse mapping.

**Using String Enums for Large Sets**: Consider union types for sets with 10+ members.

### Best Practices

1. Always initialize string enum members explicitly
2. Use consistent naming (e.g., all uppercase)
3. Prefer string enums for API/serialization boundaries
4. Consider union of string literals for simple cases

### Performance Considerations

String enums generate an object at runtime. For zero-cost alternatives, use const enum (inlines values) or union types (completely erased).

### Interview Questions

1. How do string enums differ from numeric enums?
2. Do string enums have reverse mapping?
3. When would you use a string enum over a numeric enum?

### Coding Challenges

1. Create a string enum for HTTP methods and a function that uses it
2. Implement a type-safe event system using string enums

### Related Topics

- Numeric Enums
- Const Enums
- Union Types

## const enums

### What It Is

const enum is a specialized enum that is completely inlined at compile time rather than generating a runtime object. Const enums are declared with the const modifier before the enum keyword.

### Why It Is Important

Const enums provide the readability of enums with zero runtime overhead. They're ideal for performance-critical code where bundle size matters, and when you don't need runtime enum features like reverse mapping or iteration.

### How It Works Internally

Const enums are completely erased during compilation. Every reference to a const enum member is replaced with its literal value. No JavaScript object is generated — the enum exists only at compile time.

`	ypescript
// TypeScript
const enum Color {
  Red,
  Green,
  Blue,
}

const myColor = Color.Red;
const yourColor = Color.Blue;

// Compiled JavaScript
// const myColor = 0; /* Red */
// const yourColor = 2; /* Blue */
`

### Syntax

`	ypescript
// Basic const enum
const enum Direction {
  Up,
  Down,
  Left,
  Right,
}

const dir: Direction = Direction.Up; // Compiles to: const dir = 0;

// Const enum with explicit values
const enum HttpStatus {
  OK = 200,
  NotFound = 404,
  Error = 500,
}

const status = HttpStatus.OK; // Compiles to: const status = 200;
`

### Beginner Examples

`	ypescript
const enum Size {
  Small = "S",
  Medium = "M",
  Large = "L",
  ExtraLarge = "XL",
}

function getSizeLabel(size: Size): string {
  return Size: ;
}

const mySize = Size.Medium;
console.log(getSizeLabel(mySize));

// Const enum in switch
const enum Fruit {
  Apple = "apple",
  Banana = "banana",
  Orange = "orange",
}

function getFruitColor(fruit: Fruit): string {
  switch (fruit) {
    case Fruit.Apple: return "red";
    case Fruit.Banana: return "yellow";
    case Fruit.Orange: return "orange";
  }
}
`

### Intermediate Examples

`	ypescript
// Const enum with computed values
const enum BitFlags {
  None = 0,
  Ready = 1 << 0,
  Loading = 1 << 1,
  Error = 1 << 2,
  Complete = 1 << 3,
}

function setFlag(flags: BitFlags, flag: BitFlags): BitFlags {
  return flags | flag;
}

function hasFlag(flags: BitFlags, flag: BitFlags): boolean {
  return (flags & flag) === flag;
}

// Const enum with string values
const enum Environment {
  Dev = "development",
  Staging = "staging",
  Prod = "production",
}

function getConfig(env: Environment): { apiUrl: string } {
  if (env === Environment.Dev) {
    return { apiUrl: "http://localhost:3000" };
  }
  return { apiUrl: "https://api.example.com" };
}
`

### Advanced Examples

`	ypescript
// Const enum with inlining across files
// file: colors.ts
export const enum Color {
  Red = "#FF0000",
  Green = "#00FF00",
  Blue = "#0000FF",
}

// file: app.ts (with preserveConstEnums: false)
// import { Color } from "./colors";
const primary = Color.Red;
// Compiles to: const primary = "#FF0000";

// Const enum in ambient declarations
declare const enum MouseButton {
  Left = 0,
  Middle = 1,
  Right = 2,
}

// Const enum for performance-critical code
const enum Operation {
  Add = "+",
  Subtract = "-",
  Multiply = "*",
  Divide = "/",
}

function calculate(a: number, b: number, op: Operation): number {
  switch (op) {
    case Operation.Add: return a + b;
    case Operation.Subtract: return a - b;
    case Operation.Multiply: return a * b;
    case Operation.Divide: return b !== 0 ? a / b : NaN;
  }
}

// The switch compiles to direct value comparisons
`

### Real-World Use Cases

**Performance Hot Paths**: Game development, real-time systems.

**Library Development**: When bundle size is critical.

**Embedded Constants**: Mathematical constants, configuration values.

### Common Mistakes

**Assuming Runtime Enum Features**: Const enums don't exist at runtime — no Object.values, no enum[value].

**Isolated Modules Limitation**: With isolatedModules: true, const enums from other files may not work.

**Using const enum in Libraries**: Library consumers can't use const enum without the source — consider regular enums for public APIs.

### Best Practices

1. Use const enum for internal constants, not library exports
2. Use preserveConstEnums: true for debugging
3. Prefer const enums in performance-critical paths
4. Document that const enums are compile-time only

### Performance Considerations

Const enums have the best performance — zero runtime overhead, values inlined at usage sites. The trade-off is loss of runtime features. Bundle size is reduced since no enum object is generated.

### Interview Questions

1. What happens to const enums during compilation?
2. When should you NOT use const enums?
3. How do const enums differ from regular enums regarding reverse mapping?

### Coding Challenges

1. Refactor a regular enum to a const enum and compare compiled output
2. Create a const enum for bitwise flags

### Related Topics

- Regular Enums
- Preserve Const Enums
- Isolated Modules

## Reverse mappings and runtime behavior

### What It Is

Reverse mapping is the ability to look up an enum member name from its value. This is a runtime feature of numeric enums where TypeScript generates an object with both forward and reverse mappings.

### Why It Is Important

Reverse mapping enables runtime introspection of enums — converting values to human-readable names for logging, debugging, and display. It's also useful when deserializing values back to enum members.

### How It Works Internally

The compiled JavaScript for a numeric enum generates an IIFE that creates an object with bidirectional mappings:

`	ypescript
enum Direction { Up, Down, Left, Right }

// Generated JS:
// var Direction;
// (function (Direction) {
//   Direction[Direction["Up"] = 0] = "Up";
//   Direction[Direction["Down"] = 1] = "Down";
//   Direction[Direction["Left"] = 2] = "Left";
//   Direction[Direction["Right"] = 3] = "Right";
// })(Direction || (Direction = {}));
`

This creates: { "0": "Up", "1": "Down", "2": "Left", "3": "Right", Up: 0, Down: 1, Left: 2, Right: 3 }

### Syntax

`	ypescript
enum Status {
  Active = "ACTIVE", Inactive = "INACTIVE", Pending = "PENDING"
}

// Numeric enum — has reverse mapping
enum NumericStatus {
  Active,    // 0
  Inactive,  // 1
  Pending,   // 2
}

const name = NumericStatus[0]; // "Active"
const value = NumericStatus.Active; // 0

// String enum — no reverse mapping
// const name2 = Status["ACTIVE"]; // Error: no reverse mapping
// Status["ACTIVE"] returns string "ACTIVE", not the enum member
`

### Beginner Examples

`	ypescript
// Reverse mapping for logging
enum LogLevel {
  Error = 0,
  Warn = 1,
  Info = 2,
  Debug = 3,
}

function log(level: LogLevel, message: string): void {
  const levelName = LogLevel[level];
  console.log([] );
}

log(LogLevel.Info, "Application started"); // "[Info] Application started"

// Parsing from numeric value
function parseDay(value: number): string {
  enum Day { Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday }
  return Day[value] ?? "Invalid day";
}
`

### Intermediate Examples

`	ypescript
// Enum deserialization from API
enum OrderStatus {
  Pending = 0,
  Confirmed = 1,
  Shipped = 2,
  Delivered = 3,
  Cancelled = 4,
}

interface ApiOrder {
  id: string;
  status: number; // API returns numeric
}

function parseOrder(apiOrder: ApiOrder): { id: string; status: OrderStatus } {
  return {
    id: apiOrder.id,
    status: apiOrder.status as OrderStatus,
  };
}

// Enum iteration with reverse mapping
function getEnumMembers<T extends Record<string, string | number>>(
  enumObj: T
): { name: string; value: T[keyof T] }[] {
  return Object.entries(enumObj)
    .filter(([key]) => isNaN(Number(key)))
    .map(([name, value]) => ({ name, value: value as T[keyof T] }));
}

const members = getEnumMembers(Direction);
// [{ name: "Up", value: 0 }, { name: "Down", value: 1 }, ...]
`

### Advanced Examples

`	ypescript
// Safe reverse mapping with type safety
function enumValueToName<T extends Record<string, string | number>>(
  enumObj: T,
  value: T[keyof T]
): string | undefined {
  const entries = Object.entries(enumObj) as [string, T[keyof T]][];
  const entry = entries.find(([key, val]) => val === value && isNaN(Number(key)));
  return entry?.[0];
}

// Enum validation from external data
function isValidEnumValue<T extends Record<string, string | number>>(
  enumObj: T,
  value: unknown
): value is T[keyof T] {
  return Object.values(enumObj).includes(value as T[keyof T]);
}

// Enum with runtime metadata
enum Priority {
  Low = 0,
  Medium = 1,
  High = 2,
  Critical = 3,
}

namespace Priority {
  export function getLabel(value: Priority): string {
    const labels: Record<Priority, string> = {
      [Priority.Low]: "Low Priority",
      [Priority.Medium]: "Medium Priority",
      [Priority.High]: "High Priority",
      [Priority.Critical]: "Critical Priority",
    };
    return labels[value];
  }

  export function getColor(value: Priority): string {
    const colors: Record<Priority, string> = {
      [Priority.Low]: "green",
      [Priority.Medium]: "yellow",
      [Priority.High]: "orange",
      [Priority.Critical]: "red",
    };
    return colors[value];
  }
}

console.log(Priority.getLabel(Priority.High)); // "High Priority"
`

### Real-World Use Cases

**Logging Enum Values**: Convert numbers to readable names.

**API Response Parsing**: Map numeric/string values from API to enum members.

**Dropdown Options**: Generate UI options from enum members.

**Serialization/Deserialization**: Converting between types and wire formats.

### Common Mistakes

**Assuming String Enums Have Reverse Mapping**: String enums only have forward mapping.

**Relying on Enum Order**: Enum member order in iteration isn't guaranteed.

**Using Non-Numeric Reverse Mapping**: Only numeric enums generate the bidirectional object.

### Best Practices

1. Use reverse mapping for display/debugging purposes
2. Don't rely on enum iteration order
3. Prefer forward mapping (enum.Name) for actual usage
4. Use namespace merging for additional enum metadata

### Performance Considerations

Reverse mapping generates a runtime object with O(n) entries. For large enums, this adds to bundle size. const enum eliminates this entirely. Consider const enum for large enums in performance-critical code.

### Interview Questions

1. How does reverse mapping work for numeric enums?
2. Do string enums have reverse mapping?
3. How can you add methods to an enum?

### Coding Challenges

1. Implement a generic function that converts any numeric enum to an array of {name, value} pairs
2. Create a type-safe enum parser that validates and converts external values

### Related Topics

- Const Enums
- Namespace Merging
- Runtime Type Information
