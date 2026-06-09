# SQLite - sqlite3.connect(), execute(), fetchall(), transactions

## Introduction
SQLite is a self-contained, serverless, zero-configuration SQL database engine that is built into Python's standard library via the `sqlite3` module. Unlike client-server databases (PostgreSQL, MySQL), SQLite reads and writes directly to a single file on disk, making it ideal for embedded applications, prototyping, local development, and situations where simplicity and minimal setup are paramount. Python's `sqlite3` module provides a DB-API 2.0 compliant interface, allowing developers to execute SQL queries, manage transactions, and handle results with a consistent API. This module is especially useful for small to medium-sized data storage needs, testing environments, mobile applications, and any scenario where deploying a full database server would be overkill.

## sqlite3.connect()

### What It Is
`sqlite3.connect()` is the entry point for interacting with an SQLite database. It creates a Connection object that manages the connection to a database file. If the specified file does not exist, SQLite automatically creates it. You can also create an in-memory database by passing `":memory:"` as the filename.

### Why It Is Important
The Connection object is the gateway to all database operations. Without it, you cannot execute queries, manage transactions, or retrieve results. Understanding how to properly create and manage connections is foundational to using SQLite effectively in Python.

### How It Works Internally
When `sqlite3.connect()` is called, Python's C extension opens a handle to the SQLite library using `sqlite3_open()`. The library either opens an existing database file or creates a new one. The Connection object wraps this C-level handle and exposes Pythonic methods for query execution, transaction control, and result fetching. Under the hood, SQLite uses a B-tree data structure for table storage, with pages managed by a pager subsystem that handles caching, locking, and journaling for crash recovery.

### Syntax
```python
import sqlite3

# Connect to a file-based database
conn = sqlite3.connect("example.db")

# Connect to an in-memory database
conn = sqlite3.connect(":memory:")

# Connect with a timeout (seconds)
conn = sqlite3.connect("example.db", timeout=10)
```

### Beginner Examples
```python
import sqlite3

conn = sqlite3.connect("library.db")
cursor = conn.cursor()
cursor.execute("CREATE TABLE IF NOT EXISTS books (id INTEGER PRIMARY KEY, title TEXT, author TEXT)")
cursor.execute("INSERT INTO books (title, author) VALUES (?, ?)", ("1984", "George Orwell"))
conn.commit()
cursor.execute("SELECT * FROM books")
print(cursor.fetchall())
conn.close()
```

### Intermediate Examples
```python
import sqlite3
from contextlib import closing

def create_connection(db_path):
    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row  # Access columns by name
    conn.execute("PRAGMA journal_mode=WAL")
    conn.execute("PRAGMA foreign_keys=ON")
    return conn

with closing(create_connection("inventory.db")) as conn:
    conn.executescript("""
        CREATE TABLE IF NOT EXISTS categories (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL UNIQUE
        );
        CREATE TABLE IF NOT EXISTS products (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            price REAL NOT NULL,
            category_id INTEGER REFERENCES categories(id)
        );
    """)
    categories = [("Electronics",), ("Books",), ("Clothing",)]
    conn.executemany("INSERT OR IGNORE INTO categories (name) VALUES (?)", categories)
    conn.commit()
```

### Advanced Examples
```python
import sqlite3
import threading
import queue

class SQLiteConnectionPool:
    def __init__(self, db_path, pool_size=5):
        self.db_path = db_path
        self._pool = queue.Queue(pool_size)
        for _ in range(pool_size):
            conn = sqlite3.connect(db_path, check_same_thread=False)
            conn.execute("PRAGMA journal_mode=WAL")
            conn.execute("PRAGMA busy_timeout=5000")
            self._pool.put(conn)

    def get_conn(self):
        return self._pool.get()

    def return_conn(self, conn):
        self._pool.put(conn)

    def close_all(self):
        while not self._pool.empty():
            self._pool.get().close()

# Usage with threading
pool = SQLiteConnectionPool("concurrent.db")
def worker(worker_id):
    conn = pool.get_conn()
    try:
        conn.execute("INSERT INTO logs (message) VALUES (?)", (f"Worker {worker_id}",))
        conn.commit()
    finally:
        pool.return_conn(conn)

threads = [threading.Thread(target=worker, args=(i,)) for i in range(10)]
for t in threads: t.start()
for t in threads: t.join()
pool.close_all()
```

