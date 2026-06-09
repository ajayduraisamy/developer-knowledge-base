# Mocking - jest.mock(), jest.fn(), jest.spyOn(), mock implementations

## Introduction

Mocking is a technique in testing where real dependencies are replaced with controlled substitutes that simulate their behavior. In Jest, mocking is a first-class feature with three primary APIs: `jest.mock()` for module-level mocking, `jest.fn()` for creating mock functions, and `jest.spyOn()` for wrapping existing functions with spy behavior. Proper mocking enables true unit isolation, faster test execution, and deterministic verification of complex interactions.

## jest.mock() module mocking

### What It Is

`jest.mock()` automatically replaces all exports of a module with mock implementations. It operates at the module system level, intercepting `require()` or `import` calls to return mock versions instead of the real module.

```javascript
jest.mock('module-name', factory, options)
```

### Why It Is Important

Module mocking is essential for isolating the unit under test from its dependencies. It prevents tests from making real network calls, accessing real databases, or depending on complex module initialization. It also allows testing error conditions that would be difficult to trigger with real implementations.

### How It Works Internally

When `jest.mock()` is called, Jest's module registry is modified. When the test file or any of its dependencies require the mocked module, Jest returns the mock instead of the real module. The mock is created with all exports replaced by `jest.fn()` that return `undefined` by default. This happens before any imports are resolved, which is why `jest.mock()` calls are hoisted to the top of the file.

```javascript
// Internal flow:
// 1. jest.mock('fs') is hoisted to top
// 2. When module loads and encounters require('fs')
// 3. Jest intercepts and returns mock module
// 4. All methods in mock are jest.fn() returning undefined
// 5. Test can customize mock implementations
```

### Syntax

```javascript
// Auto-mock all exports
jest.mock('axios')

// Mock with factory function
jest.mock('axios', () => ({
  get: jest.fn(),
  post: jest.fn(),
  create: jest.fn()
}))

// Mock with implementation
jest.mock('uuid', () => ({
  v4: () => 'fixed-uuid-123'
}))

// Mock ES module default export
jest.mock('config', () => ({
  __esModule: true,
  default: {
    apiUrl: 'http://test-api.com',
    timeout: 5000
  }
}))

// Partial mock (keep real exports, mock specific ones)
jest.mock('fs', () => ({
  ...jest.requireActual('fs'),
  readFileSync: jest.fn()
}))
```

### Beginner Examples

```javascript
// Mocking axios for API tests
jest.mock('axios')
const axios = require('axios')
const { fetchUser } = require('./userService')

test('fetches user successfully', async () => {
  const mockUser = { id: 1, name: 'Alice' }
  axios.get.mockResolvedValue({ data: mockUser })

  const result = await fetchUser(1)

  expect(result).toEqual(mockUser)
  expect(axios.get).toHaveBeenCalledWith('/api/users/1')
})

test('handles API error', async () => {
  axios.get.mockRejectedValue(new Error('Network error'))

  await expect(fetchUser(1)).rejects.toThrow('Network error')
})

// Mocking Node.js built-in module
jest.mock('fs')
const fs = require('fs')
const { readConfig } = require('./configReader')

test('reads config file', () => {
  fs.readFileSync.mockReturnValue(JSON.stringify({ port: 3000 }))

  const config = readConfig('config.json')

  expect(config.port).toBe(3000)
  expect(fs.readFileSync).toHaveBeenCalledWith('config.json', 'utf-8')
})
```

### Intermediate Examples

