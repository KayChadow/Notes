# Host Header Attacks
You know right, the `Host` header in requests. Sometimes stuff is misconfigured. A server sometimes takes an L. \
Host header is used in HTTP/1.1 to know which back-end server to send to.

Host Header attacks are most of the time just gateways to other vulnerabilities. It is not really a thing on its own. It is an entrypoint for exploits.

Common attacks:
- Password reset poisoning


## Testing & Exploitation
Inconsistant implementations are the main cause of this vulnerability. Lol. People are stupid.

1. Try sending arbitrary hostname, will it still reach the correct back-end server? Might be because of a default fallback.
2. Try sending non numeric port after the `:` in the host header. Sometimes validation is flawed.
3. Sometimes only the tail end is validated with regards to the main domain name. Try sending stuff in front of the domain name.
4. Try sending two `Host` headers. Or try obfuscating one (or both), with space character(s) in front to make it seem like a wrapped line.
5. Try supplying an absolute URL (`https://site.com/`) in the request line, and also a host header. This might cause same effect as two host headers.
6. Try using `X-Forwarded-Host` header with malicious content, while circumventing validation on normal host header. Or use other similar headers that sometimes do the same: `X-Host`, `X-Forwarded-Server`, `X-HTTP-Host-Override`, `Forwarded`, try using [Param Miner](../burp/param_miner.md) "Guess headers" function.

### Password Reset Poisoning
If the process that creates the link that you get in your mail to reset your password uses the contents of the `Host` header. You can poison it by inputting an attacker-controlled domain as host. Or even try dangling markup attacks if you can inject HTML into the email.

### Web Cache Poisoning
When the response reflects the host header but you cannot really find a direct exploit. It might be possible to poison the web cache. Try constructing an attack that preserves the cache key, but still reflects a malicious host header value into the response. This can be powerfull.

### Server-Side Vulnerabilities | Authentication Bypass
Every header is a potetial vector for any server side vulnerability. So try stuff with the host header as well. Using `Host: localhost` might bypass some authentication mechanisms.

### Routing-Based SSRF | Internal Virtual Hosts
When public websites and internal private websites are hosted on the same server, anyone can access the private virtual host by just guessing the hostname. This can be done through brute forcing it. Sometimes there is a DNS record associated with it.

When some load balancer or reverse proxy or something looks at the host header and routes it to that, it can be cool to route it to an internal ip. This way you can sometimes access internal websites. 

Also try malforming the path from the request line. For example `@intranet/admin`, when a badly implemented reverse proxy just concatinates this with a standard `https://backend`, it results in a connection to `intranet/admin` with username `backend`

### Connection State Attack
Some crappy implemented HTTP servers that reuse connections for requests and responses might assume that all requests over the same connection use the same host header (because it is true for normal browsers). So they will only check the host header of the first request. To exploit, just send an innocent request and follow it up with a malicious request.

This can also just be used as an attack vector for other type of attacks.

# Sources
- Main info (Portswigger): https://portswigger.net/web-security/host-header 