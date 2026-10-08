# IDOR — User ID Controlled by Request Parameter

## Objective

The objective of this lab was to exploit an **Insecure Direct Object Reference (IDOR)** vulnerability in the application's account functionality.

The application uses a user-controlled request parameter to identify which user's account information should be displayed.

The goal was to access another user's account information by modifying the user ID in the request.

## Vulnerability

The application determines which user account to display based on a request parameter.

For example:

```text
id=wiener
```

The application does not properly verify that the authenticated user is authorized to access the requested account.

As a result, changing the parameter to another valid username allows access to that user's account information.

This is an example of Broken Access Control, specifically an IDOR vulnerability.

## Exploitation

### 1. Log In

I first logged into the application using the credentials provided by the lab.

```
Username: wiener
Password: peter
```

### 2. Identify the User ID Parameter

After logging in, I navigated to the account page.

The application generated a request containing the username as a request parameter.

The request was similar to:

```http
GET /my-account?id=wiener HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: session=YOUR_SESSION
```

The important part of the request was:

```
id=wiener
```

This indicated that the requested account was being selected using a user-controlled parameter.

### 3. Test for IDOR

I sent the request to Burp Suite Repeater and changed the `id` parameter from `id=wiener` to `id=carlos`.

The modified request was:

```http
GET /my-account?id=carlos HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: session=YOUR_SESSION
```

### 4. Observe the Response

The application returned the account information belonging to carlos even though I was authenticated as wiener.

This demonstrated that the application was relying on the user-controlled `id` parameter without properly checking whether the authenticated user was authorized to access the requested account.

## Why the Attack Worked

The application effectively performed an operation similar to:

```
requested_user = request_parameter("id")
display_account(requested_user)
```

Instead of verifying:

```
authenticated_user == requested_user
```

the application trusted the supplied `id` value.

Therefore:

| | |
|---|---|
| Authenticated user | `wiener` |
| Requested user | `carlos` |
| Authorization check | Missing / insufficient |

The server returned Carlos's account information.

## Attack Flow

```
Attacker logs in as wiener
          |
          v
GET /my-account?id=wiener
          |
          v
Application returns wiener's account
          |
          v
Change parameter
id=wiener -> id=carlos
          |
          v
GET /my-account?id=carlos
          |
          v
Application fails to perform authorization check
          |
          v
Carlos's account information returned
          |
          v
IDOR confirmed
```

## Key Request

Original request:

```http
GET /my-account?id=wiener HTTP/1.1
```

Modified request:

```http
GET /my-account?id=carlos HTTP/1.1
```

The important change was `id=wiener` to `id=carlos`.

No change to the authenticated session was required.

## Conclusion

The lab was successfully exploited by modifying the user ID in the request.

The exploitation process was:

1. Log in as wiener.
2. Navigate to the account page.
3. Identify the `id` request parameter.
4. Capture the request using Burp Suite.
5. Change `id=wiener` to `id=carlos`.
6. Send the modified request.
7. Observe that Carlos's account information was returned.
8. Confirm the IDOR / broken access control vulnerability.

## Key Takeaways

- IDOR vulnerabilities occur when an application uses user-controlled identifiers to access objects without performing proper authorization checks.
- Knowing or guessing another user's identifier can allow unauthorized access to their data.
- Authentication and authorization are different concepts: being logged in does not mean a user is authorized to access every account.
- Authorization must be enforced server-side for every protected resource.

## Remediation

The application should:

- Perform server-side authorization checks for every account/resource request.
- Never rely on user-supplied IDs as proof of authorization.
- Verify that the authenticated user has permission to access the requested resource.
- Prefer indirect or unpredictable identifiers where appropriate, while recognizing that obscurity alone is not an access-control mechanism.
- Return an appropriate 403 Forbidden response when an authenticated user attempts to access an unauthorized resource.
