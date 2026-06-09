# Adapter - Interface compatibility, class/object adapter patterns
## Introduction
The Adapter pattern allows objects with incompatible interfaces to collaborate. It acts as a wrapper that translates one interface into another expected by the client. This pattern is essential when integrating legacy code, third-party libraries, or systems with different interfaces. Python's duck typing and dynamic nature make Adapter implementation particularly elegant, often requiring minimal code.

## Interface compatibility
### What It Is
Interface incompatibility occurs when a client expects a certain API but the available class provides a different one. The Adapter bridges this gap by wrapping the adaptee (the class with the incompatible interface) and presenting the expected interface to the client.

### Why It Is Important
Adapter enables integration of components that weren't designed to work together. It allows reusing existing code without modification, which is crucial for working with legacy systems, third-party libraries, or multiple implementations of similar functionality.

### How It Works Internally
The client calls methods on the Adapter using the target interface. The Adapter translates these calls into the adaptee's interface. The translation may involve method renaming, parameter conversion, result formatting, or combining multiple calls.

```python
from abc import ABC, abstractmethod

# Target interface (what the client expects)
class MediaPlayer(ABC):
    @abstractmethod
    def play(self, audio_type: str, filename: str) -> str:
        pass

# Adaptee 1: MP3 player with different interface
class MP3Player:
    def play_mp3(self, file_path: str) -> str:
        return f"Playing MP3 file: {file_path}"

# Adaptee 2: WAV player with different interface
class WAVPlayer:
    def play_wav(self, file_path: str) -> str:
        return f"Playing WAV file: {file_path}"

# Adaptee 3: FLAC player with different interface
class FLACPlayer:
    def play_flac(self, file_path: str) -> str:
        return f"Playing FLAC file: {file_path}"

# Adapters
class MP3Adapter(MediaPlayer):
    def __init__(self):
        self._player = MP3Player()

    def play(self, audio_type: str, filename: str) -> str:
        if audio_type.lower() != "mp3":
            raise ValueError(f"MP3Adapter cannot play {audio_type}")
        return self._player.play_mp3(filename)

class WAVAdapter(MediaPlayer):
    def __init__(self):
        self._player = WAVPlayer()

    def play(self, audio_type: str, filename: str) -> str:
        if audio_type.lower() != "wav":
            raise ValueError(f"WAVAdapter cannot play {audio_type}")
        return self._player.play_wav(filename)

class FLACAdapter(MediaPlayer):
    def __init__(self):
        self._player = FLACPlayer()

    def play(self, audio_type: str, filename: str) -> str:
        if audio_type.lower() != "flac":
            raise ValueError(f"FLACAdapter cannot play {audio_type}")
        return self._player.play_flac(filename)

# Client
class AudioClient:
    def __init__(self):
        self._adapters = {
            "mp3": MP3Adapter(),
            "wav": WAVAdapter(),
            "flac": FLACAdapter(),
        }

    def play_audio(self, audio_type: str, filename: str) -> str:
        adapter = self._adapters.get(audio_type.lower())
        if not adapter:
            raise ValueError(f"Unsupported audio type: {audio_type}")
        return adapter.play(audio_type, filename)

# Usage
client = AudioClient()
print(client.play_audio("mp3", "song.mp3"))
print(client.play_audio("wav", "sound.wav"))
print(client.play_audio("flac", "track.flac"))
```

## Class adapter
### What It Is
The Class Adapter uses inheritance (multiple inheritance in Python) to adapt one interface to another. It inherits the target interface and the adaptee's implementation simultaneously, combining them into a single class.

### Why It Is Important
Class adapters don't require a separate adapter object, which can be more memory-efficient. In Python, multiple inheritance makes class adapters straightforward to implement.

### How It Works Internally
The class adapter inherits from both the target interface and the adaptee class. It implements the target interface methods by delegating to inherited adaptee methods (possibly with translation).

