---
description: >-
  https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-password-disclosure
icon: vial-virus
---

# User ID controlled by request parameter with password disclosure

#### [Horizontal to vertical privilege escalation](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/horizontal-to-vertical-privilege-escalation)

## Description

This lab has user account page that contains the current user's existing password, prefilled in a masked input.

To solve the lab, retrieve the administrator's password, then use it to delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

<figure><img src="../../../.gitbook/assets/image (16) (1) (1).png" alt=""><figcaption></figcaption></figure>

we can start login as wiener user

<figure><img src="../../../.gitbook/assets/image (17) (1) (1).png" alt=""><figcaption></figcaption></figure>

Checking this request we can see that this page has as the username 'wiener' as id parameter

<figure><img src="../../../.gitbook/assets/image (18) (1) (1).png" alt=""><figcaption></figcaption></figure>

and in the response there's a cleartext password!

<figure><img src="../../../.gitbook/assets/image (19) (1) (1).png" alt=""><figcaption></figcaption></figure>

So, trying to change the id with 'admnistrator' we're able to disclosure administrator's password:

<figure><img src="../../../.gitbook/assets/image (20) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (21) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we can login as administrator, access to Admin panel and delte 'Carlos' user completing the lab!

<figure><img src="../../../.gitbook/assets/image (22) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (23) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (24) (1) (1).png" alt=""><figcaption></figcaption></figure>
