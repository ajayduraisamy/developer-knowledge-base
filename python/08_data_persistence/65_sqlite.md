# SQLite - sqlite3.connect(), execute(), fetchall(), transactions

## Introduction

SQLite is a self-contained, serverless, zero-configuration, transactional SQL database engine. Python's `sqlite3` module provides a lightweight disk-based database that doesn't require a separate server process. It's embedded directly into Python and allows interacting with SQLite databases using SQL queries.

## Why It Is Important

SQLite is ideal for embedded applications, prototyping, testing, small-to-medium websites, data analysis, and any scenario where a full database server is overkill. It stores the entire database as a single cross-platform file, making it extremely portable. Python's `sqlite3` module is part of the standard library, so no additional dependencies are needed.

## Syntax

```python
import sqlite3

# Connect to database (creates file if not exists)
conn = sqlite3.connect('database.db')

# Create a cursor
cursor = conn.cursor()

# Execute a SQL statement
cursor.execute("SELECT * FROM table_name")

# Fetch results
rows = cursor.fetchall()

# Commit changes
conn.commit()

# Close connection
conn.close()
```

## Examples

### Basic Database Operations

```python
import sqlite3
import os
from datetime import date
from typing import List, Tuple, Optional, Any

DB_PATH = "example.db"


def clean_up():
    if os.path.exists(DB_PATH):
        os.remove(DB_PATH)


def create_connection(db_path: str) -> sqlite3.Connection:
    conn = sqlite3.connect(db_path)
    conn.execute("PRAGMA journal_mode=WAL;")
    conn.execute("PRAGMA foreign_keys=ON;")
    return conn


def create_tables(conn: sqlite3.Connection):
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            username TEXT NOT NULL UNIQUE,
            email TEXT NOT NULL UNIQUE,
            created_at TEXT NOT NULL DEFAULT (datetime('now'))
        )
    """)
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS posts (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id INTEGER NOT NULL,
            title TEXT NOT NULL,
            body TEXT,
            published INTEGER NOT NULL DEFAULT 0,
            created_at TEXT NOT NULL DEFAULT (datetime('now')),
            FOREIGN KEY (user_id) REFERENCES users(id)
        )
    """)
    conn.commit()


def insert_user(conn: sqlite3.Connection, username: str, email: str) -> int:
    cursor = conn.cursor()
    cursor.execute(
        "INSERT INTO users (username, email) VALUES (?, ?)",
        (username, email)
    )
    conn.commit()
    return cursor.lastrowid


def insert_users_batch(conn: sqlite3.Connection, users: List[Tuple[str, str]]):
    cursor = conn.cursor()
    cursor.executemany(
        "INSERT INTO users (username, email) VALUES (?, ?)",
        users
    )
    conn.commit()


def insert_post(conn: sqlite3.Connection, user_id: int, title: str, body: str) -> int:
    cursor = conn.cursor()
    cursor.execute(
        "INSERT INTO posts (user_id, title, body) VALUES (?, ?, ?)",
        (user_id, title, body)
    )
    conn.commit()
    return cursor.lastrowid


def get_user_by_id(conn: sqlite3.Connection, user_id: int) -> Optional[Tuple]:
    cursor = conn.cursor()
    cursor.execute("SELECT id, username, email, created_at FROM users WHERE id = ?", (user_id,))
    return cursor.fetchone()


def get_user_by_username(conn: sqlite3.Connection, username: str) -> Optional[Tuple]:
    cursor = conn.cursor()
    cursor.execute("SELECT id, username, email, created_at FROM users WHERE username = ?", (username,))
    return cursor.fetchone()


def get_all_users(conn: sqlite3.Connection) -> List[Tuple]:
    cursor = conn.cursor()
    cursor.execute("SELECT id, username, email, created_at FROM users ORDER BY created_at DESC")
    return cursor.fetchall()


def get_user_posts(conn: sqlite3.Connection, user_id: int) -> List[Tuple]:
    cursor = conn.cursor()
    cursor.execute(
        "SELECT id, title, body, published, created_at FROM posts WHERE user_id = ? ORDER BY created_at DESC",
        (user_id,)
    )
    return cursor.fetchall()


def update_user_email(conn: sqlite3.Connection, user_id: int, new_email: str):
    cursor = conn.cursor()
    cursor.execute(
        "UPDATE users SET email = ? WHERE id = ?",
        (new_email, user_id)
    )
    conn.commit()
    return cursor.rowcount


def publish_post(conn: sqlite3.Connection, post_id: int):
    cursor = conn.cursor()
    cursor.execute(
        "UPDATE posts SET published = 1 WHERE id = ?",
        (post_id,)
    )
    conn.commit()
    return cursor.rowcount


def delete_user(conn: sqlite3.Connection, user_id: int) -> int:
    cursor = conn.cursor()
    cursor.execute("DELETE FROM posts WHERE user_id = ?", (user_id,))
    cursor.execute("DELETE FROM users WHERE id = ?", (user_id,))
    conn.commit()
    return cursor.rowcount


def search_posts(conn: sqlite3.Connection, search_term: str) -> List[Tuple]:
    cursor = conn.cursor()
    cursor.execute(
        "SELECT p.id, p.title, u.username FROM posts p JOIN users u ON p.user_id = u.id WHERE p.title LIKE ?",
        (f"%{search_term}%",)
    )
    return cursor.fetchall()


def get_published_posts_with_authors(conn: sqlite3.Connection) -> List[Tuple]:
    cursor = conn.cursor()
    cursor.execute("""
        SELECT p.id, p.title, u.username, p.created_at
        FROM posts p
        JOIN users u ON p.user_id = u.id
        WHERE p.published = 1
        ORDER BY p.created_at DESC
    """)
    return cursor.fetchall()


def get_post_count_per_user(conn: sqlite3.Connection) -> List[Tuple]:
    cursor = conn.cursor()
    cursor.execute("""
        SELECT u.username, COUNT(p.id) as post_count
        FROM users u
        LEFT JOIN posts p ON u.id = p.user_id
        GROUP BY u.id
        ORDER BY post_count DESC
    """)
    return cursor.fetchall()


def main():
    clean_up()
    conn = create_connection(DB_PATH)
    create_tables(conn)

    insert_user(conn, "alice", "alice@example.com")
    insert_user(conn, "bob", "bob@example.com")
    uid3 = insert_user(conn, "charlie", "charlie@example.com")

    insert_users_batch(conn, [
        ("dave", "dave@example.com"),
        ("eve", "eve@example.com"),
        ("frank", "frank@example.com"),
    ])

    insert_post(conn, 1, "Hello World", "My first post!")
    insert_post(conn, 1, "Python Tips", "Some useful Python tips.")
    insert_post(conn, 2, "SQLite Guide", "How to use SQLite with Python.")
    insert_post(conn, 3, "Data Science", "Introduction to data science.")

    publish_post(conn, 1)
    publish_post(conn, 3)

    user = get_user_by_id(conn, 1)
    print("User 1:", user)

    bob = get_user_by_username(conn, "bob")
    print("Bob:", bob)

    all_users = get_all_users(conn)
    print("All users:", all_users)

    alice_posts = get_user_posts(conn, 1)
    print("Alice's posts:", alice_posts)

    update_user_email(conn, 2, "bob_new@example.com")
    print("Updated Bob's email")

    search_results = search_posts(conn, "Python")
    print("Search results:", search_results)

    published = get_published_posts_with_authors(conn)
    print("Published posts:", published)

    counts = get_post_count_per_user(conn)
    print("Post counts:", counts)

    delete_user(conn, uid3)
    print("Deleted Charlie")

    conn.close()
    clean_up()


if __name__ == "__main__":
    main()
```

