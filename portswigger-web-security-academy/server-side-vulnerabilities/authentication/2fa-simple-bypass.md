---
description: >-
  https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/authentication-apprentice/authentication/multi-factor/lab-2fa-simple-bypass
icon: vial-virus
---

# 2FA simple bypass

## Description

This lab's two-factor authentication can be bypassed. You have already obtained a valid username and password, but do not have access to the user's 2FA verification code. To solve the lab, access Carlos's account page.

* Your credentials: `wiener:peter`
* Victim's credentials `carlos:montoya`

## Solution

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Starting login as wiener user, after inserting username and password, the system asks us a 4-digit security code.

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

We can request the numeric secure code clicking on Email client button:

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

inserting it in the dedicated field, we log us correctly as 'wiener':

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption><p><a href="https://0a6b0064035e981b809621c7009f0018.web-security-academy.net/my-account?id=wiener">https://0a6b0064035e981b809621c7009f0018.web-security-academy.net/my-account?id=wiener</a></p></figcaption></figure>

Save the URL page: [https://0a6b0064035e981b809621c7009f0018.web-security-academy.net/my-account?id=wiener](https://0a6b0064035e981b809621c7009f0018.web-security-academy.net/my-account?id=wiener)

Checking request there's two login form:

'Login' page:

<figure><img src="../../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

and 'Login2' page, with GET method:

<figure><img src="../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

and with POST method:

<figure><img src="../../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

All clear, Logout from wiener's account and try to login into carlos's account (`carlos:montoya`):

<figure><img src="../../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

clicking on "Email client" button we can't see dedicated code message.

Remember the URL relative to 'wiener' account, we can try to change it manually and jump/bypass the 2FA check, solving the lab:

[https://0a6b0064035e981b809621c7009f0018.web-security-academy.net/my-account?id=carlos](https://0a6b0064035e981b809621c7009f0018.web-security-academy.net/my-account?id=carlos)

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>
