# Integration Testing - Testing databases, APIs, and external services
## Introduction
Integration testing verifies that different modules, services, or systems work together correctly. Unlike unit tests that isolate individual components, integration tests validate the interactions between components: database queries, API calls, message queues, file systems, and third-party services. Integration testing is critical for catching issues that unit tests cannot, such as schema mismatches, network protocol errors, authentication failures, and data format inconsistencies. While integration tests are slower and more complex than unit tests, they provide higher confidence that the system works in production-like conditions.

## Database testing
### What It Is
Database integration testing validates that code correctly interacts with databases: executing queries, handling transactions, managing connections, and processing results. Tests typically use a test database (often in-memory or containerized) to verify data access logic.

### Why It Is Important
Database interactions are a common source of bugs: incorrect SQL, schema mismatches, transaction issues, connection leaks, and data integrity problems. Testing with a real database catches issues that mocking cannot.

### How It Works Internally
Database tests typically:
1. Set up a clean database schema
2. Seed test data
3. Execute the code under test
4. Assert database state or query results
5. Clean up test data

```python
import pytest
import sqlite3

# Test with SQLite in-memory database
@pytest.fixture
def db_connection():
    conn = sqlite3.connect(":memory:")
    conn.execute("""
        CREATE TABLE users (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            email TEXT UNIQUE NOT NULL
        )
    """)
    conn.execute("""
        CREATE TABLE orders (
            id INTEGER PRIMARY KEY,
            user_id INTEGER REFERENCES users(id),
            total REAL NOT NULL
        )
    """)
    yield conn
    conn.close()

@pytest.fixture
def sample_data(db_connection):
    db_connection.execute(
        "INSERT INTO users (name, email) VALUES (?, ?)",
        ("Alice", "alice@example.com")
    )
    db_connection.execute(
        "INSERT INTO users (name, email) VALUES (?, ?)",
        ("Bob", "bob@example.com")
    )
    db_connection.commit()
    return db_connection

class UserRepository:
    def __init__(self, conn):
        self.conn = conn

    def find_by_email(self, email):
        cursor = self.conn.execute(
            "SELECT id, name, email FROM users WHERE email = ?",
            (email,)
        )
        row = cursor.fetchone()
        if row:
            return {"id": row[0], "name": row[1], "email": row[2]}
        return None

    def create_user(self, name, email):
        cursor = self.conn.execute(
            "INSERT INTO users (name, email) VALUES (?, ?)",
            (name, email)
        )
        self.conn.commit()
        return cursor.lastrowid

def test_find_user_by_email(sample_data):
    repo = UserRepository(sample_data)
    user = repo.find_by_email("alice@example.com")
    assert user is not None
    assert user["name"] == "Alice"

def test_create_duplicate_email_raises_error(sample_data):
    repo = UserRepository(sample_data)
    with pytest.raises(sqlite3.IntegrityError):
        repo.create_user("Charlie", "alice@example.com")

def test_transaction_rollback_on_error(db_connection):
    repo = UserRepository(db_connection)
    try:
        repo.create_user("Test", "test@example.com")
        repo.create_user("Test", "test@example.com")  # Duplicate
    except sqlite3.IntegrityError:
        db_connection.rollback()

    user = repo.find_by_email("test@example.com")
    assert user is None  # Transaction rolled back
```

### PostgreSQL with Testcontainers
```python
import pytest
from testcontainers.postgres import PostgresContainer
import psycopg2

@pytest.fixture(scope="module")
def postgres_container():
    with PostgresContainer("postgres:15") as postgres:
        yield postgres

@pytest.fixture
def db_connection(postgres_container):
    conn = psycopg2.connect(
        host=postgres_container.get_container_host_ip(),
        port=postgres_container.get_exposed_port(5432),
        user=postgres_container.USER,
        password=postgres_container.PASSWORD,
        dbname=postgres_container.DB_NAME
    )
    conn.autocommit = True
    yield conn
    conn.close()

def test_complex_query(db_connection):
    db_connection.execute("""
        CREATE TABLE products (
            id SERIAL PRIMARY KEY,
            name VARCHAR(100),
            price DECIMAL(10,2),
            category VARCHAR(50)
        )
    """)
    db_connection.execute("""
        INSERT INTO products (name, price, category) VALUES
        ('Laptop', 999.99, 'Electronics'),
        ('Phone', 599.99, 'Electronics'),
        ('Book', 19.99, 'Media')
    """)

    cursor = db_connection.execute("""
        SELECT category, COUNT(*) as count, AVG(price) as avg_price
        FROM products
        GROUP BY category
    """)
    results = cursor.fetchall()
    assert len(results) == 2
    electronics = [r for r in results if r[0] == 'Electronics'][0]
    assert electronics[1] == 2  # Count
```

