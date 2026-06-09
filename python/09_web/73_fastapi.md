# FastAPI - FastAPI(), @app.get(), Pydantic models, async, OpenAPI

## Introduction

FastAPI is a modern, high-performance web framework for building APIs with Python 3.8+. Created by Sebastián Ramírez, it leverages Python type hints to automatically validate request data, serialize responses, and generate interactive API documentation (OpenAPI + Swagger UI and ReDoc). Built on top of Starlette for async web capabilities and Pydantic for data validation, FastAPI is one of the fastest Python web frameworks available, rivaling Node.js and Go in performance.

## Why It Is Important

FastAPI has rapidly become one of the most popular Python web frameworks due to its unique combination of speed, developer productivity, and automatic documentation. Its use of Python type hints reduces boilerplate code significantly — request validation, serialization, and documentation are all derived from type annotations. Native async support makes it ideal for I/O-bound applications (database calls, HTTP requests, file processing). The auto-generated OpenAPI schema enables seamless frontend integration, client SDK generation, and API testing tools.

## Syntax

### Basic FastAPI Application

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float
    is_offer: bool = False

@app.get("/")
def read_root():
    return {"Hello": "World"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}

@app.post("/items")
def create_item(item: Item):
    return {"name": item.name, "price": item.price}
```

## Examples

### Minimal API

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def root():
    return {"message": "Hello, FastAPI!"}

@app.get("/health")
def health():
    return {"status": "healthy"}
```

### Path and Query Parameters

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
def get_user(user_id: int, include_details: bool = False):
    user = {"id": user_id, "name": f"User {user_id}"}
    if include_details:
        user["details"] = {"email": f"user{user_id}@example.com", "role": "member"}
    return user
```

## Beginner Examples

### 1. Path Parameters with Type Validation

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id, "message": f"Item #{item_id}"}

@app.get("/users/{username}")
def read_user(username: str):
    return {"username": username, "profile": f"Profile of {username}"}

@app.get("/files/{file_path:path}")
def read_file(file_path: str):
    return {"file_path": file_path, "content": f"Contents of {file_path}"}
```

### 2. Query Parameters with Defaults

```python
from fastapi import FastAPI
from typing import Optional

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

@app.get("/items")
def list_items(
    skip: int = 0,
    limit: int = 10,
    search: Optional[str] = None,
    sort_by: str = "name",
    descending: bool = False
):
    items = fake_items_db[skip: skip + limit]
    if search:
        items = [i for i in items if search.lower() in i["item_name"].lower()]
    return {
        "items": items,
        "skip": skip,
        "limit": limit,
        "total": len(fake_items_db),
        "search": search
    }
```

### 3. Request Body with Pydantic

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None

@app.post("/items")
def create_item(item: Item):
    item_dict = item.model_dump()
    if item.tax:
        item_dict["price_with_tax"] = item.price + item.tax
    return item_dict

@app.put("/items/{item_id}")
def update_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.model_dump()}
```

### 4. Response Models

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import List

app = FastAPI()

class ItemBase(BaseModel):
    name: str
    price: float

class ItemCreate(ItemBase):
    pass

class ItemResponse(ItemBase):
    id: int
    tax: float = None

    model_config = {"from_attributes": True}


items_db = {}

@app.post("/items", response_model=ItemResponse, status_code=201)
def create_item(item: ItemCreate):
    item_id = len(items_db) + 1
    db_item = ItemResponse(id=item_id, **item.model_dump())
    items_db[item_id] = db_item
    return db_item

@app.get("/items", response_model=List[ItemResponse])
def list_items():
    return list(items_db.values())

@app.get("/items/{item_id}", response_model=ItemResponse)
def get_item(item_id: int):
    return items_db.get(item_id)
```

### 5. HTTP Status Codes

```python
from fastapi import FastAPI, status
from pydantic import BaseModel

app = FastAPI()

class Message(BaseModel):
    text: str

@app.post("/messages", status_code=status.HTTP_201_CREATED)
def create_message(msg: Message):
    return {"id": 1, "text": msg.text}

@app.delete("/messages/{msg_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_message(msg_id: int):
    return None

@app.get("/error-example")
def error_example():
    from fastapi import HTTPException
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND,
        detail="Item not found",
        headers={"X-Error": "Not Found"}
    )
```

