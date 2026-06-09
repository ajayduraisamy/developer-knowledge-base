# Adapter - Interface compatibility, class/object adapter patterns

## Introduction

The Adapter pattern allows objects with incompatible interfaces to collaborate. It acts as a wrapper that converts the interface of a class into another interface that clients expect. This structural pattern is essential when integrating legacy code, third-party libraries, or systems with mismatched interfaces.

## Why It Is Important

Real-world systems rarely have perfectly matching interfaces. The Adapter pattern enables integration without modifying existing code, following the Open/Closed Principle. It is crucial for wrapping third-party libraries, normalizing diverse data sources, making legacy systems compatible with modern code, and bridging different architectural layers.

## Syntax

The Adapter pattern has two forms: class adapter (using multiple inheritance) and object adapter (using composition). The object adapter is more flexible and is the preferred approach in Python. It involves a Target interface, an Adaptee (existing class), and an Adapter that implements Target and delegates to Adaptee.

## Examples

```python
# Object Adapter pattern
from abc import ABC, abstractmethod


class Target(ABC):
    @abstractmethod
    def request(self) -> str:
        pass


class Adaptee:
    def specific_request(self) -> str:
        return "Specific behavior from Adaptee"


class Adapter(Target):
    def __init__(self, adaptee: Adaptee):
        self._adaptee = adaptee

    def request(self) -> str:
        return f"Adapter: {self._adaptee.specific_request()}"


def client_code(target: Target):
    print(target.request())


adaptee = Adaptee()
adapter = Adapter(adaptee)
client_code(adapter)
```

```python
# Class Adapter (multiple inheritance)
class ClassAdapter(Target, Adaptee):
    def request(self) -> str:
        return f"ClassAdapter: {self.specific_request()}"


class_adapter = ClassAdapter()
client_code(class_adapter)
```

```python
# Adapting different payment APIs
class PaymentProcessor(ABC):
    @abstractmethod
    def pay(self, amount: float) -> str:
        pass


class StripeAPI:
    def charge(self, amount_cents: int) -> str:
        return f"Charged ${amount_cents / 100:.2f} via Stripe"


class StripeAdapter(PaymentProcessor):
    def __init__(self, stripe: StripeAPI):
        self._stripe = stripe

    def pay(self, amount: float) -> str:
        return self._stripe.charge(int(amount * 100))


class PayPalAPI:
    def send_payment(self, usd_amount: float) -> str:
        return f"Sent ${usd_amount:.2f} via PayPal"


class PayPalAdapter(PaymentProcessor):
    def __init__(self, paypal: PayPalAPI):
        self._paypal = paypal

    def pay(self, amount: float) -> str:
        return self._paypal.send_payment(amount)


class SquareAPI:
    def process(self, data: dict) -> dict:
        return {"status": "ok", "amount": data["amount"]}


class SquareAdapter(PaymentProcessor):
    def __init__(self, square: SquareAPI):
        self._square = square

    def pay(self, amount: float) -> str:
        result = self._square.process({"amount": amount})
        return f"Processed ${amount:.2f} via Square (status: {result['status']})"


def process_payment(processor: PaymentProcessor, amount: float):
    print(processor.pay(amount))


process_payment(StripeAdapter(StripeAPI()), 49.99)
process_payment(PayPalAdapter(PayPalAPI()), 29.99)
process_payment(SquareAdapter(SquareAPI()), 99.99)
```

## Beginner Examples

```python
# Adapting temperature sensors
class CelsiusSensor:
    def get_temperature_celsius(self) -> float:
        return 25.0


class FahrenheitSensor:
    def get_temperature_fahrenheit(self) -> float:
        return 77.0


class TemperatureDisplay:
    def show(self, celsius: float) -> str:
        return f"Current temperature: {celsius:.1f} C"


class FahrenheitAdapter:
    def __init__(self, sensor: FahrenheitSensor):
        self._sensor = sensor

    def get_temperature_celsius(self) -> float:
        f = self._sensor.get_temperature_fahrenheit()
        return (f - 32) * 5 / 9


display = TemperatureDisplay()
celsius_sensor = CelsiusSensor()
fahrenheit_sensor = FahrenheitAdapter(FahrenheitSensor())

print(display.show(celsius_sensor.get_temperature_celsius()))
print(display.show(fahrenheit_sensor.get_temperature_celsius()))
```

