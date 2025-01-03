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

Just use DOM Invader, which is preinstalled in Burps browser. It automatically tests for prototype pollution soources when you browse.

### Gadget
> I was going to write a step by step instruction on how to find prototype pollution gadgets here. But then I saw that DOM Invader can also do this automatically. This saves a lot of time. If you still want to read how to do it manually, this is the [link](https://portswigger.net/web-security/learning-paths/prototype-pollution/client-side-prototype-pollution/prototype-pollution/client-side/finding-client-side-prototype-pollution-gadgets-manually).

### Sink




# Sources
- Main info (Portswigger): https://portswigger.net/web-security/learning-paths/prototype-pollution