# LazyAdmin

<div align="left">

<figure><img src="../.gitbook/assets/efbb70493ba66dfbac4302c02ad8facf.jpeg" alt="" width="159"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

🔗 [LazyAdmin](https://tryhackme.com/room/lazyadmin)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.129.248`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

```bash
su
echo "10.10.129.248 lazyadmin.thm" >> /etc/hosts

mkdir thm/lazyadmin.thm  
cd thm/lazyadmin.thm

# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 lazyadmin.thm
PING lazyadmin.thm (10.10.129.248) 56(84) bytes of data.
64 bytes from lazyadmin.thm (10.10.129.248): icmp_seq=1 ttl=63 time=448 ms
64 bytes from lazyadmin.thm (10.10.129.248): icmp_seq=2 ttl=63 time=658 ms
64 bytes from lazyadmin.thm (10.10.129.248): icmp_seq=3 ttl=63 time=141 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

#### 2.1 - Find open ports on the machine

```bash
nmap --open -n -Pn -vvv -T4 lazyadmin.thm
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-29 15:42 EDT
Initiating SYN Stealth Scan at 15:42
Scanning lazyadmin.thm (10.10.129.248) [1000 ports]
Discovered open port 22/tcp on 10.10.129.248
Discovered open port 80/tcp on 10.10.129.248
Completed SYN Stealth Scan at 15:42, 3.60s elapsed (1000 total ports)
Nmap scan report for lazyadmin.thm (10.10.129.248)
Host is up, received user-set (0.11s latency).
Scanned at 2023-06-29 15:42:27 EDT for 4s
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

```bash
nmap -p22,80 -sCV -vvv lazyadmin.thm
```

```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCo0a0DBybd2oCUPGjhXN1BQrAhbKKJhN/PW2OCccDm6KB/+sH/2UWHy3kE1XDgWO2W3EEHVd6vf7SdrCt7sWhJSno/q1ICO6ZnHBCjyWcRMxojBvVtS4kOlzungcirIpPDxiDChZoy+ZdlC3hgnzS5ih/RstPbIy0uG7QI/K7wFzW7dqMlYw62CupjNHt/O16DlokjkzSdq9eyYwzef/CDRb5QnpkTX5iQcxyKiPzZVdX/W8pfP3VfLyd/cxBqvbtQcl3iT1n+QwL8+QArh01boMgWs6oIDxvPxvXoJ0Ts0pEQ2BFC9u7CgdvQz1p+VtuxdH6mu9YztRymXmXPKJfB
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBC8TzxsGQ1Xtyg+XwisNmDmdsHKumQYqiUbxqVd+E0E0TdRaeIkSGov/GKoXY00EX2izJSImiJtn0j988XBOTFE=
|   256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILe/TbqqjC/bQMfBM29kV2xApQbhUXLFwFJPU14Y9/Nm
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: POST OPTIONS GET HEAD
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

It looks like there are only two open ports on the machine: SSH and HTTP.

#### Task 3 - What is the user flag? 

#### We can strat to explore http://lazyadmin.thm (port 80)

<figure><img src="../.gitbook/assets/Schermata del 2023-06-30 00-12-42.png" alt=""><figcaption></figcaption></figure>

In the page source code we don't found nothing of interisting, the good route is to explore website hidden pathes using gobuster:

```bash
gobuster dir -u lazyadmin.thm -w /usr/share/wordlists/dirb/common.txt  
```

```bash
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://lazyadmin.thm/content/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/06/29 15:51:01 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/_themes              (Status: 301) [Size: 324] [--> http://lazyadmin.thm/content/_themes/]
/as                   (Status: 301) [Size: 319] [--> http://lazyadmin.thm/content/as/]
/attachment           (Status: 301) [Size: 327] [--> http://lazyadmin.thm/content/attachment/]
/images               (Status: 301) [Size: 323] [--> http://lazyadmin.thm/content/images/]
/inc                  (Status: 301) [Size: 320] [--> http://lazyadmin.thm/content/inc/]
/index.php            (Status: 200) [Size: 2199]
/js                   (Status: 301) [Size: 319] [--> http://lazyadmin.thm/content/js/]
Progress: 4597 / 4615 (99.61%)
===============================================================
2023/06/29 15:53:05 Finished
===============================================================
```

We found an administrator login page:

<figure><img src="../.gitbook/assets/Schermata del 2023-06-30 00-18-34.png" alt=""><figcaption></figcaption></figure>

We try default user and password but it doesn't work, then, we see that website uses SweetRice how CMS. We can use searchsploit to find and eventually exploit:

```bash
searchsploit sweetrice
```

```bash
---------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                         |  Path
----------------------------------------------------------------------------------------------------------------------- ---------------------------------
SweetRice 0.5.3 - Remote File Inclusion                                                                                | php/webapps/10246.txt
SweetRice 0.6.7 - Multiple Vulnerabilities                                                                             | php/webapps/15413.txt
SweetRice 1.5.1 - Arbitrary File Download                                                                              | php/webapps/40698.py
SweetRice 1.5.1 - Arbitrary File Upload                                                                                | php/webapps/40716.py
SweetRice 1.5.1 - Backup Disclosure                                                                                    | php/webapps/40718.txt
SweetRice 1.5.1 - Cross-Site Request Forgery                                                                           | php/webapps/40692.html
SweetRice 1.5.1 - Cross-Site Request Forgery / PHP Code Execution                                                      | php/webapps/40700.html
SweetRice < 0.6.4 - 'FCKeditor' Arbitrary File Upload                                                                  | php/webapps/14184.txt
----------------------------------------------------------------------------------------------------------------------- ---------------------------------
```











```bash
```

<details>

<summary>🚩 Flag 1 (user.txt)</summary>



</details>

#### Task 4 - What is the root flag?  

\


```bash
# whoami
root
```

Now, we're root!

```bash
# cd /root/
# ls
root.txt
# cat root.txt
```

<details>

<summary>🚩 Flag 2 (root.txt)</summary>



</details>
