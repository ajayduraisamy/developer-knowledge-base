# MVC - Model-View-Controller, separation of concerns, MVVM

## Introduction

The Model-View-Controller (MVC) pattern separates an application into three interconnected components: the Model (data and business logic), the View (user interface), and the Controller (handles input and coordinates). This separation enables modular development, parallel work on different components, and maintainable codebases. Variants include Model-View-ViewModel (MVVM) and Model-View-Template (MTV).

## Why It Is Important

MVC is the foundation of most web frameworks and GUI applications. It enforces separation of concerns, making code more testable, maintainable, and reusable. Changes to the user interface do not affect business logic, and data models can be modified without rewriting views. This pattern scales well from small applications to large enterprise systems and is the standard architecture for frameworks like Django, Rails, Spring, and ASP.NET.

## Syntax

The Model manages data and rules. The View displays data to the user. The Controller handles user input and updates the Model. In web frameworks, the Controller receives HTTP requests, interacts with the Model, and selects a View to render. In MVVM, the ViewModel acts as a specialized controller that exposes data and commands for data binding.

## Examples

```python
# Basic MVC implementation
from typing import List, Dict, Any, Optional
import json


class Model:
    def __init__(self):
        self._data: Dict[str, Any] = {}
        self._observers: List[callable] = []

    def add_observer(self, observer: callable):
        self._observers.append(observer)

    def notify_observers(self, event: str, data: Any = None):
        for observer in self._observers:
            observer(event, data)

    def get(self, key: str):
        return self._data.get(key)

    def set(self, key: str, value: Any):
        self._data[key] = value
        self.notify_observers("data_changed", {key: value})

    def delete(self, key: str):
        if key in self._data:
            del self._data[key]
            self.notify_observers("data_deleted", key)


class View:
    def render(self, data: Any) -> str:
        raise NotImplementedError


class TextView(View):
    def render(self, data: Any) -> str:
        if isinstance(data, dict):
            return "\n".join(f"{k}: {v}" for k, v in data.items())
        return str(data)


class JSONView(View):
    def render(self, data: Any) -> str:
        return json.dumps(data, indent=2)


class HTMLView(View):
    def render(self, data: Any) -> str:
        if isinstance(data, dict):
            items = "".join(f"<li><b>{k}:</b> {v}</li>" for k, v in data.items())
            return f"<ul>{items}</ul>"
        return f"<p>{data}</p>"


class Controller:
    def __init__(self, model: Model, view: View):
        self._model = model
        self._view = view
        self._model.add_observer(self._on_model_change)

    def _on_model_change(self, event: str, data: Any):
        print(f"[Event] {event}: {data}")

    def set_data(self, key: str, value: Any):
        self._model.set(key, value)

    def get_data(self, key: str) -> Any:
        return self._model.get(key)

    def delete_data(self, key: str):
        self._model.delete(key)

    def display(self):
        return self._view.render(self._model._data)


model = Model()
view = HTMLView()
controller = Controller(model, view)

controller.set_data("name", "Alice")
controller.set_data("age", 30)
print(controller.display())

controller = Controller(model, JSONView())
print(controller.display())
```

