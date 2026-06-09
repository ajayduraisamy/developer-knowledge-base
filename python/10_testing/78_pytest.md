# Pytest - fixtures, parametrize, conftest.py, marks, plugins

## Introduction

pytest is a mature, full-featured Python testing framework that makes it easy to write simple and scalable tests. It supports automatic test discovery, powerful fixtures, parameterized testing, and an extensive plugin ecosystem. Unlike unittest, pytest uses plain `assert` statements (no need for `self.assert*` methods) through assert rewriting, and tests can be written as simple functions rather than classes.

## Why It Is Important

- **Minimal Boilerplate** – Tests are plain functions with plain `assert` statements.
- **Powerful Fixtures** – Modular, scoped, and reusable test dependencies.
- **Rich Plugin Ecosystem** – Hundreds of plugins for coverage, parallelism, Django, Flask, etc.
- **Automatic Discovery** – Finds tests by naming conventions (`test_*.py`, `*test.py`).
- **Detailed Reporting** – Clear, colored output with assert introspection.
- **Parametrize** – Built-in support for running a test with multiple arguments.
- **conftest.py** – Share fixtures and hooks across test files.

## Syntax

```python
# test_example.py
def test_addition():
    assert 1 + 1 == 2

def test_with_fixture(tmp_path):
    d = tmp_path / "subdir"
    d.mkdir()
    assert d.is_dir()
```

## Examples

### Example 1: Basic Test Discovery and Assert Rewriting

```python
# test_basic.py
def test_equality():
    assert 2 + 2 == 4

def test_string():
    assert "hello world".split() == ["hello", "world"]

def test_collection():
    assert 3 in [1, 2, 3]

def test_nested():
    result = {"a": {"b": [1, 2, 3]}}
    assert result["a"]["b"][1] == 2

def test_exception():
    with pytest.raises(ZeroDivisionError):
        1 / 0

import pytest
```

### Example 2: Fixtures with Different Scopes

```python
import pytest

@pytest.fixture(scope="function")
def function_fixture():
    print("\n  setup function")
    yield {"data": "fresh"}
    print("  teardown function")

@pytest.fixture(scope="class")
def class_fixture():
    print("\n  setup class")
    yield {"shared": "data"}
    print("  teardown class")

@pytest.fixture(scope="module")
def module_fixture():
    print("\n  setup module")
    yield {"module": "scope"}
    print("  teardown module")

@pytest.fixture(scope="session")
def session_fixture():
    print("\n  setup session")
    yield {"session": "wide"}
    print("  teardown session")

class TestFixtures:
    def test_one(self, function_fixture, class_fixture, module_fixture, session_fixture):
        assert function_fixture["data"] == "fresh"

    def test_two(self, function_fixture, class_fixture, module_fixture, session_fixture):
        assert class_fixture["shared"] == "data"
```

### Example 3: Autouse Fixtures

```python
import pytest

@pytest.fixture(autouse=True)
def ensure_environment():
    """Automatically runs for every test in the session."""
    import os
    original = os.environ.get("TEST_MODE")
    os.environ["TEST_MODE"] = "active"
    yield
    if original is None:
        del os.environ["TEST_MODE"]
    else:
        os.environ["TEST_MODE"] = original

@pytest.fixture(autouse=True)
def mock_logger():
    """Auto-mock logging for all tests."""
    from unittest.mock import patch
    with patch("logging.getLogger") as mock:
        yield mock

class TestAutouse:
    def test_env_is_set(self):
        import os
        assert os.environ.get("TEST_MODE") == "active"

    def test_another(self):
        import os
        assert os.environ.get("TEST_MODE") == "active"
```

### Example 4: conftest.py Shared Fixtures

```python
# conftest.py
import pytest
import tempfile
import os
from pathlib import Path

@pytest.fixture
def temp_dir():
    with tempfile.TemporaryDirectory() as tmp:
        yield Path(tmp)

@pytest.fixture
def sample_csv(temp_dir):
    csv_path = temp_dir / "data.csv"
    csv_path.write_text("name,age\nAlice,30\nBob,25\n")
    return csv_path

@pytest.fixture
def db_connection():
    class FakeConnection:
        def execute(self, query):
            return [{"id": 1, "name": "test"}]
    return FakeConnection()

# test_users.py
import pytest
from pathlib import Path

def test_csv_loading(sample_csv):
    import csv
    with open(sample_csv) as f:
        reader = list(csv.DictReader(f))
    assert len(reader) == 2
    assert reader[0]["name"] == "Alice"

def test_db_query(db_connection):
    result = db_connection.execute("SELECT * FROM users")
    assert len(result) == 1
    assert result[0]["name"] == "test"
```

