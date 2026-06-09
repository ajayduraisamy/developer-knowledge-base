# Strategy - Interchangeable algorithms, dependency injection

## Introduction

The Strategy pattern defines a family of interchangeable algorithms, encapsulates each one, and makes them interchangeable. It lets the algorithm vary independently from the clients that use it. This behavioral pattern is fundamental for writing flexible, maintainable code where behavior can be selected at runtime.

## Why It Is Important

Strategy eliminates conditional statements for behavior selection, promotes composition over inheritance, and makes it easy to add new algorithms without modifying existing code. It is essential for implementing different business rules, validation strategies, sorting methods, compression algorithms, routing logic, and any situation where multiple approaches to the same problem exist.

## Syntax

In Python, strategies can be implemented as classes (following the classic pattern), first-class functions (taking advantage of Python's functional features), or callable objects. The context class holds a reference to the strategy and delegates execution to it.

## Examples

```python
# Class-based Strategy pattern
from abc import ABC, abstractmethod
from typing import List, Any


class SortStrategy(ABC):
    @abstractmethod
    def sort(self, data: List[Any]) -> List[Any]:
        pass


class BubbleSort(SortStrategy):
    def sort(self, data: List[Any]) -> List[Any]:
        arr = list(data)
        n = len(arr)
        for i in range(n):
            for j in range(0, n - i - 1):
                if arr[j] > arr[j + 1]:
                    arr[j], arr[j + 1] = arr[j + 1], arr[j]
        return arr


class QuickSort(SortStrategy):
    def sort(self, data: List[Any]) -> List[Any]:
        arr = list(data)
        if len(arr) <= 1:
            return arr
        pivot = arr[len(arr) // 2]
        left = [x for x in arr if x < pivot]
        middle = [x for x in arr if x == pivot]
        right = [x for x in arr if x > pivot]
        return self.sort(left) + middle + self.sort(right)


class MergeSort(SortStrategy):
    def sort(self, data: List[Any]) -> List[Any]:
        arr = list(data)
        if len(arr) <= 1:
            return arr
        mid = len(arr) // 2
        left = self.sort(arr[:mid])
        right = self.sort(arr[mid:])
        return self._merge(left, right)

    def _merge(self, left, right):
        result = []
        i, j = 0, 0
        while i < len(left) and j < len(right):
            if left[i] <= right[j]:
                result.append(left[i])
                i += 1
            else:
                result.append(right[j])
                j += 1
        result.extend(left[i:])
        result.extend(right[j:])
        return result


class Sorter:
    def __init__(self, strategy: SortStrategy = None):
        self._strategy = strategy or BubbleSort()

    @property
    def strategy(self) -> SortStrategy:
        return self._strategy

    @strategy.setter
    def strategy(self, strategy: SortStrategy):
        self._strategy = strategy

    def sort(self, data: List[Any]) -> List[Any]:
        return self._strategy.sort(data)


sorter = Sorter()
data = [64, 34, 25, 12, 22, 11, 90]

print(sorter.sort(data))  # Bubble sort by default

sorter.strategy = QuickSort()
print(sorter.sort(data))

sorter.strategy = MergeSort()
print(sorter.sort(data))
```

```python
# Function-based strategy (first-class functions)
from typing import List, Callable


def bubble_sort(data: List) -> List:
    arr = list(data)
    n = len(arr)
    for i in range(n):
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
    return arr


def quick_sort(data: List) -> List:
    arr = list(data)
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    return quick_sort(left) + middle + quick_sort(right)


def python_sort(data: List) -> List:
    return sorted(data)


class SorterFn:
    def __init__(self, strategy: Callable = None):
        self._strategy = strategy or python_sort

    def set_strategy(self, strategy: Callable):
        self._strategy = strategy

    def sort(self, data: List) -> List:
        return self._strategy(data)


sorter = SorterFn()
print(sorter.sort([3, 1, 4, 1, 5]))
sorter.set_strategy(quick_sort)
print(sorter.sort([3, 1, 4, 1, 5]))
```

```python
# Strategy with dependency injection
from dataclasses import dataclass
from typing import Protocol


class TaxCalculator(Protocol):
    def calculate(self, amount: float) -> float:
        pass


class USATax:
    def calculate(self, amount: float) -> float:
        return amount * 0.07


class UKTax:
    def calculate(self, amount: float) -> float:
        return amount * 0.20


class NoTax:
    def calculate(self, amount: float) -> float:
        return 0.0


@dataclass
class Product:
    name: str
    price: float


class Checkout:
    def __init__(self, tax_calculator: TaxCalculator):
        self._tax = tax_calculator
        self._items: list = []

    def add_item(self, product: Product):
        self._items.append(product)

    def total(self) -> float:
        subtotal = sum(item.price for item in self._items)
        tax = self._tax.calculate(subtotal)
        return subtotal + tax


checkout = Checkout(USATax())
checkout.add_item(Product("Laptop", 999.99))
checkout.add_item(Product("Mouse", 29.99))
print(f"Total: ${checkout.total():.2f}")
```

## Beginner Examples

```python
# Payment method strategies
from abc import ABC, abstractmethod


class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount: float) -> str:
        pass


class CreditCardPayment(PaymentStrategy):
    def __init__(self, card_number: str, cvv: str):
        self.card_number = card_number[-4:]
        self.cvv = cvv

    def pay(self, amount: float) -> str:
        return f"Paid ${amount:.2f} via Credit Card ending in {self.card_number}"


class PayPalPayment(PaymentStrategy):
    def __init__(self, email: str):
        self.email = email

    def pay(self, amount: float) -> str:
        return f"Paid ${amount:.2f} via PayPal ({self.email})"


class CryptoPayment(PaymentStrategy):
    def __init__(self, wallet_address: str):
        self.wallet_address = wallet_address

    def pay(self, amount: float) -> str:
        return f"Paid ${amount:.2f} via Crypto ({self.wallet_address[:8]}...)"


class ShoppingCart:
    def __init__(self):
        self._items = []
        self._payment_strategy = None

    def add_item(self, item: str, price: float):
        self._items.append((item, price))

    def set_payment_method(self, strategy: PaymentStrategy):
        self._payment_strategy = strategy

    def checkout(self) -> str:
        total = sum(price for _, price in self._items)
        return self._payment_strategy.pay(total)


cart = ShoppingCart()
cart.add_item("Book", 19.99)
cart.add_item("Pen", 2.99)
cart.set_payment_method(CreditCardPayment("1234567890123456", "123"))
print(cart.checkout())
```

```python
# Simple text formatting strategies
class TextFormatter:
    def format(self, text: str) -> str:
        return text


class UpperCaseFormatter(TextFormatter):
    def format(self, text: str) -> str:
        return text.upper()


class LowerCaseFormatter(TextFormatter):
    def format(self, text: str) -> str:
        return text.lower()


class TitleCaseFormatter(TextFormatter):
    def format(self, text: str) -> str:
        return text.title()


class ReverseFormatter(TextFormatter):
    def format(self, text: str) -> str:
        return text[::-1]


class TextProcessor:
    def __init__(self, formatter: TextFormatter = None):
        self._formatter = formatter or TextFormatter()

    def set_formatter(self, formatter: TextFormatter):
        self._formatter = formatter

    def process(self, text: str) -> str:
        return self._formatter.format(text)


processor = TextProcessor()
print(processor.process("hello world"))
processor.set_formatter(UpperCaseFormatter())
print(processor.process("hello world"))
processor.set_formatter(TitleCaseFormatter())
print(processor.process("hello world"))
```

## Intermediate Examples

```python
# Compression strategies with file handling
import os
import shutil
from abc import ABC, abstractmethod
from typing import List


class CompressionStrategy(ABC):
    @abstractmethod
    def compress(self, files: List[str], output: str) -> str:
        pass

    @abstractmethod
    def decompress(self, archive: str, output_dir: str) -> List[str]:
        pass


class ZipCompression(CompressionStrategy):
    def compress(self, files: List[str], output: str) -> str:
        import zipfile
        output_path = f"{output}.zip"
        with zipfile.ZipFile(output_path, "w", zipfile.ZIP_DEFLATED) as zf:
            for file in files:
                zf.write(file, os.path.basename(file))
        return output_path

    def decompress(self, archive: str, output_dir: str) -> List[str]:
        import zipfile
        with zipfile.ZipFile(archive, "r") as zf:
            zf.extractall(output_dir)
            return zf.namelist()


class TarGzCompression(CompressionStrategy):
    def compress(self, files: List[str], output: str) -> str:
        import tarfile
        output_path = f"{output}.tar.gz"
        with tarfile.open(output_path, "w:gz") as tf:
            for file in files:
                tf.add(file, os.path.basename(file))
        return output_path

    def decompress(self, archive: str, output_dir: str) -> List[str]:
        import tarfile
        with tarfile.open(archive, "r:gz") as tf:
            tf.extractall(output_dir)
            return tf.getnames()


class RarCompression(CompressionStrategy):
    def compress(self, files: List[str], output: str) -> str:
        output_path = f"{output}.rar"
        # Simulating rar compression
        for file in files:
            shutil.copy(file, output_dir)
        return output_path

    def decompress(self, archive: str, output_dir: str) -> List[str]:
        return ["file1.txt", "file2.txt"]


class ArchiveManager:
    def __init__(self, strategy: CompressionStrategy = None):
        self._strategy = strategy or ZipCompression()

    def set_strategy(self, strategy: CompressionStrategy):
        self._strategy = strategy

    def create_archive(self, files: List[str], output: str) -> str:
        return self._strategy.compress(files, output)

    def extract_archive(self, archive: str, output_dir: str) -> List[str]:
        return self._strategy.decompress(archive, output_dir)


manager = ArchiveManager()
manager.set_strategy(TarGzCompression())
```

```python
# Validation strategies
from typing import Any, List, Tuple


class ValidationResult:
    def __init__(self):
        self.errors: List[str] = []
        self.warnings: List[str] = []

    @property
    def is_valid(self) -> bool:
        return len(self.errors) == 0

    def __bool__(self) -> bool:
        return self.is_valid


class Validator(ABC):
    @abstractmethod
    def validate(self, data: Any) -> ValidationResult:
        pass


class RequiredFieldValidator(Validator):
    def __init__(self, field_name: str):
        self.field_name = field_name

    def validate(self, data: dict) -> ValidationResult:
        result = ValidationResult()
        if self.field_name not in data or data[self.field_name] is None:
            result.errors.append(f"{self.field_name} is required")
        elif isinstance(data[self.field_name], str) and not data[self.field_name].strip():
            result.errors.append(f"{self.field_name} cannot be empty")
        return result


class EmailValidator(Validator):
    def validate(self, data: dict) -> ValidationResult:
        result = ValidationResult()
        email = data.get("email", "")
        if "@" not in email or "." not in email.split("@")[-1]:
            result.errors.append("Invalid email format")
        return result


class RangeValidator(Validator):
    def __init__(self, field_name: str, min_val: float = None, max_val: float = None):
        self.field_name = field_name
        self.min_val = min_val
        self.max_val = max_val

    def validate(self, data: dict) -> ValidationResult:
        result = ValidationResult()
        value = data.get(self.field_name)
        if value is not None:
            if self.min_val is not None and value < self.min_val:
                result.errors.append(f"{self.field_name} must be >= {self.min_val}")
            if self.max_val is not None and value > self.max_val:
                result.errors.append(f"{self.field_name} must be <= {self.max_val}")
        return result


class ValidatorComposite:
    def __init__(self):
        self._validators: List[Validator] = []

    def add(self, validator: Validator):
        self._validators.append(validator)

    def validate(self, data: dict) -> ValidationResult:
        combined = ValidationResult()
        for v in self._validators:
            result = v.validate(data)
            combined.errors.extend(result.errors)
            combined.warnings.extend(result.warnings)
        return combined


validator = ValidatorComposite()
validator.add(RequiredFieldValidator("email"))
validator.add(EmailValidator())
validator.add(RangeValidator("age", min_val=18, max_val=120))

result = validator.validate({"email": "invalid", "age": 15})
print(result.is_valid)
print(result.errors)
```

## Advanced Examples

```python
# Strategy selection with ML-like model routing
import random
from dataclasses import dataclass
from typing import Any, Dict, Optional


class ModelStrategy(ABC):
    @abstractmethod
    def predict(self, features: Dict[str, float]) -> float:
        pass

    @property
    @abstractmethod
    def name(self) -> str:
        pass


class LinearModel(ModelStrategy):
    @property
    def name(self) -> str:
        return "LinearRegression"

    def predict(self, features: Dict[str, float]) -> float:
        coefficients = {"x1": 0.5, "x2": -0.3, "x3": 1.2}
        return sum(features.get(k, 0) * v for k, v in coefficients.items())


class RandomForestModel(ModelStrategy):
    @property
    def name(self) -> str:
        return "RandomForest"

    def predict(self, features: Dict[str, float]) -> float:
        base = sum(features.values()) / max(len(features), 1)
        return base + random.uniform(-0.1, 0.1)


class NeuralNetworkModel(ModelStrategy):
    @property
    def name(self) -> str:
        return "NeuralNetwork"

    def predict(self, features: Dict[str, float]) -> float:
        import math
        weighted_sum = sum(features.get(k, 0) * math.sin(k.__hash__() % 10)
                           for k in features)
        return 1 / (1 + math.exp(-weighted_sum))


@dataclass
class ModelMetrics:
    accuracy: float
    latency_ms: float
    last_used: float


class ModelRouter:
    def __init__(self):
        self._models: Dict[str, ModelStrategy] = {}
        self._metrics: Dict[str, ModelMetrics] = {}

    def register(self, model: ModelStrategy):
        self._models[model.name] = model
        self._metrics[model.name] = ModelMetrics(accuracy=0.0, latency_ms=0.0, last_used=0.0)

    def select_strategy(self, features: Dict[str, float]) -> ModelStrategy:
        if len(features) < 5:
            return self._models["LinearRegression"]
        elif len(features) < 20:
            return self._models["RandomForest"]
        else:
            return self._models["NeuralNetwork"]

    def predict(self, features: Dict[str, float]) -> tuple:
        strategy = self.select_strategy(features)
        result = strategy.predict(features)
        return result, strategy.name


router = ModelRouter()
router.register(LinearModel())
router.register(RandomForestModel())
router.register(NeuralNetworkModel())

features_small = {"x1": 1.5, "x2": 2.0}
features_large = {f"f{i}": random.random() for i in range(50)}

result, name = router.predict(features_small)
print(f"Model: {name}, Result: {result:.4f}")
result2, name2 = router.predict(features_large)
print(f"Model: {name2}, Result: {result2:.4f}")
```

```python
# Dynamic strategy composition (pipeline)
from typing import List, Callable, Any


class Pipeline:
    def __init__(self):
        self._steps: List[tuple] = []

    def add_step(self, name: str, strategy: Callable, **kwargs):
        self._steps.append((name, strategy, kwargs))

    def execute(self, data: Any) -> dict:
        results = {}
        current = data
        for name, strategy, params in self._steps:
            try:
                current = strategy(current, **params)
                results[name] = {"status": "success", "output": current}
            except Exception as e:
                results[name] = {"status": "error", "error": str(e)}
                break
        return {"final_output": current, "steps": results}


def validate_data(data: dict, required_fields: list = None) -> dict:
    if required_fields:
        for field in required_fields:
            if field not in data:
                raise ValueError(f"Missing required field: {field}")
    return data


def transform_data(data: dict, mapping: dict = None) -> dict:
    if mapping:
        return {mapping.get(k, k): v for k, v in data.items()}
    return data


def enrich_data(data: dict, enrichments: dict = None) -> dict:
    if enrichments:
        data.update(enrichments)
    return data


def filter_fields(data: dict, allowed: list = None) -> dict:
    if allowed:
        return {k: v for k, v in data.items() if k in allowed}
    return data


pipeline = Pipeline()
pipeline.add_step("validate", validate_data, required_fields=["name", "email"])
pipeline.add_step("transform", transform_data, mapping={"name": "user_name", "email": "user_email"})
pipeline.add_step("enrich", enrich_data, enrichments={"source": "api", "version": "2.0"})
pipeline.add_step("filter", filter_fields, allowed=["user_name", "user_email", "source"])

result = pipeline.execute({"name": "Alice", "email": "alice@example.com", "age": 30})
print(result["final_output"])
```

```python
# Strategy with state and configuration
from typing import Dict, Any, Optional
from pathlib import Path


class ExportStrategy(ABC):
    @abstractmethod
    def export(self, data: Any, path: Path, config: Dict[str, Any]) -> Path:
        pass

    @abstractmethod
    def supports_format(self, fmt: str) -> bool:
        pass


class JSONExportStrategy(ExportStrategy):
    def export(self, data: Any, path: Path, config: Dict[str, Any]) -> Path:
        import json
        output_path = path.with_suffix(".json")
        indent = config.get("indent", 2)
        with open(output_path, "w") as f:
            json.dump(data, f, indent=indent)
        return output_path

    def supports_format(self, fmt: str) -> bool:
        return fmt in ("json", "JSON")


class CSVExportStrategy(ExportStrategy):
    def export(self, data: Any, path: Path, config: Dict[str, Any]) -> Path:
        import csv
        output_path = path.with_suffix(".csv")
        if not isinstance(data, list):
            data = [data]
        with open(output_path, "w", newline="") as f:
            if data:
                writer = csv.DictWriter(f, fieldnames=data[0].keys())
                writer.writeheader()
                writer.writerows(data)
        return output_path

    def supports_format(self, fmt: str) -> bool:
        return fmt in ("csv", "CSV")


class XMLExportStrategy(ExportStrategy):
    def export(self, data: Any, path: Path, config: Dict[str, Any]) -> Path:
        output_path = path.with_suffix(".xml")
        with open(output_path, "w") as f:
            f.write("<data>\n")
            if isinstance(data, dict):
                self._write_dict(data, f, 1)
            else:
                f.write(f"  {data}\n")
            f.write("</data>\n")
        return output_path

    def _write_dict(self, d: dict, f, level: int):
        indent = "  " * level
        for k, v in d.items():
            f.write(f"{indent}<{k}>")
            if isinstance(v, dict):
                f.write("\n")
                self._write_dict(v, f, level + 1)
                f.write(f"{indent}</{k}>\n")
            else:
                f.write(f"{v}</{k}>\n")

    def supports_format(self, fmt: str) -> bool:
        return fmt in ("xml", "XML")


class DataExporter:
    def __init__(self):
        self._strategies: Dict[str, ExportStrategy] = {}

    def register(self, strategy: ExportStrategy):
        self._strategies[strategy.__class__.__name__] = strategy

    def export(self, data: Any, path: str, format: str, config: Dict[str, Any] = None) -> Path:
        config = config or {}
        for strategy in self._strategies.values():
            if strategy.supports_format(format):
                return strategy.export(data, Path(path), config)
        raise ValueError(f"No strategy supports format: {format}")


exporter = DataExporter()
exporter.register(JSONExportStrategy())
exporter.register(CSVExportStrategy())

data = [{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]
result_path = exporter.export(data, "output", "csv")
print(f"Exported to {result_path}")
```

## Real-World Use Cases

```python
# Authentication strategies
class AuthStrategy(ABC):
    @abstractmethod
    def authenticate(self, credentials: dict) -> bool:
        pass

    @abstractmethod
    def get_user_info(self, credentials: dict) -> dict:
        pass


class OAuth2Strategy(AuthStrategy):
    def authenticate(self, credentials: dict) -> bool:
        token = credentials.get("token")
        return token is not None and len(token) > 10

    def get_user_info(self, credentials: dict) -> dict:
        return {
            "provider": "oauth2",
            "email": "user@example.com",
            "name": "OAuth User",
        }


class APIKeyStrategy(AuthStrategy):
    def authenticate(self, credentials: dict) -> bool:
        api_key = credentials.get("api_key")
        return api_key == "valid-api-key-12345"

    def get_user_info(self, credentials: dict) -> dict:
        return {
            "provider": "api_key",
            "name": "API User",
            "role": "developer",
        }


class JWTAuthStrategy(AuthStrategy):
    def authenticate(self, credentials: dict) -> bool:
        import base64
        token = credentials.get("jwt", "")
        try:
            parts = token.split(".")
            if len(parts) == 3:
                payload = base64.b64decode(parts[1] + "==")
                return True
        except Exception:
            pass
        return False

    def get_user_info(self, credentials: dict) -> dict:
        return {"provider": "jwt", "email": "jwt@example.com"}


class Authenticator:
    def __init__(self, strategy: AuthStrategy = None):
        self._strategy = strategy

    def set_strategy(self, strategy: AuthStrategy):
        self._strategy = strategy

    def login(self, credentials: dict) -> dict:
        if self._strategy.authenticate(credentials):
            user = self._strategy.get_user_info(credentials)
            return {"success": True, "user": user}
        return {"success": False, "error": "Authentication failed"}


auth = Authenticator()
auth.set_strategy(OAuth2Strategy())
print(auth.login({"token": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIn0"}))
```

```python
# Logging formatters as strategies
import datetime
from typing import Dict


class LogFormatter(ABC):
    @abstractmethod
    def format(self, level: str, message: str, context: Dict[str, any]) -> str:
        pass


class PlainTextFormatter(LogFormatter):
    def format(self, level: str, message: str, context: Dict[str, any]) -> str:
        timestamp = datetime.datetime.now().isoformat()
        return f"[{timestamp}] [{level}] {message}"


class JSONFormatter(LogFormatter):
    def format(self, level: str, message: str, context: Dict[str, any]) -> str:
        import json
        record = {
            "timestamp": datetime.datetime.now().isoformat(),
            "level": level,
            "message": message,
            "context": context,
        }
        return json.dumps(record)


class ColoredFormatter(LogFormatter):
    COLORS = {
        "ERROR": "\033[91m",
        "WARN": "\033[93m",
        "INFO": "\033[94m",
        "DEBUG": "\033[90m",
        "RESET": "\033[0m",
    }

    def format(self, level: str, message: str, context: Dict[str, any]) -> str:
        timestamp = datetime.datetime.now().isoformat()
        color = self.COLORS.get(level, self.COLORS["RESET"])
        reset = self.COLORS["RESET"]
        return f"{color}[{timestamp}] [{level}] {message}{reset}"


class Logger:
    def __init__(self, formatter: LogFormatter = None):
        self._formatter = formatter or PlainTextFormatter()

    def set_formatter(self, formatter: LogFormatter):
        self._formatter = formatter

    def log(self, level: str, message: str, **context):
        output = self._formatter.format(level, message, context)
        print(output)


logger = Logger(JSONFormatter())
logger.log("INFO", "Application started", service="api", version="1.0")
```

## Common Mistakes

```python
# MISTAKE 1: Strategy with too many methods (not cohesive)
class BloatStrategy(ABC):
    @abstractmethod
    def compute(self): pass
    @abstractmethod
    def validate(self): pass
    @abstractmethod
    def format(self): pass
    @abstractmethod
    def persist(self): pass  # Too many responsibilities
```

```python
# MISTAKE 2: Switching strategies inside the strategy
class BadStrategy(ABC):
    def execute(self, data):
        if isinstance(data, str):
            return self._handle_string(data)
        elif isinstance(data, list):
            return self._handle_list(data)
        # Strategy should focus on ONE algorithm
```

```python
# MISTAKE 3: Storing state in strategies
class StatefulStrategy:
    def __init__(self):
        self.cache = {}  # Strategies should be stateless

    def execute(self, data):
        self.cache[id(data)] = data  # Side effect!
```

## Best Practices

```python
# 1. Keep strategies stateless (reusable across contexts)
# 2. Use protocols/ABCs for strategy interfaces
# 3. Prefer function-based strategies for simple cases
# 4. Register strategies in a registry for dynamic selection
# 5. Use dependency injection for strategy selection
# 6. Document the strategy contract clearly
```

```python
# Best practice: Protocol for duck typing
from typing import Protocol

class Compressor(Protocol):
    def compress(self, data: bytes) -> bytes: ...
    def decompress(self, data: bytes) -> bytes: ...
```

## Interview Questions

```python
# Q1: Strategy vs State pattern?
# Strategy: interchangeable algorithms (how to do something)
# State: behavior changes based on internal state
```

```python
# Q2: How does Strategy compare to inheritance?
# Strategy: composition, runtime flexibility
# Inheritance: compile-time, less flexible
```

```python
# Q3: When to use function-based vs class-based strategy?
# Function: simple, single-method strategies
# Class: complex strategies with configuration
```

```python
# Q4: How to select strategies dynamically?
# Registry dict, factory method, configuration file
```

## Coding Challenges

```python
# Challenge 1: Implement routing strategies
# (shortest path, fastest time, scenic route, toll-free)
```

```python
# Challenge 2: Build a caching strategy pattern
# (LRU, LFU, TTL-based, FIFO, random eviction)
```

```python
# Challenge 3: Create rate-limiting strategies
# (token bucket, leaky bucket, sliding window, fixed window)
```

```python
# Challenge 4: Implement notification delivery strategies
# (email, SMS, push, in-app) with retry logic
```

```python
# Challenge 5: Build a data migration strategy system
# (full migration, incremental, rolling, blue-green)
```

## Summary

The Strategy pattern encapsulates interchangeable algorithms behind a common interface. Python supports both class-based and function-based strategies, with the latter being more concise for simple cases. The pattern eliminates conditional logic, supports runtime algorithm selection, and makes systems extensible without modification. Key considerations include keeping strategies stateless, defining clear interfaces, and using dependency injection for strategy selection.

## Related Topics

- Factory Pattern: Often used to create strategy objects
- State Pattern: Similar structure, different intent
- Command Pattern: Encapsulates operations as objects
- Template Method: Related but uses inheritance
- Dependency Injection: Common way to provide strategies
- First-Class Functions: Enable strategy without classes
