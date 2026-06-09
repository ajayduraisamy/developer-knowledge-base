# Destructuring - Array destructuring, object destructuring, nested, default values

## Introduction
Destructuring is a JavaScript expression that unpacks values from arrays or properties from objects into distinct variables. Introduced in ES6, it provides a concise syntax for extracting data, improving readability and reducing boilerplate.

## Array destructuring

### What It Is
Array destructuring allows extracting array elements into variables based on their position. It uses a syntax that mirrors array literals on the left side of an assignment.

### Why It Is Important
Array destructuring simplifies code that needs to extract multiple values from arrays, such as parsing return values, swapping variables, and extracting specific elements from function results.

### How It Works Internally
The destructuring pattern on the left side is matched against the array on the right side. The engine iterates the array and assigns values based on position. The pattern can include rest elements, defaults, and skipped positions.

### Syntax
```javascript
// Basic
const [a, b] = [1, 2];

// Skip elements
const [a, , b] = [1, 2, 3];

// Rest element
const [a, ...rest] = [1, 2, 3, 4];

// Default values
const [a = 0, b = 0] = [1];

// Swapping
[a, b] = [b, a];
```

### Beginner Examples
```javascript
// Basic array destructuring
const colors = ["red", "green", "blue"];
const [first, second, third] = colors;
console.log(first);  // "red"
console.log(second); // "green"
console.log(third);  // "blue"

// Skipping elements
const [firstColor, , thirdColor] = colors;
console.log(firstColor); // "red"
console.log(thirdColor); // "blue"

// Default values
const [a = 10, b = 20] = [5];
console.log(a); // 5
console.log(b); // 20

// Rest operator
const [head, ...tail] = [1, 2, 3, 4];
console.log(head); // 1
console.log(tail); // [2, 3, 4]

// Variable swapping
let x = 1, y = 2;
[x, y] = [y, x];
console.log(x); // 2
console.log(y); // 1
```

### Intermediate Examples
```javascript
// Returning multiple values from function
function getMinMax(numbers) {
  return [Math.min(...numbers), Math.max(...numbers)];
}
const [min, max] = getMinMax([3, 1, 4, 1, 5, 9]);
console.log(min); // 1
console.log(max); // 9

// Parsing CSV
const csv = "John,Doe,30";
const [firstName, lastName, age] = csv.split(",");
console.log(firstName, lastName, age); // "John" "Doe" "30"

// Nested destructuring
const nested = [1, [2, 3], 4];
const [a, [b, c], d] = nested;
console.log(a, b, c, d); // 1 2 3 4

// Destructuring function parameters
function printFirstTwo([first, second]) {
  console.log(`First: ${first}, Second: ${second}`);
}
printFirstTwo(["Alice", "Bob", "Charlie"]); // "First: Alice, Second: Bob"

// Destructuring with iteration
const points = [[1, 2], [3, 4], [5, 6]];
for (const [x, y] of points) {
  console.log(`Point: (${x}, ${y})`);
}
```

### Advanced Examples
```javascript
// Destructuring with generator
function* fibonacci() {
  let a = 0, b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}
const [f0, f1, f2, f3, f4, f5] = fibonacci();
console.log(f0, f1, f2, f3, f4, f5); // 0 1 1 2 3 5

// Conditional destructuring with default
function getCoordinates(options = {}) {
  const { x = 0, y = 0 } = options;
  return [x, y];
}
const [xPos, yPos] = getCoordinates({ x: 10 });
console.log(xPos, yPos); // 10 0

// Ignoring return values
const [fullName2] = ["John Doe"];
const [, , thirdItem] = [1, 2, 3, 4, 5];

// Destructuring regex match
const dateRegex = /(\d{4})-(\d{2})-(\d{2})/;
const [, year, month, day] = dateRegex.exec("2024-01-15") || [];
console.log(year, month, day); // "2024" "01" "15"

// Deeply nested array destructuring
const data = [1, [2, [3, 4]], 5];
const [a2, [b2, [c2, d2]]] = data;
console.log(a2, b2, c2, d2); // 1 2 3 4
```

### Real-World Use Cases
- Extracting values from array return types (like `useState` in React)
- Swapping variables without a temporary variable
- Parsing CSV or split strings
- Extracting parts of coordinate arrays
- Destructuring in `for...of` loops
- Handling multiple return values from functions
- Parsing regular expression matches

### Common Mistakes
- Trying to destructure `null` or `undefined` (throws TypeError)
- Forgetting that destructuring creates new variables, not references
- Using rest element in middle of pattern (only at end)
- Not providing defaults for potentially undefined values
- Confusing array destructuring with object destructuring syntax

