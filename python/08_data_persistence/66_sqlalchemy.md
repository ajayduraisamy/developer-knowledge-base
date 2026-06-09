# SQLAlchemy - Engine, Session, ORM models, relationships, Alembic

## Introduction

SQLAlchemy is the most popular SQL toolkit and Object-Relational Mapping (ORM) library for Python. It provides a full suite of enterprise-level persistence patterns designed for efficient and high-performing database access. SQLAlchemy offers both a low-level Core API (SQL expression language) and a high-level ORM that maps Python classes to database tables.

## Why It Is Important

SQLAlchemy abstracts away database differences, allowing you to write Python code that works with SQLite, PostgreSQL, MySQL, Oracle, and more without changing your application logic. The ORM eliminates boilerplate SQL, provides relationship management between models, and includes features like lazy/eager loading, identity maps, and unit of work pattern. It's the de facto standard for database access in Python web frameworks like FastAPI and Flask.

## Syntax

```python
from sqlalchemy import create_engine, Column, Integer, String, ForeignKey
from sqlalchemy.orm import declarative_base, sessionmaker, relationship

Base = declarative_base()
engine = create_engine("sqlite:///database.db")
Session = sessionmaker(bind=engine)
session = Session()

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    name = Column(String)

Base.metadata.create_all(engine)
```

## Examples

### Core Setup and Basic Models

```python
import os
from datetime import datetime
from typing import List, Optional

from sqlalchemy import (
    create_engine, Column, Integer, String, Float, Boolean,
    Text, DateTime, Date, Enum, ForeignKey, Table, UniqueConstraint,
    Index, and_, or_, not_, func, desc, asc
)
from sqlalchemy.orm import (
    declarative_base, sessionmaker, relationship, backref,
    joinedload, selectinload, subqueryload, contains_eager,
    Session as SASession
)
from sqlalchemy.exc import IntegrityError, SQLAlchemyError
import enum

DB_URL = os.environ.get("DATABASE_URL", "sqlite:///blog.db")
engine = create_engine(DB_URL, echo=False)
Base = declarative_base()
SessionLocal = sessionmaker(bind=engine, autocommit=False, autoflush=False)


class UserRole(str, enum.Enum):
    admin = "admin"
    editor = "editor"
    viewer = "viewer"


post_tags = Table(
    "post_tags", Base.metadata,
    Column("post_id", Integer, ForeignKey("posts.id"), primary_key=True),
    Column("tag_id", Integer, ForeignKey("tags.id"), primary_key=True)
)


class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    username = Column(String(50), unique=True, nullable=False, index=True)
    email = Column(String(120), unique=True, nullable=False)
    hashed_password = Column(String(255), nullable=False)
    role = Column(Enum(UserRole), default=UserRole.viewer)
    is_active = Column(Boolean, default=True)
    bio = Column(Text, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    posts = relationship("Post", back_populates="author", cascade="all, delete-orphan")
    profile = relationship("Profile", uselist=False, back_populates="user", cascade="all, delete-orphan")

    __table_args__ = (
        UniqueConstraint("email", name="uq_user_email"),
        Index("ix_user_username_email", "username", "email"),
    )

    def __repr__(self):
        return f"<User(id={self.id}, username='{self.username}', role='{self.role}')>"


class Profile(Base):
    __tablename__ = "profiles"

    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"), unique=True, nullable=False)
    full_name = Column(String(100))
    avatar_url = Column(String(500))
    website = Column(String(500))
    location = Column(String(100))
    birthday = Column(Date, nullable=True)
    phone = Column(String(20))

    user = relationship("User", back_populates="profile")

    def __repr__(self):
        return f"<Profile(user_id={self.user_id}, name='{self.full_name}')>"


class Category(Base):
    __tablename__ = "categories"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(50), unique=True, nullable=False)
    slug = Column(String(50), unique=True, nullable=False)
    description = Column(Text, nullable=True)

    posts = relationship("Post", back_populates="category")

    def __repr__(self):
        return f"<Category(id={self.id}, name='{self.name}')>"


class Tag(Base):
    __tablename__ = "tags"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(30), unique=True, nullable=False)
    slug = Column(String(30), unique=True, nullable=False)

    posts = relationship("Post", secondary=post_tags, back_populates="tags")

    def __repr__(self):
        return f"<Tag(id={self.id}, name='{self.name}')>"


class Post(Base):
    __tablename__ = "posts"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(200), nullable=False)
    slug = Column(String(200), unique=True, nullable=False, index=True)
    content = Column(Text, nullable=False)
    excerpt = Column(String(500), nullable=True)
    is_published = Column(Boolean, default=False)
    view_count = Column(Integer, default=0)
    rating = Column(Float, default=0.0)
    published_at = Column(DateTime, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    author_id = Column(Integer, ForeignKey("users.id"), nullable=False, index=True)
    category_id = Column(Integer, ForeignKey("categories.id"), nullable=False, index=True)

    author = relationship("User", back_populates="posts")
    category = relationship("Category", back_populates="posts")
    tags = relationship("Tag", secondary=post_tags, back_populates="posts")
    comments = relationship("Comment", back_populates="post", cascade="all, delete-orphan")

    def __repr__(self):
        return f"<Post(id={self.id}, title='{self.title}', published={self.is_published})>"


class Comment(Base):
    __tablename__ = "comments"

    id = Column(Integer, primary_key=True, index=True)
    content = Column(Text, nullable=False)
    author_name = Column(String(100), nullable=False)
    author_email = Column(String(120), nullable=False)
    is_approved = Column(Boolean, default=False)
    created_at = Column(DateTime, default=datetime.utcnow)

    post_id = Column(Integer, ForeignKey("posts.id"), nullable=False, index=True)

    post = relationship("Post", back_populates="comments")

    def __repr__(self):
        return f"<Comment(id={self.id}, post_id={self.post_id}, author='{self.author_name}')>"


Base.metadata.create_all(engine)
```

