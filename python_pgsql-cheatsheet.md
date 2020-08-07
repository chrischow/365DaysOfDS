---
layout: page
title: Python x PostgreSQL Cheatsheet
background: '/img/posts/ml.jpg'
---

## Connecting with Environment Variables
```py
import os
import psycopg2 as pg2
from dotenv import load_dotenv

# This will look in root folder for .env file and load all variables
load_dotenv()

# Database setup
conn = pg2.connect(os.environ['DATABASE_URL'])
```

## Selecting Queries with Variables
Use `%s` as the string variable:

```py
DELETE_MOVIE = "DELETE FROM movies WHERE title = %s;"
```

## Using the Cursor
Using `with conn` and `with conn.cursor()` ensures any resources associated with the cursor are released at the end of the `with` context:

```py
def pg2_function():
    with conn:
        with conn.cursor() as cursor:
            conn.execute(QUERY)
```

## Executing Same Query with Different Inputs
Use the `execute_values` command to execute a query on each item in a list of tuples.

```py
from psycopg2.extras import execute_values
execute_values(cursor, QUERY, list_of_tuples)
```

## Connection Pooling
```py
# Set up pool
pool = SimpleConnectionPool(minconn=1, maxconn=10, dsn=database_url)

# Get a connection
conn = pool.getconn()

# ---- DATABASE OPERATIONS HERE ---- #

# Put the connection back
conn.putconn(conn)
```

## Context Managers
Using the pool from above:

```py
from contextlib import contextmanager

@contextmanager
def get_connection():
    connection = pool.getconn()

    try:
        yield connection
    finally:
        pool.putconn(connection)

@contextmanager
def get_cursor(conn):
    with conn:
        with conn.cursor() as cursor:
            yield cursor

# Usage
with get_connection() as conn:
    pass

with get_cursor() as cursor:
    pass
```

## Structuring Code

1. Prepare (pooled) connection and context manager for cursor + (pooled) connection
2. Queries in string form
3. Functions for executing transactions