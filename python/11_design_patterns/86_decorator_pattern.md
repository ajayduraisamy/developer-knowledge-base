# Decorator Pattern - Structural wrapper, adding behavior dynamically

## Introduction

The Decorator pattern is a structural design pattern that lets you attach new behaviors to objects by placing them inside wrapper objects that contain the behaviors. It is distinct from Python's `@decorator` syntax, though the concept shares the same name. The structural decorator pattern wraps objects dynamically at runtime to extend their functionality.

## Why It Is Important

The Decorator pattern provides a flexible alternative to subclassing for extending functionality. Instead of creating a complex class hierarchy for every combination of features, decorators compose behaviors at runtime. This adheres to the Open/Closed Principle and Single Responsibility Principle, allowing features to be combined in any order without modifying existing code.

## Syntax

The pattern involves a component interface, concrete components, and decorator classes that wrap components. Each decorator implements the same interface as the component and delegates to the wrapped object while adding its own behavior.

## Examples

```python
# Structural Decorator pattern
from abc import ABC, abstractmethod


class Coffee(ABC):
    @abstractmethod
    def cost(self) -> float:
        pass

    @abstractmethod
    def description(self) -> str:
        pass


class SimpleCoffee(Coffee):
    def cost(self) -> float:
        return 2.00

    def description(self) -> str:
        return "Simple coffee"


class CoffeeDecorator(Coffee, ABC):
    def __init__(self, coffee: Coffee):
        self._coffee = coffee

    @abstractmethod
    def cost(self) -> float:
        pass

    @abstractmethod
    def description(self) -> str:
        pass


class MilkDecorator(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 0.50

    def description(self) -> str:
        return self._coffee.description() + ", milk"


class SugarDecorator(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 0.25

    def description(self) -> str:
        return self._coffee.description() + ", sugar"


class WhippedCreamDecorator(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 0.75

    def description(self) -> str:
        return self._coffee.description() + ", whipped cream"


class CaramelDecorator(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 0.60

    def description(self) -> str:
        return self._coffee.description() + ", caramel"


coffee = SimpleCoffee()
print(f"{coffee.description()}: ${coffee.cost():.2f}")

coffee = MilkDecorator(coffee)
coffee = SugarDecorator(coffee)
coffee = WhippedCreamDecorator(coffee)
print(f"{coffee.description()}: ${coffee.cost():.2f}")

fancy = CaramelDecorator(WhippedCreamDecorator(MilkDecorator(SimpleCoffee())))
print(f"{fancy.description()}: ${fancy.cost():.2f}")
```

```python
# Python @decorator vs structural decorator
import functools
import time


def timing_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper


@timing_decorator
def slow_function():
    time.sleep(0.1)
    return "Done"


print(slow_function())


class DataSource(ABC):
    @abstractmethod
    def read(self) -> str:
        pass

    @abstractmethod
    def write(self, data: str) -> None:
        pass


class FileDataSource(DataSource):
    def __init__(self, filename: str):
        self._filename = filename
        self._data = ""

    def read(self) -> str:
        return self._data

    def write(self, data: str) -> None:
        self._data = data


class DataSourceDecorator(DataSource, ABC):
    def __init__(self, source: DataSource):
        self._source = source


class EncryptionDecorator(DataSourceDecorator):
    def read(self) -> str:
        data = self._source.read()
        return self._decrypt(data)

    def write(self, data: str) -> None:
        encrypted = self._encrypt(data)
        self._source.write(encrypted)

    def _encrypt(self, data: str) -> str:
        return ''.join(chr(ord(c) + 1) for c in data)

    def _decrypt(self, data: str) -> str:
        return ''.join(chr(ord(c) - 1) for c in data)


class CompressionDecorator(DataSourceDecorator):
    def read(self) -> str:
        data = self._source.read()
        return self._decompress(data)

    def write(self, data: str) -> None:
        compressed = self._compress(data)
        self._source.write(compressed)

    def _compress(self, data: str) -> str:
        import zlib
        return zlib.compress(data.encode()).hex()

    def _decompress(self, data: str) -> str:
        import zlib
        return zlib.decompress(bytes.fromhex(data)).decode()


source = FileDataSource("test.txt")
source = EncryptionDecorator(source)
source = CompressionDecorator(source)
source.write("Hello World!")
print(source.read())
```

