# Type Conversion - Explicit vs implicit, String(), Number(), Boolean(), parseInt/parseFloat

## Introduction
Type conversion is the process of converting a value from one data type to another. JavaScript is a dynamically typed language with both explicit (intentional) and implicit (automatic) type conversion. Understanding both is crucial for writing predictable code.

## Explicit conversion

### What It Is
Explicit conversion, also called type casting, is when the programmer intentionally converts a value from one type to another using built-in functions or operators.

### Why It Is Important
Explicit conversion makes code intent clear and prevents bugs from unexpected implicit coercion. It is essential for input validation, data serialization, and type-safe operations.

### How It Works Internally
Functions like `String()`, `Number()`, `Boolean()` call the internal ToString, ToNumber, and ToBoolean abstract operations respectively. These operations follow the ECMAScript specification algorithms for type conversion.

### Syntax
```javascript
// String conversion
String(value);        // ToString abstract operation
value.toString();     // Object.prototype.toString (not for null/undefined)
value + '';           // Unary concatenation (implicit, but often used explicitly)

// Number conversion
Number(value);        // ToNumber abstract operation
+value;               // Unary plus (common explicit pattern)
value - 0;            // Subtraction (less common)

// Boolean conversion
Boolean(value);       // ToBoolean abstract operation
!!value;              // Double NOT (common explicit pattern)
```

### Beginner Examples
```javascript
// To String
console.log(String(42));          // "42"
console.log(String(true));        // "true"
console.log(String(null));        // "null"
console.log(String(undefined));   // "undefined"
console.log(String([1, 2, 3]));   // "1,2,3"
console.log(String({ a: 1 }));    // "[object Object]"

// To Number
console.log(Number("42"));        // 42
console.log(Number("3.14"));      // 3.14
console.log(Number(""));          // 0
console.log(Number("hello"));     // NaN
console.log(Number(true));        // 1
console.log(Number(false));       // 0
console.log(Number(null));        // 0
console.log(Number(undefined));   // NaN

// To Boolean
console.log(Boolean("hello"));    // true
console.log(Boolean(""));         // false
console.log(Boolean(42));         // true
console.log(Boolean(0));          // false
console.log(Boolean(null));       // false
console.log(Boolean(undefined));  // false
console.log(Boolean([]));         // true
console.log(Boolean({}));         // true
```

### Intermediate Examples
```javascript
// Unary plus for number conversion
console.log(+"42");           // 42
console.log(+"3.14");         // 3.14
console.log(+true);           // 1
console.log(+false);          // 0
console.log(+null);           // 0
console.log(+undefined);      // NaN
console.log(+"123abc");       // NaN

// Double NOT for boolean conversion
console.log(!!"hello");       // true
console.log(!!0);             // false
console.log(!!"");            // false
console.log(!![]);            // true
console.log(!!{});            // true
console.log(!!null);          // false
console.log(!!NaN);           // false

// toString() method
console.log((42).toString());         // "42"
console.log((255).toString(16));      // "ff"
console.log((true).toString());       // "true"
console.log([1, 2, 3].toString());    // "1,2,3"
// console.log(null.toString());      // TypeError!
// console.log(undefined.toString()); // TypeError!

// Chaining conversions
const input = "42";
const num = Number(input);
const result = Boolean(num);
console.log(result); // true
```

### Advanced Examples
```javascript
// Custom toString/ valueOf
const customObj = {
  toString() { return "custom string"; },
  valueOf() { return 42; }
};
console.log(String(customObj));  // "custom string"
console.log(Number(customObj));  // 42
console.log(Boolean(customObj)); // true

// Symbol to Primitive
const user = {
  name: "Alice",
  age: 30,
  [Symbol.toPrimitive](hint) {
    if (hint === "string") return this.name;
    if (hint === "number") return this.age;
    return null;
  }
};
console.log(String(user)); // "Alice"
console.log(Number(user)); // 30

// Safe string conversion function
function safeString(value) {
  if (value === null || value === undefined) return "";
  if (typeof value === "object" && !Array.isArray(value)) {
    try {
      return JSON.stringify(value);
    } catch {
      return Object.prototype.toString.call(value);
    }
  }
  return String(value);
}

// Conversion in array methods
["1", "2", "3"].map(Number); // [1, 2, 3]
["1", "2", "3"].map(Boolean); // [true, true, true]
```

