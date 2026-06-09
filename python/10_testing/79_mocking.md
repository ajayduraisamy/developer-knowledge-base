# Mocking - unittest.mock, Mock, patch(), side_effect, return_value

## Introduction

Mocking is a technique used in testing to replace real objects with simulated ones that mimic their behavior. Python's `unittest.mock` module provides a powerful set of tools for creating mock objects, controlling their behavior, and verifying how they are used. Mocking is essential for isolating the code under test from its dependencies, such as databases, APIs, file systems, and network services.

The core classes are `Mock` and `MagicMock`, along with the `patch()` function for temporarily replacing objects in modules.

## Why It Is Important

- **Isolation** – Tests focus on the unit under test, not external dependencies.
- **Speed** – Mocked operations run in microseconds instead of seconds.
- **Reliability** – Eliminates flaky tests caused by network/DB failures.
- **Determinism** – Mocked services return predictable values.
- **Cost** – Avoids hitting paid APIs or consuming cloud resources during testing.
- **Edge Cases** – Simulate errors, timeouts, and rare conditions easily.
- **Verification** – Assert call counts, arguments, and call order.

## Syntax

```python
from unittest.mock import Mock, MagicMock, patch, PropertyMock, AsyncMock, sentinel

# Basic mock
mock = Mock()
mock.method.return_value = 42
mock.method.assert_called_once()

# patch decorator
@patch("module.function")
def test_something(mock_function):
    mock_function.return_value = "mocked"

# patch context manager
with patch("module.Class") as MockClass:
    instance = MockClass.return_value
    instance.method.return_value = "mocked"
```

## Examples

### Example 1: Mock Basics and MagicMock

```python
from unittest.mock import Mock, MagicMock, call
import unittest

class TestMockBasics(unittest.TestCase):

    def test_mock_creation(self):
        mock = Mock()
        self.assertIsInstance(mock, Mock)

    def test_mock_return_value(self):
        mock = Mock(return_value=10)
        self.assertEqual(mock(), 10)

    def test_mock_method_return(self):
        mock = Mock()
        mock.get_data.return_value = {"id": 1, "name": "Alice"}
        result = mock.get_data()
        self.assertEqual(result["name"], "Alice")

    def test_mock_attribute(self):
        mock = Mock()
        mock.name = "test_service"
        self.assertEqual(mock.name, "test_service")

    def test_magic_mock_defaults(self):
        mm = MagicMock()
        self.assertEqual(len(mm), 0)
        self.assertIn(1, mm)
        self.assertEqual(str(mm), "")
        self.assertEqual(int(mm), 1)
        self.assertEqual(bool(mm), True)
        self.assertEqual(list(mm), [])
        self.assertEqual(dict(mm), {})

    def test_magic_mock_custom(self):
        mm = MagicMock()
        mm.__len__.return_value = 42
        self.assertEqual(len(mm), 42)

    def test_mock_as_context_manager(self):
        mock = Mock()
        mock.__enter__.return_value = "entered"
        mock.__exit__.return_value = False
        with mock as m:
            self.assertEqual(m, "entered")
```

### Example 2: patch() Decorator and Context Manager

```python
from unittest.mock import patch
import unittest
import os

class TestPatchDecorator(unittest.TestCase):

    @patch("os.getcwd")
    def test_patch_decorator(self, mock_getcwd):
        mock_getcwd.return_value = "/fake/dir"
        self.assertEqual(os.getcwd(), "/fake/dir")
        mock_getcwd.assert_called_once()

    @patch("os.listdir")
    @patch("os.getcwd")
    def test_multiple_patches(self, mock_getcwd, mock_listdir):
        mock_getcwd.return_value = "/fake"
        mock_listdir.return_value = ["file1.txt", "file2.txt"]
        self.assertEqual(os.getcwd(), "/fake")
        self.assertEqual(os.listdir("."), ["file1.txt", "file2.txt"])

    def test_patch_context_manager(self):
        with patch("os.path.exists") as mock_exists:
            mock_exists.return_value = True
            self.assertTrue(os.path.exists("/any/path"))
            mock_exists.assert_called_once_with("/any/path")

    def test_patch_object(self):
        with patch.object(os, "sep", "\\"):
            self.assertEqual(os.sep, "\\")

    def test_patch_dict(self):
        with patch.dict("os.environ", {"TEST_KEY": "test_value"}):
            self.assertEqual(os.environ["TEST_KEY"], "test_value")

    def test_patch_multiple_context(self):
        with patch("os.getcwd") as mock_cwd, \
             patch("os.listdir") as mock_list:
            mock_cwd.return_value = "/tmp"
            mock_list.return_value = []
            self.assertEqual(os.getcwd(), "/tmp")
            self.assertEqual(os.listdir(), [])
```

