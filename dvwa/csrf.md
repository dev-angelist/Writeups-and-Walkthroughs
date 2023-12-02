---
description: http://localhost/DVWA/vulnerabilities/csrf/
---

# CSRF

<details>

<summary>What is a Cross-Site Request Forgery (CSRF)?</summary>

CSRF stands for Cross-Site Request Forgery. It is a type of security vulnerability that occurs when an attacker tricks a user's browser into making an unintended and potentially malicious request on a website where the user is authenticated. The attack takes advantage of the fact that many websites rely solely on the user's authentication credentials (such as cookies) to determine the legitimacy of a request.

Here's a basic scenario of how CSRF works:

1. The victim logs into a legitimate website and receives authentication credentials (e.g., a session cookie).
2. While the victim is still authenticated, the attacker lures the victim into clicking on a specially crafted link or visiting a malicious website.
3. The malicious website or link contains a request to perform an action on the targeted website, using the victim's authentication credentials stored in the browser.
4. The browser, trusting the authentication credentials, sends the request to the legitimate website on behalf of the authenticated user, unknowingly carrying out the attacker's instructions.
5. The targeted website, not recognizing the request as malicious, performs the action as if it were a legitimate request from the authenticated user.

CSRF attacks can result in various malicious actions, such as changing account settings, making financial transactions, or even performing actions with elevated privileges if the targeted user has administrative rights.

To prevent CSRF attacks, web developers often use techniques such as anti-CSRF tokens, which are unique tokens embedded in forms or requests to verify the legitimacy of the request. These tokens make it more challenging for attackers to forge requests because they would need to obtain the token, which is typically tied to the user's session.

</details>

{% hint style="info" %}
Using BurpSuite and the FoxyProxy extension is recommended.
{% endhint %}

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption><p><a href="https://portswigger.net/web-security/csrf">https://portswigger.net/web-security/csrf</a></p></figcaption></figure>

{% embed url="https://owasp.org/www-community/attacks/csrf" %}
[https://owasp.org/www-community/attacks/csrf](https://owasp.org/www-community/attacks/csrf)
{% endembed %}

{% embed url="https://portswigger.net/web-security/csrf" %}
[https://portswigger.net/web-security/csrf](https://portswigger.net/web-security/csrf)
{% endembed %}

## Low

We've a form with two input type text:

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

that ask us to change admin password entering new password and confirming it.

As always we can start to analyze source code:

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

* There'is a condition to check if input value has been inserted
* Check if the two passwords match
* If there're the same, change admin psw.

{% hint style="warning" %}
The input is not sanitized, so I can execute any (potentially malicious) command.
{% endhint %}

Considering that GET request to change psw is this

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

we can use and insert it in a URL to send at victim usually using social engineering techniques.

#### Payload

```bash
localhost/DVWA/vulnerabilities/csrf/?password_new=password1&password_conf=password1&Change=Change#
```

using it, we can change password to a different password (password1) only opening payload URL via browser.

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

the same payload can be injected in a javascript code stored in another web page, but it can't work if the website used a **CORS** mechanism (Cross-origin resource sharing).

<details>

<summary>Cross-origin resource sharing (CORS)</summary>

Cross-origin resource sharing is an HTTP-header based mechanism that allows a server to indicate any origins (domain, scheme, or port) other than its own from which a browser should permit loading resources. CORS also relies on a mechanism by which browsers make a "preflight" request to the server hosting the cross-origin resource, in order to check that the server will permit the actual request.

</details>

## Medium

<figure><img src="../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

* There's a classic control of input text submitted
* Check if the HTTP\_REFERER request has the same name and origin of name server (request isn't sent by an external source)
* The operating system in use is checked to evaluate exactly which ping should be entered (win or \*nix OS)
* In the end, there's generate feedback for the end user.

<figure><img src="../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

If the HTTP\_REFERER is the same of the server name: csrf in our case, there're not problem;

while, using the same payload of level low (that has a different origin), HTTP\_REFERER link is not generated and we can't change password.

```url
localhost/DVWA/vulnerabilities/csrf/?password_new=password1&password_conf=password1&Change=Change#
```

<figure><img src="../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

#### Payload

To exploit it, is necessary have an HTTP\_REFERER request with the same name server, then we can use XSS Reflected attack redirecting the attacker to the desired page, where it's run a malicious javascript code.

We can try to inject an example of javascript code such as:&#x20;

```html
<Script> alert("Hello World"); </Script>
```

into XSS Reflected section\


<figure><img src="../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

<div align="left">

<figure><img src="../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

</div>

Very well, using this XSS we can trigger the malicious password change request with the following XSS script:

```html
<Script> var xmlHttp = new XMLHttpRequest(); var url = "http://localhost/DVWA//vulnerabilities/csrf/?password_new=newpass&password_conf=newpass&Change=Change"; xmlHttp.open("GET", url, false); xmlHttp.send(null); </Script>
```

<figure><img src="../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

It usually need to convert URL-encode key characters and we can redirect user to localhost/DVWA page;

<figure><img src="../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

Obtaining the HTTP\_REFERER origin desidered and run malicious javascript code to change psw.

{% hint style="warning" %}
Only checking if the HTTP\_REFERER request has the same name and origin of name server we can't be sure that request is valid, because can be redirect using XSS Reflected attack.
{% endhint %}

## High

<figure><img src="../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

* There's a classic control of input text submitted
* Check if the HTTP\_REFERER request has the same name and origin of name server (request isn't sent by an external source)
* The operating system in use is checked to evaluate exactly which ping should be entered (win or \*nix OS)
* In the end, there's generate feedback for the end user.















This command isn't write correctly, it has an extra space '`|`  '&#x20;





{% hint style="warning" %}
The input is not sanitized, so I can execute any (potentially malicious) command.
{% endhint %}

Payload

Without leaving a white space after | we can use this payload:

```bash
127.0.0.1 |whoami
```

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

## Impossible



<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>



{% hint style="info" %}
The input is sanitized and it's not vulnerable to a command injection attack.
{% endhint %}
