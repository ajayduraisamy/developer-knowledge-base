# PostgreSQL - psycopg2.connect(), asyncpg, connection pools

## Introduction

PostgreSQL is a powerful, open-source object-relational database system with over 30 years of active development. It supports advanced SQL features, custom data types, full-text search, JSON/JSONB document storage, spatial data with PostGIS, and concurrent access with MVCC. Python connects to PostgreSQL primarily through `psycopg2` (synchronous) and `asyncpg` (asynchronous).

## Why It Is Important

PostgreSQL is the most advanced open-source relational database, offering ACID compliance, robust transaction support, extensible type system, sophisticated query optimizer, and high concurrency. It is the database of choice for production web applications, data warehousing, geospatial applications, financial systems, and any scenario requiring data integrity, complex queries, and reliability.

## Syntax

```python
import psycopg2

conn = psycopg2.connect(
    host='localhost',
    port=5432,
    dbname='mydb',
    user='postgres',
    password='secret'
)

cursor = conn.cursor()
cursor.execute('SELECT * FROM users WHERE id = %s', (1,))
row = cursor.fetchone()

conn.commit()
conn.close()
```

## Examples

### Connection Setup and Basic Operations with psycopg2

```python
import os
import time
from datetime import datetime, date
from typing import Optional, List, Tuple, Dict, Any, Generator
from contextlib import contextmanager

import psycopg2
from psycopg2 import pool, sql, extras
from psycopg2.extensions import connection, cursor
from psycopg2.errors import UniqueViolation, OperationalError, IntegrityError

DB_CONFIG = {
    "host": os.environ.get("PGHOST", "localhost"),
    "port": int(os.environ.get("PGPORT", "5432")),
    "dbname": os.environ.get("PGDATABASE", "test_db"),
    "user": os.environ.get("PGUSER", "postgres"),
    "password": os.environ.get("PGPASSWORD", "postgres"),
}

connection_pool = pool.SimpleConnectionPool(minconn=2, maxconn=10, **DB_CONFIG)


@contextmanager
def get_conn() -> Generator[connection, None, None]:
    conn = connection_pool.getconn()
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        connection_pool.putconn(conn)


@contextmanager
def get_cursor() -> Generator[cursor, None, None]:
    with get_conn() as conn:
        cur = conn.cursor()
        try:
            yield cur
            conn.commit()
        except Exception:
            conn.rollback()
            raise
        finally:
            cur.close()


def init_database():
    with get_conn() as conn:
        cur = conn.cursor()
        cur.execute('CREATE EXTENSION IF NOT EXISTS "uuid-ossp";')
        cur.execute('CREATE EXTENSION IF NOT EXISTS "pgcrypto";')
        cur.execute("""
            CREATE TABLE IF NOT EXISTS users (
                id SERIAL PRIMARY KEY,
                uuid UUID DEFAULT uuid_generate_v4() UNIQUE,
                username VARCHAR(50) UNIQUE NOT NULL,
                email VARCHAR(255) UNIQUE NOT NULL,
                password_hash VARCHAR(255) NOT NULL,
                full_name VARCHAR(100),
                is_active BOOLEAN DEFAULT TRUE,
                role VARCHAR(20) DEFAULT 'user',
                login_count INTEGER DEFAULT 0,
                last_login TIMESTAMP WITH TIME ZONE,
                metadata JSONB DEFAULT '{}',
                created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
                updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
            );
        """)
        cur.execute("""
            CREATE TABLE IF NOT EXISTS categories (
                id SERIAL PRIMARY KEY,
                name VARCHAR(50) UNIQUE NOT NULL,
                slug VARCHAR(50) UNIQUE NOT NULL,
                description TEXT,
                parent_id INTEGER REFERENCES categories(id) ON DELETE SET NULL,
                created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
            );
        """)
        cur.execute("""
            CREATE TABLE IF NOT EXISTS posts (
                id SERIAL PRIMARY KEY,
                title VARCHAR(255) NOT NULL,
                slug VARCHAR(255) UNIQUE NOT NULL,
                content TEXT NOT NULL,
                excerpt VARCHAR(500),
                is_published BOOLEAN DEFAULT FALSE,
                view_count INTEGER DEFAULT 0,
                rating NUMERIC(3,2) DEFAULT 0.00,
                author_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                category_id INTEGER REFERENCES categories(id) ON DELETE SET NULL,
                published_at TIMESTAMP WITH TIME ZONE,
                created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
                updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
            );
        """)
        cur.execute("""
            CREATE TABLE IF NOT EXISTS tags (
                id SERIAL PRIMARY KEY,
                name VARCHAR(30) UNIQUE NOT NULL,
                slug VARCHAR(30) UNIQUE NOT NULL,
                created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
            );
        """)
        cur.execute("""
            CREATE TABLE IF NOT EXISTS post_tags (
                post_id INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
                tag_id INTEGER NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
                PRIMARY KEY (post_id, tag_id)
            );
        """)
        cur.execute("""
            CREATE TABLE IF NOT EXISTS comments (
                id SERIAL PRIMARY KEY,
                content TEXT NOT NULL,
                author_name VARCHAR(100) NOT NULL,
                author_email VARCHAR(255) NOT NULL,
                is_approved BOOLEAN DEFAULT FALSE,
                post_id INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
                parent_id INTEGER REFERENCES comments(id) ON DELETE CASCADE,
                created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
            );
        """)
        cur.execute("""
            CREATE TABLE IF NOT EXISTS audit_log (
                id BIGSERIAL PRIMARY KEY,
                table_name VARCHAR(50) NOT NULL,
                record_id INTEGER NOT NULL,
                action VARCHAR(20) NOT NULL,
                old_data JSONB,
                new_data JSONB,
                changed_by VARCHAR(100),
                changed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
            );
        """)
        cur.execute("""
            CREATE INDEX IF NOT EXISTS idx_posts_author_id ON posts(author_id);
            CREATE INDEX IF NOT EXISTS idx_posts_category_id ON posts(category_id);
            CREATE INDEX IF NOT EXISTS idx_posts_published_at ON posts(published_at);
            CREATE INDEX IF NOT EXISTS idx_comments_post_id ON comments(post_id);
            CREATE INDEX IF NOT EXISTS idx_audit_log_table_record ON audit_log(table_name, record_id);
        """)
        conn.commit()
        cur.close()


init_database()
```