### Example 3: side_effect and return_value

```python
from unittest.mock import Mock
import unittest

class TestSideEffect(unittest.TestCase):

    def test_side_effect_iterable(self):
        mock = Mock()
        mock.side_effect = [10, 20, 30, StopIteration]
        self.assertEqual(mock(), 10)
        self.assertEqual(mock(), 20)
        self.assertEqual(mock(), 30)
        with self.assertRaises(StopIteration):
            mock()

    def test_side_effect_function(self):
        mock = Mock()
        def compute(arg):
            return arg ** 2
        mock.side_effect = compute
        self.assertEqual(mock(2), 4)
        self.assertEqual(mock(5), 25)
        self.assertEqual(mock(10), 100)

    def test_side_effect_exception(self):
        mock = Mock()
        mock.side_effect = ValueError("custom error")
        with self.assertRaises(ValueError) as ctx:
            mock()
        self.assertEqual(str(ctx.exception), "custom error")

    def test_side_effect_different_exceptions(self):
        mock = Mock()
        mock.side_effect = [ValueError("first"), TypeError("second"), RuntimeError("third")]
        for exc_type in [ValueError, TypeError, RuntimeError]:
            with self.assertRaises(exc_type):
                mock()

    def test_return_value_with_side_effect(self):
        mock = Mock(return_value=99)
        mock.side_effect = [1, 2]
        self.assertEqual(mock(), 1)
        self.assertEqual(mock(), 2)
        self.assertEqual(mock(), 99)
        self.assertEqual(mock(), 99)

    def test_side_effect_none(self):
        mock = Mock(return_value=42)
        self.assertEqual(mock(), 42)
        mock.side_effect = None
        self.assertEqual(mock(), 42)
```

### Example 4: Spec and Autospec

```python
from unittest.mock import Mock, create_autospec
import unittest

class DataProcessor:
    def process(self, data: list) -> dict:
        return {"count": len(data), "items": data}

    def validate(self, value: str) -> bool:
        return bool(value) and len(value) > 3

class TestSpec(unittest.TestCase):

    def test_mock_with_spec(self):
        mock = Mock(spec=DataProcessor)
        mock.process.return_value = {"count": 0, "items": []}
        self.assertEqual(mock.process([]), {"count": 0, "items": []})

    def test_spec_prevents_unknown_methods(self):
        mock = Mock(spec=DataProcessor)
        with self.assertRaises(AttributeError):
            mock.unknown_method()

    def test_spec_prevents_unknown_attributes(self):
        mock = Mock(spec=DataProcessor)
        with self.assertRaises(AttributeError):
            mock.nonexistent_attr

    def test_create_autospec(self):
        mock = create_autospec(DataProcessor)
        mock.process.return_value = {"count": 3}
        result = mock.process([1, 2, 3])
        self.assertEqual(result["count"], 3)

    def test_autospec_validates_signature(self):
        mock = create_autospec(DataProcessor)
        mock.process([1, 2])
        mock.process.assert_called_once_with([1, 2])

    def test_spec_set(self):
        mock = Mock(spec_set=DataProcessor)
        with self.assertRaises(AttributeError):
            mock.new_attr = "value"
```

### Example 5: call_count and Assertion Methods

