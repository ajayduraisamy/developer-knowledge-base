# Strategy - Interchangeable algorithms, dependency injection
## Introduction
The Strategy pattern enables selecting an algorithm's implementation at runtime. It defines a family of interchangeable algorithms, encapsulates each one, and makes them interchangeable within a context object. This pattern promotes composition over inheritance and adheres to the Open/Closed principle. Python's first-class functions and duck typing make Strategy implementation exceptionally clean, often reducing the pattern to simple function passing or dictionary lookups.

## Interchangeable algorithms
### What It Is
Strategy defines a family of algorithms, encapsulates each one in a separate class (or function), and makes them interchangeable. The context object delegates algorithm execution to a strategy object rather than implementing the algorithm itself.

### Why It Is Important
Strategy eliminates conditional statements for algorithm selection, makes algorithms independently testable, and allows adding new algorithms without modifying existing code. It's ideal for situations where multiple algorithms achieve the same goal with different trade-offs.

### How It Works Internally
The pattern has three components: the Context (uses a strategy), the Strategy interface (common protocol), and Concrete Strategies (algorithm implementations). The context holds a reference to a strategy and delegates to it. Clients configure the context with the desired strategy.

```python
from abc import ABC, abstractmethod
from typing import List

class SortingStrategy(ABC):
    @abstractmethod
    def sort(self, data: List[int]) -> List[int]:
        pass

class BubbleSort(SortingStrategy):
    def sort(self, data: List[int]) -> List[int]:
        arr = data.copy()
        n = len(arr)
        for i in range(n):
            for j in range(0, n - i - 1):
                if arr[j] > arr[j + 1]:
                    arr[j], arr[j + 1] = arr[j + 1], arr[j]
        return arr

class QuickSort(SortingStrategy):
    def sort(self, data: List[int]) -> List[int]:
        if len(data) <= 1:
            return data
        pivot = data[0]
        left = [x for x in data[1:] if x <= pivot]
        right = [x for x in data[1:] if x > pivot]
        return self.sort(left) + [pivot] + self.sort(right)

class MergeSort(SortingStrategy):
    def sort(self, data: List[int]) -> List[int]:
        if len(data) <= 1:
            return data
        mid = len(data) // 2
        left = self.sort(data[:mid])
        right = self.sort(data[mid:])
        return self._merge(left, right)

    def _merge(self, left, right):
        result = []
        i = j = 0
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
    def __init__(self, strategy: SortingStrategy):
        self._strategy = strategy

    @property
    def strategy(self):
        return self._strategy

    @strategy.setter
    def strategy(self, strategy: SortingStrategy):
        self._strategy = strategy

    def sort(self, data: List[int]) -> List[int]:
        return self._strategy.sort(data)

# Usage
data = [64, 34, 25, 12, 22, 11, 90]

sorter = Sorter(BubbleSort())
print(f"Bubble sort: {sorter.sort(data)}")

sorter.strategy = QuickSort()
print(f"Quick sort: {sorter.sort(data)}")

sorter.strategy = MergeSort()
print(f"Merge sort: {sorter.sort(data)}")
```

### Strategy with Functions (Pythonic)
```python
from typing import List, Callable

# Strategies as simple functions
def bubble_sort(data: List[int]) -> List[int]:
    arr = data.copy()
    n = len(arr)
    for i in range(n):
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
    return arr

def quick_sort(data: List[int]) -> List[int]:
    if len(data) <= 1:
        return data
    pivot = data[0]
    left = [x for x in data[1:] if x <= pivot]
    right = [x for x in data[1:] if x > pivot]
    return quick_sort(left) + [pivot] + quick_sort(right)

class Sorter:
    def __init__(self, strategy: Callable = bubble_sort):
        self.strategy = strategy

    def sort(self, data: List[int]) -> List[int]:
        return self.strategy(data)

# Usage
sorter = Sorter(bubble_sort)
print(sorter.sort([3, 1, 4, 1, 5]))

sorter.strategy = quick_sort
print(sorter.sort([3, 1, 4, 1, 5]))
```

