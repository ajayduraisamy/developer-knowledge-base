# Functions - Parameter types, return types, optional params, default params, overloads

## Introduction

Functions are the fundamental building blocks of TypeScript applications. TypeScript enhances JavaScript functions with a powerful type system that allows precise specifications of parameters, return values, and variations in function signatures. This guide covers everything from basic parameter and return type annotations to advanced patterns like function overloads, ensuring your functions are both safe and expressive.

## Parameter type annotations

### What It Is

Parameter type annotations specify the expected types of function parameters. They enable TypeScript to validate that callers pass arguments of the correct type and provide contextual typing for the function body.

### Why It Is Important

JavaScript functions accept any argument regardless of type, leading to runtime errors. TypeScript's parameter type annotations catch these errors at compile time, serve as inline documentation, and enable better tooling (autocomplete, refactoring) for function consumers.

### How It Works Internally

During type checking, TypeScript performs contravariant checking on function parameters: the function parameter type must be a supertype of what the caller provides. This is part of TypeScript's structural type system — a function expecting Animal can be called with Dog (if Dog extends Animal), but a function expecting Dog cannot be called with a general Animal.

### Syntax

`	ypescript
function greet(name: string): void {
  console.log(\Hello, \!\);
}

function add(x: number, y: number): number {
  return x + y;
}

const multiply = (a: number, b: number): number => a * b;

function createUser({ name, age }: { name: string; age: number }): void {
  console.log(\\ is \ years old\);
}

function sum(...numbers: number[]): number {
  return numbers.reduce((acc, n) => acc + n, 0);
}
`

### Beginner Examples

`	ypescript
function double(x: number): number {
  return x * 2;
}
console.log(double(5));

function calculateRectangle(width: number, height: number): number {
  return width * height;
}
console.log(calculateRectangle(10, 5));

function toTitleCase(text: string): string {
  return text.charAt(0).toUpperCase() + text.slice(1).toLowerCase();
}
console.log(toTitleCase("hello world"));

function processOrder(orderId: string, urgent: boolean): void {
  if (urgent) {
    console.log(\Processing order \ urgently!\);
  } else {
    console.log(\Processing order \ normally\);
  }
}
`

### Intermediate Examples

`	ypescript
function formatId(id: string | number): string {
  return \ID: \\;
}

interface User {
  id: number;
  name: string;
  email: string;
}

function displayUser(user: User): string {
  return \\ <\>\;
}

function fetchData(url: string, callback: (data: string) => void): void {
  setTimeout(() => {
    callback(\Data from \\);
  }, 1000);
}

interface Point {
  x: number;
  y: number;
}

function distanceFromOrigin({ x, y }: Point): number {
  return Math.sqrt(x * x + y * y);
}
`

### Advanced Examples

`	ypescript
function firstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(item: T): T {
  console.log(item.length);
  return item;
}

function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

function transformer<T, U>(input: T, fn: (value: T) => U): U {
  return fn(input);
}

function concat<T extends readonly unknown[], U extends readonly unknown[]>(
  arr1: [...T],
  arr2: [...U]
): [...T, ...U] {
  return [...arr1, ...arr2];
}
`

### Real-World Use Cases

**Express.js Route Handlers**: Typing request and response objects ensures endpoints are used correctly.

`	ypescript
import { Request, Response } from "express";

function getUserHandler(req: Request, res: Response): void {
  const userId = req.params.id;
  res.json({ id: userId });
}
`

**React Components**: Props typing ensures components receive correct data.

`	ypescript
interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;
}
`

### Common Mistakes

**Not Typing Parameters**: Parameters lack contextual inference — they must be annotated.

**Too Broad Parameter Types**: Using any defeats the purpose of TypeScript.

### Best Practices

1. Always annotate function parameters
2. Use interfaces for complex parameter objects
3. Use generics for reusable functions
4. Prefer readonly parameters
5. Use destructured parameter syntax for options objects

### Performance Considerations

Parameter type annotations are erased at compile time — zero runtime cost.

### Interview Questions

1. Why must function parameters be explicitly annotated?
2. What is contravariance in function parameter typing?
3. How do generic parameter constraints work?

### Coding Challenges

1. Implement a typed pipe function that chains transformations
2. Create a generic function that accepts a callback with type safety

### Related Topics

