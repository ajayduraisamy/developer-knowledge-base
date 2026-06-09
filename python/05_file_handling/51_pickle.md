# Pickle - pickle.dump(), pickle.load(), serialization protocol

## Introduction

Pickle is Python's built-in module for serializing and deserializing Python object structures. Serialization ("pickling") converts Python objects into a byte stream, and deserialization ("unpickling") reconstructs the original objects. Unlike JSON (language-agnostic, basic types only), pickle is Python-specific and can handle virtually any Python object, including custom class instances, nested structures, and circular references. However, this Python-specific nature introduces significant security considerations.

## pickle.dump()

### What It Is

`pickle.dump()` serializes a Python object hierarchy and writes it to a file-like object. It converts the object into a byte stream using the specified protocol version. The resulting file can be read back with `pickle.load()` to reconstruct the original object.

### Why It Is Important

`pickle.dump()` enables persistent storage of complex Python objects. Machine learning models, application state, cached computations, and complex data structures can all be saved to disk and restored later. For Python-to-Python serialization, pickle is often the simplest and most comprehensive option.

### How It Works Internally

`pickle.dump()` recursively traverses the object graph, emitting opcodes that describe the object structure. The pickler maintains a "memo" dictionary to handle shared and circular references efficiently. Each object type is handled by a specific "reducer" function that knows how to decompose that type into picklable components. The protocol version determines which opcodes are available—higher protocols use more efficient representations.

### Syntax

```python
import pickle

# Serialize to file
with open("data.pkl", "wb") as f:
    pickle.dump(obj, f, protocol=None, fix_imports=True, buffer_callback=None)

# Serialize to bytes
bytes_data = pickle.dumps(obj, protocol=None, fix_imports=True, buffer_callback=None)

# Protocol versions:
# 0 - Text protocol (human-readable)
# 1 - Old binary format
# 2 - Python 2.3+ format
# 3 - Default for Python 3.0-3.7
# 4 - Default for Python 3.8+ (large objects, non-GC frames)
# 5 - Python 3.8+ (out-of-band data, zero-copy)
# pickle.HIGHEST_PROTOCOL - Auto-select highest available
```

```python
import pickle

data = {"name": "Alice", "scores": [95, 87, 91]}

# Write to file
with open("data.pkl", "wb") as f:
    pickle.dump(data, f, protocol=pickle.HIGHEST_PROTOCOL)

# Write to bytes
pickled = pickle.dumps(data)
print(f"Size: {len(pickled)} bytes")
```

### Beginner Examples

```python
import pickle

# Saving simple data
user_prefs = {
    "theme": "dark",
    "language": "en",
    "font_size": 12,
    "recent_files": ["file1.txt", "file2.txt"],
}

with open("prefs.pkl", "wb") as f:
    pickle.dump(user_prefs, f)

# Saving multiple objects
data1 = [1, 2, 3]
data2 = {"key": "value"}
data3 = "hello"

with open("multi.pkl", "wb") as f:
    pickle.dump(data1, f)
    pickle.dump(data2, f)
    pickle.dump(data3, f)

# Using protocol for compatibility
with open("data_v2.pkl", "wb") as f:
    pickle.dump(data, f, protocol=2)  # Python 2 compatible

# Always use HIGHEST_PROTOCOL for new code
with open("data.pkl", "wb") as f:
    pickle.dump(data, f, protocol=pickle.HIGHEST_PROTOCOL)

# Saving a class instance
class User:
    def __init__(self, name, score):
        self.name = name
        self.score = score

user = User("Alice", 95)
with open("user.pkl", "wb") as f:
    pickle.dump(user, f)
```

## pickle.load()

### What It Is

`pickle.load()` deserializes a byte stream from a file-like object and reconstructs the original Python object hierarchy. It reverses the pickling process, restoring objects to their original state.

### Why It Is Important

`pickle.load()` enables restoring saved state, loading cached computations, and deserializing objects sent over network connections. It's essential for machine learning model deployment, session restoration, and checkpoint resumption.

### How It Works Internally