### CRUD Operations

```python
def create_user(username, email, password_hash, full_name=None, role="user"):
    with get_conn() as conn:
        cur = conn.cursor()
        cur.execute("""
            INSERT INTO users (username, email, password_hash, full_name, role)
            VALUES (%s, %s, %s, %s, %s)
            RETURNING id
        """, (username, email, password_hash, full_name, role))
        user_id = cur.fetchone()[0]
        return user_id


def batch_create_users(users_data):
    with get_conn() as conn:
        cur = conn.cursor()
        extras.execute_values(
            cur,
            "INSERT INTO users (username, email, password_hash, full_name, role) VALUES %s RETURNING id",
            users_data,
            template="(%s, %s, %s, %s, %s)"
        )
        return [row[0] for row in cur.fetchall()]


def get_user_by_id(user_id):
    with get_cursor() as cur:
        cur.execute("SELECT * FROM users WHERE id = %s", (user_id,))
        row = cur.fetchone()
        if row:
            colnames = [desc[0] for desc in cur.description]
            return dict(zip(colnames, row))
        return None


def get_user_by_username(username):
    with get_cursor() as cur:
        cur.execute("SELECT * FROM users WHERE username = %s", (username,))
        row = cur.fetchone()
        if row:
            colnames = [desc[0] for desc in cur.description]
            return dict(zip(colnames, row))
        return None


def get_all_users(active_only=True, limit=20, offset=0):
    with get_cursor() as cur:
        query = "SELECT id, username, email, full_name, role, is_active, last_login, created_at FROM users"
        params = []
        if active_only:
            query += " WHERE is_active = TRUE"
        query += " ORDER BY created_at DESC LIMIT %s OFFSET %s"
        params.extend([limit, offset])
        cur.execute(query, tuple(params))
        rows = cur.fetchall()
        colnames = [desc[0] for desc in cur.description]
        return [dict(zip(colnames, row)) for row in rows]


def search_users(search_term):
    with get_cursor() as cur:
        cur.execute("""
            SELECT id, username, email, full_name, role
            FROM users
            WHERE username ILIKE %s
               OR email ILIKE %s
               OR full_name ILIKE %s
            LIMIT 20
        """, (f"%{search_term}%", f"%{search_term}%", f"%{search_term}%"))
        rows = cur.fetchall()
        colnames = [desc[0] for desc in cur.description]
        return [dict(zip(colnames, row)) for row in rows]


def update_user(user_id, updates):
    if not updates:
        return False
    fields = []
    values = []
    for key, val in updates.items():
        fields.append(sql.Identifier(key))
        values.append(val)
    values.append(user_id)
    query = sql.SQL("UPDATE users SET {fields}, updated_at = NOW() WHERE id = %s").format(
        fields=sql.SQL(", ").join(sql.SQL("{} = %s").format(field) for field in fields)
    )
    with get_conn() as conn:
        cur = conn.cursor()
        cur.execute(query, values)
        return cur.rowcount > 0


def increment_login_count(user_id):
    with get_conn() as conn:
        cur = conn.cursor()
        cur.execute("UPDATE users SET login_count = login_count + 1, last_login = NOW() WHERE id = %s", (user_id,))
        return cur.rowcount > 0


def delete_user(user_id):
    with get_conn() as conn:
        cur = conn.cursor()
        cur.execute("DELETE FROM users WHERE id = %s", (user_id,))
        return cur.rowcount > 0


def create_post(title, content, author_id, category_id=None, tags=None):
    slug = title.lower().replace(" ", "-").replace(":", "")
    with get_conn() as conn:
        cur = conn.cursor()
        cur.execute("""
            INSERT INTO posts (title, slug, content, author_id, category_id)
            VALUES (%s, %s, %s, %s, %s)
            RETURNING id
        """, (title, slug, content, author_id, category_id))
        post_id = cur.fetchone()[0]
        if tags:
            for tag_id in tags:
                cur.execute("INSERT INTO post_tags (post_id, tag_id) VALUES (%s, %s) ON CONFLICT DO NOTHING",
                            (post_id, tag_id))
        conn.commit()
        return post_id


def get_published_posts_with_details(limit=20, offset=0):
    with get_cursor() as cur:
        cur.execute("""
            SELECT p.id, p.title, p.slug, p.excerpt, p.view_count, p.rating,
                   p.published_at, u.username AS author, c.name AS category,
                   COALESCE(
                       (SELECT json_agg(json_build_object('id', t.id, 'name', t.name))
                        FROM post_tags pt JOIN tags t ON t.id = pt.tag_id
                        WHERE pt.post_id = p.id),
                       '[]'::json
                   ) AS tags,
                   (SELECT COUNT(*) FROM comments c2 WHERE c2.post_id = p.id AND c2.is_approved = TRUE) AS comment_count
            FROM posts p
            JOIN users u ON u.id = p.author_id
            LEFT JOIN categories c ON c.id = p.category_id
            WHERE p.is_published = TRUE
            ORDER BY p.published_at DESC
            LIMIT %s OFFSET %s
        """, (limit, offset))
        rows = cur.fetchall()
        colnames = [desc[0] for desc in cur.description]
        return [dict(zip(colnames, row)) for row in rows]


def get_post_by_slug(slug):
    with get_cursor() as cur:
        cur.execute("""
            SELECT p.*, u.username AS author, u.full_name AS author_name, c.name AS category_name
            FROM posts p
            JOIN users u ON u.id = p.author_id
            LEFT JOIN categories c ON c.id = p.category_id
            WHERE p.slug = %s
        """, (slug,))
        row = cur.fetchone()
        if row:
            colnames = [desc[0] for desc in cur.description]
            return dict(zip(colnames, row))
        return None


def increment_post_view(post_id):
    with get_conn() as conn:
        cur = conn.cursor()
        cur.execute("UPDATE posts SET view_count = view_count + 1 WHERE id = %s", (post_id,))


def add_comment(post_id, content, author_name, author_email, parent_id=None):
    with get_conn() as conn:
        cur = conn.cursor()
        cur.execute("""
            INSERT INTO comments (content, author_name, author_email, post_id, parent_id)
            VALUES (%s, %s, %s, %s, %s)
            RETURNING id
        """, (content, author_name, author_email, post_id, parent_id))
        return cur.fetchone()[0]


def get_post_comments(post_id):
    with get_cursor() as cur:
        cur.execute("""
            WITH RECURSIVE comment_tree AS (
                SELECT id, content, author_name, created_at, parent_id, 1 AS level
                FROM comments
                WHERE post_id = %s AND parent_id IS NULL AND is_approved = TRUE
                UNION ALL
                SELECT c.id, c.content, c.author_name, c.created_at, c.parent_id, ct.level + 1
                FROM comments c
                JOIN comment_tree ct ON c.parent_id = ct.id
                WHERE c.is_approved = TRUE
            )
            SELECT * FROM comment_tree ORDER BY level, created_at
        """, (post_id,))
        rows = cur.fetchall()
        colnames = [desc[0] for desc in cur.description]
        return [dict(zip(colnames, row)) for row in rows]
```

