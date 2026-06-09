# Conditions - if, elif, else, ternary operator

## Introduction

Conditional statements allow programs to make decisions and execute different code paths based on certain conditions. Python uses `if`, `elif` (short for "else if"), and `else` keywords to create conditional branches. Python also supports a ternary conditional expression for simple inline conditions, and relies on truthy/falsy value evaluation.

## Why It Is Important

Conditional logic is essential because:
- It enables decision-making in programs
- It controls program flow based on user input, data, or state
- It allows handling of edge cases and error conditions
- It implements business rules and validation logic
- It determines which code executes under which circumstances
- Short-circuit evaluation can optimize complex conditions

## Syntax

```python
# Basic if statement
if condition:
    statement(s)

# if-else
if condition:
    statement(s)
else:
    statement(s)

# if-elif-else
if condition1:
    statement(s)
elif condition2:
    statement(s)
else:
    statement(s)

# Nested if
if condition1:
    if condition2:
        statement(s)

# Ternary (conditional expression)
value_if_true if condition else value_if_false
```

## Examples

### Basic if Statements

```python
# Simple if
age = 18
if age >= 18:
    print("You are an adult.")

# if-else
temperature = 30
if temperature > 25:
    print("It's hot outside!")
else:
    print("It's cool outside.")

# if-elif-else
score = 85
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"
print(f"Score: {score}, Grade: {grade}")
```

### Truthy and Falsy Values

```python
# Python considers these values as Falsy:
# - False
# - None
# - 0, 0.0, 0j
# - Empty sequences: "", [], (), {}
# - Empty sets: set()
# - Everything else is Truthy

falsy_values = [False, None, 0, 0.0, "", [], (), {}, set()]
for value in falsy_values:
    if value:
        print(f"  {repr(value)} is Truthy")
    else:
        print(f"  {repr(value)} is Falsy")

# Using truthiness in conditions
name = ""
if name:  # Equivalent to: if len(name) > 0:
    print(f"Hello, {name}!")
else:
    print("Name is empty!")

items = []
if not items:  # Equivalent to: if len(items) == 0:
    print("No items in list!")
```

## Beginner Examples

```python
# Number guessing game
import random

secret = random.randint(1, 10)
guess = int(input("Guess a number between 1 and 10: "))

if guess == secret:
    print("Correct! You guessed it!")
elif abs(guess - secret) <= 2:
    print(f"Close! The number was {secret}")
else:
    print(f"Sorry! The number was {secret}")

# Even/Odd checker
number = int(input("Enter a number: "))
if number % 2 == 0:
    print(f"{number} is even")
else:
    print(f"{number} is odd")

# Simple login system
username = input("Username: ")
password = input("Password: ")

if username == "admin" and password == "secret":
    print("Welcome, admin!")
elif username == "admin":
    print("Wrong password!")
elif password == "secret":
    print("Unknown username!")
else:
    print("Access denied!")

# Grade calculator
score = float(input("Enter your score (0-100): "))

if score < 0 or score > 100:
    print("Invalid score!")
elif score >= 90:
    print("A - Excellent!")
elif score >= 80:
    print("B - Good!")
elif score >= 70:
    print("C - Average")
elif score >= 60:
    print("D - Below Average")
else:
    print("F - Failed")
```

## Intermediate Examples

```python
# Short-circuit evaluation
def validate_user(user):
    """Check if user is valid without unnecessary checks."""
    return user is not None and user.get("is_active") and "admin" in user.get("roles", [])

user1 = {"name": "Alice", "is_active": True, "roles": ["admin", "user"]}
user2 = {"name": "Bob", "is_active": False, "roles": ["user"]}
user3 = None

print(f"User 1 valid: {validate_user(user1)}")  # True
print(f"User 2 valid: {validate_user(user2)}")  # False
print(f"User 3 valid: {validate_user(user3)}")  # False (short-circuits on is not None)

# Ternary operator examples
age = 20
status = "Adult" if age >= 18 else "Minor"
print(status)

# Nested ternary (use sparingly)
x = 15
result = "positive" if x > 0 else "negative" if x < 0 else "zero"
print(f"x is {result}")

# Multiple conditions with logical operators
year = 2024
if (year % 4 == 0 and year % 100 != 0) or (year % 400 == 0):
    print(f"{year} is a leap year")
else:
    print(f"{year} is not a leap year")

# Using 'in' for membership testing
fruit = "apple"
available_fruits = ["apple", "banana", "cherry", "date"]
if fruit in available_fruits:
    print(f"{fruit} is available!")
else:
    print(f"{fruit} is not available!")

# Chained comparisons
x = 50
if 0 < x < 100:
    print(f"{x} is between 0 and 100")

# Conditional expressions in list comprehensions
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
even_odd = ["even" if n % 2 == 0 else "odd" for n in numbers]
print(even_odd)
```