```python
# User management MVC
class UserModel:
    def __init__(self):
        self._users: Dict[int, dict] = {}
        self._next_id = 1

    def create(self, name: str, email: str) -> dict:
        user = {"id": self._next_id, "name": name, "email": email}
        self._users[self._next_id] = user
        self._next_id += 1
        return user

    def get(self, user_id: int) -> Optional[dict]:
        return self._users.get(user_id)

    def get_all(self) -> List[dict]:
        return list(self._users.values())

    def update(self, user_id: int, **kwargs) -> Optional[dict]:
        user = self._users.get(user_id)
        if user:
            user.update(kwargs)
        return user

    def delete(self, user_id: int) -> bool:
        return self._users.pop(user_id, None) is not None


class UserView:
    def render_user(self, user: dict) -> str:
        return f"ID: {user['id']}, Name: {user['name']}, Email: {user['email']}"

    def render_user_list(self, users: List[dict]) -> str:
        if not users:
            return "No users found"
        return "\n".join(self.render_user(u) for u in users)

    def render_error(self, message: str) -> str:
        return f"Error: {message}"


class UserController:
    def __init__(self, model: UserModel, view: UserView):
        self._model = model
        self._view = view

    def create_user(self, name: str, email: str) -> str:
        if not name or not email:
            return self._view.render_error("Name and email required")
        if "@" not in email:
            return self._view.render_error("Invalid email")
        user = self._model.create(name, email)
        return self._view.render_user(user)

    def get_user(self, user_id: int) -> str:
        user = self._model.get(user_id)
        if user is None:
            return self._view.render_error("User not found")
        return self._view.render_user(user)

    def list_users(self) -> str:
        users = self._model.get_all()
        return self._view.render_user_list(users)

    def update_user(self, user_id: int, **kwargs) -> str:
        user = self._model.update(user_id, **kwargs)
        if user is None:
            return self._view.render_error("User not found")
        return self._view.render_user(user)

    def delete_user(self, user_id: int) -> str:
        if self._model.delete(user_id):
            return f"User {user_id} deleted"
        return self._view.render_error("User not found")


ctrl = UserController(UserModel(), UserView())
print(ctrl.create_user("Alice", "alice@example.com"))
print(ctrl.create_user("Bob", "bob@example.com"))
print(ctrl.list_users())
print(ctrl.update_user(1, name="Alice Smith"))
print(ctrl.delete_user(2))
print(ctrl.list_users())
```

## Beginner Examples

```python
# Simple todo app MVC
class TodoModel:
    def __init__(self):
        self._todos: List[dict] = []
        self._next_id = 1

    def add(self, title: str) -> dict:
        todo = {"id": self._next_id, "title": title, "completed": False}
        self._todos.append(todo)
        self._next_id += 1
        return todo

    def toggle(self, todo_id: int) -> Optional[dict]:
        for todo in self._todos:
            if todo["id"] == todo_id:
                todo["completed"] = not todo["completed"]
                return todo
        return None

    def remove(self, todo_id: int) -> bool:
        for todo in self._todos:
            if todo["id"] == todo_id:
                self._todos.remove(todo)
                return True
        return False

    def get_all(self) -> List[dict]:
        return list(self._todos)

    def clear_completed(self):
        self._todos = [t for t in self._todos if not t["completed"]]


class TodoView:
    def render_list(self, todos: List[dict]) -> str:
        if not todos:
            return "No todos"
        lines = []
        for todo in todos:
            status = "[X]" if todo["completed"] else "[ ]"
            lines.append(f"{todo['id']}. {status} {todo['title']}")
        return "\n".join(lines)

    def render_message(self, message: str) -> str:
        return f"*** {message} ***"


class TodoController:
    def __init__(self, model: TodoModel, view: TodoView):
        self._model = model
        self._view = view

    def add(self, title: str):
        todo = self._model.add(title)
        print(self._view.render_message(f"Added: {todo['title']}"))

    def toggle(self, todo_id: int):
        todo = self._model.toggle(todo_id)
        if todo:
            status = "completed" if todo["completed"] else "uncompleted"
            print(self._view.render_message(f"Todo {todo_id} {status}"))
        else:
            print(self._view.render_message("Todo not found"))

    def remove(self, todo_id: int):
        if self._model.remove(todo_id):
            print(self._view.render_message(f"Removed todo {todo_id}"))
        else:
            print(self._view.render_message("Todo not found"))

    def list_all(self):
        print(self._view.render_list(self._model.get_all()))

    def clear_completed(self):
        self._model.clear_completed()
        print(self._view.render_message("Cleared completed todos"))


ctrl = TodoController(TodoModel(), TodoView())
ctrl.add("Learn MVC pattern")
ctrl.add("Write Python code")
ctrl.add("Review design patterns")
ctrl.toggle(1)
ctrl.list_all()
ctrl.remove(2)
ctrl.list_all()
```