```python
# Adapting different logging libraries
class Logger(ABC):
    @abstractmethod
    def log(self, level: str, message: str) -> None:
        pass


class PythonLoggingAdapter(Logger):
    def __init__(self):
        import logging
        self._logger = logging.getLogger("app")
        self._logger.setLevel(logging.DEBUG)

    def log(self, level: str, message: str) -> None:
        level_map = {
            "DEBUG": logging.DEBUG,
            "INFO": logging.INFO,
            "WARN": logging.WARNING,
            "ERROR": logging.ERROR,
        }
        self._logger.log(level_map.get(level, logging.INFO), message)


class StructLogAdapter(Logger):
    def __init__(self):
        import structlog
        self._logger = structlog.get_logger()

    def log(self, level: str, message: str) -> None:
        getattr(self._logger, level.lower())(message)


class LoguruAdapter(Logger):
    def __init__(self):
        from loguru import logger
        self._logger = logger

    def log(self, level: str, message: str) -> None:
        self._logger.log(level, message)


def log_error(logger: Logger, message: str):
    logger.log("ERROR", message)
```

## Intermediate Examples

```python
# Adapting diverse data sources
from typing import List, Dict, Any
import json
import csv
from io import StringIO


class DataSource(ABC):
    @abstractmethod
    def get_data(self) -> List[Dict[str, Any]]:
        pass


class JSONDataSource:
    def __init__(self, json_string: str):
        self._data = json.loads(json_string)

    def read_json(self) -> list:
        return self._data


class CSVDataSource:
    def __init__(self, csv_string: str):
        self._csv_string = csv_string

    def read_csv(self) -> str:
        return self._csv_string


class XMLDataSource:
    def __init__(self, xml_string: str):
        self._xml_string = xml_string

    def parse_xml(self) -> list:
        import xml.etree.ElementTree as ET
        root = ET.fromstring(self._xml_string)
        records = []
        for item in root.findall("item"):
            record = {}
            for child in item:
                record[child.tag] = child.text
            records.append(record)
        return records


class JSONAdapter(DataSource):
    def __init__(self, source: JSONDataSource):
        self._source = source

    def get_data(self) -> List[Dict[str, Any]]:
        return self._source.read_json()


class CSVAdapter(DataSource):
    def __init__(self, source: CSVDataSource):
        self._source = source

    def get_data(self) -> List[Dict[str, Any]]:
        reader = csv.DictReader(StringIO(self._source.read_csv()))
        return list(reader)


class XMLAdapter(DataSource):
    def __init__(self, source: XMLDataSource):
        self._source = source

    def get_data(self) -> List[Dict[str, Any]]:
        return self._source.parse_xml()


def analyze_data(source: DataSource):
    data = source.get_data()
    print(f"Records: {len(data)}")
    if data:
        print(f"Fields: {list(data[0].keys())}")


json_data = '[{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]'
csv_data = "name,age\nAlice,30\nBob,25"
xml_data = "<data><item><name>Alice</name><age>30</age></item><item><name>Bob</name><age>25</age></item></data>"

analyze_data(JSONAdapter(JSONDataSource(json_data)))
analyze_data(CSVAdapter(CSVDataSource(csv_data)))
analyze_data(XMLAdapter(XMLDataSource(xml_data)))
```

