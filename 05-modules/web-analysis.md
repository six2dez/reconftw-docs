# Web Analysis Module

The web analysis module examines discovered web assets to identify technologies, extract content, and prepare targets for vulnerability testing.

---

## Why Web Analysis?

Before scanning for vulnerabilities, you need to understand what you're scanning:

1. **Target Validation**: Not all subdomains run web servers. HTTP probing identifies which hosts actually serve web content on which ports.

2. **Technology Fingerprinting**: Different technologies have different vulnerabilities. Knowing that a target runs WordPress vs Django changes your attack approach.

3. **Attack Surface Mapping**: URL collection reveals:
   - Hidden endpoints not linked in the UI
   - API routes
   - Admin panels
   - Legacy code paths

4. **Input Discovery**: Parameters and endpoints found here become test targets for vulnerability scanning. Without URL collection, scanners miss most of the attack surface.

5. **Efficiency**: Analyzing JavaScript and crawling once, then reusing results for multiple vulnerability tests, is more efficient than having each scanner crawl independently.

---

## Module Overview

| Function | Purpose | Tools |
|----------|---------|-------|
| `webprobe_simple` | Probe ports 80/443 | httpx |
| `webprobe_full` | Probe uncommon web ports | httpx |
| `favirecon_tech` | Favicon-based technology recon | favirecon |
| `screenshot` | Capture web screenshots | nuclei |
| `virtualhosts` | Virtual host discovery | VhostFinder |
| `urlchecks` | URL collection (passive + active) | urlfinder, waymore, katana, JSA |
| `url_gf` | URL pattern classification | gf, urless |
| `url_ext` | File extension sorting | custom |
| `jschecks` | JavaScript analysis | subjs, xnLinkFinder, mantra, jsluice |
| `fuzz` | Directory fuzzing | ffuf |
| `cms_scanner` | CMS detection | CMSeeK |
| `wordlist_gen` | Custom wordlist generation | custom |
| `wordlist_gen_roboxtractor` | Robots.txt wordlist | roboxtractor |
| `password_dict` | Password dictionary generation | pydictor |
| `iishortname` | IIS shortname scanning | shortscan, sns |
| `graphql_scan` | GraphQL endpoint detection | nuclei, GQLSpection |
| `grpc_reflection` | gRPC reflection probing | grpcurl |
| `param_discovery` | Parameter discovery | arjun |
| `websocket_checks` | WebSocket auditing | custom |

---

## Configuration Options

```bash
# In reconftw.cfg

# Web probing
WEBPROBESIMPLE=true            # Probe 80/443
WEBPROBEFULL=true              # Probe uncommon ports
FAVIRECON=true                 # Favicon-based technology recon
FAVIRECON_CONCURRENCY=50
FAVIRECON_TIMEOUT=10
FAVIRECON_RATE_LIMIT=0
FAVIRECON_PROXY=""
WEBSCREENSHOT=true             # Screenshots
VIRTUALHOSTS=false             # Virtual host fuzzing

# URL analysis
URL_CHECK=true                 # URL collection
URL_CHECK_PASSIVE=true         # Passive URL sources
URL_CHECK_ACTIVE=true          # Active crawling
WAYMORE_TIMEOUT=30m            # Timeout for waymore passive collection
WAYMORE_LIMIT=5000             # Optional waymore URL collection limit
URL_GF=true                    # Pattern matching
URL_EXT=true                   # Extension sorting

# JavaScript
JSCHECKS=true                  # JS analysis
XNLINKFINDER_DEPTH=3           # Link finder depth

# Fuzzing
FUZZ=true                      # Directory fuzzing
FFUF_THREADS=40
FFUF_RATELIMIT=0
FFUF_MAXTIME=900
FFUF_FLAGS=" -mc all -fc 404 -sf -noninteractive -of json"

# CMS
CMS_SCANNER=true
CMSSCAN_TIMEOUT=3600

# Other
WORDLIST=true
ROBOTSWORDLIST=true            # Robots.txt wordlist generation
PASSWORD_DICT=true             # Password dictionary generation
PASSWORD_MIN_LENGTH=5          # Min password length
PASSWORD_MAX_LENGTH=14         # Max password length
IIS_SHORTNAME=true
GRAPHQL_CHECK=true
GQLSPECTION=false
PARAM_DISCOVERY=true
GRPC_SCAN=false                # gRPC reflection probing
```

