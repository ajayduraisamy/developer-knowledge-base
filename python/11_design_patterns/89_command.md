# Command - Encapsulated requests, undo/redo, command queues

## Introduction

The Command pattern encapsulates a request as an object, thereby allowing parameterization of clients with different requests, queuing of requests, logging of requests, and support for undoable operations. It decouples the object that invokes the operation from the one that knows how to perform it.

## Why It Is Important

The Command pattern enables separating concerns between command initiation and execution. It is essential for implementing undo/redo functionality, job queues, macro recording, transaction-like operations, and menu systems. Commands can be composed, queued, logged, serialized, and executed asynchronously, providing tremendous flexibility in request handling.

## Syntax

A Command interface declares an execute method. Concrete commands hold a reference to the receiver and implement execute by calling receiver methods. An invoker stores and executes commands. Optionally, commands can implement undo for reversible operations.

## Examples

```python
# Basic Command pattern
from abc import ABC, abstractmethod
from typing import List


class Command(ABC):
    @abstractmethod
    def execute(self) -> None:
        pass

    @abstractmethod
    def undo(self) -> None:
        pass


class Light:
    def __init__(self, name: str = "Living Room"):
        self._name = name
        self._on = False

    def turn_on(self) -> None:
        self._on = True
        print(f"{self._name} light is ON")

    def turn_off(self) -> None:
        self._on = False
        print(f"{self._name} light is OFF")


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


class RemoteControl:
    def __init__(self):
        self._command: Command = None
        self._history: List[Command] = []

    def set_command(self, command: Command):
        self._command = command

    def press_button(self):
        self._command.execute()
        self._history.append(self._command)

    def press_undo(self):
        if self._history:
            command = self._history.pop()
            command.undo()


light = Light()
remote = RemoteControl()

remote.set_command(LightOnCommand(light))
remote.press_button()

remote.set_command(LightOffCommand(light))
remote.press_button()

remote.press_undo()
remote.press_undo()
```

```python
# Command with execute/undo for text editing
class TextEditor:
    def __init__(self):
        self._text = ""

    def insert(self, text: str, position: int = None):
        if position is None:
            position = len(self._text)
        self._text = self._text[:position] + text + self._text[position:]

    def delete(self, start: int, end: int = None):
        if end is None:
            end = start + 1
        deleted = self._text[start:end]
        self._text = self._text[:start] + self._text[end:]
        return deleted

    def __str__(self):
        return self._text


class InsertCommand(Command):
    def __init__(self, editor: TextEditor, text: str, position: int = None):
        self._editor = editor
        self._text = text
        self._position = position

    def execute(self):
        self._editor.insert(self._text, self._position)

    def undo(self):
        pos = self._position if self._position is not None else len(self._editor._text) - len(self._text)
        self._editor.delete(pos, pos + len(self._text))


class DeleteCommand(Command):
    def __init__(self, editor: TextEditor, start: int, end: int = None):
        self._editor = editor
        self._start = start
        self._end = end
        self._deleted_text = ""

    def execute(self):
        self._deleted_text = self._editor.delete(self._start, self._end)

    def undo(self):
        self._editor.insert(self._deleted_text, self._start)


class EditorInvoker:
    def __init__(self):
        self._history: List[Command] = []
        self._future: List[Command] = []

    def execute(self, command: Command):
        command.execute()
        self._history.append(command)
        self._future.clear()

    def undo(self):
        if self._history:
            command = self._history.pop()
            command.undo()
            self._future.append(command)

    def redo(self):
        if self._future:
            command = self._future.pop()
            command.execute()
            self._history.append(command)


editor = TextEditor()
invoker = EditorInvoker()

invoker.execute(InsertCommand(editor, "Hello"))
invoker.execute(InsertCommand(editor, " World", 5))
print(editor)

invoker.undo()
print(editor)

invoker.redo()
print(editor)
```

## Beginner Examples

