# Integration Testing - Testing databases, APIs, and external services

## Introduction

Integration testing validates that different modules, services, or components work together correctly. Unlike unit tests, which isolate individual units of code, integration tests exercise real interactions between components — databases, APIs, file systems, message queues, and external services. They bridge the gap between unit tests and end-to-end tests, catching interface mismatches, configuration errors, and data format issues.

## Why It Is Important

- **Interface Validation** – Ensures components communicate correctly (API contracts, database schemas).
- **Configuration Testing** – Catches environment-specific issues (connection strings, secrets).
- **Data Integrity** – Validates data flows correctly between systems.
- **Realistic Feedback** – Tests behavior closer to production than mocks can provide.
- **Regression Prevention** – Catches breakage when dependencies change (API versions, schema migrations).
- **CI/CD Confidence** – Provides higher confidence than unit tests alone for deployment decisions.

## Syntax

`python
# Using pytest with a test database
def test_user_can_be_created(db_session):
    user = User(email="test@example.com", name="Test User")
    db_session.add(user)
    db_session.commit()
    assert user.id is not None

# Using httpx with FastAPI TestClient
from fastapi.testclient import TestClient

def test_api_create_item(client):
    response = client.post("/items/", json={"name": "Widget", "price": 9.99})
    assert response.status_code == 201
    assert response.json()["name"] == "Widget"
`

## Examples

### Example 1: Database Integration with SQLAlchemy and pytest

`python
import pytest
from sqlalchemy import create_engine, Column, Integer, String, Float, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime

Base = declarative_base()

class Product(Base):
    __tablename__ = "products"
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    price = Column(Float, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)

class Order(Base):
    __tablename__ = "orders"
    id = Column(Integer, primary_key=True)
    product_id = Column(Integer, nullable=False)
    quantity = Column(Integer, nullable=False)
    total = Column(Float, nullable=False)
    status = Column(String(50), default="pending")

@pytest.fixture(scope="function")
def db_session():
    engine = create_engine("sqlite:///:memory:", echo=False)
    Base.metadata.create_all(engine)
    Session = sessionmaker(bind=engine)
    session = Session()
    yield session
    session.close()
    Base.metadata.drop_all(engine)

@pytest.fixture
def sample_product(db_session):
    product = Product(name="Test Widget", price=19.99)
    db_session.add(product)
    db_session.commit()
    return product

class TestDatabaseIntegration:

    def test_create_product(self, db_session):
        product = Product(name="Laptop", price=999.99)
        db_session.add(product)
        db_session.commit()
        assert product.id is not None
        saved = db_session.query(Product).filter_by(name="Laptop").first()
        assert saved.price == 999.99

    def test_create_order(self, db_session, sample_product):
        order = Order(
            product_id=sample_product.id,
            quantity=2,
            total=sample_product.price * 2,
            status="pending",
        )
        db_session.add(order)
        db_session.commit()
        assert order.id is not None

    def test_query_orders_by_status(self, db_session, sample_product):
        for status in ["pending", "shipped", "delivered"]:
            order = Order(
                product_id=sample_product.id,
                quantity=1,
                total=sample_product.price,
                status=status,
            )
            db_session.add(order)
        db_session.commit()
        pending = db_session.query(Order).filter_by(status="pending").all()
        assert len(pending) == 1

    def test_update_order_status(self, db_session, sample_product):
        order = Order(
            product_id=sample_product.id,
            quantity=1,
            total=sample_product.price,
        )
        db_session.add(order)
        db_session.commit()
        order.status = "shipped"
        db_session.commit()
        updated = db_session.query(Order).get(order.id)
        assert updated.status == "shipped"

    def test_delete_product(self, db_session):
        product = Product(name="Temporary", price=5.0)
        db_session.add(product)
        db_session.commit()
        product_id = product.id
        db_session.delete(product)
        db_session.commit()
        assert db_session.query(Product).get(product_id) is None

    def test_transaction_rollback(self, db_session):
        initial_count = db_session.query(Product).count()
        try:
            product = Product(name="Will Fail", price=-1.0)
            db_session.add(product)
            db_session.commit()
        except Exception:
            db_session.rollback()
        final_count = db_session.query(Product).count()
        assert final_count == initial_count

    def test_bulk_insert(self, db_session):
        products = [
            Product(name=f"Product {i}", price=i * 10.0)
            for i in range(1, 101)
        ]
        db_session.add_all(products)
        db_session.commit()
        assert db_session.query(Product).count() == 100
`

