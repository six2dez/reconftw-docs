# Case Studies

Real-world examples of reconFTW in action.

---

## Case Study 1: Bug Bounty - New Program Launch

**Scenario:** A new bug bounty program launched with `*.target.com` in scope. Goal is to find vulnerabilities quickly before other hunters.

### Target Profile
- **Scope:** `*.target.com` (wildcard)
- **Size:** Unknown (new program)
- **Time:** 4 hours before others catch up
- **Goal:** Quick wins - subdomain takeovers, exposed panels, known CVEs

### Configuration

```bash
# Custom config: bounty-rush.cfg

# Fast subdomain enumeration
SUBBRUTE=false           # Skip brute-force (time-consuming)
SUBPERMUTE=false         # Skip permutations
SUBIAPERMUTE=false

# Essential web checks only
WEBPROBESIMPLE=true
WEBPROBEFULL=false       # Skip detailed probing

# Priority vulnerability checks
NUCLEICHECK=true
NUCLEI_SEVERITY="critical,high"  # Only high-impact
SUBTAKEOVER=true         # Quick wins

# Skip slow modules
FUZZ=false
XSS=false
SQLI=false
SSRF_CHECKS=false
```

### Execution

```bash
# Phase 1: Quick passive recon (15 minutes)
./reconftw.sh -d target.com -p

# Review subdomains found
wc -l Recon/target.com/subdomains/subdomains.txt
# 847 subdomains

# Phase 2: Fast vulnerability scan (2 hours)
./reconftw.sh -d target.com -r -f bounty-rush.cfg --adaptive-rate

# Phase 3: Check results immediately
cat Recon/target.com/subdomains/takeover.txt
cat Recon/target.com/nuclei_output/critical_json.txt | jq -r '.'
```

### Results

| Finding | Severity | File Location |
|---------|----------|---------------|
| Subdomain takeover on `old.target.com` | High | `subdomains/takeover.txt` |
| Exposed Jenkins on `ci.target.com` | Critical | `nuclei_output/critical_json.txt` |
| S3 bucket listing | Medium | `osint/cloud_enum.txt` |
| 3 exposed admin panels | High | `nuclei_output/high_json.txt` |

**Time to first finding:** 23 minutes (subdomain takeover)

### Key Takeaways

1. **Passive first** - Always run `-p` before full recon
2. **Skip slow modules** - Brute-force can wait
3. **Focus on quick wins** - Takeovers, panels, known CVEs
4. **Review as you go** - Don't wait for full scan to complete

---

## Case Study 2: Security Assessment - Enterprise Client

**Scenario:** Contracted for a security assessment of a Fortune 500 company. Need full coverage with minimal disruption.

### Target Profile
- **Scope:** `corp.example.com` + IP range `10.0.0.0/8` (internal)
- **Size:** Large enterprise (~5,000 subdomains expected)
- **Time:** 1 week engagement
- **Goal:** Complete asset inventory, all vulnerabilities documented

### Pre-Engagement Setup

```bash
# Create scope files
echo "*.corp.example.com" > scope-in.txt
echo "*.internal.example.com" >> scope-in.txt

echo "vpn.corp.example.com" > scope-out.txt
echo "mail.corp.example.com" >> scope-out.txt
```

### Configuration

```bash
# Custom config: enterprise-assessment.cfg

# Thorough enumeration
SUBBRUTE=true
subs_wordlist="${tools}/subdomains_n0kovo_big.txt"
SUBPERMUTE=true
SUBIAPERMUTE=true

# All web checks
WEBPROBESIMPLE=true
WEBPROBEFULL=true
SCREENSHOT=true

# Full vulnerability assessment
NUCLEICHECK=true
NUCLEI_SEVERITY="critical,high,medium"
FUZZ=true
XSS=true
SQLI=true
SSRF_CHECKS=true
CORS=true

# Rate limiting (be respectful)
HTTPX_RATELIMIT=50
NUCLEI_RATELIMIT=100
ADAPTIVE_RATE_LIMIT=true

# Save everything
PORTSCANNER=true
PORTSCAN_ACTIVE=true
```

### Execution Plan

**Day 1-2: Reconnaissance**
```bash
# External reconnaissance
./reconftw.sh -d corp.example.com -r \
  -i scope-in.txt -x scope-out.txt \
  -f enterprise-assessment.cfg

# OSINT deep dive
./reconftw.sh -d corp.example.com -n  # OSINT only
```

