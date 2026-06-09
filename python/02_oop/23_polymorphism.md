# Polymorphism - Duck typing, method overriding, operator overloading

## Introduction

Polymorphism means "many forms." In Python, it allows objects of different types to respond to the same interface. Python implements polymorphism through duck typing (runtime type flexibility), method overriding (inheritance-based), and operator overloading (custom behavior for built-in operators). Python's dynamic nature makes polymorphism especially elegant and pervasive.

## Duck typing

### What It Is

Duck typing is a programming style where an object's suitability is determined by the presence of certain methods and properties, rather than by its actual type. The name comes from the saying: "If it walks like a duck and quacks like a duck, it's a duck."

```python
def make_it_quack(thing):
    thing.quack()  # Any object with quack() works
```

### Why It Is Important

Duck typing enables loose coupling and flexible code. Functions can operate on any object that supports the required interface without requiring explicit inheritance or type checking. This is a cornerstone of Python's "EAFP" (Easier to Ask for Forgiveness than Permission) philosophy.

### How It Works Internally

At the bytecode level, Python simply looks up the attribute (e.g., `.quack()`) on the object. If the attribute exists, it's called. There's no compile-time type checking — type errors only manifest at runtime when an attribute is missing.

```python
class Duck:
    def quack(self):
        return "Quack!"

class Person:
    def quack(self):
        return "I'm quacking!"

def in_the_forest(animal):
    print(animal.quack())

in_the_forest(Duck())    # Quack!
in_the_forest(Person())  # I'm quacking!
```

### Syntax

```python
# Duck typing — no type declaration needed
def func(obj):
    obj.method()  # Just call it — if it works, it works

# With type hints (structural subtyping via Protocol)
from typing import Protocol

class Quackable(Protocol):
    def quack(self) -> str:
        ...
```

### Beginner Examples

```python
class Cat:
    def sound(self):
        return "Meow"

class Dog:
    def sound(self):
        return "Woof"

class Car:
    def sound(self):
        return "Vroom"

def make_sound(animal):
    print(animal.sound())

for obj in [Cat(), Dog(), Car()]:
    make_sound(obj)
# Meow
# Woof
# Vroom
```

### Intermediate Examples

```python
class JSONSerializer:
    def serialize(self, data):
        import json
        return json.dumps(data)

class XMLSerializer:
    def serialize(self, data):
        return f"<data>{data}</data>"

class YamlSerializer:
    def serialize(self, data):
        import yaml
        return yaml.dump(data)

def export_data(serializer, data, filename):
    with open(filename, "w") as f:
        f.write(serializer.serialize(data))

# Any object with .serialize() works
export_data(JSONSerializer(), {"key": "value"}, "out.json")
export_data(XMLSerializer(), "hello", "out.xml")
```

### Advanced Examples

```python
from typing import Protocol, List, TypeVar

T = TypeVar("T")

class Drawable(Protocol):
    def draw(self) -> str:
        ...

class Circle:
    def __init__(self, radius: float):
        self.radius = radius

    def draw(self) -> str:
        return f"Circle(radius={self.radius})"

class Rectangle:
    def __init__(self, w: float, h: float):
        self.w, self.h = w, h

    def draw(self) -> str:
        return f"Rectangle({self.w}x{self.h})"

class Canvas:
    def __init__(self):
        self._shapes: List[Drawable] = []

    def add(self, shape: Drawable) -> None:
        self._shapes.append(shape)

    def render(self) -> str:
        return "\n".join(s.draw() for s in self._shapes)

# Works with anything implementing Drawable protocol
canvas = Canvas()
canvas.add(Circle(5))
canvas.add(Rectangle(3, 4))
print(canvas.render())
```

### Real-World Use Cases

- **Python's `len()`**: works with any object that has `__len__`
- **`with` statement**: works with any object having `__enter__`/`__exit__`
- **Iteration**: `for x in obj` works with any iterable
- **Django's class-based views**: different HTTP method handlers (get, post, etc.)
- **Pandas/ NumPy**: functions operate on array-like objects regardless of exact type

### Common Mistakes

1. **Type checking instead of duck typing**: `if isinstance(obj, Duck)` defeats the purpose
2. **Assuming attributes exist**: always handle `AttributeError` or use `hasattr`
3. **Inconsistent interfaces**: methods with same name but different semantics
4. **Over-relying on `hasattr`**: trying to anticipate all required methods

### Best Practices

- Write functions that expect interfaces, not concrete types
- Use `Protocol` (Python 3.8+) for static type checking with duck typing
- Document the expected interface in docstrings
- Trust that callers will pass compatible objects
- Use `try`/`except AttributeError` sparingly; let errors surface during development

### Performance Considerations

- Duck typing has zero runtime overhead over normal attribute access
- `hasattr()` internally calls `getattr()` and catches exceptions — slower than direct access
- `isinstance` checks are fast but couple code to specific types
- `Protocol` checks in mypy happen at static analysis time only