```javascript
// Mocking with factory for complex modules
jest.mock('../services/emailService', () => ({
  sendEmail: jest.fn().mockResolvedValue({ id: 'email_123', status: 'sent' }),
  sendBatch: jest.fn().mockResolvedValue({ total: 5, failed: 0 }),
  validateEmail: jest.fn().mockReturnValue(true)
}))

const emailService = require('../services/emailService')
const { notifyUser } = require('./notificationService')

test('sends welcome email on registration', async () => {
  await notifyUser('new@test.com', 'welcome')

  expect(emailService.sendEmail).toHaveBeenCalledWith(
    'new@test.com',
    expect.stringContaining('Welcome'),
    expect.objectContaining({ template: 'welcome' })
  )
})

test('handles email service failure', async () => {
  emailService.sendEmail.mockRejectedValueOnce(new Error('SMTP timeout'))

  await expect(notifyUser('test@test.com', 'welcome'))
    .rejects.toThrow('Failed to send notification')

  // Verify fallback behavior
  expect(emailService.sendBatch).toHaveBeenCalled()
})

// Mocking with __mocks__ directory
// __mocks__/stripe.js
const mockStripe = {
  charges: {
    create: jest.fn().mockResolvedValue({
      id: 'ch_mock',
      status: 'succeeded',
      amount: 2000
    })
  },
  customers: {
    create: jest.fn().mockResolvedValue({ id: 'cus_mock' })
  }
}
module.exports = mockStripe

// Test file
jest.mock('../services/stripe') // Uses __mocks__/stripe.js
const stripe = require('../services/stripe')

test('creates charge', async () => {
  const charge = await stripe.charges.create({
    amount: 2000,
    currency: 'usd'
  })
  expect(charge.status).toBe('succeeded')
})
```

### Advanced Examples

```javascript
// Mocking with timers and implementations
jest.mock('../database', () => {
  const mockDb = {
    users: new Map(),
    query: jest.fn().mockImplementation(async (sql, params) => {
      if (sql.includes('SELECT')) {
        const user = mockDb.users.get(params[0])
        return user ? [user] : []
      }
      if (sql.includes('INSERT')) {
        const id = mockDb.users.size + 1
        mockDb.users.set(id, { id, ...params[0] })
        return [{ id }]
      }
      return []
    }),
    connect: jest.fn().mockResolvedValue(true),
    disconnect: jest.fn().mockResolvedValue()
  }
  return mockDb
})

const db = require('../database')
const userService = require('./userService')

test('creates user with database mock', async () => {
  const user = await userService.create({
    name: 'Alice',
    email: 'alice@test.com'
  })

  expect(user).toHaveProperty('id')
  expect(db.query).toHaveBeenCalledTimes(2) // Check + Insert
})

// Mocking with module scope control
beforeAll(() => {
  // Mock affects all tests in this file
  jest.mock('../services/logger', () => ({
    info: jest.fn(),
    error: jest.fn(),
    warn: jest.fn()
  }))
})

// Selective unmocking for specific tests
describe('performance tests', () => {
  beforeAll(() => {
    jest.unmock('../services/cache') // Use real cache
  })
})

// Mocking environment variables
beforeEach(() => {
  process.env.TEST_MODE = 'true'
})

afterEach(() => {
  delete process.env.TEST_MODE
})

// Mock part of a module while keeping rest
jest.mock('../services/fileHandler', () => {
  const actual = jest.requireActual('../services/fileHandler')
  return {
    ...actual,
    deleteFile: jest.fn(), // Mock deleteFile
    // Keep readFile and writeFile real
  }
})
```

## jest.fn() mock functions

### What It Is

`jest.fn()` creates a new mock function that records its calls, arguments, return values, and `this` context. It can be configured with custom implementations, return values, and promises.

```javascript
const mockFn = jest.fn()
const mockFn = jest.fn(implementation)
const mockFn = jest.fn().mockReturnValue(value)
```

### Why It Is Important

`jest.fn()` is the fundamental building block for creating test doubles. It allows verifying that functions were called with correct arguments, controlling return values to test different code paths, and simulating error conditions.

### How It Works Internally

Each mock function maintains a call history including arguments, return values, `this` context, and call count. When a mock is called, it executes its implementation (or returns the configured return value), logs the call, and increments the call counter.

```javascript
// Call tracking
mockFn('arg1', 'arg2')
mockFn(42)

mockFn.mock.calls      // [['arg1', 'arg2'], [42]]
mockFn.mock.results     // [{type: 'return', value: undefined}, ...]
mockFn.mock.instances   // [this1, this2]
mockFn.mock.contexts    // [context1, context2]
```

