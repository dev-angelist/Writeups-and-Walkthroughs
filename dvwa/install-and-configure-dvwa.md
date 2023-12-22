---
description: https://github.com/digininja/DVWA
---

# Install and configure DVWA

## DAMN VULNERABLE WEB APPLICATION

Damn Vulnerable Web Application (DVWA) is a PHP/MySQL web application that is damn vulnerable. Its main goal is to be an aid for security professionals to test their skills and tools in a legal environment, help web developers better understand the processes of securing web applications and to aid both students & teachers to learn about web application security in a controlled class room environment.

The aim of DVWA is to **practice some of the most common web vulnerabilities**, with **various levels of difficulty**, with a simple straightforward interface. Please note, there are **both documented and undocumented vulnerabilities** with this software. This is intentional. You are encouraged to try and discover as many issues as possible.

### Download

While there are various versions of DVWA around, the only supported version is the latest source from the official GitHub repository. You can either clone it from the repo:

```bash
git clone https://github.com/digininja/DVWA.git
```

Or [download a ZIP of the files](https://github.com/digininja/DVWA/archive/master.zip).

### Installation and configuration

#### Installation Videos

* [Installing DVWA on Kali running in VirtualBox](https://www.youtube.com/watch?v=WkyDxNJkgQ4) 🇬🇧
* [Installing DVWA on Windows using XAMPP](https://youtu.be/Yzksa\_WjnY0) 🇬🇧🇬
* [Installing Damn Vulnerable Web Application (DVWA) on Windows 10](https://www.youtube.com/watch?v=cak2lQvBRAo) 🇬🇧
* [DVWA 01 - Installazione e Configurazione](https://www.youtube.com/watch?v=F7lX6x87gJg\&list=PLYLjKimBhcxE0u-SIQw0vwt0VM17II9M9) 🇮🇹

#### How to set difficulty

Go to DVWA Security page: [http://localhost/DVWA/security.php](http://localhost/DVWA/security.php) (URL can be changed), and set security level desired, then press submit button.

<figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
In all exercise you need to activate a proxy (between user's browser and the target app), to intercept, inspect and modify requests and responses. In my case i used **BurpSuite** with **FoxyProxy** extension.
{% endhint %}

{% embed url="https://github.com/digininja/DVWA" %}
Official website
{% endembed %}

### References

For the making of this solution the following resource were used:

* [https://github.com/digininja/DVWA](https://github.com/digininja/DVWA)
* [https://github.com/LeonardoE95/DVWA/](https://github.com/LeonardoE95/DVWA/tree/main/src/client\_side\_request\_forgery)
