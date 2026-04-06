# Lab: SQL injection UNION attack, retrieving data from other tables

## Lab Description
This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you need to combine some of the techniques you learned in previous labs. 

## Goal
The database contains a different table called users, with columns called username and password.

To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the administrator user.


## Steps to Solve

1. Open the lab in your browser.

2. Set up proxy as Burp Suite.

3. In catagories select any one of them and in burp suite HTTP history tab find the relevant request and send it to the Reapeaer.

7. Inject the SQL query there with UNION SELECT NULL  and add NULL over and over until error code is returned to get the number of columns.
8. Then to check the data type of the colomns put that data type in place of NULL and determine them.
9. Finaly send the crafted query: `Gifts'+UNION+SELECT+username,password+FROM+users`
10. This will retrun the result in its respose, open it in the browser and extract the username administrator and its password.
11. Login as administrator with extracted credentials.
12. Lab solved successfully.

## Explanation

- By using priviously learned techniques of determing number of colomns and their data types futher exploitation can be done.
- The payload `Gifts'+UNION+SELECT+username,password+FROM+users` used to get the data of the table.



