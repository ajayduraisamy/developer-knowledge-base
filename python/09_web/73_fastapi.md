# FastAPI - FastAPI(), @app.get(), Pydantic models, async, OpenAPI

## Introduction
FastAPI is a modern, high-performance web framework for building APIs with Python, leveraging Python type hints for automatic request validation, serialization, and documentation generation. Built on Starlette (ASGI framework) and Pydantic (data validation), FastAPI provides automatic OpenAPI (Swagger) and ReDoc documentation, asynchronous request handling, dependency injection, and WebSocket support. Its performance rivals Node.js and Go frameworks, making it suitable for high-throughput APIs. FastAPI's design philosophy emphasizes developer productivity, code correctness through type safety, and automatic interactive documentation.

## FastAPI() app

### What It Is
`FastAPI()` is the main application class. It creates an ASGI application that handles HTTP requests, WebSocket connections, background tasks, and lifecycle events. The application instance is the central point for registering routes, middleware, exception handlers, and dependency overrides.

### Why It Is Important
The FastAPI application is the entry point for running the server (via uvicorn) and organizing all API components. It manages OpenAPI schema generation, integrates CORS middleware, handles startup/shutdown events, and provides the dependency injection container.

### How It Works Internally
FastAPI inherits from Starlette's `Router` and adds OpenAPI generation on top. When a route is registered, FastAPI inspects the function's type hints and Pydantic models to build the OpenAPI schema. At runtime, incoming requests are parsed according to this schema, validated using Pydantic, and passed to the endpoint. Responses are similarly validated and serialized. The ASGI protocol allows FastAPI to handle multiple requests concurrently without thread pools.

### Syntax
```python
from fastapi import FastAPI

app = FastAPI(
    title="My API",
    description="A sample FastAPI application",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc",
    openapi_url="/openapi.json",
)

@app.get("/")
def read_root():
    return {"Hello": "World"}
```

