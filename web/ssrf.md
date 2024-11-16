# Server-Side Request Forgery
Allows the attacker to cause the server-side application to make requests to unintended locations. Most likely to internal-only services within the infrastructure. Or the server can be forced to connect to arbitrary external systems. This can all lead to unauthorized actions or access to data.

It exploits trust relationships to escalate an attack.

When requests from the local machine are handled differently than ordinary requests. For example, when:
- Access control might be implemented in front of the application server
- The application allows administrative access without loggin in for disaster recovery purposes
- The administrative interface might listen on a different port number to the main application, and might not be reachable directly by users
- Back-end systems probably have a weaker security, since it is protected by network topology

## Vulnerability & Protections
When the server makes a request, for example for retreiving the store stock data, and it is user-controlled. The attacker can modify the url, so the server makes a request to `localhost` or `127.0.0.1`. 

When the user-controlled URL is completely validated, but contains an **open redirection vulnerability**, you can use this to redirect it to an attacker controlled URL. For example: "/product/nextProduct?currentProductId=6&path=http://evil-user.net"

Attack surface
- Sometimes you don't fully control the URL, but only parts of it. This might limit the attack surface
- Some server-side analytics track and analyze the third-party URL that appear in the [Referer header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer)
- Some data parsers allow the inclusion of URLs for the format. For example XML, read more about [XXE](./xxe.md)

### Blacklist input filter
Sometimes `localhost`, `127.0.0.1`, or `/admin` are blacklisted in user input.

>But this can be circumvented in some cases.
> - Instead of `127.0.0.1`, use `2130706433`, `017700000001`, or `127.1`
> - Use your own domain that resolves to `127.0.0.1` (e.g. `spoofed.burpcollaborator.net`)
> - Obfuscate blocker string using URL encoding or cAsE vArIaTiOn
> - Use a controlled URL, that redirects to the target URL. Try switching from `http:` to `https:` for the target URL

### Whitelist input filter
Sometimes, the filter looks at a host match at the beginning, at the end, or somewhere in between. 

>But there are some ways that are likely to be overlooked
> - Embed credentials in a URL before the hostname using "@": `https://expected-host:fakepassword@evil-host`
> - Use "#" to indicate a URL fragment: `https://evil-host#expected-host`
> - Leverage the DNS naming hierarchy: `https://expected-host.evil-host`
> - Try URL encoding characters. Sometimes the filter handles it differently than the back-end

## Blind SSRF
When the application issues a back-end request, but no response is returned to the front-end application.

You can use Out-of-band Application Security Testing (OAST) to try to trigger a request to a system that you control. You can blindly sweep internal IP addresses, and send payloads that also use OAST techniques. Or return malicious HTTP responses to the target server.