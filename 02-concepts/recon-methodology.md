# Reconnaissance Methodology

This guide explains the reasoning behind reconFTW's reconnaissance approach.

---

## Why This Order?

reconFTW executes modules in a specific sequence based on data dependencies and resource efficiency.

### 1. OSINT First

**Functions:** `google_dorks`, `github_dorks`, `metadata`, `emails`, `domain_info`

**Why first:**
- Zero target interaction (completely passive)
- Provides context for later phases
- May reveal additional scope (related domains, acquisitions)
- Fast execution with no rate limiting concerns

**What it reveals:**
- Exposed files via search engines
- Leaked credentials in code repositories
- Document metadata (usernames, software versions)
- Email patterns for social engineering
- Domain registration history

### 2. Subdomain Enumeration Second

**Functions:** `sub_passive`, `sub_crt`, `sub_active`, `sub_brute`, `sub_permut`

**Why second:**
- Defines the attack surface
- Required input for all subsequent phases
- Passive sources first (no detection), active later

**Data flow:**
```
Passive sources → Merge → Resolve → Filter wildcards → Active enum
                                   ↓
                         subdomains/subdomains.txt
```

### 3. Web Probing Third

**Functions:** `webprobe_simple`, `webprobe_full`

**Why third:**
- Requires subdomain list as input
- Identifies HTTP/HTTPS services
- Filters to only web-accessible hosts

**Output:** `webs/webs.txt` - URLs of live web services

### 4. Host Analysis Fourth

**Functions:** `portscan`, `cdnprovider`, `favicon`, `geo_info`

**Why fourth:**
- Operates on resolved IPs
- Identifies services beyond HTTP
- CDN detection affects later scanning decisions

### 5. Web Analysis Fifth

**Functions:** `urlchecks`, `jschecks`, `fuzz`, `cms_scanner`

**Why fifth:**
- Requires live web URLs
- Discovers endpoints for vulnerability testing
- JavaScript analysis reveals API endpoints

### 6. Vulnerability Scanning Last

**Functions:** `nuclei_check`, `xss`, `sqli`, `ssrf_checks`

**Why last:**
- Most intrusive phase
- Requires discovered URLs and parameters
- Generates alerts/logs on target systems

---

## Passive vs Active

### Passive Reconnaissance

Operations that do NOT directly contact the target.

| Technique | Data Source | Detection Risk |
|-----------|-------------|----------------|
| CT log queries | crt.sh, Censys | None |
| DNS database queries | SecurityTrails, VirusTotal | None |
| Search engine dorks | Google, Bing | None |
| Code repository search | GitHub, GitLab | None |
| WHOIS lookups | WHOIS servers | None |

**reconFTW passive mode:** `./reconftw.sh -d target.com -p`

### Active Reconnaissance

Operations that directly interact with target systems.

| Technique | Target Contact | Detection Risk |
|-----------|----------------|----------------|
| DNS resolution | Target DNS servers | Low |
| HTTP probing | Target web servers | Medium |
| Port scanning | Target hosts | High |
| Directory fuzzing | Target web servers | High |
| Vulnerability scanning | Target applications | High |

---

## The Checkpoint System

reconFTW tracks completed functions to enable resumption.

### How It Works

Each function creates a marker file on completion:
```
Recon/target.com/.called_fn/
├── .sub_passive     # sub_passive() completed
├── .sub_crt         # sub_crt() completed
├── .webprobe_simple # webprobe_simple() completed
└── ...
```

### Resume Behavior

On subsequent runs:
1. Check if marker exists for function
2. If exists and `DIFF=false` → Skip function
3. If missing or `DIFF=true` → Execute function

### Clearing Checkpoints

```bash
# Re-run single function
rm Recon/target.com/.called_fn/.sub_passive

# Re-run all
rm -rf Recon/target.com/.called_fn/
```

---

## Wildcard Handling

Wildcard DNS records return valid responses for any subdomain query.

### The Problem

```
*.example.com → 1.2.3.4

Query: randomstring.example.com
Result: 1.2.3.4 (valid response)
```

Without filtering, bruteforce returns millions of false positives.

### Standard Detection

Query random string, if resolves → wildcard exists at root level.

### Deep Detection (DEEP_WILDCARD_FILTER)

Enterprises use nested wildcards:
```
*.api.example.com → 1.2.3.4
*.na45.salesforce.com → 5.6.7.8
```

Deep detection iteratively tests parent domains at all levels.

---

## Rate Limiting Strategy

### Why Rate Limit?

- Avoid overwhelming target infrastructure
- Prevent IP blocks and WAF bans
- Stay within API quotas
- Reduce detection risk

### Configuration

```bash
# Global rate limit
./reconftw.sh -d target.com -r -q 50  # 50 requests/second

# Per-tool limits
HTTPX_RATELIMIT=50
NUCLEI_RATELIMIT=150
FFUF_RATELIMIT=50
```

### Adaptive Rate Limiting

```bash
./reconftw.sh -d target.com -r --adaptive-rate
```

Automatically reduces rate on:
- HTTP 429 (Too Many Requests)
- HTTP 503 (Service Unavailable)
- Connection timeouts

---

## Scope Management

### In-Scope Filtering

```bash
# inscope.txt
*.example.com
api.example.com
staging.example.com
```

Enable: `INSCOPE=true`

### Out-of-Scope Exclusion

```bash
# outofscope.txt
admin.example.com
vpn.example.com
*.internal.example.com
```

Usage: `./reconftw.sh -d example.com -r -x outofscope.txt`

### Sensitive Domain Exclusion

Prevents scanning government, military, and critical infrastructure:

```bash
EXCLUDE_SENSITIVE=true
```

Patterns defined in `config/sensitive_domains.txt`.

---

## Output Organization

```
Recon/target.com/
├── subdomains/
│   ├── subdomains.txt      # All discovered subdomains
│   ├── subdomains_new.txt  # New since last run (incremental)
│   └── wildcards_detected.txt
├── webs/
│   ├── webs.txt            # Live web URLs
│   └── webscreenshot/      # Screenshots
├── hosts/
│   ├── ips.txt             # Resolved IPs
│   └── portscan/           # Nmap results
├── osint/
│   ├── emails.txt
│   └── github_dorks.txt
├── vulns/
│   ├── nuclei_output/
│   └── xss.txt
├── .tmp/                   # Temporary files
├── .log/                   # Execution logs
└── .called_fn/             # Checkpoint markers
```

---

## Tool Selection Rationale

reconFTW integrates specific tools for each task:

| Task | Tool | Why This Tool |
|------|------|---------------|
| Passive subdomains | subfinder | 50+ sources, fast, maintained |
| DNS resolution | puredns + massdns | Fastest resolver with wildcard filtering |
| HTTP probing | httpx | Feature-rich, handles edge cases |
| Screenshots | webscreenshot | Headless Chrome, reliable |
| Port scanning | nmap | Industry standard, scriptable |
| Fuzzing | ffuf | Fast, flexible, mature |
| Vuln scanning | nuclei | Template-based, community templates |

Tools are selected based on:
- Speed and reliability
- Active maintenance
- Community adoption
- Output format compatibility