### CRUD Operations and Queries

```python
from datetime import datetime, timedelta
from sqlalchemy import func, text


def get_session() -> SASession:
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


def create_user(session: SASession, username: str, email: str, password: str, role: UserRole = UserRole.viewer) -> User:
    user = User(
        username=username,
        email=email,
        hashed_password=password,
        role=role
    )
    session.add(user)
    session.commit()
    session.refresh(user)
    return user


def bulk_create_users(session: SASession, users_data: list) -> List[User]:
    users = [User(**data) for data in users_data]
    session.add_all(users)
    session.commit()
    return users


def get_user_by_id(session: SASession, user_id: int) -> Optional[User]:
    return session.query(User).filter(User.id == user_id).first()


def get_user_by_username(session: SASession, username: str) -> Optional[User]:
    return session.query(User).filter(User.username == username).first()


def get_active_users(session: SASession) -> List[User]:
    return session.query(User).filter(User.is_active == True).order_by(User.created_at.desc()).all()


def get_users_by_role(session: SASession, role: UserRole) -> List[User]:
    return session.query(User).filter(User.role == role).all()


def update_user_email(session: SASession, user_id: int, new_email: str) -> bool:
    rows = session.query(User).filter(User.id == user_id).update({"email": new_email})
    session.commit()
    return rows > 0


def update_user_batch(session: SASession, user_ids: list, updates: dict) -> int:
    rows = session.query(User).filter(User.id.in_(user_ids)).update(updates, synchronize_session="fetch")
    session.commit()
    return rows


def delete_user(session: SASession, user_id: int) -> bool:
    user = session.query(User).filter(User.id == user_id).first()
    if not user:
        return False
    session.delete(user)
    session.commit()
    return True


def delete_inactive_users(session: SASession) -> int:
    rows = session.query(User).filter(User.is_active == False).delete(synchronize_session="fetch")
    session.commit()
    return rows


def create_post(session: SASession, title: str, content: str, author_id: int, category_id: int, tags: List[Tag] = None) -> Post:
    slug = title.lower().replace(" ", "-")
    post = Post(
        title=title,
        slug=slug,
        content=content,
        author_id=author_id,
        category_id=category_id,
        tags=tags or []
    )
    session.add(post)
    session.commit()
    session.refresh(post)
    return post


def get_published_posts(session: SASession) -> List[Post]:
    return session.query(Post).filter(Post.is_published == True).order_by(Post.published_at.desc()).all()


def get_posts_by_author(session: SASession, author_id: int) -> List[Post]:
    return session.query(Post).filter(Post.author_id == author_id).order_by(Post.created_at.desc()).all()


def get_posts_by_category(session: SASession, category_slug: str) -> List[Post]:
    return session.query(Post).join(Category).filter(Category.slug == category_slug).all()


def get_posts_by_tag(session: SASession, tag_slug: str) -> List[Post]:
    return session.query(Post).join(Post.tags).filter(Tag.slug == tag_slug).all()


def search_posts(session: SASession, query: str) -> List[Post]:
    return session.query(Post).filter(
        or_(
            Post.title.ilike(f"%{query}%"),
            Post.content.ilike(f"%{query}%")
        )
    ).order_by(Post.created_at.desc()).all()


def get_recent_posts(session: SASession, limit: int = 10) -> List[Post]:
    return session.query(Post).filter(
        Post.is_published == True
    ).order_by(Post.published_at.desc()).limit(limit).all()


def publish_post(session: SASession, post_id: int) -> bool:
    post = session.query(Post).filter(Post.id == post_id).first()
    if not post:
        return False
    post.is_published = True
    post.published_at = datetime.utcnow()
    session.commit()
    return True


def add_comment(session: SASession, post_id: int, content: str, author_name: str, author_email: str) -> Comment:
    comment = Comment(
        content=content,
        author_name=author_name,
        author_email=author_email,
        post_id=post_id
    )
    session.add(comment)
    session.commit()
    return comment


def get_post_with_comments(session: SASession, post_id: int) -> Optional[Post]:
    return session.query(Post).options(
        joinedload(Post.author),
        joinedload(Post.category),
        selectinload(Post.tags),
        selectinload(Post.comments)
    ).filter(Post.id == post_id).first()


def get_post_count_by_category(session: SASession) -> list:
    return session.query(
        Category.name,
        func.count(Post.id).label("post_count")
    ).outerjoin(Post).group_by(Category.id).order_by(desc("post_count")).all()


def get_popular_posts(session: SASession, min_views: int = 100) -> List[Post]:
    return session.query(Post).filter(
        Post.is_published == True,
        Post.view_count >= min_views
    ).order_by(Post.view_count.desc()).all()


def increment_view_count(session: SASession, post_id: int):
    session.query(Post).filter(Post.id == post_id).update(
        {Post.view_count: Post.view_count + 1},
        synchronize_session="evaluate"
    )
    session.commit()


def get_users_with_post_counts(session: SASession) -> list:
    return session.query(
        User.id,
        User.username,
        func.count(Post.id).label("post_count"),
        func.coalesce(func.sum(Post.view_count), 0).label("total_views")
    ).outerjoin(Post, Post.author_id == User.id).group_by(User.id).order_by(desc("total_views")).all()


def get_average_rating_by_category(session: SASession) -> list:
    return session.query(
        Category.name,
        func.avg(Post.rating).label("avg_rating"),
        func.count(Post.id).label("post_count")
    ).join(Post).filter(Post.is_published == True).group_by(Category.id).all()


def get_recent_comments(session: SASession, limit: int = 20) -> List[Comment]:
    return session.query(Comment).options(
        joinedload(Comment.post)
    ).order_by(Comment.created_at.desc()).limit(limit).all()


def get_top_commenters(session: SASession) -> list:
    return session.query(
        Comment.author_name,
        Comment.author_email,
        func.count(Comment.id).label("comment_count")
    ).group_by(Comment.author_email).order_by(desc("comment_count")).limit(10).all()
```

