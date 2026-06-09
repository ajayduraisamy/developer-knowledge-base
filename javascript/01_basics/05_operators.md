# Operators - Arithmetic, comparison, logical, bitwise, ternary

## Introduction
Operators are symbols that perform operations on operands. JavaScript provides a rich set of operators for arithmetic, comparison, logical operations, bitwise manipulation, and conditional expressions. Understanding operator precedence, associativity, and behavior is crucial for writing correct expressions.

## Arithmetic operators

### What It Is
Arithmetic operators perform mathematical operations on numeric values. JavaScript supports addition, subtraction, multiplication, division, modulus, exponentiation, increment, and decrement.

### Why It Is Important
Arithmetic operations are fundamental to almost all programming tasks, from simple counters to complex financial calculations. Understanding how JavaScript handles type coercion, overflow, and special values in arithmetic is essential.

### How It Works Internally
Arithmetic operations invoke the internal ToNumber or ToPrimitive abstract operations. The `+` operator is overloaded to also perform string concatenation when either operand is a string. The engine uses the IEEE 754 double-precision format for numbers, which affects precision of decimal operations.

### Syntax
```javascript
// Basic arithmetic
a + b   // Addition
a - b   // Subtraction
a * b   // Multiplication
a / b   // Division
a % b   // Modulus (remainder)
a ** b  // Exponentiation (ES2016)

// Increment/decrement
a++     // Post-increment
++a     // Pre-increment
a--     // Post-decrement
--a     // Pre-decrement

// Unary
+a      // Unary plus (coerces to number)
-a      // Unary negation
```

### Beginner Examples
```javascript
// Basic operations
let sum = 10 + 5;      // 15
let diff = 10 - 5;     // 5
let product = 10 * 5;  // 50
let quotient = 10 / 3; // 3.3333333333333335
let remainder = 10 % 3; // 1
let power = 2 ** 3;    // 8

// Increment and decrement
let count = 0;
count++; // 0 (returns original, then increments)
console.log(count); // 1
++count; // 2 (increments, then returns)
console.log(count); // 2

// String concatenation with +
let greeting = "Hello" + " " + "World"; // "Hello World"

// Coercion in arithmetic
console.log("5" - 2);   // 3
console.log("5" * "2"); // 10
console.log("10" / 2);  // 5
```

### Intermediate Examples
```javascript
// Modulus with negative numbers
console.log(10 % 3);   // 1
console.log(-10 % 3);  // -1 (result has sign of dividend)
console.log(10 % -3);  // 1
console.log(-10 % -3); // -1

// Exponentiation
console.log(2 ** 10);           // 1024
console.log(2 ** -1);           // 0.5
console.log(16 ** 0.5);         // 4 (square root)
console.log((-2) ** 2);         // 4
// console.log(-2 ** 2);        // SyntaxError

// Unary plus for numeric coercion
console.log(+"42");       // 42
console.log(+true);       // 1
console.log(+false);      // 0
console.log(+null);       // 0
console.log(+undefined);  // NaN

// Infinity and -Infinity
console.log(1 / 0);       // Infinity
console.log(-1 / 0);      // -Infinity
console.log(Infinity + 1); // Infinity
```

### Advanced Examples
```javascript
// Operator precedence
console.log(2 + 3 * 4);     // 14 (multiplication first)
console.log((2 + 3) * 4);   // 20 (parentheses override)

// Floating point precision
console.log(0.1 + 0.2);                      // 0.30000000000000004
console.log((0.1 + 0.2).toFixed(1));         // "0.3"
console.log(Math.round((0.1 + 0.2) * 10) / 10); // 0.3

// Assignment chaining
let a, b, c;
a = b = c = 0; // All set to 0

// Compound assignment operators
let x = 10;
x += 5;  // x = 15
x -= 3;  // x = 12
x *= 2;  // x = 24
x /= 4;  // x = 6
x %= 2;  // x = 0
x **= 3; // x = 0

// Arithmetic with Date
const now = new Date();
const tomorrow = new Date(now.getTime() + 86400000);
console.log(tomorrow - now); // 86400000 (milliseconds)
```

### Real-World Use Cases
- Calculating totals and taxes in eCommerce
- Tracking scores in games
- Time calculations (duration, timestamps)
- Statistical operations (average, variance)
- Financial computations (interest, amortization)
- Physics calculations in games or simulations
- Random number generation (using modulo for ranges)