### Transactions

```python
def transfer_funds(from_user_id, to_user_id, amount):
    with get_conn() as conn:
        cur = conn.cursor()
        try:
            cur.execute("SELECT balance FROM accounts WHERE user_id = %s FOR UPDATE", (from_user_id,))
            from_balance = cur.fetchone()[0]
            if from_balance < amount:
                conn.rollback()
                return False
            cur.execute("UPDATE accounts SET balance = balance - %s WHERE user_id = %s", (amount, from_user_id))
            cur.execute("UPDATE accounts SET balance = balance + %s WHERE user_id = %s", (amount, to_user_id))
            cur.execute("INSERT INTO transfer_log (from_user, to_user, amount, status) VALUES (%s, %s, %s, 'completed')",
                        (from_user_id, to_user_id, amount))
            conn.commit()
            return True
        except Exception:
            conn.rollback()
            raise


def savepoint_example():
    with get_conn() as conn:
        cur = conn.cursor()
        cur.execute("INSERT INTO users (username, email, password_hash) VALUES (%s, %s, %s)",
                    ("temp_user", "temp@example.com", "hash1"))
        cur.execute("SAVEPOINT before_second_insert")
        try:
            cur.execute("INSERT INTO users (username, email, password_hash) VALUES (%s, %s, %s)",
                        ("temp_user", "duplicate@example.com", "hash2"))
        except UniqueViolation:
            cur.execute("ROLLBACK TO SAVEPOINT before_second_insert")
        cur.execute("INSERT INTO users (username, email, password_hash) VALUES (%s, %s, %s)",
                    ("another_user", "another@example.com", "hash3"))
        conn.commit()


def nested_transaction_demo():
    with get_conn() as conn:
        with conn.cursor() as cur:
            cur.execute("INSERT INTO audit_log (table_name, record_id, action) VALUES (%s, %s, %s)",
                        ("users", 1, "test"))
        savepoint_example()
```

