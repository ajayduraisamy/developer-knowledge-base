# SQLAlchemy - Engine, Session, ORM models, relationships, Alembic

## Introduction
SQLAlchemy is the most powerful and widely used Object-Relational Mapping (ORM) library for Python. It provides a full suite of database interaction tools: a high-level ORM that maps Python classes to database tables, a low-level Core expression language for building SQL queries programmatically, connection pooling, schema migration support via Alembic, and compatibility with virtually every major SQL database (PostgreSQL, MySQL, SQLite, Oracle, MSSQL). SQLAlchemy's design philosophy centers on the "unit of work" pattern, where changes are tracked and flushed to the database in batches, minimizing round trips and providing a clean separation between application logic and persistence concerns.

## Engine

### What It Is
The Engine is the starting point for any SQLAlchemy application. It represents a connection factory and database dialect configuration, encapsulating connection pooling, URL-based database specification, and execution strategies. The Engine does not represent an active connection itself but rather the ability to create connections on demand.

### Why It Is Important
The Engine centralizes database configuration and provides thread-safe connection management. It handles connection pooling, reconnection logic, and dialect-specific behaviors, allowing the rest of the application to remain database-agnostic.

### How It Works Internally
SQLAlchemy's Engine uses a `Pool` object to manage connections. When `engine.connect()` is called, the pool returns a `Connection` object wrapped around a DBAPI-native connection. The Dialect component translates SQLAlchemy's generic SQL expressions into database-specific syntax (e.g., PostgreSQL's `ILIKE` vs SQLite's `LIKE`). The Engine lazily initializes the pool on first use and can be configured with pool size, overflow, timeout, and recycling settings.

### Syntax
```python
from sqlalchemy import create_engine

# SQLite
engine = create_engine("sqlite:///mydb.db")

# PostgreSQL
engine = create_engine("postgresql://user:pass@localhost/mydb")

# MySQL
engine = create_engine("mysql+pymysql://user:pass@localhost/mydb")

# With connection pool settings
engine = create_engine(
    "postgresql://user:pass@localhost/mydb",
    pool_size=5,
    max_overflow=10,
    pool_timeout=30,
    pool_recycle=3600,
    echo=True  # Log all SQL
)
```

### Beginner Examples
```python
from sqlalchemy import create_engine, text

engine = create_engine("sqlite:///:memory:")
with engine.connect() as conn:
    conn.execute(text("CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT)"))
    conn.execute(
        text("INSERT INTO users (name) VALUES (:name)"),
        {"name": "Alice"}
    )
    result = conn.execute(text("SELECT * FROM users"))
    for row in result:
        print(row)
    conn.commit()
```

### Intermediate Examples
```python
from sqlalchemy import create_engine, text

engine = create_engine("sqlite:///orders.db", echo=True)

# Transactional execution
def transfer_funds(from_id, to_id, amount):
    with engine.begin() as conn:
        conn.execute(
            text("UPDATE accounts SET balance = balance - :amount WHERE id = :id"),
            {"amount": amount, "id": from_id}
        )
        conn.execute(
            text("UPDATE accounts SET balance = balance + :amount WHERE id = :id"),
            {"amount": amount, "id": to_id}
        )
    # Auto-committed if no exception, auto-rolled back on error
```

## Session

### What It Is
The Session is the central unit of work in SQLAlchemy ORM. It tracks the state of all objects loaded during its lifetime and manages their synchronization with the database. The Session implements the "Identity Map" pattern, ensuring that each database row corresponds to at most one Python object instance within the session's scope.

### Why It Is Important
The Session provides a transactional scope for ORM operations. It handles dirty tracking (detecting modified objects), cascade operations (automatically persisting related objects), and flush timing (synchronizing changes to the database at the right moment).

### How It Works Internally
The Session maintains several collections: `new` (objects pending INSERT), `dirty` (modified objects pending UPDATE), and `deleted` (objects pending DELETE). When `session.flush()` is called, SQLAlchemy generates INSERT/UPDATE/DELETE statements in the correct order respecting foreign key dependencies. The Session binds to an Engine and creates transactions lazily. The Identity Map (stored in `session.identity_map`) caches objects by their primary key, enabling fast lookups and ensuring consistency.