```python
# Counter MVC
class CounterModel:
    def __init__(self):
        self._count = 0

    def increment(self) -> int:
        self._count += 1
        return self._count

    def decrement(self) -> int:
        self._count -= 1
        return self._count

    def reset(self) -> int:
        self._count = 0
        return self._count

    @property
    def value(self) -> int:
        return self._count


class CounterView:
    def display(self, count: int) -> str:
        return f"Count: {count}"

    def display_history(self, history: List[int]) -> str:
        return "History: " + ", ".join(str(h) for h in history)


class CounterController:
    def __init__(self, model: CounterModel, view: CounterView):
        self._model = model
        self._view = view
        self._history = []

    def increment(self):
        value = self._model.increment()
        self._history.append(value)
        print(self._view.display(value))

    def decrement(self):
        value = self._model.decrement()
        self._history.append(value)
        print(self._view.display(value))

    def reset(self):
        value = self._model.reset()
        self._history.append(value)
        print(self._view.display(value))

    def show_history(self):
        print(self._view.display_history(self._history))


ctrl = CounterController(CounterModel(), CounterView())
ctrl.increment()
ctrl.increment()
ctrl.decrement()
ctrl.show_history()
ctrl.reset()
```

## Intermediate Examples

```python
# MVC with multiple views (Observer-based sync)
class MVCModel:
    def __init__(self):
        self._data = {}
        self._observers = []

    def attach(self, observer):
        self._observers.append(observer)

    def detach(self, observer):
        self._observers.remove(observer)

    def _notify(self, event: str, **kwargs):
        for obs in self._observers:
            obs.update(event, **kwargs)

    def set(self, key: str, value: Any):
        old = self._data.get(key)
        self._data[key] = value
        self._notify("changed", key=key, value=value, old_value=old)

    def get(self, key: str):
        return self._data.get(key)

    def all(self) -> dict:
        return dict(self._data)


class BaseView:
    def update(self, event: str, **kwargs):
        raise NotImplementedError

    def render(self) -> str:
        raise NotImplementedError


class ConsoleView(BaseView):
    def __init__(self, model: MVCModel):
        self._model = model
        self._model.attach(self)

    def update(self, event: str, **kwargs):
        print(f"[View] {event}: {kwargs}")
        self.render()

    def render(self) -> str:
        data = self._model.all()
        print(f"  Data: {data}")
        return str(data)


class StatsView(BaseView):
    def __init__(self, model: MVCModel):
        self._model = model
        self._model.attach(self)
        self._change_count = 0

    def update(self, event: str, **kwargs):
        self._change_count += 1
        self.render()

    def render(self) -> str:
        data = self._model.all()
        stats = f"Total keys: {len(data)}, Changes: {self._change_count}"
        print(f"  Stats: {stats}")
        return stats


class JSONExportView(BaseView):
    def __init__(self, model: MVCModel):
        self._model = model

    def update(self, event: str, **kwargs):
        pass

    def render(self) -> str:
        return json.dumps(self._model.all(), indent=2)


class MVCController:
    def __init__(self, model: MVCModel):
        self._model = model

    def update(self, key: str, value: Any):
        self._model.set(key, value)

    def delete(self, key: str):
        self._model.set(key, None)


model = MVCModel()
console = ConsoleView(model)
stats = StatsView(model)
controller = MVCController(model)

controller.update("name", "Alice")
controller.update("age", 30)
controller.update("city", "New York")

export = JSONExportView(model)
print("\nJSON Export:")
print(export.render())
```

