# Tools Reference

reconFTW integrates 80+ security tools. This reference documents each tool, its purpose, and how reconFTW uses it.

---

## Tool Categories

| Category | Tools | Purpose |
|----------|-------|---------|
| Subdomain Enumeration | 15+ | Discover subdomains |
| DNS Analysis | 8+ | DNS resolution and records |
| Web Probing | 10+ | HTTP analysis |
| Content Discovery | 6+ | Directory/file fuzzing |
| Vulnerability Scanning | 20+ | Detect vulnerabilities |
| OSINT | 12+ | Intelligence gathering |
| Utilities | 10+ | Support functions |

---

## Installation Verification

```bash
# Check all tool installations
./reconftw.sh --check-tools

# System health check
./reconftw.sh --health-check
```

---

## Subdomain Enumeration Tools

### subfinder

**Purpose:** Passive subdomain enumeration using APIs and data sources.

**Data Sources:** VirusTotal, Shodan, Censys, SecurityTrails, and 40+ more.

**Usage in reconFTW:**
```bash
# Called in sub_passive()
subfinder -d example.com -all -o output.txt
```

**Configuration:**
```bash
# Control how long passive enumeration runs (minutes)
SUBFINDER_ENUM_TIMEOUT=180
# API keys in ~/.config/subfinder/provider-config.yaml
```

**Website:** https://github.com/projectdiscovery/subfinder

---

### amass

**Purpose:** Multi-source subdomain enumeration using multiple techniques.

**Techniques:** DNS brute-force, web scraping, APIs, certificate logs.

**Usage in reconFTW:**
```bash
# Passive mode
amass enum -passive -d example.com -o output.txt
```

**Configuration:**
```bash
# API keys in ~/.config/amass/config.ini
```

**Website:** https://github.com/owasp-amass/amass

---

### assetfinder

**Purpose:** Fast passive subdomain enumeration.

**Usage in reconFTW:**
```bash
# Called for quick enumeration
echo example.com | assetfinder --subs-only
```

**Website:** https://github.com/tomnomnom/assetfinder

---

### findomain

**Purpose:** Fast subdomain enumeration using multiple APIs.

**Usage in reconFTW:**
```bash
findomain -t example.com -u output.txt
```

**Configuration:**
```bash
# API keys as environment variables
FINDOMAIN_FB_TOKEN=...
FINDOMAIN_VIRUSTOTAL_TOKEN=...
```

**Website:** https://github.com/Findomain/Findomain

---

### github-subdomains

**Purpose:** Find subdomains mentioned in GitHub code.

**Usage in reconFTW:**
```bash
github-subdomains -d example.com -t $GITHUB_TOKEN
```

**Configuration:**
```bash
GITHUB_TOKEN="your_token"  # In secrets.cfg
```

**Website:** https://github.com/gwen001/github-subdomains

---

### crt.sh / ctfr

**Purpose:** Query Certificate Transparency logs.

**Usage in reconFTW:**
```bash
# Certificate transparency lookup
curl "https://crt.sh/?q=%25.example.com&output=json"
```

---

### dnsx

**Purpose:** Fast DNS resolution and record querying.

**Usage in reconFTW:**
```bash
# DNS resolution
cat subdomains.txt | dnsx -silent -a -resp
```

**Configuration:**
```bash
DNSX_THREADS=100
```

**Website:** https://github.com/projectdiscovery/dnsx

---

### puredns

**Purpose:** High-performance DNS brute-forcing with wildcard filtering.

**Usage in reconFTW:**
```bash
# DNS brute-force
puredns bruteforce wordlist.txt example.com -r resolvers.txt
```

**Configuration:**
```bash
PUREDNS_PUBLIC_LIMIT=10
```

**Website:** https://github.com/d3mondev/puredns

---

### shuffledns

**Purpose:** Wrapper for massdns with wildcard handling.

**Usage in reconFTW:**
```bash
shuffledns -d example.com -w wordlist.txt -r resolvers.txt
```

**Website:** https://github.com/projectdiscovery/shuffledns

---

### dnsgen

**Purpose:** Generate subdomain permutations.

**Usage in reconFTW:**
```bash
# Generate permutations
cat subdomains.txt | dnsgen - | puredns resolve
```

**Website:** https://github.com/ProjectAnte/dnsgen

---

### alterx

**Purpose:** AI-powered subdomain permutation generation.

