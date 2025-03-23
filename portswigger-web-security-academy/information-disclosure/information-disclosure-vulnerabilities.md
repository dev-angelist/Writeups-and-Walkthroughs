---
icon: '1'
---

# Information disclosure vulnerabilities

## What is information disclosure? <a href="#what-is-information-disclosure" id="what-is-information-disclosure"></a>

Information disclosure, also known as information leakage, is when a website unintentionally reveals sensitive information to its users, such as: data about other users (user, financial info), sensitive commercial or business data, technical details about the website and its infrastracture, etc).

### Examples of information disclosure <a href="#examples-of-information-disclosure" id="examples-of-information-disclosure"></a>

* Revealing the names of hidden directories, their structure, and their contents via a `robots.txt` file or directory listing
* Providing access to source code files via temporary backups
* Explicitly mentioning database table or column names in error messages
* Unnecessarily exposing highly sensitive information, such as credit card details
* Hard-coding API keys, IP addresses, database credentials, and so on in the source code
* Hinting at the existence or absence of resources, usernames, and so on via subtle differences in application behavior

## Testing for information disclosure

Sensitive data can be leaked in all kinds of places, and understand if an info can be important is the key. Some examples of high-level techniques and tools to identify information disclosure vulnerabilities during testing are:

* Fuzzing
* Burp Scanner
* Burp's engagement tools
* Engineering informative responses

### Fuzzing <a href="#context-specific-decoding" id="context-specific-decoding"></a>

If you identify interesting parameters, you can try submitting unexpected data types and specially crafted fuzz strings to see what effect this has.

For example, we can have a slight difference in the time taken to process the request or different response.

Using Burp Intruder is possible to automate this process fuzzing parameter in a part of request and use a pre-buil wordlist, comparing the differences in responses (different HTTP status codes, response times, lenghts, and more (eventually grep them).

### Burp Scanner <a href="#using-burp-scanner" id="using-burp-scanner"></a>

**Burp Suite Professional** users have the benefit of Burp Scanner. This provides live scanning features for auditing items while you browse, or you can schedule automated scans to crawl and audit the target site on your behalf.

It will alert you if it finds sensitive information such as private keys, email addresses, and credit card numbers in a response. It will also identify any backup files, directory listings, and so on.

### Burp's engagement tools <a href="#using-burp-s-engagement-tools" id="using-burp-s-engagement-tools"></a>

**Burp Suite Professional** provides several engagement tools that you can use to find interesting information in the target website more easily. You can access the engagement tools from the context menu - just right-click on any HTTP message, Burp Proxy entry, or item in the site map and go to "Engagement tools".

### Engineering informative responses <a href="#engineering-informative-responses" id="engineering-informative-responses"></a>

Verbose error messages can sometimes disclose interesting information while you go about your normal testing workflow. However, by studying the way error messages change according to your input, you can take this one step further. In some cases, you will be able to manipulate the website in order to extract arbitrary data via an error message.

There are numerous methods for doing this depending on the particular scenario you encounter. One common example is to make the application logic attempt an invalid action on a specific item of data. For example, submitting an invalid parameter value might lead to a stack trace or debug response that contains interesting details.