```python
# MVC with routing (web-style)
import re


class Request:
    def __init__(self, method: str, path: str, body: dict = None):
        self.method = method
        self.path = path
        self.body = body or {}
        self.params = {}

    def __repr__(self):
        return f"{self.method} {self.path}"


class Response:
    def __init__(self, status: int = 200, body: Any = None):
        self.status = status
        self.body = body

    def __repr__(self):
        return f"HTTP {self.status}: {self.body}"


class Router:
    def __init__(self):
        self._routes: List[tuple] = []

    def add(self, method: str, pattern: str, handler: Callable):
        regex = re.compile(f"^{pattern}$")
        self._routes.append((method, regex, handler))

    def dispatch(self, request: Request) -> Response:
        for method, pattern, handler in self._routes:
            if method == request.method:
                match = pattern.match(request.path)
                if match:
                    request.params = match.groupdict()
                    return handler(request)
        return Response(404, "Not Found")


class ProductModel:
    def __init__(self):
        self._products = {
            1: {"id": 1, "name": "Laptop", "price": 999.99},
            2: {"id": 2, "name": "Mouse", "price": 29.99},
        }

    def all(self) -> List[dict]:
        return list(self._products.values())

    def get(self, product_id: int) -> Optional[dict]:
        return self._products.get(product_id)

    def create(self, name: str, price: float) -> dict:
        new_id = max(self._products.keys()) + 1
        self._products[new_id] = {"id": new_id, "name": name, "price": price}
        return self._products[new_id]

    def update(self, product_id: int, **kwargs) -> Optional[dict]:
        product = self._products.get(product_id)
        if product:
            product.update(kwargs)
        return product

    def delete(self, product_id: int) -> bool:
        return self._products.pop(product_id, None) is not None


class ProductView:
    def render_product(self, product: dict) -> str:
        return f"Product #{product['id']}: {product['name']} (${product['price']:.2f})"

    def render_list(self, products: List[dict]) -> str:
        return "\n".join(self.render_product(p) for p in products)

    def render_json(self, data: Any) -> str:
        return json.dumps(data, indent=2)

    def render_error(self, message: str) -> str:
        return json.dumps({"error": message})


class ProductController:
    def __init__(self, model: ProductModel, view: ProductView):
        self._model = model
        self._view = view

    def index(self, request: Request) -> Response:
        products = self._model.all()
        return Response(200, self._view.render_list(products))

    def show(self, request: Request) -> Response:
        product_id = int(request.params.get("id", 0))
        product = self._model.get(product_id)
        if not product:
            return Response(404, self._view.render_error("Product not found"))
        return Response(200, self._view.render_product(product))

    def create(self, request: Request) -> Response:
        name = request.body.get("name")
        price = request.body.get("price")
        if not name or price is None:
            return Response(400, self._view.render_error("Name and price required"))
        product = self._model.create(name, float(price))
        return Response(201, self._view.render_product(product))

    def update(self, request: Request) -> Response:
        product_id = int(request.params.get("id", 0))
        product = self._model.update(product_id, **request.body)
        if not product:
            return Response(404, self._view.render_error("Product not found"))
        return Response(200, self._view.render_product(product))

    def delete(self, request: Request) -> Response:
        product_id = int(request.params.get("id", 0))
        if self._model.delete(product_id):
            return Response(204, "")
        return Response(404, self._view.render_error("Product not found"))


model = ProductModel()
view = ProductView()
ctrl = ProductController(model, view)
router = Router()

router.add("GET", "/products", ctrl.index)
router.add("GET", r"/products/(?P<id>\d+)", ctrl.show)
router.add("POST", "/products", ctrl.create)
router.add("PUT", r"/products/(?P<id>\d+)", ctrl.update)
router.add("DELETE", r"/products/(?P<id>\d+)", ctrl.delete)

print(router.dispatch(Request("GET", "/products")))
print(router.dispatch(Request("GET", "/products/1")))
print(router.dispatch(Request("POST", "/products", {"name": "Keyboard", "price": 79.99})))
print(router.dispatch(Request("DELETE", "/products/999")))
```

## Advanced Examples

