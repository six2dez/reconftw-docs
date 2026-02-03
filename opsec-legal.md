# OPSEC & Legal Guide

Operational security and legal considerations for using reconFTW responsibly.

---

## ⚠️ The Golden Rule

> **NEVER scan a target without explicit written authorization.**

This isn't just advice—it's the law in most jurisdictions. Unauthorized scanning can result in:
- Criminal charges
- Civil lawsuits
- Career damage
- Account bans from bug bounty platforms

---

## Authorization Checklist

Before ANY scan, verify you have:

```markdown
## Pre-Scan Authorization Checklist

### Legal Authorization
- [ ] Written permission from asset owner (email, contract, or bug bounty policy)
- [ ] Scope document clearly defines in-scope assets
- [ ] Out-of-scope assets are documented and excluded
- [ ] Testing window defined (if applicable)
- [ ] Rate limits agreed upon (if any)

### Technical Preparation
- [ ] Scope file created (`-i inscope.txt`)
- [ ] Exclusions configured (`-x outofscope.txt`)
- [ ] Rate limits set appropriately
- [ ] VPS/cloud instance ready (recommended)

### Communication
- [ ] Emergency contact identified
- [ ] Reporting channel established
- [ ] NDA signed (if required)
```

---

## Scan Intrusiveness Matrix

Understanding how "noisy" each mode is:

### By Mode

| Mode | Intrusiveness | Detection Risk | Direct Contact |
|------|---------------|----------------|----------------|
| `-p` (passive) | 🟢 None | None | No |
| `-n` (OSINT) | 🟢 Minimal | Very Low | Minimal |
| `-s` (subdomains) | 🟡 Low | Low | DNS only |
| `-r` (recon) | 🟡 Medium | Medium | Yes |
| `-a` (all) | 🔴 High | High | Yes + Vuln tests |

### By Function

| Function | Intrusiveness | Notes |
|----------|---------------|-------|
| `sub_passive` | 🟢 None | Third-party APIs only |
| `sub_crt` | 🟢 None | Certificate Transparency |
| `sub_brute` | 🟡 Low | DNS queries |
| `webprobe_simple` | 🟡 Medium | HTTP requests |
| `screenshot` | 🟡 Medium | HTTP requests |
| `fuzz` | 🟠 High | Many HTTP requests |
| `nuclei_check` | 🔴 Very High | Active vulnerability testing |
| `sqli` | 🔴 Very High | SQL injection attempts |
| `xss` | 🔴 Very High | XSS payload injection |

### Noise Levels Explained

- 🟢 **None/Minimal**: No direct target contact, uses public data
- 🟡 **Low/Medium**: Standard HTTP/DNS traffic, looks like normal browsing
- 🟠 **High**: Elevated traffic, may trigger WAF/IDS alerts
- 🔴 **Very High**: Attack-like traffic, will likely trigger security alerts

---

## Reducing Detection Risk

### Start Passive

```bash
# Always start with passive reconnaissance
./reconftw.sh -d target.com -p

# Review results before going active
cat Recon/target.com/subdomains/subdomains.txt
```

### Use Rate Limiting

```bash
# In reconftw.cfg
HTTPX_RATELIMIT=50        # Requests per second
NUCLEI_RATELIMIT=50
FFUF_RATELIMIT=50

# Or use adaptive rate limiting
./reconftw.sh -d target.com -r --adaptive-rate
```

### Use Scope Files

```bash
# inscope.txt - Only scan these
*.example.com
api.example.org

# outofscope.txt - Never scan these
admin.example.com
legacy.example.com
*.internal.example.com
```

```bash
./reconftw.sh -d example.com -r -i inscope.txt -x outofscope.txt
```

### Use a VPS

Scanning from your personal IP:
- Links activity directly to you
- May get your home IP blocked
- Exposes your location

Instead:
```bash
# Use a cloud VPS (DigitalOcean, Linode, etc.)
# Or use Axiom for distributed scanning
./reconftw.sh -d target.com -r -v
```

---

## Legal Framework by Region

### United States

| Law | What It Covers |
|-----|----------------|
| **CFAA** (Computer Fraud and Abuse Act) | Unauthorized access to computer systems |
| **ECPA** | Wiretapping and electronic surveillance |
| State laws | Vary by state |

**Key Points:**
- "Exceeding authorized access" is a federal crime
- Bug bounty policies = authorization
- Always document permission

### European Union

| Framework | What It Covers |
|-----------|----------------|
| **GDPR** | Personal data handling |
| **NIS Directive** | Network security |
| National laws | Vary by country |

**Key Points:**
- Stricter consent requirements
- Data handling must comply with GDPR
- Some countries require explicit contracts

### United Kingdom

| Law | What It Covers |
|-----|----------------|
| **Computer Misuse Act 1990** | Unauthorized access |
| **GDPR UK** | Data protection |

**Key Points:**
- Even attempting unauthorized access is illegal
- "With intent to commit further offences" increases penalties

### General International

