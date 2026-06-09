# Command - Encapsulated requests, undo/redo, command queues
## Introduction
The Command pattern encapsulates a request as an object, allowing parameterization, queuing, logging, and undo/redo operations. It decouples the object that invokes the operation from the object that knows how to perform it. This pattern is fundamental in GUI systems, transaction management, task scheduling, and macro recording.

## Encapsulated requests
### What It Is
A Command object encapsulates all information needed to perform an action: the method to call, the object to call it on, and any parameters. This turns a request into a standalone object that can be passed around, stored, and executed later.

### Why It Is Important
Encapsulating requests enables parameterization of clients with different requests, queuing of operations, logging of changes for auditing, and implementing transactional behavior with rollback capabilities.

### How It Works Internally
The pattern has four components: Command (interface with execute), ConcreteCommand (implements execute by calling receiver methods), Receiver (knows how to perform the work), and Invoker (asks the command to execute the request).

```python
from abc import ABC, abstractmethod
from typing import List

class Command(ABC):
    @abstractmethod
    def execute(self) -> None:
        pass

    @abstractmethod
    def undo(self) -> None:
        pass

# Receiver
class Light:
    def __init__(self, location: str):
        self.location = location
        self._is_on = False

    def turn_on(self) -> None:
        self._is_on = True
        print(f"{self.location} light is ON")

    def turn_off(self) -> None:
        self._is_on = False
        print(f"{self.location} light is OFF")

    @property
    def is_on(self) -> bool:
        return self._is_on

class LightOnCommand(Command):
    def __init__(self, light: Light):
        self._light = light

    def execute(self) -> None:
        self._light.turn_on()

    def undo(self) -> None:
        self._light.turn_off()

class LightOffCommand(Command):
    def __init__(self, light: Light):
        self._light = light

    def execute(self) -> None:
        self._light.turn_off()

    def undo(self) -> None:
        self._light.turn_on()

# Invoker
class RemoteControl:
    def __init__(self):
        self._slot = None

    def set_command(self, command: Command) -> None:
        self._slot = command

    def press_button(self) -> None:
        if self._slot:
            self._slot.execute()

    def press_undo(self) -> None:
        if self._slot:
            self._slot.undo()

# Usage
light = Light("Living room")
light_on = LightOnCommand(light)
light_off = LightOffCommand(light)

remote = RemoteControl()
remote.set_command(light_on)
remote.press_button()  # Light ON
remote.press_undo()    # Light OFF

remote.set_command(light_off)
remote.press_button()  # Light OFF
remote.press_undo()    # Light ON
```

## Undo/redo
### What It Is
Undo/Redo functionality allows reversing or reapplying previously executed commands. The Command pattern naturally supports this by storing a history of executed commands and providing undo/redo operations.

### Why It Is Important
Undo/Redo is critical for user-friendly applications (text editors, image editors, IDEs). The Command pattern makes implementing this straightforward by storing command history and calling each command's undo method in reverse order.

### How It Works Internally
An invoker maintains a history stack of executed commands. When undo is requested, it pops a command from the history and calls its undo method. For redo, it pushes commands onto a redo stack and calls their execute method.