```python
# Simple command queue
from typing import List
from collections import deque


class Task:
    def execute(self) -> str:
        raise NotImplementedError


class EmailTask(Task):
    def __init__(self, recipient: str, message: str):
        self._recipient = recipient
        self._message = message

    def execute(self) -> str:
        return f"Sending email to {self._recipient}: {self._message}"


class ReportTask(Task):
    def __init__(self, report_name: str):
        self._report_name = report_name

    def execute(self) -> str:
        return f"Generating report: {self._report_name}"


class BackupTask(Task):
    def __init__(self, path: str):
        self._path = path

    def execute(self) -> str:
        return f"Backing up: {self._path}"


class TaskQueue:
    def __init__(self):
        self._queue: deque = deque()

    def add_task(self, task: Task):
        self._queue.append(task)

    def process_all(self) -> List[str]:
        results = []
        while self._queue:
            task = self._queue.popleft()
            results.append(task.execute())
        return results


queue = TaskQueue()
queue.add_task(EmailTask("alice@example.com", "Hello!"))
queue.add_task(ReportTask("monthly_sales"))
queue.add_task(BackupTask("/data/db"))

for result in queue.process_all():
    print(result)
```

```python
# Simple calculator with undo
class Calculator:
    def __init__(self):
        self._value = 0

    def add(self, x: int) -> int:
        self._value += x
        return self._value

    def subtract(self, x: int) -> int:
        self._value -= x
        return self._value

    def multiply(self, x: int) -> int:
        self._value *= x
        return self._value

    def divide(self, x: int) -> int:
        self._value //= x
        return self._value


class CalcCommand(ABC):
    @abstractmethod
    def execute(self, calc: Calculator) -> int:
        pass

    @abstractmethod
    def undo(self, calc: Calculator) -> int:
        pass


class AddCommand(CalcCommand):
    def __init__(self, value: int):
        self._value = value

    def execute(self, calc: Calculator) -> int:
        return calc.add(self._value)

    def undo(self, calc: Calculator) -> int:
        return calc.subtract(self._value)


class MultiplyCommand(CalcCommand):
    def __init__(self, value: int):
        self._value = value

    def execute(self, calc: Calculator) -> int:
        return calc.multiply(self._value)

    def undo(self, calc: Calculator) -> int:
        return calc.divide(self._value)


calc = Calculator()
history = []

cmd = AddCommand(5)
result = cmd.execute(calc)
print(f"After add 5: {result}")
history.append(cmd)

cmd = MultiplyCommand(3)
result = cmd.execute(calc)
print(f"After multiply 3: {result}")
history.append(cmd)

cmd = history.pop()
result = cmd.undo(calc)
print(f"After undo: {result}")
```

## Intermediate Examples

```python
# Macro command — composing multiple commands
from typing import List


class MacroCommand(Command):
    def __init__(self, commands: List[Command] = None):
        self._commands = commands or []

    def add(self, command: Command):
        self._commands.append(command)

    def execute(self):
        for command in self._commands:
            command.execute()

    def undo(self):
        for command in reversed(self._commands):
            command.undo()


class Stereo:
    def __init__(self):
        self._on = False
        self._volume = 10
        self._source = "CD"

    def on(self):
        self._on = True
        print("Stereo ON")

    def off(self):
        self._on = False
        print("Stereo OFF")

    def set_volume(self, level: int):
        self._volume = level
        print(f"Stereo volume set to {level}")

    def set_source(self, source: str):
        self._source = source
        print(f"Stereo source set to {source}")


class StereoOnCommand(Command):
    def __init__(self, stereo: Stereo):
        self._stereo = stereo

    def execute(self):
        self._stereo.on()

    def undo(self):
        self._stereo.off()


class StereoOffCommand(Command):
    def __init__(self, stereo: Stereo):
        self._stereo = stereo

    def execute(self):
        self._stereo.off()

    def undo(self):
        self._stereo.on()


class SetVolumeCommand(Command):
    def __init__(self, stereo: Stereo, level: int):
        self._stereo = stereo
        self._level = level
        self._previous_level = None

    def execute(self):
        self._previous_level = self._stereo._volume
        self._stereo.set_volume(self._level)

    def undo(self):
        self._stereo.set_volume(self._previous_level)


class SetSourceCommand(Command):
    def __init__(self, stereo: Stereo, source: str):
        self._stereo = stereo
        self._source = source
        self._previous_source = None

    def execute(self):
        self._previous_source = self._stereo._source
        self._stereo.set_source(self._source)

    def undo(self):
        self._stereo.set_source(self._previous_source)


class PartyMacro(Command):
    def __init__(self):
        self._commands: List[Command] = []

    def add(self, command: Command):
        self._commands.append(command)

    def execute(self):
        print("--- Party Mode: ON ---")
        for cmd in self._commands:
            cmd.execute()

    def undo(self):
        print("--- Party Mode: OFF ---")
        for cmd in reversed(self._commands):
            cmd.undo()


light = Light("Party")
stereo = Stereo()

party = PartyMacro()
party.add(LightOnCommand(light))
party.add(StereoOnCommand(stereo))
party.add(SetVolumeCommand(stereo, 20))
party.add(SetSourceCommand(stereo, "AUX"))

party.execute()
party.undo()
```

