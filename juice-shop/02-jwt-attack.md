# JWT Authentication Bypass — OWASP Juice Shop

## Objective

The objective of this exercise was to identify and exploit an insecure JSON Web Token (JWT) implementation in the OWASP Juice Shop.

The goal was to modify a valid JWT so that the application would treat the authenticated user as an administrator and grant access to the administration functionality.

---

## Target

The application used for this exercise was **OWASP Juice Shop**, running locally at:

```text
http://127.0.0.1:3000
```

## Exploitation

### 1. Obtain the Original JWT

After authenticating to the Juice Shop application, I inspected the browser's stored application data and located the JWT stored in Local Storage.

The token was stored under the key `token`.

The JWT consisted of three components:

```
HEADER.PAYLOAD.SIGNATURE
```

### 2. Decode the JWT

I decoded the JWT to inspect its header and payload.

The JWT header contained:

```json
{
  "typ": "JWT",
  "alg": "RS256"
}
```

The token was originally using the RS256 signing algorithm.

The decoded payload also contained information about the authenticated user, including the user's role. The original role was:

```json
{
  "role": "customer"
}
```

### 3. Change the JWT Algorithm

The JWT header specified the cryptographic algorithm used to validate the token.

I modified the algorithm from `RS256` to `none`. The modified header became:

```json
{
  "typ": "JWT",
  "alg": "none"
}
```

The `none` algorithm indicates that the JWT does not contain a cryptographic signature.

### 4. Modify the User Role

After modifying the JWT algorithm, I modified the authorization information in the payload.

I changed the role from `customer` to `admin`. The modified payload therefore contained:

```json
{
  "role": "admin"
}
```

This was the privilege-escalation step of the attack. The JWT was therefore modified in two important ways:

1. `alg` was changed from `RS256` to `none`.
2. `role` was changed from `customer` to `admin`.

### 5. Remove the Signature

Because the JWT was changed to use `{"alg": "none"}`, the signature portion was removed from the token.

A normal JWT has the following structure:

```
HEADER.PAYLOAD.SIGNATURE
```

The forged token therefore had the structure:

```
HEADER.PAYLOAD.
```

The final period represents the empty signature portion.

### 6. Replace the Token in Local Storage

I opened the browser's Developer Tools and navigated to:

```
Application
    └── Local Storage
        └── http://127.0.0.1:3000
```

The existing token value was replaced with the forged JWT, stored using the same `token` key.

### 7. Access the Administration Page

After replacing the original token, I reloaded the application.

The application accepted the forged JWT and interpreted the session as belonging to an administrator because the payload contained `{"role": "admin"}`.

I then navigated to:

```
http://127.0.0.1:3000/#/administration
```

The administration page became accessible. The page displayed administrative functionality including:

- Registered Users
- Customer Feedback
- Administrative controls

This confirmed that the JWT manipulation successfully resulted in administrator-level access.

## JWT Structure

A JSON Web Token consists of three Base64URL-encoded sections:

```
HEADER.PAYLOAD.SIGNATURE
```

**Original Header**

```json
{
  "typ": "JWT",
  "alg": "RS256"
}
```

**Forged Header**

```json
{
  "typ": "JWT",
  "alg": "none"
}
```

**Original Role**

```json
{
  "role": "customer"
}
```

**Forged Role**

```json
{
  "role": "admin"
}
```

**Final Token Structure**

Because the `none` algorithm was used, the signature was omitted:

```
HEADER.PAYLOAD.
```

## JWT Changes

The following changes were made during exploitation:

| Component | Original | Modified |
|---|---|---|
| Algorithm | RS256 | none |
| Role | customer | admin |
| Signature | Present | Removed |

The two most important modifications were:

- `alg`: RS256 -> none
- `role`: customer -> admin

## Attack Flow

```
Authenticate to Juice Shop
          |
          v
Obtain valid JWT
          |
          v
Decode JWT
          |
          v
Change alg: RS256 -> none
          |
          v
Change role: customer -> admin
          |
          v
Remove JWT signature
          |
          v
Replace token in Local Storage
          |
          v
Reload application
          |
          v
Access /#/administration
          |
          v
Administrator Access
```

## Why the Attack Worked

The vulnerability existed because the application trusted security-sensitive information supplied inside the JWT.

The JWT specified `{"alg": "none"}` and the payload specified `{"role": "admin"}`.

Instead of rejecting the unsigned token and independently validating the user's privileges, the application accepted the modified token. This allowed the role claim to be manipulated from `customer` to `admin`.

The application subsequently treated the authenticated session as an administrator session.

## Evidence

The exploitation was documented using the following screenshots.

**Original JWT**
`screenshots/jwt-01-original-token.png`
Shows the original JWT stored by the application.

**Decoded JWT**
`screenshots/jwt-02-token-decoded.png`
Shows the decoded JWT structure and the original claims.

**Modified JWT Header**
`screenshots/jwt-03-header-alg-none.png`
Shows the JWT header modified from `RS256` to `none`.

**Forged Token in Local Storage**
`screenshots/jwt-04-localstorage-forged-token.png`
Shows the forged JWT replacing the original token in the browser's Local Storage. The forged token contained the modified administrator role.

**Administration Access**
`screenshots/jwt-05-administration-access.png`
Shows successful access to the Juice Shop administration page after replacing the JWT.

## Conclusion

The OWASP Juice Shop JWT implementation was vulnerable to a JWT authentication and privilege-escalation attack.

The exploitation process was:

1. Obtain a valid JWT.
2. Decode the JWT header and payload.
3. Identify the original RS256 algorithm.
4. Change `alg` from `RS256` to `none`.
5. Change `role` from `customer` to `admin`.
6. Remove the JWT signature.
7. Replace the original token in Local Storage.
8. Reload the application.
9. Navigate to the administration page.
10. Confirm administrator-level access.

The successful attack demonstrated that an attacker could manipulate JWT authentication and authorization information to escalate privileges.

## Key Takeaways

- JWTs consist of a header, payload, and signature.
- The JWT header contains the algorithm used for signing and verification.
- Applications should never blindly trust the `alg` value supplied by a client-controlled JWT.
- The `none` algorithm should not be accepted for authenticated sessions unless there is an explicitly justified and secure use case.
- Authorization claims such as `role` must not be trusted unless the token has been properly authenticated and validated.
- Changing `role: customer` to `role: admin` demonstrates how insecure JWT validation can lead to privilege escalation.
- JWT signature validation is essential for ensuring that the token payload has not been modified.

## Remediation

The application should:

- Explicitly allow only approved JWT signing algorithms.
- Reject JWTs using the `none` algorithm.
- Always validate JWT signatures on the server.
- Never trust client-controlled authorization claims without successful signature verification.
- Validate claims such as issuer, audience, expiration, and subject.
- Perform authorization checks server-side.
- Use strong cryptographic keys and appropriate key management.
- Avoid relying solely on JWT claims for sensitive authorization decisions.
- Ensure that modifying the JWT payload invalidates the token.

## Final Result

The JWT was successfully manipulated by changing `RS256 -> none` and `customer -> admin`, and removing the signature.

The forged token was accepted by the application, allowing access to `/#/administration`.

This confirmed the JWT authentication and privilege-escalation vulnerability in the OWASP Juice Shop.
