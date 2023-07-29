# Bank

<div align="left">

<figure><img src=".gitbook/assets/f02481d8d8020005f8d66115b3bfae11_thumb.png" alt=""><figcaption><p>hackthebox.com - © HACKTHEBOX</p></figcaption></figure>

</div>

🔗 [Bank](https://app.hackthebox.com/machines/Bank)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.10.29`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
echo "10.10.10.29 bank.htb" >> /etc/hosts

mkdir -p htb/bank.htb
cd htb/bank.htb
mkdir {nmap,content,exploits,scripts}
# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 bank.htb
PING bank.htb (10.10.10.29) 56(84) bytes of data.
64 bytes from bank.htb (10.10.10.29): icmp_seq=1 ttl=63 time=56.0 ms
64 bytes from bank.htb (10.10.10.29): icmp_seq=2 ttl=63 time=54.0 ms
64 bytes from bank.htb (10.10.10.29): icmp_seq=3 ttl=63 time=56.3 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system, while Windows systems usually have a TTL of 128 secs.

### 2.1 - How many TCP ports are listening and accessible on Bank?

```bash
nmap --open -p0- -sS -n -Pn -vvv --min-rate 5000 bank.htb -oG nmap/port_scan
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-29 11:50 EDT
Initiating SYN Stealth Scan at 11:50
Scanning bank.htb (10.10.10.29) [65536 ports]
Discovered open port 53/tcp on 10.10.10.29
Discovered open port 80/tcp on 10.10.10.29
Discovered open port 22/tcp on 10.10.10.29
Completed SYN Stealth Scan at 11:50, 13.36s elapsed (65536 total ports)
Nmap scan report for bank.htb (10.10.10.29)
Host is up, received user-set (0.056s latency).
Scanned at 2023-07-29 11:50:07 EDT for 14s
Not shown: 65324 closed tcp ports (reset), 209 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
53/tcp open  domain  syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 3 open TCP ports on the machine: 22, 53, 80.

{% hint style="info" %}
3
{% endhint %}

### 2.2 - What virtual host returns a website that isn't the default Ubuntu Apache page?

Going to http:\\\bank.htb page, we see an hypotetical redirect to http:\\\bank.htb/login.php:

<figure><img src=".gitbook/assets/Schermata del 2023-07-29 19-05-17.png" alt=""><figcaption></figcaption></figure>

We can check and confirm it using BurpSuite:

<figure><img src=".gitbook/assets/Schermata del 2023-07-29 19-04-43.png" alt=""><figcaption></figcaption></figure>







{% hint style="info" %}

{% endhint %}



Now, we try to find potential hidden directory using gobuster:

```bash
gobuster dir -u http://bank.htb/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

```bash
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://bank.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/07/29 12:06:40 Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 305] [--> http://bank.htb/uploads/]
/assets               (Status: 301) [Size: 304] [--> http://bank.htb/assets/]
/inc                  (Status: 301) [Size: 301] [--> http://bank.htb/inc/]
/server-status        (Status: 403) [Size: 288]
/balance-transfer     (Status: 301) [Size: 314] [--> http://bank.htb/balance-transfer/]
Progress: 220518 / 220561 (99.98%)
===============================================================
2023/07/29 12:27:51 Finished
===============================================================
```

and we find an interesting path: http://bank.htb/balance-transfer/











{% hint style="info" %}

{% endhint %}

### 2.3 -&#x20;







We can try to put a file using ftp, in this case we use nmap result file (port\_scan):

```bash
ftp> put port_scan 
```

```bash
ftp> put port_scan 
local: port_scan remote: port_scan
229 Entering Extended Passive Mode (|||49219|)
150 Opening ASCII mode data connection.
100% |***************************************************************************************|   464        8.84 MiB/s    --:-- ETA
226 Transfer complete.
464 bytes sent in 00:00 (7.98 KiB/s)
ftp> ls
229 Entering Extended Passive Mode (|||49220|)
125 Data connection already open; Transfer starting.
07-24-23  11:15AM               241062 40564.exe
03-18-17  02:06AM       <DIR>          aspnet_client
07-24-23  01:26AM                 1442 cmdasp.aspx
07-24-23  12:36AM                 2914 devel.aspx
07-24-23  01:04AM                 2886 devel1.aspx
07-24-23  04:44PM                 2917 devel2.aspx
07-24-23  02:11AM                 2749 develshell.aspx
07-24-23  11:09AM                15966 fox.aspx
07-24-23  09:26AM                 2906 hacked.aspx
03-17-17  05:37PM                  689 iisstart.htm
07-24-23  07:16PM                    0 killbill.aspx
07-24-23  07:21PM                 2912 killbill1.aspx
07-24-23  10:57PM                  464 port_scan
07-24-23  12:17AM                 2783 pwned.aspx
07-24-23  03:00PM                 2923 rev.aspx
07-24-23  09:21PM                15969 shell.aspx
07-24-23  03:34PM                73802 virus.exe
07-24-23  12:34AM               112815 virus2.exe
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
```

{% hint style="info" %}
yes
{% endhint %}

### 2.4 -&#x20;







```bash
```

{% hint style="info" %}

{% endhint %}

### 2.5 -&#x20;

We launch msfconsole:

```bash
msfconsole
```



{% hint style="info" %}

{% endhint %}

### Task 3 -&#x20;

### 3.1 -&#x20;

\


We've not access to babibs' directory, we can try to find "user.txt" flag using while command in C:\ root.

```bash
where /r C:\ user.txt
```

<div align="left">

<figure><img src=".gitbook/assets/Schermata del 2023-07-25 20-45-43.png" alt=""><figcaption></figcaption></figure>

</div>

Starting to root folder (C:\\) we can find quickly flags, using where command in recusive mode (/r):

```
where /r C:\ user.txt
C:\Users\babis\Desktop\user.txt
```

and read user.txt flag using type command (equivalent to cat on \*nix):

```
type C:\Users\babis\Desktop\user.txt
```

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

5d3fc209e1fae6d5df926fe7dc8a16bd

</details>

### Task 4 - Find root flag

After that, we do the same thing for root.txt flag

```bash
where /r C:\ root.txt
C:\Users\Administrator\Desktop\root.txt
```

```bash
type C:\Users\Administrator\Desktop\root.txt
```

<details>

<summary>🚩 Flag 2 (root.txt)</summary>

cb43e154f9c2ca60b68c8150e5162f32

</details>