```python
from abc import ABC, abstractmethod
from typing import List

class EditorCommand(ABC):
    @abstractmethod
    def execute(self) -> None:
        pass

    @abstractmethod
    def undo(self) -> None:
        pass

# Receiver
class TextEditor:
    def __init__(self):
        self._content = ""

    def insert(self, text: str, position: int = None) -> None:
        if position is None:
            self._content += text
        else:
            self._content = self._content[:position] + text + self._content[position:]

    def delete(self, start: int, end: int) -> str:
        deleted = self._content[start:end]
        self._content = self._content[:start] + self._content[end:]
        return deleted

    @property
    def content(self) -> str:
        return self._content

class InsertCommand(EditorCommand):
    def __init__(self, editor: TextEditor, text: str, position: int = None):
        self._editor = editor
        self._text = text
        self._position = position

    def execute(self) -> None:
        self._editor.insert(self._text, self._position)

    def undo(self) -> None:
        if self._position is None:
            start = len(self._editor.content) - len(self._text)
            self._editor.delete(start, len(self._editor.content))
        else:
            self._editor.delete(self._position, self._position + len(self._text))

class DeleteCommand(EditorCommand):
    def __init__(self, editor: TextEditor, start: int, end: int):
        self._editor = editor
        self._start = start
        self._end = end
        self._deleted_text = ""

    def execute(self) -> None:
        self._deleted_text = self._editor.delete(self._start, self._end)

    def undo(self) -> None:
        self._editor.insert(self._deleted_text, self._start)

# Invoker with undo/redo
class EditorInvoker:
    def __init__(self):
        self._history: List[EditorCommand] = []
        self._redo_stack: List[EditorCommand] = []

    def execute(self, command: EditorCommand) -> None:
        command.execute()
        self._history.append(command)
        self._redo_stack.clear()

    def undo(self) -> None:
        if self._history:
            command = self._history.pop()
            command.undo()
            self._redo_stack.append(command)

    def redo(self) -> None:
        if self._redo_stack:
            command = self._redo_stack.pop()
            command.execute()
            self._history.append(command)

# Usage
editor = TextEditor()
invoker = EditorInvoker()

invoker.execute(InsertCommand(editor, "Hello"))
invoker.execute(InsertCommand(editor, " World"))
print(editor.content)  # "Hello World"

invoker.undo()
print(editor.content)  # "Hello"

invoker.redo()
print(editor.content)  # "Hello World"

invoker.execute(DeleteCommand(editor, 0, 5))
print(editor.content)  # " World"

invoker.undo()
print(editor.content)  # "Hello World"
```

### Composite Undo
```python
class CompositeCommand(EditorCommand):
    def __init__(self):
        self._commands: List[EditorCommand] = []

    def add(self, command: EditorCommand) -> None:
        self._commands.append(command)

    def execute(self) -> None:
        for command in self._commands:
            command.execute()

    def undo(self) -> None:
        for command in reversed(self._commands):
            command.undo()

# Usage: Macro recording
macro = CompositeCommand()
macro.add(InsertCommand(editor, "Hello, "))
macro.add(InsertCommand(editor, "World!"))
macro.add(DeleteCommand(editor, 7, 12))
invoker.execute(macro)  # Executes all as one action
```

## Command queues
### What It Is
Command queues store commands for sequential or deferred execution. They enable scheduling, prioritization, asynchronous processing, and distributed execution of commands.

### Why It Is Important
Command queues decouple command production from execution. They enable task scheduling, load leveling, background processing, and work distribution across threads or processes.

### How It Works Internally
Commands are added to a queue (FIFO typically, but can be priority-based). A worker (or pool of workers) pulls commands from the queue and executes them. Results can be collected asynchronously.

```python
from abc import ABC, abstractmethod
from queue import Queue, PriorityQueue
from typing import Any, Callable
import threading
import time

class Task(ABC):
    @abstractmethod
    def execute(self) -> Any:
        pass

class PrintTask(Task):
    def __init__(self, message: str):
        self._message = message

    def execute(self) -> str:
        print(f"Printing: {self._message}")
        time.sleep(0.1)
        return f"Printed: {self._message}"

class CalculationTask(Task):
    def __init__(self, a: float, b: float, operation: str):
        self._a = a
        self._b = b
        self._operation = operation

    def execute(self) -> float:
        operations = {
            "add": lambda: self._a + self._b,
            "subtract": lambda: self._a - self._b,
            "multiply": lambda: self._a * self._b,
            "divide": lambda: self._a / self._b if self._b != 0 else float('inf'),
        }
        result = operations[self._operation]()
        print(f"Calculation: {self._a} {self._operation} {self._b} = {result}")
        return result

class TaskQueue:
    def __init__(self, num_workers: int = 1):
        self._queue: Queue = Queue()
        self._workers = []
        self._running = True

        for _ in range(num_workers):
            worker = threading.Thread(target=self._worker_loop, daemon=True)
            self._workers.append(worker)
            worker.start()

    def _worker_loop(self):
        while self._running:
            try:
                task = self._queue.get(timeout=1)
                if task is None:
                    break
                task.execute()
                self._queue.task_done()
            except:
                pass

    def add_task(self, task: Task) -> None:
        self._queue.put(task)

    def wait_completion(self) -> None:
        self._queue.join()

    def shutdown(self) -> None:
        self._running = False
        for _ in self._workers:
            self._queue.put(None)
        for worker in self._workers:
            worker.join()

# Usage
queue = TaskQueue(num_workers=3)

queue.add_task(PrintTask("Hello"))
queue.add_task(CalculationTask(10, 5, "add"))
queue.add_task(PrintTask("World"))
queue.add_task(CalculationTask(100, 7, "subtract"))
queue.add_task(CalculationTask(6, 7, "multiply"))

queue.wait_completion()
queue.shutdown()
```

