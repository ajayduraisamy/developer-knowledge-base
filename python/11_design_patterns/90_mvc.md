# MVC - Model-View-Controller, separation of concerns, MVVM
## Introduction
MVC (Model-View-Controller) is a architectural pattern that separates an application into three interconnected components: Model (data and business logic), View (user interface), and Controller (handles user input and coordinates Model and View). This separation enables modular development, parallel workflow, and maintainability. Python frameworks like Django use a variation called MVT (Model-View-Template), while desktop and mobile frameworks often use MVVM (Model-View-ViewModel).

## Model
### What It Is
The Model represents the application's data and business logic. It manages the data, responds to queries, and encapsulates the rules for manipulating data. The Model is independent of the user interface.

### Why It Is Important
The Model centralizes data management and business rules. Changes to data logic only affect the Model, not the Views or Controllers. This separation makes the code more maintainable and testable.

### How It Works Internally
The Model typically contains data structures (classes, databases), validation logic, computation rules, and data access methods. In frameworks like Django, the Model maps to database tables via ORM.

```python
from typing import List, Optional, Dict
from datetime import datetime
import json

class UserModel:
    def __init__(self):
        self._users: Dict[int, dict] = {}
        self._next_id = 1
        self._observers = []

    def add_observer(self, observer):
        self._observers.append(observer)

    def _notify_observers(self, event, data):
        for observer in self._observers:
            observer.update(event, data)

    def create_user(self, name: str, email: str) -> dict:
        user = {
            "id": self._next_id,
            "name": name,
            "email": email,
            "created_at": datetime.now().isoformat()
        }
        self._users[self._next_id] = user
        self._next_id += 1
        self._notify_observers("user_created", user)
        return user

    def get_user(self, user_id: int) -> Optional[dict]:
        return self._users.get(user_id)

    def get_all_users(self) -> List[dict]:
        return list(self._users.values())

    def update_user(self, user_id: int, **kwargs) -> Optional[dict]:
        user = self._users.get(user_id)
        if user:
            user.update(kwargs)
            self._notify_observers("user_updated", user)
        return user

    def delete_user(self, user_id: int) -> bool:
        user = self._users.pop(user_id, None)
        if user:
            self._notify_observers("user_deleted", user)
            return True
        return False

class ProductModel:
    def __init__(self):
        self._products = {}
        self._next_id = 1

    def add_product(self, name: str, price: float, quantity: int) -> dict:
        product = {
            "id": self._next_id,
            "name": name,
            "price": price,
            "quantity": quantity
        }
        self._products[self._next_id] = product
        self._next_id += 1
        return product

    def get_product(self, product_id: int) -> Optional[dict]:
        return self._products.get(product_id)

    def update_stock(self, product_id: int, quantity: int) -> Optional[dict]:
        product = self._products.get(product_id)
        if product:
            product["quantity"] = quantity
        return product
```

## View
### What It Is
The View is the visual representation of the Model. It displays data to the user and provides the user interface for interaction. The View observes the Model and updates when data changes.

### Why It Is Important
The View encapsulates all presentation logic. By separating it from the Model and Controller, different Views (web, desktop, mobile, CLI) can reuse the same Model and business logic.

### How It Works Internally
The View subscribes to Model changes and re-renders when notified. It receives user input events and forwards them to the Controller. The View should not contain business logic or direct data manipulation.