### Real-World Use Cases
- **Embedded databases in desktop applications**: Firefox, Chrome, and many apps use SQLite for local data storage.
- **Mobile applications**: Both Android and iOS include SQLite for local persistence.
- **Data analysis and ETL pipelines**: Temporary databases for transforming and querying datasets.
- **Testing**: In-memory SQLite databases provide isolated, fast test fixtures.
- **Prototyping**: Rapid application development without setting up a database server.

### Common Mistakes
- **Forgetting to commit**: INSERT/UPDATE/DELETE changes are lost if `commit()` is not called.
- **Not closing connections**: Leaving connections open can cause file locking issues.
- **Using string formatting for queries**: Using f-strings for SQL parameters opens SQL injection vulnerabilities. Always use parameterized queries with `?` placeholders.
- **Sharing connections across threads without `check_same_thread=False`**: By default, SQLite connections are single-threaded.
- **Ignoring `row_factory`**: Default behavior returns tuples; using `sqlite3.Row` or custom factories improves readability.

### Best Practices
- Use `contextlib.closing()` or context managers to ensure connections are closed.
- Enable WAL mode for better concurrent read performance.
- Use parameterized queries (`?` placeholders) to prevent SQL injection.
- Set `row_factory = sqlite3.Row` for named column access.
- Use `executemany()` for batch inserts instead of looping.
- Enable foreign keys with `PRAGMA foreign_keys=ON` at connection time.

## execute() and executemany()

### What It Is
`execute()` runs a single SQL statement, optionally with parameters. `executemany()` runs the same SQL statement repeatedly for each item in a sequence of parameter tuples. Both methods are called on a Cursor object obtained from the connection.

### Syntax
```python
cursor = conn.cursor()

# Single execution with parameters
cursor.execute("INSERT INTO users (name, age) VALUES (?, ?)", ("Alice", 30))

# Multiple executions
users = [("Bob", 25), ("Charlie", 35), ("Diana", 28)]
cursor.executemany("INSERT INTO users (name, age) VALUES (?, ?)", users)

conn.commit()
```

### Intermediate Examples
```python
conn = sqlite3.connect(":memory:")
cursor = conn.cursor()
cursor.execute("CREATE TABLE inventory (item TEXT, quantity INTEGER, price REAL)")

# Using named placeholders
items = [
    {"item": "Widget", "qty": 100, "price": 9.99},
    {"item": "Gadget", "qty": 50, "price": 24.99},
]
cursor.executemany(
    "INSERT INTO inventory VALUES (:item, :qty, :price)",
    items
)

# Bulk insert with error handling
data = [("A", 10), ("B", 20), ("A", 5)]  # Duplicate intentional
try:
    conn.execute("CREATE UNIQUE INDEX idx_item ON inventory(item)")
    cursor.executemany("INSERT INTO inventory VALUES (?, 0, 0.0)", data)
    conn.commit()
except sqlite3.IntegrityError as e:
    conn.rollback()
    print(f"Integrity error: {e}")
```

## fetchall() and fetchone()

### What It Is
`fetchall()` retrieves all remaining rows of a query result as a list of tuples (or Row objects). `fetchone()` retrieves the next row, returning `None` when no more rows exist. `fetchmany(size)` retrieves a specified number of rows.

### Syntax
```python
cursor.execute("SELECT id, name, age FROM users")

# Fetch one row
row = cursor.fetchone()
print(f"First user: {row[0]}, {row[1]}, {row[2]}")

# Fetch all remaining rows
all_rows = cursor.fetchall()
for row in all_rows:
    print(row)

# Fetch in batches
while True:
    batch = cursor.fetchmany(100)
    if not batch:
        break
    for row in batch:
        process(row)
```

