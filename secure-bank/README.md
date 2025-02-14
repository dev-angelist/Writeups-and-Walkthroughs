---
description: https://owasp.org/www-project-securebank/
---

# Secure Bank

<figure><img src="../.gitbook/assets/image (451).png" alt=""><figcaption><p><a href="https://owasp.org/www-project-securebank/">https://owasp.org/www-project-securebank/</a></p></figcaption></figure>

**SecureBank** is a FinTech application which contains all **OWASP TOP 10** security vulnerabilities along with some other security flaws found in real-world applications.

You can read more about SecureBank and OWASP top 10 vulnerabilities [here](https://ssrd.gitbook.io/securebank/).&#x20;

* [Install & configure Secure Bank](install-and-configure-secure-bank.md)
* aaaaa
* bbbbb
* cccccc
* cccc

***

## [Install & configure](install-and-configure-secure-bank.md)

## Infrastructure

On the image below you can review how the application is built from the infrastructure point of view.&#x20;

<figure><img src="../.gitbook/assets/image (449).png" alt=""><figcaption></figcaption></figure>

***

### Default users:

```
admin@ssrd.io:admin
developer@ssrd.io:test
yoda@ssrd.io:test
tester@ssrd.io:test
```

### Ports

* 80 on this port SecureBank is accessible
* 1080 is maildev server for user registration
* 5000 is hidden API

### CTF-Mode

If you want to run SecureBank in CTF mode we have also prepared this option. It will create CTFd compatible export file.

Run `docker run -d -p 80:80 -p 5000:5000 -p 1080:1080 -e 'AppSettings:Ctf:Enabled=true' -e 'AppSettings:Ctf:Seed=example' -e 'SeedingSettings:Admin=admin@ssrd.io' -e 'SeedingSettings:AdminPassword=admin' ssrd/securebank`