## Beginner Examples

```python
import sqlite3

conn = sqlite3.connect("school.db")
cursor = conn.cursor()

cursor.execute("""
    CREATE TABLE IF NOT EXISTS students (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        grade INTEGER
    )
""")

cursor.execute("INSERT INTO students (name, grade) VALUES (?, ?)", ("Alice", 85))
cursor.execute("INSERT INTO students (name, grade) VALUES (?, ?)", ("Bob", 92))
cursor.execute("INSERT INTO students (name, grade) VALUES (?, ?)", ("Charlie", 78))
conn.commit()

cursor.execute("SELECT * FROM students")
rows = cursor.fetchall()
for row in rows:
    print(row)

cursor.execute("SELECT * FROM students WHERE grade >= 80")
above_80 = cursor.fetchall()
print("Students with grade >= 80:", above_80)

cursor.execute("UPDATE students SET grade = ? WHERE name = ?", (95, "Alice"))
conn.commit()

cursor.execute("DELETE FROM students WHERE name = ?", ("Charlie",))
conn.commit()

cursor.execute("SELECT name, grade FROM students ORDER BY grade DESC")
ranked = cursor.fetchall()
print("Rankings:", ranked)

conn.close()
```

## Intermediate Examples

```python
import sqlite3
from contextlib import contextmanager
from typing import Generator, Any


@contextmanager
def get_db_connection(db_path: str) -> Generator[sqlite3.Connection, None, None]:
    conn = sqlite3.connect(db_path)
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        conn.close()


def dict_factory(cursor: sqlite3.Cursor, row: tuple) -> dict:
    columns = [col[0] for col in cursor.description]
    return dict(zip(columns, row))


with get_db_connection(":memory:") as conn:
    conn.row_factory = dict_factory
    conn.execute("CREATE TABLE inventory (id INTEGER PRIMARY KEY, item TEXT, qty INTEGER)")

    items = [("Apple", 50), ("Banana", 30), ("Cherry", 20)]
    conn.executemany("INSERT INTO inventory (item, qty) VALUES (?, ?)", items)

    cursor = conn.execute("SELECT * FROM inventory WHERE qty > 25")
    for row in cursor.fetchall():
        print(row["item"], row["qty"])

    with conn:
        conn.execute("UPDATE inventory SET qty = qty - 10 WHERE item = ?", ("Apple",))
        conn.execute("UPDATE inventory SET qty = qty + 15 WHERE item = ?", ("Banana",))

    cursor = conn.execute("SELECT SUM(qty) as total FROM inventory")
    print("Total qty:", cursor.fetchone()["total"])
```

