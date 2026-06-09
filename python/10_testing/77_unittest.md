# Unittest - TestCase, setUp, tearDown, assertions, test discovery
## Introduction
Python's built-in `unittest` framework, inspired by Java's JUnit, provides a robust foundation for writing and running tests. It includes test automation, shared setup/shutdown code, aggregation of tests into collections, and independence of tests from the reporting framework. The module is part of the Python standard library, making it available without any external dependencies. Understanding unittest is essential for Python developers as it forms the basis for most testing in enterprise Python environments, CI/CD pipelines, and large-scale codebases. The framework's design follows the xUnit pattern, making it familiar to developers coming from other languages.

## TestCase Class
### What It Is
`unittest.TestCase` is the base class for all unit tests. It provides a rich set of assertion methods, automated test discovery, and lifecycle hooks. Each test case is defined by creating a subclass of `TestCase` and implementing methods whose names begin with `test_`. The framework automatically discovers and runs these methods.

### Why It Is Important
The TestCase class provides a structured, repeatable way to verify code behavior. It ensures that tests are isolated from one another, can be run independently, and produce consistent results. By subclassing TestCase, developers gain access to a comprehensive assertion library and test infrastructure.

### How It Works Internally
When unittest runs, it:
1. Discovers test classes that subclass `TestCase`
2. Instantiates the class for each test method
3. Calls `setUp()` before each test method
4. Calls the test method
5. Calls `tearDown()` after each test method
6. Collects results and reports pass/fail status

```python
import unittest

class TestMathOperations(unittest.TestCase):
    def test_addition(self):
        result = 1 + 1
        self.assertEqual(result, 2)

    def test_subtraction(self):
        result = 5 - 3
        self.assertEqual(result, 2)

if __name__ == '__main__':
    unittest.main()
```

### Syntax
```python
import unittest

class TestFeature(unittest.TestCase):
    def test_functionality(self):
        self.assertTrue(True)
        self.assertIn('key', {'key': 'value'})
```

### Beginner Examples
```python
import unittest

def add(a, b):
    return a + b

class TestAddFunction(unittest.TestCase):
    def test_add_integers(self):
        self.assertEqual(add(2, 3), 5)

    def test_add_floats(self):
        self.assertAlmostEqual(add(0.1, 0.2), 0.3, places=1)

    def test_add_strings(self):
        self.assertEqual(add('hello ', 'world'), 'hello world')
```

### Intermediate Examples
```python
import unittest
import os
import tempfile

class TestFileOperations(unittest.TestCase):
    def setUp(self):
        self.temp_dir = tempfile.mkdtemp()
        self.test_file = os.path.join(self.temp_dir, 'test.txt')
        with open(self.test_file, 'w') as f:
            f.write('initial content')

    def tearDown(self):
        import shutil
        shutil.rmtree(self.temp_dir, ignore_errors=True)

    def test_file_read(self):
        with open(self.test_file, 'r') as f:
            content = f.read()
        self.assertEqual(content, 'initial content')

    def test_file_write(self):
        with open(self.test_file, 'w') as f:
            f.write('new content')
        with open(self.test_file, 'r') as f:
            content = f.read()
        self.assertEqual(content, 'new content')
```

### Advanced Examples
```python
import unittest
from unittest.mock import patch
import requests

class APIClient:
    def fetch_data(self, url):
        response = requests.get(url)
        response.raise_for_status()
        return response.json()

class TestAPIClient(unittest.TestCase):
    def setUp(self):
        self.client = APIClient()

    @patch('requests.get')
    def test_fetch_data_success(self, mock_get):
        mock_get.return_value.status_code = 200
        mock_get.return_value.json.return_value = {'key': 'value'}

        result = self.client.fetch_data('https://api.example.com/data')
        self.assertEqual(result, {'key': 'value'})
        mock_get.assert_called_once_with('https://api.example.com/data')

    @patch('requests.get')
    def test_fetch_data_http_error(self, mock_get):
        mock_get.return_value.raise_for_status.side_effect = requests.HTTPError('404 Not Found')

        with self.assertRaises(requests.HTTPError):
            self.client.fetch_data('https://api.example.com/data')
```

### Real-World Use Cases
- Testing Django models, views, and forms
- Validating API endpoint behavior
- Ensuring database migration correctness
- Regression testing for bug fixes
- CI/CD pipeline test suites

