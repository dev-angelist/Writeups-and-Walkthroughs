# Ice

🔗 [Ice](https://tryhackme.com/room/ice)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.139.241`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Recon

<pre class="language-bash"><code class="lang-bash">su
echo "10.10.139.241 ice.thm" >> /etc/hosts

mkdir thm/ice.thm
cd thm/ice.thm
mkdir {nmap,content,exploits,scripts}

# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 ice.thm
PING ice.thm (10.10.139.241) 56(84) bytes of data.
64 bytes from ice.thm (10.10.139.241): icmp_seq=1 ttl=127 time=63.7 ms
64 bytes from ice.thm (10.10.139.241): icmp_seq=2 ttl=127 time=64.0 ms
64 bytes from ice.thm (10.10.139.241): icmp_seq=3 ttl=127 time=63.8 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~128 secs. this indicates that the target is a Windows system, while \*nix systems usually have a TTL of 64 secs.

### 2.1 - Launch a scan against our target machine, I recommend using a SYN scan set to scan all ports on the machine.&#x20;

```bash
nmap --open -p0- -sS -n -Pn -vvv --min-rate 5000 ice.thm -oG nmap/port_scan
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-30 10:28 EDT
Initiating SYN Stealth Scan at 10:28
Scanning ice.thm (10.10.139.241) [65536 ports]
Discovered open port 445/tcp on 10.10.139.241
Discovered open port 135/tcp on 10.10.139.241
Discovered open port 139/tcp on 10.10.139.241
Discovered open port 3389/tcp on 10.10.139.241
Discovered open port 49154/tcp on 10.10.139.241
Discovered open port 49158/tcp on 10.10.139.241
Discovered open port 49152/tcp on 10.10.139.241
Discovered open port 49153/tcp on 10.10.139.241
Discovered open port 49159/tcp on 10.10.139.241
Discovered open port 8000/tcp on 10.10.139.241
Discovered open port 49160/tcp on 10.10.139.241
Discovered open port 5357/tcp on 10.10.139.241
Completed SYN Stealth Scan at 10:28, 15.41s elapsed (65536 total ports)
Nmap scan report for ice.thm (10.10.139.241)
Host is up, received user-set (0.066s latency).
Scanned at 2023-07-30 10:28:00 EDT for 15s
Not shown: 62056 closed tcp ports (reset), 3468 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       REASON
135/tcp   open  msrpc         syn-ack ttl 127
139/tcp   open  netbios-ssn   syn-ack ttl 127
445/tcp   open  microsoft-ds  syn-ack ttl 127
3389/tcp  open  ms-wbt-server syn-ack ttl 127
5357/tcp  open  wsdapi        syn-ack ttl 127
8000/tcp  open  http-alt      syn-ack ttl 127
49152/tcp open  unknown       syn-ack ttl 127
49153/tcp open  unknown       syn-ack ttl 127
49154/tcp open  unknown       syn-ack ttl 127
49158/tcp open  unknown       syn-ack ttl 127
49159/tcp open  unknown       syn-ack ttl 127
49160/tcp open  unknown       syn-ack ttl 127
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

Now, we need to search which services are running on open ports:

```bash
nmap -p135,139,445,3389,5357,8000,49152,49153,49154,49158,49159,49160 -n -Pn -vvv -sCV --min-rate 5000 ice.thm -oN nmap/open_port
```

```bash
PORT      STATE SERVICE     REASON          VERSION
135/tcp   open  msrpc       syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  Eicrosof�   syn-ack ttl 127 Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  tcpwrapped  syn-ack ttl 127
| rdp-ntlm-info: 
|   Target_Name: DARK-PC
|   NetBIOS_Domain_Name: DARK-PC
|   NetBIOS_Computer_Name: DARK-PC
|   DNS_Domain_Name: Dark-PC
|   DNS_Computer_Name: Dark-PC
|   Product_Version: 6.1.7601
|_  System_Time: 2023-07-30T14:34:07+00:00
| ssl-cert: Subject: commonName=Dark-PC
| Issuer: commonName=Dark-PC
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2023-07-29T13:59:58
| Not valid after:  2024-01-28T13:59:58
| MD5:   9fbe:1d20:5575:8e3d:82c8:2174:cfc5:691f
| SHA-1: 0d47:c268:ab69:de9b:55d7:ae6b:8446:9dd2:1aa3:a243
| -----BEGIN CERTIFICATE-----
| MIIC0jCCAbqgAwIBAgIQP+JwSZ4ShbhNZaZg/fD2kTANBgkqhkiG9w0BAQUFADAS
| MRAwDgYDVQQDEwdEYXJrLVBDMB4XDTIzMDcyOTEzNTk1OFoXDTI0MDEyODEzNTk1
| OFowEjEQMA4GA1UEAxMHRGFyay1QQzCCASIwDQYJKoZIhvcNAQEBBQADggEPADCC
| AQoCggEBAN9U/vuy6wyZ/pJKOut4KqAdo3j3DIKorY2AAnMewCvUJufZmM6cBuRd
| 4HD8TRmAsN9tIgft6C3L8rCiMUuIMxVGqf562DSYnx7eFidoiCw+XshrjzKDUjTW
| JO431V58yn/6rXOrrCayJKHKKqzlG3BjOYLc+2MV0JwG8lmUAaHmHTVOVxo/8JhQ
| KyknchVese9a4czr9BL4PI3frBtKTuoSD335k+aElMxgJz49trOsgFTRf6kRqA2p
| cK6YH6E8ty5WxpIojrUaZ6MyePYBl5lN+i7RMx4O5iWcbs0F/nbt/I+EbqnVoz6h
| Lsa2cjkIYPpo8XPVdqGGeVLY6e+e4XsCAwEAAaMkMCIwEwYDVR0lBAwwCgYIKwYB
| BQUHAwEwCwYDVR0PBAQDAgQwMA0GCSqGSIb3DQEBBQUAA4IBAQB+EakCXBt4Ru7J
| 1mNrr1aRVYBoBTN/J0RONVOVkhiAcMSm8vgO/I+hlnjbJaJA6QyiQTWr33F9+1tS
| gCkpJWNncQdfosEWvIreF75U5+rND7rp/CnGBqboF0GW82r8b05l2hWBTGHxl0uc
| THF2W++5WGsFEOjziHMkilHiR/MEnWdiPzfXEQtu66s829WmcU/HcK1hIF4g/1Bx
| k8D3FRv55vJtaqXAilNcLNP63FzT45DC8K/hp+8VaUPvN4zz474sEb/rvGqwZSZS
| YFabVfHtoIUxkzjsxrXlONyJWhQgIMQCREOmI6Oe3MDtRk5Wv+i9UknIqxQjw4NK
| py8rnRih
|_-----END CERTIFICATE-----
|_ssl-date: 2023-07-30T14:34:22+00:00; 0s from scanner time.
5357/tcp  open  http        syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Service Unavailable
|_http-server-header: Microsoft-HTTPAPI/2.0
8000/tcp  open  http        syn-ack ttl 127 Icecast streaming media server
| http-methods: 
|_  Supported Methods: GET
|_http-title: Site doesn't have a title (text/html).
49152/tcp open  msrpc       syn-ack ttl 127 Microsoft Windows RPC
49153/tcp open  msrpc       syn-ack ttl 127 Microsoft Windows RPC
49154/tcp open  msrpc       syn-ack ttl 127 Microsoft Windows RPC
49158/tcp open  msrpc       syn-ack ttl 127 Microsoft Windows RPC
49159/tcp open  msrpc       syn-ack ttl 127 Microsoft Windows RPC
49160/tcp open  tcpwrapped  syn-ack ttl 127
Service Info: Host: DARK-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 59m59s, deviation: 2h14m09s, median: 0s
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Dark-PC
|   NetBIOS computer name: DARK-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-07-30T09:34:07-05:00
| smb2-time: 
|   date: 2023-07-30T14:34:08
|_  start_date: 2023-07-30T13:59:57
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| nbstat: NetBIOS name: DARK-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:94:10:4a:84:79 (unknown)
| Names:
|   DARK-PC<00>          Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|   DARK-PC<20>          Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
| Statistics:
|   02:94:10:4a:84:79:00:00:00:00:00:00:00:00:00:00:00
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_  00:00:00:00:00:00:00:00:00:00:00:00:00:00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 26968/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 64093/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 26942/udp): CLEAN (Timeout)
|   Check 4 (port 58078/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
```

### 2.2 - Once the scan completes, we'll see a number of interesting ports open on this machine. As you might have guessed, the firewall has been disabled (with the service completely shutdown), leaving very little to protect this machine. One of the more interesting ports that is open is Microsoft Remote Desktop (MSRDP). What port is this open on?

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 16-47-13.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
3389
{% endhint %}

### 2.3 - What service did nmap identify as running on port 8000? (First word of this service)

<figure><img src=".gitbook/assets/Schermata del 2023-07-30 16-49-25.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Icecast
{% endhint %}

### 2.4 - What does Nmap identify as the hostname of the machine? (All caps for the answer)

\












{% hint style="info" %}

{% endhint %}

## Task 3 - Gain Access

<div align="left">

<figure><img src=".gitbook/assets/image.png" alt="" width="106"><figcaption></figcaption></figure>

</div>

### 3.1 - Now that we've identified some interesting services running on our target machine, let's do a little bit of research into one of the weirder services identified: Icecast. Icecast, or well at least this version running on our target, is heavily flawed and has a high level vulnerability with a score of 7.5 (7.4 depending on where you view it). What type of vulnerability is it? Use https://www.cvedetails.com for this question and the next.















{% hint style="info" %}

{% endhint %}

### 3.2 - What is the CVE number for this vulnerability? This will be in the format: CVE-0000-0000

\














{% hint style="info" %}

{% endhint %}

### 3.3 - After Metasploit has started, let's search for our target exploit using the command 'search icecast'. What is the full path (starting with exploit) for the exploitation module? This module is also referenced in '[RP: Metasploit](https://tryhackme.com/room/rpmetasploit)' which is recommended to be completed prior to this room, although not entirely necessary.&#x20;













{% hint style="info" %}

{% endhint %}

### 3.4 - Following selecting our module, we now have to check what options we have to set. Run the command `show options`. What is the only required setting which currently is blank?

















{% hint style="info" %}

{% endhint %}

## Task 4 - Escalate













































## Task 5 - Looting















## Task 6 - Post-Exploitation



















<details>

<summary>🚩 Flag 1 (user.txt)</summary>

THM{03ce3d619b80ccbfb3b7fc81e46c0e79}

</details>

















<details>

<summary>🚩 Flag 2 (root.txt)</summary>

THM{f963aaa6a430f210222158ae15c3d76d}

</details>