```python
# Adapting different cache backends
from typing import Any, Optional
import time


class CacheBackend(ABC):
    @abstractmethod
    def get(self, key: str) -> Optional[Any]:
        pass

    @abstractmethod
    def set(self, key: str, value: Any, ttl: int = 0) -> None:
        pass

    @abstractmethod
    def delete(self, key: str) -> None:
        pass


class RedisCache:
    def __init__(self):
        self._store = {}

    def get_value(self, key: str) -> Optional[str]:
        item = self._store.get(key)
        if item and (item["expires"] == 0 or item["expires"] > time.time()):
            return item["value"]
        return None

    def set_value(self, key: str, value: str, expiry: int = 0) -> None:
        self._store[key] = {"value": value, "expires": time.time() + expiry if expiry else 0}

    def remove(self, key: str) -> None:
        self._store.pop(key, None)


class MemcachedCache:
    def __init__(self):
        self._data = {}

    def fetch(self, key: str) -> Any:
        return self._data.get(key)

    def store(self, key: str, value: Any, seconds: int = 0) -> None:
        self._data[key] = value

    def erase(self, key: str) -> None:
        self._data.pop(key, None)


class RedisAdapter(CacheBackend):
    def __init__(self, redis: RedisCache):
        self._redis = redis

    def get(self, key: str) -> Optional[Any]:
        return self._redis.get_value(key)

    def set(self, key: str, value: Any, ttl: int = 0) -> None:
        self._redis.set_value(key, value, ttl)

    def delete(self, key: str) -> None:
        self._redis.remove(key)


class MemcachedAdapter(CacheBackend):
    def __init__(self, memcached: MemcachedCache):
        self._memcached = memcached

    def get(self, key: str) -> Optional[Any]:
        return self._memcached.fetch(key)

    def set(self, key: str, value: Any, ttl: int = 0) -> None:
        self._memcached.store(key, value, ttl)

    def delete(self, key: str) -> None:
        self._memcached.erase(key)


class CacheManager:
    def __init__(self, backend: CacheBackend):
        self._backend = backend

    def get_or_set(self, key: str, factory, ttl: int = 60):
        value = self._backend.get(key)
        if value is None:
            value = factory()
            self._backend.set(key, value, ttl)
        return value


cache = CacheManager(RedisAdapter(RedisCache()))
result = cache.get_or_set("my_key", lambda: "computed_value", ttl=300)
print(result)
```

## Advanced Examples

```python
# Adapting third-party authentication providers
from typing import Dict, Any, Optional
from abc import ABC, abstractmethod


class AuthProvider(ABC):
    @abstractmethod
    def authenticate(self, credentials: Dict[str, str]) -> Dict[str, Any]:
        pass

    @abstractmethod
    def get_user_profile(self, token: str) -> Dict[str, Any]:
        pass


class GoogleOAuth2:
    def verify_google_token(self, token: str) -> dict:
        return {"sub": "12345", "email": "user@gmail.com", "name": "Google User"}

    def get_profile(self, access_token: str) -> dict:
        return {"id": "12345", "display_name": "Google User", "email_address": "user@gmail.com"}


class FacebookGraphAPI:
    def validate_access_token(self, token: str) -> dict:
        return {"user_id": "67890", "is_valid": True}

    def fetch_user(self, user_id: str, fields: str) -> dict:
        return {"id": user_id, "first_name": "Facebook", "last_name": "User", "email": "fb@example.com"}


class GitHubOAuth:
    def check_token(self, token: str) -> Optional[dict]:
        return {"login": "githubuser", "id": 11111}

    def get_emails(self) -> list:
        return [{"email": "github@example.com", "primary": True, "verified": True}]


class GoogleAdapter(AuthProvider):
    def __init__(self, google: GoogleOAuth2):
        self._google = google

    def authenticate(self, credentials: Dict[str, str]) -> Dict[str, Any]:
        result = self._google.verify_google_token(credentials["token"])
        return {"provider": "google", "id": result["sub"], "token": credentials["token"]}

    def get_user_profile(self, token: str) -> Dict[str, Any]:
        profile = self._google.get_profile(token)
        return {
            "name": profile["display_name"],
            "email": profile["email_address"],
            "provider": "google",
        }


class FacebookAdapter(AuthProvider):
    def __init__(self, facebook: FacebookGraphAPI):
        self._facebook = facebook

    def authenticate(self, credentials: Dict[str, str]) -> Dict[str, Any]:
        result = self._facebook.validate_access_token(credentials["token"])
        return {"provider": "facebook", "id": result["user_id"], "token": credentials["token"]}

    def get_user_profile(self, token: str) -> Dict[str, Any]:
        auth = self._facebook.validate_access_token(token)
        profile = self._facebook.fetch_user(auth["user_id"], "first_name,last_name,email")
        return {
            "name": f"{profile['first_name']} {profile['last_name']}",
            "email": profile["email"],
            "provider": "facebook",
        }


class GitHubAdapter(AuthProvider):
    def __init__(self, github: GitHubOAuth):
        self._github = github

    def authenticate(self, credentials: Dict[str, str]) -> Dict[str, Any]:
        result = self._github.check_token(credentials["token"])
        return {"provider": "github", "id": str(result["id"]), "token": credentials["token"]}

    def get_user_profile(self, token: str) -> Dict[str, Any]:
        emails = self._github.get_emails()
        primary_email = next((e["email"] for e in emails if e["primary"]), emails[0]["email"])
        profile = self._github.check_token(token)
        return {"name": profile["login"], "email": primary_email, "provider": "github"}


def login(auth: AuthProvider, credentials: Dict[str, str]) -> Dict[str, Any]:
    auth_result = auth.authenticate(credentials)
    profile = auth.get_user_profile(auth_result["token"])
    return {"auth": auth_result, "profile": profile}


providers = {
    "google": GoogleAdapter(GoogleOAuth2()),
    "facebook": FacebookAdapter(FacebookGraphAPI()),
    "github": GitHubAdapter(GitHubOAuth()),
}

for name, provider in providers.items():
    result = login(provider, {"token": "sample_token"})
    print(f"{name}: {result['profile']['name']} ({result['profile']['email']})")
```

