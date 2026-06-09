# JSON - JSON.parse(), JSON.stringify(), serialization, reviver/replacer

## Introduction

JSON (JavaScript Object Notation) is a lightweight data-interchange format that has become the standard for API communication, configuration files, and data storage in modern web development. Derived from JavaScript object literal syntax, JSON is language-independent, with parsers available in virtually every programming language. JavaScript provides two built-in methods for JSON handling: `JSON.stringify()` for serialization and `JSON.parse()` for deserialization, along with powerful customization options via reviver and replacer functions.

## JSON.parse()

### What It Is

`JSON.parse()` converts a JSON string into a corresponding JavaScript value or object. It is the inverse of `JSON.stringify()`. The method can optionally transform the resulting object using a reviver function.

```javascript
JSON.parse(text, reviver)
```

### Why It Is Important

`JSON.parse()` is the gateway for consuming JSON data from external sources - API responses, configuration files, WebSocket messages, localStorage data, and any system that uses JSON for data exchange. Without it, JSON data would remain as opaque strings in JavaScript applications.

### How It Works Internally

The parser operates in two phases. First, it tokenizes the JSON string, identifying strings, numbers, booleans, null, arrays, and objects. Then it constructs the corresponding JavaScript value tree. The parser is strict - it enforces JSON syntax rules including double quotes for strings (no single quotes or unquoted keys), proper number formatting, and no trailing commas. If the reviver function is provided, it is called for each value in the tree during a post-parse depth-first traversal.

```javascript
// Internal parse steps:
// 1. Lexical analysis - tokenize the string
// 2. Syntactic analysis - build value tree
// 3. If reviver provided, traverse and transform
```

### Syntax

```javascript
// Basic parsing
JSON.parse('{"name":"John","age":30}')

// With reviver function
JSON.parse('{"name":"John","birth":"2024-01-01"}', (key, value) => {
  if (key === 'birth') return new Date(value)
  return value
})

// Parsing arrays
JSON.parse('[1,2,3]')

// Parsing primitive values
JSON.parse('"hello"')   // "hello"
JSON.parse('42')        // 42
JSON.parse('true')      // true
JSON.parse('null')      // null
```

### Beginner Examples

```javascript
// Parse API response
const jsonString = '{"userId":1,"title":"Hello","completed":false}'
const todo = JSON.parse(jsonString)
console.log(todo.title) // "Hello"

// Parse array of objects
const usersJson = '[{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]'
const users = JSON.parse(usersJson)
users.forEach(u => console.log(u.name))

// Safe parsing with try-catch
function safeParse(jsonString) {
  try {
    return { data: JSON.parse(jsonString), error: null }
  } catch (error) {
    return { data: null, error: error.message }
  }
}

const result = safeParse('{"valid": true}')
if (result.error) {
  console.error('Invalid JSON:', result.error)
}
```

### Intermediate Examples

```javascript
// Reviver function for date revival
const data = '{"name":"Event","date":"2024-06-15T10:30:00Z","tags":["js","json"]}'

const parsed = JSON.parse(data, (key, value) => {
  if (key === 'date' || key === 'createdAt' || key === 'updatedAt') {
    return new Date(value)
  }
  return value
})
console.log(parsed.date.getFullYear()) // 2024

// Reviver for computed properties
const config = '{"baseUrl":"/api","timeout":5000,"retries":3}'

const configObj = JSON.parse(config, (key, value) => {
  if (key === 'timeout' || key === 'retries') {
    return { value, unit: key === 'timeout' ? 'ms' : 'count' }
  }
  return value
})

// Reviver for filtering
const filtered = JSON.parse(
  '{"keep":"this","remove":"that","keep2":"also this"}',
  (key, value) => key.startsWith('remove') ? undefined : value
)
// Result: { keep: "this", keep2: "also this" }

// Parse and validate with schema
function parseAndValidate(jsonString, schema) {
  const data = JSON.parse(jsonString)
  for (const [key, type] of Object.entries(schema)) {
    if (typeof data[key] !== type) {
      throw new Error(`Invalid type for ${key}: expected ${type}`)
    }
  }
  return data
}

const schema = { name: 'string', age: 'number', active: 'boolean' }
const user = parseAndValidate('{"name":"John","age":30,"active":true}', schema)
```

