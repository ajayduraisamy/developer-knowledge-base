# Strings - String() constructor, methods, template literals, escaping

## Introduction
Strings represent sequences of characters in JavaScript. They are primitive values that are immutable, meaning once created, they cannot be changed. JavaScript provides extensive built-in methods for string manipulation, as well as template literals for enhanced string formatting.

## String() constructor

### What It Is
`String()` is a built-in constructor that creates string primitives. When called as a function, it converts values to strings. When called as a constructor with `new`, it creates a String wrapper object.

### Why It Is Important
Understanding String conversion is essential for type conversion, serialization, and safe string handling. The `String()` function is the preferred way to explicitly convert values to strings.

### How It Works Internally
`String()` calls the internal ToString abstract operation. For primitives, it returns the string representation. For objects, it calls `toString()` if available, then `valueOf()`. The `new String()` constructor creates a String object wrapper (rarely used).

### Syntax
```javascript
// As a function (preferred)
String(value);

// As a constructor (avoid)
new String(value);

// Examples
String(42);         // "42"
String(true);       // "true"
String(null);       // "null"
String(undefined);  // "undefined"
String([1, 2, 3]);  // "1,2,3"
String({a: 1});     // "[object Object]"
```

### Beginner Examples
```javascript
// Converting numbers
console.log(String(42));           // "42"
console.log(String(3.14));         // "3.14"
console.log(String(Infinity));     // "Infinity"
console.log(String(NaN));          // "NaN"

// Converting booleans
console.log(String(true));         // "true"
console.log(String(false));        // "false"

// Converting null/undefined
console.log(String(null));         // "null"
console.log(String(undefined));    // "undefined"

// Converting arrays
console.log(String([1, 2, 3]));    // "1,2,3"
console.log(String([]));           // ""

// Converting objects
console.log(String({}));           // "[object Object]"
console.log(String({a: 1}));       // "[object Object]"

// Using + operator (implicit conversion)
console.log("" + 42);              // "42"
```

### Intermediate Examples
```javascript
// Custom toString method
const user = {
  name: "Alice",
  age: 30,
  toString() {
    return `${this.name} (${this.age})`;
  }
};
console.log(String(user)); // "Alice (30)"

// String wrapper object (avoid)
const strObj = new String("Hello");
console.log(typeof strObj);        // "object"
console.log(typeof "Hello");       // "string"
console.log(strObj === "Hello");   // false
console.log(String(strObj));       // "Hello"

// String() vs toString()
const num = 123;
console.log(num.toString());       // "123"
console.log(String(num));          // "123"

// null and undefined don't have toString()
// console.log(null.toString());   // TypeError
console.log(String(null));         // "null" (safe)

// Array-like index access on strings
const str = new String("abc");
console.log(str[0]);               // "a"
console.log(str.length);           // 3
```

### Advanced Examples
```javascript
// Symbol conversion
const sym = Symbol("description");
console.log(String(sym));          // "Symbol(description)"
// console.log(sym.toString());    // "Symbol(description)"
// console.log("" + sym);          // TypeError: Cannot convert symbol to string

// BigInt conversion
console.log(String(9007199254740991n)); // "9007199254740991"
console.log(String(-42n));              // "-42"

// String() for safe JSON stringification
function safeStringify(obj) {
  return JSON.stringify(obj, (key, value) => {
    if (typeof value === "bigint") return String(value);
    return value;
  });
}

// Overriding @@toPrimitive
const custom = {
  [Symbol.toPrimitive](hint) {
    if (hint === "string") return "custom string";
    return 42;
  }
};
console.log(String(custom)); // "custom string"
```

### Real-World Use Cases
- Converting user input (always strings from forms)
- Safe serialization for logging
- Type normalization in API handlers
- Generating display text from various data types
- Template text generation for reports/documents

