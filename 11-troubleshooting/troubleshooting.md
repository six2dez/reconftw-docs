# Troubleshooting Guide

This guide covers common issues, their causes, and solutions when running reconFTW.

---

## Installation Issues

### Go Not Found

**Symptom:**
```
go: command not found
```

**Solution:**
```bash
# Install Go
wget https://go.dev/dl/go1.21.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz

# Add to PATH
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc

# Verify
go version
```

---

### Permission Denied

**Symptom:**
```
bash: ./reconftw.sh: Permission denied
```

**Solution:**
```bash
chmod +x reconftw.sh
chmod +x install.sh
```

---

### macOS Homebrew Issues

**Symptom:**
```
sed: illegal option -- r
```

**Cause:** macOS uses BSD sed, not GNU sed.

**Solution:**
```bash
# Install GNU tools
brew install coreutils gnu-sed findutils grep

# Add to PATH (add to ~/.zshrc or ~/.bashrc)
export PATH="/opt/homebrew/opt/coreutils/libexec/gnubin:$PATH"
export PATH="/opt/homebrew/opt/gnu-sed/libexec/gnubin:$PATH"
export PATH="/opt/homebrew/opt/findutils/libexec/gnubin:$PATH"
export PATH="/opt/homebrew/opt/grep/libexec/gnubin:$PATH"
```

---

### Python Dependency Conflicts

**Symptom:**
```
ERROR: pip's dependency resolver does not support...
```

**Solution:**
```bash
# Create virtual environment
python3 -m venv ~/reconftw-venv
source ~/reconftw-venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

---

### Tool Installation Failures

**Symptom:**
```
go install: module not found
```

**Solution:**
```bash
# Check Go proxy settings
go env GOPROXY

# Reset to default
go env -w GOPROXY=https://proxy.golang.org,direct

# Retry installation
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
```

---

## Runtime Errors

### No Domain Provided

**Symptom:**
```
Error: No domain provided
```

**Solution:**
```bash
# Provide domain with -d flag
./reconftw.sh -d example.com -r

# Or use target list with -l flag
./reconftw.sh -l targets.txt -r
```

---

### Tool Not Found

**Symptom:**
```
subfinder: command not found
```

**Solution:**
```bash
# Check if tool exists
which subfinder

# If missing, install manually
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest

# Verify Go bin in PATH
echo $PATH | grep -q "$HOME/go/bin" || export PATH=$PATH:$HOME/go/bin

# Run tool check
./reconftw.sh --check-tools
```

---

### Memory Issues (OOM)

**Symptom:**
```
Killed
```
or
```
Out of memory
```

**Solution:**
```bash
# Reduce thread counts in reconftw.cfg
HTTPX_THREADS=20
KATANA_THREADS=10
FFUF_THREADS=10
DALFOX_THREADS=50

# Add swap space
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

---

### Disk Space Issues

**Symptom:**
```
No space left on device
```

**Solution:**
```bash
# Check disk usage
df -h

# Find large files
du -sh Recon/*

# Clean old scans
rm -rf Recon/old-target/

# Clean temp files
rm -rf Recon/*/.tmp/*
```

---

### Config File Not Found

**Symptom:**
```
Config file not found
```

**Solution:**
```bash
# Check config exists
ls -la reconftw.cfg

# Create from example
cp reconftw.cfg.example reconftw.cfg

# Or specify path
./reconftw.sh -d example.com -r -f /path/to/reconftw.cfg
```

---

## Performance Issues

### Slow Scans

**Causes and Solutions:**

1. **Too many subdomains:**
   ```bash
   # Use DEEP mode limits
   DEEP_LIMIT=500
   DEEP_LIMIT2=1500
   ```

2. **DNS resolution slow:**
   ```bash
   # Use validated resolvers
   dnsvalidator -tL resolvers.txt -threads 100 -o valid_resolvers.txt
   ```

3. **Rate limiting by target:**
   ```bash
   # Reduce rate limits
   HTTPX_RATELIMIT=50
   NUCLEI_RATELIMIT=50
   ```

4. **Network latency:**
   ```bash
   # Use VPS closer to target
   # Enable connection reuse
   ```

---

### High CPU Usage

**Solution:**
```bash
# Reduce parallel threads
HTTPX_THREADS=20
KATANA_THREADS=10
RESOLVE_DOMAINS_THREADS=80

# Use nice to lower priority
nice -n 10 ./reconftw.sh -d example.com -r
```

