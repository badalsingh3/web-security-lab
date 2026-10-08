# Web Security Labs

A hands-on collection of web application security labs and write-ups covering common vulnerabilities and exploitation techniques.

The labs in this repository were performed in intentionally vulnerable, authorized training environments, primarily **PortSwigger Web Security Academy** and **OWASP Juice Shop**.

---

## 📚 Contents

### PortSwigger Web Security Academy

| Lab | Vulnerability | Write-up |
|---|---|---|
| SQL Injection | SQL Injection / Database Enumeration | [SQLi](portswigger/sqli.md) |
| Cross-Site Scripting | Stored XSS | [XSS](portswigger/xss.md) |
| CSRF | CSRF Token Validation Bypass | [CSRF](portswigger/csrf.md) |
| IDOR / Access Control | Broken Access Control / IDOR | [IDOR](portswigger/idor-access-control.md) |

### OWASP Juice Shop

| Lab | Vulnerability | Write-up |
|---|---|---|
| SQL Injection | Authentication Bypass | [SQL Injection](juice-shop/01-sql-injection.md) |
| JWT Attack | JWT Authentication / Privilege Escalation | [JWT Attack](juice-shop/02-jwt-attack.md) |

---

## 🧪 Vulnerabilities Covered

This repository currently covers:

- SQL Injection (SQLi)
- Authentication Bypass
- Stored Cross-Site Scripting (XSS)
- Cross-Site Request Forgery (CSRF)
- Insecure Direct Object References (IDOR)
- Broken Access Control
- JWT Authentication Vulnerabilities
- JWT `alg: none` attacks
- JWT-based privilege escalation
- Database enumeration
- HTTP request manipulation

---

## 🛠️ Tools Used

- Burp Suite
- Burp Suite Repeater
- Browser Developer Tools
- OWASP Juice Shop
- PortSwigger Web Security Academy
- SQL injection payloads
- JWT decoding and manipulation
- HTTP request/response analysis

---

## 📁 Repository Structure

```text
.
├── juice-shop/
│   ├── 01-sql-injection.md
│   └── 02-jwt-attack.md
│
├── portswigger/
│   ├── csrf.md
│   ├── idor-access-control.md
│   ├── sqli.md
│   └── xss.md
│
├── screenshots/
│   ├── csrf-portswigger.png
│   ├── idor-portswigger.png
│   ├── jwt-01-original-token.png
│   ├── jwt-02-token-decoded.png
│   ├── jwt-03-header-alg-none.png
│   ├── jwt-04-localstorage-forged-token.png
│   ├── jwt-05-administration-access.png
│   ├── sqli-01-login-page.png
│   ├── sqli-02-normal-login-request.png
│   ├── sqli-03-sqli-burp-repeater.png
│   ├── sqli-04-authenticated-admin.png
│   ├── sqli-portswigger.png
│   └── xss-portswigger.png
│
└── README.md
```

---

## 🎯 Purpose

The purpose of this repository is to document practical web application security testing and build a reference of common vulnerabilities, exploitation techniques, and mitigation strategies.

Each write-up documents:

1. The vulnerability being tested
2. The vulnerable functionality
3. The exploitation methodology
4. Relevant HTTP requests or payloads
5. Evidence from the lab
6. Why the vulnerability works
7. Security implications
8. Recommended remediation

---

## 🔬 Methodology

The general workflow used throughout the labs was:

```
Reconnaissance
      |
      v
Identify Attack Surface
      |
      v
Capture HTTP Request
      |
      v
Analyze Request / Response
      |
      v
Identify Vulnerability
      |
      v
Develop Payload
      |
      v
Exploit Vulnerability
      |
      v
Verify Impact
      |
      v
Document Findings
      |
      v
Recommend Remediation
```

Burp Suite was primarily used to intercept, modify, and replay HTTP requests during testing.

---

## 📖 Write-ups

**PortSwigger**

- [SQL Injection](portswigger/sqli.md)
- [Stored XSS](portswigger/xss.md)
- [CSRF — Token Validation Depends on Request Method](portswigger/csrf.md)
- [IDOR / Access Control](portswigger/idor-access-control.md)

**OWASP Juice Shop**

- [SQL Injection — Authentication Bypass](juice-shop/01-sql-injection.md)
- [JWT Authentication Bypass & Privilege Escalation](juice-shop/02-jwt-attack.md)

---

## ⚠️ Disclaimer

All testing documented in this repository was performed against intentionally vulnerable applications and authorized security-training labs.

The techniques and payloads documented here are provided for educational purposes and should only be used against systems for which you have explicit permission to conduct security testing.

---

## 📌 Progress

This repository is an ongoing collection of web security labs.

More vulnerability classes, labs, and detailed write-ups will be added as the learning process continues.
