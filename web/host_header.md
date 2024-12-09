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
5. Try supplying an absolute URL in the request line, and also a host header. This might cause same result as two host headers.
6. Try using `X-Forwarded-Host` header with malicious content, while circumventing validation on normal host header. Or use other similar headers that sometimes do the same: `X-Host`, `X-Forwarded-Server`, `X-HTTP-Host-Override`, `Forwarded`, try using [Param Miner](../burp/param_miner.md) "Guess headers" function.

### Password Reset Poisening
If the process that creates the link that you get in your mail to reset your password uses the contents of the `Host` header. You can poisen it by inputting an attacker-controlled domain as host. Or even try dangling markup attacks if you can inject HTML into the email.