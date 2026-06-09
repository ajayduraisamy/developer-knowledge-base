# Strategy Pattern - Interchangeable algorithms, strategy selection, function-based strategies

## Introduction

The Strategy Pattern defines a family of interchangeable algorithms, encapsulates each one, and makes them interchangeable at runtime. The pattern lets the algorithm vary independently from the clients that use it. In JavaScript, this pattern is exceptionally natural due to first-class functions — strategies can be simple functions, objects with methods, or classes.

This file covers three core aspects: how interchangeable algorithms enable flexible design, how strategy selection works at runtime, and how JavaScript's first-class functions enable lightweight function-based strategies without class overhead.

## Interchangeable Algorithms

### What It Is

Interchangeable algorithms are different implementations of the same conceptual operation that can be swapped at runtime without changing the code that uses them. Each algorithm is encapsulated as a separate "strategy" that conforms to the same interface or contract.

### Why It Is Important

Hard-coding algorithms leads to rigid, difficult-to-maintain code with complex conditional logic. The Strategy Pattern enables:
- **Open/Closed Principle**: Classes are open for extension but closed for modification
- **Runtime flexibility**: Change behavior without recompiling or modifying existing code
- **Separation of concerns**: Each algorithm is isolated and independently testable
- **Elimination of conditionals**: Replace if/else and switch chains with lookup tables

### How It Works Internally

The pattern involves three participants:
1. **Context**: The object that uses a strategy. It holds a reference to a strategy object.
2. **Strategy Interface**: The common contract all strategies implement.
3. **Concrete Strategies**: Individual implementations of the algorithm.

The context delegates algorithm execution to the current strategy object. Changing the strategy at runtime is as simple as replacing the context's strategy reference.

### Syntax

```javascript
// Strategy interface (implicit in JavaScript)
interface SortStrategy {
  sort(data: number[]): number[];
}

// Concrete strategies
class BubbleSort {
  sort(data) { /* ... */ }
}

class QuickSort {
  sort(data) { /* ... */ }
}

// Context
class Sorter {
  constructor(strategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy) {
    this.strategy = strategy;
  }

  sort(data) {
    return this.strategy.sort(data);
  }
}
```

### Beginner Examples

```javascript
// Payment processing strategies
const creditCardPayment = {
  type: "credit_card",
  process(amount) {
    console.log(`Processing credit card payment of $${amount}`);
    return { success: true, transactionId: "CC-" + Date.now() };
  }
};

const paypalPayment = {
  type: "paypal",
  process(amount) {
    console.log(`Processing PayPal payment of $${amount}`);
    return { success: true, transactionId: "PP-" + Date.now() };
  }
};

const cryptoPayment = {
  type: "crypto",
  process(amount) {
    console.log(`Processing crypto payment of $${amount}`);
    return { success: true, transactionId: "CR-" + Date.now() };
  }
};

class PaymentProcessor {
  constructor() {
    this.strategies = {
      credit_card: creditCardPayment,
      paypal: paypalPayment,
      crypto: cryptoPayment
    };
  }

  processPayment(type, amount) {
    const strategy = this.strategies[type];
    if (!strategy) throw new Error(`Unsupported payment type: ${type}`);
    return strategy.process(amount);
  }
}

const processor = new PaymentProcessor();
processor.processPayment("paypal", 99.99);
processor.processPayment("credit_card", 149.99);

// Shipping cost calculation
const shippingStrategies = {
  standard: { cost: (weight) => weight * 1.5, estimate: () => "5-7 business days" },
  express: { cost: (weight) => weight * 3.0, estimate: () => "2-3 business days" },
  overnight: { cost: (weight) => weight * 5.0 + 10, estimate: () => "Next business day" }
};

function calculateShipping(strategyName, weight) {
  const strategy = shippingStrategies[strategyName];
  if (!strategy) throw new Error("Unknown shipping method");
  return {
    cost: strategy.cost(weight),
    delivery: strategy.estimate()
  };
}

console.log(calculateShipping("express", 5));
// { cost: 15, delivery: "2-3 business days" }
```

### Intermediate Examples

