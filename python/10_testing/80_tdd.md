# TDD - Test-Driven Development, Red-Green-Refactor cycle

## Introduction

Test-Driven Development (TDD) is a software development methodology where tests are written before the production code. The cycle follows three phases: **Red** (write a failing test), **Green** (write the simplest code to make it pass), and **Refactor** (improve code quality while keeping tests green). This approach ensures that every line of production code is covered by tests and drives the design of the system from the outside in.

## Why It Is Important

- **Design Feedback** – Tests force you to think about API design before implementation.
- **Regression Prevention** – Every feature's test suite catches regressions immediately.
- **Documentation** – Tests serve as executable documentation of expected behavior.
- **Confidence** – Refactoring becomes safe with comprehensive test coverage.
- **Reduced Debugging** – Bugs are caught within minutes, not weeks.
- **Modular Design** – TDD naturally leads to loose coupling and high cohesion.
- **Sustainable Pace** – Defect rates drop, reducing firefighting and overtime.

## Syntax

```python
# RED: Write a failing test first
def test_addition():
    calc = Calculator()
    result = calc.add(2, 3)
    assert result == 5  # Calculator doesn't exist yet -> FAIL

# GREEN: Write minimal code to pass
class Calculator:
    def add(self, a, b):
        return a + b  # Just enough to pass

# REFACTOR: Improve without changing behavior
class Calculator:
    """A simple calculator with basic operations."""
    def add(self, a, b):
        return a + b
```

## Examples

### Example 1: Full TDD Cycle – FizzBuzz

```python
# Step 1: RED - Write failing tests
import pytest

def test_fizzbuzz_returns_1_for_1():
    assert fizzbuzz(1) == "1"

def test_fizzbuzz_returns_2_for_2():
    assert fizzbuzz(2) == "2"

def test_fizzbuzz_returns_fizz_for_3():
    assert fizzbuzz(3) == "Fizz"

def test_fizzbuzz_returns_buzz_for_5():
    assert fizzbuzz(5) == "Buzz"

def test_fizzbuzz_returns_fizz_for_6():
    assert fizzbuzz(6) == "Fizz"

def test_fizzbuzz_returns_buzz_for_10():
    assert fizzbuzz(10) == "Buzz"

def test_fizzbuzz_returns_fizzbuzz_for_15():
    assert fizzbuzz(15) == "FizzBuzz"


# Step 2: GREEN - Minimal implementation
def fizzbuzz(n):
    if n % 15 == 0:
        return "FizzBuzz"
    if n % 3 == 0:
        return "Fizz"
    if n % 5 == 0:
        return "Buzz"
    return str(n)


# Step 3: REFACTOR - Clean up
def fizzbuzz_refactored(n):
    result = ""
    if n % 3 == 0:
        result += "Fizz"
    if n % 5 == 0:
        result += "Buzz"
    return result or str(n)


# Additional edge case tests
def test_fizzbuzz_negative():
    assert fizzbuzz(-3) == "Fizz"
    assert fizzbuzz(-5) == "Buzz"
    assert fizzbuzz(-15) == "FizzBuzz"

def test_fizzbuzz_zero():
    assert fizzbuzz(0) == "FizzBuzz"

def test_fizzbuzz_large_numbers():
    assert fizzbuzz(100) == "Buzz"
    assert fizzbuzz(99) == "Fizz"
    assert fizzbuzz(90) == "FizzBuzz"
```

### Example 2: TDD for a Shopping Cart

