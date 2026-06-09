# Data Types - Primitive types, typeof operator, type coercion

## Introduction
JavaScript has dynamic typing, meaning variable types are determined at runtime. The language has seven primitive types and one complex type (Object). Understanding these types, how to check them, and how implicit coercion works is fundamental to writing predictable JavaScript code.

## Primitive types (string, number, boolean, null, undefined, symbol, bigint)

### What It Is
Primitive types are the most basic data types in JavaScript. They are immutable values that are not objects and have no methods. There are seven primitive types: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, and `bigint`.

### Why It Is Important
Primitives are the building blocks of all JavaScript data. Understanding their behavior, immutability, and how they differ from objects prevents subtle bugs related to assignment, comparison, and type coercion.

### How It Works Internally
Primitives are stored directly in memory on the stack (for simple values) or in the heap for large strings. When you access a method on a primitive (e.g., `"hello".toUpperCase()`), JavaScript temporarily wraps the primitive in a corresponding object wrapper (`String`, `Number`, `Boolean`) via "autoboxing," calls the method, then discards the wrapper. Primitives are compared by value, unlike objects which are compared by reference.

### Syntax
```javascript
// String
const str = "Hello";
const str2 = 'World';
const str3 = `Template literal`;

// Number
const int = 42;
const float = 3.14;
const negative = -10;
const notANumber = NaN;
const infinity = Infinity;

// Boolean
const isTrue = true;
const isFalse = false;

// Null
const emptyValue = null;

// Undefined
let notAssigned;
const alsoUndefined = undefined;

// Symbol (ES6)
const sym1 = Symbol('description');
const sym2 = Symbol('description'); // Different from sym1

// BigInt (ES2020)
const big = 9007199254740991n;
const big2 = BigInt("9007199254740991");
```

### Beginner Examples
```javascript
// String examples
let firstName = "Alice";
let lastName = "Johnson";
let fullName = firstName + " " + lastName; // "Alice Johnson"
console.log(fullName.length); // 13

// Number examples
let age = 30;
let price = 19.99;
let total = age + price; // 49.99
console.log(typeof price); // "number"

// Boolean examples
let isLoggedIn = true;
let hasAccess = isLoggedIn && age >= 18; // true

// Null vs undefined
let user = null; // Explicitly empty
let data;        // Undefined (not assigned)
console.log(null == undefined);  // true (loose equality)
console.log(null === undefined); // false (strict equality)

// Checking types
console.log(typeof "hello");        // "string"
console.log(typeof 42);             // "number"
console.log(typeof true);           // "boolean"
console.log(typeof null);           // "object" (historical bug!)
console.log(typeof undefined);      // "undefined"
console.log(typeof Symbol());       // "symbol"
console.log(typeof 42n);            // "bigint"
```

### Intermediate Examples
```javascript
// Number precision issues
console.log(0.1 + 0.2); // 0.30000000000000004
console.log(0.1 + 0.2 === 0.3); // false

// NaN peculiarities
console.log(typeof NaN); // "number"
console.log(NaN === NaN); // false
console.log(isNaN("hello")); // true (confusing)
console.log(Number.isNaN("hello")); // false (correct)

// Symbol uniqueness
const symA = Symbol('id');
const symB = Symbol('id');
console.log(symA === symB); // false
console.log(symA.description); // "id"

// BigInt operations
const big1 = 100n;
const big2 = 200n;
console.log(big1 + big2); // 300n
console.log(big1 * big2); // 20000n
// console.log(big1 + 100); // TypeError: Cannot mix BigInt and other types

// String immutability
let text = "hello";
text[0] = "H"; // No effect
console.log(text); // "hello"
text = "Hello"; // New assignment creates new string

// Autoboxing example
const name = "alice";
console.log(name.toUpperCase()); // "ALICE"
// Behind the scenes: new String(name).toUpperCase()
```

### Advanced Examples
```javascript
// Symbol as object property keys
const USER_ID = Symbol('userId');
const user = {
  name: "Alice",
  [USER_ID]: 12345
};
console.log(user[USER_ID]); // 12345
// Symbols are not enumerated in for...in or Object.keys()
console.log(Object.keys(user)); // ["name"]

// BigInt precision for large numbers
console.log(Number.MAX_SAFE_INTEGER); // 9007199254740991
console.log(9007199254740991 + 1); // 9007199254740992 (correct)
console.log(9007199254740991 + 2); // 9007199254740992 (precision loss!)
console.log(9007199254740991n + 2n); // 9007199254740993n (correct)

// Null vs undefined operations
console.log(Number(null));     // 0
console.log(Number(undefined)); // NaN
console.log(5 + null);        // 5
console.log(5 + undefined);   // NaN

// Checking for null/undefined properly
function greet(name) {
  name = name ?? "Guest"; // Nullish coalescing
  return `Hello, ${name}!`;
}
console.log(greet(null));  // "Hello, Guest!"
console.log(greet());      // "Hello, Guest!"

// typeof quirks with undeclared variables
// console.log(undeclaredVar); // ReferenceError
console.log(typeof undeclaredVar); // "undefined" (doesn't throw)

// Short-circuit with boolean primitives
const config = { debug: true };
const isDebug = config.debug && console.log("Debugging"); // "Debugging"
```

