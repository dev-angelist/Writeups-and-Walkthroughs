---
icon: flask-vial
---

# File path traversal, simple case

## Description

This lab contains a path traversal vulnerability in the display of product images.

To solve the lab, retrieve the contents of the `/etc/passwd` file.

## Solution

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Click to one of products shop: [https://0ac8007304f7f39b81adf85000a300b1.web-security-academy.net/product?productId=2](https://0ac8007304f7f39b81adf85000a300b1.web-security-academy.net/product?productId=2)

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The parameter productsID seems to not be vulnerable, than we can try to open the relative image, that usually is located into `/var/www/images` web server directory.

[https://0ac8007304f7f39b81adf85000a300b1.web-security-academy.net/image?filename=1.jpg](https://0ac8007304f7f39b81adf85000a300b1.web-security-academy.net/image?filename=1.jpg)

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

In this case directory and parameter are different, capturing it with Burp and trasfer to Repeater using CTRL+R

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here, we can modify the filename reference adding `../../../../../etc/passwd` to do five jump back into directory, arriving to root `/` and accessing to `/etc/passwd` file:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