```python
# Adapting different database query interfaces
from typing import List, Dict, Any, Optional


class DatabaseAdapter(ABC):
    @abstractmethod
    def find(self, collection: str, query: Dict[str, Any]) -> List[Dict[str, Any]]:
        pass

    @abstractmethod
    def insert(self, collection: str, document: Dict[str, Any]) -> str:
        pass

    @abstractmethod
    def update(self, collection: str, query: Dict[str, Any], update: Dict[str, Any]) -> int:
        pass

    @abstractmethod
    def delete(self, collection: str, query: Dict[str, Any]) -> int:
        pass


class MongoDB:
    def find_documents(self, db: str, coll: str, filter_: dict) -> list:
        return [{"_id": "1", "name": "Alice"}, {"_id": "2", "name": "Bob"}]

    def insert_document(self, db: str, coll: str, doc: dict) -> str:
        return "generated_id_123"

    def update_documents(self, db: str, coll: str, filter_: dict, update_: dict) -> int:
        return 1

    def delete_documents(self, db: str, coll: str, filter_: dict) -> int:
        return 1


class PostgreSQL:
    def execute_query(self, sql: str, params: tuple = ()) -> list:
        return [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]

    def execute_insert(self, sql: str, params: tuple = ()) -> int:
        return 123

    def execute_update(self, sql: str, params: tuple = ()) -> int:
        return 1

    def execute_delete(self, sql: str, params: tuple = ()) -> int:
        return 1


class MongoDBAdapter(DatabaseAdapter):
    def __init__(self, mongo: MongoDB, db_name: str = "default"):
        self._mongo = mongo
        self._db_name = db_name

    def find(self, collection: str, query: Dict[str, Any]) -> List[Dict[str, Any]]:
        return self._mongo.find_documents(self._db_name, collection, query)

    def insert(self, collection: str, document: Dict[str, Any]) -> str:
        return self._mongo.insert_document(self._db_name, collection, document)

    def update(self, collection: str, query: Dict[str, Any], update: Dict[str, Any]) -> int:
        return self._mongo.update_documents(self._db_name, collection, query, update)

    def delete(self, collection: str, query: Dict[str, Any]) -> int:
        return self._mongo.delete_documents(self._db_name, collection, query)


class PostgreSQLAdapter(DatabaseAdapter):
    def __init__(self, pg: PostgreSQL):
        self._pg = pg

    def _build_select(self, collection: str, query: Dict[str, Any]) -> tuple:
        if not query:
            return f"SELECT * FROM {collection}", ()
        conditions = " AND ".join(f"{k} = %s" for k in query)
        return f"SELECT * FROM {collection} WHERE {conditions}", tuple(query.values())

    def find(self, collection: str, query: Dict[str, Any]) -> List[Dict[str, Any]]:
        sql, params = self._build_select(collection, query)
        return self._pg.execute_query(sql, params)

    def insert(self, collection: str, document: Dict[str, Any]) -> str:
        columns = ", ".join(document.keys())
        placeholders = ", ".join("%s" for _ in document)
        values = tuple(document.values())
        sql = f"INSERT INTO {collection} ({columns}) VALUES ({placeholders}) RETURNING id"
        return str(self._pg.execute_insert(sql, values))

    def update(self, collection: str, query: Dict[str, Any], update: Dict[str, Any]) -> int:
        set_clause = ", ".join(f"{k} = %s" for k in update)
        where_clause = " AND ".join(f"{k} = %s" for k in query)
        params = tuple(update.values()) + tuple(query.values())
        sql = f"UPDATE {collection} SET {set_clause} WHERE {where_clause}"
        return self._pg.execute_update(sql, params)

    def delete(self, collection: str, query: Dict[str, Any]) -> int:
        where_clause = " AND ".join(f"{k} = %s" for k in query)
        sql = f"DELETE FROM {collection} WHERE {where_clause}"
        return self._pg.execute_delete(sql, tuple(query.values()))


class UserService:
    def __init__(self, db: DatabaseAdapter):
        self._db = db

    def get_users(self) -> List[Dict[str, Any]]:
        return self._db.find("users", {})

    def create_user(self, name: str, email: str) -> str:
        return self._db.insert("users", {"name": name, "email": email})


mongo_service = UserService(MongoDBAdapter(MongoDB()))
pg_service = UserService(PostgreSQLAdapter(PostgreSQL()))

print(mongo_service.get_users())
print(pg_service.get_users())
```

