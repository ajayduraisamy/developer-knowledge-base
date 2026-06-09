# Conditions - if/else, switch, ternary, truthy/falsy

## Introduction
Conditional statements control the flow of execution based on boolean conditions. JavaScript provides `if/else`, `switch`, ternary expressions, and relies on truthy/falsy coercion for condition evaluation. Mastering conditions is essential for writing logical, branching program flow.

## if/else statements

### What It Is
The `if` statement executes a block of code if a specified condition is truthy. Optional `else if` and `else` blocks handle alternative conditions and default cases.

### Why It Is Important
`if/else` is the most fundamental control flow mechanism. It enables decision-making, validation, error handling, and state-dependent logic. Understanding truthiness and proper structuring of conditions is critical.

### How It Works Internally
The condition is evaluated by the engine using the ToBoolean abstract operation. If truthy, the first block executes. Otherwise, subsequent `else if` conditions are evaluated sequentially. If none match, the `else` block (if present) executes. The engine compiles conditional branches into jump instructions.

### Syntax
```javascript
if (condition) {
  // Executes if condition is truthy
} else if (anotherCondition) {
  // Executes if first condition is falsy and anotherCondition is truthy
} else {
  // Executes if all conditions are falsy
}

// Single-line (omit braces for single statement, use sparingly)
if (condition) doSomething();
```

### Beginner Examples
```javascript
// Basic if
let age = 20;
if (age >= 18) {
  console.log("Adult");
}

// if/else
let temperature = 30;
if (temperature > 25) {
  console.log("Hot day");
} else {
  console.log("Cool day");
}

// if/else if/else
let score = 85;
if (score >= 90) {
  console.log("A");
} else if (score >= 80) {
  console.log("B");
} else if (score >= 70) {
  console.log("C");
} else {
  console.log("Failing grade");
}

// Truthy/falsy conditions
let name = "";
if (name) {
  console.log(`Hello, ${name}`);
} else {
  console.log("Name is empty");
}
```

### Intermediate Examples
```javascript
// Nested conditions
let user = { role: "admin", active: true };
if (user) {
  if (user.active) {
    if (user.role === "admin") {
      console.log("Admin dashboard");
    } else {
      console.log("User dashboard");
    }
  } else {
    console.log("Account deactivated");
  }
} else {
  console.log("User not found");
}

// Early return pattern (guard clauses)
function processOrder(order) {
  if (!order) {
    return { error: "Invalid order" };
  }
  if (!order.items || order.items.length === 0) {
    return { error: "Order has no items" };
  }
  if (!order.customer) {
    return { error: "No customer specified" };
  }
  // Process the order...
  return { success: true, total: calculateTotal(order) };
}

// Combining conditions
function canAccess(role, isActive, hasPermission) {
  if (role === "admin" && isActive) {
    return true;
  }
  if (role === "editor" && isActive && hasPermission) {
    return true;
  }
  return false;
}
```

### Advanced Examples
```javascript
// Dynamic condition evaluation
const conditions = {
  isLoggedIn: checkAuth(),
  hasPermission: checkPermission("write"),
  isOwner: checkOwnership()
};

if (Object.values(conditions).every(Boolean)) {
  console.log("Full access granted");
}

// Strategy pattern with conditions
function getShippingCost(strategy) {
  const strategies = {
    standard: () => 5.99,
    express: () => 14.99,
    overnight: () => 29.99,
    free: () => 0
  };
  const fn = strategies[strategy];
  if (fn) {
    return fn();
  }
  throw new Error(`Unknown shipping strategy: ${strategy}`);
}

// Short-circuit with assignment
const config = {};
if (!config.timeout) config.timeout = 5000;
if (!config.retries) config.retries = 3;

// Conditional execution based on type
function handle(input) {
  if (typeof input === "string") {
    return handleString(input);
  }
  if (typeof input === "number") {
    return handleNumber(input);
  }
  if (Array.isArray(input)) {
    return handleArray(input);
  }
  if (input && typeof input === "object") {
    return handleObject(input);
  }
  throw new Error(`Unsupported input type: ${typeof input}`);
}
```

### Real-World Use Cases
- Form validation (checking required fields, formats)
- Authentication and authorization
- Feature flag evaluation
- API response handling (success vs error)
- Permission checking in dashboards
- Multi-step form wizard navigation
- E-commerce (discount eligibility, stock availability)
- Game logic (collision detection, win conditions)

### Common Mistakes
- Using assignment `=` instead of comparison `===` in condition
- Not accounting for all possible else-if branches
- Deep nesting (more than 3 levels) reducing readability
- Forgetting to return/break after conditional blocks
- Using mutable conditions that change during evaluation

