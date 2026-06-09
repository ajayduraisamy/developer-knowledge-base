# Strings - str() constructor, slicing, f-strings, methods

## Introduction

Strings are sequences of characters used to represent text data in Python. They are immutable (cannot be changed after creation) and support a wide range of operations including slicing, concatenation, formatting, and numerous built-in methods. Python provides several ways to create and manipulate strings, making text processing straightforward and powerful.

## Why It Is Important

Strings are essential because:
- They handle all text processing in applications
- They're used for user input/output, file I/O, and data serialization
- Python's string methods make text manipulation easy
- String formatting produces readable output
- String processing is fundamental to web development, data analysis, and automation

## Syntax

```python
# String creation
single_quotes = 'Hello'
double_quotes = "Hello"
multi_line = """This is
a multi-line
string"""
triple_single = '''Also multi-line'''

# Raw strings (backslashes are literal)
raw = r"C:\Users\Name\Documents"

# F-strings (Python 3.6+)
name = "Alice"
greeting = f"Hello, {name}!"

# String slicing
text = "Python"
substring = text[0:3]  # "Pyt"
reverse = text[::-1]   # "nohtyP"

# String methods
text.upper()
text.lower()
text.split(",")
" ".join(["a", "b", "c"])
text.replace("old", "new")
```

## Examples

### String Creation and Basic Operations

```python
# Different ways to create strings
str1 = 'Hello'
str2 = "World"
str3 = '''Multi-line
string example'''
str4 = """Also a
multi-line string"""

print(str1, str2)
print(str3)

# String concatenation
full = str1 + " " + str2
print(full)  # "Hello World"

# String repetition
print("Ha" * 3)  # "HaHaHa"

# String length
text = "Python Programming"
print(f"Length: {len(text)}")

# Escape sequences
print("Line1\nLine2")       # Newline
print("Tab\tseparated")     # Tab
print("Backslash: \\")      # Backslash
print("Quote: \"Hello\"")   # Double quote inside string

# Raw strings (for file paths, regex)
path = r"C:\Users\Name\Documents"
print(path)  # C:\Users\Name\Documents (backslashes preserved)
```

### String Indexing and Slicing

```python
text = "Python Programming"
print(f"String: '{text}'")
print(f"Length: {len(text)}")

# Indexing
print(f"First char: text[0] = '{text[0]}'")
print(f"Last char: text[-1] = '{text[-1]}'")
print(f"Second char: text[1] = '{text[1]}'")
print(f"Second to last: text[-2] = '{text[-2]}'")

# Slicing [start:stop:step]
print(f"text[0:6] = '{text[0:6]}'")       # "Python"
print(f"text[7:18] = '{text[7:18]}'")     # "Programming"
print(f"text[:6] = '{text[:6]}'")         # "Python" (from start)
print(f"text[7:] = '{text[7:]}'")         # "Programming" (to end)
print(f"text[:] = '{text[:]}'")           # Full copy
print(f"text[::2] = '{text[::2]}'")       # Every other char
print(f"text[::-1] = '{text[::-1]}'")     # Reversed

# Slicing with negative indices
print(f"text[-6:] = '{text[-6:]}'")       # Last 6 chars
print(f"text[:-7] = '{text[:-7]}'")       # All but last 7
```

## Beginner Examples

```python
# String methods for common operations
text = "  Hello, Python World!  "
print(f"Original: '{text}'")

# Case conversion
print(f"upper(): '{text.upper()}'")
print(f"lower(): '{text.lower()}'")
print(f"title(): '{text.title()}'")
print(f"capitalize(): '{text.capitalize()}'")
print(f"swapcase(): '{text.swapcase()}'")

# Whitespace removal
print(f"strip(): '{text.strip()}'")
print(f"lstrip(): '{text.lstrip()}'")
print(f"rstrip(): '{text.rstrip()}'")

# Finding and replacing
sentence = "The quick brown fox jumps over the lazy dog"
print(f"find('fox'): {sentence.find('fox')}")         # 16
print(f"find('cat'): {sentence.find('cat')}")         # -1 (not found)
print(f"index('fox'): {sentence.index('fox')}")       # 16
print(f"count('the'): {sentence.count('the')}")       # 2
print(f"replace('fox', 'cat'): {sentence.replace('fox', 'cat')}")

# Checking string properties
word = "Python3"
print(f"isalpha(): {word.isalpha()}")      # False (has digits)
print(f"isdigit(): {word.isdigit()}")      # False
print(f"isalnum(): {word.isalnum()}")      # True
print(f"isspace(): {'   '.isspace()}")     # True
print(f"startswith('Py'): {word.startswith('Py')}")  # True
print(f"endswith('3'): {word.endswith('3')}")         # True
```

