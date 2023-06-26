# Overpass

<div align="left">

<figure><img src=".gitbook/assets/2048656e072dd7caffe455ae2d44b65f (1).png" alt="" width="128"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

🔗 [Overpass](https://tryhackme.com/room/overpass)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.164.129`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

```bash
su
echo "10.10.164.129 overpass.thm" >> /etc/hosts

mkdir thm/overpass.thm  
cd thm/overpass.thm

# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 overpass.thm
PING overpass.thm (10.10.164.129) 56(84) bytes of data.
64 bytes from overpass.thm (10.10.164.129): icmp_seq=1 ttl=63 time=61.7 ms
64 bytes from overpass.thm (10.10.164.129): icmp_seq=2 ttl=63 time=62.6 ms
64 bytes from overpass.thm (10.10.164.129): icmp_seq=3 ttl=63 time=66.9 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

#### 2.1 - Find open ports on the machine

```bash
nmap --open -n -Pn -vvv -T4 overpass.thm
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-26 16:34 EDT
Initiating SYN Stealth Scan at 16:34
Scanning overpass.thm (10.10.164.129) [1000 ports]
Discovered open port 80/tcp on 10.10.164.129
Discovered open port 22/tcp on 10.10.164.129
Completed SYN Stealth Scan at 16:34, 2.30s elapsed (1000 total ports)
Nmap scan report for overpass.thm (10.10.164.129)
Host is up, received user-set (0.066s latency).
Scanned at 2023-06-26 16:34:46 EDT for 2s
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

```bash
nmap --open  -vvv -T4 -sCV overpass.thm 
```

```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 37:96:85:98:d1:00:9c:14:63:d9:b0:34:75:b1:f9:57 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDLYC7Hj7oNzKiSsLVMdxw3VZFyoPeS/qKWID8x9IWY71z3FfPijiU7h9IPC+9C+kkHPiled/u3cVUVHHe7NS68fdN1+LipJxVRJ4o3IgiT8mZ7RPar6wpKVey6kubr8JAvZWLxIH6JNB16t66gjUt3AHVf2kmjn0y8cljJuWRCJRo9xpOjGtUtNJqSjJ8T0vGIxWTV/sWwAOZ0/TYQAqiBESX+GrLkXokkcBXlxj0NV+r5t+Oeu/QdKxh3x99T9VYnbgNPJdHX4YxCvaEwNQBwy46515eBYCE05TKA2rQP8VTZjrZAXh7aE0aICEnp6pow6KQUAZr/6vJtfsX+Amn3
|   256 53:75:fa:c0:65:da:dd:b1:e8:dd:40:b8:f6:82:39:24 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMyyGnzRvzTYZnN1N4EflyLfWvtDU0MN/L+O4GvqKqkwShe5DFEWeIMuzxjhE0AW+LH4uJUVdoC0985Gy3z9zQU=
|   256 1c:4a:da:1f:36:54:6d:a6:c6:17:00:27:2e:67:75:9c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINwiYH+1GSirMK5KY0d3m7Zfgsr/ff1CP6p14fPa7JOR
80/tcp open  http    syn-ack ttl 63 Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Overpass
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 0D4315E5A0B066CEFD5B216C8362564B
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

It looks like there are only two open ports on the machine: SSH and HTTP.

#### Task 3 - Hack the machine and get the flag in user.txt

We can strat to explore http://overpass.thm (port 80)

<figure><img src=".gitbook/assets/Schermata del 2023-06-26 22-50-02.png" alt=""><figcaption><p>http://overpass.thm:80</p></figcaption></figure>

In the page source code we don't found nothing of interisting, the good route is to explore website hidden pathes using gobuster:

```bash
gobuster dir -u overpass.thm -w /usr/share/wordlists/dirb/common.txt  
```

```bash
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://overpass.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/06/26 16:48:20 Starting gobuster in directory enumeration mode
===============================================================
/aboutus              (Status: 301) [Size: 0] [--> aboutus/]
/admin                (Status: 301) [Size: 42] [--> /admin/]
/css                  (Status: 301) [Size: 0] [--> css/]
/downloads            (Status: 301) [Size: 0] [--> downloads/]
/img                  (Status: 301) [Size: 0] [--> img/]
/index.html           (Status: 301) [Size: 0] [--> ./]
Progress: 4590 / 4615 (99.46%)
===============================================================
2023/06/26 16:49:01 Finished
===============================================================

```

We found an administrator login page:

<figure><img src=".gitbook/assets/Schermata del 2023-06-26 22-53-52.png" alt=""><figcaption><p><a href="http://overpass.thm/admin/">http://overpass.thm/admin/</a></p></figcaption></figure>

Looking at the source code of the page, we see that there are three js scripts, let's go look at them!

<figure><img src=".gitbook/assets/Schermata del 2023-06-26 23-56-43.png" alt=""><figcaption></figcaption></figure>

The login.js file contains a “login” function, which says that if the response of the authentication request is not “Incorrect Credentials” i.e. if the authentication was successful, it then sets the SessionToken to “statusOrCookie”:

<figure><img src=".gitbook/assets/Schermata del 2023-06-27 00-01-01.png" alt=""><figcaption><p>login.js</p></figcaption></figure>

Manually creating a SessionToken cookie with a value of “statusOrCookie” in the browser:













```bash
```

<details>

<summary>🚩 Flag 1 (user.txt)</summary>



</details>

#### Task 3 - Escalate your privileges and get the flag in root.txt 

\


```bash
# whoami
root
```

Now, we're root!

```bash
# cd /root/
# ls
root.txt
# cat root.txt
```

<details>

<summary>🚩 Flag 2 (root.txt)</summary>



</details>