```python
# Transaction-like command with rollback
from typing import List, Dict, Any
import json


class DatabaseCommand(Command):
    def __init__(self, db: dict, operation: str, *args):
        self._db = db
        self._operation = operation
        self._args = args
        self._backup = None

    def execute(self):
        if self._operation == "set":
            key, value = self._args
            self._backup = (key, self._db.get(key))
            self._db[key] = value
            print(f"SET {key} = {value}")
        elif self._operation == "delete":
            key = self._args[0]
            self._backup = (key, self._db.get(key))
            self._db.pop(key, None)
            print(f"DELETE {key}")

    def undo(self):
        if self._backup:
            key, value = self._backup
            if value is None:
                self._db.pop(key, None)
                print(f"ROLLBACK: restored delete of {key}")
            else:
                self._db[key] = value
                print(f"ROLLBACK: {key} = {value}")


class Transaction:
    def __init__(self, db: dict):
        self._db = db
        self._commands: List[Command] = []

    def add(self, command: Command):
        self._commands.append(command)

    def commit(self):
        for cmd in self._commands:
            cmd.execute()

    def rollback(self):
        for cmd in reversed(self._commands):
            cmd.undo()
        self._commands.clear()


db = {"name": "Alice", "age": 30}
tx = Transaction(db)
tx.add(DatabaseCommand(db, "set", "name", "Bob"))
tx.add(DatabaseCommand(db, "set", "age", 31))
tx.add(DatabaseCommand(db, "delete", "temp"))

print(f"Before: {db}")
tx.commit()
print(f"After commit: {db}")
tx.rollback()
print(f"After rollback: {db}")
```

## Advanced Examples

```python
# Command pattern with asyncio and delayed execution
import asyncio
from typing import List, Callable
from datetime import datetime, timedelta


class AsyncCommand(ABC):
    @abstractmethod
    async def execute(self) -> Any:
        pass

    async def undo(self) -> None:
        pass


class ScheduledCommand:
    def __init__(self, command: AsyncCommand, delay_seconds: float):
        self._command = command
        self._delay = delay_seconds
        self._scheduled_time = datetime.now() + timedelta(seconds=delay_seconds)

    async def run(self):
        await asyncio.sleep(self._delay)
        return await self._command.execute()


class SendEmailCommand(AsyncCommand):
    def __init__(self, to: str, subject: str, body: str):
        self._to = to
        self._subject = subject
        self._body = body
        self._sent = False

    async def execute(self) -> str:
        await asyncio.sleep(0.5)
        self._sent = True
        return f"Email sent to {self._to}: {self._subject}"

    async def undo(self) -> None:
        if self._sent:
            print(f"Attempting to recall email to {self._to}...")


class ProcessPaymentCommand(AsyncCommand):
    def __init__(self, user_id: str, amount: float):
        self._user_id = user_id
        self._amount = amount
        self._transaction_id = None

    async def execute(self) -> str:
        await asyncio.sleep(0.3)
        self._transaction_id = f"txn-{id(self)}"
        return f"Payment of ${self._amount} from {self._user_id}: {self._transaction_id}"

    async def undo(self) -> None:
        if self._transaction_id:
            print(f"Refunding transaction {self._transaction_id}...")


class CommandScheduler:
    def __init__(self):
        self._queue: asyncio.Queue = asyncio.Queue()
        self._results = []

    async def schedule(self, command: AsyncCommand, delay: float = 0):
        if delay > 0:
            scheduled = ScheduledCommand(command, delay)
            result = await scheduled.run()
        else:
            result = await command.execute()
        self._results.append(result)
        return result

    async def schedule_many(self, commands: List[tuple]):
        tasks = []
        for cmd, delay in commands:
            tasks.append(self.schedule(cmd, delay))
        return await asyncio.gather(*tasks)


async def main():
    scheduler = CommandScheduler()
    commands = [
        (SendEmailCommand("alice@example.com", "Hello", "Body"), 0.1),
        (ProcessPaymentCommand("user123", 49.99), 0.2),
        (SendEmailCommand("bob@example.com", "Receipt", "Thanks!"), 0.3),
    ]
    results = await scheduler.schedule_many(commands)
    for r in results:
        print(r)


asyncio.run(main())
```

