# JSON - json.dumps(), json.loads(), serializing/deserializing data

## Introduction

JSON (JavaScript Object Notation) is a lightweight, text-based data interchange format that is easy for humans to read and write and easy for machines to parse and generate. Python's `json` module provides a comprehensive interface for encoding Python objects to JSON strings and decoding JSON strings back to Python objects. JSON is language-independent, making it the most widely used data interchange format for web APIs, configuration files, and data storage.

## json.dumps()

### What It Is

`json.dumps()` (dump string) serializes a Python object to a JSON string. It converts Python data types to their JSON equivalents: `dict` to object, `list`/`tuple` to array, `str` to string, `int`/`float` to number, `True`/`False` to boolean, `None` to null.

### Why It Is Important

Serialization converts Python objects into a portable, language-independent format that can be transmitted over a network, stored in a file, or passed to other systems. `json.dumps()` is the primary serialization function, with parameters to control formatting, encoding, and handling of non-serializable types.

### How It Works Internally

`json.dumps()` walks the Python object tree recursively, converting each value to its JSON representation. It uses an encoder class (`JSONEncoder`) that maps Python types to JSON types. For unsupported types, it calls the `default` method, which by default raises `TypeError`. Custom encoders can override `default` to handle special types.

### Syntax

```python
import json

json_string = json.dumps(
    obj,
    skipkeys=False,       # Skip non-string dict keys
    ensure_ascii=True,    # Escape non-ASCII characters
    check_circular=True,  # Check for circular references
    allow_nan=True,       # Allow NaN, Infinity, -Infinity
    cls=None,             # Custom encoder class
    indent=None,          # Pretty-print indentation
    separators=None,      # (item_separator, key_separator)
    default=None,         # Function for non-serializable objects
    sort_keys=False,       # Sort dictionary keys
)
```

```python
import json

data = {"name": "Alice", "age": 30, "scores": [95, 87, 91]}

# Basic serialization
json_str = json.dumps(data)
print(json_str)  # {"name": "Alice", "age": 30, "scores": [95, 87, 91]}

# Pretty printing
print(json.dumps(data, indent=2))
# {
#   "name": "Alice",
#   "age": 30,
#   "scores": [
#     95,
#     87,
#     91
#   ]
# }

# Sorted keys for consistency
print(json.dumps(data, sort_keys=True, indent=2))

# Compact output (no spaces)
print(json.dumps(data, separators=(",", ":")))
# {"name":"Alice","age":30,"scores":[95,87,91]}

# Handling non-serializable types
from datetime import datetime
data["date"] = datetime.now()
try:
    json.dumps(data)  # TypeError: Object of type datetime is not JSON serializable
except TypeError:
    print("Can't serialize datetime")

# Use default parameter
json.dumps(data, default=str)  # Converts datetime to string
```

### Beginner Examples

```python
import json

# Serializing different types
print(json.dumps({"key": "value"}))       # {"key": "value"}
print(json.dumps([1, 2, 3]))             # [1, 2, 3]
print(json.dumps("hello"))               # "hello"
print(json.dumps(42))                    # 42
print(json.dumps(3.14))                  # 3.14
print(json.dumps(True))                  # true
print(json.dumps(None))                  # null
print(json.dumps((1, 2, 3)))             # [1, 2, 3] (tuple becomes array)

# Skipkeys: skip non-string keys
data = {1: "one", 2: "two"}
# json.dumps(data)  # TypeError: keys must be str, int, float, bool, None
json.dumps(data, skipkeys=True)  # {} (keys skipped)

# ensure_ascii
data = {"name": "José"}
print(json.dumps(data))                    # {"name": "Jos\u00e9"}
print(json.dumps(data, ensure_ascii=False))  # {"name": "José"}

# Writing to file
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)  # dump() not dumps() for files
```

## json.loads()

### What It Is

`json.loads()` (load string) deserializes a JSON string back into a Python object. It reverses the mapping: JSON object to `dict`, array to `list`, string to `str`, number to `int`/`float`, boolean to `bool`, null to `None`.

