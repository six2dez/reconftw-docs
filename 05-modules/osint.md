# OSINT Module

The OSINT (Open Source Intelligence) module gathers publicly available information about the target without direct interaction. This passive intelligence gathering helps understand the target's digital footprint.

---

## Why OSINT First?

OSINT is executed before any other module for strategic reasons:

1. **Zero Target Interaction**: All queries go to third-party services (Google, GitHub, crt.sh), not the target. This means:
   - No logs generated on target systems
   - No risk of detection or blocking
   - No rate limiting concerns from the target

2. **Context Building**: Information gathered here informs later phases:
   - Email patterns → potential usernames for login bruteforce
   - Exposed API keys → direct access without exploitation
   - Related domains → expanded attack surface

3. **Quick Wins**: OSINT often reveals immediate vulnerabilities:
   - Leaked credentials in GitHub
   - Exposed admin panels via Google dorks
   - Misconfigured cloud storage buckets

4. **Scope Discovery**: May reveal additional targets:
   - Acquisitions mentioned in documents
   - Azure tenant domains
   - Related domains via registrant data

---

## Module Overview

| Function | Purpose | Tools Used |
|----------|---------|------------|
| `google_dorks` | Find exposed files/pages via Google | dorks_hunter |
| `github_dorks` | Search GitHub for leaked secrets | gitdorks_go |
| `github_repos` | Analyze organization repositories | enumerepo, gitleaks, trufflehog |
| `metadata` | Extract document metadata | metagoofil, exiftool |
| `apileaks` | Detect exposed APIs | porch-pirate, SwaggerSpy |
| `emails` | Harvest email addresses | EmailHarvester, LeakSearch |
| `domain_info` | WHOIS and domain intelligence | whois, msftrecon, scopify |
| `third_party_misconfigs` | Third-party service misconfigs | misconfig-mapper |
| `spoof` | Email spoofing vulnerability | spoofy |
| `mail_hygiene` | SPF/DMARC analysis | dig |
| `cloud_enum_scan` | Cloud storage enumeration | cloud_enum |
| `ip_info` | IP intelligence (for IP targets) | WhoisXML API |

---

## Configuration Options

```bash
# In reconftw.cfg

# Master toggle
OSINT=true

# Individual toggles
GOOGLE_DORKS=true
GITHUB_DORKS=true
GITHUB_REPOS=true
METADATA=true
EMAILS=true
DOMAIN_INFO=true
IP_INFO=true
API_LEAKS=true
THIRD_PARTIES=true
SPOOF=true
MAIL_HYGIENE=true
CLOUD_ENUM=true

# Limits
METAFINDER_LIMIT=20  # Max documents to fetch (max 250)
```

---

## Google Dorks

### What It Does

Searches Google for sensitive files, pages, and information exposure using predefined dork queries.

### How It Works

```
Target domain → dorks_hunter → Google search → Results filtered
```

### Example Dorks Searched

- `site:example.com filetype:pdf`
- `site:example.com filetype:sql`
- `site:example.com inurl:admin`
- `site:example.com intitle:"index of"`
- `site:example.com ext:log`

### Output

```
osint/dorks.txt
```

**Sample Output:**
```
[+] site:example.com filetype:pdf
    - https://example.com/docs/report.pdf
    - https://example.com/files/manual.pdf

[+] site:example.com inurl:login
    - https://admin.example.com/login
    - https://portal.example.com/user/login
```

### Configuration

```bash
GOOGLE_DORKS=true
```

> **Note:** Google may rate-limit or block automated queries. Results vary based on Google's indexing.

---

## GitHub Analysis

### GitHub Dorks (`github_dorks`)

Searches GitHub for secrets, credentials, and sensitive information related to the target.

**Requires:** GitHub tokens in `$GITHUB_TOKENS` file

**How It Works:**

```
Target domain → gitdorks_go → GitHub API → Secret patterns matched
```

**Dork Categories:**
- API keys and tokens
- Passwords and credentials
- Configuration files
- Database connection strings
- Private keys

**Output:**
```
osint/gitdorks.txt
```

**Sample Output:**
```
https://github.com/user/repo/blob/main/config.js - "api_key": "sk_live_xxxxx"
https://github.com/org/project/blob/master/.env - DATABASE_URL=postgres://user:pass@host
```

### GitHub Repos (`github_repos`)

Analyzes organization repositories for leaked secrets using multiple detection tools.

**How It Works:**

```
Target domain → Extract org name → enumerepo (find repos) → Clone repos → 
→ gitleaks (scan) → trufflehog (scan) → Combine results
```

**Output:**
```
osint/github_company_secrets.json
```

**Sample Output:**
```json
{
  "Description": "AWS API Key",
  "File": "deploy/config.yaml",
  "Commit": "a1b2c3d4",
  "Match": "AKIA...",
  "Repository": "https://github.com/example-org/infra"
}
```

