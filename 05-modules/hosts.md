# Host Analysis Module

The host analysis module examines the infrastructure behind discovered assets, including port scanning, CDN detection, WAF identification, cloud enumeration, and geolocation.

---

## Module Overview

| Function | Purpose | Tools |
|----------|---------|-------|
| `portscan` | Port discovery (passive + active) | nmap, smap |
| `cdnprovider` | CDN detection and filtering | cdncheck |
| `waf_checks` | WAF detection | wafw00f |
| `favicon` | Real IP discovery via favicon | fav-up |
| `cloud_extra_providers` | Extra cloud storage enumeration | curl |
| `geo_info` | IP geolocation | ipinfo |
| `banner_grabber` | Service banner extraction | nmap |

---

## Configuration Options

```bash
# In reconftw.cfg

# Master toggle
PORTSCANNER=true

# Port scanning
PORTSCAN_PASSIVE=true     # Shodan-based (requires API key)
PORTSCAN_ACTIVE=true      # Nmap-based

# Nmap options
PORTSCAN_ACTIVE_OPTIONS="--top-ports 200 -sV -n -Pn --open --max-retries 2 --script vulners"

# Other host checks
FAVICON=true              # Favicon IP discovery
CDN_IP=true               # CDN detection
GEO_INFO=true             # Geolocation
WAF_DETECTION=true        # WAF detection

# IPv6
IPV6_SCAN=true            # IPv6 discovery
```

---

## Port Scanning

### Passive Port Scanning (Shodan)

Uses Shodan API to retrieve port/service information without touching the target.

**How It Works:**

```
IP addresses → smap (Shodan query) → 
→ Return known open ports → hosts/portscan_passive.txt
```

**Advantages:**
- No direct target interaction
- Historical data available
- Fast results

**Limitations:**
- Requires Shodan API key
- Data may be outdated
- Only indexed hosts

**Output:**
```
hosts/portscan_passive.txt
```

**Sample Output:**
```
192.168.1.10
  22/tcp    ssh         OpenSSH 8.2
  80/tcp    http        nginx 1.18
  443/tcp   https       nginx 1.18
  3306/tcp  mysql       MySQL 8.0
```

**Configuration:**
```bash
PORTSCAN_PASSIVE=true
SHODAN_API_KEY="your_key"  # In secrets.cfg
```

---

### Active Port Scanning (Nmap)

Performs direct port scanning against target IPs.

**How It Works:**

```
IP addresses (non-CDN) → nmap → 
→ Scan ports → Service detection → Vuln scripts → Output
```

**Default Scan Options:**
```bash
PORTSCAN_ACTIVE_OPTIONS="--top-ports 200 -sV -n -Pn --open --max-retries 2 --script vulners"
```

**Option Breakdown:**
| Option | Description |
|--------|-------------|
| `--top-ports 200` | Scan top 200 ports |
| `-sV` | Version detection |
| `-n` | No DNS resolution |
| `-Pn` | Skip host discovery |
| `--open` | Show only open ports |
| `--max-retries 2` | Retry limit |
| `--script vulners` | Check for CVEs |

**Output:**
```
hosts/portscan_active.txt      # Human readable
hosts/portscan_active.xml      # Nmap XML format
hosts/portscan_active.gnmap    # Greppable format
```

**Sample Output:**
```
Nmap scan report for 192.168.1.10
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 8.2p1
| vulners: 
|   CVE-2020-15778  6.8
80/tcp    open  http       nginx 1.18.0
443/tcp   open  ssl/https  nginx 1.18.0
3306/tcp  open  mysql      MySQL 8.0.26
```

**Configuration:**
```bash
PORTSCAN_ACTIVE=true
PORTSCAN_ACTIVE_OPTIONS="--top-ports 200 -sV -n -Pn --open"
```

**Custom Scan Profiles:**

```bash
# Quick scan (top 100 ports)
PORTSCAN_ACTIVE_OPTIONS="--top-ports 100 -Pn --open"

# Comprehensive scan (all ports)
PORTSCAN_ACTIVE_OPTIONS="-p- -sV -sC -Pn --open"

# Stealth scan (slower, less detectable)
PORTSCAN_ACTIVE_OPTIONS="-sS -T2 --top-ports 1000 -Pn"
```

---

## CDN Detection

### `cdnprovider` - CDN Identification

Identifies IPs behind CDN providers to avoid scanning CDN infrastructure.

**Why It Matters:**
- CDN IPs don't represent the actual target
- Scanning CDNs wastes resources
- May violate CDN terms of service
- Focus on real infrastructure

**CDN Providers Detected:**
- Cloudflare
- Akamai
- Fastly
- CloudFront
- Incapsula
- And many more...