### Real-World Use Cases
- Form validation: checking string lengths, numeric ranges
- API responses: parsing numbers, boolean flags
- Unique object keys with Symbols (internal metadata)
- Large number handling (cryptography, timestamps) with BigInt
- State management: boolean flags for loading/error states
- Optional values: null vs undefined conventions in databases
- Unique identifiers: Symbol for preventing property collision
- Financial calculations: BigInt for precise decimal arithmetic

### Common Mistakes
- Confusing `null` and `undefined`
- Assuming `NaN === NaN` returns true
- Mixing BigInt with regular numbers accidentally
- Expecting `typeof null` to return `"null"`
- Relying on floating-point precision for monetary values
- Using `new String()` / `new Number()` / `new Boolean()` instead of primitives
- Not accounting for `-0` in comparisons (Object.is handles this)

### Best Practices
- Always use primitive literals (`"string"`, `42`) instead of wrapper objects
- Use `===` and `!==` for comparisons
- Use `Number.isNaN()` instead of global `isNaN()`
- Use `Object.is()` for strict equality including `-0` and `NaN`
- Use `??` (nullish coalescing) for default values instead of `||`
- Prefer `BigInt` for financial calculations requiring precision
- Use `typeof` for type checking primitives (except `null`)
- Use `x === null` for explicit null checks

### Performance Considerations
- Primitive operations are faster than object operations
- String concatenation with `+` creates new strings (garbage)
- Autoboxing has minimal overhead but avoid in hot paths
- Number operations are optimized by JIT (SMI - Small Integer optimization)
- BigInt operations are slower than Number operations
- Symbols don't create garbage collection issues
- Property access with Symbol keys has same performance as string keys

### Interview Questions
1. What are the primitive types in JavaScript?
2. Why does `typeof null` return "object"?
3. What is the difference between `null` and `undefined`?
4. How does autoboxing work in JavaScript?
5. What is the purpose of Symbol type?
6. When would you use BigInt?
7. Why is `0.1 + 0.2 !== 0.3`?
8. How do you properly check for NaN?

### Coding Challenges
1. Implement a function that accurately checks if a value is null or undefined.
2. Create a function that safely adds numbers and BigInts without throwing.
3. Write a deep comparison function that handles primitives correctly.
4. Implement a function that counts the number of each primitive type in an array.

### Related Topics
- typeof operator
- Type coercion
- Object wrapper types
- Value vs reference comparison

## typeof operator

### What It Is
The `typeof` operator returns a string indicating the type of the operand. It is the primary way to check primitive types at runtime.

### Why It Is Important
Type checking is essential in a dynamically-typed language. `typeof` helps validate function inputs, handle different data structures, and implement type-safe operations. Understanding its quirks prevents bugs.

