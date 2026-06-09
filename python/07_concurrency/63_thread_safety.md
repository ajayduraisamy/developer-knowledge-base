# Thread Safety - race conditions, Lock, RLock, deadlock prevention
## Introduction
Thread safety is a critical concern in concurrent programming. When multiple threads access shared data without proper synchronization, the program can produce incorrect results, crash, or enter an unrecoverable state. This file covers the fundamental problems (race conditions, deadlocks), the synchronization primitives that solve them (Lock, RLock), and strategies for writing safe concurrent code.

## Race conditions
### What It Is
A race condition occurs when the behavior of a program depends on the relative timing or interleaving of multiple threads accessing shared data. The outcome becomes unpredictable because thread scheduling is nondeterministic.

### Why It Is Important
Race conditions are the most common concurrency bug. They can cause data corruption, crashes, security vulnerabilities, and bugs that are extremely hard to reproduce and debug. They may pass testing and only manifest under specific timing conditions in production.

### How It Works Internally
A race condition typically involves a "read-modify-write" operation that is not atomic at the hardware level. For example, counter += 1 compiles to multiple CPU instructions:
1. Load counter value into register
2. Add 1 to register
3. Store register back to memory

If two threads execute this concurrently, the interleaving can cause one increment to be lost:
- Thread A loads counter=5, Thread B loads counter=5
- Thread A stores counter=6, Thread B stores counter=6 (overwrites A increment!)
- Final value = 6, expected 7

### Beginner Examples
```python
import threading

counter = 0

def increment():
    global counter
    for _ in range(100000):
        counter += 1  # Read-modify-write -- not atomic!

threads = [threading.Thread(target=increment) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()

print(f"Expected: 1000000, Got: {counter}")
```

### Intermediate Examples
```python
import threading
import time

class BankAccount:
    def __init__(self, balance=0):
        self.balance = balance

    def deposit(self, amount):
        new_balance = self.balance + amount
        time.sleep(0.0001)
        self.balance = new_balance

    def withdraw(self, amount):
        if self.balance >= amount:
            new_balance = self.balance - amount
            time.sleep(0.0001)
            self.balance = new_balance
            return True
        return False

account = BankAccount(100)

def deposit_thread():
    for _ in range(100):
        account.deposit(10)

def withdraw_thread():
    for _ in range(100):
        account.withdraw(10)

t1 = threading.Thread(target=deposit_thread)
t2 = threading.Thread(target=withdraw_thread)
t1.start(); t2.start()
t1.join(); t2.join()

print(f"Expected: 100, Got: {account.balance}")
```

### Advanced Examples
```python
import threading

# Check-then-act race condition
shared_dict = {}

def writer():
    for i in range(10000):
        shared_dict[i] = i
        del shared_dict[i]

def reader():
    for _ in range(10000):
        try:
            for key in list(shared_dict.keys()):
                value = shared_dict[key]
        except KeyError:
            print("KeyError race condition!")
            break

t1 = threading.Thread(target=writer)
t2 = threading.Thread(target=reader)
t1.start(); t2.start()
t1.join(); t2.join()
```

### Real-World Use Cases
- E-commerce double-charging due to race on balance check
- Banking overdraft when balance read then debited without lock
- Inventory overselling when two requests decrement count simultaneously
- Caching stale data when cache updates race with reads

### Common Mistakes
- Assuming x += 1 is atomic
- Reading a value, computing, then writing without lock
- Check-then-act patterns without locking
- Iterating a collection while another thread modifies it

### Best Practices
- Protect shared mutable state with locks
- Keep critical sections short
- Prefer immutable data structures
- Use queue.Queue for thread-safe data transfer

### Performance Considerations
- Locking adds overhead
- Contended locks reduce parallelism
- Fine-grained locking increases throughput with more complexity

### Interview Questions
1. What is a race condition and how does it occur?
2. Why is x += 1 not thread-safe in Python?
3. How can you detect race conditions?

### Coding Challenges
- Write a program demonstrating a race condition and fix it
- Create a test that reproduces a specific race condition

### Related Topics
- Lock, RLock, deadlock, thread safety, synchronization

## Lock
### What It Is
threading.Lock ensures mutual exclusion -- only one thread holds the lock at a time, protecting critical sections that access shared data.

### Why It Is Important
Lock is the most fundamental tool for preventing race conditions. It makes non-atomic read-modify-write sequences atomic by ensuring exclusive access.

