# Observer - Subject/Observer, event system, publish/subscribe

## Introduction

The Observer pattern defines a one-to-many dependency between objects so that when one object (the subject) changes state, all its dependents (observers) are notified and updated automatically. It is a fundamental behavioral pattern that enables loose coupling between components while maintaining communication about state changes.

## Why It Is Important

The Observer pattern is essential for building event-driven systems, UI frameworks, notification services, and reactive architectures. It enables decoupled communication between components, supports broadcasting changes to multiple listeners without the subject knowing their details, and simplifies adding or removing observers at runtime. Without it, systems would require tight coupling, polling loops, or complex callback management.

## Syntax

The pattern consists of a Subject interface with attach/detach/notify methods and an Observer interface with an update method. Python's built-in support includes `weakref` for memory-safe observer lists, `asyncio` for async event emitters, and `property` descriptors for attribute change observation.

## Examples

```python
# Classic Observer pattern
from abc import ABC, abstractmethod
from typing import List, Any


class Observer(ABC):
    @abstractmethod
    def update(self, subject: "Subject") -> None:
        pass


class Subject:
    def __init__(self):
        self._observers: List[Observer] = []
        self._state: Any = None

    def attach(self, observer: Observer) -> None:
        if observer not in self._observers:
            self._observers.append(observer)

    def detach(self, observer: Observer) -> None:
        self._observers.remove(observer)

    def notify(self) -> None:
        for observer in self._observers:
            observer.update(self)

    @property
    def state(self) -> Any:
        return self._state

    @state.setter
    def state(self, value: Any) -> None:
        self._state = value
        self.notify()


class ConcreteObserverA(Observer):
    def update(self, subject: Subject) -> None:
        print(f"Observer A: State changed to {subject.state}")


class ConcreteObserverB(Observer):
    def update(self, subject: Subject) -> None:
        print(f"Observer B: Reacted to {subject.state}")


subject = Subject()
observer_a = ConcreteObserverA()
observer_b = ConcreteObserverB()
subject.attach(observer_a)
subject.attach(observer_b)
subject.state = "active"
# Observer A: State changed to active
# Observer B: Reacted to active
```

```python
# Push vs Pull model
class PushSubject:
    def __init__(self):
        self._observers = []

    def attach(self, observer):
        self._observers.append(observer)

    def notify(self, data):
        for observer in self._observers:
            observer.update(data)  # Push — sends data


class PullSubject:
    def __init__(self):
        self._observers = []
        self._data = None

    def attach(self, observer):
        self._observers.append(observer)

    def notify(self):
        for observer in self._observers:
            observer.update(self)  # Pull — observer calls subject

    def get_data(self):
        return self._data

    def set_data(self, value):
        self._data = value
        self.notify()


class PushObserver:
    def update(self, data):
        print(f"Push observer received: {data}")


class PullObserver:
    def update(self, subject):
        data = subject.get_data()
        print(f"Pull observer fetched: {data}")
```

```python
# Observer with weak references to avoid memory leaks
import weakref
from typing import Set


class WeakSubject:
    def __init__(self):
        self._observers: Set[weakref.ref] = set()

    def attach(self, observer):
        ref = weakref.ref(observer, self._remove_ref)
        self._observers.add(ref)

    def detach(self, observer):
        for ref in set(self._observers):
            if ref() is observer:
                self._observers.discard(ref)
                return

    def _remove_ref(self, ref):
        self._observers.discard(ref)

    def notify(self, **kwargs):
        dead = []
        for ref in self._observers:
            observer = ref()
            if observer:
                observer.update(**kwargs)
            else:
                dead.append(ref)
        for ref in dead:
            self._observers.discard(ref)


class WeakObserver:
    def update(self, **kwargs):
        print(f"Received: {kwargs}")


subject = WeakSubject()
obs = WeakObserver()
subject.attach(obs)
subject.notify(event="change", value=42)
# Received: {'event': 'change', 'value': 42}
del obs
# Observer is automatically cleaned up
```

## Beginner Examples

```python
# Simple event system
class EventEmitter:
    def __init__(self):
        self._handlers = []

    def on(self, handler):
        self._handlers.append(handler)

    def off(self, handler):
        self._handlers.remove(handler)

    def emit(self, *args, **kwargs):
        for handler in self._handlers:
            handler(*args, **kwargs)


def on_message(text):
    print(f"Message received: {text}")


def on_message_upper(text):
    print(f"UPPERCASE: {text.upper()}")


emitter = EventEmitter()
emitter.on(on_message)
emitter.on(on_message_upper)
emitter.emit("hello world")
```