### Advanced Examples

```javascript
// Circular reference detection in parse
function createReviverWithCircularCheck() {
  const visited = new WeakSet()
  return function reviver(key, value) {
    if (typeof value === 'object' && value !== null) {
      if (visited.has(value)) {
        throw new Error('Circular reference detected')
      }
      visited.add(value)
    }
    return value
  }
}

// Reviver for object reconstruction (class instances)
class Person {
  constructor(name, birthDate) {
    this.name = name
    this.birthDate = birthDate
  }
  get age() {
    return new Date().getFullYear() - this.birthDate.getFullYear()
  }
}

const personData = '{"__type":"Person","name":"Alice","birthDate":"1990-05-20"}'

const person = JSON.parse(personData, (key, value) => {
  if (value && value.__type === 'Person') {
    return new Person(value.name, new Date(value.birthDate))
  }
  return value
})

// Streaming JSON parser pattern
function parseJsonStream(stream, reviver) {
  return new Promise((resolve, reject) => {
    const reader = stream.getReader()
    const decoder = new TextDecoder()
    let buffer = ''

    function readChunk() {
      reader.read().then(({ done, value }) => {
        if (done) {
          try {
            resolve(JSON.parse(buffer, reviver))
          } catch (e) {
            reject(e)
          }
          return
        }
        buffer += decoder.decode(value, { stream: true })
        readChunk()
      }).catch(reject)
    }

    readChunk()
  })
}

// Custom JSON parser with error context
function parseWithContext(jsonString) {
  try {
    return JSON.parse(jsonString)
  } catch (error) {
    const position = error.message.match(/position (\d+)/)
    if (position) {
      const pos = parseInt(position[1])
      const start = Math.max(0, pos - 20)
      const end = Math.min(jsonString.length, pos + 20)
      const context = jsonString.slice(start, end)
      const pointer = ' '.repeat(Math.min(20, pos - start)) + '^'
      throw new Error(`JSON parse error at position ${pos}:\n${context}\n${pointer}`)
    }
    throw error
  }
}
```

### Real-World Use Cases

- **API response parsing**: Every HTTP response carrying JSON goes through JSON.parse()
- **Configuration files**: Node.js and browser configs stored as JSON
- **localStorage/sessionStorage**: Data stored as JSON strings
- **WebSocket messages**: Structured data exchanged over WebSockets
- **Structured logging**: Parsing JSON log entries
- **Database exports**: Import/export data in JSON format
- **Internationalization**: i18n translation files in JSON format

### Common Mistakes

```javascript
// Mistake: Parsing without try-catch
const data = JSON.parse(invalidInput) // Throws on invalid JSON

// Mistake: Trailing commas
JSON.parse('{"a":1,}') // SyntaxError

// Mistake: Single quotes
JSON.parse("{'a': 1}") // SyntaxError

// Mistake: Unquoted keys
JSON.parse('{a: 1}') // SyntaxError

// Mistake: Not handling dates properly
const raw = JSON.parse('{"date":"2024-01-01T00:00:00Z"}')
console.log(raw.date.getFullYear()) // Error: getFullYear is not a function

// Mistake: Using undefined as reviver return value means "remove this key"
// but only for object properties, not for arrays
```

### Best Practices

```javascript
// Always wrap parse in try-catch
function safeJsonParse(str) {
  try {
    return JSON.parse(str)
  } catch {
    return null
  }
}

// Use reviver for type conversion
const data = JSON.parse(jsonString, (key, value) => {
  if (typeof value === 'string' && /^\d{4}-\d{2}-\d{2}T/.test(value)) {
    return new Date(value)
  }
  return value
})

// Validate structure after parsing
function parseWithValidation(str, validator) {
  const data = JSON.parse(str)
  if (!validator(data)) throw new Error('Invalid data structure')
  return data
}

// Pre-validate JSON before parse
function isValidJSON(str) {
  try {
    JSON.parse(str)
    return true
  } catch {
    return false
  }
}
```

### Performance Considerations

