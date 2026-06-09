# JSON - json.dumps(), json.loads(), serializing/deserializing data

## Introduction

JSON (JavaScript Object Notation) is a lightweight, text-based data interchange format that is easy for humans to read and write and easy for machines to parse and generate. Python's `json` module provides a comprehensive interface for encoding Python objects to JSON strings and decoding JSON strings back to Python objects.

JSON supports primitive data types (strings, numbers, booleans, null) and structured data types (objects/dictionaries, arrays/lists). The format is language-independent, making it the most widely used data interchange format for web APIs, configuration files, and data storage.

Python's `json` module maps JSON types to Python types: JSON objects become `dict`, arrays become `list`, strings become `str`, numbers become `int` or `float`, `true`/`false` become `True`/`False`, and `null` becomes `None`.

## Why It Is Important

JSON has become the de facto standard for data interchange in modern software development. Understanding Python's JSON module is crucial because:

- **Web APIs**: Most REST APIs return JSON responses and accept JSON in request bodies
- **Configuration Files**: Many applications use JSON for configuration instead of INI or custom formats
- **Data Storage**: JSON is commonly used for storing semi-structured data in NoSQL databases and files
- **Interoperability**: JSON enables communication between systems written in different programming languages
- **Serialization**: JSON provides a standardized way to serialize complex data structures for transmission or storage
- **Human Readability**: JSON files can be easily read and edited by humans, unlike binary formats
- **Schema Validation**: JSON Schema provides a way to validate the structure and content of JSON data

Mastering JSON processing in Python is essential for web development, data engineering, API integration, and configuration management.

## Syntax

```python
import json

# Serialization: Python -> JSON string
json_string = json.dumps(obj, skipkeys=False, ensure_ascii=True, 
                         check_circular=True, allow_nan=True, 
                         cls=None, indent=None, separators=None, 
                         default=None, sort_keys=False)

# Deserialization: JSON string -> Python
python_obj = json.loads(json_string, cls=None, object_hook=None,
                        parse_float=None, parse_int=None,
                        parse_constant=None, object_pairs_hook=None)

# Serialization to file
with open('file.json', 'w') as f:
    json.dump(obj, f, indent=2, sort_keys=True)

# Deserialization from file
with open('file.json', 'r') as f:
    python_obj = json.load(f)

# Custom JSON encoding/decoding
class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        return super().default(obj)

class CustomDecoder(json.JSONDecoder):
    def __init__(self, *args, **kwargs):
        super().__init__(object_hook=self.object_hook, *args, **kwargs)
    
    def object_hook(self, dct):
        return dct
```

## Examples

### Basic JSON Operations

```python
import json

data = {
    'name': 'Alice',
    'age': 30,
    'city': 'New York',
    'is_student': False,
    'grades': [95, 87, 91],
    'address': None
}

json_string = json.dumps(data, indent=2)
print(json_string)

parsed = json.loads(json_string)
print(parsed['name'])
print(parsed['grades'][0])
print(parsed['address'])
```

### Reading and Writing JSON Files

```python
import json

data = {
    'users': [
        {'id': 1, 'name': 'Alice', 'email': 'alice@example.com'},
        {'id': 2, 'name': 'Bob', 'email': 'bob@example.com'},
        {'id': 3, 'name': 'Charlie', 'email': 'charlie@example.com'}
    ],
    'metadata': {
        'version': '1.0',
        'exported_at': '2024-01-15T10:30:00Z'
    }
}

with open('users.json', 'w') as f:
    json.dump(data, f, indent=2)

with open('users.json', 'r') as f:
    loaded_data = json.load(f)

for user in loaded_data['users']:
    print(f"{user['id']}: {user['name']} ({user['email']})")
```

### Pretty Printing with indent and sort_keys

```python
import json

data = {'z': 1, 'a': 2, 'm': 3, 'b': {'nested': 4, 'alpha': 5}}

print(json.dumps(data))

print(json.dumps(data, indent=4))

print(json.dumps(data, indent=4, sort_keys=True))
```

## Beginner Examples

### Example 1: User Preferences Manager

