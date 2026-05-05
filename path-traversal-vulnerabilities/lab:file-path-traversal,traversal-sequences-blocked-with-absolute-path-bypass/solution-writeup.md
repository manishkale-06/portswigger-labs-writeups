# Lab: File Path Traversal, traversal sequences blocked with absolute path bypass

## Overview
This lab is vulnerable to a file path traversal attack where traversal sequences are blocked, but the application fails to prevent access using absolute paths.

## Objective
Bypass the input validation and retrieve the contents of the `/etc/passwd` file using an absolute path.

## Tools Used
- Burp Suite (Proxy, HTTP History, Repeater)
- Browser (Firefox)

## Vulnerability Explanation
The application loads images using a request like:


/image?filename=58.jpg


The server attempts to block directory traversal sequences such as:


../../../etc/passwd


However, it does **not block absolute paths**, allowing direct access to sensitive files.

Instead of traversal, we use:


/etc/passwd


This bypasses the filter and exposes system files.

## Steps to Reproduce

1. Open the lab in the browser.
2. Enable Burp Suite Proxy and intercept requests.
3. Navigate to a product page.
4. Go to **HTTP History** in Burp Suite.
5. Locate request:

GET /image?filename=58.jpg

6. Right-click → **Send to Repeater**.
7. Modify the request parameter:


GET /image?filename=/etc/passwd


8. Click **Send**.
9. Observe the response.

## Proof of Concept

The response contains:


root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...


This confirms that the application is vulnerable and sensitive system files can be accessed.

## Payload Used


/etc/passwd


## Impact

- Direct access to sensitive system files
- Information disclosure (usernames, services)
- Potential for further exploitation

## Mitigation

- Restrict file access to specific directories
- Reject absolute paths in user input
- Normalize and validate file paths
- Use allowlists for valid filenames
- Implement proper access controls

## Key Learnings

- Learned how absolute paths can bypass weak filters
- Understood difference between traversal (`../`) and absolute path (`/`)
- Practiced modifying requests using Burp Repeater
- Observed how improper validation leads to data exposure
