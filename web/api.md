# API

## Recon
Find out as much info about the API as possible. Identify endpoints, input data, request types, and authentication mechanisms. For endpoints, also investigate the base path of it. Javascript files might also contain references to API endpoints.

If there is a public documentation, review it to understand it. When it is not public, you can try to fuzz for endpoints (maybe use [Param miner](https://portswigger.net/bappstore/17d2949a985c4b7ca092728dba871943)), and try different [request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods). Use a tool like [OpenAPI Parser](https://portswigger.net/bappstore/6bf7574b632847faaaa4eb5e42f1757c) to parse the OpenAPI documentation.

Interact with the API, and observe what it does. Try changing the content type, and switch between XML and JSON.

## Vulnerabilities

### Mass Assignment
When the server creates paramenters from object fields. When you input more parameters in the API request, is can assign those extra variables to the object. For example, try adding an "isAdmin" field to the request object.

```json
{
    "username": "foo",
    "email": "bar@mail.com",
    "isAdmin": true,
}
```

This might give you admin permissions.

### Server-side parameter pollution
Internal APIs that aren't directly accessable might not use good encoding. This sometimes allows the attack to
- Override existing parameters
- Modify the application behavior
- Access unauthorized data

This vulnerability can occur through any kind of input like, query parameters, form fields, headers, and URL path parameters. 

Try inputting characters like `#`, `&`, `=`, [path traversal](./path_traversal.md) sequences `../`, and all of these URL encoded. 

> When you try to input two parameters with the `&` sign, this is how different applications parse it
> - PHP parses the last parameter only
> - ASP.NET combines both parameters, resulting in `foo,bar`
> - Node.js / express parses the first parameter only

Try adding extra parameters with some values to input used in XML or JSON. Use correct format for XML/JSON, also try escaping the qoutes `\"`.

You can also use the [Backslash Powered Scanner BApp](https://portswigger.net/bappstore/9cff8c55432a45808432e26dbb2b41d8) to identify server-side injection vulnerabilities.

# Sources
- Main info (Portswigger): https://portswigger.net/web-security/learning-paths/api-testing
