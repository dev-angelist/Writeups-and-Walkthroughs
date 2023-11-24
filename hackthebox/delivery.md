# Delivery

<div align="left">

<figure><img src="../.gitbook/assets/c55af6eadd5b60bac831d73c1a951327.png" alt="" width="150"><figcaption><p>hackthebox.com - © HACKTHEBOX</p></figcaption></figure>

</div>

🔗 [Delivery](https://app.hackthebox.com/machines/Delivery)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.10.222`

Create a directory for machine on the Desktop and a directories containing: nmap, content, exploits, scripts materials.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
echo "10.10.10.222 delivery.htb" >> /etc/hosts

mkdir -p htb/delivery.htb
cd htb/delivery.htb
mkdir {nmap,content,exploits,scripts}
# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 delivery.htb
PING delivery.htb (10.10.10.222) 56(84) bytes of data.
64 bytes from delivery.htb (10.10.10.222): icmp_seq=1 ttl=63 time=67.4 ms
64 bytes from delivery.htb (10.10.10.222): icmp_seq=2 ttl=63 time=62.2 ms
64 bytes from delivery.htb (10.10.10.222): icmp_seq=3 ttl=63 time=61.7 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system, while Windows systems usually have a TTL of 128 secs.

### 2.1 - How many TCP ports are open on the target?

```bash
nmap --open -p0- -sS -n -Pn -vvv --min-rate 5000 delivery.htb -oG nmap/port_scan
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-10 13:48 EDT
Initiating SYN Stealth Scan at 13:48
Scanning delivery.htb (10.10.10.222) [65536 ports]
Discovered open port 80/tcp on 10.10.10.222
Discovered open port 22/tcp on 10.10.10.222
Discovered open port 8065/tcp on 10.10.10.222
Completed SYN Stealth Scan at 13:48, 13.48s elapsed (65536 total ports)
Nmap scan report for delivery.htb (10.10.10.222)
Host is up, received user-set (0.066s latency).
Scanned at 2023-08-10 13:48:25 EDT for 14s
Not shown: 65462 closed tcp ports (reset), 71 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 63
80/tcp   open  http    syn-ack ttl 63
8065/tcp open  unknown syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>vvv</td><td>verbosity</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 3 open TCP ports on the machine: 22, 80, 8065.

{% hint style="info" %}
3
{% endhint %}

### 2.2 - What is the FQDN for the Help Desk?

We do a specific scan using -sCV parameter:

```bash
nmap  -p22,80,8065 -n -Pn -sCV -vvv --min-rate 5000 delivery.htb -oN nmap/open_ports
```

```bash
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 9c:40:fa:85:9b:01:ac:ac:0e:bc:0c:19:51:8a:ee:27 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCq549E025Q9FR27LDR6WZRQ52ikKjKUQLmE9ndEKjB0i1qOoL+WzkvqTdqEU6fFW6AqUIdSEd7GMNSMOk66otFgSoerK6MmH5IZjy4JqMoNVPDdWfmEiagBlG3H7IZ7yAO8gcg0RRrIQjE7XTMV09GmxEUtjojoLoqudUvbUi8COHCO6baVmyjZRlXRCQ6qTKIxRZbUAo0GOY8bYmf9sMLf70w6u/xbE2EYDFH+w60ES2K906x7lyfEPe73NfAIEhHNL8DBAUfQWzQjVjYNOLqGp/WdlKA1RLAOklpIdJQ9iehsH0q6nqjeTUv47mIHUiqaM+vlkCEAN3AAQH5mB/1
|   256 5a:0c:c0:3b:9b:76:55:2e:6e:c4:f4:b9:5d:76:17:09 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAiAKnk2lw0GxzzqMXNsPQ1bTk35WwxCa3ED5H34T1yYMiXnRlfssJwso60D34/IM8vYXH0rznR9tHvjdN7R3hY=
|   256 b7:9d:f7:48:9d:a2:f2:76:30:fd:42:d3:35:3a:80:8c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEV5D6eYjySqfhW4l4IF1SZkZHxIRihnY6Mn6D8mLEW7
80/tcp   open  http    syn-ack ttl 63 nginx 1.14.2
|_http-title: Welcome
|_http-server-header: nginx/1.14.2
| http-methods: 
|_  Supported Methods: GET HEAD
8065/tcp open  unknown syn-ack ttl 63
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest, SSLSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Accept-Ranges: bytes
|     Cache-Control: no-cache, max-age=31556926, public
|     Content-Length: 3108
|     Content-Security-Policy: frame-ancestors 'self'; script-src 'self' cdn.rudderlabs.com
|     Content-Type: text/html; charset=utf-8
|     Last-Modified: Thu, 10 Aug 2023 04:15:56 GMT
|     X-Frame-Options: SAMEORIGIN
|     X-Request-Id: m6r8ow4omfni7q1iy3jdqzntae
|     X-Version-Id: 5.30.0.5.30.1.57fb31b889bf81d99d8af8176d4bbaaa.false
|     Date: Thu, 10 Aug 2023 17:55:14 GMT
|     <!doctype html><html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=0"><meta name="robots" content="noindex, nofollow"><meta name="referrer" content="no-referrer"><title>Mattermost</title><meta name="mobile-web-app-capable" content="yes"><meta name="application-name" content="Mattermost"><meta name="format-detection" content="telephone=no"><link re
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Date: Thu, 10 Aug 2023 17:55:14 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8065-TCP:V=7.94%I=7%D=8/10%Time=64D52481%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(GetRequest,DF3,"HTTP/1\.0\x20200\x20OK\r\nAccept-Ranges:\
SF:x20bytes\r\nCache-Control:\x20no-cache,\x20max-age=31556926,\x20public\
SF:r\nContent-Length:\x203108\r\nContent-Security-Policy:\x20frame-ancesto
SF:rs\x20'self';\x20script-src\x20'self'\x20cdn\.rudderlabs\.com\r\nConten
SF:t-Type:\x20text/html;\x20charset=utf-8\r\nLast-Modified:\x20Thu,\x2010\
SF:x20Aug\x202023\x2004:15:56\x20GMT\r\nX-Frame-Options:\x20SAMEORIGIN\r\n
SF:X-Request-Id:\x20m6r8ow4omfni7q1iy3jdqzntae\r\nX-Version-Id:\x205\.30\.
SF:0\.5\.30\.1\.57fb31b889bf81d99d8af8176d4bbaaa\.false\r\nDate:\x20Thu,\x
SF:2010\x20Aug\x202023\x2017:55:14\x20GMT\r\n\r\n<!doctype\x20html><html\x
SF:20lang=\"en\"><head><meta\x20charset=\"utf-8\"><meta\x20name=\"viewport
SF:\"\x20content=\"width=device-width,initial-scale=1,maximum-scale=1,user
SF:-scalable=0\"><meta\x20name=\"robots\"\x20content=\"noindex,\x20nofollo
SF:w\"><meta\x20name=\"referrer\"\x20content=\"no-referrer\"><title>Matter
SF:most</title><meta\x20name=\"mobile-web-app-capable\"\x20content=\"yes\"
SF:><meta\x20name=\"application-name\"\x20content=\"Mattermost\"><meta\x20
SF:name=\"format-detection\"\x20content=\"telephone=no\"><link\x20re")%r(H
SF:TTPOptions,5B,"HTTP/1\.0\x20405\x20Method\x20Not\x20Allowed\r\nDate:\x2
SF:0Thu,\x2010\x20Aug\x202023\x2017:55:14\x20GMT\r\nContent-Length:\x200\r
SF:\n\r\n")%r(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConten
SF:t-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n
SF:400\x20Bad\x20Request")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r
SF:\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close
SF:\r\n\r\n400\x20Bad\x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x2
SF:0Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nCon
SF:nection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(TerminalServerCookie
SF:,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;
SF:\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request"
SF:);
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```





Now, we try to find potential hidden directory using gobuster:

```bash
gobuster dir -u http://delivery.htb/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
```

```bash
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://delivery.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/08/10 14:13:03 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 185] [--> http://delivery.htb/images/]
/assets               (Status: 301) [Size: 185] [--> http://delivery.htb/assets/]
/error                (Status: 301) [Size: 185] [--> http://delivery.htb/error/]
Progress: 87617 / 87665 (99.95%)
===============================================================
```

and we find an interesting path:&#x20;

http://delivery.htb/images/

http://delivery.htb/assets/ http://delivery.htb/error/]















Going to http:\\\delivery.htb page, and start to check it.

<figure><img src="../.gitbook/assets/Schermata del 2023-08-11 20-13-58.png" alt=""><figcaption></figcaption></figure>

Clicking on "HELPDESK" link, we found this error page:

<figure><img src="../.gitbook/assets/Schermata del 2023-08-11 20-16-22.png" alt=""><figcaption></figcaption></figure>

While, clicking on contact-us and next to MatterMost server link, we found a great resource!

<figure><img src="../.gitbook/assets/Schermata del 2023-08-11 20-28-58.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Schermata del 2023-08-11 20-29-09.png" alt=""><figcaption></figcaption></figure>



{% hint style="info" %}

{% endhint %}











{% hint style="info" %}

{% endhint %}

### 2.3 -&#x20;







We can try to put a file using ftp, in this case we use nmap result file (port\_scan):

```bash
ftp> put port_scan 
```

```bash
ftp> put port_scan 
local: port_scan remote: port_scan
229 Entering Extended Passive Mode (|||49219|)
150 Opening ASCII mode data connection.
100% |***************************************************************************************|   464        8.84 MiB/s    --:-- ETA
226 Transfer complete.
464 bytes sent in 00:00 (7.98 KiB/s)
ftp> ls
229 Entering Extended Passive Mode (|||49220|)
125 Data connection already open; Transfer starting.
07-24-23  11:15AM               241062 40564.exe
03-18-17  02:06AM       <DIR>          aspnet_client
07-24-23  01:26AM                 1442 cmdasp.aspx
07-24-23  12:36AM                 2914 devel.aspx
07-24-23  01:04AM                 2886 devel1.aspx
07-24-23  04:44PM                 2917 devel2.aspx
07-24-23  02:11AM                 2749 develshell.aspx
07-24-23  11:09AM                15966 fox.aspx
07-24-23  09:26AM                 2906 hacked.aspx
03-17-17  05:37PM                  689 iisstart.htm
07-24-23  07:16PM                    0 killbill.aspx
07-24-23  07:21PM                 2912 killbill1.aspx
07-24-23  10:57PM                  464 port_scan
07-24-23  12:17AM                 2783 pwned.aspx
07-24-23  03:00PM                 2923 rev.aspx
07-24-23  09:21PM                15969 shell.aspx
07-24-23  03:34PM                73802 virus.exe
07-24-23  12:34AM               112815 virus2.exe
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
```

{% hint style="info" %}
yes
{% endhint %}

### 2.4 -&#x20;







```bash
```

{% hint style="info" %}

{% endhint %}

### 2.5 -&#x20;

We launch msfconsole:

```bash
msfconsole
```



{% hint style="info" %}

{% endhint %}

### Task 3 -&#x20;

### 3.1 -&#x20;

\


We've not access to babibs' directory, we can try to find "user.txt" flag using while command in C:\ root.

```bash
where /r C:\ user.txt
```

<div align="left">

<figure><img src="../.gitbook/assets/Schermata del 2023-07-25 20-45-43.png" alt=""><figcaption></figcaption></figure>

</div>

Starting to root folder (C:\\) we can find quickly flags, using where command in recusive mode (/r):

```
where /r C:\ user.txt
C:\Users\babis\Desktop\user.txt
```

and read user.txt flag using type command (equivalent to cat on \*nix):

```
type C:\Users\babis\Desktop\user.txt
```

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

5d3fc209e1fae6d5df926fe7dc8a16bd

</details>

### Task 4 - Find root flag

After that, we do the same thing for root.txt flag

```bash
where /r C:\ root.txt
C:\Users\Administrator\Desktop\root.txt
```

```bash
type C:\Users\Administrator\Desktop\root.txt
```

<details>

<summary>🚩 Flag 2 (root.txt)</summary>

cb43e154f9c2ca60b68c8150e5162f32

</details>