### Syntax
```python
from sqlalchemy.orm import Session

# Basic usage
session = Session(engine)

# Context manager
with Session(engine) as session:
    user = session.get(User, 1)
    user.name = "Bob"
    session.commit()

# Sessionmaker (recommended)
from sqlalchemy.orm import sessionmaker
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
session = SessionLocal()
```

### Beginner Examples
```python
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.orm import declarative_base, Session

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    name = Column(String)

engine = create_engine("sqlite:///:memory:")
Base.metadata.create_all(engine)

with Session(engine) as session:
    user = User(name="Alice")
    session.add(user)
    session.commit()
    print(f"Created user with id: {user.id}")
```

### Intermediate Examples
```python
from sqlalchemy.orm import sessionmaker

SessionLocal = sessionmaker(bind=engine)

# Dependency injection pattern (e.g., FastAPI)
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Bulk operations
def create_users(names):
    with SessionLocal() as session:
        users = [User(name=name) for name in names]
        session.add_all(users)
        session.commit()
        return users

# Query with filtering
with SessionLocal() as session:
    users = session.query(User).filter(User.name.like("A%")).order_by(User.id).all()
    for user in users:
        print(user.name, user.id)
```

## ORM models

### What It Is
ORM models are Python classes that map to database tables. Each class attribute mapped to a Column becomes a table column. SQLAlchemy's declarative base system allows you to define models using a concise, class-based syntax. Models can include constraints, indexes, default values, and relationships to other models.

### Syntax
```python
from sqlalchemy import Column, Integer, String, Float, DateTime, ForeignKey, Text
from sqlalchemy.orm import declarative_base, relationship
from datetime import datetime

Base = declarative_base()

class Product(Base):
    __tablename__ = "products"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(200), nullable=False)
    price = Column(Float, nullable=False)
    description = Column(Text)
    created_at = Column(DateTime, default=datetime.utcnow)
    category_id = Column(Integer, ForeignKey("categories.id"))

    category = relationship("Category", back_populates="products")
```

### Advanced Examples
```python
from sqlalchemy import (
    Column, Integer, String, Float, DateTime, ForeignKey, Text,
    UniqueConstraint, CheckConstraint, Enum as SAEnum, Index
)
from sqlalchemy.orm import declarative_base, relationship, validates
from datetime import datetime
import enum

Base = declarative_base()

class OrderStatus(enum.Enum):
    PENDING = "pending"
    CONFIRMED = "confirmed"
    SHIPPED = "shipped"
    DELIVERED = "delivered"
    CANCELLED = "cancelled"

class Customer(Base):
    __tablename__ = "customers"

    id = Column(Integer, primary_key=True)
    email = Column(String(255), unique=True, nullable=False, index=True)
    name = Column(String(100), nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)

    orders = relationship("Order", back_populates="customer")

    @validates("email")
    def validate_email(self, key, value):
        assert "@" in value, "Invalid email"
        return value.lower()

    __table_args__ = (
        UniqueConstraint("name", "email", name="uq_customer_name_email"),
        CheckConstraint("LENGTH(name) > 0", name="ck_customer_name_not_empty"),
    )

class Order(Base):
    __tablename__ = "orders"

    id = Column(Integer, primary_key=True)
    customer_id = Column(Integer, ForeignKey("customers.id"), nullable=False)
    status = Column(SAEnum(OrderStatus), default=OrderStatus.PENDING, nullable=False)
    total = Column(Float, default=0.0)
    created_at = Column(DateTime, default=datetime.utcnow)

    customer = relationship("Customer", back_populates="orders")
    items = relationship("OrderItem", back_populates="order", cascade="all, delete-orphan")

class OrderItem(Base):
    __tablename__ = "order_items"

    id = Column(Integer, primary_key=True)
    order_id = Column(Integer, ForeignKey("orders.id"), nullable=False)
    product_name = Column(String(200), nullable=False)
    quantity = Column(Integer, nullable=False, default=1)
    unit_price = Column(Float, nullable=False)

    order = relationship("Order", back_populates="items")

    __table_args__ = (
        Index("ix_order_items_order_id", "order_id"),
    )
```

