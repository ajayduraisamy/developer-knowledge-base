# Flask - Flask(), @app.route(), Jinja2 templates, blueprints

## Introduction
Flask is a lightweight, micro web framework for Python that enables rapid development of web applications and APIs. Its core philosophy is simplicity and flexibility: Flask provides the essential tools for routing, request handling, and templating, while leaving decisions about database, authentication, and other components to the developer. This "micro" approach, combined with comprehensive documentation and a vast ecosystem of extensions, makes Flask ideal for everything from small prototypes to large-scale production applications. Flask leverages Werkzeug (WSGI toolkit) for HTTP handling and Jinja2 for templating, both of which integrate seamlessly into the framework.

## Flask() app

### What It Is
`Flask()` is the application class that serves as the central object for a Flask application. It configures routing, template loading, static file serving, error handling, and extension integration. Creating a Flask application instance is the first step in any Flask project.

### Why It Is Important
The Flask application object is the hub through which all configuration, request handling, and response generation flows. It registers routes, initializes extensions, loads configuration, and runs the development server. Without it, none of Flask's features are accessible.

### How It Works Internally
When instantiated, `Flask(__name__)` creates a WSGI application. The `__name__` parameter helps the framework locate resources (templates, static files) relative to the application module. Internally, Flask maintains a `url_map` (Werkzeug `Map`) that stores all registered routes, a `view_functions` dictionary mapping endpoint names to functions, and a `config` object for application settings. When a request arrives, Flask creates a `Request` object, pushes an application context and request context, dispatches to the matching view function, and wraps the return value into a `Response` object.

### Syntax
```python
from flask import Flask

app = Flask(__name__)

# Configuration
app.config["DEBUG"] = True
app.config["SECRET_KEY"] = "your-secret-key"
app.config["DATABASE_URI"] = "sqlite:///app.db"

# Or load from object
app.config.from_object("config.DevelopmentConfig")
app.config.from_envvar("FLASK_SETTINGS")
app.config.from_pyfile("settings.cfg")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
```

### Beginner Examples
```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "<h1>Hello, Flask!</h1>"

@app.route("/about")
def about():
    return "<p>This is a Flask application.</p>"

if __name__ == "__main__":
    app.run(debug=True)
```

### Intermediate Examples
```python
from flask import Flask, jsonify, request, abort, make_response

app = Flask(__name__)

# Application factory pattern
def create_app(config_name="development"):
    app = Flask(__name__)
    app.config.from_object(f"config.{config_name.capitalize()}Config")

    with app.app_context():
        from .routes import init_routes
        init_routes(app)
        return app

# Error handlers
@app.errorhandler(404)
def not_found(error):
    return jsonify({"error": "Not found", "status": 404}), 404

@app.errorhandler(500)
def server_error(error):
    return jsonify({"error": "Internal server error", "status": 500}), 500

# Before/after request hooks
@app.before_request
def log_request_info():
    app.logger.debug(f"Request: {request.method} {request.path}")

@app.after_request
def add_security_headers(response):
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    return response

# Custom response
@app.route("/api/data")
def get_data():
    data = {"message": "Success", "items": [1, 2, 3]}
    response = make_response(jsonify(data), 200)
    response.headers["X-Custom-Header"] = "value"
    return response
```

## @app.route() decorator

### What It Is
The `@app.route()` decorator binds a URL pattern to a Python function (the view function). When a request matches the URL pattern, Flask calls the associated function and returns its result as the HTTP response.

### Why It Is Important
Routing is the core of any web framework. The `@app.route()` decorator provides a clean, declarative way to define URL patterns, extract URL parameters, specify HTTP methods, and organize application logic by endpoint.

### How It Works Internally
When `@app.route("/users/<int:id>")` is executed, Flask registers the URL rule with Werkzeug's routing system. The `<int:id>` syntax defines a URL variable converter (int, string, uuid, path, float). Werkzeug compiles these rules into a regex and stores them in the `url_map`. When a request arrives, Werkzeug matches the URL against all registered rules, extracts the variable values, and Flask calls the view function with those values as keyword arguments.

### Syntax
```python
from flask import Flask

app = Flask(__name__)

# Basic route
@app.route("/")
def index():
    return "Home Page"

# Route with methods
@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        return "Logging in..."
    return "Login form"

# Route with URL parameters
@app.route("/user/<int:user_id>")
def user_profile(user_id):
    return f"User ID: {user_id}"

# Route with multiple parameters
@app.route("/posts/<int:year>/<slug>")
def post_detail(year, slug):
    return f"Post from {year}: {slug}"

# Same URL pattern, different methods
@app.route("/api/items", methods=["GET"])
def list_items():
    return jsonify({"items": []})

@app.route("/api/items", methods=["POST"])
def create_item():
    return jsonify({"created": True}), 201
```

