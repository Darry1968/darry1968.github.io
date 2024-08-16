---
categories: [Portswigger, Authentication Vulnerability]
tags: [web exploitation]     # TAG names should always be lowercase
author: Darshan
description: Authentication is the process of verifying that a user is who they claim to be. Authorization involves verifying whether a user is allowed to do something.
image:
    path: /assets/img/Banners/Authentication-Vulnerabilities.jpeg
pin: False
---

## Vulnerabilities in Password based login

- Brute-forcing usernames
    
    Usernames are especially easy to guess if they conform to a recognizable pattern, such as an email address. For example, it is very common to see business logins in the format `firstname.lastname@somecompany.com`. However, even if there is no obvious pattern, sometimes even high-privileged accounts are created using predictable usernames, such as `admin` or `administrator`.
    
    During auditing, check whether the website discloses potential usernames publicly. For example, are you able to access user profiles without logging in? Even if the actual content of the profiles is hidden, the name used in the profile is sometimes the same as the login username.
    
- Brute-forcing passwords
    
    While high-entropy passwords are difficult for computers alone to crack, we can use a basic knowledge of human behavior to exploit the vulnerabilities that users unwittingly introduce to this system. Rather than creating a strong password with a random combination of characters, users often take a password that they can remember and try to crowbar it into fitting the password policy. For example, if `mypassword` is not allowed, users may try something like `Mypassword1!` or `Myp4$$w0rd` instead
    

### Lab1: **Username enumeration via different responses**

Simple lab brute force wordlist given for username and password

for try bruteforce for username only using burp intuder

![Untitled](/assets/img/Porswigger/authVuln/Lab1%20Username%20enumeration%20via%20different%20responses/Untitled.png)
_Username bruteforce_
this will give you username now bruteforce password

![Untitled](/assets/img/Porswigger/authVuln/Lab1%20Username%20enumeration%20via%20different%20responses/Untitled%201.png)
_Password Bruteforce_
got the password. Login using these credentials and lab is solved.  
<br>

### Lab2: **Username enumeration via subtly different responses**

same method applied here to solve this lab as lab 1

Bruteforce username first using the given wordlist and and apply grep extract to see the response

![Untitled](/assets/img/Porswigger/authVuln/Lab2%20Username%20enumeration%20via%20subtly%20different%20res/Untitled.png)

as we can see at the end of response . is missing that means this is the correct username

use that username for bruteforcing password

![Untitled](/assets/img/Porswigger/authVuln/Lab2%20Username%20enumeration%20via%20subtly%20different%20res/Untitled%201.png)

and we got the hit and this is the password.
<br>

### Lab3: **Username enumeration via response timing**

```
To solve this lab they said they have added IP based protection so to bypass that use HTTP Headers

will use Header - `X-Forwarded-For`

using this header bruteforce username.

In Burp intruder add columns - Response Received and Response Completed

if correct username hits response time increases so a username with high response time is the correct username. use that username to bruteforce the password and make sure to use http header.

this way lab will be solved.
```

Logically, brute-force protection revolves around trying to make it as tricky as possible to automate the process and slow down the rate at which an attacker can attempt logins. The two most common ways of preventing brute-force attacks are:

- Locking the account that the remote user is trying to access if they make too many failed login attempts
- Blocking the remote user's IP address if they make too many login attempts in quick succession

Both approaches offer varying degrees of protection, but neither is invulnerable, especially if implemented using flawed logic.

For example, you might sometimes find that your IP is blocked if you fail to log in too many times. In some implementations, the counter for the number of failed attempts resets if the IP owner logs in successfully. This means an attacker would simply have to log in to their own account every few attempts to prevent this limit from ever being reached.

### Lab4: **Broken brute-force protection, IP block**

Logic fails when user enter correct username and password so he created a wordlist with combination of correct username and his username same for the password and then tried to bruteforce it and we got the hit

![Untitled](/assets/img/Porswigger/authVuln/Lab4%20Broken%20brute-force%20protection,%20IP%20block/Untitled.png)
<br>