## Method overriding

### What It Is

Method overriding allows a subclass to provide a specific implementation of a method already defined in its parent class, enabling polymorphic behavior through inheritance.

### Why It Is Important

Overriding is how polymorphic behavior is achieved in class hierarchies. It allows a client to call the same method on different subclasses and get appropriate behavior without knowing the concrete type.

### How It Works Internally

The MRO determines which version of a method is called. Python searches the instance's class first, then parent classes in MRO order. The first match is used.

```python
class Shape:
    def area(self):
        return 0

class Circle(Shape):
    def __init__(self, r):
        self.r = r

    def area(self):
        return 3.14 * self.r ** 2

class Square(Shape):
    def __init__(self, s):
        self.s = s

    def area(self):
        return self.s ** 2

# Polymorphic call
shapes = [Circle(10), Square(5), Shape()]
for s in shapes:
    print(s.area())  # 314.0, 25, 0
```

### Syntax

```python
class Parent:
    def method(self):
        return "parent"

class Child(Parent):
    def method(self):
        return "child"
```

### Beginner Examples

```python
class PaymentProcessor:
    def process(self, amount):
        raise NotImplementedError

class CreditCard(PaymentProcessor):
    def process(self, amount):
        return f"Charged ${amount} to credit card"

class PayPal(PaymentProcessor):
    def process(self, amount):
        return f"Processed ${amount} via PayPal"

class Crypto(PaymentProcessor):
    def process(self, amount):
        return f"Transferred ${amount} in cryptocurrency"

def checkout(processor, amount):
    print(processor.process(amount))

checkout(CreditCard(), 100)
checkout(PayPal(), 50)
checkout(Crypto(), 200)
```

### Advanced Examples

```python
from abc import ABC, abstractmethod
from typing import List, Optional

class ReportGenerator(ABC):
    @abstractmethod
    def header(self, title: str) -> str:
        pass

    @abstractmethod
    def body(self, rows: List[dict]) -> str:
        pass

    @abstractmethod
    def footer(self) -> str:
        pass

    def generate(self, title: str, rows: List[dict]) -> str:
        parts = [
            self.header(title),
            self.body(rows),
            self.footer()
        ]
        return "\n".join(parts)

class HTMLReport(ReportGenerator):
    def header(self, title: str) -> str:
        return f"<html><head><title>{title}</title></head><body>"

    def body(self, rows: List[dict]) -> str:
        table = "<table>"
        for row in rows:
            table += "<tr>" + "".join(f"<td>{v}</td>" for v in row.values()) + "</tr>"
        return table + "</table>"

    def footer(self) -> str:
        return "</body></html>"

class MarkdownReport(ReportGenerator):
    def header(self, title: str) -> str:
        return f"# {title}\n\n"

    def body(self, rows: List[dict]) -> str:
        if not rows:
            return ""
        headers = "| " + " | ".join(rows[0].keys()) + " |\n"
        separator = "| " + " | ".join("---" for _ in rows[0]) + " |\n"
        data = "\n".join(
            "| " + " | ".join(str(v) for v in row.values()) + " |"
            for row in rows
        )
        return headers + separator + data

    def footer(self) -> str:
        return ""

data = [{"Name": "Alice", "Score": 95}, {"Name": "Bob", "Score": 87}]
print(HTMLReport().generate("Scores", data))
print(MarkdownReport().generate("Scores", data))
```

## Operator overloading

### What It Is

Operator overloading allows user-defined classes to define behavior for Python's built-in operators (+, -, *, [], etc.) by implementing special "magic" methods.

### Why It Is Important

Operator overloading makes custom objects behave intuitively with standard operators. It enables mathematical objects (vectors, matrices) to use arithmetic syntax and container-like objects to use indexing syntax.

### How It Works Internally

When Python encounters `a + b`, it calls `a.__add__(b)`. If that returns `NotImplemented`, Python tries `b.__radd__(a)`. This two-way protocol ensures symmetric operator behavior.

```python
x = 3 + 4
# Internally: int.__add__(3, 4)
```

### Syntax

```python
class Vector:
    def __add__(self, other): ...
    def __sub__(self, other): ...
    def __mul__(self, other): ...
    def __truediv__(self, other): ...
    def __getitem__(self, key): ...
    def __setitem__(self, key, value): ...
    def __len__(self): ...
```

### Beginner Examples

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Point(self.x + other.x, self.y + other.y)

    def __str__(self):
        return f"({self.x}, {self.y})"

    def __repr__(self):
        return f"Point({self.x}, {self.y})"

