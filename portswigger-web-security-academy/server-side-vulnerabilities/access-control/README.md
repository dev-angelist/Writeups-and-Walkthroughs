---
description: >-
  https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/what-is-access-control
icon: '2'
---

# Access control

## What is access control?

Access control is the application of constraints on who or what is authorized to perform actions or access resources. In the context of web applications, access control is dependent on authentication and session management:

* **Authentication** confirms that the user is who they say they are.
* **Session management** identifies which subsequent HTTP requests are being made by that same user.
* **Access control** determines whether the user is allowed to carry out the action that they are attempting to perform.

Broken access controls are common and often present a critical security vulnerability. Design and management of access controls is a complex and dynamic problem that applies business, organizational, and legal constraints to a technical implementation. Access control design decisions have to be made by humans so the potential for errors is high.

## Labs 🔬

* [Unprotected admin functionality](unprotected-admin-functionality.md)
* [Unprotected admin functionality with unpredictable URL](unprotected-admin-functionality-with-unpredictable-url.md)
* [User role controlled by request parameter](user-id-controlled-by-request-parameter-with-password-disclosure.md)
* [User ID controlled by request parameter, with unpredictable user IDs](user-id-controlled-by-request-parameter-with-unpredictable-user-ids.md)
* [User ID controlled by request parameter with password disclosure](user-id-controlled-by-request-parameter-with-password-disclosure.md)
