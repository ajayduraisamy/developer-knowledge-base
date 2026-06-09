# Jest - Jest setup, describe/it/expect, matchers, setup/teardown

## Introduction

Jest is a comprehensive JavaScript testing framework developed by Facebook, widely used for unit testing, integration testing, and snapshot testing. It provides a complete testing solution including assertions, mocking, code coverage, and parallel test execution out of the box. Jest's zero-config setup, built-in coverage reports, and intuitive API make it the most popular testing framework in the JavaScript ecosystem.

## Jest setup

### What It Is

Setting up Jest involves installing the package, creating a configuration file (optional), and writing test files that Jest can discover and execute. Jest works with JavaScript, TypeScript, React, Node.js, and most modern frameworks.

```javascript
npm install --save-dev jest
```

### Why It Is Important

Proper setup ensures Jest runs efficiently with your project's specific needs. Configuration includes test file patterns, module resolution, transform rules, coverage thresholds, and environment settings.

### How It Works Internally

Jest uses workers to run tests in parallel across multiple processes. It discovers test files by pattern (default: `*.test.js` or `*.spec.js` in `__tests__` folders), transforms files using configured transformers, provides a global test environment with built-in assertions and mocking, and collects coverage using V8's built-in coverage or Istanbul.

```javascript
// Test execution flow:
// 1. Jest reads configuration
// 2. Discovers test files matching patterns
// 3. Applies transforms (babel-jest, ts-jest, etc.)
// 4. Sets up global test environment
// 5. Runs tests in parallel workers
// 6. Collects results and coverage
// 7. Generates reports
```

### Syntax

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'node',
  testMatch: ['**/*.test.js'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  setupFilesAfterSetup: ['./jest.setup.js'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1'
  }
}

// package.json alternative
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:ci": "jest --ci --coverage --maxWorkers=2"
  }
}
```

### Beginner Examples

```javascript
// math.js
function add(a, b) { return a + b }
function subtract(a, b) { return a - b }

module.exports = { add, subtract }

// math.test.js
const { add, subtract } = require('./math')

test('adds 1 + 2 to equal 3', () => {
  expect(add(1, 2)).toBe(3)
})

test('subtracts 5 - 3 to equal 2', () => {
  expect(subtract(5, 3)).toBe(2)
})
```

### Intermediate Examples

```javascript
// jest.config.js with full configuration
module.exports = {
  // Test environment
  testEnvironment: 'jsdom',
  testEnvironmentOptions: {
    url: 'http://localhost:3000',
    customExportConditions: ['node', 'node-addons']
  },

  // File patterns
  testMatch: [
    '**/__tests__/**/*.[jt]s?(x)',
    '**/?(*.)+(spec|test).[jt]s?(x)'
  ],
  testPathIgnorePatterns: [
    '/node_modules/',
    '/dist/',
    '/cypress/'
  ],

  // Transform and module resolution
  transform: {
    '^.+\\.jsx?$': 'babel-jest',
    '^.+\\.tsx?$': 'ts-jest'
  },
  moduleNameMapper: {
    '\\.(css|less|scss)$': '<rootDir>/__mocks__/styleMock.js',
    '\\.(jpg|jpeg|png|gif)$': '<rootDir>/__mocks__/fileMock.js',
    '^@components/(.*)$': '<rootDir>/src/components/$1',
    '^@utils/(.*)$': '<rootDir>/src/utils/$1'
  },

  // Coverage
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js',
    '!src/reportWebVitals.js'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 85,
      lines: 85,
      statements: 85
    },
    'src/components/**': {
      branches: 90,
      functions: 90
    }
  },

  // Global setup and teardown
  globalSetup: './jest.globalSetup.js',
  globalTeardown: './jest.globalTeardown.js',
  setupFiles: ['./jest.setup.js'],
  setupFilesAfterFramework: ['./jest.setupAfter.js'],

  // Test behavior
  bail: 1,
  verbose: true,
  maxWorkers: '50%',
  testTimeout: 10000,
  forceExit: true,
  detectOpenHandles: true
}

