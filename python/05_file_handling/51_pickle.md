# Pickle - pickle.dump(), pickle.load(), serialization protocol

## Introduction

Pickle is Python's built-in module for serializing and deserializing Python object structures. Serialization (or "pickling") converts a Python object hierarchy into a byte stream, and deserialization ("unpickling") reconstructs the original object hierarchy from the byte stream.

The pickle module implements a powerful, recursive algorithm for converting Python objects to a portable binary format. It can handle a wide range of Python types, including custom class instances, nested structures, and circular references.

Unlike JSON, which is language-agnostic and only supports basic types, pickle is Python-specific and can serialize virtually any Python object, including functions, classes, and complex custom objects. However, this Python-specific nature also introduces important security considerations.

## Why It Is Important

Pickle plays a critical role in Python applications that need to preserve complex program state:

- **Object Persistence**: Save complex Python objects (models, configurations, ML models) to disk for later use
- **Caching**: Cache expensive computations or processed data in a serialized form
- **Inter-Process Communication**: Transfer Python objects between processes or across a network
- **Machine Learning**: Serialize trained ML models (scikit-learn, etc.) for deployment
- **Session State**: Save and restore application state across sessions
- **Data Pipelines**: Pass complex data structures between pipeline stages
- **Checkpointing**: Save computation progress for long-running processes to enable recovery

Understanding pickle is essential for any Python developer working with data science, machine learning, caching systems, or distributed computing.

## Syntax

```python
import pickle

# Serialization: Python object -> bytes
bytes_data = pickle.dumps(obj, protocol=None, fix_imports=True, buffer_callback=None)

# Serialization to file
with open('file.pkl', 'wb') as f:
    pickle.dump(obj, f, protocol=pickle.HIGHEST_PROTOCOL)

# Deserialization: bytes -> Python object
obj = pickle.loads(bytes_data, fix_imports=True, encoding='ASCII', errors='strict')

# Deserialization from file
with open('file.pkl', 'rb') as f:
    obj = pickle.load(f)

# Protocol versions (integer 0-5)
# 0: Text protocol (human-readable)
# 1: Binary protocol (old)
# 2: Binary protocol (Python 2.3+)
# 3: Default for Python 3.0-3.7
# 4: Default for Python 3.8+ (added support for large objects)
# 5: Python 3.8+ (with out-of-band data support)
```

## Examples

### Basic Pickling and Unpickling

```python
import pickle

# Simple data structures
data = {
    'name': 'Alice',
    'scores': [95, 87, 91],
    'metadata': {
        'class': 'Math 101',
        'year': 2024,
        'active': True
    }
}

# Pickle to bytes
pickled_bytes = pickle.dumps(data)
print(f"Pickled bytes: {pickled_bytes[:50]}...")
print(f"Size: {len(pickled_bytes)} bytes")

# Unpickle from bytes
unpickled = pickle.loads(pickled_bytes)
print(f"Unpickled: {unpickled}")
print(f"Same object? {data == unpickled}")

# Pickle to file
with open('data.pkl', 'wb') as f:
    pickle.dump(data, f)

# Unpickle from file
with open('data.pkl', 'rb') as f:
    loaded = pickle.load(f)

print(f"Loaded from file: {loaded}")
```

### Protocol Version Comparison

```python
import pickle
import sys

data = {
    'integers': list(range(1000)),
    'strings': ['hello'] * 1000,
    'nested': {'key' * 10: 'value' * 100}
}

print("Protocol comparison:")
for protocol in range(6):
    try:
        pickled = pickle.dumps(data, protocol=protocol)
        size = len(pickled)
        print(f"  Protocol {protocol}: {size} bytes")
    except Exception as e:
        print(f"  Protocol {protocol}: Error - {e}")
```

## Beginner Examples

### Example 1: Simple Object Persistence

