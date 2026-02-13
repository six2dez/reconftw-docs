# Subdomain Enumeration: Technical Deep Dive

This guide explains the technical foundations behind each subdomain enumeration technique used in reconFTW.

---

## Execution Pipeline

reconFTW executes subdomain enumeration in a specific order based on data dependencies:

```
Phase 1: Passive Sources (no target contact)
├── sub_passive   → API queries (subfinder, github-subdomains)
└── sub_crt       → Certificate Transparency logs

Phase 2: DNS Resolution (validates passive results)
├── sub_active    → Resolve collected subdomains with puredns
├── sub_noerror   → DNSSEC NOERROR response analysis
└── sub_dns       → DNS record extraction

Phase 3: Post-Resolution Analysis (requires resolved subdomains)
├── sub_tls       → TLS certificate extraction from live hosts
└── sub_analytics → Google Analytics ID correlation

Phase 4: Bruteforce (resource intensive)
├── sub_brute         → DNS bruteforce with wordlists
├── sub_permut        → Permutation generation (gotator/ripgen)
├── sub_regex_permut  → Regex-based pattern permutations
└── sub_ia_permut     → AI-powered permutation generation

Phase 5: Recursive (multiplies work)
├── sub_recursive_passive → Passive enum on discovered subdomains
├── sub_recursive_brute   → Bruteforce on discovered subdomains
└── sub_scraping          → Web scraping for subdomain extraction
```

---

## Passive Techniques

### sub_passive

Queries multiple APIs and databases that aggregate DNS data globally.

**Tools used:**
- `subfinder` - Queries 50+ passive sources
- `github-subdomains` - Searches GitHub code for subdomains
- `gitlab-subdomains` - Searches GitLab code for subdomains

**Data sources include:**
- SecurityTrails, VirusTotal, AlienVault OTX
- Shodan, Censys, BinaryEdge
- Wayback Machine, Common Crawl
- DNS aggregators (DNSDB, PassiveTotal)

**Why multiple sources:**
Each source has different coverage. A subdomain may appear in VirusTotal but not SecurityTrails. Combining sources maximizes discovery.

### sub_crt

Queries Certificate Transparency (CT) logs via crt.sh API.

**How CT works:**
- Certificate Authorities must log every issued certificate to public logs
- Logs are append-only and publicly queryable
- Certificates contain Subject Alternative Names (SANs) listing all domains

**What it reveals:**
- Internal subdomains (admins often request certs for internal tools)
- Wildcard certificate patterns
- Historical infrastructure (old certificates remain in logs)

**Time fencing (`DNS_TIME_FENCE_DAYS`):**
```bash
# Filter to last 90 days
DNS_TIME_FENCE_DAYS=90
```
CT logs contain years of historical data. Time fencing filters results to recent certificates, reducing dead subdomain noise.

**Implementation:**
```bash
crt -s -json -l "${CTR_LIMIT}" "$domain" \
    | jq -r --arg cutoff "$cutoff_date" \
      '[.[] | select(.not_before >= $cutoff)] | .[].subdomain'
```

---

## Active DNS Resolution

### sub_active

Resolves all collected subdomains to filter non-existent ones.

**Tool used:** `puredns` with massdns backend

**Process:**
1. Combine all passive results into single file
2. Resolve against trusted resolvers
3. Filter wildcards using random probe technique
4. Output only subdomains that resolve

**Wildcard handling:**
```bash
puredns resolve input.txt \
    --wildcard-tests 30 \
    --wildcard-batch 1500000 \
    -r "$resolvers_trusted"
```

**Why wildcard detection matters:**
A wildcard DNS record (`*.example.com`) returns a valid response for ANY subdomain. Without filtering, you get thousands of false positives.

### sub_noerror

Exploits DNSSEC NOERROR responses to enumerate subdomains.

**Technical background:**
- Normal DNS returns NXDOMAIN for non-existent domains
- Some DNSSEC-signed zones return NOERROR instead (black lies technique)
- This allows enumeration via response code analysis

**When useful:**
- Targets with DNSSEC-signed zones
- Can find subdomains not in any passive source

### sub_dns

Extracts additional information from DNS records.

**Records analyzed:**
- A/AAAA → IP addresses
- CNAME → Alias targets (potential takeover)
- MX → Mail servers
- TXT → SPF, DKIM, verification records
- NS → Nameservers

---

## Post-Resolution Techniques

### sub_tls

Connects to discovered hosts and extracts TLS certificate SANs.

**Tool used:** `tlsx`

**Ports checked:**
```bash
TLS_PORTS=$(cat "${SCRIPTPATH}/config/tls_ports.txt" 2>/dev/null | tr -d '\n')
```

**What it finds:**
- Additional domains sharing the same certificate
- Internal names in SAN fields
- Related infrastructure

**Why run after resolution:**
Requires IP addresses from resolved subdomains. Must run after `sub_active`.

