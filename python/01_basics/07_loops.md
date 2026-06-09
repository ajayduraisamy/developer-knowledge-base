# Loops - for, while, break, continue, else clause

## Introduction

Loops are programming constructs that repeat a block of code multiple times. Python provides two main loop types: `for` loops (for iterating over sequences) and `while` loops (for repeating while a condition is true). Python also offers loop control statements (`break`, `continue`) and the `else` clause on loops, which executes when the loop completes normally (without `break`).

## Why It Is Important

Loops are essential because:
- They automate repetitive tasks
- They process collections of data (lists, files, databases)
- They enable efficient batch processing
- They implement algorithms that require iteration
- They reduce code duplication
- Combined with `range()` and `enumerate()`, they provide powerful iteration patterns

## Syntax

```python
# For loop with range
for i in range(start, stop, step):
    statement(s)

# For loop over sequence
for item in iterable:
    statement(s)

# For loop with enumerate
for index, value in enumerate(iterable):
    statement(s)

# While loop
while condition:
    statement(s)

# Loop control
break       # Exit loop immediately
continue    # Skip to next iteration

# Else clause on loops
for item in iterable:
    statement(s)
else:
    # Runs only if loop completed without break
    statement(s)
```

## Examples

### Basic For Loops

```python
# Range-based for loop
print("Counting from 0 to 4:")
for i in range(5):
    print(i, end=" ")
print()

# Range with start and stop
print("\nCounting from 2 to 8:")
for i in range(2, 9):
    print(i, end=" ")
print()

# Range with step
print("\nEven numbers from 0 to 10:")
for i in range(0, 11, 2):
    print(i, end=" ")
print()

# Reverse range
print("\nCountdown from 5 to 1:")
for i in range(5, 0, -1):
    print(i, end=" ")
print("\nBlast off!")
```

### For Loops over Sequences

```python
# Iterating over a list
fruits = ["apple", "banana", "cherry", "date"]
print("Fruits:")
for fruit in fruits:
    print(f"  - {fruit}")

# Iterating over a string
message = "Hello"
print("\nCharacters in 'Hello':")
for char in message:
    print(f"  '{char}' (ASCII: {ord(char)})")

# Iterating over a dictionary
person = {"name": "Alice", "age": 30, "city": "New York"}
print("\nDictionary keys and values:")
for key, value in person.items():
    print(f"  {key}: {value}")

# Iterating over a set
unique_numbers = {3, 1, 4, 1, 5, 9, 2}
print("\nSet iteration (unordered):")
for num in unique_numbers:
    print(f"  {num}")
```

## Beginner Examples

```python
# Sum of numbers from 1 to 100
total = 0
for i in range(1, 101):
    total += i
print(f"Sum of 1 to 100: {total}")

# Multiplication table
num = int(input("Enter a number: "))
print(f"Multiplication table for {num}:")
for i in range(1, 11):
    print(f"{num} x {i} = {num * i}")

# Factorial calculation
n = int(input("Enter a number: "))
factorial = 1
for i in range(1, n + 1):
    factorial *= i
print(f"{n}! = {factorial}")

# Fibonacci series
terms = int(input("How many Fibonacci terms? "))
a, b = 0, 1
print("Fibonacci series:")
for _ in range(terms):
    print(a, end=" ")
    a, b = b, a + b
print()

# Pattern printing
rows = 5
print("\nRight triangle pattern:")
for i in range(1, rows + 1):
    print("*" * i)

# While loop example - number guessing
import random
secret = random.randint(1, 20)
guess = -1
attempts = 0

while guess != secret:
    guess = int(input("Guess (1-20): "))
    attempts += 1
    if guess < secret:
        print("Too low!")
    elif guess > secret:
        print("Too high!")

print(f"Correct! You got it in {attempts} attempts!")
```

## Intermediate Examples

