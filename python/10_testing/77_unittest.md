# Unittest - TestCase, setUp, tearDown, assertions, test discovery

## Introduction

Python's built-in `unittest` module provides a robust framework for writing and running tests. Inspired by Java's JUnit, it supports test automation, shared setup/teardown code, aggregation of tests into collections, and test independence from the reporting framework. It is part of the Python standard library, meaning no external dependencies are required.

The framework revolves around `TestCase` subclasses, where each method starting with `test_` is a individual test case. Assertions are performed using a rich set of `assert*` methods inherited from `TestCase`.

## Why It Is Important

- **Standard Library** – No extra dependencies; available in every Python installation.
- **Structured Testing** – Encourages organized, maintainable test suites via classes and inheritance.
- **Test Automation** – Can discover and run tests automatically.
- **CI/CD Ready** – Integrates with virtually all continuous integration pipelines.
- **Mock Integration** – Works seamlessly with `unittest.mock` for isolating code under test.
- **Community Standard** – Widely understood; many open-source projects use it.

## Syntax

```python
import unittest

class TestExample(unittest.TestCase):

    def setUp(self):
        # runs before every test method
        pass

    def tearDown(self):
        # runs after every test method
        pass

    def test_something(self):
        self.assertEqual(1, 1)
        self.assertTrue(True)
        self.assertIn("a", "abc")

if __name__ == "__main__":
    unittest.main()
```

## Examples

### Example 1: Basic TestCase with setUp and tearDown

```python
import unittest

class MathOperations:
    @staticmethod
    def add(a, b):
        return a + b

    @staticmethod
    def subtract(a, b):
        return a - b

    @staticmethod
    def multiply(a, b):
        return a * b

    @staticmethod
    def divide(a, b):
        if b == 0:
            raise ValueError("Cannot divide by zero")
        return a / b

class TestMathOperations(unittest.TestCase):

    def setUp(self):
        self.op = MathOperations()
        self.a = 10
        self.b = 5

    def tearDown(self):
        self.op = None

    def test_add(self):
        result = self.op.add(self.a, self.b)
        self.assertEqual(result, 15)
        self.assertIsInstance(result, (int, float))

    def test_subtract(self):
        result = self.op.subtract(self.a, self.b)
        self.assertEqual(result, 5)

    def test_multiply(self):
        result = self.op.multiply(self.a, self.b)
        self.assertEqual(result, 50)

    def test_divide(self):
        result = self.op.divide(self.a, self.b)
        self.assertEqual(result, 2.0)

    def test_divide_by_zero(self):
        with self.assertRaises(ValueError):
            self.op.divide(self.a, 0)

if __name__ == "__main__":
    unittest.main()
```

### Example 2: Assert Methods Showcase

```python
import unittest

class TestAssertMethods(unittest.TestCase):

    def test_equality(self):
        self.assertEqual(5, 5)
        self.assertNotEqual(5, 6)
        self.assertAlmostEqual(3.14159, 3.14158, places=4)
        self.assertNotAlmostEqual(3.14159, 3.14, places=3)

    def test_truthiness(self):
        self.assertTrue(True)
        self.assertFalse(False)
        self.assertTrue(1)
        self.assertFalse(0)
        self.assertTrue("non-empty")
        self.assertFalse("")

    def test_collections(self):
        self.assertIn(3, [1, 2, 3])
        self.assertNotIn(10, [1, 2, 3])
        self.assertListEqual([1, 2, 3], [1, 2, 3])
        self.assertTupleEqual((1, 2), (1, 2))
        self.assertSetEqual({1, 2, 3}, {3, 2, 1})
        self.assertDictEqual({"a": 1}, {"a": 1})

    def test_comparisons(self):
        self.assertGreater(10, 5)
        self.assertGreaterEqual(5, 5)
        self.assertLess(3, 7)
        self.assertLessEqual(7, 7)

    def test_types_and_identity(self):
        self.assertIsInstance(5, int)
        self.assertNotIsInstance(5, str)
        x = [1, 2, 3]
        y = x
        self.assertIs(x, y)
        self.assertIsNone(None)
        self.assertIsNotNone(0)

    def test_regular_expression(self):
        self.assertRegex("abc123", r"abc\d+")
        self.assertNotRegex("abcxyz", r"\d+")

    def test_exceptions(self):
        with self.assertRaises(ZeroDivisionError):
            1 / 0
        with self.assertRaisesRegex(ValueError, "invalid"):
            int("invalid")

    def test_warnings_and_logs(self):
        import warnings
        with self.assertWarns(DeprecationWarning):
            warnings.warn("deprecated", DeprecationWarning)
        with self.assertLogs("test_logger", level="INFO") as log:
            import logging
            logger = logging.getLogger("test_logger")
            logger.info("test message")
            self.assertIn("test message", log.output[0])
```