### Lab5: **Username enumeration via account lock**

If we made some incorrect attepts then our account is blocked for a minutes.

Using this we can first brute force the username just to find out the correct username

![Untitled](/assets/img/Porswigger/authVuln/Lab5%20Username%20enumeration%20via%20account%20lock/Untitled.png)

to do that first add username payload and another one empty like above image for null payload this will help use to use cluster bomb in the burp intruder.

After starting an attack we can see one username has different response than other

![Untitled](/assets/img/Porswigger/authVuln/Lab5%20Username%20enumeration%20via%20account%20lock/Untitled%201.png)

that means our username is this.

now wait for a minute and replace username with correct username and add payload to password and start the attack which will provide you the correct password but it can be observed by the seeing response cuz if we made incorrect attempts then out account will be locked for a minute and correct password will not produce an error but will not be able to login for a minute using this we found the correct password.

![Untitled](/assets/img/Porswigger/authVuln/Lab5%20Username%20enumeration%20via%20account%20lock/Untitled%202.png)

Wait for a minute and login using these username and password lab is solved.
<br>

### Lab6: **Broken brute-force protection, multiple credentials per request**

This lab has rate limiting as a protection against bruteforcing to bypass this is sending multiple enteries at the same time just like this:

![Untitled](/assets/img/Porswigger/authVuln/lab6%20Broken%20brute-force%20protection,%20multiple%20crede/Untitled.png)

after sending this request we got 302

![Untitled](/assets/img/Porswigger/authVuln/lab6%20Broken%20brute-force%20protection,%20multiple%20crede/Untitled%201.png)

request this in browser and we are logged in as carlos.
<br>

## **Vulnerabilities in multi-factor authentication**

Some websites send verification codes to a user's mobile phone as a text message. While this is technically still verifying the factor of "something you have", it is open to abuse. Firstly, the code is being transmitted via SMS rather than being generated by the device itself. This creates the potential for the code to be intercepted. There is also a risk of SIM swapping, whereby an attacker fraudulently obtains a SIM card with the victim's phone number. The attacker would then receive all SMS messages sent to the victim, including the one containing their verification code.

### Lab1: **2FA simple bypass**

If the user is first prompted to enter a password, and then prompted to enter a verification code on a separate page, the user is effectively in a "logged in" state before they have entered the verification code.

It means that after login we are in logged in state so it doesn’t really matters we enter 2FA or not.

To solve this lab they have given the victim username and password

`carlos:montoya`

if we login using these credentials it ask for 4 digit security code 

![Untitled](/assets/img/Porswigger/authVuln/Lab1%202FA%20simple%20bypass/Untitled.png)