- JSON.parse() with very large strings can block the main thread (use streaming for huge payloads)
- Reviver functions add O(n) overhead proportional to the number of values in the JSON
- The native JSON.parse() is highly optimized (C++ implementation) - much faster than custom parsers
- Parsing is memory-intensive: the entire string and resulting object tree coexist in memory
- For extremely large JSON (>100MB), consider JSON streaming parsers like Oboe.js or clarinet
- Avoid deep nesting as it increases parse time and memory usage
- Repeated parsing of the same data should be cached

### Interview Questions

**Q: What is the difference between JSON and JavaScript object literals?**

A: JSON is a strict subset of JavaScript object literal syntax. JSON requires double-quoted property names and string values, does not allow trailing commas, does not support comments, and only supports a limited set of data types (objects, arrays, strings, numbers, booleans, null). JavaScript objects support functions, undefined, Symbols, dates, and other types not representable in JSON.

**Q: How does the reviver function in JSON.parse() work?**

A: The reviver is called for every value in the parsed object tree during a depth-first traversal. It receives the key and value. The return value replaces the original value. If the reviver returns undefined, the property is deleted from the object (but not from arrays - array elements become undefined). The root object is called with key = '' and the root value.

**Q: Can JSON.parse() handle BigInt values?**

A: No. JSON does not have a BigInt type. Attempting to parse a large integer may lose precision. The TC39 proposal for JSON.parse with a source text accessor (toSource) would allow revivers to access the original string representation to handle BigInt conversion manually.

### Coding Challenges

```javascript
// Challenge 1: Deep clone using JSON (with caveats)
function deepClone(obj) {
  return JSON.parse(JSON.stringify(obj))
}
// Note: Fails for dates, functions, undefined, symbols, circular refs

// Challenge 2: Parse JSON with BigInt support
function parseBigIntJSON(jsonString) {
  return JSON.parse(jsonString, (key, value) => {
    if (typeof value === 'number' && !Number.isSafeInteger(value)) {
      return BigInt(value)
    }
    // Or for string-encoded bigints
    if (typeof value === 'string' && /^\d+n$/.test(value)) {
      return BigInt(value.slice(0, -1))
    }
    return value
  })
}

// Challenge 3: JSON parse with path tracking in reviver
function parseWithPath(jsonString) {
  const paths = []
  return JSON.parse(jsonString, function reviver(key, value) {
    if (key !== '') {
      const parentPath = this._path || ''
      this._path = parentPath ? `${parentPath}.${key}` : key
    }
    paths.push({ path: this._path || '$', value })
    return value
  })
}
```

## JSON.stringify()

### What It Is

`JSON.stringify()` converts a JavaScript value to a JSON string representation. It accepts optional replacer function or array for selective serialization, and a space parameter for formatting.

```javascript
JSON.stringify(value, replacer, space)
```

### Why It Is Important

`JSON.stringify()` enables JavaScript data to be transmitted over networks, stored in localStorage, logged to consoles, and exchanged with other systems. It is the standard serialization mechanism for JavaScript applications and is critical for API communication, state persistence, and data export.

### How It Works Internally

The serializer traverses the JavaScript object graph recursively, converting values according to JSON rules. It calls toJSON() on objects that define it, applies the replacer function/array, and handles circular reference detection (throwing an error). Special JavaScript values like undefined, functions, and symbols are omitted (or replaced with null in arrays). Dates are converted to ISO strings.

### Syntax

```javascript
JSON.stringify(value)
JSON.stringify(value, replacer)
JSON.stringify(value, replacer, space)

// replacer as array of keys to include
JSON.stringify(obj, ['name', 'age'])

// replacer as function
JSON.stringify(obj, (key, value) => {
  if (key.startsWith('_')) return undefined
  return value
})

// space for formatting
JSON.stringify(obj, null, 2)   // 2-space indent
JSON.stringify(obj, null, '\t') // tab indent
```

### Beginner Examples

