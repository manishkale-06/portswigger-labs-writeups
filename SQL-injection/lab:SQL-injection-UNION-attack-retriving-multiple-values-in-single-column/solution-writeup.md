# Lab: SQL injection UNION attack, retrieving multiple values in single column

## Lab Description
This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables.

## Goal
The database contains a table called `users` with columns `username` and `password`.

To solve the lab, perform a SQL injection UNION attack to retrieve all usernames and passwords, and use the information to log in as the administrator user.

## Steps to Solve

1. Open the lab in your browser.

2. Configure Burp Suite as a proxy and turn interception ON.

3. Select any product category and capture the request in Burp Suite.

4. Send the request to **Repeater**.

5. Determine the number of columns using:

'+UNION+SELECT+NULL,NULL--


6. Identify which column supports text using:

'+UNION+SELECT+NULL,'abc'--

(This confirms only one column supports text.)

7. Extract usernames and passwords using:

'+UNION+SELECT+NULL,username||'~'||password+FROM+users--


8. Observe the response and locate the output containing:

username~password


9. Find the credentials for the **administrator** user.

10. Go to the login page and log in using the extracted administrator credentials.

11. Lab solved successfully.

## Explanation

- The UNION attack is used to combine the original query with another query to extract data.
- `NULL` is used to match the number of columns.
- `username||'~'||password` concatenates both fields into one column for display.
- The response reveals sensitive data directly, allowing account takeover.