---

### Rate Limiting (429 Errors)

**Symptom:**
```
Too Many Requests
429
```

**Solution:**
```bash
# Reduce rate limits
HTTPX_RATELIMIT=50
NUCLEI_RATELIMIT=50

# Enable adaptive rate limiting
./reconftw.sh -d example.com -r --adaptive-rate

# Use global rate limit
./reconftw.sh -d example.com -r -q 50
```

---

## Axiom Issues

### Fleet Won't Start

**Symptom:**
```
Error: Failed to launch fleet
```

**Solutions:**

1. **Check cloud credentials:**
   ```bash
   axiom-configure
   ```

2. **Check API limits:**
   ```bash
   # DigitalOcean
   doctl account get
   
   # Check droplet limits
   doctl account ratelimit
   ```

3. **Check available images:**
   ```bash
   axiom-images ls
   ```

4. **Rebuild image:**
   ```bash
   axiom-build default
   ```

---

### SSH Connection Issues

**Symptom:**
```
ssh: connect to host ... port 22: Connection refused
```

**Solutions:**

1. **Regenerate SSH keys:**
   ```bash
   axiom-init --regenerate
   ```

2. **Check security groups:**
   ```bash
   # Ensure port 22 is open
   # Check cloud provider firewall
   ```

3. **Wait for instance:**
   ```bash
   # Instances may take 1-2 minutes to be ready
   sleep 120
   axiom-ls
   ```

---

### Resolver Problems on Fleet

**Symptom:**
```
DNS resolution failing on fleet
```

**Solution:**
```bash
# Upload resolvers to fleet
axiom-scp resolvers.txt "reconftw*":/home/op/lists/

# Update resolver config
axiom-exec "cat /home/op/lists/resolvers.txt | head -5"
```

---

## Output Issues

### Empty Results

**Possible Causes:**

1. **Scope too restrictive:**
   ```bash
   # Check scope files
   cat inscope.txt
   cat outofscope.txt
   ```

2. **Target doesn't exist:**
   ```bash
   # Verify domain
   dig example.com
   ```

3. **All tools failed:**
   ```bash
   # Check logs
   cat Recon/example.com/.log/*.txt | grep -i error
   ```

4. **API keys missing:**
   ```bash
   # Verify secrets.cfg
   cat secrets.cfg
   ```

---

### Missing Files

**Symptom:**
Files expected but not present in output.

**Solutions:**

1. **Check if function ran:**
   ```bash
   ls Recon/example.com/.called_fn/
   ```

2. **Check logs for errors:**
   ```bash
   grep -i "error\|fail" Recon/example.com/.log/*.txt
   ```

3. **Rerun specific function:**
   ```bash
   rm Recon/example.com/.called_fn/.nuclei_check
   ./reconftw.sh -d example.com -c nuclei_check
   ```

---

### Parallel Output Too Noisy

**Symptom:**
Terminal output is hard to follow during parallel execution.

**Solutions:**

1. **Use clean parallel UI (recommended):**
   ```bash
   # reconftw.cfg
   PARALLEL_UI_MODE="clean"
   ```

2. **Force sequential mode for debugging:**
   ```bash
   ./reconftw.sh -d example.com -r --no-parallel
   ```

3. **Use trace mode only when investigating orchestration issues:**
   ```bash
   # reconftw.cfg
   PARALLEL_UI_MODE="trace"
   ```

4. **Reduce heartbeat refresh frequency if live progress updates are too frequent:**
   ```bash
   PARALLEL_HEARTBEAT_SECONDS=30
   ```

---

### Corrupted Output

**Symptom:**
```
unexpected EOF
parse error
```

**Solution:**
```bash
# Clear temp files
rm -rf Recon/example.com/.tmp/*

# Clear checkpoints
rm -rf Recon/example.com/.called_fn/*

# Rescan
./reconftw.sh -d example.com -r
```

---

### Report Files Not Generated

**Symptom:**
`Recon/<target>/report/` exists but files are missing or empty.

**Solution:**
```bash
# Rebuild reports from existing results
./reconftw.sh -d example.com --report-only --export all

# Verify source artifacts
ls -la Recon/example.com/{subdomains,webs,hosts,nuclei_output}
```

If `jq` is missing, install it; JSON/CSV rendering quality degrades without it.

---

### Monitor Mode Not Progressing

**Symptom:**
Monitor exits early or does not rotate cycles.

