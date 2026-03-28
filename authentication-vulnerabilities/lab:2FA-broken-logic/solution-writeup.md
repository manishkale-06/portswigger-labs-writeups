# Lab: 2FA broken logic

## Overview
The lab's two-factor authentication is vulnerable due to its flawed logic.

## Objective
Access Carlos's account page.

## Tools Used
Burp Suite, Browser(Firefox).

## Vulnerability Explanation
The application generates authentication code for account based on verify field in request alone.

When that field is modified to another user then authentication code is generated for that user and one can login as that user by submitting that code to the application. This results in login without the need to know the password of other user.
## Steps to Reproduce

1. Set up Burp Suite to proxy, by which HTTP history can be captured.
2. Open the login page.
3. Enter valid username and password.
4. Go to Email Client to get the authentication code, then use it to log in as valid user.
5. Go to the Burp Suite HTTP history tab, find /login2 and send it to burp repeater.
6. Now in burp repeater change the verify value to target username and send the request, this will create an authentication code for target user.
7. Now logout and again login to valid username and password but on 2FA page enter invalid code.
8. Go to the HTTP history tab and find the POST request of invalid code and send it to Burp intuder.
9. In Burp intruder add $$ to the authentication code and and change verify field to target user, also select the payload type as brute forcer and set 4 in min and max length and also change the charater set to 0-9.
10. Start the attack and sort by respose code, 302 response will give the correct code.
11. Enter the code in 2FA page.
!2. Login Successfully.

## Payload Used

Valid Username: wiener
Valid Password: peter

Target Username: carlos

## Proof of Concept

Using Burp Suite 2FA code is generated for the target username.

Then by modifying 2FA POST bruteforce attack is executed on the authentication code to login as targer user.

## Impact
- Attackers can misuse 2FA to login to another users account
- Enables attackers to login without the need of burteforcing the password
- Can lead to account takeover

## Mitigation
- Use better 2FA logic
- Avoid using verify field for validating user accross stages 
- Implement uniqe cookie mechanism

## Key Learnings
- Learned how 2FA logic flaws can lead to serious problems
- Understood it is possible to login as other user based on only 2FA code if it is not implemeted correctly
