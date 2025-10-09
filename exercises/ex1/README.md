# Exercise 1 - Broken Access Control

## 📖 Overview :
Broken Access Control  is the most critical web application security risk, according to the [OWASP Top 10 2021 list (A01)](https://owasp.org/Top10/A01_2021-Broken_Access_Control/). It occurs when an application fails to enforce proper authorization, allowing users to view or modify resources they are not permitted to access. When access control is broken, threat actors can act outside of their intended permissions. This can manifest in several ways:

- Horizontal Privilege Escalation.
- Vertical Privilege Escalation.
- Insecure Direct Object References (IDOR).

## ⚠️ Why This Matters

* **Business Impact:** Unauthorized modifications could lead to incorrect incident handling, data tampering, and workflow disruption.
* **Compliance Risk:** Violates [OWASP Top 10 A01](https://owasp.org/Top10/A01_2021-Broken_Access_Control/) and the principle of least privilege.
* **Security Risk:** malicious or careless users could alter other peoples' work, close tickets improperly, or delete evidence.

## 🔐 CAP Security Concept: 

CAP provides a multi-layered security approach:

- Authentication: Verifies user identity (managed by XSUAA/IAS).

- Authorization: Controls what authenticated users can do:
    - Role-based (@requires annotations).
    - Instance-based (@restrict annotations).
    - Programmatic checks (in service handlers).

There are two excercises related to this topic.

Continue to [Exercise 1.1 - Horizontal Privilege Escalation](./ex1.1/README.md)

and then  [Exercise 1.2 - Vertical Privilege Escalation](./ex1.2/README.md)


## Summary

When you have finished the two excercises related to Broken Access Control

continue to - [Exercise 2 - SQL injection](../ex2/README.md)