### Advanced Examples
```python
from flask import Flask, request, jsonify, url_for, redirect

app = Flask(__name__)

# Custom URL converter
from werkzeug.routing import BaseConverter

class RegexConverter(BaseConverter):
    def __init__(self, url_map, *items):
        super().__init__(url_map)
        self.regex = items[0] if items else ".*"

    def to_python(self, value):
        return value

    def to_url(self, value):
        return value

app.url_map.converters["regex"] = RegexConverter

@app.route("/<regex('[a-z]{3}-\\d{4}'):code>")
def match_code(code):
    return f"Matched code: {code}"

# Route with strict slashes
@app.route("/strict/", strict_slashes=True)
def strict():
    return "Trailing slash required"

# Redirect from old URL
@app.route("/old-page")
def old_page():
    return redirect(url_for("new_page"), 301)

@app.route("/new-page")
def new_page():
    return "This is the new page"

# URL building
with app.test_request_context():
    url = url_for("user_profile", user_id=42, _external=True)
    print(url)  # http://localhost:5000/user/42
```

## Jinja2 templates

### What It Is
Jinja2 is a powerful, modern templating engine for Python that Flask integrates as its default template renderer. It allows embedding dynamic content in HTML files using a template language with variables, filters, control structures, template inheritance, and automatic escaping.

### Why It Is Important
Templates separate presentation logic from application logic. Instead of building HTML strings in Python code, developers define templates that are populated with data from view functions. This separation improves maintainability, allows frontend developers to work on templates independently, and provides protection against XSS attacks through auto-escaping.

### Syntax
```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/hello/<name>")
def hello(name):
    return render_template("hello.html", name=name, title="Hello Page")

@app.route("/users")
def users():
    users_list = [
        {"name": "Alice", "email": "alice@example.com"},
        {"name": "Bob", "email": "bob@example.com"},
    ]
    return render_template("users.html", users=users_list)
```

```html
{# templates/hello.html #}
<!DOCTYPE html>
<html>
<head>
    <title>{{ title }}</title>
</head>
<body>
    <h1>Hello, {{ name }}!</h1>
    {% if name == "Admin" %}
        <p>You have special privileges.</p>
    {% endif %}
</body>
</html>
```

```html
{# templates/users.html #}
{% extends "base.html" %}

{% block title %}Users List{% endblock %}

{% block content %}
    <h1>Users</h1>
    <ul>
    {% for user in users %}
        <li>{{ user.name }} - {{ user.email }}</li>
    {% else %}
        <li>No users found</li>
    {% endfor %}
    </ul>
{% endblock %}
```

### Advanced Examples
```python
# Custom Jinja2 filters
from datetime import datetime

@app.template_filter("date_format")
def date_format_filter(date, format="%Y-%m-%d"):
    return date.strftime(format)

@app.template_filter("pluralize")
def pluralize(count, singular="item", plural="items"):
    if count == 1:
        return f"{count} {singular}"
    return f"{count} {plural}"

# Template context processor (available in all templates)
@app.context_processor
def inject_globals():
    return {
        "app_name": "My Flask App",
        "current_year": datetime.now().year,
    }

# Jinja2 macro
# templates/macros.html
{% macro input(name, label="", type="text", value="") %}
    <div class="form-group">
        {% if label %}
            <label for="{{ name }}">{{ label }}</label>
        {% endif %}
        <input type="{{ type }}" name="{{ name }}" id="{{ name }}" value="{{ value }}">
    </div>
{% endmacro %}

# Usage in another template
{% from "macros.html" import input %}
<form>
    {{ input("username", label="Username") }}
    {{ input("password", label="Password", type="password") }}
    <button type="submit">Login</button>
</form>
```

## Blueprints

### What They Are
Blueprints are Flask's mechanism for organizing application code into modular, reusable components. A blueprint defines a collection of routes, templates, static files, and error handlers that can be registered on an application. They are similar to Django's apps or Express routers and are essential for scaling Flask applications beyond simple scripts.

### Why They Are Important
Blueprints prevent the "single-file spaghetti" problem as applications grow. They enable logical separation of features (auth, admin, API, blog), allow teams to work on different modules independently, and support reusable packages that can be shared across projects.

### Syntax
```python
# auth/__init__.py
from flask import Blueprint

auth_bp = Blueprint("auth", __name__, url_prefix="/auth", template_folder="templates")

@auth_bp.route("/login", methods=["GET", "POST"])
def login():
    return render_template("auth/login.html")

@auth_bp.route("/logout")
def logout():
    return "Logged out"

@auth_bp.route("/register", methods=["GET", "POST"])
def register():
    return render_template("auth/register.html")
```

