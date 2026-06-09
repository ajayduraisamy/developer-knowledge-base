# APIs - Designing REST APIs, status codes, versioning, pagination

## Introduction
Application Programming Interfaces (APIs) define how software components interact. Representational State Transfer (REST) has become the dominant architectural style for web APIs due to its simplicity, scalability, and use of standard HTTP protocol semantics. Designing good REST APIs requires thoughtful decisions about resource modeling, URL structure, HTTP methods, status codes, error formats, versioning strategies, and pagination. A well-designed API is intuitive to use, easy to maintain, and capable of evolving without breaking existing clients. This document covers the fundamental principles and best practices for building robust, developer-friendly REST APIs.

## REST API design

### What It Is
REST (Representational State Transfer) is an architectural style for designing networked applications. RESTful APIs treat server data as resources that can be created, read, updated, and deleted using standard HTTP methods. Each resource is identified by a URL, and representations of resources (typically JSON or XML) are transferred between client and server.

### Why It Is Important
REST APIs are the backbone of modern web and mobile applications. They enable decoupled architectures where frontend and backend can evolve independently. A well-designed REST API reduces integration effort, improves developer experience, and ensures the API can scale to support multiple clients (web, mobile, third-party).

### How It Works Internally
REST relies on the stateless client-server model. Each request from client to server must contain all information needed to understand and process the request (no server-side session state). Resources are identified in requests (usually via URL), and the server manipulates these resources using the received representations. Responses include metadata (status codes, headers) and potentially a representation of the resource. The uniform interface constraint (standard HTTP methods, self-descriptive messages, HATEOAS) distinguishes REST from other API styles.

### Resource Naming
```python
# Good resource naming (nouns, plural, hierarchical)
GET    /users                    # List users
GET    /users/{id}               # Get specific user
POST   /users                    # Create user
PUT    /users/{id}               # Replace user
PATCH  /users/{id}               # Partial update user
DELETE /users/{id}               # Delete user

# Nested resources
GET    /users/{id}/orders                  # User's orders
GET    /users/{id}/orders/{order_id}       # Specific order
GET    /products/{id}/reviews              # Product reviews

# Filtering, sorting, searching
GET    /products?category=electronics&sort=price:desc
GET    /products?search=laptop&in_stock=true

# Actions as sub-resources (not verbs in URL)
POST   /orders/{id}/cancel      # OK (cancel is a sub-resource concept)
POST   /orders/{id}/cancel-order  # Bad (verb)

# Better: use state changes
PATCH  /orders/{id}             {"status": "cancelled"}
```

### Designing Responses
```python
# Consistent response envelope
{
    "success": true,
    "data": {
        "id": 1,
        "name": "Alice",
        "email": "alice@example.com"
    },
    "meta": {
        "request_id": "req-abc123",
        "timestamp": "2024-01-01T12:00:00Z"
    }
}

# Error response format
{
    "success": false,
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Invalid email format",
        "details": [
            {"field": "email", "reason": "must be a valid email address"}
        ]
    },
    "meta": {
        "request_id": "req-def456"
    }
}
```

## HTTP status codes

### What They Are
HTTP status codes are three-digit numbers returned by servers to indicate the result of a request. They are grouped into five classes: 1xx (Informational), 2xx (Success), 3xx (Redirection), 4xx (Client Error), and 5xx (Server Error). Proper use of status codes makes APIs self-documenting and enables clients to handle responses programmatically.

### Why They Are Important
Correct status codes allow clients to implement generic error handling, retry logic, and conditional behavior without parsing response bodies. Misusing status codes (e.g., returning 200 with an error in the body) forces clients to implement error-prone custom parsing for every API call.

### Key Status Codes
```python
# 2xx Success
200 OK                    # Successful GET, PUT, PATCH
201 Created               # Successful POST (new resource created)
202 Accepted              # Request accepted for async processing
204 No Content            # Successful DELETE (or PUT with no body)

# 3xx Redirection
301 Moved Permanently     # Resource has new URL
304 Not Modified           # Cached version still valid (ETag/If-Modified-Since)

# 4xx Client Errors
400 Bad Request           # Malformed request syntax
401 Unauthorized          # Authentication required/missing/invalid
403 Forbidden             # Authenticated but not authorized
404 Not Found             # Resource doesn't exist
405 Method Not Allowed    # Wrong HTTP method for resource
409 Conflict              # Resource state conflict (e.g., duplicate)
422 Unprocessable Entity  # Validation errors (request body invalid)
429 Too Many Requests     # Rate limit exceeded

# 5xx Server Errors
500 Internal Server Error # Generic server error
502 Bad Gateway           # Upstream server error
503 Service Unavailable   # Server overloaded or down
504 Gateway Timeout       # Upstream server timeout
```