### Configuration

```bash
GITHUB_DORKS=true
GITHUB_REPOS=true

# Token file path
GITHUB_TOKENS=${tools}/.github_tokens
```

**Creating Token File:**

```bash
# Create file with one token per line
cat > ~/Tools/.github_tokens << EOF
ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
ghp_yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
EOF

# Secure permissions
chmod 600 ~/Tools/.github_tokens
```

> **Tip:** Multiple tokens help avoid rate limiting. Create tokens with `read:org` and `repo` scopes.

---

## Metadata Extraction

### What It Does

Downloads indexed documents (PDF, DOCX, XLSX) and extracts metadata that may reveal:
- Author names (potential usernames)
- Email addresses
- Software versions
- Internal paths
- Creation/modification dates

### How It Works

```
Target domain → metagoofil (download docs) → exiftool (extract metadata) → Filter results
```

### Output

```
osint/metadata_results.txt
```

**Sample Output:**
```
Author                          : John Smith
Creator                         : Microsoft® Word 2019
Producer                        : Microsoft® Word 2019
Template                        : C:\Users\jsmith\Templates\Report.dotx
Email                           : john.smith@example.com
```

### Configuration

```bash
METADATA=true
METAFINDER_LIMIT=20  # Max documents to download (max 250)
```

> **Note:** Downloading many documents can be slow. Adjust `METAFINDER_LIMIT` based on needs.

---

## API Leaks

### What It Does

Searches for exposed API documentation and collections in:
- **Postman** - Public workspaces and collections
- **SwaggerHub** - Public API specifications

### How It Works

```
Target domain → porch-pirate (Postman) → SwaggerSpy (Swagger) → 
→ trufflehog (analyze for secrets) → Combined results
```

### Output

```
osint/postman_leaks.txt           # Raw Postman findings
osint/swagger_leaks.txt           # Raw Swagger findings
osint/postman_leaks_trufflehog.json   # Secrets from Postman
osint/swagger_leaks_trufflehog.json   # Secrets from Swagger
```

**Sample Postman Output:**
```
[Collection] Example API v2
  Environment: Production
  Variables:
    - api_key: sk_live_xxxxx
    - base_url: https://api.example.com
  
[Request] POST /auth/login
  Body: {"username": "admin", "password": "{{password}}"}
```

### Configuration

```bash
API_LEAKS=true
```

---

## Email Harvesting

### What It Does

Discovers email addresses associated with the target domain through:
- Search engine results
- Public databases
- Leaked credential databases

### How It Works

```
Target domain → EmailHarvester (search engines) → LeakSearch (breach data) →
→ Deduplicate → Output
```

### Output

```
osint/emails.txt      # Discovered email addresses
osint/passwords.txt   # Leaked credentials (if found)
```

**Sample Output:**
```
# emails.txt
admin@example.com
support@example.com
john.doe@example.com
careers@example.com

# passwords.txt
john.doe@example.com:password123
admin@example.com:admin2020
```

### Configuration

```bash
EMAILS=true
```

> **⚠️ Ethics:** Handle leaked credentials responsibly. Only use for authorized testing.

---

## Domain Intelligence

### What It Does

Gathers complete domain information:
- WHOIS registration data
- Microsoft 365/Azure tenant domains
- Related domains via Scopify

### How It Works

```
Target domain → whois → WHOIS data
             → msftrecon → Azure/M365 tenants
             → scopify → Related domains
```

### Output

```
osint/domain_info_general.txt     # WHOIS data
osint/azure_tenant_domains.txt    # Microsoft tenant domains
osint/scopify.txt                 # Related scope domains
```

**Sample WHOIS Output:**
```
Domain Name: EXAMPLE.COM
Registry Domain ID: 123456789_DOMAIN_COM-VRSN
Registrar: Example Registrar, Inc.
Creation Date: 1995-08-14T04:00:00Z
Registrant Organization: Example Corporation
Registrant Country: US
Name Server: NS1.EXAMPLE.COM
```

### Configuration

```bash
DOMAIN_INFO=true
```

---

## Third-Party Misconfigurations

### What It Does

Checks for misconfigurations in third-party services used by the target:
- Atlassian (Jira, Confluence)
- Slack
- Zendesk
- HubSpot
- And many more...

### How It Works

```
Target domain → Extract company name → misconfig-mapper → 
→ Check all services → Report findings
```

### Output

```
osint/3rdparts_misconfigurations.txt
```

**Sample Output:**
```
[+] Jira: Open project listing found
    URL: https://example.atlassian.net/browse
    
[+] Slack: Workspace enumeration possible
    Workspace: example-company
    
[+] Zendesk: Public support tickets accessible
    URL: https://example.zendesk.com/hc
```

