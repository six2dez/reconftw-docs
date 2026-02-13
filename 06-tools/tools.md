# Tools Reference

This page documents the tools currently integrated in reconFTW.

---

## Installation Verification

```bash
./reconftw.sh --check-tools
./reconftw.sh --health-check
```

---

## OSINT Tools

| Tool | Purpose |
|------|---------|
| `dorks_hunter` | Google dork automation |
| `gitdorks_go` | GitHub dorking/secrets |
| `enumerepo` | Repository enumeration |
| `gitleaks` | Secret scanning in code |
| `trufflehog` | Secret scanning in files/repos |
| `metagoofil` | Public document collection |
| `exiftool` | Metadata extraction |
| `porch-pirate` | Postman public search |
| `SwaggerSpy` | Swagger/OpenAPI discovery |
| `postleaksNg` | Postman leak discovery (JSON output) |
| `EmailHarvester` | Email discovery |
| `LeakSearch` | Breach/credential correlation |
| `whois` | Domain intelligence |
| `msftrecon` | Microsoft tenant discovery |
| `Scopify` | Related domain/company scope mapping |
| `misconfig-mapper` | Third-party misconfiguration checks |
| `Spoofy` | Spoofable domain checks |
| `cloud_enum` | Cloud storage enumeration |

---

## Subdomain and DNS Tools

| Tool | Purpose |
|------|---------|
| `subfinder` | Passive subdomain enumeration |
| `github-subdomains` | Subdomains from GitHub code |
| `gitlab-subdomains` | Subdomains from GitLab code |
| `crt` | Certificate transparency enumeration |
| `dnsx` | DNS resolution and records |
| `puredns` | High-scale DNS resolution/bruteforce |
| `gotator` | Permutation generation |
| `ripgen` | Fast permutation generation |
| `subwiz` | AI permutation generation |
| `regulator` | Regex-based permutations |
| `urlfinder` | Passive URL/subdomain source |
| `waymore` | Historical/passive URL sources |
| `csprecon` | Subdomains from CSP parsing |
| `tlsx` | TLS certificate-based discovery |
| `dnstake` | Subdomain takeover checks |
| `s3scanner` | S3 bucket checks |
| `CloudHunter` | Cloud takeovers/misconfig checks |
| `dsieve` | Ranking/prioritization for recursive workflows |
| `hakip2host` | Reverse IP to host candidates |
| `asnmap` | ASN enumeration |

---

## Web and Content Discovery Tools

| Tool | Purpose |
|------|---------|
| `httpx` | HTTP probing and metadata |
| `nuclei` | Screenshot module and web checks |
| `VhostFinder` | Virtual host discovery |
| `fav-up` | Favicon-based real IP lookup |
| `favirecon` | Favicon-based technology fingerprinting |
| `katana` | Active crawling |
| `github-endpoints` | Endpoints from GitHub |
| `JSA` | JavaScript endpoint extraction |
| `gf` | URL pattern classification |
| `urless` | URL deduplication/normalization |
| `subjs` | JS file discovery |
| `xnLinkFinder` | Endpoint discovery in JS |
| `sourcemapper` | Source map extraction |
| `jsluice` | JS URL/secret extraction |
| `mantra` | JS secret hunting |
| `ffuf` | Directory and endpoint fuzzing |
| `CMSeeK` | CMS fingerprinting |
| `pydictor` | Password dictionary generation |
| `shortscan` | IIS shortname detection |
| `sns` | IIS shortname checks |
| `gqlspection` | GraphQL analysis |
| `arjun` | Parameter discovery |
| `grpcurl` | gRPC reflection probing |
| `wget` | Auxiliary content retrieval |

---

## Host and Infrastructure Tools

| Tool | Purpose |
|------|---------|
| `smap` | Passive port intelligence |
| `nmap` | Active port scanning |
| `nmapurls` | URL extraction from Nmap XML |
| `cdncheck` | CDN provider detection |
| `wafw00f` | WAF detection |

---

## Vulnerability Tools

| Tool | Purpose |
|------|---------|
| `dalfox` | XSS testing |
| `Gxss` | Reflected parameter identification |
| `qsreplace` | Parameter mutation for payload injection |
| `Corsy` | CORS misconfiguration checks |
| `Oralyzer` | Open redirect checks |
| `interactsh-client` | OOB callbacks for blind vulns |
| `crlfuzz` | CRLF injection checks |
| `interlace` | Parallelized target execution |
| `tinja` | SSTI detection (default engine) |
| `sqlmap` | SQL injection testing |
| `ghauri` | SQL injection testing (alternative) |
| `testssl.sh` | TLS/SSL checks |
| `brutespray` | Password spraying |
| `commix` | Command injection checks |
| `nomore403` | 403/401 bypass checks |
| `ppmap` | Prototype pollution checks |
| `smugglex` | HTTP request smuggling checks |
| `Web-Cache-Vulnerability-Scanner` | Web cache poisoning checks |
| `toxicache` | Complementary web cache poisoning checks |
| `second-order` | Broken links / takeover-oriented checks |

---

## Framework and Utility Tools

| Tool | Purpose |
|------|---------|
| `anew` | Incremental dedup append |
| `unfurl` | URL component parsing |
| `mapcidr` | CIDR manipulation |
| `dnsvalidator` | Resolver validation |
| `notify` | Notifications |
| `axiom-scan` / `axiom-exec` | Distributed execution |

---

## Support Assets and Helpers

These components are installed and maintained by reconFTW even when they are not always invoked directly as standalone commands in every scan:

| Component | Purpose |
|-----------|---------|
| `Gf-Patterns` | Additional `gf` pattern packs |
| `sus_params` | Supplemental parameter patterns |
| `ffufPostprocessing` | Post-processing helper for ffuf workflows |
| `ultimate-nmap-parser` | Nmap output parsing helper scripts |
| `xnldorker` | Extra dorking dependency/helpers in dorking stack |
| `p1radup` | Auxiliary recon helper package |

---

## Tool Notes

- Some tools are optional by configuration and mode.
- A few tools are only used in specific branches (for example Axiom workflows).
- Python-based tools are usually installed under tool-specific virtual environments.

---

## API-Dependent Tools

These require credentials in `secrets.cfg` or environment variables:

| Tool/Feature | Credential |
|--------------|------------|
| Passive port data (`smap`) | `SHODAN_API_KEY` |
| WhoisXML API calls | `WHOISXML_API` |
| ASN enumeration (`asnmap`) | `PDCP_API_KEY` |
| GitHub dorks/endpoints | `GITHUB_TOKENS` |
| GitLab subdomains | `GITLAB_TOKENS` |

---

## Next Steps

- [Modules Overview](../05-modules/README.md)
- [Configuration Reference](../04-configuration/configuration.md)
- [Output Interpretation](../07-output/output.md)
