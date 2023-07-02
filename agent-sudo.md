# Agent Sudo

<div align="left">

<figure><img src=".gitbook/assets/aedc6b66c222e15ff740c282a0c3f44e.png" alt="" width="188"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

🔗 [Agent Sudo](https://tryhackme.com/room/agentsudoctf)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.3.152`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

```bash
su
echo "10.10.3.152 agent_sudo.thm" >> /etc/hosts

mkdir thm/agent_sudo.thm  
cd thm/agent_sudo.thm

# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 agent_sudo.thm
PING agent_sudo.thm (10.10.3.152) 56(84) bytes of data.
64 bytes from agent_sudo.thm (10.10.3.152): icmp_seq=1 ttl=63 time=132 ms
64 bytes from agent_sudo.thm (10.10.3.152): icmp_seq=2 ttl=63 time=81.8 ms
64 bytes from agent_sudo.thm (10.10.3.152): icmp_seq=3 ttl=63 time=123 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

### Task 3 - Enumerate

#### 3.1 - How many open ports?

```bash
nmap --open -n -Pn -vvv -T4 agent_sudo.thm 
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-02 14:49 EDT
Warning: Hostname agent_sudo.thm resolves to 2 IPs. Using 10.10.3.152.
Initiating SYN Stealth Scan at 14:49
Scanning agent_sudo.thm (10.10.3.152) [1000 ports]
Discovered open port 80/tcp on 10.10.3.152
Discovered open port 22/tcp on 10.10.3.152
Discovered open port 21/tcp on 10.10.3.152
Completed SYN Stealth Scan at 14:49, 1.15s elapsed (1000 total ports)
Nmap scan report for agent_sudo.thm (10.10.3.152)
Host is up, received user-set (0.078s latency).
Other addresses for agent_sudo.thm (not scanned): 10.10.3.152
Scanned at 2023-07-02 14:49:50 EDT for 1s
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

```bash
nmap -p21,22,80 -sCV -vvv -T4 agent_sudo.thm
```

```bash
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5hdrxDB30IcSGobuBxhwKJ8g+DJcUO5xzoaZP/vJBtWoSf4nWDqaqlJdEF0Vu7Sw7i0R3aHRKGc5mKmjRuhSEtuKKjKdZqzL3xNTI2cItmyKsMgZz+lbMnc3DouIHqlh748nQknD/28+RXREsNtQZtd0VmBZcY1TD0U4XJXPiwleilnsbwWA7pg26cAv9B7CcaqvMgldjSTdkT1QNgrx51g4IFxtMIFGeJDh2oJkfPcX6KDcYo6c9W1l+SCSivAQsJ1dXgA2bLFkG/wPaJaBgCzb8IOZOfxQjnIqBdUNFQPlwshX/nq26BMhNGKMENXJUpvUTshoJ/rFGgZ9Nj31r
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHdSVnnzMMv6VBLmga/Wpb94C9M2nOXyu36FCwzHtLB4S4lGXa2LzB5jqnAQa0ihI6IDtQUimgvooZCLNl6ob68=
|   256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOL3wRjJ5kmGs/hI4aXEwEndh81Pm/fvo8EvcpDHR5nt
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
|_http-title: 400 Bad Request
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```

It looks like there are three open ports on the machine: 21, 22, 80.

#### 3.2 - How you redirect yourself to a secret page?

\






#### 3.3 - What is the agent name?

\


### Task 4 - Hash cracking and brute-force











### Task 5 - Capture the user flag









<details>

<summary>🚩 Flag 1 (flag.txt)</summary>

THM{wh0\_d035nt\_l0ve5\_b0l7\_r1gh7?}

</details>







### Task 6 - Privilege escalation



