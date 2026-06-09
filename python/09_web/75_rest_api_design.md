# REST API Design - HTTP methods, idempotency, HATEOAS, best practices

## Introduction

REST (Representational State Transfer) is an architectural style for designing networked applications, introduced by Roy Fielding in his 2000 PhD dissertation. REST APIs use HTTP protocols and methods to perform CRUD (Create, Read, Update, Delete) operations on resources, which are identified by URLs. REST has become the dominant approach for building web APIs due to its simplicity, scalability, and use of standard HTTP semantics. A well-designed REST API is intuitive, consistent, and easy for clients to consume.

## Why It Is Important

REST API design directly impacts developer experience, system interoperability, and long-term maintainability. Poorly designed APIs lead to confusion, inconsistent client implementations, and costly breaking changes. Good REST API design follows principles like statelessness, resource orientation, proper HTTP method usage, meaningful status codes, consistent naming conventions, and versioning. These principles make APIs predictable, self-documenting, and scalable. As APIs become products themselves (API-as-a-Product), their design quality determines adoption, developer satisfaction, and business success.

## Syntax

### Basic REST Endpoints

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route("/api/users", methods=["GET"])
def list_users():
    return jsonify({"users": []})

@app.route("/api/users/<int:user_id>", methods=["GET"])
def get_user(user_id):
    return jsonify({"id": user_id, "name": "Alice"})

@app.route("/api/users", methods=["POST"])
def create_user():
    data = request.get_json()
    return jsonify({"id": 1, **data}), 201

@app.route("/api/users/<int:user_id>", methods=["PUT"])
def update_user(user_id):
    data = request.get_json()
    return jsonify({"id": user_id, **data})

@app.route("/api/users/<int:user_id>", methods=["DELETE"])
def delete_user(user_id):
    return jsonify({"deleted": user_id}), 204
```

## Examples

### Resource Naming Convention

```
# Good: plural nouns, hierarchical
GET    /api/users
GET    /api/users/123
POST   /api/users
PUT    /api/users/123
DELETE /api/users/123
GET    /api/users/123/orders
GET    /api/users/123/orders/456

# Avoid: verbs in URLs
GET    /api/getUsers
POST   /api/createUser
GET    /api/userOrders?userId=123
```

### Status Code Usage

```python
from flask import jsonify

# 200 OK - Successful GET, PUT, PATCH
@app.route("/api/users/<int:user_id>")
def get_user(user_id):
    return jsonify({"id": user_id, "name": "Alice"}), 200

# 201 Created - Successful POST
@app.route("/api/users", methods=["POST"])
def create_user():
    return jsonify({"id": 1, "name": "Bob"}), 201

# 204 No Content - Successful DELETE
@app.route("/api/users/<int:user_id>", methods=["DELETE"])
def delete_user(user_id):
    return "", 204

# 400 Bad Request - Validation error
@app.route("/api/users", methods=["POST"])
def create_user_bad():
    return jsonify({"error": "Name is required"}), 400

# 404 Not Found
@app.route("/api/users/<int:user_id>")
def get_user_missing(user_id):
    return jsonify({"error": "User not found"}), 404

# 409 Conflict
@app.route("/api/users", methods=["POST"])
def create_user_conflict():
    return jsonify({"error": "Email already exists"}), 409

# 429 Too Many Requests
@app.route("/api/users")
def rate_limited():
    return jsonify({"error": "Rate limit exceeded"}), 429
```

## Beginner Examples

### 1. Simple CRUD API Design

```python
from flask import Flask, jsonify, request, abort

app = Flask(__name__)

users_db = {}
next_id = 1

@app.route("/api/users", methods=["GET"])
def list_users():
    return jsonify({"data": list(users_db.values()), "count": len(users_db)})

@app.route("/api/users/<int:user_id>", methods=["GET"])
def get_user(user_id):
    user = users_db.get(user_id)
    if not user:
        abort(404, description="User not found")
    return jsonify({"data": user})

