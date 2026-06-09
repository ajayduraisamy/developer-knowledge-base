# Static and Class Methods - @staticmethod, @classmethod

## Introduction

Static methods and class methods are two special method types in Python that don't operate on instances. @staticmethod defines methods that belong to a class but don't need access to the class or instance. @classmethod defines methods that receive the class as the first argument, enabling alternative constructors and class-level operations. Understanding when and how to use each is essential for writing idiomatic Python.

## @staticmethod

### What It Is

A static method is a method that belongs to a class but doesn't receive an implicit first argument (neither self nor cls). It behaves like a plain function but lives in the class's namespace.

`python
class MathUtils:
    @staticmethod
    def add(a, b):
        return a + b
`

### Why It Is Important

Static methods group utility functions that are logically related to a class but don't need access to class or instance state. They improve code organization by keeping related functions namespaced within the class, and they can be inherited and overridden.

### How It Works Internally

@staticmethod wraps a function in a staticmethod descriptor. When accessed on an instance or class, it returns the underlying function directly with no binding and no automatic argument injection.

`python
MathUtils.add(1, 2)   # Just calls the function
obj = MathUtils()
obj.add(1, 2)         # Still just calls the function (instance ignored)
`

### Syntax

`python
class ClassName:
    @staticmethod
    def method_name(args):
        pass

ClassName.method_name(args)
instance.method_name(args)
`

### Beginner Examples

`python
class StringUtils:
    @staticmethod
    def is_palindrome(text):
        cleaned = text.lower().replace(" ", "")
        return cleaned == cleaned[::-1]

    @staticmethod
    def reverse_words(text):
        return " ".join(text.split()[::-1])

    @staticmethod
    def count_vowels(text):
        return sum(1 for ch in text.lower() if ch in "aeiou")

print(StringUtils.is_palindrome("A man a plan a canal Panama"))  # True
print(StringUtils.reverse_words("hello world"))                  # world hello
print(StringUtils.count_vowels("Hello World"))                   # 3
`

### Intermediate Examples

`python
import re
from typing import List

class Validator:
    @staticmethod
    def email(email_str: str) -> bool:
        pattern = r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
        return bool(re.match(pattern, email_str))

    @staticmethod
    def phone(phone_str: str) -> bool:
        cleaned = re.sub(r"[\s\-\(\)]", "", phone_str)
        return cleaned.isdigit() and len(cleaned) >= 10

    @staticmethod
    def zip_code(code: str) -> bool:
        return bool(re.match(r"^\d{5}(-\d{4})?$", code))

    @staticmethod
    def validate_all(data: dict) -> List[str]:
        errors = []
        if "email" in data and not Validator.email(data["email"]):
            errors.append("Invalid email")
        if "phone" in data and not Validator.phone(data["phone"]):
            errors.append("Invalid phone")
        if "zip" in data and not Validator.zip_code(data["zip"]):
            errors.append("Invalid zip code")
        return errors

print(Validator.email("user@example.com"))  # True
print(Validator.phone("(555) 123-4567"))    # True
`

### Advanced Examples

