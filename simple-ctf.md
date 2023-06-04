# Simple CTF

<figure><img src=".gitbook/assets/f28ade2b51eb7aeeac91002d41f29c47 (1).png" alt=""><figcaption></figcaption></figure>

[Simple CTF](https://tryhackme.com/room/easyctf)

### Task 1.1 - Deploy the machine

🎯 Target IP: `10.10.213.135`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

```bash
su
echo "10.10.213.135 simple_ctf.thm" >> /etc/hosts

mkdir thm/simple_ctf
cd thm/simple_ctf

# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 simple_ctf.thm
```

```bash
PING simple_ctf.thm (10.10.213.135) 56(84) bytes of data.
64 bytes from simple_ctf.thm (10.10.213.135): icmp_seq=1 ttl=63 time=72.8 ms
64 bytes from simple_ctf.thm (10.10.213.135): icmp_seq=2 ttl=63 time=80.6 ms
64 bytes from simple_ctf.thm (10.10.213.135): icmp_seq=3 ttl=63 time=61.8 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

#### 2.1 - How many services are running under port 1000?

```bash
nmap -p- --open -sS -n -Pn simple_ctf.thm -oG open_ports
```

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-04 12:36 CEST
Nmap scan report for simple_ctf.thm (10.10.213.135)
Host is up (0.066s latency).
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
21/tcp   open  ftp
80/tcp   open  http
2222/tcp open  EtherNetIP-1
```

> 2 ports open under port 1000

#### 2.2 - What is running on the higher port?

The higher port is 2222

```bash
2222/tcp open  EtherNetIP-1
```

```bash
nmap -p2222 -sC -sV simple_ctf.thm 
```

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-04 12:43 CEST
Nmap scan report for simple_ctf.thm (10.10.213.135)
Host is up (0.062s latency).

PORT     STATE SERVICE VERSION
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 294269149ecad917988c27723acda923 (RSA)
|   256 9bd165075108006198de95ed3ae3811c (ECDSA)
|_  256 12651b61cf4de575fef4e8d46e102af6 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

> SSH is running on port 2222

#### 2.3 - What's the CVE you're using against the application?  

\