```javascript
// Validation strategies for forms
const validators = {
  required: {
    validate: (value) => value !== null && value !== undefined && value !== "",
    message: "This field is required"
  },
  email: {
    validate: (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
    message: "Invalid email address"
  },
  minLength: (min) => ({
    validate: (value) => value.length >= min,
    message: `Must be at least ${min} characters`
  }),
  maxLength: (max) => ({
    validate: (value) => value.length <= max,
    message: `Must be at most ${max} characters`
  }),
  numeric: {
    validate: (value) => /^\d+$/.test(value),
    message: "Must be a number"
  },
  pattern: (regex) => ({
    validate: (value) => regex.test(value),
    message: `Must match pattern ${regex}`
  })
};

class FormValidator {
  constructor(rules) {
    this.rules = rules;
  }

  validate(data) {
    const errors = {};

    for (const [field, fieldRules] of Object.entries(this.rules)) {
      const value = data[field];
      const fieldErrors = [];

      for (const rule of fieldRules) {
        if (typeof rule === "function") {
          const result = rule(value);
          if (result !== true) fieldErrors.push(result);
        } else if (!rule.validate(value)) {
          fieldErrors.push(rule.message);
        }
      }

      if (fieldErrors.length > 0) {
        errors[field] = fieldErrors;
      }
    }

    return {
      valid: Object.keys(errors).length === 0,
      errors
    };
  }
}

const userFormRules = {
  name: [validators.required, validators.minLength(2), validators.maxLength(50)],
  email: [validators.required, validators.email],
  age: [validators.required, validators.numeric, {
    validate: (v) => parseInt(v) >= 18,
    message: "Must be at least 18"
  }]
};

const validator = new FormValidator(userFormRules);
console.log(validator.validate({ name: "Al", email: "bad", age: "15" }));

// Compression strategies
class CompressionContext {
  constructor() {
    this.strategies = {};
  }

  register(name, strategy) {
    this.strategies[name] = strategy;
  }

  compress(data, algorithm) {
    const strategy = this.strategies[algorithm];
    if (!strategy) throw new Error(`Unknown algorithm: ${algorithm}`);
    return strategy.compress(data);
  }

  decompress(data, algorithm) {
    const strategy = this.strategies[algorithm];
    if (!strategy) throw new Error(`Unknown algorithm: ${algorithm}`);
    return strategy.decompress(data);
  }
}

const gzipStrategy = {
  compress: (data) => `gzip(${data})`,
  decompress: (data) => data.replace(/^gzip\(|\)$/g, "")
};

const deflateStrategy = {
  compress: (data) => `deflate(${data})`,
  decompress: (data) => data.replace(/^deflate\(|\)$/g, "")
};

const ctx = new CompressionContext();
ctx.register("gzip", gzipStrategy);
ctx.register("deflate", deflateStrategy);

const compressed = ctx.compress("Hello World", "gzip");
console.log(ctx.decompress(compressed)); // Hello World
```

### Advanced Examples

```javascript
// AI workflow with composable strategies
class AIPipeline {
  constructor() {
    this.steps = [];
  }

  addStep(name, strategy) {
    this.steps.push({ name, strategy });
    return this;
  }

  async execute(input) {
    let result = input;

    for (const { name, strategy } of this.steps) {
      console.log(`Running step: ${name}`);
      result = await strategy.execute(result);
    }

    return result;
  }
}

const textPreprocessing = {
  execute: async (text) => ({
    original: text,
    cleaned: text.toLowerCase().replace(/[^\w\s]/g, ""),
    tokens: text.toLowerCase().match(/\b\w+\b/g)
  })
};

const sentimentAnalysis = {
  execute: async (data) => {
    const positiveWords = ["good", "great", "excellent", "amazing", "love"];
    const negativeWords = ["bad", "terrible", "awful", "hate", "worst"];
    const tokens = data.tokens;

    const positiveCount = tokens.filter(t => positiveWords.includes(t)).length;
    const negativeCount = tokens.filter(t => negativeWords.includes(t)).length;

    const score = positiveCount - negativeCount;
    return {
      ...data,
      sentiment: score > 0 ? "positive" : score < 0 ? "negative" : "neutral",
      sentimentScore: score
    };
  }
};

const keywordExtraction = {
  execute: async (data) => {
    const stopWords = new Set(["the", "a", "an", "is", "are", "was", "were"]);
    const wordFreq = {};

    for (const token of data.tokens) {
      if (!stopWords.has(token)) {
        wordFreq[token] = (wordFreq[token] || 0) + 1;
      }
    }

    const keywords = Object.entries(wordFreq)
      .sort((a, b) => b[1] - a[1])
      .slice(0, 5)
      .map(([word]) => word);

    return { ...data, keywords };
  }
};

const pipeline = new AIPipeline();
pipeline
  .addStep("preprocessing", textPreprocessing)
  .addStep("sentiment", sentimentAnalysis)
  .addStep("keywords", keywordExtraction);

pipeline.execute("I love this product! It is truly amazing and great.")
  .then(result => console.log(result));

// Caching strategies with disk/memory/redis
class CacheManager {
  constructor() {
    this.strategies = new Map();
  }

  register(name, strategy) {
    this.strategies.set(name, strategy);
  }

  async get(key, strategyName = "memory") {
    const strategy = this.strategies.get(strategyName);
    if (!strategy) throw new Error(`Unknown cache strategy: ${strategyName}`);
    return strategy.get(key);
  }

  async set(key, value, ttl, strategyName = "memory") {
    const strategy = this.strategies.get(strategyName);
    return strategy.set(key, value, ttl);
  }
}

const memoryCache = {
  store: new Map(),
  get(key) {
    const entry = this.store.get(key);
    if (!entry) return null;
    if (Date.now() > entry.expiresAt) {
      this.store.delete(key);
      return null;
    }
    return entry.value;
  },
  set(key, value, ttl = 3600000) {
    this.store.set(key, { value, expiresAt: Date.now() + ttl });
  }
};

const localStorageCache = {
  get(key) {
    const raw = localStorage.getItem(`cache:${key}`);
    if (!raw) return null;
    const { value, expiresAt } = JSON.parse(raw);
    if (Date.now() > expiresAt) {
      localStorage.removeItem(`cache:${key}`);
      return null;
    }
    return value;
  },
  set(key, value, ttl = 3600000) {
    localStorage.setItem(`cache:${key}`, JSON.stringify({
      value, expiresAt: Date.now() + ttl
    }));
  }
};

const cache = new CacheManager();
cache.register("memory", memoryCache);
cache.register("localStorage", localStorageCache);
```