---

## HTTP Probing

### `webprobe_simple` - Standard Port Probing

Identifies live web servers on standard ports (80, 443).

**How It Works:**

```
Subdomains → httpx (80, 443) → Filter responses →
→ Extract metadata → webs.txt
```

**Information Extracted:**
- Status codes
- Page titles
- Web server type
- Technologies detected
- Content length
- Redirect locations

**Output:**
```
webs/webs.txt              # Live web servers
webs/web_full_info_plain.txt         # Detailed probe results
```

**Sample Output (web_full_info_plain.txt):**
```
https://www.example.com [200] [Example Site] [nginx] [PHP,WordPress]
https://api.example.com [401] [API Gateway] [cloudflare]
https://admin.example.com [403] [Forbidden] [Apache]
```

**Configuration:**
```bash
WEBPROBESIMPLE=true
HTTPX_THREADS=50
HTTPX_RATELIMIT=150
HTTPX_TIMEOUT=10
HTTPX_FLAGS=" -follow-redirects -random-agent -status-code -silent -title -web-server -tech-detect -location -content-length"
```

---

### `webprobe_full` - Uncommon Port Probing

Probes an extended list of ports commonly used for web services.

**Ports Checked:**
```
81,300,591,593,832,981,1010,1311,1099,2082,2095,2096,2480,3000,
3001,3002,3003,3128,3333,4243,4567,4711,4712,4993,5000,5104,5108,
5280,5281,5601,5800,6543,7000,7001,7396,7474,8000,8001,8008,8014,
8042,8060,8069,8080,8081,8083,8088,8090,8091,8095,8118,8123,8172,
8181,8222,8243,8280,8281,8333,8337,8443,8500,8834,8880,8888,8983,
9000,9001,9043,9060,9080,9090,9091,9092,9200,9443,9502,9800,9981,
10000,10250,11371,12443,15672,16080,17778,18091,18092,20720,32000,
55440,55672
```

**Output:**
```
webs/webs_uncommon_ports.txt
```

**Configuration:**
```bash
WEBPROBEFULL=true
HTTPX_UNCOMMONPORTS_THREADS=100
HTTPX_UNCOMMONPORTS_TIMEOUT=10
UNCOMMON_PORTS_WEB=$(cat "${SCRIPTPATH}/config/uncommon_ports_web.txt" 2>/dev/null | tr -d '\n')
```

---

### `favirecon_tech` - Favicon Technology Recon

Runs `favirecon` against discovered web targets to fingerprint technologies from favicon hashes.

**How It Works:**

```
webs/webs_all.txt → favirecon (JSON + summary) →
→ Normalize findings → webs/favirecon.txt
```

**Output:**
```
webs/favirecon.json
webs/favirecon.txt
```

**Configuration:**
```bash
FAVIRECON=true
FAVIRECON_CONCURRENCY=50
FAVIRECON_TIMEOUT=10
FAVIRECON_RATE_LIMIT=0
FAVIRECON_PROXY=""
```

---

## Screenshots

### `screenshot` - Web Screenshot Capture

Captures screenshots of all discovered web servers for visual analysis.

**How It Works:**

```
webs.txt → nuclei (headless browser) → PNG screenshots
```

**Output:**
```
screenshots/
├── screenshot_www.example.com_443.png
├── screenshot_api.example.com_443.png
├── screenshot_admin.example.com_8080.png
└── hashes.txt  # For change detection
```

**Change Detection:**

reconFTW creates SHA256 hashes of screenshots to detect visual changes between scans:

```
screenshots/hashes.txt       # Current scan hashes
screenshots/hashes_prev.txt  # Previous scan hashes
screenshots/diff_changed.txt # Changed screenshots
```

**Configuration:**
```bash
WEBSCREENSHOT=true
```

<!-- IMAGE PLACEHOLDER: screenshot-gallery.png
     Description: A grid of web screenshot thumbnails showing various 
     websites discovered during reconnaissance - login pages, admin panels,
     APIs, etc. Shows the visual diversity of discovered assets.
     Size: 800x500px
-->

---

## Virtual Hosts

### `virtualhosts` - Virtual Host Discovery