## Beginner Examples

```python
# Text formatting decorators
class Text(ABC):
    @abstractmethod
    def render(self) -> str:
        pass


class PlainText(Text):
    def __init__(self, content: str):
        self._content = content

    def render(self) -> str:
        return self._content


class TextDecorator(Text, ABC):
    def __init__(self, text: Text):
        self._text = text


class BoldDecorator(TextDecorator):
    def render(self) -> str:
        return f"<b>{self._text.render()}</b>"


class ItalicDecorator(TextDecorator):
    def render(self) -> str:
        return f"<i>{self._text.render()}</i>"


class UnderlineDecorator(TextDecorator):
    def render(self) -> str:
        return f"<u>{self._text.render()}</u>"


text = PlainText("Hello World")
text = BoldDecorator(text)
text = ItalicDecorator(text)
print(text.render())
```

```python
# Pizza ordering decorators
class Pizza(ABC):
    @abstractmethod
    def cost(self) -> float:
        pass

    @abstractmethod
    def ingredients(self) -> list:
        pass


class Margherita(Pizza):
    def cost(self) -> float:
        return 8.99

    def ingredients(self) -> list:
        return ["mozzarella", "tomato sauce", "basil"]


class PizzaDecorator(Pizza, ABC):
    def __init__(self, pizza: Pizza):
        self._pizza = pizza


class ExtraCheese(PizzaDecorator):
    def cost(self) -> float:
        return self._pizza.cost() + 1.50

    def ingredients(self) -> list:
        return self._pizza.ingredients() + ["extra cheese"]


class Pepperoni(PizzaDecorator):
    def cost(self) -> float:
        return self._pizza.cost() + 2.00

    def ingredients(self) -> list:
        return self._pizza.ingredients() + ["pepperoni"]


class Mushrooms(PizzaDecorator):
    def cost(self) -> float:
        return self._pizza.cost() + 1.25

    def ingredients(self) -> list:
        return self._pizza.ingredients() + ["mushrooms"]


pizza = Mushrooms(Pepperoni(ExtraCheese(Margherita())))
print(f"Ingredients: {', '.join(pizza.ingredients())}")
print(f"Total: ${pizza.cost():.2f}")
```

## Intermediate Examples