```python
# Adapter for file system operations
from pathlib import Path
from typing import List, Optional


class FileSystem(ABC):
    @abstractmethod
    def read_file(self, path: str) -> str:
        pass

    @abstractmethod
    def write_file(self, path: str, content: str) -> None:
        pass

    @abstractmethod
    def list_files(self, directory: str) -> List[str]:
        pass

    @abstractmethod
    def file_exists(self, path: str) -> bool:
        pass


class LocalFileSystem:
    def read(self, filepath: str) -> str:
        with open(filepath, "r") as f:
            return f.read()

    def write(self, filepath: str, data: str) -> None:
        with open(filepath, "w") as f:
            f.write(data)

    def listdir(self, dirpath: str) -> List[str]:
        import os
        return os.listdir(dirpath)

    def exists(self, filepath: str) -> bool:
        import os
        return os.path.exists(filepath)


class S3FileSystem:
    def __init__(self, bucket: str):
        self._bucket = bucket
        self._store = {}

    def get_object(self, key: str) -> Optional[str]:
        return self._store.get(f"{self._bucket}/{key}")

    def put_object(self, key: str, data: str) -> None:
        self._store[f"{self._bucket}/{key}"] = data

    def list_objects(self, prefix: str = "") -> List[str]:
        return [k for k in self._store if k.startswith(f"{self._bucket}/{prefix}")]

    def object_exists(self, key: str) -> bool:
        return f"{self._bucket}/{key}" in self._store


class LocalAdapter(FileSystem):
    def __init__(self, fs: LocalFileSystem):
        self._fs = fs

    def read_file(self, path: str) -> str:
        return self._fs.read(path)

    def write_file(self, path: str, content: str) -> None:
        self._fs.write(path, content)

    def list_files(self, directory: str) -> List[str]:
        return self._fs.listdir(directory)

    def file_exists(self, path: str) -> bool:
        return self._fs.exists(path)


class S3Adapter(FileSystem):
    def __init__(self, s3: S3FileSystem):
        self._s3 = s3

    def read_file(self, path: str) -> str:
        result = self._s3.get_object(path)
        if result is None:
            raise FileNotFoundError(f"{path} not found in S3")
        return result

    def write_file(self, path: str, content: str) -> None:
        self._s3.put_object(path, content)

    def list_files(self, directory: str) -> List[str]:
        return self._s3.list_objects(directory)

    def file_exists(self, path: str) -> bool:
        return self._s3.object_exists(path)


class FileManager:
    def __init__(self, fs: FileSystem):
        self._fs = fs

    def read_config(self, path: str) -> dict:
        import json
        content = self._fs.read_file(path)
        return json.loads(content)

    def save_config(self, path: str, config: dict) -> None:
        import json
        self._fs.write_file(path, json.dumps(config, indent=2))

    def backup_files(self, directory: str) -> List[str]:
        return self._fs.list_files(directory)


local_manager = FileManager(LocalAdapter(LocalFileSystem()))
s3_manager = FileManager(S3Adapter(S3FileSystem("my-bucket")))
```

