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
Nowadays, javascript is also widely used on the server-side of applications. This allows for this vulnerability to also happen on the server.

But searching for server-side prototype pollution is way harder than client-side. Since you have no source code, no devtools, and every change persists until the whole server resets.


# Sources
- Main info (Portswigger): https://portswigger.net/web-security/learning-paths/prototype-pollution