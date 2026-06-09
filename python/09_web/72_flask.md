# Flask - Flask(), @app.route(), Jinja2 templates, blueprints

## Introduction

Flask is a lightweight, micro web framework for Python that makes it easy to build web applications and REST APIs. Created by Armin Ronacher, it is designed to be minimal and extensible, giving developers the freedom to choose their tools and libraries. Unlike Django, Flask does not enforce a particular project structure or include built-in ORM, admin panel, or authentication — instead, it provides the core essentials (routing, request handling, templating) and lets developers add components as needed via extensions.

## Why It Is Important

Flask's simplicity and flexibility make it one of the most popular Python web frameworks, especially for small to medium-sized applications, microservices, and API backends. Its gentle learning curve makes it ideal for beginners, while its extensibility supports production-scale applications through a rich ecosystem of extensions (Flask-SQLAlchemy, Flask-Login, Flask-Migrate). Understanding Flask is fundamental for Python web developers, as it teaches core web concepts (routing, middleware, request/response cycle, templates) without overwhelming abstraction.

## Syntax

### Basic Flask Application

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route("/")
def home():
    return jsonify({"message": "Hello, World!"})

@app.route("/hello/<name>")
def hello(name):
    return jsonify({"greeting": f"Hello, {name}!"})

if __name__ == "__main__":
    app.run(debug=True)
```

### Route Methods

```python
@app.route("/items", methods=["GET"])
def list_items():
    return jsonify({"items": []})

@app.route("/items", methods=["POST"])
def create_item():
    data = request.json
    return jsonify({"created": data}), 201

@app.route("/items/<int:item_id>", methods=["PUT"])
def update_item(item_id):
    data = request.json
    return jsonify({"updated": item_id, **data})

@app.route("/items/<int:item_id>", methods=["DELETE"])
def delete_item(item_id):
    return jsonify({"deleted": item_id}), 204
```

## Examples

### Minimal Flask App

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def index():
    return "<h1>Welcome to Flask!</h1>"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
```

### JSON API Endpoint

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route("/api/status")
def status():
    return jsonify({"status": "ok", "version": "1.0.0"})

@app.route("/api/echo", methods=["POST"])
def echo():
    data = request.get_json(silent=True)
    return jsonify({"you_sent": data, "method": request.method})
```

## Beginner Examples

### 1. Hello World with Templates

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def home():
    return render_template("index.html", title="Home", message="Welcome to Flask!")

@app.route("/about")
def about():
    return render_template("about.html", title="About")

if __name__ == "__main__":
    app.run(debug=True)
```

Create `templates/index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ title }}</title>
</head>
<body>
    <h1>{{ message }}</h1>
</body>
</html>
```

### 2. URL Parameters

```python
from flask import Flask

app = Flask(__name__)

@app.route("/user/<username>")
def show_user(username):
    return f"<h1>Profile: {username}</h1>"

@app.route("/post/<int:post_id>")
def show_post(post_id):
    return f"<h1>Post #{post_id}</h1>"

@app.route("/path/<path:subpath>")
def show_subpath(subpath):
    return f"<h1>Subpath: {subpath}</h1>"

@app.route("/greet/<name>/<int:age>")
def greet(name, age):
    return f"<h1>Hello {name}, you are {age} years old</h1>"
```

### 3. Query Parameters

```python
from flask import Flask, request

app = Flask(__name__)

@app.route("/search")
def search():
    query = request.args.get("q", "")
    page = request.args.get("page", 1, type=int)
    per_page = request.args.get("per_page", 10, type=int)

    return {
        "query": query,
        "page": page,
        "per_page": per_page,
        "results": [f"Result {i}" for i in range(per_page)]
    }
```

### 4. POST Form Data

```python
from flask import Flask, request

app = Flask(__name__)

@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        username = request.form.get("username")
        password = request.form.get("password")
        if username == "admin" and password == "secret":
            return {"message": "Login successful"}
        return {"message": "Invalid credentials"}, 401
    return """
        <form method="post">
            <input name="username" placeholder="Username">
            <input name="password" type="password" placeholder="Password">
            <button type="submit">Login</button>
        </form>
    """
```

