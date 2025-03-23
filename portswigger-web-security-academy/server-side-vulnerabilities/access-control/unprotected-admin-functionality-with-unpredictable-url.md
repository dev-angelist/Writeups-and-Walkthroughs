---
description: >-
  https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-unprotected-admin-functionality-with-unpredictable-url
icon: vial-virus
---

# Unprotected admin functionality with unpredictable URL

#### [Unprotected functionality](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/unprotected-functionality-5i3h)

## Description

This lab has an unprotected admin panel. It's located at an unpredictable location, but the location is disclosed somewhere in the application.

Solve the lab by accessing the admin panel, and using it to delete the user `carlos`.

## Solution

<figure><img src="../../../.gitbook/assets/image (24) (1).png" alt=""><figcaption></figcaption></figure>

The situation is similar to the last lab, but in this case we can't access to robots.txt file idea is access to the admin panel, trying some path there're not results, then we can try to see the robots.txt file:&#x20;

<figure><img src="../../../.gitbook/assets/image (23) (1).png" alt=""><figcaption></figcaption></figure>

then, we can try to search a potentially information disclosure into page source (CTRL+U)

<figure><img src="../../../.gitbook/assets/image (25) (1).png" alt=""><figcaption></figcaption></figure>

checking for 'href' or eventually 'admin' we found a great reference that potentially can be the name of an admin panel page: `/admin-yqeueq` go there!

[https://0a5200610459467f82f57493001d0046.web-security-academy.net/admin-yqeueq](https://0a5200610459467f82f57493001d0046.web-security-academy.net/admin-yqeueq)

<figure><img src="../../../.gitbook/assets/image (26) (1).png" alt=""><figcaption></figcaption></figure>

Great, it's correct!

<figure><img src="../../../.gitbook/assets/image (22) (1).png" alt=""><figcaption></figcaption></figure>

Now, we can finishing deleting user Carlos:

<figure><img src="../../../.gitbook/assets/image (27) (1).png" alt=""><figcaption></figcaption></figure>
