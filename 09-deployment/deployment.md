# Deployment Guide

This guide covers all deployment options for reconFTW: local installation, Docker, cloud platforms, and infrastructure as code.

---

## Deployment Options Overview

| Method | Best For | Complexity |
|--------|----------|------------|
| Local Install | Development, single machine | Low |
| Docker | Portability, isolation | Low |
| VPS | Long-running scans | Medium |
| Terraform/Ansible | Reproducible infrastructure | High |
| Proxmox | Home lab, dedicated hardware | Medium |

---

## Local Installation

### System Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 2 cores | 4+ cores |
| RAM | 4 GB | 8+ GB |
| Disk | 20 GB | 50+ GB |
| OS | Ubuntu 20.04+ | Ubuntu 22.04 |

### Quick Install

```bash
# Clone repository
git clone https://github.com/six2dez/reconftw.git
cd reconftw

# Run installer
./install.sh

# Verify installation
./reconftw.sh --check-tools
```

### Manual Installation

```bash
# Install system dependencies
sudo apt update
sudo apt install -y git curl wget python3 python3-pip ruby golang jq

# Set up Go environment
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bashrc

# Clone and install
git clone https://github.com/six2dez/reconftw.git
cd reconftw
./install.sh
```

### macOS Installation

```bash
# Install Homebrew dependencies
brew install git curl wget python3 go ruby jq coreutils gnu-sed

# Use GNU versions
export PATH="/opt/homebrew/opt/coreutils/libexec/gnubin:$PATH"
export PATH="/opt/homebrew/opt/gnu-sed/libexec/gnubin:$PATH"

# Clone and install
git clone https://github.com/six2dez/reconftw.git
cd reconftw
./install.sh
```

---

## Docker Deployment

### Using Official Image

```bash
# Pull latest image
docker pull six2dez/reconftw:main

# Run scan
docker run -it --rm \
  -v $(pwd)/Recon:/reconftw/Recon \
  -v $(pwd)/reconftw.cfg:/reconftw/reconftw.cfg \
  -v $(pwd)/secrets.cfg:/reconftw/secrets.cfg \
  six2dez/reconftw:main \
  -d example.com -r
```

### Docker Compose

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  reconftw:
    image: six2dez/reconftw:main
    container_name: reconftw
    volumes:
      - ./Recon:/reconftw/Recon
      - ./reconftw.cfg:/reconftw/reconftw.cfg
      - ./secrets.cfg:/reconftw/secrets.cfg
      - ./wordlists:/reconftw/wordlists
    environment:
      - TERM=xterm-256color
    stdin_open: true
    tty: true
    command: ["-d", "example.com", "-r"]
```

Run with:
```bash
docker-compose up
```

### Building Custom Image

```dockerfile
# Dockerfile.custom
FROM six2dez/reconftw:main

# Add custom tools
RUN go install -v github.com/custom/tool@latest

# Add custom wordlists
COPY ./wordlists /reconftw/wordlists

# Add custom config
COPY ./reconftw.cfg /reconftw/reconftw.cfg
```

Build and run:
```bash
docker build -t reconftw-custom -f Dockerfile.custom .
docker run -it --rm -v $(pwd)/Recon:/reconftw/Recon reconftw-custom -d example.com -r
```

### Volume Mounts

| Mount | Purpose |
|-------|---------|
| `/reconftw/Recon` | Output directory |
| `/reconftw/reconftw.cfg` | Configuration |
| `/reconftw/secrets.cfg` | API keys |
| `/reconftw/wordlists` | Custom wordlists |

### Resource Limits

```yaml
# docker-compose.yml with limits
services:
  reconftw:
    image: six2dez/reconftw:main
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 8G
        reservations:
          cpus: '2'
          memory: 4G
```

---

## VPS Deployment

### Recommended Providers

| Provider | Instance | Specs | Cost/mo |
|----------|----------|-------|---------|
| DigitalOcean | Basic Droplet | 2vCPU/4GB | $24 |
| Linode | Linode 4GB | 2vCPU/4GB | $24 |
| Vultr | Cloud Compute | 2vCPU/4GB | $24 |
| Hetzner | CX21 | 2vCPU/4GB | €5.83 |
| AWS | t3.medium | 2vCPU/4GB | ~$30 |

### Initial VPS Setup

```bash
# Connect to VPS
ssh root@your-vps-ip

# Update system
apt update && apt upgrade -y

# Create non-root user
adduser recon
usermod -aG sudo recon

# Switch to user
su - recon

# Install reconFTW
git clone https://github.com/six2dez/reconftw.git
cd reconftw
./install.sh
```

### Security Hardening

```bash
# Disable root login
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

