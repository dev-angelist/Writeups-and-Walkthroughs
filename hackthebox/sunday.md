---
description: https://www.hackthebox.com/machines/sunday
---

# Sunday

🔗 [Sunday](https://www.hackthebox.com/machines/sunday)

<div align="left"><figure><img src="../.gitbook/assets/image (1) (1).png" alt="" width="150"><figcaption><p>@hackthebox.com</p></figcaption></figure></div>

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

🎯 Target IP: `10.129.237.16`

Create a directory on the Desktop with the machine's name, and inside this directory, create another directory to store the materials and outputs needed to run the machine, including the scans made with nmap.

## Task 1 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
<strong>echo "10.129.237.16 sunday.htb" >> /etc/hosts
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
PING sunday.htb (10.129.237.16) 56(84) bytes of data.
64 bytes from sunday.htb (10.129.237.16): icmp_seq=1 ttl=254 time=53.7 ms
64 bytes from sunday.htb (10.129.237.16): icmp_seq=2 ttl=254 time=50.8 ms
64 bytes from sunday.htb (10.129.237.16): icmp_seq=3 ttl=254 time=54.7 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) isn't 64 or 128 secs. This is a little strange and googling we can see that our target is a **Solaris OS** system.

<div align="left"><figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure></div>

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

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>







```bash
whatweb sau.htb:55555
http://sau.htb:55555 [302 Found] Country[RESERVED][ZZ], IP[10.129.229.26], RedirectLocation[/web]
http://sau.htb:55555/web [200 OK] Bootstrap[3.3.7], Country[RESERVED][ZZ], HTML5, IP[10.129.229.26], JQuery[3.2.1], PasswordField, Script, Title[Request Baskets]
```

```bash
gobuster dir -u http://sau.htb:55555 -w /usr/share/wordlists/dirb/common.txt
```

We discover only this web dir: `/web (Status: 200)` that unfortunely corrispond to our index page.







Default TCP port 79 is running the finger service.

{% hint style="info" %}
79
{% endhint %}

### 1.2 - How many users can be found by enumerating the finger service? Consider only users who shows a pts?

#### How function finger service?







{% embed url="https://github.com/dev-angelist/Finger-User-Enumeration" %}



{% hint style="info" %}
2
{% endhint %}

### 1.3 - What is the password for the sunny user on Sunday?







{% hint style="info" %}

{% endhint %}

## Task 2 - Find user flag

### 2.1 - &#x20;











{% hint style="info" %}

{% endhint %}

### 2.2 - &#x20;

\
Attacker machine:

```bash
```



{% hint style="info" %}

{% endhint %}

### 2.3 -&#x20;







{% hint style="info" %}

{% endhint %}

### 2.4 -&#x20;





```bash
```

<details>

<summary>🚩 Flag 1 (user.txt)</summary>



</details>

## Task 3 - Find root flag

### 3.1 - What is the full path to the application the user puma can run as root on Sau?

Very good, we can proceed with privilege escalation for obtaining the root flag.

Executing `sudo -l` command we can commands that user puma can execute with sudo privileges

<figure><img src="../.gitbook/assets/image (359).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
/usr/bin/systemctl
{% endhint %}

### 3.2 - What is the full version string for the instance of systemd installed on Sau?

We know that systemctl is a service associated at process systemd, we can search version digiting: `systemctl --version`

<figure><img src="../.gitbook/assets/image (361).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
systemd 245 (245.4-4ubuntu3.22)
{% endhint %}

### 3.3 - What is the CVE ID for a local privilege escalation vulnerability that affects that particular systemd version?

Googling 'usr/bin/systemctl status trail.service', we discover this CVE:

{% embed url="https://nvd.nist.gov/vuln/detail/cve-2023-26604" %}

and this useful resource:&#x20;

{% embed url="https://securityonline.info/cve-2023-26604-systemd-privilege-escalation-flaw-affects-linux-distros/" %}

then, only executing: `sudo /usr/bin/systemctl status trail.service`

and adding `!sh`  we can spawn a new shell, directly with root privileges.

<div align="left"><figure><img src="../.gitbook/assets/image (362).png" alt=""><figcaption></figcaption></figure></div>

{% hint style="info" %}
CVE-2023-26604
{% endhint %}

### 3.4 - Submit the flag located in the root user's home directory.

Let's go into root folder for catching root flag!

![](<../.gitbook/assets/image (363).png>)\


<details>

<summary>🚩 Flag 2 (root.txt)</summary>

c4fc01d4944cf3925f079da70abdaea7

</details>

<figure><img src="../.gitbook/assets/image (364).png" alt=""><figcaption></figcaption></figure>