- Generic Functions
- Function Type Expressions
- Rest Parameters and Spread

## Return type annotations

### What It Is

Return type annotations specify the type of value a function produces. They appear after the parameter list, preceded by a colon.

### Why It Is Important

Explicit return type annotations provide a contract between the function and its callers. They prevent accidental changes to the return type during refactoring.

### How It Works Internally

TypeScript checks that every return path in a function body produces a value assignable to the annotated return type. For functions with no return annotation, TypeScript infers the return type from the function body.

### Syntax

`	ypescript
function greet(name: string): string {
  return \Hello, \!\;
}

function log(message: string): void {
  console.log(message);
}

function throwError(message: string): never {
  throw new Error(message);
}

async function fetchData(url: string): Promise<string> {
  const response = await fetch(url);
  return response.text();
}

const double = (x: number): number => x * 2;
`

### Beginner Examples

`	ypescript
function add(a: number, b: number): number {
  return a + b;
}

function isEven(n: number): boolean {
  return n % 2 === 0;
}

function repeat(str: string, times: number): string {
  return str.repeat(times);
}

function showToast(message: string): void {
  console.log(\Toast: \\);
}
`

### Intermediate Examples

`	ypescript
function getValue(key: string): string | null {
  return localStorage.getItem(key);
}

function reverse<T>(items: T[]): T[] {
  return items.slice().reverse();
}

type Result<T> = { success: true; data: T } | { success: false; error: string };

function processApiResponse<T>(response: string): Result<T> {
  try {
    const data = JSON.parse(response) as T;
    return { success: true, data };
  } catch {
    return { success: false, error: "Invalid JSON" };
  }
}
`

### Advanced Examples

`	ypescript
interface Builder<T> {
  add(value: T): this;
  build(): T[];
}

class NumberBuilder implements Builder<number> {
  private items: number[] = [];
  add(value: number): this {
    this.items.push(value);
    return this;
  }
  build(): number[] {
    return this.items;
  }
}
`

### Real-World Use Cases

**Service Layer**: Service functions with explicit return types ensure API contracts.

**API Handlers**: Return types document what data endpoints return.

### Common Mistakes

**Omitting Return Type on Public Functions**: Changes to function body can silently change the return type.

**Using void When Function Actually Returns**: void means the return value is ignored.

### Best Practices

1. Always annotate return types on exported functions
2. Let internal functions rely on inference
3. Use void for side-effect functions

### Performance Considerations

Return type annotations are erased at compile time.

### Interview Questions

1. When should you annotate return types vs rely on inference?
2. What is the difference between void and undefined return types?

### Coding Challenges

1. Write a function with complex nested return type
2. Create a generic identity function with explicit return type

### Related Topics

- Void vs Undefined
- Never Type
- Async Function Return Types

## Optional and default parameters

### What It Is

Optional parameters (marked with ?) allow callers to omit certain arguments. Default parameters provide fallback values when arguments are undefined.

### Why It Is Important

Optional and default parameters reduce boilerplate by eliminating manual undefined checks. They make APIs more ergonomic.

### How It Works Internally

Optional parameters have type T | undefined. Default parameters are evaluated at call time only when the argument is undefined.

`	ypescript
function greet(name: string, greeting?: string): string {
  return \\, \!\;
}

function greetWithDefault(name: string, greeting = "Hello"): string {
  return \\, \!\;
}
`

### Syntax

`	ypescript
function createUser(name: string, age?: number) {
  return { name, ...(age !== undefined ? { age } : {}) };
}

function multiply(a: number, b: number = 1): number {
  return a * b;
}

function fetch(url: string, callback?: (data: string) => void): void {}
`

### Beginner Examples

`	ypescript
function wrapInTag(content: string, tag?: string): string {
  const tagName = tag ?? "div";
  return \<\>\</\>\;
}

function calculatePrice(basePrice: number, taxRate: number = 0.1): number {
  return basePrice * (1 + taxRate);
}
`

### Intermediate Examples

`	ypescript
function configureServer(
  host: string,
  port?: number,
  ssl?: boolean,
  timeout?: number
) {
  return {
    host,
    port: port ?? 8080,
    ssl: ssl ?? true,
    timeout: timeout ?? 30000,
  };
}

function formatDate(date: Date, format?: "short" | "long"): string {
  if (format === "short") {
    return date.toLocaleDateString();
  }
  return date.toLocaleDateString(undefined, {
    weekday: "long", year: "numeric", month: "long", day: "numeric",
  });
}
`