```python
# MVVM pattern (Model-View-ViewModel)
import tkinter as tk
from typing import Any, Dict, List, Callable


class ObservableProperty:
    def __init__(self, default=None):
        self._value = default
        self._observers: List[Callable] = []

    def observe(self, callback: Callable):
        self._observers.append(callback)

    @property
    def value(self):
        return self._value

    @value.setter
    def value(self, new_value):
        old = self._value
        self._value = new_value
        for cb in self._observers:
            cb(new_value, old)


class UserModel:
    def __init__(self):
        self.name = ObservableProperty("")
        self.email = ObservableProperty("")
        self.age = ObservableProperty(0)

    def validate(self) -> List[str]:
        errors = []
        if not self.name.value:
            errors.append("Name is required")
        if "@" not in self.email.value:
            errors.append("Invalid email")
        if self.age.value < 0 or self.age.value > 150:
            errors.append("Invalid age")
        return errors


class UserViewModel:
    def __init__(self, model: UserModel):
        self._model = model
        self.name = ""
        self.email = ""
        self.age = 0
        self.errors: List[str] = []
        self.on_errors_changed: Callable = None

    def save(self) -> bool:
        self._model.name.value = self.name
        self._model.email.value = self.email
        self._model.age.value = self.age
        self.errors = self._model.validate()
        if self.on_errors_changed:
            self.on_errors_changed(self.errors)
        return len(self.errors) == 0


class UserView:
    def __init__(self, view_model: UserViewModel):
        self._vm = view_model
        self._vm.on_errors_changed = self.show_errors

    def show_errors(self, errors: List[str]):
        if errors:
            print("Validation errors:")
            for e in errors:
                print(f"  - {e}")
        else:
            print("User saved successfully!")


model = UserModel()
vm = UserViewModel(model)
view = UserView(vm)

vm.name = "Alice"
vm.email = "invalid"
vm.age = 200
vm.save()

vm.email = "alice@example.com"
vm.age = 30
vm.save()
```

```python
# Django-style MTV (Model-Template-View)
class MTVModel:
    def __init__(self):
        self._data = {}

    def save(self):
        print(f"[DB] Saved: {self._data}")

    def to_dict(self) -> dict:
        return dict(self._data)


class ArticleModel(MTVModel):
    def __init__(self, title: str, content: str, author: str):
        super().__init__()
        self._data = {
            "title": title,
            "content": content,
            "author": author,
            "published": False,
            "views": 0,
        }

    def publish(self):
        self._data["published"] = True
        self.save()

    def view(self):
        self._data["views"] += 1


class Template:
    def render(self, context: dict) -> str:
        raise NotImplementedError


class ArticleTemplate(Template):
    def render(self, context: dict) -> str:
        article = context["article"]
        published = "Published" if article["published"] else "Draft"
        return f"""
<html>
<head><title>{article['title']}</title></head>
<body>
  <h1>{article['title']}</h1>
  <p>By {article['author']} | Status: {published}</p>
  <p>Views: {article['views']}</p>
  <article>{article['content']}</article>
</body>
</html>"""


class ArticleListView(Template):
    def render(self, context: dict) -> str:
        articles = context["articles"]
        items = "\n".join(
            f"<li><a href='/article/{a['title']}'>{a['title']}</a> by {a['author']}</li>"
            for a in articles
        )
        return f"<html><body><ul>{items}</ul></body></html>"


class View:
    def get(self, request: Request, **kwargs) -> Response:
        raise NotImplementedError


class ArticleDetailView(View):
    def __init__(self, model_class, template: Template):
        self._model_class = model_class
        self._template = template

    def get(self, request: Request, **kwargs) -> Response:
        article = self._model_class("Sample", "Hello World", "Alice")
        article.view()
        html = self._template.render({"article": article.to_dict()})
        return Response(200, html)


class ArticleListView(View):
    def __init__(self, template: Template):
        self._template = template

    def get(self, request: Request, **kwargs) -> Response:
        articles = [
            ArticleModel("Post 1", "Content 1", "Alice"),
            ArticleModel("Post 2", "Content 2", "Bob"),
        ]
        html = self._template.render({
            "articles": [a.to_dict() for a in articles]
        })
        return Response(200, html)


detail_view = ArticleDetailView(ArticleModel, ArticleTemplate())
list_view = ArticleListView(ArticleListView())

router2 = Router()
router2.add("GET", "/article/(?P<title>\\w+)", detail_view.get)
router2.add("GET", "/articles", list_view.get)

print(router2.dispatch(Request("GET", "/articles")))
```