```python
import pytest
from decimal import Decimal

# --- RED: Write tests first ---

def test_empty_cart_total_is_zero():
    cart = ShoppingCart()
    assert cart.total == Decimal("0")

def test_add_item_increases_total():
    cart = ShoppingCart()
    cart.add_item("Apple", Decimal("0.50"))
    assert cart.total == Decimal("0.50")

def test_add_multiple_items():
    cart = ShoppingCart()
    cart.add_item("Apple", Decimal("0.50"), 3)
    assert cart.total == Decimal("1.50")

def test_add_different_items():
    cart = ShoppingCart()
    cart.add_item("Apple", Decimal("0.50"))
    cart.add_item("Banana", Decimal("0.75"))
    assert cart.total == Decimal("1.25")

def test_remove_item_decreases_total():
    cart = ShoppingCart()
    cart.add_item("Apple", Decimal("0.50"), 2)
    cart.remove_item("Apple", 1)
    assert cart.total == Decimal("0.50")

def test_remove_nonexistent_item_raises():
    cart = ShoppingCart()
    with pytest.raises(KeyError):
        cart.remove_item("Nonexistent", 1)

def test_clear_cart():
    cart = ShoppingCart()
    cart.add_item("Apple", Decimal("0.50"))
    cart.add_item("Banana", Decimal("0.75"))
    cart.clear()
    assert cart.total == Decimal("0")
    assert cart.item_count == 0

def test_item_count():
    cart = ShoppingCart()
    cart.add_item("Apple", Decimal("0.50"), 2)
    cart.add_item("Banana", Decimal("0.75"), 3)
    assert cart.item_count == 5


# --- GREEN: Implement ShoppingCart ---

class ShoppingCart:
    def __init__(self):
        self._items = {}

    def add_item(self, name, price, quantity=1):
        if name in self._items:
            existing = self._items[name]
            self._items[name] = (existing[0], existing[1] + quantity)
        else:
            self._items[name] = (price, quantity)

    def remove_item(self, name, quantity=1):
        if name not in self._items:
            raise KeyError(f"Item '{name}' not in cart")
        price, qty = self._items[name]
        if quantity >= qty:
            del self._items[name]
        else:
            self._items[name] = (price, qty - quantity)

    def clear(self):
        self._items.clear()

    @property
    def total(self):
        return sum(price * qty for price, qty in self._items.values())

    @property
    def item_count(self):
        return sum(qty for _, qty in self._items.values())


# --- REFACTOR: Could add Decimal rounding, coupons, etc. ---
# Now test a coupon feature
def test_apply_percentage_discount():
    cart = ShoppingCart()
    cart.add_item("Item", Decimal("100.00"))
    cart.apply_discount(10)  # 10% off
    assert cart.total == Decimal("90.00")

# Extend ShoppingCart (GREEN step)
# Keep tests short; exercise left for reader
```

### Example 3: TDD for a Password Validator

```python
import pytest
import re

# --- RED: Define expected behaviors ---

def test_password_must_be_at_least_8_chars():
    assert not is_valid_password("Ab1!")  # too short
    assert is_valid_password("Ab1!defg")  # just long enough

def test_password_must_contain_uppercase():
    assert not is_valid_password("abc1!defg")
    assert is_valid_password("Abc1!defg")

def test_password_must_contain_lowercase():
    assert not is_valid_password("ABC1!DEFG")
    assert is_valid_password("ABCd!EFG")

def test_password_must_contain_digit():
    assert not is_valid_password("Abc!defgh")
    assert is_valid_password("Abc1defgh")

def test_password_must_contain_special_char():
    assert not is_valid_password("Abc1defgh")
    assert is_valid_password("Abc1!defgh")

def test_password_rejects_common_patterns():
    assert not is_valid_password("Password1!")
    assert not is_valid_password("Abcdefg1!")

def test_password_minimum_requirements():
    valid = "Str0ng!Pass"
    assert is_valid_password(valid)

def test_password_too_long():
    assert not is_valid_password("A" * 129)

def test_password_whitespace():
    assert not is_valid_password("Pass word1!")
    assert is_valid_password("NoSpaces1!")

def test_password_rejects_null():
    assert not is_valid_password("")


# --- GREEN: Minimal implementation ---

def is_valid_password(password):
    if len(password) == 0:
        return False
    if len(password) < 8 or len(password) > 128:
        return False
    if " " in password:
        return False
    if not re.search(r"[A-Z]", password):
        return False
    if not re.search(r"[a-z]", password):
        return False
    if not re.search(r"\d", password):
        return False
    if not re.search(r"[!@#$%^&*()_\-+=\[\]{}|;:'\",.<>?/~`]", password):
        return False
    common = ["password", "123456", "qwerty", "abcdef"]
    if any(pattern in password.lower() for pattern in common):
        return False
    return True