### Example 2: API Integration with FastAPI TestClient

`python
import pytest
from fastapi import FastAPI, HTTPException
from fastapi.testclient import TestClient
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

class ItemCreate(BaseModel):
    name: str
    price: float
    description: Optional[str] = None

class ItemResponse(BaseModel):
    id: int
    name: str
    price: float
    description: Optional[str] = None

items_db = {}
next_id = 1

@app.post("/items/", response_model=ItemResponse, status_code=201)
def create_item(item: ItemCreate):
    global next_id
    item_id = next_id
    next_id += 1
    items_db[item_id] = {
        "id": item_id,
        "name": item.name,
        "price": item.price,
        "description": item.description,
    }
    return items_db[item_id]

@app.get("/items/{item_id}", response_model=ItemResponse)
def get_item(item_id: int):
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    return items_db[item_id]

@app.get("/items/")
def list_items(skip: int = 0, limit: int = 10):
    all_items = list(items_db.values())
    return all_items[skip : skip + limit]

@app.put("/items/{item_id}", response_model=ItemResponse)
def update_item(item_id: int, item: ItemCreate):
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    items_db[item_id].update(item.dict())
    return items_db[item_id]

@app.delete("/items/{item_id}", status_code=204)
def delete_item(item_id: int):
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    del items_db[item_id]

@pytest.fixture
def client():
    return TestClient(app)

@pytest.fixture(autouse=True)
def reset_db():
    items_db.clear()
    global next_id
    next_id = 1

class TestAPIIntegration:

    def test_create_item(self, client):
        response = client.post("/items/", json={
            "name": "Widget",
            "price": 9.99,
            "description": "A useful widget",
        })
        assert response.status_code == 201
        data = response.json()
        assert data["name"] == "Widget"
        assert data["price"] == 9.99
        assert data.get("description") == "A useful widget"
        assert "id" in data

    def test_get_item(self, client):
        create_resp = client.post("/items/", json={"name": "Gadget", "price": 14.99})
        item_id = create_resp.json()["id"]
        response = client.get(f"/items/{item_id}")
        assert response.status_code == 200
        assert response.json()["name"] == "Gadget"

    def test_get_nonexistent_item(self, client):
        response = client.get("/items/999")
        assert response.status_code == 404
        assert response.json()["detail"] == "Item not found"

    def test_list_items(self, client):
        for i in range(5):
            client.post("/items/", json={"name": f"Item {i}", "price": float(i)})
        response = client.get("/items/")
        assert response.status_code == 200
        assert len(response.json()) == 5

    def test_list_items_with_pagination(self, client):
        for i in range(20):
            client.post("/items/", json={"name": f"Item {i}", "price": float(i)})
        page = client.get("/items/?skip=5&limit=5")
        assert len(page.json()) == 5
        assert page.json()[0]["name"] == "Item 5"

    def test_update_item(self, client):
        create_resp = client.post("/items/", json={"name": "Old", "price": 1.0})
        item_id = create_resp.json()["id"]
        response = client.put(f"/items/{item_id}", json={
            "name": "Updated",
            "price": 99.99,
        })
        assert response.status_code == 200
        assert response.json()["name"] == "Updated"

    def test_delete_item(self, client):
        create_resp = client.post("/items/", json={"name": "Delete Me", "price": 0.0})
        item_id = create_resp.json()["id"]
        response = client.delete(f"/items/{item_id}")
        assert response.status_code == 204
        get_resp = client.get(f"/items/{item_id}")
        assert get_resp.status_code == 404

    def test_validation_error(self, client):
        response = client.post("/items/", json={"name": "Bad", "price": "not_a_number"})
        assert response.status_code == 422
`

### Example 3: Testing External Services with HTTPX