### Implementation Example
```python
from flask import Flask, jsonify, request
from werkzeug.exceptions import HTTPException

app = Flask(__name__)

class APIException(Exception):
    def __init__(self, message, status_code=400, details=None):
        self.message = message
        self.status_code = status_code
        self.details = details or []

@app.errorhandler(APIException)
def handle_api_error(error):
    response = {
        "success": False,
        "error": {
            "code": error.__class__.__name__,
            "message": error.message,
            "details": error.details
        }
    }
    return jsonify(response), error.status_code

@app.route("/users", methods=["POST"])
def create_user():
    data = request.get_json()
    if not data or "email" not in data:
        raise APIException("Email is required", 400)
    if "@" not in data["email"]:
        raise APIException(
            "Invalid email",
            status_code=422,
            details=[{"field": "email", "reason": "must contain @"}]
        )
    user = {"id": 1, "email": data["email"], "name": data.get("name")}
    return jsonify({"success": True, "data": user}), 201
```

## API versioning

### What It Is
API versioning allows APIs to evolve and introduce breaking changes without disrupting existing clients. Different clients can continue using older versions while newer clients adopt the latest version. Common strategies include URL path versioning, header versioning, and query parameter versioning.

### Why It Is Important
Without versioning, any breaking change forces all clients to update simultaneously, which is impractical for mobile apps, third-party integrations, and distributed systems. Versioning provides a safety net for backward compatibility.

### Versioning Strategies
```python
# Strategy 1: URL Path Versioning (most common)
# GET /api/v1/users
# GET /api/v2/users

from flask import Blueprint

v1 = Blueprint("api_v1", __name__, url_prefix="/api/v1")
v2 = Blueprint("api_v2", __name__, url_prefix="/api/v2")

@v1.route("/users")
def list_users_v1():
    return jsonify([{"id": 1, "name": "Alice"}])

@v2.route("/users")
def list_users_v2():
    return jsonify([{"id": 1, "name": "Alice", "email": "alice@example.com"}])

# Strategy 2: Header Versioning
# Accept: application/vnd.myapp.v1+json

@app.route("/users")
def list_users():
    version = request.headers.get("Accept", "")
    if "vnd.myapp.v2" in version:
        return jsonify([{"id": 1, "name": "Alice", "email": "alice@example.com"}])
    return jsonify([{"id": 1, "name": "Alice"}])

# Strategy 3: Query Parameter Versioning
# GET /users?version=1
# GET /users?version=2

@app.route("/users")
def list_users():
    version = request.args.get("version", "1")
    if version == "2":
        return jsonify([{"id": 1, "name": "Alice", "email": "alice@example.com"}])
    return jsonify([{"id": 1, "name": "Alice"}])
```

### Version Lifecycle
```python
# Best practices for version management
VERSIONS = {
    "v1": {"deprecated": True, "sunset": "2024-06-01"},
    "v2": {"deprecated": False, "sunset": None},
    "v3": {"deprecated": False, "sunset": None},
}

@app.after_request
def add_version_headers(response):
    version = getattr(request, "api_version", None)
    if version and VERSIONS.get(version, {}).get("deprecated"):
        response.headers["Warning"] = f'299 - "API version {version} is deprecated"'
        response.headers["Sunset"] = VERSIONS[version]["sunset"]
    return response
```

## Pagination

### What It Is
Pagination divides large result sets into manageable pages, reducing server load, network transfer, and client processing time. Without pagination, a single request could return thousands or millions of records, overwhelming both server and client.

### Why It Is Important
Pagination is essential for API performance and usability. It prevents timeouts on large queries, reduces bandwidth usage, and provides a consistent browsing experience. Well-designed pagination also allows clients to request specific pages, prefetch next pages, and estimate total result counts.