## Intermediate Examples

```python
# String formatting methods
name = "Alice"
age = 30
score = 95.5678

# f-strings (Python 3.6+)
print(f"Name: {name}, Age: {age}, Score: {score:.1f}")
print(f"Total: {score * 10:.2f}")

# str.format()
print("Name: {}, Age: {}, Score: {:.1f}".format(name, age, score))
print("Name: {n}, Score: {s:.1f}, Age: {a}".format(n=name, a=age, s=score))

# %-formatting (older style)
print("Name: %s, Age: %d, Score: %.1f" % (name, age, score))

# Split and join
csv_data = "apple,banana,cherry,date"
fruits = csv_data.split(",")
print(f"Split: {fruits}")

# Join back
rejoined = " | ".join(fruits)
print(f"Joined: {rejoined}")

# Multi-line strings and dedent
import textwrap
long_text = """
    This is a long text
    that spans multiple lines
    and has indentation.
"""
print(textwrap.dedent(long_text))

# String alignment
text = "Python"
print(f"|{text:<20}|")   # Left align
print(f"|{text:>20}|")   # Right align
print(f"|{text:^20}|")   # Center
print(f"|{text:*^20}|")  # Center with fill character

# Checking prefixes and suffixes
url = "https://www.example.com/page.html"
print(f"Is HTTPS? {url.startswith('https')}")
print(f"Is .html? {url.endswith('.html')}")
```

## Advanced Examples

```python
# String internment and immutability
a = "hello"
b = "hello"
print(f"a is b: {a is b}")  # True (CPython interns short strings)

c = "hello world!"
d = "hello world!"
print(f"c is d: {c is d}")  # False or True (implementation dependent)

# Strings are immutable
s = "Hello"
# s[0] = "J"  # TypeError: 'str' object does not support item assignment
s = "J" + s[1:]  # Creates a new string
print(s)  # "Jello"

# String translation (maketrans/translate)
trans_table = str.maketrans({
    'a': '@',
    'e': '3',
    'i': '1',
    'o': '0',
    's': '$'
})
leet = "hello world".translate(trans_table)
print(f"Leet: {leet}")

# Using string methods for validation
def validate_email(email):
    """Simple email validation."""
    if "@" not in email:
        return False
    if email.count("@") != 1:
        return False
    local, domain = email.split("@")
    if not local or not domain:
        return False
    if "." not in domain:
        return False
    return True

emails = ["alice@example.com", "bob@.com", "charlie@site", "@domain.com"]
for email in emails:
    print(f"{email}: {'valid' if validate_email(email) else 'invalid'}")

# Regular expressions with strings
import re

text = "Contact us at support@example.com or sales@company.org"
emails = re.findall(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', text)
print(f"Found emails: {emails}")

# String buffer (efficient concatenation)
parts = ["Hello", "World", "Python", "Programming"]
# Inefficient:
result = ""
for part in parts:
    result += part + " "  # Creates many intermediate strings
print(f"Inefficient: {result.strip()}")

# Efficient:
result = " ".join(parts)
print(f"Efficient: {result}")

# Text wrapping
paragraph = "Python is an interpreted, high-level, general-purpose programming language."
wrapped = textwrap.fill(paragraph, width=40)
print(f"\nWrapped to 40 chars:\n{wrapped}")

# Template strings
from string import Template

template = Template("Hello, $name! You are $age years old.")
message = template.substitute(name="Alice", age=30)
print(f"\nTemplate: {message}")
```

## Real-World Use Cases

- **Data Cleaning**: Stripping whitespace, normalizing case, removing punctuation
- **Web Scraping**: Extracting text from HTML, parsing URLs
- **Natural Language Processing**: Tokenizing text, analyzing sentiment
- **File Processing**: Reading/writing CSV, JSON, and log files
- **User Interfaces**: Formatting output for command-line or GUI applications
- **Email Processing**: Parsing email addresses, formatting messages
- **Database Queries**: Building SQL queries, sanitizing inputs
- **Configuration Files**: Parsing INI, YAML, or configuration text

## Common Mistakes

