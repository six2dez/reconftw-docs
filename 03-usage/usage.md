# Command Line Usage Guide

This guide covers every command-line option available in reconFTW, with detailed explanations and practical examples.

---

## Basic Syntax

```bash
./reconftw.sh [TARGET_OPTIONS] [MODE_OPTIONS] [ADDITIONAL_OPTIONS]
```

### Quick Examples

```bash
# Single domain reconnaissance
./reconftw.sh -d example.com -r

# Multiple targets from file
./reconftw.sh -l targets.txt -r

# Full scan with vulnerabilities
./reconftw.sh -d example.com -a

# Passive only (stealth)
./reconftw.sh -d example.com -p
```

---

## Target Options

### `-d, --domain <domain>`

Specify a single target domain or IP address.

```bash
# Domain target
./reconftw.sh -d example.com -r

# Subdomain target
./reconftw.sh -d api.example.com -r

# IP address target
./reconftw.sh -d 192.168.1.1 -r

# CIDR range
./reconftw.sh -d 192.168.1.0/24 -r
```

**Input Validation:**
- Domains are sanitized (lowercase, safe characters only)
- IPs/CIDRs are validated for proper format
- Invalid inputs are rejected with error messages

### `-l, --list <file>`

Specify a file containing multiple targets (one per line).

```bash
# Create targets file
cat > targets.txt << EOF
example.com
test.com
demo.org
EOF

# Run scan on all targets
./reconftw.sh -l targets.txt -r
```

**File Format:**
```
# targets.txt
example.com
api.example.com
192.168.1.0/24
test.org
```

**Behavior:**
- Each target is scanned sequentially
- Results are saved in separate directories
- Progress continues if one target fails

### `-m <name>`

Multi-target mode with custom name for the output directory.

```bash
# Scan multiple targets, save under "client-project"
./reconftw.sh -m client-project -l targets.txt -r
```

**Output Structure:**
```
Recon/client-project/
├── targets/
│   ├── example.com/
│   ├── test.com/
│   └── demo.org/
└── .log/
```

---

## Scan Modes

reconFTW offers several scan modes optimized for different use cases.

### `-r, --recon` (Full Reconnaissance)

**The standard bug bounty reconnaissance mode.** Performs full subdomain enumeration, web analysis, and light vulnerability scanning (nuclei on discovered webs).

> ⚠️ **Note:** This is an active scanning mode. It sends requests to the target. Ensure you have authorization.

```bash
./reconftw.sh -d example.com -r
```

**What it does:**
| Phase | Functions |
|-------|-----------|
| OSINT | domain_info, emails, dorks, metadata |
| Subdomains | passive, bruteforce, permutations, scraping |
| Hosts | portscan, CDN detection, WAF detection |
| Web | probing, screenshots, URL extraction, JS analysis |
| Light Vulns | nuclei (info/low), subdomain takeover |

**Duration:** 1-4 hours depending on target size

**Best for:** Initial reconnaissance, bug bounty hunting

### `-s, --subdomains` (Subdomain Enumeration Only)

**Fast subdomain discovery without web analysis.**

```bash
./reconftw.sh -d example.com -s
```

**What it does:**
- Passive subdomain enumeration (APIs, CT logs)
- Active DNS bruteforce
- Permutation generation
- DNS resolution
- Subdomain takeover checks

**Does NOT do:**
- Web probing
- Screenshot capture
- URL extraction
- Vulnerability scanning

**Duration:** 15-60 minutes

**Best for:** Quick attack surface mapping, asset discovery

### `-p, --passive` (Passive Reconnaissance)

**Non-intrusive reconnaissance using only passive techniques.**

```bash
./reconftw.sh -d example.com -p
```

**What it does:**
- Passive subdomain enumeration only
- No DNS bruteforce
- No active web crawling
- No vulnerability scanning
- OSINT gathering

**Techniques disabled:**
```
SUBNOERROR=false
SUBANALYTICS=false
SUBBRUTE=false
SUBSCRAPING=false
SUBPERMUTE=false
PORTSCAN_ACTIVE=false
```

**Duration:** 10-30 minutes

**Best for:** Stealth recon, initial scoping, when you can't touch the target