`pickle.load()` reads opcodes from the byte stream and executes them to reconstruct objects. It maintains a stack and a memo during unpickling. When it encounters a `GLOBAL` opcode referencing a class, it imports that class using `find_class()`. This is where the security risk comes from—a malicious pickle can reference arbitrary classes and execute arbitrary code during unpickling.

### Syntax

```python
import pickle

# Deserialize from file
with open("data.pkl", "rb") as f:
    obj = pickle.load(f, fix_imports=True, encoding="ASCII", errors="strict")

# Deserialize from bytes
obj = pickle.loads(bytes_data, fix_imports=True, encoding="ASCII", errors="strict")

# Loading multiple objects
with open("multi.pkl", "rb") as f:
    obj1 = pickle.load(f)
    obj2 = pickle.load(f)
    obj3 = pickle.load(f)
```

### Beginner Examples

```python
import pickle

# Loading simple data
with open("prefs.pkl", "rb") as f:
    user_prefs = pickle.load(f)
print(user_prefs["theme"])  # "dark"

# Loading class instances
with open("user.pkl", "rb") as f:
    user = pickle.load(f)
print(user.name, user.score)  # Alice 95

# Loading multiple objects
with open("multi.pkl", "rb") as f:
    while True:
        try:
            obj = pickle.load(f)
            print(f"Loaded: {obj}")
        except EOFError:
            break

# Error handling
try:
    with open("data.pkl", "rb") as f:
        data = pickle.load(f)
except (pickle.UnpicklingError, FileNotFoundError) as e:
    print(f"Failed to load: {e}")

# Checking file existence
import os
if os.path.exists("data.pkl"):
    with open("data.pkl", "rb") as f:
        data = pickle.load(f)
```

## Serialization protocol

### What It Is

The pickle protocol defines the format and opcodes used to represent serialized objects. There are six protocol versions (0-5), each adding features and efficiency improvements. Higher protocols produce smaller, faster-to-process byte streams but may not be readable by older Python versions.

### Why It Is Important

Choosing the right protocol balances compatibility and efficiency. Protocol 0 is human-readable but slow and large. Protocol 4 (default in Python 3.8+) supports large objects and is efficient. Protocol 5 adds out-of-band data support for zero-copy serialization.

### How It Works Internally

Each protocol version is a set of opcode numbers and their semantics. The pickler emits opcodes that describe types and their structure: `NONE` for None, `INT` for integers, `STRING` for strings, `MARK` for framing, `PUT`/`GET` for memo operations, `REDUCE` for calling constructors, `STACK_GLOBAL` for importing classes. Higher protocols batch operations, use more compact representations for common types, and add support for features like bytearray and out-of-band data.

### Syntax

```python
import pickle

# Protocol comparison
data = {"list": list(range(1000))}

for protocol in range(6):
    try:
        pickled = pickle.dumps(data, protocol=protocol)
        print(f"Protocol {protocol}: {len(pickled)} bytes")
    except Exception:
        print(f"Protocol {protocol}: not supported")

# Using highest protocol
with open("data.pkl", "wb") as f:
    pickle.dump(data, f, protocol=pickle.HIGHEST_PROTOCOL)
```

### Beginner Examples

```python
import pickle

# Text protocol (0) - human readable
data = {"name": "Alice", "score": 95}
text_pickle = pickle.dumps(data, protocol=0)
print(text_pickle.decode())  # Human-readable format

# Binary protocol comparison
small_data = {"a": 1, "b": 2}
for protocol in range(6):
    try:
        size = len(pickle.dumps(small_data, protocol=protocol))
        print(f"P{protocol}: {size}B")
    except:
        pass

# Default protocol in different Python versions
print(f"Default: {pickle.DEFAULT_PROTOCOL}")
print(f"Highest: {pickle.HIGHEST_PROTOCOL}")
```

### Intermediate Examples

