---
icon: '2'
---

# Common sources of information disclosure

## Common sources of information disclosure <a href="#common-sources-of-information-disclosure" id="common-sources-of-information-disclosure"></a>

Information disclosure can occur in a wide variety of contexts within a website. The following are some common examples of places where you can look to see if sensitive information is exposed.

* Files for web crawlers
* Directory listings
* Developer comments
* Error messages
* Debugging data
* User account pages
* Backup files
* Insecure configuration
* Version control history

### Files for web crawlers <a href="#files-for-web-crawlers" id="files-for-web-crawlers"></a>

Many websites provide files at `/robots.txt` and `/sitemap.xml` to help crawlers navigate their site. Among other things, these files often list specific directories that the crawlers should skip, for example, because they may contain sensitive information.

### Directory listings

Web servers can be configured to automatically list the contents of directories that do not have an index page present. This can aid an attacker by enabling them to quickly identify the resources at a given path, and proceed directly to analyzing and attacking those resources.

It's not a security vulnerability, but it can potentially leak interesting sensitive resources.

### Developer comments

During development, in-line HTML comments are sometimes added to the markup. These comments are typically stripped before changes are deployed to the production environment.

In the CTFs easy is more frequent find visible comments viewing page source (CTRL+U).

### Error messages

One of the most common causes of information disclosure is verbose error messages. As a general rule, you should pay close attention to all error messages you encounter during auditing.

The content of error messages can reveal information about what input or data type is expected from a given parameter. This can help you to narrow down your attack by identifying exploitable parameters. Verbose error messages can also provide information about different technologies being used by the website. For example, they might explicitly name a template engine, database type, or server that the website is using, along with its version number.

### Debugging data

For debugging purposes, many websites generate custom error messages and logs that contain large amounts of information about the application's behavior. While this information is useful during development, it is also extremely useful to an attacker if it is leaked in the production environment.

Debug messages can sometimes contain vital information for developing an attack, including:

* Values for key session variables that can be manipulated via user input
* Hostnames and credentials for back-end components
* File and directory names on the server
* Keys used to encrypt data transmitted via the client

### User account pages

By their very nature, a user's profile or account page usually contains sensitive information, such as the user's email address, phone number, API key, and so on.

However, some websites contain logic flaws that potentially allow an attacker to leverage these pages in order to view other users' data (IDOR): `GET /user/personal-info?user=carlos`.

### Backup files

Obtaining source code access makes it much easier for an attacker to understand the application's behavior and construct high-severity attacks. Sensitive data is sometimes even hard-coded within the source code. Typical examples of this include API keys and credentials for accessing back-end components.

### Insecure configuration

Websites are sometimes vulnerable as a result of improper configuration. This is especially common due to the widespread use of third-party technologies, whose vast array of configuration options are not necessarily well-understood by those implementing them.

In other cases, developers might forget to disable various debugging options in the production environment. For example, the HTTP `TRACE` method is designed for diagnostic purposes. If enabled, the web server will respond to requests that use the `TRACE` method by echoing in the response the exact request that was received.&#x20;

### Version control history

Virtually all websites are developed using some form of version control system, such as Git. By default, a Git project stores all of its version control data in a folder called `.git`. Occasionally, websites expose this directory in the production environment. In this case, you might be able to access it by simply browsing to `/.git`.

### How to prevent information disclosure vulnerabilities <a href="#how-to-prevent-information-disclosure-vulnerabilities" id="how-to-prevent-information-disclosure-vulnerabilities"></a>

Preventing information disclosure completely is tricky due to the huge variety of ways in which it can occur. However, there are some general best practices that you can follow to minimize the risk of these kinds of vulnerability creeping into your own websites.

* Make sure that everyone involved in producing the website is fully aware of what information is considered sensitive.
* Audit any code for potential information disclosure as part of your QA or build processes.
* Use generic error messages as much as possible.
* Double-check that any debugging or diagnostic features are disabled in the production environment. Make sure you fully understand the configuration settings, and security implications, of any third-party technology that you implement.

## Labs 🔬

* [Information disclosure in error messages](information-disclosure-in-error-messages.md)
* [Information disclosure on debug page](information-disclosure-on-debug-page.md)
* [Source code disclosure via backup files](source-code-disclosure-via-backup-files.md)
* [Authentication bypass via information disclosure](authentication-bypass-via-information-disclosure.md)
* [Information disclosure in version control history](information-disclosure-in-version-control-history.md)