```python
from unittest.mock import Mock, call
import unittest

class TestCallAssertions(unittest.TestCase):

    def setUp(self):
        self.mock = Mock()

    def test_assert_called(self):
        self.mock()
        self.mock.assert_called()

    def test_assert_called_once(self):
        self.mock()
        self.mock.assert_called_once()
        self.mock()
        with self.assertRaises(AssertionError):
            self.mock.assert_called_once()

    def test_assert_called_with(self):
        self.mock(1, 2, key="value")
        self.mock.assert_called_with(1, 2, key="value")

    def test_assert_called_once_with(self):
        self.mock(42)
        self.mock.assert_called_once_with(42)

    def test_assert_any_call(self):
        self.mock(1)
        self.mock(2)
        self.mock(3)
        self.mock.assert_any_call(2)

    def test_assert_has_calls(self):
        self.mock(1)
        self.mock(2)
        self.mock(3)
        self.mock.assert_has_calls([call(1), call(2), call(3)])

    def test_assert_has_calls_any_order(self):
        self.mock(3)
        self.mock(1)
        self.mock.assert_has_calls([call(1), call(3)], any_order=True)

    def test_call_count_property(self):
        for _ in range(5):
            self.mock()
        self.assertEqual(self.mock.call_count, 5)

    def test_call_args(self):
        self.mock(1, 2, key="value")
        self.assertEqual(self.mock.call_args, call(1, 2, key="value"))
        args, kwargs = self.mock.call_args
        self.assertEqual(args, (1, 2))
        self.assertEqual(kwargs, {"key": "value"})

    def test_call_args_list(self):
        self.mock("first")
        self.mock("second")
        self.assertEqual(self.mock.call_args_list, [call("first"), call("second")])

    def test_method_calls(self):
        self.mock.method_a(1)
        self.mock.method_b(2)
        self.assertEqual(self.mock.method_calls, [call.method_a(1), call.method_b(2)])
```

### Example 6: PropertyMock 

```python
from unittest.mock import Mock, PropertyMock, patch
import unittest

class TemperatureSensor:
    @property
    def temperature(self):
        return self._read_hardware()

    def _read_hardware(self):
        return 22.5

class TestPropertyMock(unittest.TestCase):

    def test_property_mock_basic(self):
        sensor = Mock()
        type(sensor).temperature = PropertyMock(return_value=25.0)
        self.assertEqual(sensor.temperature, 25.0)

    def test_property_mock_side_effect(self):
        sensor = Mock()
        mock_prop = PropertyMock(side_effect=[20.0, 25.0, 30.0])
        type(sensor).temperature = mock_prop
        self.assertEqual(sensor.temperature, 20.0)
        self.assertEqual(sensor.temperature, 25.0)
        self.assertEqual(sensor.temperature, 30.0)
        self.assertEqual(mock_prop.call_count, 3)

    def test_property_setter(self):
        sensor = Mock()
        type(sensor).temperature = PropertyMock(return_value=18.0)
        sensor.temperature = 30.0
        self.assertEqual(sensor.temperature, 18.0)

    def test_property_setter_verification(self):
        sensor = Mock()
        mock_prop = PropertyMock()
        type(sensor).temperature = mock_prop
        sensor.temperature = 42
        mock_prop.__set__.assert_called_once_with(sensor, 42)

    @patch("__main__.TemperatureSensor._read_hardware", return_value=100.0)
    def test_patch_property_backing(self, mock_read):
        sensor = TemperatureSensor()
        self.assertEqual(sensor.temperature, 100.0)
```

### Example 7: Sentinel 

