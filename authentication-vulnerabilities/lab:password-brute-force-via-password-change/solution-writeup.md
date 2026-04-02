# Brute Force Password Change Lab

## Objective
Find Carlos's password and access his account.

---

## Given
- Username: wiener  
- Password: peter  
- Victim: carlos  

---

## Steps

1. Login with:
   - wiener / peter

2. Go to **Change Password**

3. Intercept request in Burp Suite

4. Observe behavior:
   - Wrong current password → "Current password is incorrect"
   - Correct current password + different new passwords → "New passwords do not match"

Use this to find correct password

---

## Attack

1. Send request to **Burp Intruder**

2. Modify request:

username=carlos&current-password=§pass§&new-password-1=123&new-password-2=abc


3. Add password list in payloads

4. Add Grep Match:

New passwords do not match


5. Start attack

---

## Result

- Find password that gives:

New passwords do not match


- Login as:
  - carlos / found password

- Open **My Account**

---

## Conclusion
Different error messages help to brute-force the password.