@app.route("/api/users", methods=["POST"])
def create_user():
    global next_id
    data = request.get_json()
    if not data:
        abort(400, description="Request body is required")
    if "name" not in data:
        abort(400, description="Name is required")
    if "email" not in data:
        abort(400, description="Email is required")

    user = {"id": next_id, "name": data["name"], "email": data["email"]}
    users_db[next_id] = user
    next_id += 1
    return jsonify({"data": user}), 201

@app.route("/api/users/<int:user_id>", methods=["PUT"])
def update_user(user_id):
    user = users_db.get(user_id)
    if not user:
        abort(404, description="User not found")
    data = request.get_json()
    if not data:
        abort(400, description="Request body is required")
    user["name"] = data.get("name", user["name"])
    user["email"] = data.get("email", user["email"])
    return jsonify({"data": user})

@app.route("/api/users/<int:user_id>", methods=["DELETE"])
def delete_user(user_id):
    if user_id not in users_db:
        abort(404, description="User not found")
    del users_db[user_id]
    return jsonify({"message": "User deleted"}), 200

@app.errorhandler(400)
def bad_request(error):
    return jsonify({"error": "bad_request", "message": str(error)}), 400

@app.errorhandler(404)
def not_found(error):
    return jsonify({"error": "not_found", "message": str(error)}), 404
```

### 2. Request and Response Formats

```python
# Consistent JSON envelope
{
    "data": { ... },
    "data": [ ... ],
    "error": {
        "code": "validation_error",
        "message": "Name is required",
        "details": [{"field": "name", "issue": "required"}]
    },
    "meta": {
        "page": 1,
        "per_page": 20,
        "total": 100,
        "total_pages": 5
    }
}

# Paginated collection response
{
    "data": [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}],
    "meta": {
        "page": 1,
        "per_page": 20,
        "total": 50,
        "total_pages": 3
    },
    "links": {
        "self": "/api/users?page=1",
        "next": "/api/users?page=2",
        "prev": null,
        "first": "/api/users?page=1",
        "last": "/api/users?page=3"
    }
}
```

### 3. Query Parameter Filtering

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

products = [
    {"id": 1, "name": "Laptop", "category": "electronics", "price": 999, "in_stock": True},
    {"id": 2, "name": "Shirt", "category": "clothing", "price": 29, "in_stock": True},
    {"id": 3, "name": "Phone", "category": "electronics", "price": 699, "in_stock": False},
]

@app.route("/api/products", methods=["GET"])
def list_products():
    result = products
    category = request.args.get("category")
    if category:
        result = [p for p in result if p["category"] == category]

    min_price = request.args.get("min_price", type=float)
    if min_price is not None:
        result = [p for p in result if p["price"] >= min_price]

    max_price = request.args.get("max_price", type=float)
    if max_price is not None:
        result = [p for p in result if p["price"] <= max_price]

    sort_by = request.args.get("sort_by", "id")
    sort_order = request.args.get("sort_order", "asc")
    if sort_by in ("name", "price", "category"):
        result.sort(key=lambda x: x[sort_by], reverse=(sort_order == "desc"))

    page = request.args.get("page", 1, type=int)
    per_page = min(request.args.get("per_page", 20, type=int), 100)
    start = (page - 1) * per_page
    end = start + per_page
    page_items = result[start:end]

    return jsonify({
        "data": page_items,
        "meta": {
            "page": page,
            "per_page": per_page,
            "total": len(result),
            "total_pages": (len(result) + per_page - 1) // per_page
        }
    })
```

### 4. Idempotency in REST

```python
# GET - Idempotent and safe (no side effects)
@app.route("/api/users/<int:user_id>", methods=["GET"])
def get_user(user_id):
    return jsonify({"data": users_db.get(user_id)})

# PUT - Idempotent (same result if called multiple times)
@app.route("/api/users/<int:user_id>", methods=["PUT"])
def update_user(user_id):
    data = request.get_json()
    users_db[user_id] = {**users_db[user_id], **data}
    users_db[user_id]["id"] = user_id
    return jsonify({"data": users_db[user_id]})

# DELETE - Idempotent (deleting already deleted returns same)
@app.route("/api/users/<int:user_id>", methods=["DELETE"])
def delete_user(user_id):
    users_db.pop(user_id, None)
    return "", 204

# PATCH - Not necessarily idempotent
@app.route("/api/users/<int:user_id>", methods=["PATCH"])
def patch_user(user_id):
    data = request.get_json()
    if "visit_count" in data:
        users_db[user_id]["visit_count"] += data["visit_count"]
    return jsonify({"data": users_db[user_id]})
```