`python
import pytest
import httpx

class WeatherService:
    def __init__(self, api_key, base_url="https://api.weather.com"):
        self.api_key = api_key
        self.base_url = base_url

    async def get_temperature(self, city):
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"{self.base_url}/v1/current",
                params={"city": city, "apikey": self.api_key},
                timeout=10,
            )
            response.raise_for_status()
            data = response.json()
            return data["current"]["temp_c"]

    def get_forecast(self, city, days=3):
        with httpx.Client() as client:
            response = client.get(
                f"{self.base_url}/v1/forecast",
                params={"city": city, "days": days, "apikey": self.api_key},
                timeout=10,
            )
            response.raise_for_status()
            return response.json()

class TestWeatherService:

    @pytest.mark.asyncio
    async def test_get_temperature_success(self, respx_mock):
        respx_mock.get("https://api.weather.com/v1/current").respond(
            json={"current": {"temp_c": 22.5}},
            status_code=200,
        )
        service = WeatherService(api_key="test_key")
        temp = await service.get_temperature("London")
        assert temp == 22.5

    @pytest.mark.asyncio
    async def test_get_temperature_api_error(self, respx_mock):
        respx_mock.get("https://api.weather.com/v1/current").respond(
            status_code=401,
        )
        service = WeatherService(api_key="bad_key")
        with pytest.raises(httpx.HTTPStatusError):
            await service.get_temperature("London")

    @pytest.mark.asyncio
    async def test_get_temperature_timeout(self, respx_mock):
        respx_mock.get("https://api.weather.com/v1/current").mock(
            side_effect=httpx.TimeoutException("Request timed out")
        )
        service = WeatherService(api_key="test_key")
        with pytest.raises(httpx.TimeoutException):
            await service.get_temperature("London")

    def test_get_forecast_success(self, respx_mock):
        expected = {"forecast": [{"day": "Mon", "high": 25}]}
        respx_mock.get("https://api.weather.com/v1/forecast").respond(
            json=expected, status_code=200,
        )
        service = WeatherService(api_key="test_key")
        result = service.get_forecast("Paris", days=1)
        assert result == expected

    @pytest.mark.asyncio
    async def test_integration_with_real_api(self):
        import os
        api_key = os.getenv("WEATHER_API_KEY")
        if not api_key:
            pytest.skip("WEATHER_API_KEY not set")
        service = WeatherService(api_key=api_key)
        temp = await service.get_temperature("New York")
        assert isinstance(temp, (int, float))
        assert -50 < temp < 60
`

### Example 4: Testing with Docker Containers (testcontainers)

`python
import pytest
import redis
import psycopg2
from testcontainers.postgres import PostgresContainer
from testcontainers.redis import RedisContainer

class TestDockerContainers:

    @pytest.fixture(scope="class")
    def postgres_container(self):
        with PostgresContainer("postgres:15-alpine") as pg:
            yield pg

    def test_postgres_connection(self, postgres_container):
        conn = psycopg2.connect(
            host=postgres_container.get_container_host_ip(),
            port=postgres_container.get_exposed_port(5432),
            user=postgres_container.USER,
            password=postgres_container.PASSWORD,
            dbname=postgres_container.DBNAME,
        )
        cur = conn.cursor()
        cur.execute("SELECT version();")
        version = cur.fetchone()
        assert "PostgreSQL" in version[0]
        cur.close()
        conn.close()

    def test_postgres_create_table_and_insert(self, postgres_container):
        conn = psycopg2.connect(
            host=postgres_container.get_container_host_ip(),
            port=postgres_container.get_exposed_port(5432),
            user=postgres_container.USER,
            password=postgres_container.PASSWORD,
            dbname=postgres_container.DBNAME,
        )
        cur = conn.cursor()
        cur.execute("CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(100))")
        cur.execute("INSERT INTO users (name) VALUES ('Alice'), ('Bob')")
        conn.commit()
        cur.execute("SELECT COUNT(*) FROM users")
        assert cur.fetchone()[0] == 2
        cur.close()
        conn.close()

    @pytest.fixture(scope="function")
    def redis_client(self):
        with RedisContainer("redis:7-alpine") as redis_container:
            client = redis.Redis(
                host=redis_container.get_container_host_ip(),
                port=redis_container.get_exposed_port(6379),
                decode_responses=True,
            )
            yield client
            client.flushall()

    def test_redis_set_and_get(self, redis_client):
        redis_client.set("key", "value")
        assert redis_client.get("key") == "value"

    def test_redis_expiry(self, redis_client):
        import time
        redis_client.setex("temp", 1, "expires")
        assert redis_client.get("temp") == "expires"
        time.sleep(1.5)
        assert redis_client.get("temp") is None

    def test_redis_list_operations(self, redis_client):
        redis_client.rpush("queue", "job1", "job2", "job3")
        assert redis_client.llen("queue") == 3
        assert redis_client.lpop("queue") == "job1"

    def test_redis_hash_operations(self, redis_client):
        redis_client.hset("user:1", mapping={"name": "Alice", "age": "30"})
        assert redis_client.hget("user:1", "name") == "Alice"
        assert redis_client.hgetall("user:1") == {"name": "Alice", "age": "30"}
`