**How It Works:**

```
All IPs → cdncheck → 
→ Identify CDN IPs → Separate into cdn.txt and non-cdn IPs
```

**Output:**
```
hosts/cdn.txt          # CDN IPs (excluded from active scans)
hosts/ips.txt          # Non-CDN IPs (scanned)
```

**Sample Output:**
```
# cdn.txt
104.16.132.229 [cloudflare]
151.101.1.195 [fastly]
13.32.123.45 [cloudfront]

# ips.txt (non-CDN, will be scanned)
192.168.1.10
10.0.0.5
```

**Configuration:**
```bash
CDN_IP=true
```

---

## WAF Detection

### `waf_checks` - Web Application Firewall Detection

Identifies WAF/security products protecting web applications.

**WAFs Detected:**
- Cloudflare
- AWS WAF
- Akamai
- Imperva/Incapsula
- ModSecurity
- Sucuri
- F5 BIG-IP
- And 100+ more...

**How It Works:**

```
webs.txt → wafw00f → 
→ Send probe requests → Analyze responses → Identify WAF
```

**Output:**
```
hosts/waf.txt
```

**Sample Output:**
```
https://www.example.com
  WAF: Cloudflare
  Detected by: Response headers, Server header
  
https://api.example.com
  WAF: AWS WAF
  Detected by: Response behavior
  
https://admin.example.com
  WAF: None detected
```

**Why It Matters:**
- Adjust attack strategies
- Understand defensive posture
- Identify bypass opportunities
- Report in findings

**Configuration:**
```bash
WAF_DETECTION=true
```

---

## Favicon Analysis

### `favicon` - Real IP Discovery

Discovers real IP addresses behind CDN/proxy by analyzing favicon hashes.

**How It Works:**

```
Known favicon hash → Shodan search → 
→ Find servers with same favicon → Potential real IPs
```

**Technique:**
1. Download favicon from target
2. Calculate hash (MurmurHash3)
3. Search Shodan for matching hashes
4. Servers with same favicon may be the real origin

**Output:**
```
hosts/favicontest.txt
```

**Sample Output:**
```
# Favicon hash: -1234567890
Real IP candidates:
  192.168.1.100 (Direct match)
  10.0.0.50 (Partial match)
```

**Configuration:**
```bash
FAVICON=true
SHODAN_API_KEY="your_key"  # Required
```

---

## Geolocation

### `geo_info` - IP Geolocation

Retrieves geographic information for discovered IP addresses.

**Information Gathered:**
- Country
- Region/State
- City
- Organization/ISP
- ASN

**How It Works:**

```
IP addresses → ipinfo.io API → 
→ Geolocation data → hosts/geo.txt
```

**Output:**
```
hosts/geo.txt
```

**Sample Output:**
```
192.168.1.10
  Country: United States
  Region: California
  City: San Francisco
  Org: Example Hosting Inc.
  ASN: AS12345
  
10.0.0.5
  Country: Germany
  Region: Hesse
  City: Frankfurt
  Org: AWS
  ASN: AS16509
```

**Use Cases:**
- Understand infrastructure distribution
- Identify hosting providers
- Compliance/jurisdiction issues
- Attack surface mapping

**Configuration:**
```bash
GEO_INFO=true
```

---

## Cloud Storage Enumeration

### `cloud_extra_providers` - Extra Cloud Provider Checks

Discovers misconfigured cloud storage buckets beyond standard S3 enumeration.

**Cloud Providers Checked:**
- **Google Cloud Storage (GCS)**
  - `https://storage.googleapis.com/{name}/`
  - `https://{name}.storage.googleapis.com/`
- **Azure Blob Storage**
  - `https://{name}.blob.core.windows.net/{container}`
  - Tests common containers: public, static, media, images, assets, backup, files, cdn

**How It Works:**

```
Domain → Extract company/brand names → 
→ Generate candidate bucket names → Test cloud URLs →
→ Check for public access (200/403) → Report findings
```

**Name Generation:**
1. Domain root (e.g., "example" from "example.com")
2. Company name variations
3. Subdomain prefixes (e.g., "api", "dev", "staging")
4. Combined with common container names

**Output:**
```
subdomains/cloud_extra.txt
```

**Sample Output:**
```
GCS examplecorp https://storage.googleapis.com/examplecorp/ 
GCS example-backup https://example-backup.storage.googleapis.com/
AZURE examplecorp/public https://examplecorp.blob.core.windows.net/public
AZURE example-dev/static https://example-dev.blob.core.windows.net/static
```

**What It Finds:**
- Publicly accessible storage buckets
- Buckets returning 403 (exist but restricted - worth manual testing)
- Misconfigured backup/static file storage

