# Observer - Subject/Observer, event system, publish/subscribe
## Introduction
The Observer pattern defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically. This pattern is fundamental to event-driven programming, GUI systems, and distributed communication. Python implements Observer naturally through callbacks, events, and signals due to its first-class function support.

## Subject/Observer pattern
### What It Is
The classic Observer pattern consists of a Subject (Observable) that maintains a list of Observers. When the Subject's state changes, it broadcasts notifications to all registered Observers. Observers can subscribe or unsubscribe at runtime.

### Why It Is Important
Observer promotes loose coupling between components. The Subject doesn't need to know concrete Observer classes; it only knows they implement a common interface. This enables flexible, extensible systems where new observers can be added without modifying existing code.

### How It Works Internally
The Subject maintains a list of observer references. On state change, the Subject iterates through observers and calls their update method. Observers can query the Subject for the new state. This model is sometimes called "push" (subject pushes notification, observers pull data they need).

```python
from abc import ABC, abstractmethod
from typing import List

class Observer(ABC):
    @abstractmethod
    def update(self, subject: 'Subject') -> None:
        pass

class Subject:
    def __init__(self):
        self._observers: List[Observer] = []
        self._state = None

    def attach(self, observer: Observer) -> None:
        if observer not in self._observers:
            self._observers.append(observer)

    def detach(self, observer: Observer) -> None:
        self._observers.remove(observer)

    def notify(self) -> None:
        for observer in self._observers:
            observer.update(self)

    @property
    def state(self):
        return self._state

    @state.setter
    def state(self, value):
        self._state = value
        self.notify()

class ConcreteObserverA(Observer):
    def update(self, subject: Subject) -> None:
        if subject.state < 10:
            print(f"Observer A: reacted to state {subject.state}")

class ConcreteObserverB(Observer):
    def update(self, subject: Subject) -> None:
        if subject.state >= 10:
            print(f"Observer B: reacted to state {subject.state}")

# Usage
subject = Subject()

observer_a = ConcreteObserverA()
observer_b = ConcreteObserverB()

subject.attach(observer_a)
subject.attach(observer_b)

subject.state = 5   # Observer A reacts
subject.state = 15  # Observer B reacts
subject.detach(observer_a)
subject.state = 8   # Only Observer B still listening
```

## Event system
### What It Is
An event system extends the Observer pattern with event types. Instead of a generic notification, the Subject emits named events with associated data. Observers subscribe to specific events they're interested in.

### Why It Is Important
Event systems are more flexible than basic Observer because they allow fine-grained subscription. Components can listen only to relevant events, reducing unnecessary notifications and simplifying observer logic.

### How It Works Internally
The Subject maintains a dictionary mapping event names to lists of callbacks. When an event occurs, the Subject looks up callbacks for that event type and calls them with event data. This is similar to how DOM events work in JavaScript.

```python
from typing import Dict, List, Callable, Any

class EventEmitter:
    def __init__(self):
        self._listeners: Dict[str, List[Callable]] = {}

    def on(self, event: str, callback: Callable) -> None:
        if event not in self._listeners:
            self._listeners[event] = []
        self._listeners[event].append(callback)

    def off(self, event: str, callback: Callable) -> None:
        if event in self._listeners:
            self._listeners[event].remove(callback)

    def emit(self, event: str, *args, **kwargs) -> None:
        for callback in self._listeners.get(event, []):
            callback(*args, **kwargs)

    def once(self, event: str, callback: Callable) -> None:
        def wrapper(*args, **kwargs):
            self.off(event, wrapper)
            callback(*args, **kwargs)
        self.on(event, wrapper)

class UserService(EventEmitter):
    def create_user(self, username: str, email: str):
        user = {"username": username, "email": email}
        print(f"Creating user: {username}")
        self.emit("user.created", user)
        self.emit("user.saved", user)
        return user

    def delete_user(self, username: str):
        print(f"Deleting user: {username}")
        self.emit("user.deleted", username)

# Usage
user_service = UserService()

def on_user_created(user):
    print(f"  [Email] Send welcome email to {user['email']}")

def on_user_deleted(username):
    print(f"  [Audit] Log deletion of {username}")

user_service.on("user.created", on_user_created)
user_service.on("user.deleted", on_user_deleted)

user_service.create_user("alice", "alice@example.com")
user_service.delete_user("alice")
```

