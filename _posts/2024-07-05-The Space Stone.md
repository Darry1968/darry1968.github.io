---
categories: [ShunyaCTF]
tags: [web exploitation]     # TAG names should always be lowercase
author: Darshan
description: Web exploitation category challenge created by me for ShunyaCTF 2k24
image:
    path: /assets/img/Banners/the-space-stone.jpg
pin: False
---

# Solution for Pickle RCE Challenge

## Challenge Overview

In this challenge, you are tasked with exploiting a Python web application that uses the `pickle` module to deserialize user-provided data. The application is vulnerable to a Remote Code Execution (RCE) attack due to unsafe deserialization practices. The objective is to craft a malicious payload that, when deserialized, executes a command to reveal the contents of a file named `flag.txt`.

## Vulnerability Explanation

Python's `pickle` module is used to serialize and deserialize Python objects. However, it is inherently insecure when handling untrusted data. The `pickle` module allows arbitrary code execution during deserialization if a malicious payload is provided. This vulnerability arises because `pickle` can execute methods like `__reduce__` or `__reduce_ex__` defined in objects, enabling attackers to run arbitrary commands on the server.

### Key Points:
1. **Deserialization**: The application deserializes user input without validation.
2. **Arbitrary Code Execution**: By defining a custom class with a `__reduce__` method, attackers can control the behavior during deserialization.
3. **Target File**: The payload is crafted to read the contents of `flag.txt`.

## Application Walkthrough

The following screenshots showcase the vulnerable application in action:
 
![HomePage](/assets/img/ShunyaCTF/SpaceStone/homepage.png)

1. This the home page provided explaining about what a space stone is and what are the key facts about the space stone.
2. After reviewing the source code, we understand that the Tresseract seems to be clickable and after clicking the button we are redirected to another location named `/space`.
3. This is the main application page as shown below.  

![Untitled](/assets/img/ShunyaCTF/SpaceStone/mainpage.png)

1. This main page contains information about the Marvel characters and there is one input box also to search for any specific character like `Thor`.
2. And below that we have a comment section where people can comment about Marvel characters.
3. This comment section is vulnerable to insecure deserialization vulnerability and users have to exploit this and gain RCE to read the `flag.txt`.
4. Screenshot of comment section is as shown below.  

![Untitled](/assets/img/ShunyaCTF/SpaceStone/commentsection.png)

## Solution Script Walkthrough

The solution script creates two payloads—one for a Windows server and another for a Linux server. Each payload exploits the deserialization vulnerability to execute commands that display the contents of `flag.txt`.

### Windows Payload
```python
import pickle
import base64
import os

class RCE:
    def __reduce__(self):
        return (os.system, ('more flag.txt',))

malicious_payload = base64.b64encode(pickle.dumps(RCE())).decode()
print("Payload for Windows server: ", malicious_payload)
```

#### Explanation:
1. **Custom Class**: The `RCE` class defines the `__reduce__` method, which specifies the function (`os.system`) and its argument (`more flag.txt`).
2. **Serialization**: The `pickle.dumps` method serializes the `RCE` object.
3. **Encoding**: The serialized payload is Base64 encoded to ensure safe transmission over the network.
4. **Command Execution**: When deserialized, the payload executes the `more` command to display the contents of `flag.txt`.

### Linux Payload
```python
import pickle
import base64
import os
import subprocess

class RCE:
    def __reduce__(self):
        cmd = ('cat', 'flag.txt')
        return (subprocess.check_output, (cmd,))

malicious_payload = base64.b64encode(pickle.dumps(RCE())).decode()
print("Payload for linux server: ", malicious_payload)
```

#### Explanation:
1. **Custom Class**: The `RCE` class defines the `__reduce__` method, which specifies the function (`subprocess.check_output`) and its argument (`('cat', 'flag.txt')`).
2. **Serialization**: The `pickle.dumps` method serializes the `RCE` object.
3. **Encoding**: The serialized payload is Base64 encoded.
4. **Command Execution**: When deserialized, the payload executes the `cat` command to display the contents of `flag.txt`.

## How to Use the Solution Script

1. **Run the Script**:
   - Save the script to a file (e.g., `generate_payload.py`).
   - Execute it in the environment corresponding to the target server (Windows or Linux).
   - Example:
     ```bash
     python generate_payload.py
     ```

2. **Copy the Payload**:
   - The script outputs the payload in Base64 format.
   - Copy the payload to use in the exploit.

3. **Exploit the Vulnerability**:
   - Send the payload to the vulnerable application.
   - The application will deserialize the payload, execute the embedded command, and reveal the contents of `flag.txt`.

## Working of the Exploit

1. **Payload Creation**:
   - The script defines a malicious object that executes commands during deserialization.
   - It serializes and Base64-encodes the object for safe delivery.

2. **Payload Delivery**:
   - The attacker sends the payload to the vulnerable application (e.g., through an API endpoint).

3. **Command Execution**:
   - The application deserializes the payload, invoking the `__reduce__` method of the malicious object.
   - The specified command is executed on the server.

4. **Flag Retrieval**:
   - The command reads the contents of `flag.txt` and outputs them to the attacker.

## Conclusion

This challenge highlights the dangers of insecure deserialization and the importance of following secure coding practices. By crafting and analyzing the solution script, participants gain hands-on experience with a critical vulnerability and learn how to mitigate it in real-world applications.

