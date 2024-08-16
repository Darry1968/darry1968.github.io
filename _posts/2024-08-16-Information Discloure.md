---
categories: [Portswigger, Information Disclousre]
tags: [web exploitation]     # TAG names should always be lowercase
author: Darshan
description: Information disclosure, also known as information leakage, is when a website unintentionally reveals sensitive information to its users.
image:
    path: /assets/img/Banners/Authentication-Vulnerabilities.jpeg
pin: False
---

- Data about other users, such as usernames or financial information
- Sensitive commercial or business data
- Technical details about the website and its infrastructure

Sensitive data can be leaked in all kinds of places, so it is important not to miss anything that could be useful later.

Even if the content of an error message doesn't disclose anything, sometimes the fact that one error case was encountered instead of another one is useful information in itself.

## Common sources of information disclosure:

- Files for web crawlers:
    
    sitemap.xml - An XML sitemap is a file that lists a website’s essential pages, making sure Google can find and crawl them all. It also helps search engines understand your website structure.
    
    robots.txt
    
- Directory listing:
    
    Directory traversing can be helpful to locate important files.
    
- Developer comments:
    
    They might hint at the existence of hidden directories or provide clues about the application logic.
    
- Error messages:
    
    You should pay close attention to all error messages you encounter during auditing.
    

### Lab1: **Information Disclosure in error message:**

play around with this website you’ll find one option `view Details` this will produce this URL-

https://0ae8005403600cff8544919300810071.web-security-academy.net/product?productId=1

just change the value of 1 with something else and you’ll see some error message like this:

[https://0ae8005403600cff8544919300810071.web-security-academy.net/product?productId="](https://0ae8005403600cff8544919300810071.web-security-academy.net/product?productId=%22)

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab1%20Information%20Disclosure%20in%20error%20message/Untitled.png)

Here is the version that is required to solve the lab.
<br>
<br>

- Debugging Data:
    
    While developing an application developer might use debug option to understand behaviour of an application.
    
    this will also allow attacker to understand behaviour if it was kept on. This include:
    
    - Values for key session variables that can be manipulated via user input
    - Hostnames and credentials for back-end components
    - File and directory names on the server
    - Keys used to encrypt data transmitted via the client
    
### Lab2: **Information disclosure on debug page**

View the source code first you’ll find one directory

`/cgi-bin/phpinfo.php`

If you visit this page you’ll find the value of the secret key - 

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab2%20Information%20disclosure%20on%20debug%20page/Untitled.png)
<br>
<br>
    
- Source code disclosure via backup files:
    
    Text editors often generate temporary backup files while the original file is being edited.
    
### Lab3: **Source code disclosure via backup files**

First i thought i should try to visiting the `/robots.txt` 

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab3%20Source%20code%20disclosure%20via%20backup%20files/Untitled.png)

Found this hidden directory by visiting that URL we found this:

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab3%20Source%20code%20disclosure%20via%20backup%20files/Untitled%201.png)

It contains java code and to solve this lab we have to find the password and submit that password,

here we found the password,

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab3%20Source%20code%20disclosure%20via%20backup%20files/Untitled%202.png)

submit this password and lab is solved.
<br>
<br>

- Information disclosure due to insecure configuration:
    
    This is common due to the widespread use of third-party technologies, whose vast array of configuration options are not necessarily well-understood by those implementing them.
    
    For example, HTTP `TRACE` method.
    
    It is designed for diagonstic purpose.
    
    - If enabled, the web server will respond to requests that use the `TRACE` method by echoing in the response the exact request that was received.
    
### Lab4: **Authentication bypass via Information disclosure**

As we have learned about the `TRACE` method so we knew that we have to use this HTTP method to solve the lab.

Lab’s description says, This lab's administration interface has an authentication bypass vulnerability.

So we have to focus on bypassing the logic page that’s for sure.

So we start our Burp and try the checking the request and their respective responses.

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab4%20Authentication%20bypass%20via%20Information%20disclos/Untitled.png)

Send this get request to the Burp Repeater. and change the HTTP `GET`method to `TRACE`method.

Which will result in this response.

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab4%20Authentication%20bypass%20via%20Information%20disclos/Untitled%201.png)

As we can see here their is one unique header used which is: 

```html
X-Custom-IP-Authorization: 152.57.193.50
```

Its not the actual header Portswigger created this for this lab. Setting its value to [localhost](http://localhost) what response we get we can check.

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab4%20Authentication%20bypass%20via%20Information%20disclos/Untitled%202.png)

we found there is one `/admin` page after we visit this page it says:

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab4%20Authentication%20bypass%20via%20Information%20disclos/Untitled%203.png)

so we have to send a POST request to admin page lets see what we get:

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab4%20Authentication%20bypass%20via%20Information%20disclos/Untitled%204.png)

Now we are admin and their is one delete button that will delete user from database and to solve this lab we have to delete `carlos`

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab4%20Authentication%20bypass%20via%20Information%20disclos/Untitled%205.png)

we got 302 that means user carlos is deleted.

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab4%20Authentication%20bypass%20via%20Information%20disclos/Untitled%206.png)
<br>
<br>

- Version control history:
    
    Virtually all websites are developed using some form of version control system, such as Git.
    
    You might be able to access it by simply browsing to `/.git`.
    
    This might not give you access to the full source code, but comparing the diff will allow you to read small snippets of code. As with any source code, you might also find sensitive data hard-coded within some of the changed lines.
    
### Lab5: **Information disclosure in version control history**

To solve the lab, obtain the password for the `administrator` user then log in and delete the user `carlos`.

As they previously explained about /.git directory so there might be some directory so lets try it out.

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab5%20Information%20disclosure%20in%20version%20control%20his/Untitled.png)

Here it is found YOU.

so now we have to download it using wget like this: (In Linux)

```html
wget -r https://0a1200a10314a6bf8120b697004100e6.web-security-academy.net/.git
```

Now before moving futher you must install git cola for your futher exploitation. Install that first.

After that open the download folder in git cola you’ll see two different files admin.conf and admin_panel.php

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab5%20Information%20disclosure%20in%20version%20control%20his/Untitled%201.png)

now Undo the last commit just like this:

![Untitled](/assets/img/Porswigger/InformationDisclosure/Lab5%20Information%20disclosure%20in%20version%20control%20his/Untitled%202.png)

In this you’ll find the last commit to admin.conf file which contains password use that and delete user carlos to solve the lab.