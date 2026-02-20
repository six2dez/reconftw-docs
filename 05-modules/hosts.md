# Host Analysis Module

The host analysis module examines the infrastructure behind discovered assets, including port scanning, CDN detection, WAF identification, cloud enumeration, and geolocation.

---

## Module Overview

| Function | Purpose | Tools |
|----------|---------|-------|
| `portscan` | Port discovery (passive + active) | naabu (optional), nmap, smap |
| `service_fingerprint` | Service fingerprinting on open ports | fingerprintx |
| `cdnprovider` | CDN detection and filtering | cdncheck, hakoriginfinder (optional) |
| `waf_checks` | WAF detection | wafw00f |
| `geo_info` | IP geolocation | ipinfo |

> `favirecon_tech` (favicon technology fingerprinting) is executed in the Web Analysis module and writes to `webs/favirecon.[json|txt]`.

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
PORTSCAN_ACTIVE_OPTIONS="--top-ports 200 -sV -n -Pn --open --max-retries 2"
PORTSCAN_DEEP_OPTIONS="--top-ports 1000 -sV -n -Pn --open --max-retries 2 --script vulners"

# Strategy: legacy nmap-only OR naabu discovery + targeted nmap
PORTSCAN_STRATEGY=legacy     # legacy|naabu_nmap
NAABU_ENABLE=true
NAABU_RATE=1000
NAABU_PORTS="--top-ports 1000"
SERVICE_FINGERPRINT=true
SERVICE_FINGERPRINT_ENGINE="fingerprintx"
SERVICE_FINGERPRINT_TIMEOUT_MS=2000

# Optional UDP scan (requires privileges on most systems)
PORTSCAN_UDP=false
PORTSCAN_UDP_OPTIONS="--top-ports 20 -sU -sV -n -Pn --open"

# Other host checks
CDN_IP=true               # CDN detection
CDN_BYPASS=true           # Optional origin IP discovery with hakoriginfinder
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
→ Scan ports → Service detection → (optional CVE enrichment in deep profile) → Output
```

**Default Scan Options:**
```bash
PORTSCAN_ACTIVE_OPTIONS="--top-ports 200 -sV -n -Pn --open --max-retries 2"
PORTSCAN_DEEP_OPTIONS="--top-ports 1000 -sV -n -Pn --open --max-retries 2 --script vulners"
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
| `--script vulners` | Optional (deep profile) CVE enrichment |

**Output:**
```
hosts/portscan_active.txt      # Human readable
hosts/portscan_active.xml      # Nmap XML format
hosts/portscan_active.gnmap    # Greppable format
hosts/portscan_active_targeted.xml   # When PORTSCAN_STRATEGY=naabu_nmap
hosts/portscan_active_udp.xml        # When PORTSCAN_UDP=true
hosts/naabu_open.txt                 # When PORTSCAN_STRATEGY=naabu_nmap
```

**Sample Output:**
```
Nmap scan report for 192.168.1.10
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 8.2p1
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

# Full port scan (all ports)
PORTSCAN_ACTIVE_OPTIONS="-p- -sV -sC -Pn --open"

# Stealth scan (slower, less detectable)
PORTSCAN_ACTIVE_OPTIONS="-sS -T2 --top-ports 1000 -Pn"
```

### Two-Stage Strategy (naabu + nmap)

When `PORTSCAN_STRATEGY=naabu_nmap`, reconFTW uses a 2-stage flow:

1. `naabu` quickly discovers open TCP ports and writes `hosts/naabu_open.txt`.
2. `nmap` runs only against discovered ports and writes `hosts/portscan_active_targeted.xml` (and related `-oA` outputs).

This keeps coverage while reducing total `nmap` time on medium/large targets.

### Service Fingerprinting

When `SERVICE_FINGERPRINT=true`, reconFTW runs fingerprinting against discovered `host:port` pairs and stores normalized service metadata.

**Output:**
```
hosts/fingerprintx.jsonl
hosts/fingerprintx.txt
```

### Optional UDP Scan

When `PORTSCAN_UDP=true`, reconFTW runs an additional UDP scan (separate output `hosts/portscan_active_udp.xml`). UDP scanning usually requires elevated privileges depending on OS/config.

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
hosts/cdn_providers.txt # Raw cdncheck classifications (cdn/waf hints)
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
webs/webs_wafs.txt
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

## CDN Origin Discovery (Optional)

### hakoriginfinder - Origin IP Candidates

When `CDN_BYPASS=true`, reconFTW can run `hakoriginfinder` as a best-effort attempt to discover origin IP candidates for CDN-fronted assets.

**How It Works:**

```
subdomains.txt (hosts) → hakoriginfinder → candidate origin IPs → hosts/origin_ips.txt
```

**Output:**
```
hosts/origin_ips.txt
```

**Notes / Limitations:**
- Not guaranteed (depends on the target, DNS history, and public data).
- Treat results as candidates: validate scope and ownership before scanning aggressively.
- Origin IPs are automatically added back to the scanning pipeline (they become part of `hosts/ips.txt` / `.tmp/ips_nocdn.txt`).

**Configuration:**
```bash
CDN_BYPASS=true
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
│ portscan_  portscan_  webs_wafs.txt   geo.txt                       │
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
| `hosts/cdn_providers.txt` | CDN/WAF classification output from cdncheck |
| `hosts/origin_ips.txt` | Origin IP candidates (when `CDN_BYPASS=true`) |
| `hosts/portscan_passive.txt` | Shodan port results |
| `hosts/naabu_open.txt` | Open ports discovered by naabu (when `PORTSCAN_STRATEGY=naabu_nmap`) |
| `hosts/portscan_active.txt` | Nmap scan results |
| `hosts/portscan_active.xml` | Nmap XML output |
| `hosts/portscan_active.gnmap` | Nmap greppable output |
| `hosts/portscan_active_targeted.xml` | Targeted Nmap XML output (when `PORTSCAN_STRATEGY=naabu_nmap`) |
| `hosts/portscan_active_udp.xml` | UDP Nmap XML output (when `PORTSCAN_UDP=true`) |
| `hosts/fingerprintx.jsonl` | Service fingerprints (raw JSONL) |
| `hosts/fingerprintx.txt` | Service fingerprints (normalized text) |
| `webs/webs_wafs.txt` | WAF detection results |
| `hosts/geo.txt` | Geolocation data |
| `hosts/grpc_reflection.txt` | gRPC services with reflection |

---

## Integration with Vulnerability Scanning

Host analysis results feed into vulnerability scanning:

1. **Port scan results** → Password spraying targets
2. **Service versions** → Optional CVE enrichment (deep profile via `PORTSCAN_DEEP_OPTIONS`)
3. **Non-CDN IPs + origin candidates** → Focus active testing on likely real infrastructure
4. **WAF detection** → Adjust rate limits/attack strategies

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