```python
# Command pattern with undo stack and composite transactions
from typing import List, Optional
from dataclasses import dataclass
from enum import Enum


class Operation(Enum):
    INSERT = "insert"
    UPDATE = "update"
    DELETE = "delete"


@dataclass
class Record:
    id: int
    name: str
    email: str


class Database:
    def __init__(self):
        self._records: Dict[int, Record] = {}

    def insert(self, record: Record) -> None:
        self._records[record.id] = record

    def update(self, record_id: int, **kwargs) -> Optional[Record]:
        record = self._records.get(record_id)
        if record:
            old = Record(**record.__dict__)
            for k, v in kwargs.items():
                setattr(record, k, v)
            return old
        return None

    def delete(self, record_id: int) -> Optional[Record]:
        return self._records.pop(record_id, None)

    def get(self, record_id: int) -> Optional[Record]:
        return self._records.get(record_id)


class DbCommand(Command):
    def __init__(self, db: Database):
        self._db = db
        self._backup: Any = None

    def execute(self):
        raise NotImplementedError

    def undo(self):
        raise NotImplementedError


class InsertCommand(DbCommand):
    def __init__(self, db: Database, record: Record):
        super().__init__(db)
        self._record = record

    def execute(self):
        self._db.insert(self._record)
        print(f"INSERT {self._record}")

    def undo(self):
        self._db.delete(self._record.id)
        print(f"UNDO INSERT {self._record.id}")


class UpdateCommand(DbCommand):
    def __init__(self, db: Database, record_id: int, **changes):
        super().__init__(db)
        self._record_id = record_id
        self._changes = changes

    def execute(self):
        self._backup = self._db.update(self._record_id, **self._changes)
        print(f"UPDATE {self._record_id}: {self._changes}")

    def undo(self):
        if self._backup:
            changes = {k: getattr(self._backup, k) for k in self._changes}
            self._db.update(self._record_id, **changes)
            print(f"UNDO UPDATE {self._record_id}")


class DeleteCommand(DbCommand):
    def __init__(self, db: Database, record_id: int):
        super().__init__(db)
        self._record_id = record_id

    def execute(self):
        self._backup = self._db.delete(self._record_id)
        print(f"DELETE {self._record_id}")

    def undo(self):
        if self._backup:
            self._db.insert(self._backup)
            print(f"UNDO DELETE {self._record_id}")


class TransactionManager:
    def __init__(self):
        self._history: List[Command] = []
        self._future: List[Command] = []
        self._transaction_stack: List[List[Command]] = []

    def begin_transaction(self):
        self._transaction_stack.append([])
        print("BEGIN TRANSACTION")

    def execute(self, command: Command):
        if self._transaction_stack:
            self._transaction_stack[-1].append(command)
        command.execute()

    def commit(self):
        if self._transaction_stack:
            commands = self._transaction_stack.pop()
            self._history.extend(commands)
            self._future.clear()
            print("COMMIT")

    def rollback(self):
        if self._transaction_stack:
            commands = self._transaction_stack.pop()
            for cmd in reversed(commands):
                cmd.undo()
            print("ROLLBACK")

    def undo(self):
        if self._history:
            cmd = self._history.pop()
            cmd.undo()
            self._future.append(cmd)

    def redo(self):
        if self._future:
            cmd = self._future.pop()
            cmd.execute()
            self._history.append(cmd)


db = Database()
tm = TransactionManager()

tm.begin_transaction()
tm.execute(InsertCommand(db, Record(1, "Alice", "alice@example.com")))
tm.execute(InsertCommand(db, Record(2, "Bob", "bob@example.com")))
tm.commit()

tm.begin_transaction()
tm.execute(UpdateCommand(db, 1, name="Alice Smith"))
tm.execute(DeleteCommand(db, 2))
tm.rollback()

print(f"Record 1: {db.get(1)}")
print(f"Record 2: {db.get(2)}")
```

