# Detecting NoSQL Injection

## Lab Description

This lab contains a NoSQL injection vulnerability in the product category filter. The objective is to identify the vulnerable parameter and manipulate the query logic to make the application return all products.

## Methodology

### 1. Identify the Vulnerable Parameter

Open the product listing and intercept the request in Burp Suite.

The category filter sends a request similar to:

```http
GET /filter?category=Gifts HTTP/2
```

The `category` parameter is the input tested for NoSQL injection.

### 2. Test for NoSQL Injection

Send the request to Burp Repeater and modify the `category` parameter by adding a NoSQL syntax character:

```text
Gifts'
```

The application returns:

```http
HTTP/2 500 Internal Server Error
```

This indicates that the input is being processed by the backend query and that the added syntax affects the query.

### 3. Confirm NoSQL Injection

Test the parameter with boolean conditions.

True condition:

```text
Gifts' || '1'=='1
```

False condition:

```text
Gifts' || '1'=='2
```

The responses differ depending on whether the injected condition evaluates to true or false. This confirms that the `category` parameter is vulnerable to NoSQL injection.

### 4. Override the Existing Query Condition

Use an always-true condition:

```text
Gifts' || '1'=='1
```

The resulting request is:

```http
GET /filter?category=Gifts'%20%7C%7C%20'1'=='1
```

The injected `||` operator acts as a logical OR.

Conceptually, the query becomes:

```text
category == "Gifts" || "1" == "1"
```

Since `"1" == "1"` is always true, the complete condition evaluates to true.

### 5. Observe the Result

After sending the modified request, the application returns products from all categories instead of only products belonging to the `Gifts` category.

This demonstrates that the original category restriction has been bypassed.

## Payloads Used

```text
Gifts'
```

```text
Gifts' || '1'=='1
```

```text
Gifts' || '1'=='2
```

## Key Concept

NoSQL injection occurs when user-controlled input is inserted into a NoSQL database query without proper validation or sanitization.

In this lab, the vulnerable input is:

```text
category
```

The injected logical expression is:

```text
' || '1'=='1
```

This causes the query condition to always evaluate to true, allowing the intended category restriction to be bypassed.

## Conclusion

The lab was solved by identifying the `category` parameter as vulnerable to NoSQL injection, confirming the vulnerability through boolean tests, and injecting an always-true condition to override the original query condition.

The key lesson is that applications should not directly concatenate untrusted user input into NoSQL queries. Proper input validation, parameterized query construction, and safe query APIs should be used to prevent NoSQL injection.
