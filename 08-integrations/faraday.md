# Faraday Integration

Faraday is a collaborative vulnerability management platform. reconFTW integrates with Faraday to automatically import scan results.

---

## What is Faraday?

Faraday provides:

- **Centralized vulnerability database**
- **Team collaboration features**
- **Report generation**
- **Integration with 80+ security tools**
- **Workspace management**

<!-- IMAGE PLACEHOLDER
**Image: faraday-integration.png**
Description: Diagram showing data flow from reconFTW to Faraday
- reconFTW scan → Results (nuclei, nmap) → Faraday API → Faraday Dashboard
- Show workspace with vulnerabilities listed
-->

---

## Prerequisites

### Faraday Installation

```bash
# Docker installation (recommended)
docker pull faradaysec/faraday:latest
docker run -d --name faraday -p 5985:5985 faradaysec/faraday

# Or native installation
pip install faradaysec
```

### Faraday Setup

1. Access Faraday web interface: `http://localhost:5985`
2. Create admin account
3. Create workspace for your project
4. Generate API token

---

## Configuration in reconFTW

### Enable Faraday Integration

```bash
# In reconftw.cfg

# Enable Faraday
FARADAY=true

# Faraday server URL
FARADAY_URL="http://localhost:5985"

# Workspace name
FARADAY_WORKSPACE="reconftw"

# API credentials (in secrets.cfg)
# FARADAY_API_TOKEN="your_token"
```

### secrets.cfg Setup

```bash
# In secrets.cfg
FARADAY_API_TOKEN="your_api_token_here"
```

---

## Data Imported to Faraday

reconFTW automatically sends:

| Data Type | Source | Faraday Entity |
|-----------|--------|----------------|
| Hosts | Subdomain resolution | Host |
| Services | Port scan (nmap) | Service |
| Vulnerabilities | Nuclei results | Vulnerability |
| CVEs | Nmap vulners script | Vulnerability |

---

## Integration Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Faraday Integration Flow                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐                                                    │
│  │   reconFTW   │                                                    │
│  │    Scan      │                                                    │
│  └──────┬───────┘                                                    │
│         │                                                            │
│    ┌────┴────┬────────────┬─────────────┐                           │
│    ▼         ▼            ▼             ▼                           │
│ ┌──────┐ ┌───────┐  ┌──────────┐  ┌──────────┐                     │
│ │Nuclei│ │ Nmap  │  │  Hosts   │  │ Services │                     │
│ │ JSON │ │  XML  │  │   IPs    │  │  Ports   │                     │
│ └──┬───┘ └───┬───┘  └────┬─────┘  └────┬─────┘                     │
│    │         │           │             │                            │
│    └────┬────┴───────────┴─────────────┘                           │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────┐                                                    │
│  │ faraday-cli  │ (or API calls)                                    │
│  └──────┬───────┘                                                    │
│         │                                                            │
│         │ POST /api/v3/ws/{workspace}/...                           │
│         ▼                                                            │
│  ┌──────────────────────────────────────────────────────┐           │
│  │                 Faraday Server                        │           │
│  ├──────────────────────────────────────────────────────┤           │
│  │  ┌─────────┐  ┌─────────────┐  ┌────────────────┐   │           │
│  │  │  Hosts  │  │  Services   │  │ Vulnerabilities│   │           │
│  │  │  Table  │  │   Table     │  │     Table      │   │           │
│  │  └─────────┘  └─────────────┘  └────────────────┘   │           │
│  └──────────────────────────────────────────────────────┘           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Workspace Management

### Create Workspace

```bash
# Via Faraday CLI
faraday-cli create_ws reconftw

# Via API
curl -X POST "http://localhost:5985/api/v3/ws" \
  -H "Authorization: Token $FARADAY_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "reconftw"}'
```

### Workspace Strategy

| Strategy | Description | Use Case |
|----------|-------------|----------|
| Per-target | One workspace per domain | Isolated scans |
| Per-program | One workspace per bug bounty | Program tracking |
| Unified | Single workspace | Overview of all targets |

---

## Viewing Results

### Faraday Web Interface

1. Open `http://localhost:5985`
2. Select workspace (`reconftw`)
3. Navigate to:
   - **Hosts:** All discovered hosts with IPs
   - **Services:** Ports and services per host
   - **Vulns:** All vulnerabilities by severity

### Faraday CLI

```bash
# List hosts
faraday-cli host -w reconftw

# List vulnerabilities
faraday-cli vuln -w reconftw

# Export report
faraday-cli report -w reconftw -o report.pdf
```

---

## Report Generation

### Built-in Reports

Faraday generates reports in multiple formats:

- PDF
- HTML
- Markdown
- CSV

### Custom Templates

```bash
# Create custom report template
faraday-cli report -w reconftw --template custom.html
```

---

## Advanced Configuration

### Severity Mapping

reconFTW maps nuclei severities to Faraday:

| Nuclei Severity | Faraday Severity |
|-----------------|------------------|
| critical | Critical |
| high | High |
| medium | Medium |
| low | Low |
| info | Informational |

### Custom Fields

```bash
# Add custom data to vulnerabilities
# In reconftw.cfg
FARADAY_CUSTOM_FIELDS=true
```

### Bulk Import

For large scans, results are batched:

```bash
# Batch size configuration
FARADAY_BATCH_SIZE=100
```

---

## Troubleshooting

### Connection Issues

```bash
# Test Faraday connectivity
curl -X GET "http://localhost:5985/api/v3/info" \
  -H "Authorization: Token $FARADAY_API_TOKEN"
```

### Authentication Errors

```bash
# Verify token
# Check secrets.cfg has correct token
# Regenerate token in Faraday web UI if needed
```

### Missing Data

1. Check workspace exists
2. Verify scan completed successfully
3. Review Faraday logs: `docker logs faraday`

### Duplicate Entries

Faraday deduplicates by:
- Host: IP address
- Service: IP + port + protocol
- Vulnerability: Name + host + service

---

## Best Practices

1. **Workspace naming:** Use consistent naming convention
2. **Token security:** Keep API token in secrets.cfg
3. **Regular cleanup:** Archive old workspaces
4. **Backup:** Export workspaces regularly
5. **Access control:** Use Faraday roles for team access

---

## Alternative: Manual Import

If automatic integration fails, import manually:

```bash
# Import nmap results
faraday-cli tool nmap -w reconftw hosts/portscan_active.xml

# Import nuclei results
faraday-cli tool nuclei -w reconftw nuclei_output/*_json.txt
```

---

## Next Steps

- **[Deployment Guide](../09-deployment/deployment.md)** - Full setup
- **[Output Interpretation](../07-output/output.md)** - Understanding results