### Example 3: TestSuite and TestRunner

```python
import unittest

class TestStringMethods(unittest.TestCase):
    def test_upper(self):
        self.assertEqual("hello".upper(), "HELLO")

    def test_isupper(self):
        self.assertTrue("HELLO".isupper())
        self.assertFalse("Hello".isupper())

    def test_split(self):
        s = "hello world"
        self.assertEqual(s.split(), ["hello", "world"])
        with self.assertRaises(TypeError):
            s.split(2)

class TestListMethods(unittest.TestCase):
    def test_append(self):
        lst = [1, 2, 3]
        lst.append(4)
        self.assertEqual(lst, [1, 2, 3, 4])

    def test_remove(self):
        lst = [1, 2, 3, 2]
        lst.remove(2)
        self.assertEqual(lst, [1, 3, 2])

# Create suites
suite_string = unittest.TestLoader().loadTestsFromTestCase(TestStringMethods)
suite_list = unittest.TestLoader().loadTestsFromTestCase(TestListMethods)

# Combine suites
full_suite = unittest.TestSuite([suite_string, suite_list])

# Run with a custom runner
if __name__ == "__main__":
    runner = unittest.TextTestRunner(verbosity=2)
    result = runner.run(full_suite)

    print(f"\nTests run: {result.testsRun}")
    print(f"Failures: {len(result.failures)}")
    print(f"Errors: {len(result.errors)}")
    print(f"Skipped: {len(result.skipped)}")
```

### Example 4: Skip Decorators and Conditional Skipping

```python
import unittest
import sys
import os

class TestSkipDecorators(unittest.TestCase):

    @unittest.skip("Demonstrating unconditional skip")
    def test_skip(self):
        self.fail("This should not run")

    @unittest.skipIf(sys.version_info < (3, 10), "Requires Python 3.10+")
    def test_skip_if(self):
        import match_case
        self.assertTrue(True)

    @unittest.skipUnless(sys.platform.startswith("win"), "Requires Windows")
    def test_skip_unless_windows(self):
        self.assertEqual(os.name, "nt")

    @unittest.expectedFailure
    def test_expected_failure(self):
        self.assertEqual(1, 0)

    def test_skip_in_test(self):
        if not hasattr(str, "removeprefix"):
            self.skipTest("removeprefix not available")
        result = "HelloWorld".removeprefix("Hello")
        self.assertEqual(result, "World")

    @unittest.skipIf(not hasattr(os, "link"), "os.link not available")
    def test_hard_link(self):
        self.assertTrue(True)
```

### Example 5: subTest for Iterative Testing

