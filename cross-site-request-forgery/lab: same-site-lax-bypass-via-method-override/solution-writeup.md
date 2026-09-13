# CSRF — SameSite Lax Bypass via Method Override

## Lab Objective

Exploit a CSRF vulnerability in the email-change functionality by bypassing the browser's `SameSite=Lax` cookie restriction using a method-override parameter.

## Vulnerability

The application allows the HTTP method to be overridden using the `_method` parameter.

The legitimate request is:

GET /my-account/change-email?email=...&_method=POST

Although the endpoint normally expects a POST request, the `_method=POST` parameter causes the application to process the request as a POST.

Because the request is initially a top-level GET navigation, the browser can send the victim's `SameSite=Lax` session cookie.

## Exploitation

### 1. Capture the normal email-change request

In Burp Repeater, the normal request is:

GET /my-account/change-email?email=...&_method=POST HTTP/2

The server responds with:

HTTP/2 405 Method Not Allowed
Allow: POST

This indicates that the endpoint itself expects POST, but the lab is designed to demonstrate that the method override can be abused through a cross-site GET request.

### 2. Create the CSRF exploit

On the exploit server, use:

<script>
    document.location = "https://LAB-ID.web-security-academy.net/my-account/change-email?email=attacker@web-security-academy.net&_method=POST";
</script>

Replace `LAB-ID` with the target lab's hostname.

The important part is:

&_method=POST

This makes the application interpret the request as a POST while the browser performs a top-level GET navigation.

### 3. Store the exploit

Click:

Store

Then click:

View exploit

If testing against yourself, use the browser specified by the lab.

### 4. Deliver the exploit

Click:

Deliver exploit to victim

The victim's browser navigates to the malicious URL.

Because the initial request is a top-level GET navigation, the victim's `SameSite=Lax` session cookie can be included.

The application then interprets `_method=POST` and processes the email-change action.

## Why the Attack Works

The protection provided by `SameSite=Lax` is not sufficient here because Lax cookies are generally sent with top-level cross-site GET navigations.

The application then performs a method override:

GET request
    ↓
`_method=POST`
    ↓
Application treats request as POST
    ↓
Authenticated action is performed
    ↓
Victim's email address is changed

The attack therefore combines:

1. A state-changing endpoint.
2. A method-override mechanism.
3. A top-level cross-site GET navigation.
4. `SameSite=Lax` cookies being sent with that navigation.
5. No effective CSRF protection preventing the action.

## Burp Evidence

The relevant request structure is:

GET /my-account/change-email?email=attacker@web-security-academy.net&_method=POST HTTP/2

The important parameter is:

_method=POST

The server-side application interprets the request as a POST even though the browser initiated a GET request.

## Final Exploit

<script>
    document.location = "https://LAB-ID.web-security-academy.net/my-account/change-email?email=attacker@web-security-academy.net&_method=POST";
</script>

## Key Takeaway

`SameSite=Lax` does not completely prevent CSRF.

If an application supports HTTP method overriding, a state-changing POST operation may potentially be reached through a top-level GET navigation. Since `SameSite=Lax` cookies can accompany such navigations, the application's method-override functionality can provide a CSRF bypass.

A secure application should avoid allowing state-changing operations to be triggered through GET requests and should use robust CSRF defenses such as unpredictable CSRF tokens and appropriate SameSite cookie configuration.