### Example 5: CI/CD Integration with GitHub Actions

`python
# .github/workflows/test.yml
# name: CI
# on: [push, pull_request]
# jobs:
#   test:
#     runs-on: ubuntu-latest
#     services:
#       postgres:
#         image: postgres:15
#         env:
#           POSTGRES_PASSWORD: testpass
#           POSTGRES_DB: testdb
#         ports:
#           - 5432:5432
#       redis:
#         image: redis:7
#         ports:
#           - 6379:6379
#     steps:
#       - uses: actions/checkout@v4
#       - uses: actions/setup-python@v5
#         with:
#           python-version: "3.12"
#       - run: pip install -r requirements.txt
#       - run: pytest tests/integration/ --cov=app -v
#         env:
#           DATABASE_URL: postgresql://postgres:testpass@localhost/testdb
#           REDIS_URL: redis://localhost:6379

import pytest
import os

class TestCIEnvironment:

    def test_database_url_is_set(self):
        url = os.getenv("DATABASE_URL")
        assert url is not None, "DATABASE_URL must be set in CI"

    def test_redis_url_is_set(self):
        url = os.getenv("REDIS_URL")
        assert url is not None, "REDIS_URL must be set in CI"

    def test_can_connect_to_postgres(self):
        import psycopg2
        url = os.getenv("DATABASE_URL")
        conn = psycopg2.connect(url)
        cur = conn.cursor()
        cur.execute("SELECT 1")
        assert cur.fetchone()[0] == 1
        cur.close()
        conn.close()

    def test_can_connect_to_redis(self):
        import redis
        url = os.getenv("REDIS_URL")
        client = redis.Redis.from_url(url, decode_responses=True)
        client.set("ci_test", "ok")
        assert client.get("ci_test") == "ok"
        client.close()
`

### Example 6: Integration Test with Message Queue (RabbitMQ)

`python
import pytest
import pika
import json
import threading
import time

class MessagePublisher:
    def __init__(self, host="localhost", queue="test_queue"):
        self.host = host
        self.queue = queue
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters(host=host)
        )
        self.channel = self.connection.channel()
        self.channel.queue_declare(queue=queue, durable=True)

    def publish(self, message):
        self.channel.basic_publish(
            exchange="",
            routing_key=self.queue,
            body=json.dumps(message),
            properties=pika.BasicProperties(delivery_mode=2),
        )

    def close(self):
        self.connection.close()

class MessageConsumer:
    def __init__(self, host="localhost", queue="test_queue"):
        self.host = host
        self.queue = queue
        self.messages = []
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters(host=host)
        )
        self.channel = self.connection.channel()
        self.channel.queue_declare(queue=queue, durable=True)
        self.channel.basic_consume(
            queue=queue,
            on_message_callback=self._callback,
            auto_ack=True,
        )

    def _callback(self, ch, method, properties, body):
        self.messages.append(json.loads(body))

    def consume(self, timeout=2):
        self.channel.start_consuming()

    def stop(self):
        self.connection.close()

class TestMessageQueue:

    @pytest.fixture
    def publisher(self):
        pub = MessagePublisher()
        yield pub
        pub.close()

    def test_publish_and_consume(self, publisher):
        consumer = MessageConsumer()
        test_message = {"id": 1, "text": "hello"}
        publisher.publish(test_message)
        time.sleep(0.5)
        consumer.stop()
        assert len(consumer.messages) > 0
        assert consumer.messages[0]["text"] == "hello"

    def test_multiple_messages(self, publisher):
        consumer = MessageConsumer()
        for i in range(5):
            publisher.publish({"index": i})
        time.sleep(0.5)
        consumer.stop()
        assert len(consumer.messages) == 5

    def test_message_order(self, publisher):
        consumer = MessageConsumer()
        for i in range(3):
            publisher.publish({"seq": i})
        time.sleep(0.5)
        consumer.stop()
        seqs = [m["seq"] for m in consumer.messages]
        assert seqs == [0, 1, 2]
`