```python
from abc import ABC, abstractmethod

class Observer(ABC):
    @abstractmethod
    def update(self, event: str, data):
        pass

class UserConsoleView(Observer):
    def __init__(self):
        self._displayed_users = []

    def update(self, event: str, data):
        if event == "user_created":
            print(f"[View] New user added: {data['name']}")
        elif event == "user_updated":
            print(f"[View] User updated: {data}")
        elif event == "user_deleted":
            print(f"[View] User deleted: {data['name']}")

    def show_users(self, users):
        print("\n=== User List ===")
        for user in users:
            print(f"  {user['id']}. {user['name']} ({user['email']})")
        print("================\n")

    def show_user_details(self, user):
        if user:
            print(f"\nID: {user['id']}")
            print(f"Name: {user['name']}")
            print(f"Email: {user['email']}")
            print(f"Created: {user['created_at']}\n")
        else:
            print("User not found.\n")

    def get_user_input(self) -> dict:
        name = input("Enter name: ")
        email = input("Enter email: ")
        return {"name": name, "email": email}

class UserHTMLView(Observer):
    def update(self, event: str, data):
        if event == "user_created":
            self._render_user_row(data)

    def render_user_table(self, users):
        html = "<table>\n<thead>\n<tr><th>ID</th><th>Name</th><th>Email</th></tr>\n</thead>\n<tbody>\n"
        for user in users:
            html += f"  <tr><td>{user['id']}</td><td>{user['name']}</td><td>{user['email']}</td></tr>\n"
        html += "</tbody>\n</table>"
        return html

    def _render_user_row(self, user):
        print(f"Appending row for {user['name']} to HTML table")

class UserJSONView(Observer):
    def update(self, event: str, data):
        if event == "user_created":
            print(f"[JSON] {json.dumps(data, indent=2)}")

    def render(self, users):
        return json.dumps(users, indent=2)
```

## Controller
### What It Is
The Controller handles user input, interprets it, and updates the Model or View accordingly. It acts as the intermediary between the View and Model, containing the application logic for responding to user actions.

### Why It Is Important
The Controller decouples input handling from presentation and data logic. It coordinates the application flow, validates input, and decides which View to display.

### How It Works Internally
The Controller receives user actions from the View, translates them into Model operations, and updates the View with results. It may also handle navigation, authentication, and other cross-cutting concerns.

```python
class UserController:
    def __init__(self, model: UserModel, view: UserConsoleView):
        self._model = model
        self._view = view
        self._model.add_observer(self._view)

    def list_users(self):
        users = self._model.get_all_users()
        self._view.show_users(users)

    def show_user(self, user_id: int):
        user = self._model.get_user(user_id)
        self._view.show_user_details(user)

    def create_user(self):
        data = self._view.get_user_input()
        self._validate_user_data(data)
        user = self._model.create_user(data["name"], data["email"])
        return user

    def delete_user(self, user_id: int):
        if self._model.delete_user(user_id):
            print(f"User {user_id} deleted successfully.")

    def _validate_user_data(self, data):
        if not data.get("name") or not data.get("email"):
            raise ValueError("Name and email are required")
        if "@" not in data["email"]:
            raise ValueError("Invalid email format")

class ProductController:
    def __init__(self, model: ProductModel, view):
        self._model = model
        self._view = view

    def add_product(self, name: str, price: float, quantity: int):
        if price < 0:
            raise ValueError("Price cannot be negative")
        product = self._model.add_product(name, price, quantity)
        return product

    def check_stock(self, product_id: int, requested: int) -> bool:
        product = self._model.get_product(product_id)
        if not product:
            return False
        return product["quantity"] >= requested
```

## Separation of concerns
### What It Is
Separation of concerns (SoC) divides a program into distinct sections, each addressing a separate concern. In MVC, the Model handles data, the View handles presentation, and the Controller handles interaction logic.

### Why It Is Important
SoC enables independent development, testing, and modification of each component. Changes to one concern (e.g., switching from console to web UI) don't affect others. Teams can work on different components in parallel.

### How It Works Internally
Communication between components follows strict rules:
- Model <-> View: View subscribes to Model changes (Observer pattern)
- Controller -> Model: Controller updates Model directly
- Controller -> View: Controller tells View what to display
- View -> Controller: View forwards user input events to Controller

```python
# Clean separation example
class MVCApplication:
    def __init__(self):
        self._model = UserModel()
        self._view = UserConsoleView()
        self._controller = UserController(self._model, self._view)

    def run(self):
        while True:
            action = input("Action (list/show/create/delete/quit): ").strip().lower()
            try:
                if action == "quit":
                    break
                elif action == "list":
                    self._controller.list_users()
                elif action == "show":
                    user_id = int(input("User ID: "))
                    self._controller.show_user(user_id)
                elif action == "create":
                    self._controller.create_user()
                elif action == "delete":
                    user_id = int(input("User ID: "))
                    self._controller.delete_user(user_id)
                else:
                    print("Unknown action")
            except Exception as e:
                print(f"Error: {e}")

# Usage
app = MVCApplication()
app.run()
```