which has the url: [`https://0adf009c03b8d44380e5d05100860031.web-security-academy.net/login2`](https://0adf009c03b8d44380e5d05100860031.web-security-academy.net/login2)

just simply navigate to /my-account

![Untitled](/assets/img/Porswigger/authVuln/Lab1%202FA%20simple%20bypass/Untitled%201.png)

and we are logged in as carlos without 2FA code.
<br>

## Flawed two-factor verification logic:

Sometimes flawed logic in two-factor authentication means that after a user has completed the initial login step, the website doesn't adequately verify that the same user is completing the second step.

What it means a user is logging in with their credentials they are assigned a cookie value which is verified when 2FA is asked.

SO what attacker can do is generate a cookie their credentials and change it to any another user after which they can bruteforce the 2FA code for that user

### Lab2: **2FA broken logic**

This lab is bit complex so solve this lab step by step:
We have to compromise user carlos without password.

First check the normal behaviorof the website

- we login with `wiener:peter`  which then redirects to 2FA page 

- enter the correct pin from email client which they have given.

- while inspecting we can see there is one GET request sent to /login2 with 

- cookie: verify=wiener

- send this request to Repeater and change the value of verify=wiener to verify=carlos

- this means the 2FA code will be generated for carlos

now again login using wiener and submit a invalid 2FA code and send that request to intruder and bruteforce the 2FA code unitl you get the 302 

![Untitled](/assets/img/Porswigger/authVuln/Lab2%202FA%20broken%20logic/Untitled.png)

request that response in browser and will see that you are logged in as carlos.

![Untitled](/assets/img/Porswigger/authVuln/Lab2%202FA%20broken%20logic/Untitled%201.png)
<br>

### Lab3: **2FA bypass using a brute-force attack**

To solve this lab we need to create macro in burp

macro - *macro is used to automate process in burp just like fetching CSRF token etc.*  
Each time we send mfa-code they also need a csrf token with them to prevent bruteforcing.

Create a macro:

- Go to settings → Sessions → Session handling rules → add new rule

![Untitled](/assets/img/Porswigger/authVuln/Lab3%202FA%20bypass%20using%20a%20brute-force%20attack/Untitled.png)

- First add scope which includes all url

![Untitled](/assets/img/Porswigger/authVuln/Lab3%202FA%20bypass%20using%20a%20brute-force%20attack/Untitled%201.png)

after that add rule action → run a macro

![Untitled](/assets/img/Porswigger/authVuln/Lab3%202FA%20bypass%20using%20a%20brute-force%20attack/Untitled%202.png)

add new one and select the these 3 three URL:

![Untitled](/assets/img/Porswigger/authVuln/Lab3%202FA%20bypass%20using%20a%20brute-force%20attack/Untitled%203.png)

click ok and test macro. (click ok in every box opened) and then Goto history and send POST  /login2 to intruder.

add payload to mfa-code

select payload as brute forcer.

Resource pool → create a custom resource pool of maximum concurrent request 1

and start attack wait for responce 302 like this

![Untitled](/assets/img/Porswigger/authVuln/Lab3%202FA%20bypass%20using%20a%20brute-force%20attack/Untitled%204.png)

right click on that and click show response in browser and lab is solved.
<br>

## **Vulnerabilities in other authentication mechanisms**

Such as:

password reset

Session cookies

### Lab1: **Brute-forcing a stay-logged-in cookie**

- login as wiener:peter with stay-logged-in option

- You will be assigned a cookie which is something like this:

- d2llbmVyOjUxZGMzMGRkYzQ3M2Q0M2E2MDExZTllYmJhNmNhNzcw

- which looks like base64 - after decoding it we get:

- `wiener:51dc30ddc473d43a6011e9ebba6ca770`

- username:(md5 hash of password)

same will be for carlos but we don’t know the password for carlos for that they have given a wordlist which will be used to bruteforce the cookie for carlos

- Open Burp and perform the regular stuff → send /my-account request to burp intruder

- mark cookie as the payload position + remove the session cookie so that we are able to check whether cookie is valid or not.

- paste your wordlist for the payload 1 → Goto payload processing

1st add hash (md5)

2nd add prefix (carlos:)

3rd add Encoding (base64)

- In settings add grep match flag (Update email)- which will be used to identify the correct cookie cause Update email only appears when a user is authenticated.

 

![Untitled](/assets/img/Porswigger/authVuln/Lab1%20Brute-forcing%20a%20stay-logged-in%20cookie/Untitled.png)

and this token is correct cookie for the carlos and lab is solved automatically.
<br>

## Vulnerabilities in Resetting user passwords

A more robust method of resetting passwords is to send a unique URL to users that takes them to a password reset page. Less secure implementations of this method use a URL with an easily guessable parameter to identify which account is being reset, for example:

```
http://vulnerable-website.com/reset-password?user=victim-user
```

In this example, an attacker could change the `user` parameter to refer to any username they have identified. They would then be taken straight to a page where they can potentially set a new password for this arbitrary user.

A better implementation of this process is to generate a high-entropy, hard-to-guess token and create the reset URL based on that. In the best case scenario, this URL should provide no hints about which user's password is being reset.

```
http://vulnerable-website.com/reset-password?token=a0ba0d1cb3b63d13822572fcff1a241895d893f659164d4cc550b421ebdd48a8
```

When the user visits this URL, the system should check whether this token exists on the back-end and, if so, which user's password it is supposed to reset. This token should expire after a short period of time and be destroyed immediately after the password has been reset.

However, some websites fail to also validate the token again when the reset form is submitted. In this case, an attacker could simply visit the reset form from their own account, delete the token, and leverage this page to reset an arbitrary user's password.

### Lab2: **Password reset broken logic**

First understand the logic how actually password is being reset.

1. When you click Forgot password It ask for either username or email which is then used to send the password reset link.
2. The email client they have given which will receive the mail with a token which can be used for only once. When we click the password reset link which ask for new password and confirm password when we post that data, data includes
    - token
    - username
    - new password
    - confirm password
    
    ![Untitled](/assets/img/Porswigger/authVuln/Lab2%20Password%20reset%20broken%20logic/Untitled.png)
    

1. Now we need a new token to reset password so we make request from our account which will give us the new token which is not used yet.
2. Keeping intercept on will capture this request in burp and send that request to burp Repeater and drop that request

![Untitled](/assets/img/Porswigger/authVuln/Lab2%20Password%20reset%20broken%20logic/Untitled%201.png)

By changing the username from wiener to carlos we are able to reset the password for carlos and using that password we are able to login as carlos.
<br>

### Lab3: **Password reset poisoning via middleware**

Same thing we have to do here i.e steal the fucking token

First do the basic understanding of how application works.

This time we are not able to change the username while reseting password cause token is dynamically generated.

Now to steel the token they have given the exploit server on which we can receive the token which is clicked by carlos (for understanding)

In HTTP history see `POST /forgot-password` location which supports header `X-Forwarded-Host`

This header can be used to determine which website the client intended to access.

Use this header and Exploit server address as the value on which we’ll receive a request.

![Screenshot 2024-05-13 155102.png](/assets/img/Porswigger/authVuln/Lab3%20Password%20reset%20poisoning%20via%20middleware/Screenshot_2024-05-13_155102.png)

Send this request and then go to access log of the exploit server which contains a request of `forget-password` and this request contains a token for carlos user

![Untitled](/assets/img/Porswigger/authVuln/Lab3%20Password%20reset%20poisoning%20via%20middleware/Untitled.png)

generate a password reset link for wiener and change the token of wiener to carlos and carlos user password is changed.

Login using carlos and his new password and lab is solved.
<br>

### Lab4: **Password brute-force via password change**

This lab has flaw in password change functionality Let’s see what it is:

If we enter a correct current password and same new password in both field then password is changed:

![Untitled](/assets/img/Porswigger/authVuln/Lab4%20Password%20brute-force%20via%20password%20change/Untitled.png)

but if we enter a incorrect password and same password for both field then we are redirected to /login page

![Untitled](/assets/img/Porswigger/authVuln/Lab4%20Password%20brute-force%20via%20password%20change/Untitled%201.png)

If we enter correct password and both password different then this is what we get

![Untitled](/assets/img/Porswigger/authVuln/Lab4%20Password%20brute-force%20via%20password%20change/Untitled%202.png)

as expected and now if we enter a incorrect password and both password different then response is changed and this is what we get:

![Untitled](/assets/img/Porswigger/authVuln/Lab4%20Password%20brute-force%20via%20password%20change/Untitled%203.png)

Send this request to burp Intruder and we have given a wordlist using that wordlist bruteforce the carlos password by changing these field

![Untitled](/assets/img/Porswigger/authVuln/Lab4%20Password%20brute-force%20via%20password%20change/Untitled%204.png)

add payload to current password & change username to carlos and start attack and we get hit for 

New passwords do not match (This case rises when current password is correct but new passwords do not match)

![Untitled](/assets/img/Porswigger/authVuln/Lab4%20Password%20brute-force%20via%20password%20change/Untitled%205.png)

this way we found the correct password for Carlos user.
<br>