### Real-World Use Cases

- **Authentication strategies**: Passport.js uses strategies for different auth providers.
- **Logging frameworks**: Different log levels and output formats as strategies.
- **Serialization/deserialization**: JSON, XML, YAML, Protobuf parsers as strategies.
- **Database query builders**: Different SQL dialects (MySQL, PostgreSQL, SQLite).
- **Rate limiting**: Token bucket, sliding window, fixed window as strategies.
- **Sorting in e-commerce**: Price, rating, relevance, newest as sort strategies.

### Common Mistakes

```javascript
// Mistake 1: Strategy that knows too much about the context
class BadStrategy {
  process(context) {
    context.internalState = "modified"; // Breaks encapsulation
  }
}

// Mistake 2: Over-engineering — using Strategy pattern for trivial variations
// Just use a simple condition or callback

// Mistake 3: Mutating shared state in strategies
const sharedData = { count: 0 };
const badStrategy = {
  execute() { sharedData.count++; } // Side effect!
};

// Mistake 4: Strategies that depend on each other
class StrategyA {
  execute() { return StrategyB.execute(); } // Circular dependency
}
```

### Best Practices

```javascript
// Keep strategies stateless when possible
const pureStrategy = {
  execute(input) {
    return transform(input); // Pure function, no side effects
  }
};

// Use meaningful names for strategies
const sortStrategies = {
  byPriceAscending: (a, b) => a.price - b.price,
  byPriceDescending: (a, b) => b.price - a.price,
  byName: (a, b) => a.name.localeCompare(b.name)
};

// Validate strategy interface at registration
function registerStrategy(name, strategy) {
  if (typeof strategy.execute !== "function") {
    throw new Error(`Strategy "${name}" must implement execute()`);
  }
  this.strategies.set(name, strategy);
}

// Use strategy enumeration for type safety
const StrategyType = Object.freeze({
  FAST: "fast",
  BALANCED: "balanced",
  ACCURATE: "accurate"
});
```

### Performance Considerations

- Strategy lookup is O(1) with a Map; O(n) with if/else chains.
- Function call overhead per strategy execution is negligible.
- Stateless strategies can be singletons (reuse instances).
- Avoid creating new strategy instances on every operation — pre-register them.

### Interview Questions

**Q: How does the Strategy Pattern relate to higher-order functions?**
A: In JavaScript, strategies can be implemented as plain functions passed as arguments to a context function. This is the functional programming equivalent of the Strategy Pattern. Where OOP uses objects with a shared interface, FP uses function references. Both achieve the same goal of interchangeable algorithms.

**Q: How is Strategy different from the State Pattern?**
A: Strategy is about selecting an algorithm at runtime; the client chooses the strategy. State is about changing behavior based on internal state; the state object transitions control automatically. In Strategy, the context doesn't know which algorithm is running. In State, the context's behavior changes implicitly.

**Q: Can you combine Strategy with Factory Pattern?**
A: Yes. A factory can create the appropriate strategy based on configuration or input data. This is common in library design where `createSortStrategy(type)` returns the right strategy implementation.

### Coding Challenges

```javascript
// Challenge 1: Implement a text formatter with strategies
// for different output formats: plain text, HTML, Markdown, JSON.
// Each strategy should have format(heading, body, metadata).

// Challenge 2: Build a notification dispatcher that uses strategies
// for different channels: email, SMS, push notification, Slack webhook.
// Each strategy should implement send(recipient, message, options).

// Challenge 3: Create a cost calculation system for a taxi app
// with strategies: standard pricing (per km), surge pricing (multiplier),
// flat rate, and subscription (no per-ride cost up to certain distance).
```

### Related Topics

- State Pattern
- Command Pattern
- Template Method Pattern
- Functional programming (functions as strategies)
- Dependency injection

## Strategy Selection

### What It Is

Strategy selection refers to the mechanism by which the appropriate algorithm is chosen at runtime. This can be done explicitly by the client, automatically based on conditions, through configuration, or via factory methods. The selection mechanism is a critical design decision in any system using the Strategy Pattern.