```python
# Property observer using descriptors
class ObservableProperty:
    def __init__(self, observers=None):
        self._observers = observers or []
        self._value = None

    def __get__(self, obj, objtype=None):
        return self._value

    def __set__(self, obj, value):
        old_value = self._value
        self._value = value
        for callback in self._observers:
            callback(value, old_value)

    def add_observer(self, callback):
        self._observers.append(callback)


class Person:
    name = ObservableProperty()

    def __init__(self, name):
        self.name = name


def on_name_changed(new_value, old_value):
    print(f"Name changed from '{old_value}' to '{new_value}'")


p = Person("Alice")
p.__class__.name.add_observer(on_name_changed)
p.name = "Bob"
```

## Intermediate Examples

```python
# Event system with multiple event types
from typing import Dict, List, Callable, Any
from collections import defaultdict


class EventBus:
    def __init__(self):
        self._listeners: Dict[str, List[Callable]] = defaultdict(list)

    def on(self, event_type: str, callback: Callable):
        self._listeners[event_type].append(callback)

    def off(self, event_type: str, callback: Callable):
        if callback in self._listeners[event_type]:
            self._listeners[event_type].remove(callback)

    def emit(self, event_type: str, **data):
        for callback in self._listeners[event_type]:
            callback(**data)


class ShoppingCart:
    def __init__(self, event_bus: EventBus):
        self.items = []
        self.event_bus = event_bus

    def add_item(self, product, quantity=1):
        self.items.append({"product": product, "quantity": quantity})
        self.event_bus.emit("item_added", product=product, quantity=quantity)
        self.event_bus.emit("cart_updated", item_count=len(self.items))

    def remove_item(self, product):
        self.items = [i for i in self.items if i["product"] != product]
        self.event_bus.emit("item_removed", product=product)
        self.event_bus.emit("cart_updated", item_count=len(self.items))

    def checkout(self):
        total = len(self.items)
        self.items.clear()
        self.event_bus.emit("checkout_completed", total_items=total)


def on_item_added(product, quantity):
    print(f"Added {quantity}x {product}")


def on_checkout(total_items):
    print(f"Checkout: {total_items} items purchased")


bus = EventBus()
cart = ShoppingCart(bus)
bus.on("item_added", on_item_added)
bus.on("checkout_completed", on_checkout)

cart.add_item("Laptop", 1)
cart.add_item("Mouse", 2)
cart.checkout()
```

```python
# Observer with filtering and async notification
import asyncio
from typing import List, Callable, Any


class FilteredEventEmitter:
    def __init__(self):
        self._handlers: List[tuple] = []

    def on(self, event_type: str, handler: Callable, condition: Callable = None):
        self._handlers.append((event_type, handler, condition or (lambda e: True)))

    def emit(self, event_type: str, event_data: Any = None):
        for et, handler, condition in self._handlers:
            if et == event_type and condition(event_data):
                handler(event_data)


def price_alert_handler(product):
    print(f"PRICE ALERT: {product['name']} now ${product['price']}")


def low_stock_handler(product):
    print(f"LOW STOCK: {product['name']} only {product['stock']} left")


emitter = FilteredEventEmitter()
emitter.on("price_change", price_alert_handler,
           condition=lambda e: e["price"] < e.get("threshold", 100))
emitter.on("stock_change", low_stock_handler,
           condition=lambda e: e["stock"] < 10)

emitter.emit("price_change", {"name": "Laptop", "price": 899, "threshold": 1000})
emitter.emit("price_change", {"name": "Mouse", "price": 25, "threshold": 50})
emitter.emit("stock_change", {"name": "Keyboard", "stock": 3})
```

```python
# Property observers with __setattr__ override
class ObservableModel:
    def __init__(self):
        self._observers = {}
        self._initializing = True

    def add_observer(self, attr_name: str, callback: Callable):
        if attr_name not in self._observers:
            self._observers[attr_name] = []
        self._observers[attr_name].append(callback)

    def __setattr__(self, name: str, value: Any):
        if name.startswith("_") or not hasattr(self, "_observers"):
            super().__setattr__(name, value)
            return
        old = getattr(self, name, None)
        super().__setattr__(name, value)
        if name in self._observers:
            for cb in self._observers[name]:
                cb(name, value, old)


class User(ObservableModel):
    def __init__(self, name, email):
        super().__init__()
        self.name = name
        self.email = email


def on_field_change(field, new, old):
    print(f"Field '{field}' changed: '{old}' -> '{new}'")


user = User("Alice", "alice@example.com")
user.add_observer("email", on_field_change)
user.email = "alice@newdomain.com"
user.name = "Alice Smith"
```

