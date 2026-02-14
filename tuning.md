# Performance Tuning Guide

Optimize reconFTW for your target size, hardware, and time constraints.

---

## Quick Tuning Profiles

Copy-paste these configurations based on your scenario:

### Small Target (< 100 subdomains)

```bash
# In reconftw.cfg - Small target, thorough scan

# Enable DEEP mode (more thorough)
DEEP=true
DEEP_LIMIT=500
DEEP_LIMIT2=2000

# Moderate threads (don't overwhelm small infra)
HTTPX_THREADS=50
FFUF_THREADS=40
NUCLEI_RATELIMIT=100

# Enable all checks
NUCLEICHECK=true
NUCLEI_DAST=true
FUZZ=true
XSS=true
SQLI=true

# Time estimate: 1-2 hours
```

### Medium Target (100-1,000 subdomains)

```bash
# In reconftw.cfg - Medium target, balanced

# Standard DEEP limits
DEEP_LIMIT=500
DEEP_LIMIT2=1500

# Higher threads
HTTPX_THREADS=100
FFUF_THREADS=60
NUCLEI_RATELIMIT=150

# Core checks enabled
NUCLEICHECK=true
FUZZ=true
XSS=true
SQLI=true

  # Skip some heavy checks
  SMUGGLING=false
  WEBCACHE=false
  # Nuclei DAST runs as part of the vuln pipeline (-a / VULNS_GENERAL=true).
  # To reduce impact, tune NUCLEI_RATELIMIT and/or DEEP_LIMIT2 instead of disabling it.

  # Time estimate: 2-4 hours
  ```

### Large Target (1,000-10,000 subdomains)

```bash
# In reconftw.cfg - Large target, efficient

# Lower DEEP limits (skip intensive checks for large lists)
DEEP_LIMIT=200
DEEP_LIMIT2=500

# High threads
HTTPX_THREADS=150
FFUF_THREADS=80
NUCLEI_RATELIMIT=200

# Essential checks only
NUCLEICHECK=true
FUZZ=false          # Too slow for large targets
XSS=true
SQLI=false          # Use nuclei SQLi templates instead

  # Skip slow modules
  SMUGGLING=false
  WEBCACHE=false
  GRAPHQL_CHECK=false

  # Time estimate: 4-8 hours
  ```

### Massive Target (10,000+ subdomains)

```bash
# In reconftw.cfg - Massive target, speed priority

# Minimal DEEP mode
DEEP_LIMIT=100
DEEP_LIMIT2=200

# Maximum threads
HTTPX_THREADS=200
FFUF_THREADS=100
NUCLEI_RATELIMIT=300

  # Only nuclei scanning
  NUCLEICHECK=true
  FUZZ=false
  XSS=false
  SQLI=false
  SSRF_CHECKS=false
  # Nuclei DAST runs as part of the vuln pipeline (-a / VULNS_GENERAL=true).
  # For speed, keep VULNS_GENERAL=false and run recon-only, or tune NUCLEI_RATELIMIT/DEEP_LIMIT2.

  # Use Axiom for distribution
  AXIOM=true
AXIOM_FLEET_COUNT=20

# Time estimate: 8-24 hours (or 2-4 with Axiom)
```

---

## Auto-Tuning with PERF_PROFILE

reconFTW can auto-tune key concurrency knobs from host resources.

```bash
# In reconftw.cfg
PERF_PROFILE="balanced"   # low | balanced | max
```

Practical guidance:
- `low`: small VPS, stealth scans, conservative concurrency
- `balanced`: default for most environments
- `max`: dedicated runners with enough CPU/RAM

`PERF_PROFILE` tunes job/threads at runtime and is recorded in `.log/perf_summary.json`.

---

## Cache and Refresh Strategy

Use cache TTLs to avoid re-fetching heavy resources on each run:

```bash
CACHE_MAX_AGE_DAYS=30
CACHE_MAX_AGE_DAYS_RESOLVERS=7
CACHE_MAX_AGE_DAYS_WORDLISTS=30
CACHE_MAX_AGE_DAYS_TOOLS=14
```

For one-off freshness runs:

```bash
./reconftw.sh -d target.com -r --refresh-cache
```

---

## Understanding DEEP Mode

DEEP mode runs additional checks when target size is below threshold.

### How It Works

```
Target subdomains count:
├── Below DEEP_LIMIT (500) → Run intensive checks (DEEP mode)
├── Below DEEP_LIMIT2 (1500) → Run moderate checks
└── Above DEEP_LIMIT2 → Skip intensive checks
```

### What DEEP Mode Enables