```python
import pickle
import os

DATA_FILE = 'user_prefs.pkl'

def save_preferences(prefs):
    with open(DATA_FILE, 'wb') as f:
        pickle.dump(prefs, f, protocol=pickle.HIGHEST_PROTOCOL)
    print(f"Preferences saved to {DATA_FILE}")

def load_preferences():
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, 'rb') as f:
            prefs = pickle.load(f)
        return prefs
    return None

def get_default_preferences():
    return {
        'theme': 'light',
        'language': 'en',
        'font_size': 12,
        'auto_save': True,
        'window_position': (100, 100),
        'recent_files': []
    }

preferences = get_default_preferences()
preferences['theme'] = 'dark'
preferences['recent_files'].append('/path/to/file.txt')
save_preferences(preferences)

loaded = load_preferences()
if loaded:
    print(f"Loaded preferences: {loaded}")
```

### Example 2: Saving Game State

```python
import pickle
import os

class GameState:
    def __init__(self):
        self.player = {
            'name': '',
            'level': 1,
            'health': 100,
            'max_health': 100,
            'experience': 0,
            'inventory': [],
            'position': {'x': 0, 'y': 0}
        }
        self.enemies = []
        self.quests = []
        self.game_time = 0
    
    def add_item(self, item):
        self.player['inventory'].append(item)
    
    def gain_experience(self, points):
        self.player['experience'] += points
        if self.player['experience'] >= self.player['level'] * 100:
            self.level_up()
    
    def level_up(self):
        self.player['level'] += 1
        self.player['max_health'] += 20
        self.player['health'] = self.player['max_health']
        print(f"Level up! You are now level {self.player['level']}")
    
    def save(self, filename='savegame.pkl'):
        with open(filename, 'wb') as f:
            pickle.dump(self, f, protocol=pickle.HIGHEST_PROTOCOL)
        print(f"Game saved to {filename}")
    
    @staticmethod
    def load(filename='savegame.pkl'):
        if os.path.exists(filename):
            with open(filename, 'rb') as f:
                return pickle.load(f)
        return GameState()

game = GameState.load()
game.player['name'] = input("Enter your name: ") or 'Hero'
game.add_item('Sword')
game.add_item('Shield')
game.gain_experience(50)
game.save()

loaded_game = GameState.load()
print(f"Loaded game: {loaded_game.player['name']} (Level {loaded_game.player['level']})")
print(f"Inventory: {loaded_game.player['inventory']}")
```

### Example 3: Caching with Pickle

```python
import pickle
import os
import time
import hashlib

CACHE_DIR = 'cache'

def ensure_cache_dir():
    if not os.path.exists(CACHE_DIR):
        os.makedirs(CACHE_DIR)

def get_cache_key(func_name, *args, **kwargs):
    key_data = f"{func_name}:{args}:{sorted(kwargs.items())}"
    return hashlib.md5(key_data.encode()).hexdigest()

def cached(func):
    def wrapper(*args, **kwargs):
        ensure_cache_dir()
        cache_key = get_cache_key(func.__name__, *args, **kwargs)
        cache_file = os.path.join(CACHE_DIR, f"{cache_key}.pkl")
        
        if os.path.exists(cache_file):
            with open(cache_file, 'rb') as f:
                result, timestamp = pickle.load(f)
            age = time.time() - timestamp
            print(f"Cache hit! ({age:.1f}s old)")
            return result
        
        print("Cache miss, computing...")
        result = func(*args, **kwargs)
        
        with open(cache_file, 'wb') as f:
            pickle.dump((result, time.time()), f, protocol=pickle.HIGHEST_PROTOCOL)
        
        return result
    return wrapper

@cached
def expensive_computation(n):
    time.sleep(2)
    return sum(i * i for i in range(n))

print("First call (will compute):")
result1 = expensive_computation(1000000)
print(f"Result: {result1}")

print("\nSecond call (should use cache):")
result2 = expensive_computation(1000000)
print(f"Result: {result2}")
```

## Intermediate Examples

### Example 4: Pickling Custom Classes