```python
# HTTP request/response decorators
from typing import Dict, Any, List, Callable
import json


class HTTPRequest(ABC):
    @abstractmethod
    def get(self, url: str, headers: Dict[str, str] = None) -> Dict[str, Any]:
        pass

    @abstractmethod
    def post(self, url: str, data: Any = None, headers: Dict[str, str] = None) -> Dict[str, Any]:
        pass


class SimpleHTTPClient(HTTPRequest):
    def get(self, url: str, headers: Dict[str, str] = None) -> Dict[str, Any]:
        return {"status": 200, "body": f"GET response from {url}", "headers": headers or {}}

    def post(self, url: str, data: Any = None, headers: Dict[str, str] = None) -> Dict[str, Any]:
        return {"status": 201, "body": f"POST response from {url}", "data": data}


class HTTPDecorator(HTTPRequest, ABC):
    def __init__(self, client: HTTPRequest):
        self._client = client


class LoggingDecorator(HTTPDecorator):
    def get(self, url: str, headers: Dict[str, str] = None) -> Dict[str, Any]:
        print(f"[LOG] GET {url}")
        result = self._client.get(url, headers)
        print(f"[LOG] Response: {result['status']}")
        return result

    def post(self, url: str, data: Any = None, headers: Dict[str, str] = None) -> Dict[str, Any]:
        print(f"[LOG] POST {url}")
        result = self._client.post(url, data, headers)
        print(f"[LOG] Response: {result['status']}")
        return result


class RetryDecorator(HTTPDecorator):
    def __init__(self, client: HTTPRequest, max_retries: int = 3):
        super().__init__(client)
        self._max_retries = max_retries

    def get(self, url: str, headers: Dict[str, str] = None) -> Dict[str, Any]:
        for attempt in range(self._max_retries):
            try:
                return self._client.get(url, headers)
            except Exception as e:
                if attempt == self._max_retries - 1:
                    raise
                print(f"[RETRY] Attempt {attempt + 1} failed: {e}")

    def post(self, url: str, data: Any = None, headers: Dict[str, str] = None) -> Dict[str, Any]:
        for attempt in range(self._max_retries):
            try:
                return self._client.post(url, data, headers)
            except Exception as e:
                if attempt == self._max_retries - 1:
                    raise
                print(f"[RETRY] Attempt {attempt + 1} failed: {e}")


class CacheDecorator(HTTPDecorator):
    def __init__(self, client: HTTPRequest):
        super().__init__(client)
        self._cache = {}

    def get(self, url: str, headers: Dict[str, str] = None) -> Dict[str, Any]:
        cache_key = f"GET:{url}:{json.dumps(headers or {}, sort_keys=True)}"
        if cache_key in self._cache:
            print(f"[CACHE] Hit for GET {url}")
            return self._cache[cache_key]
        result = self._client.get(url, headers)
        self._cache[cache_key] = result
        return result

    def post(self, url: str, data: Any = None, headers: Dict[str, str] = None) -> Dict[str, Any]:
        return self._client.post(url, data, headers)


class RateLimitDecorator(HTTPDecorator):
    def __init__(self, client: HTTPRequest, max_calls: int = 10, period: float = 1.0):
        super().__init__(client)
        self._max_calls = max_calls
        self._period = period
        self._calls = []
        import time
        self._time = time

    def get(self, url: str, headers: Dict[str, str] = None) -> Dict[str, Any]:
        self._check_rate_limit()
        return self._client.get(url, headers)

    def post(self, url: str, data: Any = None, headers: Dict[str, str] = None) -> Dict[str, Any]:
        self._check_rate_limit()
        return self._client.post(url, data, headers)

    def _check_rate_limit(self):
        now = self._time.time()
        self._calls = [t for t in self._calls if now - t < self._period]
        if len(self._calls) >= self._max_calls:
            raise Exception("Rate limit exceeded")
        self._calls.append(now)


client = SimpleHTTPClient()
client = LoggingDecorator(client)
client = RetryDecorator(client, max_retries=2)
client = CacheDecorator(client)
client = RateLimitDecorator(client, max_calls=5)

result = client.get("https://api.example.com/users")
print(result)
result = client.get("https://api.example.com/users")
```

```python
# Stream processing decorators
from typing import Iterable, Any, Callable


class StreamProcessor(ABC):
    @abstractmethod
    def process(self, data: Iterable) -> Iterable:
        pass


class IdentityStream(StreamProcessor):
    def process(self, data: Iterable) -> Iterable:
        return data


class StreamDecorator(StreamProcessor, ABC):
    def __init__(self, processor: StreamProcessor):
        self._processor = processor


class FilterDecorator(StreamDecorator):
    def __init__(self, processor: StreamProcessor, predicate: Callable):
        super().__init__(processor)
        self._predicate = predicate

    def process(self, data: Iterable) -> Iterable:
        return filter(self._predicate, self._processor.process(data))


class MapDecorator(StreamDecorator):
    def __init__(self, processor: StreamProcessor, transform: Callable):
        super().__init__(processor)
        self._transform = transform

    def process(self, data: Iterable) -> Iterable:
        return map(self._transform, self._processor.process(data))


class LimitDecorator(StreamDecorator):
    def __init__(self, processor: StreamProcessor, limit: int):
        super().__init__(processor)
        self._limit = limit

    def process(self, data: Iterable) -> Iterable:
        count = 0
        for item in self._processor.process(data):
            if count >= self._limit:
                break
            yield item
            count += 1


class DebugDecorator(StreamDecorator):
    def __init__(self, processor: StreamProcessor, label: str = "debug"):
        super().__init__(processor)
        self._label = label

    def process(self, data: Iterable) -> Iterable:
        for item in self._processor.process(data):
            print(f"[{self._label}] {item}")
            yield item


pipeline = IdentityStream()
pipeline = FilterDecorator(pipeline, lambda x: x > 0)
pipeline = MapDecorator(pipeline, lambda x: x * 2)
pipeline = LimitDecorator(pipeline, 5)
pipeline = DebugDecorator(pipeline, "result")

result = list(pipeline.process([-3, -2, -1, 0, 1, 2, 3, 4, 5, 6]))
print(f"Final: {result}")
```