```python
from unittest.mock import Mock, sentinel
import unittest

class TestSentinel(unittest.TestCase):

    def test_sentinel_as_return_value(self):
        mock = Mock()
        mock.get.return_value = sentinel.DEFAULT_USER
        result = mock.get()
        self.assertIs(result, sentinel.DEFAULT_USER)

    def test_sentinel_uniqueness(self):
        self.assertIsNot(sentinel.A, sentinel.B)
        self.assertIsNot(sentinel.ONE, sentinel.TWO)
        self.assertIs(sentinel.A, sentinel.A)

    def test_sentinel_in_dict(self):
        cache = {}
        cache[sentinel.KEY] = "cached_value"
        self.assertEqual(cache[sentinel.KEY], "cached_value")

    def test_sentinel_object(self):
        obj = sentinel.OBJECT
        self.assertTrue(hasattr(obj, "__sentinel__"))

    def test_sentinel_default_values(self):
        def get_user(user_id, default=sentinel.NOT_FOUND):
            if user_id == 1:
                return "Alice"
            return default

        self.assertEqual(get_user(1), "Alice")
        self.assertIs(get_user(999), sentinel.NOT_FOUND)

    def test_sentinel_identity_check(self):
        mock = Mock()
        mock.process.return_value = sentinel.PROCESSED
        result = mock.process()
        self.assertTrue(result is sentinel.PROCESSED)

    def test_sentinel_enum_style_usage(self):
        class Status:
            PENDING = sentinel.PENDING
            ACTIVE = sentinel.ACTIVE
            COMPLETED = sentinel.COMPLETED

        self.assertIs(Status.PENDING, sentinel.PENDING)
        self.assertIsNot(Status.PENDING, Status.ACTIVE)
```

### Example 8: AsyncMock 

```python
from unittest.mock import AsyncMock, Mock
import unittest
import asyncio

class TestAsyncMock(unittest.TestCase):

    def test_async_mock_basic(self):
        mock = AsyncMock()
        mock.async_method.return_value = 42
        result = asyncio.run(mock.async_method())
        self.assertEqual(result, 42)

    def test_async_mock_await_count(self):
        mock = AsyncMock()
        coro = mock()
        asyncio.run(coro)
        self.assertEqual(mock.await_count, 1)

    def test_async_mock_side_effect(self):
        mock = AsyncMock()
        mock.side_effect = [1, 2, ValueError("error")]
        async def run():
            r1 = await mock()
            r2 = await mock()
            with self.assertRaises(ValueError):
                await mock()
            return r1, r2
        r1, r2 = asyncio.run(run())
        self.assertEqual(r1, 1)
        self.assertEqual(r2, 2)

    def test_async_mock_assert_awaited(self):
        mock = AsyncMock()
        asyncio.run(mock())
        mock.assert_awaited()

    def test_async_mock_assert_awaited_once(self):
        mock = AsyncMock()
        asyncio.run(mock())
        mock.assert_awaited_once()

    def test_async_mock_assert_awaited_with(self):
        mock = AsyncMock()
        asyncio.run(mock(1, key="value"))
        mock.assert_awaited_with(1, key="value")

    def test_async_mock_assert_any_await(self):
        mock = AsyncMock()
        asyncio.run(mock(1))
        asyncio.run(mock(2))
        mock.assert_any_await(2)

    def test_async_context_manager(self):
        mock = AsyncMock()
        mock.__aenter__.return_value = "entered"
        mock.__aexit__.return_value = False
        async def use_context():
            async with mock as m:
                return m
        result = asyncio.run(use_context())
        self.assertEqual(result, "entered")

    def test_async_iterator(self):
        mock = AsyncMock()
        mock.__aiter__.return_value = mock
        mock.__anext__.side_effect = [1, 2, 3, StopAsyncIteration()]
        async def iterate():
            return [item async for item in mock]
        result = asyncio.run(iterate())
        self.assertEqual(result, [1, 2, 3])
```

### Example 9: Advanced Mock Patterns