### Common Mistakes
- Using `new String()` instead of `String()` (creates object, not primitive)
- Assuming `.toString()` works on null/undefined
- Forgetting that Symbol can't be implicitly converted to string
- Expecting String() to deeply serialize objects

### Best Practices
- Use `String()` for explicit type conversion
- Prefer template literals for string building
- Never use `new String()` constructor
- Use `toString()` with appropriate radix for numbers
- Override `toString()` on custom objects for meaningful string output

### Performance Considerations
- `String()` is highly optimized
- `new String()` creates unnecessary object wrappers (avoid)
- `"" + value` is similarly fast to `String(value)`
- Template literals have similar performance to concatenation

### Interview Questions
1. What is the difference between `String()` and `new String()`?
2. How does `String()` handle `null` and `undefined`?
3. How does `toString()` on a number work with different radixes?
4. What is the `Symbol.toPrimitive` method?
5. Why should you avoid `new String()`?

### Coding Challenges
1. Implement a `safeString` function that converts any value to a string.
2. Create a custom object with a `toString()` method.
3. Write a function that converts an object's properties to a query string.
4. Implement a `format` function similar to `String.format` in other languages.

### Related Topics
- Type conversion
- toString() method
- Template literals

## String methods

### What It Is
String methods are built-in functions available on all string primitives and objects. They provide operations for searching, extracting, transforming, and inspecting strings.

### Why It Is Important
String manipulation is one of the most common programming tasks. Built-in methods enable efficient text processing without external libraries.

### How It Works Internally
When a method is called on a string primitive, JavaScript wraps it in a temporary String object (autoboxing), calls the method, and discards the wrapper. All methods return new strings (strings are immutable) and do not modify the original string.

### Syntax
```javascript
str.length                    // Property, not method
str.charAt(index)             // Character at index
str.charCodeAt(index)         // Unicode code point at index
str.at(index)                 // Character at index (supports negative)
str.indexOf(substring)        // First occurrence index
str.lastIndexOf(substring)    // Last occurrence index
str.includes(substring)       // Boolean check
str.startsWith(prefix)        // Checks prefix
str.endsWith(suffix)          // Checks suffix
str.slice(start, end)         // Extracts substring
str.substring(start, end)     // Extracts substring
str.substr(start, length)     // Deprecated, extracts substring
str.concat(...strings)        // Concatenation
str.toUpperCase()             // Uppercase
str.toLowerCase()             // Lowercase
str.trim()                    // Remove whitespace both ends
str.trimStart()               // Remove leading whitespace
str.trimEnd()                 // Remove trailing whitespace
str.padStart(targetLen, pad)  // Pad beginning
str.padEnd(targetLen, pad)    // Pad end
str.repeat(count)             // Repeat string
str.replace(search, replace)  // Replace first match
str.replaceAll(search, replace) // Replace all matches
str.split(separator)          // Split into array
str.match(regex)              // Match regex
str.matchAll(regex)           // Match all regex
str.search(regex)             // Search regex
str.localeCompare(other)      // Locale-aware comparison
str.normalize()               // Unicode normalization
```

### Beginner Examples
```javascript
const str = "Hello, World!";

// Length
console.log(str.length); // 13

// Access characters
console.log(str[0]);          // "H" (bracket notation)
console.log(str.charAt(7));   // "W"
console.log(str.at(-1));      // "!" (negative index)

// Search
console.log(str.indexOf("World"));  // 7
console.log(str.indexOf("x"));      // -1
console.log(str.includes("Hello")); // true
console.log(str.startsWith("He"));  // true
console.log(str.endsWith("!"));     // true

// Extract
console.log(str.slice(7, 12));      // "World"
console.log(str.slice(-6));         // "World!"
console.log(str.substring(7, 12));  // "World"

// Transform
console.log(str.toUpperCase());   // "HELLO, WORLD!"
console.log(str.toLowerCase());   // "hello, world!"
console.log("  hi  ".trim());     // "hi"
```

