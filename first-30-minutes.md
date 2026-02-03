# First 30 Minutes with reconFTW

Get from zero to your first scan results in 30 minutes or less.

---

## Minute 0-5: Install

```bash
# Clone and install
git clone https://github.com/six2dez/reconftw.git
cd reconftw
./install.sh
```

☕ **While it installs (~15-20 min):** Read the next sections.

---

## Minute 5-10: Understand What You're Running

### reconFTW in One Sentence

> reconFTW finds subdomains, probes them, and scans for vulnerabilities—automatically.

### Available Scan Modes

reconFTW offers multiple modes for different use cases:

| Mode | Flag | Description | Activity Level |
|------|------|-------------|----------------|
| **Passive** | `-p` | No direct contact with target, uses only public sources | None |
| **Subdomains** | `-s` | Subdomain enumeration only | Low-Medium |
| **OSINT** | `-n` | OSINT gathering only | Low |
| **Web** | `-w` | Web analysis on known subdomains | Medium |
| **Recon** | `-r` | **Default/recommended.** Full recon + light vuln scan (nuclei on webs) | Medium-High |
| **All** | `-a` | Full recon + aggressive vulnerability scanning | **Very High** |

> ⚠️ **Important:** Even `-r` mode performs active scanning (DNS queries, HTTP requests, port scans). Always ensure you have authorization.

### About the `-a` (All) Mode

> 🔴 **WARNING:** The `-a` flag runs aggressive vulnerability testing including SQLi payloads, fuzzing, and multiple scanner tools. This generates significant traffic and may trigger security alerts. Only use when you have **explicit written authorization** for penetration testing.

### Quick Decision Tree

```
Do I have written authorization for this target?
├── No → STOP. Get permission first.
└── Yes
    ├── First time? → Start with -p (passive)
    ├── Need subdomains only? → Use -s
    ├── Standard recon? → Use -r (recommended)
    └── Full pentest scope? → Use -a (read warning above)
```

---

## Minute 10-15: Configure API Keys (Optional but Recommended)

API keys improve results by adding more data sources. Set up at least these 3:

### 1. Shodan (Free tier available)
```bash
# Get key: https://shodan.io → Account → API Key
echo 'SHODAN_API_KEY="your_key"' >> secrets.cfg
```

### 2. GitHub Token (Free)
```bash
# Get token: GitHub → Settings → Developer settings → Personal access tokens
echo 'GITHUB_TOKEN="your_token"' >> secrets.cfg
```

### 3. SecurityTrails (Free tier)
```bash
# Get key: https://securitytrails.com/app/api
# Add to ~/.config/subfinder/provider-config.yaml
```

**No keys?** reconFTW still works, just with fewer data sources.

---

## Minute 15-20: Verify Installation

```bash
# Check all tools installed
./reconftw.sh --check-tools

# You should see green checkmarks ✓
# Red X means a tool failed - usually fixed by running install.sh again
```

### Common Issues

| Problem | Fix |
|---------|-----|
| `go: command not found` | `source ~/.bashrc` or restart terminal |
| Tool shows ✗ | Run `./install.sh` again |
| Permission denied | `chmod +x reconftw.sh` |

---

## Minute 20-25: Run Your First Scan

### Option A: Safe Passive Scan (Recommended First)

```bash
./reconftw.sh -d example.com -p
```

This:
- ✅ No direct contact with target
- ✅ Uses only public data sources
- ✅ Fast (~15 minutes)
- ✅ Safe for any authorized target

### Option B: Quick Subdomain Discovery

```bash
./reconftw.sh -d example.com -s
```

This:
- ⚠️ Makes DNS queries (minimal noise)
- ✅ Finds subdomains only
- ✅ ~30 minutes

### What's Happening?

```
[sub_passive] Running: Subdomain enumeration...
  → Querying 40+ data sources
  → Certificate Transparency logs
  → DNS brute-forcing (if -s or -r)
  
[webprobe_simple] Running: Web probing...
  → Checking which subdomains are alive
  → Detecting technologies
```

---

## Minute 25-30: Check Your Results

### Where Are Results?

```bash
ls Recon/example.com/
```

```
Recon/example.com/
├── subdomains/
│   └── subdomains.txt    ← All found subdomains
├── webs/
│   └── webs.txt          ← Live web servers
├── osint/
│   └── emails.txt        ← Found email addresses
└── .log/
    └── reconftw.log      ← Execution log
```

### Quick Results Check

```bash
# How many subdomains?
wc -l Recon/example.com/subdomains/subdomains.txt

# What's alive?
cat Recon/example.com/webs/webs.txt

# Any interesting findings?
cat Recon/example.com/osint/*.txt
```

---

## What's Next?

### If Passive Scan Looks Good → Run Full Recon

```bash
./reconftw.sh -d example.com -r
```

### If You Need Vulnerabilities → Run All

```bash
./reconftw.sh -d example.com -a
```

### If Scan Was Interrupted → Just Resume

```bash
# Run the same command again - it continues from where it stopped
./reconftw.sh -d example.com -r
```

---

## Quick Reference Card

### Essential Commands

```bash
# Passive (safe, fast)
./reconftw.sh -d target.com -p

# Recon (full discovery)
./reconftw.sh -d target.com -r

# All (recon + vulns)
./reconftw.sh -d target.com -a

# Multiple targets
./reconftw.sh -l targets.txt -r

# Resume interrupted scan
./reconftw.sh -d target.com -r  # just run again
```

### Essential Locations

| What | Where |
|------|-------|
| Results | `Recon/<domain>/` |
| Config | `reconftw.cfg` |
| API Keys | `secrets.cfg` |
| Logs | `Recon/<domain>/.log/` |

### Getting Help

```bash
./reconftw.sh -h              # Show help
./reconftw.sh --check-tools   # Verify installation
```

---

## Common First-Timer Questions

### "It's taking forever"

Normal! Full scans take 1-8 hours. Use `-p` for quick results.

### "I got rate limited"

Add to `reconftw.cfg`:
```bash
HTTPX_RATELIMIT=50
NUCLEI_RATELIMIT=50
```

### "Results are empty"

1. Check target exists: `dig target.com`
2. Check API keys are set
3. Look at logs: `cat Recon/target.com/.log/*.log`

### "Can I stop and resume?"

Yes! Just Ctrl+C to stop, run same command to resume.

---

## 🎉 You're Ready!

You now know enough to:
- [x] Run passive reconnaissance
- [x] Find subdomains
- [x] Check results
- [x] Resume interrupted scans

**Next steps:**
- **[Full Usage Guide](03-usage/usage.md)** - All flags explained
- **[Configuration](04-configuration/configuration.md)** - Customize behavior
- **[OPSEC & Legal](opsec-legal.md)** - Stay safe and legal