```python
import json
import os

class UserPreferences:
    def __init__(self, filename='preferences.json'):
        self.filename = filename
        self.preferences = self.load()
    
    def load(self):
        if os.path.exists(self.filename):
            try:
                with open(self.filename, 'r') as f:
                    return json.load(f)
            except (json.JSONDecodeError, IOError):
                return self.defaults()
        return self.defaults()
    
    def defaults(self):
        return {
            'theme': 'light',
            'language': 'en',
            'font_size': 12,
            'auto_save': True,
            'recent_files': [],
            'window_position': {'x': 100, 'y': 100}
        }
    
    def save(self):
        with open(self.filename, 'w') as f:
            json.dump(self.preferences, f, indent=2, sort_keys=True)
    
    def get(self, key, default=None):
        return self.preferences.get(key, default)
    
    def set(self, key, value):
        self.preferences[key] = value
        self.save()
    
    def add_recent_file(self, filepath):
        recent = self.preferences.get('recent_files', [])
        if filepath in recent:
            recent.remove(filepath)
        recent.insert(0, filepath)
        self.preferences['recent_files'] = recent[:10]
        self.save()

prefs = UserPreferences()
print(f"Current theme: {prefs.get('theme')}")
prefs.set('theme', 'dark')
prefs.add_recent_file('/path/to/document.txt')
print(f"Recent files: {prefs.get('recent_files')}")
```

### Example 2: Simple Contact Book

```python
import json
import os

CONTACTS_FILE = 'contacts.json'

def load_contacts():
    if os.path.exists(CONTACTS_FILE):
        with open(CONTACTS_FILE, 'r') as f:
            return json.load(f)
    return []

def save_contacts(contacts):
    with open(CONTACTS_FILE, 'w') as f:
        json.dump(contacts, f, indent=2)

def add_contact():
    name = input("Name: ")
    phone = input("Phone: ")
    email = input("Email: ")
    
    contacts = load_contacts()
    contacts.append({
        'name': name,
        'phone': phone,
        'email': email
    })
    save_contacts(contacts)
    print(f"Contact '{name}' added.")

def list_contacts():
    contacts = load_contacts()
    if not contacts:
        print("No contacts found.")
        return
    print("\nContacts:")
    for i, contact in enumerate(contacts, 1):
        print(f"{i}. {contact['name']} - {contact['phone']} - {contact['email']}")

def search_contacts():
    query = input("Search: ").lower()
    contacts = load_contacts()
    results = [c for c in contacts if query in c['name'].lower() 
               or query in c['phone'] or query in c['email'].lower()]
    
    if results:
        print(f"\nFound {len(results)} contact(s):")
        for contact in results:
            print(f"  {contact['name']} - {contact['phone']} - {contact['email']}")
    else:
        print("No contacts found.")

while True:
    print("\n--- Contact Book ---")
    print("1. Add Contact")
    print("2. List Contacts")
    print("3. Search Contacts")
    print("4. Exit")
    
    choice = input("Choose: ")
    if choice == '1':
        add_contact()
    elif choice == '2':
        list_contacts()
    elif choice == '3':
        search_contacts()
    elif choice == '4':
        print("Goodbye!")
        break
    else:
        print("Invalid choice.")
```

### Example 3: JSON to Table Converter

```python
import json

def json_to_table(json_data):
    if not isinstance(json_data, list) or not json_data:
        return "Empty data"
    
    headers = []
    for item in json_data:
        if isinstance(item, dict):
            for key in item.keys():
                if key not in headers:
                    headers.append(key)
    
    if not headers:
        return "No data to display"
    
    col_widths = {h: len(str(h)) for h in headers}
    for item in json_data:
        if isinstance(item, dict):
            for h in headers:
                val = str(item.get(h, ''))
                col_widths[h] = max(col_widths[h], len(val))
    
    separator = '+' + '+'.join('-' * (col_widths[h] + 2) for h in headers) + '+'
    
    lines = [separator]
    header_row = '|' + '|'.join(f' {h:<{col_widths[h]}} ' for h in headers) + '|'
    lines.append(header_row)
    lines.append(separator)
    
    for item in json_data:
        if isinstance(item, dict):
            row = '|' + '|'.join(f' {str(item.get(h, "")):<{col_widths[h]}} ' for h in headers) + '|'
            lines.append(row)
    
    lines.append(separator)
    return '\n'.join(lines)

sample_data = [
    {"name": "Alice", "age": 30, "city": "New York"},
    {"name": "Bob", "age": 25, "city": "Los Angeles"},
    {"name": "Charlie", "age": 35, "city": "Chicago"}
]

print(json_to_table(sample_data))

with open('formatted_output.json', 'w') as f:
    json.dump(sample_data, f, indent=2)
```