**Security Implications:**
- Public buckets may contain sensitive data
- 403 responses confirm bucket existence (enumeration value)
- Backup buckets often contain valuable data

**Note:** This complements the OSINT module's `cloud_enum` function by testing additional providers and name variations.

---

## IPv6 Scanning

### IPv6 Discovery and Scanning

Discovers and scans IPv6 addresses when available.

**How It Works:**

```
Subdomains → DNS AAAA records → 
→ IPv6 addresses → Include in scanning
```

**Configuration:**
```bash
IPV6_SCAN=true
```

**Note:** IPv6 scanning may reveal additional attack surface not visible via IPv4.

---

## Data Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Host Analysis Flow                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────┐                                                  │
│  │ subdomains.txt │                                                  │
│  └───────┬────────┘                                                  │
│          │                                                           │
│          ▼                                                           │
│  ┌────────────────┐                                                  │
│  │  DNS Resolution│ (sub_dns)                                        │
│  └───────┬────────┘                                                  │
│          │                                                           │
│          ▼                                                           │
│  ┌────────────────┐                                                  │
│  │   All IPs      │                                                  │
│  └───────┬────────┘                                                  │
│          │                                                           │
│          ▼                                                           │
│  ┌────────────────┐     ┌────────────────┐                          │
│  │  cdnprovider   │────▶│    cdn.txt     │ (excluded)               │
│  └───────┬────────┘     └────────────────┘                          │
│          │                                                           │
│          ▼                                                           │
│  ┌────────────────┐                                                  │
│  │  Non-CDN IPs   │                                                  │
│  └───────┬────────┘                                                  │
│          │                                                           │
│    ┌─────┴─────┬─────────────┬─────────────┐                        │
│    ▼           ▼             ▼             ▼                        │
│ ┌──────┐  ┌────────┐   ┌─────────┐   ┌─────────┐                   │
│ │ nmap │  │ shodan │   │ wafw00f │   │ geo_info│                   │
│ │active│  │passive │   │  (WAF)  │   │  (geo)  │                   │
│ └──┬───┘  └───┬────┘   └────┬────┘   └────┬────┘                   │
│    │          │             │             │                         │
│    ▼          ▼             ▼             ▼                         │
│ portscan_  portscan_     waf.txt      geo.txt                       │
│ active.txt passive.txt                                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Output Files Summary

| File | Content |
|------|---------|
| `hosts/ips.txt` | All resolved IP addresses |
| `hosts/cdn.txt` | IPs identified as CDN |
| `hosts/portscan_passive.txt` | Shodan port results |
| `hosts/portscan_active.txt` | Nmap scan results |
| `hosts/portscan_active.xml` | Nmap XML output |
| `hosts/portscan_active.gnmap` | Nmap greppable output |
| `hosts/waf.txt` | WAF detection results |
| `hosts/geo.txt` | Geolocation data |
| `hosts/favicontest.txt` | Favicon real IP discovery |
| `hosts/grpc_reflection.txt` | gRPC services with reflection |
| `subdomains/cloud_extra.txt` | Extra cloud storage findings |

---

## Integration with Vulnerability Scanning

Host analysis results feed into vulnerability scanning:

1. **Port scan results** → Password spraying targets
2. **Service versions** → CVE matching (vulners script)
3. **Non-CDN IPs** → Focus active testing
4. **WAF detection** → Adjust attack strategies

---

## Best Practices

1. **CDN Awareness:** Don't waste resources scanning CDN IPs

2. **Rate Limiting:** Aggressive port scans can trigger alerts

3. **Authorization:** Ensure port scanning is in scope

4. **Service Detection:** Version info helps identify vulnerabilities

5. **Passive First:** Start with Shodan to minimize noise

6. **IPv6 Coverage:** Don't forget IPv6 attack surface

---

## Nmap Scan Customization

### Quick Discovery
```bash
PORTSCAN_ACTIVE_OPTIONS="--top-ports 100 -Pn -T4"
```

### Full Scan
```bash
PORTSCAN_ACTIVE_OPTIONS="-p- -sV -sC -Pn"
```

### Stealth Scan
```bash
PORTSCAN_ACTIVE_OPTIONS="-sS -T2 --top-ports 1000 -Pn"
```

### Service Focus
```bash
PORTSCAN_ACTIVE_OPTIONS="-p 21,22,23,25,80,443,3306,5432,6379,27017 -sV -Pn"
```

---

## Next Steps

- **[Tools Reference](../06-tools/tools.md)** - All integrated tools
- **[Output Interpretation](../07-output/output.md)** - Understand results