```python
# Enumerate for index and value
colors = ["red", "green", "blue", "yellow"]
print("Colors with indices:")
for index, color in enumerate(colors):
    print(f"  {index}: {color}")

# Enumerate with custom start index
for index, color in enumerate(colors, start=1):
    print(f"  {index}. {color}")

# Using break to find first occurrence
numbers = [5, 8, 12, 3, 9, 15, 7]
target = 9

for i, num in enumerate(numbers):
    if num == target:
        print(f"Found {target} at index {i}")
        break
else:
    print(f"{target} not found")  # Only runs if break wasn't hit

# Using continue to skip items
print("\nOdd numbers:")
for i in range(1, 21):
    if i % 2 == 0:
        continue  # Skip even numbers
    print(i, end=" ")
print()

# Nested loops - multiplication table
print("\nMultiplication table (1-10):")
for i in range(1, 11):
    for j in range(1, 11):
        print(f"{i * j:4d}", end="")
    print()

# While loop with sentinel
print("\nEnter numbers to sum (0 to stop):")
total = 0
while True:
    num = int(input("Enter number: "))
    if num == 0:
        break
    total += num
    print(f"Running total: {total}")
print(f"Final sum: {total}")

# Iterating multiple sequences with zip
names = ["Alice", "Bob", "Charlie", "Diana"]
scores = [85, 92, 78, 95]
grades = ["B", "A", "C", "A"]

print("\nStudent records:")
for name, score, grade in zip(names, scores, grades):
    print(f"  {name}: Score={score}, Grade={grade}")
```

## Advanced Examples

```python
# List comprehension vs for loop (performance comparison)
import time

# Using for loop
numbers = range(1_000_000)
squares_loop = []
start = time.time()
for n in numbers:
    squares_loop.append(n ** 2)
loop_time = time.time() - start

# Using list comprehension
start = time.time()
squares_comp = [n ** 2 for n in numbers]
comp_time = time.time() - start

print(f"For loop: {loop_time:.4f}s")
print(f"Comprehension: {comp_time:.4f}s")
print(f"Comprehension is {loop_time/comp_time:.1f}x faster")

# Generator expression (memory efficient)
def fibonacci_generator(limit):
    a, b = 0, 1
    count = 0
    while count < limit:
        yield a
        a, b = b, a + b
        count += 1

print("\nFibonacci with generator:")
for fib in fibonacci_generator(15):
    print(fib, end=" ")
print()

# Custom iterator class
class Countdown:
    def __init__(self, start):
        self.current = start
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current < 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value

print("\nCountdown iterator:")
for num in Countdown(5):
    print(num, end=" ")
print()

# Break with else clause (prime number check)
def is_prime(n):
    if n <= 1:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            print(f"{n} is divisible by {i}")
            break
    else:
        return True  # Executed if no break occurred
    return False

for num in [17, 25, 37, 49]:
    print(f"{num} is prime: {is_prime(num)}")
```

## Real-World Use Cases

- **Data Processing**: Iterating through CSV rows, JSON arrays, database results
- **Web Scraping**: Looping through pages of a website
- **File Operations**: Reading/writing multiple files, processing log files
- **API Calls**: Paginating through API responses, retrying failed requests
- **Batch Processing**: Processing records in chunks for performance
- **Game Loops**: Main game loop that updates and renders frames
- **GUI Programming**: Event loops that handle user interactions

## Common Mistakes

```python
# Mistake 1: Modifying list while iterating
numbers = [1, 2, 3, 4, 5]
# Wrong:
for num in numbers:
    if num % 2 == 0:
        numbers.remove(num)  # Skips elements!
print(numbers)  # [1, 3, 5] but actually [1, 3, 4, 5]?

# Correct:
numbers = [1, 2, 3, 4, 5]
numbers[:] = [num for num in numbers if num % 2 != 0]
print(numbers)  # [1, 3, 5]

# Mistake 2: Infinite while loops
# while True:
#     print("Infinite!")  # No break condition

# Mistake 3: Off-by-one with range
# Wrong - prints 1 to 10:
for i in range(1, 11):
    print(i)  # Correct!

# Wrong - skips last element:
items = ["a", "b", "c", "d"]
for i in range(len(items) - 1):
    print(items[i])  # Only prints a, b, c

# Mistake 4: Using mutable default in loop
# def add_item(item, items=[]):
#     items.append(item)
#     return items

# Mistake 5: Confusing break and continue
# break: exits the loop entirely
# continue: skips to the next iteration

# Mistake 6: Not using enumerate when index is needed
fruits = ["apple", "banana", "cherry"]
# Wrong:
for i in range(len(fruits)):
    print(f"{i}: {fruits[i]}")
# Correct:
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")
```

## Best Practices

- Use `for` loops when iterating over sequences; use `while` loops when the number of iterations is unknown
- Use `enumerate()` when you need both index and value
- Use `zip()` to iterate over multiple sequences in parallel
- Use `range()` for numeric loops
- Prefer list comprehensions for simple transformations
- Use `break` and `continue` judiciously; too many can make code hard to follow
- Use `else` clause on loops sparingly; it can be confusing
- Avoid modifying collections while iterating over them
- Use descriptive loop variable names (`for student in students:` not `for s in st:`)
- Consider using `itertools` for complex iteration patterns
- Use `_` for loop variables that aren't needed
- Consider memory usage with large sequences (use generators)