### Syntax

```javascript
// Creation
jest.fn()
jest.fn(() => 'return value')

// Return values
mockFn.mockReturnValue('value')
mockFn.mockReturnValueOnce('once')
mockFn.mockResolvedValue('async value')
mockFn.mockResolvedValueOnce('async once')
mockFn.mockRejectedValue(new Error('error'))
mockFn.mockRejectedValueOnce(new Error('once'))

// Implementations
mockFn.mockImplementation((a, b) => a + b)
mockFn.mockImplementationOnce((a) => a * 2)

// Assertions
expect(mockFn).toHaveBeenCalled()
expect(mockFn).toHaveBeenCalledTimes(1)
expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2')
expect(mockFn).toHaveBeenLastCalledWith('arg')
expect(mockFn).toHaveBeenNthCalledWith(2, 'arg')

// Access calls
mockFn.mock.calls[0]         // First call arguments
mockFn.mock.results[0].value // First call return value
mockFn.mock.lastCall         // Last call arguments (Jest 29+)
```

### Beginner Examples

```javascript
// Basic mock function
const callback = jest.fn()

test('calls callback on completion', () => {
  executeTask(callback)
  expect(callback).toHaveBeenCalled()
  expect(callback).toHaveBeenCalledTimes(1)
})

// Mock with return value
const calculator = { add: jest.fn().mockReturnValue(10) }

test('uses mock return value', () => {
  const result = calculator.add(2, 3)
  expect(result).toBe(10)
  expect(calculator.add).toHaveBeenCalledWith(2, 3)
})

// Mock implementation
const double = jest.fn(x => x * 2)

test('mock implementation', () => {
  expect(double(5)).toBe(10)
  expect(double(3)).toBe(6)
})

// Promise mocks
const asyncFn = jest.fn().mockResolvedValue('success')

test('async mock', async () => {
  const result = await asyncFn()
  expect(result).toBe('success')
})

// Multiple return values
const mock = jest.fn()
  .mockReturnValueOnce('first')
  .mockReturnValueOnce('second')
  .mockReturnValue('default')

test('sequential returns', () => {
  expect(mock()).toBe('first')
  expect(mock()).toBe('second')
  expect(mock()).toBe('default')
  expect(mock()).toBe('default') // Default continues
})
```

### Intermediate Examples

```javascript
// Mocking for event handlers
const handler = jest.fn()

test('event handler receives event object', () => {
  const button = new Button()
  button.on('click', handler)
  button.click()

  expect(handler).toHaveBeenCalledWith(
    expect.objectContaining({
      type: 'click',
      target: button
    })
  )
})

// Mocking for middleware
const middleware = jest.fn((req, res, next) => next())

test('middleware calls next', () => {
  const req = { user: { id: 1 } }
  const res = {}
  const next = jest.fn()

  middleware(req, res, next)

  expect(next).toHaveBeenCalled()
})

// Mocking Math.random
const mockRandom = jest.fn()

beforeEach(() => {
  mockRandom.mockReturnValue(0.5)
  jest.spyOn(Math, 'random').mockImplementation(() => mockRandom())
})

afterEach(() => {
  jest.restoreAllMocks()
})

test('deterministic random', () => {
  const result = Math.random()
  expect(result).toBe(0.5)
})

// Mock chaining (common in libraries like Mongoose)
const mockQuery = {
  sort: jest.fn().mockReturnThis(),
  populate: jest.fn().mockReturnThis(),
  exec: jest.fn().mockResolvedValue([{ id: 1 }])
}

jest.mock('../models/User', () => ({
  find: jest.fn(() => mockQuery)
}))

test('chained query', async () => {
  const User = require('../models/User')
  const result = await User.find({ active: true }).sort('-created').populate('posts').exec()

  expect(result).toHaveLength(1)
  expect(User.find).toHaveBeenCalledWith({ active: true })
  expect(mockQuery.sort).toHaveBeenCalledWith('-created')
})
```

