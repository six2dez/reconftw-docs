# Data Model & I/O Reference

Complete reference for reconFTW's input formats and output structure.

---

## Input Formats

### Single Domain (-d)

```bash
# Standard domain
./reconftw.sh -d example.com -r

# Subdomain (scans parent automatically)
./reconftw.sh -d api.example.com -r

# With protocol (protocol is stripped)
./reconftw.sh -d https://example.com -r
```

### Domain List (-l)

```bash
./reconftw.sh -l targets.txt -r
```

**targets.txt format:**
```
example.com
target.org
subdomain.another.com
```

- One domain per line
- No protocols, paths, or ports
- Blank lines ignored
- Comments NOT supported

### CIDR/IP Range (-m)

```bash
./reconftw.sh -m 192.168.1.0/24 -r
```

Supported formats:
- CIDR: `192.168.1.0/24`
- Range: `192.168.1.1-255`
- Single IP: `192.168.1.1`

### Scope Files

**In-scope (-i):**
```
*.example.com
api.example.com
192.168.1.0/24
```

**Out-of-scope (-x):**
```
admin.example.com
*.internal.example.com
10.0.0.0/8
```

Wildcards supported: `*.example.com` matches `anything.example.com`

---

## Output Directory Structure

All results stored in `Recon/<domain>/`:

```
Recon/example.com/
├── .called_fn/              # Checkpoint markers (hidden)
├── .log/                    # Execution logs (hidden)
│   └── reconftw.log
├── .tmp/                    # Temporary files (hidden)
│
├── subdomains/              # Subdomain enumeration results
│   ├── subdomains.txt       # [KEY] Final merged subdomains
│   ├── subdomains_crt.txt   # CT log subdomains
│   ├── subdomains_passive.txt
│   ├── subdomains_brute.txt
│   ├── subdomains_permut.txt
│   ├── subdomains_dnsrecords.txt  # DNS record details
│   └── takeover.txt         # Subdomain takeover findings
│
├── webs/                    # Web probing results
│   ├── webs.txt             # [KEY] Live HTTP/HTTPS servers
│   ├── webs_all.txt         # All probed URLs with metadata
│   ├── webs_info.txt        # Detailed web info (title, status, tech)
│   └── url_extract.txt      # Extracted URLs from crawling
│
├── hosts/                   # Host/IP information
│   ├── ips.txt              # [KEY] Resolved IP addresses
│   ├── cdn.txt              # CDN IPs (filtered)
│   ├── portscan_active.txt  # Open ports
│   └── portscan_active.xml  # Nmap XML output
│
├── osint/                   # OSINT results
│   ├── dorks.txt            # Google dork results
│   ├── github_company_secrets.json  # GitHub leaks
│   ├── github_dorks.txt
│   ├── emails.txt           # [KEY] Discovered emails
│   ├── passwords.txt        # Leaked credentials
│   └── metadata_results.txt # Document metadata
│
├── vulns/                   # Vulnerability findings
│   ├── nuclei_output.txt    # Nuclei text output
│   ├── nuclei_output.json   # [KEY] Nuclei JSON (parseable)
│   ├── xss.txt              # XSS vulnerabilities
│   ├── sqli.txt             # SQL injection
│   ├── cors.txt             # CORS misconfigs
│   ├── ssrf.txt             # SSRF findings
│   └── ...                  # Other vuln types
│
├── js/                      # JavaScript analysis
│   ├── js_livelinks.txt     # Found JS files
│   ├── js_secrets.txt       # [KEY] Secrets in JS
│   └── js_endpoints.txt     # API endpoints
│
├── screenshots/             # Web screenshots
│   └── *.png
│
├── fuzzing/                 # Directory fuzzing
│   └── fuzzing_full.txt     # Discovered paths
│
└── hotlist.txt              # [KEY] Priority findings (risk-scored)
```

---

## Key Output Files

### subdomains/subdomains.txt

The master list of all discovered subdomains.

```
api.example.com
www.example.com
staging.example.com
dev.example.com
```