Discovers virtual hosts by fuzzing the HTTP Host header.

**How It Works:**

```
IP addresses → Fuzz Host header with subdomain wordlist →
→ Compare responses → Identify unique virtual hosts
```

**Why It's Useful:**
Many servers host multiple websites on the same IP. Virtual host fuzzing reveals:
- Hidden admin panels
- Development sites
- Internal applications
- Additional attack surface

**Output:**
```
webs/virtualhosts.txt
```

**Configuration:**
```bash
VIRTUALHOSTS=false  # Disabled by default (can be slow)
```

---

## URL Collection

### `urlchecks` - Full URL Extraction

Collects URLs from multiple sources for full coverage.

**Passive Sources:**
- Wayback Machine
- Common Crawl
- AlienVault OTX
- URLScan.io
- Additional passive sources via waymore

**Active Sources:**
- Katana web crawler
- JavaScript parsing
- Sitemap analysis

**How It Works:**

```
                   ┌─────────────────┐
                   │    webs.txt     │
                   └────────┬────────┘
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
    ┌──────────┐     ┌──────────┐     ┌──────────┐
    │ urlfinder│     │ waymore  │     │  katana  │
    │ (passive)│     │ (passive)│     │ (active) │
    └────┬─────┘     └────┬─────┘     └────┬─────┘
         │                │                │
         └────────────────┼────────────────┘
                          ▼
                ┌──────────────────┐
                │ Combine & Dedup  │
                └────────┬─────────┘
                         ▼
                ┌──────────────────┐
                │  url_extract.txt │
                └──────────────────┘
```

**Output:**
```
webs/url_extract.txt        # All discovered URLs
webs/url_extract_comb.txt   # Combined and cleaned
```

**Configuration:**
```bash
URL_CHECK=true
URL_CHECK_PASSIVE=true
URL_CHECK_ACTIVE=true
WAYMORE_TIMEOUT=30m
WAYMORE_LIMIT=5000
KATANA_THREADS=20
```

---

### `url_gf` - URL Pattern Classification

Classifies URLs by potential vulnerability patterns using gf patterns.

**Patterns Detected:**
| Pattern | Description |
|---------|-------------|
| `xss` | Potential XSS parameters |
| `sqli` | SQL injection candidates |
| `ssrf` | SSRF-prone URLs |
| `redirect` | Open redirect parameters |
| `rce` | Command injection candidates |
| `lfi` | Local file inclusion |
| `ssti` | Template injection |
| `idor` | Insecure direct object reference |
| `debug_logic` | Debug/admin endpoints |

**Output:**
```
gf/
├── xss.txt
├── sqli.txt
├── ssrf.txt
├── redirect.txt
├── rce.txt
├── lfi.txt
├── ssti.txt
├── idor.txt
└── debug_logic.txt
```

**Sample XSS Pattern Match:**
```
https://example.com/search?q=test
https://example.com/page?msg=hello
https://example.com/api?callback=func
```

**Configuration:**
```bash
URL_GF=true
```

---

### `url_ext` - File Extension Sorting

Organizes URLs by file extension for targeted analysis.

**Categories:**
```
webs/urls_by_extension/
├── js.txt          # JavaScript files
├── json.txt        # JSON endpoints
├── php.txt         # PHP files
├── asp.txt         # ASP/ASPX files
├── jsp.txt         # JSP files
├── xml.txt         # XML files
├── pdf.txt         # PDF documents
├── config.txt      # Configuration files
└── backup.txt      # Backup files
```

**Configuration:**
```bash
URL_EXT=true
```

---

## JavaScript Analysis

### `jschecks` - Full JS Analysis

Extracts secrets, endpoints, and sensitive information from JavaScript files.

**What It Finds:**
- API keys and tokens
- AWS credentials
- Internal endpoints
- Hardcoded passwords
- Debug information
- Hidden functionality

**Tools Used:**
- **subjs**: JS file discovery
- **xnLinkFinder**: Endpoint extraction
- **mantra**: Secret patterns
- **jsluice**: Advanced JS parsing
- **nuclei**: JS secret templates
- **sourcemapper**: Source map extraction

**How It Works:**

```
webs.txt → subjs (find JS files) → Download JS →
→ Multiple analyzers → Extract secrets/endpoints → Output
```