### Common Mistakes
- Forgetting that `+` also concatenates strings (order matters)
- Expecting precise decimal results
- Using increment/decrement in complex expressions (side effects)
- Not accounting for division by zero (returns Infinity, not error)
- Confusing `%` (modulus) with remainder in other languages

### Best Practices
- Use parentheses for complex expressions to improve readability
- Avoid mixing numeric and string types with `+`
- Use `Math.round()`, `toFixed()`, or library (like decimal.js) for precise decimals
- Prefer `++` and `--` as standalone statements, not in larger expressions
- Use `**` over `Math.pow()` for exponentiation (faster, more readable)
- Use `Number.EPSILON` for floating-point comparisons

### Performance Considerations
- Arithmetic on integers (SMIs) is highly optimized by JIT compilers
- `**` operator is faster than `Math.pow()`
- Division is slower than multiplication in most engines
- Repeated arithmetic in hot loops is aggressively optimized by JIT

### Interview Questions
1. What is the difference between `++i` and `i++`?
2. Why does `0.1 + 0.2` not equal `0.3`?
3. What is the result of `"5" + 3` vs `"5" - 3`?
4. How do you safely compare floating-point numbers?
5. What does `NaN === NaN` return?

### Coding Challenges
1. Implement a function `add(a, b)` that handles floating point precision.
2. Write a function that calculates compound interest.
3. Create a simple calculator that parses arithmetic expressions.
4. Implement a `mod` function that always returns a positive remainder.

### Related Topics
- Number type and IEEE 754
- Operator precedence
- Type coercion

## Comparison operators

### What It Is
Comparison operators compare two values and return a boolean result. JavaScript provides equality (`==`, `===`, `!=`, `!==`), relational (`<`, `>`, `<=`, `>=`), and special comparison utilities.

### Why It Is Important
Comparisons control program flow through conditionals and loops. Understanding strict vs loose equality, lexicographic string comparison, and how coercion works in comparisons prevents bugs.

### How It Works Internally
The `===` operator checks both type and value without coercion. The `==` operator performs type coercion before comparison using the Abstract Equality Comparison algorithm. Relational operators use the Abstract Relational Comparison algorithm, which may call `valueOf()` or `toString()` on objects.

### Syntax
```javascript
// Equality
a === b  // Strict equality (recommended)
a !== b  // Strict inequality
a == b   // Loose equality
a != b   // Loose inequality

// Relational
a < b    // Less than
a > b    // Greater than
a <= b   // Less than or equal
a >= b   // Greater than or equal

// Special (ES6+)
Object.is(a, b) // Same-value equality
Number.isNaN(a) // NaN check
```

### Beginner Examples
```javascript
// Strict equality (recommended)
console.log(5 === 5);       // true
console.log(5 === "5");     // false (different types)
console.log(0 === false);   // false
console.log(null === undefined); // false

// Loose equality (avoid)
console.log(5 == "5");      // true
console.log(0 == false);    // true
console.log("" == false);   // true
console.log(null == undefined); // true

// Relational comparisons
console.log(5 < 10);        // true
console.log(10 > 20);       // false
console.log(10 <= 10);      // true
console.log(5 >= 3);        // true

// String comparison (lexicographic)
console.log("apple" < "banana");  // true
console.log("Apple" < "apple");   // true (A=65, a=97)
console.log("2" > "12");          // true (lexicographic, '2' > '1')
```

### Intermediate Examples
```javascript
// NaN comparisons
console.log(NaN === NaN);     // false (NaN is not equal to anything)
console.log(NaN == NaN);      // false
console.log(Object.is(NaN, NaN)); // true
console.log(Number.isNaN(NaN));   // true

// Null/undefined comparisons
console.log(null > 0);       // false
console.log(null == 0);      // false
console.log(null >= 0);      // true (null coerces to 0)
console.log(undefined > 0);  // false
console.log(undefined == 0); // false
console.log(undefined >= 0); // false

// Object comparison (by reference)
let obj1 = { value: 10 };
let obj2 = { value: 10 };
let obj3 = obj1;
console.log(obj1 === obj2); // false (different references)
console.log(obj1 === obj3); // true (same reference)

// Comparing arrays
console.log([] == []);     // false
console.log([] == ![]);    // true (complex coercion!)
```

