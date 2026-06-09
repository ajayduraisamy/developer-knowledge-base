# Mocking - unittest.mock, Mock, patch(), side_effect, return_value
## Introduction
Mocking is a technique used in testing to replace real objects with simulated ones that mimic their behavior. Python's `unittest.mock` module provides a powerful framework for creating mock objects, controlling their behavior, and verifying interactions. Mocking allows developers to isolate the code under test from its dependencies, such as databases, APIs, file systems, and network services. This isolation makes tests faster, more reliable, and independent of external factors.

## unittest.mock
### What It Is
`unittest.mock` is a library for testing in Python that replaces parts of your system under test with mock objects. It was added to the standard library in Python 3.3 (backported as `mock` for earlier versions). The library provides `Mock`, `MagicMock`, `patch`, `PropertyMock`, and `AsyncMock` classes for comprehensive mocking capabilities.

### Why It Is Important
Mocking enables testing of units in isolation without side effects from external services. It eliminates dependencies on network availability, database state, file system permissions, and third-party APIs. This leads to faster test execution, deterministic results, and the ability to test error conditions that are hard to reproduce with real dependencies.

```python
from unittest.mock import Mock, MagicMock, patch, PropertyMock, AsyncMock
```

## Mock objects
### What It Is
A `Mock` object is a flexible substitute for any Python object. It records how it is used (what calls are made, with what arguments) and can be configured with return values, side effects, and attributes.

### Why It Is Important
Mock objects serve as test doubles that record interactions for later verification. They enable assertions about how code under test interacts with its dependencies, such as confirming that specific methods were called with the right arguments.

### How It Works Internally
When a Mock object is called, it creates a new Mock object as the return value by default (unless configured otherwise). Every attribute access also returns a new Mock. This allows chaining and deep attribute access without configuration. Mocks record all calls in their `call_args`, `call_args_list`, `method_calls`, and `mock_calls` attributes.

```python
from unittest.mock import Mock

# Basic mock creation
mock = Mock()
mock.some_method()
mock.some_method(1, 2, key="value")

# Checking calls
mock.some_method.assert_called()
mock.some_method.assert_called_once()
mock.some_method.assert_called_with(1, 2, key="value")
mock.some_method.assert_called_once_with(1, 2, key="value")

# Call count
print(mock.some_method.call_count)  # 2
print(mock.some_method.call_args)   # call(1, 2, key='value')
print(mock.some_method.call_args_list)  # [call(), call(1, 2, key='value')]
```

### Mock Configuration
```python
from unittest.mock import Mock

# Configuring return values
mock = Mock()
mock.return_value = 42
mock.side_effect = [1, 2, 3]  # Returns different values on consecutive calls

# Configuring method return values
mock.get_user.return_value = {"id": 1, "name": "Alice"}
mock.process_data.side_effect = lambda x: x * 2

# Configuring attributes
mock.some_attribute = "test_value"
mock.nested.attribute.value = 100

# Using PropertyMock
from unittest.mock import PropertyMock
mock_obj = Mock()
type(mock_obj).property_name = PropertyMock(return_value="property_value")
print(mock_obj.property_name)  # "property_value"
```

### MagicMock
MagicMock extends Mock with default implementations of Python magic methods. It supports iteration, context managers, comparison operators, and more.

```python
from unittest.mock import MagicMock

# MagicMock supports magic methods automatically
mock = MagicMock()
mock.__len__.return_value = 5
mock.__iter__.return_value = iter([1, 2, 3])
mock.__getitem__.return_value = "item"
mock.__enter__.return_value = mock  # for context managers

assert len(mock) == 5
assert list(mock) == [1, 2, 3]
assert mock[0] == "item"

# Context manager support
with mock as m:
    assert m is mock
    print("Inside context")
```

### Spec and Autospec
```python
from unittest.mock import Mock, create_autospec

class Database:
    def connect(self, host, port):
        pass
    def query(self, sql):
        return []
    def close(self):
        pass

# Using spec to restrict mock to actual class interface
mock = Mock(spec=Database)
mock.connect("localhost", 5432)  # OK
mock.nonexistent_method()  # Raises AttributeError

# Using autospec to match signatures
auto_mock = create_autospec(Database)
auto_mock.connect("localhost", 5432)  # OK
auto_mock.connect("localhost")  # Raises TypeError: missing required argument
```

## patch() decorator
### What It Is
`patch()` temporarily replaces a target object with a mock during a test. It can be used as a decorator, context manager, or manual start/stop. After the test completes, the original object is restored.