## Real-World Use Cases

```python
# Adapting different messaging/queue systems
class MessageQueue(ABC):
    @abstractmethod
    def publish(self, topic: str, message: str) -> None:
        pass

    @abstractmethod
    def subscribe(self, topic: str, callback) -> None:
        pass


class RabbitMQClient:
    def send(self, exchange: str, routing_key: str, body: str) -> None:
        print(f"RabbitMQ: Sent to {exchange}/{routing_key}: {body}")

    def consume(self, queue: str, on_message) -> None:
        print(f"RabbitMQ: Consuming from {queue}")


class KafkaClient:
    def produce(self, topic: str, value: str) -> None:
        print(f"Kafka: Produced to {topic}: {value}")

    def poll(self, topic: str, handler) -> None:
        print(f"Kafka: Polling {topic}")


class SQSClient:
    def send_message(self, queue_url: str, message_body: str) -> None:
        print(f"SQS: Sent to {queue_url}: {message_body}")

    def receive_messages(self, queue_url: str, handler) -> None:
        print(f"SQS: Receiving from {queue_url}")


class RabbitMQAdapter(MessageQueue):
    def __init__(self, client: RabbitMQClient, exchange: str = "default"):
        self._client = client
        self._exchange = exchange

    def publish(self, topic: str, message: str) -> None:
        self._client.send(self._exchange, topic, message)

    def subscribe(self, topic: str, callback) -> None:
        self._client.consume(topic, callback)


class KafkaAdapter(MessageQueue):
    def __init__(self, client: KafkaClient):
        self._client = client

    def publish(self, topic: str, message: str) -> None:
        self._client.produce(topic, message)

    def subscribe(self, topic: str, callback) -> None:
        self._client.poll(topic, callback)


class SQSAdapter(MessageQueue):
    def __init__(self, client: SQSClient, queue_prefix: str = ""):
        self._client = client
        self._queue_prefix = queue_prefix

    def publish(self, topic: str, message: str) -> None:
        queue_url = f"{self._queue_prefix}/{topic}"
        self._client.send_message(queue_url, message)

    def subscribe(self, topic: str, callback) -> None:
        queue_url = f"{self._queue_prefix}/{topic}"
        self._client.receive_messages(queue_url, callback)


class NotificationService:
    def __init__(self, queue: MessageQueue):
        self._queue = queue

    def send_welcome_email(self, email: str):
        self._queue.publish("email.welcome", f"Welcome {email}!")

    def send_order_confirmation(self, order_id: str):
        self._queue.publish("order.confirmed", f"Order {order_id} confirmed")


service = NotificationService(RabbitMQAdapter(RabbitMQClient()))
service.send_welcome_email("user@example.com")
```

