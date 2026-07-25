# Client-side Prototype Pollution in Third-party Libraries

## Objective

Exploit a client-side prototype pollution vulnerability in a third-party library to execute arbitrary JavaScript and steal the administrator's cookie.

---

## Steps

### Step 1: Enable Prototype Pollution Detection

1. Open the lab in Burp's browser.
2. Launch **DOM Invader**.
3. Enable **Prototype pollution** under **Attack types**.
4. Click **Update Canary**.

![Using DOM Invader](using-DOM-invader-for-prototype-pollution.png)

---

### Step 2: Identify Prototype Pollution Sources

1. Open the **DOM Invader** tab in DevTools.
2. Click **Search**.
3. DOM Invader identifies the following prototype pollution sources:
   - `__proto__[property]=value`
   - `constructor[prototype][property]=value`

![Prototype Pollution Sources](DOM-invader-found-interesting-sinks.png)

---

### Step 3: Scan for Exploitable Gadgets

1. Click **Scan for gadgets**.
2. DOM Invader discovers a gadget that reaches a `setTimeout()` sink.
3. Click **Exploit**.

![Scanning Results](DOM-invader-scanning-results.png)

---

### Step 4: Verify Code Execution

DOM Invader automatically generates the following payload:

```
#constructor[prototype][hitCallback]=alert(1)
```

Reloading the page executes the callback and displays an alert, confirming JavaScript execution.

![Alert Execution](exploit-result-throughj-DOM-invader.png)

---

### Step 5: Craft the Exploit

Open the **Exploit Server** and replace the body with:

```html
<script>
location="https://YOUR-LAB-ID.web-security-academy.net/#__proto__[hitCallback]=alert%28document.cookie%29"
</script>
```

Replace `YOUR-LAB-ID` with your own lab URL.

Store the exploit and click **Deliver exploit to victim**.

![Exploit Server](crafting-exploit-for-getting-cookie.png)

---

## Explanation

The vulnerable third-party library copies URL parameters into JavaScript objects without filtering dangerous prototype-related properties.

By polluting the global object prototype, an attacker can inject a new property (`hitCallback`) that is later invoked by an unsafe gadget using `setTimeout()`. Since the gadget executes the polluted property as JavaScript, arbitrary code runs in the victim's browser.

Redirecting the victim to a URL containing the malicious hash causes the prototype pollution to occur automatically. The injected callback executes `alert(document.cookie)`, proving that JavaScript runs in the administrator's session and completing the lab.

---

## Payload Used

```
#constructor[prototype][hitCallback]=alert(1)
```

Exploit payload:

```html
<script>
location="https://YOUR-LAB-ID.web-security-academy.net/#__proto__[hitCallback]=alert%28document.cookie%29"
</script>
```

---

## Result

The exploit executes JavaScript in the administrator's browser via prototype pollution in a third-party library, successfully demonstrating DOM-based XSS and solving the lab.