### Relationships (One-to-Many, Many-to-Many)

```python
from sqlalchemy.orm import joinedload, selectinload, subqueryload


def demonstrate_one_to_many(session: SASession):
    user = session.query(User).filter(User.username == "alice").first()
    print(f"User: {user.username}")
    for post in user.posts:
        print(f"  Post: {post.title}")

    post = session.query(Post).filter(Post.title.ilike("%Python%")).first()
    if post:
        print(f"Author of '{post.title}': {post.author.username}")


def demonstrate_many_to_many(session: SASession):
    post = session.query(Post).options(selectinload(Post.tags)).first()
    if post:
        print(f"Post: {post.title}")
        for tag in post.tags:
            print(f"  Tag: {tag.name}")

    tag = session.query(Tag).filter(Tag.name == "python").first()
    if tag:
        print(f"Tag: {tag.name}")
        for post in tag.posts:
            print(f"  Post: {post.title}")


def eager_loading_demo(session: SASession):
    posts = session.query(Post).options(
        joinedload(Post.author),
        joinedload(Post.category),
        selectinload(Post.tags)
    ).limit(10).all()

    for post in posts:
        print(f"{post.title} by {post.author.username} in {post.category.name}")
        tag_names = [t.name for t in post.tags]
        print(f"  Tags: {', '.join(tag_names)}")
```

## Beginner Examples

```python
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.orm import declarative_base, sessionmaker

engine = create_engine("sqlite:///test.db", echo=True)
Base = declarative_base()
Session = sessionmaker(bind=engine)
session = Session()

class Person(Base):
    __tablename__ = "persons"
    id = Column(Integer, primary_key=True)
    name = Column(String(50))
    age = Column(Integer)

Base.metadata.create_all(engine)

p1 = Person(name="Alice", age=30)
p2 = Person(name="Bob", age=25)
session.add(p1)
session.add(p2)
session.commit()

people = session.query(Person).all()
for p in people:
    print(p.name, p.age)

alice = session.query(Person).filter(Person.name == "Alice").first()
alice.age = 31
session.commit()

session.query(Person).filter(Person.name == "Bob").delete()
session.commit()

session.close()
```