### Priority Command Queue
```python
from dataclasses import dataclass, field
from typing import Any

@dataclass(order=True)
class PrioritizedTask:
    priority: int
    task: Any = field(compare=False)

    def execute(self) -> None:
        self.task.execute()

class PriorityTaskQueue:
    def __init__(self):
        self._queue = PriorityQueue()

    def add_task(self, task: Task, priority: int = 0) -> None:
        self._queue.put(PrioritizedTask(priority, task))

    def get_task(self) -> PrioritizedTask:
        return self._queue.get()

    def task_done(self) -> None:
        self._queue.task_done()

    def join(self) -> None:
        self._queue.join()

# Usage
pq = PriorityTaskQueue()
pq.add_task(PrintTask("Low priority"), priority=10)
pq.add_task(PrintTask("High priority"), priority=0)
pq.add_task(PrintTask("Medium priority"), priority=5)

while not pq._queue.empty():
    pt = pq.get_task()
    pt.execute()
    pq.task_done()
```

### Async Command Queue
```python
import asyncio
from asyncio import Queue

class AsyncCommand:
    async def execute(self):
        raise NotImplementedError

class AsyncPrintCommand(AsyncCommand):
    def __init__(self, message):
        self.message = message

    async def execute(self):
        await asyncio.sleep(0.1)
        print(f"Async: {self.message}")

class AsyncWorkerPool:
    def __init__(self, num_workers=3):
        self.queue = Queue()
        self.workers = [
            asyncio.create_task(self._worker(f"Worker-{i}"))
            for i in range(num_workers)
        ]

    async def _worker(self, name):
        while True:
            command = await self.queue.get()
            if command is None:
                self.queue.task_done()
                break
            print(f"{name} processing...")
            await command.execute()
            self.queue.task_done()

    async def add_command(self, command):
        await self.queue.put(command)

    async def shutdown(self):
        for _ in self.workers:
            await self.queue.put(None)
        await asyncio.gather(*self.workers)

async def main():
    pool = AsyncWorkerPool(2)
    await pool.add_command(AsyncPrintCommand("Hello"))
    await pool.add_command(AsyncPrintCommand("World"))
    await pool.add_command(AsyncPrintCommand("From"))
    await pool.add_command(AsyncPrintCommand("Async"))
    await pool.queue.join()
    await pool.shutdown()

asyncio.run(main())
```

### Real-World Use Cases
- GUI menu items and buttons
- Transaction processing systems
- Macro recording in applications
- Task schedulers and job queues
- Game action systems (replay, undo)
- Database transaction logs
- Remote procedure calls

### Common Mistakes
- Making commands too large (should be single operation)
- Not implementing undo for all commands that need it
- Memory leaks from unbounded command history
- Thread safety issues in shared command queues
- Commands that depend on external state at execution time

### Best Practices
- Keep commands small and focused on single action
- Implement both execute and undo for reversible operations
- Use Memento pattern with Command for state snapshots
- Limit undo history to prevent memory issues
- Make commands serializable for distributed systems

### Performance Considerations
- Each command is a small object (minimal memory per command)
- Undo history grows with number of operations
- Consider compressing or limiting history for long-running apps
- Command queue adds minimal latency (queue operations are O(1))

### Interview Questions
1. How does Command pattern enable undo/redo?
2. What's the relationship between Command and Memento patterns?
3. How would you implement a transaction rollback system using Command?
4. What are the differences between Command and Strategy patterns?
5. How do you make Command objects thread-safe?

### Coding Challenges
1. Implement a full undo/redo system for a text editor
2. Build a task scheduler with priority-based command queue
3. Create a macro recorder that records and replays commands
4. Implement a distributed command queue with worker pool

### Related Topics
- Memento pattern
- Strategy pattern
- Observer pattern
- Chain of Responsibility
- Queue data structures
- Transaction management
- Macro systems