### Why It Is Important

Deserialization is how Python programs consume JSON data from APIs, configuration files, and data sources. Understanding how to safely parse JSON and handle edge cases (malformed JSON, unexpected types) is essential for robust data processing.

### How It Works Internally

`json.loads()` tokenizes the JSON string using a state machine parser, then recursively builds Python objects. The parser uses a decoder class (`JSONDecoder`) with configurable hooks (`object_hook`, `parse_float`, `parse_int`, `parse_constant`) that allow custom deserialization logic.

### Syntax

```python
import json

python_obj = json.loads(
    json_string,
    cls=None,               # Custom decoder class
    object_hook=None,        # Called for each decoded object (dict)
    parse_float=None,        # Called for each float (default: float)
    parse_int=None,          # Called for each int (default: int)
    parse_constant=None,     # Called for NaN, Infinity, -Infinity
    object_pairs_hook=None,  # Called for each decoded object with ordered pairs
)
```

```python
import json

json_str = '{"name": "Alice", "age": 30, "scores": [95, 87, 91]}'

# Basic deserialization
data = json.loads(json_str)
print(data["name"])       # Alice
print(data["scores"][0])  # 95

# Type conversions
json_str = '{"is_active": true, "value": null, "count": 42}'
data = json.loads(json_str)
print(data["is_active"])  # True
print(data["value"])      # None
print(data["count"])      # 42 (int)

# Reading from file
with open("data.json", "r") as f:
    data = json.load(f)  # load() not loads() for files

# Object hook for custom decoding
def decode_person(dct):
    if "__type__" in dct and dct["__type__"] == "Person":
        return Person(dct["name"], dct["age"])
    return dct

json_str = '{"__type__": "Person", "name": "Alice", "age": 30}'
person = json.loads(json_str, object_hook=decode_person)
print(type(person).__name__)  # Person
```

### Beginner Examples

```python
import json

# Simple parsing
data = json.loads('{"name": "Alice", "age": 30}')
print(data["name"])  # Alice

# Parsing arrays
items = json.loads('[1, 2, 3, "four"]')
print(items[0])  # 1
print(items[3])  # four

# Nested JSON
nested = json.loads('{"user": {"name": "Bob", "scores": [10, 20]}}')
print(nested["user"]["name"])     # Bob
print(nested["user"]["scores"])   # [10, 20]

# Handling JSON from API
import urllib.request

response = urllib.request.urlopen("https://api.example.com/data")
data = json.load(response)  # load from file-like object

# Error handling
try:
    data = json.loads("invalid json")
except json.JSONDecodeError as e:
    print(f"Invalid JSON at line {e.lineno}, column {e.colno}: {e.msg}")

# Loading boolean and null
data = json.loads('{"active": true, "data": null}')
print(data["active"])  # True
print(data["data"])    # None
```

## Serialization

### What It Is

Serialization (encoding) converts Python objects to JSON-formatted strings or bytes for storage or transmission. The `json` module provides `json.dumps()` for strings and `json.dump()` for file objects.

### Why It Is Important

Serialization enables data persistence, API communication, and cross-language data exchange. Python's JSON serialization handles common types automatically and can be extended for custom types through encoder classes.

### How It Works Internally

The JSON encoder uses a `make_encoder()` function that creates a state machine for efficient serialization. It iterates over the input object recursively, writing JSON tokens to an output buffer. The encoder handles circular reference detection, NaN/infinity handling, and key sorting.

### Beginner Examples

```python
import json

# Basic serialization to file
data = {"name": "Alice", "age": 30}
with open("data.json", "w") as f:
    json.dump(data, f)

# Reading it back
with open("data.json", "r") as f:
    loaded = json.load(f)
print(loaded)

# Serializing complex nested data
data = {
    "users": [
        {"id": 1, "name": "Alice", "scores": [95, 87]},
        {"id": 2, "name": "Bob", "scores": [78, 92]},
    ],
    "metadata": {"version": "1.0", "exported": True},
}
with open("users.json", "w") as f:
    json.dump(data, f, indent=2, sort_keys=True)
```

