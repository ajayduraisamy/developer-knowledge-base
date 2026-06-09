# PostgreSQL - psycopg2.connect(), asyncpg, connection pools

## Introduction
PostgreSQL is a powerful, open-source object-relational database system with a strong reputation for reliability, feature robustness, and performance. It supports advanced SQL features including window functions, common table expressions (CTEs), partial indexes, table inheritance, and custom data types. Python developers connect to PostgreSQL primarily through `psycopg2` (the most mature synchronous driver) and `asyncpg` (a high-performance async/await driver). Both libraries implement the PostgreSQL wire protocol, providing access to PostgreSQL's full feature set including prepared statements, COPY operations for bulk data transfer, LISTEN/NOTIFY for asynchronous notifications, and connection pooling for scalable applications.

## psycopg2.connect()

### What It Is
`psycopg2.connect()` creates a connection to a PostgreSQL database. psycopg2 is the de facto standard PostgreSQL adapter for Python, fully implementing DB-API 2.0 and adding PostgreSQL-specific extensions like server-side cursors, COPY support, and asynchronous notifications.

### Why It Is Important
psycopg2 is battle-tested, extensively documented, and supports all PostgreSQL features including native UDTs, array types, hstore, JSONB, and SSL connections. It is the recommended driver for synchronous PostgreSQL access in production Python applications.

### How It Works Internally
When `psycopg2.connect()` is called, the library opens a TCP connection to the PostgreSQL server, performs SSL negotiation (if configured), and authenticates using the specified method (password, md5, SCRAM-SHA-256, or certificate). The connection object maintains the session state (transaction status, prepared statements, notifications). The underlying C extension (`_psycopg`) handles the PostgreSQL wire protocol efficiently, minimizing Python object overhead for data transfer.

### Syntax
```python
import psycopg2

# Basic connection
conn = psycopg2.connect(
    host="localhost",
    port=5432,
    dbname="mydb",
    user="postgres",
    password="secret"
)

# Connection string
conn = psycopg2.connect("postgresql://postgres:secret@localhost:5432/mydb")

# With connection parameters
conn = psycopg2.connect(
    "postgresql://user:pass@localhost:5432/mydb",
    connect_timeout=10,
    sslmode="require",
    application_name="myapp"
)
```

### Beginner Examples
```python
import psycopg2

conn = psycopg2.connect("postgresql://postgres:postgres@localhost:5432/testdb")
cursor = conn.cursor()

# Create table
cursor.execute("""
    CREATE TABLE IF NOT EXISTS employees (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100) NOT NULL,
        salary NUMERIC(10, 2),
        hire_date DATE DEFAULT CURRENT_DATE
    )
""")

# Insert
cursor.execute(
    "INSERT INTO employees (name, salary) VALUES (%s, %s)",
    ("Alice", 75000.00)
)

# Insert multiple
employees = [("Bob", 65000), ("Charlie", 80000)]
cursor.executemany(
    "INSERT INTO employees (name, salary) VALUES (%s, %s)",
    employees
)

conn.commit()

# Query
cursor.execute("SELECT id, name, salary FROM employees ORDER BY salary DESC")
for row in cursor.fetchall():
    print(f"{row[1]}: ${row[2]:.2f}")

cursor.close()
conn.close()
```

### Intermediate Examples
```python
import psycopg2
from psycopg2.extras import DictCursor, RealDictCursor, NamedTupleCursor

conn = psycopg2.connect("postgresql://postgres:postgres@localhost:5432/testdb")

# Using DictCursor for named field access
with conn.cursor(cursor_factory=DictCursor) as cur:
    cur.execute("SELECT * FROM employees WHERE salary > %s", (50000,))
    for row in cur.fetchall():
        print(row["name"], row["salary"])

# Context manager for automatic commit/rollback
with conn:
    with conn.cursor() as cur:
        cur.execute("UPDATE employees SET salary = salary * 1.1 WHERE id = %s", (1,))

# Error handling and rollback
try:
    with conn:
        with conn.cursor() as cur:
            cur.execute("UPDATE employees SET salary = %s WHERE id = %s", (99999, 1))
            # Simulate error
            raise ValueError("Something went wrong")
except ValueError:
    print("Transaction rolled back")

# Server-side cursors for large result sets
with conn.cursor(name="server_side_cursor") as cur:
    cur.execute("SELECT * FROM large_table")
    for row in cur:
        process(row)

conn.close()
```

