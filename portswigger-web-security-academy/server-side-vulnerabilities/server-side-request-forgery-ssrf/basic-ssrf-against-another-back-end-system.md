---
description: >-
  https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/ssrf-apprentice/ssrf/lab-basic-ssrf-against-backend-system
icon: vial-virus
---

# Basic SSRF against another back-end system

## Description

This lab has a stock check feature which fetches data from an internal system.

To solve the lab, use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port `8080`, then use it to delete the user `carlos`.

## Solution

<figure><img src="../../../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

Every product has a dedicate check function to retrieve if a product is availble or not:

value="http://192.168.0.1:8080/product/stock/check?productId=2\&storeId=1"

<figure><img src="../../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

clicking to "Check stock" button we obtain the number of pieces in stock

<figure><img src="../../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

We can try to change the stock check URL to access the admin interface inseriting the stockApi value selecting it and updating it into Inspector field at `http://`192.168.0.1:8080`/admin` but the page doesn't exists. So, remember that in the lab description was indicated as URL: `http://`192.168.0.X:8080 we can try to change the x value using Burp Intruder:

<figure><img src="../../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

as result we obtain a different length response for the value: 157.

Changing it: `http://192.168.0.157:8080/admini`n this way we're able to access in the admin panel via a SSRF:

<figure><img src="../../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

&#x20;and checking into the reponse there're links for deleting users Wiener and Carlos:

<figure><img src="../../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

The scope of the lab is to delete Carlos user, we can do it inserting the deletion link into stockApi value and solving the lab:

<figure><img src="../../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>
