# TDD - Test-Driven Development, Red-Green-Refactor cycle
## Introduction
Test-Driven Development (TDD) is a software development methodology where tests are written before the production code. The process follows a strict Red-Green-Refactor cycle: write a failing test (Red), make it pass with minimal code (Green), then improve the code (Refactor). TDD was popularized by Kent Beck in the late 1990s as part of Extreme Programming (XP). It shifts testing from a verification activity to a design activity, fundamentally changing how developers approach problem-solving. While TDD may initially feel slower, it produces cleaner, better-designed code with comprehensive test coverage and fewer defects.

## Red-Green-Refactor cycle
### What It Is
The Red-Green-Refactor cycle is the fundamental rhythm of TDD. Each cycle typically lasts 30 seconds to a few minutes. Developers write a single test (Red), implement just enough code to pass it (Green), and then improve the code's structure without changing its behavior (Refactor). This tight feedback loop builds quality into the code from the start.

### Why It Is Important
The cycle enforces discipline that prevents common pitfalls: writing too much code at once, skipping tests, or neglecting code quality. The rapid feedback ensures that defects are caught immediately and code remains clean throughout development.

### How It Works Internally
The cycle operates at multiple levels: micro-cycles within a function, meso-cycles within a feature, and macro-cycles within a release. At each level, the same pattern applies: specify desired behavior through a test, implement the behavior, then improve the implementation.

```
RED (Write a failing test)
  |
  v
GREEN (Write minimal code to pass)
  |
  v
REFACTOR (Improve code quality)
  |
  v
  (Repeat)
```

### Step 1: Red - Write a Failing Test
```python
import unittest

# RED: Write a test for a function that doesn't exist yet
class TestFizzBuzz(unittest.TestCase):
    def test_fizzbuzz_returns_fizz_for_multiples_of_three(self):
        result = fizzbuzz(3)  # fizzbuzz doesn't exist yet!
        self.assertEqual(result, "Fizz")

if __name__ == '__main__':
    unittest.main()
```

### Step 2: Green - Write Minimal Code
```python
# GREEN: Write just enough code to pass the test
def fizzbuzz(n):
    return "Fizz"

class TestFizzBuzz(unittest.TestCase):
    def test_fizzbuzz_returns_fizz_for_multiples_of_three(self):
        result = fizzbuzz(3)
        self.assertEqual(result, "Fizz")
```

### Step 3: Refactor - Improve Code
```python
# GREEN still passes after refactoring
def fizzbuzz(n):
    return "Fizz" if n % 3 == 0 else str(n)

class TestFizzBuzz(unittest.TestCase):
    def test_fizzbuzz_returns_fizz_for_multiples_of_three(self):
        self.assertEqual(fizzbuzz(3), "Fizz")

    def test_fizzbuzz_returns_number_for_non_multiples(self):
        self.assertEqual(fizzbuzz(1), "1")
```

## Writing tests first
### What It Is
Writing tests first means specifying the expected behavior of code before implementing it. This ensures that the code is designed for testability and that tests verify actual requirements rather than implementation details.

### Why It Is Important
Tests written after code often test what the code does rather than what it should do. Writing tests first ensures that every line of production code has a test that drove its creation. It also provides living documentation of the system's expected behavior.

### How It Works Internally
Writing tests first forces developers to think about interfaces before implementations. This leads to better API design because the developer must consider how the code will be used before writing it. The test serves as the first client of the code.

```python
# Step 1: Write test for the desired behavior
import unittest

class TestShoppingCart(unittest.TestCase):
    def test_empty_cart_has_zero_total(self):
        cart = ShoppingCart()
        self.assertEqual(cart.total, 0)

    def test_add_item_increases_total(self):
        cart = ShoppingCart()
        cart.add_item("Apple", 1.50)
        self.assertEqual(cart.total, 1.50)

    def test_add_multiple_items(self):
        cart = ShoppingCart()
        cart.add_item("Apple", 1.50)
        cart.add_item("Banana", 0.75)
        self.assertEqual(cart.total, 2.25)

# Step 2: Implement to pass tests
class ShoppingCart:
    def __init__(self):
        self.items = []
        self.total = 0

    def add_item(self, name, price):
        self.items.append((name, price))
        self.total += price

# Step 3: Refactor with confidence
class ShoppingCart:
    def __init__(self):
        self.items = []

    def add_item(self, name, price):
        self.items.append({"name": name, "price": price})

    @property
    def total(self):
        return sum(item["price"] for item in self.items)
```

