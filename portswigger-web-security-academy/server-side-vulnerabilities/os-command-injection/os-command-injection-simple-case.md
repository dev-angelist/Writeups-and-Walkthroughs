---
description: >-
  https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/os-command-injection-apprentice/os-command-injection/lab-simple
icon: vial-virus
---

# OS command injection, simple case

## Description

This lab contains an OS command injection vulnerability in the product stock checker.

The application executes a shell command containing user-supplied product and store IDs, and returns the raw output from the command in its response.

To solve the lab, execute the `whoami` command to determine the name of the current user.

## Solution

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Every product has a dedicate check function to retrieve if a product is availble or not:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>





<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

We can try to concatenate it with a payload like as ; & or |: `;whoami`

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

and we obtain the user of system: peter-Gu9oqX solving the lab.

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