### Best Practices
- Use guard clauses (early return) to reduce nesting
- Extract complex conditions into named boolean variables
- Keep condition blocks focused on a single responsibility
- Prefer `===` over `==` in conditions
- Consider switch statement for many discrete value checks
- Avoid negative conditions where possible (e.g., `isActive` over `!isInactive`)
- Use consistent brace style (always use braces, even for single statements)

### Performance Considerations
- `if/else` performance is similar across all branching constructs
- Modern JIT optimizes predictable branches (branch prediction)
- Frequently-true branches are optimized better than random branches
- Deeply nested conditions can cause cache misses
- Extracting conditions to functions has negligible overhead

### Interview Questions
1. What is the difference between `if (x)` and `if (x == true)`?
2. Explain the guard clause pattern.
3. How do you avoid deep nesting in conditional logic?
4. What is the difference between `else if` and nested `if`?
5. Can you have multiple `else` blocks?

### Coding Challenges
1. Refactor a deeply nested if/else chain using guard clauses.
2. Implement a discount calculator using nested conditions.
3. Write a validation function that checks multiple conditions with meaningful error messages.

### Related Topics
- Truthy and falsy values
- Switch statement
- Ternary operator
- Short-circuit evaluation

## switch statement

### What It Is
The `switch` statement evaluates an expression and executes code blocks based on matching case values. It is typically used for discrete value comparison instead of long if/else chains.

### Why It Is Important
`switch` provides cleaner syntax for multi-way branching when comparing a single value against many possible matches. It improves readability over lengthy if/else chains and can be faster in some implementations.

### How It Works Internally
The switch expression is evaluated once. The engine compares the result against each `case` value using strict equality (`===`). When a match is found, execution jumps to that case block and continues until a `break` or the end of the switch. Without `break`, execution "falls through" to the next case.

### Syntax
```javascript
switch (expression) {
  case value1:
    // Code for value1
    break;
  case value2:
    // Code for value2
    break;
  default:
    // Default code if no match
}
```

### Beginner Examples
```javascript
// Basic switch
let day = 3;
let dayName;
switch (day) {
  case 1:
    dayName = "Monday";
    break;
  case 2:
    dayName = "Tuesday";
    break;
  case 3:
    dayName = "Wednesday";
    break;
  case 4:
    dayName = "Thursday";
    break;
  case 5:
    dayName = "Friday";
    break;
  case 6:
    dayName = "Saturday";
    break;
  case 7:
    dayName = "Sunday";
    break;
  default:
    dayName = "Invalid day";
}
console.log(dayName); // "Wednesday"

// Switch with string
let fruit = "apple";
switch (fruit) {
  case "banana":
    console.log("Yellow");
    break;
  case "apple":
  case "cherry":
    console.log("Red");
    break;
  case "orange":
    console.log("Orange");
    break;
  default:
    console.log("Unknown color");
}
```

### Intermediate Examples
```javascript
// Fall-through intentionally for shared logic
let grade = "B";
switch (grade) {
  case "A":
  case "B":
    console.log("Pass with honors");
    break;
  case "C":
    console.log("Pass");
    break;
  case "D":
  case "F":
    console.log("Fail");
    break;
  default:
    console.log("Invalid grade");
}

// Switch with boolean conditions (use if/else instead)
function getDrink(age) {
  switch (true) {
    case age < 13:
      return "Juice";
    case age < 18:
      return "Soda";
    case age >= 18:
      return "Beer";
    default:
      return "Water";
  }
}

// Using return instead of break (in functions)
function getType(value) {
  switch (typeof value) {
    case "string": return "String";
    case "number": return "Number";
    case "boolean": return "Boolean";
    case "undefined": return "Undefined";
    case "object":
      if (value === null) return "Null";
      if (Array.isArray(value)) return "Array";
      return "Object";
    default: return "Unknown";
  }
}
```