## Advanced Examples

```python
import sqlite3
import json
import threading
import queue
from datetime import datetime
from pathlib import Path


class ThreadSafeDatabasePool:
    def __init__(self, db_path: str, max_connections: int = 5):
        self.db_path = db_path
        self._pool = queue.Queue(maxsize=max_connections)
        self._local = threading.local()
        for _ in range(max_connections):
            conn = sqlite3.connect(db_path, check_same_thread=False)
            conn.execute("PRAGMA journal_mode=WAL;")
            conn.execute("PRAGMA foreign_keys=ON;")
            conn.row_factory = sqlite3.Row
            self._pool.put(conn)

    def get_connection(self) -> sqlite3.Connection:
        if hasattr(self._local, "conn") and self._local.conn is not None:
            return self._local.conn
        conn = self._pool.get()
        self._local.conn = conn
        return conn

    def return_connection(self, conn: sqlite3.Connection):
        if hasattr(self._local, "conn") and self._local.conn is conn:
            self._local.conn = None
            self._pool.put(conn)

    def close_all(self):
        while not self._pool.empty():
            conn = self._pool.get()
            conn.close()


class VersionedDatabase:
    def __init__(self, db_path: str):
        self.db_path = db_path
        self.conn = sqlite3.connect(db_path)
        self.conn.row_factory = sqlite3.Row
        self._init_schema()

    def _init_schema(self):
        self.conn.executescript("""
            CREATE TABLE IF NOT EXISTS schema_version (
                version INTEGER PRIMARY KEY,
                applied_at TEXT NOT NULL
            );
            CREATE TABLE IF NOT EXISTS documents (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                doc_id TEXT NOT NULL UNIQUE,
                title TEXT NOT NULL,
                content TEXT,
                metadata TEXT DEFAULT '{}',
                current_version INTEGER NOT NULL DEFAULT 1,
                created_at TEXT NOT NULL DEFAULT (datetime('now')),
                updated_at TEXT NOT NULL DEFAULT (datetime('now'))
            );
            CREATE TABLE IF NOT EXISTS document_versions (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                doc_id TEXT NOT NULL,
                version INTEGER NOT NULL,
                title TEXT NOT NULL,
                content TEXT,
                metadata TEXT DEFAULT '{}',
                created_at TEXT NOT NULL DEFAULT (datetime('now')),
                UNIQUE(doc_id, version)
            );
        """)
        self.conn.commit()

    def create_document(self, doc_id: str, title: str, content: str = "", metadata: dict = None) -> int:
        meta_json = json.dumps(metadata or {})
        cursor = self.conn.execute(
            "INSERT INTO documents (doc_id, title, content, metadata) VALUES (?, ?, ?, ?)",
            (doc_id, title, content, meta_json)
        )
        doc_pk = cursor.lastrowid
        self.conn.execute(
            "INSERT INTO document_versions (doc_id, version, title, content, metadata) VALUES (?, 1, ?, ?, ?)",
            (doc_id, title, content, meta_json)
        )
        self.conn.commit()
        return doc_pk

    def update_document(self, doc_id: str, title: str = None, content: str = None, metadata: dict = None) -> bool:
        cursor = self.conn.execute("SELECT current_version, title, content, metadata FROM documents WHERE doc_id = ?", (doc_id,))
        row = cursor.fetchone()
        if not row:
            return False
        new_version = row["current_version"] + 1
        new_title = title if title is not None else row["title"]
        new_content = content if content is not None else row["content"]
        new_meta = json.dumps(metadata) if metadata is not None else row["metadata"]
        self.conn.execute(
            "UPDATE documents SET title=?, content=?, metadata=?, current_version=?, updated_at=datetime('now') WHERE doc_id=?",
            (new_title, new_content, new_meta, new_version, doc_id)
        )
        self.conn.execute(
            "INSERT INTO document_versions (doc_id, version, title, content, metadata) VALUES (?, ?, ?, ?, ?)",
            (doc_id, new_version, new_title, new_content, new_meta)
        )
        self.conn.commit()
        return True

    def get_document(self, doc_id: str) -> dict:
        cursor = self.conn.execute(
            "SELECT doc_id, title, content, metadata, current_version, created_at, updated_at FROM documents WHERE doc_id = ?",
            (doc_id,)
        )
        row = cursor.fetchone()
        if not row:
            return None
        result = dict(row)
        result["metadata"] = json.loads(result["metadata"])
        return result

    def get_document_version(self, doc_id: str, version: int) -> dict:
        cursor = self.conn.execute(
            "SELECT doc_id, version, title, content, metadata, created_at FROM document_versions WHERE doc_id = ? AND version = ?",
            (doc_id, version)
        )
        row = cursor.fetchone()
        if not row:
            return None
        result = dict(row)
        result["metadata"] = json.loads(result["metadata"])
        return result

    def get_version_history(self, doc_id: str) -> list:
        cursor = self.conn.execute(
            "SELECT version, title, created_at FROM document_versions WHERE doc_id = ? ORDER BY version DESC",
            (doc_id,)
        )
        return [dict(row) for row in cursor.fetchall()]

    def search_documents(self, query: str) -> list:
        cursor = self.conn.execute(
            "SELECT doc_id, title, updated_at FROM documents WHERE title LIKE ? OR content LIKE ? ORDER BY updated_at DESC",
            (f"%{query}%", f"%{query}%")
        )
        return [dict(row) for row in cursor.fetchall()]

    def close(self):
        self.conn.close()


class FullTextSearchIndex:
    def __init__(self, db_path: str):
        self.conn = sqlite3.connect(db_path)
        self.conn.execute("CREATE VIRTUAL TABLE IF NOT EXISTS fts_articles USING fts5(title, body, author)")
        self.conn.commit()

    def add_article(self, title: str, body: str, author: str):
        self.conn.execute("INSERT INTO fts_articles (title, body, author) VALUES (?, ?, ?)", (title, body, author))
        self.conn.commit()

    def search(self, query: str) -> list:
        cursor = self.conn.execute(
            "SELECT rank, title, author FROM fts_articles WHERE fts_articles MATCH ? ORDER BY rank",
            (query,)
        )
        return [dict(row) for row in cursor.fetchall()]

    def close(self):
        self.conn.close()


def main():
    vdb = VersionedDatabase("versions.db")
    vdb.create_document("doc1", "My Document", "Initial content", {"author": "Alice", "tags": ["draft"]})
    vdb.update_document("doc1", content="Updated content with more details", metadata={"author": "Alice", "tags": ["final"], "reviewed": True})
    vdb.update_document("doc1", title="My Updated Document")
    doc = vdb.get_document("doc1")
    print("Current:", json.dumps(doc, indent=2, default=str))
    history = vdb.get_version_history("doc1")
    print("History:", json.dumps(history, indent=2, default=str))
    v1 = vdb.get_document_version("doc1", 1)
    print("Version 1:", json.dumps(v1, indent=2, default=str))
    results = vdb.search_documents("Updated")
    print("Search results:", results)
    vdb.close()

    Path("versions.db").unlink(missing_ok=True)


if __name__ == "__main__":
    main()
```