**Usage in reconFTW:**
```bash
# Called in sub_ia_permut()
echo example.com | alterx
```

**Website:** https://github.com/projectdiscovery/alterx

---

### gotator

**Purpose:** Fast subdomain permutation generator.

**Usage in reconFTW:**
```bash
gotator -sub subdomains.txt -perm permutations.txt -depth 1
```

**Website:** https://github.com/Josue87/gotator

---

### regulator

**Purpose:** Generate subdomains based on regex patterns.

**Usage in reconFTW:**
```bash
# Called in sub_regex_permut()
regulator -d example.com
```

---

### analyticsrelationships

**Purpose:** Find related domains via Google Analytics IDs.

**Usage in reconFTW:**
```bash
# Find related domains
analyticsrelationships -ch $BUILTWITH_API_KEY
```

**Website:** https://github.com/Josue87/AnalyticsRelationships

---

### tlsx

**Purpose:** TLS/SSL certificate analysis and subdomain discovery.

**Usage in reconFTW:**
```bash
# Extract subdomains from certificates
echo example.com | tlsx -san -cn -silent
```

**Website:** https://github.com/projectdiscovery/tlsx

---

## Web Probing Tools

### httpx

**Purpose:** Fast HTTP probing with metadata extraction.

**Features:** Status codes, titles, technologies, content length.

**Usage in reconFTW:**
```bash
# Probe for live hosts
cat subdomains.txt | httpx -ports 80,443,8080 -title -tech-detect
```

**Configuration:**
```bash
HTTPX_THREADS=50
HTTPX_RATELIMIT=150
HTTPX_TIMEOUT=10
```

**Website:** https://github.com/projectdiscovery/httpx

---

### gowitness

**Purpose:** Web screenshot tool.

**Usage in reconFTW:**
```bash
# Screenshot web pages
gowitness file -f webs.txt --delay 2
```

**Configuration:**
```bash
GOWITNESS_THREADS=8
```

**Website:** https://github.com/sensepost/gowitness

---

### webanalyze

**Purpose:** Technology detection (Wappalyzer-based).

**Usage in reconFTW:**
```bash
# Detect technologies
webanalyze -host https://example.com
```

**Website:** https://github.com/rverton/webanalyze

---

### wafw00f

**Purpose:** Web Application Firewall detection.

**Usage in reconFTW:**
```bash
# Detect WAF
wafw00f https://example.com
```

**Website:** https://github.com/EnableSecurity/wafw00f

---

## Content Discovery Tools

### ffuf

**Purpose:** Fast web fuzzer for directory/file discovery.

**Usage in reconFTW:**
```bash
# Directory fuzzing
ffuf -u https://example.com/FUZZ -w wordlist.txt -mc 200,301,302
```

**Configuration:**
```bash
FFUF_THREADS=40
FFUF_RATELIMIT=
```

**Website:** https://github.com/ffuf/ffuf

---

### feroxbuster

**Purpose:** Recursive content discovery.

**Usage in reconFTW:**
```bash
# Recursive fuzzing
feroxbuster -u https://example.com -w wordlist.txt
```

**Website:** https://github.com/epi052/feroxbuster

---

### dirsearch

**Purpose:** Web path discovery.

**Usage in reconFTW:**
```bash
# Directory enumeration
dirsearch -u https://example.com -e php,html,js
```

**Website:** https://github.com/maurosoria/dirsearch

---

### hakrawler

**Purpose:** Web crawler for URL discovery.

**Usage in reconFTW:**
```bash
# Crawl website
echo https://example.com | hakrawler -d 3
```

**Website:** https://github.com/hakluke/hakrawler

---

### katana

**Purpose:** Modern web crawler.

**Usage in reconFTW:**
```bash
# Crawl and extract URLs
katana -u https://example.com -d 3 -jc
```

**Website:** https://github.com/projectdiscovery/katana

---

### gospider

**Purpose:** Fast web spidering.

**Usage in reconFTW:**
```bash
# Spider website
gospider -s https://example.com -d 2
```

**Website:** https://github.com/jaeles-project/gospider

---

### gau

**Purpose:** Fetch known URLs from web archives.

**Sources:** Wayback Machine, Common Crawl, URLScan.

**Usage in reconFTW:**
```bash
# Get archived URLs
echo example.com | gau --threads 5
```

**Website:** https://github.com/lc/gau

---

### waybackurls

