# Configuration Reference

> **Documentation for reconFTW `dev` branch** | Variables match `reconftw.cfg`

This guide provides a complete reference for reconFTW's configuration file (`reconftw.cfg`), covering every option with detailed explanations.

---

## Configuration Files Overview

reconFTW uses several configuration files:

| File | Purpose | Git Tracked |
|------|---------|-------------|
| `reconftw.cfg` | Main configuration | ✅ Yes |
| `secrets.cfg` | API keys and tokens | ❌ No (gitignored) |
| `custom_config.cfg` | User overrides (optional) | ❌ No |

### Load Order

1. `reconftw.cfg` is loaded first (defaults)
2. `secrets.cfg` is sourced if it exists (API keys)
3. Custom config via `-f` flag overrides all

---

## General Settings

### Tool Paths

```bash
# Path where tools are installed
tools=$HOME/Tools

# Auto-detected script path (don't change)
SCRIPTPATH="$( cd "$(dirname "$0")" >/dev/null 2>&1 ; pwd -P )"
```

### Shell Configuration

```bash
# Detected shell profile (.bashrc, .zshrc, etc.)
profile_shell=".$(basename "${SHELL:-/bin/bash}")rc"
```

### Version Information

```bash
# Auto-detected from git
reconftw_version="$(git rev-parse --abbrev-ref HEAD)-$(git describe --tags)"
```

### Resolver Settings

```bash
# Generate custom resolvers with dnsvalidator
generate_resolvers=false

# Fetch resolvers from trickest before scanning
update_resolvers=true

# Resolver URLs
resolvers_url="https://raw.githubusercontent.com/trickest/resolvers/main/resolvers.txt"
resolvers_trusted_url="https://gist.githubusercontent.com/six2dez/.../trusted_resolvers.txt"
```

**When to change:**
- Set `generate_resolvers=true` for custom resolver validation (slower but more accurate)
- Set `update_resolvers=false` if you maintain your own resolver list

### Proxy Settings

```bash
# HTTP proxy for tools that support it
proxy_url="http://127.0.0.1:8080/"

# Enable proxy usage
PROXY=false
```

**Usage:** Set `PROXY=true` to route web requests through Burp Suite or similar proxy.

### Golang Configuration

```bash
install_golang=true              # Install Go if not found
export GOROOT="/usr/local/go"    # Go installation path
export GOPATH="$HOME/go"         # Go workspace
```

### Update Settings

```bash
upgrade_tools=true               # Allow tool updates
upgrade_before_running=false     # Update tools before each scan
```

### Output Settings

```bash
# Custom output directory (uncomment to enable)
#dir_output=/custom/output/path

# Log executed commands (verbose, may contain sensitive data)
SHOW_COMMANDS=false
```

### Disk Space Check

```bash
# Minimum required disk space in GB (0 to disable)
MIN_DISK_SPACE_GB=0
```

---

## API Keys and Tokens

### Environment Variables (Preferred)

Set these in your shell or `secrets.cfg`:

```bash
# Shodan API for passive port scanning
SHODAN_API_KEY="your_shodan_api_key"

# WhoisXML API for domain lookups
WHOISXML_API="your_whoisxml_api_key"

# Blind XSS callback server
XSS_SERVER="https://your.xss.hunter"

# SSRF/OOB callback server
COLLAB_SERVER="https://your.interact.sh"

# Slack notifications
slack_channel="C0XXXXXXXXX"
slack_auth="xoxb-xxxxx-xxxxx-xxxxx"
```

### secrets.cfg File

Create from the example:

```bash
cp secrets.cfg.example secrets.cfg
chmod 600 secrets.cfg  # Restrict permissions
```

Edit `secrets.cfg`:

```bash
# API Keys
SHODAN_API_KEY="abc123..."
WHOISXML_API="xyz789..."

# Callback servers
XSS_SERVER="https://xss.example.com"
COLLAB_SERVER="https://interact.example.com"

# Notifications
slack_channel="C0XXXXXXXXX"
slack_auth="xoxb-..."
```

### Token Files

