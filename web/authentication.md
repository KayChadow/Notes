# Authentication Vulnerabilities
Authentication consist of three types of factors:
- Something you **know**, password or question answer
- Something you **have**, phone or security token
- Something you **are**, biometrics or behaviour

## Password Brute-Force
You can try brute-forcing a password. But first you need a valid username. That is why you can enumerate for usernames. When the server gives a ever so slightly different response when a username is valid, an attacker now knows what username to use for the password brute-force attack. The different response can be:
- Different response length
    - Different text
    - Typo
- Different response time
    - Server might have to check password or something
- Different response code
    - This will probably not be the case
- JUST ANYTHING

