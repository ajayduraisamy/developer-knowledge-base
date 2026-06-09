# Integration Testing - Testing modules together, API integration, database integration

## Introduction

Integration testing verifies that different modules, services, or components work together correctly. While unit tests isolate individual units, integration tests validate the interactions between units - how they communicate, share data, and handle errors across boundaries. Integration tests typically involve real databases, API calls, file systems, or external services, and they catch issues that unit tests miss, such as incorrect assumptions about module contracts, data format mismatches, and transaction handling problems.

## Module integration tests

### What It Is

Module integration tests verify that two or more modules work together correctly. Instead of mocking dependencies, these tests use real implementations (or lightweight alternatives) to validate the interaction between components.

```javascript
// Testing database layer with business logic
test('user creation flow with repository', async () => {
  const repository = new UserRepository(database)
  const service = new UserService(repository, emailService)

  const user = await service.createUser({
    name: 'Alice',
    email: 'alice@test.com'
  })

  // Verify user was persisted
  const savedUser = await repository.findById(user.id)
  expect(savedUser.name).toBe('Alice')
})
```

### Why It Is Important

Integration tests catch issues that unit tests miss: incorrect API usage, data format mismatches, transaction rollback failures, authentication middleware misconfigurations, and assumptions about how modules will behave when connected. They provide confidence that the system works as a whole, not just in isolation.

### How It Works Internally

Integration tests typically set up real infrastructure (test databases, test servers) before tests, execute flows that cross module boundaries, and verify outcomes by checking state changes across multiple components. They are slower than unit tests but faster than end-to-end tests.

```javascript
// Integration test setup often involves:
// 1. Starting test database (Docker, in-memory, or test instance)
// 2. Running migrations
// 3. Seeding test data
// 4. Starting application server
// 5. Executing test scenarios
// 6. Cleaning up test data
// 7. Tearing down infrastructure
```

### Beginner Examples

```javascript
// Testing two modules together: UserService + UserRepository
describe('UserService with repository', () => {
  let database
  let repository
  let service

  beforeAll(async () => {
    database = new InMemoryDatabase()
    await database.migrate()
  })

  beforeEach(async () => {
    repository = new UserRepository(database)
    service = new UserService(repository)
    await database.seed({ users: [] })
  })

  afterEach(async () => {
    await database.clear()
  })

  test('creates and retrieves user', async () => {
    const createdUser = await service.createUser({
      name: 'Alice',
      email: 'alice@test.com'
    })

    const retrievedUser = await repository.findById(createdUser.id)
    expect(retrievedUser.name).toBe('Alice')
    expect(retrievedUser.email).toBe('alice@test.com')
  })

  test('enforces unique email constraint', async () => {
    await service.createUser({
      name: 'Alice',
      email: 'alice@test.com'
    })

    await expect(service.createUser({
      name: 'Bob',
      email: 'alice@test.com'
    })).rejects.toThrow('Email already exists')
  })

  test('cascading delete removes related records', async () => {
    const user = await service.createUser({ name: 'Alice', email: 'a@t.com' })
    await service.createPost(user.id, { title: 'Post 1' })

    await service.deleteUser(user.id)

    const posts = await database.query('SELECT * FROM posts WHERE userId = ?', [user.id])
    expect(posts).toHaveLength(0)
  })
})
```

### Intermediate Examples

```javascript
// Express API integration test
const request = require('supertest')
const app = require('../app')
const { createTestUser, generateToken } = require('./test-helpers')

describe('POST /api/users', () => {
  let server

  beforeAll(() => {
    server = app.listen(4001)
  })

  afterAll(async () => {
    await server.close()
    await db.disconnect()
  })

  beforeEach(async () => {
    await db.clear()
  })

  test('creates user with valid data', async () => {
    const response = await request(server)
      .post('/api/users')
      .send({
        name: 'Alice',
        email: 'alice@test.com',
        password: 'SecurePass123!'
      })
      .expect(201)

    expect(response.body).toHaveProperty('id')
    expect(response.body.name).toBe('Alice')
    expect(response.body).not.toHaveProperty('password')
  })

  test('returns 409 for duplicate email', async () => {
    await request(server)
      .post('/api/users')
      .send({ name: 'Alice', email: 'same@test.com', password: 'Pass123!' })

    await request(server)
      .post('/api/users')
      .send({ name: 'Bob', email: 'same@test.com', password: 'Pass123!' })
      .expect(409)
  })

  test('returns 400 for missing fields', async () => {
    const response = await request(server)
      .post('/api/users')
      .send({ name: 'Alice' })
      .expect(400)

    expect(response.body.errors).toContainEqual(
      expect.objectContaining({ field: 'email' })
    )
  })
})

// Integration test with authentication flow
describe('Protected routes', () => {
  let token

  beforeEach(async () => {
    const user = await createTestUser({ role: 'admin' })
    token = generateToken(user)
  })

  test('GET /api/admin/users requires admin role', async () => {
    const userToken = generateToken(await createTestUser({ role: 'user' }))

    await request(server)
      .get('/api/admin/users')
      .set('Authorization', `Bearer ${userToken}`)
      .expect(403)

    await request(server)
      .get('/api/admin/users')
      .set('Authorization', `Bearer ${token}`)
      .expect(200)
  })

  test('refresh token flow', async () => {
    const loginResponse = await request(server)
      .post('/api/auth/login')
      .send({ email: 'admin@test.com', password: 'password' })

    const { accessToken, refreshToken } = loginResponse.body

    // Wait for access token to expire (or use a short-lived test token)
    const refreshResponse = await request(server)
      .post('/api/auth/refresh')
      .send({ refreshToken })

    expect(refreshResponse.body).toHaveProperty('accessToken')
    expect(refreshResponse.body.accessToken).not.toBe(accessToken)
  })
})
```