## Advanced Examples

```python
# Match-case statement (Python 3.10+)
def http_status_handler(status_code):
    match status_code:
        case 200:
            return "OK"
        case 201:
            return "Created"
        case 301 | 302:
            return "Redirect"
        case 400:
            return "Bad Request"
        case 401 | 403:
            return "Unauthorized/Forbidden"
        case 404:
            return "Not Found"
        case 500:
            return "Internal Server Error"
        case _:
            return "Unknown Status"

print(http_status_handler(200))  # OK
print(http_status_handler(404))  # Not Found
print(http_status_handler(999))  # Unknown Status

# Match-case with patterns
def process_command(command):
    match command.split():
        case ["quit"]:
            return "Quitting..."
        case ["hello", name]:
            return f"Hello, {name}!"
        case ["add", *items]:
            return f"Adding: {', '.join(items)}"
        case ["delete", item]:
            return f"Deleting: {item}"
        case _:
            return "Unknown command"

print(process_command("hello Alice"))     # Hello, Alice!
print(process_command("add apple banana"))  # Adding: apple, banana
print(process_command("quit"))             # Quitting...

# Conditional logic with dictionaries (replacing long if-elif chains)
def get_grade(score):
    grades = {
        90: "A",
        80: "B",
        70: "C",
        60: "D",
    }
    for threshold, grade in sorted(grades.items(), reverse=True):
        if score >= threshold:
            return grade
    return "F"

print(get_grade(95))  # A
print(get_grade(72))  # C
print(get_grade(45))  # F

# Using conditional expressions for early returns
def divide(a, b):
    if b == 0:
        return "Error: Division by zero"
    return a / b

# Comprehensive validation function
def validate_password(password):
    """Validate password strength with clear error messages."""
    errors = []
    
    if len(password) < 8:
        errors.append("Password must be at least 8 characters")
    
    if not any(c.isupper() for c in password):
        errors.append("Password must contain an uppercase letter")
    
    if not any(c.islower() for c in password):
        errors.append("Password must contain a lowercase letter")
    
    if not any(c.isdigit() for c in password):
        errors.append("Password must contain a digit")
    
    if not any(c in "!@#$%^&*" for c in password):
        errors.append("Password must contain a special character")
    
    if errors:
        print("Password validation failed:")
        for error in errors:
            print(f"  - {error}")
        return False
    
    print("Password is strong!")
    return True

validate_password("weak")
print("---")
validate_password("Str0ng!Pass")
```

## Real-World Use Cases

- **Form Validation**: Checking user input meets requirements
- **User Authentication**: Verifying credentials and permissions
- **Business Logic**: Applying pricing rules, discounts, eligibility
- **Error Handling**: Deciding how to respond to different error types
- **Game Development**: Checking game states, collisions, win conditions
- **Data Processing**: Filtering, categorizing, and transforming data
- **API Development**: Routing requests, checking auth tokens, validating payloads

## Common Mistakes

```python
# Mistake 1: Using assignment (=) instead of comparison (==)
# if x = 5:  # SyntaxError

# Mistake 2: Forgetting colon at end of if/elif/else lines
# if x > 5  # SyntaxError
#     print("x is large")

# Mistake 3: Using multiple if instead of elif
x = 10
if x > 5:
    print("Greater than 5")
if x > 8:  # This should be elif to be mutually exclusive
    print("Greater than 8")
# Both print! Changed to elif, only one would print

# Mistake 4: Unnecessary comparison with True/False
is_active = True
# Don't do this:
if is_active == True:
    print("Active")
# Do this:
if is_active:
    print("Active")

# Mistake 5: Using == for None comparison
value = None
# Don't:
if value == None:
    print("None")
# Do:
if value is None:
    print("None")

# Mistake 6: Not considering short-circuit evaluation
# If expensive_func() has side effects, be careful with:
if condition or expensive_func():
    pass

# Mistake 7: Overly complex conditions
# Bad:
if (age >= 18 and age <= 65) or (has_permission and not is_banned):
    pass
# Better:
is_working_age = 18 <= age <= 65
has_access = has_permission and not is_banned
if is_working_age or has_access:
    pass
```

## Best Practices

- Prefer `elif` over nested `if` for mutually exclusive conditions
- Use `is` for `None` comparisons, not `==`
- Use truthiness checks instead of explicit comparisons:
  - `if items:` instead of `if len(items) > 0:`
  - `if not items:` instead of `if len(items) == 0:`
  - `if value:` instead of `if value == True:`
