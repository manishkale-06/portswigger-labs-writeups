# Lab: Username enumeration via response timing

## Overview
This lab is vulnerable to username enumeration using its response times. To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page. 

## Objective
Enumarate a valid username, brute-force this user's password, then access their account page.

## Tools Used
Burp Suite, Browser(Firefox).

## Vulnerability Explanation
The application takes much more time to respond if username is valid based on size of password.

When a valid username is entered, the response time is much higher compared to invalid usernames if long password is submitted.

## Steps to Reproduce

1. Open the login page.
2. Enter a random username and a long password.
3. Observe the error message.
4. Use Burp Suite to intercept the request.
5. Send the request to Intruder.
6. Use a username wordlist and battle ram mode for bruteforce also set X-Forwarded-For field to same usename for each iteration to bypass the IP-based protection.
7. Identify valid username based on response time comparision.
8. Perform brute-force attack on the valid username's password also use X-Forwarded-For field but with iterating password this time to bypass IP-based protection.
9. Login successfully.

## Payload Used

Username wordlist:
- carlos
- root
- admin
- test
- guest
- info
- adm
- mysql
- user
- administrator
- oracle
- ftp
- pi
- puppet
- ansible
- ec2-user
- vagrant
- azureuser
- academico
- acceso
- access
- accounting
- accounts
- acid
- activestat
- ad
- adam
- adkit
- admin
- administracion
- administrador
- administrator
- administrators
- admins
- ads
- adserver
- adsl
- ae
- af
- affiliate
- affiliates
- afiliados
- ag
- agenda
- agent
- ai
- aix
- ajax
- ak
- akamai
- al
- alabama
- alaska
- albuquerque
- alerts
- alpha
- alterwind
- am
- amarillo
- americas
- an
- anaheim
- analyzer
- announce
- announcements
- antivirus
- ao
- ap
- apache
- apollo
- app
- app01
- app1
- apple
- application
- applications
- apps
- appserver
- aq
- ar
- archie
- arcsight
- argentina
- arizona
- arkansas
- arlington
- as
- as400
- asia
- asterix
- at
- athena
- atlanta
- atlas
- att
- au
- auction
- austin
- auth
- auto
- autodiscover

Password wordlist:
- 123456
- password
- 12345678
- qwerty
- 123456789
- 12345
- 1234
- 111111
- 1234567
- dragon
- 123123
- baseball
- abc123
- football
- monkey
- letmein
- shadow
- master
- 666666
- qwertyuiop
- 123321
- mustang
- 1234567890
- michael
- 654321
- superman
- 1qaz2wsx
- 7777777
- 121212
- 000000
- qazwsx
- 123qwe
- killer
- trustno1
- jordan
- jennifer
- zxcvbnm
- asdfgh
- hunter
- buster
- soccer
- harley
- batman
- andrew
- tigger
- sunshine
- iloveyou
- 2000
- charlie
- robert
- thomas
- hockey
- ranger
- daniel
- starwars
- klaster
- 112233
- george
- computer
- michelle
- jessica
- pepper
- 1111
- zxcvbn
- 555555
- 11111111
- 131313
- freedom
- 777777
- pass
- maggie
- 159753
- aaaaaa
- ginger
- princess
- joshua
- cheese
- amanda
- summer
- love
- ashley
- nicole
- chelsea
- biteme
- matthew
- access
- yankees
- 987654321
- dallas
- austin
- thunder
- taylor
- matrix
- mobilemail
- mom
- monitor
- monitoring
- montana
- moon
- moscow

## Proof of Concept

Using Burp Suite Intruder, a valid username was identified based on response time differences.

X-Forwarded-For feild is used to bypass the IP-based-brute-force protection.

After performing a brute-force attack on the password, the correct credentials were found and access to the user account was successfully obtained.

## Impact
- Attackers can identify valid usernames by observing respose time
- Uneffectiveness of IP-based-brute-force protection
- Enables targeted brute-force attacks
- Can lead to account takeover

## Mitigation
- Use other protection methods that are reliable
- Disable X-Forwarded-For field in HTTP header
- Implement rate limiting and account lockout

## Key Learnings
- Learned how response time differences can lead to username enumeration
- Understood how to use Burp Suite Intruder for brute-force attacks by manipulating HTTP request
- Learned about X-Forwarded-For field
- Took insights how some protection measures can be bypassed easily