# --- REFACTOR: Extract validation rules ---
class PasswordValidator:
    def __init__(self):
        self.rules = [
            self._check_length,
            self._check_uppercase,
            self._check_lowercase,
            self._check_digit,
            self._check_special_char,
            self._check_whitespace,
            self._check_common_patterns,
        ]

    def validate(self, password):
        errors = []
        for rule in self.rules:
            error = rule(password)
            if error:
                errors.append(error)
        return errors

    def is_valid(self, password):
        return len(self.validate(password)) == 0

    def _check_length(self, pwd):
        if len(pwd) < 8:
            return "Too short"
        if len(pwd) > 128:
            return "Too long"

    def _check_uppercase(self, pwd):
        if not re.search(r"[A-Z]", pwd):
            return "Missing uppercase"

    def _check_lowercase(self, pwd):
        if not re.search(r"[a-z]", pwd):
            return "Missing lowercase"

    def _check_digit(self, pwd):
        if not re.search(r"\d", pwd):
            return "Missing digit"

    def _check_special_char(self, pwd):
        if not re.search(r"[!@#$%^&*()_\-+=\[\]{}|;:'\",.<>?/~`]", pwd):
            return "Missing special char"

    def _check_whitespace(self, pwd):
        if " " in pwd:
            return "Contains whitespace"

    def _check_common_patterns(self, pwd):
        common = ["password", "123456", "qwerty", "abcdef"]
        if any(p in pwd.lower() for p in common):
            return "Contains common pattern"

# Test the refactored version
def test_refactored_validator():
    validator = PasswordValidator()
    assert validator.is_valid("MyStr0ng!Pass")
    assert not validator.is_valid("weak")
    assert len(validator.validate("")) > 0
```

### Example 4: TDD for an API Client

```python
import pytest
from unittest.mock import Mock, patch
import json

# --- RED: Write test for API client behavior ---

def test_api_client_gets_data():
    client = APIClient("https://api.example.com")
    data = client.get_user(1)
    assert "id" in data
    assert "name" in data

def test_api_client_handles_404():
    client = APIClient("https://api.example.com")
    with pytest.raises(UserNotFoundError):
        client.get_user(999)

def test_api_client_sets_auth_header():
    client = APIClient("https://api.example.com", token="abc123")
    assert client._headers["Authorization"] == "Bearer abc123"

def test_api_client_sends_correct_url():
    client = APIClient("https://api.example.com")
    with patch("requests.get") as mock_get:
        mock_get.return_value.json.return_value = {"id": 1}
        mock_get.return_value.raise_for_status = Mock()
        client.get_user(1)
        mock_get.assert_called_with(
            "https://api.example.com/users/1",
            headers={},
            timeout=10,
        )

def test_api_client_default_timeout():
    client = APIClient("https://api.example.com")
    assert client.timeout == 10
    client2 = APIClient("https://api.example.com", timeout=30)
    assert client2.timeout == 30

def test_api_client_get_all_users():
    client = APIClient("https://api.example.com")
    with patch.object(client, "_request") as mock_req:
        mock_req.return_value = [{"id": 1}, {"id": 2}]
        users = client.get_all_users()
        assert len(users) == 2
        mock_req.assert_called_with("GET", "users")