### Real-World Use Cases
- Parsing form input (strings to numbers)
- Displaying numbers as formatted strings
- Serializing data for API calls
- Type validation/assertion in utility functions
- Converting between JSON and JavaScript objects
- Normalizing input from various sources
- Preparing data for comparison operations

### Common Mistakes
- Using `toString()` on null/undefined (throws TypeError)
- Assuming `Number("")` returns NaN (returns 0)
- Using `Boolean("false")` thinking it returns false (returns true)
- Forgetting `Number(" 42 ")` works but `Number("42 ").trim()` is wrong
- Not handling NaN results from Number conversion

### Best Practices
- Use `String()` instead of `.toString()` for null-safe conversion
- Use `Number()` over `parseInt()` for general numeric conversion
- Use `Boolean()` or `!!` for explicit boolean conversion
- Use `JSON.stringify()` for object serialization
- Validate conversions with `Number.isNaN()` after `Number()`
- Prefer explicit conversion over relying on implicit coercion

### Performance Considerations
- `String()` and `Number()` are similarly fast
- `+value` (unary plus) is slightly faster than `Number(value)`
- `!!value` is slightly faster than `Boolean(value)`
- Custom `toString()`/`valueOf()` add function call overhead
- Repeated conversions of the same value should be cached

### Interview Questions
1. What is the difference between explicit and implicit conversion?
2. How do you convert a string to a number?
3. How do you convert a value to a boolean?
4. What happens when you call `toString()` on null?
5. What is the `Symbol.toPrimitive` method?

### Coding Challenges
1. Write a function that safely converts any value to a string.
2. Create a function that converts a string to a number with error handling.
3. Implement a function that normalizes input types for a calculation.
4. Write a JSON-safe serialization function that handles all types.

### Related Topics
- Implicit coercion
- String(), Number(), Boolean() functions
- parseInt/parseFloat

## Implicit coercion

### What It Is
Implicit coercion occurs when JavaScript automatically converts types during evaluation of expressions based on operator context, comparison, or conditional context.

### Why It Is Important
Implicit coercion is pervasive in JavaScript. Understanding it prevents bugs, enables certain programming patterns, and helps read and maintain existing code.

### How It Works Internally
The JavaScript engine uses internal abstract operations (ToPrimitive, ToNumber, ToString, ToBoolean) based on the operator and operand types. The rules are deterministic and defined in the ECMAScript specification.

### Syntax
```javascript
// No explicit syntax - coercion happens automatically in contexts like:
// - Arithmetic operators (except + with strings)
// - Comparison operators (==)
// - Conditional contexts (if, while, &&, ||, !)
// - Template literals
```

### Beginner Examples
```javascript
// Arithmetic coercion
console.log("5" - 2);     // 3 (string to number)
console.log("5" * "2");   // 10 (strings to numbers)
console.log("10" / 2);    // 5
console.log("5" % "2");   // 1

// The + operator ambiguity
console.log("5" + 2);     // "52" (number to string, concatenation)
console.log(2 + "5");     // "25"
console.log(2 + 2 + "5"); // "45" (left to right: 4 + "5")
console.log("5" + 2 + 2); // "522" ("52" + 2)

// Comparison coercion
console.log(5 == "5");      // true
console.log(0 == false);    // true
console.log("" == false);   // true
console.log(null == undefined); // true

// Boolean contexts
if ("hello") console.log("truthy");
while (1) { /* runs once then breaks */ }
console.log(!0);            // true
console.log(!!"false");     // true
```