## Relationships

### What It Is
Relationships define how models connect to each other: one-to-many, many-to-one, many-to-many, and one-to-one. SQLAlchemy uses the `relationship()` function to define these associations, with `ForeignKey` columns establishing the database-level constraint.

### Syntax
```python
from sqlalchemy import Table, Column, ForeignKey
from sqlalchemy.orm import relationship

# One-to-Many / Many-to-One
class Parent(Base):
    __tablename__ = "parents"
    id = Column(Integer, primary_key=True)
    children = relationship("Child", back_populates="parent")

class Child(Base):
    __tablename__ = "children"
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey("parents.id"))
    parent = relationship("Parent", back_populates="children")

# Many-to-Many
association_table = Table(
    "student_courses",
    Base.metadata,
    Column("student_id", Integer, ForeignKey("students.id")),
    Column("course_id", Integer, ForeignKey("courses.id")),
)

class Student(Base):
    __tablename__ = "students"
    id = Column(Integer, primary_key=True)
    courses = relationship("Course", secondary=association_table, back_populates="students")

class Course(Base):
    __tablename__ = "courses"
    id = Column(Integer, primary_key=True)
    students = relationship("Student", secondary=association_table, back_populates="courses")
```

### Advanced Examples
```python
from sqlalchemy.orm import relationship, backref

# Self-referential relationship (tree structure)
class Category(Base):
    __tablename__ = "categories"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    parent_id = Column(Integer, ForeignKey("categories.id"))

    parent = relationship("Category", remote_side=[id], back_populates="children")
    children = relationship("Category", back_populates="parent")

# Relationship with join conditions
class Employee(Base):
    __tablename__ = "employees"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    department_id = Column(Integer, ForeignKey("departments.id"))
    manager_id = Column(Integer, ForeignKey("employees.id"))

    manager = relationship("Employee", remote_side=[id], back_populates="subordinates")
    subordinates = relationship("Employee", back_populates="manager")

# Eager loading strategies
with Session(engine) as session:
    # Lazy loading (default) - loads on access
    orders = session.query(Order).all()
    for order in orders:
        print(order.items)  # Triggers additional query

    # Eager loading with joinedload
    from sqlalchemy.orm import joinedload
    orders = session.query(Order).options(joinedload(Order.items)).all()
    for order in orders:
        print(order.items)  # No additional query

    # Eager loading with selectinload (for collections)
    from sqlalchemy.orm import selectinload
    customers = session.query(Customer).options(selectinload(Customer.orders)).all()
```

## Alembic migrations

### What It Is
Alembic is a lightweight database migration tool that works hand-in-hand with SQLAlchemy. It allows you to version-control your database schema, apply incremental changes, and roll back to previous versions when needed. Migrations are Python scripts that describe schema changes (adding tables, columns, indexes, etc.).

### Why It Is Important
As applications evolve, database schemas must change. Alembic provides a systematic, reproducible way to apply these changes across development, staging, and production environments without manual SQL scripts. It integrates with SQLAlchemy models to auto-generate migration stubs.

### How It Works Internally
Alembic maintains a migration chain in a directory (usually `alembic/versions/`). Each migration script has a `revision` identifier and a `down_revision` pointing to the previous migration. Alembic tracks which migrations have been applied in the `alembic_version` table. The `upgrade()` function applies changes, while `downgrade()` reverts them.

### Syntax
```bash
# Install
pip install alembic

# Initialize
alembic init alembic

# Configure alembic.ini with database URL
# sqlalchemy.url = sqlite:///mydb.db

# Auto-generate migration
alembic revision --autogenerate -m "add_users_table"

# Apply migration
alembic upgrade head

# Rollback
alembic downgrade -1

# View history
alembic history
```

