# Stored XSS into HTML Context with Nothing Encoded

## Objective

The objective of this lab was to identify and exploit a **stored Cross-Site Scripting (XSS)** vulnerability in the application's blog comments functionality.

Unlike reflected XSS, the malicious input is stored by the application and is later included in a response when the affected page is viewed.

## Vulnerability

The application allows users to submit comments on a blog post.

The submitted comment is stored by the application and later rendered as HTML without properly encoding the user's input.

Because the input is inserted directly into the HTML response, an attacker can inject HTML containing JavaScript.

## Exploitation

### 1. Identify the Vulnerable Input

I navigated to one of the blog posts and examined the available comment fields.

The comment form contained several user-controlled fields, including:

- Comment
- Name
- Email
- Website

The **Comment** field was used to test for stored XSS.

### 2. Test the Comment Field

I submitted the following payload in the comment field:

```html
<script>alert(1)</script>
```

### 3. Submit the Comment

After submitting the comment, the application stored the supplied input.

When the blog post was loaded again, the submitted comment was displayed on the page.

Instead of displaying the `<script>` element as plain text, the browser interpreted it as HTML and executed the JavaScript.

### 4. Confirm XSS Execution

The injected JavaScript executed:

```js
alert(1)
```

This caused a browser alert displaying `1`.

This confirmed that the application was vulnerable to stored XSS.

### 5. Complete the Lab

After the payload successfully executed, the PortSwigger lab was marked as solved.

## Payload

The XSS payload used was:

```html
<script>alert(1)</script>
```

The payload consists of:

- `<script>` — starts a JavaScript element.
- `alert(1)` — executes JavaScript and displays an alert containing `1`.
- `</script>` — closes the script element.

## Why the Payload Worked

The application failed to properly encode the contents of the comment before inserting it into the HTML page.

As a result, the browser interpreted the injected `<script>` element as executable HTML:

```html
<script>alert(1)</script>
```

The JavaScript was therefore executed whenever the page containing the stored comment was loaded.

## Stored XSS Attack Flow

```
Attacker
   |
   | Submit malicious comment
   v
Application
   |
   | Stores the comment
   v
Database
   |
   | Comment retrieved
   v
Blog Post
   |
   | Unsanitized HTML inserted
   v
Victim's Browser
   |
   | <script>alert(1)</script>
   v
JavaScript Executes
   |
   v
alert(1)
```

## Reflected XSS vs Stored XSS

### Reflected XSS

The malicious input is returned immediately in the server's response.

```
Attacker Input
      |
HTTP Request
      |
Server
      |
Immediate Response
      |
JavaScript Executes
```

### Stored XSS

The malicious input is first stored by the application and executes when the stored content is later viewed.

```
Attacker Input
      |
HTTP Request
      |
Server
      |
Database
      |
Stored Comment
      |
Page Loads
      |
JavaScript Executes
```

## Conclusion

The lab was successfully exploited using a stored XSS payload.

The exploitation process was:

1. Identify the blog comment functionality.
2. Determine that comments were stored and later displayed.
3. Inject the payload `<script>alert(1)</script>`.
4. Submit the malicious comment.
5. Reload the blog post containing the comment.
6. Confirm that `alert(1)` executed.
7. The lab was marked as solved.

## Key Takeaways

- Stored XSS occurs when malicious user input is stored by an application and later rendered in a victim's browser.
- Stored XSS can be more persistent than reflected XSS because the payload remains in the application's stored data.
- User-controlled content should not be inserted into HTML without appropriate context-aware output encoding.
- Input validation and output encoding should be implemented as appropriate defenses against XSS.
- A Content Security Policy (CSP) can provide an additional layer of defense.

## Remediation

The application should:

- Properly encode untrusted data before inserting it into HTML.
- Sanitize HTML when users are intentionally allowed to submit HTML content.
- Use safe DOM APIs when manipulating user-controlled content.
- Avoid inserting untrusted data into dangerous HTML contexts.
- Implement a suitable Content Security Policy as defense in depth.