### Example 5: Parametrize Decorator

```python
import pytest

@pytest.mark.parametrize("input_str,expected", [
    ("hello", "HELLO"),
    ("world", "WORLD"),
    ("pytest", "PYTEST"),
    ("123", "123"),
    ("", ""),
    ("already UPPER", "ALREADY UPPER"),
])
def test_upper(input_str, expected):
    assert input_str.upper() == expected

@pytest.mark.parametrize("a,b,expected", [
    (1, 2, 3),
    (0, 0, 0),
    (-1, 1, 0),
    (100, -50, 50),
])
def test_add(a, b, expected):
    assert a + b == expected

@pytest.mark.parametrize("value", [0, 1, "", "text", [], [1, 2], {}, {"a": 1}])
def test_bool_conversion(value):
    assert bool(value) == (value not in (0, "", [], {}, None))

@pytest.mark.parametrize("x", [1, 2])
@pytest.mark.parametrize("y", [10, 20])
def test_cartesian_product(x, y):
    assert x * y > 0
```

### Example 6: Custom Marks and Built-in Marks

```python
import pytest
import time

@pytest.mark.slow
def test_slow_operation():
    time.sleep(0.5)
    assert True

@pytest.mark.smoke
def test_smoke_check():
    assert True

@pytest.mark.smoke
@pytest.mark.slow
def test_smoke_and_slow():
    assert True

@pytest.mark.skip(reason="Not implemented yet")
def test_not_ready():
    assert False

@pytest.mark.skipif(1 == 1, reason="Always skipped")
def test_conditional_skip():
    assert False

@pytest.mark.xfail(reason="Known bug #123")
def test_expected_failure():
    assert 1 == 2

@pytest.mark.xfail(strict=True, reason="Must fail")
def test_strict_xfail():
    assert 1 == 2

@pytest.mark.timeout(1)
def test_timeout():
    import time
    time.sleep(0.5)
    assert True

@pytest.mark.filterwarnings("ignore::DeprecationWarning")
def test_ignore_warnings():
    import warnings
    warnings.warn("deprecated", DeprecationWarning)
    assert True
```

### Example 7: monkeypatch and tmp_path

```python
import pytest
import os
import json

class ConfigManager:
    def __init__(self, path):
        self.path = path

    def load(self):
        with open(self.path) as f:
            return json.load(f)

    def get_setting(self, key, default=None):
        data = self.load()
        return data.get(key, default)

class TestMonkeypatch:

    def test_env_variable(self, monkeypatch):
        monkeypatch.setenv("DATABASE_URL", "sqlite:///test.db")
        assert os.environ["DATABASE_URL"] == "sqlite:///test.db"

    def test_remove_env_variable(self, monkeypatch):
        monkeypatch.delenv("HOME", raising=False)
        assert "HOME" not in os.environ

    def test_monkeypath_dict(self, monkeypatch):
        config = {"key": "value"}
        monkeypatch.setitem(config, "new_key", "new_value")
        assert config["new_key"] == "new_value"

    def test_monkeypatch_function(self, monkeypatch):
        def mock_getpass(prompt="Password: "):
            return "mocked_password"
        import getpass
        monkeypatch.setattr(getpass, "getpass", mock_getpass)
        assert getpass.getpass() == "mocked_password"

class TestTmpPath:

    def test_tmp_path_creation(self, tmp_path):
        d = tmp_path / "subdir"
        d.mkdir()
        assert d.is_dir()
        assert d.parent == tmp_path

    def test_write_and_read(self, tmp_path):
        file = tmp_path / "test.txt"
        file.write_text("hello pytest")
        assert file.read_text() == "hello pytest"

    def test_tmp_path_unique(self, tmp_path):
        import uuid
        file = tmp_path / f"{uuid.uuid4()}.tmp"
        file.touch()
        assert file.exists()

    def test_config_loading(self, tmp_path, monkeypatch):
        config_file = tmp_path / "config.json"
        config_file.write_text(json.dumps({"host": "localhost", "port": 8080}))
        manager = ConfigManager(str(config_file))
        assert manager.get_setting("host") == "localhost"
        assert manager.get_setting("port") == 8080
        assert manager.get_setting("missing", "default") == "default"

    def test_config_file_not_found(self, tmp_path):
        manager = ConfigManager(str(tmp_path / "nonexistent.json"))
        import json
        with pytest.raises(FileNotFoundError):
            manager.load()
```