## Intermediate Examples

### 1. Dependency Injection

```python
from fastapi import FastAPI, Depends, HTTPException, Header
from typing import Optional

app = FastAPI()

def common_parameters(q: Optional[str] = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

def verify_token(authorization: Optional[str] = Header(None)):
    if not authorization:
        raise HTTPException(status_code=401, detail="Authorization header missing")
    token = authorization.replace("Bearer ", "")
    if token != "valid-token":
        raise HTTPException(status_code=401, detail="Invalid token")
    return {"token": token, "user": "admin"}

def get_db():
    db = {"connection": "mock-db"}
    try:
        yield db
    finally:
        print("DB connection closed")


@app.get("/items")
def list_items(commons: dict = Depends(common_parameters)):
    return commons

@app.get("/secure-items")
def secure_items(
    auth: dict = Depends(verify_token),
    commons: dict = Depends(common_parameters),
    db: dict = Depends(get_db)
):
    return {
        "user": auth["user"],
        **commons,
        "db_status": db["connection"]
    }
```

### 2. Async Routes

```python
from fastapi import FastAPI
import asyncio
import httpx

app = FastAPI()

async def fetch_post(post_id: int):
    async with httpx.AsyncClient() as client:
        response = await client.get(f"https://jsonplaceholder.typicode.com/posts/{post_id}")
        return response.json()

@app.get("/posts/{post_id}")
async def get_post(post_id: int):
    data = await fetch_post(post_id)
    return data

@app.get("/posts/batch/{count}")
async def get_batch_posts(count: int):
    tasks = [fetch_post(i) for i in range(1, min(count, 20) + 1)]
    results = await asyncio.gather(*tasks)
    return {"count": len(results), "posts": results}

@app.get("/slow")
async def slow_endpoint():
    await asyncio.sleep(2)
    return {"message": "This took 2 seconds"}
```

### 3. Form Data and File Uploads

```python
from fastapi import FastAPI, File, UploadFile, Form
from typing import List
import shutil
import os

app = FastAPI()

@app.post("/upload/single")
async def upload_single(file: UploadFile = File(...)):
    contents = await file.read()
    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "size": len(contents)
    }

@app.post("/upload/multiple")
async def upload_multiple(files: List[UploadFile] = File(...)):
    results = []
    for file in files:
        contents = await file.read()
        results.append({
            "filename": file.filename,
            "size": len(contents)
        })
    return {"files": results, "count": len(results)}

@app.post("/upload/with-metadata")
async def upload_with_metadata(
    file: UploadFile = File(...),
    description: str = Form(...),
    tags: str = Form("general")
):
    return {
        "filename": file.filename,
        "description": description,
        "tags": tags.split(","),
        "content_type": file.content_type
    }
```

### 4. Error Handling

```python
from fastapi import FastAPI, HTTPException, Request
from fastapi.responses import JSONResponse
from fastapi.exception_handlers import http_exception_handler
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

class ErrorResponse(BaseModel):
    error: str
    detail: str
    status_code: int


@app.exception_handler(HTTPException)
async def custom_http_exception_handler(request: Request, exc: HTTPException):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": "http_error",
            "detail": exc.detail,
            "status_code": exc.status_code
        },
        headers=exc.headers
    )

@app.exception_handler(ValueError)
async def value_error_handler(request: Request, exc: ValueError):
    return JSONResponse(
        status_code=400,
        content=ErrorResponse(
            error="validation_error",
            detail=str(exc),
            status_code=400
        ).model_dump()
    )


@app.get("/items/{item_id}")
def get_item(item_id: int):
    if item_id <= 0:
        raise HTTPException(status_code=400, detail="Item ID must be positive")
    if item_id > 100:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"id": item_id, "name": f"Item {item_id}"}

@app.post("/divide")
def divide_numbers(a: int, b: int):
    if b == 0:
        raise ValueError("Division by zero is not allowed")
    return {"result": a / b}
```

### 5. OpenAPI Metadata and Tags

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import List