### Why It Is Important
Patching allows replacing dependencies at their usage point rather than at creation. This is essential for testing modules that import and use external libraries. It enables isolation without modifying production code.

### How It Works Internally
`patch()` saves a reference to the original object, replaces it with a mock (or specified object), and restores the original after the test. The target is specified as a string in `'package.module.ClassName'` format, resolving the import path.

```python
from unittest.mock import patch
import os

# As decorator - arguments passed in order of patches
@patch('os.path.exists')
@patch('os.listdir')
def test_file_ops(mock_listdir, mock_exists):
    mock_exists.return_value = True
    mock_listdir.return_value = ['file1.txt', 'file2.txt']

    assert os.path.exists('/some/path') == True
    assert os.listdir('/some/path') == ['file1.txt', 'file2.txt']

# As context manager
def test_with_context():
    with patch('os.getcwd') as mock_getcwd:
        mock_getcwd.return_value = '/fake/dir'
        assert os.getcwd() == '/fake/dir'

# Manual start/stop
def test_manual_patch():
    patcher = patch('os.getpid')
    mock_getpid = patcher.start()
    mock_getpid.return_value = 9999
    assert os.getpid() == 9999
    patcher.stop()
```

### Where to Patch
```python
# WRONG: patching the import location where class is defined
@patch('mylibrary.Database')
def test_wrong(mock_db):
    pass  # Won't work if Database is imported in your module

# CORRECT: patching the module where Database is USED
# mymodule.py
# from mylibrary import Database  -> patch 'mymodule.Database'

# your_module.py
from unittest.mock import patch

@patch('your_module.Database')
def test_database(mock_db):
    instance = mock_db.return_value
    instance.query.return_value = [{"id": 1}]
    # Now when your_module.Database() is called, it returns the mock
```

### patch.object
```python
from unittest.mock import patch

class Calculator:
    @staticmethod
    def add(a, b):
        return a + b

# Patch a specific object's method
with patch.object(Calculator, 'add', return_value=100):
    calc = Calculator()
    assert calc.add(2, 3) == 100  # Returns mocked value

# Verify original functionality restored
calc = Calculator()
assert calc.add(2, 3) == 5  # Original behavior
```

### patch.multiple and patch.dict
```python
from unittest.mock import patch

# Patch multiple attributes at once
@patch.multiple('os', getcwd=Mock(return_value='/fake'), listdir=Mock(return_value=[]))
def test_multiple():
    assert os.getcwd() == '/fake'
    assert os.listdir('.') == []

# Patch dictionary values (e.g., os.environ)
@patch.dict('os.environ', {'DATABASE_URL': 'sqlite:///test.db'})
def test_env():
    assert os.environ['DATABASE_URL'] == 'sqlite:///test.db'

# With clear=True to start with an empty dict
@patch.dict('os.environ', {'MY_VAR': 'value'}, clear=True)
def test_clear_env():
    assert os.environ == {'MY_VAR': 'value'}
```

## return_value
### What It Is
`return_value` is the value returned when a mock is called. It can be set directly or configured to return different values on each call.

```python
from unittest.mock import Mock

# Simple return value
mock = Mock(return_value=42)
assert mock() == 42

# Setting after creation
mock.return_value = 100
assert mock() == 100

# Method return values
mock.fetch_data.return_value = {"status": "ok"}
assert mock.fetch_data() == {"status": "ok"}

# Chained return values
mock.return_value = mock  # Self-referencing for chaining
mock().some_method()  # OK - returns self endlessly
```

## side_effect
### What It Is
`side_effect` provides advanced behavior beyond simple return values. It can raise exceptions, return different values on consecutive calls, or execute custom logic through a callable.

### Why It Is Important
side_effect enables testing error handling, retry logic, and state-dependent behavior. It simulates real-world scenarios like network timeouts, database errors, and varying API responses.

```python
from unittest.mock import Mock

# Raise exceptions
mock = Mock()
mock.side_effect = ValueError("Invalid input")
try:
    mock()
except ValueError as e:
    assert str(e) == "Invalid input"

# Different return values on consecutive calls
mock = Mock()
mock.side_effect = [10, 20, 30, StopIteration]
assert mock() == 10
assert mock() == 20
assert mock() == 30
try:
    mock()
except StopIteration:
    pass

# Callable for custom logic
mock = Mock()
mock.side_effect = lambda x: x ** 2
assert mock(3) == 9
assert mock(5) == 25
```

