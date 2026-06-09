# Input and Output - input(), print(), formatted output

## Introduction

Input and output (I/O) operations allow programs to interact with users and external systems. Python provides the `input()` function for reading user input, the `print()` function for displaying output, and various methods for formatting output. Proper handling of input and output is essential for creating interactive and useful programs.

## Why It Is Important

I/O is fundamental because:
- It enables user interaction via the console
- It allows programs to read configuration and data files
- It formats output for readability and further processing
- It handles type conversion of user input
- It supports various output formats (strings, numbers, formatted tables)

## Syntax

```python
# Basic input
name = input("Enter your name: ")

# Basic output
print("Hello, World!")

# Formatted output
print(f"Hello, {name}!")
print("Hello, {}!".format(name))
print("Hello, %s!" % name)

# Multiple items
print("Value is", 42)

# Custom separator and end
print("a", "b", "c", sep="-", end="!\n")
```

## Examples

### The input() Function

```python
# Basic input
name = input("What is your name? ")
print(f"Hello, {name}!")

# input() always returns a string
age = input("How old are you? ")
print(f"Age type: {type(age)}")  # <class 'str'>

# Convert input to other types
age = int(input("Enter your age: "))
height = float(input("Enter your height in meters: "))
print(f"You are {age} years old and {height}m tall")

# Reading multiple inputs on one line
x, y = input("Enter two numbers: ").split()
x, y = int(x), int(y)
print(f"Sum: {x + y}")

# Reading multiple lines
print("Enter 3 numbers (one per line):")
numbers = [int(input()) for _ in range(3)]
print(f"Numbers: {numbers}")
```

## Beginner Examples

### Basic Input and Output

```python
# Simple greeting
name = input("What is your name? ")
print(f"Nice to meet you, {name}!")

# Simple calculator
num1 = float(input("Enter first number: "))
num2 = float(input("Enter second number: "))

print(f"Sum: {num1 + num2}")
print(f"Difference: {num1 - num2}")
print(f"Product: {num1 * num2}")
print(f"Quotient: {num1 / num2}")

# Age calculator
from datetime import datetime
birth_year = int(input("Enter your birth year: "))
current_year = datetime.now().year
age = current_year - birth_year
print(f"You are approximately {age} years old")

# Temperature converter
celsius = float(input("Enter temperature in Celsius: "))
fahrenheit = (celsius * 9/5) + 32
print(f"{celsius}°C = {fahrenheit}°F")
```

### The print() Function

```python
# Basic print
print("Hello, World!")

# Multiple arguments
print("The answer is", 42)

# Custom separator
print("apple", "banana", "cherry", sep=", ")
print("one", "two", "three", sep=" - ")

# Custom end character
print("Loading", end="")
print(".", end="")
print(".", end="")
print(".")
print("Done!")

# Printing special characters
print("Hello\nWorld")  # Newline
print("Column1\tColumn2")  # Tab

# Printing to a file
with open("output.txt", "w") as f:
    print("This goes to a file", file=f)
    print("This also goes to the file", file=f)
```

## Intermediate Examples

### Formatted Output

```python
# f-strings (Python 3.6+)
name = "Alice"
age = 30
score = 95.5678

print(f"Name: {name}")
print(f"Age: {age}")
print(f"Score: {score:.1f}")
print(f"Score: {score:.2f}")
print(f"Score: {score:.0f}")

# Alignment and padding
for i in range(1, 6):
    print(f"|{i:>5}|{i**2:<5}|{i**3:^5}|")

# Percentage formatting
rate = 0.8567
print(f"Rate: {rate:.1%}")  # 85.7%

# Thousands separator
large = 1234567
print(f"Large: {large:,}")

# Binary, octal, hex
val = 255
print(f"Binary: {val:b}")
print(f"Octal: {val:o}")
print(f"Hex: {val:x}")
print(f"Hex (upper): {val:X}")

# Using format() method
print("Name: {}, Age: {}".format(name, age))
print("Name: {n}, Age: {a}".format(n=name, a=age))
print("Score: {:.2f}".format(score))

# Old-style % formatting
print("Name: %s, Age: %d" % (name, age))
print("Score: %.2f" % score)
```

### Reading and Writing Files