## Advanced Examples

```python
# Data validation and sanitization decorators
from typing import Any, Dict, Optional
import re
import json


class DataProcessor(ABC):
    @abstractmethod
    def process(self, data: Dict[str, Any]) -> Dict[str, Any]:
        pass


class BaseProcessor(DataProcessor):
    def process(self, data: Dict[str, Any]) -> Dict[str, Any]:
        return dict(data)


class ProcessorDecorator(DataProcessor, ABC):
    def __init__(self, processor: DataProcessor):
        self._processor = processor


class ValidationDecorator(ProcessorDecorator):
    def __init__(self, processor: DataProcessor, rules: Dict[str, Any]):
        super().__init__(processor)
        self._rules = rules

    def process(self, data: Dict[str, Any]) -> Dict[str, Any]:
        errors = []
        for field, rule in self._rules.items():
            value = data.get(field)
            if rule.get("required") and value is None:
                errors.append(f"{field} is required")
            if value is not None:
                if "min_length" in rule and len(str(value)) < rule["min_length"]:
                    errors.append(f"{field} too short")
                if "max_length" in rule and len(str(value)) > rule["max_length"]:
                    errors.append(f"{field} too long")
                if "pattern" in rule and not re.match(rule["pattern"], str(value)):
                    errors.append(f"{field} format invalid")
                if "type" in rule and not isinstance(value, rule["type"]):
                    errors.append(f"{field} must be {rule['type'].__name__}")
        if errors:
            raise ValueError(f"Validation failed: {', '.join(errors)}")
        return self._processor.process(data)


class SanitizationDecorator(ProcessorDecorator):
    def __init__(self, processor: DataProcessor, sanitizers: Dict[str, Callable]):
        super().__init__(processor)
        self._sanitizers = sanitizers

    def process(self, data: Dict[str, Any]) -> Dict[str, Any]:
        cleaned = dict(data)
        for field, sanitizer in self._sanitizers.items():
            if field in cleaned:
                cleaned[field] = sanitizer(cleaned[field])
        return self._processor.process(cleaned)


class LoggingDecorator(ProcessorDecorator):
    def process(self, data: Dict[str, Any]) -> Dict[str, Any]:
        result = self._processor.process(data)
        print(f"Processed: {json.dumps(result, default=str)}")
        return result


class TimingDecorator(ProcessorDecorator):
    def process(self, data: Dict[str, Any]) -> Dict[str, Any]:
        import time
        start = time.time()
        result = self._processor.process(data)
        elapsed = time.time() - start
        print(f"Processing took {elapsed:.4f}s")
        return result


def strip_whitespace(value):
    return value.strip() if isinstance(value, str) else value


rules = {
    "email": {"required": True, "pattern": r"^[^@]+@[^@]+\.[^@]+$"},
    "name": {"required": True, "min_length": 2, "max_length": 100},
    "age": {"type": int, "min_length": 1},
}

sanitizers = {
    "email": lambda x: x.lower().strip(),
    "name": strip_whitespace,
}

processor = BaseProcessor()
processor = ValidationDecorator(processor, rules)
processor = SanitizationDecorator(processor, sanitizers)
processor = LoggingDecorator(processor)
processor = TimingDecorator(processor)

data = {"email": " Alice@Example.COM ", "name": "  Alice  ", "age": 30}
result = processor.process(data)
print(result)
```

