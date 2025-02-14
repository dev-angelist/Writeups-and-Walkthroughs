# XSS Stored

## XSS - Stored - Second Order

Go to login page form [https://127.0.0.1/index.php?page=login.php](https://127.0.0.1/index.php?page=login.php)

and log in using login bypass or inserting password.

Go to a page vulnerable to XSS stored like as: [https://127.0.0.1/index.php?page=add-to-your-blog.php](https://127.0.0.1/index.php?page=add-to-your-blog.php)

<figure><img src="../../.gitbook/assets/image (33) (1).png" alt=""><figcaption></figcaption></figure>

in this textarea (not sanitizated) we can add whatever we want, save it and it will be stored internally and display to users that will click on 'View Blogs'.

<figure><img src="../../.gitbook/assets/image (34) (1).png" alt=""><figcaption></figcaption></figure>

injecting the javascript payload: `<script>alert(document.cookie)</script> the`command will be execute on the click of the page: [https://127.0.0.1/index.php?page=view-someones-blog.php](https://127.0.0.1/index.php?page=view-someones-blog.php) clicking on View Blog Entries:

<figure><img src="../../.gitbook/assets/image (35) (1).png" alt=""><figcaption></figcaption></figure>



