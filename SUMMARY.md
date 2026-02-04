# Table of Contents

## Welcome

* [Introduction](README.md)
* [First 30 Minutes](first-30-minutes.md)
* [FAQ](FAQ.md)
* [Glossary](GLOSSARY.md)

## Getting Started

* [Installation & Setup](01-getting-started/getting-started.md)
  * [System Requirements](01-getting-started/getting-started.md#system-requirements)
  * [Installation Methods](01-getting-started/getting-started.md#installation-methods)
  * [First Scan](01-getting-started/getting-started.md#your-first-scan)
  * [Verifying Installation](01-getting-started/getting-started.md#verifying-installation)
  * [Updating reconFTW](01-getting-started/getting-started.md#updating-reconftw)

## Understanding reconFTW

* [Concepts & Architecture](02-concepts/concepts.md)
  * [What is reconFTW](02-concepts/concepts.md#what-is-reconftw)
  * [Reconnaissance Methodology](02-concepts/concepts.md#reconnaissance-methodology)
  * [Architecture Overview](02-concepts/concepts.md#architecture-overview)
  * [Data Flow](02-concepts/concepts.md#data-flow)
  * [Scan Phases](02-concepts/concepts.md#scan-phases)
  * [Checkpoint System](02-concepts/concepts.md#checkpoint-system)
  * [OPSEC and Legal](02-concepts/concepts.md#opsec-and-legal)
  * [Recommended Workflows](02-concepts/concepts.md#recommended-workflows)

* [Reconnaissance Methodology Deep Dive](02-concepts/recon-methodology.md)
  * [Why This Order](02-concepts/recon-methodology.md#why-this-order)
  * [Passive vs Active](02-concepts/recon-methodology.md#passive-vs-active)
  * [Wildcard Handling](02-concepts/recon-methodology.md#wildcard-handling)
  * [Rate Limiting Strategy](02-concepts/recon-methodology.md#rate-limiting-strategy)
  * [Scope Management](02-concepts/recon-methodology.md#scope-management)

## Usage

* [Command Line Guide](03-usage/usage.md)
  * [Target Options](03-usage/usage.md#target-options)
  * [Scan Modes](03-usage/usage.md#scan-modes)
  * [Scope Management](03-usage/usage.md#scope-management)
  * [Advanced Flags](03-usage/usage.md#advanced-flags)
  * [Usage Examples](03-usage/usage.md#usage-examples)

## Configuration

* [Configuration Reference](04-configuration/configuration.md)
  * [General Settings](04-configuration/configuration.md#general-settings)
  * [API Keys & Tokens](04-configuration/configuration.md#api-keys-and-tokens)
  * [Module Toggles](04-configuration/configuration.md#module-toggles)
  * [Threading & Rate Limits](04-configuration/configuration.md#threading-and-rate-limits)
  * [Timeouts](04-configuration/configuration.md#timeouts)
  * [Wordlists](04-configuration/configuration.md#wordlists)
  * [Axiom Settings](04-configuration/configuration.md#axiom-settings)
  * [Faraday Settings](04-configuration/configuration.md#faraday-settings)
  * [AI Settings](04-configuration/configuration.md#ai-settings)

## Modules

* [Modules Overview](05-modules/README.md)

* [OSINT Module](05-modules/osint.md)
  * [Google Dorks](05-modules/osint.md#google-dorks)
  * [GitHub Analysis](05-modules/osint.md#github-analysis)
  * [Metadata Extraction](05-modules/osint.md#metadata-extraction)
  * [API Leaks](05-modules/osint.md#api-leaks)
  * [Email Harvesting](05-modules/osint.md#email-harvesting)
  * [Domain Intelligence](05-modules/osint.md#domain-intelligence)
  * [Cloud Enumeration](05-modules/osint.md#cloud-enumeration)

* [Subdomain Module](05-modules/subdomains.md)
  * [New Features (v3.x)](05-modules/subdomains.md#new-features-v3x)
  * [Passive Enumeration](05-modules/subdomains.md#passive-enumeration)
  * [Certificate Transparency](05-modules/subdomains.md#certificate-transparency)
  * [DNS Bruteforce](05-modules/subdomains.md#dns-bruteforce)
  * [Permutations](05-modules/subdomains.md#permutations)
  * [Recursive Enumeration](05-modules/subdomains.md#recursive-enumeration)
  * [Subdomain Takeover](05-modules/subdomains.md#subdomain-takeover)
  * [DNS Analysis](05-modules/subdomains.md#dns-analysis)

* [Subdomain Deep Dive](05-modules/subdomains-deep-dive.md)
  * [Execution Pipeline](05-modules/subdomains-deep-dive.md#execution-pipeline)
  * [Passive Techniques](05-modules/subdomains-deep-dive.md#passive-techniques)
  * [Active DNS Resolution](05-modules/subdomains-deep-dive.md#active-dns-resolution)
  * [Deep Wildcard Detection](05-modules/subdomains-deep-dive.md#deep-wildcard-detection)
  * [Sensitive Domain Exclusion](05-modules/subdomains-deep-dive.md#sensitive-domain-exclusion)

* [Web Analysis Module](05-modules/web-analysis.md)
  * [HTTP Probing](05-modules/web-analysis.md#http-probing)
  * [Screenshots](05-modules/web-analysis.md#screenshots)
  * [URL Collection](05-modules/web-analysis.md#url-collection)
  * [JavaScript Analysis](05-modules/web-analysis.md#javascript-analysis)
  * [Directory Fuzzing](05-modules/web-analysis.md#directory-fuzzing)
  * [CMS Detection](05-modules/web-analysis.md#cms-detection)
  * [Parameter Discovery](05-modules/web-analysis.md#parameter-discovery)

* [Vulnerability Module](05-modules/vulnerabilities.md)
  * [Nuclei Scanning](05-modules/vulnerabilities.md#nuclei-scanning)
  * [XSS Testing](05-modules/vulnerabilities.md#xss-testing)
  * [SQL Injection](05-modules/vulnerabilities.md#sql-injection)
  * [SSRF Testing](05-modules/vulnerabilities.md#ssrf-testing)
  * [LFI/SSTI Testing](05-modules/vulnerabilities.md#lfi-ssti-testing)
  * [Other Vulnerabilities](05-modules/vulnerabilities.md#other-vulnerabilities)

* [Host Module](05-modules/hosts.md)
  * [Port Scanning](05-modules/hosts.md#port-scanning)
  * [CDN Detection](05-modules/hosts.md#cdn-detection)
  * [WAF Detection](05-modules/hosts.md#waf-detection)
  * [Geolocation](05-modules/hosts.md#geolocation)

## Tools Reference

* [Integrated Tools](06-tools/tools.md)
  * [OSINT Tools](06-tools/tools.md#osint-tools)
  * [Subdomain Tools](06-tools/tools.md#subdomain-tools)
  * [Web Analysis Tools](06-tools/tools.md#web-analysis-tools)
  * [Vulnerability Tools](06-tools/tools.md#vulnerability-tools)
  * [Utility Tools](06-tools/tools.md#utility-tools)

## Output

* [Output Interpretation](07-output/output.md)
  * [Directory Structure](07-output/output.md#directory-structure)
  * [Subdomain Files](07-output/output.md#subdomain-files)
  * [Web Files](07-output/output.md#web-files)
  * [Host Files](07-output/output.md#host-files)
  * [OSINT Files](07-output/output.md#osint-files)
  * [Vulnerability Files](07-output/output.md#vulnerability-files)
  * [Log Files](07-output/output.md#log-files)
  * [Hotlist](07-output/output.md#hotlist)

## Integrations

* [Axiom Integration](08-integrations/axiom.md)
  * [What is Axiom](08-integrations/axiom.md#what-is-axiom)
  * [Setup & Configuration](08-integrations/axiom.md#prerequisites)
  * [Fleet Management](08-integrations/axiom.md#fleet-management)
  * [Distributed Scanning](08-integrations/axiom.md#distributed-scanning-flow)
  * [Troubleshooting](08-integrations/axiom.md#troubleshooting)

* [Faraday Integration](08-integrations/faraday.md)
  * [What is Faraday](08-integrations/faraday.md#what-is-faraday)
  * [Setup](08-integrations/faraday.md#setup)
  * [Automatic Import](08-integrations/faraday.md#automatic-import)
  * [Reports](08-integrations/faraday.md#reports)

## Deployment

* [Deployment Guide](09-deployment/deployment.md)
  * [Local Installation](09-deployment/deployment.md#local-installation)
  * [Docker](09-deployment/deployment.md#docker)
  * [Terraform & Ansible](09-deployment/deployment.md#terraform-and-ansible)
  * [VPS Setup](09-deployment/deployment.md#vps-setup)
  * [CI/CD Integration](09-deployment/deployment.md#cicd-integration)

## Advanced

* [Advanced Usage](10-advanced/advanced.md)
  * [Custom Functions](10-advanced/advanced.md#custom-functions)
  * [Plugin System](10-advanced/advanced.md#plugin-system)
  * [Performance Tuning](10-advanced/advanced.md#performance-tuning)
  * [Multi-Target Strategies](10-advanced/advanced.md#multi-target-strategies)
  * [Incremental Mode](10-advanced/advanced.md#incremental-mode)

## Guides

* [Performance Tuning](tuning.md)
  * [Quick Tuning Profiles](tuning.md#quick-tuning-profiles)
  * [Understanding DEEP Mode](tuning.md#understanding-deep-mode)
  * [Thread Optimization](tuning.md#thread-optimization)
  * [Rate Limiting Strategies](tuning.md#rate-limiting-strategies)
  * [Parallelization](tuning.md#parallelization)
  * [Axiom Scaling](tuning.md#axiom-scaling)

* [Data Model & I/O](data-model.md)
  * [Input Formats](data-model.md#input-formats)
  * [Output Directory Structure](data-model.md#output-directory-structure)
  * [Key Output Files](data-model.md#key-output-files)
  * [Module I/O Matrix](data-model.md#module-inputoutput-matrix)
  * [Checkpoint System](data-model.md#checkpoint-system)

* [OPSEC & Legal](opsec-legal.md)
  * [Know Before You Scan](opsec-legal.md#know-before-you-scan)
  * [Intrusiveness Matrix](opsec-legal.md#intrusiveness-matrix)
  * [Staying Under the Radar](opsec-legal.md#staying-under-the-radar)
  * [Legal Framework](opsec-legal.md#legal-framework-by-region)
  * [Bug Bounty Guidelines](opsec-legal.md#bug-bounty-guidelines)

* [Case Studies](case-studies.md)
  * [Bug Bounty Rush](case-studies.md#case-study-1-bug-bounty---new-program-launch)
  * [Enterprise Assessment](case-studies.md#case-study-2-security-assessment---enterprise-client)
  * [Red Team Distributed](case-studies.md#case-study-3-red-team---distributed-scanning-with-axiom)
  * [CI/CD Monitoring](case-studies.md#case-study-4-continuous-monitoring---cicd-integration)

## Help

* [Troubleshooting](11-troubleshooting/troubleshooting.md)
  * [Installation Issues](11-troubleshooting/troubleshooting.md#installation-issues)
  * [Runtime Errors](11-troubleshooting/troubleshooting.md#runtime-errors)
  * [Performance Issues](11-troubleshooting/troubleshooting.md#performance-issues)
  * [Axiom Issues](11-troubleshooting/troubleshooting.md#axiom-issues)
  * [Getting Help](11-troubleshooting/troubleshooting.md#getting-help)
