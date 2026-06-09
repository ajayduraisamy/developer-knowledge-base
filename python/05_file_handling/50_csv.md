# CSV - csv.reader, csv.writer, DictReader, DictWriter

## Introduction

CSV (Comma-Separated Values) is a simple file format for storing tabular data. Each line represents a row, and fields within a row are separated by a delimiter (typically a comma). Python's `csv` module provides classes for reading and writing CSV files, handling various dialects and formatting options. Despite its simplicity, CSV has many variations—different delimiters, quoting rules, line endings—which the `csv` module handles through its dialect system.

## csv.reader

### What It Is

`csv.reader` creates an iterable object that reads CSV rows as lists of strings. It accepts any iterable of strings (typically a file object) and handles parsing complexities like quoted fields, escaped delimiters, and embedded newlines.

### Why It Is Important

`csv.reader` provides a robust way to parse CSV data without writing fragile string-splitting code. It handles edge cases like fields containing commas, newlines, and quotes that would break naive `line.split(",")` approaches.

### How It Works Internally

The reader implements a state machine that processes characters one at a time. It tracks whether it's inside a quoted field, handles escape sequences, and properly manages field boundaries. It yields rows as it completes them, enabling streaming of large files without loading everything into memory.

### Syntax

```python
import csv

with open("file.csv", "r", newline="") as f:
    reader = csv.reader(
        f,
        dialect="excel",      # Default dialect
        delimiter=",",         # Field separator
        quotechar='"',         # Quote character
        escapechar=None,       # Escape character
        doublequote=True,      # "" escapes " inside quoted field
        skipinitialspace=False, # Skip spaces after delimiter
        quoting=csv.QUOTE_MINIMAL,  # Quoting behavior
    )
    for row in reader:
        print(row)  # Each row is a list of strings
```

### Beginner Examples

```python
import csv

# Basic reading
with open("people.csv", "r", newline="") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)  # ['Name', 'Age', 'City'], ['Alice', '30', 'NY'], ...

# Skipping header
with open("data.csv", "r", newline="") as f:
    reader = csv.reader(f)
    header = next(reader)  # Read header separately
    for row in reader:
        print(f"Processing: {row}")

# Tab-delimited files (TSV)
with open("data.tsv", "r", newline="") as f:
    reader = csv.reader(f, delimiter="\t")
    for row in reader:
        print(row)

# Custom delimiter
with open("data.psv", "r", newline="") as f:
    reader = csv.reader(f, delimiter="|")
    for row in reader:
        print(row)

# Handling quoted fields
with open("data.csv", "r", newline="") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)  # Fields with commas are properly handled

# Converting types
with open("numbers.csv", "r", newline="") as f:
    reader = csv.reader(f)
    next(reader)  # Skip header
    for row in reader:
        name = row[0]
        value = int(row[1])  # Convert to int
        print(f"{name}: {value}")

# Reading only specific columns
with open("data.csv", "r", newline="") as f:
    reader = csv.reader(f)
    next(reader)
    names = [row[0] for row in reader]
```

## csv.writer

### What It Is

`csv.writer` writes rows to a CSV file. It takes a file-like object and optional formatting parameters. It handles proper quoting, escaping, and line termination automatically.

### Why It Is Important

`csv.writer` produces correctly formatted CSV output, handling field quoting, embedded delimiters, and newlines that would otherwise corrupt the output. It ensures the output can be read back correctly by any CSV parser.

### How It Works Internally

The writer examines each field and decides whether quoting is needed based on the `quoting` parameter. `QUOTE_MINIMAL` quotes only fields containing special characters. `QUOTE_ALL` quotes every field. `QUOTE_NONNUMERIC` quotes all non-numeric fields. `QUOTE_NONE` never quotes (requires escapechar). Fields containing the delimiter, quotechar, or newlines are properly escaped or quoted.

### Syntax

```python
import csv

with open("output.csv", "w", newline="") as f:
    writer = csv.writer(
        f,
        dialect="excel",
        delimiter=",",
        quotechar='"',
        escapechar=None,
        doublequote=True,
        skipinitialspace=False,
        quoting=csv.QUOTE_MINIMAL,  # QUOTE_ALL, QUOTE_MINIMAL, QUOTE_NONNUMERIC, QUOTE_NONE
    )
    writer.writerow(["Name", "Age", "City"])     # Single row
    writer.writerows([                           # Multiple rows
        ["Alice", 30, "NY"],
        ["Bob", 25, "LA"],
    ])
```

### Beginner Examples

