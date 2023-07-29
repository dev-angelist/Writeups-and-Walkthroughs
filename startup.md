# Startup

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







{% hint style="info" %}
Microsoft ftpd
{% endhint %}



<figure><img src=".gitbook/assets/Schermata del 2023-07-29 00-41-30.png" alt=""><figcaption></figcaption></figure>

We can find exploit.exe file using where command and run it to escalate privilege!

```bash
where /r C:\ exploit.exe
c:\inetpub\wwwroot>exploit.exe
whoami
nt authority\system
```

<div align="left">

<figure><img src=".gitbook/assets/Schermata del 2023-07-29 00-47-23.png" alt=""><figcaption></figcaption></figure>

</div>

### Task 3 - What are the contents of user.txt?

\










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



</details>

### Task 4 - What are the contents of root.txt?

\


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



</details>
