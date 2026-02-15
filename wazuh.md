# Wazuh Deployment & Configuration

## Objective

Deploy and integrate Wazuh into the homelab to simulate a realistic SOC telemetry and detection pipeline.

Primary goals:

• Centralized log visibility  
• Agent-based monitoring  
• Detection experimentation  
• Infrastructure troubleshooting  
• TLS / certificate management  

---

## Architecture Role

Within this lab, Wazuh functions as:

• Security Monitoring Platform  
• Log Aggregation Layer  
• Detection Experimentation Surface  
• Investigation Interface  

---
## Environment Profile:

• Virtualized infrastructure (Proxmox)
• Segmented internal network
• DNS-based certificate validation
• Mixed endpoint telemetry sources

---
## Core Components

### Wazuh Manager
Responsible for:

• Agent communication  
• Event processing  
• Rule evaluation  
• Alert generation  

---

### Wazuh Indexer
Responsible for:

• Event storage  
• Data indexing  
• Search performance  
• Dashboard data source  

---

### Wazuh Dashboard
Responsible for:

• Visualization  
• Investigation workflows  
• Alert analysis  
• System monitoring  

---

## Initial Deployment Model

Deployment Type:

Single-node lab deployment

Rationale:

• Simplified troubleshooting  
• Reduced infrastructure overhead  
• Faster iteration cycles  

---

## Installation Summary

High-level installation flow:

```bash
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

Installation mode:

All-in-one deployment (Manager, Indexer, Dashboard)

---

## Post Installation Validation

Service verification:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

Port validation:

```bash
sudo ss -lntp
```

Dashboard test:

```bash
curl -kI https://127.0.0.1
```

---

## Host Firewall Configuration (UFW)

### Problem Encountered

Initial dashboard access failure:

**ERR_CONNECTION_REFUSED**

Despite:

• Services running  
• Ports listening locally  
• Correct IP address  

---

### Root Cause Analysis

Issue:

Ubuntu host firewall blocking inbound HTTPS traffic.

Validation test:

```bash
sudo ufw status
```

Observed behavior:

Port 443 not permitted.

---

### Resolution

Allowed HTTPS traffic:

```bash
sudo ufw allow 443/tcp
```

Reloaded firewall rules:

```bash
sudo ufw reload
```

---

### Validation

Connectivity verification:

```bash
curl -kI https://<wazuh-server-ip>
```

Result:

Dashboard accessible.

---

### Key Lesson

Security tooling failures are frequently caused by:

• Host firewall restrictions  
• Network ACLs  
• Port exposure assumptions  

Not necessarily application misconfiguration.

---
## TLS / Certificate Configuration

### Problem Encountered

Initial dashboard failure:

Dashboard unreachable despite firewall allowance

Root cause:

Dashboard service failing to start due to missing certificate files.

Observed error:

```bash
ENOENT: no such file or directory, open '/etc/wazuh-dashboard/certs/dashboard-key.pem'
```

---

### Root Cause Analysis

Issue:

Dashboard configuration referenced certificate filenames that did not exist.

Actual files present:

```bash
/etc/wazuh-dashboard/certs/wazuh-dashboard.pem
/etc/wazuh-dashboard/certs/wazuh-dashboard-key.pem
```

Mismatch:

```yaml
server.ssl.key: "/etc/wazuh-dashboard/certs/dashboard-key.pem"
server.ssl.certificate: "/etc/wazuh-dashboard/certs/dashboard.pem"
```

---

### Resolution

Updated dashboard configuration:

```yaml
server.ssl.key: "/etc/wazuh-dashboard/certs/wazuh-dashboard-key.pem"
server.ssl.certificate: "/etc/wazuh-dashboard/certs/wazuh-dashboard.pem"
```

Restarted service:

```bash
sudo systemctl restart wazuh-dashboard
```

Validation:

```bash
sudo systemctl status wazuh-dashboard
sudo journalctl -u wazuh-dashboard -n 50 --no-pager```

---

## Certificate Authority Integration

### Objective

Replace self-signed certificates with trusted certificates issued via Let's Encrypt using ACME DNS validation (Cloudflare).

---

### Method

Certificate Issuance Model:

DNS-based validation via Cloudflare API

Advantages:

• No port forwarding required  
• Works with dynamic WAN IP  
• Suitable for internal-only services  

---

### ACME Process Summary

Certificate issued for:

• techysec.com  
• *.techysec.com  

Validation Flow:

```bash
Adding TXT record _acme-challenge.techysec.com
Verification successful
Certificate issued
```

---

### Deployment Steps

Created custom certificate directory:

```bash
sudo mkdir /etc/wazuh-dashboard/custom-certs
```

Updated ownership:

```bash
sudo chown -R wazuh-dashboard:wazuh-dashboard /etc/wazuh-dashboard/custom-certs
```

Permissions hardening:

```bash
sudo chmod 600 /etc/wazuh-dashboard/custom-certs/*
```

---

### Dashboard Configuration Update

Modified TLS paths:

```yaml
server.ssl.key: "/etc/wazuh-dashboard/custom-certs/wazuh-key.pem"
server.ssl.certificate: "/etc/wazuh-dashboard/custom-certs/wazuh.pem"
```

Restarted dashboard:

```bash
sudo systemctl restart wazuh-dashboard
```

---

## Agent Enrollment

### Objective

Simulate enterprise endpoint telemetry ingestion.

---

### Agent Installation Example (Linux)

```bash
curl -sO https://packages.wazuh.com/4.x/wazuh-agent.sh
sudo bash wazuh-agent.sh
```

---

### Enrollment Workflow

Agent registration:

```bash
sudo /var/ossec/bin/agent-auth -m <manager-ip>
```

Verification:

Dashboard → Agents View

---

## Observed Learning Scenarios

Realistic issues encountered:

• TLS certificate trust problems  
• Service startup failures  
• Port binding validation  
• Agent connectivity troubleshooting  
• DNS resolution dependencies  
• Certificate mismatch failures  

---

## Key Technical Lessons

### Certificates Are Operational Dependencies
TLS misconfiguration can fully prevent service startup.

---

### Error Logs Are Primary Truth Sources
Systemd + journalctl output provided immediate root cause visibility.

---

### DNS-Based Validation Simplifies Dynamic Environments
DNS challenges eliminate WAN IP stability requirements.

---

### Monitoring Platforms Are Infrastructure-Heavy
Security tooling introduces operational complexity beyond basic installation.

---

## Why This Matters for SOC / Detection Roles

Real SOC environments frequently involve:

• Broken telemetry pipelines  
• Agent failures  
• Certificate problems  
• Connectivity issues  
• Parsing failures  
• Infrastructure misconfigurations  

Understanding failure mechanics is a core defensive skill.

---

## Future Expansion Areas

Planned explorations:

• Multi-node Wazuh deployment  
• Log forwarding integrations  
• Detection rule experimentation  
• Adversary simulation workflows  
• Reverse proxy integration  
• TLS hardening scenarios
• Detection rule development & tuning