### Advanced Examples
```javascript
// Enum-like pattern
const OrderStatus = Object.freeze({
  PENDING: "pending",
  CONFIRMED: "confirmed",
  SHIPPED: "shipped",
  DELIVERED: "delivered",
  CANCELLED: "cancelled"
});

function handleOrderStatus(status) {
  switch (status) {
    case OrderStatus.PENDING:
      return sendConfirmation();
    case OrderStatus.CONFIRMED:
      return prepareForShipping();
    case OrderStatus.SHIPPED:
      return trackShipment();
    case OrderStatus.DELIVERED:
      return requestFeedback();
    case OrderStatus.CANCELLED:
      return processRefund();
    default:
      throw new Error(`Unknown status: ${status}`);
  }
}

// Switch in reducer (Redux pattern)
function reducer(state, action) {
  switch (action.type) {
    case "ADD_TODO":
      return { ...state, todos: [...state.todos, action.payload] };
    case "TOGGLE_TODO":
      return {
        ...state,
        todos: state.todos.map(t =>
          t.id === action.payload ? { ...t, done: !t.done } : t
        )
      };
    case "DELETE_TODO":
      return {
        ...state,
        todos: state.todos.filter(t => t.id !== action.payload)
      };
    default:
      return state;
  }
}

// Dynamic case matching
function getErrorMessage(code) {
  const messages = {
    400: "Bad Request",
    401: "Unauthorized",
    403: "Forbidden",
    404: "Not Found",
    500: "Internal Server Error"
  };
  return messages[code] ?? "Unknown Error";
}
```

### Real-World Use Cases
- HTTP status code handling
- State machine transitions
- Redux reducers (action type dispatching)
- Command pattern implementations
- Menu/command navigation
- Protocol message parsing
- Event type handling in game engines

### Common Mistakes
- Forgetting `break`, causing unintended fall-through
- Not including a `default` case
- Using switch for range comparisons (use if/else)
- Complex case expressions (they are compared with strict equality)
- Forgetting that `case` values are evaluated at switch time
- Using switch with non-discrete values

### Best Practices
- Always include a `default` case
- Use `break` for each case (or `return` in functions)
- Comment intentional fall-through: `// fallthrough`
- Keep case bodies simple (extract complex logic to functions)
- Use switch for 3+ discrete value comparisons
- Consider object literal dispatch as an alternative to switch
- Use `default` for error handling or unexpected values

### Performance Considerations
- Switch can be faster than if/else chains (jump table optimization)
- Engines may optimize switch into binary search or hash tables
- Performance difference is negligible for small numbers of cases
- Object literal dispatch is often as fast or faster

### Interview Questions
1. What happens if you omit `break` in a switch case?
2. Can you use expressions in `case` clauses?
3. How does `switch` compare to `if/else` chains?
4. What is the purpose of the `default` clause?
5. Can you use `switch` with strings?

### Coding Challenges
1. Convert a long if/else chain to a switch statement.
2. Implement a state machine using switch.
3. Write a calculator using switch that handles +, -, *, /.
4. Create a switch-based HTTP status code handler.

### Related Topics
- if/else statements
- Object literal dispatch
- State machines
- Redux reducers

## Ternary operator

See section in Operators file (05_operators.md) for full coverage.

### What It Is
The ternary operator (`? :`) is a concise conditional expression that returns one of two values based on a condition.

### Why It Is Important
It provides inline conditional logic for assignments, returns, and JSX, reducing boilerplate compared to if/else.

### How It Works Internally
The condition is evaluated via ToBoolean. If truthy, the first expression is evaluated and returned; otherwise the second. The unselected branch is never evaluated.

### Syntax
```javascript
condition ? exprIfTrue : exprIfFalse;
```

### Beginner Examples
```javascript
let age = 20;
let canVote = age >= 18 ? "Yes" : "No";
console.log(canVote); // "Yes"

// In template literals
let message = `You are ${age >= 18 ? "an adult" : "a minor"}.`;

// Conditional function call
let result = isValid ? process() : reject();
```

### Real-World Use Cases
- Conditional JSX in React
- Class name toggling
- Dynamic default values
- Conditional route selection

