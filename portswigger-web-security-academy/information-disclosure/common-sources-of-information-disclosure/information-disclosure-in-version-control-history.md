---
description: >-
  https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-in-version-control-history
icon: flask
---

# Information disclosure in version control history

## Description

This lab discloses sensitive information via its version control history. To solve the lab, obtain the password for the `administrator` user then log in and delete the user `carlos`.

## Solution

<figure><img src="../../../.gitbook/assets/image (52) (1) (1).png" alt=""><figcaption></figcaption></figure>

Adding `.git` we obtain a version control history:

[https://0afe00be03be902083195529004100e5.web-security-academy.net/.git](https://0afe00be03be902083195529004100e5.web-security-academy.net/.git)

<figure><img src="../../../.gitbook/assets/image (53) (1) (1).png" alt=""><figcaption></figcaption></figure>

Deep dive to each dir/file to check if there're some interesting data:

#### HEAD

[https://0afe00be03be902083195529004100e5.web-security-academy.net/.git/HEAD](https://0afe00be03be902083195529004100e5.web-security-academy.net/.git/HEAD)

<figure><img src="../../../.gitbook/assets/image (54) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
ref: refs/heads/master
```

#### CONFIG

[https://0afe00be03be902083195529004100e5.web-security-academy.net/.git/config](https://0afe00be03be902083195529004100e5.web-security-academy.net/.git/config)

<figure><img src="../../../.gitbook/assets/image (55) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
[user]
	email = carlos@carlos-montoya.net
	name = Carlos Montoya
```

#### COMMIT\_EDITMSG

[https://0afe00be03be902083195529004100e5.web-security-academy.net/.git/COMMIT\_EDITMSG](https://0afe00be03be902083195529004100e5.web-security-academy.net/.git/COMMIT_EDITMSG)

<figure><img src="../../../.gitbook/assets/image (56) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### /refs/heads/master

In the 'Head' page there's a potential path, trying to go there: `/refs/heads/master`

there's an alphanumeric string:&#x20;

<figure><img src="../../../.gitbook/assets/image (57) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
ff5435104086dbedd8f46b0a70ffb51cca1b1a44
```

#### /logs/HEAD

and remembering COMMIT\_EDITMSG page and searching into others directories, theres a great info into: `/logs/HEAD`

[https://0afe00be03be902083195529004100e5.web-security-academy.net/.git/logs/HEAD](https://0afe00be03be902083195529004100e5.web-security-academy.net/.git/logs/HEAD)

<figure><img src="../../../.gitbook/assets/image (58) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
0000000000000000000000000000000000000000 b25706d68b5971f903aadce94f299ef2371ee46f Carlos Montoya <carlos@carlos-montoya.net> 1742510797 +0000	commit (initial): Add skeleton admin panel
b25706d68b5971f903aadce94f299ef2371ee46f ff5435104086dbedd8f46b0a70ffb51cca1b1a44 Carlos Montoya <carlos@carlos-montoya.net> 1742510797 +0000	commit: Remove admin password from config
```

#### index

[https://0afe00be03be902083195529004100e5.web-security-academy.net/.git/index](https://0afe00be03be902083195529004100e5.web-security-academy.net/.git/index)

Downloading and opening index file there're an encoded text in hex:

```
DIRC\00\00\00\02\00\00\00\02\67\DC\9A\CD\0D\1F\F9\23\67\DC\9A\CD\0D\1F\F9\23\00\00\00\4B\00\31\42\CF\00\00\81\A4\00\00\2E\E2\00\00\2E\E2\00\00\00\25\21\D2\3F\13\CE\6C\70\4B\81\85\73\79\A3\E2\47\E3\43\6F\4B\26\00\0A\61\64\6D\69\6E\2E\63\6F\6E\66\00
\00\00\00\00\00\00\00\67\DC\9A\CD\01\AE\43\06\67\DC\9A\CD\01\AE\43\06\00\00\00\4B\00\31\42\CE\00\00\81\A4\00\00\2E\E2\00\00\2E\E2\00\00\00\58\89\44\E3\B9\85\36\91\43\1D\C5\8D\5F\49\78\D3\94\0C\EA\4A\F2\00\0F\61\64\6D\69\6E\5F\70\61\6E\65\6C
\2E\70\68\70\00\00\00\54\52\45\45\00\00\00\19\00\32\20\30\0A\21\54\55\59\44\00\27\91\A4\D2\74\12\BF\6E\9A\6F\29\E9\42\FA\0E\DD\0E\69\1B\C0\88\2E\F4\9C\F4\9F\4A\5B\13\19\96\04\1A\B7sa
```

that in clear text contains interesting data:

* `admin.conf`
* `admin_panel.php`

<figure><img src="../../../.gitbook/assets/image (59) (1) (1).png" alt=""><figcaption></figcaption></figure>

Adding those paths, I've not found a solution, so i decided to download the entire git directories and use a dedicated tool.

Download Git Dir: `wget -r https://0afe00be03be902083195529004100e5.web-security-academy.net/.git`

Now we've download all git files locally, so go there to investigate well (`cd ~/Documents/0a2000e40417d00885e2135600ed00cb.web-security-academy.net/.git`), files are hidden by default, so we can see them using the flag -h `(ls -lah`).

It seems the same thing, so we can try to use git commands to check logs: `git log`

<figure><img src="../../../.gitbook/assets/image (458).png" alt=""><figcaption></figcaption></figure>

Great, only now i've undestand that those values were about git commits, so the first one seems more interesting, explore it using git show command:

`git show e06350084adb1d7a44eef13faf0a9cd6cac55bd5`

<figure><img src="../../../.gitbook/assets/image (459).png" alt=""><figcaption></figcaption></figure>

and finally we obtain the admin password value!

Awesome, now we can login us as administrator (`administrator::bohd9ui3rn3yqzgsktq1`)&#x20;

<figure><img src="../../../.gitbook/assets/image (460).png" alt=""><figcaption></figcaption></figure>

go to admin portal page:

[https://0a3f00cb040ae66c819d1b8e00a700ea.web-security-academy.net/admin](https://0a3f00cb040ae66c819d1b8e00a700ea.web-security-academy.net/admin)

<figure><img src="../../../.gitbook/assets/image (461).png" alt=""><figcaption></figcaption></figure>

and delete 'Carlos' user completing the lab.

<figure><img src="../../../.gitbook/assets/image (462).png" alt=""><figcaption></figcaption></figure>
