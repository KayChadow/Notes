# Web

*\<protocol>://\<username>:\<password>@\<subdomain>.\<hostname>:\<port>/\<page>#\<URL fragment>*

Port 80 for HTTP\
Port 443 for HTTPS\
Port 53 for DNS

## Recon

**Common files** include `/robots.txt`, `/index.html`, `/index.php`, `/favicon.ico`

1. **Inquire** about the target. What does it do? What is the application for? Any potentially critical endpoints?

2. Get **ASN** and **WHOIS**. More info [on YouTube](https://www.youtube.com/watch?v=6rKHTPp_kgk).
    - Ipinfo tool
    - Amass tool
    - ASNLookup tool
    - ... more

3. Find all **subdomains**, even the ones that aren't active. And scan all the ports. Use a brote forcer tool for this.
    - [trickest.io](https://trickest.io/)
    - Use amass and subfinder
    - Use wordlists with bruteforcer tool on domains found with ASN
    - Scrape web pages
    - gobuster
    - ... more

4. Get an **eyewitness** to verify the page is real and up.
    - Maybe possibility for **subdomain takeover**, use nuclei

## Vulnerabilities

### CSRF

### NoSQL injection

### Race conditions

### SSRF

### SQL injection

### XXE injection