### Pagination Strategies
```python
# Strategy 1: Offset/Limit (page-based)
# GET /users?page=1&per_page=20

@app.route("/api/v1/users")
def list_users():
    page = request.args.get("page", 1, type=int)
    per_page = min(request.args.get("per_page", 20, type=int), 100)
    offset = (page - 1) * per_page

    total = get_user_count()
    users = get_users(offset=offset, limit=per_page)
    total_pages = (total + per_page - 1) // per_page

    return jsonify({
        "data": users,
        "pagination": {
            "page": page,
            "per_page": per_page,
            "total": total,
            "total_pages": total_pages,
            "has_next": page < total_pages,
            "has_prev": page > 1,
        }
    })

# Strategy 2: Cursor-based (keyset pagination)
# GET /users?cursor=eyJpZCI6IDUwfQ&limit=20
# Recommended for real-time data, avoids offset drift

import base64
import json

def encode_cursor(last_value):
    return base64.urlsafe_b64encode(json.dumps(last_value).encode()).decode()

def decode_cursor(cursor):
    return json.loads(base64.urlsafe_b64decode(cursor.encode()))

@app.route("/api/v2/users")
def list_users_cursor():
    cursor = request.args.get("cursor")
    limit = min(request.args.get("limit", 20, type=int), 100)

    if cursor:
        last_values = decode_cursor(cursor)
        users = get_users_after(last_values["id"], limit=limit + 1)
    else:
        users = get_users(limit=limit + 1)

    has_more = len(users) > limit
    if has_more:
        users = users[:limit]

    next_cursor = None
    if has_more:
        last_user = users[-1]
        next_cursor = encode_cursor({"id": last_user["id"]})

    return jsonify({
        "data": users,
        "pagination": {
            "next_cursor": next_cursor,
            "has_more": has_more,
        }
    })
```

### Advanced Pagination with Response Headers
```python
from flask import url_for

@app.route("/api/v1/users")
def list_users():
    page = request.args.get("page", 1, type=int)
    per_page = request.args.get("per_page", 20, type=int)
    total = get_user_count()
    total_pages = (total + per_page - 1) // per_page

    users = get_users(page=page, per_page=per_page)

    # Link header for RESTful pagination
    links = []
    if page > 1:
        links.append(f'<{url_for("list_users", page=1, per_page=per_page, _external=True)}>; rel="first"')
        links.append(f'<{url_for("list_users", page=page-1, per_page=per_page, _external=True)}>; rel="prev"')
    if page < total_pages:
        links.append(f'<{url_for("list_users", page=page+1, per_page=per_page, _external=True)}>; rel="next"')
        links.append(f'<{url_for("list_users", page=total_pages, per_page=per_page, _external=True)}>; rel="last"')

    response = jsonify({"data": users, "total": total})
    if links:
        response.headers["Link"] = ", ".join(links)
    return response
```

### Real-World Use Cases
- **Social media APIs**: Twitter, Facebook, Instagram APIs serving feeds, posts, comments.
- **E-commerce APIs**: Product catalogs, order history, inventory listings.
- **SaaS platforms**: Stripe, Twilio, GitHub APIs with versioning and pagination.
- **Mobile backends**: APIs serving mobile apps with limited bandwidth and offline support.

### Common Mistakes
- Returning HTTP 200 for all responses (including errors).
- Not using proper status codes (e.g., 400 for validation, 401 for auth).
- Mixing snake_case and camelCase in responses.
- Not versioning the API from day one (very hard to add later).
- Using offset pagination for real-time data (duplicates/missing records).
- Not rate limiting, allowing abuse and DoS attacks.
- Exposing internal implementation details in responses.

### Best Practices
- Use nouns for resources, verbs for HTTP methods.
- Return consistent JSON envelope structure.
- Use appropriate HTTP status codes consistently.
- Version your API from the start (URL path versioning recommended).
- Use cursor-based pagination for real-time or append-heavy data.
- Include metadata (total count, available pages) in paginated responses.
- Document all error codes, request/response formats (OpenAPI/Swagger).
- Rate limit with meaningful error messages (Retry-After headers).

### Performance Considerations
- Cursor-based pagination is O(1) per page for indexed keys; offset pagination is O(n) for large offsets.
- Use database indexes on pagination columns (created_at, id).
- Limit maximum page size to prevent abuse (e.g., max 100 per page).
- Use ETags and conditional requests (If-None-Match) for caching.
- Implement response compression (gzip) for large payloads.

### Interview Questions
1. What are the constraints of REST architecture?
2. Compare offset-based pagination with cursor-based pagination.
3. What is the difference between 401 Unauthorized and 403 Forbidden?
4. Which HTTP status code would you return for a validation error?
5. How do you version a REST API? What are the trade-offs of different strategies?
6. What is HATEOAS and why is it important for true REST?
7. How would you design an API that supports partial responses (field selection)?

### Coding Challenges
1. **API Client Library**: Build a Python client that handles pagination transparently, automatically fetching all pages for a given endpoint.
2. **Versioned API Server**: Implement a Flask/FastAPI server with two API versions and a deprecation warning header.
3. **Cursor Pagination Implementation**: Build a paginated endpoint using cursor-based pagination with a SQL database that handles new insertions correctly.

### Related Topics
- OpenAPI/Swagger (API documentation)
- GraphQL (alternative to REST for flexible queries)
- gRPC (high-performance RPC framework)
- Rate limiting strategies (token bucket, sliding window)
- API gateway patterns (authentication, throttling, routing)