### Intermediate Examples
```javascript
// Object coercion to primitive
console.log([] + []);       // "" (both become "")
console.log([] + {});       // "[object Object]" ([] → "", {} → "[object Object]")
console.log({} + []);       // 0 or "[object Object]" (context dependent)
console.log({} + {});       // "[object Object][object Object]"

// valueOf vs toString precedence
const obj = {
  valueOf() { return 1; },
  toString() { return "one"; }
};
console.log(obj + 1);   // 2 (valueOf used for addition)
console.log(String(obj)); // "one" (toString used for string)

// NaN in comparisons
console.log(NaN == NaN);   // false
console.log(NaN === NaN);  // false
console.log(NaN < 1);      // false
console.log(NaN > 1);      // false
console.log(NaN != NaN);   // true

// Edge cases
console.log(null > 0);    // false
console.log(null == 0);   // false
console.log(null >= 0);   // true (null coerces to 0 for relational comparison)
console.log(undefined >= 0); // false (undefined coerces to NaN)
```

### Advanced Examples
```javascript
// The Abstract Equality Comparison algorithm
function looseEqual(a, b) {
  // Simplified representation of the == algorithm
  if (a === b) return true;
  if (typeof a !== typeof b) {
    if (a === null || b === null) return a === null && b === null;
    if (typeof a === "string" && typeof b === "number") return looseEqual(Number(a), b);
    if (typeof a === "number" && typeof b === "string") return looseEqual(a, Number(b));
    if (typeof a === "boolean") return looseEqual(Number(a), b);
    if (typeof b === "boolean") return looseEqual(a, Number(b));
    if ((typeof a === "object" || typeof a === "function") && typeof b === "string") {
      return looseEqual(ToPrimitive(a), b);
    }
  }
  return false;
}

// Exploiting coercion for comparison
const defaultSettings = { theme: "dark" };
function getSetting(settings, key, defaultValue) {
  return settings[key] ?? defaultValue;
}
// ?? avoids coercion (null/undefined only)

// Implicit coercion in logical operators
const count = 0;
const display = count || "No items"; // "No items" (0 is falsy)
const display2 = count ?? "No items"; // 0 (correct)

// Date coercion
const date = new Date();
const timestamp = +date; // Same as date.getTime()
console.log(timestamp); // Milliseconds timestamp

// String coercion in template literals
const num = 42;
console.log(`The answer is ${num}`); // "The answer is 42"
```

### Real-World Use Cases
- Loose equality `== null` (checks both null and undefined)
- Default values with `||` operator
- Guard expressions with `&&` operator
- Truthiness checks in conditionals
- String building with implicit concatenation
- Boolean filtering: `array.filter(Boolean)`