// Custom reporter
class CustomReporter {
  onRunComplete(contexts, results) {
    console.log(`Tests: ${results.numPassedTests}/${results.numTotalTests} passed`)
    results.testResults.forEach(testFile => {
      testFile.testResults.forEach(test => {
        if (test.status === 'failed') {
          console.error(`FAIL ${testFile.testFilePath}: ${test.fullName}`)
        }
      })
    })
  }
}
```

### Advanced Examples

```javascript
// Custom test environment
const NodeEnvironment = require('jest-environment-node')

class CustomEnvironment extends NodeEnvironment {
  constructor(config, context) {
    super(config, context)
    this.global.testDb = null
  }

  async setup() {
    await super.setup()
    this.global.testDb = await createTestDatabase()
    this.global.testUser = { id: 1, name: 'Test User' }
  }

  async teardown() {
    await this.global.testDb.destroy()
    await super.teardown()
  }

  getVmContext() {
    return super.getVmContext()
  }
}

module.exports = CustomEnvironment

// jest.config.js uses it:
// testEnvironment: './CustomEnvironment.js'

// Custom matcher with jest.extend
expect.extend({
  toBeWithinRange(received, floor, ceiling) {
    const pass = received >= floor && received <= ceiling
    if (pass) {
      return {
        message: () => `expected ${received} not to be within range ${floor} - ${ceiling}`,
        pass: true
      }
    }
    return {
      message: () => `expected ${received} to be within range ${floor} - ${ceiling}`,
      pass: false
    }
  },

  toBeValidDate(received) {
    const pass = received instanceof Date && !isNaN(received.getTime())
    return {
      message: () => `expected ${received} to be a valid date`,
      pass
    }
  }
})

// Usage
test('custom matchers', () => {
  expect(100).toBeWithinRange(50, 150)
  expect(new Date('2024-01-01')).toBeValidDate()
})

// Snapshot testing with dynamic updates
test('matches component snapshot', () => {
  const tree = renderer.create(<MyComponent prop="value" />).toJSON()
  expect(tree).toMatchSnapshot()
})

// Inline snapshots
test('inline snapshot', () => {
  const data = { name: 'test', value: 42 }
  expect(data).toMatchInlineSnapshot(`
    {
      "name": "test",
      "value": 42,
    }
  `)
})

// Watch mode plugins
// --watch with --watchAll
// Jest watch mode:
// Press a to run all tests
// Press f to run only failed tests
// Press p to filter by filename
// Press t to filter by test name
// Press q to quit
```

### Real-World Use Cases

- **Component testing**: Verify React/Vue/Angular component output
- **API testing**: Test endpoint responses and error handling
- **Utility function testing**: Pure function behavior verification
- **Redux/state testing**: Reducer and selector testing
- **Snapshot testing**: Catch UI regressions
- **Integration testing**: Module interaction verification
- **Configuration validation**: Test environment config correctness

### Common Mistakes

```javascript
// Mistake: Not isolating tests (shared state)
let counter = 0

test('increment', () => {
  counter++ // Affects other tests!
  expect(counter).toBe(1)
})

test('increment again', () => {
  expect(counter).toBe(1) // Fails if first test ran!
})

// Mistake: Testing implementation details, not behavior
test('calls setState internally', () => {
  // Don't test that setState was called
  // Test the component output/behavior instead
})

// Mistake: Forgetting async handling
test('async test', () => {
  fetchData().then(data => {
    expect(data).toBeDefined()
  })
  // Test ends before assertion runs!
})

// Mistake: Using toBe for objects
test('object comparison', () => {
  expect({ a: 1 }).toBe({ a: 1 }) // Fails! Use toEqual instead
})

// Mistake: Over-mocking
// Mocking everything makes tests brittle and hard to maintain
```

### Best Practices

```javascript
// Structure tests with AAA pattern
test('should add a new user', async () => {
  // Arrange
  const userData = { name: 'Alice', email: 'alice@test.com' }

  // Act
  const result = await userService.create(userData)

  // Assert
  expect(result).toHaveProperty('id')
  expect(result.name).toBe('Alice')
})

// Use describe for logical grouping
describe('UserService', () => {
  describe('create()', () => {
    test('creates user with valid data', () => {})
    test('throws with invalid email', () => {})
  })

  describe('delete()', () => {
    test('deletes existing user', () => {})
    test('throws for non-existent user', () => {})
  })
})