# Setup firewall
sudo ufw allow ssh
sudo ufw enable

# Install fail2ban
sudo apt install fail2ban
sudo systemctl enable fail2ban
```

### Running Long Scans

```bash
# Use tmux for persistent sessions
sudo apt install tmux
tmux new -s recon

# Start scan inside tmux
./reconftw.sh -d example.com -a

# Detach: Ctrl+B, then D
# Reattach: tmux attach -t recon
```

### Screen Alternative

```bash
# Use screen
screen -S recon
./reconftw.sh -d example.com -a

# Detach: Ctrl+A, then D
# Reattach: screen -r recon
```

---

## Terraform Deployment

> Note (best-effort): This is a reference setup. AMIs, SSH usernames, and cloud defaults change over time, so you may
> need to adjust it for your environment.

The repository includes a reference implementation under `Terraform/` that provisions an EC2 instance and runs an
Ansible playbook to install reconFTW and copy a baseline config.

### AWS (Terraform + Ansible)

**Requirements:** AWS credentials configured (e.g., `AWS_PROFILE`), `terraform`, and `ansible`.

```bash
cd Terraform

# Create a key pair used by Terraform/Ansible
ssh-keygen -f terraform-keys -t ecdsa -b 521

terraform init

# Restrict SSH to your public IP
terraform apply -var 'allowed_ssh_cidr=YOUR_PUBLIC_IP/32'
```

**Optional:** deploy a specific branch/tag:

```bash
terraform apply -var 'allowed_ssh_cidr=YOUR_PUBLIC_IP/32' -var 'reconftw_branch=dev'
```

**What gets deployed:**
- An EC2 instance + security group (SSH restricted by `allowed_ssh_cidr`)
- Ansible provisioning using `Terraform/reconFTW.yml`
- A baseline config copied from `Terraform/files/reconftw.cfg` to `/opt/reconftw/reconftw.cfg`

**Outputs:**
- `public_ip`
- `ssh_command`

---

## Ansible Deployment

If you already have a VM, you can run the included playbook directly (see `Terraform/reconFTW.yml`).

```bash
cd Terraform

# Single host inventory (note the trailing comma)
ansible-playbook -i 'YOUR_INSTANCE_IP,' -u admin --private-key terraform-keys reconFTW.yml

# Optional: deploy a specific branch/tag
ansible-playbook -i 'YOUR_INSTANCE_IP,' -u admin --private-key terraform-keys -e reconftw_branch=dev reconFTW.yml
```

---

## Proxmox Deployment

### Create VM

1. Upload Ubuntu ISO to Proxmox
2. Create VM:
   - CPU: 2+ cores
   - RAM: 4+ GB
   - Disk: 50+ GB
3. Install Ubuntu
4. Install reconFTW

### LXC Container

```bash
# Create container
pct create 100 local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst \
  --hostname reconftw \
  --memory 4096 \
  --cores 2 \
  --rootfs local-lvm:50 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp

# Start container
pct start 100

# Enter container
pct enter 100

# Install reconFTW
apt update && apt install -y git
git clone https://github.com/six2dez/reconftw.git
cd reconftw && ./install.sh
```

---

## CI/CD Integration

### GitHub Actions

#### Basic Weekly Scan

```yaml
# .github/workflows/reconftw.yml
name: reconFTW Scan

on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sunday
  workflow_dispatch:
    inputs:
      target:
        description: 'Target domain'
        required: true

jobs:
  scan:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup reconFTW
        run: |
          chmod +x reconftw.sh install.sh
          ./install.sh
          
      - name: Configure secrets
        run: |
          echo "SHODAN_API_KEY=${{ secrets.SHODAN_API_KEY }}" >> secrets.cfg
          chmod 600 secrets.cfg
          
      - name: Run scan
        run: |
          ./reconftw.sh -d ${{ github.event.inputs.target }} -r
          
      - name: Upload results
        uses: actions/upload-artifact@v3
        with:
          name: reconftw-results
          path: Recon/
```

#### Advanced: Multi-Target with Diff and Notifications

```yaml
# .github/workflows/reconftw-advanced.yml
name: reconFTW Multi-Target Scan

on:
  schedule:
    - cron: '0 2 * * 0'  # Weekly Sunday 2AM
  workflow_dispatch:

env:
  SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}

