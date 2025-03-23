---
description: >-
  https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-on-debug-page
icon: vial-virus
---

# Information disclosure on debug page

## Description

This lab contains a debug page that discloses sensitive information about the application. To solve the lab, obtain and submit the `SECRET_KEY` environment variable.

## Solution

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Searching into page source (CTRL+U) the world 'debug' we found this info disclosure into comments, with a link for this config website: /cgi-bin/phpinfo.php

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

Adding it to the orginal URL ([https://0a5400be03dcbfb683c47334006b00dd.web-security-academy.net/cgi-bin/phpinfo.php](https://0a5400be03dcbfb683c47334006b00dd.web-security-academy.net/cgi-bin/phpinfo.php)) we can obtain info about php version and all other configration data (in our case the 'SECRET\_KEY')

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>