### Advanced Examples
```javascript
// SameValueZero (used in Map, Set, includes)
console.log(Object.is(0, -0));   // false
console.log(0 === -0);           // true
console.log([1].includes(-0));   // true (SameValueZero)

// Object.is polyfill
function sameValue(x, y) {
  return x === y ? (x !== 0 || 1 / x === 1 / y) : x !== x && y !== y;
}

// Deep comparison utility
function deepEqual(a, b) {
  if (a === b) return true;
  if (typeof a !== typeof b) return false;
  if (typeof a !== 'object' || a === null || b === null) return false;

  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  if (keysA.length !== keysB.length) return false;

  return keysA.every(key =>
    Object.prototype.hasOwnProperty.call(b, key) && deepEqual(a[key], b[key])
  );
}
```

### Real-World Use Cases
- Form validation: checking if inputs match requirements
- Sorting algorithms (custom compare functions)
- Filtering data (age >= 18, price <= 100)
- Search functionality (text matching)
- User authorization (role === 'admin')
- State management (status !== loading)

### Common Mistakes
- Using `==` instead of `===`
- Comparing objects with `===` expecting deep equality
- Assuming string comparison is numeric ("2" > "12" is true lexicographically)
- Not handling `NaN` comparisons correctly
- Forgetting that `null >= 0` is true but `null == 0` is false

### Best Practices
- Always prefer `===` and `!==` over `==` and `!=`
- Use `Object.is()` for `-0` and `NaN` checks
- Use `Number.isNaN()` for NaN checks
- Write explicit type checks before comparison
- Use `Array.some()` or `Array.includes()` for value existence checks
- Implement custom `equals()` methods for complex objects

### Performance Considerations
- `===` is slightly faster than `==` (no coercion step)
- String comparison is O(n) in length
- Object property comparison triggers lookups
- JIT optimizes repeated comparisons of the same shape objects

### Interview Questions
1. What is the difference between `==` and `===`?
2. How do you compare two objects for equality?
3. Why does `NaN !== NaN`?
4. What is the difference between `Object.is()` and `===`?
5. How does JavaScript compare strings?

### Coding Challenges
1. Implement a deep comparison function.
2. Write a function `looseEqual` that mimics `==` algorithm.
3. Create a sorting function with a custom comparator.
4. Implement `SameValueZero` comparison.

### Related Topics
- Type coercion
- Object equality
- Sorting algorithms

## Logical operators

### What It Is
Logical operators combine or invert boolean expressions. JavaScript has AND (`&&`), OR (`||`), NULLISH COALESCING (`??`), and NOT (`!`). They support short-circuit evaluation and return operand values (not necessarily booleans).

### Why It Is Important
Logical operators are essential for conditional logic, control flow, and concise expressions. Short-circuit evaluation enables patterns like default values, conditional execution, and guards.

### How It Works Internally
The engine evaluates left to right. For `&&`, if the left operand is falsy, it returns the left operand without evaluating the right (short-circuit). For `||`, if the left operand is truthy, it returns it without evaluating the right. `??` checks for null/undefined specifically. `!` coerces the operand to boolean and negates it.

### Syntax
```javascript
a && b    // Logical AND
a || b    // Logical OR
a ?? b    // Nullish coalescing (ES2020)
!a        // Logical NOT
!!a       // Coerce to boolean

// Logical assignment operators (ES2021)
a &&= b   // a && (a = b)
a ||= b   // a || (a = b)
a ??= b   // a ?? (a = b)
```

### Beginner Examples
```javascript
// AND (&&) - returns first falsy value or last truthy value
console.log(true && true);      // true
console.log(true && false);     // false
console.log(1 && 2 && 3);       // 3 (all truthy)
console.log(1 && 0 && 3);       // 0 (first falsy)

// OR (||) - returns first truthy value or last falsy value
console.log(true || false);     // true
console.log(false || true);     // true
console.log(null || 0 || 1);    // 1 (first truthy)
console.log(null || 0 || "");   // "" (all falsy, last)

// NOT (!)
console.log(!true);             // false
console.log(!false);            // true
console.log(!"hello");          // false (truthy string)
console.log(!0);                // true (falsy)
console.log(!!"hello");         // true (double NOT coerces to boolean)

// Short-circuit in conditionals
let user = { name: "Alice" };
if (user && user.name) {
  console.log(user.name); // "Alice"
}
```