### Best Practices
- Use array destructuring for position-based extraction
- Provide defaults for potentially missing values
- Use rest element for collecting remaining items
- Use destructuring in `for...of` loops for cleaner code
- Destructure function return values at call site
- Use `|| []` or `?? []` to handle null/undefined sources

### Performance Considerations
- Array destructuring has minimal overhead
- Destructuring creates variable bindings (like normal assignments)
- Modern engines optimize destructuring well
- Repeated destructuring of same source in hot paths may be optimized

### Interview Questions
1. How do you swap two variables using destructuring?
2. How do you skip elements in array destructuring?
3. What is the rest element in array destructuring?
4. How do you provide default values in array destructuring?
5. How do you destructure nested arrays?

### Coding Challenges
1. Implement a function that returns the first and last element of an array using destructuring.
2. Create a function that splits a full name into first and last name using destructuring.
3. Write a function that rotates array elements using destructuring assignment.
4. Implement a CSV parser using array destructuring.

### Related Topics
- Object destructuring
- Spread and rest operators
- Default parameters

## Object destructuring

### What It Is
Object destructuring extracts properties from objects into variables based on property names. It uses a syntax that mirrors object literals on the left side of an assignment.

### Why It Is Important
Object destructuring is essential for working with configuration objects, API responses, and function parameters. It enables extracting specific properties without repetitive `obj.property` syntax.

### How It Works Internally
The engine matches property names on the left side with property names in the source object. The value of the matching property is assigned to the variable. The pattern can include renaming, defaults, nested destructuring, and rest elements.

### Syntax
```javascript
// Basic
const { prop1, prop2 } = obj;

// Renaming
const { prop1: var1, prop2: var2 } = obj;

// Default values
const { prop1 = default1, prop2 = default2 } = obj;

// Renaming with default
const { prop1: var1 = defaultValue } = obj;

// Rest
const { prop1, ...rest } = obj;

// Nested
const { outer: { inner } } = obj;
```

### Beginner Examples
```javascript
// Basic object destructuring
const user = { name: "Alice", age: 30, email: "alice@example.com" };
const { name, age, email } = user;
console.log(name);  // "Alice"
console.log(age);   // 30
console.log(email); // "alice@example.com"

// Renaming properties
const { name: userName, age: userAge } = user;
console.log(userName); // "Alice"
console.log(userAge);  // 30

// Default values
const { name, role = "user" } = user;
console.log(role); // "user" (default)

// Renaming with default
const { name: fullName = "Unknown", status = "active" } = user;
console.log(fullName); // "Alice"
console.log(status);   // "active"

// Function parameter destructuring
function printUser({ name, age }) {
  console.log(`${name} is ${age} years old`);
}
printUser({ name: "Bob", age: 25 }); // "Bob is 25 years old"
```

### Intermediate Examples
```javascript
// Destructuring function parameters with defaults
function createUser({ name, email, role = "user", active = true } = {}) {
  return { name, email, role, active, createdAt: new Date() };
}
const newUser = createUser({ name: "Charlie", email: "charlie@example.com" });
console.log(newUser.role); // "user"
console.log(newUser.active); // true

// Nested destructuring
const response = {
  status: 200,
  data: {
    user: { id: 1, name: "Alice" },
    posts: [{ id: 101 }, { id: 102 }]
  }
};
const { status, data: { user: { name: userName2 }, posts } } = response;
console.log(status);    // 200
console.log(userName2); // "Alice"
console.log(posts);     // [{ id: 101 }, { id: 102 }]

// Combined array and object destructuring
const users = [
  { id: 1, name: "Alice", roles: ["admin", "editor"] },
  { id: 2, name: "Bob", roles: ["user"] }
];
for (const { id, name, roles: [firstRole] } of users) {
  console.log(`${name} (ID: ${id}) first role: ${firstRole}`);
}

// Rest in object destructuring
const { name: name2, ...details } = user;
console.log(name2);    // "Alice"
console.log(details);  // { age: 30, email: "alice@example.com" }
```