## Real-World Use Cases

SQLite is used in mobile applications (Android/iOS), desktop software, embedded systems, IoT devices, browser local storage (IndexedDB alternative), data science pipelines, caching layers, configuration storage, and prototyping database-backed applications before migrating to production-grade databases.

## Common Mistakes

- Not committing changes after INSERT/UPDATE/DELETE operations.
- Using string formatting (`f"SELECT * FROM users WHERE name = '{name}'"`) instead of parameterized queries (`?` placeholders), leading to SQL injection vulnerabilities.
- Forgetting to close connections, causing resource leaks.
- Not enabling WAL mode for concurrent reads.
- Using the same connection across threads without `check_same_thread=False`.

## Best Practices

- Always use parameterized queries with `?` placeholders to prevent SQL injection.
- Use `with conn:` (context manager) for automatic transaction management.
- Enable WAL mode (`PRAGMA journal_mode=WAL;`) for better concurrent read performance.
- Use `row_factory = sqlite3.Row` or a custom dict factory for readable results.
- Close connections using context managers or try/finally blocks.
- Use `executemany` for batch inserts instead of individual INSERT statements.
- Set `PRAGMA foreign_keys=ON;` to enforce foreign key constraints.

## Interview Questions

1. What is SQLite and when should you use it? — A self-contained, serverless, zero-configuration SQL database engine ideal for embedded use, prototyping, and small applications.
2. How do parameterized queries prevent SQL injection? — They separate SQL logic from data by sending query structure and values separately, preventing malicious input from altering the query structure.
3. What is the difference between `execute` and `executemany`? — `execute` runs a single SQL statement; `executemany` runs the same statement with multiple parameter sets efficiently.
4. Explain SQLite transactions and how to use them in Python. — Transactions group multiple operations into an atomic unit; use `conn.commit()` to save or `conn.rollback()` to revert; the connection context manager commits on success and rolls back on exception.
5. What is `row_factory` and how do you use it? — A connection attribute that controls how rows are returned; `sqlite3.Row` returns rows accessible by column name; custom factories can return dictionaries or custom objects.

## Coding Challenges

1. **Task Manager** — Build a SQLite-backed task manager with tables for tasks, categories, and tags. Implement CRUD operations, filtering by status/priority, and full-text search.
2. **Inventory System** — Create an inventory management system with products, stock levels, and suppliers. Use transactions for atomic stock updates.
3. **Blog Engine** — Build a simple blog engine with users, posts, comments, and likes. Implement pagination, search, and version history for posts.
4. **Analytics Dashboard** — Store page views, user sessions, and events in SQLite. Write aggregation queries to generate daily/weekly/monthly reports.
5. **Contact Book** — Create a contact management system with groups, phone numbers, and email addresses. Implement import/export CSV functionality using SQLite.

## Summary

Python's `sqlite3` module provides a full-featured, serverless SQL database engine right in the standard library. Key concepts include creating connections, using cursors for execution, parameterized queries for security, transactions for atomicity, and row factories for result formatting. SQLite is perfect for embedded applications, prototyping, and any scenario where simplicity and portability matter.

## Related Topics

- SQLAlchemy (higher-level ORM abstraction over SQLite)
- PostgreSQL / MySQL (production-grade client-server databases)
- Redis (in-memory key-value store)
- Pandas (data analysis with SQLite backend)
- Alembic (database migration tool)