### Intermediate Examples
```javascript
// Default values with ||
let name = input || "default";

// Guard operator (&&)
let user = getCurrentUser();
let name = user && user.profile && user.profile.name;

// Nullish coalescing (only null/undefined, not falsy)
let count = 0;
console.log(count || 10);     // 10 (0 is falsy)
console.log(count ?? 10);     // 0 (0 is not null/undefined)

let name = "";
console.log(name || "default"); // "default"
console.log(name ?? "default"); // ""

// Chaining logical operators
let result = (a && b) || c ?? d; // Note: ?? has lower precedence than && and ||

// Logical assignment
let config = {};
config.timeout ??= 3000;     // config.timeout = 3000 (if null/undefined)
config.retries ||= 3;        // config.retries = 3 (if falsy)
config.debug &&= false;      // config.debug = false (if truthy)
```

### Advanced Examples
```javascript
// Short-circuit function calls
function logError(err) {
  console.error(err);
}
const debug = false;
debug && logError("Error occurred"); // Won't execute

// Conditional rendering pattern (React)
function Welcome({ user }) {
  return (
    <div>
      {user && <h1>Welcome, {user.name}!</h1>}
      {user ?? <p>Please log in</p>}
    </div>
  );
}

// Multiple defaults
function createConfig(options) {
  return {
    host: options.host ?? "localhost",
    port: options.port ?? 3000,
    ssl: options.ssl ?? false,
    timeout: options.timeout ?? 5000
  };
}

// Complex guard chain
function getAddress(user) {
  return user?.profile?.address ?? "Address not provided";
}

// De Morgan's laws
if (!(a && b)) { /* same as !a || !b */ }
if (!(a || b)) { /* same as !a && !b */ }
```

### Real-World Use Cases
- Conditional rendering in React/Vue (&& for display, || for fallback)
- Default parameter values
- Optional chaining with nullish coalescing
- Permission checks (user && user.isAdmin)
- Form validation (errors.length && showErrors)
- Event handler guards (evt && evt.preventDefault())
- Feature flags (featureEnabled && renderComponent())

### Common Mistakes
- Confusing `??` with `||` (different falsy handling)
- Using `&&` for default values instead of `||` or `??`
- Forgetting that `0` and `""` are valid values (use `??`)
- Chaining too many logical operators without parentheses
- Assuming logical operators always return boolean

### Best Practices
- Use `??` for default values where `null`/`undefined` are the only "empty" cases
- Use `||` for fallbacks where all falsy values should trigger the default
- Use `&&` for guards (conditional execution)
- Use parentheses to clarify complex logical expressions
- Prefer optional chaining (`?.`) over long `&&` chains
- Use `!!` explicitly when a boolean return type is needed

### Performance Considerations
- Short-circuit evaluation prevents unnecessary computation
- JIT engines optimize predictable branch patterns
- Deeply nested `&&` chains can be less readable (use optional chaining)
- Logical operators are extremely fast (single CPU instructions)

### Interview Questions
1. How does short-circuit evaluation work with `&&` and `||`?
2. What is the difference between `||` and `??`?
3. What do `&&`, `||`, and `??` actually return?
4. What are logical assignment operators?
5. Explain De Morgan's laws in JavaScript.

### Coding Challenges
1. Implement a function that safely accesses nested object properties using `&&`.
2. Write a function that merges two config objects with defaults using `??`.
3. Create a validation function using logical operators.
4. Implement `??` operator behavior without using it.

### Related Topics
- Truthy and falsy values
- Short-circuit evaluation
- Optional chaining

## Bitwise operators

### What It Is
Bitwise operators perform operations on binary representations of numbers. JavaScript supports AND (`&`), OR (`|`), XOR (`^`), NOT (`~`), left shift (`<<`), right shift (`>>`), and zero-fill right shift (`>>>`).

### Why It Is Important
Bitwise operations are useful for low-level programming, flags/bitmasks, performance optimization, and certain algorithms (permissions, graphics, cryptography). They operate on 32-bit signed integers.

### How It Works Internally
Operands are converted to 32-bit signed integers using ToInt32 abstract operation. The operation is performed on each bit position. The result is converted back to a 64-bit floating-point number. Bitwise operators are among the fastest operations at the CPU level.

### Syntax
```javascript
a & b   // Bitwise AND
a | b   // Bitwise OR
a ^ b   // Bitwise XOR
~a      // Bitwise NOT
a << b  // Left shift
a >> b  // Sign-propagating right shift
a >>> b // Zero-fill right shift
```