### How It Works Internally
The `typeof` operator is evaluated at runtime by the JavaScript engine. It inspects the internal type tag of the value (stored in the value's representation). The engine returns a predetermined string based on the internal type. The well-known `typeof null === "object"` bug exists because values were tagged with type bits, and `null` was represented as a zero pointer (object type tag).

### Syntax
```javascript
typeof operand;
typeof(operand); // Function syntax (same result)

// Returns one of: "undefined", "boolean", "number", "bigint", "string",
// "symbol", "function", "object"
```

### Beginner Examples
```javascript
// Basic type checks
console.log(typeof "hello");      // "string"
console.log(typeof 42);           // "number"
console.log(typeof true);         // "boolean"
console.log(typeof undefined);    // "undefined"
console.log(typeof null);         // "object" (bug)
console.log(typeof Symbol("id")); // "symbol"
console.log(typeof 42n);          // "bigint"

// typeof with variables
let x = 10;
if (typeof x === "number") {
  console.log("x is a number");
}

// typeof with undeclared variables (safe)
if (typeof someGlobal === "undefined") {
  console.log("someGlobal is not defined");
}
```

### Intermediate Examples
```javascript
// typeof with functions
console.log(typeof function(){});         // "function"
console.log(typeof (() => {}));           // "function"
console.log(typeof class A {});           // "function"
console.log(typeof async () => {});       // "function"
console.log(typeof function*(){});        // "function"

// typeof with arrays and objects
console.log(typeof [1, 2, 3]);    // "object"
console.log(typeof {});            // "object"
console.log(typeof new Date());    // "object"
console.log(typeof /regex/);       // "object"

// Checking for arrays correctly
Array.isArray([1, 2, 3]); // true

// typeof with special numbers
console.log(typeof NaN);        // "number"
console.log(typeof Infinity);   // "number"
console.log(typeof -Infinity);  // "number"
console.log(typeof 1/0);        // NaN (typeof (1/0) is "number")
```

### Advanced Examples
```javascript
// Type checking utility function
function getType(value) {
  if (value === null) return "null";
  if (Array.isArray(value)) return "array";
  const baseType = typeof value;
  if (baseType === "object") {
    return Object.prototype.toString.call(value).slice(8, -1).toLowerCase();
  }
  return baseType;
}

console.log(getType(null));           // "null"
console.log(getType([]));             // "array"
console.log(getType(new Date()));     // "date"
console.log(getType(new Map()));      // "map"
console.log(getType(new Set()));      // "set"
console.log(getType(Promise.resolve())); // "promise"

// Object.prototype.toString for precise type detection
const toString = Object.prototype.toString;
console.log(toString.call("hello"));  // "[object String]"
console.log(toString.call(42));       // "[object Number]"
console.log(toString.call(true));     // "[object Boolean]"
console.log(toString.call(null));     // "[object Null]"
console.log(toString.call(undefined));// "[object Undefined]"
console.log(toString.call([1,2]));    // "[object Array]"
```

### Real-World Use Cases
- Input validation in utility functions
- Conditional logic based on argument types
- Safe access to potentially undefined global variables
- Serialization: determining how to encode values
- Dynamic dispatch based on parameter types

### Common Mistakes
- Using `typeof` to check for arrays (returns "object")
- Using `typeof` to check for null (returns "object")
- Assuming `typeof` returns lowercase strings only (it does, but this is a common worry)
- Forgetting that `typeof NaN` returns "number"
- Using `typeof` with `new String()` (returns "object" not "string")

### Best Practices
- Use `typeof` only for primitive type checking
- Use `Array.isArray()` for arrays
- Use `=== null` for null checks
- Use `Object.prototype.toString.call()` for precise object type detection
- Use `typeof` for safe checks of potentially undeclared variables
- Combine `typeof` with `===` for exhaustive type checks

### Performance Considerations
- `typeof` is extremely fast (single instruction in the engine)
- `Object.prototype.toString.call()` is slower but more precise
- In hot paths, avoid complex type-checking chains if possible
- Type narrowing with TypeScript reduces runtime type checks

### Interview Questions
1. What does `typeof null` return and why?
2. How do you check if a value is an array?
3. What is the difference between `typeof` and `instanceof`?
4. How do you check for NaN correctly?
5. What does `typeof` return for a class?

### Coding Challenges
1. Write a robust type-detection function that handles all JavaScript types.
2. Create a function that validates if a value matches a given type string.
3. Implement an `isPrimitive()` function.
4. Write a function that recursively checks all values in an object match expected types.

### Related Topics
- Type coercion
- instanceof operator
- Object.prototype.toString
- Duck typing

## Type coercion

### What It Is
Type coercion is the automatic or implicit conversion of values from one type to another. JavaScript can be loosely typed, meaning it will try to convert values to expected types automatically. Coercion can be implicit (automatic) or explicit (intentional).

### Why It Is Important
Understanding coercion is critical because JavaScript performs it extensively. Unintended coercion causes many subtle bugs. Mastering coercion allows developers to understand the language's behavior, write safer code, and use coercion intentionally for cleaner patterns.

### How It Works Internally
The JavaScript engine follows the Abstract Equality Comparison Algorithm for `==` and the ToPrimitive/ToNumber/ToString abstract operations. When a type mismatch occurs, the engine converts one or both values using internal conversion functions (ToPrimitive, ToNumber, ToString, ToBoolean). These algorithms are defined in the ECMAScript specification and are deterministic.

### Syntax
```javascript
// Explicit coercion (intentional)
String(123);       // "123"
Number("123");     // 123
Boolean(1);        // true

// Implicit coercion (automatic)
"5" - 2;           // 3 (string coerced to number)
"5" + 2;           // "52" (number coerced to string)
!"hello";          // false (string coerced to boolean)
```

### Beginner Examples
```javascript
// String coercion
console.log("The number is " + 42);   // "The number is 42"
console.log("5" + 2);                  // "52" (number → string)
console.log("5" - 2);                  // 3  (string → number)

// Number coercion
console.log("5" * "2");                // 10
console.log("10" / 2);                 // 5
console.log("5" - "3");                // 2
console.log("hello" * 2);              // NaN

// Boolean coercion
console.log(Boolean(""));              // false
console.log(Boolean("hello"));         // true
console.log(Boolean(0));               // false
console.log(Boolean(1));               // true
console.log(Boolean(null));            // false
console.log(Boolean(undefined));       // false
console.log(Boolean([]));              // true (empty array)
console.log(Boolean({}));              // true (empty object)

// Loose equality
console.log(5 == "5");                 // true (coercion)
console.log(0 == false);               // true
console.log("" == false);              // true
console.log(null == undefined);        // true
console.log([] == false);              // true (complex coersion)
```

### Intermediate Examples
```javascript
// The + operator behavior
console.log(1 + 2 + "3");             // "33" (left to right: 3 + "3")
console.log("1" + 2 + 3);             // "123" (left to right: "12" + 3)

// Subtraction, multiplication, division always coerce to numbers
console.log("12" - 4);                // 8
console.log("hello" - 1);             // NaN
console.log("10" * "2.5");            // 25
console.log("100" / "10");            // 10
console.log("5" % "2");               // 1
console.log("10" ** "2");             // 100

// Unary plus/minus for explicit coercion
console.log(+"42");                    // 42
console.log(+"hello");                 // NaN
console.log(-"42");                    // -42

// Comparison with coercion
console.log(1 < "2");                  // true ("2" → 2)
console.log("a" < "b");                // true (lexicographic)
console.log("2" > "12");               // true (lexicographic, "2" > "1")
console.log(null > 0);                 // false
console.log(null == 0);                // false
console.log(null >= 0);                // true (null → 0)
```

### Advanced Examples
```javascript
// Abstract Relational Comparison
console.log({} > []);           // false
console.log({} < []);           // false
console.log({} == []);          // false
console.log({} >= []);          // true (both → NaN, NaN >= NaN is false... wait)

// The infamous coercion table
console.log([] + {});           // "[object Object]" ("" + "[object Object]")
console.log({} + []);           // 0 or "[object Object]" depending on context
console.log([] + []);           // ""
console.log({} + {});           // "[object Object][object Object]"

// Coercion with objects
const obj = {
  valueOf() { return 42; },
  toString() { return "forty-two"; }
};
console.log(obj + 10);          // 52 (uses valueOf)
console.log(String(obj));       // "forty-two" (uses toString by default)

// Date coercion
const date = new Date();
console.log(date + 1);          // "Wed Jun ...1"
console.log(+date);             // 1717000000000 (timestamp)

// ToBoolean conversion table
console.log(Boolean(0));          // false
console.log(Boolean(-0));         // false
console.log(Boolean(NaN));        // false
console.log(Boolean(""));         // false
console.log(Boolean(null));       // false
console.log(Boolean(undefined));  // false
console.log(Boolean(false));      // false
// Everything else is truthy
```

### Real-World Use Cases
- User input parsing (form values are strings, need conversion to numbers)
- API response handling (JSON values need type normalization)
- Conditional rendering with truthy/falsy checks
- String template building with automatic coercion
- Comparison of values from different sources
- Default parameter values using `||` or `??`

### Common Mistakes
- Using `==` instead of `===` and relying on implicit coercion
- Not accounting for `+` behaving as string concatenation
- Assuming empty array `[]` is falsy (it's truthy!)
- Forgetting that `null >= 0` is true but `null > 0` is false
- Using `isNaN()` (which coerces) instead of `Number.isNaN()`

### Best Practices
- Always use `===` and `!==` unless you explicitly need coercion
- Use explicit coercion (`Number()`, `String()`, `Boolean()`) for clarity
- Avoid using `==` with different types
- Use `Number.isNaN()` instead of `isNaN()`
- Use `??` (nullish coalescing) for null/undefined defaults
- Use `||` for falsy defaults (covers null, undefined, 0, "", false)
- Be explicit when comparing potential string numbers

### Performance Considerations
- Explicit coercion is as fast as implicit coercion (same internal operations)
- Repeated coercion in loops can be optimized by engines
- `+value` (unary plus) is the fastest explicit number conversion
- `String(value)` is optimized similarly to `value + ''`
- Cache coerced values when needed multiple times

### Interview Questions
1. What is the difference between implicit and explicit coercion?
2. Why is `0 == false` true in JavaScript?
3. Explain how `+` operator handles mixed types.
4. What is the value of `[] + {}` and `{} + []`?
5. How do `valueOf` and `toString` affect coercion?
6. Why is `null >= 0` true but `null > 0` false?
7. What are the falsy values in JavaScript?

### Coding Challenges
1. Create a function `isFalsy` that identifies all falsy values.
2. Implement a function that safely adds any two values with type checking.
3. Write a function that converts an object to a primitive using custom logic.
4. Create a function `looseEqual` that implements JavaScript's `==` algorithm.
5. Build a deep comparison function that handles type coercion edge cases.

### Related Topics
- Falsy and truthy values
- == vs ===
- valueOf and toString
- NaN and its behavior