```python
import pickle
from datetime import datetime

class User:
    def __init__(self, user_id, name, email):
        self.user_id = user_id
        self.name = name
        self.email = email
        self.created_at = datetime.now()
        self.is_active = True
        self.permissions = set()
    
    def add_permission(self, perm):
        self.permissions.add(perm)
    
    def __repr__(self):
        return f"User({self.user_id}, {self.name}, active={self.is_active})"

class UserManager:
    def __init__(self):
        self.users = {}
    
    def add_user(self, user):
        self.users[user.user_id] = user
    
    def get_user(self, user_id):
        return self.users.get(user_id)
    
    def save(self, filename):
        with open(filename, 'wb') as f:
            pickle.dump(self, f, protocol=pickle.HIGHEST_PROTOCOL)
        print(f"UserManager saved with {len(self.users)} users")
    
    @staticmethod
    def load(filename):
        with open(filename, 'rb') as f:
            return pickle.load(f)

manager = UserManager()
user1 = User(1, 'Alice', 'alice@example.com')
user1.add_permission('read')
user1.add_permission('write')

user2 = User(2, 'Bob', 'bob@example.com')
user2.add_permission('read')

manager.add_user(user1)
manager.add_user(user2)
manager.save('users.pkl')

loaded_manager = UserManager.load('users.pkl')
for uid, user in loaded_manager.users.items():
    print(f"Loaded: {user}")
    print(f"  Permissions: {user.permissions}")
```

### Example 5: Pickle with __getstate__ and __setstate__

```python
import pickle

class SecureConnection:
    def __init__(self, host, port, password):
        self.host = host
        self.port = port
        self.password = password
        self._connection = None
        self._connect()
    
    def _connect(self):
        print(f"Connecting to {self.host}:{self.port} (simulated)")
        self._connection = f"Connection-{self.host}-{self.port}"
    
    def __getstate__(self):
        state = self.__dict__.copy()
        state['_connection'] = None
        state.pop('password', None)
        return state
    
    def __setstate__(self, state):
        self.__dict__.update(state)
        self._connect()
    
    def query(self, sql):
        if not self._connection:
            self._connect()
        print(f"Executing: {sql} on {self._connection}")
        return "results"

conn = SecureConnection('db.example.com', 5432, 'supersecret')
result = conn.query('SELECT * FROM users')
print(f"Query result: {result}")

pickled = pickle.dumps(conn)
print(f"\nPickled size: {len(pickled)} bytes")
print("Note: Password is NOT in the pickled data!")

restored = pickle.loads(pickled)
result2 = restored.query('SELECT * FROM orders')
print(f"Query after restore: {result2}")
```

### Example 6: Compressed Pickle for Large Objects

```python
import pickle
import gzip
import lzma
import os

def save_compressed(obj, filename, compression='gzip', protocol=pickle.HIGHEST_PROTOCOL):
    pickled = pickle.dumps(obj, protocol=protocol)
    
    if compression == 'gzip':
        compressed = gzip.compress(pickled)
    elif compression == 'lzma':
        compressed = lzma.compress(pickled)
    else:
        compressed = pickled
    
    with open(filename, 'wb') as f:
        f.write(compressed)
    
    return len(pickled), len(compressed)

def load_compressed(filename, compression='gzip'):
    with open(filename, 'rb') as f:
        compressed = f.read()
    
    if compression == 'gzip':
        decompressed = gzip.decompress(compressed)
    elif compression == 'lzma':
        decompressed = lzma.decompress(compressed)
    else:
        decompressed = compressed
    
    return pickle.loads(decompressed)

import numpy as np

large_data = {
    'matrix': np.random.rand(1000, 100).tolist(),
    'metadata': {'created': '2024-01-15', 'version': '2.0'},
    'values': list(range(100000))
}

for comp in ['none', 'gzip', 'lzma']:
    filename = f'data_{comp}.pkl'
    if comp == 'none':
        raw, comp_size = save_compressed(large_data, filename, compression='none')
    else:
        raw, comp_size = save_compressed(large_data, filename, compression=comp)
    
    file_size = os.path.getsize(filename)
    ratio = comp_size / raw * 100
    print(f"{comp}: raw={raw/1024:.1f}KB -> compressed={comp_size/1024:.1f}KB ({ratio:.1f}%)")

loaded = load_compressed('data_gzip.pkl', compression='gzip')
print(f"\nLoaded data type: {type(loaded)}")
print(f"Matrix shape: {len(loaded['matrix'])}x{len(loaded['matrix'][0])}")
print(f"Values length: {len(loaded['values'])}")
```

## Advanced Examples

### Example 7: Secure Unpickling with Restricted Execution

