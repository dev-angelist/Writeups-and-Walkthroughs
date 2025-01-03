# Active Directory Basics

<div align="left"><figure><img src="../.gitbook/assets/image (7) (1).png" alt="" width="150"><figcaption><p>tryhackme.com - © TryHackMe</p></figcaption></figure></div>

🔗 [Active Directory Basics](https://tryhackme.com/r/room/winadbasics)

### Task 1 - Introduction and deploy

🎯 Target IP: `10.10.130.43`

We start lab and spawn Windows Server machine. After we'll spawn an attacker box machine directly on THM (available in the free version).

<figure><img src="../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

### Task 2 - Windows Domains

<div align="left"><figure><img src="../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure></div>

<figure><img src="../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

### 2.1 - In a Windows domain, credentials are stored in a centralised repository called...

To overcome these limitations, we can use a Windows domain. Simply put, a **Windows domain** is a group of users and computers under the administration of a given business. The main idea behind a domain is to centralise the administration of common components of a Windows computer network in a single repository called **Active Directory (AD)**.&#x20;

{% hint style="info" %}
Active Directory
{% endhint %}

### 2.2 - The server in charge of running the Active Directory services is called...

The server that runs the Active Directory services is known as a **Domain Controller (DC)**.

{% hint style="info" %}
Domain Controller
{% endhint %}

## Task 3  - Active Directory

### 3.1 - Which group normally administrates all computers and resources in a domain?

| Security Group     | Description                                                                                                                                               |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Domain Admins      | Users of this group have administrative privileges over the entire domain. By default, they can administer any computer on the domain, including the DCs. |
| Server Operators   | Users in this group can administer Domain Controllers. They cannot change any administrative group memberships.                                           |
| Backup Operators   | Users in this group are allowed to access any file, ignoring their permissions. They are used to perform backups of data on computers.                    |
| Account Operators  | Users in this group can create or modify other accounts in the domain.                                                                                    |
| Domain Users       | Includes all existing user accounts in the domain.                                                                                                        |
| Domain Computers   | Includes all existing computers in the domain.                                                                                                            |
| Domain Controllers | Includes all existing DCs on the domain.                                                                                                                  |

{% hint style="info" %}
Domain Admin
{% endhint %}

### 3.2 - What would be the name of the machine account associated with a machine named TOM-PC?

Identifying machine accounts is relatively easy. They follow a specific naming scheme. The machine account name is the computer's name followed by a dollar sign. For example, a machine named `DC01` will have a machine account called `DC01$`.

{% hint style="info" %}
TOM-PC$
{% endhint %}

### 3.3 - Suppose our company creates a new department for Quality Assurance. What type of containers should we use to group all Quality Assurance users so that policies can be applied consistently to them?

**Security Groups vs OUs**

You are probably wondering why we have both groups and OUs. While both are used to classify users and computers, their purposes are entirely different:

* **OUs** are handy for applying policies to users and computers, which include specific configurations that pertain to sets of users depending on their particular role in the enterprise. Remember, a user can only be a member of a single OU at a time, as it wouldn't make sense to try to apply two different sets of policies to a single user.
* **Security Groups**, on the other hand, are used to grant permissions over resources. For example, you will use groups if you want to allow some users to access a shared folder or network printer. A user can be a part of many groups, which is needed to grant access to multiple resources.

{% hint style="info" %}
Organizational Unit
{% endhint %}

## Task 4  - Managing Users in AD

**Deleting extra OUs and users**

Your first task as the new domain administrator is to check the existing AD OUs and users, as some recent changes have happened to the business. You have been given the following organisational chart and are expected to make changes to the AD to match it:

<figure><img src="../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

**Delegation**

One of the nice things you can do in AD is to give specific users some control over some OUs. This process is known as **delegation** and allows you to grant users specific privileges to perform advanced tasks on OUs without needing a Domain Administrator to step in.

One of the most common use cases for this is granting `IT support` the privileges to reset other low-privilege users' passwords. According to our organisational chart, Phillip is in charge of IT support, so we'd probably want to delegate the control of resetting passwords over the Sales, Marketing and Management OUs to him.

<figure><img src="../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

Now let's use Phillip's account to try and reset Sophie's password. Here are Phillip's credentials for you to log in via RDP:

<figure><img src="../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

We can use xfreerdp to connect via RDP, on the attacker box of THM:











### 4.1 - What was the flag found on Sophie's desktop?

{% hint style="info" %}

{% endhint %}

### 4.2 - The process of granting privileges to a user over some OU or other AD Object is called...

{% hint style="info" %}

{% endhint %}

## Task 5  - Managing Computers in AD

### 5.1

{% hint style="info" %}

{% endhint %}

### 5.2

{% hint style="info" %}

{% endhint %}

###



## Task 6  - Group Policies

### 6.1 - &#x20;

{% hint style="info" %}

{% endhint %}

### -

{% hint style="info" %}

{% endhint %}

###



## Task 7 - Authentication Methods

### 7.1

{% hint style="info" %}

{% endhint %}

### -

{% hint style="info" %}

{% endhint %}

###



## Task 8 - Trees, Forests and Trusts

### 8.1 - &#x20;

{% hint style="info" %}

{% endhint %}

### -

{% hint style="info" %}

{% endhint %}