```python
# Command pattern with logging and replay
import json
import time
from typing import List, Any


class LoggedCommand(Command):
    def __init__(self, receiver, method: str, *args, **kwargs):
        self._receiver = receiver
        self._method = method
        self._args = args
        self._kwargs = kwargs

    def execute(self):
        method = getattr(self._receiver, self._method)
        return method(*self._args, **self._kwargs)

    def undo(self):
        pass

    def to_log(self) -> dict:
        return {
            "method": self._method,
            "args": self._args,
            "kwargs": self._kwargs,
            "timestamp": time.time(),
        }

    @classmethod
    def from_log(cls, receiver, log: dict) -> "LoggedCommand":
        return cls(receiver, log["method"], *log["args"], **log["kwargs"])


class CommandLogger:
    def __init__(self):
        self._log: List[dict] = []

    def record(self, command: LoggedCommand):
        entry = command.to_log()
        self._log.append(entry)
        return entry

    def save(self, filepath: str):
        with open(filepath, "w") as f:
            json.dump(self._log, f, indent=2)

    def load(self, filepath: str):
        with open(filepath, "r") as f:
            self._log = json.load(f)

    def replay(self, receiver, count: int = None):
        commands = self._log[:count] if count else self._log
        results = []
        for entry in commands:
            cmd = LoggedCommand.from_log(receiver, entry)
            results.append(cmd.execute())
        return results


class ShoppingCartService:
    def __init__(self):
        self.items = []

    def add_item(self, product_id: str, quantity: int = 1):
        self.items.append({"product_id": product_id, "quantity": quantity})
        return f"Added {product_id} x{quantity}"

    def remove_item(self, product_id: str):
        self.items = [i for i in self.items if i["product_id"] != product_id]
        return f"Removed {product_id}"

    def clear(self):
        count = len(self.items)
        self.items.clear()
        return f"Cleared {count} items"


cart = ShoppingCartService()
logger = CommandLogger()

cmd1 = LoggedCommand(cart, "add_item", "product-1", quantity=2)
logger.record(cmd1)
print(cmd1.execute())

cmd2 = LoggedCommand(cart, "add_item", "product-2")
logger.record(cmd2)
print(cmd2.execute())

cmd3 = LoggedCommand(cart, "remove_item", "product-1")
logger.record(cmd3)
print(cmd3.execute())

print(f"Cart: {cart.items}")

new_cart = ShoppingCartService()
results = logger.replay(new_cart)
print(f"Replayed: {results}")
print(f"Replayed cart: {new_cart.items}")
```

## Real-World Use Cases

```python
# Menu system with command pattern
class MenuItem:
    def __init__(self, name: str, command: Command):
        self._name = name
        self._command = command

    def click(self):
        print(f"Menu: {self._name}")
        self._command.execute()


class Application:
    def __init__(self):
        self._clipboard = ""
        self._document = ""

    def copy(self, text: str):
        self._clipboard = text
        print(f"Copied: {text}")

    def paste(self) -> str:
        print(f"Pasted: {self._clipboard}")
        return self._clipboard

    def cut(self, text: str):
        self._clipboard = text
        print(f"Cut: {text}")

    def save(self, filename: str):
        print(f"Saved to {filename}")

    def print_doc(self):
        print("Printing document...")

    def export_pdf(self, filename: str):
        print(f"Exported to {filename}")


class CopyCommand(Command):
    def __init__(self, app: Application, text: str):
        self._app = app
        self._text = text

    def execute(self):
        self._app.copy(self._text)

    def undo(self):
        pass


class PasteCommand(Command):
    def __init__(self, app: Application):
        self._app = app
        self._backup = ""

    def execute(self):
        self._backup = self._app.paste()

    def undo(self):
        pass


class MenuBar:
    def __init__(self):
        self._menus: dict = {}

    def add_menu(self, name: str, items: List[MenuItem]):
        self._menus[name] = items

    def click(self, menu_name: str, item_index: int):
        if menu_name in self._menus and item_index < len(self._menus[menu_name]):
            self._menus[menu_name][item_index].click()


app = Application()
edit_menu = [
    MenuItem("Copy", CopyCommand(app, "selected text")),
    MenuItem("Paste", PasteCommand(app)),
]

file_menu = [
    MenuItem("Save", SaveCommand(app, "document.txt")),
    MenuItem("Print", PrintCommand(app)),
    MenuItem("Export PDF", ExportPDFCommand(app, "document.pdf")),
]

menu_bar = MenuBar()
menu_bar.add_menu("File", file_menu)
menu_bar.add_menu("Edit", edit_menu)

menu_bar.click("Edit", 0)
menu_bar.click("File", 0)
```

