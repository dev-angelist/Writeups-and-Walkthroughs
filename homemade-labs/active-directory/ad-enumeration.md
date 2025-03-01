---
description: https://dev-angelist.gitbook.io/home/active-directory/ad-enumeration
icon: input-numeric
---

# AD Enumeration

To practice I created a local lab thanks to the following [guide](https://dev-angelist.gitbook.io/building-a-vulnerable-active-directory-lab), then I run the enumeration of a Domain Controller (an unrealistic hypothesis because it is rarely directly exposed to the network).

In addition to what is indicated in the guide, i've added DNS and Web Server (IIS) services.

The target is a DC running Windows Server 2019, while the attacking machine is a Kali Linux machine (both machines are into a custom NAT\_Network called: NAT\_AD `192.168.57.0/24`).

[AD Lab Setup](https://dev-angelist.gitbook.io/building-a-vulnerable-active-directory-lab/)

[AD Enumeration Methodology](https://dev-angelist.gitbook.io/home/active-directory/ad-enumeration)

## **Host Identification**

```bash
sudo nmap -sn 192.168.57.0/24 #Host Discovery
```

<figure><img src="https://dev-angelist.gitbook.io/~gitbook/image?url=https%3A%2F%2F3004761193-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSw7dppa9JTXteSCpJkUd%252Fuploads%252F24ZzBlcMnZk75pRwmVff%252Fimage.png%3Falt%3Dmedia%26token%3D1df5a69f-a102-4b16-b76e-6e532992e0f5&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=48b2fee5&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Save it into /etc/hosts file: `sudo echo "192.168.57.9 corp-dc" >> /etc/hosts` (optional)

## **Open Ports Discovery**

```bash
nmap -p0- -sCV -Pn  corp-dc -oN open_ports
```

```bash
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-02-21 18:19:55Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: dev-angelist.lab0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: dev-angelist.lab0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Service Unavailable
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49670/tcp open  msrpc         Microsoft Windows RPC
49672/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49682/tcp open  msrpc         Microsoft Windows RPC
49687/tcp open  msrpc         Microsoft Windows RPC
MAC Address: 08:00:27:C0:12:91 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_nbstat: NetBIOS name: CORP-DC, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:c0:12:91 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
|_clock-skew: -1s
| smb2-time: 
|   date: 2025-02-21T18:20:44
|_  start_date: N/A
```

I will focus on potentially active and vulnerable services, in this case for example I have not configured the DNS so I will skip it.

### **80 - HTTP**

We can use whatweb command to retrieve info regarding web server, a GET request using curl or visiting page via browser.

```bash
whatweb http://corp-dc
curl -v http://corp-dc          
```

<figure><img src="https://dev-angelist.gitbook.io/~gitbook/image?url=https%3A%2F%2F3004761193-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSw7dppa9JTXteSCpJkUd%252Fuploads%252FRI7okEUWZnU1TY9y43gE%252Fimage.png%3Falt%3Dmedia%26token%3Dce2a7d91-24ff-41d0-8195-07fe34151d29&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=637e2b1a&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Kerberos protocol is a master topic of AD, but in this case i'm doing enumeration phase.

**135 - Microsoft Remote Procedure Call (msrpc)**

This protocol allows application to communicate with other machine into network.

We can user RPC Client for login and enumerate domain users

```bash
rpcclient -U devan corp-dc     #access via recclient (devan::P@ssword123!)
enumdomusers     #enumerate domain users
```

### **139 - NetBios**

Protocol that facilitate communication for file and printer sharing into networks, it is the predecessor of SMB.

```bash
nbtscan 192.168.57.9
```

<figure><img src="https://dev-angelist.gitbook.io/~gitbook/image?url=https%3A%2F%2F3004761193-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSw7dppa9JTXteSCpJkUd%252Fuploads%252Fr7ki6jZKGhwW6PAToPK4%252Fimage.png%3Falt%3Dmedia%26token%3D20b2f3f9-a6fd-4331-b987-f46e2f4e7938&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=37bb4007&#x26;sv=2" alt=""><figcaption></figcaption></figure>

in this case we obtain NetBIOS Name, eventually server, user and MAC address info.

### **389 - LDAP**

Lightweight directory access protocol (LDAP) is a protocol that makes it possible for applications to query user information rapidly. We can perform enumerion using various tools:

#### **`LdapSearch`**

Perform anonymous or credentialed enumeration of the LDAP directory:

```bash
ldapsearch -H ldap://192.168.1.1 -x -s base namingcontexts
ldapsearch -H ldap://CORP-DC -D "devan@dev-angelist.lab" -w "P@ssword123!" -b "DC=dev-angelist,DC=lab" "(objectClass=user)"
#Other additional useful queries:
$Filter = "(objectClass=user)"
$RootOU = "DC=dev-angelist,DC=lab"
$Searcher = New-Object DirectoryServices.DirectorySearcher
$Searcher.SearchRoot = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$($RootOU)")
$Searcher.Filter = $Filter
$Searcher.SearchScope = "Subtree"
$Searcher.FindAll()
```

<figure><img src="https://dev-angelist.gitbook.io/~gitbook/image?url=https%3A%2F%2F3004761193-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSw7dppa9JTXteSCpJkUd%252Fuploads%252FOwAqEE3Ei6E2Zdfzg8Ok%252Fimage.png%3Falt%3Dmedia%26token%3Dee900768-304b-47dd-b2ec-e494e0e3c0b4&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=f7991fc9&#x26;sv=2" alt=""><figcaption></figcaption></figure>

<figure><img src="https://dev-angelist.gitbook.io/~gitbook/image?url=https%3A%2F%2F3004761193-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSw7dppa9JTXteSCpJkUd%252Fuploads%252FzYZgC4sOej3ecAg8ArTl%252Fimage.png%3Falt%3Dmedia%26token%3Df58a9782-19cb-4d28-b0c4-3a81ef2d5b5e&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=50c1584b&#x26;sv=2" alt=""><figcaption></figcaption></figure>

#### **`LdapWhoami`**

Obtain user via Ldapwhoami

```bash
ldapwhoami -H ldap://CORP-DC -D "CN=devan,CN=Users,DC=dev-angelist,DC=lab" -w "P@ssword123!"
```

<figure><img src="https://dev-angelist.gitbook.io/~gitbook/image?url=https%3A%2F%2F3004761193-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSw7dppa9JTXteSCpJkUd%252Fuploads%252Fkd4zhe6yqTBaKT1zxQej%252Fimage.png%3Falt%3Dmedia%26token%3Da105a792-8fb6-4721-9e8d-16e0b17ea892&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=cfcc0ecc&#x26;sv=2" alt=""><figcaption></figcaption></figure>

#### **`LdapDomainDump`**

Dump LDAP data in JSON and HTML formats for easier analysis:

```bash
ldapdomaindump -u 'dev-angelist\devan' -p 'P@ssword123!' 192.168.57.9
```

### **445 - SMB**

The SMB protocol is a network file sharing protocol that allows applications on a computer to read and write to files. SMB also requests services from server programs in a computer network. It's the most critical attack vector if it's not protected well.

The v1 is deprecated and have several vulnerabilities (Eternal Blue, WannaCry, etc).

It can run over multiple ports: 445, 137-139 (NetBIOS), and over UDP.

To test it, i've created a share folder called "SharedFiles" with a text file. This directory is shared with 'devan' with read/write rights (Properties->Share->AddUser: 'devan')

<figure><img src="https://dev-angelist.gitbook.io/~gitbook/image?url=https%3A%2F%2F3004761193-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSw7dppa9JTXteSCpJkUd%252Fuploads%252Ffep32LbK18LCheon7brY%252Fimage.png%3Falt%3Dmedia%26token%3D5bf21382-a065-4d32-bca5-a1b5e05e69a1&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=11248981&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Location: `\\CORP-DC\SharedFiles`

Then, we can access to it on Devan's workstation machine using that location

<figure><img src="https://dev-angelist.gitbook.io/~gitbook/image?url=https%3A%2F%2F3004761193-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSw7dppa9JTXteSCpJkUd%252Fuploads%252FJpet7xfpx7cc77YUFzNL%252Fimage.png%3Falt%3Dmedia%26token%3D8f133faa-74c1-4eba-8aad-540d36ed2dc7&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=6e4247fa&#x26;sv=2" alt=""><figcaption></figcaption></figure>

We can enumerate SMB shares and access to system using these tools:

```bash
smbmap -H corp-dc
smbclient //corp-dc/SharedFiles -U "dev-angelist.lab/devan%P@ssword123!"
```

<figure><img src="https://dev-angelist.gitbook.io/~gitbook/image?url=https%3A%2F%2F3004761193-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSw7dppa9JTXteSCpJkUd%252Fuploads%252FqXZEZO3AvOmfbUaAYtMY%252Fimage.png%3Falt%3Dmedia%26token%3Dbfc5ab24-769c-48a9-852a-9115c7406c89&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=9fa09c5&#x26;sv=2" alt=""><figcaption></figcaption></figure>

***

### &#x20; <a href="#references" id="references"></a>
