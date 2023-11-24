# Blaster

<div align="left">

<figure><img src="../.gitbook/assets/image (90).png" alt="" width="188"><figcaption><p><a href="https://tryhackme.com/room/blaster">https://tryhackme.com/room/blaster</a></p></figcaption></figure>

</div>

🔗 [Blaster](https://tryhackme.com/room/blaster)

## Task 1 - Deploy the machine

🎯 Target IP: `10.10.53.21`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

## Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
echo "10.10.53.21 blaster.thm" >> /etc/hosts

mkdir thm/blaster
cd thm/blaster
mkdir {nmap,content,exploits,scripts}

# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 blaster.thm

```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~128 secs. this indicates that the target is Windows, while \*nix systems usually have a TTL of 64 secs.

## Task 3 - Activate Forward Scanners and Launch Proton Torpedoes

### 3.1 How many ports are open on our target system?

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





{% hint style="info" %}

{% endhint %}

### 3.2 - Looks like there's a web server running, what is the title of the page we discover when browsing to it?

\
\


Then we can start to see website (port 80):



We find this info in the footer of website:

>

We continue exploring web source code to find eventual information disclosure.





{% hint style="info" %}

{% endhint %}

### 3.3 - Interesting, let's see if there's anything else on this web server by fuzzing it. What hidden directory do we discover?

\




No other informations, another good thing to do, is find hidden paths on website using gobuster

```bash
gobuster dir -u lian_yu.thm -w /usr/share/wordlists/dirb/common.txt
```

<div align="left">

<figure><img src="../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

</div>



{% hint style="info" %}

{% endhint %}

### 3.4 - Navigate to our discovered hidden directory, what potential username do we discover?

\






We can try to use a bigger wordlist such as directory-list-2.3-medium.txt

```bash
gobuster dir -u lian_yu.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

<div align="left">

<figure><img src="../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

</div>

Excellent! We find a new web path: /island, search it!













{% hint style="info" %}

{% endhint %}

### 3.5 - Crawling through the posts, it seems like our user has had some difficulties logging in recently. What possible password do we discover?

\








{% hint style="info" %}

{% endhint %}

### 3.6 - Log into the machine via Microsoft Remote Desktop (MSRDP) and read user.txt. What are it's contents? 

\


###

###

###



{% hint style="info" %}

{% endhint %}

## Task 4 - Breaching the Control Room

### 4.1 - When enumerating a machine, it's often useful to look at what the user was last doing. Look around the machine and see if you can find the CVE which was researched on this server. What CVE was it?







\


{% hint style="info" %}

{% endhint %}

### 4.2 - Looks like an executable file is necessary for exploitation of this vulnerability and the user didn't really clean up very well after testing it. What is the name of this executable?











{% hint style="info" %}

{% endhint %}

### 4.3 - Now that we've spawned a terminal, let's go ahead and run the command 'whoami'. What is the output of running this?

\






{% hint style="info" %}

{% endhint %}

### 4.4- Now that we've confirmed that we have an elevated prompt, read the contents of root.txt on the Administrator's desktop. What are the contents? Keep your terminal up after exploitation so we can use it in next task!











```bash
find / -type f -iname "user.txt" 2>/dev/null
```

<div align="left">

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

</div>

<details>

<summary>🚩User Flag (user.txt)</summary>

THM{P30P7E\_K33P\_53CRET5\_\_C0MPUT3R5\_D0N'T}

</details>



## Task 5 - Adoption into the Collective

Return to your attacker machine for this next bit. Since we know our victim machine is running Windows Defender, let's go ahead and try a different method of payload delivery! For this, we'll be using the script web delivery exploit within Metasploit. Launch Metasploit now and select 'exploit/multi/script/web\_delivery' for use.

### 5.1 - First, let's set the target to PSH (PowerShell). Which target number is PSH?









{% hint style="info" %}

{% endhint %}

* After setting your payload, set your lhost and lport accordingly such that you know which port the MSF web server is going to run on and that it'll be running on the TryHackMe network.
* Finally, let's set our payload. In this case, we'll be using a simple reverse HTTP payload. Do this now with the command: 'set payload windows/meterpreter/reverse\_http'. Following this, launch the attack as a job with the command 'run -j'.
* Return to the terminal we spawned with our exploit. In this terminal, paste the command output by Metasploit after the job was launched. In this case, I've found it particularly helpful to host a simple python web server (python3 -m http.server) and host the command in a text file as copy and paste between the machines won't always work. Once you've run this command, return to our attacker machine and note that our reverse shell has spawned.

### 5.2 - Last but certainly not least, let's look at persistence mechanisms via Metasploit. What command can we run in our meterpreter console to setup persistence which automatically starts when the system boots? Don't include anything beyond the base command and the option for boot startup.







* Run this command now with options that allow it to connect back to your host machine should the system reboot. Note, you'll need to create a listener via the handler exploit to allow for this remote connection in actual practice. Congrats, you've now gain full control over the remote host and have established persistence for further operations!
