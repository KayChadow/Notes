# Race Conditions
Unintended things that happens when (similar) requests are processed at (about) the same time (e.g. try to apply a discount code twice at the sime time, to get double discount). 

> ### **!**
>Some frameworks attempt to prevent accidental data corruption by using some form of request locking. For example, PHP's native session handler module only processes one request per session at a time.
>
>If you notice that all of your requests are being processed sequentially, try sending each of them using a different session token.
> - Easily get a new session token by changing your session cookie to some invalid token "aaa", and refresh the browser

## TOCTOU
Time-Of-Check to Time-Of-Use vulnerability, when the time of check is before the time of use in the application. :bird: 

## Single-Endpoint
Sending two requests with different values to a single endpoint can cause race conditions. For example, a password reset endpoint bound to the user session. If they overwrite some session variables at the same time, it can cause user A to get the reset token of account B. 

Email based operations are a good target for these type of race conditions, since emails are often sent in a background thread after the server response.

## Hidden Multi-Step Sequences
When a single request might trigger multiple hidden states within the application. These are sub-states. For example, flawed multi-factor authentication (MFA) workflows that let you perform the first part of the login using known credentials, then navigate straight to the application via forced browsing, effectively bypassing MFA entirely.

![Methodology_hidden_multi-step_seq](../images/Methodology_hidden_multi-step_seq.png)

### Predict
Ask yourself,
- Is this endpoint security critical?
- Is there any collision potential?
And predict if there might be a vulnerability.

### Probe
1. Research normal workflow and responses
2. Send group of requests at the same time
3. Look for ANY clues, changes in responses, visible changes in the application

### Prove
Try to understand what's happening, remove superfluous requests, and make sure you can still replicate the effects.
Advanced race conditions can cause unusual and unique primitives, so the path to maximum impact isn't always immediately obvious.

## Time-Sensitive Attacks

### Partial construction race conditions
When an object is not constructed in one go. For example, first is the object created with a few values, and after, api-keys get generated and assigned to the object. Now in this short window, there is no api key, so you can try parsing parameters like `/?api-key[]=` without a value. When succesful within this short window, it should allow you to make authenticated api requests. 

### Timestamp usage
When high-resolution timestamps are used instead of cryptographically secure random strings to generate security tokens, it might be vulnerable to an attack. If you just trigger two password resets for two different users so they use the same timestamp, then you have one valid token for both users. 

## Timing Problems & Solutions
Sometimes the race windows are not well aligned,\
    and nothing is working, \
    and you don't know what to do, \
    and you tested it for so long, \
    and it seems like it is not vulnerable, \
    but it is just a race window problem.

### Network architecture delays
For example, there may be a delay whenever the front-end server establishes a new connection to the back-end. The protocol used can also have a major impact.

### Endpoint-specific processing delays
Different endpoints inherently vary in their processing times, sometimes significantly so, depending on what operations they trigger.

> ### Warming Connection
> Back-end connection usually delays ore often queal for all parallel requests. Distinguish this type of delay by "warming" the connection with one or more inconsequential requests to see if this smoothes out the remaining processing times.
> - Try adding a `GET` request to the homepage at the start of the group of requests, and send them in sequence
> - If the first request is longer, but the rest is kinda the same, it is fine, just ignore
> - If it is still way longer for a single endpoint, it probably is the back-end. In Turbo intruder, try sending more warning requests before the attack

> ### Abusing Rate/Resource Limits
> If connection warming doesn't work. You can try splitting the attack requests in multiple TCP packets with a delay, but on high-jitter targets, this probably won't work
> So, webservers often delay the processing of requests if too many are sent too quickly.
> - So try sending a large number of dummy requests to trigger the resource limit, this might cause a nice server-side delay. This is a viable single-packet attack strategy with delayed execution