### Advanced Examples
```python
conn = sqlite3.connect(":memory:")
conn.row_factory = sqlite3.Row
conn.execute("CREATE TABLE sales (id INTEGER PRIMARY KEY, product TEXT, amount REAL)")

sales_data = [("Laptop", 1200.00), ("Mouse", 25.50), ("Keyboard", 89.99)]
conn.executemany("INSERT INTO sales (product, amount) VALUES (?, ?)", sales_data)
conn.commit()

# Named column access with Row factory
cursor = conn.execute("SELECT * FROM sales WHERE amount > ?", (50,))
for row in cursor.fetchall():
    print(f"{row['product']}: ${row['amount']:.2f}")

# Memory-efficient iteration (no fetchall needed)
cursor = conn.execute("SELECT * FROM sales")
for row in cursor:  # Cursor as iterator
    print(dict(row))

# Paginated fetching
cursor = conn.execute("SELECT * FROM sales ORDER BY id")
page_size = 2
page = 1
while True:
    rows = cursor.fetchmany(page_size)
    if not rows:
        break
    print(f"Page {page}: {[dict(r) for r in rows]}")
    page += 1
```

## Transactions

### What It Is
A transaction is a sequence of database operations treated as a single atomic unit. SQLite's `sqlite3` module handles transactions implicitly: any data modification statement (INSERT, UPDATE, DELETE) automatically starts a transaction, and changes are only visible to other connections after `commit()`. Calling `rollback()` discards all changes since the last commit.

### Why It Is Important
Transactions ensure data consistency and integrity. They guarantee that either all operations in a group succeed or none are applied, which is critical for financial transactions, inventory management, and any system requiring ACID compliance.

### How It Works Internally
SQLite implements transactions using a journal file (rollback journal or WAL file). When a transaction begins, SQLite records the original page data in the journal. If a COMMIT occurs, the journal is cleared and changes become permanent. If a ROLLBACK occurs, the original pages are restored from the journal. In WAL mode, changes are appended to a write-ahead log, and readers can continue reading the old data until a checkpoint moves changes to the main database.

### Syntax
```python
conn = sqlite3.connect("bank.db")
conn.execute("CREATE TABLE IF NOT EXISTS accounts (id INTEGER PRIMARY KEY, name TEXT, balance REAL)")
conn.execute("INSERT INTO accounts VALUES (1, 'Alice', 1000.00)")
conn.execute("INSERT INTO accounts VALUES (2, 'Bob', 500.00)")
conn.commit()

# Explicit transaction control
try:
    conn.execute("BEGIN TRANSACTION")
    conn.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
    conn.execute("UPDATE accounts SET balance = balance + 100 WHERE id = 2")
    conn.commit()
    print("Transfer successful")
except sqlite3.Error:
    conn.rollback()
    print("Transfer failed, rolled back")
```

### Intermediate Examples
```python
conn = sqlite3.connect("orders.db")
conn.execute("PRAGMA foreign_keys=ON")
conn.executescript("""
    CREATE TABLE IF NOT EXISTS orders (
        id INTEGER PRIMARY KEY,
        customer TEXT NOT NULL,
        total REAL NOT NULL
    );
    CREATE TABLE IF NOT EXISTS order_items (
        id INTEGER PRIMARY KEY,
        order_id INTEGER REFERENCES orders(id),
        product TEXT NOT NULL,
        price REAL NOT NULL
    );
""")

def place_order(customer, items):
    try:
        conn.execute("BEGIN TRANSACTION")
        cursor = conn.execute(
            "INSERT INTO orders (customer, total) VALUES (?, ?)",
            (customer, sum(p for _, p in items))
        )
        order_id = cursor.lastrowid
        conn.executemany(
            "INSERT INTO order_items (order_id, product, price) VALUES (?, ?, ?)",
            [(order_id, product, price) for product, price in items]
        )
        conn.commit()
        return order_id
    except sqlite3.Error as e:
        conn.rollback()
        raise RuntimeError(f"Order failed: {e}")

place_order("Alice", [("Widget", 9.99), ("Gadget", 24.99)])
```

