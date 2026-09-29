# Lab 2 - Add Exception Handling to the Tiki PostgreSQL Loader

## DoD

Extend Lab 1 by adding exception handling when:

- reading product data from files
- working with PostgreSQL

Lab 2 keeps the same data flow and database logic from Lab 1, but makes the program safer when file or database errors occur.

Reference for Python file handling:

https://www.w3schools.com/python/python_file_handling.asp

---

## Folder Structure

`project2` and `lab2` are sibling folders:

```text
Desktop/
├── project2/
│   └── tiki_crawl/
│       └── output/
│           ├── products_001.json
│           ├── products_002.json
│           └── ...
│
└── lab2/
    ├── config.py
    ├── database.ini.example
    ├── create_tables.py
    ├── load_products.py
    ├── delete_products.py
    ├── verify.py
    ├── requirements.txt
    ├── .gitignore
    └── README.md
```

The loader reads source data from:

```text
../project2/tiki_crawl/output/
```

---

## Data Flow

```text
products_*.json
       ↓
try to open/read file
     │
     ├── file error → handle exception
     │
       ↓
parse JSON
     │
     ├── invalid JSON → handle exception
     │
       ↓
prepare product data
       ↓
connect to PostgreSQL
     │
     ├── connection/database error → handle exception
     │
       ↓
SELECT product by id
     │
  ┌──┴──┐
   ↓      ↓
UPDATE  INSERT
      ↓
   commit
```

---

## PostgreSQL Table

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY,
    name TEXT,
    url_key TEXT,
    price BIGINT,
    description TEXT,
    images_url TEXT[]
);
```

---

## Exception Handling

### File Exceptions

When reading JSON files, the loader handles common file-related errors.

Examples:

```python
try:
    with open(file_path, "r", encoding="utf-8") as file:
        data = json.load(file)

except FileNotFoundError:
    print(f"File not found: {file_path}")

except PermissionError:
    print(f"Permission denied: {file_path}")

except json.JSONDecodeError:
    print(f"Invalid JSON: {file_path}")

except OSError as error:
    print(f"File error: {error}")
```

These exceptions cover cases such as:

- the file does not exist
- Python does not have permission to read the file
- the file is not valid JSON
- another file I/O error occurs

---

### Database Exceptions

Database operations are also wrapped in `try` / `except`.

Example:

```python
try:
    with psycopg2.connect(**config) as conn:
        with conn.cursor() as cur:
            # database operations

except psycopg2.OperationalError as error:
    print(f"Database connection error: {error}")

except psycopg2.DatabaseError as error:
    print(f"Database error: {error}")

except Exception as error:
    print(f"Unexpected error: {error}")
```

This allows the program to report database problems more clearly instead of stopping with an unhandled exception.

---

## Setup

### 1. Open Lab 2

```bash
cd ~/Desktop/lab2
```

### 2. Create and activate the virtual environment

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Create the PostgreSQL database

```bash
sudo -u postgres psql
```

Inside PostgreSQL:

```sql
CREATE DATABASE lab2;
```

Exit:

```text
\\q
```

### 5. Create the local database config

```bash
cp database.ini.example database.ini
nano database.ini
```

Example:

```ini
[postgresql]
host=localhost
database=lab2
user=postgres
password=YOUR_PASSWORD
port=5432
```

---

## Create the Table

```bash
python3 create_tables.py
```

Expected output:

```text
Created products table.
```

If a database error occurs, the program catches and prints the exception.

---

## Load Product Data

```bash
python3 load_products.py
```

The loader:

1. Finds every `products_*.json` file.
2. Opens each file inside a `try` block.
3. Handles file and JSON exceptions.
4. Prepares product records.
5. Connects to PostgreSQL.
6. Checks whether each product already exists.
7. Uses `INSERT` for new products.
8. Uses `UPDATE` for existing products.
9. Commits progress in batches.
10. Handles PostgreSQL exceptions.

Example progress:

```text
Processed 1000/124299 | inserted=1000 | updated=0
Processed 2000/124299 | inserted=2000 | updated=0
...
```

Running the loader again updates existing products instead of creating duplicate rows.

---

## Verify the Load

```bash
python3 verify.py
```

Expected row count for the current Project 02 output:

```text
Products in PostgreSQL: 124299
```

Database errors are handled with `try` / `except`.

---

## Delete All Product Data

```bash
python3 delete_products.py
```

The script keeps the `products` table but removes all rows:

```sql
DELETE FROM products;
```

Example:

```text
Deleted 124299 products.
```

---

## Python Files

### `config.py`

Loads PostgreSQL connection settings from `database.ini`.

### `create_tables.py`

Creates the PostgreSQL table and handles database exceptions.

### `load_products.py`

Reads JSON files, handles file exceptions, and loads products into PostgreSQL using:

```text
SELECT -> INSERT or UPDATE
```

### `verify.py`

Queries the table and handles database exceptions.

### `delete_products.py`

Deletes all product rows and handles database exceptions.

---

## Concepts Practiced

Lab 2 builds on Lab 1 and adds:

- Python file handling with `open()`
- `with open(...)`
- `try` / `except`
- `FileNotFoundError`
- `PermissionError`
- `OSError`
- `json.JSONDecodeError`
- PostgreSQL exception handling with `psycopg2`
- `OperationalError`
- `DatabaseError`
- handling unexpected exceptions