| Check | Without DEEP | With DEEP |
|-------|--------------|-----------|
| Permutations | Basic | AI + Regex permutations |
| Fuzzing | Top dirs only | Full wordlist |
| JS Analysis | Extract endpoints | Full secret scanning |
| Nuclei | Critical/High only | All severities |
| Parameter discovery | Skip | Full discovery |

### Configuring DEEP Limits

```bash
# For thorough scanning (small targets)
DEEP_LIMIT=1000
DEEP_LIMIT2=3000

# For fast scanning (large targets)
DEEP_LIMIT=100
DEEP_LIMIT2=300

# Disable DEEP mode entirely
DEEP=false
```

---

## Thread Optimization

### By Tool

| Tool | Default | Min (Stealth) | Max (Speed) | Notes |
|------|---------|---------------|-------------|-------|
| `HTTPX_THREADS` | 50 | 10 | 200 | HTTP probing |
| `FFUF_THREADS` | 40 | 10 | 100 | Directory fuzzing |
| `NUCLEI_RATELIMIT` | 150 | 30 | 500 | Vuln scanning (req/sec) |
| `DALFOX_THREADS` | 200 | 50 | 500 | XSS testing |
| `TLSX_THREADS` | 1000 | 200 | 2000 | TLS analysis |
| `RESOLVE_DOMAINS_THREADS` | 150 | 50 | 300 | DNS resolution |

### By Hardware

| System | Threads Multiplier | Example |
|--------|-------------------|---------|
| 1 CPU / 1GB RAM | 0.5x | HTTPX_THREADS=25 |
| 2 CPU / 4GB RAM | 1x (default) | HTTPX_THREADS=50 |
| 4 CPU / 8GB RAM | 2x | HTTPX_THREADS=100 |
| 8+ CPU / 16GB+ RAM | 3-4x | HTTPX_THREADS=200 |

### Memory Considerations

High-memory tools:
- `nuclei` - Keep under 200 ratelimit on low-memory systems
- `ffuf` - Reduce threads on large wordlists
- `katana` - Can be memory-intensive on large sites
- `second-order` - Lower `SECOND_ORDER_THREADS` on constrained hosts

```bash
# Low memory system (2GB)
HTTPX_THREADS=30
FFUF_THREADS=20
NUCLEI_RATELIMIT=50

# High memory system (16GB+)
HTTPX_THREADS=150
FFUF_THREADS=80
NUCLEI_RATELIMIT=300
```

---

## Rate Limiting Strategies

### Adaptive Rate Limiting

```bash
# Enable automatic adjustment
./reconftw.sh -d target.com -r --adaptive-rate

# Or in config
ADAPTIVE_RATE_LIMIT=true
MIN_RATE_LIMIT=10        # Never go below this
MAX_RATE_LIMIT=500       # Never exceed this
RATE_LIMIT_BACKOFF_FACTOR=0.5    # Reduce by 50% on errors
RATE_LIMIT_INCREASE_FACTOR=1.2   # Increase by 20% on success
```

### Manual Rate Limiting

```bash
# Global rate limit
./reconftw.sh -d target.com -r -q 50  # 50 req/sec

# Per-tool limits in config
HTTPX_RATELIMIT=50
NUCLEI_RATELIMIT=50
FFUF_RATELIMIT=50
```

### When to Reduce Rates

| Symptom | Action |
|---------|--------|
| 429 errors | Reduce ratelimit by 50% |
| Connection timeouts | Reduce threads |
| WAF blocks | Use `--adaptive-rate`, reduce to 30 req/sec |
| Server errors (5xx) | Reduce threads AND ratelimit |

---

## Timeout Configuration

```bash
# In reconftw.cfg

# HTTP timeouts
HTTPX_TIMEOUT=10        # Seconds per request
FFUF_TIMEOUT=10
NUCLEI_TIMEOUT=15

# DNS timeouts  
DNS_TIMEOUT=5

# For slow targets
HTTPX_TIMEOUT=30
NUCLEI_TIMEOUT=30
```

---

## Wordlist Optimization

### Subdomain Wordlists

| Setting / Wordlist | Default | Use When |
|-------------------|---------|----------|
| `subs_wordlist` | `${WORDLISTS_DIR}/subdomains.txt` | Standard scans |
| `subs_wordlist_big` | `${tools}/subdomains_n0kovo_big.txt` | DEEP mode / thorough (if installed) |
| Custom small list | (any path) | Fast scans / CI/CD |

```bash
# Fast scan - use a small custom list
subs_wordlist="/path/to/subdomains_small.txt"

# Default (bundled)
subs_wordlist="${WORDLISTS_DIR}/subdomains.txt"

# DEEP mode uses this list (override if you want a different "big" list)
subs_wordlist_big="${tools}/subdomains_n0kovo_big.txt"
```