**Solution:**
```bash
# Validate monitor inputs
./reconftw.sh -d example.com -r --monitor --monitor-interval 30 --monitor-cycles 2

# For web mode with lists
./reconftw.sh -d example.com -l urls.txt -w --monitor --monitor-interval 30
```

Checks:
- `-m` is not supported in monitor mode
- `-l` is only allowed with `-w`
- `MONITOR_INTERVAL_MIN` must be integer >= 1

---

## API Key Issues

### Invalid API Key

**Symptom:**
```
401 Unauthorized
API key invalid
```

**Solution:**
```bash
# Verify key in secrets.cfg
grep SHODAN secrets.cfg

# Test key directly
curl "https://api.shodan.io/api-info?key=YOUR_KEY"
```

---

### Rate Limited by API

**Symptom:**
```
Rate limit exceeded
Too many requests
```

**Solution:**
```bash
# Wait and retry
sleep 3600  # Wait 1 hour

# Use multiple API keys (if allowed)
# Reduce parallel requests
```

---

### Missing API Keys

**Symptom:**
```
Warning: SHODAN_API_KEY not set
```

**Solution:**
```bash
# Edit secrets.cfg
nano secrets.cfg

# Add keys
SHODAN_API_KEY="your_key_here"
GITHUB_TOKEN="your_token_here"
```

---

## Common Fixes

### Clear Cache

```bash
# Clear all cache
rm -rf Recon/example.com/.tmp/*
rm -rf Recon/example.com/.cache/*
```

### Update Resolvers

```bash
# Download fresh resolvers
wget https://raw.githubusercontent.com/trickest/resolvers/main/resolvers.txt

# Validate
dnsvalidator -tL resolvers.txt -threads 100 -o valid_resolvers.txt

# Update config
cp valid_resolvers.txt ~/.config/reconftw/resolvers.txt
```

### Reinstall Tools

```bash
# Reinstall all tools
./install.sh

# Or update specific tool
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
```

### Reset Checkpoints

```bash
# Reset all (full rescan)
rm -rf Recon/example.com/.called_fn/

# Reset specific function
rm Recon/example.com/.called_fn/.function_name
```

### Check Logs

```bash
# Latest run log
ls -1t Recon/example.com/.log/*.txt | head -n1 | xargs -I {} tail -n 200 {}

# Live monitoring
tail -f "$(ls -1t Recon/example.com/.log/*.txt | head -n1)"
```

---

## Diagnostic Commands

### System Check

```bash
# Check all tools
./reconftw.sh --check-tools

# System health
./reconftw.sh --health-check
```

### Resource Check

```bash
# Memory
free -h

# Disk
df -h

# CPU
nproc
cat /proc/cpuinfo | grep "model name" | head -1
```

### Network Check

```bash
# DNS resolution
dig example.com

# HTTP connectivity
curl -I https://example.com

# Check resolvers
cat resolvers.txt | head -5 | xargs -I {} dig @{} example.com
```

---

## Getting Help

### Community Resources

- **GitHub Issues:** https://github.com/six2dez/reconftw/issues
- **Discord:** [Link to Discord]
- **Twitter:** @six2dez

### Reporting Bugs

Include:
1. commit hash and branch: `git rev-parse --abbrev-ref HEAD && git rev-parse --short HEAD`
2. OS version: `cat /etc/os-release`
3. Error message (full)
4. Steps to reproduce
5. Relevant log sections

### Feature Requests

Use GitHub Issues with `[FEATURE]` prefix.

---

## Error Code Reference

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid argument |
| 3 | Tool not found |
| 4 | Configuration error |
| 5 | Network error |
| 6 | Permission denied |
| 7 | Disk full |
| 8 | Memory exhausted |

---

## Quick Reference

| Issue | Quick Fix |
|-------|-----------|
| Tool not found | `./install.sh` |
| Empty results | Check API keys in secrets.cfg |
| Slow scan | Reduce THREADS in config |
| 429 errors | Reduce RATELIMIT values |
| OOM | Add swap, reduce threads |
| Resume scan | Just run same command again |
| Full rescan | Delete `.called_fn/` directory |

---

## Next Steps

- **[Getting Started](../01-getting-started/getting-started.md)** - Fresh installation
- **[Configuration](../04-configuration/configuration.md)** - Optimize settings
- **[Advanced Usage](../10-advanced/advanced.md)** - Power user features