**Purpose:** Fetch URLs from Wayback Machine.

**Usage in reconFTW:**
```bash
echo example.com | waybackurls
```

**Website:** https://github.com/tomnomnom/waybackurls

---

## Vulnerability Scanning Tools

### nuclei

**Purpose:** Template-based vulnerability scanner.

**Usage in reconFTW:**
```bash
# Scan with templates
nuclei -l urls.txt -t ~/nuclei-templates -severity critical,high
```

**Configuration:**
```bash
NUCLEI_RATELIMIT=150
NUCLEI_SEVERITY="critical,high,medium"
NUCLEI_TEMPLATES_PATH="$HOME/nuclei-templates"
NUCLEI_EXTRA_ARGS=""
```

**Website:** https://github.com/projectdiscovery/nuclei

---

### dalfox

**Purpose:** XSS vulnerability scanner.

**Usage in reconFTW:**
```bash
# Test for XSS
dalfox url https://example.com/search?q=test
```

**Configuration:**
```bash
DALFOX_THREADS=30
```

**Website:** https://github.com/hahwul/dalfox

---

### sqlmap

**Purpose:** Automatic SQL injection detection.

**Usage in reconFTW:**
```bash
# Test for SQLi
sqlmap -u "https://example.com/page?id=1" --batch
```

**Configuration:**
```bash
SQLMAP_THREADS=5
```

**Website:** https://github.com/sqlmapproject/sqlmap

---

### ghauri

**Purpose:** Advanced SQL injection scanner.

**Usage in reconFTW:**
```bash
# Alternative SQLi testing
ghauri -u "https://example.com/page?id=1"
```

**Website:** https://github.com/r0oth3x49/ghauri

---

### commix

**Purpose:** Command injection exploitation.

**Usage in reconFTW:**
```bash
# Test for command injection
commix -u "https://example.com/ping?host=test"
```

**Website:** https://github.com/commixproject/commix

---

### crlfuzz

**Purpose:** CRLF injection scanner.

**Usage in reconFTW:**
```bash
# Test for CRLF
crlfuzz -l urls.txt
```

**Website:** https://github.com/dwisiswant0/crlfuzz

---

### interactsh-client

**Purpose:** Out-of-band interaction detection.

**Usage in reconFTW:**
```bash
# OOB testing server
interactsh-client
```

**Website:** https://github.com/projectdiscovery/interactsh

---

### ssrf-sheriff

**Purpose:** SSRF vulnerability detection.

**Usage in reconFTW:**
```bash
# Test for SSRF
ssrf-sheriff -u "https://example.com/fetch?url="
```

---

### tplmap

**Purpose:** Server-side template injection detection.

**Usage in reconFTW:**
```bash
# Test for SSTI
tplmap -u "https://example.com/page?name=test"
```

**Website:** https://github.com/epinna/tplmap

---

### ppfuzz

**Purpose:** Prototype pollution scanner.

**Usage in reconFTW:**
```bash
# Test for prototype pollution
ppfuzz -l urls.txt
```

---

### smuggler

**Purpose:** HTTP request smuggling detection.

**Usage in reconFTW:**
```bash
# Test for smuggling
python3 smuggler.py -u https://example.com
```

**Website:** https://github.com/defparam/smuggler

---

### Web-Cache-Vulnerability-Scanner

**Purpose:** Web cache poisoning detection.

**Usage in reconFTW:**
```bash
# Test for cache poisoning
wcvs -u https://example.com
```

---

### testssl.sh

**Purpose:** SSL/TLS vulnerability testing.

**Usage in reconFTW:**
```bash
# Test SSL configuration
testssl.sh https://example.com
```

**Website:** https://github.com/drwetter/testssl.sh

---

### byp4xx

**Purpose:** 403/401 bypass techniques.

**Usage in reconFTW:**
```bash
# Bypass 4xx
byp4xx https://example.com/admin
```

**Website:** https://github.com/lobuhi/byp4xx

---

### gf

**Purpose:** Pattern extraction from URLs.

**Patterns:** XSS, SQLi, SSRF, LFI, etc.

**Usage in reconFTW:**
```bash
# Extract SQLi candidates
cat urls.txt | gf sqli
```

**Website:** https://github.com/tomnomnom/gf

---

### Gxss

**Purpose:** Check for reflected parameters.

**Usage in reconFTW:**
```bash
# Check reflection
cat urls.txt | Gxss
```