### Advanced Examples

```javascript
// Mocking with callback patterns
function processUserData(id, callback) {
  const user = { id, name: 'Alice' }
  callback(null, user)
}

test('callback mock with error handling', () => {
  const callback = jest.fn((err, user) => {
    expect(err).toBeNull()
    expect(user.name).toBe('Alice')
  })

  processUserData(1, callback)

  expect(callback).toHaveBeenCalledWith(null, expect.objectContaining({
    id: 1,
    name: 'Alice'
  }))
})

// Complex mock implementation with tracking
const mockValidator = jest.fn().mockImplementation(value => {
  if (typeof value !== 'string') {
    return { valid: false, error: 'Must be string' }
  }
  if (value.length < 3) {
    return { valid: false, error: 'Too short' }
  }
  return { valid: true, error: null }
})

test.each([
  ['', false, 'Too short'],
  ['ab', false, 'Too short'],
  ['abc', true, null],
  [123, false, 'Must be string']
])('validate(%p) => valid:%p', (input, expectedValid) => {
  const result = mockValidator(input)
  expect(result.valid).toBe(expectedValid)
})

// Reset behavior between tests
describe('mock lifecycle', () => {
  const handler = jest.fn()

  beforeEach(() => {
    handler.mockReset()
    // or: handler.mockClear() - reset calls but keep implementation
  })

  test('first scenario', () => {
    handler('a')
    expect(handler).toHaveBeenCalledWith('a')
  })

  test('second scenario isolated', () => {
    expect(handler).not.toHaveBeenCalled() // Reset by beforeEach
  })
})
```

## jest.spyOn() spies

### What It Is

`jest.spyOn()` creates a mock function that wraps an existing method on an object. It records calls and arguments like `jest.fn()`, but by default calls through to the original implementation.

```javascript
jest.spyOn(object, methodName)
jest.spyOn(object, methodName, accessType)
```

### Why It Is Important

Spies are useful when you want to observe calls to existing methods without replacing their behavior, or when you want to temporarily replace a method for a specific test and restore it afterward. Spies provide the safety of automatic restoration.

### How It Works Internally

`jest.spyOn()` replaces the method on the object with a mock function that wraps the original. The original is stored and can be restored with `.mockRestore()`. The spy records all calls while optionally executing the original implementation.

### Syntax

```javascript
// Basic spy
const spy = jest.spyOn(console, 'log')

// Spy with mock implementation
jest.spyOn(console, 'log').mockImplementation(() => {})

// Spy on getter/setter
jest.spyOn(obj, 'property', 'get')
jest.spyOn(obj, 'property', 'set')

// Restore original
spy.mockRestore()

// Assertions (same as jest.fn())
expect(spy).toHaveBeenCalledWith('message')
```

### Beginner Examples

```javascript
// Spy on console.log
test('logs message', () => {
  const spy = jest.spyOn(console, 'log')
  spy.mockImplementation(() => {}) // Suppress console output

  greet('Alice')

  expect(spy).toHaveBeenCalledWith('Hello, Alice!')
  spy.mockRestore()
})

// Spy on Date.now
test('tracks time', () => {
  const spy = jest.spyOn(Date, 'now').mockReturnValue(1000000)

  const result = getTimestamp()

  expect(result).toBe(1000000)
  spy.mockRestore()
})

// Spy on object method
const calculator = {
  add: (a, b) => a + b
}

test('calls add method', () => {
  const spy = jest.spyOn(calculator, 'add')

  const result = calculator.add(2, 3)

  expect(result).toBe(5) // Original still called
  expect(spy).toHaveBeenCalledWith(2, 3)
  spy.mockRestore()
})
```

### Intermediate Examples

