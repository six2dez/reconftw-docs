# Documentation Audit Report

> **Generated:** February 2026  
> **Branch:** `dev`

---

## 1. Coverage Matrix: Functions vs Documentation

### OSINT Module (`osint.sh`)

| Function | Documented | File | Notes |
|----------|------------|------|-------|
| `google_dorks` | ✅ | osint.md | Complete |
| `github_dorks` | ✅ | osint.md | Complete |
| `github_repos` | ✅ | osint.md | Complete |
| `metadata` | ✅ | osint.md | Complete |
| `apileaks` | ✅ | osint.md | Complete |
| `emails` | ✅ | osint.md | Complete |
| `domain_info` | ✅ | osint.md | Complete |
| `third_party_misconfigs` | ✅ | osint.md | Complete |
| `spoof` | ✅ | osint.md | Complete |
| `mail_hygiene` | ✅ | osint.md | Complete |
| `cloud_enum_scan` | ✅ | osint.md | Complete |
| `ip_info` | ✅ | osint.md | Complete |

**Coverage: 12/12 (100%)**

---

### Subdomains Module (`subdomains.sh`)

| Function | Documented | File | Notes |
|----------|------------|------|-------|
| `sub_passive` | ✅ | subdomains.md | Complete |
| `sub_crt` | ✅ | subdomains.md | Complete |
| `sub_analytics` | ✅ | subdomains.md | Complete |
| `sub_brute` | ✅ | subdomains.md | Complete |
| `sub_noerror` | ✅ | subdomains.md | Complete |
| `sub_scraping` | ✅ | subdomains.md | Complete |
| `sub_tls` | ✅ | subdomains.md | Complete |
| `sub_permut` | ✅ | subdomains.md | Complete |
| `sub_ia_permut` | ✅ | subdomains.md | Complete |
| `sub_regex_permut` | ✅ | subdomains.md | Complete |
| `sub_recursive_passive` | ✅ | subdomains.md | Complete |
| `sub_recursive_brute` | ✅ | subdomains.md | Complete |
| `sub_dns` | ✅ | subdomains.md | Complete |
| `sub_active` | ⚠️ | subdomains.md | Mentioned but not detailed |
| `subtakeover` | ✅ | subdomains.md | Complete |
| `zonetransfer` | ✅ | subdomains.md | Complete |
| `s3buckets` | ✅ | subdomains.md | Complete |
| `geo_info` | ✅ | hosts.md | Documented in hosts module |
| `subdomains_full` | ❌ | - | Internal orchestration, skip |

**Coverage: 18/19 (95%)**

---

### Web Module (`web.sh`)

| Function | Documented | File | Notes |
|----------|------------|------|-------|
| `webprobe_simple` | ✅ | web-analysis.md | Complete |
| `webprobe_full` | ✅ | web-analysis.md | Complete |
| `favirecon_tech` | ✅ | web-analysis.md | Complete |
| `screenshot` | ✅ | web-analysis.md | Complete |
| `virtualhosts` | ✅ | web-analysis.md | Complete |
| `urlchecks` | ✅ | web-analysis.md | Complete |
| `url_gf` | ✅ | web-analysis.md | Complete |
| `url_ext` | ✅ | web-analysis.md | Complete |
| `jschecks` | ✅ | web-analysis.md | Complete |
| `fuzz` | ✅ | web-analysis.md | Complete |
| `cms_scanner` | ✅ | web-analysis.md | Complete |
| `wordlist_gen` | ✅ | web-analysis.md | Complete |
| `wordlist_gen_roboxtractor` | ✅ | web-analysis.md | **Documented (Phase 3)** |
| `iishortname` | ✅ | web-analysis.md | Complete |
| `graphql_scan` | ✅ | web-analysis.md | Complete |
| `grpc_reflection` | ✅ | web-analysis.md | **Documented (Phase 3)** |
| `param_discovery` | ✅ | web-analysis.md | Complete |
| `websocket_checks` | ⚠️ | web-analysis.md | Brief mention |
| `password_dict` | ✅ | web-analysis.md | **Documented (Phase 3)** |
| `portscan` | ✅ | hosts.md | In hosts doc |
| `cdnprovider` | ✅ | hosts.md | In hosts doc |
| `waf_checks` | ✅ | hosts.md | In hosts doc |
| `favicon` | ✅ | hosts.md | In hosts doc |
| `nuclei_check` | ✅ | vulnerabilities.md | In vulns doc |
| `brokenLinks` | ✅ | vulnerabilities.md | Complete |