```python
import unittest

class TestSubTest(unittest.TestCase):

    def test_multiple_values(self):
        data = [
            ("hello", "HELLO"),
            ("world", "WORLD"),
            ("python", "PYTHON"),
            ("unittest", "UNITTEST"),
            ("subtest", "SUBTEST"),
        ]
        for value, expected in data:
            with self.subTest(value=value):
                result = value.upper()
                self.assertEqual(result, expected)

    def test_database_rows(self):
        rows = [
            {"id": 1, "name": "Alice", "age": 30},
            {"id": 2, "name": "Bob", "age": 25},
            {"id": 3, "name": "Charlie", "age": 35},
        ]
        for row in rows:
            with self.subTest(row_id=row["id"]):
                self.assertIn("name", row)
                self.assertIn("age", row)
                self.assertIsInstance(row["age"], int)
                self.assertGreater(row["age"], 0)

    def test_matrix_operations(self):
        matrices = [
            ([[1, 2], [3, 4]], [[5, 6], [7, 8]], [[19, 22], [43, 50]]),
            ([[1, 0], [0, 1]], [[1, 2], [3, 4]], [[1, 2], [3, 4]]),
        ]
        for i, (a, b, expected) in enumerate(matrices):
            with self.subTest(matrix_pair=i):
                result = self._matrix_multiply(a, b)
                self.assertEqual(result, expected)

    def _matrix_multiply(self, a, b):
        n = len(a)
        result = [[0] * n for _ in range(n)]
        for i in range(n):
            for j in range(n):
                for k in range(n):
                    result[i][j] += a[i][k] * b[k][j]
        return result
```

### Example 6: Mock Integration with unittest.mock

```python
import unittest
from unittest.mock import Mock, patch, MagicMock
from datetime import datetime

class UserService:
    def __init__(self, db_session):
        self.db = db_session

    def get_user(self, user_id):
        return self.db.query("SELECT * FROM users WHERE id = ?", user_id)

    def create_user(self, name, email):
        if self._user_exists(email):
            raise ValueError("User already exists")
        return self.db.execute("INSERT INTO users (name, email) VALUES (?, ?)", name, email)

    def _user_exists(self, email):
        result = self.db.query("SELECT id FROM users WHERE email = ?", email)
        return len(result) > 0

class TestUserService(unittest.TestCase):

    def setUp(self):
        self.mock_db = MagicMock()
        self.service = UserService(self.mock_db)

    def tearDown(self):
        self.mock_db.reset_mock()

    def test_get_user_returns_user(self):
        expected_user = {"id": 1, "name": "Alice", "email": "alice@example.com"}
        self.mock_db.query.return_value = [expected_user]

        result = self.service.get_user(1)

        self.mock_db.query.assert_called_once_with(
            "SELECT * FROM users WHERE id = ?", 1
        )
        self.assertEqual(result, [expected_user])

    def test_create_user_success(self):
        self.mock_db.query.return_value = []
        self.mock_db.execute.return_value = 1

        result = self.service.create_user("Bob", "bob@example.com")

        self.mock_db.query.assert_called_once()
        self.mock_db.execute.assert_called_once()
        self.assertEqual(result, 1)

    def test_create_user_duplicate_raises(self):
        self.mock_db.query.return_value = [{"id": 1}]

        with self.assertRaises(ValueError) as ctx:
            self.service.create_user("Bob", "bob@example.com")

        self.assertEqual(str(ctx.exception), "User already exists")
        self.mock_db.execute.assert_not_called()

    def test_mock_with_patch_decorator(self):
        with patch("datetime.datetime") as mock_datetime:
            mock_datetime.now.return_value = datetime(2025, 1, 1, 12, 0, 0)
            result = mock_datetime.now()
            self.assertEqual(result, datetime(2025, 1, 1, 12, 0, 0))

    def test_mock_side_effect(self):
        mock_query = Mock()
        mock_query.query.side_effect = [["result1"], ["result2"], Exception("DB error")]
        self.assertEqual(mock_query.query(), "result1")
        self.assertEqual(mock_query.query(), "result2")
        with self.assertRaises(Exception):
            mock_query.query()

    def test_mock_call_count_and_args(self):
        self.mock_db.query("q1", 1)
        self.mock_db.query("q2", 2)

        self.assertEqual(self.mock_db.query.call_count, 2)
        self.mock_db.query.assert_any_call("q1", 1)
        self.mock_db.query.assert_any_call("q2", 2)

    @patch("__main__.UserService._user_exists")
    def test_create_user_with_patch(self, mock_exists):
        mock_exists.return_value = False
        self.mock_db.execute.return_value = 99
        result = self.service.create_user("Test", "test@test.com")
        self.assertEqual(result, 99)

if __name__ == "__main__":
    unittest.main()
```

### Example 7: Mock Side Effects and Return Values

