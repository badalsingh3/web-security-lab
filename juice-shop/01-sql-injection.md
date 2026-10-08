# SQL Injection — OWASP Juice Shop

## Objective

The objective of this exercise was to identify and exploit a **SQL Injection (SQLi)** vulnerability in the OWASP Juice Shop login functionality.

The goal was to bypass the authentication mechanism and gain access to the administrator account without knowing the administrator's password.

## Target

The application used for this exercise was **OWASP Juice Shop**, running locally at:

```text
http://127.0.0.1:3000
```

## Exploitation

### 1. Identify the Login Page

I first navigated to the Juice Shop login page.

The application provided two input fields:

- Email
- Password

The login page was the initial attack surface tested for SQL injection.

### 2. Capture a Normal Login Request

I entered normal test credentials and intercepted the login request using Burp Suite.

The application sent the credentials to:

```
POST /rest/user/login
```

The request body used JSON and contained the supplied email and password:

```json
{
  "email": "test@gmail.com",
  "password": "test123"
}
```

This confirmed that the login credentials were being sent to the `/rest/user/login` endpoint.

### 3. Test for SQL Injection

I sent the captured login request to Burp Suite Repeater so that I could modify the request manually.

I replaced the email value with a SQL injection payload:

```
' OR 1=1-- a
```

The resulting JSON request was:

```json
{
  "email": "' OR 1=1-- a",
  "password": "test123"
}
```

The payload attempts to alter the SQL query's logic so that the authentication condition evaluates as true.

The `--` begins a SQL comment, causing the remainder of the original query to be ignored.

The additional `a` after the comment marker was used to ensure the comment syntax was handled correctly by the backend.

### 4. Analyze the Response

After sending the modified request, the server returned a successful response.

The response contained authentication information, including a JWT token and the authenticated user's details.

The response indicated that the SQL injection had successfully bypassed the normal authentication process.

The returned user information included:

```json
{
  "bid": 1,
  "email": "admin@juice-sh.op"
}
```

This demonstrated that the application had authenticated the request as the administrator.

### 5. Access the Administrator Account

After the successful SQL injection, the application issued an authentication token and the browser became authenticated as the administrator.

The Juice Shop user profile showed:

```
Email: admin@juice-sh.op
```

This confirmed that the authentication bypass was successful.

## SQL Injection Payload

The payload used during the exploitation was:

```
' OR 1=1-- a
```

It was supplied through the email parameter:

```json
{
  "email": "' OR 1=1-- a",
  "password": "test123"
}
```

### Payload Breakdown

| Part | Purpose |
|---|---|
| `'` | Closes the string context in the original SQL query. |
| `OR 1=1` | Adds a condition that is always true. |
| `-- a` | Comments out the remainder of the SQL statement. |

Conceptually, the injection changes the authentication logic from a normal credential comparison into a condition that can evaluate as true.

## Attack Flow

```
Juice Shop Login Page
        |
        v
Capture Login Request
        |
        v
POST /rest/user/login
        |
        v
Send Request to Burp Repeater
        |
        v
Inject SQL Payload
' OR 1=1-- a
        |
        v
Server Processes Modified Query
        |
        v
Authentication Bypass
        |
        v
JWT Authentication Token Returned
        |
        v
Authenticated as Administrator
```

## Evidence

The exploitation was documented using the following screenshots:

**Login Page**
`screenshots/sqli-01-login-page.png`
Shows the initial OWASP Juice Shop login interface.

**Normal Login Request**
`screenshots/sqli-02-normal-login-request.png`
Shows the legitimate `POST /rest/user/login` request captured in Burp Suite.

**SQL Injection Request**
`screenshots/sqli-03-sqli-burp-repeater.png`
Shows the modified request containing the SQL injection payload: `' OR 1=1-- a`

**Authenticated Administrator**
`screenshots/sqli-04-authenticated-admin.png`
Shows successful authentication as the administrator account.

## Conclusion

The OWASP Juice Shop login functionality was vulnerable to SQL Injection.

By modifying the email parameter with `' OR 1=1-- a`, I was able to manipulate the authentication query and bypass the normal login process.

The server subsequently returned a valid authentication token and authenticated the session as `admin@juice-sh.op`.

This demonstrated that an attacker could bypass authentication by injecting SQL into the login request.

## Key Takeaways

- SQL Injection occurs when untrusted user input is incorporated into SQL queries without proper parameterization.
- Authentication endpoints are particularly sensitive SQL injection targets because successful exploitation can result in account takeover.
- Burp Suite Repeater can be used to modify and replay HTTP requests when testing for SQL injection.
- Boolean conditions such as `1=1` can be used to test whether injected SQL changes the application's query logic.
- SQL comments can be used to remove the remainder of the original query from execution.

## Remediation

The application should:

- Use parameterized queries / prepared statements instead of dynamically constructing SQL queries.
- Never concatenate user-controlled input directly into SQL statements.
- Apply appropriate input validation as an additional security layer.
- Use secure authentication mechanisms and properly handle authentication failures.
- Ensure database accounts used by the application have only the minimum privileges required.
- Avoid exposing sensitive database or authentication information in application responses.