**Output:**
```
js/
├── js_livelinks.txt         # Extracted endpoints
├── js_secrets.txt           # Discovered secrets
├── js_secrets_jsmap.txt     # Sourcemap secrets
├── js_secrets_jsluice.txt   # jsluice findings
└── js_getjswords.txt        # Wordlist from JS
```

**Sample Secrets Found:**
```
[AWS_ACCESS_KEY] AKIA... in https://example.com/app.js
[API_KEY] sk_live_... in https://example.com/config.js
[PRIVATE_KEY] -----BEGIN RSA PRIVATE KEY----- in /bundle.js
```

**Configuration:**
```bash
JSCHECKS=true
XNLINKFINDER_DEPTH=3
NUCLEI_FLAGS_JS="-silent -tags exposure,token -severity info,low,medium,high,critical"
```

---

## Directory Fuzzing

### `fuzz` - Web Directory Fuzzing

Discovers hidden directories, files, and endpoints.

**How It Works:**

```
webs.txt → ffuf (with wordlist) → Filter responses →
→ Identify interesting paths → Output
```

**Wordlists:**
- Primary: `$fuzz_wordlist`
- Custom generated from target

**Output:**
```
fuzzing/
├── fuzzing_full.json    # Complete results
└── fuzzing_interesting.txt  # Filtered findings
```

**Sample Findings:**
```
/admin [200] [Admin Panel]
/backup [403] [Forbidden]
/.git/config [200] [Git Config]
/api/swagger [200] [Swagger UI]
/phpinfo.php [200] [PHP Info]
```

**Configuration:**
```bash
FUZZ=true
FFUF_THREADS=40
FFUF_RATELIMIT=0
FFUF_MAXTIME=900
FFUF_FLAGS=" -mc all -fc 404 -sf -noninteractive -of json"
fuzz_wordlist=${tools}/fuzz_wordlist.txt
```

---

## CMS Detection

### `cms_scanner` - CMS Identification

Identifies content management systems and their versions.

**CMS Detected:**
- WordPress
- Joomla
- Drupal
- Magento
- Shopify
- And 170+ more...

**Information Extracted:**
- CMS type and version
- Installed plugins/themes
- Known vulnerabilities
- Configuration issues

**Output:**
```
webs/cms_scanner.txt
```

**Sample Output:**
```
[WordPress] https://blog.example.com
  Version: 6.2.1
  Plugins: contact-form-7, yoast-seo
  Theme: flavor
  Users: admin, editor

[Drupal] https://cms.example.com
  Version: 9.4.0
  Modules: views, pathauto
```

**Configuration:**
```bash
CMS_SCANNER=true
CMSSCAN_TIMEOUT=3600
```

---

## Advanced Analysis

### `iishortname` - IIS Shortname Scanner

Exploits IIS shortname vulnerability to discover hidden files/directories.

**How It Works:**

Windows IIS servers may expose 8.3 format filenames through timing attacks, revealing:
- Hidden directories
- Backup files
- Configuration files

**Output:**
```
webs/iis_shortname.txt
```

**Sample Output:**
```
[FOUND] /BACKUP~1 -> likely: /backups, /backup_old
[FOUND] /CONFIG~1 -> likely: /config, /configuration
[FOUND] /ASPNET~1 -> likely: /aspnet_client
```

**Configuration:**
```bash
IIS_SHORTNAME=true
```

---

### `graphql_scan` - GraphQL Endpoint Detection

Discovers and analyzes GraphQL endpoints.

**What It Checks:**
- `/graphql`
- `/graphiql`
- `/api/graphql`
- Custom endpoints

**Analysis:**
- Introspection enabled?
- Schema extraction
- Query suggestions

**Output:**
```
webs/graphql_endpoints.txt
webs/graphql_introspection.json  # If GQLSPECTION enabled
```

**Configuration:**
```bash
GRAPHQL_CHECK=true
GQLSPECTION=false  # Deep introspection (heavier)
```

---

### `param_discovery` - Parameter Discovery

Discovers hidden parameters on web endpoints.

**How It Works:**

```
URLs with parameters → arjun → Fuzz parameter names →
→ Identify valid parameters → Output
```

**Output:**
```
webs/param_discovery.txt
```

