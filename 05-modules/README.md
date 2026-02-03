# Modules Overview

reconFTW is organized into specialized modules, each handling a specific phase of reconnaissance. This page provides an overview and quick navigation to each module's detailed documentation.

---

## Module Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                      reconFTW Module System                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐                                                    │
│  │    Target    │                                                    │
│  │   Input      │                                                    │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐     Intelligence gathering                        │
│  │    OSINT     │────▶ Dorks, emails, metadata, leaks               │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐     Asset discovery                               │
│  │  Subdomains  │────▶ Passive, brute, permutations, takeover       │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐     Infrastructure analysis                       │
│  │    Hosts     │────▶ Ports, CDN, WAF, geolocation                 │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐     Web application analysis                      │
│  │ Web Analysis │────▶ Probing, screenshots, JS, fuzzing            │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐     Security testing                              │
│  │Vulnerabilities│───▶ Nuclei, XSS, SQLi, SSRF, etc.                │
│  └──────────────┘                                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Module Summary

| Module | Functions | Primary Tools | Output Directory |
|--------|-----------|---------------|------------------|
| [OSINT](osint.md) | 12 | theHarvester, gitdorker, trufflehog | `osint/` |
| [Subdomains](subdomains.md) | 17 | subfinder, amass, puredns, dnsx | `subdomains/` |
| [Hosts](hosts.md) | 6 | nmap, smap, cdncheck, wafw00f | `hosts/` |
| [Web Analysis](web-analysis.md) | 17 | httpx, katana, ffuf, gowitness | `webs/`, `fuzzing/`, `js/` |
| [Vulnerabilities](vulnerabilities.md) | 18 | nuclei, dalfox, sqlmap, ssrf-sheriff | `vulns/` |

---

## OSINT Module

**Purpose:** Gather intelligence about the target organization without direct interaction.

**Key Capabilities:**
- Google dorking for sensitive files
- GitHub secret scanning
- Email harvesting
- Document metadata extraction
- Cloud storage enumeration
- API leak detection

**When to use:** At the beginning of an engagement for initial intelligence gathering.

➡️ **[Full OSINT Documentation](osint.md)**

---

## Subdomains Module

**Purpose:** Discover all subdomains associated with the target domain.

**Key Capabilities:**
- Passive enumeration (40+ sources)
- Certificate Transparency logs
- DNS brute-forcing
- Permutation generation
- Recursive enumeration
- Subdomain takeover detection

**When to use:** After OSINT, to map the attack surface.

➡️ **[Full Subdomains Documentation](subdomains.md)**

---

## Hosts Module

**Purpose:** Analyze infrastructure behind discovered assets.

**Key Capabilities:**
- Port scanning (passive + active)
- CDN detection and filtering
- WAF identification
- IP geolocation
- Favicon-based IP discovery

**When to use:** After subdomain enumeration, to understand infrastructure.

➡️ **[Full Hosts Documentation](hosts.md)**

---

## Web Analysis Module

**Purpose:** Analyze web applications and discover endpoints.

**Key Capabilities:**
- HTTP probing and status detection
- Screenshot capture
- URL extraction from archives
- JavaScript analysis for secrets
- Directory/file fuzzing
- Technology detection

**When to use:** After identifying live web servers.

➡️ **[Full Web Analysis Documentation](web-analysis.md)**

---

## Vulnerabilities Module

**Purpose:** Identify security vulnerabilities in discovered assets.

**Key Capabilities:**
- Template-based scanning (Nuclei)
- XSS testing
- SQL injection detection
- SSRF testing
- CORS misconfiguration
- SSL/TLS analysis
- And many more...

**When to use:** Final phase, after mapping all assets.

➡️ **[Full Vulnerabilities Documentation](vulnerabilities.md)**

---

## Module Execution Order

In a full scan (`-a` flag), modules execute in this order:

```
1. OSINT          → Intelligence gathering
2. Subdomains     → Asset discovery  
3. Hosts          → Infrastructure analysis
4. Web Analysis   → Application mapping
5. Vulnerabilities → Security testing
```

Each module builds on the previous one's output, creating a full reconnaissance pipeline.

---

## Enabling/Disabling Modules

### Via Configuration

```bash
# In reconftw.cfg
OSINT=true
SUBDOMAINS_GENERAL=true
PORTSCANNER=true
WEBPROBESIMPLE=true
WEBPROBEFULL=true
NUCLEICHECK=true
# Use VULNS_GENERAL=true if you want to enable the full vulnerability module by config
```

### Via Command Line

```bash
# Run only specific modules
./reconftw.sh -d example.com -s          # Subdomains only
./reconftw.sh -d example.com -n          # OSINT only
./reconftw.sh -d example.com -w          # Web analysis only

# Custom function selection
./reconftw.sh -d example.com -c sub_passive
./reconftw.sh -d example.com -c webprobe_simple
```

---

## Module Dependencies

Some modules depend on outputs from others:

```
OSINT ─────────────────────────────────────┐
                                           │
Subdomains ───────┬────────────────────────┤
                  │                        │
                  ▼                        │
              Hosts ───────────────────────┤
                  │                        │
                  ▼                        │
           Web Analysis ───────────────────┤
                  │                        │
                  ▼                        │
          Vulnerabilities ◄────────────────┘
```

---

## Next Steps

Choose a module to explore in detail:

- **[OSINT Module](osint.md)** - Start with intelligence gathering
- **[Subdomains Module](subdomains.md)** - Discover your attack surface
- **[Hosts Module](hosts.md)** - Understand the infrastructure
- **[Web Analysis Module](web-analysis.md)** - Map web applications
- **[Vulnerabilities Module](vulnerabilities.md)** - Find security issues