```python
# Adapter pattern in testing — mocking external APIs
class WeatherAPI:
    def get_forecast(self, city: str, country: str) -> dict:
        import requests
        resp = requests.get(f"https://api.weather.com/{country}/{city}")
        return resp.json()


class WeatherService:
    def __init__(self, api: WeatherAPI):
        self._api = api

    def get_temperature(self, city: str, country: str) -> float:
        data = self._api.get_forecast(city, country)
        return data["current"]["temp"]


# Testing adapter
class MockWeatherAPI:
    def __init__(self):
        self._data = {
            ("London", "UK"): {"current": {"temp": 15.0, "condition": "Cloudy"}},
            ("Paris", "FR"): {"current": {"temp": 22.0, "condition": "Sunny"}},
        }

    def get_forecast(self, city: str, country: str) -> dict:
        return self._data.get((city, country), {"current": {"temp": 0.0}})


def test_weather_service():
    api = MockWeatherAPI()
    service = WeatherService(api)
    assert service.get_temperature("London", "UK") == 15.0
    assert service.get_temperature("Paris", "FR") == 22.0
    print("Tests passed!")


test_weather_service()
```

## Common Mistakes

```python
# MISTAKE 1: Adapter modifying the Adaptee's interface
class BadAdapter:
    def __init__(self, adaptee):
        self._adaptee = adaptee

    def request(self):
        self._adaptee.specific_request()
        self._adaptee.new_method()  # Don't add behavior
```

```python
# MISTAKE 2: Not handling adaptee exceptions properly
class FragileAdapter:
    def request(self):
        return self._adaptee.specific_request()
        # What if specific_request raises?
```

```python
# MISTAKE 3: Using adapter when refactoring is possible
# If you own the adaptee code, refactor instead of adapting
```

```python
# MISTAKE 4: Building too many adapters (overengineering)
# Not every interface mismatch needs an adapter
```

## Best Practices

```python
# 1. Prefer object adapter (composition) over class adapter
# 2. Keep the adapter interface focused and minimal
# 3. Handle adaptee exceptions and translate them
# 4. Document the interface mapping
# 5. Consider using a facade instead for complex subsystems
```

```python
# Best practice: Adapter with error translation
class SafeAdapter(Target):
    def request(self) -> str:
        try:
            return self._adaptee.specific_request()
        except AdapteeError as e:
            raise TargetError(f"Adaptation failed: {e}") from e
```

## Interview Questions

```python
# Q1: Object Adapter vs Class Adapter?
# Object adapter: composition, flexible, Pythonic
# Class adapter: multiple inheritance, less flexible
```

```python
# Q2: Adapter vs Facade pattern?
# Adapter: converts interface for compatibility
# Facade: simplifies complex subsystem interface
```

```python
# Q3: Adapter vs Bridge pattern?
# Adapter: makes incompatible interfaces work together
# Bridge: separates abstraction from implementation
```

## Coding Challenges

```python
# Challenge 1: Create an adapter that makes different
# ORMs (SQLAlchemy, Django ORM, Peewee) interchangeable
```

```python
# Challenge 2: Build adapters for different AI/ML model
# serving frameworks (TensorFlow Serving, TorchServe, ONNX)
```

```python
# Challenge 3: Implement adapters for different
# configuration file formats (YAML, TOML, INI, JSON)
```

```python
# Challenge 4: Create adapters for different caching
# backends (Redis, Memcached, local dict, database)
```

```python
# Challenge 5: Build adapters for different email
# services (SMTP, SendGrid, Mailgun, AWS SES)
```

## Summary

The Adapter pattern enables incompatible interfaces to work together through a wrapper that translates calls. Python's dynamic typing and duck typing make adapters particularly lightweight. The object adapter (composition) is preferred over the class adapter (inheritance). Adapters are essential for integrating third-party libraries, legacy systems, and diverse data sources without modifying existing code.

## Related Topics

- Facade Pattern: Simplifies interfaces rather than converting them
- Bridge Pattern: Decouples abstraction from implementation
- Proxy Pattern: Controls access rather than converting interfaces
- Decorator Pattern: Adds behavior without changing interface
- Dependency Injection: Often used with adapters