## MVVM (Model-View-ViewModel)
### What It Is
MVVM is an evolution of MVC popularized by Microsoft for WPF/Silverlight. The ViewModel replaces the Controller, acting as a specialized intermediary that exposes data from the Model in a form suitable for the View, often using data binding.

### Why It Is Important
MVVM enables a more declarative UI approach where Views bind directly to ViewModel properties. This reduces boilerplate code and enables two-way data binding, making UIs more responsive and easier to maintain.

### How It Works Internally
The ViewModel converts Model data into View-friendly formats and exposes commands for View actions. Data binding mechanisms automatically synchronize the View and ViewModel. The ViewModel has no reference to the View (unlike MVC's Controller that may update the View directly).

```python
from typing import List, Callable
import tkinter as tk
from tkinter import ttk, messagebox

class UserViewModel:
    def __init__(self, model: UserModel):
        self._model = model
        self._on_users_changed: List[Callable] = []
        self._selected_user = None

    @property
    def users(self):
        return self._model.get_all_users()

    @property
    def selected_user(self):
        return self._selected_user

    @selected_user.setter
    def selected_user(self, user):
        self._selected_user = user
        self._notify_property_changed("selected_user")

    def bind(self, callback):
        self._on_users_changed.append(callback)

    def _notify(self, event, data=None):
        for callback in self._on_users_changed:
            callback()

    def _notify_property_changed(self, property_name):
        pass

    def create_user(self, name: str, email: str):
        self._model.create_user(name, email)
        self._notify("users_changed")

    def delete_user(self, user_id: int):
        self._model.delete_user(user_id)
        self._notify("users_changed")

    def get_user_display_text(self, user_id: int) -> str:
        user = self._model.get_user(user_id)
        if user:
            return f"{user['name']} ({user['email']})"
        return ""

# View (Tkinter)
class UserView:
    def __init__(self, view_model: UserViewModel):
        self._vm = view_model
        self._vm.bind(self._refresh)

        self.root = tk.Tk()
        self.root.title("User Manager")
        self.root.geometry("500x400")

        # Listbox
        self.listbox = tk.Listbox(self.root)
        self.listbox.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)
        self.listbox.bind('<<ListboxSelect>>', self._on_select)

        # Entry fields
        frame = ttk.Frame(self.root)
        frame.pack(fill=tk.X, padx=10)

        ttk.Label(frame, text="Name:").grid(row=0, column=0, sticky=tk.W)
        self.name_entry = ttk.Entry(frame)
        self.name_entry.grid(row=0, column=1, padx=5, pady=2, sticky=tk.EW)

        ttk.Label(frame, text="Email:").grid(row=1, column=0, sticky=tk.W)
        self.email_entry = ttk.Entry(frame)
        self.email_entry.grid(row=1, column=1, padx=5, pady=2, sticky=tk.EW)

        frame.columnconfigure(1, weight=1)

        # Buttons
        button_frame = ttk.Frame(self.root)
        button_frame.pack(fill=tk.X, padx=10, pady=10)

        ttk.Button(button_frame, text="Add", command=self._add_user).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="Delete", command=self._delete_user).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="Refresh", command=self._refresh).pack(side=tk.LEFT, padx=5)

    def _refresh(self):
        self.listbox.delete(0, tk.END)
        for user in self._vm.users:
            self.listbox.insert(tk.END, f"{user['name']} ({user['email']})")

    def _on_select(self, event):
        selection = self.listbox.curselection()
        if selection:
            index = selection[0]
            user = self._vm.users[index]
            self._vm.selected_user = user

    def _add_user(self):
        name = self.name_entry.get().strip()
        email = self.email_entry.get().strip()
        if name and email:
            self._vm.create_user(name, email)
            self.name_entry.delete(0, tk.END)
            self.email_entry.delete(0, tk.END)

    def _delete_user(self):
        if self._vm.selected_user:
            self._vm.delete_user(self._vm.selected_user["id"])
            self._vm.selected_user = None

    def run(self):
        self._refresh()
        self.root.mainloop()

# Usage
model = UserModel()
view_model = UserViewModel(model)
view = UserView(view_model)
# view.run()
```

### Django MVT (Model-View-Template)
```python
"""
Django's interpretation of MVC:
- Model: Database schema and business logic (models.py)
- View: Controller equivalent (views.py)
- Template: View equivalent (HTML templates)

# models.py
from django.db import models

class User(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.name

# views.py
from django.shortcuts import render, get_object_or_404
from django.http import JsonResponse
from .models import User

def user_list(request):
    users = User.objects.all()
    return render(request, 'users/list.html', {'users': users})

def user_detail(request, user_id):
    user = get_object_or_404(User, id=user_id)
    return render(request, 'users/detail.html', {'user': user})

def user_create(request):
    if request.method == 'POST':
        user = User.objects.create(
            name=request.POST['name'],
            email=request.POST['email']
        )
        return JsonResponse({'id': user.id, 'name': user.name, 'email': user.email})
    return render(request, 'users/create.html')

# templates/users/list.html
# <ul>
# {% for user in users %}
#   <li>{{ user.name }} - {{ user.email }}</li>
# {% endfor %}
# </ul>
"""
```

### Flask MVC Example
```python
"""
Flask application with MVC structure

app/
├── __init__.py
├── models/
│   ├── __init__.py
│   └── user.py
├── views/
│   ├── __init__.py
│   └── user_views.py
├── controllers/
│   ├── __init__.py
│   └── user_controller.py
├── templates/
│   └── users/
│       ├── list.html
│       └── form.html
└── static/
"""

# models/user.py
class User:
    def __init__(self, db):
        self.db = db

    def get_all(self):
        return self.db.execute("SELECT * FROM users").fetchall()

    def get_by_id(self, user_id):
        return self.db.execute(
            "SELECT * FROM users WHERE id = ?", (user_id,)
        ).fetchone()

    def create(self, name, email):
        cursor = self.db.execute(
            "INSERT INTO users (name, email) VALUES (?, ?)", (name, email)
        )
        self.db.commit()
        return cursor.lastrowid

# controllers/user_controller.py
from flask import Blueprint, render_template, request, redirect, url_for

user_bp = Blueprint('users', __name__)

class UserController:
    def __init__(self, model):
        self.model = model

    def list_users(self):
        users = self.model.get_all()
        return render_template('users/list.html', users=users)

    def create_user(self):
        if request.method == 'POST':
            name = request.form['name']
            email = request.form['email']
            self.model.create(name, email)
            return redirect(url_for('users.list'))
        return render_template('users/form.html')

# views/user_views.py
@user_bp.route('/users')
def list():
    controller = UserController(User(get_db()))
    return controller.list_users()

@user_bp.route('/users/create', methods=['GET', 'POST'])
def create():
    controller = UserController(User(get_db()))
    return controller.create_user()
```

### Real-World Use Cases
- Web frameworks (Django, Flask, Rails, Spring MVC)
- Desktop applications (Qt, Tkinter, WPF)
- Mobile apps (iOS, Android)
- Single-page applications (React, Angular, Vue)
- Game development (entity-component systems)
- Data visualization dashboards

### Common Mistakes
- Putting business logic in Views or Controllers
- Making Models too anemic (just data without behavior)
- Tight coupling between components
- Controller becoming a "god object" with too much responsibility
- Not using Observer pattern for Model-View communication
- Mixing presentation logic with business logic

### Best Practices
- Keep Models rich with business logic (domain-driven design)
- Keep Views dumb (only presentation logic)
- Keep Controllers/ViewModels thin (coordination only)
- Use dependency injection for component communication
- Implement Observer pattern for Model updates
- Use services for complex business logic that spans multiple Models

### Performance Considerations
- Observer notifications can cause cascading updates
- Two-way data binding in MVVM can create overhead
- Thick Views (complex UI) can slow rendering
- Caching in ViewModel can improve performance
- Lazy loading helps manage memory in data-heavy applications

### Interview Questions
1. Explain the responsibilities of Model, View, and Controller.
2. How does MVVM differ from MVC?
3. What is the role of the Controller in a web application?
4. How does Django's MVT compare to classic MVC?
5. Explain how data binding works in MVVM.

### Coding Challenges
1. Implement a simple MVC to-do list application
2. Build a note-taking app with MVVM architecture
3. Create a Django-like MVT framework from scratch
4. Convert a monolithic script into MVC architecture

### Related Topics
- Observer pattern
- Strategy pattern
- Dependency injection
- Three-tier architecture
- Domain-driven design
- RESTful API design
- Frontend frameworks (React, Vue, Angular)