### Intermediate Examples
```javascript
// Split and join
const csv = "apple,banana,cherry";
const fruits = csv.split(",");
console.log(fruits); // ["apple", "banana", "cherry"]
const back = fruits.join(" | ");
console.log(back);   // "apple | banana | cherry"

// Replace
const text = "I love JavaScript. JavaScript is great!";
console.log(text.replace("JavaScript", "TypeScript"));
// "I love TypeScript. JavaScript is great!"
console.log(text.replaceAll("JavaScript", "TypeScript"));
// "I love TypeScript. TypeScript is great!"

// Padding
console.log("5".padStart(3, "0"));   // "005"
console.log("hello".padEnd(8, ".")); // "hello..."
console.log("-42".padStart(5));      // "  -42"

// Repeat
console.log("ha ".repeat(3)); // "ha ha ha "

// Trim variations
console.log("  hello  ".trimStart());    // "hello  "
console.log("  hello  ".trimEnd());      // "  hello"

// startsWith/endsWith with position
console.log(str.startsWith("World", 7)); // true
console.log(str.endsWith("Hello", 5));   // true
```

### Advanced Examples
```javascript
// Regular expression methods
const phone = "Call me at (555) 123-4567";
const match = phone.match(/\((\d{3})\)\s(\d{3})-(\d{4})/);
console.log(match[0]);  // "(555) 123-4567"
console.log(match[1]);  // "555"
console.log(match[2]);  // "123"
console.log(match[3]);  // "4567"

// matchAll with global regex
const dates = "2024-01-15, 2024-02-20";
const dateRegex = /(\d{4})-(\d{2})-(\d{2})/g;
for (const match of dates.matchAll(dateRegex)) {
  console.log(`Year: ${match[1]}, Month: ${match[2]}, Day: ${match[3]}`);
}

// localeCompare for sorting
const words = ["äpfel", "Apfel", "apfel"];
words.sort((a, b) => a.localeCompare(b, "de"));
console.log(words); // German dictionary order

// Unicode normalization
const str1 = "\u00e9";       // é (precomposed)
const str2 = "e\u0301";      // é (decomposed)
console.log(str1 === str2);            // false
console.log(str1.normalize() === str2.normalize()); // true

// Chaining methods
const result = "  Hello, World!  "
  .trim()
  .toUpperCase()
  .replace("WORLD", "JAVASCRIPT")
  .split("")
  .reverse()
  .join("");
console.log(result); // "!TPIRCSAVAJ OLLEH"
```

### Real-World Use Cases
- Form input validation (trim, length checks)
- URL parsing and construction
- CSV/JSON parsing (split, join)
- Text formatting and padding (reports, tables)
- Search and highlighting (indexOf, replace)
- Email/phone validation (match, test)
- Template rendering (replaceAll)
- Sanitization (trim, replace)
- Localization (localeCompare, toLocaleUpperCase)

### Common Mistakes
- Forgetting strings are immutable (methods return new strings)
- Using `==` instead of `localeCompare` for locale-aware comparison
- Assuming `indexOf` returns a boolean (returns -1 for not found)
- Confusing `slice` and `substring` (negative index handling differs)
- Using `substr` (deprecated) instead of `slice`
- Not accounting for Unicode astral symbols (use `Array.from()`)
- Chaining methods without considering intermediate results

### Best Practices
- Use `includes()` for boolean existence checks
- Use `startsWith()` and `endsWith()` for prefix/suffix checks
- Use `slice()` for substring extraction (supports negative indices)
- Use `replaceAll()` instead of global regex for simple replacements
- Use template literals instead of concatenation with `+`
- Use `Array.from()` for character arrays with Unicode symbols
- Normalize Unicode strings before comparison

### Performance Considerations
- Methods create new strings (memory allocation)
- Chaining many methods creates intermediate strings
- In hot paths, combine operations to reduce intermediate strings
- `charCodeAt` is faster than bracket notation for character access
- Regex methods are slower than simple string methods
- `replaceAll` is faster than `replace` with global regex for simple strings

