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

#### 3.2 -&#x20;

#### 3.2 -&#x20;

\