```javascript
// Spy on class method
class AuthService {
  async login(email, password) {
    const user = await this.validateCredentials(email, password)
    return this.generateToken(user)
  }

  async validateCredentials(email, password) {
    // Real implementation
  }

  generateToken(user) {
    return `token_${user.id}`
  }
}

test('login calls validateCredentials', async () => {
  const auth = new AuthService()
  const validateSpy = jest.spyOn(auth, 'validateCredentials')
    .mockResolvedValue({ id: 1, email: 'test@test.com' })

  const token = await auth.login('test@test.com', 'password')

  expect(validateSpy).toHaveBeenCalledWith('test@test.com', 'password')
  expect(token).toBe('token_1')

  validateSpy.mockRestore()
})

// Spy with mockImplementationOnce
test('error on first call, success on second', async () => {
  const dbSpy = jest.spyOn(db, 'query')
  dbSpy.mockImplementationOnce(() => { throw new Error('Connection timeout') })
  dbSpy.mockImplementationOnce(() => [{ id: 1 }])

  const service = new UserService(db)

  // First call should trigger retry
  const result = await service.getUser(1)

  expect(result).toEqual({ id: 1 })
  expect(dbSpy).toHaveBeenCalledTimes(2)

  dbSpy.mockRestore()
})

// Spy on global object
const originalFetch = global.fetch
global.fetch = jest.fn()

afterAll(() => {
  global.fetch = originalFetch
})

test('fetch spy', async () => {
  global.fetch.mockResolvedValue({
    ok: true,
    json: () => Promise.resolve({ data: 'test' })
  })

  const response = await fetch('/api/data')
  const data = await response.json()

  expect(data.data).toBe('test')
})
```

### Advanced Examples

```javascript
// Auto-restore with afterEach
describe('spy lifecycle management', () => {
  let spies = []

  afterEach(() => {
    spies.forEach(s => s.mockRestore())
    spies = []
  })

  function spyOn(obj, method) {
    const spy = jest.spyOn(obj, method)
    spies.push(spy)
    return spy
  }

  test('auto-restored spy', () => {
    const logSpy = spyOn(console, 'log')
    console.log('test')
    expect(logSpy).toHaveBeenCalled()
  })

  test('spy from previous test is restored', () => {
    // console.log is original here
  })
})

// Spy with object.defineProperty
test('spy on getter', () => {
  const user = {
    _name: 'Alice',
    get name() { return this._name }
  }

  const spy = jest.spyOn(user, 'name', 'get')

  const name = user.name

  expect(name).toBe('Alice')
  expect(spy).toHaveBeenCalled()

  spy.mockRestore()
})

// Spy tracking with detailed assertions
test('detailed spy call tracking', () => {
  const spy = jest.spyOn(fs, 'writeFileSync')

  writeConfig({ port: 3000 })

  // Check first argument
  expect(spy.mock.calls[0][0]).toBe('config.json')

  // Check second argument was JSON
  const writtenData = JSON.parse(spy.mock.calls[0][1])
  expect(writtenData.port).toBe(3000)

  spy.mockRestore()
})
```

### Real-World Use Cases

- **Module mocking in unit tests**: Mock database, network, file system
- **API client testing**: Mock axios/fetch for controlled responses
- **Time-dependent code**: Mock Date, setTimeout, performance.now
- **Third-party library mocking**: Mock Stripe, AWS SDK, etc.
- **Environment testing**: Mock process.env, window.location
- **Event handler verification**: Assert on callback invocations
- **Error path testing**: Force failures in dependencies
- **Performance optimization**: Mock heavy computations in tests

### Common Mistakes

```javascript
// Mistake: Not restoring mocks between tests
test('test 1', () => {
  jest.spyOn(console, 'log').mockImplementation(() => {})
})

test('test 2', () => {
  // console.log is still mocked!
})

// Mistake: Creating mocks inside test instead of before
test('example', () => {
  jest.mock('../service') // Too late! Import already happened
})

// Mistake: Not clearing mock call history
test('first', () => {
  handler()
  expect(handler).toHaveBeenCalledTimes(1)
})

test('second', () => {
  // handler was called in first test too - still counts!
  expect(handler).toHaveBeenCalledTimes(1) // Actually 2!
})

// Mistake: Over-mocking
// Mocking everything makes tests fragile and tied to implementation

// Mistake: Mocking but not asserting
jest.mock('fs')
test('reads file', () => {
  readConfig()
  // Should assert that fs.readFileSync was called
})

// Mistake: Partial mock with incorrect spread
jest.mock('../module', () => ({
  ...jest.requireActual('../module'), // Might import unmocked dependencies
  mockFn: jest.fn()
}))
```

