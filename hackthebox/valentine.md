# Valentine

🔗 [Valentine](https://www.hackthebox.com/machines/Valentine)

### Task 1 - Deploy the machine

🎯 Target IP: `10.129.236.216`

Create a directory on the Desktop with the machine's name, and inside this directory, create another directory to store the materials and outputs needed to run the machine, including the scans made with nmap.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
<strong>echo "10.129.236.216 valentine.htb" >> /etc/hosts
</strong>
mkdir -p htb/valentine.htb
cd htb/valentine.htb
mkdir {nmap,content,exploits,scripts}
# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 valentine.htb
PING valentine.htb (10.129.236.216) 56(84) bytes of data.
64 bytes from valentine.htb (10.129.236.216): icmp_seq=1 ttl=63 time=61.0 ms
64 bytes from valentine.htb (10.129.236.216): icmp_seq=2 ttl=63 time=59.5 ms
64 bytes from valentine.htb (10.129.236.216): icmp_seq=3 ttl=63 time=60.0 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target should be a \*nix system, while Windows systems usually have a TTL of 128 secs.

### 2.1 - How many TCP ports are open on the remote host?

```bash
nmap -p0- -sS -Pn -vvv valentine.htb -oN nmap/tcp_port_scan
```

```bash
PORT    STATE SERVICE REASON
22/tcp  open  ssh     syn-ack ttl 63
80/tcp  open  http    syn-ack ttl 63
443/tcp open  https   syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sS</td><td>SynScan</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 3 open TCP ports on the machine: 22, 80, 443.

{% hint style="info" %}
3
{% endhint %}

### 2.2 - Which flag is used with nmap to execute its vulnerability discovery scripts (with the category "vuln") on the target??

Now, we take more precise scan utilizing -sCV flags to retrieve versioning services and test common scripts.

```bash
nmap -p22,80,443 -sS -Pn -n -v -sCV --script vuln -T4 valentine.htb -oN nmap/port_scan
```

```
PORT    STATE    SERVICE   VERSION
22/tcp  filtered ssh
80/tcp  open     http      Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html).
443/tcp open     ssl/https Apache/2.2.22 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Issuer: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2018-02-06T00:45:25
| Not valid after:  2019-02-06T00:45:25
| MD5:   a413:c4f0:b145:2154:fb54:b2de:c7a9:809d
|_SHA-1: 2303:80da:60e7:bde7:2ba6:76dd:5214:3c3c:6f53:01b1
|_ssl-date: 2024-07-14T09:44:55+00:00; 0s from scanner time.
```

Since we lack credentials for SSH login, we will begin by examining ports 80 and 443.

#### Port 80 and 443

Browsing it we don't find nothing of interesting, we can launch a whatweb and gobuster dir scan enumeration, same thing for each ports

<figure><img src="../.gitbook/assets/image (328).png" alt=""><figcaption></figcaption></figure>

```bash
whatweb valentine.htb
http://valentine.htb [200 OK] Apache[2.2.22], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.2.22 (Ubuntu)], IP[10.129.236.216], PHP[5.3.10-1ubuntu3.26], X-Powered-By[PHP/5.3.10-1ubuntu3.26]
```

```bash
gobuster dir -u http://valentine.htb -w /usr/share/wordlists/dirb/common.txt
```

<div align="left">

<figure><img src="../.gitbook/assets/image (330).png" alt=""><figcaption></figcaption></figure>

</div>

We discover interested directories, I'll explore them later.

In this case, the question doesn't concern the machine, but is a generic question regarding nmap's parameters.

{% hint style="info" %}
\--script vuln
{% endhint %}

### 2.3 - What is the 2014 CVE ID for an information disclosure vulnerability that the service on port 443 is vulnerable to?

These and the previous questions hint at using the `--script vuln` flag for port 443. By googling, we can identify potential vulnerabilities. We then try to find a vulnerability related to the machine's theme (Valentine) and the image on the index page.

There's a dedicated nmap script to test following Heartbleed vulnerability:

```bash
nmap -p443 -Pn --script ssl-heartbleed -T4 valentine.htb -oN nmap/vuln
```

```bash
PORT    STATE SERVICE
443/tcp open  https
| ssl-heartbleed: 
|   VULNERABLE:
|   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL versions 1.0.1 and 1.0.2-beta releases (including 1.0.1f and 1.0.2-beta1) of OpenSSL are affected by the Heartbleed bug. The bug allows for reading memory of systems protected by the vulnerable OpenSSL versions and could allow for disclosure of otherwise encrypted confidential information as well as the encryption keys themselves.
|           
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160
|       http://www.openssl.org/news/secadv_20140407.txt 
|_      http://cvedetails.com/cve/2014-0160/
```

{% embed url="https://heartbleed.com/" %}

{% embed url="https://nvd.nist.gov/vuln/detail/cve-2014-0160" %}

{% hint style="info" %}
CVE-2014-0160
{% endhint %}

### 2.4 -  What password can be leaked using (CVE-2014-0160)?

This github repo contains PoC and relative exploit for this vulnerability:

{% embed url="https://github.com/sensepost/heartbleed-poc" %}

Using the above exploit, it is fairly straightforward to obtain some sensitive information from memory. Running it several times should yield a base64-encoded string.

In this case, we decide to exploit it using metasploit module dedicated for this vulnerability:

<div align="left">

<figure><img src="../.gitbook/assets/image (334).png" alt=""><figcaption></figcaption></figure>

</div>

We use `spool memory_leak.txt` command to save it locally, and run it more times to see various memory output.

<figure><img src="../.gitbook/assets/image (335).png" alt=""><figcaption></figcaption></figure>

Doing it we found this interesting string, in details:&#x20;

```
$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==
```

that is a base64 string.

<figure><img src="../.gitbook/assets/image (336).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
heartbleedbelievethehype
{% endhint %}

### 2.5 - What is the relative path of a folder on the website that contains two interesting files, including note.txt?

Going to 'hidden' web dir discovered using gobuster: /dev we can see that there're two interesting files:

<figure><img src="../.gitbook/assets/image (337).png" alt=""><figcaption></figcaption></figure>

notes.txt

<figure><img src="../.gitbook/assets/image (331).png" alt=""><figcaption></figcaption></figure>

hype\_key

<figure><img src="../.gitbook/assets/image (333).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
/dev
{% endhint %}

### 2.6 -  What is the filename of the RSA key found on the website?



hype\_key

<figure><img src="../.gitbook/assets/image (333).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
hype\_key
{% endhint %}

### Task 3 - Find user flag

### 3.1 - Submit the flag located in the hype user's home directory.

We can convert hype\_key (hex) to ASCII

<figure><img src="../.gitbook/assets/image (338).png" alt=""><figcaption></figcaption></figure>

and we obtain an encrypted RSA private key (maybe as id\_rsa to SSH access), saved into id\_rsa\_psw.

<div align="left">

<figure><img src="../.gitbook/assets/image (339).png" alt=""><figcaption></figcaption></figure>

</div>

We just know an hypotetic password: heartbleedbelievethehype then we can try decode it:

`openssl rsa -in id_rsa_psw`

<div align="left">

<figure><img src="../.gitbook/assets/image (340).png" alt=""><figcaption></figcaption></figure>

</div>





`ssh hype@valentine.htb -i id_rsa`



<details>

<summary>🚩 Flag 1 (user.txt)</summary>

5d03664f2ed9ba6767660926fcaa97b9

</details>

### 3.2 - What is the name of the terminal multiplexing software that the hype user has run previously?





{% hint style="info" %}
tmux
{% endhint %}

### 3.3 - What is the full path to the socket file used by the tmux session?



{% hint style="info" %}
/.devs/dev\_sess
{% endhint %}

### 3.4 - What user is that tmux session running as?



{% hint style="info" %}
root
{% endhint %}

### Task 4 - Find root flag

### 4.1 - Submit the flag located in root's home directory.



<details>

<summary>🚩 Flag 2 (root.txt)</summary>

2ec3181b64936d66aae6e2bc1215a477

</details>