## Beginner Examples

`python
import pytest
import sqlite3

# Simple database integration test
def test_sqlite_in_memory():
    conn = sqlite3.connect(":memory:")
    conn.execute("CREATE TABLE test (id INTEGER PRIMARY KEY, value TEXT)")
    conn.execute("INSERT INTO test VALUES (1, 'hello')")
    cursor = conn.execute("SELECT value FROM test WHERE id = 1")
    assert cursor.fetchone()[0] == "hello"
    conn.close()

# Simple API test with requests
def test_httpbin_get():
    import requests
    response = requests.get("https://httpbin.org/get")
    assert response.status_code == 200
    data = response.json()
    assert "url" in data
`

## Intermediate Examples

`python
import pytest
import httpx
from fastapi import FastAPI
from fastapi.testclient import TestClient

# Testing a FastAPI app that connects to a database
app = FastAPI()

@app.get("/health")
def health():
    return {"status": "ok"}

@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"id": user_id, "name": "Alice"}

class TestIntermediateIntegration:

    def test_health_endpoint(self):
        client = TestClient(app)
        response = client.get("/health")
        assert response.status_code == 200
        assert response.json()["status"] == "ok"

    def test_get_user(self):
        client = TestClient(app)
        response = client.get("/users/1")
        assert response.status_code == 200
        assert response.json()["name"] == "Alice"

    def test_async_http_call(self):
        import asyncio
        async def fetch():
            async with httpx.AsyncClient() as client:
                resp = await client.get("https://httpbin.org/json")
                return resp.status_code
        status = asyncio.run(fetch())
        assert status == 200
`

## Advanced Examples

`python
import pytest
import asyncio
import asyncpg
from contextlib import asynccontextmanager

class DatabaseManager:
    def __init__(self, dsn):
        self.dsn = dsn
        self.pool = None

    async def connect(self):
        self.pool = await asyncpg.create_pool(self.dsn, min_size=2, max_size=10)

    async def close(self):
        if self.pool:
            await self.pool.close()

    async def fetch_user(self, user_id):
        async with self.pool.acquire() as conn:
            row = await conn.fetchrow(
                "SELECT id, name, email FROM users WHERE id = ", user_id
            )
            if row:
                return dict(row)
            return None

    async def create_user(self, name, email):
        async with self.pool.acquire() as conn:
            row = await conn.fetchrow(
                "INSERT INTO users (name, email) VALUES (, ) RETURNING id",
                name, email,
            )
            return row["id"]

@pytest.mark.asyncio
async def test_database_manager(postgres_dsn):
    mgr = DatabaseManager(postgres_dsn)
    await mgr.connect()
    try:
        user_id = await mgr.create_user("Test User", "test@example.com")
        assert user_id is not None
        user = await mgr.fetch_user(user_id)
        assert user["name"] == "Test User"
        assert user["email"] == "test@example.com"
    finally:
        await mgr.close()

@pytest.mark.asyncio
async def test_transaction_rollback_integration(postgres_dsn):
    mgr = DatabaseManager(postgres_dsn)
    await mgr.connect()
    try:
        user_id = await mgr.create_user("Rollback", "rollback@test.com")
        assert user_id is not None
        # Simulate a failure scenario
        with pytest.raises(Exception):
            async with mgr.pool.acquire() as conn:
                async with conn.transaction():
                    await conn.execute("INSERT INTO users (name) VALUES ('bad')")
                    raise ValueError("Simulated failure")
        # User from before should still exist
        user = await mgr.fetch_user(user_id)
        assert user is not None
    finally:
        await mgr.close()
`

## Real-World Use Cases

