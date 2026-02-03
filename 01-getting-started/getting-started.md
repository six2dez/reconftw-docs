# Getting Started

This guide will walk you through installing reconFTW, setting it up, and running your first reconnaissance scan.

---

## System Requirements

### Minimum Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| **OS** | Ubuntu 20.04+ / Debian 11+ / Kali | Ubuntu 22.04 LTS |
| **RAM** | 4 GB | 8-16 GB |
| **Disk Space** | 20 GB | 50+ GB |
| **CPU** | 2 cores | 4+ cores |
| **Network** | Stable internet | High bandwidth |

### Supported Operating Systems

- ✅ **Ubuntu** 20.04, 22.04, 24.04 (Recommended)
- ✅ **Debian** 11, 12
- ✅ **Kali Linux** (latest rolling)
- ✅ **Parrot OS**
- ✅ **macOS** 12+ (with Homebrew)
- ✅ **Docker** (any platform)
- ⚠️ **Windows WSL2** (experimental)

### Required Dependencies

reconFTW will install most dependencies automatically, but these base packages are required:

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y git curl wget python3 python3-pip ruby golang jq

# macOS (via Homebrew)
brew install git curl wget python@3 ruby go jq gnu-getopt coreutils gnu-sed bash
```

---

## Installation Methods

### Method 1: Local Installation (Recommended)

This is the standard installation for Linux/macOS systems.

```bash
# Clone the repository
git clone https://github.com/six2dez/reconftw.git
cd reconftw

# Run the installer (interactive)
./install.sh

# Or run non-interactively (option 1 = install all)
echo 1 | ./install.sh
```

The installer will:
1. Check system requirements
2. Install Go, Rust, and Python dependencies
3. Install 80+ security tools
4. Configure PATH and environment variables
5. Download required wordlists and resolvers

> **⏱️ Installation Time**: 15-45 minutes depending on internet speed and system resources.

#### Installation Options

When running `./install.sh`, you'll see these options:

```
Select an option:
1. Install all dependencies and tools
2. Install only tools (skip dependencies)
3. Update tools only
4. Check tool installation status
```

### Method 2: Docker Installation

Docker provides a consistent environment across all platforms.

```bash
# Pull the official image
docker pull six2dez/reconftw:latest

# Run a scan
docker run --rm -v $(pwd)/Recon:/reconftw/Recon six2dez/reconftw:latest -d example.com -r

# Interactive mode
docker run -it --rm -v $(pwd)/Recon:/reconftw/Recon six2dez/reconftw:latest /bin/bash
```

#### Docker with Custom Configuration

```bash
# Mount your custom config
docker run --rm \
  -v $(pwd)/Recon:/reconftw/Recon \
  -v $(pwd)/my_config.cfg:/reconftw/reconftw.cfg \
  -e SHODAN_API_KEY="your_key" \
  six2dez/reconftw:latest -d example.com -r
```

#### Building Custom Docker Image

```bash
cd reconftw/Docker
docker build -t my-reconftw .
```

### Method 3: Terraform + Ansible (Cloud Deployment)

For deploying reconFTW on AWS or other cloud providers:

```bash
cd reconftw/Terraform

# Configure your AWS credentials
export AWS_ACCESS_KEY_ID="your_key"
export AWS_SECRET_ACCESS_KEY="your_secret"

# Deploy infrastructure
terraform init
terraform apply

# Run Ansible playbook
ansible-playbook -i inventory reconFTW.yml
```

See [Deployment Guide](../09-deployment/deployment.md) for detailed cloud setup.

---

## macOS-Specific Setup

macOS requires additional Homebrew packages due to BSD tool differences:

```bash
# Install Homebrew if not installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install required formulae
brew install bash coreutils gnu-sed gnu-getopt findutils grep

# Use Homebrew's bash (required for reconFTW)
# reconFTW will automatically use /opt/homebrew/bin/bash on Apple Silicon
```

> **Note**: reconFTW automatically detects macOS and uses Homebrew GNU tools. Make sure they're installed before running.

---

## Your First Scan

### Basic Reconnaissance Scan

```bash
cd reconftw

