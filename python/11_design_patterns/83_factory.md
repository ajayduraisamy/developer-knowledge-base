# Factory - Factory Method, Abstract Factory, dynamic class selection
## Introduction
The Factory pattern provides an interface for creating objects without specifying their concrete classes. It decouples object creation from usage, making code more flexible and maintainable. Python offers three main factory approaches: Factory Method (single method), Abstract Factory (family of related objects), and dynamic class selection (runtime choice). Python's first-class functions and dynamic typing make factory implementation particularly elegant compared to statically-typed languages.

## Factory Method
### What It Is
Factory Method defines an interface for creating a single object, letting subclasses decide which class to instantiate. The creation logic is encapsulated in a method (often a class method or static method) rather than in the constructor.

### Why It Is Important
Factory Method allows code to work with interfaces rather than concrete classes. It enables subclassing to change the type of objects created without modifying client code.

### How It Works Internally
The pattern defines a creator class with a factory method. Subclasses override this method to change the product type. The client calls the factory method without knowing the concrete product class.

```python
from abc import ABC, abstractmethod

class Document(ABC):
    @abstractmethod
    def create(self):
        pass

    @abstractmethod
    def open(self):
        pass

class PDFDocument(Document):
    def create(self):
        print("Creating PDF document")

    def open(self):
        print("Opening PDF document with PDF viewer")

class WordDocument(Document):
    def create(self):
        print("Creating Word document")

    def open(self):
        print("Opening Word document with Word processor")

class TextDocument(Document):
    def create(self):
        print("Creating text document")

    def open(self):
        print("Opening text document with text editor")

class DocumentCreator(ABC):
    @abstractmethod
    def create_document(self) -> Document:
        pass

    def edit_document(self):
        doc = self.create_document()
        doc.create()
        doc.open()
        return doc

class PDFCreator(DocumentCreator):
    def create_document(self) -> Document:
        return PDFDocument()

class WordCreator(DocumentCreator):
    def create_document(self) -> Document:
        return WordDocument()

class TextCreator(DocumentCreator):
    def create_document(self) -> Document:
        return TextDocument()

# Usage
def client_code(creator: DocumentCreator):
    print(f"Working with {creator.__class__.__name__}")
    document = creator.edit_document()
    return document

client_code(PDFCreator())
client_code(WordCreator())
```

### Pythonic Factory Method
```python
# Using classmethod for factory (most Pythonic)
class User:
    def __init__(self, username, email):
        self.username = username
        self.email = email

    @classmethod
    def from_dict(cls, data):
        return cls(data['username'], data['email'])

    @classmethod
    def from_csv(cls, csv_string):
        username, email = csv_string.strip().split(',')
        return cls(username, email)

    @classmethod
    def anonymous(cls):
        return cls("anonymous", "anon@localhost")

# Usage
user1 = User("alice", "alice@example.com")
user2 = User.from_dict({"username": "bob", "email": "bob@example.com"})
user3 = User.from_csv("charlie,charlie@example.com")
user4 = User.anonymous()

print(user1.username)  # alice
print(user2.username)  # bob
print(user3.username)  # charlie
print(user4.username)  # anonymous
```

### Parameterized Factory Method
```python
class Payment:
    def pay(self, amount):
        raise NotImplementedError

class CreditCardPayment(Payment):
    def pay(self, amount):
        print(f"Paid ${amount} via Credit Card")

class PayPalPayment(Payment):
    def pay(self, amount):
        print(f"Paid ${amount} via PayPal")

class CryptoPayment(Payment):
    def pay(self, amount):
        print(f"Paid ${amount} via Cryptocurrency")

class PaymentFactory:
    @staticmethod
    def create_payment(method: str) -> Payment:
        payments = {
            "credit_card": CreditCardPayment,
            "paypal": PayPalPayment,
            "crypto": CryptoPayment,
        }
        payment_class = payments.get(method.lower())
        if not payment_class:
            raise ValueError(f"Unknown payment method: {method}")
        return payment_class()

# Usage
payment = PaymentFactory.create_payment("paypal")
payment.pay(100.50)
```

## Abstract Factory
### What It Is
Abstract Factory provides an interface for creating families of related or dependent objects without specifying their concrete classes. It composes multiple factory methods into a cohesive unit.

### Why It Is Important
Abstract Factory ensures that products from the same family are used together, maintaining consistency. It's particularly useful for UI toolkits, database adapters, or any system that needs to support multiple "looks" or configurations.

### How It Works Internally
The pattern defines abstract product interfaces and an abstract factory interface. Concrete factories implement the factory interface to produce concrete products. Client code is written against the abstract interfaces, not concrete implementations.