- **E-commerce Platform** – Test checkout flow: cart -> payment -> inventory -> shipping -> notification.
- **SaaS Application** – Test user registration -> email verification -> subscription activation -> feature access.
- **Data Pipeline** – Test: ingest from Kafka -> transform in Spark -> load to Redshift -> verify BI reports.
- **Microservices** – Test service A calls service B with correct payload and handles responses/errors.
- **Mobile Backend** – Test: mobile client -> API gateway -> auth service -> data service -> push notification.
- **IoT System** – Test: device sends telemetry -> MQTT broker -> stream processor -> time-series DB -> dashboard.

## Common Mistakes

1. **Testing too much at once** – Integration tests should focus on specific interactions, not entire workflows.
2. **Not cleaning up test data** – Leftover data causes flaky tests and环境污染.
3. **Using production-like data** – Test data should be minimal and controlled.
4. **Slow tests** – Integration tests are slower; keep them focused and use parallel execution.
5. **Testing with real external services** without mock fallback in CI.
6. **Hardcoding connection strings** – Use environment variables and fixtures instead.
7. **Not testing error paths** – Network failures, timeouts, and invalid responses.
8. **Over-relying on integration tests** – Balance with unit tests for fast feedback.

## Best Practices

1. **Use test containers** (Docker) for reproducible dependency environments.
2. **Isolate test data** – Each test gets its own database/collection/queue.
3. **Use fixtures for setup/teardown** – Ensure clean state before each test.
4. **Tag integration tests** – Separate from unit tests with @pytest.mark.integration.
5. **Run integration tests in CI** but separately from fast unit tests.
6. **Use service virtualization** – WireMock, LocalStack, or Testcontainers for external services.
7. **Implement health checks** in tests to fail fast if dependencies aren't available.
8. **Use transactions for DB tests** – Roll back after each test to avoid cleanup code.
9. **Add timeouts** to prevent hung tests from blocking CI.

## Interview Questions

1. **What is the difference between unit and integration testing?** – Unit tests isolate single components; integration tests verify real interactions between components.
2. **How do you handle external API dependencies in integration tests?** – Use a combination of mocked responses (via respx/requests-mock) and a real test service for contract verification.
3. **What is Testcontainers and why use it?** – A library that provides disposable Docker containers for testing, ensuring consistent, reproducible dependency environments.
4. **How do you test database interactions without affecting production data?** – Use in-memory databases (SQLite), testcontainers with Docker, or transactional rollback patterns.
5. **What is the FastAPI TestClient and how does it work?** – A Starlette TestClient that makes HTTP requests to your FastAPI application without running a real server.
6. **How would you structure integration tests in a CI/CD pipeline?** – Use separate CI job stages: unit tests first (fast), then integration tests (slower, with service containers).
7. **What is the difference between respx and requests-mock?** – respx works with httpx (sync/async); requests-mock works with the requests library.

## Coding Challenges

1. Write integration tests for a REST API that manages a library (books, authors, members, borrowing). Use FastAPI TestClient and SQLite.
2. Create a test suite for a file upload service that validates file types, scans for malware (ClamAV via Docker), stores to S3 (MinIO), and records metadata in PostgreSQL.
3. Implement integration tests for a notification system that sends emails (MailHog Docker), push notifications (Firebase mock), and SMS (Twilio mock).
4. Build a test harness for a gRPC service: start the server in a Docker container, make RPC calls using grpcio, and assert responses.
5. Write integration tests for a Redis-backed rate limiter that resets counters, checks TTLs, and handles concurrent requests.

## Summary

Integration testing validates that components work together correctly by testing real interactions with databases, APIs, message queues, and external services. It provides higher confidence than unit testing alone by catching interface mismatches, configuration errors, and integration bugs. Tools like FastAPI TestClient, httpx, testcontainers, and Docker service containers make integration testing practical and reproducible. A balanced test suite includes both unit tests (fast, isolated) and integration tests (realistic, comprehensive).

## Related Topics

- [unittest](./77_unittest.md) – Foundation for writing structured tests.
- [pytest](./78_pytest.md) – Test runner and fixture system used in integration tests.
- [Mocking](./79_mocking.md) – When to mock vs. when to use real dependencies.
- [TDD](./80_tdd.md) – TDD principles applied to integration-level tests.
- [Docker](https://docs.docker.com/) – Containers for reproducible test environments.
- [httpx](https://www.python-httpx.org/) – HTTP client for async API testing.
- [testcontainers-python](https://testcontainers-python.readthedocs.io/) – Docker-based test dependencies.
