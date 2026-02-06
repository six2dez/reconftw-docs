# reconFTW Documentation


<p align="center">
  <img src="https://github.com/six2dez/reconftw/blob/main/images/banner.png" alt="reconFTW Banner">
</p>

<p align="center">
  <strong>Automated Reconnaissance Framework</strong>
</p>

<p align="center">
  <a href="https://github.com/six2dez/reconftw">GitHub</a> •
  <a href="https://twitter.com/Six2dez1">Twitter</a> •
  <a href="https://discord.gg/R5DdXVEdTy">Discord</a> •
  <a href="https://t.me/joinchat/H5bAaw3YbzzmI5co">Telegram</a>
</p>

---

## Welcome to reconFTW

**reconFTW** is a modular reconnaissance automation framework designed for security researchers, penetration testers, and bug bounty hunters. It orchestrates 80+ security tools to perform full reconnaissance on your targets, from subdomain enumeration to vulnerability scanning.

### Why reconFTW?

| Feature | Description |
|---------|-------------|
| **Automated Workflow** | Complete reconnaissance pipeline with a single command |
| **Modular Design** | Enable/disable any module or function as needed |
| **Distributed Scanning** | Scale with [Axiom](08-integrations/axiom.md) across cloud infrastructure |
| **Structured Output** | Organized results with multiple export formats |
| **Continuous Monitoring** | Recurrent scan cycles with delta and alert snapshots |
| **Report Rebuild Mode** | Recreate report/export artifacts from existing scan data |
| **Highly Configurable** | 300+ configuration options for fine-tuning |
| **Incremental Scans** | Only scan new findings since last run |
| **AI Integration** | Generate structured JSON + markdown/txt security summaries with local AI models |

### What Can reconFTW Do?

```
┌─────────────────────────────────────────────────────────────────┐
│                        reconFTW Capabilities                     │
├─────────────────────────────────────────────────────────────────┤
│  OSINT           │ Google dorks, GitHub secrets, metadata,      │
│                  │ email harvesting, API leaks, cloud enum,     │
│                  │ leaked credentials, S3 buckets               │
├──────────────────┼──────────────────────────────────────────────┤
│  Subdomains      │ 10+ passive sources, DNS bruteforce,         │
│                  │ permutations (AI-powered), recursive enum,   │
│                  │ CT logs, scraping, zone transfer, takeover   │
├──────────────────┼──────────────────────────────────────────────┤
│  Web Analysis    │ HTTP probing, screenshots, JS secrets,       │
│                  │ URL extraction, directory fuzzing, CMS,      │
│                  │ virtual hosts, parameters, GraphQL, gRPC     │
├──────────────────┼──────────────────────────────────────────────┤
│  Vulnerabilities │ Nuclei templates, XSS, SQLi, SSRF, LFI,     │
│                  │ SSTI, CORS, CRLF, command injection,         │
│                  │ prototype pollution, 403 bypass, smuggling   │
├──────────────────┼──────────────────────────────────────────────┤
│  Host Analysis   │ Port scanning (nmap/naabu), CDN detection,   │
│                  │ WAF fingerprinting, geolocation, banners     │
├──────────────────┼──────────────────────────────────────────────┤
│  Automation      │ Checkpoint/resume system, incremental scans, │
│                  │ notifications (Slack/Discord/Telegram),      │
│                  │ Axiom distributed scanning, AI reports       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Quick Start

```bash
# Install reconFTW
git clone https://github.com/six2dez/reconftw.git
cd reconftw
./install.sh

# Run your first scan
./reconftw.sh -d example.com -r