`python
import json
from typing import Any, Dict

class JSONProcessor:
    @staticmethod
    def minify(json_str: str) -> str:
        return json.dumps(json.loads(json_str), separators=(",", ":"))

    @staticmethod
    def pretty(json_str: str, indent: int = 2) -> str:
        return json.dumps(json.loads(json_str), indent=indent)

    @staticmethod
    def validate(json_str: str) -> bool:
        try:
            json.loads(json_str)
            return True
        except json.JSONDecodeError:
            return False

    @staticmethod
    def flatten(data: Dict[str, Any], parent_key: str = "", sep: str = ".") -> Dict[str, Any]:
        items = []
        for key, value in data.items():
            new_key = f"{parent_key}{sep}{key}" if parent_key else key
            if isinstance(value, dict):
                items.extend(JSONProcessor.flatten(value, new_key, sep).items())
            else:
                items.append((new_key, value))
        return dict(items)

    @staticmethod
    def diff(left: Dict[str, Any], right: Dict[str, Any]) -> Dict[str, Any]:
        result = {}
        all_keys = set(left) | set(right)
        for key in all_keys:
            if key not in left:
                result[key] = {"added": right[key]}
            elif key not in right:
                result[key] = {"removed": left[key]}
            elif left[key] != right[key]:
                result[key] = {"old": left[key], "new": right[key]}
        return result

class BaseFormatter:
    @staticmethod
    def format(value):
        return str(value)

class UpperFormatter(BaseFormatter):
    @staticmethod
    def format(value):
        return str(value).upper()

class LowerFormatter(BaseFormatter):
    @staticmethod
    def format(value):
        return str(value).lower()

def process(formatter_cls, values):
    return [formatter_cls.format(v) for v in values]

print(process(UpperFormatter, ["hello", "world"]))
print(process(LowerFormatter, ["Hello", "World"]))
`

### Real-World Use Cases

- **Utility functions**: validation, formatting, encoding helpers
- **Factory helper methods**: simple object creation without class context
- **Conversion methods**: Temperature.celsius_to_fahrenheit(100)
- **Configuration defaults**: Config.default_values() returns a dict
- **Math/statistics functions**: grouped under a related class

### Common Mistakes

1. **Using @staticmethod when a module-level function suffices**
2. **Using @staticmethod when @classmethod is needed**
3. **Referring to the class by name inside a static method**: breaks with inheritance
4. **Overusing static methods**: module-level functions are often simpler

### Best Practices

- Use static methods for utility functions related to the class
- Consider module-level functions if there's no strong grouping reason
- Static methods can be inherited — use this for strategy-like patterns
- Prefer @classmethod if subclasses might need to override behavior

### Performance Considerations

- Static methods avoid the self/cls parameter overhead
- Slightly faster than instance methods (no binding step)
- Identical performance to module-level functions

## @classmethod

### What It Is

A class method is a method that receives the class (cls) as its first argument instead of the instance (self). It can access and modify class state and serves as an alternative constructor or factory method.

`python
class MyClass:
    @classmethod
    def factory(cls, arg):
        return cls(arg)
`

### Why It Is Important

Class methods are essential for polymorphic construction — they ensure that when a subclass calls the factory, it creates an instance of the subclass, not the parent. They also provide access to class-level state and configuration.

### How It Works Internally