### Advanced Examples
```javascript
// Computed property names in destructuring
const prop = "name";
const { [prop]: value } = { name: "Alice" };
console.log(value); // "Alice"

// Destructuring with prototype chain
const proto = { inherited: "yes" };
const obj = Object.create(proto);
obj.own = "property";
const { own, inherited } = obj;
console.log(own);       // "property"
console.log(inherited); // "yes" (inherited properties work)

// Multiple source objects
const config1 = { host: "localhost", port: 3000 };
const config2 = { ssl: true };
const merged = { ...config1, ...config2 };
const { host, port, ssl } = merged;

// Destructuring null/undefined safely
function safeDestructure(obj) {
  const { name = "default", age = 0 } = obj || {};
  return { name, age };
}
console.log(safeDestructure(null));    // { name: "default", age: 0 }
console.log(safeDestructure(undefined)); // { name: "default", age: 0 }

// Dynamic destructuring with function
function extract(obj, ...props) {
  const result = {};
  for (const prop of props) {
    if (prop in obj) {
      result[prop] = obj[prop];
    }
  }
  return result;
}
const extracted = extract(user, "name", "email", "nonexistent");
console.log(extracted); // { name: "Alice", email: "alice@example.com" }
```

### Real-World Use Cases
- Extract specific properties from API responses
- Configuration object handling in libraries
- React props destructuring
- Redux state destructuring in mapStateToProps
- Function parameter options (named parameters pattern)
- Environment variable processing
- Database query result destructuring

### Common Mistakes
- Forgetting curly braces in destructuring assignment
- Trying to destructure null/undefined (provide fallback)
- Confusing property renaming with type annotation (both use `:`)
- Assuming destructuring creates a copy (it doesn't, it's a reference)
- Using object destructuring without declaring variables (needs `let`/`const`/`var`)

### Best Practices
- Use object destructuring for named property extraction
- Provide default values for optional properties
- Use renaming to avoid naming conflicts
- Destructure at the top of functions for clear dependencies
- Use nested destructuring sparingly (readability)
- Destructure function parameters for clean option objects
- Use rest properties (`...rest`) to collect remaining props

### Performance Considerations
- Object destructuring is comparable to manual property access
- No runtime overhead for creating the destructuring pattern
- Destructuring parameters creates local variables (like normal parameter assignment)
- Modern engines optimize destructuring well

### Interview Questions
1. What is the difference between array and object destructuring?
2. How do you rename a property during destructuring?
3. How do you provide default values in object destructuring?
4. How do you destructure nested objects?
5. What is the rest pattern in object destructuring?

### Coding Challenges
1. Write a function that takes a config object and extracts specific properties with defaults.
2. Create a function that flattens nested object destructuring.
3. Implement a function that picks specific properties from an object using destructuring.
4. Write a function that renames properties during destructuring.

### Related Topics
- Array destructuring
- Default values
- Rest and spread operators

## Nested destructuring

### What It Is
Nested destructuring allows extracting values from deeply nested arrays or objects using a destructuring pattern that mirrors the source structure.

### Why It Is Important
Nested destructuring eliminates repetitive access chains and makes code more declarative. It is especially useful for processing complex API responses, configuration files, and deeply structured data.

### How It Works Internally
The engine recursively matches the destructuring pattern against the nested structure. Each level of nesting creates its own variable bindings according to the pattern.

### Syntax
```javascript
// Nested object
const { outer: { inner } } = obj;

// Nested array
const [first, [second, third]] = arr;

// Mixed
const { data: [firstItem] } = obj;

// Deeply nested with defaults
const { a: { b: { c = 0 } = {} } = {} } = obj;
```

### Beginner Examples
```javascript
// Nested object destructuring
const user = {
  name: "Alice",
  address: {
    street: "123 Main St",
    city: "New York",
    zip: "10001"
  }
};
const { name, address: { street, city } } = user;
console.log(name);   // "Alice"
console.log(street); // "123 Main St"
console.log(city);   // "New York"

// Nested array destructuring
const numbers = [1, [2, 3], 4];
const [a, [b, c], d] = numbers;
console.log(a, b, c, d); // 1 2 3 4

// Mixed destructuring
const data = {
  id: 1,
  items: ["apple", "banana", "cherry"]
};
const { id, items: [firstItem, secondItem] } = data;
console.log(firstItem);  // "apple"
console.log(secondItem); // "banana"
```