```bash
# GitHub tokens (one per line for rate limit distribution)
GITHUB_TOKENS=${tools}/.github_tokens

# GitLab tokens
GITLAB_TOKENS=${tools}/.gitlab_tokens
```

**Create GitHub tokens file:**

```bash
cat > $HOME/Tools/.github_tokens << EOF
ghp_token1xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
ghp_token2xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
ghp_token3xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
EOF
chmod 600 $HOME/Tools/.github_tokens
```

---

## Module Toggles

### OSINT Module

```bash
OSINT=true                   # Master toggle for OSINT module
GOOGLE_DORKS=true            # Google dorking
GITHUB_DORKS=true            # GitHub secret searching
GITHUB_REPOS=true            # GitHub repository analysis
METADATA=true                # Document metadata extraction
EMAILS=true                  # Email harvesting
DOMAIN_INFO=true             # WHOIS lookups
IP_INFO=true                 # IP reverse lookup and geolocation
API_LEAKS=true               # Postman/Swagger leak detection
THIRD_PARTIES=true           # Third-party misconfiguration checks
SPOOF=true                   # Domain spoofing checks
MAIL_HYGIENE=true            # SPF/DMARC analysis
CLOUD_ENUM=true              # Cloud storage enumeration
METAFINDER_LIMIT=20          # Max documents to analyze (max 250)
```

### Subdomain Module

```bash
SUBDOMAINS_GENERAL=true      # Master toggle for subdomain module
SUBPASSIVE=true              # Passive enumeration (APIs, CT logs)
SUBCRT=true                  # Certificate transparency search
CTR_LIMIT=999999             # Max CT results
SUBNOERROR=false             # DNS NOERROR response checking
SUBANALYTICS=true            # Google Analytics correlation
SUBBRUTE=true                # DNS bruteforcing
SUBSCRAPING=true             # Web scraping for subdomains
SUBPERMUTE=true              # DNS permutations
SUBIAPERMUTE=true            # AI-powered permutations
SUBREGEXPERMUTE=true         # Regex-based permutations
SUBTAKEOVER=true             # Subdomain takeover checks
SUB_RECURSIVE_PASSIVE=false  # Recursive passive (uses many API queries)
DEEP_RECURSIVE_PASSIVE=10    # Top N subdomains for recursion
SUB_RECURSIVE_BRUTE=false    # Recursive bruteforce (disk/time intensive)
ZONETRANSFER=true            # DNS zone transfer checks
S3BUCKETS=true               # S3 bucket misconfiguration checks
REVERSE_IP=false             # Reverse IP lookups (enable for IP/CIDR targets)
INSCOPE=false                # Apply inscope filtering
```

### Permutation Settings

```bash
# Permutation tool: "gotator" (deeper) or "ripgen" (faster)
PERMUTATIONS_OPTION=gotator

# Gotator flags
GOTATOR_FLAGS=" -depth 1 -numbers 3 -mindup -adv -md"
```

### TLS Port Discovery

```bash
# Ports to check for TLS certificates
TLS_PORTS="21,22,25,80,110,135,143,261,443,465,563,587,636,853,990,993,995,..."
```

### Web Detection Module

```bash
WEBPROBESIMPLE=true          # Probe ports 80/443
WEBPROBEFULL=true            # Probe uncommon web ports
WEBSCREENSHOT=true           # Capture screenshots
VIRTUALHOSTS=false           # Virtual host fuzzing (slower)

# Uncommon web ports to probe
UNCOMMON_PORTS_WEB="81,300,591,593,832,981,1010,1311,..."
```

### Host Module

```bash
FAVICON=true                 # Favicon-based IP discovery
PORTSCANNER=true             # Port scanning module
GEO_INFO=true                # IP geolocation
PORTSCAN_PASSIVE=true        # Shodan passive port scan
PORTSCAN_ACTIVE=true         # Nmap active port scan
CDN_IP=true                  # CDN detection

# Nmap options
PORTSCAN_ACTIVE_OPTIONS="--top-ports 200 -sV -n -Pn --open --max-retries 2 --script vulners"
```

### Web Analysis Module