```python
import csv

# Basic writing
with open("output.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["Name", "Age"])
    writer.writerow(["Alice", 30])
    writer.writerow(["Bob", 25])

# Writing multiple rows at once
data = [
    ["Product", "Price", "Quantity"],
    ["Widget", 19.99, 100],
    ["Gadget", 29.99, 50],
]
with open("products.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(data)

# Tab-delimited output
with open("output.tsv", "w", newline="") as f:
    writer = csv.writer(f, delimiter="\t")
    writer.writerow(["Name", "Score"])
    writer.writerow(["Alice", 95])

# Quoting all fields
with open("output.csv", "w", newline="") as f:
    writer = csv.writer(f, quoting=csv.QUOTE_ALL)
    writer.writerow(["Name", "Description"])
    writer.writerow(["Alice", "Engineer, Python expert"])

# Appending to existing CSV
with open("log.csv", "a", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["2024-01-15", "INFO", "Started"])

# Writing with different quoting
import csv

# QUOTE_MINIMAL: only quote when necessary
writer = csv.writer(f, quoting=csv.QUOTE_MINIMAL)
writer.writerow(["Hello", "Has, comma", "Has\nnewline"])

# QUOTE_NONNUMERIC: quote all text fields
writer = csv.writer(f, quoting=csv.QUOTE_NONNUMERIC)
writer.writerow(["Alice", 30])  # Alice is quoted, 30 is not
```

## DictReader

### What It Is

`csv.DictReader` reads CSV rows as dictionaries, using the first row (or explicitly provided fieldnames) as keys. Each row is an `OrderedDict` mapping column names to field values.

### Why It Is Important

`DictReader` makes CSV code more readable and maintainable by accessing columns by name instead of numeric index. It also handles column reordering gracefully—the code still works if columns are rearranged in the source file.

### How It Works Internally

`DictReader` wraps a `csv.reader`. On initialization, it reads the first row as fieldnames (unless provided explicitly). For each subsequent row, it creates an `OrderedDict` mapping fieldnames to field values. If a row has more fields than fieldnames, the extra fields are assigned to `None` keys.

### Syntax

```python
import csv

# Using first row as fieldnames
with open("data.csv", "r", newline="") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row["Name"], row["Age"])

# Explicit fieldnames
with open("data.csv", "r", newline="") as f:
    reader = csv.DictReader(f, fieldnames=["Name", "Age", "City"])
    for row in reader:
        print(row["Name"])

# Skip header and use custom fieldnames
with open("data.csv", "r", newline="") as f:
    reader = csv.DictReader(f, fieldnames=["a", "b", "c"])
    next(reader)  # Skip the real header
    for row in reader:
        print(row["a"], row["b"])
```

### Beginner Examples

```python
import csv

# Basic DictReader
with open("employees.csv", "r", newline="") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(f"{row['Name']} works in {row['Department']}")

# Accessing specific fields
with open("data.csv", "r", newline="") as f:
    reader = csv.DictReader(f)
    names = [row["Name"] for row in reader]

# Handling missing columns
with open("data.csv", "r", newline="") as f:
    reader = csv.DictReader(f)
    for row in reader:
        email = row.get("Email", "N/A")  # Safe access
        print(email)

# Converting types
with open("data.csv", "r", newline="") as f:
    reader = csv.DictReader(f)
    for row in reader:
        age = int(row["Age"])  # Convert string to int
        print(f"{row['Name']} is {age} years old")
```

## DictWriter

### What It Is

`csv.DictWriter` writes dictionaries to CSV files. It requires a list of fieldnames that defines the column order and writes a header row when `writeheader()` is called.

### Why It Is Important

`DictWriter` allows writing CSV data using dictionaries keyed by column name, making the code self-documenting. It automatically handles column ordering and ensures all specified fields are written in the correct positions.

### How It Works Internally

`DictWriter` wraps a `csv.writer`. The `writerow()` method extracts values from the dict in the order specified by `fieldnames`. Missing keys produce empty fields. Extra keys in the dict that aren't in `fieldnames` are ignored unless `extrasaction='raise'` is set.

### Syntax

```python
import csv

with open("output.csv", "w", newline="") as f:
    fieldnames = ["Name", "Age", "City"]
    writer = csv.DictWriter(
        f,
        fieldnames=fieldnames,
        extrasaction="raise",  # or "ignore" (default)
        restval="",            # Default for missing fields
    )
    writer.writeheader()  # Writes header row
    writer.writerow({"Name": "Alice", "Age": 30, "City": "NY"})
    writer.writerows([
        {"Name": "Bob", "Age": 25, "City": "LA"},
        {"Name": "Charlie", "Age": 35, "City": "Chicago"},
    ])
```

### Beginner Examples