```python
import unittest
from unittest.mock import Mock, call

class TestMockAdvanced(unittest.TestCase):

    def test_side_effect_iterable(self):
        mock = Mock()
        mock.side_effect = [1, 2, 3, Exception("Stop")]
        results = []
        for _ in range(3):
            results.append(mock())
        self.assertEqual(results, [1, 2, 3])
        with self.assertRaises(Exception):
            mock()

    def test_side_effect_function(self):
        mock = Mock()
        def side_effect(arg):
            return arg * 2
        mock.side_effect = side_effect
        self.assertEqual(mock(5), 10)
        self.assertEqual(mock("hello"), "hellohello")

    def test_side_effect_exception(self):
        mock = Mock()
        mock.side_effect = ValueError("custom error")
        with self.assertRaises(ValueError):
            mock()

    def test_return_value_and_side_effect_interaction(self):
        mock = Mock(return_value=42)
        mock.side_effect = [1, 2]
        self.assertEqual(mock(), 1)
        self.assertEqual(mock(), 2)
        self.assertEqual(mock(), 42)

    def test_spec_limits_attributes(self):
        class MyClass:
            def method_a(self):
                return "a"
        mock = Mock(spec=MyClass)
        mock.method_a.return_value = "mocked"
        self.assertEqual(mock.method_a(), "mocked")
        with self.assertRaises(AttributeError):
            mock.nonexistent_method()

    def test_mock_reset_mock(self):
        mock = Mock()
        mock("test")
        mock.reset_mock()
        self.assertEqual(mock.call_count, 0)
        mock.assert_not_called()

    def test_mock_assertions(self):
        mock = Mock()
        mock(1, 2, key="value")
        mock(3, 4, key="other")
        mock.assert_called()
        mock.assert_called_once()
        mock.assert_any_call(1, 2, key="value")
        mock.assert_has_calls([
            call(1, 2, key="value"),
            call(3, 4, key="other"),
        ])

    def test_nested_mocks(self):
        mock = Mock()
        mock.parent.child.method.return_value = 42
        result = mock.parent.child.method()
        self.assertEqual(result, 42)
        mock.parent.child.method.assert_called_once()
```

## Beginner Examples

```python
import unittest

# Simple calculator to test
class Calculator:
    def add(self, a, b):
        return a + b

    def subtract(self, a, b):
        return a - b

class TestCalculatorBeginner(unittest.TestCase):

    def test_add_positive_numbers(self):
        calc = Calculator()
        result = calc.add(2, 3)
        self.assertEqual(result, 5)

    def test_add_negative_numbers(self):
        calc = Calculator()
        result = calc.add(-1, -1)
        self.assertEqual(result, -2)

    def test_subtract(self):
        calc = Calculator()
        result = calc.subtract(10, 4)
        self.assertEqual(result, 6)

if __name__ == "__main__":
    unittest.main()
```

## Intermediate Examples

```python
import unittest
from unittest.mock import patch, MagicMock
import json

class APIClient:
    def __init__(self, base_url):
        self.base_url = base_url

    def get_data(self, endpoint):
        import requests
        response = requests.get(f"{self.base_url}/{endpoint}")
        response.raise_for_status()
        return response.json()

class DataProcessor:
    def __init__(self, api_client):
        self.client = api_client

    def process_user_data(self, user_id):
        data = self.client.get_data(f"users/{user_id}")
        return {
            "full_name": f"{data['first_name']} {data['last_name']}",
            "age_group": "adult" if data["age"] >= 18 else "minor",
            "email_domain": data["email"].split("@")[1],
        }

class TestDataProcessorIntermediate(unittest.TestCase):

    def setUp(self):
        self.mock_client = MagicMock()
        self.processor = DataProcessor(self.mock_client)

    def test_process_user_data_adult(self):
        self.mock_client.get_data.return_value = {
            "first_name": "John",
            "last_name": "Doe",
            "age": 30,
            "email": "john@example.com",
        }
        result = self.processor.process_user_data(1)
        self.assertEqual(result["full_name"], "John Doe")
        self.assertEqual(result["age_group"], "adult")
        self.assertEqual(result["email_domain"], "example.com")

    def test_process_user_data_minor(self):
        self.mock_client.get_data.return_value = {
            "first_name": "Jane",
            "last_name": "Smith",
            "age": 15,
            "email": "jane@school.edu",
        }
        result = self.processor.process_user_data(2)
        self.assertEqual(result["age_group"], "minor")

    def test_api_error_handling(self):
        self.mock_client.get_data.side_effect = ConnectionError("API timeout")
        with self.assertRaises(ConnectionError):
            self.processor.process_user_data(999)

    @patch("builtins.open", new_callable=MagicMock)
    def test_file_processing(self, mock_open):
        mock_open.return_value.__enter__.return_value.read.return_value = json.dumps({"key": "value"})
        import json as j
        data = j.loads(mock_open().__enter__.read())
        self.assertEqual(data["key"], "value")
```

