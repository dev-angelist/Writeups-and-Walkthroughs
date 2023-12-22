---
description: http://localhost/DVWA/vulnerabilities/exec/
---

# Command Injection

<details>

<summary>What is a command injection?</summary>

Command injection is a type of security vulnerability that occurs when an attacker is able to inject and execute arbitrary commands or code into a software application. This vulnerability is typically found in applications that accept and process input from users or other untrusted sources without proper validation or sanitization.

The most common form of command injection involves the exploitation of input fields or parameters that are used to construct system commands. When the application fails to properly validate or sanitize user input, an attacker can inject malicious commands that get executed by the underlying system with the privileges of the application.

</details>

{% hint style="info" %}
Using BurpSuite and the FoxyProxy extension is recommended.
{% endhint %}

## Low

We've a form with an input type text:

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

that ask us to enter and IP address to ping.

Inserting an IP address we can confirm that will do a ping request to it:

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

As always we can start to analyze source code:

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* There'is a condition to check if input value has been inserted
* The operating system in use is checked to evaluate exactly which ping should be entered (win or \*nix OS)
* In the end, there's generate feedback for the end user.

{% hint style="warning" %}
The input is not sanitized, so I can execute any (potentially malicious) command.
{% endhint %}

#### Payload

```bash
127.0.0.1 ; whoami ; cat /etc/passwd
```

using it, we ping machine with IP 127.0.0.1 and join two extra commands using ; or another join char as |, to take a whoami and see /etc/passwd file:

<figure><img src="../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Medium



<figure><img src="../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

* There's a classic control of input text submitted
* An eventually blacklist word (&& and ;) is replace with a '' black char
* The operating system in use is checked to evaluate exactly which ping should be entered (win or \*nix OS)
* In the end, there's generate feedback for the end user.

{% hint style="warning" %}
The input is not sanitized because blacklist words are eventually removed only one time and not recursevely and we can use others join chars to add a new commands such as |.
{% endhint %}

#### Payload

We've replaced `;` or `&&` with `|`:

```bash
127.0.0.1 | whoami
```

<figure><img src="../.gitbook/assets/image (7) (1).png" alt=""><figcaption><p>Using another word to anchor commands</p></figcaption></figure>

`;` will be replace by '' and will submit this prohibit payload:

```bash
127.0.0.1 && whoami
```

<figure><img src="../.gitbook/assets/image (8) (1).png" alt=""><figcaption><p>Using phoibit anchor commands</p></figcaption></figure>

## High



<figure><img src="../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>



This command isn't write correctly, it has an extra space '`|`  '&#x20;

<div align="left">

<figure><img src="../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

</div>





{% hint style="warning" %}
The input is not sanitized, so I can execute any (potentially malicious) command.
{% endhint %}

Payload

Without leaving a white space after | we can use this payload:

```bash
127.0.0.1 |whoami
```

<figure><img src="../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

## Impossible



<figure><img src="../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>



{% hint style="info" %}
The input is sanitized and it's not vulnerable to a command injection attack.
{% endhint %}

## References

For the making of this solution the following resource were used:

* [https://github.com/LeonardoE95/DVWA/](https://github.com/LeonardoE95/DVWA/tree/main/src/client\_side\_request\_forgery)
