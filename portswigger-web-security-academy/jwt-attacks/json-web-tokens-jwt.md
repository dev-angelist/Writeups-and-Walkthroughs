---
icon: '1'
---

# Json Web Tokens (JWT)

## What are JWTs? <a href="#what-are-jwts" id="what-are-jwts"></a>

JSON web tokens (JWTs) are a standardized format for sending cryptographically signed JSON data between systems. They can contain any kind of data, but are most commonly used to claim/send info about users as a part of authentication, session handling and access control mechanisms.

In this case, all data that a server needs is stored client-side within the JWT, so it is a good way that permits to interact users with multiple back-end servers (that's the standard situation in the modern system architectures).

### JWT format

A JWT consists of 3 parts: a header, a payload, and a signature, each separated by a dot.

* Header: base64url-encoded, contains metadata about the token;
* Payload: base64url-encoded, contains the actual "claims" about the user (decoding it we can see more interesting data: name, email, etc);
* Signature: it usually generated from the server that issues the token throught the hashing of the header and payload.

To analyze well the structure of JWTs we can use the debugger on `jwt.io`.

### JWT vs JWS vs JWE <a href="#jwt-vs-jws-vs-jwe" id="jwt-vs-jws-vs-jwe"></a>

<figure><img src="../../.gitbook/assets/image (498).png" alt=""><figcaption><p><a href="https://portswigger.net/web-security/jwt/images/jwt-jws-jwe.jpg">https://portswigger.net/web-security/jwt/images/jwt-jws-jwe.jpg</a></p></figcaption></figure>

The JWT defines a format for representing information ("claims") as a JSON object that can be transferred between two parties. It's really used as a standalone entity and is more frequently extended by: JSON Web Signature (JWS) and JSON Web Encryption (JWE) specifications.

So, JWT is usually either a JWS or JWE token (90% JWS). In JWS token the data are encoded, while into JWE are encrypted.
