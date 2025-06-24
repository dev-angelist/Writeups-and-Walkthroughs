# Extracting User Accounts

## Lab 12: Command injection - Extracting User Accounts with Command Injection

<figure><img src="../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Go to DNS Lookup page: [https://127.0.0.1/index.php?popUpNotificationCode=AU1\&page=dns-lookup.php](https://127.0.0.1/index.php?popUpNotificationCode=AU1\&page=dns-lookup.php)

We can use a valid value and concatenate it with an operator like as ; & | etc to insert a command not allowed: `8.8.8.8;whoami`

<figure><img src="../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Our last command confirm the vulnerability, however we know that the userlist are retrievable into /etc/passwd file and using this payload: `8.8.8.8;cat /etc/passwd`

<figure><img src="../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