### Why It Is Important

The way strategies are selected determines the flexibility and usability of the system. Well-designed selection mechanisms make adding new strategies easy without changing existing code, while poor selection mechanisms reintroduce the conditional logic the pattern was meant to eliminate.

### How It Works Internally

Strategy selection works through one of several mechanisms:
1. **Direct assignment**: Client directly sets the strategy via a setter
2. **Parameter-based selection**: Context selects based on a parameter (lookup in a Map)
3. **Conditional selection**: Context uses conditionals to determine which strategy
4. **Factory-based selection**: A factory creates the right strategy based on context
5. **Automatic selection**: The strategy is determined by data characteristics (e.g., data size determines sort algorithm)

### Syntax

```javascript
// 1. Direct assignment
const context = new Context();
context.setStrategy(new ConcreteStrategyA());

// 2. Parameter-based (map lookup)
function execute(type, data) {
  const strategy = registry.get(type);
  return strategy.execute(data);
}

// 3. Strategy chain (auto-selection)
function selectStrategy(data) {
  if (data.length < 100) return new QuickSort();
  if (data.length < 10000) return new MergeSort();
  return new ExternalSort();
}
```

### Beginner Examples

```javascript
// Parameter-based strategy selection
const formatters = {
  json: (data) => JSON.stringify(data, null, 2),
  csv: (data) => {
    const headers = Object.keys(data[0]).join(",");
    const rows = data.map(row => Object.values(row).join(","));
    return [headers, ...rows].join("\n");
  },
  xml: (data) => {
    let xml = "<data>";
    for (const item of data) {
      xml += "<item>";
      for (const [key, value] of Object.entries(item)) {
        xml += `<${key}>${value}</${key}>`;
      }
      xml += "</item>";
    }
    return xml + "</data>";
  }
};

function exportData(data, format) {
  const formatter = formatters[format];
  if (!formatter) throw new Error(`Unsupported format: ${format}`);
  return formatter(data);
}

const users = [
  { id: 1, name: "Alice", email: "alice@example.com" },
  { id: 2, name: "Bob", email: "bob@example.com" }
];

console.log(exportData(users, "csv"));
// id,name,email
// 1,Alice,alice@example.com
// 2,Bob,bob@example.com

// Context-based selection
class ImageProcessor {
  selectStrategy(image) {
    if (image.type === "png") {
      return pngStrategy;
    } else if (image.type === "jpg" || image.type === "jpeg") {
      return jpgStrategy;
    } else if (image.type === "webp") {
      return webpStrategy;
    }
    throw new Error(`Unsupported image type: ${image.type}`);
  }

  process(image) {
    const strategy = this.selectStrategy(image);
    return strategy.process(image);
  }
}
```

### Intermediate Examples

```javascript
// Automatic strategy selection based on data characteristics
class SortFactory {
  static create(data, options = {}) {
    const size = data.length;

    if (size <= 1) {
      return new NoOpSort(); // Nothing to sort
    }

    if (options.stability) {
      return new MergeSort(); // Stable sort
    }

    if (size <= 10) {
      return new InsertionSort(); // Fast for tiny arrays
    }

    if (size <= 1000) {
      return new QuickSort(); // Fast average case
    }

    if (size <= 100000) {
      return new HeapSort(); // Consistent O(n log n)
    }

    return new ExternalSort(); // Disk-based for huge data
  }
}

class SortManager {
  sort(data, options = {}) {
    const strategy = SortFactory.create(data, options);
    return strategy.sort([...data]);
  }
}

// Strategy chain with middleware-like composition
class CompressorSelector {
  constructor() {
    this.strategies = new Map();
    this.defaultStrategy = "snappy";
  }

  register(name, strategy, condition) {
    this.strategies.set(name, { strategy, condition });
  }

  select(data, options = {}) {
    // 1. Explicit override
    if (options.algorithm) {
      return this.strategies.get(options.algorithm).strategy;
    }

    // 2. Auto-select based on conditions
    for (const [name, { strategy, condition }] of this.strategies) {
      if (condition && condition(data, options)) {
        console.log(`Auto-selected ${name} compression`);
        return strategy;
      }
    }

    // 3. Fallback to default
    return this.strategies.get(this.defaultStrategy).strategy;
  }

  compress(data, options = {}) {
    const strategy = this.select(data, options);
    return strategy.compress(data);
  }
}

const compressor = new CompressorSelector();
compressor.register("none", { compress: d => d, decompress: d => d },
  (data) => data.length < 100
);
compressor.register("gzip", { compress: d => `gzip:${d}`, decompress: d => d.slice(5) },
  (data) => data.length >= 100 && data.length < 10000
);
compressor.register("snappy", { compress: d => `snappy:${d}`, decompress: d => d.slice(7) });

// Policy-based selection from configuration
class CacheStrategySelector {
  constructor(config = {}) {
    this.config = config;
    this.rules = [
      { pattern: /^user:\d+:profile$/, strategy: "long_lived" },
      { pattern: /^session:\w+$/, strategy: "short_lived" },
      { pattern: /^search:.*$/, strategy: "medium_lived" },
      { pattern: /.*/, strategy: "default" }
    ];
  }

  select(key) {
    for (const rule of this.rules) {
      if (rule.pattern.test(key)) {
        const strategyConfig = this.config[rule.strategy] || this.config.default;
        return createStrategy(strategyConfig);
      }
    }
  }
}

function createStrategy(config) {
  return {
    ttl: config.ttl || 300000,
    storage: config.storage || "memory",
    get(key) { /* ... */ },
    set(key, value) { /* ... */ }
  };
}
```