## Advanced Examples

```python
# Async event emitter with asyncio
import asyncio
from typing import Dict, List, Callable, Any
from collections import defaultdict


class AsyncEventEmitter:
    def __init__(self):
        self._listeners: Dict[str, List[Callable]] = defaultdict(list)

    def on(self, event: str, callback: Callable):
        self._listeners[event].append(callback)

    def off(self, event: str, callback: Callable):
        if callback in self._listeners[event]:
            self._listeners[event].remove(callback)

    async def emit(self, event: str, **kwargs):
        tasks = []
        for callback in self._listeners[event]:
            if asyncio.iscoroutinefunction(callback):
                tasks.append(callback(**kwargs))
            else:
                callback(**kwargs)
        if tasks:
            await asyncio.gather(*tasks)


class DataPipeline:
    def __init__(self, emitter: AsyncEventEmitter):
        self.emitter = emitter
        self.data = []

    async def process_batch(self, items):
        for item in items:
            self.data.append(item)
            await self.emitter.emit("item_processed", item=item)
        await self.emitter.emit("batch_complete", count=len(items))


async def log_item(item):
    print(f"Processing: {item}")
    await asyncio.sleep(0.1)


async def aggregate_batch(count):
    print(f"Batch complete: {count} items")


async def main():
    emitter = AsyncEventEmitter()
    pipeline = DataPipeline(emitter)

    emitter.on("item_processed", log_item)
    emitter.on("batch_complete", aggregate_batch)

    await pipeline.process_batch(["A", "B", "C", "D"])


asyncio.run(main())
```

```python
# Observer with priority ordering and event propagation control
from dataclasses import dataclass, field
from typing import Any, List, Callable


class Event:
    def __init__(self, name: str, data: Any = None):
        self.name = name
        self.data = data
        self.propagation_stopped = False

    def stop_propagation(self):
        self.propagation_stopped = True


@dataclass
class HandlerRegistration:
    priority: int
    handler: Callable
    once: bool = False


class PriorityEventEmitter:
    def __init__(self):
        self._handlers: List[HandlerRegistration] = []

    def on(self, event_type: str, handler: Callable, priority: int = 0):
        self._handlers.append(HandlerRegistration(priority, handler))

    def once(self, event_type: str, handler: Callable, priority: int = 0):
        self._handlers.append(HandlerRegistration(priority, handler, once=True))

    def emit(self, event_type: str, data: Any = None):
        event = Event(event_type, data)
        sorted_handlers = sorted(self._handlers, key=lambda h: h.priority, reverse=True)
        to_remove = []

        for registration in sorted_handlers:
            if event.propagation_stopped:
                break
            registration.handler(event)
            if registration.once:
                to_remove.append(registration)

        for reg in to_remove:
            self._handlers.remove(reg)


def high_priority_handler(event):
    print(f"[HIGH] {event.name}: {event.data}")


def medium_priority_handler(event):
    print(f"[MEDIUM] {event.name}: {event.data}")
    if event.data == "stop":
        event.stop_propagation()


def low_priority_handler(event):
    print(f"[LOW] {event.name}: {event.data}")


emitter = PriorityEventEmitter()
emitter.on("test", low_priority_handler, priority=1)
emitter.on("test", medium_priority_handler, priority=5)
emitter.on("test", high_priority_handler, priority=10)

emitter.emit("test", "hello")
print("---")
emitter.emit("test", "stop")
```

