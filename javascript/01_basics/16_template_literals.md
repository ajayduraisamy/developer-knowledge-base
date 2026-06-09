# Template Literals - Template literals, interpolation, tagged templates, multi-line strings

## Introduction
Template literals, introduced in ES6, are string literals enclosed by backticks (`) that support embedded expressions, multi-line strings, and customized tag functions. They provide a more powerful and readable way to work with strings compared to traditional single or double quotes.

## Template literal syntax

### What It Is
Template literals are string literals using backtick characters instead of quotes. They can contain placeholders for expression interpolation and span multiple lines.

### Why It Is Important
Template literals replace complex string concatenation with clean, readable interpolation. They eliminate the need for explicit newline characters in multi-line strings and provide a consistent syntax for dynamic text generation.

### How It Works Internally
The parser tokenizes template literals as a sequence of alternating static string parts and expression parts. At runtime, expressions are evaluated using the current scope, coerced to strings, and concatenated with the static parts into a single string result.

### Syntax
```javascript
// Basic template literal
const str = `Hello`; // Same as "Hello" or 'Hello'

// With expression interpolation
const str2 = `Value: ${expression}`;

// Multi-line
const str3 = `Line 1
Line 2
Line 3`;

// Tagged template
const result = tagFunction`string ${expr} string`;
```

### Beginner Examples
```javascript
// Basic usage
const greeting = `Hello, World!`;
console.log(greeting); // "Hello, World!"

// Simple interpolation
const name = "Alice";
console.log(`Hello, ${name}!`); // "Hello, Alice!"

// Expressions
const a = 5, b = 10;
console.log(`${a} + ${b} = ${a + b}`); // "5 + 10 = 15"

// Function calls
function double(n) { return n * 2; }
console.log(`Double 5 is ${double(5)}`); // "Double 5 is 10"

// Ternary operator
const age = 20;
console.log(`${age >= 18 ? "Adult" : "Minor"}`); // "Adult"

// Escaping backticks
const escaped = `This is a backtick: \``;
```

### Intermediate Examples
```javascript
// Nested template literals
const people = ["Alice", "Bob", "Charlie"];
const html = `
  <ul>
    ${people.map(name => `<li>${name}</li>`).join("\n")}
  </ul>
`;
console.log(html);

// Template with object access
const user = { name: "Alice", role: "admin" };
console.log(`User ${user.name} has role: ${user.role}`);

// Conditional template
const isAdmin = true;
const message = `User is ${isAdmin ? "an admin" : "a regular user"}`;

// Chained templates
const firstName = "John";
const lastName = "Doe";
const fullName = `${firstName} ${lastName}`;
const welcome = `Welcome, ${fullName}!`;

// Template with new Date
const now = new Date();
const formatted = `Today is ${now.toLocaleDateString()}`;
```

### Advanced Examples
```javascript
// Computed property names with templates
const id = 42;
const key = `user_${id}`;
const data = {
  [key]: { name: "Alice" }
};
console.log(data.user_42); // { name: "Alice" }

// Template literal as function argument
function log(strings, ...values) {
  const message = strings.reduce((acc, str, i) =>
    acc + str + (values[i] !== undefined ? `[${values[i]}]` : ""), ""
  );
  console.log(`[LOG] ${message}`);
}
const status = "success";
const code = 200;
log`Request completed with status ${status} (code ${code})`;

// Recursive template evaluation
function evaluateTemplate(str, data) {
  return str.replace(/\$\{(.*?)\}/g, (_, expr) => {
    return Function(...Object.keys(data), `return ${expr}`)(...Object.values(data));
  });
}
const tpl = "Hello, ${name}! You are ${age} years old.";
console.log(evaluateTemplate(tpl, { name: "Alice", age: 30 }));

// Template literal for regex construction
const pattern = "hello";
const flags = "gi";
const regex = new RegExp(`${pattern}`, flags);
```

### Real-World Use Cases
- HTML string generation
- SQL query construction (with proper escaping)
- Dynamic URL building
- Logging and error messages
- Code generation
- Email/SMS templates
- CSS-in-JS (styled-components, emotion)
- CLI output formatting
- Dynamic import paths
- GraphQL query construction

### Common Mistakes
- Forgetting backticks when using interpolation
- Not escaping backticks within template literals
- Trying to use template literals with `Symbol` (implicit conversion throws)
- Overcomplicating templates with too much logic in expressions
- Using template literals when simple quotes are sufficient

### Best Practices
- Use template literals for all strings with dynamic content
- Keep template expressions simple (extract complex logic)
- Use multi-line templates instead of `\n` concatenation
- Use tagged templates for escaping/sanitization
- Prefer template literals over `+` concatenation
- Use `String.raw` for paths and regex
- Avoid nesting template literals more than 2 levels deep

### Performance Considerations
- Template literals have similar performance to string concatenation
- Modern JIT engines optimize them well
- Complex expressions in `${}` are evaluated each time
- Tagged templates have minimal overhead
- Very large template literals may impact parse time

### Interview Questions
1. What are template literals and how do they differ from regular strings?
2. How do you include a backtick in a template literal?
3. Can you use template literals for multi-line strings?
4. What is the difference between template literals and tagged templates?

### Coding Challenges
1. Write a function that converts a traditional concatenation string to use template literals.
2. Create a nested template that renders a list of items.
3. Implement a simple template engine using template literals.

### Related Topics
- String interpolation
- Tagged templates
- Multi-line strings

## String interpolation

### What It Is
String interpolation is the process of embedding expressions directly within a string literal. Template literals use `${expression}` syntax for interpolation.

### Why It Is Important
Interpolation eliminates error-prone string concatenation, improves readability, and allows inline evaluation of expressions. It is more maintainable and less verbose than `+` concatenation.

### How It Works Internally
The `${}` syntax is parsed as an expression context. The expression is evaluated, converted to a string via ToString, and embedded in the result. Complex expressions can include arithmetic, ternary operators, function calls, and nested templates.

### Syntax
```javascript
`text ${expression} text`
`${fn()} ${a + b} ${condition ? val1 : val2}`
```

### Beginner Examples
```javascript
// Variables
const name = "Alice";
console.log(`Hello, ${name}`); // "Hello, Alice"

// Arithmetic
const price = 19.99;
const tax = 0.08;
console.log(`Total: $${(price * (1 + tax)).toFixed(2)}`); // "Total: $21.59"

// Ternary
const loggedIn = true;
console.log(`Status: ${loggedIn ? "Online" : "Offline"}`);

// Method calls
const str = "hello";
console.log(`${str.toUpperCase()}`); // "HELLO"

// Accessing properties
const user = { name: "Bob", age: 25 };
console.log(`${user.name} is ${user.age}`); // "Bob is 25"
```

### Intermediate Examples
```javascript
// Nested interpolation
const items = [
  { name: "Apple", price: 0.99, qty: 3 },
  { name: "Banana", price: 0.59, qty: 5 }
];
const receipt = `
  ${items.map(item =>
    `${item.name}: $${(item.price * item.qty).toFixed(2)}`
  ).join("\n  ")}
`;
console.log(receipt);

// Multiple expressions
const x = 10, y = 20;
console.log(`(${x}, ${y}) sum=${x + y}, product=${x * y}`);

// Expression with logical operators
const config = { theme: "dark" };
console.log(`Theme: ${config.theme || "light"}`);

// Conditional formatting
const score = 85;
const grade = score >= 90 ? "A" : score >= 80 ? "B" : score >= 70 ? "C" : "D";
console.log(`Score: ${score}, Grade: ${grade}`);
```

### Advanced Examples
```javascript
// IIFE in template
const result = `Result: ${(() => {
  const complex = calculate(42);
  return formatResult(complex);
})()}`;

// Tagged function call in interpolation
function highlight(str) {
  return `**${str}**`;
}
console.log(`This is ${highlight("important")}`); // "This is **important**"

// Template as a function to delay evaluation
const tpl = (data) => `Hello, ${data.name}! You have ${data.messages.length} messages.`;
console.log(tpl({ name: "Alice", messages: [1, 2, 3] }));

// Interpolation with nullish coalescing
const displayName = name ?? `User_${id}`;
console.log(`Welcome, ${displayName}`);

// Formatting with Intl
const amount = 1234.56;
console.log(`Amount: $${amount.toLocaleString("en-US", { minimumFractionDigits: 2 })}`);
```

### Real-World Use Cases
- Dynamic email templates
- Error messages with context
- Log messages with variables
- SQL query parameter embedding
- API URL construction
- HTML templating
- Localized message formatting

### Common Mistakes
- Including complex logic in interpolation (extract to variable)
- Forgetting that `${}` evaluates expressions, not statements
- Trying to use `${if(x)}` (use ternary instead)
- Not escaping user-generated content in HTML templates (XSS risk)
- Overusing nested interpolation (affects readability)

### Best Practices
- Keep interpolation expressions simple
- Extract complex logic to named variables or functions
- Use conditional (ternary) for inline if/else
- Avoid doing side effects in interpolation
- Use template literals with tagged templates for security (escaping)
- Be mindful of readability: inline short expressions, extract long ones

### Performance Considerations
- Interpolation has similar cost to string concatenation
- Expression evaluation cost depends on the expression
- JIT optimizes simple property access and arithmetic
- Repeated use of the same template should be cached in a function

### Interview Questions
1. What syntax does JavaScript use for string interpolation?
2. Can you include arithmetic expressions in interpolation?
3. How do you handle conditional logic in template literals?
4. Can you call functions within interpolation?

### Coding Challenges
1. Create a function that takes a template string and data object and returns the interpolated result.
2. Write a receipt generator using template literals with interpolation.
3. Implement a function that highlights interpolated values.

### Related Topics
- Template literal syntax
- Expression evaluation
- Tagged templates

## Tagged templates

### What It Is
Tagged templates allow you to call a function (the "tag") on a template literal. The tag function receives the static string parts and the interpolated values, allowing custom processing of the template.

### Why It Is Important
Tagged templates provide a powerful mechanism for creating DSLs (domain-specific languages), sanitizing or transforming template output, building components (like styled-components), and implementing custom string formatting.

### How It Works Internally
The tag function is called with two arguments: an array of string parts (with a `.raw` property) and the interpolated values as separate arguments. The function can process these and return any value (not necessarily a string).

### Syntax
```javascript
function tag(strings, ...values) {
  // strings: Array of string literals
  // values: Interpolated expressions
  return result;
}

const result = tag`string ${expr} string`;
```

### Beginner Examples
```javascript
// Basic tag function
function simple(strings, ...values) {
  console.log(strings); // ["Hello, ", "!"]
  console.log(values);  // ["World"]
  return strings[0] + values[0] + strings[1];
}
console.log(simple`Hello, ${"World"}!`); // "Hello, World!"

// Tag that uppercases values
function upper(strings, ...values) {
  let result = "";
  strings.forEach((str, i) => {
    result += str + (values[i] ? String(values[i]).toUpperCase() : "");
  });
  return result;
}
const name = "Alice";
console.log(upper`Hello, ${name}!`); // "Hello, ALICE!"

// HTML escaping tag
function escapeHtml(strings, ...values) {
  const escape = (str) => String(str)
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#39;");
  
  return strings.reduce((acc, str, i) =>
    acc + str + (values[i] ? escape(values[i]) : ""), ""
  );
}
const userInput = '<script>alert("xss")</script>';
console.log(escapeHtml`<div>${userInput}</div>`);
```

### Intermediate Examples
```javascript
// Currency formatting tag
function currency(strings, ...values) {
  return strings.reduce((acc, str, i) => {
    const value = values[i];
    if (typeof value === "number") {
      return acc + str + `$${value.toFixed(2)}`;
    }
    return acc + str + (value ?? "");
  }, "");
}
const price = 19.99;
const tax = 1.60;
console.log(currency`Price: ${price}, Tax: ${tax}, Total: ${price + tax}`);
// "Price: $19.99, Tax: $1.60, Total: $21.59"

// String.raw built-in tag
const path = String.raw`C:\Users\${name}\Documents`;
console.log(path); // "C:\Users\Alice\Documents" (backslashes preserved)

// Tag that repeats
function repeat(strings, ...values) {
  let result = "";
  strings.forEach((str, i) => {
    const times = values[i] || 1;
    result += str.repeat(times);
  });
  return result;
}
console.log(repeat`${3}Hello `); // "Hello Hello Hello "

// CSS tag (styled-components style)
function css(strings, ...values) {
  return strings.reduce((acc, str, i) =>
    acc + str + (values[i] ? String(values[i]) : ""), ""
  );
}
const color = "blue";
const style = css`
  color: ${color};
  font-size: 16px;
`;
```

### Advanced Examples
```javascript
// SQL query builder tag
function sql(strings, ...values) {
  let result = "";
  strings.forEach((str, i) => {
    result += str;
    if (i < values.length) {
      const value = values[i];
      if (typeof value === "string") {
        result += `'${value.replace(/'/g, "''")}'`;
      } else if (value === null || value === undefined) {
        result += "NULL";
      } else if (Array.isArray(value)) {
        result += value.map(v => typeof v === "string" ? `'${v}'` : v).join(", ");
      } else {
        result += String(value);
      }
    }
  });
  return result;
}
const table = "users";
const userName2 = "O'Brien";
const query = sql`SELECT * FROM ${table} WHERE name = ${userName2}`;
console.log(query);
// "SELECT * FROM users WHERE name = 'O''Brien'"

// Internationalization tag
function i18n(strings, ...values) {
  const translations = {
    "Hello": { fr: "Bonjour", es: "Hola" },
    "World": { fr: "Monde", es: "Mundo" }
  };
  const lang = "fr";
  
  return strings.reduce((acc, str, i) => {
    const translated = translations[str]?.[lang] ?? str;
    return acc + translated + (values[i] ?? "");
  }, "");
}
console.log(i18n`Hello, ${"dear"} World!`); // "Bonjour, dear Monde!"

// Array-building tag
function list(strings, ...values) {
  const items = [];
  strings.forEach((str, i) => {
    if (str.trim()) items.push(str.trim());
    if (values[i]) items.push(values[i]);
  });
  return items;
}
const result2 = list`Item 1, ${"item2"}, Item 3`;
console.log(result2); // ["Item 1,", "item2", "Item 3"]
```

### Real-World Use Cases
- styled-components (CSS-in-JS)
- lit-html (HTML templating)
- GraphQL template tags (gql)
- SQL escaping and building
- HTML sanitization
- i18n translation helpers
- Log formatting
- CSS preprocessing
- Markdown generation
- Code formatting (prettier-style tags)

### Common Mistakes
- Forgetting that tag functions receive arrays, not single strings
- Not handling the case where there are more strings than values
- Assuming the return value must be a string (it can be anything)
- Modifying the `strings` array (it's frozen)
- Confusing `strings` and `strings.raw`

### Best Practices
- Use tagged templates for domain-specific string processing
- Always escape user input in HTML/SQL tags
- Access `strings.raw` when you need raw (unescaped) string content
- Return meaningful types from tag functions
- Document tag function behavior thoroughly
- Use descriptive names for custom tags

### Performance Considerations
- Tag functions are called once per template evaluation
- The strings array is reused by the engine (cached)
- Values array is created fresh each evaluation
- Tag function overhead is minimal
- Complex processing in tags should be optimized

### Interview Questions
1. What is a tagged template literal?
2. What parameters does a tag function receive?
3. What is the difference between `strings` and `strings.raw`?
4. How are tagged templates used in libraries like styled-components?
5. Can a tagged template return a non-string value?

### Coding Challenges
1. Implement a tag function that uppercases all interpolated values.
2. Create an HTML escaping tag function.
3. Build a simple i18n tag function.
4. Implement a SQL query builder tag with injection prevention.
5. Write a tag function that highlights interpolated values with ANSI colors.

### Related Topics
- Template literal syntax
- String.raw
- Domain-Specific Languages (DSLs)
- CSS-in-JS

## Multi-line strings

### What It Is
Multi-line strings allow strings to span multiple lines in source code without explicit newline characters. Template literals natively support multi-line strings.

### Why It Is Important
Multi-line strings simplify creating formatted text, code snippets, and templates. They eliminate the need for `\n` concatenation, making code more readable and maintainable.

### How It Works Internally
When a template literal contains actual newline characters in the source, those newlines are preserved in the resulting string value. The parser captures the raw source text between the backticks, including line breaks.

### Syntax
```javascript
const multiLine = `
  Line 1
  Line 2
  Line 3
`;
```

### Beginner Examples
```javascript
// Simple multi-line
const poem = `
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.
`;
console.log(poem);

// Multi-line with indentation
const code = `
function greet(name) {
  return \`Hello, \${name}!\`;
}
`;
console.log(code);

// Multi-line string (non-template, old way)
const oldWay = "Line 1\n" +
              "Line 2\n" +
              "Line 3";

// Multi-line with template
const newWay = `Line 1
Line 2
Line 3`;
```

### Intermediate Examples
```javascript
// HTML template
const html = `
  <div class="container">
    <h1>${title}</h1>
    <ul>
      ${items.map(item => `
        <li class="${item.active ? 'active' : ''}">
          ${item.name}
        </li>
      `).join("")}
    </ul>
  </div>
`;

// Preserving indentation in output
function dedent(strings, ...values) {
  const result = strings.reduce((acc, str, i) =>
    acc + str + (values[i] || ""), ""
  );
  const lines = result.split("\n");
  const minIndent = lines
    .filter(l => l.trim())
    .reduce((min, line) => Math.min(min, line.match(/^\s*/)[0].length), Infinity);
  return lines.map(l => l.slice(minIndent)).join("\n").trim();
}

const formatted = dedent`
  This is a
    multi-line string
  with proper formatting
`;
console.log(formatted);
```

### Advanced Examples
```javascript
// Multi-line with template expressions
const name2 = "Alice";
const multi = `
  Dear ${name2},
  
  We are pleased to inform you that your
  application has been ${"approved"}.
  
  Best regards,
  The Team
`;

// Raw string with preserved backslashes
const raw = String.raw`
  Path: C:\Users\${name}\Documents
  Regex: \d+\.\d+
`;

// Function to strip indentation
function stripIndent(str) {
  const lines = str.split("\n");
  if (lines[0].trim() === "") lines.shift();
  if (lines[lines.length - 1].trim() === "") lines.pop();
  const indent = lines.reduce((min, line) => {
    if (!line.trim()) return min;
    const leading = line.match(/^\s*/)[0].length;
    return Math.min(min, leading);
  }, Infinity);
  return lines.map(line => line.slice(indent)).join("\n");
}

const cleanCode = stripIndent(`
    function test() {
      console.log("Hello");
    }
`);
console.log(cleanCode);
// function test() {
//   console.log("Hello");
// }
```

### Real-World Use Cases
- HTML templates in JavaScript
- Code generation tools
- Email templates
- SQL query formatting
- Documentation strings
- CLI help text
- Markdown generation
- Configuration file content generation
- Error message formatting

### Common Mistakes
- Including leading whitespace that affects output
- Forgetting that multi-line strings include the newline character
- Not accounting for indentation when generating formatted output
- Using `\` continuation (not supported in template literals)
- Confusing template literal newlines with `\n` (they're equivalent)

### Best Practices
- Use multi-line template literals for any string spanning multiple lines
- Use a `dedent` or `stripIndent` utility to handle indentation
- Be explicit about trailing newlines
- Consider using `.trim()` to remove leading/trailing newlines
- Use `join("\n")` for dynamic multi-line generation
- Prefer multi-line templates over `\n` concatenation

### Performance Considerations
- Multi-line template literals have the string content including newlines
- No runtime difference between `\n` and actual newlines
- Very long multi-line strings are parsed once at compile time
- Template literals with many lines have no runtime overhead

### Interview Questions
1. How do you create multi-line strings in JavaScript?
2. What is the difference between multi-line template literals and `\n` concatenation?
3. How do you handle indentation in multi-line template literals?
4. Can you mix multi-line strings with interpolation?

### Coding Challenges
1. Write a `dedent` function that removes common leading whitespace.
2. Create a function that generates a multi-line ASCII table.
3. Implement a function that converts indented multi-line strings to flat strings.
4. Build a code generator that outputs properly formatted multi-line code.

### Related Topics
- Template literal syntax
- String concatenation
- String methods (trim, split, join)
