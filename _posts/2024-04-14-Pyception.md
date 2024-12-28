---
categories: [ShunyaCTF]
tags: [web exploitation]     # TAG names should always be lowercase
author: Darshan
description: Web exploitation category challenge created by me for ShunyaCTF Aarambha
image:
    path: /assets/img/Banners/pyception.jpg
pin: False
---

# Solution for Pyception Challenge

## Description
My friend loved making lame ahh dad jokes, so I made a website for him where I rated how lame his dad jokes quotes were. Try it out!

## Vulnerability Explained
The challenge exploits a string concatenation vulnerability. By crafting a payload that uses escape sequences and string manipulation, it is possible to bypass the input restrictions and access sensitive files on the server. Specifically, the issue arises from improper validation and sanitization of user input, allowing the attacker to inject malicious commands. Double quotes within double quotes are not allowed when you provide input so instead of providing that we just need to escape double quotes. 

## Application Walkthrough

Upon visiting website you will see this page:  
![homepage](/assets/img/ShunyaCTF/Pyception/homepage.png)

1. Input should be in double quotes.
2. Double quotes within double quotes is not allowed.

That means the key to solve this challenge is break this second condition.  

## Solution Walkthrough
To solve this challenge, the key was understanding how to exploit the string concatenation functionality by providing a payload in double quotes. The payload used the escape sequence `\x22` to represent double quotes, which enabled breaking out of the intended string and injecting a malicious command to read the flag file.

The payload:
```python
"\x22+open('static/flag.txt').read()+\x22"
```
Explanation of the payload:
- `\x22`: Represents a double quote (") in hexadecimal.
- `open('static/flag.txt').read()`: Reads the content of the `flag.txt` file located in the `static` directory.
- The payload concatenates this injected command into the input string, effectively bypassing restrictions and retrieving the flag.

## How to Use the payload
1. Navigate to the application where the challenge is hosted.
2. In the input field that requires a string, insert the payload:
   ```
   "\x22+open('static/flag.txt').read()+\x22"
   ```
3. Submit the input.
4. The flag should be displayed if the payload executes successfully.

## Conclusion
This challenge demonstrates the importance of validating and sanitizing user input to prevent injection attacks. The ability to manipulate strings through escape sequences and concatenation highlights the need for secure programming practices to safeguard sensitive data and application functionality.