## Intermediate Examples

### Example 4: Deep JSON Merge and Patch

```python
import json
from copy import deepcopy

def json_deep_merge(base, override):
    result = deepcopy(base)
    
    for key, value in override.items():
        if key in result and isinstance(result[key], dict) and isinstance(value, dict):
            result[key] = json_deep_merge(result[key], value)
        else:
            result[key] = deepcopy(value)
    
    return result

def json_apply_patch(doc, patch):
    result = deepcopy(doc)
    
    for operation in patch:
        op = operation['op']
        path = operation['path'].strip('/').split('/') if operation['path'] != '/' else []
        
        if op == 'add':
            target = result
            for part in path[:-1]:
                if isinstance(target, list):
                    target = target[int(part)]
                else:
                    target = target[part]
            
            key = path[-1] if path else None
            if isinstance(target, list):
                idx = int(key) if key else len(target)
                target.insert(idx, operation['value'])
            else:
                target[key] = operation['value']
        
        elif op == 'remove':
            target = result
            for part in path[:-1]:
                if isinstance(target, list):
                    target = target[int(part)]
                else:
                    target = target[part]
            
            key = path[-1]
            if isinstance(target, list):
                del target[int(key)]
            else:
                del target[key]
        
        elif op == 'replace':
            target = result
            for part in path[:-1]:
                if isinstance(target, list):
                    target = target[int(part)]
                else:
                    target = target[part]
            
            key = path[-1]
            target[key] = operation['value']
        
        elif op == 'test':
            pass
    
    return result

base_config = {
    'database': {
        'host': 'localhost',
        'port': 5432,
        'credentials': {
            'user': 'admin',
            'password': 'secret'
        }
    },
    'logging': {
        'level': 'INFO',
        'file': '/var/log/app.log'
    }
}

user_config = {
    'database': {
        'host': 'prod-server.example.com',
        'credentials': {
            'user': 'prod_user'
        }
    },
    'logging': {
        'level': 'WARNING'
    }
}

merged = json_deep_merge(base_config, user_config)
print(json.dumps(merged, indent=2))

patch = [
    {'op': 'replace', 'path': '/database/host', 'value': 'new-host'},
    {'op': 'add', 'path': '/logging/format', 'value': 'json'},
    {'op': 'add', 'path': '/features', 'value': ['analytics', 'reporting']}
]

patched = json_apply_patch(merged, patch)
print(json.dumps(patched, indent=2))
```

### Example 5: Streaming JSON Processing

```python
import json
import ijson

def process_large_json_stream(filename):
    """Process a large JSON array using streaming parser."""
    results = []
    
    with open(filename, 'rb') as f:
        parser = ijson.parse(f)
        
        current_item = None
        current_key = None
        
        for prefix, event, value in parser:
            if prefix == 'item' and event == 'start_map':
                current_item = {}
            elif prefix == 'item' and event == 'end_map':
                if current_item:
                    results.append(process_item(current_item))
                    current_item = None
            elif event == 'map_key':
                current_key = value
            elif current_item is not None:
                current_item[current_key] = value
    
    return results

def process_item(item):
    """Transform or filter a single JSON item."""
    if item.get('active') == True:
        return {
            'id': item.get('id'),
            'name': item.get('name', '').upper(),
            'score': item.get('score', 0) * 1.1
        }
    return None

def create_large_json():
    """Create a large JSON file for testing."""
    data = []
    for i in range(10000):
        data.append({
            'id': i,
            'name': f'User_{i}',
            'active': i % 2 == 0,
            'score': i * 10
        })
    
    with open('large_data.json', 'w') as f:
        json.dump(data, f)

def efficient_json_dump(data, filename, chunk_size=1000):
    """Write large JSON data efficiently in chunks."""
    with open(filename, 'w') as f:
        f.write('[')
        for i in range(0, len(data), chunk_size):
            chunk = data[i:i + chunk_size]
            chunk_str = json.dumps(chunk)[1:-1]
            if i > 0:
                f.write(',')
            f.write(chunk_str)
        f.write(']')

if __name__ == '__main__':
    print("Creating large JSON file...")
    data = [{'id': i, 'value': f'item_{i}'} for i in range(50000)]
    efficient_json_dump(data, 'efficient_large.json')
    print("Done!")
```