### Django Database Testing
```python
import pytest
from django.test import TestCase

class TestUserModel(TestCase):
    def setUp(self):
        from myapp.models import User
        self.user = User.objects.create(
            username="alice",
            email="alice@example.com",
            is_active=True
        )

    def test_user_creation(self):
        from myapp.models import User
        user = User.objects.get(username="alice")
        assert user.email == "alice@example.com"
        assert user.is_active

    def test_user_deactivation(self):
        from myapp.models import User
        self.user.is_active = False
        self.user.save()
        
        user = User.objects.get(pk=self.user.pk)
        assert not user.is_active

    def test_unique_email(self):
        from myapp.models import User
        from django.db import IntegrityError
        with self.assertRaises(IntegrityError):
            User.objects.create(
                username="bob",
                email="alice@example.com"  # Duplicate email
            )
```

## API testing
### What It Is
API integration testing validates that HTTP endpoints work correctly: request handling, authentication, validation, response formatting, and error handling. Tests exercise the actual HTTP layer, making real requests to a test server.

### Why It Is Important
API tests catch issues with routing, serialization, authentication middleware, rate limiting, and protocol-level errors that unit tests miss. They validate the full request-response cycle.

### How It Works Internally
API tests typically:
1. Start a test server (or use a test client)
2. Send HTTP requests with various payloads
3. Validate response status, headers, and body
4. Check side effects in the database

```python
import pytest
from fastapi import FastAPI
from fastapi.testclient import TestClient
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float
    in_stock: bool = True

items_db = {}

@app.post("/items/")
def create_item(item: Item):
    item_id = len(items_db) + 1
    items_db[item_id] = item.model_dump()
    return {"id": item_id, **item.model_dump()}

@app.get("/items/{item_id}")
def get_item(item_id: int):
    item = items_db.get(item_id)
    if not item:
        return {"error": "Item not found"}, 404
    return item

@app.get("/items/")
def list_items():
    return list(items_db.values())

# Integration tests
class TestItemAPI:
    @pytest.fixture(autouse=True)
    def setup(self):
        items_db.clear()
        self.client = TestClient(app)

    def test_create_item(self):
        response = self.client.post("/items/", json={
            "name": "Laptop",
            "price": 999.99
        })
        assert response.status_code == 200
        data = response.json()
        assert data["name"] == "Laptop"
        assert data["id"] == 1

    def test_get_nonexistent_item(self):
        response = self.client.get("/items/999")
        assert response.status_code == 404

    def test_create_invalid_item(self):
        response = self.client.post("/items/", json={
            "name": "Test",
            "price": "not_a_number"  # Invalid type
        })
        assert response.status_code == 422  # Validation error

    def test_list_items(self):
        self.client.post("/items/", json={"name": "A", "price": 1.0})
        self.client.post("/items/", json={"name": "B", "price": 2.0})
        
        response = self.client.get("/items/")
        assert len(response.json()) == 2
```

### Flask API Testing
```python
import pytest
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    if not data or 'email' not in data:
        return jsonify({"error": "Email required"}), 400
    return jsonify({"id": 1, "email": data['email']}), 201

@pytest.fixture
def client():
    app.config['TESTING'] = True
    with app.test_client() as client:
        yield client

def test_create_user_success(client):
    response = client.post('/api/users', json={
        "email": "user@example.com",
        "name": "Alice"
    })
    assert response.status_code == 201
    assert response.json['email'] == "user@example.com"

def test_create_user_missing_email(client):
    response = client.post('/api/users', json={"name": "Alice"})
    assert response.status_code == 400

def test_create_user_invalid_json(client):
    response = client.post('/api/users', data="not json",
                          content_type='application/json')
    assert response.status_code == 400
```

## External services testing
### What It Is
External service integration testing validates interactions with third-party services: payment gateways, email providers, cloud services, message queues, and external APIs. These tests often use sandbox environments, test accounts, or containerized dependencies.

### Why It Is Important
External services introduce failure modes not present in internal code: network latency, rate limits, authentication expiry, API version changes, and service outages. Testing these interactions ensures proper error handling and resilience.

### How It Works Internally
Three main strategies for testing external services:

1. **Sandbox/Test Environment**: Using the service's test mode
2. **Service Virtualization**: Running a local mock server
3. **Contract Testing**: Verifying API contracts independently