### Triangle Kata Example
```python
# Test-First Development of triangle classifier
import unittest

# RED: Write tests
class TestTriangle(unittest.TestCase):
    def test_equilateral_triangle(self):
        self.assertEqual(classify_triangle(3, 3, 3), "equilateral")

    def test_isosceles_triangle(self):
        self.assertEqual(classify_triangle(3, 3, 4), "isosceles")

    def test_scalene_triangle(self):
        self.assertEqual(classify_triangle(3, 4, 5), "scalene")

    def test_invalid_triangle_negative_sides(self):
        self.assertEqual(classify_triangle(-1, 2, 3), "invalid")

    def test_invalid_triangle_inequality(self):
        self.assertEqual(classify_triangle(1, 2, 3), "invalid")

# GREEN: Minimal implementation
def classify_triangle(a, b, c):
    sides = sorted([a, b, c])
    a, b, c = sides
    if a <= 0 or a + b <= c:
        return "invalid"
    if a == b == c:
        return "equilateral"
    if a == b or b == c:
        return "isosceles"
    return "scalene"

# REFACTOR: Clean up
def classify_triangle(*sides):
    a, b, c = sorted(sides)
    if a <= 0 or a + b <= c:
        return "invalid"
    distinct = len({a, b, c})
    return {1: "equilateral", 2: "isosceles", 3: "scalene"}[distinct]
```

## TDD benefits
### What It Is
TDD provides benefits beyond just testing. It influences design, documentation, debugging, and team collaboration.

### Why It Is Important
Understanding the full range of TDD benefits helps developers commit to the practice even when it feels slower. The long-term benefits far outweigh the initial investment.

### Key Benefits

**1. Better API Design**
```python
# TDD forces you to think about the user of your API
# This often leads to cleaner, more intuitive interfaces

# Without TDD: complex constructor
class UserManager:
    def __init__(self, db_config, cache_config, auth_config, logger):
        pass

# With TDD: simpler, tested interface
class UserManager:
    def __init__(self, db, cache, auth, logger):
        self.db = db
        self.cache = cache
        self.auth = auth
        self.logger = logger

# Test drives the design
class TestUserManager(unittest.TestCase):
    def setUp(self):
        self.db = Mock()
        self.cache = Mock()
        self.auth = Mock()
        self.logger = Mock()
        self.manager = UserManager(self.db, self.cache, self.auth, self.logger)
```

**2. Comprehensive Regression Protection**
```python
# Every bug found first needs a test that reproduces it
# This creates a regression suite that grows with the project

class TestRegression(unittest.TestCase):
    def test_edge_case_found_in_production(self):
        """Regression: Empty string caused IndexError in v2.1"""
        result = process_data("")
        self.assertEqual(result, [])

    def test_special_characters_bug(self):
        """Regression: Unicode chars truncated in v2.3"""
        result = process_data("héllo wörld")
        self.assertEqual(result, ["héllo", "wörld"])
```

**3. Living Documentation**
```python
# Tests serve as executable specifications
class TestPaymentProcessing(unittest.TestCase):
    def test_valid_credit_card_charge(self):
        """A valid credit card should be charged successfully"""
        payment = Payment(amount=100, card_number="4111111111111111")
        result = payment_processor.charge(payment)
        self.assertTrue(result.success)
        self.assertEqual(result.amount, 100)

    def test_expired_card_rejected(self):
        """Expired cards should be rejected with appropriate message"""
        payment = Payment(amount=100, card_number="4111111111111111",
                         exp_date="01/20")  # Past date
        with self.assertRaises(PaymentError) as ctx:
            payment_processor.charge(payment)
        self.assertIn("expired", str(ctx.exception))
```