### 5. Static Files

```python
from flask import Flask, url_for

app = Flask(__name__)

@app.route("/")
def home():
    css_url = url_for("static", filename="style.css")
    js_url = url_for("static", filename="app.js")
    return f"""
        <link rel="stylesheet" href="{css_url}">
        <h1>Static Files Demo</h1>
        <script src="{js_url}"></script>
    """
```

Place files in `static/style.css` and `static/app.js`.

## Intermediate Examples

### 1. Blueprints for Modular Apps

```python
from flask import Blueprint, jsonify

# Create a blueprint
users_bp = Blueprint("users", __name__, url_prefix="/users")

@users_bp.route("/")
def list_users():
    return jsonify({"users": ["Alice", "Bob", "Charlie"]})

@users_bp.route("/<int:user_id>")
def get_user(user_id):
    return jsonify({"id": user_id, "name": f"User {user_id}"})

@users_bp.route("/<int:user_id>/posts")
def get_user_posts(user_id):
    return jsonify({"user_id": user_id, "posts": []})
```

```python
# main.py
from flask import Flask
from users import users_bp

app = Flask(__name__)
app.register_blueprint(users_bp)

@app.route("/")
def home():
    return {"message": "Blueprint Example"}

if __name__ == "__main__":
    app.run(debug=True)
```

### 2. Error Handlers

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.errorhandler(404)
def not_found(error):
    return jsonify({
        "error": "not_found",
        "message": "The requested resource was not found"
    }), 404

@app.errorhandler(400)
def bad_request(error):
    return jsonify({
        "error": "bad_request",
        "message": str(error)
    }), 400

@app.errorhandler(500)
def server_error(error):
    return jsonify({
        "error": "internal_error",
        "message": "An unexpected error occurred"
    }), 500

@app.route("/items/<int:item_id>")
def get_item(item_id):
    if item_id <= 0:
        abort(400, description="Item ID must be positive")
    if item_id > 100:
        abort(404)
    return jsonify({"id": item_id, "name": f"Item {item_id}"})
```

### 3. Request Hooks (before_request / after_request)

```python
from flask import Flask, g, request, jsonify
import time
import uuid

app = Flask(__name__)

@app.before_request
def before_request():
    g.start_time = time.time()
    g.request_id = str(uuid.uuid4())
    print(f"[{g.request_id}] {request.method} {request.path}")

@app.after_request
def after_request(response):
    elapsed = time.time() - g.start_time
    response.headers["X-Request-ID"] = g.request_id
    response.headers["X-Response-Time"] = f"{elapsed:.4f}s"
    print(f"[{g.request_id}] Completed {response.status_code} in {elapsed:.4f}s")
    return response

@app.route("/")
def home():
    return jsonify({"message": "Hooks demo", "request_id": g.request_id})

@app.route("/slow")
def slow():
    time.sleep(0.5)
    return jsonify({"message": "This was slow"})
```

### 4. Flask with SQLAlchemy

```python
from flask import Flask, jsonify, request
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///items.db"
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False

db = SQLAlchemy(app)

