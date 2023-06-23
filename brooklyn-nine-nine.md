# Brooklyn Nine Nine

<div align="left" data-full-width="false">

<figure><img src=".gitbook/assets/95b2fab20e29a6d22d6191a789dcbe1f.jpeg" alt="" width="158"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure>

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

#### 2.1 - Scan the machine, how many ports are open?&#x20;

```bash
nmap --open brooklyn.thm
```
