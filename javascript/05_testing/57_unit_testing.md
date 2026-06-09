# Unit Testing - Unit test principles, test isolation, mocking dependencies

## Introduction

Unit testing is a software testing methodology where individual units or components of source code are tested in isolation to verify they behave correctly. A unit is the smallest testable part of an application, typically a function, method, or class. Unit tests are automated, run quickly, and provide rapid feedback about code correctness. They form the foundation of the testing pyramid and are essential for maintaining code quality in large codebases.

## Unit test principles (F.I.R.S.T.)

### What It Is

The F.I.R.S.T. principles define five characteristics of well-written unit tests: Fast, Isolated, Repeatable, Self-validating, and Timely. These principles guide developers in creating maintainable and valuable test suites.

```javascript
// F.I.R.S.T. Acronym:
// Fast - Tests should run quickly
// Isolated - Tests should not depend on each other
// Repeatable - Tests should produce the same result every time
// Self-validating - Tests should have a boolean pass/fail outcome
// Timely - Tests should be written at the right time (before code in TDD)
```

### Why It Is Important

Following F.I.R.S.T. principles ensures that unit tests provide value without becoming a maintenance burden. Fast tests encourage frequent execution. Isolated tests prevent cascading failures. Repeatable tests eliminate flakiness. Self-validating tests remove manual inspection. Timely tests (written before code in TDD) guide better design.

### How It Works Internally

Each F.I.R.S.T. principle addresses a specific failure mode in testing:

```javascript
// Fast: Tests run in milliseconds, enabling immediate feedback
test('fast calculation', () => {
  expect(add(1, 2)).toBe(3) // Runs in <1ms
})

// Isolated: Each test has its own setup and teardown
beforeEach(() => {
  db = new InMemoryDatabase() // Fresh instance per test
})

// Repeatable: No reliance on external state
test('always produces same result', () => {
  jest.useFakeTimers()
  jest.setSystemTime(new Date('2024-01-01'))
  expect(getTodaysDate()).toBe('2024-01-01')
  jest.useRealTimers()
})

// Self-validating: Assertions provide clear pass/fail
test('self-validating', () => {
  const result = calculateTotal([10, 20, 30])
  expect(result).toBe(60) // Boolean pass/fail
  // No manual console.log inspection needed
})
```

### Beginner Examples

```javascript
// Fast example
function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1)
}

test('capitalizes first letter', () => {
  expect(capitalize('hello')).toBe('Hello')
})

// Isolated example (no shared state between tests)
test('array push works', () => {
  const arr = []
  arr.push(1)
  expect(arr).toEqual([1])
})

test('array push is isolated', () => {
  const arr = []
  // arr from previous test is not accessible here
  expect(arr).toEqual([]) // Fresh array
})

// Repeatable example
function calculateDiscount(price, code) {
  if (code === 'SAVE10') return price * 0.9
  return price
}

test('SAVE10 code gives 10% discount', () => {
  expect(calculateDiscount(100, 'SAVE10')).toBe(90)
})

test('no code gives full price', () => {
  expect(calculateDiscount(100, null)).toBe(100)
})
```

### Intermediate Examples