### Configuration

```bash
THIRD_PARTIES=true
```

---

## Email Spoofing Check

### What It Does

Analyzes if the domain is vulnerable to email spoofing attacks by checking:
- SPF record configuration
- DMARC policy strength
- DKIM presence

### How It Works

```
Target domain → spoofy → Analyze DNS records → 
→ Determine spoofability → Report
```

### Output

```
osint/spoof.txt
```

**Sample Output:**
```
Domain: example.com
SPF Record: v=spf1 include:_spf.google.com ~all
DMARC Record: v=DMARC1; p=none; rua=mailto:dmarc@example.com
Result: POTENTIALLY SPOOFABLE
Reason: DMARC policy is 'none' (not enforced)
```

### Configuration

```bash
SPOOF=true
```

---

## Mail Hygiene

### What It Does

Performs a quick check of email security DNS records:
- TXT records (SPF)
- DMARC records

### How It Works

```
Target domain → dig TXT → SPF record
             → dig TXT _dmarc → DMARC record
```

### Output

```
osint/mail_hygiene.txt
```

**Sample Output:**
```
Domain: example.com

TXT records:
  "v=spf1 include:_spf.google.com include:amazonses.com ~all"

DMARC record:
  "v=DMARC1; p=quarantine; pct=100; rua=mailto:dmarc@example.com"
```

### Configuration

```bash
MAIL_HYGIENE=true
```

---

## Cloud Enumeration

### What It Does

Searches for exposed cloud storage buckets across multiple providers:
- Amazon S3
- Azure Blob Storage
- Google Cloud Storage
- DigitalOcean Spaces

### How It Works

```
Target domain → Extract keywords → cloud_enum → 
→ Check all providers → Report accessible buckets
```

### Output

```
osint/cloud_enum.txt
```

**Sample Output:**
```
[+] S3 Bucket Found: example-backups.s3.amazonaws.com
    Status: Public (READ)
    
[+] Azure Blob Found: exampledata.blob.core.windows.net
    Status: Public (LIST)
    
[+] GCS Bucket Found: example-assets.storage.googleapis.com
    Status: Authenticated Users
```

### Configuration

```bash
CLOUD_ENUM=true
```

---

## IP Information

### What It Does

For IP address targets, gathers:
- Reverse IP lookups (domains on same IP)
- WHOIS information
- Geolocation data

**Requires:** `WHOISXML_API` key

### How It Works

```
IP target → WhoisXML API → Reverse IP
                        → WHOIS data
                        → Geolocation
```

### Output

```
osint/ip_<IP>_relations.txt   # Domains on IP
osint/ip_<IP>_whois.txt       # WHOIS data
osint/ip_<IP>_location.txt    # Geolocation
```

**Sample Output:**
```
# ip_192.168.1.1_relations.txt
example.com 192.168.1.1
test.com 192.168.1.1
demo.org 192.168.1.1

# ip_192.168.1.1_location.txt
192.168.1.1
{
  "city": "San Francisco",
  "region": "California",
  "country": "US",
  "org": "Example Hosting Inc."
}
```

### Configuration

```bash
IP_INFO=true
WHOISXML_API="your_api_key"  # In secrets.cfg
```

---

## Running OSINT Only

```bash
# Run only OSINT module
./reconftw.sh -d example.com -n
```

This executes all enabled OSINT functions without subdomain enumeration or vulnerability scanning.

---

## Output Summary

| Function | Output File(s) |
|----------|---------------|
| google_dorks | `osint/dorks.txt` |
| github_dorks | `osint/gitdorks.txt` |
| github_repos | `osint/github_company_secrets.json` |
| metadata | `osint/metadata_results.txt` |
| apileaks | `osint/postman_leaks*.txt`, `osint/swagger_leaks*.txt` |
| emails | `osint/emails.txt`, `osint/passwords.txt` |
| domain_info | `osint/domain_info_*.txt`, `osint/scopify.txt` |
| third_party_misconfigs | `osint/3rdparts_misconfigurations.txt` |
| spoof | `osint/spoof.txt` |
| mail_hygiene | `osint/mail_hygiene.txt` |
| cloud_enum_scan | `osint/cloud_enum.txt` |
| ip_info | `osint/ip_*_*.txt` |

---

## Best Practices

1. **Configure API Keys:** Many OSINT functions work better with API keys (GitHub, WhoisXML)

2. **Rate Limiting:** Google may block automated searches; space out scans

3. **Legal Considerations:** OSINT is generally legal but respect terms of service

4. **Credential Handling:** Handle any discovered credentials responsibly

5. **Verification:** Always verify OSINT findings with additional sources

---

## Next Steps

- **[Subdomain Module](subdomains.md)** - Discover attack surface
- **[Output Interpretation](../07-output/output.md)** - Understand results
