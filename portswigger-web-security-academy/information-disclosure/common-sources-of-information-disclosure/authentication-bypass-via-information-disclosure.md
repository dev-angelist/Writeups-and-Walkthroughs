---
description: >-
  https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-authentication-bypass
icon: vials
---

# Authentication bypass via information disclosure

## Description

This lab's administration interface has an authentication bypass vulnerability, but it is impractical to exploit without knowledge of a custom HTTP header used by the front-end.

To solve the lab, obtain the header name then use it to bypass the lab's authentication. Access the admin interface and delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

<figure><img src="../../../.gitbook/assets/image (27) (1) (1).png" alt=""><figcaption></figcaption></figure>

Starting access to wiener user [https://0ae80062039d9b5194f2429800130047.web-security-academy.net/login](https://0ae80062039d9b5194f2429800130047.web-security-academy.net/login)

Checking page sources there're not of interesting, so we can try access to admin page adding /admin path to our URL: [https://0a160012030fde108045dab900300013.web-security-academy.net/admin](https://0a160012030fde108045dab900300013.web-security-academy.net/admin)

and we can see this information disclosure message: "Admin interface only available to local users"

<figure><img src="../../../.gitbook/assets/image (28) (1) (1).png" alt=""><figcaption></figcaption></figure>

so, we start to analyze request for understanding more things

<figure><img src="../../../.gitbook/assets/image (29) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can try to change request method to see if the answer changes.

It will happen via TRACE method with the response status code 200.

<figure><img src="../../../.gitbook/assets/image (30) (1).png" alt=""><figcaption></figcaption></figure>

Trace is a debug method and it display us an important information as: X-Custom-IP-Authorization: 37.101.171.137

We can copy it inserting a localhost IP: 127.0.0.1 in our request and change again method to GET

<figure><img src="../../../.gitbook/assets/image (32) (1).png" alt=""><figcaption></figcaption></figure>

Now we're into admin panel having the permission to delete 'carlos' user

<figure><img src="../../../.gitbook/assets/image (34) (1).png" alt=""><figcaption></figcaption></figure>

analyzing the reponse, there's a URL that permits to delete 'carlos' account: `GET /admin/delete?username=carlos`

<figure><img src="../../../.gitbook/assets/image (35) (1).png" alt=""><figcaption></figcaption></figure>

Copy and add it into our request to delete account and solve the lab.

<figure><img src="../../../.gitbook/assets/image (36) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (37) (1).png" alt=""><figcaption></figcaption></figure>