```python
# Observable collections with change tracking
from typing import List, Callable, Any


class ObservableList:
    def __init__(self, initial: List = None):
        self._data = list(initial) if initial else []
        self._callbacks: List[Callable] = []

    def observe(self, callback: Callable):
        self._callbacks.append(callback)

    def _notify(self, action: str, *args):
        for cb in self._callbacks:
            cb(action, *args)

    def append(self, item):
        self._data.append(item)
        self._notify("append", item)

    def insert(self, index: int, item):
        self._data.insert(index, item)
        self._notify("insert", index, item)

    def remove(self, item):
        self._data.remove(item)
        self._notify("remove", item)

    def pop(self, index: int = -1):
        item = self._data.pop(index)
        self._notify("pop", index, item)
        return item

    def __getitem__(self, index):
        return self._data[index]

    def __len__(self):
        return len(self._data)

    def __iter__(self):
        return iter(self._data)

    def __repr__(self):
        return repr(self._data)


def list_changed(action, *args):
    print(f"Change: {action} {args}")


obs_list = ObservableList([1, 2, 3])
obs_list.observe(list_changed)
obs_list.append(4)
obs_list.insert(0, 0)
obs_list.remove(2)
obs_list.pop()
```

```python
# Observer with undo support
from typing import Any, Dict, List, Tuple
from collections import deque


class UndoableObservable:
    def __init__(self):
        self._state: Dict[str, Any] = {}
        self._observers: List[Callable] = []
        self._history: deque = deque(maxlen=100)
        self._future: deque = deque(maxlen=100)

    def observe(self, callback: Callable):
        self._observers.append(callback)

    def set(self, key: str, value: Any):
        old = self._state.get(key)
        self._state[key] = value
        if old != value:
            self._history.append(("set", key, old, value))
            self._future.clear()
            self._notify("set", key, value, old)

    def undo(self):
        if not self._history:
            return
        action, key, old, value = self._history.pop()
        self._state[key] = old
        self._future.append((action, key, old, value))
        self._notify("undo", key, old, value)

    def redo(self):
        if not self._future:
            return
        action, key, old, value = self._future.pop()
        self._state[key] = value
        self._history.append((action, key, old, value))
        self._notify("redo", key, value, old)

    def get(self, key: str, default=None):
        return self._state.get(key, default)

    def _notify(self, action: str, key: str, new_value: Any, old_value: Any):
        for cb in self._observers:
            cb(action, key, new_value, old_value)


def state_changed(action, key, new, old):
    print(f"[{action}] {key}: {old} -> {new}")


obj = UndoableObservable()
obj.observe(state_changed)
obj.set("name", "Alice")
obj.set("age", 30)
obj.set("name", "Bob")
obj.undo()
obj.undo()
obj.redo()
```

## Real-World Use Cases

```python
# UI framework — button click events
from enum import Enum, auto


class UIEvent(Enum):
    CLICK = auto()
    HOVER = auto()
    FOCUS = auto()
    BLUR = auto()
    KEYDOWN = auto()
    KEYUP = auto()


class UIElement:
    def __init__(self, name: str):
        self.name = name
        self._listeners: Dict[UIEvent, List[Callable]] = {e: [] for e in UIEvent}

    def bind(self, event: UIEvent, handler: Callable):
        self._listeners[event].append(handler)

    def trigger(self, event: UIEvent, **kwargs):
        for handler in self._listeners[event]:
            handler(self, **kwargs)


def on_click(element, **kwargs):
    print(f"Button '{element.name}' clicked at ({kwargs.get('x', 0)}, {kwargs.get('y', 0)})")


def on_hover(element, **kwargs):
    print(f"Button '{element.name}' hovered")


button = UIElement("Submit")
button.bind(UIEvent.CLICK, on_click)
button.bind(UIEvent.HOVER, on_hover)

button.trigger(UIEvent.CLICK, x=100, y=50)
button.trigger(UIEvent.HOVER)
```