```python
from abc import ABC, abstractmethod

# Abstract Products
class Button(ABC):
    @abstractmethod
    def render(self):
        pass

class Checkbox(ABC):
    @abstractmethod
    def render(self):
        pass

class ScrollBar(ABC):
    @abstractmethod
    def render(self):
        pass

# Concrete Products for Windows
class WindowsButton(Button):
    def render(self):
        return "Windows Style Button"

class WindowsCheckbox(Checkbox):
    def render(self):
        return "Windows Style Checkbox"

class WindowsScrollBar(ScrollBar):
    def render(self):
        return "Windows Style ScrollBar"

# Concrete Products for Mac
class MacButton(Button):
    def render(self):
        return "Mac Style Button"

class MacCheckbox(Checkbox):
    def render(self):
        return "Mac Style Checkbox"

class MacScrollBar(ScrollBar):
    def render(self):
        return "Mac Style ScrollBar"

# Concrete Products for Linux
class LinuxButton(Button):
    def render(self):
        return "Linux Style Button"

class LinuxCheckbox(Checkbox):
    def render(self):
        return "Linux Style Checkbox"

class LinuxScrollBar(ScrollBar):
    def render(self):
        return "Linux Style ScrollBar"

# Abstract Factory
class GUIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button:
        pass

    @abstractmethod
    def create_checkbox(self) -> Checkbox:
        pass

    @abstractmethod
    def create_scrollbar(self) -> ScrollBar:
        pass

# Concrete Factories
class WindowsFactory(GUIFactory):
    def create_button(self) -> Button:
        return WindowsButton()

    def create_checkbox(self) -> Checkbox:
        return WindowsCheckbox()

    def create_scrollbar(self) -> ScrollBar:
        return WindowsScrollBar()

class MacFactory(GUIFactory):
    def create_button(self) -> Button:
        return MacButton()

    def create_checkbox(self) -> Checkbox:
        return MacCheckbox()

    def create_scrollbar(self) -> ScrollBar:
        return MacScrollBar()

class LinuxFactory(GUIFactory):
    def create_button(self) -> Button:
        return LinuxButton()

    def create_checkbox(self) -> Checkbox:
        return LinuxCheckbox()

    def create_scrollbar(self) -> ScrollBar:
        return LinuxScrollBar()

# Application
class Application:
    def __init__(self, factory: GUIFactory):
        self.factory = factory
        self.ui_elements = []

    def create_ui(self):
        self.ui_elements.append(self.factory.create_button())
        self.ui_elements.append(self.factory.create_checkbox())
        self.ui_elements.append(self.factory.create_scrollbar())

    def render(self):
        return [element.render() for element in self.ui_elements]

# Usage
def get_factory(os_type: str) -> GUIFactory:
    factories = {
        "windows": WindowsFactory(),
        "mac": MacFactory(),
        "linux": LinuxFactory(),
    }
    return factories.get(os_type, WindowsFactory())

app = Application(get_factory("mac"))
app.create_ui()
print(app.render())
# ['Mac Style Button', 'Mac Style Checkbox', 'Mac Style ScrollBar']
```

### Abstract Factory with Data Adapters
```python
from abc import ABC, abstractmethod

class DatabaseConnection(ABC):
    @abstractmethod
    def connect(self):
        pass

class DatabaseQuery(ABC):
    @abstractmethod
    def execute(self, sql):
        pass

class PostgreSQLConnection(DatabaseConnection):
    def connect(self):
        return "Connected to PostgreSQL"

class PostgreSQLQuery(DatabaseQuery):
    def execute(self, sql):
        return f"PostgreSQL executing: {sql}"

class MySQLConnection(DatabaseConnection):
    def connect(self):
        return "Connected to MySQL"

class MySQLQuery(DatabaseQuery):
    def execute(self, sql):
        return f"MySQL executing: {sql}"

class DatabaseFactory(ABC):
    @abstractmethod
    def create_connection(self) -> DatabaseConnection:
        pass

    @abstractmethod
    def create_query(self) -> DatabaseQuery:
        pass

class PostgreSQLFactory(DatabaseFactory):
    def create_connection(self):
        return PostgreSQLConnection()

    def create_query(self):
        return PostgreSQLQuery()

class MySQLFactory(DatabaseFactory):
    def create_connection(self):
        return MySQLConnection()

    def create_query(self):
        return MySQLQuery()

# Client code uses only abstract interfaces
class DatabaseClient:
    def __init__(self, factory: DatabaseFactory):
        self.connection = factory.create_connection()
        self.query = factory.create_query()

    def run(self, sql):
        conn_result = self.connection.connect()
        query_result = self.query.execute(sql)
        return f"{conn_result}\n{query_result}"

client = DatabaseClient(PostgreSQLFactory())
print(client.run("SELECT * FROM users"))
```