```python
# FastAPI-style MVC
from dataclasses import dataclass
from typing import List, Optional


@dataclass
class Item:
    id: int
    name: str
    price: float
    in_stock: bool = True


class ItemModel:
    def __init__(self):
        self._items = [
            Item(1, "Laptop", 999.99),
            Item(2, "Mouse", 29.99),
            Item(3, "Keyboard", 79.99, in_stock=False),
        ]
        self._next_id = 4

    def list_items(self) -> List[Item]:
        return self._items

    def get_item(self, item_id: int) -> Optional[Item]:
        for item in self._items:
            if item.id == item_id:
                return item
        return None

    def create_item(self, name: str, price: float) -> Item:
        item = Item(self._next_id, name, price)
        self._items.append(item)
        self._next_id += 1
        return item

    def update_item(self, item_id: int, **kwargs) -> Optional[Item]:
        item = self.get_item(item_id)
        if item:
            for k, v in kwargs.items():
                if hasattr(item, k):
                    setattr(item, k, v)
        return item

    def delete_item(self, item_id: int) -> bool:
        item = self.get_item(item_id)
        if item:
            self._items.remove(item)
            return True
        return False


class ItemSchema:
    @staticmethod
    def to_dict(item: Item) -> dict:
        return {"id": item.id, "name": item.name, "price": item.price, "in_stock": item.in_stock}

    @staticmethod
    def to_json(items) -> str:
        if isinstance(items, list):
            return json.dumps([ItemSchema.to_dict(i) for i in items], indent=2)
        return json.dumps(ItemSchema.to_dict(items), indent=2)


class ItemService:
    def __init__(self, model: ItemModel):
        self._model = model

    def search(self, query: str = None, min_price: float = None, max_price: float = None) -> List[Item]:
        items = self._model.list_items()
        if query:
            items = [i for i in items if query.lower() in i.name.lower()]
        if min_price is not None:
            items = [i for i in items if i.price >= min_price]
        if max_price is not None:
            items = [i for i in items if i.price <= max_price]
        return items


class ItemController:
    def __init__(self, model: ItemModel, service: ItemService):
        self._model = model
        self._service = service

    def list_all(self, request: Request) -> Response:
        query = request.body.get("query")
        min_price = request.body.get("min_price")
        max_price = request.body.get("max_price")
        items = self._service.search(query, min_price, max_price)
        return Response(200, ItemSchema.to_json(items))

    def get(self, request: Request) -> Response:
        item = self._model.get_item(int(request.params["id"]))
        if not item:
            return Response(404, json.dumps({"error": "Item not found"}))
        return Response(200, ItemSchema.to_json(item))

    def create(self, request: Request) -> Response:
        name = request.body.get("name")
        price = request.body.get("price")
        if not name or price is None:
            return Response(400, json.dumps({"error": "Name and price required"}))
        item = self._model.create_item(name, float(price))
        return Response(201, ItemSchema.to_json(item))

    def update(self, request: Request) -> Response:
        item = self._model.update_item(int(request.params["id"]), **request.body)
        if not item:
            return Response(404, json.dumps({"error": "Item not found"}))
        return Response(200, ItemSchema.to_json(item))

    def delete(self, request: Request) -> Response:
        if self._model.delete_item(int(request.params["id"])):
            return Response(204, "")
        return Response(404, json.dumps({"error": "Item not found"}))


model = ItemModel()
service = ItemService(model)
ctrl = ItemController(model, service)
router3 = Router()

router3.add("GET", "/api/items", ctrl.list_all)
router3.add("GET", r"/api/items/(?P<id>\d+)", ctrl.get)
router3.add("POST", "/api/items", ctrl.create)
router3.add("PUT", r"/api/items/(?P<id>\d+)", ctrl.update)
router3.add("DELETE", r"/api/items/(?P<id>\d+)", ctrl.delete)

print(router3.dispatch(Request("GET", "/api/items")))
print(router3.dispatch(Request("POST", "/api/items", {"name": "Monitor", "price": 299.99})))
```

## Real-World Use Cases