def test_api_client_create_user():
    client = APIClient("https://api.example.com")
    with patch.object(client, "_request") as mock_req:
        mock_req.return_value = {"id": 3, "name": "New User"}
        result = client.create_user({"name": "New User"})
        assert result["id"] == 3
        mock_req.assert_called_with("POST", "users", json={"name": "New User"})

def test_api_client_delete_user():
    client = APIClient("https://api.example.com")
    with patch.object(client, "_request") as mock_req:
        mock_req.return_value = {"success": True}
        result = client.delete_user(5)
        assert result["success"] is True
        mock_req.assert_called_with("DELETE", "users/5")


# --- GREEN: Minimal API Client ---

class UserNotFoundError(Exception):
    pass

class APIClient:
    def __init__(self, base_url, token=None, timeout=10):
        self.base_url = base_url.rstrip("/")
        self.timeout = timeout
        self._headers = {}
        if token:
            self._headers["Authorization"] = f"Bearer {token}"

    def get_user(self, user_id):
        import requests
        url = f"{self.base_url}/users/{user_id}"
        response = requests.get(url, headers=self._headers, timeout=self.timeout)
        if response.status_code == 404:
            raise UserNotFoundError(f"User {user_id} not found")
        response.raise_for_status()
        return response.json()

    def get_all_users(self):
        return self._request("GET", "users")

    def create_user(self, data):
        return self._request("POST", "users", json=data)

    def delete_user(self, user_id):
        return self._request("DELETE", f"users/{user_id}")

    def _request(self, method, path, **kwargs):
        import requests
        url = f"{self.base_url}/{path.lstrip('/')}"
        response = requests.request(
            method, url, headers=self._headers, timeout=self.timeout, **kwargs
        )
        response.raise_for_status()
        return response.json()
```

### Example 5: TDD for a Todo List Manager

```python
import pytest
from datetime import datetime

# --- RED ---

def test_create_todo():
    manager = TodoManager()
    todo = manager.create("Buy milk")
    assert todo["id"] == 1
    assert todo["title"] == "Buy milk"
    assert todo["completed"] is False
    assert "created_at" in todo

def test_create_todo_increments_id():
    manager = TodoManager()
    t1 = manager.create("First")
    t2 = manager.create("Second")
    assert t1["id"] == 1
    assert t2["id"] == 2

def test_get_todo_by_id():
    manager = TodoManager()
    manager.create("Task")
    todo = manager.get(1)
    assert todo["title"] == "Task"

def test_get_nonexistent_todo_returns_none():
    manager = TodoManager()
    assert manager.get(999) is None

def test_list_all_todos():
    manager = TodoManager()
    manager.create("A")
    manager.create("B")
    manager.create("C")
    assert len(manager.list()) == 3

def test_update_todo_title():
    manager = TodoManager()
    manager.create("Old title")
    updated = manager.update(1, title="New title")
    assert updated["title"] == "New title"

def test_update_todo_completed():
    manager = TodoManager()
    manager.create("Task")
    updated = manager.update(1, completed=True)
    assert updated["completed"] is True

def test_update_nonexistent_raises():
    manager = TodoManager()
    with pytest.raises(KeyError):
        manager.update(999, title="X")

def test_delete_todo():
    manager = TodoManager()
    manager.create("To delete")
    assert manager.delete(1) is True
    assert manager.get(1) is None

def test_delete_nonexistent_returns_false():
    manager = TodoManager()
    assert manager.delete(999) is False

def test_list_filter_by_completed():
    manager = TodoManager()
    manager.create("A")
    manager.create("B")
    manager.update(1, completed=True)
    active = manager.list(completed=False)
    done = manager.list(completed=True)
    assert len(active) == 1
    assert len(done) == 1

def test_todo_defaults_to_not_completed():
    manager = TodoManager()
    todo = manager.create("Test")
    assert todo["completed"] is False


# --- GREEN ---