```javascript
// Applying FIRST principles in real scenarios
class OrderProcessor {
  constructor(taxCalculator, inventoryService) {
    this.taxCalculator = taxCalculator
    this.inventoryService = inventoryService
  }

  processOrder(order) {
    if (!this.inventoryService.hasStock(order.items)) {
      throw new Error('Insufficient stock')
    }

    const subtotal = order.items.reduce((sum, item) => sum + item.price, 0)
    const tax = this.taxCalculator.calculate(subtotal, order.address)
    const total = subtotal + tax

    return { subtotal, tax, total, orderId: generateId() }
  }
}

// FIRST-compliant tests
describe('OrderProcessor', () => {
  let processor
  let mockTaxCalculator
  let mockInventory

  // Isolated: fresh mocks before each test
  beforeEach(() => {
    mockTaxCalculator = {
      calculate: jest.fn().mockReturnValue(10)
    }
    mockInventory = {
      hasStock: jest.fn().mockReturnValue(true)
    }
    processor = new OrderProcessor(mockTaxCalculator, mockInventory)
  })

  // Fast: no database or network calls
  test('calculates order total correctly', () => {
    const order = {
      items: [{ price: 50 }, { price: 30 }],
      address: { zip: '10001' }
    }

    const result = processor.processOrder(order)

    expect(result.subtotal).toBe(80)
    expect(result.tax).toBe(10)
    expect(result.total).toBe(90)
  })

  // Repeatable: deterministic mock returns
  test('throws when stock insufficient', () => {
    mockInventory.hasStock.mockReturnValue(false)

    const order = { items: [{ price: 50 }] }

    expect(() => processor.processOrder(order))
      .toThrow('Insufficient stock')
  })

  // Self-validating: clear assertions
  test('calls tax calculator with correct arguments', () => {
    const order = {
      items: [{ price: 100 }],
      address: { zip: '94102' }
    }

    processor.processOrder(order)

    expect(mockTaxCalculator.calculate)
      .toHaveBeenCalledWith(100, order.address)
  })
})
```

### Advanced Examples

```javascript
// Testing strategies for complex units
class UserService {
  constructor(userRepository, emailService, logger) {
    this.userRepository = userRepository
    this.emailService = emailService
    this.logger = logger
  }

  async createUser(userData) {
    this.logger.info('Creating user', { email: userData.email })

    const existing = await this.userRepository.findByEmail(userData.email)
    if (existing) {
      throw new DuplicateUserError(userData.email)
    }

    this.validateUserData(userData)

    const user = await this.userRepository.create({
      ...userData,
      createdAt: new Date()
    })

    await this.emailService.sendWelcomeEmail(user.email, user.name)

    this.logger.info('User created', { userId: user.id })
    return user
  }

  validateUserData(data) {
    if (!data.email || !data.email.includes('@')) {
      throw new ValidationError('Invalid email')
    }
    if (!data.name || data.name.length < 2) {
      throw new ValidationError('Name must be at least 2 characters')
    }
  }
}

// Comprehensive unit tests
describe('UserService', () => {
  let service
  let mockRepo
  let mockEmail
  let mockLogger

  beforeEach(() => {
    mockRepo = {
      findByEmail: jest.fn(),
      create: jest.fn()
    }
    mockEmail = {
      sendWelcomeEmail: jest.fn().mockResolvedValue()
    }
    mockLogger = {
      info: jest.fn(),
      error: jest.fn()
    }
    service = new UserService(mockRepo, mockEmail, mockLogger)
  })

  describe('createUser()', () => {
    // Happy path
    test('creates user and sends welcome email', async () => {
      const userData = { email: 'test@test.com', name: 'Alice' }
      const expectedUser = { id: 1, ...userData, createdAt: expect.any(Date) }

      mockRepo.findByEmail.mockResolvedValue(null)
      mockRepo.create.mockResolvedValue(expectedUser)

      const result = await service.createUser(userData)

      expect(result).toEqual(expectedUser)
      expect(mockRepo.findByEmail).toHaveBeenCalledWith('test@test.com')
      expect(mockEmail.sendWelcomeEmail).toHaveBeenCalledWith('test@test.com', 'Alice')
    })

    // Error path: duplicate email
    test('throws DuplicateUserError for existing email', async () => {
      const userData = { email: 'existing@test.com', name: 'Bob' }
      mockRepo.findByEmail.mockResolvedValue({ id: 1 })

      await expect(service.createUser(userData))
        .rejects.toThrow(DuplicateUserError)

      expect(mockRepo.create).not.toHaveBeenCalled()
      expect(mockEmail.sendWelcomeEmail).not.toHaveBeenCalled()
    })

    // Error path: validation
    test('throws ValidationError for invalid data', async () => {
      const invalidData = { email: 'invalid', name: 'A' }

      await expect(service.createUser(invalidData))
        .rejects.toThrow(ValidationError)
    })

    // Verify side effects
    test('logs user creation', async () => {
      const userData = { email: 'test@test.com', name: 'Alice' }
      mockRepo.findByEmail.mockResolvedValue(null)
      mockRepo.create.mockResolvedValue({ id: 1, ...userData })

      await service.createUser(userData)

      expect(mockLogger.info).toHaveBeenCalledTimes(2)
      expect(mockLogger.info).toHaveBeenCalledWith('Creating user', { email: 'test@test.com' })
      expect(mockLogger.info).toHaveBeenCalledWith('User created', { userId: 1 })
    })

    // Edge case: email service failure
    test('handles email service failure gracefully', async () => {
      const userData = { email: 'test@test.com', name: 'Alice' }

      mockRepo.findByEmail.mockResolvedValue(null)
      mockRepo.create.mockResolvedValue({ id: 1, ...userData })
      mockEmail.sendWelcomeEmail.mockRejectedValue(new Error('SMTP error'))

      await expect(service.createUser(userData))
        .rejects.toThrow('SMTP error')

      // User was created but email failed
      expect(mockRepo.create).toHaveBeenCalled()
    })
  })

  describe('validateUserData()', () => {
    test.each([
      [{ email: '', name: 'Test' }, 'Invalid email'],
      [{ email: 'noat', name: 'Test' }, 'Invalid email'],
      [{ email: 'a@b.com', name: '' }, 'Name must be at least 2 characters'],
      [{ email: 'a@b.com', name: 'A' }, 'Name must be at least 2 characters']
    ])('validates %o throws %s', (data, expectedMessage) => {
      expect(() => service.validateUserData(data))
        .toThrow(expectedMessage)
    })

    test('passes with valid data', () => {
      expect(() => service.validateUserData({
        email: 'test@test.com',
        name: 'Alice'
      })).not.toThrow()
    })
  })
})
```