### `-a, --all` (Full Scan with Vulnerabilities)

**Complete reconnaissance plus full vulnerability scanning.**

> 🔴 **WARNING:** This is the "YOLO mode". It sends attack payloads (SQLi, XSS, etc.) to the target. Only use with explicit written authorization for penetration testing. May trigger WAFs and security alerts.

```bash
./reconftw.sh -d example.com -a
```

**What it does:**
Everything in `-r` mode PLUS:
- Full Nuclei scanning (all severities)
- XSS testing
- SQL injection testing
- SSRF checks
- LFI/SSTI testing
- CORS misconfiguration
- Open redirect testing
- Command injection
- Prototype pollution
- HTTP smuggling
- And more...

**Duration:** 4-24+ hours

**Best for:** Full security assessment, when you have explicit authorization and time

> ⚠️ **Warning:** This mode performs intrusive testing. Ensure you have explicit authorization.

### `-w, --web` (Web Analysis Only)

**Analyze a list of known URLs without subdomain enumeration.**

```bash
# Create a list of URLs
cat > urls.txt << EOF
https://www.example.com
https://api.example.com
https://admin.example.com
EOF

# Run web analysis
./reconftw.sh -l urls.txt -w
```

**What it does:**
- HTTP probing
- Screenshot capture
- URL extraction
- JavaScript analysis
- Directory fuzzing
- CMS detection

**Does NOT do:**
- Subdomain enumeration
- DNS analysis
- Port scanning

**Duration:** 30 minutes - 2 hours

**Best for:** When you already have a list of targets, analyzing specific endpoints

### `-n, --osint` (OSINT Only)

**Gather open-source intelligence without active scanning.**

```bash
./reconftw.sh -d example.com -n
```

**What it does:**
- Domain WHOIS information
- Email harvesting
- Google dorking
- GitHub repository analysis
- Metadata extraction
- API leak detection
- Third-party misconfiguration checks
- SPF/DMARC analysis
- Cloud storage enumeration

**Does NOT do:**
- Active subdomain enumeration
- Web crawling
- Port scanning
- Vulnerability testing

**Duration:** 15-45 minutes

**Best for:** Intelligence gathering, pre-engagement research

### `-c <function>` (Custom Function)

**Execute a specific function from the reconFTW modules.**

```bash
# Run only subdomain bruteforce
./reconftw.sh -d example.com -c sub_brute

# Run only nuclei scanning
./reconftw.sh -d example.com -c nuclei_check

# Run only screenshot capture
./reconftw.sh -d example.com -c screenshot
```

**Note:** `-c` accepts a single function per run. To execute multiple functions, run separate commands (or create a custom mode).

**Available Functions:**

<details>
<summary>Click to expand function list</summary>

**OSINT Functions:**
- `google_dorks`
- `github_dorks`
- `github_repos`
- `metadata`
- `apileaks`
- `emails`
- `domain_info`
- `third_party_misconfigs`
- `spoof`
- `mail_hygiene`
- `cloud_enum_scan`
- `ip_info`

**Subdomain Functions:**
- `sub_passive`
- `sub_crt`
- `sub_brute`
- `sub_permut`
- `sub_ia_permut`
- `sub_regex_permut`
- `sub_recursive_passive`
- `sub_recursive_brute`
- `sub_scraping`
- `sub_analytics`
- `sub_noerror`
- `sub_dns`
- `sub_tls`
- `subtakeover`
- `zonetransfer`
- `s3buckets`

**Web Functions:**
- `webprobe_simple`
- `webprobe_full`
- `screenshot`
- `virtualhosts`
- `urlchecks`
- `url_gf`
- `url_ext`
- `jschecks`
- `fuzz`
- `cms_scanner`
- `wordlist_gen`
- `iishortname`
- `graphql_scan`

**Vulnerability Functions:**
- `nuclei_check`
- `xss`
- `cors`
- `open_redirect`
- `ssrf_checks`
- `crlf_checks`
- `lfi`
- `ssti`
- `sqli`
- `command_injection`
- `prototype_pollution`
- `smuggling`
- `webcache`
- `4xxbypass`
- `fuzzparams`
- `test_ssl`
- `spraying`
- `brokenLinks`