class TodoManager:
    def __init__(self):
        self._todos = {}
        self._next_id = 1

    def create(self, title):
        todo = {
            "id": self._next_id,
            "title": title,
            "completed": False,
            "created_at": datetime.now().isoformat(),
        }
        self._todos[self._next_id] = todo
        self._next_id += 1
        return todo

    def get(self, todo_id):
        return self._todos.get(todo_id)

    def list(self, completed=None):
        todos = list(self._todos.values())
        if completed is not None:
            todos = [t for t in todos if t["completed"] == completed]
        return todos

    def update(self, todo_id, **kwargs):
        if todo_id not in self._todos:
            raise KeyError(f"Todo {todo_id} not found")
        self._todos[todo_id].update(kwargs)
        return self._todos[todo_id]

    def delete(self, todo_id):
        if todo_id in self._todos:
            del self._todos[todo_id]
            return True
        return False


# --- REFACTOR ---
class TodoManagerRefactored:
    def __init__(self):
        self._todos = {}
        self._next_id = 1

    def create(self, title):
        todo = Todo(self._next_id, title)
        self._todos[self._next_id] = todo
        self._next_id += 1
        return todo

    def get(self, todo_id):
        return self._todos.get(todo_id)

    def list(self, completed=None):
        if completed is None:
            return list(self._todos.values())
        return [t for t in self._todos.values() if t.completed == completed]

    def update(self, todo_id, **kwargs):
        todo = self._todos.get(todo_id)
        if not todo:
            raise KeyError(f"Todo {todo_id} not found")
        for key, value in kwargs.items():
            setattr(todo, key, value)
        return todo

    def delete(self, todo_id):
        return self._todos.pop(todo_id, None) is not None

class Todo:
    def __init__(self, todo_id, title):
        self.id = todo_id
        self.title = title
        self.completed = False
        self.created_at = datetime.now().isoformat()

    def to_dict(self):
        return {
            "id": self.id,
            "title": self.title,
            "completed": self.completed,
            "created_at": self.created_at,
        }
```

### Example 6: TDD Baby Steps – String Calculator

```python
import pytest

# RED: Write one test at a time
def test_empty_string_returns_zero():
    assert add("") == 0

def test_single_number():
    assert add("1") == 1
    assert add("5") == 5

def test_two_numbers():
    assert add("1,2") == 3

def test_multiple_numbers():
    assert add("1,2,3,4,5") == 15

def test_newline_delimiter():
    assert add("1\n2,3") == 6

def test_custom_delimiter():
    assert add("//;\n1;2") == 3

def test_negative_numbers_raise():
    with pytest.raises(ValueError) as exc:
        add("1,-2,3")
    assert "negatives not allowed: -2" in str(exc.value)

def test_multiple_negatives_in_message():
    with pytest.raises(ValueError) as exc:
        add("-1,-2,3")
    assert "-1, -2" in str(exc.value)

def test_numbers_over_1000_ignored():
    assert add("2,1001") == 2

def test_multi_char_delimiter():
    assert add("//[***]\n1***2***3") == 6

def test_multiple_delimiters():
    assert add("//[*][%]\n1*2%3") == 6


# GREEN: Incremental implementation
def add(numbers):
    if not numbers:
        return 0

    delimiter = ","
    if numbers.startswith("//"):
        parts = numbers.split("\n", 1)
        delimiter_section = parts[0][2:]
        numbers = parts[1]
        if delimiter_section.startswith("["):
            import re
            delimiters = re.findall(r"\[([^\]]+)\]", delimiter_section)
            delimiter = "|".join(map(re.escape, delimiters))
        else:
            delimiter = re.escape(delimiter_section)

    numbers = numbers.replace("\n", ",")
    import re
    nums = [int(x) for x in re.split(delimiter, numbers) if x]

    negatives = [n for n in nums if n < 0]
    if negatives:
        raise ValueError(f"negatives not allowed: {', '.join(map(str, negatives))}")

    return sum(n for n in nums if n <= 1000)