```python
import pytest
import responses
import requests

# External payment service
class PaymentGateway:
    def __init__(self, api_key, base_url="https://api.payments.com"):
        self.api_key = api_key
        self.base_url = base_url

    def charge(self, amount, currency="USD", source="tok_visa"):
        response = requests.post(
            f"{self.base_url}/charges",
            json={
                "amount": amount,
                "currency": currency,
                "source": source
            },
            headers={"Authorization": f"Bearer {self.api_key}"}
        )
        response.raise_for_status()
        return response.json()

    def refund(self, charge_id):
        response = requests.post(
            f"{self.base_url}/charges/{charge_id}/refund"
        )
        response.raise_for_status()
        return response.json()

class PaymentService:
    def __init__(self, gateway):
        self.gateway = gateway

    def process_payment(self, order_id, amount, card_token):
        try:
            result = self.gateway.charge(amount, source=card_token)
            return {
                "status": "success",
                "charge_id": result["id"],
                "amount": result["amount"]
            }
        except requests.RequestException as e:
            return {
                "status": "failed",
                "error": str(e)
            }

# Integration test with responses library
@responses.activate
def test_successful_payment():
    responses.add(
        responses.POST,
        "https://api.payments.com/charges",
        json={
            "id": "ch_123456",
            "amount": 5000,
            "currency": "usd",
            "status": "succeeded"
        },
        status=200
    )

    gateway = PaymentGateway("sk_test_key")
    service = PaymentService(gateway)
    result = service.process_payment(1, 50.00, "tok_visa")
    
    assert result["status"] == "success"
    assert result["charge_id"] == "ch_123456"

@responses.activate
def test_payment_declined():
    responses.add(
        responses.POST,
        "https://api.payments.com/charges",
        json={"error": {"message": "Card declined"}},
        status=402
    )

    gateway = PaymentGateway("sk_test_key")
    service = PaymentService(gateway)
    result = service.process_payment(1, 50.00, "tok_decline")
    
    assert result["status"] == "failed"

@responses.activate
def test_payment_network_timeout():
    responses.add(
        responses.POST,
        "https://api.payments.com/charges",
        body=requests.ConnectionError("Connection timeout")
    )

    gateway = PaymentGateway("sk_test_key")
    service = PaymentService(gateway)
    result = service.process_payment(1, 50.00, "tok_visa")
    
    assert result["status"] == "failed"
    assert "timeout" in result["error"].lower()
```

### Email Service Testing
```python
import pytest
import smtplib
from unittest.mock import patch
import socket

class EmailService:
    def __init__(self, smtp_host, smtp_port, username, password):
        self.smtp_host = smtp_host
        self.smtp_port = smtp_port
        self.username = username
        self.password = password

    def send_email(self, to_addr, subject, body):
        try:
            with smtplib.SMTP(self.smtp_host, self.smtp_port, timeout=10) as server:
                server.starttls()
                server.login(self.username, self.password)
                message = f"Subject: {subject}\n\n{body}"
                server.sendmail(self.username, [to_addr], message)
            return {"status": "sent"}
        except smtplib.SMTPException as e:
            return {"status": "failed", "error": str(e)}
        except socket.timeout:
            return {"status": "failed", "error": "timeout"}

# Integration test with local SMTP server (e.g., MailHog)
class TestEmailService:
    @pytest.fixture
    def email_service(self):
        return EmailService(
            smtp_host="localhost",
            smtp_port=1025,  # MailHog default
            username="test@example.com",
            password="testpass"
        )

    @pytest.mark.integration
    def test_send_email_success(self, email_service):
        result = email_service.send_email(
            to_addr="recipient@example.com",
            subject="Test",
            body="Hello, this is a test email."
        )
        assert result["status"] == "sent"
```

### Redis Cache Testing
```python
import pytest
import redis

class CacheService:
    def __init__(self, host="localhost", port=6379, db=0):
        self.client = redis.Redis(host=host, port=port, db=db, decode_responses=True)

    def get(self, key):
        return self.client.get(key)

    def set(self, key, value, ttl=None):
        if ttl:
            self.client.setex(key, ttl, value)
        else:
            self.client.set(key, value)

    def delete(self, key):
        self.client.delete(key)

    def exists(self, key):
        return self.client.exists(key) > 0

class TestCacheService:
    @pytest.fixture
    def cache(self):
        service = CacheService(db=1)  # Use separate database
        service.client.flushdb()       # Clean slate
        yield service
        service.client.flushdb()       # Cleanup

    def test_set_and_get(self, cache):
        cache.set("name", "Alice")
        assert cache.get("name") == "Alice"

    def test_set_with_ttl(self, cache):
        cache.set("temp", "value", ttl=1)
        assert cache.get("temp") == "value"
        import time
        time.sleep(1.1)
        assert cache.get("temp") is None

    def test_nonexistent_key(self, cache):
        assert cache.get("nonexistent") is None

    def test_exists(self, cache):
        cache.set("key", "value")
        assert cache.exists("key")
        cache.delete("key")
        assert not cache.exists("key")
```