**How it's built:**
1. Passive sources (subfinder, APIs)
2. Certificate transparency (crt.sh)
3. Brute-force (if enabled)
4. Permutations (if enabled)
5. All sources merged and deduplicated

### webs/webs.txt

Live web servers (responding HTTP/HTTPS).

```
https://www.example.com
https://api.example.com
http://staging.example.com:8080
```

**Filtering applied:**
- DNS resolves ✓
- TCP connection succeeds ✓
- HTTP response received ✓

### webs/webs_all.txt

Detailed web server information.

```
https://www.example.com [200] [Example Corp] [nginx/1.18] [text/html]
https://api.example.com [401] [API Gateway] [kong/2.5] [application/json]
```

Format: `URL [status] [title] [server] [content-type]`

### hosts/ips.txt

All resolved IP addresses (CDN filtered).

```
93.184.216.34
93.184.216.35
```

### vulns/nuclei_output.json

Machine-readable vulnerability data.

```json
{
  "template-id": "cve-2021-44228",
  "name": "Log4j RCE",
  "severity": "critical",
  "host": "https://api.example.com",
  "matched-at": "https://api.example.com/search",
  "extracted-results": ["${jndi:ldap://...}"],
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**Useful jq queries:**
```bash
# Get critical vulns
cat nuclei_output.json | jq -r 'select(.severity=="critical")'

# List unique templates triggered
cat nuclei_output.json | jq -r '."template-id"' | sort -u