# Full scan with vulnerabilities
./reconftw.sh -d example.com -a
```

---

## Documentation Overview

This documentation is organized to help you get the most out of reconFTW:

### 📚 For Beginners

1. **[First 30 Minutes](first-30-minutes.md)** - Quick start guide to get scanning
2. **[Getting Started](01-getting-started/getting-started.md)** - Installation and setup
3. **[Concepts](02-concepts/concepts.md)** - Understanding how reconFTW works
4. **[Usage Guide](03-usage/usage.md)** - All command-line options explained

### 🔧 For Configuration

5. **[Configuration](04-configuration/configuration.md)** - Deep dive into reconftw.cfg
6. **[Modules](05-modules/)** - Detailed documentation for each module
7. **[Tools Reference](06-tools/tools.md)** - All 80+ integrated tools

### 📊 For Results

8. **[Output Interpretation](07-output/output.md)** - Understanding your results
9. **[Data Model & I/O](data-model.md)** - Complete input/output reference
10. **[Integrations](08-integrations/)** - Axiom and Faraday setup

### For Advanced Users

11. **[Deployment](09-deployment/deployment.md)** - Docker, Terraform, VPS, CI/CD
12. **[Performance Tuning](tuning.md)** - Optimize for speed and target size
13. **[Case Studies](case-studies.md)** - Real-world usage examples
14. **[Advanced Usage](10-advanced/advanced.md)** - Custom functions and optimization
15. **[Troubleshooting](11-troubleshooting/troubleshooting.md)** - Common issues and solutions
16. **[Release Gate](release-gate.md)** - Required checks before publishing updates

### ⚖️ Legal & Security

17. **[OPSEC & Legal](opsec-legal.md)** - Stay safe and authorized

---

## Scan Modes at a Glance

| Mode | Flag | Description | Use Case |
|------|------|-------------|----------|
| **Recon** | `-r` | Full reconnaissance | Standard bug bounty recon |
| **Subdomains** | `-s` | Subdomain enumeration only | Quick subdomain discovery |
| **Passive** | `-p` | Passive reconnaissance | Stealth/non-intrusive |
| **All** | `-a` | Full recon + vulnerabilities | Full assessment |
| **Web** | `-w` | Web analysis only | Analyze known URLs |
| **OSINT** | `-n` | OSINT gathering only | Intelligence gathering |
| **Custom** | `-c` | Run custom function | Advanced workflows |
| **Zen** | `-z` | Minimal output mode | Clean terminal output |

Additional workflow flags:
- `--monitor`, `--monitor-interval`, `--monitor-cycles`
- `--report-only`
- `--export json|html|csv|all`
- `--refresh-cache`

Bundled config profiles:
- `config/reconftw_quick.cfg`
- `config/reconftw_full.cfg`
- `config/reconftw_stealth.cfg`

---

## ⚠️ Legal & OPSEC

> **IMPORTANT**: reconFTW is designed for authorized security testing only.

### Authorization Checklist

Before running any scan, verify:

- [ ] Written permission from target owner
- [ ] Defined scope (in-scope and out-of-scope assets)
- [ ] Rate limits agreed upon
- [ ] Testing window defined (if applicable)
- [ ] Emergency contact available
- [ ] NDA signed (if required)

### OPSEC Considerations

| Risk | Mitigation |
|------|------------|
| **IP Blocking** | Use VPS, rotate IPs with Axiom |
| **WAF Detection** | Start with passive mode (`-p`) |
| **Rate Limiting** | Use `--adaptive-rate` flag |
| **Legal Issues** | Always have written authorization |
| **Data Exposure** | Keep `secrets.cfg` secure, never commit |

### Legal Disclaimer

By using this tool, you confirm that:

- You have explicit written permission to test the target
- You will comply with all applicable laws and regulations
- You understand that unauthorized testing is illegal

The developers assume no liability for misuse of this tool. **Use responsibly.**

➡️ **[Full OPSEC Guide](02-concepts/concepts.md#opsec-and-legal)**

---

## Community & Support

- **GitHub Issues**: [Report bugs or request features](https://github.com/six2dez/reconftw/issues)
- **Discord**: [Join our community](https://discord.gg/R5DdXVEdTy)
- **Telegram**: [Discussion group](https://t.me/joinchat/H5bAaw3YbzzmI5co)
- **Twitter**: [@Six2dez1](https://twitter.com/Six2dez1)

---

## Contributing

reconFTW is open source and welcomes contributions! See our [Contributing Guide](https://github.com/six2dez/reconftw/blob/main/CONTRIBUTING.md) for details.

---

<p align="center">
  Made with ❤️ by <a href="https://github.com/six2dez">six2dez</a> and the security community
</p>

---