### COPY for Bulk Operations

```python
import io
import csv


def copy_from_csv(conn, table, columns, file_path):
    cur = conn.cursor()
    with open(file_path, "r") as f:
        cur.copy_from(f, table, sep=",", columns=columns, null="NULL")
    conn.commit()
    cur.close()


def copy_from_string(conn, table, columns, data):
    cur = conn.cursor()
    buffer = io.StringIO(data)
    cur.copy_from(buffer, table, sep=",", columns=columns, null="NULL")
    conn.commit()
    cur.close()


def bulk_insert_via_copy(users):
    buffer = io.StringIO()
    writer = csv.writer(buffer)
    for user in users:
        writer.writerow([
            user["username"], user["email"],
            user.get("password_hash", "default"),
            user.get("full_name", ""),
            user.get("role", "user")
        ])
    buffer.seek(0)
    with get_conn() as conn:
        cur = conn.cursor()
        cur.copy_from(buffer, "users", sep=",",
                       columns=("username", "email", "password_hash", "full_name", "role"))
        conn.commit()
        return cur.rowcount


def bulk_insert_via_execute_values(users):
    with get_conn() as conn:
        cur = conn.cursor()
        extras.execute_values(
            cur,
            "INSERT INTO users (username, email, password_hash, full_name, role) VALUES %s",
            users,
            template="(%s, %s, %s, %s, %s)"
        )
        conn.commit()
        return len(users)


def export_to_csv(query, params, output_path):
    with get_cursor() as cur:
        cur.execute(query, params)
        with open(output_path, "w", newline="") as f:
            writer = csv.writer(f)
            writer.writerow([desc[0] for desc in cur.description])
            for row in cur:
                writer.writerow(row)
```

### Async with asyncpg