- Keep conditions simple and readable; extract complex logic to variables
- Avoid deep nesting (3+ levels); consider early returns
- Use the ternary operator only for simple, readable expressions
- Cover all branches in if-elif chains (always include an `else`)
- Use `in` for membership testing
- Leverage `any()` and `all()` for collection conditions
- Consider dictionary dispatch for long if-elif chains
- Use match-case (Python 3.10+) for pattern matching

## Interview Questions

1. What are truthy and falsy values in Python?
2. Explain short-circuit evaluation with examples.
3. What is the ternary operator and how do you use it?
4. How do you write a conditional expression in a list comprehension?
5. What is the difference between `if` and `elif`?
6. Explain the match-case statement (Python 3.10+).
7. How do you check if a value is None in Python?
8. What is the best way to replace a long if-elif chain?
9. How does Python handle multiple conditions with `and`/`or`?
10. Explain how to use `all()` and `any()` with conditions.

## Coding Challenges

1. Write a program that determines if a year is a leap year.
2. Create a grading system with letter grades based on percentage scores.
3. Write a password strength checker.
4. Implement a simple calculator with operations based on user choice.
5. Create a program that categorizes a triangle based on its side lengths.

```python
# Challenge 1: Leap Year Checker
def is_leap_year(year):
    if year % 400 == 0:
        return True
    elif year % 100 == 0:
        return False
    elif year % 4 == 0:
        return True
    else:
        return False

for year in [1900, 2000, 2024, 2025]:
    print(f"{year}: {'Leap' if is_leap_year(year) else 'Not leap'}")

# Challenge 2: Grade Calculator
def calculate_grade(score):
    if not isinstance(score, (int, float)):
        return "Invalid input"
    if score < 0 or score > 100:
        return "Score out of range"
    
    if score >= 90: return "A"
    elif score >= 80: return "B"
    elif score >= 70: return "C"
    elif score >= 60: return "D"
    else: return "F"

scores = [95, 83, 72, 61, 45, 101, -5]
for s in scores:
    print(f"Score {s}: Grade {calculate_grade(s)}")

# Challenge 3: Password Strength Checker
def password_strength(password):
    if len(password) < 6:
        return "Weak"
    
    has_upper = any(c.isupper() for c in password)
    has_lower = any(c.islower() for c in password)
    has_digit = any(c.isdigit() for c in password)
    has_special = any(c in "!@#$%^&*" for c in password)
    
    strength = sum([has_upper, has_lower, has_digit, has_special])
    
    if strength == 4 and len(password) >= 12:
        return "Very Strong"
    elif strength >= 3:
        return "Strong"
    elif strength >= 2:
        return "Medium"
    else:
        return "Weak"

passwords = ["abc", "Password1", "Str0ng!Pass#", "12345678"]
for p in passwords:
    print(f"'{p}': {password_strength(p)}")

# Challenge 4: Simple Calculator Menu
def menu_calculator():
    print("Select operation:")
    print("1. Add")
    print("2. Subtract")
    print("3. Multiply")
    print("4. Divide")
    
    choice = input("Enter choice (1/2/3/4): ")
    num1 = float(input("Enter first number: "))
    num2 = float(input("Enter second number: "))
    
    if choice == '1':
        print(f"{num1} + {num2} = {num1 + num2}")
    elif choice == '2':
        print(f"{num1} - {num2} = {num1 - num2}")
    elif choice == '3':
        print(f"{num1} * {num2} = {num1 * num2}")
    elif choice == '4':
        if num2 != 0:
            print(f"{num1} / {num2} = {num1 / num2}")
        else:
            print("Cannot divide by zero!")
    else:
        print("Invalid choice!")

# Challenge 5: Triangle Categorizer
def triangle_type(a, b, c):
    """Categorize a triangle by its side lengths."""
    # Check if valid triangle
    if (a + b <= c) or (a + c <= b) or (b + c <= a):
        return "Not a valid triangle"
    
    if a == b == c:
        return "Equilateral"
    elif a == b or a == c or b == c:
        return "Isosceles"
    else:
        return "Scalene"

triangles = [
    (5, 5, 5),
    (5, 5, 8),
    (3, 4, 5),
    (1, 1, 3),
]
for sides in triangles:
    print(f"{sides}: {triangle_type(*sides)}")
```

## Summary

Conditional statements in Python, using `if`, `elif`, and `else`, allow programs to make decisions and execute different code paths. Python evaluates truthy/falsy values, supports short-circuit evaluation with `and`/`or`, and provides a ternary operator for simple inline conditions. The match-case statement (Python 3.10+) adds powerful pattern matching capabilities. Writing clean, readable, and efficient conditional logic is a cornerstone of effective Python programming.

## Related Topics

- Boolean Logic and Operators
- Truthy and Falsy Values
- Short-Circuit Evaluation
- Match-Case (Structural Pattern Matching)
- Control Flow and Loops
- Error Handling with try/except