# Count by severity
cat nuclei_output.json | jq -r '.severity' | sort | uniq -c
```

### hotlist.txt

Risk-scored priority findings.

```
[CRITICAL] https://api.example.com/admin - nuclei:cve-2021-44228
[HIGH] staging.example.com - subdomain takeover
[MEDIUM] https://www.example.com - cors misconfiguration
```

---

## Module Input/Output Matrix

| Module | Primary Input | Key Outputs |
|--------|---------------|-------------|
| **OSINT** | Domain | `osint/emails.txt`, `osint/github_*.json`, `osint/dorks.txt` |
| **Subdomains** | Domain | `subdomains/subdomains.txt`, `subdomains/takeover.txt` |
| **Web Probing** | Subdomains | `webs/webs.txt`, `webs/webs_all.txt` |
| **URL Collection** | Live webs | `webs/url_extract.txt`, `js/js_endpoints.txt` |
| **Fuzzing** | Live webs | `fuzzing/fuzzing_full.txt` |
| **Ports** | IPs | `hosts/portscan_active.txt`, `hosts/portscan_active.xml` |
| **Vulns** | URLs | `vulns/nuclei_output.json`, `vulns/*.txt` |

---

## Data Flow

```
┌─────────────┐
│   INPUT     │
│  -d / -l    │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────────────────────┐
│              OSINT MODULE                        │
│  Domain → emails, dorks, GitHub leaks           │
└──────┬──────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────┐
│           SUBDOMAIN MODULE                       │
│  Domain → subdomains.txt (merged, deduped)      │
└──────┬──────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────┐
│            WEB MODULE                            │
│  Subdomains → webs.txt (live servers)           │
│  Live webs → URLs, JS files, screenshots        │
└──────┬──────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────┐
│           HOST MODULE                            │
│  IPs → port scans, CDN detection                │
└──────┬──────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────┐
│           VULN MODULE                            │
│  URLs → nuclei, XSS, SQLi, etc.                 │
└──────┬──────────────────────────────────────────┘
       │
       ▼
┌─────────────┐
│   OUTPUT    │
│ Recon/domain│
└─────────────┘
```

---

## File Formats

### Text Lists (.txt)

One item per line, no headers:
```
item1
item2
item3
```

### JSON Output (.json)

NDJSON format (one JSON object per line):
```json
{"host":"example.com","finding":"vuln1"}
{"host":"example.org","finding":"vuln2"}
```

### XML Output (.xml)

Standard Nmap XML for port scans:
```xml
<?xml version="1.0"?>
<nmaprun scanner="nmap">
  <host>
    <address addr="93.184.216.34"/>
    <ports>
      <port portid="443" protocol="tcp">
        <state state="open"/>
        <service name="https"/>
      </port>
    </ports>
  </host>
</nmaprun>
```

### CSV Output

Some tools output CSV (httpx with custom flags):
```csv
url,status_code,title,tech
https://example.com,200,Example,nginx
```

---

## Checkpoint System

The `.called_fn/` directory tracks completed functions.

### How It Works

```bash
# After sub_passive() completes:
ls Recon/example.com/.called_fn/
# sub_passive

# After more functions:
# sub_passive  sub_brute  webprobe_simple
```

### Resuming Scans

If interrupted, reconFTW skips completed functions:
```bash
# Scan interrupted at 50%
./reconftw.sh -d example.com -r
# Resumes from where it stopped
```

### Force Re-run

```bash
# Delete specific checkpoint
rm Recon/example.com/.called_fn/nuclei_check

# Delete all checkpoints (full re-scan)
rm -rf Recon/example.com/.called_fn/
```

---

## Log Files

### Main Log

`Recon/<domain>/.log/reconftw.log`

```
[2024-01-15 10:30:00] [INFO] Starting reconnaissance for example.com
[2024-01-15 10:30:05] [INFO] Running sub_passive
[2024-01-15 10:35:00] [INFO] sub_passive found 150 subdomains
[2024-01-15 10:35:01] [INFO] Running sub_brute
[2024-01-15 10:45:00] [ERROR] nuclei timeout on https://slow.example.com
```

### Debug Mode

```bash
DEBUG=true ./reconftw.sh -d example.com -r
```

Outputs verbose tool commands and intermediate results.

---

## Aggregated Outputs

### assets.jsonl

Machine-readable asset inventory:
```json
{"type":"subdomain","value":"api.example.com","source":"subfinder"}
{"type":"url","value":"https://api.example.com/v1","source":"katana"}
{"type":"vulnerability","value":"cve-2021-44228","host":"api.example.com"}
```

### HTML Report (with AI)

When using `-y/--ai`:
```
Recon/example.com/
└── report.html    # AI-generated summary report
```

---

## Integration Exports

### Faraday

Automatic import when configured:
- Nuclei JSON → Faraday vulnerabilities
- Host info → Faraday hosts
- Services → Faraday services

### Custom Export

```bash
# Export to CSV
cat vulns/nuclei_output.json | jq -r '[.host, .severity, ."template-id"] | @csv'

# Export subdomains for other tools
cat subdomains/subdomains.txt | httpx -silent > live.txt
```

---

## Common Data Operations

### Find All Critical Vulns

```bash
cat Recon/*/vulns/nuclei_output.json | jq -r 'select(.severity=="critical")'
```

### Merge Results from Multiple Scans

```bash
# Merge subdomains
cat Recon/*/subdomains/subdomains.txt | sort -u > all_subs.txt

# Merge vulnerabilities
cat Recon/*/vulns/nuclei_output.json > all_vulns.json
```

### Generate Statistics

```bash
# Count subdomains per domain
for d in Recon/*/; do 
  echo "$(basename $d): $(wc -l < $d/subdomains/subdomains.txt)"
done

# Vulnerability summary
cat Recon/*/vulns/nuclei_output.json | jq -r '.severity' | sort | uniq -c
```

---

## Storage Requirements

Approximate disk usage per scan:

| Target Size | Storage |
|-------------|---------|
| Small (<100 subs) | 50-200 MB |
| Medium (100-1K subs) | 200-500 MB |
| Large (1K-10K subs) | 500 MB - 2 GB |
| Massive (10K+ subs) | 2-10 GB |

**Space-heavy components:**
- Screenshots (~500KB each)
- Fuzzing results (large wordlists)
- Full port scan XML

**Cleanup command:**
```bash
# Remove temporary files
rm -rf Recon/example.com/.tmp/
```

---

> **Documentation Info**  
> Branch: `dev` | Version: `v3.0.0+` | Last updated: February 2026
