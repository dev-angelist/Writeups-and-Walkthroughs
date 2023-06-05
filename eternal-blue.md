# Eternal Blue

<div align="left">

<figure><img src=".gitbook/assets/7717bbc69c486931e503a74f3192a4d8.gif" alt="" width="130"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

🔗 [Blue](https://tryhackme.com/room/blue)

### Task 1 Reconnaissance - Deploy the machine

### 1.1 - Deploy the machine

🎯 Target IP: `10.10.33.26`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

```bash
su
echo "10.10.33.26 blue.thm" >> /etc/hosts

mkdir thm/blue
cd thm/blue

# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 blue.thm
```

```bash
PING blue.thm (10.10.33.26) 56(84) bytes of data.
64 bytes from blue.thm (10.10.33.26): icmp_seq=1 ttl=127 time=73.9 ms
64 bytes from blue.thm (10.10.33.26): icmp_seq=2 ttl=127 time=70.3 ms
64 bytes from blue.thm (10.10.33.26): icmp_seq=3 ttl=127 time=75.1 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~128 secs. this indicates that the target is a windows system, while \*nix systems usually have a TTL of 64 secs.

#### 1.2 - How many services are running under port 1000?

```bash
nmap -p1-1000 --open -sS -n -Pn blue.thm -oG open_ports
```

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-05 19:35 CEST
Nmap scan report for blue.thm (10.10.33.26)
Host is up (0.069s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

> 3 ports open under port 1000

#### 1.3 - What is this machine vulnerable to? (Answer in the form of: ms??-???, ex: ms08-067)

```bash
nmap -p135,139,445 -sV -sC -vvv -n -Pn blue.thm
```

```bash
PORT    STATE SERVICE      REASON          VERSION
135/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
139/tcp open  netbios-ssn  syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds syn-ack ttl 127 Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h40m00s, deviation: 2h53m12s, median: 0s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 39544/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 58246/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 23738/udp): CLEAN (Timeout)
|   Check 4 (port 12563/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-06-05T12:36:48-05:00
| smb2-security-mode: 
|   210: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-06-05T17:36:48
|_  start_date: 2023-06-05T17:02:02
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 0243a7b722e5 (unknown)
| Names:
|   JON-PC<00>           Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|   JON-PC<20>           Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
| Statistics:
|   0243a7b722e50000000000000000000000
|   0000000000000000000000000000000000
|_  0000000000000000000000000000

```

This is an important info:

```bash
445/tcp open  microsoft-ds syn-ack ttl 127 Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
```

Microsoft Windows 7/8.1/2008 R2/2012 R2/2016 R2 - 'EternalBlue' SMB Remote Code Execution (MS17-010) [https://www.exploit-db.com/exploits/42315](https://www.exploit-db.com/exploits/42315) or we can find it with searchsploit (CLI).

> MS17-010

### Task 2 - Gain Access

#### 2.1 - Find the exploitation code we will run against the machine. What is the full path of the code? (Ex: exploit/........)

\


\