```python
class USBConnector:
    def connect_usb(self):
        return "USB device connected"

    def transfer_data_usb(self, data):
        return f"USB transferring: {data}"

class HDMIConnector:
    def connect_hdmi(self):
        return "HDMI device connected"

    def transfer_data_hdmi(self, data):
        return f"HDMI transferring: {data}"

# Target interfaces
class DisplayInterface:
    def connect(self):
        raise NotImplementedError

    def display(self, data):
        raise NotImplementedError

# Class adapters using multiple inheritance
class USBToDisplayAdapter(USBConnector, DisplayInterface):
    def connect(self):
        return self.connect_usb()

    def display(self, data):
        return self.transfer_data_usb(data)

class HDMIToDisplayAdapter(HDMIConnector, DisplayInterface):
    def connect(self):
        return self.connect_hdmi()

    def display(self, data):
        return self.transfer_data_hdmi(data)

# Client expects DisplayInterface
class Computer:
    def __init__(self, display: DisplayInterface):
        self._display = display

    def show(self, data):
        print(self._display.connect())
        print(self._display.display(data))

# Usage
computer = Computer(USBToDisplayAdapter())
computer.show("Hello via USB")

computer = Computer(HDMIToDisplayAdapter())
computer.show("Hello via HDMI")
```

### Class Adapter for Third-Party Library
```python
# Third-party library (cannot be modified)
class LegacyLogger:
    def write_log(self, level, message):
        return f"[{level.upper()}] {message}"

    def set_log_level(self, level):
        pass

# Target interface used by our application
class LoggerInterface:
    def debug(self, message):
        raise NotImplementedError

    def info(self, message):
        raise NotImplementedError

    def warning(self, message):
        raise NotImplementedError

    def error(self, message):
        raise NotImplementedError

# Class adapter
class LegacyLoggerAdapter(LegacyLogger, LoggerInterface):
    def debug(self, message):
        self.set_log_level("debug")
        return self.write_log("debug", message)

    def info(self, message):
        self.set_log_level("info")
        return self.write_log("info", message)

    def warning(self, message):
        self.set_log_level("warning")
        return self.write_log("warning", message)

    def error(self, message):
        self.set_log_level("error")
        return self.write_log("error", message)

# Application code
class Application:
    def __init__(self, logger: LoggerInterface):
        self._logger = logger

    def run(self):
        self._logger.info("Application started")
        self._logger.debug("Debug information")
        try:
            raise ValueError("Something went wrong")
        except ValueError as e:
            self._logger.error(str(e))

app = Application(LegacyLoggerAdapter())
app.run()
```

## Object adapter
### What It Is
The Object Adapter uses composition instead of inheritance. It holds a reference to the adaptee instance and wraps its methods to match the target interface.

### Why It Is Important
Object adapters are more flexible than class adapters because they work with any subclass of the adaptee. They don't require multiple inheritance and can adapt multiple adaptees simultaneously.

### How It Works Internally
The adapter contains an instance of the adaptee (passed via constructor). When the target interface method is called, the adapter translates and delegates to the adaptee's methods.