**Website:** https://github.com/KathanP19/Gxss

---

### kxss

**Purpose:** Find reflected XSS endpoints.

**Usage in reconFTW:**
```bash
# Find reflected params
cat urls.txt | kxss
```

**Website:** https://github.com/Emoe/kxss

---

## OSINT Tools

### theHarvester

**Purpose:** Email and subdomain harvesting.

**Usage in reconFTW:**
```bash
# Harvest emails
theHarvester -d example.com -b all
```

**Website:** https://github.com/laramies/theHarvester

---

### emailfinder

**Purpose:** Find email addresses.

**Usage in reconFTW:**
```bash
# Find emails
emailfinder -d example.com
```

**Website:** https://github.com/Josue87/EmailFinder

---

### pwndb

**Purpose:** Check for leaked credentials.

**Usage in reconFTW:**
```bash
# Check leaks
pwndb -t example.com
```

---

### gitdorker

**Purpose:** GitHub dorking for secrets.

**Usage in reconFTW:**
```bash
# Search GitHub
python3 GitDorker.py -tf GITHUB_TOKEN -q example.com
```

**Website:** https://github.com/obheda12/GitDorker

---

### trufflehog

**Purpose:** Secret scanning in repositories.

**Usage in reconFTW:**
```bash
# Scan for secrets
trufflehog github --org example
```

**Website:** https://github.com/trufflesecurity/trufflehog

---

### gitrob

**Purpose:** GitHub organization reconnaissance.

**Usage in reconFTW:**
```bash
# Scan GitHub org
gitrob -o example-org
```

---

### cloud_enum

**Purpose:** Cloud storage enumeration.

**Usage in reconFTW:**
```bash
# Enumerate cloud storage
python3 cloud_enum.py -k example
```

**Website:** https://github.com/initstring/cloud_enum

---

### dnsrecon

**Purpose:** DNS enumeration and zone transfer.

**Usage in reconFTW:**
```bash
# Zone transfer check
dnsrecon -d example.com -t axfr
```

**Website:** https://github.com/darkoperator/dnsrecon

---

### spoof.py

**Purpose:** Email spoofing check.

**Usage in reconFTW:**
```bash
# Check SPF/DMARC
spoofcheck.py example.com
```

---

### metagoofil

**Purpose:** Metadata extraction from documents.

**Usage in reconFTW:**
```bash
# Extract metadata
metagoofil -d example.com -t pdf,doc
```

**Website:** https://github.com/laramies/metagoofil

---

## Port Scanning Tools

### nmap

**Purpose:** Network discovery and security auditing.

**Usage in reconFTW:**
```bash
# Port scan
nmap -sV -sC --top-ports 200 -Pn target.txt
```

**Configuration:**
```bash
PORTSCAN_ACTIVE_OPTIONS="--top-ports 200 -sV -n -Pn --open"
```

**Website:** https://nmap.org/

---

### smap

**Purpose:** Shodan-based passive port scanning.

**Usage in reconFTW:**
```bash
# Passive port scan
smap target.txt
```

**Website:** https://github.com/s0md3v/Smap

---

### masscan

**Purpose:** Fast port scanning.

**Usage in reconFTW:**
```bash
# Quick port scan
masscan -p1-65535 target -rate=1000
```

**Website:** https://github.com/robertdavidgraham/masscan

---

## JavaScript Analysis Tools

### getJS

**Purpose:** Extract JavaScript files from pages.

**Usage in reconFTW:**
```bash
# Get JS files
getJS -url https://example.com
```

**Website:** https://github.com/003random/getJS

---

### subjs

**Purpose:** Find JavaScript files in pages.

**Usage in reconFTW:**
```bash
# Find JS
cat urls.txt | subjs
```

**Website:** https://github.com/lc/subjs

---

### linkfinder

**Purpose:** Find endpoints in JavaScript files.

**Usage in reconFTW:**
```bash
# Extract endpoints
python3 linkfinder.py -i https://example.com/app.js
```

**Website:** https://github.com/GerbenJav);do/LinkFinder

---

### secretfinder

**Purpose:** Find secrets in JavaScript.

**Usage in reconFTW:**
```bash
# Find secrets in JS
python3 SecretFinder.py -i https://example.com/app.js
```

**Website:** https://github.com/m4ll0k/SecretFinder

---

### mantra