### Combined with return_value
```python
from unittest.mock import Mock

# side_effect takes precedence over return_value
mock = Mock(return_value=42)
mock.side_effect = [1, 2, 3]
assert mock() == 1  # Uses side_effect
assert mock() == 2
assert mock() == 3
assert mock() == 42  # Falls back to return_value after side_effect exhausted
```

### Real-World side_effect Patterns
```python
from unittest.mock import Mock, patch

# Simulate network requests with different responses
class APIClient:
    def get_user(self, user_id):
        import requests
        response = requests.get(f"https://api.example.com/users/{user_id}")
        response.raise_for_status()
        return response.json()

@patch('requests.get')
def test_api_retry_logic(mock_get):
    # First call fails, second succeeds
    mock_response_ok = Mock()
    mock_response_ok.status_code = 200
    mock_response_ok.json.return_value = {"id": 1, "name": "Alice"}

    mock_get.side_effect = [
        ConnectionError("Timeout"),
        mock_response_ok
    ]

    client = APIClient()
    result = client.get_user(1)
    assert result["name"] == "Alice"
    assert mock_get.call_count == 2
```

### Advanced Mocking Examples
```python
from unittest.mock import Mock, patch, PropertyMock
import json

# Mocking a database cursor
mock_cursor = Mock()
mock_cursor.fetchall.return_value = [("Alice", 30), ("Bob", 25)]
mock_cursor.fetchone.side_effect = [("Alice", 30), None]
mock_cursor.description = [("name",), ("age",)]

@patch('database.connection')
def test_database_query(mock_conn):
    mock_conn.cursor.return_value = mock_cursor
    mock_conn.cursor.return_value.__enter__.return_value = mock_cursor

    results = run_query("SELECT * FROM users")
    assert len(results) == 2

# Mocking datetime
from unittest.mock import patch
import datetime

@patch('datetime.datetime')
def test_date_handling(mock_datetime):
    mock_datetime.now.return_value = datetime.datetime(2024, 1, 15, 12, 0, 0)
    mock_datetime.side_effect = lambda *args, **kw: datetime.datetime(*args, **kw)

    result = process_date()
    assert "2024" in result

# Mocking file operations
mock_file = MagicMock()
mock_file.__enter__.return_value = mock_file
mock_file.read.return_value = '{"key": "value"}'
mock_file.readlines.return_value = ["line1\n", "line2\n"]

@patch('builtins.open')
def test_file_processing(mock_open):
    mock_open.return_value = mock_file
    data = read_json_file("test.json")
    assert data["key"] == "value"
```

### AsyncMock
```python
from unittest.mock import AsyncMock
import asyncio

# For testing async functions
mock_async = AsyncMock()
mock_async.some_async_method.return_value = "async result"

async def test_async():
    result = await mock_async.some_async_method()
    assert result == "async result"

# Async side effects
mock_async.side_effect = [1, 2, 3]
async def test_async_sequence():
    assert await mock_async() == 1
    assert await mock_async() == 2
    assert await mock_async() == 3
```

### Real-World Use Cases
- Mocking external API calls in service tests
- Simulating database failures for error handling tests
- Testing email sending without actual SMTP
- Mocking file system operations in data processing
- Testing retry logic with intermittent failures
- Verifying cache behavior with mocked Redis

### Common Mistakes
- Patching at the wrong location (definition vs. usage point)
- Forgetting to restore patches (always use context manager or decorator)
- Over-mocking: testing implementation details instead of behavior
- Not using `spec` to catch interface mismatches
- Sharing mutable mock state across tests

### Best Practices
- Patch at the usage point, not the definition point
- Use `spec` or `create_autospec` to validate interfaces
- Prefer context managers over manual start/stop
- Verify interactions with `assert_called_once_with`
- Keep mocks simple and focused on the test scenario

### Performance Considerations
- Mocking is fast since no real I/O occurs
- Complex side_effect chains can be slower than simple return_value
- Creating many unique Mock objects has minimal overhead
- Over-mocking can make tests brittle

### Interview Questions
1. What is the difference between `return_value` and `side_effect`?
2. Where should you apply `patch()` - at definition or usage point?
3. How do you mock a context manager?
4. Explain `spec` parameter in Mock.
5. How do you mock async functions?

### Coding Challenges
1. Mock an external API that sometimes times out, testing retry logic
2. Mock a database connection and verify SQL queries
3. Mock `datetime.now()` to test time-sensitive logic

### Related Topics
- pytest monkeypatch fixture
- pytest-mock plugin
- responses library for HTTP mocking
- vcrpy for recording/replaying HTTP
- Factory Boy for test data
