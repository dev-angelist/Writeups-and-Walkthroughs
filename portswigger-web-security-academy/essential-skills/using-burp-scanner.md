---
description: >-
  https://portswigger.net/web-security/essential-skills/using-burp-scanner-during-manual-testing
icon: '2'
---

# Using Burp Scanner

## Scanning a specific request <a href="#scanning-a-specific-request" id="scanning-a-specific-request"></a>

When you come across an interesting function or behavior, your first instinct may be to send the relevant requests to Repeater or Intruder and investigate further.

In the **Pro vs**, If you right-click on a request and select **Do active scan**, Burp Scanner will use its default configuration to audit only this request.

This may not catch every last vulnerability, but it could potentially flag things up in seconds that could otherwise have taken hours to find. It may also help you to rule out certain attacks almost immediately.\


## Scanning custom insertion points

In the **Pro vs**, is possible to limit scans setting a custom insertion points.

For people that have Community Edition is possible to use a similar method thanks to Intruder, where we select an insertion point, typically a parameter value (eg. id value) and inject which we want.

## Scanning non-standard data structures

&#x20;Talking about non-standard data structures eg. `user=048857-carlos` the '-' character can undestand an additional data, so we can consider onlu 'carlos' value scanning a defined selected insertion point or using Intruder.

## Labs 🔬

* Discovering vulnerabilities quickly with targeted scanning
* Scanning non-standard data structures

{% hint style="info" %}
Since I don't have the Pro version at the moment, I decided to skip these labs for now.
{% endhint %}
