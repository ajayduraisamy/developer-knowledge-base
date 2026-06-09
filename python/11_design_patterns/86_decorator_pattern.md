# Decorator Pattern - Structural wrapper, adding behavior dynamically
## Introduction
The Decorator pattern attaches additional responsibilities to an object dynamically. It provides a flexible alternative to subclassing for extending functionality. In Python, the decorator pattern manifests in two distinct forms: the classic Gang of Four structural pattern (wrapping objects) and Python's language-level decorator syntax (`@decorator`). Both forms enable clean, composable extension of behavior without modifying existing code.

## Structural wrapper
### What It Is
A structural wrapper (the classic Decorator pattern) wraps an object with one or more decorator objects that add behavior before, after, or around the wrapped object's methods. Multiple decorators can be stacked, creating a chain of responsibility for behavior augmentation.

### Why It Is Important
The structural wrapper pattern enables adding features to objects at runtime without affecting other objects of the same class. It avoids the "class explosion" problem where every combination of features requires a new subclass.

### How It Works Internally
Decorators implement the same interface as the component they wrap. Each decorator holds a reference to a component (which could be another decorator). When a method is called, the decorator adds its behavior and delegates to the wrapped component. This recursion continues through the entire decorator chain.

```python
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
        return 2.0

    def description(self) -> str:
        return "Simple coffee"

class CoffeeDecorator(Coffee):
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
        return self._coffee.cost() + 0.5

    def description(self) -> str:
        return f"{self._coffee.description()}, milk"

class SugarDecorator(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 0.25

    def description(self) -> str:
        return f"{self._coffee.description()}, sugar"

class WhippedCreamDecorator(CoffeeDecorator):
    def cost(self) -> float:
        return self._coffee.cost() + 0.75

    def description(self) -> str:
        return f"{self._coffee.description()}, whipped cream"

# Usage
coffee = SimpleCoffee()
print(f"{coffee.description()}: ${coffee.cost():.2f}")

coffee_with_milk = MilkDecorator(coffee)
print(f"{coffee_with_milk.description()}: ${coffee_with_milk.cost():.2f}")

coffee_with_milk_and_sugar = SugarDecorator(MilkDecorator(coffee))
print(f"{coffee_with_milk_and_sugar.description()}: ${coffee_with_milk_and_sugar.cost():.2f}")

full_coffee = WhippedCreamDecorator(SugarDecorator(MilkDecorator(coffee)))
print(f"{full_coffee.description()}: ${full_coffee.cost():.2f}")
```

### Data Processing Pipeline
```python
from abc import ABC, abstractmethod
from typing import Dict

class DataProcessor(ABC):
    @abstractmethod
    def process(self, data: Dict) -> Dict:
        pass

class BaseProcessor(DataProcessor):
    def process(self, data: Dict) -> Dict:
        return data

class ProcessorDecorator(DataProcessor):
    def __init__(self, processor: DataProcessor):
        self._processor = processor

class ValidationDecorator(ProcessorDecorator):
    def process(self, data: Dict) -> Dict:
        if not data.get("name"):
            raise ValueError("Name is required")
        if "age" in data and data["age"] < 0:
            raise ValueError("Age cannot be negative")
        return self._processor.process(data)

class NormalizationDecorator(ProcessorDecorator):
    def process(self, data: Dict) -> Dict:
        if "name" in data:
            data["name"] = data["name"].strip().title()
        if "email" in data:
            data["email"] = data["email"].strip().lower()
        return self._processor.process(data)

class LoggingDecorator(ProcessorDecorator):
    def process(self, data: Dict) -> Dict:
        print(f"Processing: {data}")
        result = self._processor.process(data)
        print(f"Result: {result}")
        return result

class EnrichmentDecorator(ProcessorDecorator):
    def process(self, data: Dict) -> Dict:
        data = self._processor.process(data)
        if "full_name" not in data and "first_name" in data:
            data["full_name"] = f"{data['first_name']} {data.get('last_name', '')}".strip()
        return data

# Usage
pipeline = LoggingDecorator(
    ValidationDecorator(
        NormalizationDecorator(
            EnrichmentDecorator(
                BaseProcessor()
            )
        )
    )
)

result = pipeline.process({
    "name": "  alice  ",
    "email": "Alice@Example.COM",
    "age": 30
})
print(result)
```

## Dynamic behavior addition
### What It Is
Dynamic behavior addition allows extending an object's functionality at runtime without changing its class. This is the core purpose of the Decorator pattern - adding responsibilities to individual objects, not entire classes.

### Why It Is Important
Dynamic decoration enables flexible, composable architectures where features can be mixed and matched at runtime. It supports configuration-driven behavior, AOP-like cross-cutting concerns, and runtime customization.

### How It Works Internally
Decorators intercept method calls and can modify arguments, execute pre/post processing, cache results, or entirely replace behavior. The decorated object is unaware of the decoration, maintaining encapsulation.