### Example 8: pytest Plugins (pytest-cov, pytest-xdist)

```python
# Run with: pytest --cov=my_module --cov-report=term-missing
# or: pytest -n auto

import pytest

class TestCoverage:
    def test_basic(self):
        assert True

    @pytest.mark.parametrize("i", range(10))
    def test_param(self, i):
        assert i >= 0

# conftest.py for pytest-xdist
# Add this to conftest.py to ensure worker isolation:
# def pytest_configure(config):
#     config.option.dist = "load"
#     config.option.tx = ["popen"]

# pytest.ini or pyproject.toml configuration:
# [pytest]
# addopts = -v --tb=short --strict-markers
# testpaths = tests
# markers =
#     slow: marks tests as slow
#     smoke: marks tests as smoke tests
```

### Example 9: Testing with Temporary Databases

```python
import pytest
import sqlite3

@pytest.fixture
def in_memory_db():
    conn = sqlite3.connect(":memory:")
    conn.execute("CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT)")
    conn.execute("INSERT INTO users VALUES (1, 'Alice')")
    conn.execute("INSERT INTO users VALUES (2, 'Bob')")
    conn.commit()
    yield conn
    conn.close()

@pytest.fixture
def db_cursor(in_memory_db):
    return in_memory_db.cursor()

class TestDatabase:
    def test_fetch_all_users(self, db_cursor):
        db_cursor.execute("SELECT * FROM users")
        users = db_cursor.fetchall()
        assert len(users) == 2

    def test_fetch_by_id(self, db_cursor):
        db_cursor.execute("SELECT name FROM users WHERE id = ?", (1,))
        name = db_cursor.fetchone()[0]
        assert name == "Alice"

    def test_insert_user(self, in_memory_db):
        in_memory_db.execute("INSERT INTO users VALUES (3, 'Charlie')")
        cursor = in_memory_db.cursor()
        cursor.execute("SELECT COUNT(*) FROM users")
        assert cursor.fetchone()[0] == 3
```

### Example 10: Fixtures with Request and Parametrization

```python
import pytest

@pytest.fixture
def user_data(request):
    """Fixture that accepts parametrized data via request.param."""
    return request.param

@pytest.mark.parametrize("user_data", [
    {"name": "Alice", "age": 30, "active": True},
    {"name": "Bob", "age": 25, "active": False},
    {"name": "Charlie", "age": 35, "active": True},
], indirect=True)
def test_user_activation(user_data):
    if user_data["age"] >= 18:
        assert user_data["active"] or not user_data["active"]
    else:
        assert not user_data.get("active", False)

@pytest.fixture(params=[1, 2, 3, 4, 5])
def number(request):
    return request.param

def test_numbers(number):
    assert 1 <= number <= 5

@pytest.fixture(params=["json", "xml", "yaml"])
def serializer(request):
    if request.param == "json":
        import json
        return json
    elif request.param == "xml":
        import xml.etree.ElementTree as ET
        return ET
    else:
        import yaml
        return yaml

def test_serialization(serializer):
    assert hasattr(serializer, "dumps") or hasattr(serializer, "dump")
```

## Beginner Examples

```python
import pytest

def test_addition():
    result = 2 + 3
    assert result == 5

def test_string_contains():
    text = "Hello, pytest!"
    assert "pytest" in text

def test_list_length():
    items = [1, 2, 3]
    assert len(items) == 3

def test_dictionary_key():
    data = {"name": "Alice", "age": 30}
    assert "name" in data
    assert data["age"] == 30

def test_type_check():
    value = 42
    assert isinstance(value, int)

def test_float_precision():
    result = 0.1 + 0.2
    assert abs(result - 0.3) < 1e-6

def test_truthy_values():
    assert True
    assert 1
    assert "non-empty"
    assert [1, 2, 3]
    assert {"key": "value"}
```