## Intermediate Examples

```python
from datetime import datetime, timedelta
from sqlalchemy import func, and_, or_
from sqlalchemy.orm import joinedload


def advanced_query_examples(session: SASession):
    posts_today = session.query(Post).filter(
        func.date(Post.created_at) == func.current_date()
    ).all()

    posts_last_week = session.query(Post).filter(
        Post.created_at >= datetime.utcnow() - timedelta(days=7)
    ).all()

    high_rated_published = session.query(Post).filter(
        and_(
            Post.is_published == True,
            Post.rating >= 4.0
        )
    ).all()

    users_with_posts = session.query(User).outerjoin(Post).filter(
        Post.id == None
    ).all()

    stats = session.query(
        func.date(Post.created_at).label("day"),
        func.count(Post.id).label("count")
    ).filter(
        Post.is_published == True
    ).group_by(
        func.date(Post.created_at)
    ).having(
        func.count(Post.id) > 1
    ).all()

    subq = session.query(Post.author_id, func.count(Post.id).label("pc")).group_by(Post.author_id).subquery()
    prolific = session.query(User).join(subq, User.id == subq.c.author_id).filter(subq.c.pc >= 5).all()


def transaction_example(session: SASession):
    try:
        user = User(username="test_user", email="test@example.com", hashed_password="secret")
        session.add(user)
        session.flush()
        post = Post(title="Test Post", slug="test-post", content="Content", author_id=user.id, category_id=1)
        session.add(post)
        session.commit()
    except SQLAlchemyError:
        session.rollback()
        raise
```

## Advanced Examples

```python
from contextlib import contextmanager
from sqlalchemy.orm import Session, joinedload, selectinload, contains_eager
from sqlalchemy import event, inspect, text
from sqlalchemy.pool import QueuePool


@contextmanager
def session_scope():
    session = SessionLocal()
    try:
        yield session
        session.commit()
    except SQLAlchemyError:
        session.rollback()
        raise
    finally:
        session.close()


class PaginatedQuery:
    def __init__(self, query, page: int = 1, per_page: int = 20):
        self.query = query
        self.page = page
        self.per_page = per_page
        self.total = query.count()
        self.pages = (self.total + per_page - 1) // per_page

    def items(self):
        return self.query.offset((self.page - 1) * self.per_page).limit(self.per_page).all()

    def has_prev(self) -> bool:
        return self.page > 1

    def has_next(self) -> bool:
        return self.page < self.pages

    def iter_pages(self, left_edge=2, left_current=2, right_current=5, right_edge=2):
        last = self.pages
        pages = []
        for num in range(1, last + 1):
            if num <= left_edge or (num > self.page - left_current - 1 and num < self.page + right_current) or num > last - right_edge:
                pages.append(num)
            elif pages and pages[-1] is not None:
                pages.append(None)
        return pages


def paginate_posts(session: SASession, page: int = 1, per_page: int = 10) -> PaginatedQuery:
    query = session.query(Post).filter(Post.is_published == True).order_by(Post.published_at.desc())
    return PaginatedQuery(query, page=page, per_page=per_page)


def complex_report(session: SASession) -> dict:
    total_users = session.query(func.count(User.id)).scalar()
    total_posts = session.query(func.count(Post.id)).scalar()
    total_comments = session.query(func.count(Comment.id)).scalar()
    published_posts = session.query(func.count(Post.id)).filter(Post.is_published == True).scalar()
    avg_rating = session.query(func.avg(Post.rating)).filter(Post.is_published == True).scalar()

    top_authors = session.query(
        User.username,
        func.count(Post.id).label("post_count"),
        func.coalesce(func.sum(Post.view_count), 0).label("total_views")
    ).outerjoin(Post).group_by(User.id).order_by(desc("total_views")).limit(5).all()

    return {
        "total_users": total_users,
        "total_posts": total_posts,
        "total_comments": total_comments,
        "published_posts": published_posts,
        "avg_rating": float(avg_rating or 0),
        "top_authors": [
            {"username": u, "posts": pc, "views": tv}
            for u, pc, tv in top_authors
        ]
    }


@event.listens_for(engine, "before_execute")
def log_statements(conn, clause, multiparams, params):
    print(f"SQL: {clause}")


def raw_sql_example(session: SASession):
    result = session.execute(text("SELECT COUNT(*) FROM users"))
    count = result.scalar()
    print(f"Total users: {count}")

    result = session.execute(
        text("SELECT id, username FROM users WHERE is_active = :active"),
        {"active": True}
    )
    for row in result:
        print(f"Active user: {row.id} - {row.username}")


def batch_operations(session: SASession):
    session.bulk_insert_mappings(User, [
        {"username": f"user{i}", "email": f"user{i}@example.com", "hashed_password": "pw", "role": UserRole.viewer}
        for i in range(100)
    ])
    session.commit()

    session.bulk_update_mappings(User, [
        {"id": i, "is_active": False}
        for i in range(1, 11)
    ])
    session.commit()


def soft_delete_and_filter(session: SASession):
    class SoftDeleteMixin:
        is_deleted = Column(Boolean, default=False)

    def filter_deleted(cls):
        return cls.is_deleted == False

    @event.listens_for(session, "before_flush")
    def before_flush(session, flush_context, instances):
        for instance in session.deleted:
            if hasattr(instance, "is_deleted"):
                instance.is_deleted = True
                session.expunge(instance)
```