```python
import pickle
import builtins

class RestrictedUnpickler(pickle.Unpickler):
    SAFE_CLASSES = {
        'builtins': {'list', 'dict', 'tuple', 'set', 'str', 'int', 'float', 'bool', 'bytes', 'bytearray', 'type', 'range', 'slice', 'object', 'property'},
        'collections': {'OrderedDict', 'defaultdict', 'Counter', 'namedtuple'},
        'datetime': {'datetime', 'date', 'time', 'timedelta'},
        'decimal': {'Decimal'},
        'fractions': {'Fraction'},
    }
    
    def find_class(self, module, name):
        if module in self.SAFE_CLASSES and name in self.SAFE_CLASSES[module]:
            return super().find_class(module, name)
        
        for safe_module, safe_names in self.SAFE_CLASSES.items():
            if name in safe_names:
                return super().find_class(safe_module, name)
        
        raise pickle.UnpicklingError(f"Forbidden class: {module}.{name}")

def safe_loads(data):
    return RestrictedUnpickler(io.BytesIO(data)).load()

def safe_load(file_obj):
    return RestrictedUnpickler(file_obj).load()

# Test with safe data
import io

safe_data = {'name': 'Alice', 'scores': [1, 2, 3]}
pickled = pickle.dumps(safe_data)
restored = safe_loads(pickled)
print(f"Safe load successful: {restored}")

# Test with restricted classes
try:
    malicious_data = b"cos\nsystem\n(S'echo hacked'\ntR."
    safe_loads(malicious_data)
    print("Malicious data loaded! (bad)")
except pickle.UnpicklingError as e:
    print(f"Restricted class blocked: {e}")
```

### Example 8: Custom Pickle Protocol with Versioning

```python
import pickle
import hashlib

class VersionedObject:
    def __init__(self, name, data):
        self.name = name
        self.data = data
        self._version = 1
    
    def __getstate__(self):
        state = self.__dict__.copy()
        state['_format_version'] = 2
        state['_checksum'] = hashlib.md5(str(self.data).encode()).hexdigest()
        return state
    
    def __setstate__(self, state):
        format_version = state.pop('_format_version', 1)
        checksum = state.pop('_checksum', None)
        
        self.__dict__.update(state)
        self._version = format_version
        
        if format_version == 1:
            self._migrate_v1_to_v2()
        
        if checksum and hashlib.md5(str(self.data).encode()).hexdigest() != checksum:
            print("Warning: Data integrity check failed!")
    
    def _migrate_v1_to_v2(self):
        if isinstance(self.data, list):
            self.data = {'items': self.data, 'count': len(self.data)}
        self._version = 2
        print(f"Migrated {self.name} from v1 to v2")

obj_v1 = VersionedObject('test', [1, 2, 3])
with open('versioned_v1.pkl', 'wb') as f:
    pickle.dump(obj_v1, f, protocol=2)

original = obj_v1.__getstate__()
original.pop('_format_version')
original.pop('_checksum')
with open('versioned_v1.pkl', 'wb') as f:
    pickle.dump(original, f, protocol=2)

with open('versioned_v1.pkl', 'rb') as f:
    obj_loaded = pickle.load(f)

print(f"Object: {obj_loaded}")
print(f"Name: {obj_loaded.name}")
print(f"Data: {obj_loaded.data}")
print(f"Version: {obj_loaded._version}")
```

## Real-World Use Cases

1. **Machine Learning Model Persistence**: Serializing trained models from scikit-learn, PyTorch, or TensorFlow for deployment in production environments.

2. **Distributed Computing**: Transferring task definitions and results between worker processes in frameworks like multiprocessing, Celery, or Dask.

3. **Application State Management**: Saving and restoring complex application state including undo history, window layouts, and user sessions.

4. **Caching and Memoization**: Caching computationally expensive results with pickle, especially when results include complex Python objects.

5. **Scientific Computing Checkpoints**: Saving intermediate computation results for long-running scientific simulations to enable recovery from failures.

6. **Queue Systems**: Serializing messages in job queues like Redis Queue (RQ) or Celery when JSON is insufficient.

7. **Data Pipeline Artifacts**: Passing complex data structures between stages of a data processing pipeline.