### Advanced Examples

```javascript
// Dynamic strategy loading based on runtime capabilities
class StrategyLoader {
  constructor() {
    this.cache = new Map();
  }

  async load(type) {
    if (this.cache.has(type)) {
      return this.cache.get(type);
    }

    let module;
    switch (type) {
      case "database":
        module = await this.detectDatabase();
        break;
      case "cache":
        module = await this.detectCacheBackend();
        break;
      case "search":
        module = await this.detectSearchEngine();
        break;
      default:
        throw new Error(`Unknown strategy type: ${type}`);
    }

    this.cache.set(type, module);
    return module;
  }

  async detectDatabase() {
    try {
      const pg = await import("pg");
      return { type: "postgres", client: new pg.Pool(/* ... */) };
    } catch {
      try {
        const mysql = await import("mysql2");
        return { type: "mysql", client: mysql.createPool(/* ... */) };
      } catch {
        return { type: "sqlite", client: await this.initSQLite() };
      }
    }
  }

  async detectCacheBackend() {
    try {
      const redis = await import("redis");
      const client = redis.createClient();
      await client.connect();
      return { type: "redis", client };
    } catch {
      return { type: "memory", client: new Map() };
    }
  }

  async detectSearchEngine() {
    // Similar pattern for Elasticsearch, Meilisearch, etc.
  }
}

// A/B testing as strategy selection
class ABTestSelector {
  constructor(experiments) {
    this.experiments = experiments;
    this.assignments = new Map();
  }

  getStrategy(userId, experimentName) {
    const experiment = this.experiments[experimentName];
    if (!experiment) return experimentName;

    const cacheKey = `${userId}:${experimentName}`;
    if (!this.assignments.has(cacheKey)) {
      const bucket = this.hash(userId, experimentName) % 100;
      let cumulative = 0;
      for (const [variant, weight] of Object.entries(experiment.variants)) {
        cumulative += weight;
        if (bucket < cumulative) {
          this.assignments.set(cacheKey, variant);
          break;
        }
      }
    }

    const variant = this.assignments.get(cacheKey);
    return experiment.strategies[variant];
  }

  hash(userId, experimentName) {
    let hash = 0;
    const str = `${userId}:${experimentName}`;
    for (let i = 0; i < str.length; i++) {
      hash = ((hash << 5) - hash) + str.charCodeAt(i);
      hash |= 0;
    }
    return Math.abs(hash);
  }
}

// Usage
const experiments = {
  checkout_button: {
    variants: { red: 50, blue: 50 },
    strategies: {
      red: { render: () => "Red button" },
      blue: { render: () => "Blue button" }
    }
  }
};
const abTest = new ABTestSelector(experiments);
const button = abTest.getStrategy("user123", "checkout_button");
```

### Real-World Use Cases

- **Passport.js authentication**: Strategies are registered for different providers and selected by name during authentication.
- **Log4j-style logging**: Logger selects appender strategy based on log level and category.
- **Webpack loaders**: File types select different loader strategies.
- **Express middleware**: Middleware selection based on route and method.
- **Feature flags**: Flags select which strategy variant to use at runtime.

### Common Mistakes

```javascript
// Mistake 1: Giant switch/if chain that defeats the purpose
function selectStrategy(type) {
  // This should be a map lookup, not a chain
  if (type === "a") return new StrategyA();
  else if (type === "b") return new StrategyB();
  else if (type === "c") return new StrategyC();
  // ...
}

// Mistake 2: Hardcoding selection logic inside the context
class Context {
  process(data) {
    if (data.type === "small") {
      this.strategy = new FastStrategy();
    } else {
      this.strategy = new AccurateStrategy();
    }
    return this.strategy.process(data);
  }
}

// Mistake 3: Not handling missing strategies gracefully
function execute(name) {
  return strategies[name].execute(); // Crash if name not found
}
```

### Best Practices

```javascript
// Use a registry (Map) for strategy lookup
const registry = new Map();
registry.set("fast", new FastStrategy());
registry.set("accurate", new AccurateStrategy());

// Provide a default/fallback strategy
function getStrategy(name) {
  return registry.get(name) || registry.get("default");
}

// Validate strategy existence early
function execute(type, data) {
  const strategy = strategies.get(type);
  if (!strategy) {
    throw new Error(`Unknown strategy: ${type}. Available: ${[...strategies.keys()].join(", ")}`);
  }
  return strategy.execute(data);
}

// Centralize selection logic in a dedicated selector class
class StrategySelector {
  // selection logic lives here, not scattered
}
```