**Purpose:** Hunt for API keys and secrets.

**Usage in reconFTW:**
```bash
# Find API keys
mantra -u https://example.com
```

---

### jsluice

**Purpose:** JavaScript analysis and URL extraction.

**Usage in reconFTW:**
```bash
# Analyze JS
cat app.js | jsluice urls
```

**Website:** https://github.com/BishopFox/jsluice

---

## Utility Tools

### anew

**Purpose:** Append lines to file if they don't exist.

**Usage in reconFTW:**
```bash
# Deduplicate append
cat new.txt | anew existing.txt
```

**Website:** https://github.com/tomnomnom/anew

---

### qsreplace

**Purpose:** Replace query string parameters.

**Usage in reconFTW:**
```bash
# Replace params
cat urls.txt | qsreplace FUZZ
```

**Website:** https://github.com/tomnomnom/qsreplace

---

### unfurl

**Purpose:** Parse and extract URL components.

**Usage in reconFTW:**
```bash
# Extract domains
cat urls.txt | unfurl domains
```

**Website:** https://github.com/tomnomnom/unfurl

---

### urldedupe

**Purpose:** Remove duplicate URLs.

**Usage in reconFTW:**
```bash
# Deduplicate URLs
cat urls.txt | urldedupe
```

**Website:** https://github.com/ameenmaali/urldedupe

---

### inscope

**Purpose:** Filter URLs by scope.

**Usage in reconFTW:**
```bash
# Filter in-scope
cat urls.txt | inscope
```

**Website:** https://github.com/tomnomnom/inscope

---

### interlace

**Purpose:** Run commands across multiple targets.

**Usage in reconFTW:**
```bash
# Parallel execution
interlace -tL targets.txt -threads 10 -c "cmd _target_"
```

**Website:** https://github.com/codingo/Interlace

---

### notify

**Purpose:** Send notifications (Slack, Discord, etc.).

**Usage in reconFTW:**
```bash
# Send notification
echo "message" | notify
```

**Configuration:**
```bash
NOTIFICATION=true
```

**Website:** https://github.com/projectdiscovery/notify

---

### cdncheck

**Purpose:** Identify CDN providers.

**Usage in reconFTW:**
```bash
# Check for CDN
cat ips.txt | cdncheck
```

**Website:** https://github.com/projectdiscovery/cdncheck

---

### mapcidr

**Purpose:** CIDR manipulation and expansion.

**Usage in reconFTW:**
```bash
# Expand CIDR
echo "192.168.1.0/24" | mapcidr
```

**Website:** https://github.com/projectdiscovery/mapcidr

---

### dnsvalidator

**Purpose:** Validate DNS resolvers.

**Usage in reconFTW:**
```bash
# Validate resolvers
dnsvalidator -tL resolvers.txt -threads 100
```

**Website:** https://github.com/vortexau/dnsvalidator

---

## API-Dependent Tools

These tools require API keys configured in `secrets.cfg`:

| Tool | API Required |
|------|--------------|
| subfinder | Multiple (optional) |
| shodan | SHODAN_API_KEY |
| censys | CENSYS_API_ID, CENSYS_API_SECRET |
| github-subdomains | GITHUB_TOKEN |
| gitdorker | GITHUB_TOKEN |
| whoisxml | WHOISXML_API |
| securitytrails | SECURITYTRAILS_KEY |
| intelx | INTELX_KEY |
| hunter | HUNTER_API_KEY |

---

## Tool Update Commands

```bash
# Update reconFTW and reinstall all tools
cd reconftw
git pull
./install.sh

# Update specific tool
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest

# Update nuclei templates
nuclei -update-templates
```

---

## Tool Troubleshooting

### Common Issues

1. **Tool not found:** Ensure `~/go/bin` is in PATH
2. **Permission denied:** Check executable permissions
3. **API errors:** Verify API keys in secrets.cfg
4. **Rate limiting:** Reduce thread counts

### Verification

```bash
# Check tool path
which httpx

# Check version
httpx -version

# Test functionality
echo example.com | httpx -silent
```

---

## Adding Custom Tools

To integrate a new tool:

1. Install the tool
2. Create wrapper function in custom module
3. Add configuration options to reconftw.cfg
4. Test integration

---

## Next Steps

- **[Output Interpretation](../07-output/output.md)** - Understanding results
- **[Configuration](../04-configuration/configuration.md)** - Tool settings