```python
import asyncio
import asyncpg
from asyncpg import Pool, Connection
from typing import Optional

DATABASE_URL = os.environ.get(
    "DATABASE_URL",
    "postgresql://postgres:postgres@localhost:5432/test_db"
)


class AsyncDatabaseManager:
    def __init__(self, dsn, min_size=2, max_size=10):
        self.dsn = dsn
        self.min_size = min_size
        self.max_size = max_size
        self.pool = None

    async def connect(self):
        self.pool = await asyncpg.create_pool(
            dsn=self.dsn, min_size=self.min_size, max_size=self.max_size, command_timeout=30
        )

    async def close(self):
        if self.pool:
            await self.pool.close()

    async def execute(self, query, *args):
        async with self.pool.acquire() as conn:
            return await conn.execute(query, *args)

    async def fetch(self, query, *args):
        async with self.pool.acquire() as conn:
            return await conn.fetch(query, *args)

    async def fetchrow(self, query, *args):
        async with self.pool.acquire() as conn:
            return await conn.fetchrow(query, *args)

    async def fetchval(self, query, *args):
        async with self.pool.acquire() as conn:
            return await conn.fetchval(query, *args)

    async def transaction(self, queries):
        async with self.pool.acquire() as conn:
            async with conn.transaction():
                for query, args in queries:
                    await conn.execute(query, *args)

    async def copy_from_records(self, table, records, columns):
        async with self.pool.acquire() as conn:
            await conn.copy_records_to_table(table, records=records, columns=columns)


async def async_example():
    mgr = AsyncDatabaseManager(DATABASE_URL)
    await mgr.connect()
    try:
        result = await mgr.execute("""
            CREATE TABLE IF NOT EXISTS async_users (
                id SERIAL PRIMARY KEY,
                name VARCHAR(100),
                email VARCHAR(255) UNIQUE,
                created_at TIMESTAMPTZ DEFAULT NOW()
            )
        """)
        print(f"Table created: {result}")
        result = await mgr.execute(
            "INSERT INTO async_users (name, email) VALUES ($1, $2)",
            "Alice", "alice@example.com"
        )
        print(f"Inserted: {result}")
        users = await mgr.fetch("SELECT * FROM async_users")
        for user in users:
            print(f"User: {user['name']} ({user['email']})")
        batch = [("Bob", "bob@example.com"), ("Charlie", "charlie@example.com")]
        async with mgr.pool.acquire() as conn:
            async with conn.transaction():
                for name, email in batch:
                    await conn.execute(
                        "INSERT INTO async_users (name, email) VALUES ($1, $2)", name, email
                    )
        count = await mgr.fetchval("SELECT COUNT(*) FROM async_users")
        print(f"Total users: {count}")
        await mgr.execute("DROP TABLE async_users")
    finally:
        await mgr.close()


asyncio.run(async_example())
```

### SQLAlchemy with PostgreSQL

```python
from sqlalchemy import create_engine, Column, Integer, String, Text, Boolean, DateTime, ForeignKey, Float
from sqlalchemy.orm import declarative_base, sessionmaker, relationship
from sqlalchemy.dialects.postgresql import UUID, JSONB, ARRAY
from sqlalchemy import text

PG_URL = os.environ.get("PG_URL", "postgresql://postgres:postgres@localhost:5432/sqlalchemy_db")
engine = create_engine(PG_URL, pool_size=10, pool_pre_ping=True, pool_recycle=3600)
Base = declarative_base()
SessionPG = sessionmaker(bind=engine)


class UserPG(Base):
    __tablename__ = "sa_users"
    id = Column(Integer, primary_key=True)
    username = Column(String(50), unique=True, nullable=False)
    email = Column(String(255), unique=True, nullable=False)
    metadata_ = Column("metadata", JSONB, default={})
    tags = Column(ARRAY(String), default=[])
    created_at = Column(DateTime, server_default=text("NOW()"))


Base.metadata.create_all(engine)


def pg_with_sqlalchemy():
    session = SessionPG()
    user = UserPG(username="alice_sa", email="alice_sa@example.com",
                  metadata_={"theme": "dark"}, tags=["python", "postgres"])
    session.add(user)
    session.commit()
    users = session.query(UserPG).filter(UserPG.tags.any("python")).all()
    for u in users:
        print(u.username, u.metadata_)
    session.query(UserPG).filter(UserPG.username == "alice_sa").update({"metadata_": {"theme": "light"}})
    session.commit()
    session.close()
```

## Beginner Examples