### Performance Considerations

- Map/object lookup for strategy selection is O(1) and highly optimized.
- Conditional chains are O(n) and should be avoided for many strategies.
- Pre-register strategies at initialization rather than creating them on the fly.
- For auto-selection based on data characteristics, the selection logic itself should be O(1) or O(log n).

### Interview Questions

**Q: How would you implement strategy selection for a plugin system?**
A: Use a registry pattern where plugins register themselves with a name and condition function. Selection can iterate over registered plugins, evaluate conditions, and return the first match or the most specific match. The registry can be populated at startup via configuration or plugin discovery.

**Q: What's the difference between strategy selection and the Chain of Responsibility?**
A: Strategy selection picks one algorithm from a set. Chain of Responsibility passes a request along a chain of handlers until one handles it. The intent differs: selection is "pick the right tool," while chain is "try these in sequence."

### Coding Challenges

```javascript
// Challenge 1: Build a routing system where URL patterns map to
// handler strategies. Selection should support exact match, parameterized
// routes (/users/:id), and wildcard routes.

// Challenge 2: Create a caching layer that automatically selects between
// memory, localStorage, IndexedDB, and remote cache based on:
// - Data size
// - Expected access frequency
// - Browser capability
// - User preference (offline mode)

// Challenge 3: Implement a parser selector that picks the right parser
// (JSON, XML, YAML, CSV, TOML) based on file extension, content sniffing,
// and explicit override.
```

### Related Topics

- Registry pattern
- Chain of Responsibility
- Factory pattern (strategy creation)
- Feature flag systems
- Plugin architectures

## Function-Based Strategies

### What It Is

Function-based strategies leverage JavaScript's first-class functions to implement the Strategy Pattern without classes or interfaces. Instead of wrapping algorithms in objects with a common method name, strategies are simply functions passed as arguments or stored in data structures.

### Why It Is Important

Function-based strategies are more concise, more flexible, and more idiomatic in JavaScript than class-based strategies. They reduce boilerplate, support functional composition, and are naturally interchangeable. This approach aligns with JavaScript's functional programming capabilities.

### How It Works Internally

The function-based strategy pattern works exactly like the class-based version, but instead of objects with methods, plain functions are used. The context accepts a function (or collection of functions) and calls it when needed. Closures allow capturing configuration without creating objects.

### Syntax

```javascript
// Function-based strategy
const addStrategy = (a, b) => a + b;
const multiplyStrategy = (a, b) => a * b;

// Context that accepts a function strategy
function calculate(a, b, strategy) {
  return strategy(a, b);
}

calculate(5, 3, addStrategy); // 8
calculate(5, 3, multiplyStrategy); // 15

// Map of function strategies
const operations = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
  multiply: (a, b) => a * b,
  divide: (a, b) => b !== 0 ? a / b : NaN
};

function compute(a, b, operation) {
  const fn = operations[operation];
  if (!fn) throw new Error(`Unknown operation: ${operation}`);
  return fn(a, b);
}
```

### Beginner Examples

```javascript
// Sorting strategies as simple compare functions
const sortStrategies = {
  byName: (a, b) => a.name.localeCompare(b.name),
  byAge: (a, b) => a.age - b.age,
  byEmail: (a, b) => a.email.localeCompare(b.email)
};

function sortUsers(users, strategy) {
  return [...users].sort(strategy);
}

const users = [
  { name: "Charlie", age: 25, email: "charlie@example.com" },
  { name: "Alice", age: 30, email: "alice@example.com" },
  { name: "Bob", age: 22, email: "bob@example.com" }
];

console.log(sortUsers(users, sortStrategies.byName));

// Discount calculator as function strategies
const discountStrategies = {
  none: (price) => price,
  tenPercent: (price) => price * 0.9,
  twentyPercent: (price) => price * 0.8,
  fixedFifty: (price) => Math.max(0, price - 50),
  buyOneGetOne: (price) => price * 0.5
};

function calculatePrice(basePrice, discountCode) {
  const strategy = discountStrategies[discountCode] || discountStrategies.none;
  return strategy(basePrice);
}

console.log(calculatePrice(100, "twentyPercent")); // 80
console.log(calculatePrice(100, "fixedFifty")); // 50

// Filter functions as strategies
const filters = {
  active: (user) => user.status === "active",
  premium: (user) => user.plan === "premium",
  recent: (user) => Date.now() - user.lastLogin < 86400000,
  verified: (user) => user.emailVerified
};

function filterUsers(users, filterFns) {
  return users.filter(user => filterFns.every(fn => fn(user)));
}

const activePremium = filterUsers(users, [filters.active, filters.premium]);
```