```python
# Decorator for API response formatting
class APIResponse(ABC):
    @abstractmethod
    def render(self) -> Dict[str, Any]:
        pass


class UserResponse(APIResponse):
    def __init__(self, user: dict):
        self._user = user

    def render(self) -> Dict[str, Any]:
        return {"id": self._user["id"], "name": self._user["name"]}


class APIResponseDecorator(APIResponse, ABC):
    def __init__(self, response: APIResponse):
        self._response = response


class PaginationDecorator(APIResponseDecorator):
    def __init__(self, response: APIResponse, page: int, per_page: int, total: int):
        super().__init__(response)
        self._page = page
        self._per_page = per_page
        self._total = total

    def render(self) -> Dict[str, Any]:
        data = self._response.render()
        return {
            "data": data,
            "pagination": {
                "page": self._page,
                "per_page": self._per_page,
                "total": self._total,
                "total_pages": (self._total + self._per_page - 1) // self._per_page,
            },
        }


class MetadataDecorator(APIResponseDecorator):
    def __init__(self, response: APIResponse, metadata: dict):
        super().__init__(response)
        self._metadata = metadata

    def render(self) -> Dict[str, Any]:
        data = self._response.render()
        data["_metadata"] = self._metadata
        return data


class ErrorWrapperDecorator(APIResponseDecorator):
    def render(self) -> Dict[str, Any]:
        try:
            return self._response.render()
        except Exception as e:
            return {"error": str(e), "status": "error"}


response = UserResponse({"id": 1, "name": "Alice"})
response = PaginationDecorator(response, page=1, per_page=10, total=42)
response = MetadataDecorator(response, {"request_id": "abc-123", "timestamp": "2024-01-01T00:00:00Z"})
print(json.dumps(response.render(), indent=2))
```

```python
# Decorator for authentication/authorization layers
class SecureComponent(ABC):
    @abstractmethod
    def execute(self, user: str, action: str) -> str:
        pass


class FileManager(SecureComponent):
    def execute(self, user: str, action: str) -> str:
        return f"Executed '{action}' for {user}"


class SecurityDecorator(SecureComponent, ABC):
    def __init__(self, component: SecureComponent):
        self._component = component


class AuthenticationDecorator(SecurityDecorator):
    def __init__(self, component: SecureComponent, valid_users: set):
        super().__init__(component)
        self._valid_users = valid_users

    def execute(self, user: str, action: str) -> str:
        if user not in self._valid_users:
            raise PermissionError(f"User '{user}' not authenticated")
        return self._component.execute(user, action)


class AuthorizationDecorator(SecurityDecorator):
    def __init__(self, component: SecureComponent, permissions: dict):
        super().__init__(component)
        self._permissions = permissions

    def execute(self, user: str, action: str) -> str:
        allowed_actions = self._permissions.get(user, set())
        if action not in allowed_actions:
            raise PermissionError(f"User '{user}' not allowed to '{action}'")
        return self._component.execute(user, action)


class AuditDecorator(SecurityDecorator):
    def execute(self, user: str, action: str) -> str:
        result = self._component.execute(user, action)
        print(f"[AUDIT] {user} performed '{action}': {result}")
        return result


class RateLimitDecorator(SecurityDecorator):
    def __init__(self, component: SecureComponent, max_per_minute: int = 10):
        super().__init__(component)
        self._limits = {}
        self._max = max_per_minute

    def execute(self, user: str, action: str) -> str:
        import time
        now = time.time()
        if user not in self._limits:
            self._limits[user] = []
        self._limits[user] = [t for t in self._limits[user] if now - t < 60]
        if len(self._limits[user]) >= self._max:
            raise Exception(f"Rate limit exceeded for {user}")
        self._limits[user].append(now)
        return self._component.execute(user, action)


component = FileManager()
component = AuthenticationDecorator(component, {"alice", "bob"})
component = AuthorizationDecorator(component, {"alice": {"read", "write", "delete"}, "bob": {"read"}})
component = AuditDecorator(component)
component = RateLimitDecorator(component, max_per_minute=100)

print(component.execute("alice", "read"))
print(component.execute("bob", "read"))
```

