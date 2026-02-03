# Output Interpretation

Understanding reconFTW's output structure and how to interpret results is crucial for effective reconnaissance.

---

## Output Directory Structure

When you run a scan against `example.com`, reconFTW creates:

```
Recon/
└── example.com/
    ├── subdomains/           # Subdomain enumeration results
    ├── webs/                 # Web probing and analysis
    ├── hosts/                # IP and port scanning
    ├── osint/                # OSINT findings
    ├── vulns/                # Vulnerability scan results
    ├── nuclei_output/        # Nuclei JSON results
    ├── fuzzing/              # Directory/file fuzzing
    ├── js/                   # JavaScript analysis
    ├── screenshots/          # Web screenshots
    ├── .tmp/                 # Temporary files
    ├── .log/                 # Execution logs
    ├── .called_fn/           # Checkpoint markers
    ├── assets.jsonl          # Automation-friendly asset list
    └── hotlist.txt           # Risk-scored findings
```

---

## Subdomain Files (`subdomains/`)

### subdomains.txt

**Content:** Final deduplicated list of discovered subdomains.

**Format:** One subdomain per line.

```
www.example.com
api.example.com
mail.example.com
dev.example.com
staging.example.com
```

**Use Cases:**
- Input for further enumeration
- Scope verification
- Asset inventory

---

### subdomains_crt.txt

**Content:** Subdomains from Certificate Transparency logs.

**Source:** crt.sh queries

```
www.example.com
mail.example.com
*.dev.example.com
autodiscover.example.com
```

---

### subdomains_dnsrecords.txt

**Content:** DNS records for resolved subdomains.

**Format:** Subdomain with record types.

```
www.example.com [A] 93.184.216.34
mail.example.com [CNAME] mail.example.com.mail.protection.outlook.com
api.example.com [A] 93.184.216.35
example.com [MX] 10 mail.example.com
example.com [TXT] "v=spf1 include:_spf.google.com ~all"
```

---

### subdomains_noerror.txt

**Content:** Subdomains discovered via DNS NOERROR response.

**Technique:** DNS response code analysis.

---

### subdomains_permut.txt

**Content:** Subdomains found via permutation techniques.

---

### subdomains_recursive.txt

**Content:** Subdomains from recursive enumeration.

---

### subdomains_scraping.txt

**Content:** Subdomains extracted from web scraping.

---

## Web Files (`webs/`)

### webs.txt

**Content:** Live web servers (HTTP 200/30x responses).

**Format:** Full URLs.

```
https://www.example.com
https://api.example.com
http://dev.example.com:8080
https://staging.example.com
```

**Use Cases:**
- Target list for vulnerability scanning
- Web application testing
- Screenshot generation

---

### webs_all.txt

**Content:** All probed URLs with HTTP response data.

**Format:** URL with metadata.

```
https://www.example.com [200] [Example Site] [nginx]
https://api.example.com [200] [API v2] [Express]
http://dev.example.com:8080 [403] [Forbidden] [Apache]
```

---

### url_extract.txt

**Content:** All discovered URLs from crawling and archives.

**Sources:** gau, waybackurls, katana, hakrawler

```
https://example.com/login
https://example.com/api/v1/users
https://example.com/admin/dashboard
https://example.com/uploads/document.pdf
https://example.com/search?q=test
```

---

### takeover.txt

**Content:** Potential subdomain takeover vulnerabilities.

**Format:** Subdomain with service info.

```
VULNERABLE: docs.example.com [GitHub Pages]
VULNERABLE: blog.example.com [Heroku]
EDGE CASE: old.example.com [S3 bucket]
```

**Action Required:** Verify and claim vulnerable subdomains.

---

### url_gf/

**Content:** URLs categorized by vulnerability patterns.

**Files:**
- `xss.txt` - XSS candidates
- `sqli.txt` - SQL injection candidates
- `ssrf.txt` - SSRF candidates
- `lfi.txt` - LFI candidates
- `redirect.txt` - Open redirect candidates
- `rce.txt` - RCE candidates
- `idor.txt` - IDOR candidates

**Example (xss.txt):**
```
https://example.com/search?q=FUZZ
https://example.com/user?name=FUZZ
https://example.com/callback?url=FUZZ
```

---

### url_extensions/

**Content:** URLs grouped by file extension.

**Files:**
- `url_pdf.txt`
- `url_doc.txt`
- `url_js.txt`
- `url_json.txt`
- `url_xml.txt`
- `url_config.txt`

---

## Host Files (`hosts/`)

### ips.txt

**Content:** All resolved IP addresses (non-CDN).

