---
description: >-
  https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/ssrf-apprentice/ssrf/lab-basic-ssrf-against-localhost
icon: vial-virus
---

# Basic SSRF against the local server

## Description

This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.

## Solution

<figure><img src="../../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

Every product has a dedicate check function to retrieve if a product is availble or not:

value="http://stock.weliketoshop.net:8080/product/stock/check?productId=1\&storeId=1"

<figure><img src="../../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

clicking to "Check stock" button we obtain the number of pieces in stock

<figure><img src="../../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

We can change the stock check URL to access the admin interface inseriting the stockApi value selecting it and updating it into Inspector field at `http://localhost/admin`

<figure><img src="../../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

In this way we're able to access in the admin panel via a SSRF,  and checking into the reponse there're links for deleting users Wiener and Carlos:

<figure><img src="../../../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

The scope of the lab is to delete Carlos user, we can do it inserting the deletion link into stockApi value and solving the lab:

<figure><img src="../../../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>
