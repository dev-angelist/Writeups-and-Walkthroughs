# Wgel

<div align="left">

<figure><img src=".gitbook/assets/image (40).png" alt="" width="188"><figcaption></figcaption></figure>

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

<figure><img src=".gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

and see page source for checking information disclosure.

<figure><img src=".gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Very good! Thanks to this message, we know that Jessie is a user/web master.

Another good thing to do, is find hidden paths on website using gobuster

```
gobuster dir -u wgel.thm -w /usr/share/wordlists/dirb/common.txt
```

<figure><img src=".gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

We can explore /sitemap path:

<figure><img src=".gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

We can try to do a new gobuster search start at this point:

```bash
gobuster dir -u wgel.thm/sitemap -w /usr/share/wordlists/dirb/common.txt  
```

<figure><img src=".gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

<div align="left">

<figure><img src=".gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

</div>

we've find and id\_rsa:

<figure><img src=".gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

remembering that we've user and id rsa, first take permission to id\_rsa file and try login:

```bash
chmod 600 id_rsa
ssh -i id_rsa jessie@wgel.thm
```

















I

###

We quickly try to find user.txt flag using find command:\


```bash
find / -type f -iname user.txt 2>/dev/null
```



```bash
ssh lennie@startup.thm
```

```bash
lennie@startup.thm's password: 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-190-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

44 packages can be updated.
30 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

$ ls
Documents  scripts  user.txt
$ cat user.txt
```

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

THM{03ce3d619b80ccbfb3b7fc81e46c0e79}

</details>





### Task 4 - What are the contents of root.txt?

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 12-37-21.png" alt=""><figcaption></figcaption></figure>

Well done! We find root flag:

```bash
ls
cat root.txt
```

<div align="left">

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 12-38-55.png" alt=""><figcaption></figcaption></figure>

</div>

<details>

<summary>🚩 Flag 2 (root.txt)</summary>

THM{f963aaa6a430f210222158ae15c3d76d}

</details>