```bash
WAF_DETECTION=true           # WAF detection
NUCLEICHECK=true             # Nuclei vulnerability scanning
URL_CHECK=true               # URL collection
URL_CHECK_PASSIVE=true       # Passive URL collection (archives)
URL_CHECK_ACTIVE=true        # Active URL crawling
URL_GF=true                  # URL pattern matching
URL_EXT=true                 # File extension sorting
JSCHECKS=true                # JavaScript analysis
FUZZ=true                    # Directory fuzzing
IIS_SHORTNAME=true           # IIS shortname scanning
CMS_SCANNER=true             # CMS detection
WORDLIST=true                # Custom wordlist generation
ROBOTSWORDLIST=true          # Robots.txt historical analysis
PASSWORD_DICT=true           # Password dictionary generation
PASSWORD_MIN_LENGTH=5        # Min password length
PASSWORD_MAX_LENGTH=14       # Max password length
GRAPHQL_CHECK=true           # GraphQL endpoint detection
GQLSPECTION=false            # Deep GraphQL introspection
PARAM_DISCOVERY=true         # Parameter discovery with Arjun
GRPC_SCAN=false              # gRPC reflection probing
```

### Vulnerability Module

```bash
VULNS_GENERAL=false          # Master toggle for vuln scanning
XSS=true                     # XSS testing
CORS=true                    # CORS misconfiguration
TEST_SSL=true                # SSL/TLS analysis
OPEN_REDIRECT=true           # Open redirect detection
SSRF_CHECKS=true             # SSRF testing
CRLF_CHECKS=true             # CRLF injection
LFI=true                     # Local file inclusion
SSTI=true                    # Server-side template injection
SQLI=true                    # SQL injection
SQLMAP=true                  # SQLMap testing
GHAURI=false                 # Ghauri SQLi testing
BROKENLINKS=true             # Broken link detection
SPRAY=true                   # Password spraying
COMM_INJ=true                # Command injection
PROTO_POLLUTION=true         # Prototype pollution
SMUGGLING=true               # HTTP request smuggling
WEBCACHE=true                # Web cache issues
BYPASSER4XX=true             # 4XX bypass attempts
FUZZPARAMS=true              # Parameter fuzzing
```

### Nuclei Configuration

```bash
# Nuclei templates path
NUCLEI_TEMPLATES_PATH="$HOME/nuclei-templates"

# Severity levels to run
NUCLEI_SEVERITY="info,low,medium,high,critical"

# Extra arguments (exclusions, etc.)
NUCLEI_EXTRA_ARGS=""
# Example with exclusions:
# NUCLEI_EXTRA_ARGS="-etags openssh,ssl -eid node-express-dev-env"

# Standard flags
NUCLEI_FLAGS="-silent -retries 2"

# JS secret scanning flags
NUCLEI_FLAGS_JS="-silent -tags exposure,token -severity info,low,medium,high,critical"
```

---

## Threading and Rate Limits

### Thread Configuration

```bash
FFUF_THREADS=40                      # Directory fuzzing
HTTPX_THREADS=50                     # HTTP probing
HTTPX_UNCOMMONPORTS_THREADS=100      # Uncommon port probing
KATANA_THREADS=20                    # Web crawling
BRUTESPRAY_THREADS=20                # Password spraying
BRUTESPRAY_CONCURRENCE=10            # Concurrent targets
DNSTAKE_THREADS=100                  # DNS takeover checks
DALFOX_THREADS=200                   # XSS testing
TLSX_THREADS=1000                    # TLS certificate scanning
INTERLACE_THREADS=10                 # Parallel tool execution
RESOLVE_DOMAINS_THREADS=150          # DNS resolution
DNSVALIDATOR_THREADS=200             # Resolver validation
XNLINKFINDER_DEPTH=3                 # Link finder depth
ARJUN_THREADS=10                     # Parameter discovery
```

### Rate Limits

```bash
HTTPX_RATELIMIT=150                  # HTTP requests/second
NUCLEI_RATELIMIT=150                 # Nuclei requests/second
FFUF_RATELIMIT=0                     # Fuzzing requests/second (0=unlimited)
```

### PureDNS Limits