```
93.184.216.34
93.184.216.35
10.0.0.100
```

---

### cdn.txt

**Content:** IP addresses identified as CDN.

```
104.16.132.229 [cloudflare]
151.101.1.195 [fastly]
13.32.123.45 [cloudfront]
```

---

### portscan_passive.txt

**Content:** Port scan results from Shodan.

```
93.184.216.34
  22/tcp    open  ssh        OpenSSH 8.2
  80/tcp    open  http       nginx 1.18
  443/tcp   open  https      nginx 1.18
  3306/tcp  open  mysql      MySQL 8.0
```

---

### portscan_active.txt

**Content:** Active nmap scan results.

**Format:** Standard nmap output.

```
Nmap scan report for 93.184.216.34
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 8.2p1 Ubuntu
| vulners: 
|   CVE-2020-15778  6.8
80/tcp    open  http       nginx 1.18.0
443/tcp   open  ssl/https  nginx 1.18.0
| ssl-cert: Subject: CN=example.com
```

---

### portscan_active.xml

**Content:** Nmap results in XML format.

**Use Cases:**
- Import to vulnerability scanners
- Parse with scripts
- Import to Faraday

---

### portscan_active.gnmap

**Content:** Nmap greppable format.

```
Host: 93.184.216.34 () Ports: 22/open/tcp//ssh//OpenSSH 8.2p1/, 80/open/tcp//http//nginx 1.18.0/, 443/open/tcp//https//
```

---

### waf.txt

**Content:** WAF detection results.

```
https://www.example.com
  WAF: Cloudflare
  Detected by: Response headers

https://api.example.com
  WAF: AWS WAF
  Detected by: Response behavior
```

---

### geo.txt

**Content:** IP geolocation data.

```
93.184.216.34
  Country: United States
  Region: California
  City: Los Angeles
  Org: Edgecast Inc.
  ASN: AS15133
```

---

## OSINT Files (`osint/`)

### dorks.txt

**Content:** Google dork results.

```
[site:example.com filetype:pdf]
https://example.com/documents/report.pdf
https://example.com/downloads/manual.pdf

[site:example.com inurl:admin]
https://example.com/admin/login
https://example.com/administrator/
```

---

### emails.txt

**Content:** Discovered email addresses.

```
admin@example.com
contact@example.com
john.doe@example.com
support@example.com
```

---

### passwords.txt

**Content:** Leaked credential data.

**Format:** Email:password pairs (from breach databases).

```
user@example.com:password123
admin@example.com:admin2020
```

**⚠️ IMPORTANT:** Handle with care, sensitive data.

---

### metadata_results.txt

**Content:** Document metadata extraction.

```
File: report.pdf
  Author: John Doe
  Creator: Microsoft Word
  Creation Date: 2023-01-15
  Software: Adobe Acrobat
  
File: presentation.pptx
  Author: Jane Smith
  Company: Example Corp
  Last Modified: 2023-03-20
```

---

### github_company_secrets.json

**Content:** Secrets found in GitHub repositories.

**Format:** JSON with file locations and secret types.

```json
[
  {
    "file": "config.js",
    "repo": "example/webapp",
    "secret_type": "AWS_KEY",
    "match": "AKIA..."
  },
  {
    "file": ".env.example",
    "repo": "example/api",
    "secret_type": "API_KEY",
    "match": "sk_live_..."
  }
]
```

---

### apileaks.txt

**Content:** API endpoints from Postman/Swagger leaks.

```
[Postman Collection: example-api]
POST /api/v1/users
GET /api/v1/orders
DELETE /api/v1/sessions

[Swagger: api.example.com]
/api/auth/login
/api/users/{id}
/api/admin/settings
```

---

### domain_info.txt

**Content:** WHOIS and domain intelligence.

```
Domain: example.com
Registrar: GoDaddy
Created: 1995-08-14
Expires: 2025-08-13
Registrant: Example Corporation
Email: domains@example.com

Nameservers:
  ns1.example.com
  ns2.example.com
```

---

### spf_dmarc.txt

**Content:** Email security analysis.

```
Domain: example.com

SPF Record: v=spf1 include:_spf.google.com ~all
  Status: CONFIGURED
  Policy: Soft fail

DMARC Record: v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com
  Status: CONFIGURED
  Policy: Quarantine
  
Email Spoofing: PROTECTED (moderate)
```

---

## Vulnerability Files (`vulns/`)

### nuclei_output/

**Content:** Nuclei scan results in JSON format.

**Files:**
- `nuclei_critical.json`
- `nuclei_high.json`
- `nuclei_medium.json`
- `nuclei_low.json`
- `nuclei_info.json`

