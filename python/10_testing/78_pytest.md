# Pytest - fixtures, parametrize, conftest.py, marks, plugins
## Introduction
pytest has become the de facto standard testing framework for Python. It offers simple syntax, powerful fixtures, automatic test discovery, extensive plugin architecture, and detailed reporting. Unlike unittest, pytest uses plain assert statements instead of self.assertEqual(), making tests more readable and Pythonic. Its rich ecosystem includes plugins for coverage, parallel execution, Django testing, and more. pytest's fixture system provides modular, scalable, and reusable test dependencies.

## Fixtures
### What It Is
Fixtures are functions decorated with `@pytest.fixture` that provide a fixed baseline for tests. They handle setup, teardown, and resource provisioning with automatic cleanup through yield statements.

### Why It Is Important
Fixtures replace unittest's setUp/tearDown patterns with a more flexible, composable system. They support scoping (function, class, module, session), automatic cleanup, and dependency injection, reducing boilerplate and improving test maintainability.

### How It Works Internally
pytest creates a fixture dependency graph and resolves fixtures through dependency injection. When a test function declares a parameter matching a fixture name, pytest calls the fixture function and passes the result. Fixtures can request other fixtures, forming a directed acyclic graph.

```python
import pytest

@pytest.fixture
def sample_data():
    return {"key": "value", "numbers": [1, 2, 3]}

def test_sample_data(sample_data):
    assert sample_data["key"] == "value"
    assert len(sample_data["numbers"]) == 3
```

### Fixture Scopes
```python
import pytest

@pytest.fixture(scope="function")  # Default: run for each test
def function_fixture():
    return "per-function"

@pytest.fixture(scope="class")
def class_fixture():
    return "per-class"

@pytest.fixture(scope="module")
def module_fixture():
    return "per-module"

@pytest.fixture(scope="session")
def session_fixture():
    return "per-session"
```

### Fixture Teardown with yield
```python
import pytest

@pytest.fixture
def database():
    db = Database(":memory:")
    db.create_tables()
    yield db  # Test runs here
    db.close()  # Teardown runs after test

def test_insert(database):
    database.insert("user", {"name": "Alice"})
    assert database.count("user") == 1
```

### Autouse Fixtures
```python
import pytest

@pytest.fixture(autouse=True)
def setup_test_environment():
    print("\nSetting up environment...")
    yield
    print("\nTearing down environment...")

def test_one():
    assert True

def test_two():
    assert True
```

### Built-in Fixtures
```python
import pytest

def test_tmp_path(tmp_path):
    d = tmp_path / "subdir"
    d.mkdir()
    p = d / "hello.txt"
    p.write_text("content")
    assert p.read_text() == "content"

def test_capsys(capsys):
    print("hello")
    captured = capsys.readouterr()
    assert captured.out == "hello\n"

def test_monkeypatch(monkeypatch):
    def mock_get():
        return {"status": "ok"}
    monkeypatch.setattr("module.api.get_data", mock_get)
```

### Advanced Fixture Usage
```python
import pytest

@pytest.fixture
def user_data(request):
    """Parametrized fixture using request.param"""
    return request.param

@pytest.mark.parametrize("user_data", [
    {"name": "Alice", "age": 30},
    {"name": "Bob", "age": 25},
], indirect=True)
def test_user_validation(user_data):
    assert "name" in user_data
    assert "age" in user_data
    assert user_data["age"] > 0

@pytest.fixture
def factory():
    """Factory as fixture pattern"""
    def create_user(name, age):
        return {"name": name, "age": age, "id": hash(name)}
    return create_user

def test_factory_pattern(factory):
    user = factory("Alice", 30)
    assert user["name"] == "Alice"
```

## parametrize decorator
### What It Is
`@pytest.mark.parametrize` allows running a test function multiple times with different argument values. It generates a separate test case for each combination of parameters.

### Why It Is Important
Parametrization eliminates duplicated test code, improves coverage by testing edge cases systematically, and makes it easy to add new test cases without writing new test functions.

### Syntax
```python
import pytest

@pytest.mark.parametrize("a, b, expected", [
    (1, 1, 2),
    (2, 3, 5),
    (-1, 1, 0),
    (0, 0, 0),
])
def test_add(a, b, expected):
    assert a + b == expected
```

### Multiple Parameters
```python
import pytest

@pytest.mark.parametrize("x", [0, 1])
@pytest.mark.parametrize("y", [0, 1])
def test_cartesian_product(x, y):
    """Runs 4 times: (0,0), (0,1), (1,0), (1,1)"""
    assert x in (0, 1) and y in (0, 1)
```