### Common Mistakes and Best Practices
See full coverage in [05_operators.md](#ternary-operator).

## Truthy and falsy values

### What It Is
In JavaScript, every value has an inherent boolean "truthiness" when evaluated in a boolean context. Falsy values coerce to `false`; truthy values coerce to `true`. This concept is central to condition evaluation, logical operators, and loops.

### Why It Is Important
Understanding truthiness is essential because all conditions in JavaScript depend on it. It enables concise patterns (guard clauses, default values) but also causes bugs when specific falsy values (0, "") are unexpected.

### How It Works Internally
The ToBoolean abstract operation determines truthiness. It simply checks if a value is in the list of falsy values: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`. Everything else is truthy. The engine applies ToBoolean in `if`, `while`, `for`, `&&`, `||`, `!`, and ternary conditions.

### Syntax
```javascript
// Falsy values - all of these coerce to false
false
0, -0, 0n
"" (empty string)
null
undefined
NaN

// Everything else is truthy, including:
true
1, -1, 3.14, Infinity, -Infinity
"hello", "false", "0" (non-empty strings)
[], {} (empty arrays and objects)
function() {}, () => {}
Symbol()
```

### Beginner Examples
```javascript
// Checking truthiness
if ("hello") console.log("truthy");   // "truthy"
if (0) console.log("falsy");           // (no output)
if ([]) console.log("empty array");    // "empty array" (truthy!)
if ({}) console.log("empty object");   // "empty object" (truthy!)

// Common falsy checks
let name = "";
if (name) {
  console.log("Has name");
} else {
  console.log("No name"); // This runs
}

// The Boolean() function
console.log(Boolean("hello"));   // true
console.log(Boolean(0));          // false
console.log(Boolean([]));         // true
console.log(Boolean("false"));    // true (non-empty string!)
```

### Intermediate Examples
```javascript
// Truthiness in logical operators
let value = 0 || "default";
console.log(value); // "default" (0 is falsy)

let count = 0;
let displayCount = count || "No items";
console.log(displayCount); // "No items" (unexpected if 0 is valid)

// Fix with nullish coalescing
let displayCount2 = count ?? "No items";
console.log(displayCount2); // 0 (0 is not null/undefined)

// Array emptiness check
if (arr.length) { // Only truthy if array has elements
  console.log("Array has items");
}

// Object emptiness check (no direct way)
let obj = {};
if (Object.keys(obj).length === 0) {
  console.log("Object is empty");
}

// Combining truthy checks
if (user && user.name && user.email) {
  console.log("User is complete");
}
```

### Advanced Examples
```javascript
// Custom truthiness check (impossible to change)
function isTruthy(value) {
  // This is how JavaScript determines truthiness internally
  if (value === false) return false;
  if (value === null) return false;
  if (value === undefined) return false;
  if (typeof value === "number" && (value === 0 || Object.is(value, -0))) return false;
  if (typeof value === "bigint" && value === 0n) return false;
  if (typeof value === "string" && value === "") return false;
  if (typeof value === "object" && Object.is(value, NaN)) return false;
  return true;
}

// Truthy check pitfalls
const allFalsy = [false, 0, -0, 0n, "", null, undefined, NaN];
allFalsy.forEach(v => {
  if (v) {
    console.log("This should not print");
  } else {
    console.log(`${String(v)} is falsy`);
  }
});

// The !! (double NOT) operator
console.log(!!"hello");  // true
console.log(!!0);         // false
console.log(!!"");        // false
console.log(!!NaN);       // false

// Practical: filtering truthy values
const mixedArray = [0, 1, "", "hello", null, undefined, false, true, {}];
const truthyValues = mixedArray.filter(Boolean);
console.log(truthyValues); // [1, "hello", true, {}]
```

### Real-World Use Cases
- Checking if a user exists: `if (user)`
- Checking if an array has items: `if (arr.length)`
- Conditional default values: `const name = input || "Guest"`
- Feature flags: `if (featureEnabled)`
- Validating optional configuration
- Form field validation (required fields)
- Conditional rendering in frameworks
- Guarding against null/undefined access

### Common Mistakes
- Assuming empty array `[]` is falsy (it's truthy!)
- Assuming empty object `{}` is falsy (it's truthy!)
- Using `||` for defaults when `0` or `""` are valid values
- Confusing `false` (boolean) with falsy values
- Not knowing that `"false"` is truthy (non-empty string)
- Using `if (value === true)` instead of `if (value)` for truthy checks

### Best Practices
- Use `if (value)` for existence checks (null/undefined)
- Use `if (arr.length)` to check for non-empty arrays
- Use `if (Object.keys(obj).length)` for non-empty objects
- Use `??` (nullish coalescing) when 0, "", and false are valid
- Use `!!` to explicitly coerce to boolean when needed
- Be explicit about which falsy values you expect
- Use `Boolean()` constructor for clarity: `arr.filter(Boolean)`

### Performance Considerations
- Truthiness checks are extremely fast (single CPU instruction)
- `if (value)` is faster than `if (value !== null && value !== undefined)`
- `!!value` is slightly faster than `Boolean(value)`
- No performance concern in any realistic scenario

### Interview Questions
1. What are the falsy values in JavaScript?
2. Is an empty array truthy or falsy?
3. Why is `"false"` considered truthy?
4. How would you check if a value is truly empty (null/undefined)?
5. What is the difference between `||` and `??` regarding falsy values?

### Coding Challenges
1. Write a function that returns an array of all falsy values from an input array.
2. Implement a function that safely extracts a value based on truthiness.
3. Create a deep truthy check utility that recursively validates object properties.
4. Write a function that demonstrates all falsy values behave the same in boolean context.

### Related Topics
- Boolean type
- Logical operators
- Type coercion
- Nullish coalescing (??)
