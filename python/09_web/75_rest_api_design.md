# REST API Design - HTTP methods, idempotency, HATEOAS, best practices

## Introduction
Representational State Transfer (REST) is an architectural style that defines a set of constraints for creating web services. RESTful APIs use standard HTTP methods to perform operations on resources, which are identified by URLs and represented in formats like JSON or XML. Good REST API design makes APIs intuitive, consistent, and easy to consume. Key principles include proper use of HTTP methods, understanding of idempotency and safety semantics, hypermedia-driven navigation (HATEOAS), consistent error handling, and pragmatic best practices for real-world APIs. This document covers these foundational concepts in depth.

## HTTP methods (GET/POST/PUT/PATCH/DELETE)

### What They Are
HTTP methods (also called verbs) indicate the desired action to be performed on a resource. The most commonly used methods in REST APIs are GET (retrieve), POST (create), PUT (replace), PATCH (partial update), and DELETE (remove). Each has defined semantics regarding safety, idempotency, and cacheability.

### Why They Are Important
Proper use of HTTP methods makes APIs self-describing. A client can understand what an endpoint does just from the HTTP method and URL pattern, without reading documentation. This consistency enables generic HTTP libraries, proxy servers, and caching infrastructure to work correctly.

### How They Work Internally
When a client sends an HTTP request, the method is the first element of the request line (`GET /users HTTP/1.1`). The server routes the request based on the method and path combination. REST APIs map these methods to CRUD operations:

- **GET**: Read a resource or collection. Safe (no side effects), idempotent, cacheable.
- **POST**: Create a new resource. Not safe, not idempotent, not cacheable (by default).
- **PUT**: Replace an entire resource. Not safe, idempotent, not cacheable.
- **PATCH**: Partially modify a resource. Not safe, not idempotent (though can be), not cacheable.
- **DELETE**: Remove a resource. Not safe, idempotent, not cacheable.

### Syntax
```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

# In-memory storage
items = {}
counter = 1

class Item(BaseModel):
    name: str
    price: float
    description: str = None

class ItemUpdate(BaseModel):
    name: str = None
    price: float = None
    description: str = None

# GET: Retrieve collection
@app.get("/items")
def list_items():
    return items

# GET: Retrieve single resource
@app.get("/items/{item_id}")
def get_item(item_id: int):
    if item_id not in items:
        raise HTTPException(404, "Item not found")
    return items[item_id]

# POST: Create new resource
@app.post("/items", status_code=201)
def create_item(item: Item):
    global counter
    items[counter] = item.dict()
    items[counter]["id"] = counter
    counter += 1
    return items[counter - 1]

# PUT: Complete replacement
@app.put("/items/{item_id}")
def replace_item(item_id: int, item: Item):
    if item_id not in items:
        raise HTTPException(404, "Item not found")
    items[item_id] = item.dict()
    items[item_id]["id"] = item_id
    return items[item_id]

# PATCH: Partial update
@app.patch("/items/{item_id}")
def update_item(item_id: int, update: ItemUpdate):
    if item_id not in items:
        raise HTTPException(404, "Item not found")
    current = items[item_id]
    update_data = update.dict(exclude_unset=True)  # Only provided fields
    current.update(update_data)
    return current

# DELETE: Remove resource
@app.delete("/items/{item_id}", status_code=204)
def delete_item(item_id: int):
    if item_id not in items:
        raise HTTPException(404, "Item not found")
    del items[item_id]
    return None
```