### Common Mistakes
- Using `==` when `===` is intended
- Expecting `"" == false` to be false (it's true)
- Not realizing `null >= 0` is true while `null > 0` is false
- Assuming `+` always means addition (could be concatenation)
- Confusing `||` with `??` for default values
- Relying on implicit coercion for complex comparisons

### Best Practices
- Use `===` and `!==` by default
- Use `== null` only for checking both null and undefined
- Use `??` for null/undefined defaults instead of `||`
- Be explicit with type conversions in complex expressions
- Avoid relying on coercion in conditions where many falsy values exist
- Prefer explicit conversion over implicit in function returns

### Performance Considerations
- Implicit coercion has same performance as explicit
- The engine must still perform the same ToNumber/ToString operations
- Repeated implicit coercion in loops can be optimized
- No meaningful performance difference between implicit and explicit

### Interview Questions
1. What is the difference between `==` and `===`?
2. When would you use `==` intentionally?
3. What is the result of `[] == ![]`? (true)
4. Why is `null >= 0` true but `null > 0` false?
5. What values coerce to false in a boolean context?

### Coding Challenges
1. Implement a function that mimics the `==` algorithm.
2. Write a function that safely compares two values after explicit conversion.
3. Create a truthy/falsy checker that demonstrates all edge cases.
4. Implement a function that converts any value to a number with implicit-like behavior.

### Related Topics
- Explicit conversion
- Type coercion in operators
- Truthy and falsy values

## String() and Number() and Boolean()

### What It Is
These are global functions used for explicit type conversion. They convert values to string, number, or boolean primitives respectively.

### Why It Is Important
These functions are the primary tools for explicit type conversion. They provide safe, consistent conversion for all JavaScript types, including null and undefined (which lack `.toString()`).

### How It Works Internally
Each function calls the corresponding abstract operation: `String()` calls ToString, `Number()` calls ToNumber, `Boolean()` calls ToBoolean. These operations are defined in the ECMAScript specification sections 7.1.12, 7.1.3, and 7.1.2.

### Syntax
```javascript
String(value);   // → primitive string
Number(value);   // → primitive number (or NaN)
Boolean(value);  // → primitive boolean
```

### Beginner Examples
```javascript
// String()
String(123);           // "123"
String(true);          // "true"
String([1, 2, 3]);     // "1,2,3"
String({ key: "v" });  // "[object Object]"

// Number()
Number("42");          // 42
Number("3.14");        // 3.14
Number("");            // 0
Number("  ");          // 0
Number("\t\n");        // 0
Number("123abc");      // NaN
Number(true);          // 1
Number(false);         // 0
Number(null);          // 0
Number(undefined);     // NaN

// Boolean()
Boolean(1);            // true
Boolean(0);            // false
Boolean("hello");      // true
Boolean("");           // false
Boolean([]);           // true
Boolean({});           // true
Boolean(null);         // false
Boolean(NaN);          // false
```

### Intermediate Examples
```javascript
// Number() with different bases
Number("0xFF");        // 255 (hex)
Number("0o77");        // 63 (octal)
Number("0b1010");      // 10 (binary)

// Number() coercion edge cases
Number("Infinity");    // Infinity
Number("-Infinity");   // -Infinity
Number("1e10");        // 10000000000
Number("1e-10");       // 0.0000000001

// String() with symbols
const sym = Symbol("desc");
String(sym);           // "Symbol(desc)"
// Cannot use implicit conversion with symbols:
// "" + sym // TypeError

// Boolean() with objects
Boolean(new Boolean(false)); // true (object is truthy)
const falseObj = new Boolean(false);
console.log(falseObj == false); // true (coercion)
console.log(falseObj === false); // false (different types)

// Using as argument to array methods
["1", "2", "3"].map(Number);   // [1, 2, 3]
[0, 1, "", null].filter(Boolean); // [1]
```

### Advanced Examples
```javascript
// Number() vs unary +
console.time("Number");
for (let i = 0; i < 1000000; i++) Number("42");
console.timeEnd("Number");

console.time("unary");
for (let i = 0; i < 1000000; i++) +"42";
console.timeEnd("unary");
// Both similarly fast, unary + slightly ahead

// Safe number conversion
function toNumber(value) {
  const num = Number(value);
  return Number.isNaN(num) ? 0 : num;
}

// String() with BigInt
String(9007199254740991n);     // "9007199254740991"
String(-42n);                  // "-42"

// Custom valueOf/toString interaction
const custom = {
  valueOf() { return "42"; }, // valueOf should return primitive
  toString() { return 100; }
};
console.log(Number(custom)); // 42 (valueOf used, returns "42" → 42)
console.log(String(custom)); // "100" (toString is preferred)

// Round-tripping
const value = 42;
console.log(Number(String(value)) === value); // true (for intergers)
console.log(Number(String(0.1 + 0.2)) === 0.1 + 0.2); // false (precision)
```

### Real-World Use Cases
- Form input processing: `Number(formInput.value)`
- Display formatting: `String(amount)`
- Type validation: `typeof value === "string" ? value : String(value)`
- Boolean flag normalization: `Boolean(flagValue)`
- Array transformation: `items.map(Number)`
- Safe JSON-like serialization

### Common Mistakes
- Using `Number("")` expecting NaN (returns 0)
- Using `new String()` / `new Number()` / `new Boolean()` (creates objects, not primitives)
- Assuming `Number(null)` returns NaN (returns 0)
- Forgetting `String(Symbol())` works but implicit string conversion doesn't
- Using `Boolean()` on `new Boolean(false)` (returns true, it's an object)

### Best Practices
- Use `String()` over `.toString()` for null-safe conversion
- Use `Number()` over `parseInt()` when you want strict numeric conversion
- Use `Boolean()` or `!!` for explicit boolean conversion
- Validate with `Number.isNaN()` after `Number()`
- Prefer `??` over `||` when 0 or "" are valid values
- Never use `new String()`, `new Number()`, or `new Boolean()`

### Performance Considerations
- `String()` is slightly faster than `"" + value` in some engines
- `Number()` and `+value` are comparably fast
- `Boolean()` and `!!value` are comparably fast
- Repeated calls in hot paths should be cached
- No meaningful difference in most real-world scenarios

### Interview Questions
1. What is the difference between `String()` and `.toString()`?
2. When would `Number("")` return 0 instead of NaN?
3. Why does `new Boolean(false)` return a truthy value?
4. How does `Number()` handle hex, octal, and binary strings?
5. Can you convert a Symbol to a string?

### Coding Challenges
1. Write a safe number conversion function that handles edge cases.
2. Create a `toString` polyfill for all types.
3. Implement a function that detects conversion failures.
4. Write a type-normalization function that converts inputs to expected types.

### Related Topics
- Type conversion
- Implicit coercion
- parseInt/parseFloat

## parseInt() and parseFloat()

### What It Is
`parseInt()` and `parseFloat()` are global functions that parse strings and extract numeric values. Unlike `Number()`, they tolerate non-numeric trailing characters.

### Why It Is Important
These functions are useful for extracting numbers from strings with trailing units, user input with extra characters, and formatted numbers. `parseInt` also supports radix conversion.

### How It Works Internally
`parseInt()` reads characters from left to right until it encounters a non-digit character. It then converts the accumulated characters to a number. `parseFloat()` similarly parses floating-point numbers. Both skip leading whitespace.

### Syntax
```javascript
parseInt(string, radix);   // radix is base (2-36), default 10
parseFloat(string);
```

### Beginner Examples
```javascript
// parseInt
console.log(parseInt("42"));         // 42
console.log(parseInt("42px"));       // 42 (stops at 'p')
console.log(parseInt("3.14"));       // 3 (stops at '.')
console.log(parseInt("  42  "));     // 42 (trims whitespace)
console.log(parseInt("abc"));        // NaN

// parseInt with radix
console.log(parseInt("FF", 16));     // 255
console.log(parseInt("77", 8));      // 63
console.log(parseInt("1010", 2));    // 10
console.log(parseInt("10", 10));     // 10

// parseFloat
console.log(parseFloat("3.14"));     // 3.14
console.log(parseFloat("3.14px"));   // 3.14
console.log(parseFloat("3.14.15"));  // 3.14 (stops at second .)
console.log(parseFloat("  1.5e2"));  // 150
console.log(parseFloat("abc"));      // NaN
```

### Intermediate Examples
```javascript
// parseInt edge cases
console.log(parseInt("0xFF"));       // 255 (auto-detects hex)
console.log(parseInt("0o77"));       // 0 (stops at 'o')
console.log(parseInt("0b1010"));     // 0 (stops at 'b')
console.log(parseInt("   +42"));     // 42
console.log(parseInt("   -42"));     // -42
console.log(parseInt("  +-42"));     // NaN

// Always specify radix
console.log(parseInt("010"));        // 10 (ES5+, formerly 8 in some engines)
console.log(parseInt("010", 10));    // 10 (safe)
console.log(parseInt("010", 8));     // 8 (octal)

// parseFloat edge cases
console.log(parseFloat(".5"));       // 0.5
console.log(parseFloat("Infinity")); // Infinity
console.log(parseFloat("1e-5"));     // 0.00001
console.log(parseFloat("1.23e2"));   // 123

// parseInt with leading zeros
console.log(parseInt("00042"));      // 42
console.log(parseInt("00042", 16));  // 66 (hex 42)
```

### Advanced Examples
```javascript
// parseInt for color extraction
function parseHexColor(color) {
  const hex = color.replace("#", "");
  return {
    r: parseInt(hex.substr(0, 2), 16),
    g: parseInt(hex.substr(2, 2), 16),
    b: parseInt(hex.substr(4, 2), 16)
  };
}
console.log(parseHexColor("#FF6600")); // { r: 255, g: 102, b: 0 }

// parseFloat for CSS value extraction
function parseCSSValue(value) {
  const match = value.match(/^(-?\d+\.?\d*)(px|em|rem|%|vh|vw)?$/);
  if (!match) return null;
  return {
    value: parseFloat(match[1]),
    unit: match[2] || "px"
  };
}
console.log(parseCSSValue("42.5px")); // { value: 42.5, unit: "px" }

// parseInt vs Number
function compareParsing(str) {
  return {
    parseInt: parseInt(str),
    parseFloat: parseFloat(str),
    Number: Number(str)
  };
}
console.log(compareParsing("42px"));
// { parseInt: 42, parseFloat: 42, Number: NaN }

// Safe integer extraction
function extractInteger(str) {
  const parsed = parseInt(str, 10);
  return Number.isNaN(parsed) ? null : parsed;
}

// Binary string to number
function parseBinary(binStr) {
  return parseInt(binStr.replace(/[^01]/g, ""), 2);
}
console.log(parseBinary("1010")); // 10
```

### Real-World Use Cases
- Parsing CSS values: `parseInt("42px")` → 42
- Extracting numbers from formatted strings: `"$1,234.56"`
- Parsing user input that may include units
- Color code parsing (hex to RGB)
- Binary/octal/hex string parsing
- Reading configuration values from environment variables
- Extracting numeric IDs from strings

### Common Mistakes
- Forgetting to specify radix 10 (historic issue with leading zeros)
- Using `parseInt` for floating-point values (use `parseFloat`)
- Assuming `parseInt` returns NaN for `"0xFF"` (returns 255, auto-detects hex)
- Using `parseInt` on non-string inputs (coerces to string first)
- Expecting `parseInt` to handle scientific notation (`"1e5"` → 1, not 100000)

### Best Practices
- Always specify radix: `parseInt(string, 10)`
- Use `parseFloat` for decimal numbers
- Use `Number()` when partial parsing is unacceptable
- Validate parsing results with `Number.isNaN()`
- Use `parseInt` for extracting numbers from strings with units
- Use `Number` for strict numeric conversion from user input

### Performance Considerations
- `parseInt` and `parseFloat` are slower than `Number()` for simple strings
- Partial parsing is the trade-off for speed loss
- `parseInt(string, 10)` is optimized in modern engines
- Avoid parsing in hot loops when possible
- Parsing same string multiple times should be cached

### Interview Questions
1. What is the difference between `parseInt` and `Number`?
2. Why should you always provide a radix to `parseInt`?
3. How does `parseInt` handle "0xFF" and "010"?
4. What is the difference between `parseInt` and `parseFloat`?
5. How does `parseFloat` handle scientific notation?

### Coding Challenges
1. Implement a function that extracts a number from a string with units.
2. Write a hex color parser using parseInt.
3. Create a currency string parser ("$1,234.56" → 1234.56).
4. Implement a binary string to number converter using parseInt.
5. Write a function that safely extracts a version number string (e.g., "v2.3.1").

### Related Topics
- Number() function
- String conversion
- Numeric type handling