### Beginner Examples
```javascript
// Bitwise AND (&) - bits are 1 if both are 1
console.log(5 & 3);  // 1  (101 & 011 = 001)

// Bitwise OR (|) - bits are 1 if either is 1
console.log(5 | 3);  // 7  (101 | 011 = 111)

// Bitwise XOR (^) - bits are 1 if different
console.log(5 ^ 3);  // 6  (101 ^ 011 = 110)

// Bitwise NOT (~) - inverts all bits
console.log(~5);     // -6

// Left shift (<<)
console.log(5 << 2); // 20 (101 << 2 = 10100)

// Right shift (>>)
console.log(20 >> 2);// 5  (10100 >> 2 = 101)

// Zero-fill right shift (>>>)
console.log(-1 >>> 0); // 4294967295 (unsigned 32-bit max)
```

### Intermediate Examples
```javascript
// Check if number is even
function isEven(n) {
  return (n & 1) === 0;
}
console.log(isEven(4)); // true
console.log(isEven(7)); // false

// Power of 2 check
function isPowerOf2(n) {
  return n > 0 && (n & (n - 1)) === 0;
}
console.log(isPowerOf2(8));  // true
console.log(isPowerOf2(10)); // false

// Swap without temp variable
let a = 5, b = 3;
a ^= b;
b ^= a;
a ^= b;
console.log(a, b); // 3, 5

// Bitmask flags
const READ = 1;   // 001
const WRITE = 2;  // 010
const EXECUTE = 4;// 100

let permission = READ | WRITE; // 011 = 3
console.log(permission & READ);    // 1 (true)
console.log(permission & EXECUTE); // 0 (false)

// Integer rounding using bitwise OR
console.log(5.9 | 0);   // 5
console.log(-5.9 | 0);  // -5
console.log(5.9 >>> 0); // 5
```

### Advanced Examples
```javascript
// Count bits set (population count)
function popcount(n) {
  n = n - ((n >>> 1) & 0x55555555);
  n = (n & 0x33333333) + ((n >>> 2) & 0x33333333);
  return ((n + (n >>> 4) & 0xF0F0F0F) * 0x1010101) >>> 24;
}
console.log(popcount(7));  // 3 (111)

// Find the rightmost set bit
function lowestSetBit(n) {
  return n & -n;
}
console.log(lowestSetBit(12)); // 4 (1100 -> 0100)

// Color extraction from hex
const color = 0x3366CC;
const red = (color >> 16) & 0xFF;   // 0x33 = 51
const green = (color >> 8) & 0xFF;  // 0x66 = 102
const blue = color & 0xFF;          // 0xCC = 204

// Check if only one bit is set
function isSingleBitSet(n) {
  return n > 0 && (n & (n - 1)) === 0;
}

// Flipping bits for complement
const FLAGS = 0b1010;
const toggled = FLAGS ^ 0b1100; // 0b0110
```

### Real-World Use Cases
- Permission systems (read/write/execute flags)
- Graphics and color manipulation
- Low-level protocol parsing
- Hashing algorithms
- Cryptography primitives
- Game development (tile maps, collision masks)
- Performance-critical code paths
- Network packet encoding/decoding

### Common Mistakes
- Forgetting bitwise ops work on 32-bit signed integers
- Using `>>` when `>>>` is needed for unsigned numbers
- Expecting bitwise operations on non-integer values
- Confusing `~` (bitwise NOT) with `!` (logical NOT)
- Not accounting for sign bit when shifting negative numbers

### Best Practices
- Use descriptive names for bitmask constants
- Prefer readability over clever bit tricks in most code
- Use `>>> 0` to ensure unsigned 32-bit conversion
- Document bitwise operations with comments explaining the intent
- Use `Number()` or `Math.floor()` for clarity if not performance-critical

### Performance Considerations
- Bitwise ops are extremely fast (CPU-level instructions)
- Combining flags into a single integer saves memory
- 32-bit truncation can cause unexpected results with large numbers
- Modern JIT compilers already optimize common patterns

### Interview Questions
1. How do you check if a number is a power of 2 using bitwise ops?
2. What is the difference between `>>` and `>>>`?
3. How do you extract RGB values from a hex color?
4. What is the result of `~5` and why?
5. How do bitmasks work for permissions?

### Coding Challenges
1. Implement addition without using `+` operator (use bitwise).
2. Count the number of 1 bits in a number.
3. Write a function that finds the only non-repeating element in an array where every other element repeats twice.
4. Implement a bitmask permission system.
5. Reverse the bits of a 32-bit integer.

### Related Topics
- Binary number representation
- Two's complement
- 32-bit signed integers

## Ternary operator

### What It Is
The ternary operator (`condition ? exprIfTrue : exprIfFalse`) is a conditional operator that takes three operands. It is the only JavaScript operator that takes three operands.