### Advanced Examples
```python
# alembic/versions/xxxx_add_users_table.py
"""add users table

Revision ID: xxxx
Revises:
Create Date: 2024-01-01
"""
from alembic import op
import sqlalchemy as sa

revision = "xxxx"
down_revision = None

def upgrade():
    op.create_table(
        "users",
        sa.Column("id", sa.Integer(), nullable=False),
        sa.Column("name", sa.String(length=100), nullable=False),
        sa.Column("email", sa.String(length=255), nullable=False),
        sa.Column("created_at", sa.DateTime(), server_default=sa.func.now()),
        sa.PrimaryKeyConstraint("id"),
        sa.UniqueConstraint("email"),
    )
    op.create_index("ix_users_email", "users", ["email"])

def downgrade():
    op.drop_index("ix_users_email", table_name="users")
    op.drop_table("users")
```

```python
# alembic/env.py - Customizing autogenerate
from alembic import context
from myapp.models import Base

target_metadata = Base.metadata

def run_migrations_offline():
    url = config.get_main_option("sqlalchemy.url")
    context.configure(url=url, target_metadata=target_metadata)
    with context.begin_transaction():
        context.run_migrations()

def run_migrations_online():
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )
    with connectable.connect() as connection:
        context.configure(connection=connection, target_metadata=target_metadata)
        with context.begin_transaction():
            context.run_migrations()
```

### Real-World Use Cases
- **Web applications with evolving schemas**: Adding features that require new tables or columns.
- **Multi-environment deployments**: Ensuring schema consistency across dev, staging, and production.
- **Team collaboration**: Multiple developers can work on schema changes without conflicts.
- **Audit compliance**: Complete history of schema changes for regulatory requirements.

### Common Mistakes
- Using `session.query().all()` without filtering on large tables.
- Forgetting to call `session.commit()` after mutations.
- Creating relationships without corresponding ForeignKey columns.
- Ignoring N+1 query problems by not using eager loading.
- Modifying auto-generated migration scripts after they've been applied.

### Best Practices
- Always use `sessionmaker` instead of creating Session instances directly.
- Use `joinedload` or `selectinload` to solve N+1 query issues.
- Define `__repr__` methods on models for debugging.
- Use `cascade="all, delete-orphan"` for parent-child relationships.
- Write explicit downgrade functions in Alembic migrations.
- Run `alembic check` to detect schema/model mismatches before deploying.

### Performance Considerations
- Use `selectinload` for to-many relationships (avoids Cartesian product of joinedload).
- Use `contains_eager()` when you've already joined in the query.
- Batch inserts with `session.add_all()` and commit once.
- Use `bulk_insert_mappings()` and `bulk_update_mappings()` for thousands of rows.
- Consider `with_expression()` and column properties for computed fields.
- Profile queries with `echo=True` or SQLAlchemy's query event system.

### Interview Questions
1. Explain the difference between `engine.execute()` and `session.execute()`.
2. What is the N+1 query problem and how does SQLAlchemy solve it?
3. How does the Identity Map pattern work in SQLAlchemy?
4. What is the difference between `joinedload` and `selectinload`?
5. How do you handle concurrent updates with SQLAlchemy (optimistic locking)?
6. Explain the purpose of Alembic's `autogenerate` feature.
7. What is the "unit of work" pattern?

### Coding Challenges
1. **Blog Platform Models**: Create User, Post, Comment, and Tag models with appropriate relationships, then write queries to fetch posts with their comments and tags.
2. **E-commerce Checkout**: Implement an order placement function that creates orders, validates stock, and handles inventory updates atomically.
3. **Migration Workflow**: Create an Alembic migration that adds a `last_login` column to a users table, populates it for existing users, and tests the downgrade.

### Related Topics
- Pydantic (data validation models often paired with SQLAlchemy)
- FastAPI integration (SQLAlchemy sessions as dependencies)
- Database indexing strategies
- SQLAlchemy Core (lower-level API without ORM)
- AioSQLAlchemy (async SQLAlchemy for asyncio applications)