### Real-World Use Cases

- **Business logic validation**: Test pricing, discounts, tax calculations
- **Data transformation**: Test parsing, formatting, and conversion functions
- **State management**: Test reducers, actions, and selectors
- **API client libraries**: Test request building and response parsing
- **Validation logic**: Test input validation rules and error messages
- **Utility functions**: Test sorting, filtering, and search algorithms
- **Configuration**: Test configuration parsing and defaults

### Common Mistakes

```javascript
// Mistake: Testing too much in one test
test('everything', () => {
  expect(validate(input)).toBe(true)
  expect(transform(input)).toEqual(expected)
  expect(save(input)).toBe(true)
  // One assertion fails - which one?
})

// Mistake: Testing implementation details
// Testing private methods indirectly through public API is better

// Mistake: Shared mutable state
let db // Don't do this

beforeAll(() => {
  db = new Database() // Shared across all tests!
})

// Mistake: Conditional assertions
test('maybe works', () => {
  if (someCondition) {
    expect(result).toBe(true)
  }
  // Non-deterministic test!

// Mistake: Testing the framework, not your code
test('Array works', () => {
  expect([1, 2, 3].length).toBe(3) // Testing JavaScript, not your code
})
```

### Best Practices

```javascript
// 1. One concept per test
test('rejects empty email', () => {
  expect(validate('')).toBe(false)
})

test('rejects email without @', () => {
  expect(validate('notanemail')).toBe(false)
})

test('accepts valid email', () => {
  expect(validate('user@example.com')).toBe(true)
})

// 2. Use descriptive test names
test('returns user object with all required fields on successful creation', () => {})

// 3. Arrange-Act-Assert pattern
test('applies discount for VIP customers', () => {
  // Arrange
  const customer = { type: 'VIP', purchaseAmount: 100 }
  const expected = 90

  // Act
  const result = applyDiscount(customer)

  // Assert
  expect(result).toBe(expected)
})

// 4. Use factories for test data
function createUser(overrides = {}) {
  return {
    id: 1,
    name: 'Default Name',
    email: 'default@test.com',
    role: 'user',
    ...overrides
  }
}

test('updates user email', () => {
  const user = createUser({ id: 42 })
  const updated = updateEmail(user, 'new@test.com')
  expect(updated.email).toBe('new@test.com')
})

// 5. Test edge cases
test('handles empty array', () => {})
test('handles null input', () => {})
test('handles maximum value', () => {})
test('handles special characters', () => {})
test('handles concurrent calls', () => {})
```

