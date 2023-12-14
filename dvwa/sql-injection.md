---
description: http://localhost/DVWA/vulnerabilities/sqli/
---

# SQL Injection --

<details>

<summary>What is a SQL Injection?</summary>

SQL injection is a type of security vulnerability that occurs when an attacker is able to manipulate a SQL query by injecting malicious SQL code into user-input fields or other parameters. This can happen when an application does not properly validate or sanitize user inputs before constructing SQL queries.

SQL injection attacks are most common in web applications that interact with a database. When user inputs are directly concatenated into SQL queries without proper validation, an attacker can input specially crafted strings that alter the intended logic of the SQL query.

</details>

{% hint style="info" %}
Using BurpSuite and the FoxyProxy extension is recommended.
{% endhint %}

## Low

We've an input type text that received an User ID in I by user and submit request using the Submit button:

<figure><img src="../.gitbook/assets/image (112).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>

This's our request captured by Burp Suite, while here below there's a php source code:

<figure><img src="../.gitbook/assets/image (114).png" alt=""><figcaption><p>localhost/DVWA/vulnerabilities/view_source.php?id=sqli&#x26;security=low</p></figcaption></figure>

How the same in the low level, there're not input sanitation, then we can send what we want into input type text field. In this case request will arrive to DB located into webserver, but query will be preparared using php language.

Analyzing source code, we know that mysql query is:

```sql
SELECT first_name, last_name FROM users WHERE user_id = '$id';
```

#### 1st Payload

Regarding query and that our input type value is insert into $id variable, if we want to see all rows of DB, we need to send a query request always boolean true (e.g. 1=1), in addition we can hide all parts not important using ' character.

Then in this case, we can use following payload: `' OR 1=1 --`&#x20;

```sql
SELECT first_name, last_name FROM users WHERE user_id = '' OR 1=1 -- ';
```

in the where condition there're a first search to user with this id '' OR a true condition, plus a comment.

This permit us to see all DB results:

<div align="left">

<figure><img src="../.gitbook/assets/image (115).png" alt=""><figcaption></figcaption></figure>

</div>

#### 2nd Payload

Regarding that query selects: first\_name, last\_name field from users table, we can use [UNION](https://www.w3schools.com/sql/sql\_union.asp) operator to add a new query:  `' UNION select first_name,password from users --`&#x20;

```sql
SELECT first_name, last_name FROM users WHERE user_id = '' UNION SELECT first_name,password FROM users -- ';
```

<div align="left">

<figure><img src="../.gitbook/assets/image (117).png" alt=""><figcaption></figcaption></figure>

</div>

Note: Every `SELECT` statement within `UNION` must have the same number of columns.

{% hint style="warning" %}
The input is not sanitized, so I can execute any (potentially malicious) command.
{% endhint %}

## Medium



<div align="left">

<figure><img src="../.gitbook/assets/image (118).png" alt=""><figcaption></figcaption></figure>

</div>



<figure><img src="../.gitbook/assets/image (119).png" alt=""><figcaption></figcaption></figure>







## High





<figure><img src="../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>





<figure><img src="../.gitbook/assets/image (121).png" alt=""><figcaption></figcaption></figure>





## Impossible



## References

For the making of this solution the following resource were used:

* [https://github.com/LeonardoE95/DVWA/tree/main/src/sqli](https://github.com/LeonardoE95/DVWA/tree/main/src/sqli)

{% content-ref url="https://app.gitbook.com/s/iS3hadq7jVFgSa8k5wRA/practical-ethical-hacker-notes/main-contents/15-sql-injection" %}
[15 - SQL Injection](https://app.gitbook.com/s/iS3hadq7jVFgSa8k5wRA/practical-ethical-hacker-notes/main-contents/15-sql-injection)
{% endcontent-ref %}

{% content-ref url="https://app.gitbook.com/s/iS3hadq7jVFgSa8k5wRA/practical-ethical-hacker-notes/tools/sqlmap" %}
[SQLMap](https://app.gitbook.com/s/iS3hadq7jVFgSa8k5wRA/practical-ethical-hacker-notes/tools/sqlmap)
{% endcontent-ref %}