```python
# Writing to a file
with open("data.txt", "w", encoding="utf-8") as f:
    f.write("Line 1\n")
    f.write("Line 2\n")
    f.write("Line 3\n")

# Reading entire file
with open("data.txt", "r", encoding="utf-8") as f:
    content = f.read()
print(f"File content:\n{content}")

# Reading line by line
with open("data.txt", "r", encoding="utf-8") as f:
    for line in f:
        print(f"Read: {line.strip()}")

# Reading all lines into a list
with open("data.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()
print(f"Lines: {lines}")

# Appending to a file
with open("data.txt", "a", encoding="utf-8") as f:
    f.write("Appended line\n")
```

## Advanced Examples

### Robust Input Handling

```python
# Input validation with try/except
def get_int(prompt):
    """Get an integer from the user with validation."""
    while True:
        try:
            return int(input(prompt))
        except ValueError:
            print("Invalid input. Please enter an integer.")

def get_float(prompt):
    """Get a float from the user with validation."""
    while True:
        try:
            return float(input(prompt))
        except ValueError:
            print("Invalid input. Please enter a number.")

def get_choice(prompt, options):
    """Get a valid choice from the user."""
    while True:
        user_input = input(prompt).strip().lower()
        if user_input in options:
            return user_input
        print(f"Please choose from: {', '.join(options)}")

age = get_int("Enter age: ")
print(f"Age: {age}")

height = get_float("Enter height: ")
print(f"Height: {height}")

color = get_choice("Favorite color (red/green/blue): ", ["red", "green", "blue"])
print(f"Color: {color}")
```

### Formatted Tables and Reports

```python
# Creating a formatted table
def print_table(header, data):
    """Print a formatted table."""
    # Calculate column widths
    col_widths = [len(h) for h in header]
    for row in data:
        for i, cell in enumerate(row):
            col_widths[i] = max(col_widths[i], len(str(cell)))
    
    # Print header
    header_line = " | ".join(h.ljust(w) for h, w in zip(header, col_widths))
    print(header_line)
    print("-" * len(header_line))
    
    # Print data rows
    for row in data:
        line = " | ".join(str(cell).ljust(w) for cell, w in zip(row, col_widths))
        print(line)

header = ["Name", "Age", "City", "Score"]
data = [
    ["Alice", 30, "New York", 95.5],
    ["Bob", 25, "Los Angeles", 87.3],
    ["Charlie", 35, "Chicago", 92.8],
    ["Diana", 28, "Houston", 88.1]
]
print_table(header, data)
```

### Progress Bar

```python
# Simple progress bar
def progress_bar(current, total, bar_length=30):
    """Display a progress bar."""
    fraction = current / total
    filled = int(bar_length * fraction)
    bar = "█" * filled + "░" * (bar_length - filled)
    percent = int(fraction * 100)
    print(f"\rProgress: |{bar}| {percent}% ({current}/{total})", end="")

import time
total = 50
for i in range(total + 1):
    progress_bar(i, total)
    time.sleep(0.05)
print()  # New line at the end

# Logging with timestamps
from datetime import datetime
def log(message, level="INFO"):
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    print(f"[{timestamp}] [{level}] {message}")

log("Application started")
log("Processing data...")
log("Error connecting to database", level="ERROR")
log("Operation completed")
```

## Real-World Use Cases

- **CLI Tools**: Interactive command-line applications
- **Data Processing**: Reading input files, writing output files
- **Configuration**: Reading settings from config files
- **Logging**: Writing timestamped log entries
- **User Interfaces**: Menus, prompts, form inputs
- **Data Export**: Writing CSV, JSON, or formatted reports
- **Testing**: Automated test input/output verification

## Common Mistakes

```python
# Mistake 1: Not converting input() type
age = input("Enter age: ")
# print(age + 5)  # TypeError: can only concatenate str (not "int") to str
# Correct:
age = int(input("Enter age: "))
print(age + 5)

# Mistake 2: Not handling EOF or keyboard interrupt
try:
    data = input("Enter data: ")
except EOFError:
    print("No input received")
except KeyboardInterrupt:
    print("\nUser cancelled input")

# Mistake 3: Forgetting newline characters
print("Hello")
print("World")  # Each on separate line
# To print on same line:
print("Hello", end=" ")
print("World")

# Mistake 4: Using print() for debugging in production
# Use logging module instead
import logging
logging.basicConfig(level=logging.DEBUG)
logging.debug("Debug message")
logging.info("Info message")

# Mistake 5: Not specifying encoding for file I/O
# Always specify encoding:
with open("file.txt", "r", encoding="utf-8") as f:
    content = f.read()

# Mistake 6: Assuming split() works as expected
data = input("Enter values: ").split()  # Splits on any whitespace
# For CSV input:
data = input("Enter CSV: ").split(",")
```