**4. Fearless Refactoring**
```python
# With comprehensive tests, you can refactor aggressively
# The test suite catches regressions instantly

# Original implementation
def calculate_discount(items):
    total = sum(item.price for item in items)
    if total > 100:
        return total * 0.9
    return total

# Refactored implementation (with tests passing)
DISCOUNT_THRESHOLDS = [
    (500, 0.8),
    (200, 0.85),
    (100, 0.9),
    (0, 1.0),
]

def calculate_discount(items):
    total = sum(item.price for item in items)
    for threshold, multiplier in sorted(DISCOUNT_THRESHOLDS, reverse=True):
        if total >= threshold:
            return total * multiplier
    return total
```

**5. Reduced Debugging Time**
```python
# TDD catches bugs within minutes instead of weeks
# When a test fails, the cause is usually the last change

# Typical TDD session: 2-minute cycle
# 1. Write test (30s)
# 2. See it fail (5s)
# 3. Write code (30s)
# 4. See it pass (5s)
# 5. Refactor (30s)
# 6. All tests pass (5s)

# Traditional development: days to find bugs later
```

### TDD Best Practices

```python
import unittest

class TestTDDPractices(unittest.TestCase):
    """Follow these practices for effective TDD"""

    def test_one_assert_per_test(self):
        """Each test should verify ONE behavior"""
        result = calculate(2, 3)
        self.assertEqual(result, 5)

    def test_descriptive_names(self):
        """Test names describe the scenario and expectation"""
        self.assertEqual(
            format_name("john", "doe"),
            "John Doe"
        )

    def test_dont_test_implementation(self):
        """Test behavior, not internal details"""
        result = sort_list([3, 1, 2])
        self.assertEqual(result, [1, 2, 3])
        # Don't test: was sort() or sorted() used internally?

    def test_test_edge_cases(self):
        """Always include boundary conditions"""
        self.assertEqual(process([]), [])  # Empty
        self.assertEqual(process([1]), [1])  # Single
        self.assertEqual(process([1, 1]), [1, 1])  # Duplicates

    def test_red_green_refactor_sequence(self):
        """
        RED: Write test, see it fail
        GREEN: Write simplest code to pass
        REFACTOR: Improve design, keep tests green
        """
        pass
```

### Real-World TDD Flow
```python
"""
Full TDD workflow for building a URL shortener:

RED: Write test for shorten_url
- test_shorten_returns_short_string
- test_shorten_same_url_returns_same_hash
- test_shorten_different_urls_return_different_hashes

GREEN: Implement shorten_url
- Return first 8 chars of base64 encoded hash

REFACTOR: Extract hashing logic
- Create separate HashService class

RED: Add resolve_url test
- test_resolve_returns_original_url
- test_resolve_invalid_hash_raises_error

GREEN: Implement resolve_url
- Store mapping in dictionary

REFACTOR: Extract storage to repository
- Create URLRepository interface

Continue cycle for persistence, API layer, etc.
"""
```

### Common Mistakes
- Writing tests that are too large (testing too much at once)
- Skipping the refactor step
- Writing tests after code instead of before
- Testing implementation instead of behavior
- Not running tests frequently enough
- Writing fragile tests tied to internal structure

### Best Practices
- Keep the cycle short (seconds to minutes)
- Run tests after every change
- Write the simplest code to pass the test
- Refactor only when tests are green
- Use descriptive test names
- Test one behavior per test

### Performance Considerations
- TDD test suites should run in milliseconds per test
- Use mocks for slow dependencies
- Keep unit tests fast (avoid I/O)
- Slow test suites discourage TDD

### Interview Questions
1. Explain the Red-Green-Refactor cycle in detail.
2. Why write tests before code instead of after?
3. How does TDD influence software design?
4. What types of projects benefit most from TDD?
5. How do you handle legacy code with TDD?

### Coding Challenges
1. Implement FizzBuzz using strict TDD
2. Build a calculator with TDD (add, subtract, multiply, divide)
3. Implement a shopping cart with TDD
4. Use TDD to create a palindrome checker

### Related Topics
- Behavior-Driven Development (BDD)
- Acceptance Test-Driven Development (ATDD)
- unittest framework
- pytest framework
- Mocking and test doubles
- Continuous Integration
- Refactoring techniques
