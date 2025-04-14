---
description: >-
  https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/authentication-apprentice/authentication/password-based/lab-username-enumeration-via-different-responses
icon: vial-virus
---

# Username enumeration via different responses

## Description

This lab is vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:

* [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)
* [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.

## Solution

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

We've a usernames and passwords lists, the idea is first to enumerate users and after take a password brute-force.

### User Enumeration

<details>

<summary>Authentication lab usernames</summary>

carlos\
root\
admin\
test\
guest\
info\
adm\
mysql\
user\
administrator\
oracle\
ftp\
pi\
puppet\
ansible\
ec2-user\
vagrant\
azureuser\
academico\
acceso\
access\
accounting\
accounts\
acid\
activestat\
ad\
adam\
adkit\
admin\
administracion\
administrador\
administrator\
administrators\
admins\
ads\
adserver\
adsl\
ae\
af\
affiliate\
affiliates\
afiliados\
ag\
agenda\
agent\
ai\
aix\
ajax\
ak\
akamai\
al\
alabama\
alaska\
albuquerque\
alerts\
alpha\
alterwind\
am\
amarillo\
americas\
an\
anaheim\
analyzer\
announce\
announcements\
antivirus\
ao\
ap\
apache\
apollo\
app\
app01\
app1\
apple\
application\
applications\
apps\
appserver\
aq\
ar\
archie\
arcsight\
argentina\
arizona\
arkansas\
arlington\
as\
as400\
asia\
asterix\
at\
athena\
atlanta\
atlas\
att\
au\
auction\
austin\
auth\
auto\
autodiscover

</details>

Go to login, insert random data and capture the request:

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

Send the request to Burp Intruder and select as parameter the username value "admin", copy and paste userlist to Payload section and start attack:

<figure><img src="../../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

Ordering length value, we can see that only one has a different value, and there's not the error message: "Invalid username" but "Incorrect password"&#x20;

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

so we know that "argentina" user is a valid username.

### Password Brute-Force

Now, is possible to proceed with password brute-force, go back to intruder, click on clear, modify the username value inserting 'argentina' and select password field adding passwordlist values

<details>

<summary>Authentication lab passwords</summary>

123456\
password\
12345678\
qwerty\
123456789\
12345\
1234\
111111\
1234567\
dragon\
123123\
baseball\
abc123\
football\
monkey\
letmein\
shadow\
master\
666666\
qwertyuiop\
123321\
mustang\
1234567890\
michael\
654321\
superman\
1qaz2wsx\
7777777\
121212\
000000\
qazwsx\
123qwe\
killer\
trustno1\
jordan\
jennifer\
zxcvbnm\
asdfgh\
hunter\
buster\
soccer\
harley\
batman\
andrew\
tigger\
sunshine\
iloveyou\
2000\
charlie\
robert\
thomas\
hockey\
ranger\
daniel\
starwars\
klaster\
112233\
george\
computer\
michelle\
jessica\
pepper\
1111\
zxcvbn\
555555\
11111111\
131313\
freedom\
777777\
pass\
maggie\
159753\
aaaaaa\
ginger\
princess\
joshua\
cheese\
amanda\
summer\
love\
ashley\
nicole\
chelsea\
biteme\
matthew\
access\
yankees\
987654321\
dallas\
austin\
thunder\
taylor\
matrix\
mobilemail\
mom\
monitor\
monitoring\
montana\
moon\
moscow

</details>

<figure><img src="../../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

Ordering the different length values we can quicky discover the argentina's password:

<figure><img src="../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Now we can go to login page and access using the following credentials: argentina::computer solving the lab.

<figure><img src="../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>