```python
# Flask-style MVC structure
# models/user.py
class User:
    def __init__(self, db):
        self._db = db
        self._table = "users"

    def find_by_id(self, user_id: int) -> Optional[dict]:
        return self._db.query(f"SELECT * FROM {self._table} WHERE id = ?", (user_id,))

    def find_by_email(self, email: str) -> Optional[dict]:
        return self._db.query(f"SELECT * FROM {self._table} WHERE email = ?", (email,))

    def create(self, name: str, email: str) -> int:
        return self._db.execute(
            f"INSERT INTO {self._table} (name, email) VALUES (?, ?)",
            (name, email),
        )

    def update(self, user_id: int, **kwargs) -> bool:
        if not kwargs:
            return False
        sets = ", ".join(f"{k} = ?" for k in kwargs)
        values = list(kwargs.values()) + [user_id]
        return self._db.execute(
            f"UPDATE {self._table} SET {sets} WHERE id = ?", values
        )


# views/user_view.py
class UserTemplate:
    def render_profile(self, user: dict) -> str:
        return f"""
        <div class="profile">
            <h2>{user['name']}</h2>
            <p>Email: {user['email']}</p>
        </div>"""

    def render_list(self, users: List[dict]) -> str:
        items = "".join(
            f"<li><a href='/users/{u['id']}'>{u['name']}</a></li>"
            for u in users
        )
        return f"<ul>{items}</ul>"


# controllers/user_controller.py
class UserController:
    def __init__(self, user_model, template):
        self._model = user_model
        self._template = template

    def show(self, user_id: int) -> str:
        user = self._model.find_by_id(user_id)
        if not user:
            return "<h1>404 Not Found</h1>"
        return self._template.render_profile(user)

    def register(self, name: str, email: str) -> str:
        existing = self._model.find_by_email(email)
        if existing:
            return "<h1>Email already registered</h1>"
        user_id = self._model.create(name, email)
        return f"<h1>User created with ID {user_id}</h1>"
```

```python
# Django MTV architecture simulation
# Django's MTV: Model defines data, Template renders HTML, View handles logic
class DjangoLikeModel:
    class Meta:
        table_name = ""

    def __init__(self, **kwargs):
        for k, v in kwargs.items():
            setattr(self, k, v)

    def save(self):
        fields = {k: v for k, v in self.__dict__.items() if not k.startswith("_")}
        print(f"[ORM] SAVED {self.Meta.table_name}: {fields}")

    @classmethod
    def objects_all(cls):
        return cls._mock_db.get(cls.Meta.table_name, [])

    @classmethod
    def objects_filter(cls, **kwargs):
        results = cls.objects_all()
        for k, v in kwargs.items():
            results = [r for r in results if getattr(r, k, None) == v]
        return results


class BlogPost(DjangoLikeModel):
    class Meta:
        table_name = "blog_posts"

    _mock_db = {"blog_posts": []}

    def __init__(self, title="", content="", author=""):
        super().__init__(title=title, content=content, author=author)
        self.id = len(self._mock_db["blog_posts"]) + 1


def populate_db():
    posts = [
        BlogPost(title="First Post", content="Hello World", author="Alice"),
        BlogPost(title="Second Post", content="MVC is great", author="Bob"),
    ]
    for p in posts:
        p.save()
        BlogPost._mock_db["blog_posts"].append(p)


class DjangoTemplate:
    def render(self, template_name: str, context: dict) -> str:
        if template_name == "blog/list.html":
            posts = context.get("posts", [])
            items = "\n".join(
                f"<h2><a href='/post/{p.id}/'>{p.title}</a></h2>"
                f"<p>By {p.author}</p>"
                for p in posts
            )
            return f"<html><body><h1>Blog</h1>{items}</body></html>"

        elif template_name == "blog/detail.html":
            post = context.get("post")
            return (
                f"<html><body>"
                f"<h1>{post.title}</h1>"
                f"<p>By {post.author}</p>"
                f"<article>{post.content}</article>"
                f"</body></html>"
            )
        return "<html><body><h1>Template not found</h1></body></html>"


class DjangoView:
    def __init__(self, template: DjangoTemplate):
        self._template = template

    def get(self, request: Request, **kwargs):
        raise NotImplementedError


class BlogListView(DjangoView):
    def get(self, request: Request, **kwargs):
        posts = BlogPost.objects_all()
        html = self._template.render("blog/list.html", {"posts": posts})
        return Response(200, html)


class BlogDetailView(DjangoView):
    def get(self, request: Request, **kwargs):
        post_id = int(kwargs.get("id", 0))
        posts = BlogPost.objects_filter(id=post_id)
        if not posts:
            return Response(404, "<h1>Not Found</h1>")
        html = self._template.render("blog/detail.html", {"post": posts[0]})
        return Response(200, html)


populate_db()
template = DjangoTemplate()
router4 = Router()
router4.add("GET", "/", BlogListView(template).get)
router4.add("GET", r"/post/(?P<id>\d+)/", BlogDetailView(template).get)

print(router4.dispatch(Request("GET", "/")))
print(router4.dispatch(Request("GET", "/post/1/")))
```