### IDs for Readable Test Names
```python
import pytest

@pytest.mark.parametrize("input,expected", [
    ("hello", "HELLO"),
    ("world", "WORLD"),
    ("pytest", "PYTEST"),
], ids=["hello_case", "world_case", "pytest_case"])
def test_upper(input, expected):
    assert input.upper() == expected
```

### Stacked Parametrize with Fixtures
```python
import pytest

@pytest.fixture
def multiplier():
    return 2

@pytest.mark.parametrize("value,expected", [
    (1, 2),
    (3, 6),
    (5, 10),
])
def test_multiply(multiplier, value, expected):
    assert value * multiplier == expected
```

## conftest.py
### What It Is
`conftest.py` is a special configuration file that pytest loads automatically. It can contain fixtures, hooks, and plugins that are shared across tests in the same directory and subdirectories.

### Why It Is Important
conftest.py enables sharing fixtures across multiple test modules without importing them. It provides a hierarchical fixture system where fixtures in higher-level conftest.py files are available to lower-level test files.

### How It Works Internally
pytest discovers conftest.py files during test collection. Fixtures defined in a conftest.py are available to all tests in that directory and its subdirectories. Fixtures closer to the test (lower in the directory tree) take precedence.

```python
# tests/conftest.py
import pytest

@pytest.fixture
def db_connection():
    conn = create_database(":memory:")
    yield conn
    conn.close()

@pytest.fixture
def api_client():
    return APIClient(base_url="http://testserver")

# tests/test_users.py
def test_create_user(db_connection, api_client):
    response = api_client.post("/users", {"name": "Alice"})
    assert response.status_code == 201
```

### conftest.py Hooks
```python
# conftest.py
import pytest

def pytest_configure(config):
    """Register custom markers"""
    config.addinivalue_line("markers", "slow: mark test as slow")
    config.addinivalue_line("markers", "integration: mark as integration test")

def pytest_addoption(parser):
    """Add custom command line options"""
    parser.addoption("--env", action="store", default="test",
                     help="Environment: test, staging, or production")

@pytest.fixture
def env(request):
    return request.config.getoption("--env")

def pytest_runtest_setup(item):
    """Skip tests based on markers"""
    if "slow" in item.keywords and not item.config.getoption("--runslow"):
        pytest.skip("need --runslow option to run")
```

### Directory Structure
```
tests/
├── conftest.py           # Top-level fixtures
├── unit/
│   ├── conftest.py       # Unit test fixtures
│   └── test_models.py
└── integration/
    ├── conftest.py       # Integration test fixtures
    └── test_api.py
```

## Marks
### What It Is
Marks are metadata tags applied to test functions using `@pytest.mark.*` decorators. They enable selective test execution, categorization, and behavior modification.

### Why It Is Important
Marks provide a flexible way to organize tests, skip tests conditionally, expect failures, and run specific subsets of tests without modifying test code.

### Built-in Marks
```python
import pytest

@pytest.mark.skip(reason="Not implemented yet")
def test_not_ready():
    assert False

@pytest.mark.skipif(sys.version_info < (3, 8), reason="requires Python 3.8+")
def test_python_version():
    assert True

@pytest.mark.xfail(reason="known bug", strict=True)
def test_known_failure():
    assert 1 == 0  # Expected to fail

@pytest.mark.timeout(5)
def test_timeout():
    import time
    time.sleep(1)

@pytest.mark.slow
def test_slow_operation():
    import time
    time.sleep(10)
```

### Custom Marks
```python
# conftest.py
def pytest_configure(config):
    config.addinivalue_line("markers", "smoke: quick smoke tests")
    config.addinivalue_line("markers", "regression: regression tests")

# test_suite.py
import pytest

@pytest.mark.smoke
def test_login():
    assert login("admin", "pass")

@pytest.mark.regression
def test_old_feature():
    assert old_feature_works()

# Running: pytest -m smoke
# Running: pytest -m "not slow"
# Running: pytest -m "smoke or regression"
```

### Filtering with Marks
```bash
# Run only smoke tests
pytest -m smoke

# Run all except slow tests
pytest -m "not slow"

# Run smoke AND integration tests
pytest -m "smoke and integration"

# Run either smoke or regression tests
pytest -m "smoke or regression"
```

## Plugins
### What It Is
pytest's plugin architecture allows extending functionality through installed packages or local conftest.py files. The ecosystem includes hundreds of plugins for coverage, parallelism, Django, Flask, and more.

### Popular Plugins
```python
# pytest-cov: Code coverage
# Install: pip install pytest-cov
# Usage: pytest --cov=myproject --cov-report=html

# pytest-xdist: Parallel test execution
# Install: pip install pytest-xdist
# Usage: pytest -n auto

# pytest-django: Django testing
# Install: pip install pytest-django
# Usage: pytest --ds=myproject.settings

# pytest-flask: Flask testing
# Install: pip install pytest-flask

# pytest-benchmark: Performance benchmarking
# Install: pip install pytest-benchmark
```

