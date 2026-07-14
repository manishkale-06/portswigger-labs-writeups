# Lab: SSRF with Blacklist-Based Input Filter

## Overview

This lab demonstrates a **Server-Side Request Forgery (SSRF)** vulnerability where the application attempts to block SSRF attacks using a blacklist-based filter. The filter blocks requests containing `localhost` and `127.0.0.1`, but it can be bypassed using alternative representations of the loopback address and URL encoding.

The objective is to bypass the filter, access the internal admin interface, and delete the user **carlos**.

---

## Objective

Exploit the SSRF vulnerability to:

- Bypass the blacklist-based filter.
- Access the internal admin interface.
- Delete the user **carlos**.
- Solve the lab.

---

## Tools Used

- Burp Suite Community Edition
- Burp Repeater
- Firefox
- PortSwigger Web Security Academy

---

## Vulnerability Explanation

The application retrieves stock information from the URL supplied in the `stockApi` parameter.

Original request:

```http
POST /product/stock HTTP/2

stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```

To prevent SSRF, the application blocks requests containing:

- `localhost`
- `127.0.0.1`

It also blocks access to the `/admin` path.

However, the blacklist can be bypassed by:

- Using the shortened loopback address:

```text
127.1
```

- URL encoding characters in the path:

```text
%2561
```

which is interpreted as the letter `a` after double decoding.

---

## Steps to Reproduce

### Step 1: Capture the Stock Check Request

1. Open any product.
2. Click **Check stock**.
3. Intercept the request in Burp Suite.

Locate the parameter:

```text
stockApi=http://stock.weliketoshop.net:8080/...
```

---

### Step 2: Bypass the Blacklist

Replace the value of the `stockApi` parameter with:

```text
http://127.1/%2561dmin
```

Forward the request.

The blacklist is bypassed, and the response redirects to the internal admin interface.

---

### Step 3: Identify the Delete Endpoint

Inspect the admin page and locate the delete endpoint:

```text
/admin/delete?username=carlos
```

---

### Step 4: Delete the User

Modify the request once more:

```text
http://127.1/%2561dmin/delete?username=carlos
```

Send the request.

The server accesses the internal endpoint and deletes the user **carlos**.

---

### Step 5: Verify the Solution

Return to the lab page.

The lab is marked as solved after the user **carlos** has been deleted.

---

## Payloads Used

Access the admin panel:

```text
http://127.1/%2561dmin
```

Delete Carlos:

```text
http://127.1/%2561dmin/delete?username=carlos
```

---

## Proof of Concept

1. Intercepted the stock check request.
2. Replaced `localhost` with the shortened loopback address `127.1`.
3. Bypassed the `/admin` blacklist by URL encoding the letter `a` as `%2561`.
4. Successfully accessed the internal admin interface.
5. Identified the delete endpoint for the user `carlos`.
6. Sent another SSRF request to the delete endpoint.
7. Successfully deleted the user and solved the lab.

---

## Impact

Successful exploitation can allow an attacker to:

- Bypass blacklist-based security controls.
- Access internal services that are not publicly exposed.
- Perform unauthorized administrative actions.
- Access sensitive internal resources.
- Demonstrate the weakness of blacklist-only input validation.

---

## Mitigation

- Use an allowlist instead of a blacklist for permitted destinations.
- Block requests to loopback and private IP ranges.
- Normalize and fully decode URLs before validation.
- Validate both the hostname and resolved IP address.
- Restrict outbound network connections using firewall rules.
- Separate internal administrative services from user-controlled functionality.

---

## Key Learnings

- Learned why blacklist-based input validation is ineffective against SSRF.
- Practiced bypassing filters using alternative loopback address formats.
- Understood how double URL encoding can evade path-based filters.
- Learned how attackers access protected internal resources through SSRF.
- Gained hands-on experience exploiting SSRF filter bypasses in a controlled environment.