### 5. HATEOAS Links

```python
from flask import Flask, jsonify, url_for

app = Flask(__name__)

users_db = {1: {"id": 1, "name": "Alice", "email": "alice@example.com"}}

@app.route("/api/users/<int:user_id>", methods=["GET"])
def get_user(user_id):
    user = users_db.get(user_id)
    if not user:
        return jsonify({"error": "not_found"}), 404
    user_with_links = {
        **user,
        "_links": {
            "self": {"href": url_for("get_user", user_id=user_id), "method": "GET"},
            "update": {"href": url_for("get_user", user_id=user_id), "method": "PUT"},
            "delete": {"href": url_for("get_user", user_id=user_id), "method": "DELETE"},
            "orders": {"href": f"/api/users/{user_id}/orders", "method": "GET"}
        }
    }
    return jsonify({"data": user_with_links})

@app.route("/api/users", methods=["GET"])
def list_users():
    users = list(users_db.values())
    for user in users:
        user["_links"] = {
            "self": {"href": url_for("get_user", user_id=user["id"]), "method": "GET"}
        }
    return jsonify({
        "data": users,
        "_links": {"create": {"href": url_for("list_users"), "method": "POST"}}
    })
```

## Intermediate Examples

### 1. API Versioning

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

# URL path versioning
@app.route("/api/v1/users/<int:user_id>")
def get_user_v1(user_id):
    return jsonify({"id": user_id, "name": "User V1", "version": 1})

@app.route("/api/v2/users/<int:user_id>")
def get_user_v2(user_id):
    return jsonify({
        "id": user_id, "first_name": "Alice", "last_name": "Smith",
        "full_name": "Alice Smith", "version": 2
    })

# Header versioning
@app.route("/api/users/<int:user_id>")
def get_user_header_version(user_id):
    accept = request.headers.get("Accept", "")
    if "application/vnd.myapp.v2+json" in accept:
        return jsonify({"id": user_id, "name": "Version 2", "version": 2})
    return jsonify({"id": user_id, "name": "Version 1", "version": 1})
```

### 2. Pagination Strategies

```python
from flask import Flask, jsonify, request, url_for
import math

app = Flask(__name__)
items = [{"id": i, "name": f"Item {i}"} for i in range(1, 101)]

@app.route("/api/items", methods=["GET"])
def list_items_page():
    page = request.args.get("page", 1, type=int)
    per_page = min(request.args.get("per_page", 10, type=int), 100)
    total = len(items)
    total_pages = math.ceil(total / per_page)
    start = (page - 1) * per_page
    end = start + per_page
    page_items = items[start:end]

    return jsonify({
        "data": page_items,
        "meta": {"page": page, "per_page": per_page, "total": total, "total_pages": total_pages},
        "links": {
            "self": f"/api/items?page={page}&per_page={per_page}",
            "first": f"/api/items?page=1&per_page={per_page}",
            "last": f"/api/items?page={total_pages}&per_page={per_page}",
            "next": f"/api/items?page={page + 1}&per_page={per_page}" if page < total_pages else None,
            "prev": f"/api/items?page={page - 1}&per_page={per_page}" if page > 1 else None,
        }
    })

@app.route("/api/items/cursor", methods=["GET"])
def list_items_cursor():
    cursor = request.args.get("cursor", None, type=int)
    limit = min(request.args.get("limit", 10, type=int), 100)
    filtered = [i for i in items if i["id"] > cursor] if cursor else items
    page_items = filtered[:limit]
    next_cursor = page_items[-1]["id"] if len(page_items) == limit else None
    return jsonify({
        "data": page_items,
        "meta": {"limit": limit, "next_cursor": next_cursor, "has_more": next_cursor is not None}
    })