## Intermediate Examples

```python
import pytest
from pathlib import Path

@pytest.fixture
def temp_project(tmp_path):
    src = tmp_path / "src"
    src.mkdir()
    (src / "__init__.py").touch()
    (src / "module.py").write_text("VERSION = '1.0.0'")
    return tmp_path

class TestProjectStructure:
    def test_init_exists(self, temp_project):
        init_file = temp_project / "src" / "__init__.py"
        assert init_file.exists()

    def test_module_version(self, temp_project):
        import sys
        sys.path.insert(0, str(temp_project / "src"))
        try:
            import module
            assert module.VERSION == "1.0.0"
        finally:
            sys.path.remove(str(temp_project / "src"))

    @pytest.mark.parametrize("filename,expected", [
        ("src/__init__.py", True),
        ("src/module.py", True),
        ("README.md", False),
    ])
    def test_file_existence(self, temp_project, filename, expected):
        assert (temp_project / filename).exists() == expected

@pytest.mark.parametrize("url,expected_status", [
    ("/api/users", 200),
    ("/api/users/1", 200),
    ("/api/users/999", 404),
    ("/api/invalid", 404),
])
def test_api_routes(url, expected_status):
    assert True  # Placeholder for actual API test
```

## Advanced Examples

```python
import pytest
import asyncio
from unittest.mock import AsyncMock

@pytest.fixture
def async_mock_fixture():
    mock = AsyncMock()
    mock.fetch_data.return_value = {"key": "value"}
    return mock

@pytest.mark.asyncio
async def test_async_function(async_mock_fixture):
    result = await async_mock_fixture.fetch_data()
    assert result == {"key": "value"}

@pytest.mark.asyncio
async def test_async_with_side_effect():
    mock = AsyncMock()
    mock.process.side_effect = [1, 2, ValueError("error")]
    assert await mock.process() == 1
    assert await mock.process() == 2
    with pytest.raises(ValueError):
        await mock.process()

class TestAsyncContext:
    @pytest.mark.asyncio
    async def test_async_context_manager(self):
        mock = AsyncMock()
        mock.__aenter__.return_value = "entered"
        async with mock as value:
            assert value == "entered"

    @pytest.mark.asyncio
    async def test_async_iterator(self):
        mock = AsyncMock()
        mock.__aiter__.return_value = mock
        mock.__anext__.side_effect = [1, 2, 3, StopAsyncIteration()]
        results = [item async for item in mock]
        assert results == [1, 2, 3]

@pytest.fixture(scope="session")
def event_loop():
    """Override the default event loop to be session-scoped."""
    loop = asyncio.new_event_loop()
    yield loop
    loop.close()

@pytest.mark.parametrize("input_data,expected", [
    ({"numbers": [1, 2, 3]}, 6),
    ({"numbers": [10, -5, 3]}, 8),
    ({"numbers": []}, 0),
])
@pytest.mark.asyncio
async def test_parametrized_async(input_data, expected):
    async def async_sum(data):
        await asyncio.sleep(0.01)
        return sum(data.get("numbers", []))
    result = await async_sum(input_data)
    assert result == expected

class TestFailuresAndDebugging:
    def test_assert_introspection(self):
        expected = {"name": "Alice", "age": 30, "city": "NYC"}
        actual = {"name": "Alice", "age": 31, "city": "NYC"}
        assert actual == expected

    def test_sequence_comparison(self):
        assert [1, 2, 3, 4] == [1, 2, 3, 5]

    def test_nested_comparison(self):
        assert {"a": [1, {"b": 2}]} == {"a": [1, {"b": 3}]}

@pytest.fixture
def heavy_resource():
    class Heavy:
        def __init__(self):
            self.data = list(range(10000))
        def compute(self):
            return sum(self.data)
    return Heavy()

@pytest.mark.benchmark
def test_heavy_computation(benchmark, heavy_resource):
    result = benchmark(heavy_resource.compute)
    assert result == 49995000

def test_fixture_finalization():
    @pytest.fixture
    def resource_with_cleanup():
        resource = {"allocated": True}
        yield resource
        resource.clear()

    data = {"allocated": True}
    assert data["allocated"]
    data.clear()
    assert data == {}
```