### Intermediate Examples
```javascript
// Deeply nested with fallbacks
const apiResponse = {
  status: 200,
  data: {
    profile: {
      name: "Alice",
      settings: {
        theme: "dark"
      }
    }
  }
};

const { data: { profile: { name: userName, settings: { theme = "light" } = {} } = {} } = {} } = apiResponse;
console.log(userName); // "Alice"
console.log(theme);    // "dark"

// Nested destructuring in function parameters
function processOrder({
  customer: { name: customerName },
  items: [{ product, quantity }],
  shipping: { address: { city } }
}) {
  console.log(`${customerName} ordered ${quantity}x ${product} to ${city}`);
}

processOrder({
  customer: { name: "Alice" },
  items: [{ product: "Widget", quantity: 3 }],
  shipping: { address: { city: "NYC" } }
});

// Array of objects nested destructuring
const orders = [
  { id: 1, items: [{ price: 10, qty: 2 }, { price: 5, qty: 3 }] },
  { id: 2, items: [{ price: 20, qty: 1 }] }
];

orders.forEach(({ id, items: [{ price: firstPrice }] }) => {
  console.log(`Order ${id}, first item price: ${firstPrice}`);
});
```

### Advanced Examples
```javascript
// Recursive destructuring pattern
function flattenDestructure(obj, path = "") {
  const result = {};
  for (const [key, value] of Object.entries(obj)) {
    const newPath = path ? `${path}.${key}` : key;
    if (typeof value === "object" && value !== null && !Array.isArray(value)) {
      Object.assign(result, flattenDestructure(value, newPath));
    } else {
      result[newPath] = value;
    }
  }
  return result;
}

const flat = flattenDestructure({ a: { b: { c: 1, d: 2 }, e: 3 }, f: 4 });
console.log(flat); // { "a.b.c": 1, "a.b.d": 2, "a.e": 3, "f": 4 }

// Safe nested destructuring with defaults at each level
const safe = (obj) => ({
  a: obj?.a ?? 0,
  b: obj?.b ?? 0,
  nested: {
    c: obj?.nested?.c ?? 0,
    d: obj?.nested?.d ?? 0
  }
});

// Nested destructuring with renaming at multiple levels
const complex = {
  level1: {
    name: "L1",
    level2: {
      name: "L2",
      value: 42
    }
  }
};
const {
  level1: {
    name: l1Name,
    level2: {
      name: l2Name,
      value: l2Value
    }
  }
} = complex;
console.log(l1Name, l2Name, l2Value); // "L1" "L2" 42
```

### Real-World Use Cases
- API response parsing (nested JSON)
- Configuration file processing
- GraphQL response destructuring
- ORM query result destructuring
- Complex state object destructuring
- Event object destructuring (DOM events)
- Form data processing with nested fields

### Common Mistakes
- Forgetting defaults for nested levels (crashes on undefined)
- Making deeply nested destructuring unreadable (extract into steps)
- Confusing array and object destructuring syntax at nested levels
- Not accounting for null/undefined at intermediate levels
- Over-using nested destructuring (sometimes manual access is clearer)

### Best Practices
- Limit nesting to 2-3 levels deep for readability
- Provide defaults at each nesting level to avoid crashes
- Use optional chaining (`?.`) as an alternative for deep access
- Extract deeply nested destructuring into helper functions
- Consider using lodash `get` or similar for very deep access
- Use default empty objects: `{ nested: { prop } = {} } = {}`

### Performance Considerations
- Nested destructuring is evaluated at runtime
- Each level adds property access operations
- Default objects at each level add allocation overhead
- For hot paths, manual property access may be faster
- Modern engines optimize common nested patterns well

### Interview Questions
1. How do you safely destructure deeply nested objects?
2. How do you provide fallbacks at each nesting level?
3. What happens if you try to destructure a non-existent nested property?
4. How do you rename properties at nested levels?

### Coding Challenges
1. Write a safe nested destructuring function that handles missing properties.
2. Create a function that extracts fields from a nested API response.
3. Implement a nested destructuring pattern for a configuration object.
4. Write a utility that converts between nested objects and dot-notated flat objects.

### Related Topics
- Object destructuring
- Array destructuring
- Default values
- Optional chaining

## Default values

### What It Is
Default values in destructuring allow specifying fallback values when the extracted value is `undefined`. They provide safety and reduce the need for manual undefined checks.

### Why It Is Important
Default values make destructuring robust and self-documenting. They ensure variables always have a defined value and eliminate the need for post-destructuring undefined checks.

### How It Works Internally
During destructuring, if the source value is `undefined`, the engine evaluates the default expression. Defaults can be any expression, including function calls. The default is only used when the value is `undefined` (not for `null`).

### Syntax
```javascript
// Array default
const [a = 1, b = 2] = [undefined, null];

// Object default
const { x = 10, y = 20 } = { x: undefined, y: null };

// Function parameter default
function f({ a = 1, b = 2 } = {}) {}

// Default with renaming
const { prop: variable = defaultValue } = obj;

// Default with nested
const { outer: { inner = defaultValue } = {} } = obj;
```

