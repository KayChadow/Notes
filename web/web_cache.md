# Web Cache Vulnerabilities
This is about two different type of vulnerabilities that use the web cache as attack surface. The vulnerabilities use the same idea of exploitation, but they have a different goal. These are the topics:
- [Web Cache Deception](#web-cache-deception)
- [Web Cache Poisoning](#web-cache-poisoning)

## Web Caches
A web cache is a system that sits between the origin server and the user. When a client requests a static resource, the request is first directed to the cache. If the cache doesn't contain a copy of the resource (known as a cache miss), the request is forwarded to the origin server, which processes and responds to the request. The response is then sent to the cache before being sent to the user. The cache uses a preconfigured set of rules to determine whether to store the response.

When a request for the same static resource is made in the future, the cache serves the stored copy of the response directly to the user (known as a cache hit). You can see in the response header `X-Cache: ` whether or not you got a fresh response or not.

> Generally only requests that use `GET`, `HEAD` or `OPTIONS` are cachable

![Schematic web cache](../images/Web_cache.png)

### Cache Keys
The cache must decide if there is a cached response that it can serve directly, or if it has to forward the request. The cache does this by generating a `cache key` from elements of the HTTP request. This typically includes the URL path, query parameters, and some important headers. If the cache key is equivalent, it serves the cached response.

### Cache Rules
Cache rules determine what can be cached and for how long. These are some type of rules that can be enforced:
- Static file extension rules: Cache `.css`, `.exe` or `.js` for example.
- Static directory rules: Match URL against a prefix, for example `/static` or `/assests`.
- File name rules: Cache specific files that are universally required for web, such as `robots.txt` and `favicon.ico`.



# Web Cache Deception
Web cache deception exploits cache rules to trick the cache into storing sensitive or private content, which the attacker can then access.


# Web Cache Poisoning
Web cache poisoning manipulates cache keys to inject malicious content into a cached response, which is then served to other users.