**Host Functions:**
- `portscan`
- `cdnprovider`
- `waf_checks`
- `favicon`
- `geo_info`

</details>

**Requirement:** The target directory must already exist (from a previous scan).

### `-z, --zen` (Zen Mode)

**Minimal terminal output mode for cleaner logs.**

```bash
./reconftw.sh -d example.com -z
```

**Behavior:**
- Reduced terminal output
- Progress indicators only
- Full details in log files
- Same functionality as `-r`

**Best for:** Running in tmux/screen, CI/CD pipelines

---

## Scope Management

### `-x <file>` (Out-of-Scope)

Exclude specific domains/patterns from results.

```bash
# Create out-of-scope file
cat > outscope.txt << EOF
*.cdn.example.com
staging.example.com
*.cloudfront.net
EOF

# Run scan with exclusions
./reconftw.sh -d example.com -r -x outscope.txt
```

**Pattern Syntax:**
```
# Exact match
staging.example.com

# Wildcard (ends with)
*.cdn.example.com

# Multiple patterns
test.example.com
*.dev.example.com
```

### `-i <file>` (In-Scope)

Only include targets matching the scope file.

```bash
# Create in-scope file
cat > inscope.txt << EOF
*.example.com
*.example.org
api.partner.com
EOF

# Run scan with scope filter
./reconftw.sh -d example.com -r -i inscope.txt
```

**How it works:**
1. Results are generated normally
2. inscope filter is applied
3. Only matching entries are kept

**Enable in config:**
```bash
INSCOPE=true  # In reconftw.cfg
```

---

## Advanced Flags

### `--deep` (Deep/Thorough Mode)

Enable extended scanning with larger wordlists and more techniques.

```bash
./reconftw.sh -d example.com -r --deep
```

**Changes from standard mode:**

| Aspect | Standard | Deep |
|--------|----------|------|
| Subdomain wordlist | ~10k entries | ~100k+ entries |
| Permutation depth | 1 level | Multiple levels |
| GitHub dorks | Small list | Medium list |
| Fuzzing wordlist | Common paths | Extended |
| Recursive enumeration | Limited | Full |

### `-v, --vps` (Axiom/Distributed Mode)

Enable distributed scanning using [Axiom](08-integrations/axiom.md).

```bash
./reconftw.sh -d example.com -r -v
```

**Requirements:**
- Axiom must be installed and configured
- Cloud provider account (DigitalOcean, AWS, etc.)
- Fleet configuration in reconftw.cfg

See [Axiom Integration](../08-integrations/axiom.md) for setup details.

### `-f <file>` (Custom Config)

Use a custom configuration file instead of the default.

```bash
# Create custom config
cp reconftw.cfg custom_config.cfg
# Edit custom_config.cfg...

# Use custom config
./reconftw.sh -d example.com -r -f custom_config.cfg
```

**Use cases:**
- Different configs for different clients
- Testing configuration changes
- CI/CD with environment-specific settings

### `-q <rate>` (Rate Limiting)

Set a global rate limit for all tools.

```bash
# Limit to 50 requests/second
./reconftw.sh -d example.com -r -q 50

# Very slow/stealthy
./reconftw.sh -d example.com -r -q 10
```

**Affects:**
- `NUCLEI_RATELIMIT`
- `FFUF_RATELIMIT`
- `HTTPX_RATELIMIT`

### `-o <path>` (Custom Output Directory)

Save results to a custom location.

```bash
# Absolute path
./reconftw.sh -d example.com -r -o /home/user/results

# Relative path
./reconftw.sh -d example.com -r -o ./client-results
```

**Output structure:**
```
/home/user/results/
└── example.com/
    ├── subdomains/
    ├── webs/
    └── ...
```

### `-y, --ai` (AI Report Generation)

Generate AI-powered reports after scan completion.

```bash
./reconftw.sh -d example.com -r -y
```

**Requirements:**
- Local AI model (e.g., llama3:8b via Ollama)
- reconftw_ai tool installed
- Configured in reconftw.cfg

**Report types:**
- `executive` - High-level summary
- `brief` - Concise findings
- `bughunter` - Detailed technical report

### `--quick-rescan`

Skip heavy operations if no new subdomains/assets found.