### How It Works Internally
acquire() blocks until the lock is acquired. release() transitions back to unlocked and wakes one waiting thread. CPython uses OS-level mutexes (pthread_mutex_t on Linux, CRITICAL_SECTION on Windows).

### Syntax
```python
import threading
lock = threading.Lock()

with lock:  # preferred
    shared_data += 1

lock.acquire()
try:
    shared_data += 1
finally:
    lock.release()
```

### Beginner Examples
```python
import threading

counter = 0
lock = threading.Lock()

def safe_increment():
    global counter
    for _ in range(100000):
        with lock:
            counter += 1

threads = [threading.Thread(target=safe_increment) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()

print(f"Expected: 1000000, Got: {counter}")
```

### Intermediate Examples
```python
import threading
import time

class SafeBankAccount:
    def __init__(self, balance=0):
        self.balance = balance
        self.lock = threading.Lock()

    def deposit(self, amount):
        with self.lock:
            new_balance = self.balance + amount
            time.sleep(0.0001)
            self.balance = new_balance

    def withdraw(self, amount):
        with self.lock:
            if self.balance >= amount:
                new_balance = self.balance - amount
                time.sleep(0.0001)
                self.balance = new_balance
                return True
            return False

    def get_balance(self):
        with self.lock:
            return self.balance

account = SafeBankAccount(1000)

def safe_deposits():
    for _ in range(100):
        account.deposit(10)

def safe_withdrawals():
    for _ in range(100):
        account.withdraw(10)

threads = [threading.Thread(target=safe_deposits), threading.Thread(target=safe_withdrawals)]
for t in threads: t.start()
for t in threads: t.join()
print(f"Balance: {account.get_balance()}")
```

### Advanced Examples
```python
import threading
import time

class ReadWriteLock:
    def __init__(self):
        self._read_lock = threading.Lock()
        self._write_lock = threading.Lock()
        self._readers = 0

    def acquire_read(self):
        with self._read_lock:
            self._readers += 1
            if self._readers == 1:
                self._write_lock.acquire()

    def release_read(self):
        with self._read_lock:
            self._readers -= 1
            if self._readers == 0:
                self._write_lock.release()

    def acquire_write(self):
        self._write_lock.acquire()

    def release_write(self):
        self._write_lock.release()

class ReadLock:
    def __init__(self, rw_lock):
        self.rw_lock = rw_lock
    def __enter__(self):
        self.rw_lock.acquire_read()
    def __exit__(self, *args):
        self.rw_lock.release_read()

class WriteLock:
    def __init__(self, rw_lock):
        self.rw_lock = rw_lock
    def __enter__(self):
        self.rw_lock.acquire_write()
    def __exit__(self, *args):
        self.rw_lock.release_write()

rw = ReadWriteLock()
shared_data = []

def reader():
    with ReadLock(rw):
        print(f"Reading: {len(shared_data)} items")

def writer():
    with WriteLock(rw):
        shared_data.append("data")
```

### Real-World Use Cases
- Database transactions -- atomic read-modify-write on records
- Cache updates -- preventing stale reads during refresh
- Resource pools -- thread-safe connection checkout

### Common Mistakes
- Forgetting to release lock (always use with lock)
- Holding lock during I/O or network calls
- Acquiring locks in different order (deadlocks)

### Best Practices
- Always use with lock: context manager
- Keep critical sections as short as possible
- Use consistent lock ordering
- Use RLock if same thread needs to re-acquire

### Performance Considerations
- Uncontended lock acquire/release is fast (tens of nanoseconds)
- Contended locks cause context switches (microseconds)

### Interview Questions
1. What is the difference between a mutex and a semaphore?
2. Why use with lock: instead of acquire/release?
3. What happens calling acquire on a lock already held by the same thread?

### Coding Challenges
- Implement a thread-safe stack using Lock
- Build a reader-writer lock using threading.Lock

### Related Topics
- RLock, Semaphore, Condition, Event, deadlock

## RLock
### What It Is
threading.RLock (Reentrant Lock) can be acquired multiple times by the same thread without blocking. It maintains an ownership count -- each acquire increments it, each release decrements it. The lock is fully released only when count reaches zero.

### Why It Is Important
RLock prevents self-deadlock when a function holding a lock calls another function (or itself) that also needs the same lock.

### How It Works Internally
RLock tracks owning thread ID and acquire count. When a thread calls acquire():
- If unlocked, ownership = current thread, count = 1
- If owned by current thread, count += 1 (no blocking)
- If owned by another thread, block until released