### Common Mistakes
- Forgetting to call `super().setUp()` when overriding `setUp`
- Using `assert` instead of `self.assertEqual()` (assert doesn't provide failure messages)
- Sharing mutable state between tests via class attributes
- Writing tests that depend on execution order

### Best Practices
- Use descriptive test method names that explain the scenario and expected outcome
- Keep tests independent and isolated
- Use `setUp()` for common test data instead of repeating code
- Use `self.assertRaises` context manager for exception testing
- Follow the Arrange-Act-Assert pattern

```python
class TestCalculator(unittest.TestCase):
    def setUp(self):
        self.calc = Calculator()

    def test_divide_by_zero_raises_error(self):
        with self.assertRaises(ValueError):
            self.calc.divide(10, 0)
```

### Performance Considerations
- Test suite execution time grows linearly with the number of tests
- Use `setUpClass` / `tearDownClass` for expensive operations shared across tests
- Use `setUpModule` / `tearDownModule` for module-level setup
- Consider test parallelization with `unittest.parallel` or pytest for large suites

```python
class TestDatabase(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        cls.connection = create_database_connection()

    @classmethod
    def tearDownClass(cls):
        cls.connection.close()
```

### Interview Questions
1. What is the difference between `setUp()` and `setUpClass()`?
2. How does unittest test discovery work?
3. Explain the purpose of `assertRaises`.
4. How can you skip a test conditionally?

### Coding Challenges
1. Write a TestCase for a stack data structure (push, pop, peek, is_empty)
2. Test a function that processes CSV files, using temporary files
3. Write parameterized tests for a palindrome checker function

### Related Topics
- pytest fixtures and parameterization
- unittest.mock for mocking
- doctest for documentation tests
- nose2 as an alternative test runner
- Test-driven development (TDD)

## setUp() and tearDown()
### What It Is
`setUp()` is a method called before each test method to prepare the test environment. `tearDown()` is called after each test method to clean up resources. These methods ensure test isolation and proper resource management.

### Why It Is Important
Without proper setup and teardown, tests become brittle, leak resources, and produce false positives/negatives. These methods establish a clean state for each test, preventing test pollution.

### How It Works Internally
Python's unittest calls `setUp()` before each test method and `tearDown()` after. If `setUp()` raises an exception, the test method is skipped and `tearDown()` is still called. This guarantees cleanup even when setup fails.

```python
import unittest

class TestDatabase(unittest.TestCase):
    def setUp(self):
        self.db = Database('test.db')
        self.db.create_table()

    def tearDown(self):
        self.db.drop_table()
        self.db.close()

    def test_insert_record(self):
        self.db.insert({'id': 1, 'name': 'Alice'})
        record = self.db.find(1)
        self.assertEqual(record['name'], 'Alice')

    def test_delete_record(self):
        self.db.insert({'id': 1, 'name': 'Alice'})
        self.db.delete(1)
        self.assertIsNone(self.db.find(1))
```

## Assertion Methods
### What It Is
unittest provides a comprehensive set of assertion methods that validate conditions and provide descriptive failure messages.

### Common Assertion Methods
```python
self.assertEqual(a, b)          # a == b
self.assertNotEqual(a, b)       # a != b
self.assertTrue(x)              # bool(x) is True
self.assertFalse(x)             # bool(x) is False
self.assertIs(a, b)             # a is b
self.assertIsNot(a, b)          # a is not b
self.assertIsNone(x)            # x is None
self.assertIsNotNone(x)         # x is not None
self.assertIn(a, b)             # a in b
self.assertNotIn(a, b)          # a not in b
self.assertIsInstance(a, b)     # isinstance(a, b)
self.assertNotIsInstance(a, b)  # not isinstance(a, b)
self.assertRaises(Exc, fun)     # fun raises Exc
self.assertRaisesRegex(Exc, r, fun)  # fun raises Exc matching regex
self.assertAlmostEqual(a, b)    # round(a-b, 7) == 0
self.assertNotAlmostEqual(a, b) # round(a-b, 7) != 0
self.assertGreater(a, b)        # a > b
self.assertGreaterEqual(a, b)   # a >= b
self.assertLess(a, b)           # a < b
self.assertLessEqual(a, b)      # a <= b
self.assertRegex(s, r)          # r.search(s)
self.assertNotRegex(s, r)       # not r.search(s)
self.assertCountEqual(a, b)     # a and b have same elements
```

### Custom Assertions
```python
class TestCustomAssertions(unittest.TestCase):
    def assertListSorted(self, lst):
        for i in range(len(lst) - 1):
            self.assertLessEqual(lst[i], lst[i + 1],
                f"List not sorted at index {i}: {lst[i]} > {lst[i + 1]}")

    def test_sorted_list(self):
        self.assertListSorted([1, 2, 3, 4, 5])

    def test_unsorted_list(self):
        with self.assertRaises(AssertionError):
            self.assertListSorted([3, 1, 2])
```

## Test Discovery
### What It Is
Test discovery automatically finds and loads test modules without manually building test suites. unittest discovers tests by looking for modules matching a pattern (default: `test*.py`) in the specified directories.

### Why It Is Important
Automatic discovery eliminates the need to maintain test suite lists. It ensures new tests are automatically included when they follow naming conventions.

### How It Works Internally
The `unittest.TestLoader` class traverses directories, imports modules matching the test pattern, identifies `TestCase` subclasses, and collects methods starting with `test_`.

```python
# From command line:
# python -m unittest discover
# python -m unittest discover -s tests -p "test_*.py"

import unittest

loader = unittest.TestLoader()
suite = loader.discover('tests', pattern='test_*.py', top_level_dir='.')

runner = unittest.TextTestRunner(verbosity=2)
runner.run(suite)
```

### Advanced Discovery Configuration
```python
# custom_loader.py
import unittest

class CustomTestLoader(unittest.TestLoader):
    def getTestCaseNames(self, testCaseClass):
        names = super().getTestCaseNames(testCaseClass)
        return [n for n in names if not n.endswith('_slow')]

loader = CustomTestLoader()
suite = loader.discover('tests')
unittest.TextTestRunner().run(suite)
```

### Test Suites
```python
import unittest

def suite():
    suite = unittest.TestSuite()
    suite.addTest(TestMath('test_addition'))
    suite.addTest(TestString('test_upper'))
    suite.addTest(unittest.makeSuite(TestDatabase))
    return suite

if __name__ == '__main__':
    runner = unittest.TextTestRunner()
    runner.run(suite())
```

### Skipping Tests
```python
import unittest
import sys

class TestSkipping(unittest.TestCase):
    @unittest.skip("demonstrating skipping")
    def test_nothing(self):
        self.fail("shouldn't happen")

    @unittest.skipIf(sys.version_info < (3, 8), "requires Python 3.8+")
    def test_new_feature(self):
        pass

    @unittest.skipUnless(sys.platform.startswith("win"), "requires Windows")
    def test_windows_only(self):
        pass

    @unittest.expectedFailure
    def test_known_failure(self):
        self.assertEqual(1, 0)
```

### Related Topics
- assert methods
- setUp and tearDown
- unittest.mock
- Test suites and loaders
- test discovery patterns