```javascript
// Basic serialization
const obj = { name: 'John', age: 30, active: true }
console.log(JSON.stringify(obj)) // '{"name":"John","age":30,"active":true}'

// With formatting
const user = { id: 1, profile: { name: 'Alice', email: 'a@b.com' } }
console.log(JSON.stringify(user, null, 2))

// Array serialization
const items = [1, 'two', { three: 3 }]
console.log(JSON.stringify(items)) // '[1,"two",{"three":3}]'

// Primitive values
console.log(JSON.stringify('hello')) // '"hello"'
console.log(JSON.stringify(42))      // '42'
console.log(JSON.stringify(true))    // 'true'
console.log(JSON.stringify(null))    // 'null'
```

### Intermediate Examples

```javascript
// Replacer function to exclude private properties
const user = {
  id: 1,
  name: 'Alice',
  _password: 'secret123',
  email: 'alice@example.com'
}

const json = JSON.stringify(user, (key, value) => {
  if (key.startsWith('_')) return undefined
  if (key === 'password') return undefined
  return value
})

// Replacer array to whitelist properties
const json2 = JSON.stringify(user, ['id', 'name', 'email'])

// Serialize dates
const event = {
  title: 'Meeting',
  date: new Date('2024-06-15T10:00:00Z'),
  attendees: ['Alice', 'Bob']
}

console.log(JSON.stringify(event))
// {"title":"Meeting","date":"2024-06-15T10:00:00.000Z","attendees":["Alice","Bob"]}

// Custom toJSON method
class Product {
  constructor(id, name, price) {
    this.id = id
    this.name = name
    this.price = price
  }

  toJSON() {
    return {
      id: this.id,
      name: this.name.toUpperCase(),
      price: `$${this.price.toFixed(2)}`
    }
  }
}

const product = new Product(1, 'Widget', 19.99)
console.log(JSON.stringify(product))
// {"id":1,"name":"WIDGET","price":"$19.99"}
```

### Advanced Examples

```javascript
// Pretty printer with colored output for debugging
function prettyPrint(obj, colorize = false) {
  const json = JSON.stringify(obj, null, 2)
  if (!colorize) return json

  return json.replace(/("(?:\\.|[^"\\])*")\s*:/g, '\x1b[36m$1\x1b[0m:')
    .replace(/"((?:\\.|[^"\\])*)"/g, '\x1b[32m"$1"\x1b[0m')
    .replace(/\b(true|false)\b/g, '\x1b[35m$1\x1b[0m')
    .replace(/\b(\d+\.?\d*)\b/g, '\x1b[33m$1\x1b[0m')
}

// Safe stringify that handles circular references
function safeStringify(obj, space) {
  const seen = new WeakSet()

  return JSON.stringify(obj, (key, value) => {
    if (typeof value === 'object' && value !== null) {
      if (seen.has(value)) {
        return '[Circular]'
      }
      seen.add(value)
    }
    return value
  }, space)
}

// Stringify with custom serialization for special types
function enhancedStringify(obj, space) {
  return JSON.stringify(obj, (key, value) => {
    if (value instanceof Map) {
      return { __type: 'Map', value: [...value.entries()] }
    }
    if (value instanceof Set) {
      return { __type: 'Set', value: [...value] }
    }
    if (typeof value === 'bigint') {
      return { __type: 'BigInt', value: value.toString() }
    }
    if (value instanceof RegExp) {
      return { __type: 'RegExp', value: value.source, flags: value.flags }
    }
    return value
  }, space)
}

// Stringify with sorted keys for deterministic output
function sortedStringify(obj, space) {
  function sortKeys(obj) {
    if (obj === null || typeof obj !== 'object' || Array.isArray(obj)) {
      return obj
    }
    return Object.keys(obj).sort().reduce((acc, key) => {
      acc[key] = sortKeys(obj[key])
      return acc
    }, {})
  }

  return JSON.stringify(sortKeys(obj), null, space)
}

// Streaming stringify for large objects
function* stringifyChunks(value, chunkSize = 1000) {
  const str = JSON.stringify(value)
  for (let i = 0; i < str.length; i += chunkSize) {
    yield str.slice(i, i + chunkSize)
  }
}

for (const chunk of stringifyChunks(largeObject)) {
  process.stdout.write(chunk)
}
```

### Real-World Use Cases