```

### 3. Rate Limiting Headers

```python
from flask import Flask, jsonify, request
import time
from functools import wraps

app = Flask(__name__)
rate_limits = {}

def rate_limit(max_requests=100, window_seconds=3600):
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            client_ip = request.remote_addr or "unknown"
            now = time.time()
            if client_ip not in rate_limits:
                rate_limits[client_ip] = []
            rate_limits[client_ip] = [t for t in rate_limits[client_ip] if now - t < window_seconds]

            remaining = max_requests - len(rate_limits[client_ip])
            reset_time = int(now + window_seconds)

            if remaining <= 0:
                response = jsonify({
                    "error": "rate_limit_exceeded",
                    "message": f"Rate limit of {max_requests} per {window_seconds}s exceeded"
                })
                response.status_code = 429
                response.headers["X-RateLimit-Limit"] = str(max_requests)
                response.headers["X-RateLimit-Remaining"] = "0"
                response.headers["X-RateLimit-Reset"] = str(reset_time)
                return response

            rate_limits[client_ip].append(now)
            response = f(*args, **kwargs)
            if isinstance(response, tuple):
                resp, code = response
            else:
                resp, code = response, 200
            if hasattr(resp, "headers"):
                resp.headers["X-RateLimit-Limit"] = str(max_requests)
                resp.headers["X-RateLimit-Remaining"] = str(remaining - 1)
                resp.headers["X-RateLimit-Reset"] = str(reset_time)
            return resp
        return wrapper
    return decorator

@app.route("/api/data")
@rate_limit(max_requests=5, window_seconds=60)
def get_data():
    return jsonify({"data": "Rate limited endpoint"})
```

### 4. Error Response Format (RFC 7807)

```python
from flask import Flask, jsonify

app = Flask(__name__)

class ProblemDetail(Exception):
    def __init__(self, title, status, detail, type_url=None, instance=None, extra=None):
        self.title = title
        self.status = status
        self.detail = detail
        self.type_url = type_url or f"https://api.example.com/errors/{title.lower().replace(' ', '-')}"
        self.instance = instance
        self.extra = extra or {}

    def to_dict(self):
        body = {"type": self.type_url, "title": self.title, "status": self.status, "detail": self.detail}
        if self.instance:
            body["instance"] = self.instance
        body.update(self.extra)
        return body

@app.errorhandler(ProblemDetail)
def handle_problem(error):
    response = jsonify(error.to_dict())
    response.status_code = error.status
    response.headers["Content-Type"] = "application/problem+json"
    return response

@app.route("/api/users/<int:user_id>")
def get_user(user_id):
    if user_id <= 0:
        raise ProblemDetail(
            title="Invalid User ID", status=400,
            detail="User ID must be a positive integer",
            instance=f"/api/users/{user_id}",
            extra={"invalid_value": user_id, "valid_range": "1-99999"}
        )
    if user_id > 100:
        raise ProblemDetail(
            title="User Not Found", status=404,
            detail=f"No user exists with ID {user_id}",
            instance=f"/api/users/{user_id}"
        )
    return jsonify({"data": {"id": user_id, "name": "Alice"}})
```

### 5. Input Validation with Marshmallow

```python
from flask import Flask, jsonify, request
from marshmallow import Schema, fields, validate, ValidationError

app = Flask(__name__)

class UserSchema(Schema):
    name = fields.String(required=True, validate=validate.Length(min=1, max=100))
    email = fields.Email(required=True)
    age = fields.Integer(validate=validate.Range(min=0, max=150))
    role = fields.String(validate=validate.OneOf(["admin", "user", "moderator"]))

user_schema = UserSchema()
users_db = {}

@app.route("/api/users", methods=["POST"])
def create_user():
    try:
        data = user_schema.load(request.get_json())
    except ValidationError as e:
        return jsonify({
            "error": "validation_error",
            "message": "Invalid input",
            "details": e.messages
        }), 422
    user_id = len(users_db) + 1
    users_db[user_id] = {"id": user_id, **data}
    return jsonify({"data": users_db[user_id]}), 201