**Sample JSON:**
```json
{
  "template-id": "cve-2021-44228-log4j-rce",
  "info": {
    "name": "Apache Log4j RCE",
    "severity": "critical",
    "tags": ["cve", "rce", "log4j"]
  },
  "matcher-name": "dns",
  "host": "https://api.example.com",
  "matched-at": "https://api.example.com/api/v1/log",
  "timestamp": "2023-06-15T10:30:45Z"
}
```

---

### xss.txt

**Content:** XSS vulnerability findings.

```
[VULNERABLE] https://example.com/search?q=<script>alert(1)</script>
  Payload: <script>alert(1)</script>
  Type: Reflected XSS
  
[VULNERABLE] https://example.com/user?name="><img src=x onerror=alert(1)>
  Payload: "><img src=x onerror=alert(1)>
  Type: Reflected XSS
```

---

### sqli.txt

**Content:** SQL injection findings.

```
[VULNERABLE] https://example.com/product?id=1
  Parameter: id
  Type: Boolean-based blind
  Payload: 1 AND 1=1
  DBMS: MySQL
  
[VULNERABLE] https://example.com/user?name=admin
  Parameter: name
  Type: Error-based
  Payload: admin'
  DBMS: PostgreSQL
```

---

### ssrf.txt

**Content:** SSRF vulnerability findings.

```
[VULNERABLE] https://example.com/fetch?url=http://169.254.169.254
  Parameter: url
  Type: Blind SSRF
  Evidence: Delayed response / OOB callback
```

---

### cors.txt

**Content:** CORS misconfiguration findings.

```
[MISCONFIGURED] https://api.example.com
  Issue: Arbitrary Origin Reflection
  ACAO: *
  ACAC: true
  Impact: Credential theft possible
```

---

### redirect.txt

**Content:** Open redirect findings.

```
[VULNERABLE] https://example.com/redirect?url=https://evil.com
  Parameter: url
  Redirects to: https://evil.com
```

---

### lfi.txt

**Content:** Local File Inclusion findings.

```
[VULNERABLE] https://example.com/view?file=../../../etc/passwd
  Parameter: file
  Payload: ../../../etc/passwd
  Evidence: root:x:0:0:root:/root:/bin/bash
```

---

### testssl.txt

**Content:** SSL/TLS analysis results.

```
Testing: https://example.com

Protocol Support:
  TLS 1.3: YES
  TLS 1.2: YES
  TLS 1.1: NO (good)
  TLS 1.0: NO (good)
  SSL 3.0: NO (good)

Vulnerabilities:
  Heartbleed: NOT vulnerable
  ROBOT: NOT vulnerable
  BEAST: NOT vulnerable
  
Certificate:
  Subject: CN=example.com
  Issuer: Let's Encrypt
  Valid Until: 2024-01-15
  Key Size: 2048 bits
```

---

## JavaScript Files (`js/`)

### js_files.txt

**Content:** Discovered JavaScript file URLs.

```
https://example.com/static/app.js
https://example.com/assets/bundle.min.js
https://example.com/js/vendor.js
https://cdn.example.com/lib/jquery-3.6.0.min.js
```

---

### js_secrets.txt

**Content:** Secrets found in JavaScript files.

```
[API_KEY] https://example.com/app.js
  Line 145: apiKey: "sk_live_abc123..."
  
[AWS_KEY] https://example.com/config.js
  Line 23: accessKeyId: "AKIA..."
  
[PRIVATE_KEY] https://example.com/auth.js
  Line 89: privateKey: "-----BEGIN RSA PRIVATE KEY-----"
```

---

### js_endpoints.txt

**Content:** API endpoints extracted from JavaScript.

```
/api/v1/users
/api/v1/orders
/api/v2/products
/internal/admin/settings
/graphql
/api/auth/token
```

---

## Fuzzing Files (`fuzzing/`)

### fuzzing_full.txt

**Content:** All fuzzing results combined.

```
[200] https://example.com/admin
[200] https://example.com/api
[301] https://example.com/docs → /documentation
[403] https://example.com/.git
[200] https://example.com/backup.zip
```

---

### fuzzing_{subdomain}.txt

**Content:** Per-subdomain fuzzing results.

---

## Screenshots (`screenshots/`)

**Content:** Web page screenshots.

**Format:** PNG images named by URL hash.

**Files:**
- `https_www.example.com.png`
- `https_api.example.com.png`
- `http_dev.example.com_8080.png`

**Gallery View:** Use gowitness report for HTML gallery.

```bash
# Generate HTML report
gowitness report serve
```

---

## Log Files (`.log/`)