```bash
PUREDNS_PUBLIC_LIMIT=0               # Public resolver limit (0=unlimited)
PUREDNS_TRUSTED_LIMIT=400            # Trusted resolver limit
PUREDNS_WILDCARDTEST_LIMIT=30        # Wildcard detection limit
PUREDNS_WILDCARDBATCH_LIMIT=1500000  # Wildcard batch size
```

### Adaptive Rate Limiting

```bash
ADAPTIVE_RATE_LIMIT=false            # Auto-adjust on errors
MIN_RATE_LIMIT=10                    # Minimum rate limit
MAX_RATE_LIMIT=500                   # Maximum rate limit
RATE_LIMIT_BACKOFF_FACTOR=0.5        # Reduce by 50% on error
RATE_LIMIT_INCREASE_FACTOR=1.2       # Increase by 20% on success
```

---

## Timeouts

```bash
SUBFINDER_ENUM_TIMEOUT=180           # Subfinder timeout (minutes)
CMSSCAN_TIMEOUT=3600                 # CMS scan timeout (seconds)
FFUF_MAXTIME=900                     # Fuzzing timeout (seconds)
HTTPX_TIMEOUT=10                     # HTTP request timeout (seconds)
HTTPX_UNCOMMONPORTS_TIMEOUT=10       # Uncommon port timeout (seconds)
PERMUTATIONS_LIMIT=21474836480       # Max permutation file size (bytes, 20GB)
```

---

## Wordlists

```bash
# Fuzzing wordlist
fuzz_wordlist=${tools}/fuzz_wordlist.txt

# LFI payloads
lfi_wordlist=${tools}/lfi_wordlist.txt

# SSTI payloads
ssti_wordlist=${tools}/ssti_wordlist.txt

# Subdomain wordlists
subs_wordlist=${tools}/subdomains.txt
subs_wordlist_big=${tools}/subdomains_n0kovo_big.txt

# Resolver lists
resolvers=${tools}/resolvers.txt
resolvers_trusted=${tools}/resolvers_trusted.txt
```

### Cloud Hunter Settings

```bash
# Cloud permutation depth: DEEP, NORMAL, or NONE
CLOUDHUNTER_PERMUTATION=NORMAL
```

---

## DEEP Mode Settings

```bash
DEEP=false                           # Deep scanning mode
DEEP_LIMIT=500                       # First auto-deep threshold
DEEP_LIMIT2=1500                     # Second auto-deep threshold
```

**Behavior:**
- If subdomain count < DEEP_LIMIT, additional techniques run
- If < DEEP_LIMIT2, even more intensive techniques run

---

## Axiom Settings

```bash
# Axiom fleet configuration
AXIOM_FLEET_LAUNCH=true              # Auto-launch fleet
AXIOM_FLEET_NAME="reconFTW"          # Fleet name prefix
AXIOM_FLEET_COUNT=10                 # Number of instances
AXIOM_FLEET_REGIONS="eu-central"     # Cloud regions
AXIOM_FLEET_SHUTDOWN=true            # Auto-shutdown after scan

# Resolver paths on Axiom instances
AXIOM_RESOLVERS_PATH="/home/op/lists/resolvers.txt"
AXIOM_RESOLVERS_TRUSTED_PATH="/home/op/lists/resolvers_trusted.txt"

# Post-start script (optional)
#AXIOM_POST_START="~/Tools/axiom_config.sh"

# Extra arguments
AXIOM_EXTRA_ARGS=""
```

---

## Faraday Settings

```bash
FARADAY=false                        # Enable Faraday integration
FARADAY_SERVER="http://localhost:5985"
FARADAY_USER="faraday"
FARADAY_PASS="FARADAY_PASSWORD"
FARADAY_WORKSPACE="reconftw"
```

---

## AI Settings

```bash
AI_MODEL="llama3:8b"                 # AI model to use
AI_REPORT_TYPE="md"                  # Report format (md, txt)
AI_REPORT_PROFILE="bughunter"        # Profile: executive, brief, bughunter
```

---

## Extra Features

### Notification Settings

```bash
NOTIFICATION=false                   # Notifications for every function
SOFT_NOTIFICATION=false              # Only start/end notifications
SENDZIPNOTIFY=false                  # Send zipped results via notify
```

### Diff/Incremental Mode