## Advanced Examples

```python
import unittest
from unittest.mock import Mock, PropertyMock, patch, AsyncMock
from unittest import IsolatedAsyncioTestCase
import asyncio

class AdvancedService:
    def __init__(self):
        self._status = "stopped"

    @property
    def status(self):
        return self._status

    @status.setter
    def status(self, value):
        self._status = value

    async def async_operation(self, delay=0.1):
        await asyncio.sleep(delay)
        return "completed"

    def complex_operation(self, items):
        results = []
        for item in items:
            if item.get("active", False):
                results.append(item["value"] * 2)
        return results

class TestAdvancedFeatures(unittest.TestCase):

    def test_property_mock(self):
        service = Mock()
        type(service).status = PropertyMock(return_value="running")
        self.assertEqual(service.status, "running")

    def test_property_mock_side_effects(self):
        service = Mock()
        mock_prop = PropertyMock(side_effect=["running", "stopped", "error"])
        type(service).status = mock_prop
        self.assertEqual(service.status, "running")
        self.assertEqual(service.status, "stopped")
        self.assertEqual(service.status, "error")
        self.assertEqual(mock_prop.call_count, 3)

    def test_sentinel(self):
        from unittest.mock import sentinel
        mock = Mock()
        mock.method.return_value = sentinel.RETURN_VALUE
        result = mock.method()
        self.assertIs(result, sentinel.RETURN_VALUE)

    def test_sentinel_uniqueness(self):
        from unittest.mock import sentinel
        self.assertIsNot(sentinel.A, sentinel.B)
        self.assertEqual(sentinel.A, sentinel.A)

    def test_nested_patch_multiple(self):
        with patch("time.sleep", return_value=None), \
             patch("random.randint", return_value=42):
            import random
            self.assertEqual(random.randint(1, 100), 42)

    def test_configure_mock(self):
        mock = Mock()
        mock.configure_mock(
            name="test",
            age=30,
            nested={"key": "value"},
        )
        self.assertEqual(mock.name, "test")
        self.assertEqual(mock.age, 30)
        self.assertEqual(mock.nested, {"key": "value"})

    def test_attach_mock(self):
        parent = Mock()
        child = Mock()
        parent.attach_mock(child, "child_method")
        parent.child_method(1, 2)
        child.assert_called_with(1, 2)

class TestAsyncMock(IsolatedAsyncioTestCase):

    async def test_async_mock(self):
        mock = AsyncMock()
        mock.async_method.return_value = 42
        result = await mock.async_method()
        self.assertEqual(result, 42)

    async def test_async_mock_side_effect(self):
        mock = AsyncMock()
        mock.async_method.side_effect = [1, 2, ValueError("error")]
        self.assertEqual(await mock.async_method(), 1)
        self.assertEqual(await mock.async_method(), 2)
        with self.assertRaises(ValueError):
            await mock.async_method()

    async def test_async_with_context_manager(self):
        mock = AsyncMock()
        mock.__aenter__.return_value = mock
        mock.__aexit__.return_value = False
        async with mock as m:
            self.assertIs(m, mock)

    async def test_async_for_loop(self):
        mock = AsyncMock()
        mock.__aiter__.return_value = mock
        mock.__anext__.side_effect = [1, 2, 3, StopAsyncIteration()]
        results = []
        async for item in mock:
            results.append(item)
        self.assertEqual(results, [1, 2, 3])

    async def test_real_async_service(self):
        service = AdvancedService()
        result = await service.async_operation(0.01)
        self.assertEqual(result, "completed")
```