```python
from unittest.mock import Mock, patch, PropertyMock, ANY
import unittest

class TestAdvancedPatterns(unittest.TestCase):

    def test_nested_mock_attributes(self):
        mock = Mock()
        mock.db.users.find.return_value = [{"id": 1}]
        mock.db.users.count.return_value = 42
        self.assertEqual(mock.db.users.find(), [{"id": 1}])
        self.assertEqual(mock.db.users.count(), 42)

    def test_mock_with_any(self):
        mock = Mock()
        mock.process(1, "hello", {"key": "value"})
        mock.process.assert_called_with(ANY, ANY, {"key": "value"})

    def test_mock_reset(self):
        mock = Mock()
        mock("called")
        mock.reset_mock()
        self.assertEqual(mock.call_count, 0)
        mock.assert_not_called()

    def test_configure_mock(self):
        mock = Mock()
        mock.configure_mock(
            name="service",
            version=2.0,
            status="running",
        )
        self.assertEqual(mock.name, "service")
        self.assertEqual(mock.version, 2.0)
        self.assertEqual(mock.status, "running")

    def test_mock_wraps(self):
        class Original:
            def method(self):
                return "original"
        original = Original()
        mock = Mock(wraps=original)
        self.assertEqual(mock.method(), "original")
        mock.method.assert_called_once()

    def test_mock_wraps_with_override(self):
        class Original:
            def method(self):
                return "original"
            def other(self):
                return "other"
        original = Original()
        mock = Mock(wraps=original)
        mock.method.return_value = "mocked"
        self.assertEqual(mock.method(), "mocked")
        self.assertEqual(mock.other(), "other")

    def test_mock_name(self):
        mock = Mock(name="my_mock")
        self.assertEqual(mock._mock_name, "my_mock")

    def test_attach_mock(self):
        parent = Mock()
        child = Mock()
        parent.attach_mock(child, "child_method")
        parent.child_method(1, 2, 3)
        child.assert_called_with(1, 2, 3)

    def test_mock_filtering_call_args(self):
        mock = Mock()
        mock("debug", "message")
        mock("info", "other")
        debug_calls = [c for c in mock.call_args_list if c[0][0] == "debug"]
        self.assertEqual(len(debug_calls), 1)

    def test_mock_as_dict_key(self):
        mock = Mock()
        d = {mock: "value"}
        self.assertEqual(d[mock], "value")
```

### Example 10: Real-World Mocking Scenarios

```python
from unittest.mock import Mock, patch, MagicMock
import unittest
import json

class PaymentGateway:
    def charge(self, amount, token):
        response = self._http_post("/charges", {"amount": amount, "source": token})
        return response["id"]

    def _http_post(self, endpoint, data):
        import requests
        resp = requests.post(f"https://api.stripe.com{endpoint}", json=data)
        resp.raise_for_status()
        return resp.json()

class EmailService:
    def send_welcome(self, user_email, username):
        import smtplib
        with smtplib.SMTP("smtp.example.com") as server:
            server.login("user", "pass")
            message = f"Welcome {username}!"
            server.sendmail("noreply@example.com", [user_email], message)

class AnalyticsTracker:
    def track_event(self, event_name, properties=None):
        import http.client
        data = json.dumps({"event": event_name, "properties": properties or {}})
        conn = http.client.HTTPSConnection("api.example.com")
        conn.request("POST", "/track", data)
        return conn.getresponse().status

class TestRealWorldMocking(unittest.TestCase):

    @patch("payment.PaymentGateway._http_post")
    def test_payment_charge(self, mock_post):
        mock_post.return_value = {"id": "ch_12345"}
        gateway = PaymentGateway()
        charge_id = gateway.charge(1000, "tok_visa")
        self.assertEqual(charge_id, "ch_12345")
        mock_post.assert_called_once_with("/charges", {"amount": 1000, "source": "tok_visa"})

    @patch("smtplib.SMTP")
    def test_send_welcome_email(self, mock_smtp):
        mock_server = MagicMock()
        mock_smtp.return_value.__enter__.return_value = mock_server
        service = EmailService()
        service.send_welcome("alice@test.com", "Alice")
        mock_smtp.assert_called_once_with("smtp.example.com")
        mock_server.login.assert_called_once()
        mock_server.sendmail.assert_called_once()

    @patch("http.client.HTTPSConnection")
    def test_analytics_tracking(self, mock_conn):
        mock_response = Mock()
        mock_response.status = 200
        mock_conn.return_value.getresponse.return_value = mock_response
        tracker = AnalyticsTracker()
        status = tracker.track_event("user_signup", {"plan": "pro"})
        self.assertEqual(status, 200)

    @patch("builtins.open", new_callable=MagicMock)
    def test_file_reading(self, mock_file):
        mock_file.return_value.__enter__.return_value.read.return_value = '{"key": "value"}'
        with open("config.json") as f:
            data = json.load(f)
        self.assertEqual(data["key"], "value")

    @patch("requests.get")
    def test_api_call_with_error(self, mock_get):
        import requests
        mock_response = Mock()
        mock_response.raise_for_status.side_effect = requests.exceptions.HTTPError("404")
        mock_get.return_value = mock_response
        with self.assertRaises(requests.exceptions.HTTPError):
            response = requests.get("https://api.example.com/data")
            response.raise_for_status()
```