### Advanced Examples
```python
import psycopg2
from psycopg2 import sql
from psycopg2.extras import execute_values, execute_batch

conn = psycopg2.connect("postgresql://postgres:postgres@localhost:5432/testdb")

# Dynamic SQL with identifier escaping
table_name = "employees"
columns = ["name", "salary"]
values = [("Dave", 70000), ("Eve", 90000)]

query = sql.SQL("INSERT INTO {} ({}) VALUES ({})").format(
    sql.Identifier(table_name),
    sql.SQL(", ").join(map(sql.Identifier, columns)),
    sql.SQL(", ").join(sql.Placeholder() * len(values[0]))
)

with conn:
    with conn.cursor() as cur:
        for val in values:
            cur.execute(query, val)

# Bulk insert with execute_values (fastest)
data = [("User1", 100), ("User2", 200), ("User3", 300)]
with conn:
    with conn.cursor() as cur:
        execute_values(
            cur,
            "INSERT INTO employees (name, salary) VALUES %s",
            data,
            template="(%s, %s)"
        )

# COPY for massive data transfer
import io

csv_data = io.StringIO()
csv_data.write("John,55000\nJane,62000\nJim,48000\n")
csv_data.seek(0)

with conn:
    with conn.cursor() as cur:
        cur.copy_from(csv_data, "employees", columns=("name", "salary"), sep=",")

# LISTEN/NOTIFY
def wait_for_notification():
    conn = psycopg2.connect("postgresql://postgres:postgres@localhost:5432/testdb")
    conn.set_isolation_level(psycopg2.extensions.ISOLATION_LEVEL_AUTOCOMMIT)
    with conn.cursor() as cur:
        cur.execute("LISTEN channel_events")

    import select
    while True:
        if select.select([conn], [], [], 5) == ([], [], []):
            print("Waiting...")
        else:
            conn.poll()
            for notify in conn.notifies:
                print(f"Notification: {notify.channel} - {notify.payload}")
            conn.notifies.clear()
```

## asyncpg

### What It Is
asyncpg is a high-performance PostgreSQL driver for Python's asyncio framework. It is designed specifically for async/await usage, providing non-blocking database access. asyncpg is significantly faster than psycopg2 in benchmarks due to its pure implementation of the PostgreSQL binary protocol and zero-copy data transfer.

### Why It Is Important
For modern async web frameworks (FastAPI, aiohttp, Sanic) and high-concurrency applications, asyncpg is the optimal choice. It eliminates thread pool overhead, supports prepared statement caching, and handles thousands of concurrent connections efficiently. asyncpg also provides connection pooling built-in.

### How It Works Internally
asyncpg implements the PostgreSQL protocol at the protocol level, including the binary data format which avoids text serialization overhead. It uses Python's asyncio event loop for non-blocking I/O, maintains an internal cache of prepared statements, and uses memoryviews and zero-copy techniques to minimize data copying between C and Python.

### Syntax
```python
import asyncpg
import asyncio

async def main():
    # Connect
    conn = await asyncpg.connect(
        user="postgres",
        password="postgres",
        database="testdb",
        host="localhost"
    )

    # Execute
    await conn.execute("""
        CREATE TABLE IF NOT EXISTS users (
            id SERIAL PRIMARY KEY,
            name TEXT NOT NULL,
            created_at TIMESTAMPTZ DEFAULT NOW()
        )
    """)

    # Insert and fetch
    await conn.execute("INSERT INTO users (name) VALUES ($1)", "Alice")
    row = await conn.fetchrow("SELECT * FROM users WHERE name = $1", "Alice")
    print(row["name"], row["id"])

    # Fetch multiple
    rows = await conn.fetch("SELECT * FROM users ORDER BY id")
    for row in rows:
        print(dict(row))

    await conn.close()

asyncio.run(main())
```

