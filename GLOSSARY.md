# Glossary

Technical terms and concepts used throughout the reconFTW documentation.

---

## A

### Active Scanning
Reconnaissance that directly interacts with the target, such as sending HTTP requests or DNS queries. Contrast with [Passive Scanning](#passive-scanning).

### Amass
OWASP's comprehensive subdomain enumeration tool that uses multiple data sources and techniques.

### API Key
Authentication credential for accessing third-party services like Shodan, VirusTotal, or GitHub.

### ASN (Autonomous System Number)
A unique identifier for a network on the internet, useful for discovering related IP ranges.

### Asset
Any discoverable resource belonging to a target: subdomains, IPs, URLs, etc.

### Axiom
Infrastructure automation tool for distributing security tools across cloud instances. Commands use `axiom-*` naming (e.g., `axiom-fleet`, `axiom-scan`). See [Axiom Integration](08-integrations/axiom.md).

---

## B

### Banner Grabbing
Technique to identify services by examining their response banners (e.g., SSH version, HTTP server).

### Brute Force
Technique of systematically checking all possible values (e.g., subdomain wordlist enumeration).

### Bug Bounty
Program where organizations pay security researchers for responsibly disclosed vulnerabilities.

---

## C

### CDN (Content Delivery Network)
Distributed network of servers that cache content closer to users. Examples: Cloudflare, Akamai, CloudFront.

### Certificate Transparency (CT)
Public logs of SSL/TLS certificates, useful for discovering subdomains.

### Checkpoint
Marker file indicating a function has completed, enabling scan resumption.

### CIDR (Classless Inter-Domain Routing)
Notation for IP address ranges (e.g., 192.168.1.0/24).

### CORS (Cross-Origin Resource Sharing)
Browser security mechanism that can be misconfigured, leading to data theft vulnerabilities.

### Crawler
Tool that automatically follows links to discover URLs and content on websites.

### CRLF Injection
Vulnerability where attacker injects carriage return and line feed characters to manipulate HTTP headers.

### CRT.sh
Certificate Transparency log search engine operated by Sectigo.

### CVE (Common Vulnerabilities and Exposures)
Standardized identifier for known security vulnerabilities (e.g., CVE-2021-44228).

---

## D

### Dalfox
Fast XSS (Cross-Site Scripting) vulnerability scanner.

### DEEP Mode
reconFTW mode that runs additional intensive checks when target size is below threshold.

### DNS (Domain Name System)
System that translates domain names to IP addresses.

### DNS Bruteforce
Technique of testing wordlist entries as subdomains against DNS servers.

### Dnsx
Fast DNS toolkit for resolution and record querying.

### Dork
Search query using advanced operators to find specific information (e.g., Google dorks).

---

## E

### Endpoint
Specific URL path on a web server (e.g., /api/users).

### Enumeration
Process of systematically discovering assets or information.

---

## F

### Faraday
Collaborative vulnerability management platform for organizing and reporting findings.

### Favicon
Small icon associated with a website, can be used to discover real IPs behind CDNs.

### ffuf
Fast web fuzzer for discovering directories, files, and parameters.

### Fleet
Group of cloud instances managed by Axiom for distributed scanning. See [Axiom Integration](08-integrations/axiom.md).

### Fuzzing
Technique of sending random or semi-random data to find vulnerabilities or hidden content.

---

## G

### GAU (Get All URLs)
Tool that fetches known URLs from web archives and other sources.

### gf (grep for pentesters)
Pattern-matching tool for extracting potentially vulnerable URLs.

### GitBook
Documentation platform used to host docs.reconftw.com.

### GitHub Dorking
Using GitHub search to find sensitive information in repositories.

### Go/Golang
Programming language used by many security tools integrated with reconFTW.

### Gowitness
Screenshot tool for capturing web page images.

---

## H

### httpx
Fast HTTP toolkit for probing web servers and extracting metadata.

### Host
Individual server or IP address.

---

## I

### Incremental Scan
Scan mode that only processes new findings since the last scan.

### In-Scope
Assets that are authorized for testing in an engagement.

### Interactsh
Out-of-band interaction detection service for finding blind vulnerabilities.

---

## J

### JavaScript Analysis
Examining JavaScript files for endpoints, secrets, and vulnerabilities.

### JSON (JavaScript Object Notation)
Data format used for structured output (e.g., Nuclei results).

### JSONL (JSON Lines)
Format with one JSON object per line, useful for streaming data.

---

## K

### Katana
Modern web crawler for URL and endpoint discovery.

---

## L

### LFI (Local File Inclusion)
Vulnerability allowing attackers to read local files from the server.

### Linkfinder
Tool for extracting endpoints from JavaScript files.

---

## M

### Massdns
High-performance DNS resolver for bulk subdomain resolution.

### Metadata
Data about data; in documents, includes author, creation date, software used.

### Module
Logical grouping of related functions in reconFTW (e.g., OSINT module, Subdomains module).

---

## N

### Nmap
Network scanner for port discovery and service detection.

### NOERROR
DNS response code indicating the domain exists (used for subdomain discovery).

### Notify
Tool for sending notifications to Slack, Discord, Telegram, etc.

### Nuclei
Template-based vulnerability scanner by ProjectDiscovery.

### Nuclei Templates
YAML files defining vulnerability checks for Nuclei.

---

## O

### OOB (Out-of-Band)
Testing technique where vulnerable application makes external callback to attacker's server.

### Open Redirect
Vulnerability where application redirects users to arbitrary URLs.

### OSINT (Open Source Intelligence)
Information gathered from publicly available sources.

### Out-of-Scope
Assets explicitly excluded from testing authorization.

---

## P

### Parameter
Query string or form field name (e.g., `?id=1` where `id` is the parameter).

### Passive Scanning
Reconnaissance using only publicly available data without direct target interaction.

### Permutation
Variation of subdomain names (e.g., dev, dev1, development).

### Port
Network endpoint for services (e.g., 80 for HTTP, 443 for HTTPS).

### Port Scanning
Discovering open ports and services on hosts.

### Prototype Pollution
JavaScript vulnerability where attacker modifies Object prototype.

### Proxy
Intermediary server for routing traffic (e.g., Burp Suite).

### Puredns
Fast DNS resolution tool with wildcard filtering.

---

## R

### Rate Limiting
Restricting request frequency to avoid overwhelming targets or being blocked.

### Recon (Reconnaissance)
Information gathering phase of security testing.

### Recursive Enumeration
Discovering subdomains of subdomains (e.g., finding x.dev.example.com).

### Resolver
DNS server that translates domain names to IP addresses.

### RFI (Remote File Inclusion)
Vulnerability allowing inclusion of remote files in server-side code.

---

## S

### Scope
Defined boundaries of what can be tested in an engagement.

### Screenshot
Image capture of web page appearance.

### Secrets
Sensitive data like API keys, passwords, or tokens.

### Shodan
Search engine for internet-connected devices.

### Smap
Shodan-based passive port scanner.

### Spider
See [Crawler](#crawler).

### SQL Injection (SQLi)
Vulnerability where attacker injects SQL commands into database queries.

### SSRF (Server-Side Request Forgery)
Vulnerability where attacker makes server perform requests to internal resources.

### SSTI (Server-Side Template Injection)
Vulnerability in template engines allowing code execution.

### Subdomain
Prefix to a domain name (e.g., `www` in `www.example.com`).

### Subdomain Takeover
Vulnerability where unclaimed subdomain can be claimed by attacker.

### Subfinder
Fast passive subdomain enumeration tool.

---

## T

### Target
Domain, IP, or organization being assessed.

### Template
Reusable definition file (e.g., Nuclei vulnerability templates).

### Thread
Concurrent execution unit; more threads = faster but more resource-intensive.

### TLS/SSL
Encryption protocols for secure communication.

### Tlsx
TLS analysis tool for certificate inspection.

### Trufflehog
Secret scanning tool for Git repositories.

---

## U

### URL (Uniform Resource Locator)
Web address (e.g., https://example.com/page).

### User-Agent
HTTP header identifying the client software.

---

## V

### VPS (Virtual Private Server)
Cloud-hosted virtual machine for running tools.

### Vulnerability
Security weakness that can be exploited.

---

## W

### WAF (Web Application Firewall)
Security system that filters malicious web traffic.

### Wafw00f
WAF detection and fingerprinting tool.

### Wayback Machine
Internet Archive's historical web page database.

### Wildcard
DNS configuration returning response for any subdomain query.

### Wordlist
File containing entries for brute-force testing.

---

## X

### XSS (Cross-Site Scripting)
Vulnerability allowing injection of malicious scripts into web pages.

### XXE (XML External Entity)
Vulnerability in XML parsers allowing file disclosure or SSRF.

---

## Z

### Zen Mode
reconFTW mode with minimal terminal output.

### Zone Transfer
DNS mechanism for replicating records, sometimes misconfigured to leak all subdomains.