```python
from abc import ABC, abstractmethod

# Target interface
class TemperatureSensor(ABC):
    @abstractmethod
    def get_temperature_celsius(self) -> float:
        pass

# Adaptee 1: Fahrenheit sensor (incompatible interface)
class FahrenheitSensor:
    def __init__(self, temperature_f):
        self._temperature_f = temperature_f

    def get_temperature_fahrenheit(self) -> float:
        return self._temperature_f

# Adaptee 2: Kelvin sensor (incompatible interface)
class KelvinSensor:
    def __init__(self, temperature_k):
        self._temperature_k = temperature_k

    def get_temperature_kelvin(self) -> float:
        return self._temperature_k

# Object adapters
class FahrenheitToCelsiusAdapter(TemperatureSensor):
    def __init__(self, sensor: FahrenheitSensor):
        self._sensor = sensor

    def get_temperature_celsius(self) -> float:
        fahrenheit = self._sensor.get_temperature_fahrenheit()
        return (fahrenheit - 32) * 5.0 / 9.0

class KelvinToCelsiusAdapter(TemperatureSensor):
    def __init__(self, sensor: KelvinSensor):
        self._sensor = sensor

    def get_temperature_celsius(self) -> float:
        kelvin = self._sensor.get_temperature_kelvin()
        return kelvin - 273.15

# Client
class ClimateControl:
    def __init__(self):
        self._sensors = []

    def add_sensor(self, sensor: TemperatureSensor):
        self._sensors.append(sensor)

    def get_average_temperature(self) -> float:
        if not self._sensors:
            return 0.0
        total = sum(s.get_temperature_celsius() for s in self._sensors)
        return total / len(self._sensors)

# Usage
control = ClimateControl()
control.add_sensor(FahrenheitToCelsiusAdapter(FahrenheitSensor(77)))     # 25°C
control.add_sensor(KelvinToCelsiusAdapter(KelvinSensor(298.15)))          # 25°C
control.add_sensor(FahrenheitToCelsiusAdapter(FahrenheitSensor(68)))      # 20°C

print(f"Average temperature: {control.get_average_temperature():.1f}°C")
```

### Dual Interface Object Adapter
```python
from abc import ABC, abstractmethod

class XMLParser(ABC):
    @abstractmethod
    def parse_xml(self, xml_string):
        pass

class JSONParser(ABC):
    @abstractmethod
    def parse_json(self, json_string):
        pass

# Third-party XML library
class ThirdPartyXMLParser:
    def parse(self, xml_data):
        import xml.etree.ElementTree as ET
        root = ET.fromstring(xml_data)
        return {child.tag: child.text for child in root}

# Object adapter that provides both XML and JSON parsing
class ParserAdapter(XMLParser, JSONParser):
    def __init__(self):
        self._xml_parser = ThirdPartyXMLParser()

    def parse_xml(self, xml_string):
        return self._xml_parser.parse(xml_string)

    def parse_json(self, json_string):
        import json
        return json.loads(json_string)

# Usage
adapter = ParserAdapter()
xml_result = adapter.parse_xml("<data><name>Alice</name></data>")
json_result = adapter.parse_json('{"name": "Bob"}')
print(xml_result)  # {'name': 'Alice'}
print(json_result)  # {'name': 'Bob'}
```

### Beginner Examples
```python
# Simple adapter: adapting a European plug to US outlet
class EuropeanPlug:
    def provide_electricity(self):
        return "220V European power"

class USOutlet:
    def power_device(self, us_plug):
        print(f"US outlet: {us_plug.connect_to_us_outlet()}")

class PlugAdapter:
    def __init__(self, european_plug):
        self._plug = european_plug

    def connect_to_us_outlet(self):
        power = self._plug.provide_electricity()
        return f"Adapter converting {power} to 110V US power"

euro_plug = EuropeanPlug()
adapter = PlugAdapter(euro_plug)
outlet = USOutlet()
outlet.power_device(adapter)
```

