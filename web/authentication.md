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

### Attack
- Brute force, use logic
    - Automatically try a bunch of passwords
- Username enumeration
    - Find valid usernames through diffence in response
- Credential stuffing
    - Try a bunch compromised username:password pairs (from data breaches)

### Defense
- IP blocking
    - Use `X-Forwarded-For` header to spoof IP
    - Login to known account every few brute force iterations
- Account locking
    - Free (real estate!) confirmation that a username is valid
    - Try a few passwords on a lot of candidate usernames
- HTTP basic authentication is not really secure
    - It is just this header `Authorization: Basic base64(username:password)`

## Multi-Factor Authentication
Only real 2FA if it is two **different** factors, not two times the same factor. So email verification for passwords is kinda stupid.

### Attack
- SIM swapping
    - Just social engineer telephone provider into giving you the targets phone number (rizz for hackers)
- Bypass if badly implemented MFA
    - If the user is first prompted to enter a password, and then prompted to enter a verification code on a separate page, the user is effectively in a "logged in" state before they have entered the verification code.
    - Go to verify page, change the username cookie to another username. And try brute forcing the verification code.