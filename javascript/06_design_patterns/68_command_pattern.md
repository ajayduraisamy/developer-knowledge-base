# Command Pattern - Encapsulated requests, undo/redo, command history

## Introduction

The Command Pattern encapsulates a request as an object, allowing parameterization of clients with different requests, queuing of operations, and support for undoable operations. It decouples the object that invokes the operation from the one that knows how to perform it.

In JavaScript, the Command Pattern is essential for implementing undo/redo systems, transaction management, task queues, and macro recording. Its power lies in treating actions as first-class objects that can be stored, passed around, and executed at a later time.

## Encapsulated Requests

### What It Is

A command wraps all information needed to perform an action: the method to call, the target object, and the parameters. This command object can be stored, transmitted, and executed independently of the original caller.

### Why It Is Important

Encapsulating requests as objects provides: separation of concerns between invoker and receiver, ability to parameterize operations with different requests, support for queuing and scheduling, easy logging and auditing, and the foundation for undo/redo and transactions.

### How It Works Internally

The command object holds a reference to the receiver and the parameters. When execute() is called, the command calls the appropriate method on the receiver. The invoker (client code) doesn't know what receiver will execute or how — it only calls execute() on the command.

### Syntax

```javascript
class Command {
  constructor(receiver, ...params) { this.receiver = receiver; this.params = params; }
  execute() { this.receiver.action(...this.params); }
  undo() { this.receiver.reverseAction(...this.params); }
}
```

### Beginner Examples

```javascript
class Calculator {
  constructor() { this.value = 0; }
  add(x) { this.value += x; }
  sub(x) { this.value -= x; }
  getValue() { return this.value; }
}
class AddCommand {
  constructor(calc, val) { this.calc = calc; this.val = val; }
  execute() { this.calc.add(this.val); }
  undo() { this.calc.sub(this.val); }
}
const calc = new Calculator();
const cmd = new AddCommand(calc, 5);
cmd.execute();
console.log(calc.getValue()); // 5
cmd.undo();
console.log(calc.getValue()); // 0
```

### Intermediate Examples

```javascript
class Light {
  constructor(loc) { this.loc = loc; this.on = false; }
  turnOn() { this.on = true; console.log(this.loc + ' on'); }
  turnOff() { this.on = false; console.log(this.loc + ' off'); }
}
class LightOnCmd {
  constructor(l) { this.light = l; }
  execute() { this.light.turnOn(); }
  undo() { this.light.turnOff(); }
}
class MacroCommand {
  constructor(cmds) { this.commands = cmds; }
  execute() { this.commands.forEach(c => c.execute()); }
  undo() { this.commands.slice().reverse().forEach(c => c.undo()); }
}

// Remote control with command slots
class RemoteControl {
  constructor() { this.slots = new Array(7); }
  setCommand(slot, onCmd, offCmd) { this.slots[slot] = { on: onCmd, off: offCmd }; }
  pressOn(slot) { if (this.slots[slot]) this.slots[slot].on.execute(); }
  pressOff(slot) { if (this.slots[slot]) this.slots[slot].off.execute(); }
}
```

### Advanced Examples

```javascript
class Transaction {
  constructor() { this.commands = []; this.executed = []; }
  add(cmd) { this.commands.push(cmd); }
  execute() {
    for (const cmd of this.commands) {
      try { cmd.execute(); this.executed.push(cmd); }
      catch(e) { this.rollback(); throw e; }
    }
    this.commands = [];
  }
  rollback() { this.executed.reverse().forEach(c => { try { c.undo(); } catch(e) {} }); this.executed = []; }
}

class AsyncCommand {
  constructor(fn) { this.fn = fn; this.status = 'pending'; }
  async execute() {
    this.status = 'running';
    try { const r = await this.fn(); this.status = 'done'; return r; }
    catch(e) { this.status = 'failed'; throw e; }
  }
}

// Command queue with processing
class CommandQueue {
  constructor() { this.queue = []; this.processing = false; }
  enqueue(cmd) { this.queue.push(cmd); this.process(); }
  async process() {
    if (this.processing) return;
    this.processing = true;
    while (this.queue.length > 0) {
      const cmd = this.queue.shift();
      try { await cmd.execute(); } catch(e) { console.error('Command failed:', e); }
    }
    this.processing = false;
  }
}
```