## Dynamic class selection
### What It Is
Dynamic class selection uses Python's runtime capabilities to choose and instantiate classes based on configuration, string names, or other runtime conditions. This leverages Python's first-class nature where classes are objects that can be stored in dictionaries and looked up dynamically.

### Why It Is Important
Dynamic selection eliminates long if-elif chains and makes the system extensible without modifying factory code. New types can be added by registration rather than by editing factory logic.

### How It Works Internally
Classes are registered in a dictionary (either manually or via decorators). When a type is requested, the dictionary maps the identifier to the class, which is then instantiated. This is essentially the Registry pattern combined with Factory.

```python
class Notification:
    def send(self, message):
        raise NotImplementedError

class EmailNotification(Notification):
    def send(self, message):
        print(f"Sending email: {message}")

class SMSNotification(Notification):
    def send(self, message):
        print(f"Sending SMS: {message}")

class PushNotification(Notification):
    def send(self, message):
        print(f"Sending push: {message}")

# Registry-based factory
class NotificationFactory:
    _registry = {}

    @classmethod
    def register(cls, name):
        def decorator(notification_class):
            cls._registry[name] = notification_class
            return notification_class
        return decorator

    @classmethod
    def create(cls, name, *args, **kwargs):
        if name not in cls._registry:
            raise ValueError(f"Unknown notification type: {name}")
        return cls._registry[name](*args, **kwargs)

# Register via decorator
@NotificationFactory.register("email")
class RegisteredEmail(EmailNotification):
    pass

@NotificationFactory.register("sms")
class RegisteredSMS(SMSNotification):
    pass

@NotificationFactory.register("push")
class RegisteredPush(PushNotification):
    pass

# Usage
notif = NotificationFactory.create("sms")
notif.send("Hello via SMS!")
```

### Plugin-Style Registration
```python
import importlib
import pkgutil

class PluginFactory:
    _plugins = {}

    @classmethod
    def discover(cls, package_name):
        """Auto-discover and register plugins from a package"""
        package = importlib.import_module(package_name)
        for _, name, is_pkg in pkgutil.iter_modules(package.__path__):
            module = importlib.import_module(f"{package_name}.{name}")
            if hasattr(module, 'register_plugin'):
                module.register_plugin(cls)

    @classmethod
    def register(cls, name, plugin_class):
        cls._plugins[name] = plugin_class

    @classmethod
    def create(cls, name, *args, **kwargs):
        if name not in cls._plugins:
            raise KeyError(f"Plugin '{name}' not found")
        return cls._plugins[name](*args, **kwargs)

# plugins/email_plugin.py
def register_plugin(factory):
    from myapp.notifications import EmailNotification

    class CustomEmailNotification(EmailNotification):
        def send(self, message):
            print(f"[CustomEmail] {message}")

    factory.register("custom_email", CustomEmailNotification)
```

### Configuration-Driven Factory
```python
import json
import importlib

class ConfigurableFactory:
    def __init__(self, config_file=None, config_dict=None):
        if config_file:
            with open(config_file, 'r') as f:
                self.config = json.load(f)
        else:
            self.config = config_dict or {}
        self._class_cache = {}

    def _resolve_class(self, class_path):
        if class_path in self._class_cache:
            return self._class_cache[class_path]

        module_path, class_name = class_path.rsplit('.', 1)
        module = importlib.import_module(module_path)
        cls = getattr(module, class_name)
        self._class_cache[class_path] = cls
        return cls

    def create(self, key, *args, **kwargs):
        if key not in self.config:
            raise ValueError(f"No configuration for: {key}")
        entry = self.config[key]
        cls = self._resolve_class(entry['class'])
        init_args = {**entry.get('default_args', {}), **kwargs}
        return cls(*args, **init_args)

# config.json
# {
#     "database": {
#         "class": "myapp.database.PostgreSQLDatabase",
#         "default_args": {"host": "localhost", "port": 5432}
#     },
#     "cache": {
#         "class": "myapp.cache.RedisCache",
#         "default_args": {"host": "localhost", "port": 6379}
#     }
# }
```

### Beginner Examples
```python
# Simple utility registration
FORMATTERS = {}

def register_formatter(name):
    def decorator(cls):
        FORMATTERS[name] = cls
        return cls
    return decorator

@register_formatter("json")
class JSONFormatter:
    def format(self, data):
        import json
        return json.dumps(data)

@register_formatter("csv")
class CSVFormatter:
    def format(self, data):
        import csv, io
        output = io.StringIO()
        writer = csv.writer(output)
        for row in data:
            writer.writerow(row)
        return output.getvalue()

def format_data(data, style="json"):
    formatter = FORMATTERS.get(style)
    if not formatter:
        raise ValueError(f"Unknown format: {style}")
    return formatter().format(data)
```