## Real-World Use Cases

```python
# Middleware pipeline for web framework
class Middleware(ABC):
    @abstractmethod
    def handle(self, request: dict, next_handler: Callable) -> dict:
        pass


class CORSMiddleware(Middleware):
    def handle(self, request: dict, next_handler: Callable) -> dict:
        response = next_handler(request)
        response["headers"] = response.get("headers", {})
        response["headers"]["Access-Control-Allow-Origin"] = "*"
        return response


class LoggingMiddleware(Middleware):
    def handle(self, request: dict, next_handler: Callable) -> dict:
        print(f"Request: {request['method']} {request['path']}")
        response = next_handler(request)
        print(f"Response: {response['status']}")
        return response


class JSONMiddleware(Middleware):
    def handle(self, request: dict, next_handler: Callable) -> dict:
        if request.get("content-type") == "application/json":
            request["body"] = json.loads(request.get("body", "{}"))
        response = next_handler(request)
        if isinstance(response.get("body"), dict):
            response["body"] = json.dumps(response["body"])
            response["headers"] = response.get("headers", {})
            response["headers"]["Content-Type"] = "application/json"
        return response


class AuthMiddleware(Middleware):
    def __init__(self, valid_tokens: set):
        self._valid_tokens = valid_tokens

    def handle(self, request: dict, next_handler: Callable) -> dict:
        token = request.get("headers", {}).get("Authorization", "")
        if token not in self._valid_tokens:
            return {"status": 401, "body": "Unauthorized"}
        return next_handler(request)


class MiddlewareStack:
    def __init__(self, handler: Callable):
        self._handler = handler
        self._middleware: List[Middleware] = []

    def use(self, middleware: Middleware):
        self._middleware.append(middleware)

    def execute(self, request: dict) -> dict:
        chain = self._handler
        for mw in reversed(self._middleware):
            prev = chain
            chain = lambda req, mw=mw, prev=prev: mw.handle(req, prev)
        return chain(request)


def app_handler(request: dict) -> dict:
    return {"status": 200, "body": f"Hello from {request['path']}"}


app = MiddlewareStack(app_handler)
app.use(LoggingMiddleware())
app.use(JSONMiddleware())
app.use(CORSMiddleware())
app.use(AuthMiddleware({"Bearer token123"}))

response = app.execute({
    "method": "GET",
    "path": "/users",
    "headers": {"Authorization": "Bearer token123", "Content-Type": "application/json"},
    "body": "{}",
})
print(response)
```