### Real-World Use Cases
- GUI buttons and menu items where each action is a command object
- Transaction processing for atomic multi-step operations
- Task scheduling for delayed or periodic execution
- Game input handling where each player action is a command
- Macro recording for replaying sequences of operations

### Common Mistakes
- Commands with too much responsibility (should only execute one action)
- Not implementing undo when the command modifies state
- Command holding references to large objects preventing GC
- Not providing serialization for persistent command history

### Best Practices
- Keep commands small and focused on a single action
- Implement both execute() and undo() for state-modifying commands
- Return cleanup functions from command execution
- Use command factory for parameterized command creation

## Undo/Redo Functionality

### What It Is

Undo reverses commands; redo reapplies reversed commands. This is implemented by maintaining a history stack of executed commands with their undo capabilities.

### Why It Is Important

Undo/redo is critical for user experience in creative and productivity applications. It gives users confidence to experiment knowing they can revert mistakes. Essential in text editors, image editors, IDEs, and design tools.

### How It Works Internally

A history object maintains two stacks: undo stack and redo stack. When a command executes, it's pushed to the undo stack and the redo stack is cleared. On undo, the command is popped from undo stack, its undo() is called, and it's pushed to redo stack. On redo, the command is popped from redo stack, execute() is called, and it's pushed to undo stack.

### Syntax

```javascript
class CommandHistory {
  constructor() { this.undo = []; this.redo = []; }
  execute(cmd) { cmd.execute(); this.undo.push(cmd); this.redo = []; }
  undo() { const c = this.undo.pop(); if (c) { c.undo(); this.redo.push(c); } }
  redo() { const c = this.redo.pop(); if (c) { c.execute(); this.undo.push(c); } }
}
```

### Beginner Examples

```javascript
class Canvas {
  constructor() { this.shapes = []; }
  add(s) { this.shapes.push(s); }
  remove(s) { this.shapes = this.shapes.filter(x => x !== s); }
  getShapes() { return [...this.shapes]; }
}
class AddShapeCmd {
  constructor(c, s) { this.canvas = c; this.shape = s; }
  execute() { this.canvas.add(this.shape); }
  undo() { this.canvas.remove(this.shape); }
}
const canvas = new Canvas();
const h = new CommandHistory();
h.execute(new AddShapeCmd(canvas, {id:1}));
h.undo(); // Removes shape
h.redo(); // Re-adds shape
```

### Intermediate Examples

```javascript
class TextDocument {
  constructor() { this.content = ''; }
  insert(p, t) { this.content = this.content.slice(0,p) + t + this.content.slice(p); }
  delete(p, l) { const d = this.content.slice(p,p+l); this.content = this.content.slice(0,p) + this.content.slice(p+l); return d; }
}
class InsertCmd {
  constructor(doc, pos, text) { this.doc = doc; this.pos = pos; this.text = text; }
  execute() { this.doc.insert(this.pos, this.text); }
  undo() { this.doc.delete(this.pos, this.text.length); }
}
class DeleteCmd {
  constructor(doc, pos, len) { this.doc = doc; this.pos = pos; this.len = len; this.deleted = ''; }
  execute() { this.deleted = this.doc.delete(this.pos, this.len); }
  undo() { this.doc.insert(this.pos, this.deleted); }
}
class CompoundCommand {
  constructor() { this.commands = []; }
  add(c) { this.commands.push(c); }
  execute() { this.commands.forEach(c => c.execute()); }
  undo() { this.commands.slice().reverse().forEach(c => c.undo()); }
}
```

### Best Practices
- Limit history size to prevent memory issues (e.g., max 200 commands)
- Clear redo stack on new actions after undo
- Group rapid successive operations into one undo step
- Deep clone state for snapshot-based undo
- Use compound commands for grouping related operations