### Message Queue Testing (RabbitMQ)
```python
import pytest
import pika

class MessagePublisher:
    def __init__(self, host="localhost"):
        self.host = host

    def publish(self, queue, message):
        connection = pika.BlockingConnection(
            pika.ConnectionParameters(host=self.host)
        )
        channel = connection.channel()
        channel.queue_declare(queue=queue, durable=True)
        channel.basic_publish(
            exchange='',
            routing_key=queue,
            body=message,
            properties=pika.BasicProperties(
                delivery_mode=2  # Persistent
            )
        )
        connection.close()

class TestMessageQueue:
    @pytest.fixture
    def publisher(self):
        return MessagePublisher()

    @pytest.mark.integration
    def test_publish_message(self, publisher):
        publisher.publish("test_queue", "hello world")
        
        # Verify by consuming the message
        connection = pika.BlockingConnection(
            pika.ConnectionParameters(host="localhost")
        )
        channel = connection.channel()
        method, properties, body = channel.basic_get(
            queue="test_queue", auto_ack=True
        )
        assert body == b"hello world"
        connection.close()
```

### Beginner Examples
```python
import pytest
import json
from pathlib import Path

# Simple file-based integration test
class FileProcessor:
    def process_file(self, filepath):
        with open(filepath, 'r') as f:
            data = json.load(f)
        return [item['name'] for item in data['items']]

class TestFileProcessor:
    @pytest.fixture
    def temp_file(self, tmp_path):
        filepath = tmp_path / "data.json"
        filepath.write_text(json.dumps({
            "items": [
                {"name": "Alice", "age": 30},
                {"name": "Bob", "age": 25}
            ]
        }))
        return filepath

    def test_process_file(self, temp_file):
        processor = FileProcessor()
        result = processor.process_file(str(temp_file))
        assert result == ["Alice", "Bob"]
```

### Advanced Examples
```python
import pytest
import asyncio
import aiohttp

class AsyncAPIClient:
    def __init__(self, base_url):
        self.base_url = base_url

    async def get_user(self, user_id):
        async with aiohttp.ClientSession() as session:
            async with session.get(f"{self.base_url}/users/{user_id}") as resp:
                resp.raise_for_status()
                return await resp.json()

    async def create_user(self, data):
        async with aiohttp.ClientSession() as session:
            async with session.post(f"{self.base_url}/users", json=data) as resp:
                return await resp.json()

@pytest.mark.asyncio
class TestAsyncAPI:
    @pytest.fixture
    def client(self):
        # Use testcontainers or a real test server
        return AsyncAPIClient("http://localhost:8000")

    async def test_get_user(self, client):
        user = await client.get_user(1)
        assert user["id"] == 1
        assert "name" in user

    async def test_create_user(self, client):
        data = {"name": "Alice", "email": "alice@example.com"}
        result = await client.create_user(data)
        assert result["id"] is not None
```

### Real-World Use Cases
- Testing database migrations and schema changes
- Validating REST API contracts
- Verifying payment gateway integration
- Testing email delivery pipeline
- Cache invalidation strategies
- Message queue reliability
- Third-party API error handling

### Common Mistakes
- Not cleaning up test data between runs
- Using production-like data that causes false positives
- Tests that depend on execution order
- Not testing error conditions and timeouts
- Tests that are not repeatable (depend on external state)
- Mixing integration and unit tests in the same suite

### Best Practices
- Use dedicated test databases/containers
- Clean up data between tests
- Test error scenarios, not just happy paths
- Use retry mechanisms for flaky tests
- Separate integration tests from unit tests
- Use docker/containers for reproducible environments
- Implement circuit breakers for external service tests

### Performance Considerations
- Integration tests are 10-100x slower than unit tests
- Use test grouping to run fast tests first
- Reuse expensive fixtures (e.g., database connections)
- Parallelize independent integration tests
- Use database transactions for rollback instead of cleanup
- Consider snapshot testing for complex responses

### Interview Questions
1. How do you test database interactions without corrupting test data?
2. What strategies exist for testing third-party API integrations?
3. How do you handle flaky integration tests?
4. Explain the difference between unit, integration, and end-to-end tests.
5. How do you test async operations in integration tests?

### Coding Challenges
1. Write integration tests for a user registration flow (DB + email)
2. Test a payment processing pipeline with mocked gateway
3. Create integration tests for a caching layer with Redis
4. Test a message queue publish-consume cycle

### Related Topics
- pytest fixtures and conftest.py
- Testcontainers library
- Docker for test environments
- WireMock for API mocking
- LocalStack for AWS service testing
- Contract testing with Pact
- End-to-end testing with Selenium/Playwright