### Advanced Examples
```python
from flask import Flask, request, jsonify, abort, make_response
from werkzeug.exceptions import HTTPException

app = Flask(__name__)

# Bulk operations
@app.route("/api/items/bulk", methods=["POST"])
def bulk_create():
    items_data = request.get_json()
    if not isinstance(items_data, list):
        abort(400, "Expected a list of items")
    created = []
    for item_data in items_data:
        item = create_item_in_db(item_data)
        created.append(item)
    return jsonify(created), 201

# Conditional requests (ETags for caching)
@app.route("/api/items/<int:item_id>", methods=["GET"])
def get_item(item_id):
    item = get_item_from_db(item_id)
    etag = compute_etag(item)
    if_none_match = request.headers.get("If-None-Match")
    if if_none_match == etag:
        return "", 304
    response = make_response(jsonify(item))
    response.headers["ETag"] = etag
    return response

@app.route("/api/items/<int:item_id>", methods=["PUT"])
def update_item(item_id):
    if_match = request.headers.get("If-Match")
    current = get_item_from_db(item_id)
    current_etag = compute_etag(current)
    if if_match and if_match != current_etag:
        abort(412, "Precondition failed")
    data = request.get_json()
    new_item = update_item_in_db(item_id, data)
    response = make_response(jsonify(new_item))
    response.headers["ETag"] = compute_etag(new_item)
    return response

# Sub-resources
@app.route("/api/orders/<int:order_id>/items", methods=["GET"])
def list_order_items(order_id):
    return jsonify(get_order_items(order_id))

@app.route("/api/orders/<int:order_id>/items", methods=["POST"])
def add_order_item(order_id):
    data = request.get_json()
    item = add_item_to_order(order_id, data)
    return jsonify(item), 201
```

## Idempotency

### What It Is
Idempotency means that making the same request multiple times produces the same result as making it once (ignoring state changes from concurrent requests). In HTTP semantics, GET, PUT, DELETE are idempotent; POST and PATCH are not (by default).

### Why It Is Important
Idempotency is critical for reliability in distributed systems. When network failures cause retries, idempotent operations can be safely retried without side effects. Payment processing, order creation, and resource updates all benefit from idempotency guarantees.

### How It Works Internally
- **GET**: Reading a resource multiple times returns the same data (assuming no concurrent modifications). Safe and idempotent.
- **PUT**: Replacing a resource with the same data multiple times results in the same final state. The first PUT creates/updates; subsequent PUTs are no-ops.
- **DELETE**: Deleting a resource that no longer exists returns the same result (404). After the first DELETE succeeds, subsequent DELETEs are idempotent (resource is already gone).
- **POST**: Creating a resource typically creates a new resource each time. Not idempotent unless the server implements deduplication (idempotency keys).

```python
from flask import Flask, request, jsonify
import uuid

app = Flask(__name__)

# Idempotent PUT (complete replacement)
class InventoryAPI:
    def put(self, product_id):
        data = request.get_json()
        # Complete replacement - same result every time
        db.execute(
            "INSERT INTO products (id, name, price, quantity) VALUES (%s, %s, %s, %s) "
            "ON CONFLICT (id) DO UPDATE SET name = %s, price = %s, quantity = %s",
            [product_id, data["name"], data["price"], data["quantity"],
             data["name"], data["price"], data["quantity"]]
        )
        return jsonify({"id": product_id}), 200

# Idempotent DELETE
@app.route("/api/users/<user_id>", methods=["DELETE"])
def delete_user(user_id):
    user = find_user(user_id)
    if user:
        delete_user_from_db(user_id)
        return "", 204
    # Even if already deleted, return success for idempotency
    return "", 204

# Idempotency key pattern for POST
import redis
r = redis.Redis()

@app.route("/api/payments", methods=["POST"])
def create_payment():
    idempotency_key = request.headers.get("Idempotency-Key")
    if not idempotency_key:
        return jsonify({"error": "Idempotency-Key required"}), 400

    # Check if already processed
    existing = r.get(f"idempotent:{idempotency_key}")
    if existing:
        return jsonify(existing), 200  # Return stored result

    # Process payment
    payment = process_payment(request.get_json())
    # Store result with TTL to prevent indefinite storage
    r.setex(f"idempotent:{idempotency_key}", 86400, json.dumps(payment))
    return jsonify(payment), 201
```

## HATEOAS

### What It Is
HATEOAS (Hypermedia As The Engine Of Application State) is a constraint of REST that requires the server to guide clients through the API by including hypermedia links in responses. Clients navigate the API by following these links rather than hardcoding URL patterns.

### Why It Is Important
HATEOAS decouples clients from server URL structures. A client discovers available actions dynamically from the response, making the API self-documenting and allowing the server to evolve URLs without breaking clients. While rarely implemented fully in practice, understanding HATEOAS is essential for understanding the REST architectural style.

### How It Works Internally
The server includes a `links` or `_links` object in each response that contains URLs for related resources and available actions. The client examines these links to determine what operations are possible. The server can change URLs, add new capabilities, or deprecate old ones without breaking clients that follow links.