release() decrements count. When count = 0, ownership is cleared.

### Syntax
```python
import threading
rlock = threading.RLock()

with rlock:
    with rlock:  # Same thread -- no deadlock
        pass
```

### Beginner Examples
```python
import threading

rlock = threading.RLock()

def outer():
    with rlock:
        print("Outer acquired lock")
        inner()

def inner():
    with rlock:
        print("Inner acquired lock (reentrant)")

t = threading.Thread(target=outer)
t.start()
t.join()
```

### Intermediate Examples
```python
import threading

class Counter:
    def __init__(self):
        self.value = 0
        self.lock = threading.RLock()

    def increment(self):
        with self.lock:
            self.value += 1

    def increment_by(self, n):
        with self.lock:
            if n > 1:
                self.increment_by(n - 1)
            self.increment()

counter = Counter()
threads = [threading.Thread(target=lambda: counter.increment_by(50)) for _ in range(5)]
for t in threads: t.start()
for t in threads: t.join()
print(f"Counter: {counter.value}")
```

### Advanced Examples
```python
import threading

class TreeNode:
    def __init__(self, value):
        self.value = value
        self.children = []
        self.lock = threading.RLock()

    def add_child(self, child):
        with self.lock:
            self.children.append(child)

    def find(self, value):
        with self.lock:
            if self.value == value:
                return self
            for child in self.children:
                result = child.find(value)
                if result is not None:
                    return result
            return None

root = TreeNode("A")
root.add_child(TreeNode("B"))
root.children[0].add_child(TreeNode("C"))
print(f"Found: {root.find("C").value}")
```

### Real-World Use Cases
- Recursive algorithms -- tree/graph traversal with thread safety
- Nested method calls -- public API calling internal locked methods
- Decorators adding locking to methods that may be recursive

### Common Mistakes
- Using Lock when RLock is needed (causes deadlocks)
- Using RLock when Lock suffices (slightly more overhead)

### Best Practices
- Use RLock when same thread may acquire lock multiple times
- Prefer Lock for simple mutual exclusion

### Performance Considerations
- RLock has slightly more overhead than Lock
- Nearly identical performance in practice

### Interview Questions
1. Difference between Lock and RLock?
2. When should you use RLock?
3. How does RLock track ownership internally?

### Coding Challenges
- Implement RLock from scratch using threading.Lock
- Demonstrate deadlock with Lock fixed by switching to RLock

### Related Topics
- Lock, deadlock, threading, Condition

## Deadlock prevention
### What It Is
A deadlock occurs when two or more threads are each waiting for the other to release a resource, causing all to be stuck forever. Deadlock prevention uses techniques to avoid this condition.

### Why It Is Important
Deadlocks are catastrophic -- they freeze the program with no recovery and are hard to debug. Prevention is essential in multithreaded design.

### How It Works Internally
Four conditions must hold for deadlock (Coffman conditions):
1. Mutual exclusion -- resources cannot be shared
2. Hold and wait -- threads hold resources while waiting for others
3. No preemption -- resources cannot be forcibly taken
4. Circular wait -- a cycle of threads each waiting for the next

Breaking any one condition prevents deadlock. The most common approach is breaking circular wait via lock ordering.

### Syntax
```python
# Lock ordering -- always acquire in the same order
# Good:
with lock_a:
    with lock_b:
        pass

# Bad (potential deadlock):
def thread1():
    with lock_a:
        with lock_b: pass

def thread2():
    with lock_b:
        with lock_a: pass
```

### Beginner Examples
```python
import threading
import time

# Deadlock example
lock_a = threading.Lock()
lock_b = threading.Lock()

def thread_1():
    with lock_a:
        time.sleep(0.1)
        with lock_b:
            print("Thread 1 got both locks")

def thread_2():
    with lock_b:
        time.sleep(0.1)
        with lock_a:
            print("Thread 2 got both locks")

t1 = threading.Thread(target=thread_1)
t2 = threading.Thread(target=thread_2)
t1.start()
t2.start()
# These may deadlock!
```

