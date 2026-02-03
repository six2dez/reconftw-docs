# Frequently Asked Questions

Common questions and answers about reconFTW.

---

## General Questions

### What is reconFTW?

reconFTW is an automated reconnaissance framework that orchestrates 80+ security tools to perform comprehensive reconnaissance on targets. It's designed for bug bounty hunters, penetration testers, and security researchers.

### Is reconFTW free?

Yes, reconFTW is completely free and open source under the MIT license.

### What operating systems are supported?

- **Linux** (Ubuntu/Debian recommended) - Full support
- **macOS** - Supported with GNU tools installed via Homebrew
- **Windows** - Via WSL2 (Windows Subsystem for Linux)

### Do I need root/sudo access?

Only for the initial installation of system dependencies. Normal scans run without elevated privileges.

---

## Installation Questions

### How long does installation take?

Initial installation typically takes 15-30 minutes depending on your internet connection and system speed. Most time is spent downloading and compiling Go tools.

### Can I install on a VPS?

Yes! VPS deployment is recommended for large scans. See the [Deployment Guide](09-deployment/deployment.md) for detailed instructions.

### Which cloud provider is best?

Any provider works. Popular choices:
- **DigitalOcean** - Simple, good pricing
- **Linode** - Good performance
- **Hetzner** - Best value in Europe
- **AWS/Azure** - Enterprise features

### How do I update reconFTW?

```bash
cd reconftw
git pull
./install.sh
```

### Tools aren't installing, what do I do?

1. Ensure Go is properly installed: `go version`
2. Check PATH includes Go bin: `echo $PATH | grep go`
3. Run installer again: `./install.sh`
4. Check specific tool: `which httpx`

---

## Usage Questions

### What's the difference between `-r` and `-a`?

| Flag | Name | Includes Vulnerability Scanning |
|------|------|--------------------------------|
| `-r` | Recon | ❌ No |
| `-a` | All | ✅ Yes |

Use `-r` for reconnaissance only, `-a` for full assessment including vulnerability checks.

### How do I scan multiple domains?

Create a file with one domain per line:
```bash
# targets.txt
example.com
test.com
target.org
```

Then run:
```bash
./reconftw.sh -l targets.txt -r
```

### Can I resume an interrupted scan?

Yes! Simply run the same command again. reconFTW uses checkpoints to skip completed functions.

```bash
# Scan interrupted
./reconftw.sh -d example.com -a

# Resume - just run again
./reconftw.sh -d example.com -a
# Output: [sub_passive] Already run, skipping...
```

### How do I force a full rescan?

Delete the checkpoint directory:
```bash
rm -rf Recon/example.com/.called_fn/
./reconftw.sh -d example.com -a
```

### How do I run only specific functions?

Use the `-c` flag:
```bash
./reconftw.sh -d example.com -c nuclei_check
./reconftw.sh -d example.com -c sub_passive
```
`-c` accepts a single function per run. To execute multiple functions, run separate commands.

### What does DEEP mode do?

DEEP mode runs additional, more intensive checks when the number of assets is below a threshold. It's controlled by `DEEP_LIMIT` in the config. See [Performance Tuning](tuning.md#understanding-deep-mode) for details.

---

## Configuration Questions

### Where is the config file?

The main configuration file is `reconftw.cfg` in the reconftw directory. See [Configuration Reference](04-configuration/configuration.md) for all options.

### Where do I put API keys?