**Coverage: 24/25 (96%)**

---

### Vulnerabilities Module (`vulns.sh`)

| Function | Documented | File | Notes |
|----------|------------|------|-------|
| `xss` | ✅ | vulnerabilities.md | Complete |
| `sqli` | ✅ | vulnerabilities.md | Complete |
| `ssrf_checks` | ✅ | vulnerabilities.md | Complete |
| `crlf_checks` | ✅ | vulnerabilities.md | Complete |
| `lfi` | ✅ | vulnerabilities.md | Complete |
| `ssti` | ✅ | vulnerabilities.md | Complete |
| `cors` | ✅ | vulnerabilities.md | Complete |
| `open_redirect` | ✅ | vulnerabilities.md | Complete |
| `command_injection` | ✅ | vulnerabilities.md | Complete |
| `prototype_pollution` | ✅ | vulnerabilities.md | Complete |
| `smuggling` | ✅ | vulnerabilities.md | Complete |
| `webcache` | ✅ | vulnerabilities.md | Complete |
| `4xxbypass` | ✅ | vulnerabilities.md | Complete |
| `fuzzparams` | ⚠️ | vulnerabilities.md | Brief |
| `test_ssl` | ✅ | vulnerabilities.md | Complete |
| `spraying` | ⚠️ | vulnerabilities.md | Brief |

**Coverage: 14/16 (88%)**

---

### Core/Utils/Modes (Internal Functions)

| Function | Should Document? | Notes |
|----------|------------------|-------|
| `start_func` / `end_func` | ✅ Advanced | Lifecycle management |
| `checkpoint_*` | ✅ Advanced | Checkpoint system |
| `circuit_breaker_*` | ⚠️ Advanced | Error handling |
| `cache_*` | ⚠️ Advanced | Caching system |
| `incremental_*` | ✅ Advanced | Incremental mode internals |
| `should_run_deep` | ✅ Advanced | DEEP mode logic |
| `notification` | ✅ Config | Already documented |
| `plugins_*` | ✅ Advanced | Plugin system |

**Note:** Internal functions need documentation in advanced.md for developers.

---

## 2. Gap Analysis: Missing Topics

### HIGH PRIORITY (User-Facing)

| Topic | Status | Recommended File |
|-------|--------|------------------|
| First 30 minutes guide | ❌ Missing | `first-30-minutes.md` |
| Tuning by target size | ❌ Missing | `tuning.md` |
| Input/Output data model | ⚠️ Partial (output.md) | `data-model.md` |
| Intrusiveness matrix | ❌ Missing | `opsec-legal.md` |
| Real case studies | ❌ Missing | `case-studies.md` |
| Common mistakes | ⚠️ Partial (troubleshooting) | Expand troubleshooting |
| Wordlist guide | ⚠️ Brief mentions | `tuning.md` or config |

### MEDIUM PRIORITY (Power Users)

| Topic | Status | Recommended Location |
|-------|--------|---------------------|
| CI/CD detailed examples | ⚠️ Basic | Expand `deployment.md` |
| Multi-target strategies | ⚠️ Basic | Expand `advanced.md` |
| Checkpoint internals | ⚠️ Concepts only | `advanced.md` |
| Plugin development | ⚠️ Basic | Expand `advanced.md` |
| Custom module creation | ✅ Basic | Could expand |

### LOW PRIORITY (Developers)