### Intermediate Examples
```python
import threading
import time

# Solution: consistent lock ordering
lock_a = threading.Lock()
lock_b = threading.Lock()

def thread_1():
    with lock_a:
        time.sleep(0.1)
        with lock_b:
            print("Thread 1 done")

def thread_2():
    with lock_a:  # Same order as thread_1
        time.sleep(0.1)
        with lock_b:
            print("Thread 2 done")

# Or use resource ordering by id
def safe_acquire(locks):
    for lock in sorted(locks, key=id):
        lock.acquire()

def safe_release(locks):
    for lock in sorted(locks, key=id, reverse=True):
        lock.release()

# Timeout-based approach
def try_acquire_with_timeout(lock, timeout=5):
    return lock.acquire(timeout=timeout)

# Resource hierarchy
def transfer(from_acct, to_acct, amount):
    first = min(id(from_acct), id(to_acct))
    second = max(id(from_acct), id(to_acct))

    first_lock = from_acct.lock if id(from_acct) == first else to_acct.lock
    second_lock = to_acct.lock if id(from_acct) == first else from_acct.lock

    with first_lock:
        with second_lock:
            from_acct.balance -= amount
            to_acct.balance += amount
```

### Advanced Examples
```python
import threading
import time
from contextlib import contextmanager

# Deadlock detector and recovery
class DeadlockAwareLock:
    def __init__(self, name):
        self.lock = threading.Lock()
        self.name = name
        self.owner = None

    def acquire(self, timeout=10):
        acquired = self.lock.acquire(timeout=timeout)
        if acquired:
            self.owner = threading.current_thread().name
        else:
            raise TimeoutError(f"Deadlock suspected on {self.name}")
        return acquired

    def release(self):
        self.owner = None
        self.lock.release()

    def __enter__(self):
        self.acquire()
        return self

    def __exit__(self, *args):
        self.release()

# Usage
a = DeadlockAwareLock("A")
b = DeadlockAwareLock("B")

def worker1():
    with a:
        time.sleep(0.1)
        with b:
            print("Worker 1 done")

# Bank transfer with deadlock prevention
class Account:
    _id_counter = 0
    _id_lock = threading.Lock()

    def __init__(self, name, balance):
        with Account._id_lock:
            self._id = Account._id_counter
            Account._id_counter += 1
        self.name = name
        self.balance = balance
        self.lock = threading.RLock()

    def transfer(self, target, amount):
        # Lock ordering by account ID
        first, second = (self, target) if self._id < target._id else (target, self)
        with first.lock:
            with second.lock:
                if self.balance >= amount:
                    self.balance -= amount
                    target.balance += amount
                    return True
                return False

# Test
a1 = Account("A", 1000)
a2 = Account("B", 1000)

def transfer_many():
    for _ in range(100):
        a1.transfer(a2, 10)
        a2.transfer(a1, 10)

threads = [threading.Thread(target=transfer_many) for _ in range(5)]
for t in threads: t.start()
for t in threads: t.join()
print(f"A: {a1.balance}, B: {a2.balance}")
```

### Real-World Use Cases
- Banking systems -- transfer between accounts must prevent deadlock
- Database transaction ordering -- acquire table locks in consistent order
- Resource allocation -- multiple resource acquisition in operating systems
- Fork-join patterns -- preventing deadlock in parallel algorithms

### Common Mistakes
- Inconsistent lock ordering across threads
- Nested locks without thinking about ordering
- Holding locks while waiting for user input or network responses
- Using Lock when RLock is needed (self-deadlock)

### Best Practices
- Enforce consistent lock ordering (by id, by hash, by natural order)
- Use timeout on lock acquisition to detect potential deadlocks
- Minimize the number of locks held simultaneously
- Use a single lock to protect multiple related resources (coarse locking)
- Document lock hierarchy clearly
- Use higher-level abstractions (Queue, concurrent.futures) to reduce raw lock usage

### Performance Considerations
- Lock ordering adds minimal overhead
- Timeout-based detection adds timeout overhead
- Deadlock detection (wait-for graph) is expensive and rarely used
- Coarse locking is simpler but reduces concurrency

### Interview Questions
1. What four conditions must hold for a deadlock to occur?
2. How does lock ordering prevent deadlocks?
3. What is the difference between deadlock prevention, avoidance, and detection?
4. How would you debug a deadlocked Python program?
5. Can a single thread deadlock with itself?

### Coding Challenges
- Write a program that reliably deadlocks
- Fix the deadlock using lock ordering
- Implement a transfer() function between accounts that prevents deadlock
- Build a simple deadlock detector that monitors lock acquisition order

### Related Topics
- Lock, RLock, race conditions, threading, dining philosophers problem
