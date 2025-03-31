# XSS Reflected

## XSS - Reflected - First Order

Go to login page form [https://127.0.0.1/index.php?page=login.php](https://127.0.0.1/index.php?page=login.php)

and log in using login bypass or inserting password.

Go to a page vulnerable to XSS reflected like as: [https://127.0.0.1/index.php?page=dns-lookup.php](https://127.0.0.1/index.php?page=dns-lookup.php)

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

injecting the javascript payload: `<script>alert(document.cookie)</script>` we can execute directly the command and obtain a reflected session cookie id of current account (admin):

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