```bash
DIFF=false                           # Differential scanning
INCREMENTAL_MODE=false               # Incremental scanning
```

### Cleanup Settings

```bash
REMOVETMP=false                      # Delete .tmp after scan
REMOVELOG=false                      # Delete logs after scan
PRESERVE=true                        # Keep .called_fn markers
```

### Cache Settings

```bash
CACHE_MAX_AGE_DAYS=30                # Cache validity (days)
```

### Log Rotation

```bash
MAX_LOG_FILES=10                     # Max log files per target
MAX_LOG_AGE_DAYS=30                  # Delete logs older than this
```

### Structured Logging

```bash
STRUCTURED_LOGGING=false             # JSON format logging
```

### Asset Tracking

```bash
ASSET_STORE=true                     # Append to assets.jsonl
QUICK_RESCAN=false                   # Skip heavy steps if no new assets
CHUNK_LIMIT=2000                     # Split large lists
HOTLIST_TOP=50                       # Top risky assets to highlight
```

### IPv6

```bash
IPV6_SCAN=true                       # Enable IPv6 discovery
```

### Intrusive Mode

```bash
INTRUSIVE=false                      # Dangerous cloud/CORS tests
```

---

## HTTP Options

```bash
# Default User-Agent header
HEADER="User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:72.0) Gecko/20100101 Firefox/72.0"
```

### Tool Flags

```bash
# FFUF default flags
FFUF_FLAGS=" -mc all -fc 404 -sf -noninteractive -of json"

# HTTPX probing flags
HTTPX_FLAGS=" -follow-redirects -random-agent -status-code -silent -title -web-server -tech-detect -location -content-length"
```

---

## Color Configuration

```bash
# Terminal colors (ANSI codes)
bred='\033[1;31m'      # Bold red
bblue='\033[1;34m'     # Bold blue
bgreen='\033[1;32m'    # Bold green
byellow='\033[1;33m'   # Bold yellow
red='\033[0;31m'       # Red
blue='\033[0;34m'      # Blue
green='\033[0;32m'     # Green
yellow='\033[0;33m'    # Yellow
reset='\033[0m'        # Reset
```

---

## Debug Settings

```bash
DEBUG_STD="&>/dev/null"              # Skip stdout in installer
DEBUG_ERROR="2>/dev/null"            # Skip stderr in installer
```

---

## Configuration Examples

### Stealth Configuration

```bash
# Minimal noise configuration
SUBBRUTE=false
SUBPERMUTE=false
PORTSCAN_ACTIVE=false
FUZZ=false
VULNS_GENERAL=false
HTTPX_RATELIMIT=10
NUCLEI_RATELIMIT=10
```

### Aggressive Configuration

```bash
# Maximum coverage
DEEP=true
SUBBRUTE=true
SUBPERMUTE=true
SUB_RECURSIVE_BRUTE=true
VULNS_GENERAL=true
FUZZ=true
HTTPX_THREADS=100
NUCLEI_RATELIMIT=500
```

### Bug Bounty Configuration

```bash
# Balanced for bug bounty
OSINT=true
SUBDOMAINS_GENERAL=true
VULNS_GENERAL=false  # Enable with -a flag
NOTIFICATION=true
DIFF=true
```

---

## Using Custom Config Files

```bash
# Create custom config
cp reconftw.cfg client_config.cfg

# Edit for specific client
vim client_config.cfg

# Use custom config
./reconftw.sh -d target.com -r -f client_config.cfg
```

---

## Environment Variable Priority

Environment variables override config file settings:

```bash
# Override via environment
export SHODAN_API_KEY="my_key"
export NUCLEI_RATELIMIT=50

# Run scan (uses environment values)
./reconftw.sh -d target.com -r
```

---

## Next Steps

- **[Module Documentation](../05-modules/)** - Understand each module in detail
- **[Tools Reference](../06-tools/tools.md)** - Learn about integrated tools
- **[Advanced Usage](../10-advanced/advanced.md)** - Custom functions and optimization

---

> **Documentation Info**  
> Branch: `dev` | Version: `v3.0.0+` | Last updated: February 2026  
> Variables documented match `reconftw.cfg` in the repository root.