```python
import psycopg2

conn = psycopg2.connect(host="localhost", port=5432, dbname="testdb", user="postgres", password="postgres")
cur = conn.cursor()

cur.execute("""
    CREATE TABLE IF NOT EXISTS employees (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100),
        department VARCHAR(50),
        salary NUMERIC(10,2)
    )
""")

cur.execute("INSERT INTO employees (name, department, salary) VALUES (%s, %s, %s)",
            ("Alice", "Engineering", 85000))
cur.execute("INSERT INTO employees (name, department, salary) VALUES (%s, %s, %s)",
            ("Bob", "Marketing", 72000))
conn.commit()

cur.execute("SELECT * FROM employees")
for row in cur.fetchall():
    print(row)

cur.execute("SELECT * FROM employees WHERE salary > %s", (75000,))
high_earners = cur.fetchall()
print("High earners:", high_earners)

cur.execute("UPDATE employees SET salary = %s WHERE name = %s", (90000, "Alice"))
conn.commit()

cur.execute("DELETE FROM employees WHERE name = %s", ("Bob",))
conn.commit()

cur.close()
conn.close()
```

## Intermediate Examples

```python
import psycopg2
from psycopg2 import pool, extras
from contextlib import contextmanager

db_pool = pool.ThreadedConnectionPool(2, 10, host="localhost", dbname="testdb", user="postgres", password="postgres")


@contextmanager
def get_conn():
    conn = db_pool.getconn()
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        db_pool.putconn(conn)


with get_conn() as conn:
    cur = conn.cursor()
    cur.execute("SELECT table_name FROM information_schema.tables WHERE table_schema = 'public'")
    tables = cur.fetchall()
    print("Tables:", tables)

    cur.execute("""
        SELECT department, COUNT(id) as emp_count, AVG(salary) as avg_salary
        FROM employees GROUP BY department HAVING COUNT(id) > 1
    """)
    for row in cur.fetchall():
        print(f"Dept: {row[0]}, Count: {row[1]}, Avg: {row[2]}")

    data = [(f"User{i}", f"Engineering") for i in range(100)]
    extras.execute_values(cur, "INSERT INTO employees (name, department) VALUES %s", data)
    conn.commit()

    cur.execute("EXPLAIN ANALYZE SELECT * FROM employees WHERE salary > 50000")
    for line in cur.fetchall():
        print(line[0])
```

## Advanced Examples