## Real-World Use Cases

- **Web Framework Tests** – Django/Flask/FastAPI view and model tests.
- **Data Science Validation** – Pandas DataFrame transform correctness.
- **API Contract Tests** – Verify REST API responses with parametrized inputs.
- **Configuration Tests** – Ensure YAML/TOML/JSON parsers handle edge cases.
- **CLI Application Tests** – Use `CliRunner` (Click) or `monkeypatch` for stdin/stdout.
- **Database Migration Tests** – Verify schema evolution with temp databases.
- **CI Pipeline Quality Gates** – Fail builds on coverage drops or test failures.

## Common Mistakes

1. **Forgetting `__init__.py`** in test directories.
2. **Naming files incorrectly** – must be `test_*.py` or `*_test.py`.
3. **Using `setup`/`teardown` methods** instead of yield fixtures.
4. **Fixture scope misuse** – session-scoped fixtures modifying state.
5. **Over-parametrization** – too many combinations causing slow test suites.
6. **Relying on test order** – tests should be independent.
7. **Sharing mutable fixture objects** across tests without yield cleanup.

## Best Practices

1. **Keep tests simple** – use functions over classes unless grouping is needed.
2. **Use `conftest.py`** for shared fixtures at appropriate directory levels.
3. **Name fixtures clearly** – `db_connection`, not `conn`.
4. **Prefer `tmp_path`** over `tempfile` for temporary files.
5. **Use marks to categorize** tests (`smoke`, `slow`, `integration`).
6. **Run with `-v`** for verbose output during development.
7. **Use `--tb=short`** for concise tracebacks in CI.
8. **Configure `pytest.ini`** or `pyproject.toml` for project defaults.
9. **Use `pytest-cov`** to track coverage and enforce thresholds.

## Interview Questions

1. **How does pytest discover tests?** – Recursively finds files matching `test_*.py` or `*_test.py`, then functions/methods starting with `test_`.
2. **What is fixture scope and what options exist?** – `function` (default), `class`, `module`, `package`, `session`. Controls fixture lifetime.
3. **Explain yield fixtures vs return fixtures** – Yield fixtures run cleanup code after `yield`; return fixtures only provide setup.
4. **How is `monkeypatch` different from `unittest.mock.patch`?** – monkeypatch modifies objects in-place for the test scope; it is simpler for basic patching.
5. **What are conftest.py files for?** – Share fixtures, hooks, and plugins across multiple test files in a directory tree.
6. **Explain parametrize and its use with fixtures** – `@pytest.mark.parametrize` runs a test multiple times with different data; can be combined with `indirect=True` for fixture parametrization.
7. **How do you run tests in parallel?** – Install `pytest-xdist` and run `pytest -n auto`.

## Coding Challenges

1. Write a parametrized test for a `validate_email(email: str) -> bool` function covering 20+ cases (valid, invalid, edge cases).
2. Create a conftest.py with fixtures for a Redis-backed cache service (using fakeredis or mocking).
3. Implement a custom mark `@pytest.mark.timeout(seconds)` using hooks and test it.
4. Write fixtures that set up and tear down a PostgreSQL test database (using pytest-postgresql plugin).
5. Create a test suite for an async web scraper using `httpx` and `pytest-asyncio`.

## Summary

pytest is a powerful, flexible testing framework that simplifies Python testing with minimal boilerplate. Its fixture system, automatic discovery, parametrization, and plugin ecosystem make it suitable for projects of any size. It replaces the rigid structure of unittest with a more Pythonic, function-based approach while maintaining backward compatibility through `unittest.TestCase` support.

## Related Topics

- [unittest](./77_unittest.md) – Python's built-in testing framework, compatible with pytest.
- [Mocking](./79_mocking.md) – Using unittest.mock with pytest.
- [TDD](./80_tdd.md) – Test-Driven Development workflow with pytest.
- [Integration Testing](./81_integration_testing.md) – Combining pytest with external services.
- [tox](https://tox.wiki/) – Virtualenv management for multi-environment testing.
- [nox](https://nox.thea.codes/) – Flexible test runner alternative to tox.