```bash
./reconftw.sh -d example.com -r --quick-rescan
```

**Behavior:**
1. Performs subdomain enumeration
2. Compares with previous results
3. If no new subdomains → skips heavy modules
4. Saves significant time on repeat scans

### `--incremental`

Only scan new findings since last run.

```bash
./reconftw.sh -d example.com -r --incremental
```

**How it works:**
1. Loads previous scan baseline
2. Performs new enumeration
3. Identifies delta (new findings only)
4. Scans only new assets
5. Generates incremental report

### `--adaptive-rate`

Automatically adjust rate limits when encountering errors.

```bash
./reconftw.sh -d example.com -r --adaptive-rate
```

**Behavior:**
- Starts at configured rate limit
- Detects 429/503 errors
- Reduces rate by 50% on errors
- Increases rate by 20% on success
- Respects MIN/MAX limits

### `--dry-run`

Preview commands without executing them.

```bash
./reconftw.sh -d example.com -r --dry-run
```

**Output:**
```
[DRY-RUN] Would execute: subfinder -d example.com -all -o .tmp/subfinder.txt
[DRY-RUN] Would execute: amass enum -passive -d example.com -o .tmp/amass.txt
...
```

**Best for:** Testing configurations, understanding workflow

### `--parallel`

Run independent functions in parallel for faster scans.

```bash
./reconftw.sh -d example.com -r --parallel
```

**What it does:**
- Runs passive enumeration sources concurrently
- Runs active DNS checks in parallel after passive completes
- Runs TLS and analytics checks after resolution
- Uses native bash job control (no external dependencies)

**Performance impact:**
- ~2-3x faster subdomain enumeration
- Higher memory usage (multiple processes)
- Higher CPU usage

**When NOT to use:**
- Low memory systems (< 4GB RAM)
- Rate-limited targets
- With Axiom (already distributed)
- When debugging issues

### `--monitor`

Enable continuous monitoring mode for repeated reconnaissance cycles on the same target.

```bash
./reconftw.sh -d example.com -r --monitor
```

**Behavior:**
1. Forces incremental/diff behavior each cycle
2. Captures cycle deltas in `.incremental/history/`
3. Writes `delta.json` and `alerts.json` per cycle
4. Rebuilds report artifacts after each cycle

**Scope rules:**
- Supports a single target
- `-l` is only accepted when mode is `-w`
- Ignores `-o` to preserve stable state and history

### `--monitor-interval <minutes>`

Set sleep interval between monitor cycles.

```bash
./reconftw.sh -d example.com -r --monitor --monitor-interval 30
```

### `--monitor-cycles <n>`

Stop monitor mode after `n` cycles (`0` means infinite).

```bash
./reconftw.sh -d example.com -r --monitor --monitor-cycles 48
```

### `--report-only`

Rebuild reports from existing artifacts without running scanning modules.

```bash
./reconftw.sh -d example.com --report-only
```

**Use cases:**
- Re-render HTML/JSON report after post-processing
- Regenerate exports from an old scan folder
- CI jobs that only package reporting artifacts

**Constraints:**
- Requires `-d <domain>`
- Does not support `-l` or `-m`
- Uses existing artifacts under `Recon/<domain>/` (it does not run recon modules)

### `--refresh-cache`

Force refresh cached resources (resolvers and wordlist-related data) for the current run.

```bash
./reconftw.sh -d example.com -r --refresh-cache
```

### `--export <json|html|csv|all>`

Control report/export artifacts emitted at the end of the run (or with `--report-only`).

```bash
./reconftw.sh -d example.com -r --export all
```

**Available values:**
- `json`: JSONL exports (`report/findings_normalized.jsonl`, `report/export_all.jsonl`)
- `html`: consolidated HTML report (`report/index.html`)
- `csv`: CSV artifacts (`report/*.csv`) plus normalized JSONL
- `all`: HTML + JSONL + CSV

### `--check-tools`

Verify all required tools are installed.

```bash
./reconftw.sh --check-tools
```

**Output:**
```
[✓] subfinder
[✓] amass
[✓] httpx
[✗] nuclei (not found)
...
```

### `--health-check`

Run system health diagnostics.

```bash
./reconftw.sh --health-check
```

