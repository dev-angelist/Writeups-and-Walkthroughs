# XSS DOM-Based

## XSS - DOM-Based

Go to login page form [https://127.0.0.1/index.php?page=login.php](https://127.0.0.1/index.php?page=login.php)

and log in using login bypass or inserting password.

Go to a page vulnerable to XSS stored like as: [https://127.0.0.1/index.php?page=add-to-your-blog.php](https://127.0.0.1/index.php?page=add-to-your-blog.php)

<figure><img src="../../.gitbook/assets/image (41) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

in this two field will inject our payload:

```javascript
element.innerHTML='... <img src=1 onerror=alert(document.cookie)> ...'
```

the image 1 does not exist, so the alert will be triggered, the command will be injected into DOM and execute on the page:&#x20;

<figure><img src="../../.gitbook/assets/image (38) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (40) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (39) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