```python
# Stock market price ticker
import random
import time
from threading import Thread, Event as ThreadEvent


class StockTicker:
    def __init__(self):
        self._observers = []
        self._running = False
        self._stop_event = ThreadEvent()
        self._prices = {"AAPL": 150.0, "GOOGL": 2800.0, "MSFT": 300.0, "AMZN": 3300.0}

    def attach(self, observer):
        self._observers.append(observer)

    def detach(self, observer):
        self._observers.remove(observer)

    def _notify(self, symbol, price, change):
        for obs in self._observers:
            obs.update(symbol, price, change)

    def start(self):
        self._running = True
        self._thread = Thread(target=self._run, daemon=True)
        self._thread.start()

    def stop(self):
        self._running = False
        self._stop_event.set()

    def _run(self):
        while self._running:
            symbol = random.choice(list(self._prices.keys()))
            current = self._prices[symbol]
            change = random.uniform(-5, 5)
            new_price = round(max(0, current + change), 2)
            self._prices[symbol] = new_price
            self._notify(symbol, new_price, round(change, 2))
            self._stop_event.wait(random.uniform(0.5, 2.0))


class PriceAlert:
    def __init__(self, symbol, threshold, direction="above"):
        self.symbol = symbol
        self.threshold = threshold
        self.direction = direction

    def update(self, symbol, price, change):
        if symbol == self.symbol:
            if self.direction == "above" and price > self.threshold:
                print(f"ALERT: {symbol} crossed ABOVE {self.threshold} (now ${price})")
            elif self.direction == "below" and price < self.threshold:
                print(f"ALERT: {symbol} crossed BELOW {self.threshold} (now ${price})")


class PortfolioTracker:
    def __init__(self):
        self.holdings = {}

    def update(self, symbol, price, change):
        if symbol in self.holdings:
            shares = self.holdings[symbol]
            value = shares * price
            print(f"PORTFOLIO: {shares} shares of {symbol} = ${value:.2f}")


ticker = StockTicker()
ticker.attach(PriceAlert("AAPL", 155.0, "above"))
ticker.attach(PriceAlert("MSFT", 290.0, "below"))
portfolio = PortfolioTracker()
portfolio.holdings = {"AAPL": 10, "GOOGL": 2}
ticker.attach(portfolio)

ticker.start()
time.sleep(5)
ticker.stop()
```

## Common Mistakes

```python
# MISTAKE 1: Observer holding reference prevents garbage collection
class LeakySubject:
    def __init__(self):
        self._observers = []  # Strong references cause memory leaks

    def attach(self, observer):
        self._observers.append(observer)  # Use weakref.ref instead


# MISTAKE 2: Modifying observer list during notification
class UnsafeSubject:
    def notify(self):
        for obs in self._observers:
            obs.update()  # If observer detaches itself → RuntimeError


# MISTAKE 3: Notifying too frequently (performance issues)
# Batch state changes before notifying

# MISTAKE 4: Order-dependent observers
# Don't rely on notification order unless explicitly designed
```

```python
# Fix: safe iteration over observer list
class SafeSubject:
    def notify(self):
        for obs in list(self._observers):
            if obs in self._observers:
                obs.update()
```

## Best Practices

```python
# 1. Use weak references to prevent memory leaks
# 2. Detach observers when they are no longer needed
# 3. Batch state changes before notification
# 4. Consider async notification for I/O-bound observers
# 5. Provide error handling so one observer doesn't break others
# 6. Use event objects with metadata instead of raw calls
```

```python
# Safe notification with error isolation
class SafeEventEmitter:
    def emit(self, event, **data):
        for handler in list(self._handlers[event]):
            try:
                handler(**data)
            except Exception as e:
                print(f"Handler error: {e}")
```

## Interview Questions

```python
# Q1: Push vs Pull model — trade-offs?
# Push: simpler observers, but may send unnecessary data
# Pull: flexible observers, but more coupling to subject
```

```python
# Q2: How does observer support the Open/Closed Principle?
# New observers can be added without modifying the subject
```

```python
# Q3: Difference between Observer and Publish-Subscribe?
# Observer: direct subject→observer notification
# Pub-Sub: mediated through message broker/channel
```

```python
# Q4: How to avoid memory leaks with observers?
# Use weakref, explicit detach, or context managers
```

## Coding Challenges

```python
# Challenge 1: Implement a reactive property system
# where changing one property auto-updates dependents
```

```python
# Challenge 2: Build an async event bus with filtering
# and wildcard pattern matching on event names
```

```python
# Challenge 3: Create a middleware pattern using observers
# where each observer can modify event data
```

```python
# Challenge 4: Implement an observable dictionary
# that tracks all mutations and supports rollback
```

```python
# Challenge 5: Build a rate-limited observer wrapper
# that batches rapid notifications into periodic updates
```

## Summary

The Observer pattern enables one-to-many dependency with loose coupling between subjects and observers. Implementation choices include push vs pull models, strong vs weak references, synchronous vs async notification, and simple callbacks vs structured event objects. Python's dynamic features, weakref module, and asyncio support enable elegant implementations. Key concerns include memory management, notification ordering, error isolation, and performance with many observers.

## Related Topics

- Pub-Sub Pattern: Decoupled event distribution through channels
- MVC Architecture: Uses observer for view-model communication
- Event-Driven Architecture: Larger architectural pattern
- Reactive Programming: Extension of observer with data flows
- Async/Await: Enables non-blocking observer notification
- Weak References: Essential for memory-safe observer lists