@classmethod wraps the function in a classmethod descriptor. When accessed, it binds the function to the class (or the instance's class) and passes the class as the first argument (cls).

`python
obj = MyClass.factory("value")
# Internally: cls = MyClass, calls factory(MyClass, "value")
`

### Syntax

`python
class ClassName:
    @classmethod
    def method_name(cls, args):
        pass

ClassName.method_name(args)
instance.method_name(args)
`

### Beginner Examples

`python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author

    @classmethod
    def from_string(cls, book_str):
        title, author = book_str.split(" by ")
        return cls(title.strip(), author.strip())

    @classmethod
    def default_library(cls):
        return cls("Unknown Title", "Unknown Author")

book = Book.from_string("1984 by George Orwell")
print(book.title)   # 1984
print(book.author)  # George Orwell

default = Book.default_library()
print(default.title)  # Unknown Title
`

### Intermediate Examples

`python
import json
from datetime import datetime

class User:
    def __init__(self, username, email, created_at=None):
        self.username = username
        self.email = email
        self.created_at = created_at or datetime.utcnow()
        self.is_active = True

    @classmethod
    def from_dict(cls, data: dict) -> "User":
        return cls(
            username=data["username"],
            email=data["email"],
            created_at=datetime.fromisoformat(data["created_at"]) if "created_at" in data else None
        )

    @classmethod
    def from_json(cls, json_str: str) -> "User":
        data = json.loads(json_str)
        return cls.from_dict(data)

    @classmethod
    def anonymous(cls, prefix: str = "guest") -> "User":
        import uuid
        unique_id = uuid.uuid4().hex[:8]
        return cls(
            username=f"{prefix}_{unique_id}",
            email=f"{prefix}_{unique_id}@anonymous.local"
        )

data = '{"username": "alice", "email": "alice@example.com", "created_at": "2024-01-15T10:30:00"}'
user = User.from_json(data)
print(user.username)    # alice

anon = User.anonymous()
print(anon.username.startswith("guest_"))  # True
`

### Advanced Examples

`python
from enum import Enum
from typing import Optional, Dict, List, Type

class ConnectionType(Enum):
    SQLITE = "sqlite"
    POSTGRES = "postgres"
    MYSQL = "mysql"

class DatabaseConnection:
    _pool: Dict[str, "DatabaseConnection"] = {}
    _default_config: Dict[str, str] = {}

    def __init__(self, url: str):
        self.url = url
        self._connected = False

    def connect(self):
        self._connected = True
        print(f"Connected to {self.url}")

    def disconnect(self):
        self._connected = False

    @classmethod
    def set_default_config(cls, config: Dict[str, str]) -> None:
        cls._default_config = config

    @classmethod
    def from_url(cls, url: str) -> "DatabaseConnection":
        if url in cls._pool:
            return cls._pool[url]
        instance = cls(url)
        cls._pool[url] = instance
        return instance

    @classmethod
    def from_config(cls, config: Dict[str, str]) -> "DatabaseConnection":
        url = f"{config['type']}://{config['host']}:{config['port']}/{config['database']}"
        return cls.from_url(url)

    @classmethod
    def sqlite(cls, path: str) -> "DatabaseConnection":
        return cls.from_url(f"sqlite:///{path}")

    @classmethod
    def postgres(cls, host: str, database: str, user: str, password: str) -> "DatabaseConnection":
        return cls.from_url(f"postgres://{user}:{password}@{host}/{database}")

    @classmethod
    def connection_count(cls) -> int:
        return len(cls._pool)

    @classmethod
    def disconnect_all(cls) -> None:
        for conn in cls._pool.values():
            conn.disconnect()
        cls._pool.clear()

# Polymorphic constructors with inheritance
class PooledConnection(DatabaseConnection):
    def __init__(self, url: str, pool_size: int = 10):
        super().__init__(url)
        self.pool_size = pool_size

    @classmethod
    def from_url(cls, url: str) -> "PooledConnection":
        instance = cls(url)
        return instance

conn = DatabaseConnection.sqlite(":memory:")
conn2 = DatabaseConnection.postgres("localhost", "mydb", "admin", "secret")
print(DatabaseConnection.connection_count())  # 2
DatabaseConnection.disconnect_all()

pc = PooledConnection.from_url("sqlite:///pooled.db")
print(type(pc).__name__)  # PooledConnection (polymorphism works!)
`

### Real-World Use Cases

- **Alternative constructors**: datetime.fromtimestamp(), dict.fromkeys()
- **Factory methods**: creating objects from various input formats
- **Class-level registry**: tracking all instances or subclasses
- **Configuration management**: Config.from_env(), Config.from_file()
- **Singleton pattern**: @classmethod to return shared instance

### Common Mistakes

1. **Forgetting the cls parameter**
2. **Hardcoding the class name inside a classmethod instead of using cls**
3. **Using @classmethod when @staticmethod suffices**
4. **Modifying class state from an instance method without @classmethod**

### Best Practices

- Use cls not self as the first parameter name
- Use classmethods for alternative constructors (most common use case)
- Use cls to create instances so inheritance works correctly
- Document the return type with -> "ClassName" forward reference
- Combine with class variables for configuration and defaults

### Performance Considerations

- Class methods are slightly slower than static methods (binding overhead)
- The cls parameter binding happens once at access time
- Overhead is negligible for all practical purposes

## When to use each

### What It Is

Choosing between @staticmethod, @classmethod, and regular instance methods depends on what the method needs to access. The hierarchy is: instance method (needs self) > classmethod (needs cls) > staticmethod (needs neither).

### Why It Is Important

Using the right method type communicates intent clearly and prevents subtle bugs. Instance methods imply the method operates on instance state. Class methods imply the method operates on class state or is an alternative constructor. Static methods imply a utility function grouped with the class.

### How It Works Internally

| Method type | First param | Bound to | Access to instance | Access to class |
|---|---|---|---|---|
| Instance | self | Instance | Yes | Via self.__class__ |
| Classmethod | cls | Class | No | Yes |
| Staticmethod | (none) | Nothing | No | No |

### Syntax

`python
class Demo:
    instance_attr = "class level"

    def __init__(self, value):
        self.instance_attr = value

    def instance_method(self):
        return f"Instance: {self.instance_attr}"

    @classmethod
    def class_method(cls):
        return f"Class: {cls.instance_attr}"

    @staticmethod
    def static_method(a, b):
        return f"Static: {a + b}"
`

### Beginner Examples

`python
class Temperature:
    scale = "Celsius"

    def __init__(self, value):
        self.value = value

    def to_fahrenheit(self):
        """Instance method — needs instance state."""
        return self.value * 9 / 5 + 32

    @classmethod
    def from_fahrenheit(cls, f):
        """Classmethod — alternative constructor."""
        celsius = (f - 32) * 5 / 9
        return cls(celsius)

    @staticmethod
    def celsius_to_fahrenheit(c):
        """Static method — no instance or class needed."""
        return c * 9 / 5 + 32

    @classmethod
    def get_scale(cls):
        """Classmethod — needs class state."""
        return cls.scale

t = Temperature(100)
print(t.to_fahrenheit())               # 212.0
print(Temperature.from_fahrenheit(212))  # Temperature(100)
print(Temperature.celsius_to_fahrenheit(0))  # 32.0
`

### Intermediate Examples

`python
import csv
import json
from typing import List, Dict, Any

class DataExporter:
    export_count = 0

    def __init__(self, data: List[Dict[str, Any]]):
        self.data = data

    def validate(self) -> bool:
        """Instance method — validates this instance's data."""
        return all(isinstance(row, dict) for row in self.data)

    @classmethod
    def from_csv(cls, filepath: str) -> "DataExporter":
        """Classmethod — alternative constructor from CSV."""
        with open(filepath, "r") as f:
            reader = csv.DictReader(f)
            data = list(reader)
        return cls(data)

    @classmethod
    def from_json(cls, filepath: str) -> "DataExporter":
        """Classmethod — alternative constructor from JSON."""
        with open(filepath, "r") as f:
            data = json.load(f)
        return cls(data)

    @staticmethod
    def infer_format(filepath: str) -> str:
        """Static method — utility, no instance/class needed."""
        if filepath.endswith(".csv"):
            return "csv"
        if filepath.endswith(".json"):
            return "json"
        return "unknown"

    @classmethod
    def auto_load(cls, filepath: str) -> "DataExporter":
        """Classmethod — factory that selects the right constructor."""
        fmt = cls.infer_format(filepath)
        if fmt == "csv":
            return cls.from_csv(filepath)
        if fmt == "json":
            return cls.from_json(filepath)
        raise ValueError(f"Unsupported format: {fmt}")

    def export_json(self, output_path: str) -> None:
        """Instance method — operates on instance data."""
        with open(output_path, "w") as f:
            json.dump(self.data, f, indent=2)
        DataExporter.export_count += 1

    @staticmethod
    def get_export_count() -> int:
        return DataExporter.export_count

exporter = DataExporter.auto_load("data.csv")
if exporter.validate():
    exporter.export_json("output.json")
    print(DataExporter.get_export_count())  # 1
`

### Advanced Examples

`python
from abc import ABC, abstractmethod
from typing import List, Type, Dict

class Plugin(ABC):
    _registry: Dict[str, Type["Plugin"]] = {}

    def __init__(self, name: str, config: dict = None):
        self.name = name
        self.config = config or {}

    @abstractmethod
    def execute(self, context: dict) -> dict:
        pass

    @classmethod
    def register(cls, plugin_cls: Type["Plugin"]) -> Type["Plugin"]:
        """Classmethod — registers a plugin subclass."""
        name = getattr(plugin_cls, "plugin_name", plugin_cls.__name__.lower())
        cls._registry[name] = plugin_cls
        return plugin_cls

    @classmethod
    def create(cls, name: str, config: dict = None) -> "Plugin":
        """Classmethod — factory that creates from registry."""
        if name not in cls._registry:
            raise ValueError(f"Unknown plugin: {name}")
        return cls._registry[name](name, config)

    @classmethod
    def list_plugins(cls) -> List[str]:
        """Classmethod — returns registered plugin names."""
        return list(cls._registry.keys())

    @classmethod
    def from_config_file(cls, filepath: str) -> List["Plugin"]:
        """Classmethod — bulk create from config file."""
        import json
        with open(filepath) as f:
            configs = json.load(f)
        return [cls.create(name, cfg) for name, cfg in configs.items()]

    @staticmethod
    def validate_config(config: dict) -> bool:
        """Static method — validates config structure."""
        required = ["timeout", "retries"]
        return all(k in config for k in required)


class LoggingPlugin(Plugin):
    plugin_name = "logging"

    def execute(self, context: dict) -> dict:
        print(f"[{self.name}] Logging: {context}")
        return {"status": "logged"}

class MetricsPlugin(Plugin):
    plugin_name = "metrics"

    def execute(self, context: dict) -> dict:
        return {"status": "metrics_collected", "data": context}


Plugin.register(LoggingPlugin)
Plugin.register(MetricsPlugin)

print(Plugin.list_plugins())  # ['logging', 'metrics']
plugin = Plugin.create("logging", {"timeout": 30, "retries": 3})
if Plugin.validate_config(plugin.config):
    result = plugin.execute({"message": "hello"})
    print(result)
`

### Real-World Use Cases

- **Alternative constructors**: datetime.fromtimestamp(), cls.from_string()
- **Registry pattern**: @classmethod to track subclasses
- **Singleton pattern**: @classmethod to return shared instance
- **Config loading**: Config.from_env(), Config.from_yaml(), etc.
- **Strategy pattern**: @staticmethod in classes that serve as strategy implementations

### Common Mistakes

1. **Using instance method when classmethod is needed**: calling self.__class__ everywhere
2. **Using staticmethod when classmethod is needed**: hardcoding class name
3. **Overusing classmethods**: not every factory needs to be a classmethod
4. **Modifying mutable class variables via instance**: affects all instances

### Best Practices

- Instance method: needs instance state (most methods)
- Classmethod: needs class state OR is an alternative constructor
- Staticmethod: needs neither, grouped for organizational convenience
- Prefer classmethod over staticmethod for factories (supports polymorphism)
- Prefer module-level functions over staticmethod unless grouping is valuable

### Performance Considerations

- Instance methods: fastest (direct binding to instance dict)
- Class methods: ~10% slower than instance methods
- Static methods: ~5% slower than plain functions
- All differences are negligible (< 0.1µs per call)

### Interview Questions

1. What is the difference between @staticmethod and @classmethod?
2. Why would you use a classmethod instead of a staticmethod for a factory?
3. Can you override a staticmethod in a subclass?
4. What happens if you call a classmethod on an instance vs the class?
5. How would you implement the singleton pattern using classmethods?
6. When would you use a classmethod vs a module-level function?

### Coding Challenges

1. Build a Config class with classmethods rom_env(), rom_file(), rom_dict() and a staticmethod alidate().
2. Create a Shape hierarchy with a classmethod registry that tracks all subclasses.
3. Implement a MongoCollection class with classmethod factories for different connection modes (single, replica set, sharded).

### Related Topics

Static methods, Class methods, Instance methods, Decorators, Metaclasses
