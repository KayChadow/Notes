# Path Traversal
Read arbitrary files on the server that is running the application by pathing to the desired file.

[Canonicalized paths](https://en.wikipedia.org/wiki/Canonicalization#Filenames) prevent these types of attacks. But maybe combined with other vulnerabilities, path traversal might prove helpful when exploring new attack surfaces that were not thought of by the developer.

## Vulnerability & Protections
Use the traversal sequence `../` to go a directory up, traverse the file system to any file, and read the file using the malicious path/filename input. An exploit might look like this `https://insecure-website.com/loadImage?filename=../../../etc/passwd`, this will load the content of `/etc/passwd` as the image.

Against Windows, both `../` and `..\` are valid directory traversal sequences. 

Sometimes you can inject an absolute path from the system root (`/`), instead of a relative path, to exploit the system without using traversal sequences.

### Sanitized input
Some filters strip path traversal sequences from the user input. Or it checks if the path starts/ends the correct way.

>You can try several ways of bypassing this.
> - Use nested traversal sequences `....//` or `....\/`, when the center is cut out, a traversal sequence still exists
> - URL encode the characters (twice), you end up with `%2e%2e%2f`, or `%252e%252e%252f`
> - Might be possible to terminate the file path with `%00` before the requered extension