**Day 3-4: Vulnerability Assessment**
```bash
# Full vulnerability scan
./reconftw.sh -d corp.example.com -a \
  -i scope-in.txt -x scope-out.txt \
  -f enterprise-assessment.cfg

# Review and prioritize
cat Recon/corp.example.com/nuclei_output/*_json.txt | \
  jq -r 'select(.severity=="critical")' > critical_findings.json
```

**Day 5: Deep Dive on Findings**
```bash
# Re-scan specific high-value targets
./reconftw.sh -d api.corp.example.com -a --deep

# Manual verification of critical findings
```

### Results Summary

| Category | Count | Files |
|----------|-------|-------|
| Subdomains | 4,892 | `subdomains/subdomains.txt` |
| Live web servers | 1,247 | `webs/webs.txt` |
| Open ports | 8,432 | `hosts/portscan_active.txt` |
| Critical vulns | 12 | `nuclei_output/critical_json.txt` |
| High vulns | 47 | `nuclei_output/high_json.txt` |
| Medium vulns | 183 | `nuclei_output/medium_json.txt` |
| Email addresses | 234 | `osint/emails.txt` |
| GitHub secrets | 8 | `osint/github_company_secrets.json` |

**Notable Findings:**
- Exposed Kubernetes dashboard (critical)
- Default credentials on 3 admin panels
- SQL injection in legacy application
- 2 subdomain takeover possibilities
- 8 hardcoded API keys in public GitHub repos

### Deliverable Generation

```bash
# Export all critical/high findings
cat Recon/corp.example.com/nuclei_output/*_json.txt | \
  jq -r 'select(.severity=="critical" or .severity=="high") | 
  [.host, .severity, ."template-id", .name] | @csv' > findings.csv

# Generate AI report
./reconftw.sh -d corp.example.com -y --ai-report full
```

### Key Takeaways

1. **Scope files are essential** - Prevent accidental out-of-scope testing
2. **Rate limiting** - Enterprise WAFs will block aggressive scans
3. **OSINT is gold** - GitHub secrets often provide initial access vectors
4. **Document everything** - Use `--deep` on interesting assets

---

## Case Study 3: Red Team - Distributed Scanning with Axiom

**Scenario:** Red team engagement against a large organization. Need to enumerate quickly across multiple regions without attribution.

### Target Profile
- **Scope:** `bigcorp.com` and all subsidiaries
- **Size:** Very large (10,000+ subdomains estimated)
- **Time:** 48 hours for initial recon
- **Goal:** Complete external footprint, identify entry points

### Infrastructure Setup

```bash
# Launch Axiom fleet (20 instances across regions)
axiom-fleet reconftw -i 20 --regions nyc1,lon1,sgp1,ams3

# Verify fleet
axiom-ls
```

### Configuration

```bash
# Custom config: redteam-distributed.cfg

# Aggressive enumeration (fleet handles load)
SUBBRUTE=true
SUBPERMUTE=true
SUBRECURSIVE=true
SUB_RECURSIVE_BRUTE=true

# Full web checks
WEBPROBESIMPLE=true
WEBPROBEFULL=true

# All vulnerability checks
NUCLEICHECK=true
NUCLEI_SEVERITY="critical,high,medium,low"
FUZZ=true
XSS=true
SQLI=true

# Axiom configuration
AXIOM=true
AXIOM_FLEET_NAME="reconftw"
AXIOM_FLEET_COUNT=20
AXIOM_FLEET_SHUTDOWN=false  # Keep for multiple scans
```

### Execution

```bash
# Create target list
cat << EOF > targets.txt
bigcorp.com
subsidiary1.com
subsidiary2.com
acquired-company.com
EOF

# Launch distributed scan
./reconftw.sh -l targets.txt -a -v -f redteam-distributed.cfg
```

### Monitoring Progress

```bash
# Watch fleet activity
axiom-exec 'ps aux | grep -E "(nuclei|httpx|subfinder)"' -f reconftw

# Check intermediate results
axiom-exec 'wc -l /tmp/subdomains.txt' -f reconftw

# Pull partial results
axiom-scp 'reconftw*:/home/op/Recon' ./partial_results/ -f reconftw
```

### Results

**Scan completed in:** 14 hours (vs estimated 72 hours single-machine)

| Target | Subdomains | Web Servers | Critical | High |
|--------|------------|-------------|----------|------|
| bigcorp.com | 8,234 | 2,847 | 8 | 34 |
| subsidiary1.com | 1,892 | 623 | 3 | 12 |
| subsidiary2.com | 945 | 312 | 1 | 8 |
| acquired-company.com | 2,134 | 892 | 6 | 23 |
| **Total** | **13,205** | **4,674** | **18** | **77** |