### Intermediate Examples

```javascript
// Composable function strategies with higher-order functions
const withLogging = (strategy) => (input) => {
  console.log(`Executing strategy with input:`, input);
  const result = strategy(input);
  console.log(`Result:`, result);
  return result;
};

const withValidation = (strategy, validator) => (input) => {
  if (!validator(input)) {
    throw new Error(`Validation failed for: ${input}`);
  }
  return strategy(input);
};

const withCache = (strategy) => {
  const cache = new Map();
  return (input) => {
    if (cache.has(input)) {
      console.log("Cache hit");
      return cache.get(input);
    }
    const result = strategy(input);
    cache.set(input, result);
    return result;
  };
};

// Base strategies
const double = (x) => x * 2;
const square = (x) => x * x;

// Composed strategies
const safeDouble = withValidation(double, x => typeof x === "number");
const loggedDouble = withLogging(double);
const cachedSquare = withCache(square);

console.log(safeDouble(5)); // 10
loggedDouble(3); // "Executing..." "Result: 6"
cachedSquare(4); // 16
cachedSquare(4); // Cache hit, 16

// Strategy that returns a function (configuration via closure)
function createTaxStrategy(options = {}) {
  const { rate = 0.2, exemptCategories = [], surcharge = 0 } = options;

  return (amount, category) => {
    if (exemptCategories.includes(category)) return 0;
    return amount * rate + surcharge;
  };
}

const standardTax = createTaxStrategy({ rate: 0.2 });
const reducedTax = createTaxStrategy({ rate: 0.1, exemptCategories: ["food", "medicine"] });
const luxuryTax = createTaxStrategy({ rate: 0.25, surcharge: 5 });

console.log(standardTax(100, "electronics")); // 20
console.log(reducedTax(100, "food")); // 0
console.log(luxuryTax(100, "luxury")); // 30

// Function strategy registry with middleware
class StrategyPipeline {
  constructor() {
    this.strategies = [];
  }

  use(fn) {
    this.strategies.push(fn);
    return this;
  }

  execute(input) {
    let result = input;
    for (const strategy of this.strategies) {
      result = strategy(result);
    }
    return result;
  }
}

const pipeline = new StrategyPipeline();
pipeline
  .use(x => x * 2)
  .use(x => x + 1)
  .use(x => `Result: ${x}`);

console.log(pipeline.execute(5)); // "Result: 11"
```

### Advanced Examples

```javascript
// Strategy pattern implemented with Proxy for transparent interception
function createStrategyProxy(strategies) {
  return new Proxy(strategies, {
    get(target, prop) {
      if (prop in target) {
        return target[prop];
      }
      // Return a no-op strategy for missing names
      return (...args) => {
        console.warn(`Strategy "${prop}" not found, returning default`);
        return args[0]; // Pass-through
      };
    }
  });
}

const safeStrategies = createStrategyProxy({
  double: x => x * 2,
  triple: x => x * 3
});

console.log(safeStrategies.double(5)); // 10
console.log(safeStrategies.quadruple(5)); // Warning, returns 5

// Async function strategies
const storageStrategies = {
  local: async (key, data) => {
    localStorage.setItem(key, JSON.stringify(data));
    return true;
  },
  session: async (key, data) => {
    sessionStorage.setItem(key, JSON.stringify(data));
    return true;
  },
  indexedDB: async (key, data) => {
    const request = indexedDB.open("AppDB", 1);
    return new Promise((resolve, reject) => {
      request.onsuccess = (event) => {
        const db = event.target.result;
        const tx = db.transaction("store", "readwrite");
        tx.objectStore("store").put(data, key);
        tx.oncomplete = () => resolve(true);
        tx.onerror = () => reject(tx.error);
      };
      request.onerror = () => reject(request.error);
    });
  },
  remote: async (key, data) => {
    const response = await fetch("/api/store", {
      method: "POST",
      body: JSON.stringify({ key, data }),
      headers: { "Content-Type": "application/json" }
    });
    return response.ok;
  }
};

async function saveData(key, data, storageType = "local") {
  const strategy = storageStrategies[storageType];
  if (!strategy) throw new Error(`Unknown storage: ${storageType}`);
  return strategy(key, data);
}

// Strategy composition with reduce
const composeStrategies = (...strategies) => (input) =>
  strategies.reduce((result, strategy) => strategy(result), input);

const toLowerCase = s => s.toLowerCase();
const removePunctuation = s => s.replace(/[^\w\s]/g, "");
const trim = s => s.trim();
const truncate = max => s => s.length > max ? s.slice(0, max) + "..." : s;

const sanitize = composeStrategies(
  toLowerCase,
  removePunctuation,
  trim,
  truncate(50)
);

console.log(sanitize("Hello World!!! This is a very long sentence that needs..."));

// Conditional strategy selection using predicates
function createConditionalStrategy(strategies) {
  return (input) => {
    for (const [predicate, strategy] of strategies) {
      if (predicate(input)) {
        return strategy(input);
      }
    }
    throw new Error("No matching strategy found");
  };
}

const processPayment = createConditionalStrategy([
  [(amount) => amount < 50, (amount) => ({ method: "quick", fee: 0, total: amount })],
  [(amount) => amount < 500, (amount) => ({ method: "standard", fee: 2, total: amount + 2 })],
  [(amount) => amount < 10000, (amount) => ({ method: "secure", fee: 5, total: amount + 5 })],
  [(amount) => amount >= 10000, (amount) => ({ method: "manual", fee: 0, total: amount, requiresApproval: true })]
]);

console.log(processPayment(30)); // quick, fee 0
console.log(processPayment(200)); // standard, fee 2
```