## Beginner Examples

```python
from unittest.mock import Mock, patch

# Simple mock example
def test_basic_mock():
    mock = Mock()
    mock.return_value = "hello"
    result = mock()
    assert result == "hello"

# Patching a function
def test_patch_simple():
    with patch("builtins.input", return_value="yes"):
        from getpass import getpass
        result = input("Continue? ")
        assert result == "yes"

# Mocking a class method
def test_class_mock():
    mock = Mock()
    mock.calculate.return_value = 42
    assert mock.calculate(1, 2) == 42
    mock.calculate.assert_called_once_with(1, 2)
```

## Intermediate Examples

```python
from unittest.mock import Mock, patch, call, PropertyMock
import unittest

class DatabaseService:
    def get_user(self, user_id):
        return {"id": user_id, "name": "Real User"}

class UserController:
    def __init__(self, db):
        self.db = db

    def get_user_name(self, user_id):
        user = self.db.get_user(user_id)
        return user["name"].upper()

class TestUserController(unittest.TestCase):
    def setUp(self):
        self.mock_db = Mock(spec=DatabaseService)
        self.controller = UserController(self.mock_db)

    def test_get_user_name_returns_uppercase(self):
        self.mock_db.get_user.return_value = {"id": 1, "name": "Alice"}
        result = self.controller.get_user_name(1)
        self.assertEqual(result, "ALICE")
        self.mock_db.get_user.assert_called_once_with(1)
```

## Advanced Examples

```python
from unittest.mock import Mock, AsyncMock, PropertyMock, patch, call, ANY, sentinel
import unittest
import asyncio

class CacheService:
    def __init__(self):
        self._store = {}

    @property
    def size(self):
        return len(self._store)

    async def get(self, key):
        await asyncio.sleep(0.01)
        return self._store.get(key, sentinel.MISSING)

    async def set(self, key, value):
        await asyncio.sleep(0.01)
        self._store[key] = value

class TestAdvancedCacheMocking(unittest.TestCase):

    def test_property_and_async_mock_combined(self):
        cache = Mock(spec=CacheService)
        type(cache).size = PropertyMock(return_value=3)
        cache.get = AsyncMock(return_value="cached_value")

        self.assertEqual(cache.size, 3)
        result = asyncio.run(cache.get("key"))
        self.assertEqual(result, "cached_value")

    def test_mock_with_sentinel_for_missing_keys(self):
        cache = Mock(spec=CacheService)
        cache.get.return_value = sentinel.MISSING
        result = cache.get("nonexistent")
        self.assertIs(result, sentinel.MISSING)

    def test_complex_assertion_with_any(self):
        logger = Mock()
        logger.log("INFO", "User 123 created", extra={"user_id": 123})
        logger.log.assert_called_with("INFO", ANY, extra={"user_id": 123})

    def test_sequential_calls_with_varying_returns(self):
        fetcher = Mock()
        fetcher.fetch.side_effect = [
            {"status": "pending"},
            {"status": "processing"},
            {"status": "completed"},
        ]
        statuses = []
        for _ in range(3):
            result = fetcher.fetch()
            statuses.append(result["status"])
        self.assertEqual(statuses, ["pending", "processing", "completed"])

    @patch("some_module.api_call")
    @patch("some_module.auth_token")
    def test_nested_patches(self, mock_auth, mock_api):
        mock_auth.return_value = "token123"
        mock_api.return_value = {"success": True}
        self.assertEqual(mock_auth(), "token123")
        self.assertEqual(mock_api(), {"success": True})
```

## Real-World Use Cases

