---
icon: vial-virus
---

# User ID controlled by request parameter, with unpredictable user IDs

#### [Horizontal privilege escalation](https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/horizontal-privilege-escalation)

## Description

This lab has a horizontal privilege escalation vulnerability on the user account page, but identifies users with GUIDs.

To solve the lab, find the GUID for `carlos`, then submit his API key as the solution.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

<figure><img src="../../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

we can start login as wiener user

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

we obtain Wiener's API Key.

We can try to return to Home page, and check if there're referrement to Carlos like as posts.

[https://0a3200fc04ca12b780505890009300fd.web-security-academy.net/post?postId=3](https://0a3200fc04ca12b780505890009300fd.web-security-academy.net/post?postId=3)

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

Capturing HTTP response we discover that userId value was changed

<figure><img src="../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

Save it (Carlos userID): f26a0928-06ae-4b0d-be0a-ca03266160f0

Go back to My Account page and change the reference adding the new userID:

[https://0a3200fc04ca12b780505890009300fd.web-security-academy.net/my-account?id=f26a0928-06ae-4b0d-be0a-ca03266160f0](https://0a3200fc04ca12b780505890009300fd.web-security-academy.net/my-account?id=f26a0928-06ae-4b0d-be0a-ca03266160f0)

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

horizontal privilege escalation done!

Send the Carlos' API Key: NEvvgurN9IMYbP0WGQhRNxGCKLHuboPn

<figure><img src="../../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>