### Compression Strategies
```python
from abc import ABC, abstractmethod
import gzip, bz2, lzma

class CompressionStrategy(ABC):
    @abstractmethod
    def compress(self, data: bytes) -> bytes:
        pass

    @abstractmethod
    def decompress(self, data: bytes) -> bytes:
        pass

class GZipCompression(CompressionStrategy):
    def compress(self, data: bytes) -> bytes:
        return gzip.compress(data)

    def decompress(self, data: bytes) -> bytes:
        return gzip.decompress(data)

class BZip2Compression(CompressionStrategy):
    def compress(self, data: bytes) -> bytes:
        return bz2.compress(data)

    def decompress(self, data: bytes) -> bytes:
        return bz2.decompress(data)

class LZMACompression(CompressionStrategy):
    def compress(self, data: bytes) -> bytes:
        return lzma.compress(data)

    def decompress(self, data: bytes) -> bytes:
        return lzma.decompress(data)

class Compressor:
    def __init__(self, strategy: CompressionStrategy):
        self.strategy = strategy

    def save(self, filename: str, data: bytes):
        compressed = self.strategy.compress(data)
        with open(filename, 'wb') as f:
            f.write(compressed)

    def load(self, filename: str) -> bytes:
        with open(filename, 'rb') as f:
            compressed = f.read()
        return self.strategy.decompress(compressed)

# Usage
data = b"Hello World! " * 1000
compressor = Compressor(GZipCompression())
compressor.save("test.gz", data)
restored = compressor.load("test.gz")
assert data == restored
```

## Dependency injection
### What It Is
Dependency injection (DI) is a technique where objects receive their dependencies from external sources rather than creating them internally. The Strategy pattern is a form of dependency injection where algorithms are injected into a context. DI decouples object creation from object usage.

### Why It Is Important
DI makes code more testable, flexible, and maintainable. Dependencies can be swapped without modifying the dependent class. It enables mocking in tests and configuration-based behavior changes.

### How It Works Internally
Dependencies are passed to the object through its constructor (constructor injection), setter methods (setter injection), or method parameters (method injection). The client or a DI container is responsible for wiring dependencies together.

```python
from abc import ABC, abstractmethod

class PaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount: float) -> bool:
        pass

class StripeGateway(PaymentGateway):
    def charge(self, amount: float) -> bool:
        print(f"Charging ${amount} via Stripe")
        return True

class PayPalGateway(PaymentGateway):
    def charge(self, amount: float) -> bool:
        print(f"Charging ${amount} via PayPal")
        return True

class InvoiceService:
    def __init__(self, payment_gateway: PaymentGateway):
        self._gateway = payment_gateway

    def create_and_charge(self, customer: str, amount: float):
        print(f"Invoice for {customer}: ${amount}")
        return self._gateway.charge(amount)

class TaxService:
    def __init__(self, tax_calculator):
        self._calculator = tax_calculator

    def calculate_tax(self, amount: float) -> float:
        return self._calculator.calculate(amount)

class SalesEngine:
    def __init__(self, invoice: InvoiceService, tax: TaxService):
        self._invoice = invoice
        self._tax = tax

    def process_sale(self, customer: str, amount: float):
        tax = self._tax.calculate_tax(amount)
        total = amount + tax
        return self._invoice.create_and_charge(customer, total)

# Constructor injection
gateway = StripeGateway()
invoice = InvoiceService(gateway)
tax_service = TaxService(lambda x: x * 0.1)
engine = SalesEngine(invoice, tax_service)

engine.process_sale("Alice Corp", 1000.0)
```