### Real-World Use Cases

- **Array.sort()**: Takes a compare function — the Strategy Pattern in the standard library.
- **Array.filter()**: Takes a predicate function strategy.
- **Express/Koa middleware**: Middleware functions are processing strategies composed into a pipeline.
- **React Redux reducers**: Reducer functions are strategies for state updates.
- **Lodash/Underscore**: Higher-order functions like `_.debounce`, `_.throttle` are strategy creators.

### Common Mistakes

```javascript
// Mistake 1: Not preserving `this` context in function strategies
const obj = {
  value: 42,
  getValue: () => this.value // Wrong! Arrow function doesn't have its own this
};

// Mistake 2: Mutating input in function strategies
function badStrategy(arr) {
  arr.push("modified"); // Side effect!
  return arr;
}

// Mistake 3: Overly complex closures that are hard to debug
const complex = (a) => (b) => (c) => (d) => a + b + c + d;
// Maybe split into named functions?

// Mistake 4: Not handling async function strategies properly
function process(strategy, data) {
  return strategy(data); // If strategy is async, this returns a Promise
}
```

### Best Practices

```javascript
// Keep function strategies pure (no side effects)
const pureStrategy = (data) => ({ ...data, processed: true });

// Use descriptive names for strategy functions
// Instead of:
const fn1 = (x) => x * 2;

// Use:
const doublePrice = (x) => x * 2;

// Document the expected function signature with JSDoc
/**
 * @callback TransformStrategy
 * @param {string} input
 * @returns {string}
 */

/** @type {TransformStrategy} */
const uppercase = (input) => input.toUpperCase();

// Use default parameter strategies
function processWithDefault(input, strategy = defaultStrategy) {
  return strategy(input);
}

// Keep strategies small and composable
const addTax = rate => amount => amount * (1 + rate);
const addShipping = cost => amount => amount + cost;
const applyDiscount = discount => amount => amount * (1 - discount);
```

### Performance Considerations

- Function call overhead in JavaScript is very low (nanoseconds per call).
- Closure-based strategies can capture large scopes; be mindful of memory.
- Function composition via `reduce` adds O(n) overhead proportional to strategy count.
- V8 optimizes monomorphic function calls; frequently changing strategies may trigger deoptimization.

### Comparison: Class-Based vs Function-Based Strategies

| Aspect | Class-Based | Function-Based |
|--------|-------------|----------------|
| Boilerplate | More (class definition, method) | Less (just a function) |
| State management | Instance properties | Closures |
| Testability | Mock via inheritance | Mock via function substitution |
| Flexibility | Fixed interface per class | Duck-typed signatures |
| Composition | Object composition | Function composition |
| Performance | Slightly more overhead | Minimal overhead |
| When to use | Multiple methods per strategy, shared state | Single operation, stateless |

### Interview Questions

**Q: How do you implement the Strategy Pattern using only functions?**
A: Simply pass functions as arguments to other functions. The context function accepts a strategy function and calls it. Strategies can be stored in objects for lookup, composed using `compose`/`pipe`, or configured via higher-order functions that return strategy functions.

**Q: What are the advantages of function-based strategies over class-based?**
A: Function-based strategies have less boilerplate, are more naturally composable, support functional programming patterns (currying, partial application), and are more idiomatic in JavaScript. They also integrate seamlessly with callback-based APIs and Promises.

**Q: How do you handle async strategies in a function-based approach?**
A: Async strategies return Promises. The context function can `await` the strategy result or return a Promise. This works transparently with async/await and doesn't require any special handling in the strategy pattern implementation.

### Coding Challenges

```javascript
// Challenge 1: Implement a caching decorator for function strategies.
// Given a strategy function, return a new function that caches results
// based on a cache key derived from the arguments.

// Challenge 2: Create a rate limiter that accepts a strategy function
// and returns a throttled version that limits executions to N per minute.

// Challenge 3: Build a retry wrapper for async strategy functions
// that retries on failure with exponential backoff.
// The wrapper should accept the strategy, max retries, and backoff function.
```

### Related Topics

- Higher-order functions
- Function composition
- Callback pattern
- Decorator pattern
- Functional programming in JavaScript