### Interview Questions
1. What is the difference between `slice`, `substring`, and `substr`?
2. How do you check if a string contains a substring?
3. How do you reverse a string?
4. What is the difference between `match` and `matchAll`?
5. How does `localeCompare` work?
6. How do you handle Unicode correctly in strings?

### Coding Challenges
1. Implement `String.prototype.reverse()` without using `Array.reverse()`.
2. Write a function that capitalizes the first letter of each word.
3. Create a string truncation function that adds "..." if truncated.
4. Implement a `slugify` function that creates URL-friendly slugs.
5. Write a function that counts occurrences of a substring.

### Related Topics
- Regular expressions
- Template literals
- Unicode and character encoding

## Template literals

### What It Is
Template literals are string literals enclosed by backticks (`) that support embedded expressions, multi-line strings, and tagged templates.

### Why It Is Important
Template literals provide readable string interpolation, clean multi-line strings, and extensible tagged template patterns. They eliminate cumbersome concatenation and escaping.

### How It Works Internally
The JavaScript parser parses template literals as a series of string parts and expression parts. At runtime, expressions are evaluated and coerced to strings, then concatenated with the static string parts.

### Syntax
```javascript
// Basic template literal
const str = `Hello, World!`;

// String interpolation
const name = "Alice";
const greeting = `Hello, ${name}!`; // "Hello, Alice!"

// Multi-line string
const multi = `
  Line 1
  Line 2
`;

// Tagged template
function tag(strings, ...values) {
  return strings.reduce((acc, str, i) => acc + str + (values[i] || ""), "");
}
const result = tag`Hello, ${name}!`;
```

### Beginner Examples
```javascript
// Basic interpolation
const name = "Alice";
const age = 30;
console.log(`My name is ${name} and I am ${age} years old.`);

// Expressions in template literals
const a = 10, b = 20;
console.log(`${a} + ${b} = ${a + b}`); // "10 + 20 = 30"

// Ternary operator in template
const isMember = true;
console.log(`Price: ${isMember ? "$5.00" : "$10.00"}`);

// Method calls
const user = "john_doe";
console.log(`Welcome, ${user.toUpperCase()}`); // "Welcome, JOHN_DOE"

// Multi-line strings (no \n needed)
const poem = `
  Roses are red,
  Violets are blue,
  Sugar is sweet,
  And so are you.
`;
```

### Intermediate Examples
```javascript
// Nested template literals
const people = ["Alice", "Bob", "Charlie"];
const list = `
  <ul>
    ${people.map(name => `    <li>${name}</li>`).join("\n")}
  </ul>
`;

// Template with conditional
const formatUser = (user) => `
  <div class="user ${user.active ? "active" : "inactive"}">
    <h2>${user.name}</h2>
    <p>${user.bio || "No bio provided"}</p>
  </div>
`;

// Function call in template
const formatDate = (d) => `${d.getFullYear()}-${d.getMonth()+1}-${d.getDate()}`;
console.log(`Today is ${formatDate(new Date())}`);

// Template as function argument
function log(strings, ...values) {
  const message = strings.reduce((acc, str, i) => acc + str + (values[i] || ""), "");
  console.log(`[${new Date().toISOString()}] ${message}`);
}
log`User ${name} logged in at ${Date.now()}`;

// Raw strings (String.raw)
const path = String.raw`C:\Users\${name}\Documents`;
console.log(path); // "C:\Users\Alice\Documents" (backslashes preserved)
```

### Advanced Examples
```javascript
// Tagged template for SQL escaping
function sql(strings, ...values) {
  let result = "";
  strings.forEach((str, i) => {
    result += str;
    if (i < values.length) {
      const value = values[i];
      if (typeof value === "string") {
        result += value.replace(/'/g, "''");
      } else {
        result += String(value);
      }
    }
  });
  return result;
}
const table = "users";
const userName = "O'Brien";
const query = sql`SELECT * FROM ${table} WHERE name = '${userName}'`;
// "SELECT * FROM users WHERE name = 'O''Brien'"

// Tagged template for HTML escaping
function html(strings, ...values) {
  const escape = (str) => String(str)
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;");
  
  return strings.reduce((acc, str, i) =>
    acc + str + (values[i] ? escape(values[i]) : ""), ""
  );
}

const userInput = "<script>alert('xss')</script>";
const safeHtml = html`<div>${userInput}</div>`;

// Custom tagged template for i18n
function i18n(strings, ...values) {
  const translations = { "Hello": "Hola", "World": "Mundo" };
  const result = strings.reduce((acc, str, i) => {
    const translated = translations[str] || str;
    return acc + translated + (values[i] || "");
  }, "");
  return result;
}
console.log(i18n`Hello, World!`); // "Hola, Mundo!"
```

### Real-World Use Cases
- Building HTML strings (React's `css` prop, lit-html)
- SQL query generation with escaping
- i18n translation helpers
- Logging and debugging output
- URL construction with parameters
- Dynamic CSS generation
- Formatted error messages
- Email template generation
- GraphQL query construction

### Common Mistakes
- Forgetting backticks instead of quotes when using interpolation
- Using template literals unnecessarily when simple strings suffice
- Not escaping user input in HTML templates (XSS)
- Confusing tagged templates with regular function calls
- Using template literals with `$` signs that look like interpolation

### Best Practices
- Use template literals for any string with variables or expressions
- Use multi-line template literals instead of `\n` concatenation
- Use tagged templates for escaping/sanitization
- Use `String.raw` for paths and regex patterns
- Keep expressions simple; extract complex logic to variables
- Use template literals in JSX for dynamic content
- Prefer template literals over string concatenation

### Performance Considerations
- Template literals are comparable to string concatenation
- Modern engines optimize them similarly
- Tagged templates have minimal overhead
- Complex expressions inside `${}` are evaluated each time
- Extremely long template literals can impact parse time

### Interview Questions
1. What is the difference between template literals and regular strings?
2. How do tagged template functions work?
3. What is `String.raw` used for?
4. Can you nest template literals?
5. How do you handle escaping in template literals?

### Coding Challenges
1. Create a tagged template function that escapes HTML.
2. Write a tagged template function that strips indentation.
3. Implement a simple template engine using template literals.
4. Create a tagged template for building CSS strings.
5. Write a function that pluralizes words in template literals.

### Related Topics
- String interpolation
- Tagged templates
- String methods

## Escape sequences

### What It Is
Escape sequences are special character combinations that represent characters that cannot easily be represented directly in a string. They begin with a backslash (`\`) followed by one or more characters.

### Why It Is Important
Escape sequences enable inclusion of special characters (quotes, newlines, tabs, unicode characters) within strings. Understanding them prevents syntax errors and enables proper text formatting.

### How It Works Internally
During parsing, when the lexer encounters a backslash within a string literal, it reads the following character(s) and replaces the sequence with the corresponding character value in the string's internal representation.

### Syntax
```javascript
// Common escape sequences
\'        // Single quote
\"        // Double quote
\\        // Backslash
\n        // Newline
\r        // Carriage return
\t        // Tab
\b        // Backspace
\f        // Form feed
\v        // Vertical tab
\0        // Null character (not same as null)
\xXX      // Latin-1 character (hex)
\uXXXX    // Unicode character (hex)
\u{X...}  // Unicode code point (ES6+)
```

### Beginner Examples
```javascript
// Escaping quotes
const single = 'It\'s a sunny day';
const double = "He said, \"Hello!\"";
const backtick = `Template with \`backtick\``;

// Newlines
const multiLine = "Line 1\nLine 2\nLine 3";
console.log(multiLine);
// Line 1
// Line 2
// Line 3

// Tab
console.log("Column 1\tColumn 2\tColumn 3");

// Backslash
const path = "C:\\Users\\Alice\\Documents";

// Unicode
console.log("\u00A9");  // ©
console.log("\u2665");  // ♥
console.log("\u{1F600}"); // 😀 (ES6+)
```

### Intermediate Examples
```javascript
// Hex escape
console.log("\x48\x65\x6C\x6C\x6F"); // "Hello"

// Unicode escape variations
console.log("\u0041");      // "A"
console.log("\u{41}");      // "A" (ES6+)
console.log("\u{1F431}");   // "🐱" (cat emoji)

// Combining escapes
const header = "=== Header ===\n\t" + "Section 1\n\t\t" + "Subsection\n";
console.log(header);

// String.raw ignores escapes
const rawPath = String.raw`C:\Users\${name}\Documents`;
console.log(rawPath); // "C:\Users\Alice\Documents" (literal backslashes)

// Template literal newlines (not \n)
const multi = `First line
Second line
Third line`;
```

### Advanced Examples
```javascript
// Octal escapes (strict mode forbids)
// "use strict";
// console.log("\033"); // SyntaxError in strict mode

// Custom escape sequence processing
function interpretEscapes(str) {
  return str.replace(/\\(.)/g, (match, char) => {
    const escapeMap = {
      n: "\n", r: "\r", t: "\t", b: "\b",
      f: "\f", v: "\v", 0: "\0",
      "'": "'", '"': '"', "\\": "\\"
    };
    if (escapeMap[char]) return escapeMap[char];
    if (char === "x") return ""; // would need hex parsing
    return char;
  });
}

// Unicode normalization with escapes
const combining = "A\u0301"; // A + combining accent
console.log(combining);                // Á
console.log(combining.normalize());    // Á

// Escape sequences in regex (different escaping rules)
const regex = new RegExp("\\d+\\.\\d+"); // matches "3.14"
const regexLiteral = /\d+\.\d+/;         // same thing, cleaner
```

### Real-World Use Cases
- JSON string generation (escaping quotes and special chars)
- File path handling (Windows paths use backslashes)
- Generating formatted output (reports, tables with tabs)
- Internationalization (unicode characters)
- Regex pattern construction (need double escaping)
- CSV export (escaping commas and quotes)
- HTML entity escaping for security

### Common Mistakes
- Forgetting to escape backslashes in file paths
- Using single-quote escape in single-quoted strings unnecessarily
- Confusing `\n` (newline) with `\r\n` (Windows newline)
- Using octal escapes in strict mode (error)
- Double-escaping in template literals (backslashes are literal)
- Not using `String.raw` when literal backslashes are needed

### Best Practices
- Use template literals to avoid quote escaping issues
- Use `String.raw` for paths and regex patterns
- Prefer `\u{...}` syntax for Unicode above U+FFFF
- Use `.normalize()` for consistent Unicode comparison
- Avoid octal escapes (deprecated, forbidden in strict mode)
- Use escape sequences for readability, not obfuscation

### Performance Considerations
- Escape sequences are resolved at parse time, not runtime
- No runtime performance impact from using escapes
- Long escaped strings take slightly longer to parse
- Template literal parsing of escape sequences is efficient

### Interview Questions
1. What is the difference between `\x` and `\u` escape sequences?
2. How do you include a backtick in a template literal?
3. What is `String.raw` and when would you use it?
4. What characters need to be escaped in a string?
5. How do you represent Unicode characters beyond U+FFFF?

### Coding Challenges
1. Write a function that unescapes common escape sequences in a string.
2. Create a function that escapes special characters for use in HTML.
3. Implement a simple JSON string encoder that handles escape sequences.
4. Write a function that converts special characters to their unicode escape sequences.

### Related Topics
- Template literals
- Unicode in JavaScript
- Regular expressions
- JSON.stringify