### Intermediate Examples
```python
class Parser:
    def parse(self, data):
        raise NotImplementedError

class JSONParser(Parser):
    def parse(self, data):
        import json
        return json.loads(data)

class XMLParser(Parser):
    def parse(self, data):
        import xml.etree.ElementTree as ET
        return ET.fromstring(data)

class YAMLParser(Parser):
    def parse(self, data):
        import yaml
        return yaml.safe_load(data)

class ParserFactory:
    _parsers = {}

    @classmethod
    def register(cls, format, parser_class):
        cls._parsers[format] = parser_class

    @classmethod
    def get_parser(cls, format, **kwargs):
        parser_class = cls._parsers.get(format)
        if not parser_class:
            raise ValueError(f"Unsupported format: {format}")
        return parser_class(**kwargs)

# Register parsers
ParserFactory.register("json", JSONParser)
ParserFactory.register("xml", XMLParser)
ParserFactory.register("yaml", YAMLParser)

# Usage
parser = ParserFactory.get_parser("json")
data = parser.parse('{"name": "Alice", "age": 30}')
```

### Advanced Examples
```python
from abc import ABC, abstractmethod
from dataclasses import dataclass

@dataclass
class ModelConfig:
    name: str
    params: dict

class MLModel(ABC):
    @abstractmethod
    def train(self, X, y):
        pass

    @abstractmethod
    def predict(self, X):
        pass

class RandomForestModel(MLModel):
    def __init__(self, n_estimators=100, max_depth=None):
        self.n_estimators = n_estimators
        self.max_depth = max_depth

    def train(self, X, y):
        print(f"Training RandomForest with {self.n_estimators} trees")

    def predict(self, X):
        print("RandomForest predicting...")
        return [0] * len(X)

class NeuralNetworkModel(MLModel):
    def __init__(self, layers=None, learning_rate=0.01):
        self.layers = layers or [64, 32, 16]
        self.learning_rate = learning_rate

    def train(self, X, y):
        print(f"Training NeuralNetwork with layers {self.layers}")

    def predict(self, X):
        print("NeuralNetwork predicting...")
        return [0] * len(X)

class ModelFactory:
    _models = {
        "random_forest": RandomForestModel,
        "neural_network": NeuralNetworkModel,
    }

    @classmethod
    def create(cls, config: ModelConfig) -> MLModel:
        model_class = cls._models.get(config.name)
        if not model_class:
            raise ValueError(f"Unknown model: {config.name}")
        return model_class(**config.params)

# Usage
configs = [
    ModelConfig("random_forest", {"n_estimators": 200}),
    ModelConfig("neural_network", {"layers": [128, 64, 32], "learning_rate": 0.001}),
]

for config in configs:
    model = ModelFactory.create(config)
    model.train([[1, 2], [3, 4]], [0, 1])
```

### Real-World Use Cases
- Database abstraction layers (SQLite, PostgreSQL, MySQL)
- GUI toolkit backends (Qt, Tkinter, wxWidgets)
- Notification systems (email, SMS, push, Slack)
- Data serialization (JSON, XML, YAML, Protobuf)
- ML model training pipelines
- Payment processors (credit card, PayPal, crypto)

### Common Mistakes
- Making factories too complex with unnecessary abstraction
- Not handling unknown types gracefully
- Overusing factory pattern for simple object creation
- Creating tight coupling between factory and product classes
- Forgetting to register new product types

### Best Practices
- Use simple factory methods (@classmethod) for most cases
- Use Abstract Factory when you have families of related products
- Use registry pattern for extensible factories
- Include error messages that list available types
- Consider if a simple constructor is sufficient

### Performance Considerations
- Factory overhead is negligible for most cases
- Dictionary lookups are O(1) for dynamic selection
- Import-time registration can slow module loading
- Class caching in factories improves repeated creation

### Interview Questions
1. Explain the difference between Factory Method and Abstract Factory.
2. How do you implement a factory pattern in Python without using classes?
3. When would you use a registry-based factory?
4. How does Python's duck typing affect factory pattern implementation?

### Coding Challenges
1. Implement a factory for different compression algorithms (zip, gzip, bz2)
2. Create an Abstract Factory for cross-platform file dialogs
3. Build a plugin system using registry-based factory
4. Implement configuration-driven factory reading from YAML

### Related Topics
- Singleton pattern
- Builder pattern
- Dependency injection
- Plugin architecture
- Reflection and introspection
- Abstract base classes