## Common Mistakes

1. **Unpickling Untrusted Data**: This is the most dangerous mistake. Unpickling malicious data can execute arbitrary code. Never unpickle data from untrusted sources.

2. **Using Pickle for Long-Term Storage**: Pickle format can change between Python versions, and classes being pickled may change over time, breaking deserialization.

3. **Pickling Objects with External Dependencies**: Objects that depend on external resources (file handles, network connections, database connections) should implement `__getstate__` and `__setstate__`.

4. **Forgetting to Open in Binary Mode**: Pickle files must be opened with 'wb' or 'rb' mode, not text mode.

5. **Using Default Protocol**: Always specify `protocol=pickle.HIGHEST_PROTOCOL` for maximum efficiency.

## Best Practices

1. **Never Unpickle Untrusted Data**: Only unpickle data from sources you trust. Consider alternatives like JSON for data exchange between different systems.

2. **Use Highest Protocol**: Always use `protocol=pickle.HIGHEST_PROTOCOL` for smaller and faster serialization.

3. **Implement __getstate__ and __setstate__**: Control what gets pickled and how objects are restored, especially for objects with non-serializable attributes.

4. **Version Your Pickled Data**: Include a version number in your pickled data to handle format migrations gracefully.

5. **Use Compression for Large Objects**: Combine pickle with gzip or lzma for large objects to reduce storage and transmission costs.

6. **Consider Alternatives**: For simple data structures, consider JSON, msgpack, or other cross-language formats instead of pickle.

## Interview Questions

**Q1: What types can pickle serialize?**

A: Pickle can serialize: None, booleans, integers, floats, complex numbers, strings, bytes, bytearrays, tuples, lists, sets, dictionaries, functions (at module level), classes (at module level), and instances of classes where `__getstate__` is defined or all attributes are picklable.

**Q2: What are the pickle protocol versions?**

A: Protocol 0 (text-based, human-readable), Protocol 1 (old binary), Protocol 2 (Python 2.3+), Protocol 3 (default for Python 3.0-3.7), Protocol 4 (default for Python 3.8+, supports large objects), Protocol 5 (Python 3.8+, out-of-band data).

**Q3: Why is pickle insecure?**

A: Unpickling can execute arbitrary Python code because pickle can reconstruct arbitrary objects, including those that execute code in their `__reduce__` method. A malicious pickle can execute system commands.

**Q4: What's the difference between pickle and JSON?**

A: Pickle is Python-specific, supports all Python types (including custom classes), and produces binary output. JSON is language-agnostic, supports only basic data types, produces text output, and is safe to use with untrusted data.

## Coding Challenges

**Challenge 1: Versioned Configuration Manager** - Create a configuration system that handles pickle format versioning and migration.

**Challenge 2: Secure Pickle Proxy** - Implement a wrapper that validates pickled data against an allowlist of allowed types before unpickling.

**Challenge 3: Incremental Pickle** - Design a system that can append new objects to an existing pickle file without reading the entire file.

**Challenge 4: Pickle Analyzer** - Write a tool that analyzes a pickle file and reports the types and structure it contains without unpickling it.

**Challenge 5: Hybrid Serializer** - Create a serializer that uses pickle for complex types but falls back to JSON for simple types.

## Summary

Python's `pickle` module provides powerful serialization capabilities for Python objects, supporting everything from basic types to complex custom class instances with circular references. The module supports multiple protocol versions (0-5) with higher versions offering better efficiency. Security is a critical concern - never unpickle untrusted data. Control serialization with `__getstate__` and `__setstate__`, and use compression for large objects. While pickle is ideal for Python-to-Python serialization, cross-language scenarios should use JSON, msgpack, or other interchange formats.

## Related Topics

- `48_file_operations.md` - File I/O basics for reading/writing pickle files
- `49_json.md` - JSON serialization as a cross-language alternative
- `52_pathlib.md` - Path management for pickle file locations
- `53_temp_files.md` - Temporary files for intermediate pickle storage
- Data Serialization - Comparing pickle with JSON, YAML, msgpack, protobuf
- shelve module - Persistent dictionary backed by pickle
- copy module - Deep/shallow copy operations using pickle