### Performance Considerations

- Unit tests should run in milliseconds (hundreds per second)
- Avoid real I/O in unit tests (database, filesystem, network)
- Use in-memory data structures instead of databases
- Mock expensive operations (encryption, compression, image processing)
- Keep test data minimal (only what's needed for the assertion)
- Use `beforeEach` for setup that changes; `beforeAll` for immutable setup
- Parallel test execution is automatic in Jest (leverage it)
- Large fixture files slow down test startup

### Interview Questions

**Q: What makes a good unit test?**

A: A good unit test follows the F.I.R.S.T. principles: Fast (runs in milliseconds), Isolated (no dependencies on other tests), Repeatable (same result every run), Self-validating (boolean pass/fail), and Timely (written at the right time). It tests one behavior, has a clear Arrange-Act-Assert structure, uses descriptive names, and focuses on behavior rather than implementation details.

**Q: How do you achieve test isolation?**

A: Test isolation is achieved by: creating fresh instances in beforeEach/afterEach, using dependency injection to substitute real dependencies with mocks, avoiding shared mutable state, using in-memory databases instead of real ones, freezing time with fake timers, and ensuring tests can run in any order without interference.

**Q: What is the difference between stubs, mocks, and fakes?**

A: Stubs provide predetermined answers to calls (used for query methods). Mocks verify that certain methods were called (used for command methods). Fakes are lightweight implementations of real dependencies (like an in-memory database). In Jest, `jest.fn()` creates a function that can act as either a stub or mock depending on whether you assert on its calls.

### Coding Challenges

```javascript
// Challenge 1: Write unit tests for a rate limiter
class RateLimiter {
  constructor(maxRequests, timeWindowMs) {
    this.maxRequests = maxRequests
    this.timeWindowMs = timeWindowMs
    this.requests = new Map()
  }

  isAllowed(userId) {
    const now = Date.now()
    const userRequests = this.requests.get(userId) || []
    const recentRequests = userRequests.filter(t => now - t < this.timeWindowMs)

    if (recentRequests.length >= this.maxRequests) {
      return false
    }

    recentRequests.push(now)
    this.requests.set(userId, recentRequests)
    return true
  }
}

// Tests should cover:
// - Allowed when under limit
// - Blocked when over limit
// - Resets after time window
// - Separate tracking per user

// Challenge 2: Write tests for a function with Date dependency
function getGreeting() {
  const hour = new Date().getHours()
  if (hour < 12) return 'Good morning'
  if (hour < 18) return 'Good afternoon'
  return 'Good evening'
}

// Challenge 3: Write tests for a function with random behavior
function rollDice() {
  return Math.floor(Math.random() * 6) + 1
}
// Hint: Mock Math.random for deterministic testing
```

### Related Topics

- `Jest` - Testing framework for running unit tests
- `Mocking` - Techniques for isolating units from dependencies
- `Integration Testing` - Testing units working together
- `TDD` - Writing tests before code (Test-Driven Development)
- `Test Coverage` - Measuring which code paths are tested
- `SOLID Principles` - Design principles that make code more testable
- `Dependency Injection` - Pattern for making code testable
- `Refactoring` - Using tests as a safety net for code changes