app = FastAPI(
    title="My API",
    description="A comprehensive API for managing items and users",
    version="2.0.0",
    contact={
        "name": "API Support",
        "email": "support@example.com"
    },
    license_info={
        "name": "MIT",
        "url": "https://opensource.org/licenses/MIT"
    },
    openapi_tags=[
        {"name": "items", "description": "Item management endpoints"},
        {"name": "users", "description": "User management endpoints"},
        {"name": "health", "description": "Health check endpoints"}
    ]
)

class Item(BaseModel):
    name: str
    price: float

class User(BaseModel):
    username: str
    email: str


@app.get("/health", tags=["health"])
def health_check():
    return {"status": "healthy"}

@app.get("/items", tags=["items"], response_model=List[Item])
def list_items():
    return [{"name": "Sample", "price": 9.99}]

@app.post("/items", tags=["items"], status_code=201)
def create_item(item: Item):
    return {"id": 1, **item.model_dump()}

@app.get("/users", tags=["users"])
def list_users():
    return [{"username": "alice", "email": "alice@example.com"}]

@app.post("/users", tags=["users"], status_code=201)
def create_user(user: User):
    return {"id": 1, **user.model_dump()}
```

### 6. CORS Middleware

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://myfrontend.com", "http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE", "PATCH"],
    allow_headers=["Content-Type", "Authorization", "X-Requested-With"],
    expose_headers=["X-Request-ID"],
    max_age=600
)

@app.get("/api/data")
def get_data():
    return {"message": "CORS is configured"}

@app.options("/api/data")
def options_data():
    return {"message": "CORS preflight"}
```

### 7. Background Tasks

```python
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
import time
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI()

def send_welcome_email(email: str, username: str):
    time.sleep(2)
    logger.info(f"Welcome email sent to {email} for user {username}")

def cleanup_temp_files(file_paths: list):
    time.sleep(1)
    for path in file_paths:
        logger.info(f"Cleaned up: {path}")

@app.post("/register")
async def register_user(username: str, email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(send_welcome_email, email, username)
    return {"message": f"User {username} registered. Welcome email will be sent."}

@app.post("/upload")
async def upload_file(background_tasks: BackgroundTasks):
    background_tasks.add_task(cleanup_temp_files, ["/tmp/file1", "/tmp/file2"])
    return {"message": "File uploaded. Cleanup will happen in background."}
```

### 8. WebSocket Support

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from typing import List
import asyncio
import json

app = FastAPI()