// Write deterministic tests
test('date formatting', () => {
  jest.useFakeTimers()
  jest.setSystemTime(new Date('2024-01-15'))

  const result = formatDate(new Date())
  expect(result).toBe('2024-01-15')

  jest.useRealTimers()
})

// Prefer test.each for data-driven tests
test.each([
  [1, 1, 2],
  [1, 2, 3],
  [5, 5, 10],
  [-1, 1, 0]
])('add(%i, %i) => %i', (a, b, expected) => {
  expect(add(a, b)).toBe(expected)
})

// Use describe.each for component variations
describe.each([
  ['small', 'sm'],
  ['medium', 'md'],
  ['large', 'lg']
])('Button size: %s', (size, className) => {
  test(`renders ${size} button`, () => {
    const { container } = render(<Button size={size} />)
    expect(container.querySelector(`.btn-${className}`)).toBeTruthy()
  })
})
```

### Performance Considerations

- Jest runs tests in parallel using worker processes (configurable with maxWorkers)
- Reuse setup across tests with beforeEach, beforeAll to reduce overhead
- Use `--changedFilesWithAncestors` for CI to only test changed files
- Use `--selectProjects` for monorepos to target specific packages
- Limit snapshot file sizes for faster diffing
- Mock heavy modules (database, filesystem) to speed up tests
- Use `jest --ci` for stricter CI performance
- Consider `jest --silent` to reduce console noise in CI

### Interview Questions

**Q: How does Jest run tests in parallel?**

A: Jest uses worker threads (the `jest-worker` package) to run test files in parallel across CPU cores. Each worker is a separate Node.js process. The number of workers defaults to the number of CPU cores minus 1, or can be configured with `--maxWorkers`. Tests within a single file run sequentially.

**Q: What is the difference between toBe and toEqual?**

A: `toBe` uses `Object.is()` for strict reference equality (like `===`). For objects and arrays, it checks if they are the same reference. `toEqual` performs deep equality comparison, recursively checking the values of all properties. Use `toBe` for primitives, `toEqual` for objects and arrays.

**Q: How do you test asynchronous code in Jest?**

A: Three approaches: return the promise (Jest waits for resolution), use `.resolves`/`.rejects` matchers, or use `async/await`. The key is that Jest must know when the async operation completes. Never forget to `return` or `await` the promise, or the test will pass before the assertion runs.

### Coding Challenges

```javascript
// Challenge 1: Implement a custom matcher for sorted arrays
expect.extend({
  toBeSorted(received, order = 'asc') {
    const numbers = received.map(Number)
    const isSorted = order === 'asc'
      ? numbers.every((n, i) => i === 0 || n >= numbers[i - 1])
      : numbers.every((n, i) => i === 0 || n <= numbers[i - 1])

    return {
      pass: isSorted,
      message: () => `expected [${received}] to be sorted ${order}`
    }
  }
})

// Challenge 2: Test timeout helper
function testWithTimeout(name, fn, timeoutMs = 5000) {
  test(name, async () => {
    const timeoutPromise = new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Test timed out')), timeoutMs)
    )
    await Promise.race([fn(), timeoutPromise])
  })
}

// Challenge 3: Snapshot testing with dynamic content cleanup
function createSnapshotSerializer() {
  return {
    test: (val) => val && typeof val === 'string',
    print: (val) => {
      // Remove dynamic content like timestamps
      return val.replace(/\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}/g, '[TIMESTAMP]')
    }
  }
}

expect.addSnapshotSerializer(createSnapshotSerializer())
```

### Related Topics

- `Unit Testing` - Core principles and practices
- `Mocking` - Jest mocking with jest.mock, jest.fn, jest.spyOn
- `Integration Testing` - Testing module interactions with Jest
- `TDD` - Test-Driven Development with Jest
- `React Testing Library` - Component testing framework built on Jest
- `Enzyme` - Legacy React testing utility (compatible with Jest)
- `Snapshot Testing` - Jest's snapshot feature for UI regression
- `Code Coverage` - Istanbul integration for coverage reporting