```python
# Mistake 1: Trying to modify a string in place
s = "hello"
# s[0] = "H"  # TypeError

# Correct:
s = "H" + s[1:]

# Mistake 2: Using + for many string concatenations
items = ["apple", "banana", "cherry", "date", "elderberry"]
# Bad:
result = ""
for item in items:
    result += item + ", "  # O(n²) time complexity
# Good:
result = ", ".join(items)

# Mistake 3: Forgetting strings are immutable with methods
text = "HELLO"
text.lower()  # Returns new string, doesn't modify original
print(text)  # Still "HELLO"

# Correct:
text = text.lower()

# Mistake 4: Using == for string comparison with None
if text == None:  # Works but not idiomatic
    pass
# Better:
if text is None:
    pass

# Mistake 5: Confusing split() and split(' ')
"hello   world".split()    # ['hello', 'world'] (splits on any whitespace)
"hello   world".split(' ') # ['hello', '', '', 'world'] (splits on each space)

# Mistake 6: Not handling encoding
# Always specify encoding when reading/writing files
with open("file.txt", "r", encoding="utf-8") as f:
    content = f.read()
```

## Best Practices

- Use f-strings for string formatting (Python 3.6+)
- Use `join()` for concatenating many strings (efficiency)
- Use raw strings `r""` for regex patterns and file paths
- Use `str.strip()` to clean user input
- Always specify encoding when working with files (`utf-8`)
- Use `in` for substring checks: `if "sub" in text:`
- Use `textwrap.dedent()` for multi-line strings in code
- Prefer string methods over regex for simple operations
- Use `''.join()` for building strings from lists
- Use `str.partition()` or `str.split(sep, maxsplit=1)` for parsing
- Use `str.casefold()` for case-insensitive comparisons (locale-aware)

## Interview Questions

1. What are the different ways to format strings in Python?
2. Are strings mutable or immutable in Python? Explain with examples.
3. How does string slicing work? Give examples of [start:stop:step].
4. What is the difference between `find()` and `index()`?
5. How do you efficiently concatenate a large list of strings?
6. What are f-strings and what features do they support?
7. How do you check if a string contains a substring?
8. Explain the difference between `split()` and `split(' ')`.
9. What is `str.translate()` and when would you use it?
10. How do you reverse a string in Python?

## Coding Challenges

1. Write a function that checks if a string is a palindrome.
2. Create a function that counts the frequency of each character in a string.
3. Write a function that compresses a string (e.g., "aaabbbcc" -> "a3b3c2").
4. Create a function that checks if two strings are anagrams.
5. Write a function that extracts all email addresses from a text.

```python
# Challenge 1: Palindrome Check
def is_palindrome(s):
    s = ''.join(c.lower() for c in s if c.isalnum())
    return s == s[::-1]

tests = ["racecar", "A man, a plan, a canal: Panama", "hello"]
for t in tests:
    print(f"'{t}': {is_palindrome(t)}")

# Challenge 2: Character Frequency Counter
def char_frequency(text):
    freq = {}
    for char in text:
        if char.isalpha():
            freq[char.lower()] = freq.get(char.lower(), 0) + 1
    return dict(sorted(freq.items()))

print(char_frequency("Hello World"))

# Challenge 3: String Compression
def compress_string(s):
    if not s:
        return ""
    compressed = []
    count = 1
    for i in range(1, len(s)):
        if s[i] == s[i-1]:
            count += 1
        else:
            compressed.append(f"{s[i-1]}{count}")
            count = 1
    compressed.append(f"{s[-1]}{count}")
    result = ''.join(compressed)
    return result if len(result) < len(s) else s

print(compress_string("aaabbbcc"))      # a3b3c2
print(compress_string("abc"))            # abc (not shorter)

# Challenge 4: Anagram Checker
def are_anagrams(s1, s2):
    def clean(s):
        return ''.join(sorted(s.lower().replace(" ", "")))
    return clean(s1) == clean(s2)

print(are_anagrams("listen", "silent"))     # True
print(are_anagrams("hello", "world"))       # False
print(are_anagrams("Tom Marvolo Riddle", "I am Lord Voldemort"))  # True

# Challenge 5: Email Extractor
import re

def extract_emails(text):
    pattern = r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'
    return re.findall(pattern, text)

text = """
Contact us:
- Support: support@example.com
- Sales: sales@company.org
- Info: info@my-site.co.uk
Invalid: not-an-email, @missing, user@incomplete
"""
emails = extract_emails(text)
print(f"Found {len(emails)} emails:")
for email in emails:
    print(f"  - {email}")
```

## Summary

Strings in Python are immutable sequences of characters with rich built-in functionality. They support indexing, slicing, formatting (f-strings, format(), %-formatting), and a wide array of methods for searching, splitting, joining, transforming, and validating text. Understanding string operations is fundamental to text processing, data cleaning, and output formatting in Python applications.

## Related Topics

- String Formatting
- Regular Expressions (re module)
- Unicode and Encoding
- String Methods
- Text Processing
- Slicing
- F-Strings
- Input/Output