| Topic | Status | Notes |
|-------|--------|-------|
| Contributing guide | ❌ Missing | Link to CONTRIBUTING.md |
| Module API reference | ❌ Missing | For plugin developers |
| Exit codes reference | ⚠️ Mentioned | Could detail more |

---

## 3. Duplications & Overlaps

| Content | Appears In | Recommendation |
|---------|-----------|----------------|
| OPSEC/Legal | README.md, concepts.md | **Consolidate to opsec-legal.md** |
| Recommended workflows | concepts.md | **Move to playbooks.md** |
| API keys setup | config.md, getting-started.md, FAQ | Keep in config, reference elsewhere |
| Installation | getting-started.md, deployment.md | OK - different depth levels |
| Tool descriptions | tools.md, various modules | OK - tools.md is reference |
| Checkpoint system | concepts.md, advanced.md | OK - concepts=overview, advanced=details |
| -c flag usage | usage.md, FAQ, advanced.md | Slight redundancy, OK |

---

## 4. Undocumented Functions (New/Missing)

### Must Add:
1. No critical missing function docs identified in current modules.

### Should Expand:
1. `graphql_scan` - More details needed
2. `websocket_checks` - More details needed
3. `fuzzparams` - More details needed
4. `spraying` - More details needed

---

## 5. Proposed New SUMMARY.md Structure

```markdown
# Table of Contents

## Start Here
* [Introduction](README.md)
* [First 30 Minutes](first-30-minutes.md)
* [FAQ](FAQ.md)

## User Guides (by Profile)
* [Case Studies](case-studies.md) - Bug bounty, Pentest, Security team examples

## Installation & Setup
* [Getting Started](01-getting-started/getting-started.md)
* [Deployment Options](09-deployment/deployment.md)

## Core Concepts
* [Architecture & Concepts](02-concepts/concepts.md)
* [OPSEC & Legal](opsec-legal.md)

## Using reconFTW
* [Command Line Reference](03-usage/usage.md)
* [Configuration Deep Dive](04-configuration/configuration.md)
* [Performance Tuning](tuning.md)

## Modules Reference
* [Modules Overview](05-modules/README.md)
* [OSINT](05-modules/osint.md)
* [Subdomains](05-modules/subdomains.md)
* [Host Analysis](05-modules/hosts.md)
* [Web Analysis](05-modules/web-analysis.md)
* [Vulnerabilities](05-modules/vulnerabilities.md)

## Understanding Output
* [Output Interpretation](07-output/output.md)
* [Data Model & I/O](data-model.md)

## Integrations
* [Axiom](08-integrations/axiom.md)
* [Faraday](08-integrations/faraday.md)
* CI/CD examples in [Deployment Guide](09-deployment/deployment.md#cicd-integration)

## Advanced Topics
* [Advanced Usage](10-advanced/advanced.md)
* [Tools Reference](06-tools/tools.md)

## Help & Reference
* [Troubleshooting](11-troubleshooting/troubleshooting.md)
* [Case Studies](case-studies.md) ← NEW
* [Glossary](GLOSSARY.md)
```

---

## 6. Summary Statistics

| Metric | Value |
|--------|-------|
| Total functions in code | ~150 |
| User-facing functions | ~70 |
| Documented functions | ~60 |
| **Function coverage** | **~85%** |
| Current pages | 22 |
| Proposed new pages | 7 |
| Duplications found | 3 (minor) |
| High-priority gaps | 5 |

---

## 7. Recommended Action Plan

### Immediate (Phase 2):
1. Create `first-30-minutes.md`
2. Create `opsec-legal.md` (extract + expand)
3. Create `tuning.md`
4. Create `data-model.md`
5. Create `case-studies.md`

### Short-term (Phase 3):
1. Document missing functions (grpc_reflection, etc.)
2. Expand brief function docs
3. Add intrusiveness matrix
4. Add CI/CD examples

### Maintenance (Phase 5+):
1. Auto-generation script for flags/config
2. Git metadata system
3. Cross-link validation