## Best Practices

- Always validate and convert `input()` return values
- Use `try/except` for robust input handling
- Use f-strings for modern, readable output formatting
- Specify encoding for file operations (`utf-8`)
- Use `with open(...)` for automatic file closing
- Use logging module instead of print for production code
- Use `sep` and `end` parameters of `print()` for formatting
- Prompt clearly so users know what to enter
- Strip whitespace from input with `.strip()`
- Handle `KeyboardInterrupt` and `EOFError` gracefully

## Interview Questions

1. How does `input()` work in Python?
2. What is the difference between `input()` in Python 2 and Python 3?
3. How do you format strings in Python?
4. What are f-strings and what features do they support?
5. How do you read multiple values from a single input line?
6. How do you handle input validation?
7. What is the difference between `print()` with and without `file` parameter?
8. How do you read a file line by line?
9. What is the `with` statement and why is it used for file I/O?
10. How do you create a progress bar in the console?

## Coding Challenges

```python
# Challenge 1: Simple menu system
def menu():
    options = {
        "1": "Say hello",
        "2": "Calculate sum",
        "3": "Exit"
    }
    while True:
        print("\nMenu:")
        for key, value in options.items():
            print(f"  {key}. {value}")
        choice = input("Choose: ").strip()
        if choice == "1":
            name = input("Your name: ")
            print(f"Hello, {name}!")
        elif choice == "2":
            nums = input("Enter two numbers: ").split()
            if len(nums) == 2:
                print(f"Sum: {float(nums[0]) + float(nums[1])}")
        elif choice == "3":
            print("Goodbye!")
            break
        else:
            print("Invalid choice")

# Challenge 2: CSV file reader
def read_csv(filename):
    with open(filename, "r", encoding="utf-8") as f:
        lines = f.readlines()
    if not lines:
        return []
    header = lines[0].strip().split(",")
    data = []
    for line in lines[1:]:
        values = line.strip().split(",")
        data.append(dict(zip(header, values)))
    return data

# Create sample CSV
with open("sample.csv", "w") as f:
    f.write("name,age,city\n")
    f.write("Alice,30,NYC\n")
    f.write("Bob,25,LA\n")

csv_data = read_csv("sample.csv")
for row in csv_data:
    print(f"{row['name']} ({row['age']}) from {row['city']}")

# Challenge 3: Multi-line input collector
def collect_paragraph():
    print("Enter your text (type 'END' on a new line to finish):")
    lines = []
    while True:
        line = input()
        if line.strip() == "END":
            break
        lines.append(line)
    return "\n".join(lines)

# Challenge 4: Formatted invoice generator
def generate_invoice(items):
    print("=" * 50)
    print(f"{'ITEM':<20} {'QTY':>5} {'PRICE':>10} {'TOTAL':>10}")
    print("=" * 50)
    grand_total = 0
    for name, qty, price in items:
        total = qty * price
        grand_total += total
        print(f"{name:<20} {qty:>5} {price:>10.2f} {total:>10.2f}")
    print("=" * 50)
    print(f"{'GRAND TOTAL':>35} {grand_total:>10.2f}")
    print("=" * 50)

items = [("Apple", 10, 0.50), ("Banana", 5, 0.30), ("Orange", 8, 0.80)]
generate_invoice(items)

# Challenge 5: Calculator with history
def calculator_with_history():
    history = []
    print("Calculator (type 'hist' for history, 'exit' to quit)")
    while True:
        expr = input("> ").strip()
        if expr.lower() == "exit":
            break
        if expr.lower() == "hist":
            for h in history[-5:]:
                print(f"  {h}")
            continue
        try:
            result = eval(expr)
            print(f"= {result}")
            history.append(f"{expr} = {result}")
        except Exception as e:
            print(f"Error: {e}")
```

## Summary

Input and output operations in Python use `input()` for reading user input and `print()` for displaying output. Input is always returned as a string and must be converted for numeric operations. Output can be formatted using f-strings, `.format()`, or `%`-formatting. Proper validation, encoding handling, and formatting make I/O robust and user-friendly.

## Related Topics

- String Formatting
- File I/O
- Error Handling (try/except)
- Data Type Conversion
- Command-Line Arguments
- Logging Module
- CSV and JSON Handling