### Intermediate Examples
```python
# Adapting different database result formats to a common interface
from typing import Dict, List, Any

class QueryResult(ABC):
    def get_rows(self) -> List[Dict[str, Any]]:
        raise NotImplementedError

class SQLiteResult:
    def __init__(self, cursor):
        self._cursor = cursor

    def fetchall_as_dicts(self):
        columns = [desc[0] for desc in self._cursor.description]
        rows = self._cursor.fetchall()
        return [dict(zip(columns, row)) for row in rows]

class PostgreSQLResult:
    def __init__(self, records):
        self._records = records

    def get_records(self):
        return self._records  # Already dicts

class MongoDBResult:
    def __init__(self, documents):
        self._documents = documents

    def to_list(self):
        return list(self._documents)

class SQLiteResultAdapter(QueryResult):
    def __init__(self, sqlite_result: SQLiteResult):
        self._result = sqlite_result

    def get_rows(self):
        return self._result.fetchall_as_dicts()

class PostgreSQLResultAdapter(QueryResult):
    def __init__(self, pg_result: PostgreSQLResult):
        self._result = pg_result

    def get_rows(self):
        return self._result.get_records()

class MongoDBResultAdapter(QueryResult):
    def __init__(self, mongo_result: MongoDBResult):
        self._result = mongo_result

    def get_rows(self):
        return [dict(doc) for doc in self._result.to_list()]

# Client works with unified interface
def process_results(result_adapter: QueryResult):
    for row in result_adapter.get_rows():
        print(f"Processing: {row}")
```

### Advanced Examples
```python
from typing import Callable, Any

# Adapter that wraps a function as a class interface
class FunctionAdapter:
    def __init__(self, func: Callable):
        self._func = func

    def execute(self, *args, **kwargs):
        return self._func(*args, **kwargs)

    def __call__(self, *args, **kwargs):
        return self.execute(*args, **kwargs)

# Adapter that converts between sync and async
import asyncio

class AsyncToSyncAdapter:
    def __init__(self, async_func):
        self._async_func = async_func

    def execute(self, *args, **kwargs):
        return asyncio.run(self._async_func(*args, **kwargs))

class SyncToAsyncAdapter:
    def __init__(self, sync_func):
        self._sync_func = sync_func

    async def execute(self, *args, **kwargs):
        return self._sync_func(*args, **kwargs)

# Usage
async def fetch_data_async(url):
    await asyncio.sleep(0.1)
    return f"Data from {url}"

def process_data_sync(data):
    return f"Processed: {data}"

sync_fetch = AsyncToSyncAdapter(fetch_data_async)
result = sync_fetch.execute("http://example.com")
print(result)

async_processor = SyncToAsyncAdapter(process_data_sync)
result = asyncio.run(async_processor.execute("test"))
print(result)
```

### Real-World Use Cases
- Integrating legacy systems with modern APIs
- Making third-party libraries conform to application interfaces
- Database driver abstraction layers
- UI component libraries (adapting data models to UI widgets)
- File format conversion (JSON to XML, CSV to Parquet)
- Protocol translation (SOAP to REST)
- Hardware abstraction layers

### Common Mistakes
- Creating adapter methods that don't actually translate interfaces
- Adding unnecessary adapters when a simple function would suffice
- Not handling edge cases in translation (null values, type mismatches)
- Adapter becoming a "god object" that does too many translations
- Performance overhead from excessive translation layers

### Best Practices
- Keep adapters focused on interface translation only
- Prefer object adapters (composition) over class adapters (inheritance)
- Document the translation logic clearly
- Consider using Python's `__getattr__` for proxy-like adapters
- Test adapter in isolation with mock adaptees

### Performance Considerations
- Object adapters add one level of indirection (negligible cost)
- Class adapters have zero runtime overhead (inheritance is resolved at compile time)
- Heavy translation (format conversion) may have significant cost
- Consider caching translated objects if adaptation is expensive

### Interview Questions
1. What's the difference between class adapter and object adapter?
2. When would you choose Adapter over modifying the original class?
3. How does Python's duck typing simplify the Adapter pattern?
4. How is Adapter different from Facade pattern?
5. Can you implement an adapter using `__getattr__` magic method?

### Coding Challenges
1. Adapt a legacy CSV library to work with a new data processing pipeline
2. Create an adapter that converts between dict-based and object-based APIs
3. Build an adapter that makes different logging frameworks interchangeable
4. Implement a caching adapter that wraps any data source

### Related Topics
- Bridge pattern
- Facade pattern
- Proxy pattern
- Decorator pattern
- Dependency injection
- Duck typing in Python