### reconftw.log

**Content:** Main execution log.

```
[2023-06-15 10:00:00] Starting reconnaissance for example.com
[2023-06-15 10:00:01] Running sub_passive...
[2023-06-15 10:05:30] sub_passive completed: 150 subdomains
[2023-06-15 10:05:31] Running sub_crt...
```

---

### errors.log

**Content:** Error messages during execution.

```
[ERROR] subfinder: API rate limit exceeded for VirusTotal
[ERROR] httpx: Connection timeout for staging.example.com
[WARN] nuclei: Template cve-2021-1234 deprecated
```

---

## Checkpoint Files (`.called_fn/`)

**Content:** Function completion markers for checkpoint/resume.

**Files:**
```
.called_fn/
├── sub_passive
├── sub_crt
├── sub_brute
├── webprobe_simple
└── nuclei_check
```

**Purpose:** Resume interrupted scans from last checkpoint.

---

## Special Files

### assets.jsonl

**Content:** Automation-friendly asset list in JSON Lines format.

**Format:**
```json
{"subdomain":"www.example.com","ip":"93.184.216.34","url":"https://www.example.com","status":200,"title":"Example Site"}
{"subdomain":"api.example.com","ip":"93.184.216.35","url":"https://api.example.com","status":200,"title":"API v2"}
```

**Use Cases:**
- Pipeline integration
- Custom tooling
- Data analysis

---

### hotlist.txt

**Content:** Risk-scored priority targets.

**Format:** Assets with risk indicators.

```
[HIGH] https://admin.example.com - Admin panel exposed
[HIGH] https://api.example.com/graphql - GraphQL endpoint
[MEDIUM] https://staging.example.com - Staging environment
[MEDIUM] https://dev.example.com - Development server
```

---

## Interpreting Nuclei Results

### Severity Levels

| Severity | Description | Action |
|----------|-------------|--------|
| **Critical** | Immediate exploitation possible | Report immediately |
| **High** | Significant security impact | Prioritize remediation |
| **Medium** | Moderate risk | Schedule fix |
| **Low** | Minor issues | Best practice |
| **Info** | Informational | Document |

### Reading Nuclei JSON

```json
{
  "template-id": "cve-2021-44228-log4j-rce",
  "info": {
    "name": "Apache Log4j2 RCE (CVE-2021-44228)",
    "severity": "critical",
    "description": "Apache Log4j2 allows remote code execution...",
    "reference": ["https://nvd.nist.gov/vuln/detail/CVE-2021-44228"],
    "tags": ["cve", "cve2021", "rce", "log4j", "apache"]
  },
  "type": "http",
  "host": "https://api.example.com",
  "matched-at": "https://api.example.com/api/v1/log",
  "extracted-results": ["dns-callback-received"],
  "curl-command": "curl -X POST ...",
  "timestamp": "2023-06-15T10:30:45.123456789Z"
}
```

### Key Fields:
- `template-id`: Template identifier
- `severity`: Risk level
- `host`: Target URL
- `matched-at`: Exact vulnerable endpoint
- `extracted-results`: Evidence of vulnerability
- `curl-command`: Reproduction command

---

## Report Generation

### AI-Generated Reports

```bash
# Generate AI report
./reconftw.sh -d example.com -y

# Custom report type
./reconftw.sh -d example.com -y --report-type executive
```

**Report Types:**
- `detailed` - Technical deep-dive
- `executive` - High-level summary
- `compliance` - Compliance-focused

### Manual Report Creation

1. Collect key findings from:
   - `vulns/nuclei_output/`
   - `webs/takeover.txt`
   - `osint/github_company_secrets.json`

2. Prioritize by severity

3. Include reproduction steps from curl commands

---

## Data Export

### Export to CSV

```bash
# Convert JSON to CSV
cat nuclei_output/*.json | jq -r '[.host, .["template-id"], .info.severity] | @csv'
```

### Export to Faraday

Automatic when `FARADAY` is enabled. Results imported to workspace.

### Export to JSON

Most output files have JSON equivalents in `.tmp/` directory.

---

## Cleanup

### Temporary Files

```bash
# Clear temp files
rm -rf Recon/example.com/.tmp/*
```

### Reset Checkpoints

```bash
# Remove checkpoints to re-run functions
rm -rf Recon/example.com/.called_fn/*
```

### Full Clean

```bash
# Remove all data for target
rm -rf Recon/example.com/
```

---

## Next Steps

- **[Integrations](../08-integrations/axiom.md)** - Axiom and Faraday
- **[Advanced Usage](../10-advanced/advanced.md)** - Custom functions