### Example 6: JSON Schema Validation

```python
import json
from jsonschema import validate, ValidationError

USER_SCHEMA = {
    "$schema": "http://json-schema.org/draft-07/schema#",
    "type": "object",
    "properties": {
        "id": {"type": "integer", "minimum": 1},
        "name": {"type": "string", "minLength": 1, "maxLength": 100},
        "email": {"type": "string", "format": "email"},
        "age": {"type": "integer", "minimum": 0, "maximum": 150},
        "roles": {
            "type": "array",
            "items": {"type": "string"},
            "uniqueItems": True
        },
        "address": {
            "type": "object",
            "properties": {
                "street": {"type": "string"},
                "city": {"type": "string"},
                "zipcode": {"type": "string", "pattern": "^[0-9]{5}$"}
            },
            "required": ["city"]
        },
        "created_at": {"type": "string", "format": "date-time"}
    },
    "required": ["id", "name", "email"],
    "additionalProperties": False
}

def validate_user(user_data):
    try:
        validate(instance=user_data, schema=USER_SCHEMA)
        return True, "User data is valid"
    except ValidationError as e:
        return False, f"Validation error: {e.message}"

valid_user = {
    "id": 1,
    "name": "Alice Johnson",
    "email": "alice@example.com",
    "age": 30,
    "roles": ["admin", "user"],
    "address": {
        "street": "123 Main St",
        "city": "New York",
        "zipcode": "10001"
    },
    "created_at": "2024-01-15T10:30:00Z"
}

invalid_user = {
    "id": -1,
    "name": "",
    "email": "not-an-email"
}

for user in [valid_user, invalid_user]:
    is_valid, message = validate_user(user)
    print(f"{'Valid' if is_valid else 'Invalid'}: {message}")

# Export valid users to JSON
valid_users = [valid_user]
with open('valid_users.json', 'w') as f:
    json.dump(valid_users, f, indent=2)
```

## Advanced Examples

### Example 7: Custom JSON Encoder for Complex Objects

```python
import json
from datetime import datetime, date
from decimal import Decimal
from enum import Enum
from pathlib import Path

class Color(Enum):
    RED = 'red'
    GREEN = 'green'
    BLUE = 'blue'

class Person:
    def __init__(self, name, birth_date, salary, favorite_colors):
        self.name = name
        self.birth_date = birth_date
        self.salary = salary
        self.favorite_colors = favorite_colors

class CustomJSONEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, (datetime, date)):
            return obj.isoformat()
        elif isinstance(obj, Decimal):
            return float(obj)
        elif isinstance(obj, Enum):
            return obj.value
        elif isinstance(obj, Path):
            return str(obj)
        elif isinstance(obj, Person):
            return {
                '__type__': 'Person',
                'name': obj.name,
                'birth_date': obj.birth_date.isoformat(),
                'salary': obj.salary,
                'favorite_colors': [c.value for c in obj.favorite_colors]
            }
        elif isinstance(obj, set):
            return list(obj)
        elif isinstance(obj, bytes):
            return obj.hex()
        return super().default(obj)

class CustomJSONDecoder(json.JSONDecoder):
    def __init__(self, *args, **kwargs):
        super().__init__(object_hook=self.object_hook, *args, **kwargs)
    
    def object_hook(self, dct):
        if dct.get('__type__') == 'Person':
            return Person(
                name=dct['name'],
                birth_date=datetime.fromisoformat(dct['birth_date']).date(),
                salary=Decimal(str(dct['salary'])),
                favorite_colors=[Color(c) for c in dct['favorite_colors']]
            )
        return dct

data = {
    'name': 'Complex Data',
    'created_at': datetime.now(),
    'pi': Decimal('3.141592653589793'),
    'color': Color.RED,
    'path': Path('/usr/local/config'),
    'unique_ids': {1, 2, 3, 4, 5},
    'binary_data': b'hello',
    'person': Person(
        name='Alice',
        birth_date=date(1990, 5, 15),
        salary=Decimal('75000.50'),
        favorite_colors=[Color.BLUE, Color.GREEN]
    )
}

encoded = json.dumps(data, cls=CustomJSONEncoder, indent=2)
print("Encoded JSON:")
print(encoded)

with open('complex_data.json', 'w') as f:
    json.dump(data, f, cls=CustomJSONEncoder, indent=2)

with open('complex_data.json', 'r') as f:
    decoded_data = json.load(f, cls=CustomJSONDecoder)

print(f"\nDecoded person name: {decoded_data['person'].name}")
print(f"Decoded person salary: {decoded_data['person'].salary}")
print(f"Decoded favorite colors: {[c.value for c in decoded_data['person'].favorite_colors]}")
```