```python
# Decorator for machine learning model pipeline
class MLPipeline(ABC):
    @abstractmethod
    def predict(self, data: Any) -> Any:
        pass


class BaseModel(MLPipeline):
    def predict(self, data: Any) -> Any:
        return data


class PipelineDecorator(MLPipeline, ABC):
    def __init__(self, pipeline: MLPipeline):
        self._pipeline = pipeline


class Normalizer(PipelineDecorator):
    def predict(self, data: Any) -> Any:
        normalized = self._normalize(data)
        return self._pipeline.predict(normalized)

    def _normalize(self, data):
        if isinstance(data, (list, tuple)):
            max_val = max(abs(x) for x in data)
            return [x / max_val for x in data] if max_val else data
        return data


class FeatureExtractor(PipelineDecorator):
    def __init__(self, pipeline: MLPipeline, features: List[str]):
        super().__init__(pipeline)
        self._features = features

    def predict(self, data: Any) -> Any:
        if isinstance(data, dict):
            extracted = {k: v for k, v in data.items() if k in self._features}
            return self._pipeline.predict(extracted)
        return self._pipeline.predict(data)


class ThresholdClassifier(PipelineDecorator):
    def __init__(self, pipeline: MLPipeline, threshold: float = 0.5):
        super().__init__(pipeline)
        self._threshold = threshold

    def predict(self, data: Any) -> Any:
        result = self._pipeline.predict(data)
        if isinstance(result, (int, float)):
            return result >= self._threshold
        return result


class ExplanationDecorator(PipelineDecorator):
    def predict(self, data: Any) -> Any:
        result = self._pipeline.predict(data)
        return {"prediction": result, "input": data, "pipeline": "normalizer->features->classifier"}


pipeline = BaseModel()
pipeline = Normalizer(pipeline)
pipeline = FeatureExtractor(pipeline, ["age", "income", "score"])
pipeline = ThresholdClassifier(pipeline, threshold=0.6)
pipeline = ExplanationDecorator(pipeline)

sample = {"age": 30, "income": 75000, "score": 8.5, "name": "Alice"}
print(pipeline.predict(sample))
```

## Common Mistakes

```python
# MISTAKE 1: Confusing Python @decorator with structural decorator
# @decorator is a language feature for functions/methods
# Structural decorator wraps objects at runtime
```

```python
# MISTAKE 2: Breaking the component interface
class BadDecorator:
    def __init__(self, component):
        self._component = component

    def extra_method(self):
        pass
```

```python
# MISTAKE 3: Decorator order dependence
# When ordering matters, document the expected sequence
```

```python
# MISTAKE 4: Modifying wrapped object state directly
class StatefulDecorator:
    def __init__(self, component):
        self._component = component
        self._component.state = "modified"
```

## Best Practices

```python
# 1. Maintain the component interface (no extra methods)
# 2. Keep decorators focused on one concern
# 3. Use ABCs to enforce interface contracts
# 4. Document decorator ordering requirements
# 5. Consider composability
```

```python
# Best practice: Typed decorator with Protocol
from typing import Protocol

class Component(Protocol):
    def operation(self) -> str: ...
```

## Interview Questions

```python
# Q1: Structural Decorator vs Python @decorator?
# Structural: wraps objects, runtime, preserves interface
# @decorator: wraps functions, syntactic sugar, may change signature
```

```python
# Q2: Decorator vs inheritance for extending behavior?
# Decorator: runtime, composable, single responsibility
# Inheritance: compile-time, rigid hierarchy, class explosion
```

```python
# Q3: How to implement multiple decorators stacking?
# Each wraps the previous; outer decorator runs first on entry
```

```python
# Q4: When would you NOT use the decorator pattern?
# When wrapping adds too much complexity
# When simpler composition (like list of functions) suffices
```

## Coding Challenges

```python
# Challenge 1: Implement a caching decorator for
# expensive computations with TTL and LRU eviction
```

```python
# Challenge 2: Build a retry decorator with
# exponential backoff and jitter
```

```python
# Challenge 3: Create a circuit breaker decorator
# that stops calls after N failures, then retries
```

```python
# Challenge 4: Implement a decorator for input
# validation and output formatting in an API
```

```python
# Challenge 5: Build a logging decorator that
# supports structured logging with correlation IDs
```

## Summary

The structural Decorator pattern wraps objects dynamically to add behavior at runtime without modifying their interface. It provides a flexible alternative to subclassing for extending functionality, supporting the Open/Closed Principle. Python's implementation follows the classic GoF pattern with ABC interfaces and wrapper classes. The pattern is widely used in middleware pipelines, stream processing, input/output streams, and cross-cutting concerns like logging, caching, authentication, and rate limiting.

## Related Topics

- Proxy Pattern: Similar structure but controls access rather than adding behavior
- Adapter Pattern: Changes interface; decorator preserves it
- Composite Pattern: Can be combined with decorator for tree structures
- Strategy Pattern: Alternative for varying behavior
- Python @decorator: Language-level function decoration