### Why It Is Important
The ternary operator provides a concise way to write conditional expressions. It is more readable than if/else for simple assignments and is commonly used in JSX, variable assignments, and return statements.

### How It Works Internally
The engine evaluates the condition. If truthy, it evaluates and returns the first expression; otherwise, it evaluates and returns the second. The non-selected branch is never evaluated (short-circuit). The result is used in the surrounding expression.

### Syntax
```javascript
condition ? exprIfTrue : exprIfFalse;

// Nested ternary (use sparingly)
condition1 ? value1
  : condition2 ? value2
  : condition3 ? value3
  : defaultValue;
```

### Beginner Examples
```javascript
// Basic ternary
let age = 20;
let status = age >= 18 ? "Adult" : "Minor";
console.log(status); // "Adult"

// VS if/else
let result;
if (age >= 18) {
  result = "Adult";
} else {
  result = "Minor";
}

// Assigning values
let greeting = isLoggedIn ? `Welcome, ${user}!` : "Please log in";

// Conditional returns
function canAccess(role) {
  return role === "admin" ? true : false;
}

// Inline usage
console.log("Access " + (permission ? "granted" : "denied"));
```

### Intermediate Examples
```javascript
// Nested ternary (use with caution)
let score = 85;
let grade = score >= 90 ? "A"
  : score >= 80 ? "B"
  : score >= 70 ? "C"
  : score >= 60 ? "D"
  : "F";

// Ternary with function calls
let result = isValid(input)
  ? processValid(input)
  : handleError(input);

// Ternary for JSX (React)
return (
  <div>
    {isLoading ? <Spinner /> : <Content data={data} />}
  </div>
);

// Multiple operations (use parentheses or comma operator)
let processed = condition
  ? (prepare(), transform(), finalize())
  : (reset(), getDefault());

// Ternary with template literal
let message = `User ${user.name} is ${user.age >= 18 ? "allowed" : "not allowed"} to vote.`;
```

### Advanced Examples
```javascript
// Ternary for property assignment
const config = {
  url: isDev ? "http://localhost:3000" : "https://api.example.com",
  timeout: isDev ? 10000 : 3000,
  ssl: !isDev
};

// Functional ternary (factory pattern)
const handler = isMobile
  ? handleMobileTouch
  : handleDesktopClick;

// Memoized ternary
const connection = useMemo(() =>
  useSSL ? createSSLConnection() : createConnection(),
  [useSSL]
);

// Curried ternary
const getDiscount = (type) =>
  type === "premium" ? 0.2
    : type === "gold" ? 0.15
    : type === "silver" ? 0.1
    : 0;

// Ternary for error handling
function parseJSON(str) {
  try {
    return JSON.parse(str);
  } catch (e) {
    return null;
  }
}
// vs
const parsed = (() => { try { return JSON.parse(str); } catch { return null; } })();
```

### Real-World Use Cases
- Conditional rendering in React/Angular/Vue
- Setting default values
- Toggling CSS classes
- Conditional routing
- Permission-based UI changes
- Form field validation feedback
- Loading/error/success states
- Theme switching (dark/light mode)

### Common Mistakes
- Overusing nested ternaries (readability suffers)
- Using ternary for side effects instead of if/else
- Forgetting parentheses when mixing with other operators
- Using ternary when a simple `&&` or `||` would suffice
- Nesting ternaries beyond 2 levels

### Best Practices
- Keep ternaries simple (one condition, two clear outcomes)
- Avoid nesting ternaries (use if/else or switch for complex logic)
- Use parentheses to clarify operator precedence
- Prefer if/else for statements, ternary for expressions
- Limit to a single ternary per expression
- Consider extracting to a function if the ternary becomes complex
- Use ternaries in JSX for conditional rendering (React convention)

### Performance Considerations
- Ternary operator performance is identical to equivalent if/else
- Both compile to similar bytecode
- Modern engines optimize both equally well
- Premature optimization of conditional logic is not necessary

### Interview Questions
1. What is the ternary operator and when should you use it?
2. What are the risks of nested ternary operators?
3. How does the ternary operator compare to if/else?
4. Can you use the ternary operator for multiple conditions?

### Coding Challenges
1. Rewrite a nested if/else chain as nested ternaries.
2. Convert a ternary expression to an equivalent if/else statement.
3. Write a function that uses the ternary operator to return the larger of two numbers.
4. Implement a `clamp` function using the ternary operator.

### Related Topics
- Conditional statements (if/else)
- Short-circuit evaluation
- Logical operators
- Operator precedence