jobs:
  scan:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        target: [target1.com, target2.com, target3.com]
      max-parallel: 1  # Sequential to avoid rate limits
    
    steps:
      - name: Setup reconFTW
        run: |
          git clone https://github.com/six2dez/reconftw.git
          cd reconftw && ./install.sh
          
      - name: Configure
        run: |
          cd reconftw
          cat << 'EOF' > secrets.cfg
          SHODAN_API_KEY="${{ secrets.SHODAN_API_KEY }}"
          GITHUB_TOKEN="${{ secrets.GH_TOKEN }}"
          EOF
          
      - name: Restore previous results
        uses: actions/cache@v3
        with:
          path: reconftw/Recon/${{ matrix.target }}
          key: reconftw-${{ matrix.target }}-${{ github.run_number }}
          restore-keys: reconftw-${{ matrix.target }}-
          
      - name: Run scan
        run: |
          cd reconftw
          ./reconftw.sh -d ${{ matrix.target }} -r --incremental
          
      - name: Generate diff
        id: diff
        run: |
          cd reconftw/Recon/${{ matrix.target }}
          # Count new findings
          NEW_SUBS=$(wc -l < subdomains/subdomains.txt 2>/dev/null || echo 0)
          VULNS=$(cat nuclei_output/*_json.txt 2>/dev/null | jq -s 'length' || echo 0)
          echo "new_subs=$NEW_SUBS" >> $GITHUB_OUTPUT
          echo "vulns=$VULNS" >> $GITHUB_OUTPUT
          
      - name: Notify Slack
        if: steps.diff.outputs.vulns > 0
        run: |
          curl -X POST -H 'Content-type: application/json' \
            --data '{"text":"🔔 reconFTW: ${{ matrix.target }} - ${{ steps.diff.outputs.new_subs }} subs, ${{ steps.diff.outputs.vulns }} vulns"}' \
            $SLACK_WEBHOOK
            
      - name: Upload results
        uses: actions/upload-artifact@v3
        with:
          name: results-${{ matrix.target }}
          path: reconftw/Recon/${{ matrix.target }}
          retention-days: 30
```

#### Scheduled with Faraday Integration

```yaml
# .github/workflows/reconftw-faraday.yml
name: reconFTW to Faraday

on:
  schedule:
    - cron: '0 3 * * 1'  # Weekly Monday 3AM

jobs:
  scan:
    runs-on: ubuntu-latest
    
    steps:
      - name: Setup
        run: |
          git clone https://github.com/six2dez/reconftw.git
          cd reconftw && ./install.sh
          
      - name: Configure Faraday
        run: |
          cd reconftw
          cat << EOF >> reconftw.cfg
          FARADAY=true
          FARADAY_URL="${{ secrets.FARADAY_URL }}"
          FARADAY_USER="${{ secrets.FARADAY_USER }}"
          FARADAY_PASSWORD="${{ secrets.FARADAY_PASSWORD }}"
          FARADAY_WORKSPACE="${{ github.event.inputs.workspace || 'default' }}"
          EOF
          
      - name: Run scan
        run: |
          cd reconftw
          ./reconftw.sh -l targets.txt -r
```

### GitLab CI

#### Basic Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - scan

reconftw_scan:
  stage: scan
  image: six2dez/reconftw:main
  script:
    - ./reconftw.sh -d $TARGET_DOMAIN -r
  artifacts:
    paths:
      - Recon/
    expire_in: 1 week
  only:
    - schedules
```

#### Advanced: Parallel Targets with Reports

```yaml
# .gitlab-ci.yml
stages:
  - scan
  - report

variables:
  TARGETS: "target1.com target2.com target3.com"

.scan_template: &scan_template
  image: six2dez/reconftw:main
  before_script:
    - echo "$SECRETS_CFG" > secrets.cfg
  artifacts:
    paths:
      - Recon/
    expire_in: 2 weeks

scan_target1:
  <<: *scan_template
  stage: scan
  script:
    - ./reconftw.sh -d target1.com -r
  only:
    - schedules

scan_target2:
  <<: *scan_template
  stage: scan
  script:
    - ./reconftw.sh -d target2.com -r
  only:
    - schedules

generate_report:
  stage: report
  image: python:3.9
  dependencies:
    - scan_target1
    - scan_target2
  script:
    - pip install jinja2
    - python scripts/generate_report.py
  artifacts:
    paths:
      - report.html
    expire_in: 4 weeks
```

### Jenkins Pipeline

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    parameters {
        string(name: 'TARGET', defaultValue: 'example.com', description: 'Target domain')
        choice(name: 'MODE', choices: ['passive', 'recon', 'all'], description: 'Scan mode')
    }
    
    environment {
        SHODAN_API_KEY = credentials('shodan-api-key')
        GITHUB_TOKEN = credentials('github-token')
    }
    
    stages {
        stage('Setup') {
            steps {
                sh '''
                    git clone https://github.com/six2dez/reconftw.git || true
                    cd reconftw && git pull && ./install.sh
                '''
            }
        }
        
        stage('Configure') {
            steps {
                sh '''
                    cd reconftw
                    echo "SHODAN_API_KEY=${SHODAN_API_KEY}" > secrets.cfg
                    echo "GITHUB_TOKEN=${GITHUB_TOKEN}" >> secrets.cfg
                '''
            }
        }
        
        stage('Scan') {
            steps {
                sh '''
                    cd reconftw
                    MODE_FLAG=""
                    case "${MODE}" in
                        passive) MODE_FLAG="-p" ;;
                        recon) MODE_FLAG="-r" ;;
                        all) MODE_FLAG="-a" ;;
                    esac
                    ./reconftw.sh -d ${TARGET} ${MODE_FLAG}
                '''
            }
        }
        
        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'reconftw/Recon/**/*', fingerprint: true
            }
        }
        
        stage('Notify') {
            when {
                expression {
                    return fileExists("reconftw/Recon/${TARGET}/nuclei_output/critical_json.txt") || \
                           fileExists("reconftw/Recon/${TARGET}/nuclei_output/high_json.txt")
                }
            }
            steps {
                script {
                    def vulnCount = sh(
                        script: 'cat reconftw/Recon/${TARGET}/nuclei_output/*_json.txt 2>/dev/null | jq -s length',
                        returnStdout: true
                    ).trim()
                    
                    if (vulnCount.toInteger() > 0) {
                        slackSend(
                            color: 'danger',
                            message: "reconFTW found ${vulnCount} vulnerabilities on ${TARGET}"
                        )
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
```

### Cron-based (Linux Server)

```bash
#!/bin/bash
# /opt/reconftw/scripts/weekly_scan.sh

RECONFTW_PATH="/opt/reconftw"
TARGETS_FILE="/opt/reconftw/targets.txt"
RESULTS_PATH="/var/reconftw-results"
DATE=$(date +%Y-%m-%d)

# Ensure directories exist
mkdir -p "$RESULTS_PATH/$DATE"

# Run scans
while IFS= read -r target; do
    echo "[$(date)] Starting scan for $target"
    
    cd "$RECONFTW_PATH"
    ./reconftw.sh -d "$target" -r -o "$RESULTS_PATH/$DATE/$target"
    
    # Generate diff if previous results exist
    PREV=$(ls -1 "$RESULTS_PATH" | grep -v "$DATE" | sort -r | head -1)
    if [[ -n "$PREV" && -d "$RESULTS_PATH/$PREV/$target" ]]; then
        echo "[$(date)] Generating diff against $PREV"
        diff -rq "$RESULTS_PATH/$PREV/$target/subdomains" \
                 "$RESULTS_PATH/$DATE/$target/subdomains" \
                 > "$RESULTS_PATH/$DATE/$target/diff_subs.txt" 2>/dev/null
    fi
    
    echo "[$(date)] Completed scan for $target"
done < "$TARGETS_FILE"

# Cleanup old results (keep 4 weeks)
find "$RESULTS_PATH" -maxdepth 1 -type d -mtime +28 -exec rm -rf {} \;

# Send summary
TOTAL_VULNS=$(cat "$RESULTS_PATH/$DATE"/*/nuclei_output/*_json.txt 2>/dev/null | jq -s 'length')
echo "Weekly scan complete. Total vulnerabilities: $TOTAL_VULNS" | \
    mail -s "reconFTW Weekly Report" security@company.com
```

**Cron entry:**
```bash
# /etc/cron.d/reconftw
0 2 * * 0 root /opt/reconftw/scripts/weekly_scan.sh >> /var/log/reconftw.log 2>&1
```

---

## Post-Deployment

### Verify Installation

```bash
# Check tools
./reconftw.sh --check-tools

# System health
./reconftw.sh --health-check

# Test scan
./reconftw.sh -d example.com -p
```

### Configure API Keys

```bash
# Edit secrets.cfg
nano secrets.cfg

# Add keys
SHODAN_API_KEY="your_key"
GITHUB_TOKEN="your_token"
# ... more keys
```

### Update Tools

```bash
# Update reconFTW and reinstall tools
git pull
./install.sh
```

---

## Resource Management

### Monitor Usage

```bash
# CPU and memory
htop

# Disk usage
df -h

# Network
iftop
```

### Cleanup

```bash
# Clear old results
rm -rf Recon/old-target/

# Clear temp files
rm -rf Recon/*/tmp/*

# Docker cleanup
docker system prune -a
```

---

## Next Steps

- **[Configuration](../04-configuration/configuration.md)** - Customize settings
- **[Usage Guide](../03-usage/usage.md)** - Run scans
- **[Axiom Integration](../08-integrations/axiom.md)** - Distributed scanning
