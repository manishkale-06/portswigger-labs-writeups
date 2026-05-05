# Lab: File Path Traversal, Simple case

## Overview
This lab is vulnerable to a file path traversal attack. The application fails to properly sanitize user input in the `filename` parameter, allowing attackers to access sensitive files on the server.

## Objective
Exploit the path traversal vulnerability to retrieve the contents of the `/etc/passwd` file.

## Tools Used
- Burp Suite (Proxy, HTTP History, Repeater)
- Browser (Firefox)

## Vulnerability Explanation
The application loads images using a URL like:


/image?filename=58.jpg


The `filename` parameter is directly used by the server without proper validation.

By manipulating this parameter with directory traversal sequences (`../`), an attacker can escape the intended directory and access sensitive system files.

Example payload:


../../../etc/passwd


This allows reading files outside the web root.

## Steps to Reproduce

1. Open the lab in the browser.
2. Intercept traffic using Burp Suite (Proxy ON).
3. Navigate to any product page (image loads automatically).
4. Go to **HTTP History** in Burp.
5. Find request similar to:

GET /image?filename=58.jpg

6. Right-click → **Send to Repeater**.
7. Modify the request parameter:


GET /image?filename=../../../etc/passwd


8. Click **Send**.
9. Observe the response.

## Proof of Concept

The server responds with:


root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...


This confirms successful retrieval of the `/etc/passwd` file, proving the path traversal vulnerability.

## Payload Used


../../../etc/passwd


(You can also try deeper traversal if needed:)


../../../../../../etc/passwd


## Impact

- Unauthorized access to sensitive system files
- Information disclosure (usernames, system structure)
- Can lead to further exploitation (e.g., privilege escalation)

## Mitigation

- Validate and sanitize user input
- Use allowlists for file names
- Restrict file access to specific directories
- Normalize paths before processing
- Use secure APIs instead of direct file access

## Key Learnings

- Learned how path traversal works using `../`
- Understood how to manipulate HTTP requests in Burp Repeater
- Saw how improper input validation leads to serious vulnerabilities
- Gained hands-on experience extracting sensitive files from a server
