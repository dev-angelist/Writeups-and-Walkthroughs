# Simple CTF

<figure><img src=".gitbook/assets/f28ade2b51eb7aeeac91002d41f29c47 (1).png" alt=""><figcaption></figcaption></figure>

[Simple CTF](https://tryhackme.com/room/easyctf)

### Task 1.1 - Deploy the machine

🎯 Target IP: `10.10.213.135`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

```bash
su
echo "10.10.213.135 simple_ctf.thm" >> /etc/hosts

mkdir thm/simple_ctf
cd thm/simple_ctf

# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 simple_ctf.thm
```

```bash
PING simple_ctf.thm (10.10.213.135) 56(84) bytes of data.
64 bytes from simple_ctf.thm (10.10.213.135): icmp_seq=1 ttl=63 time=72.8 ms
64 bytes from simple_ctf.thm (10.10.213.135): icmp_seq=2 ttl=63 time=80.6 ms
64 bytes from simple_ctf.thm (10.10.213.135): icmp_seq=3 ttl=63 time=61.8 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix system (probably Linux), while Windows systems usually have a TTL of 128 secs.

#### 2.1 - How many services are running under port 1000? 

```bash
nmap -p- --open -sS -n -Pn simple_ctf.thm -oG open_ports
```

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-30 23:28 CEST
N
```

> 2 ports open