### Intermediate Examples

```python
# Handling datetime serialization
from datetime import datetime

def json_serial(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    raise TypeError(f"Type {type(obj)} not serializable")

data = {"timestamp": datetime.now(), "value": 42}
print(json.dumps(data, default=json_serial))

# Serialization with custom encoder
class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        if isinstance(obj, set):
            return list(obj)
        if isinstance(obj, bytes):
            return obj.hex()
        return super().default(obj)

data = {
    "date": datetime.now(),
    "tags": {"python", "json"},
    "binary": b"hello",
}
print(json.dumps(data, cls=CustomEncoder, indent=2))
```

## Deserialization

### What It Is

Deserialization (decoding) converts JSON-formatted data back into Python objects using `json.loads()` (strings) or `json.load()` (files).

### Why It Is Important

Deserialization is how programs consume data from external sources. Safe deserialization practices prevent issues from malformed or malicious JSON.

### How It Works Internally

The JSON decoder tokenizes the input string, validates it against the JSON grammar, and recursively builds Python objects. It uses a `scan_once` function that advances through the input positionally. Error handling tracks line and column numbers for useful error messages.

### Beginner Examples

```python
import json

# Loading from string
json_str = '{"city": "New York", "population": 8336817}'
data = json.loads(json_str)
print(f"City: {data['city']}, Population: {data['population']}")

# Loading from file
with open("config.json", "r") as f:
    config = json.load(f)

# Handling missing keys
data = json.loads('{"name": "Alice"}')
print(data.get("age", "unknown"))  # unknown

# Error handling
try:
    data = json.loads("{invalid}")
except json.JSONDecodeError as e:
    print(f"Parse error at line {e.lineno}: {e.msg}")
```

### Advanced Examples

```python
# Custom object hook for date deserialization
from datetime import datetime

def date_hook(dct):
    for key, value in dct.items():
        if isinstance(value, str) and "T" in value:
            try:
                dct[key] = datetime.fromisoformat(value)
            except ValueError:
                pass
    return dct

json_str = '{"name": "Event", "created": "2024-01-15T10:30:00"}'
data = json.loads(json_str, object_hook=date_hook)
print(type(data["created"]))  # <class 'datetime.datetime'>

# object_pairs_hook for preserving key order
from collections import OrderedDict

data = json.loads(
    '{"b": 1, "a": 2, "c": 3}',
    object_pairs_hook=OrderedDict
)
print(list(data.keys()))  # ['b', 'a', 'c']

# Custom decoder for type conversion
class TypedDecoder(json.JSONDecoder):
    def __init__(self):
        super().__init__(object_hook=self.typed_hook)

    def typed_hook(self, dct):
        for key, value in dct.items():
            if key == "id":
                dct[key] = str(value)  # Ensure id is string
        return dct

data = json.loads('{"id": 42, "name": "Alice"}', cls=TypedDecoder)
print(type(data["id"]))  # <class 'str'>
```

### Real-World Use Cases

```python
# API integration
import json
import urllib.request

def fetch_user(user_id):
    url = f"https://jsonplaceholder.typicode.com/users/{user_id}"
    with urllib.request.urlopen(url) as response:
        return json.load(response)

user = fetch_user(1)
print(f"User: {user['name']}, Email: {user['email']}")

# Configuration management
import json
import os

class ConfigManager:
    def __init__(self, path="config.json"):
        self.path = path
        self.config = self.load()

    def load(self):
        if os.path.exists(self.path):
            with open(self.path, "r") as f:
                return json.load(f)
        return {}

    def save(self):
        with open(self.path, "w") as f:
            json.dump(self.config, f, indent=2)

    def get(self, key, default=None):
        return self.config.get(key, default)

    def set(self, key, value):
        self.config[key] = value
        self.save()

# Data pipeline
import json

class JSONPipeline:
    def process(self, input_file, output_file, transforms):
        with open(input_file, "r") as f:
            data = json.load(f)

        for transform in transforms:
            data = transform(data)

        with open(output_file, "w") as f:
            json.dump(data, f, indent=2)
```