```

### 6. OpenAPI Specification

```python
from flask import Flask, jsonify
from flask_swagger_ui import get_swaggerui_blueprint

app = Flask(__name__)

SWAGGER_URL = "/docs"
API_URL = "/static/swagger.json"
swagger_ui_blueprint = get_swaggerui_blueprint(SWAGGER_URL, API_URL, config={"app_name": "My API"})
app.register_blueprint(swagger_ui_blueprint, url_prefix=SWAGGER_URL)

@app.route("/static/swagger.json")
def swagger_spec():
    spec = {
        "openapi": "3.0.0",
        "info": {"title": "My API", "description": "A sample REST API", "version": "1.0.0"},
        "servers": [{"url": "http://localhost:5000", "description": "Development"}],
        "paths": {
            "/api/users": {
                "get": {
                    "summary": "List all users",
                    "parameters": [
                        {"name": "page", "in": "query", "schema": {"type": "integer"}},
                        {"name": "per_page", "in": "query", "schema": {"type": "integer"}}
                    ],
                    "responses": {"200": {"description": "List of users"}}
                },
                "post": {
                    "summary": "Create a user",
                    "requestBody": {
                        "content": {"application/json": {"schema": {"$ref": "#/components/schemas/UserInput"}}}
                    },
                    "responses": {"201": {"description": "User created"}, "422": {"description": "Validation error"}}
                }
            }
        },
        "components": {
            "schemas": {
                "User": {
                    "type": "object",
                    "properties": {
                        "id": {"type": "integer"},
                        "name": {"type": "string"},
                        "email": {"type": "string", "format": "email"}
                    }
                },
                "UserInput": {
                    "type": "object",
                    "required": ["name", "email"],
                    "properties": {"name": {"type": "string"}, "email": {"type": "string", "format": "email"}}
                }
            }
        }
    }
    return jsonify(spec)
```

### 7. Bulk Operations

```python
from flask import Flask, jsonify, request

app = Flask(__name__)
items_db = {}

@app.route("/api/items/bulk", methods=["POST"])
def bulk_create():
    data = request.get_json()
    if not isinstance(data, list):
        return jsonify({"error": "Request body must be an array"}), 400
    results = []
    errors = []
    for i, item in enumerate(data):
        if "name" not in item:
            errors.append({"index": i, "error": "Name is required"})
            continue
        item_id = len(items_db) + len(results) + 1
        items_db[item_id] = {"id": item_id, **item}
        results.append(items_db[item_id])
    return jsonify({
        "data": results, "errors": errors,
        "success_count": len(results), "error_count": len(errors)
    }), 200 if results else 400

@app.route("/api/items/bulk", methods=["DELETE"])
def bulk_delete():
    data = request.get_json()
    if not isinstance(data, list):
        return jsonify({"error": "Request body must be an array of IDs"}), 400
    deleted = []
    not_found = []
    for item_id in data:
        if item_id in items_db:
            del items_db[item_id]
            deleted.append(item_id)
        else:
            not_found.append(item_id)
    return jsonify({"deleted": deleted, "not_found": not_found, "deleted_count": len(deleted)})
```

### 8. API Security: JWT Authentication

```python
from flask import Flask, jsonify, request
from functools import wraps
import jwt
import datetime

app = Flask(__name__)
app.config["SECRET_KEY"] = "your-secret-key"

def token_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get("Authorization", "").replace("Bearer ", "")
        if not token:
            return jsonify({"error": "Authentication required"}), 401
        try:
            payload = jwt.decode(token, app.config["SECRET_KEY"], algorithms=["HS256"])
            request.current_user = payload
        except jwt.ExpiredSignatureError:
            return jsonify({"error": "Token expired"}), 401
        except jwt.InvalidTokenError:
            return jsonify({"error": "Invalid token"}), 401
        return f(*args, **kwargs)
    return decorated