### Beginner Examples
```javascript
// Array destructuring with defaults
const [a = 10, b = 20] = [5];
console.log(a); // 5 (from array)
console.log(b); // 20 (default used)

// Undefined triggers default, null does not
const [x = "default"] = [undefined];
console.log(x); // "default"

const [y = "default"] = [null];
console.log(y); // null (null is not undefined)

// Object destructuring with defaults
const { name = "Guest", role = "user" } = { name: undefined };
console.log(name); // "Guest" (default used)
console.log(role); // "user" (default used)

// Default with renaming
const { name: userName = "Unknown" } = {};
console.log(userName); // "Unknown"

// Multiple defaults
const { host = "localhost", port = 3000, ssl = false } = {};
console.log(host, port, ssl); // "localhost" 3000 false
```

### Intermediate Examples
```javascript
// Default values in function parameters
function fetchData({
  url = "https://api.example.com",
  method = "GET",
  headers = {},
  timeout = 5000,
  retries = 3
} = {}) {
  return { url, method, headers, timeout, retries };
}

console.log(fetchData({ url: "/api/users" }));
// { url: "/api/users", method: "GET", headers: {}, timeout: 5000, retries: 3 }

console.log(fetchData());
// All defaults (empty object fallback prevents error)

// Default expression (function call)
function generateId() {
  return Math.random().toString(36).substr(2, 9);
}
const { id = generateId() } = {};
console.log(id); // Random ID

// Default based on other destructured variable
const { title, prefix = `${title}: ` } = { title: "Hello" };
console.log(prefix); // "Hello: "

// Nested defaults at every level
const config = {
  database: {
    host: undefined
  }
};
const {
  database: {
    host: dbHost = "localhost",
    port: dbPort = 5432
  } = {}
} = config;
console.log(dbHost, dbPort); // "localhost" 5432
```

### Advanced Examples
```javascript
// Conditional defaults
const { role = isAdmin ? "admin" : "user" } = { isAdmin: false };

// Default from environment
const { apiKey = process.env.API_KEY || "dev-key" } = {};

// Cascading defaults
const { a = 1, b = a * 2, c = a + b } = {};
console.log(a, b, c); // 1, 2, 3

// Default with null check
function safeConfig(obj) {
  const {
    host = "localhost",
    port = 3000,
    ...rest
  } = obj ?? {};
  return { host, port, ...rest };
}

// Complex default pattern
const defaults = {
  theme: "light",
  locale: "en-US",
  debug: false
};

const userConfig = { theme: "dark" };
const finalConfig = { ...defaults, ...userConfig };
// Destructuring not needed, but useful for extraction:
const { theme, locale, debug, ...custom } = { ...defaults, ...userConfig };
console.log(theme, locale, debug); // "dark" "en-US" false
```

### Real-World Use Cases
- Configuration objects with optional properties
- API client options with sensible defaults
- Function parameter defaults for options objects
- Redux reducer initial state
- React component default props
- Environment variable fallbacks
- Form field defaults

### Common Mistakes
- Expecting defaults to handle `null` (they only handle `undefined`)
- Forgetting that defaults are only applied when destructuring
- Using mutable objects as defaults (shared reference issue)
- Providing defaults at wrong nesting level
- Complex default expressions that are hard to read

### Best Practices
- Always provide defaults for optional destructured values
- Use `?? null` or explicit null checks for null handling
- Use primitive values for defaults (avoid shared mutable objects)
- Provide empty object fallback for entire destructuring: `= {}`
- Keep default expressions simple
- Use `Object.assign` or spread for merging configurations
- Document default values in JSDoc or comments

### Performance Considerations
- Default values are evaluated lazily (only when needed)
- Default expressions are evaluated at destructuring time
- Avoid expensive operations in default expressions
- Reusing same default object across many destructures may cause memory sharing
- Modern engines optimize default value checks

### Interview Questions
1. When are default values in destructuring applied?
2. Do defaults handle `null` values?
3. Can defaults be expressions or function calls?
4. How do you provide defaults in nested destructuring?
5. How do defaults interact with renaming?

### Coding Challenges
1. Write a function that safely destructures a config object with defaults at each level.
2. Create a function that merges user-provided options with defaults using destructuring.
3. Implement a safe nested property accessor using destructuring with defaults.
4. Write a function that demonstrates the difference between `undefined` and `null` in defaults.

### Related Topics
- Array destructuring
- Object destructuring
- Default parameters
- Nullish coalescing