```python
# Game action system with undo
class GameCharacter:
    def __init__(self, name: str):
        self._name = name
        self._x = 0
        self._y = 0
        self._health = 100

    def move(self, dx: int, dy: int):
        self._x += dx
        self._y += dy
        print(f"{self._name} moved to ({self._x}, {self._y})")

    def attack(self, target: "GameCharacter"):
        damage = 15
        target._health -= damage
        print(f"{self._name} attacks {target._name} for {damage} damage")

    def heal(self, amount: int = 20):
        self._health = min(100, self._health + amount)
        print(f"{self._name} healed to {self._health} HP")


class MoveCommand(Command):
    def __init__(self, character: GameCharacter, dx: int, dy: int):
        self._character = character
        self._dx = dx
        self._dy = dy

    def execute(self):
        self._character.move(self._dx, self._dy)

    def undo(self):
        self._character.move(-self._dx, -self._dy)


class AttackCommand(Command):
    def __init__(self, attacker: GameCharacter, target: GameCharacter):
        self._attacker = attacker
        self._target = target
        self._damage_dealt = 0

    def execute(self):
        self._damage_dealt = 15
        self._attacker.attack(self._target)

    def undo(self):
        self._target._health += self._damage_dealt
        print(f"UNDO: {self._target._name} health restored")


class GameActionHistory:
    def __init__(self):
        self._history: List[Command] = []
        self._limit = 50

    def push(self, command: Command):
        self._history.append(command)
        if len(self._history) > self._limit:
            self._history.pop(0)

    def undo_last(self):
        if self._history:
            cmd = self._history.pop()
            cmd.undo()


player = GameCharacter("Hero")
enemy = GameCharacter("Goblin")

actions = GameActionHistory()
cmd = MoveCommand(player, 5, 0)
cmd.execute()
actions.push(cmd)

cmd = AttackCommand(player, enemy)
cmd.execute()
actions.push(cmd)

actions.undo_last()
actions.undo_last()
```

## Common Mistakes

```python
# MISTAKE 1: Command doing too much
class BloatedCommand:
    def execute(self):
        self.save_to_db()
        self.send_email()
        self.update_cache()
        self.log_activity()
        # A command should delegate to a receiver
```

```python
# MISTAKE 2: Not capturing state for undo
class BadUndoCommand:
    def execute(self):
        self._receiver.change_state()
        # Must save previous state before changing!
```

```python
# MISTAKE 3: Commands that depend on each other
# Commands should be independent and composable
```

```python
# MISTAKE 4: Circular references in command history
# Avoid commands holding references that prevent GC
```

## Best Practices

```python
# 1. Keep commands focused on one operation
# 2. Capture undo state before modifying receiver
# 3. Use macro commands for composite operations
# 4. Limit command history size to prevent memory leaks
# 5. Make commands serializable for logging/replay
```

```python
# Best practice: Context manager for transactions
class TransactionScope:
    def __init__(self, tm: TransactionManager):
        self._tm = tm

    def __enter__(self):
        self._tm.begin_transaction()
        return self._tm

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            self._tm.rollback()
        else:
            self._tm.commit()
```

## Interview Questions

```python
# Q1: Command vs Strategy pattern?
# Command: encapsulates a REQUEST (what to do)
# Strategy: encapsulates an ALGORITHM (how to do it)
```

```python
# Q2: How does Command support undo/redo?
# Store executed commands in history stack;
# each command implements undo() to reverse execute()
```

```python
# Q3: What is a macro command?
# Composite command that executes multiple sub-commands
# in sequence, with inverted undo order
```

## Coding Challenges

```python
# Challenge 1: Implement a job scheduler with
# priority queue and delayed execution
```

```python
# Challenge 2: Build a text editor with full undo/redo
# supporting insert, delete, replace, and format commands
```

```python
# Challenge 3: Create a command-based workflow engine
# with conditional branching and parallel execution
```

```python
# Challenge 4: Implement a multi-level undo system
# with checkpoint/savepoint capability
```

```python
# Challenge 5: Build a remote command executor that
# serializes commands and executes them on a server
```

## Summary

The Command pattern encapsulates requests as objects, enabling parameterization, queuing, logging, and undoable operations. Each command implements execute and optionally undo, holding references to the receiver. Invokers manage command execution and history. The pattern is essential for transaction systems, text editors, job queues, game actions, and any system requiring reversible operations or deferred execution.

## Related Topics

- Strategy Pattern: Similar structure, different intent
- Memento Pattern: Captures object state for undo
- Observer Pattern: Can notify on command execution
- Composite Pattern: Used for macro commands
- Chain of Responsibility: Alternative for request handling
- Transaction Management: Real-world application of Command