class ConnectionManager:
    def __init__(self):
        self.active_connections: List[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def send_personal_message(self, message: str, websocket: WebSocket):
        await websocket.send_text(message)

    async def broadcast(self, message: str):
        for connection in self.active_connections:
            try:
                await connection.send_text(message)
            except Exception:
                self.disconnect(connection)


manager = ConnectionManager()

@app.websocket("/ws/{client_id}")
async def websocket_endpoint(websocket: WebSocket, client_id: str):
    await manager.connect(websocket)
    try:
        await manager.broadcast(json.dumps({
            "type": "join",
            "client_id": client_id,
            "message": f"Client {client_id} joined"
        }))

        while True:
            data = await websocket.receive_text()
            message = json.dumps({
                "type": "message",
                "client_id": client_id,
                "message": data
            })
            await manager.broadcast(message)

    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(json.dumps({
            "type": "leave",
            "client_id": client_id,
            "message": f"Client {client_id} left"
        }))

@app.get("/")
def index():
    return {"message": "WebSocket server running. Connect to /ws/{client_id}"}
```

### 9. Database with SQLAlchemy

```python
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy import create_engine, Column, Integer, String, Float
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from pydantic import BaseModel
from typing import List

SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

class ItemORM(Base):
    __tablename__ = "items"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    price = Column(Float)
    description = Column(String, nullable=True)

Base.metadata.create_all(bind=engine)

class ItemCreate(BaseModel):
    name: str
    price: float
    description: str = None

class ItemResponse(ItemCreate):
    id: int
    model_config = {"from_attributes": True}


app = FastAPI()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


@app.post("/items", response_model=ItemResponse, status_code=201)
def create_item(item: ItemCreate, db: Session = Depends(get_db)):
    db_item = ItemORM(**item.model_dump())
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item

@app.get("/items", response_model=List[ItemResponse])
def list_items(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    items = db.query(ItemORM).offset(skip).limit(limit).all()
    return items

@app.get("/items/{item_id}", response_model=ItemResponse)
def get_item(item_id: int, db: Session = Depends(get_db)):
    item = db.query(ItemORM).filter(ItemORM.id == item_id).first()
    if not item:
        raise HTTPException(status_code=404, detail="Item not found")
    return item

@app.put("/items/{item_id}", response_model=ItemResponse)
def update_item(item_id: int, item: ItemCreate, db: Session = Depends(get_db)):
    db_item = db.query(ItemORM).filter(ItemORM.id == item_id).first()
    if not db_item:
        raise HTTPException(status_code=404, detail="Item not found")
    for key, value in item.model_dump().items():
        setattr(db_item, key, value)
    db.commit()
    db.refresh(db_item)
    return db_item

@app.delete("/items/{item_id}", status_code=204)
def delete_item(item_id: int, db: Session = Depends(get_db)):
    db_item = db.query(ItemORM).filter(ItemORM.id == item_id).first()
    if not db_item:
        raise HTTPException(status_code=404, detail="Item not found")
    db.delete(db_item)
    db.commit()
    return None
```

### 10. Pagination and Filtering

```python
from fastapi import FastAPI, Query
from pydantic import BaseModel
from typing import List, Optional
from datetime import datetime

app = FastAPI()

fake_data = [
    {"id": i, "title": f"Post {i}", "category": "tech" if i % 2 == 0 else "science",
     "created_at": datetime(2024, 1, (i % 28) + 1).isoformat(), "views": i * 10}
    for i in range(1, 101)
]

class PostResponse(BaseModel):
    id: int
    title: str
    category: str
    created_at: str
    views: int

class PaginatedResponse(BaseModel):
    items: List[PostResponse]
    total: int
    page: int
    per_page: int
    total_pages: int
    has_next: bool
    has_prev: bool


@app.get("/posts", response_model=PaginatedResponse)
def list_posts(
    page: int = Query(1, ge=1, description="Page number"),
    per_page: int = Query(10, ge=1, le=100, description="Items per page"),
    category: Optional[str] = Query(None, description="Filter by category"),
    sort_by: str = Query("id", regex="^(id|title|views|created_at)$"),
    sort_order: str = Query("asc", regex="^(asc|desc)$"),
    search: Optional[str] = Query(None, description="Search in title"),
    min_views: Optional[int] = Query(None, ge=0),
):

    filtered = fake_data

    if category:
        filtered = [p for p in filtered if p["category"] == category]
    if search:
        filtered = [p for p in filtered if search.lower() in p["title"].lower()]
    if min_views:
        filtered = [p for p in filtered if p["views"] >= min_views]

    reverse = sort_order == "desc"
    filtered.sort(key=lambda x: x.get(sort_by, 0), reverse=reverse)

    total = len(filtered)
    total_pages = max(1, (total + per_page - 1) // per_page)
    start = (page - 1) * per_page
    end = start + per_page
    page_items = filtered[start:end]

    return PaginatedResponse(
        items=[PostResponse(**p) for p in page_items],
        total=total,
        page=page,
        per_page=per_page,
        total_pages=total_pages,
        has_next=page < total_pages,
        has_prev=page > 1
    )
```

## Advanced Examples

### 1. Full Async CRUD with PostgreSQL

```python
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from sqlalchemy import select, delete
from pydantic import BaseModel
from typing import List, Optional

DATABASE_URL = "postgresql+asyncpg://user:pass@localhost/dbname"
engine = create_async_engine(DATABASE_URL)
async_session = async_sessionmaker(engine, expire_on_commit=False)

class Base(DeclarativeBase):
    pass

class ProductORM(Base):
    __tablename__ = "products"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(index=True)
    price: Mapped[float]
    stock: Mapped[int] = mapped_column(default=0)


class ProductCreate(BaseModel):
    name: str
    price: float
    stock: int = 0

class ProductUpdate(BaseModel):
    name: Optional[str] = None
    price: Optional[float] = None
    stock: Optional[int] = None

class ProductResponse(ProductCreate):
    id: int
    model_config = {"from_attributes": True}


app = FastAPI()

async def get_db():
    async with async_session() as session:
        yield session

@app.on_event("startup")
async def startup():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

@app.on_event("shutdown")
async def shutdown():
    await engine.dispose()


@app.post("/products", response_model=ProductResponse, status_code=201)
async def create_product(product: ProductCreate, db: AsyncSession = Depends(get_db)):
    db_product = ProductORM(**product.model_dump())
    db.add(db_product)
    await db.commit()
    await db.refresh(db_product)
    return db_product

@app.get("/products", response_model=List[ProductResponse])
async def list_products(skip: int = 0, limit: int = 100, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(ProductORM).offset(skip).limit(limit))
    return result.scalars().all()

@app.get("/products/{product_id}", response_model=ProductResponse)
async def get_product(product_id: int, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(ProductORM).where(ProductORM.id == product_id))
    product = result.scalar_one_or_none()
    if not product:
        raise HTTPException(status_code=404, detail="Product not found")
    return product

@app.put("/products/{product_id}", response_model=ProductResponse)
async def update_product(product_id: int, product: ProductUpdate, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(ProductORM).where(ProductORM.id == product_id))
    db_product = result.scalar_one_or_none()
    if not db_product:
        raise HTTPException(status_code=404, detail="Product not found")
    update_data = product.model_dump(exclude_unset=True)
    for key, value in update_data.items():
        setattr(db_product, key, value)
    await db.commit()
    await db.refresh(db_product)
    return db_product

@app.delete("/products/{product_id}", status_code=204)
async def delete_product(product_id: int, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(ProductORM).where(ProductORM.id == product_id))
    product = result.scalar_one_or_none()
    if not product:
        raise HTTPException(status_code=404, detail="Product not found")
    await db.delete(product)
    await db.commit()
    return None
```

### 2. JWT Authentication

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel
from jose import JWTError, jwt
from datetime import datetime, timedelta
from typing import Optional

SECRET_KEY = "your-secret-key-here"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

app = FastAPI()
security = HTTPBearer()

class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: Optional[str] = None

class User(BaseModel):
    username: str
    email: str
    disabled: bool = False

class UserInDB(User):
    hashed_password: str

fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "email": "johndoe@example.com",
        "hashed_password": "$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW",
        "disabled": False
    }
}

def verify_password(plain_password, hashed_password):
    return plain_password == "secret"

def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)
    return None

def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user:
        return False
    if not verify_password(password, user.hashed_password):
        return False
    return user

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=15))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

async def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)):
    token = credentials.credentials
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except JWTError:
        raise credentials_exception
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user

async def get_current_active_user(current_user: User = Depends(get_current_user)):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user


@app.post("/token", response_model=Token)
async def login_for_access_token(username: str, password: str):
    user = authenticate_user(fake_users_db, username, password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/users/me", response_model=User)
async def read_users_me(current_user: User = Depends(get_current_active_user)):
    return current_user

@app.get("/users/me/items")
async def read_own_items(current_user: User = Depends(get_current_active_user)):
    return [{"item_id": 1, "owner": current_user.username}]
```

### 3. Rate Limiting Middleware

```python
from fastapi import FastAPI, Request, HTTPException
from fastapi.responses import JSONResponse
import time
from collections import defaultdict

app = FastAPI()

class RateLimiter:
    def __init__(self):
        self.requests = defaultdict(list)

    async def check_rate_limit(self, request: Request, call_next):
        client_ip = request.client.host if request.client else "unknown"
        now = time.time()
        window = 60
        max_requests = 30

        self.requests[client_ip] = [
            t for t in self.requests[client_ip]
            if now - t < window
        ]

        if len(self.requests[client_ip]) >= max_requests:
            return JSONResponse(
                status_code=429,
                content={
                    "detail": "Rate limit exceeded",
                    "retry_after": window
                },
                headers={
                    "X-RateLimit-Limit": str(max_requests),
                    "X-RateLimit-Remaining": "0",
                    "X-RateLimit-Reset": str(int(now + window))
                }
            )

        self.requests[client_ip].append(now)
        response = await call_next(request)

        remaining = max_requests - len(self.requests[client_ip])
        response.headers["X-RateLimit-Limit"] = str(max_requests)
        response.headers["X-RateLimit-Remaining"] = str(max(remaining, 0))
        response.headers["X-RateLimit-Reset"] = str(int(now + window))

        return response


limiter = RateLimiter()
app.middleware("http")(limiter.check_rate_limit)

@app.get("/")
async def root():
    return {"message": "Rate limited endpoint"}

@app.get("/data")
async def get_data():
    return {"data": "This is also rate limited"}
```

### 4. Comprehensive API with Testing

```python
from fastapi import FastAPI, Depends, HTTPException
from pydantic import BaseModel
from typing import List, Optional
from fastapi.testclient import TestClient

app = FastAPI()

items_db = {}
next_id = 1

class ItemBase(BaseModel):
    name: str
    price: float
    description: Optional[str] = None

class ItemCreate(ItemBase):
    pass

class ItemUpdate(BaseModel):
    name: Optional[str] = None
    price: Optional[float] = None
    description: Optional[str] = None

class ItemResponse(ItemBase):
    id: int


@app.post("/items", response_model=ItemResponse, status_code=201)
def create_item(item: ItemCreate):
    global next_id
    db_item = ItemResponse(id=next_id, **item.model_dump())
    items_db[next_id] = db_item
    next_id += 1
    return db_item

@app.get("/items", response_model=List[ItemResponse])
def list_items(skip: int = 0, limit: int = 100):
    return list(items_db.values())[skip:skip + limit]

@app.get("/items/{item_id}", response_model=ItemResponse)
def get_item(item_id: int):
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    return items_db[item_id]

@app.put("/items/{item_id}", response_model=ItemResponse)
def update_item(item_id: int, item: ItemUpdate):
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    db_item = items_db[item_id]
    update_data = item.model_dump(exclude_unset=True)
    for key, value in update_data.items():
        setattr(db_item, key, value)
    return db_item

@app.delete("/items/{item_id}", status_code=204)
def delete_item(item_id: int):
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    del items_db[item_id]
    return None


client = TestClient(app)

def test_create_item():
    response = client.post("/items", json={"name": "Test", "price": 9.99})
    assert response.status_code == 201
    assert response.json()["name"] == "Test"
    assert response.json()["id"] == 1

def test_list_items():
    response = client.get("/items")
    assert response.status_code == 200
    assert len(response.json()) > 0

def test_get_item():
    response = client.get("/items/1")
    assert response.status_code == 200
    assert response.json()["id"] == 1

def test_get_nonexistent_item():
    response = client.get("/items/999")
    assert response.status_code == 404

def test_update_item():
    response = client.put("/items/1", json={"price": 19.99})
    assert response.status_code == 200
    assert response.json()["price"] == 19.99

def test_delete_item():
    response = client.delete("/items/1")
    assert response.status_code == 204

def test_delete_nonexistent():
    response = client.delete("/items/999")
    assert response.status_code == 404
```

## Real-World Use Cases

- **Microservices Architecture**: High-performance async microservices in distributed systems
- **REST API Backends**: JSON APIs for SPAs, mobile apps, or third-party consumers
- **Real-time Applications**: WebSocket-based chat, notifications, live dashboards
- **Machine Learning APIs**: Serving ML model predictions with low-latency endpoints
- **IoT Data Ingestion**: High-throughput async endpoints for sensor data
- **E-commerce Platforms**: Product catalogs, order management, payment integration
- **SaaS Backends**: Multi-tenant API services with authentication and rate limiting
- **API Gateways**: Proxy and aggregate backend services with unified auth
- **Content Management**: Blog/headless CMS with auto-generated admin UI
- **Developer Tools**: Auto-generated SDKs via OpenAPI client generators

## Common Mistakes

- **Using sync routes for I/O-bound operations** — blocking the event loop
- **Not using Pydantic's `model_dump(exclude_unset=True)`** — overwriting fields with defaults
- **Forgetting `await` in async routes** — returning coroutines instead of results
- **Sharing database sessions across requests** — causing race conditions
- **Not handling WebSocket disconnects** — resource leaks in WebSocket managers
- **Hardcoded secrets in code** — committing tokens and passwords to version control
- **Missing CORS configuration** — blocking legitimate frontend requests
- **Not setting response models** — exposing internal data structures in API responses
- **Over-fetching in ORM queries** — returning entire database rows when only few fields needed
- **Ignoring dependency injection scopes** — creating new objects per request when singleton needed

## Best Practices

- Always use async routes for I/O-bound operations (DB, HTTP, file I/O)
- Leverage Pydantic models with strict type validation for request/response schemas
- Use dependency injection for shared logic (DB sessions, auth, pagination)
- Configure CORS properly for frontend applications
- Implement proper error handling with custom exception handlers
- Use background tasks for non-critical operations (email, logging, cleanup)
- Version your API through URL paths or headers
- Set appropriate response models to control data exposure
- Write comprehensive tests using `TestClient` and pytest
- Generate and serve API documentation through built-in OpenAPI/Swagger

## Interview Questions

**Q1: How does FastAPI achieve high performance?**
A: FastAPI is built on Starlette (async ASGI framework) and Pydantic (data validation). It uses async/await for non-blocking I/O and leverages Python type hints for automatic request validation and serialization with zero overhead.

**Q2: What is Pydantic and why is it central to FastAPI?**
A: Pydantic provides data validation using Python type annotations. FastAPI uses it to automatically validate request bodies, query parameters, and path parameters, and to serialize response data. It generates JSON Schema from Pydantic models.

**Q3: How does dependency injection work in FastAPI?**
A: FastAPI's `Depends()` function allows declaring dependencies in route parameters. Dependencies can be functions, classes, or callables that return values. FastAPI handles injection, caching within a request scope, and cleanup with `yield` dependencies.

**Q4: What is the difference between HTTP and WebSocket in FastAPI?**
A: HTTP uses request-response pattern; WebSocket enables bidirectional, persistent communication. FastAPI supports both through `@app.get/post` for HTTP and `@app.websocket` for WebSocket, with built-in WebSocket disconnect handling.

**Q5: How does FastAPI auto-generate OpenAPI documentation?**
A: FastAPI automatically generates an OpenAPI schema from route definitions, Pydantic models, docstrings, and parameter metadata. It serves interactive docs at /docs (Swagger UI) and /redoc (ReDoc) by default.

## Coding Challenges

**Challenge 1: Book Management API**
Build a FastAPI CRUD API for books with fields (title, author, ISBN, published_year, genre). Implement search, pagination, and filtering by genre/year range.

**Challenge 2: Real-Time Chat Application**
Create a FastAPI WebSocket-based chat room where users can join rooms, send messages, and receive broadcasts. Include join/leave notifications.

**Challenge 3: Task Queue with Background Tasks**
Build an API that accepts CPU-intensive tasks (e.g., image processing), queues them, returns a task ID, and provides a status endpoint. Simulate async processing.

**Challenge 4: Multi-Provider Auth**
Implement authentication that supports both JWT tokens and API keys. Create middleware that checks both auth methods and extracts the user identity.

**Challenge 5: E-Commerce Order System**
Build an order management API with nested Pydantic models (Order → OrderItems), inventory validation, async PostgreSQL persistence, and webhook notification on order completion.

## Summary

FastAPI represents a modern approach to building Python web APIs, combining high performance with exceptional developer productivity. Its use of type hints for automatic validation, serialization, and documentation eliminates boilerplate while improving code correctness and maintainability. Native async support, dependency injection, WebSocket capabilities, and seamless OpenAPI integration make it suitable for everything from simple microservices to complex, high-throughput production systems. Understanding FastAPI's core concepts unlocks the ability to build professional-grade APIs rapidly and reliably.

## Related Topics

- [72. Flask](./72_flask.md)
- [71. APIs](./71_apis.md)
- [75. REST API Design](./75_rest_api_design.md)
- [76. Authentication](./76_authentication.md)
- [70. HTTP Requests](./70_http_requests.md)
- [74. Web Scraping](./74_web_scraping.md)
