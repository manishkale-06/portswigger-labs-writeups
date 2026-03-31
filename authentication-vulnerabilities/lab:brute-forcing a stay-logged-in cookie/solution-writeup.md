# Lab: Brute-forcing a stay-logged-in cookie

## Overview
This lab allows users to stay logged in even after they close their browser session. The cookie used to provide this functionality is vulnerable to brute-forcing. 

## Objective
Brute-force Carlos's cookie to gain access to his My account page. 
## Tools Used
Burp Suite, Browser(Firefox).

## Vulnerability Explanation
The application generates a stay-logged-in cookie for account based on username and password in unsafe manner.

While login if stay logged in is turned on then application generates a stay-logged-in cookie using username and password which can be brute-forced.

## Steps to Reproduce

1. Set up Burp Suite to proxy, by which HTTP history can be captured.
2. Open the login page.
3. Enter valid username and password also tick the stay logged in box. 
4. Select the login request in burpsuite and notice the stay-logged-in cookie.
5. Send that request to intruder and select the cookie and base64 decode it which will show that cookie contains username and some random text like this 'wiener:51dc30ddc473d43a6011e9ebba6ca770'.
6. Now through trial and error figure out taht that number is password of the user which is hashed in MD5.
7. Now logout of that account.
8. Go to intruder change the username in request to carlos in both cookie and id and add tag to remaing.
9. Paste the provided payload in payload section then in payload processing add hash:MD5, prefix:carlos, encode: base63-encode respectively.
10. In settings go to grep-match section and add a grep to match 'Update Email' as it will be only shown if login was successful.
11. Start the attack.
12. Examine the results, only one resonse have 'Update Email' functionality.
13. Open the reponse in browser from right click options.
14. Login successful.

## Payload Used

Valid Username: wiener
Valid Password: peter

Target Username: carlos

Password payload:
123456
password
12345678
qwerty
123456789
12345
1234
111111
1234567
dragon
123123
baseball
abc123
football
monkey
letmein
shadow
master
666666
qwertyuiop
123321
mustang
1234567890
michael
654321
superman
1qaz2wsx
7777777
121212
000000
qazwsx
123qwe
killer
trustno1
jordan
jennifer
zxcvbnm
asdfgh
hunter
buster
soccer
harley
batman
andrew
tigger
sunshine
iloveyou
2000
charlie
robert
thomas
hockey
ranger
daniel
starwars
klaster
112233
george
computer
michelle
jessica
pepper
1111
zxcvbn
555555
11111111
131313
freedom
777777
pass
maggie
159753
aaaaaa
ginger
princess
joshua
cheese
amanda
summer
love
ashley
nicole
chelsea
biteme
matthew
access
yankees
987654321
dallas
austin
thunder
taylor
matrix
mobilemail
mom
monitor
monitoring
montana
moon
moscow

## Proof of Concept

Using Burp Suite intruder stay-logged-in cookie is brute forced with encoding.

Then by comparing responses corresponding cookie is obtained for carlos and then opened in browser.

## Impact
- Attackers can login to the users account bypassing password brute-force control
- Can lead to account takeover

## Mitigation
- Use random parameteres to generate the stay-logged-in cookie
- Use salt for encryption
- Implement better encoding and encryption techniques

## Key Learnings
- Learned how to use payload processing 
- Grasped the concept of cookie usage
- Learned that cookies can be brute-forced as well
