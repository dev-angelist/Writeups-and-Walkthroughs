# SQLi Login Bypass

## Lab 7: SQL Injection - Using SQLi to Bypass Authentication

<figure><img src="../../.gitbook/assets/image (441).png" alt=""><figcaption></figcaption></figure>

Go to login page form [https://127.0.0.1/index.php?page=login.php](https://127.0.0.1/index.php?page=login.php)

and check if the form is vulnerable to SQL injection vulnerability inserting a 'broken payload' for SQL such as: single quote ('):

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Very good, this is the typical error message of MySQL, than we can inject a malicious payload: `' OR 1=1 --` as username to bypass login:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and login as admin:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
