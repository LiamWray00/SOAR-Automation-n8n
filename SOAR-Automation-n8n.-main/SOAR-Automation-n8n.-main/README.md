# SOAR Automation Lab — Wazuh + n8n + AI Triage

Hello! This is my hands-on home lab building a SOAR (Security Orchestration, Automation and Response) pipeline. I deployed Wazuh as a self-hosted SIEM, connected a Windows endpoint as an agent, and built an n8n automation workflow that ingests Wazuh alerts, enriches them with AbuseIPDB and VirusTotal threat intelligence, uses AI to summarize findings, and emails a plain-English triage report — all automatically with zero analyst input.

## Architecture
 
```
Windows Laptop (Wazuh Agent)
   │  Shipping logs → Wazuh Manager
   ▼
Wazuh SIEM (self-hosted on Ubuntu VM / Proxmox)
   │  Alert fired → webhook
   ▼
n8n Automation (self-hosted on same Ubuntu VM)
   │  Extract source IP
   │  AbuseIPDB lookup
   │  VirusTotal lookup
   │  Claude AI triage summary
   ▼
Email Alert — plain English analyst report
```
 
## Highlights
 
- **Deployed Wazuh on Ubuntu Server** hosted on Proxmox — full self-hosted SIEM with no cloud dependency
- **Connected a real Windows 11 endpoint** as a Wazuh agent, generating live security events
- **Auto-detected MITRE ATT&CK T1562.001 and T1110** (Defense Evasion, Credential Access) with zero manual configuration
- **Built a webhook integration** from Wazuh to n8n — alerts flow automatically in under 1 second
- **Queried AbuseIPDB and VirusTotal** for multi-source IP threat intelligence
- **Added Claude AI summarization** to convert raw JSON threat data into plain English triage reports
- **Full pipeline runs in under 3 seconds** from Wazuh detection to email delivery
## What this demonstrates
 
- Self-hosted SIEM deployment and configuration
- Real endpoint agent deployment and management
- Webhook-based alert forwarding and automation
- Multi-source threat intelligence integration
- AI-assisted alert triage
- SOAR pipeline design and implementation
## Lessons Learned
 
- Wazuh's integration block must be placed inside `</ossec_config>` — placement after it breaks XML parsing and crashes the manager
- Docker containers need a persistent volume (`-v n8n_data`) and `--restart unless-stopped` or data is lost on unclean shutdown
- Wazuh's default alert level of 3 floods n8n — raising to 7 filters to meaningful security alerts only
- `shuffle` is required as the integration name for custom webhooks in Wazuh — `custom-webhook` is not recognized
- Source IPs from SSH brute force appear in the `text` field as `rhost=x.x.x.x` and require regex extraction
- Claude API requires `anthropic-version: 2023-06-01` header on every request
## Day-by-Day Journal
 
- [Day 1](Journal/Day-1.md) — Wazuh install, Ubuntu VM setup, dashboard access
- [Day 2](Journal/Day-2.md) — Windows agent enrollment, first MITRE ATT&CK detection
- [Day 3](Journal/Day-3.md) — n8n install, IP enrichment workflow, AbuseIPDB integration
- [Day 4](Journal/Day-4.md) — Wazuh webhook to n8n, alert pipeline connected
- [Day 5](Journal/Day-5.md) — VirusTotal, IP extraction Code node, email alert
- [Day 6](Journal/Day-6.md) — Claude AI triage summary node added, full pipeline complete
## Workflow Configs
 
See `/workflows` for exported n8n workflow JSON files:
- `ip-reputation-checker.json` — manual IP lookup form workflow
- `wazuh-alert-triage.json` — automated Wazuh alert enrichment pipeline with AI
## Environment
 
| Component | Details |
|---|---|
| Hypervisor | Proxmox VE |
| SIEM | Wazuh 4.7.5 (self-hosted) |
| Automation | n8n (self-hosted via Docker) |
| Endpoint | Windows 11 Pro (Wazuh Agent v4.14.7) |
| Threat Intel | AbuseIPDB API, VirusTotal API |
| AI | Claude Sonnet (Anthropic API) |
| Notification | Gmail SMTP |
| Host OS | Ubuntu Server 24.04 |
 
## Author
Liam Wray | [LinkedIn](http://www.linkedin.com/in/liam-wray-9002373a6) | [GitHub](https://github.com/LiamWray00)
