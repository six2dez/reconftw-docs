# Axiom Integration

> **Note**: Axiom was rebranded to "Ax Framework" but the tool and commands still use `axiom-*` naming. This documentation uses "Axiom" for consistency with the actual commands.

Axiom enables distributed scanning across cloud infrastructure, speeding up reconnaissance by parallelizing workloads across multiple instances.

---

## What is Axiom?

Axiom is an infrastructure automation framework that allows you to:

- **Spin up cloud instances on-demand** across multiple providers
- **Distribute scanning tasks** across a fleet of machines
- **Scale horizontally** for large-scale reconnaissance
- **Reduce scan time** from hours to minutes

<!-- IMAGE PLACEHOLDER
**Image: axiom-architecture.png**
Description: Diagram showing Axiom architecture with local machine controlling a fleet of cloud instances, distributing tasks, and collecting results.
- Show: Local machine → Axiom Fleet (multiple cloud instances) → Target
- Include: Provider logos (DigitalOcean, AWS, Azure, Linode)
- Arrows showing command distribution and result aggregation
-->

---

## Prerequisites

### 1. Axiom Installation

```bash
# Install axiom
bash <(curl -s https://raw.githubusercontent.com/pry0cc/axiom/master/interact/axiom-configure)

# Or clone and install
git clone https://github.com/pry0cc/axiom ~/.axiom
cd ~/.axiom && ./interact/axiom-configure
```

### 2. Cloud Provider Setup

Supported providers:
- DigitalOcean (recommended)
- AWS
- Azure
- Linode
- Google Cloud
- Hetzner
- IBM Cloud

### 3. API Keys

Configure cloud provider credentials:
```bash
# Run axiom configuration
axiom-configure
```

---

## Configuration in reconFTW

### Enable Axiom Mode

```bash
# In reconftw.cfg

# Enable Axiom/VPS mode
AXIOM=true

# Fleet configuration
AXIOM_FLEET_NAME="reconftw"        # Fleet identifier
AXIOM_FLEET_COUNT=10               # Number of instances
AXIOM_FLEET_LAUNCH=true            # Auto-launch fleet
AXIOM_FLEET_SHUTDOWN=true          # Auto-shutdown after scan

# Distributed tool settings
AXIOM_THREADS=20                   # Threads per instance
```

### Instance Configuration

```bash
# Fleet instance settings
AXIOM_INSTANCE_TYPE="s-1vcpu-1gb"  # DigitalOcean size
AXIOM_REGION="nyc1"                # Deployment region
AXIOM_IMAGE="axiom-default"        # Base image name
```

### Resolver Configuration

```bash
# Resolvers for distributed scanning
AXIOM_RESOLVERS_PATH="/home/op/lists/resolvers.txt"
```

---

## Running with Axiom

### Basic Usage

```bash
# Run with Axiom flag
./reconftw.sh -d example.com -a --vps

# Or use -v shorthand
./reconftw.sh -d example.com -a -v
```

### What Happens

1. **Fleet Launch:** Axiom spins up configured number of instances
2. **Tool Distribution:** Tools run across fleet in parallel
3. **Result Collection:** Results merged from all instances
4. **Fleet Shutdown:** Instances terminated (if configured)

---

## Distributed Scanning Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Axiom Distributed Scanning                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐                                                    │
│  │ Local Machine│                                                    │
│  │  (reconFTW)  │                                                    │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         │ axiom-fleet launch                                         │
│         ▼                                                            │
│  ┌──────────────────────────────────────────────────────┐           │
│  │              Axiom Fleet (10 instances)              │           │
│  ├──────────────────────────────────────────────────────┤           │
│  │  ┌────┐  ┌────┐  ┌────┐  ┌────┐  ┌────┐            │           │
│  │  │ 01 │  │ 02 │  │ 03 │  │ 04 │  │ 05 │            │           │
│  │  └────┘  └────┘  └────┘  └────┘  └────┘            │           │
│  │  ┌────┐  ┌────┐  ┌────┐  ┌────┐  ┌────┐            │           │
│  │  │ 06 │  │ 07 │  │ 08 │  │ 09 │  │ 10 │            │           │
│  │  └────┘  └────┘  └────┘  └────┘  └────┘            │           │
│  └──────────────────────────────────────────────────────┘           │
│         │                                                            │
│         │ axiom-scan (distribute tasks)                              │
│         ▼                                                            │
│  ┌──────────────────────────────────────────────────────┐           │
│  │ Each instance processes portion of targets:          │           │
│  │ - Instance 01: subdomains 1-1000                     │           │
│  │ - Instance 02: subdomains 1001-2000                  │           │
│  │ - ...                                                 │           │
│  └──────────────────────────────────────────────────────┘           │
│         │                                                            │
│         │ Results merged                                             │
│         ▼                                                            │
│  ┌──────────────┐                                                    │
│  │ Local Machine│                                                    │
│  │ (combined    │                                                    │
│  │  results)    │                                                    │
│  └──────────────┘                                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Tools with Axiom Support

reconFTW distributes these tools across the fleet:

| Tool | Distribution Type | Notes |
|------|-------------------|-------|
| subfinder | Target split | Each instance handles subset |
| httpx | Target split | URL probing distributed |
| nuclei | Target split | Vulnerability scanning |
| ffuf | Target split | Directory fuzzing |
| dnsx | Target split | DNS resolution |
| nmap | Target split | Port scanning |
| katana | Target split | Web crawling |
| dalfox | Target split | XSS testing |

---

## Fleet Management

### Manual Fleet Control

```bash
# Launch fleet
axiom-fleet reconftw -i 10

# Check fleet status
axiom-ls

# SSH to specific instance
axiom-ssh reconftw01

# Execute command on all instances
axiom-exec "nuclei -update-templates"

# Shutdown fleet
axiom-rm "reconftw*" -f
```

### Fleet Lifecycle

```bash
# In reconftw.cfg

# Auto-launch fleet if not running
AXIOM_FLEET_LAUNCH=true

# Shutdown after scan completion
AXIOM_FLEET_SHUTDOWN=true

# Keep fleet running for subsequent scans
AXIOM_FLEET_SHUTDOWN=false
```

---

## Cost Optimization

### Instance Sizing

| Provider | Instance | vCPU | RAM | Cost/hr |
|----------|----------|------|-----|---------|
| DigitalOcean | s-1vcpu-1gb | 1 | 1GB | $0.007 |
| DigitalOcean | s-2vcpu-2gb | 2 | 2GB | $0.018 |
| AWS | t3.micro | 2 | 1GB | $0.012 |
| Linode | g6-nanode-1 | 1 | 1GB | $0.0075 |

### Cost Estimation

```
Fleet: 10 instances × $0.007/hr = $0.07/hr
Scan duration: 2 hours
Total cost: ~$0.14 per scan
```

### Best Practices

1. **Use smallest viable instance:** 1GB RAM sufficient for most tools
2. **Auto-shutdown:** Enable `AXIOM_FLEET_SHUTDOWN=true`
3. **Spot instances:** Use when available for 60-90% savings
4. **Regional pricing:** Some regions cheaper than others

---

## Resolver Management

### Upload Custom Resolvers

```bash
# Upload resolvers to fleet
axiom-scp resolvers.txt "reconftw*":/home/op/lists/

# Or use built-in resolver update
axiom-exec "dnsvalidator -tL public-resolvers.txt -threads 100 -o resolvers.txt"
```

### Configuration

```bash
# In reconftw.cfg
AXIOM_RESOLVERS_PATH="/home/op/lists/resolvers.txt"
AXIOM_RESOLVERS_TRUSTED_PATH="/home/op/lists/resolvers_trusted.txt"
```

---

## Troubleshooting

### Fleet Won't Start

```bash
# Check cloud provider credentials
axiom-configure

# Verify API key
doctl auth init  # DigitalOcean
aws configure    # AWS

# Check available images
axiom-images ls
```

### SSH Connection Issues

```bash
# Regenerate SSH keys
axiom-init --regenerate

# Manual SSH test
ssh -i ~/.axiom/.sshkey root@<instance-ip>
```

### Tools Not Running

```bash
# Update tools on fleet
axiom-exec "axiom-build tools"

# Verify tool installation
axiom-exec "which nuclei"
```

### Result Merge Failures

```bash
# Check disk space
axiom-exec "df -h"

# Manual result collection
axiom-scp "reconftw*":/home/op/recon/results.txt ./results/
```

---

## Advanced Configuration

### Custom Axiom Image

```bash
# Build custom image with all tools
axiom-build reconftw --install-tools

# Use custom image
AXIOM_IMAGE="reconftw-image"
```

### Per-Tool Distribution

```bash
# Override distribution for specific tools
AXIOM_SUBFINDER_DISTRIBUTE=true
AXIOM_NUCLEI_DISTRIBUTE=true
AXIOM_HTTPX_DISTRIBUTE=true
```

### Scan Modules

```bash
# reconFTW Axiom modules in modules/axiom.sh
# Functions prefixed with axiom_*

axiom_launch          # Launch fleet
axiom_shutdown        # Terminate fleet  
axiom_selected       # Run tool on fleet
axiom_exec           # Execute command
```

---

## Example Workflows

### Large Target List

```bash
# Scan 10,000 subdomains with 20-instance fleet
AXIOM_FLEET_COUNT=20
./reconftw.sh -l targets_10k.txt -a -v
```

### Bug Bounty Program

```bash
# Weekly automated scan
./reconftw.sh -d target.com -a -v --incremental
```

### Red Team Assessment

```bash
# Quick initial recon
AXIOM_FLEET_COUNT=5
./reconftw.sh -d target.com -r -v
```

---

## Security Considerations

1. **Instance isolation:** Each scan uses fresh instances
2. **Credential management:** Cloud keys never touch targets
3. **Data cleanup:** Results removed from instances
4. **Network isolation:** Use private networking when possible
5. **Audit logging:** Enable cloud provider audit logs

---

## Next Steps

- **[Faraday Integration](./faraday.md)** - Vulnerability management
- **[Deployment Guide](../09-deployment/deployment.md)** - VPS setup