p1 = Point(1, 2)
p2 = Point(3, 4)
p3 = p1 + p2
print(p3)  # (4, 6)
```

### Intermediate Examples

```python
class Money:
    _exchange_rates = {"USD": 1.0, "EUR": 1.18, "GBP": 1.38}

    def __init__(self, amount, currency="USD"):
        self.amount = float(amount)
        self.currency = currency.upper()

    def _to_usd(self):
        return self.amount / self._exchange_rates[self.currency]

    def _from_usd(self, usd_amount):
        return Money(usd_amount * self._exchange_rates[self.currency], self.currency)

    def __add__(self, other):
        if not isinstance(other, Money):
            return NotImplemented
        usd_total = self._to_usd() + other._to_usd()
        return self._from_usd(usd_total)

    def __sub__(self, other):
        if not isinstance(other, Money):
            return NotImplemented
        usd_diff = self._to_usd() - other._to_usd()
        return self._from_usd(usd_diff)

    def __mul__(self, scalar):
        if not isinstance(scalar, (int, float)):
            return NotImplemented
        return Money(self.amount * scalar, self.currency)

    def __rmul__(self, scalar):
        return self.__mul__(scalar)

    def __eq__(self, other):
        if not isinstance(other, Money):
            return NotImplemented
        return abs(self._to_usd() - other._to_usd()) < 0.001

    def __str__(self):
        return f"{self.currency} {self.amount:.2f}"

    def __repr__(self):
        return f"Money({self.amount}, '{self.currency}')"

m1 = Money(100, "USD")
m2 = Money(50, "EUR")
m3 = m1 + m2
print(m3)          # USD 159.00
print(m1 * 3)      # USD 300.00
print(3 * m1)      # USD 300.00
```

### Advanced Examples

```python
class Matrix:
    def __init__(self, data):
        self.data = [list(row) for row in data]
        self.rows = len(self.data)
        self.cols = len(self.data[0]) if self.data else 0

    def __add__(self, other):
        if self.rows != other.rows or self.cols != other.cols:
            raise ValueError("Matrix dimensions must match")
        result = [
            [self.data[i][j] + other.data[i][j] for j in range(self.cols)]
            for i in range(self.rows)
        ]
        return Matrix(result)

    def __mul__(self, other):
        if isinstance(other, (int, float)):
            return Matrix([[v * other for v in row] for row in self.data])
        if isinstance(other, Matrix):
            if self.cols != other.rows:
                raise ValueError("Incompatible dimensions")
            result = [
                [
                    sum(self.data[i][k] * other.data[k][j] for k in range(self.cols))
                    for j in range(other.cols)
                ]
                for i in range(self.rows)
            ]
            return Matrix(result)
        return NotImplemented

    def __matmul__(self, other):
        return self.__mul__(other)

    def __getitem__(self, idx):
        return self.data[idx]

    def __setitem__(self, idx, value):
        self.data[idx] = list(value)

    def __len__(self):
        return self.rows

    def __str__(self):
        return "\n".join("[" + ", ".join(f"{v:4}" for v in row) + "]" for row in self.data)

    def __repr__(self):
        return f"Matrix({self.data})"

A = Matrix([[1, 2], [3, 4]])
B = Matrix([[5, 6], [7, 8]])
print(A + B)
print(A * B)
print(A * 2)
```

### Real-World Use Cases

- **NumPy arrays**: extensive operator overloading for array arithmetic
- **SQLAlchemy**: `==`, `!=`, etc. overloaded to build SQL expressions
- **pathlib**: `Path / "subdir"` constructs filesystem paths
- **Web framework query objects**: `query.filter(Model.field == value)`
- **Decimal/ Fraction types**: arithmetic operations on numeric types

### Common Mistakes

1. **Missing `__radd__`**: `2 + obj` won't work without reverse methods
2. **Returning `NotImplemented` incorrectly**: should return it, not raise it
3. **Mutating self**: `__add__` should return a new instance, not modify self
4. **Inconsistent behavior**: `__eq__` and `__hash__` must be consistent

### Best Practices

- Return `NotImplemented` for unsupported types (not raise TypeError)
- Always return a new instance from arithmetic operators
- Implement both forward and reverse methods (`__add__` + `__radd__`)
- Keep operator semantics consistent with built-in types
- Override `__eq__` and `__hash__` together for dict/set compatibility

### Performance Considerations

- Operator overloading adds a method call per operation
- For hot loops, direct attribute access is faster than overloaded operators
- NumPy avoids Python-level operator overhead with vectorized C operations
- Reverse method lookup (`__radd__`) adds an extra dispatch attempt

### Interview Questions

1. What is duck typing and how is it different from nominal typing?
2. How does Python decide which method to call in a polymorphic hierarchy?
3. Explain the `__add__`/`__radd__` protocol. Why is `__radd__` necessary?
4. What is the difference between method overriding and method overloading?
5. How do Protocols (Python 3.8+) enable static duck typing?

### Coding Challenges

1. Implement a `Polynomial` class with `__add__`, `__sub__`, `__mul__`, and `__call__`.
2. Build a `Shape` hierarchy with duck-typed `area()` and `perimeter()` methods.
3. Create a `CustomList` class that overloads slicing, addition, and multiplication like Python lists.

### Related Topics

Magic Methods, Inheritance, Abstract Base Classes, Protocols, Type Hints