```python
import csv

# Basic DictWriter
fieldnames = ["Name", "Email", "Department"]
employees = [
    {"Name": "Alice", "Email": "alice@example.com", "Department": "Engineering"},
    {"Name": "Bob", "Email": "bob@example.com", "Department": "Marketing"},
]

with open("employees.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(employees)

# Handling missing fields
fieldnames = ["Name", "Age", "Email"]
data = [
    {"Name": "Alice", "Age": 30, "Email": "alice@example.com"},
    {"Name": "Bob", "Age": 25},  # Missing Email
]

with open("output.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=fieldnames, restval="N/A")
    writer.writeheader()
    writer.writerows(data)

# Ignoring extra fields
fieldnames = ["Name", "Age"]
data = [
    {"Name": "Alice", "Age": 30, "Extra": "ignored"},
]
with open("output.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=fieldnames, extrasaction="ignore")
    writer.writeheader()
    writer.writerows(data)

# Selecting subset of columns
all_data = [
    {"Name": "Alice", "Age": 30, "City": "NY", "Phone": "555-0100"},
    {"Name": "Bob", "Age": 25, "City": "LA", "Phone": "555-0200"},
]
fieldnames = ["Name", "City"]  # Only export these columns
with open("subset.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(all_data)
```

### Intermediate Examples

```python
import csv
from collections import defaultdict

# Processing CSV with DictReader/DictWriter
def filter_and_transform(input_file, output_file, column, min_val):
    with open(input_file, "r", newline="") as f:
        reader = csv.DictReader(f)
        rows = [row for row in reader if float(row[column]) >= min_val]

    with open(output_file, "w", newline="") as f:
        if rows:
            writer = csv.DictWriter(f, fieldnames=reader.fieldnames)
            writer.writeheader()
            writer.writerows(rows)

# Merging multiple CSV files
def merge_csv_files(input_files, output_file):
    all_rows = []
    fieldnames = None

    for filename in input_files:
        with open(filename, "r", newline="") as f:
            reader = csv.DictReader(f)
            if fieldnames is None:
                fieldnames = reader.fieldnames
            for row in reader:
                all_rows.append(row)

    with open(output_file, "w", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(all_rows)

# Aggregation
def aggregate_by(input_file, group_col, value_col):
    groups = defaultdict(list)
    with open(input_file, "r", newline="") as f:
        reader = csv.DictReader(f)
        for row in reader:
            groups[row[group_col]].append(float(row[value_col]))

    results = {}
    for group, values in groups.items():
        results[group] = {
            "sum": sum(values),
            "avg": sum(values) / len(values),
            "count": len(values),
            "min": min(values),
            "max": max(values),
        }
    return results
```

### Advanced Examples

```python
import csv
import io

# Custom dialect
class PipeDialect(csv.Dialect):
    delimiter = "|"
    quotechar = '"'
    doublequote = True
    skipinitialspace = True
    lineterminator = "\n"
    quoting = csv.QUOTE_ALL

csv.register_dialect("piped", PipeDialect)

with open("data.pipe", "w", newline="") as f:
    writer = csv.writer(f, dialect="piped")
    writer.writerow(["Name", "Description", "Price"])
    writer.writerow(["Widget", "A | useful | tool", "19.99"])

with open("data.pipe", "r", newline="") as f:
    reader = csv.reader(f, dialect="piped")
    for row in reader:
        print(row)

# Sniffing dialect from unknown source
with open("unknown.csv", "r", newline="") as f:
    sample = f.read(1024)
    dialect = csv.Sniffer().sniff(sample)
    f.seek(0)
    reader = csv.reader(f, dialect=dialect)
    for row in reader:
        print(row)

# CSV in memory
csv_data = io.StringIO()
writer = csv.writer(csv_data)
writer.writerow(["Name", "Value"])
writer.writerow(["Alice", 42])
csv_string = csv_data.getvalue()

reader = csv.reader(io.StringIO(csv_string))
for row in reader:
    print(row)

# Streaming large CSV processing
def process_large_csv(filename, chunk_size=1000):
    with open(filename, "r", newline="") as f:
        reader = csv.reader(f)
        header = next(reader)
        chunk = []
        for row in reader:
            chunk.append(row)
            if len(chunk) >= chunk_size:
                process_chunk(chunk, header)
                chunk = []
        if chunk:
            process_chunk(chunk, header)
```

### Real-World Use Cases

