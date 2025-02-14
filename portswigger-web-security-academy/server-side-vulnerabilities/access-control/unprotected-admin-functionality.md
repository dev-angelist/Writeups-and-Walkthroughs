---
description: >-
  https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-unprotected-admin-functionality
icon: vial-virus
---

# Unprotected admin functionality

#### [Unprotected functionality](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/unprotected-functionality)

## Description

This lab has an unprotected admin panel.

Solve the lab by deleting the user `carlos`.

## Solution

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

The idea is access to the admin panel, trying some path there're not results, then we can try to see the robots.txt file: [https://0acf00c003d580aedfc3cb23003400e9.web-security-academy.net/robots.txt](https://0acf00c003d580aedfc3cb23003400e9.web-security-academy.net/robots.txt)

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

here was inserted the admin panel page to disallow it on google searches.

Then go there: [https://0acf00c003d580aedfc3cb23003400e9.web-security-academy.net/administrator-panel](https://0acf00c003d580aedfc3cb23003400e9.web-security-academy.net/administrator-panel)

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

and eliminate user Carlos clicking to Delete

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>