### Syntax
```python
from flask import Flask, jsonify, url_for

app = Flask(__name__)

# HATEOAS-style responses
@app.route("/api/users/<int:user_id>")
def get_user(user_id):
    user = find_user(user_id)
    response = {
        "id": user.id,
        "name": user.name,
        "email": user.email,
        "_links": {
            "self": {"href": url_for("get_user", user_id=user.id, _external=True), "method": "GET"},
            "update": {"href": url_for("update_user", user_id=user.id, _external=True), "method": "PUT"},
            "delete": {"href": url_for("delete_user", user_id=user.id, _external=True), "method": "DELETE"},
            "orders": {"href": url_for("list_user_orders", user_id=user.id, _external=True), "method": "GET"},
        }
    }
    return jsonify(response)

@app.route("/api/users/<int:user_id>/orders")
def list_user_orders(user_id):
    orders = find_orders(user_id)
    response = {
        "orders": [],
        "_links": {
            "self": {"href": url_for("list_user_orders", user_id=user_id, _external=True)},
            "parent": {"href": url_for("get_user", user_id=user_id, _external=True)},
            "create": {"href": url_for("create_order", _external=True), "method": "POST"},
        }
    }
    for order in orders:
        order_data = {
            "id": order.id,
            "total": order.total,
            "status": order.status,
            "_links": {
                "self": {"href": url_for("get_order", order_id=order.id, _external=True)},
                "items": {"href": url_for("list_order_items", order_id=order.id, _external=True)},
                "cancel": {"href": url_for("cancel_order", order_id=order.id, _external=True), "method": "POST"},
            }
        }
        response["orders"].append(order_data)
    return jsonify(response)

# HATEOAS-aware client
def navigate_api(api_root):
    import requests
    response = requests.get(api_root)
    data = response.json()

    # Follow links dynamically
    if "users" in data.get("_links", {}):
        users_url = data["_links"]["users"]["href"]
        users_response = requests.get(users_url)
        for user in users_response.json().get("users", []):
            print(f"User: {user['name']}")
            # Follow "orders" link for each user
            if "orders" in user.get("_links", {}):
                orders = requests.get(user["_links"]["orders"]["href"])
                print(f"  Orders: {orders.json()}")
```

### Advanced HATEOAS Example
```python
from flask import Flask, jsonify, url_for

app = Flask(__name__)

def make_links(*routes):
    links = {}
    for name, endpoint, method in routes:
        links[name] = {
            "href": url_for(endpoint, **{k: v for k, v in kwargs.items() if k != "self"}, _external=True),
            "method": method,
        }
    return links

class HypermediaResponse:
    @staticmethod
    def resource(resource, links):
        return {**resource, "_links": links}

    @staticmethod
    def collection(items, links):
        return {
            "data": items,
            "_links": links,
            "count": len(items),
        }

@app.route("/api")
def api_root():
    return jsonify(HypermediaResponse.collection([], {
        "users": {"href": url_for("list_users", _external=True), "method": "GET"},
        "products": {"href": url_for("list_products", _external=True), "method": "GET"},
        "docs": {"href": "https://docs.example.com/api", "method": "GET"},
    }))

@app.route("/api/users")
def list_users():
    users = get_all_users()
    user_resources = []
    for user in users:
        user_resources.append(HypermediaResponse.resource(
            {"id": user.id, "name": user.name},
            {
                "self": {"href": url_for("get_user", user_id=user.id, _external=True)},
            }
        ))
    return jsonify(HypermediaResponse.collection(user_resources, {
        "self": {"href": url_for("list_users", _external=True)},
        "create": {"href": url_for("create_user", _external=True), "method": "POST"},
    }))
```

## Best practices

### What They Are
REST API best practices are proven conventions and patterns that make APIs intuitive, consistent, reliable, and maintainable. They cover resource naming, status codes, error handling, pagination, versioning, security, and documentation.

### Key Best Practices

**Resource Naming**
```python
# Use nouns, not verbs
# Good
GET /users
GET /users/{id}/orders
POST /products

# Bad
GET /getUsers
GET /users/getOrders
POST /createProduct

# Plural for collections
GET /users       # Not /user
GET /products    # Not /product
```