### Event System with Priorities
```python
import bisect
from typing import Dict, List, Tuple, Callable

class PrioritizedEventEmitter:
    def __init__(self):
        self._listeners: Dict[str, List[Tuple[int, Callable]]] = {}

    def on(self, event: str, callback: Callable, priority: int = 0) -> None:
        if event not in self._listeners:
            self._listeners[event] = []
        bisect.insort(self._listeners[event], (priority, callback))

    def emit(self, event: str, *args, **kwargs) -> None:
        for priority, callback in self._listeners.get(event, []):
            callback(*args, **kwargs)

# Usage
emitter = PrioritizedEventEmitter()

emitter.on("data", lambda d: print(f"Low: {d}"), priority=10)
emitter.on("data", lambda d: print(f"High: {d}"), priority=0)
emitter.on("data", lambda d: print(f"Medium: {d}"), priority=5)

emitter.emit("data", "Hello")
# Output: High: Hello
#         Medium: Hello
#         Low: Hello
```

### Event Bus (Global Event System)
```python
class EventBus:
    _instance = None
    _listeners: Dict[str, List[Callable]] = {}

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    @classmethod
    def on(cls, event: str, callback: Callable) -> None:
        if event not in cls._listeners:
            cls._listeners[event] = []
        cls._listeners[event].append(callback)

    @classmethod
    def emit(cls, event: str, *args, **kwargs) -> None:
        for callback in cls._listeners.get(event, []):
            callback(*args, **kwargs)

# Module A
def module_a_init():
    EventBus.on("config.changed", lambda c: print(f"Module A: config changed to {c}"))

# Module B
def module_b_init():
    EventBus.on("config.changed", lambda c: print(f"Module B: reconnecting with {c['host']}"))

# Main
module_a_init()
module_b_init()
EventBus.emit("config.changed", {"host": "newhost", "port": 8080})
```

## Publish/subscribe
### What It Is
Publish/Subscribe (Pub/Sub) is a messaging pattern where publishers send messages to channels/topics without knowing who the subscribers are. Subscribers listen to channels without knowing who the publishers are. A message broker (or event bus) mediates communication.

### Why It Is Important
Pub/Sub provides maximum decoupling between components. Publishers and subscribers have no direct knowledge of each other. This enables scalable, distributed, and maintainable architectures, especially in microservices and event-driven systems.

### How It Works Internally
A message broker sits between publishers and subscribers. Publishers emit messages to topics. Subscribers register interest in topics. The broker routes messages from publishers to appropriate subscribers. This can be in-process (simple dict-based) or distributed (Redis Pub/Sub, RabbitMQ, Kafka).

```python
from collections import defaultdict
from typing import Callable, Dict, List, Any

class MessageBroker:
    def __init__(self):
        self._subscribers: Dict[str, List[Callable]] = defaultdict(list)

    def subscribe(self, topic: str, callback: Callable) -> None:
        self._subscribers[topic].append(callback)

    def unsubscribe(self, topic: str, callback: Callable) -> None:
        if callback in self._subscribers[topic]:
            self._subscribers[topic].remove(callback)

    def publish(self, topic: str, message: Any) -> None:
        for callback in self._subscribers[topic]:
            callback(message)

    def get_topics(self) -> List[str]:
        return list(self._subscribers.keys())

class StockExchange:
    def __init__(self, broker: MessageBroker):
        self.broker = broker
        self.prices = {}

    def update_price(self, symbol: str, price: float):
        self.prices[symbol] = price
        self.broker.publish("stock.price_change", {
            "symbol": symbol,
            "price": price,
            "timestamp": "2024-01-15 10:30:00"
        })

class TradingBot:
    def __init__(self, name: str, broker: MessageBroker):
        self.name = name
        self.broker = broker
        self.broker.subscribe("stock.price_change", self.on_price_change)

    def on_price_change(self, message):
        if message["price"] < 100:
            print(f"{self.name} BUY: {message['symbol']} at ${message['price']}")

class MarketAnalyzer:
    def __init__(self, broker: MessageBroker):
        self.prices = []
        self.broker.subscribe("stock.price_change", self.on_price_change)

    def on_price_change(self, message):
        self.prices.append(message["price"])
        avg = sum(self.prices) / len(self.prices)
        print(f"Average price: ${avg:.2f}")

# Usage
broker = MessageBroker()
exchange = StockExchange(broker)
bot = TradingBot("Bot-1", broker)
analyzer = MarketAnalyzer(broker)

exchange.update_price("AAPL", 150.0)
exchange.update_price("GOOG", 95.0)  # Bot buys
exchange.update_price("TSLA", 85.0)  # Bot buys
```

