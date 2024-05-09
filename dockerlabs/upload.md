# Upload

{% embed url="https://dockerlabs.es/#/" %}

[🔗 DockerLabs.es](https://dockerlabs.es/#/)

&#x20;[🔗](https://dockerlabs.es/#/) [How to setup a Docker Image ](https://dockerlabs.es/assets/instrucciones\_de\_uso.pdf)

<details>

<summary>CTF BootToRoot</summary>

A **CTF** (Capture The Flag) **Boot To Root** (B2R) is a type of cybersecurity challenge where participants are tasked with gaining unauthorized access to a computer system (the "Boot" part) and then obtaining and eventually capturing a specific flag or set of flags (the "Root" part). The flags could be strings of text, files, or other data that prove the participant has successfully compromised the system.

In these challenges, participants typically start with minimal information about the target system and have to use various techniques, including vulnerability analysis, exploitation, privilege escalation, and more, to gain access and ultimately root access to the system. The challenges often simulate real-world scenarios and are designed to test participants' skills in penetration testing, exploit development, reverse engineering, and other cybersecurity domains. They can be hosted online or in-person as part of cybersecurity competitions, training events, or educational exercises.

</details>

## Task 1 - Deploy the machine

Install docker.io (if you do not already have it installed) -> `sudo apt install docker.io`

Download machine from DockerLabs website and setup lab:

* `unzip trust.zip`
* `sudo bash auto_deploy.sh upload.tar`

and we obtained IP machine -> 🎯 Target IP: `172.17.0.2`

We can put the IP in the file to associate it with an easier to remember name:

<pre class="language-bash"><code class="lang-bash">su
<strong>echo "172.17.0.2 upload" >> /etc/hosts
</strong></code></pre>

Create a directory for machine on a dedicated folder and subdir containing: nmap,content,exploits,scripts

<pre class="language-bash"><code class="lang-bash"><strong>mkdir -p DockerLabs/trust
</strong>cd DockerLabs/trust
<strong>mkdir {nmap,content,exploits,scripts}
</strong><strong>
</strong># At the end of the Lab
CTRL + C #To stop running machine

<strong>sed -i '$ d' /etc/hosts # To clean up the last line from the /etc/hosts file
</strong><strong>
</strong># To repair potential problem during activities
sudo systemctl restart docker
sudo docker stop $(docker ps -q)
sudo docker container prune -force
</code></pre>

## Task 2 - Reconnaissance and Exploitation

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 upload
PING upload (172.17.0.2) 56(84) bytes of data.
64 bytes from upload (172.17.0.2): icmp_seq=1 ttl=64 time=0.068 ms
64 bytes from upload (172.17.0.2): icmp_seq=2 ttl=64 time=0.080 ms
64 bytes from upload (172.17.0.2): icmp_seq=3 ttl=64 time=0.066 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system, while Windows systems usually have a TTL of 128 secs.

### 2.1 Ports in listening and relative services

Of course, we start looking for information about our target by scanning the open ports with the nmap tool

```bash
nmap -Pn -n -p0- upload
```

```bash
PORT   STATE SERVICE
80/tcp open  http
```

There's only one open port (80), analyze it searching more info about service version and potential vulns:

```bash
nmap -A -sVC -p 80 upload -oN nmap/port_scan
```

```bash
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Upload here your file
|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), Linux 4.15 - 5.8 (96%), Linux 2.6.32 - 3.10 (96%), Linux 5.0 - 5.5 (96%), Linux 3.4 - 3.10 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (95%), Synology DiskStation Manager 5.2-5644 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>Pn</td><td>no ping</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

Focusing on port 80 we have a web server so by running the whatweb command we extract more information and then display the content via the browser

```bash
whatweb http://upload
```

<figure><img src="../.gitbook/assets/image (251).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (252).png" alt=""><figcaption></figcaption></figure>

Let's display the default Apache page, try analysing the source page with CTRL+U

<div align="left">

<figure><img src="../.gitbook/assets/image (265).png" alt=""><figcaption></figcaption></figure>

</div>

But we don't discover nothing of interisting.

### 2.2 Brute force hidden web directory

Now, we try to find potential hidden directory using gobuster:

```bash
gobuster dir -u http://upload -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

<figure><img src="../.gitbook/assets/image (254).png" alt=""><figcaption></figcaption></figure>

We find a 301 status code (redirect), that contains file uploaded using form of index page.

<figure><img src="../.gitbook/assets/image (256).png" alt=""><figcaption></figcaption></figure>

### 2.3 Upload shell

Trying to upload php file I notice that there is no trace of sanitization, so we have a clear path and we can use a backdoor or a reverse shell in php. In my case I opt for a reverse shell, specifically I use the one present on Kali usually at the path: /usr/share/webshells/php/php-reverse-shell.php

I copy it, assign it execution permissions and modify it by inserting my local IP (kali) and port where we will listen with netcat

```bash
cp /usr/share/webshells/php/php-reverse-shell.php .
chmod +x php-reverse-shell.php
code php-reverse-shell.php
```

<div align="left">

<figure><img src="../.gitbook/assets/image (260).png" alt=""><figcaption></figcaption></figure>

</div>

<div align="left">

<figure><img src="../.gitbook/assets/image (261).png" alt=""><figcaption></figcaption></figure>

</div>

Well, now we have the exploit ready, we load it using the form in the index page

<figure><img src="../.gitbook/assets/image (262).png" alt=""><figcaption></figcaption></figure>

We listen with netcat on the set port '1234': `nc -nvlp 1234` and click on the php-reverse-shell.php file to establish a connection on our machine

<div align="left">

<figure><img src="../.gitbook/assets/image (263).png" alt=""><figcaption></figcaption></figure>

</div>

<figure><img src="../.gitbook/assets/image (264).png" alt=""><figcaption></figcaption></figure>

## Task 3 - Privilege Escalation

Now that we are inside, since we are not root user we need to elevate our privileges, so let's check the current permissions using `sudo -l`

<figure><img src="../.gitbook/assets/image (259).png" alt=""><figcaption></figcaption></figure>

We see that mario user has root permissions for /usr/bin/env, then we can use [gtfobins](https://gtfobins.github.io/gtfobins/env/) to find it.

{% embed url="https://gtfobins.github.io/gtfobins/env/" %}

<figure><img src="../.gitbook/assets/image (257).png" alt=""><figcaption></figcaption></figure>

using vim sudo command, we obtain a root permission `sudo env /bin/sh`

<div align="left">

<figure><img src="../.gitbook/assets/image (258).png" alt=""><figcaption></figcaption></figure>

</div>