## Real-World Use Cases

- **Web Application Testing** – Test Flask/Django views, request handling, authentication.
- **Data Pipeline Validation** – Assert ETL transforms produce correct outputs.
- **API Contract Testing** – Verify endpoints return expected status codes and payloads.
- **Database Migration Testing** – Ensure schema changes preserve data integrity.
- **Configuration Validation** – Verify parsing logic for YAML/JSON/TOML config files.
- **CLI Tool Testing** – Mock stdin/stdout to test command-line interfaces.

## Common Mistakes

1. **Forgetting to call `super().setUp()`** in inherited TestCase classes.
2. **Modifying shared state** across tests instead of using fresh setUp.
3. **Using bare `assert`** instead of `self.assert*` methods.
4. **Testing implementation details** rather than behavior.
5. **Mocking too broadly** – mocking everything defeats the purpose of tests.
6. **Not resetting mocks** between tests leads to cross-test contamination.

## Best Practices

1. **One assertion per test method** where possible – makes failures easier to diagnose.
2. **Use descriptive test names** that explain the scenario and expected outcome.
3. **Keep tests independent** – no test should depend on another test's state.
4. **Use setUp/tearDown** for repeated setup logic.
5. **Test edge cases** – empty inputs, boundary values, error conditions.
6. **Use `subTest`** for parameterized testing without external libraries.
7. **Avoid `assertTrue` for comparisons** – use `assertEqual`, `assertIn`, etc. for better error messages.
8. **Run tests in isolation** using `python -m unittest discover`.

## Interview Questions

1. **What is the difference between `assertEqual` and `assertIs`?** – `assertEqual` checks value equality (`==`), `assertIs` checks identity (`is`).
2. **Explain setUp and tearDown lifecycle** – setUp runs before each test method, tearDown after each, ensuring test isolation.
3. **How do you skip a test conditionally?** – Use `@unittest.skipIf(condition, reason)` or `self.skipTest(reason)`.
4. **What is the purpose of `subTest`?** – To continue testing after a failure within a loop, reporting all failures rather than stopping at the first.
5. **How does `mock.patch` work?** – It temporarily replaces an object's attribute with a mock during a test scope (decorator or context manager).
6. **What is the difference between Mock and MagicMock?** – MagicMock has default implementations of magic methods (`__len__`, `__iter__`, etc.).
7. **How do you test exceptions?** – Use `with self.assertRaises(SomeException):` context manager.

## Coding Challenges

1. Write a test suite for a `Stack` class with push, pop, peek, is_empty, and size methods. Test edge cases like popping from an empty stack.
2. Write tests for a function `parse_csv_line(line: str) -> list[str]` that handles quoted fields, escaped quotes, and empty values.
3. Create a mock-based test for a `WeatherService` that fetches data from an external API and returns formatted temperature strings.
4. Implement a parametrized test (using subTest) for a prime number checker that tests 50+ values including edge cases.
5. Write a unittest suite for a custom `LRUCache` class, testing eviction order, thread safety, and capacity limits.

## Summary

The `unittest` module is Python's built-in testing framework, providing TestCase, assertion methods, fixtures (setUp/tearDown), test discovery, suites, runners, and seamless mock integration. It follows xUnit conventions and is suitable for projects of all sizes, from simple scripts to large-scale applications. Its zero-dependency nature and deep integration with the standard library make it a reliable choice for Python testing.

## Related Topics

- [pytest](./78_pytest.md) – A more feature-rich testing framework with fixtures and plugins.
- [Mocking](./79_mocking.md) – Detailed coverage of unittest.mock patterns.
- [TDD](./80_tdd.md) – Test-Driven Development methodology.
- [Integration Testing](./81_integration_testing.md) – Testing across component boundaries.
- [doctest](https://docs.python.org/3/library/doctest.html) – Lightweight testing via docstring examples.