```python
import psycopg2
import psycopg2.pool
import json
import asyncio
import asyncpg
from datetime import datetime
from typing import List, Dict, Optional


class PostgresAdvancedOperations:
    def __init__(self, dsn):
        self.dsn = dsn
        self.pool = psycopg2.pool.ThreadedConnectionPool(2, 20, dsn)

    def jsonb_operations(self):
        with self.pool.getconn() as conn:
            cur = conn.cursor()
            cur.execute("CREATE TABLE IF NOT EXISTS events (id SERIAL PRIMARY KEY, data JSONB)")
            cur.execute("INSERT INTO events (data) VALUES (%s)",
                        (json.dumps({"type": "click", "page": "/home", "user_id": 42}),))
            cur.execute("INSERT INTO events (data) VALUES (%s)",
                        (json.dumps({"type": "view", "page": "/about", "duration": 120}),))
            cur.execute("SELECT data->>'type' as event_type, COUNT(*) FROM events GROUP BY data->>'type'")
            for row in cur.fetchall():
                print(f"Event: {row[0]}, Count: {row[1]}")
            cur.execute("SELECT * FROM events WHERE data @> %s", (json.dumps({"type": "click"}),))
            print(f"Click events: {len(cur.fetchall())}")
            conn.commit()

    def full_text_search(self):
        with self.pool.getconn() as conn:
            cur = conn.cursor()
            cur.execute("CREATE TABLE IF NOT EXISTS documents (id SERIAL PRIMARY KEY, title TEXT, body TEXT)")
            cur.execute("""
                ALTER TABLE documents ADD COLUMN IF NOT EXISTS fts tsvector
                GENERATED ALWAYS AS (to_tsvector('english', title || ' ' || body)) STORED
            """)
            cur.execute("CREATE INDEX IF NOT EXISTS idx_documents_fts ON documents USING GIN(fts)")
            cur.execute("INSERT INTO documents (title, body) VALUES (%s, %s)",
                        ("PostgreSQL Tutorial", "Learn how to use PostgreSQL full text search effectively"))
            cur.execute("""
                SELECT title, ts_rank(fts, plainto_tsquery('english', %s)) as rank
                FROM documents WHERE fts @@ plainto_tsquery('english', %s)
                ORDER BY rank DESC
            """, ("postgresql full text search", "postgresql full text search"))
            for row in cur.fetchall():
                print(f"Rank {row[1]}: {row[0]}")
            conn.commit()

    def window_functions(self):
        with self.pool.getconn() as conn:
            cur = conn.cursor()
            cur.execute("""
                SELECT name, department, salary,
                       RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank,
                       AVG(salary) OVER (PARTITION BY department) as dept_avg,
                       salary - AVG(salary) OVER (PARTITION BY department) as diff_from_avg
                FROM employees
                ORDER BY department, salary DESC
            """)
            for row in cur.fetchall():
                print(f"{row[0]} ({row[1]}): ${row[2]}, Rank: {row[3]}, Diff: ${row[5]:.2f}")

    def upsert_and_merge(self):
        with self.pool.getconn() as conn:
            cur = conn.cursor()
            cur.execute("""
                CREATE TABLE IF NOT EXISTS product_inventory (
                    sku TEXT PRIMARY KEY,
                    name TEXT NOT NULL,
                    quantity INTEGER DEFAULT 0,
                    price NUMERIC(10,2),
                    updated_at TIMESTAMPTZ DEFAULT NOW()
                )
            """)
            cur.execute("""
                INSERT INTO product_inventory (sku, name, quantity, price)
                VALUES (%s, %s, %s, %s)
                ON CONFLICT (sku) DO UPDATE SET
                    quantity = EXCLUDED.quantity,
                    price = EXCLUDED.price,
                    updated_at = NOW()
            """, ("LAP001", "Laptop", 50, 999.99))
            cur.execute("""
                INSERT INTO product_inventory (sku, name, quantity, price)
                VALUES (%s, %s, %s, %s)
                ON CONFLICT (sku) DO NOTHING
            """, ("LAP001", "Laptop", 100, 899.99))
            conn.commit()

    def common_table_expressions(self):
        with self.pool.getconn() as conn:
            cur = conn.cursor()
            cur.execute("""
                WITH ranked_employees AS (
                    SELECT name, department, salary,
                           DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank
                    FROM employees
                ),
                department_stats AS (
                    SELECT department,
                           AVG(salary) as avg_salary,
                           PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) as median_salary
                    FROM employees
                    GROUP BY department
                )
                SELECT re.name, re.department, re.salary, re.rank,
                       ds.avg_salary, ds.median_salary
                FROM ranked_employees re
                JOIN department_stats ds ON re.department = ds.department
                WHERE re.rank <= 3
                ORDER BY re.department, re.rank
            """)
            for row in cur.fetchall():
                print(f"{row[0]} ({row[1]}): ${row[2]}, Rank {row[3]}")

    def listen_notify(self):
        def listener():
            import select
            conn = psycopg2.connect(self.dsn)
            conn.set_isolation_level(psycopg2.extensions.ISOLATION_LEVEL_AUTOCOMMIT)
            cur = conn.cursor()
            cur.execute("LISTEN channel_events")
            while True:
                if select.select([conn], [], [], 5) == ([], [], []):
                    continue
                conn.poll()
                while conn.notifies:
                    notify = conn.notifies.pop(0)
                    print(f"NOTIFY: {notify.channel}: {notify.payload}")

        import threading
        t = threading.Thread(target=listener, daemon=True)
        t.start()
        conn2 = psycopg2.connect(self.dsn)
        cur2 = conn2.cursor()
        cur2.execute("NOTIFY channel_events, '{"message": "hello"}'")
        conn2.commit()
        conn2.close()


async def async_advanced():
    conn = await asyncpg.connect(DATABASE_URL)
    async with conn.transaction():
        await conn.execute("CREATE TABLE IF NOT EXISTS async_items (id SERIAL PRIMARY KEY, name TEXT, value REAL)")
        await conn.copy_records_to_table(
            "async_items",
            records=[(f"item_{i}", i * 1.5) for i in range(1000)],
            columns=["name", "value"]
        )
    result = await conn.fetch("SELECT COUNT(*), AVG(value) FROM async_items")
    print(f"Count: {result[0][0]}, Avg: {result[0][1]}")
    await conn.close()
    print("Async advanced example completed")


def advanced_main():
    pg = PostgresAdvancedOperations("dbname=test_db user=postgres password=postgres host=localhost")
    pg.jsonb_operations()
    pg.full_text_search()
    pg.window_functions()
    pg.upsert_and_merge()
    pg.common_table_expressions()
    print("All advanced examples completed")
```