# REFACTOR: Extract parser
class StringCalculator:
    def add(self, numbers):
        if not numbers:
            return 0
        nums = self._parse(numbers)
        self._check_negatives(nums)
        return sum(n for n in nums if n <= 1000)

    def _parse(self, numbers):
        import re
        delimiter = ","
        if numbers.startswith("//"):
            parts = numbers.split("\n", 1)
            delimiter_section = parts[0][2:]
            numbers = parts[1]
            if delimiter_section.startswith("["):
                delimiters = re.findall(r"\[([^\]]+)\]", delimiter_section)
                delimiter = "|".join(map(re.escape, delimiters))
            else:
                delimiter = re.escape(delimiter_section)
        numbers = numbers.replace("\n", ",")
        return [int(x) for x in re.split(delimiter, numbers) if x]

    def _check_negatives(self, nums):
        negatives = [n for n in nums if n < 0]
        if negatives:
            raise ValueError(f"negatives not allowed: {', '.join(map(str, negatives))}")
```

## Beginner Examples

```python
# The simplest TDD cycle
import pytest

# 1. RED
def test_hello_returns_greeting():
    assert greet("World") == "Hello, World!"

# 2. GREEN
def greet(name):
    return f"Hello, {name}!"

# 3. REFACTOR
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

# Test the refactored version
def test_greet_with_custom_greeting():
    assert greet("Alice", "Hi") == "Hi, Alice!"
```

## Intermediate Examples

```python
import pytest
from datetime import datetime, timedelta

# RED: Tests for a subscription manager
def test_subscription_starts_as_trial():
    sub = Subscription("user@test.com")
    assert sub.status == "trial"

def test_trial_expires_after_30_days():
    sub = Subscription("user@test.com", created_at=datetime.now() - timedelta(days=31))
    assert sub.status == "expired"

def test_activate_subscription():
    sub = Subscription("user@test.com")
    sub.activate("premium_monthly")
    assert sub.status == "active"
    assert sub.plan == "premium_monthly"

def test_cancel_subscription():
    sub = Subscription("user@test.com")
    sub.activate("premium_monthly")
    sub.cancel()
    assert sub.status == "cancelled"

def test_cancelled_subscription_cannot_be_used():
    sub = Subscription("user@test.com")
    sub.cancel()
    assert sub.is_active() is False

def test_expired_subscription_sends_notification():
    notifications = []
    sub = Subscription("user@test.com",
                       on_expire=lambda s: notifications.append(s.email))
    sub.created_at = datetime.now() - timedelta(days=31)
    sub.check_status()
    assert len(notifications) == 1
    assert notifications[0] == "user@test.com"

# GREEN
class Subscription:
    def __init__(self, email, created_at=None, on_expire=None):
        self.email = email
        self.created_at = created_at or datetime.now()
        self.plan = None
        self._status = "trial"
        self.on_expire = on_expire

    @property
    def status(self):
        if self._status == "trial" and self._is_expired():
            return "expired"
        return self._status

    def activate(self, plan):
        self._status = "active"
        self.plan = plan

    def cancel(self):
        self._status = "cancelled"

    def is_active(self):
        return self.status == "active"

    def check_status(self):
        if self._is_expired() and self.on_expire:
            self.on_expire(self)

    def _is_expired(self):
        return datetime.now() - self.created_at > timedelta(days=30)
```

## Advanced Examples

```python
import pytest
from unittest.mock import Mock, patch
from datetime import datetime
import json

# RED: Tests for an Order Processing System
def test_create_order():
    processor = OrderProcessor()
    order = processor.create_order(user_id=1, items=[{"product_id": 1, "qty": 2}])
    assert order["status"] == "pending"
    assert order["user_id"] == 1