### Performance Considerations
- Command history grows linearly with operations; cap the size
- Deep cloning state for each history point is memory-intensive
- Use bounded history or serialize to disk for long sessions
- Batching operations reduces history size and memory usage

## Command History

### What It Is

Stores executed commands for later reference, analysis, replay, or undo/redo. Enables audit logging, macro recording, and time-travel debugging.

### Why It Is Important

Command history transforms applications from stateless request-response systems into stateful, auditable platforms. It enables audit trails for compliance, macro recording and replay, debugging through execution replay, analytics on user actions, and crash recovery by replaying commands.

### Syntax

```javascript
class History {
  constructor(max = 200) { this.cmds = []; this.pos = -1; this.max = max; }
  record(c) {
    this.cmds = this.cmds.slice(0, this.pos + 1);
    this.cmds.push(c);
    if (this.cmds.length > this.max) this.cmds.shift();
    this.pos = this.cmds.length - 1;
  }
  getAll() { return [...this.cmds]; }
}
```

### Advanced Examples

```javascript
class ReplicatedHistory {
  constructor(siteId) { this.siteId = siteId; this.cmds = []; this.clock = 0; }
  gen(cmd) { return { id: this.siteId + ':' + (this.clock++), cmd, ts: Date.now() }; }
  local(cmd) { const e = this.gen(cmd); cmd.execute(); this.cmds.push(e); return e; }
  remote(e) { this.cmds.push(e); this.cmds.sort((a,b) => a.id.localeCompare(b.id)); e.cmd.execute(); }
}

class AnalyticsHistory {
  constructor() { this.execs = []; this.timings = new Map(); }
  record(cmd, dur) {
    const n = cmd.constructor.name; this.execs.push({n,dur,ts:Date.now()});
    if (!this.timings.has(n)) this.timings.set(n, []);
    this.timings.get(n).push(dur);
  }
  getStats() {
    const s = {};
    for (const [n, t] of this.timings) {
      s[n] = { count: t.length, avg: (t.reduce((a,b)=>a+b,0)/t.length).toFixed(2), max: Math.max(...t).toFixed(2) };
    }
    return s;
  }
}

// Snapshot-based history for non-reversible operations
class SnapshotHistory {
  constructor(initialState) {
    this.states = [JSON.parse(JSON.stringify(initialState))];
    this.idx = 0;
    this.maxSize = 100;
  }
  save(state) {
    this.states = this.states.slice(0, this.idx + 1);
    this.states.push(JSON.parse(JSON.stringify(state)));
    if (this.states.length > this.maxSize) this.states.shift();
    this.idx = this.states.length - 1;
  }
  undo() { if (this.idx > 0) { this.idx--; return JSON.parse(JSON.stringify(this.states[this.idx])); } return null; }
  redo() { if (this.idx < this.states.length-1) { this.idx++; return JSON.parse(JSON.stringify(this.states[this.idx])); } return null; }
}
```

### Interview Questions

**Q: How does event sourcing relate to the Command Pattern?**
A: Event sourcing stores all changes as events. Current state derives from replaying events. This is the Command Pattern at the architecture level, enabling complete audit trails and time-travel debugging.

**Q: How to implement collaborative undo in a real-time editor?**
A: Use replicated command history with vector clocks or CRDTs. Undo is a compensating command applied by all clients. Conflicts are resolved through operational transformation or CRDT merge rules.

**Q: What is the difference between command-based undo and state-based undo?**
A: Command-based undo stores commands with undo logic, which is memory-efficient but requires undo implementations. State-based undo stores full state snapshots, which is simpler but uses more memory. Hybrid approaches use both.

### Coding Challenges

```javascript
// Challenge 1: Implement a text editor with commands for insert(char),
// delete(), and moveCursor(position). Support undo/redo with Ctrl+Z/Y.
// Challenge 2: Build a macro recorder that records commands and
// can replay them. Support serialization to JSON for persistence.
// Challenge 3: Create a distributed command system where commands
// are serialized, sent over WebSocket, and executed on remote servers.
```

### Related Topics
- Event sourcing, CQRS, Memento pattern, Operational transformation, CRDT