- Always research local laws before testing
- Consider both your location AND target's location
- When in doubt, get explicit written permission

---

## Bug Bounty Specific Guidelines

### Before Testing

1. **Read the policy completely**
   - Safe harbor language
   - Scope (domains, IPs, applications)
   - Out-of-scope items
   - Prohibited actions

2. **Note exclusions** (common ones):
   - DoS/DDoS testing
   - Social engineering
   - Physical attacks
   - Third-party services
   - Rate limit abuse

3. **Check asset types**:
   - Some exclude certain subdomains
   - API vs web app rules may differ
   - Mobile apps often have different rules

### During Testing

```bash
# Respect rate limits
./reconftw.sh -d target.com -r -q 30  # 30 req/sec max

# Stay in scope
./reconftw.sh -d target.com -r -x outofscope.txt

# Use passive first
./reconftw.sh -d target.com -p  # Then review before -r
```

### What NOT to Do

| ❌ Don't | Why |
|----------|-----|
| Test without reading policy | Different programs, different rules |
| Ignore scope | Instant ban risk |
| Chain to access other data | May cross legal lines |
| Store sensitive data | GDPR/privacy issues |
| Share findings publicly | Violates responsible disclosure |
| Test production during peak hours | May cause outages |

---

## Data Handling

### What reconFTW Collects

| Data Type | Location | Sensitivity |
|-----------|----------|-------------|
| Subdomains | `subdomains/` | Low |
| URLs | `webs/` | Low-Medium |
| Emails | `osint/emails.txt` | Medium |
| Leaked passwords | `osint/passwords.txt` | **High** |
| Vulnerabilities | `vulns/` | **High** |
| Screenshots | `screenshots/` | Medium |

### Security Practices

```bash
# 1. Protect secrets.cfg
chmod 600 secrets.cfg

# 2. Don't commit sensitive files
# .gitignore already includes:
# - secrets.cfg
# - Recon/

# 3. Encrypt sensitive results
tar czf results.tar.gz Recon/target.com/
gpg -c results.tar.gz

# 4. Secure deletion when done
srm -r Recon/target.com/  # Or use shred
```

### Reporting Guidelines

When you find vulnerabilities:

1. **Don't access more than necessary** to prove the bug
2. **Don't exfiltrate data** beyond proof
3. **Document everything** (timestamps, commands, evidence)
4. **Report promptly** through official channels
5. **Wait for authorization** before public disclosure

---

## Incident Response

### If You Find Critical Data

```
1. STOP further testing immediately
2. Document what you accessed (screenshot, timestamp)
3. Report through fastest channel available
4. Do NOT share with anyone else
5. Securely delete any copies
```

### If Something Breaks

```
1. STOP scanning immediately
2. Document what happened (command run, time, error)
3. Contact emergency point immediately
4. Preserve evidence of authorization
5. Do NOT try to "fix" anything on target
```

### If You Receive a Cease & Desist

```
1. STOP all testing immediately
2. Preserve all documentation of authorization
3. Do NOT delete anything
4. Contact legal counsel
5. Respond professionally
```

---

## VPS and Infrastructure OPSEC

### Recommended Setup

```bash
# Use cloud VPS, not home IP
# Provider: DigitalOcean, Linode, Vultr, Hetzner

# Change hostname to something neutral
hostnamectl set-hostname scanner-01

# Use separate VPS per engagement (if possible)
```

### What NOT to Do

| ❌ Don't | ✅ Do Instead |
|----------|---------------|
| Scan from home IP | Use cloud VPS |
| Reuse VPS across clients | Fresh instance per engagement |
| Store results on VPS long-term | Download and destroy |
| Use personal accounts | Create engagement-specific accounts |

---

## Quick Reference: Safe Scanning Checklist

```markdown
## Before Scan
- [ ] Authorization documented
- [ ] Scope file ready
- [ ] VPS configured
- [ ] Rate limits set
- [ ] Start with passive mode

## During Scan
- [ ] Monitor for errors
- [ ] Stay in scope
- [ ] Respect rate limits
- [ ] Watch for critical findings

## After Scan
- [ ] Review all findings
- [ ] Report vulnerabilities
- [ ] Secure/delete sensitive data
- [ ] Document for records
```

---

## Additional Resources

- **[EFF Know Your Rights](https://www.eff.org/issues/cfaa)** - CFAA information
- **[HackerOne Policy Guidelines](https://www.hackerone.com/disclosure-guidelines)** - Disclosure best practices
- **[Bugcrowd VRT](https://bugcrowd.com/vulnerability-rating-taxonomy)** - Vulnerability classification
- **[OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)** - Ethical testing methodology

---

## TL;DR

1. **Get written permission** before any scan
2. **Start passive** (`-p`), then go active
3. **Use rate limiting** and scope files
4. **Scan from VPS**, not personal IP
5. **Report responsibly**, don't overstep
6. **When in doubt, ask** - it's better to ask than apologize

---

> **Documentation Info**  
> Branch: `dev` | Version: `v3.0.0+` | Last updated: February 2026