### Advanced Examples
```python
import asyncpg
import asyncio
from typing import Optional

class Database:
    def __init__(self, dsn: str):
        self.dsn = dsn
        self.pool: Optional[asyncpg.Pool] = None

    async def connect(self, min_size=5, max_size=20):
        self.pool = await asyncpg.create_pool(
            self.dsn,
            min_size=min_size,
            max_size=max_size,
            command_timeout=60,
            max_inactive_connection_lifetime=300,
        )

    async def close(self):
        if self.pool:
            await self.pool.close()

    async def fetch_user(self, user_id: int):
        async with self.pool.acquire() as conn:
            return await conn.fetchrow(
                "SELECT id, name, email FROM users WHERE id = $1",
                user_id
            )

    async def create_user(self, name: str, email: str) -> int:
        async with self.pool.acquire() as conn:
            return await conn.fetchval(
                "INSERT INTO users (name, email) VALUES ($1, $2) RETURNING id",
                name, email
            )

    async def get_users_batch(self, offset: int, limit: int):
        async with self.pool.acquire() as conn:
            # Prepared statement caching
            stmt = await conn.prepare(
                "SELECT * FROM users ORDER BY id OFFSET $1 LIMIT $2"
            )
            return await stmt.fetch(offset, limit)

# Usage
async def main():
    db = Database("postgresql://postgres:postgres@localhost:5432/testdb")
    await db.connect()
    try:
        user_id = await db.create_user("Bob", "bob@example.com")
        user = await db.fetch_user(user_id)
        print(user)
    finally:
        await db.close()

asyncio.run(main())
```

## Connection pools

### What It Is
A connection pool is a cache of database connections maintained so that connections can be reused across requests, avoiding the overhead of establishing a new connection for each operation. Both psycopg2 and asyncpg provide connection pooling, and there are also external pool implementations like PgBouncer (a dedicated connection pooler).

### Why It Is Important
Creating a new PostgreSQL connection is expensive (TCP handshake, SSL negotiation, authentication). In web applications handling hundreds of requests per second, pooling is essential for performance. Pools also enforce connection limits, prevent resource exhaustion, and can perform health checks.

### psycopg2 Pooling (psycopg2.pool / SQLAlchemy)
```python
from psycopg2 import pool

# ThreadedConnectionPool (for multithreaded apps)
connection_pool = pool.ThreadedConnectionPool(
    minconn=5,
    maxconn=20,
    host="localhost",
    port=5432,
    dbname="mydb",
    user="postgres",
    password="postgres"
)

def get_db_connection():
    return connection_pool.getconn()

def return_db_connection(conn):
    connection_pool.putconn(conn)

# Usage
conn = get_db_connection()
try:
    with conn.cursor() as cur:
        cur.execute("SELECT count(*) FROM users")
        print(cur.fetchone())
finally:
    return_db_connection(conn)

# Close all connections
connection_pool.closeall()
```

### asyncpg Pooling
```python
import asyncpg
import asyncio

async def main():
    pool = await asyncpg.create_pool(
        "postgresql://postgres:postgres@localhost:5432/testdb",
        min_size=10,
        max_size=50,
        max_queries=50000,  # Recycle connection after this many queries
        max_inactive_connection_lifetime=300,  # 5 minutes
        setup=None,  # Optional: function to run on each new connection
    )

    async with pool.acquire() as conn:
        # Connection is automatically returned to pool after context exit
        result = await conn.fetchval("SELECT 1")
        print(result)

    # Using pool with FastAPI-like pattern
    async def get_user(user_id: int):
        async with pool.acquire() as conn:
            return await conn.fetchrow(
                "SELECT * FROM users WHERE id = $1", user_id
            )

    # Transaction within pooled connection
    async with pool.acquire() as conn:
        async with conn.transaction():
            await conn.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
            await conn.execute("UPDATE accounts SET balance = balance + 100 WHERE id = 2")

    await pool.close()

asyncio.run(main())
```