### Beginner Examples
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return {"message": "Hello, FastAPI!"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

### Intermediate Examples
```python
from fastapi import FastAPI, HTTPException, status
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    print("Starting up...")
    db.connect()
    yield
    # Shutdown
    print("Shutting down...")
    db.close()

app = FastAPI(lifespan=lifespan)

# Custom exception handler
class AppException(Exception):
    def __init__(self, message: str, code: int = 400):
        self.message = message
        self.code = code

@app.exception_handler(AppException)
async def app_exception_handler(request, exc):
    return JSONResponse(
        status_code=exc.code,
        content={"error": exc.message}
    )

# Middleware
@app.middleware("http")
async def add_process_time_header(request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

## @app.get() and @app.post()

### What They Are
`@app.get()`, `@app.post()`, `@app.put()`, `@app.patch()`, `@app.delete()` are decorators that register functions as API endpoint handlers for specific HTTP methods and URL paths. They are the primary way to define API routes in FastAPI.

### Why They Are Important
These decorators implement the routing layer of the API. They map HTTP methods and URL patterns to handler functions, define path/query/header/body parameters through type hints, and automatically generate the corresponding OpenAPI schema entries.

### Syntax
```python
from fastapi import FastAPI, Query, Path, Body, Header

app = FastAPI()

# GET with query parameters
@app.get("/search")
def search(
    q: str = Query(None, min_length=3, max_length=50),
    page: int = Query(1, ge=1),
    size: int = Query(20, ge=1, le=100)
):
    return {"query": q, "page": page, "size": size}

# POST with request body
@app.post("/users")
def create_user(user: UserCreate):
    return {"id": 123, **user.dict()}

# Path parameters with validation
@app.get("/items/{item_id}")
def get_item(
    item_id: int = Path(..., ge=1, title="Item ID"),
    include_details: bool = Query(False)
):
    return {"item_id": item_id, "details": include_details}

# Multiple methods on same path
@app.get("/resources")
def list_resources():
    return [{"id": 1}]

@app.post("/resources")
def create_resource(data: ResourceCreate):
    return {"id": 2, **data.dict()}

@app.put("/resources/{resource_id}")
def update_resource(resource_id: int, data: ResourceUpdate):
    return {"id": resource_id, **data.dict()}

@app.delete("/resources/{resource_id}")
def delete_resource(resource_id: int):
    return {"deleted": resource_id}
```

## Pydantic models

### What They Are
Pydantic models are Python classes that inherit from `BaseModel` and define data schemas using type annotations. FastAPI uses them for request body validation, response serialization, OpenAPI schema generation, and automatic documentation.

### Why They Are Important
Pydantic models provide compile-time and runtime data validation. They ensure that API inputs meet the specified constraints (types, ranges, formats) and that outputs conform to the expected schema. This eliminates boilerplate validation code and catches errors early.

### How They Work Internally
When a Pydantic model is instantiated, its `__init__` method performs validation using the field type annotations. Validation uses Pydantic's core engine (written in Rust for v2) to apply type coercion, run validators, check constraints, and produce error messages. FastAPI extracts the model's JSON schema via `model_json_schema()` and includes it in the OpenAPI spec.

### Syntax
```python
from pydantic import BaseModel, Field, EmailStr, validator, field_validator
from datetime import datetime
from typing import Optional, List

class UserBase(BaseModel):
    username: str = Field(..., min_length=3, max_length=50)
    email: EmailStr
    age: int = Field(18, ge=0, le=150)

class UserCreate(UserBase):
    password: str = Field(..., min_length=8)

class UserUpdate(BaseModel):
    username: Optional[str] = None
    email: Optional[EmailStr] = None
    age: Optional[int] = None

class UserResponse(UserBase):
    id: int
    created_at: datetime
    is_active: bool = True

    class Config:
        from_attributes = True

# Custom validation
class OrderCreate(BaseModel):
    items: List[dict]
    coupon_code: Optional[str] = None

    @field_validator("items")
    @classmethod
    def validate_items(cls, v):
        if not v:
            raise ValueError("Must have at least one item")
        for item in v:
            if "product_id" not in item or "quantity" not in item:
                raise ValueError("Each item must have product_id and quantity")
            if item["quantity"] < 1:
                raise ValueError("Quantity must be at least 1")
        return v

# Model with computed fields
from pydantic import computed_field

class OrderSummary(BaseModel):
    items: List[dict]
    tax_rate: float = 0.08

    @computed_field
    @property
    def subtotal(self) -> float:
        return sum(item["price"] * item["quantity"] for item in self.items)

    @computed_field
    @property
    def total(self) -> float:
        return self.subtotal * (1 + self.tax_rate)
```

## Async endpoints

### What They Are
Async endpoints are route handlers defined with `async def` instead of `def`. They allow FastAPI to handle requests concurrently using Python's asyncio event loop, without blocking on I/O operations like database queries, HTTP calls, or file reads.

### Why They Are Important
Async endpoints dramatically improve throughput for I/O-bound APIs. While a synchronous endpoint blocks the thread pool for the duration of the request, an async endpoint frees the event loop to handle other requests while awaiting I/O operations. This allows a single process to handle thousands of concurrent connections efficiently.

### Syntax
```python
import asyncio
from fastapi import FastAPI
import httpx

app = FastAPI()

# Simple async endpoint
@app.get("/async-hello")
async def async_hello():
    await asyncio.sleep(0.1)  # Simulate I/O
    return {"message": "Hello asynchronously!"}

# Async with external HTTP call
@app.get("/fetch-data")
async def fetch_data():
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.example.com/data")
        return response.json()

# Concurrent async operations
@app.get("/concurrent")
async def concurrent_fetch():
    async with httpx.AsyncClient() as client:
        urls = [
            "https://api.example.com/users",
            "https://api.example.com/products",
            "https://api.example.com/orders",
        ]
        tasks = [client.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
        return [r.json() for r in responses]
```

### Advanced Examples
```python
from fastapi import FastAPI, BackgroundTasks
import asyncio
from typing import AsyncGenerator

app = FastAPI()

# Background tasks
def write_log(message: str):
    with open("log.txt", "a") as f:
        f.write(f"{message}\n")

@app.post("/send-notification")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(send_email, email)
    background_tasks.add_task(write_log, f"Notification queued for {email}")
    return {"message": "Notification sent"}

# Async generator endpoints (streaming)
@app.get("/stream-numbers")
async def stream_numbers():
    async def generate():
        for i in range(10):
            await asyncio.sleep(0.5)
            yield f"data: {i}\n\n"
    return StreamingResponse(generate(), media_type="text/event-stream")

# Async dependency with cleanup
class DatabaseSession:
    async def __aenter__(self):
        self.conn = await connect_db()
        return self.conn

    async def __aexit__(self, *args):
        await self.conn.close()

async def get_db():
    async with DatabaseSession() as db:
        yield db

@app.get("/users")
async def get_users(db=Depends(get_db)):
    return await db.fetch_all("SELECT * FROM users")

# Mixing sync and async
@app.get("/sync-task")
def sync_task():
    # Runs in thread pool, doesn't block event loop
    import time
    time.sleep(2)
    return {"result": "done"}
```

## OpenAPI docs

### What They Are
FastAPI automatically generates OpenAPI (formerly Swagger) documentation from your code's type hints, Pydantic models, route decorators, and docstrings. This produces two interactive documentation UIs: Swagger UI (at `/docs`) and ReDoc (at `/redoc`), along with the raw OpenAPI schema (at `/openapi.json`).

### Why They Are Important
Auto-generated documentation eliminates the need to manually maintain API docs. The documentation is always in sync with the code, includes request/response schemas, provides "Try it out" functionality for testing endpoints, and enables API client generation tools (openapi-generator, Postman import).

### Configuration and Customization
```python
from fastapi import FastAPI
from fastapi.openapi.utils import get_openapi

app = FastAPI()

# Custom OpenAPI metadata
app = FastAPI(
    title="Task Manager API",
    description="A comprehensive task management system",
    version="2.0.0",
    terms_of_service="http://example.com/terms/",
    contact={
        "name": "API Support",
        "url": "http://example.com/support",
        "email": "support@example.com",
    },
    license_info={
        "name": "MIT",
        "url": "https://opensource.org/licenses/MIT",
    },
)

# Custom OpenAPI schema
def custom_openapi():
    if app.openapi_schema:
        return app.openapi_schema
    openapi_schema = get_openapi(
        title="Custom API",
        version="1.0.0",
        description="This is a custom OpenAPI schema",
        routes=app.routes,
    )
    openapi_schema["info"]["x-logo"] = {
        "url": "https://example.com/logo.png"
    }
    app.openapi_schema = openapi_schema
    return app.openapi_schema

app.openapi = custom_openapi

# Tag descriptions
from fastapi import APIRouter

users_router = APIRouter(prefix="/users", tags=["users"])
tags_metadata = [
    {
        "name": "users",
        "description": "Operations with users. The **login** logic is here.",
    },
    {
        "name": "items",
        "description": "Manage items. So _fancy_.",
    },
]
app = FastAPI(openapi_tags=tags_metadata)
```

### Real-World Use Cases
- **Microservices architecture**: FastAPI services communicating via internal HTTP/gRPC.
- **Machine learning model serving**: Deploying ML models behind a REST API with async inference.
- **Real-time data pipelines**: WebSocket connections for streaming data.
- **SaaS backends**: CRUD APIs with automatic documentation for third-party developers.
- **IoT device APIs**: Handling many concurrent device connections with async endpoints.

### Common Mistakes
- Using `def` for I/O-bound endpoints (blocks the event loop if using async database driver).
- Not using Pydantic's `from_attributes = True` when returning ORM models.
- Forgetting to configure CORS for frontend applications.
- Overlooking request validation errors (FastAPI returns 422 with details automatically).
- Not setting `response_model` on endpoints (leaks internal data structures).
- Using `sync` database drivers (psycopg2) with `async` endpoints (blocks event loop).
- Not implementing proper dependency injection patterns, leading to tight coupling.

### Best Practices
- Always define `response_model` on endpoints for response validation and documentation.
- Use dependency injection for shared logic (database sessions, authentication).
- Keep Pydantic models in separate files (`schemas.py`) for reusability.
- Use `APIRouter` to organize endpoints by domain.
- Configure CORS middleware explicitly for known origins.
- Use `BackgroundTasks` for non-critical async operations (email, logs).
- Write type hints for all function parameters and return values.
- Use `HTTPException` with proper status codes for error responses.

### Performance Considerations
- FastAPI is one of the fastest Python frameworks (on par with Node.js).
- Async endpoints with `asyncpg` or `databases` library maximize throughput.
- Use `@app.on_event("startup")` to initialize connection pools.
- Response serialization with Pydantic is optimized in v2 (Rust core).
- Enable gzip middleware for large responses.
- Use streaming responses for large datasets instead of loading everything in memory.

### Interview Questions
1. How does FastAPI achieve its performance compared to Flask?
2. Explain FastAPI's dependency injection system.
3. How does FastAPI generate OpenAPI documentation automatically?
4. What is the difference between `def` and `async def` endpoints?
5. How do Pydantic models integrate with FastAPI?
6. Explain FastAPI's `Depends()` and how it can be used for authentication.
7. What are the benefits of using `response_model` parameter?

### Coding Challenges
1. **Task Manager API**: Build a FastAPI application with CRUD endpoints for tasks, using Pydantic models, async SQLAlchemy, and automatic OpenAPI docs.
2. **Authentication System**: Implement JWT-based authentication with FastAPI dependencies, login endpoint, and protected routes.
3. **Real-time Chat**: Build a WebSocket chat endpoint using FastAPI with message broadcasting and room management.

### Related Topics
- Starlette (underlying ASGI framework)
- Pydantic v2 (data validation and serialization)
- SQLAlchemy with async (databases, aioSQLAlchemy)
- Uvicorn (ASGI server for running FastAPI)
- Alembic (database migrations)