### Example 8: JSON Data Pipeline

```python
import json
from typing import Any, Callable, List

class JSONPipeline:
    def __init__(self):
        self.stages = []
    
    def add_stage(self, name: str, transformer: Callable[[Any], Any]):
        self.stages.append((name, transformer))
        return self
    
    def execute(self, data: Any) -> Any:
        result = data
        for name, transformer in self.stages:
            print(f"Pipeline stage: {name}")
            result = transformer(result)
        return result

def load_json_file(filename: str) -> dict:
    with open(filename, 'r') as f:
        return json.load(f)

def save_json_file(filename: str, data: Any):
    with open(filename, 'w') as f:
        json.dump(data, f, indent=2)

def filter_active_users(data: dict) -> dict:
    if 'users' in data:
        data['users'] = [u for u in data['users'] if u.get('active', False)]
    return data

def anonymize_users(data: dict) -> dict:
    if 'users' in data:
        for user in data['users']:
            if 'email' in user:
                local, domain = user['email'].split('@')
                user['email'] = f"{local[0]}***@{domain}"
            if 'name' in user:
                user['name'] = user['name'][0] + '***'
    return data

def add_metadata(data: dict) -> dict:
    data['processed_at'] = '2024-01-15T12:00:00Z'
    data['pipeline_version'] = '2.0'
    data['user_count'] = len(data.get('users', []))
    return data

def validate_data(data: dict) -> dict:
    required_fields = ['users']
    for field in required_fields:
        if field not in data:
            raise ValueError(f"Missing required field: {field}")
    return data

source_data = {
    'users': [
        {'id': 1, 'name': 'Alice Johnson', 'email': 'alice@example.com', 'active': True},
        {'id': 2, 'name': 'Bob Smith', 'email': 'bob@example.com', 'active': False},
        {'id': 3, 'name': 'Charlie Brown', 'email': 'charlie@example.com', 'active': True}
    ],
    'source': 'import_2024'
}

with open('pipeline_input.json', 'w') as f:
    json.dump(source_data, f, indent=2)

pipeline = JSONPipeline()
pipeline.add_stage('Load', load_json_file)
pipeline.add_stage('Validate', validate_data)
pipeline.add_stage('Filter Active', filter_active_users)
pipeline.add_stage('Anonymize', anonymize_users)
pipeline.add_stage('Add Metadata', add_metadata)
pipeline.add_stage('Save', lambda data: save_json_file('pipeline_output.json', data) or data)

result = pipeline.execute('pipeline_input.json')
print(json.dumps(result, indent=2))
```

## Real-World Use Cases

1. **REST API Communication**: All modern web APIs use JSON for request and response payloads. Python's `json` module is used with libraries like `requests` to serialize request data and deserialize API responses.

2. **Configuration Management**: Applications store configuration in JSON files that can be modified without code changes. Tools like `json5` extend JSON with comments for better configuration files.

3. **Database Export/Import**: JSON is a common format for exporting and importing data between different database systems and for database backups.

4. **Machine Learning Data Preparation**: JSON Lines format (one JSON object per line) is used for training data in ML pipelines, especially for NLP and structured data tasks.

5. **IoT Device Communication**: IoT devices send sensor data in JSON format to cloud platforms for processing and visualization.

6. **Caching and Serialization**: JSON is used as a cache format in Redis and other caching systems, and for serializing complex data structures for storage.

7. **Internationalization (i18n)**: Translation strings are typically stored in JSON files with language codes as keys.

## Common Mistakes

1. **Forgetting to Handle Non-Serializable Types**: Python objects like `datetime`, `Decimal`, and custom classes cannot be directly serialized. You need custom encoders.

2. **Not Specifying Encoding**: Always use `encoding='utf-8'` when reading/writing JSON files to avoid encoding issues.

3. **Loading Untrusted JSON Without Validation**: `json.loads()` can create arbitrary Python objects. Always validate JSON data before using it, especially from untrusted sources.

4. **Using eval() Instead of json.loads()**: Never use `eval()` to parse JSON strings. Always use `json.loads()` for safety.