## Common Mistakes

```python
# MISTAKE 1: Fat controller with business logic
class FatController:
    def save_user(self, data):
        # Validation
        if not data.get("name"):
            return "Error"
        # Business logic
        self.send_email(data["email"])
        # Database logic
        self.db.save(data)
        # View logic
        return self.view.render(data)
```

```python
# MISTAKE 2: Model leaking into view directly
class BadView:
    def render(self):
        # View should not know about database
        return str(Model._db.query("SELECT * FROM users"))
```

```python
# MISTAKE 3: Tight coupling between components
class TightController:
    def __init__(self):
        self.model = UserModel()  # Hard-coded!
        self.view = HTMLView()    # Hard-coded!
```

## Best Practices

```python
# 1. Keep models focused on data and business rules
# 2. Controllers should be thin — delegate to services
# 3. Views should only handle presentation logic
# 4. Use dependency injection for loose coupling
# 5. Follow consistent naming conventions
# 6. Use services layer for complex business logic
# 7. Keep models testable without views
```

```python
# Best practice: Service layer between controller and model
class UserService:
    def __init__(self, user_repo, email_service):
        self._repo = user_repo
        self._email = email_service

    def register(self, name, email):
        if self._repo.find_by_email(email):
            raise ValueError("Email exists")
        user = self._repo.create(name, email)
        self._email.send_welcome(email)
        return user
```

## Interview Questions

```python
# Q1: MVC vs MVVM vs MVP?
# MVC: Controller handles input, updates Model, selects View
# MVVM: ViewModel exposes data/commands for binding
# MVP: Presenter handles all UI logic, View is passive
```

```python
# Q2: Why separate Model, View, and Controller?
# Separation of concerns, testability, maintainability,
# parallel development, multiple views per model
```

```python
# Q3: Django MTV vs classic MVC?
# Django's Model = Model, Template = View, View = Controller
# Same separation, different naming convention
```

```python
# Q4: How does MVC support testing?
# Models testable independently
# Controllers testable with mock models/views
# Views can be tested with mock data
```

## Coding Challenges

```python
# Challenge 1: Build a complete MVC todo app with
# add, edit, delete, complete, and filter capabilities
```

```python
# Challenge 2: Implement MVVM with data binding
# that auto-updates the view when model changes
```

```python
# Challenge 3: Create a REST API following MVC
# with proper separation of concerns
```

```python
# Challenge 4: Build a miniature web framework
# with URL routing, controllers, and template rendering
```

```python
# Challenge 5: Refactor a monolithic application
# into MVC layers without changing functionality
```

## Summary

MVC separates applications into Model (data/business logic), View (presentation), and Controller (input handling/coordination). This separation enables modular, testable, and maintainable code. Python web frameworks like Django (MTV) and Flask (flexible) implement MVC variants. MVVM is common in GUI applications with data binding. The key principle is separation of concerns — each component has a distinct responsibility and communicates through well-defined interfaces.

## Related Topics

- Observer Pattern: Often used for Model-View synchronization
- Command Pattern: Can be used for Controller actions
- Strategy Pattern: Controllers can use strategies for request handling
- Template Method: Used in framework base classes
- Service Layer: Between Controller and Model for business logic
- Dependency Injection: Decouples MVC components