### sub_analytics

Finds domains sharing the same Google Analytics or Tag Manager ID.

**Tool used:** `AnalyticsRelationships`

**How it works:**
1. Extract analytics IDs from resolved web pages
2. Query databases for other domains with same ID
3. Related domains likely owned by same organization

**Why run after resolution:**
Requires web-accessible hosts to extract analytics IDs.

---

## Bruteforce Techniques

### sub_brute

DNS bruteforce using wordlists.

**Tool used:** `puredns` with optimized wordlists

**Wordlist strategy:**
- Default: `~100K` common subdomain names
- DEEP mode: `~1M+` extended wordlist

**When useful:**
- New subdomains not yet indexed by passive sources
- Internal naming conventions (dev, staging, prod)
- Non-web services without certificates

### sub_permut

Generates permutations based on discovered subdomains.

**Tool used:** `gotator` or `ripgen`

**Example:**
```
Found: api-v1.example.com
Generates:
  - api-v2.example.com
  - api-v3.example.com  
  - api-dev.example.com
  - api-staging.example.com
  - api-prod.example.com
```

**Configuration:**
```bash
PERMUTATIONS_OPTION=gotator  # Deeper but slower
PERMUTATIONS_OPTION=ripgen   # Faster but less thorough
```

### sub_regex_permut

Uses regex pattern learning to generate permutations.

**Tool used:** `regulator`

**How it works:**
1. Analyze discovered subdomain patterns
2. Learn regex rules from patterns
3. Generate new candidates matching rules

### sub_ia_permut

AI-powered permutation generation.

**Tool used:** `subwiz`

**Approach:**
Uses machine learning models trained on subdomain patterns to generate likely candidates.

---

## Deep Wildcard Detection

Standard wildcard detection only checks the root level (`*.example.com`). Enterprise environments often have nested wildcards.

**Problem example:**
```
*.na45.salesforce.com  → All subdomains under na45 resolve
*.api.prod.example.com → All API subdomains in prod resolve
```

**How `deep_wildcard_filter()` works:**

```
Iteration 1:
  Input: a.b.c.example.com, x.b.c.example.com, y.c.example.com
  Test:  random123.b.c.example.com
  Result: Resolves → b.c.example.com is wildcard
  Filter: Remove all *.b.c.example.com

Iteration 2:
  Input: y.c.example.com
  Test:  random456.c.example.com
  Result: Does not resolve → c.example.com is not wildcard
  Filter: Keep y.c.example.com

Final: y.c.example.com
```

**Implementation details:**
- Maximum 5 iterations
- Uses `dnsx` with trusted resolvers
- Random probe: 12-character alphanumeric string
- Results saved to `subdomains/wildcards_detected.txt`

---

## Sensitive Domain Exclusion

The `_is_sensitive_domain()` function checks domains against patterns in `config/sensitive_domains.txt`.

**Pattern format:**
```
*.gov       → Matches any .gov domain
*.gov.*     → Matches any .gov.XX domain
*.mil       → Matches military domains
*.edu       → Matches educational domains
```

**Matching logic:**
```bash
# Wildcard pattern (*.gov)
if [[ "$domain" == *".$suffix" ]] || [[ "$domain" == "$suffix" ]]; then
    return 0  # Is sensitive
fi
```

**Use case:**
When scanning wildcard scopes, prevents enumeration of government or military subdomains that may be part of the target's acquired assets.

---

## Output Files

| File | Description |
|------|-------------|
| `subdomains/subdomains.txt` | Final deduplicated subdomain list |
| `subdomains/subdomains_new.txt` | New subdomains since last run (incremental) |
| `subdomains/wildcards_detected.txt` | Detected wildcard parent domains |
| `.tmp/passive_subs.txt` | Raw passive enumeration results |
| `.tmp/crtsh_subs.txt` | Certificate transparency results |
| `.tmp/subs_no_resolved.txt` | Subdomains before resolution |

---

## Performance Considerations

| Technique | Speed | Resource Usage | When to Disable |
|-----------|-------|----------------|-----------------|
| sub_passive | Fast | Low (API calls) | Never |
| sub_crt | Fast | Low | Never |
| sub_active | Medium | Medium | Never |
| sub_brute | Slow | High | Large targets |
| sub_permut | Slow | High | Large targets |
| sub_recursive_* | Very slow | Very high | Most scans |
| DEEP_WILDCARD_FILTER | Medium | Medium | Small targets |

**Parallel mode (`--parallel`):**
Runs independent functions concurrently. Phases maintain dependencies:
1. Passive functions run in parallel
2. Wait for passive to complete
3. `sub_active` runs first (dependency source)
4. Dependent active enrichment runs in parallel (`sub_noerror`, `sub_dns`)
5. Post-active functions run after resolution
6. Bruteforce/permutation functions run with limited parallelism (resource intensive)
