# TDD - Test-Driven Development, Red-Green-Refactor, TDD benefits

## Introduction

Test-Driven Development (TDD) is a software development methodology where tests are written before the production code they validate. The cycle is simple: write a failing test, write the minimum code to make it pass, then refactor the code while keeping tests passing. TDD shifts testing from an afterthought to a design activity, resulting in cleaner interfaces, better test coverage, and more maintainable code. While it requires discipline, TDD has proven benefits for code quality, design, and developer confidence.

## Red-Green-Refactor cycle

### What It Is

The Red-Green-Refactor cycle is the fundamental rhythm of TDD. Red means writing a test that fails (the new feature doesn't exist yet). Green means writing the minimal code to pass the test. Refactor means improving the code while keeping all tests green.

```javascript
// RED: Write a failing test first
test('adds item to shopping cart', () => {
  const cart = new ShoppingCart()
  cart.addItem({ id: 1, name: 'Widget', price: 10 })
  expect(cart.items).toHaveLength(1)
  expect(cart.total).toBe(10)
})
// This fails because ShoppingCart doesn't exist yet

// GREEN: Write minimal code to pass
class ShoppingCart {
  constructor() {
    this.items = []
    this.total = 0
  }

  addItem(item) {
    this.items.push(item)
    this.total = item.price
  }
}

// REFACTOR: Improve without changing behavior
class ShoppingCart {
  constructor() {
    this.items = []
  }

  addItem(item) {
    this.items.push(item)
  }

  get total() {
    return this.items.reduce((sum, item) => sum + item.price, 0)
  }
}
```

### Why It Is Important

The Red-Green-Refactor cycle provides a disciplined, incremental approach to building software. Each cycle produces a small piece of working, tested functionality. The cycle ensures that every line of production code is covered by tests, that code is never more complex than necessary (YAGNI - You Ain't Gonna Need It), and that refactoring is safe because tests verify behavior isn't broken.

### How It Works Internally

The cycle operates at a granular level, typically cycling every 30-120 seconds. Each iteration adds one specific behavior. The test defines the acceptance criteria, driving the design from the consumer's perspective. The cycle creates a tight feedback loop where developers know immediately if their changes work.

```javascript
// TDD cycle flow:
// 1. Add a small test
// 2. Run all tests - new test fails (RED)
// 3. Write minimal production code
// 4. Run all tests - new test passes (GREEN)
// 5. Refactor as needed
// 6. Run all tests - still green
// 7. Repeat with next test
```

### Syntax

```javascript
// Step 1: RED - Write a failing test
test('calculates fibonacci of 0', () => {
  expect(fibonacci(0)).toBe(0)
})

// Step 2: GREEN - Minimal implementation
function fibonacci(n) {
  return 0
}

// Step 3: Add another test (RED)
test('calculates fibonacci of 1', () => {
  expect(fibonacci(1)).toBe(1)
})

// Step 4: GREEN - Update implementation
function fibonacci(n) {
  if (n === 0) return 0
  return 1
}

// Step 5: Add third test (RED)
test('calculates fibonacci of 2', () => {
  expect(fibonacci(2)).toBe(1)
})

// Step 6: GREEN - Generalize
function fibonacci(n) {
  if (n <= 1) return n
  return fibonacci(n - 1) + fibonacci(n - 2)
}

// Step 7: REFACTOR - Optimize
function fibonacci(n) {
  if (n <= 1) return n
  let prev = 0, curr = 1
  for (let i = 2; i <= n; i++) {
    [prev, curr] = [curr, prev + curr]
  }
  return curr
}
```

### Beginner Examples

```javascript
// Complete TDD example: String calculator

// TEST 1 (RED)
test('returns 0 for empty string', () => {
  expect(add('')).toBe(0)
})

// CODE 1 (GREEN)
function add(numbers) {
  return 0
}

// TEST 2 (RED)
test('returns number for single number', () => {
  expect(add('1')).toBe(1)
})

// CODE 2 (GREEN)
function add(numbers) {
  if (numbers === '') return 0
  return parseInt(numbers)
}

// TEST 3 (RED)
test('sums two comma-separated numbers', () => {
  expect(add('1,2')).toBe(3)
})

// CODE 3 (GREEN)
function add(numbers) {
  if (numbers === '') return 0
  return numbers.split(',').reduce((sum, n) => sum + parseInt(n), 0)
}

// TEST 4 (RED)
test('handles unknown amount of numbers', () => {
  expect(add('1,2,3,4,5')).toBe(15)
})

// CODE 4 (GREEN) - already works!

// TEST 5 (RED)
test('handles newlines as delimiters', () => {
  expect(add('1\n2,3')).toBe(6)
})

// CODE 5 (GREEN)
function add(numbers) {
  if (numbers === '') return 0
  return numbers
    .split(/[,\n]/)
    .reduce((sum, n) => sum + parseInt(n), 0)
}

// REFACTOR
function add(numbers) {
  if (!numbers) return 0
  return numbers
    .split(/[,\n]/)
    .map(Number)
    .reduce((sum, n) => sum + n, 0)
}
```

### Intermediate Examples

```javascript
// TDD for a Todo API

// TEST 1: Create a todo
describe('Todo API', () => {
  test('creates a new todo', async () => {
    const todo = await todoService.create({
      title: 'Learn TDD',
      completed: false
    })

    expect(todo).toHaveProperty('id')
    expect(todo.title).toBe('Learn TDD')
    expect(todo.completed).toBe(false)
  })

  // CODE 1
  // class TodoService {
  //   async create(data) {
  //     const todo = { id: uuid(), ...data, createdAt: new Date() }
  //     await db.todos.insert(todo)
  //     return todo
  //   }
  // }

  // TEST 2: List todos with filtering
  test('lists todos with optional completed filter', async () => {
    await todoService.create({ title: 'Task 1', completed: false })
    await todoService.create({ title: 'Task 2', completed: true })
    await todoService.create({ title: 'Task 3', completed: false })

    const pendingTodos = await todoService.list({ completed: false })
    expect(pendingTodos).toHaveLength(2)

    const completedTodos = await todoService.list({ completed: true })
    expect(completedTodos).toHaveLength(1)

    const allTodos = await todoService.list()
    expect(allTodos).toHaveLength(3)
  })

  // CODE 2
  // async list(filters = {}) {
  //   if (filters.completed !== undefined) {
  //     return db.todos.find({ completed: filters.completed })
  //   }
  //   return db.todos.find({})
  // }

  // TEST 3: Mark todo as complete
  test('marks todo as completed', async () => {
    const todo = await todoService.create({ title: 'Test', completed: false })

    const updated = await todoService.update(todo.id, { completed: true })

    expect(updated.completed).toBe(true)
    expect(updated.completedAt).toBeDefined()
  })

  // CODE 3
  // async update(id, changes) {
  //   if (changes.completed) {
  //     changes.completedAt = new Date()
  //   }
  //   return db.todos.update(id, changes)
  // }

  // TEST 4: Handle not found
  test('throws when updating non-existent todo', async () => {
    await expect(todoService.update('non-existent', { completed: true }))
      .rejects.toThrow('Todo not found')
  })

  // CODE 4
  // async update(id, changes) {
  //   const existing = await db.todos.findById(id)
  //   if (!existing) throw new Error('Todo not found')
  //   ...
  // }

  // TEST 5: Delete todo
  test('deletes todo', async () => {
    const todo = await todoService.create({ title: 'To delete' })
    await todoService.delete(todo.id)

    const todos = await todoService.list()
    expect(todos.find(t => t.id === todo.id)).toBeUndefined()
  })

  // REFACTOR after multiple cycles
  // Extract validation, add error types, improve query performance
})
```

### Advanced Examples

```javascript
// TDD for a rate limiter service with refactoring

// RED: First test - basic rate limiting
describe('RateLimiter', () => {
  test('allows requests within limit', () => {
    const limiter = new RateLimiter({ maxRequests: 3, windowMs: 1000 })

    expect(limiter.isAllowed('user1')).toBe(true)
    expect(limiter.isAllowed('user1')).toBe(true)
    expect(limiter.isAllowed('user1')).toBe(true)
  })

  // GREEN: Minimal implementation
  // class RateLimiter {
  //   constructor({ maxRequests, windowMs }) {
  //     this.maxRequests = maxRequests
  //     this.windowMs = windowMs
  //     this.counts = new Map()
  //   }
  //
  //   isAllowed(userId) {
  //     const count = this.counts.get(userId) || 0
  //     if (count >= this.maxRequests) return false
  //     this.counts.set(userId, count + 1)
  //     return true
  //   }
  // }

  // RED: Test expiration
  test('resets window after time passes', async () => {
    jest.useFakeTimers()
    const limiter = new RateLimiter({ maxRequests: 2, windowMs: 1000 })

    expect(limiter.isAllowed('user1')).toBe(true)
    expect(limiter.isAllowed('user1')).toBe(true)
    expect(limiter.isAllowed('user1')).toBe(false)

    jest.advanceTimersByTime(1000)

    expect(limiter.isAllowed('user1')).toBe(true)
    jest.useRealTimers()
  })

  // GREEN: Add time window tracking
  // class RateLimiter {
  //   constructor({ maxRequests, windowMs }) {
  //     this.maxRequests = maxRequests
  //     this.windowMs = windowMs
  //     this.requests = new Map()
  //   }
  //
  //   isAllowed(userId) {
  //     const now = Date.now()
  //     const timestamps = this.requests.get(userId) || []
  //     const recent = timestamps.filter(t => now - t < this.windowMs)
  //
  //     if (recent.length >= this.maxRequests) return false
  //
  //     recent.push(now)
  //     this.requests.set(userId, recent)
  //     return true
  //   }
  // }

  // RED: Test per-user tracking
  test('tracks limits separately per user', () => {
    const limiter = new RateLimiter({ maxRequests: 2, windowMs: 1000 })

    expect(limiter.isAllowed('alice')).toBe(true)
    expect(limiter.isAllowed('alice')).toBe(true)
    expect(limiter.isAllowed('alice')).toBe(false)

    expect(limiter.isAllowed('bob')).toBe(true)
    expect(limiter.isAllowed('bob')).toBe(true)
    expect(limiter.isAllowed('bob')).toBe(false)
  })

  // REFACTOR: Improve data structure and cleanup
  class RateLimiter {
    constructor({ maxRequests, windowMs }) {
      this.maxRequests = maxRequests
      this.windowMs = windowMs
      this.requests = new Map()
      this.cleanupInterval = setInterval(() => this.cleanup(), windowMs)
    }

    isAllowed(userId) {
      const now = Date.now()
      const timestamps = this.requests.get(userId)

      if (!timestamps) {
        this.requests.set(userId, [now])
        return true
      }

      const recent = timestamps.filter(t => now - t < this.windowMs)

      if (recent.length >= this.maxRequests) {
        return false
      }

      recent.push(now)
      this.requests.set(userId, recent)
      return true
    }

    cleanup() {
      const now = Date.now()
      for (const [userId, timestamps] of this.requests) {
        const recent = timestamps.filter(t => now - t < this.windowMs)
        if (recent.length === 0) {
          this.requests.delete(userId)
        } else {
          this.requests.set(userId, recent)
        }
      }
    }

    destroy() {
      clearInterval(this.cleanupInterval)
    }
  }

  // RED: Test cleanup
  test('cleans up expired entries', () => {
    jest.useFakeTimers()
    const limiter = new RateLimiter({ maxRequests: 5, windowMs: 1000 })

    limiter.isAllowed('alice')
    limiter.isAllowed('bob')

    expect(limiter.requests.size).toBe(2)

    jest.advanceTimersByTime(1500)

    // Trigger cleanup
    limiter.isAllowed('charlie')

    expect(limiter.requests.size).toBe(1) // Only charlie remains
    jest.useRealTimers()
    limiter.destroy()
  })

  // REFACTOR: Extract cleanup logic, add configuration validation
  // Final production-ready version with proper error handling
})
```

### Real-World Use Cases

- **Algorithm implementation**: Sorting, searching, data structures
- **API development**: Building REST endpoints test-first
- **Business logic**: Pricing, validation, workflow engines
- **Data transformation**: Parsing, formatting, mapping functions
- **State management**: Redux reducers and selectors
- **Form validation**: Complex validation rules
- **Configuration management**: Config parsing and defaults
- **Middleware development**: Express middleware with side effects

### Common Mistakes

```javascript
// Mistake: Writing too-large tests
test('full user flow', () => {
  // Tests create, read, update, delete in one test
  // Should be 4 separate tests
})

// Mistake: Skipping the refactor step
// Code works but is poorly structured
// TDD without refactoring = TDD without the main benefit

// Mistake: Writing tests after code
// This is testing, not TDD
// TDD requires test-first for the design benefits

// Mistake: Testing implementation, not behavior
test('uses Map internally', () => {
  // Tests internal data structure
  // Should test behavior (isAllowed returns correct results)
})

// Mistake: Writing too much code to pass a test
function add(a, b) {
  // Wrote full implementation plus error handling
  // Should write minimal code to pass the current test only
  return a + b // Add more tests before adding error handling
}

// Mistake: Forgetting to run tests frequently
// Run tests every 30-60 seconds
// If you go more than 10 minutes without green, something is wrong

// Mistake: Not committing at green
// Always commit when tests are green
// Provides clean checkpoints for backtracking
```

### Best Practices

```javascript
// 1. Start simple, grow incrementally
test('starts with simplest case', () => {
  expect(f('')).toBe('')
})

// 2. One assertion concept per test
test('rejects invalid email', () => {
  expect(validateEmail('')).toBe(false)
})

test('accepts valid email', () => {
  expect(validateEmail('user@example.com')).toBe(true)
})

// 3. Use descriptive test names that describe behavior
test('returns user not found error for non-existent user id')

// 4. Keep the cycle short (30-120 seconds per cycle)
// RED: 10 seconds to write test
// GREEN: 30 seconds to write code
// REFACTOR: 20 seconds to improve

// 5. Run tests after every change
// Use Jest watch mode: npm test -- --watch

// 6. Refactor with confidence
// Tests are your safety net - use them

// 7. Write the test at the right level
// Unit tests for logic, integration tests for boundaries

// 8. When tests are hard to write, the design is wrong
// Pain in testing is feedback about the code design

// 9. Don't test trivial code (getters, setters, simple delegation)
// Use your judgment on what provides value

// 10. Commit at green, not red
// Each green state is a working checkpoint
```

### Performance Considerations

- TDD's tight feedback loop requires fast tests (milliseconds per test)
- Slow tests break the TDD flow - mock I/O dependencies
- Use Jest watch mode for instant feedback
- Each test should be independent (no shared state)
- Test suites should run in under 10 seconds for comfortable TDD
- For slow integration TDD, separate fast unit TDD from slower integration
- Consider using test doubles to keep the cycle fast

### Interview Questions

**Q: What is the difference between TDD and regular testing?**

A: In TDD, tests are written before the production code. Tests drive the design, not just validate it. TDD produces code that is naturally testable, with clear interfaces and loose coupling. Regular testing (tests after code) can validate correctness but doesn't provide the design feedback that TDD does.

**Q: What do you do when a test is hard to write?**

A: Hard-to-write tests are a signal that the code design needs improvement. Common issues include: too many dependencies (needs dependency injection), unclear responsibilities (needs SRP), tight coupling (needs interfaces), or side effects mixed with logic. Step back, refactor the design, then the test becomes easier.

**Q: How do you handle external dependencies in TDD?**

A: Use dependency injection to make dependencies replaceable. In tests, inject mock/stub implementations. This keeps tests fast and focused on the unit's behavior. For integration-level TDD, use test databases or sandbox environments but acknowledge the slower cycle.

### Coding Challenges

```javascript
// Challenge 1: TDD a FizzBuzz function
// Write tests first, then implement:
// - Divisible by 3 -> "Fizz"
// - Divisible by 5 -> "Buzz"
// - Divisible by 3 and 5 -> "FizzBuzz"
// - Otherwise -> the number as string

// Challenge 2: TDD a URL parser
// Write tests first, then implement:
// - Parse protocol, host, path, query string
// - Handle edge cases: missing protocol, trailing slash, special chars

// Challenge 3: TDD an LRU cache
// Write tests first, then implement:
// - Set and get values
// - Evict least recently used when at capacity
// - Update access order on get
// - Handle expired entries
```

### Related Topics

- `Unit Testing` - Foundation of TDD execution
- `Jest` - Framework for writing TDD tests
- `Mocking` - Enables TDD for code with dependencies
- `Integration Testing` - TDD at the integration level
- `Refactoring` - The third step of TDD (Martin Fowler's Refactoring book)
- `SOLID Principles` - Design principles that make TDD easier
- `YAGNI` - You Ain't Gonna Need It (TDD naturally enforces this)
- `Agile Development` - TDD is often practiced within Agile methodologies