def test_create_order_calculates_total():
    processor = OrderProcessor()
    with patch.object(processor, "_get_product_price") as mock_price:
        mock_price.side_effect = lambda pid: {1: 10.0, 2: 5.0}[pid]
        order = processor.create_order(1, [
            {"product_id": 1, "qty": 2},
            {"product_id": 2, "qty": 3},
        ])
    assert order["total"] == 35.0

def test_process_payment_success():
    processor = OrderProcessor()
    processor.payment_gateway = Mock()
    processor.payment_gateway.charge.return_value = {"id": "ch_123", "status": "succeeded"}
    order = processor.create_order(1, [{"product_id": 1, "qty": 1}])
    result = processor.process_payment(order["id"], "tok_visa")
    assert result["status"] == "succeeded"

def test_process_payment_failure():
    processor = OrderProcessor()
    processor.payment_gateway = Mock()
    processor.payment_gateway.charge.side_effect = PaymentError("Card declined")
    order = processor.create_order(1, [{"product_id": 1, "qty": 1}])
    result = processor.process_payment(order["id"], "tok_bad")
    assert result["status"] == "failed"
    assert "Card declined" in result["error"]

def test_fulfill_order():
    processor = OrderProcessor()
    processor.inventory = Mock()
    processor.shipping = Mock()
    order = processor.create_order(1, [{"product_id": 1, "qty": 1}])
    result = processor.fulfill(order["id"])
    assert result["status"] == "shipped"
    processor.inventory.reserve.assert_called()
    processor.shipping.create_label.assert_called()

def test_fulfill_out_of_stock():
    processor = OrderProcessor()
    processor.inventory = Mock()
    processor.inventory.reserve.side_effect = InsufficientStockError("Not enough")
    order = processor.create_order(1, [{"product_id": 1, "qty": 99}])
    result = processor.fulfill(order["id"])
    assert result["status"] == "failed"
    assert "Not enough" in result["error"]

# GREEN
class PaymentError(Exception):
    pass

class InsufficientStockError(Exception):
    pass

class OrderProcessor:
    def __init__(self):
        self.orders = {}
        self.next_id = 1
        self.payment_gateway = None
        self.inventory = None
        self.shipping = None

    def create_order(self, user_id, items):
        total = sum(
            self._get_product_price(item["product_id"]) * item["qty"]
            for item in items
        )
        order = {
            "id": self.next_id,
            "user_id": user_id,
            "items": items,
            "total": total,
            "status": "pending",
            "created_at": datetime.now().isoformat(),
        }
        self.orders[self.next_id] = order
        self.next_id += 1
        return order

    def _get_product_price(self, product_id):
        prices = {1: 10.0, 2: 5.0, 3: 20.0}
        return prices.get(product_id, 0.0)

    def process_payment(self, order_id, token):
        order = self.orders[order_id]
        try:
            result = self.payment_gateway.charge(order["total"], token)
            order["status"] = "paid"
            return result
        except PaymentError as e:
            order["status"] = "payment_failed"
            return {"status": "failed", "error": str(e)}

    def fulfill(self, order_id):
        order = self.orders[order_id]
        try:
            for item in order["items"]:
                self.inventory.reserve(item["product_id"], item["qty"])
            label = self.shipping.create_label(order)
            order["status"] = "shipped"
            order["tracking"] = label["tracking_number"]
            return {"status": "shipped", "tracking": label["tracking_number"]}
        except InsufficientStockError as e:
            order["status"] = "fulfillment_failed"
            return {"status": "failed", "error": str(e)}