### Setter Injection
```python
class ReportGenerator:
    def __init__(self):
        self._formatter = None

    def set_formatter(self, formatter):
        self._formatter = formatter

    def generate(self, data):
        if not self._formatter:
            raise RuntimeError("Formatter not set")
        return self._formatter.format(data)

class JSONFormatter:
    def format(self, data):
        import json
        return json.dumps(data, indent=2)

class HTMLFormatter:
    def format(self, data):
        html = "<table>\n"
        for item in data:
            html += f"  <tr><td>{item}</td></tr>\n"
        html += "</table>"
        return html

# Usage
report = ReportGenerator()
report.set_formatter(JSONFormatter())
print(report.generate([1, 2, 3]))
```

### Method Injection
```python
class DataProcessor:
    def process(self, data, strategy=None):
        if strategy is None:
            strategy = self._default_strategy
        return strategy(data)

    @staticmethod
    def _default_strategy(data):
        return [x * 2 for x in data]

# Custom strategies
def reverse_strategy(data):
    return list(reversed(data))

def square_strategy(data):
    return [x ** 2 for x in data]

# Usage
processor = DataProcessor()
print(processor.process([1, 2, 3]))                    # Default: [2, 4, 6]
print(processor.process([1, 2, 3], reverse_strategy))  # [3, 2, 1]
print(processor.process([1, 2, 3], square_strategy))   # [1, 4, 9]
```

### DI Container
```python
class DIContainer:
    def __init__(self):
        self._services = {}
        self._instances = {}

    def register(self, name, factory, singleton=False):
        self._services[name] = {"factory": factory, "singleton": singleton}

    def resolve(self, name):
        if name in self._instances:
            return self._instances[name]

        service = self._services.get(name)
        if not service:
            raise KeyError(f"Service '{name}' not registered")

        instance = service["factory"]()
        if service["singleton"]:
            self._instances[name] = instance
        return instance

# Usage
container = DIContainer()

container.register("config", lambda: {"db_url": "sqlite:///test.db"}, singleton=True)
container.register("db", lambda: Database(container.resolve("config")), singleton=True)
container.register("user_repo", lambda: UserRepository(container.resolve("db")))

repo = container.resolve("user_repo")
```

### Real-World Use Cases
- Payment processing (multiple gateways)
- Authentication methods (OAuth, JWT, Basic)
- Data serialization (JSON, XML, Protobuf)
- File storage (local, S3, GCS)
- Notification delivery (email, SMS, push)
- Caching backends (Redis, Memcached, in-memory)
- Logging frameworks (file, database, cloud)

### Common Mistakes
- Hard-coding algorithm selection with if-elif chains
- Making strategy objects stateful (they should be stateless)
- Not considering that simple functions may suffice instead of classes
- Over-engineering with too many small strategy classes
- Forgetting to handle the case where no strategy is set

### Best Practices
- Use functions for simple strategies (most Pythonic)
- Use classes when strategies need internal state
- Keep strategy interfaces minimal (Single Responsibility)
- Consider default strategies for fallback behavior
- Use dependency injection for strategy selection

### Performance Considerations
- Function call overhead for strategy dispatch is negligible
- Object creation per strategy invocation can add overhead; reuse strategy instances
- Strategy pattern may add indirection, but modern JIT compilers handle this well
- Consider caching strategy selection results

### Interview Questions
1. How does Strategy pattern differ from the Command pattern?
2. When would you use functions vs classes for strategies in Python?
3. Explain how Strategy enables dependency injection.
4. What are the trade-offs of using Strategy vs conditional statements?
5. How do you dynamically select strategies at runtime?

### Coding Challenges
1. Implement a text formatter with uppercase, lowercase, and title case strategies
2. Create a route calculator with different transportation strategies (car, bike, walking)
3. Build a validator that uses different validation strategies
4. Implement a caching system with pluggable cache strategies

### Related Topics
- Command pattern
- State pattern
- Template Method pattern
- Dependency injection
- Functional programming (higher-order functions)
- Open/Closed Principle