5. **Ignoring ensure_ascii Parameter**: By default, `json.dumps()` escapes non-ASCII characters. Use `ensure_ascii=False` to preserve Unicode characters.

6. **Loading Entire Large JSON Files**: For large JSON files, use streaming parsers like `ijson` instead of loading the entire file into memory.

## Best Practices

1. **Use indent for Readability**: Use `indent=2` or `indent=4` when writing JSON files meant for human reading.

2. **Sort Keys for Consistency**: Use `sort_keys=True` to ensure consistent key ordering, especially for version-controlled files.

3. **Always Specify Encoding**: Always use `encoding='utf-8'` when opening JSON files.

4. **Validate JSON Data**: Use JSON Schema for validating JSON data structure and types before processing.

5. **Handle Errors Gracefully**: Catch `json.JSONDecodeError` and `KeyError` when parsing and accessing JSON data.

6. **Use Custom Encoders/Decoders**: For complex applications, create custom JSONEncoder and JSONDecoder subclasses to handle serialization consistently.

7. **Prefer json.load/json.dump for Files**: Use `json.load()` and `json.dump()` with file objects instead of reading/writing strings with `json.loads()`/`json.dumps()`.

## Interview Questions

**Q1: What is the difference between `json.dumps()` and `json.dump()`?**

A: `json.dumps()` serializes a Python object to a JSON string and returns it. `json.dump()` serializes a Python object and writes it directly to a file object. Similarly, `json.loads()` parses a JSON string, while `json.load()` reads from a file object.

**Q2: How do you serialize a datetime object to JSON?**

A: `datetime` objects are not JSON serializable by default. You need to either:
1. Convert them to string before serialization: `datetime.isoformat()`
2. Create a custom JSONEncoder that handles datetime objects
3. Use a `default` function: `json.dumps(data, default=str)`

**Q3: Explain the `object_hook` parameter in `json.loads()`.**

A: `object_hook` is an optional function that is called with the result of each JSON object decoded. It can be used to convert dictionaries to custom Python objects or to transform the decoded data. For example, you could automatically convert date strings to datetime objects.

**Q4: How does the `ensure_ascii` parameter work?**

A: When `ensure_ascii=True` (default), all non-ASCII characters in the output are escaped using `\uXXXX` sequences. When `ensure_ascii=False`, the characters are written as-is. This is useful for preserving Unicode characters in the output.

**Q5: What is JSON Schema and how do you use it in Python?**

A: JSON Schema is a vocabulary that allows you to annotate and validate JSON documents. It describes the structure, constraints, and types of JSON data. The `jsonschema` library in Python provides validation against JSON Schema.

## Coding Challenges

**Challenge 1: JSON Difference Finder**
Write a program that compares two JSON files and reports the differences between them (added, removed, and modified keys).

**Challenge 2: JSON to YAML Converter**
Implement a converter that transforms JSON data into YAML format without using external libraries.

**Challenge 3: JSON Query Engine**
Create a simple query engine that can filter and transform JSON data using a simple query language (similar to jq).

**Challenge 4: Nested JSON Flattener**
Write a function that flattens nested JSON objects into a single-level dictionary with dot-notation keys.

**Challenge 5: JSON Merge Tool**
Implement a JSON merge tool that can merge multiple JSON files into one, handling array concatenation and object merging.

## Summary

Python's `json` module provides a powerful and flexible interface for JSON serialization and deserialization. It handles basic types automatically and can be extended with custom encoders and decoders for complex objects. Key functions include `json.dumps()`/`json.dump()` for serialization and `json.loads()`/`json.load()` for deserialization. Parameters like `indent`, `sort_keys`, and `ensure_ascii` control output formatting. For large datasets, streaming parsers like `ijson` are recommended. JSON Schema provides validation capabilities. JSON remains the most important data interchange format in modern software development.

## Related Topics

- `48_file_operations.md` - File I/O basics for reading/writing JSON files
- `50_csv.md` - CSV as an alternative data format
- `51_pickle.md` - Binary serialization with pickle
- `52_pathlib.md` - Path management for JSON file locations
- `53_temp_files.md` - Temporary files for intermediate JSON data
- Web APIs - REST API communication with JSON
- NoSQL Databases - Document databases like MongoDB use JSON-like formats
- Data Serialization - Comparing JSON with XML, YAML, Protocol Buffers