**Error Handling**
```python
from flask import jsonify

class APIError(Exception):
    def __init__(self, message, code=400, details=None):
        self.message = message
        self.code = code
        self.details = details or []

@app.errorhandler(APIError)
def handle_api_error(error):
    response = {
        "error": {
            "code": error.code,
            "message": error.message,
            "details": error.details,
        },
        "request_id": g.request_id,  # Correlation ID
    }
    return jsonify(response), error.code

# Consistent error formats
@app.route("/api/users", methods=["POST"])
def create_user():
    data = request.get_json()
    errors = []
    if not data.get("email"):
        errors.append({"field": "email", "reason": "Email is required"})
    elif "@" not in data["email"]:
        errors.append({"field": "email", "reason": "Invalid email format"})
    if not data.get("name"):
        errors.append({"field": "name", "reason": "Name is required"})
    if errors:
        raise APIError("Validation failed", 422, errors)
    return jsonify(create_user_in_db(data)), 201
```

**Security**
```python
from flask import request, abort

# Rate limiting
from flask_limiter import Limiter

limiter = Limiter(app, key_func=lambda: request.remote_addr)

@app.route("/api/login", methods=["POST"])
@limiter.limit("5 per minute")
def login():
    pass

# Input validation and sanitization
from marshmallow import Schema, fields, validate

class UserSchema(Schema):
    name = fields.String(required=True, validate=validate.Length(min=1, max=100))
    email = fields.Email(required=True)
    age = fields.Integer(validate=validate.Range(min=0, max=150))

schema = UserSchema()
errors = schema.validate(request.get_json())
if errors:
    abort(422, errors)

# CORS
from flask_cors import CORS
CORS(app, origins=["https://myfrontend.com"], methods=["GET", "POST", "PUT", "DELETE"])
```

**Documentation**
```python
# API documentation with OpenAPI
from flask_restx import Api, Resource, fields

api = Api(app, version="1.0", title="My API", description="A sample REST API")
ns = api.namespace("users", description="User operations")

user_model = api.model("User", {
    "id": fields.Integer(readonly=True),
    "name": fields.String(required=True),
    "email": fields.String(required=True),
})

@ns.route("/")
class UserList(Resource):
    @ns.doc("list_users")
    @ns.marshal_list_with(user_model)
    def get(self):
        return get_all_users()

    @ns.doc("create_user")
    @ns.expect(user_model)
    @ns.marshal_with(user_model, code=201)
    def post(self):
        return create_user(api.payload), 201
```

### Real-World Use Cases
- **Public APIs**: Stripe, Twilio, GitHub, Twitter maintain REST APIs serving millions of requests.
- **Microservices**: Internal service-to-service communication using RESTful interfaces.
- **Mobile backends**: REST APIs powering iOS and Android applications.
- **Third-party integrations**: APIs designed for external developers to build on.

### Common Mistakes
- Using verbs in URLs (`/getUsers`, `/createOrder`).
- Returning 200 for all responses with error in body.
- Not versioning the API from the beginning.
- Using inconsistent naming (mix of camelCase and snake_case).
- Ignoring idempotency for critical operations (payments, orders).
- Exposing internal database structures directly.
- Not implementing pagination for list endpoints.
- Returning unhelpful error messages.

### Performance Considerations
- Use ETags and conditional requests for caching.
- Implement pagination with sensible defaults (20-50 per page).
- Support partial responses (field selection via `?fields=id,name`).
- Use compression (gzip) for responses.
- Batch related resources to reduce round trips.
- Consider GraphQL for flexible querying needs.

### Interview Questions
1. What is the difference between PUT and PATCH?
2. Why is idempotency important in REST APIs?
3. What is HATEOAS and why is it rarely used in practice?
4. How do you version a REST API?
5. What is the correct HTTP status code for a validation error?
6. How would you design an idempotent payment endpoint?
7. What is the difference between 401 Unauthorized and 403 Forbidden?

### Coding Challenges
1. **Idempotent Payment API**: Design and implement a payment endpoint with idempotency key support.
2. **HATEOAS Client**: Build a generic API client that navigates REST APIs by following hypermedia links.
3. **API Version Migrator**: Implement middleware that routes requests to different API handlers based on version header, with deprecation warnings.

### Related Topics
- GraphQL (alternative query language for APIs)
- gRPC (high-performance RPC framework)
- OpenAPI/Swagger (API specification standard)
- API gateways (Kong, AWS API Gateway, Traefik)
- Event-driven APIs (Webhooks, Server-Sent Events)
