---
description: >-
  https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding
icon: flask-vial
---

# SQL injection with filter bypass via XML encoding

## Description

This lab contains a SQL injection vulnerability in its stock check feature. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables.

The database contains a `users` table, which contains the usernames and passwords of registered users. To solve the lab, perform a SQL injection attack to retrieve the admin user's credentials, then log in to their account.

## Solution

<figure><img src="../../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Click to one of products shop: [https://0a5b002b0394c64382a61f0e00eb00c9.web-security-academy.net/product?productId=1](https://0a5b002b0394c64382a61f0e00eb00c9.web-security-academy.net/product?productId=1)

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

Analyzing HTTP request of this request with BurpSuite there're not XML, so this is the wrong via.

<figure><img src="../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

See well the page, there's a form with a method POST that permits to display the stock value of relative product in three various store place.

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

Capturing it, we can see the XML that we're searching!

<figure><img src="../../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

We can start trying to injecting this payload: `1 UNION SELECT NULL` for understand if the print of 'NULL' value will be executed

<figure><img src="../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

WAF identify a malicious payload, so we can try to encode our payload using tools

Install HackVector (Optional), we can use other web tools

<figure><img src="../../../.gitbook/assets/image (464).png" alt=""><figcaption></figcaption></figure>

Encoding the payload to HEX\_EntitIes: `1 <@hex_entities>UNION SELECT NULL`\
`</@hex_entities>` we're able to evade WAF and obtain 'NULL' as result:

<figure><img src="../../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

So, adding a new one 'NULL' value we can see that the 2nd new column doesn't exists, than the table's column are only one,&#x20;

<figure><img src="../../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

remembering the task there're not problem "The database contains a `users` table, which contains the usernames and passwords of registered users. ", column requested are two, we can ovviate to this thing concatening the two parameters:

1 <@hex\_entities>UNION SELECT username || '\~' || password FROM USERS\
\</@hex\_entities>

<figure><img src="../../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

and obtaining credentials for login and complete the lab.

<figure><img src="../../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>