```python
# app.py
from flask import Flask
from auth import auth_bp
from api import api_bp
from admin import admin_bp

app = Flask(__name__)
app.register_blueprint(auth_bp)
app.register_blueprint(api_bp, url_prefix="/api")
app.register_blueprint(admin_bp, url_prefix="/admin")

# Blueprint with subdomain
app.register_blueprint(api_bp, subdomain="api")
```

### Advanced Examples
```python
from flask import Blueprint, jsonify, request
from flask.views import MethodView

# Blueprint with MethodViews (class-based views)
api_bp = Blueprint("api", __name__, url_prefix="/api/v1")

class UserListAPI(MethodView):
    def get(self):
        return jsonify({"users": []})

    def post(self):
        data = request.get_json()
        return jsonify({"created": data}), 201

class UserAPI(MethodView):
    def get(self, user_id):
        return jsonify({"user": {"id": user_id}})

    def put(self, user_id):
        data = request.get_json()
        return jsonify({"updated": data})

    def delete(self, user_id):
        return jsonify({"deleted": user_id}), 204

# Register method views
user_list_view = UserListAPI.as_view("users")
user_view = UserAPI.as_view("user")

api_bp.add_url_rule("/users", view_func=user_list_view)
api_bp.add_url_rule("/users/<int:user_id>", view_func=user_view)

# Blueprint with error handlers
errors_bp = Blueprint("errors", __name__)

@errors_bp.app_errorhandler(404)
def not_found(error):
    return jsonify({"error": "Not found"}), 404

@errors_bp.app_errorhandler(403)
def forbidden(error):
    return jsonify({"error": "Forbidden"}), 403

# Blueprint with before_request
api_bp.before_request(lambda: verify_api_key(request))

# Nested blueprints (using app.register_blueprint with url_prefix)
admin_bp = Blueprint("admin", __name__, url_prefix="/admin")
users_bp = Blueprint("admin_users", __name__, url_prefix="/users")

@users_bp.route("/")
def list_users():
    return "Admin: List Users"

admin_bp.register_blueprint(users_bp)
# Available at: /admin/users/
```

### Real-World Use Cases
- **E-commerce platform**: Blueprints for products, cart, checkout, orders, admin panel.
- **SaaS application**: Blueprints for authentication, billing, dashboard, API, webhooks.
- **Content management system**: Blueprints for blog, pages, media, comments, users.
- **REST API server**: Blueprint per API version or resource domain.

### Common Mistakes
- Putting all routes in a single file without blueprints once the app grows.
- Not using `url_for()` and hardcoding URLs in templates and redirects.
- Modifying global objects (`request`, `session`, `g`) across requests without thread safety.
- Forgetting `SECRET_KEY` configuration (Flask requires it for sessions).
- Not handling CORS when building APIs consumed by frontend applications.
- Overlooking Jinja2's auto-escaping (safe filter only for trusted HTML).
- Using `debug=True` in production (runs the insecure Werkzeug debugger).

### Best Practices
- Use the application factory pattern for testability and configuration management.
- Organize code using blueprints from the start.
- Always use `url_for()` for URL generation instead of hardcoding.
- Keep configuration in environment variables or config files, not in code.
- Use Flask extensions for common patterns: Flask-SQLAlchemy, Flask-Migrate, Flask-Login.
- Implement proper error handlers for 400, 404, 403, 500.
- Use `g` object for request-scoped data (e.g., database connections).
- Write unit tests using Flask's test client.

### Performance Considerations
- Enable template caching in production (`app.config["TEMPLATES_AUTO_RELOAD"] = False`).
- Use a production WSGI server (Gunicorn, uWSGI, Waitress) not the development server.
- Implement database connection pooling.
- Cache expensive computations and database queries.
- Use streaming for large responses.
- Configure static file serving through nginx or CDN in production.

### Interview Questions
1. What is the difference between Flask and Django?
2. Explain the Flask application context and request context.
3. How do blueprints help in scaling Flask applications?
4. What is the purpose of `url_for()` and why should you use it?
5. How does Jinja2 template inheritance work?
6. Explain the request-response cycle in Flask.
7. What are Flask extensions and how do they integrate with the application?

### Coding Challenges
1. **REST API with Blueprints**: Build a Flask REST API with separate blueprints for auth, users, and products, including proper error handling and JSON responses.
2. **Template Rendering**: Create a Jinja2 template for a dashboard with template inheritance, macros, custom filters, and conditional rendering.
3. **Blog Engine**: Implement a minimal blog with Flask using blueprints for posts, comments, and admin, with Jinja2 templates for the frontend.

### Related Topics
- Flask-RESTful (extension for building REST APIs)
- Flask-SQLAlchemy (database integration)
- Flask-Login (session-based authentication)
- Flask-Migrate (Alembic integration)
- Gunicorn and uWSGI (production WSGI servers)