### Topic Hierarchy (Wildcard Subscriptions)
```python
import fnmatch

class HierarchicalBroker:
    def __init__(self):
        self._subscribers: Dict[str, List[Callable]] = defaultdict(list)

    def subscribe(self, topic: str, callback: Callable) -> None:
        self._subscribers[topic].append(callback)

    def publish(self, topic: str, message: Any) -> None:
        for pattern, callbacks in self._subscribers.items():
            if fnmatch.fnmatch(topic, pattern):
                for callback in callbacks:
                    callback(message)

# Usage
broker = HierarchicalBroker()

def log_all_events(message):
    print(f"LOG: {message}")

def handle_email_events(message):
    print(f"EMAIL: {message}")

broker.subscribe("*.created", log_all_events)
broker.subscribe("user.*", handle_email_events)

broker.publish("user.created", {"name": "Alice"})  # Both match
broker.publish("order.created", {"id": 123})       # Only log matches
broker.publish("user.deleted", {"name": "Bob"})     # Only email matches
```

### Async Pub/Sub
```python
import asyncio
from collections import defaultdict
from typing import Callable, Dict, List, Any

class AsyncMessageBroker:
    def __init__(self):
        self._subscribers: Dict[str, List[Callable]] = defaultdict(list)

    def subscribe(self, topic: str, callback: Callable) -> None:
        self._subscribers[topic].append(callback)

    async def publish(self, topic: str, message: Any) -> None:
        tasks = []
        for callback in self._subscribers[topic]:
            if asyncio.iscoroutinefunction(callback):
                tasks.append(callback(message))
            else:
                callback(message)
        if tasks:
            await asyncio.gather(*tasks)

    async def publish_sequential(self, topic: str, message: Any) -> None:
        for callback in self._subscribers[topic]:
            if asyncio.iscoroutinefunction(callback):
                await callback(message)
            else:
                callback(message)

# Usage
broker = AsyncMessageBroker()

async def slow_worker(message):
    await asyncio.sleep(1)
    print(f"Worker processed: {message}")

def quick_worker(message):
    print(f"Quick: {message}")

broker.subscribe("task", slow_worker)
broker.subscribe("task", quick_worker)

async def main():
    await broker.publish("task", {"id": 1, "data": "test"})

asyncio.run(main())
```

### Real-World Use Cases
- GUI event handling (button clicks, mouse moves)
- Game engines (collision events, score updates)
- Microservice communication (via message queues)
- Real-time data feeds (stock tickers, chat apps)
- Framework signals (Django signals, Flask signals)
- Plugin systems
- State management (Redux-like patterns)

### Common Mistakes
- Memory leaks from not unsubscribing observers
- Notification storms from circular dependencies
- Thread safety issues with concurrent modification of observer lists
- Too much decoupling making code hard to debug
- Synchronous notifications blocking the publisher

### Best Practices
- Always provide unsubscribe/detach methods
- Use weak references to avoid memory leaks
- Keep observer callbacks fast (offload heavy work)
- Consider using a dedicated event bus for complex systems
- Log events for debugging in development

### Performance Considerations
- Observer notification is O(n) where n is number of observers
- Event systems with many events can have memory overhead
- Async pub/sub can improve responsiveness
- Consider batching notifications for frequently changing state

### Interview Questions
1. How does Observer differ from Pub/Sub?
2. How do you prevent memory leaks with observers?
3. Explain the event loop pattern.
4. What are the advantages of loose coupling in Observer?
5. How would you implement a thread-safe observer pattern?

### Coding Challenges
1. Implement a chat room using Pub/Sub
2. Build a simple event-driven game engine
3. Create a notification system with multiple channels
4. Implement undo/redo using event history

### Related Topics
- Command pattern
- Mediator pattern
- Reactive programming (RxPy)
- Signal/slot systems
- Message queues (RabbitMQ, Kafka)
- asyncio event loop