```python
# Data validation and cleaning
import re

def validate_and_clean(input_file, output_file):
    valid_rows = []
    with open(input_file, "r", newline="") as f:
        reader = csv.DictReader(f)
        for row_num, row in enumerate(reader, 1):
            errors = []
            if not row.get("Email") or "@" not in row["Email"]:
                errors.append(f"Row {row_num}: Invalid email")
            if row.get("Age") and not re.match(r"^\d+$", row["Age"]):
                errors.append(f"Row {row_num}: Invalid age")
            if errors:
                for error in errors:
                    print(error)
            else:
                valid_rows.append(row)

    with open(output_file, "w", newline="") as f:
        if valid_rows:
            writer = csv.DictWriter(f, fieldnames=reader.fieldnames)
            writer.writeheader()
            writer.writerows(valid_rows)

# ETL pipeline
def etl_pipeline(source_file, dest_file, transformations):
    with open(source_file, "r", newline="") as f:
        rows = list(csv.DictReader(f))

    for transform in transformations:
        rows = [transform(row) for row in rows if row]

    with open(dest_file, "w", newline="") as f:
        if rows:
            writer = csv.DictWriter(f, fieldnames=rows[0].keys())
            writer.writeheader()
            writer.writerows(rows)
```

### Common Mistakes

```python
# Mistake 1: Forgetting newline=''
with open("data.csv", "r") as f:  # Wrong: no newline=''
    reader = csv.reader(f)

# Correct:
with open("data.csv", "r", newline="") as f:
    reader = csv.reader(f)

# Mistake 2: Using split() instead of csv.reader
for line in open("data.csv"):
    fields = line.split(",")  # Fails on quoted commas

# Mistake 3: Assuming delimiter is always comma
with open("data.tsv", "r", newline="") as f:
    reader = csv.reader(f)  # Wrong: should specify delimiter="\t"

# Mistake 4: Not skipping header when needed
with open("data.csv", "r", newline="") as f:
    reader = csv.reader(f)
    for row in reader:  # First iteration is the header!
        print(row)

# Mistake 5: Ignoring the quoting parameter
# If a field contains a comma, it needs to be quoted

# Mistake 6: Writing lists to DictWriter
fieldnames = ["Name", "Age"]
with open("output.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writerow(["Alice", 30])  # Wrong: must be dict!
    writer.writerow({"Name": "Bob", "Age": 25})  # Correct
```

### Best Practices

- Always use `newline=""` when opening CSV files
- Specify encoding explicitly: `encoding="utf-8-sig"` for BOM files
- Use `DictReader`/`DictWriter` for readability and maintainability
- Validate CSV data after reading from external sources
- Process large files by iterating, not loading everything into memory
- Use `csv.Sniffer` to auto-detect dialect from unknown sources
- Write to temp file first, then rename for atomic writes
- Handle quoted fields properly—don't use manual string splitting
- Use `quoting=csv.QUOTE_NONNUMERIC` for type-preserving output

### Performance Considerations

- `csv.reader` is very efficient for sequential access
- Iterating over the reader object streams data without loading all into memory
- `DictReader` adds small overhead per row (dict creation) vs `reader` (list)
- For maximum throughput, use `csv.reader` and access by index
- Writing with `writerows()` is faster than multiple `writerow()` calls
- CSV I/O is typically I/O-bound, not CPU-bound

### Interview Questions

1. What is the purpose of `newline=""` when opening CSV files?

   It disables universal newline translation, allowing the CSV reader to handle quoted fields containing embedded newlines correctly.

2. Difference between `csv.reader` and `csv.DictReader`?

   `reader` returns rows as lists accessed by index. `DictReader` returns rows as dicts accessed by column name (using header row as keys).

3. What are CSV dialects?

   Dialects define formatting parameters (delimiter, quotechar, quoting rules). Predefined dialects include 'excel', 'excel-tab', 'unix'.

4. How does `csv.Sniffer` work?

   It analyzes a CSV sample using heuristics (delimiter frequency, quote patterns, header detection) to determine the dialect.

5. What quoting options does csv.writer support?

   `QUOTE_MINIMAL` (default), `QUOTE_ALL`, `QUOTE_NONNUMERIC`, `QUOTE_NONE`. They control which fields are wrapped in quotes.

### Coding Challenges

1. CSV to JSON converter that reads CSV and outputs JSON with appropriate data types.

2. CSV column operations: add, remove, rename, reorder columns streaming.

3. CSV diff tool that compares two CSV files and reports row-level differences.

4. CSV pivot table generator that produces grouped/aggregated output.

5. CSV validation suite that checks data types, required fields, value ranges.

### Related Topics

- `48_file_operations.md` - File I/O basics for CSV
- `49_json.md` - JSON data interchange
- `52_pathlib.md` - Path management
- `53_temp_files.md` - Temporary files for CSV processing
- pandas - Advanced CSV reading/writing
- Data cleaning and validation techniques