## Real-World Use Cases

SQLAlchemy powers database layers in major Python web frameworks (Flask-SQLAlchemy, FastAPI with SQLAlchemy), data processing pipelines, ETL systems, content management systems, e-commerce platforms, analytics dashboards, and any Python application requiring robust database interaction with support for multiple database backends.

## Common Mistakes

- Not creating a new session per request/thread (session is not thread-safe).
- Forgetting to call `session.commit()` after changes.
- N+1 query problem — accessing relationships without eager loading.
- Using `all()` on large datasets without pagination.
- Ignoring `synchronize_session` parameter in bulk updates/deletes.
- Not handling `IntegrityError` for unique constraint violations.
- Over-fetching columns when only a subset is needed.

## Best Practices

- Use dependency injection or context managers for session management.
- Use eager loading (`joinedload`, `selectinload`) to avoid N+1 queries.
- Implement pagination for list endpoints.
- Use `synchronize_session` appropriately in bulk operations.
- Define `__repr__` on models for debugging.
- Use Alembic for schema migrations.
- Set `autocommit=False` and manage transactions explicitly.
- Use `session.refresh()` after committing new objects to get generated values.

## Interview Questions

1. What is the difference between SQLAlchemy Core and ORM? — Core is a SQL abstraction toolkit using SQL expression language; ORM builds on Core to map Python classes to database tables with identity maps and unit of work.
2. Explain the N+1 query problem and how to fix it. — When accessing related objects triggers individual queries per item; fix with eager loading via `joinedload` or `selectinload`.
3. What is the Unit of Work pattern in SQLAlchemy? — The session tracks changes to objects and flushes them in a single transaction, ensuring consistency and minimizing database round trips.
4. How do you handle relationships in SQLAlchemy? — Using `relationship()` with `ForeignKey` columns; options include `back_populates`, `cascade`, `lazy` loading strategies, and secondary tables for many-to-many.
5. What is the difference between `joinedload` and `selectinload`? — `joinedload` uses a JOIN to load related objects in one query; `selectinload` uses a separate SELECT with IN clause; `selectinload` is better for collections.

## Coding Challenges

1. **Blog Platform** — Build models for users, posts, categories, tags, and comments with all relationship types. Implement CRUD, pagination, search, and full-text search with PostgreSQL.
2. **E-Commerce Catalog** — Create models for products, categories, variants, inventory, and pricing. Implement complex filtering, aggregation queries, and reporting.
3. **Social Network** — Design models for users, friendships, posts, likes, and shares. Implement feed generation with optimized eager loading.
4. **Task Management** — Build a project management system with projects, tasks, assignments, comments, and time tracking. Use Alembic for schema versioning.
5. **Multi-Tenant SaaS** — Implement a multi-tenant database schema with shared or isolated databases, tenant-aware queries, and migration strategies.

## Summary

SQLAlchemy is a powerful and flexible ORM that provides both high-level object-relational mapping and low-level SQL abstraction. Key components include Engine (database connection), Session (unit of work), declarative models, relationship management, and an expressive query API. It supports multiple database backends, relationship patterns (one-to-many, many-to-many), eager/lazy loading, and integrates with Alembic for migrations.

## Related Topics

- Alembic (database migration tool for SQLAlchemy)
- PostgreSQL (production database often paired with SQLAlchemy)
- FastAPI/Flask (web frameworks with SQLAlchemy integration)
- Pydantic (data validation, often used with SQLAlchemy models)
- asyncpg / databases (async database libraries compatible with SQLAlchemy)