```python
# Controlling serialization with __getstate__ / __setstate__
class SecureConnection:
    def __init__(self, host, port, password):
        self.host = host
        self.port = port
        self.password = password
        self._connection = None

    def __getstate__(self):
        state = self.__dict__.copy()
        state["password"] = None  # Don't serialize password
        state["_connection"] = None  # Don't serialize connection
        return state

    def __setstate__(self, state):
        self.__dict__.update(state)
        self._connection = self._reconnect()

    def _reconnect(self):
        return f"Connected to {self.host}:{self.port}"

# Compressed pickle for large objects
import gzip
import lzma

def save_compressed(obj, filename, compression="gzip"):
    pickled = pickle.dumps(obj, protocol=pickle.HIGHEST_PROTOCOL)
    if compression == "gzip":
        compressed = gzip.compress(pickled)
    elif compression == "lzma":
        compressed = lzma.compress(pickled)
    else:
        compressed = pickled
    with open(filename, "wb") as f:
        f.write(compressed)

def load_compressed(filename, compression="gzip"):
    with open(filename, "rb") as f:
        compressed = f.read()
    if compression == "gzip":
        return pickle.loads(gzip.decompress(compressed))
    elif compression == "lzma":
        return pickle.loads(lzma.decompress(compressed))
    else:
        return pickle.loads(compressed)
```

### Advanced Examples

```python
# Restricted unpickler for security
import io
import builtins

class RestrictedUnpickler(pickle.Unpickler):
    SAFE_CLASSES = {
        "builtins": {"list", "dict", "tuple", "set", "str", "int",
                     "float", "bool", "bytes", "bytearray", "range",
                     "slice", "object", "property", "type"},
        "collections": {"OrderedDict", "defaultdict", "Counter"},
        "datetime": {"datetime", "date", "time", "timedelta"},
    }

    def find_class(self, module, name):
        if module in self.SAFE_CLASSES and name in self.SAFE_CLASSES[module]:
            return super().find_class(module, name)
        raise pickle.UnpicklingError(f"Forbidden: {module}.{name}")

def safe_loads(data):
    return RestrictedUnpickler(io.BytesIO(data)).load()

def safe_load(file_obj):
    return RestrictedUnpickler(file_obj).load()

# Versioned pickling
class VersionedObject:
    def __init__(self, name, data):
        self.name = name
        self.data = data
        self._version = 1

    def __getstate__(self):
        state = self.__dict__.copy()
        state["_format_version"] = 2
        return state

    def __setstate__(self, state):
        version = state.pop("_format_version", 1)
        self.__dict__.update(state)
        if version == 1 and isinstance(self.data, list):
            self.data = {"items": self.data, "count": len(self.data)}
        self._version = version

# Pickle with out-of-band data (protocol 5)
import numpy as np

class NumpyArrayHandler:
    def __init__(self):
        self.buffers = []

    def __reduce_ex__(self, protocol):
        if protocol >= 5:
            return (self._reconstruct, (self.buffers,), None, self.buffers)
        return (self._reconstruct, (list(self.buffers),))

    @staticmethod
    def _reconstruct(buffers):
        return NumpyArrayHandler(buffers)

    def __init__(self, buffers=None):
        self.buffers = buffers or []
```

### Real-World Use Cases

```python
# ML model persistence
import pickle
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier()
model.fit(X_train, y_train)

with open("model.pkl", "wb") as f:
    pickle.dump(model, f)

with open("model.pkl", "rb") as f:
    loaded_model = pickle.load(f)

predictions = loaded_model.predict(X_test)

# Caching with pickle
import hashlib
import os
import time

def cached(func):
    os.makedirs(".pickle_cache", exist_ok=True)
    def wrapper(*args, **kwargs):
        key = hashlib.md5(
            f"{func.__name__}:{args}:{sorted(kwargs.items())}".encode()
        ).hexdigest()
        cache_file = f".pickle_cache/{key}.pkl"
        if os.path.exists(cache_file):
            result, _ = pickle.load(open(cache_file, "rb"))
            return result
        result = func(*args, **kwargs)
        pickle.dump((result, time.time()), open(cache_file, "wb"))
        return result
    return wrapper

# Application state management
class ApplicationState:
    def __init__(self, state_file="state.pkl"):
        self.state_file = state_file
        self.state = self._load()

    def _load(self):
        if os.path.exists(self.state_file):
            with open(self.state_file, "rb") as f:
                return pickle.load(f)
        return {"windows": [], "preferences": {}}

    def save(self):
        with open(self.state_file, "wb") as f:
            pickle.dump(self.state, f, protocol=pickle.HIGHEST_PROTOCOL)

    def update(self, key, value):
        self.state[key] = value
        self.save()
```

