---
description: >-
  https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-user-role-controlled-by-request-parameter
icon: vial-virus
---

# User role controlled by request parameter

#### [Parameter-based access control methods](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/parameter-based-access-control-methods)

## Description

This lab has an admin panel at `/admin`, which identifies administrators using a forgeable cookie.

Solve the lab by accessing the admin panel and using it to delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

<figure><img src="../../../.gitbook/assets/image (28) (1) (1).png" alt=""><figcaption></figcaption></figure>

as indicated into description, we know the admin panel path, but without admin credentials we can't do nothing,&#x20;

<figure><img src="../../../.gitbook/assets/image (33) (1).png" alt=""><figcaption></figcaption></figure>

so we can do normal login with followring credentials: `wiener:peter`

<figure><img src="../../../.gitbook/assets/image (29) (1) (1).png" alt=""><figcaption></figcaption></figure>

analysing the http response after login, we can see that there're into cookie and admin flag set to false and the session ID:

<figure><img src="../../../.gitbook/assets/image (30) (1).png" alt=""><figcaption></figcaption></figure>

Setting the admin flag to 'true' we can do a vertical privilege escalation

<figure><img src="../../../.gitbook/assets/image (32) (1).png" alt=""><figcaption></figcaption></figure>

then, go to admin panel and set the flag equals true:

<figure><img src="../../../.gitbook/assets/image (37) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (35) (1).png" alt=""><figcaption></figcaption></figure>

Now, we can finishing deleting user Carlos:

<figure><img src="../../../.gitbook/assets/image (36) (1).png" alt=""><figcaption></figcaption></figure>