### Writing a Simple Plugin
```python
# conftest.py
import pytest

class MyPlugin:
    def pytest_sessionstart(self, session):
        print("Test session started!")

    def pytest_sessionfinish(self, session, exitstatus):
        print("Test session finished!")

    @pytest.hookimpl(tryfirst=True)
    def pytest_runtest_protocol(self, item, nextitem):
        print(f"Running test: {item.name}")

def pytest_configure(config):
    config.pluginmanager.register(MyPlugin())
```

### Built-in Plugins
```python
# Cache plugin: stores test data across runs
@pytest.mark.repeat(3)  # Requires pytest-repeat
def test_flaky():
    import random
    assert random.random() > 0.5
```

### Advanced Configuration
```python
# pytest.ini
[pytest]
minversion = 6.0
testpaths = tests
python_files = test_*.py test_*_suite.py
python_classes = Test* *Test
python_functions = test_*
markers =
    slow: marks tests as slow (deselect with '-m "not slow"')
    smoke: smoke tests for quick verification
filterwarnings =
    ignore::DeprecationWarning
addopts = -v --tb=short
```

### Beginner Examples
```python
import pytest

def is_even(n):
    return n % 2 == 0

class TestMath:
    def test_is_even(self):
        assert is_even(2)
        assert not is_even(3)

    @pytest.mark.parametrize("n", [2, 4, 6, 8, 10])
    def test_multiple_even(self, n):
        assert is_even(n)
```

### Intermediate Examples
```python
import pytest
import requests

class TestAPI:
    @pytest.fixture(autouse=True)
    def setup(self, api_client):
        self.client = api_client

    @pytest.mark.parametrize("endpoint,expected_status", [
        ("/health", 200),
        ("/users", 200),
        ("/nonexistent", 404),
    ])
    def test_endpoints(self, endpoint, expected_status):
        response = self.client.get(endpoint)
        assert response.status_code == expected_status

    @pytest.mark.slow
    def test_report_generation(self):
        response = self.client.post("/reports", {"type": "summary"})
        assert response.status_code == 201
```

### Advanced Examples
```python
import pytest
from unittest.mock import patch, MagicMock

class TestServiceLayer:
    @pytest.fixture
    def mock_redis(self):
        with patch("app.cache.Redis") as mock:
            instance = mock.return_value
            instance.get.return_value = None
            yield instance

    @pytest.mark.parametrize("user_id,expected", [
        (1, {"id": 1, "name": "Alice"}),
        (2, {"id": 2, "name": "Bob"}),
    ], ids=["user_alice", "user_bob"])
    def test_get_user(self, user_id, expected, mock_redis, db_session):
        user = UserService.get_user(user_id)
        assert user.name == expected["name"]
        mock_redis.get.assert_called_with(f"user:{user_id}")

    @pytest.mark.skipif(not HAS_GPU, reason="GPU not available")
    def test_gpu_processing(self):
        result = process_on_gpu(large_dataset)
        assert result.accuracy > 0.95
```

### Real-World Use Cases
- Testing web applications (Django, Flask, FastAPI)
- Data pipeline validation
- API integration testing
- Database migration testing
- Performance regression testing
- Microservice contract testing

### Common Mistakes
- Using module-scoped fixtures when function-scoped is sufficient
- Forgetting `yield` for teardown in fixtures
- Over-parametrization leading to too many test cases
- Not registering custom marks in pytest.ini
- Sharing mutable objects across tests via fixtures

### Best Practices
- Use fixtures for dependencies, not setup functions
- Keep tests independent and isolated
- Name tests descriptively
- Use conftest.py for shared fixtures
- Register custom markers in pytest.ini
- Use `--strict-markers` flag

### Performance Considerations
- Use `scope="session"` for expensive fixtures
- Use `pytest-xdist` for parallel execution
- Use `--durations=10` to identify slow tests
- Avoid fixture recomputation with proper scoping

### Interview Questions
1. How do pytest fixtures differ from unittest setUp/tearDown?
2. Explain fixture scopes and when to use each.
3. What is conftest.py and how does it work?
4. How do you share fixtures across multiple test files?
5. Explain parametrization strategies.

### Coding Challenges
1. Convert a unittest TestCase to pytest fixtures and parametrize
2. Write a conftest.py with session-scoped database connection
3. Create a custom pytest plugin that measures test execution time

### Related Topics
- unittest module
- mock and patch
- doctest
- tox for test environments
- CI/CD integration
- pytest-cov coverage