### Advanced Examples

```javascript
// Database integration test with transactions and rollbacks
describe('Order processing integration', () => {
  let orderService
  let paymentService
  let inventoryService
  let db

  beforeAll(async () => {
    db = await setupTestDatabase()
    await db.runMigrations()
  })

  beforeEach(async () => {
    await db.seed({
      products: [
        { id: 1, name: 'Widget', stock: 10, price: 19.99 },
        { id: 2, name: 'Gadget', stock: 5, price: 49.99 }
      ],
      users: [
        { id: 1, name: 'Alice', email: 'alice@test.com', balance: 200 }
      ]
    })

    orderService = new OrderService(db)
    paymentService = new PaymentService(db)
    inventoryService = new InventoryService(db)
  })

  afterEach(async () => {
    await db.clear()
  })

  test('successful order reduces stock and charges user', async () => {
    const order = await orderService.createOrder(1, [
      { productId: 1, quantity: 2 },
      { productId: 2, quantity: 1 }
    ])

    await paymentService.processPayment(order.id)
    await inventoryService.deductStock(order)

    // Verify stock was deducted
    const widget = await db.products.findById(1)
    expect(widget.stock).toBe(8) // 10 - 2

    const gadget = await db.products.findById(2)
    expect(gadget.stock).toBe(4) // 5 - 1

    // Verify user was charged
    const user = await db.users.findById(1)
    const expectedCharge = (2 * 19.99) + (1 * 49.99)
    expect(user.balance).toBe(200 - expectedCharge)

    // Verify order status
    const savedOrder = await db.orders.findById(order.id)
    expect(savedOrder.status).toBe('completed')
  })

  test('failed payment rolls back inventory deduction', async () => {
    const order = await orderService.createOrder(1, [
      { productId: 1, quantity: 1 }
    ])

    // Simulate payment failure
    jest.spyOn(paymentService, 'processPayment').mockRejectedValue(
      new Error('Insufficient funds')
    )

    try {
      await paymentService.processPayment(order.id)
    } catch (e) {
      // Expected error
    }

    // Manually trigger rollback
    await inventoryService.restoreStock(order.id)

    // Verify inventory was restored
    const widget = await db.products.findById(1)
    expect(widget.stock).toBe(10) // Rolled back
  })

  test('concurrent order processing maintains data integrity', async () => {
    // Simulate two users ordering the last item simultaneously
    const results = await Promise.allSettled([
      orderService.createOrder(1, [{ productId: 1, quantity: 10 }]),
      orderService.createOrder(2, [{ productId: 1, quantity: 10 }])
    ])

    const succeeded = results.filter(r => r.status === 'fulfilled')
    const failed = results.filter(r => r.status === 'rejected')

    expect(succeeded).toHaveLength(1)
    expect(failed).toHaveLength(1)

    const product = await db.products.findById(1)
    expect(product.stock).toBe(0)
  })

  test('full order lifecycle with multiple services', async () => {
    // Create order
    const order = await orderService.createOrder(1, [
      { productId: 1, quantity: 1 }
    ])

    // Process payment
    const payment = await paymentService.processPayment(order.id)
    expect(payment.status).toBe('success')

    // Deduct inventory
    await inventoryService.deductStock(order)

    // Generate invoice
    const invoice = await orderService.generateInvoice(order.id)
    expect(invoice.total).toBe(19.99)
    expect(invoice.lineItems).toHaveLength(1)

    // Send notification
    const notification = await orderService.sendConfirmation(order.id)
    expect(notification.recipient).toBe('alice@test.com')
    expect(notification.type).toBe('order_confirmation')
  })
})

// External API integration test with mock server
describe('Payment gateway integration', () => {
  let paymentGateway
  let mockPaymentServer

  beforeAll(async () => {
    // Start a mock server that mimics the payment API
    mockPaymentServer = await startMockServer({
      port: 9001,
      routes: {
        'POST /api/charge': (req) => {
          if (req.body.amount > 1000) {
            return { status: 200, body: { id: 'ch_123', status: 'success' } }
          }
          return { status: 402, body: { error: 'amount_too_low', message: 'Minimum $10' } }
        }
      }
    })

    paymentGateway = new PaymentGateway('http://localhost:9001')
  })

  afterAll(async () => {
    await mockPaymentServer.close()
  })

  test('processes valid payment', async () => {
    const result = await paymentGateway.charge(1000, 'tok_visa')
    expect(result.status).toBe('success')
    expect(result.id).toMatch(/^ch_/)
  })

  test('rejects payment below minimum', async () => {
    await expect(paymentGateway.charge(500, 'tok_visa'))
      .rejects.toThrow('Minimum $10')
  })
})
```