### Best Practices

```javascript
// 1. Restore mocks in afterEach
afterEach(() => {
  jest.restoreAllMocks()
})

// 2. Clear call history between tests (but keep implementations)
beforeEach(() => {
  jest.clearAllMocks()
})

// 3. Use mockImplementationOnce for varied behavior
mockFn
  .mockImplementationOnce(() => 'first')
  .mockImplementationOnce(() => 'second')
  .mockImplementation(() => 'default')

// 4. Use expect.anything() for irrelevant arguments
expect(mockFn).toHaveBeenCalledWith(
  'important-arg',
  expect.anything()
)

// 5. Verify mock behavior, not implementation
// Good: verify the mock was called correctly
expect(emailService.send).toHaveBeenCalledWith(
  expect.objectContaining({ to: user.email })
)

// 6. Name mocks for better error messages
const myMock = jest.fn().mockName('myMock')
```

### Performance Considerations

- Over-mocking adds overhead (each jest.mock() requires module resolution)
- Module factory functions run once per file (not per test)
- Using jest.restoreAllMocks() is faster than individual mockRestore() calls
- jest.resetAllMocks() is more expensive than jest.clearAllMocks()
- Spies on frequently-called methods (console.log) impact test speed
- Manual mocks in __mocks__ directory are cached and fast
- Avoid creating large mock implementations in beforeEach (use beforeAll)

### Interview Questions

**Q: What is the difference between jest.fn() and jest.spyOn()?**

A: `jest.fn()` creates a standalone mock function with no original implementation. `jest.spyOn()` wraps an existing method on an object, recording calls while preserving the original behavior (unless overridden with mockImplementation). Spies should be restored with mockRestore() to avoid leaking mocks across tests.

**Q: How does jest.mock() hoisting work?**

A: Jest automatically hoists `jest.mock()` calls to the top of the file, before any imports. This ensures the mock is in place before the module is loaded. `jest.mock()` calls are the only Jest functions that are hoisted. `jest.fn()` and `jest.spyOn()` are not hoisted.

**Q: When should you use jest.mock() vs manual dependency injection?**

A: `jest.mock()` is useful for mocking entire modules (filesystem, network, database) where you have no control over how the dependency is created. Manual dependency injection (passing mocks into constructors/functions) is preferred for your own code, as it creates cleaner design and doesn't require Jest-specific mocking.

### Coding Challenges

```javascript
// Challenge 1: Mock a constructor
// lib/EmailClient.js
class EmailClient {
  constructor(config) { /* connects to SMTP */ }
  async send(to, subject, body) { /* sends email */ }
}

// Write test that mocks EmailClient constructor

// Challenge 2: Mock Date and test a function that behaves differently on weekends
// Hint: Use jest.spyOn(Date, 'getDay') via prototype

// Challenge 3: Implement a test helper that creates partial mocks
function createPartialMock(modulePath, mockMethods) {
  return jest.mock(modulePath, () => {
    const actual = jest.requireActual(modulePath)
    const mocks = {}
    mockMethods.forEach(method => {
      mocks[method] = jest.fn()
    })
    return { ...actual, ...mocks }
  })
}
```

### Related Topics

- `Jest` - Testing framework providing mock APIs
- `Unit Testing` - Mocking enables true unit isolation
- `Integration Testing` - Selective mocking for integration tests
- `TDD` - Mocks enable test-first development
- `Dependency Injection` - Design pattern that makes mocking easier
- `Test Doubles` - The broader category (dummies, fakes, stubs, spies, mocks)
- `Module system` - How Jest intercepts module imports
- `Snapshots` - Alternative testing approach in Jest
