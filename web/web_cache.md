# Web Cache Vulnerabilities
This is about two different type of vulnerabilities that use the web cache as attack surface. The vulnerabilities use the same idea of exploitation, but they have a different goal. These are the topics:
- [Web Cache Deception](#web-cache-deception)
- [Web Cache Poisoning](#web-cache-poisoning)

## Web Caches
A web cache is a system that sits between the origin server and the user. When a client requests a static resource, the request is first directed to the cache. If the cache doesn't contain a copy of the resource (known as a cache miss), the request is forwarded to the origin server, which processes and responds to the request. The response is then sent to the cache before being sent to the user. The cache uses a preconfigured set of rules to determine whether to store the response.

When a request for the same static resource is made in the future, the cache serves the stored copy of the response directly to the user (known as a cache hit). You can see in the response header `X-Cache` whether or not you got a fresh response or not. The [`Cache-Control`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) header decides/shows the "settings" of the cache kinda.

> Generally only requests that use `GET`, `HEAD` or `OPTIONS` are cachable

![Schematic web cache](../images/Web_cache.png)

### Cache Keys
The cache must decide if there is a cached response that it can serve directly, or if it has to forward the request. The cache does this by generating a `cache key` from elements of the HTTP request. This typically includes the URL path, query parameters, and some important headers. If the cache key is equivalent, it serves the cached response. You can try to get the cache key in the response with headers like `Pragma: x-get-cache-key` or something, but is probably not allowed by server.

When testing for web cache vulnerabilities, you generally don't want to receive cached responses. That is why you should use a dynamic cachebuster. This is already implemented in the [Param Miner Extension](../burp/param_miner.md#dynamic-cachebuster). This makes your life way easier.

You can use the `Vary` request header to add certain headers to the cache key. This can be used to add the `Vary: User-Agent` header to target specific users.

### Cache Rules
Cache rules determine what can be cached and for how long. These are some type of rules that can be enforced:
- Static file extension rules: Cache `.css`, `.exe` or `.js` for example.
- Static directory rules: Match URL against a prefix, for example `/static` or `/assests`.
- File name rules: Cache specific files that are universally required for web, such as `robots.txt` and `favicon.ico`.



# Web Cache Deception
Web cache deception exploits cache rules to trick the cache into storing sensitive or private content, which the attacker can then access.

## Exploiting Cache Rules

### Path Mapping
Difference between traditional and RESTful path mappings. Cache might see it as a path to a static file and cache it, while the server might just ignore the last part and retrieves the user profile.
```url
http://example.com/user/123/profile/wcd.css
```

### Delimiters
Generally, `?` is used to separate URL path from the query string. This one is probably not usable for exploits, but here are some that might be useful:

- In Java Spring `;` is used to add matrix variables, so it truncates the path.
- Ruby on Rails uses `.` as a delimiter to specify the response format. So `.ico`, which isn't recognized by Ruby on Rails, will be handled by the default HTML formatter. And maybe the cache stores `.ico` requests.
- OpenLiteSpeed server uses `%00` as a delimiter, and if the cache uses Akamai or Fastly, they would iterpret everything as path.

Here are some more delimiters that might prove helpful, provided by Portswigger: [delimiter list](https://portswigger.net/web-security/web-cache-deception/wcd-lab-delimiter-list).

Sometimes there are also some discrepancies in the handling of encoded delimiters. That is why you should also try the encoded version of these delimiters, included non-printable ones like `%00`, `%0A`, `%0D` and `%09` (or just all of them lmao).

### Path Normalization
When the cache always caches files in the directories like `/static`, you can try [path traversal attacks](./path_traversal.md). This might allow you to cache files that aren't in the static directory.

You can also try it in reverse with static files like `/index.html` using a payload like `/profile%2f%2e%2e%2findex.html`.


# Web Cache Poisoning
Web cache poisoning manipulates cache keys to inject malicious content into a cached response, which is then served to other users.

Web cache poisoning involves three steps:
- Identify and evaluate unkeyed inputs
- Elicit a harmful response from the back-end server
- Get the response cached

### Identify and Evaluate Unkeyed Inputs
You can try adding headers, query strings, or other things. To try and make the server return a different response. Luckily, there exists [Param Miner](../burp/param_miner.md). This extension can "guess headers" automatically for you, and it lists them if there is a difference in response.

## Exploiting Cache Flaws
> DoS : Denial of Service\
> XSS : Cross-Site Scripting\
> AUR : Arbitrary URL Redirect

### Internal Route Header Attack
>DoS ; ;

CDN's implement some special headers to pass routing information during internal transmission. You can exploit these headers to trigger a an exception by the CDN, leading to WCP. These are some of the headers: `Fastly-Client-Ip`, `Fastly-Soc-X-Request-Id`, `X-Amz-WebsiteRedirect-Location`, `X-Amzn-CDN-Cache`.

### Identify Header Attack
>DoS ; ;

When gateway systems support HTTP authentication, you can inject illegal header values to make the server return an denial of access response. And then the cache incorrectly stores it. Headers used for this attack are: `Authorization`, `X-Auth-User`, `Auth-Key`.

### Protocol Header Attack
>DoS ; AUR ;

Headers like `X-Forwarded-SSL`, `X-Forwarded-Scheme`, `X-Forwarded-Proto`, `X-Forwarded-Protocol` are used to identify client connection protocols. But some web servers respond with a 301 redirect if the protocol is not cool enough. If the redirect retains the headers, it causes a DoS attack. In the scenario that the redirect is cached, you can try using `X-Forwarded-Host` to control the redirect URL.

### HTTP Range Header Attack
>DoS ; ;

The [`Range`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Range) header is used to specify a portion of a resource, for tasks like multi-threaded downloads. When the server doesn't support this, or a malformed value is injected, it might cause errors.

### HTTP If Header Attack
>DoS ; ;

HTTP standard headers like `If-Match`, `If-Range`, `If-Modified-since` determine if a web server meets specified conditions. However, some web servers generate 4xx or 5xx errors when processing these requests. If this is then cached, kab00m! WCP.

### HTTP Upgrade Header Attack
>DoS ; ;

The `Upgrade` header allows upgrading a connection. But when upgrading it to an unsupported protocol: `Upgrade: HTTP/3.0`. Or to a malformed one: `Upgrade: HTTP/0.9`. Web servers might return incorrect status codes, that potentially get cached.

### HTTP Coding Header Attack
>DoS ; ;

Headers like `Accept`, `Accept-Encoding`, `Transfer-Encoding` are used to identify encoding formats. If these are malformed or illegal, it might cause exceptions at the web server.

### HTTP Header Oversize Attack
>DoS ; ;

The HTTP protocol standard does not impose a limit on the length of the request header. Therefore, different servers might set restrictions themselves. If you send a request with a length bigger than the web server, and smaller than the cache server. The web server will return an error, which will get cached by the cache server.

### HTTP Method Override Attack
>DoS ; ;

Some web frameworks support helper headers like `X-HTTP-Method-Override`. So when you send a GET request overridden with DELETE, and the web server doesn't support DELETE, it will generate an error. While the cache will save it since it saw it as a GET request.

### HTTP Meta Character Attack
>DoS ; ;

Metacharacters sometimes trigger error pages when the web server processes the request. These metacharacters involve all control characters, including `\r`, `\n`, or any other. 

### Fat GET Attack
> ; XSS ;

Cache servers usually cache a GET request, and exclude the body of a GET request from a cache key. Despite the HTTP standard of prohibiting GET requests from having a body, some web applications parse fat GET request bodies, allowing dynamic responses. You can even try combining it with `X-HTTP-Method-Override` header.

### HTTP Parameter Attack
> ; XSS ;

When you found an unkeyed input, you can try and use it to get a harmful response. For example, when the `X-Forwarded-Host` header is used to dynamically generate urls/links/src on the page, you can maybe try [xss](./xss.md) or injecting your malicious domain into it. Even cookies are sometimes not used in the cache key! But cookie bugs get fixed quite fast most of the time, since they are triggered without intent.

### HTTP Forwarded Header Attack
>DoS ; XSS ; AUR

Reverse proxies rely on routing host information to determine the web server for fetching web resources. Commonly, header like `Host`, `X-Forwarded-Host`, `X-Forwarded-Port`, `Forwarded` are used to identify original routing host. This can be exploited to let the cache server read malicious data, or to let the web server reject responses. As these headers are not part of the cache key.

### Blacklist Attack
>DoS ; ;

Inconsistencies in the blacklist support between the cache and the web server. This might cause the web server to throw an error, while the cache might just cache it. This can be done by adding random payloads like `<script>alert(1)</script>` in certain headers, or adding a `Referer` header with a phishing site domain name. Or `User-Agent` values with web crawlers or anything, be creative with it. 







# Sources
- Main info (Portswigger): https://portswigger.net/web-security/web-cache-poisoning
- Internet's Invisible Enemy: Detecting and Measuring Web Cache Poisoning in the Wild: https://www.jianjunchen.com/p/web-cache-posioning.CCS24.pdf
- HCache (tool): https://github.com/phantomnothingness/HCache/tree/main