class Item(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    price = db.Column(db.Float, nullable=False)

    def to_dict(self):
        return {"id": self.id, "name": self.name, "price": self.price}


@app.route("/items", methods=["GET"])
def list_items():
    items = Item.query.all()
    return jsonify([item.to_dict() for item in items])

@app.route("/items", methods=["POST"])
def create_item():
    data = request.get_json()
    item = Item(name=data["name"], price=data["price"])
    db.session.add(item)
    db.session.commit()
    return jsonify(item.to_dict()), 201

@app.route("/items/<int:item_id>", methods=["GET"])
def get_item(item_id):
    item = Item.query.get_or_404(item_id)
    return jsonify(item.to_dict())

@app.route("/items/<int:item_id>", methods=["PUT"])
def update_item(item_id):
    item = Item.query.get_or_404(item_id)
    data = request.get_json()
    item.name = data.get("name", item.name)
    item.price = data.get("price", item.price)
    db.session.commit()
    return jsonify(item.to_dict())

@app.route("/items/<int:item_id>", methods=["DELETE"])
def delete_item(item_id):
    item = Item.query.get_or_404(item_id)
    db.session.delete(item)
    db.session.commit()
    return jsonify({"deleted": item_id}), 204

with app.app_context():
    db.create_all()

if __name__ == "__main__":
    app.run(debug=True)
```

### 5. Flask-Login Authentication

```python
from flask import Flask, jsonify, request, session, redirect, url_for
from functools import wraps

app = Flask(__name__)
app.secret_key = "your-secret-key-here"

users_db = {
    "admin": {"password": "admin123", "role": "admin"},
    "user": {"password": "user123", "role": "user"}
}

def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if "user_id" not in session:
            return jsonify({"error": "Authentication required"}), 401
        return f(*args, **kwargs)
    return decorated_function

@app.route("/login", methods=["POST"])
def login():
    data = request.get_json()
    username = data.get("username")
    password = data.get("password")

    user = users_db.get(username)
    if user and user["password"] == password:
        session["user_id"] = username
        session["role"] = user["role"]
        return jsonify({"message": f"Logged in as {username}"})

    return jsonify({"error": "Invalid credentials"}), 401

@app.route("/logout")
def logout():
    session.clear()
    return jsonify({"message": "Logged out"})

@app.route("/profile")
@login_required
def profile():
    return jsonify({
        "user": session["user_id"],
        "role": session["role"]
    })

@app.route("/admin")
@login_required
def admin():
    if session.get("role") != "admin":
        return jsonify({"error": "Admin access required"}), 403
    return jsonify({"message": "Welcome admin!"})
```

### 6. File Upload Handling

```python
from flask import Flask, request, jsonify
import os

app = Flask(__name__)
app.config["UPLOAD_FOLDER"] = "uploads"
app.config["MAX_CONTENT_LENGTH"] = 16 * 1024 * 1024  # 16 MB
os.makedirs(app.config["UPLOAD_FOLDER"], exist_ok=True)

ALLOWED_EXTENSIONS = {"png", "jpg", "jpeg", "gif", "pdf"}

def allowed_file(filename):
    return "." in filename and filename.rsplit(".", 1)[1].lower() in ALLOWED_EXTENSIONS

@app.route("/upload", methods=["POST"])
def upload_file():
    if "file" not in request.files:
        return jsonify({"error": "No file provided"}), 400

    file = request.files["file"]
    if file.filename == "":
        return jsonify({"error": "No file selected"}), 400

    if not allowed_file(file.filename):
        return jsonify({"error": "File type not allowed"}), 400

    filename = f"{os.urandom(8).hex()}_{file.filename}"
    filepath = os.path.join(app.config["UPLOAD_FOLDER"], filename)
    file.save(filepath)

    return jsonify({
        "message": "File uploaded",
        "filename": filename,
        "size": os.path.getsize(filepath)
    }), 201

@app.route("/files")
def list_files():
    files = os.listdir(app.config["UPLOAD_FOLDER"])
    return jsonify({"files": files})
```

### 7. CORS Support

```python
from flask import Flask, jsonify
from flask_cors import CORS

app = Flask(__name__)

# Allow all origins
CORS(app)

# Or configure specifically
# CORS(app, origins=["https://myapp.com"], methods=["GET", "POST"])

@app.route("/api/data")
def get_data():
    return jsonify({"message": "This is accessible from any origin"})

@app.after_request
def add_cors_headers(response):
    response.headers["Access-Control-Allow-Origin"] = "*"
    response.headers["Access-Control-Allow-Headers"] = "Content-Type,Authorization"
    response.headers["Access-Control-Allow-Methods"] = "GET,POST,PUT,DELETE,OPTIONS"
    return response
```

### 8. Flask CLI Commands

```python
import click
from flask import Flask
from flask.cli import with_appcontext

app = Flask(__name__)

@app.cli.command("hello")
@click.argument("name")
def hello_command(name):
    """Say hello to a user."""
    click.echo(f"Hello {name}!")

@app.cli.command("setup-db")
@with_appcontext
def setup_db():
    """Initialize the database."""
    click.echo("Database created!")

@app.cli.command("seed")
@click.option("--count", default=10, help="Number of items")
@with_appcontext
def seed_db(count):
    """Seed database with sample data."""
    for i in range(count):
        click.echo(f"Created item {i + 1}")
    click.echo(f"Seeded {count} items!")

if __name__ == "__main__":
    app.run()
```

### 9. Configuration Management

```python
import os
from flask import Flask

class Config:
    SECRET_KEY = os.environ.get("SECRET_KEY", "dev-secret-key")
    SQLALCHEMY_DATABASE_URI = os.environ.get("DATABASE_URL", "sqlite:///app.db")
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    DEBUG = False

class DevelopmentConfig(Config):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = "sqlite:///dev.db"

class ProductionConfig(Config):
    DEBUG = False
    SQLALCHEMY_DATABASE_URI = os.environ.get("DATABASE_URL")

class TestingConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = "sqlite:///:memory:"


def create_app(config_class=Config):
    app = Flask(__name__)
    app.config.from_object(config_class)
    return app


app = create_app(DevelopmentConfig if os.environ.get("FLASK_ENV") == "development" else ProductionConfig)

@app.route("/")
def home():
    return {
        "environment": os.environ.get("FLASK_ENV", "production"),
        "debug": app.config["DEBUG"]
    }
```

### 10. Full REST API with CRUD

```python
from flask import Flask, jsonify, request, abort, url_for
from datetime import datetime

app = Flask(__name__)

items_db = {}
next_id = 1

def item_to_dict(item_id, item):
    return {
        "id": item_id,
        "name": item["name"],
        "price": item["price"],
        "created_at": item["created_at"].isoformat(),
        "updated_at": item["updated_at"].isoformat(),
        "_links": {
            "self": url_for("get_item", item_id=item_id),
            "update": url_for("update_item", item_id=item_id),
            "delete": url_for("delete_item", item_id=item_id)
        }
    }

@app.route("/api/items", methods=["GET"])
def list_items():
    page = request.args.get("page", 1, type=int)
    per_page = min(request.args.get("per_page", 10, type=int), 100)
    start = (page - 1) * per_page
    end = start + per_page

    sorted_ids = sorted(items_db.keys())
    page_ids = sorted_ids[start:end]
    items = [item_to_dict(i, items_db[i]) for i in page_ids]

    return jsonify({
        "items": items,
        "page": page,
        "per_page": per_page,
        "total": len(items_db),
        "total_pages": (len(items_db) + per_page - 1) // per_page
    })

@app.route("/api/items", methods=["POST"])
def create_item():
    global next_id
    data = request.get_json()

    if not data or "name" not in data or "price" not in data:
        abort(400, description="Name and price are required")

    if not isinstance(data["price"], (int, float)) or data["price"] <= 0:
        abort(400, description="Price must be a positive number")

    item = {
        "name": data["name"],
        "price": float(data["price"]),
        "created_at": datetime.utcnow(),
        "updated_at": datetime.utcnow()
    }

    item_id = next_id
    next_id += 1
    items_db[item_id] = item

    return jsonify(item_to_dict(item_id, item)), 201

@app.route("/api/items/<int:item_id>", methods=["GET"])
def get_item(item_id):
    item = items_db.get(item_id)
    if not item:
        abort(404)
    return jsonify(item_to_dict(item_id, item))

@app.route("/api/items/<int:item_id>", methods=["PUT"])
def update_item(item_id):
    if item_id not in items_db:
        abort(404)

    data = request.get_json()
    if not data:
        abort(400, description="No data provided")

    item = items_db[item_id]
    if "name" in data:
        item["name"] = data["name"]
    if "price" in data:
        if not isinstance(data["price"], (int, float)) or data["price"] <= 0:
            abort(400, description="Price must be a positive number")
        item["price"] = float(data["price"])

    item["updated_at"] = datetime.utcnow()
    return jsonify(item_to_dict(item_id, item))

@app.route("/api/items/<int:item_id>", methods=["DELETE"])
def delete_item(item_id):
    if item_id not in items_db:
        abort(404)
    del items_db[item_id]
    return jsonify({"deleted": item_id}), 204

@app.errorhandler(400)
def bad_request(error):
    return jsonify({"error": "bad_request", "message": str(error)}), 400

@app.errorhandler(404)
def not_found(error):
    return jsonify({"error": "not_found", "message": "Resource not found"}), 404


if __name__ == "__main__":
    app.run(debug=True)
```

## Advanced Examples

### 1. Application Factory Pattern

```python
from flask import Flask, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
import os

db = SQLAlchemy()
migrate = Migrate()

def create_app(config_name=None):
    app = Flask(__name__)

    if config_name is None:
        config_name = os.environ.get("FLASK_ENV", "development")

    config_map = {
        "development": "config.DevelopmentConfig",
        "production": "config.ProductionConfig",
        "testing": "config.TestingConfig"
    }
    app.config.from_object(config_map.get(config_name, "config.DevelopmentConfig"))

    db.init_app(app)
    migrate.init_app(app, db)

    from routes.api import api_bp
    from routes.auth import auth_bp
    from routes.admin import admin_bp

    app.register_blueprint(api_bp, url_prefix="/api")
    app.register_blueprint(auth_bp, url_prefix="/auth")
    app.register_blueprint(admin_bp, url_prefix="/admin")

    @app.route("/health")
    def health():
        return jsonify({"status": "healthy"})

    return app
```

### 2. WebSocket with Flask-SocketIO

```python
from flask import Flask, render_template
from flask_socketio import SocketIO, emit, join_room, leave_room

app = Flask(__name__)
app.config["SECRET_KEY"] = "secret!"
socketio = SocketIO(app, cors_allowed_origins="*")

@app.route("/")
def index():
    return {"message": "Socket.IO server running"}

@socketio.on("connect")
def handle_connect():
    print(f"Client connected: {request.sid}")
    emit("welcome", {"message": "Welcome to the WebSocket server!"})

@socketio.on("disconnect")
def handle_disconnect():
    print(f"Client disconnected: {request.sid}")

@socketio.on("message")
def handle_message(data):
    print(f"Received: {data}")
    emit("response", {"echo": data}, broadcast=True)

@socketio.on("join")
def on_join(data):
    username = data.get("username", "Anonymous")
    room = data.get("room", "general")
    join_room(room)
    emit("system", {"message": f"{username} joined {room}"}, room=room)

@socketio.on("leave")
def on_leave(data):
    username = data.get("username", "Anonymous")
    room = data.get("room", "general")
    leave_room(room)
    emit("system", {"message": f"{username} left {room}"}, room=room)

@socketio.on("private_message")
def private_message(data):
    target_sid = data.get("to")
    message = data.get("message")
    emit("private", {"from": request.sid, "message": message}, to=target_sid)

if __name__ == "__main__":
    socketio.run(app, debug=True)
```

### 3. Rate Limiting Middleware

```python
from flask import Flask, jsonify, request, g
import time
from collections import defaultdict

app = Flask(__name__)

class RateLimiter:
    def __init__(self, default_limit=100, window_seconds=60):
        self.limits = defaultdict(lambda: default_limit)
        self.window = window_seconds
        self.requests = defaultdict(list)

    def set_limit(self, endpoint, limit):
        self.limits[endpoint] = limit

    def is_allowed(self, key, endpoint):
        now = time.time()
        window_start = now - self.window

        self.requests[key] = [
            t for t in self.requests[key]
            if t > window_start
        ]

        limit = self.limits.get(endpoint, self.limits.default_factory())

        if len(self.requests[key]) >= limit:
            return False, limit, len(self.requests[key])

        self.requests[key].append(now)
        return True, limit, len(self.requests[key]) + 1


rate_limiter = RateLimiter(default_limit=10, window_seconds=60)

def limit_by_ip(endpoint=None):
    def decorator(f):
        def wrapper(*args, **kwargs):
            endpoint_key = endpoint or request.endpoint or request.path
            client_ip = request.remote_addr or "unknown"

            allowed, limit, current = rate_limiter.is_allowed(client_ip, endpoint_key)
            if not allowed:
                return jsonify({
                    "error": "rate_limit_exceeded",
                    "message": f"Rate limit of {limit} requests per {rate_limiter.window}s exceeded",
                    "retry_after": rate_limiter.window
                }), 429

            return f(*args, **kwargs)
        wrapper.__name__ = f.__name__
        return wrapper
    return decorator


@app.route("/api/data")
@limit_by_ip()
def get_data():
    return jsonify({"data": "This endpoint is rate limited"})

@app.route("/api/important")
@limit_by_ip(endpoint="important")
def important_data():
    return jsonify({"data": "Important data"})
```

### 4. Custom Middleware with WSGI

```python
from flask import Flask, jsonify
import time

class RequestLoggingMiddleware:
    def __init__(self, app):
        self.app = app

    def __call__(self, environ, start_response):
        method = environ.get("REQUEST_METHOD", "")
        path = environ.get("PATH_INFO", "")
        start = time.time()

        def custom_start_response(status, headers, *args):
            elapsed = time.time() - start
            print(f"[{method}] {path} -> {status} ({elapsed:.3f}s)")
            return start_response(status, headers, *args)

        return self.app(environ, custom_start_response)


class SecurityHeadersMiddleware:
    def __init__(self, app):
        self.app = app

    def __call__(self, environ, start_response):
        def custom_start_response(status, headers, *args):
            security_headers = [
                ("X-Content-Type-Options", "nosniff"),
                ("X-Frame-Options", "DENY"),
                ("X-XSS-Protection", "1; mode=block"),
                ("Strict-Transport-Security", "max-age=31536000"),
            ]
            for header in security_headers:
                headers.append(header)
            return start_response(status, headers, *args)

        return self.app(environ, custom_start_response)


app = Flask(__name__)
app.wsgi_app = RequestLoggingMiddleware(app.wsgi_app)
app.wsgi_app = SecurityHeadersMiddleware(app.wsgi_app)

@app.route("/")
def home():
    return jsonify({"message": "Middleware demo"})
```

### 5. Full Application with Testing

```python
from flask import Flask, jsonify, request, abort
import pytest

app = Flask(__name__)

items = {}

@app.route("/api/items", methods=["GET"])
def list_items():
    return jsonify(list(items.values()))

@app.route("/api/items/<int:item_id>", methods=["GET"])
def get_item(item_id):
    if item_id not in items:
        abort(404)
    return jsonify(items[item_id])

@app.route("/api/items", methods=["POST"])
def create_item():
    data = request.get_json()
    if not data or "name" not in data:
        abort(400, description="Name is required")
    item_id = max(items.keys()) + 1 if items else 1
    items[item_id] = {"id": item_id, "name": data["name"]}
    return jsonify(items[item_id]), 201

@app.route("/api/items/<int:item_id>", methods=["DELETE"])
def delete_item(item_id):
    if item_id not in items:
        abort(404)
    del items[item_id]
    return "", 204


if __name__ == "__main__":
    app.run(debug=True)

# Tests
@pytest.fixture
def client():
    app.config["TESTING"] = True
    with app.test_client() as client:
        items.clear()
        yield client

def test_list_items_empty(client):
    response = client.get("/api/items")
    assert response.status_code == 200
    assert response.json == []

def test_create_item(client):
    response = client.post("/api/items", json={"name": "Test Item"})
    assert response.status_code == 201
    assert response.json["name"] == "Test Item"
    assert "id" in response.json

def test_get_item(client):
    client.post("/api/items", json={"name": "Item 1"})
    response = client.get("/api/items/1")
    assert response.status_code == 200
    assert response.json["name"] == "Item 1"

def test_get_nonexistent_item(client):
    response = client.get("/api/items/999")
    assert response.status_code == 404

def test_delete_item(client):
    client.post("/api/items", json={"name": "Item 1"})
    response = client.delete("/api/items/1")
    assert response.status_code == 204
    response = client.get("/api/items")
    assert response.json == []
```

## Real-World Use Cases

- **REST API Backends**: Building JSON APIs for single-page apps or mobile clients
- **Microservices**: Lightweight service endpoints in a microservice architecture
- **Prototyping**: Rapidly building and testing web application ideas
- **Dashboard Applications**: Internal admin dashboards with Jinja2 templates
- **File Servers**: Upload/download services with authentication
- **Webhook Receivers**: Endpoints for receiving webhook events from third parties
- **Blog Engines**: Simple CMS with Markdown support
- **E-commerce Backends**: Product catalogs, cart management, order processing
- **IoT Data Collectors**: Receiving sensor data from IoT devices
- **Authentication Services**: OAuth providers, SSO endpoints

## Common Mistakes

- **Using debug mode in production** — exposes debugger and allows arbitrary code execution
- **Not using application factory** — making testing and scaling difficult
- **Hardcoding configuration** — storing secrets in source code
- **Ignoring request validation** — trusting user input without sanitization
- **Forgetting to close database sessions** — causing connection leaks
- **Not using blueprints** — creating monolithic route files
- **Overusing global state** — creating thread-safety issues
- **Skipping error handlers** — returning raw tracebacks to users
- **Not setting `SECRET_KEY`** — breaking session security
- **Ignoring CORS for APIs** — blocking legitimate frontend requests

## Best Practices

- Use the application factory pattern for testability and flexibility
- Organize routes with Blueprints for modularity
- Always validate and sanitize user input
- Store configuration in environment variables
- Use Flask extensions for common patterns (SQLAlchemy, Migrate, Login)
- Implement proper error handling with custom error pages
- Add rate limiting for public endpoints
- Use `g` for request-scoped data
- Write tests using `app.test_client()`
- Follow a consistent project structure (models, routes, services, templates)

## Interview Questions

**Q1: What is the difference between Flask and Django?**
A: Flask is a micro-framework with minimal built-in features, giving developers freedom to choose components. Django is a full-stack framework with ORM, admin, auth, and more included. Flask is simpler and more flexible; Django is more opinionated and batteries-included.

**Q2: What is a Flask Blueprint?**
A: A Blueprint is a way to organize related routes, templates, and static files into modular components. Blueprints can be registered on an app with URL prefixes, making the application more maintainable.

**Q3: How does Flask handle request lifecycle?**
A: Flask processes requests through `before_request` hooks, then the view function, then `after_request` hooks, and finally `teardown_request`. Middleware can also be added at the WSGI level.

**Q4: What is the `g` object in Flask?**
A: `g` is a thread-local object that stores data during a single request. It is reset after each request and is commonly used to store database connections, user objects, or request metadata.

**Q5: How do you create a REST API with Flask?**
A: Use `@app.route()` with methods, return JSON with `jsonify()`, access request data with `request.json` or `request.args`, use appropriate status codes, and implement proper error handling.

## Coding Challenges

**Challenge 1: To-Do List API**
Build a Flask REST API for a to-do list with endpoints: GET /todos, POST /todos, PUT /todos/<id>, DELETE /todos/<id>. Store items in memory and add input validation.

**Challenge 2: URL Shortener**
Create a Flask app that accepts a long URL via POST /shorten, generates a short code, and redirects GET /<code> to the original URL.

**Challenge 3: File Upload Service**
Build a Flask service that accepts file uploads, validates file types and sizes, stores files with unique names, and provides download links.

**Challenge 4: Blog with Authentication**
Build a Flask blog with user registration/login, CRUD for posts, comment functionality, and role-based access (admin vs regular user).

**Challenge 5: Rate-Limited API Gateway**
Create a Flask app that proxies requests to multiple backend APIs, applies per-IP rate limiting, logs all requests, and returns aggregated responses.

## Summary

Flask is a powerful, lightweight web framework that excels at building web applications and APIs with minimal boilerplate. Its simplicity, extensive extension ecosystem, and flexibility make it ideal for projects from small prototypes to production microservices. By mastering Flask's core concepts — routing, request handling, templating, blueprints, error handling, and middleware — developers can build maintainable, scalable web services while maintaining full control over their architecture.

## Related Topics

- [73. FastAPI](./73_fastapi.md)
- [71. APIs](./71_apis.md)
- [70. HTTP Requests](./70_http_requests.md)
- [75. REST API Design](./75_rest_api_design.md)
- [76. Authentication](./76_authentication.md)
- [74. Web Scraping](./74_web_scraping.md)