### Fuzzing Wordlists

| Setting / Wordlist | Default | Use When |
|-------------------|---------|----------|
| `fuzz_wordlist` | `${WORDLISTS_DIR}/fuzz_wordlist.txt` | Standard directory fuzzing |
| Custom small list | (any path) | Faster fuzzing |

```bash
# Fast fuzzing - use a small custom list
fuzz_wordlist="/path/to/fuzz_small.txt"

# Thorough fuzzing - use the bundled list
fuzz_wordlist="${WORDLISTS_DIR}/fuzz_wordlist.txt"
```

### Recommended Wordlists

```bash
# Download optimized wordlists
# SecLists
git clone https://github.com/danielmiessler/SecLists.git ~/wordlists

# Assetnote
wget https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt
```

---

## Module-Specific Tuning

### Nuclei Optimization

```bash
# Severity filtering (faster)
NUCLEI_SEVERITY="critical,high"  # Skip medium/low/info

# Template exclusion
NUCLEI_EXTRA_ARGS="-etags dos,fuzz"  # Skip DoS and heavy fuzz templates

# Timeout
NUCLEI_TIMEOUT=10

# For large scans
NUCLEI_RATELIMIT=200
NUCLEI_FLAGS="-silent -retries 1"  # Reduce retries
```

### Subdomain Enumeration

```bash
# Fast passive only
SUBBRUTE=false          # Skip brute-force
SUBPERMUTE=false        # Skip permutations
SUBIAPERMUTE=false      # Skip AI permutations

# Thorough
SUBBRUTE=true
SUBPERMUTE=true
SUBIAPERMUTE=true
SUBREGEXPERMUTE=true
SUB_RECURSIVE_PASSIVE=true
```

### Fuzzing

```bash
# Fast
FUZZ=false  # Skip entirely, rely on nuclei

# Or minimal fuzzing
FFUF_THREADS=20
fuzz_wordlist="/path/to/fuzz_small.txt"

# Thorough
FUZZ=true
FFUF_THREADS=80
fuzz_wordlist="${WORDLISTS_DIR}/fuzz_wordlist.txt"
```

---

## Axiom Scaling

### Fleet Sizing

| Target Size | Fleet Size | Est. Time |
|-------------|------------|-----------|
| < 100 subs | 3-5 | 30 min |
| 100-1000 | 5-10 | 1-2 hours |
| 1000-10000 | 10-20 | 2-4 hours |
| 10000+ | 20-50 | 4-8 hours |

### Cost Optimization

```bash
# Cheapest instances
AXIOM_INSTANCE_TYPE="s-1vcpu-1gb"  # $0.007/hr on DO

# Cost example:
# 10 instances × 2 hours × $0.007 = $0.14 per scan
```

### Fleet Configuration

```bash
# In reconftw.cfg
AXIOM=true
AXIOM_FLEET_NAME="reconftw"
AXIOM_FLEET_COUNT=10
AXIOM_FLEET_LAUNCH=true
AXIOM_FLEET_SHUTDOWN=true  # Destroy after scan
```

---

## Time Estimates

### By Mode

| Mode | Small (<100) | Medium (100-1K) | Large (1K-10K) |
|------|--------------|-----------------|----------------|
| `-p` (passive) | 5-10 min | 10-20 min | 20-40 min |
| `-s` (subs) | 15-30 min | 30-60 min | 1-2 hours |
| `-r` (recon) | 30-60 min | 1-3 hours | 3-6 hours |
| `-a` (all) | 1-2 hours | 2-5 hours | 5-12 hours |

### Speed vs Thoroughness

```
Fast ←————————————————————————→ Thorough

-p           -s           -r           -a
Passive    Subs only    Full recon   All + vulns
~15 min    ~30 min      ~2 hours     ~4 hours
```

---

## Parallelization

reconFTW includes built-in parallelization to speed up reconnaissance by running independent operations concurrently.

### Enabling Parallelization

```bash
# Run subdomains with parallelization
./reconftw.sh -d target.com -s --parallel

# Or use the parallel functions directly
parallel_subdomains_full
```

### Parallelization Phases

Subdomain enumeration is organized into phases with dependencies:

```
Phase 1: Passive (parallel)
├── sub_passive
└── sub_crt

Phase 2: Active DNS (parallel)
├── sub_active
├── sub_noerror
└── sub_dns

Phase 3: Post-Active (parallel) - requires resolved subs from Phase 2
├── sub_tls
└── sub_analytics

Phase 4: Brute Force (limited parallel) - resource intensive
├── sub_brute
├── sub_permut
├── sub_regex_permut
└── sub_ia_permut

Phase 5: Recursive (sequential) - depends on all previous
├── sub_recursive_passive
├── sub_recursive_brute
└── sub_scraping
```