@app.route("/api/auth/login", methods=["POST"])
def login():
    data = request.get_json()
    if not data or data.get("username") != "admin" or data.get("password") != "secret":
        return jsonify({"error": "Invalid credentials"}), 401
    token = jwt.encode({
        "sub": data["username"], "role": "admin",
        "exp": datetime.datetime.utcnow() + datetime.timedelta(hours=1)
    }, app.config["SECRET_KEY"], algorithm="HS256")
    return jsonify({"token": token, "token_type": "bearer", "expires_in": 3600})

@app.route("/api/protected")
@token_required
def protected():
    return jsonify({"message": "This is protected", "user": request.current_user})
```

## Advanced Examples

### 1. Full-Featured REST API with FastAPI

```python
from fastapi import FastAPI, HTTPException, Query, Depends
from pydantic import BaseModel, EmailStr, Field
from typing import List, Optional
from datetime import datetime
import enum

app = FastAPI(title="REST API Example", version="2.0.0")

class UserRole(str, enum.Enum):
    ADMIN = "admin"
    USER = "user"
    MODERATOR = "moderator"

class Address(BaseModel):
    street: str
    city: str
    zip_code: str
    country: str

class UserCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    email: EmailStr
    age: int = Field(ge=0, le=150)
    role: UserRole = UserRole.USER
    address: Optional[Address] = None
    tags: List[str] = []

class UserUpdate(BaseModel):
    name: Optional[str] = Field(None, min_length=1, max_length=100)
    email: Optional[EmailStr] = None
    age: Optional[int] = Field(None, ge=0, le=150)
    role: Optional[UserRole] = None
    address: Optional[Address] = None
    tags: Optional[List[str]] = None

class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    age: int
    role: UserRole
    address: Optional[Address]
    tags: List[str]
    created_at: datetime
    updated_at: datetime

class PaginatedResponse(BaseModel):
    data: List[UserResponse]
    meta: dict
    links: dict

users_db = {}
next_id = 1

@app.post("/api/users", response_model=UserResponse, status_code=201)
def create_user(user: UserCreate):
    global next_id
    for u in users_db.values():
        if u["email"] == user.email:
            raise HTTPException(status_code=409, detail="Email already exists")
    now = datetime.utcnow()
    new_user = UserResponse(id=next_id, **user.model_dump(), created_at=now, updated_at=now)
    users_db[next_id] = new_user.model_dump()
    next_id += 1
    return new_user