```

## Real-World Use Cases

- **E-commerce Checkout** – Validate cart, payment, inventory, and shipping logic.
- **Authentication Systems** – Test login, registration, password reset, MFA flows.
- **Data Pipelines** – Validate ETL transformations, aggregation, and reporting.
- **REST APIs** – Test endpoint behavior, validation, error handling, and pagination.
- **Financial Systems** – Ensure precision for tax calculation, discounts, currency conversion.
- **CLI Tools** – Test argument parsing, output formatting, and exit codes.
- **Game Logic** – Test scoring, collision detection, and state machines.

## Common Mistakes

1. **Writing too-large tests** – Tests should verify one behavior, not an entire feature.
2. **Skipping the Red phase** – Passing tests without seeing them fail first.
3. **Writing too much code at once** – The Green phase should be minimal.
4. **Skipping Refactor** – Accumulating technical debt despite passing tests.
5. **Testing implementation** – Tests should verify behavior, not internal structure.
6. **Slow tests** – Tests must run fast to maintain the TDD cadence.
7. **Not testing edge cases** – Empty inputs, boundaries, error conditions.
8. **Ignoring test maintenance** – Tests are code too; refactor them alongside production code.

## Best Practices

1. **Follow the Red-Green-Refactor cycle** strictly until it becomes habit.
2. **Write the simplest code to pass** – no more, no less.
3. **Run tests frequently** – every few seconds during development.
4. **Keep tests independent** – no shared mutable state between tests.
5. **Use meaningful test names** – `test_withdraw_reduces_balance` not `test_withdraw`.
6. **Write tests at the right level** – unit test logic, integration test boundaries.
7. **Aim for high coverage** but focus on critical paths, not vanity metrics.
8. **Refactor with confidence** – green tests mean you haven't broken anything.
9. **Pair or mob when learning** TDD to reinforce the discipline.

## Interview Questions

1. **What are the three phases of TDD?** – Red (write failing test), Green (make it pass), Refactor (improve code).
2. **What does "Red-Green-Refactor" mean?** – Write a test that fails (red), write minimal code to pass (green), then clean up (refactor).
3. **Why write tests before code?** – It forces API design upfront, ensures testability, and guarantees test coverage for new features.
4. **How is TDD different from traditional testing?** – In TDD, tests drive the design; in traditional testing, tests are written after implementation to verify correctness.
5. **What is the "baby steps" approach?** – Making tiny incremental changes (seconds between Red and Green) to maintain constant progress.
6. **How do you handle external dependencies in TDD?** – Use mocking/stubbing to isolate the unit under test.
7. **What test coverage is ideal?** – 80-100% for critical business logic; less for boilerplate/UI. Focus on risk, not numbers.
8. **Can TDD be applied to legacy code?** – Yes, but start by writing characterization tests that capture current behavior before refactoring.

## Coding Challenges

1. Implement a `BankAccount` class using TDD: deposit, withdraw, transfer, balance inquiry, overdraft protection.
2. Build a `RomanNumeralConverter` using TDD: convert between integers and Roman numerals (1-3999).
3. Create a `WordCounter` that counts word frequency in text, handles punctuation, case-insensitivity, and stop words.
4. Develop a `RateLimiter` using TDD: allow N requests per window, reject excess, reset after window expires.
5. Implement a `JSONSchemaValidator` using TDD: validate types, required fields, nested objects, and enum constraints.

## Summary

Test-Driven Development is a disciplined practice where tests are written before production code in a continuous Red-Green-Refactor cycle. It improves code quality, reduces bugs, enables safe refactoring, and produces comprehensive test suites. While it requires initial discipline, TDD pays dividends through reduced debugging time, better design, and higher confidence in code changes. It works best when combined with mocking, small test granularity, and rapid feedback loops.

## Related Topics

- [unittest](./77_unittest.md) – Python's built-in framework for writing TDD tests.
- [pytest](./78_pytest.md) – A more expressive framework commonly used with TDD.
- [Mocking](./79_mocking.md) – Essential for isolating units during TDD.
- [Integration Testing](./81_integration_testing.md) – Complementary to TDD's unit-level focus.
- [Coverage.py](https://coverage.readthedocs.io/) – Measuring test coverage.
- [Mutation Testing](https://mutpy.readthedocs.io/) – Evaluating test quality beyond coverage.