### PgBouncer (External Connection Pooler)
```python
# PgBouncer sits between application and PostgreSQL
# Application connects to PgBouncer port (usually 6432)
# PgBouncer manages actual connections to PostgreSQL

import psycopg2

# Application config connects to PgBouncer
conn = psycopg2.connect(
    host="localhost",
    port=6432,  # PgBouncer port
    dbname="mydb",
    user="app_user",
    password="app_pass"
)
```

### Real-World Use Cases
- **Web applications**: FastAPI, Django, Flask backends with high request volume.
- **Data warehouses**: Analytical queries against large PostgreSQL datasets.
- **ETL pipelines**: Extracting, transforming, and loading data between systems.
- **Geospatial applications**: PostgreSQL with PostGIS for location-based queries.
- **Financial systems**: ACID-compliant transaction processing.

### Common Mistakes
- Using string formatting (`f"SELECT ... {value}"`) instead of parameterized queries (`%s` placeholders), leading to SQL injection.
- Not using connection pooling in web applications (creating a connection per request).
- Forgetting to close connections or return them to the pool.
- Using `fetchall()` on massive result sets instead of server-side cursors.
- Ignoring SSL/TLS configuration for production database connections.
- Setting `max_connections` too high, overwhelming the PostgreSQL server.

### Best Practices
- Always use connection pooling, not per-request connection creation.
- Use parameterized queries with `%s` (psycopg2) or `$1` (asyncpg) placeholders.
- Use `with conn:` (context manager) for automatic transaction management.
- Set `application_name` in connection parameters for monitoring.
- Configure statement timeout and connection timeout.
- Use `execute_values` or `copy_from` for bulk operations (10-100x faster).
- Monitor pool usage and set appropriate `min_size`/`max_size`.

### Performance Considerations
- asyncpg is 2-3x faster than psycopg2 for OLTP workloads.
- `execute_values` in psycopg2 is 10-50x faster than individual `execute` calls for batch inserts.
- PostgreSQL's `COPY` is the fastest way to transfer bulk data.
- Prepared statements reduce query planning overhead for repeated queries.
- Connection pools should match max PostgreSQL `max_connections` divided by application instances.
- Use connection pool sizing formula: `connections = (core_count * 2) + effective_spindle_count`.

### Interview Questions
1. What is the difference between psycopg2 and asyncpg?
2. How do connection pools work and why are they important?
3. Explain PostgreSQL's MVCC (Multi-Version Concurrency Control).
4. What are prepared statements and how do they improve performance?
5. How does the `COPY` command work for bulk data transfer?
6. What is the difference between `fetchone()`, `fetchmany()`, and `fetchall()`?
7. Explain PostgreSQL transaction isolation levels and how to set them in Python.

### Coding Challenges
1. **User Management API**: Build an async FastAPI application with PostgreSQL users table, asyncpg connection pooling, and CRUD endpoints.
2. **Bulk Data Importer**: Write a script that imports a large CSV file into PostgreSQL using COPY, measuring throughput.
3. **Connection Pool Monitor**: Create a tool that monitors and reports connection pool metrics (active, idle, wait time) for both psycopg2 and asyncpg pools.

### Related Topics
- SQLAlchemy (ORM layer on top of psycopg2/asyncpg)
- Alembic (database migrations for PostgreSQL)
- PgBouncer (lightweight connection pooler)
- PostgreSQL replication and streaming
- TimescaleDB (time-series extension for PostgreSQL)