**Sample Output:**
```
https://api.example.com/search
  Found: q, page, limit, sort, debug, admin
  
https://example.com/user
  Found: id, action, redirect, token
```

**Configuration:**
```bash
PARAM_DISCOVERY=true
ARJUN_THREADS=10
```

---

## Wordlist Generation

### `wordlist_gen` - Custom Wordlist Creation

Generates target-specific wordlists from discovered content.

**Sources:**
- JavaScript content
- HTML content
- URL paths
- Parameter names
- Domain-specific terms

**Output:**
```
webs/wordlist_custom.txt
```

**Configuration:**
```bash
WORDLIST=true
PASSWORD_DICT=true
PASSWORD_MIN_LENGTH=5
PASSWORD_MAX_LENGTH=14
```

---

### `wordlist_gen_roboxtractor` - Robots.txt Analysis

Extracts historical disallowed paths from Wayback Machine.

**How It Works:**

```
Target domain → Wayback Machine → Historical robots.txt →
→ Extract disallow paths → Custom wordlist
```

**Output:**
```
webs/robots_wordlist.txt
```

**Configuration:**
```bash
ROBOTSWORDLIST=true
```

---

### `password_dict` - Password Dictionary Generation

Generates target-specific password lists based on the domain name.

**How It Works:**

```
Domain name → Extract keywords → pydictor →
→ Apply leetspeak variations → Password wordlist
```

The function takes the first part of the domain (e.g., "target" from "target.com") and generates password variations using:
- Leetspeak transformations (a→4, e→3, etc.)
- Common suffixes (123, !, 2024, etc.)
- Length constraints

**Output:**
```
webs/password_dict.txt
```

**Sample Output (for target.com):**
```
target
Target
TARGET
t4rget
targ3t
target123
Target2024!
T4rg3t!
```

**Configuration:**
```bash
PASSWORD_DICT=true
PASSWORD_MIN_LENGTH=5      # Minimum password length
PASSWORD_MAX_LENGTH=14     # Maximum password length
```

**Use Cases:**
- Password spraying attacks (with authorization)
- Testing default/weak credential policies
- Generating custom wordlists for brute-force

---

### `grpc_reflection` - gRPC Reflection Probing

Discovers gRPC services with reflection enabled.

**What is gRPC Reflection?**

gRPC reflection allows clients to query a server for available services and methods without prior knowledge. When enabled (often for debugging), it exposes the entire API surface.

**How It Works:**

```
IPs from hosts/ips.txt → Probe gRPC ports (50051, 50052) →
→ grpcurl reflection query → List available services
```

**Output:**
```
hosts/grpc_reflection.txt
```

**Sample Output:**
```
[192.168.1.10:50051] grpc.reflection.v1alpha.ServerReflection
[192.168.1.10:50051] helloworld.Greeter
[192.168.1.10:50051] api.UserService
[192.168.1.10:50051] api.AdminService
```

**Configuration:**
```bash
GRPC_SCAN=false    # Disabled by default (requires grpcurl)
```

**Security Implications:**
- Exposed reflection reveals internal API structure
- Service names may reveal business logic
- Combined with protobuf enumeration = full API mapping

**Requirements:**
- `grpcurl` installed
- Network access to gRPC ports

---

## Output Summary

| File | Content |
|------|---------|
| `webs/webs.txt` | Live web servers |
| `webs/web_full_info_plain.txt` | Detailed probe results |
| `webs/url_extract.txt` | All discovered URLs |
| `screenshots/` | Web screenshots |
| `gf/*.txt` | Pattern-classified URLs |
| `js/js_secrets.txt` | JavaScript secrets |
| `fuzzing/` | Directory fuzzing results |
| `webs/cms_scanner.txt` | CMS detection results |

---

## Best Practices

1. **Rate Limiting:** Respect target resources with appropriate rate limits

2. **Scope Filtering:** Use `-x` flag to exclude out-of-scope URLs

3. **Screenshot Review:** Visual inspection often reveals interesting assets

4. **JS Analysis Priority:** JavaScript often contains the most valuable secrets

5. **Custom Wordlists:** Generated wordlists improve fuzzing effectiveness

---

## Next Steps

- **[Vulnerability Module](vulnerabilities.md)** - Test for security issues
- **[Output Interpretation](../07-output/output.md)** - Understand results
