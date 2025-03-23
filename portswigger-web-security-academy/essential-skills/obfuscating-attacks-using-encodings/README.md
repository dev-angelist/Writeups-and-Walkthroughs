---
icon: '1'
---

# Obfuscating attacks using encodings

## Context-specific decoding <a href="#context-specific-decoding" id="context-specific-decoding"></a>

For encoding we means the convert data into another format (eg. base64) for various reason, like as: hiding, trasmit better, rapresent into a readeable format, etc; the inverted process is called 'decoding'.

Clients and server use a variety of different encodings to pass data between systems.

For example, a query paratemeter is typically URL decoded server-side, while the text content of an HTML ma be HTML decoded client-side.

Thinking as an attacker, it's necessary to undestand where exactly the payload is being inject, and potentially identify alternative ways to represent the same payload.

### Decoding discrepancies

Injection attacks often use recognizable patterns, such as HTML tags, JavaScript functions or SQL statements. These payload are usually filtered/blocked and it's vital that the deconding performed when checking the input is the same as the decoding performed by the back-end server or browser.

## Obfuscation via URL encoding <a href="#obfuscation-via-url-encoding" id="obfuscation-via-url-encoding"></a>

In URLs, there're a serios of reserved characheters for special meaning, such as: ampersand (&) is used as a delimiter to separate parameters in the query string.

Browsers automatically URL encode any characters that may cause ambiguity for parsers, for example for a search string "Fish & Chips", che & be substitute by % character and their 2-digit hex code: ?search=Fish+%26+Chips, ensuring that the ampersand will not be mistaken for a delimiter (the space first and before & can be encoded as %20).

### Obfuscation via double URL encoding <a href="#obfuscation-via-double-url-encoding" id="obfuscation-via-double-url-encoding"></a>

In some cases, servers perform two rounds of URL decoding on any URLs they receive.

This discrepancy enables an attacker to smuggle malicious input to the back-end by simply encoding it twice.\
E.g. injecting an XSS PoC such as: `<img src=x onerror=alert(1)>` when checking the request, the request is blocked from ever reaching the back-end, but in practice, the % characters are replaced with %25 and the WAF aren't able to undestand the the request is malicious.

### Obfuscation via HTML encoding <a href="#obfuscation-via-html-encoding" id="obfuscation-via-html-encoding"></a>

In HTML documents, certain characters need to be escaped or encoded to prevent the browser from incorrectly interpreting them as part of the markup.

If the server-side checks are looking for the `alert()` payload explicitly, they might not spot this if you HTML encode one or more of the characters:

`<img src=x onerror="&#x61;lert(1)">`

When the browser renders the page, it will decode and execute the injected payload.

#### Leading zeros

Interestingly, when using decimal or hex-style HTML encoding, you can optionally include an arbitrary number of leading zeros in the code points. Some WAFs and other input filters fail to adequately account for this.

If your payload still gets blocked after HTML encoding it, you may find that you can evade the filter just by prefixing the code points with a few zeros:

`<a href="javascript&#00000000000058;alert(1)">Click me</a>`

## Obfuscation via XML encoding <a href="#obfuscation-via-xml-encoding" id="obfuscation-via-xml-encoding"></a>

XML is closely related to HTML and also supports character encoding using the same numeric escape sequences. This enables you to include special characters in the text content of elements without breaking the syntax.\
So, it's not necessary to encode any special characters to avaid syntax errors

```xml
<stockCheck>
    <productId>
        123
    </productId>
    <storeId>
        999 &#x53;ELECT * FROM information_schema.tables
    </storeId>
</stockCheck>
```

## Obfuscation via unicode escaping <a href="#obfuscation-via-unicode-escaping" id="obfuscation-via-unicode-escaping"></a>

Unicode escape sequences consist of the prefix `\u` followed by the four-digit hex code for the character. For example, `\u003a` represents a colon. ES6 also supports a new form of unicode escape using curly braces: `\u{3a}`.

When parsing strings, most programming languages decode these unicode escapes.

For example, let's say you're trying to exploit DOM XSS where your input is passed to the `eval()` sink as a string. If your initial attempts are blocked, try escaping one of the characters as follows:

`eval("\u0061lert(1)")`

It's also worth noting that the ES6-style unicode escapes also allow optional leading zeros, so some WAFs may be easily fooled using the same technique we used for HTML encodings. For example:

`<a href="javascript:\u{00000000061}alert(1)">Click me</a>`

## Obfuscation via hex escaping

Another option when injecting into a string context is to use hex escapes, which represent characters using their hexadecimal code point, prefixed with `\x`. For example, the lowercase letter `a` is represented by `\x61`.

Just like unicode escapes, these will be decoded client-side as long as the input is evaluated as a string:

`eval("\x61lert")`

Note that you can sometimes also obfuscate SQL statements in a similar manner using the prefix `0x`. For example, `0x53454c454354` may be decoded to form the `SELECT` keyword.

## Obfuscation via octal escaping

Octal escaping works in pretty much the same way as hex escaping, except that the character references use a base-8 numbering system rather than base-16. These are prefixed with a standalone backslash, meaning that the lowercase letter `a` is represented by `\141`.

`eval("\141lert(1)")`

## Obfuscation via multiple encodings

It's possible to combine multiple encodings to hide your payloads behind multiple layers of obfuscation

Look at the `javascript:` URL in the following example:

`<a href="javascript:&bsol;u0061lert(1)">Click me</a>`

Browsers will first HTML decode `&bsol;,` resulting in a backslash. This has the effect of turning the otherwise arbitrary `u0061` characters into the unicode escape `\u0061`:

`<a href="javascript:\u0061lert(1)">Click me</a>`

This is then decoded further to form a functioning XSS payload:

`<a href="javascript:alert(1)">Click me</a>`

## Obfuscation via the SQL CHAR() function

Although not strictly a form of encoding, in some cases, you may be able to obfuscate your SQL injection attacks using the `CHAR()`function. This accepts a single decimal or hex code point and returns the matching character. Hex codes must be prefixed with `0x`. For example, both `CHAR(83)` and `CHAR(0x53)` return the capital letter `S`.

By concatenating the returned values, you can use this approach to obfuscate blocked keywords. For example, even if `SELECT` is blacklisted, the following injection initially appears harmless:

`CHAR(83)+CHAR(69)+CHAR(76)+CHAR(69)+CHAR(67)+CHAR(84)`

However, when this is processed as SQL by the application, it will dynamically construct the `SELECT` keyword and execute the injected query.

## Labs 🔬

* [SQL injection with filter bypass via XML encoding](sql-injection-with-filter-bypass-via-xml-encoding.md)