Create a `secrets.cfg` file (it's gitignored for security):
```bash
# secrets.cfg
SHODAN_API_KEY="your_key"
GITHUB_TOKEN="your_token"
```

### How do I change thread counts?

Edit `reconftw.cfg`:
```bash
HTTPX_THREADS=50
FFUF_THREADS=40
KATANA_THREADS=20
DALFOX_THREADS=200
RESOLVE_DOMAINS_THREADS=150
TLSX_THREADS=1000
```
For Nuclei speed, use `NUCLEI_RATELIMIT`.

### How do I use a custom wordlist?

```bash
# In reconftw.cfg
subs_wordlist="/path/to/your/wordlist.txt"
subs_wordlist_big="/path/to/your/big_wordlist.txt"
fuzz_wordlist="/path/to/your/dirs.txt"
```

### How do I exclude certain subdomains?

Create an out-of-scope file:
```bash
# outofscope.txt
blog.example.com
legacy.example.com
```

Then run:
```bash
./reconftw.sh -d example.com -r -x outofscope.txt
```

---

## Output Questions

### Where are results saved?

Results are saved in the `Recon/` directory. See [Data Model & I/O](data-model.md) for the complete output structure:
```
Recon/
└── example.com/
    ├── subdomains/
    ├── webs/
    ├── hosts/
    ├── osint/
    └── vulns/
```

### What format are vulnerability results in?

Nuclei results are saved in JSON format in `vulns/nuclei_output/`:
- `nuclei_critical.json`
- `nuclei_high.json`
- `nuclei_medium.json`
- etc.

### How do I generate a report?

Enable AI reports:
```bash
./reconftw.sh -d example.com -a -y
```

Or manually aggregate results from the output files.

### Can I export to CSV?

Results can be converted:
```bash
cat vulns/nuclei_output/*.json | jq -r '[.host, .["template-id"], .info.severity] | @csv'
```

---

## Performance Questions

### How long does a full scan take?

Depends on target size:
- Small target (< 100 subdomains): 30-60 minutes
- Medium target (100-1000 subdomains): 2-4 hours
- Large target (1000+ subdomains): 4-12+ hours

### How do I speed up scans?

1. **Use Axiom** for distributed scanning (see [Axiom Integration](08-integrations/axiom.md))
2. **Increase threads** in config (see [Performance Tuning](tuning.md))
3. **Use passive mode** (`-p`) for quick results
4. **Skip modules** you don't need

### Why is my scan slow?

Common causes:
- Large number of subdomains (normal)
- Rate limiting by target
- Slow DNS resolvers
- Limited system resources

Solutions:
- Use validated resolvers
- Reduce rate limits to avoid blocks
- Increase VPS resources
- Enable DEEP mode limits

### How much disk space do I need?

- Small scans: 1-5 GB
- Medium scans: 5-20 GB
- Large scans: 20-100+ GB

Monitor with: `du -sh Recon/example.com/`

---

## Axiom Questions

### What is Axiom?

Axiom is a tool that lets you distribute reconFTW across multiple cloud instances for faster scanning. See [Axiom Integration](08-integrations/axiom.md) for full details.

### Do I need Axiom?

No, Axiom is optional. It's useful for:
- Very large targets
- Time-sensitive engagements
- Regular/automated scanning

### How do I enable Axiom?

1. Install Axiom: `bash <(curl -s https://raw.githubusercontent.com/pry0cc/axiom/master/interact/axiom-configure)`
2. Configure cloud provider
3. Run reconFTW with `-v` flag: `./reconftw.sh -d example.com -a -v`

### How much does Axiom cost?

Axiom itself is free. You pay for cloud instances:
- ~$0.07/hour for a 10-instance fleet
- A typical scan costs $0.15-0.50

---

## Legal Questions

### Is it legal to use reconFTW?

reconFTW is a legal tool. However, using it against targets without authorization is illegal. Always:
- Get written permission before testing
- Stay within defined scope
- Follow responsible disclosure

### Can I use it for bug bounties?

Yes! reconFTW is designed for bug bounty hunting. See [OPSEC & Legal](opsec-legal.md) and [Case Studies](case-studies.md#case-study-1-bug-bounty---new-program-launch) for detailed guidance. Always:
- Read the program's rules carefully
- Respect rate limits
- Stay in scope
- Report responsibly

### What about rate limiting and being blocked?

To avoid issues (see [OPSEC & Legal](opsec-legal.md#staying-under-the-radar)):
- Use reasonable rate limits
- Respect robots.txt (optional, configurable)
- Don't scan during business hours if concerned
- Use the `-p` (passive) flag for initial recon

---

## Troubleshooting

### Where can I get help?

- **GitHub Issues:** https://github.com/six2dez/reconftw/issues
- **Discord:** https://discord.gg/R5DdXVEdTy
- **Telegram:** https://t.me/joinchat/H5bAaw3YbzzmI5co

### How do I report a bug?

Open a GitHub issue with:
1. reconFTW version: `git rev-parse --abbrev-ref HEAD` and `git describe --tags 2>/dev/null || git rev-parse --short HEAD`
2. Operating system
3. Full error message
4. Steps to reproduce

### Where are the logs?

Logs are in the scan directory:
```bash
cat Recon/example.com/.log/reconftw.log
cat Recon/example.com/.log/errors.log
```

---

## More Questions?

If your question isn't answered here:

1. Check the [Troubleshooting Guide](11-troubleshooting/troubleshooting.md)
2. Search [GitHub Issues](https://github.com/six2dez/reconftw/issues)
3. Ask on [Discord](https://discord.gg/R5DdXVEdTy)
4. Open a new GitHub issue