**Checks:**
- Critical dependencies installed
- Configuration file valid
- Required directories exist
- Network connectivity
- Disk space available

---

## Usage Examples

### Bug Bounty Workflow

```bash
# Initial reconnaissance
./reconftw.sh -d target.com -r

# Follow-up with vulnerabilities
./reconftw.sh -d target.com -a

# Quick rescan after some time
./reconftw.sh -d target.com -r --quick-rescan --incremental
```

### Stealth Assessment

```bash
# Passive only - no direct contact with target
./reconftw.sh -d target.com -p -q 5
```

### Large-Scale Scanning

```bash
# Multiple targets with Axiom
./reconftw.sh -l targets.txt -r -v --deep

# With custom output
./reconftw.sh -m client-assessment -l targets.txt -a -o /data/assessments
```

### CI/CD Integration

```bash
# Non-interactive, minimal output
./reconftw.sh -d target.com -r -z --dry-run

# With health check first
./reconftw.sh --health-check && ./reconftw.sh -d target.com -r -z
```

### Continuous Monitoring

```bash
# Every 30 minutes, stop after 48 cycles
./reconftw.sh -d target.com -r --monitor --monitor-interval 30 --monitor-cycles 48
```

### Reporting and Exports

```bash
# Rebuild report files only
./reconftw.sh -d target.com --report-only --export all

# Normal scan with export bundle
./reconftw.sh -d target.com -r --export all
```

### Scoped Assessment

```bash
# With in-scope and out-of-scope files
./reconftw.sh -d target.com -a -i scope.txt -x exclusions.txt
```

### Custom Function Execution

```bash
# Re-run only specific parts
./reconftw.sh -d target.com -c nuclei_check
./reconftw.sh -d target.com -c xss
./reconftw.sh -d target.com -c sqli
```

---

## Flag Reference Table

| Flag | Long Form | Argument | Description |
|------|-----------|----------|-------------|
| `-d` | `--domain` | domain | Single target domain/IP |
| `-l` | `--list` | file | Target list file |
| `-m` | - | name | Multi-target output name |
| `-r` | `--recon` | - | Full reconnaissance mode |
| `-s` | `--subdomains` | - | Subdomain enumeration only |
| `-p` | `--passive` | - | Passive reconnaissance |
| `-a` | `--all` | - | Full scan + vulnerabilities |
| `-w` | `--web` | - | Web analysis only |
| `-n` | `--osint` | - | OSINT gathering only |
| `-c` | - | function | Custom function execution |
| `-z` | `--zen` | - | Minimal output mode |
| `-x` | - | file | Out-of-scope file |
| `-i` | - | file | In-scope file |
| `-o` | - | path | Custom output directory |
| `-f` | - | file | Custom config file |
| `-q` | - | rate | Rate limit (req/sec) |
| `-v` | `--vps` | - | Axiom distributed mode |
| `-y` | `--ai` | - | AI report generation |
| - | `--deep` | - | Deep/thorough scanning |
| - | `--quick-rescan` | - | Skip heavy ops if no new assets |
| - | `--incremental` | - | Scan only new findings |
| - | `--adaptive-rate` | - | Auto-adjust rate limits |
| - | `--dry-run` | - | Preview without executing |
| - | `--parallel` | - | Run functions in parallel (faster) |
| - | `--no-parallel` | - | Force sequential execution |
| - | `--monitor` | - | Continuous monitoring mode |
| - | `--monitor-interval` | minutes | Minutes between monitor cycles |
| - | `--monitor-cycles` | count | Stop monitor after N cycles (0 infinite) |
| - | `--report-only` | - | Rebuild report artifacts only |
| - | `--refresh-cache` | - | Force cache refresh for this run |
| - | `--export` | format | Export artifacts: json/html/csv/all |
| - | `--check-tools` | - | Verify tool installation |
| - | `--health-check` | - | System diagnostics |
| `-h` | `--help` | - | Show help message |

---

## Next Steps

- **[Configuration Reference](../04-configuration/configuration.md)** - Customize every setting
- **[Module Documentation](../05-modules/)** - Deep dive into each module
- **[Output Interpretation](../07-output/output.md)** - Understand your results

---