**Entry Points Identified:**
1. Exposed Citrix gateway (CVE-2023-XXXX)
2. Password spraying vector (O365 enumeration)
3. VPN with default credentials
4. Subdomain takeover → potential phishing vector
5. API key leaked in JS files → internal API access

### Cost Analysis

```
Fleet: 20 × $0.03/hr × 14 hours = $8.40
Total scan cost: Under $10 for massive reconnaissance
```

### Post-Scan Cleanup

```bash
# Merge all results
axiom-scp 'reconftw*:/home/op/Recon' ./final_results/ -f reconftw

# Shutdown fleet
axiom-rm 'reconftw*' -f
```

### Key Takeaways

1. **Axiom is cost-effective** - Massive scans for dollars, not hours
2. **Regional distribution** - Avoids geo-blocking
3. **Keep fleet alive** - Reuse for multiple targets
4. **Merge results carefully** - Use provided merge scripts

---

## Case Study 4: Continuous Monitoring - CI/CD Integration

**Scenario:** Security team wants automated weekly scans of all company assets with alerts for new findings.

### Setup

**Weekly cron job:**
```bash
# /etc/cron.d/reconftw-weekly
0 2 * * 0 /opt/reconftw/scripts/weekly_scan.sh
```

**Scan script:**
```bash
#!/bin/bash
# weekly_scan.sh

TARGETS="/opt/targets/company_domains.txt"
CONFIG="/opt/reconftw/configs/weekly.cfg"
OUTPUT="/opt/recon_results/$(date +%Y-%m-%d)"

# Run scan
cd /opt/reconftw
./reconftw.sh -l "$TARGETS" -r -f "$CONFIG" -o "$OUTPUT"

# Generate diff from last week
./scripts/diff_results.sh "$OUTPUT" "$LAST_WEEK"

# Send notification
./scripts/notify.sh "$OUTPUT/diff_report.txt"
```

### Configuration

```bash
# weekly.cfg - Balanced for regular monitoring

# Standard enumeration
SUBBRUTE=false          # Passive only for regular scans
SUBPERMUTE=false

# Web checks
WEBPROBESIMPLE=true
SCREENSHOT=true

# Vulnerability focus
NUCLEICHECK=true
NUCLEI_SEVERITY="critical,high"
SUBTAKEOVER=true

# Notifications
SLACK_WEBHOOK="https://hooks.slack.com/services/XXX"
NOTIFY=true
NOTIFY_CONFIG="${tools}/notify-config.yaml"
```

### Diff Script

```bash
#!/bin/bash
# diff_results.sh

NEW=$1
OLD=$2

echo "=== New Subdomains ===" > "$NEW/diff_report.txt"
comm -23 <(sort "$NEW/subdomains/subdomains.txt") \
         <(sort "$OLD/subdomains/subdomains.txt") >> "$NEW/diff_report.txt"

echo -e "\n=== New Vulnerabilities ===" >> "$NEW/diff_report.txt"
diff "$OLD/nuclei_output/critical_json.txt" "$NEW/nuclei_output/critical_json.txt" \
  | grep "^>" >> "$NEW/diff_report.txt"
```

### Alert Output

```
🔔 Weekly Recon Report - 2024-01-15

=== New Subdomains (12) ===
new-api.company.com
staging2.company.com
...

=== New Vulnerabilities (3) ===
[CRITICAL] api.company.com - CVE-2024-XXXX
[HIGH] staging.company.com - exposed-admin
[HIGH] new-api.company.com - default-credentials

Full report: /opt/recon_results/2024-01-15/
```

### Key Takeaways

1. **Passive for monitoring** - Brute-force only for initial scans
2. **Diff is everything** - Focus on changes, not full results
3. **Alert on severity** - Critical/High only for notifications
4. **Archive results** - Keep history for trend analysis

---

## Quick Reference: Scenario → Configuration

| Scenario | Mode | Key Flags | Config Changes |
|----------|------|-----------|----------------|
| Bug bounty rush | `-p` then `-r` | `--adaptive-rate` | Disable slow modules |
| Enterprise assessment | `-a` | `-i`, `-x`, `-f` | Rate limiting, thorough |
| Red team (distributed) | `-a -v` | `--vps` | Axiom fleet config |
| CI/CD monitoring | `-r` | `-o`, `-f` | Passive only, notifications |
| Quick triage | `-p` | None | Default passive |
| Deep dive (single target) | `-a --deep` | None | Enable everything |
