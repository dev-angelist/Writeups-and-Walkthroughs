# Web Shell

## Lab 16: Command injection - Web Shell with Command injection

<figure><img src="../../.gitbook/assets/image (14) (1) (1).png" alt=""><figcaption></figcaption></figure>

Go to upload file page form [https://127.0.0.1/index.php?page=upload-file.php](https://127.0.0.1/index.php?page=upload-file.php)

<figure><img src="../../.gitbook/assets/image (22) (1).png" alt=""><figcaption></figcaption></figure>

Here we can upload a php reverse shell and custom one of the shells already present into kali linux distribution at the path: /usr/share/webshells/php (in my case i choose php-reverse-shell.php)

<figure><img src="../../.gitbook/assets/image (20) (1).png" alt=""><figcaption></figcaption></figure>

adding our Kali\_IP (10.0.2.15) and choosing a port (1339), save and upload it.

<figure><img src="../../.gitbook/assets/image (23) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (21) (1).png" alt=""><figcaption></figcaption></figure>

Now we need to take in listening mode on the attacker machine via netcat: nc -lvnp 1339 and go to relative web page to run the reverse shell: [https://127.0.0.1/index.php?page=/tmp/php-reverse-shell.php](https://127.0.0.1/index.php?page=php-reverse-shell.php)

<figure><img src="../../.gitbook/assets/image (26) (1).png" alt=""><figcaption></figcaption></figure>