# Standard reconnaissance
./reconftw.sh -d example.com -r
```

This will:
1. Enumerate subdomains (passive + active)
2. Probe for live web servers
3. Take screenshots
4. Extract URLs and JavaScript files
5. Run nuclei vulnerability scans
6. Generate organized output

### Understanding the Output

Results are saved in `Recon/<domain>/`:

```
Recon/example.com/
├── subdomains/          # Discovered subdomains
│   ├── subdomains.txt   # Final subdomain list
│   └── ...
├── webs/                # Web server information
│   ├── webs.txt         # Live web servers
│   └── ...
├── hosts/               # Host/IP information
├── vulns/               # Vulnerability findings
├── osint/               # OSINT results
├── screenshots/         # Web screenshots
├── nuclei_output/       # Nuclei scan results
├── .log/                # Scan logs
└── .tmp/                # Temporary files
```

### Quick Scan Examples

```bash
# Subdomain enumeration only (fast)
./reconftw.sh -d example.com -s

# Passive reconnaissance (non-intrusive)
./reconftw.sh -d example.com -p

# Full scan with vulnerability checks
./reconftw.sh -d example.com -a

# Scan from a list of domains
./reconftw.sh -l targets.txt -r

# Custom output directory
./reconftw.sh -d example.com -r -o /path/to/output
```

---

## Verifying Installation

### Check Tools Installation

```bash
# Verify all tools are installed
./reconftw.sh --check-tools
```

This displays a checklist of all required tools and their installation status.

### Health Check

```bash
# Run system health check
./reconftw.sh --health-check
```

The health check verifies:
- All critical dependencies are installed
- Required files and directories exist
- Configuration is valid
- Network connectivity works

### Dry Run Mode

Test your command without executing anything:

```bash
# Preview what commands would be executed
./reconftw.sh -d example.com -r --dry-run
```

---

## Updating reconFTW

### Update Everything

```bash
cd reconftw

# Pull latest code
git pull

# Update all tools
./install.sh --tools
```

### Update Only Tools

```bash
./install.sh --tools
# Or use option 3 in interactive mode
```

### Automatic Updates Before Scans

Enable in `reconftw.cfg`:

```bash
upgrade_before_running=true
```

---

## Post-Installation Configuration

After installation, you should configure API keys for maximum effectiveness:

### 1. Create secrets.cfg

```bash
cp secrets.cfg.example secrets.cfg
chmod 600 secrets.cfg
```

Edit `secrets.cfg`:

```bash
# API keys for enhanced functionality
SHODAN_API_KEY="your_shodan_api_key"
WHOISXML_API="your_whoisxml_api_key"
XSS_SERVER="your_xss_hunter_server"
COLLAB_SERVER="your_interactsh_server"
```

### 2. Configure GitHub Tokens

Create `$HOME/Tools/.github_tokens` with one token per line:

```
ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
ghp_yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
```

Multiple tokens help avoid rate limiting during GitHub reconnaissance.

### 3. Verify Configuration

```bash
# Test with a simple scan
./reconftw.sh -d example.com -p --dry-run
```

---

## Common Installation Issues

### "Go not found"

```bash
# Install Go manually
wget https://go.dev/dl/go1.21.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```

### "Permission denied"

```bash
# Fix permissions
chmod +x reconftw.sh install.sh
```

### "Tool X not found"

```bash
# Reinstall specific tools
./install.sh --tools
```

See [Troubleshooting](../11-troubleshooting/troubleshooting.md) for more solutions.

---

## Next Steps

Now that reconFTW is installed:

1. **[Learn the concepts](../02-concepts/concepts.md)** - Understand how reconFTW works
2. **[Explore usage options](../03-usage/usage.md)** - Master command-line flags
3. **[Configure settings](../04-configuration/configuration.md)** - Customize for your needs
4. **[Understand output](../07-output/output.md)** - Interpret your results

---

## Quick Reference Card

```bash
# Installation
git clone https://github.com/six2dez/reconftw.git && cd reconftw && ./install.sh

# Basic scans
./reconftw.sh -d target.com -r          # Full recon
./reconftw.sh -d target.com -s          # Subdomains only
./reconftw.sh -d target.com -p          # Passive only
./reconftw.sh -d target.com -a          # Full + vulns

# Advanced options
./reconftw.sh -d target.com -r --deep   # Deep/thorough scan
./reconftw.sh -d target.com -r -v       # With Axiom (distributed)
./reconftw.sh -l targets.txt -r         # Multiple targets

# Maintenance
./reconftw.sh --check-tools             # Verify installation
./reconftw.sh --health-check            # System health
./install.sh --tools                    # Update tools
```