### Common Mistakes

```python
# Mistake 1: Forgetting to handle non-serializable types
data = {"date": datetime.now()}
json.dumps(data)  # TypeError!

# Correct:
json.dumps(data, default=str)

# Mistake 2: Not specifying encoding for files
with open("data.json", "r") as f:  # Platform-dependent encoding!
    data = json.load(f)

# Correct:
with open("data.json", "r", encoding="utf-8") as f:
    data = json.load(f)

# Mistake 3: Using eval() instead of json.loads()
data = eval('{"key": "value"}')  # DANGEROUS! Use json.loads()

# Mistake 4: Loading untrusted JSON without validation
data = json.loads(untrusted_json)  # Safe from code execution, but validate structure

# Mistake 5: Forgetting ensure_ascii=False for Unicode
print(json.dumps("café"))  # "caf\u00e9"
print(json.dumps("café", ensure_ascii=False))  # "café"

# Mistake 6: Loading entire large JSON into memory
# Use ijson for streaming large JSON arrays
```

### Best Practices

- Use `indent=2` and `sort_keys=True` for human-readable JSON files
- Always specify `encoding="utf-8"` when reading/writing JSON files
- Handle `json.JSONDecodeError` when parsing untrusted JSON
- Use `default` parameter or custom encoder for non-serializable types
- Do NOT use `eval()` to parse JSON—always use `json.loads()`
- Validate JSON structure after loading (manually or with JSON Schema)
- Use `object_hook` for custom deserialization logic
- Prefer `json.load()`/`json.dump()` with files over string versions
- Use streaming parsers for large JSON data

### Performance Considerations

- `json.dumps()` performance depends on object complexity and size
- Use `separators=(",", ":")` for compact output (faster to write/read)
- The `indent` parameter adds ~20-50% overhead for output size
- `sort_keys=True` adds sorting overhead
- For high-throughput serialization, consider `orjson` or `ujson`
- `json.loads()` is very fast for moderate-sized data
- Large JSON arrays should be streamed (not loaded entirely) with `ijson`

### Interview Questions

1. What's the difference between `json.dumps()` and `json.dump()`?

   `dumps()` serializes to a string. `dump()` writes to a file object. Similarly, `loads()` parses a string, `load()` reads from a file.

2. How do you serialize a `datetime` object?

   Use `default=str` for simple conversion, or create a custom `JSONEncoder` subclass that handles `datetime` specially.

3. What is `object_hook` in `json.loads()`?

   It's a function called for each decoded dict. It can convert dicts to custom objects or transform data. For example, converting ISO date strings to `datetime` objects.

4. How does `ensure_ascii` work?

   When `True` (default), non-ASCII chars are escaped as `\uXXXX`. When `False`, they're written as-is, preserving Unicode.

5. What is JSON Schema validation?

   JSON Schema describes the structure and constraints of JSON data. The `jsonschema` library validates JSON against a schema, catching structural issues.

6. Can JSON handle circular references?

   No. `json.dumps()` raises `ValueError` for circular references. You must handle them manually (e.g., by removing back-references).

### Coding Challenges

1. Write a JSON differ that compares two JSON objects and reports added, removed, and modified keys.

2. Implement a JSON to YAML converter without external libraries.

3. Build a JSON query engine that filters/transforms JSON data using a simple query language.

4. Create a nested JSON flattener that converts nested objects to dot-notation keys.

5. Implement a JSON merge tool that combines multiple JSON files handling array concatenation and object merging.

### Related Topics

- `48_file_operations.md` - File I/O for JSON files
- `50_csv.md` - CSV as alternative data format
- `51_pickle.md` - Binary serialization
- `52_pathlib.md` - Path management for JSON files
- Data serialization (YAML, XML, Protocol Buffers, MessagePack)
- REST APIs and JSON communication
- JSON Schema for validation