### Common Mistakes

```python
# Mistake 1: Unpickling untrusted data
# NEVER unpickle data from untrusted sources - it can execute arbitrary code
# A malicious pickle:
# >>> import pickle
# >>> pickle.loads(b"cos\\nsystem\\n(S'echo hacked'\\ntR.")

# Mistake 2: Opening pickle files without 'b' mode
# Text mode corrupts the binary pickle data
with open("data.pkl", "r") as f:  # Wrong! Use "rb"
    data = pickle.load(f)

# Mistake 3: Using default protocol for storage
# Default may be older than highest protocol
pickle.dump(data, f)  # Might use protocol 3 or 4
# Better:
pickle.dump(data, f, protocol=pickle.HIGHEST_PROTOCOL)

# Mistake 4: Pickling objects with unpicklable attributes
class BadObject:
    def __init__(self):
        self.file = open("data.txt")  # File objects aren't picklable

# Fix with __getstate__
class GoodObject:
    def __init__(self):
        self.file = open("data.txt")

    def __getstate__(self):
        state = self.__dict__.copy()
        state["file"] = None  # Don't serialize the file
        return state

# Mistake 5: Using pickle for long-term storage
# Classes change over time, and old pickles may not load with new class definitions

# Mistake 6: Not handling version migration
# Pickled data can become incompatible when classes are modified
```

### Best Practices

- Never unpickle data from untrusted sources
- Always use `protocol=pickle.HIGHEST_PROTOCOL`
- Implement `__getstate__`/`__setstate__` for objects with non-picklable attributes
- Version your pickled data for forward compatibility
- Use compression for large objects (gzip or lzma)
- Prefer JSON or other cross-language formats for data exchange
- Store class definitions in a stable location when pickling instances
- Consider `shelve` for persistent dictionary-like storage
- Test unpickling after class modifications
- Use restricted unpicklers for partially trusted data

### Performance Considerations

- Higher protocols produce smaller, faster-to-process output
- Protocol 0 (text) is 2-5x slower than binary protocols
- Pickle is generally faster than JSON for complex Python objects
- The serialization speed depends on object complexity
- For numeric arrays, consider `numpy.save()` or `array.array`
- Compression (gzip) reduces storage at cost of CPU
- Protocol 5 supports zero-copy pickling for large buffers

### Interview Questions

1. What types can pickle serialize?

   None, booleans, ints, floats, complex numbers, strings, bytes, bytearrays, tuples, lists, sets, dicts, functions (module-level), classes (module-level), and class instances with picklable attributes.

2. What are the pickle protocol versions?

   0 (text), 1 (binary), 2 (Python 2.3+), 3 (Python 3.0-3.7 default), 4 (Python 3.8+ default, large objects), 5 (Python 3.8+, out-of-band data).

3. Why is pickle insecure?

   Unpickling can execute arbitrary code because pickle reconstructs objects using `REDUCE` and other opcodes that can call any function. A malicious pickle can execute system commands via `os.system`.

4. Difference between pickle and JSON?

   Pickle is Python-specific, supports all Python types, produces binary. JSON is language-agnostic, supports basic types only, produces text, safe with untrusted data.

5. What are `__getstate__` and `__setstate__`?

   `__getstate__` controls what gets serialized (return value replaces instance dict). `__setstate__` controls how objects are restored from their pickled state.

### Coding Challenges

1. Create a versioned configuration system that handles pickle format migration.

2. Implement a secure pickle wrapper that validates unpickled types against an allowlist.

3. Build an incremental pickle system that appends new objects without reading the entire file.

4. Write a pickle analyzer that inspects pickle file contents without unpickling.

5. Create a hybrid serializer that uses pickle for complex types and JSON for simple types.

### Related Topics

- `48_file_operations.md` - File I/O basics
- `49_json.md` - Cross-language serialization alternative
- `52_pathlib.md` - Path management
- `53_temp_files.md` - Temporary file storage
- `shelve` module - Persistent dict backed by pickle
- `copy` module - Deep/shallow copy using pickle
- `__reduce__`/`__reduce_ex__` - Custom pickling protocols