- **Database Abstraction** – Mock ORM calls to test business logic without a real database.
- **External API Clients** – Simulate HTTP responses, timeouts, and error codes.
- **File System Operations** – Test file reading/writing without touching the disk.
- **Email and Notification Services** – Verify that emails would be sent without sending them.
- **Payment Gateways** – Test charge, refund, and subscription flows safely.
- **Caching Layers** – Test cache hit/miss behavior without Redis/Memcached.
- **Authentication Services** – Mock OAuth providers, token validation, and session management.
- **Message Queues** – Simulate produce/consume patterns with RabbitMQ, Kafka, etc.

## Common Mistakes

1. **Mocking the wrong thing** – Patching the local reference instead of where it's used.
2. **Over-mocking** – Mocking simple operations that don't need isolation.
3. **Not using `spec`** – leading to tests passing with non-existent methods.
4. **Forgetting to `assert_called_once`** – missing verification that mocks were used correctly.
5. **Mutable return values** – reusing mock return values that get mutated between tests.
6. **Not cleaning up patches** – patches applied without context manager leaking.
7. **Mocking implementation details** – making tests brittle to refactoring.

## Best Practices

1. **Patch where the object is used**, not where it is defined.
2. **Use `spec` or `spec_set`** to prevent typos and catch interface changes.
3. **Use `create_autospec`** for automatic spec inference from real objects.
4. **Verify mock interactions** with `assert_called_once_with` where appropriate.
5. **Reset mocks in setUp** or use `autospec` to avoid state leakage.
6. **Prefer context managers** over decorators when only patching part of a test.
7. **Use `ANY` for arguments** you don't care about in assertions.
8. **Avoid deep nesting** of mock attributes; refactor code if possible.
9. **Keep tests readable** – use descriptive mock names.

## Interview Questions

1. **What is the difference between Mock and MagicMock?** – MagicMock has default implementations for magic methods (`__len__`, `__iter__`, etc.); Mock requires explicit configuration.
2. **How does `patch()` determine what to replace?** – It uses the string path to replace the attribute in the module namespace for the test scope.
3. **What is `side_effect` used for?** – To return different values on successive calls, call a function, or raise an exception.
4. **Explain the difference between `return_value` and `side_effect`** – `return_value` is a fixed return; `side_effect` can be an iterable, callable, or exception for dynamic behavior.
5. **What is `spec` and why use it?** – `spec` restricts mock attributes to those of a real class, catching typos and ensuring interface compliance.
6. **How does `PropertyMock` work?** – It is a mock for properties; assigned via `type(obj).property_name = PropertyMock(...)`.
7. **What is `AsyncMock`?** – A mock for async functions; supports `await_count`, `assert_awaited`, etc.
8. **What problem does `sentinel` solve?** – Provides unique objects for sentinel/default values, avoiding the risk of real data collision.

## Coding Challenges

1. Write a mock-based test for a function that reads a CSV file, processes rows, and writes results to another file. Mock both file operations.
2. Create a test suite for a `RetryHandler` class that retries failed API calls. Mock the underlying HTTP client to fail twice then succeed.
3. Implement tests for a `CircuitBreaker` pattern that tracks failure counts and opens/closes the circuit. Use PropertyMock for state.
4. Write parameterized tests for an `EventEmitter` that uses async callbacks. Use AsyncMock to verify handler invocation.
5. Create a comprehensive mock test for a `DataPipeline` that reads from S3, transforms via Spark-like operations, and writes to Redshift.

## Summary

`unittest.mock` is a powerful, flexible library for creating test doubles in Python. The `Mock` and `MagicMock` classes provide full control over return values, side effects, and attribute access. `patch()` enables temporary replacement of objects in modules, while `PropertyMock` handles property mocking, `AsyncMock` handles async code, and `sentinel` provides unique sentinel objects. Proper mocking isolates units under test, making tests fast, reliable, and deterministic.

## Related Topics

- [unittest](./77_unittest.md) – Using mocks within unittest TestCase classes.
- [pytest](./78_pytest.md) – Using mocks with pytest fixtures and monkeypatch.
- [TDD](./80_tdd.md) – Mocking as a key enabler for test-driven development.
- [Integration Testing](./81_integration_testing.md) – When not to mock: testing real integrations.
- [vcrpy](https://vcrpy.readthedocs.io/) – Record and replay HTTP interactions instead of mocking.
