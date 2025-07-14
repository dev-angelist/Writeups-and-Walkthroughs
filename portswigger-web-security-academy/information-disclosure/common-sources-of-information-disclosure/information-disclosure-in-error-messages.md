---
description: >-
  https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-in-error-messages
icon: flask-vial
---

# Information disclosure in error messages

## Description

This lab's verbose error messages reveal that it is using a vulnerable version of a third-party framework. To solve the lab, obtain and submit the version number of this framework.

## Solution

<figure><img src="../../../.gitbook/assets/image (16) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Checking into page source code there're not of interesting, so click to one of products shop: [https://0a6c00fe03599c7f8a11a3f700100029.web-security-academy.net/product?productId=1](https://0a6c00fe03599c7f8a11a3f700100029.web-security-academy.net/product?productId=1)

the idea is to generate and error, than we try to inject something with a SQLi:

<figure><img src="../../../.gitbook/assets/image (17) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and obtaining an error by Apache Struts 2 2.3.31 we've discovered the vs number of this framework.

<figure><img src="../../../.gitbook/assets/image (15) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