- **API request bodies**: Serialize JavaScript objects for HTTP requests
- **localStorage persistence**: Store complex state as JSON strings
- **Structured logging**: Log objects as JSON strings for parsing
- **State serialization**: Redux/Vuex state persistence and hydration
- **Configuration export**: Export application settings as JSON files
- **Deep cloning**: Simple (but limited) deep clone with JSON.parse(JSON.stringify(obj))
- **Cache serialization**: Store computed results in cache systems

### Common Mistakes

```javascript
// Mistake: Expecting functions to survive round-trip
const obj = { fn: () => console.log('hello') }
JSON.stringify(obj) // '{}' - functions are omitted

// Mistake: Expecting undefined to survive
const obj2 = { a: undefined, b: 1 }
JSON.stringify(obj2) // '{"b":1}' - undefined keys omitted

// Mistake: Expecting Symbol keys to survive
const obj3 = { [Symbol('id')]: 1 }
JSON.stringify(obj3) // '{}' - Symbol keys omitted

// Mistake: Circular reference
const circular = { a: 1 }
circular.self = circular
JSON.stringify(circular) // TypeError: Converting circular structure to JSON

// Mistake: NaN and Infinity become null
JSON.stringify({ x: NaN, y: Infinity, z: -Infinity })
// '{"x":null,"y":null,"z":null}'

// Mistake: BigInt is not supported
JSON.stringify({ big: 1n }) // TypeError: Do not know how to serialize a BigInt
```

### Best Practices

```javascript
// Use replacer for safe serialization
function serializeUser(user) {
  return JSON.stringify(user, ['id', 'name', 'email', 'createdAt'])
}

// Define toJSON on classes for controlled serialization
class APIResponse {
  constructor(data, meta) {
    this.data = data
    this.meta = meta
  }

  toJSON() {
    return {
      ...this.data,
      _meta: this.meta
    }
  }
}

// Use space for readable config files
const configJSON = JSON.stringify(config, null, 2)

// Handle serialization errors gracefully
function safeStringify(value, fallback = '{}') {
  try {
    return JSON.stringify(value)
  } catch {
    return fallback
  }
}
```

### Performance Considerations

- Stringify time is O(n) where n is the number of values in the object graph
- Deeply nested objects increase stringify time due to recursive traversal
- The `space` parameter adds O(depth * indent) overhead to output size
- Replacer functions add per-value overhead - use array replacer for simple whitelisting
- Large strings in objects are copied, increasing memory usage
- Creating intermediate string representations via toJSON() adds overhead
- Native JSON.stringify is highly optimized in V8 (faster than any userland alternative)

### Interview Questions

**Q: What values does JSON.stringify() omit or convert?**

A: Functions, undefined (when used as property values), Symbol keys, and properties with undefined values are omitted entirely. NaN and Infinity become null. Dates convert to ISO strings. BigInt throws a TypeError. Circular references throw an error.

**Q: How does the replacer function work in JSON.stringify()?**

A: The replacer receives (key, value) for each value in the object tree. The return value replaces the original. Return undefined to omit the property. For the root object, key is ''. The replacer is called after toJSON() transformations.

**Q: What is the difference between replacer as array vs function?**

A: An array replacer specifies a whitelist of property names to include. A function replacer provides full control: it can transform values, filter properties based on key or value, and handle specific types differently.

### Coding Challenges

```javascript
// Challenge 1: Stringify with path information in replacer
function stringifyWithPaths(obj) {
  const paths = []
  JSON.stringify(obj, function(key, value) {
    const parentPath = this._path || '$'
    const path = key ? `${parentPath}.${key}` : '$'
    paths.push({ path, type: typeof value, value })
    return value
  })
  return paths
}

// Challenge 2: JSON minifier
function minifyJSON(jsonString) {
  return JSON.stringify(JSON.parse(jsonString))
}

// Challenge 3: Stringify error objects
function errorStringify(error) {
  return JSON.stringify({
    ...error,
    name: error.name,
    message: error.message,
    stack: error.stack
  })
}
```

## Serialization rules

### What It Is

JSON serialization defines how JavaScript values map to JSON values. Understanding these rules is essential for creating reliable data exchange formats. Certain JavaScript values have special conversion behavior, and some cannot be represented in JSON at all.