### Advanced Examples - Savepoints and Nested Transactions
```python
conn = sqlite3.connect(":memory:")
conn.executescript("""
    CREATE TABLE items (id INTEGER PRIMARY KEY, name TEXT, version INTEGER DEFAULT 1);
    INSERT INTO items (name) VALUES ('Item A'), ('Item B');
""")

def update_with_savepoint(item_id, new_name):
    savepoint = f"sp_{item_id}"
    try:
        conn.execute(f"SAVEPOINT {savepoint}")
        conn.execute("UPDATE items SET name = ?, version = version + 1 WHERE id = ?",
                     (new_name, item_id))
        # Simulate an error for item 2
        if item_id == 2:
            raise ValueError("Simulated error")
        conn.execute(f"RELEASE SAVEPOINT {savepoint}")
    except Exception as e:
        print(f"Rolling back savepoint: {e}")
        conn.execute(f"ROLLBACK TO SAVEPOINT {savepoint}")

update_with_savepoint(1, "Updated A")
update_with_savepoint(2, "Updated B")  # Will rollback only this savepoint

cursor = conn.execute("SELECT * FROM items")
print(cursor.fetchall())  # Item 1 updated, Item 2 unchanged
```

### Real-World Use Cases
- **Banking systems**: Atomic transfers between accounts.
- **E-commerce order processing**: Creating orders, reducing inventory, processing payments atomically.
- **Batch data imports**: Rolling back entirely if any record fails validation.
- **Configuration management**: Atomic updates to multiple configuration tables.

### Common Mistakes
- **Assuming autocommit**: SQLite uses implicit transactions; an explicit `commit()` is required.
- **Not handling `IntegrityError`**: Foreign key violations or unique constraint failures should trigger rollbacks.
- **Long-running transactions**: Holding transactions open for too long blocks other writers.
- **Mixing explicit and implicit transactions**: Calling `execute()` after `BEGIN TRANSACTION` without proper commit/rollback.

### Best Practices
- Always wrap mutation sequences in `try/except` with explicit `rollback()`.
- Use `BEGIN IMMEDIATE` or `BEGIN EXCLUSIVE` in concurrent environments to avoid "database is locked" errors.
- Keep transactions short to minimize locking contention.
- Use savepoints for partial rollbacks in complex operations.
- Enable foreign keys and WAL mode for production applications.

### Performance Considerations
- Use `executemany()` for batch inserts (100-1000x faster than individual `execute()` calls).
- Wrap bulk operations in a single transaction rather than autocommitting each statement.
- Use `PRAGMA synchronous = NORMAL` for faster writes (slightly less crash safety).
- Increase cache size with `PRAGMA cache_size = -20000` (20MB) for large queries.
- Create appropriate indexes on columns used in WHERE and JOIN clauses.

### Interview Questions
1. What is the difference between `fetchone()`, `fetchmany()`, and `fetchall()`?
2. How do transactions work in SQLite's `sqlite3` module?
3. How can you prevent SQL injection when using `sqlite3`?
4. What is WAL mode and why would you use it?
5. How do you handle concurrent access to an SQLite database in Python?
6. Explain the purpose of `row_factory` and give an example.
7. What is a savepoint and when would you use it?

### Coding Challenges
1. **Contact Book**: Build a CLI contact manager that stores, retrieves, updates, and deletes contacts using SQLite.
2. **Transaction Logger**: Create a system that logs financial transactions and can generate account statements with proper rollback on errors.
3. **Cache Layer**: Implement a simple disk-backed key-value cache using SQLite with TTL (time-to-live) support.

### Related Topics
- SQLAlchemy (higher-level ORM abstraction over SQLite)
- Database migrations (Alembic)
- Connection pooling for concurrent applications
- SQL indexing and query optimization
- `pickle` and `json` for serializing complex data into SQLite BLOBs