## Interview Questions

1. What is the difference between `for` and `while` loops?
2. How does `range()` work? What are its parameters?
3. What is the `else` clause on loops and when does it execute?
4. How do you iterate over a dictionary in Python?
5. What is the difference between `break` and `continue`?
6. How do you iterate over multiple lists simultaneously?
7. What are generators and how do they differ from regular loops?
8. How do you create an infinite loop in Python?
9. What is the `enumerate()` function and why is it useful?
10. How do you iterate in reverse over a sequence?

## Coding Challenges

1. Write a program that finds all prime numbers up to a given number.
2. Create a program that prints a pyramid pattern using asterisks.
3. Write a program that finds the GCD of two numbers using a loop.
4. Create a program that validates a password using multiple conditions in a loop.
5. Write a program that transposes a matrix using nested loops.

```python
# Challenge 1: Prime Numbers (Sieve of Eratosthenes)
def sieve_of_eratosthenes(limit):
    is_prime = [True] * (limit + 1)
    is_prime[0] = is_prime[1] = False
    
    for i in range(2, int(limit ** 0.5) + 1):
        if is_prime[i]:
            for j in range(i * i, limit + 1, i):
                is_prime[j] = False
    
    return [i for i in range(limit + 1) if is_prime[i]]

primes = sieve_of_eratosthenes(100)
print(f"Primes up to 100: {primes}")

# Challenge 2: Pyramid Pattern
def print_pyramid(rows):
    for i in range(1, rows + 1):
        spaces = " " * (rows - i)
        stars = "*" * (2 * i - 1)
        print(spaces + stars)

print("Pyramid pattern:")
print_pyramid(5)

# Challenge 3: GCD using Euclidean Algorithm
def gcd(a, b):
    while b:
        a, b = b, a % b
    return a

print(f"\nGCD(48, 18): {gcd(48, 18)}")  # 6
print(f"GCD(56, 98): {gcd(56, 98)}")    # 14

# Challenge 4: Password Validation Loop
def get_valid_password():
    while True:
        password = input("Enter password (8+ chars, upper, lower, digit): ")
        errors = []
        
        if len(password) < 8:
            errors.append("Too short")
        if not any(c.isupper() for c in password):
            errors.append("Need uppercase")
        if not any(c.islower() for c in password):
            errors.append("Need lowercase")
        if not any(c.isdigit() for c in password):
            errors.append("Need digit")
        
        if not errors:
            print("Valid password!")
            return password
        print("Errors:", ", ".join(errors))

# Challenge 5: Matrix Transposition
def transpose_matrix(matrix):
    rows = len(matrix)
    cols = len(matrix[0])
    result = [[0] * rows for _ in range(cols)]
    
    for i in range(rows):
        for j in range(cols):
            result[j][i] = matrix[i][j]
    
    return result

original = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
transposed = transpose_matrix(original)
print("\nOriginal matrix:")
for row in original:
    print(row)
print("Transposed matrix:")
for row in transposed:
    print(row)

# Bonus: Using itertools
from itertools import cycle, count, accumulate, product

# Cycle through items
print("\nCycle stoplight:")
colors = cycle(["red", "yellow", "green"])
for i, color in enumerate(colors):
    if i >= 6:
        break
    print(f"  {color}")

# Infinite counter
print("\nFirst 5 powers of 2:")
for i in count(start=1):
    if i > 5:
        break
    print(f"  2^{i} = {2**i}")

# Accumulate
print("\nRunning total:")
numbers = [1, 2, 3, 4, 5]
for total in accumulate(numbers):
    print(f"  {total}")
```

## Summary

Loops in Python (`for` and `while`) provide powerful mechanisms for repeating code blocks. `For` loops excel at iterating over sequences and work well with `range()`, `enumerate()`, and `zip()`. `While` loops are ideal for condition-based repetition. Loop control statements (`break`, `continue`, `else`) add flexibility, while generators and comprehensions offer more Pythonic alternatives in many cases. Understanding when and how to use each loop type is fundamental to writing efficient Python code.

## Related Topics

- Range Function
- Enumerate Function
- Zip Function
- List Comprehensions
- Generators and Yield
- Iterators and Iterables
- itertools Module
- While vs For Loops