### Configuration Variables

```bash
# Global toggle and concurrency ceiling
PARALLEL_MODE=true
PARALLEL_MAX_JOBS=4

# Output/replay mode for per-job logs
PARALLEL_LOG_MODE="summary"   # summary | tail | full
PARALLEL_TAIL_LINES=20

# Live progress refresh and presentation
PARALLEL_HEARTBEAT_SECONDS=20
PARALLEL_UI_MODE="clean"      # clean | balanced | trace
PARALLEL_PROGRESS_SHOW_ETA=true   # ETA appears only when estimate is stable
PARALLEL_PROGRESS_SHOW_ACTIVE=true
PARALLEL_PROGRESS_COMPACT_ACTIVE_MAX=4
```

`PARALLEL_UI_MODE` guidelines:
- `clean`: default; single live progress line in TTY with minimal noise.
- `balanced`: adds selective process-level details for slow/problematic tasks.
- `trace`: full process trace (`Started/Finished`, PID/rc) for debugging.

### Performance Impact

| Target Size | Without Parallel | With Parallel | Speedup |
|-------------|------------------|---------------|---------|
| Small (<100) | 15 min | 8 min | ~2x |
| Medium (100-1K) | 45 min | 20 min | ~2.2x |
| Large (1K-10K) | 3 hours | 1.5 hours | ~2x |

### Parallel Functions Reference

| Function | Purpose | Group Size Variable |
|----------|---------|----------|
| Sub passive phase | `sub_passive`, `sub_crt` | `PAR_SUB_PASSIVE_GROUP_SIZE` |
| Sub dependent active | `sub_noerror`, `sub_dns` | `PAR_SUB_DEP_ACTIVE_GROUP_SIZE` |
| Sub post-active | `sub_tls`, `sub_analytics` | `PAR_SUB_POST_ACTIVE_GROUP_SIZE` |
| Sub brute/permut | brute + permutations | `PAR_SUB_BRUTE_GROUP_SIZE` |
| OSINT group 1 | domain/ip/emails/dorks/3rd parties | `PAR_OSINT_GROUP1_SIZE` |
| OSINT group 2 | repos/metadata/apileaks/zone/favicon | `PAR_OSINT_GROUP2_SIZE` |
| Web detect group | screenshot/cdn/portscan | `PAR_WEB_DETECT_GROUP_SIZE` |
| Vulns group 1..4 | vuln stages | `PAR_VULNS_GROUP1_SIZE` ... `PAR_VULNS_GROUP4_SIZE` |

All group sizes are additionally capped at runtime by `PARALLEL_MAX_JOBS` (and reduced under backpressure when rate-limit pressure is high).

### When NOT to Use Parallelization

- **Low-memory systems** (< 4GB RAM): Use sequential mode
- **Rate-limited targets**: Parallel can trigger blocks faster
- **Axiom mode**: Already distributed, parallelization adds complexity
- **Debugging**: Sequential is easier to troubleshoot

---

## Common Tuning Scenarios

### "I need results in 30 minutes"

```bash
# Passive only
./reconftw.sh -d target.com -p
```

### "Overnight scan, make it thorough"

```bash
# Config: Enable everything, DEEP mode
DEEP=true
DEEP_LIMIT=2000

./reconftw.sh -d target.com -a
```

### "Bug bounty, new program rush"

```bash
# Fast but complete
DEEP_LIMIT=300
FUZZ=false
./reconftw.sh -d target.com -r --adaptive-rate
```

### "Red team, need to stay quiet"

```bash
# Low and slow
HTTPX_RATELIMIT=10
NUCLEI_RATELIMIT=10
HTTPX_THREADS=10
./reconftw.sh -d target.com -p  # Start passive
```

---

## Monitoring Performance

### Check Progress

```bash
# Watch log
tail -f "$(ls -1t Recon/target.com/.log/*.txt | head -n1)"

# Check completed functions
ls Recon/target.com/.called_fn/

# Count results
wc -l Recon/target.com/subdomains/subdomains.txt
```

### Resource Monitoring

```bash
# CPU/Memory
htop

# Disk
df -h

# Network
iftop  # or nload
```

---

## TL;DR Quick Config

```bash
# SMALL target - be thorough
DEEP=true && HTTPX_THREADS=50 && NUCLEI_RATELIMIT=100

# LARGE target - be efficient  
DEEP_LIMIT=200 && HTTPX_THREADS=150 && FUZZ=false

# QUIET scan - low profile
HTTPX_RATELIMIT=10 && NUCLEI_RATELIMIT=10

# FAST scan - speed priority
./reconftw.sh -d target.com -p  # Just passive
```