### Serialization rules table

```javascript
// JavaScript -> JSON mapping
String        -> JSON string (escaped)
Number        -> JSON number (finite) or null (NaN, Infinity)
Boolean       -> JSON boolean
null          -> JSON null
Object        -> JSON object (after toJSON, replacer)
Array         -> JSON array (sparse arrays fill holes with null)
Date          -> JSON string (toISOString())
RegExp        -> JSON string (regex source) or {} if serialized directly
Function      -> omitted (or null in arrays)
undefined     -> omitted (property) or null (array element)
Symbol        -> omitted (property) or null (array element)
Map           -> {} (unless custom toJSON)
Set           -> {} (unless custom toJSON)
BigInt        -> TypeError
Error         -> {} (unless enumerable properties added)
```

### Beginner Examples

```javascript
// Numbers
console.log(JSON.stringify(42))          // '42'
console.log(JSON.stringify(3.14))        // '3.14'
console.log(JSON.stringify(NaN))         // 'null'
console.log(JSON.stringify(Infinity))    // 'null'

// Strings with special characters
console.log(JSON.stringify('hello "world"'))  // '"hello \\"world\\""'
console.log(JSON.stringify('line\nbreak'))     // '"line\\nbreak"'

// Arrays with holes
const arr = [1, , 3]
console.log(JSON.stringify(arr)) // '[1,null,3]'
```

### Performance Considerations

- Serialization of large arrays is slower than equivalent-sized objects
- String escaping adds overhead proportional to the number of special characters
- Deep object graphs can cause stack overflow in extreme cases
- For performance-critical serialization, consider schema-based serializers like protobuf

### Best Practices

```javascript
// Always know what types are in your data
function assertSerializable(value) {
  if (typeof value === 'bigint') throw new Error('BigInt not serializable')
  if (typeof value === 'symbol') throw new Error('Symbol not serializable')
  if (typeof value === 'function') throw new Error('Function not serializable')
  if (value !== null && typeof value === 'object') {
    Object.keys(value).forEach(k => assertSerializable(value[k]))
  }
}
```

## reviver and replacer functions

### What It Is

Reviver (JSON.parse) and replacer (JSON.stringify) are callback functions that enable transformation during parsing and serialization. They provide control over the serialization/deserialization process at every level of the object tree.

### Why It Is Important

These functions enable type reconstruction (e.g., reviving dates), data sanitization (excluding sensitive fields), schema migration (transforming legacy data formats), and selective serialization (whitelisting/blacklisting properties). They are essential for creating robust JSON pipelines.

### How It Works Internally

The reviver is called bottom-up during parsing: leaf values are processed first, then their parent objects. The replacer is called top-down during stringification: the root is processed first, then its children. Both functions receive (key, value) and `this` is the parent container.

### Syntax

```javascript
// Reviver - called during parse for each value
JSON.parse(text, function(key, value) {
  // this = parent object (or root for initial call)
  // key = property name (or '' for root)
  // value = current value
  // Return transformed value, or undefined to delete
})

// Replacer - called during stringify for each value
JSON.stringify(value, function(key, value) {
  // this = parent object
  // Return transformed value, or undefined to omit
})
```

### Interview Questions

**Q: What is the order of execution for reviver and replacer?**

A: Reviver processes bottom-up (leaves first, then parents) during JSON.parse. Replacer processes top-down (root first, then children) during JSON.stringify. The root object is always called with key = ''.

**Q: What happens when reviver returns undefined?**

A: Returning undefined from the reviver for an object property deletes that property from the result. For arrays, the element becomes the array's undefined value (sparse array effect). The root object cannot be deleted by returning undefined.

### Related Topics

- `Ajax` - JSON is the primary data format for AJAX responses
- `Fetch API` - .json() method automatically parses JSON responses
- `localStorage` - Data stored as JSON strings in browser storage
- `REST APIs` - JSON is the standard format for REST API communication
- `WebSockets` - Messages often exchanged as JSON strings
- `Deep cloning` - JSON.parse(JSON.stringify()) for simple deep copies
- `TypeScript` - JSON typing and structural validation