### Real-World Use Cases

- **API endpoint testing**: Full request-response cycle with middleware, auth, validation
- **Database CRUD operations**: Testing repository methods with real database
- **Message queue integration**: Testing producer-consumer patterns with real queue
- **Third-party API integration**: Testing webhook handling, retry logic
- **File processing pipeline**: Reading, transforming, and writing files
- **Authentication flow**: Login, token refresh, logout cycle
- **Email sending**: Verify email content and delivery through test SMTP
- **Cache integration**: Testing cache-aside, write-through patterns with Redis

### Common Mistakes

```javascript
// Mistake: Not cleaning test data between tests
test('creates user', async () => {
  await createUser({ email: 'test@test.com' })
})
test('finds user', async () => {
  const user = await findUser('test@test.com')
  // Depends on data from previous test!
})

// Mistake: Using the same database for tests and development
// Always use a separate test database

// Mistake: Not testing error paths in integration
test('success path only', async () => {
  // Only tests happy path
  // What about network timeouts? Validation errors? Auth failures?
})

// Mistake: Integration tests that are too slow to run often
// If tests take >5 seconds, developers won't run them

// Mistake: Hard-coding test data assumptions
test('user has id 1', async () => {
  const user = await createUser({ name: 'Test' })
  expect(user.id).toBe(1) // Assumes auto-increment starts at 1
})
```

### Best Practices

```javascript
// 1. Use dedicated test database
beforeAll(async () => {
  process.env.DATABASE_URL = 'postgresql://localhost/test_db'
  db = await createDatabase()
  await db.migrate()
})

// 2. Clean state between tests
beforeEach(async () => {
  await db.truncateAllTables()
})

// 3. Use factories for test data
function buildOrder(overrides = {}) {
  return {
    userId: 1,
    items: [{ productId: 1, quantity: 1 }],
    shippingAddress: { street: '123 Test St' },
    ...overrides
  }
}

// 4. Test both success and failure paths
describe('order cancellation', () => {
  test('cancels pending order', async () => {})
  test('cannot cancel shipped order', async () => {})
  test('cannot cancel already cancelled order', async () => {})
  test('refunds payment on cancellation', async () => {})
})

// 5. Use realistic data
const testProduct = {
  name: 'Test Widget',
  price: 19.99,
  sku: 'TST-001',
  category: 'Widgets'
}

// 6. Keep integration tests focused (not end-to-end)
// Test module interactions, not UI flows
```

### Performance Considerations

- Integration tests are 10-100x slower than unit tests
- Use in-memory databases for faster test execution (SQLite :memory:)
- Database connection pooling improves test performance
- Truncate tables instead of dropping and recreating
- Use transactions with rollback to isolate tests (fastest cleanup)
- Run integration tests separately from unit tests in CI
- Parellize test execution across multiple database instances
- Consider Docker for consistent test infrastructure

### Interview Questions

**Q: What is the difference between unit tests and integration tests?**

A: Unit tests verify individual components in isolation with mocked dependencies. They're fast, deterministic, and catch logic errors. Integration tests verify multiple components working together with real (or near-real) dependencies. They're slower, more complex to set up, and catch interface contract violations, configuration errors, and data format mismatches.

**Q: How do you handle test data in integration tests?**

A: Best practices include: seeding data before test suites, cleaning state between tests (truncate tables or rollback transactions), using factory functions for consistent test data creation, keeping seed data minimal and focused on what each test needs, and never sharing mutable data between tests.

**Q: How do you handle external API dependencies in integration tests?**

A: Options include: mock servers (start a local server that mimics the API), test doubles (use a lightweight implementation), sandbox/test environments (use the provider's test API), or recording/replaying HTTP interactions (like Polly.js or nock). The choice depends on whether you want to test your code's integration with the actual API (sandbox) or just your code's handling of the API's responses (mock server).

### Coding Challenges

```javascript
// Challenge 1: Write integration test for a file upload endpoint
// POST /api/upload - accepts multipart/form-data, stores file, returns metadata

// Challenge 2: Write integration test for a webhook handler
// POST /api/webhooks/payment - receives payment notifications
// Verify: signature validation, idempotency, order status update

// Challenge 3: Write integration test for a search endpoint
// GET /api/search?q=term - returns paginated results from database
// Verify: full-text search, filtering, sorting, pagination
```

### Related Topics

- `Unit Testing` - Foundation for integration testing
- `Jest` - Framework for writing integration tests
- `Mocking` - Mocking external services in integration tests
- `TDD` - Integration tests in the test-driven workflow
- `API Testing` - Testing REST or GraphQL endpoints
- `Database Testing` - Strategies for testing database interactions
- `End-to-End Testing` - Above integration tests in the test pyramid
- `Docker` - Containerizing test infrastructure