```python
from functools import wraps
import time
import logging

# Function decorators (Python's built-in syntax)
def timing_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

def retry_decorator(max_retries=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}")
                    time.sleep(delay)
            return None
        return wrapper
    return decorator

def log_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        logging.debug(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        logging.debug(f"{func.__name__} returned {result}")
        return result
    return wrapper

# Usage
@timing_decorator
@retry_decorator(max_retries=2)
def fetch_data(url):
    import requests
    response = requests.get(url, timeout=5)
    response.raise_for_status()
    return response.json()

@log_decorator
def process_order(order_id, items):
    return {"order_id": order_id, "status": "processed", "items": items}
```

### Class Decorators
```python
def singleton(cls):
    instances = {}

    @wraps(cls)
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get_instance

def add_repr(cls):
    def __repr__(self):
        items = ", ".join(f"{k}={v!r}" for k, v in self.__dict__.items())
        return f"{cls.__name__}({items})"
    cls.__repr__ = __repr__
    return cls

@singleton
class Database:
    def __init__(self):
        print("Creating database instance")
        self.connected = False

    def connect(self):
        self.connected = True
        print("Connected")

@add_repr
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

db1 = Database()
db2 = Database()
assert db1 is db2  # True, singleton

p = Point(3, 4)
print(p)  # Point(x=3, y=4)
```

### Decorator with Arguments
```python
from functools import wraps
import logging

def log_level(level=logging.INFO):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            logger = logging.getLogger(func.__module__)
            logger.log(level, f"Calling {func.__name__}")
            result = func(*args, **kwargs)
            logger.log(level, f"{func.__name__} completed")
            return result
        return wrapper
    return decorator

def rate_limit(max_calls=10, period=60):
    def decorator(func):
        calls = []

        @wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()
            calls[:] = [c for c in calls if c > now - period]
            if len(calls) >= max_calls:
                raise RuntimeError(f"Rate limit exceeded: {max_calls} calls per {period}s")
            calls.append(now)
            return func(*args, **kwargs)
        return wrapper
    return decorator

@log_level(logging.WARNING)
@rate_limit(max_calls=5, period=10)
def api_call(endpoint):
    print(f"Calling {endpoint}")
    return {"status": "ok"}
```

### Decorator Pattern with __getattr__
```python
class Wrapper:
    def __init__(self, obj):
        self._obj = obj

    def __getattr__(self, name):
        return getattr(self._obj, name)

    def extra_method(self):
        return "Extra functionality"

class MeasuredObject(Wrapper):
    def __init__(self, obj):
        super().__init__(obj)
        self._call_counts = {}

    def __getattr__(self, name):
        attr = super().__getattr__(name)
        if callable(attr):
            def counted(*args, **kwargs):
                self._call_counts[name] = self._call_counts.get(name, 0) + 1
                return attr(*args, **kwargs)
            return counted
        return attr

    def get_stats(self):
        return dict(self._call_counts)

class Worker:
    def work(self, item):
        return f"Processed {item}"

    def validate(self, item):
        return len(item) > 0

worker = MeasuredObject(Worker())
worker.work("task1")
worker.work("task2")
worker.validate("test")
print(worker.get_stats())  # {'work': 2, 'validate': 1}
```

### Real-World Use Cases
- Adding logging, timing, or caching to functions
- Input validation and sanitization
- Authentication and authorization
- Rate limiting and throttling
- Retry logic for unreliable services
- Transaction management for databases
- Serialization/deserialization wrappers
- GUI component styling (toolkits like Java Swing)

### Common Mistakes
- Decorator ordering (order matters when stacking decorators)
- Breaking function metadata (fix with `@wraps`)
- Decorators that don't handle all function types (methods, generators, async)
- Stateful decorators sharing state across instances
- Over-decoration making code hard to trace

### Best Practices
- Use `functools.wraps` to preserve function metadata
- Apply multiple decorators in the correct order (bottom-up execution)
- Keep decorators focused on single responsibility
- Consider if a simple function call is clearer than decoration
- Document decorator behavior, especially ordering requirements

### Performance Considerations
- Each decorator layer adds function call overhead
- Decorator chains increase call stack depth
- Class-based decorators have more overhead than function-based
- Consider `functools.lru_cache` or `functools.cache` for memoization decorators

### Interview Questions
1. How does Python's `@decorator` syntax work internally?
2. What's the difference between function and class decorators?
3. Explain the ordering when stacking multiple decorators.
4. How do you create a decorator with arguments?
5. How does the structural Decorator pattern differ from Python's decorator syntax?

### Coding Challenges
1. Create a `@memoize` decorator that caches function results
2. Build a `@validate` decorator that checks argument types
3. Implement a `@timeout` decorator that raises after N seconds
4. Create a class decorator that adds observable properties to a class

### Related Topics
- Proxy pattern
- Adapter pattern
- Chain of Responsibility
- AOP (Aspect-Oriented Programming)
- functools module
- Context managers (alternative to decorators for resource management)