@app.get("/api/users", response_model=PaginatedResponse)
def list_users(
    page: int = Query(1, ge=1),
    per_page: int = Query(20, ge=1, le=100),
    role: Optional[UserRole] = None,
    search: Optional[str] = None,
    sort_by: str = Query("id", regex="^(id|name|email|age|created_at)$"),
    sort_order: str = Query("asc", regex="^(asc|desc)$")
):
    result = list(users_db.values())
    if role:
        result = [u for u in result if u["role"] == role]
    if search:
        result = [u for u in result if search.lower() in u["name"].lower()]
    reverse = sort_order == "desc"
    result.sort(key=lambda x: x.get(sort_by, ""), reverse=reverse)
    total = len(result)
    total_pages = max(1, (total + per_page - 1) // per_page)
    start = (page - 1) * per_page
    end = start + per_page
    page_items = result[start:end]
    return PaginatedResponse(
        data=[UserResponse(**u) for u in page_items],
        meta={"page": page, "per_page": per_page, "total": total, "total_pages": total_pages},
        links={
            "self": f"/api/users?page={page}&per_page={per_page}",
            "next": f"/api/users?page={page + 1}&per_page={per_page}" if page < total_pages else None,
            "prev": f"/api/users?page={page - 1}&per_page={per_page}" if page > 1 else None
        }
    )

@app.get("/api/users/{user_id}", response_model=UserResponse)
def get_user(user_id: int):
    user = users_db.get(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return UserResponse(**user)

@app.put("/api/users/{user_id}", response_model=UserResponse)
def update_user(user_id: int, user: UserCreate):
    if user_id not in users_db:
        raise HTTPException(status_code=404, detail="User not found")
    now = datetime.utcnow()
    updated = UserResponse(id=user_id, **user.model_dump(), created_at=users_db[user_id]["created_at"], updated_at=now)
    users_db[user_id] = updated.model_dump()
    return updated

@app.patch("/api/users/{user_id}", response_model=UserResponse)
def patch_user(user_id: int, user: UserUpdate):
    if user_id not in users_db:
        raise HTTPException(status_code=404, detail="User not found")
    db_user = users_db[user_id]
    update_data = user.model_dump(exclude_unset=True)
    db_user.update(update_data)
    db_user["updated_at"] = datetime.utcnow()
    users_db[user_id] = db_user
    return UserResponse(**db_user)

@app.delete("/api/users/{user_id}", status_code=204)
def delete_user(user_id: int):
    if user_id not in users_db:
        raise HTTPException(status_code=404, detail="User not found")
    del users_db[user_id]
    return None
```

### 2. API with Idempotency Keys

```python
from fastapi import FastAPI, HTTPException, Header
from pydantic import BaseModel
from typing import Optional
from datetime import datetime
import uuid

app = FastAPI()
idempotency_store = {}

class PaymentRequest(BaseModel):
    amount: float
    currency: str
    description: str

class PaymentResponse(BaseModel):
    id: str
    status: str
    amount: float
    currency: str
    created_at: datetime

@app.post("/api/payments", response_model=PaymentResponse, status_code=201)
def create_payment(payment: PaymentRequest, idempotency_key: Optional[str] = Header(None, alias="Idempotency-Key")):
    if not idempotency_key:
        idempotency_key = str(uuid.uuid4())
    if idempotency_key in idempotency_store:
        return PaymentResponse(**idempotency_store[idempotency_key])
    payment_id = str(uuid.uuid4())
    created_at = datetime.utcnow()
    result = {"id": payment_id, "status": "completed", "amount": payment.amount, "currency": payment.currency, "created_at": created_at}
    idempotency_store[idempotency_key] = result
    return PaymentResponse(**result)
```

### 3. Comprehensive Error Handling

```python
from fastapi import FastAPI, Request, HTTPException
from fastapi.responses import JSONResponse
from pydantic import BaseModel, ValidationError
from typing import Any

app = FastAPI()

class ErrorResponse(BaseModel):
    error: str
    message: str
    status_code: int
    details: Any = None

@app.exception_handler(HTTPException)
async def http_exception_handler(request: Request, exc: HTTPException):
    return JSONResponse(
        status_code=exc.status_code,
        content=ErrorResponse(error="http_error", message=exc.detail, status_code=exc.status_code).model_dump(),
        headers=getattr(exc, "headers", None)
    )

@app.exception_handler(ValidationError)
async def validation_exception_handler(request: Request, exc: ValidationError):
    return JSONResponse(
        status_code=422,
        content=ErrorResponse(error="validation_error", message="Request validation failed", status_code=422, details=exc.errors()).model_dump()
    )

@app.exception_handler(Exception)
async def general_exception_handler(request: Request, exc: Exception):
    return JSONResponse(
        status_code=500,
        content=ErrorResponse(error="internal_error", message="An unexpected error occurred", status_code=500).model_dump()
    )

@app.get("/api/safe")
def safe_endpoint():
    return {"message": "This is safe"}

@app.get("/api/error/400")
def trigger_400():
    raise HTTPException(status_code=400, detail="Bad request")

@app.get("/api/error/404")
def trigger_404():
    raise HTTPException(status_code=404, detail="Not found")

@app.get("/api/error/409")
def trigger_409():
    raise HTTPException(status_code=409, detail="Conflict")

@app.get("/api/error/500")
def trigger_500():
    raise HTTPException(status_code=500, detail="Internal error")
```

## Real-World Use Cases

- **E-commerce APIs**: Product catalogs, shopping carts, checkout, order tracking
- **Social Media APIs**: User profiles, posts, feeds, likes, comments, messaging
- **Financial APIs**: Account management, transactions, payments, statements
- **Healthcare APIs**: Patient records, appointments, prescriptions, lab results
- **IoT APIs**: Device management, telemetry data, commands, firmware updates
- **Content Management APIs**: Articles, media assets, categories, tags, workflows
- **Mapping APIs**: Geocoding, directions, places, spatial queries
- **Communication APIs**: Email, SMS, push notifications, chat
- **Analytics APIs**: Events, metrics, dashboards, reports
- **Identity APIs**: Authentication, authorization, user management, SSO

## Common Mistakes

- **Using verbs in URLs** instead of nouns
- **Inconsistent naming** — mixing camelCase, snake_case, and kebab-case
- **Wrong HTTP methods** — using GET for mutations or POST for reads
- **Not versioning APIs** — breaking clients with incompatible changes
- **Inconsistent error formats** — different error shapes for different endpoints
- **Ignoring idempotency** — creating duplicate resources on retry
- **Poor pagination** — not providing pagination for list endpoints
- **Over-fetching or under-fetching** — too much or too little data
- **No rate limiting** — allowing abusive clients to overwhelm the server
- **Returning stack traces** — exposing internal implementation details

## Best Practices

- Use plural nouns for resources (`/users`, `/orders`)
- Use consistent naming (snake_case for JSON, kebab-case for URLs)
- Use proper HTTP methods: GET for read, POST for create, PUT/PATCH for update, DELETE for delete
- Return appropriate HTTP status codes for every response
- Version your API from day one
- Implement pagination for all list endpoints
- Use consistent error response format (RFC 7807)
- Include HATEOAS links for API discoverability
- Implement rate limiting with informative headers
- Document APIs with OpenAPI/Swagger specification
- Use idempotency keys for mutation endpoints
- Validate all input with schemas (Pydantic, Marshmallow)
- Use nested resources for hierarchical relationships (`/users/123/orders`)
- Keep responses minimal — return only what clients need

## Interview Questions

**Q1: What are the six constraints of REST?**
A: Client-server separation, statelessness, cacheability, uniform interface, layered system, and code on demand (optional).

**Q2: What is the difference between PUT and PATCH?**
A: PUT replaces the entire resource; sending incomplete data might clear unspecified fields. PATCH applies a partial update. PUT is idempotent, PATCH may not be.

**Q3: How do you handle API versioning?**
A: URL path versioning (`/v1/users`), header versioning (`Accept: application/vnd.example.v1+json`), or query parameter versioning (`?version=1`).

**Q4: What is HATEOAS?**
A: HATEOAS means responses include links to related actions and resources, making APIs self-documenting and discoverable.

**Q5: What status code for validation errors?**
A: 400 for simple validation, 422 for semantic validation, 409 for uniqueness violations.

## Coding Challenges

**Challenge 1: Library Management API**
Design a REST API for a library with books, members, and loans. Include check-out/check-in, overdue tracking, and search/filter.

**Challenge 2: E-Commerce Order API**
Create an API with products, carts, orders, and payments. Implement idempotency keys for order creation.

**Challenge 3: Social Media API**
Build a REST API with users, posts, comments, likes, and follows. Include pagination, nested resources, and HATEOAS links.

**Challenge 4: Task Management API**
Design a Trello-like API with boards, lists, cards, and comments. Include sorting, filtering, partial updates.

**Challenge 5: Multi-Tenant SaaS API**
Create a multi-tenant API with organizations, projects, and team members. Implement tenant isolation and RBAC.

## Summary

REST API design is a critical skill for modern web developers. A well-designed REST API follows consistent conventions, uses HTTP semantics correctly, provides meaningful error responses, and scales gracefully. Key principles include resource-oriented URLs, proper HTTP method usage, correct status codes, statelessness, versioning, pagination, rate limiting, and security. Following established best practices results in APIs that are intuitive, maintainable, and delightful for clients to consume.

## Related Topics

- [71. APIs](./71_apis.md)
- [70. HTTP Requests](./70_http_requests.md)
- [73. FastAPI](./73_fastapi.md)
- [72. Flask](./72_flask.md)
- [76. Authentication](./76_authentication.md)
- [74. Web Scraping](./74_web_scraping.md)