## Real-World Use Cases

PostgreSQL powers production databases for web applications (Django, Rails, FastAPI), geospatial applications (PostGIS), data warehousing, financial systems requiring ACID compliance, full-text search engines, JSON document stores (hybrid relational/document), time-series data with partitioning, and large-scale analytics with window functions and CTEs.

## Common Mistakes

- Not using connection pooling for production applications.
- Forgetting to commit transactions or handle rollbacks.
- Using string formatting instead of `%s` parameterized queries (SQL injection risk).
- Not setting `pool_pre_ping` or `pool_recycle` with SQLAlchemy.
- Ignoring PostgreSQL-specific types (JSONB, ARRAY, TSVECTOR, UUID) when they would simplify the schema.
- Not using `EXPLAIN ANALYZE` to profile slow queries.
- Over-fetching columns (SELECT *) in production queries.

## Best Practices

- Use connection pooling (`psycopg2.pool` or SQLAlchemy's pool) for production.
- Always use parameterized queries `%s` to prevent SQL injection.
- Use `COPY` for bulk data loading (much faster than individual INSERTs).
- Use `RETURNING` clause to get inserted/updated IDs.
- Use JSONB for semi-structured data and `@>` operator for queries.
- Use `EXPLAIN ANALYZE` to understand and optimize query plans.
- Enable `pool_pre_ping` and `pool_recycle` with SQLAlchemy.
- Use asyncpg for asyncio-based applications.
- Use transactions with appropriate isolation levels for consistency.

## Interview Questions

1. What is the difference between PostgreSQL and MySQL? — PostgreSQL is more feature-rich (JSONB, custom types, advanced indexing, full-text search, CTEs, window functions, partial indexes, GiST/GIN indexes), more SQL standards compliant, and has better concurrency control with MVCC.
2. Explain MVCC in PostgreSQL. — Multi-Version Concurrency Control allows readers to see a consistent snapshot without blocking writers; each transaction sees a snapshot of the database at a point in time.
3. What is the difference between `VACUUM` and `ANALYZE`? — `VACUUM` reclaims storage from dead tuples; `ANALYZE` updates statistics for the query planner.
4. How does PostgreSQL handle full-text search? — Uses `tsvector` (lexeme storage) and `tsquery` (query representation) with `@@` operator, GIN indexes, and ranking functions (`ts_rank`).
5. What are window functions in PostgreSQL? — Functions that perform calculations across a set of rows related to the current row without collapsing them; supports `RANK`, `ROW_NUMBER`, `LAG`, `LEAD`, `SUM` with `OVER (PARTITION BY ... ORDER BY ...)`.

## Coding Challenges

1. **Blog Engine** — Build a full blog database schema with users, posts, categories, tags, and comments. Implement full-text search, recursive comments, and pagination with window functions.
2. **E-Commerce Platform** — Create schemas for products, inventory, orders, and customers. Implement JSONB for product attributes, COPY for bulk import, and transaction-safe order processing.
3. **Analytics Dashboard** — Build a time-series analytics system using partitioning by date, window functions for running totals, and materialized views for pre-computed aggregates.
4. **Multi-Tenant SaaS** — Design a multi-tenant database using schema-per-tenant or row-level security (RLS). Implement connection pooling for efficient resource usage.
5. **Geospatial Application** — Use PostGIS extension to build a location-based service with spatial queries, distance calculations, and geospatial indexing.

## Summary

PostgreSQL is the most advanced open-source relational database system, offering ACID compliance, extensible types, full-text search, JSONB support, and sophisticated querying capabilities. Python connects via `psycopg2` (synchronous) and `asyncpg` (asynchronous). Key features include connection pooling, parameterized queries, COPY for bulk operations, transactions with savepoints, window functions, CTEs, and integration with SQLAlchemy for ORM-based access.

## Related Topics

- psycopg2 (synchronous PostgreSQL adapter)
- asyncpg (asynchronous PostgreSQL driver)
- SQLAlchemy (ORM with PostgreSQL support)
- Alembic (database migrations)
- PostGIS (geospatial extension for PostgreSQL)
- Redis (caching layer often used with PostgreSQL)
- Celery (task queue with PostgreSQL as broker/backend)