### Advanced Examples

`	ypescript
function createCache<TKey, TValue>(
  initialCapacity?: number,
  evictionPolicy: "lru" | "fifo" | "none" = "lru"
): Map<TKey, TValue> {
  return new Map();
}

function getOrCreate<T>(
  key: string,
  factory: () => T,
  cache?: Map<string, T>
): T {
  const targetCache = cache ?? new Map();
  if (targetCache.has(key)) {
    return targetCache.get(key)!;
  }
  const value = factory();
  targetCache.set(key, value);
  return value;
}
`

### Real-World Use Cases

**Library APIs**: Optional parameters make APIs easy to use with sensible defaults.

**API Clients**: HTTP clients benefit from optional parameters for headers and timeouts.

### Common Mistakes

**Placing Required After Optional**: Optional parameters must come after required ones.

**Not Handling undefined Explicitly**: Default parameters only trigger on undefined, not null.

### Best Practices

1. Place optional parameters after required ones
2. Use default parameters instead of manual || checks
3. Document default values clearly

### Performance Considerations

Default parameters are evaluated at call time. For expensive defaults, use lazy evaluation.

### Interview Questions

1. How do default parameters differ from optional parameters in the type system?
2. What value triggers a default parameter?

### Coding Challenges

1. Create a function with three optional parameters with sensible defaults

### Related Topics

- Optional Chaining
- Nullish Coalescing

## Function overloads

### What It Is

Function overloads allow multiple type signatures for a single function implementation. Each signature describes a different calling convention with different parameter and return types.

### Why It Is Important

Overloads provide precise typing for functions that accept varied argument patterns. They allow library authors to create expressive APIs where the return type depends on the argument types.

### How It Works Internally

TypeScript collects all overload signatures and checks that the implementation signature is compatible with all of them. When the function is called, TypeScript evaluates each overload signature in order and picks the first matching one.

### Syntax

`	ypescript
function process(value: string): string;
function process(value: number): number;
function process(value: string | number): string | number {
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  return value * 2;
}
`

### Beginner Examples

`	ypescript
function greet(person: string): string;
function greet(persons: string[]): string[];
function greet(person: string | string[]): string | string[] {
  if (Array.isArray(person)) {
    return person.map((p) => \Hello, \!\);
  }
  return \Hello, \!\;
}

console.log(greet("Alice"));
console.log(greet(["Alice", "Bob"]));
`

### Intermediate Examples

`	ypescript
interface User { id: number; name: string }
interface Admin { id: number; name: string; role: string }

function findUser(id: number): User | undefined;
function findUser(email: string): User | undefined;
function findUser(idOrEmail: number | string): User | undefined {
  if (typeof idOrEmail === "number") {
    return { id: idOrEmail, name: "User" };
  }
  return { id: 1, name: "User" };
}
`

### Advanced Examples

`	ypescript
type EventHandler = {
  (event: "click", handler: (x: number, y: number) => void): void;
  (event: "keypress", handler: (key: string) => void): void;
  (event: string, handler: (...args: any[]) => void): void;
};

const on: EventHandler = (event: string, handler: (...args: any[]) => void) => {
  console.log(\Registering handler for \\);
};

on("click", (x, y) => console.log(x, y));
on("keypress", (key) => console.log(key));
`

### Real-World Use Cases

**DOM APIs**: document.getElementById returns different types based on context.

**Redux Dispatch**: Dispatch has overloads for different action creators.

### Common Mistakes

**Incorrect Implementation Signature**: The implementation must be compatible with all overloads.

**Wrong Overload Order**: More specific overloads should come before more general ones.

### Best Practices

1. Place more specific overloads first
2. Keep overloads simple — prefer union types when possible
3. Use overloads for different argument patterns, not different return types

### Performance Considerations

Overloads are compile-time only — no runtime overhead.

### Interview Questions

1. How does TypeScript resolve which overload signature to use?
2. When would you use overloads instead of union types?

### Coding Challenges

1. Create a function with three overloads for string, number, and boolean inputs

### Related Topics

- Function Signatures
- Callable Types
- Union Types vs Overloads
