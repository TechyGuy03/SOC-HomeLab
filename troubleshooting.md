# Troubleshooting & Operational Scenarios

## Objective

Document realistic failure conditions encountered during homelab deployment and security tooling integration.

Focus areas:

• Root cause analysis  
• Diagnostic methodology  
• Failure pattern recognition  
• Operational decision making  

---

## Scenario 1 – Dashboard Unreachable

### Observed Behavior

Browser error:

**ERR_CONNECTION_REFUSED**

System state:

• Wazuh services installed  
• System accessible via SSH  
• No dashboard response  

---

### Diagnostic Process

Service validation:

```bash
sudo systemctl status wazuh-dashboard
```

Port inspection:

```bash
sudo ss -lntp | grep 443
```

Firewall validation:

```bash
sudo ufw status
```

---

### Root Cause

Issue:

Host firewall blocking inbound HTTPS traffic.

Port 443 not permitted.

---

### Resolution

```bash
sudo ufw allow 443/tcp
sudo ufw reload
```

---

### Key Lesson

Connectivity failures are frequently:

• Firewall-related  
• Not application failures  

Always validate network controls early.

---

## Scenario 2 – Dashboard Service Crash

### Observed Behavior

Dashboard service repeatedly restarting.

System logs:

```bash
sudo journalctl -u wazuh-dashboard -n 50 --no-pager
```

Error:

```bash
ENOENT: no such file or directory
```

---

### Diagnostic Process

Configuration inspection:

```bash
sudo nano /etc/wazuh-dashboard/opensearch_dashboards.yml
```

Certificate directory validation:

```bash
sudo ls -lah /etc/wazuh-dashboard/certs
```

---

### Root Cause

Issue:

TLS configuration referencing certificate filenames that did not exist.

Mismatch between:

• Configured filenames  
• Actual certificate files  

---

### Resolution

Updated configuration:

```yaml
server.ssl.key: "/etc/wazuh-dashboard/certs/wazuh-dashboard-key.pem"
server.ssl.certificate: "/etc/wazuh-dashboard/certs/wazuh-dashboard.pem"
```

Restarted service:

```bash
sudo systemctl restart wazuh-dashboard
```

---

### Key Lesson

TLS misconfiguration can:

• Prevent service startup  
• Mimic connectivity failures  

Logs are authoritative truth source.

---

## Scenario 3 – Agent Enrollment Failure

### Observed Behavior

Agent registration failure:

```bash
agent-auth: ERROR: Unable to connect to enrollment service
```

---

### Diagnostic Process

Connectivity validation:

```bash
nc -vz <manager-ip> 1515
```

Service verification:

```bash
sudo systemctl status wazuh-manager
```

Version validation:

```bash
apt-cache policy wazuh-agent
```

---

### Root Cause

Issue:

Agent version newer than manager version.

Wazuh compatibility requirement:

Agent version must be ≤ manager version.

---

### Resolution

Installed compatible agent version.

Re-ran enrollment:

```bash
sudo /var/ossec/bin/agent-auth -m <manager-ip>
```

---

### Key Lesson

Telemetry failures may originate from:

• Version mismatch  
• Not network or firewall issues  

Compatibility matters.

---

## Scenario 4 – Certificate Trust Confusion

### Observed Behavior

Browser warning:

**Certificate Not Trusted**

Despite:

• Valid TLS handshake  
• Let's Encrypt certificate  

---

### Root Cause

Issue:

Internal DNS resolution + certificate validation behavior varies by OS/browser trust store.

Certificate encryption ≠ certificate trust.

---

### Key Lesson

Important distinction:

• Encryption protects confidentiality  
• Trust validates identity  

Warnings do not imply unencrypted traffic.

---

## Common Failure Patterns Observed

Across scenarios:

• Firewall restrictions  
• TLS misconfiguration  
• Service dependency failures  
• Version mismatches  
• DNS resolution dependencies  

---

## Diagnostic Strategy Principles

### Validate From Outside-In

1. Network connectivity  
2. Firewall controls  
3. Port bindings  
4. Service state  
5. Application logs  

---

### Logs Over Assumptions

Always prioritize:

```bash
journalctl
systemctl
ss
```

Over speculation.

---

### Failures Are Normal

Security tooling introduces:

• Complexity  
• Dependencies  
• Operational friction  

---

## Why This Matters for SOC / IR Roles

Real environments constantly encounter:

• Broken agents  
• Missing telemetry  
• Certificate failures  
• Connectivity issues  
• Service crashes  

Understanding failure mechanics improves:

• Triage accuracy  
• Investigation speed  
• False positive reduction  
• Incident response quality  

---

## Future Troubleshooting Areas

Planned exploration:

• Log parsing failures  
• Detection rule tuning issues  
• High ingestion load scenarios  
• Indexer performance bottlenecks  
• Reverse proxy misconfigurations
